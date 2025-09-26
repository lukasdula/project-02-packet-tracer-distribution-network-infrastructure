 
# **4 - Vlany a subinterface**


## 4.1 - Úvod  

**VLANy (Virtual Local Area Networks – virtuální lokální sítě)** umožňují rozdělit síť na menší logické části, které zajišťují lepší uspořádání a správu. V projektu jich je sedm – **Admin-Server, Sales, Finance, Logistics, Management, Warehouse a Guest**. Každá VLAN má svou vlastní podsíť a výchozí bránu, čímž se odděluje provoz mezi jednotlivými částmi firmy.

V kapitole se vytvářejí VLANy na switchích, nastavují se **trunk porty**, konfigurují se subrozhraní na routerech metodou **Router-on-a-Stick**, rozšiřuje se OSPF o nové VLAN a nastavují se **statické IP adresy** pro servery a administrátorské PC.

**Cílem kapitoly je** vytvořit VLANy na switchích, nakonfigurovat trunk spoje k routerům, nastavit subrozhraní na routerech R2 a R3, přidělit IP adresy jako výchozí brány, integrovat VLAN do OSPF a nastavit statické adresy pro klíčová zařízení.

## 4.2 - Vytvoření VLAN na switchích

V této části se na switchích **SW1** a **SW2** vytvořá všechny potřebné VLAN a přiřadí se jim názvy pro lepší přehlednost. Vytvoření VLAN je prvním krokem před jejich následným přiřazením k portům.

|VLAN ID|Název VLAN|Popis / účel|
|---|---|---|
|10|Admin-Server|Servery, Admin PC, interní Wi-Fi (Access Point)|
|20|Sales|Uživatelská PC a síťová tiskárna obchodního oddělení|
|30|Finance|PC účetnictví|
|40|Logistics|PC pro správu skladu a logistiky|
|50|Management|PC vedení společnosti|
|60|Warehouse|PC ve skladu|
|70|Guest|Hostovská síť pouze s přístupem k internetu|

---

### Switch SW1

- Vytvoření VLAN 10–50.
    
- Přiřazení názvů podle účelu jednotlivých VLAN.
    

**Konfigurace v CLI:**

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

- Vytvoření VLAN 60 a 70.
    
- Přiřazení názvů podle účelu jednotlivých VLAN.
    

**Konfigurace v CLI:**

```
enable
configure terminal
vlan 60
name Warehouse
vlan 70
name Guest
exit
end
```
![name-vlan-SW2](../images/Pasted%20image%2020250914133918.png)
## 4.3 - Přiřazení portů do VLAN

Po vytvoření VLAN na switchích je potřeba přiřadit fyzické porty jednotlivým VLAN. Každý port, ke kterému se připojuje koncové zařízení (PC, server, tiskárna nebo Access Point), se nastavuje jako **access port** a zařazuje se do příslušné VLAN. Tím se zajistí, že každé zařízení je součástí správného logického segmentu.

| Port SW1 | VLAN ID | Přiřazené zařízení |
| -------- | ------- | ------------------ |
| Fa0/1    | 10      | Admin PC           |
| Fa0/12   | 10      | Server SRV01       |
| Gi0/1    | 10      | Server SRV02       |
| Fa0/14   | 10      | Access Point       |
| Fa0/2    | 20      | PC2                |
| Fa0/3    | 20      | PC3                |
| Fa0/13   | 20      | Printer            |
| Fa0/4    | 30      | PC4                |
| Fa0/5    | 40      | PC5                |
| Fa0/6    | 40      | PC6                |
| Fa0/7    | 50      | PC7                |
| Fa0/8    | 50      | PC8                |

| Port SW2 | VLAN ID | Přiřazené zařízení |
| -------- | ------- | ------------------ |
| Fa0/9    | 60      | PC9                |
| Fa0/10   | 60      | PC10               |
| Fa0/11   | 70      | PC11               |

---

### Switch SW1

- Přepnutí portů do režimu access.
    
- Zařazení portů do správných VLAN podle zapojených zařízení.
    

**Konfigurace v CLI:**

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

- Přepnutí portů do režimu access.
    
- Zařazení portů do správných VLAN pro zařízení skladu a hostů.
    

