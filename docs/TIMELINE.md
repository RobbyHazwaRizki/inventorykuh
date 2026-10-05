# TIMELINE: Smart Inventory & Demand Forecast

| | |
|---|---|
| **Terakhir diperbarui** | 1 Oktober 2026 |
| **Mengacu ke** | [`PRD.md`](./PRD.md) v1.1 |
| **Pemilik** | Robby Hazwarizki ("Boss") |

> **Cara memakai:** dokumen ini dicatat progresnya setiap kali Boss meminta *"update timeline"*. Ubah ikon status di kolom **Status** sesuai kenyataan, lalu catat di [Log Progres](#log-progres). Jika terkena limit atau pindah chat, pakai [Prompt Lanjutan](#prompt-lanjutan-setelah-limit).

**Legenda status:** ✅ selesai · 🔄 sedang dikerjakan · ⬜ belum · ⏸️ tertunda / terblokir

**Estimasi** mengasumsikan sekitar 15 sampai 20 jam per minggu. Geser sesuai kenyataan; urutan fase lebih penting daripada angka minggu.

---

## 📍 Posisi Saat Ini

> **Fase 0 selesai. Berikutnya: Fase 1, tugas T1.1.**

| Cakupan | Selesai | Total | Progres |
|---------|:-------:|:-----:|:-------:|
| MVP (Fase 1 sampai 9) | 0 | 63 | 0% |
| V1 (Fase 10 sampai 12, *provisional*) | 0 | 17 | 0% |

**Sedang dikerjakan:** belum ada
**Blocker / keputusan terbuka:** lihat [PRD bagian 14](./PRD.md#14-pertanyaan-terbuka--asumsi) (Q1 sampai Q6, semuanya punya nilai default dan tidak menghambat Fase 1)

---

## Ringkasan Fase

| Fase | Nama | Estimasi | Hasil yang bisa didemokan | Yang dipelajari | Status |
|------|------|----------|---------------------------|-----------------|:------:|
| 0 | Discovery & PRD | Selesai | PRD v1.1 dan timeline | Merancang produk dan arsitektur | ✅ |
| 1 | Fondasi & keamanan dasar | Minggu 1 | Repo, Docker, CI hijau, login jalan | Laravel dasar, Sanctum, CI | ⬜ |
| 2 | Core inventory API | Minggu 2–3 | Produk dan ledger stok via API | Eloquent, migrasi, ledger, testing | ⬜ |
| 3 | Frontend online | Minggu 3–4 | UI login, produk, stok | Vue 3, TypeScript, Pinia | ⬜ |
| 4 | Penjualan & reservasi | Minggu 5 | Soft lock + test concurrency | Transaksi DB, row lock, race condition | ⬜ |
| 5 | Dashboard & chart | Minggu 6 | Chart Owner/Manager | Query agregasi, library chart | ⬜ |
| 6 | Local-first & sync | Minggu 7–8 | Jalan offline lalu sinkron | IndexedDB (Dexie), outbox, idempotensi | ⬜ |
| 7 | Real-time | Minggu 8–9 | Perubahan muncul di semua perangkat | WebSocket, Reverb, Echo | ⬜ |
| 8 | Hardening & QA | Minggu 9–10 | Checklist OWASP lolos | Keamanan aplikasi, pengujian | ⬜ |
| 9 | Packaging, docs, **rilis v1.0 (MVP)** | Minggu 10–11 | PWA, APK, README, demo live | PWA, Capacitor, CI/CD, GitHub Releases | ⬜ |
| 10 | V1: approval, opname, supplier, viewer | Minggu 11–12 | Alur persetujuan | Workflow dan state machine | ⬜ |
| 11 | V1: forecast | Minggu 12–13 | Prediksi 30 hari + akurasi | Time series sederhana, scheduler | ⬜ |
| 12 | V1: ekspor, backup, enkripsi lokal, **rilis v1.1** | Minggu 13–14 | Rilis v1.1 | Enkripsi, backup/restore | ⬜ |

**Total estimasi:** sekitar 13 sampai 14 minggu.

---

## Rincian Tugas

### Fase 1: Fondasi & Keamanan Dasar

| ID | Tugas | Status |
|----|-------|:------:|
| T1.1 | Buat repo monorepo, struktur folder, `.gitignore`, `.env.example`, lisensi MIT; taruh `PRD.md` dan `TIMELINE.md` di `docs/` | ⬜ |
| T1.2 | `docker-compose.yml` (Laravel, PostgreSQL, Redis) berjalan | ⬜ |
| T1.3 | Skeleton Laravel API dengan struktur berlapis (Controller/Service/Repository/Policy) | ⬜ |
| T1.4 | Pint, Larastan, Pest terpasang | ⬜ |
| T1.5 | GitHub Actions CI (lint, analisis statis, test, `composer audit`, gitleaks) | ⬜ |
| T1.6 | Auth Sanctum: login, logout, `/me`, token per perangkat | ⬜ |
| T1.7 | Multi-tenant dasar (`business_id`, global scope) | ⬜ |
| T1.8 | Role dan permission + Policy | ⬜ |
| T1.9 | Audit log dasar | ⬜ |
| T1.10 | Rate limiting dan header keamanan | ⬜ |

**Selesai bila:** `docker compose up` menjalankan API, CI hijau, login dengan token berfungsi, dan test isolasi antar bisnis lulus.

### Fase 2: Core Inventory API

| ID | Tugas | Status |
|----|-------|:------:|
| T2.1 | Migrasi + CRUD kategori dan produk (soft delete) | ⬜ |
| T2.2 | Tabel ledger `stock_movements` (append-only, UUID dari klien) | ⬜ |
| T2.3 | Perhitungan stok saat ini + endpoint mutasi (batch, idempoten) | ⬜ |
| T2.4 | Reorder point dan alert stok menipis | ⬜ |
| T2.5 | Seeder data dummy yang realistis | ⬜ |
| T2.6 | Test: isolasi tenant, izin per role, idempotensi | ⬜ |

**Selesai bila:** mutasi yang dikirim dua kali tidak menggandakan stok, dan semua izin per role teruji.

### Fase 3: Frontend Online

| ID | Tugas | Status |
|----|-------|:------:|
| T3.1 | Setup SPA (Vite, Vue 3, TypeScript, router, Pinia), layer service API | ⬜ |
| T3.2 | Halaman login + penyimpanan token | ⬜ |
| T3.3 | Halaman produk (daftar, tambah, ubah) | ⬜ |
| T3.4 | Halaman stok: catat masuk/keluar, riwayat mutasi | ⬜ |
| T3.5 | UI sadar-role (menu dan tombol sesuai izin dari server) | ⬜ |
| T3.6 | Layout responsif untuk HP, tablet, dan desktop | ⬜ |

**Selesai bila:** staf bisa login, melihat produk, dan mencatat mutasi lewat UI, dengan tampilan sesuai role.

### Fase 4: Penjualan & Reservasi

| ID | Tugas | Status |
|----|-------|:------:|
| T4.1 | Tabel dan endpoint `reservations` (transaksi + row lock) | ⬜ |
| T4.2 | TTL, heartbeat, dan job pelepas tahanan kedaluwarsa | ⬜ |
| T4.3 | Penjualan dari reservasi (konfirmasi menjadi mutasi `sale`) | ⬜ |
| T4.4 | **Test concurrency** (50 permintaan paralel) | ⬜ |
| T4.5 | UI kasir: pilih barang, tahan, konfirmasi, status "tidak tersedia" | ⬜ |
| T4.6 | Unique constraint untuk barang unik | ⬜ |

**Selesai bila:** test concurrency membuktikan 0 overselling dan kasir bisa menyelesaikan penjualan lewat UI.

### Fase 5: Dashboard & Chart

| ID | Tugas | Status |
|----|-------|:------:|
| T5.1 | Endpoint agregasi (masuk/keluar, pemasukan, terlaris, komposisi) | ⬜ |
| T5.2 | Pembatasan akses laporan per role (nilai rupiah, margin) | ⬜ |
| T5.3 | UI chart: barang masuk vs keluar (harian/mingguan/bulanan) | ⬜ |
| T5.4 | UI chart: pemasukan, produk terlaris, donut komposisi | ⬜ |
| T5.5 | Filter tanggal/produk/kategori + panel stok kritis | ⬜ |
| T5.6 | Cache laporan di perangkat + label "terakhir diperbarui" | ⬜ |
| T5.7 | Test: total chart sama dengan jumlah ledger | ⬜ |

**Selesai bila:** Owner dan Manager melihat semua chart, role lain dibatasi sesuai matriks, dan angka chart terbukti cocok dengan ledger.

### Fase 6: Local-First & Sync

| ID | Tugas | Status |
|----|-------|:------:|
| T6.1 | DB lokal (IndexedDB/Dexie) + skema pencerminan data | ⬜ |
| T6.2 | Semua tulis ke lokal dulu, lalu masuk outbox | ⬜ |
| T6.3 | `POST /sync/push` (batch, idempoten, validasi server) | ⬜ |
| T6.4 | `GET /sync/pull?since=seq` + tabel `sync_changes` | ⬜ |
| T6.5 | Penanganan konflik (LWW waktu server, soft delete) | ⬜ |
| T6.6 | Status `pending_review` dan deteksi anomali stok negatif | ⬜ |
| T6.7 | Indikator koneksi/sinkronisasi di UI | ⬜ |
| T6.8 | Penolakan sinkron untuk perangkat/role yang dicabut | ⬜ |
| T6.9 | Test skenario offline → online, kirim ulang, dua perangkat | ⬜ |

**Selesai bila:** dua perangkat yang bekerja offline lalu tersambung menghasilkan data yang sama persis.

### Fase 7: Real-Time

| ID | Tugas | Status |
|----|-------|:------:|
| T7.1 | Laravel Reverb + Broadcasting + Echo di klien | ⬜ |
| T7.2 | Channel privat per bisnis + otorisasi | ⬜ |
| T7.3 | Siaran event mutasi, produk, reservasi, anomali | ⬜ |
| T7.4 | UI memperbarui otomatis (status "merah" real-time) | ⬜ |
| T7.5 | Fallback pull berkala saat WebSocket putus | ⬜ |
| T7.6 | Ukur latensi sync (target < 2 detik) | ⬜ |

**Selesai bila:** perubahan di satu perangkat muncul di perangkat lain dalam waktu < 2 detik.

### Fase 8: Hardening & QA

| ID | Tugas | Status |
|----|-------|:------:|
| T8.1 | Audit terhadap checklist OWASP Top 10 (dokumentasikan hasilnya) | ⬜ |
| T8.2 | PIN lock aplikasi + penyimpanan token aman | ⬜ |
| T8.3 | Uji dasar IDOR, mass assignment, dan rate limit | ⬜ |
| T8.4 | Perbaikan bug + cakupan test ≥ 70% layer service | ⬜ |
| T8.5 | Uji coba dengan 3 sampai 5 pengguna nyata (opsional) | ⬜ |

**Selesai bila:** checklist OWASP terdokumentasi, cakupan test tercapai, tidak ada bug kritis terbuka.

### Fase 9: Packaging, Dokumentasi, Rilis v1.0 (MVP)

| ID | Tugas | Status |
|----|-------|:------:|
| T9.1 | PWA (manifest, service worker, ikon, mode offline) | ⬜ |
| T9.2 | Capacitor: build APK bertanda tangan | ⬜ |
| T9.3 | Workflow `release.yml` + checksum SHA-256 | ⬜ |
| T9.4 | Putuskan hosting dan deploy demo + akun demo | ⬜ |
| T9.5 | README (masalah, solusi, arsitektur, install, screenshot/GIF) | ⬜ |
| T9.6 | Diagram arsitektur, ERD, dan ADR keputusan penting | ⬜ |
| T9.7 | Case study (blog/LinkedIn) | ⬜ |
| T9.8 | **Rilis v1.0.0** di GitHub Releases | ⬜ |

**Selesai bila:** APK dan PWA bisa dipasang pengguna dari halaman Releases/demo, README lengkap, dan v1.0.0 terpublikasi.

### Fase 10: V1, Approval, Opname, Supplier, Viewer *(provisional)*

| ID | Tugas | Status |
|----|-------|:------:|
| T10.1 | Tabel dan alur `stock_adjustments` (ajukan, setujui, tolak) | ⬜ |
| T10.2 | UI stok opname dan layar persetujuan | ⬜ |
| T10.3 | Modul supplier + riwayat pembelian | ⬜ |
| T10.4 | Retur penjualan | ⬜ |
| T10.5 | Role Viewer/Auditor | ⬜ |
| T10.6 | Layar penyelesaian anomali/konflik | ⬜ |

### Fase 11: V1, Forecast di Laravel *(provisional)*

| ID | Tugas | Status |
|----|-------|:------:|
| T11.1 | Tabel `forecasts` + service moving average / exponential smoothing | ⬜ |
| T11.2 | Scheduler perhitungan harian | ⬜ |
| T11.3 | Backtest dan metrik akurasi (MAE/MAPE) | ⬜ |
| T11.4 | Endpoint `GET /forecasts` + penanganan data belum cukup | ⬜ |
| T11.5 | UI dashboard forecast dan rekomendasi pesan ulang | ⬜ |
| T11.6 | Test perhitungan forecast | ⬜ |

### Fase 12: V1, Ekspor, Backup, Enkripsi Lokal, Rilis v1.1 *(provisional)*

| ID | Tugas | Status |
|----|-------|:------:|
| T12.1 | Ekspor CSV/Excel dan ekspor chart | ⬜ |
| T12.2 | Backup/restore data | ⬜ |
| T12.3 | Enkripsi data lokal | ⬜ |
| T12.4 | Riwayat login + undang staf via kode/link + PIN lokal | ⬜ |
| T12.5 | **Rilis v1.1.0** | ⬜ |

### Backlog "Later" (tidak dijadwalkan)

Installer desktop (Tauri), layanan forecast FastAPI, multi-lokasi dan transfer, batch dan kedaluwarsa, push/WhatsApp, varian dan foto produk.

---

## Definition of Done (berlaku untuk setiap tugas)

- Kode ter-merge lewat pull request ke `develop`.
- Test terkait lulus dan CI hijau.
- Tidak ada secret di commit.
- Dokumentasi atau ADR diperbarui bila ada keputusan baru.
- Hasil bisa dijelaskan dengan bahasa sendiri (tujuan belajar proyek ini).

---

## Cara Meminta Update Timeline

Setiap kali Boss meminta **"update timeline"**, jawaban akan memuat:

| Bagian | Isi |
|--------|-----|
| 📍 Posisi saat ini | Fase dan tugas aktif |
| ✅ Selesai | ID tugas yang rampung |
| 🔄 Sedang dikerjakan | Tugas yang berjalan beserta sisanya |
| ⬜ Belum dikerjakan | Sisa tugas per fase |
| ➡️ Langkah berikutnya | 3 tugas teratas yang sebaiknya dikerjakan |
| ⏸️ Blocker / keputusan terbuka | Hal yang perlu diputuskan |
| 📊 Progres | Jumlah selesai/total dan estimasi yang disesuaikan |

**Yang perlu Boss lakukan:** lapor progres singkat, misalnya *"T1.1 sampai T1.5 selesai, T1.6 masih jalan, kena masalah di Sanctum"*. Jika ada perubahan keputusan atau scope, sebutkan juga supaya PRD ikut dinaikkan versinya.

---

## Prompt Lanjutan Setelah Limit

Salin blok ini ke awal chat baru, lalu isi bagian `[...]`:

```
KONTEKS PROYEK (lanjutkan dari sini)
Aku Robby ("Boss"). Kita membangun "Smart Inventory & Demand Forecast",
project portofolio untuk lulusan S1 Sistem Informasi. PRD v1.1 dan TIMELINE
sudah dibuat (aku akan menempelkan atau mengunggah isinya jika perlu).

Keputusan final:
- Local-first + sinkronisasi real-time; satu bisnis banyak staf.
- Stok = ledger mutasi append-only; reservasi stok ber-TTL (soft lock ala kursi bioskop).
- Backend: Laravel API-first (Sanctum, Reverb, Policy), PostgreSQL, Redis.
- Frontend: SPA Vue 3 + TypeScript + Vite, DB lokal IndexedDB (Dexie), outbox + cursor sync.
- Distribusi: PWA dan APK (Capacitor) lewat GitHub Releases. Tauri desktop = Later.
- Forecast dibuat di Laravel dulu (moving average / exponential smoothing); FastAPI = stretch.
- Role: Owner, Manager, Warehouse Staff, Cashier (+ Viewer di V1), izin berbasis permission.
- Owner dan Manager melihat chart barang masuk/keluar, pemasukan, terlaris, komposisi.
- Prioritas: struktur rapi, standar kode, keamanan OWASP, mudah dipelajari.
- Hosting demo diputuskan di Fase 9; awalnya Docker lokal.

STATUS TERAKHIR:
- Fase aktif: [Fase X]
- Tugas selesai: [ID...]
- Sedang dikerjakan: [ID...]
- Kendala: [...]

Tolong lanjutkan dari tugas berikutnya. Setiap aku minta "update timeline",
tampilkan: posisi saat ini, selesai, sedang dikerjakan, belum, langkah berikutnya,
blocker, dan progres. Gunakan Bahasa Indonesia dan panggil aku "Boss".
```

---

## Log Progres

| Tanggal | Perubahan | Catatan |
|---------|-----------|---------|
| 1 Okt 2026 | PRD v1.0 disusun | Fase 0 (diskusi dan perancangan) selesai |
| 1 Okt 2026 | PRD v1.1 dan TIMELINE dibuat | Stack difinalkan (Laravel, Vue 3 + TS, IndexedDB); Tauri ditunda; forecast di Laravel |
| 1 Okt 2026 | Tugas T5.7 (test total chart = ledger) ditambahkan di Fase 5 | Total tugas MVP ditetapkan 63 (Fase 9 dirapikan menjadi 8 tugas) |
