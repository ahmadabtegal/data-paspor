# Monitoring Pengambilan Paspor
Paket awal aplikasi: frontend HTML untuk GitHub Pages, backend Google Apps Script, database Google Sheets.

## Isi paket
- `Code.gs` — API backend, setup sheet, CRUD dasar, log WhatsApp, pengiriman otomatis/manual, pengingat harian opsional.
- `index.html` — dashboard statistik, pencarian/filter, input/edit, status pengambilan, riwayat notifikasi, tombol WhatsApp manual.
- `appsscript.json` — konfigurasi runtime Apps Script.

## A. Siapkan Google Sheets dan Apps Script
1. Buat Google Spreadsheet baru.
2. Buka **Extensions > Apps Script**.
3. Tempel isi `Code.gs` ke file `Code.gs`.
4. Di Apps Script, pilih fungsi `setupSheets`, lalu klik **Run** dan setujui izin.
5. Pastikan sheet `DATA_PASPOR`, `LOG_NOTIFIKASI`, dan `PENGATURAN` dibuat.
6. Untuk backend, buka **Deploy > New deployment > Web app**.
7. Pilih **Execute as: Me**. Atur akses sesuai kebijakan organisasi. Jika frontend GitHub Pages perlu akses tanpa login, deployment mungkin perlu diatur agar dapat diakses oleh pengguna target; jangan gunakan data nyata sebelum keamanan dan kewenangan akses ditinjau.
8. Klik Deploy, salin URL Web App yang berakhiran `/exec`.

## B. Pasang frontend di GitHub Pages
1. Buat repository GitHub, misalnya `monitoring-paspor`.
2. Upload `index.html`.
3. Edit baris berikut di bagian script `index.html`:
   ```js
   const API_URL = 'PASTE_URL_WEB_APP_APPS_SCRIPT_DI_SINI';
   ```
   Ganti dengan URL `/exec` dari deployment Apps Script.
4. Aktifkan **Settings > Pages**, pilih branch dan folder root.
5. Buka URL GitHub Pages dan uji koneksi.

## C. Penting tentang CORS dan akses
Google Apps Script ContentService mengembalikan respons melalui redirect. Jika browser menolak permintaan lintas domain, cara paling stabil adalah menyajikan frontend melalui Apps Script, atau memakai backend/proxy yang mendukung CORS. Uji dari URL GitHub Pages sebelum memasukkan data produksi. Jangan memakai `mode: no-cors`, karena respons tidak dapat dibaca oleh frontend.
Jika ada kendala CORS, solusi sederhana yang paling andal adalah menempatkan HTML pada Apps Script HTML Service, atau membuat endpoint backend/proxy yang dikonfigurasi CORS. Jangan menganggap deployment berhasil hanya karena URL bisa dibuka langsung.

## D. Konfigurasi WhatsApp otomatis (opsional)
Integrasi menggunakan **Fonnte API**. Untuk mengaktifkan:
1. Pastikan akun Fonnte aktif dan perangkat WhatsApp sudah terhubung sesuai panduan Fonnte.
2. Apps Script > **Project Settings > Script Properties**, tambahkan `FONNTE_TOKEN` dengan token API dari perangkat Fonnte. Jangan masukkan token ke HTML atau GitHub.
3. Di sheet `PENGATURAN`, ubah `WA_ENABLED` menjadi `YA`. Ubah `AUTO_SEND` menjadi `YA` jika ingin mencoba pengiriman otomatis saat data baru dibuat.
4. Uji menggunakan nomor penerima yang menyetujui menerima notifikasi, lalu periksa `LOG_NOTIFIKASI` dan riwayat pesan di Fonnte.
5. Tombol `Kirim WA` mengirim melalui Fonnte; tombol ini bukan sekadar membuka WhatsApp Web.

### Status pengiriman
Aplikasi mencatat apakah permintaan diterima oleh API Fonnte. Status tersebut tidak menjamin pesan telah dibaca penerima; periksa riwayat/status pesan di dashboard Fonnte jika perlu.

### Perilaku mode gabungan
- Saat data baru disimpan, backend mencoba pengiriman API otomatis jika `AUTO_SEND=YA`.
- Jika Fonnte belum aktif atau pengiriman gagal, data tetap tersimpan dan hasil dicatat sebagai gagal/konfigurasi belum lengkap.
- Tombol `Kirim WA` mengirim melalui Fonnte setelah diklik oleh petugas.
- Log mencatat respons API Fonnte; respons diterima bukan bukti bahwa pengguna telah membaca pesan.

## E. Jam layanan dan template pesan
Edit sheet `PENGATURAN`:
- `NAMA_KANTOR`
- `JAM_LAYANAN`
- `ALAMAT_LOKET`
- `TEMPLATE_PESAN`
- `AUTO_SEND`
- `WA_ENABLED`
- `PENGINGAT_HARI`

Placeholder template yang tersedia: `{NAMA}`, `{ALAMAT}`, `{JAM}`, `{KANTOR}`.

Pastikan jam layanan dan alamat sesuai pengumuman resmi kantor imigrasi. Jam layanan yang tertulis di contoh hanyalah nilai awal.

## F. Pengingat harian (opsional)
Jalankan `createDailyReminderTrigger()` satu kali untuk membuat trigger harian pukul sekitar 09.00 zona waktu proyek. Fungsi `dailyReminderCheck()` memeriksa paspor yang belum diambil dan melewati ambang `PENGINGAT_HARI`. Untuk menghindari pesan yang melanggar kebijakan, pastikan pengiriman proaktif menggunakan template WhatsApp yang disetujui sebelum mengaktifkan fitur ini.

## G. Struktur data
### DATA_PASPOR
`ID_Paspor, Tanggal_Input, Nama_Pemohon, Nomor_WA, Nomor_Permohonan, Tanggal_Paspor_Jadi, Status_Pengambilan, Tanggal_Diambil, Tanggal_WA_Terakhir, Jumlah_Notifikasi, Status_Notifikasi, Catatan, Petugas_Input`

### LOG_NOTIFIKASI
`ID_Log, ID_Paspor, Tanggal_Kirim, Nomor_WA, Isi_Pesan, Metode_Kirim, Hasil_Kirim, Keterangan_Error, Petugas`

### PENGATURAN
`Kunci, Nilai, Keterangan`

## H. Pemeriksaan sebelum digunakan
- Uji simpan, edit, filter, perubahan status, dan log.
- Uji nomor WhatsApp dengan kode negara Indonesia (misalnya `0812...` otomatis dinormalisasi menjadi `62812...`).
- Uji auto-send hanya ke nomor uji yang sudah disetujui.
- Batasi akses spreadsheet dan backend; jangan menyimpan token API di HTML/GitHub.
- Gunakan data minimal yang diperlukan dan pastikan petugas berwenang mengakses data pemohon.
- Buat salinan cadangan spreadsheet secara berkala.
