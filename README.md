# ABRP – Fahrzeugdaten über OBD

Dokumentierte PID-Konfigurationen, Berechnungsgrundlagen und praktische Prüfungen für die direkte OBD-Anbindung an A Better Routeplanner (ABRP).

## Fahrzeuge

| Fahrzeug | Dokumentation | Stand |
| --- | --- | --- |
| Fiat Grande Panda EV | [Konfiguration und Einrichtung](vehicles/fiat/grande-panda-ev/README.md) | Native Anbindung eingerichtet; Leistung mit Divisor 10 am 07.11.2026 vom Nutzer im Fahrbetrieb bestätigt |

## Aufbau

Jedes Fahrzeug erhält einen eigenen Ordner unter `vehicles/<hersteller>/<modell>/`:

- `README.md`: Fahrzeug, Umgebung, PID-Tabelle und Einrichtung.
- `pid-config.json`: geordnete Befehle und Formeln als maschinenlesbare Referenz zum ABRP-Editor.
- `calculations.md`: Byte-Auswertung, Einheiten und Herleitung der Formeln.
- `validation.md`: Beobachtungen, Korrekturen und noch offene Prüfungen.

Für weitere Fahrzeuge gibt es eine [Vorlage](docs/vehicle-template.md). Ein gemeinsames Fahrzeugprofil in einer anderen App belegt keine Übertragbarkeit auf weitere Modelle oder Baujahre.

## Wie die Angaben zu lesen sind

Die Dateien bilden die jeweils dokumentierte Konfiguration und ihren Prüfstand ab. Zwischen im Editor eingetragen, tatsächlich vom Fahrzeug empfangen und unabhängig bestätigt wird ausdrücklich unterschieden. Die JSON-Dateien sind ein Dokumentationsformat; ein direkter Dateiimport in ABRP wurde nicht geprüft.

Veröffentlicht werden bereinigte technische Ergebnisse. Vollständige Diagnoseprotokolle, Zugangstoken, Gerätekennungen und Standortdaten gehören nicht in dieses Repository.

Dokumentationsstand: **07.11.2026**. Fahrzeugmessungen und Konfigurationsänderungen: **06.11.2026**, mit Rückmeldung zur Fahrprüfung am **07.11.2026**.
