# NexaCare Solutions — Testing & Release Planning Kit (v3.0)

> **Prepared by:** Krzysztof Macherzyński  
> **Role:** Systems Analyst  
> **Project:** Appointment Scheduling Platform — Version 3.0 Release  
> **Date:** 08.06.2026  

---

## Project scenario

NexaCare Solutions is a healthcare technology company supporting over 300 clinics across North America. The organization is preparing to launch Version 3.0 of its web-based appointment scheduling platform.

This version introduces new features that aim to improve usability, patient engagement, and clinic operations. As the Systems Analyst, your responsibility is to coordinate and document a complete testing and release plan to ensure a safe, compliant, and successful rollout.

### Key features in version 3.0

- SMS reminders
- Real-time doctor availability
- Calendar integration with Google and Outlook
- Redesigned user interface

### Key considerations

- HIPAA compliance (over 1.2M patient records)
- First-time integration with third-party SMS provider
- Diverse client configurations (from basic to API-integrated systems)
- Pilot program for 20 clinics prior to full release
- Scheduled deployment window: Saturday, 12:00 AM – 6:00 AM EST

---

## 1. Test strategy outline

### Overall approach

NexaCare v3.0 is validated with a risk-based, phased testing strategy. Effort is allocated in proportion to risk (likelihood x business impact), so the highest-risk, highest-value areas receive the deepest coverage: the first-time third-party SMS integration, real-time provider availability and concurrency, the 1.2M-record data migration, HIPAA-regulated patient data, and performance under the Monday-morning peak (1,500-2,500 concurrent users).

Testing follows a shift-left, test-pyramid model - extensive automated unit and integration tests at the base, focused system, regression and end-to-end tests in the middle, and targeted exploratory, accessibility, security and user-acceptance testing at the top. Defects are tracked in Jira with severity (Sev-1 to Sev-4); no Sev-1/Sev-2 defect may be open at go-live.

In scope: appointment booking, update/cancel; SMS reminders and fallback; real-time availability; Google/Outlook calendar sync; redesigned/responsive UI and accessibility; authentication and role-based permissions; data migration and integrity validation; performance/peak load; HIPAA security. Out of scope: the SMS vendor's internal infrastructure, third-party calendar providers' internal systems, and individual clinics' external EHR/billing systems (validated only at the integration contract boundary).

Clinic variability: the client base ranges from clinics using only basic scheduling to clinics with deep API integrations. Basic clinics are validated primarily through the UI and cross-browser/mobile testing; API-integrated clinics are validated with API contract and integration tests against the public endpoints. The 20-clinic pilot is selected as a representative sample across both profiles, regions and sizes.

Test environment: a production-like pre-production cloud instance loaded with anonymized, HIPAA-compliant data; a separate migration-staging environment holding a full-scale copy for dry runs; and the Twilio sandbox for SMS scenarios. Test data covers desktop and mobile, valid and invalid inputs, and edge records (hyphenated names, mixed phone formats, multiple time zones).

Entry criteria: code complete and peer-reviewed; unit tests >= 80% on changed modules; build deployed to pre-prod; test data provisioned. Exit criteria: 100% of P1 test cases pass; >= 95% of P2 pass; zero open Sev-1/Sev-2 defects; performance, security and accessibility targets met; migration reconciled at 100%; pilot UAT signed off.

The release is gated by the pilot: functional, performance, security and compliance testing must pass in staging before the pilot, and pilot telemetry and feedback then act as the final acceptance gate before the broader rollout to all 300 clinics.

### Types of testing and rationale

| **Type of Testing** | **Purpose / Rationale** |
| --- | --- |
| Unit testing | Verify individual functions/modules (booking logic, availability calculation, SMS payload builder, field-transformation rules) in isolation; fast feedback and high coverage of business rules before integration. |
| Integration & API contract testing | Validate interfaces with external systems - the new SMS provider, Google/Outlook calendar APIs and clinic EHR integrations - and the public API contract relied on by API-integrated clinics, where most release risk concentrates. |
| Functional / system testing | Confirm complete end-to-end use cases (book, update, cancel, view availability, sync calendar, authenticate) behave per requirements across the whole system. |
| Regression testing | Ensure the redesigned UI and new features do not break existing booking, authentication or reporting behaviour relied on by 300 clinics. |
| Performance / load / stress testing | Validate booking < 3 s, availability propagation < 5 s and stability at 2,500 concurrent users (3-5x normal) for the Monday peak; stress beyond peak to find the breaking point. |
| Security & compliance (HIPAA) testing | Verify encryption in transit/at rest, authentication, RBAC, audit logging, vulnerability scanning, and that no PHI leaks into SMS content or logs; mandatory for 1.2M regulated records. |
| Usability & accessibility testing | Confirm WCAG 2.1 AA conformance and usability across ages, digital-literacy levels and assistive technologies, on desktop and mobile. |
| Compatibility testing | Validate behaviour across major browsers and mobile devices (responsive) and against both Google Calendar and Microsoft Outlook. |
| Data-migration validation | Verify record counts, field mappings, transformations and referential integrity before and after migrating 1.2M records; prevent a repeat of the prior data-loss incident. |
| SMS failover / fallback testing | Confirm graceful degradation, retries, queueing and alerting when the third-party SMS vendor is slow or unavailable, given this first-time business-critical integration. |
| User acceptance testing (pilot) | 20 representative pilot clinics (basic and API-integrated) validate real-world workflows and sign off acceptance criteria before full rollout. |

### Feature test priorities

Priorities are assigned by a risk rating (likelihood x business impact). Critical/High items are P1 (deepest coverage, gating), Medium items P2, Low items P3:

| **Feature / Area** | **Likelihood** | **Impact** | **Risk Level** | **Test Priority** |
| --- | --- | --- | --- | --- |
| SMS reminders (new third-party vendor) | High | High | Critical | P1 |
| Data migration (1.2M patient records) | Medium | Critical | Critical | P1 |
| Peak-load performance (2,500 users) | High | High | Critical | P1 |
| Real-time availability (concurrency) | Medium | High | High | P1 |
| Authentication & permissions (HIPAA/PHI) | Low | Critical | High | P1 |
| Calendar integration (Google/Outlook) | Medium | Medium | Medium | P2 |
| Redesigned UI (accessibility, responsive) | Medium | Medium | Medium | P2 |
| Administrative reporting | Low | Low | Low | P3 |

P1 failures cause lost appointments, compliance breaches or data loss and block go-live. Because clinics range from basic to API-integrated, P1 coverage spans both the UI and the public API; the pilot prioritizes the highest-risk features first.

### Testing objectives

- Stability: zero open Sev-1/Sev-2 defects at go-live; existing functionality protected by regression.
- Performance: booking < 3 s, SMS delivered < 2 min, availability updates propagate < 5 s, stable at 2,500 concurrent users.
- Data integrity: 100% record-count and field reconciliation across the migration, with a tested rollback.
- Security & compliance: full HIPAA adherence - encryption, RBAC, audit trails, no PHI in SMS; zero critical/high vulnerabilities at release.
- Usability & accessibility: WCAG 2.1 AA conformance on desktop and mobile, and successful pilot UAT sign-off.
- Compatibility: consistent behaviour across supported browsers, devices and calendar providers, for both basic and API-integrated clinics.

### Tools and frameworks

- Unit/integration: Jest (frontend) and PyTest (backend services).
- API & contract: Postman, REST Assured and Pact for SMS, calendar and availability endpoints.
- End-to-end / UI: Selenium and Cypress for cross-browser automation.
- Performance / load / stress: Apache JMeter and k6 for peak-traffic simulation.
- Accessibility: axe-core and Google Lighthouse for WCAG checks; screen-reader (NVDA/VoiceOver) manual passes.
- Security: OWASP ZAP and dependency scanning, plus manual HIPAA control review and penetration testing.
- SMS: Twilio test/sandbox environment for reminder and fallback scenarios.
- Cross-device/browser: BrowserStack (desktop + iOS/Android).
- Test & defect management: Jira with the Xray plugin for end-to-end traceability; CI via GitHub Actions/Jenkins.

---

## 2. Traceability matrix

| **Requirement ID** | **Requirement Description** | **Test Case ID(s)** | **Coverage Status** | **Priority / Risk** |
| --- | --- | --- | --- | --- |
| REQ-01 | Schedule Appointment: patient books an appointment, completing within 3 seconds (UC: Schedule Appointment) | TC-01, TC-02, TC-03 | Covered | P1 - High |
| REQ-02 | Real-time provider availability displayed and updated across sessions within 5 s (UC: View Real-Time Availability) | TC-04, TC-05 | Covered | P1 - High |
| REQ-03 | Automated SMS reminders 24 h and 1 h before, delivered within 2 minutes (UC: Receive SMS Reminder) | TC-06, TC-07, TC-08 | Covered | P1 - High |
| REQ-04 | Sync appointments to Google Calendar and Outlook (UC: Sync Appointment to Calendar) | TC-09, TC-10 | Covered | P2 - Medium |
| REQ-05 | Update or cancel an existing appointment with confirmation (UC: Update or Cancel Appointment) | TC-11, TC-12 | Covered | P1 - High |
| REQ-06 | Authenticate user; role-based access enforced (UC: Authenticate User) | TC-13, TC-14 | Covered | P1 - High |
| REQ-07 | Migrate 1.2M patient/appointment records with full integrity (UC: Migrate Appointment Data) | TC-15, TC-16, TC-17 | Covered | P1 - High |
| REQ-08 | Redesigned UI meets WCAG 2.1 AA across desktop and mobile (UC: Display Updated UI) | TC-18, TC-19, TC-31, TC-32 | Covered | P2 - Medium |
| REQ-09 | Provider notified when a new appointment is booked (UC: Notify Provider) | TC-20 | Covered | P2 - Medium |
| REQ-10 | System sustains peak load of 2,500 concurrent users | TC-21, TC-22 | Covered | P1 - High |
| REQ-11 | Patient data encrypted; HIPAA controls enforced (no PHI in SMS/logs) | TC-23, TC-24 | Covered | P1 - High |
| REQ-12 | Release can be rolled back to v2.x with data restored (UC: Rollback Release) | TC-25, TC-43 | Covered | P1 - High |
| REQ-13 | Administrators generate appointment reports by date range (UC: Generate Appointment Reports) | TC-26 | Covered | P3 - Low |
| REQ-14 | Administrators manage user access permissions (UC: Manage User Access Permissions) | TC-27, TC-28 | Covered | P1 - High |
| REQ-15 | Validate data integrity after migration - counts and field mappings (UC: Validate Data Integrity) | TC-29, TC-30 | Covered | P1 - High |
| REQ-16 | EDGE: SMS fallback when third-party vendor is unavailable (retry/queue/alert) | TC-33 | Covered | P1 - High |
| REQ-17 | EDGE: prevent double-booking under concurrent requests | TC-37 | Covered | P1 - High |
| REQ-18 | EDGE: calendar sync correct across timezone / DST boundaries | TC-34 | Partial | P2 - Medium |
| REQ-19 | EDGE: migrate edge records (hyphenated names, invalid/landline phones, null consent) | TC-38, TC-39 | Covered | P2 - Medium |
| REQ-20 | API-integrated clinics: public API contract compatibility | TC-40 | Covered | P2 - Medium |
| REQ-21 | Basic clinics: full booking flow on mobile and desktop browsers | TC-41 | Covered | P2 - Medium |
| REQ-22 | Pilot rollout: telemetry and feedback captured from 20 clinics (UC: Monitor Post-Deployment Metrics) | TC-42 | Covered | P1 - High |
| REQ-23 | Generate deployment readiness report from test results (UC: Generate Deployment Readiness Report) | TC-44 | Covered | P2 - Medium |

