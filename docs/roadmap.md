# Prevail Application Roadmap

This document outlines the multi-phase roadmap for the Prevail gym tracking application. The architecture is deliberately designed in Phase 1 so that Phase 2 (Daily Programs) and Phase 3 (Health & Nutrition Integrations) plug in modularly without refactoring core components.

---

## Roadmap Overview

```
┌────────────────────────────────────────────────────────┐
│     Phase 1: Strength Week Assessment & Milestones     │
│  • 12 Skills, 20 Progression Tiers (Normalized Chart)  │
│  • Strength Week Scoring (Base Line, Board %, Next)    │
│  • Multi-Profile & Serverless Google Drive AppData Sync│
│  • Flutter Mobile (Android/iOS) & Web (GitHub Pages)   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│      Phase 2: Quarterly Patterns & Daily Programs      │
│  • 9 Daily Program Tracking per Quarter                │
│  • Muscle Group Translation across Skills              │
│  • Reps-to-Weight Normalization & 1RM Translators      │
│  • Recommendation Engine based on last Strength Week   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│     Phase 3: Health Ecosystem & Feedback Loop          │
│  • Body Comp: InBody (via Health Connect / HealthKit)  │
│  • Nutrition: MyFitnessPal (Energy In / Macros)        │
│  • Activity: Google Health / Apple Health (Energy Out) │
│  • Gym Platform: Kilo integration                      │
│  • Composite TDEE Calibration & Target Feedback Loop   │
└────────────────────────────────────────────────────────┘
```

---

## Phase 1: Strength Week & Milestone Tracker (Current Focus)

**Goal:** Deliver a rock-solid, serverless tracker for quarterly Strength Week assessments against the standardized 20-level milestone progression chart.

### Key Deliverables:
1. **Core Domain & Calculations:**
   * Pure Dart domain models for `Profile`, `StrengthWeek`, `SkillAssessment`, `SkillAssessmentResult`, and `Milestone`.
   * Bundled milestone catalog containing the 380 standardized milestone records across 12 skills and gender tiers.
   * Progression engine calculating:
     * **Base Line:** Minimum line number achieved across all assessed skills.
     * **Next Line Progress:** Count of skills strictly exceeding the Base Line.
     * **Board Percentage:** Overall board progress percentage:
       $$\text{Board Percentage} = \frac{\sum \text{Achieved Line Numbers}}{12 \times 20} \times 100\%$$
     * Automatic line-matching algorithm determining achieved line based on input parameters (Weight, Body Weight %, Duration, Reps).
2. **Screens & UI Flows:**
   * `StrengthWeekListScreen` (Default home screen with newest assessments on top, metrics, gear icon, and new assessment CTA).
   * `SettingsScreen` (Google Drive AppData authentication/connection testing, profile selector, profile creation).
   * `StrengthWeekDetailScreen` (Assessment summary metrics, body weight input, list of 12 skills with best results and lines).
   * `SkillAssessmentScreen` (Last 2 historical results, next milestone targets, current assessment logged results, dynamic result entry form with RPE & notes).
   * `HistoricalSkillAssessmentScreen` (Full history of best results for a skill).
   * `SkillChartScreen` (Complete 20-level milestone progression chart for a skill).
   * `SkillAssessmentResultScreen` (Read-only inspector for any logged result).
3. **Data & Synchronization:**
   * Local-first persistence (Drift or Hive).
   * Serverless cloud persistence using Google Drive AppData folder (`drive.appdata` scope).
   * Multi-profile isolation under a single Google account.
4. **Target Deployments:**
   * Android APK / local dev build.
   * Web build deployed statically to GitHub Pages.
   * iOS compatibility prepared (ready for free-tier personal team deployment).

---

## Phase 2: Quarterly Patterns & Daily Programs

**Goal:** Track training sessions throughout the quarter across the 9 standard daily programs, translating resistance, rep schemes, and fatigue across muscle groups.

### Key Deliverables:
1. **Daily Program Logging:**
   * Structured logging for the 9 recurring daily workout templates used throughout each quarter.
   * Logging of sets, reps, load, tempo, and RPE.
2. **Cross-Skill & Muscle Group Translation:**
   * Skill-to-muscle group ontology (e.g. Quad Dominant, Posterior Chain / Hinge, Horizontal Push, Vertical Pull, Anti-Rotation / Core).
   * Translating performance between varying skill variants within a family (e.g. translating RFEDSS progress to Squat, or Single Arm Press to Barbell Press).
3. **Rep-to-Weight Normalization:**
   * Dynamic weight scaling for varying rep targets (e.g. translating a 3-rep max assessment to a 10-rep hypertrophy day using verified strength curves).
   * Handling distinct equipment increments (kettlebells vs. dumbbells vs. barbells).
4. **Recommendation Engine:**
   * Warm-up and working set load recommendations driven by results from the previous Strength Week.

---

## Phase 3: Health Ecosystem & Composite Feedback Loop

**Goal:** Integrate biometric, nutrition, and gym platforms to create a fine-tuning feedback loop that calibrates caloric/activity targets.

### Key Deliverables:
1. **On-Device Health Clearinghouse:**
   * Integration with **Google Health Connect** (Android) and **Apple HealthKit** (iOS).
   * Extracting:
     * InBody body composition (Weight, Skeletal Muscle Mass, Body Fat Percentage).
     * MyFitnessPal nutrition (Total Calories In, Protein, Carbohydrates, Fats).
     * Wearable activity (Active Energy Burned, Basal Metabolic Rate / Resting Energy).
2. **Kilo Gym Platform Integration:**
   * Automated or semi-automated sync with Kilo gym programming and logged scores.
3. **Adaptive Energy Calibration Engine:**
   * Compare theoretical energy deficit/surplus against actual weekly body composition velocity ($\Delta$ Body Weight & $\Delta$ Body Fat).
   * Derive True Total Daily Energy Expenditure (TDEE).
   * Generate actionable feedback suggestions for adjusting nutrition and activity targets in MyFitnessPal / fitness trackers.
