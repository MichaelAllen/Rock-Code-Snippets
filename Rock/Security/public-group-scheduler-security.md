---
tags:
    - type/utility
    - topic/groups
    - topic/security
date created: 2022-08-18 18:53:54
date modified: 2025-11-23 18:18:06
---

# Security Permissions For Public Group Scheduling

We needed a way to give permissions for someone to schedule their volunteer team but didn't want to give them access to the internal Rock site. The settings below are the minimum permissions needed for the group scheduler block to work fully.

We created a role `RSR - Public Group Scheduling` with this narrow scope of permissions so that we can easily assign it to any user that needs access to the scanner app. This role provides all of the needed permissions and doesn't require the user to have any other roles (`RSR - Staff Workers` etc.)

## REST Controllers

`Admin Tools > Security > Rest Controllers`

| Controller  | Method | Path                                                                                                            | Verbs      |
| ----------- | ------ | --------------------------------------------------------------------------------------------------------------- | ---------- |
| Attendances | GET    | api/Attendances/GetAttendingSchedulerResources?attendanceOccurrenceId={attendanceOccurrenceId}                  | View       |
| Attendances | POST   | api/Attendances/GetSchedulerResource?personId={personId}                                                        | View       |
| Attendances | POST   | api/Attendances/GetSchedulerResources                                                                           | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonAddConfirmed?personId={personId}&attendanceOccurrenceId={attendanceOccurrenceId} | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonAddPending?personId={personId}&attendanceOccurrenceId={attendanceOccurrenceId}   | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonConfirm?attendanceId={attendanceId}                                              | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonDecline?attendanceId={attendanceId}                                              | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonPending?attendanceId={attendanceId}                                              | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonRemove?attendanceId={attendanceId}                                               | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonSendConfirmationCommunication?attendanceId={attendanceId}                        | View, Edit |
| Attendances | PUT    | api/Attendances/ScheduledPersonSendConfirmationEmail?attendanceId={attendanceId}                                | View, Edit |
