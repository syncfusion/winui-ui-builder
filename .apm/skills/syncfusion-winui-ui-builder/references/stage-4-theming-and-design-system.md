# Stage 4: Theming & Design System Selection

**Purpose:** Lock WinUI theming strategy and design system decisions before control generation in Stage 5.

## Overview

This stage is about **design decision clarity**, not code generation. You'll:

- **Confirm your WinUI theming approach** (.NET/Fluent Design) and understand platform guidelines
- **Select Syncfusion WinUI built-in theme** that aligns with Windows design standards
- **Define color architecture** based on Windows principles (accessibility, contrast, brand integration, High Contrast support)
- **Establish spacing and typography scales** that respect Windows Fluent Design System and DPI scaling
- **Plan adaptive layouts** for different window sizes and DPI configurations
- **Document XAML resource structure** so Stage 5 can generate consistent styled controls

**Key Insight:** Your WinUI theming strategy directly impacts user experience across Windows devices. Syncfusion provides built-in Light and Dark themes that automatically respect Windows user preferences and High Contrast modes. Your job is to decide on theme activation strategy and any brand customizations.

This file provides theming guidance for Syncfusion WinUI controls. Related resources:
- **[winui-xaml-implementation.md](winui-xaml-implementation.md)** — XAML resource dictionary patterns with Fluent Design
- **[Syncfusion WinUI Themes GitHub](https://github.com/syncfusion/winui-controls-theme-resource-files)** — Theme resource files for customization

**Output:** Design system decisions documented and ready for implementation in Stage 5 as XAML and C# code.

---

## Table of Contents

1. [WinUI Theming Philosophy](#1-winui-theming-philosophy)
2. [Syncfusion WinUI Built-In Themes](#2-syncfusion-winui-built-in-themes)
3. [Color System Architecture](#3-color-system-architecture)
4. [Spacing & Typography Systems](#4-spacing--typography-systems)
5. [Window Sizing & Adaptive Layouts](#5-window-sizing--adaptive-layouts)
6. [Motion & Accessibility Standards](#6-motion--accessibility-standards)
7. [XAML Resource Dictionary Architecture](#7-xaml-resource-dictionary-architecture)
8. [Syncfusion Control Integration](#8-syncfusion-control-integration)
9. [Load Your WinUI Implementation Reference (MANDATORY)](#9-load-your-winui-implementation-reference-mandatory)
10. [Stage 4 Decision Checklist](#10-stage-4-decision-checklist)
11. [What Stage 5 Does With These Decisions](#what-stage-5-does-with-these-decisions)

---

## 1. WinUI Theming Philosophy

**Input:** .NET/WinUI project detected in Stage 2

**Decision Point:** Your theming approach defines everything downstream. Understand Windows design principles:

### Understanding WinUI & Fluent Design

**Fluent Design System 2 (Primary Standard)**
- Philosophy: Modern, clean design with clarity, light, and depth
- Design implication: You choose theme activation method; Syncfusion handles visual system
- Windows integration: Respects user's Light/Dark theme preference and High Contrast mode
- Trade-off: Platform-native look-and-feel; limited customization but maximum consistency

**Key Windows Platform Concepts:**
- **RequestedTheme:** Application-level theme setting (Light or Dark)

### Decision: Confirm Your WinUI Theming Approach

Review Stage 2's detection:
- Is WinUI 3 the confirmed framework?
- Is .NET 6+ the target runtime?
- Will the app support High Contrast mode? (Recommended for production)
- Should dark mode be supported? (Recommended for all modern apps)

If Stage 2 detected wrong: **Document why.** Design decisions need reasoning.

**→ MANDATORY:** After confirming, proceed to **Section 2** to understand Syncfusion's built-in themes.

**Output:** WinUI theming philosophy understood and confirmed.

---

## 2. Syncfusion WinUI Built-In Themes

**Core Principle:** Syncfusion WinUI controls include built-in Light and Dark themes that automatically integrate with Windows design.

### Why Syncfusion's Built-In Themes Matter

Syncfusion provides **production-ready themes** for all WinUI controls. These themes:
- ✅ Automatically respect `Application.RequestedTheme` (Light or Dark)
- ✅ Support High Contrast mode via `SystemColors` integration
- ✅ Work across all DPI scales (96-200%) without custom scaling
- ✅ Follow Windows Fluent Design System 2 color palettes
- ✅ Require no manual CSS or style definitions for basic usage
- ✅ Can be customized via XAML ResourceDictionary for brand colors

### Supported Themes

Syncfusion WinUI controls support **two built-in themes**:

#### Theme 1: Light Theme
- Default color palette optimized for daytime use
- Light backgrounds (white/light gray)
- Dark text and icons
- Subtle shadows and borders
- Automatically applied when `RequestedTheme = ApplicationTheme.Light`
- Inherited by all child controls unless overridden

#### Theme 2: Dark Theme
- Default color palette optimized for low-light environments
- Dark backgrounds (charcoal/dark gray)
- Light text and icons
- Elevated surfaces for depth (no shadows)
- Automatically applied when `RequestedTheme = ApplicationTheme.Dark`
- Inherited by all child controls unless overridden

### 2.1 Standard WinUI Resource Integration (MANDATORY)

**Requirement: Ensure Application-Wide Theme Brushes are Available**
To use modern Windows App SDK styles and theme brushes (e.g., secondary fill colors, layer brushes), the `XamlControlsResources` must be explicitly included in the application's global resources.

**Fix: Resource Loading Errors**
If you encounter "Cannot locate resource" errors for `Microsoft.UI.Xaml/Themes/Themes.xaml`, ensure the project references the **Windows App SDK** NuGet package correctly and that `App.xaml` uses the correct `XamlControlsResources` syntax which automatically imports platform themes.

**Implementation:** Ensure `App.xaml` includes the foundational `XamlControlsResources` within the merged dictionaries:
```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <XamlControlsResources xmlns="using:Microsoft.UI.Xaml.Controls" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

**Output: Foundational WinUI resources and platform themes correctly integrated.**

**User Preference Handling:**
```csharp
// In App.xaml.cs
public App()
{
    // Option A: Use system preference (RECOMMENDED)
    // Automatically switches when user changes Windows theme in Settings
    this.RequestedTheme = ApplicationTheme.Default;
    
    // Option B: Force Light theme
    this.RequestedTheme = ApplicationTheme.Light;
    
    // Option C: Force Dark theme
    this.RequestedTheme = ApplicationTheme.Dark;
    
    // Syncfusion controls automatically adopt the theme
    this.InitializeComponent();
}
```

### High Contrast Mode Support

Syncfusion controls automatically respect **High Contrast mode** via Windows `SystemColors`:

```xaml
<!-- High Contrast mode automatically switches colors -->
<!-- No custom code needed -->
<syncfusion:SfDataGridControl x:Name="dataGrid" />
```

**Decision Point:** Does your application support High Contrast mode?
- **Yes (RECOMMENDED):** Users with visual impairments can use your app
- **No:** Acceptable for internal/specialized applications only

**Syncfusion themes are built-in and automatic.** The only choice is:
1. **Theme Activation Method:** Auto (system preference), Light, or Dark
2. **Brand Customization:** Optional color overrides via XAML resources

### Decision: Theme Activation Strategy

**Recommended:** Use `ApplicationTheme.Default` to respect system preference

```csharp
// RECOMMENDED: Auto-switch with Windows theme
public App()
{
    this.RequestedTheme = ApplicationTheme.Default; // Respects Windows Settings
    this.InitializeComponent();
}
```

**Alternative:** Force a specific theme

```csharp
// ALTERNATIVE: Always use Light theme
public App()
{
    this.RequestedTheme = ApplicationTheme.Light;
    this.InitializeComponent();
}
```

**Output:** Syncfusion theme activation strategy decided.

---

## 3. Color System Architecture

### 3.1 Windows Color Space

**Use ARGB hex format (standard for Windows)**
- Format: `#AARRGGBB` (e.g., `#FF0078D4` = blue, fully opaque)
- All colors auto-adjust in High Contrast mode
- No need for OKLCH or perceptually-uniform color spaces (Windows handles this)

### 3.2 Brand Color & Semantic Palette

**Define:**
1. **Primary Accent Color** (brand color, replaces Windows system accent in your app)
2. **System Colors Integration** (Windows-provided colors for text, backgrounds, borders)
3. **Semantic Colors** (optional: success/warning/error/info overrides)

**Windows Color Strategy:**
- **Light Theme Default:** Windows light background with dark text
- **Dark Theme Default:** Windows dark background with light text

**Example brand customization:**

```xaml
<!-- App.xaml -->
<Application.Resources>
    <!-- Your brand accent color (replaces Windows system accent) -->
    <SolidColorBrush x:Key="SystemAccentColor">#FF0078D4</SolidColorBrush>
    
    <!-- Semantic colors (optional) -->
    <SolidColorBrush x:Key="SuccessColor">#107C10</SolidColorBrush>
    <SolidColorBrush x:Key="WarningColor">#FFB900</SolidColorBrush>
    <SolidColorBrush x:Key="ErrorColor">#DA3B01</SolidColorBrush>
</Application.Resources>
```

**Anti-Pattern:** Don't override all Windows system colors. Syncfusion handles them. Only override if you have specific brand requirements.

### 3.3 Dark Mode in WinUI

**The Reality (not misconception like web):** WinUI dark mode is built-in and automatic.

When user switches Windows Settings → Personalization → Colors → Dark:
- ✅ All Syncfusion controls automatically switch
- ✅ All system colors automatically adjust
- ✅ No manual code needed
- ✅ High Contrast mode automatically applies if enabled

**Decision Point:** Will you support dark mode?
- **Yes (RECOMMENDED):** All modern Windows apps support this
- **No:** Set `RequestedTheme = ApplicationTheme.Light` to disable dark mode

**Output:** Color system architecture decided for Windows platform.

---

## 4. Spacing & Typography Systems

### 4.1 Spacing Grid: Windows Standard

**XAML Implementation:**

```xaml
<!-- App.xaml ResourceDictionary -->
<Thickness x:Key="CompactSpacing">4</Thickness>
<Thickness x:Key="StandardSpacing">8</Thickness>
<Thickness x:Key="SpacySpacing">12</Thickness>
<Thickness x:Key="LargeSpacing">24</Thickness>

<!-- Usage in controls -->
<Button Padding="{StaticResource StandardSpacing}" />
<StackPanel Spacing="{StaticResource StandardSpacing}" />
```

**Anti-Pattern:** Arbitrary spacing (5px, 7px, 13px). Windows apps use multiples of 4px.

### 4.2 Typography: Segoe UI & DPI Scaling

**Platform Font:** Always use `FontFamily="Segoe UI"` (Windows standard)

**Fluent Design Typography:**
- **Display:** 46px (large headlines, rarely used)
- **Title:** 32px (major headings)
- **Subtitle:** 24px (section headings)
- **Body:** 14px or 16px (default body text)
- **Caption:** 12px (helper text, labels)

**XAML Implementation:**

```xaml
<!-- App.xaml ResourceDictionary -->
<x:Double x:Key="DisplayFontSize">46</x:Double>
<x:Double x:Key="TitleFontSize">32</x:Double>
<x:Double x:Key="SubtitleFontSize">24</x:Double>
<x:Double x:Key="BodyFontSize">14</x:Double>
<x:Double x:Key="CaptionFontSize">12</x:Double>

<!-- Usage in controls -->
<TextBlock Text="Heading" FontSize="{StaticResource TitleFontSize}" 
           FontFamily="Segoe UI" FontWeight="SemiBold" />
```

**Decision Point:** Use Windows typography defaults or customize?
- **Default:** Proven, matches Windows apps, no surprises
- **Custom:** More flexibility, but must test on multiple DPI scales (125%, 150%, 200%)

**Output:** Typography and spacing systems decided for Windows DPI scaling.

---

## 5. Window Sizing & Adaptive Layouts

### 5.1 Desktop-First Thinking for Windows

**Principle:** WinUI apps respond to window size, not viewport. Design for minimum viable window width, then scale up gracefully.

### 5.2 Breakpoint Strategy for Windows

**Window Size Breakpoints:**
- **Compact (< 600px):** Minimal layout, single column, condensed controls
- **Standard (600-1200px):** Primary layout, side panels start appearing
- **Wide (1200-1600px):** Multi-column, rich layouts
- **Extended (> 1600px):** Multi-panel with detail views

**XAML Visual State Triggers:**

```xaml
<VisualStateManager.VisualStateGroups>
  <VisualStateGroup x:Name="WindowSizeStates">
    <VisualState x:Name="CompactState">
      <VisualState.StateTriggers>
        <AdaptiveTrigger MinWindowWidth="0" />
      </VisualState.StateTriggers>
      <!-- Compact layout: single column -->
    </VisualState>
    <VisualState x:Name="StandardState">
      <VisualState.StateTriggers>
        <AdaptiveTrigger MinWindowWidth="600" />
      </VisualState.StateTriggers>
      <!-- Standard layout: panels appear -->
    </VisualState>
    <VisualState x:Name="WideState">
      <VisualState.StateTriggers>
        <AdaptiveTrigger MinWindowWidth="1200" />
      </VisualState.StateTriggers>
      <!-- Wide layout: multi-column -->
    </VisualState>
  </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

**Decision Point:** What is your minimum supported window width?
- Default: 400px (balanced, accessible)
- Touch-first: 600px+ (larger touch targets)
- Dense UI: 300px+ (for expert users)

## 6. Motion & Accessibility Standards

### 6.1 Motion in WinUI

**Fluent Design Motion Principles:**
- Focus: Smooth, purposeful transitions
- Tempo: Standard transitions (200-300ms)
- Easing: Cubic ease-in-out for natural feel

**Usage:**
- Hover animations (state changes)
- Reveal effects (optional, Fluent-specific)
- Transitions between views
- Loading indicators

**Implementation:**

```xaml
<!-- Fade transition (200ms) -->
<Storyboard x:Name="FadeIn">
  <DoubleAnimation Duration="0:0:0.2" From="0" To="1" 
    Storyboard.TargetProperty="Opacity" />
</Storyboard>

<!-- Scale transition (300ms) -->
<Storyboard x:Name="ScaleUp">
  <DoubleAnimation Duration="0:0:0.3" From="0.9" To="1.0"
    Storyboard.TargetProperty="ScaleX" />
</Storyboard>
```

### 6.2 Accessibility Standards

**WCAG 2.1 AA Requirements (Mandatory):**

- **Keyboard Navigation:** All controls focusable, logical tab order
- **Color Contrast:** Text ≥ 4.5:1 (normal), ≥ 3:1 (large)
- **High Contrast Mode:** Auto-supported via SystemColors
- **Touch Targets:** ≥ 44x44px minimum
- **Screen Readers:** AutomationProperties for narration

**Keyboard Navigation:**
```csharp
// Ensure Tab order is logical
public MainWindow()
{
    this.InitializeComponent();
    
    // Set tab order if needed
    TabIndex.SetTabIndex(Button1, 0);
    TabIndex.SetTabIndex(TextBox1, 1);
    TabIndex.SetTabIndex(Button2, 2);
}
```

**AutomationProperties for Screen Readers:**
```xaml
<Button Content="Submit" 
        AutomationProperties.Name="Submit Form"
        AutomationProperties.HelpText="Click to submit the form" />
```

**Decision Point:** Are you aiming for AA (minimum legal) or AAA (higher standard)?
- **AA:** Sufficient for most applications
- **AAA:** Required for government/accessibility-focused apps

**Output:** Motion and accessibility standards understood.

---

## 7. XAML Resource Dictionary Architecture

### 7.1 Token Naming: Semantic, Not Descriptive

**Good (Semantic):**
```xaml
<SolidColorBrush x:Key="PrimaryBrush">#FF0078D4</SolidColorBrush>
<SolidColorBrush x:Key="TextOnPrimaryBrush">#FFFFFFFF</SolidColorBrush>
<Thickness x:Key="StandardPadding">8</Thickness>
```

**Bad (Descriptive):**
```xaml
<!-- Don't do this -->
<SolidColorBrush x:Key="Blue600">#FF0078D4</SolidColorBrush>
<SolidColorBrush x:Key="White">#FFFFFFFF</SolidColorBrush>
<Thickness x:Key="Padding8">8</Thickness>
```

### 7.2 Resource Dictionary Structure

**Location: App.xaml (Application-level)**

```xaml
<!-- App.xaml -->
<Application.Resources>
  <!-- Syncfusion built-in themes are automatically applied here -->
  
  <!-- Your brand color overrides -->
  <SolidColorBrush x:Key="SystemAccentColor">#FF0078D4</SolidColorBrush>
  
  <!-- Semantic colors -->
  <SolidColorBrush x:Key="SuccessBrush">#FF107C10</SolidColorBrush>
  <SolidColorBrush x:Key="WarningBrush">#FFFFB900</SolidColorBrush>
  <SolidColorBrush x:Key="ErrorBrush">#FFDA3B01</SolidColorBrush>
  
  <!-- Spacing tokens -->
  <Thickness x:Key="CompactSpacing">4</Thickness>
  <Thickness x:Key="StandardSpacing">8</Thickness>
  <Thickness x:Key="LargeSpacing">24</Thickness>
  
  <!-- Typography tokens -->
  <x:Double x:Key="TitleFontSize">32</x:Double>
  <x:Double x:Key="BodyFontSize">14</x:Double>
  <FontWeight x:Key="SemiBoldWeight">SemiBold</FontWeight>
  
  <!-- Control-specific styles (optional) -->
  <Style x:Key="PrimaryButtonStyle" TargetType="Button">
    <Setter Property="Padding" Value="{StaticResource StandardSpacing}" />
    <Setter Property="Foreground" Value="{StaticResource TextOnPrimaryBrush}" />
  </Style>
</Application.Resources>
```

**Location: Generic.xaml (Control templates)**

```xaml
<!-- Themes/Generic.xaml -->
<ResourceDictionary>
  <!-- Reusable control styles based on App.xaml tokens -->
  <Style TargetType="Button" x:Key="AccentButtonStyle">
    <Setter Property="Background" Value="{StaticResource SystemAccentColor}" />
    <Setter Property="Padding" Value="{StaticResource StandardSpacing}" />
  </Style>
  
  <Style TargetType="TextBlock" x:Key="TitleStyle">
    <Setter Property="FontSize" Value="{StaticResource TitleFontSize}" />
    <Setter Property="FontFamily" Value="Segoe UI" />
    <Setter Property="FontWeight" Value="SemiBold" />
  </Style>
</ResourceDictionary>
```

### 7.3 Syncfusion Theme Resource Customization

**Customizing Syncfusion Control Colors:**

Syncfusion provides theme resources for each control. Override them in your App.xaml:

```xaml
<!-- App.xaml -->
<Application.Resources>
  <!-- Example: Customize DataGrid header background -->
  <SolidColorBrush x:Key="SyncfusionDataGridHeaderBackground">#FF0078D4</SolidColorBrush>
  
  <!-- Example: Customize Button hover state -->
  <SolidColorBrush x:Key="SyncfusionButtonHoverBackground">#FFE0E0E0</SolidColorBrush>
  
  <!-- See Syncfusion theme resource files for all available keys -->
  <!-- https://github.com/syncfusion/winui-controls-theme-resource-files -->
</Application.Resources>
```

**Decision Point:** Centralized tokens vs distributed resources?
- **Centralized (RECOMMENDED):** All tokens in App.xaml
- **Distributed:** Per-control styles in Generic.xaml (modular but harder to track)

**Output:** XAML resource dictionary architecture understood.

---

## 8. Syncfusion Control Integration

### 8.1 Theme Initialization (Built-In)

**Important:** Syncfusion WinUI controls require **no manual theme initialization**.

Themes are **automatically applied** based on `Application.RequestedTheme`:

```csharp
// App.xaml.cs
public App()
{
    // That's it. No theme imports, no configuration needed.
    // Syncfusion controls will automatically use Light/Dark based on RequestedTheme
    this.RequestedTheme = ApplicationTheme.Default;
    this.InitializeComponent();
}
```

### 8.2 Syncfusion License Registration

```csharp
// App.xaml.cs - Register license before any Syncfusion control renders
using Syncfusion.Licensing;

public App()
{
    // Register license (required for commercial use)
    SyncfusionLicenseProvider.RegisterLicense("YOUR_LICENSE_KEY");
    
    this.RequestedTheme = ApplicationTheme.Default;
    this.InitializeComponent();
}
```

### 8.3 Custom XAML Coordination

**Pattern:** Don't override Syncfusion control colors arbitrarily. Instead:

1. Define your color system in XAML resources
2. Syncfusion controls inherit from built-in themes
3. Override only theme resource keys if needed

**Example:**
```xaml
<!-- GOOD: Override via Syncfusion theme resource key -->
<SolidColorBrush x:Key="SyncfusionDataGridHeaderBackground">
  <SolidColorBrush.Color>
    <x:ThemeResource ResourceKey="SystemAccentColor" />
  </SolidColorBrush.Color>
</SolidColorBrush>

<!-- BAD: Don't hardcode overrides in DataGrid -->
<syncfusion:SfDataGridControl Background="#FF0078D4" />
```

**Syncfusion Theme Resource Files:**
- Available on GitHub: https://github.com/syncfusion/winui-controls-theme-resource-files
- Each control has documented theme keys
- Override keys in App.xaml for brand customization

**Output:** Syncfusion WinUI control integration strategy decided.

---

## 9. Load Your WinUI Implementation Reference (MANDATORY)

**REQUIRED STEP:** Your WinUI design decisions from Sections 1-8 now determine implementation.

### WinUI Implementation References

All WinUI projects follow the same platform standards (no framework variations like web):

#### Primary Reference: WinUI XAML Implementation
→ **Load Reference:** [winui-xaml-implementation.md](winui-xaml-implementation.md)

**What this reference provides:**
- WinUI 3 project structure with Syncfusion setup
- App.xaml and resource dictionary patterns
- Code-behind (.xaml.cs) with event handlers
- MVVM ViewModel implementation
- Window sizing and adaptive layouts with VisualStateManager
- Stage 6 validation checklist for WinUI projects

**Key principle:** Centralize all theme resources in App.xaml. Use XAML bindings in controls. Keep code-behind focused on event handling and business logic.

---

**What this reference provides (if you want advanced effects):**
- Acrylic material (frosted glass effect)
- Reveal animations (light-based interactions)
- Connected animations (transitions between windows)
- Fluent motion principles (easing, timing)
- Performance optimization for visual effects

**Key principle:** Start with basic Fluent Design (solid colors, typography). Add effects only if your design calls for them.

---

#### Optional: Accessibility Compliance
→ **Reference:** [accessibility-implementation.md](accessibility-implementation.md)

**What this reference provides (if you need WCAG AAA):**
- High Contrast mode validation (automatic via SystemColors)
- Keyboard navigation implementation (Tab, arrow keys, Enter)
- Screen reader compatibility (AutomationProperties)
- Color contrast validation (WCAG AA/AAA)
- Touch target sizing (44x44px minimum)
- Reduced motion support

**Key principle:** WCAG AA compliance is mandatory. AAA is recommended for public-facing applications.

---

#### Reference: Syncfusion Theme Customization
→ **Resource:** [Syncfusion WinUI Theme Resources](https://github.com/syncfusion/winui-controls-theme-resource-files)

**What this provides:**
- Theme resource keys for each Syncfusion control
- Light and Dark theme color values
- Examples of custom theme overrides
- Control-specific customization patterns

**Key principle:** Use these files to understand which theme keys to override in your App.xaml for brand customization.

---

**You cannot proceed to Stage 5 without reviewing winui-xaml-implementation.md.**

**Optional but recommended:** Also review accessibility-implementation.md for WCAG compliance planning.

**Output:** WinUI XAML implementation reference locked based on your theming decisions.

---

## 10. Stage 4 Decision Checklist

**Load Your WinUI Implementation References (MANDATORY)**

**Upon completion, confirm the following decisions are locked:**

### WinUI Framework & Theme
- ✅ WinUI 3 confirmed as framework (.NET 6+, Windows App SDK)
- ✅ Theme activation strategy decided (Default/Light/Dark)
- ✅ High Contrast mode support confirmed (Yes/No)
- ✅ Syncfusion WinUI built-in themes will be used (automatic)

### Color System
- ✅ Brand accent color defined (ARGB hex format)
- ✅ Semantic colors identified (success/warning/error/info if needed)
- ✅ SystemColors integration confirmed (for High Contrast support)
- ✅ Dark mode support decided (Yes/No)
- ✅ No custom CSS frameworks (Windows native only)

### Spacing & Typography
- ✅ Spacing grid confirmed (4px base, multiples of 4px)
- ✅ Segoe UI typography confirmed (platform standard)
- ✅ Typography hierarchy locked (Display/Title/Subtitle/Body/Caption sizes)
- ✅ DPI scaling understood (automatic, no manual work)
- ✅ Line height and weight standards applied

### Responsive Design (Window Sizing)
- ✅ Minimum window width decided (300-600px range)
- ✅ Breakpoint strategy decided (Compact/Standard/Wide/Extended)
- ✅ Adaptive layout triggers planned (VisualStateTriggers at breakpoints)

### Accessibility
- ✅ WCAG compliance level decided (AA minimum, AAA recommended)
- ✅ High Contrast mode support confirmed (automatic via SystemColors)
- ✅ Keyboard navigation planned (logical tab order)
- ✅ Touch targets sized (44x44px minimum for touch)
- ✅ Color contrast verified or planned (WCAG AA or AAA)
- ✅ AutomationProperties planned for screen readers

### XAML Resource Architecture
- ✅ Resource dictionary location decided (App.xaml for centralized tokens)
- ✅ Token naming strategy confirmed (semantic, not descriptive)
- ✅ Syncfusion theme customization planned (brand color overrides)
- ✅ Control style inheritance understood (tokens used, not hardcoded)

### Syncfusion WinUI Integration
- ✅ Syncfusion built-in themes confirmed (Light/Dark, automatic)
- ✅ Theme activation method locked (RequestedTheme strategy)
- ✅ License registration planned (before first control renders)
- ✅ Theme resource customization understood (key overrides in App.xaml)
- ✅ No manual theme files needed (built-in themes sufficient)

### Implementation References (MANDATORY)
- ✅ WinUI XAML reference file loaded (winui-xaml-implementation.md)
- ✅ Syncfusion theme resource reference reviewed (GitHub link)
- ✅ Accessibility reference reviewed (accessibility-implementation.md)
- ✅ Fluent Design reference reviewed (optional but recommended)
- ✅ Ready to proceed to Stage 5 with WinUI design decisions locked

---

## What Stage 5 Does With These Decisions

Stage 5 (Code Generation) uses your Stage 4 decisions to generate:
- **App.xaml setup** with Syncfusion WinUI theme initialization
- **Resource dictionary** with color tokens and typography scales
- **Window definitions** (.xaml) with adaptive layouts using VisualStateTriggers
- **Code-behind files** (.xaml.cs) with event handlers and initialization
- **Syncfusion control integration** with proper theme inheritance
- **Accessibility markup** (AutomationProperties, semantic structure)
- **High Contrast support** via SystemColors integration

Stage 5 generates **production-ready WinUI code** aligned with your Stage 4 design decisions.

---

### For All WinUI Projects:
- ✅ Syncfusion built-in themes active (Light/Dark automatically applied)
- ✅ XAML resources centralized in App.xaml
- ✅ WCAG 2.1 AA accessibility (contrast, keyboard, screen readers)
- ✅ High Contrast mode support (automatic via SystemColors)
- ✅ Window responsive design verified (Compact/Standard/Wide breakpoints)
- ✅ C# and XAML code generation without errors
- ✅ Project builds and runs successfully

**Output:** Production-ready WinUI code aligned with Stage 4 design decisions and Windows platform standards
