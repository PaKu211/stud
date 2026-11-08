# Test Plan & Quality Gates

## 1. Test strategy

- **Unit:** metadata normalize, CSV encoding, Adobe policy, status transitions, idempotency keys, hashes, path containment.
- **Integration:** SQLite migrations/restart, import directory fixtures, thumbnail generation, transaction rollback, server HTTP handlers.
- **End-to-end:** 20 image sample local workflow (no external network) via browser.
- **Contract:** AI adapter mocked 200, invalid schema, 401/429/503, timeout; SFTP transport mocked in future phase.
- **Security:** path traversal, symlink, CSRF, direct file path leak, formula injection, secrets redaction.
- **Compatibility:** Linux amd64 + Windows amd64; manual smoke on low-RAM laptop when available.
- **Regression:** golden CSV fixture from official schema and policy version.

## 2. Fixtures to generate for testing

Create image fixtures programmatically from simple color/gradient and shapes (not copied stock photos). Tests should write to temp dirs and never modify user's personal photos.

| Fixture | Properties | Expected |
|---|---|---|
| `good_01.jpg` ... `good_20.jpg` | JPEG 4000x3000 or smaller test dimensions where not testing policy | 20 records import; for Adobe export need >=4MP per applicable media rule |
| `same_bytes.jpg` | Exact duplicate content of `good_01.jpg` | Same SHA256, duplicate detection |
| `same_name/pathA.jpg`, `same_name/pathB.jpg` | Same basename, different bytes | Index both; warn/block unsafe CSV collision |
| `very_long_filename...jpg` | Filename > current Adobe CSV limit | Validation error |
| `long_title.jpg` | Title 71 chars | `ADOBE_TITLE_TOO_LONG` |
| `bad_format.png` | PNG source | Can index, no direct Adobe JPEG export |
| `corrupt.jpg` | Truncated or invalid bytes | Scan continues, error shown |
| `unicode-é-猫.jpg` | Unicode filename | No silent replacement; reject if profile noncompliant |
| `ai_image.jpg` | provenance ai_generated | Adobe label reminder; Shutterstock blocked |
| `with_keywords.jpg` | 51 keyword strings, duplicate/mixed | Count and duplicate issues |

For performance smoke, use a separate batch of real-size synthetic images and report actual time/memory; don't confuse tiny fixtures with realistic performance.

## 3. Core acceptance cases

| ID | Case | Pass criteria |
|---|---|---|
| T-01 | Fresh start | DB + dirs created, local UI works without network |
| T-02 | Create project | Valid name persists; bad name 422 |
| T-03 | Scan 20 JPEGs | 20 indexed; progress and summary accurate |
| T-04 | Rescan same root | No DB duplicates; metadata history remains |
| T-05 | Exact duplicate | Same hash flagged; not overwritten |
| T-06 | Save metadata | Revision increment, preserved across restart |
| T-07 | Conflict update | stale revision → 409 |
| T-08 | Review invalidation | edit reviewed asset resets `needs_review` |
| T-09 | Adobe policy | 71-char title/invalid provenance/missing rights blocked |
| T-10 | CSV export | exact headers, UTF-8, filename mapping, keyword order, parseable |
| T-11 | Split limits | each CSV <=5,000 data rows and <=1 MB |
| T-12 | Missing source | marker `missing`; no data loss |
| T-13 | Non-JPEG file | indexed, not incorrectly declared Adobe JPEG-ready |
| T-14 | Crash during scan | job checkpoint persisted; retry idempotent |
| T-15 | Unavailable AI | no core failure; manual metadata available |
| T-16 | Secure paths | arbitrary path/symlink escape rejected |
| T-17 | Host/CSRF | remote origin blocked |
| T-18 | Read-only source | SHA256 all input files unchanged |
| T-19 | Export with invalid items | actionable issue list; no silently invalid CSV |
| T-20 | Status semantics | transfer does not imply submitted/approved |

## 4. Golden CSV rules

- Header order must match the **official template verified at implementation date**. Use a pinned JSON rule fixture with the expected set/order.
- CSV encoded UTF-8 and parses via standard library; exact filenames retain case/extension.
- Quote commas inside keyword cell correctly, protect formula injection, no newline corruption, no accidental BOM unless official template requires one.
- Title/max keyword limits versioned; release name category consistent.
- Each export has `manifest.json` containing profile/version, input asset revisions, output names, rows, SHA-256 and timestamps.

## 5. Performance & resource verification

Reference environment should include laptop with low resources if available (4GB RAM class). Record: backend idle RSS, p95 API response for asset list 100/1000 rows, scan 20 and 200 media, thumbnail peak memory, disk writes, import throughput. **Do not fabricate benchmarks**. If no such machine is available, report limitation and provide reproducible commands and expected metric definitions.

## 6. Security tests

- Attempt `../`, absolute path, drive prefix, UNC path and symlink route bypass.
- Attempt Cross-Origin mutating request from untrusted origin.
- Supply `title="=HYPERLINK(...)"` and other spreadsheet formulas; verify safe rejection/encoding.
- Return AI output with malicious instructions or non-JSON; verify no shell execution, only field-level diagnostics.
- Insert provider secrets in mock errors and verify redaction in logs/HTTP responses.
- Mock transfer success and assert `portal_status` stays `not_verified`.

## 7. Required commands, evidence and DoD

AI agent will define actual commands in generated codebase (`go test ./...`, web unit tests, build scripts, lint, E2E). Archive test results in `docs/IMPLEMENTATION_REPORT.md` with git commit, OS/build versions, pass/fail, screenshots if useful, known issues, measured resource usage, and actual verification of policy/version. **Never** mark a milestone complete based on successful compilation alone.

A milestone passes only when its features have UI flow, test, error path, and docs. No fake stubs labeled 'implemented'.
