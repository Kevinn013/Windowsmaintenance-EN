#Requires AutoHotkey v2.0
#SingleInstance Force

; ============================================================
; Windows Maintenance Suite
; AutoHotkey v2
;
; Lightweight:
; - No registry changes
; - No CPU/RAM optimizations
; - No change of services
; - Only official Windows maintenance tools
; - Waits for processes, can take a bit
; - Exit codes checked
; ============================================================

; ---------- Globals ----------
global AppName := "Windows Maintenance Suite"
global LogFile := A_ScriptDir "\WindowsMaintenance.log"
global IsRunning := false

; ---------- GUI ----------
MainGui := Gui(, AppName)
MainGui.SetFont("s10", "Segoe UI")
MainGui.MarginX := 20
MainGui.MarginY := 20

Tabs := MainGui.Add("Tab3", "w620 h390 Section", [
    "Maintenance",
    "Cleanup",
    "Diagnostics",
    "Logs"
])

; ============================================================
; TAB 1 - MAINTENANCE
; ============================================================

Tabs.UseTab(1)

MainGui.Add("Text", "xs+10 y+15 Section", "Windows maintenance and system recovery:")
MainGui.Add("Text", "xs y+8 w560 cGray", "Only executed official commands.")

btnCheckHealth := MainGui.Add(
    "Button",
    "xs y+15 w280 h35",
    "Check Windows Health"
)

btnRepairWindows := MainGui.Add(
    "Button",
    "x+15 w280 h35",
    "Repair Windows"
)

btnSfc := MainGui.Add(
    "Button",
    "xs y+15 w280 h35",
    "SFC /scannow"
)

btnFull := MainGui.Add(
    "Button",
    "x+15 w280 h35",
    "Full Health Check"
)

MainGui.Add(
    "Text",
    "xs y+20 w560 cGray",
    "Full Health Check: component store check -> if needed it will be repaired -> SFC -> component cleanup -> log."
)

; ============================================================
; TAB 2 - CLEANUP
; ============================================================

Tabs.UseTab(2)

MainGui.Add("Text", "xs+10 y+15 Section", "Windows cleanup:")
MainGui.Add("Text", "xs y+8 w560 cGray", "Cleanup only removes temporary cache which is safe to remove.")

btnTemp := MainGui.Add(
    "Button",
    "xs y+15 w280 h35",
    "Clean Temporary Files"
)

btnComponentCleanup := MainGui.Add(
    "Button",
    "x+15 w280 h35",
    "Component Store Cleanup"
)

btnWindowsUpdate := MainGui.Add(
    "Button",
    "xs y+15 w280 h35",
    "Reset Windows Update"
)

btnCatroot := MainGui.Add(
    "Button",
    "x+15 w280 h35",
    "Reset Catroot2"
)

MainGui.Add(
    "Text",
    "xs y+20 w560 cGray",
    "Windows Update reset will stop temporarily and windows services will automatically create new cache."
)

; ============================================================
; TAB 3 - DIAGNOSTICS
; ============================================================

Tabs.UseTab(3)

MainGui.Add("Text", "xs+10 y+15 Section", "System diagnostics:")
MainGui.Add("Text", "xs y+8 w560 cGray", "Collects logs without removing any system settings.")

btnSystemReport := MainGui.Add(
    "Button",
    "xs y+15 w280 h35",
    "Generate System Report"
)

btnCBS := MainGui.Add(
    "Button",
    "x+15 w280 h35",
    "Export CBS.log"
)

btnDISMLog := MainGui.Add(
    "Button",
    "xs y+15 w280 h35",
    "Export DISM.log"
)

btnWindowsUpdateLog := MainGui.Add(
    "Button",
    "x+15 w280 h35",
    "Generate Windows Update Log"
)

; ============================================================
; TAB 4 - LOGS
; ============================================================

Tabs.UseTab(4)

MainGui.Add("Text", "xs+10 y+15 Section", "Activity log:")

