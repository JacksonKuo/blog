---
layout: post
title: "GitHub Defense: Part VII - Zizmor"
date: 2026-6-14
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
How to fix zizmor template injection findings holistically

# Zizmor
Not going to explain zizmor, if ya know ya know. Documentation below:
* [https://docs.zizmor.sh/quickstart/](https://docs.zizmor.sh/quickstart/)
* [https://docs.zizmor.sh/usage/](https://docs.zizmor.sh/usage/)
* [https://docs.zizmor.sh/integrations/](https://docs.zizmor.sh/integrations/)
* [https://github.com/zizmorcore/zizmor](https://github.com/zizmorcore/zizmor)
* [https://github.com/zizmorcore/zizmor/blob/main/docs/audits.md#template-injection](https://github.com/zizmorcore/zizmor/blob/main/docs/audits.md#template-injection)

There are 4 output formats:
1. `plaintext` - cargo-style, easy to read, hard to parse
2. `json` - there seems to be multiple json versions, like `json-v1`. just use `json` which is the latest version 
3. `sarif` - super verbose, hard to read
4. `github` - very minimal, intended for annotations

There are 3 `personas`[^1]:
1. `regular` (default) actionable security findings
2. `pedantic` - code smell, non-actionable
3. `auditor` - everything

# Auto-Fix
How does auto-fix work ([auto-fixing-results](https://docs.zizmor.sh/usage/#auto-fixing-results))? Auto-fix has different modes:
* `--fix` default safe mode
* `--fix=unsafe-only` often correct, but require review
* `--fix=all` both safe and unsafe fixes

It seems like template injection type rules fall under the unsafe type of fix. Running the default `--fix` returns: `No fixes available to apply (3 held back by safe mode). Use --fix=unsafe-only or --fix=all to apply unsafe fixes.`. 

Original
{% raw %}
```yaml
name: zizmor-template-inject

on:
  pull_request_target:
    types: [opened, edited]

jobs:
  vuln-check:
    runs-on: ubuntu-latest
    steps:
      - name: 
        run: |
          #danger
          echo "PR Title: ${{ github.event.pull_request.title }}"
          echo "PR Ref: ${{ github.event.pull_request.head.ref }}"
          #safe
          echo "SHA: ${{ github.sha }}"
```
{% endraw %}

Auto-Fixed

{% raw %}
```yaml
name: zizmor-template-inject

on:
  pull_request_target:
    types: [opened, edited]

jobs:
  vuln-check:
    runs-on: ubuntu-latest
    steps:
      - name: 
        run: |
          # danger
          echo "PR Title: ${GITHUB_EVENT_PULL_REQUEST_TITLE}"
          echo "PR Ref: ${GITHUB_EVENT_PULL_REQUEST_HEAD_REF}"
          
          # safe
          echo "SHA: ${{ github.sha }}"
        env:
          GITHUB_EVENT_PULL_REQUEST_TITLE: ${{ github.event.pull_request.title }}
          GITHUB_EVENT_PULL_REQUEST_HEAD_REF: ${{ github.event.pull_request.head.ref }}
```
{% endraw %}

The output appends an `env` block with an uppercase variable, which is substituted in the `run` block. 

# Rules Filter
Unfortunately zizmor doesn't have a way to only trigger on a specific type of rule. You can feed zizmor a config file that can ignore explicit types of rules. I asked claude to generate a list of rules using the audit rules markdown page: [https://raw.githubusercontent.com/zizmorcore/zizmor/main/docs/audits.md](https://raw.githubusercontent.com/zizmorcore/zizmor/main/docs/audits.md)

<details markdown="1">
<summary>EXPAND ME - List of 39 zizmor rules</summary>

```yaml
rules:
  adhoc-packages:
    disable: true
  anonymous-definition:
    disable: true
  archived-uses:
    disable: true
  artipacked:
    disable: true
  bot-conditions:
    disable: true
  cache-poisoning:
    disable: true
  concurrency-limits:
    disable: true
  dangerous-triggers:
    disable: true
  dependabot-cooldown:
    disable: true
  dependabot-execution:
    disable: true
  excessive-permissions:
    disable: true
  forbidden-uses:
    disable: true
  github-app:
    disable: true
  github-env:
    disable: true
  hardcoded-container-credentials:
    disable: true
  impostor-commit:
    disable: true
  insecure-commands:
    disable: true
  known-vulnerable-actions:
    disable: true
  misfeature:
    disable: true
  obfuscation:
    disable: true
  overprovisioned-secrets:
    disable: true
  ref-confusion:
    disable: true
  ref-version-mismatch:
    disable: true
  secrets-inherit:
    disable: true
  secrets-outside-env:
    disable: true
  self-hosted-runner:
    disable: true
  stale-action-refs:
    disable: true
  superfluous-actions:
    disable: true
  template-injection:
    disable: false
  typosquat-uses:
    disable: true
  undocumented-permissions:
    disable: true
  unpinned-images:
    disable: true
  unpinned-tools:
    disable: true
  unpinned-uses:
    disable: true
  unredacted-secrets:
    disable: true
  unsound-condition:
    disable: true
  unsound-contains:
    disable: true
  unsound-ternary:
    disable: true
  use-trusted-publishing:
    disable: true
```

</details>
<br>

# Commands
* Basic Template Injection
  * `zizmor . --offline --fix=unsafe-only`
* Filtered Template Injection
  * `zizmor . --offline --config ../zizmor-rules/zizmor.yml`
* Filtered Self-Hosted Runners
  * To include self-hosted runners the `--persona` must be enabled to `pedantic`
  * `zizmor . --offline --persona pedantic --config ../zizmor-rules/zizmor.yml`

# Threat Model
`Approval for running fork pull request workflows from contributors`: 
Require approval for all external contributors

`zizmor . --offline --persona pedantic --config ../zizmor-rules/zizmor.yml`


# self-hosted runners

# create PRs

# References
[^1]: [https://docs.zizmor.sh/usage/#using-personas](https://docs.zizmor.sh/usage/#using-personas)