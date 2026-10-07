# Technology Stack & Tooling

This document outlines the technology stack, library choices, architecture patterns, and build strategy for the **Prevail** cross-platform application.

---

## 1. Core Framework & Language

* **Framework:** [Flutter](https://flutter.dev/) (Channel: Stable)
* **Language:** Dart 3.x
* **Target Platforms:**
  * **Android:** Primary target for mobile installation (local APK / internal distribution).
  * **Web:** Static WebAssembly/CanvasKit build deployed to **GitHub Pages** for instant access on desktop and iPhone without Apple Developer fees.
  * **iOS:** Codebase kept 100% iOS-ready for Xcode direct installation / TestFlight when desired.

---

## 2. Architectural Layers & Dependencies

```
┌────────────────────────────────────────────────────────┐
│                   Presentation Layer                   │
│   Flutter Material 3 • go_router • fl_chart • Riverpod │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│                      Domain Layer                      │
│     Pure Dart Entities • Use Cases • Calculations      │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│                       Data Layer                       │
│    Local Cache (Drift SQLite) • Drive AppData API      │
└────────────────────────────────────────────────────────┘
```

### Key Libraries & Packages:

| Domain | Package | Purpose |
| :--- | :--- | :--- |
| **State Management** | `flutter_riverpod` (v2.x) | Unidirectional data flow, reactive caching, testability, dependency injection. |
| **Routing & Nav** | `go_router` | Declarative routing with URL support for GitHub Pages and strict parent back-navigation. |
| **Local Persistence** | `drift` + `drift_flutter` | Relational local-first SQLite database. Drift compiles to native SQLite on Android/iOS and Wasm/IndexedDB on Web. |
| **Cloud Storage** | `googleapis` (Drive v3) | Interacting with the Google Drive `drive.appdata` folder. |
| **Authentication** | `google_sign_in` | Client-side OAuth 2.0 PKCE authentication. |
| **Charting** | `fl_chart` | Progression curves, milestone charts, and assessment radar/bar metrics. |
| **Date & Formatting** | `intl` | Parsing and formatting assessment week dates (Monday keys, quarter identifiers). |
| **Serialization** | `json_annotation` + `json_serializable` | Fast, type-safe JSON serialization for bundled resources and cloud syncing. |

---

## 3. Serverless & Zero-Cost Architecture

### Why No Backend Server Is Needed:
* **Direct Client OAuth (PKCE):** Standard Google OAuth 2.0 with PKCE allows mobile and single-page web applications to obtain user access tokens directly from Google's auth servers without a client secret or an intermediary backend server.
* **Google Cloud Console Configuration:**
  * A single free Google Cloud project is configured with the OAuth consent screen.
  * Three client credentials are created under this project:
    1. **Android Client ID:** Tied to the app's package name and SHA-1 signing fingerprint.
    2. **iOS Client ID:** Tied to the iOS bundle identifier.
    3. **Web Client ID:** Authorized for `localhost` (development) and `https://<username>.github.io/<repo>/` (GitHub Pages production).
* **Storage Isolation (`drive.appdata`):**
  * The `https://www.googleapis.com/auth/drive.appdata` scope grants access to the user's hidden Application Data folder in their Google Drive account.
  * The user incurs zero hosting fees, and the developer incurs zero infrastructure, server, or database maintenance costs.

> [!NOTE]
> While a small auth server on Google Cloud (e.g. Cloud Run) could be hosted using Gemini Pro credits, a purely client-side approach is significantly simpler, has zero points of server failure, and requires zero monthly monitoring.

---

## 4. UI / UX Design System

* **Design Standard:** Material Design 3 (M3) with custom high-contrast, gym-ready styling.
* **Theme Modes:** Dark Mode by default (optimized for gym environments and OLED screens) with Light Mode support.
* **Milestone Color Hierarchy:**
  The 20 progression levels feature specific visual accent colors corresponding to the gym's standard tiers:
  * White, Blue, Yellow, Green, Red, Light Blue, Orange, Bright Pink, Bright Green, Purple, Gold, Pink, Eggshell, Maroon, Grey, Brown, Silver, Burnt Orange, Light Purple, Black.
* **Form Factor Responsiveness:**
  * Handset view: Full-bleed single-column layout with fixed app bar and bottom actions.
  * Tablet / Web view: Max-width constrained content containers (e.g. 720px–900px centered canvas) to prevent stretched, empty web pages while retaining app-like ergonomics.

---

## 5. Build & Deployment Pipeline

* **Repository:** Hosted on a public GitHub repository.
* **Web Deployment Automation:**
  * GitHub Actions workflow triggers on push to `main`.
  * Runs tests, builds Flutter Web in release mode:
    ```bash
    flutter build web --release --base-href "/prevail/"
    ```
  * Automatically deploys build output (`build/web`) to the `gh-pages` branch.
* **Mobile Builds:**
  * Automated GitHub Action artifact generation for Android APKs (`app-release.apk`) available for direct download on phones.
