# BD-DSK — Web Application Rewrite: Delivery Estimate

**Prepared by:** Atomate Limited  
**Date:** 18 April 2026  
**Revision:** 1.1

---

## 1. Overview

This document estimates the full rewrite and modernisation of **BD-DSK**, a legacy Visual Basic 6 desktop application (~44,500 LOC), into a contemporary cloud-hosted web application.

> **This is a complete rewrite.** No VB6 code is carried forward. The new application is built from scratch on a modern stack, with all business logic, data models, and workflows re-implemented and improved. The existing VB6 codebase and Access database serve solely as the authoritative domain specification.

**Out of scope:** Device sync, OVT mobile integration, and PocketPC/Palm workflows are explicitly excluded from this engagement.

---

## 2. Team

| Role | Allocation |
|---|---|
| Full-Stack Developer 1 (+ DevOps) | Full-time |
| Full-Stack Developer 2 | Full-time |
| QA Engineer | Part-time (~40%) |

**Blended labour rate:** $1,200 / person-week (PW)  
QA overhead is folded into each feature's PW allocation.

---

## 3. Target Architecture

```
┌─────────────────┐  ┌─────────────────────────────┐  ┌─────────────────┐
│  Marketing &    │  │     BD-DSK Web Application  │  │  Admin Portal   │
│  Lead Capture   │  │       React SPA             │  │   React SPA     │
│  Website        │  │  TypeScript · Tailwind      │  │  (internal)     │
│  (Next.js/SSG)  │  │  React Query                │  │                 │
└────────┬────────┘  └─────────────┬───────────────┘  └────────┬────────┘
         │                         │ HTTPS / REST               │
         └─────────────────────────▼───────────────────────────┘
                       ┌───────────────────────┐
                       │   Node.js / Express   │
                       │   Auth (JWT/RBAC)     │
                       │   Business Logic      │
                       │   PDF gen · Mailer    │
                       └──────┬──────────┬─────┘
                              │          │
                    ┌─────────▼──┐  ┌────▼───────────┐
                    │ PostgreSQL │  │   S3 Storage   │
                    │ (primary)  │  │ (files/labels) │
                    └────────────┘  └────────────────┘
                              │
                    ┌─────────▼──────────────────────┐
                    │    CI/CD · Docker · Cloud      │
                    │    (Railway / Render / ECS)    │
                    └────────────────────────────────┘
```

---

## 4. Feature Breakdown & Cost

### 4.1 Core Delivery — Target: 9 Months

A fully functional web application covering all primary workflows, ready for production use.

| # | Feature | Months | PW | Cost (USD) | Scope |
|---|---|:---:|:---:|---:|---|
| 1 | **Foundation & Infrastructure** | M1–2 | 10 | $12,000 | Cloud hosting, CI/CD pipeline, authentication (JWT/RBAC), project skeleton (API + SPA + marketing site + admin portal), dev/staging/prod environments |
| 2 | **DB Schema Design & Migration** | M1–2 | 2 | $2,400 | PostgreSQL schema designed from Access/SQL Server source; migration scripts; seed data |
| 2a | **Marketing & Lead Capture Website** | M2–3 | 4 | $4,800 | Public-facing Next.js site: product presentation, pricing, lead/registration form, email capture, CMS-editable content, transactional email confirmation |
| 2b | **Admin Portal — User & Lead Management** | M3–4 | 4 | $4,800 | Internal dashboard: registered user list, lead pipeline view, account activation/suspension, role assignment, basic usage stats, email-to-lead actions |
| 2c | **Legacy Data Import (Access → New DB)** | M4–6 | 6 | $7,200 | Access DB → PostgreSQL ETL; field-level schema mapping; validation UI; duplicate resolution, full audit report, re-run capability |
| 3 | **Clients & Pets** | M2–3 | 10 | $12,000 | Full CRUD for clients and pets, search, relationships, contact management |
| 4 | **Visits — Collection & Freezing** | M3–4 | 7 | $8,400 | Semen collection records, motility/morphology evaluation, freeze log, straw batch creation |
| 5 | **Visits — Chilling** | M4 | 3.5 | $4,200 | Chilled semen shipment workflow, extender/media records |
| 6 | **Visits — Soundness** | M4–5 | 4 | $4,800 | Brucellosis testing, health check records, clearance status |
| 7 | **Label Printing** | M5 | 2 | $2,400 | Straw label generation (PDF/ZPL), browser-based print, batch printing |
| 8 | **Storage Management** | M5–6 | 9 | $10,800 | Tank → canister → goblet → straw hierarchy; location suggest/move; inventory view |
| 9 | **Transfers & Disposition** | M6–7 | 10.5 | $12,600 | Transfer workflow, straw disposition (used/destroyed/transferred), full audit trail |
| 10 | **Invoicing** | M7–8 | 6.5 | $7,800 | Services, payments, discounts, charge templates, invoice PDF export |
| 11 | **Core Reports (priority set)** | M8–9 | 7 | $8,400 | ~8 highest-priority reports delivered by M9 (semen inventory, transfer history, client summary, visit report, storage map, soundness, chilling, financials) |
| | **Core Total** | **M1–9** | **85.5** | **$102,600** | |

