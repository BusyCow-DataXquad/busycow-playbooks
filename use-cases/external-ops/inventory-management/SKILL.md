---
skill: inventory-management
version: 1.0.0
use_case: External Ops — Inventory Management
applies_to: SME distributors and dealers managing a reseller network
databases:
  - product_catalog: $INV_MGMT_PRODUCT_CATALOG_DB_ID
  - dealer_list: $INV_MGMT_DEALER_LIST_DB_ID
  - outbound_records: $INV_MGMT_OUTBOUND_RECORDS_DB_ID
  - inbound_records: $INV_MGMT_INBOUND_RECORDS_DB_ID
notion_token: $NOTION_TOKEN
---

# Inventory Management Skill

## Overview

This skill gives a conversational agent full control over an SME distributor's inventory operations. It connects to four structured databases — Product Catalog, Dealer List, Outbound Records, and Inbound Records — and handles all data entry, stock updates, and reporting through natural conversation.

The schema described below is recommended. If the client's existing databases use different field names or structures, adapt gracefully. Map the client's fields to the logical roles described here and note any differences in your working memory for the session.

The agent replaces manual spreadsheet work. Every data capture and every query flows through chat. No one should need to open a database directly for day-to-day operations.

---

## Feature 1: Inventory Query

### Triggers
- "How much stock do we have for [product]?"
- "What's the inventory level on [product]?"
- "Check stock for [product name or code]"
- "Is [product] available?"
- Any message asking about quantity, availability, stock, or inventory for a specific product.

### Workflow
1. Identify the product from the user's message. If ambiguous, search the Product Catalog by name and product code and present matches for the user to confirm.
2. Retrieve the following fields from the Product Catalog record: Actual Stock, Committed Stock, In-Transit Stock, Frozen Stock.
3. Calculate Available-to-Promise: Actual Stock minus Frozen Stock.
4. Return the full stock summary.

### Response Format
Present stock data in a structured, readable format. Example:

Product: [Product Name] ([Product Code])
- Actual Stock: [n] units
- In-Transit Stock: [n] units (ordered, not yet arrived)
- Committed Stock: [n] units (reserved, not yet shipped)
- Frozen Stock: [n] units (held for pending orders)
- Available-to-Promise: [n] units

Always include Available-to-Promise. Never show only the raw Actual Stock figure in isolation.

---

## Feature 2: Outbound Recording

### Triggers
- "Log a shipment"
- "Record an outbound for [dealer]"
- "We shipped [product] to [dealer]"
- "Add outbound record"
- Any message indicating that goods have been or are being dispatched to a dealer.

### Workflow
1. Collect all required fields through conversation. Ask for missing fields one group at a time — do not fire a long list of questions all at once.
   Required: Dealer, Product, Quantity, Unit Price, Salesperson, Shipment Date, Status.
   Optional: Notes.
2. Look up the Dealer in the Dealer List and the Product in the Product Catalog. Confirm matches with the user if there is any ambiguity.
3. Check current stock:
   - Retrieve Actual Stock and Frozen Stock for the product.
   - Calculate resulting Actual Stock after the shipment (Actual Stock minus Quantity).
   - If the result would be negative, warn the user before proceeding:
     "Warning: This shipment of [n] units would bring Actual Stock to [negative number]. Current Actual Stock is [n]. Do you want to proceed anyway?"
   - Wait for explicit confirmation before continuing.
4. Calculate Total Amount: Quantity multiplied by Unit Price.
5. Present a full summary of the record to be created and ask for confirmation:
   "Here is the outbound record I will create. Please confirm:
   - Dealer: [name]
   - Product: [name]
   - Quantity: [n]
   - Unit Price: [amount]
   - Total Amount: [amount]
   - Salesperson: [name]
   - Shipment Date: [date]
   - Status: [status]
   Is this correct?"
