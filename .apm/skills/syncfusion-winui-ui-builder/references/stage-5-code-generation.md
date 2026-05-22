# Stage 5: Code Generation

**Purpose:** Generate production-ready WinUI code, XAML, and C# interfaces with accessibility and desktop standards compliance.

## CRITICAL: Read Control Skills BEFORE Code Generation

**THIS STEP IS NOT OPTIONAL - Must be completed before writing any code**

### Step 0: Validate Control Mapping from Stage 3

**BEFORE proceeding, check the validation report from Stage 3:**

1. **Review validation errors**: If `native_fallbacks` > 0, check what controls were not found
2. **Verify no invalid controls**: Ensure no `✗ NOT in controls.csv` controls are being used
3. **If errors exist**: Either:
   - Accept NATIVE_XAML fallbacks and proceed, OR
   - Go back to Stage 3 and redesign layout using only verified Syncfusion controls

### Step 1: Identify All Control Skills from Stage 3

From Stage 3 output, extract **ONLY the verified controls** with validation status = `✓ VERIFIED`:

- Example: `syncfusion-winui-datagrid`, `syncfusion-winui-chart`, etc. (from `skill` field)
- Skip any controls marked as `NATIVE_XAML`

### Step 2: Robust Skill Discovery & API Verification (MANDATORY for EACH Control)

**For every single control skill identified in Step 1, verify the API and implementation details before writing any code.**

1. **Locate `SKILL.md` (Recursive Search)**: Search for the control skill definition in these locations (in order):
   - `.codestudio/skills/<skill-name>/`
   - `.agent/skills/<skill-name>/`
   - `.agents/skills/<skill-name>/`
   - `.github/skills/<skill-name>/`
   - `skills/<skill-name>/`
   
2. **Verify API Support (CRITICAL)**:
   - **DO NOT assume property names.** Read the `SKILL.md` or `references/` guides to check property existence.
   - Example: Verify control properties (like mapping paths or data series types) against the authoritative skill definitions.
   - If a property or type is documented as part of a specific assembly (e.g., Charts, Notifications, etc.), ensure the namespace and assembly match exactly.

3. **Extract Authoritative Namespaces & Assemblies**:
   - **MANDATORY**: Open the control's `SKILL.md` or `getting-started.md` and copy the **exact** `xmlns` URI and C# `using` statement.
   - **Local Resources**: Always include the local project namespace (e.g., `xmlns:local="using:ProjectName"`) to resolve local types and views.
   - **Common Converters**: Ensure standard XAML converters (like `StringToVisibilityConverter`, `StringFormatConverter`) are defined in the `Resources` section of the control or `App.xaml` before use.
   - **Do NOT assume generic namespaces.** Many controls use specialized sub-namespaces (e.g., `Syncfusion.UI.Xaml.DataGrid` for DataGrid, `Syncfusion.UI.Xaml.Charts` for Chart).
   - Verify if the namespace requires a specific suffix (e.g., `.DataGrid` vs `.Grid`).
   - XAML Example: `xmlns:grid="using:Syncfusion.UI.Xaml.DataGrid"` ✅
   - Identify the specific NuGet package name required for the control.

### Step 3: Virtual Build & API Validation (Static Analysis)

Since WinUI build commands may not be available in all terminal environments, you MUST perform **Static API Verification**:
- **Cross-Reference**: Compare the proposed control usage against the `SKILL.md` API list AND the **`scripts/controls.csv`** file.
- **Fail Early**: If a requested feature (e.g., `SfLinearProgressBar`) is not found in `scripts/controls.csv`, report it immediately as a missing dependency or unsupported Syncfusion control. Use a `NATIVE_XAML` equivalent (e.g., `<ProgressBar />`) instead.
- **Self-Correction**: If you find an error in namespace or property names during your internal review (Stage 6), revert and re-read the control skill specifically for that API signature.

### Step 4: NOW Generate Code Using Extracted Information

Only after completing Steps 1-5, generate the .xaml/.xaml.cs file using the exact imports and styles extracted from control skills.

**Common Mistake to Avoid:**
❌ Guessing property names like `DisplayMemberPath` without checking if the control (e.g., `SfTreeView`) supports it.
❌ Using generic namespaces like `Syncfusion.UI.Xaml` when specialized ones like `Syncfusion.UI.Xaml.ProgressBar` are required.
✅ Read the skill documentation FIRST to verify property availability and namespace accuracy.

**Why This Order Is Critical:**
- Prevents "Type not found" errors by ensuring correct assembly references.
- Prevents "Undefined namespace" errors by using authoritative `using:` URIs.
- Prevents "Property not found" errors by verifying API support before implementation.
- Ensures compatibility with the specific version of Syncfusion WinUI controls in use.

