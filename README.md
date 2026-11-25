# komga-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Komga](https://github.com/gotson/komga), the comic / manga / eBook media server.

## What this is

A four-file TOS Docker application archive (`komgadocker_<arch>.tar.gz`, one archive per
architecture) that installs Komga on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `komgadocker`)
- `komgadocker.lang` — 23-language superset (TOS requires 14)
- `komgadocker.svg` — application icon (pure vector, no text)
- `docker-compose.yml` — one service, image `gotson/komga:1.30.2`

The application itself is the upstream image, unmodified. This repository contains only the
packaging: compose file, metadata, translations, icon and documentation.

## How it works

| | |
|---|---|
| Container image | `gotson/komga:1.30.2` (Docker Hub, multi-arch: `linux/amd64` + `linux/arm64`) |
| Open | `http://<NAS-IP>:8024/` (container port 25600) |
| Runs as | non-root, `user: "1000:1000"`, `cap_drop: [ALL]`, `no-new-privileges` |
| Persistent data | `/config` → `/Volume1/DockerAppData/komgadocker/config`, `/data` → `/Volume1/DockerAppData/komgadocker/data` |
| Logs | `docker logs komgadocker` (stdout/stderr) and `.../komgadocker/config/logs/komga.log` |
| Health check | `curl` on `/` (port 25600), 180 s start-up window |

`config` holds the database, settings, thumbnails, search index and logs. `data` is the
library root that contains your comics/books. Copying both folders is a complete backup.

## First start — there is no default account

Komga ships **without** a default user and password. On the first visit the server is
“unclaimed” and the web UI asks you to create the administrator account yourself
(`GET /api/v1/claim` reports `{"isClaimed":false}` until then). There is nothing to change
afterwards and no well-known credential to abuse — but it also means the app is only
protected once you have created that account, so do it right after installing.

Then:

1. Put comics/manga/eBooks into `/Volume1/DockerAppData/komgadocker/data` (subfolders are
   fine, one folder per series is the usual layout).
2. In the web UI: **Libraries → Add library**, point it at `/data`, pick the file types and
   let the scan run.
3. Optional: enable OPDS, Kobo Sync or KOReader sync per user, and restrict accounts by age
   rating.

## Privacy

This application runs entirely on your NAS. **The publisher does not collect, receive or have
access to any data processed by the application.** There is no telemetry, no analytics, no
account and no publisher-operated server.

| Topic | Detail |
|---|---|
| Data processed | Library files, catalogue metadata, user accounts (bcrypt hashes), reading progress, collections/read lists, thumbnails, search index, logs |
| Where it is stored | `/Volume1/DockerAppData/komgadocker/config` and `.../data` on your NAS. Nothing is uploaded to the publisher |
| Outbound traffic | GitHub release-version check (GET, no app data), Rakuten Kobo **only if** you pair a Kobo device, Docker Hub image pull, and an OIDC provider **only if** you configure one. No metadata scraper, no telemetry |
| Retention | Until **you** delete it. Logs rotate (7 × 10 MB, 1 GB cap), sessions expire after 7 days |
| Your rights | Access/export: copy the data folder or open `config/database.sqlite`. Correct: edit in the UI. Delete: uninstall with “delete data” or remove the folder |
| Deletion channel | App Center uninstall with “delete data”, or stop the container and delete `/Volume1/DockerAppData/komgadocker/`. Assisted by e-mail **moechz@163.com** (answered within 30 days) |
| Security measures | Non-root uid 1000, all capabilities dropped, `no-new-privileges`, no privileged mode, no host network, no Docker socket, one published port, pinned image tag, bcrypt passwords |
| Children | Not directed at children |

**Full policy:** [PRIVACY.md](PRIVACY.md) — <https://github.com/Moechz/komga/blob/main/PRIVACY.md>

## Third-party services (API disclosure)

The complete table, with what is sent and to which country, is in
[PRIVACY.md §3](PRIVACY.md). Summary:

| When | Endpoint | Data | Region |
|---|---|---|---|
| Web UI loads (release check) | `https://api.github.com/repos/gotson/komga/releases` | HTTP GET only — no app data | GitHub (Microsoft), US |
| Kobo Sync enabled and paired | `storeapi.kobo.com`, `authorize.kobo.com`, `cdn.kobo.com`, `ereaderfiles.kobo.com`, `readingservices.kobo.com`, `discovery.kobobooks.com`, `purchasing.kobo.com` | the device's own store requests, forwarded by Komga: Kobo account token, device id, sync markers, store search terms | Rakuten Kobo, Canada / Japan |
| Install / upgrade | `registry-1.docker.io` or your mirror | anonymous image pull | Docker Hub, US |
| Only if you configure it | your OIDC/OAuth2 issuer | browser redirect flow, token exchange | your identity provider |

Komga 1.30 has **no built-in online metadata scraper** (no ComicVine, AniList, MyAnimeList,
MangaUpdates or Amazon calls). OPDS clients and KOReader devices talk to your own NAS. Block
outbound traffic for the container and everything still works locally.

## Data, backup and uninstall

- **Backup:** copy `/Volume1/DockerAppData/komgadocker/` (both `config` and `data`) while the
  container is stopped, or let a TOS backup job copy that single folder. It is the complete
  state — database, accounts, progress, thumbnails and your books.
