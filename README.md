# SiCepat Ekspres — First-Mile Logistics Analytics

Data Cleaning, Feature Engineering & Exploratory Data Analysis atas data operasional First-Mile Pickup SiCepat Ekspres, dengan fokus mendeteksi **Phantom Pickup** dan **revenue leakage** dari manipulasi berat paket.

---

## Konteks Bisnis

SiCepat Ekspres merebut hati jutaan seller UMKM lewat layanan First-Mile Pickup gratis dan paket murah HALU. Namun pertumbuhan masif ini memunculkan dua masalah kritis:

1. **Phantom Pickup** — kurir menekan tombol "Pickup Selesai" di aplikasi sebelum seller benar-benar memanggil pickup, demi menghindari denda KPI.
2. **Revenue Leakage** — seller sengaja memperkecil berat paket yang dilaporkan di aplikasi, menyebabkan potensi kerugian pendapatan hingga miliaran rupiah.

Project ini memposisikan saya sebagai Data Analyst di tim **Business Intelligence SiCepat**, bertugas membersihkan data operasional, mengidentifikasi pola kedua masalah tersebut, dan memberikan rekomendasi berbasis data untuk SLA enforcement dan revenue recovery.

### Temuan Awal yang Melatarbelakangi Analisis

**10.199 transaksi** dengan `pickup_status = 'Success'` memiliki `pickup_time` yang terjadi **sebelum** `request_time` — kondisi yang mustahil secara operasional dan menjadi indikasi kuat Phantom Pickup.

---

## Dataset

| File                   | Baris   | Deskripsi                                                             |
| ---------------------- | ------- | --------------------------------------------------------------------- |
| `sicepat_sellers.csv`  | 15.000  | Dimensi Seller (seller_id, nama, kota, tanggal bergabung)             |
| `sicepat_services.csv` | 5       | Dimensi Layanan (jenis layanan pengiriman)                            |
| `sicepat_pickups.csv`  | 300.000 | Fakta Pickup & Berat (transaksi pickup periode Agustus–Desember 2023) |

> **Catatan teknis:** ketiga file CSV menggunakan separator titik koma (`;`), bukan koma — perlu di-load dengan `sep=';'`.

---

## Alur Pengerjaan

### 0. Import & Load Data

Load ketiga tabel sumber dengan separator yang sesuai.

### 1–2. Data Cleaning

Pembersihan data mentah: penanganan missing value, duplikat, standardisasi kategori, serta penetapan threshold wajar untuk nilai anomali (mis. `stated_weight_kg`).

### 3. Feature Engineering

Menambahkan kolom turunan untuk mendukung analisis, di antaranya:

- `item_category_clean` — kategori barang yang sudah distandardisasi
- `weight_gap_kg` — selisih berat yang dilaporkan vs berat aktual
- `is_oversize` — flag paket dengan gap berat di atas threshold
- `is_phantom_pickup` — flag transaksi dengan pola Phantom Pickup
- `sla_hours` & `is_sla_met` — durasi dan status pemenuhan SLA
- `seller_tenure_days` — lama seller bergabung
- `service_name` — nama layanan hasil merge dengan tabel services
- `revenue_loss_per_package` — estimasi kerugian pendapatan per paket
- `request_hour` — jam permintaan pickup, untuk analisis pola waktu

### 4. Exploratory Data Analysis

- Investigasi Phantom Pickup: repeat offender per seller, pola berdasarkan jam request
- Segmentasi revenue leakage berdasarkan layanan, kategori barang, dan kota seller
- Estimasi total kerugian dan penyusunan rekomendasi kebijakan

### 5. Export Clean Dataset

Menghasilkan `sicepat_clean.csv` — dataset final gabungan ketiga tabel sumber, siap dipakai untuk analisis atau reporting lanjutan (300.000 baris × 23 kolom).

### 6. Ringkasan & Refleksi

Rangkuman keputusan cleaning yang paling menantang, insight utama dari EDA, dan rekomendasi bisnis untuk tim Operations, Engineering, dan Finance SiCepat.

---

## Insight Utama

- **Phantom Pickup maupun revenue leakage tersebar merata** di semua segmen (layanan, kategori barang, kota, volume seller) — bukan terkonsentrasi pada layanan promo murah (HALU) atau wilayah tertentu seperti hipotesis awal.
- Pola yang merata ini mengindikasikan akar masalah bersifat **sistemik di level proses/aplikasi**, bukan masalah lokal yang bisa diselesaikan dengan kebijakan sempit pada satu segmen.
- Estimasi total kerugian revenue leakage selama periode Agustus–Desember 2023 mencapai **± Rp 2,1 miliar** (dengan asumsi tarif Rp 5.000/kg, perlu divalidasi ulang dengan tarif resmi SiCepat).

## Rekomendasi Bisnis

1. **Tim Engineering** — implementasikan validasi backend real-time yang menolak `pickup_time` lebih awal dari `request_time`, agar Phantom Pickup dicegah sejak di sumbernya.
2. **Tim Finance** — gunakan `revenue_loss_per_package` sebagai dasar kuantitatif untuk pengajuan chargeback ke platform e-commerce.
3. **Tim Operations** — terapkan monitoring `phantom_rate` per seller/kurir dan kebijakan verifikasi berat **bertingkat** (tiered enforcement): peringatan untuk gap kecil/sporadis, penahanan dana untuk gap besar & konsisten berulang — bukan blanket policy yang sama untuk semua seller.

---

## Tools & Libraries

- Python 3.13
- pandas
- numpy
- Jupyter Notebook

## Struktur Repository

```
├── Sicepat_Project_Completed.ipynb   # Notebook utama: cleaning, feature engineering, EDA
├── sicepat_sellers.csv               # Data sumber: dimensi seller
├── sicepat_services.csv              # Data sumber: dimensi layanan
├── sicepat_pickups.csv               # Data sumber: fakta pickup
├── sicepat_clean.csv                 # Output: dataset final hasil cleaning
└── README.md
```

## Cara Menjalankan

```bash
pip install pandas numpy jupyter
jupyter notebook Sicepat_Project_Completed.ipynb
```

---

_Project ini dibuat untuk tujuan pembelajaran (Purwadhika Digital Technology School). Nama seller, angka, dan skenario bisnis pada dataset bersifat simulasi/anonim._
