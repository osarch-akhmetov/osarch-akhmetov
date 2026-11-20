# Lab 4 — Partition, mirror, fail, recover

Pair: Akhmetov (individual execution) · Date: 2026-11-20 · Hypervisor: Oracle VirtualBox 7.1

---

## Windows

```powershell
PS C:\Windows\system32> Get-Disk

Number Friendly Name Serial Number        HealthStatus OperationalStatus Total Size Partition Style
------ ------------- -------------        ------------ ----------------- ---------- ---------------
0      VBOX HARDDISK VB607d8f47-3be3ebad  Healthy      Online                 50 GB MBR
1      VBOX HARDDISK VBf0425dcd-589a8641  Healthy      Online                  1 GB GPT

PS C:\Windows\system32> Get-Volume | Where DriveLetter -in 'E','F'

DriveLetter FriendlyName FileSystemType DriveType HealthStatus OperationalStatus SizeRemaining      Size
----------- ------------ -------------- --------- ------------ ----------------- -------------      ----
E           Lab04_NTFS   NTFS           Fixed     Healthy      OK                    483.78 MB    500 MB
F           LAB04_FAT32  FAT32          Fixed     Healthy      OK                    503.92 MB 503.94 MB
