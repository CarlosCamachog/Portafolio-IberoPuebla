# 📘 Tarea 9 — ADC y Control con Señales Analógicas
---

#  Tarea 9.1 — ADC Luxómetro con LDR (0–100%)
---

## 1) Resumen

- **Nombre del subproyecto:** Luxómetro con LDR y ADC  
- **Autor:** Carlos Ernesto Camacho González  
- **Curso / Asignatura:** Sistemas Embebidos I  
- **Fecha:** 16/09/2025  
- **Descripción breve:**  
  - Se usa un **LDR** (fotoresistencia) en un **divisor de voltaje** conectado al **ADC** de la Pico 2.  
  - El microcontrolador lee el valor analógico y lo convierte a un valor de **0 a 100 % de luminosidad**.  
  - Se aplica una **calibración sencilla** con valores mínimo y máximo medidos en el laboratorio.

---

## 2) Objetivos

- Leer una señal analógica usando el **ADC** de la Raspberry Pi Pico 2.  
- Entender el uso de un **LDR** como sensor de luz en un **divisor de voltaje**.  
- Mapear la lectura del ADC (0–4095) a una escala **0–100 %** ajustada al rango real del sensor.  

---

## 3) Circuito

### Código
~~~c
#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/adc.h"
 
// Configurar el canal ADC a usar
#define ADC_INPUT 0 // canal 0
 
#define ADC_MIN 850    // valor cuando tapas la LDR
#define ADC_MAX 3100   // valor con luz máxima
 
int main() {
    stdio_init_all();
    adc_init();
    // Configura el pin GPIO correspondiente como entrada ADC
    adc_gpio_init(26); // GPIO26 suele mapear a ADC0 en Pico 2
    // Seleccionar canal
    adc_select_input(ADC_INPUT);
 
    while (true) {
        uint16_t adc = adc_read(); // 12 bits alineados a 0..4095
 
        if (adc < ADC_MIN) adc = ADC_MIN;
        if (adc > ADC_MAX) adc = ADC_MAX;
 
        // Calcular porcentaje de luz 0–100
        float luz = (adc - ADC_MIN) * 100.0f / (ADC_MAX - ADC_MIN);
 
        printf("ADC: %u\tLuz: %.1f%%\n", adc, luz);
        sleep_ms(200);
    }
}
~~~

## 4) Esquemático de conexión

![Esquemático — Luxumetro + Pico 2](../img/luxumetro.jpeg)   
*Figura 1.*

---

## 5) Video de demostracion

<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/VtgWAUczVmI"
  title="Control motor DC - demo"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen
  style="border:0;">
</iframe>


#  Tarea 9.2 — Servo con Potenciómetro y ADC
---

## 1) Resumen
- **Autor:** Carlos Ernesto Camacho González  
- **Materia:** Sistemas Embebidos I  
- **Fecha:** 16/09/2025  
- **Descripción:**  
  Control de un **servo motor** con un **potenciómetro** usando el **ADC** de la Raspberry Pi Pico 2.  
  El giro del potenciómetro controla el ángulo del servo entre **0° y 180°**.

## 2) Circuito

### Código
~~~c
#include <iostream>
#include "pico/stdlib.h"
#include "hardware/adc.h"
#include "hardware/pwm.h"
 
using namespace std;
 
#define SERVO_PIN 0    
#define POT_PIN 26      
 
int main() {
    stdio_init_all();
 
   //inicializar ADC
    adc_init();
    adc_gpio_init(POT_PIN);
    adc_select_input(0);
    adc_set_clkdiv(479.0f);          
    adc_fifo_setup(true, false, 1, false, false);  
    adc_fifo_drain();                
    adc_run(true);                    
 
   //inicializar PWM para el servo
    gpio_set_function(SERVO_PIN, GPIO_FUNC_PWM);
    uint slice_num = pwm_gpio_to_slice_num(SERVO_PIN); //aplicar configuracion al slice
 
    pwm_set_clkdiv(slice_num, 64.0f);
    pwm_set_wrap(slice_num, 39062);  
//ajustar frecuencia del pwm
    pwm_set_enabled(slice_num, true);
 
    while (true) {
 
        if (adc_fifo_get_level() > 0) { //para ver si hay algo dentro del fifo
            uint16_t valor_adc = adc_fifo_get();
 
 
            float duty = 0.025f + (valor_adc / 4095.0f) * 0.1f;
 
            pwm_set_gpio_level(SERVO_PIN, duty * 39062);
 
 
        }
 
        sleep_ms(20);
    }
 
    return 0;
}
~~~

## 3) Esquemático de conexión

![Esquemático — Servo + Pico 2](../img/motordc.jpeg)   
*Figura 2.*

---

## 4) Video de demostracion

<iframe
  width="560"
  height="315"
  src="https://www.youtube.com/embed/L8Vw-Pges6I"
  title="Servo controlado por ADC - Pico 2"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen
  style="border:0;">
</iframe>
