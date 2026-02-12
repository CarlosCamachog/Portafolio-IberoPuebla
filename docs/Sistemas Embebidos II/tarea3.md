# Session 3 — FreeRTOS Tasks, Queues, and Mutex

Implement multiple FreeRTOS tasks in ESP-IDF, including heartbeat, alive logs, queue communication, mutex protection, and error logging.

---

## 1) Activity Goals

- Implement Task 1 (Heartbeat LED)
- Implement Task 2 (Alive task every 2 seconds)
- Implement Task 3 (Queue Struct Send)
- Implement Task 4 (Queue Struct Receive)
- Implement Task 5 (Mutex protected button reading)
- Implement Task 6 (Second mutex protected button task)
- Implement Task 7 (Error logging system)

---

## 2) Materials & Setup

### BOM (Bill of Materials)

| # | Item | Qty | Link/Source | Cost (MXN) | Notes |
|---|------|-----|------------|------------|------|
| 1 | ESP32 Board | 1 | Local Store / Amazon / MercadoLibre | 365 | Main development board |
| 2 | LED | 1 | Local electronics store | 3 | Status LED connected to GPIO 8 |
| 3 | Push Button | 1 | Local electronics store / Amazon / MercadoLibre | 1.70-3.60 | Button connected to GPIO ___ |

### Tools/Software

- **Framework:** ESP-IDF + FreeRTOS  

---

## 3) Procedure (what you did)

1. Created FreeRTOS tasks for heartbeat and alive monitoring.
2. Implemented queue producer/consumer communication with structs.
3. Implemented mutex protection for shared resources.
4. Implemented error logging system.
5. Verified correct execution using serial monitor output and LED behavior.

---

# 4) Evidence (Console, Photos, Videos)

## Task without error logging system

<div style="text-align: center;">
<iframe width="560" height="315"
src="https://www.youtube.com/embed/3GtFfmuL6As"
title="Task without error logging system"
frameborder="0"
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
allowfullscreen></iframe>
</div>

---

## Task with error logging system

<div style="text-align: center;">
<iframe width="560" height="315"
src="https://www.youtube.com/embed/S55XxNcCho0"
title="Task with error logging system"
frameborder="0"
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
allowfullscreen></iframe>
</div>

---

# 5) Code

## Task with error logging system

~~~c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
#include "driver/gpio.h"
#include "freertos/queue.h"
#include "esp_log.h"


#define LED_GPIO GPIO_NUM_8   // CHANGE for your board
#define BUTTON_GPIO GPIO_NUM_10 // CHANGE for your board

static const char *TAG = "HearbeatApp";
volatile bool error[7] = {false, false, false, false, false, false};

static QueueHandle_t q_numbers;
static QueueHandle_t q_errors;
static SemaphoreHandle_t shared_counter;

struct DataPacket {
    int id;
    float value;
};

struct ErrorStatus {
    int task_id;
    bool error_state;
};

static void button_task1(void *pvParameters)
{
    gpio_reset_pin(BUTTON_GPIO);
    gpio_set_direction(BUTTON_GPIO, GPIO_MODE_INPUT);
    gpio_set_pull_mode(BUTTON_GPIO, GPIO_PULLUP_ONLY);
    float pressed_time = 0.0f;

    while (1) {
        bool current_state = !gpio_get_level(BUTTON_GPIO); // Active low
        if (current_state) {
            pressed_time += 0.05f; // Simulating time increment
        } 
        else if (!current_state && pressed_time > 0.0f) {
            ESP_LOGI(TAG, "Button 1 pressed for %f seconds", pressed_time);
            
            struct ErrorStatus status = {5, true};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            
            pressed_time = 0.0f;
        }
        
        vTaskDelay(pdMS_TO_TICKS(50)); // Polling delay
    }
}

static void button_task2(void *pvParameters)
{
    gpio_reset_pin(BUTTON_GPIO);
    gpio_set_direction(BUTTON_GPIO, GPIO_MODE_INPUT);
    gpio_set_pull_mode(BUTTON_GPIO, GPIO_PULLUP_ONLY);
    float pressed_time = 0.0f;

    while (1) {
        bool current_state = !gpio_get_level(BUTTON_GPIO); // Active low
        if (current_state) {
            pressed_time += 0.05f; // Simulating time increment
        }
        else if (!current_state && pressed_time > 0.0f) {
            ESP_LOGI(TAG, "Button 2 pressed for %f seconds", pressed_time);
            
            struct ErrorStatus status = {6, true};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            
            pressed_time = 0.0f;
        }
        
        vTaskDelay(pdMS_TO_TICKS(50)); // Polling delay
    }
}

