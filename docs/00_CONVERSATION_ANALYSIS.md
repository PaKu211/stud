# 00 — Analisis Percakapan dan Reframing

## Permintaan inti yang stabil

User ingin workflow microstock end-to-end: riset ide dan tren → ide/prompt → membuat gambar/video (opsional via AI) → QC/upscale → metadata dan keyword → paket CSV → distribusi/upload → status submit/approval bila akses memungkinkan. Sistem tidak hanya metadata editor; perlu menjadi studio yang *auditable* dan bisa berkembang dengan automation dan AI agent.

## Evolusi solusi dalam transkrip

| Urutan | Arah sebelumnya | Perubahan | Status akhir |
|---|---|---|---|
| 1 | Menemukan repo GitHub & Reddit | Fokus pada komponen microstock pipeline | Dipertahankan |
| 2 | Reverse engineering ToTag, Stockflow, RJ AIO, Adobe Stock Generator | Integrasikan AIStudio2API | Dipertahankan sebagai strategi referensi dan gateway |
| 3 | Rust + Tauri + Svelte sebagai desktop | Ingin bisa akses sebagai web app/cloud | Rust/Tauri bukan prioritas |
| 4 | Web-first di Render/Appwrite | Ingin cepat diuji di laptop | Cloud ditunda |
| 5 | Go + Web UI + SQLite local-first | AI agent akan implementasi | Final untuk MVP |
| 6 | Assistant sempat menyatakan akan membuat starter code | User mengoreksi tugas assistant sebagai **perencana** | Jangan membuat code app sekarang; buat dokumen implementasi |

## Konflik yang diselesaikan

1. **Full automation vs MVP ringan**: full pipeline adalah visi, bukan syarat hari pertama. Milestone awal harus memberi hasil nyata (CSV siap diunggah) secara offline.
2. **Rust vs Go**: Go adalah backend MVP pilihan akhir; Rust bisa dievaluasi untuk hot path kelak, bukan alasan memulai ulang core.
3. **Web-first vs cloud**: local-hosted web app memakai antarmuka berbasis browser, dengan batasan filesystem lokal yang eksplisit; cloud port nanti memerlukan auth, object storage, job workers, dan secure secret management.
4. **AIStudio2API perlu vs opsional**: provider adapter memiliki interface; aplikasi tetap berfungsi ketika gateway mati.
5. **Reverse engineering vs copy-paste repo**: lakukan audit, dokumentasikan perilaku, tulis tes kontrak, implementasi ulang berdasarkan fitur/algoritme yang diizinkan lisensi; jangan menjahit repositori tanpa pemisahan dependensi.
6. **Upload vs submit vs approved**: ketiganya status berbeda dan memerlukan bukti yang berbeda; `uploaded` tidak menyiratkan `submitted`.
7. **Shutterstock dan AI**: generative AI tidak boleh didistribusikan ke Shutterstock berdasarkan kebijakan yang diverifikasi; aturan per platform harus versioned.

## Prioritas yang dideduksi, bukan dinyatakan eksplisit

- Svelte + TypeScript dipakai sebagai Vite SPA yang dibundel ke Go agar release cukup satu backend executable dan aset statis (bukan SvelteKit SSR). Ini keputusan desain baru yang paling memenuhi tujuan sederhana.
- SQLite dengan *pure-Go driver* direkomendasikan untuk build cross-platform tanpa toolchain C; build footprint perlu diuji pada mesin memori rendah.
- `BSD` ditafsirkan sebagai **Backend Service Design**; `AFD` sebagai **Application Flow Design**; `DBD` sebagai **Database Design**. Akronim tidak dijabarkan dalam transkrip.
- Fitur upload diundur ke fase sesudah CSV karena kelayakan SFTP berbeda per akun dan tidak tersedia Contributor API publik.

## Asumsi yang harus dibuktikan oleh agent

- Versi compiler Go/Node dan driver SQLite yang masih didukung, performa di Windows/Linux 4GB RAM, kompatibilitas embedded Vite assets.
- Aturan CSV Adobe terkini (header dan limit), kebijakan upload gambar/vector/video per jenis konten, dan akses SFTP pada akun kontributor.
- Endpoint/model/response AIStudio2API yang benar-benar tersedia saat runtime, legalitas/izin provider, serta error ketika service mati.
- Lisensi, dependencies, dan perilaku dari repo upstream pada *commit pin* yang akan diaudit.

## Pembeda produk

- Provenance AI/manual secara eksplisit; policy gating untuk marketplace.
- Draft metadata dapat diperiksa pengguna, bukan langsung ditulis ke file.
- Job terpersistensi yang bisa di-retry dan dilihat hasilnya.
- Per-asset status dan per-marketplace status terpisah.
- Rule packs versioned agar kebijakan marketplace bisa diperbarui.
- Local-first, cloud-ready via interface, tanpa cloud complexity dini.

## Kesalahan yang tidak boleh diulang

Jangan mengeksekusi sebelum dokumen dan acceptance gates jelas; jangan menyatakan upload berhasil atau approval tanpa bukti; jangan mencampur milestone V2 ke MVP; jangan menganggap README pihak ketiga sebagai bukti audit keamanan/keberhasilan di environment user; jangan mengunci ke endpoint/model AI yang mungkin berubah.
