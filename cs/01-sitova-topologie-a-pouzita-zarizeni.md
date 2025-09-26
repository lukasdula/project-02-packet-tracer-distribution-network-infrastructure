
# **1 - Síťová topologie a použitá zařízení**

## 1.1 - Úvod

Projekt představuje střední model firemní sítě **distribuční společnosti**, navržený pro simulaci reálného provozu a rozšířený o více VLAN, samostatné směrovače a oddělené zóny. Síť je navržena tak, aby ukazovala principy segmentace, směrování a bezpečnosti v prostředí odpovídajícím úrovni **CCNA I**.

Topologie zahrnuje **sedm VLAN**, které oddělují jednotlivé části podniku. Každá VLAN má svůj adresní prostor a default gateway na routeru R2 nebo R3. Toto uspořádání zvyšuje úroveň bezpečnosti a umožňuje centrální správu celé sítě.

![TYPOLOGY-MAP](../images/Pasted%20image%2020250926200556.png)

**Síť je rozdělena do devíti logických zón:**

1. **ISP / Internet Zone** – poskytovatel připojení (ISP router) a hraniční router R1, který zajišťuje propojení do internetu a provádí NAT/PAT.
    
2. **Internal Switching & Routing Zone** – centrální část tvořená routery R2 a R3 a dvěma switchemi. Slouží ke směrování mezi VLAN, k přenosu provozu a k nastavování hlavních funkcí sítě, včetně centrální správy a bezpečnostních opatřeních
    
3. **Admin-Server (VLAN 10, SW1)** – servery se službami (DHCP, DNS, HTTP, Syslog), administrátorské PC stanice spravující celou firemní síť a interní Wi-Fi pro zaměstnance.
    
4. **Sales (VLAN 20, SW1)** – kancelářské pracoviště s uživatelskými PC a síťovou tiskárnou pro obchodní oddělení.
    
5. **Finance (VLAN 30, SW1)** – samostatně oddělené oddělení financí s vlastním pracovním PC.
    
6. **Logistics (VLAN 40, SW1)** – **Logistics Zone (VLAN 40, SW1)** – kancelářské pracoviště určené pro správu skladu a logistiky.
    
7. **Management (VLAN 50, SW1)** – dvě pracovní stanice vedení společnosti.
    
8. **Warehouse (VLAN 60, SW2)** – firemní logistický sklad se dvěma počítači.
    
9. **Guest (VLAN 70, SW2)** – síť pro hosty, která neslouží k práci a umožňuje pouze přístup do internetu.
    

## 1.2 - Použitá zařízení

| Typ zařízení        | Model           | Využití v síti                                                                                                 |
| ------------------- | --------------- | -------------------------------------------------------------------------------------------------------------- |
| **Router R1**       | Cisco 2911      | Hraniční propojení s ISP; NAT/PAT pro odchozí provoz; směrování do internetu. Připojen k R2 a k ISP routeru.   |
| **Router R2**       | Cisco 2911      | Vnitřní směrování mezi VLAN; default gateway pro VLAN 10/20/30/40/50; propojení na R1 a R3; trunk ke SW1.      |
| **Router R3**       | Cisco 2911      | Směrování pro VLAN 60/70; propojení na R2; trunk ke SW2.                                                       |
| **ISP Router**      | Cisco 2911      | Simulovaný router poskytovatele připojení (WAN/Internet).                                                      |
| **Switch SW1**      | Cisco 2960-24TT | Připojení serverů, administrátorského PC a kancelářských PC; access porty pro VLAN 10/20/30/40/50; trunk k R2. |
| **Switch SW2**      | Cisco 2960-24TT | Připojení PC ve VLAN 60 a 70; access porty pro VLAN 60/70; trunk k R3.                                         |
| **Server SRV01**    | Server-PT       | DHCP a Syslog server ve VLAN 10; statická IP.                                                                  |
| **Server SRV02**    | Server-PT       | DNS a HTTP server ve VLAN 10; statická IP.                                                                     |
| **Admin PC**        | PC-PT           | Administrátorské PC ve VLAN 10; statická IP, správa celé sítě.                                                 |
| **Access Point**    | AccessPoint-PT  | Wi-Fi přístup ve VLAN 10; IP z DHCP.                                                                           |
| **Sales PCs**       | PC-PT           | Dvě kancelářské stanice ve VLAN 20; IP z DHCP.                                                                 |
| **Sales Laptop**    | Laptop-PT       | Přenosný počítač ve VLAN 10 (Wi-Fi přístup); IP z DHCP.                                                        |
| **Printer (Sales)** | Printer-PT      | Síťová tiskárna ve VLAN 20; statická IP.                                                                       |
| **Finance PC**      | PC-PT           | Jedna pracovní stanice ve VLAN 30; IP z DHCP.                                                                  |
| **Logistics PCs**   | PC-PT           | Dvě pracovní stanice ve VLAN 40; IP z DHCP.                                                                    |
| **Management PCs**  | PC-PT           | Dvě pracovní stanice vedení společnosti ve VLAN 50; IP z DHCP.                                                 |
| **Warehouse PCs**   | PC-PT           | Dvě pracovní stanice ve VLAN 60 (sklad); IP z DHCP.                                                            |
| **Guest PC**        | PC-PT           | Jedna pracovní stanice ve VLAN 70; IP z DHCP, přístup pouze do internetu.                                      |

