# Lab 1 — Two Tasks, Delays, and Priorities

---
## Code (Lab 1)
~~~c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_log.h"

#define LED_GPIO GPIO_NUM_8   // CHANGE for your board

static const char *TAG = "LAB1";

static void blink_task(void *pvParameters)
{
    gpio_reset_pin(LED_GPIO);
    gpio_set_direction(LED_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        gpio_set_level(LED_GPIO, 1);
        vTaskDelay(pdMS_TO_TICKS(300));
        gpio_set_level(LED_GPIO, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

static void hello_task(void *pvParameters)
{
    int n = 0;
    while (1) {
        ESP_LOGI(TAG, "hello_task says hi, n=%d", n++);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 1 (two tasks)");

    xTaskCreate(blink_task, "blink_task", 2048, NULL, 5, NULL);
    xTaskCreate(hello_task, "hello_task", 2048, NULL, 5, NULL);
}
~~~
## Evidence

### Console Output
[INSERT SCREENSHOT HERE]

### LED Blinking
[INSERT PHOTO HERE]


## Exercises (Lab 1)


### Exercise 1 — Priority Experiment
 
Change `hello_task` priority from **5** to **2**:

~~~c
xTaskCreate(hello_task, "hello_task", 2048, NULL, 2, NULL);
~~~

### Exercise 2 - Does behavior change? Why might it (or might it not)?
NO, because its doesn’t affect the blink time of led. The waiting is not bigger to change the blink to human eye

## Evidence

### Exercise 2 — Starvation Demo
 
Temporarily remove this line from hello_task: 
~~~c
vTaskDelay(pdMS_TO_TICKS(1000));
~~~

### Exercise 4 - What happens to blinking?
it’s slower because “hello_task” is always running so “led_task” it’s waiting 

## Evidence

### Exercise 5 — Put the delay back and explain in one sentence why blocking helps
Blocking helps because vTaskDelay() puts the task in a blocked state and frees the CPU so other tasks can run.


---
# Lab 2 — Queue (Producer / Consumer)
---

## Code (Lab 2)

~~~c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_log.h"

static const char *TAG = "LAB2";
static QueueHandle_t q_numbers;

static void producer_task(void *pvParameters)
{
    int value = 0;

    while (1) {
        value++;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &value, pdMS_TO_TICKS(50)) == pdPASS) {
            ESP_LOGI(TAG, "Produced %d", value);
        } else {
            ESP_LOGW(TAG, "Queue full, dropped %d", value);
        }

        vTaskDelay(pdMS_TO_TICKS(200));
    }
}

static void consumer_task(void *pvParameters)
{
    int rx = 0;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &rx, pdMS_TO_TICKS(1000)) == pdPASS) {
            ESP_LOGI(TAG, "Consumed %d", rx);
        } else {
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 2 (queue)");

    q_numbers = xQueueCreate(5, sizeof(int)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    xTaskCreate(producer_task, "producer_task", 2048, NULL, 5, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 5, NULL);
}
~~~

## Evidence

## Exercises (Lab 2)

### Exercise 1 — Make the producer faster: change producer delay 200ms → 20ms
~~~c
vTaskDelay(pdMS_TO_TICKS(20));
~~~
### Exercise 2 — When do you see “Queue full”?
Never

## Evidence

### Exercise 3 — Increase the queue length 5 → 20
~~~c
q_numbers = xQueueCreate(20, sizeof(int));
~~~

### Exercise 4 — What changes?
The queue can store more values, so "Queue full" happens less often or takes longer to appear.

### Exercise 5 — Make the consumer “slow”: after a successful receive, add:
~~~c
vTaskDelay(pdMS_TO_TICKS(300));
~~~
### Exercise 6 — What pattern is happening now (buffering / backlog)?
The queue stores values temporarily, then it fills up and the producer starts dropping values because there is no more space.

## Exercises (Lab 3)

---

### Part A — Race demo (no mutex)

#### Question  
Why can the counter be wrong?

#### Answer  
The counter may be faulty because both tasks read the same value and overwrite it, losing increments.

## Evidence  


---

### Part B — Fix with a mutex

## Evidence  


---

### Exercise 1 — Remove the mutex again

#### Question  
Do you ever see weird behavior?

#### Answer  
Yes, sometimes the counter increases slower than expected or skips values. 

## Evidence  


---

### Exercise 2 — Change priorities (TaskA = 6, TaskB = 4)

#### Question  
What do you expect and why?

#### Answer  
TaskA (priority 6) will run more often than TaskB (priority 4). TaskA will usually print more messages because it gets CPU time first when both tasks are ready.

#### Evidence  


---

### Exercise 3 — One sentence

#### Question  
In one sentence: what does a mutex “guarantee”?

#### Answer  
A mutex guarantees that only one task at a time can access the shared resource , preventing race conditions.

---
