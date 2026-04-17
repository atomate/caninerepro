# CanineRepro — Deferred Features Estimate (v2.0)

**Atomate Limited** · 17 April 2026

---

## Preamble

This document estimates the effort and cost to deliver all features deferred from the 9-month v1.0 rewrite ([migration_estimate_9month.md](migration_estimate_9month.md)). Each deferred item is treated as a discrete, post-launch deliverable. Effort figures are derived from the original full estimate (30 March 2026) and adjusted where the v1.0 platform reduces rework.

**Assumptions:**
- Same two-developer team continues post-launch (no ramp-up overhead).
- Client domain expert remains available at ≤24h response time throughout.
- v1.0 platform is live and stable before any v2.0 work begins.
- Rates unchanged: Backend Developer $360/day ($1,800/week), Frontend Developer $360/day ($1,800/week), QA $280/day ($1,400/week), Solution Architect $450/day ($2,250/week).
- Person-week (PW) = one person, five days.

---

## Summary

| Feature | Phase | BE | FE | QA | DOM | Total PW | Labour cost |
|---|---|---|---|---|---|---|---|
| AKC sub-forms (Freezing / Chilling / Fresh) | v2.1 | 2 | 2 | 1 | 2 | 7 | ~$8,600 |
| Pelleted semen workflow | v2.1 | 2 | 2.5 | 0.5 | 0.5 | 5.5 | ~$8,800 |
| Medical history profiles | v2.1 | 2 | 2.5 | 1 | 1 | 6.5 | ~$9,500 |
| Remaining 8 reports | v2.1 | 4.5 | 3.6 | 0.9 | — | 9 | ~$15,800 |
| Charge templates & discount groups | v2.1 | 1.5 | 1.5 | 0.5 | — | 3.5 | ~$6,100 |
| Multi-language / i18n | v2.1 | 0.5 | 1.5 | 1 | — | 3 | ~$5,000 |
| **v2.1 subtotal** | | **12.5** | **13.6** | **4.9** | **3.5** | **34.5** | **~$53,800** |
| OVT integration | v2.2 | 8 | 10 | 4.5 | 4.5 | 27 | ~$38,700 |
| **v2.2 subtotal** | | **8** | **10** | **4.5** | **4.5** | **27** | **~$38,700** |
| AI-assisted semen evaluation | v2.3 | 4 | 3 | 2 | 2 | 12\* | ~$17,700\* |
| VMS integration (first platform) | v2.3 | 5.5 | 0.5 | 1 | — | 9\* | ~$16,700\* |
| **v2.3 subtotal** | | | | | | **~21** | **~$34,400\*** |
| Microservice decomposition | Background | 3 | — | — | — | ~5 | ~$9,500 |
| **TOTAL (all deferred)** | | | | | | **~87.5 PW** | **~$136,400** |

\* v2.3 items have material additional costs beyond labour — see individual sections below.

---

## Phase v2.1 — Quick Wins (Estimated 4 months, same 2-dev team)

Delivers the remaining domain features and report suite. These all build directly on existing v1.0 data models — no new infrastructure is required. i18n is woven into this phase as an ongoing task rather than a discrete sprint.

### Delivery Plan — v2.1

| Month (post-launch) | Backend (Dev 1) | Frontend (Dev 2) |
|---|---|---|
| **1** | AKC sub-forms API; Pelleted semen data model + API | AKC sub-forms UI (Freezing / Chilling / Fresh); i18n string extraction + framework setup |
| **2** | Medical history API (male + female); Charge templates + discount group API | Pelleted semen UI + mode toggle; Medical history UI (male + female) |
| **3** | Remaining 8 reports — data queries + PDF generation (parallelised across report groups) | Remaining 8 reports — UI viewers, filters, print/download; Charge templates UI |
| **4** | i18n backend strings; QA cycles; bug fixes across v2.1 | i18n UI integration (all 7 languages); UAT with domain expert; polish |

> Month 4 is split: first two weeks are QA / domain expert UAT; final two weeks are deployment and sign-off.

---

### AKC Sub-Forms (Freezing / Chilling / Fresh)

