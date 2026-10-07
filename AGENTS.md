# AI Agent Guidance & Coding Standards

This document defines guidelines, domain terminology, and development principles for AI agents working in the **Prevail** repository.

---

## 1. Domain Terminology & Vocabulary Rules

* **Skill (Mandatory Term):** Always use the term **`Skill`** in all code, data models, filenames, UI labels, and documentation. While training concepts may colloquially refer to "patterns", the codebase and user-facing copy must strictly use **`Skill`** (and `SkillVariant`). Canonical skill name for pull-ups is **`PULLUP`**.
* **Strength Week:** Assessment weeks are designated as **`Strength Week`** in all user-facing screens and models (e.g. `StrengthWeek`, `StrengthWeekListScreen`).
* **Base Line:** The minimum line level achieved across all skills in a given assessment. If a skill has not yet been attempted in the current assessment, carry forward the prior assessment's line and indicate that the value is inferred (e.g. `11*` or `(11)`). If no prior assessment exists, exclude unattempted skills from the minimum calculation.
* **Next Line Progress:** The count of skills that have progressed strictly above the assessment's Base Line.
* **Board Percentage:** Overall completion percentage calculated as the sum of all achieved skill line levels in the current assessment divided by 240 ($12 \text{ skills} \times 20$). Unattempted skills count as 0.
* **Best Result Precedence:** Highest Line level $\rightarrow$ Higher Weight $\rightarrow$ Higher Reps $\rightarrow$ Longer Duration $\rightarrow$ Most Recent.
* **Unilateral Movements:** Skills/variants that are one-arm or one-leg (Get-up, Carry, One-Arm Pushup, Pistol, Split Squat, Press) must record weight and/or reps for both **Left** and **Right**. To qualify for a milestone tier, **both sides must meet or exceed** the tier's standard.
* **Profiles & Local-First Guest Mode:** Multi-profile support. The app works fully offline without requiring a Google account. The app remembers and reloads the last used profile across app launches. When a Google account is connected later, local profiles and assessments are merged into the Google Drive AppData store.
* **Progression Zones & Color Palette:** The 20 milestone lines are organized into **5 equal zones** of 4 lines each:
  * **Zone 1 (Lines 1–4):** Primary **Blue** (light shade for line 1 $\rightarrow$ darker shades up to line 4).
  * **Zone 2 (Lines 5–8):** Primary **Red** (light shade for line 5 $\rightarrow$ darker shades up to line 8).
  * **Zone 3 (Lines 9–12):** Primary **Yellow** (light shade for line 9 $\rightarrow$ darker shades up to line 12).
  * **Zone 4 (Lines 13–16):** Primary **Green** (light shade for line 13 $\rightarrow$ darker shades up to line 16).
  * **Zone 5 (Lines 17–20):** Primary **Black** (light charcoal for line 17 $\rightarrow$ pure black for line 20).
  *(Legacy 20-color scheme is preserved in `docs/resources/milestone_zones.json` for historical reference).*

---

## 2. Technical Stack & Architecture

* **Framework:** Flutter (Targeting Android, iOS, and Web via GitHub Pages).
* **State Management:** Riverpod (or BLoC) with unidirectional data flow.
* **Architecture Pattern:** Clean Architecture / Repository Pattern.
  * `core/`: Constants, theme, utilities, failure/error handling.
  * `data/`: Models, local datasources (Drift/Hive), remote datasources (Google Drive AppData API), repository implementations.
  * `domain/`: Entities, repository interfaces, use cases / business rules (calculations for Base Line, Board Percentage, milestone progression).
  * `presentation/`: Screens, widgets, state providers.
* **Storage & Backend:** Zero-maintenance, 100% serverless / local-first.
  * Local Cache: Drift (SQLite) or Hive for immediate offline availability.
  * Cloud Persistence: Google Drive AppData space (`https://www.googleapis.com/auth/drive.appdata`). Data is stored as JSON/SQLite files hidden in the user's private app data folder.
* **Future Health Integrations:** Abstract data sources behind repository interfaces (`HealthTelemetryRepository`, `NutritionRepository`) so native OS Health Connect / Apple HealthKit plugins can be introduced in later phases without modifying core domain entities.

---

## 3. Navigation & Screen Flow Standards

1. **Default Screen:** `StrengthWeekListScreen` (displays past Strength Weeks with newest on top, summary metrics, "Start Assessment" button, and a settings gear icon).
2. **Back Navigation:** Every screen other than the root `StrengthWeekListScreen` must provide an explicit back navigation button in the top app bar that navigates to its parent screen.
3. **Screen Naming Convention:**
   * `StrengthWeekListScreen` (Root / Default)
   * `SettingsScreen` (Child of List)
   * `StrengthWeekDetailScreen` (Child of List)
   * `SkillAssessmentScreen` (Child of Detail)
   * `HistoricalSkillAssessmentScreen` (Child of Skill Assessment)
   * `SkillChartScreen` (Child of Skill Assessment)
   * `SkillAssessmentResultScreen` (Read-only modal or page, child of Skill Assessment or History)

---

## 4. Coding & Implementation Guidelines

* **Pure Dart Domain Logic:** Domain entities, scoring algorithms (Base Line, Board %, Next Line count), and milestone matchers must not depend on Flutter UI libraries. Keep them purely unit-testable.
* **Static Milestone Bundling:** The 20-tier milestone standards are stored inside the app bundle as a structured resource file (`assets/data/milestones.json`) and loaded on app initialization.
* **Defensive Calculations:** When an assessment has unattempted or missing skills, the calculations must handle `null` gracefully (see `docs/phase_1_design.md` for exact rules).
* **Cross-Platform UI:**
  * Do not use WebViews for core functionality.
  * Optimize layouts to be responsive for mobile handsets and desktop/tablet browser windows (GitHub Pages).
  * Follow Material 3 guidelines with rich aesthetic tokens (dark mode, crisp typographic hierarchy, distinct milestone tier badge colors matching the chart).

---

## 5. Testing & Verification Requirements

* Any modification to milestone calculation logic or progression formulas must include comprehensive unit tests.
* Ensure all models support bidirectional JSON serialization to maintain compatibility with Google Drive AppData sync.
* Do not introduce server dependencies or proprietary APIs that require backend hosting fees.
