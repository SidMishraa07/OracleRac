📂 Chapter 6: Oracle ASM Fundamentals

# 📖 Subtopic 6.1 – What is ASM?

ASM = Automatic Storage Management.  
It is Oracle’s volume manager + file system for databases.  

✅ Purpose:  
- Manages database files directly.  
- Provides striping & mirroring automatically.  
- Eliminates need for 3rd-party volume managers (like Veritas, LVM).  
- Optimized for Oracle database I/O.

👉 Think of ASM as the **“DBA’s Storage Assistant”**.  

---

## 🚀 Real-Time Example: Raju’s Discovery  

Raju was tired of managing dozens of datafiles manually.  
One day his senior DBA told him:  

> “Raju, why are you worrying about which disk has space? Just throw all disks into ASM, and ASM will handle it.”  

Raju realized:  
- No more manually balancing data across disks.  
- If one disk fills up, ASM rebalances automatically.  
- If a disk fails, mirroring ensures data still survives.  

---

## 🎯 Key Takeaways  

- ASM is mandatory for RAC environments.  
- ASM handles: striping, mirroring, rebalancing.  
- Simplifies DBA life.  
- Provides high performance + fault tolerance.  

###########################################################################

# 🏗️ Subtopic 6.2 – ASM Architecture

ASM has two components:  

1. **ASM Instance**  
   - Special lightweight Oracle instance.  
   - Manages metadata about storage (not user data).  
   - Background processes (RBAL, ARBn, etc.).  

2. **ASM Disk Groups**  
   - Logical collections of disks.  
   - Database files are striped and mirrored here.  
   - DBA interacts at diskgroup level (not raw devices).  

---

## 🔧 Scenario: Raju Configures ASM

Raju creates a diskgroup `+DATA` with 3 disks.  
- When he creates a tablespace → file goes into `+DATA`.  
- ASM stripes data across all 3 disks for performance.  
- If one disk fails, mirroring kicks in.  

---

## 🎯 Key Takeaways  

- ASM Instance ≠ Database Instance.  
- ASM Diskgroup = DBA’s unit of storage.  
- Databases connect to ASM via Oracle Clusterware.  

###########################################################################

# 💽 Subtopic 6.3 – ASM Disk Groups

## Types of ASM Disk Groups  

1. **External Redundancy**  
   - No mirroring (depends on storage hardware).  
   - Cheap but risky.  

2. **Normal Redundancy**  
   - Two-way mirroring.  
   - Each extent stored twice.  

3. **High Redundancy**  
   - Three-way mirroring.  
   - Best fault tolerance.  

---

## 📊 Real-Life Story  

Raju sets up `+DATA` with **Normal Redundancy**.  
One disk crashes.  
ASM automatically serves data from mirror copies → no downtime.  
Later, when Raju adds a replacement disk, ASM rebalances in background.  

---

## 🎯 Key Takeaways  

- Diskgroup = Collection of disks.  
- Redundancy levels decide fault tolerance.  
- ASM automatically rebalances when disks are added/removed.  

###########################################################################

# ⚖️ Subtopic 6.4 – ASM Rebalancing

When disks are added/removed, ASM automatically redistributes extents.  

- Performed by **ARBn** processes.  
- Online (database remains available).  
- Controlled by **ASM_POWER_LIMIT** parameter.  

---

## ⚙️ Scenario: Raju Adds Storage  

His `+DATA` group is running out of space.  
He adds 2 new disks.  

ASM instantly begins rebalancing.  
- Old + new disks share the load equally.  
- Users notice **zero downtime**.  

---

## 🎯 Key Takeaways  

- Rebalancing = automatic + online.  
- ASM_POWER_LIMIT controls speed vs overhead.  
- DBA doesn’t need manual file copy.  

###########################################################################

# ⚔️ Subtopic 6.5 – ASM vs Traditional Filesystem

| Feature            | ASM                       | Filesystem (ext4, NTFS, etc.) |
|--------------------|---------------------------|-------------------------------|
| Striping           | Automatic                | Manual / none                 |
| Mirroring          | Built-in                 | Hardware/OS dependent         |
| Rebalancing        | Automatic                | Manual (painful)              |
| Performance        | Optimized for DB I/O     | General-purpose               |
| Management         | DBA friendly             | Storage admin needed          |

---

## 🎬 Raju’s Debate with Storage Admin  

Storage Admin:  
> “We already have LVM, why use ASM?”  

Raju:  
> “LVM doesn’t rebalance online, doesn’t optimize for Oracle workloads, and requires manual intervention. ASM is tailor-made for Oracle DB.”  

---

## 🎯 Key Takeaways  

- ASM > Filesystem for Oracle DB workloads.  
- No need for 3rd-party tools like Veritas.  
- High performance, simple, fault-tolerant.  

###########################################################################

# 🤝 Subtopic 6.6 – ASM and RAC Integration

In RAC:  
- ASM runs on each node.  
- Only one ASM instance manages the storage.  
- All RAC DB instances connect to ASM to access shared disks.  

---

## 🖥️ Scenario: Raju’s 2-Node RAC

Node1 + Node2 both use ASM.  
If Node1 crashes:  
- Node2 still sees disks via ASM.  
- No data corruption.  

Clusterware + ASM = heart of RAC.  

---

## 🎯 Key Takeaways  

- ASM is cluster-aware.  
- RAC instances depend on ASM to access shared storage.  
- Without ASM, RAC cannot survive.  

###########################################################################

# 🛠️ Subtopic 6.7 – ASM Administration

Tools for managing ASM:  

1. **SQL*Plus**  
   - `ALTER DISKGROUP` commands.  

2. **ASMCA (ASM Configuration Assistant)**  
   - GUI-based.  

3. **ASMCMD**  
   - Command-line tool like UNIX shell.  

---

## 🔍 Example Commands  

```sql
SQL> CREATE DISKGROUP DATA NORMAL REDUNDANCY DISK '/dev/sd*';

SQL> ALTER DISKGROUP DATA ADD DISK '/dev/sdf1';

ASMCMD> ls
ASMCMD> du +DATA

###########################################################################