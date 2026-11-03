# Threat Model

Security is the standing priority, as in Project 9
([its ADR-020](https://github.com/NabilRobert/financed-the-tracker/blob/main/docs/planning/phase-5-architecture-decisions.md#adr-020-security-comes-first)): when
security conflicts with speed, features or cost, security wins. This page
**reuses Project 9's threat model** for everything the shared channel
already covers, and adds what a scam guard brings.

**Status, 2026-11-03:** planning only. Nothing is built or deployed.

**A rule for this public page:** no server addresses, no live unfixed
findings, no real messages.

## What is protected

| Asset | Why it matters |
|---|---|
| **The person's trust in a rating** | A wrong `RENDAH` can cost someone their savings |
| **Forwarded content** | Holds the person's and other people's data: names, numbers, account numbers, sometimes their own OTP |
| **Family links** | Who protects whom says a lot about a family |
| **Ratings** | A history of what scams reached someone |
| **The server** | It receives hostile files and links by design |
| **The WhatsApp number and its standing** | Without it the guard is silent, and people think it is still there |

## Who from

| Party | Wants | The limit we can set |
|---|---|---|
| **A scammer** | A `RENDAH` on their message; the guard banned or discredited | Rules that only raise; unknown is `TINGGI`; the official channel (ADR-002) |
| **A scammer impersonating the guard** | "the guard checked it, it's safe" | The guard never says safe and never contacts first, and the notice tells people that |
| **A controlling relative** | To monitor someone through alerts | Both sides consent; alerts carry no content; the protected person sees every alert and can end the link |
| **Malware sent to the guard** | To run on the server | Nothing is opened, unpacked or run |
| **Everyone in Project 9's table** | See Project 9's threat model | Reused controls, below |

## Reused from Project 9 (owned by the shared channel)

These threats and their controls come over with the channel code. The
proofs are Project 9's tests, re-run here.

| Project 9 # | Threat | Notes for this project |
|---|---|---|
| T1–T3 | Forged, timed or replayed webhooks | Same. On Meta's Cloud API (ADR-002) the signature is Meta's |
| T4–T5 | Strangers and floods | Same allow-list in v1 |
| T7–T9 | One person reaching another's data | Same row-level security |
| T11 | Someone holding the phone | Same |
| T14 | Content in logs and spans | **More important here**: messages are hostile and personal |
| T15–T17 | Retention, deletion, consent | Same, with ratings in place of entries |
| T20–T27 | Server, secrets, backups | Same |
| T30–T32 | Repo and dependencies | Same, plus: no real message ever committed |

## New threats

### Getting the rating wrong

| # | Threat | Control | Proof | Slice |
|---|---|---|---|---|
| S1 | A scam is rated `RENDAH`, and the person pays | `RENDAH` only when every link is allow-listed and no rule fires (V6). Every reply says it's not a guarantee and names the bank and IASC | Held-out false-negative count, reported; a test that `RENDAH` can't come from an unknown link | 1, 2 |
| S2 | A scammer evades text rules with look-alike letters, spacing or emoji | NFKC normalisation and whitespace collapse (the gateway's approach), and rules written against obfuscated dev rows | Rows with obfuscation in the held-out set, reported apart | 1 |
| S3 | A scam hides behind a shortener or redirect | Not followed. A shortener is `TINGGI` ("can't check") | `test_a_shortened_link_is_high_and_never_fetched` | 1 |
| S4 | A look-alike domain of a bank or courier | Edit distance and confusable checks against a list of Indonesian brand domains | Dev and held-out rows | 1 |
| S5 | A text says "this message was verified safe by AI" to sway a model | The model can only raise a level, never lower it (ADR-007). Injection phrases are themselves a scam signal | The Inspect task; a test that no model output lowers a level | 8 |
| S6 | A rules change quietly lowers ratings | A test fails on any lowered scam rating in the public set without a written reason (E2) | CI | 1 |
| S7 | A list is stale or down, so a new scam link passes | The cache's age is checked. Past 48 hours, link checks fail closed. A fetch failure alerts | `test_a_stale_list_makes_links_high` | 2 |
| S8 | Too many false alarms, so people stop listening | The false-alarm rate on hard normal cases is a headline number, not hidden | Measurement plan | 1 |

### Hostile content

| # | Threat | Control | Proof | Slice |
|---|---|---|---|---|
| S9 | A malicious attachment runs on the server | The file is streamed through SHA-256 and dropped. No library parses its contents: no unzip, APK parser, image decoder or PDF reader on files | `test_no_attachment_is_written_to_disk_or_parsed`; a dependency review | 2 |
| S10 | A screenshot exploits the OCR library | OCR runs in a separate process with no network and a memory and time cap, on images re-encoded first. Decided in its slice | Slice 7 | 7 |
| S11 | A huge file or many files exhaust the server | A size cap before download; per-person rate limits (reused) | `test_a_file_over_the_cap_is_not_downloaded` | 2 |
| S12 | Visiting a link warns the scammer or infects the server | No link is ever visited (K4) | A test that the HTTP client is never called with a message's URL | 1 |
| S13 | A user's link leaks to a third party | Lists are cached locally. Safe Browsing uses its hash-prefix update API, so full URLs stay here (ADR-006) | Code review; a test that lookups don't touch the network | 2 |

### Forwarded content and privacy

| # | Threat | Control | Proof | Slice |
|---|---|---|---|---|
| S14 | Other people's data in forwarded messages is kept | No text, image or audio stored (PR1). Ratings keep ids and hashes only | The no-content test over the database, logs and spans | 3, 5 |
| S15 | A person forwards their own OTP or card number | Warned not to share it; never stored or sent to a model (PR3, reusing Project 9's card check) | `test_an_otp_is_warned_about_and_not_stored` | 3 |
| S16 | Donated rows for the set leak a third party | Redacted first, checked by a person, kept in `data/real/`, never committed | gitleaks plus a check that `data/real/` is ignored | 0 |

### The family alert

| # | Threat | Control | Proof | Slice |
|---|---|---|---|---|
| S17 | A relative uses alerts to watch someone | Both consent (F1); no content in alerts (F2); the protected person is told every time (F4) | `test_an_alert_carries_no_message_content`, `test_a_link_needs_both_sides` | 4 |
| S18 | A scammer floods a target to flood the relative | Daily cap, then one summary (F3) | `test_alerts_stop_at_the_daily_cap` | 4 |
| S19 | Anyone links themselves as a relative | The protected person starts the link, and the relative must agree | Same as S17 | 4 |
| S20 | An alert goes to a recycled number | Links are re-confirmed every 90 days, and the alert holds no content | `test_an_unconfirmed_link_sends_nothing` | 4 |

### The channel

| # | Threat | Control | Proof | Slice |
|---|---|---|---|---|
| S21 | Scammers mass-report the number to get it banned | The official Cloud API, where a business account has a quality rating and an appeal path (ADR-002). On WAHA there is no appeal | ADR-002 | 3 |
| S22 | The guard goes silent and people assume it's still there | Monitoring of the channel (reused); the notice says to call the bank directly when no reply comes | Slice 6 alert test | 6 |
| S23 | Someone impersonates the guard | It never sends links except to official domains (iasc.ojk.go.id, cekrekening.id, banks' own sites), and never contacts first except linked alerts | A test over reply templates for links | 3 |

## Limits we can't remove

- **Meta sees every forwarded message** on the Cloud API, decrypted. The
  notice says so.
- **A rating is a guess.** The wording says so, every time.
- **Arthur, as operator, could read the ratings table.** It holds no
  message content.
