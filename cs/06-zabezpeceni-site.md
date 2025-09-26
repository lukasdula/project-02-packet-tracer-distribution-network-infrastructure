
# **6 - Zabezpečení sítě**


## 6.1 - Úvod

**Bezpečnost je základní součástí každé sítě.** I správně navržená topologie zůstává zranitelná, pokud není chráněna před neoprávněným přístupem. Proto tato kapitola zavádí opatření, která síť dělají spolehlivější a méně náchylnou k útokům.

Zavádí se **jednotný uživatelský účet se silným heslem**, ochrana privilegovaného režimu pomocí **enable secret** a šifrovaný vzdálený přístup přes **SSH**. Správa přepínačů probíhá odděleně v **management VLAN 99** a při přihlášení se zobrazují **varovné bannery (MOTD)**, které upozorňují na omezený přístup.

Na rozdíl od projektu 1 se zde **nevyužívá Port Security ani vypínání nevyužívaných portů**, protože přístup k síťovým zařízením mají pouze interní zaměstnanci a demonstrace těchto prvků není v tomto projektu účelem. Komunikaci mezi VLAN regulují **Access Control Lists (ACL)**. Doplňkově se řeší i **CDP (Cisco Discovery Protocol)**

## 6.2 - Uživatelský účet a heslo

**Základním prvkem zabezpečení je vytvoření vlastního uživatelského účtu se silným heslem** a ochrana privilegovaného režimu pomocí příkazu **enable secret**. Tím se nahrazuje původní jednoduchý vstup pouze s heslem a zajišťuje se, že přístup k síťovým prvkům mají pouze oprávněné osoby. **Enable secret** je automaticky šifrovaný a bezpečnější než klasické enable password.

**Konfigurujeme všechna zmíněná Zařízení:** R1, R2, R3, SW1, SW2

- vytvoření uživatelského účtu s heslem
    
- nastavení enable secret pro privilegovaný režim
    
* ukázka konfigurace v CLI je uvedena pouze na **R1** (stejný postup platí i pro další zařízení)

**Konfigurace v CLI:**

```
enable  
configure terminal  
username distadmin secret Logistic25  
enable secret Pallet25  
end  
write memory
```
![login-password](../images/Pasted%20image%2020250916184716.png)

### Závěr

Pomocí příkazu **username distadmin secret** je vytvořen uživatelský účet, který se používá pro přihlášení přes **konzoli i VTY linky (SSH)**. Příkaz **enable secret** chrání vstup do privilegovaného režimu. Oba prvky se doplňují a společně tvoří základní vrstvu zabezpečení přístupu k síťovým prvkům.

>_Poznámka:_ Pro účely projektu jsou zvolena jednodušší tematická hesla. V reálném nasazení by byla použita komplexnější a delší hesla (12+ znaků, kombinace velkých/malých písmen, číslic a speciálních znaků).


## 6.3 - Zabezpečení konzole

**Zabezpečení konzolového přístupu** je důležité proto, aby se k zařízení nemohl dostat kdokoliv cizí, kdo má fyzický přístup ke konzolovému portu. Konzole se proto nastavuje tak, aby při přihlášení vyžadovala **uživatelské jméno a heslo** z lokální databáze. Tím se využije účet vytvořený v předchozí podkapitole. Navíc lze nastavit časový limit, po jehož uplynutí bude neaktivní relace automaticky ukončena.

**Zařízení:** R1, R2, R3, SW1, SW2

- nastavení konzolového přístupu tak, aby vyžadoval přihlášení pomocí lokálního účtu
    
- využití účtu `distadmin / Logistic25` vytvořeného dříve
    
- volitelně nastavení časového limitu nečinnosti (exec-timeout)
    
- ukázka konfigurace v CLI je uvedena pouze na **R1** (stejný postup platí i pro další zařízení)
    

**Konfigurace v CLI:**

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

#### Závěr:  

Po aktivaci příkazu **login local** se při připojení přes konzoli vyžaduje zadání uživatelského jména a hesla z lokální databáze. To zajišťuje, že přístup mají pouze oprávněné osoby. Časový limit pomocí **exec-timeout** dále zvyšuje bezpečnost tím, že ukončí zapomenuté nebo neaktivní relace.


## 6.4 - SSH přístup (VTY linky)

**Zabezpečený vzdálený přístup** k síťovým prvkům se provádí pomocí protokolu **SSH**, který nahrazuje nezabezpečený Telnet. SSH šifruje komunikaci a chrání tak přihlašovací údaje i správu zařízení před odposlechem. Přístup na VTY linkách je proto omezen pouze na SSH a využívá uživatelský účet vytvořený v předchozích krocích.