- **Restore:** stop the app, copy the two folders back, start the app. The database schema is
  migrated forward automatically by Komga on start.
- **Uninstall:** App Center → uninstall. The data folder is **kept** by default; tick
  “delete data” (or delete `/Volume1/DockerAppData/komgadocker/` yourself) if you want the
  library, accounts and progress gone. Docker packages have no uninstall hook, so removing the
  folder is the only cleanup step — no files are left anywhere else on the NAS.
- **Upgrade from 1.30.0.x:** only the image tag changes. Komga migrates its SQLite database on
  first start, so a downgrade afterwards is not supported. Back up `config/` before upgrading.

## Design highlights

- **Non-root.** The upstream image declares no `USER`, so the package pins `user: "1000:1000"`,
  drops every capability and sets `no-new-privileges`. Verified on hardware: the container
  reports uid 1000 and writes only below its own data folder.
- **One port, two volumes.** Only 8024 → 25600 is published; nothing else is exposed.
- **Compressed web UI.** Spring Boot gzip compression is switched on through
  `JAVA_TOOL_OPTIONS`, cutting the reader's initial JavaScript from about 3.5 MB to about
  0.85 MB — this keeps the first paint quick on slow links and slow NAS hardware.
- **Health check with a realistic start-up window.** A cold JVM start takes 40–60 s on a NAS,
  so the check uses a 180 s `start_period` with 5 retries instead of failing the install.
- **Fixed image tag, never `:latest`** — Docker Hub only, no `privileged`, no
  `network_mode: host`, no Docker socket, no `cap_add`.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a
  timestamp, identical archives for the same inputs.

## Repository layout

```text
.
├── tosapp.json              # the app spec: image, ports, volumes, env, texts, translations
├── src/
│   ├── config.ini           # TOS App Center metadata (version/platform substituted at build)
│   ├── komgadocker.lang     # 23-language string table (rendered from tosapp.json)
│   ├── komgadocker.svg      # icon (vector, viewBox 0 0 512 512)
│   └── docker-compose.yml   # the single service
├── scripts/
│   ├── build.sh             # build one architecture -> out/
│   └── build-dual.sh        # build x86_64 + aarch64, run the official check on both
├── tools/verify.py          # review-standards self-check (hard gate in the build)
├── test/install-and-verify.sh  # on-device E2E (root, installs with the official tool)
├── docs/                    # review notes (root-privilege analysis)
├── PRIVACY.md               # privacy policy (C4/C7)
├── NOTICE.md                # licences and attributions (C2)
├── CHANGELOG.md
└── README.md
```

## Build

```bash
scripts/build-dual.sh 1.30.2-1     # both architectures + official -c check
scripts/build.sh 1.30.2-1 x86_64   # one architecture only
```

The build stages the four files, substitutes the version/platform placeholders, strips
CRLF/BOM/AppleDouble, runs `tools/verify.py` (archive layout, JSON validity, required fields,
14+ language coverage, 512-char limits, no `;` in language values, Docker-Hub-only image with a
fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data paths below the application data root, never the `/Volume*`
wildcard, comment-free sequence blocks, `x-app-meta` last, no literal secrets, icon size and
element limits) and then packs with a timestamp-free GNU tar.

## Local verification

```bash
./TOSAppSelfTestingTool -c out/komgadocker_x86_64.tar.gz       # official spec check
python3 stage.py work/komga --timeout 300                      # isolated runtime check
bash test/install-and-verify.sh out/komgadocker_x86_64.tar.gz  # on-device E2E (root)
```

The staging check rewrites the compose file to free ports and a temporary data folder, starts
the container, and asserts that it becomes healthy, answers HTTP on the mapped port and runs as
uid 1000. It never touches `/Volume1/DockerAppData`.

> Docker applications cannot be side-loaded through the App Center's manual install page (it
> accepts `.deb` only); the official self-testing tool's `-i` mode or `docker compose` are the
> only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + documentation).
2. Release tag `v1.30.2-1` with assets `komgadocker_x86_64.tar.gz` + `.sha256` and
   `komgadocker_aarch64.tar.gz` + `.sha256`.
3. Developer platform → Add Application: ID `komgadocker`, package type Docker, repository
   URL, one App record per architecture.
4. Version Management → add `1.30.2-1` for each architecture → automated validation → review.
5. The platform's parsed `file_hash` must equal the local `sha256sum` of the archive.

## Limitations

- Komga does not create a comic library. Put your files into `DockerAppData/komgadocker/data`
  and add the library folder in the web UI.
- The server is unclaimed until you create the admin account — install and first login must be
  done by someone who should own the instance.
- The database is SQLite by default. Very large libraries may benefit from more memory; set
  `JAVA_TOOL_OPTIONS` (for example to `-XX:MaxRAMPercentage=60`) in the compose file if needed.
- Plain HTTP on the LAN; do not expose port 8024 to the internet without a TLS reverse proxy.
- Kobo Sync makes Komga forward the device's store traffic to Rakuten Kobo; if you want an
  offline-only server, do not enable it.

## Support

- Issues and questions: <https://github.com/Moechz/komga/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licences: [NOTICE.md](NOTICE.md) · Changes:
  [CHANGELOG.md](CHANGELOG.md)

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
