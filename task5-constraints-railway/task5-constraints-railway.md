# Task 5: Constraints, Formalisation & Violations (Railway Crossing)

A **constraint** is a rule that must be true **at every moment**. Each one below is written so that a **test or a monitor can decide TRUE or FALSE from values it can observe** (sensor values, states, timers, flags).

---

## 0. Variables used (what the monitor observes)

| Variable | Meaning | Observed from |
|---|---|---|
| `Train_Present` | A train is inside the crossing zone | Train-detection sensors (ARLCCS) |
| `Train_Approaching` | A train is within detection distance and moving towards the crossing | Train-detection sensors |
| `Barrier_Open` | The barrier is fully up | Barrier position sensor |
| `Barrier_Opening` | The command to open the barrier has been issued | ARLCCS controller |
| `Crossing_Clear` | Sensors confirm nobody and nothing is inside the crossing zone | ARLCCS sensors |
| `Warning_On`, `Alarm_On` | Warning lights are on, audible alarm is on | ARLCCS outputs |
| `Train_Sensor_OK`, `Barrier_OK` | Self-test of the sensor or barrier passes | ARLCCS safety-monitoring unit |
| `ARLCCS_Link_OK` | The robot received a fresh ARLCCS message in the last 2 seconds | Robot communication unit |
| `Readings_Conflict` | Two sensors disagree, or a reading is outside physical limits | Safety-monitoring unit |
| `Operator_Alerted`, `Event_Logged` | An alert was sent to the operator; a log entry was written | Alert unit, log |
| `Robot_Entering_Crossing`, `Robot_In_Crossing`, `Robot_Stopped` | Robot is crossing the stop line, is inside the zone, has speed 0 | Robot position and speed |
| `Robot_State`, `Prev_State` | Current and previous robot state (S-01 to S-07) | Robot state model |
| `Robot_Moving` | `Robot_State` is NAVIGATING, AVOIDING_OBSTACLE or RETURNING | Robot state model |
| `Door_Open`, `Clinic_Confirmed` | Chamber door sensor; clinic staff identity confirmed | Chamber, handover unit |
| `MC_Link_OK`, `Link_Down_Time` | Link to Mission Control works; seconds without link | Robot communication unit |
| `Chamber_Temp`, `Battery_Pct`, `Time_In_Condition` | Chamber temperature (°C), battery percent, seconds the condition has lasted | Sensors, timer |
| `Batch_Removed`, `Chamber_Safe`, `Protection_Active` | Batch taken out; values in range and not suspended; protection response on | Chamber |
| `Command_Received`, `Command_Accepted`, `Authenticated`, `Role_Authorised`, `Command_Valid` | Command flow flags | Mission Control software |
| `State_Changed`, `Alert_Raised` | A state change happened; an alert was raised | Robot, chamber |
| `Chamber_Resumed`, `All_Sensors_OK`, `Temp_In_Range`, `Hum_In_Range` | Chamber returned to conservation; sensor and range checks (from the latest reading) | Chamber |
| `Correction_Marked_Done` | The system recorded a correction as finished | Chamber |
| `Power_Lost`, `On_Emergency_Power`, `Incident_Recorded`, `Safe_Shutdown` | Main power missing; running on emergency source; incident saved; shutdown completed | Chamber power unit |

Symbols: `∧` = AND, `∨` = OR, `¬` = NOT, `→` = IF-THEN.
A rule `A → B` is **violated only when A is TRUE and B is FALSE**.

---

## 1. Constraints in simple English (20)

| ID | Constraint (simple English) | Why it is necessary |
|---|---|---|
| C-01 | If a train is in the crossing, the barrier must not be open. | An open barrier lets people or robots drive into a train. |
| C-02 | The robot may enter the crossing only if the barrier is open and no train is approaching. | A robot on the track when a train comes can be destroyed or cause a crash. |
| C-03 | The barrier may open only when the crossing is confirmed clear and no train is present. | Opening early lets vehicles in while the train is still passing. |
| C-04 | When a train is approaching, the warning lights and alarm must be on. | People and robots must be warned before the barrier closes. |
| C-05 | If the train-detection sensor fails, the barrier must not be open and the operator must be alerted. | A blind system cannot know if a train comes, so it must assume danger. |
| C-06 | If the barrier has failed, the robot must not enter and the operator must be alerted. | A broken barrier gives no protection. |
| C-07 | If the robot has no fresh ARLCCS message, it must not enter the crossing. | Old or missing data can hide a train. |
| C-08 | If sensor readings conflict, the barrier must stay not open and the conflict must be logged. | Wrong readings may hide a train, so the safe choice is "closed". |
| C-09 | A robot inside the crossing must never be stopped. | A stopped robot on the track is the worst case. |
| C-10 | While DELIVERING, the chamber door may be open only if clinic staff are confirmed. | An unconfirmed opening can lose the medicine or hand it to the wrong person. |
| C-11 | While the robot is moving, the chamber door must be closed. | An open door while moving ruins the temperature and can lose the batch. |
| C-12 | The robot may be DELIVERING only if its previous state was NAVIGATING. | Stops delivery from IDLE or from AVOIDING_OBSTACLE. |
| C-13 | If the robot is NAVIGATING and the link to Mission Control has been down for more than 30 seconds, it must be in LINK_LOST. | A robot without supervision must stop at a safe place. |
| C-14 | If chamber temperature is above 8 °C, or battery is below 10%, for 3 seconds, the robot must be in SAFE_MODE. | Matches CR-01; protects the medicine and the robot. |
| C-15 | A batch may be removed only if the chamber is safe and no protection response is active. | Removing a batch during protection could hide a spoiled batch or cause harm. |
| C-16 | A command may be accepted only if the operator is authenticated, role-authorised, and the command is valid. | Matches CR-03 and FR-02. |
| C-17 | Every received command, state change and alert must be logged. | The log is the proof of what happened (FR-10). |
| C-18 | The chamber may resume after a door opening only if the door is closed, all sensors work and the values are in range. | Matches FR-05; stops resuming with unknown conditions. |
| C-19 | A correction may be marked as done only if the latest sensor readings are in range. | Matches FR-06; stops "assumed" corrections. |
| C-20 | If main power is lost, the chamber must be on emergency power, or else the incident must be recorded and a safe shutdown done. | Matches SR-04. |

