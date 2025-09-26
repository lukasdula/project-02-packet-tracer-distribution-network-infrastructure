# **Distribution Network Infrastructure**

## Introduction and Objectives

This project is part of my study portfolio and practical preparation for the CCNA I course.  
It indirectly follows the first project, where a network for a smaller company was built. Compared to that, this project is goes one step further and represents a more complex medium-sized business environment with broader deployment of routing, services, and security mechanisms.

The network is implemented in Cisco Packet Tracer using CLI. Communication between departments is managed through VLAN segmentation and OSPF routing. One of the routers functions as an ISP, complementing the setup with a realistic corporate topology. Servers provide the necessary infrastructure for the entire network and communication between departments. A more complex ACL policy regulates access between VLANs and simulates rules corresponding to real-world business requirements.

The network is systematically divided into several zones that reflect common enterprise infrastructure:

- **ISP Zone** -> simulation of the internet provider  
    
- **Routing and Switching Zone** -> interconnection of routers, switches, and backbone infrastructure  
    
- **Server Zone** -> SRV01 (DHCP, Syslog), SRV02 (DNS, HTTP, NTP), and Admin PC  
    
- **Company VLANs** -> Admin-Server, Sales, Finance, Logistics, Management, Warehouse, Guest

## Project Structure

1. [Network Topology and Used Devices](01-network-topology-and-devices.md)  
    
2. [Addressing and VLAN Planning](02-addressing-and-vlan-planning.md)  
    
3. [Basic Network Configuration](03-basic-network-configuration.md)  
    
4. [VLANs and Subinterfaces](04-vlans-and-subinterfaces.md)  
    
5. [Network Services](05-network-services.md)  
    
6. [Network Security](06-network-security.md)  
    
7. [Connectivity Testing](07-connectivity-testing.md)  
    
8. [Troubleshooting](08-troubleshooting.md)  
    
9. [Summary and Conclusion](09-conclusion-and-summary.md)

## Access to CLI Devices

For demonstration purposes, login credentials were configured for network devices.  
These credentials allow access to the CLI and subsequently to privileged mode.

- **Username (console + SSH):** distadmin  
- **User password:** Logistic25  
- **Enable secret (privileged mode):** Pallet25  

For the purpose of this project, simplified thematic passwords are used.  
In a real deployment, more complex and longer passwords would be applied (12+ characters, combination of uppercase/lowercase letters, numbers, and special characters).

---

## Used Tools

- **Cisco Packet Tracer (version 8.2+)** -> network simulation environment  
- **Visual Studio Code / Obsidian** -> writing and editing documentation

## How to Run the Project

1. Open the `.pkt` file in Cisco Packet Tracer (version 8.2+).  
2. Follow the project folders 01–09.

---

## Key Project Features

- OSPF routing between routers -> dynamic route sharing  
    
- Subnetting with /30 masks for efficient address utilization  
    
- Extended VLAN segmentation -> separation of company departments  
    
- Management VLAN 99 -> dedicated switch management via SSH  
    
- DHCP, DNS, HTTP, NTP, and Syslog services  
    
- ACL policy -> detailed and realistic restrictions of inter-VLAN traffic  
    
- Network diagnostics -> systematic verification of connectivity and service availability  
    
- Troubleshooting demonstrations -> example solutions to common issues
    

## Author's Note

I really enjoyed working on this second project. Thanks to the experience from the first one, I could better understand how networks work and apply my knowledge to a more complex design.  
For the first time, I tried configuring OSPF and saw how routing improves communication between routers.

A big step forward was using ACL policies, which made the network look more like a real company network with clear access rules.  
I also added VLAN 99 for secure switch management, and new services like NTP and Syslog. Subnetting became an important tool for me, giving me more confidence in addressing and planning the network.

This project shows that even a medium-size network can include many features and still work as one stable system.  
For the next step, I want to create an **enterprise network project**, connecting two branches with GRE tunnels, adding more services, more devices, and a detailed example of Wi-Fi deployment.


---

© 2025 – Lukáš Dula | Home Network Lab & Portfolio
