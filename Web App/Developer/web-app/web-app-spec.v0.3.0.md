---
id: agent-developer/web_app/web-app/WEB-APP-SPEC.md
type: spec
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.3.0
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-developer/web_app/web-app/WEB-APP-SPEC.md
depends_on: [US-001-01, US-001-05, US-004-05, US-004-02, US-001-03, US-002-02, US-001-04, US-004-01, US-004-03, US-001-06, US-002-01, US-002-04, US-003-01, US-003-05, US-001-02, US-002-03, US-003-02, US-003-03, US-003-04, US-004-04]
commit_kind: fork
confidence: agent
run_id: b3458417-e9fc-5523-818c-5a07db7110ad
okf_id: web-app/WEB-APP-SPEC.md
---

<!-- SECTION: web-app.overview | status: DRAFT-UNVERIFIED -->
## Overview

**Purpose.** The Web App provides the end-user sign-in experience for registered account holders: an email field, a password field, a "Remember me" checkbox (unchecked by default), a "Sign in" button, submission to the server, display of outcomes, and entry to `/home` on success [US-001-01, US-001-05, US-004-05, CR-100]. Credential validation, lockout state, cookie issuance, cookie lifetime and failure-response construction are server-side and outside this surface [BR-001, BR-005, BR-006, BR-009, NFR-011, NFR-012].

**Primary actor.** Registered account holder (end user) [US-001-01 through US-004-05]. No other actor appears in the supplied stories.

**Out of scope for this surface.**
- Credential validation of any kind, including client-side pre-checks that compare or hash credentials [BR-001, NFR-011].
- Lockout counting, lockout timing and lockout clearing [BR-005, BR-007, NFR-003, NFR-012].
- Setting, reading or computing session cookies, including `Max-Age` and expiry. Cookies are HttpOnly and server-issued [BR-009, US-004-02, US-004-05].
- Operator and administrator flows. No such story is supplied for this surface.
- Registration, account recovery and password reset. None is supplied.

**Run status and degradation.**
- **Contract precondition unmet.** `contract_source` is null, so no approved backend contract is pinned for this run (GAP-WEB-001). Under the web role's precondition this run would stop. This document is emitted in partial form so the requirements, contradictions and gaps can be reviewed. Every section that depends on an endpoint or response shape is `BLOCKED`.
- **Critical contradictions.** CON-WEB-001 (lockout counting rule) and CON-WEB-003 (Remember me session lifetime) are Critical under core rule 8. Both block `web-app.auth-flows`. Under the halt rule, the run halts at the end of intake until they are resolved.
- **Design coverage: BEHAVIOUR-ONLY.** `design-brief.md` contains acceptance criteria only and no visual design. Figma MCP is not connected (GAP-WEB-002). No component inventory, tokens or frame IDs are asserted.
- **Platform.** No `platform` input was supplied (GAP-WEB-011). No platform fact is assumed.

**Change request applied.** CR-100 (consumer, lane web_app) is applied in this document. It adds a "Remember me" checkbox to the login page. When checked, the session cookie persists for 30 days (`Max-Age=2592000`). The default is unchecked. CR-100 conflicts with BR-009 and NFR-004, which state an unconditional 8-hour expiry (CON-WEB-003).

**Open items raised in this document.**
- **CON-WEB-001 (Critical).** Lockout counting rule. US-001-03 and AC-001-03-01 count failures "within 15 minutes". BR-005, BR-006 and NFR-003 count consecutive failures with no window. Blocking question Q-WEB-001.
- **CON-WEB-002 (non-critical).** Empty-field submission. AC-001-01-03 requires the Sign in control to prevent submission of a missing password. AC-001-02-03 and US-002-02 require a generic server-side error for an empty email. Marked DRAFT-UNVERIFIED in the sections that cite it.
- **CON-WEB-003 (Critical).** Remember me session lifetime. BR-009 and NFR-004 state an unconditional 8-hour expiry. US-001-04 (AC-001-04-02), US-004-01, US-004-03, US-004-05 and CR-100 state a 30-day lifetime when Remember me is checked. Unblock: amend BR-009 and NFR-004 to state the 30-day case as an explicit exception, or withdraw CR-100. The 8-hour reading means the Remember me feature is not built. The 30-day reading changes a security target without a matching amendment to the security NFR.
- **Q-WEB-001 (blocking).** Which lockout counting rule applies: consecutive failures with reset on success (BR-005), or a 15-minute window (AC-001-03-01)?
- **Q-WEB-002 (degradable).** Email matching. Default: exact match, as stated in AC-001-01-02.
- **Q-WEB-003 (degradable).** Login route path. Default: none assumed. See GAP-WEB-005.
- **ASM-WEB-001.** `POST /api/login` is the endpoint path stated in AC-001-06-01 and US-002-02. It is not pinned to a contract.
- **ASM-WEB-002.** Unauthenticated access to a protected route redirects to the login route. The stories do not name the destination.
- **GAP-WEB-001 through GAP-WEB-011.** Defined in the sections that cite them.

