# Checking for Orderly vs. Unexpected Shutdowns: Linux and Windows

This document outlines the commands used to check system logs to determine if a shutdown was **orderly** (planned) or **unexpected** (due to a crash or power loss).

---

## 🐧 Linux

In Linux, the primary command for this is `last`, which reads from the `/var/log/wtmp` file.

### Primary Command

Use the `last` command with the `-x` flag to include shutdown and runlevel changes.

```bash
# Look at syslog for shutdown and reboot messages
last -x shutdown reboot
