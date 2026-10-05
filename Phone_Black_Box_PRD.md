# Phone Black Box — Product Requirements Document (PRD)

**Document version:** 1.0  
**Date:** 2026-10-01  
**Status:** Draft for implementation  
**Project:** B.Tech Major Project — Phone Black Box  
**Project Team ID:** MP2026CSE229  
**Primary platform:** Android (Flutter application)  
**Future platform:** iOS  
**Backend baseline:** Node.js + Express + MongoDB  
**Push notification baseline:** Firebase Cloud Messaging (FCM)  
**ML baseline:** Python + scikit-learn / XGBoost, deployed on-device  
**Local persistence:** SQLite  
**Location/maps:** Android location APIs + Google Maps integration/linking

> **Purpose of this document:** Turn the existing Phone Black Box synopsis into an implementation-ready product and engineering specification. This document preserves the synopsis' core architecture and objectives while adding concrete requirements, interfaces, data contracts, acceptance criteria, failure modes, security/privacy controls, ML methodology, testing strategy, and a phased implementation plan.

---

## 1. Executive Summary

Phone Black Box is a smartphone-only safety application designed to detect probable road accidents and falls using sensors already available on modern smartphones, provide a short user-cancellable confirmation period, automatically alert pre-registered trusted contacts when the user cannot respond, maintain a rolling pre/post-incident sensor record, and support configurable geofencing.

The project synopsis defines a two-stage detection design:

1. A lightweight threshold-based candidate-event pre-screening stage.
2. A machine-learning classification stage using sensor-fusion features.

When a probable incident is classified, the application opens a short confirmation window. If the user cancels, the event is treated as a false alarm. If the countdown expires, the application captures a precise location, creates an incident record, sends it to the backend, and dispatches notifications to trusted contacts.

The core product is **not** an autonomous replacement for emergency services, a certified accident reconstruction device, a medical device, or a guarantee that a crash will be detected or an alert delivered. The PRD therefore treats network loss, OS background restrictions, disabled permissions, battery exhaustion, damaged hardware, GPS unavailability, and false positives/negatives as first-class failure modes.

The MVP focuses on Android because reliable background sensor monitoring requires platform-specific behavior. Flutter remains the application layer and iOS remains a future target.

---

## 2. Source Synopsis — Requirements Preserved

The supplied 21-page project synopsis establishes the following baseline:

- Continuous monitoring of accelerometer, gyroscope and GPS/location signals.
- Smartphone-only operation; no dedicated wearable or vehicle hardware.
- Two-stage detection: threshold pre-screening followed by ML classification.
- Random Forest / XGBoost as initial classifier candidates.
- A 15–20 second user-cancellable confirmation window.
- Automatic incident alerting when the user does not respond.
- Location-tagged incident records.
- Rolling pre- and post-incident local sensor storage — the "black box".
- Geo-fencing for the user or for a trusted person.
- Flutter/Dart mobile application.
- Node.js/Express backend.
- MongoDB backend database.
- Firebase Cloud Messaging for push notifications.
- SQLite local persistence.
- Python, NumPy, pandas, scikit-learn and optionally XGBoost/PyTorch/TensorFlow for ML.
- Evaluation using detection accuracy, false-positive rate, alert latency, battery consumption, labelled offline data and limited field trials.

The synopsis also identifies three important engineering limitations:

- Threshold-only detection can generate false positives from potholes, hard braking and phone drops.
- Continuous high-frequency sensing can consume substantial battery.
- Existing research is less representative of two-wheeler riding conditions, which is important to the intended Indian context.

These points are treated as design constraints rather than merely literature observations.

---

## 3. Product Vision

### 3.1 Vision

Create an affordable smartphone-based safety layer that can continue monitoring when the user is not actively interacting with the app and can notify trusted people when an incident is suspected and the user cannot respond.

### 3.2 Product principles

1. **Safety over convenience:** a false negative and false positive have different costs; both must be measured explicitly.
2. **Local-first detection:** raw sensor streams should remain on the device by default.
3. **Minimal data sharing:** only incident data needed for alerting, investigation or an explicitly enabled feature should leave the device.
4. **Fail explicitly:** the app must surface disabled sensing, location, notifications, network, battery and contact-delivery problems.
5. **User-controlled monitoring:** monitoring must have clear status, permission explanations and an obvious stop/control mechanism.
6. **Platform-aware implementation:** Android background restrictions are part of the system design, not an implementation detail.
7. **Evidence-driven ML:** the classifier must be evaluated subject-wise and scenario-wise, not only by a random row-level train/test split.
8. **No staged dangerous crashes:** project data collection must not deliberately expose people to real collision risk.

---

## 4. Goals and Non-Goals

### 4.1 MVP Goals

- Detect probable vehicle crash/fall events from accelerometer, gyroscope and location/speed signals.
- Use a low-cost pre-screening stage so ML does not run continuously.
- Use an on-device ML classifier to reduce cloud dependency and latency.
- Show a full-screen countdown before an automatic alert.
- Dispatch an incident to the backend after timeout.
- Deliver alert notifications to registered trusted contacts who have the companion app.
- Keep a configurable rolling sensor buffer locally.
- Persist incidents locally when offline and synchronize when connectivity returns.
- Create and monitor configurable geofences.
- Provide an incident history and basic diagnostics.
- Evaluate recall, precision, false-positive rate, alert latency and battery impact.

### 4.2 Future / Stretch Goals

- SMS/voice fallback through a compliant communication provider.
- Direct integration with emergency-service APIs where legally and technically available.
- Multi-model severity classification.
- Smartphone microphone features, subject to an explicit privacy review.
- V2V/V2X integration.
- Cloud model update pipeline.
- Personalized calibration.
- Federated or privacy-preserving model improvement.
- iOS implementation.
- Optional trusted-contact web dashboard.
- Secure upload of complete black-box incident packages.
- Assisted accident reconstruction visualization.

### 4.3 Non-Goals for the B.Tech MVP

- Certified emergency dispatch.
- Guaranteed accident detection.
- Medical diagnosis or injury severity prediction.
- Law-enforcement-grade evidence preservation.
- Automatic continuous audio/video recording for all journeys.
- Autonomous intervention with a vehicle.
- Consumer-scale multi-region commercial deployment.
- Clinical validation.

---

## 5. Target Users and Personas

### Persona A — Rider / Driver

A person using a motorcycle, scooter or car who wants automatic incident detection without installing dedicated vehicle hardware.

**Primary needs**
- Low battery impact.
- Background operation.
- Very few false alarms.
- Fast trusted-contact notification.
- Simple monitoring status.
- Reliable location sharing.

### Persona B — Pedestrian / General User

A person who wants fall/incident detection while walking or performing normal activities.

**Primary needs**
- Fall detection.
- Ability to configure trusted contacts.
- Optional geofencing.
- Clear distinction between ordinary motion and an incident.

### Persona C — Trusted Contact

A family member, friend or caregiver receiving an alert.

**Primary needs**
- Immediate, understandable notification.
- Incident time.
- Current/last-known location.
- Map link.
- User identity.
- Delivery status.
- Ability to acknowledge an alert.

### Persona D — Project / Operations Administrator

Used only for development, testing and demonstration.

**Primary needs**
- View system health.
- Inspect anonymous/authorized incident metadata.
- View model version.
- Review notification delivery failures.
- Export evaluation logs.

---

## 6. User Stories

### Monitoring

- As a user, I can enable accident monitoring so the application can operate when I am not looking at the screen.
- As a user, I can see whether monitoring is active, degraded or stopped.
- As a user, I am clearly informed when background location/sensor access is required.
- As a user, I can stop monitoring at any time.

### Trusted contacts

- As a user, I can add a trusted contact.
- As a user, I can remove a trusted contact.
- As a contact, I can accept or reject a monitoring relationship.
- As a user, I can test whether a contact can receive alerts.

### Detection

- As a user, the app can identify a probable incident from sensor patterns.
- As a user, I receive an immediate confirmation screen after a probable incident.
- As a user who is safe, I can cancel an alert.
- As a user who is incapacitated, I do not need to interact for the alert to continue.

### Alerting

- As a trusted contact, I receive an alert containing the user's identity, incident type, timestamp and location.
- As a user, I can see that an alert was dispatched.
- As a user, I can view previous incidents.

### Black box

- As a user, the app maintains a rolling local buffer without continuously uploading it.
- As a user, an incident preserves the sensor window surrounding the event.
- As a developer, I can export a diagnostic incident package with appropriate authorization.

### Geofencing

- As a user, I can create a safe zone.
- As a trusted contact, I can configure a zone for a linked user subject to permission.
- As a user/contact, I receive an exit notification when a boundary is crossed.

---

## 7. High-Level Functional Architecture

