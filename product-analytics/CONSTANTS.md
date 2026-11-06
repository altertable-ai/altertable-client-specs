# SDK Constants Reference

Canonical constant values for the Altertable Product Analytics SDK. Use these exact values when implementing any tier.

## Storage Keys

| Constant               | Value    |
| ---------------------- | -------- |
| `STORAGE_KEY_PREFIX`   | `"atbl"` |
| `STORAGE_KEY_SEPARATOR`| `"."`    |

Web storage keys: `atbl.v2.{apiKey}.session` and `atbl.v2.{apiKey}.events`.

Legacy web keys: `atbl.{apiKey}.{environment}` and `atbl.{apiKey}.{environment}.events`. The deprecated option is used only to select these migration sources, defaulting to `production` when omitted. Mobile SDKs retain their native key encoding and use an API-key-scoped versioned namespace with equivalent migration rules.

`STORAGE_KEY_TEST` is built as `atbl.check` — used to verify storage availability.

## ID Prefixes

| Constant              | Value         |
| --------------------- | ------------- |
| `PREFIX_SESSION_ID`   | `"session"`   |
| `PREFIX_ANONYMOUS_ID` | `"anonymous"` |
| `PREFIX_DEVICE_ID`    | `"device"`    |

## Timing and Limits

| Constant                     | Value       | Notes                      |
| ---------------------------- | ----------- | -------------------------- |
| `AUTO_CAPTURE_INTERVAL_MS`   | `100`       | SPA URL polling interval   |
| `SESSION_EXPIRATION_TIME_MS` | `1_800_000` | 30 minutes in milliseconds |
| `MAX_QUEUE_SIZE`             | `1_000`     | Drop oldest on overflow    |
| `REQUEST_TIMEOUT_MS`         | `5_000`     | Web tier HTTP timeout      |

## Event and Property Names

| Constant               | Value            |
| ---------------------- | ---------------- |
| `EVENT_PAGEVIEW`       | `"$pageview"`    |
| `PROPERTY_LIB`         | `"$lib"`         |
| `PROPERTY_LIB_VERSION` | `"$lib_version"` |
| `PROPERTY_REFERER`     | `"$referer"`     |
| `PROPERTY_RELEASE`     | `"$release"`     |
| `PROPERTY_URL`         | `"$url"`         |
| `PROPERTY_VIEWPORT`    | `"$viewport"`    |

## Timing and Limits (Mobile)

| Constant                      | Value   | Notes                      |
| ----------------------------- | ------- | -------------------------- |
| `MOBILE_REQUEST_TIMEOUT_MS`   | `10_000` | Mobile tier HTTP timeout   |

## Config Interfaces and Defaults

### WebConfig

| Option             | Type                      | Default                   | Description                                                                       |
| ------------------ | ------------------------- | ------------------------- | --------------------------------------------------------------------------------- |
| `apiKey`           | `string`                  | _(required)_              | Environment-bound Product Analytics API key                                       |
| `mode` | `"product" \| "audience"` | `"product"` | Collection policy, fixed for the client lifecycle where supported |
| `baseUrl`          | `string`                  | `https://api.altertable.ai` | Override the API base URL                                                       |
| `environment` | `string` | _(deprecated)_ | Accepted and ignored for routing; use an environment-bound Product Analytics API key |
| `persistence`      | `"localStorage+cookie" \| "localStorage" \| "sessionStorage" \| "cookie" \| "memory"` | `"localStorage+cookie"` | Storage backend |
| `trackingConsent`  | `TrackingConsentState`    | `"granted"`               | Initial tracking consent state                                                    |
| `autoCapture`      | `boolean`                 | `true`                    | Automatically capture pageviews on SPA navigation                                 |
| `release`          | `string \| null`          | `null`                    | App release/version string attached to every event as `$release`                 |
| `onError`          | `(error: AltertableError) => void \| null` | `null`  | Callback invoked on SDK errors (must not throw)                                   |
| `debug`            | `boolean`                 | `false`                   | Enable verbose console logging                                                    |
| `requestTimeout`   | `number`                  | `REQUEST_TIMEOUT_MS`      | HTTP request timeout in milliseconds                                              |
| `maxBatchSize`     | `number`                  | `20`                      | Maximum payloads per request per endpoint; values below `1` are clamped to `1`   |

**`WEB_DEFAULTS`** (canonical values):

```
apiKey:           (none — required)
mode:             "product"
baseUrl:          "https://api.altertable.ai"
persistence:      "localStorage+cookie"
trackingConsent:  "granted"
autoCapture:      true
release:          null
onError:          null
debug:            false
requestTimeout:   5000
maxBatchSize:     20
```

