# FreightBox Zero: Legal Gate Architecture for AI-Operated Regulated Entities

**Authors:** Anil Kumar Sharma, ANKR / PowerBox IT Solutions Pvt Ltd
**Date:** 2026-05-06 (v3: 2026-05-06 — AEGIS expanded to Hard Gate + Five Locks + Quality + Frontend)
**Classification:** Public — technical note
**Supersedes:** DOI 10.5281/zenodo.20047601 (v2 — DAN gate only) · DOI 10.5281/zenodo.20047472 (v1 — session-layer only)

---

## Abstract

We describe FreightBox Zero, an operational model for a Non-Vessel Operating Common Carrier (NVOCC) in which six AI agents handle 90% of freight operations and three humans handle exclusively what international commercial law makes non-delegable. The central contribution is the **three-layer agent safety stack**: independent enforcement of human-mandatory actions across three layers that must all fail simultaneously for a legal gate to be breached. Layer 1 uses TRUST mask bits in the AnkrOS session system to make certain actions — bill of lading issuance under MLETR Article 7, letter of credit presentation under UCP 600, OFAC sanctions adjudication — structurally impossible for any AI agent session to execute, independent of application logic. Layer 2 uses the AEGIS Hard Gate system (port 4850) — a tiered capability enforcement engine validated over 104 batch soak runs with zero false positives across ~9,800 checks — to intercept every agent tool call before execution. Hard Gate tiers HG-1 through HG-2C classify capabilities by authority class. The `@ankr/aegis-guard` SDK exposes Five Locks cryptographic primitives (approval token mint/verify/consume, idempotency fingerprint, `IrrNoApprovalError`) that FreightBox agents import directly. Actions at L3/L4 on the DAN (Destructive/Authorization-Required/Normal) scale are blocked and routed to the Captain or Finance Principal via WhatsApp; the agent waits for a cryptographically-scoped approval token before proceeding. Layer 3 uses KavachOS (port 4855) to enforce a deterministic seccomp-bpf kernel profile per agent session — derived from the agent's trust_mask and domain — and seals every syscall in a PRAMANA SHA-256 tamper-evident receipt chain. The combination produces court-admissible, kernel-level evidence of what each agent actually did, independent of application logs. We further introduce three operating modes — Man Mode, Hybrid Mode, and Agentic Mode — on the same platform without architectural change. VIVECHANA score: V = 40,824 (I=9, L=9, R=8, Rep=7, K=4,536, F=9).

---

## 1. Introduction

Every major digital freight platform built between 2016 and 2024 operates on the same model: humans make decisions, AI improves the quality and speed of those decisions. Flexport, Freightos, Forto, and Maersk Twill are all "AI-assisted human operations." This model has a hard economic floor: the staff cost of operating an NVOCC cannot fall below the number of humans required to make regulatory-compliant decisions at scale.

FreightBox Zero inverts this model. AI is the principal operator. Humans are exception handlers at legal gates. The operational question shifts from "how do we help humans work faster?" to "where does law require a human, and how do we enforce that boundary architecturally?"

Three convergent conditions in 2024–2025 make this model viable for the first time:

1. **MLETR legal recognition** — The UK Electronic Trade Documents Act 2023 and Singapore Electronic Transactions Act 2021 establish eBL as a legally negotiable title document. The physical document bottleneck — the primary reason NVOCCs required documentation staff — is removable.

2. **Carrier API maturity** — Maersk, MSC, CMA CGM, and Hapag-Lloyd publish booking, SI submission, and container event APIs. The majority of FCL carrier interactions are now machine-handleable.

3. **AI domain sufficiency** — Large language models trained on maritime domain data can draft shipping instructions, detect SI-to-booking discrepancies, and surface detention risk with sufficient accuracy that a structured review layer produces legally defensible outputs.

The question this paper addresses: given that AI can handle most NVOCC operations, what is the correct architectural mechanism to ensure that the operations law requires humans to perform are truly performed by humans, not by AI bypassing a policy check?

---

## 2. The Three-Layer Agent Safety Stack

