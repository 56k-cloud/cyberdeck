# Vendored PicoCalc hardware sources

These files are vendored (copied, pinned) from upstream and translated into the
`init` role. They are **not** modified here; the role compiles/installs them on
the device. Pin and provenance are tracked in `upstream-refs.yml` at the repo root.

- **Canonical upstream:** `nibheis/picocalc_trixie` (Debian Trixie port), which
  builds on `wasdwasd0105/picocalc-pi-zero-2` (original keyboard driver + audio).
- **Vendored snapshot:** `d1ce84d8`, taken 2026-06-03. The bytes are committed
  here, so provenance does not depend on re-fetching anything.

## Licensing

- `keyboard/picocalc_kbd.c` and headers: **GPL-2.0-only** (see the SPDX header in
  the source). They retain their own license; this directory is a vendored GPL
  component inside an otherwise Apache-2.0 repository.
- `display/picomipi.bin` / `picomipi.txt`: panel init sequence for the
  `panel-mipi-dbi` driver.

## Updating

Bump the `pinned_sha` in `upstream-refs.yml`, re-vendor these paths at that SHA,
re-check the `init` role still matches, and record the change. Do not edit the
vendored files in place.
