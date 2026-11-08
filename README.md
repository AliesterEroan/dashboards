# Dashboards — PLC/HMI Web Interfaces

> Static HTML HMI pages designed for a Siemens PLC web server (TIA Portal / WinCC-style AWP tags): a conveyor + RFID control dashboard and earlier mockups — front-end only, no backend.

These are **front-end mockups/templates for industrial HMI screens**. The "Web Interface" version is wired to a PLC: it polls a `data.html` JSON endpoint every 400 ms and POSTs forms whose field names are PLC tag names. The "Testing Web" versions are earlier static iterations with hardcoded values.

---

## 📜 Table of Contents

- [✨ Features](#-features)
- [🚀 Installation](#-installation)
- [📖 Usage](#-usage)
- [🏗️ Architecture](#-architecture)
- [🛠️ Technologies Used](#-technologies-used)
- [⚠️ Limitations](#-limitations)
- [📜 License](#-license)
- [📧 Contact](#-contact)

---

## ✨ Features

### `Web Interface/dashboard.html` — PLC-connected HMI (main file)
- **Three screens** — welcome menu → Conveyor terminal and RFID station, with session-persisted screen restore.
- **Live PLC polling** — `updateRealTimeData()` fetches `data.html` every 400 ms and maps JSON fields to the UI: `Marche`, `km1`, `km2`, `b1`, `b2` (status dots), `Courant_ScaleX`, `Tension_ScaleX`, `input_compteur`, `temps_output`, and `RFID_*` flags.
- **Command writes** — POST forms carry PLC tag names directly (`Web_CMD_StartMarche`, `Web_CMD_StopMarche`, `Web_CMD_Reset`, `Temps input`, `Pos`, `Vitesse`, `RFID_commandBteOut`, `RFID_Write_Data 1`, `RFID_Write_Data 2`-style fields), matching Siemens AWP web-variable conventions; `Testing 1.html` even carries `AWP_In_Variable` comments.
- **i18n** — FR/EN/AR dictionary with full RTL layout switching for Arabic (`dir="rtl"`, `html[lang]`).
- **4 themes** — dark, light, "tunisia" (red), "lebanon" (green), persisted in `localStorage`.
- **Visual polish** — glassmorphism panels, animated conveyor belt/wheels (spin tied to the `Marche` bit via `data-motor-run`), neon status indicators.

### `Testing Web/` — earlier iterations
- **`dashboard.html`** — simpler glassmorphism version of the same conveyor/RFID layout with **hardcoded values** (`142` pieces, `02.45 A`, `231.2 V`) and `console.log` stubs for PLC commands — a static mockup.
- **`Testing 1.html`** — intermediate variant of the themed dashboard (same theme/i18n system, `AWP_In_Variable` comments at the top).

### `Web Interface/Data.html`
- Empty placeholder file — intended to be **generated/served by the PLC** as the JSON telemetry endpoint the dashboard polls.

---

## 🚀 Installation

### Prerequisites
- A web server capable of serving static files (for local preview)
- For real use: a Siemens PLC/HMI web server (TIA Portal WinCC / industrial PC) that produces `data.html` JSON and accepts POSTed AWP variables — or adapt the fetch/POST targets

### Steps
1. Open `Web Interface/dashboard.html` directly in a browser for a UI-only preview (telemetry fetch will fail harmlessly — the code catches errors).
2. To integrate with a PLC, host the file where the PLC's web server can serve it alongside a `data.html` JSON producer, and map the form field names to your PLC tags.
3. `Testing Web/` files can be opened standalone; they contain no live data wiring.

---

## 📖 Usage

### Preview the UI
- Open `dashboard.html`, pick a theme (top-right) and language (FR/EN/AR), navigate via the welcome menu.
- In `Web Interface/dashboard.html`, values stay at defaults until `data.html` returns valid JSON; button clicks POST the tag-name forms to the current URL.

### Adapt to a PLC
- Rename form `name` attributes to your PLC's AWP variable names; adjust the JSON keys parsed in `updateRealTimeData()` to match your `data.html` schema; change the 400 ms poll interval if needed.

---

## 🏗️ Architecture

```
dashboards/
├── Web Interface/
│   ├── dashboard.html   # main HMI: conveyor + RFID, i18n, themes, PLC polling
│   └── Data.html        # empty placeholder for PLC-generated JSON telemetry
└── Testing Web/
    ├── dashboard.html   # earlier static mockup (hardcoded values, console.log stubs)
    └── Testing 1.html   # intermediate themed variant (AWP_In_Variable comments)
```

### Data Flow (`Web Interface/dashboard.html`)
1. Every 400 ms: `fetch('data.html?ts=…')` → `JSON.parse` → update status dots, values, RFID badges, and the conveyor animation state.
2. User clicks a command button → form POST (intercepted by JS, sent via `fetch`) → PLC web server writes the named variable → next poll reflects the change.
3. Theme/language/screen choices persist in `localStorage`/`sessionStorage`.

---

## 🛠️ Technologies Used

### Core
- **HTML5 + vanilla CSS** - glassmorphism layout, CSS custom properties for 4 themes
- **Vanilla JavaScript** - screen router, i18n dictionary, `fetch` polling/POSTing, `localStorage`/`sessionStorage`

### PLC integration conventions
- **Siemens AWP / WinCC web tags** - form field names and `AWP_In_Variable` comments follow TIA Portal web-variable naming
- **JSON over HTTP** - `data.html` as the PLC→browser telemetry channel

---

## ⚠️ Limitations

### Completeness
- **Front-end only** — there is no backend, no real `data.html` producer, and no PLC in the repo; all live values depend on an external Siemens web server.
- **`Data.html` is empty** — out of the box the dashboard shows default/zero values and logs "PLC Unreachable" warnings.
- **Duplicate/iterative files** — `Testing Web/` contains superseded versions of the same dashboard; `Web Interface/dashboard.html` is the most complete.
- **Hardcoded tag schema** — the JSON keys (`Marche`, `km1`, `Courant_ScaleX`, `RFID_*`, etc.) are specific to one PLC program; retargeting requires code edits.

### Technical
- **400 ms polling** — fixed-interval `fetch`; no backoff, no WebSocket, and each poll bypasses cache via a timestamp query.
- **POST target is `window.location.href`** — commands POST back to the page URL; correct only if the PLC web server handles form posts on that endpoint.
- **No authentication or input validation** — anyone who can reach the page can send commands.
- **RTL/i18n is string-level** — layout uses logical properties in key places but was designed primarily for FR/EN/AR labels, not full localization of every string.

### Testing Limitations
- **What a clone can re-check**: all HTML/JS/CSS renders in any browser; theme/language switching and screen navigation work standalone.
- **What a clone cannot re-check**: actual PLC data binding, tag names, and command behavior — these require the original Siemens PLC/HMI runtime, which is not in the repo.

---

## 📜 License

Dashboards is proprietary software — see [LICENSE](LICENSE) for the full terms.

**Copyright © 2026 AliesterEroan. All rights reserved.**

This software is provided for personal, non-commercial use only. Commercial use, redistribution, modification, and sublicensing are prohibited without explicit written permission. The copyright holder may distribute copies through the project's official GitHub Releases page; third parties may not host, mirror, or redistribute this software.

For licensing inquiries, open an issue at <https://github.com/AliesterEroan/dashboards/issues>.

---

## 📧 Contact

- **GitHub**: [AliesterEroan/dashboards](https://github.com/AliesterEroan/dashboards)
- **Issues**: [Create an issue](https://github.com/AliesterEroan/dashboards/issues)

---

*Built with ❤️ by AliesterEroan*
