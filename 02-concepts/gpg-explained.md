# GPG, explained from first principles

This section exists so the commands in `04-implementation/` aren't
cargo-culted. Skip it if you already understand GPG's key hierarchy and
smartcard model.

## The key hierarchy: master key vs. subkeys

A GPG identity is not one key. It's a small hierarchy:

- **The master key (also called the primary key).** Its only real job
  in this scheme is *certification* — vouching "these subkeys belong to
  this identity." It is not used for day-to-day signing or encryption.
- **Subkeys**, each with one specific capability: Sign, Encrypt, or
  Authenticate. These are the keys actually used in daily operations.

Why split it this way instead of one key doing everything? Because the
master key is the one piece of key material that, if ever compromised,
lets an attacker impersonate the identity permanently and mint new
"valid" subkeys at will. Keeping it offline and using only subkeys
day-to-day means the thing that's exposed to daily risk (a subkey on a
card that travels with you) is never the thing whose compromise is
catastrophic. If a subkey is ever compromised, it's revoked and
replaced; the master key, and the identity it represents, is untouched.

## Why GnuPG calls the YubiKey a "card"

This trips people up, reasonably: nobody hands you a plastic card. GnuPG
predates USB security keys. It was built against the **OpenPGP card
specification** — an ISO/IEC 7816-4/-8 compatible smart card standard
for storing and using OpenPGP keys in tamper-resistant hardware [7, 8].
YubiKey (and other vendors, e.g. Nitrokey) implement that same protocol
over USB/CCID, so from GnuPG's point of view, a YubiKey *is* a card —
same interface, different physical form factor [8]. Commands like
`gpg --card-status` and `gpg --card-edit` are talking through this
protocol regardless of whether there's literal plastic involved.

## What each subkey capability actually does

- **Sign** — produces a cryptographic signature over data (a git
  commit, a file, an email) that anyone with the public key can verify
  came from this identity and hasn't been altered since.
- **Encrypt** — lets someone encrypt data *to* this identity, such that
  only the holder of the corresponding private key (i.e., whichever
  physical card it lives on) can decrypt it.
- **Authenticate** — proves possession of the private key for login
  purposes, historically used to let an Auth subkey double as an SSH
  key via `gpg-agent`.

This scheme deliberately doesn't use the Authenticate capability — see
`03-architecture/the-split.md` for why SSH is handled by FIDO2 instead.
Every card in this scheme carries Sign and Encrypt subkeys only.

## The property that makes this secure: non-exportability

Once a subkey is moved onto a card (the `keytocard` operation), the
private key material is deleted from the local keyring and exists only
inside the card's secure hardware. The OpenPGP card specification
requires that private keys cannot be read back out by any command or
function [8]. Every cryptographic operation — signing, decrypting —
happens *inside* the card; the private key bytes never cross the USB
connection. This is what makes "steal the laptop" and "steal the key
material" two genuinely different, unrelated events under this scheme.

## PINs, not passwords

A card has two PINs: a **User PIN** (used for routine sign/decrypt
operations, default `123456`) and an **Admin PIN** (used for card
management like setting a new PIN or writing a new key, default
`12345678`) [7, 9]. Both defaults must be changed before real use.
Entering the wrong Admin PIN three times in a row on many cards
**destroys the card's key material permanently** — there is no recovery
from this beyond re-provisioning from the offline master key backup
[7, 9]. This is a deliberate anti-brute-force design, not a bug, but
it's worth knowing before you're guessing at a PIN under pressure.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [7, 8, 9].