---

## 3. Annotated test plans

### Feature 1: SMS Reminders (third-party SMS integration)

**Test objective:** Verify automated SMS reminders are generated and delivered 24 h and 1 h before the appointment within 2 minutes of the scheduled send, and that the system degrades gracefully when the third-party provider (Twilio) is unavailable.

**Preconditions:** A confirmed appointment exists with a valid, opted-in mobile number; the Twilio integration and fallback queue are configured; the patient SMS-consent flag is set.

**Edge/variability coverage:** UI vs API booking; invalid/landline numbers; opt-out; timezone; vendor outage; basic vs API-integrated clinics.

### Test cases

| **Step** | **Test Action** | **Expected Result** | **Risk Tag / Notes** |
| --- | --- | --- | --- |
| 1 | Schedule an appointment 25 h ahead and advance the clock to the 24 h mark (UI and API booking). | Reminder SMS generated and delivered within 2 minutes; delivery logged. | [Medium \| Timezone, PHI] Verify scheduler timezone; logs must contain no PHI. |
| 2 | Advance to the 1 h pre-appointment mark. | A second reminder is delivered within 2 minutes. | [Medium \| Edge case] Bookings < 1 h ahead must suppress the 24 h reminder. |
| 3 | Simulate the SMS provider returning errors/timeouts. | Message is queued and retried; on continued failure an alert fires and email fallback triggers; no patient-facing crash. | [Critical \| Integration/Failover] First-time vendor; validate retry limits and queue. |
| 4 | Send a reminder to an invalid or landline number (migrated data). | Failure captured, flagged to the clinic, does not block other reminders. | [High \| Data quality] Edge case from migrated phone numbers. |
| 5 | Patient replies STOP or has opted out. | No further SMS is sent; opt-out recorded. | [High \| Compliance: TCPA/HIPAA] Consent must be respected. |

### Feature 2: Real-Time Doctor Availability

**Test objective:** Verify availability is accurate and a clinic update propagates to all active patient sessions within 5 seconds, preventing double-bookings.

**Preconditions:** At least one provider with a synchronized schedule; two concurrent patient sessions; the availability service running.

**Edge/variability coverage:** Concurrency/double-booking; API vs UI updates; cache staleness; peak load.

### Test cases

| **Step** | **Test Action** | **Expected Result** | **Risk Tag / Notes** |
| --- | --- | --- | --- |
| 1 | Clinic marks a slot unavailable while two patients view that provider. | Both patient sessions reflect the change within 5 seconds. | [High \| Concurrency/Cache] Race condition / stale-cache risk. |
| 2 | Two patients attempt to book the same open slot simultaneously. | Only one booking succeeds; the other gets "slot no longer available". | [Critical \| Concurrency] Double-booking prevention - high impact. |
| 3 | Provider availability updated via an external EHR API integration. | Availability reflects the external update within 5 seconds. | [High \| Integration: API vs UI] Validate for API-integrated clinics. |
| 4 | Load the availability view under peak concurrency (2,500 users). | Availability renders within SLA with no stale data. | [High \| Performance] Tie to performance tests TC-21/TC-22. |

### Feature 3: Calendar Integration (Google & Outlook)

**Test objective:** Verify confirmed appointments sync to Google Calendar and Outlook and that updates and cancellations reflect correctly, including across time zones.

**Preconditions:** A confirmed appointment; valid Google and Outlook test accounts with OAuth; calendar integration enabled.

**Edge/variability coverage:** Desktop vs mobile; Google vs Outlook; reschedule/cancel; timezone/DST.

### Test cases

| **Step** | **Test Action** | **Expected Result** | **Risk Tag / Notes** |
| --- | --- | --- | --- |
| 1 | Patient selects "Add to Calendar" and authorizes Google Calendar (desktop and mobile). | Correct event (title, time, location, provider) created in Google Calendar. | [Medium \| Security/OAuth] Token handling; expose minimal PHI. |
| 2 | Repeat the sync to Microsoft Outlook. | Equivalent event created in Outlook. | [Medium \| Compatibility] Validate iCal/ICS formatting. |
| 3 | Patient reschedules the appointment in NexaCare. | Synced event updates to the new time. | [Medium \| Sync] Update propagation; avoid duplicate events. |
| 4 | Patient cancels the appointment. | Calendar event removed or marked cancelled. | [Low \| Edge case] Event manually deleted by the user. |
| 5 | Book across a timezone / DST boundary. | Event time is correct in the patient's local calendar. | [Medium \| Timezone/DST] Conversion correctness. |

### Feature 4: Redesigned User Interface (responsive & accessible)

**Test objective:** Verify the redesigned UI is usable, accessible (WCAG 2.1 AA) and renders correctly across desktop and mobile for users of varying digital literacy.

**Preconditions:** v3.0 UI deployed to pre-prod; device/browser matrix and assistive tech available; axe-core/Lighthouse configured.

**Edge/variability coverage:** Mobile vs desktop; assistive technology; low-literacy users; low bandwidth.

### Test cases

