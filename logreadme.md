# LOGWATCH — Log File Intrusion Detection Simulator

**File:** `logfile.html`
**Type:** single-file, static HTML/CSS/JS app — no build step, no backend, no dependencies except two Google Fonts and a system font stack.
**Deploy target:** GitHub Pages (or any static host — S3, Netlify, a plain folder opened in a browser).

---

## What it does

LOGWATCH is a self-contained console that simulates a SOC-style intrusion detection workflow entirely in the browser. It parses SSH/auth-log lines, runs them through a configurable rule engine and a set of statistical detectors, and surfaces the results as a live alert dashboard — all without a server, a database, or any data leaving the user's machine.

Concretely, it lets you:

- **Feed it log data** three ways: paste raw text, upload a `.log`/`.txt` file, or generate synthetic SSH traffic (random background noise, a single attack burst, or a full pre-built "sample attack" scenario).
- **Watch a simulated live tail**, where new log lines are appended on an interval, standing in for `tail -f` / `inotify` on a growing file (see [Note on the inotify stretch goal](#note-on-the-inotify-stretch-goal) — a real filesystem watch isn't reachable from a static web page).
- **Detect intrusions** using a regex-based rule engine (built-in + user-defined rules) plus three statistical detectors: brute-force login attempts, IP traffic spikes, and off-hours logins — each with adjustable thresholds.
- **Review results** on a dashboard: summary stat cards, a top-offending-IP chart, a filterable/sortable alerts table, and a "threat pulse" waveform that spikes visually every time something is detected.
- **Export findings** as JSON or CSV for use elsewhere.

---

## Quick start

1. Open `logfile.html` in any modern browser (or deploy it to GitHub Pages — see below).
2. Click **"Load sample attack log"** in the Log source panel to see the whole pipeline work end-to-end immediately, no setup required.
3. Watch the log tail (center column) populate and the Alerts panel (right column) fill in as brute-force, compromise, spike, and off-hours events are detected.

### Deploying to GitHub Pages

1. Add `logfile.html` to a GitHub repo (rename it to `index.html` if you want it to be the site root, or keep the name and link to it directly, e.g. `https://username.github.io/repo/logfile.html`).
2. In the repo settings, enable **Pages** and point it at the branch/folder containing the file.
3. No build step, no `package.json`, no server config needed — it's a static file.

---

## Layout / workflow

The app is organized into three columns that mirror a real monitoring workflow: **configure → watch → respond**.

### 1. Log source & live monitoring (left column)

| Control | What it does |
|---|---|
| Paste box + **Parse pasted text** | Parses whatever raw log lines you paste, one per line |
| **Upload file** | Reads a local `.log`/`.txt` file via the File API and parses every line |
| **Load sample attack log** | Loads a curated scenario: normal traffic → an off-hours login → a brute-force burst that ends in a compromised login → a traffic spike from a second IP. Good for demoing every detector at once. |
| **Generate 40 random lines** | Produces realistic-looking background SSH noise (mix of accepted/failed/invalid/closed events) |
| **Reset everything** | Clears all logs, alerts, and per-IP tracking state |
| **Start / Stop live monitoring** | Begins appending a new synthetic log line every *N* seconds (adjustable via the speed slider), simulating a file that's actively growing |
| **Inject attack burst now** | Manually fires a concentrated brute-force burst from an attacker IP into the live stream, so you can watch detection happen in real time |

### 2. Log tail (center column)

A scrolling, color-coded console view of every parsed line:

- Left border color indicates event type — green (accepted), amber (failed), red (invalid user), gray (connection closed), blue (other).
- A **search box** filters visible lines by any substring — IP, username, or raw text.
- **Autoscroll** keeps the newest line in view; uncheck it to read history without the view jumping.
- New lines briefly flash to draw the eye when they arrive (respects `prefers-reduced-motion`).

### 3. Rule engine & thresholds (left column, below log source)

**Rule engine** — regex rules that are checked against every raw log line as it's parsed:

- 4 built-in rules ship enabled by default: root login attempts, no-identification-string probes (possible port scanning), invalid-user probes, and failed publickey auth.
- Each rule can be toggled on/off individually.
- You can add your own: a name, a regex pattern, and a severity (critical/high/medium/low). Any match produces an immediate "rule-match" alert.

**Detection thresholds** — tunable parameters for the statistical detectors described below:

- Brute force: number of failed attempts + time window (seconds)
- IP spike: number of total attempts + time window (seconds)
- Off-hours window: start/end time (wraps midnight correctly, e.g. 22:00–06:00)

### 4. Alerts dashboard (right column)

- **Summary cards**: total alerts, critical count, high-severity count, current top offending IP (by failed-login count).
- **Chart**: horizontal bar chart of the top 5 IPs by failed-login count.
- **Alerts table**: time, severity badge, message, source IP — filterable by severity via the chip row above it.
- **Export JSON / Export CSV**: downloads the full current alert list.
- **Clear alerts**: empties the alerts table without touching the parsed log history.
- The **threat pulse** strip at the top of the page is a live waveform: it idles at a low baseline and spikes — taller and redder for higher severity — every time a detector fires, giving an at-a-glance sense of activity level even before reading the table.

---

## Detection logic

All detection runs client-side, per parsed line, against rolling per-IP windows.

| Detector | Trigger | Default severity |
|---|---|---|
| **Rule match** | Any enabled regex rule matches the raw line | as configured per rule |
| **Brute force** | ≥ N failed logins from the same IP within the failed-login window | high |
| **Possible compromise** | An *accepted* login from an IP that just tripped the brute-force threshold, within the same window | critical |
| **IP spike** | ≥ N total connection attempts (any type) from the same IP within the spike window | medium |
| **Off-hours login** | An *accepted* login whose timestamp falls inside the configured off-hours range | medium |

Notes on the implementation:

- Each per-IP counter only re-fires once the count *exceeds* its previous alerted value, so an ongoing burst doesn't spam duplicate alerts for the same threshold crossing.
- Timestamps are extracted from an `HH:MM:SS` pattern in the line and combined with today's date (standard syslog lines omit the year, so this is a best-effort reconstruction — fine for a simulator, not something to rely on for forensic accuracy).
- If a pasted/uploaded line doesn't match any of the known sshd sub-patterns, it's still parsed and displayed (tagged "other") but won't drive the statistical detectors — only regex rules can flag it.

---

## Note on the inotify stretch goal

The original brief's stretch goal was real-time monitoring via `inotify` watching a log file as it grows. That's a Linux filesystem API — it has no browser equivalent, and a static page hosted on GitHub Pages has no access to the visitor's filesystem at all (by design, for security).

What's implemented instead is a **simulated live tail**: an interval timer appends new synthetic log lines on a configurable cadence, and everything downstream (parsing, detection, rendering) runs exactly as it would on real streaming data. This demonstrates the full detection pipeline in real time without requiring a backend.

If literal `inotify`-driven monitoring of a real file is needed, that requires a small local process (Node/Python watching the file and pushing updates over WebSocket, for example) — a different deployment shape than a static site.

---

## Privacy / data handling

Everything — parsing, detection, rendering — happens in the browser's memory. No log data, alerts, or configuration are sent to any server, stored in cookies, or persisted between page loads. Refreshing the page clears all state.
