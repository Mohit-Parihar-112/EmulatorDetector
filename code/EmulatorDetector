# ========================================
# Full Emulator Detection (Now.gg + BlueStacks + MSI)
# Digital Signature + Executable Path + Process Name
# ========================================

# --- Configuration ---
$TargetSignatures = @(
    "Now.gg, INC",
    "BlueStack Systems, Inc.",
    "Micro-Star International Co., Ltd."
)

$TargetProcessNames = @(
    "HD-Player.exe",
    "HD-Frontend.exe",
    "HD-Plus.exe",
    "Bluestacks.exe",
    "BstkSVC.exe",
    "BstkVMMgr.exe",
    "nowgg.exe"
)

$TargetPaths = @(
    "BlueStacks_nxt",
    "MSI\\App Player",
    "nowgg"
)

$LogPath = "$env:ProgramData\EmulatorDetection\log.txt"
$Webhook = "https://discord.com/api/webhooks/1392104830985568277/1UcN5o85gbYslTyvdzUxn10lhpgeRf3cmNlub6_cvuM2Ww1U1IpGTkAJmoIMjLWw9PBS"
$Detections = @()

# --- Ensure Log Directory Exists ---
$null = New-Item -ItemType Directory -Path (Split-Path $LogPath) -Force

# --- Get Digital Signature ---
function Get-SignatureSubject {
    param ([string]$filePath)
    try {
        $sig = Get-AuthenticodeSignature -FilePath $filePath
        if ($sig.Status -eq "Valid" -and $sig.SignerCertificate -ne $null) {
            return $sig.SignerCertificate.Subject
        }
    } catch {
        return $null
    }
    return $null
}

# --- Log & Webhook ---
function Log-Detection {
    param([string]$Message)
    $timestamp = (Get-Date).ToString("u")
    $logEntry = "$timestamp - $Message"
    Add-Content -Path $LogPath -Value $logEntry
    Write-Host "[!] $Message" -ForegroundColor Red

    try {
        $json = @{ content = "**[Emulator Detected]**`n$Message" } | ConvertTo-Json -Depth 3
        Invoke-RestMethod -Uri $Webhook -Method Post -Body $json -ContentType "application/json"
    } catch {
        Write-Warning "Webhook failed: $_"
    }
}

# --- Start Process Scan ---
Write-Host "`n[*] Scanning running processes for emulator indicators..." -ForegroundColor Cyan

$RunningProcesses = Get-Process | ForEach-Object {
    try {
        [PSCustomObject]@{
            Name = $_.Name
            Path = $_.Path
            Id   = $_.Id
        }
    } catch { $null }
} | Where-Object { $_.Path -and (Test-Path $_.Path) }

foreach ($proc in $RunningProcesses) {
    $exe = $proc.Path
    $name = $proc.Name
    $id = $proc.Id
    $subject = Get-SignatureSubject -filePath $exe

    # --- Signature Check ---
    if ($subject) {
        foreach ($target in $TargetSignatures) {
            if ($subject -like "*$target*") {
                $msg = "PID $id ($name) at '$exe' signed by '$subject'"
                $Detections += $msg
                Log-Detection $msg
            }
        }
    }

    # --- Path Match Check ---
    foreach ($pattern in $TargetPaths) {
        if ($exe -like "*$pattern*") {
            $msg = "PID $id ($name) is running from path: $exe (matched pattern: $pattern)"
            $Detections += $msg
            Log-Detection $msg
        }
    }

    # --- Name Match Check ---
    foreach ($pname in $TargetProcessNames) {
        if ($name -ieq [System.IO.Path]::GetFileNameWithoutExtension($pname)) {
            $msg = "PID $id running known emulator process: $name"
            $Detections += $msg
            Log-Detection $msg
        }
    }
}

# --- Final Output ---
if ($Detections.Count -eq 0) {
    Write-Host "`n[✓] No emulator indicators found." -ForegroundColor Green
} else {
    Write-Host "`n[!] Detection Summary:`n" -ForegroundColor Red
    $Detections | ForEach-Object { Write-Host $_ -ForegroundColor Yellow }
}

Write-Host "`n[✓] Scan complete. Log saved to: $LogPath`n" -ForegroundColor Cyan
