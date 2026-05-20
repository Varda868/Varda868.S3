# /"\  Varda868

**Lizenzfreier Gipfelfunk auf 868 MHz · Mesh · Summit · Far**

> *JS8-Geist, SOTA-Spielidee, WSPR-Disziplin — auf einem 15-Euro-Chip.*

---

## Was ist das?

Varda868 ist eine **offene Gipfelfunk-Aktivität** für das lizenzfreie 868-MHz-g3-ISM-Band (EU). Zwei Leute, zwei Berge, ein signierter Funkkontakt — das ist das fertige Erlebnis. Das Mesh wächst, wenn mehr mitmachen.

Die Spielidee lehnt sich an drei Vorbilder:

| Vorbild | Was Varda868 davon nimmt |
|---|---|
| **SOTA** — Summits on the Air | Aktivierer auf dem Gipfel, Jäger von unten; Gipfel sammeln |
| **Statshunter / OSM** | Kartenraster (Slippy-Tiles Zoom 14) als zweite Sammlerdimension |
| **JS8Call** | Asynchron, store-and-forward, zähe Verbindungen über weite Strecken |

Das Kästchen ist ein eigenständiges Gerät — kein Server, kein Konto, kein Internet nötig.
Die Spielsammlung lebt lokal, signiert mit Ed25519, exportierbar als JSON-Datei.

---

## ⚠ Alpha-Status

Dies ist eine **Alpha-Firmware**. Hardware funktioniert, Spielschicht ist spielbar,
aber noch nicht feldgehärtet:

- Kein automatischer Reboot bei Ausnahmen
- Gipfelkatalog enthält bisher nur Norddeutschland (Harz-Region, 584 Gipfel)
- Wanderer-Transfer über Far-Funk: Protokoll stabil, noch wenig Feldtest
- Keine Over-the-Air-Updates

Bugs und Feedback bitte als [Issue](../../issues) melden.

---

## Hardware

| Teil | Modell | Preis (ca.) |
|---|---|---|
| Mikrocontroller | Seeed Studio **XIAO ESP32-S3** (8 MB PSRAM, 8 MB Flash) | ~10 € |
| Funkmodul | Seeed **Wio-SX1262** (SX1262 mit TCXO) | ~8 € |
| Antenne | 868-MHz-Whip oder Omni 3–5 dBi | ~5–15 € |
| Stromquelle | USB-Powerbank oder LiPo | — |

Die beiden Platinen werden direkt gestapelt (Board-to-Board-Stecker).
**Antenne immer zuerst anschließen, niemals ohne Antenne senden.**

### Pinbelegung (SPI, Board-to-Board-Stecker)

| Signal | GPIO |
|---|---|
| NSS (CS) | 41 |
| DIO1 (IRQ) | 39 |
| BUSY | 40 |
| RST | 42 |

---

## Build & Flash

### Voraussetzungen

