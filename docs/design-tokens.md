# LIPILY Design Tokens

> **Single source of truth for all UI colours, typography, spacing, and component-level tokens.**  
> All Flutter code MUST reference these token names — never hardcode hex values or raw font sizes.  
> Tokens are defined in `lib/core/theme/tokens.dart` and used via `Theme.of(context)`.

---

## 1. Brand Colours

```dart
// lib/core/theme/tokens.dart

abstract class LipilyColors {
  // ─── Brand ────────────────────────────────────────────────────────────
  static const brandPrimary      = Color(0xFF1A1A2E); // Deep navy — primary bg
  static const brandAccent       = Color(0xFFE94560); // Electric crimson — CTA, links
  static const brandGold         = Color(0xFFFFD700); // Award / pro badge
  static const brandInk          = Color(0xFF0F3460); // Dark ink — secondary bg

  // ─── Neutrals ─────────────────────────────────────────────────────────
  static const white             = Color(0xFFFFFFFF);
  static const grey50            = Color(0xFFFAFAFA);
  static const grey100           = Color(0xFFF5F5F5);
  static const grey200           = Color(0xFFEEEEEE);
  static const grey300           = Color(0xFFE0E0E0);
  static const grey400           = Color(0xFFBDBDBD);
  static const grey500           = Color(0xFF9E9E9E);
  static const grey600           = Color(0xFF757575);
  static const grey700           = Color(0xFF616161);
  static const grey800           = Color(0xFF424242);
  static const grey900           = Color(0xFF212121);
  static const black             = Color(0xFF000000);

  // ─── Semantic ─────────────────────────────────────────────────────────
  static const success           = Color(0xFF4CAF50);
  static const successLight      = Color(0xFFE8F5E9);
  static const warning           = Color(0xFFFF9800);
  static const warningLight      = Color(0xFFFFF3E0);
  static const error             = Color(0xFFF44336);
  static const errorLight        = Color(0xFFFFEBEE);
  static const info              = Color(0xFF2196F3);
  static const infoLight         = Color(0xFFE3F2FD);
}
```

---

## 2. Dark Theme (Primary) Colour Roles

LIPILY's primary theme is **dark** (writer-friendly, low-eye-strain). Light theme is available but secondary.

| Role | Dark Token | Hex | Light Token | Hex |
|---|---|---|---|---|
| `scaffoldBackground` | `brandPrimary` | `#1A1A2E` | `grey50` | `#FAFAFA` |
| `surface` | `brandInk` | `#0F3460` | `white` | `#FFFFFF` |
| `surfaceVariant` | `grey900` | `#212121` | `grey100` | `#F5F5F5` |
| `primary` | `brandAccent` | `#E94560` | `brandAccent` | `#E94560` |
| `onPrimary` | `white` | `#FFFFFF` | `white` | `#FFFFFF` |
| `onSurface` | `grey100` | `#F5F5F5` | `grey900` | `#212121` |
| `onSurfaceVariant` | `grey400` | `#BDBDBD` | `grey600` | `#757575` |
| `outline` | `grey700` | `#616161` | `grey300` | `#E0E0E0` |
| `error` | `error` | `#F44336` | `error` | `#F44336` |

---

## 3. Content-Source Badge Colours

Every AI-touched block shows a coloured badge. These colours are FIXED across light and dark themes.

| ContentSource value | Badge colour | Hex | Text colour |
|---|---|---|---|
| `ai_generated` | AI Violet | `#7B2FBE` | White |
| `human_confirmed` | Confirm Teal | `#00897B` | White |
| `human_edited` | Edit Amber | `#F59E0B` | Black |
| `human_written` | Ink Navy (no badge) | — | — |

```dart
abstract class ContentSourceColors {
  static const aiGenerated    = Color(0xFF7B2FBE);
  static const humanConfirmed = Color(0xFF00897B);
  static const humanEdited    = Color(0xFFF59E0B);
  // human_written: no badge rendered
}
```

---

## 4. Block-Type Accent Colours

Used as left-border or heading accent in the script editor.

| Block type | Colour name | Hex |
|---|---|---|
| `scene_heading` | Scene Blue | `#1565C0` |
| `action` | Action Slate | `#455A64` |
| `dialogue` | Dialogue Teal | `#00695C` |
| `character` | Character Purple | `#6A1B9A` |
| `parenthetical` | Paren Grey | `#78909C` |
| `transition` | Transition Rose | `#AD1457` |
| `shot` | Shot Orange | `#E65100` |
| `note` | Note Yellow | `#F9A825` |
| `beat` | Beat Pink | `#C2185B` |

