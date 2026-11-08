# PRD — Product Requirements Document

**Product:** Microstock Automation Studio  | **Version:** 1.0  | **Target:** MVP-A local-first

## 1. Problem statement

Kontributor microstock mengulang pengelolaan folder, cek kualitas, penulisan metadata, normalisasi keyword, pembuatan CSV per platform, dan pelacakan proses upload. Alat terpisah menciptakan duplikasi kerja dan status sulit dilacak. Produk menggabungkan pekerjaan *sebelum submission* ke sebuah workstation berbasis browser yang berjalan offline untuk fungsi inti dan tetap aman ketika AI belum tersedia.

## 2. Outcome & metric

- O1: Dalam satu sesi, user menyiapkan **20 JPEG** menjadi batch Adobe CSV tanpa menulis file CSV secara manual.
- O2: **100%** aset yang gagal validasi menyebut rule ID, field, alasan, dan usulan perbaikan.
- O3: **0** file source berubah tanpa operasi eksplisit pengguna; hash aslinya sama sebelum/sesudah MVP flow.
- O4: Aplikasi berfungsi ketika provider AI/SFTP tidak tersedia.
- O5: Project, metadata, job state, export manifest tetap ada setelah restart.
- O6: CSV yang dibuat lolos parser CSV standar dan rule set Adobe yang diverifikasi.

Angka ini target engineering test, bukan klaim peningkatan penjualan/approval.

## 3. Target persona dan skenario

- Kontributor individual yang memiliki 20–200 gambar di laptop dan ingin membuat metadata/CSV massal.
- Pembuat konten AI yang perlu deklarasi provenance dan validasi marketplace.
- Pengguna tanpa AI endpoint yang mengerjakan metadata manual.

## 4. Scope matrix

| Code | Feature | MVP-A | MVP-B | MVP-C/V2 |
|---|---|---|---|---|
| P-01 | Project create/list/open | Required | | |
| P-02 | Add allowed folder root, scan batch | Required | | |
| P-03 | Thumbnail, basic image properties, hash duplicate detection | Required | | |
| P-04 | Manual metadata & ordered keywords, bulk operations | Required | | |
| P-05 | Provenance and rights/release flags | Required | | |
| P-06 | Adobe rule validation, actionable errors | Required | | |
| P-07 | Adobe CSV export, export manifest, previews | Required | | |
| P-08 | Persistent jobs, status & audit log | Required | | |
| P-09 | AI vision metadata draft + opt-in send | | Required | |
| P-10 | Image QC advanced / ExifTool embedding | | Required | |
| P-11 | Adobe SFTP upload if eligible | | | MVP-C |
| P-12 | Video inspection/transcode | | | MVP-C |
| P-13 | Additional marketplace adapters | | | MVP-C/V2 |
| P-14 | AI image/video generation, prompt library | | | V2 |
| P-15 | Workflow canvas, agent/MCP, cloud auth/storage | | | V2 |

## 5. User stories and acceptance

### US-01: Create project
**As** kontributor **I want** membuat project yang menyimpan dataset aset terpisah.  
**Given** aplikasi baru, **when** user memberi nama project valid, **then** project muncul pada daftar; rename tidak mengubah ID; nama kosong ditolak dengan pesan jelas. `P-01`

### US-02: Add root and scan
**As** kontributor **I want** memilih root lokal sehingga file terindeks.  
**Given** path folder valid dan user memasukkannya, **when** scan dimulai, **then** hanya file berjenis diizinkan yang tercatat, progress terlihat, file source tetap utuh; path nonexistent atau bukan folder mendapat 4xx `INVALID_ROOT`. Scan berulang tidak menduplikasi file. `P-02`

### US-03: Browse and duplicate check
**Given** dua source memiliki SHA-256 identik, **when** scan selesai, **then** duplicate diberi grup yang sama; tile memuat filename, dimensions, preview, review status; kegagalan preview tidak menggagalkan indeks keseluruhan. `P-03`

### US-04: Edit metadata
**Given** aset terindeks, **when** judul, keyword berurutan dan kategori disimpan, **then** revisi bertambah, isi persisten, dan mengubah metadata yang sebelumnya reviewed mengembalikannya ke `needs_review`. Bulk edit memperlihatkan jumlah target serta preview. `P-04`

### US-05: Provenance, rights, releases
**Given** aset baru, **then** `source_kind=unknown` dan `rights_status=unverified` default; tidak boleh siap ekspor sebelum user mengklasifikasikan dan menyatakan rights/release relevan. `ai_generated` selalu tercatat untuk policy validation. `P-05`

