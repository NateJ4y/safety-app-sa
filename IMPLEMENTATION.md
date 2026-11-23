# GBV Safety App — Full Implementation Plan

**Document type:** Product, technical architecture, security and delivery specification  
**Status:** Implementation baseline  
**Prepared:** 23 November 2026  
**Initial release strategy:** Prototype → controlled pilot → South African national rollout

---

## 1. Product overview

### 1.1 Purpose

Build a survivor-centred gender-based violence (GBV) safety application that helps a person request urgent assistance, notify trusted contacts, share relevant location information, and connect to verified nearby helpers where an approved response network exists.

The app must make it easier to ask for help without creating a false promise that emergency services or a responder will always be available. It must be privacy-first, resilient under poor network conditions, accessible, discreet, and designed around the safety and agency of the person using it.

### 1.2 Product principles

1. **Survivor control:** The person requesting help remains in control wherever immediate safety permits.
2. **Safety before features:** Avoid features that expose a survivor, reveal their location, or create unsafe expectations.
3. **Trust and verification:** Nearby helpers must be vetted, trained for their defined role, and subject to ongoing review.
4. **Reliable escalation:** A failed push notification must not silently become a failed emergency request.
5. **Privacy by design:** Collect the minimum data required, restrict access, encrypt sensitive data, and define retention periods.
6. **Transparency:** Clearly communicate what has been sent, to whom, and whether anyone has acknowledged it.
7. **Low-bandwidth operation:** Core SOS actions must remain usable on modest devices and unreliable connections.
8. **Human-centred language:** Use calm, direct, non-judgmental language. Never blame or pressure a survivor.

### 1.3 Explicit non-goals for the first release

- Replacing police, ambulance, hospitals, shelters, courts, or established crisis organisations.
- Guaranteeing a physical rescue or guaranteed responder arrival time.
- Publicly displaying survivor locations or incidents.
- Open, unverified volunteer access to live incidents.
- Automated legal, medical, or psychological diagnosis.
- Public social feeds, community gossip, or user-to-user discovery.
- Recording audio continuously or secretly by default.
- Using demo incidents or fabricated people in production.

---

## 2. Users, roles and permissions

### 2.1 Survivor / person requesting assistance

Can:
- Register and manage their own account, subject to a safe onboarding flow.
- Add, verify, edit, reorder, and remove trusted contacts.
- Configure discreet safety preferences and emergency behaviour.
- Trigger, cancel, or resolve an SOS request.
- View the status of their own active and previous incidents, subject to retention rules.
- Choose whether optional audio evidence is captured or shared, with clear consent and local-law review.
- Request account export or deletion, subject to legally required preservation.

### 2.2 Trusted contact

Can:
- Accept or decline an invitation.
- Verify their contact channel.
- Receive an alert when the survivor triggers an SOS and the relevant permissions allow it.
- View only the information explicitly shared for that incident.
- Acknowledge an alert and communicate through the approved workflow.
- Never browse the survivor's general location history, profile, or other incidents.

### 2.3 Verified nearby helper / responder

Can:
- Apply for access and complete identity, suitability, training, and role checks.
- Set availability and service area.
- Receive a limited incident offer when eligible and authorised.
- Accept or decline an incident.
- See only the minimum location and incident information required for the assigned response.
- Update response status and record a factual outcome.
- Be suspended or removed by authorised operations staff.

A helper is not automatically an emergency professional. The app must clearly distinguish community helpers from accredited emergency services and must never imply that a volunteer has police, medical, or statutory authority unless verified.

### 2.4 Operations administrator

Can:
- Review and manage responder applications and verification status.
- Monitor operational queues and delivery failures.
- Manage incidents only where their assigned duties require it.
- Suspend accounts, investigate misuse, and handle safety escalations.
- Review audit records under controlled access.
- Configure service areas, escalation policies, and approved message templates.

### 2.5 Support / safeguarding officer

A restricted role for authorised staff handling survivor support, complaints, safeguarding concerns, and referrals. Access must be case-based, logged, and limited to the minimum necessary information.

### 2.6 Permission model

Use server-enforced role-based access control (RBAC), supplemented by record ownership and incident assignment checks. Hiding a button in the interface is not access control.

