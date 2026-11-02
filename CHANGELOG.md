# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-11-02

First public release.

### Added

- Release catalog read live from [godotengine/godot-builds](https://github.com/godotengine/godot-builds), cached for an
  hour and still available offline when GitHub cannot be reached. Alpha releases are recognized.
- Standard and C# editions (.NET for Godot 4, Mono for Godot 3) can be installed side by side; after installing a .NET
  edition the app checks for the required .NET SDK.
- SHA-256 verification of every download for which GitHub publishes an asset digest.
- Resumable downloads with stall detection, automatic retries and recovery from HTTP 416 responses.
- Redesigned interface: custom window chrome, dark / light / system themes, Home dashboard, Library with search and
  filters, Installed and Downloads pages, About page with project and Godot links, in-app dialogs and notifications.
- Localization in English, Turkish, German, Spanish, French, Portuguese (Brazil), Russian, Chinese (Simplified) and
  Japanese, switchable while the app is running.
- Original logo and app icon; branded installer that follows the Windows light / dark setting.
- Settings for the install location (folder picker and validation), shortcuts, theme, language, update check and cache
  cleanup, plus the detected .NET SDK.
- Notification when a newer Godot Version Manager release is available.
- Single-instance app (launching it again brings the open window to the front), portable mode (`portable.txt`) and an
  optional `GVM_GITHUB_TOKEN` for a higher GitHub API limit.
- Self-contained single-file executable and Inno Setup installer, a build script and a GitHub Actions release workflow
  that publishes both with checksums.

### Fixed

- Godot 4 installations always failed because the archives contain an additional `*_console.exe`.
- Library filters never applied, and changing a filter reset installations in progress.
- The Downloads page stayed empty; installations now show phase, size, speed and time remaining and can be cancelled.
- The default install directory setting was ignored, and installing to a different drive than the app data failed.
- The C# edition could not be installed, and the standard and C# builds of a version would have shared one folder.
- Refresh never bypassed the one-hour catalog cache.
- A complete leftover `.part` file (HTTP 416) blocked a download permanently, resumed downloads reported inflated speeds
  and HTTP timeouts were reported as cancellations.
- Reading and writing the installations file at the same time could fail and roll back an installation.
- Uninstall left shortcuts behind and could leave an item stuck in a busy state; it now also refuses to delete protected
  system or user folders.
- Installation dates were displayed in UTC, and several controls were unreadable in the dark theme.
- Unhandled exceptions could close the app without a message, and the log file grew without limit.

[Unreleased]: https://github.com/yusuf-toktas/GodotVersionManager/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/yusuf-toktas/GodotVersionManager/releases/tag/v1.0.0