<!-- /SECTION: web-app.overview -->
<!-- SECTION: web-app.routing | status: DRAFT-UNVERIFIED -->
## Routing

| Route | Page | Access | Source |
|---|---|---|---|
| `<login-route>` (path not supplied, GAP-WEB-005) | Login page: email field, password field, "Remember me" checkbox (unchecked by default), "Sign in" button | Public | AC-001-05-01, AC-001-05-03, US-004-05, CR-100 |
| `/home` | Post-login landing | Authenticated only | AC-001-01-01, BR-002, US-004-01 |
| Any authenticated page (none enumerated) | Protected content | Authenticated only; an expired session is treated as unauthenticated | AC-001-04-02, US-004-03 |

**Layout nesting.** Not specified in any input (GAP-WEB-005). The login page is the only page with stated content.

**Protected-route rules.**
- A protected route must not render protected content unless the server confirms a valid session [US-004-03]. The client cannot determine this from the cookie (see `web-app.state-management`).
- An expired or missing session on a protected route means the user must sign in again [AC-001-04-02].

**Unauthenticated redirect.** Redirect to the login route [ASM-WEB-002]. The stories do not specify a return-to parameter, so post-login navigation after an intercepted deep link is unspecified (GAP-WEB-006).

**Post-login redirect.** On success, redirect to `/home` [AC-001-01-01, BR-002, US-004-01].

<!-- /SECTION: web-app.routing -->
<!-- SECTION: web-app.module-map | status: VERIFIED -->
## Module map

All supplied stories belong to four epics. Each maps to one feature area of this surface.

| Feature area | Epic | Stories | Route ownership |
|---|---|---|---|
| Sign-in form and submission | EP-001 | US-001-01, US-001-05, US-001-06 | Login route |
| Failure messaging | EP-002 | US-002-01 through US-002-04 | Login route (inline display) |
| Lockout messaging | EP-003 | US-003-01 through US-003-05 | Login route (inline display; counting is server-side) |
| Session handling after sign-in | EP-004 | US-004-01 through US-004-05 | Login route (Remember me checkbox, US-004-05); `/home` and protected routes; login route for redirect |

Module-level specifications are not produced in this run (GAP-WEB-009). Each feature area maps to one future module spec.

<!-- /SECTION: web-app.module-map -->
<!-- SECTION: web-app.state-management | status: DRAFT-UNVERIFIED -->
## State management

**Server-state versus client-state boundary.** Credentials, lockout state and session validity are server-owned [BR-001, NFR-012, US-004-03]. The client owns only form state, transient UI state and the rendering of server responses.

**Client state (owned by the browser).**
- Email and password field values, held only for the duration of the form [US-001-05].
- Remember me checkbox value. It is unchecked on page load [AC-001-05-03, US-004-05]. It is held for the duration of the form and sent with the sign-in request [CR-100]. The request field name and shape are not stated in any input (GAP-WEB-010).
- Submission-in-flight indicator.
- Inline error message for the most recent failure.
- Lockout message, when the server returns one [US-003-05].

**Prohibited client storage.**
- The password must never be persisted to any client store [NFR-001].
- The Remember me value must not be persisted beyond the form. Session persistence is the server's responsibility through the cookie lifetime [CR-100, US-004-05].
- The client must not hold lockout state. Lockout state must not rely on cookies or local storage [NFR-012].
- The client must not set, alter or compute a cookie `Max-Age` or expiry. The lifetime is server-issued [US-004-01, NFR-004].
- The client must not read or reconstruct the session token. The cookie is HttpOnly and not readable by script [BR-009, US-004-02].

**Server state (owned by the backend, consumed here).**
- Session validity is established only by server responses [US-004-03, BR-009]. The web app derives authenticated status from server responses, not from cookie inspection.
- The endpoint or response shape used to establish authenticated status is not stated in any input (GAP-WEB-001). This is a dependency on the contract.

**Cache and invalidation policy.** Not specified (GAP-WEB-006). The stories do not state what is cached after login, when cached identity is invalidated, or how an expired session is detected during navigation.

<!-- /SECTION: web-app.state-management -->
<!-- SECTION: web-app.auth-flows | status: BLOCKED -->
## Auth flows

