# Key topology: how many keys, and what's on each

## The starting instinct, and why it's the wrong metric

The instinct most people start with is "I want exactly one entry per
service" — one SSH key, one GPG key, full stop. It's a reasonable
starting instinct, born from a real annoyance (six SSH keys in a
GitHub account, and no memory of which is which). But it's the wrong
thing to optimize for once you look at what it actually costs.

The right question isn't "how many entries do I have." It's: **when a
single key is lost, what does recovery cost, and what does it expose in
the meantime?** Optimizing purely for entry-count pushes toward cloning
identical key material across every device — and cloned key material
means that losing any one card compromises all of them equally, forcing
a full re-key of every remaining device even though only one was ever at
risk (walked through in detail in
`01-why/alternatives-considered.md`, Option 5).

## The chosen topology

**Every physical key is cryptographically distinct.** No key material
is ever copied from one physical device to another. Concretely, for
each of Sam's 4 YubiKeys:

- **GPG side:** its own Sign subkey and its own Encrypt subkey, both
  certified by one shared offline master key.
- **FIDO2 side:** its own resident SSH credential (`ed25519-sk`), its
  own FIDO2 credential enrolled into each machine's LUKS header, its own
  U2F mapping for `sudo`.

Nothing here depends on which 2 keys are "active" and which 2 are
"cold" at any given moment, or whether that split becomes 3-and-1 later.
Every key is provisioned identically and independently; "active" vs
"cold storage" is just a statement about where Sam happens to be
keeping a given key this month, not a property built into the key
itself.

## Why this doesn't cost what it looks like it costs

**GPG: one master key, many subkey pairs, still one entry.** A GPG
identity on GitHub or GitLab is keyed off the *primary key's
fingerprint*, not off each individual subkey [11, 12]. That means Sam
can generate 4 separate Sign+Encrypt subkey pairs — one pair per
physical card, fully distinct key material — all certified under a
single master key, export that master key's public component once, and
upload it once. GitHub shows one entry. Commits signed by any of the 4
cards' Sign subkeys verify correctly, because they're all part of the
same exported public key block [11]. Revoking one card's subkey doesn't
touch the others, and doesn't invalidate that card's past signatures
retroactively [17, 18, 19]. This is, in effect, a free upgrade: full
per-card isolation, zero added entries, zero added cost.

**FIDO2/SSH: genuinely does cost extra entries, and that's fine.**
Unlike GPG, GitHub and GitLab match SSH keys by the literal public key
string, with no concept of a shared parent identity tying multiple keys
together [13]. So 4 distinct FIDO2 SSH credentials really do mean 4
entries in GitHub's SSH key list, not 1. This was worked through
directly against Sam's actual stated preference: given that entry count
"doesn't genuinely bother" Sam once separated from the isolation
question, and given that isolated revocation was the property Sam
actually cared about, distinct FIDO2 credentials per card is the
correct trade here. Someone reading this who *does* care more about
entry count than isolation can reasonably choose to clone one Auth
identity across cards instead — that's a legitimate, smaller-scale
version of Option 5 from the alternatives page, just scoped to SSH only
rather than the whole GPG identity. It is a real trade-off either way,
just not the one made in this document.

## Naming and tracking convention

With 4 cards each holding distinct key material, keeping track of which
physical key maps to which fingerprint/credential stops being optional.
Before provisioning, decide on a labeling scheme and write it down
somewhere durable (a password manager note, not just memory):

| Physical key | Role (subject to change) | GPG subkey fingerprints | FIDO2 SSH pubkey (last 8 chars) |
|---|---|---|---|
| YubiKey #1 | Daily carry | *(fill in after provisioning)* | |
| YubiKey #2 | Daily carry / travel spare | | |
| YubiKey #3 | Cold storage, home | | |
| YubiKey #4 | Cold storage, off-site | | |

`ykman info` printed on each key (which includes its unique serial
number) is the fastest way to confirm which physical key you're holding
mid-provisioning — worth running before every `keytocard` step to avoid
writing a subkey to the wrong card.

## Expiry policy

All subkeys: 3-year expiry, renewed from the offline master key backup
without needing to touch any physical card (renewal only changes the
expiry timestamp on the existing certificate; it doesn't regenerate or
move key material). Why 3 years and not "no expiry": an expiry date is
mainly useful as a forced, periodic check that the offline master key
backup still restores correctly — bit rot and "actually I have no idea
if that USB stick still works" are real, mundane risks that a permanent,
never-checked key doesn't defend against. It is not primarily a defense
against key compromise, since compromising hardware-backed key material
requires physical possession plus a PIN in the first place.

## Sources for claims on this page

Full citations in `appendix/sources.md`. Key ones: GitHub's GPG-key
matching behavior via primary key fingerprint, including multi-subkey
identities [11, 12]; per-subkey revocation not affecting sibling subkeys
or past signatures [17, 18, 19]; GitHub/GitLab's SSH key matching being
per-literal-key rather than identity-based [13].
