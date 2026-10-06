# Alur Analisis DataQuest 2026: UN World Population Prospects 2024

Judul kerja: **Same transition, different clocks.** Semua negara melewati transisi demografi yang sama, tetapi dengan kecepatan yang sangat berbeda.

Dokumen ini menjelaskan konteks data, urutan analisis, alasan memilih setiap metode, dan temuan utama beserta angkanya. Seluruh angka berasal dari notebook `DataQuest2026_WPP_Analysis.ipynb` dan bisa direproduksi dengan menjalankannya ulang.

---

## 1. Konteks

### 1.1 Kompetisinya

DataQuest 2026 bersifat terbuka. Panitia memberi data World Population Prospects (WPP) 2024 dari Divisi Kependudukan PBB dan delapan pertanyaan pemantik. Tidak ada target prediksi yang wajib dijawab. Penilaian bergantung pada kualitas pertanyaan, ketepatan metode, dan kejelasan cerita.

### 1.2 Isi folder

| File | Isi | Dipakai untuk |
|---|---|---|
| `WPP2024_Demographic_Indicators_Medium_csv.gz` | Data utama: 84.360 baris, satu baris per lokasi per tahun, 1950-2101, 54 indikator | Hampir semua analisis |
| `WPP2024_Demographic_Indicators_OtherVariants_csv.gz` | 18 skenario proyeksi alternatif dan interval prediksi, 2024-2101 | Analisis ketidakpastian (bagian 7) |
| `WPP2024_Demographic_Indicators_notes.csv` | Kamus indikator (kode, nama, satuan) | Rujukan |
| `WPP2024_Supplement_Births_and_LE60.csv` | Kelahiran menurut jenis kelamin dan harapan hidup di usia 60 | Analisis penuaan (bagian 8) |
| `*.docx` | Panduan awal dan deskripsi file suplemen | Rujukan |

### 1.3 Hal penting tentang data (sudah ditangani di notebook)

1. Kolom `Location` mencampur 237 negara dengan sekitar 280 agregat (wilayah, kelompok pendapatan, kelompok ad hoc seperti "African Union" atau "AUKUS"). Penjumlahan semua baris akan menghitung orang yang sama beberapa kali. Filter yang benar adalah `ISO3_code` tidak kosong. Notebook memverifikasi bahwa total 237 negara sama dengan baris World di setiap tahun (selisih maksimal 14 orang).
2. Data sampai 2023 adalah estimasi. Mulai 2024 adalah proyeksi varian Medium, yaitu satu skenario dan bukan kepastian. Di semua grafik deret waktu, periode proyeksi diberi latar abu-abu.
3. Tahun 2101 hanya berisi populasi 1 Januari, jadi analisis dibatasi sampai 2100.
4. Pemetaan negara ke wilayah memakai `ParentID` (negara ke subwilayah ke wilayah). Kode 918 (Northern America) tidak punya baris sendiri, sehingga diisi manual.
5. Data populasi per kelompok umur tidak tersedia, sehingga rasio ketergantungan (dependency ratio) tidak bisa dihitung. Sebagai gantinya dipakai median umur, LE65, dan LE60.
6. File tidak mencantumkan negara mana yang masuk tiap kelompok pendapatan. Perbandingan antar-kelompok pendapatan hanya bisa memakai baris agregatnya.

---

## 2. Pertanyaan utama

> Apakah ada satu indikator yang cukup untuk menggambarkan posisi dan arah demografi sebuah negara, atau ceritanya baru muncul bila beberapa indikator dibaca bersama?

Pertanyaan ini dipilih karena mencakup hampir semua pertanyaan pemantik panitia dalam satu alur cerita:

| Pertanyaan panitia | Dijawab di bagian |
|---|---|
| Q1 Divergensi struktur umur antarnegara | 2, 8 |
| Q2 Negara dengan rentang High-Low terlebar | 7 |
| Q3 Sensitivitas proyeksi 2100 terhadap asumsi fertilitas | 7 |
| Q4 Hubungan migrasi dengan pertumbuhan/penurunan penduduk | 3 |
| Q5 Menggabungkan beberapa indikator menjadi satu ukuran | 4 |
| Q6 Apakah negara membentuk kelompok berdasarkan pergerakan indikator dari waktu ke waktu | 5 |
| Q7 Fertilitas, mortalitas, dan migrasi bersama-sama membentuk lintasan penduduk | 3, 5 |
| Q8 Apakah satu indikator saja cukup | 6 |

