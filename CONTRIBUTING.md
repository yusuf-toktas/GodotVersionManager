# Contributing to Godot Version Manager

Thanks for your interest in improving Godot Version Manager! Bug reports, ideas, translations and pull requests are all
welcome. By participating you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Getting started

Requirements:

- Windows 10 or 11
- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- An editor with C# support (Visual Studio 2026, Rider or VS Code with C# Dev Kit)
- Optional: [Inno Setup 6.6+](https://jrsoftware.org/isinfo.php) to build the installer (`winget install JRSoftware.InnoSetup`)

```powershell
git clone https://github.com/yusuf-toktas/GodotVersionManager.git
cd GodotVersionManager
dotnet build
dotnet test
dotnet run --project src/GodotVersionManager.App
```

`./build/build.ps1` produces the release files (standalone executable, installer, checksums) in `artifacts/release`.

> [!TIP]
> To keep your own data untouched while developing, create an empty `portable.txt` next to the built
> `GodotVersionManager.exe` (in `src/GodotVersionManager.App/bin/...`). The app then stores everything beside the
> executable.

## Project layout

| Project | Responsibility |
| --- | --- |
| `GodotVersionManager.Core` | Domain models and interfaces. No I/O. |
| `GodotVersionManager.Infrastructure` | GitHub catalog, HTTP downloads, safe ZIP extraction, installation, JSON storage, Windows integration. |
| `GodotVersionManager.App` | WPF user interface (MVVM), themes, localization, app services. |
| `tests/GodotVersionManager.UnitTests` | Tests for Core and Infrastructure. |
| `tests/GodotVersionManager.App.Tests` | Tests for app logic and the translation files. |
| `tests/GodotVersionManager.IntegrationTests` | Live GitHub tests, skipped unless `GVM_RUN_LIVE_TESTS=1`. |

See [docs/architecture.md](docs/architecture.md) for how the pieces fit together.

## Guidelines

- **Keep the build clean.** `dotnet build` must finish with zero warnings; the code style is defined in `.editorconfig`.
- **Write tests** for behavior changes in Core, Infrastructure and app services. Bug fixes should come with a test that
  fails without the fix.
- **UI changes** must work in both the dark and the light theme. Use the brushes from `Themes/Colors.*.xaml` through
  `DynamicResource` (never hard-coded colors) and reuse the styles in `Themes/Controls.xaml`.
- **No hard-coded UI text.** Every visible string comes from the language files (see below).
- **Downloads stay safe.** Engine builds may only come from HTTPS GitHub URLs; keep checksum verification and the
  archive extraction limits intact.

## Translations

UI text lives in `src/GodotVersionManager.App/Localization/Languages/<code>.json` — one flat JSON object per language.
`en.json` is the reference; missing keys fall back to English at runtime.

**Add a language**

1. Copy `en.json` to a file named after the language code, for example `it.json` or `pt-PT.json`.
2. Set `Language.NativeName` (for example `Italiano`) and `Language.EnglishName` (`Italian`).
3. Translate every value. Keep placeholders such as `{0}` and `{1}` — they are replaced with numbers, names or dates.
4. Add the code to `ShippedLanguages` in `tests/GodotVersionManager.App.Tests/Localization/LocalizationFilesTests.cs`
   and, if WPF ships resources for it, to `SatelliteResourceLanguages` in the app's `.csproj`.
5. Run `dotnet test`. The tests verify that your file has exactly the same keys and placeholders as `en.json`.

New files are embedded and offered in **Settings → Language** automatically.

**Change or add text**: add the key to `en.json` and to every other language file in the same pull request (an
English placeholder is fine if you cannot translate it — mention it in the PR so a native speaker can help).

## Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `docs:`, `refactor:`, `test:`,
`build:`, `ci:`, `chore:`, `i18n:`. Example: `fix: pick the editor executable when a console wrapper is present`.

## Pull requests

1. Fork the repository and create a branch from `main`.
2. Make your change with tests, keeping commits focused.
3. Make sure `dotnet build` has no warnings and `dotnet test` passes.
4. Update `CHANGELOG.md` (under *Unreleased*) for user-facing changes.
5. Open the pull request and fill in the template. Screenshots are very welcome for UI changes.

## Reporting bugs

Please use the [bug report form](https://github.com/yusuf-toktas/GodotVersionManager/issues/new/choose) and include the
app version, your Windows version and, if possible, an excerpt from the log
(`%LOCALAPPDATA%\GodotVersionManager\Logs\gvm.log`). Report security issues privately — see [SECURITY.md](SECURITY.md).
