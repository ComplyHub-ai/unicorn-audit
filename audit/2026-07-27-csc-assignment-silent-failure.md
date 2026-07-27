# Audit: 2026-07-27 — csc-assignment-silent-failure

**Trigger:** drift-surfaced
**Scope:** Root-caused and fixed a frontend bug in the CSC (Client Success Champion) quick-reassign UI on the tenant/client detail page in `unicorn-cms-f09c59e5`. Did not audit the wider "CSC vs consultant assignment" split surfaced along the way — parked below, not actioned this session.

## Findings
- Carl reported that reassigning the CSC for a client ("Absolute Medic") via the CSC dropdown on the client detail page "doesn't really change it."
- Root cause: `useTenantCSCAssignment.tsx`'s `assignCSC`/`removeCSC` mutations called `admin_set_tenant_csc_assignment` / `admin_remove_tenant_csc_assignment` via RPC and only checked the Postgrest transport-level `error`. Both RPCs reject invalid requests (caller isn't SuperAdmin, or the target user isn't flagged `is_csc`/`staff_team = 'client_success'`) by returning `jsonb_build_object('success', false, 'error', ...)` — a normal 200 response, no SQL exception. The mutation's `onSuccess` fired regardless: toast said "CSC Assigned"/"CSC Removed", the dialog closed, queries invalidated — but the `tenant_csc_assignments` row was never touched, so the refetch silently reverted to the previous CSC with no error surfaced anywhere.
- Same soft-fail contract exists on both RPCs (confirmed in migrations `20260702031811_958ec350-e0c6-4e84-b15b-0b0719023b0f.sql` for assign, `20260203064857_051c0830-b580-4e58-be18-e5fb999a48d1.sql` for remove) — this was a single shared bug in the frontend caller, not a per-RPC issue.
- Adjacent observation, not investigated further this session: `tenants.assigned_consultant_user_id` is a second, only partially-synced "who owns this client" column, read by capacity/load dashboard views (`v_dashboard_labour_efficiency`, `v_dashboard_weekly_wins`, `v_executive_consultant_distribution`, `vw_consultant_capacity`, `vw_consultant_load`) and written by `useConsultantAssignment.tsx`'s manual-override/auto-assign paths. The quick CSC dropdown (`CSCAssignmentSelector`, the one in this bug report) never touches that column. Only the bulk reassignment RPC (`bulk_reassign_primary_csc`) keeps both in sync. Not confirmed as user-visible drift this session — flagged for a future audit.

## KB changes shipped
- No changes.

## Codebase observations (read-only)
- unicorn (`unicorn-cms-f09c59e5`) @ `00f99403` (branch `hotfix/2026-07-27-csc-assignment-silent-failure`, PR [#46](https://github.com/vivacityrto/unicorn-cms-f09c59e5/pull/46), not yet merged) — `src/hooks/useTenantCSCAssignment.tsx`: both mutations now check `data.success === false` post-RPC-call and throw with the RPC's own `error` message, so the existing `onError` toast surfaces the real rejection reason instead of a false-positive success.

## Decisions
- Fixed via a direct hand-edit under the workspace `CLAUDE.md` explicit-override provision (Carl authorized in-session: "just do the fix, no need for lovable") rather than routing through a Lovable prompt. Scoped narrowly to the two mutation call sites — no RPC/migration change, so the Lovable production DB change workflow does not apply (no schema/RLS/trigger/enum/backfill touched).
- Did not chase the `assigned_consultant_user_id` vs `tenant_csc_assignments` split in the same session — different blast radius, needs its own scoping conversation about which one is meant to be authoritative for dashboards vs. the quick-assign UI.
- PR opened, not merged — per standing session-end default, merge is left for Carl to trigger explicitly.

## Open questions parked
- Whether `tenants.assigned_consultant_user_id` should be deprecated in favour of `tenant_csc_assignments` everywhere, or whether the two are intentionally distinct concepts (capacity-planning "consultant" vs. relationship "CSC") that happen to usually be the same person. Dashboard views listed above should be checked against `tenant_csc_assignments` once that's decided.
- Next Lovable session should be made aware of this hand-applied hotfix (per `unicorn-kb/handoffs/lovable-to-codebase.md`) so it doesn't get silently overwritten or treated as something Lovable itself shipped.

## Tag
audit-2026-07-27-csc-assignment-silent-failure
