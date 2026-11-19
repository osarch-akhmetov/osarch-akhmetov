# Lab 0 - Inventory

Pair: Akhmetov / Partner
Machine: Host (Windows 11)
Date: 17.10.2026

## Hardware Inventory

| Component | Clean Value | Command Used (PowerShell) |
| :--- | :--- | :--- |
| **CPU** | Intel Core i7-1280P, 14 cores / 20 threads | `Get-CimInstance Win32_Processor` |
| **RAM** | 16 GB, 2 modules | `Get-CimInstance Win32_PhysicalMemory` |
| **Disk** | Micron_2400_MTFDKBA1T0QFM, 953.87 GB, SSD | `Get-PhysicalDisk` |
| **BIOS** | MSI E17R1IMS.10C, 08/03/2022 | `Get-CimInstance Win32_BIOS` |
| **Virtualization** | Enabled (HyperVisorPresent: True) | `Get-ComputerInfo` |

## Result
System components correctly inventoried using native Windows PowerShell CIM/WMI commands.

## Evidence
- [evidence/disk.txt](evidence/disk.txt) - raw disk command output
- [evidence/virtualization.png](evidence/virtualization.png) - Task Manager proof of enabled virtualization
