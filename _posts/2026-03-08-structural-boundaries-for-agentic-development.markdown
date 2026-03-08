---
layout: post
title: "BEAR: Structural Boundaries for Agentic Development"
category: posts
comments: true
description: BEAR is an experiment in enforcing explicit architectural boundaries so agentic development can move fast without hiding structural changes.
---
In the last few posts I've been thinking about how development might change when agents can write large amounts of code very quickly. If generating implementation becomes easy, the constraint in development shifts. The interesting question is no longer how we produce code, but where control should live when a system can evolve at that speed.

One thing that kept coming to mind is that many of the most dangerous changes in a system are not ordinary code edits. They are structural changes. Accessing something new, dependency changes, side effects. These are important architectural changes, but they often get lost in a pull request, especially when an agent can introduce many changes in the same PR.

That made me wonder what would happen if the important architectural structure of a system was described explicitly at a level above the generated code.

I started thinking about describing a system as a set of blocks. Each block represents an architectural unit with clearly defined boundaries. It declares what it can access, what it exposes, and what dependencies it is allowed to have. Inside those boundaries, development can move quickly. Agents can generate code, refactor it, and evolve it freely, but the boundaries themselves remain visible and enforceable.

It's a bit like putting a bear in a cage. Inside the cage the bear can move however it wants, but the cage still defines the limits of where it can go.

![BEAR logo](/images/bear-header.png)

I started experimenting with this idea in a small project called BEAR - Block Enforceable Architectural Representation. The idea is that an agent starts from a small representation of the application as a set of blocks, and then works within those boundaries. BEAR CLI generates deterministic checks from it, and CI can use BEAR to surface structural changes. If a pull request expands the authority of a component, that expansion becomes visible rather than hidden inside ordinary code changes, and a human-in-the-loop review can be triggered.

This is still very much an experiment, but the project and how it works are available here:
[BEAR on GitHub](https://github.com/rore/bear-cli)

As development velocity continues to increase, it seems likely that some form of explicit structural boundary layer will become necessary.

BEAR is a small attempt to explore what that layer might look like.
