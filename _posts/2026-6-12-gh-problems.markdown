---
layout: post
title: "GitHub: Feature - Gaps and Limitations"
date: 2026-6-12
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Creating a list of missing features, oddities, and frustrations with GitHub. Things that should be a one click button, but instead you need create a whole project plan to make up for the inherent missing functionality.

#### Custom Roles
There's no method to construct a Audit Custom Role with org-level read-only permissions, beause custom roles don't have standard READ / WRITE permissions. Instead GitHub provides a weird mishmash of random verbs.[^1]

For example, there's no way to have a read-only webhooks custom role because there's only `Manage organization webhooks` available

And there's no way to have a read-only org ruleset role because there's only `Manage organization ref update rules and rulesets` available

#### PATs
Only classic PATs have the ability to READ packages. Fine-grained PATs still have no package READ permission

#### Authentication
The following 2FA enforcement is all or nothing, doesn't differentiate between internal and external users.

`Require two-factor authentication for everyone in the jkuo-org organization.`

> Organization members who do not have two-factor authentication enabled will be unable to access resources owned by the jkuo-org organization, but will remain a member of jkuo-org until they update their settings. Outside collaborators who do not have two-factor authentication enabled will be removed from the organization and notified. View organization membership to see which users will be impacted.

Some orgs enforce 2FA through Okta. It would be nice if 2FA enforcement had granularity and could only target outside collaborators. 

#### Org Secrets
As mentioned in this post, [https://jacksonkuo.github.io/blog/2026/05/27/gh-actions-org-secret-rightsizing.html](https://jacksonkuo.github.io/blog/2026/05/27/gh-actions-org-secret-rightsizing.html), there's no REST API endpoint to switch a existing org secret repo access from `all` to `selected` without knowing the the original secret

# References
[^1]: Organization Roles > Org Management > Organization

[^2]: Authentication Security > Two-factor authentication