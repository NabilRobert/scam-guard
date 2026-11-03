# Measurement Plan

Written before any rule or code, so the numbers can't be shaped to fit
the rules. It answers four questions: what set, how it was labelled, what
baseline, and which numbers are reported.

## The labelled set

**Target:** about **200 real Indonesian scam messages** and **200 real
normal ones**. "Real" means sent to a real person, not written for the
set. Hand-written messages are kept apart (below) and never counted as
real.

**Settled with Arthur, 2026-11-03:** the real set stays private in
`data/real/`; the two unlicensed SMS datasets are used privately; a family
member gives the second label; and the set mixes messages Arthur received,
scams he finds online, and messages Claude writes, kept apart.

### Sources

Checked 2026-11-03. **No source found ships Indonesian scam messages under
an open licence.** That decides where the real set lives: in `data/real/`,
ignored by git, never committed. The repo commits the collection script,
the source register, the label guide and the results, with a fingerprint
of the set they came from.

| Source | Gives | Licence or terms | Use |
|---|---|---|---|
| **Warning pages from banks, OJK, IASC, Komdigi, police and couriers** | Scam messages quoted as examples, labelled scam by the publisher | Copyright of the publisher; short quotes of the scam text, which is not their work | **Scam, main source.** Private copy with the URL and date. Phone and account numbers redacted |
| **News articles** (Detik, Kompas, CNN Indonesia and others) quoting scam texts | Same | Same | Scam, second source |
| `nadyatyandra/IndonesianSMSSpamDetector` | Indonesian SMS labelled normal, fraud, promo | **No licence file** (GitHub API, 2026-11-03) | Private use only, never redistributed (settled). Fraud class as scam, normal class as normal. SMS, not WhatsApp: tagged by channel |
| `ekaputra07/ina-sms-classifier` | About 3,000 labelled SMS | **No licence file** | Same as above |
| BINUS SMS dataset published with its paper | Indonesian SMS | Not found | Only if a licence turns up |
| `M-Arjun/SpamShield-Datasets` (Hugging Face) | 149k messages in 23 languages, including Indonesian | CC BY 4.0 | **Not counted as real:** 20% is synthetic or translated, and Indonesian rows aren't marked. A secondary check at most |
| **Messages Arthur received** | Real scams, and real normal messages | Arthur's own; other people's numbers and names redacted before saving | Scam and normal |
| **Scams Arthur finds online** | Real scams quoted on warning pages, news, forums | As the warning-page row: URL and date kept | Scam |
| **Official messages from banks, couriers, e-commerce and government** | Real legitimate notifications: OTP notices, delivery updates, promos | From Arthur's own inbox | **Normal, the hard cases.** A checker that flags every courier message is useless |
| **Legitimate digital wedding invitations** (web links, common in Indonesia) | Real invitation messages that aren't scams | From Arthur's own inbox |
| **Messages Claude writes** | Scams and hard normal cases in the style of the real ones | Written for this repo | **Never counted as real, never in held-out.** Dev only, plus the public CI set; scored and reported apart | Normal, hard cases for the invitation rule |

**Normal messages must include hard cases.** At least 60 of the 200 are
legitimate bank, courier, e-commerce, invitation or government messages,
because those are what scams copy. The rest are ordinary family and
friends' messages with links.

**Privacy of the set.** Forwarded and donated messages hold other people's
data. Before a row is saved: phone numbers, account numbers, names other
than brands, and addresses are replaced by typed placeholders
(`<NOMOR_HP>`, `<REKENING>`, `<NAMA>`). The redaction script runs first and
a person checks each row. Images and audio are not part of this set.

### Label guide

Each row carries:

| Field | Meaning |
|---|---|
| `id` | Stable id |
| `text` | Redacted message text, with links kept |
| `label` | `scam` or `normal` |
| `scam_type` | One of: `apk_undangan`, `apk_kurir`, `apk_lain`, `phishing_bank`, `hadiah_undian`, `lowongan_kerja`, `investasi`, `pinjol`, `impersonasi_keluarga`, `belanja_online`, `tagihan_palsu` (PLN, BPJS, e-tilang), `lain` |
| `channel` | `whatsapp`, `sms`, `unknown` |
| `source_url`, `source_kind`, `licence` | From the register |
| `seen_on` | Date the message was sent or published |
| `source_label` | What the source said, before our labelling |
| `label_a`, `label_b` | The two independent labels |
| `notes` | Why, when it was hard |

