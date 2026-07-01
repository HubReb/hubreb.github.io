---
layout: post
title: "Grade Your Evidence, Not Your Confidence"
subtitle: "\"I know\" hides three different claims. Say which one you're making."
tags: [verification, methodology, engineering-culture]
permalink: /blog/grade-your-evidence
---

Earlier I found the one rule that survived a model-offloading experiment: [verify the artifact, never the account](/blog/offload-boundary). The [last post](/blog/feedback-that-changes-behaviour) turned the same rule on instructions: make the expectation checkable, so the work can fail against it. This one turns it on your own findings. Same discipline, third surface: bookkeeping. Trust facts, not emotion.

A technical finding says "I know that the migrated query returns identical results." That single word, know, hides three completely different things. It might mean:

- I ran both versions and diffed the bytes.
- I read the source and it clearly does the same thing.
- I've done this a hundred times and I'm confident.

That's the ambiguity of our language. They carry wildly different certainty, and it collapses them into one word. The reader can't tell which one you meant. Six months later, neither can you, when a decision built on that finding turns out to be wrong and you go back to see what you actually knew.

Confidence is the wrong axis. Emotions do not substitute information. What matters is where the claim comes from: what kind of evidence sits under it. Certainty isn't a property of the claim, it's a property of its provenance. Move the question from "how sure am I" to "what kind of proof do I have" and two things happen. The claim becomes checkable by someone else, without having to trust you. Then the gaps become visible, because "I don't actually know this" turns into a named state instead of a silent omission.

## Three grades, by provenance

So grade every claim by its provenance. Three grades, kept strictly apart:

**Executed.** The system did the thing and I observed the output. I ran it, I saw the result, the bytes matched. This is the only grade that is actually proof.

**Derived from source.** I read the source and concluded it behaves a certain way. I did not watch it run. This looks like proof and is not. "The function obviously coerces the value there" is derived, however obvious it looks. Reading is not running.

**Open.** Neither. Assumption, experience, plausibility. Perfectly legitimate to hold, as long as it's labelled as what it is and not smuggled in wearing the coat of the other two.

Open dressed as executed is its own quiet trap. I hit it in [an eval once](/blog/offload-boundary): a clean-looking pass record that was really an absence of observations, not proof. That is exactly what the grades exist to catch.

Three is enough. I have never needed a fourth, and every extra grade is one nobody uses. But the grade that trips people most often is the middle. Executed is honest by construction and open is honest by admission. Derived is in danger of self-deception: it feels like knowledge and costs nothing to assert. Most overclaiming is a derived reading quietly promoted to an executed fact.

## A grade needs an anchor

A grade on its own is just another claim. So every graded statement carries its source with it. Something concrete a third party can follow to check the grade without asking me to vouch for it: a file and line, a test-run identifier, or best of all a commit. The grade is evaluable precisely because the anchor makes it checkable. Without the anchor, "executed" is a word I typed. With it, anyone can walk to the exact place and see whether the word holds.

## The payoff is catching yourself, early

The payoff isn't the tidy document afterwards. When you are forced to put a grade on every line, you cannot lie to yourself by accident. The moment you reach for "executed" and your hand stops because you only actually read it, that is the method working. It catches the overclaim at the source, in your own head, before it becomes a finding someone else builds on. This prevented self-deception is the point.

## When it's worth the overhead

The cost of the overhead isn't always worth it. Work out what rests on the claim, and think ahead: use your past experience, your own habits, your peers' habits, your company. A throwaway prototype? Probably overhead. Now, a "throwaway" prototype in an organisation where throwaway prototypes have a habit of quietly becoming the MVP, then the product nobody agreed to ship? A different question entirely, because now other people build on it and your later self has to trust it. The discipline is a tool, not a virtue. Applying it everywhere is its own kind of flexing.

The question a finding has to answer is "how do I prove each part, and can someone check that on their own, without a 'trust me, I know'?"

---

*Part of a loose series on checkable certainty. The [last post](/blog/feedback-that-changes-behaviour) turned the same rule on feedback and agent instructions.*

*Grade your evidence by where it came from, not by how sure you feel. A finding whose certainty grades are implicit is an opinion with footnotes.*
