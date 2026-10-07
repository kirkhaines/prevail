# Health Ecosystem Integrations & Feedback Loop Specification

This document details the architectural strategy and design for **Phase 3: Health Ecosystem Integrations**, including InBody, MyFitnessPal, Google Health Connect, Apple HealthKit, Kilo, and the adaptive energy balance feedback loop.

---

## 1. Architectural Strategy: The On-Device Health Clearinghouse

To maintain a **100% serverless, zero-maintenance, zero-cost** application, Prevail integrates external health and nutrition platforms through the user's mobile operating system health stores (**Google Health Connect** on Android and **Apple HealthKit** on iOS) rather than building complex, brittle server-to-server OAuth bridges.

```
┌─────────────────┐       ┌─────────────────┐
│   InBody App    │       │  MyFitnessPal   │
│  (Weight & SMM) │       │ (Calories/Macros│
└────────┬────────┘       └────────┬────────┘
         │ Syncs                   │ Syncs
         ▼                         ▼
┌────────────────────────────────────────────────────────┐
│     OS Health Store (On-Device, Serverless Hub)        │
│    • Apple HealthKit (iOS) / Health Connect (Android)  │
│    • Also receives Wearable Active & Basal Energy Burn │
└──────────────────────────┬─────────────────────────────┘
                           │ Read via Flutter 'health' plugin
                           ▼
┌────────────────────────────────────────────────────────┐
│            Prevail Adaptive Energy Engine              │
│    1. Aggregates Weekly Energy In (Nutrition)          │
│    2. Aggregates Weekly Energy Out (Active + Basal)    │
│    3. Tracks Actual Δ Body Comp (Fat & Muscle Mass)    │
│    4. Calculates True TDEE                             │
│    5. Outputs Calibrated Nutrition & Activity Targets  │
└────────────────────────────────────────────────────────┘
```

---

## 2. Platform Integrations Breakdown

### 2.1. InBody (Body Composition)
* **Target Metrics:** Body Weight (kg), Skeletal Muscle Mass (kg), Body Fat Percentage (%), Body Fat Mass (kg).
* **Integration Path:** InBody automatically syncs weight and body fat percentage into Apple Health and Google Health Connect. Prevail reads these records with zero cloud API keys needed.
* **Fallback / Offline:** Manual entry screen for InBody test printouts if OS sync is disabled.

### 2.2. MyFitnessPal (Energy In)
* **Target Metrics:** Daily Total Calories In, Protein (g), Carbohydrates (g), Fats (g).
* **Integration Path:** MyFitnessPal syncs full dietary energy and macronutrient breakdowns to HealthKit / Health Connect. Prevail aggregates daily intake over rolling 7-day and 14-day averages.

### 2.3. Activity Trackers & Wearables (Energy Out)
* **Target Metrics:** Active Energy Burned (kcal), Basal / Resting Energy Burned (kcal).
* **Integration Path:** Smartwatches (Apple Watch, Garmin, Pixel Watch, Whoop, etc.) write total and active calories into the native OS health store.

### 2.4. Kilo Gym Platform
* **Purpose:** Sync gym daily workouts, assessment history, and workout scores.
* **Integration Path Analysis:**
  * Kilo currently does not offer a public consumer API.
  * *Option A (Initial Phase 3):* CSV export parser allowing members to upload their Kilo training history.
  * *Option B (Advanced):* Client-side session scraper / member credential login if Kilo member portal allows HTTPS session access.

---

## 3. The Composite Energy Feedback Loop

### 3.1. Theoretical vs. Actual Energy Balance
Most fitness apps estimate Total Daily Energy Expenditure (TDEE) using static formulas (e.g., Mifflin-St Jeor) combined with wearable estimates, which often suffer from 10–25% variance. Prevail uses actual tissue changes to derive **True TDEE**.

1. **Theoretical Energy Deficit/Surplus ($\Delta E_{\text{theoretical}}$):**
   $$\Delta E_{\text{theoretical}} = \text{Energy In (Diet)} - \text{Energy Out (Basal + Active)}$$

2. **Actual Energy Velocity ($\Delta E_{\text{actual}}$):**
   Tissue energy density equivalents:
   * 1 kg Fat Mass $\approx 7,700 \text{ kcal}$
   * 1 kg Skeletal Muscle Mass $\approx 1,800 \text{ kcal}$

   $$\Delta E_{\text{actual}} = (\Delta \text{Fat Mass (kg)} \times 7700) + (\Delta \text{Muscle Mass (kg)} \times 1800)$$

3. **True TDEE Derivation:**
   $$\text{True TDEE} = \text{Daily Energy In} - \frac{\Delta E_{\text{actual}}}{\text{Days in Period}}$$

### 3.2. Target Fine-Tuning Recommendations
Once True TDEE is computed, Prevail compares it against the user's current goal (e.g., Lean Hypertrophy, Maintenance, or Fat Loss):
* **Feedback Recommendation Output:**
  * **Daily Caloric Target Adjustment:** (e.g., *"Your active burn is overestimated by 180 kcal/day. Set MyFitnessPal calorie budget to 2,450 kcal."*)
  * **Activity Burn Adjustment:** (e.g., *"Target 650 active calories/day on non-Strength Week training days."*)

---

## 4. Phase 1 Preparatory Interfaces

To avoid refactoring Phase 1 when Phase 3 is implemented, the domain layer defines the following abstract contracts:

```dart
/// Interface for external health telemetry
abstract class IHealthTelemetryRepository {
  Future<bool> hasPermissions();
  Future<void> requestPermissions();

  Future<List<BodyCompositionSample>> getBodyCompositionSamples({
    required DateTime startDate,
    required DateTime endDate,
  });

  Future<DailyEnergySummary> getDailyEnergySummary({
    required DateTime date,
  });
}

class BodyCompositionSample {
  final DateTime timestamp;
  final double weightKg;
  final double? bodyFatPercentage;
  final double? skeletalMuscleMassKg;

  const BodyCompositionSample({
    required this.timestamp,
    required this.weightKg,
    this.bodyFatPercentage,
    this.skeletalMuscleMassKg,
  });
}

class DailyEnergySummary {
  final DateTime date;
  final double caloriesIn;
  final double activeCaloriesOut;
  final double restingCaloriesOut;
  final double? proteinGrams;

  const DailyEnergySummary({
    required this.date,
    required this.caloriesIn,
    required this.activeCaloriesOut,
    required this.restingCaloriesOut,
    this.proteinGrams,
  });
}
```
These contracts are cleanly decoupled from the `StrengthWeek` domain entities, allowing seamless activation in Phase 3.
