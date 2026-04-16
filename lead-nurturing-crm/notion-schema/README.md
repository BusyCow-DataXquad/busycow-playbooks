# Notion Schema — Lead Nurturing CRM

## Required Databases

You need to create 3 databases in Notion and connect them with relations.

---

### 1. Accounts

| Property | Type | Notes |
|----------|------|-------|
| Name | Title | Company name |
| Stage | Status | Lead / Contacted / Qualified / Closing / Customer / Lost |
| Industry | Select | |
| Website | URL | |
| Notes | Rich Text | |

---

### 2. Contacts

| Property | Type | Notes |
|----------|------|-------|
| Name | Title | Contact person |
| Account | Relation → Accounts | Link to company |
| Role | Select | CEO / Sales / IT / Other |
| Email | Email | |
| Phone | Phone | |
| LINE / WeChat | Rich Text | |

---

### 3. Activities

| Property | Type | Notes |
|----------|------|-------|
| Summary | Title | One-line description |
| Type | Select | Call / Meeting / Email / Demo / Follow-up |
| Date | Date | |
| Account | Relation → Accounts | |
| Contact | Relation → Contacts | |
| Client Response | Rich Text | What the client said |
| Next Action | Rich Text | What to do next |
| Stage Advanced? | Checkbox | Did this move the deal forward? |
| Owner | People | Who handled this |

---

## After Creating the Databases

1. Share each database with your Notion integration
2. Copy each database ID from the URL:
   `https://www.notion.so/YOUR-DATABASE-ID?v=...`
3. Paste the IDs into your `config.yaml`
