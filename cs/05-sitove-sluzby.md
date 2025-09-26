# **5 - Síťové služby**


## 5.1 - Úvod

**Síťové služby** tvoří základ podnikové infrastruktury, protože zajišťují efektivní provoz bez nutnosti ruční správy. V této kapitole budou popsány **interní služby** dostupné pouze v rámci firemní sítě.

Cílem je ukázat roli **DHCP, DNS, HTTP, Syslog, NTP. **DHCP** přiděluje adresy klientům, **DNS** převádí jména na IP adresy, **HTTP** zpřístupňuje interní web, **NTP** synchronizuje čas, **Syslog** shromažďuje logy.

Služby jsou rozděleny mezi **dva servery ve VLAN 10 Admin-Server**. **SRV01** zajišťuje DHCP a Syslog, **SRV02** poskytuje DNS a HTTP. Díky tomu je správa sítě přehledná a bezpečná.

## 5.2 - DHCP

**Dynamic Host Configuration Protocol (DHCP)** automatizuje přidělování IP adres a dalších síťových parametrů, čímž zjednodušuje správu sítě a snižuje riziko chyb. V projektu je DHCP služba provozována na **serveru SRV01** ve VLAN 10 (Admin-Server), který má statickou adresu 10.31.10.10. Aby bylo možné, aby klienti v jiných VLAN komunikovali s DHCP serverem, je nutné na routerech R2 a R3 nakonfigurovat příkaz **ip helper-address** na všech subrozhraních klientských VLAN.

### Nastavení ip helper-address na routerech

#### ROUTER R2

* Přihlásit se do konfiguračního režimu a otevřít každou subinterface pro VLAN 20, 30, 40 a 50.
    
* Na každou subinterface zadat příkaz `ip helper-address 10.31.10.10`.
    

**Konfigurace v CLI:**

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

* Přihlásit se do konfiguračního režimu a otevřít každou subinterface pro VLAN 60 a 70.
    
* Na každou subinterface zadat příkaz `ip helper-address 10.31.10.10`.
    

**Konfigurace v CLI:**

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


### Konfigurace DHCP poolů na SRV01

Na serveru SRV01 budou vytvořeny DHCP pooly pro jednotlivé VLAN. Každý pool má nastavenou výchozí bránu, DNS server, startovní adresu a maximální počet uživatelů. Tímto způsobem získávají zařízení ve všech VLAN adresy dynamicky, zatímco servery a další klíčové prvky mají adresy statické.

| Pool Name                 | Default Gateway | DNS Server  | Start IP Address | Subnet Mask   | Max User |
| ------------------------- | --------------- | ----------- | ---------------- | ------------- | -------- |
| serverPool (Server-Admin) | 10.31.10.1      | 10.31.10.11 | 10.31.10.100     | 255.255.255.0 | 30       |
| Sales                     | 10.31.20.1      | 10.31.10.11 | 10.31.20.20      | 255.255.255.0 | 30       |
| Finance                   | 10.31.30.1      | 10.31.10.11 | 10.31.30.30      | 255.255.255.0 | 30       |
| Logistics                 | 10.31.40.1      | 10.31.10.11 | 10.31.40.40      | 255.255.255.0 | 30       |
| Management                | 10.31.50.1      | 10.31.10.11 | 10.31.50.50      | 255.255.255.0 | 30       |
| Warehouse                 | 10.31.60.1      | 10.31.10.11 | 10.31.60.60      | 255.255.255.0 | 30       |
| Guest                     | 10.31.70.1      | 10.31.10.11 | 10.31.70.70      | 255.255.255.0 | 30       |

![SRV01-DHCP](../images/Pasted%20image%2020250915180653.png)

### Postup konfigurace:

* Na serveru SRV01 otevřít **Services → DHCP**.
    
* Vybrat přednastavený `serverPool`, doplnit hodnoty a kliknout na **Save**.
    
* Poté vytvořit nové pooly pro ostatní VLAN a vyplnit odpovídající hodnoty podle tabulky.
    
* Konfiguraci uložit pomocí **Save**.
    

