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
        │       ├── Ex: Google Vision AI  
        │       └── Ex: Clarifai
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
* Deep learning vs Neural networks[^1]

Maybe adding these other terms cause category conflicts that aren't easy to map via a tree structure. I might need to understand the underlying mathmathical principles to sort this all out, maybe a venn bubble diagram would be better...

# LLM companies[^2]

| **Dev**       | **Model**     | **Access** |
|---------------|---------------|------------|
| OpenAI        | **GPT**       | Closed     |
| Google        | **Gemini**    | Closed     |
| Google        | **Gemma**     | Open       |
| Meta          | **Llama**     | Open       |
| Anthropic     | **Claude**    | Closed     |
| Cohere        | **Command**   | Closed     |
| Microsoft     | **Phi-3**     | Open       |
| Mistral AI    | **Mistral**   | Open       |
| Stability AI  | **Stable LM** | Open       |


# References

[^1]: `The terms deep learning and neural networks are used interchangeably because all deep learning systems are made of neural networks.` [https://aws.amazon.com/compare/the-difference-between-deep-learning-and-neural-networks/#:~:text=The%20terms%20deep%20learning%20and,are%20made%20of%20neural%20networks.](https://aws.amazon.com/compare/the-difference-between-deep-learning-and-neural-networks/#:~:text=The%20terms%20deep%20learning%20and,are%20made%20of%20neural%20networks.)

[^2]: [https://zapier.com/blog/best-llm/](https://zapier.com/blog/best-llm/)