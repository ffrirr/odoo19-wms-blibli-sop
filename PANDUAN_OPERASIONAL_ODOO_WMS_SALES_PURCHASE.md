# STANDAR OPERASIONAL PROSEDUR (SOP) TERPADU ODOO 19 ERP
## End-to-End Flow: Procurement, WMS Multi-Lokasi (DC & DS), Omnichannel Blibli, dan Kontrol Stok

> **Versi Target**: Odoo 19 (Enterprise / Community)  
> **Fitur Kunci Odoo 19**: Command Palette (`Ctrl+K` / `Cmd+K`), Modern Action Bar, In-App Knowledge App, Real-Time Stock Reservation & Dispatching, Two-Step Delivery with Virtual In-Transit.

---

## DAFTAR ISI KELANGKAPAN ALUR (END-TO-END)

```
[FASE 0: SETUP & MASTER DATA]
  0.1 Registrasi Vendor ──► 0.2 Master Produk & SKU ──► 0.3 Virtual In-Transit ──► 0.4 Pricelist Promo
                                                                                         │
[FASE 1: PENGADAAN & PENERIMAAN DC]                                                      ▼
  1.1 Purchase Order (PO) ──► 1.2 Goods Receipt (GR di DC) ──► [Opsi: 1.3 PO Return jika Cacat]
                                           │
[FASE 2: DISTRIBUSI DC KE TOKO (DS)]       ▼
  2.1 Internal Transfer (DC/Stock ──► DS/Stock)
                               │
[FASE 3: ORDER BLIBLI & HANDOVER KURIR]  ▼
  3.1 Auto-Create SO ──► 3.2 Picking/Packing ──► 3.3 Handover Kurir (DS/Stock ──► Blibli In-Transit)
                                                           │
[FASE 4: PENYELESAIAN PENGIRIMAN]                          ▼
  4.1 Kurir "Delivered" ──► 4.2 Auto-Cut Final Stok (Blibli In-Transit ──► Partner Locations/Customers)
                               │
[FASE 5: FLOW PENGECUALIAN / EXCEPTION]
  ├── 5.1 Pembatalan Pesanan (Sales Cancellation: Pre-Shipment & In-Transit)
  ├── 5.2 Retur Penjualan Pelanggan (Sales Return, QC Karantina & Refund)
  └── 5.3 Rekonsiliasi & Kontrol Stok Toko (Daily Stock Opname & Scrap Kerusakan)
```

---

## FASE 0: SETUP & MASTER DATA (PREREQUISITE)

### Langkah 0.1: Registrasi Pemasok Resmi (Vendor Registration)
- **Menu**: `Purchase > Orders > Vendors`
- **Langkah Operasi**:
  1. Klik **New**.
  2. Pilih **Company**.
  3. Isi **Name**: `PT Supplier Utama Indonesia`.
  4. Isi **Address**, **Phone**, **Email**, dan **Tax ID (NPWP)**.
  5. Tab **Sales & Purchase**: Set **Payment Terms** = `30 Days`.
  6. Tab **Invoicing**: Masukkan detail rekening bank pemasok.
  7. Klik **Save**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Kontak tersimpan dengan label "Vendor" aktif.
  - Form menampilkan smart button di kanan atas: **Purchases (0)**, **On-time Delivery Rate (100%)**, dan **Partner Ledger**.

---

