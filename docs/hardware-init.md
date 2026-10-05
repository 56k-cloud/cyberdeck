# Hardware init (the `init` role)

Turns a stock, flashed Raspberry Pi OS card into a working PicoCalc — screen,
keyboard, audio — then reboots so the overlays load. It runs **first** in the
play; after the reboot, the system stages (`base`, `tooling`, …) continue on a
device that has a display and keyboard.

## What it does (one step at a time, following the upstream guide)

Tasks are named by what they do — install display panel firmware, configure the
SPI overlay, build the keyboard driver, and so on:

| Group | Steps |
|---|---|
| display | panel firmware → `/lib/firmware`; `mipi-dbi-spi` overlay in `config.txt`; `fbcon` tokens in `cmdline.txt` |
| keyboard | build the `picocalc_kbd` module on-device, install it + `depmod`, compile the DT overlay, enable I2C + the overlay in `config.txt` |
| audio | `dtparam=audio=on` + `audremap` overlay (takes effect on the same reboot) |
| poweroff | `picopoweroff` helper + a systemd unit that cuts mainboard power on shutdown |
| verify | assert the module, firmware, and overlay exist, then write `/etc/cyberdeck/init-complete` |
| reboot | once, only if any of the above changed, so the overlays take effect |

The hardware values (GPIOs, SPI speed, panel size, overlays) are **inline in the
role** — this is a faithful, PicoCalc-specific translation. The role runs only when
the profile selects it with `hardware: picocalc`; a different panel is a different
init flow, not a parameter sweep. `/etc/cyberdeck/init-complete` records that the
stage ran (and the kernel it built against).

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

The fragile point is the kernel headers: current Raspberry Pi OS uses
`linux-headers-rpi-v8` (set in `roles/init/defaults/main.yml`). If apt can't find
it on your image, run `uname -r; apt-cache search '^linux-headers'` on the deck and
set `init_kernel_headers_pkg` to match. Then watch the module build live with the
`ssh … 'journalctl -f'` line the play prints.
