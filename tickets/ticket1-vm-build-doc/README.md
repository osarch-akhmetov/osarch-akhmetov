# Ticket 1 — Virtual Machine Build & Network Verification

This directory contains the reproducible build documentation and verification artifacts for **Ticket 1** (Operating Systems Administration).

## Summary of Environment

* **Host:** MSI Stealth 17M A12UE (Windows 11, VirtualBox 7.1)
* **Primary Guest:** `lab01-ubuntu` (Hostname: `lab01-akhmetov`, User: `akhmetov`, Ubuntu Server 24.04.3 LTS)
* **Resources:** 2 vCPUs, 2048 MB RAM, 25 GB VDI storage
* **Networking:** VirtualBox NAT (SSH port forwarding `2222 -> 22`) and `NatNetwork` (`10.0.2.0/24`) for inter-guest ICMP verification

## Structure

* [`BUILD.md`](./BUILD.md) — Step-by-step build instructions, hardware allocation, network setup, and verification notes.
* [`evidence/`](./evidence/) — Raw screenshots and terminal logs captured directly from the host and `akhmetov@lab01-akhmetov` guest environment.