```text
                    ┌──────────────────────────────┐
                    │          Flutter UI          │
                    │ onboarding / monitoring /    │
                    │ contacts / incident / fence  │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │    Native Android Runtime    │
                    │                              │
                    │ Foreground sensing service   │
                    │ sensor lifecycle / permissions│
                    └───────┬─────────┬────────────┘
                            │         │
                ┌───────────┘         └─────────────┐
                ▼                                   ▼
       ┌─────────────────┐                 ┌──────────────────┐
       │ Sensor adapters │                 │ Location adapter │
       │ accel / gyro    │                 │ GPS / speed /   │
       │ sampling        │                 │ accuracy        │
       └────────┬────────┘                 └────────┬─────────┘
                │                                   │
                └──────────────┬────────────────────┘
                               ▼
                    ┌──────────────────────────────┐
                    │ Rolling Sensor Buffer        │
                    │ SQLite / bounded ring buffer  │
                    └──────────────┬───────────────┘
                                   │
                                   ▼
                    ┌──────────────────────────────┐
                    │ Candidate Pre-Screener       │
                    │ SMV / jerk / gyro / speed    │
                    │ low-cost trigger logic       │
                    └──────────────┬───────────────┘
                                   │ candidate
                                   ▼
                    ┌──────────────────────────────┐
                    │ Feature Extraction           │
                    │ time + signal + context     │
                    └──────────────┬───────────────┘
                                   ▼
                    ┌──────────────────────────────┐
                    │ On-Device ML Classifier      │
                    │ RF / XGBoost (ONNX candidate)│
                    └──────────────┬───────────────┘
                                   │ probable event
                                   ▼
                    ┌──────────────────────────────┐
                    │ Confirmation State Machine   │
                    │ 15–20s countdown             │
                    └────────┬───────────┬─────────┘
                             │cancel      │timeout
                             ▼             ▼
                    ┌──────────────┐  ┌────────────────┐
                    │ False Alarm  │  │ Incident       │
                    │ Log + resume │  │ finalization   │
                    └──────────────┘  └───────┬────────┘
                                              │
                       ┌──────────────────────┴────────────────────┐
                       ▼                                           ▼
            ┌────────────────────┐                       ┌──────────────────┐
            │ Local incident DB  │                       │ Backend API       │
            │ queue/retry        │                       │ Node.js/Express   │
            └────────────────────┘                       └─────────┬────────┘
                                                                    │
                                         ┌──────────────────────────┼──────────────┐
                                         ▼                          ▼              ▼
                                 ┌──────────────┐          ┌──────────────┐ ┌─────────────┐
                                 │ MongoDB      │          │ FCM service  │ │ Audit/logs  │
                                 └──────────────┘          └──────┬───────┘ └─────────────┘
                                                                  │
                                                                  ▼
                                                        Trusted Contact App
```

### Architecture decision

The synopsis presents Flutter plugins for sensor/location access. For the MVP, **continuous accident monitoring should be implemented around a native Android foreground service**, with Flutter used for the UI and orchestration layer.

Reason: Android restricts continuous sensor delivery while apps are in the background, and current Android guidance recommends foreground services for continuous sensor access. Flutter can communicate with the native service through platform channels. This reduces the risk that a purely Dart-side stream stops when the process is backgrounded.

---

## 8. Detection System Requirements

### 8.1 Sensor inputs

Minimum:

- Accelerometer: `ax, ay, az`
- Gyroscope: `gx, gy, gz`
- Location: latitude, longitude, accuracy, timestamp
- Derived speed
- Derived bearing where available

Optional future inputs:

- Linear acceleration
- Gravity vector
- Rotation vector
- Barometer
- Activity-recognition state
- Magnetometer
- Microphone-derived impact cues

### 8.2 Coordinate and normalization requirements

The classifier must avoid dependence on a specific phone orientation.

Required derived signals:

- Signal Magnitude Vector:

  `SMV = sqrt(ax² + ay² + az²)`

- Jerk:

  `jerk(t) = ΔSMV / Δt`

- Angular speed magnitude:

  `omega = sqrt(gx² + gy² + gz²)`

- Orientation/rotation change over an event window.
- Speed delta / relative speed drop.
- Stillness duration after impact.
- Peak values and statistics over the candidate window.

Do not feed raw device-axis values directly into the final classifier without evaluating orientation sensitivity.

### 8.3 Candidate pre-screening

The pre-screener is designed to be extremely cheap.

Candidate event may be triggered when one or more of the following occur within a short rolling window:

- SMV exceeds calibrated threshold.
- Jerk exceeds calibrated threshold.
- Gyroscope magnitude exceeds calibrated threshold.
- Speed changes sharply.
- Impact-like acceleration is followed by abnormal stillness.
- Multiple signals cross their thresholds within a temporal relationship.

Thresholds must be configurable through a versioned configuration rather than hard-coded throughout the application.

### 8.4 Feature window

Recommended starting point for experimentation:

- Pre-event feature context: configurable 3–10 seconds.
- Post-event feature context: configurable 3–10 seconds.
- Final incident archive: larger than model window; default 30 seconds before candidate and 60 seconds after candidate.

These are **engineering starting points**, not research-proven universal values. They must be tuned on validation data.

### 8.5 Initial feature set

| Feature group | Features |
|---|---|
| Acceleration | peak, mean, RMS, standard deviation, percentiles |
| SMV | peak, RMS, area-under-magnitude, duration above threshold |
| Jerk | peak, RMS, duration above threshold |
| Gyroscope | peak angular speed, RMS, orientation delta |
| Speed | pre-event speed, post-event speed, speed drop |
| Temporal | impact duration, post-impact stillness |
| Context | movement state, GPS availability, location accuracy |
| Window statistics | min/max/mean/std/skewness where justified |

Start with interpretable features before moving to deep temporal models.

### 8.6 ML problem formulation

The recommended MVP training dataset should include both positive incidents and realistic negative/nuisance scenarios.

Example labels:

- `NORMAL`
- `WALKING`
- `RUNNING`
- `HARD_BRAKING`
- `POTHOLE_ROUGH_ROAD`
- `PHONE_DROP`
- `FALL`
- `VEHICLE_CRASH`

The production alert policy should not simply map `model_probability > threshold` to "emergency". It should combine:

```text
ML prediction
+ confidence
+ sensor/context consistency
+ post-impact stillness
+ confirmation response
+ location quality
+ monitoring state
```

### 8.7 Model candidates

Primary:

- Random Forest
- XGBoost

Baseline comparison:

- Logistic Regression
- SVM

Future:

- 1D CNN / temporal CNN
- compact LSTM/GRU
- temporal transformer only if justified by the dataset size and latency budget

The project must select the final production model based on subject-wise evaluation, not on one metric.

### 8.8 Model deployment

Recommended pipeline:

```text
Python training
    ↓
Preprocessing pipeline
    ↓
Model + feature schema
    ↓
Validation test
    ↓
Export to ONNX where supported
    ↓
ONNX Runtime on Android
    ↓
Flutter/native inference wrapper
```

Random Forest is supported by sklearn-onnx, which makes ONNX a practical deployment format for a lightweight classical model. The app should ship the exact scaler/transformer + model version together so preprocessing cannot drift from training.

A TFLite implementation can remain an alternative if the team later moves to a neural model.

### 8.9 ML versioning

Every prediction must be traceable to:

- `model_id`
- `model_version`
- feature-schema version
- threshold configuration version
- application version

Do not silently change thresholds or model files in production.

---

## 9. Confirmation Window Requirements

### FR-CW-001

After a positive ML classification, the app must display a full-screen confirmation flow.

### FR-CW-002

Countdown default: 15–20 seconds, configurable.

### FR-CW-003

Primary action:

**I'M OK — CANCEL ALERT**

### FR-CW-004

Secondary information:

- "Possible incident detected"
- countdown timer
- optional vibration
- optional sound
- current location quality
- emergency contacts that will be notified after timeout

### FR-CW-005

If the user cancels:

- mark event as `CANCELLED_FALSE_POSITIVE`
- store a local diagnostic record
- resume normal monitoring
- optionally include the record in an explicitly consented ML feedback workflow

### FR-CW-006

If countdown expires:

- acquire best available location
- finalize event package
- enqueue incident locally
- attempt backend submission
- trigger notification delivery
- mark incident state based on actual delivery outcome

### FR-CW-007

If the UI cannot be displayed because the app is backgrounded, locked or OS-constrained, the service must still preserve the event and use a platform-compatible high-priority notification/full-screen mechanism where permitted. Do not assume that arbitrary background UI launch is always permitted.

---

## 10. Incident State Machine

```text
MONITORING
   |
   | candidate threshold
   v
CANDIDATE
   |
   | ML negative --------------------> MONITORING
   |
   | ML positive
   v
AWAITING_CONFIRMATION
   |              |
   | user OK      | timeout
   v              v
CANCELLED      DISPATCH_PENDING
                  |
             +----+------------------+
             |                       |
        network available       offline
             |                       |
             v                       v
       SENT_TO_BACKEND         LOCAL_QUEUED
             |                       |
             v                       | retry
      NOTIFICATION_PENDING <---------+
             |
       +-----+------+
       |            |
    delivered     failed
       |            |
       v            v
  COMPLETED     RETRY/DEGRADED
```

Backend incident status:

- `CLIENT_DETECTED`
- `CLIENT_QUEUED`
- `RECEIVED`
- `NOTIFICATION_PENDING`
- `NOTIFIED`
- `PARTIALLY_NOTIFIED`
- `NOTIFICATION_FAILED`
- `RESOLVED`
- `CANCELLED_FALSE_POSITIVE`

---

