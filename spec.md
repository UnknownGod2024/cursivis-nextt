# Cursivis Next - Technical Specification

## Architecture Overview

Cursivis Next follows a layered architecture with clear dependency rules:

```
┌─────────────────────────────────────────────────────────────┐
│                      WinUI 3 Application                     │
│  (Overlays, Settings, Controllers, XAML Views)              │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────┐
│              Windows Platform Layer                         │
│  (Hotkeys, UI Automation, Audio, Overlays, Clipboard)      │
└────────────────┬────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────┐
│              Application Services                           │
│  (Context Trigger, Live Mode, Dictation, Actions)          │
└─────┬──────────────────────────────────────────────┬────────┘
      │                                              │
┌─────▼──────────────────┐             ┌────────────▼─────────┐
│   Infrastructure       │             │      Domain          │
│  (AI, Storage, Bridge) │             │  (Settings, Models)  │
└────────────────────────┘             └──────────────────────┘
```

**Dependency Rule**: Inner layers (Domain) do not depend on outer layers (Infrastructure, Platform, UI).

## Technology Stack

### Core Platform
- **.NET 8.0**: Runtime and SDK
- **WinUI 3**: Native Windows UI framework
- **C# 12**: Language features (records, pattern matching, nullable references)
- **Windows 10 SDK**: Platform APIs (UI Automation, Hotkeys, Audio)

### Key NuGet Packages
- `Microsoft.WindowsAppSDK`: WinUI 3 runtime
- `Microsoft.Extensions.DependencyInjection`: IoC container
- `System.Text.Json`: Settings serialization
- `xunit`: Testing framework

### Browser Integration
- **Chromium Manifest V3 Extension**: Browser action handler
- **Native Messaging Protocol**: Chrome/Edge to native host communication
- **Named Pipes**: Authenticated inter-process communication

### AI Integration
- **HTTP APIs**: AI provider REST endpoints (Claude, GPT, Gemini)
- **WebSockets**: Realtime bidirectional communication (Live Mode)
- **Structured Outputs**: JSON-based typed responses

## Project Structure

```
cursivis-nextt/
├── apps/
│   └── windows/
│       ├── Cursivis.Windows.App/           # WinUI 3 application
│       │   ├── Controllers/                # Feature controllers
│       │   ├── Overlays/                   # Custom overlay windows
│       │   ├── Pages/                      # Settings pages
│       │   └── App.xaml.cs                 # Application entry
│       ├── Cursivis.Windows.Platform/      # Windows-specific services
│       │   ├── Hotkeys/                    # Global shortcut management
│       │   ├── Selection/                  # Text capture (UI Automation)
│       │   ├── Audio/                      # Microphone/speaker
│       │   └── Security/                   # Credential storage
│       └── Cursivis.Windows.NativeHost/    # Browser native messaging host
├── src/
│   ├── Cursivis.Domain/                    # Core domain models
│   │   ├── Settings/                       # Configuration types
│   │   ├── QuickTasks/                     # Task definitions
│   │   └── Actions/                        # Action contracts
│   ├── Cursivis.Application/               # Business logic
│   │   ├── Context/                        # Context Trigger service
│   │   ├── Dictation/                      # Smart Dictation
│   │   ├── Realtime/                       # Live Mode
│   │   ├── Actions/                        # Take Action orchestration
│   │   └── QuickTasks/                     # Quick Task execution
│   ├── Cursivis.Infrastructure.OpenAI/     # AI provider gateways
│   │   ├── OpenAiResponsesGateway.cs       # Chat completions
│   │   ├── OpenAiRealtimeGateway.cs        # Live conversation
│   │   └── OpenAiTranscriptionGateway.cs   # Voice-to-text
│   ├── Cursivis.Infrastructure.Storage/    # Persistence layer
│   │   └── ApplicationSettingsStore.cs     # JSON settings file
│   └── Cursivis.Infrastructure.BrowserBridge/  # Extension communication
├── tests/
│   ├── Cursivis.UnitTests/                 # Fast isolated tests
│   └── Cursivis.IntegrationTests/          # Component integration tests
├── extensions/
│   └── chromium/                           # Manifest V3 extension
│       ├── manifest.json
│       ├── background.js                   # Service worker
│       └── content.js                      # Page script
├── installer/                              # Inno Setup script
├── docs/                                   # Architecture & product docs
├── scripts/                                # Build & release automation
└── schemas/                                # JSON contract schemas
```

