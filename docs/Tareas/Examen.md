# 📘 Examen — Simón Dice (4 colores) · Raspberry Pi Pico 2

> Juego de **memoria** con 4 LEDs y 4 botones en Raspberry Pi Pico 2 (**RP2350**), con **secuencia aleatoria** que crece por ronda, **tiempo límite** por entrada y **fallos** por botón incorrecto, por **dos pulsaciones simultáneas** o por **timeout**.  

---

## 1) Resumen

- **Nombre del proyecto:** Simón Dice (4 colores) – Pico 2  
- **Autor:** Carlos Ernesto Camacho González  
- **Curso / Asignatura:** Sistemas Embebidos  
- **Fecha:** 22/09/2025  
- **Descripción breve:** Implementación de un juego “Simón Dice” con **reproducción de secuencia**, **entrada del jugador** con **TL = 5 s + longitud de la ronda**, y **antirrebote por flanco**. Se usa el **timer de sistema** con **ALARM0/ALARM1** para marcar la **reproducción** y el **timeout de entrada**, y **GPIO IRQ** para leer botones sin *polling*.

!!! tip "Consejo"
    Mantén la lógica **sin bucles de *polling***: usa **interrupciones** del timer y de GPIO para conseguir respuesta inmediata y *timing* preciso.

---

## 2) Objetivos

- **General:** Desarrollar un juego completo con **interrupciones (timer + GPIO)**, **antirrebote por flancos** y **manejo de estados**.
- **Específicos:**
  - Reproducir la **secuencia actual** LED por LED con temporización fija.
  - Aceptar la **entrada del jugador** bajo un **tiempo límite** dependiente de la ronda.
  - Detectar **fallos** por botón equivocado, **doble pulsación** simultánea o **exceso de tiempo**.
  - Asegurar **aleatoriedad** de la secuencia en cada ejecución.

---

## 3) Alcance y Exclusiones

- **Incluye:**  
  - GPIO como salidas (LED0..LED3) y entradas con **pull-up** (BTN0..BTN3).  
  - **Timer HW** en modo µs con **ALARM0** (reproducción) y **ALARM1** (timeout).  
  - **IRQ de GPIO** con **antirrebote por flanco** (30 ms).  
  - Detección de **pulsación múltiple** (>1 botón) como fallo.
- **No incluye:**  
  - **Display** de 7 segmentos (omitido).  
  - Almacenamiento de puntaje en memoria o audio.

---

## 4) Requisitos

**Hardware**
- Raspberry Pi Pico 2 (RP2350).  
- 4 LEDs con resistencias de **220–330 Ω** conectados a **GPIO 0..3**.  
- 4 botones tipo *push* conectados a **GPIO 4..7** con **pull-up interno** (a GND al presionar).  
- Protoboard y cables.

**Software**
- Pico SDK configurado (RP2350).  
- CMake + Ninja o tu entorno preferido.

**Conocimientos previos**
- **Interrupciones** (timer y GPIO).  
- Manejo de **máscaras** y **GPIO**.  
- Conceptos de **debounce** y **deadline**.

---

## 5) Reglas del juego (aplicadas)

- **Inicio/Reset:** al energizar, LEDs apagados; **cualquier botón** inicia la **Ronda 1**.  
- **Reproducción:** se **muestra** la secuencia actual (LED por LED) a **0.5 s** por paso.  
- **Entrada:** el jugador repite la secuencia **completa** dentro de **TL = 5 s + ronda**.  
- **Fallo (Game Over):**  
  - Botón **incorrecto** en cualquier paso.  
  - **Dos o más** botones presionados **a la vez** durante la fase de entrada.  
  - **Exceder TL** (timeout).  
- **Progresión:** si acierta, **ronda++**, se **añade 1 color aleatorio** y se repite.  
- **Fin:** al fallar o al completar **NMAX_RONDAS** (configurada en el código como 8).

---

## 6) Mapa de pines y conexión

