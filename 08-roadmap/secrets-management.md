# Roadmap: secrets management

Deliberately out of scope for this document's main body — noted here so
it's not forgotten, not because it isn't a real near-term need.

## The problem this will eventually solve

API keys, tokens for self-hosted services, and similar secrets need
somewhere to live that isn't a plaintext file in a dotfiles repo or a
`.env` file that's one `git add .` away from being committed by
accident.

## Why this connects to work already done

The GPG Encrypt subkeys provisioned per card in
`04-implementation/02-gpg-subkeys-per-card.md` already provide exactly
the asymmetric encryption primitive this needs — nothing new has to be
generated. The standard FOSS tool built directly on this is `pass`
(and its more feature-rich fork `gopass`): a plaintext-only-when-decrypted
password store where each secret is a GPG-encrypted file, versioned in
git, decryptable by any card holding the right Encrypt subkey.

## Alternatives worth evaluating before committing

- **`sops`** — encrypts structured config files (YAML/JSON/ENV)
  in place, can use GPG or cloud KMS backends, popular in
  infrastructure-as-code contexts. Worth a look given the
  self-hosting/homelab direction in `01-why/threat-model.md`.
- **`age`** (and `age-plugin-yubikey`) — a deliberately simpler,
  modern alternative to GPG for pure file encryption, using the PIV
  applet rather than OpenPGP. Genuinely worth learning, but the right
  next step is reading up on it directly rather than this document
  making the call prematurely — noted here per an explicit decision not
  to fold it into the main scheme yet, pending more hands-on
  familiarity with both GPG and age individually first.

## What "done" looks like for this section, later

A follow-up to this repo (or a new top-level section here) covering:
initializing a `pass` store against the existing per-card Encrypt
subkeys, a sane directory structure for API keys vs. service
credentials vs. personal secrets, and how this interacts with the
existing recovery design in `03-architecture/recovery-design.md` when a
card holding decrypt access is lost.
