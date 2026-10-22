---
id: agent-developer/web_app/web-app/WEB-APP-SPEC.md
type: spec
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.1.1
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-developer/web_app/web-app/WEB-APP-SPEC.md
depends_on: [US-001-01, US-001-05, US-004-01, US-004-04, US-004-02, US-004-03, US-001-06, US-002-01, US-002-04, US-003-01, US-003-05, US-001-02, US-001-03, US-001-04, US-002-02, US-002-03, US-003-02, US-003-03, US-003-04]
commit_kind: append
confidence: agent
run_id: 5afbad28-d83e-5d9d-acae-b9d73ed5f6ad
okf_id: web-app/WEB-APP-SPEC.md
---

<!-- SECTION: web-app.overview | status: DRAFT-UNVERIFIED -->
## Overview

**Purpose.** The Web App provides the end-user sign-in experience for registered account holders: a login form, submission to the server, display of outcomes, and entry to `/home` on success [US-001-01, US-001-05]. Credential validation, lockout state, cookie issuance and failure-response construction are server-side and out of this surface's scope [BR-001, BR-005, BR-006, NFR-012, US-004-01].

**Primary actor.** Registered account holder (end user) [US-001-01 through US-004-04]. No other actor appears in the supplied stories.

**Out of scope for this surface.**
- Credential validation of any kind, including client-side pre-checks that compare or hash credentials [BR-001, NFR-011].
- Lockout counting, lockout timing and lockout clearing [BR-005, BR-007, NFR-003, NFR-012].
- Setting, reading or inspecting session cookies. The cookie is HttpOnly and server-issued [BR-009, US-004-02].
- Admin and operator flows. No admin story is supplied in this run.
- Any account-recovery, registration or password-reset flow. None is supplied.

**Run status and degradation.**
- Halt conditions present: `HALT-CRITICAL-CONTRADICTION` (CON-WEB-001) and an unmet contract precondition (`contract_source` is null; GAP-WEB-001). Sections depending on either are `BLOCKED`.
- Design coverage: `BEHAVIOUR-ONLY`. The supplied `design-brief.md` contains acceptance criteria, not visual design, and Figma MCP is not connected (GAP-WEB-002). No component inventory, tokens or frame IDs are asserted.

**Open items raised in this document.**
- CON-WEB-001 (Critical): lockout counting rule. Blocking question Q-WEB-001.
- CON-WEB-002 (non-critical): empty-field submission behaviour.
- Q-WEB-002 (degradable): email matching. Default: exact match [AC-001-01-02].
- Q-WEB-003 (degradable): login route path. See GAP-WEB-005.
- ASM-WEB-001: `POST /api/login` is the endpoint path stated in the stories. It is not pinned to a contract.
- ASM-WEB-002: unauthenticated access to protected routes redirects to the login page. The stories do not name the destination.
- GAP-WEB-001 through GAP-WEB-009: see `GAPS.md` entries cited in each section.

<!-- /SECTION: web-app.overview -->
<!-- SECTION: web-app.routing | status: DRAFT-UNVERIFIED -->
## Routing

| Route | Page | Access | Source |
|---|---|---|---|
| `<login-route>` (path not supplied, GAP-WEB-005) | Login page: email field, password field, "Sign in" button | Public | [AC-001-05-01] |
| `/home` | Post-login landing | Authenticated only | [AC-001-01-01], [BR-002], [US-004-01] |
| Any authenticated page (none enumerated) | Protected content | Authenticated only; expired session treated as unauthenticated | [AC-001-04-02], [US-004-03] |

**Layout nesting.** Not specified in any input (GAP-WEB-005). The login page is the only page with stated content.

**Protected-route rules.**
- A protected route must not render protected content unless the server confirms a valid session [US-004-03]. The client cannot read the session cookie to determine this (see `web-app.state-management`).
- An expired or missing session on a protected route means the user must sign in again [AC-001-04-02].

**Unauthenticated redirect.** Redirect to the login route, with the login route as the destination [ASM-WEB-002]. The stories do not specify a return-to parameter, so post-login navigation after an intercepted deep link is unspecified (GAP-WEB-006).

**Post-login redirect.** On success, redirect to `/home` [AC-001-01-01, BR-002, US-004-01].

<!-- /SECTION: web-app.routing -->
<!-- SECTION: web-app.module-map | status: VERIFIED -->
## Module map

All supplied stories belong to four epics, each mapped to one feature area of this surface.

| Feature area | Epic | Stories | Route ownership |
|---|---|---|---|
| Sign-in form and submission | EP-001 | US-001-01, US-001-05, US-001-06 | Login route |
| Failure messaging | EP-002 | US-002-01 through US-002-04 | Login route (inline display) |
| Lockout messaging | EP-003 | US-003-01 through US-003-05 | Login route (inline display; counting is server-side) |
| Session handling after sign-in | EP-004 | US-004-01 through US-004-04 | `/home` and protected routes; login route for redirect |

