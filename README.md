Galeri Karya Komputer
Galeri Karya Komputer adalah aplikasi web berbasis Google Apps Script untuk mengelola data siswa, kelas, proyek pembelajaran komputer, dan karya yang dikumpulkan siswa. Aplikasi menggunakan React untuk antarmuka, Google Sheets untuk database, serta Google Drive untuk menyimpan file karya.
Aplikasi menyediakan area kerja guru untuk mengelola siswa dan proyek, serta rancangan area siswa untuk mengakses tugas dan karya. Dokumentasi ini disusun dari dua sumber kode yang diberikan: `Code.gs` dan `Index.html`.
> **Catatan kelengkapan kode:** pada `Index.html` yang diberikan, aplikasi memanggil komponen `StudentDashboard`, `StudentUpload`, dan `StudentGallery`, tetapi definisi ketiga komponen tersebut tidak ditemukan di dalam file. Karena itu, fitur tampilan dan alur siswa belum dapat dipastikan berjalan sampai komponen tersebut tersedia.
Daftar Isi
Fitur Aplikasi
Teknologi
Arsitektur Aplikasi
Struktur Database
Fitur Guru
Area Siswa
Pengelolaan File dan Scratch
Sinkronisasi Data
Instalasi dan Deployment
Akun Awal
Daftar Fungsi Backend
Preview Lokal
Catatan Teknis dan Keamanan
Fitur Aplikasi
Login dengan pilihan peran guru atau siswa.
Dashboard guru dengan ringkasan jumlah siswa, proyek, karya, dan siswa yang belum mengumpulkan tugas aktif.
Manajemen data siswa: tambah, edit, cari, dan hapus.
Import data siswa dari Excel (`.xlsx` / `.xls`) dan unduh template Excel.
Pengelolaan proyek dengan target semua kelas atau beberapa kelas tertentu.
Pengaturan instruksi, batas waktu, dan status proyek.
Galeri karya siswa yang dapat difilter berdasarkan kelas.
Tampilan grid karya dan pratinjau gambar.
Penilaian karya menggunakan nilai dan catatan/feedback.
Dukungan pengambilan dan pengunduhan file Scratch `.sb3`.
Sinkronisasi data manual dan pemuatan ulang otomatis setiap 10 detik.
Notifikasi status dan dialog konfirmasi tindakan.
Panduan deployment Google Apps Script di dalam antarmuka.
Teknologi
Teknologi	Peran
Google Apps Script	Backend, Web App, dan akses layanan Google
Google Sheets	Database untuk pengguna, kelas, proyek, dan submission
Google Drive	Penyimpanan file karya yang diunggah
HTML Service	Menyajikan `Index.html` sebagai halaman Web App
React 18 dan ReactDOM	Antarmuka dan pengelolaan state
Babel Standalone	Transformasi JSX di browser
Tailwind CSS	Styling antarmuka
xlsx-js-style	Membuat template Excel dan membaca file Excel
Web Crypto API	Hash SHA-256 password pada proses login guru
Library frontend dimuat melalui CDN pada `Index.html`.
Arsitektur Aplikasi
Aplikasi terdiri dari dua bagian utama:
1. Backend — `Code.gs`
Backend berjalan di Google Apps Script dan bertanggung jawab untuk:
Menyajikan halaman melalui `doGet()`.
Menyiapkan sheet database pada spreadsheet aktif.
Membaca dan mengelola data melalui operasi CRUD.
Membuat folder upload di Google Drive.
Menyimpan file submission ke Drive dan menyimpan URL-nya di database.
Mengambil isi file Drive dalam bentuk Base64 untuk kebutuhan frontend.
2. Frontend — `Index.html`
Frontend dibangun dengan React dan berkomunikasi dengan backend melalui `google.script.run`. Fungsi `callGas()` membungkus pemanggilan tersebut menjadi Promise dan menggunakan success handler serta failure handler.
Alur data
Halaman aplikasi dimuat oleh Google Apps Script.
Frontend memanggil `getInitialData()` untuk mengambil data awal.
Guru mengelola data siswa dan proyek melalui antarmuka.
Data dikirim ke backend menggunakan `addDocument()`, `updateDocument()`, atau `deleteDocument()`.
Saat file submission dikirim sebagai Data URL, backend mengunggah file ke Google Drive.
Data submission beserta URL file disimpan di Google Sheets.
Frontend memuat ulang data untuk menampilkan perubahan.
Struktur Database
Backend memastikan empat sheet tersedia di Google Spreadsheet:
Sheet	Kegunaan
`users`	Data akun guru dan siswa
`classes`	Data kelas
`projects`	Data proyek/tugas
`submissions`	Data pengumpulan karya siswa
Setiap sheet menggunakan dua kolom:
Kolom	Keterangan
`ID`	ID unik untuk setiap data
`DATA_JSON`	Properti data yang disimpan dalam format JSON
Contoh ilustrasi isi `DATA_JSON`:
```json
{
  "role": "siswa",
  "name": "Andi Saputra",
  "classId": "class-1",
  "pin": "1234"
}
```
Objek aktual dapat memiliki properti tambahan sesuai koleksinya. Ketika data dibaca oleh `getInitialData()`, ID dari kolom pertama ditambahkan ke objek sebagai properti `id`.
Relasi data utama
Siswa pada `users` terhubung ke kelas melalui `classId`.
Proyek pada `projects` memiliki target kelas melalui `classId` (dapat berupa `all`, satu ID kelas, atau array ID kelas).
Submission pada `submissions` menghubungkan siswa melalui `studentId` dan proyek melalui `projectId`.
Fitur Guru
1. Dashboard
Dashboard menampilkan ringkasan:
Siswa Terdaftar: jumlah akun siswa.
Total Project: jumlah proyek yang dibuat.
Total Karya: jumlah submission yang tersimpan.
Belum Kumpul (Aktif): perkiraan jumlah pengumpulan yang belum masuk untuk proyek berstatus aktif.
Dashboard juga menampilkan aktivitas submission terbaru dan pintasan menuju pengelolaan proyek serta galeri.
2. Data Siswa
Guru dapat:
Menambahkan siswa.
Mengubah nama, kelas, dan PIN siswa.
Mencari siswa berdasarkan nama.
Menghapus satu siswa.
Memilih beberapa siswa dan menghapusnya sekaligus.
Mengunduh template Excel.
Mengimpor data siswa dari file `.xlsx` atau `.xls`.
Format kolom template Excel:
Nama Siswa	Nama Kelas	PIN
Andi Saputra	Kelas 5A	1234
Budi Santoso	Kelas 5B	5678
Saat import, aplikasi mencari kelas berdasarkan nama. Jika kelas belum tersedia, kelas baru dibuat. Jika siswa dengan nama dan kelas yang sama sudah ada, PIN siswa tersebut diperbarui; jika belum ada, akun siswa baru dibuat.
3. Kelola Project
Guru dapat membuat proyek dengan informasi:
Judul project.
Target kelas: semua kelas atau beberapa kelas.
Instruksi pengerjaan.
Batas waktu.
Status project (`aktif` atau `selesai`).
Daftar proyek dapat difilter berdasarkan folder kelas. Proyek yang ditujukan untuk semua kelas akan tetap ditampilkan pada folder kelas terkait.
Guru dapat menutup proyek aktif dan membuka kembali proyek yang sudah selesai.
4. Galeri Karya
Guru dapat melihat submission dan memfilter karya berdasarkan kelas.
Fitur galeri yang terlihat pada kode:
Tampilan tabel submission.
Tampilan grid untuk karya bergambar.
Pratinjau karya dalam layar penuh.
Navigasi karya sebelumnya dan berikutnya pada mode grid.
Zoom in, zoom out, reset zoom, dan geser gambar ketika diperbesar.
Membuka file asli melalui tautan file.
Mengunduh file `.sb3` untuk kebutuhan proyek Scratch.
Melihat daftar siswa yang belum mengumpulkan karya pada folder kelas tertentu.
Pada tabel galeri, guru dapat mengisi:
Nilai — input angka.
Catatan — komentar atau feedback untuk siswa.
Tombol Kirim menyimpan nilai dan catatan ke data submission. Tombol hapus menghapus data submission dari database.
Area Siswa
Pada navigasi utama, kode frontend mendefinisikan menu siswa:
Dashboard.
Upload Karya.
Karya Saya.
Namun, implementasi komponen `StudentDashboard`, `StudentUpload`, dan `StudentGallery` tidak ditemukan pada file `Index.html` yang diberikan. Menu tersebut sudah dipanggil pada proses render aplikasi, tetapi komponen perlu ditambahkan agar area siswa dapat berjalan.
Login siswa yang tersedia meminta siswa memilih kelas, mencari dan memilih nama, lalu memasukkan PIN 4 angka.
Pengelolaan File dan Scratch
Upload file submission
Pada backend `Code.gs`, jika data yang masuk ke koleksi `submissions` memiliki `fileData` berupa Data URL, backend akan:
Membaca MIME type dan payload Base64.
Menentukan ekstensi file.
Membuat Blob dari data Base64.
Mengunggah file ke folder `Galeri_Komputer_Uploads` di Google Drive.
Mengganti `fileData` dengan URL Google Drive.
Menyimpan data submission dalam `DATA_JSON` di Google Sheets.
Ekstensi file yang ditangani backend
Kondisi MIME type	Ekstensi
Spreadsheet / Excel	`.xlsx`
Presentation / PowerPoint	`.pptx`
PDF	`.pdf`
Octet-stream / Scratch	`.sb3`
Lainnya	Subtipe MIME atau `.bin`
Pengolahan gambar di frontend
`Index.html` menyediakan fungsi `compressImage()` yang:
Membaca gambar sebagai Data URL.
Mengubah ukuran maksimum menjadi 800 × 800 piksel dengan mempertahankan rasio.
Menghasilkan gambar JPEG dengan kualitas 0,6.
Fungsi ini tersedia pada frontend. Alur pemanggilannya perlu diperiksa bersama implementasi form upload yang lengkap.
File Scratch `.sb3`
Backend menyediakan fungsi `getFileBase64()` dan `getFileBase64ForDownload()` untuk mengambil byte file Drive dalam bentuk Base64.
Pada galeri guru, fungsi `handleDownloadSb3()` menggunakan `getFileBase64ForDownload()` untuk membentuk kembali file Blob dan memicu pengunduhan dengan ekstensi `.sb3`.
Sinkronisasi Data
Frontend memuat data saat aplikasi pertama kali dibuka melalui `getInitialData()`.
Selain itu:
Tombol Sinkron Data memuat ulang data secara manual.
Aplikasi menjalankan pemuatan ulang data di background setiap 10 detik.
Data lokal pada state React diperbarui setelah operasi tambah, ubah, dan hapus berhasil.
Interval 10 detik dimaksudkan agar perubahan data dari pengguna lain lebih cepat terlihat.
Instalasi dan Deployment
Persiapan
Siapkan:
Akun Google.
Google Spreadsheet untuk database.
Project Google Apps Script yang terikat pada spreadsheet tersebut.
Source code `Code.gs`.
Source code `Index.html`.
Langkah pemasangan
Buat Google Spreadsheet baru.
Dari spreadsheet, buka Extensions → Apps Script agar project terikat pada spreadsheet tersebut.
Masukkan kode backend terbaru ke file `Code.gs`.
Buat file HTML bernama `Index` melalui tombol + → HTML.
Tempel seluruh kode frontend ke file `Index.html`.
Simpan project.
Jalankan `getInitialData()` dari editor Apps Script untuk menginisialisasi database dan membuat data awal. Berikan izin akses Google yang diminta.
Pilih Deploy → New deployment.
Pilih jenis deployment Web app.
Atur siapa yang menjalankan aplikasi dan siapa yang dapat mengaksesnya sesuai kebutuhan.
Klik Deploy, selesaikan otorisasi, lalu buka URL Web App.
`doGet()` memanggil `HtmlService.createHtmlOutputFromFile('Index')`, sehingga nama file HTML di Apps Script harus `Index`.
Memperbarui deployment
Setelah melakukan perubahan kode:
Simpan perubahan pada Apps Script.
Buka Deploy → Manage deployments.
Edit deployment yang digunakan.
Pilih versi terbaru, lalu lakukan deployment ulang.
> **Penting:** gunakan backend `Code.gs` yang menggunakan Google Sheets dan Google Drive. Komponen `GasGuide` di dalam `Index.html` masih menampilkan contoh backend lama yang memakai `PropertiesService` sebagai database. Kode contoh tertanam tersebut tidak sama dengan backend terbaru dan jangan digunakan sebagai pengganti `Code.gs` yang diberikan.
Akun Awal
Backend `Code.gs` membuat data contoh ketika koleksi `users` masih kosong:
Peran	Nama	Username / PIN
Guru	Admin Guru	Username: `admin`
Siswa	Andi Saputra	PIN: `1234`
Kode backend menyimpan password guru dalam bentuk hash SHA-256. Pada mock preview lokal, komentar kode menyebut password contoh admin adalah `admin123`. Gunakan data akun yang tersimpan pada database sebagai acuan dan ubah kredensial contoh sebelum aplikasi digunakan secara nyata.
Data kelas awal:
`Kelas 5A`
Daftar Fungsi Backend
Fungsi	Kegunaan
`doGet()`	Menampilkan halaman `Index` sebagai Web App
`getSpreadsheet()`	Mengambil spreadsheet aktif dan memastikan sheet database tersedia
`getUploadFolder()`	Mengambil atau membuat folder upload Google Drive
`getInitialData()`	Membaca data seluruh koleksi dan membuat data contoh jika `users` kosong
`addDocument(collection, data)`	Menambahkan data baru dan menangani upload file submission
`updateDocument(collection, id, data)`	Menggabungkan properti baru ke data lama berdasarkan ID
`deleteDocument(collection, id)`	Menghapus baris data berdasarkan ID
`getFileBase64(fileUrl)`	Mengambil byte file Drive dalam bentuk Base64
`getFileBase64ForDownload(fileUrl)`	Mengambil byte file Drive dalam bentuk Base64 untuk proses download
Preview Lokal
`Index.html` memiliki mock environment untuk preview lokal ketika objek `google` tidak tersedia.
Mock ini:
Meniru sebagian fungsi `google.script.run`.
Menyimpan data pada browser menggunakan `localStorage`.
Menyediakan mock untuk `getInitialData`, `addDocument`, `updateDocument`, dan `deleteDocument`.
Menambahkan akun admin, kelas contoh, dan siswa contoh jika belum tersedia.
Memberi jeda sekitar 400 ms untuk mensimulasikan komunikasi server.
Batasan preview lokal:
Data tersimpan pada browser/perangkat yang digunakan, bukan pada Google Sheets.
Upload ke Google Drive dan fungsi pengambilan file Base64 tidak disimulasikan oleh mock yang tersedia.
Preview lokal tidak menggantikan pengujian deployment Google Apps Script sebenarnya.
Catatan Teknis dan Keamanan
Akses Google Drive: `getUploadFolder()` mengatur folder upload menjadi `ANYONE_WITH_LINK` dengan izin `VIEW`. Siapa pun yang memiliki tautan dapat melihat file. Pertimbangkan privasi karya dan data siswa sebelum menggunakan pengaturan ini.
Kredensial contoh: ganti password admin dan PIN contoh sebelum penggunaan nyata. Jangan membagikan kredensial admin kepada siswa.
Pemeriksaan akses: proses login pada frontend memeriksa data akun yang dimuat ke browser. Untuk aplikasi yang menyimpan data siswa, pertimbangkan validasi otorisasi pada backend untuk setiap operasi penting.
Validasi input: validasi nama koleksi, ID, ukuran file, MIME type, dan isi data pada backend sebelum menyimpan atau menghapus data.
Hapus submission: `deleteDocument()` menghapus baris data pada Google Sheets, tetapi tidak menghapus file terkait dari Google Drive. File Drive dapat tertinggal setelah submission dihapus.
Penanganan error: beberapa fungsi backend mengembalikan `true` meskipun ID tidak ditemukan. Pertimbangkan mengembalikan status operasi yang lebih akurat.
Kinerja: penggunaan Google Sheets secara langsung dapat menjadi lambat jika data dan jumlah pengguna bertambah. Pertimbangkan optimasi pembacaan/penulisan dan `LockService` untuk mengurangi risiko benturan operasi.
Kuota layanan: perhatikan kuota Google Apps Script, Google Sheets, dan Google Drive, khususnya ketika banyak siswa mengunggah file.
Komponen siswa: lengkapi definisi `StudentDashboard`, `StudentUpload`, dan `StudentGallery` sebelum mengaktifkan penggunaan area siswa.
Panduan deployment tertanam: kode backend di `GasGuide` masih merupakan contoh lama berbasis `PropertiesService`; selalu gunakan file `Code.gs` terbaru yang menjadi backend aplikasi ini.
Lisensi
Lisensi proyek belum ditentukan. Tambahkan file atau keterangan lisensi jika aplikasi akan dibagikan atau dipublikasikan secara terbuka.
---
Galeri Karya Komputer  
Dokumentasi berdasarkan source code `Code.gs` dan `Index.html`.
