

# Unitat 3: Serveis de Configuració Automàtics de Xarxa (IPv4 / IPv6)

**Mòdul:** Serveis de Xarxa i Internet  
**Cicle:** Administració de Sistemes Informàtics en Xarxa   
**Professor:** José Maria Madroñal Oliver 

Note:
Per veure les notes de l'orador pitja S

---

## 📋 Continguts

1. **Evolució dels protocols d'assignació d'IP**
   * ARP, RARP, BOOTP i DHCP.
2. **El Protocol DHCP (IPv4)**
   * Tipus d'assignació i paràmetres.
   * Missatges DHCP i flux de treball (DORA).
   * Adreçament d'emergència: APIPA.
   * Agent Relé (DHCP Relay).
3. **Configuració automàtica de xarxa en IPv6**
   * Autoconfiguració d'enllaç local (Link-Local).
   * Protocol ICMPv6 i Neighbor Discovery (NDP).
   * SLAAC (Stateless Address Autoconfiguration).
   * DHCPv6: Stateless vs. Stateful.
4. **Eines i comandaments actuals (Linux i Windows)**

---

## 1. Evolució dels protocols d'assignació IP

---

### Ubicació dels protocols al model TCP/IP

![Protocol tcp ip](/Unitat03/img/protocol_tcp_ip.png)

note:
* **Capa d'Aplicació / Transport:** 
  * DHCP / DHCPv6 (Port UDP 67/68 per IPv4, UDP 546/547 per IPv6)
  * BOOTP (UDP 67/68)
* **Capa d'Internet / Enllaç:**
  * **IPv4:** ARP, RARP, ICMP
  * **IPv6:** ICMPv6 (substitueix ARP i integra NDP)

---

### Els predecessors de DHCP

#### **1. ARP (Address Resolution Protocol - RFC 826)**
* Resol la **@MAC** de destinació a partir d'una @IP en una xarxa local.
* Tipus de lliurament: Directe (host) o Indirecte (Router/Gateway).

