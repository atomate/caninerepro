# Workflow: Invoicing & Billing

## Forms Involved

| Form | Role |
|---|---|
| `frServices` | Manage the clinic's service catalog (list + edit) |
| `frEditService` | Modal editor for a single service item |
| `frDiscountGroups` | Manage discount group definitions |
| `frPayment` | Modal — add a payment to an invoice |
| `rpInvoice` (`.DCA`) | DataReport — single invoice printout |
| `rpInvoiceList` (`.Dsr`) | DataReport — invoice list / batch print |

## Data Model

```
SERVICES          — clinic service catalog (prices, descriptions)
CHARGE_TEMPLATES  — default charges auto-applied for a service type
INVOICES          — one invoice per visit or transaction
  └─ CHARGES      — line items on an invoice (linked to SERVICES)
PAYMENTS          — payments against an invoice
DISCOUNT_GROUPS   — named discount tiers
CLIENTS.Discount  — client's assigned discount group
DOCTORS           — attending doctor per invoice
```

## Flow: Service Catalog

```
frServices (list — DBListView lsServices, no KeyField — read-only list)
  btEditService → frEditService (modal)
    Fields: service name, default price, description
  btDeleteService → delete from SERVICES
  btChange → reorder / reassign

CHARGE_TEMPLATES: default charges associated with service types;
  auto-populated onto new invoices when a visit of that type is created.
```

## Flow: Invoice & Payment

```
(Invoice created automatically when a visit is saved, or manually)

Invoice header: Client, Pet, Doctor, Date, Invoice #
Line items (CHARGES):
  - each charge links to a SERVICE record
  - quantity × unit price = line total
  - discount applied based on client's DISCOUNT_GROUP

frPayment (modal — "Add Payment"):
  Fields: Payment Date (DTPicker, M/d/yyyy format), Payment Amount
  btAddPayment → INSERT into PAYMENTS with amount + date
  → invoice balance recalculated

Reports:
  rpInvoice   → single invoice with client address, line items, balance due
  rpInvoiceList → batch list of invoices (date range or filter)
```

## Flow: Discount Groups

```
frDiscountGroups (list — DBListView lsDiscountGroups)
  Define named tiers (e.g. "Staff 20%", "Referral 10%")
  Assign to a client via frEditClientPet > Client tab

  Timer1 (Interval=10ms, initially disabled) — used for deferred UI refresh
  after list edit operations.
```

## Notes

- Credit card info is stored on the client record (Card Type, Card Number, Card Expires Month/Year) — see DataEnv.DCA client query.
- `DOCTORS` table drives the doctor dropdown on invoices.
- Reports use the `RS_` prefixed variables (e.g. `RS_Email`, `RS_Address`) stored via `SetVar`/`GetVar` from `frReportSetup` — these appear in the report header.
