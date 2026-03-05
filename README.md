
# **Automatically Open All Your Morning Chrome Tabs with PowerShell**

Are you bored or is it time-consuming to open all those tabs sometimes more than 10 or 20 browser tabs when you start your morning at work? 

If that is the scenario, here is a PowerShell script which you can just run, and it will **automatically open all your required browser tabs**. 

This saves time. If we have the technology or trick, why not give it a try?

I have provided a **step-by-step easy guide** and **full PowerShell script**, ready to run. 

Give it a try and feel free to recommend improvements!

---

## **Step 1: Prepare URLs**

1. Create a folder for this automation, e.g., `C:\Automation\MorningWorkspace`.
2. Inside the folder, create a file called `urls.txt`.
3. Add all your required URLs (websites), one per line in "urls.txt" file. Lines starting with `#` are ignored.

Example:

```powershell
https://outlook.office.com
https://teams.microsoft.com
https://servicenow.company.com
```


---

## **Step 2: Save the Script**

1. Create a `MorningChromeLauncher.ps1` PowerShell file **in the same folder** as `urls.txt` and copy the following script into the PowerShell file.

```powershell
# MorningChromeLauncher.ps1
# Opens all URLs from urls.txt in Chrome automatically

$UrlFile = Join-Path $PSScriptRoot "urls.txt"
$ChromeRunningOption = "A"
$DelayIfChromeRunning = 0.3
$DelayIfChromeClosed = 0

function Get-ChromePath {
    $paths = @("$Env:ProgramFiles\Google\Chrome\Application\chrome.exe", "$Env:ProgramFiles(x86)\Google\Chrome\Application\chrome.exe")
    foreach ($path in $paths) { if (Test-Path $path) { return $path } }
    return $null
}

function Get-ChromeProfiles {
    $UserDataDir = "$Env:LOCALAPPDATA\Google\Chrome\User Data"
    if (!(Test-Path $UserDataDir)) { return @("Default") }
    $profiles = Get-ChildItem -Path $UserDataDir -Directory | Where-Object { $_.Name -match "Default|Profile" } | Select-Object -ExpandProperty Name
    if ($profiles.Count -eq 0) { return @("Default") }
    return $profiles
}

function Select-ChromeProfile {
    $profiles = Get-ChromeProfiles
    Write-Host "Available Chrome Profiles:"
    for ($i=0; $i -lt $profiles.Count; $i++) { Write-Host "$($i+1) - $($profiles[$i])" }
    $choice = Read-Host "Enter choice (number)"
    if ($choice -match '^\d+$' -and $choice -ge 1 -and $choice -le $profiles.Count) { return $profiles[$choice-1] }
    else { Write-Host "Invalid choice. Using Default profile."; return "Default" }
}

function Get-Urls {
    if (!(Test-Path $UrlFile)) { Write-Host "URL file not found: $UrlFile"; exit }
    return Get-Content $UrlFile | Where-Object { $_ -and -not $_.StartsWith("#") }
}

Write-Host "Morning Chrome Workspace Launcher"
$ChromePath = Get-ChromePath
if (!$ChromePath) { Write-Host "Google Chrome not found."; exit }
$Profile = Select-ChromeProfile
$Urls = Get-Urls
if ($Urls.Count -eq 0) { Write-Host "No URLs found."; exit }
$ChromeRunning = Get-Process chrome -ErrorAction SilentlyContinue
if ($ChromeRunning) {
    Write-Host "Chrome already running."
    if ($ChromeRunningOption -eq "B") { Write-Host "Option B: No new tabs."; exit }
    if ($ChromeRunningOption -eq "C") { Start-Process $ChromePath "--new-window --profile-directory=$Profile" }
    $Delay = $DelayIfChromeRunning
} else {
    Start-Process $ChromePath "--profile-directory=$Profile"
    Start-Sleep 2
    $Delay = $DelayIfChromeClosed
}
foreach ($url in $Urls) { Start-Process $ChromePath "--profile-directory=$Profile $url"; if ($Delay -gt 0) { Start-Sleep -Seconds $Delay } }
Write-Host "All tabs launched."
```

---

## **Step 3: Allow Scripts to Run**

Open PowerShell. If PowerShell’s execution policy is preventing scripts from running for security reasons, fix it safely with run **once** in PowerShell:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Press `Y` to confirm.

<img width="974" height="282" alt="image" src="https://github.com/user-attachments/assets/0a764d8e-e9bd-43fa-aacf-e726e604ea66" />

In my case, I am using my office PC, igonore my file path. 

---

## **Step 4: Run the Script**

```powershell
cd C:\Automation\MorningWorkspace
.\MorningChromeLauncher.ps1
```
1. Choose the Chrome profile from the list.
2. Chrome will open and load all your URLs automatically.


<img width="977" height="386" alt="image" src="https://github.com/user-attachments/assets/05b75295-28cb-46cd-83f7-2c76875c9b0b" />

The script checks both Program Files and Program Files (x86) paths and exits if Chrome is not installed.

The script reads Chrome’s User Data folder and lists all profiles automatically, falls back to Default if no profiles are found.

The script reads urls.txt and ignores empty lines and lines starting with #.

The tab delay is automatic based on Chrome running or not on the computer. 


<img width="967" height="669" alt="image" src="https://github.com/user-attachments/assets/26b3c787-aeee-4392-8d3e-dd5c29042a7b" />

---

## **Step 5: Desktop Shortcut**
To make it more easier to run, create a desktop shortcut. 

1. Right-click desktop → **New → Shortcut**
2. Location:

```
powershell.exe -ExecutionPolicy Bypass -File "C:\Automation\MorningWorkspace\MorningChromeLauncher.ps1"
```

3. Name it `Morning Workspace`.
4. Double-click it every morning, all tabs open automatically.

<img width="754" height="456" alt="image" src="https://github.com/user-attachments/assets/cadba2fa-9929-4989-849a-94af7db0c5a8" />

<img width="835" height="347" alt="image" src="https://github.com/user-attachments/assets/fdeb03ac-0486-4296-be04-621af9ac7185" />

---

## **Step 6: Update URLs**

Edit `urls.txt` to add or remove URLs. Next time you run the script, the changes are applied.

---

