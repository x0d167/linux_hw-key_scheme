# Border crossing notes

**Not legal advice.** This page describes the technical situation only.
Whether an authority can lawfully compel you to unlock a device, and
what happens if you decline, varies enormously by country, your
citizenship/visa status relative to that country, and specific
circumstances — genuinely a legal question, and one worth researching
for any jurisdiction that's a real concern for you specifically, ideally
before you're standing at the border needing the answer.

## What the technical situation actually is

A laptop that is powered off, with full-disk encryption via LUKS, and
with no physical key attached to it, is — as a pure technical matter —
inert to anyone who doesn't have the passphrase [4]. This is the same
property described throughout `03-architecture/recovery-design.md`, and
it doesn't require any special border-specific configuration; it's just
what "powered off and encrypted" already means.

## What it doesn't buy you

- **A device that's on, unlocked, or asleep-not-off** is a different
  situation entirely — no encryption scheme protects an already-decrypted
  session. Fully powering down (not just closing the lid) before
  crossing is the behavioral piece that makes the rest of this relevant
  at all.
- **A physical key carried alongside the device** somewhat defeats the
  purpose if both are taken together — see the general point in
  `01-why/threat-model.md` about a device seized while a key is attached
  to it being out of scope for any software mitigation.
- **Compelled disclosure of a passphrase** is the actual legal question
  mentioned above, not a technical one this document can answer.
- **Cloud-synced or backed-up data** reachable from accounts on the
  device is a separate exposure entirely — encrypting local disk
  contents says nothing about what's recoverable from a logged-in cloud
  account.

## A practical, non-legal suggestion

If border crossings with sensitive personal devices are a recurring
concern, consider what data actually needs to physically travel with
you at all. A machine with minimal local secrets and heavy reliance on
this scheme's remote/self-hosted infrastructure
(`01-why/threat-model.md`'s home server and VPS) may be a smaller
practical exposure than a laptop carrying years of accumulated local
history, independent of any legal question about compelled unlocking.

## Sources for claims on this page

Full citations in `appendix/sources.md`: [4].
