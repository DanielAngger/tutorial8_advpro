# Reflection Tutorial 8

<details>
<summary>1. Perbedaan Utama Metode RPC: Unary, Server Streaming, dan Bidirectional Streaming</summary>

### Unary RPC  
>Cara Kerja:   
Klien mengirim satu permintaan, server merespons sekali.  

>Contoh Penggunaan:  
-. Mengambil data statis (e.g., informasi pengguna berdasarkan ID).  
-. Operasi CRUD sederhana (e.g., membuat atau menghapus entri database).  

>Kelebihan:  
Sederhana, cocok untuk operasi sinkron.  

### Server Streaming RPC
>Cara Kerja:  
Klien mengirim satu permintaan, server mengirim serangkaian respons secara streaming.  

>Contoh Penggunaan:  
-. Mengirim pembaruan real-time (e.g., harga saham, notifikasi).
-. Streaming log atau data sensor.  

>Kelebihan:  
Efisien untuk data yang terus berubah.  

### Bidirectional Streaming RPC
>Cara Kerja:  
Klien dan server saling mengirim aliran data secara independen.

>Contoh Penggunaan:  
-. Aplikasi chat (pesan dikirim dan diterima secara real-time).
-. Game multiplayer dengan interaksi simultan.

>Kelebihan:  
Mendukung komunikasi dua arah yang dinamis.
</details>

<details>
<summary>2. Pertimbangan Keamanan untuk gRPC di Rust</summary>

### Autentikasi:
>TLS/SSL:  
Wajib untuk mengenkripsi komunikasi. Gunakan crate seperti rustls atau openssl. 

>Token-Based Auth:  
JWT atau OAuth2 untuk validasi identitas klien.  

>mTLS (Mutual TLS):  
Verifikasi sertifikat di kedua sisi (klien dan server).  

### Autorisasi:
>Implementasi middleware untuk memeriksa izin (e.g., role-based access control).  
Contoh: Gunakan tower-layer untuk memvalidasi hak akses sebelum memproses RPC. 

### Enkripsi Data:  
>Pastikan payload Protobuf dienkripsi end-to-end jika mengandung data sensitif.  

>Hindari penyimpanan rahasia (e.g., kunci API) di kode sumber (gunakan environment variables).  

### Pencegahan Serangan:  
>Validasi input untuk menghindari serangan seperti SQL injection atau buffer overflow.  

>Rate limiting untuk mencegah DDoS.  

### Khusus Rust:
>Manfaatkan sistem kepemilikan (ownership) untuk mencegah race condition.

>Audit dependensi dengan cargo-audit untuk celah keamanan.

</details>

<details>
<summary>3. Tantangan Bidirectional Streaming di Rust (Contoh: Aplikasi Chat)</summary>

>Manajemen Konkurensi:  
-. Async/await di Rust memerlukan pengelolaan Future dan Stream yang hati-hati.  
-. Contoh: Deadlock jika task tidak di-schedule dengan benar di Tokio runtime.  

>Backpressure:  
-. Jika server/client tidak bisa mengimbangi kecepatan data, gunakan mekanisme seperti tokio::sync::mpsc untuk mengontrol aliran.  

>Error Handling:  
-. Jika satu sisi terputus (e.g., jaringan error), pastikan koneksi bisa dipulihkan atau diberi notifikasi.  
-. Implementasi timeout dengan tokio::time::timeout.

>Resource Leaks:  
-. Pastikan stream ditutup (StreamExt::close) setelah selesai untuk menghindari kebocoran memori.

>Sinkronisasi State:  
-. Untuk aplikasi chat, gunakan Arc<Mutex<...>> atau saluran async untuk mengelola daftar pengguna yang terhubung.

</details>

<details>
<summary>4. Kelebihan & Kekurangan ReceiverStream di Rust gRPC</summary>

>Kelebihan:  
-. Integrasi dengan Tokio: Cocok untuk runtime async yang sudah menggunakan Tokio.  
-. Simplifikasi kode: Konversi mudah dari mpsc::Receiver ke stream gRPC.  
-. Efisien untuk data berukuran besar atau throughput tinggi.

>Kekurangan:  
-. Overhead: Tidak cocok untuk skenario low-latency ekstrem.  
-. Ketergantungan pada Tokio: Sulit digunakan dengan runtime async lain (e.g., async-std).   
-. Kurang fleksibel: Jika memerlukan kontrol granular, lebih baik implementasi manual.  