#### **2. RARP (Reverse ARP - RFC 903)**
* Permetia a un host conèixer la seva @IP a partir de la seva @MAC.
* **Limitacions:** Només assignava la IP (no la màscara, ni la porta d'enllaç), requeria taules manuals al servidor i no travessava routers.

#### **3. BOOTP (Bootstrap Protocol - RFC 951)**
* Dissenyat per a equips sense disc dur (bootstrapping / PXE).
* Utilitza segments UDP (Port 67 Servidor, Port 68 Client).
* Permet carregar la imatge del SO des d'un servidor TFTP.

---

## 2. El Protocol DHCP (IPv4)

---

### Tipus d'assignacions en DHCP

* **Assignació Manual / Estàtica (Reserva):**
  * S'associa una adreça IP fixa a una adreça MAC concreta.
  * Utilitzat per a servidors, impresores de xarxa i dispositius crítics.

* **Assignació Automàtica:**
  * El servidor assigna permanentment una IP lliure de l'interval al client.

* **Assignació Dinàmica:**
  * El servidor "lluga" (*lease*) una adreça IP per un temps limitat.
  * Quan el temps expira, el client ha de demanar la renovació. Permet reoptimitzar i reutilitzar adreces IP.

Note:
Que passa si la renovació és cada setmana?

---

### Paràmetres principals de configuració

Més enllà de la IP, DHCP transmet opcions de configuració (RFC 2132):

* Adreça IP i Màscara de subxarxa
* Porta d'enllaç (*Default Gateway*)
* Servidors DNS i Nom de Domini
* Servidors de Temps (NTP)
* MTU (Maximum Transmission Unit)
* Servidors TFTP / Fitxer d'arrencada (per PXE Boot)

---

### Missatges del protocol DHCP

| Missatge | Origen | Descripció |
| :--- | :--- | :--- |
| **DHCPDISCOVER** | Client | Difusió (*Broadcast*) per trobar servidors DHCP disponibles. |
| **DHCPOFFER** | Servidor | Resposta Unicast/Broadcast amb una oferta de configuració. |
| **DHCPREQUEST** | Client | Sol·licita l'adreça oferts o demana renovar una IP existent. |
| **DHCPACK** | Servidor | Confirmació final amb els paràmetres assignats. |
| **DHCPNAK** | Servidor | Denegació (IP no vàlida o caducada). |
| **DHCPDECLINE** | Client | Notifica que la IP oferida ja està en ús a la xarxa. |
| **DHCPRELEASE** | Client | Allibera la IP voluntàriament. |
| **DHCPINFORM** | Client | Demana opcions locals (ja té la IP configurada). |

---

### Estructura del missatge DHCP (RFC 2131)

![Missatges DHCP](/Unitat03/img/missatge_DHCP.png)

 https://www.rfc-editor.org/info/rfc2131/#page-8

note:
Possar en marxa el wireshark i capturar missatge bootp

---

### Flux de funcionament: Procés DORA

```
 Client (DHCP Client)                 Servidor (DHCP Server)
        |                                       |
        | ------ DHCPDISCOVER (Broadcast) ----> |
        | <----- DHCPOFFER (Unicast/Bcast) ---- |
        |                                       |
        | ------ DHCPREQUEST (Broadcast) -----> |
        | <----- DHCPACK (Unicast/Bcast) ------ |
        |                                       |
```

* **Renovació de llicència (*Lease*):**
  * Al 50% del temps de convalidació ($T_1$), el client envia un `DHCPREQUEST` Unicast directament al servidor.
  * Si no respon, al 87.5% ($T_2$), envia un `DHCPREQUEST` en Broadcast.

---

### Adreçament d'emergència: APIPA

Si un client no troba cap servidor DHCP, el sistema assigna una adreça **APIPA** (*Automatic Private IP Addressing*):

* **Rang d'adreces:** `169.254.0.0/16`
* **Mecanisme:** Utilitza *Gratuitous ARP* per comprovar que cap altre equip utilitza la IP seleccionada a la xarxa local.
* **Avantatges:** Permet la comunicació immediata entre equips del mateix segment local.
* **Limitacions:** No té porta d'enllaç, per tant, **no permet sortir a Internet** ni comunicar-se amb altres subxarxes.

---

### Agent Relé (DHCP Relay Agent)

* Els missatges `DHCPDISCOVER` són **Broadcasts limitats (`255.255.255.255`)** i els encaminadors els bloquegen.
* Un **Agent Relé** (normalment configurat al router o switch L3):
  1. Rep el Broadcast local del client.
  2. El transforma en un missatge **Unicast** cap a la IP del servidor DHCP centralitzat.
  3. Afegeix la informació de subxarxa (*Option 82*).

```
[ Client ] --(Broadcast)--> [ Router / Agent Relé ] --(Unicast)--> [ Servidor DHCP ]
```

---

## 3. Configuració Automàtica de Xarxa en IPv6

---

### Novetats de la configuració automàtica en IPv6

IPv6 **elimina la necessitat absoluta d'un servidor DHCP tradicional** gràcies a l'autoconfiguració nativa, tot i que manté DHCPv6 per a escenaris avançats.

### Mecanismes principals:
1. **Link-Local Address:** Generació automàtica d'adreça local (`fe80::/10`).
2. **SLAAC (Stateless Address Autoconfiguration):** Autoconfiguració sense estat.
3. **DHCPv6 Stateless (Sense estat):** SLAAC per a la IP + DHCPv6 per a dades extra (ex: DNS).
4. **DHCPv6 Stateful (Amb estat):** Assignació centralitzada completa (similar a IPv4).

---

### Protocol ICMPv6 i Neighbor Discovery (NDP)

En IPv6 **no existeix ARP**. Es substitueix per **NDP** (*Neighbor Discovery Protocol*), que utilitza missatges **ICMPv6**:

* **Router Solicitation (RS - Tipus 133):** El host demana informació als routers de la xarxa.
* **Router Advertisement (RA - Tipus 134):** El router respon/anuncia els paràmetres del prefix IPv6 i de configuració.
* **Neighbor Solicitation (NS - Tipus 135) / Neighbor Advertisement (NA - Tipus 136):** Reemplacen les sol·licituds i respostes ARP.
* **DAD (Duplicate Address Detection):** Verfica que la IP IPv6 generada no estigui duplicada a la xarxa local.

---

### 1. SLAAC (Stateless Address Autoconfiguration - RFC 4862)

Permet a un host assignar-se una adreça IP Global Unicast **sense cap servidor DHCP**.

#### Com funciona?
1. El host envia un missatge ICMPv6 **Router Solicitation (RS)**.
2. El router de la xarxa respon amb un **Router Advertisement (RA)** que conté:
   * El **Prefix de xarxa** (ex: `2001:db8:1::/64`).
   * El temps de vida (*Lifetime*).
   * Els atributs de configuració (*Flags A, M, O*).
3. El host genera la segona meitat de l'adreça (Identificador d'interfície / 64 bits) mitjançant:
   * **EUI-64** (basat en la MAC) o adreces aleatòries privades (RFC 8981).
