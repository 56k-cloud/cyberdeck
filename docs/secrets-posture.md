# Secrets posture

cyberdeck is a **public repo** that provisions a **device that leaves the house**.
Those two facts set the rules for every secret the project touches.

## The one-line rule

**No secrets in the *repo*; secrets on the *device* are fine — injected at
provision time, never committed.**

Your real values live only in `config.yml` (gitignored). The provisioning machine
reads them and pushes them straight onto the device (WiFi profiles, a `0600` env
file). No secret — plaintext or ciphertext — ever enters tracked files or history.

## No SOPS in this project

There is exactly one consumer of the secrets (your provisioning machine) and one
file (`config.yml`), so the project does not adopt SOPS or mint an encryption key.
If you want `config.yml` encrypted **at rest on your own machine**, use your
personal vault (age / pass / SOPS) — that is your choice and the project depends on
none of it. A cyberdeck encryption key living on a roaming device would only widen
the blast radius of a lost device.

## Two credential tiers

The deck operates as **your own dev identity** — treat it like a dev laptop
that happens to leave the house. What it may hold, and what it must never, split
cleanly:

| Tier | Examples | On the deck? |
|---|---|---|
| **Dev identity (revocable)** | its own on-device SSH keys; scoped GitHub + Gitea PATs; WiFi PSKs; a spend-capped Claude key | **Yes** |
| **Lab owner / infra** | the homelab's admin tokens, automation/root keys, the SOPS age key | **Never** |

Everything in the first tier is a **scoped, revocable device identity**: the deck
clones projects, pushes to forks, opens PRs, and SSHes into lab boxes to work
— exactly what a dev laptop does. None of it is the lab's keys-to-the-kingdom.

What makes this safe on a device that leaves the house is **revocability, not
harmlessness**: a lost deck is contained by revoking its identity — pull its
GitHub/Gitea PATs and drop its key from any machine's `authorized_keys`, and it can
do nothing — while the lab's owner credentials were never on it to begin with.
Residual risk, stated honestly: until you revoke, a lost deck can act as you (push,
open PRs, SSH where its key is trusted). That is the conscious tradeoff for
dev-machine parity, and the reason the lab-owner tier stays off it entirely.

## SSH keys: you provide them, Ansible copies them

- `ssh.authorized_keys` — **public** keys allowed to log *into* the deck (your
  laptop / "Neo" key). The `base` role installs them; the Pi Imager also injects
  one, so SSH-in works from first boot. Not secret.
- `ssh.identity_keys` — keypairs the deck uses to authenticate *out* (infra,
  Gitea, GitHub). You generate them where your key distribution already lives (the
  Terraform box) and point config at them; Ansible copies them onto the deck
  (private key `0600`). The private key is a secret — it rides in gitignored
  config, never in the repo.

Recommended but not enforced: use a **dedicated** cyberdeck key rather than your
everyday one. Your Terraform distributes its pubkey like any other, so you keep the
convenience — but a lost deck then means pulling *that one key* from the authorized
set, not rotating your main key across the fleet. Pair it with a passphrase for
at-rest protection.

## Known gaps (accepted, not hidden)

- Secrets sit **plaintext at rest on an unencrypted microSD** in a device that
  leaves the house. The mitigation is tier discipline + revocability, not disk
  encryption (full-disk crypto on a Pi Zero 2 W is not worth the cost). Treat a
  lost deck as "revoke the device's identities" — its GitHub/Gitea PATs, and its
  key from every machine's `authorized_keys`.
- `config.yml` is a single gitignored file with **no backup story** — lose the
  provisioning machine, lose the restore inputs. Back it up in your own vault.
