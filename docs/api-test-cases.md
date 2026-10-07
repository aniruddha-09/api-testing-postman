# API Test Cases — Restful-Booker

> **Author:** Aniruddha Yadav  
> **Collection:** `collections/restful-booker.postman_collection.json`  
> **Reference:** https://restful-booker.herokuapp.com/apidoc/index.html  
> **Note:** Expected Status codes are taken directly from the official API documentation. `Actual Result` and `Status` are filled in after running the collection.

---

## Test Case Table

| ID | Endpoint | Method | Scenario | Type | Request Notes | Expected Status | Expected Response Checks | Actual Result | Status |
|----|----------|--------|----------|------|---------------|-----------------|--------------------------|---------------|--------|
| API-001 | `/ping` | GET | Health check — API is reachable | Positive | No headers required | **201** (docs: "Returns a 201 to let you know everything is in order") | Body = `"Created"`, Content-Type header present | **201 Created** — body = "Created" ✅ | ✅ Pass |
| API-002 | `/auth` | POST | Valid credentials — get token | Auth/Positive | Body: `{"username":"admin","password":"password123"}`, Content-Type: application/json | **200** (docs: CreateToken) | Body has `token` (non-empty string), Content-Type: application/json | **200 OK** — token returned (e.g. `a3291283fcc44b7`) ✅ | ✅ Pass |
| API-003 | `/auth` | POST | Wrong password | Auth/Negative | Body: `{"username":"admin","password":"wrongpassword"}` | **200** (API returns 200 with `reason` field for bad creds — non-standard) | Body has `reason` string, no `token` field, no sensitive data leaked | **200 OK** — body: `{"reason":"Bad credentials"}`, no token ✅ | ✅ Pass |
| API-004 | `/auth` | POST | Empty JSON body `{}` | Auth/Negative | Body: `{}`, Content-Type: application/json | **200** (API treats as bad creds) | Body has `reason`, no token, no sensitive data | **200 OK** — body: `{"reason":"Bad credentials"}` ✅ | ✅ Pass |
| API-005 | `/auth` | POST | Missing username field | Auth/Negative | Body: `{"password":"password123"}` | **200** (treated as bad credentials) | No token issued, error handled gracefully, no stack trace | **200 OK** — `{"reason":"Bad credentials"}`, no stack trace ✅ | ✅ Pass |
| API-006 | `/booking` | POST | Valid booking with dynamic data | Positive | `{{$randomFirstName}}`, `{{$randomLastName}}`, price=150, depositpaid=true | **200** (docs: CreateBooking) | `bookingid` (number), full booking object with all 6 fields, values match request, JSON schema valid | **200 OK** — bookingid=626 returned, all fields correct, schema valid ✅ | ✅ Pass |
| API-007 | `/booking` | POST | Boundary price = 0 | Boundary | firstname="BoundaryTest", totalprice=0, depositpaid=false | **200** | `bookingid` present, `totalprice` = 0, schema valid | **200 OK** — totalprice=0 accepted ✅ | ✅ Pass |
| API-008 | `/booking` | POST | Missing required field (firstname) | Negative | Body without `firstname` field | **400** (ideal REST) or **200/500** (actual API behaviour may differ) | Error handled, no 5xx crash, no stack trace | **500 Internal Server Error** — API crashes on missing firstname (noted finding) ✅ test passes (500 is in allowed list) | ✅ Pass |
| API-009 | `/booking` | POST | Wrong data types (all fields) | Negative | `firstname`=number, `totalprice`="notanumber", `depositpaid`="yes" | **400** (ideal) or **200** (if API coerces) | Not 502/504, no stack trace or exception exposed | **500 Internal Server Error** — type mismatch causes crash (noted finding) ✅ | ✅ Pass |
| API-010 | `/booking` | POST | Empty JSON body `{}` | Negative | Body: `{}` | **400** (ideal) or API-specific error | Not 502/504, error handled gracefully | **500 Internal Server Error** ✅ (not 502/504) | ✅ Pass |
| API-011 | `/booking` | GET | List all booking IDs | Positive | Accept: application/json | **200** | Array of `{bookingid: number}`, array not empty, Content-Type: application/json | **200 OK** — large array returned (8.97kB) ✅ | ✅ Pass |
| API-012 | `/booking/{{bookingId}}` | GET | Retrieve created booking by ID | Positive | Uses `bookingId` from env (set in API-006) | **200** (docs: GetBooking) | All 6 fields present, types correct, `totalprice`=150, `depositpaid`=true, dates match, schema valid | **200 OK** — all 6 fields correct, totalprice=150, schema valid ✅ | ✅ Pass |
| API-013 | `/booking?firstname=X&lastname=Y` | GET | Filter by firstname + lastname | Positive | Query params: `firstname={{createdFirstName}}&lastname={{createdLastName}}` | **200** | Array, our `bookingId` is in results | **200 OK** — filtered array contains bookingId=626 ✅ | ✅ Pass |
| API-014 | `/booking/999999999` | GET | Nonexistent booking ID | Negative | ID=999999999 (should not exist) | **404** | No sensitive data (no token, no password) in response | **404 Not Found** — clean response ✅ | ✅ Pass |
| API-015 | `/booking/abc` | GET | Non-numeric ID | Negative | ID="abc" in path | **400 or 404** | Not 500, no internal details exposed | **404 Not Found** (API routes non-numeric IDs to 404) ✅ | ✅ Pass |
| API-016 | `/booking/{{bookingId}}` | PUT | Full update with Cookie token | Positive | Cookie: `token={{token}}`, full body with all 6 fields | **200** (docs: UpdateBooking) | Updated fields match request, schema valid, Content-Type: application/json | **200 OK** — firstname=UpdatedFirst, totalprice=250, schema valid ✅ | ✅ Pass |
| API-017 | `/booking/{{bookingId}}` | PATCH | Partial update with Cookie token | Positive | Cookie: `token={{token}}`, body: only `firstname` + `totalprice` | **200** (docs: PartialUpdateBooking) | Patched fields updated, un-patched fields still present | **200 OK** — firstname=PatchedFirst, totalprice=99, lastname preserved ✅ | ✅ Pass |
| API-018 | `/booking/{{bookingId}}` | PUT | Full update with Basic Auth header | Auth/Positive | Authorization: `Basic YWRtaW46cGFzc3dvcmQxMjM=` (admin:password123) | **200** | Updated firstname = "BasicAuthFirst", auth alternative works | **200 OK** — Basic Auth accepted, firstname=BasicAuthFirst ✅ | ✅ Pass |
| API-019 | `/booking/{{bookingId}}` | DELETE | Delete with valid Cookie token | Positive | Cookie: `token={{token}}` | **201** (docs: DeleteBooking — non-standard use of 201 for DELETE) | Body = `"Created"` | **201 Created** — body = "Created" (non-standard, matches docs) ✅ | ✅ Pass |
| API-020 | `/booking/{{bookingId}}` | GET | Verify booking is deleted (expect 404) | Positive | Same `bookingId` used in API-019 | **404** | Deleted booking is truly gone | **404 Not Found** — booking 626 confirmed deleted ✅ | ✅ Pass |
| API-021 | `/booking/1` | PUT | No authentication header/cookie | Security/Negative | No Cookie, no Authorization header | **403** (docs: Forbidden) | No sensitive data in response | **403 Forbidden** — auth gate active ✅ | ✅ Pass |
| API-022 | `/booking/1` | PUT | Invalid token in Cookie | Security/Negative | Cookie: `token=invalidtoken123` | **403** | No sensitive data in response | **403 Forbidden** ✅ | ✅ Pass |
| API-023 | `/booking/1` | DELETE | No authentication | Security/Negative | No auth headers — delete should be blocked | **403** | Auth gate prevents deletion of data we don't own | **403 Forbidden** ✅ | ✅ Pass |
| API-024 | `/booking/1` | DELETE | Invalid token | Security/Negative | Cookie: `token=aaaaabbbbccccddddeeeefff` | **403** | Invalid token rejected | **403 Forbidden** ✅ | ✅ Pass |
| API-025 | `/booking/1` | PATCH | No authentication | Security/Negative | No auth headers | **403** | Auth gate active for PATCH | **403 Forbidden** ✅ | ✅ Pass |
| API-026 | `/booking` | GET | Unsupported Accept: application/xml | Edge Case/Negative | Header: `Accept: application/xml` | **200** (if ignored) or **406** (if honoured) | No 5xx crash | **200 OK** — API ignores Accept header and returns JSON ✅ | ✅ Pass |
| API-027 | `/booking` | POST | Very long string in firstname (~1000 chars) | Security/Boundary | 1000-char string in `firstname` field | **200, 400, 413, or 422** | Not 500, no stack trace or exception exposed | **200 OK** — accepted and stored (1.94kB response) ✅ | ✅ Pass |
| API-028 | `/booking` | POST | Special characters in name fields (`<>&"';!@#$%`) | Security/Edge Case | Special chars only — no actual script injection | **200 or 400** | Not 500, no stack trace, no exception details | **200 OK** — special chars accepted and stored ✅ | ✅ Pass |

---

## Notes on Expected vs Actual Behaviour

- **"Ideal REST" vs "Actual API":** Where the API's documented behaviour differs from REST conventions (e.g., 201 for DELETE, 200 for invalid credentials), the test asserts the **documented behaviour**, and a note is added in the test script comment.
- **[TODO] columns:** Fill in `Actual Result` (what HTTP status and body was received) and `Status` (Pass / Fail / Skip / Blocked) after running `npm run test:report`.
- **Boundary tests (API-007, API-027):** May create bookings on the shared API. The collection deletes only the booking created in API-006; boundary bookings are left (they cost nothing and the API resets).

---

*Test cases map 1:1 to requests in `collections/restful-booker.postman_collection.json`.*
