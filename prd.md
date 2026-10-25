# Cursivis Next - Product Requirements Document

## Executive Summary

**Product Name**: Cursivis Next  
**Version**: 0.1.0-beta  
**Target Platform**: Windows 10 2004+, Windows 11 (x64)  
**Category**: AI-Powered Productivity Tool  
**Hackathon**: Devpost Build with AI: Basics  

Cursivis Next eliminates the friction between AI capabilities and active work by providing context-aware AI operations triggered via global hotkeys. Instead of copying text to a separate chat interface, users select content and invoke AI transformations inline.

## Problem Statement

### Current Pain Points
1. **Context Switching**: Users must leave their active application to use AI tools
2. **Manual Context Transfer**: Copy-paste cycles waste time and break flow
3. **Reconstruction Overhead**: AI tools don't know what the selected text represents
4. **Result Management**: Getting AI output back into the original app requires manual steps
5. **Cognitive Load**: Remembering which AI chat to use and reconstructing prompts

### User Impact
- 30-60 seconds lost per AI interaction due to app switching
- 15-20 AI queries per day for knowledge workers = **7-20 minutes daily friction**
- Broken flow state reduces deep work quality
- Incomplete context leads to suboptimal AI responses

## Target Users

### Primary Personas

#### Alex - Software Developer
- **Needs**: Quick code explanations, refactoring suggestions, documentation generation
- **Current Solution**: Copies code to ChatGPT, loses IDE context
- **Cursivis Value**: Select code → Ctrl+Alt+O → instant AI response → insert back into IDE

#### Jordan - Content Writer
- **Needs**: Grammar fixes, tone adjustments, expansion/summarization
- **Current Solution**: Google Docs sidebar or separate AI tab
- **Cursivis Value**: Select text → trigger → refined output → replace original

#### Sam - Researcher
- **Needs**: Summarize articles, extract key points, reformulate queries
- **Current Solution**: Manual note-taking while consulting AI
- **Cursivis Value**: Select paragraph → get summary → continue reading

#### Morgan - Business Analyst
- **Needs**: Format data, generate reports, draft emails from notes
- **Current Solution**: Multiple tool switches between Excel, Outlook, AI
- **Cursivis Value**: Select data → transform → paste result

## Product Goals

### For Hackathon Submission
1. **Demonstrate AI Integration Excellence**: Show seamless AI in real-world workflows
2. **Exhibit Technical Sophistication**: WinUI 3 + .NET 8 + browser bridge + platform APIs
3. **Prove Real-World Utility**: Solve actual productivity pain points
4. **Showcase Best Practices**: Security, testing, documentation, installer

### For Users
1. **Reduce friction** in AI-assisted workflows by 80%+
2. **Preserve focus** by eliminating app switching
3. **Maintain context** from selection to result
4. **Enable customization** via hotkeys and quick tasks

## Functional Requirements

### FR-1: Context Trigger
**Priority**: P0 (Must Have)

**User Story**: As a user, I want to select text and press a hotkey to get AI-powered insights without leaving my application.

**Acceptance Criteria**:
- Hotkey (default Ctrl+Alt+O) captures selected text from any application
- If no text selected, opens screen region selector
- Smart Mode automatically determines best operation
- Guided Mode offers contextual operation choices
- Result appears in draggable panel with manipulation options

**Technical Implementation**:
- UI Automation for text selection reading
- Clipboard fallback for inaccessible controls
- Foreground window identity capture
- Smart/Guided mode orchestration via ContextTriggerService

### FR-2: Smart Dictation
**Priority**: P0

**User Story**: As a user, I want to dictate text with proper formatting and insert it where I'm working.

**Acceptance Criteria**:
- Hotkey (default Ctrl+Alt+U) starts microphone capture
- Speech transcribed with punctuation and structure
- Result automatically inserted at cursor or copied to clipboard
- Visual feedback shows recording state
- Cancellation stops recording without side effects

**Technical Implementation**:
- Managed audio session via Windows platform layer
- AI transcription API integration
- Insertion via Windows UI Automation or clipboard

### FR-3: Quick Tasks / Prompt Optimizer
**Priority**: P0

**User Story**: As a user, I want customizable AI operations accessible via hotkey.

**Acceptance Criteria**:
- Default task: Prompt Optimizer (cleans and structures prompts)
- Hotkey (default Ctrl+Alt+Y) triggers current quick task
- Settings allow changing quick task instruction
- Works with selected text or manual input

**Technical Implementation**:
- QuickTaskFinalizationService orchestrates execution
- Saved instructions in ApplicationSettings
- Same result panel as Context Trigger

