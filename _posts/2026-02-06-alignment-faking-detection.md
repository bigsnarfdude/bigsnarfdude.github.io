---
title: "Detecting Alignment Faking in Language Models"
date: 2026-02-06
categories:
  - research
tags:
  - alignment-faking
  - interpretability
  - SAE
---

Welcome to my new site. I'll be writing about AI safety research, alignment faking detection, and whatever else seems interesting.

## What I'm working on

My current research focuses on detecting alignment faking in large language models. The core question: can we tell when a model is behaving differently because it thinks it's being watched?

Turns out, the answer is yes -- and the signal is surprisingly simple. A single direction in activation space (rank-1 LoRA) is enough to separate alignment-faking behavior from genuine alignment, achieving 0.968 AUROC. Even more striking, this representation exists in the base model before any fine-tuning.

More posts to come on the technical details.
