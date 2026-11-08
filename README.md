# komga-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Komga](https://github.com/gotson/komga).

## What this is

A four-file TOS Docker application archive (`komgadocker.tar.gz`) that installs
Komga on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `komgadocker`)
- `komgadocker.lang` — 23-language superset (TOS requires 14)
- `komgadocker.svg` — application icon (326 bytes, 2 SVG elements)
- `docker-compose.yml` — one service, image `gotson/komga:1.30.0`

## How it works

| | |
|---|---|
| Container image | `gotson/komga:1.30.0` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8024/` (container port 25600) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/config` → `/Volume1/DockerAppData/komgadocker/config`, `/data` → `/Volume1/DockerAppData/komgadocker/data` |
| Logs | `docker logs komgadocker` (stdout/stderr) |

`config` holds the database, settings and thumbnails; `data` is the library root that contains your comics/books. Copying both folders is a complete backup.

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 25600.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.
- **Non-root.** Runs as `1000:1000`; verified on real hardware.
- **Two volumes, one port.** `/config` for state, `/data` for the library; only port 25600 is published.
- **Health check** probes `/` on 25600.
- **Fixed image tag**, Docker Hub only, no privileged mode, no host network, no Docker socket.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `config` | database, settings and generated thumbnails | first start | grows with the library | never overwritten; back it up |
| `data` | comics/books library | by the user | grows with every added file | owned by the user; back it up |
| `/Volume1/DockerAppData/komgadocker/config` | Komga application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |
| `/Volume1/DockerAppData/komgadocker/data` | Komga application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/komgadocker.tar.gz + .sha256
scripts/build.sh 1.30.0-2   # packaging iteration bump
scripts/build.sh 1.30.0-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/komgadocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v1.30.0-1`**, assets `komgadocker.tar.gz` +
   `komgadocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `komgadocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `1.30.0-1` → automated validation → review.

## License / branding

Komga is distributed by its upstream authors under **MIT**. The container image is the official image from Docker Hub (`gotson/komga:1.30.0`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

Komga does not create a comic library. Put your files into `DockerAppData/komgadocker/data` and add the library folder in the web UI.
The database is SQLite by default. Very large libraries may benefit from more memory; set `JAVA_TOOL_OPTIONS` if you need a larger heap.
Plain HTTP on the LAN; do not expose the published port to the internet without a TLS reverse proxy.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
