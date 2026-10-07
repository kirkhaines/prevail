# Phase 1: Strength Week & Milestone Tracker Design Document

This document provides the exhaustive specification for Phase 1 of **Prevail**. It details screen layouts, widget hierarchies, navigation paths, dynamic input fields, and calculation rules.

---

## 1. Domain Terminology & Display Rules

* **Skill:** The single canonical term used across all screens, code, and documentation (replacing "pattern").
* **Strength Week:** The title used on all screens to denote an assessment week.
* **Assessment Date Key:** Formatted as the Monday date of the assessment week (e.g. `2026-10-05` or `Oct 5, 2026`).
* **Back Navigation:** Every screen (except the root `StrengthWeekListScreen`) includes an explicit back navigation button in the top app bar returning to its direct parent.

---

## 2. Screen Hierarchy & Navigation Map

```
┌────────────────────────────────────────────────────────────────────────┐
│                   StrengthWeekListScreen (Root / Default)              │
│       • Gear Icon (Top Right) ──► SettingsScreen                       │
│       • "+ Start Assessment" Button                                    │
│       • Tap Assessment Row ─────► StrengthWeekDetailScreen             │
└────────────────────────────────────────┬───────────────────────────────┘
                                         │
                                         ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      StrengthWeekDetailScreen                          │
│       • Header: Date, Base Line, Next Line, Board %, Body Weight Field │
│       • Skills List (12 rows)                                          │
│       • Tap Skill Row ──────────► SkillAssessmentScreen                │
└────────────────────────────────────────┬───────────────────────────────┘
                                         │
            ┌────────────────────────────┼────────────────────────────┐
            ▼                            ▼                            ▼
┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│HistoricalSkillAssess- │   │ SkillAssessmentScreen │   │   SkillChartScreen    │
│      mentScreen       │   │                       │   │                       │
│• All past best results│◄──┤• Last 2 results       ├──►│• Full 20 milestone   │
│• Tap ──► Result Screen│   │• Next Line milestones │   │  lines for this skill │
└───────────┬───────────┘   │• Current logged tests │   └───────────────────────┘
            │               │• Log New Result Form  │
            │               └───────────┬───────────┘
            │                           │
            └───────────┬───────────────┘
                        ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      SkillAssessmentResultScreen                       │
│       • Read-only inspector for logged result                          │
│       • Header: Assessment Date & Skill                                │
│       • Variant, Weight, Duration, Reps, Calculated Line, RPE, Notes   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Screen Specifications

### 3.1. StrengthWeekListScreen (Default Home Screen)
* **Route:** `/`
* **App Bar:**
  * Title: `Prevail Strength Weeks`
  * Action: Settings gear button (navigates to `/settings`).
* **Header / Top Action:**
  * Primary Action Button: `+ Start Assessment` (opens date-picker defaulted to current week's Monday, creates new Strength Week, navigates to its detail screen).
* **Assessment List:**
  * Sorted chronologically with **most recent assessment on top**.
  * **Row Card Elements:**
    1. **Date:** Monday of the assessment week (e.g., `Oct 5, 2026`).
    2. **Base Line:** Minimum line number across all assessed skills (e.g. `Base Line: 11`).
    3. **Next Line Progress:** Count of skills strictly greater than Base Line (e.g. `Next Line: 7 / 12`).
    4. **Board Percentage:** Aggregate score percentage badge (e.g. `74%`).
  * **Tap Interaction:** Opens `/assessment/:id` (`StrengthWeekDetailScreen`).

---

### 3.2. SettingsScreen
* **Route:** `/settings`
* **App Bar:** Back button $\leftarrow$ `Prevail Strength Weeks`
* **Section 1: Active Profile**
  * Profile dropdown picker (automatically persists selection to device preferences so the last profile used is reloaded on app start).
  * Shows active profile details: Name, Gender (`Male` / `Female`).
  * Button: `+ Add Profile` (dialog prompting for Name and Gender; saved immediately to local storage).
* **Section 2: Google Drive Cloud Sync (Optional / Guest Mode Support)**
  * **Status:** Works 100% offline without connecting a Google account.
  * Connection Status indicator (`Not Connected (Local Only)` / `Connected as user@gmail.com`).
  * Button: `Sign in with Google & Merge` / `Disconnect`.
  * **Merge Behavior:** When connecting a Google account for the first time, any existing local profiles and assessment data are automatically merged with whatever already exists in Google Drive `appDataFolder` without data loss or overwriting.
  * Button: `Test Connection & Sync Now` (triggers bidirectional push/pull check).
  * Last Synced timestamp display.
* **Section 3: App Information & Reference Catalogs**
  * App version.
  * Milestone catalog version (380 records loaded from `milestones.json`).
  * Link / viewer to Skill Catalog Reference (`docs/resources/skills_catalog.json`).

---

### 3.3. StrengthWeekDetailScreen (Assessment Screen)
* **Route:** `/assessment/:id`
* **App Bar:** Back button $\leftarrow$ `Strength Weeks`, Title: `Strength Week (Date)`
* **Top Summary Section:**
  * Cards displaying:
    * **Assessment Date:** (Monday of the week)
    * **Base Line:** Current minimum line (marked with `*` or parens if any value is inferred from prior assessments).
    * **Next Line Progress:** Count of skills exceeding Base Line.
    * **Board Percentage:** Real-time percentage badge (unattempted skills count as 0, divided by 240).
  * **Body Weight Input Field:**
    * Numeric text field with toggle for **lbs** and **kg** (persisted to profile preference).
    * Used dynamically to calculate required exact unrounded targets for % BW skills (Deadlift, RFEDSS, Single Side Carry March, weighted Pullups/Pushups).
* **Skills List Section (12 Standard Skills):**
  * Rows for each skill:
    * `GETUP`, `PLANK`, `CRAWL`, `SINGLE SIDE CARRY MARCH`, `PULLUP`, `DEAD HANG`, `PUSHUP`, `PISTOL`, `RFEDSS`, `SQUAT`, `PRESS`, `DEADLIFT`.
  * **Row Layout:**
    * **Skill Name:** Bold headline (e.g., `DEADLIFT`).
    * **Best Result Variant:** e.g., `DEADLIFT`.
    * **Best Result Summary:** Subdued caption (e.g., `125% BW x 3 reps` or `24 kg L / 24 kg R x 1 rep`).
    * **Line Number Badge:** Highlighted tier badge with milestone zone color & shade (e.g. `Line 11 - Deep Amber` or `Line (11)*` if inferred).
  * **Tap Interaction:** Navigates to `/assessment/:id/skill/:skillName` (`SkillAssessmentScreen`).

---

### 3.4. SkillAssessmentScreen
* **Route:** `/assessment/:id/skill/:skillName`
* **App Bar:** Back button $\leftarrow$ `Strength Week`, Title: `Skill: {Skill Name}`
* **Section 1: Historical Precedent (Last 2 Assessments)**
  * **Header:** Title `Recent History`, right action button `All History ➔` (navigates to `HistoricalSkillAssessmentScreen`).
  * List displaying up to the last 2 recorded assessments for this skill:
    * Assessment Date | Skill Variant | Best Result Summary | Line Number Badge.
    * Tap on row opens read-only `SkillAssessmentResultScreen`.
* **Section 2: Next Milestone Targets**
  * **Header:** Title `Upcoming Milestones`, right action button `Full Chart ➔` (navigates to `SkillChartScreen`).
  * **Dynamic Progression Rule:** Displays milestone lines starting immediately from the next level above the user's current highest result (combining prior historical best and any newly logged current assessment attempts). If a new attempt is saved, this list automatically refreshes to start at the new next line.
  * Displays: Level number, zone color & shade badge, variant name, and requirement summary.
* **Section 3: Current Assessment Results**
  * Header: `Current Assessment Attempts`
  * Displays all attempts logged during this Strength Week session for this skill.
  * Tap on an attempt opens read-only `SkillAssessmentResultScreen`.
* **Section 4: Log Result Form**
  * Header: `Log Result`
  * **Skill Variant Dropdown:** Populated with valid variants for this skill from the milestone chart.
  * **Dynamic Inputs (Rendered based on selected Variant from `skills_catalog.json`):**
    * **Bilateral Movements (e.g. Deadlift, Squat, Plank, Standard Pushup):**
      * *Weight (kg/lbs):* Single load input.
      * *Duration (mm:ss):* Single duration input.
      * *Reps:* Single reps input.
    * **Unilateral Movements (Get-up, Carry, One-Arm Pushup, Pistol, Split Squat, Press):**
      * Two-column or split inputs for **Left** and **Right**:
        * *Weight Left (kg/lbs)* & *Weight Right (kg/lbs)*
        * *Reps Left* & *Reps Right*
    * *Target Preview:* Live display of exact calculated target load for % BW milestones (unrounded).
  * **Calculated Line Number Display:** Live reactive badge showing the matched milestone line as the user enters values (both Left and Right must meet or exceed the criteria for unilateral variants).
  * **RPE (Rate of Perceived Exertion):** Slider or selector (1–10).
  * **Notes:** Multi-line text field.
  * **Submit Button:** `Save Result`.

---

### 3.5. HistoricalSkillAssessmentScreen
* **Route:** `/assessment/:id/skill/:skillName/history`
* **App Bar:** Back button $\leftarrow$ `{Skill Name}`, Title: `{Skill Name} History`
* **Content:**
  * Full reverse-chronological list of all historical assessments for this skill.
  * Row elements: Assessment Date, Skill Variant, Best Result Summary, Line Number Badge.
  * Tap on row opens read-only `SkillAssessmentResultScreen`.

---

### 3.6. SkillChartScreen
* **Route:** `/assessment/:id/skill/:skillName/chart`
* **App Bar:** Back button $\leftarrow$ `{Skill Name}`, Title: `{Skill Name} Standards`
* **Content:**
  * Complete 20-level milestone table filtered for this skill and the active profile's gender.
  * Columns:
    1. **Level:** 1 to 20 with zone color chip.
    2. **Zone:** Zone 1–5 indicator.
    3. **Variant:** Variant name.
    4. **Standard Summary:** Formatted requirements (e.g. `16 kg x 5 reps`, `0:01:30 hold`, `150% BW x 1 rep`).

---

### 3.7. SkillAssessmentResultScreen
* **Route:** `/result/:resultId`
* **App Bar:** Back button $\leftarrow$, Title: `Assessment Result`
* **Content (Read-Only Card View):**
  * Header: Assessment Date & Skill Name.
  * Achieved Level Chip (e.g. `Level 11 - Zone 3 (Deep Amber)`).
  * Variant Name.
  * Metrics Card:
    * If Bilateral: Weight / Load, Reps, Duration, Calculated % Body Weight.
    * If Unilateral: Left Side (Weight & Reps) vs. Right Side (Weight & Reps).
  * RPE Score (e.g. `8.5 / 10`).
  * Notes / Coach Comments.

---

## 4. Skills Catalog & Dynamic Input Reference

Reference dataset defined in `docs/resources/skills_catalog.json`:

| Skill | Category | Unilateral? | Applicable Form Fields | Target Evaluation |
| :--- | :--- | :--- | :--- | :--- |
| **CRAWL** | Core / Locomotion | No | `Duration`, `Reps` | Duration threshold |
| **DEAD HANG** | Grip / Hanging | No | `Duration` | Duration threshold |
| **DEADLIFT** | Posterior Chain | No | `Weight`, `Reps` | % BW load $\times$ Reps |
| **GETUP** | Stability / Arm | **Yes (Arm)** | `Weight (L/R)`, `Reps (L/R)` | Weight lifted $\times$ Reps (both sides) |
| **PISTOL** | Single Leg Squat | **Yes (Leg)\*** | `Weight (L/R)`, `Reps (L/R)` | Variant progression $\rightarrow$ Weight (both legs) |
| **PLANK** | Core Stability | No | `Duration` | Duration threshold |
| **PRESS** | Vertical Push | **Yes (Arm)** | `Weight (L/R)`, `Reps (L/R)` | Weight lifted $\times$ Reps (both arms) |
| **PULLUP** | Vertical Pull | No | `Variant`, `Duration`, `Weight`, `Reps` | Variant $\rightarrow$ Duration / Reps / Weight |
| **PUSHUP** | Horizontal Push | **Yes (for OAPU/OAOLPU)** | Bilateral `Reps` OR `Reps (L/R)` for one-arm | Angle variant $\rightarrow$ Reps $\rightarrow$ 1-Arm (L/R) |
| **RFEDSS** | Split Squat | **Yes (Leg)** | `Weight (L/R)`, `Reps (L/R)` | % BW load $\times$ Reps (both legs) |
| **SINGLE SIDE CARRY MARCH** | Loaded Carry | **Yes (Arm)** | `Weight (L/R)`, `Reps (L/R)` | % BW load $\times$ Reps/Marches (both sides) |
| **SQUAT** | Bilateral Squat | No | `Weight`, `Reps` | Weight lifted $\times$ Reps |

*\*Note: Early Pistol variants like Narrow Stance Squat are bilateral, while lunges, split squats, heel taps, and pistols are unilateral.*

---

## 5. Calculation Specifications & Business Rules

### 5.1. Base Line Calculation
$$\text{Base Line} = \min_{i \in \text{Available Skills}} (\text{Effective Line}_i)$$
* **Inferred Carry-Forward:** If a skill has not been attempted in the current Strength Week:
  * If a prior assessment exists for that skill, carry forward the prior line level and render with an inferred indicator (`*` or parens).
  * If no prior assessment exists, exclude the skill from the minimum calculation.

### 5.2. Next Line Progress
$$\text{Next Line Progress} = \text{Count of skills where } \text{Effective Line} > \text{Base Line}$$

### 5.3. Board Percentage
$$\text{Board Percentage} = \frac{\sum_{i=1}^{12} \text{Achieved Current Line}_i}{240} \times 100\%$$
* Unattempted skills in the current week contribute **0** to the numerator. The denominator is always fixed at $240$ ($12 \times 20$).

### 5.4. Best Result Selection Precedence
When multiple attempts are recorded for a skill in a single assessment, the best result is chosen by:
1. **Line Level:** Highest line achieved.
2. **Load / Weight:** Higher weight lifted.
3. **Reps:** Higher rep count.
4. **Duration:** Longer duration.
5. **Recency:** Most recent attempt timestamp.

### 5.5. Unilateral Qualification Rule
For unilateral variants, both left and right sides must meet or exceed the required target to qualify for that milestone tier:
$$\text{Achieved Line} = \min(\text{Line}_{\text{Left}}, \text{Line}_{\text{Right}})$$

---

## 6. Progression Zones & Color Tokens (5-Zone System)

The 20 progression levels are grouped into 5 equal zones of 4 lines each. Within each zone, the color progresses from a light tint on the lowest line to a deep/dark shade on the highest line:

| Zone | Levels | Primary Color | Shade Progression | Legacy Reference Color |
| :---: | :---: | :---: | :--- | :--- |
| **Zone 1** | **1–4** | **Blue** | Line 1: Light Blue (`#90CAF9`)<br>Line 2: Sky Blue (`#42A5F5`)<br>Line 3: Royal Blue (`#1E88E5`)<br>Line 4: Deep Navy (`#1565C0`) | Line 1: White<br>Line 2: Blue<br>Line 3: Yellow<br>Line 4: Green |
| **Zone 2** | **5–8** | **Red** | Line 5: Light Coral (`#EF9A9A`)<br>Line 6: Salmon Red (`#E57373`)<br>Line 7: Crimson (`#E53935`)<br>Line 8: Deep Burgundy (`#B71C1C`) | Line 5: Red<br>Line 6: Light Blue<br>Line 7: Orange<br>Line 8: Bright Pink |
| **Zone 3** | **9–12** | **Yellow** | Line 9: Pale Amber (`#FFF59D`)<br>Line 10: Gold Yellow (`#FFEE58`)<br>Line 11: Deep Amber (`#FDD835`)<br>Line 12: Dark Goldenrod (`#FBC02D`) | Line 9: Bright Green<br>Line 10: Purple<br>Line 11: Gold<br>Line 12: Pink |
| **Zone 4** | **13–16** | **Green** | Line 13: Light Sage (`#A5D6A7`)<br>Line 14: Jade Green (`#66BB6A`)<br>Line 15: Forest Green (`#388E3C`)<br>Line 16: Deep Hunter Green (`#1B5E20`) | Line 13: Eggshell<br>Line 14: Maroon<br>Line 15: Grey<br>Line 16: Brown |
| **Zone 5** | **17–20** | **Black** | Line 17: Medium Charcoal (`#757575`)<br>Line 18: Dark Charcoal (`#424242`)<br>Line 19: Near Black (`#212121`)<br>Line 20: Jet Black (`#0A0A0A`) | Line 17: Silver<br>Line 18: Burnt Orange<br>Line 19: Light Purple<br>Line 20: Black |

*Full token mapping and hex values are stored in `docs/resources/milestone_zones.json`.*
