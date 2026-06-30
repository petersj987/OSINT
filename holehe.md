
# OSINT Tool Study: Holehe

**Environment:** Kali Linux (Python virtualenv `holehe-env`)
---

## 1. What Holehe Is

Holehe is an email OSINT tool that checks whether a given email address is registered on a wide range of websites and online services (Twitter, Instagram, Spotify, Pinterest, Adobe, etc.) — without logging in or alerting the account owner.

Analogy: testing a single key against many locks and observing which ones click, without the building owner knowing.

## 2. Use Case in Investigations

- Recon/OSINT phase of a pentest or investigation when a target email is known
- Mapping an email's digital footprint (which platforms it's tied to)
- Confirming an email is actively in use
- Pivoting to new platforms (profile photos, usernames, bios) for further identification
- **Personal use case:** auditing your own digital footprint for cleanup/OPSEC purposes

## 3. How It Works (Technical)

Holehe abuses each target site's own "forgot password" or "sign up" flow. Most sites respond differently depending on whether the email is already registered (e.g. "this email already exists" vs "account created"). Holehe automates these requests and parses responses into: exists / not exists / rate-limited / unknown — without completing any password reset or signup.

## 4. Installation (Kali Linux)

Kali enforces PEP 668 (externally-managed-environment), so a venv or pipx is required instead of raw `pip install`.

**Method used (venv):**
```bash
python3 -m venv ~/holehe-env
source ~/holehe-env/bin/activate
pip install holehe
```
Must run `source ~/holehe-env/bin/activate` each session before use.

**Alternative (pipx):**
```bash
sudo apt update
sudo apt install pipx -y
pipx ensurepath
pipx install holehe
```

**Verify install:**
```bash
holehe --help
```

## 5. Command Syntax

```
holehe [-h] [--only-used] [--no-color] [--no-clear] [-NP] [-C] [-T TIMEOUT] EMAIL [EMAIL ...]
```

| Flag | Purpose |
|---|---|
| `EMAIL` | Required. Target email(s); multiple can be passed in one run |
| `--only-used` | Show only sites where the account exists (cleaner output) |
| `--no-color` | Disable colored terminal output |
| `--no-clear` | Don't clear terminal before displaying results |
| `-NP, --no-password-recovery` | Skip modules relying on password-reset abuse (some are noisier/riskier) |
| `-C, --csv` | Export results to CSV (useful for reports) |
| `-T, --timeout` | Set max timeout in seconds (default: 10) |

## 6. Basic Usage

```bash
holehe target_email@example.com
```

## 7. Ethical / Legal Notes

- Holehe only queries public signup/reset endpoints of third-party sites — it does not access the target's inbox or device.
- Only run against emails you are authorized to investigate: your own accounts, or a client's email under signed engagement scope.

## 8. Personal Footprint Cleanup Workflow (when a site is flagged)

1. **Confirm ownership** — try login/"forgot password" on the flagged site to verify it's actually your account.
2. **Delete via platform settings** — check account/privacy/security settings for a delete option.
3. **Use data-deletion rights if no delete option exists** — email the site's privacy contact citing GDPR Article 17 (EU/UK) or Kenya's Data Protection Act 2019.
4. **If deletion isn't possible** — randomize password, strip personal info from profile, unlink real email if possible.
5. **Re-scan periodically** — re-run Holehe against your own email to verify deletions took effect (some platforms soft-delete and still resolve as "exists" for months).

```