- [PlatformIO](https://platformio.org/) (VS Code Extension oder CLI)
- Python 3.x

### Schritte

```bash
git clone https://github.com/varda868/varda868.git
cd varda868
```

In VS Code: **PlatformIO → Upload** — oder auf der Kommandozeile:

```bash
# Firmware flashen
pio run --target upload

# LittleFS-Image flashen (Gipfelkatalog, beim ersten Mal zwingend)
pio run --target uploadfs
```

> **Windows-Hinweis:** Das Build-Skript `scripts/scons_jobs.py` erzwingt seriellen
> Build wegen einer `ar.exe`-Race-Condition unter Windows. Das ist beabsichtigt.

---

## Erster Start

1. **Antenne anschließen**, dann Strom anlegen.
2. Mit dem WLAN des Kästchens verbinden — SSID beginnt mit `Varda868-`.
   Kein Passwort.
3. Browser öffnen: **`192.168.4.1`**
   (Alternativ: `varda868.local` auf iOS / macOS / Windows 10+;
   Android Chrome benötigt die IP-Adresse)
4. Beim ersten Start: **„Neue Identität"** wählen, 12-Wort-Phrase notieren.
   Diese Wörter sind die einzige Sicherung der Identität — sicher aufbewahren.

### GPS einrichten (optional, aber empfohlen)

Varda868 nutzt [OwnTracks](https://owntracks.org/) als GPS-Quelle:

- OwnTracks installieren (Android/iOS, kostenlos, Open Source)
- Konfigurationsdatei `tools/owntracks_varda868.json` in OwnTracks importieren
- OwnTracks mit dem Kästchen-WLAN verbinden → GPS fließt automatisch

Ohne GPS sind Gipfelaktivierungen und DX-Streckenrekorde nicht möglich.

---

## Die drei Räume

| Raum | Button | SF/BW | Zweck |
|---|---|---|---|
| **#&nbsp;Mesh** | Ping | SF10 / 62,5 kHz | Beacons, Karte, Passivbetrieb |
| **^&nbsp;Summit** | CQ | SF7 / 62,5 kHz | Kontakte machen, Gipfel aktivieren |
| **~&nbsp;Far** | CQ DX | SF12 / 15,6 kHz | Weitstrecke, DX-Rekord, Wanderer-Flug |

Alle drei Räume liegen im **g3-Sub-Band (869,4–869,65 MHz)**,
maximal 500 mW ERP, Duty Cycle ≤ 10 % / Stunde — das Kästchen überwacht das selbst.

---

## Was gesammelt wird

- **Kontakte** — jeder signierte HELLO/ACK/CONFIRM-Handshake
- **Gipfelaktivierungen** — 4 unabhängige Kontakte von einem Gipfel aus
- **Kacheln** — OSM Slippy-Map Zoom 14; jeder Kontakt färbt deine aktuelle Kachel ein
- **DX-Rekord** — die weiteste einzelne Verbindung in km (GPS beider Seiten nötig)
- **Wanderer** — virtuelle Reisegefährten, die per Funk von Knoten zu Knoten reisen

---

## Technische Highlights

- **Signierter Handshake**: Ed25519-Schlüsselpaar pro Knoten; ein Kontakt braucht
  beide privaten Schlüssel — kein Einzelner kann ihn fälschen
- **Passive Propagationskarte**: Radarplot aller gehörten Knoten (RSSI → Radius),
  gespeist aus normalen Beacons, kein aktives Sondieren
- **Drei feste Räume** statt freier Einstellungen — alle Knoten hören sich
  garantiert; kein Aushandeln, kein Fummeln
- **Kein Server, kein Konto**: Identität = 12-Wort-Phrase (BIP39-ähnlich),
  Sammlung = lokales JSON-Log auf LittleFS
- **Beobachter-Modus**: Kästchen ohne verbundenen Browser wechselt automatisch
  zwischen den drei Räumen und zeichnet Propagation auf

---

## Verzeichnisstruktur

```
src/           Firmware (C++, PlatformIO)
  main.cpp     Hauptschleife, Beacon, Observer, Seek/CQ
  webui.cpp    Web-UI, REST-API, WebSocket
  contact.cpp  HELLO/ACK/CONFIRM-Handshake
  wanderer.*   Wanderer/Trackables
  scoring.*    Punkte, Aktivierungen, DX
  catalog.*    Gipfelkatalog (peaks.bin → PSRAM)
  ...
data/          LittleFS-Image
  peaks.bin    Gipfelkatalog (Harz, 584 Einträge)
tools/         Build- und Datenpipeline-Skripte
  owntracks_varda868.json   OwnTracks-Importkonfiguration
  build_catalog.py          Katalog aus GeoJSON/DEM bauen
```

---

## Dokumentation

| Datei | Inhalt |
|---|---|
| `CLAUDE.md` | Vollständige Konzept- und Implementierungsreferenz |
| `Schnellstart.md` | Kurzanleitung für den ersten Einsatz |
| `Spielanleitung.md` | Spielregeln, Mechaniken, Wanderer-System |
| `Wertung.md` | Punkteformel, Sterngrenzen, Aktivierungsschwellen |
| `spielkonzept.md` | Detailliertes Spielkonzept |
| `CHANGELOG.md` | Versionshistorie |

---

## Funkrecht (EU)

Varda868 nutzt ausschließlich das **g3-Sub-Band 869,4–869,65 MHz**
(ERC REC 70-03 / ETSI EN 300 220):

- Sendeleistung: max. **500 mW ERP**
- Duty Cycle: max. **10 % / Stunde** — der Duty-Cycle-Tracker im Kästchen
  erzwingt das automatisch
- **Keine Amateurfunklizenz erforderlich** — das Band ist lizenzfrei nutzbar

Alle Kanäle liegen mit Schutzabstand im 250-kHz-Fenster; BW250 wird nie genutzt.
Details: `CLAUDE.md` Kapitel 4.

---

## Roadmap

- [x] Hardware Bring-up (XIAO ESP32-S3 + Wio-SX1262)
- [x] Signierter Kontakt-Handshake (Ed25519)
- [x] Drei Räume, Duty-Cycle-Tracker
- [x] Beacon-Layer, Propagationskarte, WebSocket-Live-Feed
- [x] Store-and-Forward (TTL-Relay), Beobachter-Modus
- [x] Spielschicht: Kacheln, Gipfelkatalog, Scoring, Wanderer
- [x] Seek/CQ, Far-DX, Wanderer-Flug (OFFER/ACCEPT/CLAIM)
- [ ] Feld-Härtung: Watchdog, Solar, Stromoptimierung
- [ ] Interaktiver Scan-Takt (4-s-Zyklus)
- [ ] Gipfelkatalog erweitern (Deutschland gesamt)
- [ ] Globale Rangliste (optional, föderiert)

---

## Mitmachen

Das Projekt ist jung und die Hardware-Basis günstig — jeder mit einem
XIAO ESP32-S3 + Wio-SX1262 kann mitmachen.

Nützliche Beiträge:

- **Feldtests** mit zwei oder mehr Knoten und Feedback als Issue
- **Gipfelkatalog** für weitere Regionen (Werkzeug: `tools/build_catalog.py`)
- **Protokoll-Review** — insbesondere Ed25519-Signaturkontext und Wanderer-Claim

---

*Lizenz: MIT · Logo `/"\` ist gemeinfrei · Funkrecht liegt beim Betreiber*