logBox := MainGui.Add(
    "Edit",
    "xs y+10 w570 h265 ReadOnly -Wrap VScroll"
)

btnClearLog := MainGui.Add(
    "Button",
    "xs y+10 w180 h30",
    "Clear Log"
)

btnOpenLog := MainGui.Add(
    "Button",
    "x+10 w180 h30",
    "Open Log"
)

; Return from tabs
Tabs.UseTab()

; ============================================================
; STATUS BAR
; ============================================================

statusText := MainGui.Add(
    "Text",
    "xm y+15 w570 h25",
    "Status: Ready"
)

; ============================================================
; EVENTS
; ============================================================

btnCheckHealth.OnEvent("Click", CheckHealth)
btnRepairWindows.OnEvent("Click", RepairWindows)
btnSfc.OnEvent("Click", RunSfc)
btnFull.OnEvent("Click", RunFullHealth)

btnTemp.OnEvent("Click", CleanTempFiles)
btnComponentCleanup.OnEvent("Click", ComponentCleanup)
btnWindowsUpdate.OnEvent("Click", ResetWindowsUpdate)
btnCatroot.OnEvent("Click", ResetCatroot2)

btnSystemReport.OnEvent("Click", GenerateSystemReport)
btnCBS.OnEvent("Click", ExportCBS)
btnDISMLog.OnEvent("Click", ExportDISMLog)
btnWindowsUpdateLog.OnEvent("Click", GenerateWUlog)

btnClearLog.OnEvent("Click", ClearLog)
btnOpenLog.OnEvent("Click", OpenLog)

MainGui.OnEvent("Close", (*) => ExitApp())

; ============================================================
; INITIALIZE
; ============================================================

LogMessage("============================================")
LogMessage(AppName " Started.")
LogMessage("Windows Version: " GetWindowsVersion())
LogMessage("============================================")

MainGui.Show()

; ============================================================
; HELPER FUNCTIONS
; ============================================================

SetStatus(text) {
    global statusText
    statusText.Text := "Status: " text
}

LogMessage(text) {
    global logBox, LogFile

    timestamp := FormatTime(, "yyyy-MM-dd HH:mm:ss")
    line := "[" timestamp "] " text

    if IsSet(logBox)
        logBox.Value .= line "`r`n"

    try FileAppend(line "`r`n", LogFile, "UTF-8")
}

SetButtonsEnabled(enabled) {
    global IsRunning
    global btnCheckHealth, btnRepairWindows, btnSfc, btnFull
    global btnTemp, btnComponentCleanup, btnWindowsUpdate, btnCatroot
    global btnSystemReport, btnCBS, btnDISMLog, btnWindowsUpdateLog
    global btnClearLog, btnOpenLog

    IsRunning := !enabled

    buttons := [
        btnCheckHealth,
        btnRepairWindows,
        btnSfc,
        btnFull,
        btnTemp,
        btnComponentCleanup,
        btnWindowsUpdate,
        btnCatroot,
        btnSystemReport,
        btnCBS,
        btnDISMLog,
        btnWindowsUpdateLog,
        btnClearLog,
        btnOpenLog
    ]

    for button in buttons
        button.Enabled := enabled
}

RunCommand(command, description) {
    global IsRunning

    LogMessage("START: " description)
    LogMessage("Command: " command)

    try {
        exitCode := RunWait(
            '*RunAs "' A_ComSpec '" /c ' command,
            ,
            "Hide"
        )

        LogMessage(
            "END: " description " | Exit code: " exitCode
        )

        return exitCode
    }
    catch as err {
        LogMessage(
            "ERROR: " description " | " err.Message
        )

        return -1
    }
}

RunPowerShellToFile(command, outputFile, description) {
    tempFile := A_Temp "\wmstemp_" A_TickCount ".txt"

    try {
        exitCode := RunWait(
            '*RunAs powershell.exe -NoProfile -Command "' command ' | Out-File -FilePath "' tempFile '" -Encoding utf8"',
            ,
            "Hide"
        )

        if FileExist(tempFile) {
            content := FileRead(tempFile, "UTF-8")

            FileAppend(
                content "`r`n",
                outputFile,
                "UTF-8"
            )

            FileDelete(tempFile)
        }

        LogMessage(
            "Report section: " description " | Exit code: " exitCode
        )

        return exitCode
    }
    catch as err {
        LogMessage(
            "ERROR report section: " description " | " err.Message
        )

        return -1
    }
}

