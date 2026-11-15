# Engine Launcher

A modern WPF desktop hub (`.NET Framework 4.8`) for managing, updating, and launching the **Engine Suite** applications. Features glassmorphic theming, HWID-backed authentication and tool whitelisting, AES-256-CBC encrypted package delivery, and dynamic remote manifest (`versions.json`) synchronization.

> **Current Version:** `v1.3` (`1.3.0.0`)

---

## Key Features

### Suite Hub & App Management
- **Unified Dashboard:** Launch installed tools directly, install missing modules, trigger one-click updates, or uninstall existing tools cleanly.
- **Versions & Updates Tab:** Dedicated per-tool and launcher update checker displaying installed PE file versions alongside remote manifest versions and release notes.
- **Automatic Label Synchronization:** Card titles, list rows, tooltips, and uninstall prompts automatically reflect the installed executable version (or remote/default version when not yet installed), keeping labels in sync with status badges (e.g. `BeatUp Engine v1.4` matching `INSTALLED v1.4`).
- **Silent Cleanup:** Cleans up temporary update artifacts (`.old` / `.update.exe`) on startup.

### Dynamic `versions.json` Auto-Detection
- **Automatic `IN DEV` → `READY` Promotion:** Modules registered in development state (such as `Guitar Engine` and `Shooting Engine`) are automatically promoted to downloadable `READY` tools as soon as a valid entry appears in the remote `versions.json` manifest—without requiring a launcher rebuild.
- **Flexible Package & Metadata Matching:**
  - Key matching is case-insensitive and format-tolerant (e.g. `"guitar"`, `"guitarengine"`, `"Guitar Engine"`, `"guitar_engine"`).
  - Supports custom encrypted filename overrides directly in `versions.json` (`"enc"`, `"package"`, `"file"`, `"filename"`, or `"archive"`), plus optional `"exe"`, `"title"`, `"category"`, `"description"`, and `"notes"`.
  - Can explicitly stay or revert to `IN DEV` via `"in_dev": true`, `"ready": false`, `"status": "IN DEV"`, or by removing the section from `versions.json`.
- **Manual Gate for Brand-New Modes:** Unregistered tools do not spawn dynamically from JSON—completely new modes outside the existing `IN DEV` slots are added intentionally via launcher updates.

### Authentication & Encrypted Delivery
- **Account & HWID Gate:** Supports Sign In, Account Registration (with Charter Name), persistent 30-day device sessions, Pending Approval polling, and per-tool whitelist enforcement.
- **AES-256-CBC Package Decryption:** Downloads encrypted `.enc` executables from remote storage with live progress reporting and decrypts directly into each tool's target `apps\` directory.
- **Self-Update Flow:** Checks the `"launcher"` section of `versions.json`, prompts when a newer build is available, downloads the encrypted launcher binary, swaps the executable cleanly, and restarts.

### Settings & Theme Studio
- **Profile Menu & Engine Settings (`SettingsWindow`):**
  - **Profile Settings:** Account overview, charter name, and avatar customization.
  - **Hub & Options:** UI layout toggles (such as hiding tool descriptions on hub cards for a denser grid).
  - **Appearance & Glass:** Quick Light/Dark mode and Windows DWM backdrop / glassmorphism opacity controls.
  - **Network Diagnostics:** Live connection checks for auth backend and manifest/package storage endpoints.
- **Theme & Gradient Editor (`DesignWindow`):**
  - Built-in presets (`Mica Dark`, `Mocha`, `Nord`, `Latte`, `Cloud`) plus custom theme creation.
  - Live preview card, RGB/Hex sliders, toggleable background gradients, Light Mode conversion, and transparency controls.

---

## Included Tools Registry

| ID | Default Name | Relative Executable Path | Default State |
| --- | --- | --- | --- |
| `beatup` | BeatUp Engine | `apps\BeatUpEngine\BeatUpEngine.exe` | Ready |
| `ballroom` | Ballroom Converter | `apps\SanBallroom\SanBallroom.exe` | Ready |
| `silencer` | Silencer | `apps\Silencer\Silencer.exe` | Ready |
| `auconverter` | Au Converter | `apps\AuConverter\AuConverter.exe` | Ready |
| `onetwo` | One-Two Party Engine | `apps\OneTwoParty\OTPEngine.exe` | Ready |
| `aupiano` | AuPiano Engine | `apps\AuPiano\AuPianoEngine.exe` | Ready |
| `guitar` | Guitar Engine | `apps\GuitarEngine\GuitarEngine.exe` | In Dev *(auto-unlocks via `versions.json`)* |
| `shooting` | Shooting Engine | `apps\ShootingEngine\ShootingEngine.exe` | In Dev *(auto-unlocks via `versions.json`)* |

