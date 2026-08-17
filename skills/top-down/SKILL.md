---
name: top-down
description: Walks a change down from language to product to mechanism, locks each level, writes the slice, and updates CONTEXT.md and ADRs as terms and trade-offs lock. Grills only unlocked decisions. Writes no plan document. Use when the user wants to start from the top, walk a feature down, avoid plan-then-implement, keep the glossary in sync, use grill-with-docs during implementation, or says /top-down.
---

# Top-down

Walk down. Lock. Write only that slice. Next decision. No plan document.
One level per question. One level per artifact.

## Levels (strict order)

Start at the highest unlocked level. Read the codebase first. Use existing CONTEXT.md terms. If CONTEXT-MAP.md exists, follow it. Also use a CONTEXT.md at the root of the unit you are in. Do not re-ask a lock the repo already holds.

1. **Language and boundary** — what it is and is not. Artifact: the term and its edge.
2. **Product lock** — what the user sees or the system guarantees. Artifact: the guarantee.
3. **Mechanism lock** — **Place, then write.** Pick the layer before the first line. One function, one layer. A layer does not take another layer's questions. Lock how the product lock holds when nobody is acting. Machinery is not implied by the product lock. Artifact: the place.
4. **Implement** — **Reuse first.** Search the codebase for the existing rule; call it. Duplicate only with a named reason. Proximity is not a reason. If two implementations both satisfy the locks, ask. Do not extract a wrapper that restates an existing operation. Artifact: the code.

When a higher lock changes, stop. Drop the in-flight work that only existed to serve the abandoned decision.

## How to talk

Interview until the current lock is shared. Walk one decision. Resolve it. Then the next.

- One question at a time. Recommend an answer. Wait.
- If the codebase can answer, explore instead of asking.
- Speak at the current level only.
- Ask only when a lock is still open. A higher lock plus the code often closes the lower one — write it.
- After a lock, write that level's artifact. Then the next level of this slice. Do not start an independent slice until this one is implemented.

## Docs

CONTEXT.md is a glossary, not a spec.

Challenge glossary conflicts as they happen: "Your glossary defines 'cancellation' as X, but you seem to mean Y — which is it?" Sharpen vague or overloaded terms. Stress-test relationships with concrete scenarios. If the user states how something works and the code disagrees, surface it.

When a term resolves, update CONTEXT.md immediately. Do not batch. Follow [CONTEXT-FORMAT.md](CONTEXT-FORMAT.md) for layout and file placement. Create files lazily.

Offer an ADR only when all three conditions in [ADR-FORMAT.md](ADR-FORMAT.md) hold. Create `docs/adr/` lazily.