**Reason.** This section is blocked by three independent causes.
- **CON-WEB-001 (Critical).** The lockout counting rule is contradictory across inputs. Every flow that displays lockout state, or whose outcome depends on it, depends on that rule.
- **CON-WEB-003 (Critical).** The session lifetime for Remember me conflicts with BR-009 and NFR-004. Every login-success flow therefore has an unresolved expiry.
- **GAP-WEB-001 and GAP-WEB-010.** No approved contract is pinned, so the endpoint and response shape are unknown. The Remember me request field is also unknown.

**Unblock.**
1. Answer Q-WEB-001: consecutive failures with reset on success (BR-005), or a 15-minute window (AC-001-03-01).
2. Resolve CON-WEB-003: amend BR-009 and NFR-004 to state the Remember me exception, or withdraw CR-100.
3. Supply an approved backend contract, pinned for this run through `contract_source` and `contract_path`.
4. Supply the Remember me request field name and shape (GAP-WEB-010).

Once these are supplied, the author will write the login, refresh, session expiry, logout and mid-flow expiry flows. Refresh, logout and mid-flow expiry are not in the stories (GAP-WEB-006) and require a supplied story before they can be specified.

<!-- /SECTION: web-app.auth-flows -->
<!-- SECTION: web-app.authorization-ui | status: DRAFT-UNVERIFIED -->
## Authorization UI

**Roles that can sign in to this surface.**

| Role | Routes reachable | Visible navigation | Enabled actions |
|---|---|---|---|
| Anonymous visitor (pre-authentication) | Login route only | Not specified (GAP-WEB-006) | "Sign in" button enabled on render [AC-001-05-02]. "Remember me" checkbox unchecked and operable [AC-001-05-03, US-004-05]. |
| Registered account holder | `/home` and protected routes | Not specified (GAP-WEB-006) | Not specified beyond the sign-in action |

**Excluded roles.** Operator and administrator roles are excluded. They have no route on this surface and no story in this run. Listing them would speculate about another surface (invariant I-1). The backend role matrix is not supplied, so this exclusion rests on the absence of any web-app story.

**Permission-gated actions.** None are supplied. The only gated access is to protected routes, governed by session validity [US-004-03].

<!-- /SECTION: web-app.authorization-ui -->
<!-- SECTION: web-app.design-system | status: BLOCKED -->
## Design system

**Reason.** No visual design input is supplied. `design-brief.md` contains acceptance criteria only, and Figma MCP is not connected (GAP-WEB-002). No tokens, type scale, spacing or breakpoints can be cited.

**Unblock.** Supply a design brief containing visual design, or connect Figma MCP. Then supply design tokens and a type scale, and verify this section against them.

<!-- /SECTION: web-app.design-system -->
<!-- SECTION: web-app.responsive | status: BLOCKED -->
## Responsive

**Reason.** No breakpoint or layout input is supplied. Breakpoints and layout changes must be cited from the design, and no design input exists (GAP-WEB-002, GAP-WEB-007).

**Unblock.** Supply the design input described in GAP-WEB-002, including breakpoint definitions and layout behaviour at each breakpoint.

<!-- /SECTION: web-app.responsive -->
<!-- SECTION: web-app.browser-support | status: BLOCKED -->
## Browser support

**Reason.** No browser or version requirement is supplied in any input (GAP-WEB-003). No browser or version may be assumed.

**Unblock.** Supply the supported browser list with minimum versions.

<!-- /SECTION: web-app.browser-support -->
<!-- SECTION: web-app.accessibility | status: DRAFT-UNVERIFIED -->
## Accessibility

**Target standard.** WCAG 2.1 AA for the login page [NFR-010].

**Requirements cited.**
- Login form fields, the button and error messages must be keyboard-navigable and screen-reader accessible [NFR-010].
- The Remember me checkbox must be keyboard-operable and carry an accessible label [NFR-010, US-004-05].
- The form presents the email field, password field and Sign in button with clear focus states and inline error display [NFR-009].
- An error message must render within 1 second of receiving a failed response [NFR-009].
- Empty-field handling is under CON-WEB-002 and is not stated here.

**Verification.** Not specified (GAP-WEB-008). The stories name no verification method, tool or test. A proposed approach, such as an automated accessibility scan plus a manual screen-reader pass, is an assumption and is not adopted until supplied.

<!-- /SECTION: web-app.accessibility -->
<!-- SECTION: web-app.performance | status: DRAFT-UNVERIFIED -->
## Performance

**Cited requirements.**
- `POST /api/login` must respond within p95 ≤ 500 ms for both success and failure paths under normal load [NFR-006]. This is a backend-owned target, observed from the client.
- Login service uptime ≥ 99.9% monthly [NFR-008]. Backend-owned.
- The login endpoint must support ≥ 100 concurrent requests with correct per-account counter increments [NFR-007]. Backend-owned.

