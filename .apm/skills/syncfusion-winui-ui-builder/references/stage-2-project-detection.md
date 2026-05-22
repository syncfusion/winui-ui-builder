# Stage 2: Project Detection

## Purpose
Auto-detect WinUI project structure and configuration, or create new project if none found.

## Project Search Strategy
1. Scan workspace root for `.csproj` files
2. Search subdirectories recursively (max 5 levels, skip `obj/`, `bin/`, `.git/`, `node_modules/`)
3. Detect WinUI indicators: `MainWindow.xaml`, `app.xaml`, `Package.appxmanifest`
4. If multiple projects found, prompt user to select

## WinUI Project Creation (CLI Primary Flow)

### Prerequisites Check
```bash
dotnet --version
```
- Verify .NET 6.0+ is installed

### List Available Templates
```bash
dotnet new --list
```
- Confirm `winui3` template is available

### Create WinUI Project
```bash
dotnet new winui3 -n <ProjectName>
```
- Creates complete WinUI 3 project with:
  - `.csproj` with Windows App SDK 1.5.0+
  - `MainWindow.xaml` and `App.xaml`
  - Standard folder structure (Views, Models, ViewModels)
  - .NET 8.0 targeting windows10.0.19041.0

### Fallback: Manual .csproj (if CLI unavailable)
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows10.0.19041.0</TargetFramework>
    <WindowsAppSDKSelfContained>true</WindowsAppSDKSelfContained>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.5.240311000" />
  </ItemGroup>
</Project>
```

## Project Detection Steps

1. **Locate .csproj**
   - File path: Required for project root identification
   - Extract: TargetFramework, .NET version, Windows App SDK version

2. **Scan for Configuration Files**
   - `.editorconfig` - Apply coding style conventions
   - `appsettings.json` - Check for `SYNCFUSION_LICENSE_KEY`
   - `app.manifest` - Verify Windows packaging

3. **Detect Syncfusion Integration**
   - Scan `.csproj` for `Syncfusion.UI.Xaml.*` packages
   - Extract version number (e.g., `25.1.35`)
   - Use same version for ALL new packages (prevents conflicts)
   - If no Syncfusion packages exist, use latest stable version

4. **Establish Directory Structure**
   - Project Root: Directory containing `.csproj`
   - Standard folders: `Views/`, `Models/`, `ViewModels/`, `Controls/`
   - **CRITICAL:** ALL generated files must be inside Project Root

## User Prompts

### No Project Found
```
⚠ No WinUI project detected

[Create New WinUI Project] [Specify Path] [Cancel]

If Create Selected:
  - Project name: ____
  - .NET version: [8.0] [7.0] [6.0]
  - Add Syncfusion: [Yes/No]
```

### Project Found
```
✓ Framework: WinUI 3
✓ .NET: .NET 8
✓ Language: C#
✓ Syncfusion: 25.1.35 (detected) or Latest
✓ View Dir: Views/

[Confirm] [Override] [Cancel]
```

## Detection Results
- **Confirmed:** Proceed to Stage 3 (Configuration)
- **Override:** Allow custom version/settings selection
- **Cancelled:** Stop process
