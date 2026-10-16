# Task 4: Operation Identification & Operation Schema (Conservation Chamber)

---

## 1. Operations

Operation names are **actions** (verbs). They are not state names.

### 1.1 Chamber operations (18)

| Op ID | Operation | What it does | Triggered by |
|---|---|---|---|
| O-01 | RunSelfCheck | Tests all essential sensors at power-on | Power-on |
| O-02 | LoadBatchProfile | Loads the batch ID and its temperature and humidity limits | Batch placed in the chamber |
| O-03 | StartConservation | Starts active control of temperature and humidity | Door closed and profile loaded |
| O-04 | ReadSensors | Reads temperature, humidity, vibration, door and power sensors every 1 second | Timer |
| O-05 | CorrectTemperature | Turns the cooler or heater on to bring the temperature back in range | Temperature out of range |
| O-06 | CorrectHumidity | Turns the humidifier or drier on to bring the humidity back in range | Humidity out of range |
| O-07 | VerifyRecovery | Checks new sensor readings to confirm the value is really back in range | After a correction command |
| O-08 | ActivateProtection | Starts the protection response | Correction failed within the recovery period |
| O-09 | ReduceLight | Lowers the light level inside the chamber | Protection response |
| O-10 | AlertOperator | Sends a warning to the operator and Mission Control | Any alarm |
| O-11 | SuspendRiskyActivities | Pauses risky actions | Vibration above the threshold |
| O-12 | ResumeAfterStabilisation | Restarts paused actions when vibration stays below the threshold for 30 seconds | Vibration low for 30 seconds |
| O-13 | SuspendOnDoorOpen | Pauses normal operation and sets a "do not resume" lock | Door opens during conservation |
| O-14 | VerifyAndResume | Checks door closed, all sensors working and values in range, then removes the lock | Door closed again |
| O-15 | SwitchToEmergencyPower | Changes the chamber supply to the emergency power source | Main power lost |
| O-16 | RecordIncident | Writes the incident (time, reason, last readings) to storage | Power lost with no emergency power |
| O-17 | ShutDownSafely | Turns off the chamber in a safe order | After O-16 |
| O-18 | AllowBatchRemoval | Unlocks the batch for removal | Authorised removal request |

### 1.2 System and robot operations (12, used in the RTM of Task 2)

| Op ID | Operation | Op ID | Operation |
|---|---|---|---|
| SO-01 | AuthenticateOperator | SO-07 | TriggerSafeMode |
| SO-02 | AuthoriseCommand | SO-08 | DetectLinkLoss |
| SO-03 | ValidateCommand | SO-09 | PlanAndFollowRoute |
| SO-04 | ReportStatus | SO-10 | AvoidObstacle |
| SO-05 | WriteEventLog | SO-11 | HandOverPackage |
| SO-06 | CheckCrossingPermission | SO-12 | CheckBattery |

---

## 2. Chamber states (used in Parts 3 and 4)

| ID | Chamber state | Meaning |
|---|---|---|
| CH-01 | SELF_CHECK | Testing sensors after power-on |
| CH-02 | MONITORING | All essential sensors work; no batch is being conserved yet |
| CH-03 | READY | Batch profile loaded; waiting for the door to close |
| CH-04 | CONSERVING | Active control keeps temperature and humidity in range |
| CH-05 | CORRECTING | A value is out of range and is being corrected |
| CH-06 | SUSPENDED | Normal operation paused (door open or vibration high) |
| CH-07 | PROTECTION | Correction failed; light reduced; operator alerted |
| CH-08 | ON_EMERGENCY_POWER | Running on the emergency source |
| CH-09 | SHUT_DOWN | Safely switched off |

---

## 3. Operation Schemas (8 operations)

### O-01 RunSelfCheck

| Field | Content |
|---|---|
| Operation name | RunSelfCheck |
| Description | Tests every essential sensor when the chamber is switched on |
| Inputs | Power-on signal, test readings from each essential sensor |
| Outputs | Result PASS or FAIL, list of failed sensors |
| Reads | Temperature, humidity, vibration, door and power sensors |
| Changes | Chamber state (SELF_CHECK to MONITORING on PASS), SelfCheckResult |
| Sends/Alerts | Alert to the operator if any essential sensor fails |
| Assumes (precondition) | Chamber has power (main or emergency) and is in SELF_CHECK |
| Result (postcondition) | If all essential sensors work: state is MONITORING. If not: state stays SELF_CHECK, monitoring is NOT entered, an alert is sent and the event is logged |

### O-02 LoadBatchProfile

