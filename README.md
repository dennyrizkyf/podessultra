# Peta Potensi Desa

Aplikasi web untuk memvisualisasikan data tabel (Podes dan data wilayah lainnya) di atas peta batas wilayah Sulawesi Tenggara. Data CSV/Excel dan GeoJSON dibaca **secara lokal di browser** dan tidak pernah diunggah ke server.

Demo: https://dennyrizkyf.github.io/podessultra/

## Fitur

- **Impor data**: tabel `.csv` / `.xlsx` / `.xls`, peta `.geojson` / `.json`, dan berkas label variabel (opsional).
- **Join ID**: pilih kolom ID di tabel dan properti ID di peta. Aplikasi menebak pasangan yang paling cocok secara otomatis.
- **Cek kecocokan ID** di layar unggah: persentase ID yang cocok, jumlah ID yang hanya ada di tabel atau hanya ada di peta, dan laporan Excel yang bisa diunduh (dua sheet: *Tabel tanpa peta* dan *Peta tanpa tabel*).
- **Tingkat administrasi**: desa/kelurahan, kecamatan, atau kabupaten/kota.
- **Filter wilayah** bertingkat (kabupaten, kecamatan, desa) dan pencarian nama/ID.
- **Metode visualisasi**
  - Warna area (choropleth), kontinu atau kategorik
  - Simbol proporsional (lingkaran)
  - Dua variabel: area + simbol, atau bivariat (3x3)
- **Klasifikasi kontinu**: Jenks (natural breaks), kuantil, interval sama, dengan jumlah kelas 3 sampai 9.
- **Legenda rentang kelas**: tiap kelas ditampilkan lengkap dengan batas angkanya, ditambah baris *Tanpa data* bila ada wilayah kosong.
- **Kustomisasi**: palet warna, gradasi kustom, warna kategori, transparansi, tebal dan warna garis batas, warna wilayah kosong.
- **Variabel baru** (kalkulator): operasi antar variabel, operasi dengan konstanta, dan logika bersyarat.
- **Tabel data** yang bisa diurutkan, dan **Crosstab** (tabulasi silang) dengan tampilan tabel atau grafik, ekspor ke Excel / PNG.
- **Ekspor peta** ke PNG dan SVG, dengan opsi background transparan dan sembunyikan legenda/teks.
- **Kartu keadaan awal**: sebelum variabel dipilih, peta menampilkan kartu dengan pintasan variabel (variabel terakhir dipakai, kolom ber-label, lalu kolom numerik yang bukan kolom pengenal). Kartu bisa ditutup.
- Tema terang dan gelap.

## Antarmuka

- **Sidebar accordion**: judul section (Wilayah, Variabel, Metode Visualisasi, Pengaturan Warna & Garis, Peta Dasar & Poligon, Ekspor Peta) bisa dilipat dengan satu klik, lengkap dengan ikon. *Peta Dasar & Poligon* dan *Ekspor Peta* terlipat secara default.
- **Kontrol segmen** untuk mode 1/2 variabel, tipe data kontinu/kategorik, dan metode visualisasi.
- **Tombol aksi menempel** di dasar sidebar (Tabel Data, Reset Pengaturan). Tombol *Tampilkan Peta* hanya muncul di layar sempit (≤1024 px), karena semua perubahan di desktop langsung diterapkan.
- **Header adaptif**: saat sidebar disembunyikan, ikon peta dan judul di header ikut disembunyikan.

## Cara pakai

1. Buka aplikasi, lalu unggah **Dataset Tabel** dan **Dataset Peta** (wajib). **Label** bersifat opsional.
2. Periksa panel **Kecocokan ID**. Atur **Kunci Penghubung (Join ID)** dan **Tingkat Administrasi** bila perlu. Unduh laporan bila ada ID yang tidak cocok.
3. Klik **Buka Ruang Kerja**.
4. Pilih variabel (atau pakai pintasan di kartu awal), lalu atur metode visualisasi dan palet di sidebar. Peta langsung diperbarui.

### Format data

| Berkas | Syarat |
|---|---|
| Tabel (`.csv` / `.xlsx`) | Baris pertama adalah header. Harus ada kolom ID wilayah. Sebaiknya ada kolom nama kabupaten, kecamatan, dan desa (nama kolom mengandung `KAB`, `KEC`, `DESA`) agar filter berfungsi. |
| Peta (`.geojson`) | `FeatureCollection` dengan properti ID yang nilainya sinkron dengan kolom ID di tabel. Angka nol di depan diabaikan saat pencocokan. |
| Label (`.csv` / `.xlsx`) | Kolom 1 adalah kode variabel, kolom 2 adalah nama/label variabel. |

