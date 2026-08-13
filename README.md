# 🧰 Windows Maintenance Suite (EN)

A lightweight **AutoHotkey v2** application for safe Windows maintenance and diagnostics.  
No registry edits, no performance tweaks — only official Microsoft servicing commands.
This tool is ideal for users who want safe, automated Windows maintenance without modifying system configuration.

---

## Features
- **[DISM CheckHealth](ca://s?q=DISM_CheckHealth)** – Verifies component store integrity  
- **[DISM RestoreHealth](ca://s?q=DISM_RestoreHealth)** – Repairs corrupted system components  
- **[SFC /scannow](ca://s?q=SFC_scannow)** – Scans and restores protected system files  
- **[Component Store Cleanup](ca://s?q=DISM_StartComponentCleanup)** – Removes outdated Windows components  
- **[Windows Update Reset](ca://s?q=Windows_Update_Reset)** – Recreates cache folders safely  
- **[Catroot2 Reset](ca://s?q=Catroot2_Reset)** – Fixes cryptographic service issues  
- **[System Report](ca://s?q=System_Report_generation)** – Generates detailed hardware and OS diagnostics  
- **[Log Management](ca://s?q=Log_management_in_AHK)** – View, clear, and export logs directly from the GUI  

---

## Screenshot
*(<img width="659" height="479" alt="image" src="https://github.com/user-attachments/assets/78cb0e1a-7822-4efc-ac5b-6cb834144df5" />)*  

---

## How to Use
1. Download the latest `.exe` or `.ahk` file from the repository.  
2. **Run as Administrator** (required for DISM/SFC commands).  
3. Choose your desired maintenance action from the tabbed interface.  
4. Check the **Logs** tab for detailed results and exit codes.

---

## Requirements
- Windows 10 or 11  
- Administrator privileges  
- AutoHotkey v2 (only for `.ahk` version)

---

## Why This Tool?
This suite automates official Windows maintenance commands without altering system configuration.  
It’s designed for **preventive health checks**, not aggressive optimization — perfect for users who want reliability and transparency.

---

## Author
**Kevin013**  
Created with ❤️ and curiosity for system integrity and safe automation.

---

## License
MIT License - free to use, modify, and distribute.

---

## Resources
Autohotkey 2.0 official webpage: https://www.autohotkey.com/v2/


---

## Usage & Windows Security Exclusion
Because this tool is a custom executable, Windows Security may flag or block it.
To ensure the application runs correctly, add it as an exclusion:

- Open Windows Security
- Go to Virus & threat protection
- Click Manage settings under Virus & threat protection settings
- Scroll down to Exclusions
- Click Add an exclusion → choose File
- Select the .exe of the Weekly Maintenance