---

## 3. Alur analisis langkah demi langkah

```
Data mentah
  -> [1] Bersihkan: pisahkan negara dari agregat, cek total = World
  -> [2] Gambaran global + divergensi antarnegara        (deskriptif)
  -> [3] Akuntansi pertumbuhan: alami vs migrasi         (identitas demografi)
  -> [4] Indeks Transisi Demografi (DTI)                 (PCA)
  -> [5] Klaster lintasan 1950-2100                      (K-Means + validasi Ward)
  -> [6] Uji "satu indikator cukup?"                     (Random Forest + CV)
  -> [7] Ketidakpastian proyeksi                         (OtherVariants)
  -> [8] Penuaan dan LE60                                (file suplemen)
  -> [9] Ringkasan angka + ekspor tabel
```

### Langkah 1. Memuat dan membersihkan data

- Tujuan: memastikan unit analisis adalah negara, bukan agregat.
- Cara: filter `ISO3_code`, buang tahun 2101, tambahkan kolom wilayah dan subwilayah, gabungkan file suplemen LE60.
- Validasi: total populasi negara sama dengan World di setiap tahun (assert).
- Nilai kosong yang tersisa wajar dan tidak perlu diimputasi: `DoublingTime` kosong saat penduduk tidak tumbuh, dan Holy See tidak ada di file suplemen.

### Langkah 2. Gambaran global dan divergensi (Q1)

- Metode: grafik deret waktu per wilayah memakai baris agregat wilayah, ditambah pita persentil P10-P50-P90 antarnegara per tahun. Lebar P90-P10 dipakai sebagai ukuran sebaran karena tahan terhadap pencilan.
- Alasan: sederhana, mudah dibaca juri, dan langsung menjawab pertanyaan apakah negara-negara makin mirip atau makin berbeda.
- Temuan:
  - Penduduk dunia naik dari 2,5 miliar (1950) ke 8,09 miliar (2023), memuncak sekitar 10,29 miliar pada 2084, lalu 10,18 miliar pada 2100.
  - TFR dunia turun dari 4,85 ke 2,25 dan menyentuh angka pengganti 2,1 sekitar 2050. Harapan hidup naik dari 46 ke 73 tahun.
  - Eropa sudah mengalami kematian lebih banyak dari kelahiran sejak 1993, dan penduduknya memuncak pada 2020. Asia dan Amerika Latin memuncak pada 2050-an. Afrika masih tumbuh sampai 2100, dan pangsanya naik dari 18% ke 38% penduduk dunia.
  - Sebaran fertilitas antarnegara paling lebar pada 1979 lalu menyempit. Sebaran median umur baru mencapai puncaknya pada 2033. Struktur umur menyusul fertilitas sekitar satu generasi kemudian. Sebaran LE65 terus melebar sampai 2100.
- Grafik: `01_global_trends.png`, `02_divergence_bands.png`

### Langkah 3. Akuntansi pertumbuhan: alami vs migrasi (Q4, Q7)

- Metode: identitas demografi `PopGrowthRate (%) ≈ (NatChangeRT + CNMR) / 10`. Setelah dicek di data (r = 0,999), setiap negara-tahun diklasifikasikan ke empat rezim: pertambahan/penurunan alami × imigrasi/emigrasi neto.
- Alasan: ini dekomposisi yang pasti benar secara matematis, bukan model statistik, sehingga sulit dibantah juri.
- Temuan:
  - Negara dengan kematian lebih banyak dari kelahiran: 10 pada 1975, 106 pada 2050, dan 164 dari 237 pada 2100.
  - Pangsa penduduk dunia yang tinggal di negara dengan penurunan alami naik dari 28% (2023) ke 52% (2100).
  - Pada 2023 ada 18 negara yang penduduknya turun secara alami tetapi tetap tumbuh karena imigrasi. Jerman yang terbesar.
  - Pada 2075 hampir semua negara berkumpul di dekat titik nol. Untuk kebanyakan negara, masa depan ditentukan oleh fertilitas dan penuaan. Pengecualiannya adalah negara tujuan imigrasi.
