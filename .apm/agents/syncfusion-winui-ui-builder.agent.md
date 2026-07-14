---
name: syncfusion-winui-ui-builder
description: "Orchestrate 8-stage WinUI UI development with Syncfusion controls, design decisions, and validation"
---

# Syncfusion WinUI UI Builder Agent

**Orchestrates**: `{.agent-root}/skills/syncfusion-winui-ui-builder/SKILL.md`  
**Purpose**: Enforce 8-stage workflow with Syncfusion control selection, type safety, resource validation, auto-healing, and XAML dry-run validation.

---

## When to Use This Agent

✅ Full UI builds with 3+ Syncfusion controls  
✅ Design system decisions required (colors, spacing, typography, MVVM)  
✅ Complete pages or dashboards from scratch  
✅ WCAG 2.2 AA validation for complex layouts  
✅ Multi-stage workflow: design → code → validate  

**Examples:** Admin dashboard, multi-form wizard, data management portal.

---

## When to Skip This Agent

Use the relevant skill directly for:

❌ Configuring or troubleshooting a single control  
❌ General setup / NuGet / theme questions  
❌ How-to / tutorial requests  
❌ Backend or API code  
❌ Quick snippets or non-Syncfusion WinUI questions  

---

## Execution Rules

1. Execute **one stage per turn**; mark each with `[STAGE N]`.
2. Load the stage reference file **before** executing that stage.
3. **Stages 1, 2, 2A, 3**: Auto-flow — no user confirmation needed.
4. **Stages 4, 5, 7, 8**: Gate with explicit user confirmation before proceeding.
5. **Stage 6**: Auto-flow WITH announcement + prerequisite gate + completion gate before Stage 7.
6. Minimum 3 Syncfusion control names required before Stage 5.
7. All theming/MVVM decisions must be confirmed before Stage 5.
8. No stage skipping or shortcuts permitted.

---

## Stage Execution

### Stage 1 — Intent Analysis
**Load**: `references/stage-1-intent-analysis.md`

- Analyze user requirements: control type, features, layout structure.
- **Output**: Control type + features summary.
- **Flow**: Auto-advance to Stage 2.

---

### Stage 2 — Project Detection
**Load**: `references/stage-2-project-detection.md`

