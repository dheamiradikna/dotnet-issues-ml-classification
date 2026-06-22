# 🤖 Klasifikasi Kualitas GitHub Issues .NET — ML Pipeline

Proyek ini melakukan **scraping**, **pelabelan**, **ekstraksi fitur**, dan **pelatihan model Machine Learning** untuk mengklasifikasikan kualitas diskusi pemrograman .NET dari GitHub Issues.

---

## 📁 Struktur File

```
├── scraping_github_dotnet.ipynb     # Notebook scraping GitHub Issues via REST API
├── pelatihan_model_ml.ipynb         # Notebook pelatihan 3 skema ML
├── requirements.txt                 # Dependensi Python
└── so_dotnet_dataset_scraped.csv    # Dataset hasil scraping (3.365 baris)
```

---

## 🔄 Alur Penggunaan

```
scraping_github_dotnet.ipynb
        │
        │  menghasilkan
        ▼
so_dotnet_dataset_scraped.csv
        │
        │  digunakan sebagai input
        ▼
pelatihan_model_ml.ipynb
        │
        ▼
  Model + Hasil Evaluasi
```

---

## 📊 Dataset

| Item | Detail |
|------|--------|
| File | `so_dotnet_dataset_scraped.csv` |
| Sumber | GitHub REST API v3 (scraping mandiri) |
| Total data | 3.365 baris |
| Topik | bug, aspnetcore, ef-core, blazor, wpf, maui, signalr, linq, performance, dll |
| Kolom utama | `id`, `judul`, `topik`, `labels_github`, `state`, `komentar`, `reactions`, `body_clean`, `panjang_judul`, `panjang_body`, `ada_code_block`, `ditutup`, `url` |

### Aturan Pelabelan (3 Kelas Otomatis)

| Label | Kondisi dari kolom CSV |
|-------|------------------------|
| 🟢 `positif` | `ditutup=1` AND (`komentar≥3` OR label done/feature) |
| 🔴 `negatif` | `ditutup=0` AND `komentar=0` |
| 🟡 `netral` | Kondisi lainnya |

---

## 🧪 3 Skema Pelatihan

| Skema | Algoritma | Fitur | Split | Test Acc |
|-------|-----------|-------|-------|----------|
| 1 | Logistic Regression (C=1.5) | TF-IDF + Numerik + Label GitHub | 80/20 | ≥ 85% |
| 2 | LinearSVC (C=1.0) | TF-IDF + Numerik + Label GitHub + Topik | 80/20 | ≥ 85% |
| 3 | Logistic Regression (C=3.0) | TF-IDF + Numerik + Label GitHub + Topik | 70/30 | ≥ 85% |

### Ekstraksi Fitur

| Jenis | Sumber Kolom CSV | Keterangan |
|-------|-----------------|------------|
| TF-IDF | `judul` (3×) + `body_clean` | ngram (1,3), max 10.000 fitur |
| Numerik | `komentar`, `reactions`, `panjang_judul`, `panjang_body`, `ada_code_block`, `ditutup` | Log-transform |
| Label GitHub | `labels_github` | One-hot: bug, feature, done, dll |
| Topik | `topik` | One-hot dari kolom topik scraping |

### ✅ Anti Data Leakage

Pipeline ekstraksi fitur mengikuti urutan yang benar:

```python
# 1. Split dulu
idx_tr, idx_te = train_test_split(idx, ...)

# 2. Fit TF-IDF HANYA pada data train
tfidf.fit(texts[idx_tr])

# 3. Transform train
X_tr_tfidf = tfidf.transform(texts[idx_tr])

# 4. Transform test — BUKAN fit_transform!
X_te_tfidf = tfidf.transform(texts[idx_te])
```

---

## 🔮 Output Inference

Model menghasilkan kelas kategorikal:

```
🟢 positif  — Issue berkualitas tinggi (resolved, banyak diskusi)
🟡 netral   — Issue cukup (ada respons, belum selesai)
🔴 negatif  — Issue kurang berkualitas (tidak ada respons)
```

---

## 🚀 Cara Menjalankan di Google Colab

### Langkah 1 — Scraping

1. Buka `scraping_github_dotnet.ipynb` di Google Colab
2. *(Opsional)* Isi `GITHUB_TOKEN` untuk rate limit lebih tinggi
3. Klik **Runtime → Run all**
4. Download file `so_dotnet_dataset_scraped.csv` yang dihasilkan

### Langkah 2 — Pelatihan

1. Buka `pelatihan_model_ml.ipynb` di Google Colab
2. Jalankan **Cell 2** → upload `so_dotnet_dataset_scraped.csv`
3. Klik **Runtime → Run all**
4. Hasil akurasi dan model tersimpan otomatis

### Install dependensi (lokal)

```bash
pip install -r requirements.txt
```

---

## 📦 Requirements

```
requests>=2.31.0
pandas>=2.0.0
numpy>=1.24.0
scikit-learn>=1.3.0
scipy>=1.11.0
matplotlib>=3.7.0
seaborn>=0.12.0
tqdm>=4.65.0
openpyxl>=3.1.0
```

---

## 📝 Catatan

- Scraping menggunakan **GitHub REST API v3** tanpa token (limit 10 req/menit)
- Untuk scraping lebih cepat, daftarkan token gratis di [github.com/settings/tokens](https://github.com/settings/tokens)
- Dataset bersifat **real** — bukan data sintetis
- Semua file dalam repo ini saling terhubung melalui nama file `so_dotnet_dataset_scraped.csv`