| Resource | Survivor | Trusted contact | Assigned helper | Operations admin |
|---|---|---|---|---|
| Own profile | Read/write | No | Own responder profile | Limited administration |
| Survivor location | Own, when enabled | Only when shared for active incident | Only for assigned incident | Restricted, justified access |
| Trusted contacts | Own list | No | No | No routine access |
| Incident details | Own incidents | Shared incident subset | Assigned incident subset | Operationally scoped |
| Audio evidence | Own, subject to consent | Only if explicitly shared | No default access | Restricted case access |
| Verification records | Own application status | No | Own status | Verification team |
| Audit logs | No | No | Own relevant actions only | Restricted audit role |

---

## 3. Core product journeys

### 3.1 Survivor onboarding

1. Open the app and see a concise explanation of what it does and what it cannot guarantee.
2. Choose a safe sign-up method.
3. Verify the chosen contact channel.
4. Set a safe display name and optional profile details.
5. Review location, notification, and microphone permissions separately, with plain-language explanations.
6. Add at least one trusted contact where possible.
7. Send an invitation and wait for the contact to accept and verify.
8. Configure optional safety preferences.
9. Complete a guided test that does not dispatch a real emergency alert.
10. Land on the safety dashboard.

Do not force a survivor to disclose a detailed abuse history to use the SOS feature.

### 3.2 Trusted contact invitation

1. Survivor enters a phone number or supported contact method.
2. Server creates a time-limited invitation.
3. Contact receives a neutral invitation that avoids revealing sensitive information in lock-screen previews.
4. Contact verifies the channel and accepts.
5. Survivor sees the contact's verified status.
6. Survivor can revoke access at any time.

### 3.3 SOS activation

1. Survivor activates SOS through a clear, accessible control.
2. The app displays a short confirmation step only if the chosen safety mode requires it; avoid unnecessary friction in a genuine emergency.
3. Client obtains the best available current location and records its accuracy and timestamp.
4. Backend creates an incident using an idempotency key.
5. Backend queues alerts to eligible trusted contacts and, where enabled, the verified responder network.
6. Push notification is attempted; fallback channels such as SMS are used according to the configured policy.
7. The app shows delivery and acknowledgement states accurately.
8. A responder may accept the incident and update their status.
9. Survivor can cancel or mark the incident resolved.
10. The system records a minimal, tamper-evident event history.

**Important:** Do not claim an alert was delivered merely because the app queued it. Show distinct states for queued, sent, delivered where technically confirmed, acknowledged, and failed.

### 3.4 Responder journey

1. Applicant submits required information and consent.
2. Verification team reviews identity and eligibility using an approved process.
3. Applicant completes role-specific safeguarding and response training.
4. Administrator approves, rejects, or requests more information.
5. Approved helper sets availability and coverage area.
6. Eligible incident is offered with minimum necessary details.
7. Helper accepts or declines.
8. On acceptance, access is scoped to that incident.
9. Helper updates status: acknowledged, en route, arrived / contact established, unable to assist, or completed.
10. Helper submits a factual outcome note; no speculative or judgemental language.
11. Access to the incident expires when no longer needed.

### 3.5 Operations journey

- Review responder applications and expiring checks.
- Monitor incident queue, failed notifications, and unresolved escalations.
- Review access logs and unusual activity.
- Handle reports of misuse, harassment, or unsafe conduct.
- Suspend access rapidly when a safety concern is raised.
- Maintain referral directory and operational coverage information.
- Produce aggregated, de-identified operational reporting.

---

## 4. Functional requirements

### 4.1 Authentication and account safety

- Secure sign-up and sign-in.
- Phone or email verification based on supported channel.
- Strong session management and secure token storage.
- Optional biometric unlock where supported by the platform.
- Device/session list and remote session revocation.
- Rate limiting and abuse protection.
- Account recovery designed to avoid exposing the survivor to an unsafe shared device or inbox.
- No sensitive details in push notification previews by default.

### 4.2 Safety dashboard

- Prominent SOS action.
- Clear trusted-contact status.
- Current safety settings and permission status.
- Access to emergency guidance and verified support resources.
- No unnecessary content or distracting social features.
- Offline state and last successful sync clearly indicated.

### 4.3 SOS and incident management

