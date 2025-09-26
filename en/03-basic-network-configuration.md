# **3 - Basic Network Configuration**

## 3.1 - Introduction

In this chapter, **hostnames** are assigned to all network devices (routers, switches, servers) to keep the configuration clear.  
Next, **IP addresses** are configured on the physical router interfaces according to the addressing plan, creating the basic connectivity between **R1, R2, R3**, and the **ISP.**  
**This step is necessary for deploying the OSPF protocol, which ensures dynamic route sharing between routers.

**Static IP addresses** are assigned only to servers and the administrator PC in **VLAN 10** and the printer.  
All other devices are managed through **DHCP**.

## 3.2 - Device Naming

In this section, hostnames are configured on all network devices with CLI access.  
This ensures better orientation in the network and simplifies later configuration and management through the CLI environment.

| Device     | Hostname |
|------------|----------|
| ISP Router | ISP      |
| Router R1  | R1       |
| Router R2  | R2       |
| Router R3  | R3       |
| Switch SW1 | SW1      |
| Switch SW2 | SW2      |

### ISP Router

- **Hostname:** ISP  

**Configuration in CLI:**


```
enable  
configure terminal  
hostname ISP  
exit  
write memory
```
![ISP-hostname](../images/20250913210024.png)

### Router R1

- **Hostname:** R1
    

**Configuration in CLI:**

```
enable
configure terminal
hostname R1
exit
write memory
```
![R1-hostname](../images/Pasted%20image%2020250913211636.png)

---

### Router R2

- **Hostname:** R2
    

**Configuration in CLI:**

```
enable
configure terminal
hostname R2
exit
write memory
```
![R2-hostname](../images/Pasted%20image%2020250913211932.png)

### Router R3

- **Hostname:** R3
    

**Configuration in CLI:**

```
enable
configure terminal
hostname R3
exit
write memory
```
![R3-hostname](../images/Pasted%20image%2020250913212122.png)

---

### Switch SW1

- **Hostname:** SW1
    

**Configuration in CLI:**

```
enable
configure terminal
hostname SW1
exit
write memory
```
![SW-hostname](../images/Pasted%20image%2020250913212456.png)

### Switch SW2

- **Hostname:** SW2
    

**Configuration in CLI:**

```
enable
configure terminal
hostname SW2
exit
write memory
```
![SW2-hostname](../images/Pasted%20image%2020250913212625.png)

---

### Conclusion

Each device with CLI access receives a unique **hostname**. This ensures clear configuration and simplifies network management. The configured hostname is stored in both the running and startup configuration, and remains preserved even after the device restarts.


## 3.3 - Configuring IP Addresses on Router Interfaces

### ISP Router

- **Port:** GigabitEthernet0/0 (connection to R1)
    
- **IP address:** `100.64.5.1 255.255.255.252`
    

**Configuration in CLI:**

```
enable
configure terminal
interface GigabitEthernet0/0
ip address 100.64.5.1 255.255.255.252
no shutdown
exit
end
write memory
```
![IP-ISP](../images/Pasted%20image%2020250913225757.png)

### Router R1

- **Port:** `GigabitEthernet0/1` (connection to ISP)
    
- **IP address:** `100.64.5.2 255.255.255.252`
    
- **Port:** `GigabitEthernet0/2` (connection to R2)
    
- **IP address:** `10.31.0.1 255.255.255.252`
    

**Configuration in CLI:**

```
enable
configure terminal
interface GigabitEthernet0/1
ip address 100.64.5.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/2
ip address 10.31.0.1 255.255.255.252
no shutdown
exit
end
write memory
```
![IP-R1](../images/Pasted%20image%2020250913215512.png)


### Router R2

- **Port:** `GigabitEthernet0/0` (connection to R1)
    
- **IP address:** `10.31.0.2 255.255.255.252`
    
- **Port:** `GigabitEthernet0/1` (connection to SW1, trunk for VLAN 10,20,30,40,50)
    
- **IP address:** not assigned
    
- **Port:** `GigabitEthernet0/2` (connection to R3)
    
- **IP address:** `10.31.1.1 255.255.255.252`
    

**Configuration in CLI:**

```
enable
configure terminal
interface GigabitEthernet0/0
ip address 10.31.0.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/1
no ip address
no shutdown
exit
interface GigabitEthernet0/2
ip address 10.31.1.1 255.255.255.252
no shutdown
exit
end
write memory
```
![IP-R2](../images/Pasted%20image%2020250913224133.png)


