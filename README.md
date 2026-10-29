# Start Menu: App List Only

A small Windows 10 utility that turns the Start menu into a clean list of installed apps. It removes every tile, locks the layout so nothing can be pinned back, and hides **Pin to Start** from context menus. One click applies it, one click undoes it.

---

<img width="859" height="560" alt="image" src="https://github.com/user-attachments/assets/4476f0b1-9513-4536-96f6-02808511e43d" />

---

<p align="left">
  <img alt="Platform: Windows 10" src="https://img.shields.io/badge/Platform-Windows%2010-0078D6?style=flat-square&logo=windows&logoColor=white">
  <img alt=".NET Framework 4.x" src="https://img.shields.io/badge/.NET%20Framework-4.x-512BD4?style=flat-square&logo=dotnet&logoColor=white">
  <img alt="Built with C# and WinForms" src="https://img.shields.io/badge/Built%20with-C%23%20%2B%20WinForms-239120?style=flat-square">
  <img alt="Installer: NSIS" src="https://img.shields.io/badge/Installer-NSIS-0A66C2?style=flat-square">
  <img alt="Version 1.1.0" src="https://img.shields.io/badge/Version-1.1.0-2EA44F?style=flat-square">
  <img alt="Requires administrator" src="https://img.shields.io/badge/Requires-Administrator-5B5B5B?style=flat-square">
  <img alt="Code signing: unsigned" src="https://img.shields.io/badge/Code%20signing-unsigned-E36209?style=flat-square">
</p>

