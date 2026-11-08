# BSD — Backend Service Design

**Acronym chosen:** Backend Service Design. Implementation language Go.

## 1. Service responsibilities

The backend is a **modular monolith**, not microservices, so it works on a laptop. Package boundaries are kept clean enough to extract workers or storage adapters later.

| Module | Responsibility | Main interfaces/commands |
|---|---|---|
| `project` | Projects and registered allowed roots | `CreateProject`, `AddRoot`, `ListProjects` |
| `catalog` | Scan/index/hash/thumbnail/filter | `ScanRoot`, `GetAsset`, `ListAssets` |
| `metadata` | Edit, revisions, provenance, releases, review | `PatchMetadata`, `MarkReviewed` |
| `validation` | Marketplace rule checks | `ValidateSelection` |
| `export` | CSV, manifest, atomic files | `BuildAdobeExport`, `GetExport` |
| `job` | Queue, worker claims, progress, restart reconciliation | `Enqueue`, `Claim`, `Progress`, `Complete`, `Cancel`, `Retry` |
| `audit` | Human decisions and safe event log | `RecordAuditEvent` |
| `provider` (MVP-B) | AI text/vision gateway | `GenerateDraft`, `ListModels`, `Health` |
| `media` (MVP-B/C) | ExifTool, FFmpeg, QC & derivatives | `Probe`, `Transform`, `EmbedMetadata` |
| `distribution` (MVP-C) | Upload through supported official transport | `Prepare`, `Transfer`, `VerifyReceipt` |

## 2. Proposed package layout

```text
microstock-automation-studio/
├── cmd/studio/main.go
├── internal/
│   ├── app/                   # dependency wiring, config, lifecycle
│   ├── httpapi/               # router, DTOs, middleware, static embedded UI
│   ├── domain/                # types, invariants, errors
│   ├── project/
│   ├── catalog/
│   ├── metadata/
│   ├── validation/
│   ├── exporter/
│   ├── jobs/
│   ├── audit/
│   ├── integrations/
│   │   ├── ai/                # optional OpenAI-compatible adapter
│   │   ├── media/             # optional ExifTool, ffprobe
│   │   └── marketplace/       # optional SFTP; no Adobe contributor API
│   ├── persistence/sqlite/
│   └── storage/local/
├── migrations/
├── web/                        # Svelte + TS + Vite
├── rules/adobe/
├── testdata/
├── docs/
└── README.md
```

Do not create separate network services simply for module names. External AIStudio2API is a distinct optional process managed by user.

## 3. Startup, shutdown, recovery

1. Read config; validate app data dirs, config values, and loopback binding.
2. Create required dirs with restrictive permissions, open SQLite, configure FK/WAL/busy timeout.
3. Apply migrations with prior backup; fail closed on corrupt DB and point to recovery docs.
4. Reconcile jobs left `running` at crash: event `interrupted`, set `paused` or schedule deterministic resume according to job type.
5. Start bounded workers and HTTP server; graceful shutdown stop taking new work, cancel/flush worker checkpoints and close DB.
6. Open browser only as convenience, not a dependency for processing.

## 4. Command details

### `ScanRoot`
Inputs: `project_id`, `root_id`, selection policy; outputs `job_id` and eventually summary. Check root whitelisted and realpath in authorized subtree. Walk directory with configurable maximum depth and file count; stream hashing, inspect dimensions with bounds; write per-file in short transactions. `upsert` by `(root_id,relative_path)`, preserve metadata. Track count success/skipped/errors. Avoid queuing one goroutine per file; no source file moves.

### `PatchMetadata`
Inputs: asset ID, expected revision, patch of title/description/ordered keywords/category/source_kind/rights. Begin DB tx; verify revision; normalize `keywords` to array; increment revision; insert immutable snapshot; set `needs_review` and `reviewed_at=NULL`; emit audit event. Return 409 on conflict, 422 for invalid fields.

### `MarkReviewed`
Inputs: asset_id + expected metadata revision; require user assertion provenance/rights and release consistency; set reviewed_revision; no Adobe policy assumption yet. Marking reviewed is not official submission.

### `ValidateSelection`
Load asset snapshot and policy rules; check file presence/format, rights, AI classification, title, filename, category, keywords order/count, required releases and duplicate/too-similar warnings; record `validation_run`, per-asset `validation_items` snapshot (metadata revision and source hash) and issues. Validation is pure and tests cover each issue.

