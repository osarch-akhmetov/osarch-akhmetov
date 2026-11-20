# Lab 0 - Inventory

Pair: Akhmetov / Partner
Machine: Host (Windows 11)
Date: 20.11.2026

## Hardware Inventory

| Component | Clean Value | Command Used (PowerShell) |
| :--- | :--- | :--- |
| **CPU** | 12th Gen Intel(R) Core(TM) i7-1280P, 14 cores / 20 threads | `Get-CimInstance Win32_Processor \| Select Name, NumberOfCores, NumberOfLogicalProcessors, VirtualizationFirmwareEnabled` |
| **RAM** | 16 GB, 2 modules, <Speed из memory.txt> MT/s | `Get-CimInstance Win32_PhysicalMemory \| Select Capacity, Speed, Manufacturer` |
| **Disk** | Micron_2400_MTFDKBA1T0QFM, <MediaType из disk.txt>, <BusType из disk.txt> | `Get-PhysicalDisk \| Select FriendlyName, MediaType, BusType` |
| **Free Space (C:)** | <SizeRemaining из volume.txt> GB free of <Size из volume.txt> GB | `Get-Volume C` |
| **BIOS** | E17R1IMS.10C, 08/03/2022 | `Get-CimInstance Win32_BIOS \| Select SMBIOSBIOSVersion, ReleaseDate` |
| **Firmware Type** | Uefi | `(Get-ComputerInfo).BiosFirmwareType` |
| **Virtualization** | Enabled (`VirtualizationFirmwareEnabled: True`) | `Get-CimInstance Win32_Processor \| Select Name, NumberOfCores, NumberOfLogicalProcessors, VirtualizationFirmwareEnabled` |

## What did not work the first time
- Initially, values in the table were taken from another machine and Task Manager screenshots were used instead of native CLI outputs.
- Running multiple `Get-CimInstance` commands in one line without formatting hid the second output, so each query was run and saved separately into `evidence/`.

## Result
System components correctly inventoried using native Windows PowerShell CIM/WMI commands matching the actual host laptop.

## Evidence
- [evidence/cpu.txt](evidence/cpu.txt) - CPU cores, threads, and hardware virtualization status
- [evidence/memory.txt](evidence/memory.txt) - installed RAM modules, capacity, and speed
- [evidence/disk.txt](evidence/disk.txt) - physical disk model, media type, and bus type
- [evidence/volume.txt](evidence/volume.txt) - C: drive total and free space
- [evidence/bios.txt](evidence/bios.txt) - BIOS firmware version and release date
- [evidence/firmware.txt](evidence/firmware.txt) - system firmware boot mode (UEFI)
