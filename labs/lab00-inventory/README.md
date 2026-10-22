# Lab 0 — Inventory

Pair: Akhmetov / Partner
Machine: Host (Windows 11)
Date: 17.10.2026

## Hardware Inventory

| Component | Clean Value | Command Used (PowerShell) |
| :--- | :--- | :--- |
| **CPU** | Intel / AMD, 8 cores / 8 threads | `Get-CimInstance Win32_Processor` |
| **RAM** | 16 GB, 2 modules | `Get-CimInstance Win32_PhysicalMemory` |
| **Disk** | SKHynix HFS256GEM9X169N, 238.47 GB, SSD | `Get-PhysicalDisk` |
| **BIOS** | LENOVO 071KT08A, 09/22/2025 | `Get-CimInstance Win32_BIOS` |
| **Virtualization** | Enabled (HyperVisorPresent: True) | `Get-ComputerInfo` |

## Result
System components correctly inventoried using native Windows PowerShell CIM/WMI commands.

## Evidence
- [evidence/disk.txt](evidence/disk.txt) — raw disk command output
- [evidence/virtualization.png](evidence/virtualization.png) — Task Manager proof of enabled virtualization
