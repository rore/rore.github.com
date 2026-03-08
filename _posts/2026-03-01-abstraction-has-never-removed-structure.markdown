---
layout: post
title: Abstraction Has Never Removed Structure
category: posts
comments: true
description: Every programming abstraction shift increased expressiveness but also introduced new structure, and natural language software development will be no different.
---
When we look at the history of programming language revolutions, there's a consistent pattern. Every major shift in programming increased expressiveness, but it also introduced new structure and constraints. Each shift created a new level of abstraction. We could express what we wanted more clearly, with a larger vocabulary, but always under formal rules.

But this shift feels different. For the first time, our abstraction layer is natural language. In previous transitions, we introduced new programming languages to support the new abstraction, but they remained formal and deterministic. Natural language is the most expressive language we have, yet it is not formal or deterministic. It is open-ended and interpretive. That changes the nature of the abstraction itself.

Natural language is a powerful tool. The fact that you can describe what you want, almost as you would to another person, and see it built, is a real leap. It works especially well in contexts where ambiguity is acceptable and iteration is fast. But in systems that require precision and high confidence, ambiguity can become a liability.

I think we may see a divergence in how language is used for software development. For many projects, natural language may be sufficient. But history suggests that when expressiveness rises, constraint follows. In high-precision systems, structure will need to reappear.

I see two levels where this might happen:

First, the specification layer. We may need a more formal representation of intent. Not just free-form requirements, but structured ways to describe what a system must do, or to review how natural language was translated.

Second, the structural boundary layer. As code generation becomes more fluid and high velocity, boundaries matter more. We will need clearer definitions of what parts of a system are allowed to access or modify other parts, so critical behavior does not change without being visible.

Abstraction has never removed structure. Structure needs to reappear. The question is where.