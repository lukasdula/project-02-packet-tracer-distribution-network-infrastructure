
# **7 - Testování konektivity**


## 7.1 - Úvod

V této kapitole se **testuje konektivita a správná funkce sítě**. Kontroluje se postupně, že všechna zařízení mají přístup podle návrhu, služby běží a bezpečnostní pravidla fungují. Díky testům je jasně vidět, že síť je **provozuschopná**. 

**Na začátku je přehledná tabulka testů pro lepší orientaci a na ní navazují jednotlivé ukázky výstupů**


## 7.2 - Přehled testů/diagnostiky

| Test | Funkce                     | Popis testu                                       | Příkaz / Akce                                                                           | Diagnostické výstupy                                     |
| ---- | -------------------------- | ------------------------------------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| 7.2  | Páteřní spoje mezi routery | Ověření propojení mezi routery a ISP              | `ping` na IP rozhraní                                                                   | `ping`, `show ip interface brief`                        |
| 7.3  | Směrování (OSPF)           | Kontrola sousedství a tras                        | `show ip ospf neighbor`, `show ip route ospf`                                           | výpis sousedů a tras                                     |
| 7.4  | VLANy                      | Test pingů na gateway z PC + kontrola konfigurace | `ping` na gateway IP, `show vlan brief`, `show interfaces trunk`, `show running-config` | výsledky pingů, tabulky VLAN a trunků                    |
| 7.5  | DHCP                       | Ověření, že PC získávají adresu automaticky       | GUI → Desktop → IP Config, `show running-config` (routery – `ip helper-address`)        | výpis přidělené adresy z DHCP, konfigurace helper adres  |
| 7.6  | DNS + HTTP                 | Překlad jména a přístup na web                    | `ping www.firma.local`, Web Browser → [www.firma.local](http://www.firma.local/)        | odpověď z 10.31.10.11, načtení stránky                   |
| 7.7  | NTP                        | Ověření synchronizace času                        | `show clock`, `show ntp status`                                                         | aktuální čas a stav synchronizace                        |
| 7.8  | SSH+VLAN99                 | Přihlášení na zařízení pomocí SSH                 | `ssh -l distadmin <IP>`, `show ip ssh`, `show ssh`, `ping` na VLAN99                    | výpis stavu služby, aktivní relace, potvrzení přihlášení |
| 7.9  | ACL pravidla               | Test povolených a zakázaných přístupů mezi VLAN   | `ping` mezi zařízeními                                                                  | povolené/zakázané výsledky                               |



## 7.2 - Páteřní spoje mezi routery

V této části se testují základní propojení mezi routery a směrem k ISP. Cílem je ověřit, že každé rozhraní na páteřních linkách funguje správně a že routery mají mezi sebou přímou konektivitu. Tento krok je nutný pro správnou funkci směrování a navazujících služeb.

### Postup testování

1. Ověří se stav ip adres na portech všech routerech pomocí příkazu `show ip interface brief`.
    
2. Provede se ping mezi sousedními routery (ISP<->R1, R1<->R2, R2<->R3).
    
3. Úspěšný ping potvrzuje, že propojení funguje a adresy jsou správně nastavené.
    
---

#### Použité příkazy v CLI:

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

### Závěr

Páteřní spoje mezi ISP, R1, R2 a R3 fungují bezchybně. Všechny rozhraní jsou ve stavu **up/up** a pingy mezi sousedními routery potvrzují plnou konektivitu. Tento krok tvoří základ pro následné směrování pomocí OSPF a dalším funkcím.


## 7.3 - Směrování (OSPF)

V této části se testuje funkčnost dynamického směrování pomocí protokolu **OSPF**. Cílem je ověřit, že routery mezi sebou navazují sousedství, sdílejí směrovací informace a že směrovací tabulky obsahují všechny potřebné trasy. Správná činnost OSPF je klíčová pro propojení celé sítě.

### Postup testování

1. Ověří se navázání sousedství mezi routery pomocí příkazu `show ip ospf neighbor`.
    
2. Zkontroluje se obsah směrovacích tabulek pomocí `show ip route ospf`.
    
3. Provede se několik pingů přes více skoků, aby se potvrdila konektivita napříč topologií.
    
---

#### Použité příkazy v CLI:

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
![OSPF-TEST-2](Pasted%20image%2020250922210728.png)

**Router R3:**

```
show ip ospf neighbor
show ip route ospf
ping 100.64.5.1
```
![OSPF-test-3](../images/Pasted%20image%2020250922210857.png)


>**Pozn.:** Hodnota zobrazená ve sloupci _Neighbor ID_ odpovídá **Router ID** souseda, ne jeho adrese na páteřním rozhraní. Pokud není Router ID nastaveno ručně, OSPF vybere **nejvyšší IP adresu z aktivních rozhraní** (například z management VLAN). Proto se může stát, že v přehledu sousedů vidíme adresy z VLAN 99 místo adresy páteřní linky.

### Závěr

OSPF směrování funguje správně. Routery mezi sebou navazují sousedství, vyměňují směrovací informace a směrovací tabulky obsahují všechny potřebné záznamy. Úspěšné pingy mezi R3 a ISP potvrzují, že komunikace probíhá i přes více skoků a směrování je plně funkční.


## 7.4 - VLANy

V této části se testuje funkčnost jednotlivých **VLAN** a jejich propojení přes trunkové porty. Cílem je ověřit, že koncová zařízení ve VLANách dosáhnou na svou gateway, konfigurace na switchích je správná a trunky přenášejí potřebný provoz.

### Postup testování


1. Ověří se konektivita mezi vybranými VLANami pomocí pingů:
    
    - `ping` z PC7 ve VLAN 50 (Management) na gateway ve VLAN 20 (Sales),
        
    - `ping` z PC5 ve VLAN 40 (Logistic) na gateway ve VLAN 60 (Warehouse),
        
    - `ping` z Admin PC1 ve VLAN 10 na gateway v Guest VLAN 70.
        
2. Na switchích se zobrazí konfigurace VLAN pomocí `show vlan brief`.
    
3. Ověří se stav trunkových portů příkazem `show interfaces trunk`.
    
4. Kontrola konfigurace se doplní příkazem `show running-config`, **ale kvůli délce výpisu vynecháme ukázku ze CLI**.
    
---

### Použité příkazy

**1. Ping mezi VLANy:**

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

**2. Switch (příklad SW1):**

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

### Závěr

Pingy na gateway ve všech VLANách i mezi vybranými VLANami proběhly úspěšně. Výpisy ze switchů potvrzují, že VLANy jsou správně vytvořené a přiřazené k portům. Trunkové porty mezi switchi a routerem přenášejí označkovaný provoz bezchybně. Konfigurace subrozhraní na routeru  R2 odpovídá nastaveným VLANám.


## 7.5 - DHCP

V této části se testuje funkčnost služby **DHCP** v celé síti. Cílem je ověřit, že klienti ve všech VLANách získají adresy automaticky ze serveru a že přidělené hodnoty odpovídají nastaveným poolům.

### Postup testování

1. Na PC v každé VLAN se nastaví získávání adresy přes DHCP (Desktop → IP Configuration → DHCP).
    
2. Na routerech se zkontroluje konfigurace `ip helper-address` v příkazu `show running-config`. Pro ukázku uvedeme pouze R2
    
3. Provede se test konektivity:
    
    - `ping` z PC1 (PC1-admin) na PC3 (Finance)
        
    - `ping` z PC8 (Management) na PC2 (Sales)
         
    * `ping` z PC10 (Warehouse) na PC4 (Logistics)

---

### Použité příkazy

**1. **PC (příklad PC2 / PC5 / PC9):**

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

**3. Test konektivity:**

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

### Závěr

Klienti ve všech VLANách obdrželi správné adresy ze serveru 10.31.10.10. Konfigurace `ip helper-address` na routerech je funkční. Testy ping mezi PC1 a PC3, PC10 a PC4 a také mezi PC8 a PC2, ověřil konektivitu po přidělení adresy.

> **Pozn.:** Aktivní ACL již v této fázi blokují některé pingy mezi vybranými VLAN podle nastavené bezpečnostní politiky..


## 7.6 - DNS a HTTP

#### Úvod

V této části se ověřuje funkčnost služeb DNS a HTTP v síti. DNS překládá doménová jména na IP adresy a HTTP poskytuje webový obsah klientům. Diagnostika ukazuje, zda klienti mohou přistupovat na webový server prostřednictvím doménového jména.

#### Postup ověření

1. **Ověření DNS překladu**
    
    - Na PC3 a PC6 se v příkazovém řádku spustí příkaz:  
        `ping www.firma.local`
        
    - Ověří se, že doménové jméno je správně přeloženo na IP adresu serveru `10.31.10.10` a že odpovědi přicházejí zpět.
        
2. **Ověření HTTP přístupu**
    
    - Na PC5 a PC9 se v internetovém prohlížeči zadá adresa:  
        `http://www.firma.local`
        
    - Zobrazí se výchozí webová stránka uložená na serveru (např. obsah `index.html`).
        
3. **Kontrola ACL**
    
    - ACL pravidla umožňují klientům z interních VLAN využívat DNS a HTTP služby serveru.
        
    - VLAN Guest je od těchto služeb oddělena a nemá přístup.
        

---

#### 1. Ověření DNS překladu

![DNS-PC3](../images/Pasted%20image%2020250923123627.png)

![DNS-PC6](../images/Pasted%20image%2020250923123709.png)

#### 2. Ověření HTTP přístupu

![HTTP-PC5](../images/Pasted%20image%2020250923123825.png)

![HTTP-PC9](../images/Pasted%20image%2020250923123906.png)

#### 3. Kontrola ACL

* Ověřujeme blokaci z PC11 ve VLAN70 na firemní local web

![HTTTP-ACL-PC11](../images/Pasted%20image%2020250923124111.png)

### Závěr

Ověření potvrdilo správnou funkci DNS překladu i HTTP služby. Vybrané klientské stanice byly schopny překládat doménové jméno na IP adresu a načíst webový obsah prostřednictvím prohlížeče. Politika ACL byla dodržena -> interní VLAN mají přístup, zatímco VLAN Guest zůstává izolována.

## 7.7 - NTP

Tady ověřujeme funkčnost služby NTP (Network Time Protocol) v síti. NTP slouží k synchronizaci času na všech síťových prvcích, což je zásadní pro správné fungování protokolů, logování a bezpečnostních mechanismů.

#### Postup ověření

1. **Kontrola na R2**
    
    - Na routeru R2 se spustí příkaz:  
        `show clock`
        
        
    - Pro detailní ověření se použije příkaz:  
        `show ntp status`
        
2. **Kontrola na R3**
    
    - Na routeru R3 se spustí příkaz:  
        `show clock`
        
        
    - Pro diagnostiku NTP se využije také:  
        `show ntp associations`
        
---

#### 1. Kontrola na R2

```
show clock
show ntp status
```
![NTP-R2-TEST](../images/Pasted%20image%2020250923124738.png)

#### 2. Kontrola na R3

```
show clock
show ntp associations
```
![NTP-R3-TEST](../images/Pasted%20image%2020250923125119.png)

### Závěr  

Ověření potvrdilo správnou funkci NTP služby. Routery R2 a R3 mají synchronizovaný čas s NTP serverem R1, což zajišťuje konzistenci logů a správné fungování časově závislých procesů.


# 7.8 - SSH a VLAN 99

V této části se ověřuje funkčnost zabezpečeného vzdáleného přístupu pomocí SSH a také správná konfigurace a dostupnost management VLAN 99. SSH zajišťuje šifrovanou správu síťových prvků, zatímco VLAN 99 slouží jako vyhrazená VLAN pro správu.

#### Postup ověření

1. **Ověření SSH připojení**
    
    - Na routeru R1 se navazuje SSH spojení na router R2:  
        `ssh -l distadmin 10.31.x.x`
        
    - Ověří se úspěšné přihlášení uživatele a zobrazení příkazového režimu na R2.
        
2. **Ověření VLAN 99**
    
    - Na routeru R3 se provede ping na adresu VLAN 99 na switchi SW2:  
        `ping 10.31.99.x`
        
    - Ověří se, že VLAN 99 je dosažitelná a správně nakonfigurovaná pro management.
        

---

#### 1. Ověření SSH připojení

```
ssh -l distadmin 10.31.0.2
```
![SSH-TEST](../images/Pasted%20image%2020250923130252.png)

#### 2. Ověření VLAN 99

* ping z R3 na  SW2

![VLAN99-TEST](../images/Pasted%20image%2020250923130444.png)

### Závěr

Diagnostika potvrdila funkční SSH přístup mezi routery R1 a R2, což umožňuje bezpečnou správu přes vzdálené připojení. Test VLAN 99 ukázal, že management VLAN je dostupná ze sítě, konkrétně z routeru R3 na switch SW2. To potvrzuje správné fungování management infrastruktury.


## 7.9 - ACL a pravidla

 
Oěřujeme funkčnost přístupových kontrolních seznamů (ACL) v síti. Testuje se komunikace mezi bránami, mezi koncovými zařízeními v jednotlivých VLAN a zároveň se ukazuje aplikace specifických pravidel, jako je omezený přístup VLAN Guest, povolený přístup k tiskárně nebo privilegovaný přístup Admin a Management VLAN.

#### Postup ověření

1. **Ping mezi bránami**
    
    - Ověří se vzájemná dostupnost mezi subrozhraními routerů R2 a R3 (např. 10.31.20.1 ↔ 10.31.60.1).
        
    - Pingy mezi bránami potvrzují správné směrování a propojení VLAN.
        
2. **Ping mezi koncovými zařízeními**
    
    - Z PC2 ve VLAN 20 (10.31.20.21, DHCP) se provede ping na PC4 ve VLAN 30 (10.31.30.30).
        
    - Ověří se, že přístup je blokován podle pravidel ACL.
        
    - Z PC5 ve VLAN 40 (10.31.40.41) se provede ping na PC10 ve VLAN 60 (10.31.60.61).
        
    - Ověří se, že přístup je povolen, protože ACL umožňuje komunikaci mezi Logistics a Warehouse.
        
3. **Demonstrace VLAN Guest**
    
    - Z PC11 ve VLAN 70 (10.31.70.70, DHCP) se provede ping na interní servery (10.31.10.10, 10.31.10.11).
        
    - Ověří se, že pingy selžou podle politiky ACL.
        
    - Následně se z PC11 provede ping na IP adresu ISP routeru (např. 100.64.5.1).
        
    - Ověří se, že ping je úspěšný, což demonstruje přístup VLAN Guest pouze k síti ISP, nikoli do interních VLAN.
        
4. **Ověření přístupu na tiskárnu**
    
    - Z PC6 ve VLAN 40 (10.31.40.40) a PC7 ve VLAN 50 (10.31.50.50) se provede ping na tiskárnu (10.31.20.10).
        
    - Ověří se, že komunikace je povolena podle ACL.
        
5. **Privilegovaný přístup Admin a Management VLAN**
    
    - Z Admin PC VLAN10 se provede ping na PC4 ve VLAN 30 (Finance, 10.31.30.30).
        
    - Ověří se, že přístup je povolen.
        
    - Z PC8 ve VLAN 50 (Management, 10.31.50.51) se provede ping na Finance (10.31.30.30).
        
    - Ověří se, že přístup je také povolen.
        
6. **Ověření komunikace se servery**
    
    - Z PC2 ve VLAN 20 (10.31.20.21) se provede ping na server 10.31.10.10.
        
    - Ověří se, že přístup je povolen podle politiky ACL.
        

--- 

### Provedené příkazy:


#### 1. Ping mezi bránami

* Ping z R2 na brány VLAN20 a VLAN60 

```
ping 10.31.20.1
ping 10.31.60.1
```
![ACL-testing1](../images/Pasted%20image%2020250923152643.png)


* Ping z R3 na brány VLAN20 a VLAN60

```
ping 10.31.20.1
ping 10.31.60.1
```
![ACL-testing2](../images/Pasted%20image%2020250923152925.png)


#### 2. Ping mezi koncovými zařízeními

* Ping z PC2 (Sales) na PC4 (Finance)

```
ping 10.31.30.30
```
![ACL-ping1](../images/Pasted%20image%2020250923153311.png)


* Ping z PC5 (Logistics) na PC10 (Warehouse)

```
ping 10.31.60.61
```
![ACL-ping2](../images/Pasted%20image%2020250923154236.png)


#### 3. Demonstrace VLAN Guest

* ping z PC11 na serevery SRV01 a SRV02

```
ping 10.31.10.10
ping 10.31.10.11
```
![ACL-ping3](../images/Pasted%20image%2020250923154607.png)


* Ping z PC11 směrem ven na internet k ISP

```
ping 100.64.5.1
```
![ACL-ping4](../images/Pasted%20image%2020250923155157.png)

#### 4. Ověření přístupu na tiskárnu

* ping z PC6 (Logistics) na tiskárnu ve VLAN20

```
ping 10.31.20.10
```
![ACL-ping5](../images/Pasted%20image%2020250923155728.png)

* ping z PC7 (Management) na tiskárnu ve VLAN20

```
ping 10.31.20.10
```
![ACL-ping6](../images/Pasted%20image%2020250923155915.png)

#### 5. Privilegovaný přístup Admin a Management VLAN


- Z Admin PC1 (Admin-Server) se provede ping na PC4 (Finance)

```
ping 10.31.30.30
```
![ACL-ping7](../images/Pasted%20image%2020250923160341.png)


- Z PC8 ve VLAN 50 (Management) se provede ping na (Finance).


```
ping 10.31.30.30
```
![ACL-ping8](../images/Pasted%20image%2020250923160512.png)


#### 6. Ověření komunikace se servery

* Z PC2 (Sales) se provede ping na server SRV01.

```
ping 10.31.10.10
```
![ACL-ping9](../images/Pasted%20image%2020250923160757.png)

### Závěr

Diagnostika potvrdila správnou aplikaci ACL pravidel. Bylo ověřeno, že VLAN Guest má přístup pouze k ISP a nikoli k interním zdrojům. Přístup k tiskárně funguje pro definované VLAN, zatímco Admin a Management VLAN mají privilegovaný přístup i do Finance. Servery jsou dostupné z interních VLAN podle politiky ACL, čímž je potvrzena správná implementace pravidel zabezpečení.


---
## Shrnutí

Kapitola 7 ověřuje funkčnost celé sítě od základního propojení až po bezpečnostní pravidla. Nejprve byly testovány spoje mezi routery a směrem k ISP, kde pingy i OSPF potvrdily správné směrování. Diagnostika VLAN ukázala funkční gateway i trunky a DHCP přidělovalo adresy dle poolů. DNS a HTTP poskytovaly jmenný překlad a webový obsah, zatímco ACL správně blokovalo VLAN Guest. Ověření NTP potvrdilo synchronizaci času na routerech.

Následně se testoval SSH přístup mezi routery a dostupnost management VLAN 99. Závěrečná diagnostika ACL potvrdila, že komunikace mezi VLAN odpovídá nastavené politice ,Guest má pouze přístup k ISP, Admin a Management VLAN mají privilegia k Finance a přístup k serverům i tiskárně je povolen dle pravidel. Kapitola tak potvrzuje stabilní a bezpečnou funkci celé topologie.


**Pokračovat na další kapitolu:** Troubleshooting



























