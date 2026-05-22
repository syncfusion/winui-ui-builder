# Stage 3: Layout Analysis & Control Mapping (Combined)

**Purpose:** Analyze user requirements, create optimal control-mapping.json, execute mapping script, and produce Syncfusion WinUI control assignments. **FULLY AUTOMATED WITH MANDATORY SCRIPT EXECUTION.**

---

## ⚠️ CRITICAL: Understanding Control Skill Names vs. Official NuGet Packages

**The `skill_hint` and `Skill Name` values in controls.csv (e.g., `syncfusion-winui-datagrid`, `syncfusion-winui-textbox`) are REFERENCE LABELS for BM25 semantic search purposes ONLY.** They are NOT official Syncfusion NuGet package names.

**Stage 3 generates control mappings using these reference labels.** 
**Stage 5-7 must convert these reference labels to official Syncfusion NuGet package names that are EXPLICITLY documented in:**
- The corresponding control skill file (`.apm/skills/syncfusion-winui-<name>/SKILL.md`), OR
- Official Syncfusion documentation at [Syncfusion WinUI Controls](https://www.syncfusion.com/winui-controls)

**Examples of correct NuGet packages (EXPLICITLY documented in official sources):**
- Control: `SfDataGrid` → Skill: `syncfusion-winui-datagrid` → Official NuGet: `Syncfusion.Grid.WinUI` (verified in official docs)
- Control: `SfChart` → Skill: `syncfusion-winui-chart` → Official NuGet: `Syncfusion.Charts.WinUI` (verified in official docs)
- Control: `SfTextBox` → Skill: `syncfusion-winui-textbox` → Official NuGet: `Syncfusion.Editors.WinUI` (verified in official docs)

**DO NOT use inferred package names.** Verify that EACH package name is EXPLICITLY documented in control skill files or official Syncfusion resources before use.

---

## Stage 3: Layout Analysis

### AI Should:

1. **Read control type** from Stage 1 intent analysis
2. **Analyze user query** for specific requirements and context
3. **Determine optimal layout variant** (no user choice needed)
4. **Create structured `control-mapping.json`** with all elements (MANDATORY - saved to project root)
5. **Execute controls_search.cjs script** to map elements to Syncfusion WinUI controls (REQUIRED - NOT OPTIONAL)
6. **Output results** in chat for Stage 4 theming + Stage 5 code generation

### Decision Framework

Based on user requirements, AI selects the **best** layout (not multiple options):

| Control Type | Decision Criteria | Best Variant |
|---|---|---|
| **Login Form** | Enterprise? 2FA needed? Social login? | Choose: Minimal/Standard/Advanced |
| **Data Table** | Read-only or editable? Export needed? | Choose: Simple/Interactive/Full-featured |
| **Dashboard** | Internal or customer-facing? Complexity? | Choose: Focused/Standard/Enterprise |
| **Navigation** | Mobile traffic high? Multi-level needed? | Choose: Simple/Sidebar/Progressive |
| **Form** | Single-step or multi-step? Validation level? | Choose: Basic/Standard/Advanced |

**Key Principle:** AI makes the optimal choice based on the user query context and best practices. No variant selection UI needed.

### Output: Structured JSON (for Stage 3-4 Control Mapping)

```json
{
  "control_type": "Login Form",
  "variant": "Standard",
  "elements": [
    {
      "element_id": "email_input",
      "element_name": "Email Address",
      "description": "Email field with validation",
      "type_hint": "text input email form validation",
      "icon_hint": "email envelope mail user input",
      "control": "",
      "skill": "",
      "score": 0,
      "validation": ""
    },
    {
      "element_id": "password_input",
      "element_name": "Password",
      "description": "Password field, masked",
      "type_hint": "text input password form masked",
      "icon_hint": "lock password secure",
      "control": "",
      "skill": "",
      "score": 0,
      "validation": ""
    },
    {
      "element_id": "remember_me",
      "element_name": "Remember Me",
      "description": "Keep me logged in",
      "type_hint": "checkbox form input",
      "icon_hint": "check checkbox remember",
      "control": "",
      "skill": "",
      "score": 0,
      "validation": ""
    },
    {
      "element_id": "submit_button",
      "element_name": "Submit",
      "description": "Login button",
      "type_hint": "button primary action submit cta",
      "icon_hint": "arrow right send proceed forward",
      "control": "",
      "skill": "",
      "score": 0,
      "validation": ""
    }
  ],
  "icon_elements": [
    {
      "element_id": "forgot_password_link",
      "element_name": "Forgot Password",
      "description": "Password reset link",
      "type": "link",
      "icon_hint": "help question info reset",
      "control": "",
      "skill": "",
      "score": 0,
      "validation": ""
    },
    {
      "element_id": "error_message",
      "element_name": "Error",
      "description": "Error indicator",
      "type": "icon",
      "icon_hint": "error warning alert",
      "control": "",
      "skill": "",
      "score": 0,
      "validation": ""
    }
  ]
}
```

**JSON Structure Details:**

- `control_type`: The UI control being built (e.g., "Login Form", "Data Table", "Dashboard")
- `variant`: Chosen variant based on user requirements (e.g., "Standard", "Advanced", "Minimal")
- `sections` (optional): For complex layouts, group elements into logical sections
  - `section_id`: Unique identifier (e.g., "header_section")
  - `section_name`: Display name (e.g., "Header")
  - `elements`: Array of elements within section
- `elements`: Array of control elements
  - `element_id`: Unique identifier (snake_case) - maps to output
  - `element_name`: Display name for UI - maps to output
  - `description`: What this element does
  - `type_hint`: UI element type for BM25 search (e.g., "text input", "button", "dropdown", "table", "chart")
  - `icon_hint`: Icon keywords for EJ2 icon mapping (e.g., "email envelope mail" for email input)
  - `control`: Syncfusion control name (populated by script, e.g., "MaskedTextBox")
  - `skill`: Skill reference label (populated by script, e.g., "syncfusion-winui-masked-textbox")
  - `score`: BM25 relevance score (populated by script, e.g., 13.24)
  - `validation`: Verification status (populated by script, e.g., "✓ VERIFIED in controls.csv")
- `icon_elements` (optional): Array of icon-only elements without Syncfusion components
  - `element_id`: Unique identifier (snake_case) - maps to output
  - `element_name`: Display name - maps to output
  - `description`: What this element does
  - `type`: Element type (e.g., "link", "icon", "spinner", "badge")
  - `icon_hint`: Icon keywords for EJ2 icon mapping
  - `control`: Syncfusion control name (populated by script)
  - `skill`: Skill reference label (populated by script)
  - `score`: BM25 relevance score (populated by script)
  - `validation`: Verification status (populated by script)

**Important:**
- `type_hint` is critical for Stage 4 ControlMapper BM25 search accuracy
- Keep descriptions concise and functional
- Use lowercase for `id` and `type_hint`
- Create `control-mapping.json` in project root (reused in Stage 4 & 5)
- Output minimal summary table in chat for visibility

### File Organization During Code Generation (Stage 5-8)

**Important:** All generated files will be organized inside your project directory structure:

**XAML Views (inside project):**
- Location: `<ProjectRoot>/Views/[ControlName]/[ControlName].xaml`
- Example: `MyWinUIApp/Views/LoginForm/LoginForm.xaml`

**Code-Behind (inside project):**
- Location: `<ProjectRoot>/Views/[ControlName]/[ControlName].xaml.cs`
- Example: `MyWinUIApp/Views/LoginForm/LoginForm.xaml.cs`

**Data Models (inside project):**
- Location: `<ProjectRoot>/Models/[ModelName].cs`
- Example: `MyWinUIApp/Models/LoginFormModel.cs`

**ViewModels (inside project, if MVVM is used):**
- Location: `<ProjectRoot>/ViewModels/[ViewModelName].cs`
- Example: `MyWinUIApp/ViewModels/LoginFormViewModel.cs`

**Reusable Controls (inside project):**
- Location: `<ProjectRoot>/Controls/[ControlName]/[ControlName].xaml`
- Example: `MyWinUIApp/Controls/CustomButton/CustomButton.xaml`

✅ **All files are created INSIDE the project directory where `.csproj` exists. Never outside.**

---

### Type Hint Best Practices

**For Header/AppBar Elements:**
- Always include `appbar` or `header` keyword in type_hint
- Examples:
  - Logo: `"image logo appbar header branding"` (not just `"image logo"`)
  - Notifications: `"icon button notification appbar header"`
  - User Menu: `"dropdown button menu user profile appbar header"`

**General Guidelines:**
- Use **compound keywords** - `"icon button notification"` scores better than `"notification"`
- Include **context keywords** - appbar/header/sidebar context improves matching
- Add **modifiers** - sortable, filterable, collapsible, paginated
- Reference **control keywords from CSV** - Use exact control keywords for best BM25 matching

**Scoring Guide:**
- Score **40+** = Excellent match (multiple keywords + context)
- Score **20-40** = Good match (several keywords)
- Score **<20** = Weak match (may fall back to NATIVE_HTML)
- Score **0** = No match (unrelated keywords)

### Icon Guidelines

**Icons in WinUI are handled natively via Segoe MDL2 Assets.**

**For Control Elements:**
- Use semantic keywords in instructions to guide icon selection in Stage 5.
- Icons are implemented as Segoe MDL2 Assets glyphs (e.g., `&#xE715;` for Mail).
- Examples for Stage 5 instruction:
  - Email input: Use "Mail" icon (Segoe MDL2)
  - Password input: Use "Lock" icon (Segoe MDL2)
  - Submit button: Use "ChevronRight" icon (Segoe MDL2)

**Icon-Only Elements:**
- Included in the layout as standard UI elements with functional descriptions.
- Resolved to native Windows glyphs during code generation.

---

## Complex Layouts (Multi-Section)

For dashboards, admin panels, or multi-section layouts:

```json
{
  "control_type": "Admin Dashboard",
  "variant": "Classic Admin Dashboard",
  "layout_grid": "2-column",
  "sections": [
    {
      "section_id": "header",
      "section_name": "Header",
      "description": "Fixed top navigation",
      "responsive": "fixed",
      "elements": [
        {
          "element_id": "logo",
          "element_name": "Company Logo",
          "description": "Brand logo in app bar",
          "type_hint": "image logo appbar header branding",
          "control": "",
          "skill": "",
          "score": 0,
          "validation": ""
        },
        {
          "element_id": "notification_bell",
          "element_name": "Notifications",
          "description": "Bell icon with count in header",
          "type_hint": "icon button notification appbar header",
          "control": "",
          "skill": "",
          "score": 0,
          "validation": ""
        },
        {
          "element_id": "user_menu",
          "element_name": "User Profile",
          "description": "User avatar dropdown menu",
          "type_hint": "dropdown button menu user profile appbar header",
          "control": "",
          "skill": "",
          "score": 0,
          "validation": ""
        }
      ]
    },
    {
      "section_id": "sidebar",
      "section_name": "Sidebar",
      "description": "Left navigation",
      "responsive": "collapsible",
      "elements": [
        {
          "element_id": "nav_menu",
          "element_name": "Navigation Menu",
          "description": "Main navigation",
          "type_hint": "sidebar navigation collapsible",
          "control": "",
          "skill": "",
          "score": 0,
          "validation": ""
        }
      ]
    },
    {
      "section_id": "main_content",
      "section_name": "Main Content",
      "responsive": "flexible",
      "elements": [
        {
          "element_id": "kpi_cards",
          "element_name": "KPI Cards",
          "description": "Metric cards displaying statistics",
          "type_hint": "card grid statistics dashboard metrics kpi",
          "control": "",
          "skill": "",
          "score": 0,
          "validation": ""
        },
        {
          "element_id": "data_grid",
          "element_name": "Users Table",
          "description": "Data table with sorting and filtering",
          "type_hint": "grid data grid table sortable filterable paging",
          "control": "",
          "skill": "",
          "score": 0,
          "validation": ""
        }
      ]
    }
  ]
}
```

---

## Stage 3-4 File-Based Workflow (TOKEN OPTIMIZATION)

### ⚠️ MANDATORY: Create control-mapping.json in Project Root

**Step 1: Create `control-mapping.json`** in project root (NOT in scripts folder)
```bash
# File: ./control-mapping.json (at workspace root in WinUI project)
```

**Step 2: Run ControlMapper Script** with control-mapping.json input
```powershell
cd .apm\skills\syncfusion-winui-ui-builder\scripts
node controls_search.cjs <absolute-path-to>\control-mapping.json
```

**Step 3: Capture Output in Chat Context**
- ✅ Script outputs control + icon mapping JSON (WinUI controls)
- ✅ Keep mapping results in conversation context ONLY (no file)
- ✅ Do NOT save script output to file
- ✅ Reference mapping in chat for Stages 4-5

### Workflow Benefits
| Aspect | Benefit |
|--------|---------|
| **Token Efficiency** | Create control-mapping.json once, reuse in script without re-passing JSON in chat |
| **Persistence** | `control-mapping.json` stays in project for version control + auditing |
| **Scriptability** | Script reads `control-mapping.json` directly from filesystem (faster) |
| **Clean Context** | Only control mapping output in chat (not redundant control-mapping.json) |
| **Reusability** | Stage 5 code generation can reference `control-mapping.json` if needed |

---

## Stage 3-4 Part 2: Control & Icon Picking (Script-Based)

### Input: Control-Mapping JSON from Stage 3

Receive the control-mapping.json created above with `icon_hint` fields for control elements and `icon_elements` for icon-only elements.

### ⚠️ MANDATORY: Verify Controls EXIST BEFORE Mapping

**CRITICAL VALIDATION BEFORE RUNNING SCRIPT:**

1. **Read Available Controls**: Open the script's `scripts/controls.csv` and verify that ALL controls to be mapped actually exist in the authoritative list
2. **Review Control Names**: Confirm exact control names (DataGrid, not SfDataGrid; TreeGrid, not SfTreeGrid, etc.)
3. **Check Skills Directory**: Verify that corresponding SKILL.md files exist in the project's skill directories:
   - `.codestudio/skills/<skill-name>/SKILL.md`
   - `.agent/skills/<skill-name>/SKILL.md`
   - `.agents/skills/<skill-name>/SKILL.md`
   - `.github/skills/<skill-name>/SKILL.md`
   - `skills/<skill-name>/SKILL.md`

If a control is NOT in `controls.csv`, it will be replaced with `NATIVE_XAML` during script execution.

### Processing: ControlMapper Script with Validation

**⚠️ MANDATORY STEPS (DO NOT SKIP):**

1. **Create** `control-mapping.json` at project root (e.g., `D:\skills-ui-Builder-update\testing-demo\vite-project\control-mapping.json`)
2. **VALIDATE**: Ensure all controls exist in `scripts/controls.csv` (script will auto-validate)
3. **Execute** script with ABSOLUTE PATH (Windows best practice)
4. **Review Validation Report**: Check script output for validation errors and NATIVE_XAML fallbacks
5. **Capture** script output in chat context (DO NOT save to file)
6. **Reference** control mapping in subsequent stages

**Execution (REQUIRED) - Windows:**

Use absolute path for reliable execution:
```powershell
cd <your-winui-project-root>\<skills-dir>\syncfusion-winui-ui-builder\scripts
node controls_search.cjs <your-winui-project-root>\control-mapping.json
```

**Replace placeholders:**
- `<your-winui-project-root>` = Your WinUI project's root directory (e.g., `d:\my-winui-app`)
- `<skills-dir>` = Skills directory name (configuration-specific, e.g., `.apm\skills`, `.codestudio\skills`, `.agents\skills`, `.github\skills`, or `skills`)

**Real example (with hidden config directory like .apm):**
```powershell
cd d:\my-winui-app\.apm\skills\syncfusion-winui-ui-builder\scripts
node controls_search.cjs d:\my-winui-app\control-mapping.json
```

**Real example (with hidden config directory like .codestudio):**
```powershell
cd d:\my-winui-app\.codestudio\skills\syncfusion-winui-ui-builder\scripts
node controls_search.cjs d:\my-winui-app\control-mapping.json
```

**Real example (with hidden config directory like .agents):**
```powershell
cd d:\my-winui-app\.agents\skills\syncfusion-winui-ui-builder\scripts
node controls_search.cjs d:\my-winui-app\control-mapping.json
```

**Real example (with visible skills directory):**
```powershell
cd d:\my-winui-app\skills\syncfusion-winui-ui-builder\scripts
node controls_search.cjs d:\my-winui-app\control-mapping.json
```

**Path Resolution Rules (for Script):**
- ✅ **Absolute paths work best** - Full path from C:\ or D:\ or any drive (most reliable)
- ✅ **Fully IDE-agnostic** - Works with ANY skills directory structure (`.codestudio/`, `.agents/`, `.github/`, `skills/`, or custom)
- ✅ **Editor-independent** - Not tied to specific IDE names or conventions
- ✅ Script automatically resolves relative paths from script location
- ✅ Script validates path exists before processing and shows full path if error occurs
- ❌ Avoid relative paths like `../control-mapping.json` (can cause "file not found" errors)

**Output Destination:**
- ✅ Control + Icon mapping JSON printed to terminal
- ✅ **Captured in chat context ONLY** (not saved to file)
- ✅ Used for Stage 4 theming + Stage 5 code generation

This automatically:
- Maps elements → Syncfusion controls (BM25 search)
- Maps icon_hint → EJ2 icons (BM25 search)  
- Produces Stage 4 output JSON with BOTH controls and icons
- **Results stay in conversation, not persisted to disk**

### How It Works

**CRITICAL 3-STAGE VALIDATION PROCESS:**

1. **Stage A - BM25 Search**: Query `type_hint` against 37+ Syncfusion WinUI controls in `scripts/controls.csv`
   - Tokenizes keywords and calculates semantic relevance
   - Returns top-1 control candidate
   - Score threshold: > 0 = valid match

2. **Stage B - Existence Verification** (NEW - MANDATORY):
   - **VALIDATE**: Verify that matched control actually exists in `controls.csv`
   - If control found in CSV → Mark as `✓ VERIFIED` and proceed to Stage 5
   - If control NOT found in CSV → Mark as `✗ NOT in controls.csv` and use NATIVE_XAML fallback

3. **Stage C - Skill Discovery Verification** (NEW - MANDATORY for Stage 5):
   - Verify that corresponding SKILL.md file exists in project's skill directories
   - Ensure skill files have correct namespace and API information
   - If skill not found → Report warning during Stage 5 code generation

**Fallback Rules:**
- No BM25 match → `NATIVE_XAML` (native WinUI: TextBox, Button, ProgressBar, etc.)
- BM25 match but NOT in CSV → `NATIVE_XAML` (invalid control, use native)
- CSV verified but SKILL.md missing → Use generic Syncfusion namespace from documentation

**Output**: Complete mapping with validation status for each control (verified, fallback, or error)

### BM25 Search Algorithm

The ControlMapper uses **BM25 (Best Matching 25)** for semantic WinUI control matching:

- **Data source**: `scripts/controls.csv` containing 37+ Syncfusion WinUI controls
- **Tokenizes** query and control keywords from CSV
- **Calculates** term frequency (TF) in each control's keywords
- **Calculates** inverse document frequency (IDF) across all 37 WinUI controls
- **Applies** BM25 formula for semantic relevance ranking
- **Returns** ranked results with scores

**Quality:** Only controls with score > 0 are matched; unmatched → `NATIVE_XAML` (native WinUI control)

### Output: Control & Icon Mapping with Validation

```json
{
  "control_type": "Login Form",
  "variant": "Standard",
  "validation": {
    "total_elements": 5,
    "controls_found": 4,
    "native_fallbacks": 1,
    "validation_errors": [
      "Element 'remember_me': Mapped control 'CheckBox' not in controls.csv - using NATIVE_XAML"
    ]
  },
  "mapped_controls": [
    {
      "element_id": "email_input",
      "element_name": "Email Address",
      "control": "MaskedTextBox",
      "skill": "syncfusion-winui-masked-textbox",
      "score": 13.24,
      "validation": "✓ VERIFIED in controls.csv"
    },
    {
      "element_id": "password_input",
      "element_name": "Password",
      "control": "MaskedTextBox",
      "skill": "syncfusion-winui-masked-textbox",
      "score": 12.87,
      "validation": "✓ VERIFIED in controls.csv"
    },
    {
      "element_id": "remember_me",
      "element_name": "Remember Me",
      "control": "NATIVE_XAML",
      "skill": null,
      "score": 0,
      "validation": "✗ CheckBox NOT in controls.csv - using NATIVE_XAML"
    },
    {
      "element_id": "submit_button",
      "element_name": "Submit",
      "control": "Button",
      "skill": null,
      "score": 10.89,
      "validation": "NATIVE_XAML - no Syncfusion match"
    }
  ]
}
```

**Validation Report Meanings:**

| Status | Meaning | Action |
|--------|---------|--------|
| ✓ VERIFIED in controls.csv | Control found and confirmed to exist | Use Syncfusion control + skill |
| ✗ NOT in controls.csv | Control matched by BM25 but doesn't exist | Use NATIVE_XAML equivalent |
| No Syncfusion match | No BM25 match found in CSV | Use NATIVE_XAML (TextBox, Button, etc.) |

**Error Analysis:**
- If you see many `native_fallbacks`, it means the controls requested are not available in Syncfusion WinUI
- Review `scripts/controls.csv` to see what Syncfusion WinUI controls ARE available
- Adjust your layout requirements in Stage 3 to use only available Syncfusion controls

---

## Status

✅ **FULLY AUTOMATED** - No user interaction
✅ **Single pass** - control-mapping.json created once, controls + icons mapped immediately
✅ **Token efficient** - No duplication or variant selection overhead
✅ **Data-driven** - BM25 semantic search on 76+ Syncfusion controls + 20+ EJ2 icons
✅ **Icon Support** - Integrated icon mapping for both control elements and icon-only elements
✅ **Ready for Stage 5** - Control + icon mapping feeds directly to code generation

---

## Architecture

- **Input**: User requirements + control type from Stage 1
- **Processing**: 
  - Control analysis → JSON structure with `type_hint` + `icon_hint`
  - ControlMapper (BM25 algorithm on 76 controls)
  - IconMapper (BM25 algorithm on 20+ EJ2 icons)
- **Output**: 
  - `control-mapping.json` (project root) - layout structure + icon hints for Stage 4 & 5
  - Chat summary table - element count, controls matched, icons matched
   "Icons Selected: [name1], [name2], [name3]"
- **Data sources**: 
  - `scripts/controls.csv` (Syncfusion WinUI controls)
  - `scripts/controls_search.cjs` (BM25 semantic search engine)
- **Artifacts**: `control-mapping.json` (persistent artifact, reused by Stage 5)
- **Context**: Control + icon mapping results kept in conversation (no file artifact)

---

## Stage 3-4 Icon Integration Summary

| Step | Task | Input | Output | Artifact |
|------|------|-------|--------|----------|
| **Stage 3** | Analyze requirements, create control-mapping.json with icon hints | User requirements + control type | `control-mapping.json` with element + icon hints | ✅ `control-mapping.json` |
| **Stage 4** | Map elements to Syncfusion controls + icons (script-based) | `control-mapping.json` | Control + icon mapping results | Context only (no file) |
| **Stage 5** | Generate code with controls + icons | `control-mapping.json` + control mapping from context | WinUI `.xaml` with icons + styling | ✅ Control files |