- Create incident, capture timestamp and optional location.
- Idempotent request creation to prevent duplicate incidents from retries.
- Dispatch alerts in parallel using a durable queue.
- Retry transient delivery failures with bounded backoff.
- Escalate to fallback channels when configured and lawful.
- Track delivery, acknowledgement, responder acceptance, and resolution separately.
- Allow survivor cancellation and safe resolution.
- Record an immutable event trail.
- Provide an operations intervention path for delivery failures.

### 4.4 Location

- Request location only when needed and explain why.
- Capture current location at SOS activation; avoid continuous tracking by default.
- Store coordinates with accuracy, timestamp, and source.
- Handle denied permission, stale GPS, low accuracy, and unavailable network.
- Allow survivor to share a manually selected safe location where appropriate.
- Restrict location visibility to the active incident's authorised recipients.
- Never expose live location through public URLs or guessable identifiers.

### 4.5 Audio capture

Audio recording is optional and must be subject to a dedicated legal, safety, platform-permission, and consent review before release.

Requirements:
- No background or continuous recording by default.
- Explicit, understandable consent and clear recording state.
- Explain who can access the recording and for what purpose.
- Encrypt before upload where feasible; use private object storage and short-lived access links.
- Resumable upload for poor connectivity.
- Record metadata such as creation time and upload status without altering the original file.
- Define retention, deletion, legal hold, and access procedures.
- Do not promise admissibility as evidence.
- Provide a no-audio SOS path that works fully.

### 4.6 Notifications and communication

- Push notifications through Firebase Cloud Messaging (FCM) or an equivalent provider.
- Pluggable SMS provider for fallback.
- Neutral message templates with minimal sensitive information.
- Delivery receipts where available.
- Retry policy and dead-letter handling.
- Notification preferences for non-emergency content.
- Emergency notifications must follow a separate, clearly explained policy.

### 4.7 Responder verification

- Application workflow and status tracking.
- Identity and eligibility checks via approved providers/processes.
- Training completion and renewal records.
- Role and geographic coverage assignment.
- Availability controls.
- Complaint and incident review workflow.
- Suspension, revocation, and re-verification.
- No responder access until verification and training gates are passed.

### 4.8 Support directory

- Curated, verified organisations and emergency resources.
- Service area, hours, languages, contact methods, and last verification date.
- Clear distinction between official emergency services, professional organisations, and community resources.
- Admin review and expiry reminders.
- Offline access to a small set of essential guidance where possible.

### 4.9 Admin console

- Role-restricted dashboard.
- Responder verification queue.
- Active incident operations view.
- Notification and integration health.
- Safety reports and complaints.
- Resource directory management.
- Audit log viewer.
- Aggregate reporting with privacy thresholds.
- No unrestricted search across survivor records.

---

## 5. Incident lifecycle and data model

### 5.1 Incident states

Use explicit, server-owned state transitions:

`created → dispatching → alerted → acknowledged → en_route → arrived → resolved`

Additional terminal or exception states:
- `cancelled`
- `delivery_failed`
- `unable_to_assist`
- `expired` (only under a documented policy)

Not every incident must pass through every state. A trusted contact acknowledging an alert must not be confused with a responder accepting the incident.

### 5.2 Suggested core entities

- `profiles`
- `user_roles`
- `trusted_contacts`
- `trusted_contact_invitations`
- `responder_profiles`
- `responder_verifications`
- `responder_training_records`
- `responder_availability`
- `incidents`
- `incident_events`
- `incident_assignments`
- `incident_recipients`
- `notification_jobs`
- `notification_attempts`
- `media_assets`
- `support_resources`
- `safety_reports`
- `admin_actions`
- `consent_records`
- `audit_logs`

### 5.3 Key data rules

- Use UUID primary keys and server-generated timestamps.
- Store times in UTC; render in the user's local timezone.
- Use explicit foreign keys and indexes for ownership, status, and queue operations.
- Separate sensitive data from ordinary profile data where practical.
- Avoid storing unnecessary narrative details.
- Do not put coordinates, phone numbers, tokens, or sensitive notes in analytics events or application logs.
- Use soft deletion only where required for operational/legal reasons; define what is retained and why.
- Apply database-level row security and server-side checks.

---

## 6. Technical architecture