## 11. Alert Delivery Architecture

### 11.1 Important implementation constraint

FCM delivers push messages to app clients. It does **not** directly send an arbitrary SMS or phone call to any phone number.

Therefore the PRD separates:

**Primary MVP**
- Trusted contact installs the Phone Black Box companion app.
- Their device registers an FCM token.
- Backend sends an FCM notification.

**Optional fallback**
- Backend invokes a compliant SMS/voice provider.
- Provider and regulatory requirements are selected later.

This distinction must be preserved in system documentation so the project does not claim that FCM alone can notify arbitrary phone numbers.

### 11.2 Notification payload

Do not put excessive sensitive data in the push payload.

Recommended payload:

```json
{
  "type": "INCIDENT_ALERT",
  "incidentId": "inc_...",
  "subjectDisplayName": "User",
  "incidentType": "VEHICLE_CRASH",
  "occurredAt": "2026-10-01T05:00:00Z",
  "locationAvailable": true,
  "mapUrl": "https://maps.google.com/?q=...",
  "severity": "probable"
}
```

The full incident details are fetched over authenticated HTTPS.

### 11.3 Delivery strategy

For each trusted contact:

1. Create a notification delivery record.
2. Send FCM.
3. Record provider response.
4. Record device acknowledgement when available.
5. Retry transient backend/provider errors.
6. Mark permanent failures separately.
7. Display degraded status to the incident owner.

---

## 12. Black Box Requirements

### 12.1 Purpose

The black box is a bounded rolling sensor history designed to reconstruct what happened around an incident.

### 12.2 Local retention

Default:

- 30 seconds pre-event
- 60 seconds post-event

Configurable later.

### 12.3 Storage

SQLite tables:

- `sensor_samples`
- `location_samples`
- `candidate_events`
- `incident_records`
- `upload_queue`

The rolling buffer must be bounded by:

- time
- row count
- disk storage limit

### 12.4 Local data format

Recommended normalized record:

```json
{
  "timestamp": 1737349200123,
  "ax": 0.12,
  "ay": -0.94,
  "az": 9.71,
  "gx": 0.04,
  "gy": 0.13,
  "gz": 0.21,
  "latitude": 30.3165,
  "longitude": 78.0322,
  "accuracyM": 8.4,
  "speedMps": 12.1
}
```

Actual production storage should use a compact SQLite schema rather than JSON rows to reduce overhead.

### 12.5 Privacy requirement

Raw sensor streams should not be continuously uploaded.

Upload only:

- incident metadata
- required sensor summary
- incident window when explicitly configured/authorized
- diagnostics required for development

---

## 13. Geofencing Requirements

### FR-GF-001

User can create a circular geofence.

### FR-GF-002

Future version may support polygonal zones.

### FR-GF-003

Each geofence contains:

- id
- owner
- monitored user/device
- center latitude/longitude
- radius
- name
- active state
- exit alert enabled
- entry alert optional
- trusted recipients
- createdAt
- updatedAt

### FR-GF-004

The system must use native geofencing APIs rather than continuous GPS polling for geofence-only monitoring whenever practical.

### FR-GF-005

Geofence events must support:

- ENTER
- EXIT
- DWELL (future)

### FR-GF-006

The geofence must tolerate GPS noise.

Recommended policy:

- Require platform geofence event.
- Optionally require repeated location confirmation for high-noise conditions.
- Apply configurable debounce.
- Do not generate repeated exit alerts for the same excursion until the user re-enters or the event is closed.

### FR-GF-007

Geofencing and crash detection share the same alert backend but remain logically independent features.

---

## 14. Background Execution Strategy — Android MVP

### 14.1 Required native components

Android side:

- `SensorManager`
- Foreground service
- Location APIs
- Geofencing APIs
- Notification channels
- WorkManager for non-real-time retry/synchronization
- Boot/restart handling only where legally/policy permissible
- Battery optimization guidance

### 14.2 Flutter integration

Flutter remains responsible for:

- UI
- local state
- navigation
- user settings
- account management
- incident history
- contact management
- geofence UI
- analytics/diagnostics display

Native Android service is responsible for:

- persistent sensor collection
- lifecycle handling
- candidate detection
- wake-up behavior
- reliable local buffering
- coordination with ML inference where necessary

Communication options:

- Flutter platform channels for commands/state.
- Event channel or a controlled stream for status events.
- SQLite as the durable shared application data layer where appropriate.

### 14.3 Android permission model

Expected permissions should be requested progressively and explained before prompting.

Potential permissions:

- motion/sensor access as applicable to platform
- `ACCESS_COARSE_LOCATION`
- `ACCESS_FINE_LOCATION` only when justified
- background location where required and approved
- foreground service permissions
- notifications on Android 13+
- vibration where useful

The app must not request every permission at first launch.

### 14.4 Android 14+

If targeting Android 14/API 34+, foreground services require declared service types and corresponding permissions. The exact service type selection must be validated against the final implementation and Google Play policy before release.

### 14.5 Android 15/16+

The implementation must be re-tested against current Android background/foreground-service behavior before release. OS behavior is a moving dependency.

---

## 15. Battery Strategy

### 15.1 Principle

Do not run the most expensive processing continuously.

Pipeline:

```text
Low-cost sensing
     ↓
cheap threshold gate
     ↓
candidate only
     ↓
feature extraction
     ↓
ML inference
```

### 15.2 Sampling strategy

Starting experimental profile:

| Signal | Initial target | Purpose |
|---|---:|---|
| Accelerometer | 25–50 Hz | impact / motion |
| Gyroscope | 25–50 Hz | rotation |
| GPS | ~1 Hz while active | speed/location |
| Candidate mode | temporarily higher | event characterization |
| Geofence | native hardware-assisted events | low-power boundary detection |

These values are tunable and must be validated on real devices.

### 15.3 Battery benchmark

Target for engineering evaluation:

- Continuous active monitoring should aim for **≤10% additional battery consumption over an 8-hour reference test**, excluding abnormal device conditions.
- Measure at least three device classes.
- Report mean and standard deviation.
- Separate:
  - sensing-only cost
  - location cost
  - geofence-only cost
  - ML candidate-trigger cost
  - notification/backend cost

The project should report measured results even if the target is not achieved.

---

## 16. Backend Requirements

### 16.1 Technology

- Node.js
- Express.js
- MongoDB
- Firebase Admin SDK
- HTTPS
- JWT or equivalent secure session mechanism
- structured logging
- rate limiting
- validation

### 16.2 Responsibilities

- authentication/session management
- trusted-contact relationships
- FCM token registration
- incident ingestion
- incident idempotency
- geofence storage/configuration
- notification dispatch
- notification status tracking
- audit logs
- model/config version metadata
- diagnostics

### 16.3 Backend must not perform real-time accident inference for normal operation

On-device inference is preferred for:

- privacy
- low latency
- offline operation
- reduced network dependency
- reduced cloud cost

Backend inference may exist later for research/validation, but should be a non-critical secondary path.

---

## 17. Data Model

### 17.1 `users`

```text
_id
displayName
email
phone
passwordHash / authProviderId
timezone
status
createdAt
updatedAt
```

### 17.2 `devices`

```text
_id
userId
platform
appVersion
osVersion
deviceModel
fcmToken
monitoringEnabled
sensorCapabilities
lastSeenAt
createdAt
updatedAt
```

### 17.3 `trusted_contacts`

```text
_id
ownerUserId
contactUserId (nullable for pending external contact)
name
phone
email
relationship
status
alertPriority
createdAt
updatedAt
```

### 17.4 `geofences`

```text
_id
ownerUserId
monitoredUserId
name
shape
center
radiusM
polygon
active
notifyOnEnter
notifyOnExit
recipients[]
createdAt
updatedAt
```

### 17.5 `incidents`

```text
_id
clientIncidentId
userId
deviceId
incidentType
modelVersion
thresholdVersion
confidence
severity
occurredAt
detectedAt
location
locationAccuracyM
speedMps
status
cancelledAt
createdAt
updatedAt
```

### 17.6 `incident_windows`

Do not store unbounded raw sensor streams in the primary MongoDB incident document.

Store metadata:

```text
_id
incidentId
storageType
objectKey / localReference
checksum
sampleRate
startAt
endAt
retentionUntil
createdAt
```

### 17.7 `notification_deliveries`

```text
_id
incidentId
recipientId
channel
provider
providerMessageId
status
attemptCount
sentAt
deliveredAt
acknowledgedAt
lastError
createdAt
updatedAt
```

### 17.8 `model_versions`

```text
_id
modelId
version
artifactUri
featureSchemaVersion
thresholdConfigVersion
createdAt
isActive
validationMetrics
```

### 17.9 `audit_logs`

```text
_id
actorUserId
action
resourceType
resourceId
metadata
createdAt
```

---

## 18. API Specification

Base path:

`/api/v1`

### Authentication

```http
POST /auth/register
POST /auth/login
POST /auth/refresh
POST /auth/logout
POST /auth/verify-contact
```

### Device

```http
POST /devices/register
POST /devices/fcm-token
GET  /devices/me
PATCH /devices/me
```

### Trusted contacts

