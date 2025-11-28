# Windows Server Automated Full Image Backup Repository

This repository contains a complete, ready‑to‑use setup for automated Windows Server image backups.

Below is the full repo layout followed by the original README.

```
windows-server-backup-repo/
├── scripts/
│   └── Full-Image-Backup.ps1
└── README.md
```

---

# Windows Server Automated Full Image Backup (README)

This README provides a universal, production-ready backup solution for **any Windows Server version** (2012 / 2016 / 2019 / 2022 / 2025) running on **physical hardware**.

The solution creates a **full system image backup** that can be fully restored on the same or different hardware.

---

## 📌 Overview

This setup performs:

* Full System Image Backup
* OS Partition Backup
* Boot + EFI Partition Backup
* System State Backup
* Automated Scheduled Backups
* Restore capability to any physical server

The backup uses Windows' built‑in `wbadmin` tool.

---

## 📁 Folder Structure

Create the following directory on the Windows Server:

```
C:\scripts\
```

Place the backup script inside:

```
C:\scripts\Full-Image-Backup.ps1
```

Create (or provide access to) a backup destination:

```
\<SERVER>\WinBackup
```

Example:

```
\\192.168.1.10\WinBackup
```

---

## 🔧 Step 1 — Install Windows Server Backup Feature

Open **PowerShell (Run as Administrator)** and run:

```
Install-WindowsFeature Windows-Server-Backup
```

---

## 🔧 Step 2 — Create the Universal Backup Script

Create a file:

```
C:\scripts\Full-Image-Backup.ps1
```

and insert this:

```
# ============================
# UNIVERSAL WINDOWS SERVER IMAGE BACKUP SCRIPT
# ============================

# Backup destination (NAS / USB / HDD / Network Share)
$BackupTarget = "\\192.168.1.10\WinBackup"   # Change to your environment

# Windows system drive
$SystemDrive = "C:"

# Check if backup destination is reachable
if (!(Test-Path $BackupTarget)) {
    Write-Output "Backup target unreachable!"
    exit
}

# Run full image backup (System + OS + Boot)
wbadmin start backup `
    -backupTarget:$BackupTarget `
    -include:$SystemDrive `
    -allCritical `
    -quiet
```

This will create a complete system image usable for bare‑metal recovery.

---

## 🔧 Step 3 — Create an Automated Scheduled Backup

1. Open Task Scheduler:

```
taskschd.msc
```

2. Click **Create Task**
3. Name it: `Windows Full Image Backup`
4. Select:

   * **Run whether user is logged on or not**
   * **Run with highest privileges**
5. Go to **Triggers → New**

   * Set schedule (Daily/Weekly)
6. Go to **Actions → New**

   * Program:

```
powershell.exe
```

* Arguments:

```
-ExecutionPolicy Bypass -File "C:\scripts\Full-Image-Backup.ps1"
```

7. Save the task and enter your admin password.

Backup will now run automatically.

---

## 🧪 Step 4 — Verify Backup

Check backup history:

```
wbadmin get versions
```

Verify backup location has `.vhdx` or `.vhd` images.

---

## 🛠️ Restore (Bare‑Metal Recovery)

If the server crashes or you need to restore to a new machine:

1. Boot from Windows Server ISO
2. Choose **Repair your computer**
3. Go to **Troubleshoot → System Image Recovery**
4. Choose:

   * Network location
   * USB drive
   * External disk
5. Select the backup created by `wbadmin`
6. Run the full restore

Windows will restore:

* OS
* Bootloader
* Drivers (auto-detect)
* System State
* Entire disk layout

This works even on different hardware.

---

## 📝 Notes

* Restore requires the target disk to be **equal or larger** in size.
* If restoring to different hardware, Windows will install drivers automatically.
* Script is compatible with:

  * BIOS / UEFI
  * MBR / GPT
  * SATA / NVMe
  * Any Windows Server edition

---

## ✔️ This README Gives You

* Full automated backup solution
* Full bare‑metal restore capability
* A single script that works everywhere
* No third‑party software needed

---

If you want, I can add:

* Email notification after backup
* Zip/compress old backups
* Retention policy cleanup script
* Support for multiple partitions

Just tell me and I’ll update the README.
