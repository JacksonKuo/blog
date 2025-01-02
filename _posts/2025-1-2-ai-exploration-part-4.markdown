---
layout: post
title: "AI Exploration: Part IV - Security"
date: 2025-1-1
tags: ["ai"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement

system is not a security boundary.

end of the data 101 to llm

security
need authorization for external tools, rag, db, vector
dont' store secrets in system prompt
many apps, no markdown and links/images

evolution of prompt injection
ignore
more complex instructions



here's chatgpt prompt

direct prompt injection 
second prompt
jail break
x-openai-isConsequential

```
Spotlighting [1]: A preventative defense that “marks” data (as opposed to instructions) that is provided to an LLM using methods like adding special delimiters, encoding data (e.g., in base64), or marking each token in the data with a special preceding token. 

PromptShield [2]: A black-box classifier designed to detect prompt injections, ensuring that malicious prompts are identified and mitigated. 

LLM-as-a-judge: This defense uses an LLM to detect attacks by evaluating prompts instead of relying on a trained classifier. 

TaskTracker [3]: Based on analyzing the model’s internal states to detect task drift, this defense extracts activations when the user first prompts the LLM and again when the LLM processes external data. It then contrasts these activation sets to detect drift via a linear probe on the activation deltas. 
```

reading material

embedded 
simon williams prompt 

ways to call external tools
function calls
python sandbox


# References

[^1]: [https://msrc.microsoft.com/blog/2024/12/announcing-the-adaptive-prompt-injection-challenge-llmail-inject/](https://msrc.microsoft.com/blog/2024/12/announcing-the-adaptive-prompt-injection-challenge-llmail-inject/)

owasp top ten for llm