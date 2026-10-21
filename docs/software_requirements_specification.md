# Software Requirements Specification (SRS)
## Subsistem Rekonsiliasi dan Settlement — Baitulmal Tazkia

| Informasi Proyek | Detail |
| :--- | :--- |
| **Nama Proyek / Produk** | Subsistem Rekonsiliasi dan Settlement Baitulmal Tazkia |
| **Kelompok** | Kelompok 5 — Klaster Keuangan |
| **Program Studi** | Sistem Informasi, STMIK Tazkia — Gasal 2026/2027 |
| **Versi & Tanggal** | Versi 1.0 (draf) \| 21 Oktober 2026 |
| **Penulis / Tim** | Kelompok 5 — Klaster Keuangan |
| **Standar Format** | ISO/IEC/IEEE 29148:2018 |

---

## 1. Pendahuluan (Introduction)

### 1.1 Tujuan (Purpose)
Dokumen *Software Requirements Specification* (SRS) ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk Subsistem Rekonsiliasi dan Settlement Baitulmal Tazkia. Subsistem ini membandingkan catatan donasi dengan mutasi rekening bank dan laporan penyelesaian (*settlement*) penyedia pembayaran, sehingga setiap donasi yang tercatat dapat dipastikan benar-benar diterima di rekening.

Dokumen ini disusun berdasarkan hasil elisitasi kebutuhan (wawancara, observasi, dan survei) bersama pemangku kepentingan Baitulmal Tazkia dari domain keuangan dan akuntansi. Setiap kebutuhan di dalamnya harus dapat ditelusuri ke catatan wawancara atau hasil survei melalui *Requirements Traceability Matrix* (RTM).

**Audiens dokumen ini:**

| Audiens | Kegunaan Dokumen |
| :--- | :--- |
| **Staf keuangan & Pemangku Kepentingan** | Memastikan kebutuhan tertulis dengan benar, lalu memberi *sign-off*. |
| **Tim Pengembang Kelompok 5** | Acuan utama perancangan, implementasi, dan pengujian. |
| **Dosen Pengampu (MISI.302, 303, 304)** | Review dan penilaian *deliverable* proyek. |
| **Tim Pendamping (Tim 3)** | Peninjauan kontrak API dan arsitektur. |
| **Kelompok Hulu dan Hilir** | Memahami batas tanggung jawab dan kontrak API antarsubsistem. |
| **Pengelola Balaka** | Memahami bentuk data hasil rekonsiliasi yang akan diterima. |

Setelah memperoleh *sign-off*, dokumen ini menjadi *baseline*. Setiap perubahan sesudahnya melalui prosedur *change control*: usulan tertulis, analisis dampak terhadap jadwal dan lingkup, serta persetujuan pemilik *baseline*.

### 1.2 Ruang Lingkup (Scope)
Baitulmal Tazkia menerima dana melalui beberapa kanal, sedangkan uangnya bermuara pada beberapa rekening bank. Saat ini staf membandingkan berkas satu per satu untuk memastikan kecocokannya. Subsistem ini menggantikan pekerjaan manual tersebut dengan cakupan modul sebagai berikut:

| Kode | Modul | Deskripsi |
| :---: | :--- | :--- |
| **A** | Pemasukan Data | Menerima mutasi rekening dan laporan penyelesaian penyedia pembayaran melalui unggahan berkas atau API. Mencegah unggahan ganda dengan menyimpan *hash* berkas beserta rentang tanggalnya. |
| **B** | Pencocokan Otomatis | Mencocokkan catatan donasi, mutasi rekening, dan laporan penyedia secara bertingkat: (1) nomor rujukan donasi, (2) nominal + tanggal dalam rentang toleransi, (3) kemiripan keterangan mutasi. Tingkat yang dipakai disimpan pada setiap hasil. |
| **C** | Manajemen Selisih | Menampilkan daftar selisih beserta alasan dugaannya. Selisih tidak dihapus; statusnya berpindah sampai selesai dan setiap perpindahan mencatat pelakunya. |
| **D** | Pencocokan Manual | Staf keuangan mencocokkan secara manual dengan jejak audit: siapa mencocokkan apa dan kapan. |
| **E** | Penyerahan ke Balaka | Menyerahkan hasil rekonsiliasi bersih dalam bentuk siap dijurnal, tanpa perapian manual. |