| Field | Content |
|---|---|
| Operation name | LoadBatchProfile |
| Description | Loads the identity of the batch and its allowed temperature and humidity limits |
| Inputs | Batch ID, profile (minimum and maximum temperature, minimum and maximum humidity) |
| Outputs | Loaded profile, confirmation message |
| Reads | Batch ID, profile database |
| Changes | Active profile, chamber state (MONITORING to READY) |
| Sends/Alerts | Error to the operator if the batch ID has no profile |
| Assumes (precondition) | State is MONITORING and a batch has been placed in the chamber |
| Result (postcondition) | Profile is loaded and matches the batch ID; state is READY. If the profile is not found: no change, error sent, event logged |

### O-03 StartConservation

| Field | Content |
|---|---|
| Operation name | StartConservation |
| Description | Starts active control of temperature and humidity for the loaded batch |
| Inputs | Start request (or automatic start) |
| Outputs | Conservation status |
| Reads | Door sensor, profile-loaded flag, temperature and humidity sensors |
| Changes | Chamber state (READY to CONSERVING), control outputs switched on |
| Sends/Alerts | Refusal message to the operator if a condition is not met |
| Assumes (precondition) | State is READY, door is closed, profile is loaded |
| Result (postcondition) | State is CONSERVING and control targets the profile limits. If the door is open or no profile is loaded: refused, state does not change |

### O-05 CorrectTemperature

| Field | Content |
|---|---|
| Operation name | CorrectTemperature |
| Description | Tries to bring an out-of-range temperature back into range, then proves it worked |
| Inputs | Temperature reading, profile limits |
| Outputs | Cooler or heater command, outcome RECOVERED or FAILED |
| Reads | Temperature sensor, profile limits, recovery timer |
| Changes | Cooler or heater output, recovery timer, chamber state (CONSERVING to CORRECTING, then back to CONSERVING) |
| Sends/Alerts | Warning if the value is not improving; recovery-failed event if the timer ends |
| Assumes (precondition) | Temperature is outside the limits, state is CONSERVING, temperature sensor works |
| Result (postcondition) | Only when a **new sensor reading** is inside the limits: state returns to CONSERVING and the timer stops. If the recovery period (10 minutes) ends first: recovery-failed event, and O-08 starts |

### O-08 ActivateProtection

| Field | Content |
|---|---|
| Operation name | ActivateProtection |
| Description | Puts the chamber in its protection response when correction has failed |
| Inputs | Recovery-failed event |
| Outputs | Protection status, alert message |
| Reads | Recovery timer, temperature sensor, humidity sensor |
| Changes | Chamber state (CORRECTING to PROTECTION), light level (reduced by O-09), ProtectionActive = TRUE, batch removal locked |
| Sends/Alerts | Alert to the operator and Mission Control within 5 seconds; event logged |
| Assumes (precondition) | Correction did not succeed within the 10-minute recovery period |
| Result (postcondition) | State is PROTECTION, light is reduced, operator is alerted, ProtectionActive = TRUE, batch removal is blocked |

### O-13 SuspendOnDoorOpen

| Field | Content |
|---|---|
| Operation name | SuspendOnDoorOpen |
| Description | Pauses normal operation as soon as the door opens during conservation |
| Inputs | Door sensor changes to OPEN |
| Outputs | Suspension status |
| Reads | Door sensor, chamber state, robot state |
| Changes | Chamber state (CONSERVING or CORRECTING to SUSPENDED), "do not resume" lock set |
| Sends/Alerts | Alert to Mission Control; event logged; if the robot is moving, SafeModeTrigger is sent to the robot |
| Assumes (precondition) | Chamber state is CONSERVING or CORRECTING |
| Result (postcondition) | Normal operation is suspended and cannot resume until O-14 removes the lock |

### O-15 SwitchToEmergencyPower

| Field | Content |
|---|---|
| Operation name | SwitchToEmergencyPower |
| Description | Moves the chamber to the emergency power source when main power is lost |
| Inputs | Main power-loss signal |
| Outputs | Power source status, remaining emergency time |
| Reads | Main power sensor, emergency source voltage |
| Changes | Power source, chamber state (to ON_EMERGENCY_POWER) |
| Sends/Alerts | Alert to Mission Control: "running on emergency power"; event logged |
| Assumes (precondition) | Main power is lost and an emergency source is present |
| Result (postcondition) | Within 1 second the voltage sensor shows power on the emergency line and the controls are running. If this is not confirmed: O-16 (RecordIncident) and O-17 (ShutDownSafely) |

### O-18 AllowBatchRemoval