## Domain Layer

### Core Entities

#### ApplicationSettings
```csharp
public sealed record ApplicationSettings
{
    public string ApiKey { get; init; }
    public Dictionary<string, string> Hotkeys { get; init; }
    public string QuickTaskInstruction { get; init; }
    public bool LaunchAtSignIn { get; init; }
    public bool NavigationGuidanceEnabled { get; init; }
    // ... other settings
}
```

#### QuickTaskDefinition
```csharp
public sealed record QuickTaskDefinition
{
    public string Id { get; init; }
    public string DisplayName { get; init; }
    public string Instruction { get; init; }
}
```

#### ActionContract
```csharp
public interface IActionPlan
{
    string Id { get; }
    RiskLevel Risk { get; }
    IReadOnlyList<IActionStep> Steps { get; }
}
```

### Value Objects
- `HotkeyMapping`: Configured vs active shortcut
- `CapturedContext`: Selected text + source identity
- `OperationResult`: Success/failure with typed data
- `RiskLevel`: Safe, RequiresConfirmation, Denied

## Application Layer

### Context Trigger Service

**Responsibilities**:
- Orchestrate capture → analysis → execution → result flow
- Manage operation state (prevents stale async work from overwriting)
- Route between Smart Mode (automatic) and Guided Mode (user choice)

**Key Methods**:
```csharp
public interface IContextTriggerService
{
    Task<OperationResult> ExecuteSmartModeAsync(CapturedContext context, CancellationToken ct);
    Task<IReadOnlyList<GuidedOption>> GetGuidedOptionsAsync(CapturedContext context, CancellationToken ct);
    Task<OperationResult> ExecuteGuidedOptionAsync(string optionId, CapturedContext context, CancellationToken ct);
}
```

**State Management**:
- Single `InteractionState` model tracks current operation
- Each operation assigned unique ID
- Results checked against current operation ID before display

### Smart Dictation Service

**Flow**:
1. Acquire managed audio session
2. Start recording with visual feedback
3. Stream audio to transcription API
4. Clean and format transcript
5. Attempt insertion at original target
6. Fallback to clipboard if insertion unavailable
7. Release audio resources cleanly

**Cancellation**: User can press Ctrl+Alt+Escape or cancel button to abort recording immediately.

### Live Mode Service

**Architecture**:
- Duplex WebSocket connection to Realtime API
- Separate audio input/output streams
- Event-driven: ConnectionOpened, UserTranscript, AssistantTranscript, ToolCallRequested, Error
- Bounded tools: RefreshSelection, AnalyzeScreenRegion, StartNavigationGuidance, SafeAction

**Tool Validation**:
```csharp
public interface ILiveModeToolExecutor
{
    Task<ToolResult> ExecuteAsync(string toolName, JsonElement arguments, CancellationToken ct);
}
```

Each tool validates arguments locally before execution.

### Take Action Service

**Flow**:
1. User selects content and target action
2. Generate ActionPlan via AI (structured output)
3. Classify risk level for each step
4. Display plan in review UI
5. User confirms or cancels
6. Send to browser via native messaging bridge
7. Execute steps with verification
8. Report success/failure for each step

**Risk Classification**:
- **Safe**: Text input, navigation, reading
- **RequiresConfirmation**: Form submission, account changes
- **Denied**: Purchases, deletions, irreversible operations

## Infrastructure Layer

### AI Gateways

#### Responses Gateway (Chat Completions)
```csharp
public interface IOpenAiResponsesGateway
{
    Task<string> GetCompletionAsync(string systemPrompt, string userMessage, CancellationToken ct);
    Task<T> GetStructuredOutputAsync<T>(string systemPrompt, string userMessage, CancellationToken ct);
}
```

**Features**:
- Retry logic for transient failures
- Timeout handling (30s default)
- Error classification (user-facing messages)
- Model selection based on task

#### Realtime Gateway (Live Mode)
```csharp
public interface IOpenAiRealtimeGateway
{
    Task ConnectAsync(CancellationToken ct);
    Task SendAudioAsync(byte[] audioData, CancellationToken ct);
    IObservable<RealtimeEvent> Events { get; }
    Task DisconnectAsync();
}
```

