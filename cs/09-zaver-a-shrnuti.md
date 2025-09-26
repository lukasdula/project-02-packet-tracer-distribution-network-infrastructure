
# **9 - Shrnutí a závěr**


## Závěrečné zhodnocení


Projekt představuje návrh **střední podnikové distribuční sítě** se čtyřmi routery (jeden jako ISP), dvěma přepínači a servery. Síť je směrována pomocí **OSPF**, páteřní spoje využívají 30bitové masky (/30) a VLAN segmentace rozděluje oddělení do samostatných sítí. NAT/PAT a internet nebyly součástí – místo toho DNS a HTTP fungují lokálně uvnitř firmy.

Funkcionalita infrastruktury je rozšířena oproti prvnímu projektu o více VLAN, nově také o služby **NTP** pro čas a **Syslog** pro logy (s omezeními PT), a **management VLAN 99**, která umožňuje bezpečnou správu switchů přes SSH. Zabezpečení se opírá o účty s hesly, enable secret a bannery MOTD, zatímco port security a administrativní blokace portů byly ponechány z prvního projektu.

Velký důraz byl kladen na **ACL politiku**. Finance VLAN je izolovaná, Guest má pouze přístup k ISP, Admin a Management VLAN mají plná privilegia. ACL a jejich různorodá pravidla pokrývají DHCP, servery, tiskárnu i mezivztahy VLAN a simulují realistickou firemní politiku. ISP a Wi-Fi sloužily jen jako doplňková demonstrace.

Testování a troubleshooting potvrdily správnou funkci všech částí -> páteřní spoje, směrování, služby i bezpečnostní politiky. Síť je stabilní, plně funkční a odpovídá modelu reálné střední podnikové infrastruktury.

## Přehled hlavních kroků projektu

- **Kapitola 1:** Návrh topologie -> čtyři routery (včetně ISP), dva přepínače, servery, popis fyzického propojení a použitých zařízení.
    
- **Kapitola 2:** Adresní a VLAN plán ->definice adresního prostoru, využití /30 masek pro páteřní spoje, rozdělení sítě do více VLAN pro jednotlivá oddělení.
    
- **Kapitola 3:** Základní konfigurace -> pojmenování zařízení, nastavení IP adres, konfigurace rozhraní a propojení routerů a switchů.
    
- **Kapitola 4:** VLANy a subinterface -> vytvoření VLAN, přiřazení portů, konfigurace trunků a Router-on-a-Stick na směrovačích.
    
- **Kapitola 5:** Síťové služby -> DHCP a Syslog na SRV01, DNS a HTTP na SRV02, implementace NTP serveru a základní Wi-Fi přístup pro administrátora.
    
- **Kapitola 6:** Zabezpečení -> uživatelské účty a hesla, enable secret, SSH přístup, MOTD bannery, oddělená management VLAN 99 a detailně zpracované ACL.
    
- **Kapitola 7:** Testování konektivity -> ověření spojů mezi routery pomocí pingů na IP adresy jejich portů, kontrola směrování OSPF, funkce VLAN, DHCP, DNS/HTTP, NTP, SSH i správnost ACL politik.
    
- **Kapitola 8:** Troubleshooting -> ukázky řešení chyb v OSPF, VLAN, DHCP a ACL, náprava konfigurace a potvrzení plné funkčnosti sítě.



**Zpět na přehled projektu:** [Czech version](cs/00-README.cs.md)