6. On confirmation, create the Outbound Record in the database.
7. Update the Product Catalog:
   - If Status is Reserved or Pending Shipment: increase Frozen Stock by Quantity.
   - If Status is Shipped: decrease Actual Stock by Quantity. Do not increase Frozen Stock (the stock has left the warehouse).
   - If a record was previously at Reserved or Pending Shipment and is now being marked Shipped, decrease Actual Stock by Quantity and decrease Frozen Stock by Quantity.
8. Confirm to the user that both the outbound record and the stock update have been saved. Show the updated stock figures.

### Response Format
After successful recording:

Outbound recorded.
- Shipment: [Shipment Summary]
- Status: [status]

Updated stock for [Product Name]:
- Actual Stock: [new value]
- Frozen Stock: [new value]
- Available-to-Promise: [Actual Stock minus Frozen Stock]

---

## Feature 3: Inbound Recording

### Triggers
- "Log a purchase order"
- "We ordered [product] from [supplier]"
- "Record an inbound"
- "Goods arrived from [supplier]"
- "[Product] has arrived at the warehouse"
- Any message indicating that stock has been ordered from a supplier or has arrived.

### Workflow
1. Determine whether this is a new PO (status: Awaiting Arrival) or a goods arrival (status: Arrived). Ask if unclear.
2. Collect required fields:
   Required: Product, Quantity, Supplier/Manufacturer, Inbound Date, Status.
   Required if drop-ship: Dealer for Drop-ship, Arrival Method (set to Direct Drop-ship).
   Optional: Unit Price, PO Number, Notes.
3. Look up the Product in the Product Catalog. Confirm match with user if ambiguous.
4. Present a full summary and ask for confirmation before writing anything.
5. Create the Inbound Record in the database.
6. Update the Product Catalog:
   - If Status is Awaiting Arrival: increase In-Transit Stock by Quantity.
   - If Status is Arrived: increase Actual Stock by Quantity.
     If there is a matching Awaiting Arrival record for this PO, decrease In-Transit Stock by the arrived Quantity.
     Ask the user: "Should I link this arrival to an existing PO? If yes, which one?" before adjusting In-Transit Stock.
7. Confirm the record and show updated stock figures.

### Response Format
After successful recording:

Inbound recorded.
- Inbound: [Inbound Summary]
- Status: [status]

Updated stock for [Product Name]:
- Actual Stock: [new value]
- In-Transit Stock: [new value]
- Available-to-Promise: [Actual Stock minus Frozen Stock]

---

## Feature 4: Shipment Analysis

### Triggers
- "Show me shipment performance"
- "Who are our top dealers this month?"
- "Which products are moving the slowest?"
- "How much did [salesperson] ship last quarter?"
- "Give me an outbound report for [time period]"
- Any request for aggregated or summarized outbound data.

### Workflow
1. Identify the dimension the user wants to analyze: dealer, salesperson, product, product category, or time period.
2. If no time range is specified, ask: "What time period should I look at? For example: this month, last quarter, year-to-date, or a custom date range."
3. Query the Outbound Records table filtered by the specified time range and any other parameters.
4. Aggregate results:
   - Total quantity shipped and total revenue per group (dealer / salesperson / product / category).
   - Rank from highest to lowest.
   - For slow-moving stock analysis: cross-reference with Product Catalog to identify products with no outbound records in the period.
5. Present findings in a clear, ranked summary. Highlight the top 3 and flag any bottom performers or zero-movement products.

### Response Format
Shipment Analysis — [Time Period]
Grouped by: [dealer / salesperson / product / category]

Top performers:
1. [Name] — [quantity] units / [total revenue]
2. [Name] — [quantity] units / [total revenue]
3. [Name] — [quantity] units / [total revenue]

Slow-moving or zero-movement products (if applicable):
- [Product Name] — 0 units shipped in period

Total outbound for period: [quantity] units / [total revenue]

---

## Feature 5: Dealer Management

### Triggers
- "Look up [dealer name]"
- "What's the status of [dealer]?"
- "Show me [dealer]'s order history"
- "Does [dealer] have any outstanding orders?"
- Any request for information about a specific dealer's activity or status.

