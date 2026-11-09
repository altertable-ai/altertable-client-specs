# Standardized Compliance Test Plan

This document outlines the mandatory test scenarios for all Altertable Product Analytics SDKs. The goal is to ensure consistent behavior across languages and platforms, particularly for edge cases like identity management and queue handling.

Every SDK must implement these scenarios in its test suite. While implementation details (mocks, syntax) will vary by language, the **inputs** and **expected outcomes** must match this specification.

## 1. Identity Management

### Scenario: Identify (First Call)
- **Input:** Call `identify(userId: "user_123")` when no user is currently identified.
- **Expectation:**
  - `userId` is stored in persistent storage.
  - Subsequent events include `userId: "user_123"`.
  - Session ID is generated/maintained.

### Scenario: Identify (Same User)
- **Input:** Call `identify(userId: "user_123")` when `user_123` is already the current user.
- **Expectation:**
  - No-op.
  - No new session is started.
  - No warning logs.

### Scenario: Identify (New User - Identity Shift)
- **Input:** Call `identify(userId: "user_456")` when `user_123` is currently identified.
- **Expectation:**
  - **Log Warning:** SDK should log a warning that the user identity has changed without a `reset()`.
  - **Auto-Reset:** The SDK must automatically call `reset()` internally to clear the old session and traits.
  - **Update:** The new `userId` ("user_456") is stored.
  - **New Session:** A new session ID is generated for the new user.

### Scenario: Alias (Linking)
- **Input:** Call `alias(newId: "user_123")` when anonymousId is "anon_abc".
- **Expectation:**
  - An `alias` event is enqueued with `previousId: "anon_abc"` and `userId: "user_123"`.
  - The stored `userId` is updated to "user_123".

### Scenario: Reset (Logout)
- **Input:** Call `reset()`.
- **Expectation:**
  - `userId` is cleared from storage.
  - `anonymousId` is regenerated (or preserved depending on config, default: regenerate).
  - Session ID is cleared/regenerated.
  - Traits are cleared.

## 2. Event Tracking

### Scenario: Basic Track
- **Input:** Call `track(event: "Button Clicked", properties: { "color": "blue" })`.
- **Expectation:**
  - Event is enqueued.
  - Payload includes:
    - `event`: "Button Clicked"
    - `properties`: { "color": "blue" }
    - `userId`: (current user ID or null)
    - `anonymousId`: (current anonymous ID)
    - `timestamp`: (ISO 8601)
    - `context`: (library info, os, device)

### Scenario: Track with NIL/Null Properties
- **Input:** Call `track(event: "Viewed", properties: null)`.
- **Expectation:**
  - Event is enqueued.
  - `properties` defaults to `{}` (empty object) in the payload, or is omitted if the spec allows, but must not crash.

## 3. Queue & Batching

### Scenario: Offline Queueing
- **Input:**
  1. Disconnect network (mock).
  2. Call `track("Event A")`.
  3. Call `track("Event B")`.
- **Expectation:**
  - Events are stored locally (disk/memory).
  - No network requests are attempted immediately (or they fail gracefully).
  - `queueSize` increases.

### Scenario: Batch Flush
**Web and mobile tiers only.**

- **Input:**
  1. Queue contains 21 track payloads.
  2. Network is restored.
  3. Flush triggered (manual or auto-timer).
- **Expectation:**
  - With the default `maxBatchSize` of `20`, the SDK sends two `/track`
    requests: the first has 20 payloads and the second has 1.
  - No request for a given endpoint contains more than `maxBatchSize`
    payloads; payload order is preserved across the resulting requests.
  - A configured value below `1` is clamped to `1` before flushing.
  - On 200 OK: Events are removed from the queue.
  - On 5xx Error: Events are kept in the queue for retry (with backoff).
  - On 4xx Error (e.g., 400 Bad Request): Events are dropped (to prevent infinite loops) and an error is logged.

### Scenario: Server Batch Passthrough
**Server tier only.**

- **Input:** Call a server batch method with an array of 101 valid payloads.
- **Expectation:**
  - The SDK submits the caller-supplied array as one request and does not
    split or reject it based on client-side batch-size policy.
  - Server configuration does not expose `maxBatchSize`.

## 4. Configuration

### Scenario: Disable Tracking
- **Input:**
  1. Initialize with `enabled: false`.
  2. Call `track("Event A")`.
- **Expectation:**
  - API call returns immediately.
  - No event is enqueued.
  - No network activity.

## 5. System Context

### Scenario: Automatic Context
- **Input:** Any `track` call.
- **Expectation:**
  - Payload `context` object is automatically populated with:
    - `library.name` (e.g., "altertable-swift")
    - `library.version`
    - `os.name`
    - `os.version`
    - `device.manufacturer` (if available)
    - `device.model` (if available)

## 6. Audience Collection

