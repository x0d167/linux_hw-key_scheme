# FAQ

## Isn't this overkill for a personal setup?

The individual pieces (LUKS, GPG, FIDO2, PAM) are all standard, widely
used tools — nothing here is exotic. What might look like "a lot" is
mostly documentation and reasoning, not operational complexity; the
actual daily experience is a touch or a PIN, described in
`09-vignettes/`. See `01-why/threat-model.md` for what this is and
isn't defending against — it's deliberately scoped to not be overkill.

## Can I do this with just one YubiKey?

Yes. Everything in `04-implementation/` works identically with one
card; you simply don't get the loss-recovery benefits described in
`03-architecture/recovery-design.md`, since there's no second card to
fall back on. The passphrase/password fallback (Layer 0) still applies
regardless of how many cards you own. I recommend at least two.

## What if I only care about SSH, not GPG at all?

Skip `04-implementation/01`, `02`, and `06` entirely — FIDO2 SSH
(`04-implementation/03`), LUKS (`04-implementation/04`), and `sudo`
(`04-implementation/05`) don't depend on any GPG setup existing. You'll
lose commit signing and the future secrets-management path in
`08-roadmap/`, which is a reasonable trade if that's genuinely not a
need.

## Why not just use a password manager for everything?

A password manager (Bitwarden or similar) is still very much part of
this scheme — it's where the LUKS/`sudo` fallback passphrases live, per
`03-architecture/recovery-design.md`. It's a complement to hardware
keys, not an alternative to them: a password manager protects secrets
behind a master password, which is still a single point of failure if
that master password (or the account itself) is compromised. Hardware
keys move the "what could go wrong" surface to something that requires
physical possession, not just knowledge of a secret.

## What happens if GitHub changes or removes the `.keys`/`.gpg` endpoints?

`05-bootstrap/the-instant-clone-trick.md` becomes less convenient, not
broken — you'd fall back to fetching public keys from wherever you also
publish them (a personal website, a keyserver for GPG). Nothing about
the core scheme depends on GitHub specifically; it's a convenience
layer on top of key material that's yours regardless of where it's
hosted.

## Do I need to redo everything when a subkey expires?

No — see `03-architecture/key-topology.md`'s expiry section. Renewal is
an offline-master-key operation that extends the existing subkey's
expiry date; it doesn't regenerate or move key material, so no physical
card needs to be touched.

## Does this work the same way if I switch my main OS to NixOS?

The underlying scheme — GPG-on-card, FIDO2, LUKS, PAM — is identical
regardless of distro, per this document's scope note in the README. The
_declarative configuration_ of it changes meaningfully on NixOS,
which is why that's planned as a separate follow-up document rather
than folded in here.
