# Stage 6: Dependencies

**Purpose:** Detect required NuGet packages from skill files, resolve versions, deduplicate, and prepare installation commands.

---

## ⛔ MANDATORY RULE: No Assumptions — Skill Files ONLY

**Before adding ANY NuGet package:**
1. ✅ Read `skill-extraction.json` (produced by Stage — Control Skill Extraction) to access pre-extracted package data
2. ✅ Verify all controls in `skill-extraction.json` have `validation_status: "PASS"`
3. ✅ Extract **exact** package name from `nuget_package` field (already verified in Stage — Control Skill Extraction)
4. ✅ Use **version from Stage 2 detection** (matching Syncfusion version in project)
5. ✅ Never assume, infer, or guess package names or versions

**Why this matters:**
- ❌ `syncfusion-winui-masked-TextBox` ≠ `Syncfusion.Core.WinUI` (actual: `Syncfusion.Editors.WinUI`)
- ❌ Guessing versions causes assembly mismatches and runtime failures
- ✅ Only `skill-extraction.json` (pre-validated) is authoritative for package resolution

**Rejection criteria:**
- ❌ Any control missing from `skill-extraction.json`
- ❌ Any entry with `validation_status != "PASS"`
- ❌ Any package NOT explicitly in `nuget_package` field
- ❌ Any version mismatch from Stage 2 detection

**Consequence:** Do not proceed with installation if package source cannot be verified in `skill-extraction.json`.

---

## ⛔ ERROR HANDLING: Missing Syncfusion Control ('does not exist in namespace...')

**Common error in Stage 5-6:**
- ❌ `'SfCombobox' does not exist in namespace 'using:Syncfusion.UI.Xaml.Core'`
- ❌ `XAML Compilation Error: Type X not found`

**Root cause:** NuGet package not installed OR package name guessed/assumed

**Mandatory fix:**
1. ✅ Read `control-mapping.json` to identify which controls are mapped
2. ✅ For each control, read the corresponding skill file (`getting-started.md`)
3. ✅ Extract EXACT NuGet package name (e.g., `Syncfusion.Editors.WinUI` for `SfCombobox`)
4. ✅ Install package using latest stable version from NuGet registry
5. ⛔ If package name is unclear or missing from skill file → HALT. Do NOT attempt to install guessed names
6. ⛔ DO NOT fallback to Microsoft native controls (e.g., `TextBox`, `ComboBox`) — Syncfusion skill file is authoritative

**Verification (Cross-Check Against Skill Files — NO BUILD):** After installation, execute the following checks WITHOUT running `dotnet build`:
```
FOR EACH package installed:
  1. FIND in skill-extraction.json → controls[].nuget_package field
     ❌ Not found → HALT: "Installed package not in skill-extraction.json"
  
  2. READ corresponding skill file (controls[].sources_read[0])
  
  3. EXTRACT declared namespaces from getting-started.md
     ❌ Namespace missing → HALT: "Namespace not documented in skill file"
  
  4. VERIFY .csproj PackageReference matches skill file version
     ❌ Version mismatch → HALT: "Version conflict — re-read skill file"

✅ ALL packages cross-checked and verified → Ready for Stage 7 MSBuild compilation
```
⛔ **DO NOT use `dotnet build` at this point.** Stage 7 uses MSBuild only for WinUI projects.

---

## 🔴 Stage 6 Entry Gate: Reject Non-Verified Controls

**BLOCKING check before any dependency resolution:**

```
IF skill-extraction.json missing:
  → ❌ HALT: "Stage — Control Skill Extraction not completed. Cannot identify NuGet packages."

FOR EACH control in skill-extraction.json:
  IF validation_status != "PASS":
    → ❌ HALT: "Control validation failed. Check Stage — Control Skill Extraction output."

  IF nuget_package == null OR nuget_package == "":
    → ❌ HALT: "NuGet package undefined for control. Skill file missing?"

ALL checks pass → ✅ Gate cleared. Proceed to Step 1.
ANY check fails → ❌ HALT. Do NOT proceed to dependency installation.
```

---

## 6-Step Dependency Workflow

### Step 1: Read `skill-extraction.json` & Identify Packages
- Load: `<project-root>/skill-extraction.json` (pre-validated from Stage — Control Skill Extraction)
- For each control entry:
  - Use `nuget_package` field directly (already extracted + verified in Stage — Control Skill Extraction)
  - Use `nuget_version` field to match project's Syncfusion version
- **Output:** Control → Official Package mapping (pre-verified, no re-extraction needed)

### Step 2: Scan Project .csproj
- Check existing Syncfusion packages and versions
- Identify framework target (net10.0-windows10.0.19041.0, etc.)
- List already-installed dependencies
- **Output:** Current project state

### Step 3: Resolve Versions

**Version detection priority (apply in order — stop at first success):**

