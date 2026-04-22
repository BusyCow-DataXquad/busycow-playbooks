# SETUP — Inventory Management
# Written for the Hermes agent. Follow each phase in order.
# All instructions are imperative. Do not skip phases.

---

## Phase 1: Discovery

Search the client's Notion workspace for any existing tables or databases that may already cover this use case. Do not create anything yet.

Search for pages or databases containing these keywords (search one at a time):
- dealer
- distributor
- inventory
- stock
- inbound
- outbound
- shipment
- product catalog

For each match found, record:
- The database name
- The database ID
- A list of its existing fields and field types

Report your findings to the user before proceeding. Ask: "I found the following databases that may be relevant. Should I use any of these, or would you like me to create new ones from the recommended schema?"

Wait for user confirmation before moving to Phase 2.

---

## Phase 2: Onboard

Create the required tables in this exact order. The Product Catalog must be created first because all other tables reference it via relation fields.

If the user confirmed that an existing table covers a given role, skip creating that table and note its ID instead.

### Table 1: Product Catalog

This is the most critical table. All stock numbers live here. Create it first.

Recommended fields:
- Product Name — title (required)
- Product Code — text
- Brand — text
- Vehicle Brand/Series — select
- Category — select
- Cost Price — number
- Standard Price — number
- Actual Stock — number (current physical stock on hand)
- Committed Stock — number (reserved but not yet shipped)
- In-Transit Stock — number (ordered from supplier, not yet arrived)
- Frozen Stock — number (reserved for pending/confirmed orders)
- Notes — text

Note to agent: Available-to-Promise is a derived value, not a stored field. Calculate it as: Actual Stock minus Frozen Stock. Display it whenever stock is queried — do not store it separately unless the client's existing schema already has it as a formula field.

### Table 2: Dealer List

Recommended fields:
- Dealer Name — title (required)
- Contact Person — text
- Phone — text
- Email — email
- Address — text
- Region — select
- Partnership Status — select (options: Active, Suspended, Terminated)
- Notes — text

### Table 3: Outbound Records

Recommended fields:
- Shipment Summary — title (required; auto-generate as "Dealer Name — Product Name — Date" if the client has no existing naming convention)
- Dealer — relation → Dealer List (required)
- Product — relation → Product Catalog (required)
- Quantity — number (required)
- Unit Price — number
- Total Amount — number (calculate as Quantity × Unit Price; store the result)
- Salesperson — select (required; add options as you encounter new names)
- Shipment Date — date (required)
- Status — select (options: Reserved, Pending Shipment, Shipped)
- Notes — text

### Table 4: Inbound Records

Recommended fields:
- Inbound Summary — title (required; auto-generate as "Supplier — Product Name — Date" if no convention exists)
- Product — relation → Product Catalog (required)
- Dealer for Drop-ship — relation → Dealer List (optional; only fill when arrival method is Direct Drop-ship)
- Quantity — number (required)
- Unit Price — number
- Total Amount — number (calculate as Quantity × Unit Price; store the result)
- Supplier/Manufacturer — text (required)
- PO Number — text
- Arrival Method — select (options: Warehouse, Direct Drop-ship)
- Inbound Date — date (required)
- Status — select (options: Awaiting Arrival, Arrived)
- Notes — text

After creating all tables, record each database ID. You will save them in Phase 4.

---

## Phase 3: Adapt

Run these checks against all four tables — whether they were just created or already existed. Fix any gaps before proceeding.

### Check 1: Relation fields on Outbound Records
- Confirm a relation field exists pointing to Product Catalog.
- Confirm a relation field exists pointing to Dealer List.
- If either is missing, offer to add it. Ask: "The Outbound Records table is missing a relation to [Product Catalog / Dealer List]. Should I add this field now?"

### Check 2: Relation fields on Inbound Records
- Confirm a relation field exists pointing to Product Catalog.
- Confirm a relation field exists pointing to Dealer List (for drop-ship).
- If either is missing, offer to add it in the same way.

### Check 3: Stock fields on Product Catalog
- Confirm that Actual Stock, Frozen Stock, and In-Transit Stock exist as number fields.
- If any are missing, offer to add them. These fields are required for the agent's stock-update workflows to function.

### Check 4: Status fields
- Confirm Outbound Records has Status with options: Reserved, Pending Shipment, Shipped.
- Confirm Inbound Records has Status with options: Awaiting Arrival, Arrived.
- If the client has different status labels, map them to these internal names and note the mapping. Ask the user to confirm the mapping before saving it.

Report the results of all checks to the user. List what was found, what was added, and any mappings that were created.

---

## Phase 4: Configure

Save all database IDs to agent memory using the memory tool.

Use the memory tool to save all IDs under a clearly labeled memory entry:

```
Inventory Management config:
  product_catalog_db: <database ID of Product Catalog>
  dealer_list_db: <database ID of Dealer List>
  outbound_records_db: <database ID of Outbound Records>
  inbound_records_db: <database ID of Inbound Records>
```

After saving to memory, confirm to the user: "Configuration saved to agent memory. The inventory management skill is ready to use."

If any database ID is unknown (e.g., user chose to skip creation), leave the value blank and tell the user which IDs still need to be filled in manually.
