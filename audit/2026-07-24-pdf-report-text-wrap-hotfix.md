# Audit: 2026-07-24 — pdf-report-text-wrap-hotfix

**Trigger:** ad-hoc (troubleshooting session escalated to a direct hotfix)
**Scope:** Audit Report Generation troubleshooting for a specific extreme-risk
Due Diligence audit (Upskill You Pty Ltd, tenant 7553, audit
`ccdfa46f-cc46-4f69-85dc-d0c7b30b3bb9`) — Report Generation permission gap,
PDF/DOCX mismatch investigation, DOCX wrong-table bug, PDF text-overflow bug.
Did not look at other report types, other audits, or other tenants.

## Findings
- Sharwari Rajurkar (`unicorn_role = 'CSC'`) was blocked from generating
  reports because `role_permissions` had `audits.report` / `audits.operate`
  set to `'none'` for CSC. This was deliberate RBAC v5 design (documented in
  `unicorn-kb/handoffs/rbac-v5-implementation-plan.md`), not a bug — resolved
  via the in-app Role Permission Editor (Carl/Angela set CSC to `'full'` on
  both features), no code change.
- `generate-client-audit-report-docx` queried the wrong table
  (`audit_template_questions`, an unrelated form-builder table) instead of
  `compliance_template_questions` for question text, silently swallowing the
  resulting Postgres "column does not exist" error and rendering the literal
  string `"Question"` in place of every question in Detailed Responses.
  Fixed by Lovable — `unicorn-cms-f09c59e5@d2e95762` ("Fixed audit question
  text query").
- `generate-client-audit-report` (PDF) drew the finding heading, the finding
  section/regulatory-reference line, and the action title using a single
  non-wrapping `drawText()` call (only a character-count `.slice(0, 200)`,
  not width-aware), causing long text to overflow off the right edge of the
  page instead of wrapping.
- **This function has no git-tracked source in `unicorn-cms-f09c59e5`** —
  confirmed via exhaustive search this session — unlike its DOCX sibling,
  which at least has git history. It exists only as a deployed Supabase Edge
  Function, presumed managed via Cursor/MCP or a manual deploy outside
  Lovable's normal flow.
- Because there was no git-tracked source to route through a Lovable/Cursor
  prompt, and Carl explicitly authorised a direct fix in-session, the fix
  (wrap heading/reference/title via the file's existing `wrapLines()`
  helper; size the coloured accent bar and following content's y-offset to
  the actual wrapped line count instead of an assumed single line) was
  hand-written and deployed straight to the live Supabase project via
  Supabase MCP (`generate-client-audit-report`, now **version 12**),
  bypassing Lovable/Cursor entirely.

## KB changes shipped
- No changes this session. Follow-up recommended: add a note to
  `unicorn-kb/codebase-state/` that `generate-client-audit-report` (like its
  DOCX sibling) has no git-tracked source — mirrors an existing gap already
  flagged for the DOCX function.

## Codebase observations (read-only)
- `unicorn-cms-f09c59e5@286c25cf3b32944a9337686b40da421a1ce129fc` —
  `generate-client-audit-report-docx` fix (commit `d2e95762`) present at this
  SHA. `generate-client-audit-report` (PDF) has no corresponding file in the
  repo at any SHA — confirmed via `git log --oneline --follow` and directory
  search.

## Decisions
- None formal this session — residual risks parked below for follow-up.

## Open questions parked
- Should `generate-client-audit-report` (PDF) be brought under git/Lovable
  management, matching its DOCX sibling's at-least-partial git history?
  Currently both functions' true source of record is only the live Supabase
  deploy, not git. Worth clarifying with Carl/Dave who owns these two
  functions' deploys going forward (Cursor Composer vs. Lovable vs. manual).
- Server-side permission asymmetry noted this session:
  `generate-client-audit-report-docx` enforces
  `check_permission('audits.report', 'full')` server-side;
  `generate-client-audit-report` (PDF) does not — any authenticated user who
  can read the `client_audits` row (via RLS) can trigger PDF generation
  regardless of their `role_permissions` level. Not fixed this session,
  flagged for follow-up.

## Tag
audit-2026-07-24-pdf-report-text-wrap-hotfix
