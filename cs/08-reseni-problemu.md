
# **8 - Troubleshooting**


## 8.1 - Úvod

**Troubleshooting** je důležitou součástí správy sítě. I když je konfigurace provedena správně, mohou se v infrastruktuře objevit **problémy**, které naruší **komunikaci** nebo **dostupnost služeb**. **Diagnostika** umožňuje rychle určit **zdroj potíží**, provést **opravu** a následně ověřit **funkčnost**. V této kapitole jsou popsány vybrané **situace**, které se mohou v síti vyskytnout a vyžadují provedení **systematického řešení**.

**Oblasti problémů:**

**8.2. Problém se směrováním mezi routery v rámci OSPF.**
    
**8.3. Chyba v nastavení VLAN.**
    
**8.4. Klienti v některé VLAN nezískávají adresu z DHCP serveru.**
    
**8.5. Přístup mezi VLANami neodpovídá pravidlům ACL.**
    

>**Pozn.:** Následující příklady představují modelové situace, které pomáhají procvičit postupy řešení a zlepšení dovedností při práci se sítí.


## 8.2 - Problém se směrováním OSPF

**Popis problému**  

Při testování konektivity mezi routery se zjistilo, že ping z R3 na ISP selhává. Routery R1 a R2 mají sousedství navázané správně, ale R3 nevytváří OSPF spojení s R2. To naznačuje problém v konfiguraci nebo stavu rozhraní.

---
#### Diagnostika

**1. Na R3 se ověří stav OSPF sousedů:**
    

```
R3#show ip ospf neighbor
```
![OSPF-diagnostic-R3](../images/Pasted%20image%2020250923182349.png)
Výsledek ukazuje, že R3 nemá žádné OSPF sousedy.

**2. Na R3 se zkontroluje stav rozhraní:**
    

```
R3#show ip interface brief
```
![OSPF-brief-R3](../images/Pasted%20image%2020250923182749.png)

Rozhraní g0/1 směrem na R2 má status **up**, ale protokol je **down**. To znamená, že rozhraní je zapnuté, ale protokol neběží, protože na druhé straně (R2) je port vypnutý.

**3. Na R2 se ověří stav rozhraní:**
    

```
R2#show ip interface brief
```
![R2-BRIEF-OSPF](../images/Pasted%20image%2020250923182737.png)

Rozhraní g0/2 směrem na R3 je ve stavu **administratively down**. To znamená, že bylo vypnuté příkazem `shutdown`.

---

#### Opravy

**1. Aktivace rozhraní g0/2 na R2:**
    

```
R2(config)# interface g0/2
R2(config-if)# no shutdown
```
![port-OSPF](../images/Pasted%20image%2020250923183713.png)

Po této opravě je port aktivní, ale OSPF sousedství stále nevzniká.


**2. Další diagnostika ukazuje, že OSPF stále nefunguje. Na R3 se proto zkontroluje konfigurace protokolu:
    

```
R3#show ip protocols
```
![OSPF-protocols](../images/Pasted%20image%2020250923183947.png)

Výstup ukazuje, že síť 10.31.1.0/30 byla omylem přiřazena do **area 1** místo do **area 0**.


**3. Oprava OSPF oblasti na R3:**
    

```
R3(config)#router ospf 1
R3(config-router)#no network 10.31.1.0 0.0.0.3 area 1
R3(config-router)#network 10.31.1.0 0.0.0.3 area 0
```
![OSPF-fix-R3](../images/Pasted%20image%2020250923184121.png)

---

#### Ověření

**1. Na R2 se ověří stav sousedů po opravách:**
    

```
R2#show ip ospf neighbor
```
![check-R2](../images/Pasted%20image%2020250923184258.png)

Výstup ukazuje sousedství s R1 i R3 ve stavu FULL.

**2. Na R3 se ověří, že se objevily OSPF trasy:**
    

```
R3# show ip route ospf
```
![OSPF-OK](../images/Pasted%20image%2020250923184430.png)


**3. Ověření ping směrem na ISP**

```
ping 100.64.5.1
```
![ping-ISP-OSPF](../images/Pasted%20image%2020250923184535.png)

Trasy z R2 a dál na ISP jsou nyní dostupné. Ping z R3 na ISP je úspěšný.

---

### Závěr  

