# The core design: two protocols, split by job

## The shape of it

Every YubiKey used in this scheme runs two independent applications on
the same physical device — they don't conflict, and using one doesn't
disable the other [10]:

```
                    ONE PHYSICAL YUBIKEY
        ------------------------------------------
        |                        |                |
   FIDO2 / U2F applet                 OpenPGP applet
   ("access")                         ("identity")
        |                                    |
        |-- LUKS disk unlock                 |-- Git commit signing
        |-- sudo (PAM)                       |-- File / email encryption
        |-- SSH login                        |-- Long-term identity
```

*(Note: this repo will get a proper diagram image in a later revision —
see the project README. For now, this text version is accurate and
sufficient to work from.)*

FIDO2 handles anything that's fundamentally a yes/no access decision:
*is the person holding this device allowed to unlock this disk, run this
command, open this SSH session?* GPG handles anything that's
fundamentally about proving identity or protecting content over time:
*did this specific person write this commit, can only this specific
person read this file?*

## Why not put everything on one protocol

This was the actual open question worked through while building this
scheme — see `01-why/alternatives-considered.md` for the two rejected
single-protocol options. The short version:

**FIDO2 can't do what GPG does.** It has no encryption capability at
all — it's purely an authentication protocol. It cannot encrypt a file,
protect a password vault, or send someone an encrypted message. If
that's ever needed (and per Sam's stated goals around self-hosting and
secrets management, it will be), GPG or an equivalent has to exist
somewhere in the stack regardless.

**GPG is worse than FIDO2 at what FIDO2 is for.** Routing SSH
authentication through `gpg-agent` works, but it's a second system
bolted onto a job it wasn't originally built for — GPG's smartcard
support predates modern hardware-key SSH support by well over a decade,
and it shows in the tooling. OpenSSH gained native, first-class hardware
security key support in version 8.2 (February 2020), with key types
`ecdsa-sk` and `ed25519-sk` built directly into `ssh-keygen` and
`ssh-agent` — no `gpg-agent`, no `SSH_AUTH_SOCK` redirection, no
smartcard daemon translation layer [6]. It is simpler to operate and,
critically for Sam's stated direction, it has dramatically better
mobile support: modern SSH clients on iOS and Android (Termius, for
one) support FIDO2 hardware keys over NFC directly, across desktop,
iOS, and Android uniformly [14, 15]. There is no equivalent story for
carrying a GPG smartcard identity onto a phone — OpenPGP-card-over-NFC
on mobile exists but is niche and rough, especially on iOS, where
there's no real system-level equivalent to `gpg-agent` acting as an SSH
agent.

**Neither is a compromise, once split this way.** LUKS and `sudo` were
always going to be FIDO2 regardless (see `04-implementation/`) — GPG
smartcards were never a serious option for disk unlock. So the real
decision was narrower than "GPG vs FIDO2" made it sound: it was "should
SSH join LUKS and sudo under FIDO2, or stay with GPG for the sake of
using one fewer protocol." Given the mobile and tooling advantages
above, and that dropping GPG's Auth capability costs nothing (GPG's
Sign and Encrypt capabilities are unaffected either way), SSH moved to
FIDO2 too.

## What this means day to day

In practice, Sam will interact with the FIDO2 side of this constantly —
several times an hour, every `sudo`, every SSH session, every disk
unlock. The GPG side gets touched far less often — once per `git commit`,
occasionally for encrypting a file. This asymmetry lines up naturally
with the underlying protocol strengths: FIDO2 is deliberately fast and
low-friction by design (that's the entire point of a passwordless
authentication standard), while GPG operations are allowed to be a
little more deliberate, since they're rarer and often carry more
permanent consequences (a commit signature is part of the historical
record forever; an SSH login is not).

## Sources for claims on this page

Full citations in `appendix/sources.md`. Key ones: OpenPGP applet and
FIDO2 applet independence on YubiKey hardware [10]; OpenSSH 8.2 native
FIDO/U2F support [6]; Termius cross-platform FIDO2 SSH support including
NFC on mobile [14, 15].
