# Ticket 1 — Test Machine for New Developer

| Field          | Details                                      |
|----------------|----------------------------------------------|
| **From**       | Aigerim T. (Office Manager)                  |
| **Subject**    | Test machine for new developer               |
| **Start date** | Monday                                       |
| **Hypervisor** | Oracle VirtualBox                            |

---

## 1. Original Request

> Hi, we have someone starting on Monday and he needs a test computer set up.  
> Nothing special, just so he can try things without breaking anything.  
> Can you write down how to do it so I can do the next one myself?  
> Thanks.

---

## 2. What Was Actually Needed

The request looks simple, but almost every important detail is missing.

### Missing information
- Which operating systems are required (Linux, Windows, or both)
- How much RAM and CPU the host machine has
- Whether internet access is needed
- Whether the developer must test communication between machines
- Whether a snapshot / rollback point is required
- Who will actually follow the instructions (Aigerim has never used a hypervisor)

### Why “one test computer” is not enough
Calling it “a test computer” implies a single machine.  
For a developer who needs to experiment safely, one operating system is rarely sufficient. Real testing almost always requires both a **Linux server** and a **Windows client**. A single machine also cannot demonstrate network isolation or inter-guest communication.

### Assumptions I made
- Host is a typical student laptop (16 GB RAM, 6 cores / 12 threads)
- Developer needs both Ubuntu Server and Windows 11 for cross-platform testing
- Both guests must reach the internet and each other
- A clean snapshot is mandatory so the developer can break things and roll back
- Instructions must be written for a person who has never opened VirtualBox

These assumptions guided every decision below.

---

## 3. What Was Done

1. Measured real host resources (16 GB RAM, 6 cores / 12 threads).
2. Chose **VirtualBox** as the hypervisor.
3. Created two virtual machines:
   - `lab01-ubuntu` — Ubuntu Server 26.04.1 LTS · 2 vCPU · 4 GB RAM · 25 GB dynamic disk
   - `lab02-windows` — Windows 11 Enterprise Evaluation · 2 vCPU · 4 GB RAM · 64 GB dynamic disk · UEFI + Secure Boot + TPM
4. Placed both machines on the same **NAT Network** so they can communicate with each other and still reach the internet.
5. Added a Windows Firewall rule that allows ICMP (ping) without disabling the firewall.
6. Verified connectivity in both directions and to the internet.
7. Created a clean snapshot of the Ubuntu guest named `clean-install`.
8. Wrote a detailed `BUILD.md` that a non-technical person can follow.

### Resource allocation

| Resource | Host has   | Ubuntu     | Windows    | Left for host |
|----------|------------|------------|------------|---------------|
| RAM      | 16 GB      | 4 GB       | 4 GB       | ~8 GB         |
| CPU      | 12 threads | 2 vCPU     | 2 vCPU     | 8 threads     |

Never allocate more than half of the host’s cores to a single guest.  
4 GB per guest leaves enough headroom so the host itself does not start swapping.

### Non-default settings

| Setting          | VirtualBox default           | Chosen value                 | Reason                                                                 |
|------------------|------------------------------|------------------------------|------------------------------------------------------------------------|
| Network mode     | NAT                          | **NAT Network**              | Ordinary NAT isolates guests. NAT Network keeps internet + inter-VM communication. |
| Windows firmware | BIOS                         | **UEFI + Secure Boot + TPM** | Required by Windows 11.                                                |
| Disk allocation  | Pre-allocate full size       | **Dynamically allocated**    | Saves host disk space until the guest actually needs it.               |
| Windows RAM      | 2 GB                         | **4 GB**                     | Windows 11 is unusable under 4 GB for real work.                       |
| Snapshot         | None                         | **clean-install**            | The whole point of a test machine is the ability to roll back.         |

### Alternative considered

**VMware Workstation Player** was also evaluated.  
It is smoother with Windows guests and has better 3D support, but:

- The free Player edition has more restrictions on snapshots and networking.
- VirtualBox is completely free and already installed on most student machines.

VirtualBox was chosen because the goal is a document that Aigerim can follow on a normal office laptop without purchasing anything.

---

## 4. Evidence

| File                                | What it proves                                              |
|-------------------------------------|-------------------------------------------------------------|
| `evidence/01-new-vm-wizard.png`     | Correct creation of the Windows 11 VM                       |
| `evidence/02-disk-settings.png`     | Disk is 64 GB and dynamically allocated                     |
| `evidence/03-network-adapter.png`   | Both guests are attached to the same NAT Network            |
| `evidence/04-guest-uname.txt`       | Ubuntu is running (`uname -a` + `lsb_release -a`)           |
| `evidence/05-ping-between-guests.txt` | Ping works both ways + both machines reach the internet   |
| `evidence/06-snapshot-list.png`     | Snapshot `clean-install` exists and can be restored         |

---

## 5. What I Would Do Differently

I would prepare a one-page **cheat sheet** with only the five most important clicks and the exact numbers (RAM, CPU, disk size).  

The full `BUILD.md` is correct, but for someone in a hurry it is still long. A one-page version would reduce the chance of skipping a cr
