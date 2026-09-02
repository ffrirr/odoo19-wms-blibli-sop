# SOP Odoo 19 ERP — Integrasi Yango WMS (DC/DS) & Blibli Omnichannel

> **Arsitektur**: Option 3 Architecture (Requirements Brief)  
> **Central Hub ERP**: Odoo 19 (Procurement, Master Data SSOT, Finance & Inventory Valuation)  
> **Physical Execution**: Yango WMS & Yango Mobile (DC Inbound & Store Fulfillment)  
> **Marketplace**: Blibli Seller Center (Omnichannel Sales)  
> **Middleware**: Real-time Event Broker & Data Reconciliation Engine (21 Steps)

---

## 📌 Ringkasan Proyek

Repositori ini memuat **Standar Operasional Prosedur (SOP) Terpadu & Progressive Web App (PWA)** untuk pengoperasian Odoo 19 ERP yang diintegrasikan dengan Yango WMS dan Blibli Seller Center sesuai brief kebutuhan implementasi resmi.

Karena seluruh eksekusi fisik (scan barcode, picking, serah terima kurir, stock opname) ditangani langsung oleh **Yango Mobile**, peran Odoo 19 difokuskan sebagai **Headless Core Hub & Financial Engine**:
1. **Odoo Procurement**: Registrasi Supplier & Rilis PO resmi (Step 2 & 3).
2. **Odoo Master Catalog (SSOT)**: Master SKU, Barcode, Bobot, dan Harga Promo Blibli (Step 1a & 1b).
3. **Automated Inventory & In-Transit**:
   - Auto-validate Goods Receipt DC saat Yango Mobile scan terima (Step 4).
   - Auto-reconcile Stock Transfer DC ke Toko (Step 11).
   - Pemindahan stok toko ke `Virtual: Blibli In-Transit` saat kurir handover (Step 9).
   - Pemotongan final dari In-Transit ke Customer saat paket Delivered (Step 10).
4. **Alur Pengecualian & Keuangan**:
   - Pembatalan Pesanan (Step 15, 16, 17).
   - Retur Kurir Gagal Antar / GR Delivery Fail (Step 18).
   - Retur Pelanggan / GR Customer Return & Credit Note (Step 19 & 20).
   - Stock Adjustment Toko & Transaksi Dapur / Kitchen Usage (Step 12 & 21).

---

## 🌐 Portal Web Interaktif (PWA & Offline Ready)

- **Live Demo**: [https://ffrirr.github.io/odoo19-wms-blibli-sop/](https://ffrirr.github.io/odoo19-wms-blibli-sop/)
- **Fitur Aplikasi**:
  - 📱 *Mobile-First UX*: Sticky navigation chip, target sentuh min. 44x44px, zero horizontal scroll.
  - ⚡ *PWA Offline-First*: Dilengkapi Service Worker (`sw.js`) dan `manifest.json`.
  - 🔍 *Live Search*: Filter instan berdasarkan nama langkah, menu Odoo, atau nomor step.
  - 📊 *Mermaid 21-Step Map*: Visualisasi alur terpadu antar-sistem.
  - 📋 *Interactive Checklist*: Operator dapat mencentang skenario yang telah diuji (tersimpan di LocalStorage).
  - 🖨️ *Print & PDF Ready*: Format cetak bersih tanpa elemen navigasi website.

---

## 📁 Struktur Berkas

```text
.
├── index.html                                    # Portal Web Interaktif PWA SOP Odoo 19
├── PANDUAN_OPERASIONAL_ODOO_WMS_SALES_PURCHASE.md # Dokumen Lengkap SOP Markdown (21 Steps)
├── manifest.json                                 # Konfigurasi PWA Manifest
├── sw.js                                         # Service Worker Cache & Offline HUD
├── icons/                                        # Aset Ikon PWA (192x192 & 512x512)
├── .nojekyll                                     # Konfigurasi GitHub Pages
└── README.md                                     # Dokumentasi Utama Proyek
```
