# Decision Ledger — Youtube-Channel-Blocker

Durable record of the significant decisions made in this repository and the reasoning behind them.

- **Confirmed** decisions are human-reviewed and binding. This section is maintained by the repository owner; the automated decision-ledger pass never edits it.
- **Inferred** decisions are hypotheses proposed automatically from the code, commit history, and any agent instructions (CLAUDE.md / AGENTS.md). They are **not binding** until the owner moves them into Confirmed.

## Confirmed

_None yet. Merge a proposal from Inferred to confirm it._

## Inferred (proposed — awaiting confirmation)

> Every item below is a hypothesis generated automatically on 2026-06-21. Where the rationale could not be recovered from the available evidence it is marked "rationale unknown — please supply".

### [hypothesis] Build as a Chrome extension on Manifest V3
- **Decision:** Ship the product as a browser extension targeting Chrome's Manifest V3 (`"manifest_version": 3`), with a popup action UI plus content scripts injected into YouTube.
- **Rationale (hypothesis):** Manifest V3 is the only manifest version Chrome accepts for new Web Store submissions, so a new extension must target it.
- **Evidence:** `Chrome_Extension_YTChannelBlocker_1.20.zip` → `Chrome/manifest.json` (`manifest_version: 3`, `action`/`default_popup`, `content_scripts`); same in v1.0 (`youtube-channel-blocker.zip` → `manifest.json`); commit `c977090` "Youtube Channel Blocker Chrome Extension 1.0".
- **First observed:** commit `c977090` (2026-02-21)

### [hypothesis] Local-only, zero-server data handling (no tracking/analytics)
- **Decision:** All user data (the blocked-channels list) is stored locally on the device via the browser's extension storage; nothing is sent to any server, and there is no analytics, advertising, or tracking.
- **Rationale (hypothesis):** Stated as a deliberate privacy posture in the published Privacy Policy ("No data leaves your device. No tracking, no analytics, no ads.").
- **Evidence:** `Privacy-Policy` (sections "In short", "Data sharing", "Third-party services"); implementation uses `chrome.storage.local` only — `Chrome/shared.js` lines 146, 169, 181; commits `afac4db` (create) and `5b1cbcb` (update Privacy-Policy).
- **First observed:** commit `afac4db` (2026-02-21)

### [hypothesis] Persist blocked channels with `chrome.storage.local` (not `storage.sync`)
- **Decision:** The blocked list is persisted in `chrome.storage.local` rather than `chrome.storage.sync`.
- **Rationale (hypothesis):** rationale unknown — please supply
- **Evidence:** `Chrome/shared.js` lines 146/169/181 use `chrome.storage.local` exclusively; manifest declares the `storage` permission.
- **First observed:** commit `7688b2b` (2026-04-06)

### [hypothesis] Versioned storage schema with legacy migration
- **Decision:** Blocked-channel state is stored under key `blockedChannelsState` with an explicit `STORAGE_VERSION = 2`, and a `migrateStateIfNeeded()` routine upgrades data from the legacy key `blockedChannels`, removing the legacy key after migration.
- **Rationale (hypothesis):** Introducing a versioned schema with a migration path preserves existing users' blocked lists across the data-model change between v1.0 and v1.2.
- **Evidence:** `Chrome/shared.js` lines 3–4 (`STORAGE_KEY`, `LEGACY_STORAGE_KEY`, `STORAGE_VERSION = 2`), lines 145–170 (`migrateStateIfNeeded`); the v1.0 build (`youtube-channel-blocker.zip`) has no `shared.js`, indicating the versioning was added later.
- **First observed:** commit `7688b2b` (2026-04-06)

### [hypothesis] Channel-identity model based on normalized URL aliases
- **Decision:** Channels are identified by normalized alias keys derived from YouTube URL paths — `handle:` for `/@name`, `channel:` for `/channel/ID`, `user:` for `/user/name`, `custom:` for `/c/name` — produced by URL/path normalization helpers.
- **Rationale (hypothesis):** YouTube exposes channels through several URL forms; normalizing them to canonical alias keys lets the blocker match the same channel regardless of which link form appears in feeds, search, or recommendations.
- **Evidence:** `Chrome/shared.js` `normalizeText`/`normalizeAbsoluteUrl`/`normalizePath`/`normalizeAliasValue`, `aliasFromPath` (handle/channel/user/custom mapping), `aliasFromHref`.
- **First observed:** commit `7688b2b` (2026-04-06)

### [hypothesis] Shared logic extracted into `shared.js`, loaded before `content.js`
- **Decision:** Common normalization/storage logic lives in `shared.js`, which is listed first in the content-script `js` array and is also reused by the popup, rather than duplicating it.
- **Rationale (hypothesis):** Centralizing alias/storage logic in one module shared by the content script and popup avoids divergence between how each surface reads and matches the blocked list.
- **Evidence:** `Chrome/manifest.json` `content_scripts.js: ["shared.js", "content.js"]`; `shared.js` exists in v1.20 but not in the v1.0 build (`youtube-channel-blocker.zip` content scripts were `["content.js"]` only).
- **First observed:** commit `7688b2b` (2026-04-06)

### [hypothesis] Narrowed permissions in v1.2 (dropped `tabs`)
- **Decision:** The extension's permission set was reduced from `["storage", "tabs", "activeTab"]` in v1.0 to `["storage", "activeTab"]` in v1.2.
- **Rationale (hypothesis):** rationale unknown — please supply
- **Evidence:** v1.0 `manifest.json` `permissions: ["storage", "tabs", "activeTab"]` vs v1.2 `Chrome/manifest.json` `permissions: ["storage", "activeTab"]`.
- **First observed:** commit `7688b2b` (2026-04-06)

### [hypothesis] Cross-browser target: Chrome plus Safari Web Extension
- **Decision:** Support both Chrome and a Safari Web Extension for macOS and iOS, with permission models documented per platform.
- **Rationale (hypothesis):** Stated intent to distribute on both the Chrome Web Store and the Apple App Store, with the Privacy Policy describing each platform's permissions and data-deletion steps.
- **Evidence:** `Privacy-Policy` (intro "available as a Chrome extension and as a Safari Web Extension for macOS and iOS"; per-platform "Permissions", "How to delete your data", "Apple privacy" sections).
- **First observed:** commit `afac4db` (2026-02-21)

---
*Decision-ledger automated pass. Operation: Bootstrap. Last reflection: commit `5b1cbcb` (2026-06-21). Decisions above are AI-inferred hypotheses; nothing is binding until merged into Confirmed.*
