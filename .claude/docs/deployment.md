# Deployment & Distribution — BD-DSK

## Application Distribution

BD-DSK is a single-user Windows desktop application. There is no installer source in this repository; deployment is handled by:

1. Compiling `bd-dsk.exe` from the VB6 IDE
2. Distributing the executable alongside its dependencies
3. The application can update itself via `frReproUpdate` (see below)

## Database File

`bd.mdb` is a Microsoft Access database that lives on the user's local machine (or a mapped network drive). It is not a server — there is no concurrent multi-user access mechanism.

- **Location**: configured in `DataEnv.DCA` connection string at compile time
- **Demo data**: [project/demo_data/bd_demo_data.mdb](../../project/demo_data/bd_demo_data.mdb)
- **Backup**: `OVT_BAK_PATH = App.path & "\backup.xml"` — backup is XML-serialised OVT data, set in [project/ovt/ovt_settings.bas:13](../../project/ovt/ovt_settings.bas#L13)

## Database Schema Updates (frSyncDBProgress)

When a new version of the application includes schema changes (new fields, tables, relationships), `frSyncDBProgress` handles the migration:

- Caption: "Database Update"
- `mdlSyncDB.bas` contains the migration SQL (`ALTER TABLE`, `CREATE TABLE`, etc.)
- `ProgressBar pbComplete` and `txtLog` (locked, read-only) show progress
- The "Continue" button (`btClose`) is enabled only after migration completes
- This dialog is shown automatically on startup when the app detects a version mismatch

The version check mechanism and the `mdlSyncDB.bas` source are not in this repository (likely in the shared library). The dialog and progress reporting are in [project/frSyncDBProgress.frm](../../project/frSyncDBProgress.frm).

## Application Self-Update (frReproUpdate)

`frReproUpdate` ([project/frReproUpdate.frm](../../project/frReproUpdate.frm)) downloads and applies application updates:

- Uses `InetCtlsObjects.Inet` (from `msinet.ocx`) for HTTP download
- `ProgressBar pbComplete` shows download progress
- `lblStatus` shows percentage
- `btStop` cancels download
- On completion: launches the downloaded installer executable

The update server URL is derived from `REPRO_HOST` (not in this repository).

## OVT Data Persistence Between Installs

Notes.txt records: "Need to implement data persistence between installations" — [notes.txt:39](../../project/notes.txt#L39). The `backup.xml` mechanism (`OVT_BAK_PATH`) is the planned solution: OVT records are exported to XML before uninstall and re-imported after reinstall.

The XML format is defined by `rec2xml`/`xml2rec` in [project/ovt/ovt_xml.bas](../../project/ovt/ovt_xml.bas).

## Registry Footprint

The application stores settings in the Windows registry under:
```
HKCU\SOFTWARE\OVT Charting System
```
See [project/ovt/ovt_common.bas:10](../../project/ovt/ovt_common.bas#L10). Keys written include:
- `ID_DVC` — unique device ID (set once, never changes)
- `PATH_PICTURES` — last-used picture browser folder
- `RS_*` — report header/clinic settings
- Language selection

## Sync Targets

BD-DSK can sync data with:

| Target | Mechanism |
|---|---|
| Web server | HTTP POST XML to `ReproServer.asp` via `clsReproServer` |
| PocketPC | RAPI (`rapi.dll`) — file-based sync via `sync.txt` |
| Palm | `frSyncPalm` — Palm-specific sync dialog |
| Desktop-to-Desktop | File sync via `sync.txt` (`SYNC_FILE` constant) |

The sync server hostname for OVT is `ovtvet.com` (`OVT_HOST`). The BD-DSK server hostname is `REPRO_HOST` (constant defined in the shared library).

## Files Required at Runtime

The following files must be present in or alongside the application directory:

| File | Required for |
|---|---|
| `bd.mdb` | All database operations |
| `mscomctl.ocx` | ListView, TreeView, ProgressBar |
| `mscomct2.ocx` | DTPicker |
| `msdatlst.ocx` | DataCombo |
| `comdlg32.ocx` | File open dialogs |
| `msinet.ocx` | Application self-update download |
| `ceutil.dll` | PocketPC sync |
| `rapi.dll` | PocketPC sync |
| `zlib.dll` | Compression (sync/backup) |
| `win.tlb` | Windows type library |
