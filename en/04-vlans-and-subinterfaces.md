
## **4 - VLANs and Subinterfaces**


## 4.1 - Introduction

VLANs (Virtual Local Area Networks) allow dividing the network into smaller logical segments, ensuring better organization and management. In this project, there are seven VLANs – **Admin-Server, Sales, Finance, Logistics, Management, Warehouse, and Guest.** Each VLAN has its own subnet and default gateway, which separates traffic between different parts of the company.

In this chapter, VLANs are created on switches, trunk ports are configured, subinterfaces are set up on routers using the **Router-on-a-Stick** method, OSPF is extended with new VLANs, and **static IP addresses** are configured for servers and the admin PC.

**The goal of this chapter** is to create VLANs on switches, configure trunk links to routers, set up subinterfaces on routers R2 and R3, assign IP addresses as default gateways, integrate VLANs into OSPF, and configure static addresses for key devices.


## 4.2 - VLAN Creation on Switches

In this section, all required VLANs are created on switches **SW1** and **SW2** and assigned names for better clarity. Creating VLANs is the first step before assigning them to ports.

|VLAN ID|VLAN Name|Description / Purpose|
|---|---|---|
|10|Admin-Server|Servers, Admin PC, internal Wi-Fi (Access Point)|
|20|Sales|User PCs and network printer of the Sales department|
|30|Finance|PCs for accounting|
|40|Logistics|PCs for warehouse and logistics management|
|50|Management|PCs of company management|
|60|Warehouse|PCs in the warehouse|
|70|Guest|Guest network with internet-only access|


---

### Switch SW1

- Creation of VLANs 10–50.
    
- Assigning names based on the purpose of each VLAN.
    

**Configuration in CLI:**

```
enable
configure terminal
vlan 10
name Admin-Server
vlan 20
name Sales
vlan 30
name Finance
vlan 40
name Logistics
vlan 50
name Management
exit
end
```
![name-vlan-SW1](../images/Pasted%20image%2020250914133538.png)

---

### Switch SW2

- Created VLAN 60 and 70.
    
- Assigned names according to the purpose of each VLAN.
    

**Configuration in CLI:**

```
enable
configure terminal
vlan 60
name Warehouse
vlan 70
name Guest
exit
end
write memory
```
![name-vlan-SW2](../images/Pasted%20image%2020250914133918.png)

## 4.3 - Assigning Ports to VLANs

After creating VLANs on the switches, it is necessary to assign physical ports to individual VLANs. Each port connected to an end device (PC, server, printer, or Access Point) is configured as an access port and placed into the corresponding VLAN. This ensures that every device is part of the correct logical segment.

|Port SW1|VLAN ID|Assigned Device|
|---|---|---|
|Fa0/1|10|Admin PC|
|Fa0/12|10|Server SRV01|
|Gi0/1|10|Server SRV02|
|Fa0/14|10|Access Point|
|Fa0/2|20|PC2|
|Fa0/3|20|PC3|
|Fa0/13|20|Printer|
|Fa0/4|30|PC4|
|Fa0/5|40|PC5|
|Fa0/6|40|PC6|
|Fa0/7|50|PC7|
|Fa0/8|50|PC8|

|Port SW2|VLAN ID|Assigned Device|
|---|---|---|
|Fa0/9|60|PC9|
|Fa0/10|60|PC10|
|Fa0/11|70|PC11|

---

### Switch SW1

- Switch ports are set to **access mode**.
    
- Ports are assigned to the correct VLANs based on the connected devices.


**Configuration in CLI:**

```
enable
configure terminal
interface fa0/1
switchport mode access
switchport access vlan 10
exit
interface fa0/12
switchport mode access
switchport access vlan 10
exit
interface gig0/1
switchport mode access
switchport access vlan 10
exit
interface fa0/14
switchport mode access
switchport access vlan 10
exit
interface fa0/2
switchport mode access
switchport access vlan 20
exit
interface fa0/3
switchport mode access
switchport access vlan 20
exit
interface fa0/13
switchport mode access
switchport access vlan 20
exit
interface fa0/4
switchport mode access
switchport access vlan 30
exit
interface fa0/5
switchport mode access
switchport access vlan 40
exit
interface fa0/6
switchport mode access
switchport access vlan 40
exit
interface fa0/7
switchport mode access
switchport access vlan 50
exit
interface fa0/8
switchport mode access
switchport access vlan 50
exit
end
write memory
```
![vlan-access-SW1](../images/Pasted%20image%2020250914163738.png)

---

### Switch SW2

- Switch ports are set to access mode.
    
- Ports are assigned to the correct VLANs for warehouse and guest devices.
    