Nastavení příkazu **ip domain-name** slouží pro generování RSA klíčů a pro vytvoření plného názvu zařízení (FQDN). Doménové jméno je stejné na všech zařízeních, například `firma.local`. Díky tomu má každé zařízení svůj unikátní název ve tvaru `hostname.firma.local`.

**Zařízení:** R1, R2, R3, SW1, SW2

- nastavení DNS domény pro generování RSA klíčů
    
- vygenerování RSA klíčů pro šifrování SSH komunikace
    
- povolení SSH verze 2
    
- konfigurace VTY linek pro přihlášení pomocí lokální databáze uživatelů
    
- omezení přístupu pouze na SSH (žádný Telnet)
    
- nastavení časového limitu nečinnosti (exec-timeout)
    
- ukázka konfigurace v CLI je uvedena pouze na **R1** (stejný postup výše platí i pro další zařízení)
    

**Konfigurace v CLI:**

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


#### Ověření SSH přístupu:

- na R2 se spustí příkaz `ssh -l distadmin 10.31.1.2`
    
- po výzvě se zadá heslo **Logistic25**
    
- po přihlášení se zařízení nachází v user módu `>`
    
- pro vstup do privilegovaného módu se zadá `enable`
    
- po výzvě se zadá heslo **Pallet25**
    
- zařízení přejde do privilegovaného módu `#`
    
* uživatel na **R2** je přihlášen do **R3** a všechny příkazy se provádějí na tomto zařízení vzdáleně


![SSH-login](../images/Pasted%20image%2020250916212006.png)


#### Diagnostika:

- `show ip ssh` – zobrazuje stav SSH na zařízení
    
- `show ssh` – zobrazuje aktivní SSH relace
    

![SSH-diagnostic](../images/Pasted%20image%2020250916212345.png)

#### Závěr:  

Po nastavení je vzdálený přístup možný pouze přes **SSH**. Zařízení vyžaduje přihlášení pomocí uživatelského účtu (`distadmin / Logistic25`) a následně heslo k privilegovanému režimu (**Pallet25**). Díky šifrování SSH jsou přihlašovací údaje chráněny a relace je bezpečná i při vzdáleném přístupu.


## 6.5 - MOTD banner

**Varovný banner (MOTD - Message of the Day)** slouží k tomu, aby při přihlášení do zařízení zobrazil uživateli bezpečnostní upozornění. Banner **neblokuje přístup**, ale informuje, že zařízení je určeno pouze pro autorizované osoby. V praxi jde o běžný prvek firemní bezpečnostní politiky.

**Zařízení:** R1, R2, R3, SW1, SW2
### Postup

- vstoupí se do globální konfigurace
    
- nastaví se MOTD banner s varovným textem
    
- text se uzavře vybraným oddělovačem (např. `#`)
    
- ukázka konfigurace v CLI je uvedena pouze na **R1** (stejný postup výše platí i pro další zařízení)
    

**Konfigurace v CLI**:

```
enable
configure terminal
banner motd # Unauthorized access to Logistic Network is prohibited! #
end
write memory
```
![MOTD-BANNER-R1](../images/Pasted%20image%2020250916213457.png)

### Ověření

- po odhlášení (`exit`) a novém přihlášení do zařízení se varovný banner zobrazí ještě před výzvou k zadání hesla
    

**Příklad výstupu:**

```
Unauthorized access to Logistic Network is prohibited!
R1>
```
![MOTD-message](../images/Pasted%20image%2020250916213636.png)

### Závěr

MOTD banner zvyšuje bezpečnost tím, že **upozorňuje na neoprávněný přístup** a jasně deklaruje, že síťové prostředky patří organizaci. V rámci projektové dokumentace je banner považován za standardní součást firemního zabezpečení.


## 6.6 - Management VLAN 99

### Úvod

**Management VLAN** slouží k oddělení správy síťových prvků od běžného uživatelského provozu. Díky tomu je zajištěno, že přístup k zařízení (například přes SSH) probíhá v samostatné síti, což zvyšuje bezpečnost i přehlednost. V tomto projektu je pro správu vyhrazena **VLAN 99**.

Bez vytvoření a nastavení management VLAN na přepínači není možné zařízení na dálku spravovat, switch by nebylo možné pingnout ani se na něj připojit přes SSH. Takže je nutné nakonfigurovat routery a SWITCH (například VLAN 99) s vyhrazenou IP adresou pro admina. 

**Zařízení:** R1, R2, R3, SW1, SW2

### Postup

- vytvořit VLAN 99 na přepínačích
    
- přiřadit jí IP adresu na SVI (Switch Virtual Interface) pro možnost správy
    
- nastavit trunk porty, aby přenášely VLAN 99
    
- ověřit funkčnost pomocí ping a SSH
    

**Konfigurace v CLI (SW1)**

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

**Konfigurace v CLI (SW2)**

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

**Konfigurace v CLI (R2)**

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

