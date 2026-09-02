# STANDAR OPERASIONAL PROSEDUR (SOP) ODOO 19 ERP
## Arsitektur Integrasi: Odoo ERP (Central Hub) ↔ Middleware ↔ Yango WMS (DC & Toko) ↔ Blibli

> **Dokumen Referensi**: *Meeting Requirements Brief — Integration Architecture Option 3*  
> **Peran Odoo 19**: Central Hub (Akuntansi & Keuangan, Procurement, Master Katalog/Harga SSOT, dan Valuasi Persediaan).  
> **Peran Yango WMS**: Eksekusi Fisik Gudang Pusat (DC) dan Toko Distribusi (DS) melalui **Yango Mobile**.  
> **Peran Blibli Seller Center**: Kanal Penjualan Marketplace Omnichannel.  
> **Peran Middleware**: Jembatan API real-time, event broker, retry mechanism, dan rekonsiliasi data otomatis.

---

## 1. PETA PERAN SISTEM & PEMBAGIAN KERJA

| Domain Operasional | Sistem Eksekusi (Front-End) | Sistem Pencatatan & Finansial (Back-End) | Peran Middleware |
|---|---|---|---|
| **Master Produk & Harga** | **Odoo 19** (Single Source of Truth) | **Odoo 19** | Kirim data produk & harga promo ke Blibli & Yango |
| **Registrasi Supplier & PO** | **Odoo 19** (Tim Procurement) | **Odoo 19** | Sinkronisasi nomor PO & ekspektasi barang ke Yango DC |
| **Penerimaan Barang DC (GR)** | **Yango Mobile** (Staf scan di DC) | **Odoo 19** (Validasi GR & update HPP) | Tangkap event scan GR Yango ➔ Validasi `WH/IN` di Odoo |
| **Transfer Stok DC ke Toko** | **Yango Mobile** (Staf scan transfer) | **Odoo 19** (Mutasi buku DC ke DS) | Tangkap transfer Yango ➔ Buat Internal Transfer Odoo |
| **Pesanan Masuk Blibli** | **Blibli Seller Center** | **Odoo 19** (Terbit Sales Order & Reservasi) | Blibli ➔ Yango (Picking) & Odoo (Sales Creation) |
| **Handover ke Kurir Blibli** | **Yango Mobile** (Kru toko serah kurir) | **Odoo 19** (Pindah ke Blibli In-Transit) | Tangkap status serah kurir ➔ Pindah ke In-Transit |
| **Kurir Sukses Antar** | **Kurir Blibli** (Status Delivered) | **Odoo 19** (Potong In-Transit, HPP & Invoice) | Tangkap event Delivered ➔ Potong stok final & Jurnal |
| **Retur, Batal, & Opname** | **Yango Mobile** (Toko fisik) | **Odoo 19** (Credit Note, Scrap, Selisih Buku) | Tangkap QC retur/opname ➔ Sesuaikan GL Odoo |

---

## 2. APA SAJA YANG HARUS ADA DI SOP ODOO 19?

Karena seluruh pemindaian barcode fisik di lantai gudang/toko ditangani oleh **Yango Mobile**, maka SOP pengoperasian Odoo 19 difokuskan pada **4 Peran Utama Pengguna Odoo**:

1. **SOP Tim Procurement & Purchasing**: Membuat data vendor, negosiasi termin, dan merilis PO resmi di Odoo.
2. **SOP Tim Master Data & Marketplace**: Mengelola katalog SKU, barcode, bobot kemasan, dan aturan harga promo Blibli (Pricelist) sebagai *Single Source of Truth*.
3. **SOP Tim ERP & WMS Operations**: Memantau automasi dokumen masuk dari Yango (GR DC, Stock Transfer DC-DS, In-Transit Handover) dan menangani antrean transaksi middleware.
4. **SOP Tim Finance & Accounting**: Mengelola penagihan piutang Blibli (settlement), penerbitan Credit Note retur, pencatatan beban kerugian selisih opname / barang rusak (Scrap), serta transaksi dapur (*Kitchen Transaction*).

---

## 3. PANDUAN 21 LANGKAH INTEGRASI TERPADU (END-TO-END)