**Di luar lingkup subsistem ini:**

| Hal di Luar Lingkup | Dikerjakan Oleh |
| :--- | :--- |
| Pencatatan jurnal akuntansi | Balaka |
| Pemrosesan pembayaran | Payment Gateway Artivisi |
| Pemisahan dana menurut peruntukan | Kelompok Pengelolaan Dana per Peruntukan |
| Penyajian angka kepada publik | Kelompok Pelaporan Publik dan Transparansi Dana |

### 1.3 Konteks Sistem (Product / System Context)
Subsistem ini adalah bagian dari ekosistem sistem Baitulmal Tazkia yang lebih besar, bukan sistem mandiri. Dua belas kelompok mengerjakan bagian berbeda dari persoalan yang sama, dan setiap kelompok membangun satu subsistem utuh dengan database, aplikasi, dan deployment sendiri. Subsistem ini juga menggantikan proses manual yang berjalan sekarang: pembukuan dengan spreadsheet dan pembandingan berkas satu per satu.

| Arah | Pihak | Peran Terhadap Subsistem Ini |
| :---: | :--- | :--- |
| **Hulu** | Penerimaan Donasi dan Akun Donatur | Menyediakan catatan donasi sebagai sisi pembukuan yang dibandingkan dengan mutasi rekening. |
| **Hulu** | Penerimaan Donasi Luring | Menyediakan data setoran tunai dan transaksi QRIS booth. |
| **Hulu** | Donasi Berulang & Kanal Digital | Menyediakan hasil penagihan berulang, termasuk yang gagal. |
| **Hulu** | Payment Gateway Artivisi | Menyediakan laporan penyelesaian harian. |
| **Hilir** | Pengelolaan Dana per Peruntukan | Mengambil hasil rekonsiliasi bersih sebagai dasar penempatan dana. |
| **Hilir** | Pelaporan Publik & Transparansi | Mengambil angka penerimaan yang sudah terekonsiliasi. |
| **Samping** | Balaka (Pemasok) | Menerima hasil rekonsiliasi yang siap dijurnal. |

### 1.4 Definisi, Akronim, dan Singkatan (Definitions & Acronyms)

| Istilah / Akronim | Definisi |
| :--- | :--- |
| **SRS** | *Software Requirements Specification*, dokumen spesifikasi kebutuhan perangkat lunak. |
| **API** | *Application Programming Interface*, antarmuka komunikasi antar-perangkat lunak. |
| **REST / OpenAPI** | Gaya arsitektur API berbasis HTTP dan format standar dokumen kontraknya. |
| **Rekonsiliasi** | Proses membandingkan catatan donasi dengan mutasi rekening dan laporan penyedia pembayaran. |
| **Settlement** | Proses penyedia pembayaran meneruskan dana transaksi ke rekening penerima. |
| **Selisih (Discrepancy)** | Perbedaan antara catatan donasi dan mutasi/laporan penyedia yang belum berpasangan. |
| **Pencocokan Bertingkat** | Pencocokan hierarkis: nomor rujukan, nominal + tanggal, lalu kemiripan keterangan. |
| **Jejak Audit (Audit Trail)** | Catatan permanen (*who, what, when*) pada setiap pencocokan manual dan perubahan status selisih. |
| **Hash Berkas** | Sidik digital isi berkas untuk mendeteksi pengunggahan ganda. |
| **Balaka** | Sistem pembukuan yang menerima hasil rekonsiliasi dan melakukan pencatatan jurnal akuntansi. |
| **UU PDP** | Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi. |

### 1.5 Dokumen Referensi (References)
1. **ISO/IEC/IEEE 29148:2018** — *Systems and software engineering — Life cycle processes — Requirements engineering*.
2. **Workbook Proyek:** Rekonsiliasi dan Settlement, Kelompok 5 — Baitulmal Tazkia, STMIK Tazkia, Gasal 2026/2027.
3. Piagam Proyek dan Stakeholder Register Kelompok 5.
4. **Undang-Undang Nomor 27 Tahun 2022** tentang Pelindungan Data Pribadi.
5. **CMMI for Development, Version 1.3** — CMU/SEI-2010-TR-033.

---