Problém byl způsoben dvěma chybami: administrativně vypnutým portem na R2 a nesprávně nastavenou OSPF oblastí na R3. Po odstranění obou chyb byla obnovena plná konektivita a OSPF směrování funguje správně.


## 8.3 - Chyba v nastavení VLAN

**Popis problému**  

Z PC1 (Admin, VLAN 10) nelze pingovat žádné zařízení ve VLAN 20. Ping na výchozí bránu **10.31.20.1** i na cílové zařízení **10.31.20.20** **selhává**. Podezření je na chybu v inter‑VLAN routingu (brána pro VLAN 20 na R2).

---
#### Diagnostika

**1.  Z PC1 nelze pingovat do VLAN20**
      
```
ping 10.31.20.1
ping 10.31.20.20
```
![VLAN-diagnostic-error](../images/Pasted%20image%2020250923200052.png)


**2. Na SW1 se ověří, že VLAN 20 existuje a porty jsou přiřazené správně:**
    

```
SW1#show vlan brief
```
![VLAN-brief1](../images/Pasted%20image%2020250923200335.png)

VLAN 20 je vytvořena, porty vypadají v pořádku.

**3. Na SW1 se ověří trunk směrem k R2 (uplink Gi0/1) – že VLAN 20 je povolená:**
    

```
SW1#show interfaces trunk
```
![trunk-test](../images/Pasted%20image%2020250923200452.png)

Trunk běží a VLAN 20 je v seznamu povolených VLAN. Na switchi tedy chyba patrná není.

**4. Diagnostika pokračuje na R2 -> rychlá kontrola rozhraní:**
    

```
R2#show ip interface brief
```
![brief-vlan20](../images/Pasted%20image%2020250923200623.png)

Zobrazuje se subinterface **g0/1.20** s IP adresou **10.31.2.1/24** -> **vidíme chybu, nesprávná brána pro VLAN 20**.

---

## Opravy

**1. Oprava IP adresy brány pro VLAN 20 na R2:**
    

```
R2(config)#interface g0/1.20
R2(config-subif)#no ip address
R2(config-subif)#ip address 10.31.20.1 255.255.255.0
R2(config-subif)#end
R2# write memory
```
![fixed-vlan20](../images/Pasted%20image%2020250923200834.png)

---

#### Ověření

**1. Kontrola subinterface po opravě:**
    

```
R2#show ip interface brief
```
![fixed-brief](../images/Pasted%20image%2020250923200915.png)

`g0/1.20` má nyní **10.31.20.1/24** a je **up/up**.

**2. Testy z PC1 (Admin):**
    

```
ping 10.31.20.20
ping 10.31.20.1
```
![ping-vlan20-fixed](../images/Pasted%20image%2020250923201008.png)

Ping na **PC2 (10.31.20.20)** i na **gateway (10.31.20.1)** je úspěšný.

---

### Závěr

Problém způsobil chybně nastavená **výchozí brána** pro VLAN 20 na subrozhraní **R2 g0/1.20**. Switchová část (VLAN/trunky) byla v pořádku, chyba se projevila až na routeru. Po nápravě na správnou adresu výchozí brány **10.31.20.1/24** byla konektivita mezi VLANami obnovena a pingy procházejí.

## 8.4 - Problém s DHCP 

**Popis problému**  

Z administrátorského PC1 (VLAN 10) nelze pingovat koncová zařízení **PC9 (10.31.60.61)** a **PC10 (10.31.60.60)** ve **VLAN 60 (Warehouse)**. **Ping na bránu 10.31.60.1 funguje**. Podezření padá na relay (DHCP helper) pro VLAN 60 na R3.

---

#### Diagnostika

**1. Test z PC1 (Admin) pingy na PC9 / PC10 a gateway -> Warehouse:**
    

```
ping 10.31.60.60
ping 10.31.60.61
ping 10.31.60.1
```
![DHCP-test-vlan60](../images/Pasted%20image%2020250923213544.png)

První dva pingy **selhávají**, ping na **gateway 10.31.60.1** je **úspěšný**.

**2. Na SW2 se ověří L2 stav – klientské porty ve VLAN 60 a trunk směrem k R3 přenáší VLAN 60:**
    

