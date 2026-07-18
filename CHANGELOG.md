# Changelog

## [Unreleased] — v1 content complete, pending review pass

### Drafted — full document
- `01-why/` — philosophy, threat model (persona "Sam"), alternatives
  considered
- `02-concepts/` — GPG explained, FIDO2 explained, glossary
- `03-architecture/` — the split, key topology, recovery design
- `04-implementation/` — prerequisites through dotfiles, all 8 files,
  three distro families (Arch/Fedora/Debian) throughout
- `05-bootstrap/` — new machine checklist, the instant-clone trick
- `06-mobile/` — iOS, Android
- `07-recovery/` — lost one key, lost all keys, border crossing notes
- `08-roadmap/secrets-management.md`
- `09-vignettes/a-week-in-the-life.md`
- `appendix/` — sources (26 entries), command cheat sheet, FAQ

### Not yet done
- Review pass: a full read-through for consistency, since this was
  drafted across multiple sessions and cross-references between
  sections haven't been independently verified end to end.
- Images/diagrams — currently text-only per the README's stated
  scope; a visual pass is a deliberate later step, not an oversight.
- PDF export/formatting for the print-and-hand-to-a-friend use case.
- The separate NixOS follow-up document (explicitly out of scope for
  this repo).

### Decisions locked in during drafting (see relevant section for reasoning)
- FIDO2 handles all *access*: disk unlock, `sudo`, SSH. GPG handles all
  *identity*: commit signing, file/email encryption.
- Every physical key gets its own distinct GPG signing/encryption subkey
  pair and its own distinct FIDO2 SSH credential — no key material is
  cloned across devices.
- One GPG master key backs every card's subkeys; this still resolves to
  a single entry on GitHub/GitLab despite each card being cryptographically
  distinct.
- No GPG Authenticate subkey is generated at all in this scheme — SSH
  is FIDO2-only.
- Subkey expiry: 3 years, renewed from offline backup, cards not touched
  during renewal.
- FIDO2 SSH keys use resident, verify-required credentials.
- `pass`/`sops`/`age` and PIV explicitly deferred to `08-roadmap/`,
  not part of v1.
- NixOS-specific declarative config deferred to a separate document.

### Source list
26 sources, numbered append-only — see `appendix/sources.md`.
