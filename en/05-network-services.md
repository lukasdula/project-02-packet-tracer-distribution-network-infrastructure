
# 5 - **Network Services**


## 5.1 - Introduction

**Network services** form the basis of corporate infrastructure, because they ensure efficient operation without the need for manual management. This chapter describes **internal services** available only within the corporate network.  

The goal is to show the role of DHCP, DNS, HTTP, Syslog, NTP. **DHCP** assigns addresses to clients, **DNS** converts names to IP addresses, **HTTP** provides access to the internal web, **NTP** synchronizes time, **Syslog** collects logs.  

Services are divided between **two servers in VLAN 10 Admin-Server**. **SRV01** provides DHCP and Syslog, **SRV02** provides DNS and HTTP. This way, network management is clear and secure.  

## 5.2 - DHCP

**Dynamic Host Configuration Protocol (DHCP)** automates the assignment of IP addresses and other network parameters, which simplifies network management and reduces the risk of errors. In the project, the DHCP service operates on server **SRV01 in VLAN 10 (Admin-Server)**, which has the static address 10.31.10.10. To make it possible for clients in other VLANs to communicate with the DHCP server, it is necessary to configure the command **ip helper-address** on all subinterfaces of client VLANs on routers R2 and R3.  


### Setting ip helper-address on routers

#### ROUTER R2  

- Log in to configuration mode and open each subinterface for VLAN 20, 30, 40 and 50.  
      
- On each subinterface enter the command `ip helper-address 10.31.10.10`.  

**Configuration in CLI:**

```
R2>enable
R2#configure terminal
R2(config)#interface gi0/1.20
R2(config-subif)#ip helper-address 10.31.10.10
R2(config-subif)#interface gi0/1.30
R2(config-subif)#ip helper-address 10.31.10.10
R2(config-subif)#interface gi0/1.40
R2(config-subif)#ip helper-address 10.31.10.10
R2(config-subif)#interface gi0/1.50
R2(config-subif)#ip helper-address 10.31.10.10
R2(config-subif)#end
R2#write memory
```
![ip-helper-R2](../images/Pasted%20image%2020250915163252.png)

#### ROUTER R3  

- Log in to configuration mode and open each subinterface for VLAN 60 and 70.  
    
- On each subinterface enter the command `ip helper-address 10.31.10.10`.  

**Configuration in CLI:**  

```
R3>enable  
R3#configure terminal  
R3(config)#interface gi0/2.60  
R3(config-subif)#ip helper-address 10.31.10.10  
R3(config-subif)#interface gi0/2.70  
R3(config-subif)#ip helper-address 10.31.10.10  
R3(config-subif)#end  
R3#write memory  
```
![ip-helper-R3](../images/Pasted%20image%2020250915163434.png)

### DHCP pool configuration on SRV01  

On server SRV01 DHCP pools are created for individual VLANs. Each pool has a default gateway, DNS server, start address and maximum number of users. This way, devices in all VLANs receive dynamic addresses, while servers and other key elements have static addresses.  

| Pool Name               | Default Gateway | DNS Server   | Start IP Address | Subnet Mask   | Max User |
|--------------------------|-----------------|--------------|------------------|---------------|----------|
| serverPool (Server-Admin) | 10.31.10.1      | 10.31.10.11  | 10.31.10.100     | 255.255.255.0 | 30       |
| Sales                   | 10.31.20.1      | 10.31.10.11  | 10.31.20.20      | 255.255.255.0 | 30       |
| Finance                 | 10.31.30.1      | 10.31.10.11  | 10.31.30.30      | 255.255.255.0 | 30       |
| Logistics               | 10.31.40.1      | 10.31.10.11  | 10.31.40.40      | 255.255.255.0 | 30       |
| Management              | 10.31.50.1      | 10.31.10.11  | 10.31.50.50      | 255.255.255.0 | 30       |
| Warehouse               | 10.31.60.1      | 10.31.10.11  | 10.31.60.60      | 255.255.255.0 | 30       |
| Guest                   | 10.31.70.1      | 10.31.10.11  | 10.31.70.70      | 255.255.255.0 | 30       |

![SRV01-DHCP](../images/Pasted%20image%2020250915180653.png)

### Configuration steps:

- On server SRV01 open Services → DHCP.  
    
- Select the default `serverPool`, fill in the values and click Save.  
    
- Then create new pools for other VLANs and fill in the corresponding values according to   the table.  
    
- Save the configuration with `Save`.  


>**note:** The pool `serverPool` in Packet Tracer cannot be removed or renamed. For DHCP to work, this pool must be filled in, even if VLAN 10 uses static addresses. Other pools can already be named according to VLAN or department.  



### Conclusion to DHCP

