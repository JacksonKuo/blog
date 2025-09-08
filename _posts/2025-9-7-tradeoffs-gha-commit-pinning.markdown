---
layout: post
title: "Tradeoffs - GHA Commit Pinning"
date: 2025-9-7
tags: ["github"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
What are the trade offs with following tools for GitHub Action auto-commit pinning?

* stacklok/frizbee
* suzuki-shunuske/pinact
* sethvargo/ratchet
* Dependabot
* Renovate

# Definitions
There's many terms for pinning and there are two types of pinning:
* Type 1, e.g. `actions/checkout@ff7abcd0c3c05ccf6adc123a8cd1fd4fb30fb493` also known as:
    * commit pinning
    * SHA pinning
    * hash pinning
    * digests
* Type 2, e.g. `5.0.0` also known as:
    * version pinning
    * tag pinning
    * semantic version pinning

# Tools
There are many more tools available than just these three, but these first three I'd posit are the most popular of the boutique tools:

#### [stacklok/frizbee](https://github.com/stacklok/frizbee)
Pretty standard. Converts checksums to tag. 
* GitHub Actions
* Container Images

#### [sethvargo/ratchet](https://github.com/sethvargo/ratchet)
Supports many more platforms beyond GHA. Has a list of [known issues](https://github.com/sethvargo/ratchet?tab=readme-ov-file#known-issues).
* Circle CI
* GitHub Actions
* GitLab CI
* Google Cloud Build
* Harness Drone
* Tekton

#### [suzuki-shunuske/pinact](https://github.com/suzuki-shunsuke/pinact)
Only supports SemVer. 
* GitHub Actions

And the README also provides a nice list of cases where you might not want to use Renovate's `helpers:pinGitHubActionDigestsToSemver` preset[^1]:
> 1. Renovate can't pin actions in pull requests before merging them. If you use linters such as ghalint in CI, you need to pin actions before merging pull requests (ref. ghalint policy to enforce actions to be pinned)
> 2. Even if you use Renovate, sometimes you would want to update actions manually
> 3. pinact is useful for non Renovate users
> 4. pinact supports verifying version annotations

#### Dependabot

Generally we want to lean on the more established tools like Dependabot/Renovate. Dependabot will doing action commit pinning via `package-ecosystem: "github-actions"`.[^2] 

```yaml
  - package-ecosystem: "github-actions" 
    directory: "/" 
    schedule:
      interval: "weekly"
```

However there are some caveats. It's also important to note that Dependabot version updates and security alert are different systems. 

Dependabot alerts only trigger on semantic versioning and not SHA versioning.[^3] [^4]

> Dependabot will only create Dependabot alerts for vulnerable GitHub Actions that use semantic versioning. You will not receive alerts for a vulnerable action that uses SHA versioning. If you use GitHub Actions with SHA versioning, we recommend enabling Dependabot version updates for your repository or organization to keep the actions you use updated to the latest versions.

Next, I downgraded my `checkout` action from the latest `5.0.0` to `4.3.0` using only the SHA and no version in the comment.[^5]

```yaml
    - name: Checkout src
      uses: actions/checkout@08eba0b27e820071cde6df949e0beb9ba4906955
```

Note that Dependabot will follow whatever format you've chosen. If you're using tags, Dependabot will provide a PR with updated tags. If you provide a SHA, Dependabot will provide a SHA. If you're using SHA you do not have to provide a SemVer comment. But remember SHA pinning will not include Security Alerts because Security Alerts require semantic versioning. So if you're using SHA pinning, you'll have to rely on version updates to get the latest versions. 

My `package-ecosystem: "github-actions"` is set to weekly, so in order to manually trigger Dependabot I had to update via `Insights > Dependency graph > Dependabot > Recent update jobs > Check for updates`, which is equivalent. This created the following PRs:

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/hashpin-both.png){: width="600"}
{: refdef}

GitHub is previewing immutable actions - [https://github.com/features/preview/immutable-actions](https://github.com/features/preview/immutable-actions), which means:

> migrate your Actions to GitHub Packages as an OCI artifact in the GitHub Container Registry, with immutable package versions, semantic versioning, and build provenance.

#### Renovate
Renovate updated instantly. The renovate PR doesn't show the underlying tags unfortunately. 

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/hashpin-renovate.png){: width="600"}
{: refdef}

My `renovate.json` uses

```json
  "extends": [
    "config:recommended"
  ]
```

which includes[^6]

```json
{
  "extends": [
    ":dependencyDashboard",
    ":semanticPrefixFixDepsChoreOthers",
    ":ignoreModulesAndTests",
    "group:monorepos",
    "group:recommended",
    "mergeConfidence:age-confidence-badges",
    "replacements:all",
    "workarounds:all"
  ]
}
```
By default, Renovate will create a PR with updated Actions hash. Renovate follows the same update rules as Dependabot. If the format is tags, Renovate will update using tags. If the format is hashes Renovate will update with hashes. 

```yaml
    uses: actions/checkout@08eba0b27e820071cde6df949e0beb9ba4906955
    uses: actions/checkout@ff7abcd0c3c05ccf6adc123a8cd1fd4fb30fb493
```

Renovate has two preset helpers: `pinGitHubActionDigests`[^7] and `helpers:pinGitHubActionDigestsToSemver`. What these helpers do is cause  all actions in the PR to use digests. Without these helpers, actions with tags are still allowed. The helper belows creates a PR called `Pin dependencies`.

```json
{
  "packageRules": [
    {
      "matchDepTypes": [
        "action"
      ],
      "pinDigests": true
    }
  ]
}
```
{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/hashpin-renovate-digest-pr.png){: width="600"}
{: refdef}
{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/hashpin-renovate-digest-commit.png){: width="600"}
{: refdef}

The SemVer version will add the SemVer to the PR instead of the SHA for readability. This helpers looks to only proc on tags. 

```json
{
  "packageRules": [
    {
      "extends": [
        "helpers:pinGitHubActionDigests"
      ],
      "extractVersion": "^(?<version>v?\\d+\\.\\d+\\.\\d+)$",
      "versioning": "regex:^v?(?<major>\\d+)(\\.(?<minor>\\d+)\\.(?<patch>\\d+))?$"
    }
  ]
}
```

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/hashpin-renovate-digestsemver-pr.png){: width="600"}
{: refdef}
{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/hashpin-renovate-digestsemver-commit.png){: width="600"}
{: refdef}

