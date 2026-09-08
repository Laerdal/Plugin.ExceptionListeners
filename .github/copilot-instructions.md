# GitHub Copilot Instructions for Plugin.ExceptionListeners

## Project Overview

**Plugin.ExceptionListeners** is a unified exception-listening framework for .NET, covering
first-chance exceptions, unhandled exceptions, and unobserved Task exceptions through one
consistent event model — plus a `.Maui` companion package for native platform exception handling
(iOS, Android, macOS, Windows).

**Key Features:**
- First-chance exception listening (before application code handles it)
- Unhandled exception monitoring (`AppDomain.CurrentDomain.UnhandledException`)
- Unobserved Task exception handling (`TaskScheduler.UnobservedTaskException`), auto-marked observed
- Unified `ExceptionEventArgs` model shared by every listener
- `IDisposable`-based lifecycle — every listener unsubscribes cleanly on dispose
- MAUI native exception bridging (`NSException`, `Java.Lang.Throwable`, Win32 exceptions)

## Technology Stack

- **SDK:** .NET SDK `10.0.100` (see `global.json`); projects target `net9.0` (core/tests) and `net10.0*` (MAUI)
- **Language:** C# (latest)
- **Testing:** xUnit + FluentAssertions (v7.x — stays open source indefinitely per Directory.Packages.props comment)
- **Package Management:** Central Package Management (`Directory.Packages.props`)
- **CI/CD:** GitHub Actions
- **Documentation:** DocFX (`Plugin.ExceptionListeners.Docs/`, published to GitHub Pages)

## Project Structure

```
Plugin.ExceptionListeners/
├── .github/
│   ├── workflows/ci.yml
│   └── copilot-instructions.md          # This file
├── Plugin.ExceptionListeners/            # Core, platform-agnostic
│   ├── ExceptionEventArgs.cs             # Shared event payload for every listener
│   ├── ExceptionListener.cs              # Base listener abstraction
│   └── Listeners/
│       ├── CurrentDomainFirstChanceExceptionListener.cs
│       ├── CurrentDomainUnhandledExceptionListener.cs
│       └── TaskSchedulerUnobservedTaskExceptionListener.cs
├── Plugin.ExceptionListeners.Maui/       # MAUI-specific native exception bridging
│   ├── Imports.cs
│   ├── NativeUnhandledException.cs
│   └── NativeUnhandledExceptionListener.cs
├── Plugin.ExceptionListeners.Tests/      # xUnit tests
└── Plugin.ExceptionListeners.Docs/       # DocFX documentation site
```

## Development Setup

### Prerequisites
- .NET SDK 10.0.100 or later (see `global.json`)
- For `Plugin.ExceptionListeners.Maui`: MAUI workloads installed

### Build Commands
```bash
dotnet restore Plugin.ExceptionListeners.slnx
dotnet build Plugin.ExceptionListeners.slnx --configuration Release --no-restore
dotnet test Plugin.ExceptionListeners.Tests/Plugin.ExceptionListeners.Tests.csproj --configuration Release --no-build
```

## Code Organization Patterns

- **Platform-agnostic core stays in `Plugin.ExceptionListeners`.** Anything that touches a native
  platform exception type (`NSException`, `Java.Lang.Throwable`, Win32 exceptions) belongs in
  `Plugin.ExceptionListeners.Maui`, not the core package.
- **Every listener** implements `IDisposable`, subscribes in its constructor, and unsubscribes on
  `Dispose()` — no listener should leak its underlying event subscription.
- **Every listener raises the shared `ExceptionEventArgs`** — don't introduce a parallel event
  payload type; extend `ExceptionEventArgs` if a new source needs extra data.
- New listener sources go in `Listeners/` (core) following the existing `*Listener.cs` naming.

## Coding Standards

### Naming Conventions (enforced by `.editorconfig`)
- Classes/methods/properties/constants: PascalCase
- Private fields: `_camelCase`
- Parameters/locals: camelCase

### Style Guidelines
- Line length: max 240 characters; indentation: 4 spaces; line endings: LF
- Braces always required, even for single-line statements
- `System` usings first, separated from the rest

### Documentation Requirements
- XML docs (`<summary>`, `<param>`, `<returns>`, `<exception>`) required on all public APIs

## Testing Guidelines

- **Framework:** xUnit, assertions via FluentAssertions (v7.2.2, open-source)
- **File naming:** `{ClassName}Tests.cs`, matching the source file
- **Test naming:** `MethodName_Scenario_ExpectedBehavior`
- For each listener: cover the exception being raised, the event firing with correct
  `ExceptionEventArgs`, and clean unsubscription on `Dispose()`

## Dependencies

### Production (core)
- `Microsoft.Extensions.Logging.Abstractions` (10.0.1), `Microsoft.Extensions.Logging.Debug` (10.0.1)

### Production (Maui)
- `Microsoft.Maui.Core` / `.Controls` / `.Essentials` (`$(MauiVersion)`)
- `CommunityToolkit.Mvvm` (8.4.0), `CommunityToolkit.Maui` (12.3.0)
- `NLog.Extensions.Logging` (6.1.0), `NLog.Targets.MauiLog` (10.0.3)

### Development
- `xunit` (2.9.3), `xunit.runner.visualstudio` (4.0.0), `FluentAssertions` (7.2.2)
- `coverlet.collector` / `coverlet.msbuild` (10.0.1)
- `Microsoft.NET.Test.Sdk` (18.9.0), `JetBrains.Annotations` (2025.2.0)
- `Microsoft.CodeAnalysis.NetAnalyzers` (10.0.400), `Microsoft.SourceLink.GitHub` (10.0.400)

(Versions drift via Dependabot — check `Directory.Packages.props` for current values rather than
trusting this list long-term. Note: `Plugin.ByteArrays`/`Plugin.BaseTypeExtensions`/
`Plugin.ExceptionListeners` package versions are declared in `Directory.Packages.props` under
"INTERNAL DEPENDENCIES" but are not currently referenced by any `.csproj` in this repo — don't
assume they're an active dependency without checking.)

## CI/CD Pipeline

`ci.yml` runs: version generation → build/test/package → DocFX build and deploy to GitHub Pages →
publish to NuGet.org (on `main`) → tag & GitHub release.

## Useful Resources

- **Repository:** https://github.com/laerdal/Plugin.ExceptionListeners
- **NuGet Package:** https://www.nuget.org/packages/Plugin.ExceptionListeners
- **Documentation:** https://laerdal.github.io/Plugin.ExceptionListeners/
