---
id: agent-developer/web_app/web-app/SELF-CHECK.md
type: self-check
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.4.0
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-developer/web_app/web-app/SELF-CHECK.md
depends_on: []
commit_kind: fork
confidence: agent
run_id: c12a20be-3d1f-50a3-b992-25fc1531fe29
okf_id: web-app/SELF-CHECK.md
---

<!-- SECTION: web-app.self-check | status: VERIFIED -->
## Verification pass

| # | Check | Result | Detail |
|---|---|---|---|
| 1 | anchors open and close; ids unique and well-formed | **PASS** | 12 section(s); all 12 sections present, in order |
| 2 | every referenced id resolves | **PASS** | every id cited carries this run's surface code |
| 3 | every minted id carries this surface's code | **PASS** | 19 minted, all carrying `WEB` |
| 4 | no VERIFIED section contains an uncited claim | **NOT RUN** | a judgement about prose; the document's own verification pass covers it |
| 5 | locked sections reproduced byte-for-byte | **PASS** | the document is written through the locking engine |
| 6 | every story this document answers for appears in the coverage table | **PASS** | 4 in scope, 0 not found in the document |
| 7 | no schema is restated | **NOT RUN** | a judgement about prose, not a pattern this module can assert |
| 8 | no file written outside output_root | **PASS** | every write is jailed to `/tmp/qbit/specialist/c12a20be-3d1f-50a3-b992-25fc1531fe29/002_325c3d2f-99ab-4396-bf87-8a8b58158084-web_app-agent-developer_agent-developer/devagent/output/web-app` |
| 9 | every Q- has a class; every GAP- names its closing artefact | **NOT RUN** | 11 GAP- id(s) are defined in the specification's prose; whether each names a closing artefact is the document's own verification pass |
| 10 | no GAP- raised for a condition that cannot occur | **NOT RUN** | a judgement about the condition, not the id |

**6 passed, 0 failed, 4 not run.**

A check marked `NOT RUN` has nothing to check yet at this milestone — no specification sections exist. It is reported as not run rather than as a pass: reporting a pass that was never performed is invariant I-2 broken against ourselves.
<!-- /SECTION: web-app.self-check -->
