# Capítol 1 · Xarxes i Commutació (M08)

!!! abstract "Sobre aquest document"
    Fonaments teòrics del **Projecte 1 — Posem en marxa la xarxa corporativa**.
    Mòdul **M08 · Planificació i Administració de Xarxes**.

## 1. Com viatja un missatge? El model de capes

### 1.1 El problema que calia resoldre

Als anys 70, els fabricants de xarxes (IBM, DEC, Honeywell...) construïen els seus sistemes de forma propietària: els ordinadors d'IBM no podien parlar amb els de DEC, i viceversa. Internet, tal com la coneixem, hauria sigut impossible. La ISO (International Organization for Standardization) va proposar el model OSI el 1984 com a estàndard universal que qualsevol fabricant podria implementar.

La idea fonamental és elegant: dividir el problema de la comunicació en set subproblemes independents, cadascun resolt per una capa. Cada capa fa una feina concreta i no li importa com la fan les capes de sobre o de sota, sempre que compleixin el "contracte" que hi ha entre elles. Exactament com el sistema postal: el carter no necessita saber si la carta conté un amor o una factura. Ell simplement la porta de A a B.

### 1.2 L'analogia de la carta postal

Per entendre l'encapsulació —el mecanisme central de les xarxes— imagina que vols enviar un contracte confidencial a un advocat d'una altra ciutat. El procés que fas és exactament el que fa un ordinador quan envia dades:

| **El que fas tu (carta postal)** | **El que fa l'ordinador (xarxa)** |
|---|---|
| Escrius el document (el contingut) | L'aplicació genera les dades (ex: una pàgina web) |
| El poses en un sobre i escrius els noms de remitent i destinatari | La capa de transport afegeix els ports origen i destí (ex: port 80 per HTTP) |
| Poses el sobre dins d'un sobre de missatgeria amb el codi postal i l'adreça exacta | La capa de xarxa afegeix les adreces IP origen i destí |
| La missatgeria ho posa en una bossa amb la resta de paquets del camió | La capa d'enllaç l'empaqueta en una trama Ethernet amb les MACs origen i destí |
| El camió circula per la carretera portant els bits físicament | La capa física transmet els bits pel cable com a senyals elèctrics |

Quan el paquet arriba al destinatari, el procés s'inverteix: la missatgeria obre la bossa, l'advocat obre el sobre exterior, obre el sobre interior i llegeix el document. Cada "capa" desempaqueta la seva part i passa la resta a la capa de sobre.

**Conclusió clau:** cada capa de l'ordinador emissor es comunica lògicament amb la capa equivalent de l'ordinador receptor, ignorant tot el que hi ha entre mig. La capa de transport del teu ordinador "parla" amb la capa de transport del servidor web, com si estiguessin connectats directament. Les capes inferiors (routers, switches, cables) només són el camió i la carretera.

### 1.3 El model OSI capa a capa

Veiem ara cada capa en detall, sempre amb un exemple concret de per a què serveix:

#### Capa 7 — Aplicació

És la capa amb la qual interactua l'usuari (o el programari de l'usuari). Defineix com l'aplicació empaqueta les dades que vol enviar i com les interpreta quan les rep. No és l'aplicació en sí (el teu navegador no és la capa 7), sinó el protocol que l'aplicació utilitza.

**Exemples:** Quan escrius una URL al navegador, el protocol HTTP (o HTTPS) de la capa 7 formata la petició. Quan el teu client de correu descarrega emails, usa IMAP o POP3. Quan un servidor Linux fa una còpia de seguretat remota, usa SSH o SFTP. Quan el teu ordinador busca la IP associada a "google.com", usa DNS.

#### Capa 6 — Presentació

S'encarrega de la traducció, el xifratge i la compressió de les dades. La raó d'existir: dos sistemes poden representar les dades de forma diferent (un text en Windows usa codificació diferent a Unix; una imatge pot estar comprimida de diverses formes). La capa 6 garanteix que l'emissor i el receptor s'entenguin.

**Exemple concret:** Quan navegues per HTTPS, la capa de presentació és la que gestiona el xifrat TLS/SSL. Les teves dades es xifren a la capa 6 del teu ordinador i es desxifren a la capa 6 del servidor. Tot el que hi ha entre mig (routers, switches) veu paquets xifrats, incomprensibles.

#### Capa 5 — Sessió

Gestiona l'establiment, manteniment i tancament de sessions de comunicació entre dues aplicacions. Una sessió és una conversa amb context: si baixa la connexió a la meitat d'una transferència de fitxers, la capa de sessió permet reprendre-la des d'on s'havia deixat en lloc de reiniciar des del principi.

**Exemple concret:** Quan fas una videotrucada, la capa de sessió gestiona que les dades d'àudio i vídeo vagin sincronitzades i que la sessió es pugui recuperar si hi ha una interrupció breu de xarxa.

#### Capa 4 — Transport

Aquesta és una de les capes més importants i on apareix una de les decisions fonamentals de les xarxes: envio les dades de forma fiable o ràpida?

**TCP (Transmission Control Protocol) — fiable i ordenat:** Estableix una connexió entre els dos extrems abans d'enviar res (el famós "three-way handshake": SYN → SYN-ACK → ACK). Numera cada paquet (segments) i el receptor confirma cada segment rebut (ACK). Si un segment no arriba o arriba corrupte, TCP el retransmet automàticament. Els paquets sempre arriben en ordre.

**Cost:** latència afegida per les confirmacions. Usat quan la integritat és crítica: descàrregues, webs, correu, SSH.

**UDP (User Datagram Protocol) — ràpid i sense garanties:** Envia els paquets (datagrams) sense establir connexió ni confirmar la recepció. Si un paquet es perd, senzillament desapareix. No hi ha retransmissió ni reordenació.

**Cost:** possible pèrdua de dades. Usat quan la velocitat és crítica i una mica de pèrdua és acceptable: videoconferència (un frame perdut és millor que aturar el vídeo), jocs en línia, streaming, DNS.

