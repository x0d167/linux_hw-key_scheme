# Android

## SSH: works, similarly to iOS

Same shape as iOS — Android has no system SSH agent, so a dedicated
client app is required. Termius supports FIDO2 hardware keys over NFC
on Android as well as iOS and desktop, using the same underlying
resident credentials [14, 15]. Android's more open app ecosystem means
there's slightly more room for alternative or future FOSS clients to
catch up here than on iOS, but as of this writing the same caveat
applies: the mature options aren't open source.

## GPG: better than iOS, still not great

Android has a more established (if still niche) OpenPGP-smartcard
story via apps like OpenKeychain, which can talk to a YubiKey's OpenPGP
applet over NFC/USB for signing and decryption within supporting apps.
This is meaningfully more than iOS offers, but it's app-scoped rather
than system-wide — there's no equivalent of transparently signing a
`git commit` run from a generic terminal app the way `gpg-agent` does
on desktop Linux. Treat Android as "possible for specific supported
apps," not as a full extension of the desktop GPG workflow.

## Practical takeaway for both platforms

Mobile, under this scheme, is realistically an SSH access surface
(reading logs, running commands on the home server, checking on a VM)
rather than a full identity/signing surface. That's a reasonable
division given what phones are actually used for day to day — see the
vignettes in `09-vignettes/` for what this looks like in practice.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [14, 15].
