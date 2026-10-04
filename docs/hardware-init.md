# Hardware init (the `init` role)

Turns a stock, flashed Raspberry Pi OS card into a working PicoCalc — screen,
keyboard, audio — then reboots so the overlays load. It runs **first** in the
play; after the reboot, the system stages (`base`, `tooling`, …) continue on a
device that has a display and keyboard.

## What it does (translated from upstream, pinned)

| Step | What | From the profile |
|---|---|---|
| Display | panel firmware → `/lib/firmware`; `mipi-dbi-spi` overlay + geometry in `config.txt`; `fbcon` tokens in `cmdline.txt` | `display:` + `screen:` |
| Keyboard | build the `picocalc_kbd` module on-device, install it, compile the DT overlay, enable I2C + the overlay in `config.txt` | `keyboard:` |
| Audio | `dtparam=audio=on` + `audremap` overlay | `audio:` |
| Poweroff | `picopoweroff` helper + a systemd unit that cuts mainboard power on shutdown | `poweroff.enabled` |
| Reboot | once, only if any of the above changed, so the overlays take effect | — |

Every hardware parameter (GPIOs, SPI speed, panel size, overlays) lives in the
**device profile**, not the role — a different panel or board is a new profile.

## Provenance

The keyboard driver + panel firmware are **vendored** (not fetched at run time) and
**pinned** — see `upstream-refs.yml` and `roles/init/files/picocalc/UPSTREAM.md`.
We translate the upstream's manual steps into these idempotent tasks; we never run
its install script. The keyboard driver is GPL-2.0-only and keeps its license.

## Idempotency / re-runs

Re-running makes no changes and does **not** reboot — the `config.txt` block,
firmware, module, and overlay are all checked first. The module is built against
the running kernel; after a kernel upgrade, re-run `init` to rebuild it.

## First-run caveats

This is unverified on real hardware yet. The fragile points to watch on the first
flash: the kernel-headers package name (`raspberrypi-kernel-headers`) on your OS
release, and the module build succeeding against `/lib/modules/$(uname -r)/build`.
Watch it live with the `ssh … 'journalctl -f'` line the play prints.