| Señal | GPIO | Tipo | Conexión recomendada |
|------:|:----:|:----:|----------------------|
| LED0  |  0   | OUT  | LED→R(220–330 Ω)→GPIO 0; cátodo a GND |
| LED1  |  1   | OUT  | LED→R→GPIO 1; cátodo a GND |
| LED2  |  2   | OUT  | LED→R→GPIO 2; cátodo a GND |
| LED3  |  3   | OUT  | LED→R→GPIO 3; cátodo a GND |
| BTN0  |  4   | IN   | Botón a **GND**; habilita **pull-up interno** |
| BTN1  |  5   | IN   | Botón a **GND**; habilita **pull-up interno** |
| BTN2  |  6   | IN   | Botón a **GND**; habilita **pull-up interno** |
| BTN3  |  7   | IN   | Botón a **GND**; habilita **pull-up interno** |

> Con esta topología, el **nivel activo es bajo (0)** al presionar.

---

## 7) Conceptos clave (rápido)

- **Debounce (antirrebote):** Filtrado temporal (p. ej., **30 ms**) para ignorar *rebotes* al cambiar de estado el botón. Aquí se hace **por flanco**, validando solo transiciones **1→0** estables.  
- **Deadline / Timeout (TL):** Límite de tiempo para completar la entrada. Se programa con una **alarma del timer**: si llega y la tarea no se completó, **falla**.  
- **Edge IRQ:** Interrupción por **flanco** (subida/bajada); evita *polling* y mejora la latencia.  
- **Secuencia aleatoria:** `rand()%4` con **semilla** desde `time_us_32()` desfasada.

---

## 8) Código completo (C · Pico SDK)

~~~c
#include <stdio.h>
#include <stdlib.h>
#include "pico/stdlib.h"
#include "pico/time.h"
#include "hardware/gpio.h"
#include "hardware/irq.h"
#include "hardware/structs/timer.h"

// LEDs y botones
#define LED0 0
#define LED1 1
#define LED2 2
#define LED3 3
#define BTN0 4
#define BTN1 5
#define BTN2 6
#define BTN3 7

#define NMAX_RONDAS 8

// Estados del juego
volatile bool START = false;
volatile bool START_SECUENCIA = false;
volatile bool START_ESPERA = false;

static volatile uint8_t SECUENCIA[NMAX_RONDAS];
static volatile uint8_t RONDA = 0;
static volatile uint8_t RONDA_ACTUAL = 0;
static volatile uint8_t CONTADOR = 0;

// Alarmas
#define ALARMA_SECUENCIA 0
#define ALARMA_ESPERA    1
#define ALARMA_SECUENCIA_IRQ timer_hardware_alarm_get_irq_num(timer_hw, ALARMA_SECUENCIA)
#define ALARMA_ESPERA_IRQ    timer_hardware_alarm_get_irq_num(timer_hw, ALARMA_ESPERA)

static volatile uint32_t NEXT_AS_US, NEXT_AE_US;
static const uint32_t INTERVALO_AS_US = 500000u;  // 0.5 s (ritmo de reproducción)
static const uint32_t BASE_AE_US      = 5000000u; // 5 s base del TL

// Antirrebote por flanco
static const uint32_t DEBOUNCE_US = 30000u; // 30 ms
static volatile uint32_t LAST_EDGE_US[4] = {0,0,0,0};
static volatile uint8_t  LAST_STATE[4]   = {1,1,1,1}; // 1 = suelto (pull-up), 0 = presionado

// Prototipos
static void GENERAR_ELEMENTO(void);
static void BLINK_SECUENCIA(void);
static void SISTEMA_DE_ESPERA(void);
static void START_AS(void);
static void STOP_AS(void);
static void START_AE(void);
static void STOP_AE(void);
static void BTN_ISR(uint gpio, uint32_t events); // ISR de botones
static inline uint8_t BOTONES_PRESIONADOS(void);
static inline void GAME_OVER(void);

