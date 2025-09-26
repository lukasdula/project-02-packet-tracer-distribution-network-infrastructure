
# 2 - Addressing and VLAN Planning

## 2.1 - Introduction

This chapter defines the logical division of the entire network and assigns **IP addresses** to individual interfaces.  
It also includes the design of **VLAN segmentation**, which determines how devices are grouped into separate logical units according to their function and location.

**The addressing and VLAN plan** forms the basic structure of the network, which is then followed by the configuration of routing, switching, and network services in the next chapters.


## 2.2 - IP Addressing and Subnet Overview

| Device / Interface   | Description        | IP Address                      | Subnet Mask     | Default Gateway | Assigned Network |
| -------------------- | ------------------ | ------------------------------- | --------------- | --------------- | ---------------- |
| **ISP Router Gi0/0** | ISP -> R1          | 100.64.5.1                      | 255.255.255.252 | N/A             | 100.64.5.0/30    |
| **Router R1 Gi0/1**  | R1 -> ISP          | 100.64.5.2                      | 255.255.255.252 | N/A             | 100.64.5.0/30    |
| **Router R1 Gi0/2**  | Link R1 -> R2      | 10.31.0.1                       | 255.255.255.252 | N/A             | 10.31.0.0/30     |
| **Router R2 Gi0/0**  | Link R2 -> R1      | 10.31.0.2                       | 255.255.255.252 | N/A             | 10.31.0.0/30     |
| **Router R2 Gi0/2**  | LinkR2 -> R3       | 10.31.1.1                       | 255.255.255.252 | N/A             | 10.31.1.0/30     |
| **Router R3 Gi0/1**  | Link R3 -> R2      | 10.31.1.2                       | 255.255.255.252 | N/A             | 10.31.1.0/30     |
| **SRV01**            | Server VLAN 10     | 10.31.10.10                     | 255.255.255.0   | 10.31.10.1      | 10.31.10.0/24    |
| **SRV02**            | Server VLAN 10     | 10.31.10.11                     | 255.255.255.0   | 10.31.10.1      | 10.31.10.0/24    |
| **PC1**              | Admin PC (VLAN 10) | 10.31.10.20                     | 255.255.255.0   | 10.31.10.1      | 10.31.10.0/24    |
| **PC2 / PC3**        | Sales VLAN 20      | DHCP 10.31.20.20 – 10.31.20.49, | 255.255.255.0   | 10.31.20.1      | 10.31.20.0/24    |
| **PC4**              | Finance VLAN 30    | DHCP 10.31.30.30 – 10.31.30.59  | 255.255.255.0   | 10.31.30.1      | 10.31.30.0/24    |
| **PC5 / PC6**        | Logistics VLAN 40  | DHCP 10.31.40.40 – 10.31.40.69  | 255.255.255.0   | 10.31.40.1      | 10.31.40.0/24    |
| **PC7 / PC8**        | Management VLAN 50 | DHCP 10.31.50.50 – 10.31.50.79  | 255.255.255.0   | 10.31.50.1      | 10.31.50.0/24    |
| **PC9 / PC10**       | Warehouse VLAN 60  | DHCP 10.31.60.60 – 10.31.60.89  | 255.255.255.0   | 10.31.60.1      | 10.31.60.0/24    |
| **PC11**             | Guest VLAN 70      | DHCP 10.31.70.70 – 10.31.70.99  | 255.255.255.0   | 10.31.70.1      | 10.31.70.0/24    |
| Printer              | Sales VLAN 20      | 10.31.20.10                     | 255.255.255.0   | 10.31.20.1      | 10.31.20.0/24    |


>**Note:** The Admin PC uses a **static IP address**, ensuring that the address never changes.  
This simplifies management, security, and the configuration of access rules.

## 2.3 - VLAN Planning

For this project, all workstations, servers, and other devices are divided into separate VLANs.  
Each VLAN has its own IP subnet with a /24 mask (255.255.255.0) and a default gateway configured on the appropriate router (R2 or R3).

**This segmentation allows to: **

- increase security by isolating departments and zones,  
    
- efficiently route traffic between VLANs through routers R2 and R3,  
    
- simplify management of access rules and internal network services (DHCP, DNS, HTTP, Syslog).
    

The DHCP server dynamically assigns addresses for VLANs 20 to 70.  
In VLAN 10 (Admin & Server Zone), key devices are configured with static addresses.

| VLAN ID | VLAN Name     | Description / Purpose                                   | Assigned Network | Default Gateway | Assigned Devices                |
|---------|---------------|---------------------------------------------------------|------------------|-----------------|---------------------------------|
| 10      | Admin-Server  | Servers (DHCP, DNS, HTTP, Syslog), admin PCs, internal Wi-Fi | 10.31.10.0/24   | 10.31.10.1      | SRV01, SRV02, PC1, Access Point |
| 20      | Sales         | User PCs and printer for the sales department           | 10.31.20.0/24   | 10.31.20.1      | PC2, PC3, Printer               |
| 30      | Finance       | Dedicated PCs for accounting                            | 10.31.30.0/24   | 10.31.30.1      | PC4                             |
| 40      | Logistics     | PCs for logistics and warehouse management              | 10.31.40.0/24   | 10.31.40.1      | PC5, PC6                        |
| 50      | Management    | Workstations for company management                     | 10.31.50.0/24   | 10.31.50.1      | PC7, PC8                        |
| 60      | Warehouse     | PCs in the warehouse                                    | 10.31.60.0/24   | 10.31.60.1      | PC9, PC10                       |
| 70      | Guest Zone    | Guest network with internet access only                 | 10.31.70.0/24   | 10.31.70.1      | PC11                            |


## 2.4 - Summary


This part of the project defines the detailed addressing plan for the entire network and divides it into separate VLANs.  
Each VLAN has its own IP subnet with a defined default gateway, and client workstations are assigned addresses from the DHCP pool.  
Static addresses are reserved only for servers, network services in VLAN 10 (Admin-Server), and the printer.

The IP addressing and VLAN segmentation design is a key element for the subsequent configuration of routing, switching, and network services.


---

**Continue to the next chapter:** [Basic Network Configuration](03-basic-network-configuration.md)  
