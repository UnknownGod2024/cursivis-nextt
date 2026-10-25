# Cursivis Next - Devpost Submission

## Project Title
**Cursivis Next: Context-Aware AI for Windows**

## Tagline
Transform selected text into AI-powered actions without leaving your app - hotkey-driven productivity that preserves your flow.

## Inspiration
Every day, knowledge workers lose 15-20 minutes to a simple tax: copy text → switch to AI chat → reconstruct context → wait → copy result → switch back → paste. This friction compounds across dozens of interactions daily.

We asked: **What if AI came to your context instead of forcing you into its interface?**

Cursivis Next eliminates the app-switching penalty by making AI operations contextual, immediate, and inline.

## What it does
Cursivis Next is a Windows-native AI productivity layer that turns your current selection and context into focused AI actions via global hotkeys:

- **Select text anywhere** → Press `Ctrl+Alt+O` → Get instant AI transformation (summarize, explain, translate, format)
- **Place cursor** → Press `Ctrl+Alt+U` → Dictate with your voice → Text inserted automatically
- **Need interaction?** → Press `Ctrl+Alt+P` → Real-time AI conversation with screen analysis
- **Browser task?** → Press `Ctrl+Alt+I` → AI fills forms safely with your approval

All results can be copied, refined, inserted, replaced, or undone - your workflow stays unbroken.

## How we built it
**Architecture**: Multi-layered .NET 8 application following clean architecture principles

- **UI**: WinUI 3 native Windows application with custom non-client overlays
- **Platform Integration**: Windows APIs (UI Automation for cross-app text capture, RegisterHotKey for global shortcuts, Core Audio for voice)
- **AI Integration**: REST and WebSocket connections to AI providers (Claude, GPT, Gemini)
- **Browser Bridge**: Chromium Manifest V3 extension + native messaging host + authenticated named pipe
- **State Management**: Single interaction state model prevents stale async operations
- **Testing**: 169 unit tests + 76 integration tests with deterministic AI mocks

**Key Technical Challenges**:
1. **Cross-app text capture**: Multi-strategy (UI Automation → protected clipboard fallback) with safeguards against browser chrome/omnibox misreads
2. **Hotkey conflicts**: Transactional registration with conflict detection and fallback persistence
3. **Realtime audio**: Duplex conversation with interruption, bounded tool calls, separate transcript/audio streams
4. **Safe browser automation**: Typed action contracts, risk classification, confirmation flow, step verification

## Challenges we ran into
1. **Hotkey lifecycle**: `RegisterHotKey` requires WM_HOTKEY messages on the registering thread - moved registration to WinUI window thread
2. **Transparent overlays**: DPI-aware rounded clipping, non-activating drag, monitor-aware positioning
3. **Selection capture edge cases**: Chromium URL bar vs page selection disambiguation, clipboard restoration race conditions
4. **Realtime audio buffering**: Overflow protection, cancellation propagation, managed session lifecycle
5. **Unpackaged WinUI distribution**: Self-contained publish requires manually including compiled WinUI resources

## Accomplishments that we're proud of
- **169 unit tests + 76 integration tests** - comprehensive coverage with fast, deterministic execution
- **Zero exposed secrets** - API keys encrypted with Windows DPAPI, validated via `check-secrets.ps1`
- **Cross-app compatibility** - works in Notepad, VS Code, Chrome, Word, Outlook, Slack, Teams
- **Clean architecture** - Domain layer has zero dependencies on AI SDKs or Windows APIs
- **Safety-first browser automation** - typed contracts, risk tiers, verification, no generic command execution
- **Professional installer** - Inno Setup with native host registration, Start Menu integration, clean uninstall

## What we learned
- **Windows platform depth**: UI Automation patterns vary wildly (Win32 vs WPF vs UWP vs Chromium)
- **Audio session management**: Exclusive capture, buffer overflow handling, graceful disposal
- **WinUI 3 overlay behavior**: Non-client regions, DPI awareness, multi-monitor clamping
- **Browser extension security**: Native messaging allowlisting, typed message validation, same-user pipe authentication
- **Repository-scale refactoring**: Codex/GPT-5.6 collaboration accelerated cross-cutting changes (hotkey lifecycle, overlay persistence, test coverage)

## What's next for Cursivis Next
### Short-term (Post-Hackathon)
- Code signing for Windows SmartScreen
- Public website with demo video
- Community feedback integration
- Firefox and Safari extension support

### Long-term Vision
- **macOS support** via AppKit
- **Local model integration** (Ollama, LM Studio)
- **Custom action templates** - user-defined browser automation
- **Voice output** (TTS) for Live Mode responses
- **Cloud sync** for memories and settings across devices
- **Team features** - shared quick tasks and action templates
- **Plugin system** - third-party tool integrations

## Built With
- `.net`
- `c-sharp`
- `winui-3`
- `windows`
- `ai`
- `claude`
- `gpt`
- `gemini`
- `chromium-extension`
- `ui-automation`
- `realtime-api`
- `speech-to-text`

## Try it out
- **GitHub**: [github.com/UnknownGod2024/cursivis-nextt](https://github.com/UnknownGod2024/cursivis-nextt)
- **Download**: [Releases](https://github.com/UnknownGod2024/cursivis-nextt/releases)

## Screenshots & Demo

### 1. Context Trigger in Action
*Select text in VS Code → Press Ctrl+Alt+O → AI explains code → Click Insert*

### 2. Smart Dictation
*Place cursor in email → Press Ctrl+Alt+U → Speak → Text appears formatted*

### 3. Live Mode Conversation
*Press Ctrl+Alt+P → Ask about selected error message → Get step-by-step guidance*

### 4. Take Action - Browser Automation
*Select survey responses → Press Ctrl+Alt+I → Review plan → Confirm → Form filled*

### 5. Settings & Configuration
*Ctrl+Alt+S opens settings: configure hotkeys, AI provider, privacy preferences*

## Team
Solo developer submission

## Additional Information

### Why this matters
**Problem scale**: Knowledge workers make 15-20 AI queries per day. At 30-60s friction per query, that's **7-20 minutes daily** lost to app switching alone. Multiply by broken flow state and incomplete context, and the true cost is much higher.

**Solution impact**: Cursivis reduces that friction by 80%+ while preserving workflow context. It's not just faster - it's fundamentally less disruptive.

### Technical depth
This isn't a simple AI chat wrapper. It's:
- Deep Windows platform integration (UI Automation, global hotkeys, audio sessions, native overlays)
- Cross-process communication (browser ↔ native host ↔ named pipe ↔ app)
- Multi-AI-capability orchestration (chat, realtime, transcription, vision)
- Production-ready installer, testing, error handling, and security

### Extensibility
Clean architecture enables future expansion:
- `IAiProvider` interface supports swapping AI backends (local models, custom APIs)
- `IActionStep` allows new browser action types
- QuickTask system supports user-defined operations
- Settings schema versioning enables safe migrations

---

**Hackathon**: Devpost Build with AI: Basics  
**Submission Date**: October 26, 2026  
**Repository**: https://github.com/UnknownGod2024/cursivis-nextt  
**License**: MIT