---

## Project Structure

```text
Engine Suite Launcher WPF/
├── Assets/                  # Launcher & per-tool .ico icons
├── Properties/
│   └── AssemblyInfo.cs      # Assembly & File version (used at runtime)
├── Services/
│   ├── AuthService.cs       # Auth flow, HWID verification, whitelist checks
│   ├── BackdropHelper.cs    # Windows DWM acrylic / mica / blur interop
│   ├── DesignManager.cs     # Theme engine, presets, RGB/gradient brushes
│   ├── ObfuscatedSecrets.cs # Endpoint & vault configuration (redacted in public repos)
│   ├── SettingsManager.cs   # Local user preferences & UI toggles
│   └── SuiteManager.cs      # Manifest parsing, version sync, AES decryption, installs
├── App.xaml / App.xaml.cs
├── MainWindow.xaml(.cs)     # Auth view, Suite Hub cards, Versions & Updates view
├── SettingsWindow.xaml(.cs) # Multi-tab settings dialog
├── DesignWindow.xaml(.cs)   # Theme & Gradient Studio
├── EngineLauncher.crproj    # ConfuserEx obfuscation project (Release builds)
└── EngineLauncherWPF.csproj
```

---

## Building from Source

### Prerequisites
- Windows 10 / 11
- Visual Studio 2022+ (or Build Tools) with **.NET Desktop Development** workload (`.NET Framework 4.8`)
- `Confuser.MSBuild` (`1.6.0`, restored automatically via NuGet for Release builds)
- Valid endpoint & vault configuration in `Services/ObfuscatedSecrets.cs`

### Build Commands
**Release build (with ConfuserEx protection):**
```powershell
MSBuild.exe .\EngineLauncherWPF.csproj /p:Configuration=Release /t:Rebuild
```
Output: `bin\Release\EngineLauncher.exe`

**Debug build (unobfuscated, for local testing):**
```powershell
MSBuild.exe .\EngineLauncherWPF.csproj /p:Configuration=Debug /t:Build
```
Output: `bin\Debug\EngineLauncher.exe`

---

## Remote Manifest (`versions.json`) Reference

The launcher fetches a single JSON manifest on startup and during update checks.

### Standard Example
```json
{
  "launcher": {
    "version": "1.3.0.0",
    "notes": "Auto-synced tool labels and automatic IN DEV promotion."
  },
  "beatup": {
    "version": "1.4.0.0",
    "notes": "Release notes shown in Versions & Updates."
  }
}
```

### Promoting an `IN DEV` Mode Live (Example: `guitar`)
Upload the encrypted package (e.g. `guitarengine.enc`) to your package storage and add an entry to `versions.json`:

```json
{
  "guitar": {
    "version": "0.1.0",
    "enc": "guitarengine.enc",
    "exe": "GuitarEngine.exe",
    "notes": "Initial Guitar Engine release."
  }
}
```

| Field | Required | Description |
| --- | --- | --- |
| `version` | Yes | Semantic version string (`1.4.0.0`, `0.2.0`, etc.). |
| `notes` | No | Changelog text shown in update rows and modals. |
| `enc` / `package` / `file` | No | Exact `.enc` filename in remote storage if different from default. |
| `exe` | No | Custom executable filename or relative path under `apps\`. |
| `title` / `category` / `description` | No | Optional metadata overrides for the tool card. |
| `status` / `in_dev` / `ready` | No | Override availability state (`"IN DEV"` / `"READY"` or boolean). |

---

## Security & Publishing Notes
- **Never commit** `Services/ObfuscatedSecrets.cs` values, AES vault keys, session signing secrets, admin tokens, or private worker endpoints to a public repository.
- Encrypted `.enc` artifacts use **AES-256-CBC** with a random 16-byte IV prepended to the ciphertext and PKCS7 padding.
