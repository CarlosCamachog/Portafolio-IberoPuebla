# 📘 Tarea 1 — Comparación y ranking de microcontroladores

> Documento de comparación y **ranking** de 4 MCUs.

---

## 1) Resumen

- **Nombre del proyecto:** Comparación y ranking de microcontroladores  
- **Autor:** Carlos Ernesto Camacho González
- **Curso / Asignatura:** Sistemas Embebidos  
- **Fecha:** 02/09/2025  
- **Descripción breve:** Se comparan ESP32, STM32F103C8T6, RP2040 y ATmega328P en periféricos, memoria, ecosistema, costo y rendimiento para proponer un **ranking** de uso recomendado.

!!! tip "Consejo"
     Elige el MCU según el **contexto**: conectividad IoT, facilidad para aprender, costo o escalabilidad industrial.

---

## 2) Objetivos

- **General:** Identificar el microcontrolador más conveniente según requisitos típicos de proyectos de mecatrónica.

- **Específicos:**
  - Comparar **periféricos** y **conectividad** integrada.
  - Contrastar **memoria**, **arquitectura** y **frecuencia** de trabajo.
  - Valorar el **ecosistema** (IDE/SDK, documentación y comunidad).
  - Considerar **costo aproximado** en MXN y **facilidad de uso**.
  - Proponer un **ranking** justificado y casos de uso sugeridos.

---

## 3) Alcance y Exclusiones

- **Incluye:** 4 MCUs representativos (ESP32, STM32F103C8T6, RP2040, ATmega328P) y su comparación a nivel hoja de datos/experiencia general.  
- **No incluye:** Benchmarks eléctricos finos (sleep, ruido analógico), pruebas RF, certificaciones ni costos de producción en volumen.

---

## 4) Requisitos

**Software**
- Editor (VS Code) y navegador.
- (Opcional) MkDocs + Material para publicar.

**Hardware (si se desea experimentar)**
- ESP32-DevKit, Blue Pill (STM32F103C8T6), Raspberry Pi Pico (RP2040), Arduino UNO (ATmega328P).

**Conocimientos previos**
- Lectura básica de **datasheets** y **pinouts**.
- Periféricos: **ADC, UART, SPI, I²C, PWM**.
- Flujo con **IDE/SDK** (Arduino IDE, ESP-IDF, Pico SDK, STM32CubeIDE).

---

## 5) Microcontroladores elegidos
- **ESP32** (Espressif)  
- **STM32F103C8T6** (STMicroelectronics)  
- **RP2040** (Raspberry Pi Pico)  
- **ATmega328P** (Arduino UNO, Microchip)  

---

## 6) Tabla comparativa

| MCU + Marca                | Periféricos                                                              | Memoria (Flash / RAM)               | Ecosistema                                | Costo aprox. (MXN) | Arquitectura                     | Velocidad de trabajo |
|---------------------------|---------------------------------------------------------------------------|-------------------------------------|-------------------------------------------|--------------------|----------------------------------|----------------------|
| ESP32 (Espressif)         | Wi-Fi, Bluetooth, ADC 12b, 2×DAC, UART, SPI, I²C, PWM, touch             | ~4 MB Flash (módulo), ~520 KB SRAM  | Arduino IDE o ESP-IDF, comunidad IoT       | 120–250            | Xtensa LX6 **dual-core** 32 bits | 240 MHz              |
| STM32F103C8T6 (ST)        | ADC 12b, UART, CAN, I²C, SPI, timers                                     | 64 KB Flash, 20 KB SRAM             | STM32CubeIDE/HAL, uso industrial           | 100–250            | ARM Cortex-M3 32 bits            | 72 MHz               |
| RP2040 (Raspberry Pi)     | 2×ADC, UART, SPI, I²C, PWM, **PIO programable**                          | 2 MB Flash, 264 KB SRAM             | SDK C/C++ y MicroPython, comunidad amplia  | 150–200            | ARM Cortex-M0+ **dual-core** 32 bits | 133 MHz           |
| ATmega328P (Microchip)    | ADC 10b, UART, SPI, I²C, timers                                          | 32 KB Flash, 2 KB SRAM              | Arduino IDE, comunidad educativa enorme    | 200–300 (Arduino UNO) | AVR 8 bits                     | 16 MHz               |

    note "Precios"
    Costos referenciales en MXN; varían por marca/placa, memoria y disponibilidad.

---

## 7) Criterios usados para el ranking
- **Conectividad integrada** (Wi-Fi/BLE) para IoT.  
- **Ecosistema/soporte** (IDE oficiales, SDKs, documentación, comunidad).  
- **Rendimiento** (arquitectura, núcleos y frecuencia).  
- **Memoria disponible** (Flash/RAM útiles).  
- **Costo** y **facilidad** de aprendizaje/puesta en marcha.

---

## 8) Ranking de microcontroladores (con justificación)

1. **ESP32**  
   Mayor cantidad de periféricos y **conectividad Wi-Fi/Bluetooth** integrada, buen rendimiento y bajo costo. Ideal para **IoT**, prototipado rápido y proyectos de mecatrónica actuales.

2. **RP2040 (Raspberry Pi Pico)**  
   Doble núcleo, **PIO** muy flexible y comunidad enorme. No trae radio integrada, pero es **versátil** y económico para control, procesamiento ligero y aprendizaje.

3. **STM32F103C8T6 (Blue Pill)**  
   Arquitectura ARM Cortex-M3 y presencia en **industria**. Excelente para aprender HAL/STM32CubeIDE. Curva de aprendizaje más alta que ESP32/Pico.

4. **ATmega328P (Arduino UNO)**  
   Limitado en memoria/velocidad, pero con la comunidad educativa más grande y **facilísimo** para empezar. Útil en prácticas introductorias y proyectos simples.

---

## 9) Conclusiones y recomendaciones

- Si necesitas **IoT** con bajo costo y rapidez → **ESP32**.  
- Si quieres **control flexible** y aprender concurrente/PIO → **RP2040**.  
- Si el enfoque es **industrial/ARM** y HAL → **STM32F103C8T6**.  
- Para **educación básica** y prototipos simples → **ATmega328P**.


