# Database Schema — BD-DSK

Source of truth is `bd.mdb` (Microsoft Access). Schema is derived from [project/bdTables.cls](../../project/bdTables.cls), [project/DataEnv.DCA](../../project/DataEnv.DCA) (binary, partially readable), form bindings, and query patterns in [project/mdlReports.bas](../../project/mdlReports.bas).

## Key Conventions

- All primary keys named `ID_XX` are GUIDs (generated via `GetGUID`) — see [project/bdTables.cls:127](../../project/bdTables.cls#L127)
- Each table also has a human-readable AutoID field (e.g. "Client ID", "Visit ID") generated sequentially by `AutoID(tableName, fieldName)`
- `AutoID` for invoices uses a prefix: `"INV-001"`, for doctors: `"DOC-001"`, for label formats: `"Label Format 001"`

---

## Core Domain Tables

### CLIENTS
| Field | Notes |
|---|---|
| `ID_CL` | GUID PK |
| `Client ID` | Human-readable AutoID |
| `First Name`, `Last Name` | |
| `Address`, `City`, `State`, `Zip Code`, `Country` | |
| `Home Phone`, `Work Phone`, `Cell Phone` | |
| `Email`, `SSN` | |
| `Card Number`, `Card Type` | |
| `Card Expires Month`, `Card Expires Year` | Stored as separate fields |
| `Notes` | Memo |
| `Associate Client` | FK → CLIENTS.ID_CL (linked account) |

### PETS
| Field | Notes |
|---|---|
| `ID_PT` | GUID PK |
| `Pet ID` | Human-readable AutoID |
| `ID_CL` | FK → CLIENTS |
| `Pet Name`, `Species` | Species defaults to "Canine" on new |
| `Breed ID` | FK → `_BREED.ID_B` |
| `Sex` | |
| `DOB` | Date of birth |
| `Color`, `Eye Color` | |
| `Weight`, `Height` | |
| `Stud ID` | Registration stud ID (separate from `Pet ID`) |
| `Registered Name`, `Registration Number` | |
| `Sire's Registered Name`, `Sire's Registration Number` | |
| `Dam's Name`, `Dam's Registration Number` | |
| `DNA`, `Chip Number`, `Tattoo` | |
| `Notes` | Memo |
| `Picture ID` | FK → PICTURES |

### VISITS
| Field | Notes |
|---|---|
| `ID_VI` | GUID PK |
| `Visit ID` | Human-readable AutoID |
| `ID_PT` | FK → PETS |
| `Visit Date` | |
| `Collection Time` | Dropdown value (8am–10pm) |
| `Semen Owner` | Defaults to client |
| `Soundness`, `Freezing`, `Collection` | Bool flags — used to compute "Reason" display |
| `Semen Volume`, `Concentration`, `Motility` | Numeric semen parameters |
| `Photo Front`, `Photo Left`, `Photo Right` | File paths (not blobs) |
| `Send To Name`, `Send To Address`, `Send To City`, `Send To State`, `Send To Zip` | Chilling destination (split from old "Send To" memo) |
| `Testes Width R/L`, `Testes Length R/L` | Soundness fields |
| `Comments` | Memo — use `CanGrow=True` on reports |

### STORAGE
| Field | Notes |
|---|---|
| `ID_ST` | GUID PK |
| `Parent` | FK → STORAGE.ID_ST (parent node). `NULL` = out of storage |
| `ID_VI` | FK → VISITS (cascade delete) |
| `ID_TR` | FK → TRANSFERS (no cascade — set NULL before deleting transfer) |
| `Label` | Straw label text (this is the user-visible "Straw ID" on reports, not `ID_ST`) |
| `LabelNum` | Numeric sort order for labels |
| `Storage` | Integer level: 1=Tank, 2=Canister, 3=Goblet, 4=Straw |
| `Last Location` | Previous goblet path — updated on every move |
| `Disposition Date` | Set when straw is used/disposed |

### TRANSFERS
| Field | Notes |
|---|---|
| `ID_TR` | GUID PK |
| `Transfer ID` | Human-readable AutoID |
| `Date` | |
| `Transfer Date` | |
| `Purpose Of Transfer` | Dropdown |
| `To Name`, `To Hospital`, `To Address`, `To City`, `To State`, `To Zip`, `To Phone` | Destination |
| `Client State` | Client's state at time of transfer |
| `Agent Print Name`, `Agent City`, `Agent Zip` | Shipping agent |
| `Bitch Breed ID` | FK → `_BREED` |

### INSEMINATIONS
| Field | Notes |
|---|---|
| `ID_IN` | GUID PK |
| `Insemination ID` | Human-readable AutoID |

### AKC_FREEZINGS / AKC_CHILLINGS / AKC_FRESH
Each is a 1:1 extension of a VISIT record for AKC registration data.
| Field | Notes |
|---|---|
| `ID_AKCFR` / `ID_AKCCH` / `ID_AKCFH` | GUID PK |

---

## Billing Tables

### SERVICES
`ID_SE`, `Service ID` (AutoID), `Service Description`, `Price`

### DISCOUNT_GROUPS
`ID_DG`, `Discount Group` (AutoID)

### INVOICES
`ID_INV`, `Invoice ID` (AutoID prefix "INV"), `Invoice Date` (defaults to today), `ID_CL`

### CHARGES (invoice line items)
`ID_CH`, `Date1`, `Date2`, `ID_INV` FK, `ID_SE` FK

### CHARGE_TEMPLATES (default charges per service type)
`ID_CHT`, `QuantityMultiply` (defaults to "pet")

### PAYMENTS
`ID_PAY`, `Payment Date` (defaults to today), `ID_INV` FK

### DOCTORS
`ID_DO`, `Doctor ID` (AutoID prefix "DOC")

---

## Configuration / Lookup Tables

### LABEL_FORMATS
`ID_LABELFORMAT`, `LabelFormat` (AutoID prefix "Label Format "), `LabelTemplate`

### USER_LISTS
`ID_DT`, data field, `UserDefined` (bool), `Type` — drives the editable dropdowns in frReportSetup. `UserDefined=False` rows are read-only (system defaults).

### ABNORMALITIES
`ID_AB` — abnormality definitions (Head / Midpiece / Tail categories)

### PICTURES
`ID_PI`, `TITLE`, `Priority` (bool)

### _BREED / _BREED_GROUP
`_BREED`: `ID_B`, `ID_BG`, `BREED`, `BREED_ABREV`
`_BREED_GROUP`: `ID_BG`, `BREED_GROUP`

### ListStates / ListStorage
Read-only lookup tables. `ListStorage` drives the storage hierarchy level names (Tank, Canister, Goblet, Straw). `ListStates` drives US state dropdowns.

---

## OVT Tables (Ovulation Timing subsystem)

### FORMS (OVT patient charts)
`ID_F`, `TITLE`, `NAME`, `ID_B` FK breed, `DOB`, `DHS` (date heat started), `ID_PA`, `ID_LA`, `PREGNANCY`, `AWD` (actual whelping date), `PUPS_M/F_0/1/2` (pup counts by litter), `COMMENTS`, `CREATED`, `COMPLETE` bool, `ID_PT` FK pets

### INSEMS (OVT insemination records per chart)
`ID_F` FK, `NINSEM` (index), `INSEM_DATE`, `ID_TS`, `ID_TI`, `S_VOL`, `CONC`, `N_STRAW`, `MOTILITY`, `ID_Q`

### DAYS (OVT daily readings grid)
`ID_F` FK, `NDAY`, `PROG_NG` (progesterone ng/mL), `LH` (LH test result), `VC_P/I/S` (vaginal cytology), `RBC`, `WBC`, `COMMENTS`

### SYNC
`ID_DVC`, `ID_F`, `AGE` — tracks sync state per device per OVT chart

---

## Temporary Print-Queue Tables

These are populated immediately before printing and cleared immediately after:

| Table | Fields | Used by |
|---|---|---|
| `InvoicesPrint` | `PrintID`, `ID_INV` | `ShowInvoiceReport`, `InvoicesPrint` — see [project/mdlReports.bas:564](../../project/mdlReports.bas#L564) |
| `StoragePrint` | `PrintID`, `ID_CL` | `StorageReportPrint` — see [project/mdlReports.bas:617](../../project/mdlReports.bas#L617) |

---

## Relationships Summary

```
CLIENTS ──< PETS ──< VISITS ──< STORAGE (cascade delete)
                            └── AKC_FREEZINGS (1:1)
                            └── AKC_CHILLINGS (1:1)
                            └── AKC_FRESH     (1:1)
PETS ──< FORMS (OVT) ──< INSEMS
                      └── DAYS
TRANSFERS ──< STORAGE (ID_TR, no cascade — must NULL manually before delete)
SERVICES ──< CHARGES ──> INVOICES ──< PAYMENTS
DISCOUNT_GROUPS ── CLIENTS
_BREED_GROUP ──< _BREED ── PETS
```