**Required links:** the two robot-chamber constraints are **C-10** and **C-11**. The sensor-failure or communication-loss constraints are **C-05, C-07 and C-13**.

---

## 2. Formal logic

| ID | Formal statement |
|---|---|
| C-01 | `Train_Present → ¬Barrier_Open` |
| C-02 | `Robot_Entering_Crossing → (Barrier_Open ∧ ¬Train_Approaching)` |
| C-03 | `Barrier_Opening → (Crossing_Clear ∧ ¬Train_Present)` |
| C-04 | `Train_Approaching → (Warning_On ∧ Alarm_On)` |
| C-05 | `¬Train_Sensor_OK → (¬Barrier_Open ∧ Operator_Alerted)` |
| C-06 | `¬Barrier_OK → (¬Robot_Entering_Crossing ∧ Operator_Alerted)` |
| C-07 | `¬ARLCCS_Link_OK → ¬Robot_Entering_Crossing` |
| C-08 | `Readings_Conflict → (¬Barrier_Open ∧ Event_Logged)` |
| C-09 | `Robot_In_Crossing → ¬Robot_Stopped` |
| C-10 | `(Robot_State = DELIVERING ∧ Door_Open) → Clinic_Confirmed` |
| C-11 | `Robot_Moving → ¬Door_Open` |
| C-12 | `Robot_State = DELIVERING → Prev_State = NAVIGATING` |
| C-13 | `(Robot_State = NAVIGATING ∧ ¬MC_Link_OK ∧ Link_Down_Time > 30) → Robot_State = LINK_LOST` |
| C-14 | `((Chamber_Temp > 8 ∨ Battery_Pct < 10) ∧ Time_In_Condition ≥ 3) → Robot_State = SAFE_MODE` |
| C-15 | `Batch_Removed → (Chamber_Safe ∧ ¬Protection_Active)` |
| C-16 | `Command_Accepted → (Authenticated ∧ Role_Authorised ∧ Command_Valid)` |
| C-17 | `(Command_Received ∨ State_Changed ∨ Alert_Raised) → Event_Logged` |
| C-18 | `Chamber_Resumed → (¬Door_Open ∧ All_Sensors_OK ∧ Temp_In_Range ∧ Hum_In_Range)` |
| C-19 | `Correction_Marked_Done → (Temp_In_Range ∧ Hum_In_Range)` |
| C-20 | `Power_Lost → (On_Emergency_Power ∨ (Incident_Recorded ∧ Safe_Shutdown))` |

---

## 3. Violation scenarios (variable values)