```
[FASE A: MASTER DATA & PROCUREMENT DI ODOO]
  Step 1a & 1b: Catalog & Pricing Creation (Odoo SSOT ──► Blibli & Yango)
  Step 2 & 3  : Supplier Registration & PO Creation (Odoo ──► Yango DC)
                               │
[FASE B: LOGISTIK FISIK DI YANGO DC]
  Step 4      : Staf DC scan bongkar muat di Yango Mobile ──► Odoo GR Validated (WH/IN)
  Step 11     : Transfer DC ke Toko (DS) di Yango Mobile ──► Odoo Internal Transfer Validated
  [Opsi Cacat]: Step 13 & 14 PO Return di Yango ──► Odoo PO Return & Vendor Credit Note
                               │
[FASE C: ORDER FULFILLMENT BLIBLI]
  Step 5      : Sinkronisasi Stok Siap Jual (Odoo ──► Blibli)
  Step 6      : Pesanan Blibli Masuk ──► Odoo Auto Sales Order & Reservasi Stok Toko
  Step 9      : Kru toko serahkan paket ke kurir via Yango Mobile ──► Odoo pindah ke In-Transit
  Step 10     : Kurir update Delivered ──► Odoo potong In-Transit ke Customer & terbit Invoice/HPP
                               │
[FASE D: SIKLUS RETUR, PEMBATALAN & KONTROL TOKO]
  Step 15     : Customer Cancel sebelum kirim ──► Odoo Cancel SO & Unreserve Stok
  Step 16 & 17: Failed to Fulfill / Logistic Cancel ──► Odoo Cancel SO & Pelepasan Reservasi
  Step 18     : Kurir gagal antar (Delivery Fail) ──► Yango scan balik ──► Odoo In-Transit ke Toko
  Step 19 & 20: Retur Pelanggan (Customer Return) ──► Yango scan QC ──► Odoo Restock/Scrap & Credit Note
  Step 12 & 21: Stock Opname Toko & Kitchen Transaction ──► Odoo catat Beban Kerugian / HPP Dapur
```

---

### FASE A: MASTER DATA & PROCUREMENT (OPERASI DI ODOO)

#### Step 1a & 1b: Catalog Creation, SKU & Pricing/Promo Management (SSOT)
- **Pelaksana**: Tim Master Data & Marketplace Admin di Odoo.
- **Menu Odoo**: `Sales > Products > Products` & `Sales > Products > Pricelists`.
- **Langkah Operasional**:
  1. Buat produk baru bertipe wajib **Storable Product**.
  2. Masukkan **Internal Reference (SKU)**: e.g. `KOP-ARA-250` (Wajib sama persis dengan Merchant SKU Blibli & Yango).
  3. Masukkan Barcode EAN-13, berat kemasan (**Weight**), dan harga normal (**Sales Price**).
  4. Di menu Pricelist, tentukan harga promo Blibli (misal Rp 85.000) dan tanggal berlakunya.
- **Expected Result di Odoo**:
  - Master produk tersimpan. Middleware otomatis mengirim payload SKU dan harga ke Yango WMS dan Blibli Seller Center.

#### Step 2 & 3: Registrasi Pemasok & Rilis Purchase Order (PO)
- **Pelaksana**: Tim Procurement di Odoo.
- **Menu Odoo**: `Purchase > Orders > Requests for Quotation`.
- **Langkah Operasional**:
  1. Pilih Vendor terdaftar, tentukan **Deliver To**: `DC/Receipts`.
  2. Tambahkan baris produk: `KOP-ARA-250`, Quantity = `500` Pcs @ Rp 55.000.
  3. Klik tombol **Confirm Order**.
- **Expected Result di Odoo**:
  - Status berubah menjadi **Purchase Order**.
  - Smart button `Receipt (1)` muncul. Dokumen penerimaan berstatus *Ready* menanti konfirmasi fisik dari Yango DC.
  - Middleware otomatis mereplikasi data PO ini ke Yango WMS DC agar staf gudang tahu ada 500 pcs barang yang akan tiba.

---

### FASE B: INBOUND DC & DISTRIBUSI TOKO (EKSEKUSI DI YANGO MOBILE)

#### Step 4: Goods Receipt (GR Creation) di DC
- **Pelaksana**: Staf Gudang Pusat menggunakan **Yango Mobile**.
- **Aksi Lapangan**:
  - Truk vendor tiba di DC. Staf membuka aplikasi Yango Mobile, memilih nomor PO terkait, scan barcode produk, dan memverifikasi kuantitas fisik yang diterima (500 pcs).