### US-06: Validate Adobe
**Given** title lebih dari batas, filename panjang, keyword duplikat, unresolved rights, non-JPEG, atau AI deklarasi ambigu, **when** Validate diklik, **then** layar menampilkan semua error/warning dengan kode rule dan perbaikan yang disarankan. Fail items tidak masuk export yang diklaim *ready*. `P-06`

### US-07: Export CSV
**Given** set aset `reviewed` dan lolos validator Adobe, **when** user mengeklik Export, **then** sistem membuat file `.csv` UTF-8, header dan kolom sesuai profil, row sesuai filename, kategori dan keywords terurut; jika >5.000 baris atau >1MB maka dibagi menjadi part yang memenuhi batas dan disimpan manifest. Jika 0 valid assets → error `NO_EXPORTABLE_ASSETS`. `P-07`

### US-08: Review before external action
**Given** user memilih fitur optional external AI/SFTP, **when** data akan keluar dari device, **then** UI menyebut provider/destination, jenis data dan jumlah aset; tindakan perlu explicit confirmation; tanpa konfirmasi operasi tidak dipanggil. `P-09/P-11`

### US-09: AI metadata draft (MVP-B)
**Given** gateway mendukung vision, **when** user mengirim aset untuk draft, **then** usulan muncul sebagai `ai_draft`, pengguna boleh apply/edit/reject; tidak langsung mengganti metadata reviewed. Jika service mati atau timeout → metadata manual tetap utuh. `P-09`

### US-10: Distribution status (MVP-C)
**Given** SFTP transfer selesai, **then** per-platform status `transferred` dengan receipt; portal submission tetap `not_verified`. Pengguna dapat menandai `submitted`/`approved` secara manual dengan bukti dan timestamp, bukan auto-asumsi. `P-11`

## 6. UX pages

`Dashboard / Projects`, `Project Assets` (grid, filters), `Asset Detail` (Preview / Metadata / QC / Provenance), `Batch Review`, `Validate`, `Export History`, `Job Center`, `Settings / Integrations`. Fase lanjut: `Distribution Center`, `AI Studio`, `Workflow Builder`.

**UX policies:** bahasa UI utama Indonesia; technical terms familiar boleh Inggris. Status/gagal tidak hanya diwakili warna. Tampilkan progress & detail action; provide keyboard navigation, consistent loading and empty states. Tidak auto-delete atau auto-send.

## 7. Functional constraints

- Input accepted for MVP: `.jpg/.jpeg`, `.png` untuk indeks dan review; per-marketplace output Adobe hanya JPEG dan jenis lain yang diverifikasi profil aktif. Video input dicatat nanti setelah probe/video QC siap.
- Timestamps UTC, tampil timezone user.
- Source filesystem path tidak diberikan sebagai absolute URLs ke frontend; seluruh akses melewati opaque `asset_id`.
- User dapat meninjau data/manifest tanpa mengakses akun Adobe.
- CSV generation bukan pengiriman otomatis; label tombol harus **Ekspor CSV**, bukan **Submit**.

## 8. Edge cases

1. Folder hilang/removable disk tidak tersedia → asset status `missing`, metadata tidak hilang.
2. Sama basename berbeda folder → export conflict; minta rename derivative atau reject, jangan salah mencocokkan file.
3. Filename Unicode/case sensitivity → CSV punya exact filename yang sesuai target; Unicode NFC dan aturan nama diverifikasi; jangan ubah sumber diam-diam.
4. Image corrupt atau dimensi tidak terbaca → simpan error per item, scan terus.
5. Stop/restart saat scan → last scanned files persist; retry idempotent.
6. Keyword memakai koma/quote → escape CSV standar dan keyword cell benar.
7. Image AI memilih Shutterstock → blocked, terlepas dari kegagalan/berhasilnya koneksi.
8. Tidak ada internet → fitur offline tetap bekerja.

## 9. Out of scope & dependencies

Out of scope MVP: produk SaaS multi-user, analytics penjualan live, portal auto-submit, malware scanning enterprise, AI generator, bulk video, scraper marketplace, mobile filesystem access. Optional dependencies ExifTool/FFmpeg baru digunakan ketika terdeteksi; absennya alat tidak membuat MVP gagal launch.

## 10. Release gate

MVP-A accepted jika: semua US-01 hingga US-07 lulus test manual & otomatis, source immutable dibuktikan hash, CSV golden fixture cocok, resume/restart test lulus, dan tidak ada secret di build/log. Optional tasks tidak boleh dianggap blocker milestone pertama.
