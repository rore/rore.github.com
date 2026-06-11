---
layout: post
title: "agent-redline: Red Zones for Agentic Development"
category: posts
comments: true
description: agent-redline is a new skill I created for agent governance. It makes coding agents classify architectural risk before editing, then verifies the same policy on PRs.
---
I've been thinking quite a lot about AI-driven agentic development and what it takes to make it safer when developing sensitive products.

I keep coming back to the "cage" idea. There are places where agents can roam free, and there are places that are sensitive and require deeper review. And human attention is scarce, especially when agent PRs become larger and more frequent.

So how do we draw attention to the right things at the right moment?

My first iteration around that was <a href="https://github.com/rore/bear-cli" target="_blank" rel="noopener noreferrer">BEAR</a>, a governance CLI that uses an intermediate language to define boundaries and creates project scaffolding that allows boundary enforcement. I still think this is a worthy concept, and I actually believe that at some point, a similar idea will become an inherent part of development platforms or runtime. But for now, this concept may be too intrusive to adopt.

But can it be simpler? That was my next attempt at this problem. What if we define areas, files or locations, where agents can develop freely, let's call them Blue Zones, areas that are riskier and require attention, Red Zones, and use existing architectural tests to enforce boundary rules?

Then a skill can tell the agent how to work with these definitions and when a human should be involved, and a simple CI script can enforce this in PRs and require human attention when needed.

This is the concept behind <a href="https://github.com/rore/agent-redline" target="_blank" rel="noopener noreferrer">agent-redline</a>. It currently supports Java and Python, and other languages and stacks can be added as extensions to the skill. I'm trying this now on some of my projects and am curious to see if this really helps filter out the changes that require more attention.
