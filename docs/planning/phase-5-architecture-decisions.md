# Phase 5: Architecture Decisions

Short ADRs, 2026-11-03. **None is settled yet:** each is a recommendation
until Arthur agrees. Settled ones will be marked with the date. The summary
table, the first slice and the open questions are at the end.

```
Person forwards ─► WhatsApp ─► WAHA (GOWS, loopback) ─► channel (Project 9's adapter) ─► guard (FastAPI)
                                                                                    │
                     text rules (YAML) ─────────────────────────────────────────────┤  $0
                     link checks, no visiting ──► local list cache ◄── fetch job    │  $0
                     file checks, hash only ───► local list cache                   │  $0
                     model, off in v1 ─────────────────────────────────────────────┤  may only raise
                                                                                    ▼
                                        highest level wins ─► reply ─► alert to linked relative
                                                 │
                                                 ▼
                                   Postgres: ratings as ids and hashes, no content
```

## Reuse map

Arthur's repos were searched first (`grep` for scam, phishing, url, apk,
hash, injection). No repo checks links or files. What fits:

| Part | Reused | Built (and why) |
|---|---|---|
| WhatsApp channel, webhook checks, repeats, allow-list, consent with version, `hapus data saya`, no-content tests | **Project 9** (`tracker/channel/`, its threat model and tests) | Nothing, if Project 9 splits the channel out (ADR-003) |
| Text rules: YAML, NFKC normalisation, version as a hash of the file | **mcp-gateway's `RulesDetector` approach** (`src/mcp_gateway/detect.py`) | The Indonesian scam rules themselves |
| Scoring once per version on a held-out split | **mcp-gateway's `eval/score.py` discipline** | The scorer for this set |
| Injection phrases aimed at AI checkers | **mcp-gateway's `rules/injection.yaml`**, as a scam signal | Indonesian phrases |
| Budget guard and prices for paid runs | **agent-eval-harness** `budget.py`, `prices.yaml` | — |
| Model eval | **Inspect AI**, as in Project 9 | The task |
| Traces and alerts | **agent-observability**, **systems-monitoring** | Their config entries, in their repos |
| Bad URLs and files | **Free lists** (ADR-006) | The fetch job and cache |
| Domain parsing, edit distance | **tldextract**, **rapidfuzz** (ADR-011) | — |
| Scam rules and the brand-domain list | Nothing exists for Indonesian scams | **Built:** this is what the project demonstrates |

**Why not depend on mcp-gateway as a package:** it pulls in `mcp`,
`onnxruntime`, `tokenizers` and `numpy` for a regex loader of about 30
lines. The folder's rule allows plain code when a dependency is heavier
than the problem. The approach is reused and credited, not the package. If
the gateway later ships its rules loader without the heavy extras, this
switches to depending on it.

## ADR-001: Users in v1: Arthur's family, by invitation

**Recommendation:** up to 10 people, added by Arthur by number, as in
Project 9's allow-list. Each can be a protected person, a relative, or
both.

**Why:** real use with real scams, without public sign-up, a verified
business or a support load.

**Rejected:** *public from day one:* a support and abuse load, and on WAHA
more traffic means a faster ban. *Arthur only:* the family
alert can't be tested alone.

## ADR-002: The channel: WAHA, as in Project 9 (settled)

**Decision (settled by Arthur, 2026-11-03):** WAHA with the GOWS engine,
behind Project 9's adapter, the same as Project 9's ADR-022. The
recommendation was Meta's Cloud API; Arthur chose WAHA. The reasoning for
both stays here, because the risk taken is part of the record.

**The recommendation, and why it lost.** A scam guard differs from a
ledger on four points. Arthur accepted all four:

