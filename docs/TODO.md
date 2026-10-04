# TODO / ideas to discuss

Captured from field use — not yet designed/implemented.

## 1. SSH keys by path, not pasted

Let `ssh.authorized_keys` entries optionally be a **path to a `.pub` file** on the
control machine (read with `lookup('file', ...)`) instead of a pasted key string,
so you don't copy the key text into `config.yml`. (`ssh.identity_keys` already use
paths.) Accept either form per entry.

## 2. Generate the inventory from config

Username + IP currently live in **both** `config.yml` and `inventory/hosts.ini` —
editing two places per device, and a chore as devices multiply. Add a helper
(generator script, or a dynamic inventory) that builds the inventory from a
single `devices:` list (name / ip / user), so it's defined once and rebuilt.

## 3. `init` role: direct repo-flow, not over-parameterised

The hardware-init role should read as the upstream's segmented steps (install
display driver → edit config.txt → install keyboard driver → audio → poweroff)
done in Ansible, rather than pushing every value into the device profile. Keep the
profile lean (screen geometry, board). Rework tracked in the init PR.
