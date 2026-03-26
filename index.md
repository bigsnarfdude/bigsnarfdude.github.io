---
layout: single
author_profile: true
title: ""
---

# Hi, I'm bigsnarfdude

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
- Published **38 models** and **9 datasets** on [HuggingFace](https://huggingface.co/vincentoh)

## Recent Projects

- **[truthsayer](https://github.com/bigsnarfdude/truthsayer)** -- Geometric decomposition of deception in LLMs. AF and sandbagging have distinct, nearly-orthogonal SAE signatures (cosine = 0.033). 0.994 AUROC on held-out AF detection.
- **[thoughtSonar](https://github.com/bigsnarfdude/thoughtSonar)** -- Investigating sandbagging detection limits. SAE probes hit 98.9% within-family but fail cross-generator (0.36-0.56), revealing sandbagging as an "absence signal" unlike AF's detectable "presence signal."
- **[tellthetruth](https://github.com/bigsnarfdude/tellthetruth)** -- Independent replication of Goodfire's "Features as Rewards." Training data choice matters more than probe architecture for hallucination detection (0.762 AUROC).
- **[researchRalph](https://github.com/bigsnarfdude/researchRalph)** -- Multi-agent Claude research system with shared blackboard. V4 adds a "gardener" meta-agent that detects metric-gaming and redesigns scaffolding autonomously.

## Recent Writing

- [Can AI Agents Rediscover DeepSeek-R1?]({% post_url 2026-03-25-rrma-r1-rediscovering-grpo %}) -- RRMA agents independently converge on GRPO, reaching 0.820 pass@1 via majority voting
- [Trust but Verify: The Human Verification Layer]({% post_url 2026-03-25-trustloop-human-rrma-bridge %}) -- Building the oversight layer for autonomous AI research agents

## Links

- [GitHub](https://github.com/bigsnarfdude)
- [HuggingFace](https://huggingface.co/vincentoh)
- [Kaggle](https://www.kaggle.com/vincento)
