# Workflow: AKC Registration Reporting

## Background

The American Kennel Club (AKC) requires specific documentation when frozen or chilled semen is collected, stored, or used for artificial insemination. BD-DSK captures AKC-specific data fields alongside each visit and generates the corresponding AKC forms.

## Tables Involved

Each AKC table is a 1:1 extension of a `VISITS` record, holding fields that appear only on AKC documents:

| Table | PK | Linked to |
|---|---|---|
| `AKC_FREEZINGS` | `ID_AKCFR` | Frozen semen visit |
| `AKC_CHILLINGS` | `ID_AKCCH` | Chilled semen visit |
| `AKC_FRESH` | `ID_AKCFH` | Fresh semen / soundness visit |

These are initialised in [project/bdTables.cls:76](../../../project/bdTables.cls#L76) and each raises `AddNewBeforeUpdate` / `AddNewAfterUpdate` / `AfterDelete` events like all other tables.

## Data Entry

AKC fields are entered on the relevant tab of `frEditVisit`:

- **Freezing tab** → writes to `AKC_FREEZINGS`
- **Chilling tab** → writes to `AKC_CHILLINGS`
- **Soundness / Fresh tab** → writes to `AKC_FRESH`

The `Soundness`, `Freezing`, `Collection` boolean flags on the `VISITS` record determine which tabs are active and which AKC sub-record is applicable.

## Reports

| Report | File | AKC document |
|---|---|---|
| Freezing Application | `rpFreezingApplication.DCA` / `rpFrzSemProgApp.dsx` | AKC Semen Freezing Application |
| Freezing Storage | `rpFreezingStorage.DCA` / `rpFrzSemStor.DCA` | AKC Frozen Semen Storage Certificate |
| Chilling | `rpChilling.dsx` | AKC Chilled Semen document |
| Soundness Evaluation | `rpSoundnessEval.DCA` / `rpSoundnessEvaluation.DCA` | AKC Soundness Evaluation |
| Collection Report | `rpCollectionReport.dsx` | Semen collection record |

## Report Generation

All AKC reports follow the same pattern in [project/mdlReports.bas](../../../project/mdlReports.bas):

1. Call `TranslateReportControls` to apply active language
2. Call `ReportSetup(rpSetupPrev)` + `CopyReportHeader` to stamp clinic header + logo
3. Open `DataEnv.VisitInfo(id_vi)` — a parameterised query joining VISITS, PETS, CLIENTS
4. Compute any derived fields (age, breed name, total motile sperm, testes area)
5. Show report modal

### Key computed fields

**Total Motile Sperm** (Freezing and Chilling reports):
```
TSperm = (Semen Volume × Concentration × Motility / 100)
```
Displayed as "X.XX mln" if > 0, blank otherwise — [project/mdlReports.bas:287](../../../project/mdlReports.bas#L287).

**Testes area** (Soundness report):
```
Area R = Testes Width R × Testes Length R
Area L = Testes Width L × Testes Length L
```
Blank if either dimension is NULL — [project/mdlReports.bas:448](../../../project/mdlReports.bas#L448).

**Stud ID visibility**: Hidden on all reports when `Sex = "F"` — [project/mdlReports.bas:403](../../../project/mdlReports.bas#L403).

## Freezing Storage Report: SHAPE Query

The Freezing Storage report uses an ADO SHAPE (hierarchical recordset) command to join the visit with its straws in a single recordset — [project/mdlReports.bas:339](../../../project/mdlReports.bas#L339):

```sql
SHAPE {SELECT * FROM Q_Visits WHERE ID_VI = Param_ID_VI} AS VisitInfo
APPEND (
  (SHAPE {SELECT * FROM Storage} AS StorageInfo
   APPEND NEW adInteger RecIdx, NEW adChar(255) StorLoc)
  AS StorageInfo RELATE 'ID_VI' TO 'ID_VI'
) AS StorageInfo
```

After opening, the code iterates each straw row and computes `StorLoc = dbStorageLocation(ID_ST)` for display. The total straw count is shown in the report footer.

## Insemination Instructions (Chilling Report)

The Chilling report includes insemination instructions at the bottom of the page. These are stored via `GetMemo("RS_InsemInstr", "")` and configured in `frReportSetup` — see [project/mdlReports.bas:285](../../../project/mdlReports.bas#L285).

## Transfer Report: Thawing Instructions

The Transfer report optionally prints thawing instructions on a second page. The `PrintInstructions` parameter controls this:
- `True` → append `RS_ThawingInstr` memo with a page break before it
- `False` → hide the thawing instructions section entirely
See [project/mdlReports.bas:540](../../../project/mdlReports.bas#L540).
