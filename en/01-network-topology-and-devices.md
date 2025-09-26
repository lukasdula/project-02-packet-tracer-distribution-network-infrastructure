
# 1 - Network Topology and Used Devices

## 1.1 - Introduction

This project presents a medium-scale model of a company **distribution network**, designed to simulate real traffic and extended with multiple VLANs, dedicated routers, and separated zones.  
The network is structured to demonstrate the principles of segmentation, routing, and security in an environment corresponding to the **CCNA I level.**

The topology includes seven **VLANs,** which separate different parts of the company. Each VLAN has its own address space and a default gateway configured on router R2 or R3.  
This arrangement increases the overall level of security and allows centralized management of the entire network.

![Typology-map](../images/Pasted%20image%2020250924020629.png)

**The network is divided into nine logical zones:**

1. **ISP / Internet Zone** – the internet provider (ISP router) and the border router R1, which connects the network to the internet and performs NAT/PAT.
    
2. **Internal Switching & Routing Zone** – the central part formed by routers R2 and R3 with two switches. It handles inter-VLAN routing, traffic forwarding, and configuration of the main network functions, including centralized management and security measures.
    
3. **Admin-Server (VLAN 10, SW1)** – servers running services (DHCP, DNS, HTTP, Syslog) and the administrator’s PC station managing the entire company network, including internal Wi-Fi for employees.
    
4. **Sales (VLAN 20, SW1)** – office workspace with user PCs and a network printer for the sales department.
    
5. **Finance (VLAN 30, SW1)** – a separate finance department with its own workstation PCs.
    
6. **Logistics (VLAN 40, SW1)** – office workspace dedicated to warehouse and logistics management.
    
7. **Management (VLAN 50, SW1)** – two workstation PCs for the company’s executive management.
    
8. **Warehouse (VLAN 60, SW2)** – the company’s logistics warehouse with two PCs.
    
9. **Guest (VLAN 70, SW2)** – a guest network that is not used for work purposes and provides internet access only.
    
## 1.2 - Used Devices

| Device Type         | Model           | Network Usage                                                                                                |
| ------------------- | --------------- | ------------------------------------------------------------------------------------------------------------ |
| **Router R1**       | Cisco 2911      | Border connection with ISP; NAT/PAT for outgoing traffic; internet routing. Connected to R2 and ISP router.  |
| **Router R2**       | Cisco 2911      | Internal inter-VLAN routing; default gateway for VLANs 10/20/30/40/50; connected to R1 and R3; trunk to SW1. |
| **Router R3**       | Cisco 2911      | Routing for VLAN 60/70; connected to R2; trunk to SW2.                                                       |
| **ISP Router**      | Cisco 2911      | Simulated ISP router (WAN/Internet).                                                                         |
| **Switch SW1**      | Cisco 2960-24TT | Connection of servers, admin PC, and office PCs; access ports for VLANs 10/20/30/40/50; trunk to R2.         |
| **Switch SW2**      | Cisco 2960-24TT | Connection of PCs in VLAN 60/70; access ports for VLAN 60/70; trunk to R3.                                   |
| **Server SRV01**    | Server-PT       | DHCP and Syslog server in VLAN 10; static IP.                                                                |
| **Server SRV02**    | Server-PT       | DNS and HTTP server in VLAN 10; static IP.                                                                   |
| **Admin PC**        | PC-PT           | Administrator PC in VLAN 10; static IP; manages the entire network.                                          |
| **Access Point**    | AccessPoint-PT  | Wi-Fi access in VLAN 10; IP from DHCP.                                                                       |
| **Sales PCs**       | PC-PT           | Two office PCs in VLAN 20; IP from DHCP.                                                                     |
| **Sales Laptop**    | Laptop-PT       | Portable laptop in VLAN 10 (Wi-Fi access); IP from DHCP.                                                     |
| **Printer (Sales)** | Printer-PT      | Network printer in VLAN 20; static IP.                                                                       |
| **Finance PC**      | PC-PT           | One workstation in VLAN 30; IP from DHCP.                                                                    |
| **Logistics PCs**   | PC-PT           | Two workstations in VLAN 40; IP from DHCP.                                                                   |
| **Management PCs**  | PC-PT           | Two executive PCs in VLAN 50; IP from DHCP.                                                                  |
| **Warehouse PCs**   | PC-PT           | Two warehouse PCs in VLAN 60; IP from DHCP.                                                                  |
| **Guest PC**        | PC-PT           | One guest workstation in VLAN 70; IP from DHCP; internet access only.                                        |
Straight-through copper cables are used throughout the network, as they are the standard choice for connecting different types of devices.

