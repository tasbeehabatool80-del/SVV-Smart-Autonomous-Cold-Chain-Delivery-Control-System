# SACDCS: Software Quality Engineering Open-Ended Lab (OEL)

## What this project is

This repository holds my individual work for the OEL on the **Smart Autonomous Cold-Chain Delivery & Control System (SACDCS)**. In the scenario, autonomous robots carry vaccines and medicines from a central warehouse to remote clinics. Each robot has a sealed conservation chamber and is supervised from a Mission Control Centre. The robots must also cross railway level-crossings safely.

My role is **Software Quality Engineering Lead**. I analyse, model, constrain, verify and document the quality of the system. This is a paperwork and analysis project, so there is no program code.

## Repository structure

```
SVV-Smart-Autonomous-Cold-Chain-Delivery-Control-System/
├── README.md
├── CHANGELOG.md
├── task1-requirements/
│   └── task1-requirements.md
├── task2-change-requests-rtm/
│   └── task2-change-requests-rtm.md
├── task3-state-modelling/
│   └── task3-state-modelling.md
├── task4-operation-schema/
│   └── task4-operation-schema.md
└── task5-constraints-railway/
    └── task5-constraints-railway.md
```

## What is in each task

| Task | File | Content |
|---|---|---|
| 1 | `task1-requirements/task1-requirements.md` | 10 functional, 5 non-functional and 4 safety requirements (baseline v1.0, updated to v1.1 after the CRs) |
| 2 | `task2-change-requests-rtm/task2-change-requests-rtm.md` | Change log for CR-01, CR-02, CR-03; why the new wording is better; 22 test cases; re-run list; multi-layer RTM |
| 3 | `task3-state-modelling/task3-state-modelling.md` | Robot states, events, transition table, Safe Mode and Link Lost, verifier investigation |
| 4 | `task4-operation-schema/task4-operation-schema.md` | 18 chamber operations, 12 system operations, 8 operation schemas, state changes, sensor-checked operations |
| 5 | `task5-constraints-railway/task5-constraints-railway.md` | 20 constraints, logic, violation scenarios, detection and response |
| Log | `CHANGELOG.md` | Version history of the requirements |

## ID guide

| Prefix | Meaning | Prefix | Meaning |
|---|---|---|---|
| FR | Functional requirement | O | Chamber operation |
| NFR | Non-functional requirement | SO | System / robot operation |
| SR | Safety (high-risk) requirement | CH / CT | Chamber state / chamber transition |
| S / E / T | Robot state / event / transition | C | Constraint |
| CR | Change request | TC | Test case |

## Version control

The requirements were first committed as **v1.0** (baseline). Each Change Request was then applied as its own commit, giving **v1.1**. The Git history and `CHANGELOG.md` show every change.

## Assumptions

The scenario gives no exact numbers, so I chose some (for example Safe Mode within 3 seconds, critical battery 20%, emergency battery 10%, chamber limit 8 °C, link-loss time 30 seconds). They are listed in `task1-requirements/task1-requirements.md`, section 3.
