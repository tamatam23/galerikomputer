Galeri Karya Komputer
Aplikasi berbasis Google Apps Script untuk mengelola galeri karya komputer siswa. Aplikasi menggunakan Google Sheets sebagai penyimpanan data terstruktur dan Google Drive sebagai tempat penyimpanan berkas karya yang diunggah.
> Dokumentasi ini disusun berdasarkan kode backend `Code.gs` yang tersedia. Detail tampilan dan interaksi pada `Index.html` belum didokumentasikan karena kode frontend belum disertakan.
Daftar Isi
Gambaran Umum
Teknologi
Arsitektur Penyimpanan
Struktur Database
Fitur Backend
Jenis Berkas
Instalasi dan Deployment
Akun dan Data Awal
Fungsi Backend
Catatan Keamanan dan Pengembangan
Gambaran Umum
Galeri Karya Komputer dirancang sebagai tempat pengelolaan data kelas, pengguna, proyek, dan pengumpulan karya siswa. Backend menyediakan fungsi untuk:
Menyiapkan tabel database pada Google Sheets secara otomatis.
Membuat folder penyimpanan karya di Google Drive jika belum tersedia.
Menambahkan, membaca, memperbarui, dan menghapus data.
Menyimpan berkas submission ke Google Drive, bukan menaruh data Base64 berukuran besar di Google Sheets.
Mengambil isi berkas dari Drive dalam bentuk Base64 untuk kebutuhan frontend, termasuk berkas proyek Scratch (`.sb3`).
Teknologi
Teknologi	Kegunaan
Google Apps Script	Backend dan penghubung layanan Google
Google Sheets	Penyimpanan data aplikasi
Google Drive	Penyimpanan berkas karya
HTML Service	Menyajikan halaman aplikasi dari `Index.html`
JavaScript	Logika backend dan pengolahan data
Arsitektur Penyimpanan
Aplikasi menggunakan dua layanan utama:
Google Sheets menyimpan data setiap koleksi dalam format dua kolom: `ID` dan `DATA_JSON`.
Google Drive menyimpan berkas yang diunggah pada koleksi `submissions`. Data di Google Sheets menyimpan URL berkas Drive sebagai pengganti string Base64.
Folder Drive yang digunakan bernama `Galeri_Komputer_Uploads`. ID folder disimpan pada Script Properties dengan nama `DB_FOLDER_ID`, sehingga folder yang sama dapat digunakan kembali pada eksekusi berikutnya.
Struktur Database
Saat `getSpreadsheet()` dijalankan, aplikasi memastikan empat sheet berikut tersedia:
Sheet	Keterangan
`users`	Data akun guru dan siswa
`classes`	Data kelas
`projects`	Data proyek/tugas
`submissions`	Data pengumpulan karya siswa
Setiap sheet menggunakan struktur:
Kolom	Isi
`ID`	Identitas unik data
`DATA_JSON`	Seluruh properti data dalam bentuk JSON
Contoh ilustrasi satu baris data:
ID	DATA_JSON
`user-123`	`{"role":"siswa","name":"Andi","classId":"class-1"}`
Properti di dalam `DATA_JSON` dapat berbeda sesuai jenis koleksinya. Fungsi `getInitialData()` membaca JSON dan menambahkan nilai kolom `ID` sebagai properti `id` pada objek yang dikembalikan.
Fitur Backend
1. Inisialisasi database otomatis
Menggunakan spreadsheet tempat project Apps Script terpasang.
Membuat sheet `users`, `classes`, `projects`, dan `submissions` jika belum ada.
Menambahkan header `ID` dan `DATA_JSON`.
Membekukan baris pertama sebagai header.
Menghapus sheet bawaan `Sheet1` jika masih ada dan terdapat sheet lain.
2. Pembuatan folder Google Drive otomatis
Memeriksa ID folder dari Script Properties (`DB_FOLDER_ID`).
Menggunakan folder yang sudah tersimpan jika masih dapat diakses.
Membuat folder `Galeri_Komputer_Uploads` jika folder belum tersedia atau tidak dapat diakses.
Menyimpan ID folder baru ke Script Properties.
Perhatian: kode saat ini mengatur folder menjadi `ANYONE_WITH_LINK` dengan izin `VIEW`. Artinya, siapa pun yang memiliki tautan dapat melihat berkas di folder tersebut. Pertimbangkan risiko privasi karya dan identitas siswa sebelum menggunakan pengaturan ini.
3. CRUD data
Backend menyediakan operasi dasar:
Create: menambahkan data baru melalui `addDocument()`.
Read: mengambil seluruh data awal melalui `getInitialData()`.
Update: memperbarui properti data melalui `updateDocument()`.
Delete: menghapus baris data melalui `deleteDocument()`.
4. Penyimpanan berkas submission
Jika data yang ditambahkan ke koleksi `submissions` memiliki `fileData` berupa Data URL (`data:...`), backend akan:
Mengambil MIME type dan isi Base64.
Menentukan ekstensi berkas berdasarkan MIME type.
Membuat Blob dari data Base64.
Mengunggah Blob ke folder Google Drive.
Mengganti `fileData` dengan URL Google Drive.
Menyimpan objek submission sebagai JSON di Google Sheets.
Dengan demikian, isi berkas tidak disimpan langsung dalam satu sel Google Sheets.
5. Dukungan berkas Scratch / TurboWarp
Tersedia dua fungsi untuk mengambil berkas dari Google Drive:
`getFileBase64(fileUrl)` — membaca byte berkas dan mengembalikannya sebagai Base64. Dapat digunakan frontend untuk kebutuhan pemuatan berkas.
`getFileBase64ForDownload(fileUrl)` — mengambil byte berkas dan mengembalikannya sebagai Base64 untuk proses pengunduhan.
Kedua fungsi menggunakan ID file yang diekstrak dari parameter `id` pada URL Drive.
Jenis Berkas
Kode backend memiliki penanganan ekstensi khusus berikut:
MIME type / kondisi	Ekstensi yang digunakan
Spreadsheet / Excel	`.xlsx`
Presentation / PowerPoint	`.pptx`
PDF	`.pdf`
Octet-stream / Scratch	`.sb3`
MIME type lainnya	Menggunakan subtipe MIME sebagai ekstensi, atau `.bin` jika kosong
Ekstensi `.sb3` dipaksakan untuk MIME type yang mengandung `octet-stream` atau `scratch`. Pastikan tipe berkas yang dikirim frontend sesuai dengan berkas sebenarnya.
Instalasi dan Deployment
Persiapan
Siapkan:
Akun Google yang memiliki akses ke Google Sheets, Google Drive, dan Google Apps Script.
Satu Google Spreadsheet sebagai database.
File backend `Code.gs`.
File HTML bernama `Index.html` untuk tampilan aplikasi.
Langkah instalasi
Buat Google Spreadsheet baru untuk database aplikasi.
Buka Extensions → Apps Script dari spreadsheet tersebut agar project terikat pada spreadsheet.
Masukkan kode backend ke file `Code.gs`.
Buat file HTML bernama `Index` melalui Apps Script (+ → HTML) dan masukkan kode frontend.
Simpan seluruh perubahan.
Jalankan fungsi `getInitialData()` dari editor Apps Script untuk menginisialisasi sheet dan data awal. Saat pertama kali dijalankan, berikan izin akses yang diminta Google.
Pilih Deploy → New deployment.
Pilih jenis deployment Web app.
Atur Execute as dan Who has access sesuai kebutuhan serta kebijakan privasi sekolah.
Klik Deploy, selesaikan otorisasi, lalu buka URL Web App.
> Nama file HTML harus `Index` pada Apps Script karena `doGet()` memanggil `HtmlService.createHtmlOutputFromFile('Index')`.
Menjalankan ulang / memperbarui aplikasi
Jika kode diperbarui, simpan perubahan lalu kelola deployment melalui Deploy → Manage deployments. Perbarui versi deployment yang digunakan agar perubahan tersedia pada Web App.
Akun dan Data Awal
Saat `getInitialData()` dijalankan dan koleksi `users` masih kosong, backend membuat data awal:
Jenis	ID	Nama	Username / PIN
Guru	`guru-1`	Admin Guru	Username: `admin`
Siswa	`siswa-1`	Andi Saputra	PIN: `1234`
Kelas	`class-1`	Kelas 5A	—
Data akun guru menyimpan nilai password berupa hash SHA-256, bukan password teks biasa. Kode yang tersedia tidak memperlihatkan mekanisme login frontend, sehingga cara verifikasi password dan PIN perlu diperiksa pada `Index.html`.
Penting: akun dan PIN di atas adalah data contoh bawaan. Ganti atau hapus kredensial contoh sebelum aplikasi digunakan secara nyata. Jangan membagikan kredensial admin kepada siswa.
Fungsi Backend
Fungsi	Keterangan
`doGet()`	Menampilkan halaman HTML `Index` sebagai Web App
`getSpreadsheet()`	Mengambil spreadsheet aktif dan memastikan sheet database tersedia
`getUploadFolder()`	Mengambil atau membuat folder upload Google Drive
`getInitialData()`	Membaca seluruh koleksi dan membuat data awal jika `users` kosong
`addDocument(collection, data)`	Menambahkan data baru; mengunggah berkas submission jika berupa Data URL
`updateDocument(collection, id, data)`	Menggabungkan properti baru ke data lama berdasarkan ID
`deleteDocument(collection, id)`	Menghapus baris data berdasarkan ID
`getFileBase64(fileUrl)`	Mengambil isi berkas Drive dalam bentuk Base64
`getFileBase64ForDownload(fileUrl)`	Mengambil isi berkas Drive dalam bentuk Base64 untuk diunduh
Catatan Keamanan dan Pengembangan
Sebelum aplikasi digunakan untuk data siswa sungguhan, perhatikan beberapa hal berikut:
Akses file Drive: folder upload saat ini dibuat dapat diakses siapa pun yang memiliki tautan. Evaluasi apakah karya siswa boleh dibagikan secara publik. Batasi akses jika diperlukan.
Autentikasi: pastikan proses login, penyimpanan password, dan pemeriksaan PIN di frontend/backend aman. Jangan mengandalkan penyembunyian elemen UI sebagai kontrol akses.
Validasi input: validasi nama koleksi, ID, ukuran berkas, MIME type, dan isi data sebelum operasi database atau upload.
Penghapusan berkas: `deleteDocument()` hanya menghapus baris pada Google Sheets. Fungsi ini tidak menghapus file terkait dari Google Drive. Perlu mekanisme tambahan jika file juga harus dihapus.
Pembaruan data: `updateDocument()` mengembalikan `true` meskipun ID tidak ditemukan. Pertimbangkan mengembalikan status yang menunjukkan apakah data benar-benar diperbarui.
Penanganan error: beberapa fungsi menangkap error dan mengembalikan `null` atau mengabaikan JSON yang gagal dibaca. Tambahkan pencatatan error agar masalah lebih mudah ditelusuri.
Kinerja dan konkurensi: operasi baca/tulis menggunakan Google Sheets secara langsung. Untuk penggunaan banyak pengguna, pertimbangkan `LockService`, pembatasan ukuran data, dan optimasi pembacaan/penulisan.
Integritas data: `getInitialData()` membuat seluruh data contoh ketika koleksi `users` kosong. Pastikan kondisi database sudah benar sebelum menjalankannya pada data produksi.
Validasi URL: fungsi pembaca file mengambil ID dari parameter `id`. Pastikan URL berasal dari file yang memang boleh diakses oleh aplikasi.
Lisensi
Belum ditentukan. Tambahkan informasi lisensi jika proyek ini akan dibagikan atau dipublikasikan secara terbuka.
---
Galeri Karya Komputer  
Dokumentasi backend berdasarkan implementasi Google Apps Script.