- Grafik: `03_growth_regimes.png`, `04_growth_components_region.png`

### Langkah 4. Indeks Transisi Demografi (Q5)

- Metode: PCA (Principal Component Analysis) pada enam indikator tahun 2023, lalu PC1 diskalakan ke 0-100.
- Indikator dan alasannya (satu per dimensi, menghindari duplikasi):

  | Indikator | Dimensi | Transformasi |
  |---|---|---|
  | TFR | tingkat fertilitas | z-score |
  | Q5 (kematian balita) | mortalitas anak | log, karena sangat miring ke kanan |
  | LEx | harapan hidup | z-score |
  | MedianAgePop | struktur umur | z-score |
  | MAC | waktu melahirkan | z-score |
  | Births1519 / Births | kelahiran usia remaja | z-score |

  Migrasi sengaja tidak dimasukkan karena bukan tahap transisi. Migrasi dibahas tersendiri di Langkah 3.
- Alasan memakai PCA: bobot ditentukan oleh data, bukan dipilih sendiri. Pendekatan ini umum dipakai untuk indeks komposit (misalnya dalam panduan indeks komposit OECD).
- Uji ketahanan:
  - PC1 menjelaskan 74,7% variasi, dan keenam indikator berbobot hampir setara (|0,30-0,44|).
  - Peringkat dengan bobot sama hampir identik (Spearman 0,999).
  - Membuang indikator mana pun tetap menghasilkan korelasi peringkat di atas 0,99.
- Bobot dan skala dikunci pada 2023, lalu diterapkan ke semua tahun. Hasilnya, setiap negara punya lintasan DTI 1950-2100 dalam satu skala yang sama.
- Temuan:
  - Skor terendah: Republik Afrika Tengah, Chad, Niger, Mali, Somalia.
  - Skor tertinggi: San Marino, Hong Kong, Korea Selatan, Jepang, Italia.
  - Negara median Eropa melewati DTI 50 pada 1983, Asia dan Amerika Latin pada 2013-2015, dan Afrika baru sekitar 2069. Negara median Afrika mencapai pada 2022 level yang sudah dimiliki Eropa pada 1950.
- Grafik: `05_dti_correlations.png`, `06_dti_map_2023.html` (peta interaktif), `07_dti_over_time.png`

### Langkah 5. Klaster lintasan 1950-2100 (Q6)

- Metode:
  1. Fitur: TFR, LEx, MedianAgePop, CNMR, PopGrowthRate pada setiap tahun kelipatan 10 (1950, 1960, ..., 2100), sehingga 5 × 16 = 80 nilai per negara.
  2. Standardisasi per indikator dengan satu rata-rata dan simpangan baku untuk seluruh negara-tahun. Dengan begitu level dan waktu perubahan tetap terbawa. Nilai dipotong di ±3 SD agar negara mikro dengan migrasi ekstrem tidak mendominasi.
  3. PCA mempertahankan 90% varians (8 komponen) untuk membuang redundansi antartahun.
  4. K-Means (k-means++, 50 kali inisialisasi).
  5. Jumlah klaster dipilih dengan skor silhouette dan metode siku (elbow).
  6. Validasi dengan Ward hierarchical clustering (Adjusted Rand Index) dan bootstrap 50 kali pada sampel 80%.
