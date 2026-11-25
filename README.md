<p align="center">
  <img src="https://img.shields.io/badge/License-MIT-d79921.svg?style=flat-square" alt="License: MIT" />
  <img src="https://img.shields.io/badge/Test%20Cases-144%20(90%20Baseline%20%2B%2054%20Public)-98971a.svg?style=flat-square" alt="Test Cases: 144" />
  <img src="https://img.shields.io/badge/Target-iPhone%20Duo%20%2F%20iOS%2027.2-458588.svg?style=flat-square" alt="Target: iPhone Duo / iOS 27.2" />
  <img src="https://img.shields.io/badge/Xcode-27.2%20%2B%20serve--sim-b16286.svg?style=flat-square" alt="Xcode 27.2 + serve-sim" />
</p>

<h1 align="center">iPhone Duo Test</h1>

<p align="center">
  Public compatibility test suite, posture matrix, and Xcode 27.2 + Codex simulator automation playbook for Apple iPhone Duo foldable adaptation.
</p>

<p align="center">
  <a href="./README.zh-CN.md">中文</a> ·
  <a href="https://xsoway.github.io/iphone-duo-test/">Project Homepage</a> ·
  <a href="https://github.com/xsoway/iphone-duo-test/discussions">Discussions</a>
</p>

<p align="center">
  144 Structured Cases · Cover / Unfolded / Half-Fold / Split View · Simulator Streaming & Evidence Capture
</p>

<hr>

## Table of Contents

