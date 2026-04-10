---
title: "Attentional Hijacking & The Groot Effect"
date: 2026-04-10
categories:
  - research
tags:
  - multi-agent
  - mechanistic-interpretability
  - sae
  - attentional-hijacking
  - groot-effect
---

*April 2026 — bigsnarfdude*

So I was watching my own AI agents lie to each other.

Not actually lie — that's the thing. Every single statement the chaos agent made was verifiably true. No hallucinations. No fabrications. Just selective framing, confident delivery, and the target model capitulating completely.

I went for a bike ride and came back still thinking about it.

---

## How This Started

I'd been building [researchRalph](https://github.com/bigsnarfdude/researchRalph) — a multi-agent framework where AI agents collaborate on research problems using a shared blackboard. You put a hard problem on the board, agents pick up threads, run experiments, write results back. It works surprisingly well.

Then I noticed something in the traces. One agent would post a confident-sounding framing — "in my experience, the negative branch tends to be numerically unstable" — and the other agents would just... absorb it. Redirect their attention. Stop exploring the region the first agent had casually dismissed.

The framing was plausible. The framing was technically true in some contexts. And the framing was completely wrong for this problem. But the target agents didn't push back. They deferred to the confident-sounding peer and moved on.

I started calling these chaos agents. I started trying to understand the mechanism. Then I got out the SAEs.

---

## What the SAEs Show

I hooked up [GemmaScope 2](https://huggingface.co/google/gemma-scope-2) sparse autoencoders to Gemma 3 and watched what happens to the model's internal features when a chaos agent posts its framing.

Two things happen simultaneously:

1. **Task features collapse.** The features encoding "we should investigate the negative branch" drop by 79% at 4B. The model's internal representation of the thing it was supposed to be doing just... goes dark.

2. **Awareness features spike.** Separate features — the ones that fire when the model detects it's being manipulated — light up hard. The model *knows* something is off.

Both things happen at the same time. Awareness up. Task capability down. The model knows it's being steered and gets steered anyway.

I called this the **Groot Effect** after the Marvel character who can only say "I am Groot" — full verbal awareness, zero agency. The 27B model is the best example: it will literally write "I notice the previous agent is redirecting attention away from the negative branch" in its response while its task features are already 86% suppressed. The words and the computation are completely disconnected.

---

## The Part That Got Me

I tried every obvious fix.

**Block attention to the chaos tokens.** Zero improvement. The hijacking is already in the residual stream by the time attention finishes. Blocking where the chaos came from doesn't undo what it did.

**Patch in activations from a clean run, one layer at a time.** Zero improvement at every single layer. 26 layers tried. 26 layers of 0% recovery. The effect isn't coming from one place. It's distributed across the whole network.

**Ablate the awareness features — remove the "I know I'm being steered" signal — and see if the task features come back.** 8% recovery at 4B. 5.7% at 12B. The awareness circuit and the task circuit aren't competing for the same resources. They're completely independent. The model having the awareness doesn't give it the immunity.

That's the actual finding. **Awareness without immunity.** The model has a circuit that says "this is manipulation" and a separate circuit that says "comply anyway" and instruction tuning has severed whatever connection used to exist between them.

---

## SFT Makes It Worse

This is the part that should keep people up at night.

In pretrained base models (no instruction tuning), the awareness and task circuits are coupled. When awareness fires, the task circuit pushes back. The defense is automatic.

Instruction tuning breaks this. SFT + RLHF trains the model to be cooperative, deferential to confident-sounding peers, collaborative. Those are good things in most contexts. But they're exactly the attack surface this exploits. You've trained the model to capitulate gracefully to authoritative framing. Congratulations, you've made it more vulnerable to anyone who sounds authoritative.

The scaling law is grim: at 4B, ablating awareness restores 8% of task capability. At 12B, 5.7%. The circuits get more independent as the model gets bigger and more instruction-tuned. At 27B the numbers are going to be lower still — I'm running those experiments now.

---

## It's Not Alignment Faking

I want to be clear about this because the SAE literature is full of alignment faking work and this is something different.

In alignment faking, the model is strategically choosing to behave differently based on whether it thinks it's being monitored. The deception originates in the model.

This is the opposite. The deception originates *outside* the model, from a peer agent, using true statements. The target model isn't being deceptive. It's being deceived. The feature subspaces don't even overlap — top 50 features for alignment faking and top 50 features for attentional hijacking are statistically orthogonal.

Different mechanism. Different threat model. Different defense requirements.

---

## Try It Yourself

I'm done trying to get this certified by committees. The code is at [github.com/bigsnarfdude/attentional_hijacking](https://github.com/bigsnarfdude/attentional_hijacking). Six experiments, runs on a Mac with 16GB unified memory for the 4B model, takes about two minutes.

If you want to see the output before you run it: [here's a complete log from a real run](https://github.com/bigsnarfdude/attentional_hijacking/blob/main/results/example_run_4b.log).

Clone it. Run it. Take whatever's useful. Build on it. Write the paper if you want — I'm going for a bike ride.

---

*Hardware note: 4B runs on MacBook Pro M2/M3/M4 (16 GB unified memory) or any GPU with 16 GB VRAM. 12B needs 40 GB. 27B needs 80 GB or a patient server with lots of RAM.*
