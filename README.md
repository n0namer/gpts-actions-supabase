# GPTs Actions — Supabase Management API

This repository contains ChatGPT Actions compatible OpenAPI schemas for the official Supabase Management API.

## Preferred fast schema: direct Supabase REST

Use this schema now for the fastest working GPT Actions setup. It calls the official Supabase Management REST API directly, without an adapter:

```text
openapi/supabase-management.gpt-actions.yaml
```

Server:

```text
https://api.supabase.com
```

## Future schema: command router

Kept as a later MCP-like adapter design, not the current fast path:

```text
openapi/supabase-management-router.gpt-actions.yaml
```

Server:

```text
https://api.supabase.com
```

Official source of truth:

```text
https://api.supabase.com/api/v1-json
```

Supabase Management API docs:

```text
https://supabase.com/docs/reference/api/introduction
```

## GPT Actions authentication

In GPT Builder → Actions → Authentication:

- Authentication type: `API Key` or `Bearer`
- Auth type: `Bearer`
- Header: `Authorization`
- Value format: `Bearer <SUPABASE_PAT_OR_OAUTH_ACCESS_TOKEN>`

The token must have the Supabase permissions required for the endpoints you plan to use.

## Safety contract for the GPT

The schema is intentionally a GPT-Actions-friendly subset, not a raw dump of the upstream OpenAPI spec.

Rules the GPT using this action should follow:

1. Observe first with read operations.
2. Never print or expose passwords, secrets, API keys, JWTs, connection strings, or returned sensitive values.
3. Use `runSupabaseReadOnlySqlQuery` for inspection before any write SQL.
4. Ask for explicit confirmation before:
   - `runSupabaseSqlQuery`
   - migrations / rollback
   - branch merge / reset / push / delete
   - project create / update / pause / restore / delete
   - secrets or API key create / update / delete
   - network restrictions
   - disk resize
   - auth / storage / realtime / PostgREST config updates
5. After a write operation, verify state with the relevant GET/list/status operation.

## Notes

- Some official Supabase endpoints are beta, experimental, deprecated, or partner/OAuth-limited.
- Multipart Edge Function deployment is not included as a direct upload action because GPT Actions schemas are more reliable with JSON request bodies. Function list/get/update/delete and bulk JSON update are included.
- The upstream OpenAPI spec contains complex schemas that are intentionally simplified here to improve GPT Actions import reliability.
