## 🚧 Status: Active Development

> ⚠️ **Heads up before you dive in.**

ApexUIBridge is fully functional and suitable for integration and testing, but it is not yet a finished product. Core features work as documented, and the tool is built with production use as the goal.

A few things to be aware of:

- **🔧 Bundled libraries have been modified.** The included versions of [LlamaSharp](https://github.com/SciSharp/LLamaSharp) and [FlaUI](https://github.com/FlaUI/FlaUI) contain targeted patches for stability. They are not vanilla upstream releases. If you replace them with stock versions, the application could become unstable.
- **⚠️ APIs and behavior may change.** Interfaces, configuration, and output formats are still being refined. Breaking changes can occur between commits.
- **🚧 Not hardened for production.** Some timing and resource management scenarios are still being refined. Use accordingly.

Bugs, unexpected behavior, and rough edges should be reported — that feedback is what moves this forward.

---

<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/45a55136-6369-4434-aeca-3b00e6bcc699" />


---

# ApexUIBridge

**ApexUIBridge** is a Windows UI automation framework for autonomous AI agents, built on top of
[FlaUI](https://github.com/FlaUI/FlaUI) (a managed wrapper around the Windows UI Automation
API), with an integrated AI-assisted command workflow for exploring, describing, and
interacting with external application UIs.

![ApexUIBridge](ApexUIBridge.png)

---

## Token Economics

Map rendering isn't just a debugging convenience — it has compounding implications for token consumption at scale.

### The Core Difference

With screen-capture-based AI automation, every interaction requires sending a fresh image to the model. At typical resolutions that's **2,000–30,000+ tokens per capture** — every single time, for every action. With ApexUIBridge's map approach, the UI is rendered once as a structured, text-based representation. After that initial render, each individual interaction references elements by name, costing **5–20 tokens on average** — comparable to the overhead of a single API tool call.

The initial map render is a one-time cost per session. Everything after it is nearly free by comparison.

### Assumptions Used Below

| | Screen Capture | Map Approach |
|---|---|---|
| Per-interaction cost | 2,500–10,000 tokens (image) | 5–20 tokens (text reference) |
| Session setup cost | none — image sent every time | 400–1,800 tokens (one-time map render) |
| Interactions per person/day | 100 | 100 |

---

### Example 1 — Small App *(Calculator, tray utility, simple tool)*

> Screenshot: **2,500 tokens each** · Initial map: **400 tokens** · Per-action after map: **8 tokens**

**By time period — 1 person:**

| Timeframe | Screen Capture | Map Approach | Tokens Saved |
|---|---|---|---|
| 1 day | 250,000 | 1,192 | 248,808 |
| 1 week | 1,750,000 | 8,344 | 1,741,656 |
| 1 year | 91,250,000 | 435,080 | 90,814,920 |

**Annual totals — by team size:**

| Team Size | Screen Capture | Map Approach | Reduction Factor |
|---|---|---|---|
| 1 person | 91,250,000 | 435,080 | **~210x** |
| 10 people | 912,500,000 | 4,350,800 | **~210x** |
| 50 people | 4,562,500,000 | 21,754,000 | **~210x** |

---

### Example 2 — Medium App *(File Explorer, settings panel, line-of-business app)*

> Screenshot: **10,000 tokens each** · Initial map: **1,800 tokens** · Per-action after map: **12 tokens**

**By time period — 1 person:**

| Timeframe | Screen Capture | Map Approach | Tokens Saved |
|---|---|---|---|
| 1 day | 1,000,000 | 2,988 | 997,012 |
| 1 week | 7,000,000 | 20,916 | 6,979,084 |
| 1 year | 365,000,000 | 1,090,620 | 363,909,380 |

**Annual totals — by team size:**

| Team Size | Screen Capture | Map Approach | Reduction Factor |
|---|---|---|---|
| 1 person | 365,000,000 | 1,090,620 | **~335x** |
| 10 people | 3,650,000,000 | 10,906,200 | **~335x** |
| 50 people | 18,250,000,000 | 54,531,000 | **~335x** |

---

> **Note:** These figures assume 100 AI-driven UI interactions per person per day and one map render per session. High-DPI or 4K displays push screen capture costs toward the upper end of the range, widening the gap further. Map size scales with UI complexity, not screen resolution, so it grows much more predictably.

---

## Example UI Map

[Complete map of the calculator](complete_window_map_of_calc.json)

Sample json from a UI map:
```json
{
  "elementId": 643244285,
  "name": "Calculator",
  "controlType": "Window",
  "boundingRectangle": {
    "x": 875,
    "y": 22,
    "width": 228,
    "height": 323
  },
  "details": {
    "Identification": {
      "AutomationId": "Not Supported",
      "Name": "Calculator",
      "ClassName": "CalcFrame",
      "ControlType": "Window",
      "LocalizedControlType": "window",
      "FrameworkType": "Win32",
      "FrameworkId": "Win32",
      "ProcessId": "29148"
    },
    "Details": {
      "IsEnabled": "True",
      "IsOffscreen": "False",
      "BoundingRectangle": "{X=875,Y=22,Width=228,Height=323}",
      "HelpText": "",
      "IsPassword": "False",
      "NativeWindowHandle": "22677858 (015A0962)"
    },
```

---

## Map Rendering

<img width="287" height="343" alt="window_render" src="https://github.com/user-attachments/assets/7bc628bb-d905-4ab3-992f-507b004263a2" />

Map rendering generates a spatial, labeled wireframe of a target window's UI automation tree. Each element — buttons, menus, text fields — is drawn as a bordered, labeled block positioned to reflect its actual on-screen bounds.

The AI agent uses this view when it needs visual confirmation of the UI structure — not a screenshot, but a structured, element-level representation. When standard UIA interaction is insufficient, the map gives the agent a spatial reference for layout, hierarchy, and available controls.

The agent uses it to:
- **Verify element detection** — confirm that controls are present and correctly identified before acting
- **Diagnose interaction failures** — determine which elements are visible to the automation layer when an action doesn't behave as expected
- **Resolve ambiguity** — when the element tree alone is unclear, the map provides spatial context to make the correct decision

---

## What it does

| Capability | Description |
|---|---|
| **Process / window listing** | Enumerates all top-level desktop windows, filterable by title, PID, or element ID. |
| **UI tree inspection** | Builds a lazy-loading element tree for any selected window; shows name, control type, automation ID, bounding rectangle, and all supported UIA patterns. |
| **Direct actions** | Click, double-click, right-click, type, set value, scroll, drag, toggle, expand/collapse, focus, highlight, window minimize/maximize/restore/close. |
| **Screenshot capture** | Captures any element or full window to PNG. |
| **Export** | Copies the tree as JSON or XML; exports element details as XML. |
| **UI map rendering** | Renders a PNG diagram of all element bounding rectangles overlaid on the screen canvas. |
| **AI-assisted automation** | Sends prompts to Anthropic (cloud), LlamaSharp Chat (local GGUF), or LlamaSharp Instruct (local GGUF); optionally auto-executes `[CMD: …]` Bridge commands parsed from the AI response. |
| **Vision describe** | Captures an element screenshot and sends it to an LM Studio vision endpoint for natural-language description. |
| **Menu tree export** | Traverses a target application's menu bar and returns the full menu hierarchy as JSON. |

---

## Platform requirements

- **OS**: Windows 10 (build 19041) or newer
- **Runtime**: .NET 10 (`net10.0-windows10.0.19041`)
- **Architecture**: x64 (LlamaSharp native libraries require it)
- UI Automation requires the target application to expose UIA properties. Reliability
  varies by framework (WPF, WinForms, Win32, Electron/CEF).

---

## Build and run

```bash
# Build the full solution (includes FlaUI and LlamaSharp local projects)
dotnet build ApexUIBridge.sln

# Run the main app
dotnet run --project ApexUIBridge/ApexUIBridge/ApexUIBridge.csproj
```

> **Note**: The app must run on Windows. It opens a WinForms window — do not run
> headlessly.

---

## Configuration

### `appsettings.json` (next to the executable)

Controls theme and the three visual overlay styles used during hover, selection, and
pick-mode:

```json
{
  "Theme": "Light",
  "HoverOverlay":     { "Size": 2, "Margin": "0", "OverlayColor": "#FF00FF00", "OverlayMode": "Border" },
  "SelectionOverlay": { "Size": 2, "Margin": "0", "OverlayColor": "#FF0000FF", "OverlayMode": "Border" },
  "PickOverlay":      { "Size": 2, "Margin": "0", "OverlayColor": "#FFFF0000", "OverlayMode": "Border" }
}
```

`OverlayMode` is `"Border"` (four thin strips) or `"Fill"` (solid semi-transparent rect).
`OverlayColor` is an ARGB hex string — the alpha byte controls opacity.

### AI settings (`%AppData%\ApexUIBridge\ai-settings.json`)

Persisted automatically by the app. Key fields:

| Field | Default | Description |
|---|---|---|
| `Provider` | `"LlamaSharp Instruct"` | `"Anthropic"`, `"LlamaSharp (Local)"`, or `"LlamaSharp Instruct"` |
| `AnthropicApiKey` | `""` | Anthropic API key (also read from `ANTHROPIC_API_KEY` env var) |
| `AnthropicModel` | `"claude-haiku-4-5-20251001"` | Model ID for Anthropic requests |
| `ModelPath` | `""` | Absolute path to a local `.gguf` model file |
| `Temperature` | `0.8` | Sampling temperature |
| `MaxTokens` | `2048` | Max generation tokens per turn |
| `ContextSize` | `4096` | KV-cache size (LlamaSharp) |
| `GpuLayers` | `10` | GPU offload layers (LlamaSharp) |
| `AutoExec` | `false` | Auto-execute `[CMD: …]` commands from AI responses |
| `ShowThinking` | `false` | Show extended-thinking tokens in the chat panel |
| `SystemPrompt` | (built-in) | Override the default command-oriented system prompt |

---

## Project layout

```
ApexUIBridge.sln
├── ApexUIBridge/ApexUIBridge/          ← Main WinForms application (this document)
│   ├── Program.cs                      ← Entry point, DI setup, LlamaSharp init
│   ├── App.xaml.cs                     ← Static app container: Services, Logger, overlays
│   ├── appsettings.json                ← Theme and overlay defaults
│   ├── ElementRecord.cs                ← Snapshot record + ElementFindCriteria
│   │
│   ├── Forms/
│   │   ├── StartupForm.cs              ← Main window (logic)
│   │   ├── StartupForm.Designer.cs     ← Auto-generated WinForms layout
│   │   └── AiSettingsDialog.cs         ← AI settings dialog (code-built, no designer)
│   │
│   ├── ViewModels/
│   │   ├── StartupViewModel.cs         ← Process list, pick-mode mouse hook
│   │   ├── ProcessViewModel.cs         ← Tree build, hover/focus/highlight modes, export
│   │   ├── ElementViewModel.cs         ← Single element wrapper + ID generation + details
│   │   ├── DetailGroupViewModel.cs     ← Property group for the details panel
│   │   ├── PatternItem.cs              ← Key-value pair for a pattern property
│   │   ├── SettingsViewModel.cs        ← Thin wrapper for the settings dialog
│   │   └── AboutViewModel.cs
│   │
│   ├── Core/
│   │   ├── Bridge.cs                   ← Command-first automation façade (see command table)
│   │   ├── ElementOperations.cs        ← Low-level UIA interactions (click, type, scroll…)
│   │   ├── ElementIdGenerator.cs       ← SHA-256 hash → deterministic integer element ID
│   │   ├── ElementRegistry.cs          ← Global WeakRef map: ElementId → ElementViewModel
│   │   ├── ElementOverlay.cs           ← Screen overlay renderer (border / fill modes)
│   │   ├── HoverManager.cs             ← 300 ms poll: Ctrl+hover element detection
│   │   ├── FocusTrackingMode.cs        ← UIA focus-changed event subscriber
│   │   ├── GlobalMouseHook.cs          ← Low-level WH_MOUSE_LL hook helper
│   │   ├── ElementFinderService.cs     ← Window/element search with fuzzy matching
│   │   ├── ElementFinderResult.cs      ← Result type for finder queries
│   │   ├── ElementSearchFilter.cs      ← Include/exclude filter for finder queries
│   │   ├── UiMapRenderer.cs            ← Renders element bounding boxes to a PNG
│   │   ├── PatternItemsFactory.cs      ← Builds PatternItem arrays from UIA patterns
│   │   ├── AutomationError.cs          ← Typed automation error enum
│   │   ├── CommandResult.cs            ← Bridge command result (success, message, data)
│   │   ├── OperationResult.cs          ← Low-level operation result
│   │   ├── ObservableObject.cs         ← INotifyPropertyChanged base
│   │   ├── ExtendedObservableCollection.cs
│   │   ├── RelayCommand.cs / RelayCommandAsync.cs
│   │   ├── Editable.cs
│   │   ├── IDialogViewModel.cs
│   │   ├── Extensions/
│   │   │   ├── AutomationPropertyExtensions.cs
│   │   │   ├── StringExtensions.cs
│   │   │   └── TaskExtensions.cs
│   │   ├── Exporters/
│   │   │   ├── ITreeExporter.cs
│   │   │   ├── IElementDetailsExporter.cs
│   │   │   ├── JsonTreeExporter.cs     ← Full tree → indented JSON
│   │   │   ├── XmlTreeExporter.cs      ← Full tree → XML
│   │   │   └── XmlElementDetailsExporter.cs ← Single element details → XML
│   │   └── Logger/
│   │       ├── ILogger.cs
│   │       ├── InternalLogger.cs
│   │       ├── InternalLoggerMessage.cs
│   │       ├── LogLevel.cs
│   │       └── LoggerExtensions.cs
│   │
│   ├── Models/
│   │   ├── AiSettings.cs               ← Persisted AI/provider configuration
│   │   ├── Element.cs
│   │   └── ElementPatternItem.cs       ← UI model for the patterns grid
│   │
│   ├── Settings/
│   │   ├── FlaUiAppSettings.cs         ← JSON-backed app settings (theme + overlays)
│   │   ├── FlaUiAppOptions.cs          ← Runtime overlay factory delegates
│   │   ├── OverlaySettings.cs          ← Per-overlay JSON schema
│   │   ├── JsonSettingsService.cs      ← Generic JSON read/write service
│   │   ├── ISettingsService.cs
│   │   └── ISettingViewModel.cs
│   │
│   ├── Anthropic/                      ← Cloud AI provider
│   │   ├── AnthropicClient.cs          ← HTTP client (streaming + single-shot)
│   │   ├── MessageRequest.cs / MessageResponse.cs
│   │   ├── Message.cs
│   │   ├── IMessageContent.cs
│   │   ├── TextContent.cs / ImageContent.cs
│   │   └── StreamingEvents.cs
│   │
│   ├── LMStudio/                       ← Vision describe endpoint
│   │   └── LMStudioClient.cs           ← Chat-completions client with image support
│   │
│   ├── LlamaSharpAI/                   ← Local GGUF inference
│   │   ├── LlamaSharpClient.cs         ← Chat session (streaming, anti-prompts)
│   │   ├── LlamaSharpInstructClient.cs ← Instruct/completion session
│   │   └── UserSettings.cs
│   │
│   └── OldCodeForReference/            ← Archived tree walker experiments (not in active use)
│       ├── FilteredTreeWalker.cs
│       └── TreeWalkerFilterLists.cs
│
├── FlaUI.Core/                         ← FlaUI source project (local reference)
├── FlaUI.UIA2/                         ← UIA2 automation backend
├── FlaUI.UIA3/                         ← UIA3 automation backend (used by the app)
├── LLamaSharp/                         ← LlamaSharp source project (local reference)
└── TestApplications/                   ← Standalone target apps for Bridge testing
    ├── WinFormsApplication/            ← Comprehensive WinForms control surface
    ├── WpfApplication/                 ← Comprehensive WPF control surface
    └── MenuTestApp/                    ← Focused WinForms menu-hierarchy test app
```

---

## Test applications

Three standalone Windows applications are included under `TestApplications/` to provide a
known, repeatable UI surface for validating Bridge commands and element ID stability.

### WinFormsApplication

A comprehensive WinForms host covering every common control type:

| Group | Controls |
|---|---|
| Buttons | Two push buttons |
| CheckBoxes | Simple checkbox, three-state checkbox |
| RadioButtons | Two mutually exclusive radio buttons |
| ComboBoxes | Editable combo, non-editable combo (Item 4 triggers a MessageBox) |
| TextBoxes | TextBox, PasswordBox |
| Numeric / Range | Slider, Spinner (NumericUpDown), ProgressBar |
| Lists | ListBox, ListView, TreeView, DataGridView (data-bound) |
| Tabs | TabControl with two pages |
| Menus | MenuBar → File, Edit |
| Other | StatusBar, DateTimePicker |

Element IDs for a scanned `WinFormsApplication` session are captured in
`Core/WinFormsTestAppElements.cs`.

### WpfApplication

A comprehensive WPF host with three tabs:

**Simple Controls tab**
- TextBox, PasswordBox
- Editable ComboBox, non-editable ComboBox (Item 4 triggers a MessageBox)
- ListBox
- Simple CheckBox, three-state CheckBox
- Two RadioButtons
- ProgressBar, Slider
- Button with ContextMenu (Context 1, Context 2 → Inner Context)
- InvokableButton (bound to `ICommand`)
- ScrollViewer with oversized button (scroll test)
- Two ToggleButton/Popup pairs
- Checkable menu item (`Show Label`) that toggles a visible Label

**Complex Controls tab**
- TreeView (three levels deep)
- ListView with Key/Value columns
- Expander with three checkboxes
- Large scrollable ListView
- DataGrid (data-bound via `MainViewModel`)

**More Controls tab**
- Calendar (multi-range selection)
- DatePicker
- Large multi-select ListBox (7 items)

Menu bar: File → Exit; Edit → Copy (Plain / Fancy), Paste, Show Label (checkable)

Element IDs for a scanned `WpfApplication` session are captured in
`Core/WpfTestAppElements.cs`.

### MenuTestApp

A minimal WinForms app whose sole purpose is to expose a deep, nested menu hierarchy for
testing `GetMenus` / `SCAN_WINDOW` / `CLICK` against menu items:

```
File    → New (Ctrl+N), Open (Ctrl+O), ─, Save (Ctrl+S), Save As…, ─, Exit
Edit    → Copy (Ctrl+C), Paste (Ctrl+V), ─, Find → Find… (Ctrl+F), Find Next (F3), Find Previous (Shift+F3)
View    → Zoom → Zoom In (Ctrl++), Zoom Out (Ctrl+-), Reset Zoom; ─, Status Bar (checkable toggle)
Help    → Help Contents, ─, About
```

The status bar shows "Ready". Clicking **About** opens a MessageBox.

### Element record classes

Each test app has a companion record class that stores the deterministic element IDs
produced by a `Bridge.ScanWindowByName` call. These are used by integration tests to
reference elements without hard-coding integers inline:

| Class | File | Target app |
|---|---|---|
| `WinFormsTestAppElements` | `Core/WinFormsTestAppElements.cs` | WinFormsApplication |
| `WpfTestAppElements` | `Core/WpfTestAppElements.cs` | WpfApplication |
| `EcommerceTestPageElements` | `Core/EcommerceTestPageElements.cs` | E-commerce web page (browser-hosted) |

`EcommerceTestPageElements` covers a browser-based shopping flow: search input, category
select, price slider, add-to-cart button, cart quantity, coupon input, apply coupon,
checkout, newsletter subscribe.

---

## Logic map — execution flows

The following traces show how control flows from user action to side-effect for each
major feature.

### 1. Application startup

```
Program.Main()
  │
  ├─ ServiceCollection.AddSingleton<ISettingsService<FlaUiAppSettings>>
  │     └─ JsonSettingsService<FlaUiAppSettings>("appsettings.json")
  │
  ├─ App.Services = ServiceProvider
  │
  ├─ settingsService.Load()  ──►  App.ApplyAppOption(settings)
  │                                 └─ builds Func<ElementOverlay> for:
  │                                      App.FlaUiAppOptions.HoverOverlay
  │                                      App.FlaUiAppOptions.SelectionOverlay
  │                                      App.FlaUiAppOptions.PickOverlay
  │
  ├─ NativeLibraryConfig.All.WithCuda(false).WithVulkan(false)
  │     └─ NativeApi.llama_empty_call()   (forces native lib load now)
  │
  └─ Application.Run(new StartupForm(App.Logger))
```

### 2. StartupForm initialisation

```
StartupForm.ctor(logger)
  │
  ├─ InitializeComponent()                  (Designer.cs, all WinForms controls)
  ├─ WireEvents()                           (all button/timer/tree event handlers)
  │
  ├─ _grid.DataSource = _gridItems          (BindingList<ProcessWindowInfo>)
  ├─ HoverManager.Initialize(UIA3Automation, HoverOverlay factory, logger)
  │
  ├─ _aiSettingsService.Load()  ──►  _aiSettings
  ├─ ApplySettingsToUI()                    (populates API key, model path, system prompt)
  ├─ SetupProviderCombo()                   (wires provider combo; restores saved provider)
  │
  └─ Form.Load event:
        ├─ _windowedOnly.Checked = true
        ├─ await _viewModel.Init()          ──► SyncProcessesAsync (see §3)
        ├─ BindGrid()
        └─ _windowChangeTimer.Start()       (periodic background sync)
```

### 3. Process list — discovery and pick mode

```
StartupViewModel.Init()
  └─ SyncProcessesAsync()
       ├─ New STA background thread (UIA3 requires STA)
       │     ├─ automation.GetDesktop().FindAllChildren()
       │     │     [optionally filtered to ControlType.Window]
       │     └─ for each top-level element:
       │           ├─ skip self (current PID) and empty-name elements
       │           └─ ElementIdGenerator.GenerateElementHash/Id
       │                 └─ SHA-256(controlType | className | automationId |
       │                            frameworkId | processName) → int
       └─ diff result against _processes ObservableCollection
            └─ fires PropertyChanged → StartupForm.BindGrid()

StartupViewModel.PickProcessAsync()          [user clicks "Pick" button]
  └─ WaitForMouseClickWindowAsync(30s timeout)
       ├─ SetWindowsHookEx(WH_MOUSE_LL)      (low-level mouse hook)
       ├─ Cursor = Cursors.Cross
       ├─ on WM_MOUSEMOVE:
       │     ├─ WindowFromPoint → GetAncestor(GA_ROOT)
       │     ├─ skip own process
       │     └─ show PickOverlay around top-window bounding rect
       └─ on WM_LBUTTONUP:
             ├─ UnhookWindowsHookEx
             └─ return HWND  ──►  SelectedProcess = match from _processes
```

### 4. Tree inspection — loading a process

```
StartupForm.LoadSelectedProcess()
  └─ new ProcessViewModel(UIA3Automation, processId, hwnd, logger)
       │
       └─ ProcessViewModel.Initialize()
             ├─ rootElement = automation.FromHandle(hwnd)
             ├─ rootViewModel = new ElementViewModel(rootElement, null, level=0)
             │     ├─ reads Name, AutomationId, ControlType eagerly
             │     ├─ GenerateElementHash(parentHash=null, controlType, className,
             │     │                     automationId, frameworkId, processName, name)
             │     ├─ ElementId = SHA-256 first 8 hex chars → int
             │     └─ ElementRegistry.Register(this)   (WeakRef map: Id → vm)
             │
             ├─ rootViewModel.LoadChildren()
             │     └─ AutomationElement.FindAllChildren()
             │           └─ for each child: new ElementViewModel(child, parent, level+1)
             │                (children register themselves in ElementRegistry)
             │
             ├─ Elements = ObservableCollection<ElementViewModel>(topChildren)
             ├─ FocusTrackingMode = new(automation, callback)
             └─ SelectedItem = Elements[0]
                   └─ triggers ReadPatternsForSelectedItem (background thread)
                         └─ PatternItemsFactory.CreatePatternItemsForElement
                              └─ for each supported pattern: read properties
                                   → ObservableCollection<ElementPatternItem>

Tree expand (user expands a node):
  StartupForm._tree.BeforeExpand
    └─ ProcessViewModel.ExpandElement(node)
          └─ node.LoadChildren()  ──►  inserts child ElementViewModels into Elements
```

### 5. Overlay modes (Hover / Selection highlight / Focus tracking)

```
ProcessViewModel.SetMode()          [any mode checkbox toggled]
  │
  ├─ HoverManager.Disable(hwnd)     (always disable first)
  ├─ _trackHighlighterOverlay?.Dispose()
  └─ _focusTrackingMode?.Stop()

  [exactly one mode active:]
  ├─ EnableHoverMode      ──►  HoverManager.Enable(hwnd)
  │                              HoverManager timer (300ms):
  │                                if Ctrl held:
  │                                  automation.FromPoint(cursor)
  │                                  ──►  ProcessViewModel.ElementToSelectChanged(element)
  │                                         walks tree path → selects node → overlay
  │
  ├─ EnableHighLightSelectionMode
  │                        ──►  TrackSelectedItem(SelectedItem)
  │                                SelectionOverlay.Show(boundingRect)
  │
  └─ EnableFocusTrackingMode ──►  FocusTrackingMode.Start()
                                    UIA focus event  ──►  ElementToSelectChanged(element)
```

### 6. Bridge command execution (manual or AI-driven)

```
StartupForm [Bridge command panel or AI auto-exec]
  └─ _bridge.ExecuteCommand("CLICK 12345")
        │
        ├─ parse action token = "CLICK"
        ├─ parse elementId = 12345
        ├─ ResolveElement(12345)
        │     └─ _elements[12345].AutomationElement   (ConcurrentDictionary registry)
        │
        └─ RunOp(element, Operations.Click)
              └─ ElementOperations.Click(element, id)
                    ├─ InvokePattern.Invoke()           (buttons, links)
                    ├─ TogglePattern.Toggle()           (checkboxes)
                    ├─ SelectionItemPattern.Select()    (list items)
                    └─ element.Click()                  (fallback: simulated mouse)
                          └─ return OperationResult { IsSuccess, ErrorMessage }
        └─ return CommandResult { IsSuccess, Message, Data }

Bridge.ScanWindowByName("Notepad")       [SCAN_WINDOW command]
  └─ Task.Run:
        ├─ automation.GetDesktop().FindAllChildren()
        │     ──►  title/process-name substring match
        ├─ _elements.Clear(); _idGenerator.Reset()
        └─ ScanElementRecursive(root, parentId=null, parentHash=null, hwnd, depth=0, max=25)
              ├─ GenerateElementHash (parentHash | controlType | className |
              │                       automationId | frameworkId | processName | name)
              ├─ GenerateIdFromHash  (SHA-256 → first 8 hex → int)
              ├─ ElementRecord.FromAutomationElement(element, id, parentId, hash, …)
              │     captures: Name, AutomationId, ControlType, ClassName, BoundingRect,
              │               IsEnabled, IsOffScreen, WindowHandle, ProcessId,
              │               AutomationElement (live ref), FindCriteria
              ├─ recurse children (depth+1)
              └─ _elements[id] = record with { ChildIds = childIdList }
```

### 7. AI chat — multi-turn command loop

```
StartupForm.AiSendMessageAsync(stream=true)
  │
  ├─ [provider = "Anthropic"]
  │     for turn in 0..19:
  │       ├─ AnthropicClient.SendMessageStreamAsync(request)
  │       │     SSE parser  ──►  StreamingTextReceived  ──►  AiAppendOutput()
  │       ├─ _aiConversationHistory.Add(AssistantMessage)
  │       ├─ [AutoExec on] AiParseCommands(responseText)
  │       │     ──►  Regex: \[CMD:\s*([^\]]+)\]
  │       └─ AiExecuteCommandsFromResponse(commands)
  │             ├─ foreach cmd: _bridge.ExecuteCommand(cmd)
  │             ├─ append [RESULT: …] to output panel
  │             └─ _aiConversationHistory.Add(UserMessage(resultSummary))
  │             (loop: AI sees result and issues next command)
  │
  ├─ [provider = "LlamaSharp (Local)"]
  │     LlamaSharpClient.LoadModelAsync(modelPath, systemPrompt)
  │         ──►  LLamaWeights.LoadFromFile + LLamaContext + ChatSession
  │     foreach turn: LlamaSharpClient.SendMessageAsync(input)
  │         streaming via StreamingTextReceived event
  │         (same auto-exec loop as Anthropic)
  │
  └─ [provider = "LlamaSharp Instruct"]
        LlamaSharpInstructClient (instruct/completion template)
        same turn loop pattern
```

### 8. Export flows

```
CopyJsonToClipboard
  └─ JsonTreeExporter.Export(rootViewModel)
        ├─ recursive ExportElement:
        │     elementId, name, controlType, boundingRectangle, xpath (optional)
        │     PatternItemsFactory.CreatePatternItemsForElement  ──►  "details" object
        └─ JsonNode.ToJsonString(indented)  ──►  Clipboard

        side-effect:
        └─ UiMapRenderer.ShowOverlay(json, 5000ms)
              ├─ parses BoundingRectangle from JSON
              └─ draws coloured labelled rectangles on a full-screen transparent form

RenderUiMapCommand
  └─ same JSON export, then:
        UiMapRenderer.Render(json, filePath, screenWidth, screenHeight)
          ──►  draws all elements to a Bitmap  ──►  saves as PNG

CaptureSelectedItem
  └─ AutomationElement.Capture()  ──►  Bitmap  ──►  SaveFileDialog  ──►  PNG

DESCRIBE command (Bridge)
  └─ element.Capture()  ──►  PNG temp file
     LMStudioClient.SendMessageWithImagesAsync(prompt, [imagePath])
       ──►  chat-completions POST with base64 image  ──►  description string
     File.Delete(tempPath)
```

---

## Bridge command reference

> **All commands are issued as plain text.** The AI system prompt instructs the model
> to wrap each command in `[CMD: COMMAND args]`. The parser regex
> `\[CMD:\s*([^\]]+)\]` extracts the inner text and passes it to
> `Bridge.ExecuteCommand`.

### Mouse / interaction

| Command | Syntax | Notes |
|---|---|---|
| `CLICK` | `CLICK <id>` | InvokePattern → TogglePattern → SelectionItem → mouse |
| `DOUBLE_CLICK` | `DOUBLE_CLICK <id>` | |
| `RIGHT_CLICK` | `RIGHT_CLICK <id>` | |
| `MIDDLE_CLICK` | `MIDDLE_CLICK <id>` | |
| `CLICK_OFFSET` | `CLICK_OFFSET <id> <x> <y>` | Click at pixel offset from element centre |
| `CLICK_COORDS` | `CLICK_COORDS <id>` | X-Y fallback: bring window to front, click centre of stored bounding rect using raw mouse coordinates. Use when UIA pattern interaction fails. |
| `DRAG` | `DRAG <id> <targetX> <targetY>` | Drag element to absolute screen position |
| `DRAG_TO_ELEMENT` | `DRAG_TO_ELEMENT <srcId> <dstId>` | Drag from one element to another |
| `HOVER` | `HOVER <id>` | Move mouse to element centre |
| `HIGHLIGHT` | `HIGHLIGHT <id>` | Draw selection overlay on element |

### Keyboard / text

| Command | Syntax | Notes |
|---|---|---|
| `TYPE` | `TYPE <id> <text>` | Sets value via ValuePattern or keyboard simulation |
| `SEND_KEYS` | `SEND_KEYS <id> <keys>` | Raw key sequence (e.g. `{ENTER}`, `^a`) |

### Value / state

| Command | Syntax |
|---|---|
| `SET_VALUE` | `SET_VALUE <id> <value>` |
| `SET_SLIDER` | `SET_SLIDER <id> <value>` |
| `TOGGLE` | `TOGGLE <id>` |
| `EXPAND` | `EXPAND <id>` |
| `COLLAPSE` | `COLLAPSE <id>` |
| `SELECT` | `SELECT <id>` |
| `SELECT_BY_TEXT` | `SELECT_BY_TEXT <id> <text>` |
| `SELECT_BY_INDEX` | `SELECT_BY_INDEX <id> <index>` |
| `FOCUS` | `FOCUS <id>` |

### Scroll

| Command | Syntax | Direction values |
|---|---|---|
| `SCROLL` | `SCROLL <id> <direction> [amount]` | `up`, `down`, `left`, `right`, `pageup`, `pagedown` |
| `SCROLL_HORIZONTAL` | `SCROLL_HORIZONTAL <id> <amount>` | positive = right, negative = left |
| `SCROLL_INTO_VIEW` | `SCROLL_INTO_VIEW <id>` | |

### Window management

| Command | Syntax |
|---|---|
| `WINDOW_ACTION` | `WINDOW_ACTION <id> minimize\|maximize\|restore\|close` |
| `LIST_WINDOWS` | `LIST_WINDOWS` — returns JSON array of window titles |
| `SCAN_WINDOW` | `SCAN_WINDOW <windowName>` — title substring match; returns full element tree text |

### Inspection

| Command | Syntax |
|---|---|
| `GET_ELEMENT` | `GET_ELEMENT <id>` — returns JSON `{elementId, name, controlType, isEnabled}` |
| `GET_TEXT` | `GET_TEXT <id>` — reads element value/text content |
| `GET_TREE` | `GET_TREE [maxDepth]` — indented text tree of scanned elements |
| `SEARCH` | `SEARCH <text>` — name/automationId substring match in registry |
| `REFRESH` | `REFRESH` — no-op acknowledgement |
| `HELP` | `HELP` — returns this command list |

### Capture / vision

| Command | Syntax |
|---|---|
| `CAPTURE` | `CAPTURE <id>` — saves element screenshot to temp PNG, returns path |
| `CAPTURE_WINDOW` | `CAPTURE_WINDOW <windowName> [id1 id2 …]` — captures window root |
| `DESCRIBE` | `DESCRIBE <id> [prompt]` — captures element, sends to LM Studio vision model |

---

## Element ID system

Element IDs are **deterministic integers** derived from a structural hash. The same
element always gets the same ID across runs (as long as the UI structure doesn't change).

```
SHA-256(
  parentHash | controlType | className | automationId
  | frameworkId | processName [| name] [| idx:siblingIndex]
)
─── take first 8 hex characters ───►  Convert.ToInt32(hex, 16) ───►  Math.Abs()
```

- **Window/Pane** elements exclude `name` (window titles change on navigation) and use
  the native window handle for stability.
- **Incremental mode** (`UseIncrementalIds = true`) assigns sequential integers (1, 2, 3…)
  keyed by hash — useful when the same control appears multiple times (content rescans).
- IDs are scoped to a scan. Rescanning clears and rebuilds the registry.

Two separate registries track live elements:

| Registry | Key | Value | Purpose |
|---|---|---|---|
| `Bridge._elements` | `int id` | `ElementRecord` | Command execution (Bridge commands) |
| `ElementRegistry` (static) | `int id` | `WeakReference<ElementViewModel>` | UI toolbar direct-click |

---

## Key types at a glance

| Type | File | Role |
|---|---|---|
| `Program` | `Program.cs` | Entry point, DI, native lib init |
| `App` | `App.xaml.cs` | Static container: services, logger, overlay factories |
| `StartupForm` | `Forms/StartupForm.cs` | Main window, AI chat, event wiring |
| `AiSettingsDialog` | `Forms/AiSettingsDialog.cs` | AI provider/model configuration dialog |
| `StartupViewModel` | `ViewModels/StartupViewModel.cs` | Process discovery, pick-mode mouse hook |
| `ProcessViewModel` | `ViewModels/ProcessViewModel.cs` | Tree, hover/focus/highlight, export commands |
| `ElementViewModel` | `ViewModels/ElementViewModel.cs` | UIA element wrapper, ID generation, detail/pattern loading |
| `Bridge` | `Core/Bridge.cs` | Command façade: scan, dispatch, element registry |
| `ElementOperations` | `Core/ElementOperations.cs` | Low-level UIA interactions |
| `ElementIdGenerator` | `Core/ElementIdGenerator.cs` | SHA-256 hash → deterministic int ID |
| `ElementRegistry` | `Core/ElementRegistry.cs` | Global WeakRef map: ID → ElementViewModel |
| `ElementRecord` | `ElementRecord.cs` | Immutable snapshot of a scanned element |
| `ElementFindCriteria` | `ElementRecord.cs` | Re-find metadata for a scanned element |
| `ElementOverlay` | `Core/ElementOverlay.cs` | Screen border/fill overlay via SetWindowPos |
| `HoverManager` | `Core/HoverManager.cs` | 300 ms Ctrl+hover polling |
| `FocusTrackingMode` | `Core/FocusTrackingMode.cs` | UIA focus-changed event subscriber |
| `ElementFinderService` | `Core/ElementFinderService.cs` | Window/element search with fuzzy matching |
| `JsonTreeExporter` | `Core/Exporters/JsonTreeExporter.cs` | Element tree → JSON |
| `XmlTreeExporter` | `Core/Exporters/XmlTreeExporter.cs` | Element tree → XML |
| `XmlElementDetailsExporter` | `Core/Exporters/XmlElementDetailsExporter.cs` | Element details → XML |
| `UiMapRenderer` | `Core/UiMapRenderer.cs` | Element bounds → PNG diagram |
| `AnthropicClient` | `Anthropic/AnthropicClient.cs` | Anthropic Messages API (streaming + single-shot) |
| `LlamaSharpClient` | `LlamaSharpAI/LlamaSharpClient.cs` | Local GGUF chat session |
| `LlamaSharpInstructClient` | `LlamaSharpAI/LlamaSharpInstructClient.cs` | Local GGUF instruct/completion session |
| `LMStudioClient` | `LMStudio/LMStudioClient.cs` | LM Studio vision endpoint client |
| `FlaUiAppSettings` | `Settings/FlaUiAppSettings.cs` | JSON app settings schema |
| `FlaUiAppOptions` | `Settings/FlaUiAppOptions.cs` | Runtime overlay factory delegates |
| `JsonSettingsService<T>` | `Settings/JsonSettingsService.cs` | Generic JSON read/write |
| `AiSettings` | `Models/AiSettings.cs` | Persisted AI provider configuration |

---

## Practical caveats

- **UIA accessibility**: some processes (elevated/UAC, certain sandboxed apps) are
  inaccessible to UIA from a non-elevated process. Run ApexUIBridge as Administrator
  if you need to inspect elevated targets.
- **Electron / CEF apps**: control types are often `Custom`; the `IsChromeBased` helper
  in `Bridge` detects Chrome-based windows by class name for adjusted handling.
- **Stale elements**: UIA elements become stale when the target UI changes. The
  `SafeGetProperty` wrappers throughout `ElementRecord` and `ElementIdGenerator`
  swallow COM exceptions to avoid crashes.
- **ID stability**: IDs are stable relative to the scanned structure. If the target
  app adds/removes/reorders elements, IDs for other elements may shift.
- **AI auto-exec**: the system prompt enforces one command per turn and a 20-turn cap.
  Still, always review AI-generated commands before enabling `AutoExec` in
  non-test environments.
- **LlamaSharp GPU**: the native lib is initialised with `WithCuda(false).WithVulkan(false)`
  by default. To enable GPU offload, modify `Program.cs` and set `GpuLayers > 0` in
  settings.

---

Copyright (c) 2026 John Brodowski.

The embedded FlaUI projects (`FlaUI.Core`, `FlaUI.UIA2`, `FlaUI.UIA3`) and `LLamaSharp`
are included as local project references — see their `LICENSE.txt` files for their
respective licences.

> **Important**: The bundled copies of FlaUI and LlamaSharp contain targeted modifications
> to improve reliability and stability in the ApexUIBridge context. Substituting upstream
> or otherwise unmodified versions of these libraries **will cause application instability**
> and is not supported.
