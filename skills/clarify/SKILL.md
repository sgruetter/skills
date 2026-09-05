---
name: clarify
description: Edit local code so a reader who wasn't here can follow it, without changing what it does. Use when the user says clarify, /clarify, or points at a file, function, hunk, or uncommitted session diff that's hard to follow. Not for architecture, TDD, formatting, or prose.
argument-hint: "file, function, or hunk"
---

# Clarify

Make this readable to someone who wasn't here. Don't change what it does — including the ugly cases and what callers depend on.

Scope: what the user pointed at, else the uncommitted diff. Already clear → leave it. Unreadable because it belongs somewhere else → stop.

A function reads as the steps it takes. Name anything you'd otherwise have to remember. Extract only when the name stands alone as a concept. Comment only if a name still leaves it unclear — why, traps, invariants, never a restatement. Similar-looking blocks that would change for different reasons are not duplication.

This file is the style guide. When that fights clarity: keep how it talks, still make the story obvious.

No tests is not a skip. Don't ask. Don't weaken tests. Cheap coverage exists → run it; fail → revert. Recap what now reads, not what you renamed.
