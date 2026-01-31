# Nexus (Guardian AI)

Nexus is an Android-first Flutter application backed by Firebase Realtime Database. It is designed to help users build **daily discipline**, track **health/steps**, and follow **AI-assisted routines** via a structured “Daily Timeline” experience.

This repository is prepared for professional review (internal/company demo) and for GitHub upload (private recommended). The notification system is intentionally **local-only** to avoid Cloud Functions requirements.

## Table of Contents

- Overview
- Key Capabilities
- Architecture
- Notifications & Background Execution
- Firebase Data Model
- Local Development Setup
- Configuration (Secrets)
- Android Permissions & OEM Settings
- Build & Release
- Troubleshooting
- Security & Privacy
- License

## Overview

Nexus combines:

- A **Dashboard** for current metrics and “next task” countdown
- A **Daily Timeline** generated from a plain-text schedule stored in RTDB
- **Local notifications/alarms** to trigger task reminders reliably on-device
- Optional integrations (watch/BLE, food logging, AI-powered analysis)

## Modules (Screens / Pages)

Below is a practical module map of the app. Each item corresponds to a concrete screen file.

### Core

- **Dashboard** — `lib/dashboard_screen.dart`
  - Main landing screen, realtime overview, bootstraps schedule resync and background behaviors.
- **Daily Timeline** — `lib/daily_timeline_screen.dart`
  - Shows today’s tasks, completion state, and timeline-driven UX.

### AI

- **AI Chat** — `lib/ai_chat_screen.dart`
  - AI conversation flow, schedule suggestions, and chat history.
- **AI Service** — `lib/ai_service.dart`
  - AI request/response layer used by multiple screens.

### Auth / Onboarding

- **Sign In** — `lib/signin_screen.dart`
- **Sign Up** — `lib/signup_screen.dart`
- **Interview / Intake** — `lib/interview_screen.dart`
  - Initial user intake questions and profile setup.

### Reports & Review

- **Night Review** — `lib/screens/night_review_screen.dart`
  - End-of-day review for incomplete tasks and reasons.
- **Daily Report** — `lib/screens/daily_report_screen.dart`
  - Performance report view (daily summary + history).

### Notifications

- **Notification History** — `lib/screens/notification_history_screen.dart`
  - In-app log of recent notifications (tap to open related screens where supported).

### Settings & System

- **Settings** — `lib/screens/settings_screen.dart`
- **Background Service Control** — `lib/screens/background_service_screen.dart`
  - Start/stop/status for Android foreground service.
- **Profile** — `lib/screens/profile_screen.dart`
- **Profile (HUDglass)** — `lib/screens/profile_screen_hudglass.dart`
  - Alternate profile UI presentation.

### Watch & Sensors

- **Watch Connection** — `lib/watch_connection_screen.dart`
  - BLE/device connection workflows.
- **Movement Map** — `lib/screens/movement_map_screen.dart`
  - Location/movement visualization.
- **Sleep Tracker** — `lib/screens/sleep_tracker_screen.dart`
  - Sleep tracking and sleep-related signals.

### Food

- **Food Log** — `lib/screens/food_log_screen.dart`
  - Food entry and optional AI analysis.
- **Food History** — `lib/screens/food_history_screen.dart`
  - Past food records.

### Notes

- **Smart Notes** — `lib/screens/smart_notes_screen.dart`
  - Notes + reminders (uses alarms/TTs where enabled).

### Guardian / System UI

- **Omni Guardian** — `lib/screens/omni_guardian_screen.dart`
  - Guardian-style UI and system scanning visuals.

## Key Capabilities

- **Schedule → Tasks pipeline**: parses `best_ai_schedule/{userId}` into structured tasks.
- **Task reminders (local-only)**: schedules reminders using `android_alarm_manager_plus` and `flutter_local_notifications`.
- **Background monitoring**: Android foreground service can sync steps and trigger a low-activity warning.
- **Night Review & Daily Report**: end-of-day workflows that can be opened from local notifications and in-app history.
- **Watch connectivity**: BLE integration (device dependent).

## Architecture

High-level flow:

```
Realtime DB (best_ai_schedule)
        │
        ▼
Timeline parsing (Vault) ─────► active_timeline/{userId}
        │
        ├─► LocalTaskAlarmScheduler (Android alarms)
        │        │
        │        └─► flutter_local_notifications (task reminder)
        │
        └─► Background Service (optional)
                 └─► watch_data/{userId}/current (steps sync)
```

Core modules:

- UI: `lib/` (screens/widgets)
- Services: `lib/services/` (timeline vault, alarms, notifications, background)
- Platform: `android/` (Android configuration)

## Modules (Services)

Services are concentrated in `lib/services/`.

### Core runtime

- `app_navigator.dart` — app-wide navigation helper.
- `session_service.dart` — session/user state helpers.

### Timeline + schedule

- `timeline_models.dart` — task models and day-key helpers.
- `timeline_vault_service.dart` — reads `best_ai_schedule/{userId}` and maintains `active_timeline/{userId}`.
- `discipline_analyzer.dart` — scoring/discipline calculations.

### Notifications + alarms

- `local_task_alarm_scheduler.dart` — schedules exact Android alarms for task start.
- `timeline_notification_scheduler.dart` — other local notifications (Night Review, Daily Report, tactical alerts).
- `notification_tap_router.dart` — routes notification payloads into screens.
- `app_notification_log.dart` — persistent notification history log.

### Background + sensors