| Field | Content |
|---|---|
| Operation name | AllowBatchRemoval |
| Description | Lets an authorised person take the batch out, only when it is safe |
| Inputs | Removal request from an authorised operator |
| Outputs | Removal allowed or refused, with the reason |
| Reads | Chamber state, ProtectionActive flag, temperature and humidity sensors, door sensor |
| Changes | Removal lock (OFF if allowed), event log |
| Sends/Alerts | Confirmation or refusal message to the operator |
| Assumes (precondition) | Chamber is safe (values in range, not SUSPENDED) and ProtectionActive = FALSE |
| Result (postcondition) | Removal is allowed and logged. If a protection response is active or the chamber is not safe: refused with a reason |

---

## 4. How operations change the chamber state, and the link to the robot model (Task 3)

### 4.1 Chamber transition table

| CT-ID | Current chamber state | Event | Operation | Next chamber state |
|---|---|---|---|---|
| CT-01 | SELF_CHECK | All essential sensors pass | O-01 | MONITORING |
| CT-02 | SELF_CHECK | A sensor fails | O-01, O-10 | SELF_CHECK (blocked, alert) |
| CT-03 | MONITORING | Batch placed, profile loaded | O-02 | READY |
| CT-04 | READY | Door closed | O-03 | CONSERVING |
| CT-05 | CONSERVING | Temperature or humidity out of range | O-05 or O-06 | CORRECTING |
| CT-06 | CORRECTING | New reading is in range | O-07 | CONSERVING |
| CT-07 | CORRECTING | Recovery period ended | O-08, O-09, O-10 | PROTECTION |
| CT-08 | CONSERVING | Vibration above threshold | O-11 | SUSPENDED |
| CT-09 | SUSPENDED (vibration) | Below threshold for 30 seconds | O-12 | CONSERVING |
| CT-10 | CONSERVING or CORRECTING | Door opens | O-13 | SUSPENDED |
| CT-11 | SUSPENDED (door) | Door closed, sensors and values verified | O-14 | CONSERVING |
| CT-12 | CONSERVING or CORRECTING | Main power lost, emergency power available | O-15 | ON_EMERGENCY_POWER |
| CT-13 | Any working state | Main power lost, no emergency power | O-16, O-17 | SHUT_DOWN |
| CT-14 | PROTECTION | Values verified safe, operator acknowledges | O-04, O-10 | MONITORING |
| CT-15 | CONSERVING | Authorised removal request, no protection active | O-18 | MONITORING |
| CT-16 | ON_EMERGENCY_POWER | Main power back and verified | O-04 | CONSERVING |

### 4.2 Operation 1: SuspendOnDoorOpen (O-13), "door opened during conservation" (CT-10)

| Item | Detail |
|---|---|
| Chamber change | CONSERVING to SUSPENDED, resume lock set |
| If the robot is NAVIGATING, AVOIDING_OBSTACLE or RETURNING (moving) | An open door is a serious fault. O-13 sends SafeModeTrigger, so the robot goes to **SAFE_MODE** (T-21, T-22, T-24) |
| If the robot is DELIVERING | The door should open only after clinic staff are confirmed (constraint C-10). Opening it earlier is a serious fault and also sends SafeModeTrigger (T-23) |
| If the robot is IDLE | Normal. The chamber is suspended and checked again by O-14 when the door closes |
| Link to constraints | C-10, C-11, C-18 (Task 5) |

### 4.3 Operation 2: ActivateProtection (O-08), "correction failed" (CT-07)

| Item | Detail |
|---|---|
| Chamber change | CORRECTING to PROTECTION, light reduced, batch removal blocked |
| Link to the robot model (Task 3) | The operator is alerted. If the chamber temperature stays above 8 °C for 3 seconds, SafeModeTrigger is sent and the robot goes to **SAFE_MODE** (CR-01). While protection is active, the batch cannot be removed (constraint C-15) |
| Link to constraints | C-14, C-15 (Task 5) |

---

## 5. Two operations whose result must be checked with sensors

### 5.1 CorrectTemperature (O-05)
- **What can go wrong:** the command "cooler ON" can be sent and the cooler can still do nothing (broken cooler, blocked fan, open door, hot weather).
- **Why we must not assume:** if the system assumed the temperature was fixed after sending the command, the vaccines could slowly spoil while the screen says everything is fine.
- **What we do:** after the command, read the temperature sensor again. Mark the problem as fixed **only** when the new reading is inside the limits (FR-06, constraint C-19). If the recovery period ends first, start protection mode (O-08).

### 5.2 SwitchToEmergencyPower (O-15)
- **What can go wrong:** the switch command is sent, but the emergency source can be empty, disconnected or broken.
- **Why we must not assume:** the chamber would be "switched over" to a dead line and stop working without any warning.
- **What we do:** read the power sensor (voltage on the emergency line) within 1 second. If power is not confirmed, record the incident (O-16) and shut down safely (O-17) (SR-04, constraint C-20).

(Th
