# Command cheat sheet

Every command from the implementation and recovery sections, no prose.
Read the linked section first if any of this is unfamiliar — this page
assumes you already understand why, and just wants the how.

## Prerequisites (04-implementation/00)
```bash
# Arch
sudo pacman -S --needed gnupg pcsclite ccid yubikey-manager pam-u2f libfido2 opensc
# Fedora/RHEL
sudo dnf install -y gnupg2 pcsc-lite ccid yubikey-manager pam-u2f libfido2 opensc
# Debian/Ubuntu
sudo apt install -y gnupg2 pcscd scdaemon yubikey-manager libpam-u2f libfido2-1 libu2f-udev opensc

sudo systemctl enable --now pcscd.service
ykman info
gpg --card-status
```

## Master key (04-implementation/01)
```bash
gpg --expert --full-generate-key
gpg --list-secret-keys --keyid-format=long
gpg --gen-revoke <keyid> > revoke.asc
gpg --export-secret-keys --armor <keyid> > master-secret.asc
gpg --export --armor <keyid> > pubkey.asc
```

## GPG subkeys per card (04-implementation/02)
```bash
ykman info
gpg --expert --edit-key <keyid>
#   addkey → (10) sign only → curve25519 → 3y
#   addkey → (12) encrypt only → curve25519 → 3y
#   save
#   key N → keytocard → Signature key → key N
#   key M → keytocard → Encryption key → key M
#   save
gpg --card-edit
#   admin → passwd
gpg --card-status
echo "test" | gpg --clearsign
```

## FIDO2 SSH keys per card (04-implementation/03)
```bash
ssh-keygen -t ed25519-sk -O resident -O verify-required -O application=ssh:<label> -f ~/.ssh/id_yk_<label>
```

## LUKS enrollment (04-implementation/04)
```bash
sudo systemd-cryptenroll --fido2-device=auto /dev/nvme0n1p2
# root partition: regenerate initramfs after crypttab/hook changes
sudo mkinitcpio -P          # Arch
sudo dracut -f               # Fedora / dracut-based
```

## sudo pam_u2f (04-implementation/05)
```bash
mkdir -p ~/.config/Yubico
pamu2fcfg > ~/.config/Yubico/u2f_keys
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys   # additional cards
# /etc/pam.d/sudo:
#   auth sufficient pam_u2f.so cue
#   auth include system-auth
```

## Git hosting (04-implementation/06)
```bash
gpg --armor --export <keyid> > pubkey.asc     # upload once to GitHub/GitLab
git config --global user.signingkey <keyid>!
git config --global commit.gpgsign true
git config --global tag.gpgsign true
git commit --allow-empty -m "test signing"
git log --show-signature -1
```

## New machine bootstrap (05-bootstrap)
```bash
ssh-keygen -K
curl https://github.com/<you>.gpg | gpg --import
gpg --edit-key <keyid>
#   trust → 5 → save
curl https://github.com/<you>.keys >> ~/.ssh/authorized_keys
```

## Lost one key (07-recovery/lost-one-key)
```bash
export GNUPGHOME=$(mktemp -d)
gpg --import master-secret.asc
gpg --edit-key <keyid>
#   key N → revkey → key N
#   key M → revkey → key M
#   save
gpg --armor --export <keyid> > pubkey-updated.asc
unset GNUPGHOME

sudo systemd-cryptenroll /dev/nvme0n1p2 --wipe-slot=<slot-number>
```
