# Implementation Plan — Agent Execution Backlog

**Principle:** build vertical slices with working UI + backend + tests instead of implementing every backend package first. Do **not** generate Tauri app, Render/Appwrite infrastructure, or autonomous bot for first milestone. Time estimates intentionally omitted; verify based on implementation results.

## Phase 0 — Repo audit and environment proof (must finish first)

**Tasks**
- [ ] Create new Git repository and pin current Go, Node, Svelte/Vite, SQLite driver versions.
- [ ] Read all planning docs and create `docs/OPEN_QUESTIONS.md` only for truly blocking contradictions; resolve non-blocking implementation assumptions in ADR.
- [ ] Clone/pin audited upstream commits of ToTag, Stockflow, RJ AIO, AIStudio2API **outside app runtime dependency**, and write `docs/UPSTREAM_AUDIT.md` + `docs/upstream-lock.json`.
- [ ] Verify official Adobe CSV template and version, live source links, source/media limits.
- [ ] Prove minimal Hello World Go server can serve compiled Svelte assets and use SQLite under Windows/Linux.
- [ ] Create stubbed CI for Go tests, frontend typecheck, lint, build; no claimed production readiness.

**Gate G0:** Architectural choices validated, upstream research recorded, core tools compile, source/copyright risks logged.

## Phase 1 — Skeleton and persistence

**Tasks**
- [ ] Implement project structure from BSD, versioned config, app data dir, loopback binding, startup/migration/shutdown.
- [ ] Implement SQLite v1 schema, repository interfaces, transaction helpers and migration backup.
- [ ] Build Svelte shell with Projects, Assets, Jobs, Settings route stubs that clearly say 'Not implemented'.
- [ ] Add `/health`, `/api/v1/projects` create/list/detail endpoints and tests.
- [ ] Add structured errors/request IDs, Host/Origin validation, CSRF protection, logs redaction.

**Gate G1:** New project persists across restart; Linux/Windows binary boots without Docker, cloud, AI or ExifTool.

## Phase 2 — Asset catalog (first usable slice)

**Tasks**
- [ ] Root registration validation, realpath and symlink confinement.
- [ ] SQLite-backed job engine (enqueuing, checkpoints, job events, retry/cancel and interrupted job recovery).
- [ ] Scan root, file identification, dimensions, streaming SHA256, duplicate grouping, incremental rescan.
- [ ] Thumbnail generator with bounds and error fallback.
- [ ] Assets grid, project filters, detail view, and job progress UI.
- [ ] Add T-02/T-05/T-12/T-14/T-16 integration tests.

**Gate G2:** 20 JPEGs can be indexed, previewed and survive restart; re-scan idempotent; source hashes unchanged.

## Phase 3 — Metadata & review

**Tasks**
- [ ] Metadata CRUD with optimistic revision and revision history.
- [ ] Ordered keywords editor, title/description/category controls; provenance and rights flags; release records.
- [ ] `mark-reviewed`, multi-select editing with confirmation and audit events.
- [ ] Prevent stale edits and reset reviewed state on mutation.
- [ ] Unit/integration tests for model constraints, revisions, review state and user confirmation.

**Gate G3:** Metadata edit/review persists; clear conflict handling; no accidental source modifications.

## Phase 4 — Adobe validator & exporter (MVP-A completion)

**Tasks**
- [ ] Implement versioned `AdobeCSVProfile` with official CSV format and policy links.
- [ ] Implement rules: title, filename, keyword count/order, valid category, media format, rights/AI and release checks.
- [ ] Validation UI with field-level code, explanation and suggestions.
- [ ] Deterministic CSV serializer, size/row splitting and safe filenames; atomic file + manifest writing.
- [ ] Export History screen with clear file path and row/byte summary.
- [ ] Golden fixtures, Adobe policy tests, end-to-end 20 JPEG manual flow, restart and security tests.

**Gate G4 = MVP-A release:** complete acceptance T-01..T-20 excluding future-only full network integration, no critical security issue, zero unexpected source edits; attach implementation report. This is the **first handoff checkpoint to the user**.

## Phase 5 — Optional AI & advanced QC (MVP-B)

- [ ] Implement `AIProvider` adapter and settings, support model listing and feature detection.
- [ ] Connect AIStudio2API `/v1/models` and OpenAI-compatible chat vision with explicit data-send confirmation.
- [ ] JSON schema validation of model output, human draft preview/apply/reject; retry/backoff; graceful offline.
- [ ] Add optional ExifTool metadata embedding with derivative/backup and explicit opt-in.
- [ ] Add image QC for size/artifact heuristics, perceptual similarity warnings with tests.

**Gate G5:** Disabled/unavailable AI doesn't break G4; no auto-apply of AI output; all external transfers require consent.

## Phase 6 — Distribution and video (MVP-C)

- [ ] Verify SFTP entitlement and official policy; add secure host-key verified SFTP transport for Adobe only when eligible.
- [ ] Build Transfer Center and `distribution_records`; manual portal submission evidence entry.
- [ ] Add file-level receipt, mismatch/retry reconciliation, no false status.
- [ ] Add video probe and optional FFmpeg transforms using Stockflow QC references, with per-site profiles and copy-on-write derivatives.
- [ ] Research additional marketplaces one at a time; default deny unknown AI policy and no auto-submit.

**Gate G6:** Mock transfer and permitted live smoke tests (explicit user opt-in) pass; `transfer_success` never implies `submitted`.

## Phase 7 — V2 roadmap (not MVP deliverables)

- [ ] Prompt library, niche research workflow, AI image/video generation from provider with provenance and cost controls.
- [ ] Visual workflow node editor, agent/MCP permissions and approval gates.
- [ ] Cloud design: identity/tenant, object storage, durable worker, Postgres, billing/resource quotas and secret service.
- [ ] Optional desktop wrapper after actual need, rather than as core architecture.

## Agent working protocol per task

1. Read relevant PRD/story, architecture/DB/API contract and test gate.
2. Create a tiny vertical slice and a test that fails before implementation.
3. Implement code, run formatter/types/lint/tests, fix failures.
4. Document output: changed files, tests, decisions, limitations.
5. Update `docs/IMPLEMENTATION_REPORT.md` and only then check off task.
6. If official policy and spec disagree, **official policy controls compliance**, record change in ADR and update golden fixtures before shipping.
7. Never silently scope-creep. Build G0→G4 first, stop and report ready for user review before G5.

## Traceability matrix

| Requirement | Primary module | Test gates | Milestone |
|---|---|---|---|
| P-01 | project + DB | T-01,T-02 | G1 |
| P-02 | catalog/jobs | T-03,T-04,T-14,T-16 | G2 |
| P-03 | catalog/storage | T-05,T-12 | G2 |
| P-04,P-05 | metadata/provenance | T-06,T-07,T-08 | G3 |
| P-06 | rules/validation | T-09,T-13,T-19 | G4 |
| P-07 | exporter | T-10,T-11 | G4 |
| P-08 | jobs/audit | T-14,T-17,T-18 | G2/G4 |
| P-09 | provider | T-15 + AI mocks | G5 |
| P-11 | distribution | T-20 + SFTP mocks | G6 |