Nilai kosong (`-`, `NA`, `N/A`, atau sel kosong) dianggap tidak ada data. Angka berformat Indonesia (`1.234,5`) dan notasi ilmiah (`1e-7`) dikenali otomatis. Sel angka pada file Excel dibaca sebagai angka asli, bukan teks berformat.

## Teknologi

Aplikasi berupa **satu berkas `index.html`** tanpa proses build. Pustaka dimuat lewat CDN:

| Pustaka | Fungsi |
|---|---|
| Leaflet 1.9.4 | Peta interaktif |
| chroma-js 2.4.2 | Skala warna dan klasifikasi |
| SheetJS (xlsx 0.18.5) | Baca dan tulis Excel |
| PapaParse 5.4.1 | Baca CSV |
| Chart.js | Grafik crosstab |
| dom-to-image 2.6.0 | Ekspor PNG |
| Nunito Sans (Google Fonts) | Tipografi |

**Basemap**: Esri (World Light Gray, World Dark Gray, World Imagery). Tidak memerlukan API key. Tile Carto sebelumnya diganti karena sekarang mewajibkan API key.

**Peramban**: Chrome, Edge, dan Safari modern, serta Firefox 121 ke atas (header adaptif memakai selektor CSS `:has()`).

## Struktur kode (di dalam `index.html`)

Urutan bagian: CSS (tema lewat CSS variables), HTML (layar unggah, header, sidebar, peta, modal), lalu satu blok `<script>`.

| Fungsi / variabel | Peran |
|---|---|
| `initMapAndUI()` | Membuat peta Leaflet dan basemap awal |
| `updateJoinSettingsUI()` | Menebak kolom ID di tabel dan peta |
| `computeJoinReport()`, `updateJoinReport()` | Menghitung dan menampilkan kecocokan ID, serta mengunduh laporan |
| `initApp()` | Join tabel ke GeoJSON, mengisi dropdown filter dan variabel |
| `applyViz(renderToMap, fitView)` | Inti render: filter fitur, klasifikasi, warna, legenda, layer. `fitView = true` hanya untuk perubahan filter wilayah atau tombol Tampilkan Peta, agar zoom tidak melompat saat slider digeser |
| `applyVizDebounced()` | Versi `applyViz` dengan debounce 120 ms untuk event `input` (slider, color picker) |
| `syncSegs()` | Menyinkronkan kontrol segmen dengan nilai `<select>` yang disembunyikan |
| `renderEmptyHint()` | Kartu keadaan awal dan pintasan variabel |
| `getRecentVars()`, `pushRecentVar()` | Riwayat variabel terakhir dipakai (disimpan di `localStorage`) |
| `isIdLikeColumn()`, `isUsableVar()` | Menyaring kolom pengenal dari saran pintasan |
| `getSafeNum(val)` | Pembersih angka (format Indonesia, notasi ilmiah, nilai kosong BPS) |
| `populateCustomSelect(...)` | Dropdown variabel dengan pencarian |
| `executeMapExportPNG` / `executeMapExportSVG` | Ekspor peta |
| `renderTable`, `renderXtabChart` | Tabel data dan crosstab |

Variabel global penting: `geojsonData`, `csvRows`, `allCols`, `labelMap`, `currentFilteredFeatures`, `leafletLayer`, `catColorMap`.

### Catatan untuk pengembangan

- `sel-var-count` dan `sel-var-type` adalah **sumber nilai** yang dibaca banyak fungsi. Keduanya hanya disembunyikan lewat CSS, jangan dihapus dari HTML. Kontrol segmen hanya mengubah nilainya lalu memicu event `change`.
- `btn-apply` disembunyikan di desktop lewat CSS, tetapi harus tetap ada di HTML karena handler-nya didaftarkan saat script dimuat.
- Ikon judul section dikaitkan lewat atribut `data-icon` (`wilayah`, `variabel`, `metode`, `warna`, `lanjutan`, `ekspor`), bukan lewat teks judul. Judul boleh diganti tanpa memengaruhi ikon.
- Event `input` (slider, color picker) memakai `applyVizDebounced()`. Event `change` boleh langsung memanggil `applyViz(true)`.
- Basemap dilepas dari peta saat ekspor transparan, bukan diubah opasitasnya, supaya basemap bertipe `layerGroup` (satelit + label) juga berfungsi.

