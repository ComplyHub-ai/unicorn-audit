# Audit Index

Chronological list of audits. Git tags are the authoritative reference
(`git tag --list "audit-*" "remix-*"`), this is the human-readable mirror.

Newest first.

---

## 2026

### May

- [2026-05-11 — PDP views security_invoker fix — v_pdp_user_currency and v_pdp_cycle_summary ran as DB owner, bypassing RLS on all four underlying tables; security_invoker=true applied, verified live; manager policy gap flagged to Angela](audit/2026-05-11-pdp-views-security-invoker-fix.md) · tag: `audit-2026-05-11-pdp-views-security-invoker-fix` · author: Khian (Brian)
- [2026-05-11 — BUG-018 email delivery investigation — confirmed non-issue; Mailgun delivered successfully (2.0.0 OK), email landed in Gmail Promotions tab; same-browser session conflict during QA also a non-bug](audit/2026-05-11-bug-018-email-delivery-investigation.md) · tag: `audit-2026-05-11-bug-018-email-delivery-investigation` · author: Khian (Brian)
- [2026-05-11 — BUG-011/019 login routing timeout fix — staff and full-access users routed to Academy fallback; profile load race condition in useUserAccess.ts; single-line fix, commit 893fc01a](audit/2026-05-11-bug-011-019-login-routing-timeout.md) · tag: `audit-2026-05-11-bug-011-019-login-routing-timeout` · author: Khian (Brian)
- [2026-05-11 — BUG-002/003 secondary contact portal access fix — Invite button, checkboxes, bulk actions disabled; getTenantRole() replaced with canManagePortalUsers in ClientUsersPage + ClientTasksPage; BUG-020 follow-on parked](audit/2026-05-11-bug-002-003-secondary-contact-portal-access.md) · tag: `audit-2026-05-11-bug-002-003-secondary-contact-portal-access` · author: Khian (Brian)
- [2026-05-11 — Cron JWT rotation blocked — private schema + helper deployed; job command update blocked by supabase_read_only_user ownership; Supabase support ticket required](audit/2026-05-11-cron-jwt-rotation-blocked.md) · tag: `audit-2026-05-11-cron-jwt-rotation-blocked` · author: Carl
- [2026-05-11 — Angela sessions 7–10 May — Academy impersonation backend, support tickets system, security hardening (users_update_own escalation fix, storage lockdown, security_invoker), research findings RLS, academy routes restructure (249 commits, 14 migrations)](audit/2026-05-11-angela-sessions-may7-10-academy-support.md) · tag: `audit-2026-05-11-angela-sessions-may7-10-academy-support` · author: Carl
- [2026-05-08 — Staff directory RPC hardening — get_vivacity_team_directory + get_vivacity_team_directory_staff, useVivacityTeamUsers switched to RPC, 2 duplicate raw queries removed, email/job_title no longer returned to client callers](audit/2026-05-08-staff-directory-rpc-hardening.md) · tag: `audit-2026-05-08-staff-directory-rpc-hardening`
- [2026-05-08 — Messaging unread/read visibility + nav badge inflation + Direct label — staff/client unread row contrast, badge scoped to participant-only convos + realtime clear fix, "Message your CSC" subject replaced with "Direct message" in staff view](audit/2026-05-08-messaging-unread-badge-direct-label.md) · tag: `audit-2026-05-08-messaging-unread-badge-direct-label` · author: Khian (Brian)
- [2026-05-08 — Client packages action buttons fix — "Open Tasks" routed to staff route (→ homepage), "Messages CSC" triggered mailto popup; both fixed to correct client portal routes, dead code removed](audit/2026-05-08-client-packages-action-buttons-fix.md) · tag: `audit-2026-05-08-client-packages-action-buttons-fix` · author: Khian (Brian)
- [2026-05-08 — Bug 2 + Bug 12 — dedupe_key index cleanup (drop duplicate partial index, keep both arbiters) + Today's Focus smoke-test close](audit/2026-05-08-bug2-bug12-dedupe-index-cleanup.md) · tag: `audit-2026-05-08-bug2-bug12-dedupe-index-cleanup` · author: Khian (Brian)
- [2026-05-08 — Client URL access audit — full route inventory, 13 HIGH + 8 MEDIUM unguarded staff routes, staff member enumeration via useVivacityTeamUsers, Phase 2/3 plan](audit/2026-05-08-client-url-access-audit.md) · tag: `audit-2026-05-08-client-url-access-audit`
- [2026-05-08 — Client packages RLS perf fix — `security_invoker` timeout root cause, RLS semi-join rewrite, SECURITY DEFINER RPC `get_client_package_dashboard`, Academy lesson count fallback](audit/2026-05-08-rls-perf-client-package-dashboard.md) · tag: `audit-2026-05-08-rls-perf-client-package-dashboard`
- [2026-05-08 — Dashboard bugs 9/10/11 + Bug 7/8 smoke-test close — label text, View Tasks nav, 999-days v_dashboard_tenant_portfolio fix (391 tenants corrected)](audit/2026-05-08-dashboard-bugs-9-10-11-fix.md) · tag: `audit-2026-05-08-dashboard-bugs-9-10-11-fix` · author: Khian (Brian)
- [2026-05-07 — Bug 7 + Bug 8 — CSC realtime messages + client unread/bell clear (filtered subscription root cause, source_id backfill, regression fix)](audit/2026-05-07-bug7-bug8-realtime-messaging-client-unread.md) · tag: `audit-2026-05-07-bug7-bug8-realtime-messaging-client-unread` · author: Brian
- [2026-05-07 — Day 2 messaging unread badge — realtime bell, source_id trigger fix, CSC unread dot, bell-clears-on-open, client bell realtime, Bugs 3/4/5 resolved](audit/2026-05-07-day2-messaging-unread-badge.md) · tag: `audit-2026-05-07-day2-messaging-unread-badge` · author: Brian
- [2026-05-07 — Client home "Message" button routed to CSC tab — two `openHelpCenter("chatbot")` → `openHelpCenter("csc")` swaps in `ClientHomePage.tsx`](audit/2026-05-07-client-home-message-csc-tab-fix.md) · tag: `audit-2026-05-07-client-home-message-csc-tab-fix` · author: Khian (Brian)
- [2026-05-07 — Angela security hardening — auth_tokens, profiles, eos_workspaces, task-evidence, task-files, views security_invoker (7 fixes, 1 accepted legacy risk)](audit/2026-05-07-angela-security-hardening-rls-storage-views.md) · tag: `audit-2026-05-07-angela-security-hardening-rls-storage-views` · author: Khian (Brian)
- [2026-05-06 — CSC card "Not yet assigned" RLS fix — missing users_select_assigned_csc policy on users table](audit/2026-05-06-csc-card-not-yet-assigned-rls-fix.md) · tag: `audit-2026-05-06-csc-card-not-yet-assigned-rls-fix` · author: Khian (Brian)
- [2026-05-06 — Invite acceptance tenant_members fix — accept_invitation_v2 extended, 416-row backfill (83 active restored, 333 inactive)](audit/2026-05-06-invite-acceptance-tenant-members-fix.md) · tag: `audit-2026-05-06-invite-acceptance-tenant-members-fix` · author: Khian (Brian)
- [2026-05-06 — Messaging RLS cutover — tenant_messages wired, participant-scoped RLS, audit triggers, Help Center CSC unified, 15 live CI tests](audit/2026-05-06-messaging-rls-cutover.md) · tag: `audit-2026-05-06-messaging-rls-cutover`
- [2026-05-06 — Suggestion Register client portal — is_client_visible flag, 4 migrations, RLS hardening, notify edge function, client portal pages](audit/2026-05-06-suggest-register-client-portal.md) · tag: `audit-2026-05-06-suggest-register-client-portal`
- [2026-05-05 — Ask Viv client Gemini integration — replaced deterministic formatter with Gemini, fixed two silent tasks tenant_id bugs](audit/2026-05-05-ask-viv-client-gemini-integration.md) · tag: `audit-2026-05-05-ask-viv-client-gemini-integration`
- [2026-05-04 — Angela's client portal rebuild, package dashboard, invitations overhaul, RLS hardening (296 commits, 23 migrations)](audit/2026-05-04-angela-client-portal-packages-invitations.md) · tag: `audit-2026-05-04-angela-client-portal-packages-invitations`

### April

- [2026-04-30 — Ask Viv client mode — edge function + frontend build (Prompts 2–5 of build-ask-viv-client-mode-v1)](audit/2026-04-30-ask-viv-client-mode-build.md) · tag: `audit-2026-04-30-ask-viv-client-mode-build`
- [2026-04-30 — Ask Viv client mode — ai_client_query_usage migration (Prompt 1 of build-ask-viv-client-mode-v1)](audit/2026-04-30-ask-viv-client-mode-migration.md) · tag: `audit-2026-04-30-ask-viv-client-mode-migration`
- [2026-04-30 — Angela's AI Executive Summary Drafter & multi-framework corpus (26 commits, 2 migrations, 2 edge functions + patches)](audit/2026-04-30-angela-ai-executive-summary-multi-framework-corpus.md) · tag: `audit-2026-04-30-angela-ai-executive-summary-multi-framework-corpus`
- [2026-04-30 — Angela's AI RAG corpus, Finding Drafter & Evidence Analyser (26 commits, 4 migrations, 5 edge functions)](audit/2026-04-30-angela-ai-rag-finding-drafter-evidence-analyser.md) · tag: `audit-2026-04-30-angela-ai-rag-finding-drafter-evidence-analyser`
- [2026-04-29 — Evidence files open/view — RLS verified, Lovable prompt issued](audit/2026-04-29-audit-documents-open-view.md) · tag: `audit-2026-04-29-audit-documents-open-view`
- [2026-04-29 — audit_type full session: constraint fix, filter dropdown, edge function labels, dd_audit_type conversion (Dave direction)](audit/2026-04-29-audit-type-fixes-and-dd-conversion.md) · tag: `audit-2026-04-29-audit-type-fixes-and-dd-conversion` · author: Khian (Brian)
- [2026-04-29 — Ask Viv full restoration (V4 compliance-assistant — 3 bugs fixed, 5 prompts verified)](audit/2026-04-29-ask-viv-restoration.md) · tag: `audit-2026-04-29-ask-viv-restoration`
- [2026-04-29 — Audit workspace sidebar UX fixes (dead `isSelected` prop + false sidebar highlight)](audit/2026-04-29-audit-workspace-sidebar-ux-fixes.md) · tag: `audit-2026-04-29-audit-workspace-sidebar-ux-fixes`
- [2026-04-29 — `client_audits.audit_type` CHECK constraint gap (`due_diligence_combined` missing)](audit/2026-04-29-client-audits-audit-type-constraint.md) · tag: `audit-2026-04-29-client-audits-audit-type-constraint`
- [2026-04-28 — AcceptInvitation blank screen fix + platform-wide anti-pattern sweep](audit/2026-04-28-accept-invitation-blank-screen-fix.md) · tag: `audit-2026-04-28-accept-invitation-blank-screen-fix`
- [2026-04-28 — ManageTenants performance investigation & React Query migration](audit/2026-04-28-manage-tenants-perf-optimisation.md) · tag: `audit-2026-04-28-manage-tenants-perf-optimisation`
- [2026-04-28 — Staff UUID relink fix (infinite spinner, 108 FK CASCADE)](audit/2026-04-28-staff-uuid-relink-fix.md) · tag: `audit-2026-04-28-staff-uuid-relink-fix`
- [2026-04-28 — Clean Architecture proposal review + audit-repo reconciliation](audit/2026-04-28-clean-arch-proposal-and-audit-reality.md) · tag: `audit-2026-04-28`
- [2026-04-27 — Codebase week review (2026-04-20 → 2026-04-25)](audit/2026-04-27-codebase-week-review.md) · tag: `audit-2026-04-27-codebase-week-review`
- [2026-04-24 — KB restructure](audit/2026-04-24-kb-restructure.md) · tag: `audit-2026-04-24`

