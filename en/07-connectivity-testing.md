
# 7 - Connectivity Testing


## 7.1 - Introduction

This chapter tests **connectivity** and the correct operation of the network. It is checked step by step that all devices have access as designed, services are running, and security rules are functioning. Thanks to these tests, it is clearly visible that the network is **operational**.

At the beginning, there is a clear **test table** for better orientation, followed by individual examples of outputs.

## 7.2 - Test/Diagnostics Overview

|Test|Function|Test Description|Command / Action|Diagnostic Outputs|
|---|---|---|---|---|
|7.2|Backbone link between routers|Verification of connection between routers and ISP|`ping` to IP interface|`ping`, **show ip interface brief**|
|7.3|Routing (OSPF)|Check neighbors and routes|**show ip ospf neighbor**, **show ip route ospf**|neighbor and route output|
|7.4|VLANs|Test `ping` to gateway IP from PC + configuration check|`ping` gateway IP, **show vlan brief**, **show interfaces trunk**, **show running-config**|ping results, VLAN and trunk tables|
|7.5|DHCP|Verification that PCs obtain address automatically|GUI → Desktop → IP Config, **show running-config** (router → **ip helper-address**)|assigned DHCP addresses, helper address configuration|
|7.6|DNS + HTTP|Name translation and web access|`ping www.firma.local`, Web Browser → `www.firma.local`|response from 10.31.10.11, page loaded|
|7.7|NTP|Verification of time synchronization|**show clock**, **show ntp status**|current time and sync status|
|7.8|SSH + VLAN99|Login to device using SSH|`ssh -l distadmin <IP>`, **show ip ssh**, **show ssh**, `ping` on VLAN99|service status output, active sessions, login confirmation|
|7.9|ACL rules|Test of allowed and denied access between VLANs|`ping` between devices|allowed/denied results|



## 7.2 - Backbone Links Between Routers

In this section, the basic connections between routers and towards the ISP are tested. The goal is to verify that each interface on the backbone links operates correctly and that the routers have direct connectivity between each other. This step is necessary for proper routing operation and related services.

### Testing Procedure

1. Verify the IP address status on all router interfaces using the command `show ip interface brief`.
    
2. Perform `ping` between neighboring routers (ISP <-> R1, R1 <-> R2, R2 <-> R3).
    
3. A successful `ping` confirms that the connection works and that the addresses are configured correctly.

---

#### Used CLI Commands:

**ISP router:**

```
ping 100.64.5.2
show ip interface brief
```
![7.2-test-1](../images/Pasted%20image%2020250922205221.png)

**Router R1:**

```
ping 100.64.5.1
ping 10.31.0.2
show ip interface brief
```
![7.2-test-2](../images/Pasted%20image%2020250922205501.png)

**Router R2:**

```
ping 10.31.0.1
ping 10.31.1.2
show ip interface brief
```
![7.2-test-3](../images/Pasted%20image%2020250922205752.png)

**Router R3:**

```
ping 10.31.1.1
show ip interface brief
```
![7.2-test-4](../images/Pasted%20image%2020250922205912.png)

### Conclusion

The backbone links between ISP, R1, R2, and R3 work flawlessly. All interfaces are in the **up/up** state, and pings between neighboring routers confirm full connectivity. This step forms the foundation for subsequent routing with OSPF and other functions.

### 7.3 - Routing (OSPF)

In this section, the functionality of dynamic routing using the **OSPF** protocol is tested. The goal is to verify that routers establish neighbor relationships, share routing information, and that the routing tables contain all necessary routes. Proper OSPF operation is essential for connecting the entire network.

#### Testing Procedure

1. Verify the establishment of neighbor relationships between routers using the command `show ip ospf neighbor`.
    
2. Check the content of routing tables using `show ip route ospf`.
    
3. Perform several `ping` tests across multiple hops to confirm connectivity across the topology.
    
---

#### Used CLI Commands:

**Router R1:**

```
show ip ospf neighbor
show ip route ospf
ping 100.64.5.1
```
![OSPF-test-1](../images/Pasted%20image%2020250922210613.png)

**Router R2:**

```
show ip ospf neighbor
show ip route ospf
ping 10.31.1.2
```
![OSPF-TEST2](../images/Pasted%20image%2020250926185615.png)

**Router R3:**

```
show ip ospf neighbor
show ip route ospf
ping 100.64.5.1
```
![OSPF-test-3](../images/Pasted%20image%2020250922210857.png)

>**Note:** The value shown in the **Neighbor ID** column corresponds to the neighbor’s **Router ID**, not the address of the backbone interface. If the Router ID is not set manually, OSPF selects the highest IP address from active interfaces (for example, from the management VLAN). Therefore, the neighbor overview may display VLAN 99 addresses instead of backbone link addresses.