**Per quin motiu els ports TCP i UDP van del 0 al 65535?**
```
El camp de port a la capçalera TCP/UDP ocupa 16 bits. 2^16 = 65.536 valors possibles (0 a 65.535).
Ports 0-1023: ports "coneguts" (well-known), reservats per a serveis estàndard (root requerit a Unix).
- 22: SSH - 25: SMTP - 53: DNS - 67/68: DHCP - 80: HTTP - 443: HTTPS
Ports 1024-49151: ports registrats per aplicacions (MySQL: 3306, RDP: 3389).
Ports 49152-65535: ports efímers, assignats dinàmicament als clients per cada connexió.
```

#### Capa 3 — Xarxa

La capa de xarxa s'encarrega de l'encaminament (routing): decidir com enviar un paquet des d'un punt qualsevol de la xarxa fins a un altre, passant per múltiples dispositius intermedis. Utilitza adreces lògiques (IP) que, a diferència de les MACs, es poden estructurar jeràrquicament i encaminar de forma eficient.

**El dispositiu de la capa 3 és el router.** Quan un paquet ha de saltar de la teva xarxa local a Internet, o d'una VLAN a una altra, sempre passa per un router. Els routers llegeixen l'adreça IP destí, consulten la seva taula d'encaminament i decideixen per on enviar el paquet.

#### Capa 2 — Enllaç de dades

La capa d'enllaç gestiona la comunicació entre dispositius dins d'una mateixa xarxa local (el mateix segment de xarxa). Utilitza adreces físiques (MAC) i organitza els bits en trames (frames). Inclou mecanismes de detecció d'errors (FCS - Frame Check Sequence).

**El dispositiu de la capa 2 és el switch.** El switch llegeix les adreces MAC de les trames i les envia al port corresponent, sense mirar les adreces IP. Per al switch, "IP" no existeix; ell només veu trames Ethernet amb MACs.

#### Capa 1 — Física

La capa física és el cable (o l'aire, en el cas del Wi-Fi). Es limita a transmetre bits (0 i 1) en forma de senyals: variacions de tensió elèctrica al cable de coure, polsos de llum a la fibra òptica, ones de ràdio a l'aire. No entén trames, ni adreces, ni protocols. Simplement transmet bits.

**Estàndard Ethernet (IEEE 802.3):** defineix les característiques físiques del cablejat, els connectors (RJ-45), les velocitats (100 Mbps, 1 Gbps, 10 Gbps) i com es codifiquen els bits elèctricament.

### 1.4 Encapsulació: el procés pas a pas

Quan envies una petició HTTP a un servidor web, el procés d'encapsulació és exactament aquest (de dalt a baix a l'emissor, de baix a dalt al receptor):

| **Pas** | **Capa** | **Acció** | **Unitat resultant** |
|---|---|---|---|
| 1 | Aplicació (7) | El navegador crea la petició HTTP: 'GET /index.html HTTP/1.1' | Dades HTTP |
| 2 | Transport (4) | TCP afegeix capçalera: port origen (efímer, ex: 54321), port destí (80), número de seqüència, flags. | Segment TCP |
| 3 | Xarxa (3) | IP afegeix capçalera: IP origen (la teva), IP destí (del servidor), TTL=64, protocol=TCP. | Paquet IP |
| 4 | Enllaç (2) | Ethernet afegeix capçalera (MAC origen, MAC destí del router) i FCS al final. | Trama Ethernet |
| 5 | Física (1) | El bit es converteix a senyal elèctrica i viatja pel cable. | Bits →→→→ |

**Observació important sobre les MACs en el camí**
```
Cada vegada que el paquet passa per un router, les adreces MAC CANVIEN.
Les adreces IP origen i destí es mantenen igual durant tot el trajecte (excepte si hi ha NAT).
Exemple: El teu PC envia una trama al router.
MAC origen: \[MAC del teu PC\] → al router, la trama s'elimina i es crea una nova
MAC destí: \[MAC del router\] → nova MAC origen: \[MAC interfície WAN del router\]
IP origen: 192.168.10.5 (no canvia mai) MAC destí: \[MAC del proper router\]
IP destí: 93.184.216.34 (no canvia mai)
Per tant: MACs = comunicació local, port a port. IPs = comunicació extrema, end-to-end.
```

### 1.5 El model TCP/IP: el que s'usa a la pràctica