### 6.1 Recommended stack

| Layer | Recommendation | Responsibility |
|---|---|---|
| Prototype | FlutterFlow | Validate journeys and interaction patterns quickly |
| Mobile app | Flutter | iOS and Android application |
| Backend | Supabase | Auth, PostgreSQL, Edge Functions, realtime where appropriate |
| Database | PostgreSQL | Core transactional data, constraints, row-level security |
| Push | Firebase Cloud Messaging | Push delivery |
| SMS | Pluggable provider | Fallback alert channel |
| Maps/location | Platform location APIs + suitable map provider | Location capture and map display |
| Media | Private encrypted object storage | Audio and approved attachments |
| Admin portal | React + TypeScript | Restricted operations interface |
| Monitoring | Error tracking + structured operational metrics | Reliability and incident diagnosis |

The prototype must not be mistaken for a production emergency-response service. Production release requires tested backend enforcement, real notification integrations, verification operations, safeguarding processes, and legal review.

### 6.2 High-level flow

```text
Flutter mobile app
   ├── Supabase Auth
   ├── Secure API / Edge Functions
   │      ├── PostgreSQL + Row Level Security
   │      ├── Incident orchestration
   │      ├── Durable notification queue
   │      ├── Push provider (FCM)
   │      ├── SMS provider adapter
   │      ├── Responder matching / assignment
   │      └── Private media storage
   └── Location / maps services

React admin portal
   └── Same authenticated backend, with strict admin permissions
```

### 6.3 Backend design

- All privileged operations run server-side.
- Never expose service-role keys in a mobile app or browser bundle.
- Use Edge Functions or a dedicated backend service for SOS creation, dispatch, responder assignment, and privileged admin actions.
- Use a durable queue or database-backed job pattern for notifications.
- Make jobs idempotent and safe to retry.
- Keep provider adapters replaceable to avoid vendor lock-in.
- Separate development, staging, and production environments.
- Apply database migrations through version control.

### 6.4 Reliability requirements

- SOS creation must be idempotent.
- Notification dispatch must survive app termination after the incident is created.
- Use bounded retries with exponential backoff and jitter.
- Persist each provider attempt and its result.
- Detect jobs that exceed processing time and move them to a dead-letter/review queue.
- Surface degraded service and fallback-channel status to operations staff.
- Test duplicate taps, app restarts, lost connectivity, delayed GPS, expired sessions, and provider outages.
- Define service-level objectives only after pilot data exists; do not invent response-time guarantees.

---

## 7. Security, privacy and safeguarding

This product handles highly sensitive personal and location data. Security and safeguarding are release blockers, not later enhancements.

### 7.1 Baseline controls

- TLS for all network traffic.
- Encryption at rest for database backups and object storage.
- Strong access control and least privilege.
- Row Level Security policies tested for every sensitive table.
- Short-lived signed media URLs.
- Secrets held in managed secret storage.
- Multi-factor authentication for privileged staff.
- Admin access logging and periodic review.
- Rate limiting, bot protection, and suspicious activity detection.
- Secure mobile storage for tokens and sensitive local data.
- Dependency scanning, secret scanning, and security review before release.
- Backup, restore, and incident response procedures.

### 7.2 Threat model

Explicitly assess:
- Abusive partner or household member with access to the survivor's phone.
- Shared devices and lock-screen notification previews.
- Stalker attempting to infer location or account activity.
- Malicious or compromised trusted contact.
- Impersonating or predatory responder.
- Insider misuse by an administrator.
- SIM swap, account takeover, phishing, and stolen devices.
- Notification provider outage or delayed delivery.
- Network interception, replay, duplicate SOS, and API abuse.
- Data breach, accidental export, and insecure analytics.

### 7.3 Survivor safety safeguards

- Discreet app naming/icon and quick-exit behaviour should be evaluated with survivor advocates; do not implement deceptive behaviour without usability and safety testing.
- Avoid revealing sensitive app content in push previews.
- Provide a clear way to revoke trusted contacts and sessions.
- Avoid persistent location sharing.
- Do not allow helpers to browse or contact survivors outside assigned incidents.
- Provide report/block/escalation paths.
- Never expose incident details in public links.
- Avoid language that blames, pressures, or guarantees outcomes.
- Conduct safety testing with relevant experts and compensated lived-experience reviewers.