### Conclusion

OSPF routing works correctly. Routers establish neighbor relationships, exchange routing information, and the routing tables contain all necessary entries. Successful pings between R3 and the ISP confirm that communication works even across multiple hops and that routing is fully functional.

## 7.4 - VLANs

In this section, the functionality of individual **VLANs** and their interconnection through trunk ports is tested. The goal is to verify that end devices in VLANs can reach their gateways, that the configuration on switches is correct, and that trunks carry the necessary traffic.

### Testing Procedure

1. Verify connectivity between selected VLANs using `ping`:
    
    - `ping` from PC7 in VLAN 50 (Management) to the gateway in VLAN 20 (Sales),
        
    - `ping` from PC5 in VLAN 40 (Logistics) to the gateway in VLAN 60 (Warehouse),
        
    - `ping` from Admin PC1 in VLAN 10 to the gateway in Guest VLAN 70.
        
2. Display VLAN configuration on switches using `show vlan brief`.
    
3. Verify trunk port status with `show interfaces trunk`.
    
4. Complement the configuration check with `show running-config`, but omit the CLI output due to its length.
    

---
### Used CLI Commands:

**1. Ping Between VLANs:**

```
ping 10.31.20.1  
```
![PC-7-PING](../images/Pasted%20image%2020250922232304.png)

```
ping 10.31.60.1 
```
![PC5-PING](../images/Pasted%20image%2020250922232343.png)

```
ping 10.31.70.1  
```
![PC1-PING](../images/Pasted%20image%2020250922232422.png)

**2. Switch (example SW1):**

```
show vlan brief
show interfaces trunk
```
![diagnostic-SW1](../images/Pasted%20image%2020250922232819.png)

**3. Router R2 (subinterface VLAN 10-50):**

```
show running-config
```
![subinterface-R2](../images/Pasted%20image%2020250922233334.png)


### Conclusion

Pings to gateways in all VLANs and between selected VLANs were successful. Switch outputs confirm that VLANs are created correctly and assigned to the appropriate ports. Trunk ports between switches and the router carry tagged traffic without errors. The configuration of subinterfaces on router R2 matches the assigned VLANs.

## 7.5 - DHCP

In this section, the functionality of the **DHCP** service across the network is tested. The goal is to verify that clients in all VLANs automatically obtain addresses from the server and that the assigned values match the configured pools.

### Testing Procedure

1. On PCs in each VLAN, configure address assignment via DHCP (Desktop → IP Configuration → DHCP).
    
2. On routers, check the configuration of `ip helper-address` using the command `show running-config`. For demonstration, only R2 is shown.
    
3. Perform connectivity tests:
    
    - `ping` from PC1 (Admin) to PC3 (Finance),
        
    - `ping` from PC8 (Management) to PC2 (Sales),
        
    - `ping` from PC10 (Warehouse) to PC4 (Logistics).

---

### Used CLI Commands:

**1. **PC (example PC2 / PC5 / PC9):**

```
Desktop → IP Configuration → DHCP
```
![DHCP-PC2](../images/Pasted%20image%2020250922234716.png)

![DHCP-PC5](../images/Pasted%20image%2020250922234740.png)

![DHCP-PC9](../images/Pasted%20image%2020250922234805.png)

**2. Router (R2):**

```
show running-config
```
![IP-helper](../images/Pasted%20image%2020250923001034.png)

**3. Connectivity Test:**

```
ping 10.31.30.30
```
![DHCP-PC1](../images/Pasted%20image%2020250923122309.png)

```
ping 10.31.20.20
```
![DHCP-PC8](../images/Pasted%20image%2020250923122417.png)

```
ping 10.31.40.40
```
![DHCP-PC10](../images/Pasted%20image%2020250923122944.png)


### Conclusion

Clients in all VLANs received correct addresses from the server 10.31.10.10. The configuration of `ip helper-address` on routers is functional. Pings between PC1 and PC3, PC10 and PC4, as well as PC8 and PC2 verified connectivity after address assignment.

>**Note:** Active ACLs already block some pings between selected VLANs at this stage, according to the configured security policies.

## 7.6 - DNS and HTTP

#### Introduction

In this section, the functionality of **DNS** and **HTTP** services in the network is verified. DNS translates domain names to IP addresses, and HTTP provides web content to clients. The diagnostics show whether clients can access the web server using a domain name.

#### Verification Procedure

1. **DNS Translation Verification**
    
    - On PC3 and PC6, run the following command in the command line:  
        `ping www.firma.local`
        
    - Verify that the domain name is correctly translated to the server IP address **10.31.10.10** and that replies are received.
        
