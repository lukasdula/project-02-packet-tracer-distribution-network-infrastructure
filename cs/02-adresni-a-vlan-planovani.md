# **2 – Adresní a VLAN plánování**

## 2.1 - Úvod

V této kapitole se definuje **logické rozdělení** celé sítě a přidělují se **IP adresy** jednotlivým rozhraním. Součástí je také návrh **VLAN segmentace**, která určuje rozdělení zařízení do samostatných logických celků podle jejich funkce a umístění.

**Adresní a VLAN plán** tvoří základní strukturu sítě, na kterou navazuje konfigurace **směrování**, **přepínání** a **síťových služeb** v dalších kapitolách.



## 2.2 - IP adresace a přehled podsítí


| Zařízení / Rozhraní  | Popis              | IP adresa                       | Maska podsítě   | Výchozí brána | Přiřazená síť |
| -------------------- | ------------------ | ------------------------------- | --------------- | ------------- | ------------- |
| **ISP Router Gi0/0** | ISP -> R1          | 100.64.5.1                      | 255.255.255.252 | N/A           | 100.64.5.0/30 |
| **Router R1 Gi0/1**  | R1 -> ISP          | 100.64.5.2                      | 255.255.255.252 | N/A           | 100.64.5.0/30 |
| **Router R1 Gi0/2**  | Spoj R1 -> R2      | 10.31.0.1                       | 255.255.255.252 | N/A           | 10.31.0.0/30  |
| **Router R2 Gi0/0**  | Spoj R2 -> R1      | 10.31.0.2                       | 255.255.255.252 | N/A           | 10.31.0.0/30  |
| **Router R2 Gi0/2**  | Spoj R2 -> R3      | 10.31.1.1                       | 255.255.255.252 | N/A           | 10.31.1.0/30  |
| **Router R3 Gi0/1**  | Spoj R3 -> R2      | 10.31.1.2                       | 255.255.255.252 | N/A           | 10.31.1.0/30  |
| **SRV01**            | Server VLAN 10     | 10.31.10.10                     | 255.255.255.0   | 10.31.10.1    | 10.31.10.0/24 |
| **SRV02**            | Server VLAN 10     | 10.31.10.11                     | 255.255.255.0   | 10.31.10.1    | 10.31.10.0/24 |
| **PC1**              | Admin PC (VLAN 10) | 10.31.10.20                     | 255.255.255.0   | 10.31.10.1    | 10.31.10.0/24 |
| **PC2 / PC3**        | Sales VLAN 20      | DHCP 10.31.20.20 – 10.31.20.49, | 255.255.255.0   | 10.31.20.1    | 10.31.20.0/24 |
| **PC4**              | Finance VLAN 30    | DHCP 10.31.30.30 – 10.31.30.59  | 255.255.255.0   | 10.31.30.1    | 10.31.30.0/24 |
| **PC5 / PC6**        | Logistics VLAN 40  | DHCP 10.31.40.40 – 10.31.40.69  | 255.255.255.0   | 10.31.40.1    | 10.31.40.0/24 |
| **PC7 / PC8**        | Management VLAN 50 | DHCP 10.31.50.50 – 10.31.50.79  | 255.255.255.0   | 10.31.50.1    | 10.31.50.0/24 |
| **PC9 / PC10**       | Warehouse VLAN 60  | DHCP 10.31.60.60 – 10.31.60.89  | 255.255.255.0   | 10.31.60.1    | 10.31.60.0/24 |
| **PC11**             | Guest VLAN 70      | DHCP 10.31.70.70 – 10.31.70.99  | 255.255.255.0   | 10.31.70.1    | 10.31.70.0/24 |
| **Printer**          | Sales VLAN 20      | 10.31.20.10                     | 255.255.255.0   | 10.31.20.1    | 10.31.20.0/24 |

>**Pozn.:** **Admin PC má **statickou IP**, aby se adresa nikdy neměnila. To usnadňuje správu, bezpečnost a nastavování přístupových pravidel.


## 2.3 - VLAN plánování

Pro účely tohoto projektu jsou všechny stanice, servery a další zařízení rozděleny do samostatných VLAN. Každá VLAN má vlastní IP podsíť s maskou /24 (255.255.255.0) a výchozí bránu nakonfigurovanou na příslušném routeru (R2 nebo R3).

**Toto rozdělení umožňuje:**

- zvýšit bezpečnost oddělením jednotlivých oddělení a zón,
    
- efektivně směrovat provoz mezi VLAN prostřednictvím routerů R2 a R3,
    
- jednoduše spravovat přístupová pravidla a síťové interní služby (DHCP, DNS, HTTP, Syslog).
    

DHCP server přiděluje adresy dynamicky pro VLAN 20 až 70. Ve VLAN 10 (Admin & Server Zone) jsou klíčová zařízení nastavena se statickými adresami.

| VLAN ID | Název VLAN   | Popis / účel                                                | Přiřazená síť | Výchozí brána | Přiřazená zařízení              |
| ------- | ------------ | ----------------------------------------------------------- | ------------- | ------------- | ------------------------------- |
| 10      | Admin-Server | Servery (DHCP, DNS, HTTP, Syslog), admin PCs, interní Wi-Fi | 10.31.10.0/24 | 10.31.10.1    | SRV01, SRV02, PC1, Access Point |
| 20      | Sales        | Uživatelská PCs a tiskárna obchodního oddělení              | 10.31.20.0/24 | 10.31.20.1    | PC2, PC3, Printer               |
| 30      | Finance      | Oddělená PCs pro účetnictví                                 | 10.31.30.0/24 | 10.31.30.1    | PC4                             |
| 40      | Logistics    | PCs pro logistiku a správu skladu                           | 10.31.40.0/24 | 10.31.40.1    | PC5, PC6                        |
| 50      | Management   | Pracovní stanice vedení společnosti                         | 10.31.50.0/24 | 10.31.50.1    | PC7, PC8                        |
| 60      | Warehouse    | PCs ve skladu                                               | 10.31.60.0/24 | 10.31.60.1    | PC9, PC10                       |
| 70      | Guest Zone   | Hostovská síť s přístupem pouze k internetu                 | 10.31.70.0/24 | 10.31.70.1    | PC11                            |


## 2.4 - Shrnutí


V této části projektu se stanovuje detailní adresní plán celé sítě a rozděluje se do samostatných VLAN. Každá VLAN má vlastní IP podsíť s definovanou výchozí bránou a v případě klientských stanic také přiřazený DHCP pool. Statické adresy jsou vyhrazeny pouze pro servery a síťové služby ve VLAN 10 (Admin-Server) a tiskárnu.

Návrh IP adresace a VLAN segmentace představuje základní kámen pro následnou konfiguraci směrování, přepínání a síťových služeb.

**Pokračovat na další kapitolu:** [Základní síťová konfigurace](03‑zakladni-sitova-konfigurace.md)















































