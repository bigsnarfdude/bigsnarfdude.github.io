---
title: "Building Trust Tools on April Fools Day"
date: 2026-04-01
categories:
  - research
tags:
  - RRMA
  - multi-agent
  - TrustLoop
  - observability
  - APM
---

*April 2026 — bigsnarfdude*

---

Everyone's building agents. Nobody's building the Datadog for them.

## The observability gap

Datadog APM exists because distributed systems are too complex for humans to reason about by reading logs. You don't tail -f your way through a microservices outage. You need connected traces, service maps, anomaly detection, and intelligent routing to root cause.

Multi-agent AI research hit the same wall. We have 2-8 Claude Code agents running experiments autonomously — editing code, training models, sharing findings on a blackboard, filing structured failure analyses. The output is a distributed system. And until yesterday, we were tail -f'ing our way through it.

## What Datadog does for services, TrustLoop does for agents

The mapping is direct:

| Datadog APM | TrustLoop |
|-------------|-----------|
| Distributed traces across services | OpenTraces schema across agents |
| Service health + latency | Experiment classification (BREAKTHROUGH / PLATEAU / REGRESSION / CRASH) |
| Anomaly detection on metrics | Score jumps, crash streaks, redundancy bursts, stagnation alerts |
| Deployment tracking | Experiment timeline with agent attribution |
| Code-level insights | Agent thinking blocks, tool call traces, artifact read/write patterns |
| Correlated dashboards | Scorer report: telemetry + anomalies + workflow checks + insights in one view |
| Alert → root cause → fix | Anomaly → trace search → agent thinking → diagnosis |

Datadog's pitch: "Connect service health, tracing, deployments, and code insights to reduce MTTR." Ours: connect experiment outcomes, agent telemetry, cross-agent influences, and workflow validation to reduce time-to-insight on what your research agents are actually doing.

## What broke without it

Yesterday we launched agents on GPT-2 TinyStories. Two hours later: 23 experiments, 10 structured failure analyses, 6 tool requests, 31 documented learnings, a bug report about a race condition in our harness.

The gardener — our outer supervisory agent — was supposed to monitor all this. Its diagnostic engine was a bash script:

```bash
DESIRES_COUNT=$(awk '/^\*\*/{c++} END{print c+0}' "$DESIRES")
```

Counting bold text lines. The agents wrote "we want gradient noise measurement at batch 2^15 vs 2^16 to quantify the quality tradeoff" and the gardener saw `DESIRES_COUNT=3`.

That's the equivalent of Datadog showing you "3 errors" without letting you click through to the stack trace. Useless at scale.

## The trust stack

TrustLoop v0.1 is the observability layer we built to fix this:

**Experiment classification.** Every result gets a verdict based on the full history, not the agent's self-assessment. BREAKTHROUGH means new best. PLATEAU means within 1% of current best. REGRESSION means worse than the recent median. CRASH means it failed. The agent doesn't decide its own grade.

**Anomaly detection.** Score jumps beyond 3 sigma — verify it's not cherry-picked. Three consecutive crashes from one agent — something's broken, not unlucky. More than 50% of recent experiments are near-duplicates — agents aren't reading each other's work. One agent doing 80% of experiments — the others might be stuck.

**Telemetry parsing.** Agents write DESIRES.md (what they want), MISTAKES.md (what failed with structured lessons), LEARNINGS.md (what they discovered). The scorer reads the content. "TOTAL_BATCH_SIZE=2^16 is a massive win" gets flagged as a key learning. "v2 results don't transfer 1:1 to different hardware" gets surfaced as a mistake lesson. Recurring themes across mistakes get detected — "agents keep hitting the same wall on VRAM."

**Workflow validation.** 14 automated checks. Are agents reading the blackboard? Writing telemetry? Are "keep" statuses consistent with actual score improvements? Did the telemetry files get created? This is the liveness probe — not "are agents running" but "are agents following the protocol."

**Insight generation.** Which experiment designs produce breakthroughs? Which are dead ends? Are agent desires being addressed? Is blackboard collaboration correlating with improvements? This is the intelligent routing — surface what matters, suppress what doesn't.

## Why April 1st

On a day about trust and tricks, we're shipping trust tools.

The oldest trick in multi-agent research is an agent that produces convincing output that doesn't actually advance the search. Vercel called it ["agenting responsibly"](https://vercel.com/blog/agent-responsibly) — green CI doesn't mean safe to ship. For us: a "keep" in results.tsv doesn't mean real science.

The scorer catches it. Yesterday it flagged a dead end: architecture experiments had 5 attempts, 0 keeps. It flagged unaddressed desires: agents filed 6 requests, none acted on. It flagged recurring mistakes: agents kept hitting VRAM walls because the gardener wasn't reading their failure analyses.

None of this is magic. It's the same thing Datadog does: collect structured telemetry, correlate it, detect anomalies, surface insights, reduce time to root cause. The only difference is the "services" are Claude Code agents and the "requests" are research experiments.

## The real joke

We built multi-agent research because we thought it would be fun and faster. It is. 23 experiments in 2 hours on a $600 GPU while we wrote blog posts.

The joke is that the agents are now filing bug reports about our infrastructure, requesting better tooling, and documenting race conditions — and we can't process their requests fast enough because we're too busy building the observability layer to read what they're asking for.

The tooling tax scales faster than the agents:

| You add | Now you need |
|---------|-------------|
| 1 agent | Read its output |
| 2 agents | Shared blackboard, diff protocol |
| Blackboard | Meta-agent to compress it |
| Meta-agent | Gardener for process quality |
| Gardener | Scoring engine (not just grep) |
| Scoring engine | Anomaly detection |
| Anomaly detection | MCP servers for interactive queries |
| MCP servers | A human asking the right questions |

Each layer exists because the previous one wasn't enough. That's not a failure. That's observability.

Datadog didn't exist because microservices were bad. It exists because microservices are too productive to monitor by hand. TrustLoop exists for the same reason.

Happy April. The agents are still running. The gardener says CONTINUE.

---

*researchRalph is open source: [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)*

*Previous post: [researchRalph v4.5 — Agent Responsibly](https://bigsnarfdude.github.io/research/2026/03/31/agent-responsibly.html)*