**What it is:** Regulatory documents for American Kennel Club semen submissions. Three sub-forms (AKC Freezing, AKC Chilling, AKC Fresh) accessible from within the existing Visit workflow. Currently deferred because they require significant domain expert involvement to validate form fields against AKC requirements.

**Effort:** 2 PW BE · 2 PW FE · 1 PW QA · 2 PW DOM = **7 PW**

| Task | BE | FE | QA | DOM |
|---|---|---|---|---|
| AKC data model extension (AKC_FREEZINGS, AKC_CHILLINGS, AKC_FRESH tables) | 0.5 | — | — | — |
| AKC Freezing sub-form API + UI | 0.5 | 0.7 | 0.3 | 0.7 |
| AKC Chilling sub-form API + UI | 0.5 | 0.7 | 0.3 | 0.7 |
| AKC Fresh sub-form API + UI | 0.5 | 0.6 | 0.4 | 0.6 |
| AKC report output (PDF) | — | — | — | — |
| **Totals** | **2** | **2** | **1** | **2** |

**Labour cost:** (2 × $1,800) + (2 × $1,800) + (1 × $1,400) = **~$8,600**

**Dependencies:** Visits module (v1.0 complete). AKC tables already exist in the migrated PostgreSQL schema.

**Risk:** AKC form field requirements change periodically. Build the form layout from a configuration-driven template rather than hardcoding fields, so future AKC requirement changes do not require code changes.

---

### Pelleted Semen Workflow

**What it is:** Elevates the hidden VB6 "Use pelleted semen" preference to a first-class workflow. Pellets are a distinct semen format (different unit, thawing protocol, storage form factor) requiring a separate data model from straws. Includes a Pelleted navigation section, mode toggle on the Freezing tab, and receiving pellets via Transfer In.

**Effort:** 2 PW BE · 2.5 PW FE · 0.5 PW QA · 0.5 PW DOM = **5.5 PW**

| Task | BE | FE | QA | DOM |
|---|---|---|---|---|
| Pellet data model + API (form factor, quantity unit, thawing protocol) | 1 | — | 0.5 | 0.5 |
| Pelleted navigation section + screens | — | 1.5 | — | — |
| Freezing ↔ Pelleted mode toggle | 0.5 | 0.5 | — | — |
| Receiving pellets via Transfer In (extends v1.0 Transfer In) | 0.5 | 0.5 | — | — |
| **Totals** | **2** | **2.5** | **0.5** | **0.5** |

**Labour cost:** (2 × $1,800) + (2.5 × $1,800) + (0.5 × $1,400) = **~$8,800**

**Dependencies:** Storage module (v1.0), Transfer In (v1.0). The pellet data model is a new table — no changes to existing straw schema.

---

### Medical History Profiles

**What it is:** Structured per-animal health history beyond what is captured in visit records. Males: ultra-semen quality notes, collection problems, corrective steps, breeding history. Females: heat cycle outcomes, pregnancy result, breeder quality notes (stood well, etc.), lab work from medical exams. Female cycle data that overlaps with OVT covers only the non-OVT structured notes; full cycle grid remains in v2.2 OVT.

**Effort:** 2 PW BE · 2.5 PW FE · 1 PW QA · 1 PW DOM = **6.5 PW**

| Task | BE | FE | QA | DOM |
|---|---|---|---|---|
| Male health history (lab work, collection problems, corrective steps) | 1 | 1.5 | 0.5 | 0.5 |
| Female breeding history (cycle outcomes, breeder quality, pregnancy result) | 1 | 1 | 0.5 | 0.5 |
| **Totals** | **2** | **2.5** | **1** | **1** |

**Labour cost:** (2 × $1,800) + (2.5 × $1,800) + (1 × $1,400) = **~$9,500**

**Dependencies:** Clients & Pets module (v1.0), Visits module (v1.0).

---

### Remaining 8 Reports

**What it is:** The full original suite contains 20 reports; v1.0 delivers 12. The remaining 8 span AKC reporting, additional freezing/chilling breakdowns, statistical practice summary variants, and the storage map visual.

