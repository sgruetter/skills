---
name: diff-triage
description: Orders uncommitted changes for human review, highest-stakes first, and flags which low-risk changes can probably ship without human review. Use when the user wants a review order, a manual review queue, diff-triage, or to know which local changes can skip human review.
---

# Diff-triage

Read the highest-stakes, hardest-to-judge changes first. Later files get less attention.

Queue only. Not findings, not approval.

## 1. Collect

Git repo required. Staged + unstaged + untracked:

- `git status --porcelain`
- `git diff HEAD`
- `git ls-files --others --exclude-standard` — read those files

Stop if clean. Classify from the **diff**, not the path. Ignore generated/binary noise unless that file is the change. If this session wrote the diffs, mark **self-authored**. Huge diffs: sample hunks, lower confidence, say so.

## 2. Classify the change

Each path gets the **highest** tier any hunk reaches. Tag mixed. Path is a hint.

| Tier | The change itself |
|------|-------------------|
| **domain** | Central business rules with domain complexity: invariants, calculations, eligibility, state machines, money, identity-as-rule |
| **security** | Authn, authz, secrets, crypto, injection, PII, session, permission checks |
| **boundary** | Trust or process edge: public API, events, schema, network, third-party, untrusted parse, process spawn |
| **supporting** | Tests, types, adapters, wiring that exist only so a higher-tier change compiles, runs, or can be verified |
| **generic** | Same-shape, no behavior: rename, format, comment-only, generated identical output |
| **plumbing** | IO, logging, retries, config load, serialization, filesystem — no rule, security property, or contract change |

Two tiers → higher one. Domain+security → tag both; still first-attention. Untrusted IO is **security**, not plumbing.

## 3. Cluster, then order

Walking all domain, then all security, splits a slice.

1. Cluster files of one slice (rule + boundary + its tests). Name the slice in the project's terms (CONTEXT.md if present).
2. Order clusters by highest tier: domain > security > boundary > supporting (orphans) > generic > plumbing.
3. Inside a cluster: domain, security, boundary, then that cluster's supporting.
4. Orphans sit in their own tier.

Warn if more than ~20 files.

## 4. Independent risk vet

Spawn one `explore` subagent and wait. Put diffs and paths in the prompt or a file it can read — not your tiers or skip guesses. Do not ask it to run git.

It must assume human review is required. Return per change: **high-risk** or **low-risk** (one-line why); for low-risk, whether it can **probably ship without human review** (one-line why). Conservative on mixed files, self-authored code, and the never-skip list.

Subagent fail or disagreement → needs-human.

## 5. Never skip

Authn/authz/session, crypto/secrets, money, permissions, PII, schema/migrations, public API, concurrency/locking, default-on flags, untrusted input, or subagent uncertainty.

## 6. Skip bar

**Probably skippable** only if all of:

- Highest tier is **generic**, or **plumbing** that is comments / log-text / formatting only
- Subagent says low-risk and skip-ok
- No never-skip hit
- Not mixed with a higher tier

Say "probably skippable". The human still owns the merge.

## Output

```markdown
# Diff-triage

Self-authored: yes/no
Files: N (warn if >20)

## Read in this order
1. **domain** `path` — why this tier; mixed if so
   cluster: <slice> — also `a`, `b`
2. …

## High-risk
- `path` — why

## Low-risk
- `path` — why

## Probably skippable without human review
- `path` — why it clears the skip bar
(or none)

## Keep for human review
- `path` — tier / never-skip / subagent / mixed
```