| **Step** | **Test Action** | **Expected Result** | **Risk Tag / Notes** |
| --- | --- | --- | --- |
| 1 | Load the booking flow on desktop (Chrome, Edge, Firefox) and mobile (iOS Safari, Android Chrome). | Layout is responsive and fully functional on all targets; no broken elements. | [High \| Compatibility: mobile vs desktop] Core cross-platform risk. |
| 2 | Run axe-core/Lighthouse accessibility audits on key pages. | WCAG 2.1 AA: no critical violations; contrast, labels and focus order pass. | [High \| Accessibility/Compliance] WCAG 2.1 AA requirement. |
| 3 | Navigate the booking flow using only keyboard and a screen reader. | All actions reachable; ARIA labels announced correctly. | [Medium \| Accessibility] Low-vision / motor-impaired users. |
| 4 | Complete booking as a low-digital-literacy / first-time persona. | Booking completed unaided; errors and help are clear. | [Medium \| Usability] Diverse clinic user base. |
| 5 | Simulate a slow 3G mobile connection. | Pages load within acceptable time with graceful loading states. | [Medium \| Performance: mobile] Rural/low-bandwidth clinics. |

### Feature 5: Authentication & Access Permissions

**Test objective:** Verify secure authentication and role-based access so patients, clinic staff and admins reach only permitted data, protecting 1.2M HIPAA records.

**Preconditions:** Accounts exist for patient, clinic-staff and admin roles; auth service and RBAC configured; audit logging enabled.

**Edge/variability coverage:** API vs UI; horizontal and vertical privilege escalation; session expiry; PHI auditing.

### Test cases

| **Step** | **Test Action** | **Expected Result** | **Risk Tag / Notes** |
| --- | --- | --- | --- |
| 1 | Log in with valid and invalid credentials via UI and API. | Valid succeeds; invalid rejected with a generic error; lockout after repeated failures. | [Critical \| Security/HIPAA; API vs UI] Brute-force protection. |
| 2 | Patient attempts to access another patient's records via the API (IDOR). | Access denied; the attempt is audit-logged. | [Critical \| Security/PHI] Horizontal privilege escalation. |
| 3 | Clinic staff attempts an admin-only action. | Denied per RBAC; an audit entry is created. | [High \| Security] Vertical privilege escalation. |
| 4 | Leave a session idle; reuse an expired token. | Idle session expires (15 min); expired token rejected. | [High \| Security] Session management. |
| 5 | Read/write PHI as an authorized user. | Every PHI access is recorded immutably in the audit log. | [High \| Compliance/HIPAA] Audit trail. |

### Feature 6: Data Migration Validation

**Test objective:** Verify 1.2M patient/appointment records migrate from MySQL v2.x to PostgreSQL v3.0 with full integrity, correct transformations and no data loss.

**Preconditions:** Migration dry run executed in staging; source snapshot available; validation scripts ready.

**Edge/variability coverage:** Hyphenated/unicode names; invalid/landline phones; null consent; orphaned records; rollback.

### Test cases

| **Step** | **Test Action** | **Expected Result** | **Risk Tag / Notes** |
| --- | --- | --- | --- |
| 1 | Reconcile source vs target record counts per clinic batch. | 100% count match for all tables. | [Critical \| Data integrity] Prevents data loss. |
| 2 | Validate transformed fields: phone to E.164, datetime to UTC. | All sampled records correctly transformed; 0 invalid E.164. | [High \| Data quality] Edge: mixed legacy formats. |
| 3 | Migrate edge-case names (hyphenated, multi-part, unicode). | Names preserved; split logic correct. | [Medium \| Edge case] Name parsing. |
| 4 | Process invalid/landline phones and null SMS consent. | Routed to exceptions; consent defaults to false. | [High \| Compliance] Consent never assumed. |
| 5 | Verify referential integrity (appointments to patients/providers). | No orphaned records after migration. | [High \| Data integrity] Foreign-key consistency. |
| 6 | Execute a migration rollback from the pre-cutover snapshot. | v2.x restored; counts match pre-migration baseline. | [Critical \| Recoverability] Rollback path validated. |

---

## 4. Deployment checklist

### Pre-deployment tasks

| **Task Description** | **Owner / Team** | **Est. Time** | **Dependencies / Notes** |
| --- | --- | --- | --- |
| QA sign-off on all P1/P2 test cases in staging | QA Lead (R), Release Mgr (A) | T-3 days | Critical/blocker defects closed; regression green. |
| Pilot UAT sign-off (20 clinics) | Product Owner (R/A), QA Lead (C) | T-2 days | Acceptance criteria met; pilot feedback triaged. |
| Full DB backup & verified v2.x snapshot | DBA (R), DevOps Lead (A) | T-1 day, 2 h | Restorable snapshot is a prerequisite for rollback. |
| Data-migration dry run, 100% reconciliation | Data Eng Lead (R), QA (C) | T-1 day, 3 h | Counts and checksums match in staging. |
| SMS (Twilio) readiness & fallback verified | Integrations Eng (R), DevOps Lead (A) | T-1 day | Credentials, rate limits, fallback queue confirmed. |
| Rollback plan rehearsed & approved | Release Manager (R/A) | T-1 day, 1 h | Timed rehearsal within the window budget. |
| Monitoring dashboards & alerts configured | SRE (R), DevOps Lead (A) | T-1 day | Golden-signal alerts and on-call schedule active. |
| HIPAA / security review & pen-test sign-off | Security Officer (R/A) | T-2 days | Encryption, RBAC, audit logging, 0 critical vulns. |
| Stakeholder & clinic communications sent | Support Lead (R), Product (A) | T-2 days | Maintenance window (Sat 12-6 AM EST) announced. |
| Go/No-Go decision (Gate 0) | Release Manager (A) + leads (C) | T-1 day | All criteria reviewed and signed off. |

