# Sistem Manajemen Peminjaman Aset //TALMS (Technology Asset Loan Management System)
## Model Data & Logika Validasi

---

## 1️⃣ Model Data Firebase Realtime Database

### Struktur Database JSON

```json
{
  "aset": {
    "aset_001": {
      "id": "aset_001",
      "namaBarang": "Proyektor EPSON",
      "nomorBarang": "01",
      "status": "Tersedia",
      "tanggalTambah": "2024-01-15T10:30:00.000Z"
    },
    "aset_002": {
      "id": "aset_002", 
      "namaBarang": "Converter ROBOT RHV01",
      "nomorBarang": "01",
      "status": "Dipinjam",
      "tanggalTambah": "2024-01-15T10:30:00.000Z"
    }
  },
  "peminjaman": {
    "peminjaman_001": {
      "id": "peminjaman_001",
      "nim": "236151098",
      "namaLengkap": "Ahmad Fajar Sidiq",
      "noHp": "082290708076",
      "mataPelajaran": "Pemrograman Web",
      "kelas": "TI-3A",
      "tanggalPinjam": "2024-06-13",
      "jamPinjam": "08:30",
      "tanggalKembali": "2024-06-15",
      "status": "Dipinjam",
      "asetRef": "aset_002",
      "namaBarang": "Converter ROBOT RHV01",
      "nomorBarang": "01"
    }
  },
  "deleted": {
    "peminjaman": {
      // Soft deleted peminjaman records
    }
  }
}
```

### Koleksi `aset`

| Field | Type | Deskripsi | Contoh |
|-------|------|-----------|---------|
| `id` | string | UUID unik untuk aset | "aset_001" |
| `namaBarang` | string | Nama/jenis barang | "Proyektor EPSON" |
| `nomorBarang` | string | Nomor urut barang | "01", "02", "03" |
| `status` | string | Status ketersediaan | "Tersedia" / "Dipinjam" |
| `tanggalTambah` | string (ISO) | Tanggal aset ditambahkan | "2024-01-15T10:30:00.000Z" |

### Koleksi `peminjaman`

| Field | Type | Deskripsi | Validasi | Contoh |
|-------|------|-----------|----------|---------|
| `id` | string | UUID unik peminjaman | Required | "peminjaman_001" |
| `nim` | string | NIM mahasiswa | 9 digit, required | "236151098" |
| `namaLengkap` | string | Nama lengkap peminjam | Required, min 3 char | "Ahmad Fajar Sidiq" |
| `noHp` | string | Nomor HP peminjam | 12 digit, required | "082290708076" |
| `mataPelajaran` | string | Mata kuliah/pelajaran | Required | "Pemrograman Web" |
| `kelas` | string | Kelas peminjam | Required | "TI-3A" |
| `tanggalPinjam` | string (YYYY-MM-DD) | Tanggal peminjaman | Required, ≤ today | "2024-06-13" |
| `jamPinjam` | string (HH:mm) | Jam peminjaman | Required | "08:30" |
| `tanggalKembali` | string (YYYY-MM-DD) | Tanggal rencana kembali | Required, ≥ tanggalPinjam | "2024-06-15" |
| `status` | string | Status peminjaman | "Dipinjam" / "Dikembalikan" | "Dipinjam" |
| `asetRef` | string | Referensi ke ID aset | Required, must exist | "aset_002" |
| `namaBarang` | string | Salinan nama barang dari aset | Auto-filled | "Converter ROBOT RHV01" |
| `nomorBarang` | string | Salinan nomor barang dari aset | Auto-filled | "01" |

---

## 2️⃣ Logika Validasi Data

### Fungsi Validasi Input

```javascript
function validatePeminjamanData(data) {
    const errors = [];
    
    // Validasi NIM (9 digit)
    if (!data.nim || !/^\d{9}$/.test(data.nim)) {
        errors.push("NIM harus 9 digit angka");
    }
    
    // Validasi Nama Lengkap
    if (!data.namaLengkap || data.namaLengkap.trim().length < 3) {
        errors.push("Nama lengkap minimal 3 karakter");
    }
    
    // Validasi No HP (12 digit)
    if (!data.noHp || !/^\d{12}$/.test(data.noHp)) {
        errors.push("Nomor HP harus 12 digit angka");
    }
    
    // Validasi Mata Pelajaran
    if (!data.mataPelajaran || data.mataPelajaran.trim().length === 0) {
        errors.push("Mata pelajaran wajib diisi");
    }
    
    // Validasi Kelas
    if (!data.kelas || data.kelas.trim().length === 0) {
        errors.push("Kelas wajib diisi");
    }
    
    // Validasi Tanggal Pinjam
    if (!data.tanggalPinjam) {
        errors.push("Tanggal pinjam wajib diisi");
    } else {
        const today = new Date().toISOString().split('T')[0];
        if (data.tanggalPinjam > today) {
            errors.push("Tanggal pinjam tidak boleh lebih dari hari ini");
        }
    }
    
    // Validasi Jam Pinjam
    if (!data.jamPinjam || !/^([01]\d|2[0-3]):([0-5]\d)$/.test(data.jamPinjam)) {
        errors.push("Format jam pinjam tidak valid (HH:mm)");
    }
    
    // Validasi Tanggal Kembali
    if (!data.tanggalKembali) {
        errors.push("Tanggal kembali wajib diisi");
    } else if (data.tanggalPinjam && data.tanggalKembali < data.tanggalPinjam) {
        errors.push("Tanggal kembali harus sama atau setelah tanggal pinjam");
    }
    
    // Validasi Aset Reference
    if (!data.asetRef) {
        errors.push("Aset yang dipinjam wajib dipilih");
    }
    
    return {
        isValid: errors.length === 0,
        errors: errors
    };
}
```

