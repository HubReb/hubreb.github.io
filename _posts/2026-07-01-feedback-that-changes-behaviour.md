---
layout: post
title: "Feedback That Changes Behaviour Is Feedback You Can Check"
subtitle: "The same property that makes feedback land on people makes an agent follow an instruction"
tags: [feedback, ai-assisted-development, agents, engineering-culture]
permalink: /blog/feedback-that-changes-behaviour
---

## We're taught to soften feedback so it lands. That's the part that breaks it.

Most feedback changes nothing. It gets nodded at, defended against, or ignored. The feedback that actually moves someone has two properties: the recipient can check whether it's true, and can see the specific thing to change.

**Evaluable.** Tie the feedback to observable behaviour, give an example and highlight the concrete effects. "This PR merged with the suite red, and it took down staging on Tuesday" is something the other person can verify. "You're careless" is not — it's absolute, a judgement. The recipient can only accept it on authority or dismiss it as opinion.

**Actionable.** The example points at the thing to change: red test before merge, so add the gate. That's a move. "You're careless" contains no move.

Character judgements fail both tests. You can't verify "sloppy" and you can't act on it. So they produce one of two things — compliance, where the person submits without knowing what to do differently, or resistance, where they defend the self you just attacked. Neither helps anyone. Improvement needs something concrete to aim at.

## The same rule governs agent instructions

The same property decides whether an agent follows an instruction or drifts from it.

Write "follow our architecture standards" into a markdown file and the agent produces something that looks compliant and often isn't. The instruction isn't evaluable — there's no check the output can fail, so nothing catches the gap. The agent hands you a confident account of having followed it. The account doesn't survive inspection.

Write the standard as something checkable instead: a test, a type, a schema, a contract. Now the output can fail against it in CI, before anyone reviews. The instruction is evaluable — the check passes or it doesn't — and actionable, because a failure says exactly what to fix.

Humans and agents fail the vague version the same way. Both read "be careful" or "follow best practices," both produce plausible output, and both leave you no way to tell whether the instruction was applied. Reading isn't complying. Enable complying: make the expectation something the work can be checked against.

## Delivery

Dictation is one-directional. You hand down the verdict and the other side has no room to respond. Even if you truly believe the verdict is correct, a handed-down judgement is hard to swallow — the person didn't arrive at it, and people defend against what they're handed.

Engagement gives the other side something to reason about. You provide the example and the consequence; they check it, they see the move, and because they walked the last step themselves, they own the change instead of resenting it. For those interested, there's an entire library on this — psychological safety, SBI, and so on — with far better examples than I could give here.

For agents, it is not engagement that produces results. Checkable means it can — and must — be enforced. Failure to comply has consequences.

## One discipline

This goes both ways. The structure that makes feedback land is the same one that makes it easier to take and to give: observable behaviour, an example, a checkable expectation, offered as dialogue. Tie criticism to something checkable and you can evaluate it instead of submitting to it or bristling at it. It holds on people and on agents alike — a vague instruction can't be checked, so it can't reliably change anything.

Strangely, we tell people that to have their feedback met with open ears, they should wrap it in "I think", "I consider" or "My impression is." That's the wrong layer. The library above would tell you the same: name the behaviour and the impact, don't dress up the verdict. On agents, that backfires spectacularly: "I think" can be happily discarded. Personally, my impression is that "My impression is you are careless" backfires on humans, too — most of the time less visible, but far more damaging in the long run.

---

*We're taught to make feedback easy to say. The feedback that changes anything is easy to check and hardest to give: a behaviour, an example, an effect the other side can verify. Softening doesn't make it land — it makes it optional. Agents drop the optional. People keep it and slowly stop hearing it.*
