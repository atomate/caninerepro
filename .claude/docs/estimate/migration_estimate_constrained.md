# CanineRepro — Constrained Budget Estimate

**Atomate Limited** · 3 April 2026

---

## Engagement Model

| Parameter | Value |
|---|---|
| Monthly budget | $5,000 |
| Duration | 18 months maximum |
| Total budget | ~$90,000 |
| Primary resource | 1 full-stack developer (retainer, ~$4,500/month) |
| Occasional resources | DevOps, Domain Expert (~$500/month average) |
| Project management | Developer reports directly to client — no PM |

> This is a significantly reduced-scope engagement compared to the full estimate (~$223k–$245k). The choices below reflect what can realistically be delivered by one developer in 18 months while maintaining a working, production-ready system at each milestone.

---

## What We Are Building

A cloud-hosted React SPA backed by a lightweight Node.js API, replacing the core workflows of the BD-DSK desktop app. The full microservices architecture is deferred — the initial system will use a single deployable API service with a modular internal structure that can be split into microservices later. Database: PostgreSQL migrated from SQL Server 2005.

---

## Scope — What Is Included (v1.0 MVP)

| Area | Included | Notes |
|---|---|---|
| **Auth** | ✅ | Email/password login; per-clinic data isolation (ported from GUID_USER model) |
| **Clients & Pets** | ✅ | Full CRUD, breed lookup, pedigree fields, photos; practice short-code URL |
| **Visits — Collection & Freezing** | ✅ | Core semen collection workflow; abnormalities; straw creation + label printing |
| **Visits — Chilling** | ✅ | Send-to details, insemination instructions |
| **Visits — Soundness** | ⚠️ Simplified | Testes measurements and basic notes; AKC Soundness sub-form deferred |
| **Cryogenic Storage** | ✅ | Full tank→canister→goblet→straw hierarchy; move suggestions |
| **Transfer Out** | ✅ | Shipping straws to other practices; straw assignment; disposition |
| **Transfer In** | ⚠️ Deferred | Receive semen from external practices → v2.0 |
| **Invoicing** | ⚠️ Simplified | Services, invoice creation, line items, payments. Charge templates and discount groups deferred |
| **Core Reports** | ✅ 8 reports | Storage Report, Freezing Application, Collection Report, Chilling, Transfer, Invoice, Client, Pet |
| **Full report suite** | ⚠️ Deferred | Remaining 12 reports (soundness, AKC, storage map, etc.) → v2.0 |
| **AKC Sub-forms** | ⚠️ Deferred | AKC Freezing, Chilling, Fresh sub-forms → v2.0 |
| **Label Printing** | ✅ | Integrated into Freezing tab |
| **Sync Adapter** | ✅ | Legacy XML shim for VB6 desktop clients during transition |
| **Backup / Restore** | ✅ Basic | Automated DB snapshots; manual export |
| **English only** | ✅ | Multi-language deferred → v2.0 |
| **DB Migration** | ✅ | SQL Server 2005 → PostgreSQL; password hashing |

### Deferred to v2.0

- OVT integration (ovulation timing charts, cycle grid, sync) — most complex single module
- AKC sub-forms (Freezing / Chilling / Fresh)
- Transfer In (receiving semen from external practices)
- Pelleted semen workflow
- Medical history profiles (lab work, collection problems, breeding outcomes)
- Remaining 12 reports + full report setup
- Charge templates and discount groups
- Multi-language / i18n
- AI-assisted semen evaluation
- VMS integration

---

## Delivery Plan (18 Months)

| Month | Milestone | DevOps days |
|---|---|---|
| 1–2 | **Foundation** — cloud setup (dev + prod), PostgreSQL, Auth, DB migration scripts, CI/CD pipeline | 5 days (setup) |
| 3–5 | **Clients & Pets** — full CRUD, breed lookup, photos, pet detail view, search | — |
| 6–8 | **Visits** — Collection, Freezing (straws + label print), Chilling, basic Soundness | 1 day/month |
| 9–10 | **Storage** — hierarchy, tank populate, move/suggest, disposition | — |
| 11–12 | **Transfers & Invoicing** — Transfer Out, simplified invoicing, payments | — |
| 13–14 | **Reports** — 8 core reports; report header/logo setup | — |
| 15–16 | **Sync Adapter + Data Migration** — VB6 XML shim live; migrate real client data from SQL Server | 3 days |
| 17 | **UAT** — developer-led testing with client; bug fixes | — |
| 18 | **Go-live + buffer** — production deployment, handover documentation, monitoring setup | 3 days |

> The developer will provide fortnightly progress updates directly to the client. No formal sprint ceremonies are required, but a shared task board (Linear or Trello) is recommended.

---

## Budget Breakdown

### Monthly Allocation

