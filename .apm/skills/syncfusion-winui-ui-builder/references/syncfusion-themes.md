# Syncfusion WinUI Theming Resources

**⚠️ MANDATORY:** After selecting your WinUI color mode, you MUST consult **Skill: syncfusion-winui-theming** for detailed implementation guidance before proceeding to Stage 5.

## Theming in WinUI

WinUI uses the built-in Windows App SDK light and dark theme system. There are no separate Syncfusion theme packages in WinUI. Color mode is controlled via the `RequestedTheme` property on the root element, and Syncfusion controls adapt to the active theme automatically.

### Implementation Checklist:
1. **License Registration:** Call `SyncfusionLicenseProvider.RegisterLicense()` in `App.xaml.cs` → `OnLaunched()` before any control is used.
2. **Color Mode:** Set `RequestedTheme` on the root `Window` or `Application` element to `Light`, `Dark`, or `Default` (follows OS setting).
3. **Custom Resources:** Define app-level color, spacing, and typography tokens in `Themes/Colors.xaml`, `Themes/Spacing.xaml`, and `Themes/Typography.xaml`; merge them into `<Application.Resources>`.

## Color Mode Reference

| Color Mode | How to Apply | Description |
|---|---|---|
| **Light** | `RequestedTheme="Light"` on root element | Windows light appearance |
| **Dark** | `RequestedTheme="Dark"` on root element | Windows dark appearance |
| **Default** | `RequestedTheme="Default"` or omit property | Follows OS light/dark setting |

## By Use Case:

### Applying a Color Mode
- Set `RequestedTheme="Light"` or `RequestedTheme="Dark"` on the root `Window` element in XAML, or set `Application.Current.RequestedTheme` at runtime in code-behind.
- Refer to **Skill: syncfusion-winui-theming** → **color-mode-setup.md**.

### Implementing Dark Mode
- Set `RequestedTheme="Dark"` on the root `Window` or the `Application` element.
- To toggle at runtime, set `Application.Current.RequestedTheme = ApplicationTheme.Dark`.
- Refer to **Skill: syncfusion-winui-theming** → **color-mode-setup.md**.

### Customizing Colors
- Define semantic `SolidColorBrush` resources in `Themes/Colors.xaml`.
- Use `{ThemeResource}` for system-provided brush keys and `{StaticResource}` for custom app resources.
- Refer to **Skill: syncfusion-winui-theming** → **theme-customization.md**.

### Using Icons
- Refer to **Icon Library** for:
  - Setting up Syncfusion WinUI icon fonts (Syncfusion MDL2 Assets)
  - TextBlock sizing with FontSize property (small, medium, large)
  - Icon customization via Foreground brush and FontSize binding

### Advanced Theming
- Refer to **Advanced Features** for:
  - Compact mode / Normal mode sizing
  - Font customization across WinUI controls via ResourceDictionary tokens
  - Per-monitor DPI-aware theme scaling