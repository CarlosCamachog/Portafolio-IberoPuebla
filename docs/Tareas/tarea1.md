# Tarea 1 – Comparación y ranking de microcontroladores

## Índice
- [Microcontroladores elegidos](#microcontroladores-elegidos)  
- [Tabla comparativa](#tabla-comparativa)  
- [Ranking de microcontroladores](#ranking-de-microcontroladores)  

---

## Microcontroladores elegidos
- ESP32 (Espressif)  
- STM32F103C8T6 (STMicroelectronics)  
- RP2040 (Raspberry Pi Pico)  
- ATmega328P (Arduino UNO, Microchip)  

---

## Tabla comparativa

| MCU + Marca                | Periféricos                                                              | Memoria (Flash / RAM)        | Ecosistema                                | Costo aprox. (MXN) | Arquitectura               | Velocidad de trabajo |
|-----------------------------|-------------------------------------------------------------------------|------------------------------|-------------------------------------------|--------------------|----------------------------|----------------------|
| ESP32 (Espressif)           | Wi-Fi, Bluetooth, ADC 12b, 2×DAC, UART, SPI, I²C, PWM, touch            | ~4 MB Flash (módulo), ~520 KB SRAM | Arduino IDE o ESP-IDF, comunidad IoT       | 120–250            | Xtensa LX6 dual-core 32 bits | 240 MHz             |
| STM32F103C8T6 (ST)          | ADC 12b, UART, CAN, I²C, SPI, timers                                    | 64 KB Flash, 20 KB SRAM      | STM32CubeIDE/HAL, usado en industria       | 100–250            | ARM Cortex-M3 32 bits      | 72 MHz               |
| RP2040 (Raspberry Pi)       | 2×ADC, UART, SPI, I²C, PWM, PIO (programable)                          | 2 MB Flash, 264 KB SRAM      | SDK C/C++ y MicroPython, comunidad amplia  | 150–200            | ARM Cortex-M0+ dual-core 32 bits | 133 MHz             |
| ATmega328P (Microchip)      | ADC 10b, UART, SPI, I²C, timers                                         | 32 KB Flash, 2 KB SRAM       | Arduino IDE, comunidad educativa enorme    | 200–300 (Arduino UNO) | AVR 8 bits                 | 16 MHz               |

---

## Ranking de microcontroladores

1. **ESP32**  
   Ofrece la mayor cantidad de periféricos, conectividad (Wi-Fi y Bluetooth), bajo costo y muy buen soporte en IoT.  
   Es el más flexible para proyectos actuales de mecatrónica.

2. **RP2040 (Raspberry Pi Pico)**  
   Buena potencia con doble núcleo y un ecosistema fuerte con soporte oficial de Raspberry.  
   Aunque no tiene conectividad inalámbrica integrada, es muy versátil y económico.

3. **STM32F103C8T6 (Blue Pill)**  
   Uso común en la industria, arquitectura ARM Cortex-M3 y buena velocidad.  
   Es más complejo de programar que ESP32 y RP2040, lo que lo hace menos amigable para principiantes.

4. **ATmega328P (Arduino UNO)**  
   Más limitado en memoria y velocidad.  
   Se eligió porque es el micro más común en la enseñanza, con gran comunidad y documentación.

---
