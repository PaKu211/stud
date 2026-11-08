# Architecture Decision Records (ADRs)

## ADR-001 — Local-hosted web application, bukan desktop shell
**Status: accepted.** Go API & embedded static Svelte SPA dijalankan di `127.0.0.1:8080`. Tauri/Electron tidak diperlukan untuk MVP. Konsekuensi: folder selection awal via typed/paste directory path dengan validation, bukan desktop native picker.

## ADR-002 — Go over Rust sebagai backend MVP
**Status: accepted.** Go dipilih untuk packaging, HTTP jobs, koncurrency, cross-platform. Rust hanya jika profiling kemudian menunjukkan bottleneck. Jangan rewrite setelah sprint dimulai tanpa ADR baru.

## ADR-003 — SQLite and local filesystem first
**Status: accepted.** Single user, small batch, no cloud credentials. DB access melalui repository interface; migrations terpisah; app-owned cache/export dirs. Multi-user cloud nanti perlu DB dan storage adapters.

## ADR-004 — Optional AI through provider interface
**Status: accepted.** AIStudio2API optional localhost gateway, jangan dependency startup. Provider officially-supported API juga bisa dipilih; explicit opt-in untuk asset transfer. Structured output parse+validate, review human.

## ADR-005 — Separate status domains
**Status: accepted.** `asset_review_status` ≠ `distribution_status`; remote submission dan approval hanya saat ada explicit evidence, bukannya inference dari SFTP return code.

## ADR-006 — Minimal release profile Adobe Stock CSV
**Status: accepted.** Prioritaskan Adobe image JPEG, local CSV. Multi-marketplace/other media later; rules and examples di test fixtures. Version `adobe-2026-06` sebagai label profil awal, bukan klaim kebijakan tak berubah.

## ADR-007 — No destructive source transformations
**Status: accepted.** Import indeks/read-only; derivatives ke storage app. Metadata embedding in-place hanya bila user memilih opsi non-default dengan backup, masuk fase lanjutan.

## ADR-008 — Embedded frontend vs SSR
**Status: accepted.** SPA statis via Vite; Go serves frontend artifacts using `embed`. Tidak membutuhkan SSR dan Node runtime saat aplikasi dipakai. Agent wajib menguji packaging kedua OS.

## ADR-009 — Queue persisted, not a perpetual worker daemon
**Status: accepted.** Local jobs disimpan di DB, goroutine workers dalam proses Go. Saat restart, `running` job direkonsiliasi menjadi retryable/paused lalu diproses sesuai aturan. Tidak menjalankan in-process jobs ketika app ditutup; cloud mode nanti pakai worker separate.

## ADR-010 — Never rely on portal DOM for MVP compliance
**Status: accepted.** Browser extension RJ AIO adalah referensi adapter. Tidak memasukkan bot auto-submit, termasuk bypass anti-bot, ke MVP. User tetap mengerjakan langkah terakhir di portal resmi.

## ADR-011 — Definition of AFD/DBD/BSD
**Status: accepted for documentation.** AFD = Application Flow Design, DBD = Database Design, BSD = Backend Service Design. Perlu revisi hanya jika user mempunyai definisi lain.

## ADR-012 — Policy versioning
**Status: accepted.** Rule and validation report menyimpan `marketplace`, `rules_version`, `checked_at`, dan evidence URL. Ketika rules berubah, batch lama harus bisa divalidasi ulang; jangan mengubah status tanpa event.
