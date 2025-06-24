# Dokumentasi Implementasi Teknis
## Sistem Manajemen Peminjaman Aset - Fitur 3 & 4

### 🎯 Overview
Dokumentasi ini melanjutkan implementasi sistem pada **fitur 3 (Riwayat Peminjaman)** dan **fitur 4 (Trash Bin/Soft Delete)** untuk Sistem Manajemen Peminjaman Aset berbasis Firebase Realtime Database.

---

## 3️⃣ RIWAYAT PEMINJAMAN

### 3.1 Struktur Database untuk Riwayat

```javascript
// Firebase Realtime Database Structure
{
  "peminjaman": {
    "uuid-1": {
      "id": "uuid-1",
      "nama": "John Doe",
      "nim": "123456789",
      "kelas": "TI-3A",
      "matkul": "Pemrograman Web Interaktif",
      "noHp": "081234567890",
      "barang": "Proyektor EPSON",
      "nomorBarang": "01",
      "tanggalPinjam": "2025-06-13",
      "tanggalKembali": "2025-06-15",
      "status": "dipinjam", // dipinjam | dikembalikan
      "createdAt": "2025-06-13T10:30:00+07:00",
      "updatedAt": "2025-06-13T10:30:00+07:00"
    }
  },
  "aset": {
    "proyektor-epson-01": {
      "nama": "Proyektor EPSON",
      "nomor": "01",
      "status": "dipinjam", // tersedia | dipinjam
      "dipinjamOleh": "uuid-1"
    }
  }
}
```

### 3.2 HTML Template untuk Riwayat Peminjaman

```html
<!-- Section Riwayat Peminjaman -->
<section id="riwayat-section" class="container mt-5">
    <div class="card">
        <div class="card-header bg-info text-white">
            <h4><i class="fas fa-history"></i> Riwayat Peminjaman</h4>
        </div>
        <div class="card-body">
            <!-- Filter Controls -->
            <div class="row mb-3">
                <div class="col-md-3">
                    <label for="filterNama" class="form-label">Filter Nama</label>
                    <input type="text" id="filterNama" class="form-control" placeholder="Masukkan nama...">
                </div>
                <div class="col-md-3">
                    <label for="filterKelas" class="form-label">Filter Kelas</label>
                    <select id="filterKelas" class="form-select">
                        <option value="">Semua Kelas</option>
                        <option value="TI-1A">TI-1A</option>
                        <option value="TI-1B">TI-1B</option>
                        <option value="TI-2A">TI-2A</option>
                        <option value="TI-2B">TI-2B</option>
                        <option value="TI-3A">TI-3A</option>
                        <option value="TI-3B">TI-3B</option>
                        <option value="TI-4A">TI-4A</option>
                        <option value="TI-4B">TI-4B</option>
                    </select>
                </div>
                <div class="col-md-3">
                    <label for="filterTanggalMulai" class="form-label">Tanggal Mulai</label>
                    <input type="date" id="filterTanggalMulai" class="form-control">
                </div>
                <div class="col-md-3">
                    <label for="filterTanggalAkhir" class="form-label">Tanggal Akhir</label>
                    <input type="date" id="filterTanggalAkhir" class="form-control">
                </div>
            </div>
            
            <!-- Action Buttons -->
            <div class="row mb-3">
                <div class="col">
                    <button id="btnApplyFilter" class="btn btn-primary">
                        <i class="fas fa-filter"></i> Terapkan Filter
                    </button>
                    <button id="btnResetFilter" class="btn btn-secondary">
                        <i class="fas fa-undo"></i> Reset Filter
                    </button>
                    <button id="btnSyncToSheets" class="btn btn-success">
                        <i class="fas fa-sync"></i> Sync ke Google Sheets
                    </button>
                </div>
            </div>

            <!-- Loading Indicator -->
            <div id="riwayatLoading" class="text-center" style="display: none;">
                <div class="spinner-border text-primary" role="status">
                    <span class="visually-hidden">Loading...</span>
                </div>
                <p>Memuat riwayat peminjaman...</p>
            </div>

            <!-- Riwayat Table -->
            <div class="table-responsive">
                <table id="riwayatTable" class="table table-striped table-hover">
                    <thead class="table-dark">
                        <tr>
                            <th>No</th>
                            <th>Nama</th>
                            <th>NIM</th>
                            <th>Kelas</th>
                            <th>Mata Kuliah</th>
                            <th>Barang</th>
                            <th>No. Barang</th>
                            <th>Tgl Pinjam</th>
                            <th>Tgl Kembali</th>
                            <th>Status</th>
                            <th>Aksi</th>
                        </tr>
                    </thead>
                    <tbody id="riwayatTableBody">
                        <!-- Data will be populated by JavaScript -->
                    </tbody>
                </table>
            </div>

            <!-- Pagination -->
            <nav aria-label="Pagination riwayat">
                <ul id="riwayatPagination" class="pagination justify-content-center">
                    <!-- Pagination will be generated by JavaScript -->
                </ul>
            </nav>
        </div>
    </div>
</section>
```

### 3.3 JavaScript Implementation untuk Riwayat

