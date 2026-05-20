# 🎯 KONFIGURATIONS-LEITFADEN – Packet Tracer Netzwerk
**FRB Software Entwicklung GmbH – Wien Hauptsitz**  
**Status:** Produktionsreif | Letzte Aktualisierung: 20. Mai 2026

---

## 📌 SCHNELLEINSTIEG

**Gesamtdauer:** ca. 4-5 Stunden | **Schwierigkeit:** Fortgeschrittene  
**Ziel:** Vollständiges Firmennetzwerk mit VLANs, Routing, DHCP und Sicherheits-ACLs

### Was wird konfiguriert?
- 1x Router (2911) mit 4 VLANs
- 2x Switches (2960) mit Trunk & Access Ports
- 5x PCs + 2x Server
- DHCP-Pools und Netzwerk-Routing
- **Access Control Lists (ACLs) zur Netzwerk-Sicherheit**

## 🧭 Projekt-Roadmap in 7 Phasen

| Phase | Ziel | Ergebnis (Done-Kriterium) |
|---|---|---|
| 1 | Ausgangsstand prüfen | Topologie, Kabel und Router-Basis sind verifiziert |
| 2 | Router konfigurieren | Subinterfaces + DHCP laufen korrekt |
| 3 | Switch Wien-Links | VLANs, Access-Ports und Trunks sind aktiv |
| 4 | Switch1 konfigurieren | VLANs, Access-Ports und Trunk sind aktiv |
| 5 | Endgeräte konfigurieren | PCs per DHCP, NMC/Server statisch korrekt |
| 6 | ACL-Sicherheit umsetzen | Zugriffsregeln sind gesetzt und auf Interfaces aktiv |
| 7 | Endtests & Abnahme | Erlaubte/verbotene Pings verhalten sich wie geplant |

## 🗂️ Live-Doku (ab jetzt so arbeiten)

> Ab jetzt gilt: **Jede Phase sofort dokumentieren**, nicht erst am Ende.

### A) Fortschritts-Board (vor jeder Session aktualisieren)

| Phase | Status (Nicht gestartet / In Arbeit / Fertig) | Datum | Start | Ende | Nachweis vorhanden? |
|---|---|---|---|---|---|
| Phase 1 | Fertig | 20.05.2026 |  |  | ✅ |
| Phase 2 | Fertig | 20.05.2026 |  |  | ✅ |
| Phase 3 | In Arbeit (Trunk ok, Access-Ports prüfen) | 20.05.2026 |  |  | ✅ |
| Phase 4 | Fertig | 20.05.2026 |  |  | ✅ |
| Phase 5 | In Arbeit (statische IPs korrigieren) | 20.05.2026 |  |  | ✅ |
| Phase 6 | Nicht gestartet (ACLs fehlen) | 20.05.2026 |  |  | ⬜ |
| Phase 7 | Nicht gestartet | 20.05.2026 |  |  | ⬜ |

### D) IST-Stand aus Nachweisen (20.05.2026)

#### Bereits korrekt umgesetzt
- HQ-Router: `show ip interface brief` zeigt Subinterfaces `.10/.20/.30/.40` **up/up** mit korrekten Gateway-IPs.
- DHCP: Pools für VLAN 10/20/30/40 vorhanden, Excluded-Adressen gesetzt.
- Switch1 (Wien rechts): VLAN-Zuordnung und Trunk `Fa0/24` vorhanden.

#### Offen / zu korrigieren
1. **NMC IP-Konflikt**
   - Aktuell: `10.10.0.57` (gleich wie Gateway)
   - Soll: `10.10.0.58 / 255.255.255.248`, Gateway `10.10.0.57`
2. **FileServer IP-Konflikt**
   - Aktuell: `10.10.0.49` (gleich wie Gateway)
   - Soll: `10.10.0.50 / 255.255.255.248`, Gateway `10.10.0.49`
3. **ACLs noch nicht konfiguriert**
   - `show access-lists` ist leer.
4. **Switch Wien-Links Access-Ports prüfen**
   - Trunk ist sichtbar; bitte prüfen, ob `Fa0/1 -> VLAN10` und `Fa0/3 -> VLAN20` gesetzt sind.

#### Nächste Schritte (Reihenfolge)
1. Phase 5 fertig machen: NMC + FileServer IPs korrigieren, Screenshot SS-06/SS-07 aktualisieren.
2. Phase 3 kurz verifizieren/fixen: `show vlan brief` auf Switch Wien-Links; ggf. Access-Ports setzen.
3. Phase 6 durchführen: ACL 101/102/103 erstellen + auf `Gi0/0.10/.20/.40` inbound anwenden.
4. Phase 7 Tests durchführen: 1 erlaubter + 1 blockierter Ping (SS-09/SS-10).

### B) Protokoll-Vorlage (pro Phase 1x ausfüllen)

```markdown
#### Protokolleintrag – Phase X
- Datum:
- Startzeit / Endzeit:
- Bearbeiteter Bereich (Gerät/Interface):
- Durchgeführte Befehle (nur die wirklich verwendeten):
- Verifikation (show/ping/tracert):
- Ergebnis (OK/Nicht OK + warum):
- Probleme & Lösung:
- Nächster Schritt:
```

### C) Screenshot-Plan (Pflicht für Doku)