**Konfigurace v CLI:**

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


### Závěr  

V rámci částí **4.2 a 4.3** jsou na switchích **SW1** a **SW2** vytvořeny **VLANy** s přiřazenými názvy a následně jsou nastaveny porty pro koncová zařízení. Konfigurace je ověřena na obou switchích pomocí příkazů `show vlan brief` a `show running-config`, které potvrzují **správné vytvoření VLAN i přiřazení portů podle návrhu**.

## 4.4 - Konfigurace trunků na switchích

Trunk porty umožňují přenášet rámce více VLAN přes jedno fyzické rozhraní. V této části se nastavují trunk porty na switchích, které propojují switche s routery. Zapouzdření 802.1Q a konfigurace subrozhraní se řeší v následující části (4.5).

|Spoj|Rozhraní na switchi|Rozhraní na routeru|Režim|
|---|---|---|---|
|SW1–R2|Gi0/2|Gi0/1|trunk|
|SW2–R3|Gi0/1|Gi0/2|trunk|

---

### Switch SW1

- Přepnutí portu Gi0/2 do trunk režimu.
    

**Konfigurace v CLI:**

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

- Přepnutí portu Gi0/1 do trunk režimu.
    

**Konfigurace v CLI:**

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


### Závěr  

V části 4.4 jsou na switchích SW1 a SW2 nakonfigurovány trunk porty, které propojují switche s routery. Pro kontrolu je použit příkaz `show interfaces trunk`, který potvrzuje, že trunky jsou nastaveny správně a přenášejí rámce všech požadovaných VLAN.


## 4.5 - Konfigurace subrozhraní na routerech (Router-on-a-Stick)

Aby bylo možné směrovat mezi jednotlivými VLAN, vytvářejí se na routerech **R2** a **R3** subrozhraní. Každé subrozhraní je přiřazené konkrétní VLAN, nastavuje se zapouzdření **802.1Q** a přiděluje se mu **IP adresa** fungující jako výchozí brána pro danou VLAN.

| Router | Subrozhraní | VLAN ID | IP adresa (gateway) | Maska         |
| ------ | ----------- | ------- | ------------------- | ------------- |
| R2     | Gi0/1.10    | 10      | 10.31.10.1          | 255.255.255.0 |
| R2     | Gi0/1.20    | 20      | 10.31.20.1          | 255.255.255.0 |
| R2     | Gi0/1.30    | 30      | 10.31.30.1          | 255.255.255.0 |
| R2     | Gi0/1.40    | 40      | 10.31.40.1          | 255.255.255.0 |
| R2     | Gi0/1.50    | 50      | 10.31.50.1          | 255.255.255.0 |
| R3     | Gi0/2.60    | 60      | 10.31.60.1          | 255.255.255.0 |
| R3     | Gi0/2.70    | 70      | 10.31.70.1          | 255.255.255.0 |

---
### Router R2

**Postup konfigurace:**

- Vytvořit subrozhraní pro každou VLAN.
    
- Nastavit zapouzdření 802.1Q (`encapsulation dot1q <VLAN>`).
    
- Přiřadit IP adresu jako výchozí bránu VLAN.
    

---


**Konfigurace v CLI:** **

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


**Konfigurace v CLI:**

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

### Závěr  

Na routerech R2 a R3 jsou vytvořená subrozhraní se zapouzdřením 802.1Q a přiřazenými IP adresami (gateway/VLAN). Funkčnost je ověřena diagnostikou (`show ip interface brief`, `show running-config interface …`) a testovacími pingy na brány jednotlivých VLAN. 
**Všechny testy byly úspěšné**.



## 4.6 - Přidání VLAN sítí do OSPF

Aby probíhala komunikace i mezi VLAN, které jsou za různými routery, přidávají se do OSPF na routerech **R2** a **R3** nově vytvořené VLAN sítě. Díky tomu mohou sítě za R2 a R3 vzájemně komunikovat. Páteřní OSPF sousedství zůstává beze změny podle předchozí konfigurace.

