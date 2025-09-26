
# **6 - Network Security**


## 6.1 - Introduction

**Security is a key part of every network.** Even a well-designed topology can be vulnerable if it is not protected against unauthorized access. This chapter introduces measures that make the network more reliable and better protected against attacks.

A **single user account with a strong password** is implemented, along with the protection of privileged mode using **enable secret** and encrypted remote access through **SSH**. Switch management is separated in **management VLAN 99**, and upon login, **warning banners (MOTD)** are displayed to notify users about restricted access.

Unlike in Project 1, **Port Security and disabling of unused ports are not applied**, since access to switching devices is limited to internal staff and the demonstration of these features is not the goal of this project. Communication between VLANs is regulated by **Access Control Lists (ACL)**. Additional functions are covered with **CDP (Cisco Discovery Protocol)**.

## 6.2 - User Account and Password

A basic security measure is the creation of a **dedicated user account with a strong password** and the protection of privileged mode using the **enable secret** command. This replaces the simpler password‑only login and ensures that access to network devices is restricted to authorized personnel. The **enable secret** is automatically encrypted and more secure than the classic enable password.

**We configure all mentioned devices:** R1, R2, R3, SW1, SW2.

- Create a user account with a password
    
- Set the enable secret for privileged mode
    
- Example CLI configuration is shown only for R1 (the same procedure applies to other devices)
    

**Configuration in CLI:**

```plaintext
enable
configure terminal
username distadmin secret Logistic25
enable secret Pallet25
end
write memory
```
![login-password](../images/Pasted%20image%2020250916184716.png)

### Conclusion:  

The command `username distadmin secret` creates a user account that is used for login through the **console** and **VTY lines (SSH)**. The `enable secret` command protects access to privileged mode. Both commands complement each other and together form the basic layer of access security for network devices.

>Note: For project purposes, simpler thematic passwords are chosen. In a real deployment, a more complex and longer password (12+ characters, a mix of uppercase/lowercase letters, numbers, and special symbols) would be required.

## 6.3 - Console Security

**Securing console access** is important to prevent unauthorized persons with physical access from reaching the console port. The console is therefore configured to require a **username and password** from the local database during login. The account created in the previous subsection is used for this purpose. In addition, an inactivity timeout can be set, after which the session will automatically close.

Devices: R1, R2, R3, SW1, SW2.

- Configure console access to require login with the local account
    
- Use the account **distadmin / Logistic25** created earlier
    
- Optionally set an inactivity timeout (exec-timeout)
    
- Example CLI configuration is shown only for R1 (the same procedure applies to other devices)

**Configuration in CLI:**

```
enable  
configure terminal  
line console 0  
login local  
exec-timeout 5  
end  
write memory
```
![security-console](../images/Pasted%20image%2020250916201828.png)


### Conclusion:  

After activating the `login local` command, console access requires entering a username and password from the local database. This ensures that only authorized personnel can log in. The inactivity timeout set with `exec-timeout` further increases security by ending forgotten or idle sessions.

## 6.4 - SSH Access (VTY Lines)

**Secured remote access** to network devices is configured using the **SSH protocol**, which replaces the insecure Telnet. SSH encrypts communication and protects login credentials as well as device management from eavesdropping. Access on VTY lines is therefore restricted to SSH only, using the user account created in the previous steps.

The `ip domain-name` command is used for generating RSA keys and for creating the full domain name of the device (FQDN). The same domain name is applied on all devices, for example **firma.local**. Each device then has its unique name in the form **hostname.firma.local**.

**Devices:** R1, R2, R3, SW1, SW2.

- Configure DNS domain for generating RSA keys
    
- Generate RSA keys for SSH encryption
    
- Enable SSH version 2
    
- Configure VTY lines to require login with the local user database
    
- Restrict access to SSH only (no Telnet)
    
- Set inactivity timeout (exec-timeout)
    
- Example CLI configuration is shown only for R1 (the same procedure applies to other devices)

**Configuration in CLI:**

```
enable  
configure terminal  
ip domain-name firma.local  
crypto key generate rsa 
1024  
ip ssh version 2  
line vty 0 4  
login local  
transport input ssh  
exec-timeout 10  
end  
write memory
```
![SSH-configuration](../images/Pasted%20image%2020250916211121.png)

