# Lab 2 — Windows Guest & Inter-VM Network

| Field          | Details                                                                                      |
|----------------|----------------------------------------------------------------------------------------------|
| **Student**    | Alzhan Akhmetov                                                                              |
| **Pair**       | Working alone                                                                                |
| **Host**       | Laptop · Windows 11 · AMD Ryzen 5 7430U (6c/12t) · 16 GB RAM                                 |
| **Guests**     | Ubuntu Server 26.04 (`lab01-ubuntu`) + Windows 11 Enterprise Evaluation (`lab02-windows`)    |
| **Date**       | 01.11.2026                                                                                   |
| **Hypervisor** | Oracle VirtualBox                                                                            |

---

## What was done

1. Downloaded the **Windows 11 Enterprise Evaluation** ISO from the official Microsoft Evaluation Center.  
   License: https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise

2. Created the virtual machine `lab02-windows` with the following configuration:
   - 2 vCPUs
   - 4 GB RAM
   - 64 GB dynamically allocated (thin-provisioned) disk
   - UEFI + Secure Boot + virtual TPM 2.0

3. Installed Windows 11 Enterprise Evaluation and VirtualBox Guest Additions.

4. Created a **NAT Network** named `NatNetwork` (`10.0.2.0/24`) and attached **both** guests to it.

5. Obtained IP addresses:
   - Ubuntu (`lab01-ubuntu`) → **10.0.2.3**
   - Windows (`lab02-windows`) → **10.0.2.4**

6. Initial ping from Ubuntu to Windows timed out because Windows Defender Firewall blocks incoming ICMP Echo Requests by default.  
   Fixed by adding a targeted inbound rule (firewall remained enabled):

   ```powershell
   New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
   ```

7. Verified full connectivity in both directions and internet access from both guests.

8. Performed the memory overcommitment experiment: both guests were raised to **10 GB RAM each** (20 GB total requested on a 16 GB host).

---

## Network diagram

```text
                          Internet
                              |
                     Host laptop (16 GB RAM)
                              |
          [VirtualBox NAT Network · NatNetwork · 10.0.2.0/24]
                 /                               \
        lab01-ubuntu                        lab02-windows
        10.0.2.3                            10.0.2.4
        (Ubuntu Server)                     (Windows 11 Enterprise Eval)
```

### Why NAT Network?

Both guests must be able to communicate with each other **and** reach the internet.

| Mode            | Inter-VM communication | Internet access | Notes                                        |
|-----------------|------------------------|-----------------|----------------------------------------------|
| Plain NAT       | No                     | Yes             | Each VM gets an isolated network             |
| Bridged         | Yes                    | Yes             | Often blocked on campus Wi-Fi (MAC filtering)|
| Host-only       | Yes                    | No              | No external connectivity                     |
| **NAT Network** | **Yes**                | **Yes**         | Best choice for this lab                     |

---

## Overcommitment experiment results

| Metric                    | Before (both VMs off) | After (both VMs @ 10 GB) |
|---------------------------|-----------------------|--------------------------|
| **Host RAM in use**       | ~5.5 GB               | 15.0 GB                  |
| **Available RAM**         | ~9.9 GB               | 0.37 GB                  |
| **Committed / Swap used** | ~12 GB                | 30.1 / 31.2 GB           |
| **Observations**          | —                     | Severe lag, mouse stuttering, windows freezing for several seconds. Host became almost unusable until one guest was shut down. |

---

## Issues encountered

### 1. Ubuntu lost internet after switching to NAT Network
- **Symptom:** `ping ubuntu.com` → 100 % packet loss  
- **Fix:** Ran `sudo netplan apply` and performed a full reboot of the Ubuntu guest. Internet access was restored.

### 2. Ping from Ubuntu to Windows timed out
- **Cause:** Windows Defender Firewall blocks inbound ICMPv4 Echo Requests by default.  
- **Solution:** Added a specific inbound rule for ICMPv4 type 8 instead of disabling the firewall.

---

## Evidence

| File                              | Description                                              |
|-----------------------------------|----------------------------------------------------------|
| `evidence/ipconfig-windows.txt`   | Windows IP configuration (10.0.2.4)                      |
| `evidence/ip-addr-linux.txt`      | Ubuntu IP address (10.0.2.3)                             |
| `evidence/ping-both-ways.txt`     | Bidirectional pings + internet connectivity from both VMs |
| `evidence/host-memory-before.png` | Host memory usage with both guests powered off           |
| `evidence/host-memory-after.png`  | Host memory under overcommitment (both guests at 10 GB)  |
