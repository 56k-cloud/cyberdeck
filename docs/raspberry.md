# Deck reference — handy TUIs & commands

A cheat sheet for operating the deck from its own 320×320 console. Not every tool
is installed on every build; the ones cyberdeck adds are marked *(cyberdeck)*.

## Network

| Command | What |
|---|---|
| `nmtui` | NetworkManager TUI — connect/edit WiFi, see status (the easy one) |
| `nmcli device wifi list` | scan for networks from the CLI |
| `iwgetid` | show the SSID you're on |
| `raspi-config` | Pi settings TUI — also has networking under *System Options* |

## System

| Command | What |
|---|---|
| `raspi-config` | the big Pi settings TUI — interfaces (SPI/I2C), boot/console, locale, overclock |
| `htop` | process/mem monitor (watch that 512 MB) |
| `alsamixer` | audio levels TUI |
| `bluetoothctl` | interactive Bluetooth control |
| `journalctl -f` | follow the system log live |

## Terminal

| Command | What |
|---|---|
| `tmux` | terminal multiplexer — split/detach; essential on one small screen |
| `nano` / `vim` | editors |
| `deck-font list` / `set <font>` | *(cyberdeck)* switch the console font (size vs. legibility on 320×320) |
| `deck-catalogue` | *(cyberdeck)* what's installed — repos + apt, with descriptions |

## Comms *(cyberdeck)*

| Command | What |
|---|---|
| `bbs-<name>` | dial a configured BBS (telnet) |
| `news-<name>` | read a configured Usenet server |
| `tin` | the Usenet reader itself |

## Python on the deck

- **Default env:** `~/.venvs/base` is always on PATH, so `python` has **`rich`**
  (pretty output) and **`textual`** (build TUIs) available with nothing to
  activate. Add more always-available libs via `packages.pip` in `config.yml`.
- **`curses` is built in** — `import curses` works out of the box (Python stdlib);
  no install needed. Use it for low-level TUIs, or `textual` for a high-level one.
- **Per-project envs:** `uv venv .venv && source .venv/bin/activate`, or just
  `uv run <cmd>` (runs in the project env, nothing to activate/deactivate).
- **Heads-up:** `source .venv/bin/activate` stays active for the whole shell
  session — `cd` elsewhere does *not* turn it off; run `deactivate` or use
  `uv run` / `direnv` if you want it tied to the directory.

## Finding more

There's no "list all TUIs" command (TUI isn't a package category). To explore:
`apt list --installed`, `deck-catalogue`, `compgen -c | sort` (every command —
noisy), and `man <cmd>`.
