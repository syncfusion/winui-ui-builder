# Stage 6: Dependencies

**Purpose:** Detect required packages, resolve version conflicts, prepare NuGet install command.

## ⚠️ CRITICAL: Check Skill Files BEFORE Adding NuGet Packages

**This step is MANDATORY before finalizing any NuGet package installation.**

### Step 0: Skill File Verification (REQUIRED)

**For every Syncfusion control used in the generated code, you MUST verify the correct NuGet package and namespace from the skill file BEFORE adding packages.**

1. **Locate Skill Files** (recursive search in order):
   - `.codestudio/skills/<skill-name>/SKILL.md`
   - `.agent/skills/<skill-name>/SKILL.md`
   - `.agents/skills/<skill-name>/SKILL.md`
   - `.github/skills/<skill-name>/SKILL.md`
   - `skills/<skill-name>/SKILL.md`
   - `.apm/Winui/skills/<skill-name>/SKILL.md`

2. **Extract Authoritative Information from EACH Skill:**
   - **NuGet Package Name**: The exact package that provides the control (e.g., `Syncfusion.UI.Xaml.Grid`, `Syncfusion.UI.Xaml.Charts`)
   - **Namespace**: The exact `using:` URI for XAML (e.g., `using:Syncfusion.UI.Xaml.DataGrid`)
   - **Version Requirements**: Any specific version constraints mentioned
   - **Additional Dependencies**: Any peer packages required (e.g., `Syncfusion.Licensing`)

3. **Cross-Reference with controls.csv:**
   - Verify the control exists in `scripts/controls.csv`
   - Use the `Syncfusion Control Name` column (e.g., `SfDataGrid`, `SfCalendar`)
   - Compare with the skill file's stated package

4. **Build Verification Query:**
   ```
   Skill: <skill-name>
   → NuGet Package: <package-name>
   → Namespace: <namespace>
   → Version: <if specified>
   ```

**Example Skill File Check:**

| Control | Skill File Location | NuGet Package | Namespace |
|---------|-------------------|---------------|-----------|
| SfDataGrid | `.apm/Winui/skills/syncfusion-winui-datagrid/SKILL.md` | Syncfusion.Grid.WinUI | Syncfusion.UI.Xaml.DataGrid |
| SfCalendar | `.apm/Winui/skills/syncfusion-winui-calendar/SKILL.md` | Syncfusion.UI.Xaml.Calendars | Syncfusion.UI.Xaml.Calendar |
| SfComboBox | `.apm/Winui/skills/syncfusion-winui-combobox/SKILL.md` | Syncfusion.UI.Xaml.Inputs | Syncfusion.UI.Xaml.Inputs |

**⚠️ WARNING: Do NOT assume NuGet package names**
- `SfDataGrid` → `Syncfusion.Grid.WinUI` (NOT `Syncfusion.UI.Xaml.Grid`)
- `SfCalendar` → `Syncfusion.UI.Xaml.Calendars` (NOT `Syncfusion.UI.Xaml.Calendar`)
- `SfComboBox` → `Syncfusion.UI.Xaml.Inputs` (NOT `Syncfusion.UI.Xaml.ComboBox`)
- Always verify from the skill file!

---

## Step 1: Detect Required Packages

**AI Should:**

1. **Scan generated code imports** and list all Syncfusion namespaces used
2. **Query EACH control's skill file** to get the authoritative NuGet package name
3. **List all Syncfusion.UI.Xaml.* packages** identified from skill files
4. **Check for other WinUI dependencies** (Windows App SDK 1.5.0+, .NET 6.0+, etc.)
5. **Identify any community toolkit or MVVM Framework packages** if used

---

## Step 2: Check Project's .csproj

- What packages already installed?
- What versions are currently in use?
- Any version conflicts?

---

## Step 3: Skill-Based Package Resolution

**Using the information extracted from skill files in Step 0:**

1. **If Syncfusion package already installed:**
   - Is version compatible? (All Syncfusion.UI.Xaml.* packages should match major.minor version)
   - Suggest upgrade if needed for security/features

2. **If peer dependencies conflict:**
   - Windows App SDK version must be 1.5.0+
   - .NET version must be 6.0+
   - Recommend resolution (upgrade, downgrade, or compromise)

3. **Critical Namespace Verification:**
   - Cross-reference the `SKILL.md` of each control to find the exact NuGet package that provides the namespace
   - Example: `SfAvatarView` requires `Syncfusion.UI.Xaml.Notifications`; `SfLinearProgressBar` requires `Syncfusion.UI.Xaml.ProgressBar`
   - Ensure the `using:` URI in XAML matches the detected NuGet package namespace

