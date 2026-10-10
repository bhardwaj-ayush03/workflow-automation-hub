# SLA and Escalation Design

## 1. Purpose

This document describes the service-level agreement (SLA) logic implemented in the Workflow Automation Hub prototype.

The objective is to identify overdue tasks, notify the responsible manager, track acknowledgement, and escalate eligible tasks without repeatedly sending the same notification.

## 2. Current Demonstration Policy

| Event | Rule | System action |
|---|---|---|
| Task due time passes | Task remains pending | Task becomes eligible for overdue detection |
| Manager alert | At least 30 minutes overdue | Send manager email and record `Last_Manager_Alert_At` |
| Manager acknowledgement | Expected within 2 hours of the alert | Record `Manager_Acknowledged_At` |
| Leadership escalation | At least 2 hours after the manager alert, if the task remains pending/overdue and unacknowledged | Send escalation email and record `Escalated_At` |
| Duplicate manager alert | Alert timestamp already exists | Skip another manager alert for that cycle |
| Duplicate escalation | Escalation timestamp already exists | Skip another escalation for that cycle |

## 3. Rationale

### Manager alert delay: 30 minutes

A short delay provides a buffer for ordinary operational delays while still allowing the manager to intervene on the same day.

### Acknowledgement window: 2 hours

The manager is given time to review the alert. If the alert remains unacknowledged beyond the configured window, the workflow can escalate it.

### Timestamp-based duplicate prevention

The workflow records alert and escalation timestamps against each task cycle. These fields help prevent repeated emails every time the monitoring workflow runs.

These values are prototype settings and should be confirmed with the operations team before production deployment.

## 4. Acknowledgement Is Not Completion

Acknowledging an alert means that the manager has reviewed it. It does not mean the employee's task is complete.

The employee completion workflow separately records the task's completion status and evidence.

The business policy should also specify what happens when a manager acknowledges an alert but the task remains unresolved.

## 5. Current Implementation Caveats

The `Config` sheet contains a 24-hour escalation setting. However, the current escalation Code node uses a 2-hour window after `Last_Manager_Alert_At`.

Therefore, the current prototype should be described as using a **2-hour acknowledgement-based escalation window**, not as enforcing a 24-hour unresolved-task SLA.

Some thresholds are hardcoded in Code nodes instead of being read dynamically from `Config`.

Before production deployment, the policy should distinguish between:

1. Alert delay after the task due time.
2. Manager acknowledgement SLA.
3. Resolution or unresolved-task escalation SLA.
4. Optional repeat-notification intervals.

## 6. Production Recommendations

- Use a persistent production webhook only after activating the acknowledgement workflow.
- Require manager authentication or validate a signed, expiring token before accepting acknowledgement requests.
- Preserve the original acknowledgement timestamp on repeated clicks.
- Standardize all due-date calculations and comparisons to the business timezone.
- Define escalation behaviour for acknowledged but unresolved tasks.
- Maintain an auditable history of completion, alerts, acknowledgements, and escalations.
- Read SLA thresholds from a single authoritative configuration source.
