# Workflow: Straw Transfer & Disposition

## Forms Involved

| Form | Role |
|---|---|
| `frTransfer` | MDI child — transfer record editor + straw list |
| `frAddStraws` | Modal — select straws from storage to add to a transfer |
| `frTransferAddStraws` | (helper form for adding straws to transfer) |
| `frDisposition` | Modal — record how/when straws were used or disposed of |
| `frOVTInseminations` | Modal — link a disposition to an existing OVT insemination record |

## What is a Transfer?

A transfer records the shipment of frozen straws from this clinic to another location (another vet, owner, stud station). Each transfer has:
- Date, Purpose of Transfer
- Destination: To Name, To Hospital, To Address, City, State, Zip, Phone
- Client info (copied at time of transfer)
- Agent info (shipping agent)
- Bitch info (recipient female's breed, etc.)
- A list of straws being transferred

## Flow: Creating a Transfer

```
frTransfer (new record)
  1. Fill in transfer details (date, purpose, destination, agent)
  2. btAddStraws → frAddStraws (modal)
       - shows straws currently in storage for this pet
       - user selects desired straws
       - prompt: "Are you sure you want to transfer the selected straws
                  out of storage? Straws will be removed from storage."
       - on confirm:
           UPDATE STORAGE SET Parent=NULL WHERE ID_S IN (selected)
           UPDATE STORAGE SET [Last Location] = <previous goblet> (remember location)
           → opens frDisposition (see below)
  3. Straws appear in frTransfer's ls (DBListView, KeyField=ID_ST)
     Column shows "Last Location" (not Storage Location) — see notes.txt:13
  4. btDeleteStraws → removes straw from transfer list
       UPDATE STORAGE SET ID_TR=NULL WHERE ID_S=<key>  ← must do BEFORE any delete
```

## Flow: Disposition (Straw Use/Removal)

`frDisposition` is shown after straws are added to a transfer, and can also be opened independently.

```
frDisposition
  Disposition Date (defaults to today)
  Radio options:
    ○ New Insemination     → creates new insemination record linked to straws
    ○ An Insemination I've already entered:
        → frOVTInseminations modal — pick existing insemination from lsI list
  lsI (DBListView, KeyField=ID_IN) — shows inseminations for this pet

  on Cancel: no disposition recorded; straws remain with Parent=NULL
  on OK:     UPDATE STORAGE SET [Disposition Date]=<date> for selected straws
```

## Delete Transfer Rule

Before deleting a transfer record (`ID_TR=100`), you must first:
```sql
UPDATE STORAGE SET ID_TR=NULL WHERE ID_TR=100
```
Otherwise the FK constraint will block the delete. The straw record is intentionally **not** cascade-deleted — straws must remain in the database without a transfer link. See [project/notes.txt:51-54](../../project/notes.txt#L51-L54).

## Database Tables

`TRANSFERS` (ID_TR) ← `STORAGE.ID_TR` (no cascade — manual NULL before delete)
`INSEMINATIONS` (ID_IN) ← linked to straws via disposition
`STORAGE` (ID_S, Parent, ID_TR, ID_VI, Last Location, Disposition Date)
