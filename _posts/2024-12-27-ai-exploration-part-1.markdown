---
layout: post
title: "AI Exploration: Part I - Landscape"
date: 2024-12-27
tags: ["ai"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement

Create a hierarchical structure of the AI landscape based on tools I've seen or heard of

# Landscape

```
Artificial Intelligence
└── Machine Learning
    └── Deep Learning-Based Models (a.k.a. neural networks)
        ├── Discriminative Models
        │   └── CNNs (Convolutional Neural Networks)  
        │       ├── Ex: AWS Rekognition    
        └── Generative Models
            ├── GANs (Generative Adversarial Networks)
            │   └── Ex: Artbreeder                
            ├── VAEs (Variational Autoencoders)
            │   └── Ex: DeepFaceLab              
            ├── Diffusion Models
            │   ├── Ex: Stable Diffusion
            │   ├── Ex: MidJourney
            │   └── Ex: DALL-E
            └── LLMs (Large Language Models)
                └── GPT (Generative Pre-trained Transformer) Model
                    └── Ex: ChatGPT            
```                           

This AI hierarchy is likely incorrect. Unfortunately I don't know enough about the field to be able to sus out the distinctions, but this will work as a template. I got hard stuck with including the following terms in the hierarchy: 

* Generative AI
* Supervised learning
* Semi-supervised learning
* Self-supervised learning
* Unsupervised learning
* Reinforcement learning
* Deep learning vs Neural networks[^1]

Maybe adding these other terms cause category conflicts that aren't easy to map via a tree structure. I might need to understand the underlying mathematical principles to sort this all out, maybe a venn bubble diagram would be better...

# LLM companies[^2]

| **Dev**       | **Model**     | **Access** |
|---------------|---------------|------------|
| OpenAI        | **GPT**       | Closed     |
| Google        | **Gemini**    | Closed     |
| Google        | **Gemma**     | Open       |
| Meta          | **Llama**     | Open       |
| Anthropic     | **Claude**    | Closed     |
| Cohere        | **Command**   | Closed     |
| Microsoft     | **Phi-4**     | Open       |
| Mistral AI    | **Mistral**   | Open       |
| Stability AI  | **Stable LM** | Open       |
| Amazon        | **Nova**      | Closed     |

# Learning Material

* [Neural Network Course by 3Blue1Brown](https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi)
* [AI Engineering by Chip Nyugen](https://www.amazon.com/AI-Engineering-Building-Applications-Foundation-ebook/dp/B0DPLNK9GN/)
* [Simon Willison's Blog](https://simonwillison.net/tags/ai/)
* [LLMs in 2024](https://simonwillison.net/2024/Dec/31/llms-in-2024/#prompt-driven-app-generation-is-a-commodity-already)
* [2024 AI Timeline](https://huggingface.co/spaces/reach-vb/2024-ai-timeline)

# References

[^1]: *"The terms deep learning and neural networks are used interchangeably because all deep learning systems are made of neural networks."* [https://aws.amazon.com/compare/the-difference-between-deep-learning-and-neural-networks/#:~:text=The%20terms%20deep%20learning%20and,are%20made%20of%20neural%20networks.](https://aws.amazon.com/compare/the-difference-between-deep-learning-and-neural-networks/#:~:text=The%20terms%20deep%20learning%20and,are%20made%20of%20neural%20networks.)

[^2]: [https://zapier.com/blog/best-llm/](https://zapier.com/blog/best-llm/)