### Langkah 0.2: Master Produk, SKU & Barcode (Single Source of Truth)
- **Menu**: `Sales > Products > Products` (atau `Inventory > Products > Products`)
- **Langkah Operasi**:
  1. Klik **New**.
  2. Isi tab **General Information**:
     - **Product Name**: `Kopi Arabika Specialty 250g`
     - **Product Type**: Wajib pilih **Storable Product** (Best Practice Odoo agar kuantitas stok terlacak).
     - **Internal Reference (SKU)**: `KOP-ARA-250` *(Wajib identik dengan Merchant SKU di Blibli)*.
     - **Barcode**: Masukkan 13 digit EAN barcode (misal: `8991234567890`).
     - **Sales Price**: Rp 95.000 (Harga jual normal SRP).
     - **Customer Taxes**: PPN 11%.
     - **Cost**: Rp 55.000 (HPP acuan).
  3. Tab **Inventory**:
     - **Tracking**: Pilih `By Unique Serial Number` / `By Lots` (jika ada kadaluarsa) atau `No Tracking`.
     - **Weight**: `0.28 kg` (termasuk berat kemasan pengiriman).
  4. Tab **Purchase**: Klik **Add a line**, pilih vendor `PT Supplier Utama Indonesia`, masukkan **Price** = Rp 55.000.
  5. Klik **Save**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Smart button di kanan atas menampilkan:
    - **On Hand**: `0.00 Units` (warna hitam/netral).
    - **Forecasted**: `0.00 Units`.
  - Produk siap digunakan di seluruh modul (Purchase, Inventory, Sales).

---

### Langkah 0.3: Konfigurasi Lokasi Virtual Kurir ("Blibli In-Transit")
- **Menu**: `Inventory > Configuration > Locations`
- **Langkah Operasi**:
  *(Pastikan Settings > Storage Locations & Multi-Step Routes sudah dicentang)*
  1. Klik **New**.
  2. **Location Name**: `Blibli In-Transit`
  3. **Parent Location**: `Virtual Locations`
  4. **Location Type**: Pilih **Transit Location** (atau **Internal Location** jika ingin nilai persediaan tetap masuk di neraca balance sheet hingga status Delivered).
  5. Klik **Save**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Tercipta lokasi virtual dengan hirarki path: `Virtual Locations/Blibli In-Transit`.
  - Lokasi ini siap dipilih sebagai tujuan pengeluaran barang saat kurir Blibli melakukan pick-up.

---

### Langkah 0.4: Penentuan Harga Jual & Promo Blibli (Pricelists)
- **Menu**: `Sales > Products > Pricelists`
- **Langkah Operasi**:
  1. Pilih pricelist khusus: `Pricelist Blibli Official Store` (atau buat baru jika belum ada).
  2. Pada tab **Price Rules**, klik **Add a line**:
     - **Apply On**: `Product` -> pilih `Kopi Arabika Specialty 250g`.
     - **Min. Quantity**: `1`.
     - **Computation**: Pilih **Fixed Price** -> Masukkan `Rp 85.000` (Harga promo diskon).
     - **Validity**: Tentukan **Start Date** dan **End Date** kampanye Blibli.
  3. Klik **Save & Close**, lalu klik **Save** pada form pricelist.
- **Expected Result (Hasil yang Diharapkan)**:
  - Di dalam tab pricelist muncul baris aktif: `KOP-ARA-250 | Fixed Price: 85,000 | Active Dates`.
  - API connector dapat menarik data harga promo ini untuk otomatis mengupdate harga listing di Blibli.

---

## FASE 1: PENGADAAN & PENERIMAAN GUDANG PUSAT (PROCUREMENT TO DC)

### Langkah 1.1: Pembuatan & Konfirmasi Purchase Order (PO)
- **Menu**: `Purchase > Orders > Requests for Quotation`
- **Langkah Operasi**:
  1. Klik **New**.
  2. **Vendor**: `PT Supplier Utama Indonesia`.
  3. **Deliver To**: Pastikan mengarah ke `DC/Receipts` (Gudang Pusat).
  4. Tab **Products** > **Add a product**:
     - Produk: `Kopi Arabika Specialty 250g`.
     - **Quantity**: `500` Pcs.
     - **Unit Price**: Rp 55.000.
  5. Klik tombol **Confirm Order**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Status dokumen berubah dari **RFQ** menjadi **Purchase Order**.
  - Muncul smart button baru di pojok kanan atas PO: **Receipt (1)**.
  - Pada master produk, smart button **Forecasted** berubah menjadi `500 Units` (meskipun On Hand masih `0 Units`).

---

