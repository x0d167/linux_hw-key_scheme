# Generating the offline master key

Do this once, ever (barring total disaster recovery). Do it on a
machine you're comfortable calling clean for this session — a live USB
environment is ideal; disconnecting network on your normal machine for
the duration is a reasonable middle ground. The goal is simply that
this key material never has a moment where it's both generated and
network-connected.

## Generate the master key

```bash
gpg --expert --full-generate-key
```
Choices when prompted:
- Key type: `(9) ECC and ECC`, then curve `(1) Curve 25519`
- **Immediately toggle capabilities down to Certify only.** This is the
  step people skip and shouldn't — the master key's only job is
  certifying subkeys, per `02-concepts/gpg-explained.md`.
- Expiry: `3y` (see `03-architecture/key-topology.md` for reasoning)
- Real name / email as prompted

Note the key ID it prints. You'll need it repeatedly:
```bash
gpg --list-secret-keys --keyid-format=long
```

## Generate a revocation certificate immediately

Before anything else touches this key:
```bash
gpg --gen-revoke <keyid> > revoke.asc
```
This lets you invalidate the entire identity later even if you lose
access to the private key material itself — keep it with the backup
below, not on any daily-use machine.

## Back up the master key material

```bash
gpg --export-secret-keys --armor <keyid> > master-secret.asc
gpg --export --armor <keyid> > pubkey.asc
```

Put `revoke.asc` and `master-secret.asc` somewhere offline and durable
— an encrypted USB stick (a small LUKS container works fine, or a
password-protected archive) kept physically separate from all 4 daily
YubiKeys. This is the one piece of this entire scheme that, if lost
alongside every physical card, means starting over from zero. Treat it
with the seriousness of a legal document, not a config file.

`pubkey.asc` is not sensitive — it's the public half — but keep a copy
somewhere convenient, since it's needed once per new card during
provisioning (`04-implementation/02-gpg-subkeys-per-card.md`) and again
when uploading to GitHub/GitLab (`04-implementation/06-git-hosting-config.md`).

## What happens next

The master key's job is now essentially done until the next expiry
renewal (roughly every 3 years) or a full disaster recovery. Every
remaining GPG step happens on the *subkeys*, generated per card in the
next section.
