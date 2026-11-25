# Release Checklist — iphone-duo-test

Evidence-backed open-source release checklist following `oss-release-prep`.

## A. Repository State

- [x] Git initialized on `main` branch (`git init -b main`) — verified before push.
- [x] Remote tracking configured (`origin` -> `https://github.com/xsoway/iphone-duo-test`).
- [x] No tracked `.env`, credentials, `.DS_Store`, or build artifacts.

## B. Required Files

- [x] `README.md` (English, full version) — linked to `./README.zh-CN.md`.
- [x] `README.zh-CN.md` (Simplified Chinese, mirrored structure) — linked to `./README.md`.
- [x] `LICENSE` (MIT License, Copyright (c) 2026 Alan Hsu).
- [x] `.gitignore` (excludes `.DS_Store`, `.env*`, `evidence/`, `*.log`, `*.png`, `*.apns`, `__pycache__/`).
- [x] `.gitattributes` (enforces LF line endings and marks `*.xlsx` as binary).
- [x] Normalized directories: `docs/` and `data/`.

## C. README Quality

- [x] Includes centered badges, positioning, language switcher, TOC, What/Why, Core Concepts, Structure, Quick Start, Matrix & Coverage, Evidence & Diagnostics, Principles, FAQ, Roadmap, Contributing, License, Maintainer.
- [x] All commands (`xcodebuild`, `xcrun simctl`, `xcrun vtool`, `serve-sim@0.1.49`, `shasum`) and counts (`144` = `90` baseline + `54` `PUB-*`, `5` XLSX sheets) are grounded in repository files.
- [x] `README.md` and `README.zh-CN.md` mirror each other section by section.

## D. Sensitive Information Scan (Hard Block)

- [x] Zero API keys, tokens, cookies, private keys, passwords, personal emails/phones, or local absolute home paths across all `.md`, `.html`, `.xlsx` XMLs, and repository config files.
- [x] Verified `.xlsx` metadata (`docProps/core.xml` creator = `openpyxl`, `docProps/app.xml` = `Openpyxl 3.1.5`).
- [x] Verified placeholder paths/IDs only (`/absolute/path/to/TestApp.app`, `<iPhone Duo UDID>`, `com.example.testapp`).

## E. Verifiability

- [x] Repository structure, markdown links, and XLSX integrity verified locally.
- [ ] Note: `xcrun simctl` / `serve-sim@0.1.49` commands target macOS + Xcode 27.2 + iOS 27.2 Runtime and were not executed in this documentation release prep run.

## F–I. Release, Discoverability & Homepage

- [x] `index.html` generated following gruvbox-material dark-gold theme with EN/ZH toggle, typing hero terminal, live clock & schedule, animated testing pipeline, expandable repo tree, interactive command chips, and Tweaks panel.
- [x] Headless browser self-test passed via Playwright (zero console errors, EN/ZH toggle, tree expand/collapse, clock, pipeline, command chips, tweaks, and 380px mobile responsiveness verified).
- [x] GitHub repo created, About/topics configured, Discussions enabled, and pushed to `main`.
