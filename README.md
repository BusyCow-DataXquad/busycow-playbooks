# BusyCow Playbooks

Reusable AI agent use cases for SMEs — packaged for fast client deployment.

Each playbook is a self-contained module that teaches a Hermes agent how to handle a specific business function. Clients pick only what they need. Nothing more, nothing less.

---

## Folder Structure

```
busycow-playbooks/
│
├── data-foundation/              ← Human reference only — schema docs and setup guides
│   ├── core-business/            ← Business KB templates: brand, audience, offer, model, ops
│   ├── growth/                   ← Growth data table reference: Accounts, Contacts, Deals, etc.
│   └── internal-ops/             ← Internal data table reference: Discussions, Tasks, HR, Finance
│
├── use-cases/                    ← Pick what the client needs — each is self-contained
│   ├── growth/
│   │   ├── lead-nurturing/
│   │   │   ├── README.md         ← What it does, schema overview, example interactions
│   │   │   ├── SETUP.md          ← Agent runs this once to set up the client's workspace
│   │   │   ├── SKILL.md          ← Agent uses this daily — behavioral rules and logic
│   │   │   └── config-template/
│   │   │       └── env-template.txt
│   │   └── content-intelligence/
│   │       ├── README.md
│   │       ├── SETUP.md
│   │       ├── SKILL.md
│   │       └── config-template/
│   │           └── env-template.txt
│   ├── internal-ops/
│   │   ├── hr-management/
│   │   │   ├── README.md
│   │   │   ├── SETUP.md
│   │   │   ├── SKILL.md
│   │   │   └── config-template/
│   │   │       └── env-template.txt
│   │   ├── financial-intelligence/
│   │   │   ├── README.md
│   │   │   ├── SETUP.md
│   │   │   ├── SKILL.md
│   │   │   └── config-template/
│   │   │       └── env-template.txt
│   │   └── discussion-todo-tracker/
│   │       ├── README.md
│   │       ├── SETUP.md
│   │       ├── SKILL.md
│   │       └── config-template/
│   │           └── env-template.txt
│   └── external-ops/
│       └── inventory-management/
│           ├── README.md
│           ├── SETUP.md
│           ├── SKILL.md
│           └── config-template/
│               └── env-template.txt
│
└── skills/                       ← Separate infrastructure — not use cases
    └── autonomous-ai-agents/
        ├── team-gbrain/          ← Shared knowledge brain for multi-agent teams
        └── team-gstack/          ← Shared infra orchestration for multi-agent teams
```

---

## How Each Use Case is Structured

Every use case folder contains three files:

| File | Purpose |
|------|---------|
| `README.md` | What this use case does, recommended schema, example interactions |
| `SETUP.md` | Instructions the agent follows once to set up the client's Notion workspace |
| `SKILL.md` | The agent's daily operating guide — behavioral rules, logic, and configuration |
| `config-template/env-template.txt` | Environment variable template to copy into `~/.hermes/.env` |

---

## Use Cases

### Growth

| Use Case | What it does |
|----------|-------------|
| [Lead & Partners Nurturing](./use-cases/growth/lead-nurturing/) | Conversational CRM — log activities, track pipeline, daily briefing, draft follow-up messages, import business cards |
| [Content Intelligence](./use-cases/growth/content-intelligence/) | Monitor sources, extract insights, generate on-brand content, save to content library |

### External Ops

| Use Case | What it does |
|----------|-------------|
| [Inventory Management](./use-cases/external-ops/inventory-management/) | Log inbound and outbound shipments via conversation, auto-update stock levels, analyze dealer performance and sales trends |

### Internal Ops

| Use Case | What it does |
|----------|-------------|
| [HR Management](./use-cases/internal-ops/hr-management/) | Employee directory, attendance and leave queries, approval processing, payroll and expense tracking |
| [Financial Intelligence](./use-cases/internal-ops/financial-intelligence/) | Expense logging, budget tracking, financial summaries, invoice and payment status |
| [Discussion & Todo Tracker](./use-cases/internal-ops/discussion-todo-tracker/) | Log meetings in plain language, extract todos and open issues, auto-update Business Core when strategy shifts |

---

## How to Install a Use Case

### 1. Copy the folder

```bash
cp -r use-cases/internal-ops/discussion-todo-tracker ~/.hermes/use-cases/internal-ops/
```

### 2. Fill in the environment template

```bash
cat use-cases/internal-ops/discussion-todo-tracker/config-template/env-template.txt >> ~/.hermes/.env
# Open ~/.hermes/.env and fill in the database IDs and tokens
```

### 3. Run SETUP.md with your agent

Open `SETUP.md` and follow the instructions with your agent. Setup covers:
- Discovering existing Notion databases
- Creating databases if they don't exist yet
- Verifying relations between databases
- Collecting and confirming all config values

Run setup once per client. It only needs to be repeated if the workspace is restructured.

### 4. Use daily via SKILL.md

Point your agent at `SKILL.md` for daily operations. The skill file contains all behavioral rules — how the agent should handle input, what to create, what to confirm before writing, and how to track status.

---

## Data Foundation

The `data-foundation/` folder is for **human reference only** — it contains schema documentation, setup guides, and table definitions. Agents do not read from this folder during normal operation.

Use it when:
- Planning a new deployment and deciding which tables are needed
- Onboarding a new client and explaining the data architecture
- Checking the recommended schema for a data layer before creating databases in Notion

| Folder | What it covers |
|--------|---------------|
| `core-business/` | 5 Business KB templates: Brand Identity, Target Audience, Offer, Business Model, Operations |
| `growth/` | Growth data tables: Accounts, Contacts, Activities, Deals, Partnership, Content Library |
| `internal-ops/` | Internal data tables: Discussions, Tasks, HR, Finance |

---

## Infrastructure Skills

The `skills/autonomous-ai-agents/` folder is separate infrastructure for advanced multi-agent deployments. These are not use cases.

| Skill | What it does |
|-------|-------------|
| [team-gbrain](./skills/autonomous-ai-agents/team-gbrain/) | Shared knowledge brain — agents read and write a common GBrain instance |
| [team-gstack](./skills/autonomous-ai-agents/team-gstack/) | Shared infra orchestration — agents coordinate on a common GStack environment |

Install these when deploying Hermes across a team. Not needed for single-user setups.

---

> Maintained by [BusyCow](https://busycow.ai)