#### Verification of SSH Access

- On R2, run the command `ssh -l distadmin 10.31.1.2`
    
- When prompted, enter the password **Logistic25**
    
- After login, the device is in user mode `>`
    
- To enter privileged mode, type `enable`
    
- When prompted, enter the password **Pallet25**
    
- The device switches to privileged mode `#`
    
- The user on R2 is logged into R3, and all commands are executed on that remote device


![SSH-login](../images/Pasted%20image%2020250916212006.png)


### Diagnostics

- `show ip ssh` – displays the SSH status on the device
    
- `show ssh` – displays active SSH sessions



![SSH-diagnostic](../images/Pasted%20image%2020250916212345.png)

### Conclusion:  

After configuration, remote access is possible only via SSH. The device requires login with the user account (**distadmin / Logistic25**) and then the password for privileged mode (**Pallet25**). Thanks to SSH encryption, login credentials are protected and the session remains secure during remote access.



## 6.5 - MOTD Banner

The **Message of the Day (MOTD) banner** is used to display a security warning message to users when they log in to the device. The banner does **not block access**, but informs users that the device is intended only for authorized personnel. In practice, this is a standard part of corporate security policies.

**Devices:** R1, R2, R3, SW1, SW2.

### Procedure

- Enter global configuration mode
    
- Set the MOTD banner with a warning text
    
