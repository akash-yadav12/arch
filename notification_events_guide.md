# 📬 LMS Notification Events & Recipient Guide

This guide provides a comprehensive, easy-to-understand breakdown of every notification event type available in the LMS. It covers **how notifications are triggered**, **how recipients are resolved**, **how template filters work**, and **all available placeholder tokens**.

---

## 🧭 Executive Summary & Core Concepts

### 1. Delivery Schedules
Notifications in LMS operate under two schedules:
- ⚡ **Instant (Transactional)**: Dispatched immediately when a user or admin performs an action (e.g. submitting a nomination, approving an enrollment).
- ⏰ **Scheduled (Cron-based)**: Evaluated periodically in the background (e.g. hourly/daily checks for upcoming session reminders, certificate expiry warnings, or monthly reports).

### 2. How Recipients Are Resolved
Whenever a notification is triggered, the system resolves recipients through a 4-step pipeline:

```
┌──────────────────────────┐      ┌──────────────────────────┐
│ Event Payload Data       │ ───> │ User & Role Hydration    │
│ (User IDs, Rig ID)       │      │ (MongoDB + RoleBinding)  │
└──────────────────────────┘      └──────────────────────────┘
                                               │
                                               ▼
┌──────────────────────────┐      ┌──────────────────────────┐
│ Final Email List         │ <─── │ Template Filtering       │
│ (To / CC Deduplicated)   │      │ (triggerRoles + Static)  │
└──────────────────────────┘      └──────────────────────────┘
```

- **Participant Matching**: Gathers primary participants (learners, nominators, rig staff).
- **Role & Designation Filtering**: Filters participants by their assigned role (`hse_advisor`, `instructor`, `manager`, etc.) or designation (`NTP`, `HSE Advisor`, `Medic`, etc.) if specified on the template's `triggerRoles`. If `triggerRoles` is empty, all participants are included.
- **Static TO & CC**: Appends any fixed admin/department emails entered in `staticTo` and `staticCc`.
- **Deduplication**: Ensures no email address receives duplicate emails for the same trigger.

---

## 📁 1. Enrollment Events

### 1.1 Enrollment Approved (`ENROLLMENT_APPROVED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when an admin approves a learner's enrollment request into a course.
* **Filters Available**:
  * `triggerCourseIds`: Restrict this template to specific course(s).
  * `triggerRigIds`: Restrict to specific rig(s).
  * `triggerRoles`: Restrict which learner roles/designations receive the notification.

#### 👥 Recipient Resolution
- **Learner (`To`)**: The employee whose enrollment was approved.
- **Static TO & CC**: Any extra addresses in `staticTo` or `staticCc` (e.g. Training Admin).

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{USER_NAME}}` | Full name of the enrolled employee | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Name of the approved course | `H2S Safety Awareness` | ✅ Required |
| `{{INSTANCE_START_DATE}}` | Session start date | `01 July 2026` | Optional |
| `{{INSTANCE_END_DATE}}` | Session end date | `03 July 2026` | Optional |
| `{{INSTRUCTOR_NAME}}` | Assigned instructor | `Mohammed Al-Farsi` | Optional |
| `{{DASHBOARD_URL}}` | Direct link to learner's scheduled courses | `https://lms.kdckwt.com/my-courses` | Optional |

---

### 1.2 Enrollment Rejected (`ENROLLMENT_REJECTED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when an admin rejects a learner's enrollment request.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Learner (`To`)**: The employee whose enrollment was declined.
- **Static TO & CC**: Additional static addresses configured on the template.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{USER_NAME}}` | Full name of the employee | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Name of the course | `H2S Safety Awareness` | ✅ Required |
| `{{REJECTION_REASON}}` | Explanation provided by the admin | `Prerequisites not completed` | Optional |

---

## 📁 2. Nomination Events

### 2.1 Nomination Submitted (`NOMINATION_SUBMITTED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when a line manager or HSE advisor submits a nomination batch for employee(s).
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Static TO & CC**: Primary destination for notifying the **HSE Admin / Training Department** (`staticTo`).
- **Nominator**: The user who submitted the nomination (e.g., Rig Superintendent / Manager).
- **Nominees**: The employees included in the nomination batch.
- **Rig Staff**: HSE Advisors or staff assigned to the target rig(s).

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{NOMINATOR_NAME}}` | Name of person who submitted nomination | `Sara Al-Jaber` | ✅ Required |
| `{{COURSE_NAME}}` | Nominated course name | `Fire Fighting Basic` | ✅ Required |
| `{{NOMINEE_COUNT}}` | Number of employees nominated | `5` | ✅ Required |
| `{{NOMINATION_NOTES}}` | Notes entered during submission | `Urgent — rig rotation next month` | Optional |
| `{{REVIEW_URL}}` | Direct link to review the nomination | `https://lms.kdckwt.com/nominations/123` | Optional |

---

