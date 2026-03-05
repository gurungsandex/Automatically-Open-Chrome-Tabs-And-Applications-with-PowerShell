
I prefer incognito mode. To run the browser in **incognito mode**, use this script instead. 



```powershell
<#
.SYNOPSIS
Morning Chrome Workspace Launcher (Incognito)

.DESCRIPTION
Opens Google Chrome in incognito mode and loads multiple URLs from an external configuration file.

FEATURES
- Detects Chrome installation
- Automatically lists all Chrome profiles for selection
- Supports behavior if Chrome is already running (A/B/C)
- Reads URLs from external config file
- Adjustable tab delay
- Compatible with Windows 10 / 11 and PowerShell 5.1 / 7
#>

# -------------------------------
# CONFIGURATION
# -------------------------------
$UrlFile = Join-Path $PSScriptRoot "urls.txt"

# Chrome behavior if already running: "A"=open in same window, "B"=do nothing, "C"=open new window
$ChromeRunningOption = "A"

# Delay between tabs
$DelayIfChromeRunning = 0.3
$DelayIfChromeClosed = 0

# -------------------------------
# FUNCTION: Detect Chrome Path
# -------------------------------
function Get-ChromePath {
    $paths = @(
        "$Env:ProgramFiles\Google\Chrome\Application\chrome.exe",
        "$Env:ProgramFiles(x86)\Google\Chrome\Application\chrome.exe"
    )
    foreach ($path in $paths) { if (Test-Path $path) { return $path } }
    return $null
}

# -------------------------------
# FUNCTION: Detect Chrome Profiles
# -------------------------------
function Get-ChromeProfiles {
    $UserDataDir = "$Env:LOCALAPPDATA\Google\Chrome\User Data"
    if (!(Test-Path $UserDataDir)) { return @("Default") }

    $profiles = Get-ChildItem -Path $UserDataDir -Directory | Where-Object {
        $_.Name -match "Default|Profile"
    } | Select-Object -ExpandProperty Name

    if ($profiles.Count -eq 0) { return @("Default") }

    return $profiles
}

# -------------------------------
# FUNCTION: Choose Chrome Profile
# -------------------------------
function Select-ChromeProfile {
    $profiles = Get-ChromeProfiles

    Write-Host ""
    Write-Host "Available Chrome Profiles:"
    for ($i=0; $i -lt $profiles.Count; $i++) {
        Write-Host "$($i+1) - $($profiles[$i])"
    }

    $choice = Read-Host "Enter choice (number)"
    if ($choice -match '^\d+$' -and $choice -ge 1 -and $choice -le $profiles.Count) {
        return $profiles[$choice-1]
    }
    else {
        Write-Host "Invalid choice. Using Default profile."
        return "Default"
    }
}

# -------------------------------
# FUNCTION: Load URLs
# -------------------------------
function Get-Urls {
    if (!(Test-Path $UrlFile)) {
        Write-Host "URL configuration file not found: $UrlFile"
        exit
    }

    $urls = Get-Content $UrlFile | Where-Object { $_ -and -not $_.StartsWith("#") }
    return $urls
}

# -------------------------------
# MAIN SCRIPT
# -------------------------------
Write-Host ""
Write-Host "Morning Chrome Workspace Launcher (Incognito)"
Write-Host "---------------------------------------------"

# Detect Chrome
$ChromePath = Get-ChromePath
if (!$ChromePath) {
    Write-Host "Google Chrome is not installed."
    exit
}

# Select Profile
$Profile = Select-ChromeProfile

# Load URLs
$Urls = Get-Urls
if ($Urls.Count -eq 0) {
    Write-Host "No URLs found in configuration file."
    exit
}

# Check if Chrome is running
$ChromeRunning = Get-Process chrome -ErrorAction SilentlyContinue
if ($ChromeRunning) {
    Write-Host "Chrome is already running."

    if ($ChromeRunningOption -eq "B") {
        Write-Host "Option B selected: No new tabs will open."
        exit
    }

    if ($ChromeRunningOption -eq "C") {
        Write-Host "Opening new Chrome window in incognito."
        Start-Process $ChromePath "--new-window --incognito --profile-directory=$Profile"
    }

    $Delay = $DelayIfChromeRunning
}
else {
    Write-Host "Launching Chrome in incognito."
    Start-Process $ChromePath "--incognito --profile-directory=$Profile"
    Start-Sleep 2
    $Delay = $DelayIfChromeClosed
}

# Open URLs in tabs in incognito
foreach ($url in $Urls) {
    Start-Process $ChromePath "--incognito --profile-directory=$Profile $url"
    if ($Delay -gt 0) { Start-Sleep -Seconds $Delay }
}

Write-Host ""
Write-Host "All workspace tabs launched in incognito mode."
```

---
