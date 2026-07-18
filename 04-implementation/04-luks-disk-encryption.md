# LUKS + FIDO2 disk encryption

Per-machine, not per-card — run this once for every laptop, enrolling
however many of your 4 keys you want that specific machine to accept.
Requires `systemd` version 248 or newer, which is standard on any
current Arch, Fedora, or Debian/Ubuntu install [1].

## Enroll a FIDO2 key into an existing LUKS2 volume

```bash
sudo systemd-cryptenroll --fido2-device=auto /dev/nvme0n1p2
```
Replace the device path with your actual encrypted partition (check
with `lsblk` or `blkid` if unsure). You'll be prompted for your
**existing LUKS passphrase** first (to authorize adding a new unlock
method), then to touch the key [1, 2].

**Do this once per card you want this specific machine to accept.**
Swap keys between runs. There's no need for every machine to accept
all 4 cards — a machine that only ever leaves the house with you might
reasonably only enroll your 2 daily-carry keys, while your home server
might enroll all 4.

## Critical: never remove the passphrase slot

LUKS2 supports up to 32 key slots on one volume [1]. Enrolling FIDO2
keys adds slots; it does not remove the original passphrase slot unless
you explicitly wipe it. **Don't wipe it.** This passphrase, stored in
your password manager, is Layer 0 of the recovery design in
`03-architecture/recovery-design.md` — the thing that means losing every
physical key you own still doesn't lock you out of your own machine.

## Making this apply to the root partition

If you're encrypting the *root* filesystem (as opposed to a secondary
data volume), the FIDO2 unlock needs to happen in early boot, before the
real root filesystem is even mounted — which means it has to work from
inside the `initramfs`. The exact configuration differs by how your
distro builds its initramfs:

**Arch (mkinitcpio):** ensure the `systemd` and `sd-encrypt` hooks are
present (not the older `encrypt` hook) in `/etc/mkinitcpio.conf`, then
regenerate: `sudo mkinitcpio -P`. See ArchWiki's dm-crypt system
configuration page for the current hook ordering requirements — this is
one of the more fiddly steps and worth reading directly rather than
copying blind [4].

**Fedora (dracut):** `systemd-cryptenroll` support is generally present
out of the box on current Fedora since it ships a `systemd`-based
initramfs by default; confirm with `lsinitrd | grep fido2` after a
`dracut -f` regeneration if the key isn't being offered at boot.

**Debian/Ubuntu (initramfs-tools or dracut, depending on setup):**
behavior varies more here than on Arch or Fedora since Debian-family
distros don't always default to a `systemd`-based initramfs. Confirm
which initramfs generator is in use (`dpkg -l | grep -E 'initramfs-tools|dracut'`)
before assuming the steps above apply as-is.

## Add the persistent crypttab entry

For non-root volumes (or root volumes on distros using `/etc/crypttab`
for early-boot unlocking via `sd-encrypt`), add `fido2-device=auto` to
the relevant line in `/etc/crypttab` [3]:
```
myvolume UUID=xxxx-xxxx none fido2-device=auto
```
Test before rebooting on it:
```bash
sudo systemd-cryptsetup attach mytest /dev/nvme0n1p2 none fido2-device=auto
```

## What actually happens at boot

At boot, `systemd-cryptsetup` waits briefly for a FIDO2 device. If one
enrolled for this volume is present, it prompts for the FIDO2 PIN, then
a touch, and unlocks automatically. If no key is present, or the timeout
elapses, it falls back to the standard passphrase prompt [1, 26]. This
fallback is automatic and requires no special action — it's simply what
happens when the enrolled hardware isn't there.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [1, 2, 3, 4, 26].