```http
GET    /contacts
POST   /contacts
PATCH  /contacts/:id
DELETE /contacts/:id
POST   /contacts/:id/test-alert
```

### Incidents

```http
POST /incidents
GET  /incidents
GET  /incidents/:id
POST /incidents/:id/ack
POST /incidents/:id/cancel
POST /incidents/:id/retry
```

### Geofences

```http
GET    /geofences
POST   /geofences
PATCH  /geofences/:id
DELETE /geofences/:id
POST   /geofences/:id/test
```

### Monitoring configuration

```http
GET  /monitoring/config
PUT  /monitoring/config
POST /monitoring/status
```

### Diagnostics

```http
POST /diagnostics/client-events
GET  /diagnostics/health
```

### Idempotency

`POST /incidents` must accept a client-generated idempotency key/clientIncidentId.

Repeated upload of the same incident must not create duplicate incidents or duplicate notification sequences.

---

## 19. Example Incident API Request

```json
{
  "clientIncidentId": "c9de8d3d-...",
  "occurredAt": "2026-10-01T05:05:31.421Z",
  "detectedAt": "2026-10-01T05:05:35.902Z",
  "incidentType": "VEHICLE_CRASH",
  "confidence": 0.94,
  "modelVersion": "crash-rf-0.1.0",
  "thresholdVersion": "threshold-0.1.0",
  "location": {
    "latitude": 30.3165,
    "longitude": 78.0322,
    "accuracyM": 7.8
  },
  "speedMps": 13.1,
  "sensorWindow": {
    "startAt": "2026-10-01T05:05:01.421Z",
    "endAt": "2026-10-01T05:06:31.421Z",
    "storageRef": "local://incident/c9de..."
  }
}
```

---

## 20. Authentication and Authorization

### 20.1 Required

Every backend resource must be scoped to an authenticated user.

### 20.2 Authorization rules

- User can read own incidents.
- User can manage own contacts.
- User can manage own geofences.
- Trusted contact can read only incidents explicitly shared with them.
- A trusted contact must not gain unrestricted access to the monitored user's history.
- Admin/test roles are isolated from production personal data.
- Diagnostics must redact identifiers where possible.

### 20.3 Contact verification

A trusted contact relationship should be verified before it can receive real alerts.

Possible MVP flow:

```text
Owner creates contact
        ↓
Invite sent
        ↓
Contact opens app / link
        ↓
Authentication + consent
        ↓
Relationship accepted
        ↓
FCM token registered
        ↓
Test alert
        ↓
Contact becomes ACTIVE
```

---

## 21. Security Requirements

The app handles highly sensitive information:

- precise location
- movement data
- emergency relationships
- potentially health-adjacent incident data
- device identifiers
- authentication data

Security baseline:

- TLS for all network communication.
- No plaintext passwords.
- Argon2id or a strong platform-supported password hashing mechanism if the backend owns passwords.
- Short-lived access tokens + refresh-token protection.
- Secure token storage using Android Keystore / iOS Keychain through platform-secure storage.
- Never hard-code backend secrets or Firebase server credentials.
- Never ship Firebase Admin credentials inside the mobile application.
- Input validation on all backend endpoints.
- Rate limiting on authentication and incident endpoints.
- Audit logs for security-sensitive operations.
- Encryption at rest where infrastructure supports it.
- Signed/verified model artifacts.
- Dependency scanning.
- Secret scanning in CI.
- Production logging must avoid raw location traces and sensor streams unless explicitly required.

### OWASP baseline

Use OWASP MASVS as the mobile security verification baseline, especially:

- MASVS-STORAGE
- MASVS-CRYPTO
- MASVS-AUTH
- MASVS-NETWORK
- MASVS-PLATFORM
- MASVS-CODE
- MASVS-RESILIENCE
- MASVS-PRIVACY

---

## 22. Privacy Requirements

### 22.1 Data minimization

Default data policy:

- Raw sensor data: local only.
- Continuous location: local processing only unless a feature explicitly requires remote sharing.
- Incident location: uploaded after confirmed timeout.
- Incident window: stored locally by default; remote upload is configurable.
- Contact information: stored for notification functionality.
- FCM token: stored only for delivery.

### 22.2 User controls

Provide:

- Enable/disable monitoring.
- List of active permissions.
- List of trusted contacts.
- Geofence list.
- Incident history.
- Delete local incident data.
- Delete account/data where supported.
- Export diagnostic data only through an explicit user/developer action.

### 22.3 India data protection

The PRD should be implemented with the Digital Personal Data Protection Act, 2023 and the notified Digital Personal Data Protection Rules, 2025 considered as the India privacy baseline. Exact legal obligations depend on the final deployment model, data fiduciary/processor roles, user population and service operation; the development team should obtain legal review before public deployment.

The implementation should therefore include:

- clear purpose notices
- consent/permission UX where applicable
- data retention rules
- deletion/withdrawal workflows
- breach-response procedures
- access controls
- processor/vendor inventory
- records of what data is collected and why

---

## 23. Data Retention Policy

Suggested initial policy:

| Data | Default retention |
|---|---|
| Rolling local sensor buffer | bounded, continuously overwritten |
| Cancelled false-alarm metadata | 30 days locally |
| Confirmed incident metadata | 90 days in backend for MVP |
| Full incident sensor window | 30 days unless exported |
| Notification delivery logs | 90 days |
| Application logs | 30 days |
| Security audit logs | longer retention according to deployment policy |

Retention periods must be configurable and documented before real-user deployment.

---

## 24. ML Dataset Strategy

### 24.1 Core problem

Real crashes are rare and ethically difficult to collect. A dataset containing only simulated impacts can produce impressive test accuracy but poor real-world performance.

Therefore the project must use a **multi-source dataset strategy**.

### 24.2 Dataset categories

#### A. Public fall datasets

Useful for the fall branch and nuisance/negative classes:

- SisFall
- FARSEEING
- MobiAct / MobiFall-related datasets
- other public human-activity datasets after licensing review

Important limitation: many fall datasets use body-worn sensors or controlled simulations and are not equivalent to a phone mounted on a moving motorcycle.

#### B. Public vehicle/crash research data

Use published smartphone/vehicle crash experiments when licensing and data access permit.

#### C. Controlled non-crash field data

Collect from the team's own phones:

- normal walking
- running
- sitting
- getting into/out of vehicles
- pocket movement
- phone drop onto safe surfaces
- hard braking under safe driving conditions
- potholes/rough road
- speed changes
- cornering
- acceleration
- phone mounted on handlebar
- phone in pocket
- phone in jacket/bag
- driver vs passenger conditions

#### D. Safe impact proxies

Use controlled, non-human experiments or instrumented lab equipment for high-acceleration patterns. Do **not** stage real vehicle crashes involving people.

#### E. Synthetic augmentation

Where scientifically justified:

- sensor noise
- orientation rotation
- sampling-rate jitter
- missing GPS windows
- dropout
- timestamp jitter
- controlled amplitude scaling

Never let synthetic variants cross train/test subject boundaries in a way that leaks samples.

### 24.3 Dataset split

Primary evaluation split:

```text
Train subjects     → 70%
Validation subjects → 15%
Test subjects       → 15%
```

The exact proportions can change with dataset size.

Critical rule:

**Split by person/session/drive, not by random sensor rows.**

Otherwise the same person's movement signature can appear in both train and test and inflate performance.

### 24.4 Evaluation scenarios

At minimum report:

- walking
- running
- normal riding
- acceleration
- hard braking
- pothole
- speed breaker
- turning
- phone pickup
- phone drop
- fall
- simulated/controlled crash-like event

### 24.5 Class imbalance

Report:

- class count
- class weights
- precision/recall/F1
- PR-AUC where applicable
- confusion matrix

Do not rely on accuracy alone.

---

## 25. ML Evaluation Requirements

Required metrics:

- Accuracy
- Precision
- Recall/Sensitivity
- F1
- Specificity
- False-positive rate
- False alarms per hour
- PR-AUC
- ROC-AUC
- inference latency
- model size
- memory use

Safety-relevant metrics:

1. **Recall on genuine incidents**
2. **False alarms per hour/day**
3. **Time from event to user prompt**
4. **Time from prompt timeout to backend receipt**
5. **Notification delivery latency**
6. **Battery cost**

### Initial engineering targets

These are project acceptance targets, not claims about real-world safety:

- Subject-wise test recall: ≥90% for target incident classes.
- Precision: ≥90% on the final held-out test set.
- False alarms: ≤0.1/hour during dedicated nuisance testing.
- Candidate-to-confirmation prompt: ≤3 seconds under normal conditions.
- Confirmation duration: 15–20 seconds.
- Backend receipt after timeout: p95 ≤10 seconds with working mobile connectivity.
- End-to-end notification delivery: p95 ≤15 seconds under a healthy network/provider path.
- Local incident persistence: must succeed even when network is unavailable.

Targets must be revised if the actual dataset distribution makes them statistically inappropriate.

---

## 26. Field Trial Protocol

### 26.1 Participants

Start with a small consenting internal pilot group.

### 26.2 Trial duration

Target:

- multiple sessions per device
- multiple phone models
- multiple mounting positions
- both urban and rural/rough-road conditions where safe
- day/night conditions if relevant

### 26.3 Safety

