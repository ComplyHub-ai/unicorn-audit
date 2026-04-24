# Claude Code Session Rituals — unicorn-audit

This repo is Angela's audit trail. Sessions opened here are for running
audits and producing audit documents — **not** for editing the codebase
or KB directly.

If editing `<codebase>/` or `unicorn-kb/` is needed mid-audit, change
working directory to that repo, make the edits there with their own
commit, then return here to record the SHA in the audit doc. Never
co-mingle repos in a single commit.

`<codebase>` resolves per the workspace root CLAUDE.md at
`~/code/unicorn-workspace/CLAUDE.md`.

---

## Session start

Before doing anything else:

1. **Pull updates:** `git pull --ff-only`. If this fails (conflict,
   divergence, dirty working tree), STOP and report. Do not attempt to
   resolve conflicts autonomously.
2. Read the most recent file in `audits/` — what was the last
   reconciliation? Anything still open?
3. Check `../unicorn-kb/pinned/kb-hygiene.md → Freshness policy`
   mentally: have any shelf-life dates passed in the last 30 days? If
   yes, surface them.
4. Check `../unicorn-kb/pinned/decisions.md → Open` — any decisions
   older than 90 days? Surface them.
5. State the audit scope for this session before any other work.

---

## During session

- Audit docs live in `audits/` — one file per audit, named
  `YYYY-MM-DD-<slug>.md`, follow the template in `README.md`.
- Codebase observations are **read-only** from this session. If a code
  change is needed, that's a separate session in `<codebase>/` or
  `unicorn-kb/`.
- KB changes happen in `../unicorn-kb/` as their own commits. Record
  the SHA in the audit doc; do not duplicate the KB change here.
- Never co-mingle audit narrative and KB content — the audit references
  the KB by SHA, doesn't restate it.
- `<codebase>/` is **read-only from Claude Code**. No commits, no edits,
  no `git add`. Fetch is allowed; pull only when explicitly requested.
  See workspace root CLAUDE.md for the full rule.

---

## Write permissions

- `unicorn-audit/`: commit + push to feature branches autonomously.
  Branch naming: `audit/YYYY-MM-DD-<slug>` or `remix/YYYY-MM-DD-<slug>`.
- Never push to `main`. Never force-push. Never delete branches or tags
  without explicit confirmation. Never amend pushed commits.

---

## Session end

Before ending a session that made changes:

1. Audit doc is complete per template — no empty sections; "n/a" is a
   valid value but empty isn't.
2. Every "KB changes shipped" line has a real SHA.
3. Every "Codebase observations" line has a real SHA.
4. Commit on the feature branch with a conventional-commits message
   (`audit:` or `remix:` prefix).
5. Push the branch: `git push -u origin <branch>`.
6. Tag the commit: `audit-YYYY-MM-DD` or `remix-YYYY-MM-DD`. Push the
   tag: `git push --tags`.
7. **Open a PR automatically.** Prefer `gh pr create` (GitHub CLI); if
   unavailable, use the GitHub MCP's create-PR tool. PR title should
   match the commit summary; PR description should list (a) what was
   audited, (b) KB SHAs referenced, (c) codebase SHAs referenced, (d)
   any superseded ADRs. Do NOT auto-merge — stop after PR creation.
8. Append the audit to `INDEX.md` (if not already done during the session).
9. Summarise what shipped: branch name, commit SHA, PR URL, tag name.
10. Surface any KB-worthy insights from this session that haven't yet
    landed in `../unicorn-kb/` — draft ready-to-paste text per the
    "Push back and recommend an update" rule in
    `../unicorn-kb/pinned/kb-hygiene.md`.

If `gh` CLI and GitHub MCP are both unavailable, stop at push (step 6)
and report the branch URL so the user can open the PR manually.

---

## Source precedence inside this repo

When reasoning during an audit:
1. Actual `<codebase>/` source via filesystem read.
2. `../unicorn-kb/` via filesystem read.
3. Previous audit docs in `audits/` for historical context.
4. Inference — flag with "Inferring from …".

When the codebase and KB disagree, the codebase wins and the divergence
is the finding.