### Langkah 1.2: Penerimaan Fisik di DC (Goods Receipt - GR / WH/IN)
- **Menu**: Klik smart button **Receipt** pada PO, atau navigasi ke `Inventory > Operations > Transfers` cari dokumen receipt terkait (`WH/IN/xxxxx`).
- **Langkah Operasi**:
  1. Buka dokumen receipt yang berstatus **Ready**.
  2. Tim gudang membongkar muat dan menghitung fisik barang yang tiba:
     - Jika 500 unit tiba utuh: Klik tombol **Set Quantities** (kolom **Done** otomatis terisi `500`).
     - Jika tiba sebagian (misal baru datang 300 unit): Ketik manual `300` pada kolom **Done**.
  3. Klik tombol **Validate**.
  4. *(Jika parsial)*: Muncul pop-up dialog **Create Backorder?**:
     - Pilih **Create Backorder** jika 200 unit sisanya akan dikirim menyusul.
- **Expected Result (Hasil yang Diharapkan)**:
  - Status dokumen transfer berubah menjadi **Done** (badge hijau).
  - Smart button master produk terupdate seketika:
    - **On Hand**: bertambah menjadi `500 Units` (atau `300 Units` jika parsial).
    - Lokasi stok tercatat jelas: `DC/Stock` = 500 Units.
  - Di PO, kolom **Received Quantity** terisi `500.00`.

---

### [Opsi Alur Cabang] Langkah 1.3: Retur Pembelian ke Vendor (PO Return)
*Gunakan langkah ini hanya jika saat pemeriksaan di DC ditemukan barang cacat/rusak.*
- **Menu**: Dari dokumen receipt `WH/IN/xxxxx` yang sudah **Done**, klik tombol **Return** di pojok kiri atas.
- **Langkah Operasi**:
  1. Pada pop-up **Reverse Transfer**:
     - Masukkan kuantitas barang reject (misal: `20` unit).
     - **Return Location**: `Partner Locations/Vendors`.
  2. Klik tombol **Return**.
  3. Sistem membuat dokumen transfer pengeluaran baru (`WH/RET/xxxxx`).
  4. Buka dokumen tersebut, isi kolom **Done** = `20`, lalu klik **Validate**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Status retur menjadi **Done**.
  - Stok `DC/Stock` langsung berkurang dari `500 Units` menjadi `480 Units`.
  - Di PO, kuantitas received berkurang dan tim finance dapat membuat **Credit Note** pada vendor bill.

---

## FASE 2: DISTRIBUSI STOK DC KE TOKO DISTRIBUSI (DC TO DS TRANSFER)

### Langkah 2.1: Transfer Stok DC ke Toko Distribusi (DS)
- **Menu**: `Inventory > Operations > Transfers`
- **Langkah Operasi**:
  1. Klik tombol **New**.
  2. **Operation Type**: Pilih `DC: Internal Transfers` (atau `DC to DS Transfers`).
  3. **Source Location**: `DC/Stock`.
  4. **Destination Location**: `DS01/Stock` (Toko Distribusi Jakarta Selatan).
  5. Tab **Products** > **Add a line**:
     - Produk: `Kopi Arabika Specialty 250g`.
     - **Demand**: `100` Pcs.
  6. Klik **Mark as Todo**:
     - Status berubah menjadi **Ready** dan stok 100 unit otomatis terkunci (*Reserved*) di `DC/Stock`.
  7. Barang fisik dinaikkan ke armada pengiriman toko:
     - Klik **Set Quantities** atau isi kolom **Done** = `100`.
     - Klik tombol **Validate**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Status transfer menjadi **Done**.
  - Buka menu `Inventory > Reporting > Stock`:
    - `DC/Stock`: Berkurang menjadi `380 Units`.
    - `DS01/Stock`: Bertambah menjadi `100 Units`.
  - Toko Distribusi `DS01` kini resmi memiliki 100 unit stok yang siap melayani pesanan Blibli.

---

## FASE 3: OMNICHANNEL SALES BLIBLI & HANDOVER KURIR