2. **HTTP Access Verification**
    
    - On PC5 and PC9, enter the following address in the web browser:  
        `http://www.firma.local`
        
    - Confirm that the default web page stored on the server (e.g., **index.html**) is displayed.
        
3. **ACL Check**
    
    - ACL rules allow clients from internal VLANs to use DNS and HTTP services from the server.
        
    - The Guest VLAN is separated from these services and has no access.

---


#### 1. DNS Translation Verification

![DNS-PC3](../images/Pasted%20image%2020250923123627.png)

![DNS-PC6](../images/Pasted%20image%2020250923123709.png)

#### 2. HTTP Access Verification

![HTTP-PC5](../images/Pasted%20image%2020250923123825.png)

![HTTP-PC9](../images/Pasted%20image%2020250923123906.png)


#### 3. ACL Check

- Verify the block from PC11 in VLAN70 when accessing the company local web.

![HTTTP-ACL-PC11](../images/Pasted%20image%2020250923124111.png)


### Conclusion

The verification confirmed the correct operation of DNS translation and HTTP services. Selected client stations were able to resolve the domain name to an IP address and load web content through the browser. The ACL policy was followed → internal VLANs have access, while the Guest VLAN remains isolated.

## 7.7 - NTP

Here the functionality of the **NTP** (Network Time Protocol) service in the network is verified. NTP ensures time synchronization across all network devices, which is essential for proper protocol operation, logging, and security mechanisms.

#### Verification Procedure

1. **Check on R2**
    
    - On router R2, run the command:  
        `show clock`
        
    - For detailed verification, use:  
        `show ntp status`
        
2. **Check on R3**
    
    - On router R3, run the command:  
        `show clock`
        
    - For NTP diagnostics, also use:  
        `show ntp associations`

---

#### 1. Check on R2

```
show clock
show ntp status
```
![NTP-R2-TEST](../images/Pasted%20image%2020250923124738.png)

#### 2. Check on R3

```
show clock
show ntp associations
```
![NTP-R3-TEST](../images/Pasted%20image%2020250923125119.png)

 ### **Conclusion**

The verification confirmed the correct operation of the NTP service. Routers R2 and R3 have synchronized time with the NTP server R1, ensuring consistent log entries and proper functioning of time-dependent processes.

## 7.8 - SSH and VLAN 99

In this section, the functionality of secure remote access using **SSH** and the correct configuration and availability of the management VLAN 99 are verified. SSH provides encrypted management of network devices, while VLAN 99 serves as a dedicated management VLAN.

#### Verification Procedure

1. **SSH Connection Verification**
    
    - On router R1, establish an SSH connection to router R2:  
        `ssh -l distadmin 10.31.x.x`
        
    - Verify successful user login and access to the CLI on R2.
        
2. **VLAN 99 Verification**
    
    - On router R3, perform a `ping` to the VLAN 99 address on switch SW2:  
        `ping 10.31.99.x`
        
    - Confirm that VLAN 99 is reachable and properly configured for management.

---

#### 1. SSH Connection Verification

```
ssh -l distadmin 10.31.0.2
```
![SSH-TEST](../images/Pasted%20image%2020250923130252.png)


#### 2. VLAN 99 Verification


* From router R3, a `ping` is executed to the VLAN 99 address on switch SW2.

![VLAN99-TEST](../images/Pasted%20image%2020250923130444.png)

### Conclusion

The diagnostics confirmed functional SSH access between routers R1 and R2, allowing secure remote management. The VLAN 99 test showed that the management VLAN is reachable from the network, specifically from router R3 to switch SW2. This confirms the correct operation of the management infrastructure.

## 7.9 - ACL Rules

The functionality of **Access Control Lists (ACLs)** in the network is verified. Communication is tested between gateways and between end devices in individual VLANs, while also demonstrating the application of specific rules, such as restricted access for the Guest VLAN, allowed access to the printer, or privileged access for the Admin and Management VLANs.

#### Verification Procedure

**1. Ping Between Gateways**

- Verify the mutual availability of subinterfaces between routers R2 and R3 (e.g., 10.31.20.1 ↔ 10.31.60.1).
    
- Pings between gateways confirm correct routing and VLAN interconnection.
    

**2. Ping Between End Devices**

- From PC2 in VLAN 20 (10.31.20.21, DHCP), perform a `ping` to PC4 in VLAN 30 (10.31.30.30).
    
- Verify that access is blocked according to ACL rules.
    
- From PC6 in VLAN 40 (10.31.40.41), perform a `ping` to PC10 in VLAN 60 (10.31.60.61).
    
- Verify that access is allowed because ACL permits communication between Logistics and Warehouse.
    

**3. VLAN Guest Demonstration**

- From PC11 in VLAN 70 (10.31.70.70, DHCP), perform a `ping` to internal servers (10.31.10.10, 10.31.10.11).
    
