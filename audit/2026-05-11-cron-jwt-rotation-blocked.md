# Audit: 2026-05-11 — cron-jwt-rotation-blocked

**Trigger:** ad-hoc — P0 remediation attempt: replace hard-coded anon JWT in cron jobs 1 & 2 with vault-backed helper function.
**Author:** Carl
**Scope:** Unicorn 2.0-dev project `yxkgdalkbrriasiyyrwk`. Cron job command bodies for `generate-notifications-meetings` (job 1) and `generate-notifications-daily` (job 2) only. No codebase changes. No other DB objects touched.

---

## Findings

- **vault.cron_function_jwt secret**: confirmed present on Unicorn 2.0-dev, created 2026-05-10 (from Sunday session). Not readable via MCP (`permission denied for function _crypto_aead_det_decrypt`) — verified via `vault.secrets` metadata only.
- **private schema**: was absent at session start (Sunday's migration was accidentally applied to ComplyHub `gdwhlstfguxarnxasrrs` instead of Unicorn). Successfully created this session via Studio SQL Editor.
- **private.cron_function_jwt() helper**: absent at session start. Successfully deployed this session via Studio SQL Editor. SECURITY DEFINER, SET search_path = '', reads `vault.decrypted_secrets` WHERE name = 'cron_function_jwt'. EXECUTE granted to `supabase_read_only_user`.
- **Cron jobs 1 & 2 command bodies**: still contain hard-coded anon JWT (`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`). Rotation blocked — see below.
- **Cron jobs 3–6**: SQL-only commands, no JWT, not affected.
- **Ownership wall — root cause confirmed**: Jobs 1 and 2 are owned by the `supabase_read_only_user` Postgres role. All rotation paths were exhausted and failed:
  - `cron.alter_job()` via MCP `execute_sql` → `permission denied for function alter_job`
  - `UPDATE cron.job` via MCP `execute_sql` → `permission denied for table job`
  - Studio Cron Jobs UI edit → blocked (ownership check in UI)
  - `cron.alter_job()` via Studio SQL Editor as `postgres` → `Job 1 does not exist or you don't own it`
  - `UPDATE cron.job` via Studio SQL Editor as `postgres` → `permission denied for table job`
- **postgres is not a superuser on Supabase managed projects**: confirmed via `SELECT usesuper FROM pg_user WHERE usename = current_user` — returns `false`. The actual superuser is `supabase_admin`, inaccessible to project owners. This is why `postgres` cannot override the ownership check on `cron.job`.
- **MCP in read-only mode**: `apply_migration` returns `Cannot apply migration in read-only mode`. DDL had to be applied via Studio SQL Editor throughout this session.

## What shipped

- `private` schema created on Unicorn 2.0-dev with PUBLIC access revoked.
- `private.cron_function_jwt()` function deployed — infrastructure is fully ready for the job rotation once the ownership blocker is resolved.
- EXECUTE on `private.cron_function_jwt()` granted to `supabase_read_only_user` so the cron jobs can call it once their command bodies are updated.

## What did not ship

- Cron job 1 (`generate-notifications-meetings`) command body: still hard-coded JWT.
- Cron job 2 (`generate-notifications-daily`) command body: still hard-coded JWT.
- ComplyHub cleanup (`private` schema + `private.cron_function_jwt()` accidentally deployed to `gdwhlstfguxarnxasrrs` on Sunday): not actioned this session. No functional impact on ComplyHub; artefacts are inert but should be removed for tidiness.

## Unblocking path

Raise a Supabase support ticket against project `yxkgdalkbrriasiyyrwk`:

> Cron jobs 1 (`generate-notifications-meetings`) and 2 (`generate-notifications-daily`) are owned by `supabase_read_only_user`. We cannot update their command bodies via `cron.alter_job`, direct SQL, or the Studio Cron Jobs UI — `postgres` is not a superuser on managed projects and cannot override the ownership check. We need to replace the hard-coded anon JWT in each command body with a call to `private.cron_function_jwt()`. Can you either transfer ownership to `postgres`, or update the command bodies directly?

Once Supabase resolves ownership (or updates the command bodies directly), paste these into the job commands:

**Job 1 — generate-notifications-meetings:**
```sql
SELECT net.http_post(
  url := 'https://yxkgdalkbrriasiyyrwk.supabase.co/functions/v1/generate-notifications',
  headers := jsonb_build_object(
    'Content-Type', 'application/json',
    'Authorization', 'Bearer ' || private.cron_function_jwt()
  ),
  body := '{"scope": "meetings"}'::jsonb
) AS request_id;
```

**Job 2 — generate-notifications-daily:**
```sql
SELECT net.http_post(
  url := 'https://yxkgdalkbrriasiyyrwk.supabase.co/functions/v1/generate-notifications',
  headers := jsonb_build_object(
    'Content-Type', 'application/json',
    'Authorization', 'Bearer ' || private.cron_function_jwt()
  ),
  body := '{"scope": "tasks_obligations"}'::jsonb
) AS request_id;
```

Then verify with:
```sql
SELECT jobid, jobname, command FROM cron.job WHERE command LIKE '%eyJ%';
```
Expected: 0 rows.

## Risk note

The hard-coded token is the project's **anon key**, not the service_role key. The anon key is already embedded in the client-side JavaScript bundle and is designed to be public-facing. Actual risk is lower than the P0 label suggests — the rotation mechanism matters for future key rotation events, not because the anon key is currently a secret. This does not gate deployment but should be resolved before any anon key rotation.

## KB changes shipped

No KB changes this session.

## Codebase observations (read-only)

- `unicorn-cms-f09c59e5` @ `1fb3ecc3` (Fixed research findings RLS) — local HEAD. Remote `origin/main` is at `42ba7327` (Updated the AI tool use). Local is behind; user has not yet pulled.

## Decisions

No new ADRs.

## Open questions parked

- ComplyHub cleanup: drop `private.cron_function_jwt()` and `private` schema from project `gdwhlstfguxarnxasrrs`. Safe to do any time; confirm no other objects exist in `private` on ComplyHub before dropping.
- Whether the `vault.cron_function_jwt` secret on Unicorn contains the correct current anon key — could not verify value via MCP (read permission denied on vault decryption). Verify in Studio → SQL Editor: `SELECT left(decrypted_secret, 20) FROM vault.decrypted_secrets WHERE name = 'cron_function_jwt'` and compare against Studio → Settings → API anon key prefix.

## Tag

`audit-2026-05-11-cron-jwt-rotation-blocked`
