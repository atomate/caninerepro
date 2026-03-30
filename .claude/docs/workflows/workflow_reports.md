# Workflow: Report Generation & Print Setup

## Forms Involved

| Form | Role |
|---|---|
| `frReportSetup` | Settings for report header, logo, and configurable lists |
| `frPrint` | Modal — report preview / print with logo scaling |
| `mdlReports.bas` | Utility module — Win API image loading, file dialog, GDI |

## Report Files

| File | Report |
|---|---|
| `rpClient.DCA` | Client record |
| `rpPet.dsx` | Pet record |
| `rpVisit.Dsr` | Visit summary |
| `rpCollectionReport.dsx` | Semen collection report |
| `rpChilling.dsx` | Chilling semen report |
| `rpFrzSemProgApp.dsx` | Freezing semen — progressive application |
| `rpFrzSemStor.DCA` | Freezing semen — storage |
| `rpFreezingApplication.DCA` | Freezing application |
| `rpFreezingStorage.DCA` | Freezing storage |
| `rpSoundnessEval.DCA` | Soundness evaluation |
| `rpSoundnessEvaluation.DCA` | Soundness evaluation (alternate) |
| `rpStorage.DCA` | Storage map |
| `rpStorageMap.Dsr` | Storage map (detailed) |
| `rpInvoice.DCA` | Invoice |
| `rpInvoiceList.Dsr` | Invoice list |
| `rpTransfer` | Transfer document |

## frReportSetup Tab Structure

`frReportSetup` is a tabbed dialog (`oeTabsFrames` extender). Tabs include:

1. **Hospital / Report Header** — clinic name, address, phone, email, website, logo
   - Variables stored as `RS_Address`, `RS_Email`, etc. (via `SetVar`/`GetVar`)
   - Logo stored by path; loaded via `LoadImg` in `mdlReports.bas`
2. **Insemination Instructions** — stored via `GetMemo`/`SetMemo` with key `RS_INSEMINATION_INSTRUCTIONS`; appears at bottom of Chilling report
3. **Configurable Lists** (each uses `DBListEdit` UserControl):
   - Head abnormalities, Midpiece abnormalities, Tail abnormalities
   - Extender types

Changes saved via OK; Cancel discards. Save/Cancel buttons follow app-wide style (same size as all other forms).

## Logo Rules

- Must resize preserving aspect ratio — if reserved area is 100×100 and image is 200×100, resize to 100×50
- Do not save logo path until `dbfP_AfterSave` is triggered; set `dbfP.Modified=True` if user selects/clears
- If user selects a new logo and clicks Preview, the new logo must appear in the preview — not the saved one

## Report Generation Flow

```
User selects report from menu / button
  → mdlReports.bas PrintXxx() sub called
  → DataEnvironment query opened with relevant parameters (ID_CL, ID_PT, ID_VI, …)
  → DataReport .Show or .PrintReport
  → frPrint handles logo overlay (PictureBox + GDI StretchBlt for aspect-ratio scale)
```

## Image Loading (mdlReports.bas)

`LoadImg` ([project/mdlReports.bas:50](../../project/mdlReports.bas#L50)):
- Opens `GetOpenFileName` (comdlg32) file dialog; remembers last folder in `GetVar("PATH_PICTURES")`
- Loads image into a `StdPicture`
- Calculates scale to fit target `PictureBox` while preserving aspect ratio
- Uses GDI: `CreateCompatibleDC`, `StretchBlt`, `SetStretchBltMode`

## Report Design Conventions (from notes.txt)

- No double spacing — make reports compact
- Remove "Address" label from header
- Do not print N/A for missing fields — leave blank
- `CanGrow=True` on memo fields (e.g. Comments on Freezing tab) to handle variable-length text
- Logo region: 2× height reservation
- Straw ID in Storage Report = straw **Label** (not internal `ID_S`)
- Include both Pet ID and Stud ID on reports (Stud ID shown only if Sex ≠ F)
- Collection Time: US format (e.g. "1am" not "13:00")
