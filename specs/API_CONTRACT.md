# API Contract — v1

**Base:** `http://127.0.0.1:8080/api/v1` · JSON UTF-8, ISO-8601 UTC timestamps, opaque UUID IDs. **Not publicly exposed**. These are proposed contracts for agent implementation; route shapes must remain consistent with the frontend and tests.

## 1. Conventions

- Request `Content-Type: application/json`. Responses: `200` GET/action, `201` create, `202` queued jobs, `204` no payload, `4xx` client error, `5xx` server error.
- Pagination: `?limit=50&cursor=...` (bounded `limit` 1–100); return `{items:[...],next_cursor:null}`.
- `revision` in PATCH is required. No blind last-write-wins.
- Never return `absolute_path` or credential bytes. Relative path is presentational only.
- All mutating calls protect against CSRF/Origin/Host misuse on localhost; if using CSRF token, document its retrieval in generated OpenAPI docs.

## 2. Endpoints

| Method / path | Input | Successful output |
|---|---|---|
| `GET /health` (outside /api/v1) | - | `{"status":"ok","db":"ready"}` |
| `GET /projects` | cursor | paginated projects |
| `POST /projects` | `{name}` | 201 project `{id,name,created_at}` |
| `GET /projects/{id}` | - | project details |
| `POST /projects/{id}/roots` | `{absolute_path}` | 201 root `{id,display_path}` |
| `GET /projects/{id}/roots` | - | roots |
| `POST /roots/{id}/scan` | `{rescan:true}` | 202 `{job_id}` |
| `GET /projects/{id}/assets` | status,type,search,limit,cursor | paginated asset summaries |
| `GET /assets/{id}` | - | asset detail & metadata |
| `GET /assets/{id}/thumbnail` | - | binary image or safe error image status |
| `PATCH /assets/{id}/metadata` | `{expected_revision, title?, description?, keywords?, category?, language?, source_kind?, rights_status?}` | `{metadata,revision,review_status:"needs_review"}` |
| `POST /assets/{id}/review` | `{expected_revision,rights_confirmed:true}` | metadata reviewed state |
| `POST /projects/{id}/validate` | `{marketplace:"adobe",asset_ids:[...]}` | 202 `{job_id}` or sync `validation_run_id` (choose async consistent) |
| `GET /validations/{id}` | - | summary and per-asset issues |
| `POST /projects/{id}/exports` | `{marketplace:"adobe",asset_ids:[...]}` | 202 `{job_id}` |
| `GET /projects/{id}/exports` | - | export history |
| `GET /exports/{id}` | - | manifest & parts metadata |
| `GET /exports/{id}/files/{part}` | part int | UTF-8 CSV download (allowlisted export-only path) |
| `GET /jobs/{id}` | - | job status/progress/result |
| `GET /jobs/{id}/events` | cursor | job events |
| `POST /jobs/{id}/retry` | `{}` | 202 new or resumed job |
| `POST /jobs/{id}/cancel` | `{}` | 202 cancel requested |
| `GET /integrations` | - | enabled/available providers without secrets |
| `GET /integrations/ai/models` (MVP-B) | - | capabilities/models or 503 |
| `POST /assets/{id}/ai-draft` (MVP-B) | `{provider_id,model_id,confirm_external_send:true}` | 202 `{job_id}` |

Use `GET /health` path outside `/api/v1` as explicitly listed. `POST /validate` returns a job for consistency; `GET /jobs/{id}` gives `result.validation_run_id`.

## 3. Examples

**Create project:**
```http
POST /api/v1/projects
Content-Type: application/json

{"name":"Adobe Batch November"}
```

**Asset summary (representative)**
```json
{
  "id": "e4c3f63d-a900-45a1-9ea8-189a24f57342",
  "display_filename": "forest_001.jpg",
  "relative_path": "nature/forest_001.jpg",
  "media_type": "image/jpeg",
  "width": 4000,
  "height": 3000,
  "sha256": "...",
  "source_kind": "ai_generated",
  "rights_status": "confirmed",
  "review_status": "needs_review",
  "metadata_revision": 2,
  "issue_count": 0
}
```

**Save metadata:**
```http
PATCH /api/v1/assets/e4c3f63d-a900-45a1-9ea8-189a24f57342/metadata
Content-Type: application/json

{"expected_revision":2,"title":"Misty evergreen forest in early morning","keywords":["forest","mist","trees","nature"],"category":"11","source_kind":"ai_generated","rights_status":"confirmed"}
```

**Validation result:**
```json
{
  "id": "<validation-id>",
  "marketplace": "adobe",
  "rules_version": "adobe-2026-06",
  "checked_at": "2026-11-08T05:00:00Z",
  "eligible_count": 18,
  "blocked_count": 2,
  "issues": [
    {"asset_id":"<asset-id>","field":"title","code":"ADOBE_TITLE_TOO_LONG","severity":"error","message":"Judul melebihi 70 karakter","max":70}
  ]
}
```

**Error standard:**
```json
{
  "error": {
    "code": "REVISION_CONFLICT",
    "message": "Metadata sudah berubah. Muat versi terbaru.",
    "details": {"current_revision": 3},
    "request_id": "<id>"
  }
}
```

## 4. Job response contract

```json
{
  "id":"<job-id>",
  "type":"scan_assets",
  "status":"running",
  "progress":{"completed":8,"total":20,"failed":1},
  "attempts":1,
  "error_code":null,
  "result":null,
  "created_at":"2026-11-08T05:00:00Z"
}
```

Client can poll every 1–2s while job active with backoff when idle. SSE may be added later but is **not required** in MVP-A; do not force WebSocket infrastructure.

## 5. Invariants to verify in API tests

- A nonexistent ID is 404, not 200 with empty object.
- Root scan rejects traversal and symlink escape.
- Metadata PATCH creates new revision and invalidates review.
- Review of old revision is 409.
- Export with no passing items is 422.
- No API endpoint permits arbitrary absolute path read.
- Request cannot alter portal `submitted` state via transferring file.
- User-provided content is never returned without output escaping in frontend.
