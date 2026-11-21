# Lab 6 — Live processes and memory pressure

## Goal
Examine a live process across Windows Task Manager, Sysinternals Process Explorer, and Linux `htop`/`ps`, reconcile the different memory and handle numbers they report, and observe how Linux handles severe memory pressure and invokes the kernel OOM killer.

---

## Part 1: Same process, three tools

### Chosen common process
Because `VBoxService` was not running inside the guest VM, a clean Python HTTP server (`python -m http.server 8000` on Windows 11 host and `python3 -m http.server 8000` on Ubuntu 24.04 guest) was used as the common process across both operating systems.

### Comparison Table

| Metric | Task Manager (`python.exe`) | Process Explorer (`python.exe`) | `htop` / `ps` (`python3`) |
| :--- | :--- | :--- | :--- |
| **PID** | `3204` | `3204` | `1800` |
| **Parent (name and PID)** | — *(not shown in Details view)* | `powershell.exe` (`8592`) | `-bash` (`1797`) |
| **Default "Memory" column** | `11 260 K` *(Private working set)* | `12 660 K` *(Private Bytes)* | `20 296 KiB` *(RES)* |
| **Private memory** | `11 260 K` *(Memory - private working set)* | `10 276 K` *(WS Private)* / `12 660 K` *(Private Bytes)* | `9 416 KiB` *(RES − SHR = 20296 − 10880)* |
| **Working set / RES** | `22 544 K` *(Working set)* | `22 020 K` *(Working Set)* | `20 296 KiB` *(RES / RSS)* |
| **Virtual size / Commit** | `12 696 K` *(Commit size)* | `12 660 K` *(Private Bytes)* | `29 244 KiB` *(VIRT / VSZ)* |
| **Handles / file descriptors** | `155` | `155` | `4` *(FDs in `/proc/1800/fd`)* |
| **Threads** | `2` | `1` *(at snapshot time)* | `1` *(`NLWP`)* |

### Key Takeaways (Part 1)
1. **Why default "Memory" numbers differ between Task Manager, Process Explorer, and `htop`:**
   Each tool highlights a different aspect of process memory by default. Windows Task Manager displays the **Private Working Set** (`11 260 K` — physical RAM exclusively owned by `python.exe`, excluding shared DLLs) to reflect the immediate physical RAM impact of closing an app. Process Explorer defaults to **Private Bytes** (`12 660 K` — total private virtual memory committed to the process in RAM and pagefile). Linux `htop` displays **Resident Set Size (`RES` = `20 296 KiB`)**, which includes both private anonymous pages and shared libraries (`SHR` = `10 880 KiB`) mapped into RAM, matching Windows **Working Set** (`22 020 K`–`22 544 K`) much more closely than private memory.
2. **Why `VIRT` (`29 244 KiB`) is larger than `RES` (`20 296 KiB`):**
   `VIRT` (`VSZ`) counts the entire virtual address space mapped by the process — including shared C/Python runtime libraries, read-only code segments, and reserved heap/stack regions that have not been faulted into physical RAM yet. `RES` counts only the pages currently backed by physical RAM frames.
3. **Why Windows reports `155` handles while Linux reports `4` file descriptors:**
   In Linux, file descriptors (`0` stdin, `1` stdout, `2` stderr, `3` listening TCP socket) represent only open files, pipes, and sockets in the process's FD table; shared libraries are memory-mapped (`mmap`) rather than kept as open FDs. In Windows, the kernel **Object Manager** tracks every kernel object reference in the process handle table — including registry keys (`HKLM\...`, `HKCU`), synchronization primitives (events, mutants, semaphores), desktop/window station objects, directory objects (`\KnownDlls`), and device handles (`\Device\Afd`, `\Device\ConDrv`), resulting in `155` open handles even for a minimal CLI server.

### Evidence (Part 1)
* `evidence/task-manager.png` — Windows Task Manager Details tab showing `python.exe` (PID `3204`).
* `evidence/process-explorer.png` — Sysinternals Process Explorer showing `python.exe` (PID `3204`) under parent `powershell.exe` (PID `8592`), memory columns, and lower-pane handle table.
* `evidence/htop.png` — Linux `htop` showing `python3 -m http.server 8000` (PID `1800`).

---

## Part 2: Memory Pressure and the OOM Killer

### Baseline and Snapshot
* Created VirtualBox snapshot **`before-lab6`** prior to running memory stress tests.
* Baseline memory (`free -h` at `10:59:11 UTC`): **`3.8 GiB`** physical RAM (`435 MiB` used, `812 MiB` buff/cache, **`3.4 GiB` available**) and **`3.8 GiB`** swap (`0 B` used).

### Memory Progression (`evidence/memory-pressure.txt`)

| Phase | Timestamp (UTC) | Mem Total | Mem Used | Mem Free | Buff/Cache | Mem Available | Swap Used |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Before** | `10:59:11` | `3.8Gi` | `435Mi` | `2.8Gi` | `812Mi` | `3.4Gi` | `0B` |
| **During (Peak)** | ~`11:05` | `3.8Gi` | `3.8Gi` | `97Mi` | `77Mi` | **`16Mi`** | `61Mi`+ |
| **After** | `11:12:46` | `3.8Gi` | `330Mi` | `3.6Gi` | `40Mi` | **`3.5Gi`** | `61Mi` |

### Observations Under Pressure
1. **Page cache reclamation first:** As `stress-ng --vm` allocated memory (`70%` ➔ `90%` ➔ `120%` ➔ `200%`), `buff/cache` shrunk aggressively from `812 MiB` down to `72–77 MiB` before anonymous pages were pushed out, dropping `available` memory from `3.4 GiB` to **`16 MiB`**.
2. **Responsiveness & swap behavior:** During peak allocation, `stress-ng` reported `metrics-check: all bogo-op counters are zero` and took `95.96s` for a `90s` run due to heavy page-reclaim stalls and swap I/O (`61 MiB` moved to `/swap.img` and remained swapped out after the run until touched).
3. **Kernel OOM killer invocation (`evidence/kernel-log-extract.txt`):**
   Because the `3.8 GiB` swap file initially absorbed gradual allocations, a targeted test (`sudo swapoff -a && stress-ng --vm 2 --vm-bytes 110% --vm-keep --timeout 20s; sudo swapon -a`) was run to exhaust physical memory and trigger the kernel OOM killer cleanly:
   * At `11:16:49`, `stress-ng-vm` (PID `2833`) invoked `oom-killer` (`gfp_mask=0x140dca`, `order=0`).
   * The kernel selected and killed **`stress-ng-vm` (PID `2831`)** (`total-vm:2116100kB`, `anon-rss:2048348kB`, `oom_score_adj:1000`) and then **PID `2832`** (`anon-rss:2048476kB`) at `11:16:51`, immediately reclaiming ~`2 GiB` of anonymous RSS per worker while keeping the SSH session (`sshd`) and system services alive.

### Windows Event Log Check
Checked the Windows `System` event log for `Microsoft-Windows-Resource-Exhaustion-Detector` (Event ID `2004`):