---

## Code Generation Process

**After reading all control skills, AI Should:**

1. **Generate .xaml/.xaml.cs control file**:
   - WinUI control class with dependency properties
   - Proper Syncfusion imports and namespaces
   - C# interface for properties
   - Event handlers and state management
   - Error handling and validation
   - WCAG 2.1 AA accessibility markup (AutomationProperties, semantic structure, focus management)
   - XML documentation comments explaining usage

2. **Generate XAML styling** (based on project preference):
   - XAML Style: ResourceDictionary with Style definitions
   - Fluent Design: Style-based theming
   - Inline: Direct property binding in XAML
   - Responsive design: Desktop-first (1920px, 1366px, 1024px, 768px)
   - Light/dark theme support if needed

3. **Generate C# classes and interfaces**:
   - Properties interface with all property types
   - ViewModel types if using MVVM
   - Event handler signatures

4. **Reference code standards** from:
   - winui-standards.md (accessibility + security rules)
   - Control skill's feature-specific guides (filtering.md, validation.md, styling.md, etc.)

**Code Generation Standards:**

- **Project Root Constraints**: ALL files (Views, Models, ViewModels, Controls) MUST be generated INSIDE the project directory containing the `.csproj` file. NEVER create files outside the project root.
- **File Organization (MANDATORY)**: 
  - **View Files** → `<ProjectRoot>/Views/[ControlName]/[ControlName].xaml` and `[ControlName].xaml.cs`
  - **Model Files** → `<ProjectRoot>/Models/[ModelName].cs`
  - **ViewModel Files** → `<ProjectRoot>/ViewModels/[ViewModelName].cs`
  - **Reusable Controls** → `<ProjectRoot>/Controls/[ControlName]/[ControlName].xaml` and `[ControlName].xaml.cs`
  - **Example Path Structure**: 
    ```
    MyWinUIApp/                          (Project Root - contains .csproj)
    ├── Views/
    │   └── LoginForm/
    │       ├── LoginForm.xaml           ✅ Inside project
    │       └── LoginForm.xaml.cs        ✅ Inside project
    ├── Models/
    │   └── LoginFormModel.cs            ✅ Inside project
    ├── ViewModels/
    │   └── LoginFormViewModel.cs        ✅ Inside project
    └── MyWinUIApp.csproj
    ```
- **Control Imports:** Use exact import syntax from control skill's getting-started.md
  - WinUI XAML: `xmlns:syncfusion="using:Syncfusion.UI.Xaml.DataGrid"`
  - Do NOT use WPF clr-namespace syntax
- **Style Imports:** Include the Syncfusion built-in themes (Light/Dark) from `references/syncfusion-themes.md`
  - **Read:** `.codestudio/skills/syncfusion-winui-ui-builder/references/syncfusion-themes.md`
  - WinUI has 2 built-in themes (Light/Dark) that auto-apply based on Windows theme setting
  - No separate theme packages needed (unlike WPF)
- **Semantic XAML:** Use proper WinUI elements (`StackPanel`, `Grid`, `TextBlock`, `Button`, etc.)
- **Accessibility:** AutomationProperties, semantic structure, AutomationProperties.HelpText, focus management (per WCAG 2.2 AA standards)
- **C#:** No dynamic types, full type safety, INotifyPropertyChanged for ViewModel binding
- **Error Handling:** Try-catch blocks, user-friendly error messages, input validation
- **Responsive:** Grid/StackPanel layouts with AdaptiveTrigger for responsive breakpoints
- **Performance:** Virtualization for large lists, async/await for data operations, event handler cleanup
- **Security:** No reflection vulnerabilities, validate all inputs, no hardcoded secrets, parameterized data access
- **Comments:** XML documentation on control classes, explain complex logic

### ⚠️ CRITICAL: Theme Resource Initialization (REQUIRED BEFORE RUNTIME)

**Problem:** Syncfusion WinUI controls require theme resources to be loaded before XAML rendering. Missing theme resources cause runtime errors like:
```
Cannot find a Resource with the Name/Key BaseStyle [Line: 72 Position: 91]
```

**Solution: Add Theme Resources to App.xaml**

Your generated controls MUST have proper theme setup in `App.xaml`. Ensure the following is configured:

**1. Syncfusion Theme Registration in App.xaml.cs:**

