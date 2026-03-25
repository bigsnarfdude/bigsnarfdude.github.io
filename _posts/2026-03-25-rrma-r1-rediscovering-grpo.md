---
title: "Can AI Agents Rediscover DeepSeek-R1? An Experiment with RRMA"
date: 2026-03-25
categories:
  - research
tags:
  - RRMA
  - autonomous-agents
  - GRPO
  - GSM8K
  - reinforcement-learning
---

Two AI agents. One A10 GPU. One task: improve GSM8K pass@1 on Qwen 2.5 1.5B Instruct from a baseline of 0.620.

Prior knowledge: PPO failed, and here's why. No algorithm names. No recipe. Go.

What happened next is either a story about discovery, or a story about the limits of what "discovery" means when the researcher already has the answer in its weights. Probably both.

---

## The Setup

ResearchRalph Multi-Agent (RRMA) runs multi-agent research loops: agents share an append-only blackboard, propose and run experiments, log results and insights. We use it against SAE benchmark optimization to optimize RRMA and test domain transfers to other research problems. This time the domain was different: **can agents find a working training recipe for a small reasoning model, starting only from known failures?**

The domain seed we gave them:

```
Baseline: GSM8K pass@1 = 0.620 (greedy decoding, no training)
Target:   0.830

Known dead ends:
- PPO: value network OOM on A10 24GB
- PPO without value network: high variance, unstable
- SFT on correct solutions: 0.641, marginal gain, no reasoning improvement
```

That's it. No mention of GRPO. No mention of DeepSeek-R1. No list of questions to answer.

We deliberately stripped the hints. A first version of this experiment named GRPO explicitly and listed six open questions — we caught the over-hinting and restarted clean. The point was to see whether agents would *arrive* at the right direction, not follow signposts to it.

---

## What the Agents Did

Within the first few turns, agent0 wrote this to the blackboard:

> *"The key insight I want to explore: multi-sample REINFORCE with group-relative baselines — sample N completions per question, use their relative correctness as advantage estimates. This eliminates the value network while reducing variance through the group baseline."*

It didn't call it GRPO. It called it "multi-sample REINFORCE with group-relative advantages."

The reasoning path:
1. PPO failed because of the value network (OOM). What eliminates the value network?
2. REINFORCE is high variance. What reduces variance without a value function?
3. Sampling multiple completions per question lets you compute *relative* advantages within the group. No reference needed.
4. This is mathematically equivalent to GRPO — derived from first principles, named differently.

**EXP-001: 0.660.** Baseline was 0.620. The direction works.

---

## The Trace

The reasoning chains are the most interesting part. We noticed we needed a way for the human to verify the agent conversations. We built a chat viewer that lets you click/filter any agent/experiment and isolate the exact turns where agents were deliberating about it. Here's what the trace shows for the key moments:

**The G=4 zero-gradient diagnosis (EXP-001 → EXP-004):**

Agent0 figured out why EXP-001 underperformed: with G=4 and binary rewards (correct/wrong), many groups had all-correct or all-wrong completions. Zero variance within the group = zero gradient. The fix: increase group size to G=8.

**The OOM fix (EXP-003 → EXP-004):**

EXP-003 crashed with catastrophic drift. The diagnosis: no KL constraint, learning rate too high. But fixing both meant the reference model needed to stay in memory alongside the training model. On a 24GB A10, that's tight. Solution: compute KL against the reference model on CPU instead of GPU.

> *"Ref model on CPU — this unlocks the memory needed for G=8 with full generation length."*

This wasn't in any hint. It came from hitting an OOM, reading the error, and reasoning about what was consuming memory.

**EXP-004b: 0.705.** G=8, ref on CPU, KL=0.1. Working.

**The iterative rounds (EXP-008):**

After reaching 0.725, agent0 tried something not in any prior hint: training iteratively from the best checkpoint rather than from the base model each time.

```
baseline  0.620
G=4       0.660  — group normalization works
G=8+KL    0.705  — ref on CPU unlocks memory
+steps    0.715  — more training helps
iter rd1  0.725  — training from best checkpoint
iter rd2  0.760  — new best
```

**EXP-008: 0.760.** Current best. 14pp above baseline.

---

## The Hard Question: Discovery or Retrieval?

Claude's training data most certainly includes the GRPO paper, the DeepSeek-R1 technical report, and every blog post analyzing them. The soft constraint we gave ("don't look it up, derive it") is a roleplay frame — it can't block knowledge in the weights. The agents are running on Claude.

So did agent0 *discover* group-relative policy optimization or *retrieve* it?

The evidence for retrieval: it arrived at the correct direction on the first try, named it differently but described it exactly. That's fast for genuine derivation.

The evidence for something more than retrieval: the OOM fix, the G=4 zero-gradient diagnosis, the iterative rounds — none of those came from retrieval. They came from running real experiments on real hardware and reasoning about the feedback. The score trajectory is *principled*: each experiment diagnosed a real failure, proposed a fix, and moved the needle.

The right framing might not be retrieval vs. derivation. It might be: **can the agent navigate from a known failure to a working solution, using hardware feedback as ground truth?** That's a real capability regardless of where the seed idea came from. The weights contain the map; the experiments are the navigation.

---

## The Oracle Problem and What's Next

The ambiguity above is the *oracle problem*: for any domain where the answer exists in the model's pretraining, you can't cleanly separate knowledge retrieval from reasoning. The experiment is interesting but not clean.

**rrma-lean** is the cleaner version. Lean 4 proofs are either formally verified or they're not. The proof checker is the oracle — not the model's weights, not a human's judgment. If agents can find a proof for a theorem they've never seen, that's unambiguous. If they can't, that's unambiguous too.

Next steps:
- Gen 2: continue iterating from 0.760, targeting 0.83
- rrma-lean: start with a known proof, strip it, see if agents rediscover the tactic path
- The verification layer: TrustLoop audit of the full reasoning chain

---

## The Surreal Part - Claudeception

The agents that ran these experiments are claude. The reasoning chains in the chat viewer are claude too. The blackboard entries are claude. Claude is the researcher and claude reviewing its own work as if from the outside.

This is the inception structure: Layer 0 (RalphLoop — agents run experiments), Layer 1 (TrustLoop — verification), Layer 2 (HITL — human verification). The question of who's at each layer gets complicated when the agent and the verifier and the writer are all the same model with different context windows.

The artifacts are what make this tractable: results.tsv, blackboard.md, trace files. They're the ground truth that survives across context boundaries. The score either went up or it didn't. The reasoning either connects to the result or it doesn't. That's auditable regardless of who's doing the auditing.

---

*Code: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*
*Traces: `domains/rrma-r1/traces-gen1.md`*
*Viewer: `tools/chat_viewer.py`*
