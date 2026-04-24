# Claude Code Session Rituals — unicorn-audit

This repo is Angela's audit trail. Sessions opened here are for running
audits and producing audit documents — **not** for editing the codebase
or KB directly.

If editing `unicorn/` or `unicorn-kb/` is needed mid-audit, change
working directory to that repo, make the edits there with their own
commit, then return here to record the SHA in the audit doc. Never
co-mingle repos in a single commit.

---

## Session start

Before doing anything else:

1. Read the most recent file in `audits/` — what was the last
   reconciliation? Anything still open?
2. Check `unicorn-kb/pinned/kb-hygiene.md → Freshness policy` mentally:
   have any shelf-life dates passed in the last 30 days? If yes, surface
   them.
3. Check `unicorn-kb/pinned/decisions.md → Open` — any decisions older
   than 90 days? Surface them.
4. State the audit scope for this session before any other work.

---

## During session

- Audit docs live in `audits/` — one file per audit, named
  `YYYY-MM-DD-<slug>.md`, follow the template in `README.md`.
- Codebase observations are **read-only** from this session. If a code
  change is needed, that's a separate session in `unicorn/` or `unicorn-kb/`.
- KB changes happen in `unicorn-kb/` as their own commits. Record the
  SHA in the audit doc; do not duplicate the KB change here.
- Never co-mingle audit narrative and KB content — the audit references
  the KB by SHA, doesn't restate it.

---

## Session end

1. Audit doc is complete per template — no empty sections; "n/a" is a
   valid value but empty isn't.
2. Every "KB changes shipped" line has a real SHA.
3. Every "Codebase observations" line has a real SHA.
4. Commit on a branch named `audit/YYYY-MM-DD-<slug>`.
5. Merge the branch into `main`.
6. Tag the merge commit: `audit-YYYY-MM-DD` (or `remix-YYYY-MM-DD` for
   remixes).
7. Append the audit to `INDEX.md`.
8. Push commits + tags: `git push && git push --tags`.
9. Surface any KB-worthy insights from this session that haven't yet
   landed in `unicorn-kb/` — draft ready-to-paste text per the "Push
   back and recommend an update" rule in `unicorn-kb/pinned/kb-hygiene.md`.

---

## Source precedence inside this repo

When reasoning during an audit:
1. Actual codebase at `../unicorn/` via filesystem read.
2. `../unicorn-kb/` via filesystem read.
3. Previous audit docs in `audits/` for historical context.
4. Inference — flag with "Inferring from …".

When the codebase and KB disagree, the codebase wins and the divergence
is the finding.
