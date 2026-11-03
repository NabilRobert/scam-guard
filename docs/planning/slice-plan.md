# Slice Plan and Cost Model

## Prerequisites (outside this repo)

| # | Prerequisite | Needed for | Who |
|---|---|---|---|
| P1 | Arthur OKs Phases 1, 2 and 5 and the measurement plan | Slice 0 | Arthur |
| P2 | Answers to the open questions on the real set, the second labeller and donations | Slice 0 | Arthur |
| P3 | The GitHub repo exists | Pushing anything | Arthur |
| P4 | Free keys: abuse.ch Auth-Key, Google Safe Browsing | Slice 2 | Arthur |
| P5 | Project 9's channel split into a package, by Project 9's session | Slice 3 | Project 9 |
| P6 | A Meta developer app with the test number | Slice 3 | Arthur |
| P7 | Project 9 deployed; the server ready for this data | Slice 6 | Arthur |
| P8 | A dedicated SIM and Meta's business approval | Slice 6 | Arthur |
| P9 | A model budget and the provider's terms read | Slice 8 | Arthur |

## Spikes (local, $0)

| # | Question | Method | Pass | Time |
|---|---|---|---|---|
| 1 | Meta's current test-number limits, prices, and the alert template's category | Read Meta's own pages and the dashboard | Recorded in ADR-002 and ADR-012 | 1 h |
| 2 | List sizes, per-hash lookup, and PhishTank, OpenPhish and Tranco terms | Read the docs; download nothing large without Arthur's OK | Recorded in ADR-006 | 1 h |
| 3 | How many scams in the set arrive only as an image or a file? | Count in the set | Decides whether slice 7 moves earlier | 30 min |

## Slices

**Evaluation first, as in Project 9:** tests and scorer come before the
rules. Each new test is mutation-checked once. Each slice updates
`docs/guide.md`.

| Slice | Delivers | Done when | Cost |
|---|---|---|---|
| **0. The set** | Repo scaffold (README, CI, complexity rules); the source register; the collection and redaction scripts; the label guide; the public paraphrased set | About 200 scam and 200 normal real rows labelled twice, kappa reported, held-out split frozen and fingerprinted | $0 |
| **1. Rules baseline (B0)** | Text rules (YAML); link shape checks; file-type rules; the combining rule; the scorer; `docs/accuracy.md` | Held-out scored once; catch and false-alarm rates with intervals published; G3 and G4 tests pass | $0 |
| **2. Lists (B1)** | Fetch job and local cache for URLhaus, MalwareBazaar, Phishing.Database, Safe Browsing update API; stale-cache fail-closed; file hashing without parsing | B1 scored on the same held-out set with a dated snapshot; no lookup touches the network per message | $0 |
| **3. Channel** | Project 9's channel as a dependency; Meta Cloud API on the test number; consent; reply wording; `bantuan`; `hapus data saya` | Signed fixtures cover every path; one live round trip on the test number with invented messages; no-content tests pass | $0 |
| **4. Family alert** | Two-sided linking; alert template; caps; re-confirmation; `berhenti` | Alert tests in the threat model pass; one live alert on the test number | $0 |
| **5. Privacy and operations** | Rating purge; spans; `/healthz` with list ages; threat-model proofs | Every S-row for slices 0 to 5 is Done | $0 |
| **6. Deploy, gate, dogfood** | Container; probes through systems-monitoring; the real-data gate; Arthur's family for 4 weeks | Gate passes; misses become dev rows (never held-out) | Meta replies and alerts, about Rp10,000 a month; SIM |
| **7. Screenshots (Should)** | OCR in a capped, offline process; an image set | Image set scored; memory measured under the cap | $0 |
| **8. Model fallback (Could)** | Redaction; typed output; may-only-raise; Inspect task through the harness's budget guard | B2 scored; no output lowered a level; Arthur's OK for each paid run | Under the approved cap |
| **9. Voice notes (Could)** | Only if a spike shows whisper.cpp fits | — | $0 to spike |
| **10. Evidence and write-up** | Results from run records; JOURNEY; portfolio entry with the portfolio session | Every number traces to a file; privacy check passes | $0 |

## The first slice, in detail

**Slices 0 and 1 together, $0, local, no network per message.**

Slice 0 needs Arthur's answers on the real set before collection starts.
Until then, only the scaffold, the scripts, the label guide and the public
paraphrased set can be built.

Slice 1 is done when:
- `uv run pytest` passes, including V1 (no "safe" wording) and K4 (no
  visit, no open) as tests;
- the scorer runs on the public set in CI;
- the held-out real set is scored once for the first rules version, and
  `docs/accuracy.md` shows catch rate, false-alarm rate, hard normal
  cases, per scam type, the unknown-as-high effect and intervals, with
  the set's fingerprint;
- ruff (with the complexity rules), pyright and gitleaks pass in CI.

## Security done-when for each slice

| Slice | Security done-when |
|---|---|
| 0 | `data/real/` is ignored and gitleaks passes; every saved row is redacted and checked by a person |
| 1 | No code path fetches a URL or opens a file (test); no "safe" wording (test) |
| 2 | No per-message network lookup except Safe Browsing prefixes; a stale cache fails closed |
| 3 | Project 9's channel tests re-run green here; no content in logs or spans |
| 4 | No alert carries content; links need both sides |
| 5 | Every S-row for slices 0 to 5 is Done |
| 6 | The real-data gate, as Project 9's |
| 7 | OCR runs with no network and a cap; images are never stored |
| 8 | No model output lowers a level; redaction tested |
