# BusyCow Playbooks

Reusable AI agent use cases for SMEs — packaged for fast client deployment.

Each use case is a self-contained module that includes:
- **Skills** — Hermes skill files that teach the agent how to operate
- **README** — what it does, required data tables, and setup steps
- **Config Template** — settings to fill in (tokens, IDs, chat IDs)

Data tables are shared across use cases and live separately in `data-foundation/`.

---

## How It Works

```
data-foundation/          ← Shared data — install once, used by many use cases
  core-business/          ← Business KB (brand, audience, offer, model, ops)
  growth/                 ← CRM tables (Accounts, Contacts, Activities, etc.)

use-cases/                ← Pick and install what the client needs
  growth/
    lead-nurturing/
    content-intelligence/
```

Clients select use cases. Each use case README tells you which data tables to install.
You never install more than what's needed.

---

## Data Foundation

### Core Business KB

Required by all use cases. Fill this in first.

| Document | What it answers |
|----------|----------------|
| Brand Identity | Who we are, what we stand for, how we sound |
| Target Audience | Who we're building for, their real problem |
| Offer | What exactly we sell, what the customer gets |
| Business Model | How we make money, how deals get done |
| Operations | How the business actually runs day-to-day |

→ [Setup instructions](./data-foundation/core-business/README.md)

### Growth Data

Shared tables for all Growth use cases.

| Table | Layer | Used By |
|-------|-------|---------|
| Accounts | Fact | Lead Nurturing, Partner Nurturing |
| Contacts | Fact | Lead Nurturing, Partner Nurturing |
| Activities | Record | Lead Nurturing, Partner Nurturing |
| Deals | Relationship | Lead Nurturing |
| Partnership | Relationship | Partner Nurturing |
| Content Library | Standalone | Lead Nurturing, Content Intelligence |

→ [Setup instructions](./data-foundation/growth/README.md)

---

## Use Cases

### Growth

| Use Case | Description | Required Tables |
|----------|-------------|----------------|
| [Lead Nurturing](./use-cases/growth/lead-nurturing/) | AI-powered CRM — log interactions, track pipeline, daily briefing, bulk outreach | Accounts, Contacts, Activities, Deals, Content Library |
| [Content Intelligence](./use-cases/growth/content-intelligence/) | Monitor sources, generate on-brand content, save to Content Library | Content Library, Sources DB |

---

## Installation

### 1. Set up Core Business KB

```bash
# Fill in the templates and add page IDs to ~/.hermes/.env
# See data-foundation/core-business/README.md
```

### 2. Create required Notion databases

```bash
# Follow data-foundation/growth/README.md
# Only create the tables your selected use cases need
```

### 3. Install the skill(s)

```bash
# Example: Lead Nurturing
cp -r use-cases/growth/lead-nurturing/skills/growth/lead-nurturing ~/.hermes/skills/growth/
```

### 4. Configure

```bash
cat use-cases/growth/lead-nurturing/config-template/env-template.txt >> ~/.hermes/.env
# Fill in all values
```

---

> Maintained by [BusyCow](https://busycow.ai)
