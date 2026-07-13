# Second — MVP Technical Specification

**Status:** Working Draft  
**Version:** 0.1  
**Date:** 13 July 2026

> **Primary interaction:** `Share → Second`

## Contents

1. [Product Summary](#1-product-summary)
2. [Confirmed Product Decisions](#2-confirmed-product-decisions)
3. [Working Assumptions](#3-working-assumptions)
4. [MVP Goals](#4-mvp-goals)
5. [MVP Scope](#5-mvp-scope)
6. [Core Domain Model](#6-core-domain-model)
7. [Item Lifecycle](#7-item-lifecycle)
8. [Capture and Review Decision Flow](#8-capture-and-review-decision-flow)
9. [High-Level Architecture](#9-high-level-architecture)
10. [API Surface](#10-api-surface)
11. [Cloud Data and Synchronization](#11-cloud-data-and-synchronization)
12. [Security and Privacy Baseline](#12-security-and-privacy-baseline)
13. [Reliability and Performance Targets](#13-reliability-and-performance-targets)
14. [Observability](#14-observability)
15. [First Vertical Slice](#15-first-vertical-slice)
16. [Acceptance Criteria](#16-acceptance-criteria-for-the-first-vertical-slice)
17. [Test Strategy](#17-test-strategy)
18. [Open Decisions](#18-open-decisions)
19. [Definition of MVP Foundation Complete](#19-definition-of-mvp-foundation-complete)

---

## 1. Product Summary

Second is a personal operational counterpart that helps users preserve and act on information they explicitly choose to share.

Second captures the shared content, determines its likely intent, extracts useful structured information, and then either saves it automatically or asks the user to review it. It does not continuously monitor messages, email, photos, files, or other applications.

The MVP is built around a single domain concept: an **Item**. Calendar events, reminders, subscriptions, birthdays, notes, and memories are different representations of an Item rather than independent capture systems.

---

## 2. Confirmed Product Decisions

1. Content enters Second only through an explicit user action.
2. Whether sharing interrupts the user's current application depends on both:
   - whether the captured Item requires review; and
   - the user's capture preference.
3. If an Item does not require review, it is saved immediately.
4. If an Item requires review and the user selected **Review Later**, it is placed in the Inbox without opening the main Second app.
5. If an Item requires review and the user selected **Review Immediately**, Second opens the review experience.
6. The MVP has its own in-app calendar. External Apple or Google calendar synchronization is deferred.
7. The MVP uses cloud authentication, storage, and synchronization.
8. Detailed AI-provider policy, advanced extraction edge cases, and complete requirements traceability are deferred until after the foundation is working.
9. Existing HTML mockups are illustrative only and are not requirements or design references.

---

## 3. Working Assumptions

These assumptions allow implementation to begin but may be changed without altering the core product model.

- The main mobile application is built with Flutter and Dart.
- Initial delivery remains iOS-first because native sharing is the primary interaction; the iOS Share Extension is a thin native Swift target connected to the Flutter app through shared App Group resources.
- The Flutter application, API, and domain model remain Android-ready so an Android Sharesheet adapter can be added later.
- The cloud provider has not been selected. The design must not depend on provider-specific behavior at the domain layer.
- Processing may use deterministic parsers, platform capabilities, or an AI service behind one common interface.
- The app keeps a small local cache for fast loading and offline capture, while the cloud is the synchronized system of record.

---

## 4. MVP Goals

The MVP must demonstrate that a user can:

1. Share supported content to Second from another application.
2. Continue their existing flow when review is deferred.
3. Review uncertain or incomplete Items when needed.
4. Save an obvious Item without unnecessary interaction.
5. Find captured Items in the Inbox, calendar, or general Item list.
6. Edit, archive, and permanently delete their own Items.
7. Receive a clear processing result or failure state.

---

## 5. MVP Scope

### 5.1 Included

- User registration, sign-in, sign-out, and authenticated sessions
- User capture preference: Review Immediately or Review Later
- Native share extension
- Capture of plain text, selected text, URLs, images, screenshots, PDFs, and files supported by the operating system share interface
- Preservation of original shared content and basic source metadata
- Asynchronous classification and structured information extraction
- Review decision engine
- Inbox for Items awaiting review
- Review and edit experience
- Item categories:
  - Calendar event
  - Reminder
  - Subscription
  - Birthday
  - Memory
  - Note
  - Unknown
- Separate in-app calendar backed by Second's event records
- Basic reminder and subscription records
- Item list and Item details
- Basic search across titles, descriptions, extracted text, and structured fields
- Archive and permanent deletion
- Cloud database, object storage, and synchronization
- Processing retries and visible failure states

### 5.2 Deferred

- Apple Calendar and Google Calendar synchronization
- Android, web, desktop, browser extensions, and smart-watch clients
- Shared family, partner, or team spaces
- Bills, travel, warranty, and shopping-specific modules
- Voice capture
- Advanced daily, weekly, or monthly briefings
- Smart suggestions and autonomous actions
- Full notification center and notification history
- Detailed AI-provider governance and advanced privacy controls
- Data export and local-only mode
- Connected email or messaging accounts
- Background monitoring of any external application
- Production-scale multi-region architecture

---

## 6. Core Domain Model

### 6.1 Item

An Item is the canonical record for everything captured by Second.

Required fields:

| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | UUID | Stable Item identifier |
| `user_id` | UUID | Owner of the Item |
| `category` | enum | Current user-approved or system-suggested category |
| `status` | enum | Current lifecycle state |
| `title` | string | Generated or user-edited title |
| `description` | text, nullable | Generated or user-edited description |
| `source_type` | enum | Text, URL, image, screenshot, PDF, or file |
| `source_app` | string, nullable | Originating application when provided by the OS |
| `original_text` | text, nullable | Original or extracted textual representation |
| `source_url` | string, nullable | Original shared URL |
| `review_required` | boolean | Result of the review decision engine |
| `review_reasons` | JSON array | Reasons the Item needs review |
| `confidence` | decimal, nullable | Normalized overall processing confidence |
| `metadata` | JSON object | Non-canonical source and extraction metadata |
| `captured_at` | timestamp | When the user shared the content |
| `created_at` | timestamp | When the cloud record was created |
| `updated_at` | timestamp | Last modification time |
| `reviewed_at` | timestamp, nullable | When the user confirmed the Item |
| `archived_at` | timestamp, nullable | When the Item was archived |
| `deleted_at` | timestamp, nullable | Soft-deletion timestamp before permanent deletion |
| `version` | integer | Optimistic concurrency version |

`metadata` may preserve flexible extraction output, but fields required by product behavior must be stored in typed subtype records rather than only in JSON.

### 6.2 Item Categories

```text
calendar_event
reminder
subscription
birthday
memory
note
unknown
```

The user can always change the category. Reclassification must update the corresponding typed record within the same database transaction.

### 6.3 Calendar Event

| Field | Type |
| :--- | :--- |
| `item_id` | UUID, primary and foreign key |
| `starts_at` | timestamp |
| `ends_at` | timestamp, nullable |
| `all_day` | boolean |
| `timezone` | string |
| `location` | string, nullable |
| `participants` | JSON array |
| `recurrence_rule` | string, nullable |

The in-app calendar reads only Second calendar-event records for the MVP. It does not write to an external calendar.

### 6.4 Reminder

| Field | Type |
| :--- | :--- |
| `item_id` | UUID, primary and foreign key |
| `due_at` | timestamp |
| `timezone` | string |
| `recurrence_rule` | string, nullable |
| `completed_at` | timestamp, nullable |
| `snoozed_until` | timestamp, nullable |

### 6.5 Subscription

| Field | Type |
| :--- | :--- |
| `item_id` | UUID, primary and foreign key |
| `service_name` | string |
| `amount_minor` | integer, nullable |
| `currency` | string, nullable |
| `billing_interval` | enum |
| `next_renewal_at` | timestamp, nullable |
| `trial_ends_at` | timestamp, nullable |
| `cancelled_at` | timestamp, nullable |

Money is stored in minor currency units rather than floating-point values.

### 6.6 Birthday

| Field | Type |
| :--- | :--- |
| `item_id` | UUID, primary and foreign key |
| `person_name` | string |
| `birth_date` | date |
| `year_known` | boolean |

### 6.7 Attachment

| Field | Type |
| :--- | :--- |
| `id` | UUID |
| `item_id` | UUID |
| `storage_key` | string |
| `file_name` | string |
| `media_type` | string |
| `byte_size` | integer |
| `checksum` | string |
| `created_at` | timestamp |

Binary content is stored in private object storage. Database records contain storage identifiers, never public permanent URLs.

### 6.8 User Preference

| Field | Type | Description |
| :--- | :--- | :--- |
| `user_id` | UUID | User identifier |
| `capture_mode` | enum | `review_immediately` or `review_later` |
| `timezone` | string | Default interpretation and display timezone |
| `locale` | string | Date, number, and language context |
| `notifications_enabled` | boolean | Master notification setting |

Automatic saving is an outcome for Items that do not require review; it is not a separate capture preference in this MVP.

### 6.9 Capture Job

| Field | Type |
| :--- | :--- |
| `id` | UUID |
| `item_id` | UUID |
| `status` | enum |
| `attempt_count` | integer |
| `processor_version` | string |
| `error_code` | string, nullable |
| `error_message` | string, nullable |
| `started_at` | timestamp, nullable |
| `completed_at` | timestamp, nullable |
| `next_attempt_at` | timestamp, nullable |

Capture jobs make retries observable and prevent processing state from being hidden inside Item metadata.

---

## 7. Item Lifecycle

### 7.1 States

```text
local_pending_upload
uploaded
processing
awaiting_review
saved
completed
archived
processing_failed
deleted
```

`local_pending_upload` is a client-only state. All other states are persisted in the cloud.

### 7.2 State Transitions

```text
local_pending_upload
        │
        ▼
     uploaded
        │
        ▼
    processing ───────────────► processing_failed
        │                              │
        │                              └── retry ──► processing
        │
        ├── review required ──► awaiting_review ──► saved
        │
        └── no review required ──────────────────► saved

saved ──► completed
  │
  ├──► archived
  └──► deleted

completed ──► archived
archived ──► saved
archived ──► deleted
```

Deleting an Item must remove or schedule removal of its subtype records, search index entries, attachments, and pending notifications.

---

## 8. Capture and Review Decision Flow

### 8.1 Processing Sequence

1. The user selects content and chooses Second from the operating-system share sheet.
2. The share extension creates a local capture record and idempotency key.
3. The extension uploads the original content and metadata.
4. The capture API acknowledges receipt quickly and queues processing.
5. The processor:
   - normalizes input;
   - extracts text when applicable;
   - proposes a category;
   - extracts category-specific fields;
   - calculates confidence;
   - validates required fields; and
   - determines whether review is required.
6. The routing service reads the user's capture preference.
7. The Item follows exactly one route:

| Review required? | User preference | Result |
| :--- | :--- | :--- |
| No | Either | Save automatically |
| Yes | Review Later | Add to Inbox and return control to the current app |
| Yes | Review Immediately | Open or foreground Second's review experience |

8. The client displays a lightweight confirmation or failure result appropriate to the chosen route.

### 8.2 Initial Review Rules

An Item requires review when one or more of the following is true:

- Category is `unknown`.
- Overall confidence is below a configurable threshold.
- A required typed field is missing.
- Two or more plausible interpretations would produce materially different outcomes.
- A date or time is ambiguous relative to the user's locale or timezone.
- Processing detects conflicting values.
- The requested operation could create a duplicate active Item.
- Processing fails partially and the original content can still be preserved.

Thresholds and category-specific rules must be configuration, not hard-coded UI logic.

### 8.3 Review Result

During review, the user can:

- confirm the proposed category and fields;
- edit extracted fields;
- select a different category;
- save as a general Memory or Note;
- delete the captured Item.

User confirmation changes the Item to `saved`, records `reviewed_at`, and writes the correct subtype record atomically.

---

## 9. High-Level Architecture

```text
┌─────────────────────────────── Mobile Client ───────────────────────────────┐
│ Flutter App   Native Share Adapter   Local Cache   Upload/Sync Coordinator  │
└──────────────────────────────────────┬───────────────────────────────────────┘
                                       │ TLS
                                       ▼
┌──────────────────────────────── Cloud Platform ─────────────────────────────┐
│ Authentication                                                            │
│ API Layer ── Capture Service ── Job Queue ── Processing Workers             │
│      │              │                              │                        │
│      ├──────── Item Service / Review Routing ──────┤                        │
│      │              │                              │                        │
│      ▼              ▼                              ▼                        │
│ Relational DB   Private Object Storage       Extraction Adapter             │
│      │                                             │                        │
│      ├──────── Search ─────── Notification Scheduler                        │
│      │                                                                      │
│      └──────── Sync / Change Feed                                           │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 9.1 Component Responsibilities

#### Share Extension

- Accept supported operating-system share payloads.
- Preserve original content.
- Create an idempotency key.
- Upload immediately when possible.
- Queue locally when offline.
- Avoid performing classification or other long work inside the extension.
- On iOS, use a thin native Swift target and an App Group shared with the Flutter application.
- Keep the capture contract platform-neutral for a future Android Sharesheet adapter.

#### Capture Service

- Authenticate the request.
- Validate size and content type.
- Create Item, Attachment, and Capture Job records.
- Deduplicate repeated requests with the idempotency key.
- Queue processing.

#### Processing Worker

- Normalize input and extract text.
- Call the extraction adapter.
- Validate and normalize extracted fields.
- Store processing confidence and reasons.
- Produce a review decision.
- Retry transient failures safely.

#### Item Service

- Enforce ownership and lifecycle transitions.
- Apply edits and category changes transactionally.
- Maintain typed subtype records.
- Coordinate archive and deletion.

#### Sync Coordinator

- Upload queued local captures.
- Pull server changes.
- Maintain an offline-readable cache.
- Resolve conflicts using Item versions and explicit server responses.

---

## 10. API Surface

The first API should be versioned under `/v1`. All mutating endpoints require authentication and accept an idempotency key where retries are possible.

### 10.1 Capture

```http
POST /v1/captures
```

Creates a capture and returns an acknowledgement.

**Request concept**

```json
{
  "sourceType": "text",
  "sourceApp": "com.example.messaging",
  "originalText": "Coffee on Friday at 6 PM at Third Wave",
  "sourceUrl": null,
  "capturedAt": "2026-07-13T10:00:00Z",
  "clientTimezone": "Asia/Kolkata",
  "clientLocale": "en-IN"
}
```

**Response concept**

```json
{
  "itemId": "uuid",
  "jobId": "uuid",
  "status": "uploaded"
}
```

Large binary attachments use a short-lived upload flow rather than being embedded in JSON.

### 10.2 Capture Status

```http
GET /v1/captures/{jobId}
```

Returns processing status, route, and failure information.

### 10.3 Inbox

```http
GET /v1/items?status=awaiting_review&cursor={cursor}
```

Returns only the authenticated user's Items awaiting review.

### 10.4 Review

```http
POST /v1/items/{itemId}/review
```

Confirms or changes the category and structured fields. The request includes the Item version to prevent silent overwrites.

### 10.5 Items

```http
GET    /v1/items/{itemId}
PATCH  /v1/items/{itemId}
POST   /v1/items/{itemId}/archive
POST   /v1/items/{itemId}/restore
DELETE /v1/items/{itemId}
```

### 10.6 Calendar

```http
GET /v1/calendar/events?from={timestamp}&to={timestamp}
```

Returns Second's internal calendar events in the requested range.

### 10.7 Search

```http
GET /v1/search?q={query}&category={category}&cursor={cursor}
```

Search is limited to the authenticated user's non-deleted Items.

### 10.8 Preferences

```http
GET   /v1/preferences
PATCH /v1/preferences
```

---

## 11. Cloud Data and Synchronization

- The relational cloud database is the synchronized source of record.
- Each Item belongs to exactly one user in the MVP.
- Every read and write must enforce user ownership server-side.
- The client stores only data needed for offline capture and fast recent views.
- Client mutations include the last known Item version.
- The server rejects stale conflicting updates with a conflict response rather than silently overwriting them.
- Offline captures remain in a durable local queue until acknowledged by the server.
- Re-uploading the same queued capture must not create duplicates.
- Attachment upload completion must be verified before an Item can leave `uploaded` state.

---

## 12. Security and Privacy Baseline

Detailed policy is deferred, but the MVP must satisfy this minimum baseline:

- Explicit user initiation is required for every capture.
- All network traffic uses TLS.
- Cloud data and object storage are encrypted at rest.
- Authentication is required for all user data.
- Authorization is checked on every server request; client-side filtering is not authorization.
- Object storage is private and accessed through short-lived signed requests.
- Logs must not contain original shared text, attachment contents, authentication tokens, or extracted sensitive fields.
- Secrets are stored outside the source repository.
- Permanent deletion removes user-visible data and schedules attachment deletion.
- Processing services receive only the data required for the active capture job.

---

## 13. Reliability and Performance Targets

| Area | MVP target |
| :--- | :--- |
| Capture acknowledgement | Within 1 second under normal network conditions |
| Text processing result | Within 3 seconds for the common path |
| Dashboard or primary list load | Within 2 seconds from network; faster from cache |
| Search response | Within 1 second for typical MVP data volumes |
| Retry behavior | Automatic retry for transient processing failures |
| Data durability | No acknowledged capture is silently discarded |
| Idempotency | Retried capture requests create one logical Item |

If processing cannot finish within the share-extension lifetime, the extension acknowledges capture and processing continues asynchronously.

---

## 14. Observability

The MVP records structured operational events without recording private content:

- Capture accepted
- Upload completed or failed
- Processing started and completed
- Processing duration
- Proposed category
- Review required decision and reason codes
- Retry count
- User review completed
- Sync conflict
- Notification scheduled or failed

Metrics must use Item and job identifiers, not captured content.

---

## 15. First Vertical Slice

The first implementation proves one complete path using shared plain text and calendar events.

### 15.1 Supported Scenario

The user shares:

> Coffee with Sarah on Friday at 6 PM at Third Wave.

The system must:

1. Receive the text through the share extension.
2. Store the original text in the cloud.
3. Detect a calendar event.
4. Extract title, date, time, timezone, and location.
5. Decide whether review is required.
6. Route according to the user's preference.
7. Save the confirmed or automatically accepted event.
8. Display the event in Second's in-app calendar.
9. Allow the user to edit, archive, restore, or delete the Item.

### 15.2 Initial Build Order

1. Define database migrations for users, Items, calendar events, capture jobs, and preferences.
2. Implement authentication and user-level data isolation.
3. Implement capture and status APIs with idempotency.
4. Implement a deterministic calendar-event parser behind the extraction interface.
5. Implement review-decision and routing logic.
6. Build the Flutter application, native iOS Share Extension, App Group bridge, and durable local upload queue.
7. Build Inbox and Review Item screens.
8. Build the in-app calendar list or agenda view.
9. Add failure, retry, editing, archive, restore, and deletion flows.
10. Add automated tests and privacy-safe operational metrics.

The first parser may be deliberately narrow. The extraction interface must allow it to be replaced or supplemented later without changing the capture API or Item lifecycle.

---

## 16. Acceptance Criteria for the First Vertical Slice

### Capture

- A signed-in user can share plain text to Second.
- Second preserves the exact original text.
- Repeating the same upload with the same idempotency key does not create a second Item.
- An offline capture remains queued and uploads after connectivity returns.

### Processing

- A supported event phrase produces a structured calendar event.
- Missing or ambiguous required fields cause `review_required = true` with reason codes.
- A transient processing failure is retried.
- A permanent failure is visible to the user and never silently discarded.

### Routing

- An Item that does not require review is saved regardless of capture preference.
- A review-required Item enters the Inbox when the preference is Review Later.
- A review-required Item opens the review experience when the preference is Review Immediately, subject to operating-system capabilities.
- Changing the capture preference affects future routing decisions.

### Calendar

- A saved event appears on the correct date and time in Second's calendar.
- Editing the Item updates the calendar representation.
- Archiving or deleting the Item removes it from active calendar results.
- Restoring an archived Item returns it to the calendar.

### Ownership and Privacy

- One user cannot access another user's Item by guessing identifiers.
- Private attachment URLs are not permanently public.
- Application logs do not contain captured text.

---

## 17. Test Strategy

### Unit tests

- State-transition rules
- Review-decision rules
- Category and field validation
- Date, time, timezone, and currency normalization
- Idempotency behavior
- Authorization policy helpers

### Integration tests

- Capture API through queued processing
- Database transaction for review and subtype creation
- Retry behavior
- Archive, restore, and deletion cascades
- User isolation
- Signed attachment access

### Client tests

- Share-extension input handling
- Offline capture queue
- Review routing by preference
- Inbox refresh after processing
- Calendar rendering after edits

### End-to-end tests

- Confident Item → automatic save → calendar
- Review required + Review Later → Inbox → review → calendar
- Review required + Review Immediately → review → calendar
- Offline share → reconnect → process → correct route
- Processing failure → retry or visible permanent failure

---

## 18. Open Decisions

These decisions are intentionally not blockers for defining the domain model, but they must be resolved before their affected components are implemented.

1. Android delivery timing after the iOS-first vertical slice.
2. Cloud provider and deployment region.
3. Authentication methods for the first release.
4. Extraction approach and any external AI provider.
5. Confidence thresholds by category.
6. Exact behavior permitted by iOS for opening the containing Flutter app from the Share Extension.
7. Attachment size and supported-format limits.
8. Search implementation.
9. Notification delivery and scheduling architecture.
10. Data retention period after deletion.
11. Whether `completed` applies to all Item categories or only actionable categories.

---

## 19. Definition of MVP Foundation Complete

The technical foundation is complete when:

- The first vertical slice passes its acceptance criteria.
- The Item lifecycle is enforced by server-side code and tests.
- Cloud ownership boundaries are verified by automated tests.
- Capture retries do not create duplicates.
- Review routing follows the confirmed decision table.
- Second's internal calendar reflects saved event Items consistently.
- Processing failures are recoverable or visible.
- No architecture decision in the first slice prevents later Android support or additional Item categories.
