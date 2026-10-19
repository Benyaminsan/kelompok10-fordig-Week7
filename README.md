# Computer Forensics

**Mata Kuliah:** Forensik Digital

**Kelompok:** 10

1. Benjamin Khawarizmi Habibi — 5027231078
2. Fadillah Cantika Sari Hermawan - 5027231042
3. Seyal Zeruya Tandi Lalong - 5027231076
4. Gandhi Ert Julio - 5027231081
5. Kharisma Fahrun Nisa` - 5027231086

> **Petunjuk (hapus sebelum dikumpulkan):** Sesuaikan jumlah anggota. Semua teks berlatar kuning pada template asli adalah bagian yang harus diisi dari hasil analisis sendiri.

## 1. Persiapan Barang Bukti

> **Petunjuk (hapus sebelum dikumpulkan):** Jelaskan dari mana image diperoleh, bagaimana integritasnya dipastikan (hash), dan bagaimana image dilindungi dari perubahan (read-only) sebelum dianalisis. Sebutkan juga tools yang dipakai beserta versinya.

| Item | Nilai |
| --- | --- |
| Nama berkas | `linux_project_inode_lab.dd` |
| Ukuran | [isi dari `ls -l`] |
| SHA-256 hasil perhitungan | [isi dari `sha256sum`] |
| Hash acuan | [isi hash acuan dari penyedia image, jika ada] |
| Status verifikasi | [Cocok / Tidak cocok / Tidak ada acuan] |
| Tools dan versi | [mis. The Sleuth Kit x.y.z, dijalankan di Ubuntu WSL2] |

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
sha256sum linux_project_inode_lab.dd
chmod 444 linux_project_inode_lab.dd
ls -l linux_project_inode_lab.dd
```

## 2. Langkah Analisis dan Hasil

> **Petunjuk (hapus sebelum dikumpulkan):** Jalankan perintah di Ubuntu (WSL). Dari cmd atau Git Bash, awali dengan `wsl`. Untuk setiap langkah: tempel screenshot hasil Anda, lalu tulis analisis dengan kata-kata sendiri (apa yang terlihat, apa artinya, dan mengapa itu penting bagi investigasi).

### 2.1 Informasi filesystem

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
fsstat linux_project_inode_lab.dd | head -40
```

**Analisis:** [Catat tipe filesystem, ukuran blok, jumlah blok dan inode, inode terpakai, serta nilai Last Mounted at, Last Written, dan Last Checked. Jelaskan apa arti masing-masing bagi investigasi (mis. siapa/di mana filesystem terakhir dipakai, dan kapan).]

### 2.2 Daftar file termasuk yang terhapus

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
fls -r -p linux_project_inode_lab.dd
fls -rd -p linux_project_inode_lab.dd
```

**Analisis:** [Bedakan entri normal, entri bertanda `*` (terhapus), dan entri virtual Sleuth Kit. Catat nomor inode setiap file dan tandai mana yang tersembunyi (lazimnya diawali titik).]

### 2.3 Metadata inode

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
istat -z UTC linux_project_inode_lab.dd <inode>  # ulangi untuk setiap inode penting
```

**Analisis:** [Untuk tiap inode catat status alokasi, uid/gid, mode, jumlah link, ukuran, dan keempat timestamp (accessed, modified, changed, created). Bandingkan timestamp antar-inode dan antar-timestamp, lalu jelaskan apa yang bisa disimpulkan.]

### 2.4 Membaca isi file

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
icat linux_project_inode_lab.dd <inode>  # ulangi untuk setiap inode
```

**Analisis:** [Tulis isi setiap file yang berhasil dibaca. Jelaskan apakah file terhapus masih bisa dibaca lewat nomor inode, dan mengapa.]

### 2.5 Bukti penghapusan di direktori

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
istat linux_project_inode_lab.dd <inode_direktori>
blkcat linux_project_inode_lab.dd <nomor_blok> | hexdump -C | head -20
```

**Analisis:** [Cari nomor blok direktori dari hasil istat, lalu periksa isi mentahnya. Perhatikan nilai rec_len tiap entri dan apakah masih ada nama/inode sisa. Jelaskan mekanisme mengapa file tidak tampil pada listing normal.]

### 2.6 Pemeriksaan ekstensi file

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
icat linux_project_inode_lab.dd <inode> > hasil.bin
file hasil.bin
xxd -l 16 hasil.bin
```

