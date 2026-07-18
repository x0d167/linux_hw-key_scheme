# Recovery runbook: total loss

Every physical card gone at once. Rare, genuinely disruptive, and the
one scenario this entire scheme's offline master key backup exists for.
Expect an afternoon of work, not five minutes — but full recovery
without losing identity continuity.

## Before anything else: you are not locked out

Per `03-architecture/recovery-design.md`, Layer 0 — the LUKS passphrase
and `sudo` password fallback — never depended on any physical key. You
can still log into and unlock every one of your machines right now,
using the passphrases stored in your password manager. This runbook is
about restoring the *convenience layer*, not regaining access you never
actually lost.

## 1. Retrieve the offline master key backup

`master-secret.asc` and `revoke.asc` from
`04-implementation/01-master-key.md`. This is the whole reason that
backup lives somewhere separate from all 4 daily cards.

## 2. Acquire replacement hardware

New YubiKeys (or equivalent FIDO2/OpenPGP-card-compatible devices).

## 3. Restore the master key and generate fresh subkeys

```bash
export GNUPGHOME=$(mktemp -d)
gpg --import master-secret.asc
```
Then repeat `04-implementation/02-gpg-subkeys-per-card.md` in full for
every new card — fresh Sign+Encrypt pairs, not a restore of the old
ones. The old subkeys are presumed compromised (someone may physically
have the old cards) and should be revoked, not reused.

## 4. Revoke every old subkey

```bash
gpg --edit-key <keyid>
gpg> key 1
gpg> revkey
# repeat for every old subkey across all previously-lost cards
gpg> save
gpg --armor --export <keyid> > pubkey-recovered.asc
```

## 5. Re-provision FIDO2 SSH credentials

Fresh resident credentials per new card
(`04-implementation/03-fido2-ssh-keys.md`) — there's no "restore" for
FIDO2 credentials, since they were never exportable in the first place
(`02-concepts/fido2-explained.md`). This is a clean generate, not a
recovery.

## 6. Update every external system

- GitHub/GitLab GPG entry: replace with `pubkey-recovered.asc`.
- GitHub/GitLab SSH entries: remove all old ones, add new ones.
- Every server's `authorized_keys`: same.
- Every machine's LUKS enrollments and `sudo` u2f_keys: wipe old slots,
  enroll new cards.

## 7. What you don't lose

Old commits signed by the now-revoked subkeys remain verified — GPG
revocation doesn't retroactively invalidate history, only future
signatures under the revoked key [17, 18, 19]. Your identity, in the
sense of "the continuous signed history of your work," survives this
event intact. What's actually being replaced is the hardware and the
subkeys, not the identity itself.
