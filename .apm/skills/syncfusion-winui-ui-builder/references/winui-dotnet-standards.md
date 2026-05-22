# WinUI Standards & Compliance Reference

**Version:** 2.0.0  
**Last Updated:** April 17, 2026  
**Purpose:** WCAG 2.2 AA, security, performance, and code quality standards enforced during Stage 6 validation

---

## Table of Contents

1. [Accessibility Standards (WCAG 2.2 AA)](#accessibility-standards-wcag-22-aa)
2. [Security Standards](#security-standards)
3. [Performance Standards](#performance-standards)
4. [Code Quality Standards](#code-quality-standards)
5. [Validation Checklist](#validation-checklist)
6. [Auto-Fix Rules](#auto-fix-rules)

---

## Accessibility Standards (WCAG 2.2 AA)

### WCAG Principles: POUR

| Principle | Description |
|-----------|-------------|
| **P**erceivable | Content can be perceived through different senses (text alternatives, contrast, clear structure) |
| **O**perable | Interface can be operated by all users (keyboard, no traps, sufficient target size) |
| **U**nderstandable | Content and interface are understandable (clear language, predictable, error guidance) |
| **R**obust | Content works with assistive technologies (semantic HTML, proper ARIA, screen reader compatible) |

---

### Perceivable: Content Must Be Perceivable

#### 1.1 Text Alternatives

**Images require descriptive alt text:**

```xaml
<!-- ❌ Missing alt -->
<Image Source="ms-appx:///Assets/chart.png"/>

<!-- ✅ Descriptive alt for informative image -->
<Image Source="ms-appx:///Assets/chart.png" 
       AutomationProperties.Name="Bar chart showing 40% increase in Q3 sales"/>

<!-- ✅ Decorative image (no automation) -->
<Image Source="ms-appx:///Assets/decorative-border.png" 
       AutomationProperties.AccessibilityView="Raw"/>

<!-- ✅ Complex image with detailed description -->
<StackPanel>
  <Image Source="ms-appx:///Assets/infographic.png" 
         AutomationProperties.Name="2024 market trends infographic"
         AutomationProperties.HelpText="Market analysis showing trends..."/>
  <TextBlock x:Name="InfoDescription">
    <!-- Detailed description here -->
  </TextBlock>
</StackPanel>
```

**Icon buttons need accessible names:**

```xaml
<!-- ❌ No accessible name -->
<Button>
  <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE700;"/>
</Button>

<!-- ✅ Using AutomationProperties.Name -->
<Button AutomationProperties.Name="Open menu">
  <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE700;"/>
</Button>

<!-- ✅ Using Content with ToolTip -->
<Button ToolTipService.ToolTip="Open menu">
  <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE700;"/>
</Button>
```

**Validation Rules:**
- [ ] All informative images have descriptive AutomationProperties.Name
- [ ] Decorative images have AutomationProperties.AccessibilityView="Raw"
- [ ] Icon-only buttons have AutomationProperties.Name or ToolTip
- [ ] Complex images have AutomationProperties.HelpText with detailed description

---

#### 1.2 Media Alternatives

```xaml
<!-- Video with captions (MediaPlayerElement) -->
<MediaPlayerElement x:Name="mediaPlayer" 
                    Source="ms-appx:///Assets/video.mp4"
                    AutomationProperties.Name="Product demo video"
                    AutomationProperties.HelpText="See video description below for audio description"/>

<!-- Audio with transcript -->
<MediaPlayerElement x:Name="audioPlayer" 
                    Source="ms-appx:///Assets/podcast.mp3"
                    AutomationProperties.Name="Weekly podcast"/>
<Expander Header="Transcript">
  <TextBlock TextWrapping="Wrap">
    Full transcript text...
  </TextBlock>
</Expander>
```

**Validation Rules:**
- [ ] Videos have captions (external or embedded)
- [ ] Videos have audio descriptions provided
- [ ] Auto-playing audio can be paused/stopped
- [ ] Media controls are keyboard accessible

---

#### 1.4 Color Contrast

**Minimum Ratios (WCAG 2.2 AA):**

| Text Type | Minimum Ratio |
|-----------|---------------|
| Normal text (< 18px / < 14px bold) | 4.5:1 |
| Large text (≥ 18px / ≥ 14px bold) | 3:1 |
| UI controls & graphics | 3:1 |
| Focus indicators | 3:1 |

```xaml
<!-- ✓ GOOD - High contrast -->
<Button Background="#FFFFFF" Foreground="#000000">
  <!-- Contrast ratio: 21:1 -->
</Button>

<!-- ✗ BAD - Low contrast (fails) -->
<TextBlock Text="Hint text" Foreground="#CCCCCC" Background="#FFFFFF">
  <!-- Contrast ratio: 1.9:1 FAILS -->
</TextBlock>

<!-- ✓ GOOD - Focus indicator with sufficient contrast -->
<Style TargetType="Button">
  <Setter Property="CornerRadius" Value="2"/>
  <!-- FocusVisualPrimaryBrush set in app resources -->
</Style>
```

**Don't rely on color alone to convey information:**

```xaml
<!-- ❌ Only color indicates error -->
<TextBox BorderBrush="Red"/>

<!-- ✅ Color + icon + text -->
<StackPanel>
  <TextBox x:Name="emailInput" 
           AutomationProperties.Name="Email"
           AutomationProperties.HelpText="Valid email required"/>
  <StackPanel Orientation="Horizontal" Spacing="8">
    <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE7BA;" Foreground="Red"/>
    <TextBlock Text="Please enter a valid email address" Foreground="Red"/>
  </StackPanel>
</StackPanel>
```

**Validation Rules:**
- [ ] All text has contrast ≥ 4.5:1 (normal) or 3:1 (large)
- [ ] Focus indicators have contrast ≥ 3:1
- [ ] Placeholder/hint text has minimum 4.5:1 contrast
- [ ] Icons conveying information have 3:1 contrast
- [ ] Information not conveyed by color alone

---

### Operable: Users Must Be Able to Operate the Interface

#### 2.1 Keyboard Accessibility

**All functionality must be accessible via keyboard—no mouse-only actions:**

```csharp
// ❌ BAD - Only handles pointer (mouse-only)
button.PointerPressed += (s, e) => HandleAction();

// ✅ GOOD - Handles pointer and keyboard
button.PointerPressed += (s, e) => HandleAction();
button.KeyDown += (s, e) => {
  if (e.Key == Windows.System.VirtualKey.Enter || e.Key == Windows.System.VirtualKey.Space) {
    e.Handled = true;
    HandleAction();
  }
};
```

**No keyboard traps—users must be able to Tab out of every control:**

```xaml
<!-- ✓ GOOD - Escape closes dialog, focus can exit -->
<ContentDialog KeyDown="Dialog_KeyDown">
  <!-- content -->
</ContentDialog>

<!-- ✗ BAD - Focus can't escape -->
<UserControl PreviewKeyDown="Control_PreviewKeyDown" />
<!-- where PreviewKeyDown blocks Tab -->
```

**Validation Rules:**
- [ ] All interactive controls in tab order (no negative TabIndex)
- [ ] Tab order is logical (left-to-right, top-to-bottom)
- [ ] Can Tab into and out of every control
- [ ] Escape key closes dialogs and flyouts
- [ ] Enter key submits forms and activates buttons
- [ ] Arrow keys work for lists, dropdowns, tabs

---

#### 2.4 Focus Management

**Users must see where keyboard focus is—focus indicators must be visible:**

```xaml
<!-- ❌ NEVER remove focus visuals (accessibility violation) -->
<Style TargetType="Button">
  <!-- Don't set UseSystemFocusVisuals="False" without replacement -->
</Style>

<!-- ✅ GOOD - Always provide visible focus -->
<Style TargetType="Button">
  <Setter Property="UseSystemFocusVisuals" Value="True"/>
</Style>

<!-- ✅ GOOD - Custom focus styles work too -->
<Style TargetType="Button">
  <Setter Property="FocusVisualPrimaryBrush" Value="#0066CC"/>
  <Setter Property="FocusVisualPrimaryThickness" Value="2"/>
</Style>

<!-- ✅ GOOD - Focus not obscured by sticky headers (NEW in 2.2) -->
<ScrollViewer>
  <StackPanel Padding="0,80,0,60">
    <!-- content -->
  </StackPanel>
</ScrollViewer>
```

**Manage focus when opening dialogs:**

```csharp
public class MyDialog : ContentDialog
{
  private TextBox _firstInput;

  public MyDialog()
  {
    InitializeComponent();
    _firstInput = new TextBox();
  }

  protected override void OnApplyTemplate()
  {
    base.OnApplyTemplate();
    // Focus first input when dialog opens
    _firstInput?.Focus(FocusState.Programmatic);
  }
}
```

**Validation Rules:**
- [ ] Focus indicators always visible (never hide FocusVisuals)
- [ ] Focus visual ≥ 2px thick, ≥ 3:1 contrast
- [ ] Focus order logical (top-to-bottom, left-to-right)
- [ ] Focus managed when dialog opens
- [ ] Focused control not obscured by sticky headers/footers

---

#### 2.5 Target Size (NEW in 2.2)

**Interactive targets must be at least 24 × 24 effective pixels:**

```xaml
<!-- ✓ GOOD - Minimum target size -->
<Button MinWidth="24" MinHeight="24" Padding="8"/>

<!-- ✓ GOOD - Comfortable target size (recommended 44×44 for touch) -->
<Button MinWidth="44" MinHeight="44" Padding="8"/>

<!-- ✓ GOOD - Button with padding for larger touch area -->
<StackPanel Padding="10">
  <Button Content="Click me" Padding="12,8,12,8"/>
</StackPanel>
```

**Exceptions:** Inline text links, controls managed by system (MediaTransportControls), targets where a 24px circle centered on the bounding box doesn't overlap another target.

**Validation Rules:**
- [ ] All buttons ≥ 24×24 effective pixels
- [ ] All links ≥ 24×24 effective pixels
- [ ] All form inputs ≥ 24×24 effective pixels (or 44×44 for touch apps)
- [ ] Adequate spacing to prevent accidental activation

---

#### 2.5 Dragging Movements Alternative (NEW in 2.2)

**Any action triggered by dragging must offer a single-pointer alternative:**

```xaml
<!-- ❌ Drag-only reorder (fails accessibility) -->
<ListBox AllowDrop="True" CanReorderItems="True">
  <ListBoxItem Content="Item 1"/>
  <ListBoxItem Content="Item 2"/>
</ListBox>

<!-- ✅ Drag + button alternatives -->
<StackPanel>
  <ListBox x:Name="sortableList">
    <ListBoxItem Content="Item 1"/>
    <ListBoxItem Content="Item 2"/>
  </ListBox>
  <StackPanel Orientation="Horizontal" Spacing="8">
    <Button Content="↑" AutomationProperties.Name="Move selected item up"/>
    <Button Content="↓" AutomationProperties.Name="Move selected item down"/>
  </StackPanel>
</StackPanel>
```

**Applies to:** Sliders, map panning, color pickers, image cropping, and all drag-based interactions.

---

### Understandable: Content Must Be Understandable

#### 3.1 Language & Structure

```xaml
<!-- ✅ Primary language set (in app resources or code) -->
<!-- In App.xaml.cs: Windows.Globalization.ApplicationLanguages.PrimaryLanguageOverride = "en-US"; -->

<!-- ✅ Language elements properly structured -->
<Grid>
  <StackPanel>
    <TextBlock Text="Page Title" Style="{StaticResource HeadingStyle}"/>
    <TextBlock Text="Section Heading" Style="{StaticResource SubheadingStyle}"/>
    <TextBlock Text="Subsection" Style="{StaticResource BodyStyle}"/>
  </StackPanel>
</Grid>

<!-- ✅ Semantic structure with TextBlock levels -->
<StackPanel>
  <TextBlock Text="Main Title" FontSize="28" FontWeight="Bold"/>
  <TextBlock Text="Subtitle" FontSize="20" FontWeight="SemiBold"/>
  <TextBlock Text="Body text" FontSize="14"/>
</StackPanel>
```

**Validation Rules:**
- [ ] App language set (via ApplicationLanguages.PrimaryLanguageOverride or system)
- [ ] Text hierarchy follows semantic levels (Title > Subtitle > Body)
- [ ] Proper TextBlock styling hierarchy (no skipping levels)
- [ ] Text elements use descriptive AutomationProperties.Name

---

#### 3.2 Predictable Behavior

**Users must be able to predict what happens when they interact:**

```xaml
<!-- ✓ GOOD - Buttons perform their labeled action -->
<Button Content="Navigate to page" Click="NavigateButton_Click"/>
<Button Content="Submit form" Click="SubmitButton_Click"/>

<!-- ❌ BAD - Unexpected behavior -->
<Button Content="Click me" Click="OpenModal_Click"/>  <!-- Confusing if label says "Click me" -->
<HyperlinkButton Content="Go" NavigateUri="ms-appx:///page"/>  <!-- OK: HyperlinkButton navigates -->

<!-- ✓ GOOD - Focus doesn't trigger changes -->
<TextBox GotFocus="TextBox_GotFocus" />  <!-- OK: only highlight -->
<ComboBox SelectionChanged="ComboBox_SelectionChanged" />   <!-- BAD: unexpected submit on change -->
```

**Consistent help (NEW in 2.2)—help mechanisms appear in same relative order:**

```xaml
<!-- Help consistently placed in app resources/settings -->
<NavigationViewItemSeparator/>
<NavigationViewItem Icon="Help" Content="Help"/>
<NavigationViewItem Icon="Contact" Content="Contact us"/>
<NavigationViewItem Icon="Settings" Content="Settings"/>
```

---

#### 3.3 Forms & Error Handling

**Every input needs an associated label:**

```xaml
<!-- ❌ No label association -->
<TextBox PlaceholderText="Email"/>

<!-- ✅ Explicit label with TextBlock -->
<TextBlock Text="Email address"/>
<TextBox x:Name="emailInput" 
         PlaceholderText="name@domain.com"
         AutomationProperties.Name="Email address"
         AutomationProperties.IsRequiredForForm="True"/>

<!-- ✅ With instruction text -->
<TextBlock Text="Password"/>
<PasswordBox x:Name="passwordInput"
             AutomationProperties.Name="Password"
             AutomationProperties.HelpText="At least 8 characters with one number"/>
<TextBlock Text="At least 8 characters with one number" FontSize="12" Foreground="Gray"/>
```

**Error messages must be clear and linked to fields:**

```xaml
<!-- ❌ Error unclear -->
<TextBlock Text="Error" Foreground="Red"/>
<TextBox x:Name="emailInput"/>

<!-- ✅ Clear error with AutomationProperties -->
<TextBox x:Name="emailInput" 
         AutomationProperties.IsInvalidForForm="True"
         AutomationProperties.HelpText="Please enter a valid email address (example: name@domain.com)"/>
<StackPanel Orientation="Horizontal" Spacing="8">
  <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE7BA;" Foreground="Red"/>
  <TextBlock Text="Please enter a valid email address (example: name@domain.com)" 
             Foreground="Red" AutomationProperties.LiveSetting="Assertive"/>
</StackPanel>
```

**Don't force users to re-enter information (NEW in 2.2):**

```xaml
<!-- ✅ Auto-fill shipping from billing -->
<CheckBox x:Name="sameAsShipping" 
          Content="Shipping address same as billing"
          Checked="SameAsShipping_Checked"/>
<!-- Auto-populate when checked -->
```

**Login must not rely solely on cognitive tests (NEW in 2.2):**

```xaml
<!-- ❌ Cognitive test only (puzzle, remember pattern) -->
<Button Content="Solve puzzle to login"/>

<!-- ✅ Cognitive test + alternative -->
<StackPanel Spacing="12">
  <Button Content="Email me a login link"/>
  <Button Content="Sign in with Windows Hello"/>
  <Button Content="Use passkey"/>
</StackPanel>
```

**Validation Rules:**
- [ ] Every input has associated TextBlock label or AutomationProperties.Name
- [ ] Required fields marked with * or AutomationProperties.IsRequiredForForm="True"
- [ ] Error messages clear and specific
- [ ] Errors linked via AutomationProperties.HelpText or AutomationProperties.IsInvalidForForm
- [ ] Error messages include how to fix
- [ ] Validation happens at appropriate times (LostFocus, form submission)
- [ ] Information not re-requested (auto-fill where possible)
- [ ] Login not purely cognitive (offer alternatives)

---

### Robust: Content Must Work with Assistive Technologies

#### 4.1 Semantic XAML

**Prefer native controls—they have accessibility built in:**

```xaml
<!-- ❌ Non-semantic with custom behavior (harder to maintain) -->
<TextBlock Text="Submit" Tapped="Submit_Tapped" TextDecorations="Underline"/>

<!-- ✅ Native button (automatic: keyboard, focus, role) -->
<Button Content="Submit" Click="Submit_Click"/>

<!-- ❌ Custom checkbox (hard to manage) -->
<StackPanel Tapped="Toggle_Tapped">
  <TextBlock Text="Option"/>
</StackPanel>

<!-- ✅ Native checkbox (simple, accessible) -->
<CheckBox Content="Option"/>

<!-- ✗ Non-semantic form -->
<StackPanel>
  <TextBlock Text="Email"/>
  <TextBox x:Name="emailInput"/>
  <Button Content="Submit" Click="Submit_Click"/>
</StackPanel>

<!-- ✓ Semantic form -->
<StackPanel x:Name="loginForm">
  <TextBlock Text="Email"/>
  <TextBox x:Name="emailInput" AutomationProperties.Name="Email" AutomationProperties.IsRequiredForForm="True"/>
  <Button Content="Submit" Click="Submit_Click"/>
</StackPanel>
```

**Use AutomationProperties only when native controls won't work:**

```xaml
<!-- ✓ GOOD - AutomationProperties for custom TabView -->
<TabView>
  <TabViewItem Header="Description" AutomationProperties.Name="Description tab"/>
  <TabViewItem Header="Reviews" AutomationProperties.Name="Reviews tab"/>
</TabView>

<!-- ✓ GOOD - AutomationProperties.Name for icon buttons -->
<Button AutomationProperties.Name="Close dialog">
  <FontIcon FontFamily="Segoe MDL2 Assets" Glyph="&#xE7E6;"/>
</Button>

<!-- ✓ GOOD - AutomationProperties.HelpText for error messages -->
<TextBox x:Name="emailInput" 
         AutomationProperties.IsInvalidForForm="True"
         AutomationProperties.HelpText="Error: Invalid email format"/>
```

**Validation Rules:**
- [ ] Buttons are `<Button>` elements
- [ ] Links are `<HyperlinkButton>` elements
- [ ] Forms use semantic structure (StackPanel/Grid with organized layout)
- [ ] Inputs have associated TextBlock labels
- [ ] No custom role behavior when native controls work
- [ ] Icon buttons have AutomationProperties.Name
- [ ] Custom controls have proper AutomationProperties
- [ ] Error messages have AutomationProperties.HelpText or AutomationProperties.IsInvalidForForm

---

## Testing Accessibility

**Automated tools:**
```bash
# Accessibility Insights for Windows
# https://accessibilityinsights.io/docs/en/windows/

# Microsoft Inspect tool (included in Windows SDK)
# Accessible UI tree inspection

# NVDA Screen Reader
# https://www.nvaccess.org/

# Narrator (built into Windows)
# Settings → Ease of Access → Narrator
```

**Manual testing—test with assistive technologies:**
- [ ] **Keyboard navigation:** Tab through entire app interface
- [ ] **Screen reader (Narrator/NVDA):** Listen to app content and UI
- [ ] **Zoom:** Test at 200% zoom (no horizontal scroll)
- [ ] **High Contrast:** Windows High Contrast Mode settings
- [ ] **Reduced motion:** Windows Settings → Ease of Access → Display → Show animations

---

## Security Standards

### 2.1 Input Validation

**Requirement:** Prevent injection and reflection attacks

**What to Check:**

```csharp
// ✗ BAD - Dynamic XAML parsing (injection risk)
string userInput = GetUserInput();
UIElement control = XamlReader.Load(userInput);  // DANGEROUS!

// ✓ GOOD - User input as TextBlock content (safe)
TextBlock textBlock = new TextBlock();
textBlock.Text = userInput;  // Safe - renders as plain text

// ✓ GOOD - Validate and sanitize if needed
if (Uri.TryCreate(userInput, UriKind.Absolute, out Uri result))
{
  // Safe to use URL
}
```

**Validation Rules:**
- [ ] No dynamic XAML parsing from user input
- [ ] No `XamlReader.Load()` with untrusted input
- [ ] No reflection with user-controlled data
- [ ] User input sanitized before display as text
- [ ] URL validation before navigation (Uri.TryCreate)

---

### 2.2 Secrets & Environment Variables

**Requirement:** Never expose API keys or secrets

**What to Check:**

```csharp
// ✗ BAD - Hardcoded secret
private const string API_KEY = "sk_live_12345abcde";
var url = $"https://api.example.com/data?key={API_KEY}";

// ✓ GOOD - Environment variable or app settings
var apiKey = Environment.GetEnvironmentVariable("SYNCFUSION_LICENSE_KEY");
var url = $"https://api.example.com/data?key={apiKey}";

// ✓ GOOD - Secret in user secrets (development) or Azure Key Vault (production)
// Development: Right-click project → Manage User Secrets
// {
//   "SyncfusionLicenseKey": "xxx...xxx"
// }
```

**Validation Rules:**
- [ ] No hardcoded API keys
- [ ] No hardcoded database URLs or connection strings
- [ ] Secrets from environment variables or User Secrets
- [ ] appsettings.json in .gitignore
- [ ] Production secrets from Azure Key Vault or similar
- [ ] Secrets documented as required env vars

---

### 2.3 Dependency Security

**Requirement:** Use trustworthy, well-maintained packages

**What to Check:**

```xml
<ItemGroup>
  <PackageReference Include="Syncfusion.UI.Xaml.Grids" Version="25.1.35" />
  <PackageReference Include="Syncfusion.UI.Xaml.Buttons" Version="25.1.35" />
</ItemGroup>
```

**Validation Rules:**
- [ ] All packages from official NuGet registry (nuget.org)
- [ ] No typosquatted package names
- [ ] Syncfusion packages only from Syncfusion namespace (official source)
- [ ] Run `dotnet list package --outdated` regularly
- [ ] No deprecated or abandoned packages
- [ ] Pin versions to known safe versions

---

## Performance Standards

### 3.1 Rendering Optimization

**Requirement:** Prevent unnecessary control refreshes

**What to Check:**

```csharp
// ✗ BAD - Rebuilds entire UI on every update
public void UpdateData(string newData)
{
  this.Children.Clear();
  this.Children.Add(new TextBlock { Text = newData });
}

// ✓ GOOD - Only update the data, not the UI structure
private TextBlock _textBlock;

public void InitializeUI()
{
  _textBlock = new TextBlock();
  this.Children.Add(_textBlock);
}

public void UpdateData(string newData)
{
  _textBlock.Text = newData;  // Only update property
}

// ✓ GOOD - Use virtualization for large lists
<ListView ItemsSource="{Binding Items}" ScrollViewer.IsVerticalScrollChainingEnabled="False">
  <ListView.ItemTemplate>
    <DataTemplate>
      <TextBlock Text="{Binding Name}"/>
    </DataTemplate>
  </ListView.ItemTemplate>
</ListView>
```

**Validation Rules:**
- [ ] Large DataGrids use virtualization
- [ ] ItemsControls use ObservableCollection instead of rebuilding
- [ ] Stable event handlers stored in fields
- [ ] No infinite loops in property changes
- [ ] Cleanup handlers when disposing (subscriptions, timers)

---

### 3.2 Assembly Size

**Requirement:** Keep control assembly size reasonable

**Validation Rules:**
- [ ] Control assembly < 500KB (uncompressed)
- [ ] No duplicate dependencies or duplicate Syncfusion packages
- [ ] No large embedded resources (images)
- [ ] Lazy-loadable assemblies for large features (> 1MB)
- [ ] Use IL trimming for Release builds

---

## Code Quality Standards

### 4.1 C# & Type Safety

**Requirement:** Full type safety, no dynamic types

**What to Check:**

```csharp
// ✗ BAD - Using dynamic (loses type safety)
public void HandleChange(dynamic e)
{
  SetValue(e.Value);  // No compile-time checking
}

// ✓ GOOD - Explicit types
public class ChangeEventArgs
{
  public string Value { get; set; }
}

public void HandleChange(ChangeEventArgs e)
{
  SetValue(e.Value);  // Type-safe
}
```

**Validation Rules:**
- [ ] No `dynamic` types (except explicit escape)
- [ ] Properties have explicit types
- [ ] Event handlers properly typed (EventHandler<T>)
- [ ] Dependencies defined with interfaces
- [ ] Return types on all methods
- [ ] Nullable reference types enabled

---

### 4.2 Code Hygiene

**Requirement:** Clean, maintainable code

**What to Check:**

```csharp
// ✗ BAD - System.Diagnostics.Debug in production
var name = "user";
Debug.WriteLine("Debug: " + name);
public class Component { }

// ✓ GOOD - Clean, no debug statements
private const string DefaultName = "user";
public class Component { }
```

**Validation Rules:**
- [ ] No `System.Diagnostics.Debug.WriteLine()` in production code
- [ ] No `Console.WriteLine()` in library code
- [ ] No unused variables or using statements
- [ ] No commented-out code blocks
- [ ] Consistent indentation (4 spaces for C#)
- [ ] Follows Microsoft C# naming conventions
- [ ] Braces on separate lines (Allman style)

---

## Validation Checklist

**WCAG 2.2 AA Accessibility Checklist—run for every control:**

```
PERCEIVABLE
  ✓ All images have descriptive AutomationProperties.Name
  ✓ Icon buttons have AutomationProperties.Name or ToolTip
  ✓ Videos have captions
  ✓ Videos have audio descriptions
  ✓ Color contrast ≥ 4.5:1 for text (or 3:1 for large)
  ✓ Color contrast ≥ 3:1 for UI controls
  ✓ Information not conveyed by color alone
  ✓ No auto-playing audio
  ✓ Focus indicators have 3:1 contrast

OPERABLE
  ✓ All functionality accessible via keyboard
  ✓ Tab order logical (left-to-right, top-to-bottom)
  ✓ No keyboard traps (can Tab out)
  ✓ Focus indicators visible (never hide FocusVisuals)
  ✓ Focus visual ≥ 2px, ≥ 3:1 contrast
  ✓ Focused control not obscured by sticky headers
  ✓ Interactive targets ≥ 24×24 effective pixels
  ✓ Dragging has single-pointer alternative (buttons)
  ✓ Escape closes dialogs/flyouts
  ✓ Enter submits forms, Space activates buttons
  ✓ Arrow keys work for lists/dropdowns/tabs
  ✓ Navigation consistent across app

UNDERSTANDABLE
  ✓ App language set via ApplicationLanguages.PrimaryLanguageOverride
  ✓ Text hierarchy follows semantic levels
  ✓ TextBlock hierarchy correct (no skipping levels)
  ✓ Labels descriptive
  ✓ Every form control has associated TextBlock label
  ✓ Required fields marked (* or AutomationProperties.IsRequiredForForm="True")
  ✓ Error messages clear and specific
  ✓ Error messages linked via AutomationProperties.HelpText
  ✓ Form validation at appropriate times (LostFocus, submit)
  ✓ Information not re-requested (auto-fill where possible)
  ✓ Navigation consistent across app pages
  ✓ Help mechanisms in same relative order (NEW in 2.2)
  ✓ Login not purely cognitive test (NEW in 2.2)

ROBUST
  ✓ Native <Button>, <HyperlinkButton>, <TextBox>, <CheckBox> controls used
  ✓ AutomationProperties only when native controls won't work
  ✓ Icon buttons have AutomationProperties.Name
  ✓ Error fields have AutomationProperties.IsInvalidForForm="True"
  ✓ Error messages have AutomationProperties.HelpText
  ✓ Custom controls have proper AutomationProperties
  ✓ Proper AutomationProperties attributes (AccessibilityView, IsInvalidForForm, etc.)

SECURITY
  ✓ No dynamic XAML parsing from user input
  ✓ No XamlReader.Load() with untrusted input
  ✓ No reflection with user-controlled data
  ✓ User input sanitized before display
  ✓ No hardcoded API keys or secrets
  ✓ Secrets from Environment variables or User Secrets
  ✓ All packages from official NuGet registry

PERFORMANCE
  ✓ Large DataGrids use virtualization
  ✓ ItemsControls use ObservableCollection
  ✓ Stable event handlers stored in fields
  ✓ No infinite loops in property changes
  ✓ Cleanup handlers on disposal (subscriptions, timers)
  ✓ Assembly code < 500KB uncompressed

CODE QUALITY
  ✓ Full C# types (no dynamic)
  ✓ Dependencies have explicit interfaces
  ✓ Event handlers properly typed (EventHandler<T>)
  ✓ Return types on all methods
  ✓ Nullable reference types enabled
  ✓ No Debug.WriteLine() in production
  ✓ No unused variables or using statements
  ✓ No commented-out code
  ✓ Consistent indentation (4 spaces)
  ✓ Follows Microsoft naming conventions
```

---

## Auto-Fix Rules

**Stage 6 automatically fixes these issues:**

| Issue | Auto-Fix |
|-------|----------|
| Missing AutomationProperties.Name on icon button | Add based on icon context |
| Missing AutomationProperties.HelpText on error field | Add description text |
| Missing TextBlock label association | Add TextBlock with name |
| Missing return types on methods | Infer and add return type |
| Debug.WriteLine() in code | Remove or comment out |
| Unused using statements | Remove from imports |
| TextBlock hierarchy gaps | Reorder to proper hierarchy |
| Missing focus indicator | Add FocusVisualPrimaryBrush (never remove) |
| Non-semantic controls | Convert StackPanel buttons to <Button> |
| Missing alt text for images | Add AutomationProperties.Name (requires manual review) |

---

**End of WinUI Standards Reference**  
Updated for **WCAG 2.2 AA** with NEW criteria: Focus not obscured (2.4.11), Target size (2.5.8), Dragging alternatives (2.5.7), Redundant entry (3.3.7), Accessible authentication (3.3.8)   
For Build issues, see `build.md`
