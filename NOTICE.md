# NOTICE — third-party components and attributions

This package (`komgadocker`, TOS 7 Docker application) is an independent packaging by Moechz of
the upstream **Komga** project. It contains **no upstream source code**: the application is
pulled from Docker Hub at install time and used unmodified.

## 1. Upstream application — Komga

- Project: <https://github.com/gotson/komga>
- Maintainer: gotson and the Komga contributors
- Licence: **MIT** (verified in the upstream repository's `LICENSE` file)
- Relationship: the application runs unmodified inside the container image. This package only
  adds a compose file, App Center metadata, translations and an icon.

## 2. Container image — `gotson/komga:1.30.2`

- Registry: Docker Hub (`docker.io/gotson/komga:1.30.2`)
- Published by the upstream maintainer, not by this package. The image is pulled by Docker on
  the NAS at install/upgrade time and is **not redistributed** in this archive.
- Multi-architecture: the tag is a multi-arch manifest list that contains `linux/amd64`
  (x86_64) and `linux/arm64` (aarch64), so the same tag serves both TOS packages.
- Base and bundled components inside the image (licences as shipped by upstream):
  - Ubuntu 26.04 LTS (base userland, various licences)
  - Eclipse Temurin JDK/JRE 25 (GPL-2.0 with the Classpath Exception)
  - [kepubify](https://github.com/pgaskin/kepubify) (MIT) for Kobo KEPUB conversion
  - the Java library set in `/app/lib` (Spring Boot, jOOQ, Flyway, Lucene, imageio, Thumbnailator,
    etc. — Apache-2.0, EPL-2.0, MIT/BSD variants; see the image's own `lib` directory and each
    project's licence)
  - `libheif`, `libwebp`, `libjxl` (LGPL/BSD) for HEIF/WebP/JPEG-XL cover support

## 3. This packaging

The files authored for this package — `docker-compose.yml`, `config.ini`,
`komgadocker.lang`, `komgadocker.svg`, the build scripts and the documentation
(`README.md`, `PRIVACY.md`, `NOTICE.md`, `CHANGELOG.md`) — are released under the **MIT**
licence, matching the `LICENSE` file in this repository (Copyright © 2026 Moechz).

## 4. Trademarks

“Komga”, “TerraMaster” and “TOS” are trademarks or registered trademarks of their respective
owners. Their use here is purely descriptive (to describe compatibility and origin). This
package is **not affiliated with, sponsored or endorsed by** the upstream project or
TerraMaster.

## 5. Icon

`src/komgadocker.svg` is a vector mark drawn for this package (an open book with text lines on
a rounded tile). It is not the upstream logo and contains no third-party trademark. It is
released under the same MIT licence as the rest of the packaging.

## 6. No warranty

The package is provided “as is”, without warranty of any kind. The upstream project provides
the application; this packaging only delivers it to TOS 7.

## 7. Contact

Moechz — moechz@163.com — <https://github.com/Moechz/komga/issues>
