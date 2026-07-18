# iOS

## SSH: works, via a third-party client

iOS has no system-level SSH agent comparable to `ssh-agent` on Linux,
so FIDO2 SSH access on iPhone/iPad goes through a dedicated SSH client
app rather than anything built into the OS. Termius supports FIDO2
hardware keys — including over NFC — across iOS and iPadOS, generating
and connecting with resident credentials directly from the app [14,
15]. Other clients (Blink, Prompt) have historically offered similar
support; check current status before relying on a specific one, since
mobile SSH client capabilities shift faster than desktop tooling [16].

Practically: tap your card to the top of the phone (NFC) when
prompted, same touch/PIN flow as desktop, no separate credential to
manage — it's the same resident credential from
`04-implementation/03-fido2-ssh-keys.md`, not a new mobile-specific key.

**Being upfront about a limitation:** the mobile SSH clients with solid
FIDO2 support are not FOSS. This is a real gap given the values stated
in `01-why/philosophy.md`, and it's not unique to the FIDO2 side of
this scheme — GPG has no better mobile story (below). Worth
periodically checking whether an open-source mobile SSH client has
caught up.

## GPG: rough, largely unsupported

There is no iOS equivalent of `gpg-agent` acting as a system-level
signing or SSH service. OpenPGP-card-over-NFC support on iOS is
essentially absent in mainstream tooling. Practical implication: don't
plan on signing commits or decrypting files from an iPhone under this
scheme. This is one of the concrete reasons SSH was moved to FIDO2
rather than staying on GPG (`03-architecture/the-split.md`) — it's also
simply true that GPG's mobile story remains behind FIDO2's regardless
of which protocol handles SSH.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [14, 15, 16].
