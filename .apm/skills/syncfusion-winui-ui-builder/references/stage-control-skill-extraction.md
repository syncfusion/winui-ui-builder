# Stage: Control Skill Extraction

**Purpose:** Extract, structure, and persist all required control data from skill reference files into `skill-extraction.json` before any code is generated. This artifact is the single, verified source of truth for namespaces, properties, events, and packages used throughout Stage 5.

**Trigger:** Runs immediately after the Stage 5 atomic validation gate clears (Step 2).
**Input:** `<project-root>/control-mapping.json` + skill reference files per control
**Output:** `<project-root>/skill-extraction.json` — fully validated artifact

⛔ **Stage 5 code generation does NOT begin until `skill-extraction.json` exists and `validation_status` = `"PASS"`.**

---

## Why This Stage Exists

| Failure Mode Without 5A | Consequence |
|---|---|
| Extracted data lives only in transient context | Code generation silently falls back to guessed namespaces |
| No shared artifact across deliverables | XAML, ViewModel, and Service files use inconsistent APIs |

`skill-extraction.json` eliminates both: all data is explicit, persistent, and locked before the first file is written.

---

## Input Sources (MANDATORY — All Must Be Read)

For each control in `control-mapping.json`, files are read from:
```
<skill-root>/references/
```
where `<skill-root>` = `<skills-root>/syncfusion-winui-<control-name>/`
and `<skills-root>` is one of: `.codestudio/skills`, `.agent/skills`, `.agents/skills`, `.github/skills`, `skills`

### Reference File Detection (DYNAMIC — NOT Fixed Filenames)

⛔ **Do NOT assume fixed filenames like `filtering.md`, `validation.md`, or `styling.md`.** These are content categories, not guaranteed filenames. Real files may be named differently (e.g., `data-filter.md`, `input-validation.md`, `theme-guide.md`).

**Step A — Always read first (if present):**
- `getting-started.md` — core namespace, base properties, events, NuGet package

**Step B — Scan ALL remaining files in `/references/` folder:**
- List every `.md` file in the directory
- Read each file regardless of its name
- Categorize by content using keyword detection:

| Category | Keyword Signals in File Content | What to Extract |
|---|---|---|
| **Core Setup** | `namespace`, `xmlns`, `NuGet`, `PackageReference`, `assembly` | Namespace, package name, base setup |
| **Filtering** | `filter`, `search`, `query`, `predicate` | Filter properties, filter events |
| **Validation** | `validation`, `error`, `invalid`, `required`, `rule` | Validation properties, error events |
| **Styling** | `style`, `appearance`, `brush`, `template`, `resource` | Style properties, resource keys |
| **Other** | Any file not matching above | Read fully — extract any properties, events, or methods found in code examples |

**Rules:**
- ✅ Read every `.md` file in `/references/` — no file is skipped based on name
- ✅ Categorize by content, not filename
- ✅ Extract properties, events, and methods from code examples in every file
- ❌ Do NOT skip a file because its name doesn't match an expected pattern
- ❌ Do NOT treat `getting-started.md` as the only mandatory source — all files in the folder are potential sources

---

## Extraction Workflow (Execute in Order)

---

### Step 1: Collect Controls from `control-mapping.json`

1. Read `<project-root>/control-mapping.json`
2. Collect every unique `control` value from `mapped_controls[]`
3. Skip entries where `control == "NATIVE_XAML"` — record in `native_xaml_controls[]`, no extraction needed
4. Build extraction list (e.g., `["SfDataGrid", "SfMaskedTextBox"]`)

⛔ `control-mapping.json` missing or invalid → HALT. Return to Stage 3.

---

### Step 2: Namespace Extraction (CRITICAL)

For each control, resolve the XAML namespace using this **strict priority order**:

```
1. Read getting-started.md → search for xmlns declaration in XAML examples
   IF exact using:<Namespace> declaration found → USE IT (highest priority)

2. Read getting-started.md → check for xmlns:syncfusion="using:Syncfusion.UI.Xaml.<ControlNamespace>"
   IF this namespace URI is the only namespace declared for this control → USE IT

3. Scan ALL other .md files in /references/ folder
   IF a more specific namespace is used in any file's examples → USE the more specific one

RESULT RULES:
  ✅ Use the namespace exactly as written in the skill file
  ✅ If both a specific using: namespace and a generic one appear → prefer the specific using: form (more explicit)
  ❌ Do NOT guess, infer, or construct a namespace from the control name
  ❌ Do NOT hardcode any namespace not found in a reference file
  ❌ If no namespace found in any file in /references/ → HALT: "Namespace undefined for <control-name>"
```

