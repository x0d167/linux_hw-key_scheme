# Recovery runbook: one key lost

Corresponds to the scenario walked through conceptually in
`03-architecture/recovery-design.md`. This page is the actual commands.
Do these in whatever order is convenient — none depend on the others
finishing first.

## 1. Revoke that card's GPG subkeys

Needs the offline master key backup, not the lost card:
```bash
export GNUPGHOME=$(mktemp -d)
gpg --import master-secret.asc
gpg --edit-key <keyid>
gpg> key N        # select the lost card's Sign subkey (check with 'list')
gpg> revkey
gpg> key N
gpg> key M        # the lost card's Encrypt subkey
gpg> revkey
gpg> key M
gpg> save
gpg --armor --export <keyid> > pubkey-updated.asc
unset GNUPGHOME
```

## 2. Re-upload the updated public key

Same single entry on GitHub/GitLab as always
(`04-implementation/06-git-hosting-config.md`) — paste
`pubkey-updated.asc` over the existing entry. Past commits signed by
the revoked subkey stay verified; new signatures under it will not
[17, 18, 19].

## 3. Remove that card's SSH key

Delete the corresponding entry from GitHub/GitLab's SSH keys list and
from any server's `authorized_keys` — the entry you added in
`04-implementation/03-fido2-ssh-keys.md`, identifiable by the label you
gave it.

## 4. Clean up per-machine enrollments, at your own pace

On each machine that had this card enrolled:
```bash
# List LUKS FIDO2 credentials to identify the right slot
sudo systemd-cryptenroll /dev/nvme0n1p2 --wipe-slot=<slot-number>

# Remove the corresponding line from ~/.config/Yubico/u2f_keys
```
This step is genuinely not urgent — the lost card can no longer sign or
authenticate as you (steps 1–3 handled that), so a stale LUKS or sudo
enrollment for it is inert, not exploitable, until someone has both the
physical card *and* physical access to that specific machine *and* its
PIN. Clean these up as you naturally get to each machine rather than
treating it as an emergency.

## 5. Update your tracking table

Mark the lost card's row in the table from
`03-architecture/key-topology.md` as retired.
