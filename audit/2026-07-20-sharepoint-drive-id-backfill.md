# Audit: 2026-07-20 — sharepoint-drive-id-backfill

**Trigger:** ad-hoc
**Scope:** Data backfill of `document_versions.source_drive_item_id`/`source_site_id` for legacy document template rows in Unicorn 2.0 production (Supabase project `yxkgdalkbrriasiyyrwk`). Did not look at anything outside the `documents`/`document_versions` tables and the one throwaway edge function used to run the backfill.

## Findings
- A new "Hide already-imported" filter toggle was added to `SharePointTemplateBrowser.tsx` (UI-only Lovable change, already live on `origin/main`) that matches SharePoint browse results against `document_versions.source_drive_item_id`. Testing surfaced that this only worked for documents imported via the real `import-sharepoint-template` pipeline (15 rows) — 27 legacy rows (bulk-inserted 2026-01-13, predating that pipeline) had `source_drive_item_id IS NULL` despite having a valid `documents.source_template_url`, so the toggle silently failed to recognize them as already-imported.
- Root cause: `source_drive_item_id` can only be obtained by resolving the SharePoint sharing URL through Microsoft Graph (`resolveDriveItemFromSharingUrl` in `_shared/graph-app-client.ts`) — not derivable via SQL alone.
- Resolved a same-session dead end: a Nov 2025 migration (`20251127233001_...sql`) calls `net.http_post` using `current_setting('app.settings.service_role_key', true)` to self-invoke `send-automated-email`. That GUC is **not actually set** in this database (`current_setting(..., true)` returns null) — meaning that automated-email trigger has likely been silently no-op-failing since whenever the setting was removed/never configured (its `net.http_post` call is wrapped in `EXCEPTION WHEN OTHERS`, so it fails silently). Not fixed this session — flagged as an open question below.
- Of the 27 legacy rows, 26 resolved cleanly via Graph (all sharing the same drive/site, consistent with the single Master Documents SharePoint site). 1 failed with a 403 `accessDenied`/"file not found" — document id 55, "Demo document to see if it bee added on existing" — the underlying SharePoint file no longer exists. Left unresolved; this is a dead test/demo document, not a defect in the backfill.

## KB changes shipped
- No changes.

## Codebase observations (read-only)
- unicorn (`unicorn-cms-f09c59e5`) @ `4ed8da1c07bf9fb2dfb32a883d92cf4452a1e5ec` — state at time of backfill; `SharePointTemplateBrowser.tsx`'s "Hide already-imported" toggle already merged.

## Decisions
- Ran the backfill directly via a throwaway Supabase edge function (`tmp-backfill-sharepoint-drive-ids`, deployed/invoked/neutralized this session) rather than a Lovable prompt, since the fix never touches the tracked codebase — reasoning and the workflow this session followed (audit → design decision on where the resolver code lives → dry run → live run → verification) are per `unicorn-kb/handoffs/lovable-production-db-change.md`, applied here even though execution used direct MCP/SQL access rather than Lovable.
- Chose to resolve via a disposable one-off function (Option B) over adding a permanent `backfill` action to `import-sharepoint-template` (Option A), to avoid leaving unused maintenance code in the permanent codebase.
- Function was neutralized (redeployed as a 410-Gone stub) after use rather than deleted — no MCP tool exists to delete an edge function outright. Still listed in the project's function list; safe to delete via the Supabase dashboard whenever convenient.

## Open questions parked
- `app.settings.service_role_key` GUC referenced by the Nov 2025 automated-email trigger is unset in production — that trigger's `net.http_post` call has likely been silently failing since some unknown point. Worth a dedicated follow-up session to confirm impact and either restore the GUC or refactor the trigger.
- Document id 55 ("Demo document...") has a dead `source_template_url` pointing at a deleted SharePoint file — no action taken; low-priority cleanup candidate if it's ever surfaced to a user.
- `tmp-backfill-sharepoint-drive-ids` edge function still exists in Supabase (neutralized, not deleted) — delete via dashboard when convenient.

## Tag
audit-2026-07-20-sharepoint-drive-id-backfill