| ID | Violation scenario (values that break the rule) |
|---|---|
| C-01 | `Train_Present = TRUE`, `Barrier_Open = TRUE` (barrier motor stuck in the up position) |
| C-02 | `Robot_Entering_Crossing = TRUE`, `Barrier_Open = TRUE`, `Train_Approaching = TRUE` (train came after the check) |
| C-03 | `Barrier_Opening = TRUE`, `Crossing_Clear = FALSE`, `Train_Present = FALSE` (a car is still inside) |
| C-04 | `Train_Approaching = TRUE`, `Warning_On = TRUE`, `Alarm_On = FALSE` (siren broken) |
| C-05 | `Train_Sensor_OK = FALSE`, `Barrier_Open = TRUE`, `Operator_Alerted = FALSE` |
| C-06 | `Barrier_OK = FALSE`, `Robot_Entering_Crossing = TRUE` |
| C-07 | `ARLCCS_Link_OK = FALSE` (last message 5 s old), `Robot_Entering_Crossing = TRUE` |
| C-08 | `Readings_Conflict = TRUE` (sensor A: no train, sensor B: train), `Barrier_Open = TRUE`, `Event_Logged = FALSE` |
| C-09 | `Robot_In_Crossing = TRUE`, `Robot_Stopped = TRUE` (wheel blocked) |
| C-10 | `Robot_State = DELIVERING`, `Door_Open = TRUE`, `Clinic_Confirmed = FALSE` |
| C-11 | `Robot_State = NAVIGATING`, `Door_Open = TRUE` |
| C-12 | `Robot_State = DELIVERING`, `Prev_State = IDLE` (forbidden jump T-17) |
| C-13 | `Robot_State = NAVIGATING`, `MC_Link_OK = FALSE`, `Link_Down_Time = 45`, robot still NAVIGATING |
| C-14 | `Chamber_Temp = 11`, `Time_In_Condition = 5`, `Robot_State = NAVIGATING` |
| C-15 | `Batch_Removed = TRUE`, `Protection_Active = TRUE` |
| C-16 | `Command_Accepted = TRUE`, `Authenticated = TRUE`, `Role_Authorised = FALSE` (Viewer role sends "start delivery") |
| C-17 | `State_Changed = TRUE`, `Event_Logged = FALSE` (log write failed) |
| C-18 | `Chamber_Resumed = TRUE`, `Door_Open = FALSE`, `All_Sensors_OK = FALSE` (vibration sensor dead) |
| C-19 | `Correction_Marked_Done = TRUE`, `Temp_In_Range = FALSE` (cooler command sent but no new reading) |
| C-20 | `Power_Lost = TRUE`, `On_Emergency_Power = FALSE`, `Incident_Recorded = FALSE` |

---

## 4. How the violation is detected and what the system does

| ID | How the system detects it | Expected response |
|---|---|---|
| C-01 | The ARLCCS safety-monitoring unit compares the train sensor and the barrier position sensor every 100 ms. | Alarm on, **close the barrier** (use backup drive), alert the operator, event logged, tell robots to stop. |
| C-02 | The robot checks barrier and train status in the last 2 seconds before the stop line, and again while crossing. | **Robot stops** before the line (or clears the crossing if already in), alert operator, event logged. |
| C-03 | ARLCCS blocks the open command unless the clear-check passes; the monitor checks the command log. | Open command **cancelled**, barrier stays closed, alert operator, event logged. |
| C-04 | A monitor compares the train sensor with the status of lights and siren. | Switch on the backup siren, **operator alert**, event logged, barrier stays closed. |
| C-05 | The safety-monitoring unit self-test fails or the sensor sends no heartbeat. | **Barrier closed**, alarm on, **operator alert**, robots told to stop, event logged. |
| C-06 | Barrier position does not match the command, or its motor test fails. | **Robot stops** before the crossing, operator alert, event logged. |
| C-07 | The robot's message timer passes 2 seconds without an ARLCCS message. | **Robot stops**, retries the link, operator alert, event logged. |
| C-08 | The safety-monitoring unit compares the two sensors and the physical limits. | Treat as "train present": **barrier closed**, operator alert, **event logged**. |
| C-09 | A monitor reads robot speed while the position is inside the crossing zone. | Robot **drives out of the crossing**, operator alert, event logged. |
| C-10 | The robot monitor reads door sensor, state and the handover identity check. | **Close the door** if possible, stop the handover, operator alert, event logged. |
| C-11 | The robot monitor reads state and door sensor every second. | Robot **stops**, SafeModeTrigger, operator alert, event logged. |
| C-12 | The state model checks the previous state when entering DELIVERING. | Transition **refused**, state unchanged, event logged. |
| C-13 | The link timer in the robot reaches 30 seconds while the state is still NAVIGATING. | Robot goes to **LINK_LOST** (stops at a safe place, not on the crossing), keeps the chamber control running, logs locally. |
| C-14 | Battery and chamber monitors with a 3-second timer. | Robot goes to **SAFE_MODE**, operator alert, event logged. |
| C-15 | The chamber monitor compares the removal lock and the protection flag. | **Lock the batch again**, operator alert, event logged. |
| C-16 | Mission Control software checks the three flags before executing the command (SO-01, SO-02, SO-03). | **Reject the command**, show the reason, log with operator ID. |
| C-17 | A log monitor compares the event counter with the number of log entries. | Retry the log write, store it locally, **operator alert** if it fails again. |
| C-18 | The chamber monitor checks the four conditions before it clears the resume lock. | **Lock again**, stay SUSPENDED, alert operator, event logged. |
| C-19 | The chamber monitor looks at the time stamp and value of the latest sensor reading. | Reopen the correction, restart the recovery timer, event logged. |
| C-20 | The power sensor shows main power lost, no emergency voltage, and no incident entry. | **Record the incident**, **safe shutdown**, operator alert. |

---

## 5. Required items checklist

| What the task asked | Where it is |
|---|---|
| At least 10 constraints with ID, English and reason | Section 1 (20 constraints) |
| At least 8 formalised with logic | Section 2 (all 20) |
| At least 8 violation scenarios with values | Section 3 (all 20) |
| Detection and response for each violation | Section 4 |
| At least 2 robot-chamber constraints | C-10, C-11 (also C-12 and C-14 link robot and chamber) |
| One constraint for sensor failure or communication loss | C-05, C-07, C-13 |
| Decidable from observed values | Section 0 lists the observed variable for each term |