#### Transcription Gateway (Dictation)
```csharp
public interface IOpenAiTranscriptionGateway
{
    Task<string> TranscribeAsync(Stream audioStream, CancellationToken ct);
}
```

### Storage

#### Settings Persistence
- **Location**: `%LOCALAPPDATA%\Cursivis\settings.json`
- **Format**: JSON with versioned schema
- **Atomic Writes**: Write to `.tmp` → verify → rename
- **Migration**: Detect old schema, upgrade on load

#### Credential Storage
- **Windows DPAPI**: User-level data protection
- **No Plaintext**: API keys encrypted at rest
- **Access**: Only current Windows user can decrypt

### Browser Bridge

#### Architecture
1. **Chromium Extension**: Receives user's intent, forwards to native host
2. **Native Messaging Host** (`Cursivis.Windows.NativeHost.exe`): Validates extension ID, forwards to app
3. **Named Pipe Server**: Authenticated by Windows user SID
4. **Main App**: Receives typed contracts, executes with confirmation

**Security**:
- Extension ID allowlisted in native host
- Pipe accessible only to same user
- All messages typed and validated
- No generic command execution

#### Message Flow
```
Browser Page
    ↓ (postMessage)
Extension Content Script
    ↓ (chrome.runtime.sendMessage)
Extension Background (Service Worker)
    ↓ (chrome.runtime.sendNativeMessage)
Native Messaging Host
    ↓ (Named Pipe)
Main App Bridge Server
    ↓ (Typed Contract)
BrowserTakeActionService
```

## Windows Platform Layer

### Hotkey Management

**Registration**:
- Uses `RegisterHotKey` Win32 API
- Must run on WinUI window thread (WM_HOTKEY messages)
- Transactional: register all or rollback

**Conflict Detection**:
```csharp
public interface IHotkeyService
{
    Task<HotkeyRegistrationResult> RegisterAsync(string id, HotkeyMapping mapping);
    Task UnregisterAsync(string id);
    IObservable<HotkeyPressed> HotkeyEvents { get; }
}
```

**Result Types**:
- `Success`: Registered as configured
- `ConflictDetected`: Another app owns it, registered fallback
- `Failed`: No suitable shortcut available

### Selection Capture

**Multi-Strategy Approach**:

1. **UI Automation** (Preferred)
   - Query focused element for `SelectionPattern`
   - Read `ITextProvider` selection range
   - Works for: WPF, UWP, WinUI, some Win32

2. **Protected Clipboard Fallback**
   - Snapshot current clipboard formats
   - Send Ctrl+C to foreground window
   - Wait for clipboard sequence change
   - Read new content
   - Restore original clipboard only if no user intervention detected

3. **Safeguards**
   - Reject URL-like content from browser chrome (omnibox)
   - Cross-check UA selection vs clipboard for browsers
   - Preserve target identity for insertion

### Audio Management

**Microphone Capture**:
- Uses Windows Core Audio APIs
- Exclusive mode acquisition
- Real-time buffer delivery
- Cancellable via disposed handle

**Speaker Playback** (Live Mode):
- Queue-based with overflow protection
- Synchronized with transcript events
- Interrupt support (user can speak over assistant)

### Overlay Windows

**Custom Non-Client Regions**:
```csharp
public sealed class CursivisOverlayWindow : Window
{
    // Transparent, non-activating, always-on-top
    // DPI-aware rounded clipping
    // Drag via custom non-client hit test
    // Persists position across sessions
}
```

**State Visualization**:
- **Context Orb**: Compact status indicator near selection
- **Result Panel**: Resizable output with manipulation buttons
- **Live Orb**: Circular orb with recording/thinking/speaking states
- **Transcript Panel**: Optional, toggleable, persisted preference

## Data Flow Examples

### Example 1: Context Trigger - Smart Mode