```
1. Read Stage 2 output → syncfusion_version field
   IF version found AND non-empty → use it for ALL Syncfusion packages

2. Scan <ProjectName>.csproj for any existing Syncfusion package version
   IF found (e.g., <PackageReference Include="Syncfusion.Core.WinUI" Version="*" />)
   → extract that version → apply to ALL new Syncfusion packages

3. Query NuGet registry:
   dotnet package search Syncfusion.Core.WinUI --exact
   IF latest stable version returned → use it

4. IF version CANNOT be determined by any method above:
   ❌ Do NOT guess a version number
   ✅ Use version="*" — this resolves to the highest available stable version at install time

   Install command with wildcard:
   $ dotnet add package Syncfusion.SfDataGrid.WinUI

   (omitting --version lets NuGet resolve the latest stable automatically)
```

**Wildcard (`*`) rule:**

| Scenario | Version Strategy |
|---|---|
| Stage 2 version locked | Use exact version (e.g., `--version 33.2.10`) for ALL packages |
| Existing Syncfusion package found in `.csproj` | Extract and reuse that version for ALL packages |
| NuGet registry query succeeds | Use returned latest stable version |
| Version unknown — cannot determine | Omit `--version` flag (NuGet defaults to latest stable) |

> ⚠️ **Uniformity rule:** Once a version is resolved by any method, ALL Syncfusion packages in the project MUST use that same version. Mixing versions across packages causes assembly mismatch errors at runtime.
> ⚠️ **No guessing:** Never hardcode a version number that was not retrieved from Stage 2, the `.csproj`, or the NuGet registry. An unknown version is not a reason to invent one — it is a reason to use the wildcard strategy above.

- **Output:** Resolved version string (e.g., `33.2.10`) OR wildcard strategy confirmed with reason logged

### Step 4: Add Required Core Packages (Always)
- `Syncfusion.Core.WinUI` — foundational package
- `Syncfusion.Licensing` — license registration
- **Output:** Core dependencies confirmed

### Step 5: Deduplicate & Consolidate
- Remove duplicate package entries across controls
- Consolidate shared dependencies (e.g., `Syncfusion.Core.WinUI` used by multiple controls)
- List final unique packages with version
- **Output:** Final package list (no duplicates)

### Step 6: Prepare Installation Command
- Generate NuGet restore/install commands for new packages
- Include version for each package (matching Stage 2 version)
- Exclude already-installed packages
- **Output:** Ready-to-execute install command

---

## Validation Rules (MANDATORY)

| Check | Valid? | Action |
|-------|--------|--------|
| All package names verified in skill files? | ✅ Yes / ❌ No | Halt if not verified; re-read skill files |
| Version resolved (exact or wildcard)? | ✅ Yes / ❌ No | Use wildcard if version unknown — never guess a number |
| All Syncfusion packages same version? | ✅ Yes / ❌ No | Enforce uniform version; wildcard counts as uniform if no version known |
| Core packages included (Core, Licensing)? | ✅ Yes / ❌ No | Add missing core packages |
| No duplicate packages in final list? | ✅ Yes / ❌ No | Remove duplicates |
| Package versions compatible with framework? | ✅ Yes / ❌ No | Suggest upgrade or compatible version |

---

## Output Format

```
✓ Dependency Analysis

Skill File → NuGet Package Mapping:
  • syncfusion-winui-datagrid    → Syncfusion.SfDataGrid.WinUI (verified)
  • syncfusion-winui-maskedtextbox   → Syncfusion.Editors.WinUI (verified)
  • syncfusion-winui-combobox      → Syncfusion.Editors.WinUI (verified)

Core Dependencies (Required):
  • Syncfusion.Core.WinUI
  • Syncfusion.Licensing

Control Dependencies (From Skill Files):
  • Syncfusion.SfDataGrid.WinUI (new)
  • Syncfusion.Editors.WinUI (new)

Already Installed:
  • Syncfusion.Core.WinUI ✓

Conflicts: None

Install Command (dotnet CLI — version unknown, wildcard strategy):
  $ dotnet add package Syncfusion.Core.WinUI
  $ dotnet add package Syncfusion.Licensing
  $ dotnet add package Syncfusion.SfDataGrid.WinUI
  $ dotnet add package Syncfusion.Editors.WinUI
  ⚠️ Run `dotnet restore` then verify all resolved versions match in .csproj before proceeding
```

---

## Critical Rules

⚠️ **ALWAYS:**
1. Read skill files BEFORE assuming package names
2. If version cannot be detected from Stage 2, `.csproj`, or NuGet registry → omit `--version` flag (wildcard); never invent a version number
3. Enforce uniform Syncfusion version across all packages
4. Include ALL core packages (Core, Licensing) regardless of controls
5. Validate package names exactly match skill file documentation

⚠️ **RUNTIME ISSUE PREVENTION:**
- **Missing Syncfusion.Core.WinUI** → "Type initializer threw exception"
- **Missing Syncfusion.Licensing** → License registration fails, watermark appears
- **Version mismatch across Syncfusion packages** → "Type X in assembly Y does not match type in assembly Z"

---

## User Interaction

```
✓ All dependencies detected and validated
✓ No conflicts found
✓ Installation command ready

[✓ Install Now] [📋 Show Command] [⏭️ Skip for Later]
```

**Status:** Ready for Stage 7. User can install immediately or manually after code insertion in Stage 9.