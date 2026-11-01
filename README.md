# FastSudo

FastSudo adalah framework engine Sudoku berbasis C++ yang memuat solver, analisis kandidat, validasi, dan pencatatan hasil dalam satu program. Implementasi yang tersedia di repositori ini adalah **Sudoku Engine v10.4**.

> **Status versi:** v10.4 adalah versi tercepat yang diketahui saat dokumentasi ini ditulis. Belum tersedia informasi mengenai versi yang lebih baru. Klaim ini adalah status versi proyek, bukan hasil benchmark lintas perangkat keras.

## Fitur

- Memuat papan Sudoku 9x9 dari berkas teks dan memeriksa konflik petunjuk pada baris, kolom, dan kotak 3x3.
- Menampilkan papan serta membaca kandidat sel melalui sistem koordinat Kartesius.
- Menghasilkan laporan koordinat dan kandidat untuk seluruh sel.
- Menyelesaikan Sudoku dengan teknik propagasi kandidat dan pencarian backtracking.
- Menghitung jumlah solusi untuk memeriksa apakah papan tidak memiliki solusi, memiliki solusi unik, atau memiliki lebih dari satu solusi.
- Menampilkan log pengisian, statistik teknik, eliminasi kandidat, jumlah tebakan, dan waktu proses.
- Menyimpan papan hasil solve dan laporan sesi ke berkas.

## Persyaratan

- Kompiler C++ yang mendukung C++17, misalnya GCC atau Clang.
- Terminal yang mendukung input interaktif. Aplikasi membersihkan layar menggunakan kode ANSI.

## Build dan jalankan

Dari direktori root repositori:

```sh
g++ -std=c++17 -O2 src/EngineAllVersion/FastSudo_v10_4.cpp -o fastsudo
./fastsudo
```

Program berjalan sebagai aplikasi terminal interaktif. Masukkan satu perintah pada prompt yang ditampilkan. Berkas input dan keluaran sesi dibaca atau ditulis relatif terhadap direktori kerja saat program dijalankan.

## Format berkas Sudoku

Siapkan berkas teks berisi 81 digit untuk sembilan baris, dari kiri ke kanan dan atas ke bawah. Gunakan `1` sampai `9` untuk petunjuk dan `0` untuk sel kosong. Baris baru maupun karakter non-digit lain diabaikan oleh loader, sehingga format sembilan baris mudah dibaca:

```text
530070000
600195000
098000060
800060003
400803001
700020006
060000280
000419005
000080079
```

Contoh tersebut merupakan teka-teki Sudoku standar. Loader mengambil **81 digit pertama yang ditemukan** dalam berkas. Karena itu, jangan letakkan angka seperti nomor soal, tanggal, atau nomor halaman sebelum papan; angka tersebut dapat dianggap sebagai bagian dari papan. Baris dengan kurang dari 81 digit menyebabkan kesalahan.

## Sistem koordinat

Selain indeks baris-kolom, engine menggunakan koordinat Kartesius:

- `x` bergerak dari `-4` di kolom paling kiri sampai `4` di kolom paling kanan.
- `y` bergerak dari `4` di baris paling atas sampai `-4` di baris paling bawah.
- Pusat papan adalah `(0, 0)`.
- `/hitungxy` menampilkan nilai, baris horizontal, kolom vertikal, dan kandidat yang valid pada koordinat yang diminta.

Misalnya, koordinat pojok kiri atas adalah `(-4, 4)` dan pojok kanan bawah adalah `(4, -4)`.

## Perintah terminal

| Perintah | Fungsi |
| --- | --- |
| `/load <file>` | Memuat papan dari berkas. Loader memvalidasi konflik pada baris, kolom, dan kotak 3x3. |
| `/hitungxy` | Meminta nilai `X` dan `Y`, lalu menampilkan nilai sel dan kandidat yang memenuhi aturan Sudoku. |
| `/calculate` | Membuat laporan koordinat dan kandidat seluruh sel ke `data_koordinat_<session>.txt`. |
| `/solve` | Mencari dan menampilkan solusi pertama. Jika laporan koordinat belum tersedia, program menjalankan calculate dan meminta perintah `/solve` dijalankan lagi. |
| `/count` | Menelusuri solusi untuk memeriksa jumlah solusi dan menampilkan statistik solver. Dapat memerlukan waktu lebih lama daripada `/solve`. |
| `/seesolve` | Menampilkan kembali papan hasil solve yang tersimpan untuk sesi aktif. |
| `/back` | Kembali dari tampilan papan hasil solve ke papan awal. Perintah ini tersedia saat hasil solve sedang ditampilkan. |
| `/quit` | Keluar dari program. |