- Alasan: kebanyakan peserta akan membuat klaster dari satu tahun saja. Klaster berbasis lintasan langsung menjawab Q6, yaitu bagaimana indikator bergerak bersama dari waktu ke waktu.
- Pemilihan k: silhouette tertinggi di k = 2, tetapi itu hanya pembagian muda/tua. k = 4 adalah yang terbaik berikutnya (0,342 vs 0,334 untuk k = 3), dan memisahkan dua kelompok yang penting untuk cerita. ARI dengan Ward 0,53 (sedang: inti kelompok sama, negara di perbatasan berpindah). Stabilitas bootstrap rata-rata ARI 0,80.
- Empat klaster:

  | Klaster | Negara | Contoh | Ciri | Penduduk 2023 → 2100 |
  |---|---|---|---|---|
  | 1. Late transition, still growing | 65 | Pakistan, Nigeria, Ethiopia, Mesir, RD Kongo | TFR 6-7 sampai 1980-an, median umur 19 | +159%, pangsa dunia 22% → 45% |
  | 2. Migration-fuelled growth | 12 | Arab Saudi, Yordania, UEA, Israel | Imigrasi neto sampai 30 per 1.000, pertumbuhan pernah di atas 6%/tahun | +113% |
  | 3. Fast transition, now peaking | 80 | India, China, Indonesia, Brasil, Bangladesh | TFR turun dari 6 ke bawah 2 dalam 50 tahun | -13%, saat ini 59% penduduk dunia |
  | 4. Early transition, already aged | 80 | AS, Rusia, Jepang, Jerman, Italia | TFR sudah sekitar 3 pada 1950, median umur 42 | -11% |

  Klaster 3 dan 4 berakhir di fertilitas yang hampir sama (sekitar 1,6-1,7). Pembedanya adalah kecepatan. Klaster 3 menjalani dalam 50 tahun apa yang dijalani klaster 4 dalam satu abad, sehingga menua jauh lebih cepat.

  Indonesia masuk klaster 3.
- Grafik: `08_choose_k.png`, `09_cluster_trajectories.png`, `10_cluster_pca.png`, `11_cluster_map.html`

### Langkah 6. Apakah satu indikator cukup? (Q8)

- Metode: untuk 161 negara berpenduduk minimal 1 juta, nilai tahun 2023 dipakai untuk memprediksi dua hal:
  - Masa depan: `log(penduduk 2100 / penduduk 2023)`, dinilai dengan R².
  - Lintasan: label klaster dari Langkah 5, dinilai dengan akurasi.

  Setiap indikator diuji sendiri-sendiri, lalu semuanya digabung. Model: Random Forest, dengan validasi silang 5-fold.
- Alasan: Random Forest menangkap hubungan nonlinier, sehingga indikator tunggal mendapat kesempatan yang adil. Validasi silang mencegah overfitting. Negara di bawah 1 juta dikeluarkan karena laju migrasinya sangat bising.
- Hasil:

  | Indikator 2023 | R² perubahan penduduk 2100 | Akurasi klaster |
  |---|---|---|
  | Semua digabung | 0,86 | 0,88 |
  | PopGrowthRate | 0,77 | 0,62 |
  | TFR | 0,74 | 0,75 |
  | MedianAgePop | 0,69 | 0,86 |
  | LEx | 0,43 | 0,74 |
  | IMR | 0,35 | 0,80 |
  | MAC | -0,08 | 0,55 |
  | CNMR | -0,24 | 0,50 |

- Kesimpulan: indikator tunggal terbaik berganti tergantung pertanyaannya, dan tidak ada yang unggul di keduanya. PopGrowthRate sendiri sebenarnya sudah gabungan dari pertumbuhan alami dan migrasi. Contoh paling jelas: 44 negara punya TFR 1,3-1,7, tetapi proyeksi perubahan penduduknya sampai 2100 berkisar dari -62% (Jamaika, Albania, Bosnia) sampai +98% (Kuwait).
- Grafik: `12_single_vs_combined.png`, `13_same_tfr_different_future.png`

### Langkah 7. Ketidakpastian proyeksi (Q2, Q3)

- Metode: membandingkan varian Medium dengan interval prediksi probabilistik 95% PBB, varian High/Low (fertilitas ±0,5 anak), Constant fertility, dan Zero migration pada tahun 2100.
- Metrik per negara:
  - lebar relatif PI 95% = (Upper - Lower) / Medium
  - sebaran High-Low / Medium
  - Constant fertility vs Medium (%)
  - Zero migration vs Medium (%)
