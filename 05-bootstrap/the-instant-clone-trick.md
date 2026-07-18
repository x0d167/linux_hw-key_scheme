# The instant-clone trick

The single most useful operational fact in this entire document:
**GitHub and GitLab both expose your own public keys at a plain,
unauthenticated URL.** This is what makes zero-file-transfer bootstrap
actually work, rather than just sound nice in theory.

## The endpoints

```bash
curl https://github.com/<username>.keys
curl https://github.com/<username>.gpg
```
The first returns every SSH public key on your account, one per line,
already in `authorized_keys`-compatible format. The second returns your
full GPG public key material, ready to pipe straight into `gpg --import`
[20]. Both are public by design — this is how GitHub lets anyone verify
a signature or add you to `authorized_keys` without needing an API
token, and it works exactly as well for bootstrapping your *own* new
machine as it does for anyone else [20].

GitLab exposes equivalent data through its API
(`/api/v4/users/<username>/keys`), though not through as clean a plain
URL — check GitLab's current API docs if self-hosting or using
GitLab as primary.

## Why this matters more than it sounds like it should

The instinct on a brand new machine is to reach for a USB drive, a
password manager's secure notes, or a scp from another machine, to get
your public keys in place. All of that is unnecessary work for data
that was never secret in the first place. Every public key in this
scheme is, definitionally, safe to fetch over an open connection — the
entire point of public-key cryptography is that the public half can be
shouted from the rooftops without weakening anything.

## Practical one-liners

Pull your SSH keys straight into `authorized_keys` on a server you're
setting up:
```bash
curl https://github.com/<you>.keys >> ~/.ssh/authorized_keys
```

Pull and trust your GPG identity in one pass:
```bash
curl https://github.com/<you>.gpg | gpg --import
gpg --edit-key <keyid>
gpg> trust
    → 5
gpg> save
```

## The one thing this doesn't solve

This trick handles *public* key material only, by definition. It does
nothing for the private half — that only ever exists on whichever
physical card is plugged in (FIDO2 side) or already lives there (GPG
side). That's the correct division of labor: the instant-clone trick
solves "how does this new machine learn what my public identity looks
like," while the hardware key itself is what solves "how does this new
machine prove it's actually me."

## Sources for claims on this page

Full citations in `appendix/sources.md`: [20].