- **Respon Otomatis di Odoo 19**:
  - Middleware menangkap status `GR_COMPLETED` dari Yango.
  - Dokumen `WH/IN/xxxxx` di Odoo otomatis tervalidasi menjadi **Done**.
  - Stok `DC/Stock` bertambah +500 unit. Jurnal persediaan bertambah di Buku Besar (General Ledger). Kolom *Received* di PO terisi 500.00.

#### Step 11: Transfer Stok DC ke Toko Distribusi (DC to DS / DS to DS)
- **Pelaksana**: Staf Gudang DC & Kru Toko menggunakan **Yango Mobile**.
- **Aksi Lapangan**:
  - Yango menginstruksikan pengiriman 100 pcs kopi dari DC ke Toko Jakarta Selatan (`DS01`).
  - Staf DC scan keluar (*pick & dispatch*) di Yango Mobile. Kru toko scan terima (*putaway*) di Yango Mobile saat armada tiba.
- **Respon Otomatis di Odoo 19**:
  - Middleware mengirim data mutasi transfer fisik Yango ke Odoo.
  - Odoo otomatis membuat dan memvalidasi dokumen **Internal Transfer**:
    - Source: `DC/Stock` (berkurang -100).
    - Destination: `DS01/Stock` (bertambah +100).

#### [Kasus Pengecualian] Step 13 & 14: Retur Pembelian ke Supplier (PO Return)
- **Aksi**: Staf DC mendeteksi 20 unit rusak saat scan di Yango Mobile ➔ status reject dilaporkan.
- **Respon di Odoo**:
  - Odoo menerbitkan dokumen `WH/RET/xxxxx` dari `DC/Stock` ke `Partner Locations/Vendors`.
  - Stok DC berkurang -20 unit.
  - Finance Odoo menerima trigger untuk membuat **Vendor Credit Note** memotong tagihan pemasok.

---

### FASE C: PEMENUHAN PESANAN BLIBLI & IN-TRANSIT (OMNICHANNEL)

#### Step 5: Sinkronisasi Saldo Stok Siap Jual (Stock Update)
- Odoo menghitung stok bebas jual (*Free to Use = On Hand - Reserved*). Middleware secara periodik mengirim angka stok terupdate ke Blibli Seller Center untuk mencegah *overselling*.

#### Step 6: Pesanan Blibli Masuk (Customer Order & Sales Creation)
- **Pemicu**: Pembeli membeli 2 pcs kopi di Blibli.
- **Respon di Yango & Odoo**:
  - Blibli mengirim order ke Yango untuk ditugaskan ke kru Toko Jakarta Selatan.
  - Middleware secara bersamaan membuat dokumen **Sales Order** di Odoo:
    - Customer: `Blibli Marketplace` (Alamat pembeli masuk ke field Shipping Notes/AWB).
    - Order Lines: `KOP-ARA-250` x 2 pcs @ Rp 85.000.
    - Warehouse: `DS Jakarta Selatan`.
- **Expected Result di Odoo**:
  - Status SO menjadi **Sales Order (Confirmed)**.
  - Stok di `DS01/Stock` otomatis berstatus **Reserved: 2 pcs**, sehingga *Free to Use* menjadi 98 pcs.

#### Step 9: Serah Terima ke Kurir Blibli (Order In Delivery ➔ Move to In-Transit)
- **Pelaksana**: Kru Toko menggunakan **Yango Mobile**.
- **Aksi Lapangan**:
  - Kru toko mengambil 2 unit kopi di rak, packing, dan menempelkan resi AWB Blibli.
  - Saat kurir Blibli datang mengambil paket, kru toko scan serah terima (*handover*) di Yango Mobile.
- **Respon Otomatis di Odoo 19**:
  - Yango mengirim event `ORDER_IN_DELIVERY`.
  - Odoo memvalidasi pengeluaran tahap 1:
    - Source: `DS01/Stock` (berkurang riil -2, rak fisik toko bersih).
    - Destination: `Virtual Locations/Blibli In-Transit` (bertambah +2).
  - *Manfaat*: Toko fisik bebas dari selisih audit, sementara unit tetap terpantau sedang berada di perjalanan.

#### Step 10: Kurir Sukses Kirim (Order Delivered ➔ GI Stock from In-Transit)
- **Pemicu**: Kurir Blibli menyelesaikan pengantaran ke rumah pembeli. Status di Blibli menjadi **Delivered**.
- **Respon Otomatis di Odoo 19**:
  - Middleware memicu transfer tahap 2 di Odoo:
    - Source: `Virtual Locations/Blibli In-Transit` (berkurang -2, saldo kembali 0).
    - Destination: `Partner Locations/Customers`.
  - Odoo otomatis mengaktifkan pembuatan Faktur Penjualan (**Customer Invoice**) ke akun piutang *Blibli Marketplace* dan membukukan Harga Pokok Penjualan (HPP/COGS) di General Ledger.