>**Pozn.:** Pool `serverPool` v Packet Traceru nelze odstranit ani přejmenovat. Aby DHCP fungovalo, musí být tento pool vyplněný, i když VLAN 10 používá statické adresy. Další pooly je již možné pojmenovat podle VLAN nebo oddělení.


### Závěr k DHCP

Pro správné fungování DHCP je nutné na routerech R2 a R3 nastavit **ip helper-address**, aby se požadavky z VLAN přeposílaly na server SRV01. Díky tomu funguje přidělování adres pro všechny klientské VLAN 20 až 70. Do DHCP se úspěšně zapojuje i **laptop připojený přes WiFi k AP ve VLAN 10**, který získává adresu z výchozího serverPoolu. 

**Pingy mezi jednotlivými VLAN jsou úspěšné**, což potvrzuje správnou funkčnost DHCP a směrování; *detailní diagnostika je uvedena v kapitole 8

>**pozn.:** Kvůli omezením simulátoru Packet Tracer se po restartu mohou přidělené adresy měnit, protože server si nepamatuje **záznamy o přidělených adresách**. Proto v dokumentaci uvádíme pouze nastavené **rozsahy**, nikoli konkrétní IP adresy. Z tohoto důvodu je v Packet Traceru i ve větších projektech výhodnější provádět DHCP přímo **v CLI routeru**, kde se konfigurace chová lépe realisticky, takový postup budeme demonstrovat v příštím projektu.


## 5.3 - DNS

**Domain Name System (DNS)** zajišťuje překlad jmenných adres na číselné IP adresy. Umožňuje, aby uživatelé nemuseli zadávat IP adresy serverů, ale používali čitelné názvy. V rámci tohoto projektu je DNS služba provozována na **serveru SRV02** (10.31.10.11) ve VLAN 10. V našem případě jde o demonstraci připojení na interní firemní web, proto je nutné nejprve vytvořit odpovídající DNS záznam.

>**Pozn.:** V tomto projektu nesimulujeme připojení k veřejnému internetu za ISP. Zaměřujeme se na interní služby firmy. Náskedná konfigurace DNS probíhá pouze přes GUI serveru pro základní demonstraci funkce. Podrobnější konfiguraci a použití DNS přes CLI na routeru ukážeme až v příštím projektu.


###  SRV02

* Na serveru SRV02 byla aktivována služba DNS a vytvořen záznam pro interní webový server:

| Record Name                                | IP Address  | Description           |
| ------------------------------------------ | ----------- | --------------------- |
| [www.firma.local](http://www.firma.local/) | 10.31.10.11 | Interní webový server |

### Postup konfigurace:

1. Na serveru SRV02 otevřít **Services → DNS**.
    
2. Zapnout službu pomocí **On**.
    
3. Přidat nový záznam:
    
    - **Name:** `www.firma.local`
        
    - **Address:** `10.31.10.11`
        
    - **Type:** A (host record)
        
4. Konfiguraci přidáme pomocí **Add** a pro uložíme `Save`.
    

![DNS](../images/Pasted%20image%2020250915201540.png)

### Ověření funkčnosti

* Na PC7 ve VLAN 50 (Management) otevřít příkazový řádek a zadat:
    

```
ping www.firma.local
```

* Odpověď přišla z adresy **10.31.10.11**, což potvrzuje správný překlad jména na IP adresu a funkčnost služby DNS.
    

![ping-DNS](../images/Pasted%20image%2020250915201819.png)

### Závěr

Byl vytvořen DNS záznam `www.firma.local` směřující na interní webový server SRV02. Test na PC7 z VLAN 50 potvrdil funkčnost – jmenná adresa byla přeložena na IP a odpověď byla úspěšná. V další části se zaměříme na konfiguraci a ověření služby **HTTP**.


## 5.4 - HTTP


Služba **HTTP (Hypertext Transfer Protocol)** je základním protokolem pro přenos webových stránek. V rámci projektu slouží k demonstraci přístupu na **firemní web**, který je umístěný na serveru SRV02 ve VLAN 10 (Admin-Server). Přístup na web je řešen interně, bez připojení k veřejnému internetu. DNS zajistí překlad jména `www.firma.local` a HTTP server následně poskytuje samotný webový obsah.

V reálné praxi, pokud je web i databáze umístěna uvnitř firmy, funguje i při výpadku internetu. Pokud ale závisí na externím cloudu, stránka se načte, ale funkce nebudou dostupné. Tento projekt ukazuje interní scénář dostupný i bez internetu.

### Konfigurace HTTP serveru na SRV02

* Na serveru SRV02 otevřít záložku **Services -> HTTP**.
    
* Přepnout volbu **HTTP Service** na **On**.
    
* Volitelně lze upravit obsah webové stránky v poli **Index.html** (ponecháváme výchozí ukázkový obsah Packet Traceru).
    

![HTTP](../images/Pasted%20image%2020250915202509.png)

### Ověření funkčnosti

* Na PC4 ve VLAN 30 otevřít webový prohlížeč (Desktop -> Web Browser).
    
* Do adresního řádku zadat `www.firma.local`.
    
* Stránka se načte z HTTP serveru SRV02, čímž je potvrzena funkčnost DNS i HTTP služby.
    

![HTTP-web](../images/Pasted%20image%2020250915202625.png)

### Závěr

Byl zprovozněn vestavěný **HTTP server** na SRV02 a úspěšně otestováno připojení z klientské stanice ve VLAN 30 pomocí adresy `www.firma.local`. Tento krok doplňuje předchozí konfiguraci DNS a dokazuje funkčnost firemního webu v interní síti. V další části se zaměříme na službu **SYSLOG**.


## 5.5 - Syslog


**Syslog** je služba určená pro sběr a centralizovaný monitoring systémových zpráv a událostí v síti. Pomáhá administrátorům sledovat provoz, identifikovat chyby a reagovat na bezpečnostní incidenty. V našem projektu slouží k demonstraci ukládání logů z aktivních zařízení na serveru SRV01.

>**Pozn:** V Packet Traceru je Syslog v GUI na server zařízení zjednodušený a zobrazuje pouze omezené události. Podrobnější konfiguraci Syslogu přímo v CLI routerů (s využitím timestampů, bufferu a rozšířených logů) ukážeme až v rozsáhlejším příštím projektu, kde bude možné realisticky předvést širší možnosti této služby.

### Konfigurace Syslog serveru (SRV01)

* Na serveru SRV01 otevřít záložku **Services -> Syslog**.
    
* Přepnout volbu **Syslog Service** na **On**.
    
* Server nyní přijímá logy od ostatních zařízení v síti.
    

![Syslog-service](../images/Pasted%20image%2020250915203832.png)

### Nastavení síťových zařízení

Ukázka konfigurace na routeru R2:

```
R2> enable
R2# configure terminal
R2(config)# logging 10.31.10.10
R2(config)# end
R2# write memory
```
![logging-Syslog](../images/Pasted%20image%2020250915204147.png)

Stejné příkazy se použijí i na ostatních routerech a switchích, aby všechny posílaly své logy na SRV01, ale pro ukázku zde zobrazujeme CLI výstup pouze z R2.

### Generování logu

Pro demonstraci funkce Syslogu byla na routeru R1 provedena změna názvu zařízení:

```
R1(config)#hostname R1-Test
```
![syslog-test](../images/Pasted%20image%2020250915212711.png)

Tato změna se okamžitě projevila v CLI a měla by být zaznamenána v Syslogu na SRV01. 

### Ověření funkčnosti

* Na SRV01 otevřít záložku **Services -> Syslog**.
    
* V logovacím okně se zobrazí zprávy o změně konfigurace na R1 
    
![syslog-check](../images/Pasted%20image%2020250915212648.png)
### Závěr

Byl zprovozněn **Syslog server** na SRV01 a nastavena síťová zařízení k odesílání logů. Testovací akcí bylo ověřeno, že konfigurace je správně zaznamenána a dostupná pro administrátora na SRV01


## 5.6 - NTP


**NTP (Network Time Protocol)** zajišťuje, že všechna síťová zařízení pracují se stejným časem. Přesný čas je důležitý pro logování událostí, bezpečnostní protokoly, autentizaci a diagnostiku. 
V naší síti bude router **R1** nastaven jako hlavní časový server a ostatní routery si budou čas synchronizovat z něj.

>**Pozn.:**  V praxi se NTP nastavuje i na switchích, zde je pro zjednodušení demonstrováno pouze na routerech.

### Konfigurace časového serveru R1

* Nastavit aktuální čas a datum.
    
* Definovat časové pásmo.
    
* Aktivovat NTP server se stratum 1.
    

**Konfigurace v CLI:**

```
R1> enable
R1# clock set 12:00:00 Jul 05 2025
R1(config)# configure terminal clock timezone CEST 2
R1(config)# ntp master 1
exit
write memory
```
![NTP-R1](../images/Pasted%20image%2020250915233646.png)

>**Pozn.:** Router **R1** byl nastaven jako **NTP master** se stratum 1, což z něj dělá hlavní časový zdroj pro celou síť. Všechna ostatní zařízení se synchronizují právě podle něj.

### Konfigurace klienta R2

* Nastavit časové pásmo.
    
* Definovat R1 jako NTP server.
    

**Konfigurace v CLI:**

```
R2> enable
R2(config)# clock timezone CEST 2
R2(config)# ntp server 10.31.0.1
exit
write memory
```
![NTP-R2](../images/Pasted%20image%2020250915234912.png)

### Konfigurace klienta R3

* Nastavit časové pásmo.
    
* Definovat R1 jako NTP server.
    

**Konfigurace v CLI:**

```
R3> enable
R3(config)# clock timezone CEST 2
R3(config)# ntp server 10.31.0.1
exit
write memory
```
![NTP-R3](../images/Pasted%20image%2020250915235024.png)

### Ověření funkčnosti

* Na všech routerech lze ověřit čas a zdroj synchronizace.
    
* Pro ukázku ukážeme ověření jen z R3

**Konfigurace v CLI:**

```
show clock detail
show ntp status
```
![NTP-diagnostic](../images/Pasted%20image%2020250915235523.png)

### Závěr

Router **R1** byl nakonfigurován jako hlavní časový server pomocí příkazu `ntp master 1`. Routery **R2** a **R3** byly nastaveny jako klienti s odkazem na server R1 (`ntp server 10.31.0.1`). Následná diagnostika ukázala, že zdrojem času na R3 je skutečně **NTP**, a stav `Clock is synchronized` potvrzuje úspěšnou synchronizaci. 

Tím je zajištěno, že všechna síťová zařízení používají jednotný čas, což je klíčové pro správné logování událostí, bezpečnostní protokoly i diagnostiku sítě.



## 5.7 - Wi-Fi

V tomto projektu Wi-Fi slouží pouze **adminovi na jeho notebooku**. Access Point (AP) je umístěn ve **VLAN 10 (Management)** a poskytuje SSID **„wifi-admin“** s heslem **Logistic123**. Síť je zabezpečena pomocí **WPA2-PSK** a admin má díky tomu plný přístup do celé sítě, stejně jako PC ve VLAN 10.

Z důvodu omezení Packet Traceru zde nedemonstrujeme zaměstnaneckou Wi-Fi oddělenou do samostatné VLAN s **omezeným přístupem**. Tento projekt ukazuje jen základní implementaci AP pro administraci. Plnohodnotná Wi-Fi infrastruktura pro zaměstnance (samostatná VLAN, ACL a omezený přístup jen na internet a vybrané služby) bude řešena až v navazujícím projektu.

#### Access Point

![AC-wifi](../images/Pasted%20image%2020250922231501.png)

#### Laptop-admin

![laptop-wifi](../images/Pasted%20image%2020250922231440.png)


## 5.8 - Shrnutí

V kapitole byly nakonfigurovány klíčové interní služby ve VLAN 10: DHCP a Syslog na SRV01, DNS a HTTP na SRV02. DHCP je dostupné pro všechny VLAN přes `ip helper-address`. DNS překládá `www.firma.local` na interní web SRV02, který je funkční díky HTTP službě. Syslog centralizuje logy na SRV01 a NTP zajišťuje jednotný čas v celé síti, s R1 jako masterem. 

---

**Pokračovat na další kapitolu:** Zabezpečení sítě



