### Deployment window tasks

| **Task Description** | **Owner / Team** | **Time Window** | **Prerequisites** |
| --- | --- | --- | --- |
| Enable maintenance mode / read-only banner | DevOps Lead (R) | 12:00-12:10 AM EST | Comms sent; window started. |
| Final pre-cutover backup & snapshot | DBA (R) | 12:10-12:40 AM | Maintenance mode active. |
| Deploy v3.0 application build | DevOps + Release Eng (R) | 12:40-1:30 AM | Backup complete; build artifact signed. |
| Execute data migration (1.2M, batched) | Data Eng Lead (R) | 1:30-3:30 AM | Deployment succeeded; dry run validated. |
| Post-migration validation (counts/checksums) | Data Eng + QA (R) | 3:30-4:15 AM | Migration completed without fatal errors. |
| GATE 1: data-integrity go/no-go | Release Mgr (A), Data Eng + QA (C) | 4:15-4:20 AM | 100% reconciliation required to proceed; else rollback. |
| Smoke test critical paths (login, book, availability, SMS, calendar) | QA Lead (R) | 4:20-5:00 AM | Gate 1 passed. |
| Enable new features via feature flags | DevOps Lead (R) | 5:00-5:20 AM | Smoke tests green. |
| Cache warm-up & final health check | SRE (R) | 5:20-5:40 AM | Features enabled. |
| GATE 2: final go-live go/no-go | Release Mgr (A); QA, DevOps, Security, Product (C) | 5:40-5:45 AM | Smoke green, 5xx < 0.5%, rollback ready, stakeholder sign-off. |
| Exit maintenance mode / go live | Release Manager (R/A) | 5:45-6:00 AM | Gate 2 = GO. |

### Post-deployment tasks

| **Task Description** | **Owner / Team** | **Est. Time** | **Dependencies / Notes** |
| --- | --- | --- | --- |
| Intensive monitoring (golden signals, error rates) | SRE on-call (R), DevOps Lead (A) | First 48 h | Heightened thresholds; on-call active. |
| Pilot clinic telemetry & feedback review | Product Owner (R), QA (C) | Daily, week 1 | Informs broader rollout decision. |
| Validate Monday-morning peak performance | SRE + QA (R) | First Mon 8-10 AM | Confirm < 3 s booking at 2,500 users. |
| SMS delivery success-rate audit | Integrations Eng (R) | First 24 h | Confirm > 98% delivery within 2 min. |
| Data-integrity spot checks on live records | Data Eng (R) | First 24 h | No corruption or loss after migration. |
| Support escalation & incident-triage readiness | Support Lead (R/A) | Week 1 | Runbook and escalation path live. |
| Generate deployment readiness / post-release report | Release Manager (R/A) | End of week 1 | Compiled from results and telemetry; shared with stakeholders. |
| Post-release review & rollout sign-off | Release Mgr + Product Owner (A) | End of week 1 | Decision to expand beyond the pilot. |

### Go / No-Go criteria

Go/No-Go is decided at firm checkpoints by the Release Manager (accountable), with QA, DevOps, Security and Product as sign-off parties. Gate 0 (T-1 day) authorizes the window; Gate 1 (~4:15 AM) gates on data-integrity; Gate 2 (~5:40 AM) gates go-live. Any unmet criterion is an automatic No-Go and triggers rollback or reschedule.

**Technical:**

- All smoke tests pass; zero open Sev-1/Sev-2 defects.
- Data migration reconciled at 100% record count with matching checksums (Gate 1).
- HTTP 5xx error rate < 0.5% and booking response < 3 s in smoke tests.
- A verified, rehearsed rollback snapshot is available.

**Operational:**

- Monitoring, alerting and on-call coverage are active.
- Support team is briefed and staffed for go-live.

**Business:**

- Stakeholder and pilot sign-off obtained (Product Owner, Security Officer).
- Deployment completes within the approved Sat 12-6 AM EST window; otherwise No-Go and reschedule.

---

## 5. Rollback strategy

### Conditions that trigger rollback

A rollback to v2.x is triggered if any of the following occur during or shortly after deployment:

- Data corruption, loss, or migration validation failure (record counts or checksums do not match) - Gate 1 failure.
- Critical (Sev-1) functional failure in booking, availability or authentication that blocks clinics.
- HTTP 5xx error rate sustained above 2%, or booking response time exceeding agreed thresholds.
- Mass SMS delivery failure, or any PHI exposure / HIPAA breach.
- Severe performance degradation making the platform unusable under load.
- A confirmed security incident introduced by the release.
- Inability to complete and validate the deployment within the approved 6-hour window.

### Rollback procedure

| **Step** | **Description** | **Responsible Team / Person** |
| --- | --- | --- |
| 1 | Declare rollback (Release Manager decision); invoke the runbook; open the incident bridge. | Deployment / Release Manager |
| 2 | Re-enable maintenance mode to stop new traffic and writes. | DevOps / SRE |
| 3 | Stop and disable v3.0 services and feature flags. | DevOps |
| 4 | Restore the v2.x database from the pre-cutover snapshot. | DBA / Data Engineering |
| 5 | Revert the application to the previous v2.x build and configuration. | DevOps / Release |
| 6 | Re-point integrations (SMS, calendar) to v2.x endpoints or disable new ones. | Integrations |
| 7 | Validate restored data (record counts, checksums) and run smoke tests. | QA / Data Engineering |
| 8 | Exit maintenance mode and confirm v2.x is healthy. | SRE |
| 9 | Notify stakeholders, clinics and support of restored service and next steps. | Release Manager / Support |