### Langkah 3.1: Pembuatan Sales Order Otomatis dari Blibli
- **Mekanisme**: Pelanggan membeli 2 pcs produk di Blibli. Webhook integrasi membuat Sales Order secara otomatis di Odoo.
- **Menu**: `Sales > Orders > Orders`
- **Tampilan Dokumen di Odoo**:
  1. Terbit dokumen baru bernama `SO-BLI-98421`.
  2. **Customer**: `Budi Santoso` (atau *Blibli Guest Customer*).
  3. **Customer Reference**: `BLI-ORDER-20260902-001`.
  4. **Warehouse**: Terpetakan otomatis ke `Toko DS01` (sesuai toko pemenuhan terdekat).
  5. **Order Lines**:
     - Produk: `Kopi Arabika Specialty 250g`.
     - **Quantity**: `2.00` Pcs.
     - **Unit Price**: `Rp 85.000` (mengikuti Pricelist Promo Blibli Fase 0.4).
- **Expected Result (Hasil yang Diharapkan)**:
  - Status dokumen otomatis **Sales Order** (Confirmed).
  - Muncul smart button **Delivery (1)** di kanan atas SO.
  - Pada stok `DS01/Stock`:
    - **On Hand**: `100 Units`.
    - **Reserved**: `2 Units`.
    - **Free to Use**: `98 Units`. *(Mencegah overselling secara real-time!)*

---

### Langkah 3.2: Picking & Packing di Toko Distribusi
- **Menu**: Buka smart button **Delivery** pada SO (dokumen `DS01/OUT/xxxxx`).
- **Langkah Operasi**:
  1. Status dokumen pengiriman adalah **Ready**.
  2. Kolom **Demand**: `2`, **Reserved**: `2`.
  3. Petugas toko mengambil 2 pcs kopi dari rak toko, melakukan packing, dan menempelkan cetakan shipping label / resi AWB Blibli.
  4. Isi kolom **Done** = `2` (atau scan barcode produk).
- **Expected Result (Hasil yang Diharapkan)**:
  - Kolom **Done** berubah menjadi warna hijau bernilai `2`. Paket berada di meja ekspedisi (*ready to ship*).

---

### Langkah 3.3: Handover ke Kurir Blibli (Memindahkan ke Lokasi Virtual In-Transit)
*Best Practice Odoo: Agar barang yang dibawa kurir tidak lagi ada di rak toko, tapi belum dianggap "Delivered" sampai pembeli terima.*
- **Langkah Operasi**:
  1. Kurir logistik Blibli tiba di toko untuk pick-up paket.
  2. Di dokumen pengiriman Odoo, ubah **Destination Location** menjadi `Virtual Locations/Blibli In-Transit` (atau sistem otomatis menjalankan alur *2-Step Delivery Route: Store -> In-Transit -> Customer*).
  3. Klik tombol **Validate**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Status dokumen pengiriman tahap 1 menjadi **Done**.
  - **Dampak Posisi Stok**:
    - `DS01/Stock`: Resmi berkurang dari `100` menjadi `98 Units`.
    - `Virtual Locations/Blibli In-Transit`: Bertambah `+2 Units`.
  - **Audit Stok**: Toko bebas dari risiko audit selisih fisik, sementara manajemen perusahaan tahu ada 2 unit yang sedang berada di jalan (*In-Transit*).

---

## FASE 4: PENYELESAIAN PENGIRIMAN (DELIVERY COMPLETION)

### Langkah 4.1: Notifikasi Kurir Sukses Kirim ("Delivered")
- **Pemicu**: Kurir Blibli menyelesaikan pengantaran ke rumah pembeli. Status pesanan di Seller Center Blibli berubah menjadi **Delivered / Selesai**.

