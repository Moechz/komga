# Root privilege analysis — `komgadocker` (S11 / V1)

Response to the review items of the 1.30.0-6 submission:

- **S11 — “File permissions are not minimised: run with a non-root USER in the Dockerfile;
  where `USER=root`, the item is handled as V1.”**
- **V1 — “Application runs as root.”**

This document records what the package actually does, the measurement that backs it up, and
why no root privilege exemption is being requested.

## 1. Conclusion first

The container **does not run as root**. It runs as **uid 1000 (gid 1000) with an empty
capability set and `no_new_privs=1`**. No root exemption is required and none is applied for.

The review item exists because of the **image metadata**, not the runtime:

```
$ docker inspect gotson/komga:1.30.2 --format '{{json .Config.User}}'
""                      # upstream Dockerfile sets no USER -> the image default is root
```

The upstream image is the officially published one and must be used unmodified (platform rule:
“official image, used as published”). It is pulled from Docker Hub at install time, it is not
rebuilt and not redistributed by this package. Its `Dockerfile` ends at `ENTRYPOINT` on
`ubuntu:26.04` without a `USER` line, so the *image* has no baked-in user. A packaging layer
cannot change that field without rebuilding and re-publishing the image, which the platform
rules forbid for this app.

What the packaging can do — and does — is pin the user at run time, so the **container** never
gets root.

## 2. What the package sets

```yaml
user: "1000:1000"                 # never uid 0
cap_drop:
  - ALL                           # empty capability bounding set
security_opt:
  - no-new-privileges:true        # setuid binaries cannot raise privileges
```

together with (already in previous versions): no `privileged`, no `network_mode: host`, no
`cap_add`, no `/var/run/docker.sock`, no host devices, a single published port
(8024 → 25600), and both volumes below `/Volume1/DockerAppData/komgadocker/`.

## 3. Measurement (run on TOS 7 hardware, x86_64)

```console
$ docker inspect komgadocker --format 'User={{.Config.User}} CapDrop={{json .HostConfig.CapDrop}} CapAdd={{json .HostConfig.CapAdd}} SecurityOpt={{json .HostConfig.SecurityOpt}} Privileged={{.HostConfig.Privileged}} NetMode={{.HostConfig.NetworkMode}}'
User=1000:1000 CapDrop=["ALL"] CapAdd=null SecurityOpt=["no-new-privileges:true"] Privileged=false NetMode=bridge

$ docker exec komgadocker id
uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu)

$ docker exec komgadocker sh -c 'grep -E "^(CapEff|CapBnd|NoNewPrivs|Uid)" /proc/self/status'
Uid:      1000  1000  1000  1000
CapEff:   0000000000000000
CapBnd:   0000000000000000
NoNewPrivs: 1

$ docker exec komgadocker sh -c 'touch /config/.w && rm /config/.w && touch /data/.w && rm /data/.w'
(exit 0 — the app user owns and can write both volumes)

$ stat -c '%u:%g %a %n' /Volume1/DockerAppData/komgadocker/config
1000:1000 755 /Volume1/DockerAppData/komgadocker/config
```

The same three assertions are part of the automated gates:

- `stage.py` (isolated runtime check) asserts `Config.User` is `1000:1000`, that the container
  becomes healthy and that HTTP answers on the mapped port;
- `test/install-and-verify.sh` (on-device, official installer) additionally asserts the in-
  container `id -u` is 1000, that **both** volumes are owned by uid 1000 and that the service can
  write into each of them.

## 4. Least-privilege plan (as shipped)

| Control | Setting | Effect |
|---|---|---|
| User / group | `user: "1000:1000"` | uid 0 is never used, in any container of this app |
| Capabilities | `cap_drop: [ALL]` | `CapEff` and `CapBnd` are both `0`; no `CAP_NET_BIND_SERVICE`, no `CAP_CHOWN`, nothing |
| Privilege escalation | `no-new-privileges:true` | setuid/setgid binaries and file capabilities are inert |
| Privileged mode | not set (false) | no host device access, no host kernel namespaces |
| Networking | bridge, one published port 8024 → 25600 | no host network, nothing else reachable |
| Docker socket | not mounted | the container cannot control the NAS Docker daemon |
| Host mounts | `/Volume1/DockerAppData/komgadocker/config` → `/config`, `.../data` → `/data` | the only writable host paths; both are the app's own data folder |
| Container filesystem | writable (upstream image writes to `/tmp` and `/app`) | evaluated `read_only: true` + `tmpfs /tmp` and rejected: upstream is not tested that way and the JVM/kepubify may write scratch files; the container FS is a throwaway layer, the host is what matters |
| Listen port | 25600 (unprivileged) | no need for `CAP_NET_BIND_SERVICE`; the host mapping is done by Docker |

**Writable paths (complete list):** `/config`, `/data` (host: the app data folder, owned by the
app uid) and the container's own `/tmp`. No other host path is writable, and no host path is
mounted read-write besides those two.

## 5. Why no non-privileged alternative exists for the image metadata

| Option | Verdict |
|---|---|
| Use an image that declares `USER` in its Dockerfile | No such official image exists. `gotson/komga` publishes a single multi-arch image; there is no `-rootless`/`-nonroot` variant on Docker Hub |
| Rebuild the image with `USER 1000` and publish it | Forbidden: the platform requires the official image, unmodified, pulled from Docker Hub. It would also move the trust anchor away from the upstream maintainer |
| Run as root and chown in an entrypoint | Worse on every axis: it grants root for the lifetime of the entrypoint and adds a shell script to the privileged path |
| **Pin the user in the compose file (chosen)** | Achieves the same end state — the application process is uid 1000 with no capabilities — without rebuilding or redistributing the image. This is also the usage the upstream project documents for running the container unprivileged |

## 6. Residual risk

- The image's metadata still says “no USER”, so a tool that only reads
  `docker inspect <image>.Config.User` will report “runs as root”. This document and the
  runtime measurement above are the answer to that: the **container** the App Center starts is
  uid 1000 with an empty capability set.
- The app is a media server, so it necessarily reads the library you point it at. That library
  lives inside the same app data folder, so a compromise cannot reach beyond the folder the app
  already owns.
- Passwords are stored as bcrypt hashes. There is no default account and no default password, so
  there is no well-known credential to exploit before the administrator is created.

## 7. Reproduction

```bash
# build & install the package, then:
docker inspect komgadocker --format '{{.Config.User}}'        # 1000:1000
docker exec komgadocker id -u                                 # 1000
docker exec komgadocker sh -c 'grep -E "^(CapEff|NoNewPrivs)" /proc/self/status'
docker inspect komgadocker --format '{{json .HostConfig.SecurityOpt}}'
```

Automated equivalents: `python3 stage.py work/komga` and
`bash test/install-and-verify.sh out/komgadocker_x86_64.tar.gz`.