### Router R3

- **Port:** `GigabitEthernet0/1` (connection to R2)
    
- **IP address:** `10.31.1.2 255.255.255.252`
    
- **Port:** `GigabitEthernet0/2` (connection to SW2, trunk for VLAN 60,70)
    
- **IP address:** not assigned
    

**Configuration in CLI:**

```
enable
configure terminal
interface GigabitEthernet0/1
ip address 10.31.1.2 255.255.255.252
no shutdown
exit
interface GigabitEthernet0/2
no ip address
no shutdown
exit
end
write memory
```
![IP-R3](../images/Pasted%20image%2020250913225504.png)


### Conclusion

On routers, IP addresses are assigned only to backbone interfaces, enabling direct communication between neighboring devices. Interfaces facing switches remain without addresses, as they will later be used for trunks and VLAN subinterfaces. This setup forms the basis for subsequent OSPF routing.

---

## 3.4 - OSPF

In this section, dynamic routing is activated on routers using the **OSPF protocol**. OSPF allows routers to automatically exchange information about connected networks and route packets not only to their direct neighbors but also to more distant parts of the topology.

At this stage, only backbone point-to-point links between routers are added to OSPF to form the primary network backbone. 

VLANs and their subnets will be added to OSPF later, after creating subinterfaces and trunks.

---

### ISP Router - OSPF

- The network `100.64.5.0/30` is added to OSPF on the interface facing R1.
    
- This allows the ISP to establish adjacency with R1 and advertise its network availability throughout the topology.
    

**Configuration in CLI:**

```
enable
configure terminal
router ospf 1
network 100.64.5.0 0.0.0.3 area 0
exit
end
write memory
```
![OSPF-ISP](../images/Pasted%20image%2020250914003831.png)

>**Note:** In Packet Tracer it is necessary to enable OSPF even on the ISP router to make the simulation fully functional. In a real network, OSPF would usually not be configured on the provider’s device – the client only uses the default gateway.

### Router R1 - OSPF

- The network **100.64.5.0/30**  is added to ISP.
    
- The network  **10.31.0.0/30** is added to R2.
    

**Configuration in CLI:**

```
enable
configure terminal
router ospf 1
network 100.64.5.0 0.0.0.3 area 0
network 10.31.0.0 0.0.0.3 area 0
exit
end
write memory
```
![OSPF-R1](../images/Pasted%20image%2020250914004430.png)


### Router R2 - OSPF

- The network `10.31.0.0/30` is added to OSPF (connection to R1).
    
- The network `10.31.1.0/30` is added to OSPF (connection to R3).
    

**Configuration in CLI:**

```
enable
configure terminal
router ospf 1
network 10.31.0.0 0.0.0.3 area 0
network 10.31.1.0 0.0.0.3 area 0
exit
write memory
```
![OSPF-R2](../images/Pasted%20image%2020250914005337.png)

### Router R3 - OSPF

- The network `10.31.1.0/30` is added to OSPF on the connection to R2.
    
- This allows R3 to establish adjacency with R2, and the transit network between routers ensures end-to-end connectivity from the ISP through R3.
    

**Configuration in CLI:**

```
enable
configure terminal
router ospf 1
network 10.31.1.0 0.0.0.3 area 0
exit
end
write memory
```
![OSPF-R3](../images/Pasted%20image%2020250914010013.png)


### Conclusion

After configuring OSPF on all routers (ISP, R1, R2, R3), the basic routing backbone is established. The routers dynamically exchange information about connected networks and ensure full connectivity across all points of the topology.

**A successful test using the **ping** command from R3 toward ISP (5/5 without loss) confirms the correct routing functionality.**



## 3.5 – Summary

In this chapter, hostnames are assigned to devices with CLI access, and IP addresses are configured on backbone P2P interfaces. Ports toward switches remain without IP configuration due to trunks and future subinterfaces. OSPF is activated in area 0 with wildcards on the backbone links.

For simulation purposes, OSPF is also enabled on the ISP router. **Functionality is confirmed by a ping from R3 to ISP (5/5 without loss).**

---

**Continue to the next chapter:** [VLANs and Subinterfaces](04-vlans-and-subinterfaces.md)  







































