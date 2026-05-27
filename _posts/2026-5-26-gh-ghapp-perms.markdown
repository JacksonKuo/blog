---
layout: post
title: "GitHub - GHApp Permissions Cheatsheet"
date: 2026-5-26
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Which GHApp permissions do you need to watch out for?

# Permissions
There are 4 types of permissions: repo, org, account, enterprise. And then GHApps can be installed with `All repositories` or `Only selected repositories`. Typical a GHApp doesn't need org-wide access to all repos.

| Type | Permission | R/W | Risk |
|---||---|---|---|---|
| Repo | Contents | READ | Secrets in source code or commit history | 
| Repo | Variables | READ | May hold secrets which are viewable in plaintext |
| Repo | Actions | READ | Gain access to workflow artifacts (90 days default) | 
| Repo | Secret scanning alerts | READ | Scanned secrets are readable in plaintext |
| Repo | Pull requests | WRITE | This one is more dangerous than i initially thought. You can't directly change code. But you can edit pull requests, which can retrigger workflows <br><br>on:<br>&nbsp;&nbsp;pull_request:<br>&nbsp;&nbsp;&nbsp;&nbsp;types: [edited]<br><br>And all you need to do is find any exploitable internal GH workflow, i.e. `github.event.pull_request.head.ref​` and template inject to gain access to RCE inside the runner |

I'll continue to expand this cheatsheet over time. 