Module-level specifications have not been produced in this run (GAP-WEB-009). Each feature area above maps to one future module spec.

<!-- /SECTION: web-app.module-map -->
<!-- SECTION: web-app.state-management | status: DRAFT-UNVERIFIED -->
## State management

**Client state (owned by the browser).**
- Email and password field values, held only for the duration of the form [US-001-05].
- Submission-in-flight indicator.
- Inline error message for the most recent failure.
- Lockout message, when the server returns it [US-003-05].

**Prohibited client storage.**
- The password must never be persisted to any client store. This follows from "plaintext passwords are never logged or stored" [NFR-001].
- The client must not hold any lockout state. Lockout state is server-side, and must not rely on cookies or local storage [NFR-012].
- The client must not reconstruct a session token or attempt to determine session validity from the cookie. The session cookie is HttpOnly and not readable by script [BR-009, US-004-02].

**Server state (owned by the backend, consumed here).**
- Session validity is established only by server responses [US-004-03, BR-009]. The web app must therefore derive authenticated status from server responses, not from cookie inspection.
- The endpoint or response shape the web app uses to establish authenticated status is not stated in any input (GAP-WEB-001). This is a dependency on the contract.

**Cache and invalidation policy.** Not specified (GAP-WEB-006). The stories do not state what is cached after login, when cached identity is invalidated, or how an expired session is detected mid-navigation.

<!-- /SECTION: web-app.state-management -->
<!-- SECTION: web-app.auth-flows | status: BLOCKED -->
## Auth flows

**Reason.** Blocked by CON-WEB-001 (Critical): the lockout counting rule is contradictory across inputs. Every flow that displays lockout state, and every flow whose outcome depends on it, depends on that rule. Also blocked by GAP-WEB-001 (no contract source) for the endpoint and response shape.

**Unblock.** Answer Q-WEB-001 (lockout counting rule: consecutive with reset on success, or a 15-minute window). Then supply an approved backend contract pinned for this run (`contract_source` and `contract_path`). The author will then write login, refresh, session expiry, logout and mid-flow expiry flows. Refresh, logout and mid-flow expiry are not in the stories (GAP-WEB-006) and require a supplied story before they can be specified.

<!-- /SECTION: web-app.auth-flows -->
<!-- SECTION: web-app.authorization-ui | status: DRAFT-UNVERIFIED -->
## Authorization UI

**Roles that can sign in to this surface.**

| Role | Routes reachable | Visible navigation | Enabled actions |
|---|---|---|---|
| Anonymous visitor (pre-authentication) | Login route only | Not specified (GAP-WEB-006) | "Sign in" button enabled on render [AC-001-05-02] |
| Registered account holder | `/home` and protected routes | Not specified (GAP-WEB-006) | Not specified beyond the sign-in action |

**Excluded roles.** Operator and administrator roles are excluded. They have no route on this surface and no story in this run. Listing them would be speculation about another surface (invariant I-1). The backend role matrix is not supplied, so this exclusion is by absence of any web-app story, not by reference to the matrix.

**Permission-gated actions.** None are supplied. The only gated action is access to protected routes, governed by session validity [US-004-03].

<!-- /SECTION: web-app.authorization-ui -->
<!-- SECTION: web-app.design-system | status: BLOCKED -->
## Design system

**Reason.** No visual design input is supplied. The `design-brief.md` file contains acceptance criteria only, and Figma MCP is not connected [GAP-WEB-002]. No tokens, type scale, spacing or breakpoints can be cited.

**Unblock.** Supply a design brief containing visual design, or connect Figma MCP. Then supply design tokens and a type scale, and reverify this section against them.

<!-- /SECTION: web-app.design-system -->
<!-- SECTION: web-app.responsive | status: BLOCKED -->
## Responsive

**Reason.** No breakpoint or layout input is supplied. Breakpoints and layout changes must be cited from the design, and no design input exists (GAP-WEB-002, GAP-WEB-007).

**Unblock.** Supply the design input described in GAP-WEB-002, including breakpoint definitions and layout behaviour at each breakpoint.

<!-- /SECTION: web-app.responsive -->
<!-- SECTION: web-app.browser-support | status: BLOCKED -->
## Browser support

**Reason.** No browser or version requirement is supplied in any input (GAP-WEB-003). No browser or version may be assumed.

**Unblock.** Supply the supported browser list with minimum versions, as a requirements input.

<!-- /SECTION: web-app.browser-support -->
<!-- SECTION: web-app.accessibility | status: DRAFT-UNVERIFIED -->
## Accessibility

**Target standard.** WCAG 2.1 AA for the login page [NFR-010].