### 7.4 South African legal and policy work

Before real-user pilot, obtain qualified legal and safeguarding review covering at least:
- Protection of Personal Information Act (POPIA) and applicable regulations.
- Lawful basis, consent, purpose limitation, minimisation, access, correction, retention, deletion, and operator agreements.
- Cross-border hosting and subprocessors.
- Audio recording and interception laws, including the circumstances in which recording is lawful.
- Child safeguarding and age-related obligations.
- Mandatory reporting duties where applicable.
- Occupational, volunteer, and responder liability.
- Emergency-service coordination and use of official service names.
- Evidence handling, disclosure requests, and lawful preservation.
- Terms of service, privacy notice, consent records, and breach response.

Do not treat this document as legal advice. The organisation must appoint accountable privacy and safeguarding leads and obtain professional advice before collecting live survivor data.

---

## 8. UX and visual design direction

### 8.1 Experience goals

- Calm, respectful, clear, and dependable.
- Extremely clear SOS hierarchy.
- Low cognitive load under stress.
- Large touch targets and readable typography.
- Strong contrast and screen-reader support.
- Short, plain-language instructions.
- Fast first meaningful render.
- Support for small screens and lower-end Android devices.

### 8.2 Main screens

1. Welcome and safety explanation
2. Sign up / sign in / verification
3. Permission education
4. Survivor safety dashboard
5. Trusted contacts list
6. Add / invite contact
7. Safety settings
8. SOS confirmation / activation
9. Active incident status
10. Incident history
11. Support directory
12. Responder application
13. Responder verification status
14. Responder dashboard
15. Incident offer and response view
16. Admin dashboard
17. Verification queue
18. Incident operations view
19. Reports / safeguarding queue
20. Privacy, consent, and account controls

### 8.3 Accessibility

Target WCAG 2.2 AA for the web admin portal and apply equivalent mobile accessibility practices:
- Semantic labels and screen-reader announcements.
- Dynamic text sizing.
- Keyboard navigation for web.
- Reduced-motion preference.
- Colour is never the only status indicator.
- Error messages explain recovery steps.
- SOS interaction works with assistive technology.
- Test with users with disabilities.

---

## 9. Performance and low-connectivity strategy

- Keep the initial app bundle and dashboard lightweight.
- Lazy-load non-critical screens and large media.
- Compress and appropriately size images.
- Avoid autoplay video in the app.
- Cache only non-sensitive static content and approved support resources.
- Do not cache active incident location or sensitive content in an insecure way.
- Queue safe non-emergency actions while offline.
- Clearly state that a queued SOS has not yet been transmitted.
- Retry when connectivity returns, while avoiding duplicate incidents.
- Provide SMS fallback only through a tested, approved backend path.
- Measure startup time, crash-free sessions, API latency, notification delivery, and upload completion.

---

## 10. Analytics and reporting

Collect only operationally necessary, privacy-preserving metrics.

Potential metrics:
- Onboarding completion and drop-off.
- Trusted-contact invitation acceptance.
- SOS request creation success.
- Dispatch attempts and confirmed delivery rates by channel.
- Acknowledgement and responder acceptance rates.
- Delivery failure causes.
- App crash-free sessions and performance.
- Responder verification turnaround.
- Support-resource directory freshness.

Never send survivor names, phone numbers, exact coordinates, audio, incident narratives, or other direct identifiers to general analytics platforms. Apply minimum group sizes and de-identification to reports. Do not use analytics to profile survivors or infer sensitive behaviour.

---

## 11. Testing and quality gates

### 11.1 Automated tests

- Unit tests for state transitions, permissions, validation, and retry logic.
- API integration tests for authentication and authorisation.
- RLS tests for every role and record ownership scenario.
- Idempotency tests for repeated SOS requests.
- Notification adapter tests with provider mocks.
- Upload/resume tests for large audio files if enabled.
- Mobile widget and navigation tests.
- Admin portal component and end-to-end tests.

### 11.2 Security tests

- Attempt cross-user reads and writes.
- Attempt access to unassigned incidents.
- Test expired and revoked signed URLs.
- Test role escalation and forged client claims.
- Test brute force, replay, and duplicate requests.
- Inspect logs and analytics for sensitive data leakage.
- Conduct independent penetration testing before broad release.