**Analisis:** [Bandingkan ekstensi nama file dengan signature/magic bytes sebenarnya. Jelaskan apakah ada ketidaksesuaian dan apa tujuannya.]

### 2.7 Data sisa pada blok tidak teralokasi

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
blkls linux_project_inode_lab.dd | strings
srch_strings -t d linux_project_inode_lab.dd | grep -E "FLAG"
```

**Analisis:** [Catat string yang ditemukan di luar file yang terlihat. Nomor blok = offset desimal dibagi ukuran blok. Jelaskan dari mana sisa data itu kemungkinan berasal, dan apa yang tidak dapat ditentukan.]

### 2.8 Timeline

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
fls -r -m / linux_project_inode_lab.dd > body.txt
mactime -b body.txt -z UTC
```

**Analisis:** [Urutkan kejadian secara kronologis. Sebutkan zona waktu yang dipakai dan konversi ke WIB (UTC+7).]

### 2.9 Pemeriksaan tambahan (opsional)

> **[SISIPKAN SCREENSHOT TERMINAL]** Ganti kotak ini dengan hasil Anda.

```bash
e2fsck -fn linux_project_inode_lab.dd
debugfs -R "ls -d /src" linux_project_inode_lab.dd
```

**Analisis:** [Opsi `-n` membuka image read-only. Gunakan hanya bila diperlukan untuk menguji dugaan Anda; hapus subbagian ini bila tidak dijalankan.]

## 3. Ringkasan Temuan

### 3.1 File terhapus, tersembunyi, dan termodifikasi

> **Petunjuk (hapus sebelum dikumpulkan):** Satu baris per temuan. Kategori yang mungkin: Terhapus, Tersembunyi, Disamarkan (ekstensi palsu), Termodifikasi, Sisa data. Kolom Bukti berisi perintah/langkah yang menghasilkan temuan itu.

| No | File / Lokasi | Kategori | Inode | Bukti | Isi |
| --- | --- | --- | --- | --- | --- |
| 1 | [path] | [kategori] | [no.] | [perintah] | [isi/ringkasan] |
| 2 | [path] | [kategori] | [no.] | [perintah] | [isi/ringkasan] |
| 3 | [path] | [kategori] | [no.] | [perintah] | [isi/ringkasan] |
| 4 | [tambah baris sesuai temuan] |  |  |  |  |

### 3.2 Timeline kejadian

Seluruh waktu dalam [UTC / WIB] (WIB = UTC + 7 jam).

| Waktu | Kejadian |
| --- | --- |
| [tanggal jam UTC (jam WIB)] | [kejadian] |
| [tanggal jam UTC (jam WIB)] | [kejadian] |
| [tanggal jam UTC (jam WIB)] | [kejadian] |

## 4. Credential dan Pelaku

> **Petunjuk (hapus sebelum dikumpulkan):** Image ini mungkin tidak memuat log autentikasi atau history perintah. Bila begitu, tulis dengan jujur bahwa identitas user hanya disimpulkan dari bukti tidak langsung dan sebutkan buktinya.

- **A. Username:** [username, beserta bukti asalnya (mis. dari field mana pada fsstat atau string apa pada image)]
- **B. Hak akses:** [uid/gid pemilik file pada istat dan apa artinya; apa yang dapat/tidak dapat disimpulkan]
- **C. Waktu:** [rentang waktu kejadian dalam UTC dan WIB]
- **D. Data pendukung lain:** [mis. kesesuaian timestamp antar-artefak, mount point, pola operasi, dan temuan lain yang Anda anggap perlu]

## 5. Keterbatasan

> **Petunjuk (hapus sebelum dikumpulkan):** Tulis hal-hal yang tidak dapat dipastikan dari image (mis. tidak ada sumber pembanding untuk timestamp, ketidakpastian kecocokan nama file dengan isi inode, metode pasti penghapusan).

- A. [keterbatasan]
- B. [keterbatasan]
- C. [keterbatasan]

## 6. Kesimpulan

[Satu paragraf: apa yang ditemukan (terhapus/tersembunyi/termodifikasi), kapan, oleh siapa (beserta tingkat kepastiannya), dan bagaimana mekanismenya, berdasarkan bukti pada bagian 2 dan 3.]
