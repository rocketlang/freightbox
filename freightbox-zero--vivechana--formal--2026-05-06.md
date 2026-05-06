# FreightBox Zero — VIVECHANA Decision Score

**Date:** 2026-05-06
**Formula:** V = K × F | K = I × L × R × Rep
**Schema:** SAR-006 | DOI: 10.5281/zenodo.19456053
**Scored against:** ANKR trajectory, current resources

---

## Axis Table (full definitions required — SAR-006)

| Axis | Score (1–10) | Definition | Reasoning for this score |
|---|---|---|---|
| **I — Impact** | 9 | Users/services/decisions directly affected. Observable facts only. | Every shipment a small NVOCC processes is a direct impact event. 2,000+ freight forwarders in India alone below the old break-even. Transforms the economics of running a freight company — observable, not speculative. |
| **L — Leverage** | 9 | New capabilities unlocked downstream. Bitmask=L=9, colour change=L=1. | Agentic Mode unlocks: Mode 2 licensing (50× revenue without 50× staff), Mode 3 managed service, the data flywheel (each shipment trains carrier/customer scoring), and Zenodo publication of the legal gate architecture as a protocol paper. High leverage — this is a bitmask-level unlock, not a colour change. |
| **R — Reach** | 8 | Propagation across ANKR universe — agents, partners, connected systems. | FreightBox Zero uses AnkrClaw, Aegis, GRANTHX, AnkrOS, EON memory, Forja SENSE events, AnkrCodex. Eight ANKR services deepen their utility when FreightBox Zero ships real shipments. Reach is high but not universe-wide — doesn't propagate to maritime SLM, compliance, or CRM directly. |
| **Rep — Replicability** | 7 | Auto-propagation without human intervention. Rep≤2=Jugaad. 10=works on service #226 with zero action. | The three-mode architecture (Man/Hybrid/Agentic) means every new NVOCC licensee onboards into an already-complete system. The autonomy dial, the Aegis scoring, the TRUST mask setup — these apply identically to licensee #2 as to licensee #1 without ANKR engineering intervention. Not Rep=10 because carrier API integrations require per-carrier configuration that is not fully automatic. |
| **F — Feasibility** | 9 | Buildable now with current resources? Only axis recomputed at scoring time. | FreightBox is live and working. AnkrOS AOS sessions tested today — returning real audit results. AnkrClaw live. Aegis running. The 90-day build plan has no unknown unknowns in Phase 1. F is 9 not 10 because FMC OTI license (60–90 days, outside ANKR control) blocks the first live shipment — but not the technology build. |

---

## Score Calculation

```
K = I × L × R × Rep
K = 9 × 9 × 8 × 7
K = 4,536

V = K × F
V = 4,536 × 9
V = 40,824
```

**V = 40,824 — NOW BUILD**

---

## Three-Option Comparison

| Option | I | L | R | Rep | K | F | V |
|---|---|---|---|---|---|---|---|
| **A. FreightBox Zero (all 3 modes)** | 9 | 9 | 8 | 7 | 4,536 | 9 | **40,824** |
| B. FreightBox as SaaS TMS only (no agentic) | 6 | 5 | 5 | 6 | 900 | 9 | 8,100 |
| C. Agentic mode only, no Man/Hybrid onramp | 8 | 9 | 8 | 7 | 4,032 | 7 | 28,224 |

Option A dominates. Option C's lower F (7 vs 9) reflects the risk of building only the most complex deployment mode — Man Mode and Hybrid Mode are the onramp that generates early revenue and real shipment data. Skipping them means the first customer has to trust full Agentic Mode from Day 1, which is a harder sell. Option B's Leverage collapses (L=5) because without the agentic layer, FreightBox is just another TMS with no downstream unlock.

---

## What Would Reduce the Score

- **L falls** if carrier APIs close to third-party access (contractually protected, industry moving opposite direction)
- **R falls** if AnkrOS trust architecture is not extended to support multi-tenant licensee sessions (engineering task, not a blocker)
- **Rep falls** if each licensee requires bespoke TRUST mask configuration rather than a templated onboarding (solvable with FBZ-001 session templating)
- **F falls** if FMC OTI license is denied (extremely rare for compliant applicants) or if India-first route is chosen and FMC is deferred (changes the corridor, not the score)