## 2. Deskripsi Umum (Overall Description)

### 2.1 Perspektif Produk (Product Perspective)
Perangkat lunak Subsistem Rekonsiliasi dan Settlement beroperasi sebagai subsistem mandiri berbasis web yang berada di dalam ekosistem platform keuangan Baitulmal Tazkia. Sistem ini menerima masukan dari hulu dan mendistribusikan data hasil rekonsiliasi ke subsistem hilir menggunakan interaksi antarmuka berikut:
* **Database Server:** PostgreSQL untuk menyimpan riwayat mutasi, aturan pencocokan, data selisih, serta jejak audit.
* **Integrasi Hulu:** Payment Gateway Artivisi, Sistem Penerimaan Donasi Online, Donasi Luring (QRIS/Booth), dan Donasi Berulang.
* **Integrasi Hilir:** Balaka (Sistem Akuntansi), Pengelolaan Dana per Peruntukan, dan Pelaporan Publik & Transparansi Dana.

### 2.2 Fungsi Produk (Product Functions)
* **Pengelolaan & Ingestion Mutasi:** Mengimpor data mutasi rekening bank dan laporan *settlement* via unggahan berkas (CSV/Excel) atau API dengan pencegahan berkas ganda menggunakan *hash* berkas.
* **Pencocokan Otomatis Bertingkat:** Mencocokkan catatan donasi dengan mutasi secara otomatis menggunakan aturan hierarki (nomor rujukan $\rightarrow$ nominal & tanggal $\rightarrow$ kemiripan keterangan).
* **Manajemen Selisih:** Menampilkan daftar selisih/anomali transaksi yang tidak cocok beserta dugaan alasannya.
* **Pencocokan Manual & Jejak Audit:** Memfasilitasi staf keuangan untuk melakukan pencocokan manual pada transaksi selisih, serta mencatat jejak audit (*who, what, when*) secara *immutable*.
* **Penyaluran Data Bersih:** Memformat dan menyerahkan data hasil rekonsiliasi yang valid dan siap dijurnal ke subsistem Balaka via API.

### 2.3 Karakteristik Pengguna (User Characteristics)

| Kategori Pengguna | Peran / Hak Akses | Tingkat Keahlian Teknis |
| :--- | :--- | :---: |
| **Staf Keuangan / Admin** | Mengunggah berkas mutasi, menjalankan pencocokan, memantau daftar selisih, dan melakukan pencocokan manual. | Sedang |
| **Supervisor Keuangan** | Meninjau laporan rekonsiliasi, memberikan otorisasi penyelesaian selisih tanpa pasangan, dan mengonfirmasi penyerahan data. | Sedang |
| **Administrator Sistem** | Mengelola konfigurasi aturan pencocokan (*matching rules*), hak akses pengguna, serta integrasi API. | Tinggi |

### 2.4 Batasan-Batasan (Constraints)
* **Batas Lingkup:** Subsistem ini tidak melakukan pencatatan jurnal akuntansi (ditangani Balaka) dan tidak memproses pembayaran langsung (ditangani Artivisi).
* **Perlindungan Data Pribadi (PDP):** Mematuhi UU No. 27 Tahun 2022 tentang Pelindungan Data Pribadi. Pengujian wajib menggunakan data sintetis sebelum NDA ditandatangani.
* **Aksesibilitas:** Aplikasi berbasis web dan harus responsif pada peranti seluler.
* **Keamanan:** Dilarang menyimpan *hardcoded credentials* di dalam repositori. API wajib menerapkan otorisasi terpusat berbasis peran (RBAC).

### 2.5 Asumsi dan Ketergantungan (Assumptions & Dependencies)
* **Asumsi:** Pengguna terhubung dengan jaringan internet yang stabil. Berkas mutasi rekening bank dapat diperoleh setiap hari.
* **Ketergantungan:** Ketersediaan API hulu (Penerimaan Donasi & Artivisi) serta kepastian spesifikasi format data yang diminta oleh Balaka.

---

## 3. Kebutuhan Spesifik (Specific Requirements)

