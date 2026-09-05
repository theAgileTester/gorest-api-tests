# GoRest API Test Suite

Automated API test suite for GoRest (gorest.co.in) using Postman & Newman — covering real Bearer token auth, field validation, pagination, and full CRUD lifecycle testing, with CI/CD via GitHub Actions.

# Test Strategy

28 test cases across Users, Comments, and Todos, covering positive, negative, boundary, and authentication scenarios, plus a CSV-driven validation sweep and a complete, self-contained CRUD lifecycle chain.

# Category	Coverage
Positive	Valid IDs, valid pagination boundaries, valid creation, successful auth
Negative	Missing/invalid tokens, invalid field values, duplicate emails, non-existent IDs
Boundary	First/last pagination pages (dynamically fetched) across Users, Comments, Todos
Auth	Public GET vs. authenticated write, missing token, malformed token, invalid token
Data-driven	CSV sweep testing multiple email/gender combinations against real field validation
Chaining	Full create → read → update → delete → confirm-deletion lifecycle on Users
# Key Findings
- GET requests are public; POST/PUT/DELETE require Bearer authentication. No auth needed to read data, but write operations return 401 without a valid token.
- Distinct error messages for different auth failures. Missing token → "Authentication failed". Invalid/malformed token → "Invalid token" — including a subtle case where Bearer/token (slash instead of space) is treated as no token at all.
- Real, structured 404s for non-existent resources — a genuine contrast to FakeStoreAPI, which never used 404 at all. GoRest returns {"message": "Resource not found"}.
- Genuine field-level validation. Invalid gender values and duplicate emails are rejected with 422/409 — unlike FakeStoreAPI, which accepted anything.
- Live dataset drift. User counts and pagination totals change over time (2967→2956 users in days) — and can even shift within a single test run, since GoRest is a shared, live public database.
- Hardcoded IDs/page numbers go stale quickly. Dynamic chaining (capturing real, current IDs via pre-request scripts) is required for reliable tests — a static ID or page number cannot be trusted to remain valid.
- Full CRUD lifecycle is genuinely persistent and verifiable — create, read, update, and delete were all independently confirmed via a self-contained chained sequence, unlike FakeStoreAPI's simulated (non-persistent) writes.
- CSV files must be genuinely plain-text — files saved with wrapping quotes or non-standard line endings (as produced by some export/copy-paste workflows) will silently fail to parse into separate columns.
- Newman requires explicit handling for data-driven requests. A request depending on CSV data will crash if run without the -d flag; production-quality scripts should check for missing iteration data and degrade gracefully rather than crash.
- GoRest's CI/CD behavior differs from FakeStoreAPI's. Unlike FakeStoreAPI (blocked by Cloudflare on GitHub Actions), GoRest's API runs cleanly from GitHub's shared IP ranges — the full 28-request suite executes successfully in CI, with only the expected live-data-drift failures appearing.
# How to Run Postman

Import GoRest API Tests.postman_collection.json
Import GoRest.postman_environment.example.json
Replace the accessToken value with your own GoRest access token
Run via Collection Runner — set Delay to 500ms
For the CSV-driven validation test (TC_23), attach gorest_validation_test.csv in the Runner's Data field
Newman (command line)
bash
npm install -g newman newman-reporter-html

# Full suite (single pass, recommended for regression reporting)
newman run "GoRest API Tests.postman_collection.json" -e "GoRest.postman_environment.example.json" --env-var "accessToken=YOUR_TOKEN" --delay-request 500 -r html

# CSV-driven validation test only (5 iterations)
newman run "GoRest API Tests.postman_collection.json" -e "GoRest.postman_environment.example.json" --env-var "accessToken=YOUR_TOKEN" -d "gorest_validation_test.csv" --folder "TC_23 - POST create user - data-driven validation" --delay-request 500
CI/CD

Runs automatically on every push via GitHub Actions (.github/workflows/api-tests.yml). The access token is securely injected via GitHub Secrets (GOREST_ACCESS_TOKEN) — never stored in the repository. A downloadable HTML report is generated as a build artifact on every run.

# Retrospective

Testing GoRest versus FakeStoreAPI highlighted how differently two "free public APIs" can behave — one with no real validation or persistence, the other with genuine auth, field validation, and a live, shared dataset. The biggest lesson was that testing against live data requires designing for change: dynamic chaining, not hardcoded values, is essential for a suite that stays reliable over time. If I had another week, I'd add rate-limit testing and explore GoRest's other resources (Posts, Comments in more depth) with the same rigor applied to Users.

# Tech Stack

Postman · Newman · GitHub Actions · JavaScript (Chai assertions) · GitHub Secrets

# Author

Built as Project 2 of a self-directed API testing practice series, applying skills from a 35-day Postman/Newman/CI-CD learning program.
