---
name: diff-triage
description: Orders uncommitted changes per git repo for human review, highest-stakes first, and flags which low-risk changes can probably ship without human review. Use when the user wants a review order, a manual review queue, diff-triage, or to know which local changes can skip human review — including across more than one repo.
---

# Diff-triage

Read the highest-stakes, hardest-to-judge changes first. Later files get less attention.

Queue only. Not findings, not approval. **One git work tree = one subagent = one report.** Never merge repos into one list.

## 1. Discover (orchestrator)

Every git work tree under the workspace (cwd, workspace roots, nested checkouts, submodules). Deduplicate by toplevel. Keep trees with staged, unstaged, or untracked changes. Omit clean trees. If none, stop.

## 2. One subagent per repo (orchestrator)

Spawn one `general-purpose` subagent per dirty tree, `cwd` = that toplevel, in parallel. Point it at this SKILL.md. It runs Collect through Skip bar for **that tree only** and returns one compact report. It must not discover other repos, spawn further subagents, or rank across repos.

Print reports in toplevel-path order. Subagent fail → that repo is needs-human; do not fold it into another repo's list.

## 3. Collect

Staged + unstaged + untracked:

- `git status --porcelain`
- `git diff HEAD`
- `git ls-files --others --exclude-standard` — read those files

Classify from the **diff**, not the path. Ignore generated/binary noise unless that file is the change. If this session wrote the diffs, mark **self-authored**. Huge diffs: sample hunks, lower confidence, say so. Paths relative to this toplevel.

## 4. Classify the change

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

## 5. Cluster, then order

Walking all domain, then all security, splits a slice.

1. Cluster files of one slice (rule + boundary + its tests). Name the slice in the project's terms (CONTEXT.md if present).
2. Order clusters by highest tier: domain > security > boundary > supporting (orphans) > generic > plumbing.
3. Inside a cluster: domain, security, boundary. Put each file's test immediately after that file. Other supporting (types, adapters, unpaired tests) after the paired files.
4. Orphans sit in their own tier.

Warn if more than ~20 files (`>20` in the heading).

## 6. Risk and skip

Assume human review is required. Label each change **HIGH** or **LOW**. Conservative on mixed files, self-authored code, and the never-skip list. Uncertainty → HIGH.

**Never skip:** authn/authz/session, crypto/secrets, money, permissions, PII, schema/migrations, public API, concurrency/locking, default-on flags, untrusted input, or uncertainty.

**Probably skippable** only if all of: highest tier is **generic**, or **plumbing** that is comments / log-text / formatting only; LOW; no never-skip hit; not mixed with a higher tier.

Say "skip", not approved. The human still owns the merge.

## Output (one report per repo)

One slice heading, then a file list you can copy. No extra High-risk / Low-risk / Keep sections. Each file: `filename · path` with path relative to repo root. No backticks.

```markdown
# <repo-dir> · N files · self-authored? · >20?

pricing  domain HIGH mixed
- pricing.ts · src/pricing.ts
- pricing.test.ts · tests/pricing.test.ts
- refunds.ts · src/api/refunds.ts
- refunds.test.ts · tests/refunds.test.ts

session  security HIGH
- auth.ts · src/auth.ts

Skip
- README.md · README.md
- typo.md · docs/typo.md
```

`Skip` with `- none` when empty. Heading uses the repo directory name.