The naive approach to human-mandatory actions in an AI system is a single policy rule. Policy rules share a structural weakness: they are application logic, and application logic can be subverted. FreightBox Zero uses three independent enforcement layers. All three must fail simultaneously for a legal gate to be breached. Under normal operation, this is structurally impossible.

```
  ┌────────────────────────────────────────────────────────┐
  │  LAYER 1 — TRUST mask (AnkrOS session)                 │
  │  BL/telex/LC/OFAC bits absent from agent token.        │
  │  Structural impossibility — no bypass path in code.    │
  └─────────────────────────┬──────────────────────────────┘
                            │
  ┌─────────────────────────▼──────────────────────────────┐
  │  LAYER 2 — AEGIS Hard Gate (port 4850)                  │
  │  HG-1/2A/2B/2C capability enforcement. Five Locks SDK. │
  │  L3/L4 → block + approval token + WhatsApp HITL.       │
  └─────────────────────────┬──────────────────────────────┘
                            │
  ┌─────────────────────────▼──────────────────────────────┐
  │  LAYER 3 — KavachOS kernel receipt (port 4855)         │
  │  seccomp-bpf profile from trust_mask + domain.         │
  │  SHA-256 receipt chain. Tamper-evident. EU AI Act.     │
  └────────────────────────────────────────────────────────┘
```

### 2.1 Policy-Based vs Session-Layer Enforcement

The naive approach to human-mandatory actions in an AI system is a policy rule: "the system checks whether this action requires human approval before executing." This approach has three failure modes:

**Bug bypass:** A code defect in the approval check function allows the action to proceed without human review.

**Configuration bypass:** A misconfigured autonomy tier or role permission causes the system to evaluate the action as not requiring approval.

**Prompt injection bypass:** An adversarial input to an upstream agent causes downstream state that the approval check evaluates as already-approved.

All three failure modes share a common structure: the policy check is application logic, and application logic can be subverted.

The session-layer approach eliminates all three failure modes by moving enforcement below the application layer. In FreightBox Zero, the `issueBL` and `issueEBL` GraphQL mutations check the executing session's `trust_mask` against the required bit before any business logic executes. The trust_mask is set at session initialization and is not modifiable by application code during the session lifetime. An AI agent session is initialized without the BL issuance bit. A human Captain session is initialized with it.

```
Session initialization (Captain):
  trust_mask = 0b...1 (BL issuance bit set)

Session initialization (DOCUMENT AGENT / Priya):
  trust_mask = 0b...0 (BL issuance bit not set)

issueBL mutation preHandler:
  if (session.trust_mask & BIT_BL_ISSUE) === 0:
    throw AuthorizationError("BL issuance requires Captain session")
    // This is an authentication error, not a business logic error
```

The consequence: a bug in Priya's BL generation code cannot accidentally issue a BL. A misconfigured autonomy tier cannot issue a BL. A prompt injection to Kiran or Arjun that propagates through the agent chain cannot issue a BL. The BL issuance bit is not in any agent's trust_mask, and no application code can put it there.

### 2.2 The Five Legal Gates

FreightBox Zero identifies five actions that require session-layer enforcement:

| Action | Law | Required Session | TRUST bit |
|---|---|---|---|
| eBL issuance | MLETR Article 7 — reliable system requires identified legal person | Captain | BIT_BL_ISSUE |
| Telex release | Carrier practice — NVOCC liability on original surrender confirmation | Captain | BIT_TELEX_RELEASE |
| LC document presentation | UCP 600 Article 14 — presentation by legal person | Finance Principal | BIT_LC_PRESENT |
| OFAC confirmed match authorization | OFAC enforcement guidance — reasonable procedures | Finance Principal | BIT_OFAC_AUTH |
| Credit write-off above threshold | Internal financial control | Finance Principal | BIT_CREDIT_WRITEOFF |

No agent session holds any of these bits. All five actions return `AuthorizationError` if attempted by any agent, regardless of business logic state.

### 2.3 The Forja SENSE Audit Trail

Every legal gate passage produces a Forja SENSE event carrying:
- The Captain or Finance Principal session token (cryptographic proof of human identity)
- The action performed
- The before/after state of the affected entity (BL, payment, credit record)
- The timestamp

