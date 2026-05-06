# FreightBox

**Live platform:** [freightbox.org](https://freightbox.org)

DCSA WAVE-compliant eBL platform and AI-operated NVOCC model.

---

## FreightBox Zero — AI-Operated NVOCC

Six agents, three humans, three-layer safety stack.

| Layer | System | Mechanism |
|---|---|---|
| 1 | TRUST mask (AnkrOS) | BL/telex/LC/OFAC bits structurally absent from agent sessions |
| 2 | AEGIS Hard Gate (port 4850) | Five Locks SDK · HG-1/2A/2B/2C · WhatsApp HITL · Merkle ledger |
| 3 | KavachOS (port 4855) | seccomp-bpf kernel profile · PRAMANA SHA-256 receipt chain |

**V = 40,824** (I=9, L=9, R=8, Rep=7, K=4,536, F=9) — NOW BUILD

---

## Published Papers (Zenodo)

| Paper | DOI |
|---|---|
| Legal Gate Architecture v3 — Hard Gate + Five Locks + Kernel Receipts | [10.5281/zenodo.20047736](https://doi.org/10.5281/zenodo.20047736) |
| Formal Project Report — Three-Layer Safety Stack | [10.5281/zenodo.20047563](https://doi.org/10.5281/zenodo.20047563) |
| Legal Gate Architecture v1 (original) | [10.5281/zenodo.20047472](https://doi.org/10.5281/zenodo.20047472) |

---

## Documents in This Repo

| Document | Description |
|---|---|
| `freightbox-zero--legal-gate-architecture--paper--2026-05-06.md` | Technical paper — three-layer safety stack (v3, current) |
| `freightbox-zero--project--formal--2026-05-06.md` | Formal project report — three modes, competitive landscape, GTM |
| `freightbox-zero--brainstorm--formal--2026-05-06.md` | Original brainstorm — six agents, three humans, economics |
| `freightbox-zero--vivechana--formal--2026-05-06.md` | VIVECHANA decision score — V = 40,824 |

---

## Platform Stack

- **FreightBox backend** — Fastify + Mercurius GraphQL, port 4003
- **FreightBox frontend** — Next.js, port 3001
- **AnkrOS** — BOT_AUTO / BOT_CONFIRMED / USER autonomy tiers
- **AEGIS / KAVACH** — Agent governance, Hard Gate, Five Locks SDK (`@ankr/aegis-guard`)
- **KavachOS** — Kernel-layer seccomp-bpf governance, PRAMANA receipt chain
- **AnkrClaw** — WhatsApp + Telegram ingestion, port 4150
- **GRANTHX** — Knowledge layer, SENSE event storage, port 4130

---

*PowerBox IT Solutions Pvt Ltd · ANKR · [ankr.in](https://ankr.in)*
