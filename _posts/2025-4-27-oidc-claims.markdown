---
layout: post
title: "OIDC ID Token Claims"
date: 2025-4-25
tags: ["appsec"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement

A few years ago a colleague created a fantastic OIDC Claims security writeup. I have since forgetten the specifics, but I thought it might be fun to try create my own writeup based on public research.

# OIDC ID Token Claims

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
| idp | identity provider | 00ok1u7AsAkrwdZL3z0g3 | *Okta-specific: Okta org ID or the Identity Provider ID using Social Authentication or Inbound SAML* | 

# Provider Sample Values

| Provider Name | Subject | Issuer | Audience |
|---|---|
| Google [^1] | 10769150350006<br>150715113082367 | https://accounts.google.com | 1234987819200.<br>apps.googleusercontent.com |
| Github [^2] | repo:octo-org/octo-repo:<br>environment:prod | https://token.actions.<br>githubusercontent.com | https://github.com/octo-org |
| Okta [^3] | 00uid4BxXw6I6TV4m0g3 | https://{yourOktaDomain} | uAaunofWkaDJxukCFeBx |
| Entra [^4] | AAAAAAAAAAAAAAAAAAAAA<br>IkzqFVrSaSaFHy782bbtaQ | https://login.microsoftonline.com<br>/{tenantid}/v2.0 | 6cb04018-a3f5-46a7-b995-940c78f5aef3 | 
| Slack [^5] | U0R7MFMJM | https://slack.com | 25259531569.11152291 | 
| Apple [^6] | XXXX.XXXXX.XXXX | https://appleid.apple.com | Service ID or App ID |

# Email Claim



[https://x86.wtf/identity-what-it-isnt-da25198dfe58](https://x86.wtf/identity-what-it-isnt-da25198dfe58)
[https://trufflesecurity.com/blog/millions-at-risk-due-to-google-s-oauth-flaw](https://trufflesecurity.com/blog/millions-at-risk-due-to-google-s-oauth-flaw)

> 	The subject of the information in the token. For example, the user of an app. This value is immutable and can't be reassigned or reused. The subject is a pairwise identifier and is unique to an application ID. If a single user signs into two different apps using two different client IDs, those apps receive two different values for the subject claim. You may or may not want two values depending on your architecture and privacy requirements.


Google
> Warning: Don't use the email field as a unique identifier for a user. Always use the sub field.

> Warning: When implementing your account management system, you shouldn't use the email field in the ID token as a unique identifier for a user. Always use the sub field as it is unique to a Google Account even if the user changes their email address.

Microsoft
> Never use claims like email, preferred_username or unique_name to store or determine whether the user in an access token should have access to data. These claims aren't unique and can be controllable by tenant administrators or sometimes users, which make them unsuitable for authorization decisions.

# Problems

Provider compromised
Domain purchase
Cross provider compromise
Email verification

Facebook
Gmail
Yahoo
Github



# References
[^1]: [https://developers.google.com/identity/openid-connect/openid-connect#an-id-tokens-payload](https://developers.google.com/identity/openid-connect/openid-connect#an-id-tokens-payload)

[^2]: [https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

[^3]: [https://developer.okta.com/docs/api/openapi/okta-oauth/guides/overview/#id-token](https://developer.okta.com/docs/api/openapi/okta-oauth/guides/overview/#id-token)

[^4]: [https://learn.microsoft.com/en-us/entra/identity-platform/id-token-claims-reference](https://learn.microsoft.com/en-us/entra/identity-platform/id-token-claims-reference), [https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens](https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens), [https://learn.microsoft.com/en-us/entra/identity-platform/claims-validation](https://learn.microsoft.com/en-us/entra/identity-platform/claims-validation), [https://learn.microsoft.com/en-us/entra/identity-platform/id-tokens#sample-v20-id-token](https://learn.microsoft.com/en-us/entra/identity-platform/id-tokens#sample-v20-id-token) (jwt.ms link)

[^5]: [https://api.slack.com/methods/openid.connect.token](https://api.slack.com/methods/openid.connect.token)

[^6]: [https://developer.okta.com/blog/2019/06/04/what-the-heck-is-sign-in-with-apple](https://developer.okta.com/blog/2019/06/04/what-the-heck-is-sign-in-with-apple), [https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple](https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple)





