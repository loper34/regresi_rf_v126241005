## Estimasi Biomassa Atas Permukaan (AGBD) Menggunakan Citra Satellite Embedding dan Random Forest Regressor di Sulawesi Selatan

### Identitas Penulis
* **Nama:** Ashabul Kahfi
* **NIM:** V126241005
* **Mata Kuliah:** Praktek Pemrograman Spasial
* **Tanggal:** 20 - November - 2026

---

### Tautan Penting
* **Link Google Colab:** (https://colab.research.google.com/drive/1-UcLIz6Wj6Ip7yKqfRXc3UyA416PrGH4?usp=sharing)
* **Nama File Notebook di Repository:** `Regresi_AGBD_Menggunakan_Random_Forest.ipynb`

---

### Latar Belakang
Estimasi Biomassa Atas Permukaan (*Aboveground Biomass Density* - AGBD) merupakan salah satu parameter kunci dalam pemantauan siklus karbon global, mitigasi perubahan iklim, serta pengelolaan hutan berkelanjutan. Pengukuran biomassa secara langsung di lapangan (*in-situ*) membutuhkan biaya yang besar, waktu yang lama, dan aksesibilitas yang sulit pada area hutan yang luas.

Untuk mengatasi kendala tersebut, integrasi teknologi pengindraan jauh dan kecerdasan buatan (*machine learning*) menjadi solusi yang sangat efisien. Wilayah **Sulawesi Selatan** dipilih sebagai wilayah kajian karena memiliki karakteristik ekosistem hutan tropis yang beragam dan memiliki peran penting dalam menjaga keseimbangan ekologis di koridor biogeografi Wallacea.

---

### Tujuan
1. Membangun model prediksi spasial untuk mengestimasi Biomassa Atas Permukaan (AGBD) di wilayah hutan Sulawesi Selatan.
2. Mengintegrasikan data target LiDAR GEDI dengan prediktor multispektral canggih (*Google Satellite Embedding*).
3. Mengevaluasi performa model *Random Forest Regressor* dan mengidentifikasi prediktor yang paling berpengaruh terhadap sebaran biomassa.

---

### Wilayah Kajian
Wilayah kajian proyek ini mencakup area hutan di Provinsi **Sulawesi Selatan, Indonesia**. Batas administrasi wilayah diperoleh dari database FAO GAUL level 1, yang kemudian difilter menggunakan tutupan lahan hutan dari MODIS untuk membatasi analisis hanya pada area bervegetasi hutan asli.

<img width="601" height="692" alt="image" src="https://github.com/user-attachments/assets/5bdd47b1-e72a-4993-b109-11f559b85e7e" />

---

### Data

#### 1. Data Target (AGBD)
* **Sumber Dataset:** GEDI L4B (*LARSE/GEDI/GEDI04_B_002*)
* **Periode:** Tahun 2020
* **Resolusi:** Grid raster ~1 km (resolusi spasial asli hasil interpolasi footprint GEDI)
* **Deskripsi:** Merupakan estimasi rata-rata kerapatan biomassa di atas permukaan tanah (*Mean Aboveground Biomass Density* / MU) dalam satuan Megagram per hektar (Mg/ha).

#### 2. Data Prediktor (Fitur X)
* **Sumber Dataset:** Google Satellite Embedding (*GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL*)
* **Periode:** Tahun 2020
* **Resolusi:** Berbasis pemrosesan mosaik citra komposit tahunan
* **Alasan Pemilihan:** *Satellite Embedding* mengemas representasi spektral, tekstur, dan informasi kontekstual bumi ke dalam 64 band laten numerik (A00 hingga A63). Ini meminimalkan bias manual akibat ekstraksi indeks vegetasi tradisional dan memberikan fitur yang jauh lebih kaya untuk algoritma *machine learning*.

---

### Pra-pemrosesan Data
1. **Filtering & Clipping:** Memotong wilayah menggunakan batas administrasi Sulawesi Selatan (FAO GAUL 2015).
2. **Forest Masking:** Menggunakan citra MODIS MCD12Q1 (LC_Type1 kelas IGBP 1-5) untuk memisahkan area hutan dari non-hutan.
3. **Mosaicking & Stacking:** Menggabungkan 64 band embedding dengan band target GEDI AGBD.
4. **Data Extraction:** Mengunduh citra dari server GEE menggunakan *engine* `xee` ke bentuk Xarray Dataset dengan grid resolusi spasial 0.01 derajat (~1.1 km), lalu mengubahnya ke bentuk Pandas DataFrame dan menghapus nilai kosong (NaN).

---

### Metode

* **Skema Pembagian Data:** Data dibagi menjadi **70% Data Latih (Train Set)** sebesar 8.518 sampel dan **30% Data Uji (Test Set)** sebesar 3.651 sampel dengan metode *random split* (random state = 42).
* **Hyperparameter Tuning:** Dilakukan pencarian kombinasi terbaik secara manual (*Manual Grid Search*) melalui validasi silang (5-fold Cross-Validation) menggunakan metrik $R^2$:
  * Rentang pencarian: `max_features` = [1, 2, 3, 4, 5] dan `n_estimators` = [100, 200, 300, 400, 500].
  * **Parameter Terbaik:** `{'max_features': 4, 'n_estimators': 500}` menghasilkan nilai rata-rata Cross-Validation $R^2$ sebesar **0.577**.
* **Metrik Evaluasi:** Menggunakan Koefisien Determinasi ($R^2$) untuk mengukur proporsi variansi target yang berhasil dijelaskan oleh model, *Root Mean Squared Error* (RMSE) untuk melihat deviasi rata-rata prediksi dalam satuan Mg/ha, dan korelasi Pearson ($r$) untuk mengukur keeratan hubungan linear antara aktual dan prediksi.

---

### Hasil

#### Tabel Metrik Performa Model
| Kumpulan Data | R² (R-squared) | RMSE (Mg/ha) | Korelasi Pearson ($r$) |
| :--- | :---: | :---: | :---: |
| **Data Latih (Train)** | 0.943 | 35.82 | - |
| **Data Uji (Test)** | 0.618 | 92.07 | 0.789 |
| **Out-of-Bag (OOB)** | 0.579 | - | - |

#### Visualisasi Kunci
1. **Distribusi Target (Gambar 02):** Menunjukkan bahwa pembagian subset latih dan uji memiliki representasi distribusi nilai AGBD yang sangat identik.
2. **Scatter Plot Aktual vs Prediksi (Gambar 03):** Menunjukkan korelasi yang cukup kuat ($r = 0.79$) antara nilai biomassa aktual GEDI dengan hasil estimasi Random Forest.
3. **Feature Importance (Gambar 04):** 
   * Band **A06** merupakan prediktor paling berpengaruh dengan kontribusi kepentingan sebesar **5.36%**, disusul oleh **A24** sebesar **4.29%**.

*(Unggah gambar-gambar hasil plot Anda ke folder repo GitHub Anda agar otomatis muncul di sini)*
* **Scatter Plot:** `![Scatter Plot](03_scatter_obs_vs_pred.png)`
* **Feature Importance:** `![Feature Importance](04_feature_importance.png)`
* **Peta Prediksi AGBD:** `![Peta Prediksi](05_peta_prediksi_agbd.png)`

---

### Pembahasan
1. **Analisis Overfitting:** Model menunjukkan adanya gejala *overfitting* yang cukup terlihat, di mana $R^2$ data latih sangat tinggi (0.943) sedangkan pada data uji menurun menjadi 0.618. Hal ini wajar terjadi pada algoritma berbasis pohon keputusan (*decision trees*) seperti Random Forest ketika menghadapi data spasial yang memiliki autokorelasi spasial tinggi.
2. **Prediktor Berpengaruh:** Band representasi spektral laten **A06** dan **A24** memegang peranan kunci. Secara ilmiah, band laten ini kemungkinan besar menangkap pantulan spektral inframerah dekat (*Near-Infrared* / NIR) dan *Red-Edge* yang sangat sensitif terhadap kerapatan tajuk hutan, indeks luas daun (*LAI*), dan kadar air vegetasi.
3. **Kelemahan Estimasi (Under/Overestimate):** Berdasarkan scatter plot aktual vs prediksi, model cenderung melakukan *underestimation* pada area dengan kerapatan biomassa sangat tinggi (di atas 600 Mg/ha). Hal ini disebabkan oleh efek saturasi sensor optik spektral yang kesulitan menembus kanopi hutan yang sangat rapat dan berlapis.
4. **Keterbatasan & Saran:** 
   * Keterbatasan resolusi grid data target (~1 km) yang cukup kasar.
   * Disarankan untuk menggunakan data radar aktif (seperti ALOS PALSAR) atau menambahkan variabel topografi (DEM SRTM) untuk memperkuat sensitivitas model pada biomassa tinggi.

---

### Kesimpulan
Model regresi berbasis *Random Forest* berhasil memetakan estimasi spasial kerapatan biomassa atas permukaan (AGBD) di wilayah Sulawesi Selatan dengan akurasi moderat ($R^2$ uji = 0.62 dan RMSE = 92.07 Mg/ha). Metode ekstraksi menggunakan *Google Satellite Embedding* terbukti efektif menjadi alternatif penyedia fitur prediktor yang masif dan representatif tanpa memerlukan ekstraksi manual.

---

### Cara Menjalankan Ulang
1. **Persiapan:** Buka Google Colab dan pastikan Anda terhubung ke Google Drive Anda.
2. **Otorisasi GEE:** Pastikan Anda memiliki akun Google Earth Engine yang aktif dan ubah Project ID pada langkah inisialisasi GEE:
   ```python
   ee.Initialize(project='project-id-anda')
Eksekusi: Jalankan setiap sel secara berurutan mulai dari CELL 1 hingga CELL 14.

Output: Hasil pemodelan berupa file model .pkl dan raster akhir .tif akan otomatis tersimpan di folder Google Drive Anda pada path /content/gdrive/MyDrive/Praktek_diniyrti/Praktek_Regresi_RF.

Referensi
GEDI L4B Dataset: Dubayah, R., et al. (2022). GEDI L4B Footprint Aboveground Biomass Density, Version 2.
Google Satellite Embedding Dataset Documentation.
Gorelick, N., et al. (2017). Google Earth Engine: Planetary-scale geospatial analysis for everyone.
