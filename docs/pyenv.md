# Python environments on the deck

Three layers, from "just works" to "fully isolated per project".

## 1. The default env — always there

`~/.venvs/base` is a uv-managed venv that's **always on PATH** (a `profile.d` hook
adds it at login). Out of the box:

- `python` is that env's interpreter,
- `rich` and `textual` are importable, plus anything you put in `packages.pip`,
- nothing to activate.

It's *on PATH*, not *activated* (no `VIRTUAL_ENV` set) — so a project env cleanly
takes over when you enter one, and you fall back to this when you leave.

## 2. Per-project, no activation — `uv run` (lean on this)

From a project directory:

```sh
uv run python app.py
uv run pytest
```

`uv run` executes in the project's env (creating/syncing it as needed) **without
activating anything** in your shell — no lingering state, nothing to `deactivate`.

## 3. Per-project, activated — venv or direnv

Manual venv:

```sh
uv venv                 # creates .venv with the default interpreter
source .venv/bin/activate
# ...
deactivate              # stays active until you deactivate or close the shell —
                        # `cd` elsewhere does NOT turn it off.
```

### Auto-activate with `direnv`

`direnv` activates on `cd` **in** and deactivates on `cd` **out**. The binary is
installed by the `workstation` role; add the shell hook once (this comes via your
dotfiles, or add it by hand for now):

```sh
# ~/.zshrc   (or ~/.bashrc)
eval "$(direnv hook zsh)"
```

Then drop a `.envrc` in the project directory:

```sh
# .envrc — use this project's uv venv automatically
[ -d .venv ] || uv venv
export VIRTUAL_ENV="$PWD/.venv"
PATH_add "$PWD/.venv/bin"
```

and trust it once (direnv refuses to run an untrusted `.envrc`):

```sh
direnv allow
```

### How this interacts with the default env

The default env (#1) is always on PATH; you never "activate" it. When you `cd`
into a direnv project, direnv prepends `.venv/bin` **on top** → `python` is now the
project's env. When you `cd` out, direnv unloads it → PATH reverts and you're back
on the default env.

So: **default is the fallback, direnv overrides inside the project, leaving the
directory restores the default.** Nothing is double-activated.