### 11.3 Human testing

- Survivor-centred usability review.
- Safeguarding expert review.
- Responder workflow simulations.
- Low-end Android and poor-network testing.
- Screen-reader and accessibility testing.
- Tabletop exercises for notification outage, responder misconduct, data breach, and false alert.

### 11.4 Release blockers

Do not launch a live emergency feature until:
- Real notification delivery and fallback are tested end to end.
- Role and row-level security tests pass.
- Responder verification and safeguarding processes are operational.
- Privacy and legal reviews are signed off.
- Incident escalation and support coverage are defined.
- The app communicates delivery status truthfully.
- A tested incident response and breach process exists.

---

## 12. Delivery roadmap — 14 weeks

The schedule is a planning baseline. Dependencies, procurement, legal review, and pilot readiness may change it.

| Phase | Weeks | Deliverables | Exit criteria |
|---|---:|---|---|
| 0. Discovery and safeguarding | 1–2 | Stakeholder interviews, survivor journey mapping, threat model, legal questions, service blueprint | Product scope and safety risks reviewed |
| 1. UX and prototype | 3–4 | Wireframes, design system, clickable FlutterFlow prototype, usability sessions | Core journeys validated |
| 2. Backend foundation | 5–6 | Environments, auth, schema, RLS, audit events, admin roles | Automated access tests pass |
| 3. Survivor app core | 7–8 | Onboarding, trusted contacts, safety dashboard, SOS creation and incident status | End-to-end incident creation works |
| 4. Dispatch and responder | 9–10 | Queue, FCM, SMS adapter, responder verification, assignment workflow | Simulated dispatch and failure recovery pass |
| 5. Admin, support and hardening | 11–12 | Admin portal, support directory, observability, accessibility, security testing | Critical findings resolved |
| 6. Controlled pilot readiness | 13–14 | Training, operational runbooks, pilot cohort, monitoring, evaluation plan | Formal go/no-go approval |

### 12.1 Pilot approach

- Start with a limited, geographically defined pilot only where there are trained and verified responders and a real operational partner.
- Do not market the pilot as a guaranteed emergency response service.
- Use simulated incidents for technical testing; clearly label them and ensure they cannot trigger real dispatch.
- Establish staffed operational coverage and escalation contacts.
- Gather voluntary feedback with informed consent.
- Review incidents, false alerts, delivery failures, responder conduct, and survivor experience before expanding.

### 12.2 National rollout

Expand by province or service area only when:
- Verified responder capacity exists.
- Support and safeguarding coverage is available.
- Local referral information has been verified.
- Delivery reliability meets the agreed threshold.
- Legal, privacy, and operational reviews are complete.
- Funding and long-term maintenance are secured.

---

## 13. Team and operational responsibilities

Minimum accountable functions:
- Product owner
- Mobile developer
- Backend/database engineer
- Admin portal developer
- UX/accessibility designer
- Security/privacy lead
- Safeguarding lead
- Responder verification and operations lead
- Legal adviser
- Pilot/community partner

One person may cover multiple functions in an early prototype, but high-risk approval and oversight should not be left without a named accountable owner.

---

## 14. Environments, repository and engineering workflow

### 14.1 Repository structure

Suggested monorepo:

```text
gbv-safety-platform/
├── README.md
├── IMPLEMENTATION.md
├── docs/
│   ├── product-requirements.md
│   ├── user-journeys.md
│   ├── threat-model.md
│   ├── privacy-and-safeguarding.md
│   ├── incident-lifecycle.md
│   ├── api-contracts.md
│   ├── data-dictionary.md
│   ├── operations-runbook.md
│   └── test-plan.md
├── apps/
│   ├── mobile/
│   └── admin/
├── supabase/
│   ├── migrations/
│   ├── functions/
│   └── tests/
├── packages/
│   └── shared/
└── .github/
    └── workflows/
```

### 14.2 Branching and commits

- `main`: production-ready code only.
- `develop`: integration branch if the team needs one.
- Feature branches: `feature/<short-description>`.
- Fix branches: `fix/<short-description>`.
- Use pull requests with review and automated checks.
- Never commit secrets, production exports, real survivor data, or unredacted logs.
- Use synthetic test fixtures only in development and mark them clearly.

