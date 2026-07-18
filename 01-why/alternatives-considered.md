# Alternatives considered

Before settling on the design in `03-architecture/`, it's worth being
honest about what else was on the table, and why it didn't win. If a
friend reads this and thinks "why not just do X," the answer is
probably here.

## Option 1: Passwords/passphrases only, no hardware keys

The baseline. A strong LUKS passphrase, a strong `sudo` password, SSH
keys protected by a passphrase, GPG keys protected by a passphrase.

**Why it's not enough on its own:** every one of those passphrases has
to be typed, repeatedly, by a human, under time pressure, often
half-awake. Passphrase strength and daily usability trade off directly
against each other, and in practice usability wins — passphrases get
shortened over time. This option remains in the final design, but only
as a fallback, never as the primary daily mechanism. See
`03-architecture/recovery-design.md`.

## Option 2: A single SSH/GPG key, used everywhere, no hardware backing

The common failure mode this document exists to fix. One keypair,
copied by hand to every new machine, years of `id_rsa` or `id_ed25519`
files quietly accumulating in `~/.ssh` and `~/.gnupg` across every
device Sam has ever owned.

**Why it's rejected:** the private key exists as a plain file on disk.
Anything that can read Sam's home directory — malware, a bad backup, an
old drive that gets recycled without being wiped — can exfiltrate it
completely, silently, with no way for Sam to ever know it happened. This
is the exact failure mode hardware-backed keys are designed to make
structurally impossible: the private key material never exists outside
the hardware device in the first place [6, 8].

## Option 3: Cloud-synced passkeys (Apple/Google/Microsoft ecosystem)

Passkeys synced through iCloud Keychain or Google Password Manager are
genuinely excellent for *web login*, and if Sam's needs stopped at "log
into websites," this would be a reasonable, low-effort choice.

**Why it's rejected for this use case:** it doesn't cover SSH to a
Linux box, doesn't cover LUKS disk unlock, and doesn't cover GPG commit
signing or file encryption — the three things Sam actually needs solved.
It also means Sam's entire credential store lives inside one company's
cloud account, which runs directly against the FOSS/portability values
stated in the threat model. Worth revisiting for pure web-account 2FA
alongside this scheme, but it's not a substitute for it.

## Option 4: SSH certificates from an internal Certificate Authority

This is genuinely what most serious engineering organizations do at
scale — a service like Teleport, HashiCorp Vault, or `step-ca` issues
short-lived SSH certificates on demand, so there's no long-lived key to
lose or leak in the first place, and access can be revoked centrally in
seconds.

**Why it's rejected here, specifically:** this is solving a
fleet-management problem — hundreds of engineers, hundreds of servers,
a dedicated team operating the CA. Standing up and maintaining that
infrastructure for one person's five personal machines is pure
overhead with no corresponding benefit. If Sam's home server setup grows
into something with many users and many machines, this is worth
revisiting. For now it's noted so nobody reading this feels like they're
missing "the real industry practice" — they're not; they're correctly
right-sizing it.

## Option 5: One identical GPG identity cloned onto every hardware key

Generate one master key and one set of Sign/Encrypt/Auth subkeys, then
write byte-identical copies onto all 4 YubiKeys. Any key is
indistinguishable from any other, and there's exactly one entry on
GitHub for everything, always.

**Why it's rejected:** cloned key material means cloned trust. If one
card is lost or stolen, the private key an attacker now holds is
*identical* to the key on Sam's remaining 3 cards — cryptographically
indistinguishable. The only correct response is to revoke that subkey
entirely and re-provision every other card, even though they were never
at risk. That's a bigger, more disruptive cleanup than losing a
uniquely-keyed card, and it also means Sam can never tell, after the
fact, which physical card produced a given signature or login — useful
information when trying to figure out what happened after a compromise.
The distinct-subkey design in this document was checked against GitHub's
actual behavior and does not cost anything in exchange for solving this
— see `03-architecture/key-topology.md`.

## Option 6: FIDO2 for everything, including replacing GPG entirely

Use FIDO2 resident keys (`ed25519-sk`) for SSH, `sudo`, and LUKS, and
drop GPG altogether — including for commit signing and file/email
encryption.

**Why it's rejected:** FIDO2/WebAuthn is built for *authentication* —
proving "this request came from a device I trust" — and it does that
job better than GPG does. It was never designed for *asymmetric
encryption of arbitrary data* (encrypting a file for someone, an
encrypted email, a password-manager vault like `pass`), and it doesn't
do that job at all. If Sam wants any of that later — which the roadmap
in `08-roadmap/secrets-management.md` assumes — GPG (or one of its
lighter modern alternatives) is still necessary. Dropping GPG entirely
solves a problem Sam doesn't have (GPG is supposedly "too complex") by
creating one Sam will have soon (no encryption story at all).

## What was actually chosen, briefly

A split: FIDO2 owns authentication (LUKS, `sudo`, SSH), GPG owns identity
and encryption (commit signing, file/email encryption), and within GPG,
every physical key gets distinct, non-cloned subkeys. Full reasoning in
`03-architecture/the-split.md`.
