# Syncfusion WinUI Built-In Themes

**⚠️ MANDATORY:** Syncfusion WinUI controls include **built-in Light and Dark themes**. You MUST understand theme activation before proceeding to Stage 5.

---

## Quick Reference: Syncfusion WinUI Themes

| Theme | Activation Method | When to Use | DPI Support | High Contrast |
|-------|------------------|-------------|------------|---------|
| **Light** (Built-in) | `RequestedTheme = ApplicationTheme.Light` | Daytime use, explicit light preference | Auto ✅ | Auto ✅ |
| **Dark** (Built-in) | `RequestedTheme = ApplicationTheme.Dark` | Low-light environments, explicit dark preference | Auto ✅ | Auto ✅ |
| **Auto (Default)** | `RequestedTheme = ApplicationTheme.Default` | **RECOMMENDED** - Respects Windows user preference | Auto ✅ | Auto ✅ |

**Key Point:** No CSS imports, no configuration files, no theme selection packages. Themes are **built-in and automatic**.

---

## Theme Application Methods

### 1. Application-Level Theme (Recommended)

Set theme once in `App.xaml.cs` and all Syncfusion controls inherit it:

```csharp
// App.xaml.cs
using Microsoft.UI.Xaml;
using Syncfusion.Licensing;

public partial class App : Application
{
    public App()
    {
        // Register Syncfusion license (if commercial)
        SyncfusionLicenseProvider.RegisterLicense("YOUR_LICENSE_KEY");
        
        // Set app-level theme (RECOMMENDED: use Default to respect user preference)
        this.RequestedTheme = ApplicationTheme.Default;  // Light or Dark based on Windows Settings
        // OR
        // this.RequestedTheme = ApplicationTheme.Light;  // Force Light
        // this.RequestedTheme = ApplicationTheme.Dark;   // Force Dark
        
        this.InitializeComponent();
    }
}
```

**Result:** All Syncfusion controls automatically use Light or Dark theme.

### 2. Per-Control Theme Override

Override application theme for specific controls:

```xaml
<!-- MainWindow.xaml -->
<Window x:Class="MyApp.MainWindow">
    <!-- Light theme for this specific window (overrides app theme) -->
    <Grid RequestedTheme="Light">
        <syncfusion:SfDataGridControl x:Name="dataGrid" />
        <!-- dataGrid inherits Light theme from parent Grid -->
    </Grid>
</Window>
```

Or in code-behind:

```csharp
// MainWindow.xaml.cs
public MainWindow()
{
    this.InitializeComponent();
    
    // Set specific control to Dark theme
    this.dataGrid.RequestedTheme = ElementTheme.Dark;
}
```

### 3. Dynamic Theme Switching at Runtime

Switch between Light and Dark themes based on user input:

```csharp
// In your settings/preferences window
private void OnDarkModeToggle(object sender, RoutedEventArgs e)
{
    // Switch entire app to Dark theme
    ((App)Application.Current).RequestedTheme = ApplicationTheme.Dark;
    
    // OR switch to Light
    ((App)Application.Current).RequestedTheme = ApplicationTheme.Light;
}
```

---

## Light Theme

**Default color palette optimized for daytime use:**

