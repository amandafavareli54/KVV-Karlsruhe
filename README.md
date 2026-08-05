# KVV Störungs- & Baustellen-Karte Karlsruhe

Eine Web-App für das Netz des Karlsruher Verkehrsverbunds (KVV). Sie zeigt Echtzeit-Abfahrten,
Verbindungen und – als Schwerpunkt – **Störungen, Baustellen und Sperrungen mit ihrem
voraussichtlichen Zeitraum**.

Die offizielle KVV-Auskunft nennt Baustellen meist nur als Textmeldung. Diese App macht sichtbar,
**wo** eine Einschränkung liegt: betroffene Haltestellen werden auf der Karte rot markiert, und die
Meldung nennt Grund, betroffene Linien und Gültigkeitszeitraum.

![Status](https://img.shields.io/badge/Status-in%20Entwicklung-blue)
![Lizenz](https://img.shields.io/badge/Lizenz-MIT-green)

---

## Funktionen

| Bereich | Beschreibung |
| --- | --- |
| **Interaktive Karte** | Dunkles Kartenbild, Haltestellen als anklickbare Punkte – ohne Tippen |
| **Störungs-Markierung** | Haltestellen mit Störung oder Baustelle erscheinen rot (⚠️) |
| **Liniennetz** | Alle Tram- und Stadtbahnlinien in den offiziellen KVV-Farben, aus GTFS-Daten erzeugt |
| **Echtzeit-Abfahrten** | Nächste Abfahrten mit Verspätung in Minuten, Aktualisierung alle 30 s |
| **Meldungen nach Kategorie** | Getrennt nach ⏱️ Verspätungen, ⚠️ Störungen und 🚧 Baustellen |
| **Verbindungssuche (A→B)** | Route mit Umstiegen, auf der Karte gezeichnet; gestörte Etappen in Rot |
| **Barrierefreiheit** | Rollstuhl-Zugänglichkeit und taktiles Leitsystem je Bahnsteig (OpenStreetMap) |
| **Haltestellensuche** | Autovervollständigung, Treffer im Raum Karlsruhe werden priorisiert |

---

## Datenquellen

| Quelle | Verwendung | Lizenz |
| --- | --- | --- |
| [EFA-JSON-API (MobiData BW)](https://www.efa-bw.de/mobidata-bw/) | Haltestellen, Abfahrten, Verbindungen, Störungsmeldungen | offen, ohne Authentifizierung |
| [KVV GTFS](https://www.kvv.de/fahrplan/fahrplaene/open-data.html) | Liniennetz (`lines.geojson`) | CC0 |
| [OpenStreetMap / Overpass](https://overpass-api.de/) | Barrierefreiheit der Bahnsteige | ODbL |
| [CARTO Basemaps](https://carto.com/basemaps/) | Dunkles Kartenbild | © OpenStreetMap, © CARTO |

Die Linienfarben folgen dem offiziellen KVV-Gestaltungshandbuch
(dokumentiert im [Stadtwiki Karlsruhe](https://ka.stadtwiki.net/Stadtwiki:Projekt_KVV/Gestaltung)).

---

## Technik

Bewusst ohne Framework und ohne Backend – die App läuft vollständig im Browser:

- **HTML / CSS / JavaScript** (Vanilla, keine Build-Tools)
- **[Leaflet 1.9.4](https://leafletjs.com/)** für die Karte (via CDN)
- **GeoJSON** für das vorbereitete Liniennetz

```
.
├── index.html       # gesamte Anwendung (Markup, Styles, Logik)
├── lines.geojson    # Liniennetz mit offiziellen Farben (aus GTFS erzeugt)
└── README.md
```

### Warum kein Framework?

Das Projekt zeigt bewusst, dass sich eine vollständige Karten-Anwendung mit mehreren
Live-Datenquellen ohne React, Build-Pipeline oder Server umsetzen lässt. Das hält die App klein,
schnell ladbar und überall statisch hostbar.

---

## Lokal starten

Die App muss über einen lokalen Server laufen (nicht per Doppelklick), da Browser bei `file://`
keine API-Aufrufe erlauben.

**Mit VS Code (empfohlen für den Einstieg)**

1. Erweiterung **Live Server** installieren
2. Rechtsklick auf `index.html` → *Open with Live Server*

**Mit Python**

```bash
python -m http.server 5500
# danach http://localhost:5500 öffnen
```

---

## Liniennetz neu erzeugen

`lines.geojson` entsteht aus dem offiziellen GTFS-Datensatz des KVV. Da dieser keine `shapes.txt`
enthält, wird der Linienverlauf aus der Haltestellenreihenfolge der **häufigsten Fahrt** je Linie
rekonstruiert und auf den Raum Karlsruhe zugeschnitten.

1. GTFS herunterladen: <https://projekte.kvv-efa.de/GTFS/google_transit.zip>
2. Entpacken und das Aufbereitungsskript ausführen
3. Ergebnis als `lines.geojson` neben `index.html` legen

---

## Bekannte Einschränkungen

- Der Linienverlauf folgt der Haltestellenreihenfolge, nicht der exakten Gleisgeometrie – in engen
  Kurven wirkt die Linie dadurch begradigt.
- Auf gemeinsamen Streckenabschnitten (z. B. Kaiserstraße) überlagern sich Linien; sichtbar ist die
  oberste.
- Barrierefreiheitsdaten stammen aus OpenStreetMap und sind nicht flächendeckend gepflegt.
- Die Zuordnung Meldung → Haltestelle erfolgt über die API-Angabe `affected.stops`, ergänzt um einen
  Textabgleich als Rückfallebene.

---

## Roadmap

- [ ] Parallel versetzte Linien auf gemeinsamen Abschnitten (Netzplan-Optik)
- [ ] Exakte Gleisgeometrie über OpenStreetMap
- [ ] Vergleich Soll-/Ist-Linienweg: gesperrte Abschnitte gestrichelt darstellen
- [ ] Barrierefreiheit zusätzlich aus offenen Daten der Stadt Karlsruhe
- [ ] Favoriten-Haltestellen (lokal gespeichert)

---

## Lizenz

MIT – siehe `LICENSE`. Die verwendeten Daten unterliegen den Lizenzen der jeweiligen Anbieter
(siehe *Datenquellen*).

Dieses Projekt steht in keiner Verbindung zum Karlsruher Verkehrsverbund.
