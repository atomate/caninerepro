# CanineRepro — 9-Month Rewrite & Modernisation Estimate

**Atomate Limited** · 17 April 2026

---

## Engagement Model

| Parameter | Value |
|---|---|
| Duration | 9 months |
| Team | 2 developers (backend lead + frontend lead) |
| Domain expertise | Provided by client throughout (no cost to project) |
| Sync / legacy compatibility | **Out of scope** — clean rewrite only |
| Architecture | Modular monolith API + React SPA; microservice split deferred to post-launch |

> This estimate supersedes the 18-month constrained estimate (3 April 2026). With two developers working in parallel and no sync adapter to build, the same 18 developer-months of effort is compressed to 9 calendar months. The removal of the legacy XML shim also enables cleaner architecture decisions throughout — no backward-compatibility constraints.

---

## Disclosure

1. **Estimate basis.** All figures are indicative estimates based on the information available at time of preparation. They represent professional judgement and should not be treated as contractual commitments or guarantees of cost or schedule.
2. **Scope assumptions.** Sync adapter and all legacy VB6 compatibility shims are explicitly excluded. The existing SQL Server schema and ASP reference pages serve as the specification for the new system.
3. **Domain Expert.** The client will provide a suitably qualified domain expert (veterinary reproductive medicine) at no charge to the project. This person must be available within 24 hours to answer workflow questions. Delays in domain clarification are a direct schedule risk.
4. **Confidentiality.** This document contains commercially sensitive information prepared specifically for the named client. It should not be disclosed to third parties without prior written consent of Atomate Limited.
5. **Liability.** Atomate Limited's liability is limited to the terms of any engagement agreement in place between the parties.

---

## What We Are Building

A cloud-hosted React SPA backed by a Node.js REST API, replacing all core workflows of the BD-DSK desktop application. The system will use a single deployable API service with a modular internal structure. Database: PostgreSQL migrated from SQL Server 2005. No sync adapter, no VB6 compatibility layer — the legacy system is decommissioned at go-live.

The platform includes a self-service **Access Database Import** feature, allowing clinic staff to upload a legacy `.mdb` file and import their historical data directly into the new system without developer involvement.

---

## Team Structure & Parallelisation Model

| Role | Focus | Months |
|---|---|---|
| **Dev 1 — Backend Lead** (Senior) | API design, PostgreSQL schema, business logic, data migration, auth, DevOps | 1–9 |
| **Dev 2 — Frontend Lead** (Mid-Senior) | React SPA, component library, all UI forms, report rendering, label printing | 1–9 |
| **Client Domain Expert** | Workflow validation, UAT, question turnaround ≤24h | Throughout |

The two developers work in parallel on each functional area: Dev 1 builds and tests the API layer while Dev 2 builds the corresponding UI. Integration occurs at the end of each phase. This model is the primary reason 9 months is achievable — it eliminates sequential frontend/backend bottlenecks.

---

## Scope — What Is Included (v1.0)

| Area | Status | Notes |
|---|---|---|
| **Auth** | ✅ | Email/password login; per-clinic data isolation (ported from `GUID_USER` model); password hashing migration |
| **Clients & Pets** | ✅ | Full CRUD, breed lookup, pedigree fields, photos, practice short-code URL |
| **Visits — Collection & Freezing** | ✅ | Full semen collection workflow; abnormalities; straw creation + label printing |
| **Visits — Chilling** | ✅ | Send-to details, insemination instructions |
| **Visits — Soundness** | ✅ | Testes measurements, notes; AKC Soundness sub-form deferred |
| **Cryogenic Storage** | ✅ | Full tank→canister→goblet→straw hierarchy; move suggestions; disposition |
| **Transfer Out** | ✅ | Shipping straws to other practices; straw assignment |
| **Transfer In** | ✅ | Receiving semen from external practices (was deferred in constrained estimate) |
| **Invoicing** | ✅ | Services, invoice creation, line items, payments; charge templates deferred |
| **Reports** | ✅ 12 reports | Storage, Freezing Application, Collection, Chilling, Transfer (In + Out), Invoice, Client, Pet, Soundness, Straw Labels, Practice Summary |
| **Label Printing** | ✅ | Integrated into Freezing workflow |
| **DB Migration** | ✅ | SQL Server 2005 → PostgreSQL; password hashing; data quality remediation |
| **Access Database Import** | ✅ | Self-service import of legacy `.mdb` files — see section below |
| **Backup / Restore** | ✅ | Automated DB snapshots; manual export |
| **English only** | ✅ | Multi-language infrastructure deferred → v2.0 |