---

### FASE D: PEMBATALAN, RETUR & KONTROL STOK TOKO

#### Step 15: Pembatalan oleh Pembeli (Customer Order Cancel)
- Jika pembeli membatalkan pesanan sebelum kurir pick-up:
  - Yango membatalkan tugas picking di toko.
  - Odoo mengubah status SO menjadi **Cancelled**.
  - Reservasi 2 unit di toko langsung dilepas (**Unreserved**), saldo *Free to Use* kembali utuh menjadi 100 pcs.

#### Step 16 & 17: Gagal Pemenuhan Toko / Pembatalan Ekspedisi (Failed to Fulfill / Logistic Cancel)
- Jika barang di toko ternyata rusak atau kurir tidak kunjung datang:
  - Yango memicu status pembatalan ke Blibli.
  - Odoo membatalkan Sales Order dan mengembalikan alokasi reservasi stok.

#### Step 18: Penanganan Gagal Antar (GR Delivery Fail)
- Jika kurir gagal menemukan alamat rumah pembeli dan membawa paket kembali ke toko:
  - Kru toko scan paket retur gagal kirim di **Yango Mobile**.
  - Odoo otomatis memindahkan stok dari `Virtual Locations/Blibli In-Transit` kembali ke rak fisik `DS01/Stock`. Saldo In-Transit kembali 0 dan toko tidak rugi barang.

#### Step 19 & 20: Retur Penjualan oleh Pembeli (Customer Return & GR QC)
- Pembeli mengajukan komplain retur di Blibli dan mengirim barang kembali ke toko:
  1. Paket tiba di toko, kru toko scan penerimaan retur di **Yango Mobile** dan melakukan Quality Control (QC).
  2. **Jika Lolos QC (Barang Bagus)**: Yango memasukkan barang ke rak simpan. Odoo mencatat penambahan stok di `DS01/Stock`.
  3. **Jika Rusak / Cacat**: Yango menandai unit sebagai reject/damaged. Odoo otomatis memindahkan unit ke `Virtual Locations/Scrap`.
  4. Bagian Finance di Odoo menerbitkan **Credit Note** pada invoice Blibli untuk memotong piutang penjualan.

#### Step 12 & 21: Stock Opname Harian Toko & Transaksi Dapur (Stock Adjustment & Kitchen)
- **Stock Adjustment (Step 12)**:
  - Kru toko menghitung fisik harian via Yango Mobile. Jika ada selisih (misal -1 pcs hilang), Yango mengirim event adjustment.
  - Odoo otomatis membukukan selisih ke akun beban kerugian inventaris (**Inventory Loss**).
- **Kitchen Transaction (Step 21)**:
  - Untuk toko retail yang memiliki fasilitas kitchen/food preparation (memakai bahan baku seperti susu/sirup/kopi untuk disajikan):
  - Penggunaan bahan dicatat di Yango Mobile ➔ Odoo otomatis membukukan pemotongan stok bahan baku ke akun biaya operasional dapur (**Kitchen COGS / Operational Expense**).

---

## 4. TUGAS RUTIN HARIAN TIM ODOO (MONITORING & REKONSILIASI)

1. **Pukul 09:00 (Pagi)**:
   - Cek dashboard Middleware: Pastikan antrean pesan sinkronisasi PO dan GR Yango berstatus *Zero Errors*.
   - Cek PO masuk: Pastikan PO yang sudah divalidasi GR-nya di Yango telah berstatus *Received* di Odoo.
2. **Pukul 14:00 (Siang)**:
   - Monitor SO Blibli: Pastikan pesanan masuk dari Blibli terbit dengan benar di Odoo tanpa ada SKU yang tidak terpetakan (*Unmapped SKU*).
3. **Pukul 18:00 (Sore)**:
   - Rekonsiliasi Saldo `Virtual Locations/Blibli In-Transit`: Pastikan nomor resi yang sudah berstatus *Delivered* di Blibli telah tuntas memotong saldo In-Transit menjadi 0.
   - Validasi Credit Note dan jurnal Scrap hasil temuan harian toko Yango.
