# Lab 3 — Teardown, and the same machine from the command line

Pair: Alzhan Akhmetov  
Machine: host (Windows laptop), Linux guest (Ubuntu under KVM)  
Date: 12.11.2026

## What I did

### First half — physical components

Identified every component on the bench, photographed it, noted what it connects to and its limits.

1. **Motherboard options**
   - ASUS PRIME B650M-A (AM5, B650 chipset, DDR5, Micro-ATX) — accepts Ryzen 7000/8000/9000 series.
   - MSI PRO B760M-A WIFI DDR4 (LGA 1700, B760 chipset, DDR4) — accepts 12th/13th/14th gen Intel Core.

2. **CPUs**
   - AMD Ryzen 5 7600 (AM5, Zen 4, 6c/12t) — pairs with B650 board + Wraith Stealth cooler.
   - Intel Core i5-13400 (LGA 1700, 10c/16t) — pairs with B760 board + Laminar RM1 cooler.

3. **Coolers**
   - AMD Wraith Stealth (stock, low-profile) — included with many Ryzen 5/7 non-X models.
   - Intel Laminar RM1 (stock aluminium + copper base) — included with boxed 12/13/14 gen non-K CPUs.

4. **Memory**
   - Kingston FURY Beast DDR5-5600 (kit options 16–128 GB) — for AM5 boards.
   - Crucial Pro 16 GB DDR4-3200 UDIMM (CL22) — for the B760 DDR4 board.

5. **Storage**
   - Samsung 990 PRO 1 TB (PCIe 4.0 NVMe M.2) — primary fast drive.
   - Samsung 870 EVO 1 TB (SATA 2.5") — secondary / data drive.

6. **Power**
   - Corsair RM750e (750 W, fully modular, 80+ Gold, ATX 3.1 / PCIe 5.1 ready).

### Second half — same facts from the command line

Read CPU, memory and disk specifications on Windows (host) and on Linux (guest). Built the comparison table below.

## Result

### Component identification (with connection / limit)

| Component | What it is | Connects to / limits |
|-----------|------------|----------------------|
| ASUS PRIME B650M-A | AMD B650 Micro-ATX motherboard | AM5 socket, DDR5, PCIe 5.0 M.2, Ryzen 7000/8000/9000 |
| MSI PRO B760M-A WIFI DDR4 | Intel B760 Micro-ATX motherboard | LGA 1700, DDR4 only, 12/13/14 gen Core, Wi-Fi |
| AMD Ryzen 5 7600 | Desktop CPU (6c/12t, Zen 4) | AM5, requires DDR5, stock Wraith Stealth cooler |
| Intel Core i5-13400 | Desktop CPU (10c/16t, Raptor Lake) | LGA 1700, DDR4 or DDR5 (board dependent), Laminar RM1 |
| Kingston FURY Beast DDR5 | DDR5 UDIMM (up to 5600 MT/s) | AM5 boards only, dual-channel preferred |
| Crucial Pro DDR4-3200 | 16 GB DDR4-3200 UDIMM | LGA 1700 DDR4 boards (e.g. B760) |
| Samsung 990 PRO 1 TB | PCIe 4.0 NVMe M.2 SSD | M.2 slot (x4), ~7000 MB/s class |
| Samsung 870 EVO 1 TB | SATA 2.5" SSD | SATA port, ~560 MB/s |
| Corsair RM750e | 750 W fully modular PSU | 24-pin + CPU 8-pin + PCIe, ATX 3.1 |
| AMD Wraith Stealth | Stock AMD cooler | AM4/AM5 (low profile) |
| Intel Laminar RM1 | Stock Intel cooler | LGA 1700 (12/13/14 gen non-K) |

### Command-line comparison (host Windows + Linux guest)

| Fact | Windows (PowerShell) | Linux | Value on this machine |
|------|----------------------|-------|-----------------------|
| CPU model | `Get-CimInstance Win32_Processor \| Select Name` | `lscpu \| grep "Model name"` | AMD Ryzen 5 7430U with Radeon Graphics |
| Cores / threads | `NumberOfCores`, `NumberOfLogicalProcessors` | `lscpu` (Core(s) per socket × Socket(s), CPU(s)) | 6 cores / 12 threads (Windows); guest shows 2 vCPU |
| Total RAM | `(Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory` | `free -h` | ~16 GB (Windows), 3.3 GiB (guest) |
| RAM modules + speed | `Get-CimInstance Win32_PhysicalMemory` | (guest is virtual) | 2×8 GB DDR4-3200 |
| Disk model / type | `Get-PhysicalDisk` | `lsblk -d -o NAME,MODEL,SIZE,ROTA,TRAN` | WPBSNM8-1TMP NVMe ~1 TB (host); VBOX HARDDISK (guest) |
| Free space (VM volume) | `Get-Volume` (C:) | `df -h /` | ~43 GB free on C: (host); 5.9 G free on / (guest) |
| Firmware | `Get-CimInstance Win32_BIOS` | (not run in guest) | American Megatrends 1.17, 2025-04-24 |

Raw outputs are in `evidence/windows-specs.txt` and `evidence/linux-specs.txt`.

## What did not work the first time

On the Linux guest the `lscpu` output showed only 2 CPUs and the model still reported as “AMD Ryzen 5 7430U”. At first I thought the host was under-reporting cores. It turned out the guest was created with only 2 vCPUs (KVM), so the hypervisor was correctly limiting what the guest could see. Re-checked the Windows side and confirmed the real hardware is 6c/12t. Lesson: always note whether you are looking at the physical host or a virtual machine when comparing numbers.

## Evidence

- [evidence/photos/](evidence/photos/) — photographs of every component with the labels used in the table above
- [evidence/windows-specs.txt](evidence/windows-specs.txt) — raw PowerShell output (CPU, memory, disks, volumes, BIOS)
- [evidence/linux-specs.txt](evidence/linux-specs.txt) — raw `lscpu`, `free -h`, `lsblk`, `df -h` from the Ubuntu guest