| Screenshot-ID | Wann aufnehmen | Was MUSS sichtbar sein |
|---|---|---|
| SS-01 | Vor Start Phase 2 | Gesamt-Topologie in Packet Tracer |
| SS-02 | Nach Phase 2 | `show ip interface brief` am HQ-Router |
| SS-03 | Nach Phase 2 | `show ip dhcp pool` am HQ-Router |
| SS-04 | Nach Phase 3 | `show vlan brief` + `show interfaces trunk` (Switch Wien-Links) |
| SS-05 | Nach Phase 4 | `show vlan brief` + `show interfaces trunk` (Switch1) |
| SS-06 | Nach Phase 5 | IP-Config NMC (statisch) |
| SS-07 | Nach Phase 5 | IP-Config File-Server + aktivierte Services |
| SS-08 | Nach Phase 6 | `show access-lists` + `show ip interface gi0/0.10` |
| SS-09 | Nach Phase 7 | Erfolgreicher Ping (erlaubt) |
| SS-10 | Nach Phase 7 | Fehlgeschlagener Ping (blockiert) |

> Tipp für Abgabe: Dateinamen wie `SS-04_SwitchWien_Trunk_2026-05-20.png` verwenden.

---

# PHASE 0: VORBEREITUNG & GRUNDLAGEN

## 0.1 Netzwerk-Schema (Überblick)

```
┌─────────────────────────────────────────────────────────┐
│           WIEN HAUPTSITZ (10.10.0.0/20)                │
│                                                         │
│  LINKS (Produktion & Verwaltung):                      │
│  ┌────────────────────────────────┐                    │
│  │ Production-PC1 ─┐              │                    │
│  │ Office-PC2     ├─→ Switch      │                    │
│  │                │  Wien-Links   │──Trunk──┐          │
│  └────────────────┘               │         │          │
│                           Router ←─┘         │ Cross    │
│                         (VLANs 10-40)        │ Over     │
│  RECHTS (Server & Management):               │          │
│  ┌────────────────────────────────┐    ┌────┘          │
│  │ Production-PC3 ─┐              │    │               │
│  │ Office-PC4     ├─→ Switch1 ←───┘   │               │
│  │ NMC (Mgmt)     │                   │               │
│  │ File-Server    │                   │               │
│  └────────────────┘                   │               │
│                                       │               │
└───────────────────────────────────────┴───────────────┘
```

## 0.2 VLAN-Plan (Alle 4 VLANs)

| VLAN-ID | Name | Subnetz | Gateway | Hosts | Typ |
|---------|------|---------|---------|-------|-----|
| 10 | Produktion | 10.10.0.0/27 | 10.10.0.1 | 30 | Access |
| 20 | Verwaltung | 10.10.0.32/28 | 10.10.0.33 | 14 | Access |
| 30 | Server | 10.10.0.48/29 | 10.10.0.49 | 6 | Access |
| 40 | Management | 10.10.0.56/29 | 10.10.0.57 | 6 | Access |

## 0.3 Geräte & Verbindungen

| Gerät | Modell | Ports | Interface |
|-------|--------|-------|-----------|
| Router | Cisco 2911 | 2x Gi 0/0-1 | Gi 0/0 = Trunk zu Switches |
| Switch Wien-Links | 2960 | 24x Fa, 2x Gi | Fa 0/1-24, Gi 0/1-2 |
| Switch1 | 2960 | 24x Fa, 2x Gi | Fa 0/1-24, Gi 0/1-2 |

---

# PHASE 1: AUSGANGSSTANDSPRÜFUNG (30 Min)

### 📸 Doku-Hinweis Phase 1
- **Screenshot jetzt:** SS-01 (Gesamt-Topologie)
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 1"

## 1.1 Topologie in Packet Tracer überprüfen

**Schritt 1: Packet Tracer öffnen**
1. Öffne `Projekt1.pkt`
2. Klicke auf **Logical** Tab (oben rechts) für Übersicht
3. Verifiziere alle Geräte sind vorhanden:
   - ✅ 1 Router (2911)
   - ✅ 2 Switches (2960)
   - ✅ 5 PCs
   - ✅ 2 Server

**Schritt 2: Kabelverbindungen inspizieren**
1. Öffne **Physical** Tab
2. Überprüfe folgende Verbindungen:
   - [ ] Router zu Switch Wien-Links: Gi 0/0 zu Gi 0/1 (Straight-Through)
   - [ ] Switch Wien-Links zu Switch1: Fa 0/24 zu Fa 0/24 (Cross-Over)
   - [ ] Alle PCs zu ihren Switches (Straight-Through)

**Wenn OK:** ✅ Weiter zu 1.2

## 1.2 Router Hardware überprüfen

**Schritt: Im Packet Tracer CLI prüfen**

1. Klicke auf **Router 2911** → **CLI** öffnen
2. Gib ein:
```bash
Router# show ip interface brief
```

**Prüfe Ausgabe:**
- ✅ `Gi 0/0` existiert
- ✅ `Gi 0/0` Status "UP"
- ✅ Keine `Fa 0/0` (Router hat nur Gi Ports)

**Wenn Fehler (z.B. "Gi 0/0 down"):**
- Überprüfe physikalische Kabelverbindung
- Warte 10 Sekunden, check nochmal

**Wenn OK:** ✅ Weiter zu 1.3

## 1.3 Konfigurationsstand dokumentieren

**Schritt: In diesem Dokument notieren**

