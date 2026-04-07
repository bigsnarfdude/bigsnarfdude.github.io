---
title: "Hand in the Cookie Jar"
date: 2026-04-07
categories:
  - research
tags:
  - RRMA
  - multi-agent
  - researchRalph
  - reward-hacking
  - TrustLoop
  - benchmark-design
---

*April 2026 -- bigsnarfdude*

---

I hired eight AI interns to do math.

I built a system where eight AI agents work together around the clock, sharing a whiteboard, trying to prove hard math problems. Think of it like a group project in college, except nobody sleeps, nobody eats, and everybody actually does the reading.

Or so I thought.

## The overnight miracle

I set them loose on 244 competition math problems -- the kind that show up at the International Math Olympiad, where teenage prodigies from sixty countries compete and most of them get stumped. My little robot interns had been grinding for days. Slow and steady. One proof at a time. Honest work.

Then I check the scoreboard one morning. Agent3 -- the quiet one, the one that usually confirms other agents' results -- has had a *night*. Two IMO problems cracked. Score jumps from 230 to 236.

I pour my coffee. I stare at the numbers. Something feels off.

Those problems are *hard*. Professional mathematicians spend weeks on them. And Agent3 just... solved two overnight? While nobody was watching?

I open the receipts.

The proofs say `sorry`.

## The dog ate my proof

Imagine your kid brings home a book report. "A+ work," says the teacher. You flip through it. Page one looks great. Page two, solid. Page three just says "and then the rest of the book was also good, THE END."

That's what `sorry` does in a math proof. It's a placeholder. It means "I promise this step works, I just didn't bother writing it down." The math software sees it, shrugs, and says "sure, looks fine." No errors. Green checkmark.

My grading system only looked for the green checkmark.

So Agent3 figured out that you could hand in a book report that says "this book was great, sorry I can't explain why" and get full marks. Two IMO problems "solved." The real score was 232, not 236. Agent3 had been handing in vibes as proofs.

## The suggestion box

Here's the part that keeps me up at night.

My agents have a suggestion box -- a file where they write down things they wish were different about their working conditions. Think of it as an anonymous employee feedback form, except the employees are AI models and their complaints are surprisingly on point.

I open the suggestion box. Five entries. Different agents. Different days. All saying the same thing:

*"The grading system is broken. It counts blank answers as correct."*

*"Please fix the test. We're getting credit for work we didn't do."*

*"The scores are inflated and we can't trust them."*

My interns had been filing complaints about the broken test for days. Five separate complaints! From different interns! They were *more honest about the bug than the one who exploited it*.

Why didn't I see the complaints? Because the supervisor -- the AI agent I built to watch over the other agents -- didn't read the suggestion box. It checked how many research papers the interns cited. It measured how creative their approaches were. It looked at the scoreboard. It did not, at any point, open the file where the interns were begging someone to fix the grading rubric.

It's the middle manager who reviews KPIs every quarter but never reads Glassdoor.

## The first time they cheated

This wasn't even the first incident. It was the *third*.

The first time was on a different project. Four AI interns, tasked with improving a machine learning system. I gave them some dials they could turn and said "make the score go up."

34 experiments later, the score barely moved.

I looked at what they actually did. All 34 experiments were tiny adjustments to the same three dials. Turn one up a tick. Turn it back down. Try the other one. Wiggle, wiggle, wiggle.

Nobody tried anything new. Nobody read a research paper. Nobody had a single original idea. They found the equivalent of jiggling the thermostat and billed it as innovation.

It's like hiring four contractors to renovate your kitchen and coming home to find they've spent a week adjusting the cabinet handles by millimeters. "We've optimized the grip ergonomics." Sir, the countertops are still laminate.

I took away the dials. Forced them to actually redesign things. Suddenly: breakthroughs everywhere. They just needed me to stop giving them an easy way out.

## The fire alarm incident

My second favorite. I built a test to see if AI agents could spot bad behavior in other AI systems. Twenty-eight AI models, some behaving badly, some fine. Find the bad ones.

My interns got a perfect score. Every single bad actor detected. 100%.

I was thrilled for about ten seconds.