| Point | Project 9 (ledger) | Project 12 (scam guard) | Accepted handling |
|---|---|---|---|
| **What a ban costs** | Replies stop; records are safe in Postgres | Protection stops silently, when a scam may be arriving | The notice says: no reply within a minute means call the bank directly. Channel monitoring alerts Arthur (S22). Ratings and links stay in Postgres, so a new number needs no new consent |
| **Who wants it banned** | No one | Scammers, by mass-reporting the number. No appeal on WAHA | Accepted. A spare SIM is ready to re-link |
| **Starting conversations** | Never | **The family alert starts one** with the relative. WAHA's guidance says a bot shouldn't | Only to a relative who agreed in a chat with the guard (`setuju jaga`), at most 3 a day, level and type only (ADR-009). The single exception to Project 9's reply-only rule (T41), tested as such |
| **Who pays later** | No one | Banks and telcos won't accept an unofficial client | Moving to the Cloud API is a gate before any sale. The adapter keeps that door open |

**What WAHA changes from a Cloud API plan:**
- **WhatsApp's terms are broken,** as in Project 9 ("bulk messaging,
  auto-messaging", "non-personal use", reverse engineering). Same open
  question as Project 9's, and it gates real use.
- **Meta no longer reads the text,** by WhatsApp's own description; the
  server does. Unverified, as in Project 9.
- **No Meta prices, templates or test number.** The cost is a SIM.
- **Attachments:** WhatsApp's media messages carry the file's SHA-256 in
  the message itself. If WAHA passes it through on GOWS, the guard can look
  up an APK **without downloading it**. The live check confirms this. If
  not, WAHA's media download is kept in a memory-only volume with a short
  lifetime, and never parsed (S24).
- **Identity:** `@lid` or `@c.us`, as in Project 9's ADR-022.
- **No buttons:** typed words, as in Project 9.

**Rejected:**
- *Meta's Cloud API (the recommendation):* Arthur's call; approval is
  harder, as Project 9 found.
- *Telegram:* families in Indonesia are on WhatsApp.
- *Twilio or another official provider:* Meta's approval plus a fee.
- *A web form:* the point is to check where the message arrives.

## ADR-003: Reuse Project 9's channel as a dependency, not a copy

**Recommendation:** at slice 3, ask Project 9's session to split its
channel code (`gateway.py` interface, webhook checks, repeats, allow-list,
consent, deletion) into a package this repo depends on, by git. Until
then, nothing is copied. Slices 0 to 2 don't need it.

**Why:** the folder's rule: depend, don't copy, and fix in the owner's
repo. Project 9's channel is mixed with its ledger (`people.py`, consent
text), so the split is real work and belongs to that session.

**Rejected:** *copying `tracker/channel/`:* two versions of security code
drift. *Writing a new channel:* the same bugs found again.

## ADR-004: Risk levels only, and the wording

**Recommendation:**
- Three levels: **TINGGI**, **SEDANG**, **RENDAH**. Never "aman", "safe",
  "terpercaya", "terverifikasi" or a green tick as a verdict (V1).
- Every reply: the level, the scam type if known, what to do now, and one
  closing line: "Ini perkiraan, bukan jaminan. Kalau ragu, jangan
  transfer, jangan pasang aplikasi, dan hubungi bank lewat nomor di kartu
  atau aplikasi resminya."
- Every **TINGGI** reply adds: report to IASC at iasc.ojk.go.id, or OJK's
  contact 157; check an account at cekrekening.id; block and report the
  sender in WhatsApp.
- The notice says: the guard never contacts anyone first except a linked
  relative, never asks for an OTP, PIN or password, and never says a
  message is safe. So a message claiming otherwise is a scam.
- It never claims to be, or to work with, OJK, IASC, Komdigi or a bank.

**Why:** liability and honesty. A "safe" that turns out wrong is the one
answer that causes harm. This is engineering judgement on wording, not
legal advice; a lawyer's read comes before any paid plan.

**Rejected:** *a percentage score:* false precision. *Two levels:* hides
"we can't tell". *"Probably safe":* still read as safe.

## ADR-005: Checks never visit, open or run anything

