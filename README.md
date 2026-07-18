# Hardware Keys for Linux: A Practical Identity, Access, and Encryption Scheme

**Status:** v1 content complete (all sections drafted), pending a
consistency review pass and eventual image/PDF pass. See `CHANGELOG.md`
for full detail.

## What this is

A from-scratch, distro-agnostic scheme for using a handful of hardware
security keys (YubiKeys or equivalent) to handle disk encryption, `sudo`,
SSH access, and Git commit signing across multiple machines — without
juggling a different key for every device, and without pretending you're
defending against a nation-state.

This is written for one hypothetical person, described in full in
`01-why/threat-model.md`, but the design generalizes to most technically
comfortable Linux users who run more than one machine and are tired of
key sprawl.

It deliberately does **not** cover NixOS-specific configuration. The
underlying scheme (GPG smartcard + FIDO2) is identical on any Linux
distribution; only the *how do I declare this in config* step differs on
NixOS, and that gets its own follow-up document once this one is solid.

## Who should read this

Anyone who:
- Runs Linux (Arch, Fedora/RHEL-family, or Debian/Ubuntu-family) on more
  than one machine
- Has accumulated a pile of SSH/GPG keys across GitHub, GitLab, and
  servers and wants to consolidate
- Wants full-disk encryption that doesn't force a choice between "long
  password every boot" and "no real protection"
- Travels with personal devices and wants a sane, non-paranoid stance on
  device seizure/theft
- Cares about running on open standards and not being locked into one
  vendor's ecosystem

## How to use this repo

Read in order if you're new to this. If you already understand GPG and
FIDO2 conceptually, skip `02-concepts/` and go straight to
`03-architecture/` and `04-implementation/`.

| Section | Contents |
|---|---|
| `01-why/` | The problem, the threat model, what else was considered and rejected |
| `02-concepts/` | GPG and FIDO2 explained from first principles |
| `03-architecture/` | The actual design and why it's shaped this way |
| `04-implementation/` | Step-by-step setup, per operating system where it differs |
| `05-bootstrap/` | Bringing a brand new machine online fast |
| `06-mobile/` | iOS and Android |
| `07-recovery/` | What to do when a key is lost, stolen, or all of them are gone |
| `08-roadmap/` | Things intentionally left out of v1 (secrets management, PIV/age) |
| `09-vignettes/` | What a normal day actually looks like under this scheme |
| `appendix/` | Command cheat sheet, sources, FAQ |

## License

Text in this repo: CC BY-SA 4.0 — copy it, adapt it, hand it to friends,
just keep it open. Commands and config snippets: public domain / do
whatever you want with them.
