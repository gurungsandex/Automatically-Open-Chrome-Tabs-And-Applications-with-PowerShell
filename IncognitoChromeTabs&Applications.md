I enhanced my Morning Workspace Launcher to make mornings easier:
- Opens Chrome in Incognito mode.
- Detects if Chrome is already running and lets you close, skip, or cancel.
- Prompts you to select a Chrome profile before opening tabs.
- Opens all URLs in Incognito mode.
- Opens your applications.  

Faster, safer, and fully customizable — ready to start your workday in one click!

```
# ==========================================
# Morning Workspace Launcher
# ==========================================

Write-Host "Starting Morning Workspace Launcher..." -ForegroundColor Green

# -------------------------------------------------
# 1. Locate Chrome
# -------------------------------------------------
$chromePath = "C:\Program Files\Google\Chrome\Application\chrome.exe"
if (!(Test-Path $chromePath)) { $chromePath = "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" }
if (!(Test-Path $chromePath)) { Write-Host "Google Chrome not found." -ForegroundColor Red; exit }

# -------------------------------------------------
# 2. Load URLs
# -------------------------------------------------
$urlsFile = "$PSScriptRoot\urls.txt"
if (Test-Path $urlsFile) {
    $urls = Get-Content $urlsFile | Where-Object { $_ -and -not $_.StartsWith("#") }
} else {
    Write-Host "urls.txt not found." -ForegroundColor Yellow
    $urls = @()
}

# -------------------------------------------------
# 3. Get Chrome Profiles
# -------------------------------------------------
function Get-ChromeProfiles {
    $userDataDir = "$Env:LOCALAPPDATA\Google\Chrome\User Data"
    if (!(Test-Path $userDataDir)) { return @("Default") }

    $profiles = Get-ChildItem -Path $userDataDir -Directory | Where-Object { $_.Name -match "Default|Profile" } | Select-Object -ExpandProperty Name
    if ($profiles.Count -eq 0) { return @("Default") }
    return $profiles
}

function Select-ChromeProfile {
    $profiles = Get-ChromeProfiles
    Write-Host ""
    Write-Host "Available Chrome Profiles:"
    for ($i=0; $i -lt $profiles.Count; $i++) { Write-Host "$($i+1) - $($profiles[$i])" }
    $choice = Read-Host "Enter the number for the profile to use"
    if ($choice -match '^\d+$' -and $choice -ge 1 -and $choice -le $profiles.Count) {
        return $profiles[$choice-1]
    } else {
        Write-Host "Invalid choice. Using Default profile."
        return "Default"
    }
}

# -------------------------------------------------
# 4. Check if Chrome is running
# -------------------------------------------------
$chromeRunning = Get-Process chrome -ErrorAction SilentlyContinue
$openChromeTabs = $true

if ($chromeRunning) {
    Write-Host ""
    Write-Host "Chrome is already running." -ForegroundColor Yellow
    Write-Host "1 - Close Chrome and open workspace tabs"
    Write-Host "2 - Skip Chrome and open only applications"
    Write-Host "3 - Cancel script"
    $choice = Read-Host "Enter your choice"

    switch ($choice) {
        "1" {
            Write-Host "Closing Chrome..." -ForegroundColor Yellow
            Stop-Process -Name chrome -Force
            Start-Sleep 2
        }
        "2" {
            Write-Host "Skipping Chrome launch..."
            $openChromeTabs = $false
        }
        "3" {
            Write-Host "Script cancelled."
            exit
        }
        default {
            Write-Host "Invalid choice. Script cancelled."
            exit
        }
    }
}

# -------------------------------------------------
# 5. Select Chrome Profile if opening Chrome
# -------------------------------------------------
if ($openChromeTabs -and $urls.Count -gt 0) {
    $profile = Select-ChromeProfile
    Write-Host "Opening Chrome with profile: $profile ..." -ForegroundColor Green

    # Build argument list
    $arguments = "--incognito --profile-directory=$profile"
    foreach ($url in $urls) {
        $arguments += " $url"
    }

    Start-Process $chromePath -ArgumentList $arguments
}

# -------------------------------------------------
# 6. Open TeamViewer
# -------------------------------------------------
#$teamViewerPath = "C:\Program Files\TeamViewer\TeamViewer.exe"
#if (Test-Path $teamViewerPath) {
#    Write-Host "Opening TeamViewer..."
#    Start-Process $teamViewerPath
#} else {
#    Write-Host "TeamViewer not found." -ForegroundColor Yellow
#}

# -------------------------------------------------
# 7. Space for additional applications
# -------------------------------------------------
# $appPath = "C:\Path\To\Application.exe"
# if (Test-Path $appPath) {
#     Start-Process $appPath
# }

Write-Host ""
Write-Host "Workspace ready. Have a productive day!" -ForegroundColor Cyan
```
