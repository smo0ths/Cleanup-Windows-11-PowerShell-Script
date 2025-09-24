# Cleanup Windows 11 PowerShell Script v0.0.2
## what to do:
##### copy/pasta in PowerShell or click run cleanup.bat in zip

```python
$shell=New-Object -ComObject 'Shell.Application'
function Clear-Items{
  param([string]$Path)
  if(-not (Test-Path $Path)){return}
  Write-Host "Scanning: $Path"
  Get-ChildItem $Path -File -Recurse -Force -ErrorAction SilentlyContinue|
    ForEach-Object{
      $full=$_.FullName
      $folder=Split-Path $full -Parent
      $leaf=Split-Path $full -Leaf
      $item=$shell.Namespace($folder).ParseName($leaf)
      if($item){
        try{$item.InvokeVerb('delete');$script:TotalFiles++;$script:TotalSize+=$_.Length}
        catch{Write-Warning "Skipped: `"$full`"`n    → $($_.Exception.Message)"}
      }else{Write-Warning "Couldn’t shell-parse: $full"}
    }
}
[int]$TotalFiles=0
[int64]$TotalSize=0
$patterns=@(
"$env:LOCALAPPDATA\Crashdumps",
"$env:LOCALAPPDATA\LocalLow\Microsoft\CryptnetUrlCache\Content",
"$env:LOCALAPPDATA\Low\Temp",
"$env:LOCALAPPDATA\Microsoft\Windows\INetCache\IE",
"$env:LOCALAPPDATA\Microsoft\Windows\WebCache",
"$env:LOCALAPPDATA\Packages\Microsoft.WindowsStore_*\AC\Temp",
"$env:LOCALAPPDATA\Packages\Microsoft.WindowsStore_*\LocalCache",
"$env:LOCALAPPDATA\Roaming\discord\Code Cache\js",
"$env:LOCALAPPDATA\Temp",
"$env:ProgramData\Microsoft\Windows Defender\Scans\History\Store",
"$env:ProgramData\Microsoft\Windows\Caches",
"$env:ProgramData\Microsoft\Windows\DeliveryOptimization\Logs",
"$env:ProgramData\Microsoft\Windows\WER\ReportArchive",
"$env:ProgramData\Microsoft\Windows\WER\ReportQueue",
"$env:ProgramData\Microsoft\Windows\WER\Temp",
"$env:ProgramData\Usoshared\Logs",
"$env:SystemDrive\PerfLogs",
"$env:SystemRoot\LiveKernelReports",
"$env:SystemRoot\Logs\CBS",
"$env:SystemRoot\Logs\DISM",
"$env:SystemRoot\Minidump",
"$env:SystemRoot\Panther\*",
"$env:SystemRoot\Prefetch",
"$env:SystemRoot\ServiceProfiles\LocalService\AppData\Local\Crashdumps",
"$env:SystemRoot\SoftwareDistribution\DeliveryOptimization\Cache",
"$env:SystemRoot\SoftwareDistribution\Download",
"$env:SystemRoot\SoftwareDistribution\PostRebootEventCache",
"$env:SystemRoot\System32\Sru",
"$env:SystemRoot\Temp",
"$env:SystemRoot\WinSxS\Temp\Inflight",
"$env:TEMP",
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