```dart
abstract class BlockTypeColors {
  static const sceneHeading   = Color(0xFF1565C0);
  static const action         = Color(0xFF455A64);
  static const dialogue       = Color(0xFF00695C);
  static const character      = Color(0xFF6A1B9A);
  static const parenthetical  = Color(0xFF78909C);
  static const transition     = Color(0xFFAD1457);
  static const shot           = Color(0xFFE65100);
  static const note           = Color(0xFFF9A825);
  static const beat           = Color(0xFFC2185B);
}
```

---

## 5. Collaboration Cursor Colours

Each collaborator in a live session gets a unique cursor colour, cycled in this sequence.

```dart
abstract class CursorColors {
  static const List<Color> sequence = [
    Color(0xFF2196F3), // Blue
    Color(0xFF4CAF50), // Green
    Color(0xFFFF5722), // Deep Orange
    Color(0xFF9C27B0), // Purple
    Color(0xFFFF9800), // Orange
    Color(0xFF00BCD4), // Cyan
    Color(0xFFE91E63), // Pink
    Color(0xFF8BC34A), // Light Green
    Color(0xFF3F51B5), // Indigo
    Color(0xFFFF4081), // Pink Accent
  ];
}
```

---

## 6. Revision Marker Colours

When track-changes or revision mode is on, each revision pass uses a different colour.

| Revision pass | Colour | Hex |
|---|---|---|
| Pass 1 | Blue | `#1976D2` |
| Pass 2 | Red | `#D32F2F` |
| Pass 3 | Yellow (highlight) | `#FBC02D` |
| Pass 4 | Green | `#388E3C` |
| Pass 5+ | Asterisk only (cyan) | `#0097A7` |

---

## 7. Typography

### Font Families

| Family | Usage | Source |
|---|---|---|
| `CourierPrime` | Script editor — all block types | Google Fonts / bundled |
| `CourierPrime-Bold` | Scene headings, character names | Same family |
| `CourierPrime-Italic` | Stage directions, notes | Same family |
| `NotoSansDevanagari` | Hindi script content | Google Fonts / bundled |
| `NotoSansTamil` | Tamil script content | Google Fonts / bundled |
| `NotoNaskhArabic` | Arabic/Urdu content | Google Fonts / bundled |
| `Inter` | All non-script UI (nav, cards, labels) | Google Fonts |
| `Inter-SemiBold` | Headings, subheadings | Same family |

### Type Scale

All sizes in **logical pixels (dp)**.

| Token name | Size | Weight | Line height | Usage |
|---|---|---|---|---|
| `displayLarge` | 57 | 400 | 64 | Hero screens only |
| `displayMedium` | 45 | 400 | 52 | — |
| `displaySmall` | 36 | 400 | 44 | — |
| `headlineLarge` | 32 | 600 | 40 | Screen titles |
| `headlineMedium` | 28 | 600 | 36 | Section headers |
| `headlineSmall` | 24 | 600 | 32 | Card titles |
| `titleLarge` | 22 | 500 | 28 | List item titles |
| `titleMedium` | 16 | 500 | 24 | Dialog titles |
| `titleSmall` | 14 | 500 | 20 | Chip labels |
| `bodyLarge` | 16 | 400 | 24 | Primary body text |
| `bodyMedium` | 14 | 400 | 20 | Secondary body |
| `bodySmall` | 12 | 400 | 16 | Captions |
| `labelLarge` | 14 | 500 | 20 | Buttons |
| `labelMedium` | 12 | 500 | 16 | Tab labels |
| `labelSmall` | 11 | 500 | 16 | Badges, chips |

### Script Editor Typography (Fixed-width)

| Block type | Font | Size | Weight | Case |
|---|---|---|---|---|
| Scene heading | CourierPrime | 12pt (16sp) | Bold | UPPERCASE |
| Action | CourierPrime | 12pt (16sp) | Regular | Sentence |
| Character | CourierPrime | 12pt (16sp) | Bold | UPPERCASE |
| Dialogue | CourierPrime | 12pt (16sp) | Regular | Sentence |
| Parenthetical | CourierPrime | 12pt (16sp) | Regular | (Sentence) |
| Transition | CourierPrime | 12pt (16sp) | Bold | UPPERCASE: |
| Shot | CourierPrime | 12pt (16sp) | Regular | Sentence |
| Note | CourierPrime | 12pt (16sp) | Italic | Sentence |

> **12pt = 16sp** at 96dpi. Use `16.0` logical pixels in Flutter.

---

## 8. Spacing System

Base unit: **4dp**. All spacing values are multiples of 4.