For correct DHCP operationit is necessary to set ip helper-address on routers R2 and R3, so that requests from VLANs are forwarded to server SRV01. This way address assignment works for all client VLANs 20 to 70. A laptop connected via WiFi to AP in VLAN 10 also connects to DHCP and receives an address from the default serverPool.  

**Pings between individual VLANs are successful**, which confirms correct DHCP operation and routing; detailed diagnostics is given in chapter 8.  

>*note:* Due to Packet Tracer limitations, after restart assigned addresses may change, because the server does not remember the records of assigned addresses. That is why in documentation ranges are given, not individual IP addresses. For this reason, in larger projects it is more effective to configure DHCP directly in CLI on the router, where the configuration behaves more realistically. This approach is demonstrated in the next project. 


## 5.3 - DNS

**Domain Name System (DNS)** provides translation of domain names into numeric IP addresses. It allows users not to enter server IP addresses, but to use readable names. In this project the DNS service operates on **server SRV02 (10.31.10.11)** in VLAN 10. In our case it is a demonstration of connecting to the internal company web, so it is necessary to create the corresponding DNS record first.

>**note:** In this project we do not simulate connection to the public internet behind an ISP. We focus on internal company services. The DNS configuration is performed only via the server GUI for basic demonstration of functions. Detailed configuration and use of DNS through CLI on the router is shown in the next project.


### SRV02

- On server SRV02 the DNS service is activated and a record is created for the internal web server:
    

|Record Name|IP Address|Description|
|---|---|---|
|www.firma.local|10.31.10.11|Internal web server|

### Configuration steps:

1. On server SRV02 open Services → DNS.
    
2. Turn on the service with On.
    
3. Add a new record:
    
    - Name: `www.firma.local`
        
    - Address: `10.31.10.11`
        
    - Type: A (host record)
        
4. Add the configuration with `Add` and save with `Save.`


![DNS](../images/Pasted%20image%2020250915201540.png)


### Verification of functionality

- On PC7 in VLAN 50 (Management) open the command prompt and enter:
    

```
ping www.firma.local
```

- The reply comes from address **10.31.10.11**, which confirms the correct name-to-IP translation and the functionality of the DNS service.

![ping-DNS](../images/Pasted%20image%2020250915201819.png)

### Conclusion

A DNS record `www.firma`.local is created pointing to the internal web server SRV02. The test on PC7 in VLAN 50 confirms functionality – the domain name is translated into an IP address and the reply is successful. In the next part we focus on the configuration and verification of the **HTTP** service.

## 5.4 - HTTP


The **HTTP (Hypertext Transfer Protocol)** service is the basic protocol for transferring web pages. In this project it is used to demonstrate access to the company web, which is located on server SRV02 in VLAN 10 (Admin-Server). Access to the web is handled internally, without connection to the public internet. DNS translates the name `www.firma.local` and the HTTP server then provides the web content.

In real practice, if the web and the database are located inside the company, it continues to function even during internet outages. If it depends on an external cloud, the page loads but functions are unavailable. This project shows an internal scenario that is available even without internet.

### Configuration of the HTTP server on SRV02

- On server SRV02 open the **Services → HTTP tab.**
    
- Switch the option **HTTP Service** to **On**.
    
- Optionally edit the default web page in the field **Index.html** (we keep the default sample content of Packet Tracer).

![HTTP](../images/Pasted%20image%2020250915202509.png)


### Verification of functionality

- On PC4 in VLAN 30 open the web browser (Desktop → Web Browser).
    
- Enter `www.firma.local` in the address bar.
    
- The page loads from the HTTP server SRV02, which confirms the functionality of both DNS and HTTP services.


![HTTP-web](../images/Pasted%20image%2020250915202625.png)


### Conclusion

The built-in **HTTP server** on SRV02 is activated and the connection from the client station in VLAN 30 is successfully tested using the address `www.firma.local`. This step completes the previous DNS configuration and proves the functionality of the company web inside the internal network. In the next part we focus on the **SYSLOG** service.

## 5.5 - Syslog


**Syslog** is a service used for collecting and centrally monitoring system messages and events in the network. It helps administrators monitor traffic, identify errors and respond to security incidents. In our project it is used to demonstrate storing logs from active devices on server SRV01.

>**note:** In Packet Tracer the Syslog in the server GUI is simplified and displays only limited events. More detailed Syslog configuration directly in CLI on routers (using timestamps, buffer and extended logs) is shown in the larger next project, where it is possible to demonstrate the wider options of this service more realistically.

### Configuration of the Syslog server (SRV01)

- On server SRV01 open the `Services → Syslog tab`.
    
- Switch the option **Syslog Service** to **On**.
    
- The server now receives logs from other devices in the network.


![Syslog-service](../images/Pasted%20image%2020250915203832.png)