### 4.2 Extended Delivery — Months 10–12

Completes the full report set, finalises the data import tool, and delivers production go-live through structured UAT.

| # | Feature | Months | PW | Cost (USD) | Scope |
|---|---|:---:|:---:|---:|---|
| 12 | **Remaining Reports** | M10–11 | 5.5 | $6,600 | Remaining ~12 reports (AKC sub-forms, collection trends, label manifest, storage utilization, etc.) |
| 13 | **QA / UAT / Production Deploy** | M9–12 | 10 | $12,000 | Ongoing QA throughout; formal UAT in M11; production cutover in M12 |
| | **Extended Total** | **M10–12** | **15.5** | **$18,600** | |

### 4.3 Summary

| Tier | Calendar | PW | Labour Cost |
|---|---|:---:|---:|
| Core delivery | M1–9 | 85.5 | $102,600 |
| Extended delivery | M10–12 | 15.5 | $18,600 |
| **Total** | **12 months** | **101** | **$121,200** |

**Additional costs:**

| Item | Estimate |
|---|---|
| Cloud infrastructure (build phase) | ~$170–$310 / month |
| Cloud infrastructure (12 months) | ~$2,000–$3,700 total |
| **Total project cost** | **~$123,200–$124,900** |

---

## 5. Timeline Schematic

```
Month         M1    M2    M3    M4    M5    M6    M7    M8    M9   M10   M11   M12
              ──────────────────────────────────────────────────────────────────────
              ◄──────────── CORE DELIVERY (target go-live M9) ─────────────►◄─EXT─►

Foundation    ██████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
DB Schema     ██████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Mktg Website        ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Admin Portal              ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Data Import                     ████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░
Clients/Pets        ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Visits Coll.              ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Visits Chill              ░░░░░████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Visits Sound              ░░░░░████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Label Print                    ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Storage                              ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
Transfers                                  ████████████████░░░░░░░░░░░░░░░░░░░░░░░
Invoicing                                        ████████████████░░░░░░░░░░░░░░░░░
Core Reports                                           ████████████░░░░░░░░░░░░░░░
              ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
QA / UAT                                               ░░░░████████████████████████  ← rolling
Rem. Reports                                                     ████████████████░░
Production Deploy                                                            ██████
              ──────────────────────────────────────────────────────────────────────
              M1    M2    M3    M4    M5    M6    M7    M8    M9   M10   M11   M12

Legend:  ██ Active development   ░░ Parallel/background activity
```

**Key milestones:**

| Milestone | Month |
|---|---|
| Dev environment & auth live | End M2 |
| Marketing website & lead capture live | End M3 |
| Admin portal — user management live | End M4 |
| Client/pet records working end-to-end | End M3 |
| All visit types implemented | End M5 |
| Storage & transfers complete | End M7 |
| Invoicing complete | End M8 |
| Core application feature-complete | End M9 |
| Data import tool complete | End M6 |
| Full report set complete | End M11 |
| Production go-live | End M12 |

---

## 6. Key Assumptions

- Client provides access to the existing `.mdb` / SQL Server database for schema analysis and test data
- Domain expert (clinic staff) available for requirements clarification and UAT participation
- No mobile application or offline support required in this engagement
- Single-clinic deployment (multi-clinic support is a future scope item)
- All sync and OVT mobile workflows are **out of scope**
- Report layout/branding approved before M8 development begins

---

## 7. Ongoing Maintenance (post go-live)

| Item | Monthly | Annual |
|---|---|---|
| 0.25 FTE developer (support + minor features) | ~$2,600 | ~$31,200 |
| Cloud infrastructure (production) | ~$285–$480 | ~$3,400–$5,800 |
| **Total** | | **~$34,600–$37,000 / year** |
