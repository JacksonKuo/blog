---
layout: post
title: "Ephemeral GitHub Tokens: Octo-STS"
date: 2025-8-10
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
I've been testing Octo-STS by Chainguard which generates ephemeral GitHub tokens that can replace long-lived PAT tokens: [https://www.chainguard.dev/unchained/the-end-of-github-pats-you-cant-leak-what-you-dont-have](https://www.chainguard.dev/unchained/the-end-of-github-pats-you-cant-leak-what-you-dont-have). While implementing and testing Octo-STS, I ran into a couple of pitfalls, and will document how I resolved those issues and any lessons learned.

# Two-Modes
Something that wasn't immediately clear to me from the documentation[^1] [^2] [^3] [^4] was that Octo-STS has two different modes. The first mode will mint a ephemeral token for accessing a single repository, while the second mode will mint a ephemeral token for accessing multiple repositories and/or grant organization permissions. 

#### Mode 1 - Granting single repo access
It's important to note that for single repo access the policy file is housed in the repo that is granting access `<repo>/.github/chainguard/bar.sts.yaml` file. 

```yaml
#<org>/repo-a/.github/chainguard/bar.sts.yaml
issuer: https://token.actions.githubusercontent.com

subject_pattern: repo:<org>/repo-b:ref:refs/heads/main

permissions:
  metadata: read
  contents: read
```

{% raw %}
```yaml
#<org>/repo-b/.github/workflows/pull-action.yaml
name: Pull Action

on:
  workflow_dispatch

permissions:
  id-token: write 

steps:
- uses: octo-sts/action@6177b4481c00308b3839969c3eca88c96a91775f # v1.0.0
  id: octo-sts
  with:
    scope: <org>/repo-a
    identity: bar

- name: Checkout internal repo
  uses: actions/checkout@4
  with:
    repository: <org>/repo-a
    token: "${{ steps.octo-sts.outputs.token }}"
    path: .
```
{% endraw %}

#### Mode 2 - Granting multi-repo access and/or organization level access
For the multi-repo access the policy file is housed in the organization `.github` repo `<org>/.github/.github/chainguard/foo.sts.yaml` file. Note this policy file has the additional `repositories` field which can allow all repos via `[]` or specify repos `- <repo>`.

```yaml
# <org>/.github/.github/chainguard/foo.sts.yaml
issuer: https://token.actions.githubusercontent.com
subject: repo:<org>/.*:ref:refs/heads/main

permissions:
  metadata: read

repositories: []
```

Importantly the `scope` when requesting an multi-repo/org must be only the `<org>`. The scope should not have the additional repo, i.e. `<org>/.github`. 

{% raw %}
```yaml
# <org>/repo-a/workflows/use-action.yaml
name: Use Action

on:
  workflow_dispatch

permissions:
  id-token: write 

steps:
- uses: octo-sts/action@6177b4481c00308b3839969c3eca88c96a91775f # v1.0.0
  id: octo-sts
  with:
    scope: org
    identity: foo

- name: Use the token with gh
  env:
    GITHUB_TOKEN: "${{ steps.octo-sts.outputs.token }}"
  run: |
    gh repos list
```
{% endraw %}

# Additional Insights
The GitHub App is controlled by Chainguard and hosted at [https://octo-sts.dev](https://octo-sts.dev). There was a SSRF vulnerability found this year: [CVE-2025-52477](https://github.com/octo-sts/app/security/advisories/GHSA-h3qp-hwvr-9xcq). The organization `.github` repository does not have to be public, hence the policy file does not have to be made public. 

# References
[^1]: [https://www.chainguard.dev/unchained/the-end-of-github-pats-you-cant-leak-what-you-dont-have](https://www.chainguard.dev/unchained/the-end-of-github-pats-you-cant-leak-what-you-dont-have)
[^2]: [https://github.com/apps/octo-sts](https://github.com/apps/octo-sts)
[^3]: [https://github.com/octo-sts/app](https://github.com/octo-sts/app)
[^4]: [https://www.chainguard.dev/unchained/open-sourcing-octo-sts](https://www.chainguard.dev/unchained/open-sourcing-octo-sts)

