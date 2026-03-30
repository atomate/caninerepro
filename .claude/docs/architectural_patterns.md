# Architectural Patterns — BD-DSK

## 1. MDI Application Shell

The application uses a Multiple Document Interface. `frMDI` is the top-level shell; every view form declares `MDIChild = -1` (e.g. [project/frVisits.frm:9](../../project/frVisits.frm#L9)). Open OVT sub-windows follow the same pattern via `mdlOvtFunc.bas` loader stubs.

## 2. Central Data-Access Object (`bdTables`)

All ADO table/query instances live on a single `bdTables` class ([project/bdTables.cls](../../project/bdTables.cls)). Every table is declared `Public WithEvents`, which lets forms subscribe to record-change events without polling. The DataEnvironment (`DataEnv.DCA`) owns the ADO connection and defines all parameterized queries; `bdTables` wraps those as typed `DbTable` instances.

**Pattern**: forms receive a `bdTables` reference and access `bdTables.tiClients`, `bdTables.tiVisits`, etc. — no form issues raw SQL directly.

## 3. DataEnvironment + Parameterized Queries

All database queries are defined in [project/DataEnv.DCA](../../project/DataEnv.DCA) using the VB6 DataEnvironment designer. Queries accept parameters (`ParamID_CL`, `ParamID_PT`, …) so forms switch context by setting the parameter and re-opening the recordset, rather than constructing SQL strings at runtime.

## 4. Custom UserControls as Reusable Components

Reusable UI behaviors are packaged as UserControls (`.ctl`):

- `StorageTree.ctl` — TreeView with Add/Delete toolbar for the storage hierarchy
- `DBListEdit.ctl` — ListView with inline editing and type selector
- `ToolButton` — Custom icon button used on every toolbar across all forms ([project/frVisits.frm:27-80](../../project/frVisits.frm#L27-L80))

Each control exposes a typed public API and raises events, keeping form code thin.

## 5. Data Transfer Objects (DTOs)

Simple `.cls` classes with only public fields — no methods — are used to pass structured data between layers:

- `bdStorageNode.cls` ([project/bdStorageNode.cls](../../project/bdStorageNode.cls)) — carries storage tree node data (label, parent/child IDs, freezing info)
- `DelItemNode.cls` ([project/DelItemNode.cls](../../project/DelItemNode.cls)) — describes a cascaded-delete operation (table, key field, child collections)

## 6. Module-Based Utility Libraries

Shared logic lives in `.bas` modules (no classes, no state beyond module-level globals):

- `mdlReports.bas` — Win API declarations (GDI, file dialog) + image-loading helpers
- `mdSpecialFolder.bas` — CSIDL constants + SHGetSpecialFolderPath wrapper
- `ovt/ovt_common.bas` — Null coalescing, unit conversion, custom ID generation
- `ovt/ovt_xml.bas` — XML tag read/write with escaping

## 7. OVT Cross-Platform Compatibility Layer (`ovt/`)

The three modules under `ovt/` are intentionally constrained to syntax valid in both VB6 and eVB 3.0 (PocketPC). Comment at [project/ovt/ovt_common.bas:2](../../project/ovt/ovt_common.bas#L2):

> "This module has to work in VB6 and eVB 3.0. That's why we avoid using Optional parameters and other incompatible syntax."

Implications: no `Optional` params, no `IIf` in some contexts, explicit `ByRef`/`ByVal` everywhere.

## 8. HTTP/XML Server Sync

`clsReproServer.cls` ([project/clsReproServer.cls](../../project/clsReproServer.cls)) wraps `MSXML2.XMLHTTP` for synchronous POST requests to `ReproServer.asp`. The class raises `Report` and `failed` events so callers (`frReproServer.frm`) can update the ListView log without blocking.

- Cache-busting: URL always appended with `?r=<random>` ([project/clsReproServer.cls:31](../../project/clsReproServer.cls#L31))
- Error handling: checks both HTTP status and XML response for an `<ERROR>` root element

Sync targets: Web, Desktop, PocketPC, Palm — keyed by constants in [project/ovt/ovt_common.bas:5-8](../../project/ovt/ovt_common.bas#L5-L8).

## 9. Distributed ID Generation

IDs are generated client-side to avoid collisions across sync targets. Algorithm in [project/ovt/ovt_common.bas:51-56](../../project/ovt/ovt_common.bas#L51-L56):

```
ID = (seconds_since_2000 mod 2^bits_time) * 2^bits_rnd + random(0..2^bits_rnd)
```

Device type is embedded in the low bits of `ID_DVC` (e.g., `SYNC_DSK = 2`), set once at first launch and stored in the registry via `GetVar`/`SetVar`.

## 10. Naming Conventions

| Prefix | Type | Example |
|---|---|---|
| `fr` | Form | `frVisits`, `frClients` |
| `md` / `mdl` | Module | `mdSpecialFolder`, `mdlReports` |
| `cls` | Class (explicit) | `clsReproServer` |
| `bd` | Business-domain class | `bdTables`, `bdStorageNode` |
| `rp` | Report | `rpInvoice`, `rpStorage` |
| `bt` | Button control | `btAdd`, `btDelete` |
| `ls` | ListView | `ls` (consistent across all forms) |
| `cb` | ComboBox | `cbOperation`, `cbChoseType` |
| `fmXxx` | Frame container | `fmForm`, `fmToolBox` |
| `ID_XX` | Database primary key | `ID_CL`, `ID_PT`, `ID_VI`, `ID_S`, `ID_TR` |