### Deferred to v2.0

- OVT integration (ovulation timing charts, cycle grid) — largest single module; deferred as in previous estimates
- AKC sub-forms (Freezing / Chilling / Fresh) — regulatory documents; can be added as a separate sprint post-launch
- Pelleted semen workflow
- Medical history profiles (lab work, collection problems, breeding outcomes)
- Remaining reports (AKC reporting, storage map, statistical reports)
- Charge templates and discount groups
- Multi-language / i18n
- AI-assisted semen evaluation
- VMS integration
- Microservice decomposition (API will be modular monolith at launch)

---

## Access Database Import — Feature Detail

### Background

BD-DSK clinics hold their historical data in a local Microsoft Access `.mdb` file (Jet 4.0 format). This is the desktop cache database — the same schema as the server-authoritative SQL Server, sharing the same `ID_XX` primary key conventions. Clinics that never synced to the server, or that have local records not captured in the SQL Server migration, need a path to bring this data into the new system.

This feature provides a self-service import wizard in the web UI. No developer involvement is required after go-live. No business rules are changed — the import is purely a data ingestion path into the existing schema.

### Technical Approach

The server is Linux-hosted. Microsoft Access (Jet/ACE) is a Windows-only engine, so ODBC/ADODB are not available on the server. The import service will use **`mdb-tools`** — an open-source C library and CLI suite that reads Jet3/Jet4 `.mdb` files natively on Linux without any Windows dependency. A thin Node.js wrapper invokes the `mdb-export` and `mdb-tables` commands to extract table data as CSV/JSON, which is then transformed and inserted into PostgreSQL.

### What Is Imported

All tables that map directly to the new schema are imported. No new tables or fields are added — scope is limited to the existing data model.

| Source table (Access) | Target table (PostgreSQL) | Notes |
|---|---|---|
| CLIENTS | clients | ID_CL preserved; duplicate detection by ID or name+phone |
| PETS | pets | ID_PT preserved; linked to imported or existing client |
| VISITS | visits | ID_VI preserved; all collection/freezing/chilling/soundness fields |
| STORAGE | storage | ID_S preserved; straw records with parent hierarchy |
| TANKS / CANISTERS / GOBLETS | tanks / canisters / goblets | Storage hierarchy reconstructed |
| TRANSFERS | transfers | ID_TR preserved; straw links re-established |
| INVOICES + INVOICE_ITEMS | invoices + invoice_items | Financial history preserved |
| Photos (OLE Object) | S3 (extracted binary) | OLE wrapper stripped; raw image bytes uploaded to S3 |

USERS table is excluded — authentication is re-established in the new system; passwords are not migrated.

### Import Wizard — User Flow

1. **Upload** — clinic staff uploads a `.mdb` file (max 500 MB) via the web UI. File is stored temporarily in S3 for processing.
2. **Analyse** — the server extracts table row counts and detects potential conflicts (records whose IDs already exist in the target database). A summary is shown before any data is written.
3. **Options** — user selects conflict resolution strategy per entity type: *Skip duplicates* (default) / *Overwrite existing* / *Abort on conflict*.
4. **Import** — an async background job imports the data. A progress indicator shows records processed per table. The user can navigate away; the job continues.
5. **Results** — on completion, a summary shows records imported, skipped, and any rows that failed with the reason (e.g. broken foreign key, invalid date). The full error log is downloadable.

### Constraints & Assumptions

- The schema of the source `.mdb` must match the known BD-DSK schema. Heavily customised or third-party Access databases are out of scope.
- No business rules are modified — the import does not alter validation logic, calculated fields, or workflow state in the new application.
- The import is additive by default (skip duplicates). Bulk overwrites require explicit selection and are confirmed with a warning.
- Photo extraction handles the standard OLE Object wrapper used by VB6 image controls. Non-standard OLE embeddings (e.g. Word documents stored in image fields) are skipped with a logged warning.

### Effort Estimate

| Task | Owner | Est. days |
|---|---|---|
| `mdb-tools` integration; file upload + temp S3 handling | Dev 1 | 3 |
| Table extraction, type mapping, transformation layer (all tables) | Dev 1 | 4 |
| OLE Object photo extraction → S3 | Dev 1 | 1.5 |
| Async import job, progress tracking, error log | Dev 1 | 2.5 |
| Import API endpoints (upload, analyse, execute, status, results) | Dev 1 | 2 |
| Import wizard UI (all 5 steps, progress indicator, error report) | Dev 2 | 5 |
| Integration testing with `bd_demo_data.mdb` and a real clinic `.mdb` | Dev 1 | 2 |
| **Total** | | **~20 developer-days** |

