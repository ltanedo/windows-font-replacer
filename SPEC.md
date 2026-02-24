# SPEC: Download and Install Microsoft Store Fonts on Windows

## Goal

Download font packages from the Microsoft Store locally and optionally set them as the Windows system font by replacing Segoe UI via registry edits.

## Prerequisites

- Windows 10 (build 17040+) or Windows 11
- `winget` (Windows Package Manager) installed — ships with modern Windows by default
- Admin privileges for installing fonts system-wide or applying registry changes

## Step 1: Download a Font from the Microsoft Store

Use `winget` with the Microsoft Store product ID. The product ID is the alphanumeric code in the Store URL.

**Example:** For Verdana Pro at `https://apps.microsoft.com/detail/9n8d67vhhdc2`, the product ID is `9N8D67VHHDC2`.

```
winget install --id <PRODUCT_ID> --source msstore --accept-source-agreements --accept-package-agreements
```

Concrete example:
```
winget install --id 9N8D67VHHDC2 --source msstore --accept-source-agreements --accept-package-agreements
```

## Step 2: Locate the Installed Font Files

After install, find the package install path:

```powershell
Get-AppxPackage -Name '*<PackageName>*' | Select-Object InstallLocation
```

For Verdana Pro:
```powershell
Get-AppxPackage -Name '*VerdanaPro*' | Select-Object InstallLocation
```

Font files (.ttf) are located inside the `InstallLocation` path, which is typically:
```
C:\Program Files\WindowsApps\<PackageFullName>\
```

## Step 3: Copy Font Files Locally

The `WindowsApps` folder is access-restricted. Copy the `.ttf` files to a local directory:

```powershell
$pkg = Get-AppxPackage -Name '*VerdanaPro*'
Copy-Item "$($pkg.InstallLocation)\*.ttf" "C:\Users\<username>\projects\win-custom-font\verdana-pro\" -Force
```

## Step 4 (Optional): Set as Windows System Font

Windows uses Segoe UI as the default system font. To replace it with a custom font, use a `.reg` file that:

1. Blanks out the Segoe UI font registrations
2. Sets a font substitution to redirect Segoe UI to the desired font

### Registry file template (`set-custom-font.reg`):

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts]
"Segoe UI (TrueType)"=""
"Segoe UI Bold (TrueType)"=""
"Segoe UI Bold Italic (TrueType)"=""
"Segoe UI Italic (TrueType)"=""
"Segoe UI Light (TrueType)"=""
"Segoe UI Semibold (TrueType)"=""
; "Segoe UI Symbol (TrueType)"=""

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\FontSubstitutes]
"Segoe UI"="<Font Name Here>"
```

Replace `<Font Name Here>` with the font's registered name (e.g., `"Verdana Pro"` or `"SF Pro Regular"`).

### To revert back to default Segoe UI (`set-win-font.reg`):

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts]
"Segoe UI (TrueType)"=""
"Segoe UI Bold (TrueType)"=""
"Segoe UI Bold Italic (TrueType)"=""
"Segoe UI Italic (TrueType)"=""
"Segoe UI Light (TrueType)"=""
"Segoe UI Semibold (TrueType)"=""
"Segoe UI Symbol (TrueType)"=""
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\FontSubstitutes]
"Segoe UI"="Segoe UI"
```

### Apply a registry file:

```
reg import set-custom-font.reg
```

Or double-click the `.reg` file and confirm the prompt. A **sign-out or restart** is required for changes to take effect.

## Project File Structure

```
win-custom-font/
  SPEC.md                 # This file
  set-mac-font.reg        # Replaces Segoe UI with SF Pro Regular
  set-win-font.reg        # Reverts system font back to Segoe UI
  SF-Pro.ttf              # SF Pro font file (for macOS-style look)
  verdana-pro/            # Downloaded Verdana Pro font files
    VerdanaPro-Regular.ttf
    VerdanaPro-Bold.ttf
    VerdanaPro-Italic.ttf
    VerdanaPro-Light.ttf
    VerdanaPro-SemiBold.ttf
    VerdanaPro-Black.ttf
    VerdanaPro-CondRegular.ttf
    ... (20 TTF files total, covering all weights + condensed variants)
```

## Notes

- The `winget --source msstore` approach works for **free** Store packages. Paid packages require authentication.
- `Segoe UI Symbol` is commented out in `set-mac-font.reg` to preserve icon/symbol rendering. Uncomment it to fully blank all Segoe UI variants.
- The font must be installed on the system (either via the Store or by right-click > Install on the .ttf) before it can be used as a substitute.
- Font substitution only affects UI elements that explicitly request Segoe UI. Some apps may use hardcoded fonts.