```
User selects text in VS Code
    ↓
Presses Ctrl+Alt+O
    ↓
HotkeyService raises HotkeyPressed event
    ↓
ContextTriggerController.OnContextTriggerAsync()
    ↓
SelectionCaptureService.CaptureAsync()
    ├─ Try UI Automation
    └─ Fallback to protected clipboard
    ↓
Returns CapturedContext { Text, TargetIdentity }
    ↓
ContextTriggerService.ExecuteSmartModeAsync(context)
    ├─ Classify intent (summarize, explain, format, etc.)
    ├─ Call OpenAiResponsesGateway.GetCompletionAsync()
    └─ Return OperationResult
    ↓
ResultPanelController.ShowResult(result, context)
    ↓
User clicks "Replace"
    ↓
InsertionService.ReplaceAsync(context.TargetIdentity, result.Text)
    ├─ Focus original window
    ├─ Send selection + Delete via UI Automation
    └─ Insert new text
    ↓
Result Panel dismissed
```

### Example 2: Smart Dictation

```
User places cursor in email compose
    ↓
Presses Ctrl+Alt+U
    ↓
SmartDictationService.StartAsync()
    ↓
AudioCaptureService.AcquireSession()
    ↓
Visual feedback: recording orb appears
    ↓
User speaks: "Hi team, quick update..."
    ↓
Audio buffered and streamed
    ↓
On completion (silence detected or manual stop):
OpenAiTranscriptionGateway.TranscribeAsync(audioStream)
    ↓
Returns formatted transcript
    ↓
InsertionService.InsertAtCursorAsync(transcript)
    ↓
AudioCaptureService.ReleaseSession()
    ↓
Visual feedback dismissed
```

### Example 3: Take Action - Browser Form Fill

```
User selects: "Name: Alex\nEmail: alex@example.com"
    ↓
Presses Ctrl+Alt+I
    ↓
TakeActionController.OnTakeActionAsync()
    ↓
BrowserTakeActionService.GeneratePlanAsync(context)
    ├─ Call OpenAiResponsesGateway.GetStructuredOutputAsync<ActionPlan>()
    └─ Returns ActionPlan with steps
    ↓
ActionPlanReviewWindow.Show(plan)
    ↓
User reviews and clicks "Execute"
    ↓
BrowserBridgeClient.SendActionPlanAsync(plan)
    ├─ Message serialized to JSON
    ├─ Sent via named pipe to extension
    └─ Extension receives via native messaging
    ↓
Extension content script:
    ├─ Find field matching "Name"
    ├─ Fill with "Alex"
    ├─ Verify value set correctly
    ├─ Repeat for Email field
    └─ Report success/failure per step
    ↓
BrowserBridgeClient receives verification
    ↓
Result displayed: "2 fields filled successfully"
```

## Testing Strategy

### Unit Tests (Cursivis.UnitTests)
- **Target**: Application and Domain layers
- **Mocks**: All external dependencies (gateways, platform services)
- **Coverage**: Business logic, state management, validation, error handling
- **Speed**: < 1s total execution

**Example**:
```csharp
[Fact]
public async Task SmartMode_WithSummarizationIntent_CallsGatewayWithCorrectPrompt()
{
    var mockGateway = new Mock<IOpenAiResponsesGateway>();
    var service = new ContextTriggerService(mockGateway.Object);
    var context = new CapturedContext { Text = "Long article text..." };

    await service.ExecuteSmartModeAsync(context, CancellationToken.None);

    mockGateway.Verify(g => g.GetCompletionAsync(
        It.Is<string>(s => s.Contains("summarize")),
        It.IsAny<string>(),
        It.IsAny<CancellationToken>()));
}
```

### Integration Tests (Cursivis.IntegrationTests)
- **Target**: Cross-layer interactions
- **Real Components**: Storage, some platform APIs
- **Deterministic**: No actual AI calls (use test gateways)
- **Scenarios**: Hotkey registration/rollback, settings persistence, audio session lifecycle

**Example**:
```csharp
[Fact]
public async Task HotkeyService_WhenConflictDetected_RegistersFallback()
{
    var service = new WindowsHotkeyService();
    
    // Primary conflicts
    var result = await service.RegisterAsync("test", new HotkeyMapping 
    { 
        Primary = "Ctrl+C", // known conflict
        Fallback = "Ctrl+Shift+C" 
    });

    Assert.Equal(HotkeyResult.ConflictDetected, result.Status);
    Assert.Equal("Ctrl+Shift+C", result.ActiveMapping);
}
```

