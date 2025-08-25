# Introduction to Oracle Clusterware

## What is Clusterware?
- Oracle Clusterware is a collection of background processes, daemons, and utilities that allow multiple servers to function as a single database cluster.
- It provides node membership, messaging, failure detection, and high availability.

## Key Features
- **Node membership management** → Keeps track of which nodes are part of the cluster.
- **Failure detection & eviction** → Identifies unresponsive nodes and evicts them.
- **Voting disk & OCR** → Ensures cluster consistency.
- **Integration with RAC** → RAC depends on Clusterware for instance coordination.

---

## Raju's Real-life Example
Raju has 2 physical servers (`node1` and `node2`).  
Without Clusterware, if `node2` fails, `node1` would not even know.  
With Clusterware:
- `node1` continuously pings `node2` via private interconnect.  
- If `node2` stops responding, Clusterware evicts it (so no split-brain occurs).  

---

# Oracle Clusterware Components

## Core Components
1. **Voting Disk**
   - Used to check which nodes are alive in the cluster.
   - Majority of voting disks must be accessible.
   - Example: Raju keeps 3 voting disks on shared ASM diskgroup.

2. **OCR (Oracle Cluster Registry)**
   - Stores cluster configuration: nodes, databases, services.
   - Example: Raju adds a new service `payroll_svc` → it gets stored in OCR.

3. **OHASD (Oracle High Availability Services Daemon)**
   - Starts first on each node.
   - Responsible for initiating other daemons.

4. **Cluster Ready Services (CRSD)**
   - Manages database instances, services, and failover.

5. **Cluster Synchronization Services (CSSD)**
   - Handles voting disks and node membership.

6. **Event Manager Daemon (EVMD)**
   - Publishes cluster events.

---

## Raju’s Scenario
- Raju tries to stop `node1` suddenly.
- CSSD detects loss of heartbeat.
- CRSD triggers failover → services move to `node2`.
- OCR updates status of cluster resources.

---

# Cluster Startup & Boot Sequence

## Boot Order of Processes
1. **OHASD**  
   - Auto-starts with OS boot.  
   - Reads OCR and starts ASM.  

2. **CSSD**  
   - Manages voting disks and cluster membership.  

3. **CRSD**  
   - Brings up RAC databases, services, VIPs.  

4. **Other Daemons** (ONS, EVMD, GPNPD).  

---

## Step-by-Step Flow
1. Node boots → OHASD starts.  
2. OHASD starts ASM instance (reads voting disk).  
3. CSSD forms node membership.  
4. CRSD starts database instances.  
5. Node joins RAC successfully.  

---

## Raju’s Example
- Raju reboots `node2`.  
- Sequence:  
  1. OHASD starts → finds ASM diskgroup.  
  2. CSSD checks node2 voting.  
  3. CRSD starts Raju’s `HRDB2` instance.  
  4. Services like `HR_APP` relocate automatically.  

---

# Node Eviction & Reboot Scenarios

## What is Node Eviction?
- Process of removing an unhealthy node from cluster to prevent corruption.

## Causes of Eviction
- Network heartbeat lost.
- Disk heartbeat lost.
- Node freeze (OS hung).
- CSS miscommunication.

---

## Types of Failures
1. **Network Failure**  
   - Node loses private interconnect.  
   - Surviving node(s) evict it.  

2. **Disk Failure**  
   - Node can’t access voting disk.  
   - CSS evicts it.  

3. **OS Crash**  
   - Watchdog detects → forces reboot.  

---

## Raju’s Example
- Network cable of `node2` is unplugged.  
- `node1` sees no heartbeat → evicts `node2`.  
- `node2` reboots and rejoins cluster.  

---

# Troubleshooting Clusterware Startup

## Common Issues
1. **OHASD not starting**
   - Check `/etc/init.d/init.ohasd`.
   - Verify root ownership & permissions.

2. **CSSD errors**
   - Issue with voting disks.
   - Run `crsctl query css votedisk`.

3. **CRSD not starting**
   - OCR corruption.
   - Check with `ocrcheck`.

---

## Logs to Check
- OHASD: `$GRID_HOME/log/<node>/ohasd/ohasd.log`
- CSSD: `$GRID_HOME/log/<node>/cssd/ocssd.log`
- CRSD: `$GRID_HOME/log/<node>/crsd/crsd.log`

---

## Raju’s Example
- Raju notices his RAC didn’t start after reboot.  
- He runs:  
  crsctl check cluster -all
