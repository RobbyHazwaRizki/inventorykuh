# PRD: Smart Inventory & Demand Forecast

| | |
|---|---|
| **Versi** | 1.1 |
| **Tanggal** | 1 Oktober 2026 |
| **Pemilik** | Robby Hazwarizki ("Boss") |
| **Status** | Final, siap masuk development |
| **Nama proyek** | Nama kerja, bebas diganti |
| **Dokumen terkait** | [`TIMELINE.md`](./TIMELINE.md) (rencana dan progres development) |

> **Cara memakai dokumen ini:** PRD ini adalah sumber kebenaran untuk *apa* yang dibangun dan *kenapa*. Progres pengerjaan dicatat terpisah di `TIMELINE.md`. Setiap perubahan keputusan atau scope dicatat di bagian [Riwayat Revisi](#17-riwayat-revisi) dan nomor versi dinaikkan.

## Daftar Isi

1. [Latar Belakang & Riwayat Diskusi](#1-latar-belakang--riwayat-diskusi)
2. [Masalah, Tujuan, Non-Tujuan, Metrik](#2-masalah-tujuan-non-tujuan-metrik)
3. [Pengguna & Role](#3-pengguna--role)
4. [Catatan Keputusan](#4-catatan-keputusan-decision-log)
5. [Lingkup Fitur](#5-lingkup-fitur-mvp--v1--later)
6. [Kebutuhan Fungsional](#6-kebutuhan-fungsional)
7. [Kebutuhan Non-Fungsional](#7-kebutuhan-non-fungsional)
8. [Arsitektur](#8-arsitektur)
9. [Model Data](#9-model-data)
10. [API & Event](#10-api--event)
11. [Strategi Pengujian](#11-strategi-pengujian)
12. [Deployment & Rilis](#12-deployment--rilis)
13. [Risiko & Mitigasi](#13-risiko--mitigasi)
14. [Pertanyaan Terbuka & Asumsi](#14-pertanyaan-terbuka--asumsi)
15. [Konvensi Pengembangan](#15-konvensi-pengembangan)
16. [Glosarium](#16-glosarium)
17. [Riwayat Revisi](#17-riwayat-revisi)

---

## 1. Latar Belakang & Riwayat Diskusi

Proyek ini dibuat sebagai **project portofolio** menjelang kelulusan S1 Sistem Informasi, agar bisa dipublish di GitHub dan dilirik perusahaan. Ide dipilih dengan kriteria: berbeda total dari pekerjaan sebelumnya, relevan dengan jurusan, mudah dinilai recruiter, dan menunjukkan struktur kode yang rapi serta aman.

| Tahap | Yang dibahas | Hasil |
|-------|--------------|-------|
| 1 | Ide project fresh | Diajukan 5 ide awal |
| 2 | Permintaan ide yang berbeda total dari pekerjaan sebelumnya | Diajukan 6 ide baru (UMKM Cashflow, Smart Inventory, Antrean Digital, Dashboard Data Publik, Job Tracker, Marketplace Jasa) |
| 3 | Mana yang cocok untuk GitHub | **Smart Inventory & Demand Forecast** dipilih |
| 4 | Stack dan struktur folder | API-first, monorepo |
| 5 | Struktur rapi dan aman | Project sebelumnya dirasa asal-asalan dan rentan, sehingga dipakai layered architecture dan checklist OWASP |
| 6 | Multi-platform | PWA dan APK, didistribusikan lewat GitHub Releases |
| 7 | Hosting | Bisa gratis; tahap awal cukup Docker lokal |
| 8 | Data lokal vs server | Dipilih **model 2: local-first + sinkronisasi real-time** |
| 9 | Bentrok antar staf | Fitur **reservasi stok (soft lock)** ala kursi bioskop |
| 10 | Role dan fitur | 4 role MVP, matriks hak akses, fitur berprioritas |
| 11 | Laporan | **Chart** barang masuk/keluar dan pemasukan untuk Owner dan Manager |
| 12 | Kemudahan dan pembelajaran | Stack disederhanakan: Laravel, Vue 3 + TypeScript, IndexedDB, forecast di Laravel |

---

## 2. Masalah, Tujuan, Non-Tujuan, Metrik

### 2.1 Masalah yang diselesaikan

- Usaha kecil-menengah mencatat stok manual atau terpisah-pisah, sehingga angka tidak sinkron antar staf.
- Dua staf bisa menjual barang terakhir yang sama (overselling).
- Aplikasi online-only berhenti saat sinyal buruk.
- Pemilik sulit melihat pola barang masuk/keluar dan kapan harus restock.

### 2.2 Tujuan

1. Satu bisnis, banyak staf, data stok yang sama di semua perangkat secara real-time.
2. Tetap bisa dipakai saat offline, lalu sinkron otomatis.
3. Mencegah stok terjual ganda saat online.
4. Memberi Owner dan Manager visibilitas lewat chart dan alert.
5. Menjadi **showcase portofolio**: arsitektur rapi, aman, terdokumentasi, dan bisa dicoba.
6. Menjadi sarana belajar teknologi baru (Laravel, Vue 3 + TypeScript, IndexedDB, WebSocket, concurrency) dengan kesulitan yang terkendali.

### 2.3 Non-tujuan

- Bukan POS lengkap (tanpa payment gateway, printer struk, pajak).
- Bukan sistem akuntansi penuh.
- Tanpa integrasi marketplace atau e-commerce.
- Tanpa multi-gudang di MVP.
- Tanpa rilis App Store atau Play Store (cukup GitHub Releases dan PWA).
- Tanpa installer desktop di MVP (PWA sudah bisa di-install di desktop).

### 2.4 Metrik keberhasilan (target)

| Metrik | Target |
|--------|--------|
| Perubahan terlihat di perangkat lain | < 2 detik saat semua perangkat online |
| Uji concurrency reservasi | 0 overselling pada 50 permintaan paralel |
| CI | Hijau (lint, analisis statis, test, security scan) |
| Cakupan test layer service | ≥ 70% |
| Demo | Live + akun demo + data dummy |
| Dokumentasi | README lengkap, diagram arsitektur, GIF, 1 case study |
| Validasi nyata (opsional) | 3 sampai 5 pengguna uji beserta testimoni |

---

## 3. Pengguna & Role

**Target:** satu bisnis dengan beberapa staf (toko/UMKM dengan gudang kecil-menengah).

| Role | Persona | Kebutuhan utama |
|------|---------|-----------------|
| **Owner** | Pemilik bisnis | Kendali penuh, laporan, kelola staf |
| **Manager** | Kepala gudang/supervisor | Kelola produk, setujui koreksi, pantau operasional |
| **Warehouse Staff** | Staf gudang | Catat barang masuk/keluar, ajukan opname |
| **Cashier / Sales** | Kasir/penjual | Buat penjualan, tahan stok |
| **Viewer / Auditor** (V1) | Akuntan/pihak luar | Hanya lihat laporan dan riwayat |
| **Super Admin** (level platform) | Pengembang | Kelola akun bisnis/demo; tanpa akses baca data bisnis kecuali untuk dukungan yang tercatat |

Role disimpan **per bisnis**: satu akun bisa tergabung di beberapa bisnis dengan role berbeda.

### 3.1 Matriks Hak Akses

| Aksi | Owner | Manager | Warehouse | Cashier | Viewer |
|------|:-----:|:-------:|:---------:|:-------:|:------:|
| Kelola staf & role | ✅ | ❌ | ❌ | ❌ | ❌ |
| Pengaturan bisnis | ✅ | ❌ | ❌ | ❌ | ❌ |
| Tambah/ubah produk & harga | ✅ | ✅ | ❌ | ❌ | ❌ |
| Kelola supplier | ✅ | ✅ | ❌ | ❌ | ❌ |
| Catat barang masuk/keluar | ✅ | ✅ | ✅ | ❌ | ❌ |
| Ajukan stok opname | ✅ | ✅ | ✅ | ❌ | ❌ |
| **Setujui** koreksi stok | ✅ | ✅ | ❌ | ❌ | ❌ |
| Buat penjualan / tahan stok | ✅ | ✅ | ❌ | ✅ | ❌ |
| **Chart & dashboard lengkap** | ✅ | ✅ | ⚠️ ringkas | ❌ | ✅ |
| Lihat harga beli & margin | ✅ | ⚠️ diatur Owner | ❌ | ❌ | ✅ |
| Lihat audit log | ✅ | ✅ | ❌ | ❌ | ✅ |
| Ekspor data | ✅ | ✅ | ❌ | ❌ | ✅ |

**Prinsip:**
- Izin dicek berdasarkan **permission** (misal `stock.adjust.approve`), bukan `if role == ...`. Role hanyalah kumpulan izin.
- Kasir tidak boleh melihat harga beli dan margin.
- Staf gudang tidak boleh menyetujui koreksi stok yang ia ajukan sendiri (*separation of duties*).
- Semua pengecekan izin dilakukan **di server**; menyembunyikan tombol di UI hanya untuk kenyamanan.

---

## 4. Catatan Keputusan (Decision Log)

| # | Keputusan | Alasan | Status |
|---|-----------|--------|--------|
| D1 | Project: Smart Inventory & Demand Forecast | Cocok GitHub, nyambung jurusan SI, tanpa API berbayar | ✅ Final |
| D2 | Arsitektur **API-first**, monorepo | Satu backend, banyak klien | ✅ Final |
| D3 | **Local-first + sync real-time** (model 2) | Staf banyak, tahan offline | ✅ Final |
| D4 | Stok disimpan sebagai **ledger mutasi append-only** | Menghilangkan sebagian besar konflik sync, ada audit trail | ✅ Final |
| D5 | **Reservasi stok ber-TTL** (soft lock) | Mencegah overselling saat online | ✅ Final |
| D6 | Offline: transaksi berstatus "menunggu konfirmasi", server memvalidasi saat sinkron | Offline tidak bisa dijamin mutlak | ✅ Final |
| D7 | Server DB: **PostgreSQL** | Gratis, kuat, cocok analitik | ✅ Final |
| D8 | Distribusi: **PWA lalu APK (Capacitor)** via GitHub Releases | Gratis, satu basis kode frontend | ✅ Final |
| D9 | Frontend berupa **SPA terpisah**, bukan Inertia | Inertia berbasis halaman server, kurang cocok untuk offline-first | ✅ Final |
| D10 | Backend: **Laravel** (API-first, Sanctum, Reverb) | Fitur keamanan dan infrastruktur bawaan, paling mudah dirakit | ✅ Final |
| D11 | Frontend: **Vue 3 + TypeScript + Vite** | Kurva belajar landai, TypeScript menangkap error lebih awal | ✅ Final |
| D12 | Hosting: mulai dari **Docker lokal**; keputusan hosting demo diambil di Fase 9 | Fokus belajar dulu | 🟡 Ditunda ke Fase 9 |
| D13 | Forecast **dibuat di Laravel** (moving average / exponential smoothing); FastAPI menjadi tantangan tambahan | Menghindari dua bahasa dan dua deployment | ✅ Final |
| D14 | DB lokal: **IndexedDB via Dexie** untuk semua platform | Satu cara untuk PWA, APK; tidak perlu belajar SQLite per platform | ✅ Final |
| D15 | Installer desktop (Tauri) dipindah ke **Later** | PWA sudah bisa di-install di desktop | ✅ Final |
| D16 | Enkripsi DB lokal penuh dipindah ke **V1**; MVP memakai PIN lock dan penyimpanan token aman | IndexedDB tidak terenkripsi otomatis; MVP memakai data dummy | ✅ Final |

---

## 5. Lingkup Fitur (MVP / V1 / Later)

**MVP** = wajib ada di rilis v1.0. **V1** = tambahan setelah inti stabil (rilis v1.1). **Later** = jika waktu masih ada.

| Modul | Fitur | Prioritas |
|-------|-------|:---------:|
| Autentikasi | Login/logout, token per perangkat, daftar perangkat aktif | MVP |
| | Undang staf via kode/link, PIN lokal | V1 |
| Bisnis & staf | Multi-tenant, kelola staf & role | MVP |
| Produk | CRUD produk, kategori, satuan, SKU/barcode | MVP |
| | Varian, foto | V1 |
| | Batch & kedaluwarsa | Later |
| Supplier | Data supplier, riwayat pembelian | V1 |
| Stok | Ledger mutasi, stok saat ini, reorder point, alert stok menipis | MVP |
| | Stok opname + approval | V1 |
| | Multi-lokasi & transfer | Later |
| Penjualan | Transaksi sederhana yang mengurangi stok | MVP |
| | **Reservasi stok (soft lock)** | MVP |
| | Retur | V1 |
| Sinkronisasi | DB lokal, outbox, sync saat online | MVP |
| | Real-time WebSocket + indikator koneksi | MVP |
| | Layar penyelesaian konflik/anomali | V1 |
| **Laporan & chart** | **Chart barang masuk vs keluar, pemasukan, produk terlaris, komposisi mutasi** (Owner & Manager) | **MVP** |
| | Dashboard stok kritis | MVP |
| | Ekspor chart/data (CSV/gambar) | V1 |
| Forecast | Prediksi permintaan 30 hari per produk (di Laravel) | V1 |
| | Rekomendasi jumlah pesan ulang | V1 |
| | Layanan forecast terpisah (FastAPI) | Later |
| Notifikasi | Alert dalam aplikasi (stok menipis, anomali) | MVP |
| | Push / WhatsApp | Later |
| Keamanan | Audit log, rate limit | MVP |
| | Riwayat login, enkripsi DB lokal penuh | V1 |
| Data | Backup/restore, ekspor CSV/Excel | V1 |
| Platform | PWA, APK | MVP |
| | Installer desktop (Tauri) | Later |

---

## 6. Kebutuhan Fungsional

### FR-AUTH: Autentikasi & Perangkat
- Login email + password (hash argon2/bcrypt), token per perangkat (Laravel Sanctum).
- Pengguna bisa melihat dan mencabut perangkat aktif.
- **Kriteria terima:** perangkat yang tokennya dicabut tidak bisa mengirim atau menerima data setelah tersambung kembali.

### FR-TENANT: Multi-bisnis
- Semua tabel bisnis memuat `business_id`; semua query otomatis dibatasi per bisnis (global scope).
- **Kriteria terima:** pengguna bisnis A tidak bisa membaca atau menulis data bisnis B (diuji otomatis).

### FR-PROD: Produk
- CRUD produk dengan SKU unik per bisnis, kategori, satuan, harga beli/jual, reorder point.
- Penghapusan berupa *soft delete*.

### FR-STOCK: Ledger Stok
- Stok **tidak pernah diedit langsung**; hanya berubah lewat mutasi (`in`, `out`, `sale`, `adjustment`, `return`).
- Stok saat ini = jumlah semua mutasi terkonfirmasi.
- Setiap mutasi memuat UUID dari klien, pembuat, perangkat, waktu kejadian, dan waktu diterima server.
- Alert muncul saat stok ≤ reorder point.
- **Kriteria terima:** mengirim mutasi yang sama dua kali tidak menggandakan stok (idempoten).

### FR-RSV: Reservasi Stok (Soft Lock)
- Kasir menahan jumlah tertentu; stok tersedia untuk orang lain = stok fisik dikurangi total tahanan aktif.
- Tahanan punya `expires_at` (default 5 menit), diperpanjang lewat heartbeat selama aplikasi aktif, dan dilepas otomatis saat habis, batal, atau aplikasi mati.
- Permintaan ditangani server secara atomik (transaksi + row lock). Yang telat menerima penolakan dan tampilan "tidak tersedia" (merah).
- Perubahan status disiarkan real-time ke semua perangkat bisnis itu.
- Barang unik (bernomor seri/lokasi tertentu) dilindungi *unique constraint* di database.
- Reservasi **hanya berjalan saat online**; saat offline lihat FR-SALE.
- **Kriteria terima:** 50 permintaan paralel untuk stok 1 hanya menghasilkan 1 sukses.

### FR-SALE: Penjualan
- Transaksi dibuat dari reservasi; saat dikonfirmasi, reservasi menjadi mutasi `sale` permanen.
- Saat offline, transaksi tersimpan lokal berstatus **pending_review** dan divalidasi server saat sinkron. Jika stok tidak cukup, transaksi ditandai bermasalah dan staf diberi tahu.

### FR-SYNC: Sinkronisasi Local-First
- **Entitas yang disinkronkan di MVP:** kategori, produk, mutasi stok, penjualan (+ item). Reservasi tidak disinkronkan karena hanya online.
- Perubahan ditulis ke DB lokal (IndexedDB) lebih dulu sehingga UI instan, lalu masuk **outbox**.
- **Push:** kirim outbox ke server (batch, idempoten). **Pull:** ambil perubahan sejak `cursor` terakhir.
- **Konflik:** stok lewat ledger (tidak ada konflik); data master memakai *last-write-wins* berdasarkan waktu **server**; penghapusan lewat *soft delete*.
- Stok negatif akibat transaksi offline ditandai **anomali** dan memicu alert ke Owner/Manager.
- Indikator koneksi di UI: online (hijau), offline (kuning), menyinkronkan.
- **Kriteria terima:** dua perangkat yang bekerja offline lalu tersambung menghasilkan stok yang sama dan identik di kedua perangkat.

### FR-RT: Real-Time
- Channel privat per bisnis (`private-business.{id}`) yang diotorisasi server (Laravel Reverb + Echo).
- Event: mutasi tercatat, produk berubah, reservasi dibuat/dilepas, anomali.
- Jika WebSocket putus, aplikasi fallback ke pull berkala.

### FR-REP: Chart & Laporan (Owner & Manager)

| Chart | Isi | Akses |
|-------|-----|-------|
| Barang masuk vs keluar | Bar/line per hari, minggu, bulan; filter produk/kategori | Owner, Manager |
| Pemasukan | Nilai penjualan dari waktu ke waktu | Owner, Manager (Owner bisa membatasi) |
| Produk terlaris | Top 10 berdasarkan jumlah keluar | Owner, Manager |
| Komposisi mutasi | Donut per tipe mutasi | Owner, Manager |
| Stok kritis | Daftar produk di bawah reorder point | Owner, Manager, Warehouse (ringkas) |

- Filter: rentang tanggal, produk, kategori.
- Data diagregasi dari ledger di server, di-cache di perangkat; saat offline tampil data terakhir dengan label "terakhir diperbarui".
- Warehouse hanya melihat ringkasan tanpa nilai rupiah; Cashier tidak melihat chart.
- Library chart dipilih di Fase 5 (kandidat: Chart.js atau ECharts).
- **Kriteria terima:** total yang ditampilkan di chart sama dengan hasil penjumlahan ledger pada rentang yang sama (diuji otomatis).
- *Catatan asumsi:* "pemasukan gudang" dimaknai sebagai **barang masuk** (volume) dan **nilai pemasukan penjualan** (rupiah). Lihat [Pertanyaan Terbuka](#14-pertanyaan-terbuka--asumsi).

### FR-AUDIT: Audit Log
- Aksi sensitif (ubah harga, hapus produk, koreksi, perubahan role) dicatat: siapa, kapan, perangkat, nilai sebelum dan sesudah.

### FR-APPR (V1): Opname & Approval
1. Staf mengajukan koreksi dengan alasan; status **menunggu**.
2. Manager/Owner menyetujui atau menolak, dengan catatan.
3. Hanya yang disetujui masuk ledger dan tersiarkan.
4. Seluruh langkah tercatat di audit log.

### FR-FC (V1): Forecast di Laravel
- Job terjadwal (scheduler) membaca ledger penjualan harian per produk dan menghitung prediksi 30 hari ke depan memakai **moving average** atau **exponential smoothing**.
- Hasil disimpan di tabel `forecasts` dan ditampilkan di dashboard; dibagikan ke perangkat lewat API laporan (di-cache).
- Dashboard menampilkan metrik akurasi sederhana (misal MAE/MAPE hasil *backtest* pada data historis).
- Jika data historis produk belum cukup, tampilkan "data belum cukup" alih-alih angka menyesatkan.
- **Kriteria terima:** forecast dihitung ulang otomatis terjadwal dan dapat dijelaskan (metode dan parameter tercatat).

---

## 7. Kebutuhan Non-Fungsional

| Area | Persyaratan |
|------|-------------|
| **Keamanan** | Ikuti OWASP Top 10: validasi input (FormRequest), Eloquent/query berparameter, Policy di setiap endpoint, rate limit, API Resource (tanpa field sensitif), `APP_DEBUG=false` di produksi, HTTPS, secret hanya di `.env`/GitHub Secrets, scan secret dan dependensi |
| **Data lokal** | MVP: PIN lock aplikasi dan token di penyimpanan aman perangkat; data lokal MVP hanya data dummy. V1: enkripsi data lokal. IndexedDB tidak terenkripsi secara otomatis |
| **Kinerja** | UI merespons instan (baca/tulis lokal); sync < 2 detik pada kondisi normal; dashboard < 3 detik pada data demo |
| **Keandalan** | Idempotensi di semua endpoint sync; backup database terjadwal |
| **Offline** | Fitur inti (catat mutasi, lihat stok terakhir, buat penjualan pending) jalan tanpa internet |
| **Kualitas kode** | PSR-12 + Laravel Pint, Larastan/PHPStan, Pest/PHPUnit, ESLint + Prettier + `vue-tsc` untuk frontend, Conventional Commits |
| **Kompatibilitas** | Android (APK), PWA di semua browser modern termasuk iOS dan desktop |
| **Lokalisasi** | UI Bahasa Indonesia, mata uang IDR; README berbahasa Inggris dan Indonesia |
| **Observability** | Log terstruktur, endpoint health check |

---

## 8. Arsitektur

```
 HP / Tablet / Laptop (PWA · APK)
 ┌────────────────────────────────────────┐
 │ SPA Vue 3 + TS · IndexedDB (Dexie)     │
 │ Outbox · Sync engine · Echo (WebSocket)│
 └────────────┬───────────────▲───────────┘
         REST │ push/pull     │ WebSocket (siaran)
 ┌────────────▼───────────────┴───────────┐
 │ Laravel API · Reverb · Queue · Scheduler│──► PostgreSQL
 │ (Auth, Policy, Ledger, Reservasi,       │──► Redis
 │  Laporan, Forecast)                     │
 └─────────────────────────────────────────┘
```

**Alur utama:** tulis lokal → masuk outbox → push ke server → validasi → simpan → siarkan → perangkat lain menarik perubahan → UI diperbarui.

### 8.1 Struktur Repo (Monorepo)

```
smart-inventory/
├── backend/                       # Laravel API
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/Api/V1/   # tipis: terima request, panggil service
│   │   │   ├── Requests/             # validasi input
│   │   │   ├── Resources/            # format output JSON
│   │   │   └── Middleware/
│   │   ├── Services/                 # logika bisnis (Stock, Reservation, Sale, Report, Forecast)
│   │   ├── Repositories/             # akses data
│   │   ├── Policies/                 # hak akses
│   │   ├── Models/  Enums/  DTOs/
│   │   ├── Jobs/  Events/  Console/
│   ├── database/{migrations, seeders, factories}
│   ├── routes/api.php
│   ├── tests/{Unit, Feature, Concurrency}
│   ├── phpstan.neon
│   └── pint.json
├── frontend/                      # SPA Vue 3 + TypeScript (PWA)
│   └── src/
│       ├── pages/  components/  stores/
│       ├── services/              # pemanggil API
│       ├── db/                    # Dexie (IndexedDB)
│       └── sync/                  # outbox, push, pull, cursor
├── packaging/
│   └── capacitor/                 # build APK
├── docs/
│   ├── PRD.md
│   ├── TIMELINE.md
│   ├── adr/                       # catatan keputusan arsitektur
│   ├── architecture.png
│   ├── erd.png
│   └── screenshots/
├── docker-compose.yml
├── .env.example
├── .github/workflows/{ci.yml, release.yml}
└── README.md
```

**Aturan emas arsitektur berlapis:** alur request adalah `Route → Controller → FormRequest → Service → Repository → Model`. Controller tidak berisi logika bisnis dan tidak menyentuh database langsung.

---

## 9. Model Data

Semua ID memakai **UUID**; semua tabel bisnis memuat `business_id`.

| Tabel | Kolom penting |
|-------|---------------|
| `businesses` | name, settings |
| `users` | name, email, password_hash |
| `business_user` | business_id, user_id, role, status |
| `devices` | user_id, name, last_seen_at, revoked_at |
| `categories`, `suppliers` | name, kontak |
| `products` | sku, barcode, name, category_id, unit, cost_price, sell_price, reorder_point, deleted_at |
| `stock_movements` (ledger) | product_id, type, qty (bertanda), unit_cost, note, created_by, device_id, occurred_at, received_at, status (`confirmed` / `pending_review` / `flagged`) |
| `reservations` | product_id, staff_id, device_id, qty, status (`active` / `confirmed` / `released` / `expired`), expires_at |
| `sales`, `sale_items` | total, status, kasir, item |
| `stock_adjustments` (V1) | product_id, qty_diff, reason, status, requested_by, reviewed_by |
| `forecasts` (V1) | product_id, target_date, predicted_qty, method, params, computed_at |
| `audit_logs` | actor, action, entity, before, after, device_id, ip |
| `sync_changes` | seq (cursor), entity, entity_id, op, payload |
| `notifications` | tipe, target, dibaca |
| *Sisi klien:* `outbox`, `sync_state` | antrean perubahan, cursor terakhir |

---

## 10. API & Event

| Kelompok | Endpoint (prefix `/api/v1`) |
|----------|----------------------------|
| Auth | `POST /auth/login`, `POST /auth/logout`, `GET /me`, `GET/DELETE /devices` |
| Master | `/products`, `/categories`, `/suppliers` |
| Stok | `POST /movements` (batch, idempoten) |
| Reservasi | `POST /reservations`, `PATCH /reservations/{id}/heartbeat`, `POST /reservations/{id}/confirm`, `DELETE /reservations/{id}` |
| Penjualan | `POST /sales` |
| Sinkron | `POST /sync/push`, `GET /sync/pull?since={seq}` |
| Laporan | `GET /reports/movements-summary`, `/reports/revenue`, `/reports/top-products`, `/reports/composition` |
| Forecast (V1) | `GET /forecasts` |
| Event WebSocket | `MovementRecorded`, `ProductChanged`, `StockReserved`, `ReservationReleased`, `AnomalyFlagged` |

Semua endpoint menolak akses tanpa token, memvalidasi input di server, dan mengembalikan data lewat API Resource.

---

## 11. Strategi Pengujian

| Jenis | Fokus |
|-------|-------|
| Unit | Service stok, perhitungan stok tersedia, aturan reservasi, perhitungan forecast |
| Feature | Endpoint, validasi, **izin per role**, isolasi antar bisnis ("user A akses data bisnis B") |
| Concurrency | 50 permintaan paralel pada stok terbatas |
| Sync | Kirim ulang (idempoten), offline lalu online, dua perangkat, perangkat dicabut |
| Laporan | Total chart sama dengan jumlah ledger |
| E2E | Alur login, catat mutasi, jual, lihat chart |
| Keamanan | `composer audit`, `npm audit`, gitleaks, cek header, uji rate limit, IDOR dan mass assignment |

**CI (GitHub Actions):** lint, analisis statis, test, dan security scan di setiap push dan pull request.

---

## 12. Deployment & Rilis

- **Dev dan self-host:** `docker compose up` menjalankan Laravel (app), PostgreSQL, Redis, Reverb, queue worker, dan scheduler dengan data dummy.
- **Demo publik (keputusan di Fase 9):** pilihan berupa frontend di Cloudflare Pages + backend di server pribadi lewat Cloudflare Tunnel (tanpa buka port router), atau free tier cloud. Syarat layanan gratis bisa berubah, cek dokumen resminya saat memutuskan.
- **Hardening server pribadi (jika dipakai):** user non-root di Docker, firewall, SSH key, backup DB, hanya data dummy, dipisahkan dari data pribadi.
- **Rilis:** tag `vX.Y.Z` memicu workflow yang membangun APK dan mengunggah ke GitHub Releases, lengkap dengan `checksums.txt` (SHA-256) dan release notes.
- **Penandatanganan:** keystore APK disimpan di GitHub Secrets, bukan di repo. Jelaskan di README bahwa APK di luar Play Store memunculkan peringatan "sumber tidak dikenal".
- **Deploy otomatis:** gunakan model *pull* dari server; hindari menaruh kunci SSH server di GitHub; aktifkan 2FA GitHub; jangan pasang self-hosted runner di repo publik.
- **Versi dan cabang:** `main` (rilis), `develop` (integrasi), `feature/*` lewat pull request.

---

## 13. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|--------|--------|----------|
| Scope terlalu besar | Tidak selesai | Patuhi MVP/V1/Later; kunci MVP dulu |
| Sinkronisasi rumit | Bug data | Ledger append-only, UUID klien, entitas sync dibatasi, test sync sejak awal |
| Overselling saat offline | Stok negatif | Status pending, deteksi anomali, opname |
| Belajar teknologi baru sekaligus | Macet di tengah | Urutan fase dari mudah ke berat; tiap fase menghasilkan sesuatu yang bisa didemokan |
| Kebocoran secret | Server terekspos | `.env.example`, gitleaks, rotasi kunci jika pernah bocor |
| Konflik jam perangkat | Urutan salah | Waktu otoritatif dari server |
| Data lokal tidak terenkripsi di MVP | Data terbaca jika perangkat hilang | Data dummy di MVP, PIN lock, enkripsi di V1 |
| Server pribadi mati saat demo | Demo tak bisa diakses | GIF/video, opsi self-host, cadangan free tier |

---

## 14. Pertanyaan Terbuka & Asumsi

| # | Pertanyaan / Asumsi | Default jika tidak dijawab |
|---|---------------------|----------------------------|
| Q1 | "Pemasukan gudang" bermakna barang masuk, nilai penjualan, atau keduanya? | Keduanya |
| Q2 | Jenis bisnis untuk data dummy (toko ritel, distributor, kuliner)? | Toko ritel dengan gudang kecil |
| Q3 | Hosting demo: server pribadi + Tunnel atau free tier cloud? | Diputuskan di Fase 9 |
| Q4 | Waktu luang per minggu untuk mengerjakan proyek | Asumsi 15 sampai 20 jam/minggu; estimasi di `TIMELINE.md` bergantung pada ini |
| Q5 | Repo publik dengan lisensi MIT? | Ya |
| Q6 | Library chart: Chart.js atau ECharts? | Diputuskan di Fase 5 |

---

## 15. Konvensi Pengembangan

- **Commit:** Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`).
- **Cabang:** `main`, `develop`, `feature/<nama>`; semua perubahan lewat pull request.
- **Gaya kode:** PSR-12 (Pint) untuk PHP; ESLint + Prettier untuk TypeScript/Vue.
- **Penamaan:** tabel jamak snake_case, endpoint kata benda jamak, event berbentuk kata kerja lampau.
- **Dokumentasi keputusan:** setiap keputusan arsitektur penting dicatat singkat di `docs/adr/NNNN-judul.md` (konteks, keputusan, konsekuensi).
- **Definition of Done:** lihat `TIMELINE.md`.

---

## 16. Glosarium

| Istilah | Arti |
|---------|------|
| **API-first** | Backend satu, menyediakan API yang dipakai semua klien |
| **Local-first** | Data utama ditulis ke perangkat lebih dulu, lalu disinkronkan |
| **Ledger** | Buku besar mutasi yang hanya bertambah; stok adalah jumlah mutasi |
| **Outbox** | Antrean perubahan lokal yang menunggu dikirim ke server |
| **Cursor** | Penanda posisi sinkron terakhir (`seq`) |
| **Idempoten** | Mengirim permintaan yang sama berulang kali tidak mengubah hasil akhir |
| **Soft lock / reservasi** | Penahanan sementara berbatas waktu atas stok |
| **TTL** | Batas umur sebuah tahanan sebelum dilepas otomatis |
| **PWA** | Aplikasi web yang bisa di-install dan bekerja offline |
| **Policy** | Aturan izin per aksi di Laravel |
| **ADR** | Architecture Decision Record, catatan singkat sebuah keputusan |

---

## 17. Riwayat Revisi

| Versi | Tanggal | Perubahan |
|-------|---------|-----------|
| 1.0 | 1 Okt 2026 | Draf pertama: seluruh hasil diskusi hingga fitur, role, dan chart |
| 1.1 | 1 Okt 2026 | Stack disederhanakan: Laravel dan Vue 3 + TypeScript difinalkan (D10, D11); forecast dipindah ke Laravel (D13); IndexedDB/Dexie sebagai DB lokal (D14); Tauri ditunda ke Later (D15); enkripsi DB lokal ke V1 (D16); hosting demo diputuskan di Fase 9 (D12); entitas sync MVP dibatasi |
