Here’s a **step-by-step plan** for your IT Security team member conducting onsite forensic triage and EDR installation on **12 compromised Windows 10 desktops**. Since you're planning to reimage them, the focus should be **quick evidence collection, threat containment, and EDR installation** to monitor any post-infection behavior until the reimage is complete.

---

## 🔐 Onsite Forensic & Containment Plan (Windows 10)

### 🛠️ 1. **Preparation (Before Visit)**
- Bring:
  - Encrypted external USB drives (or SSDs) for log collection
  - Bootable Windows PE or Linux USB (if offline disk imaging is needed)
  - Cisco AMP installer for Windows (downloaded from console)
  - Trusted tools: Sysinternals Suite, FTK Imager Lite, KAPE, Velociraptor Agent (if licensed), or other IR kits
- Ensure local admin credentials are available
- Confirm no machines are powered off or tampered with before arrival

---

### 🔎 2. **Initial Triage (On Each Machine)**
1. **Do not power off or reboot the system.**
2. Disconnect from the internet **if not needed for forensic collection.**
3. Confirm local time settings (important for log correlation).
4. **Collect logs and system snapshots:**
   - **Windows Event Logs**:
     ```bash
     wevtutil epl System C:\Logs\System.evtx
     wevtutil epl Security C:\Logs\Security.evtx
     wevtutil epl Application C:\Logs\Application.evtx
     ```
   - Export key artifacts:
     - `Prefetch`
     - `$MFT`, `$LogFile`, `$UsnJrnl`
     - Registry hives: `SAM`, `SYSTEM`, `SECURITY`, `SOFTWARE`
     - Browser history (optional)
     - Run `autoruns` from Sysinternals to snapshot persistence mechanisms
   - Capture process list, network connections:
     ```bash
     tasklist /v > C:\Logs\tasks.txt
     netstat -ano > C:\Logs\netstat.txt
     ```
   - Dump current memory (optional, if using tools like DumpIt or Magnet RAM Capture)

> 📁 Store all logs securely in a `C:\Logs` folder and transfer to USB for central analysis

---

### 🧼 3. **Containment**
- If live network activity is suspicious, block outbound connections via Windows Firewall
- Stop unknown scheduled tasks or startup entries temporarily
- Revoke compromised user credentials

---

### 🧪 4. **Install Cisco AMP for Endpoints (EDR)**
1. Run the installer with local admin:
   ```bash
   amp_win_x64_installer.exe
   ```
2. Confirm device registration in the Cisco AMP console
3. Let the AMP agent run to observe any suspicious behavior pre-reimage

---

### 💾 5. **Transfer Logs for Central Review**
- Encrypt USB drives if needed
- Create SHA256 hashes of files if chain-of-custody is needed
- Upload to secure internal share/SIEM/IR analyst workstation

---

### 🔁 6. **Post-Triage: Reimage**
- Use your standard corporate Windows 10 image
- Rejoin to domain
- Harden endpoints (apply CIS baseline if applicable)
- Reinstall Cisco AMP and verify connectivity
- Restore only clean, scanned data from backups if needed

---

## 📌 Recommended Forensic Tools (Free / Lightweight)
- [KAPE](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor)
- [FTK Imager Lite](https://accessdata.com/product-download)
- [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/)
- [Velociraptor](https://www.velociraptor.app/) (if available in your org)

Would you like a checklist version you can print or share internally?
