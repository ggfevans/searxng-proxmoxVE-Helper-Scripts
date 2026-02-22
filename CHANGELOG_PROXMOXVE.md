## What's Changed

### Network and resilience
*   **Switched from urllib.request to shared searx.network.get (httpx-style).**
*   Non-200 responses and network errors are treated as an empty catalogue (no startup failure).
*   Added unit tests that mock network errors and status codes.

### Payload validation and hardening
*   Top-level payload must be a list; otherwise the request is ignored with a warning.
*   Skips malformed categories (non-dict), malformed scripts (non-dict), and malformed scripts lists (non-list).
*   Validates that script name and slug are strings; logs and skips invalid ones.
*   Trims script names, normalizes slugs with a `_slugify` helper (NFKD, remove combining marks, lowercase, replace non-alnum with hyphen), collapses collisions with numeric suffixes.
*   Skips disabled scripts, truncates descriptions to 500 chars.
*   Many informative warning logs on malformed input.

### Per-item caching and integrity checking
*   Replaces single-cache blob with per-script cache entries:
    *   Serializes to JSON, compresses with zlib, optionally prefixes with HMAC.
    *   Stores signed+compressed bytes under keys `script_{slug}`, with a `script_slugs_list` index.
    *   Limits per-cache value size (`_MAX_CACHE_VALUE_LEN = 10 KB`) and skips oversized items.
*   On search, tries to rehydrate each script from per-item cache; corrupted or missing items are logged and counted. If none recover, engine re-fetches fresh data.
*   `setup()` loads or generates a per-instance HMAC key via: `engine_settings["hmac_secret_key"]`, env var `PROXMOXVE_CACHE_HMAC_KEY`, a `.hmac_secret` file, or by generating/storing a new key file.

### Other
*   Added many unit tests covering network failures, malformed payloads, trimming/deduping, and partial-cache resilience.
*   Added a manual (skipped) analysis script in `tools/` for inspecting live API data and compression.

## Why this is beneficial
*   **More robust against unstable remote API:** startup/search won't crash when API is unreachable or returns unexpected payloads.
*   **Cleaner, predictable search results:** names trimmed, slugs normalized and de-duplicated, disabled items omitted.
*   **Per-item caching:** prevents a single corrupted cache blob from invalidating the entire catalogue.
*   **Better logging:** makes diagnosing malformed upstream data easier.
*   **Tests:** improve confidence in the new behavior.