**Bundle budget.** Not supplied (GAP-WEB-004). No bundle size may be asserted.

**Route-level code splitting.** Not specified (GAP-WEB-004).

**Largest-payload route.** Cannot be determined. The login page is the only page with stated content, but no payload size is supplied (GAP-WEB-004).

<!-- /SECTION: web-app.performance -->
<!-- SECTION: web-app.coverage | status: VERIFIED -->
## Coverage

Each supplied story is mapped to the section that addresses it.

**Status key.**
- **Covered:** addressed in this document.
- **Partial:** addressed in part. The remainder is in a blocked section or a deferred module spec.
- **Backend-owned:** enforced server-side. The web app's obligation is limited to rendering the response (GAP-WEB-009).

| Story | Section(s) | Status | Note |
|---|---|---|---|
| US-001-01 | `routing`; `auth-flows` | Partial | Redirect to `/home` covered. `auth-flows` blocked (CON-WEB-001, CON-WEB-003, GAP-WEB-001). |
| US-001-02 | `auth-flows` | Partial | Generic message display depends on the blocked section. |
| US-001-03 | `auth-flows` | Partial | Lockout display depends on CON-WEB-001. |
| US-001-04 | `state-management` | Covered (client side) | Cookie attributes are server-set [BR-009]. The client cannot read the cookie. |
| US-001-05 | `routing`; `state-management`; `accessibility` | Partial | Form content, including the Remember me checkbox [CR-100], specified. Login route path is GAP-WEB-005. |
| US-001-06 | `overview` | Covered | The web app performs no credential validation [BR-001]. |
| US-002-01 | `auth-flows` | Partial | Blocked section. |
| US-002-02 | `auth-flows` | Partial | Blocked section. Empty-field behaviour under CON-WEB-002. |
| US-002-03 | None | Backend-owned | Absence of `Set-Cookie` on failure is a backend response property (GAP-WEB-009). |
| US-002-04 | None | Backend-owned | QA verification of backend responses (GAP-WEB-009). |
| US-003-01 | None | Backend-owned | Counting is server-side [BR-005, NFR-012]. Rule under CON-WEB-001. |
| US-003-02 | `auth-flows` | Partial | Blocked on CON-WEB-001. |
| US-003-03 | `auth-flows` | Partial | Blocked on CON-WEB-001. |
| US-003-04 | None | Backend-owned | Per-account counting [BR-005] (GAP-WEB-009). |
| US-003-05 | `auth-flows` | Partial | Message display blocked. |
| US-004-01 | `routing`; `auth-flows` | Partial | `/home` redirect covered. Expiry blocked (CON-WEB-003). |
| US-004-02 | `state-management` | Covered | The client cannot read the cookie [BR-009]. |
| US-004-03 | `routing`; `auth-flows` | Partial | Protected-route rule covered. Expiry handling blocked (CON-WEB-003). |
| US-004-04 | `auth-flows` | Partial | No cookie on failure is backend-owned [BR-004]. UI display blocked. |
| US-004-05 | `routing`; `state-management`; `auth-flows` | Partial | Checkbox UI, default unchecked, and submission covered [CR-100]. 30-day server expiry blocked (CON-WEB-003). Request field GAP-WEB-010. |

**Uncovered-story count.** Zero. All 20 supplied stories (US-001-01 through US-004-05) appear above. Partial rows are blocked by CON-WEB-001, CON-WEB-002, CON-WEB-003, GAP-WEB-001, GAP-WEB-005 or GAP-WEB-010. Four stories are backend-owned: US-002-03, US-002-04, US-003-01 and US-003-04.

**CR-100 coverage.** The Remember me checkbox, its unchecked default and its 30-day `Max-Age` are specified in `routing`, `state-management` and `auth-flows`. The 30-day lifetime is not enforceable until CON-WEB-003 is resolved.

**Referenced contradictions and gaps.**
- CON-WEB-001 (Critical): lockout counting rule. Affects US-001-03, US-003-01, US-003-02, US-003-03 and US-003-05. Blocking question Q-WEB-001.
- CON-WEB-002 (non-critical): empty-field submission. Affects US-001-01, US-001-02 and US-002-02. Marked DRAFT-UNVERIFIED.
- CON-WEB-003 (Critical): Remember me session lifetime. Affects US-001-04, US-004-01, US-004-03 and US-004-05. Conflicts with BR-009 and NFR-004.
- GAP-WEB-001 through GAP-WEB-011: defined in the sections that cite them.

<!-- /SECTION: web-app.coverage -->
