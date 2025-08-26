# 📖 Chapter 3 – Voting Disks & Quorum (incl. Quorum Disks)

## 🔹 Subtopic 1: What is a Voting Disk?

A **Voting Disk** is the **heartbeat disk** of Oracle RAC.  
It is a shared file that stores the cluster **membership information** — i.e., which nodes are alive, which nodes are dead, and who stays in the cluster.

👉 Without Voting Disks, nodes **cannot communicate or agree** on cluster membership.  
This leads to *split-brain* scenarios, which can corrupt data if not managed.

---

## 🔹 Subtopic 2: Role of Voting Disks

1. **Membership Tracking**
   - Every node regularly writes its heartbeat to the Voting Disk.
   - If a node fails to update within a timeout → Clusterware assumes it is dead.

2. **Quorum / Majority Rule**
   - To avoid split-brain, nodes must have access to a majority of the Voting Disks.
   - Example: If there are 3 Voting Disks, a node must access at least 2.

3. **Failure Handling**
   - If a node loses contact with majority Voting Disks → it gets evicted.

---

## 🔹 Subtopic 3: Real-Time Story — Raju Faces Node Eviction

Raju is monitoring his **2-node RAC cluster** (Node1 & Node2).  
One day, **Node2 suddenly reboots** after a network glitch.  

The log shows:  


Raju realizes what happened:
- Both Node1 and Node2 were alive, but the private interconnect failed temporarily.
- Each node thought the other was dead (*split-brain danger*).
- To resolve conflict, Clusterware used the **Voting Disks**.
- Node1 had access to the majority → Node2 lost access → Node2 was **evicted** for safety.

👉 Lesson: Voting Disks **decide who stays in the cluster** when nodes fight.

---

## 🔹 Subtopic 4: Quorum & Quorum Disks

- **Quorum** = majority agreement.  
- In RAC:
  - With **3 Voting Disks** → need access to **2**.  
  - With **5 Voting Disks** → need access to **3**.  
- Oracle best practice = always configure an **odd number of Voting Disks** (to avoid tie).  

### 🛠 Quorum Disks
- In stretched clusters (RAC across 2 sites), sometimes nodes split 50/50.
- Oracle allows a **Quorum Disk** (or tie-breaker disk) in a **third location**.
- This ensures one site gets majority and avoids total split.

---

## 🔹 Subtopic 5: Commands for Voting Disks

Check Voting Disk location:  
```bash
crsctl query css votedisk

## STATE    File Universal Id                File Name Disk group
-- -----    -----------------                --------- ---------
  1. ONLINE 72a8f1b7a4d54f9f...              /dev/oracleasm/disks/VOTEDISK1
  2. ONLINE 8b9cd1a2e3f7450e...              /dev/oracleasm/disks/VOTEDISK2
  3. ONLINE 4f6c92d17fa14d5e...              /dev/oracleasm/disks/VOTEDISK3
Located 3 voting disk(s).

Replace Voting Disk (as root/grid user):
--> crsctl replace votedisk +OCR

Backup Voting Disk config:
--> crsctl query css votedisk > /tmp/vdisk_backup.txt


## 🔹 Subtopic 6: Advanced Voting Disk Details

### Storage
- Usually stored in **ASM Diskgroups** (e.g., `+OCR`, `+GRID`).

### Redundancy
- **Normal redundancy** → 3 Voting Disks.  
- **High redundancy** → 5 Voting Disks.

### Auto Management
- From **11gR2 onwards**:  
  - Voting Disks are stored inside ASM.  
  - Oracle automatically manages redundancy, so DBAs don’t need to manually configure multiple Voting Disks.

### Eviction Mechanism
- Controlled by **CSSD (Cluster Synchronization Services Daemon)**.  
- If a node loses access to **majority disks** → **eviction triggers** → node reboot.  
- This prevents **data corruption and split-brain** situations.

---

## 🔹 Subtopic 7: Boot Dependency on Voting Disks

When a cluster starts:

1. **OHASD** starts basic services.  
2. **CSSD** (Cluster Sync Service Daemon) checks Voting Disks.  
3. Only nodes with access to **majority Voting Disks** are allowed to join the cluster.  
4. Nodes without majority are **evicted** → ensures no split-brain.  

👉 **Voting Disks = gatekeepers during cluster startup.**

---

## 📖 Story Analogy: The Apartment Voting System

Raju explains Voting Disks to his junior using an **apartment society** story:

- Each **RAC node = flat owner** in the society.  
- Every 30 seconds → each owner **signs an attendance sheet** (Voting Disk).  
- If an owner (node) stops signing → society assumes he has **left**.  
- If two owners **fight (split-brain)**:  
  - The **majority signatures win**.  
  - The minority side is **forced to leave (evicted)**.  

Example:  
If the society has **3 attendance registers (3 Voting Disks)** →  
A flat owner must sign at least **2** to remain valid.  

👉 That’s how RAC decides who stays and who leaves in case of conflict.

---

## 🔹 Subtopic 8: Real-Time Troubleshooting

- Raju investigates → SAN team had **accidentally unmounted one LUN**.  
- **Impact** → Node2 could not access majority Voting Disks → evicted.  
- **Fix**:  
1. Restore disk access.  
2. Run command:  
   ```bash
   crsctl replace votedisk
   ```

---

### 📌 Scenario 2: Split Brain in Stretched RAC
- Cluster across **Delhi (Node1)** and **Mumbai (Node2)**.  
- Network between cities goes down.  
- Both nodes think the other side is **dead** → **split-brain condition**.  
- Solution → **Quorum Disk in Chennai** acts as tiebreaker.  
- Whichever node has majority with the Chennai Quorum Disk survives.  

👉 This mechanism **saves data consistency** across sites.

---

### 📌 Scenario 3: Loss of All Voting Disks
- If all Voting Disks are lost → Cluster **cannot function**.  
- **Fix**:  
- Recreate Voting Disks using:  
  ```bash
  crsctl replace votedisk +DATA
  ```
- Ensure diskgroups are healthy.

---

### 📌 Scenario 4: Startup Failure due to Voting Disk Inaccessibility
- During startup → **CSSD cannot access majority Voting Disks**.  
- Node is **denied entry** into the cluster.  
- DBA must:  
- Check ASM Diskgroup mount status.  
- Validate network/storage paths.  
- Restore Voting Disk accessibility.

---

## 🔹 Subtopic 9: ASCII Diagram (Cluster with Voting Disks)

   [Node1] ----\
                 >---- [ Voting Disk 1 ]
   [Node2] ----/        [ Voting Disk 2 ]
                 \---- [ Voting Disk 3 ]



- **Majority = 2 of 3**.  
- Only nodes with majority survive.

---

## 📝 Key Takeaways

- **Voting Disk = heartbeat disk of RAC cluster.**

### Purpose
- Tracks **node membership**.  
- Prevents **split-brain**.  
- Decides **node eviction**.  

### Rules
- Nodes must access **majority Voting Disks** to stay in cluster.  
- **Odd number** of Voting Disks is best (3, 5).  
- In **stretched clusters** → use a **Quorum Disk in 3rd site**.  

### Storage
- Stored in **ASM** in modern clusters (11gR2+).  
- Oracle automatically manages redundancy.

### Commands
```bash
crsctl query css votedisk   # Check current Voting Disk configuration
crsctl replace votedisk     # Replace/restore Voting Disk