The SENSE stream is append-only and stored in GRANTHX for long-term retrieval. When a regulator, bank, or litigant asks "who authorized this BL?", the answer is the `ebl.issued` SENSE event — immutable, timestamped, carrying the Captain's session token. This is not a log entry that can be edited. It is a committed event in the GRANTHX knowledge layer.

### 2.4 Layer 2 — AEGIS Hard Gate System

AEGIS (port 4850, product name KAVACH) is a full agent governance platform, not a single gate. Its enforcement engine has been validated over 104 batch soak runs across 7 live services with zero false positives and zero invariant violations across ~9,800 checks.

#### 2.4.1 Hard Gate Tiers

AEGIS classifies every service-capability pair by authority class and assigns it to a Hard Gate tier:

| Tier | Authority class | Gate behaviour |
|---|---|---|
| HG-1 | Impossible operation / empty capability surface | BLOCK on capability claim that cannot exist for this service |
| HG-2A | External call — service acts as external validator | GATE with approval token required |
| HG-2B | External state — service writes state outside its own DB | BLOCK without IRR-NOAPPROVAL scoped token |
| HG-2C | Pending — financial irreversibility doctrine | To be defined |

For the five FreightBox legal gate actions (Section 2.2), all five are classified HG-2B: they write state that is legally irreversible (an issued BL, an LC presentation, an OFAC adjudication). The AEGIS Hard Gate blocks execution until a scoped approval token is presented. The DAN (Destructive / Authorization-Required / Normal) scale provides the human-facing classification:

| DAN Level | Classification | Human delivery | Approval mechanism |
|---|---|---|---|
| L1 | Normal, low blast radius | None | Auto-approve, log |
| L2 | Notable, elevated risk | Dashboard notification | Proceed with log |
| L3 | Authorization-required | **WhatsApp via AnkrClaw** | `mintApprovalToken` → human approves → `verifyAndConsumeNonce` |
| L4 | Irreversible / dual-control | **WhatsApp to two separate humans** | Two distinct `verifyScopedApprovalToken` calls; same-person guard enforced |

#### 2.4.2 Five Locks — `@ankr/aegis-guard` SDK

The Five Locks are cryptographic primitives extracted from the carbonx-backend proof-of-concept (Batches 62–74) into a reusable SDK (`@ankr/aegis-guard`). FreightBox agents import this SDK directly:

```typescript
import {
  mintApprovalToken,       // Captain mints a scoped, time-limited token
  verifyScopedApprovalToken, // Agent verifies token before executing
  verifyAndConsumeNonce,   // Replay prevention — each token usable once
  digestApprovalToken,     // Tamper-evident digest for the receipt chain
  IrrNoApprovalError,      // Thrown if agent proceeds without valid token
  checkIdempotency,        // Fingerprint prevents duplicate execution
  buildIdempotencyFingerprint,
  emitAegisSenseEvent,     // SENSE event emission with transport config
} from '@ankr/aegis-guard';
```

`IrrNoApprovalError` is the key primitive for FreightBox Zero. When Priya attempts to present a finalized BL for issuance, she calls the `issueBL` mutation. Before the Layer 1 trust_mask check, she must hold a valid scoped approval token minted by the Captain. If no token is present, `IrrNoApprovalError` is thrown at the SDK layer — before any GraphQL resolving begins. The three enforcement surfaces (SDK exception → trust_mask check → AEGIS gate) are sequential and independent.

#### 2.4.3 Quality Mask

Every FreightBox Zero agent that operates autonomously must meet a quality threshold before being permitted to handle live shipments. AEGIS enforces this via a 16-bit quality bitmask:

```
Bits 0–11  (point-in-time at promotion, immutable after):
  0  typecheck_passed      4  migration_verified    8  audit_artifact_produced
  1  tests_passed          5  rollback_tested        9  scope_confirmed
  2  lint_passed           6  dependency_checked    10  no_secrets_exposed
  3  no_unrelated_diff     7  codex_updated         11  human_reviewed

Bits 12–15 (post-promotion drift, longitudinal):
  12  idempotency_evidenced    14  regression_suite_pass
  13  observability_evidenced  15  production_fire_zero
```

