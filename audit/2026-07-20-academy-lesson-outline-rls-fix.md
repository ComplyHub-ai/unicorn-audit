# Audit: 2026-07-20 — academy lesson outline rls fix

**Trigger:** ad-hoc — client (Think Real Estate, tenant 7541) reported Vivacity Academy courses showing no lesson/video count until enrolling
**Scope:** RLS visibility on `academy_courses` / `academy_modules` / `academy_lessons` for non-enrolled authenticated users. Did **not** touch `academy_assessments`, `academy_enrollments`, `academy_certificates`, or any other table's grants/RLS.
**Session owner:** Carl
**Supabase project:** `yxkgdalkbrriasiyyrwk` — Unicorn 2.0 prod

---

## Findings

- Two separate, unrelated bugs were reported together and needed to be split:
  1. **Videos don't play** — root cause is external: every `training_videos.vimeo_url` tested (old and new courses alike) returns `domain_status_code: 403` from Vimeo's oEmbed API, meaning Vimeo's account-level domain-restricted embedding allowlist doesn't include the app's current production domain (`unicorn-cms.au`). **Not fixed in this session** — requires action on the Vimeo account itself (add `unicorn-cms.au` to the allowed embed domains), outside this codebase. Confirmed via direct oEmbed calls against 3 videos spanning a 2019-era course and a 2023-era course; a control fetch of a normal public Vimeo video showed no `domain_status_code` field at all, confirming this is a deliberate-but-misconfigured Vimeo privacy setting, not a universal oEmbed behaviour.
  2. **Course outline (lesson/video count) invisible pre-enrolment** — a real RLS regression, fixed in this session (below).
- Root cause of (2): migration `20260625031322_be608474-…sql` (25 Jun) replaced an open "authenticated can view published" SELECT policy on `academy_modules`/`academy_lessons`/`academy_assessments` with one requiring staff or an existing `academy_enrollments` row. `AcademyCourseDetailPage.tsx` was designed to show the full published outline (lesson count, locked 🔒 rows) pre-enrolment and only gate *navigation* on enrolment — it never expected the underlying query to return zero rows. Confirmed live via `SET ROLE authenticated` + JWT-claim simulation: a non-enrolled user saw 0 modules / 0 lessons for a course that has 1 published module and 6 published lessons.
- `academy_courses`' live SELECT policy (`status = 'published'`, open to any authenticated user) has **no corresponding `CREATE POLICY` in any migration file in the repo** — confirmed by grepping all migrations and cross-checking `supabase_migrations.schema_migrations.statements`, which only shows it inside the opaque `00000000000000` baseline squash. This is why the course card rendered but the outline underneath didn't — the two tables had drifted to inconsistent policies with no paper trail for one of them. Reconciled (not changed) in this session's migration so checked-in state now matches production.
- The 25 June migration's enrolment predicate also had a latent correctness gap unrelated to the reported bug: it checked only `e.user_id = auth.uid()`, with no `status`/`revoked_at`/`expires_at` filter. Verified live data (527 enrolments: 512 `active`, 15 `completed`, 0 revoked/expired) before tightening the predicate, to guarantee zero users lose access as a side effect.
- Caught mid-review: Lovable's first implementation draft added a `get_lesson_content` SECURITY DEFINER RPC and a matching `AcademyLessonViewerPage.tsx` split to serve gated lesson content to a "wider" audience — unnecessary, since that page's audience (staff/enrolled/preview) is identical to what the corrected base-table policy already admits for the full row. Sent back for simplification; final version has zero RPC and touches exactly one frontend file.
- Caught mid-review: Lovable's own verification queries in an earlier draft were plain `SELECT`s that would run with RLS-bypassing privileges (service_role/postgres) and prove nothing regardless of policy correctness. Corrected to use a `SET ROLE authenticated` + `set_config('request.jwt.claims', …)` transaction wrapper, independently re-run by Carl before and after deploy.

## DB changes shipped

- `unicorn-cms-f09c59e5 @ 9d3ecad9` (merge; parent `02908523`) — migration `20260719234819_9377bf94-8b2c-462e-a876-bca0a780f841.sql`:
  - `academy_courses`: re-declared "authenticated view published" SELECT policy (`status = 'published'`) — no behaviour change, closes the drift.
  - `academy_modules`: SELECT policy widened to `is_published = true` for `authenticated` (was enrolment-gated).
  - `academy_lessons`: SELECT policy rewritten to admit full rows (incl. `content_markdown`, `video_id`, `resource_id`) to staff, `is_preview = true` lessons, or a qualifying enrolment (`revoked_at IS NULL AND (expires_at IS NULL OR expires_at > now()) AND status IN ('active','completed')`).
  - New view `public.v_academy_lesson_outline` — definer-style bypass view (`security_invoker = false`, `OWNER TO postgres`), exposes only structural columns (`id, module_id, course_id, title, description, lesson_type, sort_order, is_published, is_preview, estimated_minutes`) for `is_published = true` rows, granted to `authenticated`/`service_role`.
  - Rollback statements for all three policy changes committed as SQL comments in the same migration file (not executed).
