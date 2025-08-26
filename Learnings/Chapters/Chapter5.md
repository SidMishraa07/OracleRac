# 📖 Chapter 5 – Oracle RAC Networking

Networking is the *circulatory system* of Oracle RAC.  
If clusterware is the "brain", then **networking is the blood flow** that keeps everything alive.

In RAC, multiple nodes work together as **one database**, and to achieve this, reliable and well-planned networking is crucial.

---

## 🔑 Takeaways:
- Oracle RAC **requires multiple networks** (public + private).
- Different types of IPs (Public, VIP, Private, SCAN) each play a **unique role**.
- RAC networking ensures:
  - Client connections are load balanced.
  - Cluster nodes communicate seamlessly.
  - High availability during node/network failures.

👉 Without proper networking, RAC = chaos.

---

## Files in This Chapter
- **1-why-networking-important.md** → Why RAC networking matters.  
- **2-private-public-vip-ips.md** → Public IP, Private IP, and VIP explained with examples.  
- **3-scan-ips.md** → Role of SCAN IP in client connections.  
- **4-gns-vs-non-gns.md** → Grid Naming Service vs Non-GNS setups.  
- **5-interconnect.md** → Heartbeat communication & importance of private interconnect.  
- **6-network-failover.md** → Failover scenarios (node crash, NIC failure, cable cut).  
- **7-practical-scenarios.md** → Raju’s real-time RAC networking troubleshooting stories.  


# 🌐 Why Networking is Important in RAC?

In a **standalone database**, only one server handles all client requests.  
But in **RAC**, multiple nodes share the workload. For this to work:

1. **Clients** must connect seamlessly (public IPs, SCAN).
2. **Nodes** must talk to each other (private interconnect).
3. **Failover** must happen automatically (VIPs, Clusterware).

---

## 🏢 Real-World Analogy
Imagine Raju is running a **banking system** on RAC.  
Thousands of customers log in daily.

- A customer opens the app → connects to **SCAN IP**.(ScanIP like a load Balancer)
- Oracle routes them to **Node1** or **Node2**.
- If Node1 crashes → Clusterware reroutes the connection to **Node2** without customer noticing.

👉 Networking ensures **availability + performance**.

---

## ✅ Key Takeaways
- Networking is not "optional" but **foundation** of RAC.
- Poorly designed networking = frequent node eviction.
- Always plan **separate subnets** for Public, Private, and SCAN IPs.


# 🖧 Public, Private, and VIP IPs

RAC uses multiple types of IP addresses:

---

## 🔹 Public IP
- Assigned to each node.
- Used for **client connectivity**.
- Example:  
  - Node1 → 192.168.1.101  
  - Node2 → 192.168.1.102  

---

## 🔹 Private IP (Interconnect)
- Used for **node-to-node communication**.
- Must be on a **separate subnet** (non-routable).
- Example:  
  - Node1 → 10.0.0.1  
  - Node2 → 10.0.0.2  

---

## 🔹 VIP (Virtual IP)
- A floating IP associated with each node.
- Used for **client failover**.
- If Node1 crashes:
  - Node1’s VIP (192.168.1.111) automatically moves to Node2.
- This prevents clients from waiting for TCP timeout.

---

## 🏢 Raju’s Example
Raju is configuring RAC with 2 nodes.

| Node   | Public IP      | Private IP | VIP            |
|--------|----------------|------------|----------------|
| Node1  | 192.168.1.101  | 10.0.0.1   | 192.168.1.111  |
| Node2  | 192.168.1.102  | 10.0.0.2   | 192.168.1.112  |

👉 Clients always connect to **SCAN → VIP → Node**.


# 🎯 SCAN IPs (Single Client Access Name)

- Introduced in **Oracle 11g R2**.
- A **single hostname** resolves to 3 IPs (Round Robin).
- Clients connect to SCAN instead of individual node hostnames.

---

## 🎬 How it Works
1. Client connects to → `scan.mydb.com`
2. DNS resolves to one of the SCAN IPs:
   - 192.168.1.201
   - 192.168.1.202
   - 192.168.1.203
3. SCAN listener forwards request to available node.

---

## 🏢 Example
Raju configures SCAN in his RAC cluster:
- SCAN Name: `scan.raju-rac.com`
- SCAN IPs: `192.168.1.201-203`

Now, clients don’t care which node is up.  
Oracle handles **load balancing + failover** internally.

---

## ✅ Benefits
- Clients don’t need to reconfigure when nodes are added/removed.
- Simplifies **connectivity**.
- Supports **high availability**.


# 📡 GNS vs Non-GNS

## 🔹 GNS (Grid Naming Service)
- Automates IP resolution inside cluster.
- Eliminates manual `/etc/hosts` editing.
- Useful for **large RAC deployments**.

---

## 🔹 Non-GNS (Traditional)
- Requires manual entry of Public, Private, VIP, SCAN in DNS or `/etc/hosts`.
- More common in small 2-node clusters.

---

## 🏢 Raju’s Example
- In his test lab (2-node RAC), Raju uses **Non-GNS** with `/etc/hosts`.
- In production (10-node RAC), company uses **GNS** for simplicity.

👉 Use **GNS** for scalability, **Non-GNS** for learning/labs.


# 🔗 Private Interconnect (The Heartbeat)

- The **interconnect** is the lifeline of RAC.
- Used for:
  - Heartbeat communication.
  - Cache Fusion (sharing data blocks between nodes).

---

## 🏃 Importance
If interconnect fails → cluster eviction may happen.  
That’s why Oracle recommends:
- At least **2 NICs bonded** for private interconnect.
- Use **low latency** networks (10GbE/InfiniBand).

---

## 🏢 Raju’s Example
Raju configures:
- Node1 Private IP: 10.0.0.1
- Node2 Private IP: 10.0.0.2
- He uses **bonding** of two NICs for redundancy.

👉 This ensures no single point of failure.


# ⚡ RAC Network Failover Scenarios

---

## 🔹 Scenario 1: Node Failure
- Node1 crashes → VIP 192.168.1.111 relocates to Node2.
- Clients automatically reconnect via VIP.

---

## 🔹 Scenario 2: NIC Failure
- One NIC in private network fails.
- Bonding ensures traffic continues via 2nd NIC.

---

## 🔹 Scenario 3: Cable Cut
- If public NIC cable is cut:
  - Node still communicates internally (private IP works).
  - But clients connect via VIP on another node.

---

## ✅ Key Takeaways
- Always configure **VIPs** for client failover.
- Use **NIC bonding** for high availability.
- SCAN ensures client connections remain seamless.


# 🛠️ Raju’s RAC Networking Stories

---

## 🎬 Story 1: The VIP Surprise
Raju forgot to configure VIPs during setup.  
When Node1 crashed, clients hung for **2 minutes** waiting for TCP timeout.  
After adding VIPs, failover happened instantly.  

👉 Lesson: VIPs save timeouts.

---

## 🎬 Story 2: The Single NIC Disaster
Raju used only **one NIC** for the private interconnect.  
One day, the switch failed → entire RAC cluster went down.  

👉 Lesson: Always bond NICs.

---

## 🎬 Story 3: The SCAN Advantage
Initially, Raju gave clients direct node IPs.  
Every time he added/removed a node, clients had to reconfigure TNS.  
After switching to SCAN, clients never changed configs again.  

👉 Lesson: SCAN simplifies life.
