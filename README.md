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

## My greatest strength: I don't just build apps, I build a suite

Anyone can ship one hotkey tool. What I actually built is three of them — **Job Hunter AI**, **Lingo Hunter AI**, and **Code Hunter AI** — sharing one engine (multi-provider AI failover, global hotkey capture, atomic storage, tray/theme system) and designed from day one to hand off work to each other. That's the difference between three separate utilities and an ecosystem: the output of one becomes the input of the next, with zero copy-paste friction and zero context-switching.

Two workflows that run through my own daily routine:

- **Think in your language, ship in code.** I draft a request in my own language, hit a hotkey and **Lingo Hunter AI** translates it to English on a free-tier key — no reason to burn a paid model on translation. A second hotkey hands that English text straight to **Code Hunter AI**, which routes it through a stronger, separately-keyed model and pastes working code back exactly where my cursor was. Two specialized tools, two specialized keys, one uninterrupted motion.
- **Find it, apply, in your own voice.** **Job Hunter AI** scans postings, filters out the scams and MLMs, and writes a targeted cover letter. I copy that letter and one hotkey away, **Lingo Hunter AI** converts it into my own language to sanity-check tone and phrasing before it goes out — same engine, same reflex, no app-switching.

That's the strength I bring to a team: I don't just write code that works in isolation, I design systems that compose — where the seams between tools disappear and the person using them stays in flow.

---

## What I Actually Build

### Job Hunter AI — AI Recruitment Assistant
> *A full-stack desktop application that automates job search analysis end-to-end.*

**The problem:** Job boards are flooded with scam postings, MLM schemes, and predatory listings. Screening them manually wastes hours. Writing tailored cover letters wastes more.

**What I shipped:**

- **Browser-agnostic capture engine** — A global hotkey listener (`pynput`) runs as a daemon thread. On trigger, it simulates `Ctrl+A → Ctrl+C` in whatever browser is active, reads the clipboard via `pyperclip`, and enqueues the text. No browser extension, no open ports, no Chrome dependency — works on Firefox, Edge, Brave, or any site. Hardware VK keycodes ensure the hotkey fires correctly regardless of the active keyboard layout (Cyrillic, QWERTY, Dvorak). Linux X11 supported natively; Wayland raises a `PlatformSecurityException` with a clear remediation message rather than silently failing.

- **Multi-provider AI cascade with automatic failover** — The engine tries Gemini → GPT-5 → Claude 4 → DeepSeek → OpenRouter → Ollama → LM Studio in sequence. If a provider is down, rate-limited, or returns garbage, the next one picks up the task without losing it. Exponential backoff, structured error hierarchy (`AINetworkError` / `AITimeoutError` / `AIRateLimitError` / `AIAuthError`), zero manual intervention.

- **5-level JSON repair pipeline** — LLMs regularly return malformed JSON. A cascading parser strips Markdown wrappers, fixes trailing commas, corrects boolean literals, repairs broken quotes, and recovers the response. Results are never silently dropped.

- **Two-stage AI analysis** — Stage 1 is a hard filter: detects scam, MLM, toxic labor conditions (>45 h/week, unpaid overtime, mass hiring), and geographic compliance violations — up to 60% of listings rejected here. Stage 2 runs only on approved listings and generates a targeted cover letter matched to actual job requirements. No templates, no filler.

- **Relevance scoring pipeline** — `extract_relevant_context()` normalizes whitespace, drops navigation noise, scores paragraphs by keyword density and length, greedily selects the most relevant content within a char budget, then restores original document order (Narrative Rule) before submitting to the LLM. Stages 1 and 2 each run the pipeline independently to prevent context drift.

- **Crash-safe atomic storage** — `_write_json_atomic()` uses Write-Copy-Replace: `mkstemp` on the same partition → `json.dump` → `flush` → `fsync` → `os.replace()`. The live file is never opened with `O_TRUNC`. Two independent, never-nested locks: `_file_lock` for disk I/O, `_url_lock` for in-memory set mutations. Always-live `_approved_urls` / `_rejected_urls` populated at startup — O(1) dedup with zero disk reads on the hot path.

- **Local AI integration (Ollama / LM Studio)** — Full HTTP integration with local LLM servers. A background probe monitors availability and reflects live status in the UI. `LOCAL_SAFE_PARAMS` compensate for artifacts in quantized 4-bit models. A `MIN_TOKENS_PER_SEC` threshold detects stalled generations before they time out.

- **HiDPI-aware windowing via Win32 API** — Child windows open at `alpha=0.0`, compute their final coordinates for the current DPI, then fade in only after the frame is fully rendered. No flickering, no position jumps. Dark title bars and custom icons set through `ctypes` Win32 calls.

- **Thread-safe toast system** — Animated notifications slide in from the screen bottom with taskbar-aware positioning. `_notification_lock` guards `_toast_ref` mutations; `_fade_out_instance` captures the toast by identity so concurrent notifications can't corrupt each other's fade animation. Audio plays in a dedicated daemon thread.

- **Full EN/RU localization + self-healing build** — Every string routes through `jh_i18n.py` with named variable substitution, switching at runtime. `build_exe.py` patches import paths before PyInstaller bundles the executable; `jh_version.py` is the single source of truth for version strings across window titles, `.exe` VERSIONINFO, and the UI.

