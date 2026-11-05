# scam-guard

A WhatsApp scam guard for Indonesian families (working name "Orang Baik
AI"). A person forwards a suspicious message, link or file and gets back
a risk level in plain Indonesian, with what to do next. A relative they
have linked is alerted when something high-risk comes in. It never calls
anything "safe". It never visits links or opens files: it checks them
against public threat lists and treats anything unknown as high risk.
Every high-risk reply points to the person's bank and to IASC.

## Status

**Planned, paused on 2026-11-05.** Nothing is built yet. The project waits
on the WhatsApp base from Project 9, the finance tracker: its channel code
will become a package this repo depends on, so the two don't each carry a
copy.

The first slice costs $0 and runs offline:

1. Collect and label about 200 real Indonesian scam messages and 200
   normal ones. The real messages stay private and are never committed.
2. Score a rules-only baseline on a held-out set, reporting the catch
   rate and false-alarm rate with 95% intervals.

## Layout

| Path | What it holds |
|---|---|
| [`docs/planning/phase-1-problem-brief.md`](docs/planning/phase-1-problem-brief.md) | The problem, users, rivals, goals and non-goals |
| [`docs/planning/phase-2-requirements-constraints.md`](docs/planning/phase-2-requirements-constraints.md) | Requirements, constraints and the design diagram |
| [`docs/planning/phase-5-architecture-decisions.md`](docs/planning/phase-5-architecture-decisions.md) | Decisions (ADRs), what was reused and the open questions |
| [`docs/planning/measurement-plan.md`](docs/planning/measurement-plan.md) | The labelled set, the baseline and the numbers reported |
| [`docs/planning/threat-model.md`](docs/planning/threat-model.md) | Threats and their controls |
| [`docs/planning/slice-plan.md`](docs/planning/slice-plan.md) | Prerequisites, spikes and the slices in order |

There is no code yet, so there is no CI workflow and no `docs/guide.md`.
Both arrive with the first slice.
