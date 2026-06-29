<h1 align="center">Eric Esoteric</h1>

<p align="center">
  <strong>Software Developer · AI Integration · Desktop & Backend Engineering</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3670A0?style=flat&logo=python&logoColor=ffdd54">
  <img src="https://img.shields.io/badge/AI_Integration-Multi--Provider-00B981?style=flat">
  <img src="https://img.shields.io/badge/Windows-Win32_API-0078D4?style=flat&logo=windows">
  <img src="https://img.shields.io/badge/Linux-X11-FCC624?style=flat&logo=linux&logoColor=black">
  <img src="https://img.shields.io/badge/Desktop-System_Tray_App-6E40C9?style=flat">
</p>

---

I build production-grade desktop software and AI-powered automation tools — the kind that handle real edge cases, recover from failures gracefully, and ship as polished executables. My projects are designed from the architecture down, not assembled from tutorials.

---

## What I Actually Build

### Job Hunter AI — AI Recruitment Assistant
> *A full-stack desktop application that automates job search analysis end-to-end.*

**The problem:** Job boards are flooded with scam postings, MLM schemes, and predatory listings. Screening them manually wastes hours. Writing tailored cover letters wastes more.

**What I shipped:**

- **Browser-agnostic capture engine** — A global hotkey listener (`pynput`) runs as a daemon thread. On trigger, it simulates `Ctrl+A → Ctrl+C` in whatever browser is active, reads the clipboard via `pyperclip`, and enqueues the text. No browser extension, no open ports, no Chrome dependency — works on Firefox, Edge, Brave, or any site. Hardware VK keycodes ensure the hotkey fires correctly regardless of the active keyboard layout (Cyrillic, QWERTY, Dvorak). Linux X11 supported natively; Wayland raises a `PlatformSecurityException` with a clear remediation message rather than silently failing.

- **Multi-provider AI cascade with automatic failover** — The engine tries Gemini → GPT → Claude → DeepSeek → Ollama → LM Studio in sequence. If a provider is down, rate-limited, or returns garbage, the next one picks up the task without losing it. Exponential backoff, structured error hierarchy (`AINetworkError` / `AITimeoutError` / `AIRateLimitError` / `AIAuthError`), zero manual intervention.

- **5-level JSON repair pipeline** — LLMs regularly return malformed JSON. A cascading parser strips Markdown wrappers, fixes trailing commas, corrects boolean literals, repairs broken quotes, and recovers the response. Results are never silently dropped.

- **Two-stage AI analysis** — Stage 1 is a hard filter: detects scam, MLM, toxic labor conditions (>45 h/week, unpaid overtime, mass hiring), and geographic compliance violations — up to 60% of listings rejected here. Stage 2 runs only on approved listings and generates a targeted cover letter matched to actual job requirements. No templates, no filler.

- **Relevance scoring pipeline** — `extract_relevant_context()` normalizes whitespace, drops navigation noise, scores paragraphs by keyword density and length, greedily selects the most relevant content within a char budget, then restores original document order (Narrative Rule) before submitting to the LLM. Stages 1 and 2 each run the pipeline independently to prevent context drift.

- **Crash-safe atomic storage** — `_write_json_atomic()` uses Write-Copy-Replace: `mkstemp` on the same partition → `json.dump` → `flush` → `fsync` → `os.replace()`. The live file is never opened with `O_TRUNC`. Two independent, never-nested locks: `_file_lock` for disk I/O, `_url_lock` for in-memory set mutations. Always-live `_approved_urls` / `_rejected_urls` populated at startup — O(1) dedup with zero disk reads on the hot path.

- **Local AI integration (Ollama / LM Studio)** — Full HTTP integration with local LLM servers. A background probe monitors availability and reflects live status in the UI. `LOCAL_SAFE_PARAMS` compensate for artifacts in quantized 4-bit models. A `MIN_TOKENS_PER_SEC` threshold detects stalled generations before they time out.

- **HiDPI-aware windowing via Win32 API** — Child windows open at `alpha=0.0`, compute their final coordinates for the current DPI, then fade in only after the frame is fully rendered. No flickering, no position jumps. Dark title bars and custom icons set through `ctypes` Win32 calls.

- **Thread-safe toast system** — Animated notifications slide in from the screen bottom with taskbar-aware positioning. `_notification_lock` guards `_toast_ref` mutations; `_fade_out_instance` captures the toast by identity so concurrent notifications can't corrupt each other's fade animation. Audio plays in a dedicated daemon thread.

- **Full EN/RU localization + self-healing build** — Every string routes through `jh_i18n.py` with named variable substitution, switching at runtime. `build_exe.py` patches import paths before PyInstaller bundles the executable; `jh_version.py` is the single source of truth for version strings across window titles, `.exe` VERSIONINFO, and the UI.

---

## Engineering Approach

```
Problem → Architecture → Edge Cases → Ship
```

I don't prototype and hope. Before writing a line of code I map the failure modes:
what happens when the API is down, when the model returns malformed data, when the
user runs on a 4K display, when the queue backs up, when the hotkey fires on a
Wayland session. The answers shape the design, not the other way around.

---

## Tech Stack

| Layer | Tools |
|---|---|
| **Language** | Python 3.10+ |
| **GUI & Tray** | CustomTkinter · pystray · Pillow · ctypes Win32 API |
| **Hotkey & Clipboard** | pynput · hardware VK codes (layout-independent) · pyperclip |
| **AI Providers** | Gemini 2.5 · GPT-5 / o3 · Claude 4 · DeepSeek · Ollama · LM Studio |
| **Resilience** | Failover Chain · Exponential Backoff · 5-level JSON parser · custom exception hierarchy |
| **Platform** | Windows (Win32) · Linux X11 · Wayland guard with graceful degradation |
| **Build** | PyInstaller · self-healing build scripts · single-source versioning |
| **Storage** | Atomic Write-Copy-Replace + fsync · independent file/URL locks · O(1) dedup |

---

## Custom Exception Hierarchy

```python
AIBaseError
├── AINetworkError       # Provider unreachable
├── AITimeoutError       # Generation stalled / MIN_TOKENS_PER_SEC breach
├── AIRateLimitError     # 429 — route to next provider
├── AIAuthError          # Invalid key — skip provider, alert user
└── AILocalServerError   # Ollama / LM Studio not running
```

Each exception type triggers a specific UI response and fallback path. The UI never shows raw tracebacks — users see actionable messages, engineers see structured logs.

---

## What I'm Looking For

I'm open to roles where engineering quality matters — where the difference between
a working prototype and a production system is taken seriously.

**Strong fit:** backend systems, AI/LLM integration, desktop application development,
developer tooling, automation infrastructure.

**Available for:** full-time, contract, remote.

---

<p align="center">
  <a href="mailto:mashamasha.vishnya@gmail.com">
    <img src="https://img.shields.io/badge/Email-mashamasha.vishnya%40gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white">
  </a>
</p>
