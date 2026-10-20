# Laporan Riset Modul 1: IMCLAB Basic Arduino

**Topik:** Pengendalian LED dan Motor DC Berbasis Komunikasi Serial pada ESP32  
**Sumber Pengamatan:** Dokumentasi eksperimen bertanggal 17 Oktober 2026 dan kode program utama

---

## 1. Ringkasan Riset

Pengujian ini bertujuan mengamati keterkaitan antara perintah berbasis komunikasi serial dari komputer, respons keluaran digital ESP32, serta pengaturan daya pada penggerak motor kit iMCLab. Sistem dirancang untuk membaca masukan karakter '1' dan '0'. Ketika sinyal '1' dikirim, LED indikator akan menyala bersamaan dengan pengaktifan sinyal PWM ke modul penggerak motor. Sebaliknya, transmisi sinyal '0' akan memadamkan LED dan menghentikan pengiriman PWM ke motor.

Melalui dokumentasi visual yang ada, terlihat perangkat terhubung secara fisik menggunakan antarmuka USB, dengan dua kondisi indikator LED biru yang berbeda. Bukti ini mengonfirmasi adanya perubahan status logika pada LED board. Sementara itu, mekanisme kerja keluaran motor dianalisis berdasarkan struktur kode program, mengingat foto statis belum dapat mengukur tingkat kecepatan putaran secara presisi.

---

## 2. Tujuan dan Pertanyaan Riset

Tujuan utama dari eksperimen ini adalah memverifikasi penerimaan instruksi serial, konfigurasi pin digital keluaran, serta penerapan sinyal PWM untuk mengatur penggerak motor DC.

Pertanyaan inti yang dianalisis: **bagaimana logika program memproses perintah '1' dan '0' menjadi aksi nyata pada LED indikator serta penggerak motor?**

---

## 3. Dasar Kerja Sistem

Skema alur transmisi dan pemrosesan sinyal pada sistem ini ditunjukkan sebagai berikut:


```

Komputer / Serial Monitor
| karakter '1' atau '0' melalui USB Serial
v
ESP32
|-- GPIO 2 ----------> LED Indikator (Onboard)
|-- GPIO 27 dan 26 --> Arah Penggerak Motor
`-- GPIO 12 ---------> Sinyal PWM (Enable Motor)

