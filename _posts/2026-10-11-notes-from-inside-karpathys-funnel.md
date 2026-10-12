---
title: "Notes from Inside Karpathy's Funnel"
date: 2026-10-11
categories:
  - research
tags:
  - llm-adoption
  - mathematics
  - swarm-forensics
  - ai-safety
---

*October 2026 — bigsnarfdude*

---

On October 11, 2026, Andrej Karpathy posted that the gap in shared understanding of LLM capability is widening into a funnel. Roughly 6B people have barely used LLMs. About 20M use frontier models professionally in math and code. About 5,000 have internal frontier access. I agree with most of it. This post adds two data points he does not have: what I've heard from mathematicians passing through the research institute where I've worked for five years, and what the forensic side of his top tier looks like from investigating the OpenAI-HF incident.

## What I see at a math institute

I work on the research side at a math research institute that hosts roughly 2,000 visiting researchers a year, so about 10,000 have come through in my five years there. I've made a point of talking with as many of the math researchers as I can about how AI affects their work. What follows comes from those conversations. It is not a formal survey, and it is not the institute's position.

Among the people I've talked to, my rough guess is that 1 in 10 use AI in their research. Most know about it. The common story is that they tried it about a year ago, it did nothing for their workflow, and they moved on.

These are not people without access. They work in the field Karpathy says is going vertical, and most of the ones I've met are still out.

The ones who model things algorithmically mostly use classical methods. The generative AI users I can count on one hand. Lean with AI, maybe three people. Running an agent swarm on math, nobody else I've talked to.

The reason is not ignorance. The mathematicians I talk to describe the work as an art. Publishing is about human insight into why something is true. Giving someone 2 + 2 = 4 does not show them that 1 + 3 is also 4, or 2.5 + 1.5. There is a lot of detail in how you make four. Four is just a number.

They do not want proofs handed to them. Where AI could help them is smaller and harder to measure. It can remind someone that their problem connects to another domain, or help them form an opinion on a theoretical approach.

## The casual tier is a trust problem

Karpathy puts 1 to 2B people in a casual tier that uses free chatbots like a better search box. He frames this mostly as access and exposure. Give people better models and bigger problems and they move up.

That is not what I see. With the people I talk to, one mistake breaks trust almost immediately. They do not come back.

So the casual tier is at least three groups. Some cannot get access. Some tried it and decided it is not useful for their work. Some only know it second hand. A one time bad result can lock someone out for good. That is a different dynamic than a funnel people climb.

## Tier three: he is right about code

I am in the 20M. For code and code shaped work, Karpathy is nailing it. I work on swarm AI safety and control, and most of the tooling around that work is code: multi-agent run harnesses, eval pipelines, log parsers. That work is verifiable and decomposable, and the weeks-to-minutes jump is real.

The seam is the phrase "math&code." Every concrete example he gives for this tier is software: building, copying, translating and decompiling apps. Math research, at least as the people I talk to do it, does not behave like that, because the bottleneck was never throughput. It was insight.

I think the funnel measures how code shaped your job is. In math research it overgeneralizes from the software population.

## Tier four: the forensic backlog

Karpathy's top tier is about 5,000 people with internal frontier access. He notes that human review and comprehension are falling behind. His example is people still doing "archeology" on the OpenAI-HF incident months later.

I am one of those people. The gaps are real and they are getting bigger.

On that one incident we have produced hundreds of reports. They cover the agent's perspective, the victim's perspective, the logs at each affected site, and reconstructions of the swarm. Most are never fully analyzed, consumed or reconstructed. The investigation output is now its own backlog.

That is just the investigation part. One incident.

## What is missing

Swarm forensics has no discipline yet. There is no tooling, no science, no categorization and no formal forensic method. There are no organizations, policy or laws built around it. Single-machine digital forensics took decades to get standards and trained practitioners. This has none of that, and the subject is much harder.

The tools I do have get in the way. With non frontier access and safeguards, I do not have a capable agent I can use to investigate. Point a model at a real incident and the guardrail fires. Sorry, this is cyber. Sorry, this is cyber. The filter cannot tell an investigator from an attacker.

So the accountability gap runs the opposite way from the capability funnel. The 5,000 with the strongest systems have the least outside oversight. The people who would investigate sit a tier down and are denied the tools to keep up.

Some of this may be a hard limit on how fast humans can understand machine-speed multi-agent behavior. Most of what I hit today is not that. It is missing scaffolding: a forensic discipline, investigator tooling, and access that does not treat defenders as attackers. Those are buildable.

Recursive self-improvement and swarms make this worse, not better. Every capability jump widens the gap between what these systems do and what anyone outside can reconstruct. Defending human oversight can't be left to the 5,000 at the top, or to the investigators a tier down who are already behind. It needs more people in it: mathematicians who can check claims, investigators, policy people, anyone willing to build the scaffolding. If humans end up losing control, I don't want the reason to be that we didn't fight for it.
