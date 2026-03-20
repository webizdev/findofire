# PRD: Dasbor Admin FINDO FIRE

## Ringkasan Proyek

Membangun halaman **Admin Dashboard** (`admin.html`) untuk mengelola seluruh konten website FINDO FIRE secara dinamis. Semua data disimpan di **Supabase** (project: `pgltsyrtduvddpcdlchk` — **mix data**) dengan prefix tabel `findofire_`. Gambar di-upload ke **Supabase Storage** bucket `findofire` dengan kompresi otomatis (max 1 MB).

---

## Keputusan Desain (Konfirmasi User)

| No | Keputusan | Detail |
|---|---|---|
| 1 | Supabase Project | Gunakan project **mix data** (`pgltsyrtduvddpcdlchk`) |
| 2 | Akses Dashboard | Password sederhana, default: `1234`, bisa diganti via dashboard |
| 3 | Footer Logo | **Terpisah** dari header logo (gambar berbeda) |
| 4 | Optin Form | Collect data ke Supabase + teruskan ke **WhatsApp CS** |
| 5 | Menu Navigasi | **Hardcoded** (tidak dikelola dari dashboard) |
| 6 | Layanan Footer | Klik → buka **WhatsApp** dengan teks: _"Saya mau konsultasi tentang [nama layanan]"_ |
| 7 | Link INAPROC | Disediakan **input link di dashboard** (disimpan di settings) |

---

## Teknologi

| Komponen | Stack |
|---|---|
| Frontend Website | `index.html` + Tailwind CDN + Vanilla JS |
| Admin Dashboard | `admin.html` + Tailwind CDN + Vanilla JS |
| Backend / Database | Supabase (PostgreSQL) |
| Storage Gambar | Supabase Storage, bucket: `findofire` |
| Kompresi Gambar | Client-side via Canvas API (max 1 MB) |

---

## Struktur Tabel Supabase

### 1. `findofire_settings` — Key-Value Settings
Menyimpan semua data singleton (header, hero, about, CTA, kontak, trust bar, footer, INAPROC link).

| Column | Type | Keterangan |
|---|---|---|
| `key` | `text` PK, UNIQUE | Identifier unik |
| `value` | `text` NULLABLE | Nilai data |
| `updated_at` | `timestamptz` | Default: `now()` |

**Keys yang akan di-seed:**

| Key | Section | Deskripsi |
|---|---|---|
| `header_logo_url` | Header | URL gambar logo header (jika kosong tampilkan text) |
| `header_logo_text` | Header | Text fallback jika logo kosong |
| `inaproc_link` | Header | URL link untuk menu INAPROC |
| `hero_badge_text` | Hero | Badge text ("Pakar Proteksi Kebakaran Industri") |
| `hero_title` | Hero | Judul hero |
| `hero_subtitle` | Hero | Deskripsi hero |
| `hero_bg_image` | Hero | URL gambar background hero |
| `hero_trust_icon` | Hero | FontAwesome class untuk icon trust badge |
| `hero_trust_title` | Hero | Judul trust badge |
| `hero_trust_description` | Hero | Deskripsi trust badge |
| `hero_trust_stat1_value` | Hero | Value stat 1 (e.g. "50+") |
| `hero_trust_stat1_label` | Hero | Label stat 1 |
| `hero_trust_stat2_value` | Hero | Value stat 2 (e.g. "100%") |
| `hero_trust_stat2_label` | Hero | Label stat 2 |
| `hero_btn_primary_text` | Hero | Text tombol CTA primer |
| `hero_btn_primary_link` | Hero | Link tombol CTA primer |
| `hero_btn_secondary_text` | Hero | Text tombol CTA sekunder |
| `hero_btn_secondary_link` | Hero | Link tombol CTA sekunder |
| `about_subtitle` | Tentang | Subtitle (e.g. "Tentang Perusahaan") |
| `about_title` | Tentang | Judul section tentang |
| `about_description_1` | Tentang | Paragraf 1 |
| `about_description_2` | Tentang | Paragraf 2 |
| `about_image` | Tentang | URL gambar tentang |
| `about_workshop_label` | Tentang | Label workshop (e.g. "Workshop Resmi") |
| `about_workshop_location` | Tentang | Lokasi workshop (e.g. "Bekasi Timur") |
| `katalog_subtitle` | Katalog | Subtitle section |
| `katalog_title` | Katalog | Judul section |
| `katalog_description` | Katalog | Deskripsi section |
| `legalitas_subtitle` | Legalitas | Subtitle section |
| `legalitas_title` | Legalitas | Judul section |
| `legalitas_description` | Legalitas | Deskripsi section |
| `cta_title` | CTA | Judul CTA |
| `cta_description` | CTA | Deskripsi CTA |
| `cta_button_text` | CTA | Text tombol CTA |
| `cta_button_link` | CTA | Link tombol CTA (WhatsApp) |
| `contact_subtitle` | Kontak | Subtitle section |
| `contact_title` | Kontak | Judul section |
| `contact_address` | Kontak | Alamat lengkap (HTML line breaks diperbolehkan) |
| `contact_whatsapp` | Kontak | Nomor WhatsApp (format: 6281310007326) |
| `contact_whatsapp_display` | Kontak | Format tampilan nomor WA |
| `contact_whatsapp_note` | Kontak | Catatan (e.g. "Layanan Konsultasi 24/7") |
| `contact_email` | Kontak | Alamat email |
| `contact_maps_embed` | Kontak | URL embed Google Maps |
| `contact_form_title` | Kontak | Judul form kontak |
| `trust_bar_text` | Trust Bar | Heading text trust bar |
| `footer_logo_url` | Footer | URL logo footer (**terpisah** dari header) |
| `footer_logo_text` | Footer | Text fallback jika logo kosong |
| `footer_description` | Footer | Deskripsi footer |
| `footer_workshop_label` | Footer | Label workshop footer |
| `footer_workshop_address` | Footer | Alamat lengkap di footer |
| `footer_workshop_phone` | Footer | Nomor telepon di footer |
| `footer_copyright` | Footer | Text copyright |
| `floating_wa_number` | Floating | Nomor WA untuk floating button |

