# Checkliste – Ist-Stand & Ablauf

> Diese Datei kombiniert:
> 1) **aktuellen Ist-Stand**
> 2) **konkrete To-do Reihenfolge** (was wann wie)

---

## 1) Aktueller Ist-Stand (01.06.2026)

### Bereits erledigt
- [x] Router Subinterfaces `.10/.20/.30/.40` konfiguriert
- [x] Router Subinterfaces `.10/.20/.30/.40` sind **up/up** (`show ip interface brief`)
- [x] Router DHCP Pools für VLAN 10/20/30/40 vorhanden
- [x] Excluded Addresses gesetzt
- [x] Switch1 VLANs 10/20/30/40 vorhanden
- [x] Switch1 Trunk Fa0/24 aktiv
- [x] Switch Wien-Links: VLAN 10/20/30/40 vorhanden
- [x] Switch Wien-Links: Access-Ports gesetzt (`Fa0/1 -> VLAN10`, `Fa0/3 -> VLAN20`)
- [x] Switch Wien-Links: Trunk `Fa0/24` aktiv
- [x] Switch Wien-Links: Trunk `Gi0/1` zum Router aktiv
- [x] NMC statische IP korrigiert (`10.10.0.58 / 255.255.255.248`, GW `10.10.0.57`)
- [x] FileServer statische IP korrigiert (`10.10.0.50 / 255.255.255.248`, GW `10.10.0.49`)

### Teilweise erledigt / prüfen
- [x] Kein offener Punkt bei Schritt B (Switch Wien-Links verifiziert)

### Noch offen
- [x] ACL 101/102/103 konfigurieren und anwenden
- [x] Erlaubte/Blockierte Ping-Tests durchführen und dokumentieren

---

## 2) Konkreter Ablauf ab jetzt (was/wann/wie)

## Schritt A – Endgeräte-IP korrigieren (ca. 10 Min)

### A1) NMC korrigieren
- Desktop → IP Configuration → Static
- IPv4 Address: `10.10.0.58`
- Subnet Mask: `255.255.255.248`
- Default Gateway: `10.10.0.57`

### A2) FileServer korrigieren
- Desktop → IP Configuration → Static
- IPv4 Address: `10.10.0.50`
- Subnet Mask: `255.255.255.248`
- Default Gateway: `10.10.0.49`

### A3) Nachweis
- [x] Screenshot SS-06 (NMC IP)
- [x] Screenshot SS-07 (FileServer IP/Services)

---

## Schritt B – Switch Wien-Links verifizieren (ca. 10 Min)

### B1) Befehle
```bash
show vlan brief
show interfaces trunk
```

### B2) Sollzustand
- Fa0/1 in VLAN 10
- Fa0/3 in VLAN 20
- Fa0/24 trunking
- Gi0/1 trunking

### B3) Falls Access-Ports fehlen
```bash
conf t
interface fa0/1
 switchport mode access
 switchport access vlan 10
 no shut
exit
interface fa0/3
 switchport mode access
 switchport access vlan 20
 no shut
end
```

### B4) Nachweis
- [ ] Screenshot SS-04

---

## Schritt C – ACL konfigurieren (ca. 20–30 Min)

### C1) ACL erstellen (Router)
```bash
conf t
access-list 101 remark Production VLAN10
access-list 101 permit icmp 10.10.0.0 0.0.0.31 10.10.0.48 0.0.0.7
access-list 101 permit icmp 10.10.0.0 0.0.0.31 10.10.0.0 0.0.0.31
access-list 101 deny icmp 10.10.0.0 0.0.0.31 10.10.0.32 0.0.0.15
access-list 101 deny icmp 10.10.0.0 0.0.0.31 10.10.0.56 0.0.0.7
access-list 101 permit ip any any

access-list 102 remark Office VLAN20
access-list 102 permit icmp 10.10.0.32 0.0.0.15 10.10.0.48 0.0.0.7
access-list 102 permit icmp 10.10.0.32 0.0.0.15 10.10.0.32 0.0.0.15
access-list 102 deny icmp 10.10.0.32 0.0.0.15 10.10.0.0 0.0.0.31
access-list 102 deny icmp 10.10.0.32 0.0.0.15 10.10.0.56 0.0.0.7
access-list 102 permit ip any any

access-list 103 remark Management VLAN40
access-list 103 permit icmp 10.10.0.56 0.0.0.7 any
access-list 103 permit ip any any
```

### C2) ACL anwenden (INBOUND)
```bash
interface gi0/0.10
 ip access-group 101 in
exit
interface gi0/0.20
 ip access-group 102 in
exit
interface gi0/0.40
 ip access-group 103 in
end
```

### C3) Nachweis
```bash
show access-lists
show ip interface gi0/0.10
```
- [ ] Screenshot SS-08

---

## Schritt D – Tests + Abnahme (ca. 15 Min)

### D1) Erlaubte Pings (müssen funktionieren)
- [x] Produktion → FileServer: `ping 10.10.0.50`
- [x] Verwaltung → FileServer: `ping 10.10.0.50`
- [x] NMC → Produktion: `ping 10.10.0.2`

### D2) Blockierte Pings (müssen fehlschlagen)
- [x] Produktion → Verwaltung: `ping 10.10.0.34`
- [x] Verwaltung → Produktion: `ping 10.10.0.2`
- [x] Produktion → NMC: `ping 10.10.0.58`

### D3) Nachweis
- [x] Screenshot SS-09 (erlaubter Ping)
- [x] Screenshot SS-10 (blockierter Ping)

---

## 3) Abschluss-Check (Projekt fertig?)

- [ ] Alle 7 Phasen auf „Fertig“ gesetzt
- [ ] Alle 10 Screenshots vorhanden
- [ ] Router/Switch Konfiguration gespeichert (`copy run start`)
- [ ] Protokoll vollständig ausgefüllt
- [ ] Abgabe-Dateien final geordnet

---

## 4) Dateinamen-Empfehlung für Abgabe

- `PROTOKOLL_0_VORLAGE.md` (als Vorlage)
- `PROTOKOLL_AUSGEFUELLT_YYYY-MM-DD.md` (dein echtes Abgabeprotokoll)
- `CHECKLISTE_ISTSTAND_UND_ABLAUF.md`
