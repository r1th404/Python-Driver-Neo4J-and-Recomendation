# UTS Graf Sains Data: Sistem Rekomendasi RetailRocket dengan Neo4j

Ujian Tengah Semester **Graf Sains Data** (S1 Sains Data, Universitas Negeri Surabaya, Gasal 2026/2027).
Membangun sistem rekomendasi berbasis graf (user-based collaborative filtering) pada dataset e-commerce RetailRocket menggunakan Neo4j dan Python.

**Nama:** Naufal Harith | **Fold:** 3

| Fold | Periode | Fungsi |
|---|---|---|
| P1 (training) | 03/05/2015 - 03/07/2015 | membangun graf, similarity, rekomendasi |
| P2 (testing) | 12/07/2015 - 27/07/2015 | ground truth evaluasi Hit@5 dan NDCG@5 |

## Ringkasan

- **Data:** `events.csv` RetailRocket (Roman Zykov, Kaggle): `timestamp`, `visitorid`, `event`, `itemid`, `transactionid`.
- **Graf:** `(Visitor)-[:VIEWED | :ADDED_TO_CART | :BOUGHT]->(Item)`, bipartit. Subgraph P1 berisi 637.379 `Visitor`, 166.996 `Item`, dan 1.264.004 relasi.
- **VSM:** matriks sparse user × item (sparsity 99,9991%), bobot `VIEWED` = 1, `ADDED_TO_CART` = 3, `BOUGHT` = 5.
- **Proyeksi terpilih:** User-User (92,94% user punya tetangga, dibanding 60,68% item pada Item-Item).
- **Similarity:** cosine similarity, ditulis balik ke Neo4j sebagai relasi `SIMILAR_TO {score}`.
- **Rekomendasi:** KNN (k = 5) dengan aturan tetangga harus punya minimal 1 item baru.

## Hasil utama

| Evaluasi | Hit@5 | NDCG@5 |
|---|---|---|
| User acuan (`visitorid` 121532) | 0 | 0,0000 |
| KNN, seluruh 2.872 user (84 hit) | 0,0292 | 0,0108 |
| Baseline popularitas, seluruh 2.872 user (9 hit) | 0,0031 | 0,0014 |

KNN sekitar 9 kali lebih baik daripada baseline popularitas, dengan interval kepercayaan 95% yang tidak tumpang tindih. Nilai absolut rendah karena data sangat sparse: hanya 4,67% user yang item targetnya ada di kandidat rekomendasi (batas atas Hit@5).

## Struktur repositori

```
.
├── README.md
├── requirements.txt
├── .gitignore
├── Graph.ipynb              # notebook utama (jawaban 1.1 - 2.2)
├── Graph_Seluruh User.ipynb
├── data_after_extrac/
│   ├── events_p1.csv          # hasil filter fold training
│   ├── events_p2.csv          # hasil filter fold testing
│   └── hasil_eval_semua_user.csv
└── img/
    ├── graf_sebelum.png       # graf Neo4j sebelum similarity
    └── graf_sesudah.png       # graf Neo4j sesudah similarity
```

`events.csv` asli tidak disertakan karena berukuran besar. Unduh dari [Kaggle: RetailRocket](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) dan taruh di `data/events.csv` jika ingin menjalankan ulang proses filter.

## Cara menjalankan

1. **Buat dan aktifkan environment** (Windows PowerShell):
```powershell
   python -m venv myenv
   myenv\Scripts\Activate.ps1
   python -m pip install -r requirements.txt
```
2. **Jalankan Neo4j** secara lokal (Neo4j Desktop atau Docker) dan pastikan layanan aktif di `bolt://localhost:7687`.
3. **Isi kredensial** di sel koneksi notebook (`USER`, `PASSWORD`). Kredensial tidak disimpan di repositori.
4. **Buka `uts_gsd.ipynb`** di VS Code, pilih kernel `myenv`, lalu jalankan sel dari atas ke bawah.

Catatan: impor P1 ke Neo4j (1,26 juta baris) memakan waktu belasan menit, dan relasi dibuat dengan `CREATE`, sehingga **jangan menjalankan sel impor dua kali** tanpa mengosongkan graf terlebih dahulu.

## Keputusan desain

- **Bobot interaksi** (1, 3, 5) adalah asumsi berdasarkan urutan kekuatan sinyal, bukan dari data, dan dapat diganti.
- **Definisi hit:** item rekomendasi dianggap tepat jika user berinteraksi dengannya di P2 (event apa pun).
- **Evaluasi di @5:** daftar rekomendasi berisi 5 item; 3 item teratas menjadi item yang diiklankan.
- **Aturan tetangga:** KNN memilih 5 tetangga terdekat yang memiliki minimal 1 item yang belum dilihat user acuan, karena tetangga teratas pada user acuan tidak dapat menyumbang item baru.
- **Reproducibility:** pemilihan user acak memakai `np.random.default_rng(42)`. Zona waktu `timestamp` adalah UTC.

## Keterbatasan

- Data sangat sparse (79,51% user hanya mengakses 1 item), sehingga sinyal kemiripan lemah.
- Kandidat rekomendasi hanya berasal dari item milik tetangga (rata-rata 17,7 item), sehingga cakupannya sempit.
- Evaluasi hanya mencakup 2.872 user yang muncul di P1 dan P2.

## Teknologi

Python 3.11, Neo4j (driver `neo4j` 6.x), pandas, NumPy, SciPy (sparse), scikit-learn, NetworkX, Matplotlib, tqdm.