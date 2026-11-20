# Lab 4 — Partition, mirror, fail, recover

Pair: Akhmetov (individual execution) · Date: 2026-11-20 · Hypervisor: Oracle VirtualBox 7.1

---

## Windows

```powershell
PS C:\Windows\system32> Get-Disk | Select Number, FriendlyName, PartitionStyle, Size

Number FriendlyName     PartitionStyle        Size
------ ------------     --------------        ----
     0 VBOX HARDDISK    GPT            53687091200
     1 VBOX HARDDISK    GPT             1073741824

PS C:\Windows\system32> Get-Volume | Where FileSystemLabel -eq 'LAB04'

DriveLetter FriendlyName FileSystemType DriveType HealthStatus OperationalStatus SizeRemaining      Size
----------- ------------ -------------- --------- ------------ ----------------- -------------      ----
E           LAB04        NTFS           Fixed     Healthy      OK                    495.88 MB 523.23 MB
