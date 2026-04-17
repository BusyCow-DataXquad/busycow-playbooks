# BusyCow Playbooks

Reusable Hermes Agent skill templates for SME use cases.

Each playbook is a self-contained package that includes:
- **Skills** — Hermes skill files that teach the agent how to operate
- **Notion Schema** — Database structure to import into the client's workspace
- **Config Template** — Settings to fill in (tokens, IDs, chat IDs)
- **Setup Guide** — Step-by-step instructions

---

## ⚡ Start Here — Required for All Playbooks

| Playbook | Description |
|----------|-------------|
| [business-core-kb](./business-core-kb/) | **Business Core KB — mandatory foundation. Install this first before any other playbook.** Defines who the business is, who it serves, what it sells, how it makes money, and how it runs. |

---

## Playbook Categories

| Category | Description |
|----------|-------------|
| **Sales Growth** | CRM, lead nurturing, outreach, pipeline management |
| **Internal Ops** | Internal workflows, HR, finance, operations |
| **External Ops** | Client-facing operations, delivery, support |
| **Personal Tools** | Individual productivity, personal assistants |

## Available Playbooks

| Playbook | Category | Description | Status |
|----------|----------|-------------|--------|
| [lead-nurturing-crm](./lead-nurturing-crm/) | Sales Growth | 客戶關係管理 (CRM / Lead Nurturing) — Notion + Telegram | ✅ Ready |
| [content-intelligence](./content-intelligence/) | Sales Growth | AI 內容智慧系統 — 監控外部來源、主動建議主題、生成品牌一致內容 | ✅ Ready |

---

## How to Install a Playbook

### 1. Copy skills to your Hermes instance

```bash
# Install a specific playbook's skills
cp -r lead-nurturing-crm/skills/* ~/.hermes/skills/productivity/

# Or clone the whole repo and symlink
git clone https://github.com/BusyCow-DataXquad/busycow-playbooks.git
```

### 2. Set up Notion schema

Follow the instructions in each playbook's `notion-schema/README.md`.

### 3. Configure your settings

Copy the `config-template.yaml` and fill in your own values:
- Notion API token
- Database IDs
- Telegram chat IDs

---

## Adding a New Playbook

Each playbook lives in its own folder:

```
my-new-playbook/
├── README.md
├── skills/
├── notion-schema/
└── config-template.yaml
```

---

> Maintained by [BusyCow](https://busycow.ai)