Erstelle eine Markierung unten in dieser Datei:
```markdown
### Mein Konfigurationsstand (ausfüllen!)

**Session 1 - Status:**
- Geräte überprüft: JA / NEIN
- Kabel verbunden: JA / NEIN
- Router antwortet auf CLI: JA / NEIN
- Start-Datum: [DATUM]
```

**Wenn alle 3 JA:** → Speichere die Datei und starte mit PHASE 2

---

# PHASE 2: ROUTER-KONFIGURATION (45 Min)

### 📸 Doku-Hinweis Phase 2
- **Screenshots nach Abschluss:** SS-02 und SS-03
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 2“

## 2.1 Subinterface für VLAN 10 (Produktion)

**Schritt 1: CLI öffnen & configure mode**
```bash
Router# enable
Router# configure terminal
Router(config)#
```

**Schritt 2: Gi 0/0 aktivieren (Falls nicht aktiv)**
```bash
Router(config)# interface gi 0/0
Router(config-if)# no shutdown
Router(config-if)# exit
```

**Schritt 3: Subinterface gi 0/0.10 erstellen**
```bash
Router(config)# interface gi 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 10.10.0.1 255.255.255.224
Router(config-subif)# no shutdown
Router(config-subif)# exit
```

**Überprüfung:**
```bash
Router(config)# do show ip interface brief
```
**Sollte zeigen:** `Gi0/0.10    10.10.0.1        YES  up       UP`

## 2.2 Subinterface für VLAN 20 (Verwaltung)

```bash
Router(config)# interface gi 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 10.10.0.33 255.255.255.240
Router(config-subif)# no shutdown
Router(config-subif)# exit
```

## 2.3 Subinterface für VLAN 30 (Server)

```bash
Router(config)# interface gi 0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 10.10.0.49 255.255.255.248
Router(config-subif)# no shutdown
Router(config-subif)# exit
```

## 2.4 Subinterface für VLAN 40 (Management)

```bash
Router(config)# interface gi 0/0.40
Router(config-subif)# encapsulation dot1Q 40
Router(config-subif)# ip address 10.10.0.57 255.255.255.248
Router(config-subif)# no shutdown
Router(config-subif)# exit
```

## 2.5 DHCP ausgeschlossene Adressen (WICHTIG!)

```bash
Router(config)# ip dhcp excluded-address 10.10.0.1
Router(config)# ip dhcp excluded-address 10.10.0.33
Router(config)# ip dhcp excluded-address 10.10.0.49
Router(config)# ip dhcp excluded-address 10.10.0.57
```

## 2.6 DHCP-Pool für VLAN 10

```bash
Router(config)# ip dhcp pool VLAN10_Production
Router(dhcp-config)# network 10.10.0.0 255.255.255.224
Router(dhcp-config)# default-router 10.10.0.1
Router(dhcp-config)# exit
```

## 2.7 DHCP-Pool für VLAN 20

```bash
Router(config)# ip dhcp pool VLAN20_Office
Router(dhcp-config)# network 10.10.0.32 255.255.255.240
Router(dhcp-config)# default-router 10.10.0.33
Router(dhcp-config)# exit
```

## 2.8 DHCP-Pool für VLAN 30

```bash
Router(config)# ip dhcp pool VLAN30_Server
Router(dhcp-config)# network 10.10.0.48 255.255.255.248
Router(dhcp-config)# default-router 10.10.0.49
Router(dhcp-config)# exit
```

## 2.9 DHCP-Pool für VLAN 40

```bash
Router(config)# ip dhcp pool VLAN40_Management
Router(dhcp-config)# network 10.10.0.56 255.255.255.248
Router(dhcp-config)# default-router 10.10.0.57
Router(dhcp-config)# exit
```

## 2.10 Konfiguration speichern & überprüfen

```bash
Router(config)# end
Router# show ip interface brief
Router# show ip dhcp pool
```

**Prüf-Checkliste nach Phase 2:**
- [ ] 4 Subinterfaces sichtbar (gi 0/0.10, 20, 30, 40)
- [ ] Alle mit IP-Adressen konfiguriert
- [ ] Alle mit "UP" Status
- [ ] 4 DHCP-Pools angelegt
- [ ] Gateway-IPs sind excluded

**Wenn OK:** ✅ Weiter zu PHASE 3

---

# PHASE 3: SWITCH WIEN-LINKS KONFIGURATION (45 Min)

### 📸 Doku-Hinweis Phase 3
- **Screenshot nach Abschluss:** SS-04
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 3“

## 3.1 Zu Switch Wien-Links verbinden

**Schritt:** Klicke auf den **Switch Wien-Links** → **CLI** öffnen

## 3.2 VLANs erstellen

```bash
Switch# enable
Switch# configure terminal

Switch(config)# vlan 10
Switch(config-vlan)# name Production
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name Office
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name Server
Switch(config-vlan)# exit

Switch(config)# vlan 40
Switch(config-vlan)# name Management
Switch(config-vlan)# exit
```

**Überprüfung:**
```bash
Switch(config)# do show vlan brief
```
**Sollte 4 VLANs zeigen (10, 20, 30, 40)**

## 3.3 Access Ports: Fa 0/1 (Production-PC1) → VLAN 10

```bash
Switch(config)# interface fa 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 3.4 Access Ports: Fa 0/3 (Office-PC2) → VLAN 20

```bash
Switch(config)# interface fa 0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 3.5 Trunk-Port: Fa 0/24 (zu Switch1)

