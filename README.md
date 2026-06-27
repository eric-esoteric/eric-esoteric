<h1 align="center">Eric Esoteric</h1>

<p align="center">
  <strong>Software Developer · AI Integration · Desktop & Backend Engineering</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3670A0?style=flat&logo=python&logoColor=ffdd54">
  <img src="https://img.shields.io/badge/Flask-API-000000?style=flat&logo=flask">
  <img src="https://img.shields.io/badge/AI_Integration-Multi--Provider-00B981?style=flat">
  <img src="https://img.shields.io/badge/Windows-Win32_API-0078D4?style=flat&logo=windows">
  <img src="https://img.shields.io/badge/Chrome-Extension-4285F4?style=flat&logo=googlechrome&logoColor=white">
</p>

---

I build production-grade desktop software and AI-powered automation tools — the kind that handle real edge cases, recover from failures gracefully, and ship as polished executables. My projects are designed from the architecture down, not assembled from tutorials.

---

## What I Actually Build

### Job Hunter AI — AI Recruitment Assistant
> *A full-stack desktop application that automates job search analysis end-to-end.*

**The problem:** Job boards are flooded with scam postings, MLM schemes, and predatory listings. Screening them manually wastes hours. Writing tailored cover letters wastes more.

**What I shipped:**

- **Multi-provider AI cascade with automatic failover** — The engine tries Gemini → GPT → Claude → DeepSeek → Ollama → LM Studio in order. If one provider is down, rate-limited, or returns garbage, the next one picks up the task without losing it. Zero manual intervention required.

- **5-level JSON repair pipeline** — LLMs regularly return malformed JSON. Rather than crashing or discarding results, a cascading parser strips Markdown wrappers, fixes trailing commas, corrects boolean literals, repairs broken quotes, and recovers the response. Nothing gets lost to model formatting quirks.

- **Two-stage AI analysis** — Stage 1 is a hard filter: detects scam, MLM, toxic labor conditions (>45h/week, unpaid overtime, mass hiring), and geographic compliance violations. Up to 60% of listings get rejected here. Stage 2 runs only on approved listings and generates a targeted cover letter matched to the actual job requirements — no templates, no filler.

- **Local AI integration (Ollama / LM Studio)** — Full HTTP integration with local LLM servers. A background probe monitors server availability and reflects status in the UI in real time. `LOCAL_SAFE_PARAMS` compensate for artifacts in quantized 4-bit models. A `MIN_TOKENS_PER_SEC` metric detects stalled generations before they timeout.

- **Chrome extension → Flask webhook pipeline** — A Chrome extension captures vacancy data from the browser and sends it to a local Flask API. A thread-safe queue with a 15-second rate-limit timer and a live countdown status bar manages the processing pipeline. A zombie-process killer ensures port 5000 is always clean on startup.

- **HiDPI-aware windowing via Win32 API** — Child windows open at `alpha=0.0`, compute their final coordinates for the current DPI, then fade in only after the frame is fully rendered. No flickering, no position jumps. Dark title bars and custom icons set through `ctypes` Win32 calls.

- **Signature-cached UI rendering** — The vacancy list computes a hash signature for each card. On update, only cards whose content changed are re-rendered. No full redraws, no visual noise.

- **Animated toast notification system** — Custom slide-in notifications with color coding (cyan = success, red = alert), taskbar-aware positioning, and audio playback in a dedicated thread — all without touching system dialogs.

- **Full EN/RU localization** — Every string in the interface passes through a single `jh_i18n.py` module with named variable substitution. Language switches at runtime and persists to config.

- **Self-healing build system** — `build_exe.py` contains a self-healing refactor step that patches import paths before PyInstaller bundles the executable. `jh_version.py` is the single source of truth for the version string across window titles, the `.exe` VERSIONINFO metadata, and the UI.

---

## Engineering Approach

```
Problem → Architecture → Edge Cases → Ship
```

I don't prototype and hope. Before writing a line of code I map the failure modes:
what happens when the API is down, when the model returns malformed data, when the
user runs on a 4K display, when the queue backs up, when a port is already in use.
The answers shape the design, not the other way around.

---

## Tech Stack

| Layer | Tools |
|---|---|
| **Language** | Python 3.10+ |
| **GUI** | CustomTkinter · Pillow · ctypes Win32 API |
| **Backend** | Flask · flask-cors · threading · queue.Queue |
| **AI Providers** | Gemini 3.5/3.0 · GPT-5 / o3 · Claude 4 · DeepSeek · Ollama · LM Studio |
| **Resilience** | Failover Chain · Exponential Backoff · 5-level JSON parser |
| **Browser** | Chrome Extension (Manifest V3) · Webhook integration |
| **Build** | PyInstaller · Self-healing build scripts · Single-source versioning |
| **Storage** | JSON / AppData · PDF import with AI text extraction |

---

## Custom Exception Hierarchy (example of real error handling)

```python
AIBaseError
├── AINetworkError       # Provider unreachable
├── AITimeoutError       # Generation stalled / MIN_TOKENS_PER_SEC breach
├── AIRateLimitError     # 429 — route to next provider
├── AIAuthError          # Invalid key — skip provider, alert user
└── AILocalServerError   # Ollama / LM Studio not running
```

Each exception type triggers a specific UI response and fallback path. The UI never shows raw tracebacks — users see actionable messages. Engineers see structured logs.

---

## What I'm Looking For

I'm open to roles where engineering quality matters — where the difference between
a working prototype and a production system is taken seriously.

**Strong fit:** backend systems, AI/LLM integration, desktop application development,
developer tooling, automation infrastructure.

**Available for:** full-time, contract, remote.

---

<p align="center">
  <a href="mailto:teammudlo@gmail.com">
    <img src="https://img.shields.io/badge/Email-teammudlo%40gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white">
  </a>
</p>
