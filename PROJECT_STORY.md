# Cursivis Next

> **The most frustrating part of using AI isn't getting the answer. It's stopping your work to explain everything the AI could already see.**

---

## 💡 Inspiration

The usual AI workflow breaks concentration:

1. Copy content
2. Leave your application
3. Open a chatbot
4. Paste and reconstruct context
5. Wait for response
6. Copy answer back

**Real work doesn't begin in a blank prompt box.** It begins in the browser tab, code editor, document, or email already open on your screen.

### The Core Question
*What if AI could understand the context already in front of you?*

Cursivis Next answers this with a simple interaction model:

```
Selection = Context
Trigger = Intent
Cursivis = Action
```

We didn't build another chatbot. We built an assistant that **appears when needed, helps you move forward, and disappears back into your workflow.**

---

## ✨ What It Does

Cursivis Next is a context-aware Windows AI assistant that works across all your applications.

### Core Workflow
**Select → Trigger → Understand → Respond → Refine or Act**

Highlight text, press a global hotkey, and Cursivis adapts:

| Context | Result |
|---------|--------|
| Article | Concise summary |
| Difficult paragraph | Simplified explanation |
| Foreign text | Translation |
| Email draft | Professional rewrite |
| Code snippet | Explanation & review |
| Messy notes | Structured action items |
| Rough prompt | Polished AI instruction |
| Image/screen region | Visual analysis |

**One interaction. Infinite adaptations.**

### Key Features

**🎯 Smart Mode**
Instantly infers the best transformation and executes it—no menus, no choices, just results.

**🧭 Guided Mode**
Dynamically generates contextual options (Summarize, Explain, Rewrite, Translate, Extract) based on your actual selection.

**👁️ Screen & Image Understanding**
Select any screen region when text isn't available. Analyzes screenshots, error messages, diagrams, and visual documents.

**💬 Live Mode**
Real-time voice conversation with bounded tools:
- Refresh selected text
- Analyze screen regions
- Insert text into applications
- Navigate complex interfaces
- User-controlled memory

**🎤 Smart Dictation**
Transcribes speech, improves punctuation and structure, inserts polished text directly where you're working.

**🔧 Prompt Optimizer**
Transforms incomplete instructions into clear, structured prompts before sending to any AI system.

**🧭 Navigation Guidance**
Step-by-step task assistance: observe → instruct → wait → verify → continue. **You stay in control.**

**⚡ Take Action**
Structured browser automation with safety validation—fills forms, handles interactions, requires confirmation for sensitive operations.

---

## 🏗️ How We Built It

### Technology Stack
- **Platform**: C# + .NET 8 + WinUI 3
- **AI Integration**: OpenAI Responses, Realtime, & Transcription APIs
- **Windows APIs**: UI Automation, Global Hotkeys, Audio Capture
- **Browser**: Chromium Extension + Native Messaging
- **Installer**: Inno Setup

### Architecture

```
Global Keyboard Shortcuts
        ↓
Windows Context Capture
        ↓
Application Layer
        ↓
AI Services (Responses, Realtime, Audio)
        ↓
Result / Voice / Guidance / Action
```

### Technical Highlights

**Context Capture**
Multi-strategy approach combining:
- Windows UI Automation
- Protected clipboard fallback
- Foreground window validation
- Clipboard restoration
- Cross-app compatibility (browsers, editors, documents, PDFs)

**Desktop Overlays**
Native Windows integration:
- Transparent compositor backdrops
- DPI-aware positioning
- Draggable/resizable surfaces
- Persistent placement
- Non-activating windows

**Browser Action Layer**
- Chromium extension
- Authenticated local bridge
- Structured action planning
- Safety classification
- Confirmation for sensitive operations

**Privacy & Security**
- Local settings storage
- Windows encrypted credential storage
- No ambient monitoring
- Explicit user consent for screen capture

---

## 🚧 Challenges We Ran Into

### 1. **Reliable Selection Capture**
Different apps expose text differently. Chrome selected "extempore" but returned the browser URL instead.

**Solution**: Multi-fallback strategy with browser-specific safeguards and real cross-application testing.

### 2. **Global Shortcuts**
Shortcuts owned by other apps, thread ownership issues, registration rollback failures.

**Solution**: Transactional registration on WinUI owner thread with conflict detection, visible fallback reporting, and persistence of configured vs. active shortcuts.