### Alur umum

1. Jalankan program, lalu muat papan, misalnya `/load puzzle.txt`.
2. Jalankan `/calculate` untuk membuat laporan kandidat.
3. Jalankan `/solve` untuk memperoleh solusi pertama.
4. Jalankan `/count` bila perlu memeriksa apakah solusinya unik.
5. Gunakan `/seesolve` untuk melihat hasil tersimpan atau `/back` untuk kembali ke papan awal.

`/count` tidak mensyaratkan laporan `/calculate`. Saat `/solve` membutuhkan laporan yang belum ada, program membuatnya dan meminta Anda mengulangi `/solve`.

## Cara kerja solver

Kandidat setiap sel kosong disimpan sebagai bitmask. Engine menyiapkan tabel statis untuk 27 unit Sudoku (9 baris, 9 kolom, dan 9 kotak) beserta relasi antar sel. Propagasi memproses unit yang berubah menggunakan worklist, cache posisi digit dalam unit (Unit-Digit Position/UDP), dan antrean X-Wing.

Teknik yang digunakan:

- Naked Single dan Hidden Single pada baris, kolom, atau kotak.
- Locked Candidates: Claiming dan Pointing.
- Naked Pair, Naked Triple, dan Naked Quad.
- X-Wing pada baris maupun kolom.
- Backtracking dengan pemilihan sel MRV (sel dengan kandidat paling sedikit), pemutus seri berbasis jumlah tetangga kosong, dan urutan nilai LCV (nilai yang mengeliminasi kandidat tetangga paling sedikit lebih dahulu).

Teknik logika mengurangi kandidat terlebih dahulu. Jika belum selesai, pencarian mencoba kandidat dan membatalkan perubahan cabang yang gagal. `/solve` berhenti setelah menemukan satu solusi. `/count` mencari seluruh solusi yang dapat ditemukan untuk mengklasifikasikan papan, sehingga papan dengan banyak cabang dapat membutuhkan waktu yang signifikan.

## Hasil dan berkas sesi

Semua berkas berikut dibuat di direktori kerja program:

- `data_koordinat_<session>.txt`: laporan nilai, koordinat, dan kandidat tiap sel; dibuat oleh `/calculate`.
- `data_solve_<session>.txt`: papan solusi, log pengisian, statistik teknik, dan waktu solve; dibuat oleh `/solve` setelah solusi ditemukan.
- `<nama-berkas-input>.mem`: salinan papan yang digunakan untuk mendeteksi perubahan input pada pemuatan berikutnya.
- Berkas input dapat diberi penanda `[id_session: N]` oleh program bila belum memiliki ID sesi. ID sesi juga ditulis pada laporan keluaran.

Jika isi papan yang dimuat berubah, program memperbarui berkas `.mem` dan menghapus laporan koordinat serta hasil solve lama untuk sesi tersebut. Jaga berkas input dan keluaran tetap di lokasi yang dapat dibaca/ditulisi program.

## Batasan dan catatan

- Implementasi ini ditujukan untuk Sudoku standar 9x9 dengan angka 1-9; varian seperti diagonal, killer, atau ukuran papan lain tidak didukung.
- Validasi awal menolak petunjuk yang berulang dalam baris, kolom, atau kotak 3x3. Papan yang tidak memiliki solusi baru diketahui saat proses solve/count.
- Waktu proses bergantung pada teka-teki, kompiler, dan perangkat keras. Statistik waktu teknik adalah pengukuran runtime, bukan jaminan performa untuk semua papan.
- Source yang tersedia dalam repositori ini adalah `src/EngineAllVersion/FastSudo_v10_4.cpp`. Belum ada dokumentasi versi terbaru atau perbandingan benchmark resmi di repositori ini.
