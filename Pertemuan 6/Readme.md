###6.5.4 Pertanyaan Praktikum
1. Jelaskan proses bagaimana tombol dapat mengubah kondisi LED menggunakan
interrupt!
2. Apa fungsi attachInterrupt() pada program tersebut?
3. Mengapa pada ISR tidak disarankan menggunakan delay() dan Serial.print()?
4. Apa fungsi keyword volatile pada variabel ledState?
5. Pada percobaan digunakan mode interrupt FALLING. Modifikasikan program
menggunakan mode interrupt lain (RISING, CHANGE, atau LOW) kemudian:
• Jelaskan perbedaan cara kerja masing-masing mode interrupt tersebut
• Analisis perubahan perilaku LED yang terjadi pada setiap mode
• Sertakan source code dan penjelasan program dalam bentuk README.md

Jawab:
1. Ketika tombol ditekan, pin 2 Arduino mengalami perubahan sinyal dari HIGH ke LOW karena menggunakan INPUT_PULLUP. Perubahan sinyal tersebut memicu interrupt dengan mode FALLING, sehingga Arduino langsung menjalankan ISR tombolInterrupt(). Di dalam ISR, variabel ledState diubah menggunakan operator toggle (!) sehingga kondisi LED berubah dari ON menjadi OFF atau sebaliknya. Setelah ISR selesai, program kembali menjalankan loop() dan LED diperbarui sesuai nilai ledState.
2. Fungsi attachInterrupt() digunakan untuk menghubungkan pin interrupt dengan ISR tertentu. Pada program ini, attachInterrupt() menghubungkan pin 2 dengan fungsi tombolInterrupt() menggunakan mode FALLING, sehingga ISR akan dijalankan setiap kali tombol ditekan.
3. Karena selama ISR berjalan, program utama akan dihentikan sementara. Penggunaan delay() dapat membuat sistem berhenti terlalu lama, sedangkan Serial.print() membutuhkan proses komunikasi serial yang relatif lambat. Hal tersebut dapat menyebabkan interrupt lain tertunda dan sistem menjadi tidak responsif.
4. Keyword volatile digunakan agar compiler selalu membaca nilai terbaru dari variabel ledState karena nilainya dapat berubah sewaktu-waktu di dalam ISR. Tanpa volatile, compiler dapat menganggap nilai variabel tidak berubah sehingga program bisa bekerja tidak sesuai.
5. modifikasi:
   a. Rising
  ```
  #include <Arduino.h>

volatile bool ledState = false;

// ISR
void tombolInterrupt() {
  ledState = !ledState;
}

void setup() {
  pinMode(13, OUTPUT);
  pinMode(2, INPUT_PULLUP);

  attachInterrupt(
    digitalPinToInterrupt(2),
    tombolInterrupt,
    RISING
  );
}

void loop() {
  digitalWrite(13, ledState);
}
  ```
Cara Kerja:
Interrupt akan aktif ketika sinyal berubah dari LOW ke HIGH. Pada rangkaian dengan INPUT_PULLUP, kondisi ini terjadi saat tombol dilepas.
Perilaku LED:
LED berubah kondisi ketika tombol dilepas, bukan saat tombol ditekan. Setiap tombol dilepas, ISR akan dijalankan dan status LED akan di-toggle.

   b. Change
   ```
  #include <Arduino.h>

volatile bool ledState = false;

void tombolInterrupt() {
  ledState = !ledState;
}

void setup() {
  pinMode(13, OUTPUT);
  pinMode(2, INPUT_PULLUP);

  attachInterrupt(
    digitalPinToInterrupt(2),
    tombolInterrupt,
    CHANGE
  );
}

void loop() {
  digitalWrite(13, ledState);
}
   ```
Cara Kerja:
Interrupt akan aktif setiap terjadi perubahan sinyal, baik dari HIGH ke LOW maupun LOW ke HIGH.
Perilaku LED:
LED dapat berubah kondisi dua kali, yaitu saat tombol ditekan dan saat tombol dilepas. Akibatnya LED dapat terlihat berkedip lebih cepat karena ISR dipanggil pada kedua kondisi perubahan sinyal.

   c. LOW
  ```
  #include <Arduino.h>

volatile bool ledState = false;

void tombolInterrupt() {
  ledState = !ledState;
}

void setup() {
  pinMode(13, OUTPUT);
  pinMode(2, INPUT_PULLUP);

  attachInterrupt(
    digitalPinToInterrupt(2),
    tombolInterrupt,
    LOW
  );
}

void loop() {
  digitalWrite(13, ledState);
}
  ```
