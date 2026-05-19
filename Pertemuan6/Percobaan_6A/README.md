# Jawaban Pertanyaan Praktikum Modul 6: Percobaan 6A (External Interrupt)

**Nama:** Muhammad Aziz Ihza Fahriza Salam  
**NIM:** H1H024050  
**Mata Kuliah:** TK244005-Praktikum Sistem Mikrokontroller

---

## 1. Jelaskan proses bagaimana tombol dapat mengubah kondisi LED menggunakan interrupt!

Proses ini berjalan berdasarkan deteksi perubahan sinyal fisik oleh perangkat keras mikrokontroler. Saat tombol ditekan, terjadi perubahan tegangan pada pin 2 Arduino. Mikrokontroler yang mendeteksi perubahan ini akan secara otomatis memberikan "interupsi" dengan menjeda sementara eksekusi program utama yang sedang berjalan di dalam fungsi `loop()`. Alur program kemudian langsung melompat ke fungsi khusus yakni ISR `(tombolInterrupt()`). Di dalam fungsi ini, mikrokontroler membalikkan nilai logika variabel `ledState` (dari ON ke OFF atau sebaliknya). Segera setelah instruksi di dalam ISR selesai, program kembali melanjutkan tugasnya di fungsi `loop()` dan memperbarui status nyala/mati LED menggunakan perintah `digitalWrite()` berdasarkan nilai `ledState` yang baru

---

## 2. Apa fungsi attachInterrupt() pada program tersebut?

Fungsi `attachInterrupt()` bertugas sebagai konfigurator yang memetakan pin hardware ke software (fungsi ISR). Pada program ini, fungsi tersebut memberitahu mikrokontroler: "Pantau terus pin 2, dan apabila terjadi transisi tegangan dari tinggi ke rendah (FALLING), segera eksekusi fungsi `tombolInterrupt`." Tanpa fungsi ini, pin 2 hanyalah pin input biasa dan tidak akan bisa memicu interupsi.

---

## 3. Mengapa pada ISR tidak disarankan menggunakan delay() dan Serial.print()?

Sifat dasar dari ISR adalah menyela program utama, sehingga ia harus dikerjakan dengan sangat cepat agar sistem kembali berjalan normal. Menggunakan `delay()` akan menghentikan seluruh detak (clock) pemrosesan, sehingga membuat mikrokontroler hang di dalam ISR. Sedangkan `Serial.print()` merupakan operasi komunikasi data yang memakan waktu cukup lama bagi skala waktu mikrokontroler. Jika ISR berjalan terlalu lama, mikrokontroler akan mengabaikan interupsi-interupsi lain yang masuk (karena saat ISR bekerja, interupsi lain biasanya dinonaktifkan sementara) dan membuat sistem tidak responsif.

---

## 4. Apa fungsi keyword volatile pada variabel ledState?

Keyword `volatile` adalah instruksi wajib bagi compiler agar tidak melakukan optimasi berlebihan terhadap variabel tersebut. Dalam operasi normal, compiler sering menyimpan variabel di dalam register (memori sementara yang cepat) alih-alih di RAM. Namun, karena nilai `ledState` diubah di dalam ISR (yang merupakan event tak terduga dan berjalan di luar alur `loop()`), penambahan `volatile` memaksa mikrokontroler untuk selalu mengambil nilai variabel tersebut langsung dari alamat RAM utamanya. Ini menjamin program utama selalu membaca status `ledState` yang paling mutakhir.

---

## 5. Modifikasi Program Menggunakan Mode Interrupt Lain

Terdapat beberapa mode pemicu interrupt yang tersedia, yaitu `RISING`, `FALLING`, `CHANGE`, dan `LOW`. Perbedaan Cara Kerja Mode Interrupt:

* **FALLING:** Interupsi terpicu seketika saat sinyal bergeser dari logika tinggi (HIGH) ke rendah (LOW). Umumnya terjadi persis saat tombol dengan rangkaian pull-up mulai ditekan ke bawah.
* **RISING:** Interupsi terpicu saat sinyal bergeser dari logika rendah (LOW) ke tinggi (HIGH). Terjadi persis di momen jari mengangkat atau melepaskan tekanan pada tombol.
* **CHANGE:** Interupsi akan aktif pada dua kondisi sekaligus: saat tombol ditekan (HIGH ke LOW) maupun saat tombol dilepas (LOW ke HIGH).
* **LOW:** Interupsi dieksekusi terus-menerus tanpa henti selama logika sinyal berada pada titik terendah (tombol sedang ditahan).

### Analisis Perilaku LED pada Mode RISING

Jika kita menggunakan mode RISING, LED tidak akan menyala atau mati pada saat kita menekan tombol ke bawah. Mikrokontroler baru akan membalikkan status LED tepat di persekian detik saat kita mengangkat jari kita dari tombol. Mode ini sangat berguna untuk sistem yang membutuhkan validasi bahwa tindakan (penekanan) telah benar-benar selesai dilakukan oleh user.

### Source Code Modifikasi (Mode RISING)

```cpp
#include <Arduino.h>

// Deklarasi variabel volatile untuk ISR
volatile bool statusLampu = false;

// Fungsi ISR: Dieksekusi hanya saat tombol DILEPAS (LOW ke HIGH)
void ISR_TombolDilepas() {
  statusLampu = !statusLampu; // Membalik status (Toggle)
}

void setup() {
  pinMode(13, OUTPUT);         // Pin 13 sebagai output LED
  pinMode(2, INPUT_PULLUP);    // Pin 2 sebagai input tombol dengan resistor pull-up
  
  // Mengaktifkan interupsi di Pin 2 dengan mode RISING
  attachInterrupt(digitalPinToInterrupt(2), ISR_TombolDilepas, RISING);
}

void loop() {
  // LED mengikuti nilai statusLampu yang diubah oleh ISR
  digitalWrite(13, statusLampu); 
}
```

### Source Code Modifikasi (Mode CHANGE)

```cpp
#include <Arduino.h>

// Variabel volatile agar dapat diubah dalam ISR
volatile bool ledState = false;

// ISR: dijalankan saat terjadi perubahan status (CHANGE) pada pin
void tombolInterrupt() {
  ledState = !ledState; // Toggle status LED
}

void setup() {
  // Konfigurasi pin 13 sebagai output (LED)
  pinMode(13, OUTPUT);
  
  // Konfigurasi pin 2 sebagai input dengan pull-up internal
  pinMode(2, INPUT_PULLUP);
  
  // Daftarkan ISR pada pin 2, dipicu saat CHANGE (tombol ditekan/dilepas)
  attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, CHANGE);
}

void loop() {
  // Tulis status LED sesuai variabel ledState
  digitalWrite(13, ledState);
}
```


### Source Code Modifikasi (Mode LOW)

```cpp
#include <Arduino.h>

// Variabel volatile agar dapat diubah dalam ISR
volatile bool ledState = false;

// ISR: dijalankan saat terjadi perubahan status (CHANGE) pada pin
void tombolInterrupt() {
  ledState = !ledState; // Toggle status LED
}

void setup() {
  // Konfigurasi pin 13 sebagai output (LED)
  pinMode(13, OUTPUT);
  
  // Konfigurasi pin 2 sebagai input dengan pull-up internal
  pinMode(2, INPUT_PULLUP);
  
  // Daftarkan ISR pada pin 2, dipicu saat CHANGE (tombol ditekan atau dilepas)
  attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, CHANGE);
}

void loop() {
  // Tulis status LED sesuai variabel ledState
  digitalWrite(13, ledState);
}
```