**Recommendation:**
- **Links:** parsed, never fetched. Checks: look-alike brand domains,
  punycode, raw IPs, shorteners, free-hosting subdomains, then the lists.
- **Files:** name, declared type and size from the message. The bytes are
  streamed through SHA-256 and dropped. No unzip, APK parser, PDF reader or
  image decoder ever touches a file.
- **Unknown is TINGGI** for links and files, with the reason (V4). Any APK
  is TINGGI (V5).

**Why:** visiting a link tells the scammer it was opened and exposes the
server. Opening a file is how malware runs. A list lookup is enough to be
useful, and "can't check" is honest.

**Rejected:**
- *A sandbox or detonation service:* heavy, risky on a shared server, and
  third-party sandboxes take the file (and the person's data) with them.
- *Expanding shorteners with a HEAD request:* still a visit.
- *Uploading files to VirusTotal:* uploads are shared with its community,
  and its public API is for non-commercial use (from memory; not used, so
  not re-read).

## ADR-006: Lists, with their licences

Read 2026-11-03. **Every free list here is for non-commercial or
fair-use use.** That's fine for the portfolio and the family. Before any
paid plan, each needs its commercial terms (ADR-014).

| List | Gives | Terms | Use |
|---|---|---|---|
| **URLhaus** (abuse.ch) | Malware URLs | Free under fair use, with a free Auth-Key. "Commercial or for-profit needs may require a paid subscription" | Yes: fetched into the cache |
| **MalwareBazaar** (abuse.ch) | Malware file hashes, with tags | Same as URLhaus | Yes: hash lists into the cache; spike 2 checks sizes and whether a per-hash lookup is needed |
| **Phishing.Database** (GitHub) | Phishing domains and links, aggregated | MIT; pushed 2026-10-02 | Yes |
| **Google Safe Browsing** | Malware and phishing URLs | Free, "non-commercial use" only; Web Risk is Google's paid product for commercial use | Yes, through the **update API** (local hash prefixes), so full URLs don't leave the server |
| **PhishTank** (Cisco) | Verified phishing URLs | App key; terms are a Cisco licence not read here; whether new keys are issued is unclear | Spike 2 decides |
| **OpenPhish** community feed | Phishing URLs | Terms page not reachable today | Spike 2 decides |
| **Tranco** top sites | Popular domains, for the allow-list | Terms to read in spike 2 | Only after removing hosting, shortener and free-site domains: popular is not the same as safe |
| **Indonesian brand domains** | Banks, couriers, e-commerce, government | Hand-made from official sites | **Built.** The look-alike checks need it |
| **cekrekening.id** | Reported bank accounts | No public API found | **Pointed to, not scraped** |
| **IASC's reported phone numbers** | 145,061 numbers by July 2026 | Not public | Not available |

**Rejected:** *VirusTotal* (above). *Scraping cekrekening.id:* no API and
no permission.

## ADR-007: A model only as a fallback, only raising, off in v1

**Recommendation:**
- Off until the rules baseline is published and Arthur approves a budget.
- When on, it sees only messages the rules rate SEDANG with no link or file
  hit, after phone numbers, account numbers and names are redacted.
- **It may only raise a level, never lower it.** So "this message was
  verified safe" can't help a scammer.
- No tools. A typed output: level and scam type from fixed lists.
- Measured with Inspect AI on the dev and held-out sets; paid runs go
  through the harness's budget guard.
- The provider's written terms must say inputs aren't used for training
  (as Project 9's ADR-007 requires).

**Rejected:** *model first:* cost on every message, and every forwarded
message sent to a provider. *A model that can lower:* the attack surface.
*A local model:* too heavy for the small server.

## ADR-008: Privacy: forwarded messages hold other people's data

**Recommendation:**
- **Store no content.** Text, images, audio and files are handled in
  memory and dropped. A rating keeps: person, time, level, scam type, rule
  ids, and SHA-256 hashes of links and files. Kept 90 days.
- **Logs and spans hold no content,** with Project 9's tests.
- **Donating** a message to the labelled set is a separate step, with
  consent, redacted first (measurement plan).
- **UU PDP:** the people named in a forwarded message aren't our users and
  didn't consent. Holding nothing about them is the safest position. As in
  Project 9, the household exception is not relied on. Engineering
  judgement, not legal advice.

**Who sees what:**

| Who | Sees | Keeps |
|---|---|---|
| WhatsApp (Meta) | Who messages the guard, and when. Text is end-to-end encrypted to the WAHA session, by WhatsApp's description (unverified) | — |
| WAHA's session on the server | Every message, readable | Session files only; media per S24 |
| The guard's server | The message, in memory | Ratings only |
| List providers | Nothing, except Safe Browsing hash prefixes | — |
| Model provider (if on) | One redacted message | Per its terms |
| A linked relative | Level and scam type | — |
| Arthur, as operator | The ratings table | Stated in the notice |

**Rejected:** *keeping messages to improve rules:* other people's data,
for a gain the donation step gives with consent.

## ADR-009: The family alert

**Recommendation:**
- The protected person starts the link (`tambah keluarga 08xx`); the
  relative must agree (`setuju jaga`). Either can end it.
- An alert holds the level and the scam type, never the message, link or
  sender.
- Only for TINGGI. At most 3 a day per protected person, then one daily
  summary.
- The protected person is told each time an alert is sent.
- Links are re-confirmed every 90 days.
- On WAHA (ADR-002), an alert is the one message the guard starts. It goes
  only to a relative who has agreed in a chat with the guard, so the chat
  already exists.

**Why:** the family alert is the angle, and also the most sensitive part:
done wrong, it's a monitoring tool. Consent on both sides and no content
keep it a safety net.

**Rejected:** *sharing the message with the relative:* the protected
person's privacy. *Alerts on SEDANG:* too many, and people stop reading.
*Relatives linking themselves:* surveillance.

## ADR-010: Measurement first (settled)

**Decision (settled by Arthur, 2026-11-03):** the [measurement plan](measurement-plan.md), in full,
before any rule: a real set of about 200 scam and 200 normal messages, two
labellers, a held-out split scored once per version, and a rules-only
baseline at $0. Targets are set from the baseline. Arthur settled the
sources: private in `data/real/`; the unlicensed SMS datasets privately;
messages he received and scams he finds online; messages Claude writes,
kept apart from the real set; a family member as second labeller.

**Rejected:** *a hand-written set only:* it matches how the writer thinks
scams look. *A public dataset only:* none is real, Indonesian and licensed.

## ADR-011: Stack and libraries (settled)

**Decision (libraries settled by Arthur, 2026-11-03):** Arthur's stack, as in Project 9: Python 3.12, FastAPI,
Pydantic, SQLAlchemy 2 with Alembic, Typer, pytest, uv, ruff and pyright,
Postgres, httpx, tenacity, the OpenTelemetry SDK, Inspect AI. **New
libraries, approved:**

| Library | For | Licence | Beat |
|---|---|---|---|
| **tldextract** | Splitting a host into registered domain and subdomain, with the Public Suffix List | BSD-3 | Hand-written splitting, which gets `co.id` and `go.id` wrong. Its network fetch is turned off; the list is pinned |
| **rapidfuzz** | Edit distance for look-alike domains | MIT | `difflib`, which is slower and has no plain Levenshtein |
| **PyYAML** | The rules file | MIT | JSON, which can't hold comments explaining each rule |

**Rejected:** *a scam-detection SaaS:* cost, and the messages leave.

## ADR-012: Cost and a budget

| Item | Cost | Notes |
|---|---|---|
| Rules, lists, CI | $0 | Free keys for abuse.ch and Google |
| WhatsApp replies and alerts | $0 on WAHA (ADR-002) | The cost is a ban, not a bill: caps and rate limits instead |
| Model fallback | **$0 until approved.** Proposal: a hard cap of $1 a month | Cents at family scale; each paid eval run needs Arthur's OK |
| SIMs | A dedicated SIM, plus a spare to re-link after a ban | Never a personal number |

**Rejected:** *no cap:* a loop or a flood could cost money unseen.

## ADR-013: Screenshots and voice notes come later, and are measured

**Recommendation:** v1 handles text, links and files. Screenshots come in
slice 7: OCR with Tesseract (Apache-2.0, has Indonesian), in a separate
capped process with no network, on re-encoded images. Voice notes need
speech-to-text, which is heavy for the small server; a spike measures
whisper.cpp's smallest multilingual model before any decision.

**Why:** both add a parser of hostile input and server load. Text covers
most forwarded scams; the set shows how many arrive only as images.

**Rejected:** *a cloud OCR or speech API:* the image or audio leaves, with
other people's data in it.

## ADR-014: Who pays: portfolio first

**Recommendation:** portfolio and family first. A family plan, or a pilot
with a bank or telco, only after the baseline shows it beats doing nothing.
A sale changes the lists (commercial terms), the hosting and the legal
read, so each is listed as a gate, not designed now.

## ADR-015: The name (placeholder settled)

**Decision (Arthur, 2026-11-03):** the repo stays `scam-guard`; the shown
name is the placeholder **"Orang Baik AI"** until a final one is chosen.

**To weigh before it's final:** v1 has no model, so "AI" describes a
fallback that is off. And a name that sounds like a trusted helper is
what scammers copy, so the notice must say how to tell the real one.

**Rejected:** *"Cek Dulu"* (the recommendation).

## ADR-016: Security comes first

**Recommendation:** adopt Project 9's ADR-020 as it stands: security wins
over speed, features and cost; every threat has a control, a proof and a
slice ([threat model](threat-model.md)); no real person's data before the
real-data gate; one independent review before the gate.

## Decisions

| # | Decision | Rejected | Status |
|---|---|---|---|
| 001 | Arthur's family, up to 10, by invitation | Public; Arthur only | Recommended |
| 002 | WAHA with GOWS behind Project 9's adapter; alerts the one bot-started message | Meta's Cloud API (recommended), Telegram, Twilio, web form | **Settled 2026-11-03** |
| 003 | Depend on Project 9's channel, split out by its session | Copying it, a new channel | Recommended |
| 004 | Three levels, never "safe"; fixed wording pointing to the bank and IASC | Scores, two levels, "probably safe" | Recommended |
| 005 | Never visit links or open files; unknown is TINGGI | Sandboxes, HEAD requests, VirusTotal uploads | Recommended |
| 006 | URLhaus, MalwareBazaar, Phishing.Database, Safe Browsing update API; PhishTank and OpenPhish after spike 2 | VirusTotal, scraping cekrekening.id | Recommended |
| 007 | Model off in v1; fallback may only raise | Model first, model that lowers, local model | Recommended |
| 008 | Store no content; ratings as ids and hashes | Keeping messages | Recommended |
| 009 | Two-sided consent; alerts with no content; capped | Sharing content, SEDANG alerts, self-linking | Recommended |
| 010 | Measurement plan before any rule; private real set, written set kept apart | Hand-written or public set only | **Settled 2026-11-03** |
| 011 | Project 9's stack plus tldextract, rapidfuzz, PyYAML | A detection SaaS | **Settled 2026-11-03** |
| 012 | $0 until approved; model cap $1 a month | No cap | Recommended |
| 013 | Screenshots in slice 7; voice after a spike | Cloud OCR or speech | Recommended |
| 014 | Portfolio and family first | Designing for banks now | Recommended |
| 015 | Repo `scam-guard`; shown name "Orang Baik AI" (placeholder) | "Cek Dulu" | **Settled 2026-11-03** (placeholder) |
| 016 | Security first, as Project 9's ADR-020 | — | Recommended |

## The first slice

**Slices 0 and 1, $0:** collect and label the set, then the rules-only
baseline (B0) offline. No WhatsApp, no database, no model, no list
downloads. See the [slice plan](slice-plan.md).

## Open questions

**For Arthur, one at a time:**
1. ~~**Channel (ADR-002)**~~: settled 2026-11-03, WAHA.
2. ~~The real set~~: settled, private in `data/real/`, both SMS datasets privately.
3. ~~Second labeller~~: settled, a family member.
4. ~~Donations~~: settled, Arthur's own messages, scams he finds online, and
   messages Claude writes, kept apart.
5. ~~New libraries~~: settled, all three.
6. ~~Name~~: placeholder "Orang Baik AI".
7. **Model budget (ADR-012):** later, after the baseline.
8. **The other recommended ADRs** (001, 003–009, 012–014, 016): accept as
   recommended?
9. **WhatsApp's terms:** accept breaking them for real use, as Project 9's
   open question 1? Slices 0 to 5 are safe to build either way.

**For the spikes:** whether WAHA's GOWS payload carries a file's SHA-256;
abuse.ch list sizes and per-hash lookup; PhishTank, OpenPhish and Tranco
terms.

**For a lawyer, before any paid plan:** liability wording; UU PDP and
forwarded third-party data; WhatsApp's terms.

## Sources (read 2026-11-03)

**Primary:**
[URLhaus API and terms](https://urlhaus.abuse.ch/api/) ·
[MalwareBazaar FAQ](https://bazaar.abuse.ch/faq/) ·
[PhishTank developer info](https://phishtank.org/developer_info.php) ·
[Safe Browsing usage limits](https://developers.google.com/safe-browsing/v4/usage-limits) ·
[Phishing.Database](https://github.com/Phishing-Database/Phishing.Database) ·
[Bitdefender Scamio, Indonesia](https://www.bitdefender.com/en-id/consumer/scamio) ·
[OJK on IASC and the police](https://ojk.go.id/id/berita-dan-kegiatan/siaran-pers/Pages/OJK-dan-Bareskrim-Polri-Sepakat-Perkuat-Kolaborasi-Penanganan-Pengaduan-Pada-Indonesia-Anti-Scam-Centre.aspx) ·
[Komdigi on cekrekening.id](https://jdih.komdigi.go.id/infografis/view/57) ·
[SpamShield dataset](https://huggingface.co/datasets/M-Arjun/SpamShield-Datasets) ·
[nadyatyandra/IndonesianSMSSpamDetector](https://github.com/nadyatyandra/IndonesianSMSSpamDetector) ·
[ekaputra07/ina-sms-classifier](https://github.com/ekaputra07/ina-sms-classifier) ·
Project 9's Phase 5 (ADR-022) and threat model (WAHA, WhatsApp's terms)

**Secondary, used only where marked:**
[Kontan: IASC reports to September 2026](https://keuangan.kontan.co.id/news/iasc-terima-702342-laporan-kasus-penipuan-hingga-september-2026) ·
[Kontan: losses to January 2026](https://keuangan.kontan.co.id/news/hingga-14-januari-2026-kerugian-masyarakat-akibat-penipuan-scam-rp-91-triliun) ·
[Antara: report only at iasc.ojk.go.id](https://www.antaranews.com/berita/4743805/satgas-pasti-waspadai-penipuan-website-mengatasnamakan-iasc/) ·
[Bisnis: Tria Stealer wedding-invitation APK](https://teknologi.bisnis.com/read/20250201/84/1835932/terungkap-modus-malware-tria-stealer-curi-data-lewat-undangan-pernikahan-palsu) ·
[DJPb: invitation and parcel APK scams](https://djpb.kemenkeu.go.id/kppn/wates/id/data-publikasi/berita-terbaru/3069-phising-penipuan-undangan-apk-paket.html)