For some reason in the Mend dashboard the version shows up as `undefined`: `actions/checkout undefined→ [Updates: ]`. Ahh if it's in SHA form, the dashboard shows as `undefined`. In tag form, `actions/checkout 4.3.0→ [Updates: 5.0.0]` the version shows up. 

# Upsize Droplet
I finally bit the bullet and upgraded to 2GB of RAM for $12. My droplet was having trouble redeploying my services. Looks like I was running out of RAM. One day I'll refactor and get back down to 1GB by adding feature flags and turning off services I don't need running, sigh...

```
               total        used        free      shared  buff/cache   available
Mem:            2062         956         388           4         874        1105
Swap:              0           0           0
```

# References
[^1]: [https://github.com/suzuki-shunsuke/pinact?tab=readme-ov-file#why-not-using-renovates-helperspingithubactiondigeststosemver-preset](https://github.com/suzuki-shunsuke/pinact?tab=readme-ov-file#why-not-using-renovates-helperspingithubactiondigeststosemver-preset)

[^2]: [https://github.com/dependabot](https://github.com/dependabot)

[^3]: [https://developerwithacat.com/blog/202508/github-actions-commit-hash-pinning-tradeoffs/](https://developerwithacat.com/blog/202508/github-actions-commit-hash-pinning-tradeoffs/)

[^4]: [https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts#detection-of-insecure-dependencies](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts#detection-of-insecure-dependencies)

[^5]: [https://github.com/actions/checkout/commit/08eba0b27e820071cde6df949e0beb9ba4906955](https://github.com/actions/checkout/commit/08eba0b27e820071cde6df949e0beb9ba4906955)

[^6]: [https://docs.renovatebot.com/presets-config/#configrecommended](https://docs.renovatebot.com/presets-config/#configrecommended)

[^7]: [https://docs.renovatebot.com/presets-helpers/#helperspingithubactiondigests](https://docs.renovatebot.com/presets-helpers/#helperspingithubactiondigests)