---
layout: post
title: Not All Friction Is the Same
category: posts
comments: true
description: As AI removes the friction of writing code, engineering teams will need new intentional friction to preserve visibility, control, safety, and confidence.
---
Lately there are many posts celebrating the death of friction, praising how AI removes the friction of writing code and increases development velocity.

But is friction always bad?

From my current personal perspective, for example, the friction a shelter wall provides when a rocket hits it is good friction.

But let's take a less sensitive example.

When Git came along, it brought a real change to how we manage source code. Anyone could create branches locally. Branching became trivial and cheap, merging became easy. Multiple developers could work on the same code in parallel. A lot of friction that central version control systems used to impose was removed. But that also increased the velocity at which changes could reach the main codebase.

To regain visibility and control over those changes, we introduced a new intentional point of friction: Pull Requests.

With PRs, a change becomes visible before it is merged. Someone else reviews it. We can discuss it. CI gates can run. PRs deliberately slow the final step so that we regain visibility and control.

So not all friction is the same.

Some friction slows us down because of effort: writing boilerplate code, setting up environments, repetitive work. Removing this friction improves velocity.

But there is also friction that we introduce intentionally to maintain control, safety, or correctness. Things like PRs, deployment gates, controlled releases, feature flags, and even type checks.

The interesting thing is that when "bad" friction disappears and velocity increases, we often need to introduce new "good" friction so things don't get out of control.

And that's what we're witnessing now with AI. The friction of writing code is being reduced dramatically, and development velocity increases accordingly. But the risks don't disappear. So we now need to find ways to introduce new and refined friction that allows us to move at this new speed with regained visibility and confidence.