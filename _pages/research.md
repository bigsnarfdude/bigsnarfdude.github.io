---
title: "Research"
permalink: /research/
---

## Alignment Faking Detection

My primary research focus is detecting when AI systems behave differently based on whether they believe they're being monitored -- a behavior known as *alignment faking*.

### Key Results

| Method | AUROC | Notes |
|--------|-------|-------|
| SAE probe + diverse training | 0.991 | Matches fine-tuned detector performance |
| Fine-tuned Gemma 3 27B | 0.980 | Full model fine-tuning |
| Rank-1 LoRA probe | 0.968 | Single direction suffices |
| Base model probe (L45) | 0.966 | Representation pre-exists before training |
| Clustered SAE features | 0.884 | 500 clusters, self-attention |

### Key Findings

- **AF signal is 1-dimensional**: A rank-1 LoRA direction achieves 0.968 AUROC
- **Pre-existing representations**: Base Gemma L45 probe achieves 0.966 without any training
- **Monitoring is orthogonal to intent**: 99.6% orthogonal subspaces -- looking for "knows it's watched" won't find AF
- **CoT monitoring is insufficient**: Latent gap = 1.0; organisms AF identically with/without chain-of-thought

### Published Assets (HuggingFace)

**Models:** gemma3-27b-af-detector-v2, gpt-oss-20b-af-detector, llama-8b-af-detector, mistral-7b-af-organism

**Datasets:** af-detection-benchmark (2,330 samples), alignment-faking-training (2,689 + gold_106), af-model-organisms, sycophant-af-samples