|Router|Přidávané sítě do OSPF|Area|
|---|---|---|
|R2|10.31.10.0/24, 10.31.20.0/24, 10.31.30.0/24, 10.31.40.0/24, 10.31.50.0/24|0|
|R3|10.31.60.0/24, 10.31.70.0/24|0|

---

### Router R2

**Postup konfigurace:**

- Na každém routeru se do OSPF doplní příslušné VLAN sítě.
    
- Subrozhraní zůstávají pasivní (neformují sousedství v uživatelských VLAN), aktivní zůstává pouze páteřní P2P rozhraní mezi R2 a R3 dle kap. 3.
    


**Konfigurace v CLI:**

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

**Konfigurace v CLI:**

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


### Závěr

OSPF pro VLAN sítě je nakonfigurovaný. Diagnostika pomocí `show ip ospf neighbor` potvrzuje navázaná sousedství a `show ip route ospf` ukazuje přidané trasy k VLAN sítím na druhém routeru. **Testovací pingy na brány vzdálených VLAN procházejí**. Výsledkem je, že VLAN sítě na R2 a R3 se navzájem znají a jsou plně dosažitelné.


## 4.7 - Statické IP adresy pro servery a Admin PC (VLAN 10)

Ve VLAN 10 se klíčovým zařízením přidělují **statické IP adresy**, aby byly stabilně dostupné pro správu a poskytované služby. Také nastavujeme statickou IP tiskárně ve VLAN 20. 
Ostatní stanice v síti zůstávají na DHCP.

| Zařízení                    | Rozhraní | IP adresa   | Maska         | Výchozí brána |
| --------------------------- | -------- | ----------- | ------------- | ------------- |
| Server SRV01 (DHCP, Syslog) | Gi1      | 10.31.10.10 | 255.255.255.0 | 10.31.10.1    |
| Server SRV02 (DNS, HTTP)    | Gi1      | 10.31.10.11 | 255.255.255.0 | 10.31.10.1    |
| Admin PC                    | Fa0      | 10.31.10.20 | 255.255.255.0 | 10.31.10.1    |

>Pozn.: DNS pro tato zařízení nastavíme při konfiguraci DNS v další kapitole.

---

**Postup:**

- **Server SRV01** -> Desktop ->  IP Configuration -> ip adresa, maska, výchozí brána
    
![static-SRV01](../images/Pasted%20image%2020250914203000.png)


* **Server SRV02** -> Desktop -> IP Configuration -> ip adresa, maska, výchozí  brána
    

![static-SRV02](../images/Pasted%20image%2020250914203020.png)



* **Admin PC**1 -> Desktop -> IP Configuration -> ip adresa, maska, výchozí brána
    

![static-PC1](../images/Pasted%20image%2020250914203040.png)

#### Printer VLAN 20

**Síťová tiskárna**  byla nakonfigurována s následujícími parametry:

- IP adresa: **10.31.20.10**
    
- Maska sítě: **255.255.255.0 (/24)**
    
- Přiřazená síť: **10.31.20.0/24**
    
- Výchozí brána: **10.31.20.1** 

### Závěr

Statické IP adresy pro **SRV01, SRV02 a Admin PC, tiskárnu/printer** jsou nastavené. Funkčnost ověřujeme diagnostikou -> **pingy z R2 směrem k zařízení ve **VLAN 10 jsou úspěšné.



## 4.8 - Shrnutí

V této části se vytvářejí VLANy a přiřazují se porty pro jednotlivá zařízení na switchích SW1 a SW2. Konfigurují se trunk porty, které umožňují přenos rámců více VLAN přes jedno fyzické spojení, a na routerech R2 a R3 se nastavují subrozhraní s 802.1Q zapouzdřením a IP adresami fungujícími jako výchozí brány.

Dále se nové VLAN sítě integrují do směrovacího protokolu OSPF, aby mezi sebou mohly komunikovat i přes různé routery. Klíčovým zařízením ve VLAN 10 (servery a Admin PC) a tiskárně ve VLAN 20 se přidělují statické IP adresy. Funkčnost celé konfigurace je ověřena diagnostikou a všechny testovací pingy proběhly úspěšně bez ztrát.

---

**Pokračovat na další kapitolu:** [Síťové služby](05-sitove-sluzby.md)


















































































