**Default Syncfusion WinUI namespace pattern** (use ONLY if explicitly declared in a skill file — not as a fallback assumption):
```
xmlns:syncfusion="using:Syncfusion.UI.Xaml.<ControlNamespace>"
```

---

### Step 3: Property & Event Extraction (MANDATORY)

Extract from **all** `.md` files found in `/references/` — determined by the dynamic scan, not by fixed filenames.

**Process per file:**
1. Locate all fenced XAML (` ```xaml `) and C# (` ```csharp `) code blocks in the file — do NOT extract from prose sentences, inline code mentions, or commented-out text
2. For each code block, identify the target control's opening tag (e.g., `<sf:SfMaskedTextBox ...>`)
3. Extract **only** attribute names that sit directly on the target control's own opening tag — do NOT extract attributes from child elements, parent elements, or sibling elements that appear in the same block
   - ✅ `<sf:SfMaskedTextBox Mask="Email">` → `Mask` belongs to `SfMaskedTextBox`
   - ❌ `<TextBox PlaceholderText="Enter email"/>` nested inside `<sf:SfMaskedTextBox>` → `PlaceholderText` belongs to `TextBox`, not `SfMaskedTextBox`
4. Extract every event name wired directly on the target control's tag in those code blocks
5. Extract every method called on the target control instance in C# code blocks (e.g., `myControl.MethodName(...)`)
6. **CRITICAL:** Property/event extraction sources:
   - ✅ ONLY from fenced code blocks (` ```xaml ` or ` ```csharp `)
   - ✅ Property must be used as an attribute on the control's tag in XAML
   - ✅ Property must be assigned to the control instance in C#
   - ❌ NEVER extract from plain text descriptions
   - ❌ NEVER extract from inline backtick mentions
   - ❌ NEVER extract from API tables unless the property also appears in a code example
   - ⛔ If a property name appears ONLY in prose text (not in any code block) → REJECT
7. Record which file each item was found in (`source` field)