- **Backgrounds:** Light white/light gray (#FFFFFF, #F3F3F3)
- **Text:** Dark charcoal (#000000, #333333)
- **Borders:** Light gray (#E8E8E8)
- **Accent:** System accent color (customizable)
- **Shadows:** Subtle drop shadows for depth

**Automatically applied when:**
```csharp
this.RequestedTheme = ApplicationTheme.Light;
```

**Visual characteristics:**
- ✅ Bright, clean appearance
- ✅ High contrast for readability
- ✅ Professional, minimal aesthetic
- ✅ Reduces eye strain in bright environments

---

## Dark Theme

**Default color palette optimized for low-light environments:**

- **Backgrounds:** Dark charcoal/dark gray (#1E1E1E, #2D2D2D)
- **Text:** Light white/light gray (#FFFFFF, #E8E8E8)
- **Borders:** Dark gray (#404040)
- **Accent:** System accent color (customizable, slightly desaturated)
- **Surfaces:** Elevated light surfaces for depth (no shadows)

**Automatically applied when:**
```csharp
this.RequestedTheme = ApplicationTheme.Dark;
```

**Visual characteristics:**
- ✅ Reduces eye strain in low-light environments
- ✅ Modern, sophisticated appearance
- ✅ Battery-saving on OLED displays
- ✅ Respects user accessibility preferences

---

## High Contrast Mode Support

**Syncfusion WinUI controls automatically support Windows High Contrast mode:**

- Users enable: Settings → Ease of Access → High Contrast
- Syncfusion controls automatically switch to high-contrast colors
- **No custom code required**
- SystemColors automatically provide maximum contrast

```xaml
<!-- No special markup needed -->
<syncfusion:SfDataGridControl x:Name="dataGrid" />
<!-- Automatically respects High Contrast mode -->
```

**How it works:**
1. User enables High Contrast in Windows
2. Syncfusion controls detect the system setting
3. Colors automatically adjust for 7:1+ contrast ratio
4. All text remains readable
5. **Zero developer work needed**

---

## DPI Scaling (Automatic)

**Syncfusion WinUI controls automatically scale across all DPI settings:**

| DPI Setting | Scaling | Example (14px font) |
|---|---|---|
| 96 DPI | 100% | 14px → 14px |
| 120 DPI | 125% | 14px → 17.5px |
| 144 DPI | 150% | 14px → 21px |
| 192 DPI | 200% | 14px → 28px |

**No manual work required:**
- Define all sizes in DIPs (Device Independent Pixels)
- XAML automatically scales on high-DPI displays
- Spacing, typography, and controls all scale proportionally

```xaml
<!-- Define in DIPs - scaling is automatic -->
<TextBlock FontSize="14" />  <!-- Auto-scales: 14→17.5→21→28 based on DPI -->
<Button Padding="8" />       <!-- Auto-scales: 8→10→12→16 based on DPI -->
```

**Testing:**
- Test on 100% DPI (standard desktop)
- Test on 125% DPI (laptop/2-in-1 tablets)
- Test on 150% DPI (high-resolution laptops)
- Test on 200% DPI (Surface devices)

---

## Theme Resource Customization

Syncfusion provides theme resource files for advanced customization.

### Accessing Theme Resource Keys

**Official Repository:** [Syncfusion WinUI Theme Resource Files](https://github.com/syncfusion/winui-controls-theme-resource-files)

**Example: Customizing DataGrid Header Color**

1. **Find the resource key** in the theme resource file (GitHub repository)
2. **Override in App.xaml:**

```xaml
<!-- App.xaml -->
<Application.Resources>
    <!-- Override Syncfusion DataGrid header background -->
    <SolidColorBrush x:Key="SyncfusionDataGridHeaderBackground" Color="#FF0078D4" />
    
    <!-- Override button hover background -->
    <SolidColorBrush x:Key="SyncfusionButtonHoverBackground" Color="#FFE0E0E0" />
    
    <!-- Override accent color globally -->
    <SolidColorBrush x:Key="SystemAccentColor" Color="#FF0078D4" />
</Application.Resources>
```

3. **All Syncfusion controls automatically use your custom colors**

### Default vs Customized Example

**Default theme resource values:**
```
SyncfusionRibbonTabMenuButtonBackground = SystemChromeLowColor
SyncfusionRibbonTabMenuButtonForeground = SystemBaseHighColor
SyncfusionRibbonTabBorderBrushSelected = SystemAccentColor
```

**Customized values:**
```xml
<SolidColorBrush x:Key="SyncfusionRibbonTabMenuButtonBackground" Color="Green" />
<SolidColorBrush x:Key="SyncfusionRibbonTabMenuButtonForeground" Color="White" />
<SolidColorBrush x:Key="SyncfusionRibbonTabBorderBrushSelected" Color="Green" />
```

---

## By Feature: Complete Reference

### Applying a Theme (Built-In Support)

**What Syncfusion provides:**
- ✅ Built-in Light theme (no installation)
- ✅ Built-in Dark theme (no installation)
- ✅ Automatic theme inheritance from Application.RequestedTheme
- ✅ Automatic High Contrast mode support
- ✅ Automatic DPI scaling (96-200%)

**What you need to do:**
1. Set `Application.RequestedTheme` in App.xaml.cs
2. All Syncfusion controls automatically use the theme
3. That's it!

### Implementing Dark Mode

**Built-in Dark Theme:**

```csharp
// Option 1: Respect Windows user preference (RECOMMENDED)
this.RequestedTheme = ApplicationTheme.Default;

// Option 2: Force Dark mode
this.RequestedTheme = ApplicationTheme.Dark;
```

**Features:**
- ✅ Entire app switches to dark colors
- ✅ User can change in Windows Settings
- ✅ All Syncfusion controls respond automatically
- ✅ No custom CSS or resource files needed

### Customizing Colors & Tokens

**Define brand colors in App.xaml:**

```xaml
<Application.Resources>
    <!-- Brand accent color -->
    <SolidColorBrush x:Key="SystemAccentColor">#FF0078D4</SolidColorBrush>
    
    <!-- Semantic colors -->
    <SolidColorBrush x:Key="SuccessColor">#FF107C10</SolidColorBrush>
    <SolidColorBrush x:Key="WarningColor">#FFFFB900</SolidColorBrush>
    <SolidColorBrush x:Key="ErrorColor">#FFDA3B01</SolidColorBrush>
    
    <!-- Spacing tokens -->
    <Thickness x:Key="StandardSpacing">8</Thickness>
    <Thickness x:Key="LargeSpacing">24</Thickness>
    
    <!-- Typography tokens -->
    <x:Double x:Key="TitleFontSize">32</x:Double>
    <x:Double x:Key="BodyFontSize">14</x:Double>
</Application.Resources>
```

**Use in controls:**

```xaml
<TextBlock Text="Title" FontSize="{StaticResource TitleFontSize}" />
<Button Background="{StaticResource SystemAccentColor}" Padding="{StaticResource StandardSpacing}" />
```

### Using Syncfusion Icons

Syncfusion WinUI controls include built-in icon support (Segoe MDL2 Assets):

```xaml
<!-- Icon in button -->
<Button>
    <StackPanel Orientation="Horizontal">
        <TextBlock Text="&#xE72C;" FontFamily="Segoe MDL2 Assets" FontSize="16" />
        <TextBlock Text="Download" Margin="8,0,0,0" />
    </StackPanel>
</Button>
```

**Icon sizing:**
- Small: 12-16px
- Medium: 20-24px
- Large: 32-48px

### Advanced Theming Features

**Compact Mode (for dense layouts):**

```csharp
// RequestedTheme can control layout density
// Light theme: standard spacing
// Dark theme: can customize for compact spacing if needed
```

**Font Customization:**

```xaml
<Application.Resources>
    <!-- Custom font family -->
    <FontFamily x:Key="CustomFont">Segoe UI</FontFamily>
    
    <!-- Override in controls -->
    <Style TargetType="TextBlock">
        <Setter Property="FontFamily" Value="{StaticResource CustomFont}" />
    </Style>
</Application.Resources>
```

---

## Installation & Setup

### 1. Add Syncfusion NuGet Packages

```bash
# Install core Syncfusion package
dotnet add package Syncfusion.Core.WinUI

# Install specific control packages as needed
dotnet add package Syncfusion.Grid.WinUI
dotnet add package Syncfusion.Ribbon.WinUI
```

### 2. Register Syncfusion License (if commercial)

```csharp
// App.xaml.cs
using Syncfusion.Licensing;

public App()
{
    SyncfusionLicenseProvider.RegisterLicense("YOUR_LICENSE_KEY");
    this.RequestedTheme = ApplicationTheme.Default;
    this.InitializeComponent();
}
```

### 3. Add Control Namespace to XAML

```xaml
<!-- MainWindow.xaml -->
<Window
    xmlns:syncfusion="using:Syncfusion.UI.Xaml.DataGrid"
    xmlns:editors="using:Syncfusion.UI.Xaml.Editors">
    
    <!-- Controls automatically use app theme -->
    <syncfusion:SfDataGrid x:Name="dataGrid" />
    <input:SfComboBox x:Name="ComboBox" />
</Window>
```

**That's all!** Themes are built-in and automatic.

---

## Troubleshooting Theme Issues

| Issue | Solution |
|-------|----------|
| Theme not applying | Verify `RequestedTheme` is set in App.xaml.cs **before** `InitializeComponent()` |
| Controls still show old theme | Restart application (themes cache during runtime) |
| High Contrast not working | Verify Windows High Contrast is enabled; Syncfusion should auto-detect |
| Text not readable | Check color contrast (WCAG AA minimum 4.5:1); may need custom theme override |
| DPI scaling looks wrong | Test on actual high-DPI device (not just Windows scaling emulation) |

---

## Summary: WinUI Theme Activation

| Task | Method | Effort |
|------|--------|--------|
| Use Light theme | `RequestedTheme = ApplicationTheme.Light` | 1 line |
| Use Dark theme | `RequestedTheme = ApplicationTheme.Dark` | 1 line |
| Respect user preference (RECOMMENDED) | `RequestedTheme = ApplicationTheme.Default` | 1 line |
| Support High Contrast | Nothing - automatic | 0 lines |
| Support DPI scaling | Nothing - automatic | 0 lines |
| Customize brand colors | Override resource keys in App.xaml | ~5-10 lines |
| Test dark mode | Change Windows Settings → toggle theme | Manual testing |
| Test High Contrast | Enable in Windows → Ease of Access | Manual testing |

**Syncfusion WinUI themes are production-ready and require minimal setup.**
