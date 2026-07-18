# Recovery design

A key scheme is only as good as its worst day. This page walks through
every realistic loss scenario and states plainly what it costs. Full
step-by-step recovery commands live in `07-recovery/`; this page is
about the *design* reasoning, so it's readable on its own before Sam
ever needs it under stress.

## Layer 0: the passphrase fallback never goes away

Every FIDO2 enrollment in this scheme — LUKS, `sudo` — is added
*alongside* a traditional passphrase, never as a replacement for one
[1, 2, 5]. This is the single most important design decision in the
whole document, and it's worth stating why plainly: **hardware keys are
a convenience layer on top of a working fallback, not a replacement for
one.** If every physical key Sam owns is unavailable at once — lost,
stolen, left at home, all four simultaneously, however unlikely — Sam is
never locked out of their own machine. The strong passphrase behind that
fallback lives in a password manager (Bitwarden or equivalent), not in
Sam's memory.

## Scenario: one physical key lost or stolen

This is the scenario the entire distinct-per-card topology
(`03-architecture/key-topology.md`) exists to make cheap. What Sam
actually does:

1. Revoke that card's GPG subkey pair (a local operation using the
   offline master key backup — doesn't require the lost card at all).
2. Remove that card's SSH public key from GitHub/GitLab and any
   server's `authorized_keys`.
3. On each machine, remove that card's FIDO2 credential from the LUKS
   header and the `sudo` U2F mapping — an at-leisure task done from
   each machine over the following days, not an emergency.

Nothing about Sam's other 3 keys changes. No other card is re-touched,
re-provisioned, or even needs to be located during this process. If the
lost card was a daily-carry key and the remaining keys are in cold
storage at home, this whole process happens using key material Sam
already has easy access to.

## Scenario: a machine is lost or stolen, powered off

The disk is LUKS-encrypted; without the passphrase or an enrolled FIDO2
credential, the data is inert to whoever has the physical device [4].
Sam should still remove that machine's specific enrollments (its LUKS
FIDO2 credential slots, its entries in any `authorized_keys` it might
have held for inbound access) from other systems it could reach, since
the encrypted disk being safe doesn't mean the machine's *network
access* to other things should be trusted going forward.

## Scenario: all active (carried) keys lost at once, cold storage intact

Functionally identical to the single-key scenario above, just for
however many keys were lost. Cold-storage keys at home are untouched and
remain fully functional for daily use immediately. No urgency beyond
normal single-key cleanup.

## Scenario: total loss — every physical key gone

This is the one scenario that requires the offline master key backup —
see `07-recovery/lost-all-keys.md` for the full runbook once it's
written. In short: restore the master key from its offline backup,
generate a fresh set of subkeys, provision new hardware, revoke every
old subkey and credential, update GitHub/GitLab and every server. This
is a genuinely disruptive event — expect an afternoon, not five minutes
— but it is recoverable without losing continuity of identity (old
signed commits remain verifiable) [17, 18, 19], and it's precisely why
the offline backup exists and needs to be stored somewhere durable and
separate from all 4 daily-use keys.

## What this design does not solve

Recovery design answers "what do I do after something goes wrong." It
does not answer "how do I know something went wrong" — that's a
detection problem (unexpected logins, unrecognized commits, a key
physically missing from where Sam left it), and it's a human vigilance
question more than a technical one. No part of this scheme can notice a
quietly stolen key on its own; it only makes the response cheap once
Sam does notice.

## Sources for claims on this page

Full citations in `appendix/sources.md`. Key ones: LUKS/FIDO2 fallback
behavior [1, 2, 5]; disk-at-rest protection under LUKS2 [4]; subkey
revocation not retroactively invalidating past signatures [17, 18, 19].
