# CanineRepro — Feature Estimate: Timeline, Effort & Cost

**Atomate Limited** · 17 April 2026

Consolidated view across the 9-month rewrite ([migration_estimate_9month.md](migration_estimate_9month.md)) and deferred features ([migration_estimate_deferred.md](migration_estimate_deferred.md)).

---

## Methodology Notes

- **Person-week (PW)** = one person, five days.
- **v1.0 effort** — person-weeks are derived from the full original estimate (30 March 2026) using BE+FE ratios only. The 9-month retainer model absorbs testing and QA into the two developers; there is no separate QA role. The $91,600 total is then allocated across features proportionally by PW weight.
- **v2.0 effort** — person-weeks and costs are taken directly from the deferred features estimate and include BE + FE + QA (self-tested by the same two developers). Domain Expert (DOM) person-weeks are excluded from effort and cost throughout — this resource is client-provided at zero cost to the project.
- **v2.3 items** have material additional costs beyond labour (clinical validation, cloud AI APIs, vendor agreements). These are shown separately below the main table rather than in the cost column.
- **Cloud infrastructure** is not included in any labour cost figure. Expect ~$170–$310/month during build and ~$285–$480/month in production (see individual estimates for detail).

---

## Consolidated Feature Table

| Feature | Timeline | Effort (PW) | Labour cost |
|---|---|---|---|
| **— v1.0 Rewrite & Modernisation —** | **M1 – M9** | **~77 PW** | **~$91,600** |
| Foundation — cloud infra, CI/CD, environments, auth, DB schema, design system | M1 | 10 | ~$13,000 |
| Database migration — SQL Server 2005 → PostgreSQL (schema + data quality + password hashing) | M1–2 | 1.5 | ~$1,700 |
| Clients & Pets — CRUD, breed lookup, pedigree, photos, practice short-code | M2–3 | 10 | ~$11,300 |
| Visits — Collection & Freezing (semen workflow, abnormalities, straw creation) | M3–4 | 7 | ~$7,900 |
| Visits — Chilling (send-to details, insemination instructions) | M4 | 3.5 | ~$4,000 |
| Visits — Soundness (testes measurements, notes) | M4 | 4 | ~$4,500 |
| Label Printing (integrated into Freezing workflow) | M4 | 1 | ~$1,100 |
| Cryogenic Storage — tank→canister→goblet→straw hierarchy, moves, disposition | M5 | 9 | ~$10,200 |
| Transfers — Transfer Out + Transfer In (straw assignment, shipping, receiving) | M6 | 10.5 | ~$11,900 |
| Invoicing — services, line items, payments (charge templates deferred) | M7 | 6.5 | ~$7,300 |
| Core Reports — 12 reports (Storage, Freezing App, Collection, Chilling, Transfer ×2, Invoice, Client, Pet, Soundness, Straw Labels, Practice Summary) | M8 | 12.5 | ~$14,100 |
| Access Database Import — self-service `.mdb` upload wizard, async import, conflict handling | M8 | 4 | ~$4,500 |
| UAT & Production Go-live | M9 | — | included above |
| **— v2.1 Quick Wins (4 months post-launch) —** | **M10 – M13** | **~30 PW** | **~$53,800** |
| AKC Sub-Forms — Freezing, Chilling, Fresh (regulatory docs, config-driven layout) | v2.1 · M1 | 5 | ~$8,600 |
| Pelleted Semen — dedicated workflow, separate data model, Freezing mode toggle | v2.1 · M1–2 | 5 | ~$8,800 |
| Medical History Profiles — male collection problems, female cycle + breeding outcomes | v2.1 · M2 | 5.5 | ~$9,500 |
| Charge Templates & Discount Groups — reusable templates, tiered discounts | v2.1 · M2–3 | 3.5 | ~$6,100 |
| Remaining 8 Reports — AKC reports, additional freezing/chilling, storage map, statistics | v2.1 · M3 | 9 | ~$15,800 |
| Multi-Language / i18n — 7 languages seeded from existing MultiLang tables | v2.1 · M1–4 | 3 | ~$5,000 |
| **— v2.2 OVT Integration (3 months post-v2.1) —** | **M14 – M16** | **~22.5 PW** | **~$38,700** |
| OVT Integration — cycle grid, progesterone/LH/cytology, who's due, statistics, pet↔chart link, sync | v2.2 · M1–3 | 22.5 | ~$38,700 |
| **— v2.3 Specialist (separate engagements) —** | **M17+** | | |
| AI-Assisted Semen Evaluation — photo-based morphology, motility, concentration scoring | v2.3 | 10 | ~$17,700 + additional† |
| VMS Integration — embed workflows into clinic management software (first platform) | v2.3 | 9 | ~$16,500–$20,500 + additional‡ |
| **— Background Infrastructure —** | **M21+** | | |
| Microservice Decomposition — split modular monolith into independent services | ≥12 mo post-launch | 5 | ~$9,500 |

---

## Phase Cost Summary

| Phase | Calendar duration | Labour cost | Cumulative labour |
|---|---|---|---|
| v1.0 — Rewrite & Modernisation | 9 months | ~$91,600 | ~$91,600 |
| v2.1 — Quick Wins | +4 months | ~$53,800 | ~$145,400 |
| v2.2 — OVT Integration | +3 months | ~$38,700 | ~$184,100 |
| **Core platform total (v1.0 + v2.1 + v2.2)** | **16 months** | | **~$184,100** |
| v2.3 — AI Semen Evaluation | Separate engagement | ~$17,700 + additional† | — |
| v2.3 — VMS Integration (first platform) | Separate engagement | ~$16,500–$20,500 + additional‡ | — |
| Microservice Decomposition (optional) | Background | ~$9,500 | — |

> The entire 16-month core platform can be delivered by the same two-developer team with no ramp-up between phases.

---

## v2.3 Additional Costs (beyond labour)

### † AI-Assisted Semen Evaluation

| Item | Estimated cost |
|---|---|
| Cloud AI vision API or custom model training | $5,000–$15,000 setup + ongoing per-analysis fees |
| Clinical validation study (board-certified veterinary reproductive specialist) — **mandatory prerequisite** | $15,000–$25,000 |
| Legal / liability review (AI-assisted clinical decision support) | $3,000–$8,000 |
| **Total additional** | **~$23,000–$48,000** |

Total indicative cost (labour + additional): **~$40,000–$66,000**

### ‡ VMS Integration (per platform)

| Item | Note |
|---|---|
| Vendor API agreements / certification | Variable; some platforms charge for API access |
| HL7 or file-import adapter | Required for VMS platforms without a REST API |
| Additional platforms beyond the first | ~$11,300–$16,700 per platform |

---

## Cloud Infrastructure (additional in all phases)

| Period | Monthly cost |
|---|---|
| Build phase (v1.0 months 1–9) | ~$170–$310 |
| Production — post-go-live | ~$285–$480 |
| Over 16-month core programme (indicative total) | ~$5,000–$9,000 |

> Infrastructure costs are separate from and in addition to all labour figures above.
