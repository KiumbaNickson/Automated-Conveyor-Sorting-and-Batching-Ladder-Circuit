# Automated Conveyor Checkweigher and Batching System

An IEC 61131-3 compliant PLC program written in Ladder Diagram (LD) for an automated conveyor system. The system performs real-time weight checks, pneumatically rejects out-of-spec products, and tracks batch quantities with automated reset cycles.

---

## System Architecture & Functionality

* **Motor Latch & Safety Circuit:** Uses a standard seal-in contact logic for motor control. Requires `START_PB` (%IX0.0) activation along with closed `STOP_PB` (%IX0.1), `ESTOP_OK` (%IX0.2), and `OVRLOAD_OK` (%IX0.3) safety interlocks to drive `MOTOR_RUN` (%QX0.0) and status light `LAMP_RUN` (%QX0.1).
* **Analog Weight Verification:** Evaluates `ACTUAL_WEIGHT` against `MIN_WEIGHT` (49.95 kg) and `MAX_WEIGHT` (50.10 kg) using `GE` and `LE` comparison blocks. A valid evaluation sets the `WEIGHT_OK` flag.
* **Rejection Timing:** Triggers `SR0` latch when `QUALITY_SENSOR` or `WEIGHT_OK` flags fail during product presence. Utilizes a 3-second delay (`TON1`) to position the defective item at the rejector, energizes `REJECT_SELENOID` (%QX0.2) for a 2-second pulse (`TON2`), and resets the sequence.
* **Batch Counter & Auto Reset:** Uses `CTU1` to count passing products that satisfy all quality and weight checks. Upon reaching a preset value of 5 items, `BATCH_COMPLETE_LAMP` illuminates. A 5-second timer (`TON3`) automatically clears the counter to initiate the next batch cycle.

---

## I/O Mapping

| Signal Name | Address | Type | Description |
| :--- | :--- | :--- | :--- |
| `START_PB` | %IX0.0 | BOOL | Normally Open Start Push Button |
| `STOP_PB` | %IX0.1 | BOOL | Normally Closed Stop Push Button |
| `ESTOP_OK` | %IX0.2 | BOOL | Emergency Stop Safety Line (Active High) |
| `OVRLOAD_OK` | %IX0.3 | BOOL | Thermal Overload Protection Line (Active High) |
| `PRODUCT_SENSOR` | %IX0.4 | BOOL | Photoelectric sensor for product arrival |
| `MOTOR_RUN` | %QX0.0 | BOOL | Conveyor Motor Contactor Coil |
| `LAMP_RUN` | %QX0.1 | BOOL | Run Status Indicator Lamp |
| `REJECT_SELENOID`| %QX0.2 | BOOL | Pneumatic Actuator Output for Rejections |

---

## Parameters & Settings

| Parameter | Data Type | Default Value | Usage |
| :--- | :--- | :--- | :--- |
| `SCALE_FACTOR` | REAL | 0.00152588 | Linear scaling factor ($50.0\text{ kg} / 32767.0\text{ counts}$) |
| `MIN_WEIGHT` | REAL | 49.95 | Minimum weight limit threshold (kg) |
| `MAX_WEIGHT` | REAL | 50.10 | Maximum weight limit threshold (kg) |
| `TON1.PT` | TIME | T#3s | Conveyor transport delay to reject station |
| `TON2.PT` | TIME | T#2s | Pneumatic reject solenoid extension duration |
| `CTU1.PV` | INT | 5 | Target batch quantity count limit |
| `TON3.PT` | TIME | T#5s | Dwell duration before batch counter auto-reset |

---

## Task Execution Profile

* **Task Name:** `task1`
* **Execution Interval:** 20 ms
* **Priority Level:** 1
* **Target Hardware:** OpenPLC Runtime Engine / PLCOpen XML Compatible Controllers
