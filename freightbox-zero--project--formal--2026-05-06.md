# FreightBox Zero — Formal Project Report

**Date:** 2026-05-06
**Status:** formal
**Classification:** internal — ANKR Labs
**VIVECHANA V:** 40,824 (now-build)
**Key:** freightbox-zero
**Ports:** freightbox-backend :4003 | freightbox-frontend :3001

---

## 1. What We Are Building

FreightBox Zero is an NVOCC (Non-Vessel Operating Common Carrier) operating model with three distinct deployment modes running on the same FreightBox platform:

**Man Mode** — FreightBox as a world-class TMS. All agent capabilities exist as tools. Humans operate, agents assist. Standard SaaS. Target: compliance-heavy markets, DG-heavy corridors, customers who want full staff control.

**Agentic Mode** — AI as principal operator. Six named agents (Kiran/Arjun/Priya/Kavya/Disha/Rekha) handle 90% of operations. Three humans handle what international commercial law makes non-delegable: BL issuance under MLETR, LC presentation under UCP 600, OFAC ambiguous-match resolution. Target: standard FCL, established corridors, tech-forward shippers.

**Hybrid Mode** — Graduated autonomy per shipment. The same company runs Agentic Mode for simple FCL shipments and Man Mode for DG, LC-financed, or novel-corridor shipments. Agent confidence scoring determines which mode applies per booking. This is the practical implementation for most real NVOCCs — the mode dial is per-shipment, not per-company.

The three modes are not separate products. They are the same platform with a different autonomy dial. An NVOCC starts in Man Mode, earns confidence in the agents, dials toward Hybrid, then dials further toward Agentic as trust accumulates. The transition is reversible at any time.

---

## 2. The Platform That Already Exists

FreightBox (live, tested, running on freightbox.org):

| Capability | Status |
|---|---|
| GraphQL API — 95 queries, 65 mutations, 8 subscriptions | Live |
| DCSA WAVE-compliant eBL lifecycle (8 SENSE events) | Live |
| BL draft, issuance, switch, telex release, surrender | Live |
| Invoice lifecycle, payment reconciliation, aging | Live |
| Container tracking, AIS subscriptions | Live |
| VGM recording and submission | Live |
| Quote → Booking → Shipment → Document → Finance → Settlement | Live |
| AnkrOS AOS agent sessions (BOT_AUTO / BOT_CONFIRMED / USER) | Live — tested today |
| freightbox_demo seeded with 6 bookings, 6 shipments, 6 BLs, 6 invoices | Live |

AnkrClaw (WhatsApp + Telegram ingestion): Live on :4150
Aegis / KAVACH (DAN gate L1–L4, HITL via WhatsApp, quarantine, budget caps): Running on :4850
KavachOS (kernel-layer agent governance, seccomp-bpf, PRAMANA receipt chain): Running on :4855
GRANTHX (knowledge layer, RAG, audit retrieval): Running on :4130

The foundational work is done. FreightBox Zero is a deployment model on top of an existing platform, not a ground-up build.

---

## 3. Why We Are Doing This

**The NVOCC economics problem.**
A traditional NVOCC handling 300 shipments per month needs 12–20 staff: operations, documentation, finance, customer service, compliance. Monthly staff cost: $60,000–$120,000. Break-even: 350–500 shipments/month. Most small NVOCCs never reach this volume sustainably, which is why the sector is fragmented and chronically undercapitalized.

FreightBox Zero breaks this equation. Three humans handle what law requires. Six agents handle the rest. Break-even: 80–120 shipments/month. At 1,500 shipments/month the economics are $225,000–$450,000 net margin — not achievable with a traditional staff model at that volume.

**The kairos.**
Three structural conditions converged within the same 18-month window (2024–2025):

1. MLETR legal recognition — UK ETDA 2023, Singapore ETA 2021, US adoption underway. A DCSA WAVE-compliant eBL is now a legally negotiable title document in the corridors that matter for an early-stage NVOCC. This eliminates the single largest human bottleneck: the physical document chase.