El model OSI és excel·lent per entendre i diagnosticar xarxes, però cap protocol l'implementa de forma estricta. En la pràctica s'utilitza el model TCP/IP (també llamat model d'Internet), que agrupa les 7 capes OSI en 4:

| **Model TCP/IP** | **Capes OSI equivalents** | **Protocols representatius** |
|---|---|---|
| Aplicació | Sessió (5) + Presentació (6) + Aplicació (7) | HTTP, HTTPS, FTP, SSH, SMTP, IMAP, DNS, DHCP, SNMP |
| Transport | Transport (4) | TCP, UDP |
| Internet | Xarxa (3) | IPv4, IPv6, ICMP, ARP |
| Accés a la xarxa | Física (1) + Enllaç (2) | Ethernet, Wi-Fi (802.11), PPP |

A la pràctica, quan parlem de xarxes en un context professional, ens referirem a les capes TCP/IP, però usarem el model OSI per localitzar on és un problema. Si un ping falla (ICMP, capa 3), el problema és a la capa de xarxa o inferior. Si el ping funciona però no funciona HTTP, el problema és a la capa d'aplicació o transport.

## 2. Adreces MAC i Ethernet: qui és qui a la xarxa local

### 2.1 Adreces MAC

Cada interfície de xarxa (la targeta de xarxa del teu ordinador, el port del switch, la targeta Wi-Fi) porta gravada de fàbrica una adreça MAC (Media Access Control). És un identificador físic únic a tot el món, com el número de sèrie d'un dispositiu.

Una adreça MAC té 48 bits i s'escriu en hexadecimal, normalment en grups de dos dígits separats per dos punts o guions:

AA:BB:CC:DD:EE:FF

Els primers 24 bits (3 bytes) identifiquen el fabricant (OUI - Organizationally Unique Identifier). Els darrers 24 bits els assigna el fabricant per a cada dispositiu. Per exemple, adreces que comencen per 00:1A:A0 pertanyen a Dell; 00:50:56 és VMware.

**Per què les MACs no poden encaminar el trànsit per Internet?**
```
Les MACs no tenen estructura jeràrquica: no hi ha cap relació entre la MAC d'un dispositiu de
Barcelona i la d'un de Tokyo. Per saber on és una MAC, caldria una taula amb totes les MACs
del món i el seu lloc. Impossible d'escalar.
Les IPs, en canvi, SÍ estan estructurades jeràrquicament: 192.168.10.x és sempre una xarxa
privada concreta, i tots els routers saben que 8.8.8.0/24 és a Google (USA).
Conclusió: MACs per a la comunicació local. IPs per a la comunicació global.
```

### 2.2 El protocol Ethernet i les trames

Ethernet (IEEE 802.3) defineix com s'organitzen les dades en trames per viatjar per un segment de xarxa local. Una trama Ethernet té una estructura ben definida:

| **Camp** | **Mida** | **Contingut** |
|---|---|---|
| Preàmbul + SFD | 8 bytes | Sincronització: bits alternats (1010\...) per sincronitzar els rellotges. SFD (10101011) indica el final del preàmbul. |
| MAC destí | 6 bytes | Adreça MAC del receptor. Pot ser unicast (un dispositiu), multicast (grup) o broadcast (FF:FF:FF:FF:FF:FF = tots). |
| MAC origen | 6 bytes | Adreça MAC del transmissor. |
| EtherType | 2 bytes | Indica el protocol de capa 3 contingut: 0x0800 = IPv4, 0x0806 = ARP, 0x86DD = IPv6, 0x8100 = trama 802.1Q (VLAN). |
| Dades (payload) | 46--1500 bytes | El contingut: el paquet IP, o qualsevol altre protocol de capa 3. Mínim 46 bytes (s'emplena amb padding si cal). |
| FCS | 4 bytes | Frame Check Sequence: CRC-32 per detectar errors de transmissió. El receptor recalcula i compara. |

**MTU (Maximum Transmission Unit):** el camp de dades d'Ethernet estàndard pot contenir fins a 1.500 bytes. Aquest és el MTU d'Ethernet, i és el motiu pel qual el teu sistema operatiu fragmenta paquets IP grans en múltiples trames.

## 3. El commutador (Switch): el cor de la xarxa local

### 3.1 Del hub al switch: per què importa la diferència

Per entendre el switch, primer cal entendre per què els hubs eren un problema. Un hub (concentrador) és un dispositiu extremadament simple: rep un senyal per un port i l'envia per TOTS els altres ports. Tots els dispositius connectats al hub comparteixen el mateix domini de col·lisió: si dos dispositius envien dades simultàniament, els senyals col·lideixen i s'han de retransmetre.

El switch va ser la solució. Un switch aprèn quins dispositius estan connectats a cada port i envia cada trama únicament al port on es troba el destinatari. Resultat: cada port del switch té el seu propi domini de col·lisió, les col·lisions desapareixen pràcticament, i l'ample de banda s'aprofita molt millor.

### 3.2 La taula CAM: com aprèn el switch

La taula CAM (Content Addressable Memory), també coneguda com a taula de MACs, és la base de dades del switch. Conté l'associació entre cada adreça MAC i el port per on s'ha vist aquella adreça. La gestió és totalment automàtica i dinàmica.

El procés d'aprenentatge és senzill i elegant:

1.  **Arribada d'una trama:** El switch rep una trama per un port (ex: port 3). Llegeix la MAC origen (AA:BB:CC:DD:EE:01) i la registra a la taula CAM associada al port 3. Acaba d'aprendre on és aquest dispositiu.

2.  **Cerca del destinatari:** Llegeix la MAC destí (AA:BB:CC:DD:EE:02) i busca a la taula CAM.

3.  **Si la troba (unicast forwarding):** Envia la trama únicament per aquell port. Cap altre dispositiu veu la trama.

4.  **Si NO la troba (flooding):** Envia la trama per tots els ports excepte el d'entrada. El dispositiu destí, quan respongui, donarà a conèixer la seva ubicació, i el switch aprendrà.

5.  **Caducitat:** Les entrades de la taula CAM caduquen si no hi ha activitat (per defecte 300 segons = 5 minuts). Evita que la taula quedi obsoleta quan els dispositius es desconnecten o canvien de port.

**Broadcast:** La MAC FF:FF:FF:FF:FF:FF és l'adreça de broadcast d'Ethernet. El switch SEMPRE inunda les trames de broadcast per tots els ports. Això és important: protocols com ARP i DHCP Discover usen broadcast per trobar dispositius. Tots els dispositius del segment veuen i processen les trames de broadcast.

**Exemple pràctic: Taula CAM del switch de NetCorp (fragament)**
```
MAC Address VLAN Port Age
AA:BB:CC:11:22:33 10 Gi0/1 45 s ← PC de Direcció
AA:BB:CC:44:55:66 20 Gi0/5 12 s ← PC d'IT
AA:BB:CC:77:88:99 20 Gi0/6 289 s ← PC d'IT
AA:BB:CC:AA:BB:CC 99 Gi0/24 4 s ← Router Cisco
Llegir: 'Si rebo una trama amb MAC destí AA:BB:CC:11:22:33, l'envio per Gi0/1'
```

### 3.3 VLANs: dividir el switch en múltiples xarxes virtuals

Imagina que a NetCorp tots els 45 empleats estan connectats al mateix switch. Si el servidor DHCP fa un broadcast, els 45 ordinadors l'han de processar, incloent-hi els de la direcció, els de logística i els de RRHH. Si algú del departament de vendes intenta fer un escaneig de la xarxa, podria veure trànsit de tots els departaments. Cap empresa acceptaria això.

Les VLANs (Virtual Local Area Networks) resolen aquest problema creant múltiples xarxes lògiques completament independents dins del mateix switch físic. Cada VLAN és un domini de broadcast independent: el trànsit d'una VLAN no arriba mai als dispositius d'una altra VLAN (tret que passi per un router).

**Beneficis concrets de les VLANs a NetCorp:**

-   **Seguretat:** Un ordinador de Logística mai veurà el trànsit de RRHH, ni viceversa.

-   **Rendiment:** El broadcast de DHCP d'IT no molesta als usuaris de Vendes.

-   **Flexibilitat:** Pots moure un treballador de departament sense canviar el cablejat físic: simplement canvies la VLAN assignada al seu port.

-   **Gestió:** Pots aplicar polítiques de seguretat (ACLs, QoS) per VLAN de forma centralitzada.

**Ports d'accés (access ports)**

Un port d'accés pertany a una única VLAN. S'utilitza per connectar dispositius finals: ordinadors, impressores, telèfons IP, càmeres. El dispositiu connectat no sap que existeix la VLAN, ni que la trama Ethernet porta una etiqueta. El switch afegeix l'etiqueta quan la trama entra i la treu quan surt. Per al dispositiu, és totalment transparent.

**Ports trunk**

Un port trunk transporta trànsit de múltiples VLANs simultàniament. S'usa per:

-   Connectar dos switches entre ells (uplink)

-   Connectar un switch amb un router (per al router-on-a-stick)

-   Connectar un switch amb un servidor (si el servidor és host de màquines virtuals de diverses VLANs)

Com sap el switch receptor a quina VLAN pertany cada trama en un trunk? Gràcies al protocol IEEE 802.1Q:

**El protocol IEEE 802.1Q: l'etiqueta VLAN**

IEEE 802.1Q defineix com s'etiqueten les trames Ethernet per identificar la VLAN. Insereix 4 bytes addicionals a la trama original, just ENTRE les adreces MAC i el camp EtherType:

| **Camp 802.1Q** | **Mida** | **Valor / Descripció** |
|---|---|---|
| TPID (Tag Protocol ID) | 2 bytes | Sempre 0x8100. Indica al receptor que la trama porta etiqueta 802.1Q. |
| PCP (Priority Code Point) | 3 bits | Prioritat QoS (0-7). Permet prioritzar trànsit de veu o vídeo. |
| DEI (Drop Eligible) | 1 bit | Indica si la trama pot ser descartada en cas de congestió. |
| VID (VLAN Identifier) | 12 bits | L'identificador de la VLAN: valors de 0 a 4095 (1-4094 usables). 12 bits → 2^12 = 4096 VLANs possibles. |

**La VLAN nativa:** En un port trunk, la VLAN nativa és l'única que viatja sense etiqueta. Per defecte, Cisco usa la VLAN 1. Per seguretat (per evitar ataques de VLAN hopping, que veurem a C037), cal canviar la VLAN nativa a una VLAN específica i buida (ex: VLAN 999) que no s'usi per al trànsit de producció. Els dos extrems del trunk han de tenir la mateixa VLAN nativa configurada.

### 3.4 Spanning Tree Protocol (STP): evitant els bucles de xarxa

Les xarxes ben dissenyades en producció inclouen connexions redundants: si cau un cable o un switch, hi ha un camí alternatiu i la xarxa continua funcionant. Però la redundància a la capa 2 crea un problema greu: els bucles.

**El problema: broadcast storm**

Imagina dos switches (A i B) connectats amb dos cables entre ells (redundància). Un PC envia una trama de broadcast (com un DHCP Discover):

1.  El switch A rep la trama i la reenvía per tots els seus ports, incloent els dos cables cap a B.

2.  El switch B rep la trama pels dos cables i la torna a reenviar per tots els ports, incloent els dos cables cap a A.

3.  El switch A rep de nou les trames i les torna a reenviar\... i així fins a l'infinit.

En pocs mil·lisegons, la xarxa s'omple completament de trames de broadcast circulant en bucle. És la "broadcast storm": la xarxa col·lapsa completament i cap dispositiu pot comunicar-se. STP (IEEE 802.1D) resol aquest problema bloqueig lògicament els ports redundants.

**Com funciona STP: l'elecció del Root Bridge**

STP construeix un arbre lliure de bucles a partir d'un punt de referència central: el Root Bridge. El procés d'elecció és automàtic:

1.  **Tots els switches enviuen BPDUs:** Un BPDU (Bridge Protocol Data Unit) és un missatge especial que els switches s'envien entre ells per intercanviar informació sobre STP. Contenen el Bridge ID del switch emissor.

2. **Bridge ID = Prioritat + MAC:** El Bridge ID és la combinació de la prioritat del switch (valor entre 0 i 61440, en múltiples de 4096; per defecte 32768) i la seva adreça MAC. Menor Bridge ID = guanya l'elecció de Root Bridge.

3. **El switch amb el menor Bridge ID és el Root Bridge:** Tots els altres switches calculen el "camí de menor cost" fins al Root Bridge i bloquegen els ports que crearien bucles.

**Implicació pràctica:** A NetCorp, hauries de configurar manualment la prioritat del switch principal (el del rack del servidor) perquè sigui el Root Bridge. Si no ho fas, el Root Bridge podria ser qualsevol switch (el que tingui la MAC menor), i l'arbre resultant podria ser subòptim.

**Els estats dels ports en STP**

Un port STP no passa directament de "apagat" a "funcionant". Passa per diversos estats per assegurar-se que no crea un bucle:

| **Estat** | **Durada típica** | **Reenvia trames?** | **Aprèn MACs?** | **Descripció** |
|---|---|---|---|---|
| Blocking | Indefinit | No | No | El port és redundant i aporta risc de bucle. Bloquejat. Escolta BPDUs però no fa res més. |
| Listening | 15 s (Forward Delay) | No | No | Transició. El port es prepara per activar-se. Escolta i participa en STP per assegurar-se que no hi ha bucle. |
| Learning | 15 s (Forward Delay) | No | SÍ | Transició. Comença a aprendre adreces MAC de les trames que veu, però encara no reenvia. Evita flooding massiu quan s'activi. |
| Forwarding | Indefinit | SÍ | SÍ | Estat operatiu normal. Reenvia trames i aprèn MACs. |
| Disabled | Indefinit | No | No | Port apagat administrativament (shutdown). |

**Temps de convergència de STP clàssic:** En cas de fallada d'un camí, STP tarda fins a 50 segons en activar el camí redundant (20 s de Max Age + 15 s Listening + 15 s Learning). Això és inacceptable en xarxes modernes. Per això Cisco usa RSTP (Rapid STP, IEEE 802.1w), que redueix la convergència a menys de 2 segons. Cisco PVST+ executa una instància de RSTP per cada VLAN.

### 3.5 Seguretat al switch Cisco

**Port Security: qui pot connectar-se on**

Port Security permet a l'administrador controlar exactament quins dispositius (MACs) poden usar cada port del switch. És la primera línia de defensa contra dispositius no autoritzats.

**Funcionament:** Configures un port d'accés amb un nombre màxim de MACs permeses (ex: màxim 1). Si un dispositiu no autoritzat intenta connectar-se (o si el nombre de MACs supera el límit), el port actua segons el mode de violació configurat:

| **Mode de violació** | **Què passa quan es viola?** | **Incrementa el comptador?** | **Envia alerta (syslog/SNMP)?** |
|---|---|---|---|
| protect | Descarta les trames de la MAC no autoritzada. La xarxa continua funcionant per a les MACs autoritzades. | No | No |
| restrict | Igual que protect, però registra l'incident. | Sí | Sí |
| shutdown (recomanat) | Posa el port en estat err-disabled. Requereix intervenció manual per recuperar-lo: shutdown + no shutdown. | Sí | Sí |

**Sticky MAC:** El mode sticky permet al switch aprendre dinàmicament les primeres MACs que veu al port i guardar-les a la configuració de forma permanent (running-config). Combina la comoditat de l'aprenentatge automàtic amb la seguretat d'una llista estàtica. Ideal en entorns on els dispositius no canvien sovint.

**BPDU Guard: protegint l'arbre STP**

El BPDU Guard és una mesura de seguretat que protegeix l'estructura STP de la xarxa. Un port d'accés (connectat a un PC, impressora...) mai hauria de rebre BPDUs STP, ja que els dispositius finals no participen en STP. Si un port d'accés rep un BPDU, significa que algú ha connectat un switch (potser no autoritzat) a aquell port.

**Acció de BPDU Guard:** Quan un port configurat amb BPDU Guard rep un BPDU, posa el port immediatament en err-disabled. Evita que un switch extern no autoritzat participi en STP i potencialment es converteixi en Root Bridge, canviant l'arbre de commutació de tota la xarxa.

**Dynamic ARP Inspection (DAI) — context previ a C037**

ARP (Address Resolution Protocol) resol la MAC associada a una IP dins la mateixa xarxa. El problema és que ARP no té autenticació: qualsevol dispositiu pot enviar respostes ARP falses. L'atac s'anomena ARP Spoofing i permet interceptar el trànsit de la xarxa. DAI valida les respostes ARP contra la taula de bindings DHCP Snooping per detectar i descartar respostes ARP fraudulentes. Ho veurem en profunditat al capítol de C037.

## 4. Adreçament IPv4: la lògica darrere dels números

### 4.1 Per què necessitem adreces IP si ja tenim MACs?

Hem vist que les MACs identifiquen de forma única cada dispositiu de xarxa. Però hi ha un problema fonamental: les MACs no es poden encaminar. Imagina que vols enviar una carta (trama) a un dispositiu de Tokio. Amb MACs, el teu router hauria de tenir una taula amb les 20 mil milions de MACs de tots els dispositius del món i saber on es troba cadascuna. Impossible.

Les adreces IP resoler aquest problema gràcies a la seva estructura jeràrquica. Una IP no és simplement un identificador únic: conté informació sobre la xarxa on es troba el dispositiu. Tots els routers del món poden saber que "93.184.216.0/24" és una xarxa als EUA sense necessitat de conèixer cada dispositiu individual. Encaminen els paquets a la xarxa correcta, i el router local de destí s'encarrega d'entregar-lo al dispositiu específic.

### 4.2 Estructura d'una adreça IPv4

Una adreça IPv4 és un número de 32 bits. Per facilitar la lectura humana, s'escriu en notació decimal puntejada: quatre grups de 8 bits (octets), cadascun expressat en decimal (0--255), separats per punts.

**Exemple:** 192.168.10.50

Cada adreça IPv4 té dues parts:

-   **Part de xarxa (network bits):** identifica la xarxa a la qual pertany el dispositiu. Tots els dispositius de la mateixa xarxa comparteixen la mateixa part de xarxa.

-   **Part d'host (host bits):** identifica el dispositiu específic dins de la xarxa.

Quants bits formen la part de xarxa i quants la d'host? Això ho determina la màscara de subxarxa.

### 4.3 Màscares de subxarxa: on acaba la xarxa i on comença l'host

La màscara de subxarxa és un altre número de 32 bits. Els bits a '1' indiquen la part de xarxa; els bits a '0' indiquen la part d'host. Sempre forma un bloc continu de '1' seguit d'un bloc continu de '0' (mai alternats).

La notació CIDR (Classless Inter-Domain Routing) simplifica l'escriptura posant el nombre de bits a '1' com a sufix de l'adreça:

| **Màscara decimal** | **Màscara en binari (últim octet)** | **Notació CIDR** | **Nre. d'hosts útils** |
|---|---|---|---|
| 255.255.255.0 | 11111111.11111111.11111111.00000000 | /24 | 254 |
| 255.255.255.128 | 11111111.11111111.11111111.10000000 | /25 | 126 |
| 255.255.255.192 | 11111111.11111111.11111111.11000000 | /26 | 62 |
| 255.255.255.224 | 11111111.11111111.11111111.11100000 | /27 | 30 |
| 255.255.255.240 | 11111111.11111111.11111111.11110000 | /28 | 14 |
| 255.255.255.248 | 11111111.11111111.11111111.11111000 | /29 | 6 |
| 255.255.255.252 | 11111111.11111111.11111111.11111100 | /30 | 2 |

**La fórmula dels hosts útils:** Si hi ha n bits de host, el nombre total d'adreces és 2^n. Però dues adreces no es poden assignar a dispositius: la primera (adreça de xarxa, tots els bits d'host a 0) i l'última (broadcast, tots els bits d'host a 1). Per tant: hosts útils = 2^n - 2.

