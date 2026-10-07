# API Test Plan — Restful-Booker

> **Project:** Restful-Booker API Test Suite  
> **Author:** Aniruddha Yadav  
> **Date:** October 2026  
> **Version:** 1.0  
> **API Under Test:** https://restful-booker.herokuapp.com  
> **API Documentation:** https://restful-booker.herokuapp.com/apidoc/index.html

---

## 1. Objective

To design and execute a comprehensive functional and non-functional test suite against the **Restful-Booker REST API** — a publicly available practice API for QA engineers. The goal is to verify that the API behaves according to its documented specification, handle edge cases gracefully, and produce evidence of test coverage suitable for a QA portfolio.

---

## 2. Scope

### 2.1 In-Scope Endpoints

| # | Endpoint | Methods |
|---|----------|---------|
| 1 | `/ping` | GET |
| 2 | `/auth` | POST |
| 3 | `/booking` | GET, POST |
| 4 | `/booking/:id` | GET, PUT, PATCH, DELETE |

### 2.2 Test Layers

- **Functional (Positive):** Verify documented happy-path behaviour for each endpoint.
- **Negative:** Verify the API handles invalid inputs, missing fields, and wrong types without crashing.
- **Authentication:** Validate token creation, token-protected routes, Basic Auth alternative, invalid token rejection, and unauthenticated access.
- **Boundary:** Test limit values (price = 0, price = 9999, minimum-stay dates, empty strings).
- **Schema:** Validate response JSON structure against a defined schema using `pm.response.to.have.jsonSchema`.
- **Data-Driven:** Run the Create Booking request across 8 CSV rows with varied inputs.
- **Chained Workflow:** The full collection runs end-to-end: auth → create → read → update → delete, using environment variables to chain `token` and `bookingId`.
- **Security Edge Cases:** Very long strings, special characters in input fields, unauthenticated mutations (verify auth gate works).

---

## 3. Out of Scope

The following test types are **explicitly excluded** to respect the shared public API:

| Excluded | Reason |
|----------|--------|
| Load / Stress / Spike testing | Would harm a free-tier shared API used by others |
| Automated scheduled runs | Would generate unnecessary traffic; CI triggers on push to main or manual dispatch only |
| Security penetration / injection exploits | Out of scope for this practice exercise; only benign edge-case strings are sent |
| Data scraping / bulk reads | Not appropriate for a shared API |
| Editing or deleting bookings we did not create | Data hygiene — every destructive test creates and cleans up its own booking |
| Performance benchmarking / SLA assertions | Free-tier Heroku host has no SLA; response time threshold is set generously at 3000 ms |

---

## 4. Test Approach

### 4.1 Methodologies

1. **Functional Testing** — Verify each endpoint returns the documented status code, response structure, and data.
2. **Negative Testing** — Send invalid credentials, wrong types, missing fields, and empty bodies. Verify errors are handled gracefully.
3. **Authentication Testing** — Test token-based auth (cookie), Basic Auth header, missing auth, invalid token.
4. **Boundary Value Analysis** — Test at limits: price = 0, price = 9999, single-night stays, very long strings.
5. **JSON Schema Validation** — Every response from `/booking` is validated against a defined JSON schema.
6. **Data-Driven Testing** — Newman `--iteration-data` runs the Create Booking request for each of the 8 rows in `data/bookings.csv`.
7. **Chained Workflow** — Collection runs top-to-bottom; `token` is captured in the Auth folder and reused in all protected endpoints; `bookingId` is captured in Create and reused in Read, Update, Delete.

### 4.2 Tools

| Tool | Purpose |
|------|---------|
| **Postman** (Desktop) | Collection authoring, manual test execution |
| **Newman** (CLI) | Automated headless test execution |
| **newman-reporter-htmlextra** | Rich HTML test report generation |
| **npm scripts** | `test`, `test:report`, `test:data` for easy execution |
| **GitHub Actions** | CI pipeline (push to main / manual dispatch) |
| **Git** | Version control with one commit per step |

### 4.3 Environment

| Variable | Value |
|----------|-------|
| `baseUrl` | `https://restful-booker.herokuapp.com` |
| `username` | `admin` (public practice credentials from docs) |
| `password` | `password123` |
| `token` | Set dynamically by the Auth test script |
| `bookingId` | Set dynamically by the Create test script |

> **[TODO: fill in Postman version used during test execution]**  
> **[TODO: fill in Node.js version (run `node --version`)]**  
> **[TODO: fill in operating system and version]**

---

## 5. Entry Criteria

- [ ] API is reachable (`GET /ping` returns 201)
- [ ] Environment file is imported into Postman (or passed to Newman via `-e`)
- [ ] Newman and newman-reporter-htmlextra are installed (`npm install`)
- [ ] Valid credentials are confirmed (`admin` / `password123`)

## 6. Exit Criteria

- [ ] All positive/happy-path tests pass
- [ ] All negative tests assert the expected error codes (even if the API behaviour differs from REST ideal)
- [ ] HTML report is generated in `reports/`
- [ ] Findings documented in `docs/findings.md`
- [ ] Placeholder `[TODO]` items are filled in after execution

---

## 7. Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Shared public API** — another user may delete our booking during the run | Medium | Accept as known risk; re-run if needed |
| **Free-tier cold start latency** | High — tests may time out | Response time threshold set to 3000 ms; if still failing, retry |
| **No SLA or uptime guarantee** | Medium | If API is down, re-run after wait period |
| **API may change without notice** | High | Pin all expected values to documented behaviour; add comments linking to docs |
| **Data pollution** | Medium | Every test that writes data creates its own booking and deletes it in the Delete folder |
| **Rate limiting** | Low | Only a single sequential run; no loops or parallel requests |

---

## 8. Deliverables

| Deliverable | Location |
|-------------|----------|
| Postman Collection | `collections/restful-booker.postman_collection.json` |
| Postman Environment | `environments/restful-booker.postman_environment.json` |
| Test Data (CSV) | `data/bookings.csv` |
| Test Plan | `docs/api-test-plan.md` (this file) |
| Test Cases Table | `docs/api-test-cases.md` |
| Findings & Observations | `docs/findings.md` |
| Newman HTML Report | `reports/newman-report.html` (after running `npm run test:report`) |
| CI Workflow | `.github/workflows/newman.yml` |

---

*This test plan is part of a QA portfolio project. The Restful-Booker API is a public practice API maintained by Mark Winteringham.*
