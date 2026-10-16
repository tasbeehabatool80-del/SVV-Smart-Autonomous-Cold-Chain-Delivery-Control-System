# Task 1: Requirement Extraction

**Project:** Smart Autonomous Cold-Chain Delivery & Control System (SACDCS)
**Role:** Software Quality Engineering Lead
**Document version:** v1.0 (approved baseline, before any Change Request)

## Notes
- The scenario gives no exact numbers, so I chose values (listed in section 3) to make every requirement testable.
- FR-01, FR-08 and NFR-02 are written in their **original wording** on purpose. Mission Control changes them later in Task 2 (CR-03, CR-01, CR-02).
- Type: FR = Functional, NFR = Non-Functional, Safety = High-Risk.
- Verification methods: Test (run and check), Simulation (fake the train, barrier or sensor), Inspection (read log or document), Analysis (check the model on paper).

## 1. Requirements

### 1.1 Functional Requirements (10)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 | Only authenticated operators shall be permitted to issue robot commands. | FR | High | Test: valid login, wrong password, no login |
| FR-02 | Before sending a command, the system shall check it is well formed, names an existing robot, and is allowed in that robot's current state. A bad command is rejected with an error code within 2 seconds. | FR | High | Test: bad command, unknown robot, wrong state |
| FR-03 | The robot follows this order: a valid DeliveryRequest starts navigation, it delivers at the destination, returns to the warehouse, and becomes idle. If the battery reaches 20% during navigation, it abandons the delivery and returns. | FR | High | Test + Analysis against the state model |
| FR-04 | On obstacle detection the robot suspends navigation, avoids the obstacle, and resumes the same trip when cleared. If avoidance lasts over 120 seconds, it enters Safe Mode. | FR | High | Test with a simulated obstacle |
| FR-05 | The chamber runs a self-check at power-on and enters monitoring only if all essential sensors pass. Conservation starts only with the door closed and the batch profile (ID, temperature limits, humidity limits) loaded. If the door opens during conservation, normal operation is suspended and resumes only after sensors and conditions are verified. | FR | High | Test: disabled sensor, open door, no profile |
| FR-06 | The chamber reads temperature and humidity every 1 second. When a value is out of range it starts correction within 2 seconds, and marks it corrected only when a new sensor reading is back in range. | FR | High | Test: force out-of-range, block the cooler |
| FR-07 | If correction fails within the recovery period (10 minutes), the chamber enters protection mode, reduces light exposure and alerts the operator within 5 seconds. | FR | High | Test: keep correction failing 10 minutes |
| FR-08 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | FR | High | Test: raise temperature, lower battery |
| FR-09 | The robot sends telemetry (state, position, battery, temperature, humidity, door, power) every 10 seconds, and sends each state change and alert within 5 seconds. | FR | Medium | Test with 5-second link delay |
| FR-10 | The system logs every command, authentication result, state change and alert with timestamp, robot ID, operator ID and result. Operators cannot edit or delete entries. | FR | High | Test + Inspection |

### 1.2 Non-Functional Requirements (5)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| NFR-01 (Performance) | Authentication takes at most 2 seconds. With link delay up to 5 seconds, a command result shows on the screen within 15 seconds. | NFR | Medium | Test 30 commands with a delay tool |
| NFR-02 (Scalability) | The system shall support communication with multiple robots simultaneously. | NFR | Medium | Test with several simulated robots |
| NFR-03 (Security) | An account is locked for 15 minutes after 5 failed logins. A session ends after 10 minutes idle. All data between Mission Control and robots is encrypted. | NFR | High | Test + Inspection (capture traffic) |
| NFR-04 (Reliability) | Chamber monitoring and safety actions keep working with no link. No log entry is lost: each robot stores 24 hours locally and uploads
