# Restful-Booker API Test Suite

> A production-ready API testing portfolio project by **Aniruddha Yadav** (QA Engineer)

[![Newman CI](https://img.shields.io/badge/Newman-CLI-brightgreen?logo=postman)](https://www.npmjs.com/package/newman)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![API: Restful-Booker](https://img.shields.io/badge/API-Restful--Booker-orange)](https://restful-booker.herokuapp.com)

---

## Overview

This repository demonstrates professional-grade API testing skills applied to the [Restful-Booker](https://restful-booker.herokuapp.com) practice API — a publicly available REST API for QA practice, maintained by [Mark Winteringham](https://www.mwtestconsultancy.co.uk/).

The test suite covers authentication, full CRUD operations, boundary conditions, negative scenarios, security edge cases, and chained request workflows — all structured for automated execution via **Newman** from the command line or in a **GitHub Actions** CI pipeline.

---

## What I Tested

### Endpoints

| Endpoint | Methods Covered |
|----------|----------------|
| `/ping` | GET |
| `/auth` | POST (valid, wrong password, empty body, missing field) |
| `/booking` | GET (list, filtered), POST (valid, boundary, negative) |
| `/booking/:id` | GET, PUT, PATCH, DELETE |

### Test Types

| Type | Count | Description |
|------|-------|-------------|
| ✅ Positive | 10 | Happy-path scenarios from the API docs |
| ❌ Negative | 9 | Invalid inputs, missing fields, wrong types |
| 🔐 Auth | 6 | Token creation, cookie auth, Basic Auth, invalid/missing auth |
| 📐 Boundary | 3 | Price = 0, price = 9999, very long strings |
| 🔍 Schema | 28 | JSON schema validation on every booking response |
| 🧪 Data-Driven | 8 rows | CSV-driven booking creation (see below) |
| 🔗 Chained | Yes | `token` → `bookingId` linked across folders end-to-end |

**Total requests in collection: 28**

---

## Technologies

| Tool | Purpose |
|------|---------|
| [Postman](https://www.postman.com/) | Collection authoring and manual execution |
| [Newman](https://www.npmjs.com/package/newman) | CLI/headless test runner |
| [newman-reporter-htmlextra](https://github.com/DannyDainton/newman-reporter-htmlextra) | Rich HTML test reports |
| [GitHub Actions](https://docs.github.com/en/actions) | CI pipeline (push to `main` + manual dispatch) |
| Node.js / npm | Runtime and package manager |
| Git | Version control (one commit per step) |

---

## Repository Structure

```
api-testing-postman/
├── .github/
│   └── workflows/
│       └── newman.yml               # CI pipeline (push to main / manual)
├── collections/
│   └── restful-booker.postman_collection.json  # Postman v2.1 collection (28 requests)
├── data/
│   └── bookings.csv                 # 8-row CSV for data-driven testing
├── docs/
│   ├── api-test-plan.md             # Test plan, scope, risks, approach
│   ├── api-test-cases.md            # Full test case table (API-001 to API-028)
│   └── findings.md                  # Findings template (fill after run)
├── environments/
│   └── restful-booker.postman_environment.json  # Env vars: baseUrl, token, bookingId
├── reports/
│   └── .gitkeep                     # Reports folder tracked in Git
│                                    # (newman-report.html added here after test run)
├── .gitignore
├── LICENSE                          # MIT
├── package.json                     # npm scripts and devDependencies
└── README.md                        # This file
```

---

## Quick Start

### Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- npm (comes with Node.js)
- (Optional) [Postman Desktop](https://www.postman.com/downloads/) for manual exploration

### 1. Clone and Install

```bash
git clone https://github.com/aniruddhayadav12/api-testing-postman.git
cd api-testing-postman
npm install
```

### 2. Run All Tests (CLI output only)

```bash
npm test
```

### 3. Run Tests and Generate HTML Report

```bash
npm run test:report
```

Then open **`reports/newman-report.html`** in your browser for a visual, shareable test report.

### 4. Import into Postman (Manual Run)

1. Open Postman Desktop.
2. **Import** → select `collections/restful-booker.postman_collection.json`.
3. **Import** → select `environments/restful-booker.postman_environment.json`.
4. Select the **"Restful-Booker Environment"** from the environment picker.
5. Run the collection using **Collection Runner** in order (top to bottom).

> ⚠️ **Important:** Run requests top-to-bottom. The Auth folder sets `{{token}}`; the Create folder sets `{{bookingId}}`; both are required by later folders.

---

## Data-Driven Testing

The `data/bookings.csv` file contains **8 rows** of varied booking data:

| Names | Price range | Dates | Additional needs |
|-------|-------------|-------|-----------------|
| Mixed international names | 0, 1, 75, 89, 120, 250, 450, 9999 | 2025 various | Breakfast, Dinner, Spa, None… |

### Run with CSV data:

```bash
npm run test:data
```

This runs the full collection **once per CSV row** (8 iterations), substituting each row's values into the request body using `{{firstname}}`, `{{totalprice}}`, etc. variables. The generated report is saved to `reports/newman-data-report.html`.

> **Note on the data-driven approach:** The current collection uses `{{$randomFirstName}}` in the Create request for its main happy-path test. To fully use the CSV, import the CSV variables in the relevant request body or create a dedicated data-driven folder. See `docs/api-test-plan.md` for the approach explanation.

---

## Results

✅ **Real run completed on 2026-10-08 against https://restful-booker.herokuapp.com**

| Metric | Value |
|--------|-------|
| Iterations | 1 |
| Requests | 28 |
| Assertions | **112** |
| Failures | **0** |
| Total Duration | 8.9 s |
| Avg Response Time | 227 ms |
| Min / Max Response | 206 ms / 727 ms |
| Data Received | 19.55 kB |

> 📄 **[View the HTML report: `reports/newman-report.html`](reports/newman-report.html)**  
> Open the file locally in any browser after cloning the repo.

---

## Key Findings

> **[TODO: fill in after running the collection — see `docs/findings.md`]**

Examples of what to document:
- Interesting API behaviours observed.
- Status codes that differ from REST conventions.
- Any defects identified.

---

## What I Learned

> **[TODO: fill in with personal reflection after completing the project]**

Suggested topics:
- How chained variables (`token` → `bookingId`) enable reliable end-to-end testing.
- How JSON schema validation catches structural regressions early.
- The difference between documented API behaviour and REST best practice.
- How to structure a test collection for both manual Postman use and automated Newman runs.
- Setting up GitHub Actions for API test automation.

---

## A Note on Traffic and Shared API Ethics

This test suite interacts with a **free, shared public API**. I've taken the following steps to be a responsible API consumer:

- ✅ **No load or stress tests** — only a single sequential run.
- ✅ **No automated schedules** — CI triggers only on push to `main` or manual dispatch.
- ✅ **Data cleanup** — every test that creates a booking deletes it at the end of the run.
- ✅ **No scraping** — only the requests needed for test coverage.
- ✅ **No editing third-party data** — negative auth tests use booking `id=1` (a pre-existing booking) only to verify the 403 gate; the deletion test only deletes the booking we created.

---

## Contact

- 💼 **LinkedIn:** [linkedin.com/in/aniruddhayadav12](https://linkedin.com/in/aniruddhayadav12)
- 🌐 **Portfolio:** [aniruddhayadav.in](https://aniruddhayadav.in)

---

*This is a portfolio project created to demonstrate API testing skills. The Restful-Booker API is a public practice API by [Mark Winteringham](https://www.mwtestconsultancy.co.uk/).*
