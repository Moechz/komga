# Changelog

All notable user-visible changes to the Komga TOS application package.

## 1.30.2-1 — 2026-11-25

Remediation of the App Center review of 1.30.0-6 (codes C4, C8, C9, T1 plus the blocking
items F7, S11 and V1 found in the same review).

### Changed

- **T1 — upstream upgrade `1.30.0` → `1.30.2`.** Newer base and security fixes from upstream:
  `1.30.1` adds CSP headers to the API/web UI, `1.30.2` honours
  `server.servlet.session.cookie.*` properties and fixes further API issues. The image is the
  multi-arch manifest list (`linux/amd64` + `linux/arm64`), so both packages keep working.
  **Migration note:** the SQLite schema is migrated forward automatically on first start (95
  migrations on a fresh database, incremental on an existing one). Back up
  `.../komgadocker/config` before upgrading; downgrading to 1.30.0 afterwards is not supported
  by upstream.
- **C4 — privacy policy completed** (`PRIVACY.md` rewritten). Now documents: the full data
  inventory with container and host paths, the retention period per data class, the user
  rights table (access, portability, rectification, erasure, restriction, consent withdrawal),
  the security measures (non-root uid, dropped capabilities, `no-new-privileges`, one port,
  pinned image, bcrypt), the third-party sharing rules, international transfers, and the
  deletion channel with step-by-step instructions. `tosapp.json.help` and `config.ini.help` now
  point at that public URL.
- **C8 — third-party API disclosure added** (`PRIVACY.md` §3, summary in `README.md`). Table of
  endpoint → data sent → destination/region → when it happens: the GitHub release check
  (`api.github.com`, no app data), Rakuten Kobo (`storeapi.kobo.com` and related hosts, only
  when Kobo Sync is enabled and a device is paired, forwarding the device's store requests),
  Docker Hub image pulls, and an optional OIDC/OAuth2 provider. The policy also states
  explicitly that Komga 1.30 has **no** online metadata scraper, so no ComicVine / AniList /
  MyAnimeList / Amazon traffic exists.
- **C9 — description now matches the delivered functionality.** The previous text claimed the
  well-known default login `admin@example.com / admin`. Komga 1.30 has **no default account**:
  the server starts unclaimed (`GET /api/v1/claim` → `{"isClaimed":false}`) and the first
  visitor creates the administrator, so the text was wrong and is now corrected in all 23
  language sections (accurate English plus rewritten `zh-cn` and `zh-hk`). No ASCII `;` is used
  anywhere in the language values, so the store parser cannot truncate them.
- **F7 — web interface speed and install reliability.** `JAVA_TOOL_OPTIONS` turns on Spring
  Boot gzip compression for the reader (initial JavaScript goes from ~3.5 MB to ~0.85 MB, which
  is what the “first paint slower than 5 seconds” criterion measures), and the health check now
  uses a 180 s `start_period` with 5 retries so a NAS-cold JVM start of 40–60 s no longer shows
  up as a failed installation.
- **S11 / V1 — least privilege.** The container drops **all** capabilities
  (`cap_drop: [ALL]`) and sets `no-new-privileges` on top of the pinned `user: "1000:1000"`.
  `docs/ROOT-PRIVILEGE.md` records the analysis: the upstream image declares no `USER`, the
  package therefore pins uid 1000, and the running container was verified to be uid 1000 with
  an empty capability set.
- `README.md` rewritten: how it works, first start without a default account, privacy and
  third-party summary, data/backup/uninstall, repository layout, build and verification.
- `NOTICE.md`: corrected image tag, added the base-image components (Ubuntu 26.04, Temurin
  JDK 25, kepubify, libheif/libwebp/libjxl), and aligned the packaging licence with the
  repository's `LICENSE` (MIT, not GPL-3.0).

### Fixed

- `tosapp.json` version and image were out of sync with the released package (it still said
  `1.30.0-4`); both now follow the released version.
- Removed the stale duplicated design bullets and the `1.30.0-1` submission checklist from the
  README.

## 1.30.0-2 — 2026-11-12

### Changed

- Icon: replaced the letter monogram with a functional vector mark that describes what the app does (the upstream project does not publish a compliant SVG).

## 1.30.0-1 — 2026-11-09

### Added

- First release. Komga (`gotson/komga:1.30.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8024 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/komgadocker/`.