- Close the text with a chosen delimiter (e.g., #)
    
- Example CLI configuration is shown only for R1 (the same procedure applies to other devices)
    

**Configuration in CLI:**

```
enable
configure terminal
banner motd # Unauthorized access to Logistic Network is prohibited! #
end
write memory
```
![MOTD-BANNER-R1](../images/Pasted%20image%2020250916213457.png)


### Verification

- After logging out (`exit`) and logging back into the device, the warning banner is displayed before the password prompt.
    

**Example output:**

```
Unauthorized access to Logistic Network is prohibited!
R1>
```
![MOTD-message](../images/Pasted%20image%2020250916213636.png)

### Conclusion:  

The MOTD banner increases security by **warning against unauthorized access** and clearly stating that the network resources belong to the organization. Within project documentation, the banner is considered a standard part of corporate security.

## 6.6 - Management VLAN 99

### Introduction

A **Management VLAN** is used to separate network device management from regular user traffic. This ensures that access to devices (such as SSH) takes place in a dedicated network, which improves both security and clarity. In this project, **VLAN 99** is reserved for management purposes.

Without creating and configuring a Management VLAN on switches, it would not be possible to manage the devices remotely. The switch could not be pinged or accessed via SSH. Therefore, routers and switches must be configured (for example, VLAN 99) with a dedicated IP address for the administrator.

**Devices:** R1, R2, R3, SW1, SW2.

#### Procedure

- Create VLAN 99 on switches
    
- Assign it an IP address on the SVI (Switch Virtual Interface) for management
    
- Configure trunk ports to carry VLAN 99
    
- Verify functionality using ping and SSH
    

**Configuration in CLI (SW1):**

```
enable
configure terminal
vlan 99
name service-SW1
exit

interface vlan 99
ip address 10.31.99.1 255.255.255.0
no shutdown

interface GigabitEthernet0/2
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50,99
end
write memory
```
![VLAN99-SW1](../images/Pasted%20image%2020250916225221.png)

**Configuration in CLI (SW2):**

```
enable
configure terminal
vlan 99
name service-SW2
exit

interface vlan 99
ip address 10.31.99.2 255.255.255.0
no shutdown

interface GigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 60,70,99
end
write memory
```
![VLAN99-SW1](../images/Pasted%20image%2020250916225455.png)


**Configuration in CLI (R2):**

```
enable
configure terminal
interface GigabitEthernet0/1.99
encapsulation dot1Q 99
ip address 10.31.99.3 255.255.255.0
no shutdown
end
write memory
```
![VLAN99-R2](../images/Pasted%20image%2020250916225649.png)

**Configuration in CLI (R3):**

```
enable
configure terminal
interface GigabitEthernet0/2.99
encapsulation dot1Q 99
ip address 10.31.99.4 255.255.255.0
no shutdown
end
write memory
```
![VLAN99-R3](../images/Pasted%20image%2020250916225904.png)

### Verification

- From R3, run `ping 10.31.99.2` to verify connectivity to SW1
    
- From R3, run `ssh -l distadmin 10.31.99.2` and log in to SW1 using the account **distadmin / Logistic25**
    
- On SW2, run the command `show vlan brief` to check the existence and status of VLAN 99

![VLAN99-test](../images/Pasted%20image%2020250916230150.png)

### Conclusion

Switch management is carried out separately in the dedicated **VLAN 99**, which increases the security and clarity of the infrastructure. VLAN 99 is carried through trunks between devices and enables remote encrypted SSH access within the SERVICE-ADMIN network.

## 6.7 - Access Control Lists (ACL)

**Access Control Lists (ACL)** are used to control **traffic between VLANs** and to protect sensitive parts of the network. They allow defining who can access servers, who can access other departments, and who has access only to the internet.

The goal is to establish a **security policy** that matches the company’s requirements and demonstrates the practical use of ACL within an enterprise network.

#### ACL Objectives

- **Servers** must be accessible to all internal VLANs except Guest.
    
- **Admin PC and Management VLAN** have full access to all VLANs.
    
- **Finance VLAN** is strictly protected – access only to servers and the printer.
    
- **Sales, Logistics, and Warehouse** have limited but functional access (servers, printer, mutual communication).
    
- **Guest VLAN** is isolated from the internal network and directed only to the ISP.


### ACL Configuration Steps – Router R2 (VLAN 20, 30, 40, 50)

1. **Allow DHCP**  
    At the beginning of each ACL, insert the rule:  
    `permit udp any eq 68 any eq 67`
    
2. **Set ACL for each VLAN separately**  
    Create an ACL for VLAN 20, 30, 40, and 50.
    
3. **Allow access to servers and printer**
    
    
    - Servers: `10.31.10.10` and `10.31.10.11`
        
    - Printer: `10.31.20.10`
    
    
4. **Ensure access for Admin PC1**
    
    
    - Admin PC1 (`10.31.10.20`) has access to all VLANs.
        
    - Other VLANs are explicitly denied ping and access to Admin PC1.
    
    
5. **Restrict communication between VLANs**
    
    
    - Finance (VLAN 30) is isolated, allowed only servers and printer.
    
    - Sales (VLAN 20) is allowed servers and printer, denied Admin PC1.
    
    - Logistics (VLAN 40) is allowed servers, printer, and access to Warehouse (VLAN 60).
    
    - Management (VLAN 50) has full access to all VLANs.
    
    
6. **Allow internet access**  
    At the end of each ACL, insert:  
    `permit ip any`

**Configuration in CLI:**

```
enable
configure terminal
ip access-list extended V20
permit udp any eq 68 any eq 67
permit icmp 10.31.20.0 0.0.0.255 any echo-reply
permit tcp 10.31.20.0 0.0.0.255 host 10.31.10.20 established
deny icmp 10.31.20.0 0.0.0.255 host 10.31.10.20 echo
deny ip 10.31.20.0 0.0.0.255 host 10.31.10.20
permit ip 10.31.20.0 0.0.0.255 host 10.31.20.10
permit ip 10.31.20.0 0.0.0.255 host 10.31.10.10
permit ip 10.31.20.0 0.0.0.255 host 10.31.10.11
deny ip 10.31.20.0 0.0.0.255 10.31.30.0 0.0.0.255
deny ip 10.31.20.0 0.0.0.255 10.31.50.0 0.0.0.255
deny ip 10.31.20.0 0.0.0.255 10.31.10.0 0.0.0.255
permit ip 10.31.20.0 0.0.0.255 any
exit
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
ip access-list extended V40
permit udp any eq 68 any eq 67
permit icmp 10.31.40.0 0.0.0.255 any echo-reply
permit tcp 10.31.40.0 0.0.0.255 host 10.31.10.20 established
deny icmp 10.31.40.0 0.0.0.255 host 10.31.10.20 echo
deny ip 10.31.40.0 0.0.0.255 host 10.31.10.20
permit ip 10.31.40.0 0.0.0.255 host 10.31.20.10
permit ip 10.31.40.0 0.0.0.255 host 10.31.10.10
permit ip 10.31.40.0 0.0.0.255 host 10.31.10.11
permit ip 10.31.40.0 0.0.0.255 10.31.60.0 0.0.0.255
deny ip 10.31.40.0 0.0.0.255 10.31.30.0 0.0.0.255
deny ip 10.31.40.0 0.0.0.255 10.31.20.0 0.0.0.255
deny ip 10.31.40.0 0.0.0.255 10.31.50.0 0.0.0.255
deny ip 10.31.40.0 0.0.0.255 10.31.10.0 0.0.0.255
permit ip 10.31.40.0 0.0.0.255 any
exit
ip access-list extended V50
permit udp any eq 68 any eq 67
permit ip 10.31.50.0 0.0.0.255 any
exit
end
write memory
```
![ACL-R2](../images/Pasted%20image%2020250923121848.png)

>**Note:** To ensure DHCP works even when ACLs are applied, include `permit udp any eq 68 any eq 67` at the very beginning of every ACL. When a client starts, it has no IP address and sends its first DHCP request from source `0.0.0.0` to destination `255.255.255.255`. Such a packet would not normally match the allowed networks (for example, Sales 10.31.20.0/24, Warehouse 10.31.30.0/24) and the ACL would block it. This exception allows DHCP requests to reach the router, and `ip helper-address` forwards them to the server `10.31.10.10`, while all other security rules remain enforced.

---


### ACL Configuration Steps – Router R3 (VLAN 60, 70)

1. **Allow DHCP**  
    At the beginning of each ACL, insert the rule (for DHCP):  
    `permit udp any eq 68 any eq 67`
    
2. **Configure ACL separately for each VLAN**  
    Create an ACL for VLAN 60 and 70.
    
3. **Allow access to servers and printer**
    
    
    - Servers: `10.31.10.10` and `10.31.10.11`
        
    - Printer: `10.31.20.10`
    
    
4. **Ensure access for Admin PC1**
    
    
    - Admin PC1 (`10.31.10.20`) has access to all VLANs.
        
    - Other VLANs except Management are explicitly denied ping and access to Admin PC1.
    
    
5. **Restrict communication between VLANs**
    
    
    - Warehouse (VLAN 60) is allowed servers, printer, and access to Logistics (VLAN 40).
        
    - Guest (VLAN 70) is allowed internet only, with no access to the internal network.
    
    
6. **Allow internet access**  
    At the end of each ACL, insert:  
    `permit ip any`

**Configuration in CLI:**

```
enable
configure terminal
ip access-list extended V60
permit udp any eq 68 any eq 67
permit icmp 10.31.60.0 0.0.0.255 any echo-reply
permit tcp 10.31.60.0 0.0.0.255 host 10.31.10.20 established
deny icmp 10.31.60.0 0.0.0.255 host 10.31.10.20 echo
deny ip 10.31.60.0 0.0.0.255 host 10.31.10.20
permit ip 10.31.60.0 0.0.0.255 host 10.31.20.10
permit ip 10.31.60.0 0.0.0.255 host 10.31.10.10
permit ip 10.31.60.0 0.0.0.255 host 10.31.10.11
permit ip 10.31.60.0 0.0.0.255 10.31.40.0 0.0.0.255
deny ip 10.31.60.0 0.0.0.255 10.31.30.0 0.0.0.255
deny ip 10.31.60.0 0.0.0.255 10.31.50.0 0.0.0.255
deny ip 10.31.60.0 0.0.0.255 10.31.20.0 0.0.0.255
deny ip 10.31.60.0 0.0.0.255 10.31.10.0 0.0.0.255
permit ip 10.31.60.0 0.0.0.255 any
exit
ip access-list extended V70
permit udp any eq 68 any eq 67
permit icmp 10.31.70.0 0.0.0.255 any echo-reply
deny ip 10.31.70.0 0.0.0.255 10.31.0.0 0.0.255.255
permit ip 10.31.70.0 0.0.0.255 any
exit
end
write memory
```
![ACL-R3](../images/Pasted%20image%2020250923121910.png)


---

### Applying ACLs to Interfaces

For ACL rules to function properly, they must be assigned to the corresponding router subinterfaces. In our design, the ACLs are applied **inbound** (on incoming traffic from VLANs into the router). This ensures that traffic is inspected and, if necessary, dropped immediately upon entering the router.

#### Router R2

- Apply ACLs V20–V50 on subinterfaces Gi0/1.20, Gi0/1.30, Gi0/1.40, and Gi0/1.50.
    
- Each ACL is assigned inbound for traffic arriving from the respective VLAN.
    

**Configuration in CLI:**

```
enable
configure terminal
interface GigabitEthernet0/1.20
ip access-group V20 in
interface GigabitEthernet0/1.30
ip access-group V30 in
interface GigabitEthernet0/1.40
ip access-group V40 in
interface GigabitEthernet0/1.50
ip access-group V50 in
exit
end
write memory
```
![Subinterface-R2](../images/Pasted%20image%2020250922230649.png)

---

#### Router R3

- Apply ACLs V60 and V70 on subinterfaces Gi0/2.60 and Gi0/2.70.
    
- Each ACL is assigned inbound for traffic arriving from the respective VLAN.
    

 **Configuration in CLI:**

```
enable
configure terminal
interface GigabitEthernet0/2.60
ip access-group V60 in
interface GigabitEthernet0/2.70
ip access-group V70 in
exit
end
write memory
```
![Subinterface-R3](../images/Pasted%20image%2020250922230724.png)


### ACL Overview Table of Selected Rules

| VLAN                    | Allowed Access (LAN)                                           | Denied Access (LAN)                                         | Internet | Note                                                                 |
| ----------------------- | -------------------------------------------------------------- | ----------------------------------------------------------- | -------- | -------------------------------------------------------------------- |
| **V20 - Sales**         | Servers `10.31.10.10/11`, Logistics V40, Warehouse V60         | Finance V30, Management V50, Admin PC                       | YES      | Exceptions in V20 for responses from the printer to V30/V40/V50/V60. |
| **V30 - Finance**       | Servers `10.31.10.10/11`, Printer `10.31.20.10`                | All internal VLANs                                          | YES      | Finance is isolated; only servers and printer are allowed.           |
| **V40 - Logistics**     | Servers `10.31.10.10/11`, Printer `10.31.20.10`, Warehouse V60 | Finance V30, Sales V20, Management V50                      | YES      | Two-way communication V40 <-> V60 allowed. Remote logistics control. |
| **V50 - Management**    | All internal VLANs `10.31.0.0/16`                              | —                                                           | YES      | Full access (including servers and printer).                         |
| **V60 - Warehouse**     | Servers `10.31.10.10/11`, Printer `10.31.20.10`, Logistics V40 | Finance V30, Management V50, Sales V20                      | YES      | Access to Logistics V60 <-> V40 is allowed.                          |
| **V70 - Guest**         | INTERNET                                                       | Entire `10.31.0.0/16` and Internal switching & routing zone | YES      | Guests have internet only. Other VLANs have access to Guest.         |
| **V10 - Servers/Admin** | Access to internal network                                     | —                                                           | YES      | Network administration.                                              |

>**Note:** Internal VLANs may forward traffic towards the Guest VLAN, but all communication is blocked on the Guest side – Guest is allowed access only to the internet.

### Conclusion

The ACLs applied on routers R2 and R3 accurately reflect the designed policy: Sales, Finance, Logistics, and Warehouse have access only to servers, the printer, and permitted partner VLANs. Management has full access, while Guest is restricted to internet only. Pings performed during diagnostics confirmed that the permitted communication flows function correctly and that blocked traffic is denied, which matches the configured ACL policy table.

## 6.8 - Summary

Chapter 6 demonstrates the complete process of securing the network: from creating user accounts and passwords, protecting the console, and enabling remote SSH access, to configuring MOTD banners and a separate management VLAN, followed by the detailed implementation of Access Control Lists.

The ACLs were configured on routers R2 and R3, applied to the appropriate interfaces, and tested with pings between VLANs. The tests confirmed that the access policy works: servers and the printer are available as intended, sensitive VLANs are protected, Management has full access, and Guest can reach only the internet.

Port security and administrative shutdown of unused ports were omitted this time, since they were already covered in the first project _small-cafe-network_. In this type of company, where only employees have access to the internal network, these measures are not as critical.

**Continue to the next chapter: Diagnostics**




















































