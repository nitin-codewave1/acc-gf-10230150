---
id: agent-ba/web_app/discovery/discovery.md
type: discovery
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.1.0
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-ba/web_app/discovery/discovery.md
depends_on: []
commit_kind: append
confidence: agent
run_id: 71dfd526-72b5-5ec5-9c38-f65b83c3b709
okf_id: DISCOVERY
---

---
id: DISCOVERY
type: discovery
status: active
created_at: 2026-10-22T20:27:19.342342+00:00
rig: ba
---

# Discovery

**Brief quality:** sufficient (problem: strong, outcomes: strong, stakeholders: weak)

**Declaration:** Brief quality sufficient - no declaration needed.

## Problem
A registered user needs a way to sign in to the web app using email and password.

**Root cause:** Without authentication, the app cannot gate access to protected areas (e.g. /home); unauthenticated access risks data exposure and the app cannot distinguish legitimate users from anonymous traffic.

**Boundary:** Most affected: registered users needing routine access and the security/engineering team accountable for breach risk. Least affected: new users (sign-up out of scope) and users needing SSO/MFA (out of scope).

**Prior attempts:** None implied; this is a greenfield login feature (no existing auth endpoint, cookie, or lockout logic mentioned).

## Outcome
**Before:** No authenticated entry point exists; there is no POST /api/login, no session cookie, and no brute-force protection.

**After:** Registered users can submit email/password at a login page, receive a secure, time-bound session cookie on success, and are redirected to /home; invalid attempts are rejected generically and repeated failures trigger a temporary lockout.

**Measurement:** At 60 days: % of login attempts by valid credential holders that succeed and redirect to /home; % of brute-force attempt sequences (5 failures) correctly resulting in a 15-min lockout; zero incidents of field-specific error disclosure.

**Impact index:** Registered users gain a working, secure gate to the app; this is the single change that enables all authenticated functionality downstream.

## Stakeholders
**Decision maker:** Product/engineering lead shipping the MVP auth surface; success means a functioning, secure login that unblocks all other authenticated features without introducing breach risk.

**Champion:** Backend/security-conscious engineer implementing POST /api/login who is accountable for bcrypt validation, cookie flags, and lockout correctness.

**End user:** A registered user on a Tuesday afternoon, multitasking, typing from memory or a password manager; may mistype the password once or twice, expects a quick redirect to /home, and would be confused or frustrated by a non-generic error or an unexplained lockout.
