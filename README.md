# Sistem Manajemen Aset POLNES

Sistem manajemen aset berbasis web untuk Politeknik Negeri Samarinda (POLNES) yang memungkinkan pengelolaan, peminjaman, dan pengembalian aset secara digital.

## Fitur Utama

- 📦 Manajemen Aset
  - Tambah, edit, dan hapus aset
  - Kategorisasi aset
  - Status ketersediaan real-time
  - Riwayat penggunaan

- 📝 Peminjaman Aset
  - Form peminjaman digital
  - Validasi otomatis
  - Notifikasi status
  - Riwayat peminjaman

- 🔄 Pengembalian Aset
  - Proses pengembalian digital
  - Pencatatan kondisi aset
  - Update status otomatis
  - Riwayat pengembalian

- 📊 Dashboard & Laporan
  - Statistik penggunaan
  - Grafik ketersediaan
  - Laporan peminjaman
  - Export data

## Teknologi

- Frontend: HTML5, CSS3, JavaScript (Vanilla)
- Backend: Firebase Realtime Database
- Authentication: Firebase Auth
- Storage: Firebase Storage
- Hosting: Firebase Hosting

## Struktur Proyek

```
/
├── index.html          # Halaman utama aplikasi
├── css/               # File stylesheet
│   ├── style.css      # Style utama
│   └── components/    # Style komponen
├── js/                # File JavaScript
│   ├── app.js         # Logika aplikasi utama
│   ├── auth.js        # Autentikasi
│   ├── database.js    # Operasi database
│   └── utils/         # Fungsi utilitas
└── assets/            # Gambar dan aset statis
```

## Instalasi

1. Clone repositori:
   ```bash
   git clone https://github.com/username/sistem-manajemen-aset-polnes.git
   ```

2. Buka `index.html` di browser modern

3. Atau deploy ke Firebase:
   ```bash
   firebase init
   firebase deploy
   ```

## Penggunaan

1. Buka aplikasi di browser
2. Login dengan akun POLNES
3. Akses fitur sesuai kebutuhan:
   - Dashboard: Lihat statistik dan status aset
   - Aset: Kelola daftar aset
   - Peminjaman: Proses peminjaman baru
   - Riwayat: Lihat history penggunaan

## Kontribusi

1. Fork repositori
2. Buat branch fitur (`git checkout -b fitur-baru`)
3. Commit perubahan (`git commit -m 'Tambah fitur baru'`)
4. Push ke branch (`git push origin fitur-baru`)
5. Buat Pull Request

## Lisensi

Proyek ini dilisensikan di bawah [MIT License](LICENSE).

## Kontak

Tim Pengembang POLNES
- Email: dev@polnes.ac.id
- Website: https://polnes.ac.id