**Per què /30 és l'ideal per als enllaços punt a punt entre routers?**
```
Un enllaç entre dos routers necessita exactament 2 adreces: una per a cada router.
Amb /30: 2^2 - 2 = 2 hosts útils. Perfecte. Cap adreça es malgasta.
Exemple: 10.0.0.0/30 → .0 = xarxa, .1 = router A, .2 = router B, .3 = broadcast
Si usessis /24 per a aquest enllaç, malgastaríes 252 adreces que no s'usaran mai.
```

### 4.4 Subnetting: dividint una xarxa en trossos més petits

El subnetting és el procés de dividir una xarxa gran en subxarxes més petites. En comptes de tenir una sola xarxa 192.168.10.0/24 (254 hosts), pots crear 4 subxarxes de /26 (62 hosts cadascuna). Cada subxarxa pot correspondre a un departament, una VLAN, un pis d'un edifici, etc.

**Pas 1: entendre el bloc de subxarxa**

El primer pas és entendre la mida del bloc de cada màscara. El truc és senzill: 256 - valor_de_la_màscara_en_l'octet_interessant = mida del bloc.

**Exemple amb /26:** La màscara és 255.255.255.192. En l'últim octet, 256 - 192 = 64. Cada subxarxa /26 ocupa un bloc de 64 adreces: 0-63, 64-127, 128-191, 192-255.