An agent whose `quality_mask` does not meet the HG-2B minimum (bits 0, 1, 5, 8, 11 set — typecheck, tests, rollback_tested, audit artifact, human_reviewed) cannot be promoted to Agentic Mode on live shipments. The quality mask is set at promotion time and is immutable. Post-promotion drift bits 12–15 are computed continuously and surfaced in the AEGIS fleet quality dashboard.

#### 2.4.4 Merkle Ledger — Service-Level Tamper-Evidence

AEGIS builds a CT-style RFC 6962 Merkle tree over PRAMANA receipt batches. Each batch is hashed into a Signed Tree Head (STH) signed with Ed25519. STHs are anchored to S3 Object Lock with a 7-year retention policy. This provides:
- **Inclusion proofs** — any receipt can be proven to be in the tree
- **Non-repudiation** — a Captain who approved a BL cannot later deny it; the STH signature proves the receipt existed at the claimed time
- **Long-retention anchor** — S3 Object Lock 7-year WORM storage exceeds the 6-year MLETR record-keeping recommendation

#### 2.4.5 The Captain's Command Centre (AEGIS Frontend)

AEGIS provides a built frontend (4,369 lines: server.ts 1,334 + aegis.js 982 + index.html 598 + route handlers 1,455) running at port 4850. For FreightBox Zero, this IS the Captain's primary review interface:

- **Approval cards** — one card per pending L3/L4 action; STOP / ALLOW / EXPLAIN buttons; countdown timer showing time remaining before auto-block; dual-control badge for DAN-L4
- **Cost attribution tree** — per-agent token spend, EWMA projection, budget bars with 80% warn / 95% soft-stop
- **Violation log** — prompt injection detections, HanumanG delegation failures, loop count warnings
- **Fleet quality dashboard** — quality mask per agent, drift score, promotion status
- **15-second auto-refresh** — the Captain's morning review is a live view, not a stale report

This interface requires no additional development for FreightBox Zero. The Captain opens port 4850, sees all pending agent approvals across all active shipments, and acts on them from a single screen. The Finance Principal sees only actions requiring `BIT_LC_PRESENT` or `BIT_OFAC_AUTH` — AEGIS multi-tenant isolation ensures each role sees only their gate actions.

### 2.5 Layer 3 — KavachOS Kernel Receipt Chain

KavachOS (port 4855) generates a deterministic seccomp-bpf kernel profile from the agent's `trust_mask + domain` combination. The profile is sealed with a SHA-256 hash at session start, stored in the PRAMANA receipt chain (Zenodo DOI: 10.5281/zenodo.19273330). Every syscall the agent makes is filtered against this profile at the Linux kernel layer.

The profile is deterministic: the same `trust_mask + domain` always produces the same profile and the same hash. This property enables:

**Profile drift detection:** If the syscall surface during a session differs from the sealed profile, KavachOS flags a Falco violation. The violation event is itself added to the PRAMANA chain. Drift indicates either a kernel-level supply chain attack or a profile modification attempt — both are security events, neither can be silently ignored.

**EU AI Act Article 14 compliance:** Article 14 requires human oversight of high-risk AI systems. KavachOS satisfies this at the infrastructure layer — the kernel receipt chain demonstrates human-defined syscall scope was enforced, independent of whether application logs are intact.

**Court-admissible evidence:** The receipt chain records what the agent actually did at OS level, not what it said it did in application logs. Application logs can be corrupted, modified, or incomplete. The kernel receipt chain, sealed at session start and appended with tamper-evident hashes, cannot be modified retroactively without breaking the chain.

For FreightBox Zero agents, KavachOS profiles are domain-tuned:
- **Priya (DOCUMENT, maritime domain):** file read + GraphQL calls + PDF generation syscalls permitted; process spawn and raw socket blocked
- **Kavya (FINANCE, freight domain):** database read + GraphQL calls + calculation syscalls; no file write outside designated invoice directory
- **Rekha/Aegis Guard (SYSTEM, compliance domain):** OFAC API calls + database write; no external network calls to non-whitelisted endpoints

