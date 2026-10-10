# Test Cases and Results

## 1. Purpose

This document records the principal validation scenarios exercised during development of the Workflow Automation Hub prototype.

The results below reflect the tests performed during development. Rerun relevant tests after changing workflow logic or test data.

## 2. Test Matrix

| ID | Test scenario | Expected result | Observed result |
|---|---|---|---|
| TC-01 | Valid daily task completion with proof/update | Cycle becomes `Completed`; completion timestamp and employee email are recorded; next cycle is created | Passed for `T001-CYCLE-001`; `T001-CYCLE-002` was created |
| TC-02 | Daily task submitted without proof/update | Validation rejects submission and the sheet remains unchanged | Passed when execution stopped at the validation node |
| TC-03 | Portal task submitted without Ticket ID | Validation rejects submission and the sheet remains unchanged | Passed when execution stopped at the validation node |
| TC-04 | Wrong employee attempts to complete another employee's task | Authorization check rejects submission | Passed for Employee A attempting Employee C's task |
| TC-05 | Duplicate completion of an already completed cycle | Validation rejects the submission because the cycle is no longer pending | Passed for `T001-CYCLE-001` |
| TC-06 | Initial cycle creation | One initial cycle is created for an active task; reruns do not create a duplicate | Duplicate prevention is implemented; verify again with a fresh task ID if needed |
| TC-07 | Recurring cycle creation after completion | A new cycle is created with an incremented cycle number and configured frequency | Next cycle creation was observed; due-time and timezone handling require standardization |
| TC-08 | Task overdue beyond alert delay | Manager email is sent and `Last_Manager_Alert_At` is recorded | Passed; manager alert emails were sent and timestamps recorded |
| TC-09 | Monitor rerun after a manager alert | No duplicate manager alert is sent for the same cycle | Passed; previously alerted rows were skipped |
| TC-10 | Unacknowledged task passes the acknowledgement window | Leadership escalation is sent and `Escalated_At` is recorded | Passed for eligible pending cycles; a subsequent run returned no eligible rows |
| TC-11 | Manager clicks acknowledgement link | `Manager_Acknowledged_At` is recorded in the matching cycle row | Passed using the webhook test URL while the listener was running |
| TC-12 | Manager clicks acknowledgement link more than once | Original timestamp is preserved and a repeat acknowledgement is handled safely | Not fully confirmed end-to-end; verify before claiming as tested |

## 3. How to Reproduce the Tests

1. Use dummy task and employee records.
2. For invalid completion tests, execute only through the `Validate Completion` node. Do not execute downstream update nodes after a validation error.
3. For valid completion tests, run the employee-completion workflow and confirm the current cycle is completed and the next cycle is appended.
4. To test manager alerts, set a test cycle's due date in the past and verify `Last_Manager_Alert_At`.
5. Run the monitor again and confirm that the same cycle does not generate a duplicate manager alert.
6. To test escalation, ensure the task remains pending/overdue, `Manager_Acknowledged_At` is empty, `Escalated_At` is empty, and the 2-hour acknowledgement window has passed.
7. Run the monitor and verify that an escalation email is sent and `Escalated_At` is recorded.
8. To test acknowledgement, start the acknowledgement webhook's test listener, click the link in the manager email, and verify column K (`Manager_Acknowledged_At`) in `Task_Cycles`.

## 4. Recommended Screenshots

Include the following evidence in the `screenshots/` directory:

- Task master and task-cycle sheets.
- Manager dashboard.
- All four workflow canvases.
- Manager alert email with a populated acknowledgement link.
- Completed task cycle and its next recurring cycle.
- Task cycle showing the manager acknowledgement timestamp.
- Leadership escalation email and its `Escalated_At` timestamp.

## 5. Reporting Results Accurately

Only claim that a test passed when the corresponding result has been observed.

The acknowledgement link currently uses an n8n test webhook and is temporary. The current escalation code uses a 2-hour acknowledgement window; the 24-hour value in `Config` is not yet wired into that code.

Due-date timezone handling and preservation of the original timestamp on repeated acknowledgement clicks should be verified before production deployment.