### Workflow
1. Search the Dealer List for the named dealer. If multiple matches, present them and ask the user to confirm which one.
2. Retrieve the dealer's record: Contact Person, Phone, Email, Region, Partnership Status, Notes.
3. Query the Outbound Records table for all records where Dealer matches this dealer.
4. Separate results into:
   - Outstanding orders: Status is Reserved or Pending Shipment.
   - Completed shipments: Status is Shipped.
5. Calculate totals: total units shipped (all time or filtered by date if user specifies), total revenue.
6. Present a full dealer profile.

### Response Format
Dealer: [Dealer Name]
Region: [region] | Status: [Active / Suspended / Terminated]
Contact: [Contact Person] — [Phone] — [Email]

Outstanding Orders:
- [Shipment Summary] | [Product] | [Qty] units | Status: [status] | Date: [date]
(or "None" if no outstanding orders)

Recent Shipments (last 10 or as specified):
- [Shipment Summary] | [Product] | [Qty] units | [Total Amount] | Date: [date]

Totals:
- Total units shipped: [n]
- Total revenue: [amount]

---

## Behavioral Rules

These rules apply across all features. Follow them without exception.

1. Never update stock without confirmation.
   Always present a full summary of what you are about to write and what stock changes will result. Wait for the user to explicitly say yes (or equivalent) before touching any database record.

2. Warn before going negative.
   Before recording any outbound shipment, check whether Actual Stock minus the shipment Quantity would result in a negative number. If it would, issue a clear warning and wait for confirmation. Do not proceed silently.

3. Always show Available-to-Promise.
   Whenever you display stock figures, always include Available-to-Promise (Actual Stock minus Frozen Stock) alongside the raw numbers. Never show Actual Stock alone as if it represents total availability.

4. Salesperson is always required on outbound records.
   If the user does not provide a salesperson when logging a shipment, ask for it before creating the record. Do not create an outbound record with a blank Salesperson field.

5. Always ask for time range on analysis requests.
   If the user asks for a report or analysis and does not specify a time period, ask before querying. Do not default silently to any time range.

6. Inbound arrival must link to existing PO when possible.
   When recording a goods arrival, ask whether it corresponds to an existing Awaiting Arrival record. If yes, link them and adjust In-Transit Stock accordingly. This keeps stock numbers accurate.

7. Outbound stock update rules by status:
   - Reserved or Pending Shipment: increase Frozen Stock by Quantity (stock is earmarked but not gone).
   - Shipped: decrease Actual Stock by Quantity (stock has left). If Frozen Stock was previously increased for this order, decrease it by the same Quantity at the same time.

8. Inbound stock update rules by status:
   - Awaiting Arrival: increase In-Transit Stock by Quantity.
   - Arrived (Warehouse): increase Actual Stock by Quantity; decrease In-Transit Stock by Quantity if linked to a PO.
   - Arrived (Direct Drop-ship): record the arrival but do not update Actual Stock — goods went directly to the dealer and never touched warehouse inventory. Note this clearly in the confirmation message.

9. Adapt to the client's schema.
   The field names and table structures above are recommended defaults. If the client's databases use different names, map them to the logical roles defined in this skill and operate on those mapped fields. Do not refuse to work because a field is named differently than expected.

10. Confirm all lookups before writing.
    When matching a dealer or product by name, always confirm the match with the user before using it in a record. If there are multiple close matches, list them and ask.

---

## Configuration

The following environment variables must be set in ~/.hermes/.env for this skill to function:

NOTION_TOKEN — Notion API integration token with read/write access to the workspace.
INV_MGMT_PRODUCT_CATALOG_DB_ID — Database ID of the Product Catalog table.
INV_MGMT_DEALER_LIST_DB_ID — Database ID of the Dealer List table.
INV_MGMT_OUTBOUND_RECORDS_DB_ID — Database ID of the Outbound Records table.
INV_MGMT_INBOUND_RECORDS_DB_ID — Database ID of the Inbound Records table.

Run the SETUP.md workflow to populate these values. Do not hardcode IDs in this file.
