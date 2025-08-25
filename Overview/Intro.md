# 🏗️ Oracle RAC Learning Roadmap

This repository documents my preparation for **Oracle RAC (Real Application Clusters)** interview and hands-on practice.  
We’ll go through RAC in a structured way — 20 short but focused chapters — moving from fundamentals to advanced troubleshooting and interview drills.  

---

## 📘 RAC Roadmap: 20 Chapters

1. **Clusterware Fundamentals & Boot Sequence**  
   Understanding Oracle Clusterware components, startup sequence, and background processes.

2. **OCR Deep Dive**  
   Oracle Cluster Registry layout, purpose, backup, and restore procedures.

3. **Voting Disks & Quorum**  
   Role in cluster membership, majority/quorum rules, and quorum disks.

4. **RAC Networking**   
   Public & private networks, VIPs, SCAN listeners, GNS, and HAIP.

5. **ASM for RAC**  
   Disk groups, redundancy levels, and ASMFD/ASMLib integration.

6. **Cache Fusion & Global Enqueue Services (GCS/GES)**  
   How RAC maintains data consistency across nodes.

7. **RAC Database Architecture**  
   Instances, redo/undo/temp management, and key initialization parameters.

8. **Policy-Managed vs Admin-Managed Clusters**  
   Server pools, resource management, and deployment strategies.

9. **Pre-Installation Planning**  
   OS requirements, kernel parameters, packages, users, and ulimits.

10. **Grid Infrastructure Installation**  
    Using the GUI wizard and silent mode for GI setup.

11. **RAC Database Creation**  
    Creating databases with DBCA and performing post-creation checks.

12. **Services & Workload Management**  
    FAN, ONS, TAF, and edition-based routing for workload balancing.

13. **Patching GI & DB**  
    Out-of-place patching, `opatchauto`, and rolling patch strategies.

14. **Backup & Recovery in RAC**  
    RMAN in ASM environments and restore path considerations.

15. **Observability & Diagnostics**  
    ADR framework, CHM/OSWatcher, and diagnostic data collection.

16. **Node Eviction Troubleshooting**  
    CSS misscount, network timeouts, disk issues, and eviction analysis.

17. **Performance in RAC**  
    Handling `gc` waits, optimizing interconnects, and tuning strategies.

18. **Rolling Upgrades & Node Reconfiguration**  
    Adding/removing nodes and performing rolling upgrades.

19. **RAC with Data Guard**  
    Integration patterns, switchover/failover scenarios, and pitfalls.

20. **Interview Scenarios & Whiteboard Drills**  
    Practical Q&A, design discussions, and real-world troubleshooting cases.

---

📌 Each chapter will include:
- 🔑 **Concept explanation** (in simple words)  
- 💻 **Commands/examples**  
- 🛠️ **Troubleshooting notes**  

This roadmap ensures we build RAC knowledge step by step — from **core fundamentals → real-time troubleshooting → interview readiness**.
