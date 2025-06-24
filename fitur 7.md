Saya akan membuat **Tabel Transaksi Lengkap** sesuai dengan spesifikasi yang Anda berikan. Ini akan menjadi fitur nomor 7 yang menampilkan seluruh data peminjaman dari Firebase Realtime Database dengan tampilan yang responsif dan interaktif.Saya telah membuat **Tabel Transaksi Lengkap** sesuai dengan spesifikasi fitur nomor 7. Berikut adalah fitur-fitur yang telah diimplementasikan:

## ✅ Fitur yang Telah Dibuat:

### 📊 **Tampilan Tabel**
- **11 Kolom lengkap**: No | Nama Lengkap | NIM | Barang | Nomor Barang | Kelas | Mata Pelajaran | Tanggal Pinjam | Tanggal Kembali | Status | Aksi
- **Responsive design** dengan horizontal scroll untuk layar kecil
- **Zebra-striped rows** untuk kemudahan membaca
- **Header biru tua** dengan gradient yang kontras
- **Hover effects** untuk interaktivitas yang lebih baik

### 🔄 **Real-time Data**
- Menggunakan Firebase `on('value')` untuk data real-time
- Auto-refresh saat ada perubahan data di database
- Loading state yang informatif

### 🎯 **Fungsionalitas Aksi**
- **Tombol "Kembalikan"**: 
  - Hanya muncul jika status "Dipinjam"
  - Mengubah status peminjaman → "Dikembalikan"
  - Mengubah status aset → "Tersedia"
- **Tombol "Hapus"**:
  - Selalu tersedia di setiap baris
  - Soft delete → pindah ke `deleted/peminjaman`
  - Auto-update status aset → "Tersedia"

### 📅 **Format Data**
- **Tanggal**: Format DD/MM/YYYY (lokal Indonesia)
- **Status**: Badge dengan warna berbeda (Dipinjam/Dikembalikan)
- **Nomor Barang**: Ditampilkan dengan format #XX
- **Data terurut**: Berdasarkan tanggal pinjam terbaru

### ⚡ **User Experience**
- **Toast notifications** untuk feedback aksi
- **Konfirmasi dialog** sebelum aksi penting
- **Empty state** saat belum ada data
- **Connection status** monitoring
- **Mobile-friendly** responsive design

## 🔧 **Cara Penggunaan:**

1. **Ganti konfigurasi Firebase** di bagian `firebaseConfig` dengan kredensial project Anda
2. **Pastikan Firebase Rules** sudah di-set untuk public access
3. **Deploy** file HTML ini ke web server atau buka langsung di browser

## 📝 **Catatan Teknis:**
- Menggunakan **vanilla JavaScript** tanpa framework
- **Firebase Realtime Database SDK** via CDN
- **No localStorage/sessionStorage** sesuai pembatasan
- **Fully functional** dengan error handling lengkap

Tabel ini siap digunakan dan akan secara otomatis menampilkan semua data peminjaman dari Firebase dengan kemampuan real-time update. Silakan test dan beri tahu jika ada penyesuaian yang diperlukan sebelum lanjut ke fitur berikutnya! 🚀

