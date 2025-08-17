---
layout: post
title: "Chainguard"
date: 2025-8-15
tags: ["devsecops"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's build migrate our base images to Chainguard.



# Services

app-springboot
    `ARG BASE_IMAGE=openjdk:17-jdk-alpine`
app-smokescreen
    `FROM golang:1.23.6-alpine3.21`