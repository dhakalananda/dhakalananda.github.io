---
title: "The Most Beautiful Payment Bypass - Or Is It?"
description: "Technical analysis of a beautiful payment bypass in the Prestashop integration of Stripe."
---

Technical analysis of a payment bypass in the Prestashop integration of Stripe.

## Background

Back in July 2025, I gave a talk at SteelCon on the topic "Hacking Stripe Integrations to Bypass E-Commerce Payments". In that talk, I discussed about various security issues I found in the Prestashop and Magento integrations of Stripe.

After I gave the talk, I got even more excited to dig deeper into the integrations to see if I can find something even better. As a result, here I come with this blog post.


## Introduction

In 2025, I barely did bug bounties. When I did, I was working with Stripe for the majority of my time. 

## Generating HMAC signature

Since we now know that the signature has no value at all, 

## 

With a lot of expectations, I went for my trip to Bali and had an amazing time. When I got back, I got a response from the internal team after an in-depth analysis of the

## Root Cause Analysis

After realizing that the signature was empty only for localhost, I decided to dig into the root cause of why it happened. Tracing through the webhook registration request till the depth of hells, I came across the request being sent to the Stripe API: `POST /v1/whatever_endpoint`. Sending the request with appropriate parameters similar to what gets sent through the SDK, I found that if the host is localhost, the server returns 400 Bad Request.

## Conclusion

