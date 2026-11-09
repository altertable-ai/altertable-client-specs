# Shared Test Fixtures

These JSON files define the canonical expected output for standard SDK operations. All Altertable Product Analytics SDKs should include tests that load these fixtures, execute the described input, and verify the resulting payload matches the output exactly.

## How to Use

1.  **Load the Fixture:** Parse the JSON file in your test suite.
2.  **Execute:** Call your SDK method using the `input` parameters.
3.  **Mock Context:** Ensure your test environment uses the fixed values (e.g., `timestamp`, `anonymous_id`, `$lib_version`) shown in the `output` to match the fixture.
4.  **Assert:** Compare the serialized JSON payload of your SDK's request against the `output.payload`.

For batch-flush fixtures, assert the sequence of endpoint requests, each request's payload count, and the combined payload order.

## Fixture Index

| File | Description |
| :--- | :--- |
| `identify_basic.json` | Standard identify call with traits. |
| `track_basic.json` | Basic track event with properties. |
| `track_null_properties.json` | Verifies that `null` input properties result in an empty object `{}`. |
| `alias_basic.json` | Standard alias call linking IDs. |
| `batch_flush_max_size.json` | Web/mobile default batch-size chunking and payload order. |

## Audience fixtures

| File | Description |
| :--- | :--- |
| `audience_track_basic.json` | Public audience track with reserved context reduction and retained business properties. |
| `audience_raw_batch.json` | Server raw batch with conflicting modes and legacy identity fields. |
| `audience_urls.json` | Representative URL/referrer inputs and final values (web/mobile only). |

Apply each fixture through the SDK's public client. Supply its timestamp where the public API accepts one; otherwise assert a valid current timestamp and compare the remaining fields. Use real loopback HTTP for cookie/redirect behavior as described in the test plan. Wrappers without audience support must reject the option before side effects rather than run these as supported-mode fixtures.