4. El host executa **DAD** per evitar duplicats.

---

### Banderes (Flags) en els missatges RA d'IPv6

El comportament del client ve determinat per les **banderes (Flags M i O)** contingudes al missatge ICMPv6 Router Advertisement del router:

| M (Managed) | O (Other) | Mode de configuració | Descripció |
| :---: | :---: | :--- | :--- |
| **0** | **0** | **SLAAC pur** | Obté Prefix i Gateway del Router. No utilitza DHCPv6. (Si suporta RDNSS RFC 8106, també obté DNS). |
| **0** | **1** | **DHCPv6 Stateless** | Obté IP i Gateway per SLAAC. Demana DNS i altres opcions a un servidor DHCPv6. |
| **1** | **X** | **DHCPv6 Stateful** | Obté TOTA la configuració (IP, DNS, etc.) directament del servidor DHCPv6. |

---

### 2. DHCPv6 Stateless vs. Stateful

#### **DHCPv6 Stateless (Sense Estat)**
* El router dóna el Prefix de xarxa i Gateway (SLAAC).
* El servidor DHCPv6 només distribueix opcions addicionals (Servidors DNS, Noms de domini, NTP).
* **El servidor no manté registre** de quines IPs s'assignen a quins clients.

#### **DHCPv6 Stateful (Amb Estat)**
* Funciona de forma equivalent al DHCPv4 tradicional.
* El servidor assigna directament l'adreça IPv6 completa i la resta de paràmetres.
* **El servidor guarda un registre (*Lease Table*)** amb el control d'adreces assignades.
* Utilitza el **DUID** (*DHCP Unique Identifier*) en comptes de la MAC.

---

### Recompilació: Esquema de Configuració IPv6

```
                     [ Client IPv6 s'encén ]
                                |
                    Envia ICMPv6 RS (Solicitation)
                                |
                   Rep ICMPv6 RA (Advertisement)
                                |
         +----------------------+----------------------+
         | (M=0, O=0)           | (M=0, O=1)           | (M=1)
         v                      v                      v
    [ SLAAC Pur ]       [ DHCPv6 Stateless ]   [ DHCPv6 Stateful ]
   - IP per SLAAC       - IP per SLAAC         - IP per DHCPv6
   - Gateway per RA     - Gateway per RA       - Gateway per RA
                        - DNS per DHCPv6       - DNS per DHCPv6
```

---

## 4. Eines i Comandaments Actuals

---

### Comandaments de Diagnòstic i Gestió

#### **En sistemes Microsoft Windows:**
* `ipconfig /all` : Mostra la configuració completa (IPv4/IPv6, DUID, Lease).
* `ipconfig /release` / `ipconfig /renew` : Allibera i renova llicències IPv4.
* `ipconfig /release6` / `ipconfig /renew6` : Gestió de llicències IPv6.
* `netsh interface ipv6 show addresses` : Mostra les adreces IPv6 de les interfícies.

#### **En sistemes Linux (Eines modernes `iproute2`):**
* `ip addr show` / `ip -6 addr show` : Visualitza les adreces IP/IPv6.
* `ip route` / `ip -6 route` : Mostra taules d'encaminament.
* `dhclient -r` / `dhclient` : Allibera i sol·licita IP via el client `dhclient`.
* `systemd-networkctl` / `nmcli` : Gestió moderna en distribucions basades en NetworkManager o Systemd.

---

# Gràcies per la vostra atenció!

**Preguntes i Debat:**
* Quins avantatges té SLAAC respecte a un servidor DHCP tradicional?
* En quins entorns empresarials continua sent imprescindible utilitzar **DHCPv6 Stateful**?