```
SW2#show vlan brief
SW2#show interfaces trunk
```
![DHCP-brief-trunk](../images/Pasted%20image%2020250923213752.png)

VLAN 60 existuje, porty jsou přiřazené správně, trunk přenáší – L2 je v pořádku.

**3. Na R3 se ověří stav subrozhraní pro VLAN 60 **(g0/2.60)**:**
    

```
R3#show ip interface brief
```
![DHCP-R3-brief](../images/Pasted%20image%2020250923214129.png)

Rozhraní `g0/2.60` je **up/up** s IP **10.31.60.1/24**.

**4. Kontrola DHCP relay (ip helper‑address) na R3 pro VLAN 60:**
    

```
R3# show ip interface g0/2.60 | include Helper
```
![error-ip-helper](../images/Pasted%20image%2020250923214236.png)

Výstup ukazuje **Helper address is 10.31.10.100** – jedná se o **nesprávný** DHCP server (správně má být **10.31.10.10 – SRV01**). Proto klienti ve VLAN 60 po obnově nedostanou IP z DHCP.

---

#### Opravy

**1. Výměna helperu za správný na R3 (VLAN 60):**
    

```
R3(config)#interface g0/2.60
R3(config-subif)#no ip helper-address 10.31.10.100
R3(config-subif)#ip helper-address 10.31.10.10
R3(config-subif)#end
R3#write memory
```
![fix-ip-helper](../images/Pasted%20image%2020250923214430.png)

---

#### Ověření

*  Na klientu **PC9**ve VLAN 60 znovu povolit DHCP a obnovit IP a ověříme přidělení ip adres na:

```
ipconfig /release
ipconfig /renew
ipconfig
```
![ip-config-PC9](../images/Pasted%20image%2020250923214648.png)

* Na klientu **PC10** ve VLAN 60 znovu povolit DHCP a obnovit IP a ověříme přidělení ip adres na:

```
ipconfig /release
ipconfig /renew
ipconfig
```
![IP-config-PC10](../images/Pasted%20image%2020250923214745.png)

PC9/PC10 obdrží adresy **10.31.60.x/24**, **výchozí bránu 10.31.60.1** a DNS dle poolu.

* Test konektivity z PC1 (Admin):
    

```
ping 10.31.60.60
ping 10.31.60.61
ping 10.31.60.1
```
![final-ping-VLAN60](../images/Pasted%20image%2020250923215326.png)

Všechny pingy jsou **úspěšné**.

---

### Závěr  

Příčinou byla špatně nastavená adresa **ip helper‑address (10.31.10.100)** na subrozhraní **R3 g0/2.60**. L2 část (VLAN/trunky) byla správně; klienti ve VLAN 60 po obnově nedostávali IP z DHCP, a proto nebyli dostupní. Po opravě na **10.31.10.10 (SRV01)** se adresy z DHCP přidělují a konektivita funguje.

## 8.5 - Přístup mezi VLANami neodpovídá pravidlům ACL

**Popis problému**  

Po nasazení ACL se objevují dvě porušení naší zvolené bezpečnostní politiky:

- **Finance (VLAN 30)** mají **nežádoucí přístup do PC1 (Admin) VLAN 10** (na PC1 10.31.10.20).
    
- **Guest (VLAN 70)** má **přístup do interních sítí** (servery ve VLAN 10 a hosté v ostatních VLAN), ačkoliv Guest má mít pouze přístup na internet.
    

---

#### Diagnostika

**Ověření nechtěných přístupů z Finance a Guest:**

 * ping z PC4 (Finance) na PC1 (Admin)

```
ping 10.31.10.20
```
![ACL-error-1](../images/Pasted%20image%2020250924134235.png)

**Ověření nechtěných přístupů z Guest:**

* ping z PC11 na SRV02 a na PC1 (Admin)

```
ping 10.31.10.11 
ping 10.31.10.20 
```
![ACL-error-2](../images/Pasted%20image%2020250924134523.png)

**Pingy prochází ->>>> potvrzuje se porušení politiky!!!**

* **Kontrola, že jsou ACL přiřazené na správných rozhraních a směrech:**
    

```
R2#show ip interface g0/1.30 
```
![show-int-ACL-1](../images/Pasted%20image%2020250924134742.png)