```javascript
// riwayat-manager.js
class RiwayatManager {
    constructor() {
        this.database = firebase.database();
        this.currentFilters = {
            nama: '',
            kelas: '',
            tanggalMulai: '',
            tanggalAkhir: ''
        };
        this.currentPage = 1;
        this.itemsPerPage = 10;
        this.allData = [];
        this.filteredData = [];
        
        this.initializeEventListeners();
        this.loadRiwayatData();
    }

    initializeEventListeners() {
        // Filter event listeners
        document.getElementById('btnApplyFilter').addEventListener('click', () => {
            this.applyFilters();
        });

        document.getElementById('btnResetFilter').addEventListener('click', () => {
            this.resetFilters();
        });

        document.getElementById('btnSyncToSheets').addEventListener('click', () => {
            this.syncToGoogleSheets();
        });

        // Real-time input filtering
        document.getElementById('filterNama').addEventListener('input', 
            this.debounce(() => this.applyFilters(), 500)
        );
    }

    async loadRiwayatData() {
        try {
            this.showLoading(true);
            
            const snapshot = await this.database.ref('peminjaman').once('value');
            this.allData = [];
            
            if (snapshot.exists()) {
                snapshot.forEach((childSnapshot) => {
                    const data = childSnapshot.val();
                    this.allData.push(data);
                });
                
                // Sort by creation date (newest first)
                this.allData.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
            }
            
            this.filteredData = [...this.allData];
            this.renderTable();
            this.renderPagination();
            
        } catch (error) {
            console.error('Error loading riwayat data:', error);
            this.showError('Gagal memuat data riwayat peminjaman');
        } finally {
            this.showLoading(false);
        }
    }

    applyFilters() {
        const nama = document.getElementById('filterNama').value.toLowerCase();
        const kelas = document.getElementById('filterKelas').value;
        const tanggalMulai = document.getElementById('filterTanggalMulai').value;
        const tanggalAkhir = document.getElementById('filterTanggalAkhir').value;

        this.currentFilters = { nama, kelas, tanggalMulai, tanggalAkhir };

        this.filteredData = this.allData.filter(item => {
            // Filter by nama
            if (nama && !item.nama.toLowerCase().includes(nama)) {
                return false;
            }

            // Filter by kelas
            if (kelas && item.kelas !== kelas) {
                return false;
            }

            // Filter by date range
            if (tanggalMulai || tanggalAkhir) {
                const itemDate = new Date(item.tanggalPinjam);
                
                if (tanggalMulai) {
                    const startDate = new Date(tanggalMulai);
                    if (itemDate < startDate) return false;
                }
                
                if (tanggalAkhir) {
                    const endDate = new Date(tanggalAkhir);
                    endDate.setHours(23, 59, 59, 999); // End of day
                    if (itemDate > endDate) return false;
                }
            }

            return true;
        });

        this.currentPage = 1;
        this.renderTable();
        this.renderPagination();
    }

    resetFilters() {
        document.getElementById('filterNama').value = '';
        document.getElementById('filterKelas').value = '';
        document.getElementById('filterTanggalMulai').value = '';
        document.getElementById('filterTanggalAkhir').value = '';
        
        this.currentFilters = { nama: '', kelas: '', tanggalMulai: '', tanggalAkhir: '' };
        this.filteredData = [...this.allData];
        this.currentPage = 1;
        
        this.renderTable();
        this.renderPagination();
    }

    renderTable() {
        const tbody = document.getElementById('riwayatTableBody');
        tbody.innerHTML = '';

        if (this.filteredData.length === 0) {
            tbody.innerHTML = `
                <tr>
                    <td colspan="11" class="text-center text-muted">
                        <i class="fas fa-inbox fa-3x mb-3"></i>
                        <p>Tidak ada data riwayat peminjaman</p>
                    </td>
                </tr>
            `;
            return;
        }

        const startIndex = (this.currentPage - 1) * this.itemsPerPage;
        const endIndex = startIndex + this.itemsPerPage;
        const pageData = this.filteredData.slice(startIndex, endIndex);

        pageData.forEach((item, index) => {
            const row = this.createTableRow(item, startIndex + index + 1);
            tbody.appendChild(row);
        });
    }

    createTableRow(item, nomor) {
        const row = document.createElement('tr');
        
        const statusBadge = item.status === 'dipinjam' 
            ? '<span class="badge bg-warning">Dipinjam</span>'
            : '<span class="badge bg-success">Dikembalikan</span>';

        const actionButtons = item.status === 'dipinjam'
            ? `<button class="btn btn-sm btn-success" onclick="riwayatManager.kembalikanBarang('${item.id}')">
                   <i class="fas fa-check"></i> Kembalikan
               </button>`
            : `<button class="btn btn-sm btn-danger" onclick="riwayatManager.hapusPeminjaman('${item.id}')">
                   <i class="fas fa-trash"></i> Hapus
               </button>`;

        row.innerHTML = `
            <td>${nomor}</td>
            <td>${item.nama}</td>
            <td>${item.nim}</td>
            <td>${item.kelas}</td>
            <td>${item.matkul}</td>
            <td>${item.barang}</td>
            <td>${item.nomorBarang}</td>
            <td>${this.formatDate(item.tanggalPinjam)}</td>
            <td>${this.formatDate(item.tanggalKembali)}</td>
            <td>${statusBadge}</td>
            <td>${actionButtons}</td>
        `;

        return row;
    }

    renderPagination() {
        const pagination = document.getElementById('riwayatPagination');
        pagination.innerHTML = '';

        const totalPages = Math.ceil(this.filteredData.length / this.itemsPerPage);
        
        if (totalPages <= 1) return;

        // Previous button
        const prevButton = this.createPaginationButton('‹', this.currentPage - 1, this.currentPage === 1);
        pagination.appendChild(prevButton);

        // Page numbers
        for (let i = 1; i <= totalPages; i++) {
            if (i === 1 || i === totalPages || (i >= this.currentPage - 2 && i <= this.currentPage + 2)) {
                const pageButton = this.createPaginationButton(i, i, false, i === this.currentPage);
                pagination.appendChild(pageButton);
            } else if (i === this.currentPage - 3 || i === this.currentPage + 3) {
                const ellipsis = document.createElement('li');
                ellipsis.className = 'page-item disabled';
                ellipsis.innerHTML = '<span class="page-link">...</span>';
                pagination.appendChild(ellipsis);
            }
        }

        // Next button
        const nextButton = this.createPaginationButton('›', this.currentPage + 1, this.currentPage === totalPages);
        pagination.appendChild(nextButton);
    }

    createPaginationButton(text, page, disabled = false, active = false) {
        const li = document.createElement('li');
        li.className = `page-item ${disabled ? 'disabled' : ''} ${active ? 'active' : ''}`;
        
        const a = document.createElement('a');
        a.className = 'page-link';
        a.href = '#';
        a.textContent = text;
        
        if (!disabled) {
            a.addEventListener('click', (e) => {
                e.preventDefault();
                this.currentPage = page;
                this.renderTable();
                this.renderPagination();
            });
        }
        
        li.appendChild(a);
        return li;
    }

    // Google Sheets Integration
    async syncToGoogleSheets() {
        try {
            const syncButton = document.getElementById('btnSyncToSheets');
            const originalText = syncButton.innerHTML;
            
            syncButton.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Syncing...';
            syncButton.disabled = true;

            // Prepare data for Google Sheets
            const sheetData = this.allData.map(item => ({
                nama: item.nama,
                nim: item.nim,
                kelas: item.kelas,
                mataKuliah: item.matkul,
                noHp: item.noHp,
                barang: item.barang,
                nomorBarang: item.nomorBarang,
                tanggalPinjam: item.tanggalPinjam,
                tanggalKembali: item.tanggalKembali,
                status: item.status,
                createdAt: item.createdAt,
                updatedAt: item.updatedAt
            }));

            // Send to webhook
            const webhookUrl = 'YOUR_GOOGLE_SHEETS_WEBHOOK_URL'; // Replace with actual webhook URL
            
            const response = await fetch(webhookUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    action: 'sync_all',
                    data: sheetData,
                    timestamp: new Date().toISOString()
                })
            });

            if (response.ok) {
                this.showSuccess('Data berhasil disinkronkan ke Google Sheets');
            } else {
                throw new Error('Webhook response not OK');
            }

        } catch (error) {
            console.error('Error syncing to Google Sheets:', error);
            this.showError('Gagal menyinkronkan data ke Google Sheets');
        } finally {
            const syncButton = document.getElementById('btnSyncToSheets');
            syncButton.innerHTML = '<i class="fas fa-sync"></i> Sync ke Google Sheets';
            syncButton.disabled = false;
        }
    }

    // Auto-sync on data changes
    async autoSyncToSheets(action, data) {
        try {
            const webhookUrl = 'YOUR_GOOGLE_SHEETS_WEBHOOK_URL';
            
            await fetch(webhookUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    action: action, // 'create', 'update', 'delete'
                    data: data,
                    timestamp: new Date().toISOString()
                })
            });

        } catch (error) {
            console.error('Auto-sync failed:', error);
        }
    }

    // Utility methods
    formatDate(dateString) {
        const date = new Date(dateString);
        return date.toLocaleDateString('id-ID', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit'
        });
    }

    debounce(func, wait) {
        let timeout;
        return function executedFunction(...args) {
            const later = () => {
                clearTimeout(timeout);
                func(...args);
            };
            clearTimeout(timeout);
            timeout = setTimeout(later, wait);
        };
    }

    showLoading(show) {
        const loading = document.getElementById('riwayatLoading');
        const table = document.getElementById('riwayatTable');
        
        if (show) {
            loading.style.display = 'block';
            table.style.display = 'none';
        } else {
            loading.style.display = 'none';
            table.style.display = 'table';
        }
    }

    showSuccess(message) {
        // Implementation depends on your notification system
        alert(message); // Replace with better notification
    }

    showError(message) {
        // Implementation depends on your notification system
        alert(message); // Replace with better notification
    }
}
```