<!-- Optional dynamic badges: replace OWNER/REPO with your GitHub path, then uncomment.
[![Latest release](https://img.shields.io/github/v/release/OWNER/REPO?style=flat-square)](https://github.com/OWNER/REPO/releases)
[![Downloads](https://img.shields.io/github/downloads/OWNER/REPO/total?style=flat-square)](https://github.com/OWNER/REPO/releases)
[![Issues](https://img.shields.io/github/issues/OWNER/REPO?style=flat-square)](https://github.com/OWNER/REPO/issues)
[![License](https://img.shields.io/github/license/OWNER/REPO?style=flat-square)](LICENSE)
-->

> **Status:** v1.1.0. The installer is **not code-signed**, so Windows will show an "Unknown publisher" warning. See [Unknown Publisher / SmartScreen](#unknown-publisher--smartscreen).

---

## Table of contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Command-line reference](#command-line-reference)
- [What it changes](#what-it-changes)
- [Uninstalling and reverting](#uninstalling-and-reverting)
- [Testing and compatibility](#testing-and-compatibility)
- [Known limitations](#known-limitations)
- [Unknown Publisher / SmartScreen](#unknown-publisher--smartscreen)
- [Building from source](#building-from-source)
- [Troubleshooting](#troubleshooting)
- [Disclaimer](#disclaimer)
- [License](#license)
- [Developer](#developer)

---

## Features

- **Apps list only:** all Start tiles are removed and the layout is locked, so tiles cannot be added or restored.
- **No Pin to Start:** the *Pin to Start* shell handler is blocked, so it disappears from file, folder and shortcut context menus.
- **No promotional tiles:** Windows consumer features (suggested apps and similar) are switched off.
- **Persistent:** machine-wide policies, plus a small startup task that re-applies them if a Windows update resets them.
- **Fully reversible:** original registry values are saved before the first change and restored exactly on revert or uninstall.
- **Native, scalable GUI:** per-monitor DPI aware; text wraps to the window width and the window scrolls instead of cropping on small screens.
- **Scriptable:** command-line switches and exit codes for deployment and automation.
- **Standard installer:** shows up in *Settings > Apps*, supports silent install and uninstall.

## Requirements

| Item | Requirement |
|---|---|
| OS | Windows 10 (x64; 32-bit Windows 10 is untested) |
| Edition | **Pro, Enterprise or Education** recommended. See [Testing and compatibility](#testing-and-compatibility) |
| Runtime | .NET Framework 4.x (included with Windows 10, nothing to install) |
| Rights | Administrator (the app requests elevation itself) |

## Installation

1. Download `StartMenuAppListOnly-Setup-1.1.0.exe` from the [Releases](../../releases) page.
2. *(Recommended)* Verify the download. In PowerShell:
   ```powershell
   Get-FileHash .\StartMenuAppListOnly-Setup-1.1.0.exe -Algorithm SHA256
   ```
   Compare the result with the SHA-256 listed in the release notes (or `SHA256SUMS.txt`).
3. Run the installer. If Windows warns about an unknown publisher, read [that section](#unknown-publisher--smartscreen) first.
4. Launch **Start Menu App List Only** from the Start menu.

**Silent install** (for scripts or deployment, run elevated):

```bat
StartMenuAppListOnly-Setup-1.1.0.exe /S
StartMenuAppListOnly-Setup-1.1.0.exe /S /D=C:\Tools\StartMenuAppListOnly
```

(`/D=` must be the last argument and the path must not be quoted.)

## Usage

1. Open **Start Menu App List Only**. It asks for administrator rights.
2. Click **Apply changes**.
3. **Sign out and back in** (the app offers to do this) so Windows rebuilds the Start layout.

The status card shows whether the setting is **Applied**, **Partially applied** or **Not applied**, with a line for each component. To go back, click **Restore default**.

## Command-line reference

Run from an **elevated** prompt. The installed executable is at
`%ProgramFiles%\Start Menu App List Only\StartMenuAppListOnly.exe`.

| Switch | Description |
|---|---|
| *(none)* | Open the graphical interface. |
| `/apply` | Apply the changes (also creates the startup task and restarts Explorer). |
| `/revert` | Restore the original values, remove the startup task and delete the app's data folder. |
| `/status` | Report state through the exit code only (no output). |
| `/quiet` | Suppress message boxes (use for scripts). |
| `/norestart` | Do not restart Explorer. Sign out/in or reboot instead. |
| `/reapply` | Used by the startup task: re-write the policies silently, without recreating the task or restarting Explorer. |

Switches may be written with `/` or `-` and are case-insensitive.

**Exit codes**

| Code | Meaning |
|---|---|
| `0` | Success (for `/status`: fully applied) |
| `1` | Error |
| `3` | `/status` only: not (fully) applied |

**Examples**

```bat
:: Apply silently, restart Explorer later
StartMenuAppListOnly.exe /apply /quiet /norestart

:: Check state in a script
StartMenuAppListOnly.exe /status
if %errorlevel%==0 (echo applied) else (echo not applied)

:: Undo everything
StartMenuAppListOnly.exe /revert /quiet
```

## What it changes

Everything is under `HKEY_LOCAL_MACHINE`, so it applies to all users.

| Location | Value | Purpose |
|---|---|---|
| `SOFTWARE\Policies\Microsoft\Windows\Explorer` | `LockedStartLayout` = `1` | Locks the Start layout |
| `SOFTWARE\Policies\Microsoft\Windows\Explorer` | `StartLayoutFile` = path to an empty layout XML | Layout to enforce (no tiles) |
| `SOFTWARE\Policies\Microsoft\Windows\CloudContent` | `DisableWindowsConsumerFeatures` = `1` | Stops promoted/suggested tiles |
| `SOFTWARE\Microsoft\Windows\CurrentVersion\Shell Extensions\Blocked` | `{470C0EBD-5D73-4d58-9CED-E91E22E23282}` | Blocks the *Pin to Start* shell handler (64-bit and 32-bit views) |

Other artifacts:

- **Scheduled task** `StartMenuAppListOnly-Enforce`: runs as `SYSTEM` at boot and logon, calling `/apply /reapply /quiet`.
- **Data folder** `%ProgramData%\StartMenuAppListOnly`: the empty layout XML and a backup of the original registry values. Deleted on revert.

Nothing else is modified, and no files are injected into system processes.

## Uninstalling and reverting

- **Revert only:** open the app and click **Restore default** (or run `/revert`).
- **Uninstall:** *Settings > Apps > Start Menu App List Only > Uninstall*. The uninstaller **reverts the Start menu changes first**, then removes the program. Silent uninstall:
  ```bat
  "%ProgramFiles%\Start Menu App List Only\Uninstall.exe" /S
  ```

**Manual removal** (if the app is unavailable): delete the values listed in [What it changes](#what-it-changes) (or restore your own backup), then run
`schtasks /Delete /TN "StartMenuAppListOnly-Enforce" /F` and sign out and back in.

## Testing and compatibility

The Start menu lock relies on Windows policy behavior that Microsoft documents mainly for managed editions, and Start menu internals change between Windows 10 builds. **Results can differ by edition and build, so please test before wide use**, ideally in a virtual machine first.

<!-- TODO (maintainer): record the edition/build you tested on, e.g. "Windows 10 Pro 22H2 (19045.x): works". -->

| Edition | Expected | Status |
|---|---|---|
| Pro | Layout lock supported via policy | Needs verification per build |
| Enterprise / Education | Layout lock officially supported | Needs verification per build |
| Home | Policy may be **ignored**, so tiles may remain | Untested |
| Windows 11 | Not supported (different Start menu) | Not applicable |

Builds worth covering: 1809 (17763), 1909 (18363), 2004/20H2/21H1/21H2 (19041-19044) and 22H2 (19045).

**Tester checklist**

1. Take a restore point or snapshot the VM.
2. Install, click **Apply changes**, sign out and in. Start shows the apps list with no tiles.
3. Right-click an `.exe`, a folder and an app in the Start list: **Pin to Start** is absent.
4. Reboot: the setting is still applied (status card shows **Applied**).
5. *(Optional)* Install a feature update and confirm it persists or is re-applied by the startup task.
6. Click **Restore default**, sign out and in: Start behaves like stock Windows and the status card shows **Not applied**.
7. Uninstall: no `StartMenuAppListOnly` task, data folder or policy values remain.

Please report your Windows edition, build number (`winver`) and results in an [issue](../../issues).

## Known limitations

- **Right-click "More" submenu:** the Start menu draws its own app context menu, and Windows offers no supported setting to rearrange it. With *Pin to Start* gone, the menu can show only a **More** entry (Pin to taskbar, Run as administrator, Open file location, App settings, etc., depending on the app). This tool does not change that submenu.
- **Previous tile layout is not restored.** The locked empty layout replaces it. After reverting, you can re-pin tiles manually.
- **Empty strip in Start:** on some builds the Start menu may still reserve a narrow area where tiles used to be.
- **Windows Home:** the layout-lock policy may be ignored.
- **Taskbar pinning is untouched.** Only Start pinning is blocked.

## Unknown Publisher / SmartScreen

The installer and executable are **not digitally signed** (code signing requires a paid certificate or a signing service). Windows therefore shows:

- **SmartScreen:** *"Windows protected your PC"*, with the publisher shown as **Unknown publisher**.
- **UAC prompt:** *"Do you want to allow this app from an unknown publisher…?"*

The developer name (Tasmon Islam) and email are embedded in the file properties (*Details* tab), the installer and the application manifest. **This identifies the author, but it does not remove the warning.** The warning depends on a valid Authenticode signature from a trusted certificate authority, and metadata cannot substitute for one. Signing can be added once a certificate or signing service is available.

If you decide to proceed:

1. Download only from this repository's Releases page.
2. Check the file's SHA-256 against the value published in the release notes.
3. In SmartScreen choose **More info > Run anyway**. If the file was flagged as downloaded from the internet, you can also open *Properties* and tick **Unblock**.

The full source is included so you can review it or build the binaries yourself, see below. If you are not comfortable running an unsigned program, build it from source.

## Building from source

Sources: `Program.cs` (application), `installer.nsi` (NSIS installer), `app.manifest`, `make_assets.py` (generates the icon and `app.res` with manifest, icon and version info).

**Windows**: requires the built-in .NET Framework compiler and [NSIS](https://nsis.sourceforge.io/):

```bat
build.bat
```

**Linux / macOS**: requires `mono-mcs`, `mono-devel` (4.8 reference assemblies), `nsis`, and Python 3 with Pillow (only if regenerating assets):

```bash
./build.sh
```

Output: `StartMenuAppListOnly-Setup-1.1.0.exe`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Tiles still show after Apply | Sign out and back in (or reboot). On Windows 10 Home the policy may be ignored. |
| Status says **Partially applied** | Click **Apply changes** again to complete the missing parts. |
| *Pin to Start* still in a context menu | Restart Explorer (`taskkill /f /im explorer.exe` then `explorer.exe`) or sign out and in. |
| Startup task missing | Click **Apply changes** to recreate it. |
| Antivirus or SmartScreen blocks the installer | Expected for unsigned software, see [above](#unknown-publisher--smartscreen). |

## Disclaimer

This software modifies system policy in the registry and registers a scheduled task that runs as `SYSTEM`. It is provided **"as is", without warranty of any kind**. Use at your own risk and test on a non-critical machine first. The author is not liable for any damage or data loss. Start menu behavior can change with Windows updates, which may require a new release.

## License

See the [`LICENSE`](LICENSE) file.

---

## Developer

**Tasmon Islam**
Email: [tasmon@outlook.com](mailto:tasmon@outlook.com)

Bug reports and test results are welcome via [Issues](../../issues).
