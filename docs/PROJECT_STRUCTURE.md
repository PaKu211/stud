# Planned Source Tree (Blueprint — not created application code)

The structure below is the **target codebase** for the coding agent to create. The planning package itself does not contain runtime application files.

```text
microstock-automation-studio/
├── README.md
├── LICENSE                    # newly chosen license; respect upstream notices
├── AGENTS.md                  # agent governance sourced from handoff
├── go.mod
├── go.sum
├── .gitignore
├── .env.example               # no secrets
├── Makefile                   # optional, cross-platform commands also provided
├── cmd/
│   └── studio/
│       └── main.go
├── internal/
│   ├── app/                   # dependency injection, startup & shutdown
│   ├── config/
│   ├── domain/                # core types, validation-independent invariants
│   ├── httpapi/               # router, DTOs, middleware and embed assets
│   ├── project/
│   ├── catalog/
│   ├── metadata/
│   ├── validation/
│   ├── exporter/
│   ├── jobs/
│   ├── audit/
│   ├── integrations/
│   │   ├── ai/
│   │   ├── media/
│   │   └── marketplace/
│   ├── persistence/
│   │   └── sqlite/
│   └── storage/
│       └── local/
├── migrations/               # numbered .sql
├── rules/
│   └── adobe/
│       ├── rules.json        # policy version + evidence URLs
│       └── testcases.json
├── web/
│   ├── package.json
│   ├── package-lock.json
│   ├── vite.config.ts
│   ├── src/
│   │   ├── main.ts
│   │   ├── App.svelte
│   │   ├── lib/
│   │   │   ├── api/
│   │   │   ├── components/
│   │   │   ├── stores/
│   │   │   └── types/
│   │   └── routes/           # app-local route components
│   └── dist/                 # build output; ignored or embedded in release
├── testdata/                  # generated media, golden CSV and manifests
├── scripts/                   # dev/build/smoke commands (Windows and Linux)
├── docs/
│   ├── API.md
│   ├── INSTALL.md
│   ├── ARCHITECTURE.md
│   ├── UPSTREAM_AUDIT.md
│   ├── UPSTREAM_LOCK.json
│   ├── IMPLEMENTATION_REPORT.md
│   └── ADR/
└── build/                     # generated packages, never credentials
```

## Operational data tree

```text
<OS user data dir>/microstock-studio/
├── studio.sqlite
├── config.json           # no plaintext secrets
├── cache/thumbs/{asset-id}/
├── exports/{export-id}/
│   ├── adobe-part-001.csv
│   ├── adobe-part-002.csv
│   └── manifest.json
└── logs/                  # sanitized, optional
```

Source folders remain at their original user-selected locations; scanning does not move or duplicate them by default. In later cloud mode, these paths are replaced by storage URIs through repository/storage interfaces.

## Build and run behavior

**Development:** Go API on `127.0.0.1:8080`, Vite SPA dev server on own localhost port, with proxy to API. **Release:** build Vite, embed frontend assets into Go server, ship Go binary + OS install/run notes; no Node runtime required to use released app. sqlite is compiled into Go package via supported driver; no external DB server. Optional tools detected separately.

## Architecture guardrails

- No SQL in HTTP handlers.
- No raw filesystem paths accepted by thumbnail/export-download endpoints.
- No external AI calls in validation rules.
- No cyclic imports across domain/use case/adapters.
- No direct mention of `Appwrite` or `Render` in core business logic.
- All marketplace-specific behavior isolated behind versioned profiles.
- Keep ownership and distribution events audit-ready before cloud migration.
