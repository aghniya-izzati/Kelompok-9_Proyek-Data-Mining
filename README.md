# Analisis Kemampuan Numerasi Siswa SMP dan Faktor-Faktor yang Memengaruhinya Berdasarkan Asesmen Nasional 2025

Proyek akhir mata kuliah **Data Mining** Kelompok 9 Universitas Negeri Semarang (UNNES)

Repositori ini berisi notebook Google Colab (`Analisis_Numerasi_Siswa_SMP.ipynb`) yang menganalisis kemampuan numerasi siswa SMP/MTs sederajat menggunakan data **Rapor Pendidikan / Asesmen Nasional 2025** yang dipublikasikan oleh Kemendikdasmen. Analisis mencakup tiga teknik data mining utama: **regresi**, **klasifikasi**, dan **clustering (K-Means)**.

## Daftar Isi

1. [Ringkasan Proyek](#ringkasan-proyek)
2. [Sumber Data](#sumber-data)
3. [Struktur Notebook](#struktur-notebook)
4. [Penjelasan Detail Tiap Tahap](#penjelasan-detail-tiap-tahap)
5. [Hasil Utama](#hasil-utama)
6. [Cara Menjalankan](#cara-menjalankan)
7. [Library yang Digunakan](#library-yang-digunakan)
8. [Catatan dan Keterbatasan](#catatan-dan-keterbatasan)
9. [Struktur File](#struktur-file)

## Ringkasan Proyek

Pertanyaan utama yang ingin dijawab proyek ini: **faktor apa saja yang berhubungan dengan kemampuan numerasi siswa SMP**, dan **apakah siswa dapat dikelompokkan ke dalam profil-profil tertentu** berdasarkan kemampuan literasi/numerasi dan kondisi sosial-ekonominya.

Tiga pendekatan data mining yang dipakai:

- **Regresi** — memprediksi skor numerasi (NUM) sebagai variabel kontinu menggunakan Linear Regression dan Random Forest Regressor.
- **Klasifikasi** — mengubah skor numerasi menjadi 3 kategori (Rendah/Sedang/Tinggi) lalu mengklasifikasikannya dengan Decision Tree dan Random Forest Classifier.
- **Clustering** — mengelompokkan siswa ke dalam 3 segmen menggunakan K-Means tanpa label, lalu menginterpretasikan profil tiap cluster.

Dataset akhir yang dipakai dalam pemodelan berjumlah **522.303 siswa** dengan **8 fitur** dan 1 variabel target (NUM).

## Sumber Data

Data berasal dari file:
```
rapor-publik-asesmen-nasional-2025-peserta-didik-2025-smp-mts-sederajat.xlsx
```
(sheet: `rapor_publik`), yaitu data hasil Asesmen Nasional 2025 untuk jenjang SMP/MTs sederajat yang dipublikasikan Kemendikdasmen.

- Data mentah: **526.433 baris × 139 kolom**
- Diakses melalui Google Drive yang di-mount di Google Colab

Dari 139 kolom yang tersedia, dipilih 9 kolom yang relevan dengan tujuan penelitian (lihat bagian [Variabel yang Digunakan](#4-memilih-variabel-yang-digunakan)).

> **Catatan:** karena file sumber berukuran besar dan tersimpan di Google Drive pribadi, file `.xlsx` tidak disertakan dalam repositori ini. Untuk menjalankan ulang notebook, sesuaikan path `jalur_file` pada Cell 5 dengan lokasi file di Drive Anda sendiri.

## Struktur Notebook

Notebook terdiri dari **84 cell** (markdown sebagai heading bertahap + code) yang dibagi menjadi 4 bagian besar:

| Bagian | Tahap | Tujuan |
|---|---|---|
| **Persiapan** | 1–15 | Import library, load data, eksplorasi, pembersihan data, deteksi outlier, analisis korelasi |
| **Regresi** | 21–23 | Prediksi skor numerasi sebagai variabel kontinu |
| **Klasifikasi** | 24–29 | Prediksi kategori numerasi (Rendah/Sedang/Tinggi) |
| **Clustering** | 30–37 | Segmentasi siswa tanpa label menggunakan K-Means |

## Penjelasan Detail Tiap Tahap

### 1. Import Library

Mengimpor seluruh library yang dibutuhkan di awal notebook agar tidak berulang: `pandas`, `numpy` untuk manipulasi data; `matplotlib`, `seaborn` untuk visualisasi; modul `sklearn.preprocessing`, `model_selection`, dan `impute` untuk praproses; modul regresi (`LinearRegression`, `RandomForestRegressor`); modul klasifikasi (`DecisionTreeClassifier`, `RandomForestClassifier`, beserta metrik evaluasi); modul clustering (`KMeans`, `silhouette_score`, `PCA`); dan `scipy.stats` untuk kebutuhan statistik. Warning non-kritis disembunyikan dengan `warnings.filterwarnings('ignore')`.

### 2. Load Data

Google Drive di-mount ke environment Colab (`drive.mount('/content/drive')`), kemudian file Excel dibaca dengan `pandas.read_excel()` dari sheet `rapor_publik`. Shape data mentah yang dihasilkan: **(526.433, 139)**.

### 3. Eksplorasi Awal Data

Eksplorasi awal dilakukan dengan `df.info()` untuk melihat tipe data dan jumlah non-null, `df.describe()` untuk statistik deskriptif (mean, std, min, max, kuartil), dan iterasi seluruh 139 nama kolom untuk membantu memilih variabel yang relevan pada tahap berikutnya.

### 4. Memilih Variabel yang Digunakan

Dari 139 kolom, dipilih 9 kolom yang dianggap paling relevan secara konseptual untuk menjawab pertanyaan penelitian:

| Kolom | Keterangan |
|---|---|
| `NUM` | Skor kemampuan numerasi (variabel target) |
| `LIT` | Skor kemampuan literasi |
| `SES_siswa` | Status sosial-ekonomi siswa |
| `SES_sekolah` | Status sosial-ekonomi sekolah |
| `ketersediaan_internet` | Ketersediaan akses internet di sekolah (kategorikal: Ada/Tidak Ada) |
| `jumlah_komp_milik` | Jumlah komputer milik sekolah |
| `jumlah_perpus` | Jumlah perpustakaan di sekolah |
| `jumlah_pendidik` | Jumlah tenaga pendidik di sekolah |
| `rasio_pendidik_peserta_didik` | Rasio jumlah pendidik terhadap jumlah peserta didik |

Hasil subset: **526.433 baris × 9 kolom**.

### 5. Cek Tipe Data

Memeriksa tipe data tiap kolom dengan `data.dtypes`. Seluruh kolom numerik bertipe `float64`, kecuali `ketersediaan_internet` yang bertipe `object` (teks) karena berisi kategori "Ada" / "Tidak Ada".

### 6–8. Menangani Data Kategorikal

Kolom kategorikal (`ketersediaan_internet`) dipisahkan dari kolom numerik menggunakan `select_dtypes()`. Teks diseragamkan dulu dengan `.str.strip().str.title()` untuk menghindari duplikasi kategori akibat spasi atau perbedaan kapitalisasi, lalu di-encode menjadi angka menggunakan `astype('category').cat.codes`. Hasil encoding: `Ada → 0`, `Tidak Ada → 1`, dan nilai yang hilang (NaN) otomatis menjadi `-1`.

### 9–11. Menangani Missing Value

Sebelum imputasi, ditemukan missing value cukup signifikan pada beberapa kolom:

| Kolom | Jumlah Missing |
|---|---|
| `NUM` | 3.262 |
| `LIT` | 2.180 |
| `SES_siswa` | 3.260 |
| `jumlah_komp_milik` | 124.113 |
| `jumlah_perpus` | 124.113 |
| `jumlah_pendidik` | 124.113 |
| `rasio_pendidik_peserta_didik` | 124.113 |

Missing value diisi menggunakan `SimpleImputer(strategy='median')`, yaitu mengganti nilai yang hilang dengan nilai tengah (median) dari masing-masing kolom. Strategi median dipilih karena lebih tahan (robust) terhadap outlier dibanding mean. Setelah imputasi, seluruh kolom sudah bebas missing value (0 di semua kolom).

### 12–13. Cek Hasil Akhir dan Duplikat

Setelah praproses, struktur data diverifikasi ulang dengan `data_clean.info()`. Pengecekan duplikat menemukan **150 baris duplikat**, yang kemudian dihapus dengan `drop_duplicates()`, menyisakan **526.283 baris**.

### 14–15. Deteksi dan Penanganan Outlier

Outlier dideteksi menggunakan **metode IQR (Interquartile Range)**: batas bawah = Q1 − 1.5×IQR, batas atas = Q3 + 1.5×IQR. Persentase outlier per kolom:

| Kolom | Jumlah Outlier | Persentase |
|---|---|---|
| `NUM` | 3.980 | 0.8% |
| `LIT` | 7.226 | 1.4% |
| `SES_siswa` | 6.198 | 1.2% |
| `SES_sekolah` | 24.589 | 4.7% |
| `ketersediaan_internet` | 161.832 | 30.7% |
| `jumlah_komp_milik` | 44.366 | 8.4% |
| `jumlah_perpus` | 67.908 | 12.9% |
| `jumlah_pendidik` | 43.117 | 8.2% |
| `rasio_pendidik_peserta_didik` | 53.844 | 10.2% |

Outlier **hanya dihapus pada variabel target `NUM`** (bukan pada seluruh kolom), dengan pertimbangan agar fitur-fitur lain — termasuk variabel kategorikal yang persentase "outlier"-nya tinggi karena sifat biner — tidak ikut terbuang secara tidak proporsional. Setelah penghapusan outlier `NUM`, data tersisa **522.303 baris × 9 kolom**. Dataset inilah yang dipakai pada seluruh tahap pemodelan berikutnya.

### 16–17. Analisis Korelasi

Korelasi antar variabel numerik divisualisasikan dengan **heatmap** (`seaborn.heatmap`, hanya separuh segitiga bawah yang ditampilkan untuk menghindari duplikasi informasi). Korelasi tiap variabel terhadap `NUM`, diurutkan dari yang terkuat:

| Variabel | Korelasi terhadap NUM |
|---|---|
| `LIT` | 0.628 |
| `SES_sekolah` | 0.202 |
| `jumlah_pendidik` | 0.162 |
| `jumlah_komp_milik` | 0.148 |
| `SES_siswa` | 0.078 |
| `ketersediaan_internet` | 0.035 |
| `jumlah_perpus` | 0.029 |
| `rasio_pendidik_peserta_didik` | −0.109 |

Literasi (`LIT`) memiliki korelasi paling kuat terhadap numerasi — masuk akal karena kedua kemampuan ini secara konseptual saling terkait (kemampuan membaca-memahami soal berperan dalam menyelesaikan soal numerasi).

### 18–20. Memisahkan Fitur, Target, dan Normalisasi

Variabel target `y` = `NUM`; fitur `X` = 8 variabel sisanya (`LIT`, `SES_siswa`, `SES_sekolah`, `ketersediaan_internet`, `jumlah_komp_milik`, `jumlah_perpus`, `jumlah_pendidik`, `rasio_pendidik_peserta_didik`).

Data dibagi menjadi **80% training (417.842 baris)** dan **20% testing (104.461 baris)** menggunakan `train_test_split(test_size=0.2, random_state=42)`. Pembagian dilakukan **sebelum** normalisasi untuk mencegah *data leakage*. Normalisasi dilakukan dengan `StandardScaler`, di mana `scaler` di-*fit* hanya pada data training, kemudian dipakai untuk mentransformasi data testing (`transform`, bukan `fit_transform`).

## REGRESI

### 21. Model Regresi

Dua model dilatih dan dibandingkan: **Linear Regression** dan **Random Forest Regressor** (`n_estimators=100`). Evaluasi menggunakan RMSE (Root Mean Squared Error), MAE (Mean Absolute Error), dan R² Score.

| Model | RMSE | MAE | R² |
|---|---|---|---|
| Linear Regression | 10.6337 | 8.3082 | 0.3991 |
| Random Forest Regressor | 10.5441 | 8.2787 | 0.4092 |

Kedua model menghasilkan performa yang relatif setara, dengan Random Forest sedikit lebih baik. R² sekitar 0.40 berarti model hanya mampu menjelaskan ±40% variasi skor numerasi — menunjukkan bahwa masih banyak faktor lain (di luar 8 variabel yang dipakai) yang memengaruhi kemampuan numerasi siswa.

### 22. Koefisien Regresi Linear

Koefisien dari model Linear Regression (pada data yang telah dinormalisasi) menunjukkan arah dan besar pengaruh relatif tiap variabel:

| Variabel | Koefisien |
|---|---|
| `LIT` | 8.369 |
| `SES_sekolah` | 0.535 |
| `ketersediaan_internet` | 0.498 |
| `SES_siswa` | 0.431 |
| `jumlah_komp_milik` | 0.407 |
| `jumlah_pendidik` | 0.333 |
| `jumlah_perpus` | 0.041 |
| `rasio_pendidik_peserta_didik` | −0.246 |

`LIT` mendominasi secara signifikan dibanding variabel lain. Koefisien negatif pada `rasio_pendidik_peserta_didik` perlu dimaknai hati-hati — bukan berarti makin banyak pendidik per siswa justru menurunkan numerasi, melainkan kemungkinan mencerminkan karakteristik sekolah kecil/terpencil yang punya rasio pendidik tinggi namun fasilitas dan capaian belajarnya relatif rendah.

### 23. Visualisasi Aktual vs Prediksi

Scatter plot nilai aktual vs prediksi untuk kedua model, dengan garis diagonal referensi (prediksi sempurna). Titik yang menyebar di sekitar garis diagonal mengindikasikan model belum menangkap variasi data sepenuhnya — konsisten dengan R² yang masih moderat.

## KLASIFIKASI

### 24. Membuat Kategori Numerasi

Skor `NUM` (kontinu) diubah menjadi 3 kategori berdasarkan kuantil (persentil ke-33 dan ke-67):

- **Rendah**: NUM ≤ 56.66
- **Sedang**: 56.66 < NUM ≤ 69.00
- **Tinggi**: NUM > 69.00

Distribusi kelas relatif seimbang: Sedang (177.960), Rendah (174.538), Tinggi (169.805).

### 25. Split dan Normalisasi untuk Klasifikasi

Split data dilakukan ulang khusus untuk klasifikasi dengan `stratify=y_cat` agar proporsi tiap kelas tetap seimbang di training maupun testing set (417.842 training, 104.461 testing), diikuti normalisasi dengan `StandardScaler` baru.

### 26. Decision Tree

Model `DecisionTreeClassifier` dengan `max_depth=5` (dibatasi agar tidak overfitting) dan `min_samples_leaf=20`. Hasil:

- **Accuracy: 0.5781**
- F1-score per kelas: Rendah 0.64, Sedang 0.44, Tinggi 0.65

Pohon keputusan juga divisualisasikan (`plot_tree`, dibatasi `max_depth=3` untuk keterbacaan) beserta confusion matrix.

### 27. Random Forest Classifier

Model `RandomForestClassifier` (`n_estimators=100`, `max_depth=10`). Hasil:

- **Accuracy: 0.5817**
- F1-score per kelas: Rendah 0.64, Sedang 0.43, Tinggi 0.66

Kategori "Sedang" konsisten paling sulit diprediksi pada kedua model — masuk akal karena kelas tengah secara alami tumpang tindih dengan batas kategori Rendah dan Tinggi.

### 28. Feature Importance (Random Forest)

| Variabel | Importance |
|---|---|
| `LIT` | 0.809 |
| `SES_sekolah` | 0.055 |
| `SES_siswa` | 0.048 |
| `rasio_pendidik_peserta_didik` | 0.035 |
| `jumlah_pendidik` | 0.024 |
| `jumlah_komp_milik` | 0.020 |
| `ketersediaan_internet` | 0.005 |
| `jumlah_perpus` | 0.003 |

Sejalan dengan hasil korelasi dan regresi, `LIT` mendominasi secara sangat signifikan (>80% dari total importance) sebagai prediktor kategori numerasi.

### 29. Perbandingan Accuracy

Bar chart membandingkan accuracy Decision Tree (0.578) vs Random Forest (0.582) — selisih tipis, mengindikasikan kompleksitas tambahan Random Forest tidak memberi peningkatan performa yang besar untuk kasus ini.

## CLUSTERING

### 30. Persiapan Data Clustering

Fitur `X` (8 variabel, tanpa target `NUM`) dinormalisasi ulang dengan `StandardScaler` khusus untuk clustering, menghasilkan **522.303 baris × 8 fitur**.

### 31. Elbow Method + Silhouette Score

Karena ukuran data besar, proses pencarian k optimal dilakukan pada **sampel 30.000 data** (bukan seluruh dataset) demi efisiensi komputasi, menggunakan `KMeans` dengan `n_init=3` dan algoritma `elkan` (lebih cepat untuk data padat). Silhouette score dihitung untuk k=2 sampai k=8:

| k | Silhouette Score |
|---|---|
| 2 | 0.1514 |
| 3 | 0.1792 |
| 4 | 0.1630 |
| 5 | 0.1811 |
| 6 | 0.1807 |
| 7 | 0.1828 |
| 8 | 0.1840 |

Secara matematis murni, k=8 memberi silhouette score tertinggi. Namun **k=3 dipilih sebagai keputusan akhir**, dengan justifikasi eksplisit di notebook: selisih silhouette score antara k=3 (0.1792) dan k=8 (0.1840) tidak signifikan secara praktis, sementara kategori 3 kelompok (analog Rendah/Sedang/Tinggi) jauh lebih mudah diinterpretasikan untuk konteks penelitian pendidikan. Ini adalah contoh keputusan **berbasis domain knowledge**, bukan murni mengikuti angka statistik tertinggi.

### 32. Model K-Means Final

K-Means dijalankan dengan `n_clusters=3` pada **seluruh data** (522.303 baris, bukan sampel). Label cluster ditambahkan sebagai kolom baru `cluster` pada dataframe. Silhouette score pada model final (dihitung dengan sampling 5.000 titik untuk efisiensi): **0.1522**.

> Skor ini tergolong rendah secara umum (skala silhouette score: -1 hingga 1, di mana mendekati 0 berarti cluster saling tumpang tindih/tidak terpisah dengan jelas). Hal ini wajar mengingat data berasal dari populasi siswa riil yang karakteristiknya cenderung kontinu dan tidak membentuk kelompok-kelompok yang terpisah tegas secara alami.

### 33. Profil Tiap Cluster

Rata-rata nilai tiap fitur dihitung per cluster dan divisualisasikan sebagai heatmap profil:

| Variabel | Cluster 0 | Cluster 1 | Cluster 2 |
|---|---|---|---|
| LIT | 67.581 | 65.815 | 80.565 |
| SES_siswa | 48.115 | 46.619 | 49.708 |
| SES_sekolah | 71.569 | 67.008 | 76.638 |
| jumlah_komp_milik | 5.000 | 4.071 | 8.851 |
| jumlah_perpus | 1.000 | 0.999 | 1.168 |
| jumlah_pendidik | 16.000 | 13.362 | 32.993 |
| rasio_pendidik_peserta_didik | 0.070 | 0.100 | 0.063 |
| **NUM** | **60.282** | **60.200** | **68.893** |

### 34–36. Visualisasi Cluster

Tiga visualisasi disajikan untuk memahami pemisahan cluster: scatter plot Literasi vs Numerasi berwarna per cluster, visualisasi 2 dimensi hasil **PCA** (dua komponen utama menjelaskan **39.8%** total variansi data — relatif moderat, artinya proyeksi 2D ini menyederhanakan banyak informasi), dan kurva distribusi kepadatan (KDE) skor NUM per cluster untuk melihat tumpang tindih antar kelompok.

### 37. Interpretasi Hasil Cluster

Interpretasi dibuat otomatis berdasarkan data, dengan mengurutkan cluster dari rata-rata NUM tertinggi ke terendah:

- **Cluster 2 (142.814 siswa)** — performa **Tinggi**. Rata-rata NUM 68.89, LIT 80.57. Didukung kondisi sekolah relatif baik (jumlah komputer dan pendidik terbanyak, SES sekolah tertinggi).
- **Cluster 0 (122.670 siswa)** — performa **Sedang**. Rata-rata NUM 60.28, LIT 67.58. Kondisi sekolah cukup baik namun capaian siswa tidak seoptimal Cluster 2.
- **Cluster 1 (256.819 siswa)** — performa **Rendah**. Rata-rata NUM 60.20, LIT 65.81. SES sekolah dan fasilitas teknologi paling terbatas di antara ketiga cluster.

> **Catatan kehati-hatian akademik:** rata-rata NUM Cluster 0 (60.28) dan Cluster 1 (60.20) sangat berdekatan — perbedaannya hanya 0.08 poin. Pelabelan "Sedang" vs "Rendah" pada kedua cluster ini lebih didasarkan pada perbedaan profil fasilitas/SES sekolah (Cluster 1 jauh lebih rendah di hampir semua indikator fasilitas) ketimbang perbedaan skor numerasi itu sendiri, yang nyaris identik. Pembaca laporan sebaiknya tidak menafsirkan kedua cluster ini sebagai dua tingkat capaian numerasi yang jelas berbeda.

Cluster 1 juga merupakan kelompok terbesar (256.819 siswa, hampir 50% dari seluruh data), yang relevan untuk diskusi soal kesenjangan fasilitas pendidikan secara lebih luas.

## Hasil Utama

Ringkasan temuan kunci dari ketiga pendekatan:

1. **Literasi (`LIT`) adalah prediktor paling kuat dan konsisten** terhadap kemampuan numerasi di ketiga pendekatan (korelasi tertinggi 0.628, koefisien regresi terbesar, feature importance >80%).
2. **Faktor sosial-ekonomi dan fasilitas sekolah** (SES sekolah, jumlah komputer, jumlah pendidik) berkontribusi positif namun jauh lebih kecil dibanding literasi.
3. **Model prediktif (regresi maupun klasifikasi) hanya mencapai performa moderat** (R² ≈ 0.40, akurasi klasifikasi ≈ 58%), mengindikasikan kemampuan numerasi dipengaruhi banyak faktor lain yang tidak tercakup dalam 9 variabel yang dipilih.
4. **Clustering mengungkap 3 profil siswa** yang berbeda terutama dari sisi fasilitas/SES sekolah, dengan cluster performa rendah mencakup hampir separuh populasi siswa dalam dataset.

## Cara Menjalankan

1. Buka notebook `Analisis_Numerasi_Siswa_SMP.ipynb` di [Google Colab](https://colab.research.google.com/).
2. Pastikan file data `rapor-publik-asesmen-nasional-2025-peserta-didik-2025-smp-mts-sederajat.xlsx` tersedia di Google Drive Anda.
3. Sesuaikan variabel `jalur_file` pada bagian **2. LOAD DATA** dengan path file di Drive Anda sendiri.
4. Jalankan seluruh cell secara berurutan dari atas ke bawah (**Runtime → Run all**). Tahap pencarian k optimal (Elbow Method) memerlukan waktu komputasi paling lama karena melibatkan beberapa iterasi K-Means.

## Library yang Digunakan

| Library | Fungsi |
|---|---|
| `pandas`, `numpy` | Manipulasi dan operasi data |
| `matplotlib`, `seaborn` | Visualisasi data dan hasil model |
| `scikit-learn` | Praproses (`StandardScaler`, `SimpleImputer`), regresi, klasifikasi, clustering, dan metrik evaluasi |
| `scipy` | Fungsi statistik pendukung |

## Catatan dan Keterbatasan

- Dataset bersumber dari Asesmen Nasional 2025 untuk jenjang SMP/MTs sederajat; hasil dan interpretasi tidak digeneralisasi ke jenjang pendidikan lain.
- Pembersihan outlier hanya diterapkan pada variabel target (`NUM`), sehingga outlier pada fitur lain (terutama fasilitas sekolah) masih ada di data pemodelan — sebuah keputusan metodologis yang sengaja diambil dan perlu disebutkan secara eksplisit dalam laporan/artikel agar transparan.
- Performa model (R² ≈ 0.40 untuk regresi, akurasi ≈ 58% untuk klasifikasi) tergolong moderat — bukan model dengan akurasi sangat tinggi, dan ini harus dilaporkan secara jujur, bukan dibesar-besarkan.
- Silhouette score K-Means final (0.1522) tergolong rendah, menunjukkan batas antar cluster tidak terlalu tegas. Pemilihan k=3 didasarkan pada interpretabilitas domain, bukan karena k=3 adalah nilai optimal secara matematis (nilai k=8 memberi silhouette score sedikit lebih tinggi).
- `random_state=42` digunakan secara konsisten di seluruh model untuk memastikan hasil dapat direproduksi.

## Struktur File

```
.
├── Analisis_Numerasi_Siswa_SMP.ipynb   # Notebook utama (Google Colab)
└── README.md                            # Dokumentasi ini
```

---

*Proyek akhir mata kuliah Data Mining, Universitas Negeri Semarang (UNNES).*