### 2.6 Why Three Independent Layers

Each layer addresses the failure mode that the layer above it cannot prevent:

| Failure mode | Layer 1 (trust_mask) | Layer 2 (AEGIS Hard Gate) | Layer 3 (KavachOS) |
|---|---|---|---|
| Bug in approval check | ✅ trust_mask is below app logic | ✅ `IrrNoApprovalError` thrown at SDK layer before resolver | — |
| Misconfigured autonomy tier | ✅ Session bit is not tier-derived | ✅ HG tier is per-capability, not per-tier config | — |
| Prompt injection to upstream agent | ✅ Propagated state cannot set trust bit | ✅ Hard Gate evaluates the action, not the origin | — |
| Replay attack — reuse of approval token | — | ✅ `verifyAndConsumeNonce` burns token on first use | — |
| Duplicate execution of approved action | — | ✅ `checkIdempotency` + `buildIdempotencyFingerprint` | — |
| Under-quality agent promoted to Agentic Mode | — | ✅ Quality mask gate blocks promotion until bits 0/1/5/8/11 set | — |
| Supply chain attack on agent binary | — | ✅ Hard Gate runs in AEGIS process, not agent process | ✅ Kernel filter blocks unexpected syscalls |
| Application log tampering | — | ✅ Merkle STH + S3 Object Lock 7yr anchor at service level | ✅ Kernel receipt chain tamper-evident at OS level |
| Silent kernel-level exploit | — | — | ✅ Falco violation fires; chain breaks; alert raised |
| Non-repudiation dispute ("I didn't approve") | — | ✅ Ed25519-signed Merkle STH proves approval existed | ✅ KavachOS receipt proves what ran after approval |

The three layers are not redundant. They are complementary, each catching the class of failures the others cannot.

---

## 3. The Six Agents and Their Autonomy Tiers

FreightBox Zero operates six agents under the AnkrOS autonomy tier system:

| Agent | Name | Primary Domain | Autonomy Tier | Legal Gate Applies |
|---|---|---|---|---|
| INTAKE | Kiran | Quote, customer ingestion | BOT_AUTO | No |
| BOOKING | Arjun | SI, VGM, carrier booking | BOT_AUTO / BOT_CONFIRMED | No |
| DOCUMENT | Priya | BL draft, eBL chain | BOT_CONFIRMED | Yes (BIT_BL_ISSUE, BIT_TELEX_RELEASE) |
| FINANCE | Kavya | Invoice, payment, reconciliation | BOT_AUTO / BOT_CONFIRMED | Yes (BIT_LC_PRESENT, BIT_CREDIT_WRITEOFF) |
| TRACK | Disha | AIS feeds, container events | BOT_AUTO | No |
| AEGIS GUARD | Rekha | Trust enforcement, sanctions | SYSTEM | Yes (BIT_OFAC_AUTH) |

**BOT_AUTO** — agent executes without human confirmation. Reversible actions only: quotes, payment reminders, status updates, draft generation.

**BOT_CONFIRMED** — agent pauses and presents to a human for confirmation before executing. Used for: carrier bookings above value threshold, BL draft finalization, credit extensions.

**USER (legal gate)** — blocked at session layer. Cannot execute without a session holding the required trust_mask bit. BL issuance, telex release, LC presentation, OFAC adjudication.

The autonomy tier assignment is not a static configuration. Aegis Guard scores each action at runtime against five dimensions: customer familiarity, commodity complexity, corridor confidence, trade finance structure, and carrier reliability. The score determines whether a specific instance of a BOT_CONFIRMED action is elevated to USER tier or permitted to proceed with confirmation.

---

## 4. Three Operating Modes

The same platform serves three deployment configurations. The legal gate architecture is identical in all three — the gates cannot be disabled. The difference is the autonomy dial for non-gate actions.

### 4.1 Man Mode

All agent capabilities exist as UI tools. No agent executes autonomously. The human initiates every action; the agent responds with a recommendation or draft. Priya's BL draft generator is a button, not a workflow. Kiran's quote parser is a parser, not an inbound channel.