Cara Kerja:
Interrupt akan terus aktif selama pin berada pada kondisi LOW. Pada INPUT_PULLUP, kondisi LOW terjadi ketika tombol ditekan.
Perilaku LED:
Selama tombol ditekan, ISR akan terus dipanggil berulang kali sehingga LED dapat berubah kondisi sangat cepat dan terlihat tidak stabil atau berkedip terus-menerus. Hal ini terjadi karena interrupt terus aktif selama sinyal LOW masih terdeteksi.



###6.6.4 Pertanyaan Praktikum
1. Jelaskan bagaimana fungsi millis() bekerja pada program tersebut!
2. Apa perbedaan utama antara delay() dan millis()?
3. Mengapa metode millis() disebut non-blocking?
4. Modifikasi program agar:
• LED pertama berkedip setiap 1 detik
• LED kedua berkedip setiap 500 ms
• Tanpa menggunakan delay()
Berikan penjelasan setiap baris program dalam bentuk README.md

Jawab:
1. Fungsi millis() mengembalikan waktu sejak Arduino dinyalakan dalam satuan milidetik. Program membandingkan nilai currentMillis dengan previousMillis untuk mengetahui apakah interval waktu tertentu telah tercapai. Jika selisih waktunya sudah mencapai 1000 ms, maka kondisi LED diubah.
2. delay() menghentikan seluruh proses program selama waktu tertentu sehingga program tidak dapat menjalankan tugas lain. Sedangkan millis() bersifat non-blocking karena hanya memeriksa selisih waktu tanpa menghentikan jalannya program utama.
3. Karena metode millis() tidak menghentikan eksekusi program. Selama menunggu interval waktu tertentu, program tetap dapat menjalankan proses lain secara bersamaan sehingga sistem menjadi lebih responsif dan efisien.
4. modifikasi:
```
#include <Arduino.h> 

// ===================== LED 1 =====================

// Menyimpan waktu terakhir LED 1 berubah kondisi
unsigned long previousMillis1 = 0;

// Interval kedip LED 1 = 1000 ms (1 detik)
const long interval1 = 1000;

// Menyimpan status LED 1 (ON/OFF)
bool ledState1 = false;


// ===================== LED 2 =====================

// Menyimpan waktu terakhir LED 2 berubah kondisi
unsigned long previousMillis2 = 0;

// Interval kedip LED 2 = 500 ms
const long interval2 = 500;

// Menyimpan status LED 2 (ON/OFF)
bool ledState2 = false;


// ===================== SETUP =====================

void setup() {

  // Pin 13 digunakan sebagai output untuk LED 1
  pinMode(13, OUTPUT);

  // Pin 12 digunakan sebagai output untuk LED 2
  pinMode(12, OUTPUT);
}


// ===================== LOOP =====================

void loop() {

  // Mengambil waktu saat ini sejak Arduino menyala
  unsigned long currentMillis = millis();


  // ===================== LED 1 =====================

  // Mengecek apakah selisih waktu sudah mencapai 1 detik
  if (currentMillis - previousMillis1 >= interval1) {

    // Menyimpan waktu terakhir LED berubah kondisi
    previousMillis1 = currentMillis;

    // Mengubah status LED (ON menjadi OFF / OFF menjadi ON)
    ledState1 = !ledState1;

    // Menuliskan status LED ke pin 13
    digitalWrite(13, ledState1);
  }


  // ===================== LED 2 =====================

  // Mengecek apakah selisih waktu sudah mencapai 500 ms
  if (currentMillis - previousMillis2 >= interval2) {

    // Menyimpan waktu terakhir LED berubah kondisi
    previousMillis2 = currentMillis;

    // Mengubah status LED (ON menjadi OFF / OFF menjadi ON)
    ledState2 = !ledState2;

    // Menuliskan status LED ke pin 12
    digitalWrite(12, ledState2);
  }
}
```