**Fallback options to preserve patient data and clinic operations:**

- Patient-data preservation: the pre-cutover snapshot and write-ahead logs are the source of truth; no v3.0 write is irreversible because migration runs forward-only with the v2.x copy retained until sign-off.
- Degraded (read-only) mode: if only the new features fail, disable them via feature flags and keep core v2.x booking available rather than taking the whole platform down.
- Clinic continuity: clinics are given an interim manual booking/telephone procedure and a status-page link so operations continue during recovery; queued SMS reminders are held and re-sent after restore.
- Partial rollback: if a single integration (e.g., SMS) is the only failure, roll back just that component while retaining the rest of the release.

### Communication plan

If rollback is executed, communication follows a predefined escalation path:

- The incident bridge is opened immediately; the Release Manager coordinates and is the single point of decision.
- Internal: leadership, QA, DevOps, Product and Support are notified via the incident channel within 15 minutes.
- External: affected clinics and patients are informed via in-app banner, email and the public status page, in plain language with expected resolution.
- Regulatory: if any PHI exposure is suspected, the Security/Compliance Officer initiates the HIPAA breach-notification process.
- A post-incident report is shared with all stakeholders within 48 hours.

### Estimated rollback time

Best case (application/config revert or single-component partial rollback): approximately 30-45 minutes.

Worst case (full database restore and revalidation of 1.2M records): approximately 2-3 hours.

Both scenarios are designed to complete within the approved 6-hour maintenance window, with the rollback decision made no later than 3:30 AM EST to preserve sufficient recovery time.

---

## 6. Data migration plan

### Systems involved

- Source system: Legacy NexaCare Scheduler v2.x - relational database (MySQL 5.7) holding current clinic, provider, patient and appointment data.
- Destination system: NexaCare v3.0 platform - new normalized schema on PostgreSQL 15, supporting SMS, real-time availability and calendar-integration services.

### Field mapping

| **Source Field** | **Destination Field** | **Transformation / Logic** |
| --- | --- | --- |
| patient_id (INT) | patient_uuid (UUID) | Generate UUID; keep a legacy_id to uuid lookup to preserve referential integrity. |
| first_name, last_name | full_name (+ first_name, last_name) | Concatenate for display name; retain components; handle hyphenated/multi-part/unicode names. |
| phone (varchar, mixed) | phone_e164 | Normalize to E.164 (+1XXXXXXXXXX) for SMS; flag invalid/landline numbers for review. |
| sms_optin (Y/N) | sms_consent (boolean) | Map Y to true, N to false; default false when null (consent is never assumed). |
| appt_datetime (local, no tz) | appointment_start_utc (timestamptz) | Convert clinic local time to ISO-8601 UTC using the clinic timezone. |
| provider_id (INT) | provider_uuid (UUID) | Generate UUID; map via lookup; preserve clinic association. |
| status (1/2/3 codes) | status (enum) | Map 1=booked, 2=cancelled, 3=completed; route unknown codes to exceptions. |
| email (varchar) | email (citext) | Trim and lowercase; validate format; null if invalid. |

### Migration volume

Approximately 1.2 million patient records plus associated appointment, provider and consent data across 300 clinics (estimated 20-25 GB). Migration runs in clinic-scoped batches of about 50,000 records (roughly 24 batches) to allow checkpointing, parallelism and isolated error handling, with full logging and a resumable checkpoint after each batch. Estimated migration runtime in the window: ~2 hours, leaving buffer for validation.

### Validation steps

- Pre-migration sanity checks

Run per-table row counts, null/format checks on key fields (phone, email, dates), duplicate detection, and a PHI inventory in the source. Confirm the v2.x snapshot is captured and restorable before any load. Example: SELECT COUNT(*), COUNT(DISTINCT patient_id) FROM v2.patients;  and a check for rows where phone is null or malformed.

- Migration logs review

Review per-batch logs for transformation errors, rejected records and timing. Any fatal error halts the run and triggers investigation before proceeding; each batch records a checksum and a processed/ rejected count.

- Post-migration field and count verification

Reconcile source vs destination counts to 100%, compare row-level checksums (sampled and key-field), and verify transformations on a representative sample. Example validation scripts:

- Count reconciliation: SELECT COUNT(*) FROM v2.patients  must equal  SELECT COUNT(*) FROM v3.patients (per clinic batch).
- Checksum: compare MD5/hash aggregates of key columns between source and target per batch.
- Format: SELECT COUNT(*) FROM v3.patients WHERE phone_e164 !~ '^\+1\d{10}$';  must return 0 (excluding flagged exceptions).
- Referential integrity: every v3.appointments.patient_uuid must exist in v3.patients (0 orphans).
- Timezone: spot-check that appointment_start_utc equals the source local time converted by the clinic timezone.
- Exception reporting and remediation plan

All rejected, duplicate or invalid records are written to an exceptions report owned by Data Engineering. Each item is triaged, corrected at source where possible, and re-migrated; the migration is signed off only when the exception backlog is resolved or formally accepted.

**Migration rollback option:**

- If validation fails (counts/checksums mismatch, excessive exceptions, or integrity errors), the migration is rolled back by restoring the pre-cutover v2.x snapshot; the forward-only design keeps v2.x intact until sign-off, so no patient data is lost.

**Privacy & compliance (HIPAA):**

- PHI is encrypted in transit and at rest throughout migration; access is least-privilege and fully audit-logged; anonymized data is used for dry runs; and a PHI inventory and reconciliation evidence are retained for the compliance audit.

---