**Pas 2: calcular totes les subxarxes**

Per a una xarxa pare 192.168.10.0, si la dividim en /26, les quatre subxarxes resultants son:

| **Subxarxa** | **Rang complet** | **Primera assignable** | **Última assignable** | **Broadcast** |
|---|---|---|---|---|
| 192.168.10.0/26 | 0 -- 63 | 192.168.10.1 | 192.168.10.62 | 192.168.10.63 |
| 192.168.10.64/26 | 64 -- 127 | 192.168.10.65 | 192.168.10.126 | 192.168.10.127 |
| 192.168.10.128/26 | 128 -- 191 | 192.168.10.129 | 192.168.10.190 | 192.168.10.191 |
| 192.168.10.192/26 | 192 -- 255 | 192.168.10.193 | 192.168.10.254 | 192.168.10.255 |

### 4.5 VLSM: el subnetting intel·ligent

El subnetting clàssic divideix una xarxa en subxarxes de la mateixa mida. Però a la pràctica, els departaments d'una empresa no tenen tots el mateix nombre d'empleats. Si fixes /26 per a tothom, el departament de Direcció (5 persones) malgastarà 57 adreces. VLSM (Variable Length Subnet Masking) permet usar màscares de mida diferent per a cada subxarxa.

**Metodologia VLSM (pas a pas)**

