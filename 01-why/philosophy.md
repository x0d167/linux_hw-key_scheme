# Why bother

## The problem this solves

Most people who end up here didn't start by wanting a "hardware key
scheme." They started with a pile of smaller annoyances that eventually
add up:

- A different SSH key sitting on every laptop, because it was easier to
  generate a new one than to figure out how to move the old one safely.
- Six-plus entries under Settings → SSH and GPG Keys on GitHub, half of
  them from machines that don't exist anymore, and no memory of which is
  which.
- A LUKS passphrase that's either long enough to be secure and
  miserable to type every boot, or short enough to type quickly and not
  actually doing its job.
- `sudo` asking for a password twelve times a day for genuinely trivial
  commands, which trains you to either pick a weak password or start
  resenting the prompt.
- No real plan for what happens if a laptop is lost or stolen, beyond
  vague unease.

None of these are exotic problems. They're what happens by default when
you run more than one Linux machine for a few years without deciding on
a system up front.

This document is that system, decided on once, written down, and made
repeatable.

## What "good" looks like here

Three properties, in order of importance:

**1. Losing a device should be an inconvenience, not an emergency.**
If a laptop or a physical key goes missing, you should know exactly
what to do, be able to do it in minutes, and have confidence about
what the person who has it can and can't do with it.

**2. Setting up a new machine should take minutes, not an evening.**
The test used throughout this document: on a freshly installed system,
how fast can you get to a working `git clone` of a private repo,
authenticated and signed, with zero manual file transfer? See
`05-bootstrap/` for the answer.

**3. None of this should depend on trusting one company forever.**
Every mechanism used here — LUKS2, GnuPG/OpenPGP, FIDO2/WebAuthn,
`systemd`, PAM — is an open standard or open-source software with more
than one independent implementation. Nothing in this scheme requires an
Apple ID, a Google account, or a subscription to keep working. That
matters on its own merits, and it also happens to be the more durable
engineering choice: open standards don't get deprecated by one
company's product roadmap.

## What this is explicitly not trying to do

This is worth stating up front, because a lot of security advice online
is written for a threat model you probably don't have.

- This is not a guide for defending against a targeted, well-resourced
  attacker (a nation-state intelligence service, a determined
  organized-crime operation with physical surveillance capability). If
  that's your actual situation, this document is not sufficient and you
  need different, more paranoid advice than what's here.
- This does not protect a device that is stolen while it's powered on
  and unlocked, or while a security key is physically attached to it.
  Nothing meaningfully does. Encryption protects data at rest, not data
  in an already-unlocked session.
- This is not trying to eliminate all friction. A small, deliberate
  amount of friction (a PIN entry, a physical touch) is the entire
  mechanism by which "possession of a $50 USB device" becomes a real
  security boundary instead of theater. The goal is to put friction only
  where it's earning its keep, and remove it everywhere else.

## The guiding principle: convenience drives adherence

Every specific decision later in this document — why FIDO2 handles some
things and GPG handles others, why keys aren't cloned, why the LUKS
passphrase fallback never goes away — traces back to one observation:
**a security practice you find annoying is a security practice you will
eventually work around, disable, or quietly stop following.**

The single biggest failure mode in personal security setups isn't a
sophisticated attack. It's the twelve-character LUKS password that got
shortened to eight after the third time someone mistyped it half-asleep,
or the disk encryption that got disabled entirely because it made an
already-slow laptop feel unusable.

So the standard applied throughout this document isn't "maximum
security." It's "the most security this specific person will actually
keep using, every day, without cutting corners." That's a genuinely
different design goal, and it's the one this document optimizes for.
