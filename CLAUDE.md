# BD-DSK — Breeding Data Desktop

## Project Overview

Legacy VB6 desktop application for managing canine reproductive services at veterinary clinics. Core workflows: client/pet records, semen collection visits, cryogenic storage (tanks → canisters → goblets → straws), transfers/dispositions, invoicing, and report generation. Syncs with a web server (`ReproServer.asp`) and mobile devices (PocketPC, Palm) via the OVT subsystem.

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Visual Basic 6.0 |
| Database | Microsoft Access (`.mdb`) via ADO / Jet OLE DB 4.0 |
| HTTP/XML | MSXML2 (`XMLHTTP`, `DOMDocument`) |
| UI Controls | MSComctl.ocx (ListView, TreeView), custom UserControls |
| Reports | VB DataReports (`.DCA`, `.Dsr`, `.dsx`) |
| Win APIs | kernel32, gdi32, comdlg32, shell32 |

## Key Directories & Files

```
project/
  *.frm / *.frx     — Forms (UI + binary resources)
  *.cls             — Classes (business objects, data layer)
  *.bas             — Modules (utilities, Win API declarations)
  *.ctl / *.ctx     — UserControls (reusable UI components + resources)
  *.DCA / *.Dsr / *.dsx — DataReports and DataEnvironment
  DataEnv.DCA       — ADO connection string + all parameterized queries
  bdTables.cls      — Central data-access object; holds all table/query instances
  clsReproServer.cls — HTTP client for server sync (POST XML to ReproServer.asp)
  mdlReports.bas    — Report utilities + Win API declarations (GDI, file dialogs)
  mdSpecialFolder.bas — Windows CSIDL constants for special folder resolution
  ovt/              — OVT (Ovulation Timing) subsystem; shared with PocketPC/Palm builds
    ovt_common.bas  — Shared utilities (ID generation, unit conversion, null helpers)
    ovt_settings.bas — App settings, device ID init, ADO recordset factory
    ovt_xml.bas     — XML serialization/deserialization helpers
  demo_data/bd_demo_data.mdb — Demo Access database
```

## Build & Run

There are no automated build scripts. This is a VB6 project:

- **IDE**: Visual Basic 6.0 (Windows only)
- **Open**: Load `bd-dsk.vbp` (referenced in [project/CR_BD-DSK.CRB](project/CR_BD-DSK.CRB:3)) in the VB6 IDE
- **Database**: Point the connection string in [DataEnv.DCA](project/DataEnv.DCA) to a local `bd.mdb`; demo data at [project/demo_data/bd_demo_data.mdb](project/demo_data/bd_demo_data.mdb)
- **Dependencies**: `mscomctl.ocx`, `ceutil.dll`, `rapi.dll`, `zlib.dll` must be registered/present
- **Code review tool**: [project/CR_BD-DSK.CRB](project/CR_BD-DSK.CRB) is a VBCompiler 6 project (static analysis)

There are no automated tests.

## Database Schema Conventions

- Primary keys follow `ID_XX` pattern: `ID_CL` (clients), `ID_PT` (pets), `ID_VI` (visits/freezings), `ID_S` (straws), `ID_TR` (transfers)
- `Parent IS NULL` means a straw is out of storage (see [project/notes.txt:41-48](project/notes.txt#L41-L48))
- Cascade-delete: VISITS→STORAGE (straws deleted with visit), TRANSFERS→STORAGE (no cascade; set `ID_TR=NULL` before deleting a transfer)
- Key/value persistence: `GetVar`/`SetVar` helpers for registry-backed app settings; `GetMemo`/`SetMemo` for longer text (e.g. insemination instructions)

## Additional Documentation

| Topic | File |
|---|---|
| Architecture, design patterns, conventions | [.claude/docs/architectural_patterns.md](.claude/docs/architectural_patterns.md) |
| Database schema (all tables, fields, relationships) | [.claude/docs/database_schema.md](.claude/docs/database_schema.md) |
| Custom controls & shared library API | [.claude/docs/custom_controls.md](.claude/docs/custom_controls.md) |
| Environment setup (VB6, OCX registration, DB connection) | [.claude/docs/environment_setup.md](.claude/docs/environment_setup.md) |
| OVT integration (shared tables, sync protocol, device ID) | [.claude/docs/ovt_integration.md](.claude/docs/ovt_integration.md) |
| Known issues & tech debt | [.claude/docs/known_issues.md](.claude/docs/known_issues.md) |
| Deployment & distribution | [.claude/docs/deployment.md](.claude/docs/deployment.md) |
| Workflow: Client & Pet management | [.claude/docs/workflows/workflow_client_pet.md](.claude/docs/workflows/workflow_client_pet.md) |
| Workflow: Semen collection visit (tabs: Collection, Freezing, Chilling, Soundness) | [.claude/docs/workflows/workflow_visit_semen.md](.claude/docs/workflows/workflow_visit_semen.md) |
| Workflow: Cryogenic storage (tanks, goblets, straws, suggest/move) | [.claude/docs/workflows/workflow_storage.md](.claude/docs/workflows/workflow_storage.md) |
| Workflow: Straw transfers & disposition | [.claude/docs/workflows/workflow_transfer_disposition.md](.claude/docs/workflows/workflow_transfer_disposition.md) |
| Workflow: Invoicing, services, payments, discounts | [.claude/docs/workflows/workflow_invoicing.md](.claude/docs/workflows/workflow_invoicing.md) |
| Workflow: Report generation & print setup | [.claude/docs/workflows/workflow_reports.md](.claude/docs/workflows/workflow_reports.md) |
| Workflow: Database sync & app updates | [.claude/docs/workflows/workflow_sync.md](.claude/docs/workflows/workflow_sync.md) |
| Workflow: App settings & configuration | [.claude/docs/workflows/workflow_settings.md](.claude/docs/workflows/workflow_settings.md) |
| Workflow: Straw label printing | [.claude/docs/workflows/workflow_label_printing.md](.claude/docs/workflows/workflow_label_printing.md) |
| Workflow: AKC registration reporting | [.claude/docs/workflows/workflow_akc_reporting.md](.claude/docs/workflows/workflow_akc_reporting.md) |