1. **Llistar les subxarxes necessàries ordenades de MAJOR a MENOR** nombre d'hosts requerits. Sempre comencem per la més gran.

2. **Per a cada subxarxa, escollir la màscara mínima** que cobreix el nombre d'hosts (2^n - 2 ≥ hosts requerits).

3. **Assignar les adreces de forma contigua**, sense deixar buits ni solapar subxarxes.

**Aplicació a NetCorp S.L.**

NetCorp vol usar l'espai 192.168.0.0/16 i necessita les VLAN següents:

| **VLAN** | **Departament** | **Hosts necessaris** | **Màscara** | **Hosts útils** | **Xarxa assignada** |
|---|---|---|---|---|---|
| 30 | Logística | 50 | /26 | 62 | 192.168.30.0/26 |
| 20 | IT | 20 | /27 | 30 | 192.168.20.0/27 |
| 40 | Vendes | 15 | /27 | 30 | 192.168.40.0/27 |
| 10 | Direcció | 5 | /29 | 6 | 192.168.10.0/29 |
| 50 | RRHH | 5 | /29 | 6 | 192.168.50.0/29 |
| 99 | Gestió (switches/routers) | 10 | /28 | 14 | 192.168.99.0/28 |
| — | VPN WireGuard (tunnel) | 2 | /30 | 2 | 10.200.0.0/30 |

Amb VLSM assignem a cada departament exactament el que necessita. Si haguéssim usat /24 per a tots, hauríem malbaratat 5 × (254 - hosts_usats) = centenars d'adreces.

### 4.6 Adreces especials i rangs privats

No totes les adreces IPv4 es poden assignar lliurement als dispositius. Hi ha diverses categories d'adreces reservades per a usos específics:

| **Rang / Adreça** | **Ús** | **RFC** |
|---|---|---|
| 10.0.0.0/8 | Xarxa privada (classe A). Comú en grans empreses i operadors. | RFC 1918 |
| 172.16.0.0/12 | Xarxa privada (classe B). Rang 172.16.x.x fins 172.31.x.x. | RFC 1918 |
| 192.168.0.0/16 | Xarxa privada (classe C). El més usat en LANs domèstiques i de PIME. | RFC 1918 |
| 127.0.0.0/8 | Loopback. 127.0.0.1 és el dispositiu local. Mai surt per la xarxa. | RFC 5735 |
| 169.254.0.0/16 | APIPA (Automatic Private IP Addressing). El SO s'autoassigna una adreça quan no troba DHCP. | RFC 3927 |
| 0.0.0.0/0 | Ruta per defecte ("qualsevol destí"). Usada en taules d'encaminament. | — |
| 255.255.255.255 | Broadcast limitat. Envia a tots els dispositius del segment local. No travessa routers. | — |

**Per què les IPs privades no funcionen a Internet?** Els routers d'Internet estan configurats per no encaminar trànsit de les rangs RFC 1918. Si un paquet amb IP origen 192.168.1.5 arriba a un router d'Internet, el descarrega. Per sortir a Internet, els dispositius privats necessiten NAT (que veurem a la secció 6).

## 5. El router: connectant xarxes entre elles

### 5.1 Quin problema resol el router?

Un switch gestiona la comunicació dins d'una xarxa local (una VLAN): tot el que queda al mateix segment. Però quan el PC d'IT (192.168.20.5) vol enviar un missatge a un PC de Logística (192.168.30.10), les dues màquines estan en xarxes IP completament diferents. Un switch de capa 2 no entén d'IPs: no pot prendre aquesta decisió.

El router és el dispositiu que resol exactament aquest problema. Opera a la capa 3 (xarxa) i s'encarrega de:

-   Interconnectar xarxes IP diferents (inter-VLAN routing, connexió a Internet)

-   Mantenir una taula d'encaminament: sap per on enviar paquets destinats a cada xarxa

-   Decrementar el TTL de cada paquet (Time To Live), i descartar-lo si arriba a zero (evita bucles infinits)

-   Modificar les adreces MAC de les trames en cada salt (hop)