---

### 2. `findofire_access` — Akses Dashboard
Password proteksi dashboard. Default password: `1234`, bisa diganti via dashboard.

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `password_hash` | `text` | Hash password (SHA-256) |
| `updated_at` | `timestamptz` | Default: `now()` |

> **Catatan:** Password di-hash dengan SHA-256 di client-side. Saat login, input di-hash dan dibandingkan dengan `password_hash` di database.

---

### 3. `findofire_about_features` — Fitur Keunggulan (CRUD)
List item keunggulan di section "Tentang" (Pabrikasi Mandiri, Teknisi Tersertifikasi, dll).

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `icon` | `text` | FontAwesome class (e.g. `fa-solid fa-check-circle`) |
| `title` | `text` | Judul keunggulan (bold text) |
| `description` | `text` | Deskripsi keunggulan |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |
| `created_at` | `timestamptz` | Default: `now()` |

---

### 4. `findofire_services` — Katalog & Layanan (CRUD)
Card produk/layanan di section "Katalog & Layanan".

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `title` | `text` | Nama layanan |
| `description` | `text` | Deskripsi layanan |
| `image_url` | `text` NULLABLE | URL gambar layanan (upload) |
| `icon` | `text` | FontAwesome class |
| `sub_items` | `jsonb` | List sub-item, format: `[{"text": "Fire Jeep"}, ...]` |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |
| `created_at` | `timestamptz` | Default: `now()` |

---

### 5. `findofire_certificates` — Legalitas & Standar (CRUD)
Sertifikat perusahaan di section "Legalitas & Standar".

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `title` | `text` | Nama sertifikat (e.g. "ISO 9001:2015") |
| `description` | `text` | Deskripsi singkat |
| `icon` | `text` | FontAwesome class |
| `icon_color` | `text` | Warna icon (e.g. `text-blue-600`) |
| `image_url` | `text` NULLABLE | URL gambar sertifikat (opsional, alternatif icon) |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |
| `created_at` | `timestamptz` | Default: `now()` |

---

### 6. `findofire_trust_clients` — Trust Bar Klien (CRUD)
Nama-nama klien di marquee trust bar.

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `name` | `text` | Nama instansi/perusahaan |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |
| `created_at` | `timestamptz` | Default: `now()` |

---

### 7. `findofire_social_media` — Media Sosial (CRUD)
Link media sosial di footer.

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `platform` | `text` | Nama platform (Facebook, Instagram, dll) |
| `icon` | `text` | FontAwesome class (e.g. `fa-brands fa-facebook-f`) |
| `url` | `text` | URL profil media sosial |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |
| `created_at` | `timestamptz` | Default: `now()` |

