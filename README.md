# BusyCow Playbooks

Reusable AI agent use cases for SMEs — packaged for fast client deployment.

Each playbook is a self-contained module that teaches a Hermes agent how to handle a specific business function. Clients pick only what they need. Nothing more, nothing less.

---

## What's Inside

```
data-foundation/              ← Shared data layer — install once, used across use cases
  core-business/              ← Business KB: brand, audience, offer, model, ops
  growth/                     ← CRM tables: Accounts, Contacts, Activities, etc.
  internal-ops/               ← Internal tables: Discussions, Tasks, HR, Finance

use-cases/                    ← Business applications — pick what the client needs
  growth/
    lead-nurturing/           ← CRM, pipeline, daily briefing, bulk outreach
    content-intelligence/     ← Source monitoring, content generation
  internal-ops/
    discussion-todo-tracker/  ← Meeting notes, todo extraction, business core updates

skills/                       ← Infrastructure skills for advanced deployments
  autonomous-ai-agents/
    team-gbrain/              ← Shared knowledge brain for multi-agent teams
    team-gstack/              ← Shared infra orchestration for multi-agent teams
```

---

## Architecture

Every deployment starts with **Core Business KB** — 5 documents that tell the agent who the company is, who they serve, and how they operate. Without this, no use case works well.

On top of that, clients install the **data tables** their use cases need, then install the **skills** that run those use cases.

```
Core Business KB  (always required)
       ↓
Data Foundation   (install only what your use cases need)
       ↓
Use Cases         (pick from the menu below)
```

---

## Data Foundation

### Core Business KB

Required by all use cases. Fill these in before anything else.

| Document | What it covers |
|----------|---------------|
| Brand Identity | Who we are, what we stand for, tone and voice |
| Target Audience | Who we're building for, their real pain |
| Offer | What we sell, what the customer gets |
| Business Model | How we make money, how deals get done |
| Operations | How the business runs day-to-day |

→ [Setup guide](./data-foundation/core-business/README.md)

---

### Growth Data Tables

Shared across all Growth use cases.

| Table | Layer | Used By |
|-------|-------|---------|
| Accounts | Fact | Lead Nurturing, Partner Nurturing |
| Contacts | Fact | Lead Nurturing, Partner Nurturing |
| Activities | Record | Lead Nurturing, Partner Nurturing |
| Deals | Relationship | Lead Nurturing |
| Partnership | Relationship | Partner Nurturing |
| Content Library | Standalone | Lead Nurturing, Content Intelligence |

→ [Setup guide](./data-foundation/growth/README.md)

---

### Internal Ops Data Tables

Shared across all Internal Ops use cases.

| Table | Layer | Used By |
|-------|-------|---------|
| Internal Discussions | Record | Discussion & Todo Tracker |
| Task Tracker | Record | Discussion & Todo Tracker |

→ [Setup guide](./data-foundation/internal-ops/README.md)

---

## Use Cases

### Growth

| Use Case | What it does | Required Tables |
|----------|-------------|-----------------|
| [Lead & Partners Nurturing](./use-cases/growth/lead-nurturing/) | Conversational CRM — log activities, track pipeline, daily briefing, draft follow-up messages | Accounts, Contacts, Activities, Deals, Partnership, Content Library |
| [Content Intelligence](./use-cases/growth/content-intelligence/) | Monitor sources, extract insights, generate on-brand content, save to library | Content Library, Sources DB |

### Internal Ops

| Use Case | What it does | Required Tables |
|----------|-------------|-----------------|
| [Discussion & Todo Tracker](./use-cases/internal-ops/discussion-todo-tracker/) | Log meetings in plain language, extract todos and open issues, auto-update Business Core when strategy shifts | Internal Discussions, Task Tracker |

---

## Installation

### 1. Fill in Core Business KB

```bash
# Edit the 5 templates, then add page IDs to ~/.hermes/.env
# See data-foundation/core-business/README.md
```

### 2. Create the required Notion databases

```bash
# Follow the relevant data-foundation README
# Only create tables your selected use cases need
```

### 3. Install the skill(s)

```bash
# Example: Lead & Partners Nurturing
cp -r use-cases/growth/lead-nurturing/skills/growth/lead-nurturing \
      ~/.hermes/skills/growth/

# Example: Discussion & Todo Tracker
cp -r use-cases/internal-ops/discussion-todo-tracker/skills/internal-ops/discussion-todo-tracker \
      ~/.hermes/skills/internal-ops/
```

### 4. Configure

```bash
# Copy the env template and fill in your values
cat use-cases/growth/lead-nurturing/config-template/env-template.txt >> ~/.hermes/.env
```

---

## Infrastructure Skills

These are not use cases — they're building blocks for teams running multiple agents.

| Skill | What it does |
|-------|-------------|
| [team-gbrain](./skills/autonomous-ai-agents/team-gbrain/) | Shared knowledge brain — agents read/write a common GBrain instance |
| [team-gstack](./skills/autonomous-ai-agents/team-gstack/) | Shared infra — agents coordinate on a common GStack environment |

Install these when deploying Hermes across a team, not for single-user setups.

---

> Maintained by [BusyCow](https://busycow.ai)