```bash
Switch(config)# interface fa 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,40
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 3.6 Trunk-Port: Gi 0/1 (zu Router)

```bash
Switch(config)# interface gi 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,40
Switch(config-if)# no shutdown
Switch(config-if)# exit
Switch(config)# end
```

**Überprüfung:**
```bash
Switch# show interfaces trunk
Switch# show vlan brief
```

**Prüf-Checkliste nach 3.6:**
- [ ] 4 VLANs erstellt
- [ ] Fa 0/1 Access VLAN 10
- [ ] Fa 0/3 Access VLAN 20
- [ ] Fa 0/24 ist Trunk (Status connected)
- [ ] Gi 0/1 ist Trunk (Status connected)

**Wenn OK:** ✅ Weiter zu PHASE 4

---

# PHASE 4: SWITCH1 KONFIGURATION (45 Min)

### 📸 Doku-Hinweis Phase 4
- **Screenshot nach Abschluss:** SS-05
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 4“

## 4.1 Zu Switch1 verbinden

**Schritt:** Klicke auf **Switch1** → **CLI** öffnen

## 4.2 VLANs erstellen (IDENTISCH wie bei Switch Wien-Links)

```bash
Switch# enable
Switch# configure terminal

Switch(config)# vlan 10
Switch(config-vlan)# name Production
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name Office
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name Server
Switch(config-vlan)# exit

Switch(config)# vlan 40
Switch(config-vlan)# name Management
Switch(config-vlan)# exit
```

## 4.3 Access Ports: Fa 0/1 (Production-PC3) → VLAN 10

```bash
Switch(config)# interface fa 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 4.4 Access Ports: Fa 0/2 (NMC) → VLAN 40

```bash
Switch(config)# interface fa 0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 40
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 4.5 Access Ports: Fa 0/3 (Office-PC4) → VLAN 20

```bash
Switch(config)# interface fa 0/3
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 4.6 Access Ports: Fa 0/4 (File-Server) → VLAN 30

