
# 9 - Summary and Conclusion

## Final Evaluation

The project represents the design of a **medium-sized company distribution network** with four routers (one acting as ISP), two switches, and servers. The network is routed using **OSPF**, backbone links use /30 masks, and VLAN segmentation separates departments into independent networks. NAT/PAT and internet connectivity were not included – instead, DNS and HTTP function locally within the company.

The infrastructure functionality is expanded compared to the first project with more VLANs, as well as services such as **NTP** for time and **Syslog** for logs (limited in PT), and **management VLAN 99**, which enables secure switch management over SSH. Security is based on user accounts, enable secret, and MOTD banners, while port security and administrative port blocking were kept from the first project.

Significant emphasis was placed on the **ACL policy**. The Finance VLAN is isolated, the Guest VLAN has only ISP access, and Admin and Management VLANs have full privileges. ACLs and their diverse rules cover DHCP, servers, printers, and inter-VLAN communication, simulating a realistic corporate policy. ISP and Wi-Fi were included only as a supplemental demonstration.

Testing and troubleshooting confirmed the correct functionality of all components – backbone links, routing, services, and security policies. The network is stable, fully functional, and matches the model of a real medium-sized corporate infrastructure.

## Project main steps overview

- **Chapter 1:** Topology design -> four routers (including ISP), two switches, servers, description of physical connections and devices used.
    
- **Chapter 2:** Addressing and VLAN plan -> definition of the address space, use of /30 masks for backbone links, network segmentation into multiple VLANs for each department.
    
- **Chapter 3:** Basic configuration -> device naming, IP address assignment, interface configuration, and router-switch interconnections.
    
- **Chapter 4:** VLANs and subinterfaces -> creation of VLANs, port assignment, trunk configuration, and Router-on-a-Stick setup on routers.
    
- **Chapter 5:** Network services -> DHCP and Syslog on SRV01, DNS and HTTP on SRV02, implementation of an NTP server, and basic Wi-Fi access for the administrator.
    
- **Chapter 6:** Security -> user accounts and passwords, enable secret, SSH access, MOTD banners, dedicated management VLAN 99, and detailed ACL configuration.
    
- **Chapter 7:** Connectivity testing -> verification of router links using pings to IP addresses and ports, OSPF routing checks, VLAN, DHCP, DNS/HTTP, NTP functions, and ACL policy validation.
    
- **Chapter 8:** Troubleshooting -> examples of fixing OSPF, VLAN, DHCP, and ACL issues, configuration correction, and confirmation of full network functionality.
    

---

**Back to project overview:** [00-README](00-README.en.md)