### `BuildAdobeExport`
Ensure selected assets have latest valid state for requested rules version; if not revalidate first. Group by conflict-free target filenames; serialize stable sort; split at size/row boundaries; atomic write with manifest. Persist `export_batches` plus `export_items` in transaction that reflects actual durable files. On failure mark failed and clean temp. Do not include source images in CSV bundle by default.

### `EnqueueJob`
`idempotency_key = sha256(type + sorted selected IDs + metadata revision + settings hash)`; use unique insert or return existing active job. Each worker claims job atomically, records events and per-asset results. Cancellable context checked at predictable points; retries exponential backoff and jitter for transient AI/network errors only, not for permanent validation failures.

## 5. Separation of rule engines

`MarketplaceProfile` includes: `{id, version, evidence_urls, accepted_media, title_limit, keyword_hard_limit, keyword_recommended_limit, filename_limit, category_taxonomy, requires_ai_label, accept_ai, requires_releases, csv_column_order}`.

- `AdobeCSVProfile`: `accept_ai=true` if user flags correct, requires appropriate rights/releases, export primarily JPEG for MVP.
- `ShutterstockProfile` (later): generative AI source-kind must block upload (per current policy), even if other metadata validates.
- `UnknownProfile`: default **deny distribution**, not assume accepts AI.

Rules version snapshot must accompany validation and export. Automated rule updates must not overwrite locally reviewed content.

## 6. Event model

Events use `{event_id, event_type, entity_id, created_at, correlation_id, safe_details}`. Examples: `root.added`, `scan.started`, `asset.indexed`, `asset.unreadable`, `metadata.updated`, `metadata.reviewed`, `validation.completed`, `export.created`, `job.retried`; later `transfer.confirmed`, `portal.submitted_manual`, `portal.approved_manual`. Maintain explicit provenance for reported portal events.

## 7. Error taxonomy

| Code | HTTP | Meaning/handling |
|---|---|---|
| `VALIDATION_ERROR` | 422 | One or more fields invalid; list field issues |
| `INVALID_ROOT` | 422 | Nonexistent, inaccessible or outside permitted path |
| `NOT_FOUND` | 404 | Unknown asset/project/job |
| `REVISION_CONFLICT` | 409 | Metadata changed since client's revision |
| `RESOURCE_CONFLICT` | 409 | Duplicate target filename/path or already running incompatible job |
| `NO_EXPORTABLE_ASSETS` | 422 | All assets blocked/none selected |
| `STORAGE_ERROR` | 500/507 | Data dir permission/disk full; no silent success |
| `PROVIDER_UNAVAILABLE` | 503 | Optional AI service missing; manual flow works |
| `RATE_LIMITED` | 429/503 | Provider throttle; bounded retry |
| `POLICY_BLOCKED` | 422/403 | Distribution prohibited under active marketplace rules |
| `INTERNAL_ERROR` | 500 | Safe message, request ID, server logs sanitized |

## 8. Security boundaries

- API callers: localhost only until auth designed for network.
- Filesystem: allowed user-added root + app-owned directory only, no request may specify arbitrary path after registration.
- Credential management: env/OS vault; no credentials returned via API.
- AI: explicit user opt-in, escape prompt data, treat output as untrusted JSON.
- Marketplace: transfer and submission distinct; credentials scoped to target and audited.
- Observability: counters for jobs/success/failed, optional debug logs with redaction. Do not log full AI prompts carrying private media paths.

## 9. External integrations detailed later

- AI adapter should use `/v1/models` to detect models and `/v1/chat/completions` with image inputs when supported. It must test capabilities rather than infer all models have Vision.
- ExifTool and FFmpeg use external process timeouts; quote arguments via `exec.CommandContext` and do not pipe untrusted input into shells.
- SFTP transfer uses known host fingerprint validation (not `InsecureIgnoreHostKey`), destination path rules, sanitized logs, and confirmation of eligible account.

## 10. Testability requirements

All business logic interfaces support in-memory fakes. Unit tests for validation, metadata revision, export split and queue idempotency; integration tests for SQLite rollback/foreign keys/migrations and filesystem paths; SFTP/AI tests use mock transports unless explicit live-test mode. No live marketplace uploads in CI.
