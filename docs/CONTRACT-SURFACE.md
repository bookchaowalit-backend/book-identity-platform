# Contract surface: book-identity-platform

This page documents the API, event, CLI, storage and configuration surface that
`book-identity-platform` owns or consumes. The machine-readable list is `interfaces` in
[`contract.json`](../contract.json); `scripts/check.py` fails when an interface
listed there is missing from this page.

- Platform: `identity` (contract `book-platform.contract.v1`, API `v1`)
- Status: `scaffolded`; source status: `current-implementation-in-solo-empire`
- Current implementation: `solo-empire` at `infra/api/server.ts` and `infra/api/auth.ts`
- Depends on: none

Interfaces marked `current-in-solo-empire` are served by the `solo-empire`
control plane today. This repository does not serve them yet; they are the
compatibility surface a migration must preserve. The `source` column is a path
in the implementing repository.

## Interfaces

| ID | Kind | Direction | Interface | Contract | Auth | Source | Status |
|---|---|---|---|---|---|---|---|
| `identity.http.bearer-auth` | http | provides | Authorization: Bearer <token> validation for /api/* (requireAuth) | - | - | `infra/api/auth.ts` | current-in-solo-empire |
| `identity.config.scoped-tokens` | config | provides | API_TOKENS_JSON scoped token registry (subject, token_hash, scopes, expires_at) | - | - | `infra/api/auth.ts` | current-in-solo-empire |
| `identity.config.legacy-token` | config | provides | API_TOKEN legacy full-scope token and ALLOW_UNAUTH_DEV local switch | - | - | `infra/api/auth.ts` | current-in-solo-empire |

## Behaviour notes

Derived from the current source; re-check the source before changing a
consumer.

- `requireAuth(req, scope)` accepts `Authorization: Bearer <token>`.
  Failures: 401 `Missing Authorization header`, 401 `Invalid token`, 401
  `Token expired`, 403 `Insufficient scope`, 401 when `API_TOKENS_JSON` is
  invalid (production start-up refuses to run in that state).
- `API_TOKENS_JSON` is an array of at most 100 entries with `subject`,
  `token_hash` (never a raw `token` or `secret`), 1-20 `scopes` matching
  `^[a-z][a-z0-9:_-]{1,63}$`, and optional ISO `expires_at`.
- Scopes observed in the current routes: `api:read`, `api:write`,
  `ai:generate`, `finance:read`, `finance:write`, and `*`. `api:write` implies
  `api:read`; `finance:write` implies `finance:read`.
- `API_TOKEN` (legacy) grants `*` as subject `legacy-api-token`;
  `ALLOW_UNAUTH_DEV=true` grants `*` as `local-dev` only when no token is
  configured. Neither is a production subject assertion.
- Tenant, membership and RBAC capabilities are registry targets with no
  current implementation.

## Migration gates

- tenant isolation
- credential rotation
- session parity

## Out of scope

No production provider integration, credential, database writer or customer
payload lives in this repository. Record parity, privacy and rollback evidence
in the parent platform registry before any cutover.