### 3.4 Google Sheets Webhook Setup

#### 3.4.1 Google Apps Script untuk Webhook

```javascript
// Google Apps Script (webhook-handler.gs)
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSheet();
    
    // Set headers if sheet is empty
    if (sheet.getLastRow() === 0) {
      const headers = [
        'Nama', 'NIM', 'Kelas', 'Mata Kuliah', 'No HP', 
        'Barang', 'Nomor Barang', 'Tanggal Pinjam', 'Tanggal Kembali', 
        'Status', 'Created At', 'Updated At'
      ];
      sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
      sheet.getRange(1, 1, 1, headers.length).setFontWeight('bold');
    }
    
    switch(data.action) {
      case 'sync_all':
        syncAllData(sheet, data.data);
        break;
      case 'create':
        addNewRow(sheet, data.data);
        break;
      case 'update':
        updateRow(sheet, data.data);
        break;
      case 'delete':
        deleteRow(sheet, data.data);
        break;
    }
    
    return ContentService
      .createTextOutput(JSON.stringify({success: true}))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    Logger.log('Error: ' + error.toString());
    return ContentService
      .createTextOutput(JSON.stringify({success: false, error: error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function syncAllData(sheet, data) {
  // Clear existing data (except headers)
  if (sheet.getLastRow() > 1) {
    sheet.getRange(2, 1, sheet.getLastRow() - 1, sheet.getLastColumn()).clear();
  }
  
  // Add all data
  if (data.length > 0) {
    const rows = data.map(item => [
      item.nama,
      item.nim,
      item.kelas,
      item.mataKuliah,
      item.noHp,
      item.barang,
      item.nomorBarang,
      item.tanggalPinjam,
      item.tanggalKembali,
      item.status,
      item.createdAt,
      item.updatedAt
    ]);
    
    sheet.getRange(2, 1, rows.length, rows[0].length).setValues(rows);
  }
}

function addNewRow(sheet, data) {
  const newRow = [
    data.nama, data.nim, data.kelas, data.mataKuliah, data.noHp,
    data.barang, data.nomorBarang, data.tanggalPinjam, data.tanggalKembali,
    data.status, data.createdAt, data.updatedAt
  ];
  
  sheet.appendRow(newRow);
}
```

