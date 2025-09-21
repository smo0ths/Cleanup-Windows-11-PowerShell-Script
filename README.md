# Cleanup Windows 11 PowerShell Script v0.0.0
## what to do:
##### copy/pasta in PowerShell or click run cleanup.bat in zip

```python
# Cleanup Windows 11 PowerShell Script (copy/pasta in PowerShell)
Add-Type -AssemblyName Microsoft.VisualBasic
$UI=[Microsoft.VisualBasic.FileIO.UIOption]::OnlyErrorDialogs
$RB=[Microsoft.VisualBasic.FileIO.RecycleOption]::SendToRecycleBin
$JunkExtensions='*.log','*.dmp','*.tmp','*.old','*.bak','*.etl'
$Paths=@(
    "$env:APPDATA\Microsoft\Windows\Recent",
    "$env:APPDATA\Mozilla\Firefox\Profiles\*\cache2",
    "$env:APPDATA\Mozilla\Firefox\Profiles\*\jumpListCache",
    "$env:APPDATA\Mozilla\Firefox\Profiles\*\offlineCache",
    "$env:LOCALAPPDATA\Crashdumps","$env:LOCALAPPDATA\Google\Chrome\User Data\*\Cache",
    "$env:LOCALAPPDATA\Google\Chrome\User Data\*\Code Cache",
    "$env:LOCALAPPDATA\Google\Chrome\User Data\*\GPUCache",
    "$env:LOCALAPPDATA\Google\Chrome\User Data\*\Media Cache",
    "$env:LOCALAPPDATA\LocalLow\Microsoft\CryptnetUrlCache\Content",
    "$env:LOCALAPPDATA\Microsoft\Edge\User Data\*\Cache",
    "$env:LOCALAPPDATA\Microsoft\Edge\User Data\*\Code Cache",
    "$env:LOCALAPPDATA\Microsoft\Edge\User Data\*\GPUCache",
    "$env:LOCALAPPDATA\Microsoft\Edge\User Data\*\Media Cache",
    "$env:LOCALAPPDATA\Microsoft\Windows\Explorer",
    "$env:LOCALAPPDATA\Microsoft\Windows\INetCache\IE",
    "$env:LOCALAPPDATA\Microsoft\Windows\WebCache",
    "$env:LOCALAPPDATA\Roaming\discord\Code Cache\js",
    "$env:LOCALAPPDATA\Temp","$env:LOCALAPPDATA\D3DSCache",
    "$env:LOCALAPPDATA\Packages\Microsoft.WindowsStore_*\LocalCache",
    "$env:LOCALAPPDATA\Packages\Microsoft.WindowsStore_*\AC\Temp",
    "$env:LOCALAPPDATA\Low\Temp",
    "$env:ProgramData\Microsoft\Windows Defender\Scans\History\Store",
    "$env:ProgramData\Microsoft\Windows\Caches",
    "$env:ProgramData\Microsoft\Windows\WER\ReportArchive",
    "$env:ProgramData\Microsoft\Windows\WER\ReportQueue",
    "$env:ProgramData\Microsoft\Windows\WER\Temp",
    "$env:ProgramData\Microsoft\Windows\DeliveryOptimization\Logs",
    "$env:ProgramData\Usoshared\Logs","$env:SystemDrive\PerfLogs",
    "$env:SystemRoot\LiveKernelReports","$env:SystemRoot\Logs",
    "$env:SystemRoot\Logs\CBS","$env:SystemRoot\Logs\DISM",
    "$env:SystemRoot\Minidump","$env:SystemRoot\Panther",
    "$env:SystemRoot\Panther\UnattendGC",
    "$env:SystemRoot\Panther\SetupRollback",
    "$env:SystemRoot\Prefetch",
    "$env:SystemRoot\ServiceProfiles\LocalService\AppData\Local\Crashdumps",
    "$env:SystemRoot\SoftwareDistribution\DeliveryOptimization\Cache",
    "$env:SystemRoot\SoftwareDistribution\Download",
    "$env:SystemRoot\SoftwareDistribution\PostRebootEventCache",
    "$env:SystemRoot\System32\Sru","$env:SystemRoot\Temp",
    "$env:SystemRoot\WinSxS\Temp\Inflight",
    "$env:TEMP",
    "$env:USERPROFILE\Downloads"
)
[int]$TotalFiles=0; [int64]$TotalSize=0
foreach($Path in $Paths){
  if(!(Test-Path $Path)){continue}
  Get-ChildItem -Path $Path -Recurse -File -Force -ErrorAction SilentlyContinue |
  ForEach-Object{
    $TotalFiles++; $TotalSize+=$_.Length
    try{[Microsoft.VisualBasic.FileIO.FileSystem]::DeleteFile($_.FullName,$UI,$RB)}catch{}
  }
}
$WindowsOld="$env:SystemDrive\Windows.old"
if(Test-Path $WindowsOld){
  if((Read-Host "Windows.old found, send to Recycle Bin? (Y/N)") -match '^[Yy]$'){
    try{
      $oldFiles=Get-ChildItem -Path $WindowsOld -Recurse -File -Force -ErrorAction SilentlyContinue
      $TotalFiles+=$oldFiles.Count
      $TotalSize+=($oldFiles|Measure-Object -Property Length -Sum).Sum
      [Microsoft.VisualBasic.FileIO.FileSystem]::DeleteDirectory($WindowsOld,$UI,$RB)
      Write-Host "Windows.old sent to Recycle Bin."
    }catch{Write-Warning "Could not move Windows.old to Recycle Bin"}
  }else{Write-Host "Skipped Windows.old."}
}
Write-Host "Total files sent to Recycle Bin: $TotalFiles"
Write-Host ("Total size: {0:N2} MB" -f ($TotalSize/1MB))
```