```dart
abstract class LipilySpacing {
  static const double xs    =  4.0;  //  4dp
  static const double sm    =  8.0;  //  8dp
  static const double md    = 12.0;  // 12dp
  static const double base  = 16.0;  // 16dp — default padding
  static const double lg    = 24.0;  // 24dp
  static const double xl    = 32.0;  // 32dp
  static const double xxl   = 48.0;  // 48dp
  static const double xxxl  = 64.0;  // 64dp
}
```

### Layout Margins

| Context | Value |
|---|---|
| Mobile screen horizontal padding | 16dp |
| Tablet screen horizontal padding | 24dp |
| Desktop screen max content width | 960dp |
| Card internal padding | 16dp |
| Dialog padding | 24dp |
| Bottom sheet padding | 16dp horizontal, 24dp vertical |
| Script editor left margin (scene heading) | 1.5 inches = 144dp |
| Script editor right margin | 1.0 inch = 96dp |
| Script editor dialogue indent | 2.5 inches = 240dp |
| Script editor character indent | 3.7 inches = 355dp |

---

## 9. Border Radius

```dart
abstract class LipilyRadius {
  static const double none  =  0.0;
  static const double xs    =  2.0;
  static const double sm    =  4.0;
  static const double md    =  8.0;
  static const double lg    = 12.0;
  static const double xl    = 16.0;
  static const double xxl   = 24.0;
  static const double full  = 999.0; // Pill / FAB
}
```

---

## 10. Elevation & Shadow

| Level | dp | Usage |
|---|---|---|
| 0 | 0 | Flat surfaces, editor background |
| 1 | 1 | Cards at rest |
| 2 | 3 | Raised buttons, hovered cards |
| 3 | 6 | Dialogs, bottom sheets |
| 4 | 8 | FAB |
| 5 | 12 | Modals, drawers |

---

## 11. Animation Durations

```dart
abstract class LipilyDuration {
  static const micro   = Duration(milliseconds:  75); // Ripple, press feedback
  static const fast    = Duration(milliseconds: 150); // Toggle, chip
  static const normal  = Duration(milliseconds: 250); // Route transitions
  static const slow    = Duration(milliseconds: 400); // Panel expand/collapse
  static const xslow  = Duration(milliseconds: 600); // Onboarding, celebration
}
```

---

## 12. Subscription Tier Badges

| Tier | Badge text | Background | Text colour |
|---|---|---|---|
| Free | — | — | — |
| Pro | PRO | `#E94560` (brandAccent) | White |
| Studio | STUDIO | `#FFD700` (brandGold) | Black |
| Reviewer | REVIEWER | `#00897B` | White |
| Producer | PRODUCER | `#0F3460` (brandInk) | White |

---

## 13. AI Feature — "✦" Spark Icon

The AI spark icon `✦` (U+2736 SIX POINTED BLACK STAR) is used as the universal AI indicator across the app.

| State | Colour |
|---|---|
| Idle | `grey500` |
| Active (generating) | `brandAccent` with pulsing animation |
| Completed | `brandAccent` static |
| Error | `error` |

---

## 14. Flutter Theme Setup

```dart
// lib/core/theme/lipily_theme.dart

ThemeData lipilyDarkTheme() => ThemeData(
  useMaterial3: true,
  brightness: Brightness.dark,
  colorScheme: const ColorScheme.dark(
    primary:          LipilyColors.brandAccent,
    onPrimary:        LipilyColors.white,
    secondary:        LipilyColors.brandGold,
    onSecondary:      LipilyColors.black,
    surface:          LipilyColors.brandInk,
    onSurface:        LipilyColors.grey100,
    surfaceVariant:   LipilyColors.grey900,
    onSurfaceVariant: LipilyColors.grey400,
    background:       LipilyColors.brandPrimary,
    onBackground:     LipilyColors.grey100,
    error:            LipilyColors.error,
    onError:          LipilyColors.white,
    outline:          LipilyColors.grey700,
  ),
  fontFamily: 'Inter',
  textTheme: lipilyTextTheme(),
);
```

---

## 15. Token File Locations

| File | Contents |
|---|---|
| `lib/core/theme/tokens.dart` | `LipilyColors`, `ContentSourceColors`, `BlockTypeColors`, `CursorColors`, `LipilySpacing`, `LipilyRadius`, `LipilyDuration` |
| `lib/core/theme/lipily_theme.dart` | `ThemeData` construction for dark + light |
| `lib/core/theme/text_theme.dart` | `TextTheme` with all type scale entries |
| `lib/core/theme/script_theme.dart` | `TextStyle` per block type for the editor |
| `lib/core/theme/subscription_badge.dart` | Tier badge widget |
| `lib/core/theme/content_source_badge.dart` | `ContentSource` badge widget |
