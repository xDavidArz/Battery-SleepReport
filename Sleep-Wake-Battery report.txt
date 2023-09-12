
<#README README README









Ctrl + A then Paste into PowerShell Admin Window
Everything below this line is the script
#>







Write-Host "`n
`n
`n
`n
`n
`n
`n
`n
`n
`n
`n
`n"

<# Get Date variables for a week 
#>

$startDate = (Get-Date).AddDays(-7)

$endDate = Get-Date



Function computerPowerSourceChange{

Get-WinEvent -ComputerName $env:COMPUTERNAME -ProviderName 'Microsoft-Windows-Kernel-Power' |
Where-Object { 
			$_.TimeCreated -ge $startDate -and $_.TimeCreated -le $endDate -and $_.Message -match "Power Source" 
															} | 
																Sort-Object TimeCreated
		}

Function computerSleepWakeReason{
Get-WinEvent -ComputerName $env:COMPUTERNAME -ProviderName 'Microsoft-Windows-Kernel-Power' | 
Where-Object {
			$_.TimeCreated -ge $startDate -and $_.TimeCreated -le $endDate -and $_.Message -match 'Standby|Idle|InputTouchpad|Source|Power Button'
																				} | 
																					Sort-Object TimeCreated | Format-Table -AutoSize -Wrap
		}

Function computerSleepWakeReason_withFile{
$filepath = Read-Host "What is the file path for the downloaded Log? E.g. C:/users/username/documents/ 
You can also right-click the file in File Explorer and select `"Copy as Path`" then Paste it into here"


if ($filepath -match '^"(.*)"$') {
    $filepath = $matches[1]  # Extract the content between quotes
}

Write-Host $filepath

<# Check if the file exists and has the ".evtx" extension
#>
 if ((Test-Path $filepath -PathType Leaf) -and ($filepath -like "*.evtx")) {
        Get-WinEvent -Path $filepath |
            Where-Object { $_.Message -match 'Standby|Idle|InputTouchpad|Source|Power Button' } |
            Sort-Object TimeCreated |
            Format-Table -AutoSize -Wrap
    }
    else {
        Write-Host "Invalid file path or file does not exist or does not have a .evtx extension."
    }
}

Function computerBatteryReport{
<# Generate the battery report
#>
powercfg /batteryreport

<# Specify the source and destination paths
#>
$sourcePath = "C:\Windows\System32\battery-report.html"
$destinationPath = "C:\battery-report.html"

<# Move the file to the desired location
#>
Write-Host "`n
Moving File to C:\battery-report.html" -Foregroundcolor Yellow
Move-Item -Path $sourcePath -Destination $destinationPath -Force

Write-Host "Opening battery report in web browser" -Foregroundcolor Green

& "C:\battery-report.html" }



<# Checking if being run as admin
#>
Write-Host "Checking if Powershell is using Admin Privilege" -Foregroundcolor Yellow
$Adminuser=[bool](([System.Security.Principal.WindowsIdentity]::GetCurrent()).groups -match "S-1-5-32-544")
if($Adminuser -eq $True){
			Write-Host "Currently running as" "$env:USERNAME" "Proceeding with script" -Foregroundcolor Green
			$SwitchStart = Read-Host "`n `n

Type a number and then hit enter

1 - Check when user went from battery to charger

2 - Get Eventlog for Power/Sleep/Wake with Reason from Eventlog

3 - Get Eventlog for Power/Sleep/Wake with Reason using Eventlog file

4 - Generate an extensive battery report to an HTML page"

switch($SwitchStart){
		'1'{computerPowerSourceChange}
		'2'{computerSleepWakeReason}
		'3'{computerSleepWakeReason_withFile}
		'4'{computerBatteryReport}
		default{Write-Host "Invalid option selected" -ForegroundColor Red}}}
			elseif($Adminuser -eq $False){Write-Host "Powershell needs to be ran as admin" -Foregroundcolor Red

$AdminElevate= Read-host "Type Y or N on whether you'd like to run Powershell as Admin (Y=Yes,N=No)"
switch ($AdminElevate){
			Y{start-process powershell -verb runas}
			N{Write-Host "Aborting script" -Foregroundcolor Red}
		}
			}