- Verify that pings fail due to ACL policy.
    
- Then, from PC11, perform a `ping` to the ISP router IP (e.g., 100.64.5.1).
    
- Verify that the ping is successful, demonstrating VLAN Guest access only to the ISP network, not to internal VLANs.
    

**4. Printer Access Verification**

- From PC6 in VLAN 40 (10.31.40.40) and PC7 in VLAN 50 (10.31.50.50), perform a `ping` to the printer (10.31.20.10).
    
- Verify that communication is allowed according to ACL.
    

**5. Privileged Access for Admin and Management VLANs**

- From Admin PC in VLAN 10, perform a `ping` to PC4 in VLAN 30 (Finance, 10.31.30.30).
    
- Verify that access is allowed.
    
- From PC8 in VLAN 50 (Management, 10.31.50.51), perform a `ping` to Finance (10.31.30.30).
    
- Verify that access is also allowed.
    

**6. Server Communication Verification**

- From PC2 in VLAN 20 (10.31.20.21), perform a `ping` to the server 10.31.10.10.
    
- Verify that access is allowed according to ACL policy.

---


### Executed Commands:

#### 1. Ping Between Gateways

- `ping` from R2 to the gateways of VLAN20 and VLAN60

```
ping 10.31.20.1
ping 10.31.60.1
```
![ACL-testing1](../images/Pasted%20image%2020250923152643.png)

- `ping` from R3 to the gateways of VLAN20 and VLAN60

```
ping 10.31.20.1
ping 10.31.60.1
```
![ACL-testing2](../images/Pasted%20image%2020250923152925.png)



#### 2. Ping Between End Devices

- `ping` from PC2 (Sales) to PC4 (Finance)

```
ping 10.31.30.30
```
![ACL-ping1](../images/Pasted%20image%2020250923153311.png)

* `ping` from PC5 (Logistics) to PC10 (Warehouse)

```
ping 10.31.60.61
```
![ACL-ping2](../images/Pasted%20image%2020250923154236.png)


#### 3. VLAN Guest Demonstration

- `ping` from PC11 to servers SRV01 and SRV02

```
ping 10.31.10.10
ping 10.31.10.11
```
![ACL-ping3](../images/Pasted%20image%2020250923154607.png)


`ping` from PC11 to the ISP (external network)

```
ping 100.64.5.1
```
![ACL-ping4](../images/Pasted%20image%2020250923155157.png)


#### 4. Printer Access Verification

- `ping` from PC6 (Logistics) to the printer in VLAN20

```
ping 10.31.20.10
```
![ACL-ping5](../images/Pasted%20image%2020250923155728.png)

* `ping` from PC7 (Management) to the printer in VLAN20

```
ping 10.31.20.10
```
![ACL-ping6](../images/Pasted%20image%2020250923155915.png)

#### 5. Privileged Access for Admin and Management VLANs

- From Admin PC1 (Admin-Server), perform a `ping` to PC4 (Finance)


```
ping 10.31.30.30
```
![ACL-ping7](../images/Pasted%20image%2020250923160341.png)


From PC8 in VLAN 50 (Management), perform a `ping` to PC4 (Finance)

```
ping 10.31.30.30
```
![ACL-ping8](../images/Pasted%20image%2020250923160512.png)


#### 6. Server Communication Verification

- From PC2 (Sales), perform a `ping` to server SRV01

```
ping 10.31.10.10
```
![ACL-ping9](../images/Pasted%20image%2020250923160757.png)



### Conclusion

The diagnostics confirmed the correct application of ACL rules. It was verified that the Guest VLAN has access only to the ISP and not to internal resources. Printer access works for defined VLANs, while Admin and Management VLANs have privileged access to Finance as well. Servers are accessible from internal VLANs according to ACL policy, confirming the correct implementation of security rules.

## Summary

Chapter 7 verifies the functionality of the entire network, from basic interconnections to security rules. First, backbone links between routers and towards the ISP were tested, where pings and OSPF confirmed correct routing. VLAN diagnostics demonstrated functional gateways and trunks, while DHCP successfully assigned addresses from pools. DNS and HTTP provided name resolution and web content, while ACL correctly blocked the Guest VLAN. NTP verification confirmed time synchronization on the routers.

Next, SSH access between routers and the availability of management VLAN 99 were tested. The final ACL diagnostics confirmed that inter-VLAN communication followed the defined policy: Guest had access only to the ISP, Admin and Management VLANs had privileges to Finance, and access to servers and the printer was allowed by the rules. This chapter therefore confirms the stable and secure operation of the entire topology.


---

**Continue to the next chapter:** [Troubleshooting](08-troubleshooting.md)  














