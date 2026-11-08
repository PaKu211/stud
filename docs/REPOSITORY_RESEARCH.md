# Repository Research & Reverse-Engineering Plan

Verifikasi publik: 2026-11-08. Fitur disarikan dari README/dokumentasi repositori, **bukan** hasil clone, code audit menyeluruh atau tes runtime pada laptop user. Sebelum memakai ulang kode: inspect license file, dependency licenses, latest commit, security posture, data flow, and tests. Simpan commit SHA di `docs/upstream-lock.json` saat agent mengaudit. Jangan mengambil kode dari repo tanpa hak pakai jelas.

## Referensi wajib

| Repo & URL | Peran | Kenapa dipilih | Strategi |
|---|---|---|---|
| **ToTag** — https://github.com/kitleong1981/totag | Asset library, metadata IPTC/XMP, AI draft, CSV export, FTP/SFTP, jobs | Mengandung user flow lokal yang paling dekat; README mengakui masih rough dan pengujian terutama macOS | **Primary behavior/reference**; port concept/interface ke Go, uji perilaku dan input, jangan menyalin Flask app besar mentah |
| **Stockflow** — https://github.com/aRealGem/stockflow | Video QC, ffprobe/transcode, manifest, CSV per site, transfer reporting | Menyediakan pemisahan QC–transform–export–upload dan test cases | **Pipeline/QC reference**; pakai test fixture dan manifest pattern; video processing fase lanjut |
| **RJ AIO Metadata** — https://github.com/riiicil/rj-aio-metadata | Marketplace metadata validation, UI adapters, taxonomy/sanitization | Contoh adapter multi-platform; DOM integration rapuh jika situs berubah | **Rules/adapter reference**; jangan masukkan browser DOM automation ke core MVP |
| **AIStudio2API** — https://github.com/Mag1cFall/AIStudio2API | Gateway text/vision/image/video, OpenAI-compatible endpoints | Kebutuhan integrasi khusus user, Go service sendiri | **Optional external service**, bukan copy ke backend dan bukan required dependency |

Keempat repo menampilkan **MIT** di halaman GitHub saat dicek, tetapi lisensi transitive dependencies tetap wajib diperiksa. ToTag terbuka tentang belum adanya automated tests dan dukungan macOS yang lebih matang; Stockflow memiliki tests yang dijabarkan; AIStudio2API menggunakan mekanisme web-browser/AI Studio tidak resmi sehingga login, ToS dan kuota harus dipertimbangkan.

## Referensi sekunder

- **Adobe Stock Generator:** https://github.com/UeberUeber/adobe-stock-generator — konsep prompt → image processing → review → CSV. Tidak ada keputusan untuk mengopi kode; audit license lebih dulu.
- **Xpiks:** https://github.com/andsontan/xpiks — ide UX, taxonomy, keywording; GPL-3.0 membatasi pilihan reuse kode sehingga **jangan copy kode** ke codebase tanpa rencana kepatuhan GPL.
- **ExifTool:** https://exiftool.org/ — opsional read/write metadata, sebagai executable external (periksa paket/lisensi distribusi).
- **FFmpeg:** https://ffmpeg.org/ — media probe/processing opsional. Audit per-build license, termasuk codec.

## Task audit per repo

**A. ToTag**
1. Clone commit pinned; inventaris file/entrypoints/third-party deps.
2. Jalankan lokal pada environment aman bila feasible; catat OS path assumptions dan apa yang rusak di Windows/Linux.
3. Petakan schema metadata (title, description, keywords, XMP/IPTC), filesystem I/O, CSV columns, credential handling, queue semantics.
4. Tulis golden test untuk 5 image fixture (orientation, unicode, duplicate basename, long title, missing metadata).
5. Laporkan: feature map → Go package → test → risk, jangan langsung port seluruh app.

**B. Stockflow**
1. Petakan ffprobe fields, QC specs version, per-site CSV writers, manifest, transfer errors.
2. Tulis sample manifest dan QC test yang dapat diverifikasi manual.
3. Pastikan operasi file tidak memindahkan originals tanpa consent (upstream melakukan move pada sejumlah stage).
4. Video pipeline hanya setelah image MVP stabil.

**C. RJ AIO**
1. Identifikasi rules murni (categories/keywords/AI labels) vs selector DOM.
2. Pisahkan interface `MarketplaceProfile` (pure deterministic validator/exporter) dari `PortalAutomation` (out of scope MVP).
3. Tetapkan aturan source-of-truth dari dokumen resmi, bukan angka lama di extension.
4. Jangan auto-submit melalui browser tanpa pengecekan kepatuhan, keandalan, dan persetujuan user.

**D. AIStudio2API**
1. Jangan hardcode model list; panggil `/v1/models` saat service ada.
2. Uji `/v1/chat/completions` dengan vision support; output JSON strict dan retry terbatas.
3. `/v1/images/generations`, `/v1/images/edits`, `/v1/videos` ditunda sampai fase generate media.
4. Tes service unavailable, 401, 429, 5xx, timeout, malformed output; tampilkan graceful degradation.
5. Tinjau bagaimana autentikasi lewat browser dan penyimpanan state bekerja; jangan menyimpan/menyalin cookie Google ke aplikasi.

## Output audit agent (wajib sebelum implementasi reuse)

`docs/upstream-lock.json` (repo, commit SHA, URL, license); `docs/UPSTREAM_AUDIT.md` (feature matrix, architecture notes, copied-code attribution, dependencies); minimal 1 test fixture per reused behavior.

## Rujukan platform resmi

- Adobe Stock API FAQ, tidak ada Contributor API publik: https://developer.adobe.com/stock/docs/faq/
- Adobe CSV requirements (11 Juni 2026): https://helpx.adobe.com/stock/contributor/manage-your-portfolio/csv-requirements-content.html
- Adobe CSV creation: https://helpx.adobe.com/stock/contributor/manage-your-portfolio/create-csv-file.html
- Adobe CSV upload via portal: https://helpx.adobe.com/stock/contributor/manage-your-portfolio/upload-csv-file.html
- Adobe upload eligibility/SFTP: https://helpx.adobe.com/in/stock/contributor/help/uploading-content.html
- Adobe content upload guidelines (2 Sep 2026): https://helpx.adobe.com/id_id/stock/contributor/content-policies-guidelines/content-policies/content-upload-guidelines.html
- Adobe AI labeling: https://helpx.adobe.com/in/stock/contributor/submit-your-content/submit-generative-ai-content/submit-generative-ai-content.html
- Adobe near-duplicate policy: https://helpx.adobe.com/id_id/stock/contributor/submit-your-content/submit-generative-ai-content/distinct-generative-ai-submission-best-practices.html
- Shutterstock AI contribution restriction: https://submit.shutterstock.com/help/id/articles/10594633-mengapa-konten-saya-ditolak-sebagai-konten-yang-tidak-dapat-dilisensikan

**Ketidakpastian:** login eligibility, exact release behavior, dan format CSV tambahan harus divalidasi dengan akun nyata/test fixture sebelum mengaktifkan transfer. Tidak ada klaim bahwa layanan memberi akses resmi ke submission otomatis.