### Fungsi Validasi Ketersediaan Aset

```javascript
async function validateAsetAvailability(asetId) {
    try {
        const asetSnapshot = await database.ref(`aset/${asetId}`).once('value');
        const asetData = asetSnapshot.val();
        
        if (!asetData) {
            return {
                isAvailable: false,
                message: "Aset tidak ditemukan"
            };
        }
        
        if (asetData.status !== "Tersedia") {
            return {
                isAvailable: false,
                message: `${asetData.namaBarang} No.${asetData.nomorBarang} sedang dipinjam`
            };
        }
        
        return {
            isAvailable: true,
            asetData: asetData
        };
    } catch (error) {
        return {
            isAvailable: false,
            message: "Error validasi aset: " + error.message
        };
    }
}
```

---

## 3️⃣ Logika Update Otomatis Status Aset

### Fungsi Create Peminjaman (Auto Update Status)

```javascript
async function createPeminjaman(peminjamanData) {
    try {
        // 1. Validasi data input
        const validation = validatePeminjamanData(peminjamanData);
        if (!validation.isValid) {
            throw new Error(validation.errors.join(", "));
        }
        
        // 2. Validasi ketersediaan aset
        const availability = await validateAsetAvailability(peminjamanData.asetRef);
        if (!availability.isAvailable) {
            throw new Error(availability.message);
        }
        
        // 3. Generate ID peminjaman
        const peminjamanId = generateUUID();
        
        // 4. Prepare data peminjaman dengan salinan data aset
        const finalPeminjamanData = {
            ...peminjamanData,
            id: peminjamanId,
            status: "Dipinjam",
            namaBarang: availability.asetData.namaBarang,
            nomorBarang: availability.asetData.nomorBarang
        };
        
        // 5. Update dalam satu transaksi atomic
        const updates = {};
        updates[`peminjaman/${peminjamanId}`] = finalPeminjamanData;
        updates[`aset/${peminjamanData.asetRef}/status`] = "Dipinjam";
        
        await database.ref().update(updates);
        
        return {
            success: true,
            data: finalPeminjamanData,
            message: "Peminjaman berhasil dicatat"
        };
        
    } catch (error) {
        return {
            success: false,
            message: error.message
        };
    }
}
```

### Fungsi Update Status Peminjaman ke "Dikembalikan"

```javascript
async function returnPeminjaman(peminjamanId) {
    try {
        // 1. Ambil data peminjaman
        const peminjamanSnapshot = await database.ref(`peminjaman/${peminjamanId}`).once('value');
        const peminjamanData = peminjamanSnapshot.val();
        
        if (!peminjamanData) {
            throw new Error("Data peminjaman tidak ditemukan");
        }
        
        if (peminjamanData.status === "Dikembalikan") {
            throw new Error("Barang sudah dikembalikan sebelumnya");
        }
        
        // 2. Update status dalam satu transaksi atomic
        const updates = {};
        updates[`peminjaman/${peminjamanId}/status`] = "Dikembalikan";
        updates[`aset/${peminjamanData.asetRef}/status`] = "Tersedia";
        
        await database.ref().update(updates);
        
        return {
            success: true,
            message: `${peminjamanData.namaBarang} No.${peminjamanData.nomorBarang} berhasil dikembalikan`
        };
        
    } catch (error) {
        return {
            success: false,
            message: error.message
        };
    }
}
```

### Fungsi Helper Generate UUID

```javascript
function generateUUID() {
    return 'xxxx-xxxx-4xxx-yxxx-xxxx'.replace(/[xy]/g, function(c) {
        const r = Math.random() * 16 | 0;
        const v = c == 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
```

---

## 4️⃣ Pseudocode Alur Sistem

### Alur Create Peminjaman

```
MULAI CreatePeminjaman(data)
├── VALIDASI input data
│   ├── Jika TIDAK valid → RETURN error
│   └── Jika valid → LANJUT
├── CEK ketersediaan aset
│   ├── Jika aset tidak tersedia → RETURN error  
│   └── Jika tersedia → LANJUT
├── GENERATE ID peminjaman
├── PREPARE data final (tambah salinan data aset)
├── UPDATE ATOMIC:
│   ├── INSERT peminjaman baru
│   └── UPDATE status aset = "Dipinjam"
└── RETURN success
SELESAI
```

### Alur Return Peminjaman

```
MULAI ReturnPeminjaman(peminjamanId)
├── AMBIL data peminjaman berdasarkan ID
├── VALIDASI data peminjaman exists & status = "Dipinjam"
│   ├── Jika TIDAK valid → RETURN error
│   └── Jika valid → LANJUT  
├── UPDATE ATOMIC:
│   ├── UPDATE status peminjaman = "Dikembalikan"
│   └── UPDATE status aset = "Tersedia"
└── RETURN success
SELESAI
```

---

## 5️⃣ Catatan Implementasi

### Firebase Realtime Database Rules (Public)

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

### Keamanan Data

Meskipun sistem bersifat publik tanpa login, pertimbangkan:

1. **Rate Limiting**: Batasi jumlah request per IP untuk mencegah spam
2. **Data Sanitization**: Bersihkan input untuk mencegah injection
3. **Backup Berkala**: Lakukan backup database secara otomatis
4. **Monitoring**: Pantau aktivitas unusual pada database

### Optimasi Performa

1. **Indexing**: Buat index untuk field yang sering di-query (status, tanggalPinjam)
2. **Pagination**: Implementasi lazy loading untuk tabel besar
3. **Caching**: Cache data aset yang jarang berubah di localStorage