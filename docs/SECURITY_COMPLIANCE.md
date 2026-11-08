# Security, Licensing & Marketplace Compliance

Policy review date **2026-11-08**. Official platform policies change: every live uploader release must re-verify current terms, formats and eligibility. This is a technical product design document, not legal advice.

## 1. Platform-specific verified facts

### Adobe Stock

- Adobe Developer Stock API FAQ explicitly states **no public Contributor API** for uploading content, retrieving sales data, or managing contributor submission. Do not invent API endpoints for it. Source: https://developer.adobe.com/stock/docs/faq/
- Contributor Portal handles upload/metadata/releases and **submission for moderation**. Some qualified contributors may use SFTP for file transfer. SFTP eligibility must be checked against contributor account UI. Source: https://helpx.adobe.com/in/stock/contributor/help/uploading-content.html
- CSV documentation updated 11 June 2026 states: `.csv`, max **5,000 rows**, max **1 MB**, UTF-8; required headers following current official template, `Filename` up to 30 chars, `Title` up to 70, `Keywords` up to 50 comma-separated keywords. Profile must be test-backed with current template (do not treat assumptions as certainties). Sources: https://helpx.adobe.com/stock/contributor/manage-your-portfolio/csv-requirements-content.html and https://helpx.adobe.com/stock/contributor/manage-your-portfolio/create-csv-file.html
- Another Adobe keywords guide (2 Sep 2026) suggests **up to 49 keywords** in contributor submission. To be safe, validator should default to **49 recommended**, CSV spec hard ceiling **50** and warn on discrepancy; re-evaluate from policy at release. Source: https://helpx.adobe.com/id_id/stock/contributor/content-policies-guidelines/metadata/tips-effective-titles-keywords.html
- Adobe accepts generative AI subject to labeling, technical standards, rights and metadata; the `Created using generative AI tools` flag must be selected at submission. Source: https://helpx.adobe.com/in/stock/contributor/submit-your-content/submit-generative-ai-content/submit-generative-ai-content.html
- Very similar AI variants may be refused; do not implement bulk generation solely by flips/filters/crop changes. Source: https://helpx.adobe.com/id_id/stock/contributor/submit-your-content/submit-generative-ai-content/distinct-generative-ai-submission-best-practices.html

**Adobe design response:** The app prepares CSV and metadata/asset readiness; it must show a compliance checklist that AI label and model/property releases need to be confirmed in the **Contributor Portal**; CSV alone does not guarantee final classification or submission.

### Shutterstock

- Contributor help explicitly states generative AI output must **not** be submitted by contributors. Source: https://submit.shutterstock.com/help/id/articles/10594633-mengapa-konten-saya-ditolak-sebagai-konten-yang-tidak-dapat-dilisensikan
- **Guardrail:** for `source_kind=ai_generated`, `ShutterstockPolicy.CanDistribute=false`. Do not allow overrides that bypass an explicit prohibition. Other media categories require policy research before uploader is enabled.

### Other marketplaces

Pond5, Dreamstime, Vecteezy, Freepik, Depositphotos, MiriCanvas: not automatically approved just because repository adapters exist. The agent must separately verify each provider's generative-AI rules, upload interface, acceptable content and ToS before releasing its adapter. Default `disabled/not_verified`.

## 2. User rights and content provenance

- `source_kind` must be explicit: AI-generated, human-created photo, human-created illustration, or unknown.
- Capture license/right-to-submit attestation. Protect user from accidental submission of stock images downloaded from others or trademarked/copyrighted artwork.
- Capture model/property release status separately; `missing`, `available`, `not_required` are user assertions, not verified legal judgments.
- Metadata should not claim actual celebrity, brand, restricted IP, fictional characters, or real news event for AI content. Alerts can be shown; cannot guarantee perfect semantic detection.
- Generated AI assets require provider terms/license permitting commercial stock distribution; agent must record provider/model provenance and applicable terms at creation time in V2.
- Do not imply any expected royalties/sales or approval rates.

## 3. Threat model summary

| Threat | Scenario | Defense |
|---|---|---|
| Localhost CSRF / DNS rebinding | Malicious site triggers local API | Bind 127.0.0.1, Host allowlist, Origin and CSRF validation, restrictive CORS |
| Path traversal | API request reads arbitrary file | Use opaque IDs, canonical allowlisted roots, path join containment + symlink checks |
| Secret leak | Logs/CSV expose API keys | Env/OS secret store, redaction, never frontend persistence |
| Data exfiltration | AI metadata sends personal images remotely | Explicit opt-in and provider/data preview, default disabled |
| Prompt injection | Metadata/image text instructs AI to run shell | AI returns untrusted JSON only; no tool execution from untrusted content |
| Duplicate transfer | Retry reuploads after network timeout | Receipt reconciliation and `verification_required`, no blind infinite retries |
| Malicious media | Huge or crafted image crushes memory | Inspect headers first, bounded decode, CPU/memory/time limits, safe external tool invocation |
| Malicious CSV | Excel formula interpreted on open | Validation/escape dangerous formulas while preserving platform compatibility |
| Credential theft | SFTP password rendered in browser | Store server-side only, never response/log; use strict host key verification |
| Dangerous host binding | User exposes local file API to LAN | Default loopback, block non-loopback until authentication/network hardening exists |

## 4. License & source reuse rules

- ToTag, Stockflow, RJ AIO Metadata, AIStudio2API display MIT license on repository pages as of review; agent must capture actual license text and notices for code reuse.
- Source code reuse must comply with attribution, license preservation and dependency notice obligations; copying repo README assumptions without testing is insufficient.
- Xpiks GPL-3.0 should be used as UX inspiration only unless downstream distribution/legal strategy intentionally complies with GPL.
- Adobe Stock Generator source reuse prohibited in this plan until its license and reuse rights are confirmed.
- ExifTool and FFmpeg redistributed binaries may have separate licensing obligations; prefer optional tool detection and document official installation rather than silently bundling files.
- Report dependency lockfile, SBOM/license notices, and security vulnerabilities before public release.

## 5. Secure operations checklist

- [ ] Backend only binds loopback.
- [ ] No source edits by default; hashes confirmed unchanged in tests.
- [ ] User confirms AI external data transfer.
- [ ] User confirms SFTP target, entitlement and selected assets (later).
- [ ] Secrets excluded from logs/API/config examples.
- [ ] Path traversal and symlink escape tests pass.
- [ ] All policy restrictions have rule IDs and evidence links.
- [ ] No portal submit/payout claims without evidence.
- [ ] SFTP transport checks known host fingerprint; no insecure host key acceptance.
- [ ] Rule versions and upstream audit commits documented.
- [ ] Go/backend, JS deps, Go embed and binary packaging scanned before release.

## 6. Cloud security prerequisite, not MVP

Public deployment requires: authentication, authorization per project/assets, object storage signed access, encrypted secrets, tenant isolation, rate limits, CSRF/CORS policy, durable background workers, file malware screening as appropriate, quotas/billing guardrails, privacy policy, deletion/retention flows. No direct migration from local bind to `0.0.0.0` without these changes.
