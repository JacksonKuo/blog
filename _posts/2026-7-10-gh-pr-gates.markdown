---
layout: post
title: "GitHub: PR Check"
date: 2026-8-7
tags: ["github"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement

push workflow.yaml to reach repo
required workflows
status checks

shim -> reusable. requires merge into each repo, not as easily scalable. 

if you want folks to be able to modiify for this own needs, can't use reusable
becomes a real headable. or would have to somehow make it modular

required workflows, github has a check that prevents workflows from triggering other workflows to prevent a infinite loop

if a workflow creates a pr, you need to use a ghapp to circumvent the protection. need to review the workflows that create prs

status checks, but requires upfront infra work


another problem
how to prevent feature branches from access gha secrets. env secrets + deployment branch but has ptr bypass?

# Problems


