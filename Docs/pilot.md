# Second Pilot

## Product thesis

Second helps early-career professionals avoid forgotten commitments by removing the need to switch apps and manually rewrite actionable information.

Users share information directly from its originating app. Second first preserves it, then detects likely commitments and prepares a reminder or undated to-do.

## Pilot

- Closed, invitation-only, four-week pilot
- 15–20 qualified early-career professionals
- Separate iPhone and Android implementations
- Each tester uses one primary device
- No browser extension
- No accounts, backup, export, sync, or recovery
- English-only commitment detection
- All captured content remains on-device
- Optional anonymous analytics with explicit consent
- No raw captured content collected for research

## Capture contract

- Accept anything the operating system successfully provides, within published limits
- Explicitly support text, URLs, images, screenshots, and PDFs
- Persist locally before showing follow-up questions
- Never display **Saved** until persistence succeeds
- Preserve the supplied content and available source metadata
- URLs retain their URL, title, preview, source app, and capture time
- Minimal in-app plain-text creation is supported

## Commitment flow

### High-confidence commitments

1. Display **Saved**.
2. Present editable title, date, and time.
3. Offer **Confirm reminder**, **Review later**, or **Not actionable**.
4. Treat closing without a decision as **Review later**.

### Ambiguous candidates

- Do not interrupt.
- Add the candidate to **Needs review**.
- Offer **Create reminder**, **Add undated to-do**, and **Not actionable** as review actions.

The immediate-prompt precision target is at least 80%.

## Reminders and to-dos

- A date may have an optional time.
- Date-only reminders fire at a configurable default, initially 9 a.m.
- Undated commitments become minimal to-dos.
- To-dos contain a title, source, created date, optional due date, and completion state.
- Reminder notifications support **Complete**, **Snooze one hour**, **Tomorrow**, and **Choose date/time**.
- Recurring reminders are not supported in the pilot.
- Completing an action does not delete its captured source item.

## Library and review

- All captures appear in one chronological, searchable library.
- Screenshots and PDFs can receive an optional description after saving.
- If a detected commitment exists, its confirmed title becomes the description.
- Without a description, search uses the filename, source app, and capture date.
- OCR is not required for the pilot.
- Daily review contains **Needs review**, **Undated commitments**, and **Due soon**.
- Users choose one daily-review time during onboarding.
- Second sends at most one review notification and does not nag.
- Deleted items remain in encrypted trash for 30 days unless permanently deleted.

## Privacy and expansion

The pilot is deliberately local-only and warns that uninstalling the app or losing the device destroys its data.

After a successful pilot, expansion proceeds in this order:

1. End-to-end encrypted accounts, backup, and synchronization
2. Browser capture
3. Later consideration of cloud AI, more languages, and integrations

The eventual business model is a consumer subscription—not advertising or employer funding.

## Validation gates

Continue only if:

- At least 50% of the original cohort captures voluntarily during week four.
- At least 40% of the original cohort identifies a specific commitment Second plausibly prevented them from forgetting.

Baseline and weekly interviews must reference concrete captured items and actual outcomes.
