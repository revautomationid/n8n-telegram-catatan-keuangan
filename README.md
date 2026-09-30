# Catatan Keuangan — Telegram Expense Tracker (n8n)

Workflow n8n yang mengubah bot Telegram menjadi asisten pencatat pengeluaran. Kirim teks atau foto nota, dan bot akan mencatat, meringkas, atau menghapus data pengeluaran di database Postgres.

## Fitur

- Catat pengeluaran dari pesan teks
- Catat pengeluaran dari foto nota (dibaca dengan model vision, lalu diproses agent)
- Cek total pengeluaran per periode (hari ini, minggu ini, bulan ini, dst.)
- Lihat rincian/daftar transaksi
- Hapus transaksi tertentu, dengan konfirmasi eksplisit dari pengguna
- Data terpisah per pengguna berdasarkan ID Telegram
- Balasan selalu ringkas, tanpa emoji, dan hanya seputar pencatatan keuangan

## Node dan Fungsi

| Node | Fungsi |
| --- | --- |
| Telegram Trigger | Menerima pesan (`message`) dan mengunduh file. Dibatasi ke user ID tertentu. |
| Message Type Router | Memisahkan pesan teks dan foto. Jenis pesan lain diabaikan. |
| Analyze image | Transkrip teks nota dengan `gpt-4o-mini`. |
| AI Agent | Menentukan tool yang dipakai dan menyusun balasan. Model: `gpt-5.6-sol`. |
| Simple Memory | Riwayat percakapan per pengguna (kunci sesi = ID Telegram). |
| Save / Check / Get Details / Delete Transaction | Tool Postgres dengan query berparameter. |
| Send Reply | Mengirim balasan ke chat Telegram pengirim. |

## Prasyarat

- Instance n8n dengan URL publik HTTPS (diperlukan webhook Telegram). Pakai Postgres node v2.5 atau lebih baru agar *Query Parameters* tersedia.
- **Bot Telegram** dan token dari [@BotFather](https://t.me/BotFather)
- **OpenAI API key**
- Database **PostgreSQL** dengan tabel:

```sql
CREATE TABLE catatan_keuangan (
    id                SERIAL PRIMARY KEY,
    tanggal           DATE NOT NULL,
    nama_toko         TEXT,
    rincian_pembelian TEXT,
    total_belanja     NUMERIC NOT NULL,
    no_hp             TEXT NOT NULL
);
```

> Kolom `no_hp` menyimpan **ID pengguna Telegram** (bukan nomor telepon). Nama kolom dipertahankan agar kompatibel dengan data lama.

## Cara Import

1. n8n → **Workflows** → **Import from File** → pilih `catatan-keuangan-telegram-shared.json`.
2. Buat dan hubungkan credential **Telegram**, **OpenAI**, dan **Postgres** pada node yang bersangkutan.
3. Pastikan tabel `catatan_keuangan` sudah dibuat.
4. Aktifkan workflow, lalu kirim pesan ke bot Anda untuk menguji.

## Contoh Penggunaan

| Pesan | Hasil |
| --- | --- |
| Foto nota | Nota dibaca, lalu disimpan ke database |
| `berapa total pengeluaran bulan ini?` | Ringkasan total dan jumlah transaksi |
| `apa saja pengeluaran minggu ini?` | Daftar transaksi beserta ID |
| `hapus transaksi Indomaret kemarin` | Bot menyebutkan transaksinya dan meminta konfirmasi sebelum menghapus |
