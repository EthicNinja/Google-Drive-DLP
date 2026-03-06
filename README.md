# Google Drive DLP Scanner

> **Open source Data Loss Prevention untuk Google Drive — gratis, tanpa server, siap pakai dalam 15 menit.**

---

## Latar Belakang

Skrip ini menjadi alternatif gratis menggunakan **Google Apps Script** yang berjalan langsung di ekosistem Google tanpa memerlukan server eksternal.

---

## Fitur

| Fitur | Keterangan |
|-------|-----------|
| 🔍 **Scan file baru** | Deteksi file yang baru dibuat/diunggah setiap 5 menit |
| 🔗 **Cek sharing permission** | Deteksi file yang dibagikan ke publik atau domain eksternal |
| 📝 **Scan nama file** | Cocokkan nama file dengan kata kunci sensitif |
| 📄 **Scan konten** | Baca isi Google Docs & Sheets, cari pola data sensitif |
| 👁️ **Monitor aktivitas** | Pantau semua aktivitas pada file yang sudah di-flag |
| 🔒 **Auto-revoke** | Cabut permission otomatis saat pelanggaran terdeteksi *(opsional)* |
| 🧹 **Auto-cleanup** | Hapus otomatis flag lama setiap minggu |
| 📧 **Email alert** | Notifikasi langsung ke admin saat ada pelanggaran |

---

## Cara Kerja

```
File baru di-upload/create
          ↓
    scanNewFiles()              ← setiap 5 menit
          ↓
    Ada violation?
    ├── Ya → sendViolationAlert()
    │         saveViolatingFile()
    │         revokePermissions()   ← jika AUTO_REVOKE: true
    └── Tidak → log clean

File yang di-flag
          ↓
    scanFileActivity()          ← setiap 15 menit
          ↓
    Ada aktivitas baru?
    └── Ya → sendActivityAlert()
```

---

## Pola Data Sensitif yang Dideteksi

- Nomor Kartu Kredit (13–16 digit)
- NIK Indonesia (16 digit)
- Password plaintext (`password: ...`)
- Private Key (`-----BEGIN RSA PRIVATE KEY-----`)
- API Key & Token
- AWS Access Key (`AKIA...`)
- Nomor Paspor RI
- Nomor Rekening Bank

> Semua pola dapat dikustomisasi sesuai kebutuhan organisasi.

---

## Prasyarat

