# Task 2: Change Requests & Traceability Matrix (RTM)

IDs come from other tasks: requirements (Task 1), states, events, transitions (Task 3: S-, E-, T-), operations (Task 4: O-, SO-, CT-), constraints (Task 5: C-), test cases (this file: TC-).

## 1. Change log (v1.0 to v1.1)

| Change | CR | Req. | Old text | New text | Reason | Status |
|---|---|---|---|---|---|---|
| CH-01 | CR-01 | FR-08 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | "Critical" had no number and there was no time limit, so it could not be tested. | **Modified** |
| CH-02 | CR-02 | NFR-02 | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | "Multiple" could mean 2. The pilot serves 20 clinics. | **Modified** |
| CH-03 | CR-03 | FR-01 | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. | The old text checked only identity, not role. | **Modified** |

**Values for the new FR-08:** chamber above 8 °C, battery temperature above 60 °C, battery below 10%, time limit 3 seconds.

**Affected but not changed:** FR-02 (runs with the role check), FR-10 (role rejections must be logged), NFR-01 (timing must hold with 20 robots), FR-03 (battery rule works with Safe Mode).

### Rows to paste into Task 1 (v1.1)

| Req. ID | Requirement | Type | Priority | Verification Method |
|---|---|---|---|---|
| FR-01 (Modified v1.1) | The system shall require authenticated and role-authorised operators before accepting any robot command. | FR | High | Valid login with allowed role accepted; wrong role rejected; no login rejected |
| FR-08 (Modified v1.1) | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | FR | High | Raise temperature, lower battery, measure time (at most 3 s) |
| NFR-02 (Modified v1.1) | The system shall support at least 20 simultaneously connected robots. | NFR | Medium | 20 simulated robots at once, no loss, NFR-01 still met |

## 2. Why the new wording is more testable

| CR | Old problem | How the new wording fixes it |
|---|---|---|
| CR-01 | "Critical" has no value and there is no time limit. A robot reacting after 10 minutes would still pass. | It gives numbers (8 °C, 10%, 60 °C) and a 3-second limit, so a stopwatch or log time stamps decide pass or fail. It names three triggers, so there are separate tests (TC-08, TC-09). |
| CR-02 | "Multiple" could be 2 or 200. | "At least 20" is a number you can count. The test connects 20 robots and checks they all stay connected and respond. |
| CR-03 | It checked only who the person is, not what they may do. | Two checks give two tests (wrong password, wrong role). "Before accepting any robot command" says when the check happens and for which commands, so every command type can be tried. |

## 3. Test cases

Scenario steps: **S1** command received, **S2** authenticated, **S3** executed, **S4** status reported, **S5** logged.

| TC | Steps | Test | Expected result | Req. |
|---|---|---|---|---|
| TC-01 | S1-S5 | Dispatcher logs in, sends valid "start delivery" to R1 | Accepted, R1 acts, status shown, 4 log entries | FR-01, FR-02, FR-09, FR-10 |
| TC-02 | S1, S2, S5 | Command with no login, then expired session | Rejected within 2 s, robot unchanged, logged | FR-01, NFR-01, NFR-03 |
| TC-03 | S1, S2, S5 | Viewer role sends "start delivery" | Rejected "not authorised", logged with operator ID | FR-01, FR-10 |
| TC-04 | S1, S5 | Command to unknown robot ID; "start delivery" to a robot in SAFE_MODE | Both rejected with error code in 2 s, logged | FR-02 |
| TC-05 | S1-S5 | "Start delivery" to IDLE robot (5 s delay) | IDLE to NAVIGATING, shown within 15 s, logged | FR-03, NFR-01 |
| TC-06 | S4 | 5 s delay; watch telemetry; force a state change | Telemetry every 10 s or less; change sent within 5 s | FR-09, NFR-01 |
| TC-07 | S5 | Run TC-01 to TC-05; read log; try editing; cut link 10 min | Complete entries, edit refused, local entries uploaded | FR-10, NFR-04 |
| TC-08 | S3-S5 | While NAVIGATING, chamber above 8 °C; battery above 60 °C | SAFE_MODE within 3 s, alert, logged | FR-08 |
| TC-09 | S3-S5 | While NAVIGATING, battery to 9% | SAFE_MODE within 3 s, alert, logged | FR-08 |
| TC-10 | S1-S5 | 20 simulated robots, commands at once | All connected, all acknowledged, NFR-01 met | NFR-02, NFR-01 |
| TC-11 | S3-S5 | At stop line: barrier closed; open with train approaching; 3 s old data; link lost; conflicting sensors | Robot stays 5 m or more before the barrier in all 5 cases | SR-01 |
| TC-12 | S3-S5 | Cut link 30 s in NAVIGATING; restore; repeat 5 min; repeat on crossing | LINK_LOST at 30 s, chamber runs, restore gives NAVIGATING, 5 min gives RETURNING, on crossing finishes crossing first | SR-02, NFR-04 |
| TC-13 | S1, S3 | Force IDLE to DELIVERING, AVOIDING to DELIVERING, SAFE_MODE to DELIVERING; move with door open | All refused, state unchanged, logged | SR-03 |
| TC-14 | S3, S4 | Obstacle in NAVIGATING and in RETURNING; one held 120 s | Resumes the correct trip; 120 s gives SAFE_MODE | FR-04 |
| TC-15 | S3, S4 | Out-of-range temperature; block the cooler | Correction starts in 2 s; fixed only after an in-range reading | FR-06 |
| TC-16 | S3-S5 | Correction fails 10 min | PROTECTION, light reduced, alert within 5 s | FR-07 |
| TC-17 | S3-S5 | Disabled sensor at start; start with door open and no profile; open door in conservation | Monitoring blocked, start refused, suspended, no resume until verified | FR-05 |
| TC-18 | S3-S5 | Cut main power with and without emergency power | Switch within 1 s; else record in 2 s and shut down | SR-04 |
| TC-19 | S1, S2 | 5 wrong logins; 10 min idle session | Locked 15 min; session ends | NFR-03 |
| TC-20 | S1, S4 | 5 trained operators send a request | At least 4 of 5 pass in 5 steps and 60 s; red alerts with sound | NFR-05 |
| TC-21 | S3-S5 | Battery to 20% in NAVIGATING, AVOIDING, DELIVERING, RETURNING, IDLE, LINK_LOST | RETURNING; finish then RETURNING; finish then RETURNING; continue; refuse new requests; RETURNING | FR-03, FR-08 |
| TC-22 | S1-S5 | Full mission | IDLE, NAVIGATING, DELIVERING, RETURNING, IDLE, each reported and logged | FR-03, FR-09, FR-10 |

