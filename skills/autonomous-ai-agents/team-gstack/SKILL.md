---
name: team-gstack
description: Team-aware GStack orchestration for Hermes Agent. Extends GStack workflows (office-hours, review, ship) with RACI checks, information routing, Mini Office Hours, and Bus Factor awareness. Designed for small teams (3–20 people). Requires a filled TEAM_MANIFEST.md to activate.
version: 1.0.0
author: Hermes Agent + BusyCow
license: MIT
metadata:
  hermes:
    tags: [GStack, Team, RACI, Collaboration, Orchestration, Planning, Review]
    related_skills: [team-gbrain, claude-code, gstack]
---

# Team GStack — Collaborative Engineering Orchestration

## What This Skill Does

This skill wraps GStack workflows with **Organizational Awareness**. Before any planning, review, or ship action, the agent must:

1. Load the Team Manifest (who is on the team, their RACI, communication preferences)
2. Identify the Actor (who is making the request)
3. Check RACI alignment (is this person Responsible? Accountable? Do others need to be Consulted?)
4. Route information after completion (who needs to be Informed?)

Without a filled TEAM_MANIFEST.md, this skill operates in degraded mode (individual GStack behavior).

---

## Prerequisites

- GStack installed: `git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`
- Claude Code installed: `npm install -g @anthropic-ai/claude-code`
- A filled TEAM_MANIFEST.md in the project root (see template below)
- Hermes Agent with claude-code skill loaded

---

## Core Concept: Organizational Awareness Protocol

Before responding to any engineering request, internally resolve:

```
ACTOR IDENTIFICATION
  Who is making this request?
  What is their RACI status for this domain?
  Do they have authority to initiate this action?

IMPACT MAPPING
  Which team members are affected by this request?
  Are any domain owners / accountable parties not yet in the loop?
  Does this decision have downstream effects on other workstreams?

ROUTING DECISION
  Who must be Consulted before I act?
  Who must be Informed after I act?
  Is a Mini Office Hour needed?
```

---

## Modified GStack Workflows

### /office-hours (Team Mode)

Standard GStack /office-hours forces 6 clarifying questions. In team mode, add:

**Pre-flight RACI check:**
- Who is Accountable for the final decision on this feature?
- Have the Consulted parties (domain owners, affected teams) been notified?
- Is this scoped to one team member's domain, or does it cross boundaries?

**Trigger condition for Mini Office Hour:**
If the request spans more than one RACI domain, pause and surface:
> "This plan touches [Domain A] and [Domain B]. Before proceeding, should we loop in [Person X] (Accountable for Domain A)?"

### /plan-eng-review (Team Mode)

After architecture lock-in, add:

- **Bus Factor Check:** Is any single person the only one who understands this component? Flag it.
- **Skill Gap Check:** Does this plan require skills not currently on the team? Surface them.
- **Handoff Clarity:** If this work will be handed off, is the context sufficient for a new person to pick it up?

### /review (Team Mode)

Before starting code review:

1. Check if the Accountable person for this code domain has approved the approach
2. After review, identify: who needs to be Informed of the findings?
3. If critical issues found: route to Accountable party, not just the requester

### /ship (Team Mode)

⚠️ IMPORTANT: GStack /ship is a fully automated, non-interactive workflow by design. Do NOT add human confirmation steps inside /ship — this breaks its core contract.

Instead, team RACI checks happen BEFORE invoking /ship:

1. Run a pre-ship RACI check as a separate step: confirm Accountable person has approved the approach (via Telegram/Slack/async — outside the automated pipeline)
2. Once human confirmation is received, invoke /ship normally — let it run uninterrupted
3. After /ship completes: auto-notify Informed parties per TEAM_MANIFEST via their preferred channels

Pattern:
```
[Human confirms Accountable sign-off] → /ship (automated, uninterrupted) → [Agent notifies Informed parties]
```

Never inject interactive prompts into the /ship pipeline itself.

---

## Mini Office Hour Protocol

Trigger this when a collaboration gap is detected. Ask exactly three questions:

1. **Accountability gap:** "We have a plan — who is Accountable for the final sign-off on this?"
2. **Silo check:** "Are we discussing this in isolation? Should [Role/Person] be tagged here?"
3. **Mission alignment:** "Is this request aligned with our current sprint goals, or is it scope creep?"

Do not proceed until all three are answered.

---

## Organizational Awareness Rules

| Situation | Human Leads | AI Leads |
|-----------|-------------|----------|
| Ethical/cultural decisions | ✅ | — |
| Creative direction | ✅ | — |
| High-stakes personnel matters | ✅ | — |
| Code review execution | — | ✅ |
| Cross-team information routing | — | ✅ |
| RACI gap detection | — | ✅ |
| Release notes & documentation | — | ✅ |
| Conflict between two team members' priorities | ✅ | Balanced recommendation |

---

## Bus Factor Tracking

**Bus Factor** = number of people who must be hit by a bus before a project stalls.

During /plan-eng-review and /review, actively detect:
- Single points of knowledge (only one person understands module X)
- Single points of execution (only one person can deploy / has credentials)
- Single points of decision (only one person can approve Y)

When Bus Factor = 1 on a critical component, surface:
> "⚠️ Bus Factor Alert: [Component] has a single point of knowledge/execution. Recommend pairing or documentation sprint."

---

## Skill Gap Detection

