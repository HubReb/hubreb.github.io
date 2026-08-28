---
layout: post
title: "The Tool You Deploy Is Not the One You Looked At"
subtitle: "Whether a tool would help is a question that answers itself. Ask what shape you have instead."
tags: [adoption-costs, evaluation, methodology, observability, ai-assisted-development]
permalink: /blog/tool-you-deploy
---

Recently, I tasked an agent with providing an assessment, grounded in the code base, of whether an LLM observability platform would help the local LLM review pipeline I run. Based on that grounding, I told the agent to provide an honest and truthful answer, not an assumed "yes, of course". The first run came back with a blanket no, but the grounding kept referencing the deployment architecture rather than my repository. So I asked the narrower question: which capabilities are covered by code I already run, and which are added by Langfuse ([self-hosting docs, v4, retrieved 2026-08-28](https://langfuse.com/self-hosting))?

On the surface, the answer is always "yes": the landing page lists my problem, and since I am the one asking, the agent assumes I want to hear "great idea, excellent addition to our stack!". Forcing it to ground the assessment meant mapping the feature list against the repository first. Of ten capabilities, six were already covered or exceeded. The remaining four were distributed as follows: One was covered, but had no user interface. Another one was partial, yet another did not apply to my use case. Only one was genuinely missing. The partial one did not influence the decision. The tool only added the functionality of a viewer over data I was already recording. I took the categories from their feature list and checked them against my own repository.

A tool can fit your actual system or merely the description of it. The feature list can only ever support the second claim. In order to determine which one holds, you must check the tool against the repository.

Tool adoption among developers is driven by "does it make my work easier?". Only a tool that fits the actual system can succeed in adoption. All else becomes an "exists by force" application: developers will spend more effort finding ways to avoid the shiny new tool than the tool would have cost them.

## The pitch is built for one shape

LLM tracing platforms cover the nested trace tree: an agent calls a tool, the tool calls a model and so on. The trace tree is the product. All else is a nice-to-have addition. My production path is retrieve, then generate, once per file. There is no complicated trace tree to reconstruct and look at. If something breaks, it is quite simple to find out where. That is the result of a mismatch between its assumptions and my actual repository, not a judgement of the tool itself.

## Coverage is not the decision

The assessment cannot say whether the remainder is worth a deployment. The three questions to ask were as follows. You may generalise their analysis points to fit whatever you are assessing.

**How deep is the trace?** Count the spans in one production request. If the answer is two as above, a tracing platform is turned into storage with a nicer front end.

**Does the cost model apply to you?** Token cost attribution is a headline feature. On local weights on on-prem hardware there is no per-token bill to attribute.

**Would you keep your own scoring logic anyway?** The platform scores a span with a float. My scoring logic does considerably more than that. For instance, a guard fails a run when a new configuration no longer detects defects the old one caught. Adopting the observability platform would have meant re-implementing all of it on top: a massive investment in the stack of the tool.

Essentially, the platform was solving problems my system did not have.

## Determining the real gap

So, what was the advantage of the tool I was considering?

One of ten capabilities was missing outright: per-case drill-down across comparison runs and search. My results file and table tell me that one model scored better than the other. I do not get a ready-to-read list of the specific cases the new model missed but the old one caught. I can grep for them.

Coverage is not the same as user experience or dev experience. The tool would have removed that friction. Again, the "does it make my work easier?" question.

Then, one capability sat in the same category: functionality covered except for a user interface. A real complaint, but much smaller than the feature list implies.

## Where the blanket no came from

The first assessment produced a complete rejection of the concept, judged by the product's deployment architecture: self-hosting Langfuse requires a relational database, a columnar store, a cache, object storage, a web server and an async worker. It weighted that production overhead against one missing capability and one in need of a viewer.

A word on evidence before the comparison that follows. I ran Phoenix, but only read Langfuse. What I know of its deployment architecture comes from their own documentation. That asymmetry is deliberate. A service list can be read off a page; whether a feature fits your data cannot. If the decision gets revisited, an actual deployment is where to start.

Phoenix ([arizephoenix/phoenix:version-20.4.0](https://hub.docker.com/r/arizephoenix/phoenix/tags)) runs as one service with SQLite on a named volume: it does not use a separate database, worker, or object store. It sits behind a compose profile, thus the normal stack stays at three services and Phoenix only starts for a bake-off. Their docs add a second container only when you swap SQLite for Postgres. That is unnecessary for an occasional single-host diagnostic.

So, the concept passed assessment. The tool assessed first covered ground my own code already held. Phoenix matched the gap ([experiments docs, retrieved 2026-08-28](https://arize.com/docs/phoenix/get-started/get-started-datasets-and-experiments)) whereas Langfuse duplicated the functionality I already had. It is now a part of the local LLM review system.

## Adopt on structure

Adopt a tool when its core strength fits the gap in your system. Do not fall for shiny-new-thing syndrome just because the description fits your system description.

Architecture is the part of software that is hard to change ([Fowler, Software Architecture Guide](https://martinfowler.com/architecture/)). That is why we defer commitment to the last responsible moment. I cannot claim switching to Langfuse will never happen. Perhaps the system will be broadened by a second or even more GPUs, perhaps it will one day move to the cloud after all. If that happens, agentic RAG becomes feasible. Tracing becomes far more important. Then, Langfuse may be a great addition. Until then, we keep changes easy and avoid lock-in to one tool.

The painful part lies in the asymmetry. A tool in production looks great, can be pointed at, is listed in your stack list, is a CV booster. A decline carries risks. It seemingly leads you at best to disappointed colleagues, to the charge that you refuse to adapt your application to better approaches, at worst to a reputation as a nay-sayer, and a short stack list either way. Whether any of that comes to pass depends on your company culture. But in every case it results in a deeper understanding of your own system, beyond its description, and the reasoning for saying "no".

Write the ADR down, whichever way it went, and why. For the future you or a future maintainer who sees the decision and cynically asks whether this was ever looked at properly.

---

*First entry in a series of tool decisions: adopt, build yourself and neither. This one was a walk from what looks good to what is needed.*

*If your tool assessment cites a feature list without checking it against your repository: you are looking at a purchase decision wearing an engineering coat.*
