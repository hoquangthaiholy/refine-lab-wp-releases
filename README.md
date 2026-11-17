# Refine Lab: releases

Public release channel for the **Refine Lab** WordPress plugin (`refine-lab-wp`). Source code lives elsewhere; this repository only publishes installable builds and the update manifest.

- **Install:** download `refine-lab-wp-<version>.zip` from the [Releases](../../releases) page and upload it under *Plugins → Add New → Upload Plugin*.
- **Update:** installed copies check `update.json` on this repository and offer updates in *Plugins* and on the Refine Lab *Overview* page. Every package is verified against the SHA-256 in `update.json` before WordPress installs it.
- **Documentation:** [docs/](docs/README.md): HTML block syntax with examples, forms, languages, backup and restore, site tools, hooks and API reference. Also in Traditional Chinese: [繁體中文文件](docs/zh-TW/README.md).
- **Website:** https://refine-lab.com/

## Manifest (`update.json`)

| Field | Meaning |
|---|---|
| `version` | Latest released version |
| `package` | Release asset URL of the zip (must be in this repository) |
| `sha256` | SHA-256 of the zip |
| `requires`, `requires_php`, `tested` | Compatibility |
| `released` | Release date |
| `sections.changelog` | HTML shown in "What is new" |

## License

GPL-2.0-or-later, like the plugin.