4. **Skill File Missing?**
   - If a skill file is not found for a control, fall back to `scripts/controls.csv` + official Syncfusion documentation
   - Check `https://www.syncfusion.com/winui-controls/<control-name-lowercase>` for authoritative package info

---

## Step 4: Prepare Installation Command

- Generate **MSBuild restore command** using **skill-verified package names** (PRIMARY)
- List packages to add with exact names from skill files
- List packages to upgrade (if needed)
- Provide fallback `dotnet` and `nuget` commands

**Example Output:**

```
✓ Dependency Analysis

Skill Verification Complete:
┌─────────────┬──────────────────────────────────────────────────────────┬─────────────────────────────────┬─────────────┐
│ Control     │ Skill File                                             │ NuGet Package                   │ Namespace   │
├─────────────┼──────────────────────────────────────────────────────────┼─────────────────────────────────┼─────────────┤
│ SfDataGrid  │ .apm/Winui/skills/syncfusion-winui-datagrid/SKILL.md   │ Syncfusion.Grid.WinUI           │ DataGrid    │
│ SfCalendar  │ .apm/Winui/skills/syncfusion-winui-calendar/SKILL.md    │ Syncfusion.UI.Xaml.Calendars    │ Calendar    │
│ SfComboBox  │ .apm/Winui/skills/syncfusion-winui-combobox/SKILL.md    │ Syncfusion.UI.Xaml.Inputs       │ Inputs      │
└─────────────┴──────────────────────────────────────────────────────────┴─────────────────────────────────┴─────────────┘

New Packages to Install:
  - Syncfusion.Grid.WinUI (verified from skill)
  - Syncfusion.UI.Xaml.Calendars (verified from skill)
  - Syncfusion.UI.Xaml.Inputs (verified from skill)

Existing Packages:
  ✓ Windows App SDK 1.8+ (compatible)
  ✓ .NET 8.0+ (compatible)
  ✓ Syncfusion.Core.WinUI (compatible)

Conflicts: None

INSTALL COMMAND (Choose ONE based on your environment):

PRIMARY - MSBuild (Recommended for WinUI projects):
$ msbuild YourProject.csproj /t:Restore /p:Configuration=Debug /v:minimal
$ msbuild /t:Build /p:Configuration=Debug /p:Platform=x64 /v:detailed YourProject.csproj

SECONDARY - NuGet CLI:
$ nuget restore YourProject.csproj
$ nuget install Syncfusion.Grid.WinUI -Version 33.2.6
$ nuget install Syncfusion.UI.Xaml.Calendars -Version 33.2.6
$ nuget install Syncfusion.UI.Xaml.Inputs -Version 33.2.6

TERTIARY - dotnet CLI (CI/CD only):
$ dotnet restore YourProject.csproj
$ dotnet add package Syncfusion.Grid.WinUI --version 33.2.6
$ dotnet add package Syncfusion.UI.Xaml.Calendars --version 33.2.6
$ dotnet add package Syncfusion.UI.Xaml.Inputs --version 33.2.6
```

**⚠️ IMPORTANT:** 
- Always use **MSBuild as primary** for WinUI projects (`/t:Restore` + `/t:Build`)
- Use the NuGet package names verified from skill files. Do NOT infer package names from control names (e.g., SfDataGrid → Syncfusion.Grid.WinUI, NOT Syncfusion.UI.Xaml.Grid)
- Use `dotnet` commands only as fallback for CI/CD environments

**User Interaction:**
User confirms package installation or does it manually:
```
Ready to install dependencies?
[Install] [Show Command] [Skip]
```

**Status:** AI detects and prepares. User decides whether to install now or later.

---

## ⚠️ SKILL FILE FIRST - Package Addition Rule

**MANDATORY WORKFLOW for EVERY package addition:**

1. **Query the skill file FIRST** (locations listed in Step 0)
2. **Extract NuGet package name** from skill file's "Required Packages" or "Getting Started" section
3. **Verify namespace** matches the skill file's documented namespace
4. **ONLY THEN** add the package to the project

**Why This Matters:**
- `SfDataGrid` requires `Syncfusion.Grid.WinUI` (NOT `Syncfusion.UI.Xaml.Grid`)
- `SfCalendar` requires `Syncfusion.UI.Xaml.Calendars` (NOT `Syncfusion.UI.Xaml.Calendar`)
- `SfComboBox` requires `Syncfusion.UI.Xaml.Inputs` (NOT `Syncfusion.UI.Xaml.ComboBox`)

**Inferring package names leads to:**
- ❌ Wrong package references
- ❌ Missing namespaces
- ❌ Build failures
- ❌ Runtime errors

**Always verify from the authoritative skill file before adding packages.**
