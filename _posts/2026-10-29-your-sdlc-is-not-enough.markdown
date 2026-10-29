---
layout: post
title: "Your SDLC is not enough"
category: posts
comments: true
description: Examples of how durable Work Records and workflow checks help agents preserve intent, review risk, and build on earlier work.
---

**tl;dr:** Agents already use a common SDLC: plan, implement, review, test. But is it enough? I collected a couple of examples showing how durable Work Records and workflow checks improve the standard SDLC.

---

In [my previous post](https://rore.im/posts/how-i-accidentally-solved-intent-debt), I wrote about Agent Workflow and how it helps address intent debt. This is a workflow that guides the agent through development and keeps a durable Work Record to capture the decisions and changes made during implementation: what we wanted, what we discovered, what changed, and what we actually checked.

But there’s a fair question I also ask myself. If the agent already follows a regular SDLC, writing a plan, running tests, and getting another agent to review its work, what does this additional workflow add?

I’ve been using Agent Workflow in a couple of my projects, and I collected a few cases where it helped. What stood out is that it was especially useful when the work didn’t go quite as planned.

## How risky is this change really?

One change in [Pallium](https://github.com/rore/Pallium/pull/131) dealt with an area that the agent treated as moderately risky.

But that change also touched a part of the program that the repository’s rules classified as high risk. The automatic check looked at the files that actually changed and rejected the lower classification.

This is a fairly simple check. It doesn’t understand whether the implementation is good. It knows that we decided certain parts of the program deserved more attention, and this change touched one of them.

Since the risk classification of the change was elevated, the process required a more thorough review, beyond the one that was already done. That additional review found another bug and gaps in test coverage, which were fixed before the merge.

What the workflow added was a check against an earlier agreement about risk, and a place where the additional review and its findings remained part of the work.

## Is this what we really wanted to do?

Another example came from [Agent Workflow itself](https://github.com/rore/agent-workflow/blob/main/.agent-workflow/tasks/work-record-source-contract.md).

During planning, the agent proposed changing an original requirement and classified the change as equivalent. Because the Work Record preserved the original requirement, a reviewer caught the difference. The original requirement was restored in the record, and implementation stayed blocked until I explicitly approved the change.

The workflow didn’t make the judgment for us. It preserved the earlier agreement, surfaced the disagreement, and let us make an explicit decision.

## Which problem did we actually fix?

During one task, the agent made several useful fixes and the tests passed. But the original problem was still there. The Work Record kept the original problem visible despite those successful checks. We went back to the actual problem and verified it worked.

## Do we have to start over every time?

This can sound like extra work. Sometimes it is. But a Work Record can show which parts of the work have already been checked, so we don’t have to repeat things unnecessarily.

In [one Agent Workflow change](https://github.com/rore/agent-workflow/pull/48), earlier review remained useful when a follow-up changed part of the implementation. The unchanged work did not need to be reviewed from scratch, while the new change received focused attention.

A newer [Minimap update](https://github.com/rore/minimap/pull/70) made the same point more clearly. It reused earlier reviews and checks for work that had already been examined, then verified that the copied work fit correctly in its new context.

This wasn’t simply a matter of trusting that “someone already tested it.” The record showed what had already been checked and what still needed attention. That gave the next agent something concrete to work from.

## Isn’t this just good engineering practice?

Mostly, yes.

Planning, reviewing, preserving requirements, testing the real outcome, and knowing when previous evidence still applies are all things we already know we should do.

These problems aren’t unique to agents. But when work moves between agent sessions, we can’t rely on chat history and session context alone. I wanted a durable record to keep the earlier decisions and unfinished checks visible. The original requirement should still matter after the plan changes. An unresolved problem should still be tackled even after some tests pass. A completed review should still be useful when the next change arrives.

I can’t say for certain whether Agent Workflow helps produce fewer bugs. But these examples bring me back to intent debt. A plan, a review and passing tests don’t necessarily preserve why a decision was made and what changed along the way. Those things can be part of a good SDLC, but just following the stages doesn’t make them survive the next agent session. That’s what I wanted the Work Record to add. Agent Workflow may not be a perfect solution, but it makes it easier for me to trust the process.
