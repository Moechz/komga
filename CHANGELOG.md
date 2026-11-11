# Changelog

All notable user-visible changes to the Komga TOS application package.

## 1.30.0-2 — 2026-11-12

### Changed
- Icon: replaced the letter monogram with a functional vector mark that describes what the app does (the upstream project does not publish a compliant SVG).

## 1.30.0-1 — 2026-11-09

### Added
- First release. Komga (`gotson/komga:1.30.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8024 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/komgadocker/`.