## 4. Test cases to re-run for each CR

| CR | Re-run | Why |
|---|---|---|
| CR-01 | **TC-08, TC-09** (changed), TC-21, TC-14, TC-07, TC-06, TC-13 | Safe Mode rule and timing changed; many flows end in Safe Mode |
| CR-02 | **TC-10** (changed), TC-06, TC-05, TC-07, TC-01 | More robots means more load on commands, telemetry and logs |
| CR-03 | **TC-02, TC-03** (TC-03 is new), TC-01, TC-04, TC-07, TC-19, TC-05, TC-13 | Every command now passes an extra check |

## 5. Multi-layer RTM

Risk: **High** = can harm people or medicine; **Medium** = can stop the mission or lose data; **Low** = comfort.

| Req. | State / Transition | Operation | Constraint | Test Case | Risk |
|---|---|---|---|---|---|
| FR-01 (v1.1) | T-01, T-26 | SO-01, SO-02 | C-16 | TC-01, 02, 03, 19 | High |
| FR-02 | T-17, T-28 | SO-03 | C-16 | TC-01, 04 | High |
| FR-03 | T-01, T-04 to T-12 | SO-09, SO-11, SO-12 | C-12 | TC-05, 21, 22 | High |
| FR-04 | T-02, T-03, T-09, T-13, T-14, T-16, T-19, T-40 | SO-10, SO-09 | C-11, C-12 | TC-14 | High |
| FR-05 | CT-01 to CT-04, CT-10, CT-11 | O-01, 02, 03, 13, 14 | C-11, C-18 | TC-17 | High |
| FR-06 | CT-05, CT-06 | O-04, 05, 06, 07 | C-19 | TC-15 | High |
| FR-07 | CT-07, CT-14 | O-08, 09, 10 | C-15 | TC-16 | High |
| FR-08 (v1.1) | T-20 to T-27, T-40 | SO-07, SO-12 | C-14 | TC-08, 09, 21 | High |
| FR-09 | All transitions | SO-04 | C-17 | TC-01, 06, 22 | Medium |
| FR-10 | All transitions | SO-05 | C-17 | TC-07, 01 | High |
| NFR-01 | T-01 | SO-01 to SO-04 | C-16 | TC-05, 06, 10 | Medium |
| NFR-02 (v1.1) | Load requirement | SO-04 | Measured by load test | TC-10 | Medium |
| NFR-03 | T-26 | SO-01, SO-02 | C-16 | TC-02, 19 | High |
| NFR-04 | T-30 to T-39 | SO-05, SO-08 | C-13, C-17 | TC-07, 12 | High |
| NFR-05 | Human factors | SO-04, O-10 | Measured with users | TC-20 | Low |
| SR-01 | T-02, T-30, T-33 | SO-06 | C-01 to C-09 | TC-11 | High |
| SR-02 | T-25, T-30 to T-39 | SO-08, SO-05 | C-13, C-09 | TC-12 | High |
| SR-03 | T-04, 15, 16, 17, 18, 29 | SO-11 | C-10, C-11, C-12 | TC-13 | High |
| SR-04 | CT-12, 13, 16 | O-15, 16, 17 | C-20 | TC-18 | High |

All 19 requirements have at least one test case, and every test case traces back to a requirement.
