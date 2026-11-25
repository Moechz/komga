# Privacy Policy — Komga for TerraMaster TOS 7 (`komgadocker`)

**Last updated:** 25 November 2026
**Policy version:** 1.30.2-1
**Applies to:** the TOS application `komgadocker` (package version 1.30.2-1 and later),
which installs and runs the upstream [Komga](https://github.com/gotson/komga) media server.

- Publisher / data controller for the packaging: **Moechz** — moechz@163.com
- Upstream software: Komga, MIT licence, <https://github.com/gotson/komga>
- Public URL of this policy: <https://github.com/Moechz/komga/blob/main/PRIVACY.md>

This policy covers **the package and its default configuration**. If you change the compose
file, mount extra folders, install your own reverse proxy or pair additional services, those
changes are yours and may have their own privacy behaviour.

---

## 1. Summary

- The application runs **entirely on your NAS**. The publisher does not receive, collect or
  have access to your library, accounts, reading progress or logs.
- **No telemetry, no analytics, no crash reporting, no advertising, no publisher-operated
  server** is involved in any part of the app.
- The app contacts **four kinds of third parties at most**, and only when the situation
  applies: GitHub (release-version check), Rakuten Kobo (only if you enable Kobo Sync for a
  Kobo device), Docker Hub (image pull at install/update time) and — only if you configure
  one — an OIDC/OAuth2 identity provider for login. Section 3 is the complete list.
- Komga 1.30 has **no built-in online metadata scraper**. It does not call ComicVine,
  AniList, MyAnimeList, MangaUpdates, Amazon or any similar service. Metadata comes from the
  files themselves (ComicInfo.xml, EPUB OPF) and from edits you make in the web UI.
- Everything you create can be deleted by you, on your NAS, without asking the publisher
  (section 8).

---

## 2. What data the app processes, and where it lives

All paths below are on your NAS, inside the application data folder
`/Volume1/DockerAppData/komgadocker/`. Nothing in this table is uploaded to the publisher.

| Data | Stored at (container path → host path) | Why it exists |
|---|---|---|
| Comics, manga, magazines and eBooks you put in the library folder | `/data` → `.../komgadocker/data` | The content the server serves |
| Library definitions, scan state, file hashes | `/config/database.sqlite` (`.../komgadocker/config`) | Scans, duplicate detection, incremental indexing |
| Series/book metadata (title, authors, tags, summary, age rating, release date) and the metadata you edited | `database.sqlite` | Catalogue and search |
| **User accounts:** e-mail address, display name, bcrypt password hash, roles, per-user content/age restrictions, API keys | `database.sqlite` | Authentication and authorisation |
| **Reading progress** per user and per book, read status, bookmarks | `database.sqlite` | Resume reading where you stopped |
| Collections and read lists | `database.sqlite` | Organisation features |
| Generated thumbnails and covers (stored as BLOBs) | `database.sqlite` | Fast library browsing |
| Search index (Lucene) | `/config/lucene` | Full-text search |
| Web sessions | `/config/sessions.sqlite` | Keep you logged in for up to 7 days |
| Task queue state | `/config/tasks.sqlite` | Background scan/analysis jobs |
| Komga and web-server logs | `/config/logs/komga.log` | Troubleshooting only (rotated: 7 files × 10 MB, 1 GB total cap) |
| Custom fonts you upload for the reader | `/config/fonts` | Reader appearance |
| Kobo Sync pairing data (device token, sync markers) and KOReader sync hashes | `database.sqlite` | Only if you use those sync features |
| Optional user settings file, if you create one | `/config/application.yml` | Overrides for advanced users |

The publisher has no copy of any of it.

---

## 3. Data sent off the NAS

Outbound traffic happens **only** in the cases below. Rows 1–2 are the default behaviour of the
unmodified upstream image. Rows 3–4 happen only when the described situation applies. Row 5
never happens.

| # | When | Endpoint | What is sent | Destination / region |
|---|---|---|---|---|
| 1 | Docker Engine pulls the image at install or upgrade | `registry-1.docker.io` (or the registry mirror configured on your NAS) | an anonymous image pull: image name and tag, your public IP address | Docker Hub, operated by Docker Inc., United States (or your mirror) |
| 2 | The web UI asks the server for the upstream release list (Komga shows it on the updates page and in the version banner) | `https://api.github.com/repos/gotson/komga/releases` | a plain unauthenticated `GET`; no app data, no account, no library content. GitHub sees the request itself: your NAS's public IP address, a timestamp and the `User-Agent` | GitHub Inc. (Microsoft), United States. Komga caches the answer, so this is a handful of requests per day at most |
| 3 | **Only if** you enable Kobo Sync and pair a Kobo e-reader with your account | `storeapi.kobo.com`, `authorize.kobo.com`, `cdn.kobo.com`, `ereaderfiles.kobo.com`, `readingservices.kobo.com`, `discovery.kobobooks.com`, `purchasing.kobo.com`, `www.kobo.com` | Komga emulates a Kobo store for the device, so the store requests the device would normally make are **forwarded** from your NAS: the Kobo account token the device holds, its device identifier, library/sync markers, search terms you type in the store, and the book ids it asks for. Your own library content and reading progress are served locally and are **not** sent | Rakuten Kobo Inc. (Canada) and Rakuten Group (Japan). Disable Kobo Sync (or unpair the device) to stop this traffic |
| 4 | **Only if** you configure an OIDC / OAuth2 login provider (for example Keycloak, Authentik, Google or Microsoft) | the issuer URL you configure | the login is a browser redirect flow: your browser talks to the provider, and Komga receives the returned identity (subject id, e-mail, display name, groups). Your NAS sends only the token exchange | the identity provider you chose, in whichever country it operates |
| 5 | Always | — | **no other endpoint is contacted.** No telemetry, no analytics, no ad network, no crash reporting, no publisher server, no third-party metadata provider. OPDS clients and KOReader devices talk to your own NAS only | — |

If you want to be certain that no traffic leaves the NAS, block outbound traffic for the
`komgadocker` container at your firewall or Docker network. The app then still works: rows 1
and 2 are optional, and rows 3–4 only exist if you enable those features. Installing and
upgrading the package needs Docker Hub access (row 1) from the Docker daemon, not from the
container.

---

## 4. Third-party sharing

The publisher shares nothing with anyone: it never receives your data in the first place.
The recipients that can see traffic are exactly the parties named in section 3, and each of
them only sees what that row lists.

- **No sale of data.** No data is sold, rented, traded or used for advertising.
- **No tracking pixels, no third-party JavaScript.** The web reader loads all of its assets
  from your own NAS.
- **No cross-device profiling.** Komga keeps user accounts and reading progress locally and
  does not correlate them with anything outside your NAS.

---

## 5. Retention period

| Data | Retention |
|---|---|
| Library files in `data/` | kept until **you** delete them. The app never deletes your files |
| Database, metadata, accounts, reading progress, thumbnails, index, fonts | kept until **you** delete the application data folder. Nothing expires automatically |
| Logs (`config/logs/komga.log`) | rotated by the application: at most 7 files × 10 MB (1 GB total), older entries are dropped automatically |
| Web sessions | valid for up to 7 days, then expire. Rows for expired sessions are cleaned hourly |
| Kobo / KOReader sync state | kept until you unpair the device or delete the data folder |
| Backups you take | controlled by your own backup policy — the publisher has none |

Uninstalling the app **keeps** `/Volume1/DockerAppData/komgadocker/` (both `config` and
`data`) unless you tick “delete data” in App Center or remove the folder yourself.

---

## 6. Security measures

- The container is **not root**: the package pins `user: "1000:1000"`, drops **all**
  Linux capabilities (`cap_drop: [ALL]`) and forbids privilege escalation
  (`no-new-privileges`). The upstream image declares no `USER`, so this pinning is what
  enforces the non-root uid; it was verified on hardware (`docker exec komgadocker id -u`
  returns `1000`).
- **No privileged mode, no host network, no Docker socket, no `cap_add`, no host device
  access.** The container can only reach its own two volumes.
- **One published port** (8024 → 25600). Everything else is unreachable from the LAN.
- **Authentication on by default.** There is no default account and no default password: on
  first start the server is unclaimed, and the first visitor creates the administrator. Passwords
  are stored as bcrypt hashes, never in clear text. API keys and per-user roles are optional
  and revocable.
- **Least data on disk.** The database files live under `config/`, owned by uid 1000, and are
  not world-readable. The package contains no secret, token or default credential.
- **Pinned image.** The package uses the fixed tag `gotson/komga:1.30.2`, never `:latest`, so
  an upgrade is a deliberate act. Security releases of upstream are picked up by rebuilding
  the package.
- **Transport.** The app serves plain HTTP on the LAN, because that is what the App Center
  opens. If you expose it beyond your LAN, put it behind an HTTPS reverse proxy (TOS has one
  in the built-in web server) and do not publish port 8024 to the internet.

---

## 7. International transfers

The app itself does not transfer data between countries. The three optional recipients in
section 3 (Docker Hub, GitHub, Rakuten Kobo) and any identity provider you add are located
outside your NAS, in the countries noted in section 3, and only receive the traffic listed
there. If you need to keep all traffic inside your jurisdiction, block outbound traffic for
the container (section 3, last paragraph).

---

## 8. Your rights and the deletion channel

You are the only party that holds your data, so you can exercise every right directly. The
publisher assists with questions at **moechz@163.com** (subject “Komga data request”, answered
within 30 days) but holds no copy of your data to act on.

| Right | How to exercise it |
|---|---|
| **Access / information** | Read the data directly: `config/` (database, thumbnails, logs) and `data/` (your books). `config/database.sqlite` is a standard SQLite file you can open with any SQLite client |
| **Portability / export** | Copy `/Volume1/DockerAppData/komgadocker/` — that folder is the complete state. `config/database.sqlite` is a standard SQLite file, so the catalogue and your reading progress can be read or exported with any SQLite client |
| **Rectification** | Edit series/book metadata, your account, e-mail address, password and restrictions in the web UI at any time |
| **Erasure** | Delete individual books/series in the UI, delete a user in the user settings, or delete the whole folder (see below) |
| **Restriction / objection** | Disable the optional features: unpair Kobo Sync, do not configure an OIDC provider, block outbound traffic, or stop the container |
| **Withdraw consent** | Every feature that causes outbound traffic is opt-in; disable it and the traffic stops |

**Deletion channel — step by step**

1. App Center → the installed `Komga` app → uninstall **with “delete data”** ticked
   (this removes the container and `/Volume1/DockerAppData/komgadocker/`). Or:
2. Stop the container (`docker compose -p komgadocker down`) and delete the folder
   `/Volume1/DockerAppData/komgadocker/` from the file manager or over SSH.
3. If you made backups, delete those copies too — the publisher cannot do it for you.

Deleting the folder removes the database, accounts, reading progress, thumbnails, search index
and logs. There is no server-side copy anywhere else to delete.

---

## 9. Children

Komga is a self-hosted media server for your own files. It is not directed at children and it
does not create profiles of minors. Administrators can restrict accounts by age rating
(content restrictions), which is enforced locally.

---

## 10. Changes to this policy

Material changes are published in this file, in `CHANGELOG.md` with the version that
introduced them, and in the release notes of the package. The “Last updated” date and the
policy version at the top always match the newest package release.

| Package version | Date | Change |
|---|---|---|
| 1.30.2-1 | 2026-11-25 | Rewritten for the C4/C8 review: retention, rights, security measures, third-party endpoints and deletion channel added, wrong “default admin account” statement removed |
| 1.30.0-1 | 2026-11-09 | First release |

---

## 11. Contact

- Publisher: **Moechz** — moechz@163.com — <https://www.moechz.com/privacy.html>
- Package issues: <https://github.com/Moechz/komga/issues>
- Upstream project (Komga) privacy questions: <https://github.com/gotson/komga/issues>
