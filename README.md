# Peta Potensi Desa

Aplikasi web untuk memvisualisasikan data tabel (Podes dan data wilayah lainnya) di atas peta batas wilayah Sulawesi Tenggara. Seluruh proses berjalan **100% di browser**: data CSV/Excel dan GeoJSON dibaca secara lokal dan tidak pernah diunggah ke server.

Demo: https://dennyrizkyf.github.io/podessultra/

## Fitur

- **Impor data**: tabel `.csv` / `.xlsx` / `.xls`, peta `.geojson` / `.json`, dan berkas label variabel (opsional).
- **Join ID**: pilih kolom ID di tabel dan properti ID di peta. Aplikasi menebak pasangan yang paling cocok secara otomatis.
- **Tingkat administrasi**: desa/kelurahan, kecamatan, atau kabupaten/kota.
- **Filter wilayah** bertingkat (kabupaten, kecamatan, desa) dan pencarian nama/ID.
- **Metode visualisasi**
  - Warna area (choropleth), kontinu atau kategorik
  - Simbol proporsional (lingkaran)
  - Dua variabel: area + simbol, atau bivariat (3x3)
- **Klasifikasi kontinu**: Jenks (natural breaks), kuantil, interval sama, dengan jumlah kelas 3 sampai 9.
- **Kustomisasi**: palet warna, gradasi kustom, warna kategori, transparansi, tebal dan warna garis batas, warna wilayah kosong.
- **Variabel baru** (kalkulator): operasi antar variabel, operasi dengan konstanta, dan logika bersyarat.
- **Tabel data** yang bisa diurutkan, dan **Crosstab** (tabulasi silang) dengan tampilan tabel atau grafik, ekspor ke Excel / PNG.
- **Ekspor peta** ke PNG dan SVG, dengan opsi background transparan dan sembunyikan legenda/teks.
- Tema terang dan gelap.

## Cara pakai

1. Buka aplikasi, lalu unggah **Dataset Tabel** dan **Dataset Peta** (wajib). **Label** bersifat opsional.
2. Atur **Kunci Penghubung (Join ID)** dan **Tingkat Administrasi** pada panel yang muncul.
3. Klik **Buka Ruang Kerja**.
4. Pilih variabel, metode visualisasi, dan palet di sidebar, lalu klik **Tampilkan Peta**.

### Format data

| Berkas | Syarat |
|---|---|
| Tabel (`.csv` / `.xlsx`) | Baris pertama adalah header. Harus ada kolom ID wilayah. Sebaiknya ada kolom nama kabupaten, kecamatan, dan desa (nama kolom mengandung `KAB`, `KEC`, `DESA`) agar filter berfungsi. |
| Peta (`.geojson`) | `FeatureCollection` dengan properti ID yang nilainya sinkron dengan kolom ID di tabel. Angka nol di depan diabaikan saat pencocokan. |
| Label (`.csv` / `.xlsx`) | Kolom 1 adalah kode variabel, kolom 2 adalah nama/label variabel. |

Nilai kosong (`-`, `NA`, `N/A`, atau sel kosong) dianggap tidak ada data. Angka berformat Indonesia (`1.234,5`) dikenali otomatis.

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

## Struktur kode (di dalam `index.html`)

Urutan bagian: CSS (tema lewat CSS variables), HTML (layar unggah, header, sidebar, peta, modal), lalu satu blok `<script>`.

| Fungsi / variabel | Peran |
|---|---|
| `initMapAndUI()` | Membuat peta Leaflet dan basemap awal |
| `updateJoinSettingsUI()` | Menebak kolom ID di tabel dan peta |
| `initApp()` | Join tabel ke GeoJSON, mengisi dropdown filter dan variabel |
| `applyViz(renderToMap, fitView)` | Inti render: filter fitur, klasifikasi, warna, legenda, layer. `fitView = true` hanya untuk perubahan filter wilayah atau tombol Tampilkan Peta, agar zoom tidak melompat saat slider digeser |
| `applyVizDebounced()` | Versi `applyViz` dengan debounce 120 ms untuk event `input` (slider, color picker) |
| `getSafeNum(val)` | Pembersih angka (format Indonesia, notasi ilmiah, nilai kosong BPS) |
| `populateCustomSelect(...)` | Dropdown variabel dengan pencarian |
| `executeMapExportPNG` / `executeMapExportSVG` | Ekspor peta |
| `renderTable`, `renderXtabChart` | Tabel data dan crosstab |

Variabel global penting: `geojsonData`, `csvRows`, `allCols`, `labelMap`, `currentFilteredFeatures`, `leafletLayer`, `catColorMap`.

## Menjalankan secara lokal

Cukup buka `index.html` di browser. Perlu koneksi internet untuk memuat pustaka CDN dan tile peta. Bisa juga memakai ekstensi **Live Server** di VS Code.

## Deploy dan pembaruan (GitHub Pages)

1. Ubah `index.html` di folder repo.
2. Commit dan push (VS Code: Source Control, isi pesan, **Commit**, **Sync Changes**).
3. Tunggu 1 sampai 2 menit, lalu buka situs dan tekan `Ctrl+Shift+R` untuk melewati cache.

Riwayat commit bisa dipakai untuk kembali ke versi sebelumnya bila ada masalah.

## Rencana pengembangan

Urut berdasarkan manfaat untuk kerja pengolahan data:

- [ ] **Laporan ID tidak cocok**: daftar desa di tabel yang tidak ada di peta (dan sebaliknya), bisa diunduh
- [ ] Legenda kontinu dengan batas rentang tiap kelas
- [ ] Ringkasan statistik variabel aktif (min, maks, rata-rata, median, jumlah kosong) dan top/bottom 10 wilayah
- [ ] Unduh data beserta variabel hasil kalkulator ke Excel/CSV
- [ ] Simpan dan muat pengaturan visualisasi (JSON)
- [ ] Palet ramah buta warna (misal viridis) dan palet divergen
- [ ] Crosstab mengikuti filter wilayah yang aktif
- [ ] Warna kategori otomatis untuk lebih dari 10 kategori (`chroma.scale(...).colors(n)`)
- [ ] Warna default wilayah kosong diganti abu-abu muda
- [ ] Optimasi: cache centroid per fitur, join ID memakai `Map`, `setStyle` untuk perubahan gaya tanpa membangun ulang layer
- [ ] `escapeHtml` untuk nilai yang masuk ke `innerHTML` (tooltip, tabel, pratinjau)
- [ ] Kunci versi pustaka CDN (terutama Chart.js) atau simpan pustaka secara lokal agar bisa dipakai tanpa internet

## Masalah yang diketahui

- Nilai opsi basemap masih bernama `google-satellite` dan `google-hybrid` di kode, walau sumber tile-nya sudah Esri.
- Variabel hasil kalkulator memakai `labelMap[nama] = nama`, sehingga label bisa tampil ganda (`NAMA - NAMA`).
- Pada penggabungan `{ ...match, ...f.properties }`, properti GeoJSON menimpa kolom CSV yang bernama sama.
- Untuk file CSV, angka dengan koma ribuan gaya Inggris (`1,234`) dapat terbaca sebagai desimal. Bersihkan di sumber data.

## Catatan

- Perubahan kode dilakukan satu poin per satu, dicek dulu di lokal sebelum di-push.
- Ikon tab (favicon) berupa SVG tertanam di `<head>`, tidak ada berkas ikon terpisah.

## Kredit

&copy; 2026 Tim Podes BPS Provinsi Sulawesi Tenggara
