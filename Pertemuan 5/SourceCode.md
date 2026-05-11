Percobaan 1:
```
#include <Arduino_FreeRTOS.h>

void TaskBlink1(void *pvParameters);
void TaskBlink2(void *pvParameters);
void Taskprint(void *pvParameters);

void setup() {

  // initialize serial communication at 9600 bits per second:
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

  pinMode(8, OUTPUT);

  while (1) {

    Serial.println("Task1");

    digitalWrite(8, HIGH);
    vTaskDelay(200 / portTICK_PERIOD_MS);

    digitalWrite(8, LOW);
    vTaskDelay(200 / portTICK_PERIOD_MS);
  }
}

void TaskBlink2(void *pvParameters) {

  pinMode(7, OUTPUT);

  while (1) {

    Serial.println("Task2");

    digitalWrite(7, HIGH);
    vTaskDelay(300 / portTICK_PERIOD_MS);

    digitalWrite(7, LOW);
    vTaskDelay(300 / portTICK_PERIOD_MS);
  }
}

void Taskprint(void *pvParameters) {

  int counter = 0;

  while (1) {

    counter++;

    Serial.println(counter);

    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}
```

Percobaan 2:
```
#include <Arduino_FreeRTOS.h>
#include <queue.h>

struct readings {
  int temp;
  int h;
};

QueueHandle_t my_queue;

void setup() {

  Serial.begin(9600);

  my_queue = xQueueCreate(1, sizeof(struct readings));

  xTaskCreate(read_data, "read sensors", 128, NULL, 0, NULL);
  xTaskCreate(display, "display", 128, NULL, 0, NULL);
}

void loop() {

}

/*
 *
 * Blink task.
 * See Blink_AnalogRead example.
 *
 */

void read_data(void *pvParameters) {

  struct readings x;

  for (;;) {

    x.temp = 54;
    x.h = 30;

    xQueueSend(my_queue, &x, portMAX_DELAY);

    vTaskDelay(100);
  }
}

void display(void *pvParameters) {

  struct readings x;

  for (;;) {

    if (xQueueReceive(my_queue, &x, portMAX_DELAY) == pdPASS) {

      Serial.print("temp = ");
      Serial.println(x.temp);

      Serial.print("humidity = ");
      Serial.println(x.h);
    }
  }
}
```
