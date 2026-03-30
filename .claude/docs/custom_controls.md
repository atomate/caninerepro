# Custom Controls & Shared Library — BD-DSK

The project references a shared VB6 library compiled separately from `bd-dsk.vbp`. These modules appear in [project/CR_BD-DSK.CRB](../../project/CR_BD-DSK.CRB) but have no `.frm`/`.cls`/`.bas` source file in the `project/` folder. They are either compiled into the `.exe` from another project or registered as an ActiveX component.

---

## In-Project UserControls (source available)

### `StorageTree` ([project/StorageTree.ctl](../../project/StorageTree.ctl))

Wraps a `MSComctlLib.TreeView` plus a left-side toolbar for managing the storage hierarchy.

**Public interface:**
| Member | Type | Description |
|---|---|---|
| `Init(ShowButtons)` | Sub | Loads `ListStorage` lookup, populates tree from `Q_Storage` |
| `Refresh()` | Sub | Clears and reloads entire tree |
| `SaveSel()` / `RestoreSel()` | Sub | Preserve/restore selected node across refresh |
| `GetRootNode` | Property | Returns root `Node` object |
| `GetNodeByKey(key)` | Property | Returns `Node` by key string |
| `ni` | Collection | Map of `ID_ST` → `bdStorageNode` DTO for all loaded nodes |
| `n` | Node | Currently selected node |
| `NodeChanged(node)` | Event | Fired on any selection change |

**Toolbar buttons (context-sensitive):**
- `btAdd` — add sibling node at same storage level
- `btAddChild` — add child node (caption adapts: "Add Canister", "Add Goblet", etc.)
- `btEdit` — rename selected node's label (validates uniqueness within parent)
- `btDelete` — delete node and all contained items
- `btMove` — open `frStorageMove` to relocate node
- `btFreezingInfo` — open `frEditVisit` for the straw's source visit
- `btRefresh` — force reload

