# ADR-001: MVP Technology Stack

**Status:** Accepted  
**Date:** 13 July 2026  
**Decision owners:** Second project team

## Context

Second's defining interaction is native `Share → Second` capture. The MVP also needs authenticated cloud storage, structured relational data, private attachments, background processing, and strict per-user data isolation.

The initial stack should optimize for a reliable iOS share experience, fast MVP delivery, and reuse of the main application code for a future Android client.

## Decision

The Second MVP will use the following stack:

| Area | Technology |
| :--- | :--- |
| Initial platform | iOS |
| Application framework | Flutter |
| Application language | Dart |
| Application UI | Flutter widgets |
| iOS system integration | Native iOS Share Extension using Swift |
| Shared extension storage | App Group container |
| Local persistence | Lightweight local cache and durable capture queue |
| Cloud platform | Supabase |
| Primary database | Supabase PostgreSQL |
| Client SDK | `supabase_flutter` |
| Authentication | Supabase Auth with Sign in with Apple |
| Attachment storage | Private Supabase Storage buckets |
| Server-side processing | Supabase Edge Functions |
| Authorization | PostgreSQL Row-Level Security policies |
| Initial extraction | Deterministic calendar-event parser behind a replaceable extraction interface |

The main application, domain models, and API client will be written in Flutter. Platform-specific capture integrations will remain thin native adapters: Swift for the initial iOS Share Extension and Kotlin for a future Android Sharesheet integration.

## Rationale

- Flutter allows the main application, domain logic, and UI to be reused for a future Android client.
- Flutter officially supports adding iOS app-extension targets and sharing resources through an App Group.
- A thin native Swift Share Extension preserves direct access to iOS extension behavior and lifecycle controls without moving the full application to Swift.
- An App Group provides a shared container for the Flutter app and Share Extension, allowing captures to survive extension termination or temporary network failure.
- Second's Items and typed category records fit a relational PostgreSQL model.
- Supabase provides an official Flutter client, authentication, PostgreSQL, private object storage, and server functions in one MVP-oriented platform.
- Row-Level Security allows ownership rules to be enforced in the database rather than relying only on client filtering.
- A deterministic parser lets the first vertical slice be tested predictably while preserving the option to add AI-assisted extraction later.

## Consequences

### Benefits

- The core share experience uses native platform capabilities while most application code remains cross-platform.
- A future Android app can reuse the Flutter UI, domain logic, and Supabase integration.
- The team can build authentication, storage, and data access without operating a custom backend immediately.
- The database schema can closely follow the MVP technical specification.
- User isolation can be verified through database policies and automated tests.
- Future extraction implementations can change without altering the capture contract.

### Tradeoffs

- Android delivery is deferred while the iOS client is established.
- The project still requires native Swift code and Xcode configuration for the iOS Share Extension.
- Communication between the extension and Flutter application must use shared App Group resources or a controlled platform integration boundary.
- The app depends on Supabase services and operational constraints during the MVP.
- Share Extensions are short-lived, so uploads and processing cannot depend on the extension remaining active.
- App Group configuration and code signing require an Apple Developer team and consistent entitlements.
- Long-running or high-volume processing may eventually require a more specialized job system than Edge Functions alone.

## Alternatives Considered

### Native Swift and SwiftUI

Not selected for the main application because Flutter provides a practical path to Android code reuse. Native Swift remains in scope for the iOS Share Extension and any platform APIs that cannot be handled cleanly through Flutter plugins or platform channels.

### Firebase

Viable, but not selected because Second's canonical Item and typed subtype records map more naturally to PostgreSQL than to a document-first database model.

### Custom backend

Deferred because it would add infrastructure and authentication work before validating the primary capture-to-calendar flow.

### AI-first extraction

Deferred to keep the first implementation deterministic, inexpensive, and easy to test. AI may later be added through the extraction interface.

## Deferred Decisions

- Minimum supported iOS version
- Bundle identifier and App Group identifier
- Supabase project region
- Email or other fallback authentication methods
- Final Flutter state-management approach
- Final Flutter local-persistence package
- Boundary between the native extension and Dart code
- Production job-queue design
- AI provider and confidence thresholds
- External calendar synchronization

## References

- [Apple Share Extension guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Share.html)
- [Apple App Groups documentation](https://developer.apple.com/documentation/xcode/configuring-app-groups/)
- [Flutter iOS app-extension documentation](https://docs.flutter.dev/platform-integration/ios/app-extensions)
- [Flutter platform-channel documentation](https://docs.flutter.dev/platform-integration/platform-channels)
- [Supabase Flutter documentation](https://supabase.com/docs/reference/dart/introduction)
- [Supabase Row-Level Security documentation](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Supabase Sign in with Apple documentation](https://supabase.com/docs/guides/auth/social-login/auth-apple)