20 developer-days across both developers ≈ 2 weeks of parallel effort. This is absorbed into month 8 (Dev 1 builds import backend in parallel with Dev 2 building the report UI), with integration and testing completing in the first week of month 9 before UAT begins.

---

## Delivery Plan — 9 Months

| Month | Backend (Dev 1) | Frontend (Dev 2) | Integration / Milestone |
|---|---|---|---|
| **1** | Cloud infra (dev + prod), PostgreSQL schema, CI/CD, Auth API | SPA scaffolding, design system, component library, Auth UI | Foundation complete; login working end-to-end |
| **2** | DB migration scripts, SQL Server → PostgreSQL; data quality fixes | Clients & Pets UI — list, search, create, edit, photos | Auth + Clients & Pets live in dev environment |
| **3** | Clients & Pets API — CRUD, breeds, pedigree; Pets detail | Visit (Collection + Freezing) UI — all form fields, straw creation | Clients & Pets feature-complete (dev) |
| **4** | Visits API — Collection, Freezing, Chilling, Soundness | Visit (Chilling + Soundness) UI; label printing integration | Visits feature-complete (dev); label print working |
| **5** | Storage API — hierarchy, tank populate, move/suggest, disposition | Storage UI — tank visualisation, straw tree, move workflow | Storage feature-complete (dev) |
| **6** | Transfer Out + Transfer In API; straw assignment logic | Transfer Out + In UI; straw picker, shipping details | Transfers feature-complete (dev) |
| **7** | Invoicing API — services, line items, payments | Invoicing UI — invoice builder, payment recording, service catalogue | Invoicing feature-complete (dev) |
| **8** | Reports API (data queries, PDF generation via Puppeteer/PDFKit); **Access Import** — mdb-tools integration, table extraction/transformation, async import job, API endpoints | Reports UI — report viewer, filters, print/download, header/logo; **Access Import wizard** — all 5 steps, progress, error report | Full report suite + Access Import feature-complete (dev) |
| **9** | Production deployment, monitoring, import integration tests with real `.mdb` file; final SQL Server data migration | UAT bug fixes; polish; handover documentation | **Go-live** |

> Month 9 is split: the first two weeks are UAT with the client domain expert; the final two weeks are production deployment and live data migration. Any critical bugs found in UAT are fixed within this month. Non-critical items are logged for v2.0.

---

## Budget Breakdown

### Labour

| Role | Monthly Rate | Months | Total |
|---|---|---|---|
| Dev 1 — Backend Lead (Senior) | $5,500 | 9 | $49,500 |
| Dev 2 — Frontend Lead (Mid-Senior) | $4,500 | 9 | $40,500 |
| DevOps assistance (months 1, 9) | ~$800/active month | ~2 | ~$1,600 |
| Access Import feature (absorbed in month 8, no calendar slip) | — | — | included above |
| **Total labour** | | | **~$91,600** |

> The client domain expert is provided at zero cost to this budget. This is a dependency, not an optional resource — see Risk D1 below.

### Cloud Infrastructure (additional — not in labour budget)

| Component | Monthly (build phase) | Monthly (production) |
|---|---|---|
| PostgreSQL (RDS — single AZ build, Multi-AZ prod) | $100–$180 | $180–$280 |
| App server (t3.small scaling to t3.medium at go-live) | $30–$60 | $60–$120 |
| S3 (photos, PDFs, backups) | $10–$20 | $15–$30 |
| CI/CD + monitoring (GitHub Actions, CloudWatch) | $30–$50 | $30–$50 |
| **Infra total** | **~$170–$310/month** | **~$285–$480/month** |

> Infrastructure costs over the 9-month build period: ~$1,500–$2,800. These are separate from and in addition to the labour budget.

### Ongoing Post-Launch (estimated)

| Item | Monthly |
|---|---|
| Developer maintenance (part-time retainer, one of the two devs) | $2,000–$2,500 |
| Cloud infrastructure | $285–$480 |
| **Total ongoing** | **~$2,300–$3,000/month** |

---

## Risks

### Critical risks