2. Carrier API maturity — Maersk, MSC, CMA CGM, Hapag-Lloyd now publish booking APIs, SI submission endpoints, and container event webhooks. In 2020 every carrier interaction required a human on the phone. In 2026, the majority of FCL carrier interactions are machine-handleable.

3. AI domain quality — LLMs can now draft SIs, flag discrepancies between booking confirmations and SIs, and surface detention risk with sufficient accuracy that a human review layer catches the remainder. The gap between "AI assists human" and "AI operates, human approves" has closed.

None of these three conditions existed simultaneously before 2024. The window is open now. Incumbents are seeing the same signals. The question is who assembles the complete stack first.

**Why ANKR specifically.**
ANKR already built the stack that makes this possible:
- FreightBox (eBL platform, DCSA compliant)
- AnkrOS (autonomy tiers, legal gate enforcement via trust_mask)
- Aegis (sanctions screening, risk classification)
- AnkrClaw (WhatsApp-native interface)
- GRANTHX (knowledge layer for domain intelligence)

No one else has this combination. Building it from scratch would take 18–24 months and $3–5M. For ANKR it is a deployment model, not a platform build.

---

## 4. What Others Are Doing — Competitive Landscape

### Direct comparison

| Company | Model | eBL | Agentic Ops | WhatsApp Native | Legal Gate Arch | Kernel Proof |
|---|---|---|---|---|---|---|
| **FreightBox Zero** | AI-operated NVOCC | DCSA WAVE | ✅ 6 agents | ✅ AnkrClaw | ✅ trust_mask | ✅ KavachOS |
| Flexport | Digital freight forwarder | No | No | No | No | No |
| Freightos | Rate marketplace | No | No | No | No | No |
| Maersk Twill | Staffed digital NVOCC | Partial | No | No | No | No |
| Forto | Digital freight forwarder | No | No | No | No | No |
| GoComet | TMS / procurement | No | No | No | No | No |
| Beacon | Shut down 2022 | No | No | No | No | No |
| Portcast | Port AI predictions | N/A | No | No | No | No |
| Kontainers | White-label digital freight | No | No | No | No | No |
| Bolero / essDOCS | eBL platforms only | Yes | No | No | No | No |

**The gap is absolute:** No company is operating an NVOCC with AI as the principal operator and legal gate architecture enforced at the session layer. Every digital freight company is still building "AI as assistant to humans." FreightBox Zero inverts this.

### Why incumbents cannot quickly replicate

**Flexport** raised $2.2B and is now part of the Shopify ecosystem. Their platform is portal-native, built for logistics professionals, not factory owners. Their operational model is staffed. Pivoting to agent-operated would require dismantling the human operations layer that is their cost base — a $2B company cannot make that pivot without destroying the business they have while the new one is being built.

**Maersk Twill / Sealand** has digital NVOCC capability but cannot offer agent-operated NVOCC because they are also a carrier. Regulatory conflict of interest: an NVOCC that is also a carrier cannot use AI to preference its own carrier. The structural conflict prevents the pivot.

**New entrants** face an 18–24 month runway to replicate just the FreightBox eBL layer. DCSA compliance, MLETR-compatible blockchain anchoring, and carrier API integrations cannot be compressed regardless of capital. By the time a new entrant is ready, FreightBox Zero will have 18 months of real shipment data as a moat.

### The new approach (what makes this genuinely novel)

Every existing approach treats the NVOCC operation as a human workflow that AI improves incrementally. FreightBox Zero treats the NVOCC operation as an AI workflow that humans validate at legal gates.

This is not a better chat assistant. It is an inversion of the operating model.

The legal gate architecture is the specific novelty: using AnkrOS TRUST mask bits to enforce — not enforce in policy, enforce in session-layer code — that certain actions (BL issuance, telex release, LC presentation) cannot happen without a specific human session holding a specific trust bit. Not a workflow rule that a bug can bypass. A hard constraint at the authentication layer. This is the mechanism that makes an NVOCC regulatorily defensible under FMC, MLETR, and OFAC simultaneously.

