Set-StrictMode -Off;

#region Initial checks
$platform = [string][System.Environment]::OSVersion.Platform
if(!($platform.StartsWith("Win"))) {
	[console]::error.writeline("WSL-AltInstaller: This script works and is intended only for the Windows system.")
	exit 1
}

if([System.Environment]::OSVersion.Version.Build -lt 19041) {
	[console]::error.writeline("WSL-AltInstaller: This script only supports Windows 11 and Windows 10 version 2004 or higher.")
	exit 1
}

if(!((Get-CimInstance Win32_operatingsystem).OSArchitecture -eq "64-bit")) {
	[console]::error.writeline("WSL-AltInstaller: This script works only on a 64-bit Windows system.")
	exit 1
}

if([System.Environment]::UserName -eq "Administrator") {
	# This can usually happen in AME10/11, VERY rarely by the user's own intent (which is stupid IMO).
	[console]::error.writeline("WSL-AltInstaller: You literally run this script as 'Administrator', not as your personal account. WSL and a distribution of your choice will only be installed on this system account and your personal account will not be able to run these applications.")
	if (Get-Command "amecs.exe" -ErrorAction SilentlyContinue) { 
		[console]::error.writeline("WSL-AltInstaller: Since you are using AME10/11, you can open 'Central AME Script' from the Start menu or by opening 'amecs' in the 'Run' application and apply 'Elevate User' temporarily.")
	} else {
		[console]::error.writeline("WSL-AltInstaller: We recommend that you log out of this system account and use only your own.")
	}
	exit 1
}

if(!([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
	[console]::error.writeline("WSL-AltInstaller: PowerShell was not run as an Administrator.")
	exit 1
}

if(!((Get-NetConnectionProfile).IPv4Connectivity -contains "Internet" -or (Get-NetConnectionProfile).IPv6Connectivity -contains "Internet")) {
	[console]::error.writeline("WSL-AltInstaller: This script requires a stable Internet connection.")
	exit 1
}
#endregion

$prevdir = (Get-Location).Path
$tempdir = [System.IO.Path]::GetTempPath() + "\WSL-AltInstaller"
New-Item -Path $tempdir -ItemType "directory" -Force | Out-Null
Set-Location $tempdir

if ((Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform).State -ne 'Enabled'){
	"Enabling 'Virtual Machine Platform'..."
    $vmfeature = Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName VirtualMachinePlatform -LimitAccess | Out-Null
	if($vmfeature.Restartneeded -eq $true) {
		$rebootReq = $true
	}
} else {
    "Virtual Machine Platform was already enabled."
}

if ((Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux).State -ne 'Enabled'){
    "Enabling 'Windows Subsystem for Linux'..."
    $wslfeature = Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName Microsoft-Windows-Subsystem-Linux -LimitAccess | Out-Null
	if($wslfeature.Restartneeded -eq $true) {
		$rebootReq = $true
	}
} else {
    "Windows Subsystem for Linux was already enabled."
}

$aria2 = Invoke-RestMethod -Uri https://api.github.com/repos/aria2/aria2/releases/latest | Select-Object -ExpandProperty assets | Select-Object -expand browser_download_url
Invoke-WebRequest $aria2[2] -OutFile "aria2.zip"
Expand-Archive .\aria2.zip
Move-Item "aria2\*\aria2c.exe" .
Remove-Item -LiteralPath "aria2" -Force -Recurse
Remove-Item -LiteralPath "aria2.zip"

if(!((Get-AppPackage).Name -like "*WindowsSubsystemForLinux*")) {
	$wsl = Invoke-RestMethod -Uri https://api.github.com/repos/microsoft/WSL/releases
	$wsl = $wsl[0] | Select-Object -ExpandProperty assets | Select-Object -expand browser_download_url

	& ".\aria2c.exe" "-x16" "-s16" "-k4M" $wsl "-o" "wsl.msixbundle"
	Add-AppxPackage "wsl.msixbundle"
}

if(!((Get-AppPackage).Name -like "*Ubuntu*")) {
	& ".\aria2c.exe" "-x16" "-s16" "-k4M" "https://aka.ms/wslubuntu" "-o" "ubuntu.appxbundle"
	Add-AppxPackage "ubuntu.appxbundle"
}

Set-Location $prevdir
Remove-Item -LiteralPath $tempdir -Force -Recurse

if($rebootReq) {
	"WSL-AltInstaller: The requested operation is successful (hopefully). Changes will not be effective until the system is rebooted."
} else {
	"WSL-AltInstaller: The requested operation is successful (hopefully)."
}