Do not instruct participants to crash, intentionally fall, or perform dangerous maneuvers.

### 26.4 Ground truth

For safe trials, record:

- session video where consented
- manually labelled event timestamps
- GPS track
- speed
- phone placement
- vehicle type
- road condition
- user activity

Ground-truth labels must be created independently of the classifier output.

---

## 27. Reliability and Failure Modes

### Failure Mode 1 — No network

Expected behavior:

- Local incident is stored.
- Retry queue activates.
- Incident is marked `LOCAL_QUEUED`.
- App shows degraded delivery state.
- Automatic remote notification is attempted when connectivity returns.
- The app must never delete the incident simply because the first upload failed.

### Failure Mode 2 — GPS unavailable

Expected behavior:

- Use last known location if recent and clearly labelled.
- Record accuracy and timestamp.
- If no location exists, alert with `locationAvailable=false`.
- Do not fabricate coordinates.

### Failure Mode 3 — Sensor unavailable

Expected behavior:

- Monitoring status becomes `DEGRADED`.
- User is informed.
- App logs sensor capability error.
- Do not claim continuous protection.

### Failure Mode 4 — App process killed

Expected behavior:

- Android native monitoring architecture must be tested against process death.
- Recovery must be attempted according to OS-supported mechanisms.
- The UI must show last known monitoring state when the app reopens.

### Failure Mode 5 — Battery critically low

Expected behavior:

- Reduce optional work.
- Preserve pending incident data.
- Inform the user that monitoring may be unavailable at very low battery.
- Do not repeatedly wake the device for non-critical work.

### Failure Mode 6 — User force-stops app

A force-stopped Android app may not behave like a normal backgrounded app. Treat force-stop as an explicit user action and surface monitoring as unavailable until the app is reopened/re-enabled.

### Failure Mode 7 — Notification permission denied

The user must see a clear warning that trusted contacts cannot be reliably notified through push without notification capability.

### Failure Mode 8 — False positive

User cancellation ends the alert sequence and records the candidate as a false alarm for evaluation.

### Failure Mode 9 — False negative

Cannot be directly recovered by UI. This is why recall and scenario coverage are core ML metrics.

---

## 28. Notification UX

### User incident screen

```text
------------------------------------
       POSSIBLE INCIDENT DETECTED

              15

      Are you okay?

      [ I'M OK — CANCEL ALERT ]

If there is no response, your
trusted contacts will be notified.

Location: available
Battery: 64%
------------------------------------
```

### Trusted contact alert

```text
PHONE BLACK BOX ALERT

Possible accident detected for:
<Display Name>

Time:
10:35:42 AM

Location:
Available

[ OPEN LOCATION ]

[ ACKNOWLEDGE ]

If you know the person is safe,
contact them directly.
```

Do not use alarmist language that implies certainty when the ML output is only a probability.

---

## 29. Onboarding Flow

```text
Welcome
  ↓
Explain what monitoring does
  ↓
Account creation
  ↓
Add trusted contact
  ↓
Verify trusted contact
  ↓
Explain sensor + location permissions
  ↓
Request required permissions progressively
  ↓
Notification permission
  ↓
Enable monitoring
  ↓
Run sensor diagnostic
  ↓
Send test alert
  ↓
Ready
```

### Sensor diagnostic

Check:

- accelerometer available
- gyroscope available
- location available
- notification permission
- battery optimization constraints
- network
- FCM registration
- trusted-contact status

---

## 30. Monitoring Dashboard

Show:

- Monitoring: `ON / OFF / DEGRADED`
- Sensor status
- Location status
- Network status
- Battery percentage
- Active trusted contacts
- Geofence status
- Last successful health check
- Latest model version
- Test-alert button

Avoid exposing raw accelerometer streams in the standard UI.

---

## 31. Observability

Backend metrics:

- requests/sec
- error rate
- incident ingestion latency
- notification send latency
- notification failure rate
- duplicate incident count
- retry queue size
- active devices
- geofence events
- FCM token invalidation count

Mobile diagnostics:

- monitoring state transitions
- service restarts
- sensor availability
- GPS availability
- candidate count
- ML inference count
- ML inference latency
- confirmation prompts
- cancellation rate
- queued incident count
- synchronization success/failure

Privacy rule: diagnostics must not become a hidden continuous location-tracking system.

---

## 32. Testing Strategy

### 32.1 Unit tests

- SMV calculation
- jerk
- gyro magnitude
- feature extraction
- threshold logic
- classifier input schema
- state machine transitions
- incident idempotency
- geofence debounce
- serialization/deserialization

### 32.2 Integration tests

- native sensor service ↔ Flutter
- SQLite ↔ incident queue
- mobile ↔ backend
- backend ↔ MongoDB
- backend ↔ FCM
- auth ↔ contact relationship
- geofence event ↔ notification

### 32.3 Device tests

At least:

- low-end Android device
- mid-range Android device
- high-end Android device
- multiple Android versions
- multiple sensor vendors if possible

### 32.4 Background tests

Test:

- screen off
- device locked
- app backgrounded
- app swiped from recent apps
- process killed
- battery saver
- airplane mode
- Wi-Fi disconnected
- mobile data disconnected
- GPS disabled
- notification permission denied
- location permission downgraded
- approximate location only

### 32.5 ML tests

- subject-wise holdout
- scenario-wise holdout
- cross-device testing
- noise robustness
- orientation robustness
- missing sensor robustness
- GPS dropout
- class imbalance
- drift analysis

### 32.6 Security tests

- authentication bypass
- IDOR/BOLA
- invalid JWT
- token replay
- rate-limit enforcement
- injection
- insecure storage
- network interception
- leaked Firebase credentials
- excessive API data exposure

Use OWASP MASVS/MASTG as the security verification framework.

---

## 33. Acceptance Criteria

### Feature acceptance

A feature is accepted only when:

1. It works on a physical Android device.
2. It works with screen locked.
3. It logs required failures.
4. It handles network failure.
5. It is covered by tests.
6. It does not expose sensitive data unnecessarily.
7. The behavior is demonstrated in the project evaluation.

### MVP Definition of Done

The MVP is considered complete when:

- [ ] User can register/login.
- [ ] User can add and verify trusted contact.
- [ ] FCM token is registered.
- [ ] Monitoring can be turned on/off.
- [ ] Native background sensing works during screen-off tests.
- [ ] Sensor ring buffer is operational.
- [ ] Threshold pre-screening works.
- [ ] ML model loads on-device.
- [ ] Candidate event produces confirmation UI.
- [ ] Cancel prevents alert.
- [ ] Timeout creates an incident.
- [ ] Incident survives network loss.
- [ ] Backend accepts idempotent incident upload.
- [ ] FCM notification reaches companion contact device.
- [ ] Incident location is displayed.
- [ ] Geofence enter/exit event works in supported conditions.
- [ ] Battery benchmark is documented.
- [ ] ML metrics are reported on held-out subjects.
- [ ] Privacy/security checklist is completed.
- [ ] App does not claim guaranteed emergency response.

---

## 34. Recommended Repository Structure

```text
phone-black-box/
│
├── apps/
│   └── mobile/
│       ├── lib/
│       │   ├── core/
│       │   ├── features/
│       │   │   ├── auth/
│       │   │   ├── monitoring/
│       │   │   ├── incidents/
│       │   │   ├── contacts/
│       │   │   ├── geofencing/
│       │   │   └── settings/
│       │   ├── data/
│       │   ├── domain/
│       │   └── presentation/
│       ├── android/
│       │   └── app/
│       │       └── src/main/
│       │           ├── kotlin/.../
│       │           │   ├── MonitoringService
│       │           │   ├── SensorManagerAdapter
│       │           │   ├── LocationManager
│       │           │   └── NativeBridge
│       │           └── AndroidManifest.xml
│       └── test/
│
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── repositories/
│   │   ├── models/
│   │   ├── middleware/
│   │   ├── routes/
│   │   ├── jobs/
│   │   └── config/
│   └── test/
│
├── ml/
│   ├── data/
│   ├── notebooks/
│   ├── src/
│   │   ├── preprocessing/
│   │   ├── features/
│   │   ├── training/
│   │   ├── evaluation/
│   │   └── export/
│   ├── models/
│   └── reports/
│
├── docs/
│   ├── PRD.md
│   ├── architecture/
│   ├── api/
│   ├── ml/
│   ├── testing/
│   └── privacy/
│
├── scripts/
├── .github/
│   └── workflows/
└── README.md
```

---

## 35. CI/CD

### Mobile CI

- Flutter format
- Flutter analyze
- unit tests
- Android build
- APK/AAB artifact
- dependency audit

### Backend CI

- lint
- unit tests
- integration tests
- build
- security/dependency scan
- container/image scan if Docker is used

### ML CI

- data schema validation
- preprocessing test
- model reproducibility test
- metric regression test
- ONNX export test
- inference equivalence test:

```text
Python prediction == mobile/ONNX prediction
```

within an agreed numerical tolerance.

### Release gates

No production release when:

- model export fails
- permissions are inconsistent
- security scan has critical findings
- incident creation is non-idempotent
- notification pipeline is broken
- crash detection is disabled silently
- the current privacy notice does not match actual data collection

