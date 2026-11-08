# AGENTS.md — Implementation Agent Rules

This package is a **design handoff**, not application source. The human requested detailed planning documents so a separate coding agent can implement the app.

1. Read `HANDOFF_PROMPT.md` and the docs it references before creating code.
2. Implement Go + Svelte SPA + SQLite local first. No Rust, Tauri, Render, Appwrite, Docker dependency for G4.
3. Protect originals, explicit review, strict local path boundaries, offline core behavior.
4. AIStudio2API optional; consent required before any external media send.
5. Check official policy, do not invent Adobe Contributor APIs, do not send AI assets to Shutterstock.
6. Test real behavior; document evidence, failures and uncertainty. Never claim tests passed without running them.
7. Never auto-submit/claim approval based on file transfer. Require evidence.
8. Preserve upstream attribution and audit licenses, pinned commits and dependencies.
9. Prioritize G0-G4; do not scope-creep into V2 unless requested.
10. Provide actual build/run commands, implementation report and checklist after each milestone.
