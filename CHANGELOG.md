# Change Log (SACDCS Requirements)

All changes to the requirement baseline are written here. Newest version first.

---

## v1.1 (after the Change Requests from Mission Control)

Status of the changes: **Modified**

| Change ID | CR | Req. ID | Old text | New text | Reason |
|---|---|---|---|---|---|
| CH-01 | CR-01 | FR-08 | The robot shall enter Safe Mode when a critical battery or thermal condition is detected. | The robot shall enter Safe Mode within 3 seconds when chamber or battery temperature exceeds the critical threshold or battery capacity falls below the emergency level. | "Critical" had no number and there was no time limit, so it could not be tested. |
| CH-02 | CR-02 | NFR-02 | The system shall support communication with multiple robots simultaneously. | The system shall support at least 20 simultaneously connected robots. | "Multiple" is not a number. The pilot serves 20 clinics. |
| CH-03 | CR-03 | FR-01 | Only authenticated operators shall be permitted to issue robot commands. | The system shall require authenticated and role-authorised operators before accepting any robot command. | The old text checked only identity, not the role. |

Test cases to re-run: see Task 2, section 4.

---

## v1.0 (baseline)

- First approved set of requirements: 10 FR, 5 NFR and 4 safety requirements (see `task1-requirements/task1-requirements.md`).
- No changes yet.
