# Oracle Clusterware and RAC

## What is Oracle Clusterware and why is it needed in RAC?

**Answer:**  
Oracle Clusterware is a software that allows multiple servers (nodes) to work together as a single system. It manages node membership, monitors processes, and provides communication between nodes. Clusterware ensures that in case one node fails, the database instances can continue to run on the surviving nodes, providing high availability. In RAC, Clusterware is mandatory because it manages the database instances, ASM, listener, and ensures automatic failover and restart.

---

## Can you explain the key components of Oracle Clusterware and their purpose?

**Storage Components:**

- **OCR (Oracle Cluster Registry):** Stores cluster configuration (databases, resources, VIPs, services, ASM info).
- **Voting Disk:** Maintains node membership, helps in node eviction if heartbeat fails.

**Daemons/Processes:**

- **OHASD (Oracle High Availability Services Daemon):** First to start, manages cluster startup and basic services.
- **CSSD (Cluster Synchronization Services Daemon):** Manages node membership and voting disk communication.
- **CRSD (Cluster Ready Services Daemon):** Manages cluster resources (database, ASM, listeners). Handles failover & restarts.
- **EVMD (Event Manager Daemon):** Publishes cluster events to subscribers (OEM, logs).

---

## During Clusterware boot sequence, which process starts first and why?

**Answer:**  
OHASD (Oracle High Availability Services Daemon) starts first (triggered by init/systemd as root). It lays the foundation for cluster startup.

- GPnP & CSSD start next. CSSD reads the Voting Disk, manages node membership & heartbeats.
- Once membership is stable, CRSD starts to manage cluster resources (DB, ASM, services, listeners, VIPs).
- EVMD starts to broadcast cluster events.
- Finally, the actual Oracle resources (ASM, databases, listeners) start under CRSD’s control.

---

## What happens if a node loses access to the Voting Disk in a RAC environment?

**Answer:**  
If a node loses access to the Voting Disk, CSSD will detect that it can no longer write its disk heartbeat. If at the same time the network heartbeat with other nodes also fails, the cluster will evict that node to prevent a split-brain condition. Eviction ensures cluster consistency and data integrity. Causes may include network card failure, disk connectivity issues, or server hang.

---

## Can you explain the difference between Network Heartbeat and Disk Heartbeat in Oracle Clusterware?

**Answer:**

- **Network Heartbeat:** Sent every second over the private interconnect between nodes to check communication.
- **Disk Heartbeat:** Each node writes a small I/O to the Voting Disk every 3 seconds to confirm access to shared storage.

Both heartbeats work together — eviction occurs only when both fail, preventing false node eviction.

---

## Why do we need Voting Disk redundancy, and how is it usually configured in real-time environments?

**Answer:**  
Voting Disk redundancy avoids a single point of failure. Clusterware uses the majority rule (quorum) to decide which nodes remain in the cluster.

- **Normal Redundancy:** 3 voting disks (majority = 2).
- **High Redundancy:** 5 voting disks (majority = 3).  
  From 11gR2 onwards, voting disks are typically stored inside ASM disk groups (e.g., +OCR, +GRID) and managed automatically.

---

## During the Clusterware boot sequence, if the CSSD process fails to start, what will happen to the cluster?

**Answer:**  
If CSSD fails to start:

- The node cannot communicate with the voting disk or participate in the cluster heartbeat.
- Clusterware will try to restart CSSD through OHASD.
- If it still fails, the node is evicted and rebooted to protect the cluster.
- If CSSD fails on all nodes, the cluster cannot function as quorum cannot be determined.

---

## What is the role of the Oracle High Availability Services Daemon (OHASD) in the boot sequence?

**Answer:**  
OHASD is the first Clusterware process to start. It:

- Bootstraps cluster and local resources (GPnP, disk discovery, OCR/Voting Disk access).
- Spawns CSSD to manage cluster membership.
- Manages local ASM instances.  
  If OHASD fails, no other Clusterware processes can start.

---

## Can you explain what happens during node eviction and why it is necessary in a RAC environment?

**Answer:**  
Node eviction maintains consistency and prevents split-brain.

- If a node fails health check heartbeats, CSSD declares it unhealthy and evicts it.
- Node is fenced off from shared storage.
- Node is forcibly rebooted.
- Surviving nodes continue to serve requests without disruption.

---

## Can you explain the order of processes starting from OHASD to database startup and their dependencies?

**Answer:**

1. **OHASD:** Starts first, bootstraps cluster.
2. **CSSD:** Maintains node membership, checks heartbeats.
3. **CRSD:** Manages cluster resources like ASM, listeners, VIPs, and databases.
4. **ASM:** Comes up under CRSD control.
5. **Databases and services:** Start after ASM and listeners are ready.

---

## What is the difference between node eviction and node reboot in Clusterware?

**Answer:**

- **Node Eviction:** Node removed from cluster due to heartbeat failures; not necessarily rebooted immediately.
- **Node Reboot:** Node restarts OS and all cluster services after eviction or critical failure.