- Temuan:
  - Dunia 2100: PI 95% 9,0-11,4 miliar. High/Low (14,4 dan 7,0 miliar) jauh lebih lebar karena menggeser fertilitas semua negara sekaligus, sedangkan di model probabilistik galat antarnegara saling menutupi sebagian. Jika fertilitas tetap seperti sekarang, penduduk dunia menjadi 18,2 miliar.
  - Ketidakpastian per negara jauh lebih besar daripada untuk dunia. Yang terlebar adalah negara kecil yang didominasi migrasi (Hong Kong, Armenia, Georgia, Irlandia) dan negara berfertilitas sangat tinggi (Guinea Khatulistiwa, Palestina, Chad).
  - Sebaran High-Low berkorelasi negatif dengan TFR (-0,39), karena geseran 0,5 anak lebih besar artinya di negara yang fertilitasnya sudah rendah.
  - Per klaster: klaster migrasi paling tidak pasti (median lebar 2,9 kali nilai Medium), lalu klaster 1 (1,9), sedangkan klaster 3 dan 4 sekitar 1,3.
  - Sensitivitas fertilitas: dengan fertilitas konstan, penduduk Niger pada 2100 menjadi 357% lebih besar dari Medium. Sepuluh negara paling sensitif semuanya di Afrika sub-Sahara.
  - Ketergantungan pada migrasi: tanpa migrasi, Qatar dan UEA sekitar 74% lebih kecil pada 2100, Kanada 55%, Australia 49%, dan AS 37%.
  - China menyusut di semua skenario, termasuk High. Nigeria dan Pakistan tumbuh di semua skenario.
- Catatan: untuk 3 wilayah kecil, batas bawah PI dari PBB terpotong di nol, sehingga lebarnya membesar. Hal ini disebutkan di notebook.
- Grafik: `14_world_fan.png`, `15_uncertainty_cluster_and_top10.png`

### Langkah 8. Penuaan dan LE60 (Q1 lanjutan)

- Metode: lintasan LE60 per klaster (pita laki-laki sampai perempuan), ditambah scatter median umur vs LE60 tahun 2050.
- Temuan:
  - Selisih LE60 antara klaster tertua dan termuda melebar dari sekitar 7,5 tahun (2023) ke sekitar 10 tahun (2100).
  - Penurunan pada 2020-2021 adalah dampak COVID-19 dan terlihat di semua klaster.
  - Keunggulan perempuan di usia 60 naik dari 1,6 tahun (1950) ke 3,2 tahun (2023).
  - Pada 2050 Korea, Jepang, dan Italia berada di kuadran paling berat (penduduk tua dan berumur panjang setelah 60). China mencapai median umur serupa tetapi dengan LE60 sekitar 3 tahun lebih rendah.
- Grafik: `16_ageing_le60.png`

### Langkah 9. Ringkasan dan ekspor

Tiga file di `outputs/`:

- `country_profiles.csv`: satu baris per negara berisi klaster, DTI 2023, rezim pertumbuhan 2023 dan 2050, serta metrik ketidakpastian. Berguna untuk menjawab pertanyaan juri tentang negara tertentu, misalnya Indonesia.
- `cluster_summary.csv`
- `single_vs_combined.csv`

---

## 4. Rancangan alur deck (sekitar 12 slide)

| # | Judul slide (pesan utama) | Visual | Catatan bicara |
|---|---|---|---|
| 1 | Same transition, different clocks | Judul dan peta klaster | Pertanyaan utama dalam satu kalimat |
| 2 | Data dan satu jebakan yang kami hindari | Diagram sederhana: 237 negara vs ~280 agregat | Filter ISO3, cek total = World |
| 3 | Dunia memuncak di 10,3 miliar pada 2084 | `01_global_trends.png` | Wilayah menjalani langkah yang sama dengan selisih puluhan tahun |
| 4 | Fertilitas menyatu, struktur umur menyusul satu generasi kemudian | `02_divergence_bands.png` | Puncak sebaran: TFR 1979, median umur 2033 |
| 5 | Kematian akan melebihi kelahiran untuk separuh umat manusia | `03_growth_regimes.png` + angka 28% → 52% | Migrasi menahan 18 negara tetap tumbuh |
| 6 | Satu indeks, 75% cerita | `07_dti_over_time.png` dan potongan peta DTI | Jelaskan pilihan indikator dan uji ketahanan |
| 7 | Empat lintasan demografi | `09_cluster_trajectories.png` + peta klaster | Kecepatan transisi lebih penting daripada titik akhirnya |
| 8 | Fertilitas sama, masa depan berbeda | `13_same_tfr_different_future.png` | -62% sampai +98% pada TFR yang sama |
| 9 | Tidak ada satu indikator yang menang di semua pertanyaan | `12_single_vs_combined.png` | Bandingkan dua tugas prediksi |
| 10 | Seberapa yakin kita dengan 2100? | `14_world_fan.png` | PI 95% vs skenario deterministik |
| 11 | Siapa yang paling bergantung pada asumsi | `15_uncertainty_cluster_and_top10.png` | Sahel bergantung pada fertilitas, Teluk dan Kanada bergantung pada migrasi |
| 12 | Kesimpulan dan implikasi kebijakan | Tiga poin | Lihat bagian 5 |

