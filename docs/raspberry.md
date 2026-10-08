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

- Default env `~/.venvs/base` is always on PATH, so `python` has **`rich`** +
  **`textual`** ready; `curses` is built in (stdlib, no install).
- Per-project work: `uv run <cmd>`, or a venv / `direnv`.
- Full guide — the default env, `uv run` vs activate, direnv `.envrc`, and the
  "venv stays active" gotcha: **[docs/pyenv.md](pyenv.md)**.

## Finding more

There's no "list all TUIs" command (TUI isn't a package category). To explore:
`apt list --installed`, `deck-catalogue`, `compgen -c | sort` (every command —
noisy), and `man <cmd>`.