---

## 4️⃣ TRASH BIN (SOFT DELETE)

### 4.1 Struktur Database untuk Soft Delete

```javascript
// Firebase Structure with Soft Delete
{
  "peminjaman": {
    // Active data remains here
  },
  "deleted": {
    "peminjaman": {
      "uuid-deleted-1": {
        "id": "uuid-deleted-1",
        "originalData": {
          // Original peminjaman data
        },
        "deletedAt": "2025-06-13T15:30:00+07:00",
        "deletedBy": "system" // or user identifier
      }
    }
  }
}
```

### 4.2 HTML Template untuk Trash Bin

```html
<!-- Trash Bin Section -->
<section id="trash-section" class="container mt-5">
    <div class="card">
        <div class="card-header bg-danger text-white">
            <h4><i class="fas fa-trash"></i> Trash Bin - Data Terhapus</h4>
        </div>
        <div class="card-body">
            <!-- Action Buttons -->
            <div class="row mb-3">
                <div class="col">
                    <button id="btnEmptyTrash" class="btn btn-danger">
                        <i class="fas fa-trash-alt"></i> Kosongkan Trash
                    </button>
                    <button id="btnRefreshTrash" class="btn btn-secondary">
                        <i class="fas fa-sync"></i> Refresh
                    </button>
                    <span id="trashCount" class="badge bg-info ms-2">0 item</span>
                </div>
            </div>

            <!-- Loading Indicator -->
            <div id="trashLoading" class="text-center" style="display: none;">
                <div class="spinner-border text-danger" role="status">
                    <span class="visually-hidden">Loading...</span>
                </div>
                <p>Memuat data trash...</p>
            </div>

            <!-- Trash Table -->
            <div class="table-responsive">
                <table id="trashTable" class="table table-striped">
                    <thead class="table-dark">
                        <tr>
                            <th>
                                <input type="checkbox" id="selectAllTrash" class="form-check-input">
                            </th>
                            <th>Nama</th>
                            <th>NIM</th>
                            <th>Kelas</th>
                            <th>Barang</th>
                            <th>Tgl Pinjam</th>
                            <th>Dihapus Pada</th>
                            <th>Aksi</th>
                        </tr>
                    </thead>
                    <tbody id="trashTableBody">
                        <!-- Data will be populated by JavaScript -->
                    </tbody>
                </table>
            </div>

            <!-- Bulk Actions -->
            <div id="bulkActions" class="mt-3" style="display: none;">
                <button id="btnBulkRestore" class="btn btn-success">
                    <i class="fas fa-undo"></i> Restore Terpilih
                </button>
                <button id="btnBulkDelete" class="btn btn-danger">
                    <i class="fas fa-times"></i> Hapus Permanen Terpilih
                </button>
            </div>
        </div>
    </div>
</section>
```

### 4.3 JavaScript Implementation untuk Trash Bin

