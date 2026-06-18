# 📘 Panduan Penggunaan Nodeev Panel

Panduan lengkap berbahasa Indonesia untuk **Nodeev Panel** — panel kontrol web
untuk server FiveM yang membuat owner/developer **tidak perlu lagi menyentuh
SSH**. Dokumen ini menjelaskan setiap menu, fungsinya, siapa yang boleh memakai,
dan cara menggunakannya secara detail.

> Versi panel: 1.0.0 · Disusun: 2026-06-17

---

## Daftar Isi

1. [Pengenalan Singkat](#1-pengenalan-singkat)
2. [Konsep Peran: Owner vs Admin](#2-konsep-peran-owner-vs-admin)
3. [Cara Mengakses & Login](#3-cara-mengakses--login)
4. [Mengganti Password & Mengelола Akun](#4-mengganti-password--mengelola-akun)
5. [Penjelasan Setiap Menu/Modul](#5-penjelasan-setiap-menumodul)
   - [5.1 Dashboard](#51-dashboard)
   - [5.2 Kontrol Server (Start/Stop/Restart)](#52-kontrol-server-startstoprestart)
   - [5.3 Console (Konsol Langsung)](#53-console-konsol-langsung)
   - [5.4 Monitor (Pemantauan Sistem)](#54-monitor-pemantauan-sistem)
   - [5.5 Resources (Manajemen Resource)](#55-resources-manajemen-resource)
   - [5.6 Assets — Peds & Vehicles](#56-assets--peds--vehicles)
   - [5.7 File Manager (Penjelajah File)](#57-file-manager-penjelajah-file)
   - [5.8 Database Manager (In-Game) ⭐](#58-database-manager-in-game-)
   - [5.9 Server Configuration (server.cfg)](#59-server-configuration-servercfg)
   - [5.10 Backups (Pencadangan)](#510-backups-pencadangan)
   - [5.11 Update Center](#511-update-center)
   - [5.12 Owners (Manajemen Akun)](#512-owners-manajemen-akun)
   - [5.13 Audit Log (Log Rahasia)](#513-audit-log-log-rahasia)
   - [5.14 Notifications](#514-notifications)
6. [Keamanan yang Berjalan di Belakang Layar](#6-keamanan-yang-berjalan-di-belakang-layar)
7. [Mengelola Lewat Terminal (Opsional)](#7-mengelola-lewat-terminal-opsional)
8. [Troubleshooting (Pemecahan Masalah)](#8-troubleshooting-pemecahan-masalah)
9. [FAQ](#9-faq)
10. [Lampiran: Prompt untuk Deploy via Claude](#10-lampiran-prompt-untuk-deploy-via-claude)

---

## 1. Pengenalan Singkat

Nodeev Panel mengubah satu VPS Ubuntu menjadi host FiveM yang terkelola penuh
lewat antarmuka web. Dari panel kamu bisa: menyalakan/mematikan server, melihat
konsol & metrik real-time, mengelola resource, mengedit file & `server.cfg`,
mencadangkan, memperbarui FXServer, **mengelola data pemain in-game tanpa
aplikasi MySQL**, dan mengatur akun staf — semua dengan audit keamanan otomatis.

**Tumpukan teknologi:** Node.js · Express + WebSocket · MariaDB (user khusus
`fivem_panel`, bukan root) · nginx (reverse proxy TLS) · systemd · UFW ·
fail2ban.

---

## 2. Konsep Peran: Owner vs Admin

Nodeev Panel punya **dua peran**. Memahami ini penting karena menentukan menu apa
yang terlihat.

| Hal | 👑 Owner | 🛠️ Admin (Developer) |
|---|---|---|
| Kontrol server (start/stop/restart) | ✅ | ✅ |
| Console, Monitor, Resources, Assets | ✅ | ✅ |
| File Manager (seluruh `/opt/fivem`) | ✅ | ✅ |
| **Database Manager** (data in-game) | ✅ | ✅ |
| Server Configuration, Backups, Updates | ✅ | ✅ |
| **Owners (manajemen akun)** | ✅ **eksklusif** | ❌ |
| **Audit Log (log rahasia)** | ✅ **eksklusif** | ❌ |

**Inti aturannya:**
- **Owner** = pemilik. Kekuasaan *eksklusif*-nya **HANYA dua**: mengelola akun
  (Owners) dan melihat **Audit Log**. Selebihnya sama seperti admin.
- **Admin** = developer/staf. Menguasai seluruh ekosistem FiveM, **kecuali**
  manajemen akun dan audit log.
- **Setiap tindakan admin yang mengubah data** (edit file, clear cache, ubah
  uang/job pemain, dll) **diam-diam dicatat** ke log rahasia yang hanya bisa
  dilihat owner. Admin tidak bisa melihat atau menghapus catatan ini.

> Akun pertama yang dibuat installer adalah **owner**. Owner lalu membuat akun
> admin untuk staf lewat menu **Owners**.

---

## 3. Cara Mengakses & Login

1. Buka URL panel di browser:
   - **Mode Localhost/Testing:** `https://localhost` (sertifikat self-signed →
     klik "lanjutkan" pada peringatan browser).
   - **Mode Domain (Let's Encrypt):** `https://domain-kamu.com` (sertifikat
     tepercaya, tanpa peringatan).
   - **Mode Cloudflare/BYO:** `https://domain-atau-ip` (di belakang Cloudflare;
     set SSL/TLS Cloudflare ke **Full**).
2. Masukkan **username** dan **password** owner. Kredensial awal dicetak di akhir
   instalasi dan tersimpan di server:
   ```bash
   sudo cat /root/.fivem-panel-credentials
   ```
3. Centang **"Remember this device for 30 days"** bila ingin sesi bertahan lama
   di perangkat tepercaya.

> **Penting soal kredensial:** username & password awal **dibuat acak**. Kalau
> login gagal padahal password "benar", kemungkinan besar **username**-nya yang
> salah ketik (bukan password). Cek username asli di file kredensial di atas, atau
> reset lewat menu **Owners** / perintah di [Troubleshooting](#8-troubleshooting-pemecahan-masalah).

---

## 4. Mengganti Password & Mengelola Akun

- **Ganti password sendiri:** masuk ke menu **Owners**, pilih akunmu, gunakan opsi
  reset/ubah password. (Jika installer menandai *must change password*, panel akan
  memintamu menggantinya saat login pertama.)
- **Membuat akun Admin (staf):** Owner → **Owners** → *Tambah akun* → tentukan
  username, role = **admin**. Berikan kredensialnya ke staf.
- **Reset password staf:** Owner → **Owners** → pilih akun → reset password.
- **Reset hitungan login gagal:** di **Owners** ada tag "N failed logins" dan
  tombol **Reset failed logins** bila sebuah akun terlihat sering gagal login.

---

## 5. Penjelasan Setiap Menu/Modul

Setiap modul di bawah ini muncul di **sidebar** panel. Tanda 👑 = khusus owner.

### 5.1 Dashboard

Halaman ringkasan saat pertama login. Menampilkan **status server** (online/
offline), ringkasan metrik (CPU/RAM), build FXServer, dan **aksi cepat**
start/stop/restart. Gunakan ini sebagai "kokpit" untuk pandangan sekilas.

### 5.2 Kontrol Server (Start/Stop/Restart)

Mengendalikan service FiveM (`fivem.service`) langsung dari panel — tanpa SSH.

- **Start** — menyalakan server.
- **Stop** — mematikan server.
- **Restart** — mematikan lalu menyalakan kembali.

> 🧹 **Auto-clear cache:** setiap **start/restart** dari panel, folder
> `/opt/fivem/server/cache` **dibersihkan otomatis** lebih dulu (FXServer
> membangunnya ulang). `server.cfg` dan resource **tidak** disentuh. Ini mencegah
> banyak masalah "resource korup setelah update".

### 5.3 Console (Konsol Langsung)

Konsol FiveM **real-time** lewat WebSocket. Dua fungsi:
- **Melihat output** server secara langsung (log, koneksi pemain, error).
- **Mengirim perintah** ke server (mis. `refresh`, `ensure namaresource`,
  `say Halo`, perintah admin script-mu) melalui FIFO stdin server.

> Perintah hanya berfungsi saat **server menyala** (FIFO hanya ada ketika service
> hidup). Jika perintah tidak bereaksi, pastikan server **Start** dulu.

### 5.4 Monitor (Pemantauan Sistem)

Grafik & angka real-time: **CPU, RAM, disk, jaringan**, plus status proses
FXServer. Dilengkapi **peringatan ambang batas** (alert) bila pemakaian terlalu
tinggi. Berguna untuk mendeteksi resource boros atau kebocoran memori.

### 5.5 Resources (Manajemen Resource)

Mengelola resource FiveM di `/opt/fivem/resources`:
- **Daftar resource** beserta status (aktif/non-aktif).
- **Start / Stop / Restart / Ensure** resource tertentu tanpa restart seluruh
  server.
- **Menjelajah file** di dalam resource dan mengeditnya (terintegrasi editor).

Cocok untuk menyalakan/mematikan script tertentu atau memperbaiki satu resource.

### 5.6 Assets — Peds & Vehicles

Manajer **ped** dan **kendaraan** kustom:
- **Import** aset (mis. mobil/ped tambahan).
- **Kategorikan** dan **aktifkan/non-aktifkan**.
- **Edit metadata** aset.

Panel menyimpan metadata per-aset di berkas sidecar (`.helm.json`) — biarкан nama
berkas ini apa adanya agar aset yang sudah ada tetap dikenali.

### 5.7 File Manager (Penjelajah File)

Penjelajah file penuh yang **dikurung (sandbox) ke `/opt/fivem`**. Kamu bisa:
- **Browse, buat, ubah nama, hapus, unggah** file di mana saja di bawah
  `/opt/fivem` — termasuk `server/`, `resources/`, dan `server.cfg`.
- **Edit file teks** di editor CodeMirror dalam browser (highlight Lua/JS/JSON/
  SQL/cfg, simpan dengan **Ctrl-S**).
- **Clear Cache** — tombol aksi cepat yang **hanya** menghapus
  `/opt/fivem/server/cache` (dibangun ulang FXServer saat start berikutnya).

> 🔒 **Jail keras:** setiap path dikurung ke `/opt/fivem` secara leksikal **dan**
> lewat pengecekan `realpath` (sadar symlink). Upaya keluar (`../`, path absolut,
> symlink jebakan) **ditolak** — jadi file OS host (`/etc/nginx`, `/root`,
> `/opt/fivem-panel`) **tidak bisa** dijangkau dari panel.

### 5.8 Database Manager (In-Game) ⭐

**Fitur unggulan** — kelola data pemain langsung dari panel **tanpa aplikasi
MySQL**. Panel terhubung ke database framework FiveM-mu dan **mengenali
frameworknya secara otomatis** dengan membaca skema database (fingerprinting).

#### Framework yang didukung (deteksi otomatis)
| Keluarga | Generasi terdeteksi | Istilah identitas |
|---|---|---|
| **ESX** | **ESX Legacy** (uang di kolom `accounts` JSON) & **ESX v1** (multi-tabel `user_accounts`) | Identifier |
| **QBCore** | QBCore | CitizenID |
| **Qbox** | Qbox (QBCore + stack ox) | CitizenID |
| **ox_core** | ox_core | StateID |

- **Deteksi murni dari skema** — tidak menebak, tidak ada "raw SQL editor".
  Versi/generasi (mis. ESX Legacy vs ESX v1) dideteksi otomatis.
- **Pin saat instalasi (opsional):** kamu bisa memilih framework saat install
  (tersimpan sebagai `GAME_FRAMEWORK` di `.env`) atau biarkan **Auto-Detect**.
- **UI Shapeshifting:** tampilan **berubah wujud** mengikuti framework —
  istilah (Identifier/CitizenID/StateID), label, dan kolom uang menyesuaikan
  otomatis. Badge di pojok menunjukkan framework + versi yang terdeteksi.

#### Cara memakai
1. Buka **Database Manager**. Lihat badge framework (mis. "ESX Legacy").
2. **Cari pemain** berdasarkan nama atau identifier/CitizenID/StateID.
3. Klik **Manage** pada pemain untuk membuka detailnya:
   - **💰 Money** — ubah **Cash** dan **Bank**. Key lain dalam JSON (mis.
     `black_money`, `crypto`) **tetap aman** (tidak terhapus).
   - **👔 Job / Group** — set nama job & grade (ESX: `job`/`job_grade`; QBCore:
     JSON `job`; ox_core: keanggotaan **group** + grade).
   - **🚗 Vehicles** — lihat & **hapus** kendaraan milik pemain.
4. Setiap perubahan **tercatat di Audit Log** (untuk owner).

#### Catatan per-framework
- **ox_core:** **Bank** bisa diedit (`accounts.balance`), tapi **Cash bersifat
  read-only** di panel — karena di ox_core cash adalah *item* di `ox_inventory`,
  dan mengeditnya dari luar berisiko merusak inventory. Section yang tak didukung
  skema (mis. vehicles) **otomatis disembunyikan**.
- **Belum terdeteksi?** Jika `es_extended` masih kosong, import dulu SQL
  framework-mu (yang membuat tabel `users`/`players`/`characters`), lalu buka
  ulang modulnya atau Re-detect.

#### Keamanan modul ini
Semua query **terparameter** (anti SQL-injection); kolom JSON diedit dengan pola
*read-modify-write* di dalam transaksi (`SELECT … FOR UPDATE`); nilai uang dibatasi
ke bilangan bulat non-negatif. Panel hanya punya hak **DML** pada schema game (tak
bisa `DROP/ALTER/CREATE`).

### 5.9 Server Configuration (server.cfg)

Editor `server.cfg` yang aman:
- **Mode GUI** — ubah pengaturan umum lewat form.
- **Urutan muat (load-order)** resource.
- **Mode mentah (raw)** — edit teks penuh.
- **License Key** — tempel `sv_licenseKey` (`cfxk_...`) lalu **Save & Restart**.
  Server FiveM **tidak menerima pemain** sampai license diisi (default `CHANGEME`).
  Dapatkan key gratis di <https://keymaster.fivem.net>.

> Panel memakai blok **PANEL-MANAGED** di dalam `server.cfg` dan menanganinya
> secara *injection-safe* — bagian yang kamu tulis manual tetap dipertahankan.

### 5.10 Backups (Pencadangan)

- **Buat backup** — arsip `tar.gz` (file kota) **+ dump database**.
- **Jadwalкан** backup berkala.
- **Restore** dari backup yang ada.

Backup tersimpan di `/opt/fivem/backups/`. Pantau ruang disk bila banyak backup.

### 5.11 Update Center

Memeriksa & memasang **artifact FXServer** terbaru (recommended/latest). Setelah
update, ingat cache otomatis dibersihkan saat restart berikutnya.

### 5.12 Owners (Manajemen Akun) 👑

**Khusus Owner.** Pusat manajemen akun staf:
- **Buat** akun baru (role owner/admin).
- **Reset password** akun mana pun.
- **Hapus** akun.
- Lihat tag **"N failed logins"** + tombol **Reset failed logins**.

> Admin **tidak** melihat menu ini; akses API-nya pun ditolak (403/404).

### 5.13 Audit Log (Log Rahasia) 👑

**Khusus Owner.** Menampilkan **`secret_audit_logs`** — jejak diam-diam dari
**setiap tindakan admin yang berhasil mengubah sesuatu**: siapa (who), dari IP
mana, aksi apa, dan target-nya (mis. `id`, `plate`, `cash`, `job`). Termasuk edit
file, clear cache, dan perubahan uang/job/vehicle.

- **Tindakan owner TIDAK dicatat** — hanya admin yang diaudit (pemisahan tugas).
- **Tidak ada isi file/password** yang bocor ke log (hanya field identitas yang
  di-whitelist).
- Admin **tidak bisa** melihat atau mengubahnya.

Gunakan ini untuk mengawasi apa yang dilakukan staf di server.

### 5.14 Notifications

Pemberitahuan sistem panel (mis. peringatan, hasil aksi). Tempat memeriksa
notifikasi yang dihasilkan modul-modul lain.

---

## 6. Keamanan yang Berjalan di Belakang Layar

Kamu tidak perlu mengatur ini — tapi baik untuk tahu panel sudah mengamankan:

- **TLS/HTTPS** via nginx (Let's Encrypt, origin cert Cloudflare, atau
  self-signed).
- **Login anti brute-force berbasis IP** — akun **tidak pernah dikunci** (agar
  orang tak bisa mengunci kamu dengan menebak username). Sebaliknya, **IP/jaringan
  asing** diblokir sementara setelah 3 kegagalan, dan **otomatis pulih ~30 menit**.
  IP yang pernah sukses login ("tepercaya") tidak diblokir.
- **Shadow Audit** — tindakan admin tercatat ke log rahasia owner (lihat
  [5.13](#513-audit-log-log-rahasia)).
- **Sandbox File Manager** — terkurung ke `/opt/fivem` (anti path-traversal &
  symlink).
- **Hak DB terbatas** — user panel hanya DML pada schema game (tak bisa merusak
  struktur).
- **fail2ban + UFW** — menjaga SSH dan endpoint login.
- **Kredensial DB & sesi** tersimpan di `.env` mode 600 (bukan root).

---

## 7. Mengelola Lewat Terminal (Opsional)

Biasanya tak perlu, tapi tersedia:

```bash
# Server game FiveM
sudo systemctl status|start|stop|restart fivem

# Panel web
sudo systemctl status|start|stop|restart fivem-panel

# Konsol FiveM langsung
sudo journalctl -u fivem -f

# Log panel langsung
sudo journalctl -u fivem-panel -f

# Lihat kredensial owner
sudo cat /root/.fivem-panel-credentials

# Set license key manual
sudo nano /opt/fivem/server/server.cfg   # set sv_licenseKey "cfxk_..."
sudo systemctl restart fivem
```

---

## 8. Troubleshooting (Pemecahan Masalah)

| Masalah | Solusi |
|---|---|
| **Tidak bisa login padahal password benar** | Kemungkinan **username** salah (username awal acak). Cek `sudo cat /root/.fivem-panel-credentials`, atau reset password lewat menu **Owners**. Untuk reset darurat lihat baris di bawah tabel. |
| **Panel tidak terbuka dari internet** | Di VPS cloud (Azure/AWS/GCP), **firewall cloud** (NSG/Security Group) memblokir 80/443 meski UFW sudah allow. Buka inbound **80 & 443** di panel cloud provider-mu. |
| **Peringatan keamanan browser** | Wajar untuk sertifikat self-signed (testing/IP). Pakai domain asli untuk sertifikat Let's Encrypt tepercaya. |
| **FiveM tetap offline** | Hampir selalu **license key** belum diisi. Set di *Server Configuration → License Key*, lalu restart. Cek `journalctl -u fivem -e`. |
| **Pemain tak bisa connect** | Pastikan UFW & firewall cloud mengizinkan `30120/tcp` **dan** `30120/udp`. |
| **Perintah console tak bereaksi** | Server harus **menyala** (FIFO hanya ada saat service hidup). Start dulu. |
| **Database Manager: "not configured"** | `GAME_DB_NAME` kosong di `.env`. Jalankan ulang installer, atau set manual + grant + restart (lihat README). |
| **Database Manager: framework tak terdeteksi** | Schema ada tapi belum ada tabel framework. **Import SQL** ESX/QBCore/ox-mu, lalu Re-detect. |
| **Disk penuh** | Backup lama di `/opt/fivem/backups/` & log. `ncdu /opt/fivem` untuk memeriksa. |

**Reset password owner darurat (lewat SSH):** cara termudah lewat menu **Owners**.
Jika benar-benar terkunci, minta bantuan teknis untuk meng-update hash bcrypt di
tabel `owners`.

---

## 9. FAQ

**T: Apakah saya perlu tahu Linux/SSH untuk pakai panel ini?**
J: Tidak. Tujuannya **zero-touch** — semua lewat web. Terminal hanya untuk kasus
darurat.

**T: Bedanya Owner dan Admin?**
J: Owner punya 2 kuasa ekstra: **Owners** (akun) & **Audit Log**. Sisanya sama.
Admin menguasai seluruh operasional FiveM dan **diaudit diam-diam**.

**T: Database Manager-ku ESX/QBCore/Qbox/ox_core — apakah didukung?**
J: Ya, keempatnya, dengan deteksi otomatis + tampilan yang menyesuaikan. ESX
bahkan dibedakan Legacy vs v1 multi-tabel.

**T: Apakah aman mengedit uang pemain dari panel?**
J: Ya — query terparameter, JSON diedit aman dalam transaksi (key lain seperti
`black_money`/`crypto` dipertahankan), dan setiap perubahan teraudit.

**T: Server saya di Azure, panel tak terbuka dari luar — kenapa?**
J: Firewall cloud (NSG) memblokir 80/443. Buka inbound 80/443 di Azure Portal
(VM → Networking → Add inbound port rule). UFW di dalam box saja tidak cukup.

**T: Bagaimana cara melihat panel kalau port cloud belum dibuka?**
J: Pakai **SSH tunnel**: `ssh -L 8443:127.0.0.1:443 user@ip -N`, lalu buka
`https://localhost:8443`.

---

## 10. Lampiran: Prompt untuk Deploy via Claude

Saat menyiapkan VPS **baru** dan ingin Claude yang mengeksekusi instalasi
(seperti alur yang sudah terbukti), buka Claude Code lalu tempel prompt ini —
isi bagian `<...>`:

```text
Kamu Claude Code di Windows-ku. Tugas: setup & deploy "Nodeev Panel" (FiveM
control panel) ke VPS baruku, end-to-end, lewat SSH dari Windows ini — seperti
yang sudah berhasil kita lakukan sebelumnya. (Baca memori: remote-vps-deploy-
playbook & remote-vps-claude-driven-setup.)

DATA VPS:
- IP        : <isi>
- User      : <isi>   (punya sudo)
- Auth      : <password ATAU "bantu aku set SSH key dulu">

FILE YANG SUDAH KUSIAPKAN:
- Project panel di: <path Windows, mis. C:\Users\lynod\project>
- Backup kota FiveM di: <path, atau "belum">

PILIHAN INSTALASI:
- SSL      : <[1] Domain+Let's Encrypt / [2] Cloudflare-BYO / [3] Localhost>
- Domain   : <isi jika [1]/[2]>
- Framework: <auto / esx / qbcore / qbox / ox>
- GAME_DB  : <es_extended / qbcore / none>

YANG KUMINTA (konfirmasi rencana dulu sebelum langkah destruktif):
1. Konek via Posh-SSH; tes read-only dulu (OS, RAM, disk, sudo, cek
   node/mariadb/nginx) — pastikan ini Ubuntu yang sesuai.
2. Transfer project ke VPS (zip TANPA node_modules → SCP → unzip di ~/project).
3. Jalankan master_setup.sh dengan pilihan di atas. (Kalau prompt /dev/tty
   bermasalah di headless, pakai trik patch stdin sesuai playbook.)
4. Verifikasi: .env (GAME_FRAMEWORK), service aktif, nginx, panel HTTP 200,
   deteksi Database Manager.
5. Kalau VPS cloud (Azure/AWS): ingatkan aku buka inbound 80/443 di NSG/Security
   Group — kamu tak bisa melakukannya sendiri.
6. Kalau backup kota sudah kusiapkan: bantu import ke database & resources.
7. Laporkan hasil apa adanya. Ini boleh "rusak" kalau perlu (uji coba).

Mulai dari langkah 1.
```

> Tips keamanan: untuk server **produksi**, gunakan **SSH key** (bukan password di
> chat), dan jangan buka 80/443 ke "Any" — batasi ke IP-mu atau pakai Cloudflare.

---

*Dokumen ini bagian dari proyek Nodeev Panel. Untuk detail teknis/arsitektur,
lihat `README.md` dan `ARCHITECTURE.md`.*
