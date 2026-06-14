# Netzwerkdokumentation – Projekt 1
**Firma:** FRB Software Entwicklung GmbH  
**Typ:** Firmennetzwerk mit Leased Lines (VPN)  
**Erstellt:** April 2026

> **Hinweis zur Bebilderung:**
> Lege alle Screenshots im Ordner `screenshots/` ab.  
> Beispiel: `screenshots/SS-01_Topologie_2026-05-20.png`

---

## 1) Ausgangsdaten

| Parameter | Wert |
|---|---|
| Projekt Nr. | 1 |
| Abt. Produktion | 22 Hosts |
| Abt. Verwaltung (Office) | 7 Hosts |
| Linz | 4 Endgeräte |
| Graz | 40 Endgeräte |
| Interner IP-Bereich | `10.10.0.0/20` |
| Öffentlicher IP-Bereich | `199.121.123.128/29` |
| Öffentliche IP Web-Server | `199.121.123.130` |

---

## 2) VLSM-Planung (mit ~20% Puffer)

| Abteilung / Standort | Hosts (real) | Mit Puffer (~20%) | Benötigte Größe | Präfix |
|---|---:|---:|---:|---|
| Graz | 40 | 48 | 64 Adressen | `/26` (62 Hosts) |
| Wien – Produktion | 22 | 27 | 32 Adressen | `/27` (30 Hosts) |
| Wien – Verwaltung | 7 | 9 | 16 Adressen | `/28` (14 Hosts) |
| Linz | 4 | 5 | 8 Adressen | `/29` (6 Hosts) |
| Wien – Server | 2–3 | 4 | 8 Adressen | `/29` (6 Hosts) |
| Wien – Management | 2–3 | 4 | 8 Adressen | `/29` (6 Hosts) |
| WAN Wien–Linz | 2 | 2 | 4 Adressen | `/30` (2 Hosts) |
| WAN Wien–Graz | 2 | 2 | 4 Adressen | `/30` (2 Hosts) |
| Öffentlich (ISP) | 3 | 4 | 8 Adressen | `/29` (6 Hosts) |

---

## 3) Interne Subnetze (VLSM-Zuteilung)

### Wien – Hauptsitz

| Netz | Netzadresse | Subnetzmaske | Nutzbare Hosts | Gateway | Broadcast |
|---|---|---|---|---|---|
| Produktion | `10.10.0.0/27` | `255.255.255.224` | `10.10.0.1 – 10.10.0.30` (30) | `10.10.0.1` | `10.10.0.31` |
| Verwaltung | `10.10.0.32/28` | `255.255.255.240` | `10.10.0.33 – 10.10.0.46` (14) | `10.10.0.33` | `10.10.0.47` |
| Server | `10.10.0.48/29` | `255.255.255.248` | `10.10.0.49 – 10.10.0.54` (6) | `10.10.0.49` | `10.10.0.55` |
| Management | `10.10.0.56/29` | `255.255.255.248` | `10.10.0.57 – 10.10.0.62` (6) | `10.10.0.57` | `10.10.0.63` |

### Linz – Niederlassung

| Netz | Netzadresse | Subnetzmaske | Nutzbare Hosts | Gateway | Broadcast |
|---|---|---|---|---|---|
| Linz LAN | `10.10.0.64/29` | `255.255.255.248` | `10.10.0.65 – 10.10.0.70` (6) | `10.10.0.65` | `10.10.0.71` |

### Graz – Niederlassung

| Netz | Netzadresse | Subnetzmaske | Nutzbare Hosts | Gateway | Broadcast |
|---|---|---|---|---|---|
| Graz LAN | `10.10.0.128/26` | `255.255.255.192` | `10.10.0.129 – 10.10.0.190` (62) | `10.10.0.129` | `10.10.0.191` |

---

## 4) WAN-Verbindungen (Leased Lines)

| Strecke | Netzadresse | Subnetzmaske | Wien-Router | Gegenstelle | Broadcast |
|---|---|---|---|---|---|
| Wien ↔ Linz | `10.10.0.72/30` | `255.255.255.252` | `10.10.0.73` | `10.10.0.74` | `10.10.0.75` |
| Wien ↔ Graz | `10.10.0.76/30` | `255.255.255.252` | `10.10.0.77` | `10.10.0.78` | `10.10.0.79` |

---

## 5) Öffentliche IP-Adressen (ISP)

Für Border-Router, Webserver und Reserve werden mindestens 3 öffentliche IPs benötigt.  
Daher: **`199.121.123.128/29`** (6 nutzbare Hosts).