**Effort:** 4.5 PW BE · 3.6 PW FE · 0.9 PW QA = **9 PW**

| Report group | BE | FE | QA |
|---|---|---|---|
| AKC reports (2: AKC Freezing report, AKC Chilling report) | 1 | 1 | 0.5 |
| Additional freezing/chilling reports (3) | 1.8 | 1.2 | 0.3 |
| Statistical / practice reports (2: storage map, practice statistics) | 1.2 | 1 | 0.1 |
| Remaining client/visit detail report (1) | 0.5 | 0.4 | — |
| **Totals** | **4.5** | **3.6** | **0.9** |

**Labour cost:** (4.5 × $1,800) + (3.6 × $1,800) + (0.9 × $1,400) = **~$15,800**

**Dependencies:** v1.0 report infrastructure (PDF engine, report viewer UI) is already in place. AKC reports depend on AKC sub-forms (same phase). OVT-linked statistical reports depend on v2.2 OVT and should be deferred to after v2.2.

---

### Charge Templates & Discount Groups

**What it is:** Reusable charge templates that pre-fill invoice line items for common services, and tiered discount groups applied automatically per client or service combination. v1.0 invoicing supports manual line items and payments; templates and discounts are the remaining invoicing features.

**Effort:** 1.5 PW BE · 1.5 PW FE · 0.5 PW QA = **3.5 PW**

| Task | BE | FE | QA |
|---|---|---|---|
| Charge template CRUD + auto-apply on invoice creation | 0.75 | 0.75 | 0.25 |
| Discount group rules engine + client assignment | 0.75 | 0.75 | 0.25 |
| **Totals** | **1.5** | **1.5** | **0.5** |

**Labour cost:** (1.5 × $1,800) + (1.5 × $1,800) + (0.5 × $1,400) = **~$6,100**

**Dependencies:** Invoicing module (v1.0). DISCOUNT_GROUPS table already exists in the migrated schema.

---

### Multi-Language / i18n

**What it is:** The SQL Server database already contains 7 language seeds (EN, FR, IT, GER, ESP, SWE, ICE) in the MultiLang/ListLanguages tables. This feature wires a standard i18n framework across the full React SPA and API error messages, seeded from those tables. English is already the default; this adds the ability to switch language per user.

**Effort:** 0.5 PW BE · 1.5 PW FE · 1 PW QA = **3 PW**

> i18n is woven through the entire v2.1 phase rather than delivered as a single sprint — string extraction and tagging happens alongside each feature. The 3 PW represents the total overhead, not a sequential block.

**Labour cost:** (0.5 × $1,800) + (1.5 × $1,800) + (1 × $1,400) = **~$5,000**

**Dependencies:** None beyond v1.0 platform. Seed data already in PostgreSQL from migration.

---

## Phase v2.2 — OVT Integration (Estimated 3 months, same 2-dev team)

