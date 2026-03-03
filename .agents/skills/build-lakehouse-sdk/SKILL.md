---
name: build-lakehouse-client-sdk
description: Defines how to build and publish a production-grade open-source Altertable Lakehouse API client in any programming language. Use when implementing or maintaining a Lakehouse SDK from the OpenAPI spec, including typed models, streaming query parsing, auth, retries, and release workflows.
---

# Build Lakehouse Client SDK Skill

## Purpose

Use this skill to implement a language-idiomatic, strongly typed (or as strongly typed as idiomatic for the target language—best effort for dynamic or scripting languages), open-source client for the Altertable Lakehouse API.

Primary OpenAPI specification reference: `https://api.altertable.ai/openapi/lakehouse.json`

## Required Outcomes

1. Full endpoint coverage (`append`, `query`, `upload`, `validate`).
2. Package is publishable to the target language's primary registry.
3. Typed models and typed errors are first-class.
4. `/query` exposes parsed streaming output by default.
5. Transport is extensible (timeouts, retries, proxy, middleware/hooks).
6. Tests provide confidence in real runtime behavior.
7. Project is modern OSS with MIT licensing.

## Input Contract

Before implementation, collect or infer:

- Target language/runtime/version
- Package name and namespace
- Sync vs async style based on language best practices
- CI target versions
- Optional live-test credentials/settings

If these are not provided, choose ecosystem-standard defaults and document them.

## Implementation Workflow

Follow these phases in order.

### Phase 1: Scaffold

1. Create package/module scaffold with idiomatic structure.
2. Add MIT license.
3. Add README, changelog, and contribution docs.
4. Configure lint/typecheck/test scripts.

### Phase 2: Models and Serialization

1. Generate or define request/response models from OpenAPI.
2. Preserve enums and nullable semantics:
   - `ComputeSize`: `S | M | L`
   - `UploadFormat`: `csv | json | parquet`
   - `UploadMode`: `create | append | upsert | overwrite`
3. Preserve `oneOf` behavior for `AppendRequest`:
   - `{ Single: AppendPayload }` OR
   - `{ Batch: AppendPayload[] }`

### Phase 3: Client Core

Implement a configurable client constructor/factory with:

- `baseUrl` (default `https://api.altertable.ai`)
- auth options (see Authentication)
- timeout configuration
- retry policy
- custom HTTP transport/adapter
- middleware/interceptor hooks
- optional user-agent suffix

### Phase 4: Endpoint Methods

Implement typed methods for all operations:

1. `append`

   - `POST /append`
   - required query params: `catalog`, `schema`, `table`
   - JSON request body: `AppendRequest`
   - typed response: `AppendResponse`

2. `query`

   - `POST /query`
   - JSON body: `QueryRequest` (must include `statement`)
   - content type: `application/x-ndjson`
   - returns parsed stream interface (iterator/async iterator/observable/channel)

3. `upload`

   - `POST /upload`
   - required query params: `catalog`, `schema`, `table`, `format`, `mode`
   - conditional param: `primary_key` is required when `mode=upsert`
   - body: `application/octet-stream` bytes or stream

4. `validate`
   - `POST /validate`
   - JSON body: `ValidateRequest` (must include `statement`)
   - typed response: `ValidateResponse`

### Phase 5: Streaming Contract (`query`)

Default behavior must parse NDJSON lines into typed events.

Recommended event types:

- `metadata`
- `columns` (when schema row appears)
- `row`
- `done` (if representable in runtime)
- `stream_error`

Requirements:

- Include line index/context in parsing failures.
- Never silently ignore malformed lines.
- Support cancellation and resource cleanup.
- Preserve backpressure semantics of the language runtime.
- Optionally expose raw-line access for advanced consumers.

### Phase 6: Authentication

Support all common, non-surprising patterns:

1. Direct credentials in client config
2. Environment variable discovery
3. Credential provider callback/interface (for rotation)
4. Optional per-request override

Security rules:

- Never log raw credentials.
- Redact authorization headers.
- Document auth precedence rules.

### Phase 7: Error Model

Implement a typed error hierarchy at minimum:

- `AuthError`
- `BadRequestError`
- `NetworkError`
- `TimeoutError`
- `SerializationError`
- `ParseError`
- `ApiError` (unexpected status fallback)
- `ConfigurationError`

All errors should include, when available:

- operation name
- HTTP method/path
- status code
- retriable flag/classification
- request/correlation id headers
- underlying cause

### Phase 8: Transport and Reliability

Must support:

- pluggable transport
- global and per-request timeouts
- retry policy with exponential backoff + jitter
- configurable retriable conditions
- proxy support
- middleware/hooks for logging and tracing

Prefer ecosystem-standard HTTP stack and retry primitives.

### Phase 9: Testing

Implement layered tests:

1. Unit tests

   - model serialization
   - request construction
   - auth behavior and redaction
   - retries/timeouts
   - input precondition checks

2. Integration tests (opt-in)
   - run only when credentials/env are present
   - do one real `explain`, one `query` stream, one `upload`, and one `validate` against the live API

CI should always run lint + typecheck + unit + contract tests.

### Phase 10: Packaging and Release

1. Use semantic versioning.
2. Publish to primary registry for the language.
3. Implement a release-please GitHub Action to automate the release process (incl. changelog generation and release notes).
4. Include examples for all endpoints in the README.
5. Verify docs match runtime behavior.

## Endpoint Reference (Minimal)

### `POST /append`

- Query: `catalog`, `schema`, `table`
- Body: `AppendRequest`
- Response: `AppendResponse { ok: boolean, error_code?: "invalid-data" | null }`

### `POST /query`

- Body: `QueryRequest`
- Response: NDJSON stream
- Key request fields:
  - required: `statement`
  - optional: `catalog`, `schema`, `session_id`, `compute_size`, `sanitize`, `limit`, `offset`, `timezone`, `ephemeral`, `visible`, `requested_by`, `query_id`

### `POST /upload`

- Query: `catalog`, `schema`, `table`, `format`, `mode`, optional `primary_key`
- Constraint: `primary_key` required for `mode=upsert`
- Body: binary file content

### `POST /validate`

- Body: `ValidateRequest` with required `statement`
- Response: `ValidateResponse` with `valid`, `statement`, `connections_errors`, optional `error`

## Acceptance Checklist

Only mark implementation complete when all are true:

- [ ] All 4 endpoints implemented and documented
- [ ] Streaming parser returns typed events
- [ ] Typed errors are comprehensive and actionable
- [ ] Auth supports direct/env/provider patterns
- [ ] Retries/timeouts/transport hooks are configurable
- [ ] Tests provide real-world confidence
- [ ] Package is publish-ready for primary registry
- [ ] MIT license and OSS docs are present

## Output Format for Implementation Agent

When executing this skill, produce:

1. Short implementation plan
2. File-by-file change list
3. Test results summary
4. Publish readiness checklist
5. Known limitations and follow-up items
