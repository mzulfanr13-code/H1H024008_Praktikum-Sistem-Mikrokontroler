Percobaan 1:
```
#include <Arduino.h>
// Variabel volatile agar dapat diubah dalam ISR
volatile bool ledState = false;
// ISR: dijalankan saat tombol ditekan (FALLING edge)
void tombolInterrupt() {
 ledState = !ledState; // Toggle status LED
}
void setup() {
 // Konfigurasi pin 13 sebagai output (LED)
 pinMode(13, OUTPUT);
 // Konfigurasi pin 2 sebagai input dengan pull-up internal
 pinMode(2, INPUT_PULLUP);
 // Daftarkan ISR pada pin 2, dipicu FALLING (tombol ditekan)
 attachInterrupt(
 digitalPinToInterrupt(2),
 tombolInterrupt,
 FALLING
 );
}
void loop() {
 // Tulis status LED sesuai variabel ledState
 digitalWrite(13, ledState);
}
```

Percobaan 2:
```
#include <Arduino.h>
unsigned long previousMillis = 0; // waktu terakhir LED berubah
const long interval = 1000; // interval kedip: 1000 ms
bool ledState = false; // status LED saat ini
void setup() {
 pinMode(13, OUTPUT); // Pin 13 sebagai output
}
void loop() {
 // Ambil waktu saat ini
 unsigned long currentMillis = millis();
 // Cek apakah sudah melewati interval
 if(currentMillis - previousMillis >= interval) {
 previousMillis = currentMillis; // simpan waktu terakhir
 ledState = !ledState; // toggle status LED
 digitalWrite(13, ledState); // tulis ke pin LED
 }
}
```
