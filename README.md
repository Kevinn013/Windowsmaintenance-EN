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

Explanations about the certain features will be added in the end of this readme.
---

## Screenshot
*(<img width="659" height="479" alt="image" src="https://github.com/user-attachments/assets/78cb0e1a-7822-4efc-ac5b-6cb834144df5" />)*  

---

## Usage
1. Download the latest `.exe` or `.ahk` file from the repository.  
2. **Run as Administrator** (required for DISM/SFC commands).  
3. Choose your desired maintenance action from the tabbed interface.  
4. Check the **Logs** tab for detailed results and exit codes.

## Windows Security Windows Security Exclusion
Because this tool is a custom executable, Windows Security may flag or block it.
To ensure the application runs correctly, add it as an exclusion:

- Open Windows Security
- Go to Virus & threat protection
- Click Manage settings under Virus & threat protection settings
- Scroll down to Exclusions
- Click Add an exclusion → choose File
- Select the .exe of the Weekly Maintenance
  
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

## Explanations

**DISM**
DISM is an official windows tool which is a command-line tool that can be used to service and prepare Windows images.
https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/what-is-dism?view=windows-11

**SFC**
Scans and verifies the integrity of all protected system files and replaces incorrect versions with correct versions. If this command discovers that a protected file has been overwritten, it retrieves the correct version of the file from the systemroot\ folder, and then replaces the incorrect file.
https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/sfc

**Component store**
This is a core operating system directory. It manages and stores all system files, libraries, and packages required to run, update, customize, and recover Windows, safely handling multiple file versions via hard links to prevent software conflicts.
https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/manage-the-component-store?view=windows-11

**Windows Update reset**
clears and restarts the system services, temporary folders, and cache files responsible for downloading and installing system patches. This process fixes stuck updates, error codes, and corrupted data by forcing your PC to rebuild a clean update environment from scratch.
https://learn.microsoft.com/en-us/troubleshoot/windows-client/installing-updates-features-roles/additional-resources-for-windows-update

**Catroot2:**
The catroot2 folder is a crucial operating system directory in Windows located in C:\Windows\System32\catroot2 that stores digital signatures and database files for Windows Update packages. It acts as a temporary staging area used by the Cryptographic Service to verify the authenticity and integrity of update files during installation
**System Report**
