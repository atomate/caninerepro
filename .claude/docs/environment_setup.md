# Environment Setup — BD-DSK

## Requirements

| Requirement | Notes |
|---|---|
| **OS** | Windows XP / Vista / 7 (32-bit or 32-bit compatibility mode). VB6 runtime is 32-bit only. |
| **IDE** | Visual Basic 6.0 IDE (VB6 Enterprise or Professional edition). Service Pack 6 recommended. |
| **Runtime** | VB6 runtime (`msvbvm60.dll`) must be installed on any machine running the compiled `.exe` |
| **Database** | Microsoft Access (`.mdb`) via Jet OLE DB 4.0. No separate Access installation needed — Jet ships with Windows/Office. |

## OCX/DLL Registration

The following files must be registered with `regsvr32` before the project will open or run. Files marked with `†` are included in [project/](../../project/):

```
regsvr32 mscomctl.ocx       # ListView, TreeView, ProgressBar (from Windows/System32 or project/)
regsvr32 mscomct2.ocx       # DTPicker, UpDown
regsvr32 msdatlst.ocx       # DataCombo, DataList
regsvr32 comdlg32.ocx       # CommonDialog
regsvr32 msinet.ocx         # Inet HTTP control
regsvr32 ceutil.dll †       # PocketPC CE utilities
regsvr32 rapi.dll  †        # PocketPC Remote API
```

`zlib.dll` and `win.tlb` must be present in the same folder as the executable or on `PATH`.

> **Note**: On Windows 7+, run `regsvr32` as Administrator. Some OCX files may need to be copied to `C:\Windows\SysWOW64` on 64-bit systems.

## Opening the Project

1. Install VB6 and apply Service Pack 6
2. Register all OCX/DLL dependencies listed above
3. Open VB6 IDE → File → Open Project → select `bd-dsk.vbp`
   - The `.vbp` path is referenced in [project/CR_BD-DSK.CRB:3](../../project/CR_BD-DSK.CRB#L3): `D:\CODE\ROSKIN\BD-DSK\bd-dsk.vbp`
   - The actual `.vbp` file is not present in this repository; it must be located alongside the source files
4. If VB6 reports missing controls on open, re-register the relevant OCX and reopen

## Pointing to a Database

The database connection string is in `DataEnv.DCA` (binary). To change it:

1. Open `DataEnv` designer in VB6 IDE (double-click `DataEnv` in the Project Explorer)
2. Right-click the `cnn` connection → Properties
3. Change `Data Source` to point to your local `bd.mdb`
   - Current hardcoded path: `\\tsclient\D\CODE\ROSKIN\BD-DSK\ReproData\Database\bd.mdb` (a Terminal Services client path — the original developer worked via RDP)
4. A demo database is available at [project/demo_data/bd_demo_data.mdb](../../project/demo_data/bd_demo_data.mdb)

> The `REPROPICTURES` constant (used for the logo path in reports) and `REPRO_HOST` / `REPRO_VERSION` constants used by `clsReproServer` are likely defined in a module not present in this repository (`bdMain` or `dbUtils`). Search the compiled binary or the missing shared library for their values.

## Building / Compiling

In VB6 IDE: **File → Make bd-dsk.exe**

There is no command-line build script. The `SC_COMPILE.OUT` file in [project/](../../project/) is the output log from a previous compile run.

## Running the Static Analysis Tool

[project/CR_BD-DSK.CRB](../../project/CR_BD-DSK.CRB) is a project file for **VBCompiler 6** (a third-party VB6 static analysis tool, not the VB6 IDE compiler). To use it:

1. Install VBCompiler 6
2. Open `CR_BD-DSK.CRB`
3. Run analysis — results go to `CR_BD-DSK.MDB` (database) and `CR_BD-DSK.HTM` (summary report)
4. Current ruleset: "All Rules"; uncalled-procedure check and cross-reference enabled; naming rules skipped (`SKIPNAMING=1`)

## Known Environment Quirks

- **DTPicker binding**: binding `DTPicker` to `DataEnv` at form load causes "Field Is Not Updatable" error. The fix is to delay binding until after form load — see [project/notes.txt:93](../../project/notes.txt#L93).
- **DataCombo binding**: `oeFixDataCombo` extender is required to make two-way binding work correctly with `MSDataListLib.DataCombo`.
- **Terminal Services path**: the `DataEnv.DCA` connection string uses `\\tsclient\D\...`, meaning the original dev environment was accessed via RDP/Terminal Services. Local paths will differ.
