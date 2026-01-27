# 📘 Activity 1 — FreeRTOS Tasks and Priorities

> Identification, analysis, and design of logical tasks using FreeRTOS concepts.

---

## 1) Overview

- **Topic:** Task identification, priorities, and system design with FreeRTOS
- **Autor:** Carlos Ernesto Camacho González
- **Curso / Asignatura:** Sistemas Embebidos  
---

## 2) Exercise 1 — Identify Logical Tasks
| **Task** | **Trigger (Time / Event)** | **Periodic / Event-Based** |
|---------|----------------------------|----------------------------|
| Reads a temperature sensor every **50 ms** | 50 ms timer | **Periodic** |
| Sends sensor data via **Wi-Fi every 2 s** | 2 s timer | **Periodic** |
| Monitors an **emergency button** | Button press | **Event-Based** |
| Blinks a status LED at **1 Hz** | Timer (1 s) | **Periodic** |
| Stores **error messages** when failures occur | Error event | **Event-Based** |

## 3) Exercise 2 — Task Characteristics

| **Is it time-critical? (Yes / No)** | **Can it block safely? (Yes / No)** | **What happens if this task is delayed?** |
|------------------------------------|------------------------------------|-------------------------------------------|
| No | Yes | It depends on the system; however, in general, the sensor would read the information at the end. |
| No | No | If it takes a while, it would send the data, but if it gets blocked, it could affect communication. |
| Yes | No | It would cause an accident. |
| No | Yes | The LED just looks weird. |
| No | Yes | Only data is lost. |


## 4) Exercise 3 — Priority Reasoning
| **Task Name** | **Priority (H / M / L)** | **Justification** |
|--------------|--------------------------|-------------------|
| Temperature | **Medium** | It is important for system operation, but small delays are acceptable. |
| Wi-Fi | **Medium** | Communication is necessary, but short delays do not compromise safety. |
| Emergency button | **High** | It is safety-critical and must be handled immediately. |
| LED | **Low** | It is only visual feedback and not functionally critical. |
| Error messages | **Low** | Logging is useful but not urgent compared to control or safety tasks. |

## 5) Exercise 5 — Suggested Task Configuration
| **Component** | **Type** | **Period / Trigger** | **Priority** | **Description** |
|--------------|----------|---------------------|--------------|-----------------|
| Emergency Button | Interrupt (ISR) | Button press | — | Triggers ISR, no heavy processing. |
| SafetyTask | Task | Notification | **High** | Executes emergency shutdown procedures. |
| TempTask | Task | 50 ms | **Medium** | Periodic sensor reading. |
| WifiTask | Task | 2 s | **Medium** | Sends data without blocking other tasks. |
| LedTask | Task | 1 s (1 Hz) | **Low** | Visual status indication. |
| LogTask | Task | Error event | **Low** | Stores logs asynchronously. |