### 5.2 La taula d'encaminament

La taula d'encaminament (routing table) és la llista de xarxes que el router coneix i les instruccions per arribar-hi. Cada entrada conté: la xarxa destí (amb màscara), la mètrica (cost del camí), el next-hop (adreça IP del proper router), i la interfície de sortida.

Les rutes es classifiquen per origen:

-   **Directament connectades (C):** el router coneix les xarxes de les seves pròpies interfícies. S'afegeixen automàticament quan s'activa una interfície amb IP.

-   **Estàtiques (S):** configurades manualment per l'administrador. Simples i previsibles, però no s'adapten a canvis automàticament.

-   **Dinàmiques (O, R, B\...):** apreses automàticament de protocols d'encaminament com OSPF (O) o RIP (R). El router intercanvia informació amb els routers veïns i construeix la taula sol. En xarxes grans és imprescindible.

**Com llegir la taula d'encaminament Cisco (show ip route)**
```
C 192.168.10.0/29 is directly connected, GigabitEthernet0/0.10
→ Xarxa directament connectada. Arribo per la subinterfície .10.
S 0.0.0.0/0 \[1/0\] via 10.0.1.1
→ Ruta estàtica per defecte. Tot el que no sé on és, ho envio a 10.0.1.1.
→ \[distància administrativa / mètrica\]. 1 = estàtica.
O 172.16.1.0/24 \[110/2\] via 192.168.99.2
→ Ruta apresa per OSPF. Cost = 2. Next-hop: 192.168.99.2.
→ \[110 = distància adm. d'OSPF / 2 = mètrica OSPF\]
```

### 5.3 Rutes estàtiques i ruta per defecte

**Ruta estàtica:** una entrada a la taula d'encaminament configurada manualment. Forma bàsica a Cisco IOS:

> ip route \[xarxa_destí\] \[màscara\] \[adreça_next-hop\]
>
> ip route 172.16.1.0 255.255.255.0 192.168.99.2
>
> → Per arribar a 172.16.1.0/24, envia els paquets a 192.168.99.2

**Ruta per defecte (default route):** captura tots els paquets amb destí desconegut. És l'última línia de la taula d'encaminament, i correspon a "si no sé on anar, tira cap aquí". A Cisco:

> ip route 0.0.0.0 0.0.0.0 \[IP_del_proveïdor_o_gateway\]

### 5.4 Router-on-a-stick: inter-VLAN routing amb un sol cable

El router-on-a-stick és la solució per encaminar trànsit entre VLANs amb una sola interfície física del router (o amb els ports físics limitats que tenim). En lloc d'usar una interfície física per a cada VLAN, creem subinterfícies lògiques sobre la mateixa interfície física, que es connecta a un port trunk del switch.

**Com funciona, pas a pas**

Suposem que un PC de VLAN 20 (IT, 192.168.20.5) vol comunicar-se amb un PC de VLAN 30 (Logística, 192.168.30.8):

1. El PC d'IT veu que 192.168.30.8 no és la seva xarxa (diferent subxarxa). Envia el paquet al seu gateway: 192.168.20.1.

2. El switch rep la trama i la posa en un port trunk cap al router, afegint l'etiqueta VLAN 20.

3. El router rep la trama etiquetada amb VLAN 20 per la subinterfície .20. Li treu l'etiqueta.

4. El router consulta la taula d'encaminament: 192.168.30.8 pertany a la subxarxa 192.168.30.0/26, connectada a la subinterfície .30.

5. El router envia el paquet per la subinterfície .30 cap al switch, amb l'etiqueta VLAN 30.

6. El switch llegeix l'etiqueta VLAN 30, elimina l'etiqueta i envia la trama al port d'accés on és 192.168.30.8.

**Punt clau:** el paquet ha viatjat del PC d'IT al router i del router al PC de Logística pel MATEIX cable físic entre switch i router. El truc és que les dues VLANs viatgen com trames etiquetades pel trunk i el router les processa de forma independent per cada subinterfície.

**Exemple de subinterfícies al Router Cisco per a NetCorp**
```
interface GigabitEthernet0/0
no ip address
no shutdown
interface GigabitEthernet0/0.10
description VLAN10-Direccio
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.248
interface GigabitEthernet0/0.20
description VLAN20-IT
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.224
interface GigabitEthernet0/0.30
description VLAN30-Logistica
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.192
```

## 6. NAT i PAT: com sortim a Internet amb IPs privades

### 6.1 El problema que va originar NAT

A finals dels anys 90 va quedar clar que les adreces IPv4 (32 bits = 4.294.967.296 adreces totals) s'esgotarien aviat. Amb l'explosió d'Internet i la proliferació de dispositius, no hi havia prou IPs públiques per a tothom. La solució provisional —que encara és la dominant avui dia— va ser el NAT.

La idea és simple: les xarxes privades (empreses, cases) usen adreces privades (RFC 1918) internament, que no necessiten ser úniques a escala global. Un router amb NAT fa de traductor: quan un dispositiu intern vol comunicar-se amb Internet, el router substitueix la IP privada per la IP pública del router. Des d'Internet, sembla que totes les comunicacions provenen d'una sola IP.

### 6.2 Tipus de NAT

| **Tipus** | **Funcionament** | **Cas d'ús típic** |
|---|---|---|
| NAT estàtic | Mapeja de forma permanent una IP privada a una IP pública. Relació 1 a 1. | Servidors interns que cal publicar a Internet (web, FTP, correu): sempre han de ser accessibles des de la mateixa IP pública. |
| NAT dinàmic | Pool d'IPs públiques. Les IPs privades prenen IPs públiques del pool mentre les necessiten. Quan acaben, les alliberen. | Empreses que disposen de múltiples IPs públiques però no necessiten que cada dispositiu tingui sempre la mateixa. |
| PAT (Port Address Translation) = NAT Overload | Moltes IPs privades comparteixen UNA SOLA IP pública. Les connexions es distingeixen pel port TCP/UDP. | La immensa majoria d'empreses, cases i xarxes mòbils. El cas pràctic de NetCorp. |

### 6.3 Com funciona PAT en detall

PAT és el mecanisme que permet que literalment milers de dispositius d'una empresa comparteixin una única IP pública. El secret és el camp de port TCP/UDP:

1. El PC d'IT (192.168.20.5) obre una connexió HTTP al servidor 93.184.216.34:80. El SO assigna un port efímer origen, ex: 54321.

2. El paquet arriba al router: IP origen 192.168.20.5, port origen 54321.

3. El router crea una entrada a la taula NAT: IP_privada:port_privat ↔ IP_pública:port_nou. Ex: 192.168.20.5:54321 ↔ 80.20.1.1:40001.

4. El router reescriu el paquet: canvia l'IP origen per la pública (80.20.1.1) i el port origen per 40001. L'envia a Internet.

5. El servidor 93.184.216.34 respon a 80.20.1.1:40001.

6. El router rep la resposta, consulta la taula NAT i tradueix 80.20.1.1:40001 → 192.168.20.5:54321. Envia la resposta al PC.

**Per què PAT funciona amb milers de dispositius simultàniament?**
```
El port és un número de 16 bits → 65.535 valors possibles.
Cada connexió oberta pel router usa un port diferent al costat públic.
Un router pot gestionar fins a \~65.000 connexions simultànies amb una sola IP pública.
A la pràctica, les connexions es tanquen ràpidament, alliberant ports per a noves connexions.
Limitació de NAT: els dispositius externs NO poden iniciar connexions cap a dispositius interns
sense una regla NAT estàtica explícita. Això és un problema per a servidors, però una
avantatge de seguretat per als dispositius clients.
```

## 7. Diagnòstic de xarxa: llegir el que ens diu l'equip

### 7.1 Filosofia del diagnòstic

Quan una xarxa no funciona, la temptació és provar coses a l'atzar esperant que alguna solucioni el problema. El bon tècnic, en canvi, segueix una metodologia: comença per la capa 1 (el cable) i puja cap a les capes superiors fins a trobar on falla. La majoria de problemes es troben a les primeres capes.

**El mètode en cinc passos:** (1) El cable està ben connectat i les llums del port estan actives? (2) ping al gateway de la mateixa VLAN? (3) ping al gateway de l'altra VLAN? (4) ping a una IP d'Internet? (5) resolució DNS funciona? Si un pas falla, el problema és en aquella capa o les inferiors.

### 7.2 Comandes Cisco IOS imprescindibles

Cisco IOS té un conjunt de comandes de diagnòstic que cal dominar. S'executen des del mode EXEC privilegiat (després del símbol #):

| **Comanda** | **Informació clau que mostra** | **Quan usar-la** |
|---|---|---|
| show ip interface brief | Llista totes les interfícies: nom, IP, estat (up/down) i protocol (up/down). En un cop d'ull veus si totes les interfícies estan actives. | Primer pas sempre. Si una interfície mostra 'administratively down' és perquè té un shutdown. |
| show vlan brief | Llista totes les VLANs i els ports assignats a cada una. Permet veure si un port ha estat assignat a la VLAN correcta. | Quan un PC no obté IP o no es comunica amb la seva VLAN. |
| show interfaces Gi0/1 | Detall complet d'una interfície: MAC, velocitat, errors de transmissió/recepció, estadístiques. | Quan hi ha errors de cable o de compatibilitat de velocitat/duplex. |
| show ip route | Taula d'encaminament completa: xarxes directes, estàtiques, dinàmiques. | Quan un paquet no arriba a la seva destinació (routing problem). |
| show spanning-tree | Estat de STP: qui és el Root Bridge, quins ports estan en Forwarding/Blocking. | Quan hi ha problemes de convergència o sospita de bucles. |
| show mac address-table | Taula CAM: MAC → port. Permet verificar on estan connectats els dispositius. | Quan una MAC no és accessible o hi ha sospita de Port Security. |
| show port-security | Resum de Port Security: MACs apreses, violations. | Quan un port s'ha posat en err-disabled. |
| show running-config | Configuració activa en memòria RAM. La configuració real en ús ara mateix. | Per verificar qualsevol configuració que s'hagi aplicat. |
| ping \[IP\] | Envia 5 paquets ICMP Echo i mostra quants han rebut resposta. | Verificar connectivitat capa 3 entre dos punts. |
| traceroute \[IP\] | Mostra tots els routers (hops) en el camí fins al destí i la latència a cadascun. | Localitzar on s'atura el trànsit en una ruta d'encaminament. |

### 7.3 Interpretar els estats de les interfícies

La comanda show ip interface brief mostra per a cada interfície dos estats: el primer és el estat físic (capa 1), el segon és el protocol (capa 2). Les combinacions possibles:

| **Estat físic** | **Estat protocol** | **Interpretació** | **Acció** |
|---|---|---|---|
| up | up | Tot correcte. La interfície funciona. | Cap |
| up | down | Problema de capa 2. El cable està connectat però no s'estableix l'encapsulació. | Verificar tipus d'encapsulació, configuració del trunk, VLAN nativa. |
| down | down | Problema de capa 1. El cable no és actiu, no hi ha senyal. | Verificar cable, port del switch, NIC del dispositiu. |
| administratively down | down | Interfície apagada manualment (shutdown). | Aplicar 'no shutdown' a la interfície. |

### 7.4 Wireshark: veient els paquets reals

Wireshark és l'analitzador de protocols de xarxa de referència. Captura les trames que passen per una interfície de xarxa i les mostra desglossades capa a capa en temps real. És imprescindible per entendre com funcionen realment els protocols i per diagnosticar problemes complexos.

Filtres bàsics de Wireshark que usaràs al projecte:

| **Filtre Wireshark** | **Mostra\...** |
|---|---|
| arp | Totes les trames ARP. Per veure qui demana quines MACs i detectar ARP spoofing. |
| bootp o dhcp | Trànsit DHCP (Discover, Offer, Request, ACK). |
| icmp | Pings. Per veure si els ICMP arriben i hi ha resposta. |
| ip.addr == 192.168.20.5 | Tot el trànsit d'una IP específica. |
| tcp.port == 80 | Connexions HTTP. |
| ip.src == 192.168.20.0/24 | Tot el trànsit originat a la VLAN 20. |
| not arp and not icmp | Filtra tot excepte ARP i ICMP (per veure el trànsit rellevant). |