int main(void) {
    stdio_init_all();

    // LEDs
    for (uint8_t i = LED0; i <= LED3; i++) {
        gpio_init(i);
        gpio_set_dir(i, GPIO_OUT);
        gpio_put(i, 0);
    }
    // Botones (pull-up interno)
    for (uint8_t i = BTN0; i <= BTN3; i++) {
        gpio_init(i);
        gpio_set_dir(i, GPIO_IN);
        gpio_pull_up(i);
    }

    // Semilla random
    srand((time_us_32() * 7) ^ (time_us_32() >> 3));

    // Timer HW
    timer_hw->source = 0u;
    hw_clear_bits(&timer_hw->intr, (1u << ALARMA_SECUENCIA) | (1u << ALARMA_ESPERA));
    irq_set_exclusive_handler(ALARMA_SECUENCIA_IRQ, BLINK_SECUENCIA);
    irq_set_exclusive_handler(ALARMA_ESPERA_IRQ,    SISTEMA_DE_ESPERA);

    // IRQ de botones: flancos de subida y bajada
    gpio_set_irq_enabled_with_callback(BTN0, GPIO_IRQ_EDGE_FALL | GPIO_IRQ_EDGE_RISE, true, &BTN_ISR);
    gpio_set_irq_enabled(BTN1, GPIO_IRQ_EDGE_FALL | GPIO_IRQ_EDGE_RISE, true);
    gpio_set_irq_enabled(BTN2, GPIO_IRQ_EDGE_FALL | GPIO_IRQ_EDGE_RISE, true);
    gpio_set_irq_enabled(BTN3, GPIO_IRQ_EDGE_FALL | GPIO_IRQ_EDGE_RISE, true);

    // Bucle ocioso (sin polling)
    while (true) {
        tight_loop_contents(); // o __wfi()
    }
}

/*** Helpers ***/
static void GENERAR_ELEMENTO(void){
    if (RONDA <= NMAX_RONDAS) SECUENCIA[RONDA - 1] = rand() % 4;
}

static inline void GAME_OVER(void){
    for (uint8_t j = 0; j < 4; j++) gpio_put(j, 0);
    STOP_AE(); STOP_AS();
    START = false;
    START_ESPERA = false;
    START_SECUENCIA = false;
}

static inline uint8_t BOTONES_PRESIONADOS(void){
    // Cuenta cuántos están a nivel bajo (activo) en este instante
    uint8_t c = 0;
    c += (gpio_get(BTN0) == 0);
    c += (gpio_get(BTN1) == 0);
    c += (gpio_get(BTN2) == 0);
    c += (gpio_get(BTN3) == 0);
    return c;
}

/*** ISR secuencia (reproducción) ***/
static void BLINK_SECUENCIA(void){
    hw_clear_bits(&timer_hw->intr, 1u << ALARMA_SECUENCIA);
    if (!START_SECUENCIA) { STOP_AS(); return; }

    static bool ENCENDIDO = false;
    if (RONDA == 0) { STOP_AS(); return; }

    uint8_t led = SECUENCIA[CONTADOR];
    if (!ENCENDIDO) {
        gpio_put(led, 1);
        ENCENDIDO = true;
    } else {
        gpio_put(led, 0);
        ENCENDIDO = false;
        CONTADOR++;
    }

    NEXT_AS_US += INTERVALO_AS_US;
    timer_hw->alarm[ALARMA_SECUENCIA] = NEXT_AS_US;

    if (CONTADOR >= RONDA && !ENCENDIDO) {
        CONTADOR = 0;
        START_SECUENCIA = false;
        START_ESPERA = true;
        STOP_AS();
        START_AE(); // TL = 5 s + RONDA (segundos)
    }
}

/*** ISR espera límite (timeout de entrada) ***/
static void SISTEMA_DE_ESPERA(void){
    hw_clear_bits(&timer_hw->intr, 1u << ALARMA_ESPERA);
    if (START_ESPERA) {
        // Tiempo agotado => fallo
        GAME_OVER();
    }
}

