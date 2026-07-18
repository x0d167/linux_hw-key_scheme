# FIDO2: resident SSH keys per card

Repeat once per card, same as the GPG subkey step. This is independent
of the GPG provisioning above — order between the two doesn't matter,
they touch different applets on the same physical key
(`03-architecture/the-split.md`).

## Generate a resident credential

```bash
ssh-keygen -t ed25519-sk -O resident -O verify-required \
  -O application=ssh:sam-key1 \
  -f ~/.ssh/id_yk_key1
```
- `-O resident` — stores the credential entirely on the hardware token,
  not just a local key-handle file. This is what makes the
  instant-bring-up workflow in `05-bootstrap/` possible.
- `-O verify-required` — requires PIN entry in addition to touch, not
  just touch alone.
- `-O application=ssh:<label>` — a per-card label, useful for keeping
  multiple resident credentials on one token distinguishable later.
- `-f` — the local filename. This local file is a convenience copy;
  because the credential is resident, it can be regenerated from the
  hardware alone if lost (see below).

You'll be prompted to touch the key during generation, and to set/enter
its FIDO2 PIN if not already set (`ykman fido access change-pin` sets
one ahead of time if preferred).

## Register the public key

The command above produces `~/.ssh/id_yk_key1.pub`. Add this to:
- GitHub/GitLab: Settings → SSH Keys → New SSH Key
- Any server's `~/.ssh/authorized_keys`

Per `03-architecture/key-topology.md`, each card's distinct credential
means a distinct entry here — 4 cards, 4 SSH key entries. This is the
expected, intentional shape of this design, not a leftover mess to
clean up.

## Repeat for every remaining card

Same command, new label and filename per card (`id_yk_key2`, and so
on), one physical key plugged in at a time.

## Point SSH config at all of them

In `~/.ssh/config`:
```
Host github.com gitlab.com
    IdentitiesOnly yes
    IdentityFile ~/.ssh/id_yk_key1
    IdentityFile ~/.ssh/id_yk_key2
    IdentityFile ~/.ssh/id_yk_key3
    IdentityFile ~/.ssh/id_yk_key4
```
SSH tries each listed identity in order and uses whichever one has a
card physically present — you don't need to know or specify which
card you're carrying today.

## Bringing this to a brand new machine

This is the part that makes the whole scheme worth it. On a fresh
install, with `openssh-client`/`openssh` installed and *no* local key
files at all:

```bash
ssh-keygen -K
```
(Capital `K` — this discovers and downloads resident credentials from
whichever card is currently plugged in, writing the local stub files.
Not `ssh-add -K`, which is an unrelated legacy macOS Keychain flag and
does something different entirely.) Full walkthrough with the
GitHub-fetch trick for the *public* half in `05-bootstrap/`.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [6, 16].