---

## What is the purpose of the Cluster Ready Services Daemon (CRSD), and why can’t the database start properly without it?

**Answer:**  
CRSD starts after CSSD and manages all cluster resources: ASM, listeners, VIPs, and database instances. It ensures proper startup, failover, and resource allocation. Without CRSD, resources like ASM and listeners won’t be brought online, preventing the database from starting.

---

## Why is Clusterware considered mandatory in RAC, even if ASM and the database are installed?

**Answer:**  
Clusterware manages coordination between multiple nodes, ensuring they operate as a single cluster. It:

- Maintains node membership and quorum.
- Monitors heartbeats and performs node eviction.
- Starts, stops, and failovers cluster resources.  
  Without Clusterware, RAC nodes cannot communicate or provide high availability.

---

## What is the role of the private interconnect in Clusterware boot and RAC operation?

**Answer:**  
Private interconnect is a high-speed network between nodes used for:

- **Network Heartbeat:** Monitors node health.
- **Cache Fusion / Data Block Transfers:** Synchronizes block changes between instances.  
  It ensures high availability and consistency.

---

## What is split-brain in RAC, and how does Clusterware prevent it during the boot or runtime?

**Answer:**  
Split-brain occurs when nodes incorrectly assume others are dead and operate independently, causing data corruption.  
Clusterware prevents it using:

- Voting Disk quorum.
- Network Heartbeats.
- Node Eviction if both heartbeats fail.

********\*\*********Command Specific Questions****\*\*\*\*****

# Oracle RAC Clusterware Commands and Q&A

Q: How do you check the clusterware status on an Oracle RAC node?  
A: Use `crsctl stat res -t` to view the status of all cluster resources.  
Example:
Name Target State Server
Ora.CSSd ONLINE ONLINE node1
Ora.LISTENER ONLINE ONLINE node1
Ora.VoteDisk ONLINE ONLINE node1,node2
Other useful commands:

- `crsctl check cluster` → Verifies cluster health and basic configuration.
- `crsctl stat res -init` → Shows the startup state of cluster resources.

Q: How do you start the Oracle Clusterware on a node?  
A: Use `crsctl start crs` to start all clusterware daemons on the node. To start on all nodes, run on each node or use `srvctl` for managed resources.

Q: How do you stop Oracle Clusterware on a node?  
A: Use `crsctl stop crs` to stop all clusterware daemons on the node. To stop gracefully on all nodes, repeat on each node.

Q: How do you check which nodes are part of the cluster?  
A: Use `olsnodes`.  
Example output:
node1
node2
node3

Q: How do you check the status of ASM and database instances after clusterware startup?  
A: Use the following commands:

- `srvctl status asm` → Checks ASM status across nodes.
- `srvctl status database -d <DB_NAME>` → Checks RAC database instance status.

Q: How do you check detailed status of a specific cluster resource, e.g., the CSS daemon, and its dependencies?  
A: Use `crsctl stat res -t -v -w "TYPE = ora.cssd.type"`

- `-v` → verbose, shows dependencies and resource attributes.
- `-w` → filter by resource type.  
  Example use-case: Troubleshooting CSS-related issues where other resources fail to start because CSS is down.

Q: How do you verify if a node is evicted or not from the cluster?  
A: Use the following commands:

- `crsctl query css votedisk`
- `crsctl query css membership`  
  Example output:
  Node NodeNum Membership
  node1 1 ONLINE
  node2 2 OFFLINE (Evicted)

Q: How do you forcefully start a failed cluster resource, e.g., ASM, without starting the entire cluster?  
A: Use `crsctl start resource ora.asm -init`

- `-init` → starts the resource in initial mode.
- Alternatively, start ASM on a specific node: `srvctl start asm -n <node_name>`

Q: How can you check the startup type of a cluster resource (AUTOSTART / MANUAL)?  
A: Use `crsctl stat res -p | grep <resource_name>`  
Example:
NAME=Ora.LISTENER.CRS TARGET=ONLINE STATE=ONLINE
Look for `TARGET=ONLINE` vs `TARGET=OFFLINE`.

Q: How do you check the CRS startup sequence to identify which resource failed during boot?  
A: Use `crsctl log cssd -t`

Q: How do you check if the clusterware will start automatically after node reboot?  
A: Use `crsctl enable crs` followed by `crsctl stat res -t`

Q: Which command helps detect network or node failures causing split-brain?  
A: Use `crsctl log cssd -f`

- `-f` → follow logs in real-time.
- Check for messages like `NODE EVICTED` or `VOTE_DISK_UNAVAILABLE`.

Q: How do you check all dependent resources for ora.cssd.type?  
A: Use `crsctl stat res -v -w "TYPE = ora.cssd.type"`

Q: How can you get the last error reason for a failing resource?  
A: Use `crsctl stat res -v <resource_name>`
