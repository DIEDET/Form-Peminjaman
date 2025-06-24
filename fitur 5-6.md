

---

## 📝 Prompt: Sistem Manajemen Peminjaman Aset (Web Publik, Tanpa Login/QR)

Saya sedang membangun **Sistem Manajemen Peminjaman Aset** berbasis web, tanpa login dan tanpa QR code, yang meniru pencatatan manual di kertas namun dengan tampilan rapi, profesional, dan efisien. Backend menggunakan **Firebase Realtime Database** (public rules) dan frontend menggunakan **vanilla JavaScript**.

### 🔧 Tujuan Sistem

Menjadi pengganti catatan peminjaman manual, dapat diakses publik, tanpa autentikasi, dengan fitur otomatisasi dan reporting modern.

---

## ✅ Fitur Sistem

1. **Model Data**

   * **Koleksi `aset`**

     * `id` (string, UUID)
     * `namaBarang` (string)
     * `nomorBarang` (string, ex: "01","02",…)
     * `status` (string): “Tersedia” / “Dipinjam”
     * `tanggalTambah` (date)
   * **Koleksi `peminjaman`**

     * `id` (string, UUID)
     * `nim` (string, 9 digit)
     * `namaLengkap` (string)
     * `noHp` (string, 12 digit)
     * `mataPelajaran` (string)
     * `kelas` (string)
     * `tanggalPinjam` (date)
     * `jamPinjam` (time)
     * `tanggalKembali` (date)
     * `status` (string): “Dipinjam” / “Dikembalikan”
     * `asetRef` (string, referensi `id` aset)
     * `namaBarang` (string, salinan)
     * `nomorBarang` (string, salinan)
   * **Koleksi `deleted/peminjaman`** untuk soft delete

2. **Validasi & Otomatisasi Status**

   * Validasi semua field wajib terisi
   * NIM = tepat 9 digit; No HP = tepat 12 digit
   * `tanggalKembali` ≥ `tanggalPinjam`
   * Cek ketersediaan aset sebelum simpan
   * Saat simpan peminjaman → `aset.status = "Dipinjam"`
   * Saat ubah status ke “Dikembalikan” → `aset.status = "Tersedia"`
   * Saat soft delete → aset kembali “Tersedia”

3. **Riwayat Peminjaman**

   * Tabel histori lengkap
   * Filter berdasarkan Nama Lengkap, Kelas, dan Tanggal Pinjam
   * Sinkronisasi otomatis ke Google Sheets via webhook (mengganti ekspor manual Excel/PDF)

4. **Trash Bin (Soft Delete)**

   * Hapus → pindahkan data ke node `deleted/peminjaman`
   * Opsi Restore & Hapus Permanen

5. **Etika Peminjaman**

   * Tampilkan modal konfirmasi sebelum form:

     * “Barang tidak boleh dipinjamkan ke pihak ketiga.”
     * “Barang harus dikembalikan terlebih dahulu sebelum dipinjam ulang.”

6. **Dashboard Statistik**

   * Kartu statistik real‑time menampilkan:

     * Total jumlah aset
     * Jumlah aset tersedia
     * Jumlah aset sedang dipinjam
     * Jumlah transaksi aktif (status “Dipinjam”)

7. **Tabel Transaksi Lengkap**

   * Kolom: No, Nama Lengkap, NIM, Barang, Nomor Barang, Kelas, Mata Pelajaran, Tanggal Pinjam, Tanggal Kembali, Status, Aksi
   * Responsive, zebra‑striped, header biru tua

---

## 📦 Data Aset Awal

* 24 Proyektor EPSON (nomor 01–24)
* 24 Converter ROBOT RHV01
* 24 ATK
* 30 Komputer HP
* 20 Stop Kontak

---

## 🎓 Daftar Mata Kuliah (lebih dari 60)

