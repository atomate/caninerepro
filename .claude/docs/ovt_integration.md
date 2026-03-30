# OVT Integration — BD-DSK

## What is OVT?

OVT ("Ovulation Timing") is a companion application for tracking a female dog's reproductive cycle — measuring daily progesterone (ng/mL), LH test results, and vaginal cytology to predict the optimal insemination window. It is a separate product running on multiple platforms: Desktop (Windows), PocketPC, Palm, and Web.

BD-DSK is the semen banking / reproductive services application. OVT is the cycle-tracking / insemination timing application. They share a database and sync data between platforms.

## Shared Codebase

The `ovt/` subdirectory contains three modules that are compiled into **both** BD-DSK and the OVT platform builds:

| File | Purpose |
|---|---|
| [project/ovt/ovt_common.bas](../../project/ovt/ovt_common.bas) | Sync types, ID generation, null/bool/number helpers, ComboBox helpers, breed processing |
| [project/ovt/ovt_settings.bas](../../project/ovt/ovt_settings.bas) | Host config, device ID init, ADO recordset factory |
| [project/ovt/ovt_xml.bas](../../project/ovt/ovt_xml.bas) | XML serialise/deserialise for OVT records (`rec2xml`, `xml2rec`) |

Constraint: these modules must compile under **both VB6 and eVB 3.0** (PocketPC). No `Optional` parameters, no `IIf` in certain contexts, explicit `ByRef`/`ByVal` — see header comment in [project/ovt/ovt_common.bas:2](../../project/ovt/ovt_common.bas#L2).

## Shared Database Tables

OVT data lives in the **same `bd.mdb`** alongside the breeding-data tables:

| Table | Content |
|---|---|
| `FORMS` | One row per OVT patient chart (linked to a `PETS` record via `ID_PT`) |
| `DAYS` | One row per day of cycle readings (progesterone, LH, cytology, WBC, RBC) |
| `INSEMS` | Insemination events within a chart |
| `SYNC` | Sync-age counter per device per FORM record |
| `_BREED` / `_BREED_GROUP` | Shared breed lookup |

## OVT Forms in BD-DSK

BD-DSK hosts OVT UI via forms prefixed `Ovt_` and the launcher module `mdlOvtFunc.bas`:

| Form / Sub | Access point |
|---|---|
| `Ovt_frDue` | Who's due (upcoming ovulations) |
| `Ovt_frStat` | Statistics: Pregnancy Probability, Pregnancy Rates, Whelping |
| `Ovt_frSync` | Sync dialog (Web / PocketPC / Palm) |
| `Ovt_frAccount` | Account info entry |
| `Ovt_frSyncPalm` | Palm-specific sync dialog |
| `OvtList` UserControl | Embeds OVT chart list within BD-DSK views |

These forms are launched from `mdlOvtFunc.bas` — see [project/mdlOvtFunc.bas](../../project/mdlOvtFunc.bas). They are separate from the BD-DSK `fr*` forms but run in the same process and share the same `db` connection.

## OVT Open-Window Limit

`OVT_OPEN_LIMIT = 10` — if more than 10 OVT windows are open simultaneously, a warning is shown. Set in [project/ovt/ovt_settings.bas:15](../../project/ovt/ovt_settings.bas#L15).

## OVT App Path

`OVT_APP_PATH = "..\ovt-dsk\"` — relative path to the companion OVT Desktop application directory. Used when shelling out to OVT-specific executables. See [project/ovt/ovt_settings.bas:25](../../project/ovt/ovt_settings.bas#L25).

## OVT Host

`OVT_HOST = "ovtvet.com"` — the web server hostname for OVT sync and online help. Set in [project/ovt/ovt_settings.bas:10](../../project/ovt/ovt_settings.bas#L10). Note that `REPRO_HOST` (used by `clsReproServer`) is a separate constant for the BD-DSK web server.

## Sync Protocol (OVT ↔ Web/Mobile)

The sync protocol uses XML serialisation in `ovt_xml.bas`. The key functions:

- `rec2xml(id_f)` — serialises one OVT chart (FORMS + INSEMS + DAYS) to XML string
- `xml2rec(strSource)` — deserialises XML back into FORMS/INSEMS/DAYS rows (upsert logic)
- `sync2xml(id_dvc, id_f)` — serialises the SYNC age record for a device/chart pair
- `xml2sync(strSource)` — updates SYNC table from received XML
- `UpdateSync(id_dvc, id_f, age, increment)` / `GetSyncAge(id_dvc, id_f)` — manage sync counters in `ovt_common.bas`

The SYNC table tracks an `AGE` counter per (`ID_DVC`, `ID_F`) pair. When a record is modified, its age increments. During sync, ages are compared between devices to determine which records are new, updated, or unchanged.

## Device Identity

Each device (Desktop, PocketPC, Palm, Web) has a unique `ID_DVC`. For BD-DSK Desktop, it is generated once on first launch:

```vb
ID_DVC = get_id(12, 15) * 10 + SYNC_DSK   ' SYNC_DSK = 2
SetVar "ID_DVC", ID_DVC
```

See [project/ovt/ovt_settings.bas:17](../../project/ovt/ovt_settings.bas#L17). The low digit encodes the device type (`SYNC_DSK=2`), which means all Desktop-generated IDs end in `2`.

## Unit Conversion

OVT uses both ng/mL and nmol/L for progesterone. Conversion functions in `ovt_common.bas`:
- `ng2nmol(value)` — multiply by 3.18
- `nmol2ng(value)` — divide by 3.18

## Link Between OVT and BD-DSK

An OVT patient chart (`FORMS.ID_PT`) maps to a BD-DSK pet (`PETS.ID_PT`). This is the join that lets BD-DSK display OVT charts for a selected pet, and lets OVT display semen/insemination history from BD-DSK.

`frOVTInseminations` and `frDisposition` allow linking a BD-DSK straw disposition to an OVT insemination record (`INSEMS`) — connecting the semen-banking side to the cycle-tracking side.