```
R3#show ip interface g0/2.70 
```
![show-int-ACL-2](../images/Pasted%20image%2020250924135038.png)

Vidíme „**Inbound access list is V30/V70**“.

* **Zobrazení obsahu a zásahů (hitcount):**
    

```
R2# show access-lists V30
```
![ACL-access-lists-V30](../images/Pasted%20image%2020250924135204.png)


```
R3#show access-lists V70
```
![ACL-show-access-lists-V70](../images/Pasted%20image%2020250924135349.png)


#### Identifikované špatné řádky:

- V **V30**: `permit ip 10.31.30.0 0.0.0.255 10.31.10.0 0.0.0.255` (široké povolení z Finance do Admin)
    
    - **Proč je to chyba:** obchází politiku „Finance **nesmí** mít obecný přístup do Admin VLAN“. Má být povolen pouze omezený provoz (konkrétní služby/servery) dle zbytku ACL.
        
- V **V70**: `permit ip 10.31.70.0 0.0.0.255 any` je umístěn **nad** `deny ip 10.31.70.0 0.0.0.255 10.31.0.0 0.0.255.255` **což je špatně zvolené pořadí**
    
    - **Proč je to chyba:** první vyhovující ACE (řádek) se uplatní -> globální permit povolí vše, deny do interní sítě se nikdy neuplatní. Naše ACL politika přitom říká „Guest **nesmí** nikam dovnitř pouze na internet“.
        

---

#### Opravy 

>Pozn.: Vybrané skupiny ACL lze bezpečně přepsat/opravit. Pro konzistentní pořadí dočasně ACL odpojíme z rozhraní, přepíšeme celý obsah a znovu ho připojíme.

**A) V30 - Finance: odstranit široký permit Finance → Admin, zbytek ponechat stejné**

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


**B) V70 – Guest: pořadí nejdřív vložíme DENY do interní sítě, až poté PERMIT ANY**

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

#### Ověření

* **ping z PC4 (Finance) na PC1 (Admin)** -> obecný přístup má být blokován, povolené výjimky musí fungovat:
    

```
ping 10.31.10.20        
```
![Final-TEST-1](Pasted%20image%2020250926190148.png)

* **ping z PC11 (Guest)**  všude do interní sítě -> pro ověření SRV02 a PC2 (Sales) ** musí být blokováno:
    

```
ping 10.31.10.11
ping 10.31.20.20
```
![Final-test-2](../images/Pasted%20image%2020250924142334.png)


---

### Závěr  

**Chyby v ACL porušovaly bezpečnostní politiku:**

- V **V30** široký `permit` umožnil Finance obecný přístup do Admin VLAN – v rozporu s pravidlem „Finance nemají mít přístup na Admin/PC1, kromě výjimek“.
    
- Ve **V70** špatné pořadí způsobilo, že `permit any` povolil vše dřív, než se uplatnil deny do interní sítě. Guest tak neprávem přistupoval dovnitř.
    

Po přepsání ACL (odebrání širokého permitu ve V30, správné pořadí deny/permit ve V70) je přístup mezi VLANami v souladu s naší zvolenou bezpečnostní politikou: **Finance mají jen definované výjimky a Guest nemá přístup do interních sítí.**


## 8.6 - Shrnutí

V téhle kapitole jsme začaly u **OSPF**: nejdřív jsme zkontrolovali směrování mezi páteřní sítí routerů a při diagnostice sousedů zjistili problém a pravily chybně přiřazenou oblast, aby se objevily správné trasy a routery o sobě věděli. Pak jsme přešly na **VLANy** – odhalily jsme špatně nastavenou výchozí bránu na routeru pro jednu z VLAN a po nápravě se inter-VLAN komunikace rozběhla.

Ve druhém části kapitoly jsme řešily **DHCP** kdy relé požadavků mířilo na nesprávnou adresu, takže klienti v jedné VLAN nedostávali konfiguraci. Po opravě ip helperu se adresy začaly přidělovat k opravené VLAN dle správného zadání. Na závěr přišla **ACL** -> odstranily jsme široký povolovací řádek ve Finance a srovnaly pořadí pravidel u Guest, aby to odpovídalo naší zvolené bezpečnostní politice. Po těchto úpravách síť funguje podle očekávání.

**Pokračovat na další kapitolu:** Závěr a shrnutí