### Langkah 4.2: Pemotongan Stok Final ke Customer Location
- **Langkah Operasi**:
  1. Webhook Blibli memicu validasi tahap akhir di Odoo (atau diproses via menu `Inventory > Operations > Transfers` filter lokasi sumber `Blibli In-Transit`).
  2. Dokumen transfer tahap 2 tervalidasi:
     - **Source Location**: `Virtual Locations/Blibli In-Transit`.
     - **Destination Location**: `Partner Locations/Customers`.
     - **Quantity**: `2.00` Pcs.
  3. Klik **Validate**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Saldo di `Virtual Locations/Blibli In-Transit` kembali berkurang `-2 Units` (menjadi `0`).
  - Barang resmi berada di `Partner Locations/Customers`.
  - Pada Sales Order, tombol **Create Invoice** aktif. Faktur penjualan terbit dan COGS/HPP persediaan terjual terekam otomatis di General Ledger.

---

## FASE 5: SKENARIO DEVIASI & EXCEPTION (KASUS KHUSUS)

---

### SKENARIO 5.1: PEMBATALAN PESANAN BLIBLI (SALES CANCELLATION)

Terdapat 2 kemungkinan waktu pembatalan:

#### Kasus A: Pembatalan SEBELUM Handover Kurir (Status: Ready / Waiting)
1. **Langkah Operasi**:
   - Buka Sales Order terkait di `Sales > Orders > Orders`.
   - Klik tombol **Cancel** pada Sales Order.
2. **Expected Result**:
   - Status SO berubah menjadi **Cancelled**.
   - Dokumen delivery order otomatis berstatus **Cancelled**.
   - Stok 2 unit yang sebelumnya terkunci langsung dilepas (**Unreserved**).
   - Nilai **Free to Use** di `DS01/Stock` langsung kembali naik dari `98` menjadi `100 Units`.

#### Kasus B: Pembatalan SETELAH Handover Kurir (Barang Gagal Kirim / Kurir Balik Kucing)
1. **Langkah Operasi**:
   - Kurir mengembalikan paket ke toko karena alamat pembeli tidak ditemukan (*Failed Delivery*).
   - Masuk ke menu `Inventory > Operations > Transfers` > klik **New**:
     - **Source Location**: `Virtual Locations/Blibli In-Transit`.
     - **Destination Location**: `DS01/Stock`.
     - **Product**: `Kopi Arabika Specialty 250g`, **Done**: `2`.
   - Klik **Validate**.
   - Buka SO asli lalu klik **Cancel**.
2. **Expected Result**:
   - Stok `Virtual Locations/Blibli In-Transit` berkurang `-2` (kembali 0).
   - Stok fisik di rak toko `DS01/Stock` bertambah kembali `+2` menjadi `100 Units`.
   - Tidak ada selisih stok finansial.

---

### SKENARIO 5.2: RETUR PENJUALAN OLEH PELANGGAN (SALES RETURN & QC)

Pembeli menerima barang namun mengajukan retur (misal kemasan penyok atau ingin tukar).

#### Langkah Operasi:
1. Buka dokumen Delivery Order awal di SO yang sudah **Done**.
2. Klik tombol **Return** di kiri atas.
3. Pada pop-up **Reverse Transfer**:
   - Masukkan kuantitas retur = `1`.
   - **Return Location**: Arahkan ke lokasi karantina pemeriksaan toko: `DS01/Input` (atau `DS01/Stock`).
   - Klik tombol **Return**.
4. Terbentuk dokumen penerimaan barang retur (`DS01/RET/xxxxx`).
5. **Pemeriksaan Kualitas (Quality Control)**:
   - **Kondisi A (Barang Masih Segel & Bagus)**:
     - Isi kolom **Done** = `1`, lalu klik **Validate**.
     - **Expected Result**: Stok `DS01/Stock` bertambah kembali dan siap dijual ke pembeli lain.
   - **Kondisi B (Barang Rusak / Bocor / Cacat)**:
     - Validasi penerimaan retur ke toko.
     - Lanjutkan segera dengan proses **Scrap** (Langkah 5.3.B).
6. **Refund Dana Pelanggan**:
   - Buka Invoice pada Sales Order, klik **Add Credit Note**.
   - Isi alasan: *"Retur Produk dari Blibli"*.
   - Klik **Confirm** untuk memotong saldo piutang dan mengembalikan dana via Blibli settlement.

---

