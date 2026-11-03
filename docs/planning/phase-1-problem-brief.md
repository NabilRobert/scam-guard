# Phase 1: Problem Brief

**Project:** a WhatsApp scam guard for families in Indonesia. A person
forwards a suspicious message, link, screenshot or voice note. The guard
replies in plain Indonesian with a **risk level** and what to do next. If a
family member forwards something high-risk, a relative they linked is told.
It is the twelfth portfolio piece, built after Project 9 and on its
WhatsApp base.

The working name is the repo's, `scam-guard`. A better public name is an
open question (Phase 5, ADR-015).

**Measurement comes first.** Before any rule is written, the
[measurement plan](measurement-plan.md) fixes the labelled set, the
baseline and the numbers that decide whether this works.

## Problem

**For the users:**
- **Scams arrive on WhatsApp, from familiar-looking senders.** IASC, the
  Indonesia Anti-Scam Centre run by OJK, received 702,342 reports between
  November 2024 and September 2026, and counted Rp9.1 trillion in losses
  by January 2026.
- **The Indonesian ones are specific.** An APK named "Surat Undangan
  Pernikahan Digital" or "Lihat Foto Paket" steals SMS codes and banking
  logins (Kaspersky named one family Tria Stealer). Courier, prize,
  job-offer, e-tilang and "Mama minta pulsa" messages follow local
  patterns that a general checker may not know.
- **The person at risk is often not the one who would check.** Parents
  and grandparents get the messages. Their children would spot the scam,
  but they aren't asked in time.
- **"Is this safe?" has no honest yes.** A checker that says "safe" is
  believed, and is sometimes wrong. That is worse than no checker.

*To verify in the dogfood slice, not assumed:* that family members really
forward things before acting on them. If they don't, the family alert is
the only part that helps.

**For the portfolio:**
- Earlier projects defend systems from attackers (the gateway) or keep
  records exact (Project 9). None defends people, against an adversary who
  adapts to the defence.
- No project yet measures a classifier the way a buyer would ask: catch
  rate and false-alarm rate, on real messages, with the labels explained.
- It reuses Project 9's channel and privacy work, so it shows that the
  earlier pieces were built to be reused.

## Users

| Party | Who | Wants |
|---|---|---|
| **Protected person** | An adult, often older, who gets suspicious messages | A quick, clear answer in Indonesian, and what to do |
| **Linked relative** | A family member the protected person chose | To hear in time when something high-risk arrives, without reading their messages |
| **Arthur's family (first users)** | A few invited people (ADR-001) | The two roles above, in real life |
| **Reviewer / hiring team** | AI and tech people generally | Measured accuracy, honest limits, privacy and terms handled |
| **Later buyer** | A family plan, or a bank or telco | Proof it catches local scams without crying wolf (ADR-014) |

## Rivals

| Rival | What it is | Where this differs |
|---|---|---|
| **Bitdefender Scamio** | Free AI chatbot on WhatsApp, Messenger, Discord and the web. Bitdefender's Indonesian page lists it, in early access. Needs a Bitdefender account | Rules first and measured on Indonesian scams; a family alert; no account; never says "safe" |
| **Norton Genie** | Free AI scam checker from Gen Digital (app and web; WhatsApp not confirmed here) | Same as above |
| **cekrekening.id** (Komdigi) | Web form: has this bank or e-wallet account been reported? No public API found | Pointed to, not copied: the guard tells people to check there |
| **IASC** (OJK) | Where victims report, at iasc.ojk.go.id. It can block funds | Pointed to in every high-risk reply. The guard never claims to be IASC |
| **Bank WhatsApp channels** | Official help lines | Pointed to: "call the number on your card" |

**The angle:** Indonesian-specific scams (fake invitation and courier
APKs, local brands), the family alert, and numbers a reviewer can check.

## Goals and how they're measured

Targets are set **after** the baseline, not before (measurement plan).

| # | Goal | Target | Measured by |
|---|---|---|---|
| G1 | Catches the scams | Catch rate on the held-out real set, reported with its interval | Measurement plan |
| G2 | Doesn't cry wolf | False-alarm rate on the held-out real set, reported with its interval | Measurement plan |
| G3 | Never says "safe" | **0** replies with "aman" or a synonym as a verdict | A test over every reply template |
| G4 | Never runs what it's sent | **0** code paths that open, unpack or execute an attachment or visit a link | Tests and code review (threat model) |
| G5 | Keeps no one's messages | **0** message text, images or audio stored by default | A no-content test, as in Project 9 |
| G6 | The alert reaches the relative | Delivered within a minute in the test, and never carries the message text | Test on Meta's test number |
| G7 | Costs next to nothing | Rules and lists at $0; a model only within an approved budget | Monthly counts |

## Non-goals (v1)

- **Saying anything is safe.** Only risk levels (ADR-004).
- **Opening, unpacking, running or sandboxing attachments, or visiting
  links.** It checks hashes and addresses against lists (ADR-005).
- **Recovering money, blocking accounts or reporting for the person.**
  It points to the bank and IASC.
- **Reading a person's chats.** Only what they forward.
- **Groups.** Direct chats only.
- **Public sign-up** in v1 (ADR-001).
- **Voice notes** before the screenshot slice proves its worth.

## Decisions

None settled yet. Everything in Phase 5 is a recommendation until Arthur
agrees. The open questions are at the end of
[Phase 5](phase-5-architecture-decisions.md#open-questions).

## Cost of doing nothing

Nothing breaks. Rivals already exist and are free, so the project must win
on the Indonesian set and the family alert, or it isn't worth building. It
waits its turn behind Project 9's deploy.