Untuk peta: buka file `.html` di browser lalu ambil tangkapan layar, karena ekspor PNG plotly membutuhkan paket `kaleido` yang belum terpasang.

---

## 5. Pesan penutup untuk deck

1. Tidak ada satu indikator yang cukup. Fertilitas memberi arah, struktur umur memberi momentum, dan migrasi bisa membalik hasil akhirnya. Indikator terbaik berganti sesuai pertanyaan yang diajukan.
2. Pembeda utama antarnegara adalah kecepatan transisi. Negara yang bertransisi cepat (klaster 3, termasuk Indonesia) akan menua jauh lebih cepat daripada Eropa dulu, sehingga waktu untuk menyiapkan sistem pensiun dan perawatan lebih singkat.
3. Ketidakpastian terbesar tidak tersebar merata. Masa depan Afrika sub-Sahara bergantung pada seberapa cepat fertilitas turun. Masa depan negara Teluk, Kanada, dan Australia bergantung pada kebijakan migrasi.

---

## 6. Batasan (sebaiknya disebut di deck)

- Proyeksi adalah skenario, bukan ramalan. Bagian 7 menunjukkan seberapa lebar rentangnya.
- Tanpa data kelompok umur, rasio ketergantungan tidak bisa dihitung.
- Keanggotaan negara per kelompok pendapatan tidak ada di data.
- Klaster adalah ringkasan. Negara di perbatasan klaster wajar bila bisa masuk ke dua kelompok.
- Data negara mikro bising. Ini ditangani dengan clipping saat klasterisasi dan ambang 1 juta penduduk saat uji prediksi.

## 7. Persiapan tanya jawab juri

| Kemungkinan pertanyaan | Jawaban singkat |
|---|---|
| Kenapa k = 4, padahal silhouette tertinggi di k = 2? | k = 2 hanya memisahkan negara muda dan tua. k = 4 adalah silhouette terbaik berikutnya, dan memisahkan kelompok migrasi serta kelompok transisi cepat yang punya implikasi kebijakan berbeda. Hasilnya stabil di bootstrap (ARI 0,80). |
| Kenapa PCA, bukan bobot pakar? | Bobot dari data lebih bisa dipertanggungjawabkan, dan uji bobot sama memberi peringkat hampir identik (0,999). Jadi pilihan bobot tidak mengubah kesimpulan. |
| Kenapa Random Forest, bukan regresi linear? | Hubungan TFR dengan pertumbuhan tidak linear. Random Forest memberi kesempatan terbaik pada indikator tunggal, sehingga kesimpulan "tidak cukup" justru lebih kuat. |
| Bukankah ARI Ward 0,53 rendah? | Itu kesepakatan sedang. Inti kelompok sama, perbedaannya ada di negara perbatasan. Stabilitas bootstrap yang lebih relevan (0,80). |
| Bagaimana dengan Indonesia? | Klaster 3 (fast transition), DTI 2023 = 51 (tepat di tengah transisi). Jika fertilitas tetap seperti sekarang, penduduk 2100 menjadi 24% lebih besar dari Medium. Tanpa migrasi, penduduk 2100 menjadi 3% lebih besar karena Indonesia adalah negara emigrasi neto. Detail ada di `outputs/country_profiles.csv`. |

## 8. Cara menjalankan

1. Letakkan notebook di folder yang sama dengan file data.
2. Paket yang dibutuhkan: `pandas numpy matplotlib seaborn plotly scikit-learn scipy`.
3. Jalankan semua sel (sekitar 1 menit). Gambar tersimpan di `figures/`, tabel di `outputs/`.

Sumber data: United Nations, Department of Economic and Social Affairs, Population Division (2024). World Population Prospects 2024, Online Edition. CC BY 3.0 IGO.
