Ticket 1 — New developer starts Monday, needs a test machine
1. What was asked
From: Aigerim T. (Office Manager)
Subject: test machine for new dev

Hi, we have someone starting on Monday and he needs a test computer
set up. Nothing special, just so he can try things without breaking
anything. Can you write down how to do it so I can do the next one
myself? Thanks

2. What was actually needed
The request sounds simple, but it leaves out almost everything required for a usable test environment.

What the request left out:

Which operating systems are needed (Linux, Windows, or both)
How much RAM and CPU the host machine actually has
Whether the machines need internet access
Whether the developer needs to test communication between machines
Whether a snapshot / rollback point is required
Who will actually follow the instructions (Aigerim has never opened a hypervisor)
Where the request itself was wrong:
Calling it “a test computer” implies one machine. For a developer who needs to “try things without breaking anything”, a single OS is not enough. Real testing almost always needs both a Linux server and a Windows client. One machine also cannot demonstrate network isolation or inter-guest communication.

What I assumed instead of asking:

The host is a typical student laptop (mine has 16 GB RAM and 6 cores / 12 threads)
The developer needs both Ubuntu Server and Windows 11 so he can test cross-platform behaviour
Both guests must reach the internet and each other
A clean snapshot is mandatory so the developer can break things and roll back
The instructions must be written for someone who has never used VirtualBox
These assumptions shaped every decision below.

3. What I did
Measured the real host resources (16 GB RAM, 6 cores / 12 threads).
Chose VirtualBox as the hypervisor (see Decisions below).
Created two virtual machines:
lab01-ubuntu — Ubuntu Server 26.04.1 LTS, 2 vCPU, 4 GB RAM, 25 GB dynamic disk
lab02-windows — Windows 11 Enterprise Evaluation, 2 vCPU, 4 GB RAM, 64 GB dynamic disk, UEFI + Secure Boot + TPM
Put both machines on the same NAT Network so they can talk to each other and still reach the internet.
Added a Windows Firewall rule that allows ICMP (ping) without disabling the firewall.
Verified connectivity in both directions and to the internet.
Took a clean snapshot of the Ubuntu guest named clean-install.
Wrote a detailed BUILD.md that a non-technical person can follow.
Resource calculation (from my real host)
Resource	Host has	Given to Ubuntu	Given to Windows	Left for host
RAM	16 GB	4 GB	4 GB	~8 GB
CPU	12 threads	2 vCPU	2 vCPU	8 threads
I never give more than half of the host’s cores to any single guest. 4 GB per guest leaves enough headroom so the host itself does not start swapping.

Non-default settings (and why)
Setting	Default in VirtualBox	What I chose	Why
Network mode	NAT	NAT Network	Ordinary NAT isolates each guest; they cannot ping each other. NAT Network keeps internet access and lets the two guests talk.
Windows firmware	BIOS	UEFI + Secure Boot + TPM 2.0	Windows 11 requires these three things.
Disk allocation	Pre-allocate full size	Dynamically allocated	Saves host disk space until the guest actually needs it.
Windows RAM	2 GB (VirtualBox suggestion)	4 GB	Windows 11 is unusable under 4 GB for any real work.
Snapshot	None	clean-install	The whole point of a test machine is the ability to roll back after the developer breaks something.
Alternative considered
I also considered VMware Workstation Player.
It is smoother with Windows guests and has better 3D support, but:

The free Player version has more restrictions on snapshots and networking.
VirtualBox is completely free for this use case and is already installed on most student machines.
I stayed with VirtualBox because the goal is a document that Aigerim can follow on a normal office laptop without buying anything.

4. Evidence
File	What it proves
evidence/01-new-vm-wizard.png	Correct creation of the Windows 11 VM with the right type and version
evidence/02-disk-settings.png	Disk is 64 GB and dynamically allocated
evidence/03-network-adapter.png	Both guests are attached to the same NAT Network
evidence/04-guest-uname.txt	Ubuntu is running (uname -a + lsb_release -a)
evidence/05-ping-between-guests.txt	Ping works both directions + both machines reach the internet
evidence/06-snapshot-list.png	Snapshot clean-install exists and can be restored
5. What I would do differently
I would prepare a one-page “cheat sheet” with only the five most important clicks and the exact numbers (RAM, CPU, disk size). The full BUILD.md is correct, but for a person who is in a hurry it is still long. A one-page version would reduce the chance of skipping a critical step.