---

## 36. ML Reproducibility Requirements

Every model release must include:

```text
dataset version
code commit
feature schema version
random seed
hyperparameters
training date
train/validation/test subject lists
evaluation metrics
confusion matrix
model artifact checksum
preprocessing artifact checksum
threshold configuration
```

The team must be able to reconstruct how a model was produced.

---

## 37. Model Calibration and Threshold Policy

A classification probability is not automatically a trustworthy probability.

Before production use:

- Evaluate probability calibration.
- Select alert thresholds using validation data.
- Evaluate false-positive burden at the selected threshold.
- Keep thresholds separate from the trained model.
- Version thresholds independently.

Recommended policy:

```text
P(incident) < T_low
    → normal

T_low ≤ P(incident) < T_alert
    → candidate / additional evidence

P(incident) ≥ T_alert
    → confirmation flow
```

This can be refined after field testing.

---

## 38. Concept Drift and Device Variability

The same physical event can produce different sensor traces because of:

- phone model
- sensor calibration
- sampling frequency
- phone orientation
- mounting location
- case
- vehicle type
- rider/body motion
- road surface
- temperature
- GPS availability

Therefore the model evaluation must include:

- cross-device testing
- cross-position testing
- cross-user testing
- cross-scenario testing

Future versions should track drift rather than retraining blindly on every new event.

---

## 39. Geofence vs Accident Monitoring

These are two different power/latency modes.

### Accident monitoring

Requires high-frequency inertial sensing and event detection.

### Geofence monitoring

Should rely on platform geofencing where possible and must not continuously sample high-frequency sensors just to decide whether a boundary was crossed.

This separation is essential for battery efficiency.

---

## 40. Emergency Reliability Model

The application is a chain, not one feature:

```text
Sensor works
   ↓
Detection works
   ↓
Confirmation screen appears
   ↓
User does not cancel
   ↓
Location available
   ↓
Incident stored
   ↓
Network available
   ↓
Backend receives
   ↓
Notification provider accepts
   ↓
Contact device is reachable
   ↓
Contact receives notification
```

The project must measure failures at every stage.

A single "notification latency" number is insufficient.

Recommended dashboard:

| Stage | Metric |
|---|---|
| Detection | event-to-candidate latency |
| ML | inference latency |
| UX | candidate-to-prompt latency |
| Confirmation | timeout accuracy |
| Storage | local persistence success |
| Network | upload success rate |
| Backend | ingestion latency |
| Push | provider acceptance |
| Device | delivery/acknowledgement |
| Overall | end-to-end successful alert percentage |

---

## 41. Security and Safety Threat Model

### Threat: attacker reads another user's incident

Mitigation:
- strict authorization
- resource ownership validation
- opaque IDs
- audit logging

### Threat: stolen FCM token used as identity

Mitigation:
- never treat FCM token as authentication
- require authenticated user/device registration
- rotate/revoke device tokens

### Threat: replaying an old incident

Mitigation:
- server-side timestamps
- idempotency keys
- authenticated request
- nonce/request ID
- clock-skew handling

### Threat: location leakage

Mitigation:
- minimize location sharing
- authenticated fetch
- no public incident URLs by default
- short-lived map/share tokens if public sharing is ever added

### Threat: fake false-positive feedback poisoning ML

Mitigation:
- never automatically add all user cancellations directly to production training.
- route feedback to a curated dataset with labels and quality controls.

### Threat: malicious app modification

Mitigation:
- integrity checks where appropriate
- signed builds
- secure release process
- server-side validation
- avoid trusting client-provided severity as authoritative

---

## 42. Abuse and Misuse Controls

The product can be misused for covert tracking.

Therefore:

- monitoring status must be visible to the monitored user.
- geofence relationships require authorization.
- a trusted contact must not silently enable tracking.
- the app must show who can receive alerts/location.
- contact removal must be easy.
- location sharing must not be hidden.
- no "stealth mode" is part of the product.

---

## 43. Open Engineering Decisions

These decisions should be documented as Architecture Decision Records (ADRs) before final implementation.

### ADR-001 — ML runtime

Options:
- ONNX Runtime
- TFLite
- custom native inference

Recommended experiment:
- export Random Forest to ONNX and benchmark on physical Android devices.

### ADR-002 — Background sensing service

Options:
- pure Flutter background plugin
- native Android foreground service + Flutter UI
- Flutter background isolate with native support

Recommended:
- native Android foreground service + Flutter UI.

### ADR-003 — Alert fallback

Options:
- FCM-only
- FCM + SMS provider
- FCM + SMS + voice provider

MVP:
- FCM companion app.

### ADR-004 — Authentication

Options:
- backend JWT
- Firebase Auth
- another identity provider

MVP:
- choose one and avoid duplicating authentication logic.

### ADR-005 — Remote black-box storage

Options:
- metadata only
- compressed object storage
- MongoDB binary data
- user-controlled export

MVP:
- local by default + explicit upload for selected incidents.

### ADR-006 — Maps

Options:
- Google Maps SDK
- map URL
- OpenStreetMap-based display

MVP:
- location URL + optional Google Maps display.

---

## 44. Phased Implementation Plan

### Phase 0 — Architecture Spike

Deliverables:

- Flutter app skeleton
- native Android sensor foreground service
- accelerometer/gyroscope collection
- GPS collection
- SQLite ring buffer
- basic status screen

Exit criterion:
- sensor data continues while screen is locked on physical devices.

### Phase 1 — Detection Engine

Deliverables:

- SMV
- jerk
- gyro magnitude
- speed delta
- candidate-event logic
- feature extraction
- local logging

Exit criterion:
- reproducible candidate events on controlled recordings.

### Phase 2 — Dataset + ML

Deliverables:

- dataset ingestion
- feature pipeline
- baseline models
- Random Forest/XGBoost benchmark
- subject-wise evaluation
- ONNX export
- mobile inference

Exit criterion:
- mobile prediction matches Python within defined tolerance.

### Phase 3 — Confirmation + Incident

Deliverables:

- confirmation UI
- countdown
- cancel path
- timeout path
- local incident queue
- backend upload
- incident history

Exit criterion:
- airplane-mode incident survives and later synchronizes.

### Phase 4 — Trusted Contacts + FCM

Deliverables:

- contact onboarding
- FCM registration
- backend notification service
- notification delivery records
- test alert

Exit criterion:
- companion device receives a real test alert.

### Phase 5 — Geofencing

Deliverables:

- geofence CRUD
- native geofence registration
- enter/exit events
- contact notification

Exit criterion:
- controlled geofence test produces one debounced event.

### Phase 6 — Hardening

Deliverables:

- background tests
- battery tests
- crash/recovery tests
- security testing
- Play policy review
- privacy review
- release build

Exit criterion:
- MVP Definition of Done fully checked.

### Phase 7 — Evaluation and Dissertation/PPT Material

Deliverables:

- final metrics
- confusion matrices
- battery charts
- latency distributions
- architecture diagrams
- limitations
- future work
- demo script

---

## 45. Recommended Development Order

Do **not** start by building the complete UI.

Build in this order:

```text
1. Native background sensor service
2. Reliable local ring buffer
3. Candidate detection
4. Dataset pipeline
5. Offline ML benchmark
6. On-device ML
7. Confirmation state machine
8. Local incident lifecycle
9. Backend incident API
10. FCM contact notification
11. Geofencing
12. Security/privacy hardening
13. Battery optimization
14. Final UI polish
```

This ordering reduces the risk of spending weeks on UI before proving that the central sensing architecture works on real Android hardware.

---

## 46. Academic Research Contribution

For a B.Tech project, the implementation itself is the primary contribution. The report should avoid claiming a novel accident-detection algorithm unless the experiments actually demonstrate one.

A defensible project contribution is:

> An integrated smartphone-only safety architecture that combines background inertial sensing, lightweight candidate screening, ML-assisted event classification, a user-cancellable confirmation workflow, incident black-box logging, trusted-contact alerting and configurable geofencing, with an evaluation focused on false alarms, latency, battery cost and cross-user/device robustness.

Possible experimental contribution:

- compare threshold-only vs threshold+ML
- compare Random Forest vs XGBoost
- evaluate sensor-fusion vs accelerometer-only
- quantify false alarms under nuisance activities
- quantify orientation/device variability
- measure battery impact
- measure alert latency

---

## 47. Literature and Technical Evidence Review

This section records the main research and platform findings that shaped this PRD.

### 47.1 WreckWatch / smartphone black-box concept

White et al.'s WreckWatch work is directly aligned with the "phone black box" concept: smartphone accelerometers, GPS, acoustic/context information and network connectivity can be combined for accident detection, incident recording and emergency notification. The work emphasizes false-positive resistance and accident reconstruction.

**Design impact:** retain the black-box recording concept and context-aware false-positive handling, but modernize the architecture around current mobile background constraints and on-device ML.

Source:
- White et al., "WreckWatch: Automatic Traffic Accident Detection and Notification with Smartphones," Mobile Networks and Applications, 2011.
- DOI: https://doi.org/10.1007/s11036-011-0304-8

### 47.2 Smartphone accident detection research