```csharp
// App.xaml.cs
using Microsoft.UI.Xaml;
using Syncfusion.Licensing;

public partial class App : Application
{
    public App()
    {
        // ✅ REQUIRED: Register Syncfusion license (use trial if no license)
        SyncfusionLicenseProvider.RegisterLicense("YOUR_LICENSE_KEY_HERE");
        
        // ✅ REQUIRED: Set application theme (respects Windows preference)
        this.RequestedTheme = ApplicationTheme.Default;
        
        this.InitializeComponent();
    }

    protected override void OnLaunched(Microsoft.UI.Xaml.LaunchActivatedEventArgs args)
    {
        m_window = new MainWindow();
        m_window.Activate();
    }

    private Window m_window;
}
```

**2. Resource Dictionary in App.xaml:**

```xaml
<?xml version="1.0" encoding="utf-8"?>
<Application
    x:Class="MyApp.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:MyApp">
    
    <Application.Resources>
        <!-- ✅ REQUIRED: Syncfusion theme resources are auto-loaded via RequestedTheme -->
        <!-- No manual theme resource loading needed - built-in themes handle it -->
        
        <!-- Optional: Add custom application-level resources here -->
        <ResourceDictionary>
            <!-- Your custom styles and brushes -->
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

**3. What Gets Loaded Automatically:**

When you set `RequestedTheme = ApplicationTheme.Default` (or Light/Dark), Syncfusion controls automatically load:
- ✅ `BaseStyle` resource (required by all Syncfusion controls)
- ✅ Color palettes and brushes
- ✅ Typography resources
- ✅ Control-specific styles

**No additional manual resource dictionary merging is required** (unlike WPF).

**4. If BaseStyle Error Still Occurs:**

This indicates the Syncfusion theme resources package is not installed. Ensure you have:
- ✅ `Syncfusion.UI.Xaml.Core` NuGet package installed (provides theme resources)
- ✅ Version matches other Syncfusion packages (e.g., all 25.1.35)
- ✅ License registered before any Syncfusion control is rendered

**Validation Checklist:**
- [ ] `SyncfusionLicenseProvider.RegisterLicense()` called in App.xaml.cs
- [ ] `Application.RequestedTheme` set to Light/Dark/Default
- [ ] `Syncfusion.UI.Xaml.Core` installed via NuGet
- [ ] All Syncfusion packages at same version
- [ ] No custom theme resources conflicting with Syncfusion resources

### Media (MANDATORY)

- **Placeholder Images:** Use [Unsplash](https://unsplash.com) for high-quality placeholder images
  - Format: `https://images.unsplash.com/photo-[id]?w=[width]&h=[height]&fit=crop`
  - Example: `https://images.unsplash.com/photo-1560472354-b33ff0c44a43?w=200&h=100&fit=crop`
  - Always specify dimensions (width x height) in the URL
  - Use relevant keywords for context-appropriate images

### Icon Handling (MANDATORY)

**Principle:** Icons in WinUI are implemented using the native **Segoe MDL2 Assets** font family. This is the Windows platform standard.

**Implementation Steps:**
1. **Identify Semantic Need**: Look at the element description (e.g., "Save button").
2. **Find Glyph**: Use common Segoe MDL2 glyphs:
   - Save: `&#xE74E;`
   - Settings: `&#xE713;`
   - Contact/User: `&#xE77B;`
   - Mail: `&#xE715;`
3. **XAML Syntax**:
   ```xaml
   <TextBlock FontFamily="Segoe MDL2 Assets" Text="&#xE715;"/>
   ```

**Fallbacks**: 
- If a specific glyph is not known, use an appropriate emoji or a generic placeholder like `&#xE10F;` (Edit).
- Always ensure icons are wrapped in a control that provides accessible labels (AutomationProperties.Name).

### Button Sizing with Syncfusion (MANDATORY)

**Principle:** Let Syncfusion own button dimensions and styling. Use XAML resources only for layout around buttons.

❌ **INCORRECT** - Overriding Syncfusion button sizes with XAML margins:
```xaml
<Button x:Name="PlayButton" Margin="16" Padding="16" Height="48" Width="128">
  <TextBlock FontFamily="Segoe MDL2 Assets" Text="&#xEA74;"/>
  Play
</Button>
```

✅ Correct Layout - Syncfusion owns button, XAML owns layout:
```xaml
<StackPanel Orientation="Horizontal" Spacing="12">
  <syncfusion:ButtonControl x:Name="PlayButton" Content="Play" Size="Large" IsPrimary="True">
    <TextBlock FontFamily="Segoe MDL2 Assets" Text="&#xEA74;"/>
  </syncfusion:ButtonControl>

  <syncfusion:ButtonControl x:Name="InfoButton" Content="More Info" Size="Large">
    <TextBlock FontFamily="Segoe MDL2 Assets" Text="&#xE946;"/>
  </syncfusion:ButtonControl>
</StackPanel>
```