Appropriate for: compliance-heavy markets, DG-heavy corridors, NVOCCs building confidence in the agent layer, shipments below the Aegis confidence threshold.

### 4.2 Hybrid Mode

Each incoming booking is scored by Aegis Guard across the five dimensions. Bookings above the confidence threshold execute in Agentic Mode. Bookings below the threshold execute in Man Mode with agent-assist available. The threshold is operator-configurable.

The mode assignment is per-shipment, not per-company. A shipment in Agentic Mode can be transferred to Man Mode by the Captain at any time with one action — agents hand over the full SENSE event history and stop autonomous execution.

### 4.3 Agentic Mode

Six agents operate autonomously within their tier constraints. Three humans handle legal gates exclusively. Target throughput: 2,000+ shipments per month at constant staff count. Break-even: 80–120 shipments per month versus 350–500 for a traditional NVOCC staffing model.

The economic transformation: at 1,500 shipments per month, the Agentic Mode operator achieves $225,000–$450,000 net margin. The same volume with a traditional staffing model is not achievable — the staff cost exceeds the margin.

---

## 5. The Platform

FreightBox Zero is built on the FreightBox platform (live, freightbox.org):
- GraphQL API: 95 queries, 65 mutations, 8 real-time subscriptions
- DCSA WAVE-compliant eBL: 8 SENSE lifecycle events (ebl.created → ebl.archived)
- Complete NVOCC workflow: Quote → Booking → SI → BL → Shipment → Invoice → Settlement
- AnkrOS: BOT_AUTO / BOT_CONFIRMED / USER autonomy tiers with TRUST mask enforcement
- **AEGIS / KAVACH** (port 4850): Hard Gate system HG-1/2A/2B/2C; Five Locks SDK (`@ankr/aegis-guard`); DAN L1–L4 classification; PreToolUse interception; WhatsApp HITL with scoped approval tokens; Merkle ledger (RFC 6962, Ed25519 STH, S3 Object Lock 7yr); quality mask enforcement; fleet quality dashboard; multi-tenant isolation; 104-batch validated, 0 false positives (Zenodo DOI: 10.5281/zenodo.20034061)
- **AEGIS Dashboard Frontend** (port 4850): 4,369 lines — approval cards (STOP/ALLOW/EXPLAIN, countdown, dual-control badge), cost attribution tree, violation log, fleet quality view, 15s auto-refresh. Captain's Command Centre — no additional UI development required.
- **KavachOS** (port 4855): Kernel-layer agent governance, seccomp-bpf profiles from trust_mask + domain, Falco runtime monitoring, PRAMANA SHA-256 receipt chain, EU AI Act Art. 14 evidence layer
- AnkrClaw (port 4150): WhatsApp + Telegram ingestion — Kiran's primary interface and AEGIS HITL delivery channel
- GRANTHX (port 4130): Knowledge layer, SENSE event storage, long-term retrieval

No new platform infrastructure is required for FreightBox Zero. All three safety stack layers are running services, all fully built. FreightBox Zero is a deployment model on an existing, tested platform.

---

## 6. VIVECHANA Decision Score

Scored on 2026-05-06 per SAR-006 (DOI: 10.5281/zenodo.19456053):

| Axis | Score | Basis |
|---|---|---|
| I — Impact | 9 | Transforms NVOCC economics — 2,000+ freight forwarders in India alone below old break-even |
| L — Leverage | 9 | Unlocks Mode 2 licensing, data flywheel, managed service, protocol paper series |
| R — Reach | 8 | Eight ANKR services deepen utility: AnkrClaw, Aegis, GRANTHX, AnkrOS, EON, Forja, AnkrCodex, FreightBox |
| Rep — Replicability | 7 | Three-mode onramp works identically for each new licensee; carrier API config not yet fully automatic |
| F — Feasibility | 9 | All platform components live and tested; FMC OTI license (60–90 days) blocks first live shipment, not technology build |

```
K = 9 × 9 × 8 × 7 = 4,536
V = 4,536 × 9 = 40,824
```

**V = 40,824 — NOW BUILD**

---

## 7. Relationship to Prior Work

