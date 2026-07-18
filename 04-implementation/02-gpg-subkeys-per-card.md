# GPG: distinct Sign + Encrypt subkeys per card

Repeat this entire section once for **each** physical YubiKey. Per
`03-architecture/key-topology.md`, no key material is shared between
cards — every card gets its own freshly generated Sign and Encrypt
subkey pair, both certified by the one master key from the previous
step.

Note what's *not* here: an Authenticate subkey. This scheme routes SSH
through FIDO2 instead — see `03-architecture/the-split.md` for why, and
`04-implementation/03-fido2-ssh-keys.md` for that half.

## Confirm which physical card is plugged in

Before writing anything, avoid the easy mistake of provisioning the
wrong card:
```bash
ykman info
```
Note the serial number and cross-reference against your tracking table
(`03-architecture/key-topology.md`) before proceeding.

## Generate this card's subkey pair

With the master key material available (either on your normal keyring
if you're still on the same machine as the previous step, or restored
temporarily from `master-secret.asc` — see the note at the bottom):

```bash
gpg --expert --edit-key <keyid>
```
At the `gpg>` prompt:
```
addkey
    → (10) ECC (sign only) → Curve 25519 → 3y
addkey
    → (12) ECC (encrypt only) → Curve 25519 → 3y
save
```

## Write both new subkeys to this card

Still in `gpg --expert --edit-key <keyid>`, or reopen it:
```
gpg> key 1        # select the Sign subkey you just added (check index with 'list')
gpg> keytocard
    → Signature key
gpg> key 1         # deselect
gpg> key 2         # select the Encrypt subkey
gpg> keytocard
    → Encryption key
gpg> key 2
gpg> save
```
`list` inside the same session shows current key indices if you're
unsure which is which — subkeys are listed in the order they were
added, and the ones you just created will be at the bottom.

## Set this card's PIN and Admin PIN

```bash
gpg --card-edit
gpg> admin
gpg> passwd
```
Set both the User PIN and Admin PIN to values distinct from the
defaults (`123456` / `12345678`) and, ideally, distinct per card. Store
these in your password manager alongside the tracking table — not in
plaintext anywhere on disk.

## Repeat for every remaining card

If you're provisioning multiple cards from a single session, the
master key stays loaded in your temporary keyring for the duration —
just swap the physical card and repeat the `addkey` / `keytocard` /
PIN steps. If provisioning happens across separate sessions or
machines, restore the master key into a scratch keyring first:

```bash
export GNUPGHOME=$(mktemp -d)
gpg --import master-secret.asc
# ... run the addkey/keytocard/passwd steps above ...
unset GNUPGHOME
```

## Verify each card independently once done

```bash
gpg --card-status
echo "test" | gpg --clearsign
```
Confirms the Sign subkey and PIN are working correctly for this
specific card before moving to the next one.

## Update your tracking table

Record this card's new subkey fingerprints in the table from
`03-architecture/key-topology.md` before moving on — this is the detail
that's genuinely painful to reconstruct later if skipped now.