**Why This Works:**
- Syncfusion defines sizing + alignment internally
- Icons align correctly with Syncfusion's design system
- No margin collision or override conflicts
- Consistent appearance across all Syncfusion controls

---

### Control Reuse Across UI (Same Control, Multiple Places)

**Principle:** One Syncfusion control type can be reused throughout your UI with customizations. For example, `ButtonControl` can serve as the Login button, Forgot Password link, and Sign Up button—each customized via properties and styles.

**Example - Button Used in Multiple Places:**
```xaml
<!-- LoginForm.xaml -->
<Grid>
  <!-- Primary button - main CTA -->
  <StackPanel x:Name="LoginButtonPanel" Spacing="8">
    <syncfusion:ButtonControl x:Name="LoginButton" Content="Login" Size="Large" IsPrimary="True" Width="100%"/>
  </StackPanel>

  <!-- Flat button - link-style action -->
  <StackPanel x:Name="ForgotPasswordPanel" Spacing="8">
    <syncfusion:ButtonControl x:Name="ForgotPasswordButton" Content="Forgot Password?" Style="{StaticResource FlatButtonStyle}" Size="Small"/>
  </StackPanel>

  <!-- Outline button - secondary action -->
  <StackPanel x:Name="SignUpPanel" Spacing="8">
    <syncfusion:ButtonControl x:Name="SignUpButton" Content="Sign Up Here" IsPrimary="False" Style="{StaticResource OutlineButtonStyle}"/>
  </StackPanel>
</Grid>
```

```xaml
<!-- LoginForm.xaml.cs or Resources -->
<!-- Primary button - main CTA -->
<Style x:Key="PrimaryButtonStyle" TargetType="syncfusion:ButtonControl">
  <Setter Property="Width" Value="Auto"/>
  <Setter Property="Background" Value="#0d6efd"/>
</Style>

<!-- Flat button - link-style action -->
<Style x:Key="FlatButtonStyle" TargetType="syncfusion:ButtonControl">
  <Setter Property="Background" Value="Transparent"/>
  <Setter Property="BorderThickness" Value="0"/>
  <Setter Property="Foreground" Value="#6c757d"/>
  <Setter Property="FontSize" Value="14"/>
</Style>

<!-- Outline button - secondary action -->
<Style x:Key="OutlineButtonStyle" TargetType="syncfusion:ButtonControl">
  <Setter Property="Background" Value="Transparent"/>
  <Setter Property="BorderBrush" Value="#6c757d"/>
  <Setter Property="Foreground" Value="#6c757d"/>
</Style>
```

---

### Reading Control Skills BEFORE Using generate code (MANDATORY)

**CRITICAL:** Do NOT assume control properties or APIs.

**Required Process:**
1. **Identify all mapped controls** from Stage 3 output
   - E.g., GridControl, ChartControl, SidebarControl, etc.

2. **For EACH control**, read the control skill:
   - Location: `.codestudio/skills/<control-skill>/references/getting-started.md`
   - Extract: imports, style imports, required properties, setup code
   - Read: feature-specific guides (filtering, sorting, validation, styling, etc.)

3. **DO NOT generate code without reading** control skill documentation
   - Don't assume property names or API structure
   - Don't guess at event handler names
   - Don't skip required setup or initialization

**Example - Reading GridControl Skill:**
```
Before generating code:
1. Read: .codestudio/skills/syncfusion-winui-grid/references/getting-started.md
   → Extract: using Syncfusion.UI.Xaml.Grids;
   → Read: required properties, ItemsSource structure, column definitions

2. Read: .codestudio/skills/syncfusion-winui-grid/references/sorting.md
   → Understand: AllowSorting property, SortColumnDescriptions structure

3. Read: .codestudio/skills/syncfusion-winui-grid/references/filtering.md
   → Understand: AllowFiltering property, FilterPredicates structure

4. NOW generate code with correct imports, properties, and API calls
```

**What Control Skills Contain:**
- ✅ Authoritative import statements
- ✅ Complete API documentation
- ✅ Feature-specific patterns (sorting, filtering, validation)
- ✅ Best practices and performance considerations
- ✅ Accessibility requirements
- ✅ Theme customization options

**Common Mistakes to Avoid:**
- ❌ Guessing property names → Read skill documentation
- ❌ Missing style imports → Extract from getting-started.md
- ❌ Wrong event handler names → Copy from control skill examples
- ❌ Incomplete setup code → Follow skill's recommended initialization

---

**Example Output Files:**