### Manual Testing Checklist
- [ ] Context Trigger works in: Notepad, VS Code, Chrome, Word
- [ ] Smart Dictation inserts correctly in: email, notepad, browser input
- [ ] Live Mode: duplex conversation, tool calls work
- [ ] Take Action: form filling in Google Forms, simple survey
- [ ] Hotkeys: all shortcuts registered, conflict resolution works
- [ ] Settings: persist across restart, API key test succeeds
- [ ] Multi-monitor: overlays appear on correct screen
- [ ] High DPI: UI scales properly on 150%, 200% displays

## Build & Deployment

### Build Process
```powershell
# Restore with locked dependencies
dotnet restore Cursivis.sln --locked-mode -r win-x64 -p:Platform=x64

# Build Release configuration
dotnet build Cursivis.sln -c Release -p:Platform=x64 --no-restore

# Run tests
dotnet test tests/Cursivis.UnitTests/*.csproj -c Release -p:Platform=x64 --no-build
dotnet test tests/Cursivis.IntegrationTests/*.csproj -c Release -p:Platform=x64 --no-build

# Publish self-contained app
dotnet publish apps/windows/Cursivis.Windows.App/*.csproj -c Release -p:Platform=x64 -r win-x64 --self-contained
```

### Installer (Inno Setup)
- Copies published app binaries
- Registers native messaging host manifest
- Creates Start Menu shortcut
- Optional: Desktop shortcut, launch at sign-in
- Uninstaller removes all components cleanly

**Output**: `Cursivis-Setup-{version}-x64.exe`

### CI/CD (GitHub Actions)
```yaml
on: [push]
jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'
      - run: dotnet restore --locked-mode
      - run: dotnet build -c Release
      - run: dotnet test -c Release --no-build
      - run: scripts/verify-extension.ps1
      - run: scripts/check-secrets.ps1
```

## Security Considerations

### API Key Protection
- **Storage**: Windows DPAPI (user-scoped)
- **Transmission**: Only to official AI provider endpoints
- **Logging**: Never in logs, diagnostics, or error messages
- **Testing**: Use `.env` only for local dev, never committed

### Browser Bridge Security
- **Authentication**: Named pipe restricted to current user SID
- **Authorization**: Extension ID allowlist in native host
- **Validation**: All messages typed and validated before execution
- **Isolation**: No generic command execution

### Screen Capture
- **Explicit**: User must trigger region selector
- **Opt-in**: Navigation Guidance requires privacy setting enabled
- **No Ambient**: Never captures without user action

### Action Safety
- **Risk Tiers**: Safe (auto), RequiresConfirmation (manual), Denied (blocked)
- **Verification**: Each action step verified post-execution
- **Cancellation**: User can stop execution at any time

## Performance Optimization

### Startup Time
- Lazy-load heavy dependencies (AI gateways)
- Settings loaded asynchronously
- Overlays created on-demand, not at startup

### Memory Management
- Dispose audio sessions immediately after use
- Release AI gateway HTTP clients on shutdown
- Cached overlay positions, not full window state

### Network Efficiency
- Streaming for long-form outputs (not blocking on full response)
- Cancellation propagated to HTTP requests
- Retry with exponential backoff for transient failures

## Error Handling

### User-Facing Errors
- **No Selection**: "Please select text or press hotkey again to capture a screen region"
- **API Failure**: "Could not reach AI provider. Check your internet connection and API key"
- **Hotkey Conflict**: "Ctrl+Alt+O is used by [App]. Using fallback Ctrl+Shift+Alt+O"
- **Insertion Failed**: "Could not insert into original app. Result copied to clipboard"

### Developer Errors
- Logged to `%LOCALAPPDATA%\Cursivis\logs\`
- Redacted: API keys, selected content (unless diagnostic mode enabled)
- Includes: timestamps, operation IDs, stack traces, platform info

## Open Questions & Future Work

### Deferred Decisions
- Local model integration (architecture prepared, not implemented)
- Voice output (TTS) for Live Mode responses
- Multi-monitor selection capture (currently uses primary monitor)
- Custom action template editor (currently plain text)

### Extensibility Points
- `IAiProvider` interface allows swapping AI backends
- `IActionStep` allows custom browser action types
- QuickTask system supports user-defined operations
- Settings schema versioning supports migrations

---

**Document Version**: 1.0  
**Last Updated**: October 25, 2026  
**Technical Lead**: Devpost Hackathon Submission  
**Status**: Implementation Complete
