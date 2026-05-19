# Jawaban Pertanyaan Praktikum Modul 6: Percobaan 6B (Timer Menggunakan millis())

**Nama:** Muhammad Aziz Ihza Fahriza Salam  
**NIM:** H1H024050  
**Mata Kuliah:** TK244005-Praktikum Sistem Mikrokontroller

---

## 1. Jelaskan bagaimana fungsi millis() bekerja pada program tersebut!

Fungsi `millis()` bertindak layaknya stopwatch internal yang mencatat sudah berapa milidetik perangkat Arduino menyala sejak pertama kali diberikan daya. Dalam program, mikrokontroler terus mengambil waktu saat ini (`currentMillis`) dan membandingkannya dengan catatan waktu terakhir lampu berkedip (`previousMillis`). Apabila selisih waktu di antara keduanya sudah mencapai atau melampaui `interval` yang ditentukan (1000 ms), mikrokontroler akan mengeksekusi perintah membalikkan status LED dan segera memperbarui catatan `previousMillis` dengan waktu saat ini. Jika selisihnya belum mencapai 1000 ms, mikrokontroler sekadar melewati kondisi tersebut.

## 2. Apa perbedaan utama antara delay() dan millis()?

Perbedaan utamanya terletak pada pemblokiran proses. Perintah `delay(1000`) akan melumpuhkan CPU mikrokontroler secara total selama 1 detik penuh; CPU hanya menghitung waktu mundur dan menolak mengerjakan baris program lainnya. Sebaliknya, pendekatan `millis()` tidak menghentikan apapun. CPU dibiarkan terus berlari mengeksekusi instruksi secara berulang di dalam `loop()`, dan hanya menggunakan `millis()` sekadar sebagai acuan waktu untuk mengeksekusi suatu blok kode secara periodik.

---

## 3. Mengapa metode millis() disebut non-blocking?

Disebut non-blocking karena metode ini tidak "memblokir" atau menghalangi eksekusi alur program selanjutnya. Evaluasi waktu menggunakan fungsi `if` berjalan dalam hitungan mikrodetik. Apabila kondisi waktu belum terpenuhi, sistem tidak akan terdiam menunggu, melainkan langsung beranjak membaca baris kode berikutnya di bawahnya. Hal ini memungkinkan Arduino untuk mengontrol berbagai komponen secara bersamaan (multitasking)

---

## 4. Modifikasi Program: 2 LED dengan Interval Berbeda Tanpa delay()

### Source Code Modifikasi

```cpp
#include <Arduino.h>

// Definisi pin
const int pinLed1 = 13; // LED pertama
const int pinLed2 = 12; // LED kedua

// Variabel untuk menyimpan waktu terakhir tiap LED berkedip
unsigned long waktuSblmLed1 = 0;
unsigned long waktuSblmLed2 = 0;

// Interval kedipan masing-masing LED (dalam milidetik)
const long intervalLed1 = 1000; // 1 detik
const long intervalLed2 = 500;  // 500 ms

// Variabel status masing-masing LED
bool statusLed1 = false;
bool statusLed2 = false;

void setup() {
  // Konfigurasi pin sebagai output
  pinMode(pinLed1, OUTPUT);
  pinMode(pinLed2, OUTPUT);
}

void loop() {
  // 1. Catat waktu saat ini
  unsigned long waktuSekarang = millis();

  // 2. Logika untuk LED 1 (Interval 1000 ms)
  if (waktuSekarang - waktuSblmLed1 >= intervalLed1) {
    waktuSblmLed1 = waktuSekarang;   // Perbarui penanda waktu LED 1
    statusLed1 = !statusLed1;        // Balikkan status LED 1
    digitalWrite(pinLed1, statusLed1); // Eksekusi ke perangkat keras
  }

  // 3. Logika untuk LED 2 (Interval 500 ms)
  if (waktuSekarang - waktuSblmLed2 >= intervalLed2) {
    waktuSblmLed2 = waktuSekarang;   // Perbarui penanda waktu LED 2
    statusLed2 = !statusLed2;        // Balikkan status LED 2
    digitalWrite(pinLed2, statusLed2); // Eksekusi ke perangkat keras
  }
}