| Verwendung | IP-Adresse |
|---|---|
| Netzadresse | `199.121.123.128` |
| Border Router (outside) | `199.121.123.129` |
| Web-Server (öffentlich, fix) | `199.121.123.130` |
| NAT-Pool Reserve | `199.121.123.131 – 199.121.123.134` |
| Broadcast | `199.121.123.135` |

---

## 6) VLAN-Schema

### VLAN Übersicht

| VLAN-ID | Name | Subnetz | Standort |
|---|---|---|---|
| VLAN 10 | Produktion | `10.10.0.0/27` | Wien |
| VLAN 20 | Verwaltung | `10.10.0.32/28` | Wien |
| VLAN 30 | Server | `10.10.0.48/29` | Wien |
| VLAN 40 | Management | `10.10.0.56/29` | Wien |
| VLAN 50 | Linz_LAN | `10.10.0.64/29` | Linz |
| VLAN 60 | Graz_LAN | `10.10.0.128/26` | Graz |

---

### Switch Wien Links (Produktion & Verwaltung)

| Port | Gerät | VLAN | Port-Typ |
|---|---|---|---|
| Fa 0/1 | Production PC1 | VLAN 10 | Access |
| Fa 0/2 | Production PC3 | VLAN 10 | Access |
| Fa 0/3 | Office PC2 | VLAN 20 | Access |
| Fa 0/4 | Office PC4 | VLAN 20 | Access |
| Fa 0/24 | → Switch Rechts | Trunk | Trunk |
| Gi 0/1 | → HQ Wien Router | Trunk | Trunk |

### Switch Wien Rechts (Server & Management)

| Port | Gerät | VLAN | Port-Typ |
|---|---|---|---|
| Fa 0/1 | File-Server | VLAN 30 | Access |
| Fa 0/2 | Web-Server | VLAN 30 | Access |
| Fa 0/3 | NMC (Management PC) | VLAN 40 | Access |
| Fa 0/24 | → Switch Links | Trunk | Trunk |
| Gi 0/1 | → HQ Wien Router | Trunk | Trunk |

### Switch Linz

| Port | Gerät | VLAN | Port-Typ |
|---|---|---|---|
| Fa 0/1 | Linz PC1 | VLAN 50 | Access |
| Fa 0/2 | Linz PC2 | VLAN 50 | Access |
| Fa 0/3 | Linz PC3 | VLAN 50 | Access |
| Fa 0/4 | Linz PC4 | VLAN 50 | Access |
| Gi 0/1 | → Branch Linz Router | Trunk | Trunk |

### Switch Graz

| Port | Gerät | VLAN | Port-Typ |
|---|---|---|---|
| Fa 0/1 – Fa 0/24 | Graz PC1–PC24 | VLAN 60 | Access |
| Fa 0/1 – Fa 0/16 (Switch 2) | Graz PC25–PC40 | VLAN 60 | Access |
| Gi 0/1 | → Branch Graz Router | Trunk | Trunk |
| Gi 0/2 | → Switch Graz 2 | Trunk | Trunk |

> ⚠️ Graz benötigt 2x Catalyst 2960-24TT (40 Endgeräte > 24 Ports)

---

## 7) Verwendete Geräte im Packet Tracer

### Netzwerkgeräte (aktiv)

| Gerätetyp | Modell | Name | Funktion |
|---|---|---|---|
| Router | Cisco 2911 | Border-Router | Internet-Anbindung, NAT/PAT |
| Router | Cisco 2911 + WIC-2T | HQ Wien | Hauptrouter, Inter-VLAN, WAN |
| Router | Cisco 2911 + WIC-1T | Branch Linz | Router Niederlassung Linz |
| Router | Cisco 2911 + WIC-1T | Branch Graz | Router Niederlassung Graz |
| Switch | Catalyst 2960-24TT | SW-Wien-Links | Access-Switch Produktion/Verwaltung |
| Switch | Catalyst 2960-24TT | SW-Wien-Rechts | Access-Switch Server/Management |
| Switch | Catalyst 2960-24TT | SW-Linz | Access-Switch Linz |
| Switch | Catalyst 2960-24TT | SW-Graz-1 | Access-Switch Graz (PC 1–24) |
| Switch | Catalyst 2960-24TT | SW-Graz-2 | Access-Switch Graz (PC 25–40) |

### Endgeräte