### 2.2 Nomination Approved (`NOMINATION_APPROVED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when an administrator approves employees from a submitted nomination request.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **The Nominator**: Receives confirmation that their nominated employees were approved.
- **Approved Nominees**: Enrolled learners whose nominations were accepted.
- **Static TO & CC**: Additional static recipients.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{RECIPIENT_NAME}}` | Name of email recipient | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Course name | `Fire Fighting Basic` | ✅ Required |
| `{{APPROVED_BY}}` | Name of approving admin | `HSE Manager` | Optional |
| `{{INSTANCE_START_DATE}}` | Start date of enrolled session | `15 July 2026` | Optional |
| `{{INSTANCE_END_DATE}}` | End date of enrolled session | `17 July 2026` | Optional |
| `{{INSTRUCTOR_NAME}}` | Assigned instructor name | `Mohammed Al-Farsi` | Optional |

---

### 2.3 Nomination Rejected (`NOMINATION_REJECTED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when an administrator rejects employees from a nomination request.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **The Nominator**: Receives rejection feedback and reason.
- **Rejected Nominees**: Employees whose nomination was declined.
- **Static TO & CC**: Additional static recipients.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{RECIPIENT_NAME}}` | Name of email recipient | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Course name | `Fire Fighting Basic` | ✅ Required |
| `{{REJECTION_REASON}}` | Explanation for rejection | `Session is full` | Optional |
| `{{REJECTED_BY}}` | Name of rejecting admin | `HSE Manager` | Optional |

---

### 2.4 Nomination Waitlist Expired (`NOMINATION_WAITLIST_EXPIRED`)
* **Schedule**: ⏰ Scheduled (Cron)
* **Trigger**: Fired automatically when a waitlisted nomination expires without seat allocation.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Nominee**: Learner whose waitlist status expired.
- **Nominator**: User who originally submitted the nomination.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{USER_NAME}}` | Name of nominee | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Course name | `Confined Space Entry` | ✅ Required |
| `{{NOMINATOR_NAME}}` | Name of nominator | `Sara Al-Jaber` | Optional |
| `{{EXPIRY_DATE}}` | Expiry date | `20 August 2026` | Optional |

---

## 📁 3. Waitlist Events

### 3.1 Waitlist Added (`WAITLIST_ADDED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when a learner is added to a course waitlist queue.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Learner (`To`)**: User added to waitlist.
- **Static TO & CC**: Configured static recipients.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{USER_NAME}}` | Learner name | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Course name | `Confined Space Entry` | ✅ Required |
| `{{WAITLIST_POSITION}}` | Queue position number | `3` | Optional |

---

### 3.2 Waitlist Promoted (`WAITLIST_PROMOTED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when a waitlisted learner is promoted into an open course seat and enrolled.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Learner (`To`)**: Promoted employee enrolled in the session.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{USER_NAME}}` | Learner name | `Ahmed Al-Rashidi` | ✅ Required |
| `{{COURSE_NAME}}` | Course name | `Confined Space Entry` | ✅ Required |
| `{{INSTANCE_START_DATE}}` | Start date of enrolled session | `10 August 2026` | Optional |
| `{{INSTANCE_END_DATE}}` | End date of enrolled session | `12 August 2026` | Optional |
| `{{INSTRUCTOR_NAME}}` | Instructor name | `Mohammed Al-Farsi` | Optional |

---

## 📁 4. Scheduling & Course Session Events

### 4.1 Course Series Created (`COURSE_SERIES_CREATED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when a new course session batch/series is scheduled in the calendar.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Rig Staff**: HSE Advisors, Superintendents, and staff assigned to target rig(s).
- **Static TO & CC**: Configured static recipients.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{COURSE_NAME}}` | Name of scheduled course | `H2S Safety Awareness` | ✅ Required |
| `{{INSTANCE_START_DATE}}` | Session start date | `01 July 2026` | ✅ Required |
| `{{INSTANCE_END_DATE}}` | Session end date | `01 July 2026` | Optional |
| `{{INSTRUCTOR_NAME}}` | Instructor name | `Mohammed Al-Farsi` | Optional |

---

### 4.2 Course Series Cancelled (`COURSE_SERIES_CANCELLED`)
* **Schedule**: ⚡ Instant (Transactional)
* **Trigger**: Fired when a course session is cancelled in the calendar.
* **Filters Available**: Course Filter, Rig Filter, Role Filter.

#### 👥 Recipient Resolution
- **Enrolled Learners & Rig Staff**: Affected learners and rig personnel.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{COURSE_NAME}}` | Course name | `H2S Safety Awareness` | ✅ Required |
| `{{INSTANCE_DATE}}` | Cancelled session date | `01 July 2026` | ✅ Required |
| `{{CANCELLATION_REASON}}` | Reason for cancellation | `Instructor unavailable` | Optional |

---

### 4.3 Course Series Reminder (`COURSE_SERIES_REMINDER`)
* **Schedule**: ⏰ Scheduled (Cron background poller)
* **Trigger**: Scans continuously for upcoming sessions starting within `hoursBeforeReminder` (e.g. 12h, 24h, 48h).
* **Filters Available**:
  * `hoursBeforeReminder`: Look-ahead window in hours (set under Scheduler tab).
  * `triggerCourseIds`: Course filter.
  * `triggerRoles`: Role filter.