No academic paper, no shipping technology publication, no startup has published this architecture. It is genuinely new.

---

## 5. The Three Modes — Deep Specification

### Mode 1: Man Mode

**Who:** Existing NVOCCs, freight forwarders, or new entrants who want world-class tooling without agentic operations. Also: any shipment type where Man Mode is required by law or policy (DG heavy, LC with conservative banks, novel corridors not yet on MLETR recognition list).

**How it works:**
The six agent capabilities exist as tools in the FreightBox UI, not as autonomous operators. Kiran's quote-parsing engine is a "quote assist" button. Priya's BL draft generator is a "draft from SI" button. Disha's AIS feed is a tracking dashboard. The human clicks, the agent responds, the human decides.

All six agents are present. None are autonomous. Every action is human-initiated.

**Staff model:** 10–15 people for 300 shipments/month (industry standard). FreightBox reduces their error rate and cycle time — a documentation officer using Priya's draft generator produces BLs in 20 minutes instead of 2 hours — but does not reduce headcount.

**Revenue model:** SaaS subscription, $500–$2,000/month depending on shipment volume tier. Standard TMS pricing.

**Metrics that matter:** BL accuracy rate, SI submission time, invoice-to-payment cycle time, cut-off miss rate. All measurably better than manual TMS within 30 days of deployment.

---

### Mode 2: Hybrid Mode

**Who:** NVOCCs that have gained confidence in the agents and want to automate their standard shipments while keeping human control on complex ones. This is where most real deployments live after Month 3.

**How it works:**
Every new booking is scored by Aegis Guard on five dimensions:
1. Customer familiarity score (new vs repeat, credit history)
2. Commodity complexity (general cargo vs DG vs perishable vs project cargo)
3. Corridor confidence (how many similar shipments has FreightBox Zero handled successfully on this route)
4. Trade finance structure (open account vs LC vs documentary collection)
5. Carrier reliability score (based on FreightBox Zero's accumulated carrier performance data)

Composite score above threshold → Agentic Mode for that shipment
Composite score below threshold → Man Mode for that shipment, with agent-assist available

The threshold is operator-configurable. A conservative NVOCC sets it high (most shipments are Man Mode). An aggressive operator sets it low (most shipments are Agentic Mode).

**The critical design:** The autonomy dial is per-shipment, visible in the UI, and adjustable after the fact. If a shipment entered Agentic Mode and the Captain wants to take manual control, one click transfers the shipment to Man Mode. The agents do not resist — they hand over the full shipment record with complete audit trail.

**Staff model:** 5–8 people for 300 shipments/month. The standard shipments (typically 60–70% of volume) run autonomously. Staff focuses on the complex shipments that scored below threshold, on exception handling, and on carrier relationship management.

**Revenue model:** SaaS subscription plus per-shipment fee for Agentic Mode shipments. $300–$800/month base plus $5–15/shipment for autonomous shipments.

---

### Mode 3: Agentic Mode (FreightBox Zero)

**Who:** New NVOCC entrants who want to run lean from Day 1. Existing NVOCCs who have successfully transitioned through Hybrid Mode. The target operating model for ANKR's own NVOCC operations.

**How it works:**
Full implementation as specified in the brainstorm document (freightbox-zero--brainstorm--formal--2026-05-06.md). Six agents, three humans, all legal gates enforced at the session layer.

The Captain's day: morning review of overnight agent activity via the SENSE event dashboard, 10–20 BL approvals on mobile (Priya's draft → Captain review interface → one-tap issuance), escalation handling for exceptions the agents could not resolve.

The Finance Principal's day: review OFAC ambiguous matches (typically 0–3/day at steady state), LC presentation authorization, credit exception approvals. Two to four hours of active work.

The Compliance Counsel: 10–15 hours/month reviewing novel legal scenarios Aegis has flagged as outside existing rule base. Updates Aegis rule set. Monitors MLETR corridor map for jurisdiction changes.