Thompson et al. describe smartphone accident-detection architecture using sensor polling, large-acceleration detection and contextual information, with emphasis on reducing false positives and supporting responder situational awareness.

**Design impact:** use contextual evidence rather than a single acceleration threshold.

Source:
- https://eudl.eu/doi/10.1007/978-3-642-17758-3_3

### 47.3 Simple smartphone car-crash detection

Research on Android car-crash detection has combined accelerometer and location information to recognize crash patterns.

**Design impact:** retain accelerometer + location fusion as the minimum sensor combination.

Source:
- "Car crash detection on smartphones," DOI: https://doi.org/10.1145/2790044.2790049

### 47.4 Effective mobile-only collision detection

Paciorek et al. describe real-car crash-test measurements using smartphone data and professional crash-data acquisition equipment.

**Design impact:** real impact data is difficult and expensive; research-grade collision data and controlled validation are valuable references for model design.

Source:
- https://doi.org/10.1007/978-3-030-77980-1_24

### 47.5 Recent Android accident-notification work

A 2025 Android accident-detection study combines accelerometer, gyroscope and GPS with threshold-based severity evaluation and emergency notification.

**Design impact:** the three-sensor combination remains relevant; ML can be evaluated as a refinement rather than assuming it is automatically superior.

Source:
- https://link.springer.com/chapter/10.1007/978-3-032-22830-7_3

### 47.6 Sensor-fusion research for fall detection

A 2025 Sensors study compared Random Forest, SVM, XGBoost, logistic regression and majority voting for fall/activity classification using accelerometer and barometric-altimeter features. It reports that sensor fusion generally improved performance relative to individual-sensor models, while cross-data generalization was particularly relevant for RF/XGBoost.

**Design impact:** use sensor fusion and cross-subject evaluation; retain RF/XGBoost as strong classical baselines.

Source:
- https://pmc.ncbi.nlm.nih.gov/articles/PMC12694294/
- DOI: https://doi.org/10.3390/s25237220

### 47.7 Real-world fall data

A real-world fall study using 143 falls from the FARSEEING repository found that machine-learning methods based on multiphase fall features could outperform conventional feature sets, while performance remained materially dependent on realistic data.

**Design impact:** controlled/simulated data alone is insufficient.

Source:
- https://pmc.ncbi.nlm.nih.gov/articles/PMC7697900/

### 47.8 SisFall dataset

SisFall provides multiple fall types and activities of daily living, including data from younger and older adults, and shows that performance can decrease on older-user validation.

**Design impact:** dataset diversity and external validation matter; public fall datasets should be treated as supporting data, not as a direct crash dataset.

Source:
- https://www.mdpi.com/1424-8220/17/1/198

### 47.9 GPS safety alarms / geofencing evidence

A 2021 systematic review of GPS safety alarms for older adults found evidence of usability/piloting but insufficient evidence for strong claims of effectiveness according to the review's digital-health evidence framework.

**Design impact:** implement geofencing as a safety feature, but do not present project testing as clinical proof.

Source:
- https://www.jmir.org/2021/10/e27267/
- DOI: https://doi.org/10.2196/27267

### 47.10 Systematic review of mobile sensing for road safety

A systematic review of warning situations in road environments describes the broader use of mobile-device sensors and cloud technologies for road-safety applications.

**Design impact:** evaluate the system as an integrated sensing/communication pipeline rather than a standalone classifier.

Source:
- https://www.mdpi.com/2079-9292/9/3/416

### 47.11 On-device ML

On-device inference can reduce latency and improve privacy, while mobile compute and energy constraints remain important considerations.

**Design impact:** keep the critical classifier on-device and benchmark inference cost.

Source:
- https://arxiv.org/abs/1907.01989

A recent 2026 TinyML survey also emphasizes edge inference, resource limits and concept drift, supporting the need for model-size, energy and drift monitoring in future versions.

Source:
- https://arxiv.org/abs/2606.30843

---

## 48. Current Platform Constraints — 2026

### Android sensors

Android documentation states that continuous sensors such as accelerometers and gyroscopes do not provide events to ordinary background apps on Android 9+; continuous background sensing should therefore be implemented through a foreground service when required.

Source:
- https://developer.android.com/develop/sensors-and-location/sensors/sensors_overview

### Android foreground services

Android 14 requires foreground-service types and their corresponding permissions. Later Android versions add additional foreground-service restrictions.

Source:
- https://developer.android.com/develop/background-work/services/fgs/changes
- https://developer.android.com/about/versions/14/changes/fgs-types-required

### Background location

Android and Google Play require background location to be core to the user-facing functionality, clearly disclosed, and appropriately justified. The application must therefore treat background location as a core product permission rather than silently acquiring it.

Sources:
- https://developer.android.com/develop/sensors-and-location/location/background
- https://support.google.com/googleplay/android-developer/answer/9799150

### Firebase Cloud Messaging

FCM has foreground/background/terminated delivery semantics and requires notification permissions on Android 13+. Background handlers should complete quickly and should not be used for long-running work.

Sources:
- https://firebase.google.com/docs/cloud-messaging/flutter/receive-messages
- https://firebase.google.com/docs/cloud-messaging/android/get-started

### Flutter sensors

`sensors_plus` supports Android/iOS access to accelerometer and gyroscope data, but its existence does not remove native Android background execution constraints.

Source:
- https://pub.dev/documentation/sensors_plus/latest/

### Flutter location

`geolocator` documents Android background location requirements, including background location permission and foreground-service location requirements on newer Android versions.

Source:
- https://pub.dev/packages/geolocator

### Flutter geofencing

A native geofencing plugin can expose Android's hardware-assisted geofence mechanisms, which is preferable to continuously polling GPS only for boundary detection.

Source:
- https://pub.dev/packages/flutter_background_geofencing

### Mobile security

OWASP MASVS provides the baseline for secure mobile storage, authentication, networking, privacy, platform interaction and resilience.

Source:
- https://mas.owasp.org/MASVS/

### India data protection

India's DPDP Rules 2025 were notified in November 2025, making the privacy architecture materially relevant for an India-focused product.

Sources:
- https://www.meity.gov.in/documents/act-and-policies/digital-personal-data-protection-rules-2025-gDOxUjMtQWa
- https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf

---

## 49. Literature Interpretation — What We Can and Cannot Claim

### Supported by literature

- Smartphone sensors are technically capable of contributing to accident/fall detection.
- Combining multiple sensor/context signals can help distinguish events from ordinary movement.
- False positives are a central challenge.
- Real-world validation is harder than simulated-data validation.
- Geofencing is established as a technical mechanism for location-based alerts.
- On-device inference is practical for lightweight models.

### Not established by this project yet

- Guaranteed accident detection.
- Guaranteed emergency notification.
- Clinical effectiveness.
- Universal threshold values.
- Universal classifier performance across all phones, vehicles and users.
- Equivalence to vehicle event-data recorders.
- Automatic identification of injury severity.

These claims must not appear in the final application, report or demo unless supported by future evidence.

---

## 50. Research Gaps to Explicitly Test

### Gap 1 — Two-wheeler phone placement

Test how model performance changes when the phone is:

- handlebar mounted
- trouser pocket
- jacket pocket
- backpack
- vehicle storage area

### Gap 2 — Nuisance road events

Create an evaluation set for:

- speed breakers
- potholes
- hard braking
- sudden acceleration
- sharp turning
- phone drops

### Gap 3 — Cross-device generalization

Train using several phone models and test on devices not used for training.

### Gap 4 — False alarms per hour

Report operational false-alarm burden, not only test-set accuracy.

### Gap 5 — Battery/accuracy trade-off

Measure different sensor sampling profiles.

### Gap 6 — Network-independent operation

Quantify the percentage of detected incidents that remain safely stored and later synchronized after connectivity loss.

---

## 51. Demo Scenario

The final project demonstration should use a safe scripted scenario:

1. User signs in.
2. User adds trusted contact.
3. Contact completes pairing.
4. User enables monitoring.
5. App displays monitoring dashboard.
6. A pre-recorded sensor trace or safe test input triggers candidate detection.
7. ML classifies the event.
8. Countdown appears.
9. Demo case A: user taps "I'm OK".
10. Demo case B: user does not cancel.
11. Incident is stored.
12. Backend receives incident.
13. Trusted contact receives FCM notification.
14. Contact opens location.
15. Incident appears in history.
16. Geofence is demonstrated separately.
17. Network-loss case is demonstrated by disabling connectivity before simulated incident upload.
18. Connectivity is restored.
19. Incident is synchronized.

This is safer and more reproducible than attempting to demonstrate an actual collision.

---

## 52. Deliverables

### Software

- Android Flutter app
- native background sensing service
- backend API
- MongoDB schemas/indexes
- ML training pipeline
- on-device model
- FCM contact notification
- geofencing
- local incident storage
- test suite

### Documentation

- PRD
- architecture document
- API specification
- database schema
- ML experiment report
- privacy/data-flow document
- security checklist
- test plan
- user manual
- final project report
- demo script

### Research evidence

- literature matrix
- dataset matrix
- model comparison
- battery report
- latency report
- confusion matrix
- field-test report

---

## 53. Research Matrix