</details>

<details>
<summary>5. Struktur Kode Rust gRPC untuk Reusabilitas</summary>

>Modularisasi:  
-. Pisahkan logika bisnis, protobuf codegen, dan infrastruktur ke modul terpisah (e.g., src/services, src/models).  

>Traits/Interface:  
-. Definisikan trait seperti PaymentProcessor untuk memisahkan implementasi konkret.  

>Middleware:  
-. Gunakan tower-layer untuk logging, auth, atau metrics yang bisa digunakan ulang.  

>Dependency Injection:  
-. Contoh: Teruskan client database atau HTTP client sebagai parameter ke handler.  

>Codegen Terpusat:  
-. Generate kode Protobuf sekali di build script (build.rs).  

</details>

<details>
<summary>6. Implementasi Logika Pembayaran Kompleks di MyPaymentService</summary>

>Idempotensi:  
Gunakan ID unik untuk setiap transaksi untuk menghindari duplikasi.

>Retry & Circuit Breaker:  
Implementasi retry dengan backoff eksponensial (e.g., tokio-retry).  

>Validasi Fraud:  
Integrasi dengan layanan pihak ketiga (e.g., Stripe Radar) untuk mendeteksi transaksi mencurigakan.  

>Distributed Transactions:  
Gunakan pola Saga untuk konsistensi data antar mikroservis.  

>Antrean Async:  
Proses pembayaran dengan tokio::task::spawn_blocking atau background worker.  

>Audit Log:  
Simpan riwayat transaksi di database terpisah untuk kepatuhan regulasi.  

</details>

<details>
<summary>7. Dampak gRPC pada Arsitektur Sistem Terdistribusi</summary>

>Interoperabilitas:  
gRPC sulit diintegrasikan dengan sistem legacy yang menggunakan REST (solusi: gunakan gRPC Gateway untuk translate HTTP/JSON ke gRPC).  

>Performance:  
HTTP/2 dan binary Protobuf mengurangi latency, cocok untuk sistem high-throughput.  

>Coupling:  
Skema Protobuf yang terpusat memaksa tim untuk sinkronisasi API, mengurangi risiko breaking changes.  

>Tooling:  
Generate kode otomatis untuk client/server di berbagai bahasa (Go, Python, dll).  

</details>

<details>
<summary>8. Perbandingan HTTP/2 (gRPC) vs HTTP/1.1/WebSocket</summary>

| Kriteria            | HTTP/2 (gRPC)     | HTTP/1.1 + WebSocket                             |
|------------------------|-----------|----------------------------------------------|
| Multiplexing | Dukungan native untuk banyak stream | Hanya 1 koneksi per request|
| Kompresi Header | HPACK mengurangi ukuran header | Tidak ada kompresi bawaan |
| Streaming | Bawaan di Protobuf/gRPC | Membutuhkan WebSocket |
| Browser Support | Terbatas (butuh gRPC-Web) | Didukung luas |
| Kompleksitas | Lebih sulit di-debug | Sederhana untuk REST tradisional |  
</details>

<details>
<summary>9. REST vs gRPC untuk Komunikasi Real-Time</summary>

>REST:  
Model Request-Response: Klien harus polling atau menggunakan WebSocket untuk real-time.  
Contoh: Aplikasi dengan update setiap 5 detik (e.g., cuaca).  

>gRPC:  
Bidirectional Streaming: Server bisa push data segera setelah tersedia.  
Contoh: Live score olahraga, chat, atau sistem trading.  

</details>

<details>
<summary>10. Protobuf (Skema Ketat) vs JSON (Schema-less)</summary>

### Protobuf:  
>Kelebihan:  
-. Ukuran payload lebih kecil (binary encoding).  
-. Validasi otomatis melalui skema.  
-. Kompatibilitas versi (fields optional/deprecated).

>Kekurangan:  
-. Tidak bisa dibaca manusia (butuh tools seperti protoc).
-. Perlu kompilasi skema ke kode.  

### JSON:  
>Kelebihan:  
-. Fleksibel (tidak perlu skema tetap).  
-. Mudah di-debug (bisa dibaca langsung).  

>Kekurangan:  
-. Rentan ke inkonsistensi (e.g., typo di field name).  
-. Ukuran payload lebih besar.  

</details>