### 3.1 Kebutuhan Antarmuka Eksternal (External Interface Requirements)
* **3.1.1 Antarmuka Pengguna (User Interfaces):** Antarmuka web responsif (`A-UI-001`), Dasbor Utama status pencocokan & selisih (`A-UI-002`), serta pesan galat validasi formulir yang informatif (`A-UI-003`).
* **3.1.2 Antarmuka Perangkat Keras (Hardware Interfaces):** Beroperasi pada infrastruktur server dan peranti klien standar.
* **3.1.3 Antarmuka Perangkat Lunak (Software Interfaces):** Terhubung ke PostgreSQL via ORM, REST API OpenAPI dengan hulu/Artivisi, serta REST API penyerahan data ke Balaka.
* **3.1.4 Antarmuka Komunikasi (Communication Interfaces):** Seluruh komunikasi data antar-klien dan server wajib menggunakan enkripsi HTTPS / TLS 1.3.

### 3.2 Kebutuhan Fungsional (Functional Requirements)

| Kode ID | Nama Kebutuhan | Deskripsi / Fitur | Prioritas (MoSCoW) |
| :---: | :--- | :--- | :---: |
| **FR-001** | Autentikasi Pengguna | Memverifikasi kredensial login (email & kata sandi) dan menerbitkan token akses. | Must Have |
| **FR-002** | Otorisasi Berbasis Peran | Membatasi akses fitur dan API sesuai matriks peran (Staf, Supervisor, Admin). | Must Have |
| **FR-003** | Ingestion Mutasi & Laporan | Mendukung impor mutasi bank dan laporan *settlement* via berkas (CSV/Excel) atau API. | Must Have |
| **FR-004** | Pencegahan Berkas Ganda | Memvalidasi *hash* berkas dan rentang tanggal untuk mencegah unggahan ganda. | Must Have |
| **FR-005** | Pencocokan Bertingkat | Melakukan pencocokan otomatis berdasarkan rujukan, nominal/tanggal, dan keterangan. | Must Have |
| **FR-006** | Manajemen Selisih | Mencatat transaksi tidak cocok ke daftar selisih beserta alasan dugaannya. | Must Have |
| **FR-007** | Pencocokan Manual & Audit | Menyediakan fitur pencocokan manual dan mencatat *log* audit permanen (*who, what, when*). | Must Have |
| **FR-008** | Penyerahan Data ke Balaka | Menyediakan API penyerahan data hasil rekonsiliasi yang bersih dan siap dijurnal. | Must Have |
| **FR-009** | Ekspor Laporan Rekonsiliasi | Mencetak atau mengekspor laporan hasil pencocokan dalam format PDF dan Excel. | Should Have |

### 3.3 Kebutuhan Non-Fungsional / Kualitas (Quality Requirements)

| Kode ID | Kategori | Spesifikasi / Standar Performa |
| :---: | :--- | :--- |
| **NFR-001** | Keamanan (*Security*) | Kata sandi wajib di-*hash* (`bcrypt`), *secret key* di luar repo, API dilindungi RBAC terpusat. |
| **NFR-002** | Privasi Data (PDP) | Seluruh pengelolaan identitas donatur wajib mematuhi UU No. 27 Tahun 2022 (UU PDP). |
| **NFR-003** | Kinerja (*Performance*) | Pengolahan & pencocokan otomatis 1.000 baris mutasi selesai dalam waktu $t < 5$ detik. |
| **NFR-004** | Keandalan (*Reliability*) | Transaksi pencocokan majemuk wajib ACID (*rollback* penuh jika terjadi gagal proses). |
| **NFR-005** | Integritas Data | Riwayat pencocokan manual dan perubahan status selisih bersifat *immutable* (*no hard delete*). |

### 3.4 Aturan Bisnis dan Kepatuhan (Business Rules)
* **Rule-01 (Toleransi Tanggal & Biaya):** Aturan pencocokan otomatis memperhitungkan jeda waktu (*lag*) masuknya dana dan pemotongan biaya layanan (*fee*) di muka.
* **Rule-02 (Pencegahan Hapus Selisih):** Transaksi selisih tidak boleh dihapus dari basis data; selisih hanya berpindah status.
* **Rule-03 (Penyelesaian Selisih Tunggal):** Penandaan status selesai pada selisih tanpa pasangan wajib menyertakan bukti pendukung dan persetujuan supervisor.

---

## 4. Kriteria Verifikasi (Verification Criteria)

