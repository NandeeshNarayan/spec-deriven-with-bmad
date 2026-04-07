# LIPILY — Flutter `pubspec.yaml` Reference

> **Purpose:** Exact package names and version constraints for the Flutter app.  
> **Agent rule:** Never guess package names. Use exactly what is listed here.  
> **Last updated:** 2026-04-07

---

## `app/pubspec.yaml`

```yaml
name: lipily
description: Professional AI-powered screenwriting platform.
version: 1.0.0+1
publish_to: 'none'

environment:
  sdk: '>=3.3.0 <4.0.0'
  flutter: '>=3.19.0'

dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter

  # ─── Auth & Backend ────────────────────────────────────────
  supabase_flutter: ^2.5.0
  flutter_secure_storage: ^9.2.2

  # ─── State Management ──────────────────────────────────────
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5
  hooks_riverpod: ^2.5.1
  flutter_hooks: ^0.20.5

  # ─── Navigation ────────────────────────────────────────────
  go_router: ^13.2.0

  # ─── HTTP ──────────────────────────────────────────────────
  dio: ^5.4.3
  retrofit: ^4.1.0

  # ─── Offline / Sync ────────────────────────────────────────
  drift: ^2.18.0
  sqlite3_flutter_libs: ^0.5.22
  path_provider: ^2.1.3
  path: ^1.9.0
  powersync: ^1.3.0

  # ─── Yjs CRDT ──────────────────────────────────────────────
  y_crdt: ^0.3.0

  # ─── Payments ──────────────────────────────────────────────
  # Stripe: using Stripe Checkout (opens external browser)
  # No native Stripe SDK needed — all via Rust backend
  url_launcher: ^6.3.0

  # ─── Charts ────────────────────────────────────────────────
  fl_chart: ^0.68.0

  # ─── UI Components ─────────────────────────────────────────
  flutter_svg: ^2.0.10+1
  cached_network_image: ^3.3.1
  shimmer: ^3.0.0
  lottie: ^3.1.0           # Confetti / onboarding animations
  image_picker: ^1.1.2     # Avatar upload
  file_picker: ^8.0.7      # Script import (.fountain/.fdx)
  share_plus: ^9.0.0

  # ─── Audio (Listen the Script player) ─────────────────────
  just_audio: ^0.9.40
  audio_session: ^0.1.21

  # ─── Drag & Drop (Beat Board, Scene Navigator) ────────────
  reorderables: ^0.6.0

  # ─── Internationalization ──────────────────────────────────
  intl: ^0.19.0

  # ─── Analytics ─────────────────────────────────────────────
  posthog_flutter: ^4.3.1

  # ─── Error Monitoring ──────────────────────────────────────
  sentry_flutter: ^8.3.0

  # ─── Utils ─────────────────────────────────────────────────
  equatable: ^2.0.5
  freezed_annotation: ^2.4.4
  json_annotation: ^4.9.0
  collection: ^1.18.0
  uuid: ^4.4.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter

  # ─── Code Generation ───────────────────────────────────────
  build_runner: ^2.4.11
  riverpod_generator: ^2.4.0
  drift_dev: ^2.18.0
  freezed: ^2.5.2
  json_serializable: ^6.8.0
  retrofit_generator: ^8.1.0

  # ─── Testing ───────────────────────────────────────────────
  mocktail: ^1.0.4
  network_image_mock: ^2.1.1

  # ─── Linting ───────────────────────────────────────────────
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true

  assets:
    - assets/fonts/
    - assets/images/
    - assets/lottie/
    - assets/music/        # Licensed ambient music tracks for Listen feature
    - assets/fallback-images/  # 10 generic cinematic fallback images

  fonts:
    - family: CourierPrime
      fonts:
        - asset: assets/fonts/CourierPrime-Regular.ttf
        - asset: assets/fonts/CourierPrime-Bold.ttf
          weight: 700
        - asset: assets/fonts/CourierPrime-Italic.ttf
          style: italic
        - asset: assets/fonts/CourierPrime-BoldItalic.ttf
          weight: 700
          style: italic

    # Noto fonts for non-Latin script support
    - family: NotoSansDevanagari
      fonts:
        - asset: assets/fonts/NotoSansDevanagari-Regular.ttf
        - asset: assets/fonts/NotoSansDevanagari-Bold.ttf
          weight: 700

    - family: NotoSansTamil
      fonts:
        - asset: assets/fonts/NotoSansTamil-Regular.ttf
        - asset: assets/fonts/NotoSansTamil-Bold.ttf
          weight: 700

    - family: NotoNaskhArabic
      fonts:
        - asset: assets/fonts/NotoNaskhArabic-Regular.ttf
        - asset: assets/fonts/NotoNaskhArabic-Bold.ttf
          weight: 700
```

---

## Key Package Notes

### `powersync: ^1.3.0`
- Used for offline-first sync with Supabase
- Story: `story-018`
- Requires `PowerSyncDatabase` wrapped around Drift

### `y_crdt: ^0.3.0`
- Dart/Flutter port of Yjs
- Provides `YDoc`, `YText`, `YMap`, `YArray`
- Works alongside the Rust `yrs` server-side
- Story: `story-010`

### `drift: ^2.18.0`
- Type-safe SQLite ORM for Flutter
- All offline data goes through Drift
- Tables defined in `lib/db/database.dart`
- Story: `story-018`

### `fl_chart: ^0.68.0`
- Bar charts: Writing goals history (story-028)
- Line charts: Pacing analysis (story-036)
- Spider/radar charts: Review sections (story-045)

### `just_audio: ^0.9.40`
- Used in Listen the Script player (story-042) and Dialogue Read-Aloud (story-054)
- Streams audio from S3 pre-signed URLs

### Code generation commands

```bash
# Run all generators
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode (development)
flutter pub run build_runner watch --delete-conflicting-outputs
```

### Generated files (commit to git)
- `*.g.dart` — JSON serialization
- `*.freezed.dart` — Freezed unions
- `*.drift.dart` — Drift query classes
- `*.gr.dart` — go_router route definitions

---

## `analysis_options.yaml`

```yaml
include: package:flutter_lints/flutter.yaml

analyzer:
  errors:
    missing_required_param: error
    missing_return: error
    dead_code: warning
  exclude:
    - '**/*.g.dart'
    - '**/*.freezed.dart'
    - '**/*.drift.dart'
    - '**/*.gr.dart'

linter:
  rules:
    - always_use_package_imports
    - avoid_print
    - prefer_const_constructors
    - prefer_final_locals
    - use_key_in_widget_constructors
    - always_declare_return_types
```