<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tabel Transaksi Lengkap - Sistem Peminjaman Aset</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }

        .header {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }

        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            font-weight: 700;
        }

        .header p {
            font-size: 1.1rem;
            opacity: 0.9;
        }

        .content {
            padding: 30px;
        }

        .loading {
            text-align: center;
            padding: 50px;
            color: #666;
            font-size: 1.2rem;
        }

        .loading::after {
            content: '';
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid #3498db;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-left: 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .table-container {
            overflow-x: auto;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.08);
        }

        table {
            width: 100%;
            border-collapse: collapse;
            min-width: 1200px;
        }

        thead th {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            padding: 15px 12px;
            text-align: left;
            font-weight: 600;
            font-size: 0.95rem;
            position: sticky;
            top: 0;
            z-index: 10;
        }

        tbody tr {
            transition: all 0.3s ease;
        }

        tbody tr:nth-child(even) {
            background-color: #f8f9fa;
        }

        tbody tr:hover {
            background-color: #e3f2fd;
            transform: translateY(-1px);
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }

        tbody td {
            padding: 12px;
            border-bottom: 1px solid #e0e0e0;
            font-size: 0.9rem;
        }

        .status-badge {
            padding: 6px 12px;
            border-radius: 20px;
            font-size: 0.8rem;
            font-weight: 600;
            text-align: center;
        }

        .status-dipinjam {
            background-color: #fff3cd;
            color: #856404;
            border: 1px solid #ffeaa7;
        }

        .status-dikembalikan {
            background-color: #d1edff;
            color: #0c5460;
            border: 1px solid #bee5eb;
        }

        .btn {
            padding: 8px 16px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 0.85rem;
            font-weight: 600;
            transition: all 0.3s ease;
            margin: 2px;
        }

        .btn-return {
            background-color: #28a745;
            color: white;
        }

        .btn-return:hover {
            background-color: #218838;
            transform: translateY(-1px);
        }

        .btn-delete {
            background-color: #dc3545;
            color: white;
        }

        .btn-delete:hover {
            background-color: #c82333;
            transform: translateY(-1px);
        }

        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .empty-state {
            text-align: center;
            padding: 60px 20px;
            color: #666;
        }

        .empty-state i {
            font-size: 4rem;
            color: #ddd;
            margin-bottom: 20px;
        }

        .toast {
            position: fixed;
            top: 20px;
            right: 20px;
            padding: 15px 20px;
            border-radius: 8px;
            color: white;
            font-weight: 600;
            z-index: 1000;
            transform: translateX(100%);
            transition: transform 0.3s ease;
        }

        .toast.show {
            transform: translateX(0);
        }

        .toast.success {
            background-color: #28a745;
        }

        .toast.error {
            background-color: #dc3545;
        }

        @media (max-width: 768px) {
            .header h1 {
                font-size: 2rem;
            }
            
            .content {
                padding: 20px;
            }
            
            thead th, tbody td {
                padding: 8px 6px;
                font-size: 0.8rem;
            }
            
            .btn {
                padding: 6px 10px;
                font-size: 0.75rem;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📊 Tabel Transaksi Lengkap</h1>
            <p>Sistem Manajemen Peminjaman Aset - Real-time Data</p>
        </div>
        
        <div class="content">
            <div id="loading" class="loading">
                Memuat data transaksi...
            </div>
            
            <div id="tableContainer" class="table-container" style="display: none;">
                <table id="transactionTable">
                    <thead>
                        <tr>
                            <th>No</th>
                            <th>Nama Lengkap</th>
                            <th>NIM</th>
                            <th>Barang</th>
                            <th>Nomor Barang</th>
                            <th>Kelas</th>
                            <th>Mata Pelajaran</th>
                            <th>Tanggal Pinjam</th>
                            <th>Tanggal Kembali</th>
                            <th>Status</th>
                            <th>Aksi</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody">
                        <!-- Data akan dimuat di sini -->
                    </tbody>
                </table>
            </div>
            
            <div id="emptyState" class="empty-state" style="display: none;">
                <div style="font-size: 4rem; margin-bottom: 20px;">📋</div>
                <h3>Belum Ada Data Transaksi</h3>
                <p>Data peminjaman akan muncul di sini secara real-time</p>
            </div>
        </div>
    </div>

    <!-- Toast Notification -->
    <div id="toast" class="toast"></div>

    <!-- Firebase SDK -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.23.0/firebase-app-compat.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/firebase/9.23.0/firebase-database-compat.min.js"></script>

    <script>
        // Konfigurasi Firebase (ganti dengan konfigurasi Anda)
        const firebaseConfig = {
            apiKey: "your-api-key",
            authDomain: "your-project.firebaseapp.com",
            databaseURL: "https://your-project-default-rtdb.firebaseio.com/",
            projectId: "your-project",
            storageBucket: "your-project.appspot.com",
            messagingSenderId: "123456789",
            appId: "your-app-id"
        };

        // Inisialisasi Firebase
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        class TransactionTable {
            constructor() {
                this.transactions = {};
                this.assets = {};
                this.init();
            }

            init() {
                this.loadAssets();
                this.loadTransactions();
            }

            loadAssets() {
                database.ref('aset').on('value', (snapshot) => {
                    if (snapshot.exists()) {
                        this.assets = snapshot.val();
                    }
                });
            }

            loadTransactions() {
                database.ref('peminjaman').on('value', (snapshot) => {
                    const loading = document.getElementById('loading');
                    const tableContainer = document.getElementById('tableContainer');
                    const emptyState = document.getElementById('emptyState');

                    if (snapshot.exists()) {
                        this.transactions = snapshot.val();
                        this.renderTable();
                        
                        loading.style.display = 'none';
                        tableContainer.style.display = 'block';
                        emptyState.style.display = 'none';
                    } else {
                        loading.style.display = 'none';
                        tableContainer.style.display = 'none';
                        emptyState.style.display = 'block';
                    }
                });
            }

            renderTable() {
                const tableBody = document.getElementById('tableBody');
                tableBody.innerHTML = '';

                const transactionArray = Object.entries(this.transactions).map(([id, data]) => ({
                    id,
                    ...data
                }));

                // Urutkan berdasarkan tanggal pinjam terbaru
                transactionArray.sort((a, b) => new Date(b.tanggalPinjam) - new Date(a.tanggalPinjam));

                transactionArray.forEach((transaction, index) => {
                    const row = this.createTableRow(transaction, index + 1);
                    tableBody.appendChild(row);
                });
            }

            createTableRow(transaction, number) {
                const row = document.createElement('tr');
                
                const statusClass = transaction.status === 'Dipinjam' ? 'status-dipinjam' : 'status-dikembalikan';
                
                row.innerHTML = `
                    <td>${number}</td>
                    <td><strong>${transaction.namaLengkap}</strong></td>
                    <td>${transaction.nim}</td>
                    <td>${transaction.namaBarang}</td>
                    <td><strong>#${transaction.nomorBarang}</strong></td>
                    <td>${transaction.kelas}</td>
                    <td>${transaction.mataPelajaran}</td>
                    <td>${this.formatDate(transaction.tanggalPinjam)}</td>
                    <td>${this.formatDate(transaction.tanggalKembali)}</td>
                    <td><span class="status-badge ${statusClass}">${transaction.status}</span></td>
                    <td>
                        ${transaction.status === 'Dipinjam' ? 
                            `<button class="btn btn-return" onclick="transactionTable.returnItem('${transaction.id}', '${transaction.asetRef}')">
                                Kembalikan
                            </button>` : ''
                        }
                        <button class="btn btn-delete" onclick="transactionTable.deleteTransaction('${transaction.id}', '${transaction.asetRef}')">
                            Hapus
                        </button>
                    </td>
                `;

                return row;
            }

            formatDate(dateString) {
                if (!dateString) return '-';
                const date = new Date(dateString);
                return date.toLocaleDateString('id-ID', {
                    day: '2-digit',
                    month: '2-digit',
                    year: 'numeric'
                });
            }

            async returnItem(transactionId, assetId) {
                try {
                    const confirmed = confirm('Apakah Anda yakin ingin mengembalikan barang ini?');
                    if (!confirmed) return;

                    // Update status peminjaman
                    await database.ref(`peminjaman/${transactionId}/status`).set('Dikembalikan');
                    
                    // Update status aset
                    await database.ref(`aset/${assetId}/status`).set('Tersedia');
                    
                    this.showToast('Barang berhasil dikembalikan!', 'success');
                } catch (error) {
                    console.error('Error returning item:', error);
                    this.showToast('Gagal mengembalikan barang. Silakan coba lagi.', 'error');
                }
            }

            async deleteTransaction(transactionId, assetId) {
                try {
                    const confirmed = confirm('Apakah Anda yakin ingin menghapus transaksi ini? Data akan dipindahkan ke trash.');
                    if (!confirmed) return;

                    const transaction = this.transactions[transactionId];
                    
                    // Pindahkan ke deleted/peminjaman (soft delete)
                    await database.ref(`deleted/peminjaman/${transactionId}`).set({
                        ...transaction,
                        deletedAt: new Date().toISOString()
                    });
                    
                    // Hapus dari peminjaman utama
                    await database.ref(`peminjaman/${transactionId}`).remove();
                    
                    // Update status aset menjadi tersedia
                    await database.ref(`aset/${assetId}/status`).set('Tersedia');
                    
                    this.showToast('Transaksi berhasil dihapus dan dipindahkan ke trash!', 'success');
                } catch (error) {
                    console.error('Error deleting transaction:', error);
                    this.showToast('Gagal menghapus transaksi. Silakan coba lagi.', 'error');
                }
            }

            showToast(message, type) {
                const toast = document.getElementById('toast');
                toast.textContent = message;
                toast.className = `toast ${type}`;
                toast.classList.add('show');
                
                setTimeout(() => {
                    toast.classList.remove('show');
                }, 3000);
            }
        }

        // Inisialisasi aplikasi
        const transactionTable = new TransactionTable();

        // Error handling untuk Firebase
        database.ref('.info/connected').on('value', (snapshot) => {
            if (snapshot.val() === false) {
                document.getElementById('loading').innerHTML = '⚠️ Tidak dapat terhubung ke database. Periksa koneksi internet Anda.';
            }
        });
    </script>
</body>
</html>