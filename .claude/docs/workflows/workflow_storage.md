# Workflow: Cryogenic Storage Management

## Forms Involved

| Form | Role |
|---|---|
| `frStorage` | MDI child — full storage tree view |
| `frTankPopulate` | Modal — bulk-create canister/goblet structure in a tank |
| `frStorageMove` | Modal — pick a storage location for one or more straws |

## Controls Involved

| Control | File |
|---|---|
| `StorageTree` UserControl | [project/StorageTree.ctl](../../project/StorageTree.ctl) |
| `bdStorageNode` class (DTO) | [project/bdStorageNode.cls](../../project/bdStorageNode.cls) |

## Storage Hierarchy

```
Tank
 └─ Canister
      └─ Goblet
           └─ Straw (leaf node — ID_S in STORAGE table)
```

Each node in the tree is keyed by its `ID_S` (storage record). `bdStorageNode` carries the node's label, storage ID, child storage ID, `ID_VI` (which visit/freezing the straw came from), and freezing description.

The `Parent` field in the STORAGE table encodes position in the hierarchy:
- `Parent IS NULL` → straw is **out of storage** (transferred out or not yet placed)
- `Parent = <ID of goblet>` → straw is inside that goblet
- See [project/notes.txt:41-43](../../project/notes.txt#L41-L43)

## Flow: Populate a New Tank

```
frStorage → btAdd tank node → frTankPopulate (modal)
  Inputs (3 levels, control arrays Index 0/1/2):
    - txCount(0) = number of canisters per tank   (default 6)
    - txCount(1) = number of goblets per canister  (default 24)
    - txCount(2) = number of positions per goblet  (default 2)
  Each level also has cbLabelType to set label format
  → on OK: bulk-inserts STORAGE records with Parent relationships
```

## Flow: Move Straws to Storage (from Visit / Freezing tab)

```
frEditVisit (Freezing tab) → straws created → frStorageMove.Load(pNeedStorage, pSuggest=True)
  frStorageMove shows StorageTree UserControl
  If pSuggest=True:
    DoSugest() traverses each tank, finds first SuggestPerTank(=5) empty goblets
    → populates lsSugest listbox with dbStorageLocation() strings
    → clicking a suggestion selects that node in the tree
  User selects destination goblet
  ckMulti checkbox: enables multi-location mode (split straws across goblets)
    - lsMulti list + >> / << buttons to assign counts per location
  → on OK: UPDATE STORAGE SET Parent=<selected goblet ID> for each straw
           UPDATE Last Location field
```

## Flow: Move Straws Manually

```
frStorage (tree view) → select straw → context or button → frStorageMove.Load(pNeedStorage, pSuggest=False)
  (same form, suggestions hidden)
```

## Suggestion Algorithm

Implemented in [project/frStorageMove.frm:140-176](../../project/frStorageMove.frm#L140-L176):

1. Walk each tank's root node
2. For each tank call `SearchTank(tankNode, SuggestPerTank=5)`
3. `SearchTank` recurses: if a node has no children AND its `Storage` count < `NeedStorage` → add to suggestions list
4. Stops once `SuggestPerTank` suggestions are found per tank

## Last Location Rule

Whenever a straw changes `Parent` (moved to/from storage), the `Last Location` field must also be updated. This should happen in every place that modifies `Parent` — not just in `frStorageMove`. See [project/notes.txt:13](../../project/notes.txt#L13).

## Database Tables

`STORAGE` (ID_S, Parent, ID_VI, ID_TR, Label, Last Location, Disposition Date)
- Self-referencing: `Parent` → another `ID_S` (the goblet)
- `ID_VI` FK → `VISITS` (cascade delete — straw removed when visit deleted)
- `ID_TR` FK → `TRANSFERS` (no cascade — set `ID_TR=NULL` before deleting a transfer)
