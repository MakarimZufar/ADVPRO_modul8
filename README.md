# Advprog-Modul8

**Nama:** Makarim Zufar Prambudyo  
**NPM:** 2306241751  
**Kelas:** Advprog-B  

## Perbedaan Utama antara Unary, Server Streaming, dan Bi-Directional Streaming RPC

Dalam implementasi gRPC, terdapat beberapa pola komunikasi yang memiliki karakteristik berbeda:

-   **Unary RPC**: Merupakan pola komunikasi paling sederhana dimana client mengirimkan satu permintaan dan server memberikan satu respons. Model ini ideal untuk operasi yang tidak kompleks seperti autentikasi pengguna, pengambilan data tunggal, atau pengecekan status sistem. Pada platform Windows, implementasi unary RPC sangat straightforward karena mirip dengan pemanggilan fungsi biasa.

-   **Server Streaming RPC**: Dalam pola ini, client mengirimkan satu permintaan tunggal tetapi server dapat mengembalikan beberapa respons dalam bentuk stream data. Pola ini sangat cocok untuk skenario seperti penampilan riwayat transaksi berkelanjutan, pemantauan resource sistem, atau pengunduhan file berukuran besar secara bertahap. Di lingkungan Windows, penggunaan server streaming memungkinkan aplikasi mengelola memori dengan lebih efisien karena data diproses secara bertahap.

-   **Bi-Directional Streaming RPC**: Merupakan pola paling canggih dimana client dan server dapat saling mengirimkan stream data secara simultan. Model ini sangat efektif untuk aplikasi real-time seperti sistem chat, game multiplayer, atau collaborative editing tools. Pada sistem Windows, bi-directional streaming dapat dimanfaatkan untuk aplikasi yang membutuhkan komunikasi interaktif tanpa harus membuka koneksi baru setiap kali terjadi pertukaran data.

## Pertimbangan Keamanan dalam Implementasi gRPC pada Rust

Saat mengimplementasikan layanan gRPC menggunakan Rust, terdapat beberapa aspek keamanan yang perlu dipertimbangkan:

-   **Autentikasi**: Implementasi token-based authentication seperti JWT merupakan pendekatan yang solid untuk memverifikasi identitas client. Rust memiliki ekosistem crate seperti `jsonwebtoken` yang dapat diintegrasikan dengan gRPC untuk memeriksa validitas token sebelum request diproses lebih lanjut.

-   **Otorisasi**: Penerapan Role-Based Access Control (RBAC) memungkinkan pengaturan hak akses yang granular. Dalam lingkungan Windows, integrasi dengan Active Directory bisa menjadi pilihan untuk mengelola role dan permission dalam ekosistem yang lebih besar.

-   **Enkripsi Data**: gRPC mendukung TLS (Transport Layer Security) secara native, yang menjamin enkripsi end-to-end selama pertukaran data. Untuk deployment di Windows, konfigurasi sertifikat TLS dapat diintegrasikan dengan Windows Certificate Store untuk manajemen sertifikat yang lebih terpusat.

-   **Validasi Input**: Menggunakan fitur type safety dari Rust dan Protocol Buffers untuk melakukan validasi input akan mencegah serangan seperti injection atau buffer overflow. Hal ini sangat penting terutama ketika aplikasi dijalankan pada lingkungan Windows yang sering menjadi target serangan.

## Tantangan dalam Penanganan Bi-Directional Streaming pada Rust gRPC

Implementasi bi-directional streaming di Rust memiliki kompleksitas tersendiri, terutama untuk aplikasi seperti chat:

-   **Manajemen Konkurensi**: Rust menggunakan model ownership yang ketat, sehingga mengelola dua alur komunikasi asynchronous secara simultan membutuhkan pemahaman mendalam tentang lifetime dan borrowing. Pada Windows, penggunaan thread pool yang efisien menjadi krusial untuk performa yang optimal.

-   **Penanganan Error**: Ketika mengimplementasikan streaming dua arah, risiko deadlock atau message loss meningkat bila salah satu channel tertutup secara tidak terduga. Sistem Windows memiliki karakteristik networking yang spesifik yang perlu dipertimbangkan dalam strategi error handling.

-   **State Management**: Mengelola state dari multiple client secara bersamaan membutuhkan pendekatan thread-safe. Rust menawarkan primitif sinkronisasi seperti Mutex dan Arc yang dapat digunakan untuk mencegah race condition.

-   **Resource Management**: Pada Windows, pengelolaan file descriptor dan socket connection perlu mendapat perhatian khusus untuk mencegah resource leak, terutama pada aplikasi yang harus berjalan dalam waktu lama.

## Kelebihan dan Kekurangan tokio_stream::wrappers::ReceiverStream

Penggunaan ReceiverStream dalam layanan gRPC Rust memiliki trade-off tertentu:

-   **Kelebihan**:

    -   Menyederhanakan konversi channel Tokio menjadi stream yang kompatibel dengan gRPC
    -   Meningkatkan keterbacaan kode dengan abstraksi yang lebih tinggi
    -   Memudahkan implementasi pattern producer-consumer untuk streaming data
    -   Pada Windows, integrasi dengan I/O completion ports menjadi lebih seamless

-   **Kekurangan**:
    -   Performa dapat terdegradasi jika volume data sangat besar
    -   Manajemen buffer channel yang tidak optimal dapat menyebabkan bottleneck
    -   Risiko message loss jika receiver tidak mampu mengkonsumsi data secepat producer mengirimkannya
    -   Pada sistem Windows dengan banyak aplikasi berjalan, alokasi memory untuk buffer perlu dipertimbangkan dengan matang

## Struktur Kode Rust gRPC untuk Reusability dan Modularitas

Untuk mencapai kode yang maintainable dan extensible, struktur proyek gRPC di Rust sebaiknya:

-   **Pemisahan Berdasarkan Domain**: Memisahkan logic bisnis ke dalam modul-modul terpisah berdasarkan domain (payment, transaction, user management)
-   **Trait-Based Interface**: Menggunakan trait untuk mendefinisikan kontrak service yang memungkinkan implementasi alternatif dan mempermudah unit testing
-   **Dependency Injection**: Menerapkan pattern DI untuk mengurangi coupling antar komponen
-   **Error Handling Terpusat**: Membuat sistem error handling yang konsisten untuk seluruh aplikasi
-   **Abstraksi Database**: Memisahkan logic akses data dari implementasi service
-   **Configurability**: Memanfaatkan fitur environment variable dari Windows untuk konfigurasi yang fleksibel

Dengan menerapkan prinsip SOLID dan DRY, codebase menjadi lebih modular dan mudah dikembangkan oleh tim.

## Langkah Tambahan untuk Logic Pembayaran yang Kompleks

Untuk menangani skenario pembayaran yang lebih kompleks dalam MyPaymentService, diperlukan:

-   **Validasi Input Komprehensif**: Memeriksa validitas amount, currency, dan parameter lain secara menyeluruh
-   **Integrasi dengan Payment Gateway**: Menghubungkan dengan gateway pembayaran eksternal dengan timeout dan circuit breaker
-   **Transaction Management**: Implementasi mekanisme ACID untuk menjaga integritas data
-   **Logging dan Auditing**: Pencatatan detail transaksi untuk keperluan audit dan compliance
-   **Retry Mechanism**: Sistem retry otomatis untuk transaksi yang gagal dengan exponential backoff
-   **Notifikasi**: Integrasi dengan sistem notifikasi untuk memberi tahu pengguna tentang status pembayaran

Di lingkungan Windows, penggunaan Windows Task Scheduler dapat dimanfaatkan untuk menjadwalkan proses reconciliation secara berkala.

## Dampak Adopsi gRPC terhadap Arsitektur Sistem Terdistribusi

Penggunaan gRPC sebagai protokol komunikasi memberikan pengaruh signifikan pada arsitektur:

-   **Standardisasi Kontrak**: Penggunaan .proto sebagai schema definition language membuat kontrak antar layanan lebih formal dan konsisten
-   **Cross-Language Interoperability**: Layanan yang dibangun dengan bahasa berbeda dapat berkomunikasi tanpa hambatan
-   **Performa Tinggi**: Berkat HTTP/2 dan Protocol Buffers, komunikasi menjadi lebih efisien dibandingkan REST
-   **Streaming Native**: Kemampuan streaming bawaan memungkinkan implementasi use case real-time yang sulit dicapai dengan REST

Pada infrastruktur Windows, integrasi gRPC dengan aplikasi .NET dan layanan Microsoft lainnya menjadi lebih seamless berkat dukungan resmi dari Microsoft.

## Perbandingan HTTP/2 dengan HTTP/1.1 dan WebSocket

HTTP/2 yang menjadi basis gRPC memiliki beberapa keunggulan dibanding alternatifnya:

-   **Multiplexing**: Dapat menangani multiple request-response pada satu koneksi, menghilangkan head-of-line blocking
-   **Header Compression**: Mengurangi overhead jaringan dengan kompresi header
-   **Server Push**: Memungkinkan server mengirim data ke client sebelum diminta
-   **Binary Protocol**: Format binary lebih efisien dibanding text-based HTTP/1.1

Namun, terdapat trade-off berupa:

-   Kompleksitas setup dan debugging yang lebih tinggi
-   Dukungan infrastruktur yang belum merata
-   Kurva pembelajaran yang lebih curam untuk developer

Di lingkungan Windows, HTTP/2 mendapatkan dukungan penuh mulai dari Windows 10, yang membuat implementasi gRPC menjadi lebih straightforward.

## Perbedaan Model REST dan Bidirectional Streaming gRPC

Model request-response REST berbeda signifikan dengan kemampuan streaming gRPC:

-   **REST API**: Menggunakan pola request-response satu lawan satu, client harus polling untuk update. Cocok untuk operasi CRUD sederhana dan interaksi stateless.
-   **gRPC Streaming**: Memungkinkan komunikasi dua arah berkelanjutan tanpa perlu membuka koneksi baru. Ideal untuk aplikasi yang membutuhkan data real-time seperti dashboard monitoring atau collaborative tools.

Dari segi responsiveness, gRPC memberikan latensi lebih rendah karena:

-   Tidak perlu handshaking berulang
-   Format data binary yang lebih ringan
-   Multiplexing yang menghindari blocking

Pada sistem Windows, manfaat performa ini sangat terasa terutama untuk aplikasi desktop yang membutuhkan interaksi server yang responsif.

## Implikasi Pendekatan Schema-Based gRPC vs Schema-less JSON

Penggunaan Protocol Buffers dalam gRPC dan JSON dalam REST memiliki implikasi berbeda:

-   **Protocol Buffers (gRPC)**:

    -   **Kelebihan**: Type safety, ukuran payload yang lebih kecil, performa serialisasi/deserialisasi yang lebih baik
    -   **Kekurangan**: Kaku dan memerlukan redeployment untuk perubahan schema

-   **JSON (REST)**:
    -   **Kelebihan**: Fleksibel, mudah dibaca manusia, tidak memerlukan kompilasi khusus
    -   **Kekurangan**: Overhead ukuran payload, tidak ada type checking, potensi error runtime lebih tinggi

Dalam ekosistem Windows, tooling untuk Protocol Buffers tersedia dengan baik melalui VSCode extensions dan integrasi dengan Visual Studio, memudahkan developer dalam mengadopsi pendekatan schema-based.