| # | Risk | Impact | Mitigation |
|---|---|---|---|
| **D1** | **Domain expert availability** — with no sync adapter and no legacy bridge, the domain expert is the only safety net against workflow mis-specification. If they are unavailable, wrong implementations will reach UAT. | **Critical** | Nominate a single named contact. Agree ≤24h response time in writing before project start. Block milestone sign-off on domain expert review at months 3, 5, 7, and 9. |
| **D2** | **No QA resource** — both developers self-test. Bugs will reach UAT. | **High** | 20% of each sprint allocated to testing. Client must commit 2 weeks of UAT time in month 9. Consider a short contract QA engagement (~$2,000–$3,000) before go-live. |
| **D3** | **Hard go-live (clean cutover)** — with no sync adapter, there is no parallel-run option. The live SQL Server data must be migrated and the legacy system decommissioned in a single cutover event. If the migration has data quality issues, go-live is blocked. | **High** | Run migration scripts against a copy of the production database from month 2 onwards. Perform at least two full dress rehearsals before month 9. Agree a rollback plan (keep SQL Server available for 30 days post-launch). |
| **D4** | **Two-developer coordination overhead** — frontend/backend parallelisation requires clear API contracts between the two developers. If these slip, integration points become blockers. | **Medium** | Define and version API contracts (OpenAPI spec) at the start of each phase before frontend development begins on that phase. Weekly 30-minute sync between the two developers. |

### Inherited risks

| # | Risk | Level | Notes |
|---|---|---|---|
| R1 | VB6 shared library source unavailable | High | Mitigated by SQL Server reference schema and ASP pages as specification |
| R3 | SQL Server data quality (orphaned GUIDs, constraint violations) | Medium | Run data quality scripts from month 2 — do not leave for month 9 |
| R10 | Plaintext passwords in USERS table | Confirmed | Hashing migration included in scope |

### Access Import risks

| # | Risk | Impact | Mitigation |
|---|---|---|---|
| **A1** | **Clinic `.mdb` files are heavily customised** — some clinics may have added fields, renamed tables, or used a different version of BD-DSK. The import tool targets the known schema only. | **Medium** | Test against `bd_demo_data.mdb` and at least one real clinic file in month 9. Log unrecognised tables/columns as warnings; do not abort the import. Document supported schema version in the UI. |
| **A2** | **OLE Object photos are non-standard** — VB6 stores images with an OLE wrapper header. Some installations may have stored non-image objects (PDFs, documents) in photo fields. | **Low** | Strip the OLE header and attempt to detect the file type (magic bytes). If not a recognised image format, skip and log. |
| **A3** | **Large `.mdb` files (>100 MB)** — very active clinics may have large databases. Extraction and import may take several minutes. | **Low** | Import runs as a background async job with a progress indicator. Set a 500 MB upload limit. Test with the largest available `.mdb` sample before go-live. |

---

## Comparison to Previous Estimates

| Dimension | Full estimate (March 2026) | Constrained estimate (April 2026) | **This estimate (April 2026)** |
|---|---|---|---|
| Team | 8 people | 1 developer + occasional | **2 developers + client domain expert** |
| Duration | 10–12 months | 18 months | **9 months** |
| Total labour | ~$223k–$245k | ~$90k | **~$91,600** |
| Sync adapter | Included | Included | **Excluded** |
| Transfer In | Included | Deferred | **Included** |
| Reports | 20 | 8 core | **12** |
| OVT | Phase 2 | Deferred | **Deferred** |
| Architecture | Microservices | Modular monolith | **Modular monolith** |
| Risk level | Medium–High | High | **Medium–High** |
| Cutover model | Parallel run via sync | Parallel run via sync | **Hard cutover** |

---

## Key Decisions Before Starting

1. **Domain expert commitment.** Who is the nominated contact? What is their availability guarantee? This is a hard dependency, not an advisory role — block clause should be in the contract.
2. **Hard cutover acceptable?** The removal of sync means there is no parallel-run fallback. Confirm the client is prepared for a single cutover event with a 30-day rollback window.
3. **OVT deferral acceptable?** As in previous estimates, OVT is the most complex module. Deferring it is the single largest scope saving. Confirm this is acceptable for v1.0.
4. **AKC sub-forms.** If clinics require these at launch, add approximately 6–8 weeks of additional effort and revise the plan accordingly.
5. **Access Import — sample files.** Provide at least one real clinic `.mdb` file (anonymised if needed) for testing in month 9. Testing against `bd_demo_data.mdb` alone is insufficient; a real file will surface data quality issues the demo does not contain.
6. **Contract terms.** Agree IP ownership, notice period, handover obligations, and client-controlled repository access before start.
7. **QA engagement.** Confirm whether a 2-week contract QA engagement (~$2,500) will be budgeted before go-live, or whether client UAT is the only quality gate.
