
# **8 - Troubleshooting**


## 8.1 - Introduction

**Troubleshooting** is an **important part of network management.** Even when the configuration is correct, **problems** may appear in the infrastructure, which can disrupt **communication** or **service availability**. **Diagnostics** makes it possible to quickly identify the **source of an issue**, apply a fix, and then verify the **functionality**. This chapter describes selected **situations** that may occur in the network and require a **systematic solution**.

**Problem areas:**

8.2. Problem with OSPF routing between routers.  
    
8.3. VLAN configuration error.  
    
8.4. Clients in certain VLANs do not receive an address from the DHCP server.  
    
8.5. Inter-VLAN access does not match ACL rules.

>**Note:** The following examples represent model situations that help practice troubleshooting steps and improve skills in working with networks.

## 8.2 - Problem with OSPF routing

**Problem description**

During connectivity testing between routers, it was found that a ping from R3 to the ISP fails. Routers R1 and R2 have their adjacency established correctly, but R3 does not form an OSPF relationship with R2. This indicates a problem in the configuration or interface status.

---

## Diagnostics

**1. On R3 the OSPF neighbor state is verified:**
    

```
**R3#show ip ospf neighbor**
```
![OSPF-diagnostic-R3](../images/Pasted%20image%2020250923182349.png)

The result shows that R3 has no OSPF neighbors.

**2. On R3 the interface status is checked:**
    

```
**R3#show ip interface brief**
```
![OSPF-brief-R3](../images/Pasted%20image%2020250923182749.png)

The g0/1 interface toward R2 has status **up**, but the protocol is **down**. This means the interface is enabled, but the protocol is not running because the port on the other side (R2) is disabled.

**3. On R2 the interface status is verified:**
    

```
**R2#show ip interface brief**
```
![R2-BRIEF-OSPF](../images/Pasted%20image%2020250923182737.png)

The g0/2 interface toward R3 is in the state **administratively down**. This means it was disabled with the **shutdown** command.

---

#### Repairs

**1. Activation of interface g0/2 on R2:**
    

```
**R2(config)# interface g0/2**  
**R2(config-if)# no shutdown**
```

After this fix, the port is active, but the OSPF adjacency still does not form.

**2. Further diagnostics show that OSPF still does not work. On R3 the protocol configuration is checked:**
    

```
**R3#show ip protocols**
```
![OSPF-protocols](../images/Pasted%20image%2020250923183947.png)

The output shows that the network `10.31.1.0/30` was mistakenly assigned to **area 1** instead of **area 0**.

**3. Correction of the OSPF area on R3:**

```
R3(config)#router ospf 1
R3(config-router)#no network 10.31.1.0 0.0.0.3 area 1
R3(config-router)#network 10.31.1.0 0.0.0.3 area 0
```
![OSPF-fix-R3](../images/Pasted%20image%2020250923184121.png)

---

#### Verification

1. On R2 the neighbor state is checked after the fixes:
    

```
**R2#show ip ospf neighbor**
```
![check-R2](../images/Pasted%20image%2020250923184258.png)

The output shows adjacency with R1 and R3 in the FULL state.

**2. On R3 it is verified that OSPF routes appear:**
    

```
R3#show ip route ospf
```
![OSPF-OK](../images/Pasted%20image%2020250923184430.png)

The output confirms that the routes from R2 and further to the ISP are now present.

**3. Verification of the ping toward the ISP:**
    

```
**R3#ping 100.64.5.1**
```
![ping-ISP-OSPF](../images/Pasted%20image%2020250923184535.png)

The routes from R2 and beyond to the ISP are now reachable. The ping from R3 to the ISP is successful.

---

### Conclusion

The problem was caused by two issues: an administratively disabled port on R2 and an incorrectly configured OSPF area on R3. After fixing both errors, full connectivity was restored and OSPF routing works correctly.

## 8.3 - VLAN configuration error

**Problem description**

From PC1 (Admin, VLAN 10) it is not possible to ping any device in VLAN 20. The ping to the default gateway 10.31.20.1 and to the host device 10.31.20.20 fails. The suspicion is an error in inter-VLAN routing (gateway for VLAN 20 on R2).

### Diagnostics

**1. From PC1 it is not possible to ping VLAN20.**
    