static void producer_task(void *pvParameters)
{
    struct DataPacket packet;
    packet.id = 0;
    packet.value = 0.0f;
    while (1) {
       packet.id=rand()%1000;
       packet.value=(float)(rand()%1000)/10.0f;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &packet, pdMS_TO_TICKS(50)) == pdPASS) {
            struct ErrorStatus status = {3, true};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGI(TAG, "Produced %d with value %f", packet.id, packet.value);
        } else {
            struct ErrorStatus status = {3, true};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGW(TAG, "Queue full, dropped %d with value %f", packet.id, packet.value);
        }

        vTaskDelay(pdMS_TO_TICKS(200));
    }
}

static void consumer_task(void *pvParameters)
{
     struct DataPacket packet_recv;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &packet_recv, pdMS_TO_TICKS(1500)) == pdPASS) {
            struct ErrorStatus status = {4, true};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGI(TAG, "Consumed %d with value %f", packet_recv.id, packet_recv.value);
        } else {
            struct ErrorStatus status = {4, true};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

static void heartbeat_task(void *pvParameters)
{
    while (1) {
        struct ErrorStatus status = {0, true};
        xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
        ESP_LOGI(TAG, "alive");
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}

static void hearbeatLED_task(void *pvParameters)
{
    gpio_reset_pin(LED_GPIO);
    gpio_set_direction(LED_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        struct ErrorStatus status = {2, true};
        xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
        gpio_set_level(LED_GPIO, 1);
        vTaskDelay(pdMS_TO_TICKS(300));
        gpio_set_level(LED_GPIO, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

static void bpm_task(void *pvParameters){
    while (1) {
        struct ErrorStatus status = {1, true};
        xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
        printf("BPM: %d\n", rand()%40 + 60);
        vTaskDelay(pdMS_TO_TICKS(500));
    }
}

static void error_task(void *pvParameters)
{
    struct ErrorStatus status;
    while (1) {
        if (xQueueReceive(q_errors, &status, pdMS_TO_TICKS(100)) == pdPASS) {
            if (xSemaphoreTake(shared_counter, pdMS_TO_TICKS(10)) == pdTRUE) {
                error[status.task_id] = status.error_state;
                if (status.error_state) {
                    ESP_LOGE(TAG, "Error task %d", status.task_id);
                }
                xSemaphoreGive(shared_counter);
            }
        }
    }
}

void app_main(void)
{
    // Create mutex for shared resources
    shared_counter = xSemaphoreCreateMutex();
    if (shared_counter == NULL) {
        ESP_LOGE(TAG, "Mutex create failed");
        return;
    }
    
     q_numbers = xQueueCreate(5, sizeof(struct DataPacket)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    q_errors = xQueueCreate(10, sizeof(struct ErrorStatus));
    if (q_errors == NULL) {
        ESP_LOGE(TAG, "Error queue create failed");
        return;
    }

    xTaskCreate(error_task, "error_task", 2048, NULL, 0, NULL);
    xTaskCreate(heartbeat_task, "heartbeat", 2048, NULL, 0, NULL);
    xTaskCreate(bpm_task, "bpm_task", 2048, NULL, 1, NULL);
    xTaskCreate(hearbeatLED_task, "hearbeatLED_task", 2048, NULL, 1, NULL);
    xTaskCreate(producer_task, "producer_task", 2048, NULL, 2, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 2, NULL);
    xTaskCreate(button_task1, "button_task1", 2048, NULL, 3, NULL);
    xTaskCreate(button_task2, "button_task2", 2048, NULL, 3, NULL);

}
~~~

---

## Task without error logging system

~~~c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
#include "driver/gpio.h"
#include "freertos/queue.h"
#include "esp_log.h"


#define LED_GPIO GPIO_NUM_8   // CHANGE for your board
#define BUTTON_GPIO GPIO_NUM_10 // CHANGE for your board

static const char *TAG = "HearbeatApp";
volatile bool error[7] = {false, false, false, false, false, false};

static QueueHandle_t q_numbers;
static QueueHandle_t q_errors;
static SemaphoreHandle_t shared_counter;

struct DataPacket {
    int id;
    float value;
};

struct ErrorStatus {
    int task_id;
    bool error_state;
};

static void button_task1(void *pvParameters)
{
    gpio_reset_pin(BUTTON_GPIO);
    gpio_set_direction(BUTTON_GPIO, GPIO_MODE_INPUT);
    gpio_set_pull_mode(BUTTON_GPIO, GPIO_PULLUP_ONLY);
    float pressed_time = 0.0f;

    while (1) {
        bool current_state = !gpio_get_level(BUTTON_GPIO); // Active low
        if (current_state) {
            pressed_time += 0.05f; // Simulating time increment
        } 
        else if (!current_state && pressed_time > 0.0f) {
            ESP_LOGI(TAG, "Button 1 pressed for %f seconds", pressed_time);
            
            struct ErrorStatus status = {5, false};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            
            pressed_time = 0.0f;
        }
        
        vTaskDelay(pdMS_TO_TICKS(50)); // Polling delay
    }
}

static void button_task2(void *pvParameters)
{
    gpio_reset_pin(BUTTON_GPIO);
    gpio_set_direction(BUTTON_GPIO, GPIO_MODE_INPUT);
    gpio_set_pull_mode(BUTTON_GPIO, GPIO_PULLUP_ONLY);
    float pressed_time = 0.0f;

    while (1) {
        bool current_state = !gpio_get_level(BUTTON_GPIO); // Active low
        if (current_state) {
            pressed_time += 0.05f; // Simulating time increment
        }
        else if (!current_state && pressed_time > 0.0f) {
            ESP_LOGI(TAG, "Button 2 pressed for %f seconds", pressed_time);
            
            struct ErrorStatus status = {6, false};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            
            pressed_time = 0.0f;
        }
        
        vTaskDelay(pdMS_TO_TICKS(50)); // Polling delay
    }
}

static void producer_task(void *pvParameters)
{
    struct DataPacket packet;
    packet.id = 0;
    packet.value = 0.0f;
    while (1) {
       packet.id=rand()%1000;
       packet.value=(float)(rand()%1000)/10.0f;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &packet, pdMS_TO_TICKS(50)) == pdPASS) {
            struct ErrorStatus status = {3, false};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGI(TAG, "Produced %d with value %f", packet.id, packet.value);
        } else {
            struct ErrorStatus status = {3, false};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGW(TAG, "Queue full, dropped %d with value %f", packet.id, packet.value);
        }

        vTaskDelay(pdMS_TO_TICKS(200));
    }
}

static void consumer_task(void *pvParameters)
{
     struct DataPacket packet_recv;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &packet_recv, pdMS_TO_TICKS(1500)) == pdPASS) {
            struct ErrorStatus status = {4, false};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGI(TAG, "Consumed %d with value %f", packet_recv.id, packet_recv.value);
        } else {
            struct ErrorStatus status = {4, false};
            xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

static void heartbeat_task(void *pvParameters)
{
    while (1) {
        struct ErrorStatus status = {0, false};
        xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
        ESP_LOGI(TAG, "alive");
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}

static void hearbeatLED_task(void *pvParameters)
{
    gpio_reset_pin(LED_GPIO);
    gpio_set_direction(LED_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        struct ErrorStatus status = {2, false};
        xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
        gpio_set_level(LED_GPIO, 1);
        vTaskDelay(pdMS_TO_TICKS(300));
        gpio_set_level(LED_GPIO, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

static void bpm_task(void *pvParameters){
    while (1) {
        struct ErrorStatus status = {1, false};
        xQueueSend(q_errors, &status, pdMS_TO_TICKS(10));
        printf("BPM: %d\n", rand()%40 + 60);
        vTaskDelay(pdMS_TO_TICKS(500));
    }
}

static void error_task(void *pvParameters)
{
    struct ErrorStatus status;
    while (1) {
        if (xQueueReceive(q_errors, &status, pdMS_TO_TICKS(100)) == pdPASS) {
            if (xSemaphoreTake(shared_counter, pdMS_TO_TICKS(10)) == pdTRUE) {
                error[status.task_id] = status.error_state;
                xSemaphoreGive(shared_counter);
            }
        }
    }
}

void app_main(void)
{
    // Create mutex for shared resources
    shared_counter = xSemaphoreCreateMutex();
    if (shared_counter == NULL) {
        ESP_LOGE(TAG, "Mutex create failed");
        return;
    }
    
     q_numbers = xQueueCreate(5, sizeof(struct DataPacket)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    q_errors = xQueueCreate(10, sizeof(struct ErrorStatus));
    if (q_errors == NULL) {
        ESP_LOGE(TAG, "Error queue create failed");
        return;
    }

    xTaskCreate(error_task, "error_task", 2048, NULL, 0, NULL);
    xTaskCreate(heartbeat_task, "heartbeat", 2048, NULL, 0, NULL);
    xTaskCreate(bpm_task, "bpm_task", 2048, NULL, 1, NULL);
    xTaskCreate(hearbeatLED_task, "hearbeatLED_task", 2048, NULL, 1, NULL);
    xTaskCreate(producer_task, "producer_task", 2048, NULL, 2, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 2, NULL);
    xTaskCreate(button_task1, "button_task1", 2048, NULL, 3, NULL);
    xTaskCreate(button_task2, "button_task2", 2048, NULL, 3, NULL);

}
~~~

---

# 6) Files & Media

- Firmware file: [Descargar main.c](recursos/archivos/ESII_TASK3/main.c)
- Videos: 
    1. [Task without error logging system](https://youtu.be/3GtFfmuL6As)
    2. [Task with error logging system](https://youtu.be/S55XxNcCho0)