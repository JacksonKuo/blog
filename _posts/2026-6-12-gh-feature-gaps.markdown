---
layout: post
title: "GitHub: Feature - Gaps and Limitations"
date: 2026-6-12
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Create a list of missing features, oddities, and frustrations with GitHub. Things that should be a one click button, but instead you need create a whole project plan to make up for the inherent missing functionality.

This list will definitely be updated overtime.

#### Custom Roles
There's no method to construct an Audit-type Custom Role with org-level read-only permissions. This is because custom roles don't have standard READ / WRITE permissions. Instead GitHub provides a weird mishmash of random verbs.[^1]

For example, there's no way to have a read-only webhooks custom role because there's only the permission `Manage organization webhooks` available.

And there's no way to have a read-only org ruleset role because there's only the permission `Manage organization ref update rules and rulesets` available.

#### PATs
Only classic PATs have the ability to READ packages. Fine-grained PATs still have no package READ permission.[^2] Fine-grained tokens were introduced Oct, 2022, so it's been close to four years.[^3]

#### Authentication
The following 2FA enforcement is all or nothing, doesn't differentiate between internal and external users.

`Require two-factor authentication for everyone in the jkuo-org organization.`[^4]

> Organization members who do not have two-factor authentication enabled will be unable to access resources owned by the jkuo-org organization, but will remain a member of jkuo-org until they update their settings. Outside collaborators who do not have two-factor authentication enabled will be removed from the organization and notified. View organization membership to see which users will be impacted.

Some organization enforce 2FA through Okta which i don't believe is reflected here. It would be nice if 2FA enforcement had granularity and could be configured to specifically target outside collaborators. Additionally, it would be much easier if the outside collaborators were blocked from accessing resources until 2FA was enabled, instead of removed from the organization.

#### Org Secrets
As mentioned in this post, [https://jacksonkuo.github.io/blog/2026/05/27/gh-actions-org-secret-rightsizing.html](https://jacksonkuo.github.io/blog/2026/05/27/gh-actions-org-secret-rightsizing.html), there's no REST API endpoint to switch a existing org secret repo access from `all` to `selected` without re-providing the original secret.

#### Workflows
Workflow have a infinite recursive protection for the default GITHUB_TOKEN.[^5] This protection does not apply to GHApp Tokens. If a workflow using the default GITHUB_TOKEN creates a new PR, any new workflow that spawns from the PR must be approval. However this interferes with Required Workflows and causes them to fail when calls like `peter-evans/create-pull-request` are made.

#### GHApps
GHApp pull request WRITE permissions are so over scoped and dangerous. Most of the time folks just want to make comments in PRs. But PR WRITE perms can trigger and inject content into workflows. I wish there was a lower scoped comment permission that couldn't trigger anything. 

I also wish there was a way to filter GHApp by permissions both first and third-party without having to call the REST API, alas...

# References
[^1]: Organization Roles > Org Management > Organization

[^2]: [https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries](https://docs.github.com/en/packages/learn-github-packages/about-permissions-for-github-packages#about-scopes-and-permissions-for-package-registries)

[^3]: [https://github.blog/security/application-security/introducing-fine-grained-personal-access-tokens-for-github/](https://github.blog/security/application-security/introducing-fine-grained-personal-access-tokens-for-github/)

[^4]: Authentication Security > Two-factor authentication

[^5]: [https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow#triggering-a-workflow-from-a-workflow](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow#triggering-a-workflow-from-a-workflow)