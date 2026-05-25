---
layout: post
title: "GitHub - GHApp Permissions"
date: 2026-9-16
tags: ["advice"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement

why pr review + repo read is scary

edit pr title + target workflows

on:
  pull_request:
    types: [edited]

github.event.pull_request.head.ref​ 