---
layout: post
title: "Application Defense: Part VI - Twilio MFA"
date: 2025-2-9
tags: ["app"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Build a bunch of application defensive mechanisms. This problem can be further broken down into:

* Step 1: Build a sample app with Spring Boot
* Step 2: Build a deployment pipeline
* Step 3: Add hCaptcha
* Step 4: Upgrade deployment pipeline
* Step 5: Add custom rate limit
* Step 6: **Add Twilio MFA**

# Twilio MFA
The next application defense to build is multi-factor authentication (MFA) using Twilio. 

* Twilio documentation: 
    * [https://www.twilio.com/docs/verify/api](https://www.twilio.com/docs/verify/api)
* MFA endpoint: 
    * [https://bakacore.com:8087/mfa](https://bakacore.com:8087/mfa)
    * [https://bakacore.com:8087/mfa/verify](https://bakacore.com:8087/mfa/verify)
* MfaController: 
    * [https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/controller/MfaController.java](https://github.com/JacksonKuo/app-springboot/blob/main/src/main/java/com/jkuo/sample/controller/MfaController.java)

The implementation is very basic. I'm using a trial account, and Verify SMS can only be sent to max of 5 verified phone numbers. I have a RestController with a `/mfa` endpoint that redirects to a `/mfa/verify` endpoint and I'm using the Twilio Java SDK.[^1] The Twilio verification requires three secrets, which can be found at:
* Account dashboard - [https://console.twilio.com/](https://console.twilio.com/)
    * `TWILIO_ACCOUNT_SID`
    * `TWILIO_AUTH_TOKEN`
* Verify > Services - [https://console.twilio.com/us1/develop/verify/services](https://console.twilio.com/us1/develop/verify/services)
    * `VERIFY_SERVICE_SID`

#### Pricing
The pricing is pretty interesting, I didn't realize how expensive Verify is:[^2]

* SMS: *$0.05 per successful verification + $0.0079 per SMS (US)*
* Email: *$0.05 per successful verification*

For an application that has millions of users, like Twitter, Verify could be cost prohibitive. You would need to get a better flat rate, or would be better off building your own MFA system.[^3]

On the plus side, Twilio only charges for successful verification, which should help against SMS pumping, i would think..

#### Oddities
Twilio's blog has bad `curl` examples: [https://www.twilio.com/en-us/blog/implement-two-factor-authentication-2fa-30-seconds-twilio-verify](https://www.twilio.com/en-us/blog/implement-two-factor-authentication-2fa-30-seconds-twilio-verify)

```
curl -X POST https://verify.twilio.com/v2/Services/$VERIFY_SERVICE_SID/Verifications --data-urlencode "To=<your-phone-number>" --data-url "Channel=sms" -u $TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN
```

The second `--data-url` should be `--data-urlencode`

The Twilio login is a bit odd. I signed up for the trial account via Google, but there doesn't seem to be a way to resign-in using Google. The Twilio login only presents username and password. You have to go through the "Sign up via Google" again to login via SSO. 

#### Future Improvements
* Required a successful username and password first, to prevent unauthenticated SMS pumping. 
* Automatically pass phone number through to `/mfa/verify`. Could use Spring Boot flash attributes, which generated a temporary unauthenticated `JSESSIONID` to hold the values. Would need to move from `@RestController` to `@Controller` and `return "redirect:/mfa/verify";`
* Add to a proper login flow

# References
[^1]: [https://github.com/twilio/twilio-java](https://github.com/twilio/twilio-java)

[^2]: [https://www.twilio.com/en-us/verify/pricing](https://www.twilio.com/en-us/verify/pricing)

[^3]: [https://medium.com/@dwilkie_34546/build-your-own-phone-verification-service-187cc2de76ba](https://medium.com/@dwilkie_34546/build-your-own-phone-verification-service-187cc2de76ba)