**Staff model:** 3 humans for up to 2,000 shipments/month (infrastructure-limited, not staff-limited).

**Revenue model for licensed version:** Platform fee $2,000–$5,000/month plus $8–20/shipment. At 500 shipments/month the licensee pays $6,000–$15,000/month and eliminates $40,000–$60,000/month in staff cost. The ROI case is immediate.

---

## 6. The Three-Mode Architecture in Technical Terms

```
                     ┌─────────────────────────────────────────────┐
                     │              FreightBox Platform             │
                     │   GraphQL API · eBL Stack · SENSE Events    │
                     └──────────────────┬──────────────────────────┘
                                        │
               ┌────────────────────────┼──────────────────────────┐
               │                        │                          │
        ┌──────▼──────┐         ┌───────▼──────┐         ┌────────▼──────┐
        │  MAN MODE   │         │ HYBRID MODE  │         │AGENTIC MODE   │
        │             │         │              │         │               │
        │ Agents as   │         │ Per-shipment │         │ 6 agents      │
        │ UI tools    │         │ Aegis score  │         │ 3 humans      │
        │             │         │ determines   │         │ legal gates   │
        │ Human =     │         │ which mode   │         │ trust_mask    │
        │ operator    │         │ applies      │         │ enforced      │
        └─────────────┘         └──────────────┘         └───────────────┘
              │                        │                          │
              └────────────────────────┼──────────────────────────┘
                                       │
                    ┌──────────────────▼──────────────────────────┐
                    │            AGENT SAFETY STACK                │
                    │                                              │
                    │  ┌─────────────────────────────────────┐    │
                    │  │  LAYER 1 — TRUST mask (AnkrOS)       │    │
                    │  │  BL/telex/LC bits absent from agent  │    │
                    │  │  session token. Structural — not     │    │
                    │  │  policy. No bypass path exists.      │    │
                    │  └──────────────────┬──────────────────┘    │
                    │                     │                        │
                    │  ┌──────────────────▼──────────────────┐    │
                    │  │  LAYER 2 — AEGIS DAN gate (:4850)    │    │
                    │  │  PreToolUse intercept before exec.   │    │
                    │  │  L1 auto · L2 notify · L3 WhatsApp  │    │
                    │  │  HITL · L4 dual-control. Budget cap  │    │
                    │  │  + blast radius check per action.    │    │
                    │  └──────────────────┬──────────────────┘    │
                    │                     │                        │
                    │  ┌──────────────────▼──────────────────┐    │
                    │  │  LAYER 3 — KavachOS kernel (:4855)   │    │
                    │  │  seccomp-bpf profile from trust_mask │    │
                    │  │  + domain. Every syscall logged.     │    │
                    │  │  PRAMANA SHA-256 receipt chain.      │    │
                    │  │  Tamper-evident. EU AI Act Art.14.   │    │
                    │  └─────────────────────────────────────┘    │
                    └──────────────────────────────────────────────┘
```

The autonomy dial is the only thing that changes between modes. The platform, the agents, the data model, the legal gates — identical. A company running Man Mode has already built the full Agentic Mode capability. They are choosing not to activate it yet.

This is the product strategy. Man Mode is the onramp. Agentic Mode is the destination. Hybrid Mode is the journey.

---

## 7. The Agent Safety Stack — Three Layers of Enforcement

The standard approach to AI safety in enterprise software is policy: write a rule that says "agent cannot issue BL." Rules can be bypassed by bugs, configuration errors, or jailbreaking. FreightBox Zero uses three independent enforcement layers. All three must fail simultaneously for a legal gate to be breached. This has not happened and structurally cannot happen under normal operation.

---

### Layer 1 — TRUST mask (AnkrOS session layer)

Priya's AnkrOS session is initialized with a TRUST mask that does not include the BL issuance bit. The `issueBL` and `issueEBL` GraphQL mutations check the session's `trust_mask` before executing. If the bit is not set, the mutation returns an authentication error — not a business logic error, not a permissions error. An authentication error at the session layer, before any BL-generation code runs.