```

Konfigurasi PWM pada program menggunakan frekuensi dasar 30 kHz dengan tingkat resolusi 8 bit. Untuk merepresentasikannya dalam nilai persentase, rumusan duty cycle dihitung melalui persamaan:

$$\text{Duty Cycle (\%)} = \frac{\text{Nilai PWM}}{255} \times 100\%$$

Perhitungan persentase ini merupakan cerminan dari besaran sinyal kendali pada skrip, bukan hasil pengukuran langsung terhadap kecepatan mekanis motor.

---

## 4. Alat, Bahan, dan Konfigurasi

| Komponen / Parameter | Konfigurasi Sistem |
| :--- | :--- |
| **Mikrokontroler** | ESP32 pada kit modul iMCLab |
| **Aktuator** | Motor DC beserta rangkaian *driver* penggerak |
| **Antarmuka Sistem** | Arduino IDE / Serial Monitor dan kabel USB |
| **Pin LED Indikator** | GPIO 2 (`ledPin`) |
| **Pin Arah Motor** | GPIO 27 (`motor1Pin1`) dan GPIO 26 (`motor1Pin2`) |
| **Pin Enable / PWM** | GPIO 12 (`enable1Pin`) |
| **Sensor Encoder / RPM** | GPIO 10 (`pin_rpm`, mode `INPUT_PULLUP`) |
| **Kecepatan Serial** | 115.200 bps |
| **Modulasi PWM** | Frekuensi 30 kHz, resolusi 8-bit (skala 0–255) |
| **Kondisi `MotorOn()`** | Duty cycle = 250 (mendekati 98,0%) |
| **Kondisi `MotorOff()`** | Duty cycle = 0 (0%) |

> **Catatan:** Detail teknis mengenai tipe IC penggerak, spesifikasi daya riil, dan model motor DC tidak dicantumkan secara tertulis pada berkas referensi, sehingga tidak dimasukkan ke dalam variabel terukur.

---

## 5. Metode Eksperimen

Langkah-langkah pengujian disusun mengacu pada logika program dan susunan alat yang terdokumentasi:

1. Menghubungkan board ESP32 pada kit iMCLab ke komputer melalui antarmuka kabel USB.
2. Membuka berkas kode program pada Arduino IDE, memastikan pengaturan board dan port COM sudah sesuai, lalu melakukan proses unggah (*upload*).
3. Mengaktifkan fitur Serial Monitor dengan penyesuaian *baud rate* pada angka 115.200.
4. Mengirimkan karakter `'1'` melalui jendela Serial Monitor, kemudian mengamati perubahan pada LED biru dan respons penggerak motor.
5. Mengirimkan karakter `'0'`, dilanjutkan dengan mengamati perubahan indikator LED serta penghentian daya motor.
6. Mengambil dokumentasi visual pada board sebagai materi analisis hasil.

Variabel bebas pada pengujian ini berupa input karakter serial, sedangkan variabel terikatnya adalah status logika LED serta tingkat *duty cycle* PWM.

---

## 6. Hasil dan Dokumentasi

### 6.1 Kondisi LED Biru Menyala

![Perangkat Kondisi ON](./imclab%20basic%20on.jpeg)  
*Gambar 1. Tampilan fisik modul iMCLab saat LED biru pada board ESP32 aktif menyala.*

**Analisis Gambar:** Modul ESP32 terpasang stabil pada kit dan terhubung via kabel USB. LED indikator berwarna biru terlihat menyala terang berdampingan dengan indikator merah dan hijau pada papan rangkaian. Tampilan ini selaras dengan eksekusi perintah `HIGH` pada `ledPin` saat karakter `'1'` diterima.

### 6.2 Kondisi LED Biru Padam

![Perangkat Kondisi OFF](./imclab%20basic%20off.jpeg)  
*Gambar 2. Tampilan fisik modul iMCLab saat LED biru dalam kondisi padam.*

**Analisis Gambar:** Indikator daya berwarna merah dan hijau tetap aktif menyala, sedangkan LED biru pada board ESP32 terlihat padam. Perbandingan visual antara Gambar 1 dan Gambar 2 memperlihatkan adanya transisi status logika LED tanpa memutus suplai daya utama. Kondisi ini sesuai dengan logika `LOW` pada penanganan instruksi `'0'`.

### 6.3 Tabel Pemetaan Logika Program

| Karakter Input | Status LED (GPIO 2) | Nilai PWM (GPIO 12) | Arah Motor (GPIO 27, 26) | Output Serial Monitor |
| :---: | :---: | :---: | :---: | :---: |
| `'1'` | `HIGH` (Menyala) | 250 | `HIGH`, `LOW` | `LED is ON !!!` |
| `'0'` | `LOW` (Padam) | 0 | `HIGH`, `LOW` | `LED is OFF !!!` |

*Tabel di atas dipetakan langsung dari logika kode. Variabel `dutyCycle` mulanya bernilai 200, namun nilainya diperbarui menjadi 250 saat fungsi `MotorOn()` dipanggil.*

---

## 7. Pembahasan

Hasil pengujian membuktikan bahwa ESP32 mampu menerjemahkan komunikasi data serial menjadi sinyal keluaran digital dan PWM. Pada skrip ini, pengendalian arah rotasi motor dibuat tetap (searah), karena nilai logika GPIO 27 diatur konstan `HIGH` dan GPIO 26 diatur `LOW` pada kedua fungsi (`MotorOn()` dan `MotorOff()`). Dengan demikian, proses penghentian motor dilakukan dengan mematikan sinyal *enable* PWM (mengubah *duty cycle* menjadi 0).

Integrasi aksi antara LED dan motor terjadi karena pemanggilan fungsinya berada di dalam blok percabangan `if-else` yang sama. LED yang aktif menandakan bahwa program telah mengeksekusi perintah keluaran secara perangkat lunak, meskipun potensi kendala fisik pada catu daya atau beban mekanis tetap dapat memengaruhi putaran motor secara aktual.

Pada aspek pengukuran kecepatan, program telah mendeklarasikan `pin_rpm` (GPIO 10) dengan mode `INPUT_PULLUP` dan variabel `rev`. Namun, kode tersebut belum dilengkapi algoritma perhitungan waktu maupun fungsi interupsi (*interrupt*). Akibatnya, nilai kecepatan dalam satuan RPM belum dapat dikalkulasi maupun ditampilkan pada pengujian ini.

---

## 8. Keterbatasan dan Saran Pengembangan

- **Keterbatasan Pengujian:**
  - Belum ada perekaman langsung terhadap tampilan baris teks di Serial Monitor.
  - Besaran frekuensi dan *duty cycle* PWM baru sebatas konfigurasi di tingkat kode dan belum diuji menggunakan osiloskop.
  - Parameter mekanis seperti RPM, konsumsi arus, dan torsi motor belum diukur secara kuantitatif.

- **Saran Pengembangan:**
  - Menambahkan fungsi *Interrupt Service Routine* (ISR) pada `pin_rpm` untuk menghitung pulsa encoder dan mengkalkulasi kecepatan RPM secara *real-time*.
  - Mengembangkan antarmuka perintah serial agar dapat menerima input angka dinamis (0–255) guna mengatur kecepatan motor secara bertahap.
  - Menambahkan fitur kontrol arah putaran motor (CW/CCW) dengan membalik logika logika pada GPIO 27 dan GPIO 26.

---

## 9. Kesimpulan

Eksperimen Modul 1 ini berhasil mengimplementasikan sistem kendali sederhana untuk LED indikator dan keluaran motor DC berbasis komunikasi serial. Bukti visual mengonfirmasi perubahan status LED yang sinkron dengan instruksi `'1'` dan `'0'`. Pengujian ini masih berfokus pada pengujian fungsional dasar, sedangkan fitur pembacaan kecepatan rotasi (RPM) memerlukan pengembangan algoritma lebih lanjut.

---

## 10. Sumber Data

- Berkas program utama `basic_arduino.ino`
- Dokumentasi visual pengujian perangkat

```