- `background_service.dart` — Android foreground service and periodic checks.
- `phone_activity_service.dart` — phone-side activity/steps helpers.
- `movement_service.dart` — location/movement helpers.
- `sleep_service.dart` — sleep-related signals.
- `weather_service.dart` — weather-related helpers.

### Watch / BLE integration

- `ble_service.dart` — BLE device communication.
- `universal_watch_service.dart` — watch data sync abstraction.

### Food

- `food_service.dart` — food logging/AI integration helpers.

### Compatibility stubs

- `fcm_service.dart` — no-op stub (legacy import compatibility; FCM is not required).
- `firebase_storage_stub.dart`, `image_picker_stub.dart`, `notification_listener_stub.dart`, `flutter_notification_listener_shim.dart` — platform/plugin compatibility shims.

## Screenshots (Per Page)

Add your screenshots under `docs/screenshots/` and keep filenames stable.

Suggested naming (PNG):

- `docs/screenshots/01_dashboard.png`
- `docs/screenshots/02_daily_timeline.png`
- `docs/screenshots/03_ai_chat.png`
- `docs/screenshots/04_sign_in.png`
- `docs/screenshots/05_sign_up.png`
- `docs/screenshots/06_interview.png`
- `docs/screenshots/07_settings.png`
- `docs/screenshots/08_notification_history.png`
- `docs/screenshots/09_night_review.png`
- `docs/screenshots/10_daily_report.png`
- `docs/screenshots/11_food_log.png`
- `docs/screenshots/12_food_history.png`
- `docs/screenshots/13_watch_connection.png`
- `docs/screenshots/14_sleep_tracker.png`
- `docs/screenshots/15_movement_map.png`
- `docs/screenshots/16_smart_notes.png`
- `docs/screenshots/17_omni_guardian.png`

You can embed them like:

```md
![Dashboard](docs/screenshots/01_dashboard.png)
```

## Notifications & Background Execution

This project is intentionally designed to work **without Cloud Functions / Blaze plan**.

### Task start reminders

- When the Dashboard opens, the app fetches the latest schedule and re-syncs alarms.
- Each upcoming task is scheduled as an **exact one-shot Android alarm** via `android_alarm_manager_plus`.
- When an alarm fires, a local notification is shown via `flutter_local_notifications`.

### Low-activity alerts

- The background service periodically checks `watch_data/{userId}/current/steps`.
- If steps increased by **< 500** since the last check, it triggers a quiet local alert:
  **Guardian Alert: Low activity detected!**

### Android 12+ (Exact alarms)

Android 12 may restrict exact alarms. If task notifications do not fire reliably:

- Settings → Apps → Special access → Alarms & reminders → allow Nexus

The app also performs a best-effort prompt to open exact-alarm settings when scheduling.

## Firebase Data Model

Common RTDB paths used by the app:

- `Users/{userId}/...` — profile/settings
- `best_ai_schedule/{userId}` — plain-text daily schedule
- `active_timeline/{userId}/...` — today’s pending/completed tasks
- `watch_data/{userId}/current` — synced steps/kcal/last_updated
- `watch_data/{userId}/history/{yyyy-MM-dd}` — daily snapshots

## Local Development Setup (Windows)

### Prerequisites

- Flutter SDK (stable)
- Android Studio + Android SDK
- A physical Android device (recommended for background + alarm testing)

### Install dependencies

```powershell
flutter pub get
```

### Firebase setup (required)

1) Create a Firebase project.
2) Add an Android app in Firebase.
3) Download `google-services.json` and place it at:
   - `android/app/google-services.json`
4) Enable Realtime Database.

### Run

```powershell
flutter run
```

## Configuration (Secrets)

The repository should not contain runtime secrets.

Optional Food AI keys are provided via `--dart-define`:

- `IMGBB_API_KEY` (image upload)
- `OPENROUTER_API_KEY` (vision request)

Run with keys:

```powershell
flutter run --dart-define=IMGBB_API_KEY="<your_imgbb_key>" --dart-define=OPENROUTER_API_KEY="<your_openrouter_key>"
```

Note: changing `--dart-define` values requires a full restart (not hot reload).

## Android Permissions & OEM Settings

- Notifications: enable app notifications.
- Android 12+: allow “Alarms & reminders” for reliable exact alarms.
- Battery optimization: disable aggressive optimization if you want step sync + monitoring.
- Activity recognition: required for step counting.
- Location/Bluetooth: required for watch connectivity (device-dependent).

## Build & Release

### Release APK

```powershell
flutter build apk --release
```

Output:

- `build/app/outputs/flutter-apk/app-release.apk`

### Release checklist (recommended)

- Verify alarms on Android 12+ (Alarms & reminders granted)
- Verify background service behavior on target OEM device
- Verify DB rules for production
- Remove debug/demo credentials (if any)

## Troubleshooting

- No task notifications
  - Confirm Android 12+ “Alarms & reminders” is allowed.
  - Confirm app notifications are enabled.
  - Open Dashboard once to trigger an alarm resync.

- Background service running but no alerts
  - Disable aggressive battery optimization for the app.
  - Confirm steps are updating at `watch_data/{userId}/current/steps`.

- Skipped frames in logs
  - Common on lower-end devices in debug mode.
  - Prefer `--release` builds for performance testing.

## Security & Privacy

- Never commit API keys or private credentials.
- For public repositories, remove `android/app/google-services.json` and provide an example file instead.
- Prefer `--dart-define` or secure storage for secrets.
- Ensure RTDB security rules are configured before production use.

## License

Internal/private project. Add a license file if you plan to distribute externally.

---

Developed by Khizar
