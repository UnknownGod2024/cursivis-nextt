# Cursivis Next - Project Scope

## Project Vision
Cursivis Next is a Windows-first AI productivity layer that transforms selected content, visible interfaces, and voice input into focused AI-powered actions. Built with WinUI 3, .NET 8, and integrated with AI APIs, it eliminates context-switching by bringing AI capabilities directly into your active workflow.

## Core Problem Statement
Current AI workflows create friction: users must copy content, switch applications, reconstruct context, wait for responses, and manually transfer results back. This context-switching compounds throughout the day, fragmenting focus and reducing productivity.

Cursivis solves this by making AI operations contextual and immediate - select text, press a hotkey, get results inline.

## In Scope

### Phase 1: Context-Aware Operations (Current Release)
- **Context Trigger**: Capture selected text or screen regions and process through Smart/Guided modes
- **Smart Dictation**: Voice-to-text transcription with intelligent formatting
- **Quick Tasks**: Customizable AI operations (default: Prompt Optimizer)
- **Result Management**: Copy, refine, insert, replace, and undo capabilities
- **Settings UI**: Comprehensive configuration including hotkeys, AI provider, privacy controls

### Phase 2: Advanced Interactions
- **Live Mode**: Real-time AI conversation with bounded tool access
- **Navigation Guidance**: Step-by-step contextual assistance
- **Take Action**: Browser automation for form filling and interactions
- **Memory System**: Persistent context across sessions

### Platform & Integration
- **Windows Platform**: WinUI 3 native application for Windows 10 2004+ and Windows 11
- **Cross-App Capture**: UI Automation and clipboard-based selection reading
- **Browser Bridge**: Chromium extension with authenticated native messaging
- **Global Hotkeys**: System-wide shortcuts with conflict detection

## Out of Scope

### Explicitly Not Included
- **Unrestricted Automation**: No autonomous computer control or unconfirmed destructive actions
- **Multi-Platform**: macOS and Linux support deferred to future releases
- **Offline Operation**: Requires internet connectivity for AI processing
- **Mobile Apps**: Focus remains on desktop productivity
- **Generic Chat Interface**: Not a ChatGPT replacement - focused on workflow integration
- **Direct File Manipulation**: Does not read/write user files without explicit action confirmation

### Deferred Features
- Firefox and Safari extension support
- Custom local model integration
- Voice output/TTS capabilities
- Cloud sync for settings and memories
- Team/enterprise features

## Success Criteria

### For Devpost Hackathon
1. **Functional completeness**: All Phase 1 features working end-to-end
2. **Clean installation**: One-click installer with proper Windows integration
3. **Usability**: Intuitive hotkeys and visual feedback
4. **Safety**: No exposure of API keys, proper error handling
5. **Documentation**: Clear README, setup instructions, and demo materials

### Technical Milestones
- ✅ Core context capture working across major applications
- ✅ AI provider integration (Claude, GPT, Gemini support)
- ✅ WinUI 3 overlay system with DPI awareness
- ✅ Browser extension and native host communication
- ✅ Persistent settings with credential protection
- ✅ Hotkey registration with conflict resolution
- ✅ Result panel with all manipulation operations
- ✅ Smart dictation with microphone management

### Quality Gates
- Clean build with no errors or warnings
- All unit and integration tests passing
- No exposed secrets or API keys in repository
- Installer produces working application
- Handles edge cases gracefully (no selection, app conflicts, network errors)

## Boundaries & Constraints

### Technical Boundaries
- **Windows Only**: Deep platform integration requires Windows APIs
- **x64 Architecture**: Native compilation target
- **.NET 8 SDK**: Required for build and runtime
- **Chromium Browsers**: Extension currently supports Chrome/Edge only
- **AI Provider**: Requires valid API key from supported providers

### Safety Boundaries
- Screen capture is explicit, never continuous without opt-in
- Browser actions require plan review and confirmation
- No system-level changes or file access without user approval
- Clipboard operations are isolated and temporary
- All network requests go through typed gateways

### Resource Constraints
- Memory footprint kept minimal for background operation
- API usage tracked to prevent runaway costs
- Network timeouts prevent hanging operations
- Cancellation available for all long-running tasks

## Timeline

### Hackathon Submission (October 26, 2026)
- Project repository cleaned and documented
- README with clear setup instructions
- Devpost Learn Skill Pack integrated
- Demo video prepared
- All required documentation complete

### Post-Hackathon Roadmap
- Code signing for installer
- Public release with download page
- Community feedback integration
- Feature refinement based on real-world usage

## Stakeholders

### Primary Users
- Knowledge workers who frequently interact with text content
- Developers who need quick code transformations
- Writers requiring fast editing and formatting
- Researchers managing information across applications

### Technical Reviewers
- Devpost hackathon judges
- Open source community
- Windows productivity tool users
- AI-powered workflow enthusiasts

## Risk Management

### Technical Risks
- **Hotkey conflicts**: Mitigated by conflict detection and fallback registration
- **Selection capture failures**: Multiple fallback strategies (UI Automation → clipboard)
- **API failures**: Retry logic and user-friendly error messages
- **Browser integration**: Typed contracts and verification steps

### Usability Risks
- **Learning curve**: Addressed with sensible defaults and guided mode
- **Visual clutter**: Minimal, transient UI that disappears after use
- **Performance**: Background operation optimized for responsiveness

## Acceptance Criteria

The project is considered complete and ready for Devpost submission when:

1. ✅ All Phase 1 features are implemented and tested
2. ✅ Repository contains no OpenAI branding or outdated references
3. ✅ README provides clear, accurate installation instructions
4. ✅ Devpost Learn Skill Pack is integrated and visible
5. ✅ Installer downloads from GitHub releases (not defunct website)
6. ✅ All documentation references correct repository name
7. ✅ No collaborator references from previous versions
8. ✅ Common, professional commit messages
9. ✅ Clean git history starting fresh for hackathon
10. ✅ All core workflows demonstrated in video

---

**Document Version**: 1.0  
**Last Updated**: October 25, 2026  
**Status**: Hackathon Submission Ready