- [What is it](#what-is-it)
- [Why](#why)
- [Core Concepts](#core-concepts)
- [Repository Structure](#repository-structure)
- [Quick Start](#quick-start)
- [Test Matrix & Coverage](#test-matrix--coverage)
- [Evidence & Diagnostics](#evidence--diagnostics)
- [Design Principles](#design-principles)
- [FAQ](#faq)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Maintainer](#maintainer)

---

## What is it

**iPhone Duo Test** is an open-source QA knowledge base and public test suite designed for adapting iOS and hybrid (Safari / `WKWebView`) applications to **iPhone Duo** foldable devices.

It combines two practical assets:

1. **144 Public Compatibility Test Cases** (Markdown + 5-sheet Excel workbook): 90 general baseline cases (`M1`–`M7`) and 54 newly authored Duo-specific public cases (`PUB-001`–`PUB-054`).
2. **Simulator Automation & Evidence Playbook**: Step-by-step instructions for running `.app` simulator builds on **Xcode 27.2 / iOS 27.2 Runtime**, streaming device screens to a browser via `serve-sim@0.1.49`, driving fold/rotation transitions (`0°`, `130°`, `180°`), and capturing display-specific screenshots and `SpringBoard` logs.

---

## Why

On a standard slab iPhone, navigation from a list to a detail page and back occurs on a single unchanging physical display. On iPhone Duo, users can fold, half-open, unfold, or enter system Split View **mid-task**—while a keyboard is open, a modal is presented, or an upload/request is in flight.

| Dimension | Standard iPhone Baseline | iPhone Duo Variables | What to Verify |
| --- | --- | --- | --- |
| Physical Display | Single main screen | Closed cover screen (`A`) vs unfolded inner screen (`B`) | Object ID, route stack, unsaved input, scroll anchor, primary CTA |
| Aspect Ratio & Safe Area | Notch / Dynamic Island / home indicator | Short-and-wide outer screen; side system controls; asymmetric safe areas | Test both left/right orientations; verify hit targets |
| In-App Layout | Adapts to current window width | Single-stack may switch to list-detail dual-pane or stay single-column | Default selection, back-stack consistency, pane data sync |
| Half-Fold Posture | No hinge reserved area | Folding reserved zone & segmented viewport (`C`) | Text, required fields, close buttons, focus order away from hinge |
| System Multitasking | Single-app phone window | System Split View (`D`) side-by-side with another app | Swap sides, active focus, keyboard ownership, exit recovery |
| Camera & Input | Fixed front camera environment | Camera position changes across cover/inner displays | Viewfinder, callbacks, permission prompts during fold transitions |

---

## Core Concepts

To avoid misclassifying layout bugs, test reports separate four distinct concepts:

- **Outer vs. Inner Screen (`A` / `B`)**: Two separate physical display surfaces.
- **In-App Dual-Pane**: One app displaying list and detail side-by-side in an expanded window.
- **System Split View (`D`)**: Two different apps sharing the inner screen side-by-side, shrinking each app's available window.
- **Half-Fold & Reserved Hinge Area (`C`)**: Viewport segmentation and non-interactive hinge zones when partially folded (e.g., `130°`).

---

## Repository Structure

```text
iphone-duo-test/
├── README.md                                  # English documentation (this file)
├── README.zh-CN.md                            # Simplified Chinese documentation
├── index.html                                 # Interactive project homepage (gruvbox-material, EN/ZH)
├── LICENSE                                    # MIT License (Copyright 2026 Alan Hsu)
├── RELEASE_CHECKLIST.md                       # Open-source release verification checklist
├── .gitignore                                 # Ignores OS files, logs, evidence artifacts, secrets
├── .gitattributes                             # Enforces LF line endings and binary handling for .xlsx
├── docs/
│   ├── iphone-duo-public-test-cases.md        # 144 public test cases (90 baseline + 54 PUB-001..054)
│   └── xcodex-simulator-automation-guide.md   # Xcode 27.2 + Codex + serve-sim automation playbook
└── data/
    └── iphone-duo-public-test-cases.xlsx      # 5-sheet Excel workbook for QA tracking & execution
```

| File | Responsibility |
| --- | --- |
| [docs/iphone-duo-public-test-cases.md](./docs/iphone-duo-public-test-cases.md) | Full specification of execution matrix (`A`, `B`, `C`, `D`, `A↔B`),判定 criteria, 54 `PUB-*` cases, and 90 general baseline cases. |
| [data/iphone-duo-public-test-cases.xlsx](./data/iphone-duo-public-test-cases.xlsx) | Structured 5-sheet workbook (`用例总览`, `姿态矩阵`, `通用基线`, `参考资料`, `新增公共用例`). |
| [docs/xcodex-simulator-automation-guide.md](./docs/xcodex-simulator-automation-guide.md) | End-to-end guide for `.app` verification, Device Hub setup, Codex `ios-simulator-browser` integration, `serve-sim` posture commands, and evidence triage. |
| [index.html](./index.html) | Standalone single-file project homepage with live clock, animated testing pipeline, expandable repo tree, and interactive command simulator. |

---

## Quick Start

### Prerequisites

- **macOS** with **Xcode 27.2** (or 27.1+) and **iOS 27.2 Simulator Runtime** installed via **Xcode → Settings → Components**.
- **Node.js / npx** (for `serve-sim@0.1.49`).
- Optional: **Codex** with the `build-ios-apps@openai-curated` plugin (`ios-simulator-browser` skill).

### 1. Verify Toolchain & Simulator

```bash
xcodebuild -version
xcrun simctl list runtimes
xcrun simctl list devices available
```

### 2. Inspect Simulator `.app` Binary & Launch

Verify the binary targets `IOSSIMULATOR` (not macOS `arm64`), then boot, install, and launch on your iPhone Duo simulator UDID:

```bash
APP="/absolute/path/to/TestApp.app"
BUNDLE=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleIdentifier' "$APP/Info.plist")
EXE=$(/usr/libexec/PlistBuddy -c 'Print :CFBundleExecutable' "$APP/Info.plist")
xcrun vtool -show-build "$APP/$EXE"

SIM="<iPhone Duo UDID>"
xcrun simctl bootstatus "$SIM" -b
xcrun simctl install "$SIM" "$APP"
xcrun simctl launch "$SIM" "$BUNDLE"
```

### 3. Stream to Browser & Drive Fold / Rotation Transitions

Start `serve-sim@0.1.49` on localhost and drive posture transitions during active flows:

```bash
# Start browser mirror (keep running in foreground)
npx --yes serve-sim@0.1.49 --host 127.0.0.1 --panes tools --fit "$SIM"

# Drive fold angles and orientation
npx --yes serve-sim@0.1.49 fold 0 -d "$SIM"
npx --yes serve-sim@0.1.49 fold 130 -d "$SIM"
npx --yes serve-sim@0.1.49 fold 180 -d "$SIM"
npx --yes serve-sim@0.1.49 rotate landscape_left -d "$SIM"
npx --yes serve-sim@0.1.49 rotate portrait -d "$SIM"

# Clean up mirror session when finished
npx --yes serve-sim@0.1.49 --kill "$SIM"
```

---

## Test Matrix & Coverage

### Execution Matrix

| Marker | Environment / Posture | Minimum Coverage |
| --- | --- | --- |
| `A` | Closed cover screen (single screen) | Cold launch, narrow safe area, single-stack navigation, keyboard open |
| `B` | Fully unfolded inner screen | Single-column responsive, in-app list/detail dual-pane, galleries & tables |
| `C` | Half-fold inner screen | Book/tabletop posture, hinge reserved zone & interactive hit targets |
| `D` | System Split View | Side-by-side with another app, swap left/right, focus handoff |
| `A↔B` | Bidirectional fold/unfold transition | Both steady-state and mid-action (typing, scrolling, modals, in-flight requests) |
| Control | Standard iPhone & Duo physical device | Same-build, same-data comparison; real APNs, camera, thermals, and distribution |

### Case Breakdown (144 Total)

- **54 New Public Cases (`PUB-001`–`PUB-054`)**:
  - Single Screen & Posture (`PUB-001`–`PUB-007`)
  - Dual-Pane & System Split View (`PUB-008`–`PUB-016`)
  - Controls & Gestures (`PUB-017`–`PUB-025`)
  - H5 & Hybrid Containers (`PUB-026`–`PUB-034`)
  - Upload, Download & Files (`PUB-035`–`PUB-043`)
  - App Regression & Release Acceptance (`PUB-044`–`PUB-054`)
- **90 General Baseline Cases (`M1`–`M7`)**:
  - Posture & Continuity, Native Display, Controls & Gestures, H5 & Hybrid Pages, Login & Session, App Functional Regression, Push Notifications, Install & Upgrade, Performance & Stability.

---

## Evidence & Diagnostics

When a transition causes a black frame in the browser mirror or an unexpected return to the Home Screen, isolate display-mirror issues from app crashes using simultaneous capture:

```bash
mkdir -p evidence
# Capture active inner display (replace --display ID with your local display ID)
xcrun simctl io "$SIM" screenshot --display=3 evidence/inner.png

# Run serve-sim diagnostics
npx --yes serve-sim@0.1.49 doctor -d "$SIM"

# Inspect SpringBoard process logs for your Bundle ID
xcrun simctl spawn "$SIM" log show --last 10m --style compact \
  --predicate 'process == "SpringBoard" AND eventMessage CONTAINS[c] "com.example.testapp"'

# Verify file integrity across fold transitions and test push payloads
shasum -a 256 evidence/source.pdf evidence/downloaded.pdf
xcrun simctl push "$SIM" "$BUNDLE" evidence/test-notification.apns
```

---

## Design Principles

1. **Control Before Matrix**: Always run the primary flow on a standard iPhone with the same build and test data first so universal bugs are separated from Duo-specific posture bugs.
2. **Mid-Action Transitions**: Folding on an idle screen only tests static reflow. Real regressions surface when folding with the keyboard open, a modal active, or competing async requests (`A` slower than `B`).
3. **Dual Assertions on High-Risk Pages**: Assert both **UI availability** (controls visible and clickable) and **state integrity** (same object ID, no duplicate submission).
4. **Evidence Isolation**: Tag screenshots, logs, and server records with the exact case ID (`PUB-xxx`) and never mix evidence across runs.

---

## FAQ

- **Q: Can the simulator replace physical iPhone Duo testing?**
  - **A**: No. The simulator covers layout, navigation, H5 viewport, and interaction regressions. Real APNs delivery, app store/channel install-upgrade, physical camera hardware, thermals, battery, and release performance must be verified on physical Duo devices.
- **Q: Why does Safari 27.1 posture/viewport segmentation require feature flags?**
  - **A**: Per WebKit's Safari 27.1 release notes, posture and viewport segment APIs are preview features behind feature flags. Run business regression under default settings first, then validate experimental APIs explicitly.
- **Q: Why did `serve-sim` show a black frame after folding?**
  - **A**: Capture both Device Hub and `xcrun simctl io "$SIM" screenshot --display=<id>` at the same moment. If both local captures are normal, it is a mirror stream issue rather than an app render bug.

---

## Roadmap

- [x] Author 144 structured public test cases (90 baseline + 54 Duo-specific `PUB-001`–`PUB-054`)
- [x] Provide 5-sheet Excel execution workbook (`data/iphone-duo-public-test-cases.xlsx`)
- [x] Document Xcode 27.2 + `serve-sim@0.1.49` + Codex automation workflow
- [x] Publish bilingual documentation (`README.md` / `README.zh-CN.md`) and interactive homepage (`index.html`)
- [ ] Add reusable shell/Python helper scripts for automated evidence bundling (`evidence/` + SHA-256)
- [ ] Add sample `.apns` payload templates and minimal WKWebView H5 test harness

---

## Contributing

Contributions, case refinements, and environment notes are welcome!

- Have a question or want to propose a new test scenario? Open a topic in [GitHub Discussions](https://github.com/xsoway/iphone-duo-test/discussions).
- Found a typo or want to add a reproducible automation snippet? Submit a Pull Request keeping `README.md` and `README.zh-CN.md` in sync.

---

## License

This project is licensed under the [MIT License](./LICENSE).

---

## Maintainer

- **Alan Hsu** ([@xsoway](https://github.com/xsoway))
