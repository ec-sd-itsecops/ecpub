**step-by-step guide** to assess whether DC may be **infected or compromised**

---

## 🛡 **Domain Controller Compromise Assessment Guide**

###  **Step 1: Baseline Info Gathering**

####  Check Basic System Integrity
- Open powershell 
- `hostname` / `whoami` — confirm you’re on the intended DC and using a privileged account.
- Check uptime: `systeminfo | find "System Boot Time"` — long uptimes may hide persistent malware.
- Record all running services: `services.msc` or `Get-Service` in PowerShell.

---

###  **Step 2: Check for Known Indicators of Compromise (IOCs)**

####  Look for Unusual Processes
- Use **Task Manager** or `Get-Process` in PowerShell.
- Focus on:
  - Suspicious names (e.g., `svch0st.exe`, `expl0rer.exe`)
  - Parent-child process anomalies (e.g., `lsass.exe` spawning PowerShell)

####  Examine Scheduled Tasks
- Run: `schtasks /query /fo LIST /v`
- Look for **oddly named or hidden tasks**, especially with encoded PowerShell or remote payloads.

####  Review Startup Items
- Use Autoruns (Sysinternals): [https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns)
- Look for **non-Microsoft signed entries** starting in Run/RunOnce, Winlogon, Services.

---

###  **Step 3: Check for Credential Theft Tools or Behavior**

####  Inspect LSASS Access Attempts
- Use Microsoft Defender or run `eventvwr.msc` and check:
  - **Event ID 4624** – Logon type 10 (remote), 3 (network), or new accounts.
  - **Event ID 4688** – Look for suspicious command-line activity (e.g., mimikatz, rundll32, powershell -enc).
  - **Event ID 4672** – Accounts granted **admin privileges**.
  
####  Review Security Logs:
- `Event Viewer → Windows Logs → Security`
  - Look for failed logons, multiple logons from unknown sources, new user creation, etc.
  
---

###  **Step 4: Check for Persistence Mechanisms**

####  Services or Registry Entries
- Run: `reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
- Look for suspicious entries (encoded scripts, remote URLs, etc.).

####  Tools to Use:
- **Autoruns**: View persistent items.
- **Process Explorer**: Verify all running processes are signed and legit.

---

###  **Step 5: Lateral Movement & Inbound Access**

####  Check Remote Access Attempts
- Review logs for **RDP, WinRM, SMB access**.
- Use:  
  `Get-WinEvent -LogName "Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational" | where Message -like "*successful*"`

####  Network Connections
- Run: `netstat -anob` or `Get-NetTCPConnection` to see remote IPs.
- Look for **unusual or external connections** from the DC.

---

###  **Step 6: Malware & EDR Validation**

### Full AV Scan
- Run full scan using Microsoft Defender:
  - `Start-MpScan -ScanType FullScan`
  
####  Check with EDR (if available)
- Cisco EDR / Defender ATP — check for:
  - Alerts on this host
  - Unusual behavior baselines
  - Suspicious binaries

---

###  **Step 7: Review Admin Groups & Users**

- Check Domain Admins group:  
  `net group "Domain Admins" /domain`  
- Look for:
  - Recently added users
  - Unknown accounts
- Validate with:  
  `dsquery user -inactive 4` — finds users inactive for 4 weeks (unexpected reactivation = suspect)

---

### **Step 8: Dump Suspicious Artifacts for Analysis**

- Export Event Logs:  
  `wevtutil epl Security C:\Logs\Security.evtx`
- Collect:
  - Prefetch files
  - Scheduled tasks
  - Registry hives

---

###  **If Anything Looks Suspicious**
- Immediately **disconnect the system from the network (but don’t power off)**.
- Engage your **incident response team** or escalate to **forensics analysts**.

---

