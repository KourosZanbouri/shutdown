
# Checking for Orderly vs. Unexpected Shutdowns: Linux and Windows

This document outlines the commands used to check system logs to determine if a shutdown was **orderly** (planned) or **unexpected** (due to a crash or power loss).




## 🐧 Linux

In Linux, the primary command for this is `last`, which reads from the `/var/log/wtmp` file.

### Primary Command

Use the `last` command with the `-x` flag to include shutdown and runlevel changes.

```bash
# Look at syslog for shutdown and reboot messages
last -x shutdown reboot

```

### How to Interpret the Output

The command lists events from newest to oldest. You must compare the entries to see what happened.

 **✅ Orderly Shutdown:** You will see a `shutdown` entry immediately followed (in time) by a `reboot` entry. This shows the system was properly commanded to shut down before it started again.
 
 ```
 reboot   system boot  5.15.0-76-generic  Wed Oct 29 14:30   still running
 shutdown system down  5.15.0-76-generic  Wed Oct 29 14:28 - 14:30  (00:02)

 ```

> **❌ Unexpected Shutdown (Crash):** You will see a `reboot` entry _without_ a corresponding `shutdown` entry right before it. Two `reboot` entries in a row imply the previous session ended abruptly.
> 
> ```
> reboot   system boot  5.15.0-76-generic  Wed Oct 29 11:00   still running
> reboot   system boot  5.15.0-76-generic  Wed Oct 29 09:15 - 11:00  (01:45)
> 
> ```

### Alternative (Modern systemd)

On modern Linux systems using `systemd`, you can check the journal for the _previous_ boot. An orderly shutdown will have clear "Shutting down..." messages at the end. A crash log will just stop.

Bash

```
# Show logs from the previous boot (-1) and jump to the end (-e)
journalctl -b -1 -e
```





## Windows (PowerShell)

In Windows, this is done by querying the Windows Event Log for specific **Event IDs**. The `Get-WinEvent` cmdlet in PowerShell is the best tool for this.

### Key Event IDs

-   **Unexpected Shutdown (Crash):**
    
    -   **ID 41 (Kernel-Power):** "The system has rebooted without cleanly shutting down first." (Logged _after_ the crash, on startup).
        
    -   **ID 6008 (EventLog):** "The previous system shutdown at [time] on [date] was unexpected." (Also logged on startup).
        
-   **Orderly Shutdown (Clean):**
    
    -   **ID 1074 (User32):** The most descriptive. Logs _who_ initiated the shutdown/restart and _why_ (e.g., "Windows Update").
        
    -   **ID 6006 (EventLog):** "The Event log service was stopped." (A reliable sign of a clean stop).
        
-   **System Startup (Boot):**
    
    -   **ID 6005 (EventLog):** "The Event log service was started."
        
    -   **ID 12 (Kernel-General):** "The operating system started..."
        

### PowerShell Commands

#### 1. To Find _Only_ Unexpected Shutdowns (Crashes)

This command filters for the two "crash" event IDs.

PowerShell

```
# Shows only crashes and unexpected reboots
Get-WinEvent -FilterHashtable @{LogName='System'; ID=41, 6008} -MaxEvents 10 | 
  Format-Table TimeCreated, ID, Message -AutoSize -Wrap

```

#### 2. To Find _Only_ Orderly Shutdowns

This command filters for the two "clean shutdown" event IDs.

PowerShell

```
# Shows only orderly, user/app-initiated shutdowns
Get-WinEvent -FilterHashtable @{LogName='System'; ID=1074, 6006} -MaxEvents 10 | 
  Format-Table TimeCreated, ID, Message -AutoSize -Wrap

```

#### 3. The Closest Equivalent to `last -x shutdown reboot`

This command combines all relevant IDs and sorts them by time, giving you a complete picture to analyze, just like the `last` command.

PowerShell

```
# Shows all startup, orderly, and unexpected events together
Get-WinEvent -FilterHashtable @{LogName='System'; ID=1074, 6006, 6008, 41, 6005, 12} -MaxEvents 20 | 
  Sort-Object TimeCreated -Descending | 
  Format-Table TimeCreated, ID, Message -AutoSize -Wrap
```
#### Blue Screen of Death

Here's the command to find those `BugCheck` **(Blue Screen of Death)** error codes.

This command searches for **Event ID 1001** and formats the output so you can clearly see the error code.

```
Get-WinEvent -ProviderName "Microsoft-Windows-WER-SystemErrorReporting" -FilterXPath "*[System[EventID=1001]]" -MaxEvents 5 | 
  Format-List TimeCreated, Message
```