- Akun Google (personal atau Google Workspace)
- Akses ke [Google Apps Script](https://script.google.com)
- Tiga Google API diaktifkan via Services:
  - Drive API v3
  - Drive Activity API
  - People API

---

## Instalasi

### 1. Buat project baru di Apps Script

Buka [script.google.com](https://script.google.com) → **New Project**

### 2. Tambahkan API Services

Klik **"+"** di bagian **Services** (sidebar kiri) → tambahkan:
- `Drive API` (pilih versi **v3**)
- `Drive Activity API`
- `People API`

### 3. Paste kode

Copy seluruh isi `GoogleDriveDLP.gs` → paste ke `Code.gs`

### 4. Sesuaikan konfigurasi

```javascript
const CONFIG = {
  ADMIN_EMAIL: 'security@perusahaan.com',   // Email penerima alert
  YOUR_DOMAIN: '@perusahaan.com',            // Domain internal
  SCAN_INTERVAL_MINUTES: 5,                 // Interval scan file baru
  ACTIVITY_CHECK_INTERVAL_MINUTES: 15,      // Interval cek aktivitas
  CLEANUP_DAYS: 30,                         // Umur maksimal flag (hari)
  AUTO_REVOKE: false                        // true = auto-block sharing
};
```

### 5. Aktifkan trigger

Pilih fungsi `createTrigger` di dropdown → klik **Run** → authorize permissions

✅ Selesai. Skrip berjalan otomatis.

---

## Referensi Fungsi

| Fungsi | Tipe | Keterangan |
|--------|------|-----------|
| `createTrigger()` | Setup | Aktifkan semua trigger. Jalankan sekali. |
| `scanNewFiles()` | Auto | Scan file baru. Jalan tiap 5 menit. |
| `scanFileActivity()` | Auto | Monitor aktivitas file yang di-flag. Jalan tiap 15 menit. |
| `cleanOldFlaggedFiles()` | Auto | Hapus flag lama. Jalan tiap Senin. |
| `viewFlaggedFiles()` | Manual | Lihat daftar file yang sedang dimonitor. |
| `clearViolatingFiles()` | Manual | Reset seluruh daftar flag. |
| `listActiveTriggers()` | Debug | Cek trigger yang aktif. |
| `debugSetup()` | Debug | Test koneksi API & setup keseluruhan. |

---

## Contoh Email Alert

**Alert 1 — File baru terdeteksi:**
```
Subject: [DLP Alert] 1 file sensitif baru terdeteksi di Google Drive

File: Data Karyawan Gaji 2025.xlsx
Dibuat: 26/2/2026, 14.03 WIB
Issue: Nama file mengandung kata sensitif: "gaji"
       Di-share ke eksternal: vendor@gmail.com
```

**Alert 2 — Aktivitas pada file yang di-flag:**
```
Subject: [Activity Alert] File sensitif "Data Karyawan Gaji 2025.xlsx" diakses

Aktivitas Terbaru:
#1 — user@gmail.com
     Aksi: [Edit], [Download]
     Waktu: 26/2/2026, 15.30 WIB
```

---

## Limitasi

- ⏱️ **Delay ~5 menit** — bukan real-time seperti solusi enterprise
- 📄 **Scan konten terbatas** — hanya Google Docs & Sheets native; PDF/Office tidak bisa di-scan isinya
- 👤 **Resolusi identitas** — pengguna eksternal mungkin tampil sebagai ID, bukan email
- ⚡ **Quota GAS** — limit 90 menit eksekusi/hari (akun gratis), 6 jam/hari (Workspace)
- 💾 **Storage flag** — menggunakan PropertiesService, limit 500KB (~ratusan file)

---

## Tips Penggunaan

**Untuk pertama kali deploy**, jalankan tanpa `AUTO_REVOKE` selama 1–2 minggu untuk memahami pola false positive di lingkungan Anda sebelum mengaktifkan pemblokiran otomatis.

**Untuk memperluas kata kunci sensitif**, edit array `sensitiveKeywords` di fungsi `checkFileName()` sesuai konteks industri (perbankan, kesehatan, pemerintahan, dll).

**Untuk integrasi SIEM**, tambahkan fungsi pengiriman log ke endpoint Wazuh/Graylog di dalam `scanNewFiles()` setelah violation terdeteksi.

---

## Kontribusi

Kontribusi sangat disambut! Beberapa ide yang bisa dikembangkan:

- [ ] Pola regex tambahan untuk industri spesifik (BPJS, NPWP, nomor rekening BCA/Mandiri, dll)
- [ ] Notifikasi ke Slack / Telegram / WhatsApp
- [ ] Dashboard sederhana untuk visualisasi violation history
- [ ] Scan file PDF & Office via Google Drive export
- [ ] Integrasi log ke Wazuh atau Graylog
- [ ] Support untuk Google Shared Drive (Team Drive)

Silakan buka **Issue** atau **Pull Request**.

---

## Lisensi

MIT License — bebas digunakan, dimodifikasi, dan didistribusikan termasuk untuk keperluan komersial, dengan syarat mencantumkan copyright notice.

```
Copyright (c) 2026 Ethic Ninja (PT Inovasi Media Solusindo)
```

---

## Tentang Ethic Ninja

[Ethic Ninja](https://ethic.ninja) adalah perusahaan cybersecurity Indonesia yang bergerak di bidang penetration testing, IT audit, dan layanan kepatuhan. Bersertifikasi BSSN, ASPI, dan CREST.

Kami membangun tools open source sebagai bentuk kontribusi kepada ekosistem keamanan siber Indonesia.

---
