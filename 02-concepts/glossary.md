# Glossary

Terms used throughout this repo, defined once here rather than
re-explained in every file.

**Applet** — one of the independent "programs" running on a hardware
security key's chip. A single YubiKey runs a FIDO2/U2F applet and an
OpenPGP applet simultaneously and independently [10].

**Card / smartcard** — GnuPG's term for anything speaking the OpenPGP
card protocol, whether it's literal plastic or a USB device like a
YubiKey emulating the same interface. See `02-concepts/gpg-explained.md`.

**`ccid`** — the Linux driver package for USB smartcard readers,
including the interface YubiKeys use for their OpenPGP applet.

**`crypttab`** — `/etc/crypttab`, the system file listing which
encrypted block devices should be unlocked at boot and how [3].

**dm-crypt** — the Linux kernel's device-mapper target that provides
transparent disk encryption; LUKS is a specific on-disk format built on
top of it.

**FIDO2 / WebAuthn / U2F** — related open authentication standards.
U2F is the older, narrower predecessor; FIDO2 (which WebAuthn is part
of) is the current, broader standard. Most modern hardware keys speak
both [16]. See `02-concepts/fido2-explained.md`.

**`gpg-agent`** — the background process that holds GPG keys (or
communicates with a card holding them) and services requests from
`gpg`, and optionally from SSH if configured for it.

**`hmac-secret` extension** — the specific FIDO2 capability
`systemd-cryptenroll` relies on to use a hardware key for LUKS
unlocking [1].

**`initramfs` / `initrd`** — the minimal filesystem loaded into memory
early in boot, before the real root filesystem is available. Unlocking
an encrypted root partition has to happen here, which is why FIDO2/LUKS
setup sometimes requires initramfs regeneration.

**`keygrip`** — GnuPG's internal identifier for a specific key, distinct
from its more commonly seen key ID/fingerprint. Used when telling
`gpg-agent` which key to expose over SSH.

**LUKS / LUKS2** — Linux Unified Key Setup, the standard on-disk format
for encrypted volumes on Linux. LUKS2 is the current version and adds
support for enrolling multiple unlock methods (passphrase, FIDO2, TPM)
into separate key slots on the same volume [1, 2].

**PAM** — Pluggable Authentication Modules, the Linux framework that
lets things like `sudo` and login chain together multiple authentication
methods (password, hardware key, etc.) [5].

**`pam_u2f` / `pam-u2f`** — the specific PAM module that adds FIDO2/U2F
hardware key support to any PAM-aware service, including `sudo` [5].

**`pcscd`** — the PC/SC daemon, the system service that lets
applications (including `gpg-agent`) talk to smartcard readers,
including a YubiKey's OpenPGP applet over USB.

**Relying party** — in FIDO2 terminology, the service a credential was
registered with (a specific SSH server, a specific website). Credentials
are cryptographically bound to their relying party, which is the source
of FIDO2's phishing resistance.

**Resident credential** — a FIDO2 credential stored entirely on the
hardware token, retrievable from the token alone with no dependency on
local files surviving on any particular machine. See
`02-concepts/fido2-explained.md`.

**`scdaemon`** — GnuPG's smartcard daemon, the component that actually
speaks the OpenPGP card protocol to a connected card via `pcscd`.

**`ykman`** — Yubico's official command-line management tool, used
throughout this scheme for firmware checks, touch-policy configuration,
and general key administration.