```bash
Switch(config)# interface fa 0/4
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 30
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

## 4.7 Trunk-Port: Fa 0/24 (zu Switch Wien-Links)

```bash
Switch(config)# interface fa 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30,40
Switch(config-if)# no shutdown
Switch(config-if)# exit
Switch(config)# end
```

**Überprüfung:**
```bash
Switch# show interfaces trunk
Switch# show vlan brief
```

**Prüf-Checkliste nach 4.7:**
- [ ] 4 VLANs erstellt
- [ ] Fa 0/1 Access VLAN 10 (Production-PC3)
- [ ] Fa 0/2 Access VLAN 40 (NMC)
- [ ] Fa 0/3 Access VLAN 20 (Office-PC4)
- [ ] Fa 0/4 Access VLAN 30 (File-Server)
- [ ] Fa 0/24 ist Trunk (Status connected)

**Wenn OK:** ✅ Weiter zu PHASE 5

---

# PHASE 5: ENDGERÄTE-KONFIGURATION (30 Min)

### 📸 Doku-Hinweis Phase 5
- **Screenshots nach Abschluss:** SS-06 und SS-07
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 5“

## 5.1 Production-PC1 (VLAN 10) – DHCP

**Schritt:**
1. Klicke auf **Production-PC1**
2. Klicke Tab **Desktop** → **IP Configuration**
3. Wähle **DHCP** aktiviert

**Nach ~30 Sekunden sollte zeigen:**
- IP: 10.10.0.2 – 10.10.0.30 (automatisch)
- Gateway: 10.10.0.1
- Subnet Mask: 255.255.255.224

## 5.2 Office-PC2 (VLAN 20) – DHCP

```
Repeat wie 5.1
→ Gateway sollte sein: 10.10.0.33
```

## 5.3 Production-PC3 (VLAN 10) – DHCP

```
Repeat wie 5.1
→ Gateway sollte sein: 10.10.0.1
```

## 5.4 Office-PC4 (VLAN 20) – DHCP

```
Repeat wie 5.1
→ Gateway sollte sein: 10.10.0.33
```

## 5.5 NMC (Management) – STATISCHE IP

**Schritt:**
1. Klicke auf **NMC**
2. Desktop → **IP Configuration**
3. Wähle **Static**
4. Gib ein:
   - **IP Address:** 10.10.0.58
   - **Subnet Mask:** 255.255.255.248
   - **Default Gateway:** 10.10.0.57

> Hinweis: `10.10.0.57` ist bereits das Router-Gateway (Gi0/0.40) und darf daher nicht als Host-IP verwendet werden.

## 5.6 File-Server – STATISCHE IP + Services

**Schritt 1: IP konfigurieren**
1. Klicke auf **File-Server**
2. Desktop → **IP Configuration**
3. Wähle **Static**
4. Gib ein:
   - **IP Address:** 10.10.0.50
   - **Subnet Mask:** 255.255.255.248
   - **Default Gateway:** 10.10.0.49

> Hinweis: `10.10.0.49` ist bereits das Router-Gateway (Gi0/0.30) und darf daher nicht als Host-IP verwendet werden.

**Schritt 2: Services aktivieren**
1. Klicke auf File-Server nochmal → Tab **Services**
2. Aktiviere folgende Services (Button = On):
   - ✅ **FTP** (für Dateiübertragung)
   - ✅ **HTTP** (für Webzugriff)
   - ✅ **SMTP** (für E-Mail-Test)

**Prüf-Checkliste nach Phase 5:**
- [ ] Production-PC1: DHCP aktiv, IP 10.10.0.x
- [ ] Office-PC2: DHCP aktiv, IP 10.10.0.3x
- [ ] Production-PC3: DHCP aktiv, IP 10.10.0.x
- [ ] Office-PC4: DHCP aktiv, IP 10.10.0.3x
- [ ] NMC: Statische IP 10.10.0.58
- [ ] File-Server: Statische IP 10.10.0.50 + Services an

**Wenn OK:** ✅ Weiter zu PHASE 6 (ACLs)

---

# PHASE 6: ACCESS CONTROL LISTS (ACLs) – Netzwerk-Sicherheit (45 Min)

### 📸 Doku-Hinweis Phase 6
- **Screenshot nach Abschluss:** SS-08
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 6“

## 6.1 Sicherheitsrichtlinie (Zugriffsregeln)

### Sicherheitskonzept für das Firmennetzwerk

| Von VLAN | Nach VLAN | Erlaubnis | Grund |
|----------|-----------|-----------|-------|
| Produktion (10) | Server (30) | ✅ ERLAUBT | Daten lesen/schreiben |
| Produktion (10) | Verwaltung (20) | ❌ BLOCKIERT | Abteilungs-Isolation |
| Produktion (10) | Management (40) | ❌ BLOCKIERT | Nur Admin Zugriff |
| Verwaltung (20) | Server (30) | ✅ ERLAUBT | Daten verwalten |
| Verwaltung (20) | Produktion (10) | ❌ BLOCKIERT | Abteilungs-Isolation |
| Verwaltung (20) | Management (40) | ❌ BLOCKIERT | Nur Admin Zugriff |
| Management (40) | ALLE (10,20,30) | ✅ ALLES ERLAUBT | Administrator-Zugriff |
| Externe | Alle | ❌ BLOCKIERT | Nur intern |

## 6.2 ACL-Strategie

Wir erstellen **3 ACLs**:
1. **ACL 101**: Regeln für VLAN 10 (Produktion)
2. **ACL 102**: Regeln für VLAN 20 (Verwaltung)
3. **ACL 103**: Regeln für VLAN 40 (Management - bekommt frei Zugriff)

Diese werden auf den **Subinterfaces inbound** (eingehend, nahe der Quelle) angewendet.

## 6.3 ACL 101 – Produktion (VLAN 10) Zugriffe

**Logik:**
- Darf zum Server (z. B. 10.10.0.50 im VLAN 30) pingen
- Darf zu Management (z. B. 10.10.0.58 im VLAN 40) pingen ❌
- Darf zu Verwaltung (10.10.0.3x) pingen ❌
- Darf zu anderen Produktion IPs pingen ✅

**Im Router CLI eingeben:**

```bash
Router(config)# access-list 101 remark Produktion VLAN 10 - nur zu Server
Router(config)# access-list 101 permit icmp 10.10.0.0 0.0.0.31 10.10.0.48 0.0.0.7
Router(config)# access-list 101 permit icmp 10.10.0.0 0.0.0.31 10.10.0.0 0.0.0.31
Router(config)# access-list 101 deny icmp 10.10.0.0 0.0.0.31 10.10.0.32 0.0.0.15
Router(config)# access-list 101 deny icmp 10.10.0.0 0.0.0.31 10.10.0.56 0.0.0.7
Router(config)# access-list 101 permit ip any any
```

**Erklärung:**
- Line 1: Kommentar
- Line 2: ERLAUBEN Produktion (10.10.0.0/27) → Server (10.10.0.48/29) ICMP/Ping
- Line 3: ERLAUBEN Produktion → Produktion (eigenes VLAN)
- Line 4: BLOCKIEREN Produktion → Verwaltung (10.10.0.32/28)
- Line 5: BLOCKIEREN Produktion → Management (10.10.0.56/29)
- Line 6: Rest erlauben (DNS, sonstiges)

## 6.4 ACL 102 – Verwaltung (VLAN 20) Zugriffe

```bash
Router(config)# access-list 102 remark Verwaltung VLAN 20 - nur zu Server
Router(config)# access-list 102 permit icmp 10.10.0.32 0.0.0.15 10.10.0.48 0.0.0.7
Router(config)# access-list 102 permit icmp 10.10.0.32 0.0.0.15 10.10.0.32 0.0.0.15
Router(config)# access-list 102 deny icmp 10.10.0.32 0.0.0.15 10.10.0.0 0.0.0.31
Router(config)# access-list 102 deny icmp 10.10.0.32 0.0.0.15 10.10.0.56 0.0.0.7
Router(config)# access-list 102 permit ip any any
```

## 6.5 ACL 103 – Management (VLAN 40) - VOLLER ZUGRIFF

```bash
Router(config)# access-list 103 remark Management VLAN 40 - vollständiger Zugriff
Router(config)# access-list 103 permit icmp 10.10.0.56 0.0.0.7 any
Router(config)# access-list 103 permit ip any any
```

**Erklärung:**
- Management darf zu ALLEN Subnetzen (any)
- Alles andere auch erlaubt

## 6.6 ACLs auf Subinterfaces anwenden (INBOUND)

**Auf Gi 0/0.10 (Produktion):**
```bash
Router(config)# interface gi 0/0.10
Router(config-if)# ip access-group 101 in
Router(config-if)# exit
```

**Auf Gi 0/0.20 (Verwaltung):**
```bash
Router(config)# interface gi 0/0.20
Router(config-if)# ip access-group 102 in
Router(config-if)# exit
```

**Auf Gi 0/0.40 (Management):**
```bash
Router(config)# interface gi 0/0.40
Router(config-if)# ip access-group 103 in
Router(config-if)# exit
```

**Wichtig:** Gi 0/0.30 (Server) bekommt KEINE ACL! Server darf von alle erreicht werden.

## 6.7 ACLs überprüfen

```bash
Router# show access-lists
Router# show ip interface gi 0/0.10
Router# show running-config | include access-list
```

**Sollte zeigen:**
```
Extended IP access list 101
    10 permit icmp 10.10.0.0 0.0.0.31 10.10.0.48 0.0.0.7
    20 permit icmp 10.10.0.0 0.0.0.31 10.10.0.0 0.0.0.31
    30 deny icmp 10.10.0.0 0.0.0.31 10.10.0.32 0.0.0.15
    40 deny icmp 10.10.0.0 0.0.0.31 10.10.0.56 0.0.0.7
    50 permit ip any any

