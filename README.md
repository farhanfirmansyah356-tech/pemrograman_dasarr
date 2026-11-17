# SIAKAD Mini - Pengolah Data Nilai Mahasiswa (CLI)

UTS Pemrograman Dasar (TK091) - S1 Teknik Komputer, Universitas AMIKOM Yogyakarta, TA 2026/2027.
Domain data: **Mahasiswa**. Dikerjakan individu. Python 3.9+ dan **hanya standard library** (tanpa `pip install`).

> Nama: ____________  NIM: ____________  (isi sebelum dikumpulkan)

## 1. Deskripsi singkat
| Aspek | Penjelasan |
|---|---|
| Masalah | Staf prodi/dosen wali mengolah nilai mahasiswa secara manual di spreadsheet: rawan salah hitung, data ganda, dan nilai tidak valid. |
| Tujuan | Memvalidasi, membersihkan, menghitung nilai akhir/grade, dan merekap data nilai dari file CSV/JSON secara otomatis. |
| Target pengguna | Staf administrasi prodi, dosen wali, dan dosen pengampu. |
| Input | CSV/JSON dengan kolom `nim,nama,prodi,angkatan,tugas,uts,uas` (nilai 0-100, NIM format `NN.NN.NNNN`). |
| Output | Tabel di terminal, file hasil `.csv`/`.json` (ditambah `nilai_akhir,grade,status`), dan rekap `.json`. |

**Fitur utama:** (1) cari NIM/nama, (2) filter prodi/nilai/grade/angkatan, (3) kelompok per prodi, (4) rekapitulasi + distribusi grade, (5) hapus duplikat NIM, (6) ekspor hasil CSV/JSON, (7) menu interaktif.

**Rumus:** nilai akhir = 30% tugas + 30% UTS + 40% UAS. Grade: A >= 85, B >= 70, C >= 55, D >= 40, E < 40. Lulus bila nilai akhir >= 55.

## 2. Cara menjalankan
```bash
git clone <URL-REPOSITORY>
cd uts_mahasiswa
python main.py tampil                                   # semua data (default data/mahasiswa.csv)
python main.py cari bella                               # pencarian
python main.py filter --prodi komputer --min-nilai 60   # filtering
python main.py filter --grade A --angkatan 2025
python main.py kelompok                                 # pengelompokan per prodi
python main.py rekap                                    # rekapitulasi
python main.py proses -i data/mahasiswa.csv -o output/hasil.csv -r output/rekap.json
python main.py proses -i data/mahasiswa.json -o output/hasil_json.json -r output/rekap_json.json
python main.py menu                                     # menu interaktif
python main.py --debug tampil -i data/x.csv             # traceback lengkap bila error tak terduga
python -m unittest discover -v                          # 14 unit test
```
Di Windows gunakan `python` atau `py`; di Linux/macOS bisa `python3`.

## 3. Struktur program
```
uts_mahasiswa/
|-- main.py                  # entry point
|-- README.md
|-- siakad/                  # package buatan sendiri
|   |-- __init__.py
|   |-- exceptions.py        # exception khusus (SiakadError + turunannya)
|   |-- validator.py         # validasi & normalisasi satu baris data
|   |-- io_file.py           # baca/tulis CSV & JSON
|   |-- analisis.py          # logika murni: nilai, cari, filter, kelompok, rekap, dedup
|   |-- layanan.py           # pipeline: baca -> validasi -> dedup -> hitung nilai
|   |-- laporan.py           # format tampilan teks
|   '-- cli.py               # argparse + menu interaktif
|-- data/                    # contoh input (normal + edge case + data rusak)
|-- output/                  # hasil & log eksekusi (output/log/*.txt)
|-- tests/test_siakad.py     # unit test (unittest)
'-- docs/debugging_demo.py   # skenario debugging (versi sebelum diperbaiki)
```
**Alur dependensi:** `main -> cli -> layanan -> (io_file, validator, analisis)`; `cli -> laporan -> analisis`; `io_file` & `validator` -> `exceptions`. Tidak ada impor melingkar; `analisis` tidak melakukan I/O sehingga mudah diuji.

## 4. Penanganan error (defensive programming)
| Kondisi | Perilaku |
|---|---|
| File tidak ada / bukan file / tanpa izin | `FileTidakDitemukanError` -> pesan jelas, exit code 1 |
| CSV tanpa header / kolom wajib kurang, JSON rusak, ekstensi salah | `FormatDataError` -> exit code 1 |
| Baris tidak valid (nilai `abc`, `150`, NIM salah, nama kosong) | baris **ditolak & dilaporkan**, baris lain tetap diproses |
| NIM ganda | duplikat dibuang (kemunculan pertama dipertahankan) + peringatan |
| Data kosong (hanya header) | rekap aman, tanpa `ZeroDivisionError` |
| Input menu tidak sesuai | diminta ulang sampai valid; Ctrl+C/EOF keluar dengan rapi |
| Gagal menulis output | `PenyimpananGagalError` |
| Error tak terduga | pesan ringkas, exit code 2; `--debug` menampilkan traceback |

## 5. Package & dependency
Hanya standard library: `argparse`, `csv`, `json`, `math`, `os`, `re`, `sys`, `traceback`, `unittest`, `tempfile`. **Tidak ada dependency pihak ketiga.**

## 6. Debugging satu skenario error
`python docs/debugging_demo.py` menghasilkan `ValueError: could not convert string to float: 'abc'` (kode naif mengasumsikan semua nilai angka). Penyebab: baris 1 `data/data_rusak.csv` kolom `uts` berisi `abc`. Perbaikan: `validator.parse_nilai` membungkus `float()` dengan `try/except`, mengubahnya menjadi `DataTidakValidError`, dan `validasi_semua` menolak baris tersebut tanpa menghentikan program. Rincian di lembar jawaban PDF.

## 7. Referensi & bantuan AI
- Dokumentasi Python 3 (docs.python.org): `csv`, `json`, `argparse`, `unittest`.
- Bantuan AI: Claude (Anthropic) digunakan untuk membantu merancang struktur dan menulis kode/dokumentasi. Seluruh kode telah dijalankan dan diuji (lihat `output/log/`). *(Sesuaikan pernyataan ini dengan keadaan sebenarnya.)*
