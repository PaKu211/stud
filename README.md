# Microstock Automation Studio — Planning Pack v1.0

**Tanggal:** 2026-11-08  | **Status:** Ready for AI-agent implementation planning, not a claim that the app exists.  
**Peran penyusun:** Perencana produk dan arsitektur. **Peran AI Studio agent:** Implementasi, pengujian, dokumentasi hasil.

## Mulai di sini

Paket ini adalah hasil perencanaan ulang dari transkrip percakapan yang diberikan. **Bukan source code aplikasi**, dan tidak mengklaim bahwa repository referensi sudah diaudit/diuji secara runtime. Tujuan: memungkinkan coding agent membangun MVP secara bertahap, lokal lebih dulu, lalu cloud tanpa menulis ulang logika bisnis.

**Urutan baca wajib untuk agent**
1. `HANDOFF_PROMPT.md` — mandat, larangan, urutan kerja, definisi selesai.
2. `docs/00_CONVERSATION_ANALYSIS.md` — keputusan final dan perubahan arah.
3. `system.md` — visi, batasan, invariant sistem.
4. `docs/PRD.md` — requirement produk dan acceptance criteria.
5. `docs/TRD.md` — ketentuan teknis dan nonfungsional.
6. `docs/AFD.md` — application/user flows.
7. `docs/DBD.md` — database schema dan migration contract.
8. `docs/BSD.md` — backend service design.
9. `specs/API_CONTRACT.md` — kontrak endpoint & errors.
10. `docs/SECURITY_COMPLIANCE.md`, `docs/TEST_PLAN.md`, `docs/IMPLEMENTATION_PLAN.md`.
11. `docs/REPOSITORY_RESEARCH.md`, `docs/ADR.md` — referensi dan alasan desain.

## Produk dalam satu kalimat

Aplikasi web lokal untuk mengelola aset stock dari impor, review, metadata (manual/AI opsional), quality check, CSV Adobe Stock dan paket siap submit; modul generasi, upload, dan multi-marketplace ditambahkan bertahap.

## Keputusan teknologi final

- **Versi MVP:** Go HTTP server + Svelte/TypeScript (Vite, static production build disajikan oleh Go) + SQLite + local filesystem.
- **Pemakaian:** buka browser ke `http://127.0.0.1:8080`; UI development bisa memakai Vite dev server.
- **MVP tidak memerlukan:** Docker, Tauri/Electron, login cloud, pembayaran API AI, ExifTool, FFmpeg, marketplace credentials.
- **Ekstensi opsional:** AIStudio2API via localhost HTTP, provider OpenAI-compatible resmi, ExifTool, FFmpeg, SFTP Adobe bagi akun yang berhak.
- **Deploy nanti:** backend Go dipindahkan ke container/service; storage dan DB melalui ports/adapters. Cloud deployment bukan cakupan MVP.

## Batas fase

- **MVP-A (wajib):** import folder gambar, indeks/preview, hash deduplikasi, metadata editor, review & validasi Adobe, ekspor CSV valid, laporan job/status, tes otomatis.
- **MVP-B (setelah MVP-A lolos):** AI vision metadata melalui gateway opsional; QC gambar yang lebih luas; metadata embedding via ExifTool opsional.
- **MVP-C:** SFTP upload opsional untuk akun Adobe yang memenuhi syarat, *manual final submission*; foto/video QC lanjutan; adapter lain.
- **V2:** generator prompt/asset, video processing, workflow visual, agent/MCP, cloud Appwrite/Render jika diperlukan.

**Tidak boleh:** mengklaim status `submitted` hanya karena upload berhasil; memasukkan aset AI ke Shutterstock; menghapus atau mengubah source media tanpa persetujuan eksplisit; mengakali CAPTCHA / kontrol anti-bot.

## Dokumen inti

| File | Peran |
|---|---|
| `system.md` | System charter, scope, invariants |
| `project.yaml` | Project manifest machine-readable |
| `docs/00_CONVERSATION_ANALYSIS.md` | Riwayat keputusan dan prioritas |
| `docs/REPOSITORY_RESEARCH.md` | Referensi repository & lisensi |
| `docs/PRD.md` | Product requirements |
| `docs/TRD.md` | Technical requirements |
| `docs/AFD.md` | Application flow design |
| `docs/DBD.md` | Database design |
| `docs/BSD.md` | Backend service design |
| `specs/API_CONTRACT.md` | API requests/responses |
| `docs/SECURITY_COMPLIANCE.md` | Security + platform policy |
| `docs/TEST_PLAN.md` | Test matrix, fixtures, criteria |
| `docs/ADR.md` | Architecture decision records |
| `docs/IMPLEMENTATION_PLAN.md` | Milestones dan task breakdown |
| `HANDOFF_PROMPT.md` | Copy-paste prompt untuk coding agent |

## Definisi selesai MVP-A

Setelah mengimpor folder 20 JPEG sampel, pengguna bisa meninjau thumbnail, mengisi metadata, melihat error yang jelas, mengekspor CSV Adobe yang dapat diperiksa, dan memulihkan status setelah aplikasi di-restart. Seluruh tes relevan harus berjalan di Windows/Linux atau didokumentasikan bila lingkungan CI tidak tersedia. Lihat detail di `docs/TEST_PLAN.md`.

## Sumber

Setiap tautan dan status lisensi yang relevan disimpan di `docs/REPOSITORY_RESEARCH.md` dan `docs/SECURITY_COMPLIANCE.md`; sumber diverifikasi di web pada 2026-11-08. Kebijakan marketplace harus diverifikasi ulang sebelum mengaktifkan upload nyata.
