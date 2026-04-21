---
name: team-gbrain
description: Team-aware GBrain knowledge management for Hermes Agent. Extends GBrain with organizational entity models — teams, RACI assignments, communication preferences, decision logs, Bus Factor tracking, and Skill Gap detection. Designed for small teams (3–20 people). Requires GBrain installed and a filled TEAM_MANIFEST.md.
version: 1.0.0
author: Hermes Agent + BusyCow
license: MIT
metadata:
  hermes:
    tags: [GBrain, Team, Knowledge-Management, Memory, RACI, Organization, MCP]
    related_skills: [team-gstack, native-mcp]
---

# Team GBrain — Organizational Knowledge Management

## What This Skill Does

This skill extends GBrain's personal knowledge model into a **team knowledge model**. It adds:

- **Team as a living entity** — the team is a first-class object with health metrics
- **RACI-aware memory** — every stored fact is tagged with who owns it
- **Decision logging** — decisions are stored with their accountability chain
- **Bus Factor tracking** — GBrain actively monitors single points of failure
- **Skill Gap detection** — compares incoming task requirements to team capabilities
- **Communication routing** — knows how each person prefers to receive information

---

## Prerequisites

- GBrain installed: follow instructions at https://raw.githubusercontent.com/garrytan/gbrain/master/INSTALL_FOR_AGENTS.md (agent-agnostic — no Hermes-specific section exists; the guide works for any agent)
- GBrain MCP server running: `gbrain serve`
- Hermes native-mcp skill configured to connect to GBrain
- A filled TEAM_MANIFEST.md (see template linked below)
- RESOLVER.md updated with team-gbrain triggers (see section below)

### MCP Configuration (add to Hermes config.yaml)

```yaml
mcp_servers:
  gbrain:
    command: gbrain
    args: [serve]
    transport: stdio
```

Or for remote GBrain (Railway/Render deployment):

```yaml
mcp_servers:
  gbrain:
    url: https://your-gbrain.railway.app/mcp
    transport: http
    headers:
      Authorization: "Bearer YOUR_GBRAIN_TOKEN"
```

---

## Team Knowledge Model

### Entity Types — GBrain Compatibility Note

GBrain's native PageType enum is:
`person | company | deal | yc | civic | project | concept | source | media | writing | analysis | guide | hardware | architecture`

There is NO native `team` or `decision` type. To add team entities without modifying GBrain source code, use these mappings:

| Team Concept | GBrain PageType to Use | Path Convention |
|--------------|----------------------|----------------|
| Team / Org | `company` | `org/teams/[team-name].md` |
| Domain ownership | `concept` | `org/domains/[domain-name].md` |
| Decision log | `concept` | `decisions/[YYYY-MM-DD]-[title].md` |
| Skill gap | `concept` | `skill-gaps/[domain]-[skill].md` |
| Bus risk | `concept` | `bus-risks/[component]-[owner].md` |
| Team member | `person` | `people/[firstname-lastname].md` (native) |

Tag-based filtering (since no custom types): add tags to each page for filtering:
- Team pages: tag `team`
- Decision pages: tag `decision`
- Skill gap pages: tag `skill-gap`
- Bus risk pages: tag `bus-risk`

This allows `list_pages` + `search` to filter by tag without touching GBrain's TypeScript source.

**Optional (requires GBrain code contribution):** Add `'team'` and `'decision'` to the PageType union in `src/core/types.ts` and submit a PR to garrytan/gbrain. This is cleaner long-term but not required to use this skill.

### Page Naming Convention

```
teams/[team-name].md
people/[firstname-lastname].md
domains/[domain-name].md
decisions/[YYYY-MM-DD]-[short-title].md
skill-gaps/[domain]-[skill].md
bus-risks/[component]-[owner].md
```

---

## RESOLVER.md — Required Updates

After installing team-gbrain, add these trigger rows to your GBrain `skills/RESOLVER.md`:

```markdown
Team & organizational awareness:
- "who is responsible for", "who owns", "RACI", "team manifest" -> skills/team-gbrain/SKILL.md
- "decision log", "log this decision", "who decided" -> skills/team-gbrain/SKILL.md
- "bus factor", "single point of failure", "knowledge risk" -> skills/team-gbrain/SKILL.md
- "skill gap", "team capability", "who can do" -> skills/team-gbrain/SKILL.md
- "weekly brain health", "team health check" -> skills/team-gbrain/SKILL.md
```

Without these entries, team-gbrain queries will fall through to `brain-ops` (the generic catch-all) and lose team-specific routing logic.

Note: RESOLVER.md is loaded into agent memory at install time. After editing it, re-read the file: "Re-read skills/RESOLVER.md and update your routing memory."

---

## Team Manifest Integration

On first use, ingest the TEAM_MANIFEST.md into GBrain:

```
1. Read TEAM_MANIFEST.md
2. For each team member:
   - Create or update people/[name].md with role, RACI map, skills, comm_prefs
3. Create teams/[team-name].md with mission, domains, current blockers
4. For each domain in the manifest:
   - Create domains/[domain].md with accountable person and dependencies
5. Run: gbrain embed --all to index all new pages
```

