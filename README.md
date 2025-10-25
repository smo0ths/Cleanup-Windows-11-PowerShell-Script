# Cleanup Windows 11 PowerShell Script v0.0.4
## what to do:
##### copy/pasta in PowerShell

```python
Add-Type -AssemblyName Microsoft.VisualBasic   # Load the .NET helper for Recycle Bin operations
function Clear-Items {
    param([string]$Path)
    if (-not (Test-Path $Path)) { return }
    Write-Host "Scanning: $Path"
    # Delete files → Recycle Bin, no prompts
    Get-ChildItem $Path -File -Recurse -Force -ErrorAction SilentlyContinue | ForEach-Object {
        try {
            [Microsoft.VisualBasic.FileIO.FileSystem]::DeleteFile(
                $_.FullName,
                'OnlyErrorDialogs',   # suppresses confirmation prompts
                'SendToRecycleBin'    # ensures Recycle Bin, not permanent delete
            )
            $script:TotalFiles++
            $script:TotalSize += $_.Length
        }
        catch {
            Write-Warning "Skipped: `"$($_.FullName)`"`n    → $($_.Exception.Message)"
        }
    }
}
[int]$TotalFiles=0
[int64]$TotalSize=0
$patterns=@(
"$env:LOCALAPPDATA\Crashdumps",
"$env:LOCALAPPDATA\LocalLow\Microsoft\CryptnetUrlCache\Content",
"$env:LOCALAPPDATA\Low\Temp",
"$env:LOCALAPPDATA\Microsoft\Windows\INetCache\IE",
"$env:LOCALAPPDATA\Packages\Microsoft.WindowsStore_*\AC\Temp",
"$env:LOCALAPPDATA\Packages\Microsoft.WindowsStore_*\LocalCache",
"$env:LOCALAPPDATA\Roaming\discord\Code Cache\js",
"$env:LOCALAPPDATA\Temp",
"$env:ProgramData\Microsoft\Windows Defender\Scans\History\Store",
"$env:ProgramData\Microsoft\Windows\DeliveryOptimization\Logs",
"$env:ProgramData\Microsoft\Windows\WER\ReportArchive",
"$env:ProgramData\Microsoft\Windows\WER\ReportQueue",
"$env:ProgramData\Microsoft\Windows\WER\Temp",
"$env:ProgramData\Usoshared\Logs",
"$env:SystemDrive\*.log",
"$env:SystemDrive\PerfLogs",
"$env:SystemRoot\Minidump",
"$env:SystemRoot\Panther\*",
"$env:SystemRoot\Prefetch",
"$env:SystemRoot\ServiceProfiles\LocalService\AppData\Local\Crashdumps",
"$env:SystemRoot\SoftwareDistribution\DeliveryOptimization\Cache",
"$env:SystemRoot\SoftwareDistribution\Download",
"$env:SystemRoot\SoftwareDistribution\PostRebootEventCache",
"$env:SystemRoot\System32\Sru",
"$env:SystemRoot\Temp",
"$env:TEMP",
"$env:USERPROFILE\.dotnet\*.dotnetUserLevelCache",
"$env:USERPROFILE\.dotnet\*MachineId*",
"$env:USERPROFILE\.dotnet\TelemetryStorageService",
"$env:USERPROFILE\Downloads"
)
$chromeUserData=Join-Path $env:LOCALAPPDATA 'Google\Chrome\User Data'
if(Test-Path $chromeUserData){
  Get-ChildItem $chromeUserData -Directory -ErrorAction SilentlyContinue|
    ForEach-Object{$pp=$_.FullName;$patterns+="$pp\Cache","$pp\Code Cache","$pp\GPUCache","$pp\Media Cache"}
}
$edgeUserData=Join-Path $env:LOCALAPPDATA 'Microsoft\Edge\User Data'
if(Test-Path $edgeUserData){
  Get-ChildItem $edgeUserData -Directory -ErrorAction SilentlyContinue|
    ForEach-Object{$pp=$_.FullName;$patterns+="$pp\Cache","$pp\Code Cache","$pp\GPUCache","$pp\Media Cache"}
}
$ffProfileRoots=@(Join-Path $env:APPDATA 'Mozilla\Firefox\Profiles';Join-Path $env:LOCALAPPDATA 'Mozilla\Firefox\Profiles')
foreach($root in $ffProfileRoots){if(Test-Path $root){
  Get-ChildItem $root -Directory -ErrorAction SilentlyContinue|
    ForEach-Object{$pp=$_.FullName;Write-Host "Found Firefox profile: $pp";$patterns+="$pp\cache2","$pp\jumpListCache","$pp\offlineCache"}
}}
$patterns|ForEach-Object{Resolve-Path $_ -ErrorAction SilentlyContinue}|Select-Object -ExpandProperty Path -Unique|ForEach-Object{Clear-Items $_}
$wo="$env:SystemDrive\Windows.old"
if(Test-Path $wo){Write-Host "Recycling Windows.old";Clear-Items $wo}
Write-Host "`nTotal files recycled (or was in use): $TotalFiles"
Write-Host ("Total space freed (or was in use): {0:N2} MB" -f ($TotalSize/1MB))
```