/*** Alarmas ***/
static void START_AS(void){
    uint32_t now = timer_hw->timerawl;
    NEXT_AS_US = now + INTERVALO_AS_US;
    timer_hw->alarm[ALARMA_SECUENCIA] = NEXT_AS_US;
    hw_clear_bits(&timer_hw->intr, 1u << ALARMA_SECUENCIA);
    hw_set_bits(&timer_hw->inte, 1u << ALARMA_SECUENCIA);
    irq_set_enabled(ALARMA_SECUENCIA_IRQ, true);
}
static void STOP_AS(void){
    hw_clear_bits(&timer_hw->inte, 1u << ALARMA_SECUENCIA);
    irq_set_enabled(ALARMA_SECUENCIA_IRQ, false);
}
static void START_AE(void){
    uint32_t now = timer_hw->timerawl;
    NEXT_AE_US = now + BASE_AE_US + ((uint32_t)RONDA) * 1000000u; // TL = 5s + RONDA
    timer_hw->alarm[ALARMA_ESPERA] = NEXT_AE_US;
    hw_clear_bits(&timer_hw->intr, 1u << ALARMA_ESPERA);
    hw_set_bits(&timer_hw->inte, 1u << ALARMA_ESPERA);
    irq_set_enabled(ALARMA_ESPERA_IRQ, true);
}
static void STOP_AE(void){
    hw_clear_bits(&timer_hw->inte, 1u << ALARMA_ESPERA);
    irq_set_enabled(ALARMA_ESPERA_IRQ, false);
}

/*** ISR de BOTONES (GPIO) con antirrebote por flanco
     + detección de pulsación múltiple (fail) ***/
static void BTN_ISR(uint gpio, uint32_t events) {
    uint32_t now = time_us_32();
    int i = (int)gpio - BTN0; // BTN0..BTN3 => 0..3
    if (i < 0 || i > 3) return;

    // Nivel actual (pull-up => 1 = suelto, 0 = presionado)
    int level = gpio_get(gpio) ? 1 : 0;

    // Flanco de subida: solo cierra ciclo (no hay lógica de juego)
    if (events & GPIO_IRQ_EDGE_RISE) {
        if ((now - LAST_EDGE_US[i]) >= DEBOUNCE_US) {
            LAST_STATE[i]   = 1;   // estable en alto (suelto)
            LAST_EDGE_US[i] = now;
        }
        return;
    }

    // Flanco de bajada: validar debounce + transición 1->0 real
    if (events & GPIO_IRQ_EDGE_FALL) {
        if ((now - LAST_EDGE_US[i]) < DEBOUNCE_US) return;      // rebote temporal
        if (!(LAST_STATE[i] == 1 && level == 0)) return;        // no es 1->0 estable
        LAST_STATE[i]   = 0;
        LAST_EDGE_US[i] = now;

        // --- REGLA NUEVA: si en fase de entrada hay >1 botón presionado => fallo ---
        if (START_ESPERA) {
            if (BOTONES_PRESIONADOS() > 1) {
                GAME_OVER();
                return;
            }
        }

        // --- LÓGICA DE JUEGO (idéntica al diseño original) ---
        if (!START) {
            // iniciar juego
            START = true;
            RONDA = 1;
            RONDA_ACTUAL = 0;
            CONTADOR = 0;
            GENERAR_ELEMENTO();
            START_SECUENCIA = true;
            START_AS();
        } else if (START_ESPERA) {
            if ((uint8_t)i == SECUENCIA[RONDA_ACTUAL]) {
                RONDA_ACTUAL++;
                if (RONDA_ACTUAL >= RONDA) {
                    if (RONDA >= NMAX_RONDAS) {
                        // Ganó
                        for (uint8_t j = 0; j < 4; j++) gpio_put(j, 1);
                        START = false;
                        START_ESPERA = false;
                        START_SECUENCIA = false;
                        STOP_AS();
                        STOP_AE();
                    } else {
                        // siguiente ronda
                        RONDA_ACTUAL = 0;
                        RONDA++;
                        GENERAR_ELEMENTO();
                        START_SECUENCIA = true;
                        START_ESPERA = false;
                        CONTADOR = 0;
                        STOP_AE();
                        START_AS();
                    }
                }
            } else {
                // error (botón incorrecto)
                GAME_OVER();
            }
        }
    }
}
~~~
