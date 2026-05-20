# Leased Lines im Packet Tracer konfigurieren

**Firma:** FRB Software Entwicklung GmbH  
**Typ:** WAN-Verbindungen (Serial DCE/DTE)  
**Erstellt:** 11. Mai 2026

---

## Warum Leased Lines?

### Das Problem mit physischen Kabeln

Ein physisches Kupferkabel würde für die Entfernung zwischen Wien und Linz **tatsächlich nicht ausreichen**. Die maximale Reichweite von Kupferkabeln beträgt nur wenige hundert Meter, daher werden hier **Leased Lines** verwendet.

### Was ist eine Leased Line?

Eine **Leased Line ist NICHT einfach ein Kabel**, sondern ein **gemieteter Telekommunikationsservice**:

- Man mietet eine **dedizierte Verbindung** von einem Telekommunikationsanbieter (z.B. A1, Magenta Austria)
- Die **physische Infrastruktur** (Glasfaser, Kupfer, Funk) liegt beim Provider
- Der Provider verbindet Wien und Linz über seine eigenen Netzwerk-Backbones
- Man bekommt nur die **Endpunkte** (serielle Schnittstellen an den Routern) bereitgestellt

### In dieser Dokumentation:

| Wien → Linz | Lösung |
|---|---|
| **Entfernung** | ~180 km |
| **Verbindungstyp** | Leased Line (Serial DCE/DTE) |
| **Kabeltyp im Packet Tracer** | Serial DCE/DTE (Simulation) |
| **Real-Welt-Option** | Glasfaser vom ISP über lange Distanzen |

### Alternative Technologien:

- **VPN über Internet** (heute häufiger, billiger)
- **MPLS-Verbindungen** (managed services)
- **Satellit** (wenn sonst nichts geht)

> Die Leased Line ist die klassische Lösung in dieser Dokumentation – speziell für Firmennetzwerke mit hohen Anforderungen an Verfügbarkeit und Bandbreite.

---

## Schritt 1: WIC-Module hinzufügen

1. **HQ Wien Router** anklicken
2. Rechts unten: **"WIC-2T" Modul** in **Serial 0** und **Serial 1** einstecken
   - (Das Modul muss im Router bereits installiert sein oder über die physische Konfiguration hinzugefügt werden)

3. **Branch Linz Router**: **WIC-1T Modul** in **Serial 0** einstecken
4. **Branch Graz Router**: **WIC-1T Modul** in **Serial 0** einstecken

---

## Schritt 2: Serielle Kabel verbinden

1. Im **Packet Tracer**: Kabel-Werkzeug → **"Serial DCE" oder "Serial DTE"** auswählen
2. **HQ Wien (Serial 0/0)** → **Branch Linz (Serial 0/0)** verbinden
3. **HQ Wien (Serial 0/1)** → **Branch Graz (Serial 0/0)** verbinden

> ⚠️ **Wichtig**: Immer vom **DCE-Ende** (HQ Wien) starten, damit die Verbindung grün wird!

---

## Schritt 3: Interfaces konfigurieren

### An **HQ Wien Router** (im CLI):

```bash
Router# configure terminal
Router(config)# interface serial 0/0
Router(config-if)# ip address 10.10.0.73 255.255.255.252
Router(config-if)# clock rate 64000
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface serial 0/1
Router(config-if)# ip address 10.10.0.77 255.255.255.252
Router(config-if)# clock rate 64000
Router(config-if)# no shutdown
Router(config-if)# exit
```

### An **Branch Linz Router**:

```bash
Router# configure terminal
Router(config)# interface serial 0/0
Router(config-if)# ip address 10.10.0.74 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit
```

### An **Branch Graz Router**:

```bash
Router# configure terminal
Router(config)# interface serial 0/0
Router(config-if)# ip address 10.10.0.78 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit
```

---

## Schritt 4: Testen

Ping von **HQ Wien** nach **Linz**:
```bash
Router# ping 10.10.0.74
```

Sollte **erfolgreich** sein! ✅

---

## Zusammenfassung

| Router | Port | IP-Adresse | Clock Rate |
|---|---|---|---|
| HQ Wien | Serial 0/0 | 10.10.0.73 | **64000** |
| Branch Linz | Serial 0/0 | 10.10.0.74 | — |
| HQ Wien | Serial 0/1 | 10.10.0.77 | **64000** |
| Branch Graz | Serial 0/0 | 10.10.0.78 | — |

### Wichtige Hinweise:

- **Clock Rate wird NUR am DCE-Ende (Wien) gesetzt**
- Alle Interfaces müssen mit `no shutdown` aktiviert sein
- Die IP-Adressen müssen im Subnetz `10.10.0.72/30` und `10.10.0.76/30` liegen
- Serial DCE/DTE ist eine serielle Punkt-zu-Punkt-Verbindung

