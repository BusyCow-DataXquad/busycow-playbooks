# Inventory Management — BusyCow Use Case

## Problem Statement

SME distributors and dealers managing a reseller network face a common operational nightmare: inventory counts live in spreadsheets, inbound purchase orders arrive via WhatsApp or supplier forms, and outbound shipment records are tracked in yet another file — often by a different person. Admin staff spend hours each day manually copying data between systems, and by the time the owner looks at a report, the numbers are already stale.

This use case solves that by connecting a conversational AI agent to a structured database (Notion recommended). Any team member can log a shipment or purchase order through chat. The agent handles all data entry, updates stock levels automatically, and gives the owner real-time visibility into stock, shipments, and dealer performance — without touching a spreadsheet.

---

## Features

1. Inventory Query
   Ask about any product and instantly get current stock, in-transit stock, committed stock, and available-to-promise quantity.

2. Outbound Recording
   Log a shipment through conversation. The agent captures all required fields, confirms with you, then writes to the Outbound Records table and updates the Product Catalog stock figures automatically.

3. Inbound Recording
   Log a purchase order or goods arrival through conversation. The agent updates in-transit stock on PO creation and flips it to actual stock when the goods arrive.

4. Shipment Analysis
   Ask for a breakdown of outbound activity by dealer, salesperson, product category, or time period. The agent surfaces top performers and flags slow-moving stock.

5. Dealer Management
   Pull up any dealer's full shipment history, outstanding orders, and partnership status in one query.

---

## Package Contents

```
inventory-management/
  README.md              — This file. Human-readable overview and setup guide.
  SETUP.md               — Agent setup instructions (Discovery → Onboard → Adapt → Configure).
  SKILL.md               — Agent skill definition with triggers, workflows, and behavioral rules.
  config-template/
    env-template.txt     — Template for environment variables the agent needs to operate.
```

---

## Install Steps

1. Prerequisites
   - A Notion workspace with admin access (or equivalent database tool).
   - Hermes agent installed and configured at ~/.hermes/.env.
   - Notion API integration token with read/write access to the target workspace.

2. Set up the database schema
   - Open SETUP.md and run through the Discovery phase to check if tables already exist.
   - If starting fresh, follow the Onboard phase to create tables in the correct order:
     Product Catalog first, then Dealer List, then Outbound Records, then Inbound Records.
   - If tables already exist, follow the Adapt phase to check for missing relation fields.

3. Configure environment variables
   - Copy config-template/env-template.txt to ~/.hermes/.env (or append to an existing file).
   - Fill in all database IDs and your Notion API token.

4. Load the skill
   - Point the agent at SKILL.md so it knows the triggers, workflows, and behavioral rules for this use case.

5. Test
   - Ask the agent: "What is the current stock for [any product name]?"
   - Log a test outbound shipment and verify that stock figures update in the Product Catalog.
   - Log a test inbound PO and confirm in-transit stock increases.

6. Hand off to the team
   - Share the chat interface with admin staff.
   - No spreadsheet access required — all data entry goes through the agent.
