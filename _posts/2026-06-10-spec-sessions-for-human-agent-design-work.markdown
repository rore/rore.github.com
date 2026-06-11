---
layout: post
title: "Spec Sessions for Human-Agent Design Work"
category: posts
comments: true
description: Spec Sessions in Minimap make iterative spec and design work with agents visible, reviewable, and easier to follow across multiple rounds of comments and edits.
---
When I work on specs or designs with agents, it usually becomes a collaborative process.

I discuss what I want with the agent until we reach a point where I feel we can lay out the design. Then the agent writes the spec, and I do a couple of iterations of comments and amendments. Sometimes I want a second opinion, so I let Codex and Claude review each other's suggestions until we get to a better version.

This works OK, but it requires a lot of copying and pasting from agent to agent. I wanted something more collaborative and more human-friendly. I wanted to be able to review the spec, comment on it, let the agents comment and reply, and apply edit suggestions, so that the whole process is visible and easier for me to follow.

Since I already have my human-agent collaborative UI for project roadmaps, I added this as another capability, called <a href="https://github.com/rore/minimap#spec-sessions" target="_blank" rel="noopener noreferrer">Spec Sessions</a>.

It's a combination of a skill for the agent and a UI for the human. It defines a workflow for working on specs, or any .md file for that matter. The agent attaches the file to a Minimap spec session, agents can comment on it and on other comments, and the human, in this case me, can see and join the conversation through the UI.

For specs that require multiple iterations with multiple agents, this makes the process much more visible, and also more fun to work on.