**Configuration in CLI:**

```
enable
configure terminal
interface fa0/9
switchport mode access
switchport access vlan 60
exit
interface fa0/10
switchport mode access
switchport access vlan 60
exit
interface fa0/11
switchport mode access
switchport access vlan 70
exit
end
write memory
```
![vlan-access-SW2](../images/Pasted%20image%2020250914134422.png)


### Conclusion

Within sections **4.2** and **4.3**, VLANs with assigned names were created on switches **SW1** and **SW2**, and the ports were then configured for end devices. The configuration on both switches is verified using the commands `show vlan brief` and `show running-config`, which confirm the correct creation of VLANs and the assignment of ports according to the design.

## 4.4 – Trunk Configuration on Switches

Trunk ports allow the transfer of frames from multiple VLANs over a single physical interface. In this section, trunk ports are configured on the switches to interconnect them with routers. The 802.1Q encapsulation and the configuration of subinterfaces will be addressed in the following section (4.5).

|Link|Switch Interface|Router Interface|Mode|
|---|---|---|---|
|SW1–R2|Gi0/2|Gi0/1|trunk|
|SW2–R3|Gi0/1|Gi0/2|trunk|

---

### Switch SW1

- Switch port **Gi0/2** is set to trunk mode.
    

**Configuration in CLI:**

```
enable
configure terminal
interface gi0/2
switchport mode trunk
exit
end
write memory
```
![trunk-SW1](../images/Pasted%20image%2020250914142255.png)


---


### Switch SW2

- Switch port **Gi0/1** is switched to trunk mode.
    

**Configuration in CLI:**

```
enable
configure terminal
interface gi0/1
switchport mode trunk
exit
end
write memory
```
![trunk-SW2](../images/Pasted%20image%2020250914142428.png)

### Conclusion

In section 4.4, trunk ports were configured on switches SW1 and SW2 to interconnect the switches with the routers. To verify the setup, the command `show interfaces trunk` is used, confirming that the trunks are configured correctly and are carrying frames of all required VLANs.

## 4.5 - Subinterface Configuration on Routers (Router-on-a-Stick)

To enable routing between individual VLANs, subinterfaces are created on routers **R2 and R3**. Each subinterface is assigned to a specific VLAN, configured with **802.1Q encapsulation**, and given an **IP address** that acts as the default gateway for that VLAN.

|Router|Subinterface|VLAN ID|IP Address (gateway)|Subnet Mask|
|---|---|---|---|---|
|R2|Gi0/1.10|10|10.31.10.1|255.255.255.0|
|R2|Gi0/1.20|20|10.31.20.1|255.255.255.0|
|R2|Gi0/1.30|30|10.31.30.1|255.255.255.0|
|R2|Gi0/1.40|40|10.31.40.1|255.255.255.0|
|R2|Gi0/1.50|50|10.31.50.1|255.255.255.0|
|R3|Gi0/2.60|60|10.31.60.1|255.255.255.0|
|R3|Gi0/2.70|70|10.31.70.1|255.255.255.0|

---

### Router R2

**Configuration steps:**

- Create a subinterface for each VLAN.
    
- Configure 802.1Q encapsulation (`encapsulation dot1q <VLAN>`).
    
- Assign an IP address to serve as the default gateway for the VLAN.

---

**Configuration in CLI:**

```
enable
configure terminal
interface gi0/1.10
encapsulation dot1q 10
ip address 10.31.10.1 255.255.255.0
no shutdown
exit
interface gi0/1.20
encapsulation dot1q 20
ip address 10.31.20.1 255.255.255.0
no shutdown
exit
interface gi0/1.30
encapsulation dot1q 30
ip address 10.31.30.1 255.255.255.0
no shutdown
exit
interface gi0/1.40
encapsulation dot1q 40
ip address 10.31.40.1 255.255.255.0
no shutdown
exit
interface gi0/1.50
encapsulation dot1q 50
ip address 10.31.50.1 255.255.255.0
no shutdown
exit
end
write memory
```
![encapsulation-R2](../images/Pasted%20image%2020250914150123.png)

---


### Router R3


**Configuration in CLI:**

```
enable
configure terminal
interface gi0/2.60
encapsulation dot1q 60
ip address 10.31.60.1 255.255.255.0
no shutdown
exit
interface gi0/2.70
encapsulation dot1q 70
ip address 10.31.70.1 255.255.255.0
no shutdown
exit
end
write memory
```
![encapsulation-R3](../images/Pasted%20image%2020250914150557.png)

### Conclusion

