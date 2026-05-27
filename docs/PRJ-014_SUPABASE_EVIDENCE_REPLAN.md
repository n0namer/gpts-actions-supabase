# PRJ-014 Course Care Bot — Supabase evidence-based replan

Date: 2026-05-27
Project ref: hbweabzwexfuuhgoirvl
Mode: evidence-based / read-first / no secret values

## North Star

Ship a secure, maintainable Course Care Bot MVP on Supabase where:
- the database schema is stable and protected by RLS;
- bot/backend integrations operate through secure server-side keys only;
- public/client access is intentionally blocked or explicitly governed by policies;
- GPT Actions can inspect and operate Supabase Management API safely;
- production auth/config gaps are closed before real users or traffic.

## Evidence used

Observed:
- Project `PRJ-014 Course Care Bot` is `ACTIVE_HEALTHY`, region `eu-central-1`, Postgres `17.6.1`.
- Applied migrations include `v1_enable_rls_core`.
- `auth.users` count is `0`.
- All current `public` tables have RLS enabled.
- `pg_policies` returned `0` policies for `public`.
- `information_schema.table_privileges` returned no grants for `anon`, `authenticated`, or `service_role` on `public`.
- `storage.buckets` returned `0` buckets.
- Supabase secrets list returned `0` project secrets.
- API key metadata includes legacy `anon`, legacy `service_role`, publishable, and secret keys.
- Auth config still has `site_url=http://localhost:3000`, `disable_signup=false`, `password_min_length=6`, `password_hibp_enabled=false`.
- Attempted low-risk auth hardening via Management API returned unchanged config; treat as `WRITEBACK_FAILED`.

External best-practice basis:
- Supabase requires RLS on exposed schemas and recommends granting only necessary privileges.
- Supabase Data API security is grants + RLS.
- Supabase secret/service-role keys bypass RLS and must stay server-side only.
- Supabase password guidance says less than 8 characters is not recommended.
- OpenAI Actions production notes require short descriptions, HTTPS, payload limits, and consequential flags.

## Replan method

Method used: risk burn-down + impact/effort + shortest path to MVP.
Priority score favors:
1. blocking security defects,
2. irreversible/expensive risks,
3. required production config,
4. contract/evidence tasks that unblock later automation.

## Ordered plan

### 1. Freeze current access model
DoD:
- Document whether MVP is backend-only or client-access.
- If backend-only, public RLS remains default-deny.
- If client-access, required policy matrix is defined per table.

Evidence:
- `pg_policies = 0`
- no grants to `anon`/`authenticated`

Status: DONE as backend-only default for now.

### 2. Fix Auth production URL
DoD:
- `site_url` is real production/staging URL, not localhost.
- Redirect allow-list is explicit.
- Email flows tested after change.

Status: BLOCKED pending real URL.

### 3. Harden password/auth settings
DoD:
- password minimum >= 12 or product-approved value.
- leaked-password/HIBP check enabled if available.
- password change requires safe reauth/current-password policy.
- Verification proves changed values.

Status: WRITEBACK_FAILED via Management API; needs dashboard or corrected endpoint payload.

### 4. Decide signup policy
DoD:
- `disable_signup=true` for closed/internal bot, or explicit invite/signup requirement documented.
- No accidental public registration.

Status: BLOCKED pending product decision.

### 5. Preserve RLS default-deny until access policy exists
DoD:
- No broad `anon`/`authenticated` grants.
- No permissive public policies until policy matrix exists.

Status: DONE.

### 6. Create policy matrix before any client exposure
DoD:
- For each public table: actor, operation, predicate, test query.
- Migrations include policy definitions and verification SQL.

Status: PLANNED.

### 7. Secrets management setup
DoD:
- Required runtime secrets are added to Supabase/secrets or secure runtime env.
- Secret values never appear in docs, logs, GPT responses, or client bundles.

Status: TODO; current project secrets empty.

### 8. API key migration/handling policy
DoD:
- Use publishable key client-side only.
- Use secret key/service role only server-side.
- Legacy keys are not copied into GPT/frontend.
- Rotation/migration decision documented.

Status: PARTIAL; metadata observed, values not recorded.

### 9. Storage policy remains closed until needed
DoD:
- No public buckets by default.
- If storage needed, bucket + RLS policies + MIME/size limits are defined.

Status: DONE for now; no buckets.

### 10. GPT Actions schema import and safety
DoD:
- YAML imports successfully.
- Write/destructive endpoints marked consequential.
- Descriptions stay within OpenAI limits.
- Auth uses Bearer PAT, not project service_role.

Status: PARTIAL; schema patched, needs user-side import result.

### 11. Add verification migration/test pack
DoD:
- Read-only SQL pack proves RLS, grants, policies, indexes, storage, auth assumptions.
- Can be rerun after every migration.

Status: TODO.

### 12. Add operational runbook
DoD:
- How to rotate keys, update site URL, change RLS, and run incident checks is documented.
- Rollback path is defined.

Status: TODO.

## Executed tasks in this wave

1. Read project metadata — PASS.
2. Count auth users — PASS, count is 0.
3. List migrations — PASS, latest RLS migration present.
4. Check public/auth/storage table RLS — PASS for public.
5. Check public RLS policies — PASS, zero policies observed.
6. Check role grants for anon/authenticated/service_role — PASS, no public table grants observed.
7. Check storage buckets — PASS, no buckets.
8. Check secrets metadata — PASS, zero project secrets.
9. Check API key metadata — PASS, new and legacy key types exist; values not recorded.
10. Attempt low-risk Auth hardening — WRITEBACK_FAILED; config returned unchanged.

## Next safest execution wave

1. Get production/staging URL and patch `site_url`.
2. Decide signup mode and set `disable_signup` accordingly.
3. Fix Auth hardening through Dashboard or corrected API payload.
4. Create policy matrix for client-facing tables, if client direct access is required.
5. Add verification SQL pack to repo.
6. Add RLS policy migrations only after matrix approval.
7. Add required secrets through safe secret path.
8. Import GPT Actions YAML and run one read-only action.
9. Run one safe write action against non-production or preview branch.
10. Record outcomes in this report.
