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

> **Run date:** 2026-10-08 | Booking ID created: 626 | Total assertions: 112/112 passed

| # | Endpoint | Method | Observed Behaviour | REST Best Practice | Notes |
|---|----------|--------|--------------------|--------------------|-------|
| B-001 | `/auth` | POST | Returns **200 OK** with `{"reason":"Bad credentials"}` for wrong/missing credentials | Should return **401 Unauthorized** | Documented behaviour of this practice API — not a defect per its own docs |
| B-002 | `/booking/:id` | DELETE | Returns **201 Created** (body: `"Created"`) on successful delete | Should return **204 No Content** | Documented behaviour — noted as non-standard in every test comment |
| B-003 | `/booking` | GET | Returns **200 OK** when `Accept: application/xml` is sent; ignores the header and returns JSON | Should return **406 Not Acceptable** | API does not honour the `Accept` header |
| B-004 | `/ping` | GET | Returns **201 Created** with body `"Created"` | A health check should return **200 OK** with no or meaningful body | Documented behaviour; the body `"Created"` is the same as the DELETE success body |

---

## Section 2 — Defects

> **Run date:** 2026-10-08 | All 112 assertions passed in this run.

---

**DEF-001**
- **Severity:** Medium
- **Title:** Missing required field in `POST /booking` causes 500 Internal Server Error instead of 400
- **Endpoint:** `POST /booking`
- **Steps to Reproduce:**
  1. Send `POST /booking` with a valid JSON body but omit the `firstname` field.
- **Expected Result:** 400 Bad Request with a descriptive validation error.
- **Actual Result (run 2026-10-08):** **500 Internal Server Error** — the server crashes rather than validating input.
- **Impact:** In a production system, this would expose internal server errors and could be used to probe the API for vulnerabilities. In this practice API, it is harmless but notable.
- **Evidence:** API-008 | Newman run 2026-10-08.

---

**DEF-002**
- **Severity:** Medium
- **Title:** Wrong data types in `POST /booking` body causes 500 instead of 400/422
- **Endpoint:** `POST /booking`
- **Steps to Reproduce:**
  1. Send `POST /booking` with `firstname` as a number, `totalprice` as a string, `depositpaid` as a string.
- **Expected Result:** 400 or 422 Unprocessable Entity with type validation errors.
- **Actual Result (run 2026-10-08):** **500 Internal Server Error**
- **Impact:** Same as DEF-001 — a production API should validate types and return structured errors.
- **Evidence:** API-009 | Newman run 2026-10-08.

---

**DEF-003**
- **Severity:** Low
- **Title:** Empty body `{}` in `POST /booking` causes 500 instead of 400
- **Endpoint:** `POST /booking`
- **Steps to Reproduce:**
  1. Send `POST /booking` with body `{}`.
- **Expected Result:** 400 Bad Request.
- **Actual Result (run 2026-10-08):** **500 Internal Server Error**
- **Evidence:** API-010 | Newman run 2026-10-08.

---

**DEF-004**
- **Severity:** Low
- **Title:** Very long string (1000+ chars) in `firstname` is accepted and stored without truncation or error
- **Endpoint:** `POST /booking`
- **Steps to Reproduce:**
  1. Send `POST /booking` with `firstname` containing 1000 identical characters.
- **Expected Result:** 400 or 413 with input length validation error.
- **Actual Result (run 2026-10-08):** **200 OK** — booking created, 1000-char firstname stored (1.94kB response body).
- **Impact:** Could lead to storage bloat in a production system.
- **Evidence:** API-027 | Newman run 2026-10-08.

---

## Section 3 — Observations

> **Run date:** 2026-10-08 | Booking created with ID 626 | API deleted, verified gone.

**OBS-001:** `GET /ping` returns body `"Created"` — the same text returned by `DELETE /booking/:id`. These two unrelated endpoints share a response body, which is confusing but harmless in this practice context.

**OBS-002:** Cold start on Heroku free tier was observed — the first request (`GET /ping`) took **727 ms**, while all subsequent requests averaged **207 ms**. This is expected behaviour for a free-tier dyno.

**OBS-003:** `GET /booking/abc` (non-numeric ID) returns **404 Not Found** rather than **400 Bad Request**. The API routes invalid path segments to 404 rather than distinguishing between "not found" and "invalid format".

**OBS-004:** `POST /auth` returns 200 with `{"reason": "Bad credentials"}` for *all* auth failure cases — empty body, missing field, and wrong password all produce the same response. This means the API gives no hint about what specifically failed, which is actually a security best practice (avoiding username enumeration).

**OBS-005:** The API accepts special characters (`<>&"';!@#$%`) in `firstname` and `lastname` fields and stores them verbatim (200 OK). There is no HTML encoding or sanitisation visible in the response. This would be a concern in a production system rendering these values in a browser (XSS risk).

**OBS-006:** Very long strings (~1000 characters) are accepted in `firstname` without any length validation. The response body was 1.94kB vs the typical ~900B for a normal booking response, confirming the full string was stored.

---

*This document is part of the Restful-Booker API testing portfolio by Aniruddha Yadav.*
