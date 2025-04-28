---
layout: post
title: "OIDC ID Token Claims"
date: 2025-4-28
tags: ["appsec"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
A few years ago a colleague created a fantastic OIDC Claims security writeup. I have since forgetten the specifics, but I thought it might be fun to try create my own writeup based on public research.

# ID Token Claims

| Claim | Claim Type | Example | Description |
|---|---|---|---|
| aud | audience | 1234987819200.apps.<br>googleusercontent.com | *audience the ID token is for, application client ID* | 
| iss | issuer | https://accounts.google.com | *issuer of the OIDC token* | 
| sub  | subject | 10769150350006<br>150715113082367 | *unique identifier for the user* |
| email | email | jsmith@example.com | *user's email address* | 
| email_verified | email verified | true | *if user's email address has been verified* | 
| iat | issued at | 1353601026 | *JWT issued time* | 
| exp | expiration | 1353604926 | *JWT expiration time*  | 
| hd | hosted domain | example.com | *Google-specific: the domain of the Google Workspace or Cloud user. Must be checked when you want to restrict access to members of certain domains. No hd claim means user does not belong to a Google hosted domain* | 
| nbf | not before | 1632492967 | *JWT invalid before this time* | 
| idp | identity provider | 00ok1u7AsAkrwdZL3z0g3 | *Okta: Okta org ID or the Identity Provider ID using Social Authentication or Inbound SAML. Microsoft Entra: identity provider that authenticated the subject of the token* | 

# Provider Samples

| Provider Name | Subject | Issuer | Audience |
|---|---|
| Google [^1] | 10769150350006<br>150715113082367 | https://accounts.google.com | 1234987819200.<br>apps.googleusercontent.com |
| Github [^2] | repo:octo-org/octo-repo:<br>environment:prod | https://token.actions.<br>githubusercontent.com | https://github.com/octo-org |
| Okta [^3] | 00uid4BxXw6I6TV4m0g3 | https://{yourOktaDomain} | uAaunofWkaDJxukCFeBx |
| Entra [^4] | AAAAAAAAAAAAAAAAAAAAA<br>IkzqFVrSaSaFHy782bbtaQ | https://login.microsoftonline.com<br>/{tenantid}/v2.0 | 6cb04018-a3f5-46a7-b995-940c78f5aef3 | 
| Slack [^5] | U0R7MFMJM | https://slack.com | 25259531569.11152291 | 
| Apple [^6] | XXXX.XXXXX.XXXX | https://appleid.apple.com | Service ID or App ID |

# Security
There's a lot of potential security issues with OIDC, but I'm going to focus on claims. The biggest issue seems to be incorrectly using the `email` claim to identify a user. There are many reasons for not relying on emails for identity:

1. Provider compromise: if any provider that a website allows login for is compromised, the result is account takeover. For example, say `bakacore.com` allows Google, Slack, and Facebook for login. And say, I have only ever logged in using Google. If these providers use the `email` claim for identity, then a compromise of Slack results in a compromise of my `bakacore.com` account. 
2. Email recycle: if the email is ever recycled, e.g. an employee leaves the company and a new employee receives the old email, the new employee could potentially access any other accounts that the previous employee created with the original email
3. Domain recycle: if the domain is ever recycled, the same thing can happen in the email recycle scenario
4. IdP lacking email verification: some identity providers don't initially verify email, hence the `email_verified` claim. 

A much more eloquent written argument was made by Jess Trochet: [https://x86.wtf/identity-what-it-isnt-da25198dfe58](https://x86.wtf/identity-what-it-isnt-da25198dfe58). Who argues that instead of the `email` claim, the provider should use a "unique identifier".

The providers themselves also have warnings stating this very issue:

Google
> Warning: Don't use the `email` field as a unique identifier for a user. Always use the sub field.

> Warning: When implementing your account management system, you shouldn't use the email field in the ID token as a unique identifier for a user. Always use the `sub` field as it is unique to a Google Account even if the user changes their email address.

Microsoft
> Never use claims like email, preferred_username or unique_name to store or determine whether the user in an access token should have access to data. These claims aren't unique and can be controllable by tenant administrators or sometimes users, which make them unsuitable for authorization decisions.

It sounds like the `sub` claims is the ideal unique identifier for determining identity. There are some issues though with using the `sub` claim. Individual providers may guarantee that their `sub` identifier is unique, however they make no such claims that the `sub` identifier is unique between different providers. Provider A and Provider B many inadvertately use the same `sub` value. To account for this, applications should use a combination of `sub` + IdP to identify a user. 

Additionally, Trufflehog put out a blog post which went in-depth into the domain recycle scenario: [https://trufflesecurity.com/blog/millions-at-risk-due-to-google-s-oauth-flaw](https://trufflesecurity.com/blog/millions-at-risk-due-to-google-s-oauth-flaw). But interestingly they argue that the `sub` identifier does not solve this, because despite the Google documentation stating that `sub` never changes[^7]:

> According to a staff engineer at a major tech company: “The sub claim changes in about 0.04% of logins from Log in with Google. For us, that's hundreds of users last week”.

There's no further explanation for why this occurs. The crux of this issue seems to be what is causing these `sub` changes. If the `sub` changes when the account is destroyed, for example an employee leaves and a new employee takes over the old email, having the `sub` change in these situations seems 100% correct. The researcher further states that:

> Because the sub claim is inconsistent, it cannot be used to uniquely identify users - leaving services reliant on the email and hd claims.

Using `sub` as a unique identifer as far as I'm aware is the prevailing wisdom. I'm hesitant to leave this guidance until I have a better understanding on when and why `sub` is changing. 

Google `said they were working on a fix`, but there are no further details. Perhaps Google is working on making `sub` more consistent. Side note there's a lively and healthy discussion about this issue on Hacker News[^8], where someone also pointed out the `hd` field can't be trusted either.

# References
[^1]: [https://developers.google.com/identity/openid-connect/openid-connect#an-id-tokens-payload](https://developers.google.com/identity/openid-connect/openid-connect#an-id-tokens-payload)

[^2]: [https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

[^3]: [https://developer.okta.com/docs/api/openapi/okta-oauth/guides/overview/#id-token](https://developer.okta.com/docs/api/openapi/okta-oauth/guides/overview/#id-token)

[^4]: [https://learn.microsoft.com/en-us/entra/identity-platform/id-token-claims-reference](https://learn.microsoft.com/en-us/entra/identity-platform/id-token-claims-reference), [https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens](https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens), [https://learn.microsoft.com/en-us/entra/identity-platform/claims-validation](https://learn.microsoft.com/en-us/entra/identity-platform/claims-validation), [https://learn.microsoft.com/en-us/entra/identity-platform/id-tokens#sample-v20-id-token](https://learn.microsoft.com/en-us/entra/identity-platform/id-tokens#sample-v20-id-token) (jwt.ms link)

[^5]: [https://api.slack.com/methods/openid.connect.token](https://api.slack.com/methods/openid.connect.token)

[^6]: [https://developer.okta.com/blog/2019/06/04/what-the-heck-is-sign-in-with-apple](https://developer.okta.com/blog/2019/06/04/what-the-heck-is-sign-in-with-apple), [https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple](https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple)

[^7]: > `sub`: an identifier for the user, unique among all Google accounts and never reused. A Google account can have multiple email addresses at different points in time, but the sub value is never changed. Use sub within your application as the unique-identifier key for the user. Maximum length of 255 case-sensitive ASCII characters.

[^8]: [https://news.ycombinator.com/item?id=42699099](https://news.ycombinator.com/item?id=42699099)