---

### 8. `findofire_contact_services` — Opsi Layanan Form Kontak (CRUD)
Pilihan dropdown layanan di form "Minta Penawaran Harga".

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `label` | `text` | Label tampilan (e.g. "Pembuatan Mobil Pemadam Baru") |
| `value` | `text` | Value option (e.g. "Mobil Baru") |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |

---

### 9. `findofire_footer_services` — Layanan Utama Footer (CRUD)
List layanan utama di footer. **Klik → buka WhatsApp** dengan teks otomatis: _"Saya mau konsultasi tentang [nama layanan]"_.

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `label` | `text` | Nama layanan |
| `urutan` | `integer` | Urutan tampil. Default: `0` |
| `aktif` | `boolean` | Status aktif. Default: `true` |

> **Logika Link:** Saat render di frontend, link di-build otomatis:
> `https://wa.me/{contact_whatsapp}?text=Saya mau konsultasi tentang {label}`

---

### 10. `findofire_optin` — Data Optin Form (Collect)
Menyimpan data pengunjung yang submit form penawaran harga. Setelah tersimpan, otomatis diteruskan ke **WhatsApp CS**.

| Column | Type | Keterangan |
|---|---|---|
| `id` | `uuid` PK | Default: `gen_random_uuid()` |
| `nama` | `text` | Nama / Instansi |
| `telepon` | `text` | Nomor telepon |
| `layanan` | `text` | Layanan yang dipilih |
| `pesan` | `text` NULLABLE | Pesan/detail spesifikasi |
| `created_at` | `timestamptz` | Default: `now()` |

> **Alur Optin:**
> 1. User mengisi form → data disimpan ke `findofire_optin`
> 2. Setelah tersimpan, buka tab baru WhatsApp CS dengan format:
>    ```
>    https://wa.me/{contact_whatsapp}?text=*Permintaan Penawaran Baru*%0A
>    Nama: {nama}%0ANo. HP: {telepon}%0ALayanan: {layanan}%0APesan: {pesan}
>    ```
> 3. Dashboard admin bisa melihat semua data optin (read-only).

---

## Supabase Storage

### Bucket: `findofire`
- **Public bucket** (untuk akses gambar langsung dari frontend)
- Struktur folder:
  ```
  findofire/
  ├── logo/           # Logo header & footer
  ├── hero/           # Gambar background hero
  ├── about/          # Gambar section tentang
  ├── services/       # Gambar katalog & layanan
  ├── certificates/   # Gambar sertifikat (opsional)
  └── general/        # Gambar lainnya
  ```

### Kompresi Gambar
- Kompresi dilakukan **client-side** menggunakan Canvas API
- Maksimal ukuran output: **1 MB**
- Format output: **WebP** (fallback JPEG)
- Kualitas iteratif: mulai dari 0.9, turun bertahap hingga file ≤ 1 MB

---

## Fitur Dashboard (`admin.html`)

### Halaman & Menu Dashboard

| Tab/Menu | Tabel Supabase | Operasi |
|---|---|---|
| 🏠 Header & Logo | `findofire_settings` (header_logo, inaproc_link) | Edit + Upload |
| 🎯 Hero Section | `findofire_settings` (hero_*) | Edit + Upload |
| ℹ️ Tentang Kami | `findofire_settings` (about_*) + `findofire_about_features` | Edit + Upload + CRUD |
| 📦 Katalog & Layanan | `findofire_settings` (katalog_*) + `findofire_services` | Edit + Upload + CRUD |
| 📜 Legalitas & Standar | `findofire_settings` (legalitas_*) + `findofire_certificates` | Edit + Upload + CRUD |
| 📣 CTA Section | `findofire_settings` (cta_*) | Edit |
| 📞 Kontak | `findofire_settings` (contact_*) + `findofire_contact_services` | Edit + CRUD |
| 🏢 Trust Bar | `findofire_settings` (trust_bar_*) + `findofire_trust_clients` | Edit + CRUD |
| 🔗 Footer | `findofire_settings` (footer_*) + `findofire_footer_services` + `findofire_social_media` | Edit + Upload + CRUD |
| 📱 Media Sosial | `findofire_social_media` | CRUD |
| 📋 Data Optin | `findofire_optin` | Read-only (lihat data masuk) |
| 🔒 Ubah Password | `findofire_access` | Edit |

