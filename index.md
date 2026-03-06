---
layout: single
author_profile: true
title: ""
---

# Hi, I'm Vincent

I work on **AI safety research**, with a focus on alignment faking detection, mechanistic interpretability, and building tools to make AI systems more transparent.

## Research Interests

- **Alignment Faking Detection** -- SAE probes, mechanistic interpretability, and scalable oversight
- **Interpretability** -- Understanding how language models represent intent and deception
- **Adversarial Robustness** -- Testing whether deception probes survive adversarial fine-tuning

## Key Findings

- **0.991 AUROC** on alignment faking detection using SAE probes with diverse training -- matching fine-tuned detectors
- AF signal is **1-dimensional** (rank-1 LoRA suffices) and **pre-exists in base models** (L45 probe = 0.966 without training)
- AF is geometrically closer to **factual lying** than strategic deception; AF and sycophancy are **representationally independent**
- Monitoring-awareness is **99.6% orthogonal** to AF intent -- methods looking for "knows it's watched" won't find AF
- Deception probes are **adversarially robust** -- adversarial SFT can rotate the signal but fresh probes always recover 1.0 AUROC
- Published **6 models** and **2 datasets** on [HuggingFace](https://huggingface.co/vincentoh) across 61 days of research

## Links

- [GitHub](https://github.com/bigsnarfdude)
- [HuggingFace](https://huggingface.co/vincentoh)
- [Kaggle](https://www.kaggle.com/vincento)
