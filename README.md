# Windows-Registry-Forensics-Artificat-Analysis-Lab

## 📌 Project Overview
This laboratory project focuses on investigating the Windows Registry to identify potential malware persistence mechanisms, analyze unauthorized software installations, and decode user activity tracking telemetry. By navigating core registry hives, I audited critical system keys to establish structural baselines and explored obfuscated execution logs.

## 🛠️ Tools & Technologies Used
* **Windows Registry Editor (`regedit`):** For direct hive manipulation, navigation, and key identification.
* **ROT13 Decoders:** For interpreting obfuscated string values stored inside user execution vaults.
* **Forensic Methodology:** Baseline verification and cryptographic file path cross-referencing.

---

## 🔍 Forensic Investigation Steps & Methodology

### Step 1: Auditing Machine-Wide Persistence (HKLM)
I navigated to the machine-level startup configuration path to identify any binaries configured to launch with administrative or system privileges:
* **Target Key Path:** `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
* **Findings:** The keys located were restricted to verified system utilities:
  * `RtkAudUService` (Realtek Audio Driver) located at `C:\Windows\System32\`
  * `SecurityHealth` (Windows Security Health Systray) located at `%windir%\system32\`
* **Conclusion:** Clean baseline. No malicious unverified executables or rogue installers masquerading as system files were present.

### Step 2: Auditing User-Specific Persistence (HKCU)
Attackers frequently target user-level run keys because modifying them does not require administrative elevation. I isolated the user profile's startup environment:
* **Target Key Path:** `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
* **Findings:** Identified standard client application hooks:
  * Google Chrome Auto-Launch (`C:\Program Files\Google\...`)
  * Microsoft Edge Auto-Launch (`C:\Program Files (x86)\Microsoft\...`)
  * Microsoft OneDrive (`C:\Users\admin\AppData\Local\...`)
  * OpenVPN GUI (`C:\Program Files\OpenVPN\bin\openvpn-gui.exe`)
* **Conclusion:** All active binaries map directly to known, legitimate software suites installed by the user.

### Step 3: Decoding User Activity Tracking (UserAssist)
To track user execution history, I targeted the `UserAssist` keys. Windows utilizes these hives to maintain a record of GUI applications launched, execution counts, and last-run timestamps. To prevent simple tampering, Windows obfuscates these values using a **ROT13** substitution cipher.

* **Target Key Path:** `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`
* **Findings:** Decoded encrypted application tracking artifacts:
  * `{1NP14R77-02R7...}` ➡️ Decodes to `1AP14E77-02E7` (Internal Windows Explorer Component)
  * `{6Q809377-6NSO...}` ➡️ Decodes to `6D809377-6AFD` (Desktop Shortcut Tracking)

---

## 📊 Core Analytical Conclusions
1. **Hardened Baseline:** Both HKLM and HKCU run environments display zero signs of unauthorized persistence vectors or masquerading executables.
2. **Definitive Activity Logging:** The presence of structural binary hex strings in the UserAssist `Count` keys provides irrefutable forensic evidence tracking application run loops, focus time, and specific operational windows.

---

## 📂 Repository Contents
* `README.md` - Documentation and project write-up.
* `/evidence-screenshots/` - Visual captures of the audited HKLM, HKCU, and UserAssist hives.
*
