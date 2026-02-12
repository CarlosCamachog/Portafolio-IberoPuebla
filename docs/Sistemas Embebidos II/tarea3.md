# Session 3 — FreeRTOS Tasks, Queues, and Mutex

Implement multiple FreeRTOS tasks in ESP-IDF, including heartbeat, alive logs, queue communication, mutex protection, and error logging.

---

## 1) Activity Goals

- Implement Task 1 (Heartbeat LED)
- Implement Task 2 (Alive task every 2 seconds)
- Implement Task 3 (Queue Struct Send)
- Implement Task 4 (Queue Struct Receive)
- Implement Task 5 (Mutex protected button reading)
- Implement Task 6 (Second mutex protected button task)
- Implement Task 7 (Error logging system)

---

## 2) Materials & Setup

### BOM (Bill of Materials)

| # | Item | Qty | Link/Source | Cost (MXN) | Notes |
|---|------|-----|------------|------------|------|
| 1 | ESP32 Board | 1 | Local Store / Amazon / MercadoLibre | 365 | Main development board |
| 2 | LED | 1 | Local electronics store | 3 | Status LED connected to GPIO 8 |
| 3 | Push Button | 1 | Local electronics store / Amazon / MercadoLibre | 1.70-3.60 | Button connected to GPIO ___ |

### Tools/Software

- **Framework:** ESP-IDF + FreeRTOS  

---

## 3) Procedure (what you did)

1. Created FreeRTOS tasks for heartbeat and alive monitoring.
2. Implemented queue producer/consumer communication with structs.
3. Implemented mutex protection for shared resources.
4. Implemented error logging system.
5. Verified correct execution using serial monitor output and LED behavior.

---

# 4) Evidence (Console, Photos, Videos)

## Task 1 — Heartbeat LED



---

## Task 2 — Alive Task



---

## Task 3 — Queue Struct Send (Producer)



---

## Task 4 — Queue Struct Receive (Consumer)



---

## Task 5 — Mutex Button Task 1



---

## Task 6 — Mutex Button Task 2



---

## Task 7 — Error Logging



---

# 5) Analysis

### Expected vs Observed Behavior


### Issues Found


### Fixes / Improvements


---

# 6) Code

## Task 1 — Heartbeat LED Task

~~~c
// INSERT TASK 1 CODE HERE
~~~

---

## Task 2 — Alive Task (2 seconds)

~~~c
// INSERT TASK 2 CODE HERE
~~~

---

## Task 3 — Queue Struct Send (Producer)

~~~c
// INSERT TASK 3 CODE HERE
~~~

---

## Task 4 — Queue Struct Receive (Consumer)

~~~c
// INSERT TASK 4 CODE HERE
~~~

---

## Task 5 — Mutex Protected Button Task 1

~~~c
// INSERT TASK 5 CODE HERE
~~~

---

## Task 6 — Mutex Protected Button Task 2

~~~c
// INSERT TASK 6 CODE HERE
~~~

---

## Task 7 — Error Logging Task

~~~c
// INSERT TASK 7 CODE HERE
~~~

---

# 7) Files & Media

- Firmware file: `main/main.c`
- Images folder: `docs/img/`
- Videos: [INSERT YOUTUBE LINKS HERE]