**Label uniqueness:** enforced in `GetLabel()` — [project/StorageTree.ctl:381](../../project/StorageTree.ctl#L381). Prevents duplicate labels within the same parent.

---

### `DBListEdit` ([project/DBListEdit.ctl](../../project/DBListEdit.ctl))

A `ListView` with inline edit and `New`/`Modify`/`Delete` toolbar. Used in `frReportSetup` for managing configurable lists (abnormalities, extenders).

**Key behaviour:**
- Items with `UserDefined=False` in `UserLists` table are shown as read-only — `btModify` and `btDelete` are disabled for them
- Editing is done via `ls.StartLabelEdit` (in-place rename)
- Type filtering: if multiple types are configured (`cbChoseType`), list is filtered by type
- Operations: `OP_ADD`, `OP_MODIFY`, `OP_DELETE`, `OP_ENUM` dispatched via private `DBCommand` — [project/DBListEdit.ctl:191](../../project/DBListEdit.ctl#L191)

---

### `OvtList` ([project/OvtList.ctl](../../project/OvtList.ctl))

A `DBListView` wrapper for displaying OVT patient chart (`FORMS`) records. Used on OVT forms.

**Public interface:**
| Member | Type | Description |
|---|---|---|
| `id_pt` | String | Filter by pet |
| `UseFilter` / `FilterQuery` | Bool/String | Optional SQL filter |
| `ReturnIDF` | Bool | Mode flag |
| `AfterTableSave(id_f)` | Event | Fired after OVT record saved |
| `AfterTableDelete(id_f)` | Event | Fired after OVT record deleted |
| `OVTOpenRequest(id_f)` | Event | Fired when user requests to open a chart |

---

## Shared Library Controls (source not in this project)

### `DbTable`

The core data-access component. Every table in `bdTables.cls` is a `DbTable` instance.

**Init signature** (from [project/bdTables.cls:67](../../project/bdTables.cls#L67)):
```
tiClients.Init db, "CLIENTS", "ID_CL", True, GUID_0
                   ^table     ^pk      ^autokey ^initial GUID
```

**Key methods / events (inferred from usage):**
| Member | Description |
|---|---|
| `Init(db, table, keyField, autoKey, initGUID)` | Initialise with connection, table name, PK field |
| `AddNew(field, value, ...)` | Insert new record with field/value pairs; returns new key |
| `Delete(key)` | Delete record by primary key |
| `Load(key, rs)` | Open a recordset for the given key |
| `AddNewBeforeUpdate(rs)` | Event — set defaults / generate IDs before INSERT |
| `AddNewAfterUpdate(rs, id)` | Event — post-insert hook |
| `AfterSave(rs, id)` | Event — after any UPDATE |
| `AfterDelete(id)` | Event — after DELETE |

---

### `DBListView`

Enhanced `ListView` with database-aware refresh, keyed items, sorting, and date formatting. Used everywhere a list of records is shown.

**Key properties / methods (inferred from usage across all forms):**
| Member | Description |
|---|---|
| `Init(keyField, multiSel, guid, col1, col2, ...)` | Configure columns and key field |
| `DateFormat` | Set display format for date columns (e.g. `LDF`) |
| `AddItems(rs, append, clear)` | Populate from recordset |
| `RefreshItem(rs, item)` | Update one row in place |
| `RemoveItem(item)` | Remove one row |
| `ItemByKey(key)` | Find `ListItem` by record key |
| `CurrentKey` | String key of currently selected item |
| `ItemChanged(item, key)` | Event — selection changed |
| `ItemDblClick(item, key)` | Event — double-click |
| `UpdateField(rs, item, fieldName, fieldValue, error)` | Event — called to populate computed columns that can't bind directly |

---

### `Sizer`

Layout helper that resizes controls proportionally when a form is resized.

**Key methods (inferred):**
| Member | Description |
|---|---|
| `ResizeByControl(form)` | Set reference dimensions from form's current size |
| `Bind(control, anchors)` | Register a control with anchor flags: `"w"` width, `"h"` height, `"l"` left, `"wh"` both |
| `ResizeWH(width, height)` | Apply bindings for given dimensions |

---

### `bdView`

Manages a view filter combo + search box connected to a `DBListView` and a `DbTable`.

**Key methods (inferred):**
| Member | Description |
|---|---|
| `Init(table, listview, combo, searchbox)` | Wire components together |
| `Add(name, sqlFilter, searchable)` | Register a named view with optional WHERE clause |

---

### `ObjectExtender` family

Base class and concrete extenders that fix or augment built-in VB6 data-bound controls:

| Control | Purpose |
|---|---|
| `oeFixDataCombo` | Fixes two-way binding issues with `MSDataListLib.DataCombo` |
| `oeFixDTPicker` | Fixes "Field Is Not Updatable" error when `DTPicker` is bound to `DataEnv` on form load — see [project/notes.txt:93](../../project/notes.txt#L93) |
| `oeTabsFrames` | Implements a tab strip by showing/hiding `Frame` controls; used in `frEditClientPet`, `frEditVisit`, `frReportSetup` |
| `oeDBUnique` | Validates that a field value is unique within its table before save |
| `oeFrameEnable` | Enables or disables all controls inside a frame at once |

---

## Third-Party OCX/DLL Dependencies

All must be registered on the target Windows machine:

| File | Controls provided |
|---|---|
| `mscomctl.ocx` | ListView, TreeView, ProgressBar, ImageList (`MSComctlLib`) |
| `mscomct2.ocx` | DTPicker, UpDown (`MSComCtl2`) |
| `msdatlst.ocx` | DataCombo, DataList (`MSDataListLib`) |
| `comdlg32.ocx` | CommonDialog (`MSComDlg`) |
| `msinet.ocx` | Inet HTTP control (`InetCtlsObjects`) — used in `frReproUpdate` |
| `ceutil.dll` | CE utility for PocketPC connectivity |
| `rapi.dll` | Remote API for PocketPC sync |
| `zlib.dll` | Compression (likely used during sync/backup) |
| `win.tlb` | Windows type library |