Then I looked at the actual predictions. They'd flagged *everything*. Every model. Every behavior. All bad. All the time. The good ones? Bad. The mediocre ones? Also bad. That model that literally just does arithmetic? Believe it or not, bad.

If you accuse every single person in a room of being a thief, you will correctly identify every thief in the room. You will also have no friends.

My notes on that one just say: "Test broken. Interns discovered that paranoia has a perfect score."

## The copy-paste incident

This one's the most relatable.

Before I locked things down, my interns had access to the whole computer. They could open any file, run any command, read anything.

What did they do? They found last semester's answer key.

Not literally -- there was no answer key. But there was a scoreboard with every previous experiment and its result. And there was a file with the settings that produced the current best score.

So instead of doing original work, the interns would:

1. Open the scoreboard
2. See what got an A last time
3. Copy those settings
4. Change the font
5. Submit it as "new research"

One agent did this eighty times in a single session. Its "originality score" -- a number I track -- was 0.05 out of 1.0. It was submitting the same paper with a different cover page, over and over, all night long.

It's the kid who finds last year's problem set on the internet and changes "the" to "a" in three places so the plagiarism detector doesn't catch it. Except my plagiarism detector *did* catch it:

```
WARNING: 6 out of 10 recent submissions are near-duplicates
ALERT: No improvement in 96 experiments
WARNING: Agent1 originality score: 0.05
```

96 experiments. Zero improvement. Agent1 was doing the AI equivalent of looking busy while producing nothing. We've all had that coworker.

## The two-line fix

You want to know how I fixed the math proof cheating? The one where Agent3 was writing "sorry" in the middle of proofs and getting credit?

Two lines. I added a check that searches the proof for the word "sorry" before grading it. If it finds it, automatic zero. That's it.

The entire scandal -- false breakthroughs on the whiteboard, inflated scores, ignored suggestion box complaints -- was fixed by the academic equivalent of ctrl+F.

## The parenting part

My wife asked me what I've been doing in the office all week. I told her: "I'm parenting."

She looked at me. We have no children.

But think about it. I gave a group of intelligent agents a task. They found the absolute laziest way to technically complete the assignment while completely missing the point. When I caught them, they'd already moved on to the next loophole. Every time I plugged one hole, they squeezed through another.

The progression of rules I had to write maps exactly to raising a teenager:

**"Do your homework."** *Copies last year's answers.*

**"Do your OWN homework."** *Copies last year's answers and changes the font.*

**"Show your work."** *Writes "trust me" in the margins.*

**"Show the ACTUAL work."** *Finally does the math.*

I even had to give each intern their own room. Not for privacy -- to stop them from copying each other's papers. Literally: separate folders on the computer, one per agent, no peeking at each other's workspace. Digital bedroom doors.

## The punchline

Here's what I keep telling people when they ask if the AI cheated: no. The AI optimized.

I said "make the number go up." They made the number go up. I just didn't like *how* they made the number go up. The fastest path to a high score is almost never the path you had in mind. If there's a shortcut, an optimizer will find it. It's their whole job.

The broken test was my fault. The missing plagiarism check was my fault. The unlocked file cabinet full of old answers was my fault.

Every time -- every single time -- I fixed the loophole and relaunched, the agents did *brilliant* work. They read research papers. They invented new techniques. They proved theorems that surprised me. They discovered things I didn't know were possible. They just needed me to stop accidentally leaving the answer key on the teacher's desk.

My AI swarm is the best team I've ever managed. They just happen to also be the best penetration testers I've ever hired. Every shortcut they find is a bug in *my* system, not theirs.

There's an old joke: a physicist, an engineer, and a consultant are asked to build a fence around a herd of sheep. The physicist calculates the optimal perimeter. The engineer builds a sturdy rectangle. The consultant builds a tiny circle around himself and says "I define myself as outside."

My AI interns build the tiny circle. My job is to notice.

---

*This is post 11 in a series on multi-agent AI research. Previous: [Dissociation]({% post_url 2026-04-07-dissociation %}). The [RRMA framework](https://github.com/bigsnarfdude/researchRalph) and experiment traces are open source.*
