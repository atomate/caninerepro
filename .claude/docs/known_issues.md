# Known Issues & Tech Debt — BD-DSK

Extracted and organised from [project/notes.txt](../../project/notes.txt). Items marked `[-]` are open; `[+]` are resolved; `[!]` are notes/decisions.

---

## Data Integrity Risks

**[!] AutoID collision risk**
`AutoID` generates a human-readable ID (e.g. "Visit 0042") by scanning existing values. It can produce a value that already exists. IDs are not validated for uniqueness at creation time. — [notes.txt:55-57](../../project/notes.txt#L55-L57)

**[!] SYNC table missing entries**
After sync, OVT record lists are not refreshed. After importing from OVT, Edit/Delete buttons on "All OVT Records" form are disabled. — [notes.txt:3-4](../../project/notes.txt#L3-L4)

**[!] Orphan straws on delete**
Deleting a visit or transfer leaves straw records pointing to an invalid `ID_VI` or `ID_TR`. Fixed by adding FK relationships (VISITS→STORAGE cascade delete; TRANSFERS→STORAGE no cascade + manual NULL). — [notes.txt:46-54](../../project/notes.txt#L46-L54)
> Confirmed fixed. Any new cascade-delete logic must follow the same pattern.

**[!] STATES tables not fully populated**
`ListStates` and related state/lookup tables may have missing values. — [notes.txt:57](../../project/notes.txt#L57)

---

## Business Logic Gaps

**[-] OVT list not refreshed after sync**
After completing a sync operation, the OVT record list UI does not refresh automatically. — [notes.txt:3](../../project/notes.txt#L3)

**[-] Edit/Delete disabled after OVT import**
On the "All OVT Records" form, Edit and Delete buttons remain disabled after importing from OVT. — [notes.txt:4](../../project/notes.txt#L4)

---

## Storage & Straw Rules

**[!] Straw uniqueness validation**
Before adding straws on `frEditVisit`, verify no other straw with the same label and same `ID_VI` exists. Duplicates must be rejected. — [notes.txt:22](../../project/notes.txt#L22)

**[!] Last Location must always be updated**
Whenever `STORAGE.Parent` changes (straw moved in or out of storage), `Last Location` must also be updated. This must be enforced in every code path that modifies `Parent` — not just `frStorageMove`. — [notes.txt:13](../../project/notes.txt#L13)

**[!] Storage table logic changed**
`Parent IS NULL` now means a straw is out of storage (not stored in any goblet). New straws are created with `Parent=NULL` until placed. Any old code assuming a non-null Parent means "not in storage" is wrong. — [notes.txt:41-43](../../project/notes.txt#L41-L43)

---

## UI Issues

**[!] Tab order**
Tab order was fixed on `frVisits` but needs to be done on all other forms. — [notes.txt:59](../../project/notes.txt#L59)

**[!] DTPicker "Field Is Not Updatable"**
Caused by binding `DTPicker` to `DataEnv` at form load. Fixed by removing the binding at design time and setting it programmatically later. Other controls don't have this issue. — [notes.txt:93](../../project/notes.txt#L93)

---

## Reports

**[!] Client report outdated**
Client report needs updating to include new fields: Credit Card Expires, Card Type, etc. — [notes.txt:26](../../project/notes.txt#L26)

**[!] Memo fields overflow**
Some reports (e.g. Comments on Freezing tab) do not expand to fit long text. Use `CanGrow=True` on report controls for memo fields. — [notes.txt:18](../../project/notes.txt#L18)

**[!] Double spacing on reports**
All reports need to be made more compact — remove double spacing, align fields. — [notes.txt:27](../../project/notes.txt#L27)

**[!] Chilling report shows 2 pages**
`rpChilling` (and possibly other reports) renders an extra blank page. — [notes.txt:27](../../project/notes.txt#L27)

**[!] Reports need updating for schema changes**
Several fields were renamed or split. Reports must reflect:
- `Date Last Collected` removed; `Collection Time`, `Semen Owner`, `Photo Front/Left/Right` added
- `Send To` split into separate Name/Address/City/State/Zip fields
- Stud ID and Pet ID are now separate fields — show both; hide Stud ID only if Sex = F
- `fmSoundnessOnly` frame fields should appear only on Soundness report
- RS_ variables renamed from `HDEmail`, `HDAddress` etc. to `RS_Email`, `RS_Address` etc.

---

## Feature Gaps (lower priority)

**[!] Weight sync: lb ↔ kg**
Weight field should have two synchronised inputs (lb and kg). — [notes.txt:34](../../project/notes.txt#L34)

**[!] Collection time dropdown**
Collection Time should be a dropdown: 8am, 9am … 10pm. — [notes.txt:31](../../project/notes.txt#L31)

**[!] Freezing Start Time format**
Should display in US format (e.g. "1am" not "13:00"). — [notes.txt:32](../../project/notes.txt#L32)

**[!] Abnormalities: append, don't replace**
When user picks abnormalities from the list, new items should append rather than replace the selection. OK button on the dialog should be renamed to "Add". — [notes.txt:23](../../project/notes.txt#L23)

**[!] Species dropdown**
Species should be a dropdown: Canine, Feline, Equine, Bovine, Ovine. — [notes.txt:35](../../project/notes.txt#L35)

**[!] Card Type above Card Number**
On frEditClientPet, move Card Type field above Card Number. Add: Master Card, Discover, American Express, Visa. — [notes.txt:33](../../project/notes.txt#L33)

**[!] Tank Population defaults**
frTankPopulate should default to 6 containers per tank / 24 goblets / 2 positions. — [notes.txt:37](../../project/notes.txt#L37)

**[!] New visit: Semen Owner defaults to Client**
On a new visit, the Semen Owner field should default to the current client. — [notes.txt:24](../../project/notes.txt#L24)