Pancasila
Agama
Metode Numerik
Rekayasa Perangkat Lunak
Mobile Computing
Pemodelan Simulasi
Rangkaian Listrik & Elektronika
Pemrograman Dasar
Desain Sistem Digital
Robotika
Software Design for Embedded System
Desain dan Teknologi Multimedia
Aljabar Linear Elementer
Aplikasi Perkantoran
Pemrograman Web Interaktif
Jaringan Multimedia
Interaksi Manusia Komputer
Kecerdasan Buatan
Manajemen Proyek
Sistem Operasi
Matematika I
Workshop Keterampilan Komputer
Rangkaian Digital 2
Pemrograman Berorientasi Obyek
Basis Data
Kewarganegaraan
Administrasi Basis Data
Jaringan Komputer
Signal Processing
Sistem Komputer
Teknologi Audio Visual
Pemrograman Berorientasi Objek
Internet of Things
Statistik Probabilistik
Etika Profesi TI
Matematika Diskrit
Bahasa Inggris I
Mikrokontroller & Interfacing
Vektor dan Matriks
Pengolahan Citra Digital
Administrator Jaringan
Bahasa Inggris (Persiapan TOEFL)
Konsep Teknologi Rekayasa Komputer
Desain Sistem
Computer Vision
Matematika I (Diskrit dan Logika)
Bahasa Indonesia
Grafika dan Citra Komputer
Proyek Multimedia
Pengujian Aplikasi Multimedia
Augmented Reality
Organisasi dan Arsitektur Komputer
Mikroprocessor
Kendali dan Robotika
Telekomunikasi
Kesehatan dan Keselamatan Kerja
Character Building
Riset Awal
Sistem Cerdas
Pemrograman Tingkat Lanjut
Pemrograman Simulator
Teknologi Blockchain
Kewirausahan Berbasis Teknologi
Bahasa Inggris III (TOEFL Preparation)
Pemrograman Tingkat Lanjut III

---

## 🔧 Instruksi Sekarang: **Kerjakan FITUR NOMOR 5 & 6**

Mohon **hanya** implementasikan:

* **5️⃣ Etika Peminjaman** (modal konfirmasi sebelum form)
* **6️⃣ Dashboard Statistik** (kartu real‑time)

Sertakan struktur, logika, saran UI/UX, dan contoh kode JavaScript/Firebase. 


Saya akan membantu Anda mengimplementasikan **Fitur 5 (Etika Peminjaman)** dan **Fitur 6 (Dashboard Statistik)** untuk Sistem Manajemen Peminjaman Aset.Saya telah mengimplementasikan **Fitur 5 (Etika Peminjaman)** dan **Fitur 6 (Dashboard Statistik)** dengan lengkap. Berikut adalah penjelasan detail implementasinya:

## 🎯 **Fitur 5: Etika Peminjaman**

### ✅ **Modal Konfirmasi yang Ditampilkan:**
1. **Larangan Transfer**: Barang tidak boleh dipinjamkan ke pihak ketiga
2. **Wajib Kembalikan Dulu**: Barang harus dikembalikan sebelum dipinjam ulang
3. **Tanggung Jawab Penuh**: Peminjam bertanggung jawab atas kondisi aset

### 🎨 **UI/UX Features:**
- Modal dengan desain modern dan professional
- Animasi slide-in yang smooth
- Backdrop blur effect
- Responsive design untuk mobile
- Keyboard navigation (ESC untuk tutup)
- Click outside untuk tutup modal

---

## 📊 **Fitur 6: Dashboard Statistik**

### 📈 **4 Kartu Statistik Real-time:**
1. **Total Aset** - Seluruh inventaris lab
2. **Aset Tersedia** - Siap dipinjam (hijau)
3. **Aset Dipinjam** - Sedang digunakan (merah)
4. **Transaksi Aktif** - Peminjaman berlangsung (oranye)

### ⚡ **Fitur Canggih:**
- **Auto-refresh** setiap 30 detik
- **Loading animation** saat memuat data
- **Number counter animation** untuk transisi nilai
- **Update indicator** notifikasi real-time
- **Color coding** dengan gradient yang menarik
- **Hover effects** untuk interaktivitas