### MobileConfig

| Option            | Type                   | Default                     | Description                                                        |
| ----------------- | ---------------------- | --------------------------- | ------------------------------------------------------------------ |
| `apiKey`          | `string`               | _(required)_                | Environment-bound Product Analytics API key                        |
| `mode` | `"product" \| "audience"` | `"product"` | Collection policy, fixed for the client lifecycle where supported |
| `baseUrl`         | `string`               | `https://api.altertable.ai` | Override the API base URL                                          |
| `environment` | `string` | _(deprecated)_ | Accepted and ignored for routing; use an environment-bound Product Analytics API key |
| `trackingConsent` | `TrackingConsentState` | `"granted"`                 | Initial tracking consent state                                     |
| `release`         | `string \| null`       | `null`                      | App release/version string attached to every event as `$release`  |
| `onError`         | `(error: AltertableError) => void \| null` | `null` | Callback invoked on SDK errors                               |
| `debug`           | `boolean`              | `false`                     | Enable verbose logging                                             |
| `requestTimeout`  | `number`               | `MOBILE_REQUEST_TIMEOUT_MS` | HTTP request timeout in milliseconds                               |
| `flushOnBackground` | `boolean`            | `true`                      | Flush queued events when app moves to background                   |
| `maxBatchSize`    | `number`               | `20`                        | Maximum payloads per request per endpoint; values below `1` are clamped to `1` |

**`MOBILE_DEFAULTS`** (canonical values):

```
apiKey:             (none — required)
mode:             "product"
baseUrl:            "https://api.altertable.ai"
trackingConsent:    "granted"
release:            null
onError:            null
debug:              false
requestTimeout:     10000
flushOnBackground:  true
maxBatchSize:       20
```

### ServerConfig

| Option           | Type                    | Default                     | Description                                                       |
| ---------------- | ----------------------- | --------------------------- | ----------------------------------------------------------------- |
| `apiKey`         | `string`                | _(required)_                | Environment-bound Product Analytics API key                       |
| `mode` | `"product" \| "audience"` | `"product"` | Collection policy, fixed for the client lifecycle where supported |
| `baseUrl`        | `string`                | `https://api.altertable.ai` | Override the API base URL                                         |
| `environment` | `string` | _(deprecated)_ | Accepted and ignored for routing; use an environment-bound Product Analytics API key |
| `release`        | `string \| null`        | `null`                      | App release/version string attached to every event as `$release` |
| `onError`        | `(error: AltertableError) => void \| null` | `null` | Callback invoked on SDK errors                              |
| `debug`          | `boolean`               | `false`                     | Enable verbose logging                                            |
| `requestTimeout` | `number`                | `5000`                      | HTTP request timeout in milliseconds                              |

**`SERVER_DEFAULTS`** (canonical values):

```
apiKey:         (none — required)
mode:             "product"
baseUrl:        "https://api.altertable.ai"
release:        null
onError:        null
debug:          false
requestTimeout: 5000
```

## Collection Modes

| Mode | Wire behavior | State |
| ---- | ------------- | ----- |
| `product` | Omit the mode field; use the product endpoint requirements | Existing tier defaults |
| `audience` | `mode: "audience"` on track records; omit identity and environment fields | Memory only; no persistent product state or cookie/credential caches |

Use language-native enums or string literal types. A wrapper that does not expose this option must reject an untyped audience request before side effects; see [supported interfaces](SPEC.md#supported-interfaces).

## Tracking Consent States

| Constant                    | Value        | Behavior                         |
| --------------------------- | ------------ | -------------------------------- |
| `TrackingConsent.GRANTED`   | `"granted"`  | Send events immediately          |
| `TrackingConsent.DENIED`    | `"denied"`   | Drop events, clear queue         |
| `TrackingConsent.PENDING`   | `"pending"`  | Queue events                     |
| `TrackingConsent.DISMISSED` | `"dismissed"`| Queue events (same as `pending`) |

## Reserved User IDs

Reject the following IDs. `RESERVED_USER_IDS` is matched case-insensitively; `RESERVED_USER_IDS_CASE_SENSITIVE` is matched exactly.

**`RESERVED_USER_IDS`** (case-insensitive):

```
anonymous_id, anonymous, distinct_id, distinctid, false, guest,
id, not_authenticated, true, undefined, user_id, user,
visitor_id, visitor
```

**`RESERVED_USER_IDS_CASE_SENSITIVE`** (exact match):

```
[object Object], 0, NaN, none, None, null
```
