# 📘 Tarea 8 — UART

> En esta práctica trabajamos con la **UART** del Raspberry Pi Pico 2 para enviar y recibir **comandos de texto** que controlan un LED. Se abordan dos escenarios:  
> 1) Un **botón físico** genera mensajes “LED ON / LED OFF” por UART.  
> 2) Una **terminal serial** en la PC envía esos mismos comandos al microcontrolador.

---

## 1) Botón que manda “LED ON / LED OFF” por UART

### ¿Qué hace?
Al **pulsar el botón**, el microcontrolador **alterna el estado** del LED (enciende/apaga) y **envía por UART**.

Este patrón permite **probar la transmisión UART**.

### ¿Cómo funciona?
- Configuramos la **UART0** con un baud rate típico.  
- El **botón** se conecta en modo **pull-up**: al presionarlo, el pin lee **0** (activo).  
- Detectamos el **flanco de bajada** para evitar múltiples toggles.
- Al cambiar el estado del LED, **mandamos el texto** por UART.

### Código
~~~c
#include "pico/stdlib.h"
#include "hardware/uart.h"
 
#define UART_ID uart0
#define BAUD_RATE 9600
 
#define UART_TX_PIN 0
#define UART_RX_PIN 1
 
#define LED_PIN 15
#define BUTTON_PIN 16
 
int main() {
    stdio_init_all();
    uart_init(UART_ID, BAUD_RATE);
    gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
 
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
 
    gpio_init(BUTTON_PIN);
    gpio_set_dir(BUTTON_PIN, GPIO_IN);
    gpio_pull_up(BUTTON_PIN);
 
    bool last_button_state = true;
    bool led_state = false;
 
    while (true) {
        bool button_state = gpio_get(BUTTON_PIN);
 
        // Si se detecta una pulsación (de HIGH a LOW)
        if (last_button_state && !button_state) {
            uart_putc(UART_ID, 'T'); // Enviamos el carácter 'T' al otro Pico
            sleep_ms(200); // anti rebote
        }
        last_button_state = button_state;
 
        // Si se recibe un byte por UART
        if (uart_is_readable(UART_ID)) {
            char c = uart_getc(UART_ID);
            if (c == 'T') {
                led_state = !led_state; // Cambiar estado del LED
                gpio_put(LED_PIN, led_state);
            }
        }
 
        sleep_ms(10);
    }
}
~~~

---

## Esquemático de conexión

![Esquemático ](../img/esquematicoultimo.jpeg)  
*Figura 1.*

---

## Video de demostracion

<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/coAK5xvE5a8?rel=0&modestbranding=1"
  title="Control motor DC - demo"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen
  style="border:0;">
</iframe>



## 2) Terminal serial que controla el LED con “LED ON / LED OFF”

> Desde una **terminal serial** en la PC se envían **comandos de texto** al microcontrolador por **UART**.  
> El firmware **lee** los caracteres, detecta el **fin de comando** y ejecuta:

---

### Código
~~~c
#include "pico/stdlib.h"
#include "hardware/uart.h"
#include <stdio.h>
#include <string>
 
#define UART_ID uart0
#define BAUD_RATE 115200
#define TX_PIN 0
#define RX_PIN 1
#define button_pin 16
#define led_PIN 15
using namespace std;
 
int main() {
    stdio_init_all();
 
    gpio_set_function(TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(RX_PIN, GPIO_FUNC_UART);
 
    uart_init(UART_ID, BAUD_RATE);
    uart_set_format(UART_ID, 8, 1, UART_PARITY_NONE);
 
    gpio_init(button_pin);
    gpio_set_dir(button_pin, GPIO_IN);
    gpio_pull_up(button_pin);
    gpio_init(led_PIN);
    gpio_set_dir(led_PIN, GPIO_OUT);
 
    string c = "";
    string p="";
    while (true){
 
        int ch = getchar_timeout_us(0);
        if (ch != PICO_ERROR_TIMEOUT) {
            printf("Eco: %c\n", (char)ch);
            p+= (char)ch;
            if(ch=='.' || ch=='\n'){
                uart_puts(UART_ID, p.c_str());
                p="";
            }
        }
        int a;
        if (gpio_get(button_pin) == 0 && a == 1) {
            printf("Button pressed!\n");
            uart_puts(UART_ID, "LEDON\n");
            sleep_ms(200); 
        }
         a= gpio_get(button_pin);
 
        if (uart_is_readable(uart0)) {
            char character = uart_getc(uart0);
            printf(character+"\n");
            if(character=='\n' || character=='.'){
                if (c == "LEDON"){
                    gpio_put(led_PIN, 1);
                    printf("LED is ON\n");
                }
                else if (c == "LEDOFF"){
                    gpio_put(led_PIN, 0);
                    printf("LED is OFF\n");
                } else if(c=="Invalid Command"){
                    printf("Invalid Command\n");
                }
                else{
                    uart_puts(UART_ID, "Invalid Command\n");
                }
                c = "";
                continue;
            }
            else{
                c += character;
            }
        }
    }
}
~~~

## Esquemático de conexión

![Esquemático — Driver de motor + Pico 2](../img/esquematicoultimo.jpeg)   
*Figura 1.*

---

## Video de demostracion

<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/8ufet_8OE6A"
  title="Control motor DC - demo"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen
  style="border:0;">
</iframe>