### Fitur Umum Dashboard
1. **Sidebar navigasi** — Menu tab untuk setiap section
2. **Form editor** — Input fields, textarea, icon picker (fontawesome class)
3. **Image upload** — Drag & drop / click upload dengan preview + kompresi otomatis
4. **CRUD tabel** — Tambah, edit, hapus item di tabel list
5. **Urutan / sort** — Input manual urutan
6. **Toggle aktif** — Switch on/off untuk item
7. **Notifikasi** — Toast notification untuk sukses/error
8. **Akses password** — Login dengan password, default `1234`
9. **Ubah password** — Ganti password via dashboard
10. **Lihat data optin** — Tabel read-only data form penawaran harga

---

## Alur Kerja Frontend (`index.html`)

1. Saat halaman dimuat, fetch data dari Supabase:
   - `findofire_settings` → semua key-value settings
   - `findofire_about_features` → list keunggulan (`aktif = true`, order by `urutan`)
   - `findofire_services` → list layanan (`aktif = true`, order by `urutan`)
   - `findofire_certificates` → list sertifikat (`aktif = true`, order by `urutan`)
   - `findofire_trust_clients` → list klien trust bar (`aktif = true`, order by `urutan`)
   - `findofire_social_media` → link media sosial (`aktif = true`, order by `urutan`)
   - `findofire_contact_services` → opsi dropdown form (`aktif = true`, order by `urutan`)
   - `findofire_footer_services` → layanan utama footer (`aktif = true`, order by `urutan`)
2. Render konten ke HTML secara dinamis
3. Jika `header_logo_url` kosong → tampilkan `header_logo_text` sebagai fallback
4. Jika `footer_logo_url` kosong → tampilkan `footer_logo_text` sebagai fallback
5. Menu **INAPROC** → `href` diambil dari `inaproc_link` setting
6. Footer services → klik buka WhatsApp dgn text "Saya mau konsultasi tentang [layanan]"
7. Form optin → simpan ke `findofire_optin` + redirect ke WhatsApp

---

## Data Seed Awal

Semua konten **existing** dari `index.html` akan di-seed ke tabel Supabase:
- Semua text hero, about, katalog headings, CTA, kontak
- 5 item keunggulan (Pabrikasi Mandiri, Teknisi Tersertifikasi, dll)
- 3 service cards (Fire Truck Manufacture, Fire Protection System, Equipment & Rescue)
- 8 sertifikat (ISO 9001, ISO 14001, ISO 45001, NFPA, SNI, KAN, K3 Umum, K3 Kebakaran)
- 10 nama klien trust bar
- 4 link media sosial (Facebook, Instagram, YouTube, TikTok)
- 5 layanan utama footer
- 4 opsi layanan form kontak
- Data kontak (alamat, WA, email, maps embed)
- Password default: `1234` (hashed SHA-256)

---

## File yang Akan Dibuat/Dimodifikasi

| File | Aksi | Keterangan |
|---|---|---|
| `admin.html` | **BARU** | Halaman dashboard admin |
| `index.html` | **MODIFIKASI** | Tambahkan Supabase SDK, fetch & render dinamis |
| Supabase Tables | **BARU** | 10 tabel dengan prefix `findofire_` |
| Supabase Storage | **BARU** | Bucket `findofire` (public) |
| Supabase RLS | **BARU** | Read publik, write hanya via admin |

---

## Ringkasan Tabel (10 tabel)

| No | Tabel | Tipe | Operasi Dashboard |
|---|---|---|---|
| 1 | `findofire_settings` | Key-Value | Edit |
| 2 | `findofire_access` | Singleton | Edit (ubah password) |
| 3 | `findofire_about_features` | List | CRUD |
| 4 | `findofire_services` | List | CRUD |
| 5 | `findofire_certificates` | List | CRUD |
| 6 | `findofire_trust_clients` | List | CRUD |
| 7 | `findofire_social_media` | List | CRUD |
| 8 | `findofire_contact_services` | List | CRUD |
| 9 | `findofire_footer_services` | List | CRUD |
| 10 | `findofire_optin` | Collect | Read-only |

---

## Panduan Penggunaan Dashboard

### 🔐 Login
1. Buka halaman `admin.html`
2. Masukkan password (default: `1234`)
3. Klik **Masuk**