### Scenario: Public setup and wire policy
- Initialize an audience client using its public API and an environment-bound key. Track single events and raw batches where supported, including legacy identity/environment fields and records whose mode conflicts with the client.
- Observe final HTTP requests: only `mode`, `event`, `timestamp`, and `properties` are present; `mode` is always `audience`. Authentication uses the supplied key. Caller inputs remain unchanged.
- Invoke identify, alias, and trait updates with identity values that product mode rejects. No request or identity mutation occurs. Product callers retain their original identity validation and behavior.
- Use `audience_track_basic.json` and `audience_raw_batch.json` where their public interfaces apply. SDKs without a public timestamp argument should assert a valid current timestamp and compare the remaining fixture fields without introducing private clock hooks.

### Scenario: Context reduction after hooks (Web/Mobile)
- Supply credentialed URLs with queries/fragments, referrers with paths, viewport context, and safe business properties. Repeat via any public event transform and page/screen helper.
- Final `$url` is origin plus path; `$referer` is origin only; `$viewport` and automatically expanded query parameters are absent. Invalid/non-string URLs become null. Nested business properties remain intact. Use `audience_urls.json` for representative URL cases.
- Server SDKs: caller-supplied `$url`, `$referer`, and `$viewport` pass through unchanged.

### Scenario: Product state isolation and lifecycle (Web/Mobile)
- Seed product identity, consent, and queued events, then initialize audience mode with the same key and requested persistent/custom storage options.
- Audience does not read or mutate those stores. Events wait in memory for pending/dismissed consent, are dropped on denial, and never replay in a new client. Mode changes fail before other runtime updates take effect. Returning to a new product client preserves its original stored state.
- Pause a real or unavoidable external transport dependency while denial/disposal happens. Completion or retry of the old request must not restore cleared events or send later buffered audience events.

### Scenario: Cookie and credential isolation (Web/Mobile)
- Exercise public clients against an ephemeral real loopback HTTP server. Return Set-Cookie, then issue another event. Where redirects are supported, return a redirect with Set-Cookie and observe every hop. Where automatic retries are supported, return a retryable error before success.
- Audience requests never send Cookie/Cookie2 or ambient user authorization; audience does not retain server cookies or modify the product cookie jar. Required API-key authentication remains. A product client retains its existing transport behavior.
- Exercise caller-supplied session cookie/auth headers where the SDK supports session customization. Keep platform proxy/TLS behavior documented when audience isolation changes environment-derived settings.

### Scenario: Unsupported wrappers fail before side effects
- Through an untyped caller, pass audience or an unknown mode to a product-only wrapper. Observe no storage reads/writes, listeners, or HTTP requests before rejection.
- Reject an invalid mode update without changing the current delivery/consent behavior; subsequent automatic product delivery still succeeds.

## 7. Environment Deprecation

### Scenario: Compatible Inputs and Wire Omission
- **Input:** Initialize with a deprecated environment value, then exercise track, identify, alias, trait updates, and raw batch APIs where supported. Repeat in product and audience modes. Retain old positional/named constructor and event calls.
- **Expectation:** Calls remain source compatible. The environment option does not affect routing. All final request records omit top-level `environment`; hooks and raw dictionaries cannot restore it. Nested properties/traits named `environment` survive. Caller-owned inputs remain unchanged. The configured API key is delivered unchanged. The client does not expose the ignored environment value, and server SDKs write nothing to logs or stderr for it.
- **Fixture:** `environment_deprecation.json` provides a representative raw record before and after final serialization; apply the same omission to every endpoint.

### Scenario: Persistence Migration and Isolation (Web/Mobile)
- **Input:** Seed the selected legacy environment namespace, an unrelated environment namespace, and a different API key's namespace. Initialize using the same key and old option; repeat with no option to select the legacy production default.
- **Expectation:** Restore the selected identity and queue only. Current destination data takes precedence without merging. Unrelated stores remain intact. Removing/changing the deprecated option after migration does not reset identity or select another queue. Changing API keys isolates both stores. Legacy queued top-level environment fields never reach the wire.
- **Failures:** Exercise failed destination writes and source removal; retry must preserve identity and avoid lost or replayed queued records. No source deletion follows a failed copy.

### Scenario: Unowned Legacy Storage (Where Applicable)
- **Input:** Seed a legacy global identity and queue without key metadata. Initialize the intended existing product key, drain the queue, restart, then initialize a second key.
- **Expectation:** A durable one-time owner claim preserves the old state for the first key only. Existing destination data wins as a group. Restart cannot replay the drained legacy queue. Audience initialization does not claim or read the source. Failed migration writes remain recoverable for the recorded owner.

### Scenario: Required Key Migration
- **Input:** Deliver with an environment-bound Product Analytics key and no payload environment. Separately test the backend's rejection of an unbound legacy key without an environment.
- **Expectation:** The bound key selects its environment. Surface rejection through normal API error handling. Never reinsert a deprecated environment value as a fallback. Report live backend persistence separately from mock-server checks.
