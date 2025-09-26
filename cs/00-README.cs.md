# **9 - Distribuční síťová infrastruktura**

## Úvod a cíle

Tento projekt je součástí mého studijního portfolia a praktické přípravy na kurz CCNA I. 
Nepřímo navazuje na první projekt, kde byla vytvořena síť pro menší podnik. Oproti němu je tento projekt posunutý o úroveň výš a představuje komplexnější prostředí střední podnikové sítě s širším nasazením směrování, služeb a bezpečnostních prvků.

Síť je nastavena v prostředí Cisco Packet Tracer pomocí CLI. Komunikace mezi odděleními je řešena pomocí VLAN segmentace a směrování OSPF. Jeden z routerů funguje jako ISP a doplňuje tak realistickou firemní topologii. Servery poskytují potřebnou infrastrukturu pro provoz celé sítě a komunikaci mezi jednotlivými odděleními. ACL složitější politika reguluje přístupy mezi VLANami a simuluje pravidla odpovídající reálným podnikům.

Síť je systematicky rozdělena do několika zón, které odpovídají běžné firemní infrastruktuře:

- **ISP zóna** -> simulace poskytovatele internetu
    
- **Routing and Switching zóna** -> propojení routerů, přepínačů a páteřní infrastruktury
    
- **Serverová zóna** -> SRV01 (DHCP, Syslog), SRV02 (DNS, HTTP, NTP) a Admin PC
    
- **Firemní VLANy** -> Admin-Server, Sales, Finance, Logistics, Management, Warehouse, Guest

## Struktura projektu 

1. [Síťová topologie a použitá zařízení](01-sitova-topologie-a-pouzita-zarizeni.md)
    
2. [Adresní a VLAN plánování](02-adresni-a-vlan-planovani.md)
    
3. [Základní síťová konfigurace](03‑zakladni-sitova-konfigurace.md)
    
4. [VLANy a subinterface](04-vlany-a-subinterface.md)
    
5. [Síťové služby](
    
6. [Zabezpečení sítě](
    
7. [Testování konektivity](
    
8. [Řešení problémů](
    
9. [Shrnutí a závěr](
    

## Přístup do CLI zařízení

Pro ukázkové účely byly nakonfigurovány přihlašovací údaje k síťovým zařízením. Tyto údaje umožňují vstup do CLI a následně do privilegovaného režimu.

- Uživatelské jméno (console + SSH): **distadmin**
    
- Uživatelské heslo: **Logistic25**
    
- Enable secret (privilegovaný režim): **Pallet25**
    

Pro účely projektu jsou použita jednodušší tematická hesla. V reálném nasazení by byla využita komplexnější a delší hesla (12+ znaků, kombinace velkých/malých písmen, číslic a speciálních znaků).

## Použité nástroje

- Cisco Packet Tracer (verze 8.2+) -> simulace síťového prostředí
    
- Visual Studio Code / Obsidian -> psaní a úprava dokumentace
    

## Jak spustit projekt

1. Otevři .pkt soubor v Cisco Packet Traceru (8.2+).
    
2. Postupuj podle složek projektu 01–09.
    

## Klíčové funkce projektu

- OSPF směrování mezi routery -> dynamické sdílení tras
    
- Subnetting s maskami /30 pro efektivní využití adres
    
- Rozšířené VLAN segmentace -> oddělení firemních oddělení
    
- Management VLAN 99 -> oddělená správa přepínačů přes SSH
    
- DHCP, DNS, HTTP, NTP a Syslog služby
    
- ACL politika -> detailní a realistické omezení provozu mezi VLANami
    
* Síťová diagnostika -> systematické ověřování provozu a dostupnosti služeb
    
- Ukázky řešení problémů (troubleshooting)
    

## Poznámka autora

Tento druhý projekt mě hodně bavil, protože jsem díky zkušenostem z prvního projektu už lépe chápal hloubku sítě a dokázal využít nabyté znalosti pro složitější topologii. Poprvé jsem si vyzkoušel konfiguraci OSPF a ocenil, že směrování má lepší sdílení mezi routery. 

Velký přínos měla aplikace ACL politiky, která síti dodává realističtější podobu podnikových pravidel a umožňuje přesně definovat povolené i zakázané přístupy. Novinkou byla také VLAN 99, díky níž lze provádět bezpečnou správu přepínačů oddělenou od zbytku provozu, a rozšíření služeb o NTP a Syslog. I subnetting se pro mě stal důležitým nástrojem, který mi přinesl fantasii i větší jistotu při adresování a návrhu sítě.

Celý projekt ukazuje, že i střední model sítě může integrovat široké množství funkcí a přesto fungovat jako ucelený a stabilní celek. Do další fáze se chci posunout projektem zaměřeným na menší model enterprise sítě, kde plánuji propojení dvou poboček pomocí GRE tunelu, více nových funkcí, přidání desítek dalších zařízení a detailnější ukázku využití Wi-Fi. 

---

© 2025 - Lukáš Dula | Domácí síťový lab & portfolio