## 1.3 - Topology Overview – Physical Connections

| Device               | Interface       | Connected to->             | Peer Interface | Note                                              |
| -------------------- | --------------- | -------------------------- | -------------- | ------------------------------------------------- |
| **ISP Router**       | Gi0/0           | Router R1                  | Gi0/1          | WAN 100.64.5.0/30 (ISP=100.64.5.1, R1=100.64.5.2) |
| **Router R1 (Edge)** | Gi0/2           | Router R2                  | Gi0/0          | P2P 10.31.0.0/30 (R1=10.31.0.1, R2=10.31.0.2)     |
| **Router R2 (Core)** | Gi0/2           | Router R3                  | Gi0/1          | P2P 10.31.1.0/30 (R2=10.31.1.1, R3=10.31.1.2)     |
| **Router R2 (Core)** | Gi0/1           | Switch SW1                 | Gi0/2          | TRUNK (VLAN 10,20,30,40,50)                       |
| **Router R3**        | Gi0/2           | Switch SW2                 | Gi0/1          | TRUNK (VLAN 60,70)                                |
| **Switch SW1**       | Fa0/14          | Access Point (AP-WIFI)     | port0          | VLAN 10 (AP z DHCP)                               |
| **Switch SW1**       | Fa0/12          | Server SRV01 (DHCP+Syslog) | Gig1           | VLAN 10 (statická IP)                             |
| **Switch SW1**       | Gig0/1          | Server SRV02 (DNS+HTTP)    | Gig1           | VLAN 10 (statická IP)                             |
| **Switch SW1**       | Fa0/1           | PC-1 (Admin)               | Fa0            | VLAN 10 (statická IP)                             |
| **Switch SW1**       | Fa0/2           | PC-2 (SALES)               | Fa0            | VLAN 20 (DHCP)                                    |
| **Switch SW1**       | Fa0/3           | PC-3 (SALES)               | Fa0            | VLAN 20 (DHCP)                                    |
| **Switch SW1**       | Fa0/10          | Printer (SALES)            | Fa0            | VLAN 20 (statická IP)                             |
| **Switch SW1**       | _Wi-Fi přes AP_ | Laptop (Admin-Server)      | wireless       | VLAN 10 (DHCP přes AP)                            |
| **Switch SW1**       | Fa0/4           | PC-4 (FINANCE)             | Fa0            | VLAN 30 (DHCP)                                    |
| **Switch SW1**       | Fa0/5           | PC-5 (LOGISTICS)           | Fa0            | VLAN 40 (DHCP)                                    |
| **Switch SW1**       | Fa0/6           | PC-6 (LOGISTICS))          | Fa0            | VLAN 40 (DHCP)                                    |
| **Switch SW1**       | Fa0/7           | PC-7 (MANAGEMENT)          | Fa0            | VLAN 50 (DHCP)                                    |
| **Switch SW1**       | Fa0/08          | PC-8 (MANAGEMENT)          | Fa0            | VLAN 50 (DHCP)                                    |
| **Switch SW2**       | Fa0/9           | PC-9 (WAREHOUSE)           | Fa0            | VLAN 60 (DHCP)                                    |
| **Switch SW2**       | Fa0/10          | PC-10 (WAREHOUSE)          | Fa0            | VLAN 60 (DHCP)                                    |
| **Switch SW2**       | Fa0/11          | PC-11 (GUEST)              | Fa0            | VLAN 70 (DHCP)                                    |


---

## 1.4 - Summary

This introductory chapter presents the topology of the distribution company network at the **CCNA I** level, describing the used devices and physical connections including ports.  
These elements form the foundation for further configuration and the logical division of the network.

**Continue to the next chapter:** Addressing and VLAN Planning
