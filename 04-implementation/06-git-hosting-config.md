# GitHub / GitLab configuration

This is where the whole scheme's payoff becomes visible: one GPG entry,
a handful of SSH entries, and both correctly wired to `git`.

## Upload the GPG public key — once

```bash
gpg --armor --export <keyid> > pubkey.asc
```
Paste the contents into Settings → SSH and GPG keys → New GPG key.
Because this exports the **master key's** public component, and every
card's Sign subkey was certified under that same master key, this
single upload covers commits signed by *any* of your 4 cards — GitHub
matches by primary key fingerprint, not per-subkey [11, 12]. Re-export
and re-upload (replacing the existing entry, not adding a new one) any
time a new card's subkey is added later.

## Upload each SSH public key — once per card

Per `03-architecture/key-topology.md`, this part genuinely does need
one entry per card, because GitHub matches SSH keys by literal public
key string with no shared-identity concept [13]. Add each
`~/.ssh/id_yk_keyN.pub` from `04-implementation/03-fido2-ssh-keys.md`
under Settings → SSH and GPG keys → New SSH key, with a title that
matches your tracking table so future-you can tell them apart.

## Tell git which key to sign with

```bash
git config --global user.signingkey <keyid>!
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```
The trailing `!` pins git to this exact key ID rather than letting it
pick automatically among multiple candidates — recommended once you
have more than one GPG key or subkey in your keyring at all [9, 11].

Because signing happens through whichever card is physically present,
this config line doesn't need to change based on which of your 4 cards
you're carrying that day — `gpg-agent` and `scdaemon` handle finding
and using whichever card is plugged in.

## Verify the whole chain works

```bash
git commit --allow-empty -m "test signing"
git log --show-signature -1
```
Should show a good signature. Push it and confirm GitHub/GitLab shows
the commit as "Verified."

## Revoking one card's signing ability later

If a card is lost (`03-architecture/recovery-design.md`), revoke just
that card's Sign subkey locally using the offline master key, then
re-export and re-upload the updated public key — same single-entry
process as above, just with that subkey now marked revoked. Past
commits signed by it remain verified; only new signatures under that
subkey stop validating [17, 18, 19].

## Sources for claims on this page

Full citations in `appendix/sources.md`: [9, 11, 12, 13, 17, 18, 19].
