# Lab 1 — Your first virtual machine

Pair: Akhmetov / Partner   Driver first half: Akhmetov
Machine: MSI Stealth 17M A12UE (Windows 11), guest Ubuntu Server 24.04.3 LTS
Date: 20.11.2026

## What I did
1. Installed Oracle VirtualBox 7.1. Licence: https://www.virtualbox.org/wiki/Licensing_FAQ
2. Created VM `lab01-ubuntu`: 2 vCPU, 4 GB (4096 MB) RAM, 25 GB dynamic disk, NAT (with port forwarding 2222 -> 22).
3. Installed Ubuntu Server 24.04.3 LTS, powered off the VM, and took snapshot `clean-install`.
4. Broke it (Option A): verified `cat /etc/hostname` (`lab01-akhmetov`), deleted `/etc/hostname` with `sudo rm /etc/hostname`, rebooted, and confirmed with `hostname` and `cat /etc/hostname` (`No such file or directory`).
5. Rolled back to `clean-install`: restored the snapshot in VirtualBox, booted the VM, and verified with `cat /etc/hostname` that `lab01-akhmetov` was restored.

## Resource justification
I allocated 4 GB (4096 MB) of my host's 16 GB total RAM (25%) because Windows 11 and background apps already use about 6–7 GB at idle, and leaving 12 GB free ensures smooth host performance without swapping while leaving room to run a second Windows guest alongside it in Lab 2. For CPU, I assigned 2 vCPUs out of my Intel Core i7-1280P's 14 physical cores (20 logical threads), which is well under half of the host cores and gives Ubuntu Server responsive CLI execution without starving Windows. A 25 GB dynamically allocated virtual disk easily fits within my host's 795.67 GB of free NVMe space (from Lab 0) while only consuming actual written blocks.

## Result
The VM boots; the break-and-rollback cycle is shown in the evidence below.

## What did not work the first time
- On the first installation attempt, VirtualBox showed a green turtle icon in the status bar due to Windows Hyper-V/Memory Integrity conflict, causing slow emulation and a `curthooks` kernel installation error. Disabling Memory Integrity and `hypervisorlaunchtype`, rebooting the host, and installing without LVM resolved the issue cleanly.
- In the initial submission, the justification referenced mismatched host specs instead of the actual laptop inventory from Lab 0, and the break/rollback cycle lacked `cat /etc/hostname` before/after proof inside the guest.

## Evidence
- [evidence/vm-settings.png](evidence/vm-settings.png) — CPU, memory and disk as configured
- [evidence/guest-release.txt](evidence/guest-release.txt) — Ubuntu Server is running in the guest
- [evidence/broken.png](evidence/broken.png) — the guest in its broken state
- [evidence/snapshot-list.png](evidence/snapshot-list.png) — the snapshot, after the rollback