## 7. Post-deployment monitoring summary

### Key metrics to monitor

| **Metric** | **Threshold / SLA** | **Monitoring Tool / Method** |
| --- | --- | --- |
| System uptime / availability | >= 99.9% monthly (alert < 99.9%) | Synthetic checks & uptime monitoring (Pingdom / CloudWatch). |
| Booking response time (KPI: Schedule) | < 3 s end-to-end; API P95 < 500 ms (alert > 3 s) | APM transaction tracing (Datadog / New Relic). |
| Booking success rate | > 99% (alert < 98%) | APM + application logs. |
| SMS delivery success rate (KPI: SMS) | >= 98% delivered < 2 min (alert < 95%) | Twilio delivery callbacks + integration dashboard. |
| Calendar sync error rate (KPI: Calendar) | < 1% sync errors (alert > 2%) | Integration logs + Sentry. |
| Availability propagation (KPI: Real-time) | < 5 s across sessions (alert > 5 s) | Custom event-latency metric in APM. |
| HTTP 5xx error rate | < 0.5% (alert > 0.5%) | Error tracking (Sentry) + APM. |
| Authentication / login success rate | > 99% (alert on spike in failures) | Auth-service logs & SIEM dashboard. |
| UI page-load time (desktop & mobile) | < 2.5 s (alert > 4 s) | Real-user monitoring (RUM) / Lighthouse CI. |
| Concurrent active users | Capacity to 2,500 (scale trigger > 2,000) | Real-time session metrics (Grafana). |
| DB connection-pool utilization | < 80% (alert > 80%) | Database monitoring (CloudWatch / pg_stat). |
| CPU / memory utilization | < 80% sustained (alert > 85%) | Infrastructure monitoring (Grafana / CloudWatch). |

### Monitoring responsibilities

| **Team / Person** | **Monitoring Role** | **Frequency / Schedule** |
| --- | --- | --- |
| DevOps / SRE on-call | Owns golden-signal dashboards (Grafana/Datadog), responds to PagerDuty alerts, escalates | Continuous (24/7); heightened first 48 h. |
| QA team | Reviews pilot telemetry, error trends (Sentry) and defect signals | Daily during pilot / week 1. |
| Integrations Engineer | Monitors SMS (Twilio) and calendar sync dashboards and delivery KPIs | Daily, first 2 weeks. |
| Support team | First-line triage of client-reported issues (Zendesk); escalates per runbook | Continuous, business hours. |
| Data Engineering | Monitors data-integrity and migration-related metrics | Daily, week 1. |
| Security Officer | Reviews auth-failure and audit-log anomalies (SIEM) for HIPAA | Daily, week 1; then weekly. |
| Product Management | Reviews KPI/adoption reports to guide rollout | Weekly summary report. |

### Response plan

**Alerting & escalation:**

- Threshold breaches generate automated alerts (PagerDuty/Slack) routed to the on-call SRE, who acknowledges and triages within the agreed SLA (Sev-1: 15 min).
- Incidents are classified by severity (Sev-1 critical to Sev-4 minor); Sev-1/Sev-2 open an incident bridge and notify leadership.
- Predefined runbooks guide diagnosis (SMS vendor failure to enable fallback; saturation to scale out; auth-failure spike to security review).
- If instability or data-risk criteria are met, the Release Manager evaluates rollback.

**Reporting cadence:**

- Real-time dashboards (always on); an automated daily health report during the first week; a weekly KPI report to stakeholders thereafter; an incident report within 48 h of any Sev-1/Sev-2.

**Feedback channels (pilot & full rollout):**

- Pilot: a dedicated feedback form, weekly pilot-clinic check-ins, and in-app feedback feed the rollout go/no-go decision.
- Full rollout: support tickets (Zendesk), the public status page and continued KPI dashboards drive ongoing improvement and capacity planning.

---

## 8. Non-functional requirements and performance plan

### A. NFR definition

The following non-functional requirements define specific, measurable targets across performance, scalability, reliability, recoverability, availability, latency, security and compliance.

Validation methods: performance and scalability targets are validated with load, stress and soak testing (JMeter / k6); reliability and recoverability with backup/restore and rollback drills; security with penetration testing, OWASP ZAP scans and HIPAA control review; and all targets are continuously monitored after release through the golden-signal dashboards and SLOs, with alerting and periodic optimization.

| **NFR Category** | **Specific Requirement** | **Target Value** | **Measurement Method** |
| --- | --- | --- | --- |
| Performance | Appointment booking completion time | < 3 seconds | API response-time monitoring (APM, P95); load testing. |
| Performance | Provider availability search / load time | < 2 seconds | APM transaction tracing. |
| Performance | Calendar sync (Google/Outlook) completion | < 10 seconds | Integration timing logs. |
| Performance | SMS send processing throughput | >= 500 messages/min sustained | Queue throughput metrics; load testing. |
| Scalability | Normal concurrent users | 500 users | Real-time session metrics. |
| Scalability | Peak concurrent users (Mon 8-10 AM) | 2,500 users | Load-testing validation (JMeter / k6). |
| Scalability | Headroom above peak without redesign | Scale to ~3,500 users (1.4x) | Stress testing. |
| Reliability | System uptime | >= 99.9% monthly | Uptime monitoring; downtime <= ~43 min/month. |
| Reliability | Mean time to recover (MTTR) | <= 1 hour | Incident metrics. |
| Recoverability | Recovery Point Objective (RPO) | <= 15 minutes | Backup frequency + restore drill. |
| Recoverability | Recovery Time Objective (RTO) | <= 1 hour | Rollback / restore rehearsal. |
| Availability | Service availability (booking & availability APIs) | 99.9% | Synthetic monitoring & health checks. |
| Latency | SMS reminder delivery | < 2 minutes from scheduled send | SMS provider delivery callbacks. |
| Latency | Real-time availability propagation | < 5 seconds across sessions | Event-latency metric in APM. |
| Security | Data encryption (in transit & at rest) | TLS 1.2+ / AES-256 | Security scan & configuration audit. |
| Security | Authentication | MFA for staff/admin; secure sessions; 15-min idle timeout | Penetration testing & auth logs. |
| Security | Authorization (RBAC) | Least-privilege roles enforced | Access-control test cases. |
| Security | Audit logging | 100% of PHI access logged, immutable, retained >= 6 years | Log review & HIPAA audit. |
| Security | Vulnerability management | 0 critical/high vulnerabilities at release; scan each build | OWASP ZAP / dependency scanning. |
| Compliance | HIPAA compliance | Full adherence; no PHI in SMS or logs | Compliance review & audit. |
| Maintainability | Deployment / rollback capability | Rollback completed <= 1 hour within the window | Rollback rehearsal. |

