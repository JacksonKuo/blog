---
layout: post
title: "GitHub: Actions - [bot] bypass"
date: 2026-8-7
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
How does the bot bypass work again?

# Bot

github.event.pull_request.user.login refers to the author of the PR

if: github.actor == 'dependabot[bot]'

`Here the GitHub context github.actor refers to the last identity that performed an action on the PR.`

modify .github/dependabot.yml

will trigger Dependabot on the fork repository where it will create a new branch and an associated PR:

https://www.synacktiv.com/en/publications/github-actions-exploitation-dependabot

https://thehackernews.com/2026/06/claude-code-github-action-flaw-let-one.html

https://github.com/synacktiv/octoscan

https://labs.boostsecurity.io/articles/weaponizing-dependabot-pwn-request-at-its-finest/