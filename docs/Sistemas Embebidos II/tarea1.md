# 📘 Activity 1 — FreeRTOS Tasks and Priorities

> Identification, analysis, and design of logical tasks using FreeRTOS concepts.

---

## ) Overview

- **Topic:** Task identification, priorities, and system design with FreeRTOS
- **Autor:** Carlos Ernesto Camacho González
- **Curso / Asignatura:** Sistemas Embebidos  
---

## 1) Exercise 1 — Identify Logical Tasks
| **Task** | **Trigger (Time / Event)** | **Periodic / Event-Based** |
|---------|----------------------------|----------------------------|
| Reads a temperature sensor every **50 ms** | 50 ms timer | **Periodic** |
| Sends sensor data via **Wi-Fi every 2 s** | 2 s timer | **Periodic** |
| Monitors an **emergency button** | Button press | **Event-Based** |
| Blinks a status LED at **1 Hz** | Timer (1 s) | **Periodic** |
| Stores **error messages** when failures occur | Error event | **Event-Based** |

## 2) Exercise 2 — Task Characteristics

| **Is it time-critical? (Yes / No)** | **Can it block safely? (Yes / No)** | **What happens if this task is delayed?** |
|------------------------------------|------------------------------------|-------------------------------------------|
| No | Yes | It depends on the system; however, in general, the sensor would read the information at the end. |
| No | No | If it takes a while, it would send the data, but if it gets blocked, it could affect communication. |
| Yes | No | It would cause an accident. |
| No | Yes | The LED just looks weird. |
| No | Yes | Only data is lost. |


## 3) Exercise 3 — Priority Reasoning
| **Task Name** | **Priority (H / M / L)** | **Justification** |
|--------------|--------------------------|-------------------|
| Temperature | **Medium** | It is important for system operation, but small delays are acceptable. |
| Wi-Fi | **Medium** | Communication is necessary, but short delays do not compromise safety. |
| Emergency button | **High** | It is safety-critical and must be handled immediately. |
| LED | **Low** | It is only visual feedback and not functionally critical. |
| Error messages | **Low** | Logging is useful but not urgent compared to control or safety tasks. |

## 4) Exercise 4 — Design Judgment (Trick Question)

Emergency button monitoring

Explanation (2–3 sentences):
Emergency button monitoring should not necessarily be implemented as a FreeRTOS task because it is time-critical and must react immediately. According to the document, a delay could cause an accident, so handling it as a hardware interrupt is more appropriate. Interrupts provide faster response than polling inside a task.

## 5) Exercise 5.1 — Identify Hidden Tasks


| **Hidden Task** | **Trigger (Time / Event)** | **Why it should be a Task** |
|-----------------|----------------------------|-----------------------------|
| Read temperature sensor | Repeated in the loop (time-based) | It needs to run regularly, and its timing should not depend on other code. |
| Emergency button check | Button press (event) | It is very important and must react immediately to avoid accidents. |
| Send data over Wi-Fi | Every 2000 ms (time-based) | This operation can block for a long time and delay other actions. |
| Status LED blinking | Every 1 second (1 Hz) | The LED needs stable timing to blink correctly. |
| Loop delay | Every loop iteration | The delay controls execution speed but blocks the whole program. |

## 5.2) Exercise 5.2 — Blocking Analysis

**1) Which function can block the CPU?**  
- `send_data_over_wifi()` (it may block for 100–300 ms)

**2) What other behaviors are affected while it blocks?**  
- The program cannot check the emergency button during that time.  
- The LED blink becomes irregular (not a clean 1 Hz).  
- The temperature reading becomes delayed and not consistent.

**3) Which hidden task is most at risk because of this blocking?**  
- Emergency button handling, because it must react immediately and blocking can delay the shutdown.

## 5.3) Exercise 5.3 — RTOS Refactoring Thought Experiment

### Which hidden task(s) should become FreeRTOS tasks?
The temperature reading should be a task because it needs to run periodically and with stable timing.  
The Wi-Fi data transmission should also be a task because it can block the CPU for a long time.  
The LED blinking can be a low-priority task because it runs periodically and is not critical.

### Which behavior(s) should be handled using an interrupt?
The emergency button should be handled using an interrupt because it must react immediately.  
Using an interrupt avoids delays caused by other tasks.

### Which task should have the highest priority, and why?
The emergency or safety task should have the highest priority because it is safety-critical.  
If it is delayed, it could cause an accident.