### Configuration of network devices

Example of configuration on router R2:

```
R2> enable
R2# configure terminal
R2(config)# logging 10.31.10.10
R2(config)# end
R2# write memory
```
![logging-Syslog](../images/Pasted%20image%2020250915204147.png)

The same commands are used on other routers and switches so that all send their logs to SRV01, but for demonstration only the CLI output from R2 is shown here.

### Generating a log

To demonstrate the Syslog function a change of device name is performed on router R1:

```
R1(config)#hostname R1-Test
```
![syslog-test](../images/Pasted%20image%2020250915212711.png)

Tato změna se okamžitě projevila v CLI a měla by být zaznamenána v Syslogu na SRV01. 


### Verification of functionality

- On SRV01 open the Services → Syslog tab.
    
- In the logging window messages about configuration changes on R1 are displayed.
    

![syslog-check](../images/Pasted%20image%2020250915212648.png)

### Conclusion

The **Syslog server** is activated on SRV01 and the network devices are configured to send logs. The test action confirms that the configuration is correctly recorded and available for the administrator on SRV01.

## 5.6 - NTP


**NTP (Network Time Protocol)** ensures that all network devices operate with the same time. Accurate time is important for logging events, security protocols, authentication and diagnostics. In our network router R1 is configured as the main time server and other routers synchronize their time from it.

>note: In practice NTP is also configured on switches, but for simplification it is demonstrated here only on routers.

### Configuration of the time server R1

- Set the current time and date.
    
- Define the time zone.
    
- Activate the NTP server with stratum 1.
    

**Configuration in CLI:**

```
R1> enable
R1# clock set 12:00:00 Jul 05 2025
R1(config)# configure terminal clock timezone CEST 2
R1(config)# ntp master 1
exit
write memory
```
![NTP-R1](../images/Pasted%20image%2020250915233646.png)

>note: Router R1 is configured as **NTP master** with stratum 1, which makes it the main time source for the whole network. All other devices synchronize their time according to it.

### Configuration of client R2

- Set the time zone.
    
- Define R1 as the NTP server.
    

**Configuration in CLI:**

```
R2> enable
R2(config)# clock timezone CEST 2
R2(config)# ntp server 10.31.0.1
exit
write memory
```
![NTP-R2](../images/Pasted%20image%2020250915234912.png)


### Configuration of client R3

- Set the time zone.
    
- Define R1 as the NTP server.
    

**Configuration in CLI:**

```
R3> enable
R3(config)# clock timezone CEST 2
R3(config)# ntp server 10.31.0.1
exit
write memory
```
![NTP-R3](../images/Pasted%20image%2020250915235024.png)


### Verification of functionality

- On all routers it is possible to check the time and the synchronization source.
    
- For demonstration we show the verification only from R3.
    

**Configuration in CLI:**

```
show clock detail
show ntp status
```
![NTP-diagnostic](../images/Pasted%20image%2020250915235523.png)

### Conclusion

Router **R1** is configured as the main time server using the command `ntp master 1`.  
Routers **R2 and R3** are configured as clients with a reference to server R1 (`ntp server 10.31.0.1`).  
The following diagnostics show that the time source on R3 is indeed NTP, and the status **Clock is synchronized** confirms successful synchronization.

This ensures that all network devices use a unified time, which is essential for correct event logging, security protocols and network diagnostics.


## 5.7 - Wi-Fi

In this project Wi-Fi is used only by the **administrator on his laptop**.  
The Access Point (AP) is placed in **VLAN 10 (Management)** and provides the SSID _wifi-admin_ with the password **Logistic123**.  
The network is secured with **WPA2-PSK**, and the administrator therefore has full access to the whole network, the same as PCs in VLAN 10.

Due to the limitations of Packet Tracer we do not demonstrate employee Wi-Fi separated into its own VLAN with **restricted access**.  
This project shows only a basic implementation of the AP for administration.  
A complete Wi-Fi infrastructure for employees (separate VLAN, ACL and limited access only to the internet and selected services) is demonstrated in the following project.

#### Access Point

![AC-wifi](../images/Pasted%20image%2020250922231501.png)

#### Laptop-admin

![laptop-wifi](../images/Pasted%20image%2020250922231440.png)


## 5.8 - Summary

In this chapter the key internal services in VLAN 10 are configured: DHCP and Syslog on SRV01, DNS and HTTP on SRV02.  
DHCP is available for all VLANs through **ip helper-address**.  
DNS translates `www.firma.local` to the internal web on SRV02, which works thanks to the HTTP service.  
Syslog centralizes logs on SRV01 and NTP provides a unified time across the network, with R1 as the master.

---

**Continue to the next chapter:** [Network Security](06-network-security.md)  
