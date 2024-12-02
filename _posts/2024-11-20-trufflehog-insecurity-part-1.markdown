---
layout: post
title: "Trufflehog Shallow Clone Insecurity - Part I"
date: 2024-11-20
tags: ["vulnerability"]
published: false
---

# Problem Statement

Trufflehog's Github documentation uses `github.event.pull_request.head.ref` for shallow cloning. This pattern looks inherently dangerous. Does this pattern lead to shell injection within the Github Action?

{% raw %}
```
- shell: bash
  run: |
    if [ "${{ github.event_name }}" == "push" ]; then
      echo "depth=$(($(jq length <<< '${{ toJson(github.event.commits) }}') + 2))" >> $GITHUB_ENV
        echo "branch=${{ github.ref_name }}" >> $GITHUB_ENV
      fi
      if [ "${{ github.event_name }}" == "pull_request" ]; then
        echo "depth=$((${{ github.event.pull_request.commits }}+2))" >> $GITHUB_ENV
        echo "branch=${{ github.event.pull_request.head.ref }}" >> $GITHUB_ENV
      fi
```
{% endraw %}

# Shallow Cloning

Shallow cloning allows Trufflehog to checkout only commits relevant to the pull request. Instead of checking out the entire repository, only commits up to a certain depth, i.e. between a base commit and head commit are cloned. For multi-GB repositories shallow cloning is a MUST to keep PR scans fast.[^1] [^2]

# Attack Scenario

The `github.event.pull_request.head.ref` is the branch name and one of the event contexts that are user-controlled:[^3]

* `github.event.issue.title`
* `github.event.issue.body`
* `github.event.pull_request.title`
* `github.event.pull_request.body`
* `github.event.comment.body`
* `github.event.review.body`
* `github.event.pages.*.page_name`
* `github.event.commits.*.message`
* `github.event.head_commit.message`
* `github.event.head_commit.author.email`
* `github.event.head_commit.author.name`
* `github.event.commits.*.author.email`
* `github.event.commits.*.author.name`
* **`github.event.pull_request.head.ref`**
* `github.event.pull_request.head.label`
* `github.event.pull_request.head.repo.default_branch`
* `github.head_ref`

The branch name has character restrictions, which include colon or spaces.

`They cannot have ASCII control characters (i.e. bytes whose values are lower than \040, or \177 DEL), space, tilde ~, caret ^, or colon : anywhere.`[^4]

However, with a little bash magic `zzz";echo${IFS}"hello";#`[^5] and a vulnerable Github Action, the end result is arbitrary bash execution within the workflow.

**Vulnerable Github Action**
{% raw %}
```bash
# trufflehog-shallow-clone/.github/workflows/shallow-clone.yml
name: Shallow Clone

on:
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Run a multi-line script
        run: |
          if [ "${{ github.event_name }}" == "pull_request" ]; then
            echo "depth=$((${{ github.event.pull_request.commits }}+2))" >> $GITHUB_ENV
            echo "branch=${{ github.event.pull_request.head.ref }}" >> $GITHUB_ENV
          fi
```
{% endraw %}
**\*\*Job Output\*\***
```
Run if [ "pull_request" == "pull_request" ]; then
  if [ "pull_request" == "pull_request" ]; then
    echo "depth=$((1+2))" >> $GITHUB_ENV
    echo "branch=zzz";echo${IFS}"hello";#" >> $GITHUB_ENV
  fi
  shell: /usr/bin/bash -e {0}
branch=zzz
hello
```

# Mitigation

Note that using single quotes instead of double quotes does not fix this issue and is still vulnerable to bash injection. Instead store the branch name in a environment variable and then use bash variable expansion: "$zzz" instead of Github Action expressions: "$\{\{ zzz \}\}". 

**Vulnerable Github Action**
{% raw %}
```bash
# trufflehog-shallow-clone/.github/workflows/shallow-clone.yml
name: Shallow Clone

on:
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Run a multi-line script
        env:
          BRANCH: "${{ github.event.pull_request.head.ref }}"
        run: |
          if [ "${{ github.event_name }}" == "pull_request" ]; then
            echo "depth=$((${{ github.event.pull_request.commits }}+2))" >> $GITHUB_ENV
            echo "branch=$BRANCH" >> $GITHUB_ENV
            echo "Branch: $BRANCH"
          fi
```
{% endraw %}
**\*\*Job Output\*\***
```
Run if [ "pull_request" == "pull_request" ]; then
  if [ "pull_request" == "pull_request" ]; then
    echo "depth=$((1+2))" >> $GITHUB_ENV
    echo "branch=$BRANCH" >> $GITHUB_ENV
    echo "Branch: $BRANCH"
  fi
  shell: /usr/bin/bash -e {0}
  env:
    BRANCH: zzz";echo${IFS}"hello";#
Branch: zzz";echo${IFS}"hello";#
```
# Risk Analysis

See Part II for the risk and post-exploitation analysis.

### References:
[^1]: [https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#shallow-cloning](https://github.com/trufflesecurity/trufflehog?tab=readme-ov-file#shallow-cloning)
[^2]: [https://trufflesecurity.com/blog/running-trufflehog-in-travis-ci](https://trufflesecurity.com/blog/running-trufflehog-in-travis-ci)
[^3]: [https://securitylab.github.com/resources/github-actions-untrusted-input/](https://securitylab.github.com/resources/github-actions-untrusted-input/)
[^4]: [https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-check-ref-format.html](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-check-ref-format.html)
[^5]: The `#` symbol acts as the start of a comment in bash 