### 14.3 Environments

- Local development
- Shared development
- Staging / pilot rehearsal
- Production

Each environment must have separate credentials, databases, storage buckets, notification credentials, and access policies. Production data must never be copied into development.

### 14.4 CI/CD

Pipeline should run:
- Formatting and linting
- Static type checks
- Unit and integration tests
- Database migration validation
- RLS and permission tests
- Dependency and secret scanning
- Build verification

Deploy staging automatically after approved merges where practical. Production deployment requires an explicit approval gate and rollback plan.

---

## 15. Definition of done

A feature is not complete merely because its screen looks finished. It is done when:
- The intended user journey works end to end.
- Backend validation and permissions are enforced.
- Loading, empty, error, offline, and success states exist.
- Accessibility has been checked.
- Sensitive data is not exposed in logs or analytics.
- Automated tests cover important behaviour.
- Documentation and operational procedures are updated.
- Product and safeguarding acceptance criteria are met.

---

## 16. Initial implementation backlog

### Epic A — Product and safety foundation
- [ ] Confirm target users, operating model, and pilot geography.
- [ ] Appoint product, privacy, and safeguarding owners.
- [ ] Map survivor and responder journeys with expert input.
- [ ] Complete initial threat model and data inventory.
- [ ] Confirm legal review scope and consent approach.

### Epic B — Design and prototype
- [ ] Establish accessible design system.
- [ ] Design onboarding and permission education.
- [ ] Design SOS and active incident states.
- [ ] Design trusted-contact invitation flow.
- [ ] Prototype responder and admin workflows.
- [ ] Conduct usability reviews.

### Epic C — Backend foundation
- [ ] Create separate development, staging, and production projects.
- [ ] Implement authentication and session controls.
- [ ] Create database schema and migrations.
- [ ] Implement RLS and role-based permissions.
- [ ] Implement audit events.
- [ ] Add automated permission tests.

### Epic D — Survivor app
- [ ] Build onboarding and account controls.
- [ ] Build trusted-contact management.
- [ ] Build safety dashboard.
- [ ] Implement SOS creation with idempotency.
- [ ] Implement incident status and cancellation.
- [ ] Implement offline and poor-network states.

### Epic E — Dispatch and responder operations
- [ ] Implement durable notification jobs.
- [ ] Integrate FCM.
- [ ] Integrate and test SMS fallback.
- [ ] Implement responder application and verification.
- [ ] Implement availability and service-area matching.
- [ ] Implement incident offer, acceptance, and status updates.
- [ ] Implement failed-delivery operations queue.

### Epic F — Admin and support
- [ ] Build restricted admin authentication.
- [ ] Build responder verification queue.
- [ ] Build active incident operations view.
- [ ] Build safeguarding report workflow.
- [ ] Build verified support-resource directory.
- [ ] Build audit review and aggregate reports.

### Epic G — Security, testing and launch
- [ ] Complete legal and privacy review.
- [ ] Complete safeguarding review.
- [ ] Run RLS and penetration tests.
- [ ] Run accessibility and low-connectivity tests.
- [ ] Run operational simulations.
- [ ] Train pilot responders and staff.
- [ ] Approve pilot go/no-go.
- [ ] Review pilot evidence before geographic expansion.

---

## 17. Decisions still requiring formal sign-off

These are deliberate decision gates, not assumptions to silently hard-code:

1. Final product name, public identity, and discreet presentation requirements.
2. Initial pilot geography and operational partner.
3. Whether the first live release includes nearby helpers or only trusted contacts.
4. Responder eligibility, verification provider, training, insurance, and supervision.
5. Exact SMS provider, costs, and fallback rules.
6. Whether audio capture is included at all in the first live release.
7. Age policy and child safeguarding process.
8. Emergency-service coordination and referral protocols.
9. Data retention periods and legal-hold process.
10. Funding, support hours, incident coverage, and expansion criteria.

**Recommended build order:** validate the survivor journey and safety model first; build secure identity, contacts, and incident records second; then implement reliable notification delivery; only then introduce verified responder dispatch. Audio capture and broader rollout remain gated by legal, safeguarding, and operational approval.
