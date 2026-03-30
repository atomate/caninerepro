# Workflow: Semen Collection Visit

## Forms Involved

| Form | Role |
|---|---|
| `frVisits` | MDI child list of visits for the selected pet |
| `frEditVisit` | Tabbed modal editor for a single visit |
| `frReportMorphology` | Modal — morphology detail entry |

## Flow

```
frVisits (list — filtered by tbs.id_pt)
  ├─ btAdd → requires pet selected (tbs.id_pt ≠ 0)
  │           frEditVisit.Load 0 → frEditVisit.btNew_Click → Show vbModal
  └─ btEdit / double-click → frEditVisit.Load <ID_VI>
```

## frEditVisit Tab Structure

The visit editor is a tabbed form. Each tab represents a distinct type of semen service:

### Tab: Collection (General)
- Collection Date, Collection Time (dropdown: 8am–10pm)
- Semen Owner (defaults to Client)
- Dog's age, weight, height (fields visible on form)
- Photos: Front, Left, Right (same `LoadImg` helper as pet photos)
- Comments (memo field)

### Tab: Freezing
- Freezing Start Time (US format: 1am not 13:00)
- Volume, Concentration, Total Motility, Progressive Motility
- Abnormalities — 3 separate lists: Head / Midpiece / Tail
  - picked from `frReportSetup` configured lists; new items append (not replace)
- Extender type (from configured list)
- Number of straws produced → triggers straw creation
- **Storage sub-flow**: after straws are created, `frStorageMove.Load` is called with `pSuggest=True` to place straws into a goblet — see [workflow_storage.md](workflow_storage.md)
- AKC Freezing fields (`tiAKC_Freezings`)

### Tab: Chilling
- Send To: Name, Hospital, Address, City, State, Zip, Phone
- Insemination Instructions (stored via `GetMemo`/`SetMemo` with key `RS_INSEMINATION_INSTRUCTIONS`)
- AKC Chilling fields (`tiAKC_Chillings`)

### Tab: Soundness (Soundness-only evaluation)
- Fields in `fmSoundnessOnly` frame — appear only on Soundness Report
- AKC Fresh fields (`tiAKC_Fresh`)

### Tab: Straws (on frPetInfo)
- Lists all straws for this pet
- Buttons: Add / Remove / Delete / Storage Location / Remove From Storage
- Handler logic shared with frEditVisit via `bdDB.bas`

## Visit Record Fields (VISITS table)
Key fields added/changed over time (see [project/notes.txt:62-63](../../project/notes.txt#L62-L63)):
- `Collection Time`, `Semen Owner`, `Photo Front`, `Photo Left`, `Photo Right` — added; `Date Last Collected` removed
- `Soundness` (bool), `Freezing` (bool), `Collection` (bool) — used to compute the "Reason" column in `frVisits` list — see [project/frVisits.frm:147-149](../../project/frVisits.frm#L147-L149)

## Straw Uniqueness Rule
Before adding straws on frEditVisit, check that no other straw with the same label and same `ID_VI` exists — see [project/notes.txt:22](../../project/notes.txt#L22).

## Database Tables

`VISITS` (ID_VI) → `STORAGE` (straws, 1:many, cascade delete)
`VISITS` → `AKC_FREEZINGS`, `AKC_CHILLINGS`, `AKC_FRESH` (1:1 per visit type)
