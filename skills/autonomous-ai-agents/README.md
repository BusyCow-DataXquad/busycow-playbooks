# Autonomous AI Agents — Skills

Team-aware orchestration skills for Hermes Agent.
These skills add organizational intelligence (RACI, Bus Factor, Decision Logging) on top of GStack and GBrain.

## Skills in this folder

| Skill | What it does |
|-------|-------------|
| team-gstack | Wraps GStack engineering workflows (office-hours, review, ship) with RACI checks, Mini Office Hours, and Bus Factor awareness |
| team-gbrain | Extends GBrain knowledge management with team entities — RACI memory, decision logs, skill gap detection, communication routing |

## Quick Install

```bash
# Clone the repo
git clone https://github.com/BusyCow-DataXquad/busycow-playbooks.git

# Copy skills into your Hermes skills folder
mkdir -p ~/.hermes/skills/autonomous-ai-agents
cp -r busycow-playbooks/skills/autonomous-ai-agents/team-gstack ~/.hermes/skills/autonomous-ai-agents/
cp -r busycow-playbooks/skills/autonomous-ai-agents/team-gbrain ~/.hermes/skills/autonomous-ai-agents/

# Copy TEAM_MANIFEST template to your project root and fill it in
cp busycow-playbooks/skills/autonomous-ai-agents/team-gbrain/templates/TEAM_MANIFEST.md ./TEAM_MANIFEST.md
```

## Prerequisites

- Hermes Agent installed and running
- For team-gbrain: GBrain installed (see team-gbrain/SKILL.md)
- For team-gstack: GStack + Claude Code installed (see team-gstack/SKILL.md)
- TEAM_MANIFEST.md filled in and placed in your project root

## Author

BusyCow / DataXquad — https://github.com/BusyCow-DataXquad
