# Known issues — pending doc updates

Raw, precise notes from an actual live walkthrough (2026-07-19), not yet
folded into prose in the main sections. Kept here so nothing is lost
between sessions. Each item notes which file it eventually belongs in.

## GPG section

1. **`delkey`, not `delete`**, inside `gpg --edit-key` to remove a
   selected subkey. → `04-implementation/02-gpg-subkeys-per-card.md`

2. **Card ships with a default Admin PIN (`12345678`)** and this is
   needed *during* the first `keytocard` call, before the doc's own
   "now set your PIN" step is reached. Current doc implies PIN-setting
   happens cleanly after carding; in practice you hit the Admin PIN
   prompt mid-`keytocard` and need the factory default to get through
   it. → reorder in `02-gpg-subkeys-per-card.md`, mention default PIN
   earlier.

3. **Clarify `sec` is not `key 0`.** Numbering for `key N` inside
   `gpg --edit-key` starts at the first `ssb` line. `sec` is never
   selected via `key N` — it's acted on by default when nothing is
   selected. → `02-concepts/gpg-explained.md`.

4. **`GPG_TTY` unset causes `Inappropriate ioctl for device`** on
   `--clearsign` and likely other pinentry-requiring ops. Fix:
   `export GPG_TTY=$(tty)` + `gpg-connect-agent updatestartuptty /bye`,
   and add `set -gx GPG_TTY (tty)` to fish config permanently. This
   will hit nearly everyone. → add to `04-implementation/00-prerequisites.md`,
   likely as a required step, not a troubleshooting footnote.

