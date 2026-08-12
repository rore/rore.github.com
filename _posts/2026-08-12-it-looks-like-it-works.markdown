---
layout: post
title: "It Looks Like It Works"
category: posts
comments: true
description: AI agents are good at producing work that looks successful, but critical software demands careful review of what their output actually proves.
---
You would think this would be a simple task for an AI agent:

> “I have a document from our PM describing E2E scenarios for a feature in our product. Turn these scenarios into E2E tests in our test repository, according to its conventions.”

And you would be wrong.

Claude, or any other agent, will produce a nice set of tests and report back with “great success!”. But if you look a bit deeper (or just ask it to review its own work), you will see that many of the tests are not really testing what the document describes, or they assert the wrong thing, or they just echo back some value and mark it as a success.

I have been fighting with this for a while, including building a skill that requires the agent to produce a lineage of intermediate artifacts that can be structurally validated, thus forcing it to be “honest” about what it does and accurately report what is and isn’t running. This tactic helps improve the quality of what the agent produces, but it still takes many cycles of review and fixing to get to a reasonable state.

The point is that agents are quite good at making you think they did a good job. It looks like it works! But the details, oh, the details. And I see this gap in understanding in how people are working with AI.

When you’re building something that is a pet project or a hobby, it’s good enough. It seems to basically work, and that is what’s important. But if you ask developers who are responsible for critical production services, they will be much more reluctant to trust the agent’s output, and for good reason: they are aware of the gap between “it looks like it works” and what’s actually inside.

Every technology has limitations, but I think this is the first time we’ve had a technology that can actually gaslight you into thinking it is doing a good job. So while it’s easy to trust the machine, don’t ignore those developers urging you to be more critical of what it outputs.
