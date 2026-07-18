# New machine checklist

The test this whole scheme is built around: on a freshly installed
system, how fast can you get to a working, authenticated,
signature-capable `git clone` of a private repo? This page is that
sequence, start to finish.

## 1. Install prerequisites

`04-implementation/00-prerequisites.md` — package install, `pcscd`
enabled, `ykman info` confirms a card is visible.

## 2. Clone your dotfiles

This is the one bootstrapping step that needs *some* existing access —
if your dotfiles repo is public, plain HTTPS clone works with nothing
else set up first:
```bash
git clone https://github.com/<you>/dotfiles.git ~/dotfiles
```
If it's private, skip to step 4 first, then come back.

## 3. Symlink or copy portable config into place

Per `04-implementation/07-dotfiles.md` — `~/.ssh/config`,
`~/.gitconfig`, shell config. However your dotfiles repo handles this
(symlinks, a bootstrap script, GNU Stow, whatever you use) is
independent of this scheme.

## 4. Bring your SSH identity onto this machine — no file transfer

With any one of your 4 cards plugged in:
```bash
ssh-keygen -K
```
Pulls the resident credential(s) on that card down into local stub
files matching whatever filenames you used during provisioning. See
`05-bootstrap/the-instant-clone-trick.md` for an even faster variant
that doesn't require the card's own onboard label to line up with your
`~/.ssh/config` expectations.

## 5. Bring your GPG public identity onto this machine

```bash
curl https://github.com/<you>.gpg | gpg --import
```
Public key material only — this is the master key's public component
plus every card's Sign/Encrypt subkeys, all safe to fetch over plain
HTTPS since none of it is secret. Then trust it locally:
```bash
gpg --edit-key <keyid>
gpg> trust
    → 5 (ultimate)
gpg> save
```

## 6. Test the private clone

```bash
git clone git@github.com:<you>/<private-repo>.git
```
Should prompt for a touch (and PIN, given `verify-required`) on
whichever card is plugged in, and succeed with zero file transfer of
anything secret.

## 7. Machine-specific enrollment (not portable, do once here)

- `04-implementation/04-luks-disk-encryption.md` — if this machine has
  an encrypted disk, enroll whichever cards it should accept.
- `04-implementation/05-sudo-pam-u2f.md` — enroll cards for `sudo`.

## What "fast" actually looks like

Steps 1–6 above involve no manual copying of any secret material at
any point — every credential either lives on hardware you're already
holding, or is fetched as public data over HTTPS. Step 7 is the only
part that's genuinely per-machine setup work, and it's independent of
whether you need repo access *right now* — reasonable to defer it if
you're mid-task and just need `git clone` working immediately.