5. **Forgotten/wrong card User PIN**: use `gpg --card-edit` → `admin` →
   `passwd` → **option 2 (unblock PIN)**, not option 1 (which requires
   the old PIN you don't have). Unblock uses the Admin PIN instead.
   → `02-gpg-subkeys-per-card.md`.

6. **Verify `gen-revoke` output isn't empty before trusting it.**
   Redirects can silently produce a 0-byte file if the interactive
   prompt fails (e.g. due to issue #4 above, before `GPG_TTY` is set).
   Always `ls -la revoke.asc` and confirm non-zero size. Also worth
   noting: GnuPG auto-generates one at
   `<homedir>/openpgp-revocs.d/<fingerprint>.rev` at key creation —
   usable as a fallback if the manual one is bad.
   → `01-master-key.md` (partially done — the auto-generated-cert note
   is in; the "verify non-empty" check still needs adding).

7. **Verify the encrypted master key backup before shredding anything.**
   `gpg --decrypt master-secret.asc.gpg > check.asc && diff master-secret.asc check.asc`
   is necessary but not sufficient — also run
   `gpg --dry-run --import check.asc` to confirm it's a structurally
   complete, importable key, not just byte-identical to whatever you
   exported (which could itself have been broken). → `01-master-key.md`.

8. **`gpg --delete-secret-key <primary-keyid>` is a known GnuPG rough
   edge**: depending on prompt handling, it can strip local stub
   records (`ssb` → `ssb#`) for *every* subkey, not just the master's
   private key. No data is lost — physical cards are untouched, stubs
   self-heal the moment `gpg --card-status` sees that card again — but
   it's alarming and avoidable. Safer approach: find the master's own
   keygrip via `gpg --list-secret-keys --with-keygrip` (the one listed
   under `sec`, not under any `ssb`), then
   `rm "$(gpgconf --list-dirs homedir)/private-keys-v1.d/<keygrip>.key"`
   directly — can't touch subkey stub files, which have their own
   distinct keygrips/files. → rewrite the "close out the master key"
   step in `01-master-key.md` to use this method as primary, with the
   self-heal fix documented as a recovery note.

## FIDO2 / SSH section

9. **`-O verify-required` requires a PIN to already be SET on the
   device before generating the credential.** If no PIN exists yet,
   `ssh-keygen` does *not* cleanly stop and prompt you to set one — it
   can silently create a broken/orphaned resident credential (consumes
   a credential slot, but `ssh-keygen -K` later fails with
   `Unable to load resident keys: invalid format`). Fix if this
   happens: `ykman fido access change-pin`, then
   `ykman fido credentials list` / `ykman fido credentials delete <id>`
   to remove the orphaned one, then regenerate clean.
   **Doc must be reordered**: `ykman fido access change-pin` needs to
   happen *before* the first `ssh-keygen -t ed25519-sk -O resident -O
   verify-required` call, not left implicit. → `03-fido2-ssh-keys.md`,
   `00-prerequisites.md`.

10. **`ssh-keygen -K` writes to the current working directory, not
    `~/.ssh` automatically.** `cd ~/.ssh` before running it, every
    time. → `03-fido2-ssh-keys.md`, `05-bootstrap/new-machine-checklist.md`.

11. **The optional passphrase prompt during `ssh-keygen -O resident`
    is a third, independent layer** — encrypting the local stub file,
    separate from both the FIDO2 PIN and the hardware touch. Since the
    credential is resident (recoverable via `ssh-keygen -K` from the
    hardware regardless), this is low-value and recommended blank
    throughout this scheme. Also: a wrong/forgotten stub passphrase is
    trivially fixed by deleting the local file and re-running
    `ssh-keygen -K` — no need to regenerate the actual credential.
    → `03-fido2-ssh-keys.md`.

12. **Renamed local key files don't stay renamed across machines.**
    `ssh-keygen -K` on a new machine reconstructs the filename from the
    credential's `application=` string set at creation, ignoring any
    local rename done elsewhere. Recommendation: don't fight the
    auto-naming — either accept the `id_ed25519_sk_rk_key<label>`
    pattern everywhere, or be deliberate that renames are local-only
    convenience and won't propagate. → `03-fido2-ssh-keys.md`,
    `07-dotfiles.md`.

13. **THE BIG ONE — static `IdentityFile` + `IdentitiesOnly yes` in
    `~/.ssh/config`, with multiple hardware-backed identities
    registered on the same service, causes SSH to attempt a real
    signature (full PIN + touch ceremony) against EVERY listed identity
    in order until one matches the physically-present card.** This is
    an OpenSSH limitation (no cheap way to ask a security key "do you
    even hold this credential" before attempting a full challenge), not
    a config mistake. With 4 registered keys, worst case is 4 rounds of
    PIN+touch before success. This directly contradicts the "any key on
    me just works, zero thought" framing currently in
    `03-architecture/the-split.md` and the vignettes — **that framing
    needs to be corrected, not just caveated.**

    Two layers of actual fix, worked out live:
    - `ssh-add ~/.ssh/id_yb_keyN` (load only the one you're carrying)
      *can* reduce this to one round — but depends on the local
      `ssh-agent` correctly supporting FIDO2 signing via
      `ssh-sk-helper`. This failed in testing with
      `sign_and_send_pubkey: ... agent refused operation` — possibly
      because the active `SSH_AUTH_SOCK` belonged to a non-standard or
      third-party agent (the socket path
      `~/.ssh/agent/s.EEY7e2KDz9.agent...` doesn't match vanilla
      `ssh-agent`'s default naming — worth checking `ps aux | grep
      agent` to identify what's actually running; a Bitwarden SSH
      agent or similar not supporting hardware keys is a plausible
      cause). Unresolved as of this session — flag as "works if your
      agent supports FIDO2 `-sk` signing, verify before relying on it."
    - **The confirmed-working fix, adopted as the actual recommended
      pattern**: don't use a static multi-`IdentityFile` SSH config at
      all. Instead, pin one specific key per machine via
      `git config --global core.sshCommand "ssh -i ~/.ssh/id_yb_keyN"`
      (one "home" key per machine — e.g. key1 on the stationary
      desktop, key2 on laptops generally), and use the
      `GIT_SSH_COMMAND` environment variable to override *per-command*
      on the rare occasion a different physical key is in use:
      `GIT_SSH_COMMAND="ssh -i ~/.ssh/id_yb_key3" git push`.
      `GIT_SSH_COMMAND` takes precedence over `core.sshCommand`, so
      this requires no edit-then-revert step, no state left behind.
      Cold-storage keys (3 and 4, in this session's case) get *no*
      SSH-side config on any machine at all — reduces the realistic
      cycling problem to zero on the common path, and to one
      deliberate flag on the rare path.
    → **Requires real revision, not a footnote**, to:
    `03-architecture/the-split.md` (the "any key just works" claim),
    `04-implementation/03-fido2-ssh-keys.md` (remove/replace the static
    multi-`IdentityFile` config block with the `core.sshCommand` +
    `GIT_SSH_COMMAND` pattern as primary), `05-bootstrap/` (new-machine
    steps should set `core.sshCommand` for that machine's chosen key),
    `09-vignettes/a-week-in-the-life.md` (the vignettes currently imply
    zero-friction key-swapping that isn't accurate as originally
    written).

## sudo / pam_u2f section

14. **`origin=` in the `pam_u2f` line is an arbitrary label chosen at
    enrollment time (via `pamu2fcfg -o`), not derived from anything
    automatically** unless `-o` is omitted, in which case it defaults
    to `pam://$(hostname)`. It does not need to relate to hardware
    brand, model, or anything meaningful — it only needs to exactly
    match between the `u2f_keys` entry and the `/etc/pam.d/sudo` line.
    Mismatch is a **silent failure**: PIN and touch both succeed,
    authentication still fails, with no clear error pointing at the
    real cause. Recommendation: omit `-o` and let it default to
    hostname (self-documenting later), unless there's a specific reason
    to pin a different string — and if so, document *why* right next
    to the pam.d line, not just the value. → `05-sudo-pam-u2f.md`.

15. **`pam_faillock` lockout after repeated failed attempts** (3 in
    this session) is a possible side effect of sudo misconfiguration
    during setup, not something the current doc mentions at all.
    Diagnosis: `faillock --user $(whoami)`. Reset (from a still-valid
    session): `sudo faillock --user $(whoami) --reset`. If sudo itself
    is what's blocked, drop to a raw TTY and use `root` or another
    admin account. Lockout duration is configurable and may just
    require waiting it out (observed: single-digit minutes on default
    config). → add as a troubleshooting note in `05-sudo-pam-u2f.md`.

## General principle worth stating once, prominently

16. **Never trust a redirected command's output without checking it.**
    This session hit the same failure shape twice independently
    (`gen-revoke` producing a silent empty file; a stub file's optional
    passphrase silently mismatching) — a command can appear to succeed
    (no error printed) while producing garbage or nothing. Verify file
    size / round-trip decrypt / dry-run import for anything
    security-critical before deleting a source copy. Worth a short
    callout in `01-why/philosophy.md` or as its own troubleshooting
    principle, since it's a pattern, not a one-off.
