# Klasifikasi Sentimen Ulasan Produk E-Commerce

## 1. Problem Statement & Dataset

Tim produk e-commerce ingin membangun fitur otomatis untuk mengklasifikasikan sentimen ulasan pelanggan (**positif/negatif**) pada halaman produk. Tujuannya adalah membantu tim produk dan penjual memahami persepsi pelanggan terhadap suatu produk secara cepat, tanpa harus membaca seluruh ulasan satu per satu.

Sebagai tahap awal sebelum implementasi ke sistem produksi, dilakukan eksperimen untuk membandingkan dua kandidat pendekatan klasifikasi sentimen:

1. **Model klasik (Scikit-learn)** — dilatih di atas dataset ulasan pelanggan yang tersedia.
2. **LLM API (Gemini AI)** — menggunakan prompt engineering tanpa proses training.

Dataset yang digunakan berupa kumpulan ulasan pelanggan berlabel sentimen (positif/negatif) pada produk e-commerce, yang dibagi menjadi data untuk keperluan training (khusus pendekatan model klasik) dan data evaluasi yang sama untuk kedua pendekatan agar perbandingan bersifat adil (apple-to-apple).

## 2. Ringkasan Eksperimen Kedua Pendekatan

### Pendekatan 1 — Model Klasik (Scikit-learn)
- Ulasan diproses menjadi representasi fitur (misalnya melalui teknik vektorisasi teks).
- Model klasifikasi dilatih menggunakan Scikit-learn di atas data ulasan yang telah dilabeli.
- Model yang telah dilatih kemudian digunakan untuk memprediksi sentimen pada data evaluasi.

### Pendekatan 2 — LLM API (Gemini AI)
- Tidak ada proses training; setiap ulasan dikirim ke Gemini AI API bersama sebuah prompt yang dirancang untuk menghasilkan klasifikasi sentimen (positif/negatif).
- Prediksi diambil langsung dari respons API tanpa fine-tuning tambahan.

Kedua pendekatan dievaluasi pada dataset ulasan yang sama menggunakan metrik **accuracy, precision, recall, F1-score**, serta diukur **latency** (waktu eksekusi total dan rata-rata per ulasan).

## 3. Tabel Hasil Evaluasi dan Perbandingan

| Metrik                            | Scikit-learn (Model Klasik) | Gemini AI (LLM API) |
|----------------------------------|:----------------------------:|:-------------------:|
| Accuracy                          | 1.00                        | 1.00                |
| Precision                         | 1.00                        | 1.00                |
| Recall                            | 1.00                        | 1.00                |
| F1-score                          | 1.00                        | 1.00                |
| Latency total (detik)             | 0.000572                    | 77.647170           |
| Latency rata-rata/ulasan (detik)  | 0.000014                    | 1.941179            |

> Catatan: Kedua pendekatan menghasilkan skor evaluasi yang identik (1.00) pada dataset uji yang digunakan. Perbedaan paling signifikan terletak pada **latency**, di mana Scikit-learn jauh lebih cepat (~135.000x) dibandingkan Gemini AI per ulasan.

## 4. Analisis Trade-off dan Limitation

### 4.1 Performa
Kedua pendekatan memiliki skor akurasi, precision, recall, dan f1 score sama. Namun, pendekatan model ML secara latensi jauh lebih rendah

**Catatan penting:** dataset hanya berisi 200 review dan test set hanya 40 review. Karena ukuran test set kecil, satu prediksi yang berubah dapat menggeser metrik sekitar 2,5 percentage points. Hasil ini sebaiknya dianggap sebagai hasil eksperimen pada dataset ini, bukan generalisasi untuk seluruh populasi customer.

### 4.2 Implementasi dan maintenance
- **Scikit-learn:** membutuhkan preprocessing, training, evaluasi, dan maintenance model. Setelah model terlatih, inference lokal relatif sederhana.
- **Gemini:** implementasi klasifikasi dapat lebih cepat karena tidak perlu melatih model, tetapi aplikasi bergantung pada API, credential, network, quota/rate limit, dan perubahan model/API.

### 4.3 Biaya
Model klasik tidak memiliki biaya API per prediction setelah infrastruktur tersedia, sedangkan Gemini memiliki biaya berbasis token pada paid tier.

### 4.4 Limitations
1. Dataset kecil (200 baris) sehingga hasil belum cukup untuk menyimpulkan performa production-scale.
2. Split hanya satu kali (`random_state=42`); belum ada repeated cross-validation atau confidence interval.
3. Baseline ML hanya menggunakan satu algoritma (Logistic Regression) dan satu skema TF-IDF.
4. Gemini dievaluasi dengan satu prompt dan satu model, sehingga hasil dapat berubah jika prompt/model/parameter diubah.
5. LLM API memiliki faktor eksternal seperti latency, quota, network failure, dan perubahan model.


## 5. Rekomendasi Technical Approach

 Pada dataset ini, kedua pendekatan menghasilkan Accuracy, Precision, Recall, dan F1-Score sempurna, walaupun terdapat limitasi dataset seperti yang telah disebutkan. Dengan mempertimbangkan kebutuhan use case yaitu menampilkan klasifikasi sentimen ulasan pada halaman produk, technical approach yang dipilih adalah menggunakan model ML Scikit-Learn karena secara performa setara dengan LLM Gemini API tetapi tidak membutuhkan biaya dan latensi yang jauh lebih rendah, dengan catatan bahwa hasil eksperimen terbatas pada dataset dan konfigurasi yang digunakan.