Now on v3.1.1, with a 6-provider AI cascade (added OpenRouter), a full concurrency/reliability audit, and standalone operation — the Chrome extension was dropped entirely in v3.0.0 in favor of the same global-hotkey capture engine that now powers all three apps in the suite.

---

### Lingo Hunter AI — Translate in Place, No Tab-Switching
> *Type in any language. Hit a hotkey. It's translated — in place, instantly.*

**The problem:** A Slack DM, a job-board comment, a line in a game's chat — right now translating any of it means selecting the text, alt-tabbing to a browser, pasting into a translator, waiting, copying the result, and pasting it back, hoping the formatting survived.

**What I shipped:**

- **Zero-selection capture** — Type your message anywhere, hit the hotkey (`Ctrl+Shift+Z` by default). No selecting, no mouse: the app grabs everything in the active field, translates it, and pastes it straight back in place.
- **Works in any app** — Any focused text field, any application. No allow-list, no per-site integration.
- **Same 5-cloud + 2-local failover engine** as the rest of the suite (Gemini, OpenAI, Anthropic, DeepSeek, OpenRouter, plus Ollama/LM Studio) — if one provider is slow or out of quota, the next takes over mid-sentence, invisibly.
- **"Expressive" mode** — most translation tools quietly sand slang and tone down to something safe and corporate; Expressive translates as-is. A "Standard" mode is there when you want the safer default instead.
- **Bring your own key, own traffic** — talks directly to the provider you configure. No middleman server relaying or logging messages.
- **One settings panel** — target language (with starred favorites for one-click switching), hotkey, provider, and per-provider failover order, all in a single screen. Two built-in themes.

This is the translation layer that the rest of the suite is built to hand off to — the same engine, repointed, is what became Code Hunter AI below.

---

### Code Hunter AI — Natural Language to Working Code, In Place
> *Type what you want in plain English. Highlight it. Hit one key. Watch it turn into working code — right where your cursor is.*

**The problem:** You know exactly what you need — "quicksort," "a rate limiter," "parse this date string" — but still have to alt-tab to a chat window, describe it, wait, copy the answer, alt-tab back, and paste it in, hoping it's not wrapped in three paragraphs of explanation you didn't ask for.

**What I shipped:**

- **Selection-scoped, never file-scoped** — Code Hunter AI acts only on text you've deliberately selected — it never grabs "the current paragraph" or an entire file, which is what makes it safe to run inside a real IDE with a real codebase open.
- **Code back, nothing else** — By default the response is code and only code: no comments, no docstrings, no "Here's how this works!" preamble. An anti-mirror / no-hallucination system prompt stops the model from echoing the request back unchanged or inventing unrequested features. A comment-detection backstop catches any model that adds comments despite "Code only" mode.
- **Rebuilt from Lingo Hunter AI's engine** — same multi-provider failover, hotkey capture, tray and theme system, repointed at a different job: turning a natural-language request into working code in the language you pick, instead of translating between human languages.
- **16 target languages** out of the box — Python, JavaScript, TypeScript, Java, C#, C++, C, Go, Rust, PHP, Ruby, Swift, Kotlin, SQL, Bash, HTML/CSS — or type in anything else.
- **Independent provider and key from Lingo Hunter** — pointed at a more powerful model on its own key, so cheap translation and heavyweight code generation never compete for the same quota.

---

## Engineering Approach

```
Problem → Architecture → Edge Cases → Ship
```

I don't prototype and hope. Before writing a line of code I map the failure modes:
what happens when the API is down, when the model returns malformed data, when the
user runs on a 4K display, when the queue backs up, when the hotkey fires on a
Wayland session. The answers shape the design, not the other way around.

The same discipline applies across the suite, not just within one app: shared failover
logic, shared atomic storage, shared i18n system — fixed once, inherited everywhere.

---

## Tech Stack

| Layer | Tools |
|---|---|
| **Language** | Python 3.10+ |
| **GUI & Tray** | CustomTkinter · pystray · Pillow · ctypes Win32 API |
| **Hotkey & Clipboard** | pynput · hardware VK codes (layout-independent) · pyperclip |
| **AI Providers** | Gemini 2.5 · GPT-5 / o3 · Claude 4 · DeepSeek · OpenRouter · Ollama · LM Studio |
| **Resilience** | Failover Chain · Exponential Backoff · 5-level JSON parser · custom exception hierarchy |
| **Platform** | Windows (Win32) · Linux X11 · Wayland guard with graceful degradation |
| **Build** | PyInstaller · self-healing build scripts · single-source versioning |
| **Storage** | Atomic Write-Copy-Replace + fsync · independent file/URL locks · O(1) dedup |
| **Localization** | Declarative EN/RU with named variable substitution, runtime switching |

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
a working prototype and a production system is taken seriously, and where designing
tools that work together is valued as much as any single feature.

**Strong fit:** backend systems, AI/LLM integration, desktop application development,
developer tooling, automation infrastructure, multi-app / platform ecosystems.

**Available for:** full-time, contract, remote.

---

<p align="center">
  <a href="mailto:mashamasha.vishnya@gmail.com">
    <img src="https://img.shields.io/badge/Email-mashamasha.vishnya%40gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white">
  </a>
</p>