The VIVECHANA protocol (DOI: 10.5281/zenodo.19456053) provides the decision-scoring framework used in Section 6.

The AnkrOS autonomy tier system and TRUST mask architecture are developed in the ANKR platform. The specific application to NVOCC legal gate enforcement is first described in this paper.

MLETR — the UNCITRAL Model Law on Electronic Transferable Records — provides the legal foundation for eBL issuance as a negotiable instrument. The UK ETDA 2023 and Singapore ETA 2021 are the operative implementations in FreightBox Zero's primary corridors.

DCSA BL Standard 3.0 defines the 190+ attribute schema for the bill of lading. FreightBox implements this schema in the Priya DOCUMENT AGENT's draft generation pipeline.

The AEGIS system is described in: "AEGIS: An Evidence, Quality, and Authority Control Plane for Agentic AI Systems" (Zenodo DOI: 10.5281/zenodo.20034061). As of this paper (v3), AEGIS has evolved substantially beyond its Zenodo publication: the Hard Gate tiering system (HG-1/2A/2B/2C), the Five Locks `@ankr/aegis-guard` SDK, the 16-bit quality mask, the Merkle ledger with Ed25519 STH, and the 4,369-line frontend Command Centre were all built and soak-validated after the Zenodo publication date. The application of AEGIS as the Layer 2 enforcement surface in FreightBox Zero — with HG-2B classification for the five legal gate actions, Five Locks import by FreightBox agents, and AEGIS dashboard as the Captain's Command Centre — is first described in this paper.

The PRAMANA self-verifying cognition protocol (DOI: 10.5281/zenodo.19273330) provides the receipt chain foundation on which both the AEGIS Merkle ledger (service level) and KavachOS (kernel level) build their tamper-evident logs. Both services are PRAMANA-compliant: AEGIS at the approval-token and SENSE-event layer; KavachOS at the seccomp-bpf syscall layer.

No existing academic or industry publication describes the combination of session-layer trust_mask enforcement (Layer 1), Hard Gate capability enforcement with Five Locks cryptographic approval tokens (Layer 2), and kernel-level seccomp-bpf receipt chain (Layer 3) as a unified agent safety stack for regulated entity compliance. This paper is the first description of that architecture.

---

## 8. Conclusion

The inversion of the NVOCC operating model — from AI-as-assistant to AI-as-principal-operator — is made legally defensible by a three-layer agent safety stack. Layer 1 (TRUST mask) makes human-mandatory actions structurally impossible for agent sessions. Layer 2 (AEGIS Hard Gate) enforces capability-level gates via the `@ankr/aegis-guard` Five Locks SDK — approval tokens are minted by the human, cryptographically scoped, consumed once, and anchored in a Merkle ledger signed by Ed25519 — with the Captain's Command Centre dashboard providing the approval interface. Layer 3 (KavachOS kernel receipt) produces a tamper-evident, court-admissible record of what each agent actually did at OS level, independent of application logs.

The three layers are independent and complementary. Layer 1 eliminates policy bypass failures. Layer 2 catches Layer 1 bypass attempts before execution. Layer 3 provides kernel-level evidence independent of application logs. All three must fail simultaneously for a legal gate to be breached — a condition that is structurally impossible under normal operation and detectable within milliseconds under attack.

FreightBox Zero demonstrates that an NVOCC can operate legally and compliantly with three humans and six agents at volumes that would require 15–30 humans under a traditional staffing model. The three operating modes — Man, Hybrid, Agentic — provide a practical onramp that does not require operators to adopt full Agentic Mode from Day 1.

The three-layer stack is applicable beyond NVOCC operations to any regulated entity where specific actions carry legal liability assigned by law to natural persons. Trade finance presentation, customs declaration, regulated financial instrument issuance, and healthcare prescription are candidate domains. The stack's modularity — each layer is an independent deployed service — means it can be adopted incrementally: Layer 1 alone satisfies session-layer structural enforcement; Layers 1+2 add pre-execution HITL; all three add kernel-level audit evidence for the most demanding compliance regimes.

---

*FreightBox platform: freightbox.org | ANKR: ankr.in | PowerBox IT Solutions Pvt Ltd, India*