When TEAM_MANIFEST changes (role change, new hire, departure):

```
1. Update the relevant people/ and domains/ pages
2. Re-check all bus_risks/ pages for validity
3. Recalculate Bus Factor scores for affected components
4. Log the change as a decision: decisions/[date]-team-change.md
```

---

## RACI Memory Protocol

Every piece of knowledge stored in GBrain should carry RACI context:

**Page frontmatter template:**
```yaml
---
type: knowledge
domain: [domain-name]
accountable: [person-name]
responsible: [person-name]
consulted: [person1, person2]
informed: [person1, person2]
last_updated: YYYY-MM-DD
confidence: high|medium|low
---
```

When an agent stores a new fact or decision:

1. Identify which domain it belongs to
2. Look up that domain's RACI in GBrain
3. Tag the page with the correct RACI assignments
4. Route a notification to Informed parties if the fact is significant

---

## Decision Logging

Every significant decision must be logged. Trigger when:
- An architectural choice is made
- A process change is approved
- A priority is shifted
- A resource (person, budget, tool) is allocated

**Decision log format:**
```markdown
---
type: decision
date: YYYY-MM-DD
title: [Short title]
domain: [domain-name]
---

## Context
[Why this decision was needed]

## Decision
[What was decided]

## Decided By
[Name(s) — Accountable party who made the final call]

## Consulted
[Who was consulted before the decision]

## Rationale
[Key reasoning]

## Trade-offs
[What was accepted as a cost of this decision]

## Review Date
[When to revisit this decision]
```

Query decisions:
```
gbrain query "decisions in [domain] in the last 30 days"
gbrain query "decisions made by [person]"
gbrain query "open decisions pending review"
```

---

## Bus Factor Monitoring

**Bus Factor Score** = number of team members who must leave before a component becomes unrecoverable.

GBrain tracks this via `bus-risks/` pages. Maintain one page per at-risk component.

**Severity levels:**
| Score | Severity | Action |
|-------|----------|--------|
| 1 | 🔴 Critical | Immediate mitigation required |
| 2 | 🟡 High | Pair programming or documentation sprint this sprint |
| 3 | 🟢 Acceptable | Monitor quarterly |
| 4+ | ✅ Healthy | No action needed |

**Auto-detection triggers:**
- A person is tagged as sole Accountable on 3+ domains
- A deployment requires credentials only one person holds
- A component has no documentation and only one person has committed to it

**Mitigation strategies to suggest:**
1. Pair sessions — shadow the sole owner for 2+ hours
2. Documentation sprint — sole owner writes a runbook
3. Credential sharing — store in team password manager
4. Cross-training — rotate ownership of component

---

## Skill Gap Detection

When a new task or plan arrives:

1. Extract required skills from the task description
2. Query GBrain: `gbrain query "team skills in [domain]"`
3. Compare against required skills
4. For each gap, create or update: `skill-gaps/[domain]-[skill].md`
5. Suggest AI agent augmentation for gaps that can be automated

**Skill gap page format:**
```markdown
---
type: skill_gap
domain: [domain]
missing_skill: [skill name]
severity: critical|high|medium|low
identified_date: YYYY-MM-DD
---

## Gap Description
[What the team cannot currently do]

## Impact
[What this blocks or slows down]

## Mitigation Options
1. AI Agent: [Can Hermes/Claude Code cover this? How?]
2. Training: [Estimated time to upskill existing team member]
3. Hiring: [If gap is too large for current team]
4. Outsource: [If temporary or specialized]

## Status
open|mitigated|resolved
```

---

## Communication Preference Routing

Each person in GBrain has a `comm_prefs` field. When routing information:

1. Query: `gbrain get people/[name]` → read `comm_prefs`
2. Match urgency to channel:

| Urgency | Channel Options |
|---------|----------------|
| Immediate (blocks work) | Phone / WhatsApp / Slack DM |
| High (needs response today) | Preferred async channel |
| Normal (FYI) | Email / team channel post |
| Low (archival) | Log to GBrain, no active notification |

3. Always log the routing decision: who was notified, via which channel, and at what time

---

## GBrain MCP Tools Reference (Correct Tool Names)

⚠️ GBrain MCP tool names are exact — use these precisely or calls will fail.

| Correct Tool Name | Team Use Case |
|-------------------|--------------|
| `put_page` | Store a decision page, update a person's RACI page, log a bus risk |
| `get_page` | Retrieve a person's comm prefs, domain ownership, decision history |
| `search` | Keyword/FTS search — find decisions by domain, find who owns a component |
| `query` | Hybrid vector+keyword search — semantic team queries |
| `traverse_graph` | Graph traversal: "who depends on [person]?" — use max_depth=2 for performance |
| `add_link` | Connect a decision to its domain and accountable person |
| `list_pages` | List all open skill gaps, all bus risks above threshold |
| `add_timeline_entry` | Log a decision event to a person or domain's timeline |
| `get_timeline` | Retrieve decision history for a domain or person |
| `get_links` | Get all RACI relationships for a domain page |
| `get_backlinks` | Find all pages that reference a person (who depends on them) |
| `submit_job` | Queue background enrichment via Minions |

