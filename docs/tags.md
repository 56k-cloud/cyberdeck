# Running part of the playbook: tags

A full run walks every role, even the ones with nothing to change. To run only
what you're working on, use **tags**: every role in `site.yml` carries its own
name plus a slice.

| Tag | Roles | What it is |
|---|---|---|
| `core` | init, base, wifi, tooling, console_fonts, lock, just, identity, creds, dotfiles, smallscreen, tailscale | a secure, connected terminal |
| `extras` | workstation, git, downloads, catalogue, comms | what you install and iterate on |
| `access` | identity, creds, tailscale | overlay: the revocable credentials (also in `core`) |
| `<role name>` | that one role | e.g. `downloads` |

```bash
ansible-playbook site.yml -K --list-tags                    # what tags exist
ansible-playbook site.yml -K --tags extras --list-tasks     # what a tag WOULD run
ansible-playbook site.yml -K --tags downloads,catalogue     # just downloads (+ refresh deck-catalogue)
ansible-playbook site.yml -K --tags git,catalogue           # just git
ansible-playbook site.yml -K --tags extras                  # everything you install
ansible-playbook site.yml -K --tags access                  # after rotating a PAT / key / tailnet auth key
ansible-playbook site.yml -K --skip-tags init               # everything except the hardware step
ansible-playbook site.yml -K --tags downloads --check --diff  # dry run: what would change
```

How tags behave:

- `--tags X` runs what's tagged `X`, plus the setup steps tagged `always` (version
  check, loading the device profile). Untagged steps — e.g. the closing success
  banner — are skipped.
- A role can carry several tags; `access` overlaps `core` on purpose.
- Tags filter, they never reorder: a filtered run still goes top to bottom.
- A filtered run assumes the rest ran once before. `extras` needs a deck that
  `core` already provisioned.
- After changing `git` or `downloads`, add `catalogue` so `deck-catalogue` lists it.
