# API Findings — Restful-Booker

> **Author:** Aniruddha Yadav  
> **Date:** October 2026  
> **API:** https://restful-booker.herokuapp.com  
> **Status:** Placeholder — fill in after the first real run

---

## How to Use This Document

This guide explains how to separate and document three distinct types of findings:

### (a) Documented API Behaviour
**What it is:** Anything the API does that is explicitly described in the official docs (https://restful-booker.herokuapp.com/apidoc/index.html) — even if it differs from REST conventions.

**How to write it up:**
> "According to the documentation, `DELETE /booking/:id` returns **201 Created** (not 204 No Content). This is the documented, expected behaviour of this API and is not a defect."

Use this category for things like:
- The auth endpoint returning 200 with `{"reason": "Bad credentials"}` instead of 401.
- DELETE returning 201 instead of 204.
- The API using a token-in-cookie pattern rather than a Bearer token.

---

### (b) What I Observed
**What it is:** What actually happened when you ran the test — whether it matched the docs or not.

**How to write it up:**
> "When sending `POST /auth` with an empty body `{}`, the API returned **200** with `{"reason": "Bad credentials"}`. This matches the documented pattern for invalid credentials."

Or:
> "When sending `GET /booking/abc` (non-numeric ID), the API returned **404** rather than **400**. The docs do not specify the error code for this case."

Use this category for:
- Exact HTTP status codes received.
- Exact response bodies received.
- Response times observed.
- Any behaviour not covered by the docs.

---

### (c) What I Think Is a Defect
**What it is:** A difference between the documented behaviour and what was observed, OR a behaviour (even if undocumented) that is clearly a problem (e.g., a 500, a stack trace leak, a successful unauthenticated mutation).

**How to write it up:**
> **DEF-001 | Severity: Low**  
> **Title:** Missing required field `firstname` in POST /booking does not return 400  
> **Steps to reproduce:** Send `POST /booking` with all fields except `firstname`.  
> **Expected (docs/REST):** 400 Bad Request with an error message.  
> **Actual:** 200 OK with a booking created (firstname field is empty string or null).  
> **Impact:** Invalid data can be persisted.  
> **Note:** This may be intentional for a practice API.

---

## Section 1 — Behaviours That Differ from REST Best Practice

> **[TODO: fill in after running the collection]**

Examples to investigate:
- [ ] `POST /auth` returns 200 for bad credentials instead of 401.
- [ ] `DELETE /booking/:id` returns 201 instead of 204 No Content.
- [ ] Does the API return appropriate 415 when Content-Type is missing?
- [ ] Does the API return 406 when `Accept: application/xml` is sent?
- [ ] Are error response bodies consistent across endpoints?

*Document each one in the format:*

| # | Endpoint | Method | Observed Behaviour | REST Best Practice | Notes |
|---|----------|--------|--------------------|--------------------|-------|
| B-001 | [TODO] | [TODO] | [TODO] | [TODO] | [TODO] |

---

## Section 2 — Defects

> **[TODO: fill in after running the collection]**

*Document each defect in this format:*

---

**DEF-001**
- **Severity:** [Critical / High / Medium / Low]
- **Title:** [Short description]
- **Endpoint:** `METHOD /path`
- **Steps to Reproduce:**
  1. [Step 1]
  2. [Step 2]
- **Expected Result:** [What docs/REST says should happen]
- **Actual Result:** [What was actually observed — copy from test run output]
- **Impact:** [What could go wrong in a real system]
- **Evidence:** [Reference the HTML report or test case ID, e.g., API-008]

---

## Section 3 — Observations

> **[TODO: fill in after running the collection]**

General observations that are neither defects nor documented behaviour — things worth noting for understanding the API or for future improvement.

Examples to consider:
- [ ] Cold-start latency (first request of the day may be slow — document measured range).
- [ ] Shared data: did any test fail because another user deleted a booking during the run?
- [ ] Does the API set CORS headers? Are they appropriate?
- [ ] Are rate-limit headers (`X-RateLimit-*`) present?
- [ ] Consistency of `Content-Type` response headers across endpoints.
- [ ] Whether `additionalneeds` field is truly optional or if its absence causes any issue.

*Document each observation in plain language, e.g.:*

> **OBS-001:** The `GET /ping` response body is the string `"Created"`, the same text returned by `DELETE /booking/:id`. This is unusual but harmless.

---

*This document is part of the Restful-Booker API testing portfolio by Aniruddha Yadav.*
