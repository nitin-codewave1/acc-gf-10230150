---
id: agent-developer/web_app/web-app/INTAKE.md
type: intake
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.5.0
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-developer/web_app/web-app/INTAKE.md
depends_on: []
commit_kind: fork
confidence: agent
run_id: 71dfd526-72b5-5ec5-9c38-f65b83c3b709
okf_id: web-app/INTAKE.md
---

<!-- SECTION: web-app.intake | status: VERIFIED -->
## What this run read

| Input | Role(s) | Parsed | Taken from it | How the role was decided |
|---|---|---|---|---|
| `design-brief.md` | design_brief | yes | 9,187 bytes, 1 identifier(s) | declared in INPUTS.yaml |
| `feedback.md` | feedback | yes | 47,046 bytes, 14 identifier(s) | declared in INPUTS.yaml |
| `requirements.md` | requirements | yes | 22,329 bytes, 15 identifier(s) | declared in INPUTS.yaml |
| `stories.md` | user_stories | yes | 15,792 bytes, 5 identifier(s) | declared in INPUTS.yaml |

## Inputs expected

| Role | Status |
|---|---|
| `requirements` | supplied (1 file(s)) |
| `user_stories` | supplied (1 file(s)) |
| `design_brief` | supplied (1 file(s)) |
| `platform` | not supplied — the run degrades, see GAPS.md |
| `api_docs` | not supplied — the run degrades, see GAPS.md |
| `feedback` | supplied (1 file(s)) |

## Identifiers

- Distinct identifiers found: **5**
- Of those, user stories: **5**
- With established acceptance detail: **5** (100%, floor 60%)
  - counted when a story carries inline criteria, **or** a `§` reference that resolves to a real requirements section
- Requirements sections available to resolve against: **0**

Roles were declared in `INPUTS.yaml`.
<!-- /SECTION: web-app.intake -->