## Menjalankan secara lokal

Cukup buka `index.html` di browser. Perlu koneksi internet untuk memuat pustaka CDN dan tile peta. Bisa juga memakai ekstensi **Live Server** di VS Code.

## Deploy dan pembaruan (GitHub Pages)

1. Ubah `index.html` di folder repo.
2. Commit dan push (VS Code: Source Control, isi pesan, **Commit**, **Sync Changes**).
3. Tunggu 1 sampai 2 menit, lalu buka situs dan tekan `Ctrl+Shift+R` untuk melewati cache.

Riwayat commit bisa dipakai untuk kembali ke versi sebelumnya bila ada masalah.

## Rencana pengembangan

Sudah selesai:

- [x] Laporan ID tidak cocok (daftar desa di tabel yang tidak ada di peta dan sebaliknya, bisa diunduh)
- [x] Legenda kontinu dengan batas rentang tiap kelas
- [x] Warna default wilayah kosong diganti abu-abu muda
- [x] Pembacaan angka Excel dan notasi ilmiah diperbaiki
- [x] Zoom peta tidak lagi melompat saat slider atau warna diubah (`fitView`, debounce)
- [x] Sidebar accordion, ikon section, kontrol segmen, dan kartu keadaan awal

Berikutnya, urut berdasarkan manfaat untuk kerja pengolahan data:

- [ ] Ringkasan statistik variabel aktif (min, maks, rata-rata, median, jumlah kosong) dan top/bottom 10 wilayah
- [ ] Unduh data beserta variabel hasil kalkulator ke Excel/CSV
- [ ] Simpan dan muat pengaturan visualisasi (JSON)
- [ ] Dropdown variabel: kolom pengenal (`IDKAB`, `KODE_PROV`, dan sejenisnya) dipindah ke bagian bawah daftar
- [ ] Ringkasan pengaturan di kanan judul section (misalnya "Warna area, 5 kelas")
- [ ] Palet ramah buta warna (misal viridis) dan palet divergen
- [ ] Crosstab mengikuti filter wilayah yang aktif
- [ ] Warna kategori otomatis untuk lebih dari 10 kategori (`chroma.scale(...).colors(n)`)
- [ ] Optimasi: cache centroid per fitur, join ID memakai `Map`, `setStyle` untuk perubahan gaya tanpa membangun ulang layer
- [ ] `escapeHtml` untuk nilai yang masuk ke `innerHTML` (tooltip, tabel, pratinjau)
- [ ] Kunci versi pustaka CDN (terutama Chart.js) atau simpan pustaka secara lokal agar bisa dipakai tanpa internet

## Masalah yang diketahui

- Variabel hasil kalkulator memakai `labelMap[nama] = nama`, sehingga label bisa tampil ganda (`NAMA - NAMA`).
- Pada penggabungan `{ ...match, ...f.properties }`, properti GeoJSON menimpa kolom CSV yang bernama sama.
- Untuk file CSV, angka dengan koma ribuan gaya Inggris (`1,234`) dapat terbaca sebagai desimal. Bersihkan di sumber data.
- Warna default wilayah kosong dan warna garis batas sama-sama `#cbd5e1`, sehingga batas antar wilayah kosong tidak terlihat. Ubah salah satunya lewat color picker bila perlu.
- Pintasan variabel menyaring kolom berdasarkan nama (`PROV`, `KAB`, `KEC`, `DESA`, dan sejenisnya). Variabel valid yang namanya kebetulan mengandung kata itu tidak muncul di saran, tetapi tetap bisa dipilih manual.

## Privasi

Data tabel dan peta tidak dikirim ke mana pun. Satu-satunya data yang disimpan di browser adalah **nama kolom** variabel yang terakhir dipilih (`localStorage`, untuk pintasan di kartu awal). Nilai data tidak disimpan. Permintaan jaringan hanya untuk memuat pustaka CDN, font, dan tile peta.

## Catatan

- Perubahan kode dilakukan satu poin per satu, dicek dulu di lokal sebelum di-push.
- Ikon tab (favicon) berupa SVG tertanam di `<head>`, tidak ada berkas ikon terpisah.

## Kredit

&copy; 2026 Tim Podes BPS Provinsi Sulawesi Tenggara