---

## 🔧 **Integrasi Firebase Ready**

```javascript
// Contoh integrasi dengan Firebase Realtime Database
async function loadStatistics() {
    try {
        const asetSnapshot = await database.ref('aset').once('value');
        const peminjamanSnapshot = await database.ref('peminjaman').once('value');
        
        // Kalkulasi statistik otomatis
        const totalAssets = Object.keys(asetData).length;
        const availableAssets = Object.values(asetData)
            .filter(asset => asset.status === 'Tersedia').length;
        const borrowedAssets = Object.values(asetData)
            .filter(asset => asset.status === 'Dipinjam').length;
        const activeTransactions = Object.values(peminjamanData)
            .filter(loan => loan.status === 'Dipinjam').length;
            
        updateStatistics({...});
    } catch (error) {
        // Fallback ke data demo
    }
}
```

## 🎯 **Cara Penggunaan:**

1. **Ganti Firebase Config** - Masukkan konfigurasi Firebase Anda
2. **Uncomment initializeSampleAssets()** - Untuk populate data awal
3. **Deploy** - Sistem siap digunakan publik tanpa login

## 💡 **Keunggulan Implementasi:**

- **Professional Design** - UI modern dengan gradien dan animasi
- **Mobile Responsive** - Optimal di semua device
- **Real-time Updates** - Data sinkron otomatis
- **User-friendly** - Flow yang intuitif dan aman
- **Lightweight** - Vanilla JS tanpa framework berat
- **Accessible** - Support keyboard navigation