### FR-4: Live Mode
**Priority**: P1 (Should Have)

**User Story**: As a user, I want real-time voice conversation with AI for complex tasks.

**Acceptance Criteria**:
- Hotkey (default Ctrl+Alt+P) starts/stops Live Mode
- Captures initial context from selection
- Duplex audio conversation
- Transcript panel (show/hide toggle)
- Can invoke tools: refresh selection, analyze screen region, navigation guidance
- Conversation state shows: listening, thinking, speaking

**Technical Implementation**:
- AI Realtime API gateway
- Managed audio input/output
- Tool call validation and execution via LiveModeToolExecutor
- Overlay window with circular orb and transcript panel

### FR-5: Take Action
**Priority**: P1

**User Story**: As a user, I want AI to perform browser actions based on my intent.

**Acceptance Criteria**:
- Hotkey (default Ctrl+Alt+I) or Result Panel button
- Generates action plan for browser tasks
- Shows plan for user confirmation
- Executes in Chromium browser via extension
- Verifies each step completed successfully
- Safe by default: no purchases, deletions, or account changes without explicit approval

**Technical Implementation**:
- BrowserTakeActionService generates typed action plan
- Chromium extension receives via native messaging bridge
- Authenticated same-user named pipe
- Risk classification and confirmation UI

### FR-6: Global Hotkeys
**Priority**: P0

**User Story**: As a user, I want reliable shortcuts that don't conflict with other applications.

**Acceptance Criteria**:
- All hotkeys configurable in Settings
- Conflict detection shows which app owns conflicting shortcut
- Fallback shortcut registered if primary conflicts
- Settings displays both configured and active shortcuts
- Cancel hotkey (Ctrl+Alt+Escape) stops any operation

**Technical Implementation**:
- Platform layer handles RegisterHotKey on WinUI thread
- Transactional registration with rollback
- Persistent settings for configured vs active mappings

### FR-7: Settings & Configuration
**Priority**: P0

**User Story**: As a user, I want to configure AI provider, hotkeys, and privacy preferences.

**Acceptance Criteria**:
- Hotkey (Ctrl+Alt+S) opens Settings window
- Sections: Overview, Hotkeys, AI Provider, Quick Task, Privacy, Browser, Diagnostics
- API key storage uses Windows credential protection
- Test button validates AI provider connectivity
- Launch at sign-in option
- Privacy: opt-in for Navigation Guidance screen observation

**Technical Implementation**:
- WinUI 3 navigation pages
- WindowsOpenAiCredentialSource for protected storage
- SettingsService with JSON persistence
- In-app credential testing

### FR-8: Result Panel Operations
**Priority**: P0

**User Story**: As a user, I want full control over AI-generated results.

**Acceptance Criteria**:
- Copy: Copy result to clipboard
- Refine: Edit instruction and regenerate
- Insert: Place result at original cursor (when supported)
- Replace: Substitute original selection with result (when supported)
- Undo: Restore original content (when supported)
- More Options: Access additional operations
- Auto-copy setting available

**Technical Implementation**:
- ResultPanelController manages state
- Platform insertion/undo via UI Automation
- Draggable, resizable panel with position persistence

## Non-Functional Requirements

### NFR-1: Performance
- Context capture < 500ms
- AI response time depends on provider (typically 2-5s)
- Overlay rendering < 100ms
- Memory footprint < 200MB resident

### NFR-2: Reliability
- Graceful degradation when selection capture fails
- Retry logic for transient API errors
- No crashes from unexpected user inputs
- Cancellation works in all states

### NFR-3: Security
- API keys never leave user's machine (except to AI provider)
- No keys in repository or logs
- Browser bridge authenticated to single user
- Screen capture explicit, never silent

### NFR-4: Usability
- First-run onboarding explains key hotkeys
- Visual feedback for all operations (orb states, animations)
- Keyboard-accessible UI
- Draggable overlays with position memory

### NFR-5: Compatibility
- Windows 10 version 2004 (build 19041) minimum
- Works with UWP, Win32, WPF, Chromium, Electron apps
- Chrome and Edge browser support
- DPI-aware for high-resolution displays

### NFR-6: Maintainability
- Clean architecture: Domain → Application → Infrastructure
- Unit and integration test coverage
- Typed contracts between layers
- No direct AI SDK types in application layer

## User Workflows

