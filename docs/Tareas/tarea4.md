# 🕹️ Tarea 4 — Mini-Pong con ISR (5 LEDs)

## Nombre del código
Mini-Pong ISR (5 LEDs, 2 botones, RP2040)

## Qué debe hacer
La “pelota” es un LED que recorre 5 LEDs en línea (L1→L5→L1…) a un ritmo fijo.  
Cada botón genera una **interrupción (ISR)** y **solo cuenta** si se presiona **exactamente** cuando la pelota está en el **extremo** de ese lado: BTN_L solo en **L1**, BTN_R solo en **L5**. Si coincide, **rebota** (invierte dirección). Si no, **anota** el rival: parpadea 3 veces su LED de punto, se reinicia en **L3** y sale hacia quien anotó.  
Al encender, la pelota inicia en **L3** y **no se mueve** hasta presionar un botón; arrancará en la **dirección opuesta** al botón presionado.

## Código
~~~c
#include "pico/stdlib.h"
#include "hardware/gpio.h"

// ---------- Pines ----------
#define BTN_L  0   // Botón jugador IZQUIERDA
#define BTN_R  1   // Botón jugador DERECHA
#define PT_L   2   // LED punto IZQUIERDA
#define PT_R   3   // LED punto DERECHA

// Línea de 5 LEDs (L1..L5)
#define L1 4       // extremo izquierdo
#define L2 5
#define L3 6       // centro
#define L4 7
#define L5 8       // extremo derecho

// ---------- Parámetros ----------
static const uint32_t STEP_MS  = 120; // ritmo de la pelota
static const uint32_t BLINK_MS = 150; // parpadeo de punto (x3)

// ---------- Estado ----------
static const uint8_t LINE[5] = { L1, L2, L3, L4, L5 };
static volatile bool START = false;     // detenido en L3 hasta primer toque
static volatile int8_t dir = 0;         // -1 izq., +1 der.
static volatile uint8_t idx = 2;        // 0..4 (L3=2)
static volatile uint8_t cur_pin = L3;   // GPIO encendido actual
static volatile bool HIT_L = false;     // golpe válido justo en L1
static volatile bool HIT_R = false;     // golpe válido justo en L5

static inline void show_ball(uint8_t new_idx){
    idx = new_idx;
    cur_pin = LINE[idx];
    uint32_t mask_line = (1u<<L1)|(1u<<L2)|(1u<<L3)|(1u<<L4)|(1u<<L5);
    gpio_clr_mask(mask_line);
    gpio_set_mask(1u << cur_pin);
}

static void blink_point(uint pin){
    for(int i=0;i<3;++i){ gpio_put(pin,1); sleep_ms(BLINK_MS); gpio_put(pin,0); sleep_ms(BLINK_MS); }
}

static void reset_center_and_go(bool to_right){
    show_ball(2);         // L3
    START = true;
    dir = to_right ? +1 : -1;
}

// ISR: arranque y golpes exactos en extremos
static void isr_buttons(uint gpio, uint32_t events){
    if(!(events & GPIO_IRQ_EDGE_FALL)) return; // flanco de bajada (pull-up)
    if(!START){
        if(gpio==BTN_L){ START=true; dir=+1; } // opuesta al botón
        else if(gpio==BTN_R){ START=true; dir=-1; }
        return;
    }
    if(gpio==BTN_L && cur_pin==L1) HIT_L = true;
    if(gpio==BTN_R && cur_pin==L5) HIT_R = true;
}

int main(){
    stdio_init_all();
    // LEDs línea
    for(int i=0;i<5;++i){ gpio_init(LINE[i]); gpio_set_dir(LINE[i],GPIO_OUT); gpio_put(LINE[i],0); }
    // LEDs punto
    gpio_init(PT_L); gpio_set_dir(PT_L,GPIO_OUT); gpio_put(PT_L,0);
    gpio_init(PT_R); gpio_set_dir(PT_R,GPIO_OUT); gpio_put(PT_R,0);
    // Botones (pull-up)
    gpio_init(BTN_L); gpio_set_dir(BTN_L,GPIO_IN); gpio_pull_up(BTN_L);
    gpio_init(BTN_R); gpio_set_dir(BTN_R,GPIO_IN); gpio_pull_up(BTN_R);
    // ISR
    gpio_set_irq_enabled_with_callback(BTN_L, GPIO_IRQ_EDGE_FALL, true, &isr_buttons);
    gpio_set_irq_enabled(BTN_R, GPIO_IRQ_EDGE_FALL, true);

    show_ball(2); // L3
    START=false;

    absolute_time_t last = get_absolute_time();
    while(true){
        if(to_ms_since_boot(get_absolute_time()) - to_ms_since_boot(last) < STEP_MS){
            tight_loop_contents(); continue;
        }
        last = get_absolute_time();

        if(!START){ show_ball(2); continue; }

        int8_t next = idx + dir;
        if(next<0) next=0; if(next>4) next=4;
        show_ball(next);

        if(idx==0){ // L1
            if(HIT_L){ dir=+1; }                 // rebote
            else{ blink_point(PT_R); reset_center_and_go(true); } // punto derecha
            HIT_L=false;
        }
        if(idx==4){ // L5
            if(HIT_R){ dir=-1; }                 // rebote
            else{ blink_point(PT_L); reset_center_and_go(false);} // punto izquierda
            HIT_R=false;
        }
    }
    return 0;
}
~~~

### Esquemático de conexión XOR
![Esquemático de conexion](../img/XOR.svg)

## video
<iframe width="560" height="315"
  src="https://www.youtube.com/embed/9rNgV5DDlr4"
  title="Pong - demostración"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen style="border:0;">
</iframe>