**Konfigurace v CLI (R3)**

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

### Ověření

- z R3 provést `ping 10.31.99.2` pro ověření konektivity na SW1  
      
- z R3 provést `ssh -l distadmin 10.31.99.2` a přihlásit se do SW1 přes účet `distadmin / Logistic25`  
      
- na SW2 provést příkaz `show vlan brief` ověřit existenci a stav VLAN 99  

![VLAN99-test](../images/Pasted%20image%2020250916230150.png)


### Závěr

Správa přepínačů probíhá odděleně ve vyhrazené **VLAN 99**, což zvyšuje bezpečnost a přehlednost infrastruktury. VLAN 99 je přenášena trunky mezi zařízeními a umožňuje vzdálený šifrovaný přístup přes SSH pouze v rámci SERVICE-ADMIN sítě.

## 6.7 - Access Control Lists (ACL)


**Access Control Lists (ACL)** slouží k **řízení provozu mezi VLAN** a k **ochraně citlivých částí sítě**. Umožňují nastavit, kdo má přístup na servery, kdo do jiných oddělení a kdo jen ven do internetu.

Cílem je vytvořit **bezpečnostní politiku**, která odpovídá potřebám firmy a ukazuje praktické využití ACL v podnikové síti.

#### Cíle ACL

- **Servery** mají být dostupné pro všechny interní VLAN kromě Guest.
    
- **Admin PC** a **Management VLAN** mají plný přístup do všech vlanů.
    
- **Finance VLAN** je striktně chráněná – přístup jen na servery,tiskárnu.
    
- **Sales, Logistics a Warehouse** mají omezený, ale funkční přístup (servery, tiskárna, vzájemná komunikace).
    
- **Guest VLAN** je izolována od interní sítě a směřuje pouze k ISP.


### Postup konfigurace ACL – Router R2 (VLAN 20, 30, 40, 50)

1. **Povolit DHCP**  
    Na začátek každého ACL vložit pravidlo:  
    `permit udp any eq 68 any eq 67`
    
2. **Nastavit ACL pro každou VLAN zvlášť**  
    Vytvořit ACL pro VLAN 20, 30, 40 a 50.
    
3. **Povolit přístup na servery a tiskárnu**
    
    - Servery: `10.31.10.10` a `10.31.10.11`
        
    - Tiskárna: `10.31.20.10`
        
4. **Zajistit přístup pro Admin PC1**
    
    - Admin PC1 (`10.31.10.20`) má přístup do všech VLAN.
        
    - Ostatní VLAN mají na Admin PC1 výslovně zakázaný ping a přístup.
        
5. **Omezit komunikaci mezi VLAN**
    
    - Finance (VLAN 30) izolovat, povolit jen servery a tiskárnu.
        
    - Sales (VLAN 20) povolit servery a tiskárnu, zakázat Admin PC1.
        
    - Logistics (VLAN 40) povolit servery, tiskárnu a přístup k Warehouse (VLAN 60).
        
    - Management (VLAN 50) má volný přístup do všech VLAN.
        
6. **Povolit odchod do internetu**  
    Na konec každého ACL vložit:  
    `permit ip any`
    

**Konfigurace v CLI**:

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


> **Pozn.:** Aby DHCP fungovalo i při aplikovaných ACL, je nutné mít hned na začátku každé ACL řádek `permit udp any eq 68 any eq 67`. Klient při startu ještě nemá IP adresu a posílá první DHCP požadavek ze zdroje **0.0.0.0** na cíl **255.255.255.255**. Tento paket by jinak neodpovídal povoleným sítím (např. Sales 10.31.20.0/24, Warehouse 10.31.30.0/24) a ACL by ho zablokovala. Výjimka zajistí, že DHCP požadavky projdou k routeru a `ip helper-address` je předá serveru 10.31.10.10, zatímco ostatní pravidla zabezpečení zůstávají zachována.

---

### Postup konfigurace ACL – Router R3 (VLAN 60, 70)

1. **Povolit DHCP**  
    Na začátek každého ACL vložit kvůli DHCP pravidlo:  
    `permit udp any eq 68 any eq 67`
    
2. **Nastavit ACL pro každou VLAN zvlášť**  
    Vytvořit ACL pro VLAN 60 a 70.
    
3. **Povolit přístup na servery a tiskárnu**
    
    - Servery: `10.31.10.10` a `10.31.10.11`
        
    - Tiskárna: `10.31.20.10`
        
4. **Zajistit přístup pro Admin PC1**
    
    - Admin PC1 (`10.31.10.20`) má přístup do všech VLAN
        
    - Ostatní VLAN kromě management mají na Admin PC1 výslovně zakázaný ping.
        
