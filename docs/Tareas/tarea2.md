# Tarea 2 – Outputs básicos con lógica y máscaras

## Índice
- [Contador binario de 4 bits](#contador-binario-de-4-bits)  
- [Barrido de LEDs](#barrido-de-leds)  
- [Secuencia en codigo Gray](#secuencia-en-codigo-gray)  

---

## Contador binario de 4 bits 

### Nombre del código
Contador binario (4 bits)

### Qué debe hacer
Este programa genera un contador binario ascendente de 4 bits.
Los LEDs conectados muestran en tiempo real la representación binaria de los valores del 0 al 15, actualizándose cada 500 ms.

### Código
~~~c
#include "pico/stdlib.h"
#include "hardware/gpio.h"

#define A 0
#define B 1
#define C 2
#define D 3

int main() {
    const uint32_t MASK = (1u<<A) | (1u<<B) | (1u<<C) | (1u<<D);
    gpio_init_mask(MASK);
    gpio_set_dir_masked(MASK, MASK);
    while (true) {
        for (uint8_t i = 0; i < 16; i++) {
            gpio_put_masked(MASK, i << A);
            sleep_ms(500);
        }
    }
}
~~~

### Esquemático de conexión
![Esquemático del contador](../img/Contador.svg)

### Video
<iframe width="560" height="315"
src="https://www.youtube.com/embed/aDn2G8YxF9o"
frameborder="0" allowfullscreen></iframe>

---

## Barrido de LEDs 

### Nombre del código
Barrido de LEDs

### Qué debe hacer
Se enciende un único LED a la vez en orden ascendente (0→1→2→3) y posteriormente en orden descendente (3→2→1→0), repitiendo indefinidamente la secuencia.

### Código
~~~c
#include "pico/stdli-b.h"
#include "hardware/gpio.h"

#define A 0   
#define B 1   
#define C 2   
#define D 3 
#define E 4  

int main() {
    const uint32_t MASK = (1u<<A) | (1u<<B) | (1u<<C) | (1u<<D) | (1u<<E);

    gpio_init_mask(MASK);
    gpio_set_dir_out_masked(MASK);   
    gpio_clr_mask(MASK);             

    while (true) {
       
        for (int i = 0; i < 5; ++i) {
            gpio_clr_mask(MASK);                
            gpio_set_mask(1 << i);              
            sleep_ms(300);
        }
        
        for (int i = 3; i > 0; --i) {
            gpio_clr_mask(MASK);
            gpio_set_mask(1 << i);
            sleep_ms(300);
        }
    }
}
~~~

### Esquemático de conexión
![Esquemático de conexión](../img/Barrido.svg)

### Video
<iframe width="560" height="315"
src="https://www.youtube.com/embed/biWVJNRUWUI"
frameborder="0" allowfullscreen></iframe>

---

## Secuencia en codigo Gray 

### Nombre del código
Secuencia en codigo Gray

### Qué debe hacer
En este programa se representa la secuencia de Gray de 3 bits.
En cada transición únicamente cambia un bit del patrón de salida, lo que permite observar la principal característica del código Gray.
El codigo convierte números binarios a Gray mediante la operación n ^ (n >> 1).

### Código
~~~c
#include "pico/stdlib.h"
#include "hardware/gpio.h"

#define A 0
#define B 1
#define C 2

uint8_t bin_gray(uint8_t num_dec) {
    return num_dec ^ (num_dec >> 1);
}

int main() {
    const uint8_t MASK = (1u << A) | (1u << B) | (1u << C);

    gpio_init_mask(MASK);
    gpio_set_dir_masked(MASK, MASK);

    while (true) {
        for (uint8_t i = 0; i < 8; i++) {
            uint8_t gray = bin_gray(i);
            gpio_put_masked(MASK, gray);
            sleep_ms(500);
        }
    }
}
~~~

### Esquemático de conexión
![Esquemático de conexión](../img/Codigo Gray.svg)

### Video
<iframe width="560" height="315"
src="https://www.youtube.com/embed/Q9hNYVJSGg4"
frameborder="0" allowfullscreen></iframe>
