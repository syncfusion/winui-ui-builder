# Stage 1: Intent Analysis

**Purpose:** Parse and validate the user's natural language request, identify WinUI control type and requirements, resolve ambiguities.

**AI Should:**
- Read the user's raw query carefully
- Identify primary intent: `generate_page`, `generate_usercontrol`, `generate_window`, or `modify_existing`
- Extract control type (e.g., "login form" → Page/LoginView, "product grid" → UserControl/ProductGrid, "settings dialog" → Window/SettingsDialog)
- Extract XAML/data binding requirements (e.g., "with MVVM" → ViewModel:enabled, "code-behind" → ViewModel:minimal)
- Extract Syncfusion WinUI controls needed (e.g., "data grid" → SfDataGrid, "chart" → SfChart, "scheduler" → SfScheduler)
- Identify target directory if specified (e.g., "in the Views folder" → targetDir:Views/)
- Identify required features (e.g., "with validation", "async data loading", "data binding")

**Ambiguity Resolution:**
If the request is unclear, ask ONE clarifying question. Examples:

| Ambiguous Input | Clarifying Question |
|---|---|
| "Build me a form" | "What kind of form? (login, registration, contact, data entry, multi-step)" |
| "Add a control" | "What WinUI control? (Page, UserControl, dialog, or Syncfusion control like SfDataGrid, SfChart)" |
| "Make it better" | "Which control and what aspect? (accessibility, MVVM structure, styling, performance)" |
| "Create a grid" | "Display local data or remote? Single-select or multi-select? With filtering/sorting?" |

**Output to User:**
One-line confirmation:
```
✓ Understood: Generating a dark-themed login form with "Remember Me" support and MVVM pattern.
Starting project detection...
```

**WinUI-Specific Intent Examples:**

| User Request | Intent | Controls | MVVM | Theme |
|---|---|---|---|---|
| "Create a login page" | generate_page | Page + UserControl | ViewModel required | Default |
| "Add a customer data grid" | generate_usercontrol | UserControl + SfDataGrid | ViewModel + INotifyPropertyChanged | Fluent |
| "Build a settings dialog" | generate_window | Window + Validation | ViewModel + data binding | Light |
| "Make a dashboard panel" | generate_usercontrol | UserControl + SfChart | ViewModel | Current theme |

**Reference:** For control type catalog, see stage-3-layout-analysis.md

**Status:** This stage requires NO user interaction for confirmation. AI decides intent based on pure reasoning.