### Workflow 1: Quick Text Transformation
1. User selects text in any application
2. Presses Ctrl+Alt+O (Context Trigger)
3. Smart Mode analyzes and executes best operation
4. Result Panel appears with output
5. User clicks Insert/Replace to apply, or Copy to take elsewhere
6. Panel dismissed automatically or manually

**Success Metric**: <10 seconds total, zero app switches

### Workflow 2: Voice Dictation in Email
1. User opens email compose window, places cursor
2. Presses Ctrl+Alt+U (Smart Dictation)
3. Speaks message
4. Cursivis transcribes and formats
5. Text inserted at cursor automatically
6. User continues typing or editing

**Success Metric**: Faster than typing, better than raw voice-to-text

### Workflow 3: Interactive Problem Solving
1. User selects error message
2. Presses Ctrl+Alt+P (Live Mode)
3. Explains issue verbally
4. AI asks clarifying questions
5. Provides step-by-step solution
6. User executes steps, confirms completion
7. Ends session with Ctrl+Alt+P

**Success Metric**: Solves problem without web search or documentation lookup

### Workflow 4: Browser Form Filling
1. User has data to enter (e.g., survey responses)
2. Selects data, presses Ctrl+Alt+I (Take Action)
3. Describes task: "Fill out this form with these answers"
4. Reviews generated action plan
5. Confirms execution
6. Browser fields populated automatically
7. User reviews and submits manually

**Success Metric**: 5x faster than manual entry, zero errors

## Integration Requirements

### AI Provider APIs
- **Required**: API key from Claude/GPT/Gemini
- **Endpoints**: Chat completions, voice transcription, realtime (for Live Mode)
- **Fallback**: User-friendly error if provider unavailable

### Windows Platform
- **UI Automation**: For selection reading and insertion
- **Clipboard**: Protected temporary capture
- **Audio**: Microphone and speaker access
- **Hotkeys**: Global registration via RegisterHotKey
- **Overlays**: Native window regions, non-activating windows

### Browser Integration
- **Chromium Extension**: Manifest V3, requested permissions
- **Native Host**: Installed by app installer, registered in Windows
- **Communication**: Named pipe with user SID authentication

## Success Metrics (Devpost Hackathon)

### Judging Criteria Alignment

**1. Functionality & Completeness**
- All Phase 1 features working end-to-end
- No critical bugs in demo scenarios
- Edge cases handled gracefully

**2. Design & User Experience**
- Intuitive hotkey-driven interface
- Minimal visual clutter (transient overlays)
- Responsive feedback (orb animations, state indicators)

**3. Innovation & Creativity**
- Novel approach: bringing AI to user's context vs. vice versa
- Platform integration depth (UI Automation, browser bridge, hotkeys)
- Safety-first bounded automation

**4. Technical Complexity**
- Multi-layer architecture (Domain, Application, Infrastructure, Platform)
- AI provider abstraction (supports Claude, GPT, Gemini)
- Cross-process communication (browser native messaging)
- Windows platform services integration

**5. Potential Impact**
- Addresses universal productivity pain point
- Applicable to any knowledge worker
- Time savings compound across daily tasks

## Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Hotkey conflicts with other tools | High | Conflict detection, fallback registration, user notification |
| Selection capture fails in some apps | Medium | Multi-strategy fallback (UI Automation → clipboard) |
| AI API failures or rate limits | Medium | Retry logic, clear error messages, offline graceful degradation |
| Browser extension not installed | Medium | In-app status check, setup instructions in Browser settings page |
| Performance on older hardware | Low | Optimized rendering, background operation, minimal footprint |
| Users forget hotkeys | Medium | Persistent overlay with hints, Settings reminder, onboarding |

## Future Enhancements (Post-Hackathon)

- macOS support via AppKit
- Local model integration (Ollama, LM Studio)
- Custom action templates
- Shared team quick tasks
- Voice output/TTS for results
- Firefox and Safari extensions
- Cloud sync for memories and settings
- Plugin system for third-party integrations

## Acceptance Checklist

For hackathon submission, product is ready when:

- [x] All FR-1 through FR-8 implemented
- [x] All NFR-1 through NFR-6 validated
- [x] README documents accurate setup process
- [x] Installer produces working application
- [x] No API keys or secrets in repository
- [x] Demo video showcases all major workflows
- [x] Documentation references correct repository (cursivis-nextt)
- [x] Devpost Learn Skill Pack integrated
- [x] All OpenAI branding removed
- [x] GitHub releases configured for downloads

---

**Document Version**: 1.0  
**Last Updated**: October 25, 2026  
**Product Owner**: Devpost Hackathon Submission  
**Status**: Ready for Review