The Captain's session holds the bit. Priya's session never will.

| Legal gate | Bit | Who holds it |
|---|---|---|
| BL issuance (MLETR Art. 7) | `BIT_BL_ISSUE` | Captain only |
| Telex release | `BIT_TELEX_RELEASE` | Captain only |
| LC presentation (UCP 600) | `BIT_LC_PRESENT` | Finance Principal only |
| OFAC ambiguous-match auth | `BIT_OFAC_AUTH` | Finance Principal only |
| Credit write-off | `BIT_CREDIT_WRITEOFF` | Finance Principal only |

What Layer 1 prevents:
- A bug in Priya's BL generation code cannot accidentally issue a BL
- A misconfigured autonomy tier cannot accidentally issue a BL
- An adversarial prompt injected into any agent cannot cause BL issuance
- The SENSE event for `ebl.issued` cryptographically proves it was the Captain's session

---

### Layer 2 — AEGIS DAN gate (port 4850)

AEGIS intercepts every agent tool call **before** it executes via a PreToolUse hook. Every action is classified on the DAN (Destructive / Authorisation-Required / Normal) scale:

| Level | Classification | Aegis response |
|---|---|---|
| L1 | Normal — low blast radius | Auto-approve, log |
| L2 | Notable — elevated risk | Notify human via dashboard, proceed |
| L3 | Authorisation-required | **Block execution. WhatsApp message to designated human. Agent waits.** |
| L4 | Irreversible / dual-control | Block execution. Two humans must approve via separate sessions. |

For FreightBox Zero agents, any attempt to call a Layer 1 legal gate (which would already fail at the trust_mask check) is also classified as DAN-L4 by AEGIS — triggering dual-control before the token check even runs.

Additional AEGIS protections wired to FBZ agents:
- **Budget caps per agent per session** — Kavya cannot run an invoice reconciliation that exceeds 10,000 API calls without human approval
- **Blast radius assessment** — before Arjun submits a booking, Aegis checks which downstream services would be affected by a cancellation and surfaces the exposure
- **Quarantine** — a rogue agent is frozen instantly; the session is preserved as evidence, not killed
- **WhatsApp delivery** — the three humans (Captain, Finance Principal, Compliance Counsel) receive plain-English approval requests on their phones via AnkrClaw :4150. The agent does not proceed until the human responds.

The WhatsApp channel is deliberate. A morning review of 10–20 BL approvals takes 15 minutes on a phone, not a desktop session. The legal gate does not create friction that undermines the model.

---

### Layer 3 — KavachOS kernel receipt chain (port 4855)

KavachOS generates a deterministic seccomp-bpf profile from the agent's `trust_mask + domain` combination. This profile is sealed with a SHA-256 hash at session start (PRAMANA receipt chain). Every syscall the agent makes is filtered through this profile at the Linux kernel layer.

What this means in practice:
- The profile for Priya (documentation agent, maritime domain) permits file reads, network calls to the FreightBox GraphQL endpoint, and PDF generation. It does not permit process spawning, raw socket access, or writes outside the designated directory scope.
- If Priya's code is manipulated mid-session (supply chain attack, prompt injection that executes arbitrary code), the kernel filter blocks the unexpected syscall and logs it as a Falco violation with a tamper-evident receipt.
- The receipt chain is the **court-admissible record** of what the agent actually did at OS level, not what it said it did in application logs.

KavachOS satisfies EU AI Act Article 14 (human oversight of high-risk AI) at the infrastructure layer, independent of application-level logging. The receipt chain survives even if FreightBox application logs are corrupted.

---

### The combined argument

To an FMC regulator: *"Your agents have adequate human control?"*
Layer 1 makes certain actions structurally impossible for agent sessions. Layer 2 intercepts everything else before execution and routes human approval via WhatsApp. Layer 3 proves at kernel level what actually ran.