**Requirements cited.**
- Login form fields, button and error messages must be keyboard-navigable and screen-reader accessible [NFR-010].
- Form presents email field, password field and Sign in button with clear focus states and inline error display [NFR-009].
- Error message rendered within 1 second of receiving a failed response [NFR-009].
- Empty-field handling is under CON-WEB-002 and not stated here.

**Verification.** Not specified (GAP-WEB-008). The stories do not name a verification method, tool or test. A proposed approach (automated accessibility scan plus manual screen-reader pass) is an assumption, not a citation, and is not adopted here until supplied.

<!-- /SECTION: web-app.accessibility -->
<!-- SECTION: web-app.performance | status: DRAFT-UNVERIFIED -->
## Performance

**Cited requirements.**
- `POST /api/login` responds within p95 ≤ 500 ms for both success and failure paths under normal load [NFR-006]. This is a backend-owned target observed from the client.
- Login service availability ≥ 99.9% monthly [NFR-008]. Backend-owned.
- Login endpoint must support ≥ 100 concurrent requests [NFR-007]. Backend-owned.

**Bundle budget.** Not supplied (GAP-WEB-004). No bundle size may be asserted.

**Route-level code splitting.** Not specified (GAP-WEB-004).

**Largest-payload route.** Cannot be determined. The login page is the only page with stated content, but no payload size is supplied (GAP-WEB-004).

<!-- /SECTION: web-app.performance -->
<!-- SECTION: web-app.coverage | status: VERIFIED -->
## Coverage

Each supplied story is mapped to the project-level section that addresses it. Statuses:
- **Covered**: addressed in this document.
- **Partial**: addressed in part; the remainder is in a blocked section or a deferred module spec.
- **Backend-owned**: the behaviour is enforced server-side. The web app's obligation is limited to rendering the response. Tracked under GAP-WEB-009.

| Story | Section(s) | Status | Note |
|---|---|---|---|
| US-001-01 | `routing`; `auth-flows` | Partial | Redirect to `/home` covered; `auth-flows` blocked (CON-WEB-001, GAP-WEB-001) |
| US-001-02 | `auth-flows` | Partial | Generic message display depends on blocked section |
| US-001-03 | `auth-flows` | Partial | Lockout display depends on CON-WEB-001 |
| US-001-04 | `state-management` | Covered (client side) | Cookie attributes are server-set [BR-009]; client cannot read cookie |
| US-001-05 | `routing`; `accessibility` | Partial | Form content deferred to module spec (GAP-WEB-009); login route path GAP-WEB-005 |
| US-001-06 | `overview` | Covered | Web app performs no credential validation [BR-001] |
| US-002-01 | `auth-flows` | Partial | Blocked section |
| US-002-02 | `auth-flows` | Partial | Blocked section; empty-field behaviour under CON-WEB-002 |
| US-002-03 | — | Backend-owned | `Set-Cookie` absence on failure is a backend response property (GAP-WEB-009) |
| US-002-04 | — | Backend-owned | QA verification of backend responses (GAP-WEB-009) |
| US-003-01 | — | Backend-owned | Counting is server-side [BR-005, NFR-012] (GAP-WEB-009); rule under CON-WEB-001 |
| US-003-02 | `auth-flows` | Partial | Blocked on CON-WEB-001 |
| US-003-03 | `auth-flows` | Partial | Blocked on CON-WEB-001 |
| US-003-04 | — | Backend-owned | Per-account counting [BR-005] (GAP-WEB-009) |
| US-003-05 | `auth-flows` | Partial | Message display blocked |
| US-004-01 | `routing` | Partial | `/home` redirect covered; cookie attributes server-set |
| US-004-02 | `state-management` | Covered | Client cannot read cookie [BR-009] |
| US-004-03 | `routing`; `auth-flows` | Partial | Protected-route rule covered; expiry handling blocked |
| US-004-04 | `auth-flows` | Partial | No-cookie-on-failure is backend-owned [BR-004]; UI display blocked |

**Uncovered-story count.** Zero stories are uncovered. Eight stories are partial and blocked by CON-WEB-001 or GAP-WEB-001. Four are backend-owned and tracked under GAP-WEB-009. Module-level specifications are deferred for all stories (GAP-WEB-009).

**Gaps and contradictions referenced.**
- CON-WEB-001 (Critical): lockout counting rule. Affects US-003-01 through US-003-03, US-003-05, US-001-03. Question Q-WEB-001 (blocking).
- CON-WEB-002 (non-critical): empty-field submission behaviour. Affects US-001-01, US-002-02, US-001-02. Defaults to client-side prevention [AC-001-01-03]. Server behaviour for an empty submission returns the generic error [US-002-02]. Marked DRAFT-UNVERIFIED.
- GAP-WEB-001 through GAP-WEB-009: see `GAPS.md`.

<!-- /SECTION: web-app.coverage -->
