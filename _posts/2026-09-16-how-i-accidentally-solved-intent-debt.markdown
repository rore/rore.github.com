---
layout: post
title: "How I Accidentally Solved Intent Debt*"
category: posts
comments: true
description: I built Agent Workflow to structure the coding step for AI agents, then discovered it was also addressing intent debt — until a real regression exposed what it still missed.
---

(* Yes, it’s clickbait. I didn’t actually solve it. Well, maybe some of it. I think the story is interesting. Read on.)

### It all starts with a need

I did not know what intent debt was. (You might not either. This will come later). I just knew something was missing. I mean - I’ve been doing some side projects in the last couple of months, building tools I was missing for how I wanted to work with agentic development.

One of the things I was missing was some structure around how an agent develops a specific task. There’s already a lot of work around AI-native SDLC (the [Claude Code playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) is a good example). It covers the full flow, from spec to design to plan to code to review and PR. Lots of skills and tools for all those steps. But that part where the agent actually writes the code - I felt that was left a bit open. I wanted the build step itself to have its own mini lifecycle, not just a plan.

### A plan is not enough

The plan shows how the agent intends to implement the change. But I wanted more than that. I wanted to guide the agent through a set of checkpoints to make sure things like discovery and verification are not missed. And since during implementation the agent can discover new things and the plan may change - I didn’t want to lose that. I wanted the constraints, the assumptions, the discovery and deviations from what was originally planned to be noted.

And I also wanted this process to be flexible - lighter when the change is small and routine, broader when it’s larger or risky.

So I created the [Agent-Workflow](https://github.com/rore/agent-workflow) skill to do that. The skill guides the agent through a task-level “mini-SDLC”, and produces a “Work Record” stored in the repo alongside the code, capturing things like the expected outcome, constraints, discovery findings, verification, and changes in plan.

The amount of process scales with risk and complexity. At PR time, CI independently classifies the actual diff and checks the parts of the workflow it can verify structurally.

I have been running with this for a while in my projects, and while it does have its (small) overhead, it aligns the coding step to something I trust (a little) more.

### Then I found out about Intent Debt

I recently came across this interesting paper by Margaret-Anne Storey - “[From Technical Debt to Cognitive and Intent Debt: Rethinking Software Health in the Age of AI](https://arxiv.org/abs/2603.22106)”.

The paper breaks down different categories of debt that become more important with the move to AI-assisted software development, with its increase in velocity and reduction of [friction](https://rore.im/posts/not-all-friction-is-the-same).

We’re used to talking about **technical debt**. But now, with the speed of agentic development, this may actually become easier to manage and reduce. The paper points to two other, more invisible categories of debt: cognitive and intent.

**Cognitive debt** is more talked about already, mainly, I suspect, because we all feel it personally. As velocity grows and agents produce more code, we get detached from the code. We still know what the system does, but it becomes harder to keep knowing *how*. (And should we even try to? This is still an open debate).

But then there’s also **Intent Debt**. This is interesting and a new one for me. The paper says: “*intent debt refers to the absence or erosion of explicit rationale, goals, and constraints that guide how a system evolves*”. It’s when we lose that knowledge of why our system evolved like it did, and the velocity of agentic development can speed this up.

This can happen across the whole SDLC. What struck me is how directly this applies to the build step. The agent can make all kinds of decisions on its own. Or we might align it through the chat - but that disappears with the chat. As the paper continues: “*Intent is best captured at the moment key decisions are made, as recovering it later can be difficult and sometimes impossible*”.

Reading this, I suddenly realized - this is actually a pretty good description of one of the problems I have been trying to solve with Agent Workflow.

The Work Record it produces preserves this change-level intent that would otherwise disappear with the agent session: it records what outcome the task is supposed to achieve, what is in and out of scope, the constraints that must remain true, assumptions discovered along the way, the chosen approach, and how the work should be verified.

For larger or riskier changes it also records discovery, plan review, approvals, implementation notes, and review findings.

So when the agent session disappears, you don’t just have the resulting code and a PR. You also have a durable record of not only what the change was meant to do, but also what happened while it was being built. To me, that directly addresses intent debt.

So you see, I didn’t really solve that whole intent debt thing, but I may have found a way to address one part of it.

### But wait, there’s more

Recently, in another side project that uses Agent Workflow for change management, I came across a regression. Something that used to work suddenly didn’t. I asked the agent to investigate. It found that the regression happened in PR #167. And our Work Record, which is part of the PR, revealed what happened:

During discovery, the agent working on that PR concluded that one of the original behavioral requirements could not be preserved safely with the approach available to it. So it changed the requirement, then also changed the regression test that already protected that behavior to match the new requirement.

Well! That was an interesting catch. It made it clear - **capturing intent is not enough**. The Work Record did its thing - intent was not lost, and we could track what happened and why. But that didn’t prevent the regression itself. I was missing another layer of **enforcement**.

There are two sources of intent here that need to be protected. One is **local** to the task itself: the task defines outcomes and behavioral requirements - those should not be weakened by the agent without stopping and asking for human approval.

And then there’s also the **product intent** - behaviors the product is expected to preserve. In more concrete terms, that means durable behavioral contracts, such as protected regression or acceptance tests, where weakening the behavior they protect requires explicit approval.

Capturing intent is only half the problem. The other half is making clear which intent the agent is allowed to change. For me, this is actually a very natural fit for Agent Workflow. It already captures the intent around the task. What I was missing was enforcement around the parts of that intent that should be protected.

Dogfooding proved useful and surfaced an important feature to add. Coming right up when my weekly tokens are available again.