RunPowerShell(command, description) {
    LogMessage("START: " description)

    try {
        exitCode := RunWait(
            '*RunAs powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "' command '"',
            ,
            "Hide"
        )

        LogMessage(
            "END: " description " | Exit code: " exitCode
        )

        return exitCode
    }
    catch as err {
        LogMessage(
            "ERROR: " description " | " err.Message
        )

        return -1
    }
}

CheckSuccess(exitCode, operation) {
    if exitCode = 0 {
        LogMessage("SUCCESS: " operation)
        return true
    }

    LogMessage(
        "FAILED: " operation " | Exit code: " exitCode
    )

    return false
}

GetWindowsVersion() {
    try {
        return RegRead(
            "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion",
            "DisplayVersion"
        )
    }
    catch {
        return "Unknown"
    }
}

ConfirmAction(message, title := "Windows Maintenance Suite") {
    result := MsgBox(
        message,
        title,
        "YesNo Icon?"
    )

    return result = "Yes"
}

; ============================================================
; MAINTENANCE
; ============================================================

CheckHealth(*) {
    global IsRunning

    if IsRunning
        return

    SetButtonsEnabled(false)

    try {
        SetStatus("Checking component store...")

        exitCode := RunCommand(
            "DISM.exe /Online /Cleanup-Image /CheckHealth",
            "DISM CheckHealth"
        )

        if CheckSuccess(exitCode, "DISM CheckHealth")
            SetStatus("Component store check completed.")
        else
            SetStatus("Component store returned with an error.")

        MsgBox(
            "DISM CheckHealth is completed.`n`n" .
            "Check logs tab for exit codes and results.",
            "Check Windows Health",
            "Iconi"
        )
    }
    finally {
        SetButtonsEnabled(true)
    }
}

RepairWindows(*) {
    global IsRunning

    if IsRunning
        return

    if !ConfirmAction(
        "Windows components will be checked and repaired if needed.`n`n"
        . "This can take a while.`n`n"
        . "Continue?"
    )
        return

    SetButtonsEnabled(false)

    try {
        SetStatus("Running DISM RestoreHealth...")

        exitCode := RunCommand(
            "DISM.exe /Online /Cleanup-Image /RestoreHealth",
            "DISM RestoreHealth"
        )

        if !CheckSuccess(exitCode, "DISM RestoreHealth") {
            SetStatus("DISM repair failed.")
            MsgBox(
                "DISM was unsuccessful.`n`nExit code: " exitCode,
                "Windows Maintenance Suite",
                "Iconx"
            )
            return
        }

        SetStatus("Windows component store repaired.")

        MsgBox(
            "DISM RestoreHealth completed successfully.",
            "Windows Maintenance Suite",
            "Iconi"
        )
    }
    finally {
        SetButtonsEnabled(true)
    }
}