### B. Golden signals monitoring plan

The Four Golden Signals are monitored continuously to detect user-facing problems early in the healthcare-scheduling workflow:

| **Golden Signal** | **Metric Name** | **How to Measure** | **Target/ Threshold** | **Business Impact** |
| --- | --- | --- | --- | --- |
| Latency | API response time (P95) | APM transaction tracing on booking/availability endpoints | < 500 ms (booking < 3 s end-to-end) | Slow responses cause booking abandonment and clinic workflow delays. |
| Traffic | Active concurrent users / requests per second | Real-time session and request metrics | Baseline 500; alert when > 2,500 | Drives capacity planning and scaling triggers for peak periods. |
| Errors | HTTP 5xx error rate | Error tracking + APM | < 0.5% | Failed requests mean lost or failed bookings and patient frustration. |
| Saturation | DB connection-pool & CPU utilization | Database and infrastructure monitoring | < 80% | Resource exhaustion causes timeouts and cascading booking failures. |

### C. Peak traffic preparation

The Monday-morning peak (8-10 AM EST) drives 1,500-2,500 concurrent users - roughly 3-5x normal load. The monitoring strategy adapts so expected, healthy peak behaviour does not raise false alarms while genuine degradation is still caught quickly.

- Adjusted thresholds: relax CPU, database-utilization and API-response alerts during the peak window, while keeping error-rate and saturation alerts sensitive, since those signal real trouble.
- Infrastructure adjustments: enable horizontal auto-scaling ahead of the window, add read replicas for availability/booking queries, raise the connection-pool ceiling, pre-warm caches and CDN, and scale the SMS worker pool.
- Degradation monitoring: watch P95/P99 latency trend, error-rate slope, queue depth and saturation in real time on a dedicated peak dashboard with anomaly detection.
- If performance drops below acceptable levels: auto-scale out further, shed non-critical load (queue/throttle SMS and reporting), serve cached availability, and - if SLAs are breached and stability is at risk - use feature flags to gracefully degrade non-essential features before considering rollback.

| **Metric** | **Normal Threshold** | **Peak Period Threshold** | **Reasoning** |
| --- | --- | --- | --- |
| Database CPU | > 85% alert | > 95% alert | Higher utilization is expected and acceptable at peak; avoids false alarms. |
| API response time | > 2 s alert | > 4 s alert | Accept slightly degraded latency under load rather than page on expected spikes. |
| Concurrent users | Alert > 800 | Alert > 2,500 | Peak baseline is far higher; only alert above provisioned capacity. |
| Connection-pool utilization | > 70% | > 85% | Allow higher pool use at peak while still guarding against exhaustion. |
| App-server auto-scaling | Scale at > 70% CPU | Pre-scaled; scale at > 60% CPU | Proactive scaling before the window prevents latency build-up. |

### D. Prioritized performance optimization plan

Optimizations are prioritized using the 80/20 rule: the two workflows that dominate traffic and business value - appointment booking and real-time availability - are optimized first, since improving them resolves the majority of performance risk. Work is organized across the database, application and infrastructure layers.

**1. Database layer (highest priority):**

- Add and tune indexes on appointment and availability queries (provider_id, appointment_start_utc, status) - the dominant query path; validated via query-plan analysis and load tests, monitored through DB query metrics.
- Introduce read replicas to offload availability reads from the primary; validated under peak load tests, monitored via replica lag and pool utilization.
- Connection pooling and query optimization to prevent exhaustion at 2,500 users; monitored via the connection-pool saturation golden signal.

**2. Application layer:**

- Cache real-time availability with a short TTL and event-based invalidation to cut repeated database hits while honouring the < 5 s propagation SLA; validated by cache-hit-ratio and latency tests.
- Make SMS sending fully asynchronous via a message queue with retries and fallback, so reminders never block booking; validated by load and failover tests, monitored via queue depth and delivery success.
- Reduce payload sizes and add pagination on availability/listing endpoints; validated through APM response-time tracking.

**3. Infrastructure layer:**

- Horizontal auto-scaling of stateless app servers driven by CPU/latency, pre-scaled before the Monday peak; validated by peak load tests, monitored via traffic and saturation signals.
- CDN for the redesigned UI static assets to reduce origin load and improve mobile page load; validated via Lighthouse and cache metrics.
- Load balancing across application instances for even distribution and resilience; monitored via per-instance utilization.

Each optimization is validated in pre-production load tests (JMeter / k6) against the defined NFR targets and continuously monitored after release through the golden-signal dashboards, ensuring improvements hold under real Monday-peak conditions.
