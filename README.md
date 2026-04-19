

```powershell
"jsut read & follow the documentation. That's it."
```
# 🔄 IDM Trial Reset Automation Guide 

Welcome to the professional guide for automating your IDM trial reset. This guide explains how to use PowerShell's built-in profile system to create a permanent, reusable command.

## 📖 What is `$PROFILE`?

If you are coming from a Linux, Node.js, or MERN stack background, you are likely familiar with files like `.bashrc`, `.zshrc`, or `.env`. 

In Windows, `$PROFILE` is the exact same concept for PowerShell. It is a hidden configuration script that runs automatically every time you open a new PowerShell window. By saving our custom `Reset-IDM` function inside this file, Windows will load it into memory globally, allowing you to use it just like a native command (like `cd`, `ls`, or `npm`).

---

## 🚀 Step-by-Step Setup

### Step 1: Allow PowerShell to Run Scripts
By default, Windows blocks custom scripts for security. We need to allow your local profile to run.
1. Open PowerShell **as Administrator**.
2. Run the following command:
   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```
3. Type `Y` and press `Enter` if prompted.

### Step 2: Create Your PowerShell Profile
1. In the same terminal, check if a profile file already exists:
   ```powershell
   Test-Path $PROFILE
   ```
2. If it returns `False`, create it by running:
   ```powershell
   New-Item -Type File -Path $PROFILE -Force
   ```

### Step 3: Add the Automation Script
1. Open the profile file in Notepad by typing:
   ```powershell
   notepad $PROFILE
   ```
2. Copy the entire code block below and paste it into the Notepad window:

```powershell
function Reset-IDM {
    <#
    .SYNOPSIS
        Professional IDM trial reset automation.
    #>
    [CmdletBinding()]
    process {
        Write-Host "--- Initiating IDM Professional Reset ---" -ForegroundColor Cyan

        # 1. Kill all IDM related processes
        $Processes = @("IDMan", "IEMonitor", "IDMGrHlp")
        foreach ($p in $Processes) {
            if (Get-Process -Name $p -ErrorAction SilentlyContinue) {
                Stop-Process -Name $p -Force -ErrorAction SilentlyContinue
                Write-Host "[✔] Terminated $p" -ForegroundColor Green
            }
        }

        # 2. Scrub Registry Anchors
        $RegTargets = @(
            "HKCU:\Software\DownloadManager",
            "HKCU:\Software\Classes\Wow6432Node\CLSID\{07999AC3-058B-40BF-984F-69EB1E554CA7}"
        )

        foreach ($Path in $RegTargets) {
            if (Test-Path $Path) {
                if ($Path -match "DownloadManager") {
                    $Values = @("Serial", "Email", "FName", "LName", "LstCheck", "Trial")
                    foreach ($V in $Values) {
                        Remove-ItemProperty -Path $Path -Name $V -ErrorAction SilentlyContinue
                    }
                    Write-Host "[✔] Scrubbed DownloadManager licensing" -ForegroundColor Green
                } else {
                    Remove-Item -Path $Path -Recurse -Force -ErrorAction SilentlyContinue
                    Write-Host "[✔] Nuked CLSID Anchor: $Path" -ForegroundColor Green
                }
            }
        }

        # 3. Wipe local license tracking
        $IDMAppData = "$env:APPDATA\IDM"
        if (Test-Path $IDMAppData) {
            Get-ChildItem -Path $IDMAppData -Filter "*.dat" | Remove-Item -Force
            Write-Host "[✔] Local AppData cache cleared" -ForegroundColor Green
        }

        Write-Host "`n[SUCCESS] IDM reset. You can now launch IDM for a fresh trial." -ForegroundColor Green
    }
}
```
3. **Save** the file (`Ctrl + S`) and close Notepad.

### Step 4: Reload and Run
1. Tell PowerShell to reload your updated profile:
   ```powershell
   . $PROFILE
   ```
2. **You're done!** To reset IDM right now (and anytime in the future), just type:
   ```powershell
   Reset-IDM
   ```

---

## ⚡ Bonus: Create a 1-Click Desktop Shortcut
If you prefer not to open the terminal at all, you can make a clickable button on your desktop:

1. Right-click your Desktop -> **New** -> **Shortcut**.
2. Paste this exact line into the location box:
   ```cmd
   powershell.exe -WindowStyle Hidden -Command ". $PROFILE; Reset-IDM"
   ```
3. Name it `Reset IDM` and save.
4. Now, you can just double-click that icon whenever the trial expires!
