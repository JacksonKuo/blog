---
layout: post
title: "Tradeoffs - Dependabot × Renovate"
date: 2025-9-2
tags: ["github"]
published: ture
---

**Contents**
* TOC
{:toc}

# Problem Statement
What are the trade offs with dependabot vs renovate? There's been a lot of comparison breakdown already written: 

* [https://docs.renovatebot.com/bot-comparison/](https://docs.renovatebot.com/bot-comparison/)
* [https://www.jvt.me/posts/2024/04/12/use-renovate/](https://www.jvt.me/posts/2024/04/12/use-renovate/)
* [https://blog.pullnotifier.com/blog/dependabot-vs-renovate-dependency-update-tools](https://blog.pullnotifier.com/blog/dependabot-vs-renovate-dependency-update-tools)

Some of these breakdowns are already out-of-date due to GitHub adding a lot of functionality to GitHub Advanced Security (GHAS) in the last couple of years. 

# Grouped PRs
For the longest time the most important differentiator had been Dependabot not supporting grouped PRs. And Dependabot has a reputation for creating a unmanageably large amount of PRs. As a result, developers have typically gravitated towards Renovate for dependency management, due to its support for grouped PRs. However GitHub has added "recently" added `groups` and `multi-ecosystem-groups`.

* GitHub
    * 2017 Apr - Dependabot created by Grey Baker
    * 2019 Jun - [Dependabot acquired by GitHub](https://www.indiehackers.com/product/dependabot)
    * 2023 Jun - [Grouped version updates - Beta](https://github.blog/changelog/2023-06-29-grouped-version-updates-for-dependabot-public-beta/)
    * 2023 Aug - [Grouped version updates - GA](https://github.blog/changelog/2023-08-24-grouped-version-updates-for-dependabot-are-generally-available/)
* Renovate
    * 2017 Jan - Renovate released by Rhys Arkins[^1]
    * 2019 Nov - Renovate joins Mend


#### GitHub 
I updated my `dependabot.yml` file with `groups`:[^2]

```yaml
version: 2
updates:
  - package-ecosystem: "gradle" 
    directory: "/" 
    schedule:
      interval: "daily"
    groups:
      springboot:
        applies-to: version-updates
        patterns:
          - "com.google.code.gson:gson"
          - "org.redisson:redisson"
          - "com.twilio.sdk:twilio"
          - "com.googlecode.libphonenumber:libphonenumber"
        update-types:
          - "minor"
          - "patch"
```

Note that if PR hasn't been merged for 30 days, Dependabot will stop rebasing.[^3] 

#### Renovate
Renovate has supported package groups for a while now, though I'm not sure exactly when. Renovate also provides presets-group: [https://docs.renovatebot.com/presets-group/](https://docs.renovatebot.com/presets-group/). I updated my `renovate.json` with a package group:

```yaml
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended"
  ],
  "packageRules": [
    {
      "groupName": "all non-major dependencies",
      "groupSlug": "all-minor-patch",
      "matchPackageNames": [
        "*"
      ],
      "matchUpdateTypes": [
        "minor",
        "patch"
      ]
    }
  ]
}
```

#### Results
I updated both Dependabot and Renovate with groups. Renovate updated instantly. Dependabot is set to `interval: "daily"`, so I'll wait and see what's what tomorrow.

Actually, the Dependabot Graph can be triggered visa Insights > Dependency Graph > Dependabot > Check for Updates. Not super sure if that causes Dependabot to close old PRs. There might be a separate backend process that handles that every hour or so, if I had to guess, but looks pretty good...

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/grouped-pr.png){: width="600"}
{: refdef}

# References
[^1]: [https://git.shivering-isles.com/github-mirror/renovatebot/renovate/-/tree/28.20.0](https://git.shivering-isles.com/github-mirror/renovatebot/renovate/-/tree/28.20.0)

[^2]: [https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/optimizing-pr-creation-version-updates](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/optimizing-pr-creation-version-updates)

[^3]: [https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates#changing-the-rebase-strategy-for-dependabot-pull-requests](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates#changing-the-rebase-strategy-for-dependabot-pull-requests)
