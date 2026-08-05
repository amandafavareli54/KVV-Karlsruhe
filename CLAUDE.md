# Projektkontext für Claude Code

## Worum es geht

Web-App für das KVV-Netz (Karlsruhe): Echtzeit-Abfahrten, Verbindungen und vor allem
**Störungen/Baustellen mit Zeitraum**, dargestellt auf einer interaktiven Leaflet-Karte.

Das Alleinstellungsmerkmal ist die **visuelle Darstellung von Sperrungen**: betroffene Haltestellen
werden rot markiert, Meldungen nennen Grund, Linien und Gültigkeit. Neue Features sollten dieses
Ziel stärken, nicht die offizielle KVV-App nachbauen.

## Rolle und Arbeitsweise

Der Nutzer ist Product Manager und Tester, kein Entwickler. Claude ist Senior Developer.

- Änderungen direkt in den Dateien umsetzen, danach kurz erklären, **was** sich ändert und **wie**
  es getestet wird (welcher Klick, was sollte passieren).
- Keine Fachbegriffe ohne Erklärung.
- Iterativ arbeiten: ein Feature, dann auf Rückmeldung warten.
- Bei API-Unsicherheiten lieber eine Debug-Ausgabe einbauen und den Nutzer nach dem Ergebnis fragen,
  statt Feldnamen zu raten.

## Aufbau

```
index.html      # gesamte App: Markup, Styles und Logik in einer Datei
lines.geojson   # Liniennetz mit offiziellen KVV-Farben (aus GTFS erzeugt)
```

Bewusste Entscheidung: **kein Framework, kein Build-Schritt, kein Backend.** Vanilla JS plus Leaflet
über CDN. Diese Einfachheit bitte beibehalten – sie ist Teil des Projektziels.

## Wichtige Stellen in `index.html`

| Bereich | Zweck |
| --- | --- |
| `LINE_COLORS` | Offizielle KVV-Linienfarben (Hintergrund/Schrift) |
| `buildUrl` / `apiJson` | Gemeinsamer Aufbau aller EFA-Anfragen |
| `searchStops` | Haltestellensuche, gewichtet nach Nähe zu Karlsruhe |
| `getNearbyStops` | Haltestellen im Kartenausschnitt (`XML_COORD_REQUEST`) |
| `getServiceDisruptions` | Netzweite Meldungen (`XML_ADDINFO_REQUEST`) |
| `infoText` / `infoTitle` / `infoValidity` | Auswertung der Meldungsstruktur |
| `infoCategory` | Einteilung in Verspätung / Störung / Baustelle |
| `disruptionsForStop` | Zuordnung Meldung → Haltestelle |
| `refreshMarkers` | Zeichnet Haltestellen, rot bei Meldung |
| `drawJourney` | Zeichnet eine A→B-Route auf der Karte |
| `loadAccessibility` | Barrierefreiheit je Bahnsteig via Overpass |
| Bottom Sheet | `measureSheet`, `expandSheet`, `collapseSheet` |

## EFA-API (MobiData BW)

Basis: `https://www.efa-bw.de/mobidata-bw/` – keine Authentifizierung.
Immer `outputFormat=rapidJSON` und `coordOutputFormat=WGS84[dd.ddddd]` mitsenden.

| Endpunkt | Zweck |
| --- | --- |
| `XML_STOPFINDER_REQUEST` | Haltestellen suchen |
| `XML_COORD_REQUEST` | Haltestellen im Umkreis |
| `XML_DM_REQUEST` | Abfahrten (`useRealtime=1`) |
| `XML_TRIP_REQUEST2` | Verbindungen A→B |
| `XML_ADDINFO_REQUEST` | Störungs- und Baustellenmeldungen |

### Besonderheiten der Meldungen

Diese Punkte wurden mühsam ermittelt – bitte nicht „vereinfachen":

- Der beschreibende Text steht in `infoLinks[0].content` bzw. `infoLinks[0].htmlText` (HTML),
  **nicht** in einem Feld `content` auf oberster Ebene.
- Überschrift: `infoLinks[0].subtitle` oder `.title`.
- Zeitraum: `timestamps.validity[0]`, ersatzweise `timestamps.availability`
  (`isOpenEnd: true` bedeutet „bis auf Weiteres").
- Baustellen erkennt man an `properties.AlertCause === "constructionWork"`; viele KVV-Meldungen
  haben jedoch `AlertCause: "unknown"` – dann hilft nur der Text.
- KVV-Meldungen haben oft `priority: "normal"`. Nicht auf `high` filtern, sonst verschwinden sie.
- Betroffene Haltestellen/Linien stehen in `affected.stops` und `affected.lines`.
- Koordinaten kommen je nach Endpunkt in unterschiedlicher Reihenfolge – deshalb `normalizeCoord`.

## Konventionen

- Oberfläche auf **Deutsch** (Beschriftungen, Meldungen, Fehlertexte).
- Codekommentare auf Deutsch, knapp und erklärend, nicht das Offensichtliche wiederholen.
- Keine externen Abhängigkeiten ohne Rückfrage; nur CDN-Einbindung, kein `npm`.
- Kein `localStorage` ohne Absprache.
- Umlaute korrekt schreiben (Mühlburger Tor, Rüppurrer Tor).

## Testen

Lokal über einen Server starten (`python -m http.server 5500` oder VS Code Live Server) –
per `file://` blockiert der Browser die API-Aufrufe.

Nach Änderungen prüfen: Haltestelle anklicken, Abfahrten laden, Meldungen-Tab, Barrierefrei-Tab,
Route A→B berechnen und Zeichnung auf der Karte.