```
ping 10.31.20.1
ping 10.31.20.20
```
![VLAN-diagnostic-error](../images/Pasted%20image%2020250923200052.png)


**2. On SW1 it is verified that VLAN 20 exists and ports are assigned correctly:**
   
```
SW1#show vlan brief
```
![VLAN-brief1](../images/Pasted%20image%2020250923200335.png)

VLAN 20 is created, and the ports appear to be configured correctly.

**3. On SW1 the trunk toward R2 (uplink Gi0/1) is verified to confirm that VLAN 20 is allowed:**
    

```
SW1#show interfaces trunk
```
![trunk-test](../images/Pasted%20image%2020250923200452.png)

The trunk is active and VLAN 20 is in the allowed list. No error is visible on the switch.

**4. Diagnostics continue on R2 -> quick interface check:**
    

```
**R2#show ip interface brief**
```
![brief-vlan20](../images/Pasted%20image%2020250923200623.png)

The output shows the subinterface g0/1.20 with IP address 10.31.2.1/24 → this indicates an error, as the gateway for VLAN 20 is incorrect.

---


## Repairs

**1. Fixing the gateway IP address for VLAN 20 on R2:**

```
R2(config)#interface g0/1.20
R2(config-subif)#no ip address
R2(config-subif)#ip address 10.31.20.1 255.255.255.0
R2(config-subif)#end
R2# write memory
```
![fixed-vlan20](../images/Pasted%20image%2020250923200834.png)

---
#### Verification

**1. Checking the subinterface after the fix:**
    

```
**R2#show ip interface brief**
```
![fixed-brief](../images/Pasted%20image%2020250923200915.png)

The output confirms that subinterface g0/1.20 now has the correct IP address 10.31.20.1/24.


**2. Tests from PC1 (Admin):**
    

```
ping 10.31.20.20 
ping 10.31.20.1
```
![ping-vlan20-fixed](../images/Pasted%20image%2020250923201008.png)

The ping to PC2 (10.31.20.20) and to the gateway (10.31.20.1) is successful.

---

### Conclusion

The problem was caused by an incorrectly configured **default gateway** for VLAN 20 on subinterface R2 g0/1.20. The switching part (VLANs/trunks) was correct, and the error appeared only on the router. After correcting the gateway address to 10.31.20.1/24, inter-VLAN connectivity was restored and pings were successful.

## 8.4 - DHCP problem

**Problem description**

From the administrator PC1 (VLAN 10) it is not possible to ping the end devices PC9 (10.31.60.61) and PC10 (10.31.60.60) in VLAN 60 (Warehouse). The ping to the gateway 10.31.60.1 works. The suspicion is on the relay (DHCP helper) for VLAN 60 on R3.

---

### Diagnostics

**1. Test from PC1 (Admin) pings to PC9 / PC10 and the gateway -> Warehouse:**

```
ping 10.31.60.60
ping 10.31.60.61
ping 10.31.60.1
```
![DHCP-test-vlan60](../images/Pasted%20image%2020250923213544.png)

The first two pings fail, but the ping to the gateway 10.31.60.1 is successful.

**2. On SW2 the L2 state is verified – client ports in VLAN 60 and the trunk toward R3 carry VLAN 60:**


```
SW2#show vlan brief
SW2#show interfaces trunk
```
![DHCP-brief-trunk](../images/Pasted%20image%2020250923213752.png)

VLAN 60 exists, the ports are assigned correctly, and the trunk carries VLAN 60 – L2 is fine.

**3. On R3 the status of the subinterface for VLAN 60 (g0/2.60) is checked:**
    

```
R3#show ip interface brief
```
![DHCP-R3-brief](../images/Pasted%20image%2020250923214129.png)

The g0/2.60 interface is up/up with IP 10.31.60.1/24.

**4. The DHCP relay (ip helper-address) on R3 for VLAN 60 is checked:**
    

```
R3# show ip interface g0/2.60 | include Helper
```
![error-ip-helper](../images/Pasted%20image%2020250923214236.png)

The output shows the Helper address as 10.31.10.100 – this is the wrong DHCP server (the correct one should be 10.31.10.10 – SRV01). Therefore, clients in VLAN 60 do not receive an IP from DHCP after renewal.