**Rules:**
- ✅ Include properties, events, and methods found in any file in `/references/`
- ✅ **ONLY extract from fenced code blocks** (` ```xaml ` or ` ```csharp `) — **NEVER** from prose sentences, inline backtick mentions, plain text descriptions, or code comments
- ✅ Only extract attributes that appear directly on the target control's own opening tag — never from child, parent, or sibling elements in the same code block
- ✅ Property must be tied to actual control usage: XAML attribute or C# property assignment
- ✅ Copy names character-for-character (casing matters in XAML and C#)
- ✅ Mark which file each item was sourced from (aids debugging)
- ❌ Do NOT assume a property exists because it sounds logical
- ❌ Do NOT extract property names from plain text descriptions, even if they seem valid
- ❌ Do NOT include items only seen in third-party blog posts or non-skill sources
- ❌ Do NOT extract from API documentation tables unless the property also appears in a code example
- ❌ Do NOT stop after `getting-started.md` — scan every file in the folder
- ⛔ **HALT** if any property is extracted from plain text instead of code blocks

**Extraction Examples (MANDATORY — Follow This Pattern):**

✅ **VALID — Extract from XAML code block:**
```markdown
In the skill file, you find this XAML code block:
```xaml
<syncfusion:SfTextInputLayout Hint="Enter email" PasswordChar="*" />
```

**Extract:**
```json
"valid_properties": [
  { "name": "Hint", "source": "getting-started.md" },
  { "name": "PasswordChar", "source": "getting-started.md" }
]
```

✅ **VALID — Extract from C# code block:**
```markdown
In the skill file, you find this C# code block:
```csharp
textInputLayout.Hint = "Enter email";
textInputLayout.PasswordChar = "*";
```

**Extract:**
```json
"valid_properties": [
  { "name": "Hint", "source": "getting-started.md" },
  { "name": "PasswordChar", "source": "getting-started.md" }
]
```

❌ **INVALID — Do NOT extract from plain text:**
```markdown
In the skill file, you find this plain text:
"The Value property sets the input value. The Control supports custom validation."

**DO NOT EXTRACT:** ⛔ "Value" and "Control" are not in code blocks — they are plain text mentions.
```

**Attribute classification (MANDATORY — before recording any extracted item):**

Every attribute found on the target control's tag must be classified into exactly one of the following categories before being stored. This prevents events, attached properties, and markup extensions from polluting `valid_properties[]`.

| Attribute pattern | Classification | Store in |
|---|---|---|
| `PropertyName="value"` or `PropertyName="{x:Bind ...}"` | Direct property of the control | `valid_properties[]` |
| `OwnerClass.PropertyName="value"` (e.g., `Grid.Row="1"`) | Attached property — owner is `OwnerClass`, not the target control | Discard — do NOT add to `valid_properties[]` |
| `EventName="HandlerMethod"` (e.g., `Click="OnLogin"`) | Event | `valid_events[]` |
| `x:Name`, `x:Key`, `x:Uid` | XAML directive — not a control property | Discard |
| `xmlns:prefix="..."` | Namespace declaration | Namespace extraction only (Step 2) |

❌ Do NOT store attached properties (`Grid.Row`, `Canvas.Left`, `AutomationProperties.Name`) in `valid_properties[]` — they belong to a different owner class.
❌ Do NOT store event handler assignments in `valid_properties[]` — they belong in `valid_events[]`.

**Blocking validation for Step 3:**

| Condition | Action |
|---|---|
| Reference files exist in `/references/` but were not all scanned | ⛔ HALT: `"Incomplete reference analysis — not all files in /references/ were read"` |
| Extraction sourced only from `getting-started.md` when other files exist | ⛔ HALT: `"Partial extraction not allowed — re-scan all files in /references/"` |
| **Property extracted from plain text description instead of code block** | ⛔ HALT: `"Invalid extraction source for '<PropertyName>' — must be extracted from fenced code blocks (```xaml or ```csharp) only, NOT from plain text"` |
| **Property not found in any ```xaml or ```csharp code block** | ⛔ HALT: `"Property '<PropertyName>' for <ControlName> not found in any code example — plain text sources are not valid"` |
| Properties/events included without evidence from a fenced code block | ⛔ HALT: `"Unverified API extraction — must be sourced from fenced code blocks only"` |
| **Generic property name (e.g., "Value") extracted without control-specific context** | ⛔ HALT: `"Generic property '<PropertyName>' extracted without verifiable usage on <ControlName> — verify it appears as attribute on control tag in code block"` |
| Attached property stored in `valid_properties[]` | ⛔ HALT: `"Attached property '<OwnerClass.Name>' must not be stored in valid_properties for <ControlName>"` |
| Event handler name stored in `valid_properties[]` | ⛔ HALT: `"Event '<name>' stored in valid_properties — move to valid_events"` |

---

### Step 4: Validate Extracted Data (BLOCKING)

Run these checks immediately after extracting each control — before writing to the JSON file:

**4A — Deduplication (run before validation):**

After scanning all files for a control, deduplicate `valid_properties[]`, `valid_events[]`, and `valid_methods[]` by exact name (case-sensitive):
- If the same name appears from two different source files, keep the entry from the earliest-read file (`getting-started.md` wins over any feature guide)
- Log every collision: `"Duplicate property '<name>' for <ControlName> — keeping source '<earlier-file>', discarding '<later-file>'"`
- Never create two entries with the same `name` value in the same list

**4B — Blocking checks:**

| Condition | Halt Message |
|---|---|
| Control not in `control-mapping.json` | `"Control <name> not in mapping — extraction not requested"` |
| Skill folder not found | `"Skill folder missing for <control-name> — cannot extract"` |
| `getting-started.md` not found | `"Reference file missing for <control-name> — cannot extract"` |
| Namespace not found in any reference file | `"Namespace undefined for <control-name> — do not guess"` |
| `valid_properties` has fewer than 3 entries after deduplication | `"Insufficient properties for <control-name> — fewer than 3 verified; check all reference files"` |
| All entries in `valid_properties` are base-class properties only (`Visibility`, `Opacity`, `IsEnabled`, `Width`, `Height`, `Margin`, `Padding`) with no control-specific property | `"No control-specific properties found for <control-name> — extraction likely captured only inherited UIElement/FrameworkElement properties"` |
| NuGet package name not listed | `"Package undefined for <control-name> — do not assume"` |
| Duplicate name found in `valid_properties[]` after deduplication step | `"Deduplication failed for <control-name> — duplicate entry '<name>' still present"` |

---

### Step 5: Write `skill-extraction.json`

Save to `<project-root>/skill-extraction.json` after all controls pass Step 4.

#### Full Schema

```json
{
  "generated_at": "<ISO-8601 timestamp>",
  "resolved_syncfusion_version": "<version from Stage 2>",
  "controls": [
    {
      "control": "SfMaskedTextBox",
      "namespace": "using:Syncfusion.UI.Xaml.Editors",
      "namespace_source": "getting-started.md",
      "nuget_package": "Syncfusion.Editors.WinUI",
      "nuget_version": "Latest",
      "valid_properties": [
        { "name": "Mask",          "source": "getting-started.md" },
        { "name": "Description", "source": "customization.md" },
        { "name": "Header", "source": "customization.md" },
        { "name": "ErrorType",    "source": "error-indication.md" },
        { "name": "ErrorContent",     "source": "error-indication.md" }
      ],
      "valid_events": [],
      "valid_methods": [],
      "setup_instructions": "MaskedTextBox for validated text input with customizable mask patterns. Use this when implementing masked input fields, formatted data entry (phone numbers, dates, SSN, IP addresses).",
      "advanced_features_read": ["error-indication-validation.md", "events-valuechanged.md"],
      "sources_read": [
        ".codestudio/skills/syncfusion-winui-masked-textbox/references/getting-started.md",
        ".codestudio/skills/syncfusion-winui-masked-textbox/references/customization.md",
        ".codestudio/skills/skills/syncfusion-winui-masked-textbox/references/error-indication-validation.md",
        ".codestudio/skills/skills/syncfusion-winui-masked-textbox/references/error-indication.md",
        ".codestudio/skills/syncfusion-winui-masked-textbox/references/events-valuechanged.md"
      ]
    },
    {
      "control": "SfComboBox",
      "namespace": "using:Syncfusion.UI.Xaml.Editors",
      "namespace_source": "getting-started.md",
      "nuget_package": "Syncfusion.Editors.WinUI",
      "nuget_version": "Latest",
      "valid_properties": [
        { "name": "SfComboBoxItem",     "source": "getting-started.md" },
        { "name": "TextMemberPath",   "source": "getting-started.md" },
        { "name": "DisplayMemberPath",   "source": "getting-started.md" },
        { "name": "IsFilteringEnabled", "source": "filtering.md" },
        { "name": "IsEditable",    "source": "editing.md" }
      ],
      "valid_events": [   ],
      "valid_methods": [],
      "setup_instructions": "Use for single and multiple selection with filtering and token display. Bind Command for MVVM.",
      "advanced_features_read": ["filtering.md"],
      "sources_read": [
        ".codestudio/skills/syncfusion-winui-combobox/references/getting-started.md",
        ".codestudio/skills/syncfusion-winui-combobox/references/filtering.md",
        ".codestudio/skills/syncfusion-winui-combobox/references/editing.md"
      ]
    }
  ],
  "native_xaml_controls": [
    {
      "control": "NATIVE_XAML",
      "element_id": "remember_me",
      "equivalent_native": "Microsoft.UI.Xaml.Controls.CheckBox",
      "fallback_reason": "No Syncfusion checkbox control available"
    }
  ],
  "extraction_status": "COMPLETE",
  "validation_status": "PASS"
}
```

#### File Rules
- One entry per unique control — no duplicates
- `valid_properties`, `valid_events`, `valid_methods` use objects with `name` + `source` — never bare strings
- `sources_read[]` lists every `.md` file actually opened for this control — filenames are dynamic, not predetermined
- `advanced_features_read[]` records which non-`getting-started.md` files contributed data, identified by their actual filename (not assumed category name)
- `NATIVE_XAML` entries go in `native_xaml_controls[]` only — no namespace extraction

---

### Step 6: Validate `skill-extraction.json` (MANDATORY)

Run all checks before Stage 5 proceeds:

| # | Check | Fail Condition |
|---|---|---|
| 1 | All controls extracted | Any `control-mapping.json` entry missing from `controls[]` |
| 2 | No missing namespaces | Any entry with empty/null `namespace` |
| 3 | Namespace has a `namespace_source` | Any entry where `namespace_source` is missing or null |
| 4 | No missing packages | Any entry with empty/null `nuget_package` |
| 5 | Properties sourced | Any property object missing a `source` field |
| 6 | Source file cross-reference | Any property/event/method whose `source` value is NOT present in that control's `sources_read[]` array — indicates the file was never actually opened; ⛔ HALT: `"Property '<name>' on '<ControlName>' claims source '<file>' but that file is not in sources_read — re-extract"` |
| 7 | No duplicates | Any `name` value appearing more than once in `valid_properties[]`, `valid_events[]`, or `valid_methods[]` for the same control |
| 8 | Minimum property count | Any control with fewer than 3 entries in `valid_properties[]` after deduplication |
| 9 | Control-specific property present | Any control whose `valid_properties[]` contains only known base-class properties (`Visibility`, `Opacity`, `IsEnabled`, `Width`, `Height`, `Margin`, `Padding`, `HorizontalAlignment`, `VerticalAlignment`) with no Syncfusion-specific property |
| 10 | Version consistent | Any `nuget_version` not matching `resolved_syncfusion_version` |
| 11 | File is valid JSON | Any parse error (trailing comma, missing bracket, etc.) |
| 12 | `extraction_status` = `"COMPLETE"` | Status missing or set to partial/error |

**On any failure:**
- Report every failing check with control name + field
- ⛔ HALT — do not allow Stage 5 to begin
- Prompt: *"Re-read skill files for `<control-name>` and re-run Stage — Control Skill Extraction for that control only"*

**On full pass:**
- Set `validation_status: "PASS"` in the file
- Output: *"`skill-extraction.json` validated — all [N] controls verified. Proceeding to Stage 5 code generation."*

---

## Usage in Stage 5 (CRITICAL)

Stage 5 must read `skill-extraction.json` before generating any file. It must never derive namespaces, properties, or events from memory or training data.

### Lookup Pattern (mandatory for every deliverable)

```
READ <project-root>/skill-extraction.json
  ❌ File not found → HALT: "skill-extraction.json missing — run Stage — Control Skill Extraction first"
  ❌ validation_status ≠ "PASS" → HALT: "Extraction not validated — fix and re-run Stage — Control Skill Extraction"

FOR EACH control used in this deliverable:
  FIND controls[] entry where control == "<ControlName>"
  ❌ Not found → HALT: "No extraction entry for <ControlName> — re-run Stage — Control Skill Extraction"

  USE namespace               → add exact xmlns declaration to XAML
  USE valid_properties[].name → only these attribute names are valid in XAML
  USE valid_events[].name     → only these event names are valid in XAML / code-behind
  USE nuget_package           → pass to Stage 6 for dotnet add package
```

### Per-Deliverable Field Map

| Deliverable | Fields Used |
|---|---|
| XAML | `namespace` → `xmlns` prefix; `valid_properties[].name` → attributes |
| Code-behind | `valid_events[].name` → handler signatures; `setup_instructions` → constructor order |
| ViewModel | `valid_properties[].name` → bound property names; `valid_events[].name` → command triggers |
| Stage 6 (NuGet) | `nuget_package` + `nuget_version` → `dotnet add package` command |

---

## Blocking Conditions Summary

| Condition | Action |
|---|---|
| `skill-extraction.json` missing | ⛔ HALT Stage 5 — run Stage — Control Skill Extraction first |
| `validation_status` ≠ `"PASS"` | ⛔ HALT Stage 5 — fix and re-validate |
| Control needed in XAML has no entry | ⛔ HALT — re-extract that control |
| Namespace used in XAML not in entry | ⛔ HALT — namespace is unverified |
| Property used in XAML not in `valid_properties` | ⛔ HALT — property is unverified |
| Event used in XAML not in `valid_events` | ⛔ HALT — event is unverified |
| `namespace_source` missing | ⛔ HALT — namespace origin cannot be traced |
| Property `source` not in `sources_read[]` | ⛔ HALT — property was not sourced from a file that was actually read |
| Duplicate name in `valid_properties[]` or `valid_events[]` | ⛔ HALT — deduplication step was not completed correctly |
| Fewer than 3 entries in `valid_properties[]` | ⛔ HALT — re-read all reference files and re-extract |
| Only base-class properties in `valid_properties[]` | ⛔ HALT — extraction captured inherited properties only; no Syncfusion-specific APIs found |

---

## Optional: Validation Script Enhancement

If a `controls_search.cjs` script is used (Stage 3), update it to:
- Scan the entire `/references/` directory dynamically (not a fixed file list)
- Categorize files by content keywords (see Input Sources table above)
- Validate extracted APIs against all scanned files — not just `getting-started.md`
- Report which file each extracted property/event came from

---

## Stage — Control Skill Extraction Output Summary

| Artifact | Location | Consumed By |
|---|---|---|
| `skill-extraction.json` | `<project-root>/` | Stage 5 (all deliverables), Stage 6 (NuGet install) |

✅ All namespaces traced to a specific reference file — no guesses
✅ Properties and events extracted only from fenced code blocks — never from prose or comments
✅ Each property extracted only from the target control's own opening tag — child/sibling attributes excluded
✅ Every extracted item classified: direct property, event, or method — attached properties and XAML directives discarded
✅ Duplicates resolved before writing — earliest source wins; collisions logged
✅ Minimum 3 control-specific properties required — base-class-only extractions are blocked
✅ Every property's `source` cross-referenced against `sources_read[]` — unverifiable entries are blocked
✅ File categorization is content-based — real filenames may differ from expected patterns
✅ Single artifact shared across XAML, code-behind, ViewModel, and NuGet steps
✅ Re-runnable per control — one failed extraction does not require a full restart