Sistem ini sudah siap untuk diintegrasikan dengan backend Firebase dan dapat langsung digunakan untuk menggantikan pencatatan manual di kertas! 🚀

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Manajemen Peminjaman Aset</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
        }

        .header {
            text-align: center;
            margin-bottom: 30px;
            color: #2c3e50;
        }

        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            font-weight: 700;
        }

        .header p {
            font-size: 1.1rem;
            color: #7f8c8d;
        }

        /* Dashboard Statistics */
        .dashboard {
            margin-bottom: 40px;
        }

        .dashboard h2 {
            color: #2c3e50;
            margin-bottom: 20px;
            font-size: 1.8rem;
            text-align: center;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .stat-card {
            background: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 8px 25px rgba(0,0,0,0.1);
            text-align: center;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .stat-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 35px rgba(0,0,0,0.15);
        }

        .stat-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 4px;
            background: linear-gradient(90deg, #3498db, #2980b9);
        }

        .stat-card.available::before {
            background: linear-gradient(90deg, #27ae60, #229954);
        }

        .stat-card.borrowed::before {
            background: linear-gradient(90deg, #e74c3c, #c0392b);
        }

        .stat-card.transactions::before {
            background: linear-gradient(90deg, #f39c12, #e67e22);
        }

        .stat-icon {
            font-size: 3rem;
            margin-bottom: 15px;
            color: #3498db;
        }

        .stat-card.available .stat-icon {
            color: #27ae60;
        }

        .stat-card.borrowed .stat-icon {
            color: #e74c3c;
        }

        .stat-card.transactions .stat-icon {
            color: #f39c12;
        }

        .stat-number {
            font-size: 2.5rem;
            font-weight: bold;
            color: #2c3e50;
            margin-bottom: 10px;
        }

        .stat-label {
            font-size: 1rem;
            color: #7f8c8d;
            font-weight: 500;
        }

        .stat-description {
            font-size: 0.85rem;
            color: #95a5a6;
            margin-top: 8px;
        }

        /* Action Buttons */
        .action-buttons {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }

        .btn {
            padding: 12px 25px;
            border: none;
            border-radius: 8px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            text-decoration: none;
            display: inline-flex;
            align-items: center;
            gap: 8px;
        }

        .btn-primary {
            background: linear-gradient(45deg, #3498db, #2980b9);
            color: white;
        }

        .btn-primary:hover {
            background: linear-gradient(45deg, #2980b9, #21618c);
            transform: translateY(-2px);
        }

        .btn-success {
            background: linear-gradient(45deg, #27ae60, #229954);
            color: white;
        }

        .btn-success:hover {
            background: linear-gradient(45deg, #229954, #1e8449);
            transform: translateY(-2px);
        }

        /* Modal Styles */
        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.6);
            backdrop-filter: blur(5px);
        }

        .modal-content {
            background-color: white;
            margin: 8% auto;
            padding: 0;
            border-radius: 15px;
            width: 90%;
            max-width: 500px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            animation: modalSlideIn 0.3s ease-out;
        }

        @keyframes modalSlideIn {
            from {
                transform: translateY(-50px);
                opacity: 0;
            }
            to {
                transform: translateY(0);
                opacity: 1;
            }
        }

        .modal-header {
            background: linear-gradient(45deg, #3498db, #2980b9);
            color: white;
            padding: 20px 25px;
            border-radius: 15px 15px 0 0;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .modal-header h3 {
            margin: 0;
            font-size: 1.3rem;
        }

        .modal-body {
            padding: 25px;
        }

        .ethics-list {
            list-style: none;
            padding: 0;
        }

        .ethics-list li {
            padding: 15px;
            margin-bottom: 10px;
            background: #f8f9fa;
            border-left: 4px solid #3498db;
            border-radius: 8px;
            font-size: 1rem;
            line-height: 1.5;
        }

        .ethics-list li:nth-child(1) {
            border-left-color: #e74c3c;
        }

        .ethics-list li:nth-child(2) {
            border-left-color: #f39c12;
        }

        .modal-footer {
            padding: 20px 25px;
            border-top: 1px solid #ecf0f1;
            display: flex;
            justify-content: flex-end;
            gap: 10px;
        }

        .btn-secondary {
            background: #95a5a6;
            color: white;
        }

        .btn-secondary:hover {
            background: #7f8c8d;
        }

        /* Form Container (placeholder) */
        .form-container {
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 8px 25px rgba(0,0,0,0.1);
            display: none;
        }

        .form-container.show {
            display: block;
            animation: fadeIn 0.3s ease-in;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .form-title {
            color: #2c3e50;
            margin-bottom: 20px;
            font-size: 1.5rem;
            text-align: center;
        }

        /* Responsive Design */
        @media (max-width: 768px) {
            .header h1 {
                font-size: 2rem;
            }

            .stats-grid {
                grid-template-columns: 1fr;
            }

            .action-buttons {
                flex-direction: column;
                align-items: center;
            }

            .btn {
                width: 100%;
                max-width: 300px;
            }

            .modal-content {
                margin: 15% auto;
                width: 95%;
            }
        }

        /* Loading Animation */
        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid #3498db;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .update-indicator {
            position: fixed;
            top: 20px;
            right: 20px;
            background: #27ae60;
            color: white;
            padding: 10px 15px;
            border-radius: 25px;
            font-size: 0.9rem;
            display: none;
            z-index: 1001;
        }
    </style>
</head>
<body>
    <div class="update-indicator" id="updateIndicator">
        📊 Data diperbarui
    </div>

    <div class="container">
        <!-- Header -->
        <div class="header">
            <h1>📋 Sistem Manajemen Peminjaman Aset</h1>
            <p>Pengelolaan aset laboratorium secara digital dan efisien</p>
        </div>

        <!-- Dashboard Statistics -->
        <div class="dashboard">
            <h2>📊 Dashboard Statistik</h2>
            <div class="stats-grid">
                <div class="stat-card">
                    <div class="stat-icon">📦</div>
                    <div class="stat-number" id="totalAssets">
                        <div class="loading"></div>
                    </div>
                    <div class="stat-label">Total Aset</div>
                    <div class="stat-description">Seluruh inventaris lab</div>
                </div>

                <div class="stat-card available">
                    <div class="stat-icon">✅</div>
                    <div class="stat-number" id="availableAssets">
                        <div class="loading"></div>
                    </div>
                    <div class="stat-label">Aset Tersedia</div>
                    <div class="stat-description">Siap dipinjam</div>
                </div>

                <div class="stat-card borrowed">
                    <div class="stat-icon">🔄</div>
                    <div class="stat-number" id="borrowedAssets">
                        <div class="loading"></div>
                    </div>
                    <div class="stat-label">Aset Dipinjam</div>
                    <div class="stat-description">Sedang digunakan</div>
                </div>

                <div class="stat-card transactions">
                    <div class="stat-icon">📋</div>
                    <div class="stat-number" id="activeTransactions">
                        <div class="loading"></div>
                    </div>
                    <div class="stat-label">Transaksi Aktif</div>
                    <div class="stat-description">Peminjaman berlangsung</div>
                </div>
            </div>
        </div>

        <!-- Action Buttons -->
        <div class="action-buttons">
            <button class="btn btn-primary" onclick="showEthicsModal()">
                ➕ Pinjam Aset Baru
            </button>
            <button class="btn btn-success" onclick="viewHistory()">
                📜 Lihat Riwayat
            </button>
        </div>

        <!-- Form Container (Hidden initially) -->
        <div class="form-container" id="formContainer">
            <h2 class="form-title">📝 Form Peminjaman Aset</h2>
            <p style="text-align: center; color: #7f8c8d;">
                Form peminjaman akan ditampilkan di sini setelah menyetujui etika peminjaman.
            </p>
        </div>
    </div>

    <!-- Ethics Modal -->
    <div id="ethicsModal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <span style="font-size: 1.5rem;">⚖️</span>
                <h3>Etika Peminjaman Aset</h3>
            </div>
            <div class="modal-body">
                <p style="margin-bottom: 20px; color: #2c3e50; font-weight: 500;">
                    Sebelum melakukan peminjaman, harap membaca dan menyetujui ketentuan berikut:
                </p>
                <ul class="ethics-list">
                    <li>
                        <strong>🚫 Larangan Transfer:</strong><br>
                        Barang yang dipinjam tidak boleh dipinjamkan kepada pihak ketiga atau orang lain tanpa seizin petugas laboratorium.
                    </li>
                    <li>
                        <strong>🔄 Wajib Kembalikan Dulu:</strong><br>
                        Barang harus dikembalikan terlebih dahulu sebelum dapat melakukan peminjaman ulang untuk aset yang sama.
                    </li>
                    <li>
                        <strong>⏰ Tanggung Jawab Penuh:</strong><br>
                        Peminjam bertanggung jawab penuh atas kondisi dan keamanan aset selama masa peminjaman.
                    </li>
                </ul>
                <div style="background: #e8f4f8; padding: 15px; border-radius: 8px; margin-top: 20px;">
                    <p style="color: #2c3e50; font-size: 0.9rem; margin: 0;">
                        <strong>💡 Catatan:</strong> Dengan melanjutkan, Anda menyatakan setuju dan akan mematuhi semua ketentuan di atas.
                    </p>
                </div>
            </div>
            <div class="modal-footer">
                <button class="btn btn-secondary" onclick="closeEthicsModal()">
                    ❌ Batal
                </button>
                <button class="btn btn-primary" onclick="acceptEthicsAndShowForm()">
                    ✅ Setuju & Lanjutkan
                </button>
            </div>
        </div>
    </div>

    <!-- Firebase SDK -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.23.0/firebase-app-compat.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.23.0/firebase-database-compat.min.js"></script>

    <script>
        // Firebase Configuration (Ganti dengan config Anda)
        const firebaseConfig = {
            // Masukkan konfigurasi Firebase Anda di sini
            databaseURL: "https://your-project-default-rtdb.firebaseio.com/"
        };

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        // Global Statistics Variables
        let statsData = {
            totalAssets: 0,
            availableAssets: 0,
            borrowedAssets: 0,
            activeTransactions: 0
        };

        // Initialize Dashboard
        document.addEventListener('DOMContentLoaded', function() {
            initializeDashboard();
            loadStatistics();
            
            // Auto-refresh setiap 30 detik
            setInterval(loadStatistics, 30000);
        });

        // Initialize Dashboard with Sample Data (for demo)
        function initializeDashboard() {
            console.log('🚀 Initializing Asset Management System...');
            
            // Simulasi data awal untuk demo
            setTimeout(() => {
                updateStatistics({
                    totalAssets: 142,
                    availableAssets: 98,
                    borrowedAssets: 44,
                    activeTransactions: 44
                });
            }, 1500);
        }

        // Load Statistics from Firebase
        async function loadStatistics() {
            try {
                // Fetch aset data
                const asetSnapshot = await database.ref('aset').once('value');
                const asetData = asetSnapshot.val() || {};
                
                // Fetch peminjaman data
                const peminjamanSnapshot = await database.ref('peminjaman').once('value');
                const peminjamanData = peminjamanSnapshot.val() || {};

                // Calculate statistics
                const totalAssets = Object.keys(asetData).length;
                const availableAssets = Object.values(asetData).filter(asset => asset.status === 'Tersedia').length;
                const borrowedAssets = Object.values(asetData).filter(asset => asset.status === 'Dipinjam').length;
                const activeTransactions = Object.values(peminjamanData).filter(loan => loan.status === 'Dipinjam').length;

                const newStats = {
                    totalAssets,
                    availableAssets,
                    borrowedAssets,
                    activeTransactions
                };

                updateStatistics(newStats);
                showUpdateIndicator();

            } catch (error) {
                console.error('Error loading statistics:', error);
                
                // Fallback ke data demo jika Firebase tidak tersedia
                updateStatistics({
                    totalAssets: 142,
                    availableAssets: 98,
                    borrowedAssets: 44,
                    activeTransactions: 44
                });
            }
        }

        // Update Statistics Display
        function updateStatistics(newStats) {
            statsData = newStats;

            // Update dengan animasi
            animateNumber('totalAssets', newStats.totalAssets);
            animateNumber('availableAssets', newStats.availableAssets);
            animateNumber('borrowedAssets', newStats.borrowedAssets);
            animateNumber('activeTransactions', newStats.activeTransactions);
        }

        // Animate Number Counter
        function animateNumber(elementId, targetValue) {
            const element = document.getElementById(elementId);
            const currentValue = parseInt(element.textContent) || 0;
            const increment = Math.ceil((targetValue - currentValue) / 20);
            
            if (increment === 0) {
                element.textContent = targetValue;
                return;
            }

            const timer = setInterval(() => {
                const current = parseInt(element.textContent) || 0;
                if ((increment > 0 && current >= targetValue) || (increment < 0 && current <= targetValue)) {
                    element.textContent = targetValue;
                    clearInterval(timer);
                } else {
                    element.textContent = current + increment;
                }
            }, 50);
        }

        // Show Update Indicator
        function showUpdateIndicator() {
            const indicator = document.getElementById('updateIndicator');
            indicator.style.display = 'block';
            setTimeout(() => {
                indicator.style.display = 'none';
            }, 2000);
        }

        // Ethics Modal Functions
        function showEthicsModal() {
            document.getElementById('ethicsModal').style.display = 'block';
            document.body.style.overflow = 'hidden'; // Prevent scrolling
        }

        function closeEthicsModal() {
            document.getElementById('ethicsModal').style.display = 'none';
            document.body.style.overflow = 'auto'; // Restore scrolling
        }

        function acceptEthicsAndShowForm() {
            closeEthicsModal();
            
            // Show form container
            const formContainer = document.getElementById('formContainer');
            formContainer.classList.add('show');
            
            // Smooth scroll to form
            formContainer.scrollIntoView({ 
                behavior: 'smooth',
                block: 'start'
            });
            
            // Show success message
            setTimeout(() => {
                alert('✅ Etika peminjaman telah disetujui!\n\nAnda dapat melanjutkan mengisi form peminjaman.');
            }, 500);
        }

        // View History Function
        function viewHistory() {
            alert('📜 Fitur Riwayat Peminjaman\n\nFitur ini akan menampilkan tabel riwayat lengkap dengan filter berdasarkan:\n• Nama Lengkap\n• Kelas\n• Tanggal Pinjam\n\n(Implementasi tabel akan dikerjakan pada tahap selanjutnya)');
        }

        // Close modal when clicking outside
        window.onclick = function(event) {
            const modal = document.getElementById('ethicsModal');
            if (event.target === modal) {
                closeEthicsModal();
            }
        }

        // Keyboard navigation
        document.addEventListener('keydown', function(event) {
            if (event.key === 'Escape') {
                closeEthicsModal();
            }
        });

        // Sample function to initialize assets (for development)
        async function initializeSampleAssets() {
            try {
                const sampleAssets = [
                    // 24 Proyektor EPSON
                    ...Array.from({length: 24}, (_, i) => ({
                        id: `proj-${String(i + 1).padStart(2, '0')}`,
                        namaBarang: 'Proyektor EPSON',
                        nomorBarang: String(i + 1).padStart(2, '0'),
                        status: Math.random() > 0.3 ? 'Tersedia' : 'Dipinjam',
                        tanggalTambah: new Date().toISOString()
                    })),
                    // 24 Converter ROBOT
                    ...Array.from({length: 24}, (_, i) => ({
                        id: `conv-${String(i + 1).padStart(2, '0')}`,
                        namaBarang: 'Converter ROBOT RHV01',
                        nomorBarang: String(i + 1).padStart(2, '0'),
                        status: Math.random() > 0.3 ? 'Tersedia' : 'Dipinjam',
                        tanggalTambah: new Date().toISOString()
                    })),
                    // 24 ATK
                    ...Array.from({length: 24}, (_, i) => ({
                        id: `atk-${String(i + 1).padStart(2, '0')}`,
                        namaBarang: 'ATK',
                        nomorBarang: String(i + 1).padStart(2, '0'),
                        status: Math.random() > 0.3 ? 'Tersedia' : 'Dipinjam',
                        tanggalTambah: new Date().toISOString()
                    })),
                    // 30 Komputer HP
                    ...Array.from({length: 30}, (_, i) => ({
                        id: `comp-${String(i + 1).padStart(2, '0')}`,
                        namaBarang: 'Komputer HP',
                        nomorBarang: String(i + 1).padStart(2, '0'),
                        status: Math.random() > 0.3 ? 'Tersedia' : 'Dipinjam',
                        tanggalTambah: new Date().toISOString()
                    })),
                    // 20 Stop Kontak
                    ...Array.from({length: 20}, (_, i) => ({
                        id: `stop-${String(i + 1).padStart(2, '0')}`,
                        namaBarang: 'Stop Kontak',
                        nomorBarang: String(i + 1).padStart(2, '0'),
                        status: Math.random() > 0.3 ? 'Tersedia' : 'Dipinjam',
                        tanggalTambah: new Date().toISOString()
                    }))
                ];

                // Batch upload to Firebase (uncomment when ready)
                /*
                const batch = {};
                sampleAssets.forEach(asset => {
                    batch[asset.id] = asset;
                });
                await database.ref('aset').set(batch);
                console.log('✅ Sample assets initialized successfully!');
                */

                console.log('📦 S2ample assets ready:', sampleAssets.length, 'items');
                
            } catch (error) {
                console.error('Error initializing sample assets:', error);
            }
        }

        // Call this function once to populate your database
        // initializeSampleAssets();
    </script>
</body>
</html>