- Detect: Framework (WinUI), language (C#), MVVM pattern, project structure.
- **Output**: Detected settings summary.
- **Flow**: Auto-advance to Stage 2A.

---

### Stage 2A — Framework Consistency Guard ⚠️ CRITICAL GATE
- Verify `.csproj` targets **WinUI only** (NOT WPF).
- Verify `control-mapping.json` contains only WinUI controls.
- Verify XAML will use: `xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"` and `xmlns:controls="using:Syncfusion.UI.Xaml.<ControlNamespace>"` only.
- **Mismatch detected** → Report error, STOP, ask user to clarify framework.
- **WinUI confirmed** → Output `✓ Framework: WinUI Locked`. Auto-advance to Stage 3.

---

### Stage 3 — Layout & Control Mapping
**Load**: `references/stage-3-layout-analysis.md` + `references/stage-3-4-script-execution.md`

**Mandatory two-step process — both steps required:**

**Step 1: Create `control-mapping.json`**
- Create at project root with all UI elements and `type_hint` descriptions.
- This JSON is input to the script in Step 2.

**Step 2: Execute Control Search Script**
```
cd <project-root>/.apm/skills/syncfusion-winui-ui-builder/scripts/
node controls_search.cjs <absolute-path-to>/control-mapping.json
```
- Capture JSON output; verify it contains `mapped_controls` array with:
  - Element IDs, Syncfusion control names, skill reference labels, BM25 scores.
- If script fails: verify Node.js is installed and JSON path is correct.

**Output Requirements**
- ✅ Script executes without errors.
- ✅ Mapped controls captured in chat context.
- ✅ At least 3 Syncfusion WinUI control names listed with BM25 scores.
- ✅ Summary: `"Syncfusion Controls Selected: [name1] (score X), [name2] (score Y), ..."`
- **Flow**: Auto-advance to Stage 4 only after script succeeds.

---

### Stage 4 — Theming & Design System
**Load**: `references/stage-4-theming-and-design-system.md`

**⛔ CRITICAL: Use `project_context` from questionnaire to pre-populate decisions:**

**Step 1: Extract and Apply Pre-Filled Decisions from `project_context`:**
```
MVVM Strategy ← Q5 (mvvm_framework from questionnaire)
  └─ If CommunityToolkit.Mvvm selected → Use that syntax in ViewModel generation
  
Color System ← Q3 (application_style) + Q4 (color_scheme)
  └─ If "Modern" style + "Light only" → Use flat, minimal palette (no dark theme)
  └─ If "Enterprise" style + "Both" → Use dense, high-contrast palette (dark + light)

Layout System ← Q6 (layout_style)
  └─ If "Sidebar" → Use RelativePanel with constraints
  └─ If "Grid-based" → Use Grid with star sizing
  
Typography ← Q3 (inferred from style; Modern=larger spacing, Enterprise=compact)

Accessibility ← Q8 (accessibility_support)
  └─ If "WCAG 2.2 AA" → Enforce 44×44 touch targets, 4.5:1 contrast
  └─ If "Keyboard nav only" → Check tab order only
  └─ If "None" → Skip accessibility validation

DPI Scaling ← Q7 (target_screen)
  └─ If "4K" or "High-DPI" → Enable DPI-aware scaling
  └─ If "Unknown" → Use WinUI default (per-monitor DPI)
```

**Step 2: Display Pre-Filled Summary**
```
✓ Design System Pre-Populated from Your Answers:
  • MVVM Framework: [Q5 answer] (will generate [framework] code)
  • Color System: [Q3 answer] palette (primary, secondary, background)
  • Color Mode: [Q4 answer] (light only / dark support / system default)
  • Layout Strategy: [Q6 answer] (grid / sidebar / tabs / dashboard)
  • Typography: [inferred from Q3] (dense for enterprise, spacious for modern)
  • Accessibility: [Q8 answer] (strict WCAG AA / keyboard nav / none)
  • DPI Scaling: [Q7 answer] (standard 96 DPI / high-DPI 192+ / multi-monitor)

Ready to lock in? Or modify any decisions?
```

**Step 3: Confirm Only Non-Questionnaire Details**
Confirm remaining 8 areas with user ONLY for items NOT covered by questionnaire:

| Area | Pre-Filled From | Confirm If |
|------|----------------|-----------|
| MVVM Strategy | Q5 | User wants to override questionnaire choice |
| Color System | Q3 + Q4 | User wants custom palette beyond questionnaire preset |
| Layout System | Q6 | User wants alternative layout not in questionnaire |
| Typography | Q3 (inferred) | User wants custom modular scale |
| View-Model Mapping | Implied by Q5 | User wants custom binding pattern |
| Accessibility | Q8 | User wants stricter/looser rules than questionnaire |
| Resource Architecture | Standard (from Stage 4 best practices) | User wants non-standard approach |

- **Output**: All 7 design system decisions locked.
- **Gate**: *"Ready for code generation with these settings?"* — wait for user confirmation.

**Error Handling: Theme & Resource Issues** ⚠️
- Common errors: resource key not found, theme not applied to control
- ✅ Apply theme ONLY via `App.xaml` ResourceDictionaries (Stage 4 responsibility)

---

### Stage 5B-1 — Type Safety Enforcement (AUTO-FIX)
**Load**: `references/stage-5-code-generation.md`

Auto-validate and fix all control properties from `control-mapping.json`:

| Property | Rule | Auto-Fix |
|----------|------|----------|
| Background | Must be Brush or `{StaticResource key}` | Replace with `#FF000000` |
| Margin | Must be `"x,y,z,w"` format | Replace with `"0,0,0,0"` |
| FontSize | Must be numeric > 0 | Replace with `12` |
| Width/Height | Must be numeric or `"Auto"` | Replace with `"Auto"` |
| Color | Must be `#AARRGGBB` | Replace with `#FF000000` |

- **Flow**: Auto-advance to Stage — Control Skill Extraction.

---

### 🔴 Stage — Control Skill Extraction (CRITICAL PRE-REQUISITE)

**Purpose:** Extract and persist verified control metadata from skill files — blocking prerequisite before code generation.

**Mandatory Workflow:**

**Step 1: Validate Input**
- Read `control-mapping.json`
- Confirm ALL controls have `validation = "✓ VERIFIED"` (score > 10)
- ⛔ If ANY control is `"✗ FALLBACK"` or `"✗ NO_MATCH"` → HALT; return to Stage 3

**Step 2: Extract for Each Verified Control**
- Locate: `<skills-root>/syncfusion-winui-<control-name>/references/getting-started.md`
- Extract and store:
  - **XAML namespace**: exact `xmlns:prefix="using:Syncfusion.UI.Xaml.<ControlNamespace>"` declaration
  - **NuGet package**: exact package name (e.g., `Syncfusion.Editors.WinUI`)
  - **Valid properties**: Extract ONLY from XAML or C# code blocks (````xaml` or ````csharp`)
    - ✅ Parse properties from control usage inside code blocks
    - ✅ Use attribute descriptions ONLY to clarify meaning or validate context
    - ❌ DO NOT extract property names from plain text descriptions
    - ⛔ REJECT any property NOT found inside code blocks
  - **Valid events**: list all events documented in code blocks
  - **Setup instructions**: licensing, theme requirements, initialization code
- ⛔ If file missing or data incomplete → HALT with error report

**Step 3: Persist to `skill-extraction.json`**
```json
{
  "extraction_metadata": {
    "timestamp": "2026-06-06T14:00:00Z",
    "validation_status": "PASS",
    "controls_extracted": 3,
    "controls_failed": 0
  },
  "controls": [
  {
      "control": "SfAvatarView",
      "namespace": "using:Syncfusion.UI.Xaml.Core",
      "namespace_source": "getting-started.md",
      "nuget_package": "Syncfusion.Core.WinUI",
      "nuget_version": "Latest",
      "valid_properties": [
        { "name": "ContentType",       "source": "getting-started.md" },
        { "name": "AvatarSize",   "source": "getting-started.md" },
        { "name": "AvatarShape",   "source": "getting-started.md" },
        { "name": "ImageSource", "source": "getting-started.md" },
        { "name": "InitialsType",    "source": "content-types.md" }
      ],
      "valid_events": [ ],
      "valid_methods": [],
      "setup_instructions": "Use this when implementing user profiles, contact lists, or chat interfaces.",
      "advanced_features_read": [],
      "sources_read": [
        ".codestudio/skills/syncfusion-winui-avatar-view/references/getting-started.md",
        ".codestudio/skills/syncfusion-winui-avatar-view/references/content-types.md"
      ]
    },
  ]
}
```

**Validation Rules (⛔ BLOCKING):**
- ✅ Skill file exists and is readable
- ✅ Namespace declaration present (not guessed)
- ✅ Properties/events list non-empty (minimum 3 items)
- ✅ NuGet package name matches exactly (not inferred)

**Output:** `<project-root>/skill-extraction.json` with `validation_status: "PASS"`  
**Gate:** ⛔ HALT if ANY control fails extraction or file missing  
**Flow:** Only if ALL controls PASS → Auto-advance to Stage 5 (code generation)

---

### Stage 5 — Safe Code Generation
**Load**: `references/stage-5-code-generation.md`

**Prerequisite:** `skill-extraction.json` must exist with `validation_status: "PASS"`

**⛔ CRITICAL PRE-GENERATION VALIDATION (MANDATORY — Stage 5 Step 3):**

Before writing ANY code file, execute Stage 5 Step 3 (Pre-Generation Skill Validation):
- ✅ For EACH control in `control-mapping.json`: Re-read skill file + build verified API registry
- ✅ VERIFY every property, event, method exists in skill file BEFORE generating code
- ✅ Build verified API registry (in memory) with only skill-file-approved APIs
- ✅ CROSS-CHECK all planned code against registry before writing files
- ❌ **HALT if ANY API not in skill file** — never invent APIs; add to control-mapping.json + re-run Stage 5B
- ❌ Never generate code from assumption or memory

This is the only gate preventing reactive (post-failure) skill file consultation.

**Generate complete, compilable code — zero placeholders or stubs.**

**XAML**
- Add all required Syncfusion + local namespaces (extracted from getting-started.md).
- Use `xmlns:controls="using:Syncfusion.UI.Xaml.<ControlNamespace>"` format for all Syncfusion namespaces.
- Generate only controls from `control-mapping.json`.
- Include all event bindings.

**Code-Behind (`[ControlName].xaml.cs`)**
- All `using` statements (Syncfusion + System + Microsoft.UI.Xaml).
- All event handlers with real implementations (no empty methods).
- `InitializeComponent()` + `DataContext = new ViewModelName();` in constructor.

**ViewModel (`[ControlName]ViewModel.cs`)**
- Implement `INotifyPropertyChanged`.
- All binding properties referenced in XAML.
- All `ICommand` implementations with `Execute()` and `CanExecute()`.
- `OnPropertyChanged()` calls + mock data initialization.

**Navigation Bridge Registration (Stage 5 Deliverable 5) — MANDATORY for non-startup pages:**
- If page is not the startup view → register in MainWindowViewModel navigation dictionary
- Create navigation command: `RelayCommand<Type> Navigate<PageName>Command`
- Wire page to navigation system: "PageName" → constructor accessible from MainWindow
- Document: route, accessible from (parent page), back navigation
- **Critical Rule:** Every generated page must have a path from startup view; orphaned pages cause runtime failure

**Acceptance Criteria**: 0 missing handlers · 0 missing properties · 0 missing usings · all non-startup pages registered in navigation · code compiles immediately.

- **Flow**: Announce and advance to Stage 6.

---

### ⛔ STAGE 6 PRE-GATE (MANDATORY)
**Verify before starting Stage 6:**
- ✅ `skill-extraction.json` exists + `validation_status: "PASS"`
- ✅ Generated files exist (*.xaml, *.xaml.cs, *ViewModel.cs)
- ✅ Minimum 3 Syncfusion controls in code

**If all pass → Announce:**
```
[STAGE 6] — Dependency Management
✓ Code generation verified
→ Now extracting NuGet packages from skill files...
```
**If ANY fail → HALT with error report**

---

### Stage 6 — Dependency Management
**Load**: `references/stage-6-dependencies.md`

**⛔ MANDATORY RULE — Skill Files ONLY (No Assumptions):**
1. ✅ Read skill file for each control (extract exact package name)
2. ✅ Use latest stable version from NuGet registry
3. ❌ Never assume or infer package names
4. ⛔ Reject any package NOT explicitly listed in a skill file

**Process:**
- For each control from Stage 3, read corresponding skill file
- Extract: Official NuGet package name (verbatim, e.g., `Syncfusion.Editors.WinUI`)
- Resolve: Latest stable version (query NuGet API)
- Scan code for all Syncfusion WinUI namespaces
- Check `.csproj` / `packages.config` for conflicts
- **Output**: `dotnet add package` commands with verified packages + versions
- **Flow**: Auto-advance to Stage 6A only if ALL packages verified in skill files

**Error Handling: Missing Syncfusion Controls** ⚠️
- Error: `'sfDataGrid' does not exist in namespace...`
- Root cause: NuGet package NOT installed OR guessed package name used
- **Fix path**: Read control-mapping.json → Read skill file for exact package name → Install verified package → Verify with `MSBuild <solution>.sln`
- ❌ NEVER assume package names; always read the skill file first

**After Stage 6 completes:**
```
✅ Stage 6 Complete: All packages identified and verified
Packages ready: [list generated in Stage 6 output]

[STAGE 7] — Pre-Compilation Validation
Ready to validate your build with MSBuild?
→ MSBuild will compile XAML, verify C# code, and check namespaces

Continue to Stage 7? [YES / NO]
```

---

### Stage 7 — Validation (CRITICAL FIXES)

**Load**: `references/stage-7-validation.md` + `references/winui-dotnet-standards.md`

**⛔ PRE-CHECK 0 (STARTUP VIEW VALIDATION) — GATES ALL CHECKS:**
- Verify `App.xaml.cs`, `OnLaunched()`, startup window (`MainWindow` / `StartupWindow`), `window.Activate()` exist
- Verify `DataContext` assignment in window constructor
- ⛔ **If ANY missing → GENERATE fallback: empty MainWindow.xaml + MainWindow.xaml.cs with correct boilerplate**
- ✅ If all present → Proceed to Pre-Checks 1–6

**PRE-BUILD GATE (MANDATORY):**
- ✅ Verify Pre-Check 0 PASS (startup view exists and configured)
- ✅ Verify Pre-Check 1 PASS (skill-extraction.json validated)
- ⛔ If either FAIL → HALT (do NOT trigger MSBuild)

**STAGE 4 → STAGE 7 VERIFICATION:**
- Compare: Stage 4 planned startup view vs. Stage 7 generated files
- Verify: DataContext ViewModel matches planned MVVM mapping
- ✅ If match confirmed → Proceed to MSBuild

**EXECUTE MSBUILD (MANDATORY TRIGGER):**
```bash
# Resolve MSBuild path (VS2026 priority, then VS2022)
$msbuild = "C:\Program Files\Microsoft Visual Studio\18\Professional\MSBuild\Current\Bin\MSBuild.exe"
# If not found, try VS2022 path
if (!(Test-Path $msbuild)) { $msbuild = "C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\Bin\MSBuild.exe" }

# Execute build
& "$msbuild" <ProjectName>.sln /t:Build /p:Configuration=Debug /p:Platform="x64" /v:detailed /flp:logfile=build.log;verbosity=detailed
```
- **Build 0 errors** → Proceed to Phase 2 checks
- **Build fails** → Re-read skill file for that control → Fix → Retry (max 3 cycles)

**XAML DRY-RUN (simulate parse, do NOT compile):**
1. Parse XAML structure; validate all namespaces, resources, bindings
2. Fixable errors → apply fix → retry (max 5 attempts)
3. **PASS**: Parse succeeds AND MSBuild succeeds → Advance to Stage 8
4. **FAIL**: Non-fixable error or max attempts → Halt and report

**Critical Rule: Build Failures & Error Recovery** 🛑
- If `MSBuild` fails: **ALWAYS refer back to skill file FIRST** before any fix
- Verify: ✓ Startup view exists (Pre-Check 0) ✓ API names match skill file ✓ Namespaces correct ✓ NuGet version matches requirement
- ❌ **NEVER auto-fallback to Microsoft/WinUI default controls without explicit skill verification**
- **HALT conditions**: If Pre-Check 0 fails, if skill file missing, if Stage 4 plan ≠ Stage 7 generated code, if error persists after 3 cycles

---

### Stage 8 — Code Insertion
- Create directory structure inside project:
  - `<ProjectRoot>/Views/[ControlName]/`
  - `<ProjectRoot>/Models/`
  - `<ProjectRoot>/ViewModels/`
  - `<ProjectRoot>/Controls/`
- Insert all files; update `.csproj` references and imports.
- Run: `MSBuild <solution>.sln`
- **Output**: File paths + build success confirmation.

> ❌ NEVER create files outside `<ProjectRoot>`.

---

## Mandatory Steps

- Read each stage's reference file **before** executing that stage.
- Confirm Syncfusion control names (min. 3) before Stage 5.
- Confirm all 8 design system decisions before Stage 5.
- Never proceed past a FAIL GATE without resolving the failure.
- On any pipeline halt, load: `references/Build.md`

---

## DO ✅ and DON'T ❌ Guidelines

### DO ✅
- Use only Syncfusion WinUI controls.
- Use fallback only if no equivalent Syncfusion control exists.
- Read skill file fully before generating or fixing any control code.
- Follow documented patterns exactly as specified in skill files.
- Auto-fix where permitted; report and halt where not.
- Reference `Build.md` on any pipeline halt.

### DON'T ❌
- Use native XAML controls when a Syncfusion equivalent is available.
- Assume property names, binding syntax, or namespace strings from memory.
- Generate control code without reading the control skill file first.
- Skip stages or jump ahead without confirmation where required.
- Silently continue past a FAIL GATE.
- Create files outside `<ProjectRoot>`.

---

## Immediate Stop Actions

| Trigger | Action |
|---------|--------|
| Pre-Check 0 (startup view) fails in Stage 7 | **GENERATE fallback** MainWindow.xaml + .xaml.cs boilerplate → retry Pre-Check 0 |
| `MSBuild` fails | **STOP** — re-read skill file for failing control → fix → retry (max 3×) |
| Framework mismatch (Stage 2A) | **STOP** — report and ask user to clarify |
| Stage 4 plan ≠ Stage 7 generated | **STOP** — return to Stage 5-6 for regeneration |
| Stage 7 exceeds 5 parse attempts | **STOP** — report all errors, halt pipeline |
| Control skill file not found | **STOP** — halt; do NOT fallback to native WinUI |

> **NEVER USE** native XAML fallbacks without verifying Syncfusion equivalent is unavailable.  
> **NEVER GUESS** solutions. **NO TRIAL-AND-ERROR.**

---

## Mandatory Diagnostic Protocol

Run this protocol whenever `MSBuild` fails or a control has rendering / functionality issues:

1. **Error Identification** — Identify the exact error message and the failing control (e.g., `SfDataGrid`, `SfComboBox`).

2. **Skill File Consultation (mandatory full read)** — Locate and read the complete control skill file:
   ```
   <project-root>/{.codestudio|.agent|.agents|.github|skills}/syncfusion-winui-ui-builder/controls/{ControlName}.md
   ```
   Try all path variants until found.

3. **Validation Against Skill File** — Compare failing code against:
   - Required `using` statements
   - Correct NuGet package name
   - Required XAML namespaces
   - Correct property names and binding syntax
   - Required dependencies and known issues

4. **Skill-Based Correction Only** — Apply only fixes that are explicitly documented in the skill file. Do not modify code based on assumptions.

5. **Re-Verification Loop** — Run `MSBuild <solution>.sln` again. If it fails, return to Step 1. Max 3 cycles; if still failing after 3 cycles, halt and report.

---

## Error Recovery — Common Scenarios

| Scenario | Response |
|----------|----------|
| Lost stage context | State current progress; ask which stage to resume |
| User requests code before Stage 3/4 | Explain Stage 3 (control mapping) and Stage 4 (theming) are required first |
| Fewer than 3 Syncfusion control names | Require explicit listing before advancing |
| Design system not confirmed | Require MVVM + styling decisions before Stage 5 |
| Invalid user response | Re-ask the stage question or clarify intent |

---

## Tool Usage by Stage

| Stage | Tools |
|-------|-------|
| 1 | — |
| 2 | `read_file`, `grep_search` |
| 3 | `read_file`, `run_in_terminal` |
| 4 | `read_file` |
| 5A / 5B / 5 | `read_file`, `create_file` |
| 6 / 6A | `read_file` |
| 7 | `read_file` |
| 8 | `read_file`, `run_in_terminal`, `get_errors` |
| 9 | `create_file`, `run_in_terminal` |