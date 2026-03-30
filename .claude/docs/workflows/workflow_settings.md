# Workflow: Application Settings & Configuration

## Forms Involved

| Form | Role |
|---|---|
| `frOptions` | General app options (preferences, paths, display settings) |
| `frReportSetup` | Report header, logo, and configurable dropdown lists — see [workflow_reports.md](workflow_reports.md) |
| `frLanguages` | Language / locale selection — see [workflow_sync.md](workflow_sync.md) |

## Settings Persistence Mechanisms

### Registry — Short Values (`GetVar` / `SetVar`)

Used for scalar settings (strings, numbers). Registry path: `SOFTWARE\OVT Charting System` (see [project/ovt/ovt_common.bas:10](../../project/ovt/ovt_common.bas#L10)).

Key values stored this way:

| Key | Description |
|---|---|
| `ID_DVC` | Unique device ID (generated once on first run) |
| `PATH_PICTURES` | Last folder used in pet/logo photo browser |
| `RS_Address`, `RS_Email`, `RS_Phone`, etc. | Report header clinic info |
| Language selection | Active UI language file |

### Database MEMOS Table — Long Values (`GetMemo` / `SetMemo`)

Used for memo-length text that exceeds 256 chars. Unlike `GetVar`/`SetVar` (registry), `GetMemo`/`SetMemo` store values in the `MEMOS` table in `bd.mdb`.

Key values stored this way:

| Key | Description |
|---|---|
| `RS_INSEMINATION_INSTRUCTIONS` | Insemination instructions shown at bottom of Chilling report |

### Database VARS / UserLists Tables

`tbs.tiUserLists` — editable lookup lists (abnormality types, extender types, etc.) managed through `frReportSetup` using `DBListEdit` UserControls.

## OVT Window Limit

`OVT_OPEN_LIMIT = 10` — maximum number of OVT sub-windows that can be open simultaneously before a warning is shown. Set in [project/ovt/ovt_settings.bas:15](../../project/ovt/ovt_settings.bas#L15).

## OVT App Path

`OVT_APP_PATH = "..\ovt-dsk\"` — relative path to the companion OVT Desktop application. Used when launching OVT forms from BD-DSK menu items via `mdlOvtFunc.bas`.