#### 👥 Recipient Resolution
- **Enrolled Learners**: Learners enrolled in the upcoming session starting within the target window.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{COURSE_NAME}}` | Course name | `H2S Safety Awareness` | ✅ Required |
| `{{INSTANCE_START_DATE}}` | Session start date and time | `01 July 2026` | ✅ Required |
| `{{INSTANCE_END_DATE}}` | Session end date | `01 July 2026` | Optional |
| `{{INSTRUCTOR_NAME}}` | Assigned instructor name | `Mohammed Al-Farsi` | Optional |

---

## 📁 5. Expiry & Compliance Events

### 5.1 Training Expiry Reminder (`TRAINING_EXPIRY_REMINDER`)
* **Schedule**: ⏰ Scheduled (Cron)
* **Trigger**: Evaluates certificate expirations matching `expiryWindowDays` (e.g. 30, 45, 60 days).
* **Filters Available**:
  * `expiryWindowDays`: Days look-ahead for expiring certificates.
  * `startDay` / `startTime` / `frequency`: Execution time configuration.

#### 👥 Recipient Resolution
- **Rig Emails (`rig.email`)**: Sent to the comma-separated email list on each active rig model. Each rig receives **only its own rig's report**.
- **Static TO & CC**: Manager/Senior static recipients receive overall reports.

#### 📁 Attachments Generated
- Automated 2-sheet Excel report attached to each email:
  - **Sheet 1**: Certificates expiring within `expiryWindowDays`.
  - **Sheet 2**: Already expired certificates.

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{RIG_NAME}}` | Name of receiving rig | `Rig 7 — KDC-Alpha` | ✅ Required |
| `{{REPORT_MONTH}}` | Month and year | `July 2026` | ✅ Required |
| `{{EXPIRY_WINDOW_DAYS}}` | Selected window days | `30` | ✅ Required |
| `{{EXPIRING_COUNT}}` | Count of expiring certificates | `12` | ✅ Required |
| `{{EXPIRED_COUNT}}` | Count of expired certificates | `5` | ✅ Required |

---

## 📁 6. Management & Reports Events

### 6.1 Monthly Rig Report (`MONTHLY_REPORT`)
* **Schedule**: ⏰ Scheduled (Cron)
* **Trigger**: Evaluated on scheduled start day and time of every month.
* **Filters Available**: `startDay` (e.g. 1st of month), `startTime` (e.g. 08:00), `frequency` (`MONTHLY`).

#### 👥 Recipient Resolution & Attachments
1. **Per-Rig Emails**: Dispatched to each rig's configured email addresses (`rig.email`). Includes **3 rig-specific Excel attachments**:
   - `Stats_Report_<RigName>.xlsx`
   - `Training_Breakdown_<RigName>.xlsx`
   - `Training_Matrix_<RigName>.xlsx` (styled training matrix matching UI)
2. **Manager All-Rigs Combined Email**: Dispatched to `staticTo` / `staticCc` (excluding rig emails). Includes **3 all-rigs Excel attachments**:
   - `Stats_Report_All_Rigs.xlsx`
   - `Training_Breakdown_All_Rigs.xlsx`
   - `Training_Matrix_All_Rigs.xlsx` (multi-sheet workbook with one tab per rig)

#### 🏷️ Available Placeholder Tokens
| Token | Description | Example | Required? |
|---|---|---|---|
| `{{RIG_NAME}}` | Rig name or "All Rigs Combined" | `Rig 7` | ✅ Required |
| `{{REPORT_MONTH}}` | Month and year | `June 2026` | ✅ Required |
| `{{TOTAL_EMPLOYEES}}` | Total active employees | `120` | ✅ Required |
| `{{OVERALL_COMPLIANCE_PERCENT}}` | Compliance percentage | `87` | ✅ Required |
| `{{TOTAL_OVERDUE}}` | Count of overdue courses | `14` | Optional |
| `{{TOTAL_COMPLETED}}` | Count of completed courses | `45` | Optional |
| `{{TOTAL_EXPIRING}}` | Count of expiring courses | `8` | Optional |
| `{{REPORT_HTML_BODY}}` | Pre-rendered HTML KPI table | `<table>...</table>` | Optional |

---

## 🌐 7. Common Placeholders (Available Across ALL Events)

The following tokens are globally available in every subject and body editor:

| Token | Description | Example |
|---|---|---|
| `{{CURRENT_DATE}}` | Current date formatted as DD Month YYYY | `26 July 2026` |
| `{{CURRENT_YEAR}}` | Current 4-digit year | `2026` |
| `{{FRONTEND_URL}}` | LMS web application base URL | `https://lms.kdckwt.com` |
| `{{SUPPORT_EMAIL}}` | Contact support email | `support@kdckwt.com` |
