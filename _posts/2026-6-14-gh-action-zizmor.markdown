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

# Template Injection Remediation

#### Auto-Fix
How does auto-fix work ([auto-fixing-results](https://docs.zizmor.sh/usage/#auto-fixing-results))? Auto-fix has different modes:
* `--fix` default safe mode
* `--fix=unsafe-only` often correct, but require review
* `--fix=all` both safe and unsafe fixes

It seems like template injection type rules fall under the unsafe fix type. Running the default `--fix` on a injectable workflow returns: `No fixes available to apply (3 held back by safe mode). Use --fix=unsafe-only or --fix=all to apply unsafe fixes`. 

Example - Original
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

Example - Auto-Fixed
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

#### Rules Filter
Unfortunately zizmor doesn't have a way to only trigger on a specific type of rule. A workaround is you can feed zizmor a config file that can ignore explicit types of rules. I asked Claude to generate a list of rules using the audit rules markdown page: [https://raw.githubusercontent.com/zizmorcore/zizmor/main/docs/audits.md](https://raw.githubusercontent.com/zizmorcore/zizmor/main/docs/audits.md)

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

#### JSON Format Structure
Let's look at the output values: `zizmor . --offline --format json | jq '.[2]'`

* `ident`
* `desc` (unneeded cruft)
* `url` (unneeded cruft)
* `determination`
  * `confidence`
  * `severity`
  * `persona`
* `locations[]`
  * `symbolic` - paths
  * `concrete`
    * `location` - line numbers
    * `feature` - source code
* `ignored`

<details markdown="1">
<summary>EXPAND ME - JSON Format Output</summary>
```json
{
  "ident": "dangerous-triggers",
  "desc": "use of fundamentally insecure workflow trigger",
  "url": "https://docs.zizmor.sh/audits/#dangerous-triggers",
  "determinations": {
    "confidence": "Medium",
    "severity": "High",
    "persona": "Regular"
  },
  "locations": [
    {
      "symbolic": {
        "key": {
          "Local": {
            "prefix": ".",
            "given_path": "./.github/workflows/rule-template-inject.yml"
          }
        },
        "annotation": "pull_request_target is almost always used insecurely",
        "route": {
          "route": [
            {
              "Key": "on"
            }
          ]
        },
        "feature_kind": "Normal",
        "kind": "Primary"
      },
      "concrete": {
        "location": {
          "start_point": {
            "row": 2,
            "column": 0
          },
          "end_point": {
            "row": 4,
            "column": 27
          },
          "offset_span": {
            "start": 30,
            "end": 84
          }
        },
        "feature": "on:\n  pull_request_target:\n    types: [opened, edited]",
        "comments": []
      }
    }
  ],
  "ignored": false
}
```
</details>
<br>

#### Severity and Confidence
Breakdown of how zizmor template injection severity and confidence values and what actually needs fixing

Just a reminder `workflow_dispatch` is a manual trigger. `workflow_call` is for reusable workflows.[^2] Both accept either `github.event.inputs.version` or `inputs.*`[^3]

| Expression | Severity | Confidence | Public Exploit | Desc |
|---|---|---|---|
| `github.event.inputs.version` | High | High | Taint | `workflow_dispatch` No<br>`workflow_call` Maybe | 
| `inputs.*` | High | High | Taint | `workflow_dispatch` No<br>`workflow_call` Maybe | 
| `github.head_ref` | High | High | Yes | - | 
| `github.actor` | High | High | No | GH username alphanumeric + hyphen[^4] | 
| `github.event` | High | High | Yes | needs testing | 
| `github.ref` | High | High | No | - | 
| `github.ref_name` | High | High | No | - | 
| `env.*` | Low | High | No | can't set via PR |  
| `matrix.*` | Med | Med | Unk | unknown |
| `steps.*.outputs.*` | Info | Low | Taint | - |
| `steps.outputs.outcome` | Info | Low | Taint | - |
| `needs.*`  | Info | Low | No | - |
| `job.status`  | Info | Low | No | - |

Seems like zizmor is a little over zealous on the high confidence high severity. It's probably best to do a second level of filter PR user-controlled values[^5] + `github.event`.

# Commands
* Basic Template Injection
  * `zizmor . --offline --fix=unsafe-only`
* Filter Template Injection
  * `zizmor . --offline --config ../zizmor-rules/zizmor.yml`
* Filter Self-Hosted Runners
  * To include self-hosted runners the `--persona` must be enabled to `pedantic`
  * Is `--persona pedantic` the same as --pedantic?
    > This persona can also be enabled with --pedantic, which is an alias for --persona=pedantic.
  * `zizmor . --offline --persona pedantic --config ../zizmor-rules/zizmor.yml`
* Filter High Sev / High confidence
  * `zizmor . --offline --persona pedantic --config ../zizmor-rules/zizmor.yml --min-severity=High --min-confidence=High` 

# Threat Model

Fork / non-fork

* Non-forks
  * Pwn request + self-hosted runners = RCE
  * Pwn request + pull_request_target = credential theft
  * Pwn request + pull_request = GITHUB_TOKEN
* Forks
  * Pwn request + self-hosted runners = RCE
  * Pwn request + pull_request_target = credential theft
  * Pwn request + pull_request = GITHUB_TOKEN

`Approval for running fork pull request workflows from contributors`: 
Require approval for all external contributors





# create PRs

# References
[^1]: [https://docs.zizmor.sh/usage/#using-personas](https://docs.zizmor.sh/usage/#using-personas)

[^2]: [https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows)

[^3]: [https://github.blog/changelog/2022-06-09-github-actions-inputs-unified-across-manual-and-reusable-workflows/](https://github.blog/changelog/2022-06-09-github-actions-inputs-unified-across-manual-and-reusable-workflows/)

[^4]: [https://github.com/shinnn/github-username-regex](https://github.com/shinnn/github-username-regex)

[^5]: [https://jacksonkuo.github.io/blog/2024/12/18/gh-action-exploitation-part-1.html#attack-scenario](https://jacksonkuo.github.io/blog/2024/12/18/gh-action-exploitation-part-1.html#attack-scenario)