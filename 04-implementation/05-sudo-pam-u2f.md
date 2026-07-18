# sudo via pam_u2f

Per-machine. Registers whichever cards you want this machine to accept
for `sudo`, while keeping the normal password fallback intact — per
`03-architecture/recovery-design.md`, this fallback is never removed.

## Enroll each card

```bash
mkdir -p ~/.config/Yubico
pamu2fcfg > ~/.config/Yubico/u2f_keys
```
For each additional card you want this machine to accept, append rather
than overwrite:
```bash
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys
```
`-n` appends a new-line-separated entry rather than starting a fresh
file [23]. You'll be prompted to touch each card as it's registered.

## Configure PAM for sudo

Edit `/etc/pam.d/sudo`, adding the `pam_u2f` line **above** the existing
password stack, not replacing it:
```
auth sufficient pam_u2f.so cue
auth include system-auth
```
(On Debian/Ubuntu, the fallback line is typically
`@include common-auth` instead of `auth include system-auth` — check
the existing content of the file before editing, since exact fallback
line syntax varies by distro [5].)

`cue` prints an explicit "please touch the device" prompt in the
terminal — worth keeping, since a silent wait for a touch is a
confusing UX otherwise.

## What this produces day to day

Typing `sudo <command>` now prompts for a touch on any registered card.
If no card is present, PAM falls through to the normal password prompt
automatically — no special flag or fallback command needed, this is
just how the `sufficient` control value behaves when the preceding
module can't succeed [5, 23].

## A note on multi-user or centrally-managed setups

The `u2f_keys` file above lives in the user's home directory and is
the right choice for a single-user personal laptop, which is Sam's
situation throughout this document. If a machine ever needs to support
multiple separate user accounts each with their own registered keys,
`pam_u2f`'s own documentation covers a central mapping file approach
instead — worth reading directly if that situation arises, since it's
a meaningfully different setup than what's shown here [5, 23].

## Sources for claims on this page

Full citations in `appendix/sources.md`: [5, 23].