---


#### Repairs

**1. Replacing the helper with the correct one on R3 (VLAN 60):**


```
R3(config)#interface g0/2.60
R3(config-subif)#no ip helper-address 10.31.10.100
R3(config-subif)#ip helper-address 10.31.10.10
R3(config-subif)#end
R3#write memory
```
![fix-ip-helper](../images/Pasted%20image%2020250923214430.png)

---

#### Verification

- On client PC9 in VLAN 60, enable DHCP again, renew the IP, and verify the assigned IP address with:


```
ipconfig /release
ipconfig /renew
ipconfig
```
![ip-config-PC9](../images/Pasted%20image%2020250923214648.png)

On client PC10 in VLAN 60, enable DHCP again, renew the IP, and verify the assigned IP address with:

```
ipconfig /release
ipconfig /renew
ipconfig
```
![IP-config-PC10](../images/Pasted%20image%2020250923214745.png)


PC9/PC10 receive addresses **10.31.60.x/24**, the **default gateway 10.31.60.1**, and DNS according to the pool.

- Connectivity test from PC1 (Admin):

```
ping 10.31.60.60
ping 10.31.60.61
ping 10.31.60.1
```
![final-ping-VLAN60](../images/Pasted%20image%2020250923215326.png)

All pings are **successful**.

---

### Conclusion

The issue was caused by an incorrectly set ip helper-address (10.31.10.100) on subinterface R3 g0/2.60. The L2 part (VLANs/trunks) was correct; however, clients in VLAN 60 were not receiving IPs from DHCP after renewal and were therefore unreachable. After correcting it to 10.31.10.10 (SRV01), DHCP addresses are assigned and connectivity works.

## 8.5 - Inter-VLAN access does not match ACL rules

**Problem description**

After deploying ACLs, two violations of the intended security policy appear:

- **Finance (VLAN 30)** have unwanted access to PC1 (Admin) in VLAN 10 (PC1 10.31.10.20).
    
- **Guest (VLAN 70)** has access to internal networks (servers in VLAN 10 and hosts in other VLANs), even though Guest should only have internet access.

---


#### Diagnostics

**Verification of unwanted access from Finance and Guest:**

- Ping from PC4 (Finance) to PC1 (Admin)
    

```
**ping 10.31.10.20**
```
![ACL-error-1](../images/Pasted%20image%2020250924134235.png)


**Verification of unwanted access from Guest:**

- Ping from PC11 to SRV02 and to PC1 (Admin)
    

```
**ping 10.31.10.11**  
**ping 10.31.10.20**
```
![ACL-error-2](../images/Pasted%20image%2020250924134523.png)

**The pings are successful, showing that Guest has unwanted access to internal resources!!!***


**- Verification that ACLs are applied on the correct interfaces and directions:**
    


```
**R2#show ip interface g0/1.30**  
```
![show-int-ACL-1](../images/Pasted%20image%2020250924134742.png)


```
**R3#show ip interface g0/2.70**
```
![show-int-ACL-2](../images/Pasted%20image%2020250924135038.png)

The output shows "Inbound access list is V30/V70".

- **Displaying the ACL content and hitcount:**
    

```
**R2#show access-lists V30**
```
![ACL-access-lists-V30](../images/Pasted%20image%2020250924135204.png)

```
R3#show access-lists V70
```
![ACL-show-access-lists-V70](../images/Pasted%20image%2020250924135349.png)


#### Identified incorrect lines

- In V30: **permit ip 10.31.30.0 0.0.0.255 10.31.10.0 0.0.0.255** (broad permission from Finance to Admin)
    
    * Why this is an error:_ it bypasses the policy "Finance must not have general access to the Admin VLAN". Only limited traffic (specific services/servers) should be allowed according to the rest of the ACL.
    
- In V70: **permit ip 10.31.70.0 0.0.0.255 any** is placed above **deny ip 10.31.70.0 0.0.0.255 10.31.0.0 0.0.255.255**, which is an incorrect order.
    
    * Why this is an error:_ the first matching ACE (line) is applied -> the global permit allows everything, so the denies to internal networks are never applied. Our ACL policy states that "Guest must not access internal networks, only the internet".


---

