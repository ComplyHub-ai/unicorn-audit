# Audit: 2026-06-16 — user-capacity-limits

**Trigger:** ad-hoc — product requirement to enforce per-membership user seat limits
across both the staff invite flow and the client portal, with UI visibility of current
usage.
**Scope:** `packages.user_limit` column + seed; `get_tenant_user_capacity` RPC;
`invite-user` edge function gating; `useUserCapacity` hook; `CapacityPill` component;
edits to `TenantUsersTab`, `ClientUsersPage`, `InviteUserDialog`.

## Findings

- `packages` table had no `user_limit` column. 14 target slugs confirmed present.
- 15 tenants have multiple simultaneous active root `package_instances` (e.g. tenant 7408:
  KS-RTO + KS-CRI + M-SAR + M-SAC). Design decision: MAX(user_limit) with NULL-as-unlimited
  (most permissive wins). `bool_or(p.user_limit IS NULL)` required alongside `MAX` because
  Postgres `MAX` silently ignores NULLs.
- `user_invitations` has both `relationship_role` and `expires_at`. Counted `status IN
  ('pending','sent')` defensively. NULL `relationship_role` included in the count
  (they become regular users on acceptance — excluding them is a silent bypass).
- `invite-user` edge function had two write paths: standard invitation (line 605) AND
  `skip_email` direct-add (line 445). Both gated. Vivacity staff (`isVivacityStaff ||
  isSuperAdmin`) bypass the check in the helper's first line.
- Counted users = `tenant_users` (excluding primary_contact/secondary_contact, including
  NULL relationship_role) + pending/sent non-expired invitations (same exclusions).
- Tier limits set: Gold=5, Ruby=10, Sapphire=15, Diamond=NULL (unlimited), KS-*=5,
  all others=5 (baseline).
- **Flag**: tenant 6372 (Vivacity Coaching & Consulting) is on M-AM (Amethyst, defaulted
  to 5) and has 6 users — 1 over cap. Vivacity staff callers bypass the limit, but the
  cap applies to the tenant itself. M-AM `user_limit` may need to be set to NULL or a
  higher value in a follow-up.

## KB changes shipped

- No KB changes this session.

## Codebase observations

- unicorn @ `f6d0ca14` (origin/HEAD): migration `20260616064856_fa4a3e73-1f9c-4cea-bf1b-8deb41173e41.sql`
  applied — `packages.user_limit` column added + seeded; `get_tenant_user_capacity(bigint)`
  function created (`SECURITY DEFINER SET search_path=''`, `REVOKE ALL FROM PUBLIC`,
  `GRANT EXECUTE TO authenticated`). Edge function `invite-user` updated with
  `assertCapacity()` helper + 2 injection points. New `useUserCapacity.ts` hook,
  `CapacityPill.tsx` (semantic tokens only), edits to `TenantUsersTab.tsx`,
  `ClientUsersPage.tsx`, `InviteUserDialog.tsx`.
- Post-deploy verification:
  - A: all 12 target slugs seeded correctly ✅
  - B: `rpc_used = tu_count + inv_count` confirmed ✅
  - C: tenant 7408 (multi-package): `limit=15, is_unlimited=false` ✅
  - D/E: edge function at-cap rejection vs Vivacity SA bypass — **pending manual UI
    smoke test** against tenant 6372 (6/5 users, M-AM). Vivacity staff bypass confirmed
    in code; authoritative gate is the RPC which returns correct values.

## Decisions

- Count: everyone excluding primary_contact and secondary_contact (user + academy_user),
  plus pending/sent non-expired invitations.
- Multi-package rule: MAX(user_limit), NULL-as-unlimited.
- NULL `relationship_role` invitations: counted (defensive — silent bypass otherwise).
- Both write paths in `invite-user` gated; Vivacity staff bypass both.
- Default when no active package: 5.
- Diamond (`/package-m-dr`, `/package-m-dc`): `user_limit = NULL` (no check).
- Pill copy: "X of Y users" / "N users · Unlimited".
- Vivacity team see the pill in `TenantUsersTab` but Invite button stays enabled
  (mirrors edge-function bypass).

## Open questions parked

- Tenant 6372 (Vivacity Coaching & Consulting) is 1 over the M-AM default cap of 5.
  Decision needed: set `packages.user_limit = NULL` for M-AM, or set a higher number,
  or leave as-is (staff bypass makes it functional for Vivacity-originated invites).
- `TenantUsersTab` query still doesn't fetch `relationship_role` — the display bug
  identified 2026-06-11 (shows academy_user as "User") is still outstanding as a
  separate Lovable fix.

## Tag
audit-2026-06-16-user-capacity-limits
