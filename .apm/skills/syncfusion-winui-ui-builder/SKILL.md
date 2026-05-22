---
name: syncfusion-winui-ui-builder
description: Generates production-ready WinUI 3 desktop applications powered by Syncfusion WinUI Controls. Orchestrates a structured workflow that handles design thinking, control picking, code generation, and validation with built-in WCAG 2.1 AA accessibility and responsive design. Use when the user asks to create WinUI controls, build desktop UI pages, design interfaces, or generate code for WinUI applications.
metadata:
  author: "Syncfusion Inc"
  version: "1.0.0"
---

# Syncfusion WinUI UI Builder

## Overview

The **Syncfusion WinUI UI Builder** skill is a desktop-only WinUI control generator that orchestrates an AI agent through 8 stages to generate production-ready UI controls powered by Syncfusion.

## What This Skill Does

**✅ Generates:**
- WinUI controls (C# code-behind with XAML markup)
- XAML layout files with proper namespaces
- C# interfaces and data models
- Syncfusion control integration with correct NuGet references
- Client-side form validation logic
- WCAG 2.1 AA accessibility markup (accessible controls)
- Responsive layouts with proper sizing and DPI-aware positioning
- Control and resource files with proper structure

**❌ Does NOT Generate:**
- Backend code (services, database handlers, middleware)
- Database schemas or ORM models
- Authentication/authorization logic
- Server-side validation
- Navigation configuration
- Environment secrets or infrastructure config

## Quick Start

### Prerequisites

1. **Active WinUI project** (.NET 8.0+, Windows App SDK 1.8+)
2. **.NET SDK 8.0+** and Visual Studio 2022 (v17.8+) or later
3. **Syncfusion WinUI controls library** (latest):
   ```bash
   dotnet add package Syncfusion.Core.WinUI --version "*"
   ```

### Basic Usage

**Example 1: Generate a Login Form**

```
User: "Create a login form with email, password, and remember me checkbox"

Skill executes:
  → Stage 1: Identifies login form control type
  → Stage 2: Detects project structure (WinUI, .NET version, etc.)
  → Stage 3-4: AI creates optimal control-mapping.json → maps to Syncfusion controls
  → Stage 5: Generates LoginForm.xaml and LoginForm.xaml.cs with validation
  → Stage 6: Installs NuGet dependencies
  → Stage 7: Validates WCAG 2.1 AA compliance
  → Stage 8: Inserts code into project

Output:
  ✓ Views/LoginForm/LoginForm.xaml
  ✓ Views/LoginForm/LoginForm.xaml.cs
  ✓ Models/LoginFormModel.cs
```

**Example 2: Generate a Data Table**

```
User: "Build a customer data table with sorting and filtering"

Output:
  ✓ Views/CustomerTable/CustomerTable.xaml (with Syncfusion DataGrid)
  ✓ Models/CustomerModel.cs with sample data
  ✓ Responsive layout with DPI scaling
  ✓ WCAG 2.1 AA accessibility compliance
```

### ⚠️ Important: File Organization Inside Project Directory

**ALL generated files are created INSIDE your WinUI project directory:**

```
MyWinUIApp/                                (Project Root with .csproj)
├── Views/
│   ├── LoginForm/
│   │   ├── LoginForm.xaml                 ✅ View file
│   │   └── LoginForm.xaml.cs              ✅ Code-behind
│   └── CustomerTable/
│       ├── CustomerTable.xaml             ✅ View file
│       └── CustomerTable.xaml.cs          ✅ Code-behind
├── Models/
│   ├── LoginFormModel.cs                  ✅ Data model
│   └── CustomerModel.cs                   ✅ Data model
├── ViewModels/
│   └── LoginFormViewModel.cs              ✅ ViewModel (if MVVM)
├── Controls/
│   └── [Reusable controls]                ✅ Shared control components
└── MyWinUIApp.csproj                      (Project file)
```

**Key Principle:** Views, Models, and ViewModels are organized in logical subdirectories within your project root (where `.csproj` exists). Files are never created outside the project directory.

## How It Works: 8-Stage AI Orchestration (Stateless)

The skill orchestrates **8 stages of pure AI reasoning** with **two user decision points**.

**Key Architecture:**
- **Stateless design**: Conversation history maintains state
- **Pure AI reasoning**: Each stage reads guidance docs, analyzes context, makes decisions
- **2 user decision gates**: Stage 3 (control confirmation) + Stage 6 (validation result)
- **6 fully automated stages**: 1, 2, 4, 5, 7, and final code insertion
- **Dedicated theming stage**: Stage 4 locks design system before code generation

```
User Request
    ↓
[Stage 1: Intent Analysis] 
  AI reads query → identifies control type & features
    ↓
[Stage 2: Project Detection]
  AI scans project → detect framework, .NET version, theming strategy, preferences
    ↓
[Stage 3: Layout Analysis & Control Mapping] ⭐ USER DECISION #1
  AI analyzes requirements → creates optimal control-mapping.json
  AI maps to specific Syncfusion controls (3+ controls)
  User confirms control selection
    ↓
[Stage 4: Theming & Design System] (NEW)
  AI locks design tokens → Syncfusion theme mapping
  AI selects color system (brand colors, theming)
  AI confirms spacing/typography scale (DPI-aware sizing)
  Design system decisions locked before code generation
    ↓
[Stage 5: Code Generation]
  AI generates XAML, C#, data models
  Uses theming decisions from Stage 4
  With accessibility + responsive design built-in
    ↓
[Stage 6: Dependencies]
  AI detects required NuGet packages (Syncfusion + frameworks)
  Presents dotnet add command or runs it
    ↓
[Stage 7: Validation] ⭐ USER DECISION #2
  AI validates WCAG 2.1 AA + security + performance + theming
  Binary result: PASS ✓ or FAIL ✗
  User confirms or overrides
    ↓
[Stage 8: Code Insertion]
  AI inserts files into project
  Updates references, verifies build
    ↓
✓ Complete
```

**Stage Descriptions:**

- **Stage 1 (Intent Analysis)**: Parse user query, identify control type and features. Read: `references/stage-1-intent-analysis.md`
- **Stage 2 (Project Detection)**: Auto-detect framework, .NET version, theming strategy, control directory. Read: `references/stage-2-project-detection.md`
- **Stage 3 (Layout Analysis & Control Mapping)**: AI analyzes requirements, creates optimal control-mapping.json, maps to Syncfusion controls. User confirms 3+ control selection. Read: `references/stage-3-layout-analysis.md`
- **Stage 4 (Theming & Design System)**: Lock design tokens, Syncfusion theme, color system, spacing (DPI-aware), typography. Read: `references/stage-4-theming-and-design-system.md`
- **Stage 5 (Code Generation)**: Generate WinUI with design tokens from Stage 4 applied + accessibility + responsive design. Read: `references/stage-5-code-generation.md`
- **Stage 6 (Dependencies)**: Detect NuGet packages (Syncfusion + frameworks), resolve conflicts, prepare install command. Read: `references/stage-6-dependencies.md`
- **Stage 7 (Validation)**: Validate WCAG 2.1 AA, security, performance, theming integration. Binary pass/fail. Read: `references/stage-7-validation.md` + `assets/validation-rules.md` + `references/winui-standards.md`
- **Stage 8 (Code Insertion)**: AI inserts files, updates references, verifies build succeeds.

**User Interaction Summary:**

| Stage | Interaction |
|-------|-------------|
| 1 | None (AI analyzes) |
| 2 | Confirm auto-detected settings |
| 3 | ⭐ Confirm control selection (3+ Syncfusion controls) |
| 4 | Confirm theming decisions (design tokens, colors, spacing, typography) |
| 5 | None (AI generates) |
| 6 | ⭐ Confirm validation result (pass/fail/override) |
| 7 | Optional (confirm dotnet add) |
| 8 | None (AI executes) |

**Total user decision gates: 2** (Stage 3: controls, Stage 6: validation). Rest fully automated with AI reasoning + guidance docs.

## Agent Instructions

### When User Requests UI Control Generation

1. **Validate scope**: Confirm request is for WinUI controls (not backend/API)
2. **Load guidance**: Read `stage-1-intent-analysis.md` to understand Stage 1
3. **Execute 8-stage flow**: Follow the orchestration flow shown above
4. **Progressive disclosure**: Load stage guides on-demand; load support references only when needed
5. **Maintain conversation history**: Each stage reads previous decisions from conversation context (stateless)

### Stage Execution & Reference Loading

**Stage 1: Intent Analysis**
- Read: `references/stage-1-intent-analysis.md`
- Task: Parse user query, identify control type, resolve ambiguities
- Output: Control type + modifiers + target directory

**Stage 2: Project Detection**
- Read: `references/stage-2-project-detection.md`
- Task: Auto-detect WinUI framework, .NET version, theming strategy, formatting rules
- Output: Project configuration + user confirmation

**Stage 3: Layout Analysis & Control Mapping** ⭐ MANDATORY SCRIPT EXECUTION
- Read: `references/stage-3-layout-analysis.md`
- Task: Analyze user requirements → create optimal control-mapping.json → **RUN controls_search.cjs script** → map to Syncfusion controls
- Script: `scripts/controls_search.cjs` (uses BM25 algorithm for semantic control matching)
- Execution: `node controls_search.cjs <project-root>/control-mapping.json`
- Output: `control-mapping.json` (file) + Control mapping results (chat context) + Summary table

**Stage 4: Theming & Design System** (NEW)
- Read: `references/stage-4-theming-and-design-system.md`
- Task: Lock design tokens, Syncfusion theme, color system, spacing (DPI-aware), typography, responsive breakpoints
- Output: Design system decisions confirmed and ready for code generation

**Stage 5: Code Generation**
- Read: `references/stage-5-code-generation.md`
- Task: Generate WinUI XAML, C#, data models using theming from Stage 4
- Ensure: WCAG 2.1 AA accessibility compliance, responsive design, token architecture applied
- Output: Generated files ready for review

**Stage 6: Dependencies**
- Read: `references/stage-6-dependencies.md`
- Task: Detect required NuGet packages (Syncfusion + frameworks), resolve version conflicts
- Output: dotnet add command or auto-install

**Stage 7: Validation** ⭐ USER DECISION #2
- Read: `references/stage-7-validation.md` + `assets/validation-rules.md` + `references/winui-standards.md`
- Task: Validate WCAG 2.1 AA, security, performance, theming integration standards
- Auto-apply fixes where possible
- Output: Binary result (PASS ✓ or FAIL ✗) → user confirms or overrides

**Stage 8: Code Insertion**
- Task: Insert generated files into project, update references, verify build
- Output: Success report with file paths

### Boundary Rules (CRITICAL)

**AI agents executing this skill MUST:**

1. **Frontend only**: Never generate backend code (services, database schemas, middleware)
2. **Mock data only**: Use hardcoded samples or simple data models; no real API calls
3. **No secrets**: Exception: `.env` or `appsettings.json` for `SYNCFUSION_LICENSE_KEY` when user provides
4. **WinUI controls only**: Generate `.xaml`/`.xaml.cs` files in appropriate directories
5. **Redirect backend requests**: *"This skill generates WinUI UI only. Backend integration is your app's responsibility. Ready to generate the UI?"*

### Error Handling

If any stage fails:

1. **Retry once** with same approach
2. **If retry fails**, attempt workaround or skip to next stage
3. **Notify user** with error message from stage output
4. **Offer recovery**: *"Would you like to go back to Stage 3 and choose a different layout?"*
5. **Reference**: `references/build.md` for common errors

### Resource Loading Strategy (Progressive Disclosure)

**Load SKILL.md first** (you're reading it now) ~400 lines

**Load stage guides on-demand** (each <200 lines):
- `stage-1-intent-analysis.md` → During Stage 1
- `stage-2-project-detection.md` → During Stage 2
- `stage-3-layout-analysis.md` → During Stage 3
- etc.

**Load support references only when needed**:
- `winui-standards.md` → When validating in Stage 5
- `build.md` → When errors occur
- `assets/validation-rules.md` → When validating in Stage 5

**Result**: Initial load ~400 lines (SKILL.md only). Full spec available on-demand, never exceeding Agent Skills context limits.

---

## Scripts & Tools

### Stage 3: ControlMapper Script (`controls_search.cjs`)

**Purpose:** Automatically map UI elements to Syncfusion WinUI controls using BM25 semantic search algorithm.

**Location:** `scripts/controls_search.cjs`

**What It Does:**
- Reads `control-mapping.json` with element descriptions and `type_hint` values
- Searches Syncfusion WinUI control keywords using BM25 ranking algorithm
- Matches each element to the best-fit Syncfusion control
- Falls back to `NATIVE_XAML` for unmatched elements
- Returns control mapping with BM25 scores (0-100 range)

**Data Source:**
- `scripts/controls.csv` - 100+ Syncfusion WinUI controls with keywords (auto-loaded)

**Execution Syntax:**

```powershell
# Navigate to scripts directory
cd <project-root>\.apm\skills\syncfusion-winui-ui-builder\scripts

# Run with absolute path to control-mapping.json
node controls_search.cjs <project-root>\control-mapping.json
```

**Example (Windows):**

```powershell
cd d:\MyWinUIApp\.apm\skills\syncfusion-winui-ui-builder\scripts
node controls_search.cjs d:\MyWinUIApp\control-mapping.json
```

**Prerequisites:**
- Node.js 14+ installed on system
- `control-mapping.json` must exist at specified path
- `controls.csv` must be in same directory as script

**Output:**
- JSON printed to console with mapped controls + BM25 scores
- Copy output into chat for Stage 4 (theming) and Stage 5 (code generation)
- Do NOT save output to file (keep in conversation only)

**Error Handling:**
- If `control-mapping.json` not found → Error message with full path
- If `controls.csv` not found → Error message
- If JSON parse error → Error with line number and context

**BM25 Algorithm Details:**
- **Tokenization:** Splits keywords on whitespace and commas
- **Term Frequency (TF):** Counts occurrences in each control
- **Inverse Document Frequency (IDF):** Ranks rare keywords higher
- **Saturation (k1=1.5):** Diminishing returns on term frequency
- **Length Normalization (b=0.75):** Adjusts for control keyword length

---

## Configuration & User Customization

### Auto-Detected Settings

During **Stage 2 (Project Detection)**, AI automatically detects:

- **Framework**: WinUI 3, .NET 6+, Windows App SDK version
- **Language**: C# (.NET language)
- **Theming**: XAML theming, default Syncfusion theme, resource dictionaries
- **Formatting**: C# code style rules, naming conventions
- **Control Directory**: `Views/`, `Pages/`, `Controls/`, or similar

### User Override Options

In **Stage 2**, user can override any detected setting:

```
Detected Settings:
  Framework: WinUI 3
  .NET Version: .NET 7
  Theming: XAML with Syncfusion theme
  Control Directory: Views/

[Confirm] [Override Each] [Cancel]
```

### Syncfusion License Configuration

The skill handles license key setup:

1. **Check** for existing `SYNCFUSION_LICENSE_KEY` in `appsettings.json` or environment
2. **If missing**, prompt user: *"Get a free Community License at https://www.syncfusion.com/account/manage-trials"*
3. **If provided**, write to `appsettings.json` + inject `registerLicense()` in app initialization
4. **If skipped**, proceed but warn that watermark will appear in controls

---

## Code Generation Standards

All generated code includes:

### Accessibility (WCAG 2.1 AA)
- ✅ Semantic XAML controls with proper naming
- ✅ AutomationProperties for screen readers
- ✅ Keyboard navigation support (tab order, focus management)
- ✅ Color contrast ≥ 4.5:1
- ✅ Focus indicators on interactive elements

### Responsive Design
- ✅ DPI-aware sizing (logical vs physical pixels)
- ✅ Relative layouts using Grid/StackPanel (no fixed widths)
- ✅ Adaptive breakpoints for different window sizes
- ✅ Touch-friendly controls (44x44 device-independent units minimum)

### Security
- ✅ Input validation in code-behind
- ✅ No hardcoded secrets in XAML
- ✅ Secure binding and command patterns
- ✅ Protection against code injection

### Performance
- ✅ Virtualization for large lists
- ✅ Event handler optimization
- ✅ Lazy loading for heavy resources
- ✅ Efficient data binding

### C# & Types
- ✅ Full type coverage (no dynamic types without reason)
- ✅ ViewModel/Model interfaces with XML docs
- ✅ Event handler signatures
- ✅ Proper INotifyPropertyChanged implementation

## Supported Use Cases

- **Login form**: TextBox (email), TextBox (password), CheckBox (remember), Button (submit)
- **Data table**: DataGrid with sorting, filtering, pagination, row selection
- **Dashboard**: Multiple controls orchestrated (header, sidebar, main content, footer)
- **Registration wizard**: Multi-step form with ProgressBar and validation

## Troubleshooting

**Common Issues:**

| Issue | Solution |
|-------|----------|
| "Project type not detected" | Ensure `.csproj` exists with Windows App SDK dependency |
| "Syncfusion license watermark appears" | Add license key via Stage 2 prompt |
| "Build fails after insertion" | Check `references/build.md` for conflict resolution |
| "Control not rendering" | Verify namespace declarations and ensure parent control references correctly |

**Full guide**: See `references/build.md`

## Additional Resources

### Quick Reference by Use Case

| Need | Reference File |
|------|-----------------|
| Understanding workflow | This SKILL.md file |
| How Stage X works | `references/stage-X-*.md` |
| Validation rules | `assets/validation-rules.md` |
| Accessibility/security | `references/winui-standards.md` |

## Support

For issues or questions:
1. Check `references/build.md` for common problems
2. Verify your project meets prerequisites (.NET 6+, Windows App SDK 1.3+)
3. Ensure Syncfusion license is valid and registered
4. Review generated code compliance report for warnings
