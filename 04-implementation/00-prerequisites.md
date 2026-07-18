# Prerequisites

Run this section once per machine, before touching any card.

## Package installation by distro family

The same underlying tools exist on every distro; only names and package
managers differ.

**Arch (`pacman`):**
```bash
sudo pacman -S --needed \
  gnupg pcsclite ccid \
  yubikey-manager \
  pam-u2f libfido2 \
  opensc
sudo systemctl enable --now pcscd.service
```

**Fedora / RHEL family (`dnf`):**
```bash
sudo dnf install -y \
  gnupg2 pcsc-lite ccid \
  yubikey-manager \
  pam-u2f libfido2 \
  opensc
sudo systemctl enable --now pcscd.service
```

**Debian / Ubuntu family (`apt`):**
```bash
sudo apt update
sudo apt install -y \
  gnupg2 pcscd scdaemon \
  yubikey-manager \
  libpam-u2f libfido2-1 libu2f-udev \
  opensc
sudo systemctl enable --now pcscd.service
```

Package names confirmed current as of this writing against each
distro's own packaging pages and Yubico's own installation
documentation [21, 22, 23, 24, 25]. If a package name has changed by
the time you're reading this, your distro's package search is the
fastest way to confirm the current name — the tools themselves
(`gnupg`, `pcscd`/`pcsc-lite`, `pam-u2f`, `libfido2`, `yubikey-manager`)
are what matters, not the exact string.

## Verify firmware supports what this scheme needs

Plug in each key and check:
```bash
ykman info
```
Confirm firmware version **5.2.3 or newer** — this is the minimum for
ed25519 support on the OpenPGP applet, which this scheme uses for both
GPG subkeys and FIDO2 resident SSH keys [21, 86]. Older keys can still
participate using RSA instead of ed25519, but the commands in this
document assume ed25519 throughout; adjust key-type flags accordingly
if using older hardware.

## Confirm the card is visible to GnuPG

```bash
gpg --card-status
```
This should print card details (serial number, available key slots).
If it hangs or errors, `pcscd` isn't running or isn't seeing the device
— re-check the service is enabled and the key is properly seated.

## A note on udev rules

Distro packages sometimes install YubiKey udev rules automatically,
sometimes not. If commands above require `sudo` to see the key at all
(they shouldn't), missing udev rules are the likely cause — Yubico
publishes an official ruleset if your distro's package didn't include
it. Symptom to watch for: everything works as root but fails silently
as a normal user.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [21, 22, 23, 24, 25].
