## Netzwerkdokumentation – Projekt 1

### 1) Ausgangsdaten

| Parameter | Wert |
|---|---|
| Projekt Nr. | 1 |
| Abt. Office (Verwaltung) | 7 Hosts |
| Abt. Produktion | 22 Hosts |
| Linz | 4 Endgeräte |
| Graz | 40 Endgeräte |
| Interner IP-Bereich | `10.10.0.0/20` |
| Öffentlicher IP-Bereich | `199.121.123.128/?` |

**Wichtige Anpassungen:**
- Graz ist mit 40 Hosts deutlich größer.
- Linz ist mit 4 Hosts sehr klein.
- Verwaltung benötigt nur ein kleines Subnetz.
- Basisnetz intern: `10.10.0.0/20`.

---

### 2) VLSM-Planung (mit ~20% Puffer)

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

---

### 3) Interne Subnetze (VLSM-Zuteilung)

#### Wien – Hauptsitz

| Netz | Netzadresse | Subnetzmaske | Nutzbare Hosts | Gateway | Broadcast |
|---|---|---|---|---|---|
| Produktion | `10.10.0.0/27` | `255.255.255.224` | `10.10.0.1 – 10.10.0.30` (30) | `10.10.0.1` | `10.10.0.31` |
| Verwaltung | `10.10.0.32/28` | `255.255.255.240` | `10.10.0.33 – 10.10.0.46` (14) | `10.10.0.33` | `10.10.0.47` |
| Server | `10.10.0.48/29` | `255.255.255.248` | `10.10.0.49 – 10.10.0.54` (6) | `10.10.0.49` | `10.10.0.55` |
| Management | `10.10.0.56/29` | `255.255.255.248` | `10.10.0.57 – 10.10.0.62` (6) | `10.10.0.57` | `10.10.0.63` |

#### Linz – Niederlassung

| Netz | Netzadresse | Subnetzmaske | Nutzbare Hosts | Gateway | Broadcast |
|---|---|---|---|---|---|
| Linz LAN | `10.10.0.64/29` | `255.255.255.248` | `10.10.0.65 – 10.10.0.70` (6) | `10.10.0.65` | `10.10.0.71` |

#### Graz – Niederlassung

| Netz | Netzadresse | Subnetzmaske | Nutzbare Hosts | Gateway | Broadcast |
|---|---|---|---|---|---|
| Graz LAN | `10.10.0.128/26` | `255.255.255.192` | `10.10.0.129 – 10.10.0.190` (62) | `10.10.0.129` | `10.10.0.191` |

---

### 4) WAN-Verbindungen (Leased Lines)

| Strecke | Netzadresse | Subnetzmaske | Wien-Router | Gegenstelle | Broadcast |
|---|---|---|---|---|---|
| Wien ↔ Linz | `10.10.0.72/30` | `255.255.255.252` | `10.10.0.73` | `10.10.0.74` | `10.10.0.75` |
| Wien ↔ Graz | `10.10.0.76/30` | `255.255.255.252` | `10.10.0.77` | `10.10.0.78` | `10.10.0.79` |

---

### 5) Öffentliche IP-Adressen (ISP)

Für Border-Router, Webserver und Reserve werden mindestens 3 öffentliche IPs benötigt.  
Daher: **`199.121.123.128/29`** (6 nutzbare Hosts).

| Verwendung | IP-Adresse |
|---|---|
| Netzadresse | `199.121.123.128/29` |
| Border Router (outside) | `199.121.123.129` |
| Web-Server (öffentlich, fix) | `199.121.123.130` |
| NAT-Pool Reserve | `199.121.123.131 – 199.121.123.134` |
| Broadcast | `199.121.123.135` |

---

### 6) Gesamtübersicht

- `10.10.0.0/27` → Wien Produktion (30 Hosts)  
- `10.10.0.32/28` → Wien Verwaltung (14 Hosts)  
- `10.10.0.48/29` → Wien Server (6 Hosts)  
- `10.10.0.56/29` → Wien Management (6 Hosts)  
- `10.10.0.64/29` → Linz LAN (6 Hosts)  
- `10.10.0.72/30` → WAN Wien–Linz (2 Hosts)  
- `10.10.0.76/30` → WAN Wien–Graz (2 Hosts)  
- `10.10.0.128/26` → Graz LAN (62 Hosts)  
- `199.121.123.128/29` → Öffentlich (ISP)

---

### 7) Verwendete Geräte im Packet Tracer

#### Netzwerkgeräte (aktiv)

| Gerätetyp | Name im Diagramm | Funktion |
|---|---|---|
| Router | Border-Router | Internet-Anbindung, NAT/PAT |
| Router | HQ Wien | Hauptrouter, Standortkopplung |
| Router | Branch Linz | Router Niederlassung Linz |
| Router | Branch Graz | Router Niederlassung Graz |
| Switch | links (Produktion/Office) | Access-Switch Produktion |
| Switch | rechts (Produktion/Office) | Access-Switch Verwaltung |

#### Endgeräte

| Gerätetyp | Name im Diagramm | Bereich |
|---|---|---|
| PC | Production PC1 | Produktion Wien |
| PC | Office PC2 | Verwaltung Wien |
| PC | Production PC3 | Produktion Wien |
| PC | Office PC4 | Verwaltung Wien |
| PC | NMC | Network Management Center |
| Server | WEB-Server | Öffentlich |
| Server | File-Server | Intern Wien |

#### Sonstige Elemente

| Element | Bedeutung |
|---|---|
| Internet-Cloud | ISP / Internet |
| Leased Line (serielle Verbindungen) | WAN Wien↔Linz, Wien↔Graz |

---

### 8) Empfohlene Cisco-Hardware

#### Switch
**Empfohlen:** `Catalyst 2960-24TT`  
Begründung: VLANs, Trunking, Port-Security, STP, 24 FE-Ports + Uplink-Ports.

#### Router
**Empfohlen:** `Cisco 2911`

**Modulbedarf für serielle Leitungen:**
- HQ Wien: `2911 + WIC-2T` (2 serielle WAN-Links)
- Branch Linz: `2911 + WIC-1T` (1 serieller WAN-Link)
- Branch Graz: `2911 + WIC-1T` (1 serieller WAN-Link)
- Border Router: `2911` (ohne serielles Modul ausreichend)

---

### 9) Leitungen / Kabeltypen

| Verbindung | Kabeltyp |
|---|---|
| PC → Switch | Copper Straight-Through |
| Server → Switch | Copper Straight-Through |
| Switch → Router | Copper Straight-Through |
| Border Router → WEB-Server | Copper Straight-Through |
| Border Router → Internet-Cloud | Copper Straight-Through |
| HQ Wien → Border Router | Copper Straight-Through |
| Switch ↔ Switch | Copper Cross-Over |
| HQ Wien ↔ Branch Linz | Serial DCE/DTE |
| HQ Wien ↔ Branch Graz | Serial DCE/DTE |

**Hinweis Serial DCE/DTE:**  
Am DCE-Ende muss eine `clock rate` gesetzt werden (z. B. `64000`).

---

### 10) Noch zu ergänzen (laut Aufgabenfokus)

- Switch bei Branch Linz
- Switch bei Branch Graz
- Endgeräte gemäß Bedarf (Linz: 4, Graz: 40)
- Optional: Firewall zwischen Border-Router und Internet
- Optional: WLAN-Access-Point bei Laptop-Einsatz