# OSINT Tool Study: h8mail

**Environment:** Kali Linux (Python virtualenv `h8mail-env`)  
**Tool Version:** h8mail v2.5.6 — "ROCKSMASSON.6"  

---

## 1. What h8mail Is

h8mail is a passive email OSINT tool focused on **breach intelligence** — it checks whether a target email address has appeared in known data breaches and leaked datasets, and surfaces what was exposed (passwords, password hashes, IPs, names, phone numbers, etc.).

**Complementary relationship with Holehe:**

| Tool | Question it answers |
|---|---|
| Holehe | *Where is this email registered?* (platform footprint) |
| h8mail | *Has this email been in a data breach — and what was leaked?* (exposure history) |

Together they form a complete passive email recon pair.

## 2. How It Works (Technical)

h8mail queries multiple breach intelligence APIs and local breach datasets, aggregating results in a single session. It does not brute-force anything — it is entirely passive, checking whether the target email appears in publicly known or API-indexed breach data.

It supports both **online API sources** (HIBP, Snusbase, Dehashed, Hunter.io, Emailrep, Leak-Lookup, WeLeakInfo) and **offline/local breach files** (cleartext or `.tar.gz` compressed dumps), making it viable even in air-gapped or limited-connectivity environments.

## 3. Installation (Kali Linux)

```bash
python3 -m venv ~/h8mail-env
source ~/h8mail-env/bin/activate
pip install h8mail
```

Activate the venv each session before use:

```bash
source ~/h8mail-env/bin/activate
```

Verify install:

```bash
h8mail --help
```

## 4. Supported API Sources

| Source | What it surfaces | Cost |
|---|---|---|
| HaveIBeenPwned (HIBP) | Which named breaches the email appeared in | ~$4/month |
| Hunter.io | Email verification + company email patterns | Free tier available |
| Snusbase | Full breach data: passwords, hashes, IPs | Paid |
| Leak-Lookup | Breach database lookups | Free + paid tiers |
| Emailrep.io | Email reputation scoring | Free tier |
| Dehashed | Comprehensive breach database | Paid |
| WeLeakInfo | Breach data aggregation | Paid |
| Scylla.so | Free breach source (intermittently online) | Free |

## 5. Key Flags

| Flag | Purpose |
|---|---|
| `-t EMAIL` | Target email(s) — required. Supports multiple and file input |
| `-c CONFIG` | Path to config `.ini` file containing API keys |
| `-o FILE` | Export results to CSV |
| `-j FILE` | Export results to JSON |
| `--loose` | Disable strict email pattern matching (fuzzy/broader search) |
| `-ch LIMIT` | Chase mode — pull related emails from Hunter.io and scan them too |
| `--power-chase` | Chase mode querying ALL services, not just Hunter.io |
| `--hide` | Show only first 4 characters of found passwords (safe for demos) |
| `--gen-config` | Generate a blank `h8mail_config.ini` API key template |
| `-lb FILE` | Scan against a local cleartext breach file (no API needed) |
| `-gz FILE` | Scan against a local `.tar.gz` compressed breach file |
| `-sk` | Skip Scylla and Hunter.io default checks |
| `--debug` | Print raw request/response data for troubleshooting |

## 6. Config File Setup (API Keys)

Generate the config template:

```bash
h8mail --gen-config
cat h8mail_config.ini
```

Fill in API keys for whichever sources you have access to, then pass the config on every run:

```bash
h8mail -t target@email.com -c h8mail_config.ini
```

**Recommended free starting keys:**
- Hunter.io — register at hunter.io for a free tier API key
- HIBP — $4/month, the most valuable breach intelligence source for investigators

## 7. Basic Usage Examples

```bash
# Keyless baseline scan
h8mail -t target@email.com

# With API keys configured
h8mail -t target@email.com -c h8mail_config.ini

# Output to CSV for reporting
h8mail -t target@email.com -c h8mail_config.ini -o results.csv

# Multiple targets
h8mail -t email1@example.com email2@example.com -c h8mail_config.ini

# Chase mode — discover and scan related emails
h8mail -t target@email.com -c h8mail_config.ini -ch 5 --power-chase

# Scan against a local breach file (offline)
h8mail -t target@email.com -lb /path/to/breach.txt
```

## 8. Reading the Output

h8mail uses status prefixes on each output line:

| Prefix | Meaning |
|---|---|
| `[>]` | Informational — action being taken |
| `[~]` | Warning or skipped step |
| `[!]` | Error (e.g. API key missing, source down) |

**Session Recap table** at the end of each run gives a clean per-target verdict:

| Status | Meaning |
|---|---|
| `Not Compromised` | No breach records found in sources checked |
| `Compromised` | Email found in one or more breach datasets |

## 9. Real Scan: Own Email (Keyless Baseline)

**Command run:**
```bash
h8mail -t your-email@gmail.com
```

**Output summary:**
- `scylla.so is down, skipping` — free source temporarily offline, skipped automatically
- `hunter.io (public API) error` — no API key configured, query rejected
- **Result: Not Compromised** — email not found in any breach database queried

**Interpretation:** Clean result under keyless conditions. Deeper scan with HIBP and Hunter.io keys would query significantly more breach data. Not Compromised here means no exposure in free/offline sources checked — not a guarantee across all breach databases.

## 10. Ethical / Legal Notes

- h8mail is entirely passive — it queries breach APIs and databases, never touches the target's inbox or device
- Only run against emails you are authorized to investigate: your own, or a client's email under signed engagement scope
- Breach data returned may contain sensitive credentials — handle, store, and report it responsibly