| Gerätetyp | Name | Bereich | IP-Bezug |
|---|---|---|---|
| PC | Production PC1 | VLAN 10 – Wien Produktion | DHCP |
| PC | Production PC3 | VLAN 10 – Wien Produktion | DHCP |
| PC | Office PC2 | VLAN 20 – Wien Verwaltung | DHCP |
| PC | Office PC4 | VLAN 20 – Wien Verwaltung | DHCP |
| PC | NMC | VLAN 40 – Management | Fixe IP |
| Server | File-Server | VLAN 30 – Wien Server | Fixe IP (intern) |
| Server | Web-Server | VLAN 30 / öffentlich | `199.121.123.130` (fix) |
| PC | Linz PC1–PC4 | VLAN 50 – Linz LAN | DHCP |
| PC | Graz PC1–PC40 | VLAN 60 – Graz LAN | DHCP |

---

## 8) Kabeltypen

| Verbindung | Kabeltyp |
|---|---|
| PC → Switch | Copper Straight-Through |
| Server → Switch | Copper Straight-Through |
| Switch → Router | Copper Straight-Through |
| Switch ↔ Switch | Copper Cross-Over |
| Border Router → Web-Server | Copper Straight-Through |
| Border Router → Internet-Cloud | Copper Straight-Through |
| HQ Wien → Border Router | Copper Straight-Through |
| HQ Wien ↔ Branch Linz | Serial DCE/DTE |
| HQ Wien ↔ Branch Graz | Serial DCE/DTE |

> **Hinweis Serial DCE/DTE:** Am DCE-Ende muss `clock rate 64000` gesetzt werden.

---

## 9) Sicherheitskonzept

| Maßnahme | Wo | Beschreibung |
|---|---|---|
| NAT/PAT | Border Router | Alle internen PCs → eine öffentliche IP |
| Statisches NAT | Border Router | Web-Server fix auf `199.121.123.130` |
| File-Server isoliert | Kein NAT | Nur intern über VLAN 30 erreichbar |
| Port-Security | Alle Access Switches | Max. 1 MAC-Adresse pro Port |
| NMC Remote-Management | VLAN 40 | SSH-Zugriff auf alle Router/Switches |
| VLAN-Trennung | Alle Switches | Abteilungen logisch isoliert |

---

## 10) Umsetzungsstatus (Packet Tracer)

- [x] VLANs auf allen Switches erstellt
- [x] Access Ports zugewiesen
- [x] Trunk Ports konfiguriert
- [x] Subinterfaces am HQ Wien Router (Inter-VLAN Routing)
- [x] DHCP-Pools auf HQ Wien Router
- [x] ACLs für VLAN-Zugriffe umgesetzt
- [x] Testen: Ping, ACL-Verhalten, Gateway-Erreichbarkeit

---

## 11) Bilddokumentation (Screenshots)

> Füge die Bilddateien in `screenshots/` ein.  
> Wenn ein Bild noch fehlt, bleibt der Platzhalter einfach stehen.

### SS-01 – Topologie vor Konfiguration
![SS-01 Topologie](screenshots/SS-01_Topologie.png)

### SS-02 – Router `show ip interface brief`
![SS-02 Router show ip interface brief](screenshots/SS-02_Router_show_ip_interface_brief_2026-05-20.png)

### SS-03 – Router `show ip dhcp pool`
![SS-03 Router show ip dhcp pool](screenshots/SS-03_DHCP_Pool.png)

### SS-04 – Switch Wien VLAN + Trunk
![SS-04 Switch Wien VLAN Trunk](<screenshots/SS-02_Switch_show_vlan_brief_show_interfaces_trunk_2026-05-20.png>)

### SS-05 – Switch1 VLAN + Trunk
![SS-05 Switch1 VLAN Trunk](<screenshots/SS-02_Switch_show_vlan_brief_show_interfaces_trunk_sw2_2026-05-20.png>)

### SS-06 – NMC IP-Konfiguration
![SS-06 NMC + FileServer IP Config](screenshots/NMC_PC_FileServerCONFIG.png)

### SS-07 – FileServer IP + Services
![SS-07 FileServer HTTP Service](<screenshots/Bildschirmfoto vom 2026-06-13 21-03-54.png>)

### SS-08 – ACL-Nachweis (`show access-lists`)
![SS-08 ACL Nachweis](<screenshots/Bildschirmfoto vom 2026-06-13 21-08-24.png>)

### SS-09 – Erlaubter Ping
![SS-09 Erlaubter Ping Production-PC1](screenshots/ping_nachweis.png)

### SS-10 – Blockierter Ping
![SS-10 Blockierter Ping Office-PC2](screenshots/ping_nachweis2.png)