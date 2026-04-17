# Business Core KB

> **This is mandatory. Install this before any other playbook.**

Before an AI agent can help with CRM, content, operations, or anything else —
it must first understand the business it is working for.

The Business Core KB is a set of five documents that together answer the
fundamental question: **what is this business, and how does it work?**

Every playbook in this repository depends on these documents. An agent without
a Business Core KB is a tool without context. It will produce generic output
that sounds right but misses the point.

---

## The Five Documents

| # | Document | What it answers |
|---|----------|----------------|
| 1 | [Brand Identity](./templates/1-brand-identity.md) | Who are we? What do we stand for? How do we sound? |
| 2 | [Target Audience](./templates/2-target-audience.md) | Who are we building for? What is their real problem? |
| 3 | [Offer](./templates/3-offer.md) | What exactly do we sell? What does the customer get? |
| 4 | [Business Model](./templates/4-business-model.md) | How do we make money? How do deals get done? |
| 5 | [Operations](./templates/5-operations.md) | How does the business actually run day-to-day? |

These five documents are designed to work for any type of business:
physical or digital, product or service, B2B or B2C, solo operator or team.

---

## Why English

The templates are written in English because precision matters here.
Ambiguous language in the KB produces ambiguous agent behavior.

That said, **you can fill in the answers in any language** — English, Traditional
Chinese, Simplified Chinese, or a mix. The agent will adapt.
The structure and questions stay in English to keep the intent clear.

---

## How to Set Up

### Option A — Notion Pages (Recommended)

1. Create one Notion page per document inside your workspace
2. Copy the template content into each page
3. Fill in your answers
4. Share all pages with your Notion integration
5. Add the page IDs to `~/.hermes/.env`:

```
KB_BRAND_IDENTITY_PAGE_ID=your_page_id
KB_TARGET_AUDIENCE_PAGE_ID=your_page_id
KB_OFFER_PAGE_ID=your_page_id
KB_BUSINESS_MODEL_PAGE_ID=your_page_id
KB_OPERATIONS_PAGE_ID=your_page_id
```

### Option B — Local Markdown Files

1. Copy the templates to `~/.hermes/kb/`
2. Fill in your answers directly in the `.md` files

```bash
mkdir -p ~/.hermes/kb
cp templates/*.md ~/.hermes/kb/
```

No `.env` configuration needed — the agent reads from `~/.hermes/kb/` automatically.

---

## How the Agent Uses This

Every agent action that involves communication, content, or judgment will
load the relevant KB documents first. The priority order:

1. **Brand Identity** — always loaded. Sets tone and boundaries for all output.
2. **Target Audience** — loaded for any customer-facing content or CRM action.
3. **Offer** — loaded when generating proposals, emails, or product descriptions.
4. **Business Model** — loaded for pricing, ROI calculations, and sales process.
5. **Operations** — loaded for internal workflow tasks and process automation.

---

## Maintenance

The Business Core KB should be treated as a living document.
Update it when:
- The business changes its positioning or pricing
- A new product or service is added
- The target audience shifts
- The team structure changes significantly

An outdated KB is worse than no KB — the agent will confidently give wrong answers.