OVT (Ovulation Timing) is the largest single module in the original codebase. It is scoped as a standalone phase because it has its own data model (FORMS, DAYS, INSEMS, SYNC tables), its own UI sub-system (cycle grid, who's-due view, statistics), and links into Visits and Transfers in ways that require careful integration testing.

**Why it was deferred:** OVT was deferred from v1.0 purely on size grounds — it adds ~27 PW to a plan that was already fully utilising two developers for 9 months. It does not block any core clinic workflow; clinics that use OVT continue to do so via the existing OVT system until v2.2 is live.

**Full sync protocol is known:** `sync_web.asp` in the repository contains the complete OVT sync implementation (RECLIST / UPDATERECS / DELETERECS, CRC32 + zlib). There is no reverse-engineering risk here — this is a known implementation, not a discovery phase.

### Delivery Plan — v2.2

| Month (post-v2.1) | Backend (Dev 1) | Frontend (Dev 2) |
|---|---|---|
| **1** | OVT data model + API (FORMS, DAYS, INSEMS); cycle CRUD endpoints | OVT cycle grid UI — daily progesterone / LH / cytology entry; chart navigation |
| **2** | Who's Due + statistics queries; OVT ↔ BD-DSK link (pet↔chart, disposition↔insemination) | Who's Due view + statistics UI; OVT reports (if deferred from v2.1) |
| **3** | OVT sync reimplementation (protocol from `sync_web.asp`); integration tests | UAT with domain expert; bug fixes; deployment |

---

### OVT Feature Detail

**Effort:** 8 PW BE · 10 PW FE · 4.5 PW QA · 4.5 PW DOM = **27 PW**

| Task | BE | FE | QA | DOM |
|---|---|---|---|---|
| OVT chart CRUD (FORMS, DAYS, INSEMS tables) | 2 | 3 | 1 | 2 |
| Cycle grid (daily progesterone / LH / cytology input) | 2 | 3 | 1 | 1 |
| Who's Due + statistics reports | 1 | 2 | 0.5 | 1 |
| OVT ↔ BD-DSK link (pet↔chart, disposition↔insemination) | 1.5 | 1 | 1 | 0.5 |
| OVT sync reimplementation (protocol known from `sync_web.asp`) | 1.5 | 1 | 1 | — |
| **Totals** | **8** | **10** | **4.5** | **4.5** |

**Labour cost:** (8 × $1,800) + (10 × $1,800) + (4.5 × $1,400) = **~$38,700**

**Dependencies:** Visits module (v1.0) for pet↔chart link. Transfer/disposition link (v1.0). No dependency on v2.1 features.

**Risk:** The domain expert's time commitment for OVT is the highest of any module (4.5 PW = ~22 days of domain review). Cycle interpretation, progesterone thresholds, and insemination timing are specialised. Block milestone sign-off on domain expert review at the end of each of the three months.

---

## Phase v2.3 — Specialist Features (Separate Engagements)

The following features require specialist third-party involvement, regulatory considerations, or per-platform vendor agreements that place them outside a standard retainer engagement. Each should be scoped as a separate statement of work after v2.1 and v2.2 are stable.

---

### AI-Assisted Semen Evaluation

**What it is:** Photo-based morphological analysis of semen samples with automated scoring for concentration, motility (1–5), velocity, and movement patterns. Results inform the evaluation fields in the Collection tab.

**Labour effort:** 4 PW BE · 3 PW FE · 1 PW ARC · 2 PW QA · 2 PW DOM = **~12 PW**

**Labour cost:** (4 × $1,800) + (3 × $1,800) + (1 × $2,250) + (2 × $1,400) = **~$17,700**

**Additional costs (outside labour budget):**

| Item | Estimated cost |
|---|---|
| Cloud AI vision API or custom model training | $5,000–$15,000 setup + ongoing per-analysis fees |
| Clinical validation study (board-certified veterinary reproductive specialist) | $15,000–$25,000 |
| Legal / liability review (AI-assisted clinical decision support) | $3,000–$8,000 |
| **Additional total** | **~$23,000–$48,000** |

**Total indicative cost (labour + additional):** ~$40,000–$66,000

> **Critical prerequisite:** AI-based morphological analysis of semen images must be clinically validated by a board-certified veterinary reproductive specialist before use in practice. Misclassification directly affects breeding decisions. This validation study is a mandatory pre-condition of implementation and is itself a separate engagement.

---

### VMS Integration

**What it is:** Embedding CanineRepro workflows into existing veterinary management software (AVImark, Cornerstone, ezyVet, ImproMed). Each VMS platform is a separate integration workstream.

**Labour effort per platform:** 5–7 PW BE · 0.5 PW FE · 1 PW QA · 2 PW ARC (first platform only) = **8.5–10.5 PW**

**Labour cost:**

| | First platform | Each additional platform |
|---|---|---|
| BE | 5.5–7 PW avg × $1,800 | 5–7 PW × $1,800 |
| FE | 0.5 PW × $1,800 | 0.5 PW × $1,800 |
| QA | 1 PW × $1,400 | 1 PW × $1,400 |
| ARC (one-time integration design) | 2 PW × $2,250 | — |
| **Total** | **~$16,500–$20,500** | **~$11,300–$16,700** |

**Additional considerations:** Some VMS platforms require vendor API agreements or certification. HL7 file import may be needed for systems without an API. Recommend identifying the 2–3 most common VMS platforms used by the target clinic base before scoping.

---

## Background — Microservice Decomposition

**What it is:** The v1.0 (and v2.x) platform uses a modular monolith API. At some point — typically when the team grows beyond 3–4 developers or when independent scaling of specific services becomes necessary — splitting into microservices (Auth, Client/Pet, Visit, Storage, Transfer, Invoice, OVT, Report) may be warranted.

**This is not a user-facing feature.** It should only be undertaken if a clear operational or team-scaling need exists. Premature decomposition adds infrastructure complexity and cost without user benefit.

**Effort:** 3 PW BE · 1 PW OPS · 1 PW ARC = **~5 PW**

**Labour cost:** (3 × $1,800) + (1 × $1,875) + (1 × $2,250) = **~$9,525**

**Recommended trigger:** Do not initiate until at least 12 months post-launch and only if concurrent team size exceeds 3 developers or if a specific service requires independent scaling (most likely: Report service for large PDF generation loads).

---

## Budget Summary

### v2.1 + v2.2 (Core deferred features — recommended roadmap)

| Phase | Calendar time | Labour cost |
|---|---|---|
| v2.1 — Quick Wins | ~4 months | ~$53,800 |
| v2.2 — OVT | ~3 months | ~$38,700 |
| **Subtotal** | **~7 months** | **~$92,500** |

> Delivered continuously by the same 2-developer team post-launch, total elapsed time for all core deferred features is **~7 months**, at a total labour cost of **~$92,500**. Cloud infrastructure costs (same ~$285–$480/month as v1.0 production) continue throughout.

### v2.3 Specialist (separate engagements, indicative only)

| Feature | Labour cost | Additional costs | Total indicative |
|---|---|---|---|
| AI semen evaluation | ~$17,700 | ~$23,000–$48,000 | ~$40,000–$66,000 |
| VMS (first platform) | ~$16,500–$20,500 | Vendor/API agreements (variable) | ~$16,500–$20,500+ |

### Background

| Item | Labour cost | When |
|---|---|---|
| Microservice decomposition | ~$9,500 | Optional; 12+ months post-launch only |

---

## Recommended Sequencing

```
v1.0 go-live
    │
    ├─ Month 1–4:  v2.1 (Quick Wins)
    │               ├─ AKC sub-forms
    │               ├─ Pelleted semen
    │               ├─ Medical history
    │               ├─ Remaining 8 reports
    │               ├─ Charge templates
    │               └─ i18n (woven throughout)
    │
    ├─ Month 5–7:  v2.2 (OVT)
    │               └─ OVT integration (cycle grid, who's due, sync)
    │
    └─ Month 8+:   v2.3 (Specialist — separate engagements)
                    ├─ AI semen evaluation (requires clinical validation first)
                    └─ VMS integration (one platform at a time)
```

> v2.1 and v2.2 can begin immediately after v1.0 go-live with the same team. v2.3 items require additional scoping and should not begin until v2.2 is stable. Microservice decomposition is optional and should be assessed at the 12-month post-launch mark.

---

## Key Decisions Before Starting v2.0

1. **OVT before or after v2.1?** OVT has no dependency on v2.1 features. If OVT is a higher business priority than AKC sub-forms or pelleted semen, the phases can be reordered. Note that OVT demands significantly more domain expert time (4.5 PW vs 3.5 PW for all of v2.1).
2. **AKC sub-forms priority.** If clinics require AKC regulatory forms before other v2.1 features, AKC sub-forms can be pulled to the front of the phase without affecting other workstreams.
3. **AI semen evaluation pre-requisite.** Who commissions and funds the clinical validation study? This determines whether v2.3 AI work can begin at month 8 or later.
4. **VMS platform priority.** Which 2–3 VMS platforms should be targeted first? This drives the v2.3 integration architecture design.
5. **Maintenance continuity.** If only one developer continues post-v1.0 (reduced retainer), v2.1 calendar time roughly doubles to ~7 months. Confirm whether the full 2-developer team is retained through v2.0.
