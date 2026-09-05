# gorest-api-tests
Automated API test suite for GoRest (gorest.co.in) using Postman &amp; Newman — covering real Bearer token auth, field validation, pagination, and full CRUD lifecycle testing, with CI/CD via GitHub Actions..

## Key Findings

1. **GET requests are public; POST/PUT/DELETE require Bearer authentication.** No auth needed to read data, but write operations return 401 without a valid token.

2. **Distinct error messages for different auth failures.** Missing token → `"Authentication failed"`. Invalid/malformed token → `"Invalid token"` — including a subtle case where `Bearer/token` (slash instead of space) is treated as no token at all.

3. **Real, structured 404s for non-existent resources** — a genuine contrast to FakeStoreAPI, which never used 404 at all. GoRest returns `{"message": "Resource not found"}`.

4. **Genuine field-level validation.** Invalid gender values and duplicate emails are rejected with 422/409 — unlike FakeStoreAPI, which accepted anything.

5. **Live dataset drift.** User counts and pagination totals change over time (2967→2956 users in days) — and can even shift within a single test run, since GoRest is a shared, live public database.

6. **Hardcoded IDs/page numbers go stale quickly.** Dynamic chaining (capturing real, current IDs via pre-request scripts) is required for reliable tests — a static ID or page number cannot be trusted to remain valid.

7. **Full CRUD lifecycle is genuinely persistent and verifiable** — create, read, update, and delete were all independently confirmed via a self-contained chained sequence, unlike FakeStoreAPI's simulated (non-persistent) writes.

8. **CSV files must be genuinely plain-text** — files saved with wrapping quotes or non-standard line endings (as produced by some export/copy-paste workflows) will silently fail to parse into separate columns.

9. **Newman requires explicit handling for data-driven requests.** A request depending on CSV data will crash if run without the `-d` flag; production-quality scripts should check for missing iteration data and degrade gracefully rather than crash.

10. **GoRest's CI/CD behavior differs from FakeStoreAPI's.** Unlike FakeStoreAPI (blocked by Cloudflare on GitHub Actions), GoRest's API runs cleanly from GitHub's shared IP ranges — the full 28-request suite executes successfully in CI, with only the expected live-data-drift failures appearing.

## Retrospective

Testing GoRest versus FakeStoreAPI highlighted how differently two "free public APIs" can behave — one with no real validation or persistence, the other with genuine auth, field validation, and a live, shared dataset. The biggest lesson was that testing against live data requires designing for change: dynamic chaining, not hardcoded values, is essential for a suite that stays reliable over time. If I had another week, I'd add rate-limit testing and explore GoRest's other resources (Posts, Comments in more depth) with the same rigor applied to Users.    