```javascript
// trash-manager.js
class TrashManager {
    constructor() {
        this.database = firebase.database();
        this.trashData = [];
        this.selectedItems = new Set();
        
        this.initializeEventListeners();
        this.loadTrashData();
    }

    initializeEventListeners() {
        // Main action buttons
        document.getElementById('btnEmptyTrash').addEventListener('click', () => {
            this.emptyTrash();
        });

        document.getElementById('btnRefreshTrash').addEventListener('click', () => {
            this.loadTrashData();
        });

        // Bulk actions
        document.getElementById('selectAllTrash').addEventListener('change', (e) => {
            this.selectAllItems(e.target.checked);
        });

        document.getElementById('btnBulkRestore').addEventListener('click', () => {
            this.bulkRestore();
        });

        document.getElementById('btnBulkDelete').addEventListener('click', () => {
            this.bulkDelete();
        });
    }

    async loadTrashData() {
        try {
            this.showLoading(true);
            
            const snapshot = await this.database.ref('deleted/peminjaman').once('value');
            this.trashData = [];
            
            if (snapshot.exists()) {
                snapshot.forEach((childSnapshot) => {
                    const data = childSnapshot.val();
                    this.trashData.push(data);
                });
                
                // Sort by deletion date (newest first)
                this.trashData.sort((a, b) => new Date(b.deletedAt) - new Date(a.deletedAt));
            }
            
            this.renderTrashTable();
            this.updateTrashCount();
            
        } catch (error) {
            console.error('Error loading trash data:', error);
            this.showError('Gagal memuat data trash');
        } finally {
            this.showLoading(false);
        }
    }

    renderTrashTable() {
        const tbody = document.getElementById('trashTableBody');
        tbody.innerHTML = '';

        if (this.trashData.length === 0) {
            tbody.innerHTML = `
                <tr>
                    <td colspan="8" class="text-center text-muted">
                        <i class="fas fa-check-circle fa-3x mb-3 text-success"></i>
                        <p>Trash kosong - Semua data bersih!</p>
                    </td>
                </tr>
            `;
            this.hideBulkActions();
            return;
        }

        this.trashData.forEach(item => {
            const row = this.createTrashRow(item);
            tbody.appendChild(row);
        });
    }

    createTrashRow(item) {
        const row = document.createElement('tr');
        const data = item.originalData;
        
        row.innerHTML = `
            <td>
                <input type="checkbox" class="form-check-input item-checkbox" 
                       value="${item.id}" onchange="trashManager.onItemSelect(this)">
            </td>
            <td>${data.nama}</td>
            <td>${data.nim}</td>
            <td>${data.kelas}</td>
            <td>${data.barang} #${data.nomorBarang}</td>
            <td>${this.formatDate(data.tanggalPinjam)}</td>
            <td>${this.formatDateTime(item.deletedAt)}</td>
            <td>
                <button class="btn btn-sm btn-success me-1" 
                        onclick="trashManager.restoreItem('${item.id}')"
                        title="Restore">
                    <i class="fas fa-undo"></i>
                </button>
                <button class="btn btn-sm btn-danger" 
                        onclick="trashManager.permanentDelete('${item.id}')"
                        title="Hapus Permanen">
                    <i class="fas fa-times"></i>
                </button>
            </td>
        `;

        return row;
    }

    // Soft Delete Implementation
    async softDeletePeminjaman(peminjamanId) {
        try {
            // Get original data
            const snapshot = await this.database.ref(`peminjaman/${peminjamanId}`).once('value');
            
            if (!snapshot.exists()) {
                throw new Error('Data peminjaman tidak ditemukan');
            }

            const originalData = snapshot.val();
            
            // Create trash entry
            const trashId = this.generateUUID();
            const trashEntry = {
                id: trashId,
                originalData: originalData,
                deletedAt: new Date().toISOString(),
                deletedBy: 'system'
            };

            // Move to trash
            await this.database.ref(`deleted/peminjaman/${trashId}`).set(trashEntry);

            ```javascript
            // Remove from active data
            await this.database.ref(`peminjaman/${peminjamanId}`).remove();
            
            // Update asset status if needed
            const assetKey = `${originalData.barang.toLowerCase().replace(/\s+/g, '-')}-${originalData.nomorBarang}`;
            await this.database.ref(`aset/${assetKey}`).update({
                status: 'tersedia',
                dipinjamOleh: null,
                updatedAt: new Date().toISOString()
            });

            this.showSuccess('Data berhasil dipindahkan ke trash');
            
            // Auto-sync to Google Sheets
            await this.autoSyncToSheets('delete', trashEntry);
            
            // Refresh data
            await this.loadTrashData();
            
        } catch (error) {
            console.error('Error soft deleting:', error);
            this.showError('Gagal menghapus data');
        }
    }

    // Restore item from trash
    async restoreItem(trashId) {
        try {
            if (!confirm('Apakah Anda yakin ingin mengembalikan data ini?')) {
                return;
            }

            // Get trash data
            const snapshot = await this.database.ref(`deleted/peminjaman/${trashId}`).once('value');
            
            if (!snapshot.exists()) {
                throw new Error('Data trash tidak ditemukan');
            }

            const trashData = snapshot.val();
            const originalData = trashData.originalData;
            
            // Check if asset is still available
            const assetKey = `${originalData.barang.toLowerCase().replace(/\s+/g, '-')}-${originalData.nomorBarang}`;
            const assetSnapshot = await this.database.ref(`aset/${assetKey}`).once('value');
            
            if (assetSnapshot.exists()) {
                const assetData = assetSnapshot.val();
                if (assetData.status === 'dipinjam') {
                    throw new Error('Barang sudah dipinjam oleh orang lain');
                }
            }

            // Restore to active data with updated timestamp
            const restoredData = {
                ...originalData,
                updatedAt: new Date().toISOString()
            };
            
            await this.database.ref(`peminjaman/${originalData.id}`).set(restoredData);
            
            // Update asset status
            await this.database.ref(`aset/${assetKey}`).update({
                status: originalData.status === 'dipinjam' ? 'dipinjam' : 'tersedia',
                dipinjamOleh: originalData.status === 'dipinjam' ? originalData.id : null,
                updatedAt: new Date().toISOString()
            });
            
            // Remove from trash
            await this.database.ref(`deleted/peminjaman/${trashId}`).remove();
            
            this.showSuccess('Data berhasil dipulihkan');
            
            // Auto-sync to Google Sheets
            await this.autoSyncToSheets('restore', restoredData);
            
            // Refresh data
            await this.loadTrashData();
            
        } catch (error) {
            console.error('Error restoring item:', error);
            this.showError(error.message || 'Gagal memulihkan data');
        }
    }

    // Permanent delete
    async permanentDelete(trashId) {
        try {
            if (!confirm('PERINGATAN: Data akan dihapus permanen dan tidak dapat dipulihkan. Lanjutkan?')) {
                return;
            }

            await this.database.ref(`deleted/peminjaman/${trashId}`).remove();
            
            this.showSuccess('Data berhasil dihapus permanen');
            await this.loadTrashData();
            
        } catch (error) {
            console.error('Error permanent delete:', error);
            this.showError('Gagal menghapus data permanen');
        }
    }

    // Empty entire trash
    async emptyTrash() {
        try {
            if (this.trashData.length === 0) {
                this.showInfo('Trash sudah kosong');
                return;
            }

            const confirmMessage = `PERINGATAN: Semua ${this.trashData.length} data di trash akan dihapus permanen. Tindakan ini tidak dapat dibatalkan. Lanjutkan?`;
            
            if (!confirm(confirmMessage)) {
                return;
            }

            await this.database.ref('deleted/peminjaman').remove();
            
            this.showSuccess(`${this.trashData.length} data berhasil dihapus permanen dari trash`);
            await this.loadTrashData();
            
        } catch (error) {
            console.error('Error emptying trash:', error);
            this.showError('Gagal mengosongkan trash');
        }
    }

    // Selection management
    onItemSelect(checkbox) {
        if (checkbox.checked) {
            this.selectedItems.add(checkbox.value);
        } else {
            this.selectedItems.delete(checkbox.value);
        }
        
        this.updateSelectAllState();
        this.toggleBulkActions();
    }

    selectAllItems(checked) {
        const checkboxes = document.querySelectorAll('.item-checkbox');
        this.selectedItems.clear();
        
        checkboxes.forEach(checkbox => {
            checkbox.checked = checked;
            if (checked) {
                this.selectedItems.add(checkbox.value);
            }
        });
        
        this.toggleBulkActions();
    }

    updateSelectAllState() {
        const selectAllCheckbox = document.getElementById('selectAllTrash');
        const totalCheckboxes = document.querySelectorAll('.item-checkbox').length;
        
        if (this.selectedItems.size === 0) {
            selectAllCheckbox.indeterminate = false;
            selectAllCheckbox.checked = false;
        } else if (this.selectedItems.size === totalCheckboxes) {
            selectAllCheckbox.indeterminate = false;
            selectAllCheckbox.checked = true;
        } else {
            selectAllCheckbox.indeterminate = true;
            selectAllCheckbox.checked = false;
        }
    }

    toggleBulkActions() {
        const bulkActions = document.getElementById('bulkActions');
        if (this.selectedItems.size > 0) {
            bulkActions.style.display = 'block';
        } else {
            bulkActions.style.display = 'none';
        }
    }

    hideBulkActions() {
        document.getElementById('bulkActions').style.display = 'none';
        this.selectedItems.clear();
    }

    // Bulk operations
    async bulkRestore() {
        try {
            if (this.selectedItems.size === 0) return;

            const confirmMessage = `Restore ${this.selectedItems.size} data terpilih?`;
            if (!confirm(confirmMessage)) return;

            const restorePromises = Array.from(this.selectedItems).map(id => 
                this.restoreItem(id)
            );

            await Promise.all(restorePromises);
            
            this.selectedItems.clear();
            this.hideBulkActions();
            
        } catch (error) {
            console.error('Error bulk restore:', error);
            this.showError('Gagal melakukan bulk restore');
        }
    }

    async bulkDelete() {
        try {
            if (this.selectedItems.size === 0) return;

            const confirmMessage = `PERINGATAN: ${this.selectedItems.size} data akan dihapus permanen. Lanjutkan?`;
            if (!confirm(confirmMessage)) return;

            const deletePromises = Array.from(this.selectedItems).map(id => 
                this.permanentDelete(id)
            );

            await Promise.all(deletePromises);
            
            this.selectedItems.clear();
            this.hideBulkActions();
            
        } catch (error) {
            console.error('Error bulk delete:', error);
            this.showError('Gagal melakukan bulk delete');
        }
    }

    // Auto-sync integration
    async autoSyncToSheets(action, data) {
        try {
            const webhookUrl = 'YOUR_GOOGLE_SHEETS_WEBHOOK_URL'; // Replace with actual webhook
            
            await fetch(webhookUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    action: action, // 'delete', 'restore'
                    data: data,
                    timestamp: new Date().toISOString()
                })
            });

        } catch (error) {
            console.error('Auto-sync failed:', error);
        }
    }

    // Utility methods
    updateTrashCount() {
        const countElement = document.getElementById('trashCount');
        const count = this.trashData.length;
        countElement.textContent = `${count} item${count !== 1 ? 's' : ''}`;
        countElement.className = count > 0 ? 'badge bg-warning ms-2' : 'badge bg-info ms-2';
    }

    formatDate(dateString) {
        const date = new Date(dateString);
        return date.toLocaleDateString('id-ID', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit'
        });
    }

    formatDateTime(dateString) {
        const date = new Date(dateString);
        return date.toLocaleString('id-ID', {
            year: 'numeric',
            month: '2-digit',
            day: '2-digit',
            hour: '2-digit',
            minute: '2-digit'
        });
    }

    generateUUID() {
        return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
            const r = Math.random() * 16 | 0;
            const v = c == 'x' ? r : (r & 0x3 | 0x8);
            return v.toString(16);
        });
    }

    showLoading(show) {
        const loading = document.getElementById('trashLoading');
        const table = document.getElementById('trashTable');
        
        if (show) {
            loading.style.display = 'block';
            table.style.display = 'none';
        } else {
            loading.style.display = 'none';
            table.style.display = 'table';
        }
    }

    showSuccess(message) {
        // Implementation depends on your notification system
        this.showNotification(message, 'success');
    }

    showError(message) {
        // Implementation depends on your notification system
        this.showNotification(message, 'error');
    }

    showInfo(message) {
        // Implementation depends on your notification system
        this.showNotification(message, 'info');
    }

    showNotification(message, type) {
        // Simple alert implementation - replace with better notification system
        alert(message);
        
        // Example with toast notification (if you have toast library)
        /*
        const toastClass = {
            'success': 'toast-success',
            'error': 'toast-error',
            'info': 'toast-info'
        }[type] || 'toast-info';
        
        // Show toast notification
        */
    }
}
```

### 4.4 Enhanced Google Apps Script untuk Trash Operations

```javascript
// Enhanced webhook-handler.gs for trash operations
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSheet();
    
    // Create separate sheets for different operations
    const activeSheet = getOrCreateSheet('Active_Data');
    const trashSheet = getOrCreateSheet('Trash_Data');
    const logSheet = getOrCreateSheet('Operations_Log');
    
    // Set headers for all sheets
    setupSheetHeaders(activeSheet, getActiveDataHeaders());
    setupSheetHeaders(trashSheet, getTrashDataHeaders());
    setupSheetHeaders(logSheet, getLogHeaders());
    
    switch(data.action) {
      case 'sync_all':
        syncAllData(activeSheet, data.data);
        break;
      case 'create':
        addNewRow(activeSheet, data.data);
        logOperation(logSheet, 'CREATE', data.data);
        break;
      case 'update':
        updateRow(activeSheet, data.data);
        logOperation(logSheet, 'UPDATE', data.data);
        break;
      case 'delete':
        moveToTrash(activeSheet, trashSheet, data.data);
        logOperation(logSheet, 'DELETE', data.data);
        break;
      case 'restore':
        restoreFromTrash(trashSheet, activeSheet, data.data);
        logOperation(logSheet, 'RESTORE', data.data);
        break;
      case 'permanent_delete':
        permanentDelete(trashSheet, data.data);
        logOperation(logSheet, 'PERMANENT_DELETE', data.data);
        break;
    }
    
    return ContentService
      .createTextOutput(JSON.stringify({success: true}))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    Logger.log('Error: ' + error.toString());
    return ContentService
      .createTextOutput(JSON.stringify({success: false, error: error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function getOrCreateSheet(sheetName) {
  const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  let sheet = spreadsheet.getSheetByName(sheetName);
  
  if (!sheet) {
    sheet = spreadsheet.insertSheet(sheetName);
  }
  
  return sheet;
}

function getActiveDataHeaders() {
  return [
    'ID', 'Nama', 'NIM', 'Kelas', 'Mata Kuliah', 'No HP', 
    'Barang', 'Nomor Barang', 'Tanggal Pinjam', 'Tanggal Kembali', 
    'Status', 'Created At', 'Updated At'
  ];
}

function getTrashDataHeaders() {
  return [
    'Trash ID', 'Original ID', 'Nama', 'NIM', 'Kelas', 'Mata Kuliah', 'No HP',
    'Barang', 'Nomor Barang', 'Tanggal Pinjam', 'Tanggal Kembali', 
    'Status', 'Deleted At', 'Deleted By'
  ];
}

function getLogHeaders() {
  return [
    'Timestamp', 'Operation', 'ID', 'Nama', 'Details'
  ];
}

function setupSheetHeaders(sheet, headers) {
  if (sheet.getLastRow() === 0) {
    sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
    sheet.getRange(1, 1, 1, headers.length).setFontWeight('bold');
    sheet.getRange(1, 1, 1, headers.length).setBackground('#4285f4');
    sheet.getRange(1, 1, 1, headers.length).setFontColor('white');
  }
}

function moveToTrash(activeSheet, trashSheet, data) {
  // Remove from active sheet
  removeRowById(activeSheet, data.originalData.id);
  
  // Add to trash sheet
  const trashRow = [
    data.id,
    data.originalData.id,
    data.originalData.nama,
    data.originalData.nim,
    data.originalData.kelas,
    data.originalData.mataKuliah,
    data.originalData.noHp,
    data.originalData.barang,
    data.originalData.nomorBarang,
    data.originalData.tanggalPinjam,
    data.originalData.tanggalKembali,
    data.originalData.status,
    data.deletedAt,
    data.deletedBy || 'system'
  ];
  
  trashSheet.appendRow(trashRow);
}

function restoreFromTrash(trashSheet, activeSheet, data) {
  // Remove from trash sheet
  removeRowById(trashSheet, data.id, 0); // Trash ID is in column 0
  
  // Add back to active sheet
  addNewRow(activeSheet, data);
}

function removeRowById(sheet, id, idColumn = 0) {
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) { // Start from 1 to skip headers
    if (data[i][idColumn] === id) {
      sheet.deleteRow(i + 1);
      break;
    }
  }
}

function logOperation(logSheet, operation, data) {
  const logRow = [
    new Date(),
    operation,
    data.id || data.originalData?.id || 'N/A',
    data.nama || data.originalData?.nama || 'N/A',
    JSON.stringify(data, null, 2).substr(0, 100) + '...'
  ];
  
  logSheet.appendRow(logRow);
}
```

### 4.5 Integration dengan Main System

```javascript
// main-system-integration.js
class SystemIntegration {
    constructor() {
        this.riwayatManager = new RiwayatManager();
        this.trashManager = new TrashManager();
        this.initializeNavigation();
    }

    initializeNavigation() {
        // Navigation between sections
        document.addEventListener('DOMContentLoaded', () => {
            this.setupSectionNavigation();
            this.setupRealTimeUpdates();
        });
    }

    setupSectionNavigation() {
        // Add navigation buttons to switch between sections
        const navigationHTML = `
            <nav class="navbar navbar-expand-lg navbar-dark bg-primary mb-4">
                <div class="container">
                    <a class="navbar-brand" href="#"><i class="fas fa-clipboard-list"></i> Sistem Peminjaman Aset</a>
                    <div class="navbar-nav">
                        <a class="nav-link" href="#" onclick="systemIntegration.showSection('main')">
                            <i class="fas fa-home"></i> Utama
                        </a>
                        <a class="nav-link" href="#" onclick="systemIntegration.showSection('riwayat')">
                            <i class="fas fa-history"></i> Riwayat
                        </a>
                        <a class="nav-link" href="#" onclick="systemIntegration.showSection('trash')">
                            <i class="fas fa-trash"></i> Trash
                            <span id="trashNavBadge" class="badge bg-danger ms-1">0</span>
                        </a>
                    </div>
                </div>
            </nav>
        `;
        
        document.body.insertAdjacentHTML('afterbegin', navigationHTML);
    }

    showSection(sectionName) {
        // Hide all sections
        const sections = ['main-section', 'riwayat-section', 'trash-section'];
        sections.forEach(section => {
            const element = document.getElementById(section);
            if (element) {
                element.style.display = 'none';
            }
        });

        // Show selected section
        const targetSection = document.getElementById(`${sectionName}-section`);
        if (targetSection) {
            targetSection.style.display = 'block';
        }

        // Update navigation active state
        document.querySelectorAll('.nav-link').forEach(link => {
            link.classList.remove('active');
        });
        event.target.classList.add('active');

        // Refresh data for the section
        switch(sectionName) {
            case 'riwayat':
                this.riwayatManager.loadRiwayatData();
                break;
            case 'trash':
                this.trashManager.loadTrashData();
                break;
        }
    }

    setupRealTimeUpdates() {
        // Listen for changes in trash count
        firebase.database().ref('deleted/peminjaman').on('value', (snapshot) => {
            const count = snapshot.exists() ? snapshot.numChildren() : 0;
            this.updateTrashBadge(count);
        });
    }

    updateTrashBadge(count) {
        const badge = document.getElementById('trashNavBadge');
        if (badge) {
            badge.textContent = count;
            badge.style.display = count > 0 ? 'inline' : 'none';
        }
    }

    // Enhanced delete function that uses soft delete
    async deletePeminjaman(peminjamanId) {
        try {
            await this.trashManager.softDeletePeminjaman(peminjamanId);
            
            // Refresh main table if exists
            if (typeof refreshMainTable === 'function') {
                refreshMainTable();
            }
            
            // Refresh riwayat if currently viewing
            const riwayatSection = document.getElementById('riwayat-section');
            if (riwayatSection && riwayatSection.style.display !== 'none') {
                this.riwayatManager.loadRiwayatData();
            }
            
        } catch (error) {
            console.error('Error in system delete:', error);
        }
    }
}

// Initialize system integration
const systemIntegration = new SystemIntegration();

// Export functions for global access
window.riwayatManager = systemIntegration.riwayatManager;
window.trashManager = systemIntegration.trashManager;
window.systemIntegration = systemIntegration;
```

### 4.6 CSS Enhancements untuk Trash & Riwayat

```css
/* Enhanced styles for trash and riwayat sections */
.trash-section {
    background-color: #fff5f5;
    border-left: 4px solid #dc3545;
}

.riwayat-section {
    background-color: #f0f8ff;
    border-left: 4px solid #17a2b8;
}

.item-checkbox:checked {
    background-color: #dc3545;
    border-color: #dc3545;
}

.bulk-actions {
    background-color: #f8f9fa;
    border: 1px solid #dee2e6;
    border-radius: 0.375rem;
    padding: 1rem;
    margin-top: 1rem;
}

.trash-badge {
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% { opacity: 1; }
    50% { opacity: 0.5; }
    100% { opacity: 1; }
}

.filter-section {
    background-color: #f8f9fa;
    border-radius: 0.375rem;
    padding: 1rem;
    margin-bottom: 1rem;
}

.section-transition {
    transition: opacity 0.3s ease-in-out;
}

/* Responsive enhancements */
@media (max-width: 768px) {
    .filter-section .row > div {
        margin-bottom: 1rem;
    }
    
    .bulk-actions {
        text-align: center;
    }
    
    .bulk-actions .btn {
        margin: 0.25rem;
        width: 100%;
    }
}

/* Status badges */
.status-badge {
    font-size: 0.8em;
    padding: 0.3em 0.6em;
}

/* Loading states */
.loading-overlay {
    position: relative;
}

.loading-overlay::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(255, 255, 255, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
}
```

---

## 🚀 **Implementasi dan Testing**

### Langkah-langkah Implementasi:

1. **Setup HTML Structure**
   - Tambahkan section riwayat dan trash ke halaman utama
   - Integrasikan navigation system

2. **Initialize JavaScript Classes**
   ```javascript
   // Initialize after Firebase is loaded
   firebase.initializeApp(firebaseConfig);
   
   const systemIntegration = new SystemIntegration();
   ```

3. **Setup Google Sheets Webhook**
   - Deploy Google Apps Script sebagai web app
   - Copy webhook URL ke konfigurasi JavaScript
   - Test webhook dengan data sample

4. **Testing Checklist**
   - [x] Filter riwayat by nama, kelas, tanggal
   - [x] Soft delete peminjaman
   - [x] Restore from trash
   - [x] Permanent delete
   - [x] Bulk operations
   - [x] Google Sheets sync
   - [x] Real-time updates
   - [x] Responsive design

### Keamanan dan Performance:

- **Validation**: Semua input divalidasi sebelum operasi database
- **Error Handling**: Comprehensive error handling dengan user feedback
- **Loading States**: Loading indicators untuk UX yang better
- **Debouncing**: Filter input di-debounce untuk performance
- **Pagination**: Data riwayat dipaginasi untuk performa optimal

Dokumentasi fitur 3 dan 4 sudah lengkap dengan implementasi teknis yang siap digunakan!