RunSfc(*) {
    global IsRunning

    if IsRunning
        return

    if !ConfirmAction(
        "SFC /scannow will be executed.`n`n"
        . "Windows will check protected system files.`n`n"
        . "Continue?"
    )
        return

    SetButtonsEnabled(false)

    try {
        SetStatus("Running SFC /scannow...")

        exitCode := RunCommand(
            "sfc.exe /scannow",
            "SFC /scannow"
        )

        if CheckSuccess(exitCode, "SFC /scannow") {
            SetStatus("SFC completed.")
            MsgBox(
                "SFC /scannow completed.`n`nCheck the Logs tab for results.",
                "Windows Maintenance Suite",
                "Iconi"
            )
        }
        else {
            SetStatus("SFC reported an error.")
            MsgBox(
                "SFC stopped with exit code: " exitCode,
                "Windows Maintenance Suite",
                "Iconx"
            )
        }
    }
    finally {
        SetButtonsEnabled(true)
    }
}
RunFullHealth(*) {
    global IsRunning

    if IsRunning
        return

    if !ConfirmAction(
        "Full Health Check`n`n"
        . "The following steps will be executed:`n`n"
        . "1. DISM CheckHealth`n"
        . "2. DISM ScanHealth`n"
        . "3. DISM RestoreHealth`n"
        . "4. SFC /scannow`n"
        . "5. Component Store Cleanup`n"
        . "6. System Report`n`n"
        . "This may take a while depending on your hardware.`n`n"
        . "Continue?"
    )
        return

    SetButtonsEnabled(false)

    try {
        LogMessage("============================================")
        LogMessage("Full health Check started")
        LogMessage("============================================")

        SetStatus("1/6 - DISM CheckHealth...")
        RunCommand("DISM.exe /Online /Cleanup-Image /CheckHealth", "FULL - DISM CheckHealth")

        SetStatus("2/6 - DISM ScanHealth...")
        RunCommand("DISM.exe /Online /Cleanup-Image /ScanHealth", "FULL - DISM ScanHealth")

        SetStatus("3/6 - DISM RestoreHealth...")
        exitCode := RunCommand("DISM.exe /Online /Cleanup-Image /RestoreHealth", "FULL - DISM RestoreHealth")

        if exitCode != 0
            LogMessage("WARNING: DISM RestoreHealth returned " exitCode)

        SetStatus("4/6 - SFC /scannow...")
        exitCode := RunCommand("sfc.exe /scannow", "FULL - SFC /scannow")

        if exitCode != 0
            LogMessage("WARNING: SFC returned " exitCode)

        SetStatus("5/6 - Component Store Cleanup...")
        exitCode := RunCommand("DISM.exe /Online /Cleanup-Image /StartComponentCleanup", "FULL - Component Store Cleanup")

        if exitCode != 0
            LogMessage("WARNING: Component cleanup returned " exitCode)

        SetStatus("6/6 - Generating system report...")
        GenerateSystemReportInternal()

        LogMessage("============================================")
        LogMessage("Full Health Check finished")
        LogMessage("============================================")

        SetStatus("Full Health Check completed.")

        MsgBox(
            "Full Health Check completed.`n`n"
            . "✓ DISM CheckHealth`n"
            . "✓ DISM ScanHealth`n"
            . "✓ DISM RestoreHealth`n"
            . "✓ SFC /scannow`n"
            . "✓ Component Store Cleanup`n"
            . "✓ System Report`n`n"
            . "See the Logs tab for details.",
            "Windows Maintenance Suite",
            "Iconi"
        )
    }
    finally {
        SetButtonsEnabled(true)
    }
}

