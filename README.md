# Prevail

[![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter&logoColor=white)](https://flutter.dev)
[![Platforms](https://img.shields.io/badge/Platforms-Android%20%7C%20Web%20%7C%20iOS-blue)](#)
[![License: PolyForm Noncommercial](https://img.shields.io/badge/License-PolyForm%20Noncommercial-orange.svg)](LICENSE)

**Prevail** is an independent, unofficial, local-first strength assessment and training progression tracker built with Flutter. It is designed to track personal quarterly **Strength Weeks** across 12 foundational skills against a 20-level standardized milestone progression board.

---

## Key Features

* **Quarterly Strength Week Tracking:** Track quarterly gym assessments with automatic calculation of:
  * **Base Line:** Minimum line achieved across all skills (with inferred carry-forward for in-progress assessments).
  * **Next Line Progress:** Count of skills currently progressing strictly above the assessment's Base Line.
  * **Board Percentage:** Overall completion score out of 240 maximum squares ($12 \text{ skills} \times 20 \text{ levels}$).
* **Bilateral & Unilateral Movement Support:** Dedicated Left & Right tracking for unilateral movements (Get-ups, Carries, One-Arm Pushups, Pistols, Split Squats, Single-Arm Overhead Press). Both sides must meet or exceed the criteria to qualify for the milestone tier.
* **100% Serverless & Zero-Cost:**
  * **Offline-First Guest Mode:** Works immediately on-device without requiring a login or internet connection.
  * **Google Drive AppData Sync:** Syncs encrypted/hidden application data directly into the user's Google Drive `appDataFolder` using client-side OAuth 2.0 PKCE with zero backend hosting or server maintenance fees.
  * **Smart Cloud Merge:** Seamlessly merges local profiles and assessments into cloud storage when an account is connected later.
* **Multi-Profile Support:** Manage multiple athlete profiles under a single Google account, with automatic session recall for the active profile.

---

## 12 Canonical Assessment Skills

Prevail tracks 12 core skills across various load, duration, and bodyweight schemes:

| Skill | Category | Movement Nature | Progression Standard |
| :--- | :--- | :--- | :--- |
| **CRAWL** | Core / Locomotion | Quadrupedal | Timed crawl & bird dog holds |
| **DEAD HANG** | Grip / Shoulder Health | Bilateral | Timed passive & active hangs |
| **DEADLIFT** | Posterior Chain / Hinge | Bilateral | Percentage of body weight $\times$ Reps |
| **GETUP** | Full Body / Stability | **Unilateral (Arm)** | Turkish get-up load (Left & Right) |
| **PISTOL** | Single Leg Squat | **Unilateral (Leg)** | Bodyweight progression $\rightarrow$ Weighted pistol (L & R) |
| **PLANK** | Core Stability | Bilateral | Timed prone forearm plank |
| **PRESS** | Vertical Push | **Unilateral (Arm)** | Single-arm kettlebell press load (L & R) |
| **PULLUP** | Vertical Pull | Bilateral | Hangs $\rightarrow$ Chin-ups $\rightarrow$ Weighted pull-ups |
| **PUSHUP** | Horizontal Push | Bilateral / **Unilateral** | Incline progression $\rightarrow$ Reps $\rightarrow$ One-Arm (L & R) |
| **RFEDSS** | Single Leg Squat | **Unilateral (Leg)** | Rear Foot Elevated Deficit Split Squats (% BW L & R) |
| **SINGLE SIDE CARRY MARCH** | Loaded Carry | **Unilateral (Arm)** | Suitcase carry march (% BW L & R) |
| **SQUAT** | Bilateral Squat | Bilateral | Goblet & front squat load $\times$ Reps |

---

## Architecture & Technology Stack

* **Framework:** [Flutter](https://flutter.dev/) (Dart 3.x) targeting Android, Web (GitHub Pages), and iOS.
* **State Management:** [Riverpod](https://pub.dev/packages/flutter_riverpod) with unidirectional data flow.
* **Local Persistence:** [Drift](https://pub.dev/packages/drift) (SQLite on mobile, Wasm/IndexedDB on web).
* **Cloud Storage:** Google Drive API (`drive.appdata` scope via client-side OAuth 2.0 PKCE).
* **Visualizations:** [fl_chart](https://pub.dev/packages/fl_chart).
* **Routing:** [go_router](https://pub.dev/packages/go_router).

---

## Project Roadmap

* **[Phase 1: Strength Week & Assessment Tracker](docs/phase_1_design.md) (Current Focus):**
  * Full 7-screen assessment workflow, dynamic milestone matching, multi-profile, and Google Drive AppData sync.
* **[Phase 2: Quarterly Patterns & Daily Programs](docs/roadmap.md):**
  * Tracking across 9 daily quarterly programs, muscle group translation, and rep-to-weight normalization.
* **[Phase 3: Health Ecosystem & Feedback Loop](docs/health_integrations.md):**
  * On-device integration with InBody (body composition), MyFitnessPal (nutrition), Google Health Connect / Apple HealthKit (energy burn), Kilo gym platform, and an adaptive True TDEE calibration loop.

---

## Project Documentation

* **[AI Guidance & Coding Rules](AGENTS.md):** Architectural principles, domain terminology rules, and code standards.
* **[Phase 1 Design Specification](docs/phase_1_design.md):** Detailed UI/UX screen specifications, dynamic forms, and calculation formulas.
* **[Architecture Document](docs/architecture.md):** Domain model, cloud sync protocol, and extensibility interfaces.
* **[Technology Stack](docs/stack.md):** Detailed tooling, package dependencies, and serverless authentication details.
* **[Multi-Phase Roadmap](docs/roadmap.md):** Overview of Phases 1 through 3.
* **[Health Integrations Specification](docs/health_integrations.md):** Biometrics, nutrition, and wearable integration specs.
* **[Reference Datasets](docs/resources/):**
  * `docs/resources/milestones.json`: 380 standardized milestone records.
  * `docs/resources/skills_catalog.json`: Structured skills and variants schema with unilateral field mappings.
  * `docs/resources/milestone_zones.json`: 5-zone color tokens and legacy color cross-references.

---

## Disclaimer & Trademark Notice

This software is an **independent, open-source personal project** developed solely for individual tracking and training utility.

* **Non-Affiliation:** This application and its author are **not affiliated with, associated with, authorized by, endorsed by, or in any way officially connected with Prevail Strength and Fitness**, or any of their subsidiaries, branches, or affiliates.
* **Trademarks:** The name "Prevail" as well as related names, marks, emblems, and images are registered trademarks or service marks of their respective owners. The use of any trade name, trademark, or workout terminology in this project is for identification and descriptive reference purposes only and does not imply any affiliation, sponsorship, or endorsement.

---

## License

This project is licensed under the terms of the [PolyForm Noncommercial License 1.0.0](LICENSE).

