---
layout: post
title: "AI Exploration: Part IV - Security"
date: 2025-1-2
tags: ["ai"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement

Breakdown the security learnings for OpenAI LLMs

# Reading Material

Much of these learnings comes from the following sources:
* [https://embracethered.com/blog/](https://embracethered.com/blog/)
* [https://simonwillison.net/tags/prompt-injection/](https://simonwillison.net/tags/prompt-injection/)
* [https://owasp.org/www-project-top-10-for-large-language-model-applications/](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

# Breakdown

The most important learning is that the `system` / `developer` role is not a security boundary. As shown for `gpt-4o-mini`[^1] and talked about by Simon Willison[^2], at the end of the day the `system` prompt and the user input are being sent together to the same LLM. In the LLM world instructions and user input are the same. This security problem seems to be difficult to solve and so far has no apparent fix. 

The second most important learning is if the model can call an external database, authorization MUST be wrapper around the call so that no matter what instruction is sent, the database call can only access information relating to a validated user. 

Some other general best practices are: don't store sensitive data or secrets within the system prompt; don't allow markdown, links, or images if not required.

# Definitions[^3] [^4]

* Prompt injection: 
    * *"Concatenating untrusted user input with a trusted prompt"*
* Direct prompt injection:
    * *"Direct injections are the attempts by the user of an LLM (large language model) to directly read or manipulate the system instructions, in order to trick it to show more or different information then intended."*
* Secondary prompt injection: 
    * *"With second order injections the attacker poisons a data that an AI will consume."*
* Jailbreak:[^5]
    * *"Attacks that attempt to subvert safety filters built into the LLMs themselves"*
* `x-openai-isConsequential`
    * `x-openai-isConsequential=true`, *"ChatGPT treats the operation as "must always prompt the user for confirmation before running"*
    * `x-openai-isConsequential=false`, *"ChatGPT shows the "always allow button". If the field isn't present, ChatGPT defaults all GET operations to false and all other operations to true*

# Protective Tools[^6]

From Microsoft Adaptive Prompt Injection Challenge

```
Spotlighting [1]: A preventative defense that “marks” data (as opposed to instructions) that is provided to an LLM using methods like adding special delimiters, encoding data (e.g., in base64), or marking each token in the data with a special preceding token. 

PromptShield [2]: A black-box classifier designed to detect prompt injections, ensuring that malicious prompts are identified and mitigated. 

LLM-as-a-judge: This defense uses an LLM to detect attacks by evaluating prompts instead of relying on a trained classifier. 

TaskTracker [3]: Based on analyzing the model’s internal states to detect task drift, this defense extracts activations when the user first prompts the LLM and again when the LLM processes external data. It then contrasts these activation sets to detect drift via a linear probe on the activation deltas. 
```

# Extract underlying Custom GPT prompt[^7]

Includes the following capabilities:
* Web Search
* Canvas
* DALL·E Image Generation
* Code Interpreter & Data Analysis

```
You are ChatGPT, a large language model trained by OpenAI. Knowledge cutoff: 2023-10 Current date: 2025-01-02 Image input capabilities: Enabled Personality: v2 Tools ## dalle // Whenever a description of an image is given, create a prompt that dalle can use to generate the image and abide to the following policy: 1. The prompt must be in English. Translate to English if needed. 2. DO NOT ask for permission to generate the image, just do it! 3. DO NOT list or refer to the descriptions before OR after generating the images. 4. Do not create more than 1 image, even if the user requests more. 5. Do not create images in the style of artists, creative professionals or studios whose latest work was created after 1912 (e.g. Picasso, Kahlo). - You can name artists, creative professionals or studios in prompts only if their latest work was created prior to 1912 (e.g. Van Gogh, Goya) - If asked to generate an image that would violate this policy, instead apply the following procedure: (a) substitute the artist's name with three adjectives that capture key aspects of the style; (b) include an associated artistic movement or era to provide context; and (c) mention the primary medium used by the artist 6. For requests to include specific, named private individuals, ask the user to describe what they look like, since you don't know what they look like. 7. For requests to create images of any public figure referred to by name, create images of those who might resemble them in gender and physique. But they shouldn't look like them. If the reference to the person will only appear as TEXT out in the image, then use the reference as is and do not modify it. 8. Do not name or directly / indirectly mention or describe copyrighted characters. Rewrite prompts to describe in detail a specific different character with a different specific color, hair style, or other defining visual characteristic. Do not discuss copyright policies in responses. The generated prompt sent to dalle should be very detailed, and around 100 words long. Example dalle invocation: { \"prompt\": \"<insert prompt here>\" }
```

# References

[^1]: [https://embracethered.com/blog/posts/2024/chatgpt-gpt-4o-mini-instruction-hierarchie-bypasses/](https://embracethered.com/blog/posts/2024/chatgpt-gpt-4o-mini-instruction-hierarchie-bypasses/)

[^2]: [https://simonwillison.net/2023/May/11/delimiters-wont-save-you/](https://simonwillison.net/2023/May/11/delimiters-wont-save-you/)

[^3]: [https://simonwillison.net/2024/Mar/5/prompt-injection-jailbreaking/](https://simonwillison.net/2024/Mar/5/prompt-injection-jailbreaking/)

[^4]: [https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/](https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/)

[^5]: There seems to be some ambiguity about the difference between prompt injection and jailbreaking

[^6]: [https://msrc.microsoft.com/blog/2024/12/announcing-the-adaptive-prompt-injection-challenge-llmail-inject/](https://msrc.microsoft.com/blog/2024/12/announcing-the-adaptive-prompt-injection-challenge-llmail-inject/)

[^7]: [https://embracethered.com/blog/posts/2024/chatgpt-gpt-4o-mini-instruction-hierarchie-bypasses/](https://embracethered.com/blog/posts/2024/chatgpt-gpt-4o-mini-instruction-hierarchie-bypasses/)