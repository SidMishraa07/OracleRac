# Oracle Cluster Synchronization Services (CSS) – Introduction

## Overview
- CSS is the **core cluster service** in Oracle RAC.
- It maintains **synchronization between nodes**.
- CSS ensures that all nodes agree on cluster membership.
- Plays a **key role in preventing split-brain scenarios**.

## Key Functions
1. Node membership monitoring.
2. Synchronization of voting mechanism.
3. Node eviction if heartbeat fails.
4. Works with Oracle High Availability Services (OHAS).

## Real-time Example
- In a 2-node RAC, if **Node A loses connectivity** to the interconnect, CSS decides whether Node A should be evicted or not.

# CSS Architecture

## Components
1. **CSS Daemon (cssd.bin)**  
   - Runs on every node.  
   - Manages membership and heartbeat communication.

2. **CSS Monitors**  
   - Track disk heartbeat via Voting Disk.  
   - Track network heartbeat via interconnect.

3. **Cluster Time Synchronization Service (CTSS)**  
   - Ensures consistent system clocks across nodes.

## Workflow
1. Each node writes heartbeats to the Voting Disk.  
2. Each node exchanges heartbeats over interconnect.  
3. CSS checks if all nodes are alive.  
4. If a node fails to send heartbeat → eviction is triggered.

## Scenario
- If **Node B stops writing to Voting Disk** due to storage failure → CSS assumes Node B is down and evicts it to protect cluster integrity.

# Voting Disk in RAC

## What is Voting Disk?
- A shared disk used by all RAC nodes.  
- Stores **node membership information**.  
- Ensures cluster-wide agreement on which nodes are active.  

## Functions
- Keeps track of which nodes are up and communicating.  
- Participates in **quorum decisions**.  
- Assists in **eviction** during communication failures.

## Best Practices
- Minimum 3 voting disks recommended for redundancy.  
- Should be placed on **shared storage** (ASM, NFS, etc.).  

## Scenario
- In a 3-node cluster with 3 voting disks:  
  - If **Node C loses access** to 2 voting disks → it loses quorum and gets evicted.

# Node Eviction Mechanism in CSS

## Why Node Eviction?
- Prevents split-brain (two nodes believing they are masters).  
- Ensures only healthy nodes access shared resources.

## Process
1. CSS detects loss of heartbeat.  
2. CSS compares voting disk records.  
3. Node failing to maintain majority is evicted.  

## Eviction Types
- **I/O Fencing** – Node loses access to disk.  
- **Network Eviction** – Node is removed due to interconnect failure.  
- **CSS Daemon Eviction** – Internal CSS corruption.

## Real-Time Example
- In a 4-node RAC:  
  - Node D loses interconnect and can’t write to Voting Disk.  
  - CSS evicts Node D to protect database consistency.

# CSS and OHAS Integration

## How CSS Fits into OHAS
- OHASD (Oracle High Availability Services Daemon) starts first.  
- OHASD starts CSSD.  
- CSSD ensures that cluster synchronization is established.  
- Once CSS is stable → Cluster Ready Services (CRS) start.

## Dependency Flow
1. OHASD → CSSD → CRSD → ASM → RDBMS  
2. CSS must be stable before higher layers start.  

## Scenario
- During cluster boot:  
  - If CSS fails to start → OHAS cannot bring up CRS.  
  - Database instances cannot start without cluster sync.

# CSS Troubleshooting & Scenarios

## Common Issues
1. **Interconnect Failure**
   - CSS detects missing network heartbeat.
   - Node eviction is triggered.

2. **Voting Disk Corruption**
   - Node cannot update heartbeat.
   - Node eviction occurs.

3. **CSS Daemon Crash**
   - Logs in `$GRID_HOME/log/<hostname>/cssd/alert.log`.

4. **Split-Brain**
   - CSS fencing mechanism ensures only one side survives.

## Example Diagnostic Commands
- `crsctl check css`
- `crsctl stat res -t`
- `olsnodes -n`
- Check logs: `$GRID_HOME/log/<node>/cssd/`

## Scenario
- In a 2-node RAC, **network cable unplugged from Node A**:  
  - Node A cannot communicate via interconnect.  
  - Voting Disk shows Node A missing.  
  - CSS evicts Node A → cluster continues safely with Node B.


