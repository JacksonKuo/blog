---
layout: post
title: "Chainguard Migration - Trivy"
date: 2025-8-18
tags: ["chainguard"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's migrate our base images to Chainguard. This problem can be further broken down into:

Step 1: Repair my local build pipeline
Step 2: Run Trivy

#### Summary
Let's see how many CVEs my old images had before i upgrade.