| Source | Domain | Main takeaway | PRD impact |
|---|---|---|---|
| WreckWatch (White et al.) | Smartphone accident detection | Smartphone sensors + network can support accident detection and reconstruction | Black-box architecture |
| Thompson et al. | Smartphone crash detection | Context reduces false positives | Multi-signal detection |
| Fazeen-related smartphone work | Crash detection | Accelerometer + location can detect crash patterns | Minimum sensor fusion |
| Paciorek et al. | Real crash tests | Real collision data can be collected with instrumentation | Ground-truth strategy |
| 2025 Android accident study | Mobile accident alerts | Accelerometer + gyroscope + GPS remain practical | Sensor fusion |
| Jurčić & Magjarević, 2025 | Fall/ADL classification | RF/XGB and sensor fusion are useful; cross-dataset robustness matters | Classical ML baseline |
| FARSEEING | Real-world falls | Real-world data is difficult and differs from simulations | Dataset design |
| SisFall | Fall/ADL dataset | Broad activity diversity but sensor placement differs | Supporting dataset only |
| Ehn et al., 2021 | GPS safety alarms | Technical feasibility ≠ clinical effectiveness | Avoid overclaiming |
| Road-safety mobile sensing review | Road safety | Mobile sensing is a wider research area | System architecture |
| On-device ML research | Edge inference | Privacy/latency benefits with compute/energy tradeoffs | Local model deployment |
| 2026 TinyML survey | Edge ML | Device constraints + concept drift matter | Future model lifecycle |

---

## 54. Product Metrics Dashboard

### Product

- Daily active monitoring hours
- Enabled monitoring sessions
- Trusted contact pairing success
- Test alert success
- Geofence configuration success

### ML

- precision
- recall
- F1
- false alarms/hour
- latency
- cross-device performance
- cross-position performance

### Reliability

- local save success
- backend ingestion success
- notification delivery success
- retry success
- duplicate incident rate

### Performance

- battery %/hour
- CPU
- memory
- storage
- inference time

### Privacy/security

- exposed sensitive-data findings
- insecure-storage findings
- auth failures
- security test pass rate

---

## 55. Decision Log Template

Every major implementation change should have an ADR:

```text
ADR-ID:
Date:
Decision:
Context:
Options:
Chosen option:
Why:
Trade-offs:
Security impact:
Privacy impact:
Battery impact:
Testing impact:
Rollback:
```

Examples:

- ADR-001 Background sensing service
- ADR-002 ML runtime
- ADR-003 FCM vs SMS fallback
- ADR-004 Authentication
- ADR-005 Remote black-box storage
- ADR-006 Geofencing implementation
- ADR-007 Retention periods

---

## 56. Immediate Next 10 Engineering Tasks

1. Create monorepo and CI skeleton.
2. Create Flutter Android project.
3. Build native Android foreground service.
4. Read accelerometer/gyroscope continuously on a physical device with screen off.
5. Add SQLite ring buffer.
6. Build sensor replay tool so recorded traces can be replayed without driving.
7. Create ML dataset schema and data-validation script.
8. Train Logistic Regression, Random Forest and XGBoost baselines.
9. Export the winning classical model to ONNX and compare Python/mobile predictions.
10. Implement the incident state machine before building the full notification UI.

---

## 57. Definition of a Successful MVP

The MVP is successful when a physical Android phone can:

```text
monitor in background
      ↓
capture inertial data
      ↓
maintain local rolling buffer
      ↓
detect candidate event
      ↓
run on-device classifier
      ↓
show confirmation
      ↓
cancel OR timeout
      ↓
persist incident
      ↓
survive network loss
      ↓
synchronize later
      ↓
notify registered companion contact
      ↓
display incident location
```

and when the evaluation demonstrates, with held-out users/sessions, the measured trade-off between:

- detection recall
- false alarms
- alert latency
- battery use
- device variability
- network reliability

---

## 58. Final Product Positioning

**Phone Black Box is a research-grade B.Tech prototype for smartphone-based incident detection and trusted-contact alerting.**

The product should be presented as:

> "A smartphone-based safety system that uses sensor fusion and machine learning to detect probable crashes/falls, gives the user an opportunity to cancel false alarms, records a short local black-box window, and alerts trusted contacts when no response is received."

It should **not** be presented as:

- a guaranteed accident detector
- a guaranteed emergency dispatch system
- a medical device
- a certified vehicle black box
- a replacement for emergency services

---

# Appendix A — Requirement IDs

## Functional

- FR-001 Account registration
- FR-002 Authentication
- FR-003 Trusted-contact management
- FR-004 Contact verification
- FR-005 FCM registration
- FR-006 Monitoring enable/disable
- FR-007 Background sensor monitoring
- FR-008 GPS/location acquisition
- FR-009 Rolling sensor buffer
- FR-010 Candidate event detection
- FR-011 ML inference
- FR-012 Confirmation window
- FR-013 User cancellation
- FR-014 Incident creation
- FR-015 Offline incident queue
- FR-016 Backend synchronization
- FR-017 Contact notification
- FR-018 Incident history
- FR-019 Geofence creation
- FR-020 Geofence monitoring
- FR-021 Geofence alerts
- FR-022 Monitoring diagnostics
- FR-023 Data deletion
- FR-024 Audit logging

## Non-Functional

- NFR-001 Low latency
- NFR-002 bounded battery consumption
- NFR-003 offline durability
- NFR-004 security
- NFR-005 privacy
- NFR-006 scalability
- NFR-007 observability
- NFR-008 maintainability
- NFR-009 reproducibility
- NFR-010 accessibility
- NFR-011 OS-version compatibility
- NFR-012 model reproducibility

---

# Appendix B — Glossary

**SMV:** Signal Magnitude Vector, `sqrt(ax² + ay² + az²)`.

**Jerk:** Rate of change of acceleration.

**Candidate event:** A sensor pattern that crosses the low-cost pre-screening rule and requires ML classification.

**Incident:** A candidate classified as requiring user confirmation.

**True positive:** A genuine target incident correctly classified.

**False positive:** Normal/nuisance motion incorrectly classified as a target incident.

**False negative:** A genuine incident not detected by the system.

**Black box:** Rolling local record of sensor/location information around a detected event.

**Trusted contact:** A person authorized to receive safety alerts.

**Geofence:** A virtual geographic boundary that generates an event when a device crosses it.

**FCM:** Firebase Cloud Messaging.

**ONNX:** Open Neural Network Exchange model format.

**BOLA/IDOR:** Broken object-level authorization / insecure direct object reference classes of authorization vulnerabilities.

---

# Appendix C — Reference Links

1. WreckWatch: https://doi.org/10.1007/s11036-011-0304-8
2. Smartphone accident detection (Mobilware): https://eudl.eu/doi/10.1007/978-3-642-17758-3_3
3. Car crash detection on smartphones: https://doi.org/10.1145/2790044.2790049
4. Effective Car Collision Detection with Mobile Phone Only: https://doi.org/10.1007/978-3-030-77980-1_24
5. 2025 Android accident detection paper: https://link.springer.com/chapter/10.1007/978-3-032-22830-7_3
6. 2025 sensor-fusion fall research: https://doi.org/10.3390/s25237220
7. Real-world fall detection: https://pmc.ncbi.nlm.nih.gov/articles/PMC7697900/
8. SisFall: https://doi.org/10.3390/s17010198
9. FARSEEING repository: https://doi.org/10.1186/s11556-016-0168-9
10. GPS safety alarms review: https://doi.org/10.2196/27267
11. Road-safety mobile sensing review: https://www.mdpi.com/2079-9292/9/3/416
12. On-device ML: https://arxiv.org/abs/1907.01989
13. TinyML survey (2026): https://arxiv.org/abs/2606.30843
14. Android sensors overview: https://developer.android.com/develop/sensors-and-location/sensors/sensors_overview
15. Android foreground services: https://developer.android.com/develop/background-work/services/fgs/changes
16. Android FGS types: https://developer.android.com/about/versions/14/changes/fgs-types-required
17. Android background location: https://developer.android.com/develop/sensors-and-location/location/background
18. Google Play background location policy: https://support.google.com/googleplay/android-developer/answer/9799150
19. Firebase Flutter messaging: https://firebase.google.com/docs/cloud-messaging/flutter/receive-messages
20. Firebase Android messaging: https://firebase.google.com/docs/cloud-messaging/android/get-started
21. Flutter sensors_plus: https://pub.dev/documentation/sensors_plus/latest/
22. Flutter geolocator: https://pub.dev/packages/geolocator
23. Flutter background geofencing: https://pub.dev/packages/flutter_background_geofencing
24. ONNX sklearn conversion: https://onnx.ai/sklearn-onnx/
25. TFLite Flutter: https://pub.dev/packages/tflite_flutter
26. OWASP MASVS: https://mas.owasp.org/MASVS/
27. MeitY DPDP Rules 2025: https://www.meity.gov.in/documents/act-and-policies/digital-personal-data-protection-rules-2025-gDOxUjMtQWa
28. Gazette of DPDP Rules 2025: https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf

---

## Document control

**Owner:** Phone Black Box project team  
**Audience:** Student developers, project guide, reviewers, ML developers, backend developers, testers  
**Next review:** After Phase 0 architecture spike and physical-device background-sensing test