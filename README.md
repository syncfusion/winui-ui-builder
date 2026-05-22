# Syncfusion WinUI UI Builder

**Syncfusion WinUI UI Builder** is an AI-powered agent skill that transforms your UI requirements into production-ready WinUI controls through an automated 8-stage orchestration workflow. It leverages Syncfusion's extensive WinUI Control library to generate accessible, responsive, and brand-aligned user interfaces.

### Key Features

- **8-Stage AI Orchestration**: Intelligent workflow handling design thinking, component selection, code generation, and validation
- **Syncfusion Integration**: Access to professional WinUI 3 controls powered by Syncfusion
- **WCAG 2.1 AA Accessibility**: Built-in accessibility compliance with proper AutomationProperties and keyboard navigation
- **Responsive Design**: DPI-aware sizing with adaptive breakpoints
- **Design System Support**: Syncfusion theme alignment with locked design tokens

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [How It Works](#how-it-works)
- [Usage](#usage)
- [Support](#support)

## Prerequisites

Before using Syncfusion WinUI UI Builder, ensure your environment meets these requirements:

| Requirement | Description |
|-------------|-------------|
| **APM** | [Install APM](https://microsoft.github.io/apm/getting-started/installation/) - Agent Package Manager for skill installation |
| **WinUI Project** | Active WinUI 3 project (.NET 8.0+, Windows App SDK 1.8+) |
| **.NET SDK 8.0+** | Required for WinUI 3 development |
| **Visual Studio 2022+** | Visual Studio 2022 or later with v17.8+ with WinUI 3 workload |
| **Syncfusion License** | [Commercial](https://www.syncfusion.com/sales/unlimitedlicense), [Free Community](https://www.syncfusion.com/products/communitylicense), or [Free Trial](https://www.syncfusion.com/account/manage-trials/start-trials) |

## Installation

```bash
# Install for GitHub Copilot
apm install syncfusion/winui-ui-builder -t copilot

# Install for Claude Code
apm install syncfusion/winui-ui-builder -t claude

# Install for Cursor
apm install syncfusion/winui-ui-builder -t cursor

# Install for Codex
apm install syncfusion/winui-ui-builder -t codex
```

## How It Works

The **Syncfusion WinUI UI Builder** skill orchestrates **8 stages of pure AI reasoning** with **two user decision points**.


**Stage Descriptions:**

| Stage | Name | Description |
|-------|------|-------------|
| 1 | Intent Analysis | Parse user query, identify control type and features |
| 2 | Project Detection | Auto-detect framework, .NET version, theming strategy |
| 3 | Layout Analysis & Control Mapping | Create optimal control-mapping.json, map to Syncfusion controls |
| 4 | Theming & Design System | Lock design tokens, Syncfusion theme, color system, spacing |
| 5 | Code Generation | Generate WinUI XAML, C#, data models with theming applied |
| 6 | Dependencies | Detect and install NuGet packages |
| 7 | Validation | Validate WCAG 2.1 AA, security, performance, theming |
| 8 | Code Insertion | Insert files into project, verify build |

## Usage

Invoke the skill through your AI assistant by describing what you want to build:

**Example 1: Generate a Login Form**
```
Create a login form with email, password, and remember me checkbox
```

**Example 2: Generate a Data Table**
```
Build a customer data table with sorting and filtering
```

**Example 3: Generate a Dashboard**

Choose the `syncfusion-winui-ui-builder` agent in the AI chat panel and invoke the skill through your AI assistant by describing what you want to build:

```text
Create a WinUI 3 CMS Admin Dashboard using Syncfusion controls. Features:
1. Shell: NavigationView with items for Dashboard, Content, Users, Analytics, and Settings.
2. Header: Custom AppBar with "CMS Admin Dashboard" title and a profile button with person icon.
3. Statistics: A row of SfCardLayouts for "Total Content", "Total Users", and "Active Sessions", each showing a label, SfBadge, and percentage change.
4. Content: SfDataGrid with columns for Title, Author, Status, Date, and Actions, including sorting and filtering. 
5. Visuals: Side-by-side SfChart (Bar chart for "Content Over Time" and Donut chart for "Content by Category").
Use realistic sample data for 10-12 rows
```

Generated code follows best practices with WCAG 2.1 AA accessibility compliance, DPI-aware responsive design, secure input validation, and proper INotifyPropertyChanged implementation.

## Best Practices
- Maintain consistency in file structure, naming, and coding standards.
- Use advanced AI models (e.g., Claude Sonnet 4.6+) for better code quality.
- Review everything before production—replace placeholders and verify logic, security, and compatibility.

## Support

Product support is available through the following media:

- [Support ticket](https://support.syncfusion.com/support/tickets/create) - Guaranteed response in 24 hours | Unlimited tickets | Holiday support
- [Community forum](https://www.syncfusion.com/forums/winui)
- [Request feature or report bug](https://www.syncfusion.com/feedback/winui)
- [Live chat](https://www.syncfusion.com/support)
