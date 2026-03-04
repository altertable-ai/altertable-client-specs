# SDK Constants Reference

Canonical constant values for the Altertable Product Analytics SDK. Use these exact values when implementing any tier.

## Storage Keys

| Constant               | Value    |
| ---------------------- | -------- |
| `STORAGE_KEY_PREFIX`   | `"atbl"` |
| `STORAGE_KEY_SEPARATOR`| `"."`    |

Storage key format: `atbl.{apiKey}.{environment}`

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