5. **Omezit komunikaci mezi VLAN**
    
    - Warehouse (VLAN 60) povolit servery, tiskárnu a přístup k Logistics (VLAN 40).
        
    - Guest (VLAN 70) povolit pouze internet, žádný přístup do interní sítě.
        
6. **Povolit odchod do internetu**  
    Na konec každého ACL vložit:  
    `permit ip any`
    

**Konfigurace v CLI**:

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
### Aplikace ACL na rozhraní

  
Aby ACL pravidla skutečně fungovala, je nutné je přiřadit na příslušná subrozhraní routerů. V našem návrhu se ACL aplikují **inbound** (na příchozí provoz do routeru z jednotlivých VLAN). Tím se zkontroluje a případně zahodí provoz hned při vstupu na router.


#### Router R2

* Aplikovat ACL V20–V50 na subrozhraní Gi0/1.20, Gi0/1.30, Gi0/1.40, Gi0/1.50.
    
* Každé ACL je přiřazeno inbound pro provoz přicházející z dané VLAN.
    

**Konfigurace v CLI**:

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

* Aplikovat ACL V60 a V70 na subrozhraní Gi0/2.60 a Gi0/2.70.
    
* * Každé ACL je přiřazeno inbound pro provoz přicházející z dané VLAN.
    

**Konfigurace v CLI**:

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


### Přehledová tabulka ACL zvolených pravidel

| VLAN                   | Povolené přístupy (LAN)                                                 | Zakázané přístupy (LAN)                                     | Internet | Poznámka                                                                |
| ---------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------- | -------- | ----------------------------------------------------------------------- |
| **V20 - Sales**        | Servery **10.31.10.10/11**,<br>Logistics V40, Warehouse V60,            | Finance **V30**, Management **V50**, PC-admin               | ANO      | Ve **V20** jsou výjimky pro **odpovědi** z tiskárny do V30/V40/V50/V60. |
| **V30 - Finance**      | Servery **10.31.10.10/11**, Tiskárna **10.31.20.10**                    | Všechny interní VLAN                                        | ANO      | Finance izolované; jen servery a tiskárna.                              |
| **V40 - Logistics**    | Servery **10.31.10.10/11**, Tiskárna **10.31.20.10**, **Warehouse V60** | Finance **V30**, Sales V20 Management V50                   | ANO      | Povolen obousměrně směr **V40<->V60**. Řízení logistiky na dálku        |
| **V50 -Management**    | **Všechny interní VLAN 10.31.0.0/16**                                   | —                                                           | ANO      | Plný přístup (vč. servery, tiskárna).                                   |
| **V60 -Warehouse**     | Servery **10.31.10.10/11**, Tiskárna **10.31.20.10**, **Logistics V40** | Finance **V30**, Management V50, Sales V20                  | ANO      | Přístup do Logistics **V60<-> V40** povoleno.                           |
| **V70 - Guest**        | INTERNET                                                                | **Celá 10.31.0.0/16** a Internal switching and routing zone | ANO      | Hosté jen internet.<br>Ostatní vlany mají do guest přístup.             |
| **V10 -Servery/Admin** | Přístup do interní sítě                                                 | —                                                           | ANO      | Správa sítě                                                             |


>**Pozn.:** Interní VLANy mohou směrovat provoz směrem do Guest VLAN, ale veškerá komunikace je na straně Guest zablokována – Guest má povolený pouze přístup k internetu.

### Závěr

Aplikované ACL na routerech R2 a R3 přesně odrážejí navrženou politiku: Sales, Finance, Logistics a Warehouse mají přístup pouze na servery, tiskárnu a povolené partnerské VLAN, Management disponuje plným přístupem, zatímco Guest je omezen jen na internet. Pingy provedené v rámci diagnostiky potvrdily, že povolené směry komunikace fungují a zakázané jsou blokovány, což odpovídá nastavené tabulce politik ACL.


## 6.8 - Shrnutí 

Kapitola 6 ukazuje kompletní postup zabezpečení sítě: od vytvoření uživatelských účtů a hesel, přes ochranu konzole a vzdálený přístup přes SSH, nastavení MOTD bannerů a oddělenou management VLAN, až po detailní implementaci Access Control Lists. 

ACL byly nakonfigurovány na routerech R2 a R3, přiřazeny na příslušná rozhraní a otestovány pingy mezi VLANami. Testy potvrdily, že přístupová politika funguje, servery a tiskárna jsou dostupné podle pravidel, citlivé VLANy jsou chráněné, Management má plný přístup a Guest m8 přístup jen na internet. 

Port-security a administrativní vypínání portů jsme tentokrát vynechali, protože byla řešena už v prvním projektu small-cafe-network a v tomto typu firmy, kde mají do interní sítě přístup pouze zaměstnanci, nejsou tyto opatření tolik důležitá.

**Pokračovat na další kapitolu:** Diagnostika






































































































































