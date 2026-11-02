# Security policy

## Supported versions

Security fixes are released for the latest version only. Please update before reporting a problem.

## Reporting a vulnerability

Please **do not** open a public issue for security problems. Report them privately through
[GitHub private vulnerability reporting](https://github.com/yusuf-toktas/GodotVersionManager/security/advisories/new)
(Security tab → *Report a vulnerability*).

Include what you found, how to reproduce it and the impact you expect. You will get an answer as soon as possible,
usually within a week, and credit in the release notes if you wish.

## How the app protects you

- **Trusted sources only.** Godot builds are downloaded exclusively from HTTPS URLs on `github.com` that belong to the
  official [godotengine/godot-builds](https://github.com/godotengine/godot-builds) releases.
- **Integrity checks.** When GitHub publishes a SHA-256 digest for a release asset, the download is verified against it
  and discarded on mismatch. Every installed file is hashed and recorded in the installation manifest.
- **Safe extraction.** Archives are checked for absolute paths, path traversal ("Zip Slip"), invalid names, oversized
  entries and excessive total size before anything is written. Files are extracted to a private staging folder and only
  moved into place once the installation is complete.
- **Careful removal.** Uninstalling deletes only the recorded installation folder and the shortcuts the app created,
  and refuses to delete drive roots or well-known system and user folders.
- **Least privilege.** The app and its installer run without administrator rights; everything lives in the user profile.
- **Minimal network use.** The app talks to `api.github.com` and GitHub's download servers only. An optional
  `GVM_GITHUB_TOKEN` is sent to `api.github.com` exclusively. Links open in your browser only if they are `http(s)` URLs.