### 3. **Transparent Overlays**
Opaque swap chains, black edges, DPI-dependent clipping, focus stealing.

**Solution**: Native window regions, compositor handling, DPI-aware geometry, screenshot-based validation.

### 4. **Realtime Interaction**
Coordinating microphone capture, audio buffering, interruption, tool calls, transcripts, and cancellation.

**Solution**: Managed session lifecycle with coherent state transitions across all interaction points.

### 5. **Balancing Action with Safety**
Enabling useful actions without creating risk.

**Solution**: Bounded tools, structured validation, explicit permission, confirmation UI, and undo support.

---

## 🏆 Accomplishments We're Proud Of

✅ **Shortcut-driven AI assistant** that works system-wide  
✅ **Non-destructive clipboard capture** with restoration  
✅ **Smart Mode + Guided Mode** adaptive intelligence  
✅ **Image-region analysis** when text is unavailable  
✅ **Context preservation** across refinement, insertion, replacement, undo  
✅ **Voice mode with tools** (screen analysis, text insertion, navigation)  
✅ **Explicit user-controlled memory** (no silent conversation saving)  
✅ **Navigation Guidance** that observes, waits, and verifies  
✅ **Structured browser automation** with safety boundaries  
✅ **Visible shortcut conflict resolution**  
✅ **Clean installation** isolated from previous versions  
✅ **Windows encrypted API key storage**  
✅ **Self-contained installer** with startup integration

### Quality Metrics
- **167 unit tests** ✅
- **71 integration tests** ✅
- **Clean x64 Release build** (zero warnings, zero errors) ✅

Tests cover invisible reliability:
- Clipboard restoration
- Shortcut persistence & rollback
- Thread ownership
- Cancellation
- Storage migration
- Safety validation
- Error recovery

---

## 📚 What We Learned

### The Biggest Lesson
**The quality of an AI product is determined by far more than the intelligence of its model.**

A powerful model cannot compensate for:
- Capturing the wrong selection
- Losing a keyboard shortcut
- Stealing focus
- Damaging clipboard contents
- Displaying broken overlays
- Forgetting context
- Executing unintended actions

### Key Insights
- **Context capture is often harder than reasoning**
- **Selection is a better prompt than a blank text box**
- **A shortcut expresses intent faster than another menu**
- **Invisible reliability is a real product feature**
- **Users trust actions when they're visible, bounded, reversible, and cancellable**
- **Desktop AI must respect focus, DPI, accessibility, clipboard state, OS lifecycle, and native thread ownership**

**Most importantly**: AI doesn't need to dominate the workflow to transform it. It can appear for a few seconds, understand what matters, help you move forward, and disappear again.

---

## 🚀 What's Next

### Immediate Goals
- Polish into a dependable Windows productivity layer
- Improve selection reliability across all application types
- Enhance Realtime voice stability and interruption handling
- Strengthen image and screen understanding
- Refine Navigation Guidance
- Expand safe browser workflows
- Improve accessibility and keyboard-first interaction

### Long-Term Vision
- **Developer capability system** for extensible context tools
- **macOS version** using AppKit
- **Local model integration** (Ollama, LM Studio)
- **Application-specific context adapters**
- **Evaluation systems** for action correctness and safety
- **Seamless installer and update experience**

### The Ultimate Goal

> *Cursivis should make AI feel less like a destination and more like a capability already present inside the work.*

**The cursor should not merely point at information.**  
**The selection should not merely highlight it.**  
**The shortcut should not merely open another application.**

**Together, they should become the beginning of understanding—and, when you choose, the beginning of action.**

---

## 🔗 Links

- **Repository**: [github.com/UnknownGod2024/cursivis-nextt](https://github.com/UnknownGod2024/cursivis-nextt)
- **Download**: [Releases](https://github.com/UnknownGod2024/cursivis-nextt/releases)
- **License**: MIT
- **Platform**: Windows 10 2004+ | Windows 11

---

## 🏷️ Built With

`.net` · `c-sharp` · `winui-3` · `windows` · `ai` · `claude` · `gpt` · `gemini` · `chromium-extension` · `ui-automation` · `realtime-api` · `speech-to-text`

---

*Built for [Devpost Build with AI: Basics](https://build-with-ai-basics.devpost.com/)*