#### Repairs

>Note: Selected ACL groups can be safely rewritten/adjusted. For consistency, the ACL is temporarily removed from the interface, the entire content is rewritten, and then re-applied.

**A) V30 -  Finance:** remove the broad permit from Finance → Admin, keep the rest unchanged.

```
conf t
interface g0/1.30
no ip access-group V30 in
exit

no ip access-list extended V30
ip access-list extended V30
permit udp any eq 68 any eq 67
permit icmp 10.31.30.0 0.0.0.255 any echo-reply
permit tcp 10.31.30.0 0.0.0.255 host 10.31.10.20 established
deny icmp 10.31.30.0 0.0.0.255 host 10.31.10.20 echo
deny ip 10.31.30.0 0.0.0.255 host 10.31.10.20
permit ip 10.31.30.0 0.0.0.255 host 10.31.20.10
permit ip 10.31.30.0 0.0.0.255 host 10.31.10.10
permit ip 10.31.30.0 0.0.0.255 host 10.31.10.11
deny ip 10.31.30.0 0.0.0.255 10.31.20.0 0.0.0.255
deny ip 10.31.30.0 0.0.0.255 10.31.40.0 0.0.0.255
deny ip 10.31.30.0 0.0.0.255 10.31.50.0 0.0.0.255
deny ip 10.31.30.0 0.0.0.255 10.31.10.0 0.0.0.255
permit ip 10.31.30.0 0.0.0.255 any
exit

interface g0/1.30
ip access-group V30 in
exit
write memory
```
![fix-ACL-V30](../images/Pasted%20image%2020250924140934.png)



**B) V70 – Guest:** adjust the order by placing the DENY to internal networks first, followed by the PERMIT ANY.

```
interface g0/2.70
no ip access-group V70 in
exit

no ip access-list extended V70
ip access-list extended V70
permit udp any eq 68 any eq 67
permit icmp 10.31.70.0 0.0.0.255 any echo-reply
deny ip 10.31.70.0 0.0.0.255 10.31.0.0 0.0.255.255   
permit ip 10.31.70.0 0.0.0.255 any                      
exit

interface g0/2.70
ip access-group V70 in
end
write memory
```
![fix-ACL-V70](../images/Pasted%20image%2020250924141713.png)

---

#### Verification

- Ping from PC4 (Finance) to PC1 (Admin) -> general access must be blocked, allowed exceptions must work:
    

```
ping 10.31.10.20
```
![Final-TEST-1](../images/Pasted%20image%2020250926190148.png)

The ping fails, confirming that general access is blocked.


- Ping from PC11 (Guest) to internal networks -> verification with SRV02 and PC2 (Sales) **must be blocked**:
    

```
**ping 10.31.10.11**  
**ping 10.31.20.20**
```
![Final-test-2](../images/Pasted%20image%2020250924142334.png)

The pings fail, confirming that Guest is blocked from accessing internal networks.


--- 

### Conclusion

**The ACL errors violated the security policy:**

- In V30 a broad **permit** allowed Finance general access to the Admin VLAN – against the rule "Finance must not have access to Admin/PC1 except for exceptions".
    
- In V70 the wrong order caused **permit any** to allow everything before the deny to internal networks could apply. Guest therefore gained unintended access.
    

After rewriting the ACL (removing the broad permit in V30, correcting the deny/permit order in V70), inter-VLAN access is in line with the chosen security policy: Finance only has defined exceptions and Guest has no access to internal networks.

## 8.6 - Summary

In this chapter the work began with OSPF: first checking routing between backbone routers and, during neighbor diagnostics, identifying a problem caused by a misassigned area so the correct routes would appear and routers would recognize each other. Then the focus moved to VLANs – detecting an incorrectly configured gateway on the router for one VLAN, and after correction inter-VLAN communication worked.

In the second part of the chapter DHCP was addressed, where the relay was pointing to the wrong address and clients in one VLAN did not receive configuration. After fixing the helper, addresses began to be assigned to the corrected VLAN as expected. Finally, ACLs were checked – removing a broad Finance permit and fixing the rule order for Guest so that it aligned with the chosen security policy. After these corrections, the network functions as intended.

**Continue to the next chapter:** Conclusion and Summary