- Live re-verification performed independently by Carl (not just trusting Lovable's reported output), via `SET ROLE authenticated` + JWT-claim simulation, post-deploy:
  - Non-enrolled tenant-7541 user (`ac5fceae-…`) vs course 23: `course_visible=1, modules_via_base=1, lessons_via_view=6, lessons_via_base=0, gated_cols_readable=0`.
  - Completed-status enrollee (`91481df0-…`) vs course 3: `lessons_via_base=48, lessons_via_view=48, content_readable=48` — full access unchanged from pre-deploy baseline.
  - The actual affected user (tenant 7541 primary contact, `978c23d0-…`, active enrolment on course 23): `lessons_via_base=6, content_readable=6` — sees full lesson content end-to-end.
  - Pre-deploy baseline (same non-enrolled user, before migration applied): `modules=0, lessons=0, course_visible=1` — reproduced the reported bug exactly before fixing it.

## KB changes shipped

- No changes shipped.

## Codebase observations (read-only)

- `unicorn-cms-f09c59e5 @ 9d3ecad9`: also touched `src/hooks/academy/useAcademyModulesLessons.ts` (lesson query in `useModulesWithLessons` switched from `.from("academy_lessons")` to `.from("v_academy_lesson_outline")`, dropped `content_markdown`/`video_id`/`resource_id` from its select + made those fields optional on the `AcademyLesson` interface) and `src/integrations/supabase/types.ts` (regenerated). Diffed the full commit against the prior tip (`6ba1bdcd`) to confirm no unrelated scope creep — exactly these 3 files changed.
- `AcademyLessonViewerPage.tsx`, `useMyEnrolledCourses.ts`, `useAcademyCourses.ts`, and all admin builder hooks (`useAdminAcademyCourses`, `LessonEditorPanel`, `ImportVideosPanel`, `useAcademyAssessmentBuilder`, `EnrolmentProgressDrawer`) were left untouched — confirmed their existing query shapes are already admitted by the corrected base-table policy (staff bypass, or enrolment for the client pages).

## Decisions

- **Option B** (column-split via bypass view) chosen over Option A (fully open outline including `content_markdown`/`video_id`) — preserves the inferred security intent of the 25 June migration (no pre-purchase leakage of full lesson content) while fixing the reported bug.
- **Rejected** Lovable's first-draft RPC-based approach for lesson content as unneeded complexity — the base-table policy alone already serves `AcademyLessonViewerPage`'s exact audience.
- **Rejected** the audit's first-draft enrolment tightening (`status = 'active'` only) — would have broken 15 completed-course users' access; corrected to `status IN ('active','completed')` plus `revoked_at`/`expires_at` checks, verified data-safe (0 users lose or gain access) before shipping.
- Accepted the `academy_courses` policy drift as-is (re-declared, not changed) rather than investigating further how it came to exist outside the migration history — flagged as an open question below.
- No off-peak deployment window used — tables are low-traffic/read-mostly, policy/view DDL takes an `ACCESS EXCLUSIVE` lock for milliseconds.

## Open questions parked

- **Videos still don't play.** This session fixed outline visibility only. The Vimeo domain-embed 403 issue is unresolved and needs someone with access to the Vivacity Vimeo account to add `unicorn-cms.au` (and any other current production domain) to the domain-restricted embedding allowlist, at the account or showcase level. No code/migration fix is possible for this from the codebase side.
- How did `academy_courses`' live "authenticated view published" policy come to exist with no matching migration file in the repo? Only appears inside the opaque `00000000000000` baseline squash in `supabase_migrations.schema_migrations`. Worth a future session to understand whether other undocumented drift exists on other tables, and whether the baseline-squash process needs tightening so future ad-hoc dashboard changes don't silently lose their migration paper trail.
- Pre-25-June behaviour let a non-enrolled user hitting a lesson URL directly get a friendly "please enrol" toast + redirect (the row loaded, then the client-side gate handled it). Since lessons are now RLS-denied outright for non-enrolled/non-preview users, that same URL now falls through to a blunt "Lesson not found" — same end state, worse messaging. Not fixed this session (out of scope for the reported bug); a future fix could have `AcademyLessonViewerPage`'s structural fetch also read from a lesson-scoped outline-style source so the friendly gate always has enough data to fire.
- Whether `academy_assessments` and `academy_assessment_questions` warrant the same enrolment-predicate correction (`status IN ('active','completed')` vs `status = 'active'` only) applied here to modules/lessons — not touched this session since not part of the reported bug, but the same latent gap likely exists there.

## Tag

`audit-2026-07-20-academy-lesson-outline-rls-fix`
