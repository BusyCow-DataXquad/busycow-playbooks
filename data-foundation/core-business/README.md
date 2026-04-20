# Core Business KB

The Core Business KB is a set of five fill-in-the-blank documents that give every AI agent
the business context it needs to operate. Without it, agents produce generic output that
sounds right but misses the point.

**Required by: all use cases.**

---

## The Five Documents

| # | Document | What it answers |
|---|----------|----------------|
| 1 | [Brand Identity](./templates/1-brand-identity.md) | Who are we? What do we stand for? How do we sound? |
| 2 | [Target Audience](./templates/2-target-audience.md) | Who are we building for? What is their real problem? |
| 3 | [Offer](./templates/3-offer.md) | What exactly do we sell? What does the customer get? |
| 4 | [Business Model](./templates/4-business-model.md) | How do we make money? How do deals get done? |
| 5 | [Operations](./templates/5-operations.md) | How does the business actually run day-to-day? |

Templates are in English — you can fill in the answers in any language.

---

## How to Set Up

### Option A — Notion Pages (Recommended)

1. Create one Notion page per document inside your workspace
2. Copy the template content into each page and fill in your answers
3. Share all pages with your Notion integration
4. Add the page IDs to `~/.hermes/.env`:

```
KB_BRAND_IDENTITY_PAGE_ID=
KB_TARGET_AUDIENCE_PAGE_ID=
KB_OFFER_PAGE_ID=
KB_BUSINESS_MODEL_PAGE_ID=
KB_OPERATIONS_PAGE_ID=
```

### Option B — Local Markdown Files

```bash
mkdir -p ~/.hermes/kb
cp templates/*.md ~/.hermes/kb/
# Fill in your answers directly in each file
```

---

## Maintenance

Update the KB whenever the business changes its positioning, pricing, offer, or team structure.
An outdated KB is worse than no KB — the agent will confidently give wrong answers.