| Item | Monthly | Over 18 months |
|---|---|---|
| Full-stack developer (retainer) | $4,500 | $81,000 |
| DevOps / cloud setup (occasional) | ~$300 avg | ~$5,400 |
| Domain Expert validation (occasional) | ~$200 avg | ~$3,600 |
| **Total labour** | **~$5,000** | **~$90,000** |

### Cloud Infrastructure (additional — not in labour budget)

| Component | Monthly |
|---|---|
| PostgreSQL (RDS, single AZ during dev, Multi-AZ prod) | $100–$180 |
| App server (single t3.small, auto-scaling deferred) | $30–$60 |
| S3 (photos, PDFs, backups) | $10–$20 |
| CI/CD + monitoring | $30–$50 |
| **Infra total** | **~$170–$310/month** |

> Infrastructure costs are separate from the $5K/month labour budget. Expect ~$2,000–$3,700 in cloud costs over the 18-month build period, then ~$170–$310/month ongoing.

### Ongoing Post-Launch (estimated)

| Item | Monthly |
|---|---|
| Developer maintenance (part-time retainer) | $2,000–$2,500 |
| Cloud infrastructure | $170–$310 |
| **Total ongoing** | **~$2,200–$2,800/month** |

---

## Risks Specific to This Setup

### Critical risks

| # | Risk | Impact | Mitigation |
|---|---|---|---|
| **C1** | **Single point of failure** — if the developer is ill, leaves, or becomes unavailable, the project stops. No one else knows the codebase. | **Critical** | Enforce code documentation standards from day one. Maintain a private Git repository the client controls. Agree a 2-month notice / handover clause in the contract. |
| **C2** | **No QA resource** — the developer self-tests. Bugs will reach UAT and production that a dedicated QA engineer would have caught. | **High** | Allocate 20% of each milestone to self-testing. Client must commit time to UAT at month 17. Consider a 2-week contract QA engagement ($2,000–$3,000) before go-live. |
| **C3** | **No architect review** — a solo developer will make tech debt decisions under time pressure. Structural mistakes (DB schema, API design) are expensive to fix after data is live. | **High** | Front-load architectural decisions in months 1–2. Consider a one-off 2-day architecture review (~$900) with a senior consultant at project start and again before month 9. |
| **C4** | **Budget overrun risk** — the 18-month fixed budget leaves no contingency. Any scope addition, unexpected complexity, or developer down-time eats directly into the timeline. | **High** | Treat the deferred feature list as genuinely deferred — do not add scope mid-project. Build a 1-month buffer into month 18. |

### Inherited risks (carried from full estimate)

| # | Risk | Level |
|---|---|---|
| R1 | VB6 shared library source unavailable — must test in running app | High |
| R3 | SQL Server data quality (orphaned GUIDs, constraint violations) | Medium |
| R4 | Domain knowledge gap — client must be available for regular domain questions | High |
| R10 | Plaintext passwords in USERS table — hashing migration required | Confirmed |
| R11 | SQL Server stored procedures must be enumerated and ported | Medium |

> **R4 is amplified** in this setup. With no PM to coordinate domain questions, the client (or a nominated domain expert) must be available within 24 hours to answer workflow questions. Delays in domain clarification directly delay the developer and burn budget.

---

## Comparison: Constrained vs. Full Estimate

| Dimension | Full estimate | This estimate |
|---|---|---|
| Team size | 8 people | 1 developer + occasional |
| Duration | 10–12 months | 18 months |
| Total cost | ~$223k–$245k | ~$90k (+infra) |
| Scope | Full feature parity + v1.0 extensions | Core workflows only (OVT, AKC, Pellets, Medical History deferred) |
| Architecture | Microservices | Modular monolith → split later |
| Risk level | Medium-High | **High** |
| Quality assurance | Dedicated QA engineer | Self-tested + client UAT |
| OVT integration | Phase 2 | Deferred to v2.0 |
| Report suite | 20 reports | 8 core reports |

---

## Key Decisions Before Starting

1. **OVT deferral acceptable?** OVT (ovulation timing charts, cycle tracking) is the second-largest module. Deferring it is the single biggest scope saving in this estimate. Confirm this is acceptable for v1.0.
2. **AKC sub-forms deferral acceptable?** AKC Freezing/Chilling/Fresh forms are regulatory documents. If clinics need these at launch, 2–3 months of effort must be added to the plan.
3. **Client availability for domain questions.** Who is the nominated contact, and what is their availability? Delays here are the most common cause of single-developer projects running over time.
4. **Contract terms for developer.** Agree IP ownership, notice period, handover obligations, and code repository access from day one.
5. **QA engagement.** Confirm whether a short contract QA engagement (~2 weeks, ~$2,500) will be budgeted before go-live, or whether client UAT is the only quality gate.
