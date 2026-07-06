# Sleep Tracker — GPT-5.5 → Kotlin Multiplatform (KMP) Migration

> Repo 1/7 from my M.Sc. thesis, *"Who Moved My Button?"* (McMaster, 2026) — GPT-5.5's migration to Kotlin Multiplatform, tied 4th of the six. Other repos + baseline linked below.

Compose Multiplatform (Android) rewrite of the original React Native "Sleep Tracker" privacy-transparency app, produced by **GPT-5.5** under a shared 15-rule migration prompt I wrote. It talks to the same Node.js/Express backend as the original app (see [llm-migration-sleeptracker-baseline](https://github.com/MelvinMo/llm-migration-sleeptracker-baseline)).

---

## Usability findings (from my thesis)

This migration was evaluated with Nielsen's ten usability heuristics across six standardized tasks by a single assessor (severity 0–4, lower is better). Full detail is in **Chapter 5** of my thesis (App 1).

| Metric | Value |
|---|---|
| Aggregate severity total | **20** |
| vs. baseline (React Native, total 16) | **+4** |
| Rank among all 7 implementations | **Tied 4th of 7** (tied with the GPT-5.5 Flutter migration) |

The entire application logic lives in a single 2,516-line `App.kt` file rather than being split into conventional `commonMain/models/`, `commonMain/viewmodels/`, and `commonMain/data/` packages — the same monolithic-file pattern GPT-5.5 produced for its Flutter migration, suggesting a consistent generation strategy regardless of target framework. The privacy tooltip renders inside a fixed-height `Popup` with no scroll modifier, so content that exceeds the available height is clipped, and — unlike the GPT-5.5 Flutter migration — it has no tappable links to the privacy policy, PIPEDA regulation, or opt-out preferences. See the thesis for the full per-heuristic breakdown and screenshots.

A more detailed file-by-file and route-by-route mapping produced during migration is in [KMP_MIGRATION_AUDIT.md](KMP_MIGRATION_AUDIT.md).

---

## Related repositories

| Repo | Description |
|---|---|
| [llm-migration-sleeptracker-baseline](https://github.com/MelvinMo/llm-migration-sleeptracker-baseline) | Original React Native app (unmodified snapshot) |
| [llm-migration-sleeptracker-sonnet46-kmp](https://github.com/MelvinMo/llm-migration-sleeptracker-sonnet46-kmp) | Claude Sonnet 4.6 → KMP |
| [llm-migration-sleeptracker-sonnet46-flutter](https://github.com/MelvinMo/llm-migration-sleeptracker-sonnet46-flutter) | Claude Sonnet 4.6 → Flutter |
| [llm-migration-sleeptracker-sonnet46-maui](https://github.com/MelvinMo/llm-migration-sleeptracker-sonnet46-maui) | Claude Sonnet 4.6 → .NET MAUI |
| **llm-migration-sleeptracker-gpt55-kmp** | **This repo** — GPT-5.5 → KMP |
| [llm-migration-sleeptracker-gpt55-flutter](https://github.com/MelvinMo/llm-migration-sleeptracker-gpt55-flutter) | GPT-5.5 → Flutter |
| [llm-migration-sleeptracker-gpt55-maui](https://github.com/MelvinMo/llm-migration-sleeptracker-gpt55-maui) | GPT-5.5 → .NET MAUI |

---

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Android Studio | Hedgehog (2023.1) or later | https://developer.android.com/studio |
| JDK | Android Studio's bundled JBR (OpenJDK 21) — do not use a system JDK | — |
| Android SDK | API 35 (compile), API 26+ (run) | SDK Manager inside Android Studio |

---

## 1. Open the project

1. Launch Android Studio.
2. **File → Open** → select this repository's folder.
3. Wait for Gradle sync to finish (the first sync downloads ~500 MB of dependencies — this is normal).
4. If prompted about a missing JDK, point it to the JDK bundled with Android Studio.

---

## 2. Point the app at a backend

The backend URL is passed as a Gradle property at build time (see `composeApp/build.gradle.kts`). Copy `.env.example` for reference values:

```bash
cp .env.example .env
```

Build with your own backend URL:
```bash
# Windows:
./gradlew.bat :composeApp:assembleDebug --console=plain -PapiUnencryptedUrl=http://<your-computer's-LAN-IP>:7000/api

# Mac/Linux:
./gradlew :composeApp:assembleDebug --console=plain -PapiUnencryptedUrl=http://<your-computer's-LAN-IP>:7000/api
```

If you omit `-PapiUnencryptedUrl`, the build falls back to a placeholder value that will not resolve — you must supply your own. To run the backend locally, see [llm-migration-sleeptracker-baseline](https://github.com/MelvinMo/llm-migration-sleeptracker-baseline).

The debug APK is produced at:
```
composeApp/build/outputs/apk/debug/composeApp-debug.apk
```

---

## 3. Connect a device

**USB:**
1. Enable Developer Options (Settings → About Phone → tap Build Number 7 times).
2. Enable **USB Debugging** in Developer Options.
3. Connect via USB and verify:
```bash
adb devices
```

**Wireless (Android 11+):**
1. On the device: **Developer Options → Wireless Debugging** → enable → **Pair device with pairing code**.
2. Pair once from your computer:
```bash
adb pair <ip>:<pairing-port>
```
3. Connect (note: a different port than pairing):
```bash
adb connect <ip>:<port>
adb devices
```

---

## 4. Install and run

```bash
adb install -r composeApp/build/outputs/apk/debug/composeApp-debug.apk
adb shell am start -n com.mcscert.sleeptracker.kmpdev/com.mcscert.sleeptracker.MainActivity
```

Or, from Android Studio: select the connected device from the dropdown and click **Run**.

**Reinstalling cleanly** (e.g. after a code change) — if a plain reinstall fails, force-stop and uninstall first:
```bash
adb shell am force-stop com.mcscert.sleeptracker.kmpdev
adb uninstall com.mcscert.sleeptracker.kmpdev
adb install -r -d composeApp/build/outputs/apk/debug/composeApp-debug.apk
```

**Checking it's running / debugging a crash:**
```bash
adb shell pidof com.mcscert.sleeptracker.kmpdev   # prints a PID if the process is alive
adb logcat -d -t 300 | grep -E "sleeptracker|FATAL EXCEPTION|AndroidRuntime"
```

---

## Project structure

```
├── composeApp/src/
│   ├── commonMain/kotlin/com/mcscert/sleeptracker/
│   │   └── App.kt              # Shared UI, ViewModels, navigation, repositories (single file)
│   └── androidMain/kotlin/com/mcscert/sleeptracker/
│       └── MainActivity.kt     # Android entry point
├── composeApp/build.gradle.kts # Backend URL Gradle properties (see Section 2)
├── KMP_MIGRATION_AUDIT.md      # Route/component mapping notes from the migration
├── .env.example                # Backend URL reference values (see Section 2)
└── .gitignore
```

---

## Known limitations (from my thesis)

- Application logic is concentrated in a single 2,516-line file rather than split across conventional KMP source-set packages.
- The privacy tooltip has no scroll support and clips content that exceeds its fixed height.
- The privacy tooltip omits tappable links to the privacy policy, PIPEDA regulation, and opt-out preferences (present in the GPT-5.5 Flutter migration).
- See Chapter 5 of my thesis for the full task-by-task and heuristic-by-heuristic severity breakdown, including the two other GPT-5.5 migrations (Flutter, MAUI).

---

## Environment variables & secrets

This repository contains **no real credentials**. `.env.example` holds placeholder/reference values only (a backend URL, not a secret). If you deploy your own backend, keep its real `.env` (JWT secret, database/Firebase keys, LLM API keys) out of version control — it's already covered by `.gitignore`.

---

## Citing my thesis

If you're referencing this repo, here's the full citation:

> Mokhtari, M. (2026). *"Who Moved My Button?": A Usability Evaluation of LLM-Assisted Cross-Platform Migration* [Master's thesis, McMaster University]. Department of Computing and Software. Supervisor: Richard F. Paige.

---

## License

MIT — see [LICENSE](LICENSE). Use it, fork it, build on it.