### 4.1 Metode Verifikasi
* **Pengujian Unit (Unit Testing):** Pengujian terisolasi pada aturan bisnis *service layer* menggunakan *mock repository*.
* **Pengujian Integrasi (Integration Testing):** Pengujian *end-to-end* dari HTTP *request* hingga persistensi database untuk *endpoint Must Have*.
* **Pengujian Penerimaan Pengguna (UAT):** Pengujian bersama staf keuangan Baitulmal Tazkia menggunakan skenario *Given-When-Then*.
* **Automated CI/CD Pipeline:** Pemeriksaan otomatis *test suite*, migrasi database, *secret scanning*, dan validasi OpenAPI.

### 4.2 Matriks Kriteria Verifikasi

| ID Req | Kategori | Metode | Kriteria Keberhasilan |
| :---: | :--- | :--- | :--- |
| **FR-001** | Autentikasi | Unit & Integrasi | User berhasil login & dapat token; kredensial salah ditolak (HTTP 401/400). |
| **FR-002** | Otorisasi RBAC | Integrasi & Security | Akses tanpa wewenang ke endpoint API langsung ditolak (HTTP 403). |
| **FR-004** | Anti-Duplikasi | Unit & Integrasi | Berkas dengan *hash*/rentang tanggal sama ditolak sistem dengan peringatan. |
| **FR-005** | Pencocokan | Unit Testing | Akurat 100% pada nomor rujukan cocok; menyimpan level pencocokan yang dipakai. |
| **FR-007** | Audit Trail | Integrasi & UAT | Pencocokan manual tercatat di AuditLog (*who, what, when*) dan *immutable*. |
| **NFR-001** | Keamanan | Secret Scanning | Pipeline CI/CD lulus 100% tanpa *hardcoded credential* / *secret key* di repositori. |

---

## 5. Lampiran (Appendices)

### Lampiran A: Diagram Use Case Utama
Memuat *Use Case Diagram* yang mencakup aktor Staf Keuangan, Supervisor, Admin, dan API Pihak Ketiga dengan Use Case:
* **UC-01:** Unggah Mutasi
* **UC-02:** Pencocokan Otomatis
* **UC-03:** Kelola Selisih
* **UC-04:** Pencocokan Manual
* **UC-05:** Penyerahan Data Balaka
* **UC-06:** Otorisasi Supervisor

### Lampiran B: Model Data / Diagram Entitas Hubungan (ERD)
Memuat struktur entitas utama:
* `MutasiRekening`
* `CatatanDonasi`
* `LaporanSettlement`
* `HasilRekonsiliasi`
* `SelisihTransaksi`
* `AuditLog`

### Lampiran C: Matriks Penelusuran Kebutuhan (RTM)

| ID Req | Nama Kebutuhan | Komponen Perancangan | ID Pengujian |
| :---: | :--- | :--- | :--- |
| **FR-001** | Autentikasi Pengguna | `AuthController`, `AuthService` | `AuthServiceTest`, `AuthIntegrationTest` |
| **FR-004** | Pencegahan Berkas Ganda | `FileIngestionService`, `FileHashUtil` | `FileIngestionTest` |
| **FR-005** | Pencocokan Bertingkat | `ReconciliationService`, `MatchingStrategy` | `ReconciliationServiceTest` |
| **FR-008** | Penyerahan Data Balaka | `BalakaApiClient`, `BalakaExportController` | `BalakaApiIntegrationTest` |

### Lampiran D: Spesifikasi Kontrak API (OpenAPI)
* `POST /api/v1/mutasi/upload` : Unggah berkas mutasi bank.
* `POST /api/v1/rekonsiliasi/jalankan` : Memicu eksekusi pencocokan otomatis.
* `GET /api/v1/selisih` : Mengambil daftar transaksi selisih.
* `POST /api/v1/selisih/{id}/match-manual` : Pencocokan manual oleh staf.
* `GET /api/v1/balaka/export` : Endpoint ekspor data bersih untuk Balaka.

### Lampiran E: Sketsa Antarmuka (Wireframe / Mockup)
Memuat tangkapan layar / mockup antarmuka:
1. Dasbor Utama Rekonsiliasi
2. Halaman Impor Mutasi & Validasi Hash
3. Halaman Pencocokan Manual (*Side-by-Side View*)