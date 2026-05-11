5.5.4 Pertanyaan Praktikum
1. Apakah ketiga task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!
2. Bagaimana cara menambahkan task keempat? Jelaskan langkahnya!
3. Modifikasilah program dengan menambah sensor (misalnya potensiometer), lalu
gunakan nilainya untuk mengontrol kecepatan LED! Bagaimana hasilnya? Jelaskan
program pada file README.md.

jawab:
1. Ketiga task berjalan secara bergantian dengan sangat cepat menggunakan scheduler FreeRTOS. Scheduler mengatur pembagian waktu eksekusi setiap task sehingga masing-masing task mendapatkan giliran untuk dijalankan. Ketika suatu task menjalankan vTaskDelay(), task tersebut masuk ke kondisi blocked sehingga scheduler akan menjalankan task lain yang siap dieksekusi. Karena task berjalan sangat cepat, seolah-olah task berjalan secara bersamaan. 
2. Task keempat dapat ditambahkan dengan membuat fungsi task baru terlebih dahulu, misalnya TaskBlink3(). Setelah itu, task didaftarkan menggunakan fungsi xTaskCreate() di dalam setup(). Task baru dapat memiliki delay, prioritas, dan fungsi yang berbeda sesuai kebutuhan.
3. Modifikasi:
```
#include <Arduino_FreeRTOS.h>

void TaskBlink1(void *pvParameters);
void TaskBlink2(void *pvParameters);
void Taskprint(void *pvParameters);

const int led1 = 8;
const int led2 = 7;
const int potPin = A0;

int nilaiPot = 0;
int delayLed = 200;

void setup() {

  Serial.begin(9600);

  xTaskCreate(
    TaskBlink1,
    "task1",
    128,
    NULL,
    1,
    NULL
  );

  xTaskCreate(
    TaskBlink2,
    "task2",
    128,
    NULL,
    1,
    NULL
  );

  xTaskCreate(
    Taskprint,
    "task3",
    128,
    NULL,
    1,
    NULL
  );

  vTaskStartScheduler();
}

void loop() {

}

void TaskBlink1(void *pvParameters) {

  pinMode(led1, OUTPUT);

  while (1) {

    nilaiPot = analogRead(potPin);

    delayLed = map(nilaiPot, 0, 1023, 100, 1000);

    digitalWrite(led1, HIGH);
    vTaskDelay(delayLed / portTICK_PERIOD_MS);

    digitalWrite(led1, LOW);
    vTaskDelay(delayLed / portTICK_PERIOD_MS);
  }
}

void TaskBlink2(void *pvParameters) {

  pinMode(led2, OUTPUT);

  while (1) {

    digitalWrite(led2, HIGH);
    vTaskDelay((delayLed + 100) / portTICK_PERIOD_MS);

    digitalWrite(led2, LOW);
    vTaskDelay((delayLed + 100) / portTICK_PERIOD_MS);
  }
}

void Taskprint(void *pvParameters) {

  int counter = 0;

  while (1) {

    counter++;

    Serial.print("Counter: ");
    Serial.print(counter);

    Serial.print(" | Potensiometer: ");
    Serial.print(nilaiPot);

    Serial.print(" | Delay LED: ");
    Serial.println(delayLed);

    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
```
Penjelasan:
  Potensiometer digunakan sebagai input untuk mengatur kecepatan kedipan LED. Nilai analog dari potensiometer dibaca menggunakan fungsi analogRead() kemudian dipetakan menggunakan fungsi map() menjadi nilai delay antara 100 ms hingga 1000 ms. Semakin besar nilai potensiometer, semakin besar delay yang dihasilkan sehingga LED berkedip lebih lambat, sedangkan nilai potensiometer yang kecil membuat LED berkedip lebih cepat.
  Program tetap menggunakan FreeRTOS dengan tiga task yang berjalan secara concurrent, yaitu task LED pertama, task LED kedua, dan task pencetakan data pada Serial Monitor. Scheduler FreeRTOS mengatur pergantian eksekusi setiap task menggunakan vTaskDelay() sehingga seluruh task dapat berjalan secara bersamaan tanpa saling menghambat. Hasil percobaan menunjukkan bahwa perubahan nilai potensiometer dapat memengaruhi kecepatan kedipan LED secara real-time dan sistem tetap berjalan stabil meskipun menjalankan beberapa task sekaligus.
   
5.6.4 Pertanyaan Praktikum
1. Apakah kedua task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!
2. Apakah program ini berpotensi mengalami race condition? Jelaskan!
3. Modifikasilah program dengan menggunakan sensor DHT sesungguhnya sehingga
informasi yang ditampilkan dinamis. Bagaimana hasilnya? Jelaskan program pada file
README.md.

jawab:
1. Kedua task berjalan secara concurrent menggunakan scheduler FreeRTOS. Task read_data bertugas mengirim data ke queue, sedangkan task display menerima dan menampilkan data tersebut ke Serial Monitor. Ketika salah satu task sedang delay atau menunggu queue, scheduler akan menjalankan task lainnya sehingga kedua task dapat bekerja secara bergantian dengan teratur.
2. Program ini memiliki risiko race condition yang sangat kecil karena komunikasi data dilakukan menggunakan queue FreeRTOS. Queue bekerja sebagai media pertukaran data yang aman sehingga akses data antar task diatur oleh sistem RTOS. Dengan demikian, data tidak diakses secara bersamaan oleh dua task pada waktu yang sama sehingga konflik akses data dapat dihindari.
3. Modifikasi:
```
#include <Arduino_FreeRTOS.h>
#include <queue.h>
#include <DHT.h>

#define DHTPIN 2
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

struct readings {
  int temp;
  int h;
};

QueueHandle_t my_queue;

void read_data(void *pvParameters);
void display(void *pvParameters);

void setup() {

  Serial.begin(9600);

  dht.begin();

  my_queue = xQueueCreate(1, sizeof(struct readings));

  xTaskCreate(
    read_data,
    "read sensors",
    128,
    NULL,
    0,
    NULL
  );

  xTaskCreate(
    display,
    "display",
    128,
    NULL,
    0,
    NULL
  );
}

void loop() {

}

void read_data(void *pvParameters) {

  struct readings x;

  for (;;) {

    x.temp = dht.readTemperature();
    x.h = dht.readHumidity();

    xQueueSend(my_queue, &x, portMAX_DELAY);

    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

void display(void *pvParameters) {

  struct readings x;

  for (;;) {

    if (xQueueReceive(my_queue, &x, portMAX_DELAY) == pdPASS) {

      Serial.print("Temperature = ");
      Serial.println(x.temp);

      Serial.print("Humidity = ");
      Serial.println(x.h);
    }
  }
}
```
Penjelasan:
  Pada modifikasi ini, sensor DHT11 digunakan untuk membaca suhu dan kelembapan secara langsung. Program terdiri dari dua task, yaitu read_data untuk membaca data sensor dan mengirimkannya ke queue menggunakan xQueueSend(), serta display untuk menerima data dari queue menggunakan xQueueReceive() dan menampilkannya pada Serial Monitor.
  Penggunaan queue membuat komunikasi antar task menjadi lebih aman dan terstruktur. Hasil percobaan menunjukkan bahwa nilai suhu dan kelembapan berubah secara dinamis sesuai kondisi lingkungan, sementara kedua task tetap dapat berjalan secara concurrent dengan pengaturan scheduler FreeRTOS.
