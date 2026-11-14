# Laboratory Work №4: Disk Partitioning and Software RAID 1 Management

**Student:** Akhmetov  
**Environment:** Oracle VirtualBox (Isolated Linked Clones: `akhmetov-ubuntu-lab4-v2`, `akhmetov-win-lab4`)

---

## Part 1: Linux Disk Partitioning & Software RAID 1 (`mdadm`)

1. **Disk Preparation & GPT Partitioning:**
   - Attached three 1 GB virtual SATA disks (`/dev/sdb`, `/dev/sdc`, `/dev/sdd`) to the Ubuntu virtual machine.
   - Initialized GPT partition tables (`mklabel gpt`) and created primary RAID partitions (`1MiB` to `100%`) with the `raid` flag enabled using `parted`.
   - Block device layout saved to `evidence/01-lsblk.txt`[cite: 12].

2. **RAID 1 Array Creation & Mounting:**
   - Created a mirrored RAID 1 array `/dev/md0` using `mdadm --create` with two active devices (`/dev/sdb1`, `/dev/sdc1`) and one hot-spare device (`/dev/sdd1`).
   - Formatted `/dev/md0` with `ext4`, mounted it to `/mnt/raid1`, and wrote test verification data.
   - Initial healthy array state and configuration saved to `evidence/02-mdstat-healthy.txt` and `evidence/03-mdadm-detail.txt`[cite: 12].

3. **Fault Simulation & Automatic Rebuild:**
   - Simulated a disk failure by marking `/dev/sdc1` as faulty (`mdadm --manage /dev/md0 --fail /dev/sdc1`) and removing it from the array.
   - Verified degraded state (`evidence/04-mdstat-degraded.txt`)[cite: 12].
   - Confirmed automatic array synchronization onto the hot-spare drive `/dev/sdd1` (`evidence/05-mdstat-rebuilt.txt`) and verified zero data loss (`evidence/06-data-check.txt`)[cite: 12].

---

## Part 2: Windows Disk Partitioning & Volume Formatting (PowerShell)

1. **GPT Initialization & Partitioning:**
   - Attached a 1 GB virtual disk (`Disk 1`) to the Windows virtual machine.
   - Initialized the disk with a `GPT` partition style (`Initialize-Disk -Number 1 -PartitionStyle GPT`).
   - Created two partitions: a 500 MB volume formatted as `NTFS` (`Lab04_NTFS`) and a remaining-space volume formatted as `FAT32` (`Lab04_FAT32`).

2. **Verification & Evidence Collection:**
   - Wrote a timestamped verification file (`test.txt`) to the newly created `NTFS` volume.
   - Exported disk, partition, and volume configuration tables via PowerShell to `evidence/win-01-disks.txt`, `evidence/win-02-partitions.txt`, and `evidence/win-03-volumes.txt`[cite: 12].