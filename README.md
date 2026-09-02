# SOP Odoo 19 ERP — WMS Multi-Warehouse, Procurement & Blibli Omnichannel

> **Versi Target**: Odoo 19 (Enterprise / Community)  
> **Kanal Penjualan**: Marketplace Blibli Omnichannel Integration  
> **Topologi Gudang**: Distribution Center (DC), Toko Distribusi (DS), dan Virtual In-Transit

---

## 📌 Ringkasan Proyek

Repositori ini berisikan **Standar Operasional Prosedur (SOP) Terpadu & Portal Web Interaktif** untuk pengoperasian Odoo 19 dalam lingkungan rantai pasok modern:
1. **Odoo 19 Purchase**: Pendaftaran Pemasok Resmi, Purchase Order (PO), dan Retur Pembelian (PO Return) jika barang reject.
2. **Odoo 19 Inventory (WMS)**: Penerimaan Barang DC (Goods Receipt), Distribusi Suplai DC ke Toko (DS), Lokasi Virtual (`Blibli In-Transit`), serta Rekonsiliasi Stok Opname Harian & Scrap.
3. **Odoo 19 Sales & Omnichannel**: Manajemen Katalog & Pricelist Promo (SSOT), Pembuatan Sales Order Otomatis (Webhook Blibli), Alur Pembatalan Pesanan, dan Penanganan Retur Penjualan (Sales Return & QC).

---

## 🌐 Portal Web Interaktif

Website panduan ini dapat dibuka langsung secara lokal melalui file [`index.html`](index.html) atau diakses via GitHub Pages:
- **Live Demo**: `https://ffrirr.github.io/odoo19-wms-blibli-sop/`
- **Fitur Web Portal**:
  - 🔍 *Live Search & Filter*: Pencarian cepat berdasarkan nama menu, dokumen, atau status.
  - 📊 *Interactive Diagram*: Diagram alur sistem menggunakan Mermaid.js.
  - 💡 *Odoo 19 Shortcuts*: Tips efisiensi Command Palette (`Ctrl+K`).
  - 🖨️ *Print & PDF Export*: Format cetak rapi ramah cetak.
  - ✅ *Expected Result*: Indikator keberhasilan status & mutasi stok di setiap langkah.

---

## 📁 Struktur Berkas

```text
.
├── index.html                                    # Portal Web Interaktif SOP Odoo 19
├── PANDUAN_OPERASIONAL_ODOO_WMS_SALES_PURCHASE.md # Dokumen Lengkap SOP Markdown
├── .nojekyll                                     # Konfigurasi GitHub Pages
└── README.md                                     # Dokumentasi Utama Repositori
```

---

## 🔄 Peta Alur Kronologis (Single End-to-End Flow)

```
[FASE 0: Setup Master Data & Promo]
       │
       ▼
[FASE 1: Pengadaan PO & Penerimaan Barang di DC] ──(Opsi Cacat)──► [1.3 Retur ke Vendor]
       │
       ▼
[FASE 2: Distribusi Stok DC ke Toko (DS)]
       │
       ▼
[FASE 3: Pesanan Blibli Masuk -> Picking & Packing -> Handover Kurir ke Blibli In-Transit]
       │
       ▼
[FASE 4: Kurir Blibli Sukses Kirim (Delivered) -> Pemotongan Final ke Customer Location]
       │
       ▼
[FASE 5: Alur Pengecualian: Batal Pesanan / Retur Pelanggan / Stock Opname Harian]
```

---

## 🚀 Cara Menjalankan Secara Lokal

Cukup buka berkas `index.html` di peramban web modern mana saja (Google Chrome, Firefox, Safari, Edge) tanpa perlu dependensi atau build tools tambahan.

```bash
# Menggunakan python built-in server (opsional)
python3 -m http.server 8080
```
Buka browser di: `http://localhost:8080`

---

## 📄 Lisensi & Kontribusi

Dibuat untuk standardisasi operasional implementasi Odoo 19 ERP Retail & Omnichannel.
Kontribusi dan perbaikan prosedur dipersilakan melalui mekanisme Pull Request.
