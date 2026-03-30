# Workflow: Client & Pet Management

## Forms Involved

| Form | Role |
|---|---|
| `frClients` | MDI child list of all clients |
| `frPets` | MDI child list of pets for selected client |
| `frSearchClientPet` | Modal search dialog to locate a client/pet by name |
| `frEditClientPet` | Tabbed modal editor — "Client" tab + "Pet" tab |
| `frPetInfo` | MDI child detail view for a single pet (straws, visits, history) |

## Flow

```
frClients (list)
  ├─ btAdd / btEdit → frEditClientPet (modal, Client tab)
  │     Fields: First/Last Name, Address, City, State, Zip, Country,
  │             Home/Work/Cell Phone, Email, SSN,
  │             Card Type, Card Number, Card Expires Month/Year, Notes,
  │             Associate Client (linked account)
  └─ double-click row → frPets (list, filtered by ID_CL)
         ├─ btAdd / btEdit → frEditClientPet (modal, Pet tab)
         │     Fields: Pet Name, Species, Breed (DataCombo → ListBreeds),
         │             Sex, DOB, Color, Eye Color, Weight, Height,
         │             Pet ID, Stud ID, Registered Name, Registration Number,
         │             Sire/Dam pedigree fields, DNA, Chip Number, Tattoo, Notes,
         │             Photo (front / left / right via LoadImg in mdlReports.bas)
         └─ double-click row → frPetInfo (detail view)
```

## Key Details

- **Selection propagates globally**: when a pet is selected, `tbs.id_pt` (on `bdTables`) is updated; `frVisits` uses this to filter its list (`WHERE [ID_PT]=#ID_PT`) — see [project/frVisits.frm:131](../../project/frVisits.frm#L131).
- **Breed lookup**: breed dropdown is a bound `DataCombo` pulling from the `ListBreeds` DataEnvironment query (joins `_BREED` + `_BREED_GROUP`).
- **Associate Client**: a client can be linked to another client record for shared billing.
- **Photos**: stored by reference (path) not blob; `LoadImg` in [project/mdlReports.bas:50](../../project/mdlReports.bas#L50) handles file dialog, aspect-ratio scaling, and GDI copy.
- **Delete guard**: cascade relationship — deleting a client cascades to their pets; deleting a pet cascades to visits/storage (see notes.txt:47-48).

## Database Tables

`CLIENTS` → `PETS` (1:many via `ID_CL`)