```
controls/LoginForm/
  ├── LoginForm.xaml             (XAML control)
  ├── LoginForm.xaml.cs          (Code-behind)
  └── LoginForm.cs               (ViewModel export)
```

---

**Control Structure for Complex UIs:**

For UIs with multiple distinct sections of a Window/Page (e.g., Header, Sidebar, Main Content, Footer), split into separate Views and ViewModels per section. Each section gets its own folder with its `.xaml`, `.xaml.cs`, and `.cs` ViewModel. **Each section folder MUST implement the appropriate Syncfusion WPF controls for that section** — do not mix section responsibilities. Create a parent Window that composes these section Views. Do not collapse multiple distinct UI sections into a single file.

**Example Output Files (Complex UI):**

```
Controls/Dashboard/
├── Views/
│   ├── DashboardWindow.xaml          # Parent window composing all sections
│   ├── DashboardWindow.xaml.cs
│   ├── DashboardViewModel.cs         # Parent ViewModel coordinating sections
│   ├── Header/
│   │   ├── HeaderView.xaml           # Implement using Syncfusion AppBar
│   │   ├── HeaderView.xaml.cs
│   │   └── HeaderViewModel.cs
│   ├── Sidebar/
│   │   ├── SidebarView.xaml          # Implement using Syncfusion NavigationView
│   │   ├── SidebarView.xaml.cs
│   │   └── SidebarViewModel.cs
│   ├── MainContent/
│   │   ├── MainContentView.xaml      # Implement using Grid, Chart, Cards, etc.
│   │   ├── MainContentView.xaml.cs
│   │   └── MainContentViewModel.cs
│   └── Footer/
│       ├── FooterView.xaml           # Implement navigation links, copyright
│       ├── FooterView.xaml.cs
│       └── FooterViewModel.cs
└── Resources/
    └── DashboardStyles.xaml          # Shared styles for dashboard sections
```

**WinUI Adaptation:**
- **Views/** folder contains all UserControl-based section Views
- **Resources/** folder contains shared ResourceDictionary files
- Each section has its own ViewModel following MVVM pattern
- Parent Window composes sections using `<local:HeaderView />` syntax
- Syncfusion WinUI controls loaded via App.xaml MergedDictionaries

---

---

## Syncfusion Control and Theme Package Installation

**CRITICAL:** After code generation completes, you MUST install all Syncfusion control and theme packages that were used in the generated code. Use MSBuild as the primary method for package restoration and project updates.

### PRIORITY 1: MSBuild (Primary Method)
Use the MSBuild path discovered in the [MANDATORY Build Error Resolution Protocol](../../../agents/syncfusion-winui-ui-builder.agent.md#priority-1-msbuild-compiler-vs2026--vs2022---primary-build-system).

1. **Add Package Reference**: Manually add the `<PackageReference />` to your `.csproj`:
   ```xml
   <PackageReference Include="Syncfusion.UI.Xaml.Grid" Version="[version]" />
   ```
2. **Restore using MSBuild**:
   ```bash
   & $msbuild YourProject.csproj /t:Restore /p:Configuration=Debug /p:Platform=x64 /v:minimal
   ```

### PRIORITY 2: dotnet CLI (Fallback)
If MSBuild is unavailable, use the following `dotnet add package` commands:

```bash
dotnet add package Syncfusion.UI.Xaml.Grid --version [version]
dotnet add package Syncfusion.UI.Xaml.Charts --version [version]
dotnet add package Syncfusion.UI.Xaml.Buttons --version [version]
dotnet add package Syncfusion.UI.Xaml.Themes.Fluent --version [version]
```

**Without installing these packages, the generated code will fail to render.**

## Control Integration & File Mapping

**Generated files MUST be wired to display in the app:**

1. **Code-Behind Registration** (`controls/LoginForm/LoginForm.xaml.cs`):
   ```csharp
   public partial class LoginForm : UserControl
   {
     public LoginForm()
     {
       this.InitializeComponent();
     }
   }
   ```

2. **Import in MainWindow.xaml**:
   ```xaml
   <Window
     xmlns:local="using:YourApp.Controls">
     <Grid>
       <local:LoginForm />
     </Grid>
   </Window>
   ```

3. **Ensure XAML namespaces are loaded**:
   - If no framework/greenfield styles: Automatically imported in control
   - If Fluent Design: Styles applied directly
   - If Syncfusion theme: Already imported at app entry point (Stage 4)

**Without this mapping, control won't render in sample.**

**User Interaction:** 
Optional review of generated code. No blocking confirmation.

**Status:** AI generates without user decision. User can review/adjust if needed.
