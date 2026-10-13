# Lab 0 — Inventory your own machine
Pair: Akhmetov / Partner    Driver first half: Akhmetov
Machine: Host
Date: 13.10.2026

## What I did
Ran shell commands to gather hardware specs (CPU, RAM, Disk, Firmware, and Virtualization) and redirected command output into evidence files.

## Result

| What | Value | Where I got it |
| :--- | :--- | :--- |
| **CPU Model / Cores / Threads** | <Intel(R) N100             4                         4> | `lscpu` / `Get-CimInstance Win32_Processor` |
| **RAM Total / Modules / Speed** | <8589934592  5600 Controller0-ChannelA-DIMM0> | `sudo dmidecode -t memory` |
| **Disk Model / Type / Free space** | <C                        NTFS           Fixed     Healthy      OK                    191.81 GB  GB> | `df -h /` / `Get-Volume` |
| **Firmware Type / Version / Date** | UEFI | `sudo dmidecode -t bios` |
| **Hardware Virtualization** | Supported & Enabled | `lscpu` |

## What did not work the first time
Initial command output for RAM modules required elevated privileges (sudo / Administrator). Re-running with proper permissions solved the issue.

## Evidence
- [evidence/cpu.txt](evidence/cpu.txt) — Proves CPU architecture, core count, and virtualization status.
- [evidence/memory.txt](evidence/memory.txt) — Shows physical RAM stick count, total capacity, and memory bus speed.
- [evidence/disk.txt](evidence/disk.txt) — Shows physical storage medium type and available free space on root drive.
