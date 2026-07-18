# A week in the life

Everything in this document is in service of these few paragraphs. If
the scheme is working, it should be nearly invisible day to day.

## Monday morning, at the home desk

Sam's Arch laptop wakes from suspend. LUKS never needed re-entry since
boot was hours ago. A `git pull` on a private repo, an SSH session into
the Proxmox box to check on Syncthing — both just work, whichever of
the two daily-carry YubiKeys happens to be in the laptop's USB port
today, no thought given to which one it is. A `git commit` prompts a
quick touch for the GPG Sign subkey; the commit shows up "Verified" on
GitHub a minute later without Sam doing anything special to make that
happen.

## Wednesday, out with just a phone

Sam's on the bus, phone only, no laptop. A quick check that the home
server's Nextcloud sync caught up overnight: open Termius, tap the
NFC-carried YubiKey to the back of the phone when prompted, in an SSH
session in under fifteen seconds. No laptop needed, no separate mobile
credential to have set up in advance — it's the exact same resident
credential that lives on the card, doing the exact same job it does on
desktop.

## Thursday, an urgent VM

A project needs a scratch VM, right now, to test something before a
deadline. Fresh Arch install, terminal open. Following
`05-bootstrap/new-machine-checklist.md`: prerequisites installed,
`ssh-keygen -K` with the card that's on hand, GPG public key pulled
straight from `github.com/sam.gpg`, and a private repo cloned and
building inside a few minutes. Nothing copied by hand, no waiting on
access to another machine to transfer a key file.

## Saturday, a key goes missing

Sam gets home and the daily-carry key that usually lives in a jacket
pocket isn't there — must have fallen out somewhere during the day. Not
panic, just a checklist:
`07-recovery/lost-one-key.md`, using the second daily-carry key still on
hand and the offline master backup at home. Twenty minutes later,
that card's GPG subkeys are revoked, its SSH entry is removed from
GitHub, and the household's actually-not-that-urgent LUKS/`sudo`
cleanup on each laptop is queued for whenever Sam next sits down at
each one. The rest of the weekend proceeds normally on the remaining 3
cards.

## The pattern across all four

Nothing above involved typing a long passphrase under pressure, waiting
on file transfers, hunting for which of several SSH keys applies to
which machine, or treating a lost key as an emergency requiring
immediate, disruptive action everywhere at once. That's the entire
design goal from `01-why/philosophy.md` — the security is real, but the
friction is scoped to exactly the moments where a touch or a PIN is
doing genuine work, and nowhere else.