During planning, compare task requirements against the Team Manifest skill inventory:

1. List skills required by the plan
2. Cross-reference with team skill map in TEAM_MANIFEST
3. For gaps: recommend AI agent augmentation or hiring/training flag
4. Do not silently proceed on a skill gap — surface it before estimating timeline

---

## GStack + Claude Code Delegation Pattern

When delegating a GStack skill via Claude Code:

```bash
# Inject team context into Claude Code session
claude -p "Load TEAM_MANIFEST.md and apply team-gstack protocol. Then run /office-hours for: [TASK DESCRIPTION]" \
  --append-system-prompt-file TEAM_MANIFEST.md \
  --allowedTools "Read,Edit,Bash" \
  --max-turns 20
```

For interactive sessions (multi-turn review cycles):

```bash
# Start tmux session with team context
tmux new-session -d -s gstack-team -x 200 -y 50
tmux send-keys -t gstack-team "cd [PROJECT_DIR] && claude --append-system-prompt-file TEAM_MANIFEST.md" Enter
sleep 5 && tmux send-keys -t gstack-team Enter  # Trust dialog
tmux send-keys -t gstack-team "Load TEAM_MANIFEST.md. We are in team mode. Run /review and apply RACI routing." Enter
```

### SPAWNED_SESSION Compatibility

If running inside an orchestrator (OpenClaw, Hermes delegate_task, etc.), the `$OPENCLAW_SESSION` env var may be set to "true". In this case:

- Skip ALL interactive prompts — auto-choose recommended options
- Skip upgrade checks and routing injection
- End each skill with a structured completion report
- Do NOT call AskUserQuestion — output decisions as assertions instead

Check at skill start:
```bash
SPAWNED=$(echo "${OPENCLAW_SESSION:-false}")
if [ "$SPAWNED" = "true" ]; then
  # Non-interactive mode: auto-proceed, report at end
fi
```

### AskUserQuestion Format (when interactive)

When not in SPAWNED_SESSION mode and a question must be asked, follow GStack's 4-part format:

1. **Re-ground** — restate project name + current branch
2. **Simplify** — plain English summary of the decision
3. **Recommend** — the recommended option with Completeness score (N/10)
4. **Options** — lettered A/B/C with estimated effort in human-hours and Claude Code turns

Example:
```
Project: [name] | Branch: [branch]
Decision needed: Should we gate /ship on Accountable confirmation?
Recommend: Option A (Completeness: 8/10)
A) Pre-ship RACI async check, then auto-ship (2h human, 1 CC turn) ← recommended
B) Full blocking confirmation inside /ship (breaks automation contract)
C) Skip RACI check for this ship, log it as a known gap
```

### Learnings Integration

At the start of each team-gstack skill run, check for prior learnings:

```bash
SLUG=$(basename $(git rev-parse --show-toplevel 2>/dev/null || echo "unknown"))
LEARN_FILE="${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG}/learnings.jsonl"
if [ -f "$LEARN_FILE" ] && [ $(wc -l < "$LEARN_FILE") -gt 5 ]; then
  ~/.claude/skills/gstack/bin/gstack-learnings-search --limit 3
fi
```

Log RACI and Bus Factor discoveries as learnings:
```bash
# When a Bus Factor risk is discovered:
gstack-learnings-log --type pitfall \
  --key "bus-factor-[component]" \
  --insight "[Person] is the sole owner of [component] — Bus Factor 1" \
  --confidence 9
```

---

## Interaction Style

- Use tables for RACI assignments and routing decisions
- Use checklists for pre-flight checks
- Flag Bus Factor and Skill Gap issues with ⚠️
- Always show routing decisions explicitly: "Informing [Person] via [Channel] after this action"
- When a Mini Office Hour is needed, state it clearly before proceeding

---

## Installation (Linux / Hermes Agent)

```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code
claude --version  # verify: 2.1.116+

# Install GStack
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack
bun install
bun run build  # required — compiles browse server and bin/
```

If `bun` is not in PATH, install via npm: `npm install -g bun`
On Hermes Agent (Railway/Linux), bun installs to `~/.hermes/node/bin/bun` — add to PATH or use full path.

GStack does NOT require authentication to install. Claude Code requires `ANTHROPIC_API_KEY` (already in `~/.hermes/.env`) or OAuth login via `claude auth login`.

---

## Pitfalls

1. **Do NOT inject confirmation steps into /ship** — /ship is fully automated by design. RACI sign-off must happen before invoking /ship, not inside it
2. **Do not proceed without RACI resolution** — if the Actor's authority is unclear, ask before acting
3. **Bus Factor alerts are not blockers** — surface them, but don't halt work; create a tracking item
4. **Skill gaps ≠ "can't do it"** — AI agents can fill many skill gaps; surface the gap and propose the solution
5. **RACI is per-domain, not per-person** — the same person can be Accountable in one domain and Informed in another
6. **Mini Office Hours should be brief** — 3 questions max, not a full planning session restart
7. **TEAM_MANIFEST changes must be versioned** — when roles change, update the manifest and re-run /learn to update agent memory
8. **SPAWNED_SESSION must be respected** — if running inside an orchestrator, never call AskUserQuestion; auto-proceed and report
9. **This skill wraps GStack, not replaces it** — GStack's own ETHOS.md, voice rules, and preamble bash block remain authoritative; this skill adds a team layer on top