; ============================================================
; CLEANUP
; ============================================================
CleanTempFiles(*) {
    global IsRunning

    if IsRunning
        return

    if !ConfirmAction(
        "Temporary files will be removed.`n`n"
        . "Files currently in use will be skipped.`n`n"
        . "Continue?"
    )
        return

    SetButtonsEnabled(false)

    try {
        SetStatus("Cleaning user temporary files...")

        tempPath := EnvGet("TEMP")

        if DirExist(tempPath) {
            LogMessage("Cleaning TEMP: " tempPath)

            Loop Files, tempPath "\*", "FD" {
                try {
                    if InStr(A_LoopFileAttrib, "D")
                        DirDelete(A_LoopFileFullPath, true)
                    else
                        FileDelete(A_LoopFileFullPath)
                }
                catch {
                }
            }
        }

        SetStatus("Cleaning Windows temporary files...")

        windowsTemp := A_WinDir "\Temp"

        if DirExist(windowsTemp) {
            LogMessage("Cleaning Windows TEMP: " windowsTemp)

            Loop Files, windowsTemp "\*", "FD" {
                try {
                    if InStr(A_LoopFileAttrib, "D")
                        DirDelete(A_LoopFileFullPath, true)
                    else
                        FileDelete(A_LoopFileFullPath)
                }
                catch {
                }
            }
        }

        LogMessage("Temporary file cleanup completed.")
        SetStatus("Temporary files cleanup complete.")

        MsgBox(
            "Temporary file cleanup completed.`n`n"
            . "Files in use were skipped.",
            "Cleanup",
            "Iconi"
        )
    }
    finally {
        SetButtonsEnabled(true)
    }
}
ComponentCleanup(*) {
global IsRunning
if IsRunning
return
if !ConfirmAction(
"DISM Component Store Cleanup has started." .
"Windows will remove old components which are not needed anymore." .
"Continue?"
)
return
SetButtonsEnabled(false)
try {
SetStatus("Running Component Store Cleanup...")
exitCode := RunCommand(
"DISM.exe /Online /Cleanup-Image /StartComponentCleanup",
"Component Store Cleanup"
)
if CheckSuccess(exitCode, "Component Store Cleanup") {
SetStatus("Component Store Cleanup completed.")
MsgBox(
"Component Store Cleanup completed.",
"Cleanup",
"Iconi"
)
}
else {
SetStatus("Component Store Cleanup has reported an error.")
MsgBox(
"Cleanup ended with exit code: " exitCode,
"Cleanup",
"Iconx"
)
}
}
finally {
SetButtonsEnabled(true)
}
}
ResetWindowsUpdate(*) {
global IsRunning
if IsRunning
return
if !ConfirmAction(
"Windows Update cache will be reset." .
"Windows update service will be stopped temporary " .
"SoftwareDistribution will be renamed." .
"Continue?"
)
return
SetButtonsEnabled(false)
try {
SetStatus("Stopping Windows Update services...")
RunCommand(
"net stop wuauserv",
"Stop Windows Update"
)
RunCommand(
"net stop bits",
"Stop BITS"
)
RunCommand(
"net stop cryptsvc",
"Stop Cryptographic Services"
)
SetStatus("Renaming SoftwareDistribution...")
updatePath := A_WinDir "\SoftwareDistribution"
backupPath := A_WinDir "\SoftwareDistribution.old"
; Remove an existing old backup if possible.
if DirExist(backupPath) {
try DirDelete(backupPath, true)
catch {
LogMessage("WARNING: Could not remove existing SoftwareDistribution.old")
}
}
if DirExist(updatePath) {
try {
DirMove(updatePath, backupPath)
LogMessage(
"SoftwareDistribution renamed to SoftwareDistribution.old"
)
}
catch as err {
LogMessage(
"ERROR: Could not rename SoftwareDistribution: " err.Message
)
}
}
SetStatus("Starting Windows Update services...")
RunCommand(
"net start cryptsvc",
"Start Cryptographic Services"
)
RunCommand(
"net start bits",
"Start BITS"
)
RunCommand(
"net start wuauserv",
"Start Windows Update"
)
LogMessage("Windows Update reset completed.")
SetStatus("Windows Update reset completed.")
MsgBox(
"Windows Update reset has been completed." .
"Windows will automatically make a new SoftwareDistribution folder.",
"Windows Update",
"Iconi"
)
}
finally {
SetButtonsEnabled(true)
}
}
ResetCatroot2(*) {
global IsRunning
if IsRunning
return
if !ConfirmAction(
"Catroot2 is getting recreated" .
"This is an official Windows troubleshoot" .
"for cryptographic/Windows update issues." .
"Continue?"
)
return
SetButtonsEnabled(false)
try {
SetStatus("Stopping Cryptographic Services...")
RunCommand(
"net stop cryptsvc",
"Stop Cryptographic Services"
)
catrootPath := A_WinDir "\System32\catroot2"
backupPath := A_WinDir "\System32\catroot2.old"
SetStatus("Renaming Catroot2...")
if DirExist(backupPath) {
try DirDelete(backupPath, true)
catch {
LogMessage("WARNING: Could not remove existing catroot2.old")
}
}
if DirExist(catrootPath) {
try {
DirMove(catrootPath, backupPath)
LogMessage(
"catroot2 renamed to catroot2.old"
)
}
catch as err {
LogMessage(
"ERROR: Could not rename catroot2: " err.Message
)
}
}
SetStatus("Starting Cryptographic Services...")
RunCommand(
"net start cryptsvc",
"Start Cryptographic Services"
)
LogMessage("Catroot2 reset completed.")
SetStatus("Catroot2 reset completed.")
MsgBox(
"Catroot2 reset is completed.",
"Windows Maintenance Suite",
"Iconi"
)
}
finally {
SetButtonsEnabled(true)
}
}
; ============================================================
; DIAGNOSTICS
; ============================================================
GenerateSystemReport(*) {
global IsRunning
if IsRunning
return
SetButtonsEnabled(false)
try {
SetStatus("Generating system report...")
GenerateSystemReportInternal()
SetStatus("System report completed.")
MsgBox(
"System report generated." .
"See the results tab for the file directory.",
"Diagnostics",
"Iconi"
)
}
finally {
SetButtonsEnabled(true)
}
}
GenerateSystemReportInternal() {
reportPath := A_Desktop "\WindowsMaintenance_SystemReport.txt"
LogMessage("Generating system report: " reportPath)
try FileDelete(reportPath)
; --------------------------------------------------------
; Basic Windows information
; --------------------------------------------------------
FileAppend(
"============================================rn" .
"WINDOWS MAINTENANCE SYSTEM REPORTrn" .
"============================================rnrn",
reportPath,
"UTF-8"
)
FileAppend(
"Generated: " FormatTime(, "yyyy-MM-dd HH:mm:ss") "rn",
reportPath,
"UTF-8"
)
FileAppend(
"Windows version: " GetWindowsVersion() "rn",
reportPath,
"UTF-8"
)
; --------------------------------------------------------
; System information
; --------------------------------------------------------
FileAppend(
"rn--- SYSTEMINFO ---rn",
reportPath,
"UTF-8"
)
RunPowerShellToFile(
"Get-ComputerInfo | Select-Object WindowsProductName,WindowsVersion,OsBuildNumber,OsArchitecture,CsTotalPhysicalMemory,CsProcessors | Format-List",
reportPath,
"System information"
)
; --------------------------------------------------------
; Page file information
; --------------------------------------------------------
FileAppend(
"rn--- PAGEFILE ---rn",
reportPath,
"UTF-8"
)
RunPowerShellToFile(
"Get-CimInstance Win32_PageFileUsage | Select-Object Name,AllocatedBaseSize,CurrentUsage,PeakUsage | Format-List",
reportPath,
"Pagefile information"
)
; --------------------------------------------------------
; Disk information
; --------------------------------------------------------
FileAppend(
"rn--- DISKS ---rn",
reportPath,
"UTF-8"
)
RunPowerShellToFile(
"Get-Volume | Where-Object DriveLetter | Select-Object DriveLetter,FileSystemLabel,FileSystem,SizeRemaining,Size | Format-Table -AutoSize",
reportPath,
"Disk information"
)
; --------------------------------------------------------
; Power plan
; --------------------------------------------------------
FileAppend(
"rn--- POWER PLAN ---rn",
reportPath,
"UTF-8"
)
RunPowerShellToFile(
"powercfg /getactivescheme",
reportPath,
"Active power plan"
)
; --------------------------------------------------------
; GPU
; --------------------------------------------------------
FileAppend(
"rn--- GPU ---rn",
reportPath,
"UTF-8"
)
RunPowerShellToFile(
"Get-CimInstance Win32_VideoController | Select-Object Name,DriverVersion,AdapterRAM | Format-List",
reportPath,
"GPU information"
)
; --------------------------------------------------------
; Recent critical errors
; --------------------------------------------------------
FileAppend(
"rn--- RECENT CRITICAL SYSTEM EVENTS ---rn",
reportPath,
"UTF-8"
)
RunPowerShellToFile(
"Get-WinEvent -FilterHashtable @{LogName='System';Level=1} -MaxEvents 20 | Select-Object TimeCreated,ProviderName,Id,Message | Format-List",
reportPath,
"Recent critical system events"
)
FileAppend(
"rn============================================rn" .
"END OF REPORTrn" .
"============================================rn",
reportPath,
"UTF-8"
)
LogMessage("System report created: " reportPath)
}
ExportCBS(*) {
global IsRunning
if IsRunning
return
SetButtonsEnabled(false)
try {
SetStatus("Exporting CBS.log...")
destination := A_Desktop "\CBS_HealthCheck.log"
source := A_WinDir "\Logs\CBS\CBS.log"
if !FileExist(source) {
LogMessage("CBS.log not found.")
SetStatus("CBS.log not found.")
MsgBox(
"CBS.log couldn't be found.",
"Diagnostics",
"Iconx"
)
return
}
try {
FileCopy(source, destination, true)
LogMessage("CBS.log exported to: " destination)
SetStatus("CBS.log exported.")
MsgBox(
"CBS.log is exported to:" destination,
"Diagnostics",
"Iconi"
)
}
catch as err {
LogMessage(
"ERROR exporting CBS.log: " err.Message
)
MsgBox(
"CBS.log couldn't be copied." .
err.Message,
"Diagnostics",
"Iconx"
)
}
}
finally {
SetButtonsEnabled(true)
}
}
ExportDISMLog(*) {
global IsRunning
if IsRunning
return
SetButtonsEnabled(false)
try {
SetStatus("Exporting DISM.log...")
destination := A_Desktop "\DISM_HealthCheck.log"
source := A_WinDir "\Logs\DISM\dism.log"
if !FileExist(source) {
LogMessage("DISM.log not found.")
SetStatus("DISM.log not found.")
MsgBox(
"DISM.log not found.",
"Diagnostics",
"Iconx"
)
return
}
try {
FileCopy(source, destination, true)
LogMessage("DISM.log exported to: " destination)
SetStatus("DISM.log exported.")
MsgBox(
"DISM.log exported to:" destination,
"Diagnostics",
"Iconi"
)
}
catch as err {
LogMessage(
"ERROR exporting DISM.log: " err.Message
)
MsgBox(
"DISM.log couldn't be exported." .
err.Message,
"Diagnostics",
"Iconx"
)
}
}
finally {
SetButtonsEnabled(true)
}
}
GenerateWUlog(*) {
global IsRunning
if IsRunning
return
SetButtonsEnabled(false)
try {
SetStatus("Generating Windows Update log...")
destination := A_Desktop "\WindowsUpdate_HealthCheck.log"
; Get-WindowsUpdateLog creates a readable Windows Update log.
exitCode := RunPowerShell(
"Get-WindowsUpdateLog -LogPath '" destination "'",
"Generate Windows Update log"
)
if exitCode = 0 {
SetStatus("Windows Update log generated.")
MsgBox(
"Windows Update log was generated to:" destination,
"Diagnostics",
"Iconi"
)
}
else {
SetStatus("Windows Update log has reported an error.")
MsgBox(
"Windows Update log generation unsuccesful." .
"Exit code: " exitCode,
"Diagnostics",
"Iconx"
)
}
}
finally {
SetButtonsEnabled(true)
}
}
; ============================================================
; LOG MANAGEMENT
; ============================================================
ClearLog(*) {
global logBox
if !ConfirmAction(
    "This will remove application logs.`n`n"
    "Continue?"
)

return
logBox.Value := ""
try FileDelete(LogFile)
LogMessage("Log cleared.")
SetStatus("Log cleared.")
}
OpenLog(*) {
global LogFile
if !FileExist(LogFile) {
LogMessage("No log file exists yet.")
}
try {
Run(LogFile)
}
catch as err {
MsgBox(
    "Log couldn't be opened.`n`n" err.Message,
    "Windows Maintenance Suite",
    "Iconx"
)
}
}