V celé síti se používají straight-through měděné kabely, které jsou standardní volbou pro propojení rozdílných zařízení 


## 1.3 - Přehled topologie – fyzická propojení

| Zařízení             | Rozhraní        | Připojeno k ->             | Rozhraní (peer) | Poznámka                                          |
| -------------------- | --------------- | -------------------------- | --------------- | ------------------------------------------------- |
| **ISP Router**       | Gi0/0           | Router R1                  | Gi0/1           | WAN 100.64.5.0/30 (ISP=100.64.5.1, R1=100.64.5.2) |
| **Router R1 (Edge)** | Gi0/2           | Router R2                  | Gi0/0           | P2P 10.31.0.0/30 (R1=10.31.0.1, R2=10.31.0.2)     |
| **Router R2 (Core)** | Gi0/2           | Router R3                  | Gi0/1           | P2P 10.31.1.0/30 (R2=10.31.1.1, R3=10.31.1.2)     |
| **Router R2 (Core)** | Gi0/1           | Switch SW1                 | Gi0/2           | TRUNK (VLAN 10,20,30,40,50)                       |
| **Router R3**        | Gi0/2           | Switch SW2                 | Gi0/1           | TRUNK (VLAN 60,70)                                |
| **Switch SW1**       | Fa0/14          | Access Point (AP-WIFI)     | port0           | VLAN 10 (AP z DHCP)                               |
| **Switch SW1**       | Fa0/12          | Server SRV01 (DHCP+Syslog) | Gig1            | VLAN 10 (statická IP)                             |
| **Switch SW1**       | Gig0/1          | Server SRV02 (DNS+HTTP)    | Gig1            | VLAN 10 (statická IP)                             |
| **Switch SW1**       | Fa0/1           | PC-1 (Admin)               | Fa0             | VLAN 10 (statická IP)                             |
| **Switch SW1**       | Fa0/2           | PC-2 (SALES)               | Fa0             | VLAN 20 (DHCP)                                    |
| **Switch SW1**       | Fa0/3           | PC-3 (SALES)               | Fa0             | VLAN 20 (DHCP)                                    |
| **Switch SW1**       | Fa0/10          | Printer (SALES)            | Fa0             | VLAN 20 (statická IP)                             |
| **Switch SW1**       | _Wi-Fi přes AP_ | Laptop (Admin-Server)      | _bezdrátově_    | VLAN 10 (DHCP přes AP)                            |
| **Switch SW1**       | Fa0/4           | PC-4 (FINANCE)             | Fa0             | VLAN 30 (DHCP)                                    |
| **Switch SW1**       | Fa0/5           | PC-5 (LOGISTICS)           | Fa0             | VLAN 40 (DHCP)                                    |
| **Switch SW1**       | Fa0/6           | PC-6 (LOGISTICS))          | Fa0             | VLAN 40 (DHCP)                                    |
| **Switch SW1**       | Fa0/7           | PC-7 (MANAGEMENT)          | Fa0             | VLAN 50 (DHCP)                                    |
| **Switch SW1**       | Fa0/08          | PC-8 (MANAGEMENT)          | Fa0             | VLAN 50 (DHCP)                                    |
| **Switch SW2**       | Fa0/9           | PC-9 (WAREHOUSE)           | Fa0             | VLAN 60 (DHCP)                                    |
| **Switch SW2**       | Fa0/10          | PC-10 (WAREHOUSE)          | Fa0             | VLAN 60 (DHCP)                                    |
| **Switch SW2**       | Fa0/11          | PC-11 (GUEST)              | Fa0             | VLAN 70 (DHCP)                                    |



## 1.4 - Shrnutí

V této úvodní kapitole se představuje topologie distribuční firmy na úrovni **CCNA I**, popsána použitá zařízení a fyzické propojení včetně portů. Tyto prvky tvoří základ pro další konfiguraci a logické rozdělení sítě.


---

**Pokračovat na další kapitolu:** [Adresní a VLAN plánování](02-adresni-a-vlan-planovani.md)























































































