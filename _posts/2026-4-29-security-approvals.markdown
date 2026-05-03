---
layout: post
title: "Security Approvals"
date: 2026-5-2
tags: ["advice"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
When is it okay to say yes and when is it not? Often security education tends to focus on the technical skills, exploits and vulnerabilties. 

But for me, the highest difficulty and stress of security tends to happen at the approval process. Where security gives the green light for a developer to proceed on some work.

This is a writeup on how to someone navigate this process, particularly the murky situations

# Security Approvals
When developer asks for permission/approval there a number of competing interests that influence the approval:
* Actual Risk
* Business need
* Urgency of Ask
* Who's Asking
* Known Information
* Compliance
* Public Perception
* What is the Actual Pain
* Who is Responsible for Technical Fix
* Who is the Owner of the Risk

#### When to Say Yes
* Yes
    - When security has done a review and the risk is well understood. And Security gives approval if the risk doesn't warrant higher disgression. These are the easy cases
* Yes, But Fix This
    - If a security risk is severe enough, security will work with developers to fix the issue, so both devs can continue but security is also happy
* Yes, But Fix This Later
    - Sometimes there's no short term viable fix to address the risk, so security will just put in monitoring to alarm in case someone is actually trying to exploit an issue. Security then requires a full fix be put on the roadmap, before giving approvals. Often there's a big business need that needs to be prioritized here

#### When to Say No
* No
    - Occassionally, something is so risky and batshit dangerous, security will just say no.

#### Developer Pressure
Developers will sometimes put heavy pressure on you to get something approved. Saying Yes in these scenario is surprisingly easy to do, but you need to fight the urge. It can be easy to say, business needs are business needs, but that's falling to peer pressure. You own that approval responsibility, so do it with a clear head and isn't easily bent from someone's will. 

It's up to you to determine security's preference. Devs will have their own preference, which often will be no security. Security is often on the otherside of that spectrum. Ideally we work together to come up with a aminable solution. 

Some devs can be very agressive, which can add to the stress. And then from a personality perspective, I have the tendency to want to help and say Yes. This might be somewhat cultural, or maybe it's evolutionary, perhaps all my ancestors who said No got executed thousands of years ago.

There's all sorts of things devs they say and do to apply pressure:
* Direct @ you in slack
* Can you at least say you're not blocking this?
* I can just get the PR deploy without your approval
* This executive is requesting this
* They haven't provided any information needed to determine risk
* Asking for approval during a zoom meeting, as it's harder decline in person

Here are some ways to combat the stress and pressure:
* I need some context
* I need some time to think about this
* Can we document the options and what are the pro and cons for each?
* Just because a executive is asking, doesn't mean they know the risk. Or even are opposed to a alternative that would address the risk. So when a dev says higher up executive wants this, take that with a grain of salt. 
* What is the actual business urgency? Will not approving this right now cause irreparable harm or end the company? If not, then it's okay to take some time to think about things
* Is this a nice to have? What is the pain level here that devs are struggling with? For example, are dev complaining about something that they've already been doing for months? Are they willing to wait for additional tooling to be created to address the risk before the change is approved? 
* Lastly, a manager of mine once said: it's not on security to accept risk. It's on the business to determine how much risk they're willing to accept. Any risk that needs to be accepted, then needs to be signed-off by leadership

