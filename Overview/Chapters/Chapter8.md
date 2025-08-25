chapter-8-cache-fusion-gcs-ges

# Cache Fusion - Introduction

## What is Cache Fusion?
Cache Fusion is the **heart of Oracle RAC**.  
It allows multiple RAC nodes to **share data blocks directly in memory** (via interconnect), instead of reading/writing to disk again and again.

Without Cache Fusion, RAC would be like multiple people trying to read/write a single diary at once, always waiting for the diary to be put back on the table.  
With Cache Fusion, they pass the **page of the diary** directly to the other person → faster & efficient.

---

## Why Cache Fusion is Important?
- Prevents **disk I/O bottlenecks**.
- Ensures **data consistency** across nodes.
- Reduces **latency** in high-transaction systems.
- Makes RAC possible → multiple nodes acting like one big database.

---

## Raju's Real-Life Example
Raju is working in SBI Bank’s core system running on RAC.  
A customer deposits ₹5000 at **Node 1** branch, and at the same time, another customer checks balance from **Node 2** ATM.  

- Without Cache Fusion → Node 2 would wait until Node 1 writes changes to disk, then read from disk → **slow**.  
- With Cache Fusion → Node 1 sends the changed block directly to Node 2 via **interconnect** → **instant update**.  

This is why customers see real-time updates even across multiple locations.

---


#################################################################

# Global Cache Services (GCS)

## What is GCS?
- GCS manages **data block transfers** between nodes.
- It decides **who owns the current version** of a block (master node).
- Ensures **consistency** across instances.

---

## Key Functions of GCS
- Block Shipping → Transfers data blocks between buffer caches.
- Role Management → Determines who holds a block (Exclusive or Shared).
- Maintains a **Global Resource Directory (GRD)**.

---

## Example with Raju
- Raju queries `SELECT balance FROM account WHERE acc_no=123;` on **Node 1**.  
- Node 1 gets the block into its cache.  
- Now, a transaction on **Node 2** tries to update that account.  
- GCS steps in → decides **who should give the block**, ships it across → update is done.

---

## GRD in Simple Words
Think of GRD as a **central register** that keeps track of:
- Which node has which block.
- Whether it is in **Shared** mode (read-only) or **Exclusive** (write mode).

---

#################################################################

# Global Enqueue Services (GES)

## What is GES?
GES manages **locks and enqueues** at the cluster level.
- Prevents two nodes from updating the same resource at the same time.
- Works with GCS to maintain **global consistency**.

---

## Functions of GES
- Manages **DDL locks** (e.g., creating tables, altering schemas).
- Manages **dictionary cache locks**.
- Resolves **deadlocks across nodes**.

---

## Real-Life Example with Raju
- Raju is running a query on **Node 1** to `ALTER TABLE customer ADD column dob`.  
- Meanwhile, Node 2 tries to insert into the same table.  
- GES ensures Node 2 waits → avoids corruption.  

Another case:  
Two different sessions in different nodes are stuck waiting for each other’s resource → GES detects it → picks a **victim session** to resolve deadlock.

---

#################################################################

# Cache Fusion Mechanism - Real Life

## How Cache Fusion Works
1. **Read Request**
   - If Node 2 wants a block that Node 1 has, Node 1 sends it directly.
2. **Write Request**
   - If Node 2 wants to update a block Node 1 has in Shared mode → Node 1 ships the block & downgrades its role.
3. **Role Change**
   - Ownership of blocks is continuously transferred → like a **baton in relay race**.

---

## Raju’s Banking Scenario
1. Node 1 processes deposit of ₹5000 into Account A.
2. Node 2 simultaneously processes withdrawal from the same Account A.
3. Cache Fusion ensures:
   - Node 2 gets the updated block directly.
   - Balance is always correct and consistent.
   - No waiting for disk I/O.

---

## Analogy
Think of Cache Fusion like **WhatsApp sync**:  
- If Raju sends a message on his phone (Node 1), it instantly appears on his laptop (Node 2).  
- No need to wait for a backup/restore → direct memory-to-memory sync.

#################################################################

# Cache Fusion Performance Issues & Troubleshooting

## Common Performance Bottlenecks
- Slow **interconnect network**.
- Hot blocks → Too many nodes fighting for the same block.
- High contention on certain tables.

---

## Raju’s Case
Raju notices that the **end-of-day batch jobs** are running slow on RAC.  
- On investigation, he finds “gc buffer busy” waits.  
- It means multiple nodes are fighting for the same block.

---

## Troubleshooting Steps
- Check `GV$SESSION_WAIT` and `GV$SYSTEM_EVENT`.
- Monitor `gc buffer busy acquire` and `gc buffer busy release` events.
- Optimize queries to **reduce block contention**.
- Ensure interconnect is using **high-speed dedicated network** (e.g., 10GbE or Infiniband).

---

#################################################################

# Practical Scenarios with Raju - Cache Fusion & GCS/GES

## Scenario 1: Balance Check & Deposit
- Customer checks balance on Node 2.
- Another customer deposits on Node 1.
- Cache Fusion ensures Node 2 sees updated balance instantly.

---

## Scenario 2: Hot Block Contention
- Raju runs salary update script affecting **same rows** across branches.
- Multiple nodes keep fighting for blocks.
- Solution → partition workload or use service-based routing.

---

## Scenario 3: Deadlock Across Nodes
- Node 1 session updates Table A then waits on Table B.
- Node 2 session updates Table B then waits on Table A.
- GES detects deadlock → kills one session → avoids corruption.

---

## Scenario 4: Slow Interconnect
- Raju mistakenly configured interconnect on 1GbE instead of 10GbE.
- Cache Fusion becomes slow → RAC performance drops.
- Fix: Configure private high-speed interconnect.

---

#################################################################

# Takeaways - Cache Fusion & GCS/GES

- **Cache Fusion** is the backbone of Oracle RAC → enables memory-to-memory block transfer.
- **GCS** manages block ownership and consistency.
- **GES** manages locks, enqueues, and deadlocks across nodes.
- Performance issues mostly come from:
  - Poor interconnect setup.
  - Hot blocks and high contention.
- Always monitor **gc waits** and tune queries/applications accordingly.

---

## Raju’s Wisdom
Raju learned that without Cache Fusion, RAC is just like **multiple chefs trying to cook with one recipe book on a single shelf**. Everyone waits.  

With Cache Fusion → the recipe pages are **photocopied instantly** between chefs → smooth, fast, and consistent cooking.

---

#################################################################

