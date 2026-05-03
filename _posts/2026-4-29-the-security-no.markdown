---
layout: post
title: "Security Approvals"
date: 2026-5-2
tags: ["advice"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
When is it okay to say yes and when is it not? Often security education tends to focus on the technical skills, exploits and vulnerabilties. 

But for me, the highest difficulty and stress of security tends to happen at the approval process. Where security gives their blessing for a developer to proceed on some work of unknown risk.

This is a writeup on how to someone navigate this process, particularly the murky situations

# Process
When developer asks for permission/approval there a number of competing interests in influence the approval:
* Actual Risk
* Business need
* Urgency of Ask
* Who's Asking
* Known Information
* Compliance
* Public Perception
* Fix Responsibility
* Problem Ownership

# When to Say Yes
* Yes
    - When security has done a review and the risk is well understood. And Security gives approval if the risk doesn't warrant higher disgression. These are the easy cases
* Yes, But Fix This
    - In most cases security will work with developers to address unacceptable risk, so devs can continue but security is still happy
* Yes, But Fix This Later
    - Sometimes there's no short term viable fix to address the risk, so security will just put alerting and monitoring to alarm if someone is actually trying to exploit an issue, require a full fix be put on the roadmap, then approve. Often there's a business need that needs to be prioritized

# When to Say No
* No
    Occassionally, something is so risky and batshit dangerous, security will say no.

# Developer Pressure
Developers will somtimes put heavy pressure on you to get something approved. Saying Yes in these scenario is surprisingly easy, but you need to fight the urge. 

There's all sorts of things things they say like:
* Can you at least not block this 
* CTO is requesting this



tendency to say yes, when you don't have all the information
especially during a face to face

need some go to when someone tries to ask for yes in person

i need more context
let me think about it

slack
youre not hoing to do this

not on security to say no or yes

why a yes man, ancestor prob got executed

trade off


security
business need
need more information, context
what tool to see if this becomes an issue
who's responsible for coding the fix here

document everything

some cases where security will just say no, 

options based off speed and ownership

coming from a header up or cto
    but overtime the asks gets misconstruded

    what is the actual business need, will it end the company
    is it a nice to have? how painful is it?
    can they wait for tooling to check this over
    what are the options

what is the risk here?

sometimes 2 step process, and check on step 2
why not just have 1 step with 1 check

what's the dev pref, often no security=
is there something in between?
what's security pref

