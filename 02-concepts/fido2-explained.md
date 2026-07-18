# FIDO2, explained from first principles

## What FIDO2 actually is

FIDO2 is an open authentication standard developed jointly by the FIDO
Alliance and W3C (as WebAuthn), designed to replace passwords with
public-key cryptography that's resistant to phishing by construction —
a credential is bound to the specific origin (website, or in this
scheme's case, the specific SSH server) it was created for, so it
simply doesn't function if presented to an impostor [6, 16]. It's the
successor to the older U2F standard, and most modern hardware keys
(YubiKey, Nitrokey, Google Titan, SoloKeys) speak both [16].

Unlike GPG, FIDO2 was never designed to encrypt or sign arbitrary data
for later verification by a third party. It answers exactly one
question: *is the entity presenting this credential right now the same
one that registered it?* That narrower scope is exactly what makes it
fast and simple where GPG is comparatively heavyweight — see
`03-architecture/the-split.md`.

## Resident vs. non-resident credentials

This is the one FIDO2 concept worth understanding before provisioning
anything, because it determines what "bringing a key to a new machine"
actually requires.

- **Non-resident credential:** the key material is derived from a seed
  on the hardware token plus data stored locally on the host machine (a
  "key handle" file). Losing that local file means the credential is
  unusable even with the physical key present, unless the relying party
  can re-derive it.
- **Resident credential** (also called a "discoverable credential"):
  the credential is stored entirely on the hardware token itself. A new
  machine can query the token directly and pull the credential back
  down — no local file needs to have survived anywhere.

This scheme uses **resident** credentials throughout
(`ssh-keygen -O resident`), specifically because it's the property that
makes the "new machine, instant bring-up" workflow in `05-bootstrap/`
possible without any file transfer.

## Why FIDO2 credentials can't be cloned

This is the core structural difference from GPG-on-card, and it's the
reason `03-architecture/key-topology.md` treats "one shared identity
across all cards" as achievable for GPG but not for FIDO2 SSH auth.
Each physical authenticator generates its own credential locally when
it's created, using a private seed that's baked into that specific
piece of hardware and never leaves it. There's no export/import
operation, no `keytocard` equivalent — you cannot write the same FIDO2
credential onto a second physical key, by design. Every card
provisioned in this scheme therefore ends up with a genuinely distinct
SSH public key, not a copy of one shared key.

## User presence, user verification, and PIN

FIDO2 supports a few different confirmation mechanisms, and this scheme
deliberately uses more than the bare minimum:

- **User presence** — a physical touch, confirming a human (not
  malware) triggered the operation. This is the default and is always
  required.
- **User verification** — an additional PIN or biometric check,
  confirming *which* human. This scheme enables this
  (`-O verify-required` when generating SSH keys), so a stolen key alone
  isn't sufficient without also knowing its PIN.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [6, 16].
