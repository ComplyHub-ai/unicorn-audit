# Unicorn Audit Trail

> Carl's narrative record of reconciliations between the codebase, the
> KB, and reality. Not visible to the team. Not synced anywhere automatic.
>
> This repo exists to give "on my terms" commit history and tags for
> moments when something was audited, reconciled, or consciously decided —
> without depending on what Lovable, Supabase, or anyone else captured.

---

## What lives here

```
unicorn-audit/
├── README.md              ← this file
├── CLAUDE.md              ← session rituals for Claude Code in this repo
├── INDEX.md               ← chronological list of audits
└── audits/
    └── YYYY-MM-DD-<slug>.md
```

---

## When to create an audit doc

**Always:**
- After a Lovable remix (see
  `../unicorn-kb/handoffs/post-lovable-remix.md`).

**Sometimes (your judgement):**
- Resolving an Open Decision that's been open > 90 days.
- Reconciling a divergence between pinned KB and shipped code.
- Shipping a material convention change across multiple files.
- After a session that surfaced something future-you will want to find
  via `git log --grep="audit:"`.

**Never:**
- Routine PRs.
- Shelf-life re-dates.
- Typo fixes.
- Normal feature work.

Audits are ad-hoc — no calendar, no quota. Too many audit docs and the
signal drowns; too few and the point is lost.

---

## Template

Create `audits/YYYY-MM-DD-<slug>.md` with this shape:

```markdown
# Audit: YYYY-MM-DD — <slug>

**Trigger:** scheduled / post-remix / ADR-driven / drift-surfaced / ad-hoc
**Scope:** what you looked at, what you didn't

## Findings
- Concrete, one bullet per finding.
- Note discrepancies between KB and codebase.

## KB changes shipped
- unicorn-kb @ <commit-sha>: brief description
- (or "no changes" if read-only audit)

## Codebase observations (read-only)
- unicorn @ <commit-sha>: what state it was in
- (skip if nothing to observe)

## Decisions
- ADR-NNN drafted / resolved / superseded
- Brainstorm entries promoted/archived

## Open questions parked
- Things you noticed but didn't action this time

## Tag
audit-YYYY-MM-DD  (or remix-YYYY-MM-DD for remixes)
```

---

## Commit + tag conventions

**Branch:** `audit/YYYY-MM-DD-<slug>` (merge to `main`).

**Commit message:**
```
audit(<scope>): <short summary>

- Bullet finding
- Bullet finding

KB: unicorn-kb@<sha> (if KB changed)
Code: unicorn@<sha> (always — the SHA at time of audit)
Tag: audit-YYYY-MM-DD
```

**Tag format:**
- `audit-YYYY-MM-DD` — general audits.
- `remix-YYYY-MM-DD` — post-Lovable-remix reconciliations specifically.
- `adr-NNN` — point-in-time of a specific ADR resolution (optional).

Tag on `main` after the merge commit.

---

## Retrieval

- All audits: `git log --grep="audit:" --oneline`
- All remixes: `git tag --list "remix-*"`
- Latest audit: `ls -t audits/ | head -1`
- Search for a topic: `git log --grep="<keyword>" --all`

INDEX.md is a human-readable chronological list; keep it updated as a
backup to git log.
