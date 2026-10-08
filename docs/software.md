# Adding software: `git` and `downloads`

Software beyond apt goes in `config.yml` under two lists with the same shape:
**what to fetch** + **exactly what to run**. No build types, no guessing.

| You want | Use | Example |
|---|---|---|
| Source code from a repo (your projects, Python tools, scripts) | `git` | a Python CLI you `uv pip install` |
| A ready-made binary attached to a GitHub release | `downloads` | `aichat`, `glow`, `fzf` |

Rule of thumb: **anything compiled (Rust, Go, C) → `downloads`.** Building it on a
512 MB board (≈259 MB actually free) is not an option; the project's prebuilt
binary is.

## `git`

```yaml
git:
  - name: py-thing                                  # required
    url: "https://github.com/you/py-thing.git"      # required
    version: v1.2.0                                 # optional — see below
    dest: ~/src/py-thing                            # optional (this is the default)
    install:                                        # optional
      - uv venv .venv
      - uv pip install --python .venv/bin/python -r requirements.txt
      - ln -sf "$PWD/run.sh" ~/.local/bin/py-thing
    entrypoint: py-thing                            # optional — shown in deck-catalogue
    description: "An example tool"                  # optional — shown in deck-catalogue
```

### `version`: branch, tag, or commit

One field. Git resolves the name itself — branches, tags and commits share one
namespace, so `v1.2.0`, `main` and `7fd1a60…` all just work. What matters is
whether it **moves**:

| `version:` | Moves? | What a playbook run does |
|---|---|---|
| *(omitted)* | yes | follows the repo's default branch |
| a branch (`main`) | yes | pulls new upstream commits **and re-runs `install`** |
| a tag (`v1.2.0`) | no | nothing, until you change it |
| a commit SHA | no | nothing, until you change it |

Pin a tag or commit for anything you rely on; use a branch for your own projects
you want kept current.

A **GitHub release** is a tag plus files attached to it. To build the release
from source, use `git` with the tag as `version`. To use its prebuilt binary, use
`downloads` with the attached file's URL.

## `downloads`

```yaml
downloads:
  - name: aichat                                    # required
    url: "https://github.com/sigoden/aichat/releases/download/v0.30.0/aichat-v0.30.0-aarch64-unknown-linux-musl.tar.gz"
    sha256: "eb1cd0948569404c5d9d01c10b32b902e11f8231073315456454dec246bdf26e"
    install:                                        # required
      - tar -xzf "$FILE" -C ~/.local/bin aichat
    entrypoint: aichat
    description: "All-in-one LLM CLI — chat, REPL, shell assistant"
```

- **`$FILE`** is the downloaded file's path. Your `install` decides what to do
  with it (extract, move, `chmod +x`, …).
- **`sha256`** is required. The download is verified against it, and skipped on
  later runs when the file already matches.
- **Pick the right asset.** On a 64-bit Pi OS (`dpkg --print-architecture` →
  `arm64`) choose the `aarch64` / `arm64` file; `-musl` builds are fully static
  and the safest bet.
- **Getting the sha256:** use the checksum the release publishes if it has one;
  otherwise download it yourself and run `sha256sum <file>` (macOS:
  `shasum -a 256 <file>`).
- **Upgrading** = change `url` and `sha256` together.

## How `install` runs

- **In order, stopping at the first failure** — exactly like joining the lines
  with `&&`. Later lines never run after a failed one.
- **As you** (the device user), in a **bash login shell**, so it sees the same
  PATH you do: `~/.local/bin`, `uv`, the default python env.
- **Where:** in the clone dir for `git` (`$PWD` is the repo); in the download
  cache for `downloads` (use `$FILE`).
- **When:** only when something changed — the checked-out commit (`git`), the
  `sha256` (`downloads`), or **the `install` lines themselves**. Otherwise it's
  skipped. A failed install is retried on the next run.
- `~/.local/bin` exists and is on your PATH — the natural place to put a binary
  or a symlink to a launcher.

## What you see in the Ansible output

One TASK per step; one line per entry inside it:

```
TASK [downloads : Download (verified against sha256)] ***********
changed: [deck] => (item=aichat)
ok: [deck] => (item=glow)

TASK [downloads : Run install commands (in order, stop at first failure)] ***
changed: [deck] => (item=aichat)
ok: [deck] => (item=glow)
```

`ok` = already in place, nothing ran. `changed` = it just ran. An entry's whole
`install` list is one line. When one fails, its error output traces each command
as it ran, ending at the one that broke:

```
+ uv venv .venv
+ uv pip install --python .venv/bin/python -r requirements.txt
error: File not found: `requirements.txt`
```

## Seeing what's installed

```
$ deck-catalogue
GIT
  py-thing           7fd1a60    An example tool  [py-thing]
DOWNLOADS
  aichat             eb1cd094   All-in-one LLM CLI — chat, REPL, shell assistant  [aichat]
APT
  ...
```

## Migrating from `repos:`

The old `repos:` list with `build: pip|uv|make|custom` is gone. Rename `repos:` →
`git:` and write the steps out as `install:` lines:

| Old | New `install:` |
|---|---|
| `build: pip` | `uv venv .venv` · `uv pip install --python .venv/bin/python -r requirements.txt` |
| `build: uv` | `uv sync` |
| `build: make` + `make_target: setup` | `make setup` |
| `build: custom` + `commands: [...]` | the same commands |

If the old key is still in `config.yml`, the play stops with a pointer here
rather than guessing.