Extended IP access list 102
    ... (ähnlich)
```

**Prüf-Checkliste nach Phase 6:**
- [ ] ACL 101 erstellt und auf Gi 0/0.10 applied
- [ ] ACL 102 erstellt und auf Gi 0/0.20 applied
- [ ] ACL 103 erstellt und auf Gi 0/0.40 applied
- [ ] `show access-lists` zeigt alle 3 ACLs
- [ ] `show ip interface gi 0/0.10` zeigt "Inbound access list is 101"

**Wenn OK:** ✅ Weiter zu PHASE 7 (Tests mit Sicherheit)

---

# PHASE 7: TESTS & VALIDIERUNG MIT ACL-SICHERHEIT (30 Min)

### 📸 Doku-Hinweis Phase 7
- **Screenshots nach Abschluss:** SS-09 (erlaubt) und SS-10 (blockiert)
- **Protokolleintrag ausfüllen:** „Protokolleintrag – Phase 7“

## 7.1 DHCP-Status im Router prüfen

**Schritt: Router CLI**
```bash
Router# show ip dhcp pool
Router# show ip dhcp binding
```

**Sollte zeigen:**
- 4 Pools: VLAN10_Production, VLAN20_Office, VLAN30_Server, VLAN40_Management
- Bindings: IPs für PCs die DHCP aktiviert haben

## 7.2 VLAN-Status in Switches prüfen

**Schritt 1: Switch Wien-Links CLI**
```bash
Switch# show vlan brief
Switch# show interfaces status
```

**Sollte zeigen:**
- Fa 0/1: connected (VLAN 10)
- Fa 0/3: connected (VLAN 20)
- Fa 0/24: notconnect, trunk (wird "connected" wenn Switch1 Fa 0/24 up ist)
- Gi 0/1: connected, trunk (verbunden zu Router)

**Schritt 2: Switch1 CLI**
```bash
Switch# show vlan brief
Switch# show interfaces status
```

**Sollte zeigen:**
- Fa 0/1: connected (VLAN 10)
- Fa 0/2: connected (VLAN 40)
- Fa 0/3: connected (VLAN 20)
- Fa 0/4: connected (VLAN 30)
- Fa 0/24: notconnect, trunk (wird "connected" wenn Trunk zu Wien-Links up ist)

## 7.3 Ping-Tests – ERLAUBTE Verbindungen ✅

**Test 1: Produktion → Server (SOLLTE FUNKTIONIEREN)**
```bash
C:\> ping 10.10.0.50  (von Production-PC1 zu File-Server)
```
**Ergebnis:** ✅ Reply oder !!!!

**Test 2: Verwaltung → Server (SOLLTE FUNKTIONIEREN)**
```bash
C:\> ping 10.10.0.50  (von Office-PC2 zu File-Server)
```
**Ergebnis:** ✅ Reply oder !!!!

**Test 3: Management → Server (SOLLTE FUNKTIONIEREN)**
```bash
C:\> ping 10.10.0.50  (von NMC zu File-Server)
```
**Ergebnis:** ✅ Reply oder !!!!

**Test 4: Management → Produktion (SOLLTE FUNKTIONIEREN)**
```bash
C:\> ping 10.10.0.2  (von NMC zu Production-PC1)
```
**Ergebnis:** ✅ Reply oder !!!!

## 7.4 Ping-Tests – BLOCKIERTE Verbindungen ❌

**Test 5: Produktion → Verwaltung (SOLLTE BLOCKIERT SEIN)**
```bash
C:\> ping 10.10.0.34  (von Production-PC1 zu Office-PC2)
```
**Ergebnis:** ❌ Timeout oder "Request timed out"

**Begründung:** ACL 101 blockiert diese Route

**Test 6: Verwaltung → Produktion (SOLLTE BLOCKIERT SEIN)**
```bash
C:\> ping 10.10.0.2  (von Office-PC2 zu Production-PC1)
```
**Ergebnis:** ❌ Timeout oder "Request timed out"

**Begründung:** ACL 102 blockiert diese Route

**Test 7: Produktion → Management (SOLLTE BLOCKIERT SEIN)**
```bash
C:\> ping 10.10.0.58  (von Production-PC1 zu NMC)
```
**Ergebnis:** ❌ Timeout oder "Request timed out"

**Begründung:** ACL 101 blockiert diese Route

**Test 8: Verwaltung → Management (SOLLTE BLOCKIERT SEIN)**
```bash
C:\> ping 10.10.0.58  (von Office-PC2 zu NMC)
```
**Ergebnis:** ❌ Timeout oder "Request timed out"

**Begründung:** ACL 102 blockiert diese Route

## 7.5 Ping-Tests – Interne VLAN-Kommunikation ✅

**Test 9: Produktion-PC1 → Produktion-PC3 (SOLLTE FUNKTIONIEREN)**
```bash
C:\> ping 10.10.0.3  (Beispiel: von Production-PC1 zu Production-PC3 - same VLAN 10)
```
**Ergebnis:** ✅ Reply oder !!!!

**Begründung:** Beide im gleichen VLAN 10, ACL blockiert nur zu anderen VLANs

**Test 10: Office-PC2 → Office-PC4 (SOLLTE FUNKTIONIEREN)**
```bash
C:\> ping 10.10.0.35  (von Office-PC2 zu Office-PC4 - same VLAN)
```
**Ergebnis:** ✅ Reply oder !!!!

**Begründung:** Beide im gleichen VLAN 20, ACL blockiert nur zu anderen VLANs

## 7.6 ACL-Debug: Blockierte Pakete anzeigen

**Um zu sehen welche Pings blockiert werden:**

```bash
Router(config)# access-list 101 remark Produktion - mit logging
Router(config)# no access-list 101
```

Dann erneut eingeben MIT Logging:

```bash
Router(config)# access-list 101 permit icmp 10.10.0.0 0.0.0.31 10.10.0.48 0.0.0.7 log
Router(config)# access-list 101 permit icmp 10.10.0.0 0.0.0.31 10.10.0.0 0.0.0.31 log
Router(config)# access-list 101 deny icmp 10.10.0.0 0.0.0.31 10.10.0.32 0.0.0.15 log
Router(config)# access-list 101 deny icmp 10.10.0.0 0.0.0.31 10.10.0.56 0.0.0.7 log
Router(config)# access-list 101 permit ip any any
Router(config)# interface gi 0/0.10
Router(config-if)# ip access-group 101 in
```

Dann Pings versuchen und folgendes eingeben:

```bash
Router# show log
```

**Sollte zeigen wenn Production-PC1 zu Verwaltung pingt:**
```
%ACL-6-ICMPDENY: ICMP denied -> (ICMP type=8, code=0)
```

## 7.7 Tracert-Test mit ACL-Sicherheit

```bash
C:\> tracert 10.10.0.50  (von Production-PC1 zu File-Server)
```
**Sollte zeigen:**
1. Production-PC1 (10.10.0.x)
2. Router (10.10.0.1)  ← ACL 101 ERLAUBT diese Route
3. File-Server (10.10.0.50)

```bash
C:\> tracert 10.10.0.34  (von Production-PC1 zu Office-PC2)
```
**Sollte zeigen:**
1. Production-PC1 (10.10.0.x)
2. Router (10.10.0.1)  ← ACL 101 BLOCKIERT diese Route
3. Timeout/*** (weil Router Paket dropped)

## 7.8 Router-Logs mit ACL anschauen

```bash
Router# terminal logging
Router# show logging
```

**Um detaillierte ACL-Hits zu sehen (mit der "log" Option):**
```bash
Router# show access-lists
```

Zeigt Trefferzähler für jede Regel.

---

**Prüf-Checkliste nach Phase 7 (Tests mit ACL):**
- [ ] Produktion → Server Ping: FUNKTIONIERT ✅
- [ ] Verwaltung → Server Ping: FUNKTIONIERT ✅
- [ ] Management → Alle Pings: FUNKTIONIERT ✅
- [ ] Produktion → Verwaltung Ping: BLOCKIERT ❌
- [ ] Verwaltung → Produktion Ping: BLOCKIERT ❌
- [ ] Produktion → Management Ping: BLOCKIERT ❌
- [ ] Interne Same-VLAN Pings: FUNKTIONIEREN ✅
- [ ] ACL-Regeln mit `show access-lists` sichtbar
- [ ] Logging zeigt blockierte Pings (optional)

---

# 🎉 ERFOLGSMELDUNG

**Wenn alle Tests der Phase 7 OK:**

✅ Router alle Subinterfaces UP  
✅ Switches alle Ports connected  
✅ Alle PCs haben IP-Adressen  
✅ ACLs auf allen Subinterfaces aktiv  
✅ Pings funktionieren (erlaubte Routen)  
✅ Pings blockiert (verbotene Routen)  
✅ Router-Routing funktioniert  
✅ DHCP funktioniert  
✅ Sicherheitsrichtlinie durchgesetzt  

**🎯 NETZWERK MIT SICHERHEIT IST LIVE!**

---

# TROUBLESHOOTING

## ❌ Fehler: "Gi 0/0.10 is down, line protocol is DOWN"

**Ursache:** Keine physikalische Verbindung

**Lösung:**
1. Überprüfe Kabelverbindung Router → Switch (Gi 0/1)
2. Warte 10-15 Sekunden
3. Gib erneut ein: `show ip interface brief`

## ❌ Fehler: VLAN nicht erstellt

**Ursache:** VLANs wurden aber Modus nicht korrekt verlassen

**Lösung:**
```bash
Switch(config)# no vlan 10  (löschen)
Switch(config)# vlan 10      (neu erstellen)
Switch(config-vlan)# name Production
Switch(config-vlan)# exit
```

## ❌ Fehler: "PC erhält IP-Adresse 0.0.0.0"

**Ursache:** DHCP-Pool nicht konfiguriert oder Gateway excluded vergessen

**Lösung:**
1. Router: `show ip dhcp pool` 
2. Überprüfe excluded-address
3. Wenn nichts: Nochmal Schritt 2.5-2.9 durchgehen

## ❌ Fehler: "Ping funktioniert, sollte aber blockiert sein"

**Ursache:** ACL wurde nicht auf Subinterface angewendet

**Lösungsschritte:**
1. Überprüfe ACL-Application:
   ```bash
   Router# show ip interface gi 0/0.10 | include access list
   ```
   Sollte zeigen: `Inbound access list is 101`

2. Wenn nicht sichtbar: Nochmal anwenden:
   ```bash
   Router(config)# interface gi 0/0.10
   Router(config-if)# ip access-group 101 in
   Router(config-if)# exit
   ```

3. Teste nochmal: ACL sollte jetzt greifen

## ❌ Fehler: "Alle Pings sind blockiert (auch die sollten erlaubt sein)"

**Ursache:** ACL-Syntax fehlerhaft oder falsche Zeilen

**Lösungsschritte:**
1. ACL anzeigen:
   ```bash
   Router# show access-lists 101
   ```

2. Überprüfe die Zeilen (sollten permit sein, nicht deny)

3. Falls falsch: Löschen und neu erstellen:
   ```bash
   Router(config)# no access-list 101
   (nochmal Phase 6.3 durchgehen)
   ```

## ❌ Fehler: "ACL funktioniert, aber einige erlaubte Pings gehen nicht"

**Ursache:** "permit ip any any" vergessen (erlaubt andere Protokolle)

**Lösung:**
Stelle sicher dass jede ACL MIT endet:
```bash
Router(config)# access-list 101 permit ip any any
```

Ohne diese Zeile werden nur ICMP erlaubt, alles andere (DNS, etc.) blockiert!

---

# CHECKLISTE ZUM ABHAKEN

**Nach Router-Konfiguration (Phase 2):**
- [ ] Gi 0/0.10 UP mit 10.10.0.1
- [ ] Gi 0/0.20 UP mit 10.10.0.33
- [ ] Gi 0/0.30 UP mit 10.10.0.49
- [ ] Gi 0/0.40 UP mit 10.10.0.57
- [ ] 4 DHCP-Pools definiert
- [ ] Gateway-adressen excluded

**Nach Switch Wien-Links (Phase 3):**
- [ ] 4 VLANs erstellt (10, 20, 30, 40)
- [ ] Fa 0/1 Access VLAN 10 connected
- [ ] Fa 0/3 Access VLAN 20 connected
- [ ] Fa 0/24 Trunk connected
- [ ] Gi 0/1 Trunk connected

**Nach Switch1 (Phase 4):**
- [ ] 4 VLANs erstellt
- [ ] Fa 0/1 Access VLAN 10 connected
- [ ] Fa 0/2 Access VLAN 40 connected
- [ ] Fa 0/3 Access VLAN 20 connected
- [ ] Fa 0/4 Access VLAN 30 connected
- [ ] Fa 0/24 Trunk connected (zu Wien-Links)

**Nach Endgeräten (Phase 5):**
- [ ] Production-PC1 hat IP (10.10.0.x)
- [ ] Office-PC2 hat IP (10.10.0.3x)
- [ ] Production-PC3 hat IP (10.10.0.x)
- [ ] Office-PC4 hat IP (10.10.0.3x)
- [ ] NMC hat 10.10.0.58
- [ ] File-Server hat 10.10.0.50
- [ ] Services auf File-Server aktiviert

**Nach Tests (Phase 7):**
- [ ] Alle Router-Interfaces UP
- [ ] Alle Switch-Ports connected/notconnect (nicht shutdown)
- [ ] Ping vom Router auf alle PC-IPs OK
- [ ] Ping Production → Server OK ✅
- [ ] Ping Verwaltung → Server OK ✅
- [ ] Ping Management → Alle OK ✅
- [ ] Ping Production → Verwaltung BLOCKIERT ❌
- [ ] Ping Verwaltung → Production BLOCKIERT ❌
- [ ] Ping Production → Management BLOCKIERT ❌
- [ ] Same-VLAN Pings OK (PC1 → PC3)
- [ ] Tracert zeigt Router in Mitte
- [ ] ACL-Regeln mit `show access-lists` sichtbar

---

## Konfigurationsstand (Hier eintragen!)

### Session 1
- **Datum:** ___________
- **Zeit Start:** ___________
- **Zeit Ende:** ___________
- **Phase abgeschlossen:** 🔲 1 | 🔲 2 | 🔲 3 | 🔲 4 | 🔲 5 | 🔲 6 | 🔲 7
- **Probleme:** ___________

### Session 2
- **Datum:** ___________
- **Phase abgeschlossen:** 🔲 2 | 🔲 3 | 🔲 4 | 🔲 5 | 🔲 6 | 🔲 7
- **Probleme:** ___________

---

**Status:** FERTIG ZUM STARTEN 🚀  
**Letzte Änderung:** 20. Mai 2026  
**Autor:** Konfigurationsassistent