On routers R2 and R3, subinterfaces are created with 802.1Q encapsulation and assigned IP addresses (gateway/VLAN). The functionality is verified using diagnostics (`show ip interface brief`, `show running-config interface …`) and test pings to the gateways of individual VLANs. All tests were successful.

## 4.6 - Adding VLAN Networks to OSPF

To enable communication between VLANs located behind different routers, newly created VLAN networks on routers R2 and R3 are added into OSPF. This allows the networks behind R2 and R3 to communicate with each other. The backbone OSPF adjacency remains unchanged according to the previous configuration.

|Router|Networks added to OSPF|Area|
|---|---|---|
|R2|10.31.10.0/24, 10.31.20.0/24, 10.31.30.0/24, 10.31.40.0/24, 10.31.50.0/24|0|
|R3|10.31.60.0/24, 10.31.70.0/24|0|


### Router R2

**Configuration steps:**

- On each router, add the corresponding VLAN networks into OSPF.
    
- Subinterfaces remain passive (they do not form adjacencies in user VLANs); only the backbone P2P link between R2 and R3 stays active as described in Chapter 3.
    

**Configuration in CLI:**

```
enable
configure terminal
router ospf 1
network 10.31.10.0 0.0.0.255 area 0
network 10.31.20.0 0.0.0.255 area 0
network 10.31.30.0 0.0.0.255 area 0
network 10.31.40.0 0.0.0.255 area 0
network 10.31.50.0 0.0.0.255 area 0
exit
end
write memory
```
![vlan-ospf](../images/Pasted%20image%2020250914184334.png)


### Router R3

**Configuration in CLI:**

```
enable
configure terminal
router ospf 1
network 10.31.60.0 0.0.0.255 area 0
network 10.31.70.0 0.0.0.255 area 0
exit
end
write memory
```
![vlan-ospf](../images/Pasted%20image%2020250914184504.png)



### Conclusion

OSPF for VLAN networks has been configured. Diagnostics using the command `show ip ospf neighbor` confirm established **neighbor relationships**, and `show ip route ospf` displays the added routes to VLAN networks on the second router. **Test pings to the gateways of remote VLANs are successful.** The result is that VLAN networks behind R2 and R3 recognize each other and are fully reachable.

## 4.7 - Static IP Addresses for Servers and Admin PC (VLAN 10)

In VLAN 10, key devices are assigned **static IP addresses** to ensure they remain stably available for management and provided services. A static IP is also assigned to the printer in VLAN 20. All other workstations in the network remain on DHCP.

|Device|Interface|IP Address|Mask|Default Gateway|
|---|---|---|---|---|
|Server SRV01 (DHCP, Syslog)|Gi1|10.31.10.10|255.255.255.0|10.31.10.1|
|Server SRV02 (DNS, HTTP)|Gi1|10.31.10.11|255.255.255.0|10.31.10.1|
|Admin PC|Fa0|10.31.10.20|255.255.255.0|10.31.10.1|

>**Note:** DNS for these devices will be configured in the DNS setup in the following chapter.

---

**Configuration Steps>**

* **Server SRV01** -> Desktop -> IP Configuration -> IP address, subnet mask, default gateway


![static-SRV01](../images/Pasted%20image%2020250914203000.png)


* **Server SRV02** -> Desktop -> IP Configuration -> IP address, subnet mask, default gateway


![static-SRV02](../images/Pasted%20image%2020250914203020.png)


* **Admin PC1** -> Desktop -> IP Configuration -> IP address, subnet mask, default gateway


![static-PC1](../images/Pasted%20image%2020250914203040.png)


### Printer VLAN 20

The **network printer** was configured with the following parameters:

- **IP address:** 10.31.20.10
    
- **Subnet mask:** 255.255.255.0 (/24)
    
- **Assigned network:** 10.31.20.0/24
    
- **Default gateway:** 10.31.20.1
    

### Conclusion

Static IP addresses for **SRV01**, **SRV02**, **Admin PC**, and the printer are now configured. The functionality is verified through diagnostics -> **pings from R2 to devices in VLAN 10 are successful**.



## 4.8 - Summary

In this part, VLANs were created and assigned to ports for individual devices on switches SW1 and SW2. Trunk ports were configured to allow multiple VLAN frames to pass over physical links, and subinterfaces with 802.1Q encapsulation and IP addresses as gateways were set up on routers R2 and R3.

The new VLAN networks were then integrated into the OSPF routing protocol, enabling communication across different routers. Key devices in VLAN 10 (servers and Admin PC) and the printer in VLAN 20 were given static IP addresses. The configuration was tested with diagnostics, and all ping tests completed successfully without packet loss.


---

**Next chapter: Network Services.** [Network Services](05-network-services.md)  





























































