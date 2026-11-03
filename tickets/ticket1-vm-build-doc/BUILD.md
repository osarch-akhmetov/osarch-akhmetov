# How to Build the Test Machines for a New Developer

These instructions create a complete, isolated test environment consisting of two virtual machines:

- **Ubuntu Server** (Linux)
- **Windows 11 Enterprise Evaluation**

Both machines can communicate with each other and have full internet access.  
A clean snapshot is included so the developer can freely experiment and roll back at any time.

### Requirements
- Approximately **40–50 GB** of free disk space
- Host machine with at least **8 GB RAM** (16 GB recommended)

---

## Step 1. Install VirtualBox

1. Open https://www.virtualbox.org/wiki/Downloads
2. Download the latest version for **Windows hosts**
3. Install using all default options
4. Restart the computer if the installer requests it

> **Why VirtualBox?**  
> It is completely free, runs reliably on ordinary Windows laptops, and does not require any paid licence.  
> VMware Workstation Player was considered, but VirtualBox is simpler and more suitable for this use case.

---

## Step 2. Download the Operating System Images

You need two ISO files:

1. **Ubuntu Server**  
   Download the latest LTS version:  
   https://ubuntu.com/download/server

2. **Windows 11 Enterprise Evaluation**  
   Go to the Microsoft Evaluation Center, complete the short form, and download the 64-bit ISO:  
   https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise

Save both files in your **Downloads** folder.  
Do **not** place them inside any project directory.

---

## Step 3. Create the Ubuntu Virtual Machine

1. Open VirtualBox → click **New**
2. Name: `lab01-ubuntu`
3. Type: **Linux** → **Ubuntu (64-bit)**
4. Memory size: **4096 MB**
5. Processors: **2**
6. Hard disk: **Create a virtual hard disk now** → **25 GB** → **Dynamically allocated**
7. Click **Finish**

> **Why these values?**  
> The host typically has 16 GB of RAM and 12 threads. Allocating 4 GB RAM and 2 vCPUs to the guest leaves sufficient resources for the second virtual machine and for the host operating system itself.

---

## Step 4. Create the Windows Virtual Machine

1. Click **New** again
2. Name: `lab02-windows`
3. Type: **Microsoft Windows** → **Windows 11 (64-bit)**
4. Enable **Skip Unattended Installation**
5. Memory size: **4096 MB**
6. Processors: **2**
7. Hard disk: **64 GB**, **Dynamically allocated** (do **not** select “Pre-allocate Full Size”)
8. Click **Finish**

After creation, open **Settings → System** and verify the following:

- **Enable EFI** is checked
- **TPM** is set to **v2.0**
- **Secure Boot** is enabled

These three options are mandatory for Windows 11. Without them the installation will fail.

---

## Step 5. Connect Both Machines to the Same Network

1. In VirtualBox go to **File → Tools → Network Manager**
2. Open the **NAT Networks** tab
3. Create a new network named `NatNetwork` (enable DHCP if it is not already enabled)
4. For **each** virtual machine:
   - Open **Settings → Network**
   - Adapter 1 → Attached to: **NAT Network**
   - Name: **NatNetwork**

> **Why NAT Network instead of ordinary NAT?**  
> Standard NAT places every virtual machine on its own isolated network, so they cannot communicate.  
> NAT Network puts both guests on the same internal network while still providing internet access.

---

## Step 6. Install Ubuntu Server

1. Start `lab01-ubuntu`
2. Select the Ubuntu Server ISO when prompted
3. Language → **English**
4. Keyboard configuration → press **Enter**
5. Network screen → leave defaults (DHCP) and continue
6. Guided storage configuration → leave defaults and continue
7. Confirm the destructive action (this is a virtual disk — it is safe)
8. Create a username and password (write them down)
9. When asked about SSH → enable **Install OpenSSH server**
10. Skip any additional snaps
11. Wait for installation to complete → select **Reboot Now**
12. After reboot, log in with the credentials you created

Verify the system:

```bash
uname -a
lsb_release -a
```

You should see the Ubuntu version and kernel information.

---

## Step 7. Install Windows 11

1. Start `lab02-windows`
2. Immediately press any key when you see “Press any key to boot from CD or DVD…”
3. Choose language and keyboard → **Next**
4. Click **I don’t have a product key**
5. Select **Windows 11 Enterprise Evaluation**
6. Accept the licence terms
7. Choose **Custom: Install Windows only**
8. Select the virtual disk and click **Next**
9. Wait for installation and the automatic reboots
10. When asked to sign in with a Microsoft account, choose **Domain join instead** (or “Offline account”)
11. Create a local username and password
12. Skip privacy options as much as possible
13. After the desktop appears, install **Guest Additions**:
    - VirtualBox menu → **Devices → Insert Guest Additions CD image**
    - Run the installer inside Windows and reboot when finished

---

## Step 8. Verify Connectivity

### On Ubuntu

```bash
ip -br a
ping -c 4 10.0.2.4          # should reach Windows
ping -c 4 ubuntu.com        # should reach the internet
```

### On Windows (PowerShell)

```powershell
ipconfig
ping 10.0.2.3               # should reach Ubuntu
ping ubuntu.com             # should reach the internet
```

> **Firewall note**  
> If the ping from Ubuntu to Windows fails, open **PowerShell as Administrator** on Windows and run:

```powershell
New-NetFirewallRule -DisplayName "Allow ping in (ICMPv4)" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow
```

Then try the ping again.

---

## Step 9. Create a Clean Snapshot

1. Completely shut down the Ubuntu machine (do not pause it)
2. In VirtualBox select `lab01-ubuntu`
3. Click the **Snapshots** button (camera icon)
4. Click **Take**
5. Name it exactly: `clean-install`
6. Click **OK**

### How to roll back later

If the developer breaks the Ubuntu machine:

1. Shut down the virtual machine
2. Open the **Snapshots** view
3. Select `clean-install`
4. Click **Restore**
5. Confirm

The machine will return to the exact state it was in when the snapshot was taken.

---

## Quick Reference

| Setting      | Ubuntu              | Windows 11                     |
|--------------|---------------------|--------------------------------|
| **vCPU**     | 2                   | 2                              |
| **RAM**      | 4 GB                | 4 GB                           |
| **Disk**     | 25 GB (dynamic)     | 64 GB (dynamic)                |
| **Network**  | NAT Network         | NAT Network                    |
| **Firmware** | Default             | UEFI + Secure Boot + TPM 2.0   |
| **Snapshot** | `clean-install`     | —                              |

That is everything required to rebuild the complete test environment from scratch.
