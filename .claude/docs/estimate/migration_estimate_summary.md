# CanineRepro — Migration Estimate Summary

**Atomate Limited** · Prepared 30 March 2026 · Revised 3 April 2026

---

## What We Are Building

A cloud-hosted React SPA + Node.js microservices platform replacing the Windows-only BD-DSK desktop app. Migrating from SQL Server 2005 to PostgreSQL. Existing ASP/Perl server-side code is used as reference and decommissioned on go-live.

---

## Scope — v1.0

| Area | Key features |
|---|---|
| Clients & Pets | CRUD, breed lookup, pedigree, photos, practice short-code access |
| Semen Visits | Collection, Freezing (with label printing), Chilling, Soundness, AKC sub-forms; male collection problem recording |
| Cryogenic Storage | Tank → Canister → Goblet → Straw hierarchy, move suggestions |
| Transfers | Semen out (shipping) + Semen in (receiving from external practices) |
| Pelleted Semen | First-class workflow alongside visits; mode toggle; incoming pellets |
| Medical History | Structured male & female health histories, lab work, breeding outcomes |
| Invoicing | Service catalog, charge templates, discounts, payments |
| Reports | 20 PDF reports (Puppeteer); AKC registration documents |
| OVT Integration | Cycle charts, progesterone grid, who's due, sync protocol reimplemented |
| Infrastructure | Auth (JWT), i18n (7 languages), sync adapter (legacy VB6 XML shim), CI/CD, DR |

**v2.0 (out of scope for v1.0):** AI-assisted semen evaluation (photo analysis), VMS integration (AVImark, Cornerstone, ezyVet).

---

## Delivery Plan

| Phase | Content | Elapsed |
|---|---|---|
| 0 — Foundation | Cloud infra, auth, SQL Server → PG migration, CI/CD, DR | 5 weeks |
| 1 — Core Domain | Clients/Pets, Visits, Storage, Transfers, Pellets, Medical History | 10 weeks |
| 2 — Business Features | Invoicing, 20 reports, OVT integration | 10 weeks |
| 3 — Supporting | Sync adapter, i18n, photos, label printing, backup, licensing | 7 weeks |
| 4 — QA & Launch | Integration tests, UAT, data migration, ASP cutover, pen test | 5 weeks |
| **Total** | | **~37 weeks + 20% buffer = 10–12 months** |

---

## Team & Effort

| Role | Person-Weeks | Labour Cost |
|---|---|---|
| Backend Developer ×2 | 34–38 PW | $61,200–$68,400 |
| Frontend Developer ×2 | 32–35 PW | $57,600–$63,000 |
| Database Architect | 7–8 PW | $13,125–$15,000 |
| QA Engineer | 23–25 PW | $32,200–$35,000 |
| DevOps / Cloud | 12 PW | $22,500 |
| Solution Architect | 8 PW | $18,000 |
| Project Manager | 4.5 PW | $8,438 |
| Domain Expert | 17–18 PW | $0 (client-provided) |
| **Total** | **~139–149 PW** | **~$213k–$230k** |
| External pen test | | ~$10k–$15k |
| **Grand Total** | | **~$223k–$245k** |

> Rates: BE/FE $360/day · DBA/OPS/PM $375/day · ARC $450/day · QA $280/day. All estimates assume AI coding assistance (~30% efficiency gain on implementation tasks).

---

## Key Risks

| # | Risk | Level |
|---|---|---|
| R1 | VB6 shared library source unavailable — behaviour must be tested in running app | High |
| R4 | Domain knowledge gap — veterinary repro logic requires specialist validation throughout | High |
| R5 | Zero automated tests — every behaviour must be manually verified before migration | High |
| R10 | Plaintext passwords in USERS table — hashing migration + user password-reset required | Confirmed |
| R11 | SQL Server stored procedures (DELETE_STORAGE + others) must be enumerated and ported | Medium |

OVT sync protocol risk resolved — full server-side code is in the repository.

---

## Ongoing Costs (Post-Launch)

| Item | Annual |
|---|---|
| Maintenance team (1.1 FTE — BE, FE, OPS, QA) | ~$103,000 |
| Cloud infrastructure (PostgreSQL RDS, microservices, S3, CDN) | ~$6,000–$11,000 |
| **Total** | **~$109,000–$114,000/year** |

---

## Key Decisions Needed Before Starting

1. OVT: rebuild sync, integrate via existing protocol, or drop?
2. Multi-tenancy model: shared schema (existing GUID_USER) or separate schemas per clinic?
3. Which clinics' data to migrate vs. greenfield only?
4. Acceptable VB6 desktop transition window (Sync Adapter Service lifespan)?
5. AI semen evaluation: confirm v2.0 priority and commission feasibility study?
6. VMS integration: which platforms (AVImark, Cornerstone, ezyVet) and in what order?