### How the labels are made

1. **Source label.** A warning page or news article that calls a message a
   scam gives `source_label = scam`. A donor who knows the sender gives
   `normal`. A dataset's own label is copied.
2. **Two independent labels.** Arthur (`label_a`) and a family member
   who hasn't seen the sources (`label_b`) label each row from the text
   alone.
3. **Agreement.** Cohen's kappa between `label_a` and `label_b` is
   reported. A row where the source and both labellers disagree is
   **excluded and counted**, not forced.
4. **Unsure is a label.** A normal-looking message from an unknown sender,
   with no proof either way, is excluded. Only rows with a known answer
   are scored.
5. **No model labels anything.** It would cost money, and it would make
   the set agree with models.

### Splits

- **Dev, 60%:** the only rows looked at while writing rules.
- **Held-out, 40%:** scored **once per rules version**, as in the
  gateway's `score.py`: the result is written and a second run for the
  same version is refused.
- **By time:** the held-out set holds the newest messages, so drift shows.
  A scam type with fewer than 10 rows is reported but not given a rate.
- **Written set:** messages Claude writes, committed. About 40 of them
  are the public CI set. They may be used while writing rules, are
  reported apart, and are never reported as accuracy on real messages.

## The baseline: rules only, $0

Before any feed, model or channel, the rules are scored offline.

| Configuration | What runs | Cost |
|---|---|---|
| **B0: rules only** | Text patterns, link shape checks (lookalike brand domains, shorteners, raw IPs, punycode), file type from the message | $0, offline, in CI on the public set |
| **B1: rules + list snapshot** | B0 plus one dated snapshot of the free lists (ADR-006), stored locally | $0, local |
| **B2: + model fallback** | B1 plus a model on what the rules leave unsure, which may only raise the level (ADR-007) | Paid; only with Arthur's OK on a budget |

The snapshot is dated, because the lists change daily and a result must be
repeatable.

## What is reported

| Number | Definition |
|---|---|
| **Catch rate** | Scam rows rated **TINGGI**, over all scam rows. Rows rated **SEDANG** are reported beside it, not counted as caught |
| **False-alarm rate** | Normal rows rated **TINGGI**, over all normal rows. **SEDANG** on normal rows is reported beside it |
| **Per scam type** | Catch rate for each type with at least 10 rows |
| **Hard normal cases** | False-alarm rate on the bank, courier, invitation and government rows alone |
| **Unknown-as-high effect** | How many of the TINGGI ratings came only from "can't check this link or file" |
| **Intervals** | 95% Wilson intervals. With 80 held-out rows a side, they are roughly ±10 points, and the report says so |
| **Label agreement** | Cohen's kappa, and how many rows were excluded |

Each report names the rules version (a hash of the rules file), the list
snapshot date, the set fingerprint, the split and the counts. Results go
to `docs/accuracy.md`, generated from a committed run record.

## What this can't show

- **Most normal messages come from one inbox,** Arthur's. The hard cases
  match the banks and couriers he uses.
- **A set of reported scams is biased to scams people noticed.** The ones
  that worked quietly are missing.
- **SMS is not WhatsApp.** Rows from SMS datasets are tagged and reported
  apart.
- **400 messages is small.** It shows whether the approach works, not a
  production rate.
- **Screenshots and voice notes** aren't measured here. Each gets its own
  set in its slice.

## agent-eval-harness: where it fits

The harness grades agents calling MCP tools, so it doesn't fit the rules,
the same conclusion as Project 9's ADR-014. Its pieces that do fit:
`prices.yaml` and the budget guard for any paid model run, and its
run-record and provenance habits. Inspect AI, which the harness builds on,
runs the model eval in B2.