Full list of all 45 MCP tools: put_page, get_page, delete_page, list_pages, search, query, add_tag, remove_tag, get_tags, add_link, remove_link, get_links, get_backlinks, traverse_graph, add_timeline_entry, get_timeline, get_stats, get_health, get_versions, revert_version, sync_brain, put_raw_data, get_raw_data, resolve_slugs, get_chunks, log_ingest, get_ingest_log, file_list, file_upload, file_url, submit_job, get_job, list_jobs, cancel_job, retry_job, get_job_progress, pause_job, resume_job, replay_job, send_job_message, find_orphans, get_prompt, list_prompts, list_resources, read_resource

Example query pattern — before a planning session:
```
search("bus-risks")
search("skill-gaps severity:critical")
query("open decisions pending review")
traverse_graph("people/[name]", max_depth=2)
```

---

## Weekly Brain Health Check

Run this before weekly team syncs:

```
1. gbrain query "bus risks added or updated this week"
2. gbrain query "skill gaps opened this week"
3. gbrain query "decisions made this week"
4. gbrain query "team members with 3+ accountable domains" (overload check)
5. Generate summary: new risks, new gaps, decisions to review, people at capacity
```

Deliver summary to team lead via their preferred channel.

---

## Organizational Memory Lifecycle

| Event | GBrain Action |
|-------|--------------|
| New team member joins | Create people/[name].md, update domain RACIs, check Bus Factor improvements |
| Team member leaves | Flag all their bus risks as Critical, reassign domains, log as decision |
| New domain created | Create domains/[domain].md, assign RACI, check for skill gaps |
| Sprint starts | Log sprint goal as decision, tag relevant domains |
| Sprint ends | Update decision outcomes, archive resolved skill gaps |
| Architecture decision | Log to decisions/, link to affected domains |
| Incident / postmortem | Log to decisions/, update bus risks if applicable |

---

## Installation Notes (Linux / Hermes Agent)

GBrain's compiled binary (`bin/gbrain`) has a known path resolution bug on Linux when built with `bun build --compile` — it tries to open `/$bunfs/root/pglite.data` instead of the configured path. Use the source runner instead:

```bash
# Install
git clone --depth 1 https://github.com/garrytan/gbrain.git ~/gbrain
cd ~/gbrain && bun install && bun run build  # build needed for bin/ but don't use compiled binary on Linux

# Init database
cd ~/gbrain && bun run src/cli.ts init

# Hermes MCP config (use bun + source, NOT compiled binary)
# In ~/.hermes/config.yaml:
mcp_servers:
    gbrain:
        command: /path/to/bun          # e.g. ~/.hermes/node/bin/bun
        args: [run, /home/user/gbrain/src/cli.ts, serve]
        transport: stdio
```

Find your bun path: `which bun` or check `~/.hermes/node/bin/bun` if installed via npm.

GBrain config lives at `~/.gbrain/config.json`:
```json
{
  "engine": "pglite",
  "database_path": "/home/user/.gbrain/brain.pglite"
}
```

Without OpenAI key: vector/hybrid search is unavailable, but keyword search, knowledge graph, RACI pages, decision logs, and all MCP tools work fine. Sufficient for team orchestration use cases.

Doctor output "Health score 75/100 — relation pages does not exist" means `gbrain init` hasn't been run yet. Run it once to apply migrations.

---

## Pitfalls

1. **MCP tool names are exact** — use `put_page`, `get_page`, `search`, `query`, `traverse_graph`, `add_link` etc. — NOT `gbrain_put`, `gbrain_get` etc. Wrong names fail silently
2. **'team' and 'decision' are not native PageTypes** — use `company` + tag `team`, and `concept` + tag `decision` instead; or contribute a PR to add them to GBrain's types.ts
3. **RESOLVER.md must be updated** — without new trigger rows, team queries fall through to brain-ops and lose team-specific routing
4. **Compiled GBrain binary broken on Linux** — use `bun run src/cli.ts` instead of `bin/gbrain`; compiled binary misresolves PGLite path to `/$bunfs/root/pglite.data`
5. **Never store credentials in GBrain pages** — store credential locations (e.g., "in 1Password vault X"), not the credentials themselves
5. **RACI maps go stale fast** — set a quarterly review reminder for TEAM_MANIFEST and all domain pages
6. **Bus Factor alerts are not blame** — frame them as team health metrics, not individual performance issues
7. **Skill gaps ≠ people gaps** — many skill gaps can be filled by AI agents; evaluate before recommending hiring
8. **Decision log is not a meeting notes dump** — only log actual decisions with clear Accountable parties, not discussions
9. **Graph queries can be slow on large brains** — always add depth limits: `traverse_graph("people/[name]", max_depth=2)`
10. **org/ directory already exists in GBrain schema** — team pages should go in `org/teams/` not a new top-level `teams/` directory, to stay MECE with the existing recommended schema
