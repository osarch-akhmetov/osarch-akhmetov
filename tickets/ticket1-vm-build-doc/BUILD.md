# Ticket 1 — Virtual Machine Build & Verification Guide

**Author:** Akhmetov  
**Host System:** MSI Stealth 17M A12UE (Windows 11, Intel Core i7, NVMe SSD)  
**Hypervisor:** Oracle VirtualBox 7.1  
**Evidence Directory:** `evidence/`

---

## 1. Host & Hypervisor Preparation

1. Verified hardware virtualization (`VT-x` / Intel Virtualization Technology) is enabled in UEFI/BIOS and active in Windows Task Manager.
2. Disabled conflicting Windows Hyper-V hypervisor components to ensure native VirtualBox execution speed (avoiding slow software emulation fallback).
3. Configured default VM storage path on the host NVMe SSD (`C:\Users\user\VirtualBox VMs`).

---

## 2. Primary Guest VM: Ubuntu Server 24.04.3 LTS (`lab01-ubuntu`)

### 2.1 VM Creation & Hardware Allocation
* **VM Name:** `lab01-ubuntu`
* **OS Type / Version:** Linux / Ubuntu (64-bit)
* **Base Memory:** 2048 MB (2 GB RAM)
* **Processors:** 2 vCPUs (PAE/NX enabled)
* **Virtual Storage:** 25 GB dynamically allocated VDI disk (`lab01-ubuntu.vdi`) attached to SATA AHCI controller *(see `evidence/01-new-vm-wizard.png` and `evidence/02-disk-settings.png`)*.
* **Installation Media:** `ubuntu-24.04.3-live-server-amd64.iso` mounted on IDE Optical Drive.

### 2.2 Guest OS Installation & User Setup
* **Hostname:** `lab01-akhmetov`
* **Primary User:** `akhmetov` (with `sudo` privileges)
* **Storage Layout:** Guided LVM partitioning across the entire 25 GB virtual disk (`/` formatted as `ext4`).
* **Installed Packages:** OpenSSH Server (`openssh-server`) enabled during installation for headless host-to-guest administration.
* **Kernel & Release Verification:** Recorded `uname -a` and `lsb_release -a` output confirming Linux kernel `6.8.0-151-generic` on Ubuntu 24.04.3 LTS *(see `evidence/04-guest-uname.txt`)*.

---

## 3. Secondary Peer Guest & Virtual Networking

### 3.1 Network Configuration (`NAT` & `NatNetwork`)
* **Adapter 1:** Intel PRO/1000 MT Desktop (`82540EM`), attached to **NAT** for standard host-to-guest SSH access via port forwarding (`127.0.0.1:2222 -> 10.0.2.15:22`) *(see `evidence/03-network-adapter.png`)*.
* **Inter-Guest Networking:** Created a shared **NAT Network** (`NatNetwork`, IPv4 prefix `10.0.2.0/24`, DHCP enabled) to test peer-to-peer communication between two guest instances alongside outbound internet access.

### 3.2 Connectivity & ICMP Verification
* Assigned static/DHCP addresses within `10.0.2.0/24` (`10.0.2.3` on `lab01-akhmetov` and `10.0.2.4` on `lab02-akhmetov`).
* Verified bidirectional ICMP reachability between both guests (`0% packet loss`, average RTT ~1.0 ms) and outbound internet resolution/ping to `ubuntu.com` *(see `evidence/05-ping-between-guests.txt`)*.

---

## 4. State Management & Snapshots

1. Powered down `lab01-ubuntu` cleanly after initial OS setup, SSH hardening, and system updates.
2. Created a baseline recovery snapshot (`Clean Install` / baseline state) in VirtualBox Snapshot Manager to allow instant rollback before lab experiments *(see `evidence/06-snapshot-list.png`)*.

---

## 5. Evidence Index

| File | Description |
| :--- | :--- |
| `evidence/01-new-vm-wizard.png` | VirtualBox New VM wizard showing name, ISO, and OS selection |
| `evidence/02-disk-settings.png` | Virtual hard disk size (25 GB VDI) and SATA controller configuration |
| `evidence/03-network-adapter.png` | Network adapter settings (`NAT` / `NatNetwork` and port forwarding) |
| `evidence/04-guest-uname.txt` | Guest kernel (`uname -a`) and Ubuntu release (`lsb_release -a`) output |
| `evidence/05-ping-between-guests.txt` | Bidirectional ping logs between guests and external internet (`ubuntu.com`) |
| `evidence/06-snapshot-list.png` | VirtualBox Snapshot Manager showing clean baseline snapshot |
