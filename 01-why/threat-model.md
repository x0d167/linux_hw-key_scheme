# The reference persona and threat model

Security advice that doesn't name its threat model is usually wrong for
your situation, either too paranoid to be usable or too casual to be
useful. This section names it explicitly. Every design decision later in
this document can be traced back to something on this page.

We'll call the reference person **Sam** throughout the rest of this
document, including the day-in-the-life vignettes at the end.

## Sam's setup

- 4 laptops running Linux: 3 Arch, 1 Fedora, with a 5th laptop
  (NixOS) coming online soon. The scheme is written to be identical
  regardless of which of these is in front of Sam.
- 2 tablets, 1 iPhone, 1 Android phone, all current OS versions.
- A home server (Proxmox) running Syncthing, a self-hosted cloud
  (Nextcloud-family), and NAS storage, plus one unused remote VPS
  earmarked for a future self-hosted blog or similar.
- 4 YubiKeys. The intent is 2 carried day-to-day and 2 kept in cold
  storage, but that split may shift to 3 carried / 1 cold depending on
  travel. The scheme needs to not care which split is in effect at any
  given time — any key Sam has on them should just work.
- Comfortable in a terminal, can script, can troubleshoot and search
  effectively. Not a systems engineer, not a kernel contributor. An
  engineering student building toward professional dev/maker work, so
  this skill level is a floor, not a ceiling.
- Frequent GitHub/GitLab use over SSH, a git-based personal notes/blog
  setup, and occasional need to spin up a local or remote VM and get it
  fully credentialed fast.
- Values FOSS, portability, ownership, and privacy, but isn't dogmatic
  about it — a well-justified compromise is fine. Explicitly not a
  high-value target. Values low daily friction highly, on the correct
  belief that friction is what causes security hygiene to erode over
  time.

## In scope — what this scheme actually defends against

| Scenario | What actually happens to Sam |
|---|---|
| Laptop lost or stolen, powered off | Disk is LUKS-encrypted. Attacker has an inert brick without the passphrase or an enrolled key [4]. |
| One physical YubiKey lost or stolen (in isolation) | That key's SSH credential and GPG subkey pair are revoked. No other key or device is affected. See `07-recovery/lost-one-key.md`. |
| A machine gets compromised by malware while Sam is using it | Blast radius is limited to whatever that session could reach — hardware-backed keys can't be exfiltrated as files, so the malware doesn't walk away with reusable credentials [6, 8]. |
| Sam is asked to unlock or hand over a personal device at a border crossing | Sam can decline to unlock a powered-off, LUKS-encrypted device; without the passphrase or a physical key, the data is not accessible. This is about the technical situation only — see the note on legal limits below. |
| Sam needs to stand up a new or reinstalled machine fast, mid-project | Public keys are fetchable from GitHub/GitLab directly; no private key material ever needs to be copied by hand. See `05-bootstrap/`. |
| Credential stuffing / phishing against GitHub, GitLab, or self-hosted services | Hardware-backed auth can't be phished the way a password or TOTP code can — there's no secret to trick Sam into typing into a fake site [6, 8]. |

## Explicitly out of scope

Naming these matters as much as naming what's in scope.

- **A targeted, well-resourced adversary** (state intelligence services,
  a sophisticated and motivated organized-crime actor). If this is
  genuinely your situation, you need a different document than this
  one, likely with professional guidance.
- **Evil-maid attacks** — an adversary with extended, repeated physical
  access to a powered-off machine to install a hardware or firmware
  implant. Mitigating this needs measured/attested boot, which is out of
  scope here (see `08-roadmap/` for what's deferred and why).
- **A device that's on, unlocked, or has a key physically attached at
  the moment of seizure.** No software scheme meaningfully protects this
  case. The only mitigation is behavioral: lock your screen, and don't
  leave a key in a laptop you're not actively using.
- **Legal compulsion to unlock a device or reveal a passphrase.**
  Whether a border agency, employer, or court can lawfully compel this
  varies enormously by jurisdiction and circumstance, and is genuinely a
  legal question, not a technical one. This document describes what is
  technically possible; it is not legal advice, and Sam should look into
  the actual law in relevant jurisdictions if this is a real concern.

## Why this framing matters

A huge amount of hardware-key advice online is written by people whose
actual job is defending against sophisticated, resourced attackers, and
the advice reflects that: cold-boot attack mitigations, tamper-evident
seals, air-gapped signing machines, TPM-sealed measured boot chains. All
of that is legitimate for its intended audience and mostly wasted effort
for Sam. Applying it anyway doesn't make Sam safer in any way that
matters for Sam's actual life — it just adds friction, and per the
philosophy in the previous section, friction is what gets abandoned
first under real-world pressure.

Note on source [4]: full-disk encryption via LUKS2 protects data at rest
only; it does nothing once the volume has been unlocked. Note on
sources [6, 8]: this is the core security property of hardware-backed
keys under both the OpenPGP card standard and FIDO2/WebAuthn — private
key material is generated on the device and never leaves it. Full
citations are in `appendix/sources.md`.
