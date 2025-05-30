# PowerShell Script: Domain Controller Compromise Check

# Create output directory
$OutputDir = "C:\DC_Security_Check"
New-Item -ItemType Directory -Path $OutputDir -Force | Out-Null

# 1. Basic System Info
systeminfo > "$OutputDir\SystemInfo.txt"
Get-WmiObject win32_operatingsystem | Select-Object CSName, LastBootUpTime | Out-File "$OutputDir\Uptime.txt"

# 2. Running Processes
Get-Process | Sort-Object CPU -Descending | Select-Object Name, Id, CPU | Out-File "$OutputDir\Processes.txt"

# 3. Scheduled Tasks
schtasks /query /fo LIST /v > "$OutputDir\ScheduledTasks.txt"

# 4. Autorun Entries
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run > "$OutputDir\Startup_Run.txt"

# 5. Network Connections
netstat -anob > "$OutputDir\Netstat.txt"

# 6. Domain Admins Group Members
net group "Domain Admins" /domain > "$OutputDir\DomainAdmins.txt"

# 7. Security Event Logs (Last 500 Events)
Get-WinEvent -LogName Security -MaxEvents 500 | Export-Clixml -Path "$OutputDir\SecurityLogs.xml"

# 8. Recent User Logons
Get-EventLog -LogName Security -InstanceId 4624 -Newest 200 | Format-List > "$OutputDir\LogonEvents.txt"

# 9. Suspicious Services
Get-Service | Where-Object { $_.StartType -eq "Auto" -and $_.Status -eq "Running" } | Out-File "$OutputDir\RunningServices.txt"

# 10. Active Connections Summary
Get-NetTCPConnection | Group-Object State | Out-File "$OutputDir\TCPStateSummary.txt"

# 11. Inactive Users (Last 4 Weeks)
Search-ADAccount -AccountInactive -UsersOnly -TimeSpan 28.00:00:00 | Select-Object Name, LastLogonDate | Out-File "$OutputDir\InactiveUsers.txt"

Write-Output "Domain Controller security check complete. Output saved to $OutputDir"
