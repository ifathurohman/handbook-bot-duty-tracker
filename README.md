# 🤖 Duty Tracker Bot — Dokumentasi Lengkap

> Bot absensi & duty tracking untuk Discord, terintegrasi dengan Google Sheets secara real-time.

---

## 📋 Daftar Isi

- [Cara Kerja Sistem](#-cara-kerja-sistem)
- [Command Umum](#-command-umum-semua-anggota)
- [Command Admin](#-command-admin)
- [Command Moderator](#-command-moderator)
- [Sheet Google Sheets](#-sheet-google-sheets)
- [Sistem Role & Akses](#-sistem-role--akses)
- [Konfigurasi .env](#-konfigurasi-env)
- [Jabatan & Tunjangan](#-jabatan--tunjangan)
- [Status Karyawan](#-status-karyawan)
- [Catatan Penting](#-catatan-penting)

---

## ⚙️ Cara Kerja Sistem

```
/dutyin  →  Sesi dimulai, dicatat ke sheet Attendance
           ↓
/dutyout →  Sesi selesai, durasi dihitung
           ↓
           Sheet diperbarui otomatis:
           ├── Attendance        (riwayat sesi)
           ├── Duty Summary      (rekap harian)
           ├── Duty Report       (rekap mingguan)
           ├── Perhitungan Tunjangan (estimasi gaji)
           └── Audit Log         (jejak semua aksi)
```

**Target Kerja:**
| Periode | Target Default |
|---------|---------------|
| Harian  | 4 jam (240 menit) |
| Mingguan | 24 jam (1440 menit) |

> Target bisa diubah di file `.env` via `DAILY_TARGET` dan `WEEKLY_TARGET`.

---

## 👥 Command Umum (Semua Anggota)

### `/dutyin`
Memulai sesi duty.

**Syarat:**
- Tidak sedang dalam sesi duty aktif
- Status karyawan harus `Aktif` atau `Cuti`
- Role tidak masuk daftar `BLOCKED_ROLES`

**Output embed:**
```
✅ Duty Dimulai
👤 Nama       : Lorencio ipsum
💼 Jabatan    : Menteri
📋 Status     : Aktif
🕐 Waktu Masuk: 09:00:00
📆 Minggu Ini : 6.3j / 24j
📊 Progress   : 🟩🟩⬛⬛⬛⬛⬛⬛⬛⬛ 26%
```

---

### `/dutyout`
Mengakhiri sesi duty yang sedang berjalan.

**Opsi:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `catatan` | String | ❌ | Catatan opsional untuk sesi ini |

**Syarat:**
- Harus sedang dalam sesi duty aktif
- Durasi minimal 1 menit (bisa diubah via `MIN_SESSION_MIN`)

**Output embed:**
```
🔴 Duty Selesai
👤 Nama          : Lorencio ipsum
🕐 Masuk         : 09:00:00
🕑 Keluar        : 11:30:00
⏱️ Durasi Sesi  : 2j 30m 0d
📊 Total Menit   : 150 menit
📅 Total Jam Sesi: 2.50 jam
📆 Minggu Ini    : 8.8j / 24j
✅ Progress      : 🟩🟩🟩⬛⬛⬛⬛⬛⬛⬛ 37%
💰 Est. Tunjangan: Rp 5.500
```

---

### `/dutystatus`
Mengecek durasi sesi duty yang sedang berjalan.

> 🔒 Hanya terlihat oleh diri sendiri (ephemeral)

**Output embed:**
```
🟢 Sedang Duty
👤 Nama       : Lorencio ipsum
🕐 Masuk      : 09:00:00
⏱️ Durasi    : 1j 15m 30d
📊 Menit      : 75 menit
📆 Progress Minggu: 🟩⬛⬛⬛⬛⬛⬛⬛⬛⬛ 5%
💡 Info       : ~22j lagi untuk target minggu
```

---

### `/myduty`
Melihat statistik duty pribadi secara lengkap.

> 🔒 Hanya terlihat oleh diri sendiri (ephemeral)

**Output embed:**
```
📊 Statistik Duty Saya
Halo Lorencio ipsum, berikut ringkasan duty kamu.

👤 Nama     : Lorencio ipsum
💼 Jabatan  : Menteri
📋 Status   : 🟢 Aktif

📅 Hari Ini
🟩🟩⬛⬛⬛⬛⬛⬛⬛⬛ 20%
120 mnt (2.00j)

📆 Minggu Ini
🟩🟩🟩🟩⬛⬛⬛⬛⬛⬛ 40%
576 mnt (9.60j)

🏆 All Time    : 553 mnt (9.22j)
🔢 Total Sesi  : 6 sesi
💰 Est. Tunjangan: Rp 6.000
```

---

### `/topduty`
Menampilkan leaderboard duty minggu ini.

**Opsi:**
| Parameter | Tipe | Wajib | Default | Keterangan |
|-----------|------|-------|---------|------------|
| `jumlah`  | Integer | ❌ | 10 | Jumlah user yang ditampilkan (1–25) |

**Output embed:**
```
🏆 Leaderboard Duty Mingguan
Periode: 05/05/2025 — 11/05/2025

🥇 Lorencio ipsum *(Menteri)*
████████░ 9.6j (576 mnt)

🥈 Budi Santoso *(Staff)*
██████░░░ 7.2j (432 mnt)

🥉 Andi Pratama *(Kepala Dept.)*
████░░░░░ 4.8j (288 mnt)

#4 Sari Dewi *(Magang)*
██░░░░░░░ 2.4j (144 mnt)
```

---

### `/botinfo`
Menampilkan informasi bot, uptime, dan konfigurasi aktif.

> 🔒 Hanya terlihat oleh diri sendiri (ephemeral)

**Output embed:**
```
🤖 Duty Tracker Bot
Bot absensi & duty tracking terintegrasi Google Sheets.

⏱️ Uptime         : 2j 15m 30d
🟢 Sesi Aktif     : 3
🎯 Target Harian  : 4 jam
📆 Target Mingguan: 24 jam
🔑 Admin Roles    : Pemerintah Pusat, Presiden
👮 Moderator Roles: Moderator, Kepala Menteri
```

---

## 🔑 Command Admin

> ⚠️ Hanya bisa digunakan oleh role yang terdaftar di `ADMIN_ROLES`

---

### `/setjabatan`
Mengubah jabatan seorang karyawan.

**Parameter:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `user`    | User | ✅ | Target user Discord |
| `jabatan` | Pilihan | ✅ | Jabatan baru |

**Pilihan jabatan:**
`Presiden` · `Wakil Presiden` · `Kepala Menteri` · `Menteri` · `Kepala Dept.` · `Staff` · `Magang`

**Contoh:**
```
/setjabatan user:@Budi jabatan:Staff
→ ✅ Jabatan Budi diubah menjadi Staff
```

> 📝 Aksi ini tercatat di Audit Log.

---

### `/setstatus`
Mengubah status karyawan.

**Parameter:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `user`    | User | ✅ | Target user Discord |
| `status`  | Pilihan | ✅ | Status baru |

**Pilihan status:**
| Status | Efek |
|--------|------|
| `Aktif` | Bisa duty normal |
| `Nonaktif` | Tidak bisa duty, sesi aktif otomatis diakhiri |
| `Cuti` | Tidak bisa duty |
| `Blacklist` | Diblokir total, sesi aktif otomatis diakhiri |

> ⚠️ Jika diubah ke `Nonaktif` atau `Blacklist` saat sedang duty, sesi akan otomatis diakhiri dan dicatat sebagai `Pending`.

---

### `/resetduty`
Memaksa mengakhiri sesi duty user yang stuck/lupa `/dutyout`.

**Parameter:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `user`    | User | ✅ | Target user |
| `alasan`  | String | ❌ | Alasan reset |

**Contoh:**
```
/resetduty user:@Andi alasan:Lupa dutyout
→ ✅ Sesi duty Andi telah diakhiri.
   Durasi: 3j 20m
   Alasan: Lupa dutyout
```

> 📝 Aksi ini tercatat di Audit Log dengan keterangan `RESET_DUTY`.

---

### `/daftarkaryawan`
Mendaftarkan user Discord sebagai karyawan secara manual.

> Biasanya karyawan terdaftar otomatis saat pertama kali `/dutyin`. Command ini untuk pendaftaran manual oleh admin.

**Parameter:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `user`    | User | ✅ | User yang didaftarkan |
| `jabatan` | Pilihan | ❌ | Jabatan awal (default: Magang) |

---

### `/adminreport`
Menampilkan ringkasan laporan duty seluruh karyawan minggu ini.

> 🔒 Hanya terlihat oleh diri sendiri (ephemeral)

**Output embed:**
```
📋 Laporan Duty — Admin View
Periode: 05/05/2025 — 11/05/2025

👥 Total Karyawan   : 12
✅ Memenuhi Target  : 4
🟢 Sedang Aktif     : 3
⏱️ Total Jam Minggu : 86.4j
📊 Rata-rata/Orang  : 7.2j
🔢 Total Sesi       : 48
```

---

### `/resetdata`
Mengosongkan semua data duty. **Master Karyawan dan Master Tunjangan tidak tersentuh.**

**Parameter:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `konfirmasi` | String | ✅ | Ketik `RESET` (huruf kapital) |

**Sheet yang direset:**
- ✅ Attendance
- ✅ Duty Summary
- ✅ Duty Report
- ✅ Perhitungan Tunjangan

**Sheet yang TIDAK direset:**
- ❌ Master Karyawan
- ❌ Master Tunjangan
- ❌ Audit Log

> ⚠️ **Aksi ini tidak bisa dibatalkan.** Pastikan sudah backup spreadsheet sebelum reset.

---

## 👮 Command Moderator

> Bisa digunakan oleh role di `MODERATOR_ROLES` **dan** semua role `ADMIN_ROLES`

---

### `/cekduty`
Melihat statistik duty user lain.

**Parameter:**
| Parameter | Tipe | Wajib | Keterangan |
|-----------|------|-------|------------|
| `user`    | User | ✅ | Target user |

> 🔒 Hanya terlihat oleh yang menjalankan command (ephemeral)

---

### `/aktivesesi`
Melihat semua sesi duty yang sedang aktif secara realtime.

> 🔒 Hanya terlihat oleh yang menjalankan command (ephemeral)

**Output embed:**
```
🟢 Sesi Duty Aktif — 3 orang

• Lorencio ipsum *(Menteri)*
  🕐 Masuk: 09:00:00 | ⏱️ 2j 15m 30d

• Budi Santoso *(Staff)*
  🕐 Masuk: 10:30:00 | ⏱️ 0j 45m 10d

• Andi Pratama *(Kepala Dept.)*
  🕐 Masuk: 08:00:00 | ⏱️ 3j 15m 0d
```

---

## 📊 Sheet Google Sheets

| Sheet | Fungsi | Di-update Saat |
|-------|--------|----------------|
| **Attendance** | Riwayat setiap sesi duty | `/dutyin` dan `/dutyout` |
| **Duty Summary** | Rekap total per hari per user | `/dutyout` |
| **Duty Report** | Rekap mingguan per user + status target | `/dutyout` |
| **Perhitungan Tunjangan** | Estimasi tunjangan berdasarkan jam kerja | `/dutyout` dan `/myduty` |
| **Master Karyawan** | Data semua karyawan terdaftar | `/dutyin` pertama kali, `/setjabatan`, `/setstatus` |
| **Master Tunjangan** | Rate tunjangan per jabatan | Di-seed otomatis saat bot pertama jalan |
| **Audit Log** | Jejak semua aksi penting | Setiap aksi bot |

---

### Kolom Sheet Attendance
| Kolom | Isi |
|-------|-----|
| Tanggal | DD/MM/YYYY |
| Discord ID | ID unik user |
| Display Name | Nama tampil di server |
| Waktu Masuk | HH:MM:SS |
| Waktu Keluar | HH:MM:SS |
| Durasi HH:MM:SS | Format jam:menit:detik |
| Durasi Menit | Total menit (angka) |
| Durasi Jam | Total jam desimal |
| Status | `Duty In` / `Selesai` / `Pending` / `Auto-Out` |
| Catatan | Catatan dari `/dutyout` atau sistem |

---

### Kolom Master Karyawan
| Kolom | Isi |
|-------|-----|
| Discord ID | ID unik user Discord |
| Nama | Display name saat terdaftar |
| Jabatan | Jabatan saat ini |
| Status | `Aktif` / `Nonaktif` / `Cuti` / `Blacklist` |
| Terdaftar | Tanggal pertama kali terdaftar |

---

### Kolom Audit Log
| Kolom | Isi |
|-------|-----|
| Timestamp | Waktu aksi |
| Discord ID | ID pelaku |
| Display Name | Nama pelaku |
| Aksi | Jenis aksi (lihat tabel di bawah) |
| Detail | Info tambahan |
| Target ID | ID user yang terkena aksi |
| Target Name | Nama user yang terkena aksi |
| Dieksekusi Oleh | Nama admin / `System` |

**Jenis Aksi Audit Log:**
| Aksi | Keterangan |
|------|------------|
| `DUTY_IN` | Karyawan mulai duty |
| `DUTY_OUT` | Karyawan selesai duty |
| `AUTO_OUT` | Bot force-end sesi yang melebihi batas waktu |
| `SET_JABATAN` | Admin ubah jabatan karyawan |
| `SET_STATUS` | Admin ubah status karyawan |
| `RESET_DUTY` | Admin paksa akhiri sesi |
| `RESET_DATA` | Admin reset data sheet |
| `ADD_EMP` | Karyawan baru terdaftar |

---

## 🔐 Sistem Role & Akses

### Hierarki Akses
```
ADMIN_ROLES
└── Semua command termasuk: setjabatan, setstatus,
    resetduty, daftarkaryawan, adminreport, resetdata

MODERATOR_ROLES
└── cekduty, aktivesesi

SEMUA ANGGOTA
└── dutyin, dutyout, dutystatus, myduty, topduty, botinfo

BLOCKED_ROLES
└── Tidak bisa menggunakan command apapun
```

### Konfigurasi di .env
```env
# Bisa lebih dari 1 role, pisahkan dengan koma
ADMIN_ROLES=Pemerintah Pusat, Presiden, Wakil Presiden
MODERATOR_ROLES=Moderator, Kepala Menteri
BLOCKED_ROLES=Banned, Terminated
```

---

## 🛠️ Konfigurasi .env

```env
# ── Discord ───────────────────────────────────────
TOKEN=                        # Token bot Discord
CLIENT_ID=                    # Application ID bot
GUILD_ID=                     # (Opsional) ID server untuk deploy command lebih cepat

# ── Google Sheets ─────────────────────────────────
SPREADSHEET_ID=               # ID spreadsheet Google Sheets
GOOGLE_CREDENTIALS_FILE=credentials.json

# ── Target Kerja (dalam JAM) ──────────────────────
DAILY_TARGET=4                # Target jam per hari
WEEKLY_TARGET=24              # Target jam per minggu
OVERTIME_THRESHOLD=110        # % di atas target = Overtime

# ── Role Akses ────────────────────────────────────
ADMIN_ROLES=Pemerintah Pusat
MODERATOR_ROLES=Moderator
BLOCKED_ROLES=                # Kosongkan jika tidak ada

# ── Behaviour ─────────────────────────────────────
DEFAULT_JABATAN=Magang        # Jabatan default karyawan baru
DEFAULT_STATUS=Aktif          # Status default karyawan baru
TIMEZONE=Asia/Jakarta
SESSION_FILE=./sessions.json
LEADERBOARD_SIZE=10           # Jumlah user di /topduty
MIN_SESSION_MIN=1             # Minimal durasi sesi (menit)
MAX_SESSION_HOURS=12          # Maks durasi sebelum auto-timeout
```

---

## 💰 Jabatan & Tunjangan

| Jabatan | Tunjangan Mingguan | Rate / Jam |
|---------|--------------------|------------|
| Presiden | Rp 30.000 | Rp 1.250 |
| Wakil Presiden | Rp 27.000 | Rp 1.125 |
| Kepala Menteri | Rp 20.000 | Rp 833 |
| Menteri | Rp 15.000 | Rp 625 |
| Kepala Dept. | Rp 10.000 | Rp 417 |
| Staff | Rp 7.000 | Rp 292 |
| Magang | Rp 4.000 | Rp 167 |

> Rate/Jam dihitung dari: `Tunjangan Mingguan ÷ 24 jam target`
>
> Nilai tunjangan bisa diubah langsung di sheet **Master Tunjangan**.

**Rumus perhitungan:**
```
Total Tunjangan = (Total Jam Minggu Ini) × (Rate / Jam)

Contoh Menteri dengan 6.3 jam:
6.3 × 625 = Rp 3.938
```

---

## 🏷️ Status Karyawan

| Status | Warna | Bisa Duty | Keterangan |
|--------|-------|-----------|------------|
| 🟢 Aktif | Hijau | ✅ Ya | Normal |
| 🔴 Nonaktif | Merah | ❌ Tidak | Dinonaktifkan admin |
| 🟡 Cuti | Kuning | ❌ Tidak | Sedang cuti |
| ⛔ Blacklist | Hitam | ❌ Tidak | Diblokir permanen |

---

## ⚠️ Catatan Penting

### Auto-Timeout
Bot secara otomatis mengakhiri sesi yang berjalan lebih dari `MAX_SESSION_HOURS` (default: 12 jam).
- Sesi dicatat sebagai `Pending` di sheet
- User mendapat DM notifikasi
- Tercatat di Audit Log sebagai `AUTO_OUT`

### Sesi Bertahan Setelah Restart
Sesi aktif disimpan di file `sessions.json` — bot restart tidak akan menghilangkan sesi yang sedang berjalan.

### Karyawan Baru
User yang pertama kali `/dutyin` akan otomatis terdaftar di **Master Karyawan** dengan jabatan `Magang` dan status `Aktif`. Admin bisa ubah jabatan via `/setjabatan`.

### Format Tanggal
Semua tanggal menggunakan format `DD/MM/YYYY` dengan timezone `Asia/Jakarta` (bisa diubah via `TIMEZONE` di `.env`).

### Minggu Dihitung Senin–Minggu
Target mingguan dihitung dari **Senin 00:00** hingga **Minggu 23:59**. Reset otomatis setiap awal minggu.

---

*Duty Tracker Bot v2.0 — Built with discord.js + Google Sheets API*