### SKENARIO 5.3: PENYESUAIAN STOK TOKO (STOCK ADJUSTMENT & SCRAP)

#### A. Rekonsiliasi Stock Opname Harian Toko (Cycle Count / Selisih):
- **Menu**: `Inventory > Operations > Physical Inventory`
- **Langkah Operasi**:
  1. Filter lokasi: `DS01/Stock`.
  2. Cari produk `Kopi Arabika Specialty 250g`.
  3. Sistem menampilkan kolom **On Hand Quantity** = `98`.
  4. Staf menghitung fisik di rak ternyata hanya ada `97` (hilang 1 pcs).
  5. Masukkan angka `97` pada kolom **Counted Quantity**.
  6. Kolom **Difference** otomatis menunjukkan `-1.00`.
  7. Klik tombol **Apply**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Nilai On Hand di `DS01/Stock` resmi menjadi `97 Units`.
  - Mutasi selisih `-1` unit dibukukan otomatis ke akun biaya selisih inventaris (`Virtual Locations/Inventory adjustment` / Loss Account).

#### B. Scrap Produk Rusak / Kadaluarsa di Toko:
- **Menu**: `Inventory > Operations > Scrap`
- **Langkah Operasi**:
  1. Klik **New**.
  2. **Product**: `Kopi Arabika Specialty 250g`.
  3. **Quantity**: `1.00`.
  4. **Source Location**: `DS01/Stock`.
  5. **Scrap Location**: `Virtual Locations/Scrap`.
  6. Klik **Validate**.
- **Expected Result (Hasil yang Diharapkan)**:
  - Status dokumen Scrap menjadi **Done**.
  - Stok jual di `DS01/Stock` langsung berkurang `-1 Unit` dan dipindahkan ke lokasi sampah/scrap.
  - Barang rusak dijamin tidak akan pernah teralokasikan untuk pesanan online Blibli berikutnya.

---

## RINGKASAN CEK INDIKATOR KEBERHASILAN OPERASIONAL

| Langkah Operasional | Menu Odoo | Status Dokumen Sukses | Indikator Kunci Stok Berhasil |
|---|---|:---:|---|
| **0.1 Vendor** | Purchase > Vendors | Saved | Kontak memiliki tag Vendor |
| **0.2 Master SKU** | Sales > Products | Saved | Type = *Storable Product*, SKU & Barcode terisi |
| **0.4 Pricelist** | Sales > Pricelists | Active | Tanggal berlaku aktif & harga promo sesuai |
| **1.1 PO** | Purchase > RfQ | **Purchase Order** | Forecasted bertambah, Smart Button *Receipt (1)* muncul |
| **1.2 Goods Receipt** | Inventory > Receipts | **Done** | On Hand di `DC/Stock` bertambah |
| **1.3 PO Return** | Inventory > Transfers | **Done** | On Hand di `DC/Stock` berkurang, Credit Note vendor terbit |
| **2.1 Internal Transfer** | Inventory > Transfers | **Done** | `DC/Stock` berkurang, `DS01/Stock` bertambah |
| **3.1 Auto SO Blibli** | Sales > Orders | **Sale Order** | Stok di DS otomatis *Reserved*, Smart Button *Delivery (1)* |
| **3.3 Handover Kurir** | Inventory > Transfers | **Done** | `DS01/Stock` berkurang, `Blibli In-Transit` bertambah |
| **4.2 Delivered Customer** | Inventory > Transfers | **Done** | `Blibli In-Transit` menjadi 0, barang di *Customer Location* |
| **5.1 Cancel SO** | Sales > Orders | **Cancelled** | Reservasi stok lepas (*Unreserved*), Free to Use kembali |
| **5.2 Sales Return** | Inventory > Transfers | **Done** | Barang masuk kembali ke toko / QC karantina |
| **5.3 Stock Adjustment** | Inventory > Physical Inv | **Applied** | On Hand sinkron dengan fisik riil, selisih dibukukan |
| **5.3 Scrap** | Inventory > Scrap | **Done** | Barang rusak berpindah ke *Virtual Locations/Scrap* |
