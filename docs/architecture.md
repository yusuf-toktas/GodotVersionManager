# Architecture

Godot Version Manager is a .NET 10 WPF application split into three projects with a one-way dependency chain:

```text
GodotVersionManager.App  ──►  GodotVersionManager.Infrastructure  ──►  GodotVersionManager.Core
        (WPF, MVVM)              (I/O: HTTP, files, Windows)             (models, interfaces)
```

## Core

Pure domain code without I/O:

- `GodotReleaseVersion` parses and orders Godot tags (`4.3-stable`, `4.0-alpha17`, `4.6-rc2`, …).
  Order: dev < alpha < beta < rc < stable.
- `GodotRelease` / `GodotReleaseArtifact` describe a release and its Windows downloads (edition, architecture, URL,
  optional publisher SHA-256).
- `ReleaseArtifactResolver` (+ `ResolveDefault`) picks the artifact for an edition, preferring x64, then ARM64, then x86.
  It never silently swaps editions.
- Interfaces for everything with side effects (`IReleaseCatalogProvider`, `IDownloadService`, `IInstallationManager`, …).

## Infrastructure

| Area | Type | Notes |
| --- | --- | --- |
| Catalog | `GitHubReleasesCatalogProvider` | Pages through the GitHub releases API of `godotengine/godot-builds`; `WindowsArtifactClassifier` maps asset names to editions/architectures and parses `sha256:` digests. |
| | `CachedReleaseCatalogProvider` | One-hour JSON cache (`IReleaseCatalogStatus` reports age/staleness). `RefreshAsync` bypasses it; if GitHub fails, an expired cache is served instead. |
| Downloads | `HttpDownloadService` | Streams to `<file>.part`, resumes with HTTP `Range`, handles `416`, detects stalled reads, retries transient errors and reports progress (session speed, ETA). |
| Installation | `InstallationManager` | Download → verify → extract to staging → locate `Godot_v*.exe` (ignoring `*_console.exe`) → hash files → commit (rename, or copy across volumes) → shortcuts → manifest. Any failure rolls everything back. |
| | `SafeZipArchiveExtractor` | Rejects absolute paths, traversal, invalid names and oversized archives before writing. |
| Storage | `JsonInstallationRepository` | Installation manifests in `installations.json`; reads and writes share one lock and the state is cached. |
| | `JsonSettingsService` | `settings.json` (enums stored as strings, defaults for missing fields). |
| System | `WindowsShortcutService`, `SystemShell`, `DotNetSdkDetector`, `PrerequisiteChecker` | Shell links, launching, `dotnet --list-sdks`, .NET SDK requirements per Godot version. |

## App

The WPF app uses a small home-grown MVVM layer (`ObservableObject`, `RelayCommand`, `AsyncRelayCommand`) and
`Microsoft.Extensions.DependencyInjection`.

**Shared services** (singletons, one source of truth for every page):

- `CatalogService` — the release list, latest stable/preview, loading and error state. Concurrent loads share one fetch.
- `EngineService` — installed engines and running `InstallOperation`s (progress, cancel, retry), launching,
  uninstalling and shortcut creation. Reports every outcome through `NotificationService` toasts.
- `DialogService` — themed in-window confirmation dialogs.
- `UpdateService` — compares the app version with the latest GitHub release of this repository.
- `ThemeService` — swaps `Themes/Colors.Dark.xaml` / `Colors.Light.xaml` at runtime and follows the Windows setting.
- `LocalizationManager` — loads the embedded `Localization/Languages/*.json`; XAML binds through `{l:Loc Key}`, so a
  language change updates the whole UI immediately.

**Pages**: Home, Library, Installed, Downloads, Settings and About, each a view model with a matching view. The Library
uses a filtered `ListCollectionView` over `ReleaseItemViewModel`s, each with one `EditionSlotViewModel` per edition that
shows *install*, *progress* or *launch*.

**Shell**: `MainWindow` draws its own title bar with `WindowChrome`, uses DWM for dark borders and rounded corners, and
hosts the toast and dialog overlays. `SingleInstanceGuard` keeps one running instance (the installer uses the same mutex
name to detect a running app).

## Data locations

`AppPaths` resolves everything under `%LOCALAPPDATA%\GodotVersionManager` (or `GodotVersionManagerData` next to the
executable in portable mode): `Engines`, `Downloads`, `Staging`, `Cache`, `Data`, `Logs`. Staging leftovers are removed
at startup; logs rotate at 5 MB.

## Packaging

`build/build.ps1` publishes a self-contained, compressed single-file executable (`DebugType=embedded`, so no `.pdb`
files) and builds the Inno Setup installer from `installer/InnoSetup/GodotVersionManager.iss`. Brand assets (icon,
installer artwork, README images) are rendered from `Themes/Brand.xaml` by `build/generate-assets.cs`
(`dotnet run build/generate-assets.cs -- .`).
