# Windows Custom Font Replacer

Swap out the default Windows system font (Segoe UI) with any font you want using simple registry files. Includes presets for **SF Pro** (macOS look) and **Verdana Pro**.

## Quick Start

1. **Install your desired font** (right-click `.ttf` > Install, or use winget for MS Store fonts)
2. **Double-click a `.reg` file** to apply — confirm the UAC prompt
3. **Sign out and back in** to see the change

## Included Presets

| File | Effect |
|---|---|
| `set-mac-font.reg` | Replaces Segoe UI with **SF Pro Regular** |
| `set-verdana-font.reg` | Replaces Segoe UI with **Verdana Pro** |
| `set-win-font.reg` | Reverts back to **default Segoe UI** |

## Included Fonts

- **SF-Pro.ttf** — Apple's SF Pro font for a macOS-style look
- **verdana-pro/** — 20 Verdana Pro TTF files (all weights + condensed variants)

## Downloading Fonts from the Microsoft Store

You can grab free font packages from the MS Store using `winget`:

```
winget install --id <PRODUCT_ID> --source msstore
```

For example, Verdana Pro (`https://apps.microsoft.com/detail/9n8d67vhhdc2`):

```
winget install --id 9N8D67VHHDC2 --source msstore
```

See [SPEC.md](SPEC.md) for the full step-by-step process.

## Creating Your Own Preset

Use this template and replace the font name on the last line:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts]
"Segoe UI (TrueType)"=""
"Segoe UI Bold (TrueType)"=""
"Segoe UI Bold Italic (TrueType)"=""
"Segoe UI Italic (TrueType)"=""
"Segoe UI Light (TrueType)"=""
"Segoe UI Semibold (TrueType)"=""

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\FontSubstitutes]
"Segoe UI"="Your Font Name Here"
```

## Reverting

Double-click `set-win-font.reg` and sign out. Everything goes back to normal.

## Requirements

- Windows 10 or 11
- Admin privileges (for registry changes)

## Command‑line Usage

You can also perform the registry import from PowerShell with elevation. This is useful when scripting or if you prefer the terminal:

```powershell
# prompt for UAC and import the .reg file
Start-Process -FilePath reg.exe \
    -ArgumentList 'import',"\"C:\\Users\\ltanedo\\projects\\windows-font-replacer\\set-verdana-font.reg\"" \
    -Verb RunAs
```

After running the elevated import, verify the substitution has been written:

```powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\FontSubstitutes" /v "Segoe UI"
# should output:
# HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\FontSubstitutes
#     Segoe UI    REG_SZ    Verdana Pro
```

