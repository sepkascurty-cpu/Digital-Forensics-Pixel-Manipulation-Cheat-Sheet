# 🛡️ Digital Forensics & Pixel Manipulation Cheat Sheet

Repositori ini berisi panduan komprehensif mengenai **Pixel Steganography**, teknik manipulasi pixel, alur investigasi forensik menggunakan CLI (Command Line Interface), dan metode perbaikan berkas gambar yang rusak (*corrupted headers*).

---

## 📌 Daftar Isi
1. [Konsep Dasar Steganografi Pixel](#1-konsep-dasar-steganografi-pixel)
2. [Alur Investigasi Forensik (Steganalisis)](#2-alur-investigasi-forensik-steganalisis)
3. [Panduan Alat Forensik CLI & Perintahnya](#3-panduan-alat-forensik-cli--perintahnya)
4. [Cara Kerja Manipulasi Pixel (LSB)](#4-cara-kerja-manipulasi-pixel-lsb)
5. [Perbaikan Berkas Gambar Rusak (Magic Bytes)](#5-perbaikan-berkas-gambar-rusak-magic-bytes)

---

## 1. Konsep Dasar Steganografi Pixel
Steganografi adalah seni menyembunyikan data rahasia di dalam media lain (seperti gambar) sehingga keberadaannya tidak disadari. 

Ketika dikombinasikan dengan **Kriptografi**, alurnya menjadi:
1. **Pesan rahasia** dienkripsi terlebih dahulu (misalnya menggunakan AES-256).
2. **Pesan terenkripsi** disisipkan ke dalam nilai pixel gambar (misalnya pada komponen Red, Green, Blue).

*Attacker* sering menggunakan metode ini untuk **Eksfiltrasi Data** (pencurian data sensitif tanpa memicu alarm firewall) atau penyusupan **Malware** lewat teknik *Fileless Execution* (skrip berjalan langsung di RAM).

---

## 2. Alur Investigasi Forensik (Steganalisis)
Saat tim forensik digital menemukan gambar mencurigakan (*stego-object*), mereka melalui 4 tahap standar industri
# 

1. **Metadata & Struktur:** Memeriksa anomali ukuran berkas dan teks aneh pada kolom komentar gambar.
2. **Visual Analysis:** Membedah lapisan warna untuk melihat *noise* yang tidak wajar.
3. **Statistical Analysis:** Menghitung distribusi statistik nilai pixel (Uji Chi-Square atau RS Steganalysis).
4. **Cracking:** Memecahkan kata sandi pelindung payload menggunakan serangan kamus (*dictionary attack*).

---

## 3. Panduan Alat Forensik CLI & Perintahnya

Berikut adalah perintah terminal yang wajib digunakan saat melakukan investigasi pada berkas gambar (misal: `target.png` atau `target.jpg`):

### A. Inspeksi Metadata
```bash
# Melihat semua informasi berkas dan tag EXIF tersembunyi
exiftool target.jpg
```

### B. Analisis Struktur & Ekstraksi File Tambahan
```bash
# Memindai apakah ada berkas zip/rar/gambar lain yang sengaja ditempel (Append)
binwalk target.png

# Mengekstrak otomatis berkas tersembunyi yang ditemukan oleh binwalk
binwalk -e target.png
```

### C. Deteksi LSB Otomatis (Format PNG/BMP)
```bash
# Memindai semua kombinasi bit untuk mencari pola teks/payload rahasia
zsteg target.png

# Mengekstrak payload dari kanal bit spesifik ke dalam file teks
zsteg -E "b1,rgb,lsb,xy" target.png > payload.txt
```

### D. Brute Force Password Steganografi
```bash
# Memecahkan sandi steghide dengan kecepatan super tinggi menggunakan wordlist
stegseek target.jpg /usr/share/wordlists/rockyou.txt
```

### E. Inspeksi Nilai Matriks Pixel Mentah
```bash
# Mengubah visual koordinat pixel gambar menjadi data teks numerik RGB
magick target.png txt:

# Mengintip nilai koordinat area tertentu saja (misal: kotak 5x5 piksel dari pojok kiri atas)
magick target.png -crop 5x5+0+0 txt:

# One-liner Python untuk mengintip tuple RGB dari 10 pixel pertama
python3 -c "from PIL import Image; img = Image.open('target.png'); print(list(img.getdata())[:10])"
```

---

## 4. Cara Kerja Manipulasi Pixel (LSB)
Gambar digital (RGB) menyimpan warna dalam format bit (0-255). Metode **Least Significant Bit (LSB)** mengubah bit paling belakang karena tidak memengaruhi warna secara signifikan di mata manusia.
### Analogi Perubahan Nilai:
Misalkan kita menyisipkan bit rahasia ke dalam satu piksel warna **Orange**:
* Nilai Asli: `RGB(240, 140, 10)`
* Nilai Setelah Dimanipulasi LSB: `RGB(240, 141, 10)`

Perubahan `1` angka desimal ini mengubah biner warna tetapi secara visual **mustahil terdeteksi oleh mata telanjang**.

---

## 5. Perbaikan Berkas Gambar Rusak (Magic Bytes)
Jika berkas gambar tidak dapat dibuka karena format salah/rusak, kemungkinan besar **Magic Bytes (File Header)** pada bagian teratas berkas telah dimanipulasi atau korup.

### 📋 Referensi Hex Header Standar:
* **PNG:** `89 50 4E 47 0D 0A 1A 0A` (Selalu mewakili teks `.PNG`)
* **JPG / JPEG:** `FF D8 FF`
* **GIF:** `47 49 46 38` (Selalu mewakili teks `GIF8`)

### 🛠️ Alur Perbaikan Lewat Terminal:

1. **Cek jenis berkas asli menurut sistem:**
   ```bash
   file foto_rusak.png
   ```
   *(Jika terbaca `data`, berarti header rusak).*

2. **Buka file menggunakan Hex Editor CLI:**
   ```bash
   hexedit foto_rusak.png
   ```

3. **Lakukan Koreksi:**
   * Arahkan kursor ke baris pertama (koordinat `00000000`).
   * Ganti byte yang rusak dengan kode pengenal yang sesuai (misal untuk PNG masukkan: `89 50 4E 47 0D 0A 1A 0A`).
   * Simpan dengan menekan `Ctrl + X`, lalu tekan `Y`.

4. **Verifikasi Ulang:**
   ```bash
   file foto_rusak.png
   ```
   *(Pastikan output terminal kini terbaca `PNG image data...` dan berkas siap dibuka).*

---
✏️ *Catatan: Gunakan repositori ini hanya untuk tujuan edukasi, analisis keamanan, dan riset forensik digital yang legal.*