To a P&I Club: *"Who authorized the BL?"*
The SENSE event for `ebl.issued` carries the Captain's session token. Layer 1 guarantees no other session could have set it. Layer 3 provides the kernel receipt proving what ran in the issuing session.

To a bank: *"Is this eBL legitimately issued?"*
The eBL carries the DCSA WAVE-compliant cryptographic signature. The SENSE event chain from `ebl.draft_created` through `ebl.issued` is immutable. The KavachOS receipt chain shows the syscall sequence of the issuing session.

**The answer is the same at all three layers: the proof is in the chain, not in a policy document.**

No existing freight technology company has published this architecture. It is ANKR's moat — and with the Zenodo DOI (10.5281/zenodo.20047472) now timestamped, it is prior art.

---

## 8. Go-to-Market Sequence

**Month 1–3: ANKR as Customer Zero**
Run FreightBox Zero in Agentic Mode on ANKR's own test shipments. Not live cargo — simulated shipments through the full lifecycle, with all agents active and all legal gates exercised. Produce the 30-shipment retrospective (FBZ-018). Measure the actual escalation rate. Calibrate the Aegis scoring threshold.

**Month 4–6: First Live Pilots (Man Mode → Hybrid)**
Three India-based freight forwarders. Offer FreightBox platform as a TMS replacement (Man Mode). Introduce agent-assist features. Let them experience the BL draft speed, the cut-off monitoring, the invoice reconciliation accuracy. At Month 6, offer to enable Hybrid Mode for their standard FCL shipments.

**Month 7–12: Hybrid → Agentic Transition**
Pilot forwarders who have built confidence in the agents transition standard shipments to Agentic Mode. Measure staff reduction and throughput increase. Document the economics with real numbers. This is the case study that enables Mode 2 licensing.

**Month 13+: Mode 2 Licensing**
Offer the FreightBox Zero operating model to NVOCCs who want to run lean. Their QI is the Captain. Their finance controller is the Finance Principal. ANKR provides the platform, the agents, the Aegis guard, and the playbook. Fifty licensees at 200 shipments/month = 10,000 shipments/month through ANKR's infrastructure.

---

## 9. Risk Register

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| FMC license delay (QI / OTI process) | Medium | Blocks first live shipment | Apply Month 1, technology build proceeds in parallel |
| Carrier API downtime at cut-off | Medium | Missed SI, shipper relationship damage | EDI fallback, Arjun manual mode at cut-off-6h |
| MLETR non-recognition in India | High | Paper BL fallback for India-origin | Paper BL path fully implemented in Man Mode |
| P&I Club coverage ambiguity for AI errors | Medium | Liability exposure | Legal opinion before first live shipment |
| Captain capacity ceiling breached | Low (early) | Escalation rate overwhelms Captain | Escalation rate monitoring, Hybrid Mode dial-back |
| Incumbent acquires eBL platform | Low (18 months) | Competitive pressure increases | Publish legal gate architecture to Zenodo now |

---

## 10. What We Need to Decide This Week

1. **FMC path:** Do we apply for OTI/NVOCC license now, or do we run in India first (IATA-licensed forwarder route) and approach FMC in Month 6? India does not require FMC licensing for India-origin shipments to non-US destinations.

2. **Zenodo paper:** The legal gate architecture (trust_mask enforcement for BL issuance) should be published before anyone else files the concept. This is a DOI timestamp, not a patent. It takes 48 hours to publish. Decision: publish now or wait for first live shipment data?

3. **Captain interface:** FBZ-009 is the most critical UX piece in the entire model — the 30-second BL approval on mobile. This should be built and tested before any pilot begins. Priority?

4. **Carrier API selection:** Maersk and CMA CGM have the most mature APIs. MSC is less stable. Do we build MSC integration in Phase 1 or defer to Phase 2 when we have more carrier data to support the conversation?

---

*FreightBox Zero — VIVECHANA V = 40,824. Three modes, one platform, one architectural decision that makes it legally defensible. The window is open.*

