# DataQuest 2026: Same transition, different clocks

Analisis demografi 237 negara periode 1950-2100 berdasarkan data UN World Population Prospects 2024.

Semua negara menjalani transisi demografi yang sama: kematian turun, lalu kelahiran turun, lalu penduduk menua. Yang berbeda jauh adalah kecepatannya. Pertanyaan utamanya: **apakah satu indikator cukup untuk menggambarkan demografi suatu negara, atau ceritanya baru terlihat jika beberapa indikator dibaca bersama?**

## Data

| File | Isi |
|---|---|
| `WPP2024_Demographic_Indicators_Medium_csv.gz` | Data utama: 54 indikator, satu baris per lokasi per tahun, 1950-2101 |
| `WPP2024_Demographic_Indicators_OtherVariants_csv.gz` | 18 skenario proyeksi dan interval prediksi, 2024-2101 |
| `WPP2024_Demographic_Indicators_notes.csv` | Kamus indikator (kode, nama, satuan) |
| `WPP2024_Supplement_Births_and_LE60.csv` | Kelahiran menurut jenis kelamin dan harapan hidup di usia 60 |

Data sampai 2023 adalah estimasi. Mulai 2024 adalah proyeksi varian Medium.

Kolom `Location` mencampur negara dengan agregat seperti wilayah dan kelompok pendapatan. Analisis hanya memakai baris yang punya `ISO3_code` (237 negara), dan total penduduknya sudah dicek sama dengan baris World di setiap tahun.

## Analisis

| # | Bagian | Metode |
|---|---|---|
| 1 | Pembersihan data | Filter negara, pemetaan wilayah, cek total = World |
| 2 | Tren global dan divergensi antarnegara | Deret waktu per wilayah, pita persentil P10-P90 |
| 3 | Pertumbuhan alami vs migrasi | Identitas demografi, klasifikasi empat rezim |
| 4 | Indeks Transisi Demografi (DTI) | PCA pada 6 indikator, uji ketahanan bobot |
| 5 | Klaster lintasan 1950-2100 | K-Means, divalidasi dengan Ward dan bootstrap |
| 6 | Uji "satu indikator cukup?" | Random Forest, validasi silang 5-fold |
| 7 | Ketidakpastian proyeksi | Interval prediksi 95% dan skenario OtherVariants |
| 8 | Penuaan | Harapan hidup di usia 60 (file suplemen) |

## Temuan utama

- Penduduk dunia memuncak di sekitar 10,3 miliar pada 2084. Rentang 95% untuk 2100 adalah 9,0-11,4 miliar.
- Pangsa penduduk dunia yang tinggal di negara dengan kematian lebih banyak dari kelahiran naik dari 28% (2023) ke 52% (2100).
- Satu indeks dari PCA menjelaskan 75% variasi enam indikator transisi.
- Negara terbagi ke empat lintasan: late transition, migration-fuelled growth, fast transition, dan early transition (already aged). Indonesia masuk kelompok fast transition.
- Gabungan indikator memprediksi perubahan penduduk sampai 2100 lebih baik (R² 0,86) daripada indikator tunggal terbaik (0,77). Negara dengan fertilitas yang sama (1,3-1,7) bisa menyusut 62% atau tumbuh 98%.

## Struktur folder

```
DataQuest2026_WPP_Analysis.ipynb   notebook analisis lengkap (output sudah tersimpan)
ALUR_ANALISIS.md                   penjelasan alur, alasan metode, rancangan deck, persiapan tanya jawab
figures/                           grafik PNG dan peta interaktif HTML
outputs/                           tabel hasil (profil negara, ringkasan klaster, uji indikator)
```

## Cara menjalankan

```bash
pip install pandas numpy matplotlib seaborn plotly scikit-learn scipy
jupyter nbconvert --to notebook --execute DataQuest2026_WPP_Analysis.ipynb
```

Notebook harus berada di folder yang sama dengan file data. Eksekusinya sekitar satu menit.

## Batasan

- Proyeksi adalah skenario, bukan ramalan.
- Data per kelompok umur tidak tersedia, sehingga rasio ketergantungan diganti dengan median umur dan harapan hidup di usia 60/65.
- Keanggotaan negara per kelompok pendapatan tidak ada di data.

## Sumber

United Nations, Department of Economic and Social Affairs, Population Division (2024). *World Population Prospects 2024*, Online Edition. Lisensi CC BY 3.0 IGO.
