# Workflow: Label Printing

## Forms & Modules Involved

| Component | Role |
|---|---|
| `frPrint` ([project/frPrint.frm](../../../project/frPrint.frm)) | Print preview/print dialog |
| `tiLabelFormats` in `bdTables` | Access to `LabelFormats` table |
| `mdlReports.bas` | `MAX_LABEL_NR = 8` constant; `CopyReportHeader` copies up to 8 labels |

## Data Model

### `LabelFormats` table
| Field | Notes |
|---|---|
| `ID_LABELFORMAT` | GUID PK |
| `LabelFormat` | Human-readable AutoID, e.g. "Label Format 001" |
| `LabelTemplate` | Template string defining label layout |

New records created via `bdTables.tiLabelFormats` — auto-assigned label:
```
rs("LabelFormat") = AutoID(table, "LabelFormat", "Label Format ", "LabelFormat LIKE 'Label Format %'")
rs("LabelTemplate") = ""
```
See [project/bdTables.cls:421](../../../project/bdTables.cls#L421).

Up to 8 label formats are supported (`MAX_LABEL_NR = 8` in [project/mdlReports.bas:9](../../../project/mdlReports.bas#L9)).

## frPrint

`frPrint` ([project/frPrint.frm](../../../project/frPrint.frm)) is a modal print dialog containing:
- A `PictureBox` (`pic`) for label preview (aspect-ratio scaled, GDI rendering via `StretchBlt`)
- A `CommonDialog` (`CMD`) from `comdlg32.ocx` for the system print dialog

The form is opened modally when the user prints a straw label. The `LabelTemplate` from the selected `LabelFormat` controls what data fields are printed on the straw.

## Label Content

Straw labels display the **Label** field from `STORAGE`, not `ID_ST`. The Label field is what is physically printed on the straw at the time of freezing. See [project/notes.txt:60](../../../project/notes.txt#L60):

> "On the storage Report STRAW ID is actually the straw Label not ID_S which is an internal value only, like all ID_ fields in our database."

## What is Not Yet Documented

The `LabelTemplate` format string syntax and the rendering code that interprets it are in a module not present in this repository (likely `bdMain` or the shared library). The `frPrint` form receives a pre-rendered image into `pic` — the rendering logic is elsewhere.
