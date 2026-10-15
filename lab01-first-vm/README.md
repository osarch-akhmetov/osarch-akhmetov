# Lab 1 — Your first virtual machine

Pair: Akhmetov / Partner   Driver first half: Akhmetov
Machine: Host (Windows 11), guest Ubuntu Server 26.04 LTS
Date: 15.10.2026

## What I did
1. Installed VirtualBox hypervisor. Licence: https://www.virtualbox.org/wiki/Licensing_FAQ
2. Created VM lab01-ubuntu: 2 vCPU, 2 GB RAM, 25 GB dynamic disk, NAT.
3. Installed Ubuntu Server, took snapshot clean-install.
4. Broke it: deleted `/etc/hostname` using `sudo rm /etc/hostname` and rebooted.
5. Rolled back to clean-install: restored snapshot via Snapshots manager and verified system recovery.

## Resource justification
I allocated 2 GB of RAM and 2 vCPUs out of my host's 16 GB RAM and 8 CPU cores. The host OS consumes around 6 GB at idle, leaving 10 GB free. Allocating 2 GB leaves sufficient capacity for running host tasks smoothly and reserves memory for a second guest VM required in Lab 2.

## Result
The VM boots; the break-and-rollback cycle is shown in the evidence below.

## What did not work the first time
When trying to run SSH command from PowerShell to generate `guest-release.txt`, connection failed with `DirectoryNotFoundException` because the destination folder `labs/lab01-first-vm/evidence` was not created yet. Created the folder structure manually using `New-Item` command and re-ran the SSH command successfully.

## Evidence
- [evidence/vm-settings.png](evidence/vm-settings.png) — CPU, memory and disk as configured
- [evidence/guest-release.txt](evidence/guest-release.txt) — Ubuntu Server is running in the guest
- [evidence/broken.png](evidence/broken.png) — the guest in its broken state
- [evidence/snapshot-list.png](evidence/snapshot-list.png) — the snapshot, after the rollback