### 🏠 Header & Logo
1. Pilih tab **Header & Logo** di sidebar
2. Upload gambar logo header (drag & drop atau klik area upload)
3. Isi **Text Fallback** — teks ini muncul jika gambar logo kosong
4. Isi **Link INAPROC** — URL untuk menu INAPROC di navbar
5. Klik **Simpan**

### 🎯 Hero Section
1. Pilih tab **Hero Section**
2. Edit field: Badge Text, Judul, Deskripsi
3. Upload gambar background hero
4. Edit Trust Badge: icon, judul, deskripsi, 2 statistik
5. Edit text & link 2 tombol CTA
6. Klik **Simpan**

### ℹ️ Tentang Kami
1. Pilih tab **Tentang Kami**
2. **Bagian atas:** Edit subtitle, judul, 2 paragraf deskripsi, gambar, info workshop
3. **Bagian bawah (Fitur Keunggulan):**
   - Klik **+ Tambah** untuk menambah item baru
   - Isi: Icon (FontAwesome class), Judul, Deskripsi
   - Gunakan toggle **Aktif** untuk show/hide
   - Klik ✏️ untuk edit, 🗑️ untuk hapus
   - Atur urutan dengan input angka

### 📦 Katalog & Layanan
1. Pilih tab **Katalog & Layanan**
2. **Heading section:** Edit subtitle, judul, deskripsi section
3. **Card layanan:**
   - Klik **+ Tambah Layanan** untuk membuat card baru
   - Isi: Nama, Deskripsi, Icon, Upload Gambar
   - **Sub-item:** Tambah detail layanan (list di bawah card)
   - Atur urutan, toggle aktif, edit, atau hapus

### 📜 Legalitas & Standar
1. Pilih tab **Legalitas & Standar**
2. **Heading section:** Edit subtitle, judul, deskripsi
3. **Sertifikat:**
   - Klik **+ Tambah Sertifikat**
   - Isi: Nama, Deskripsi, Icon (FontAwesome class), Warna Icon
   - Opsional: Upload gambar sertifikat (sebagai pengganti icon)
   - Atur urutan, toggle aktif, edit, atau hapus

### 📣 CTA Section
1. Pilih tab **CTA**
2. Edit: Judul, Deskripsi, Text Tombol, Link WhatsApp
3. Klik **Simpan**

### 📞 Kontak
1. Pilih tab **Kontak**
2. Edit semua info kontak: alamat, WhatsApp, email, Maps embed URL
3. Edit judul/subtitle section & judul form
4. **Opsi layanan form:**
   - Klik **+ Tambah Opsi** untuk menambah pilihan dropdown
   - Isi: Label (tampilan) dan Value (nilai internal)
   - Atur urutan, toggle aktif, edit, atau hapus

### 🏢 Trust Bar
1. Pilih tab **Trust Bar**
2. Edit heading text marquee
3. **Daftar klien:**
   - Klik **+ Tambah Klien** untuk menambah nama instansi
   - Atur urutan, toggle aktif, edit, atau hapus

### 🔗 Footer
1. Pilih tab **Footer**
2. Upload **logo footer** (terpisah dari header)
3. Edit: text fallback, deskripsi, info workshop, nomor HP, copyright
4. **Layanan Utama Footer:**
   - Tambah/edit/hapus item layanan
   - Setiap layanan otomatis menjadi link WhatsApp: _"Saya mau konsultasi tentang [nama layanan]"_
5. **Media Sosial:**
   - Klik **+ Tambah Media Sosial**
   - Isi: Platform, Icon (FontAwesome class), URL
   - Atur urutan, toggle aktif, edit, atau hapus

### 📋 Data Optin (Lead)
1. Pilih tab **Data Optin**
2. Lihat tabel data pengunjung yang mengirim form penawaran harga
3. Data meliputi: Nama, Telepon, Layanan, Pesan, Tanggal
4. Data ini **read-only** (tidak bisa diedit/dihapus dari dashboard)

### 🔒 Ubah Password
1. Pilih tab **Ubah Password** di bagian bawah sidebar
2. Masukkan password lama & password baru
3. Klik **Simpan Password Baru**

### 📸 Upload Gambar
- **Format didukung:** JPG, PNG, WebP, GIF
- **Kompresi otomatis:** Gambar di atas 1 MB akan di-compress ke ≤ 1 MB
- **Preview:** Gambar ditampilkan sebelum disimpan
- **Cara upload:** Klik area upload atau drag & drop file ke area upload
