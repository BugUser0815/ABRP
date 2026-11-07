# Vorlage für weitere Fahrzeuge

Ein eigener Ordner `vehicles/<hersteller>/<modell>/` hält die Ergebnisse je Fahrzeug getrennt. Ergänze den Eintrag in der Fahrzeugtabelle der Haupt-README. Unterschiede zwischen Baujahren, Batterien und Steuergeräteversionen müssen ausdrücklich dokumentiert werden.

## README.md

- Fahrzeug, Antriebs-/Batterievariante und bekannte Softwarestände.
- Adapter, Verbindung, App-/Betriebssystemversion und Testdatum.
- Protokoll, Initialisierung, Keep-alive und geordneter Datenabruf.
- Je PID: Anfrage-/Antwortadresse, Service/DID, ABRP-Schlüssel, Formel, Einheit, Datentyp, bekannte Grenzen und Prüfstand.
- Einrichtung und Quellen; keine ungeprüfte Übertragbarkeit von verwandten Fahrzeugen behaupten.

## pid-config.json

Die JSON-Datei des Grande Panda dient als Beispiel für das Dokumentationsformat. Reihenfolge und Steuergerätewechsel gehören zur Konfiguration. Unbekannte Grenzen nicht durch Standardwerte ersetzen; aktuelle Einheiten des tatsächlich verwendeten nativen Editors erfassen. Ein Importversprechen setzt einen geprüften Importweg voraus.

## calculations.md

- Nutzdatenbytes von CAN-/Transport- und UDS-Antwortkennung unterscheiden.
- Endianness, Vorzeichen, Offset und Skalierung erklären.
- Mindestens ein bereinigtes Rohwertbeispiel je belegter Formel angeben.
- Einheiten im Fahrzeugsignal, im Editor und in der Anzeige getrennt aufführen.
- Abgeleitete Größen mit Formel und Abhängigkeiten dokumentieren.
- Eine plausible Anzeige, einen Vergleich mit einer anderen App und eine unabhängige Messung als unterschiedliche Belege behandeln.

## validation.md

Je Test Datum, Fahrzeugzustand, Konfigurationsversion, Quelle, Beobachtung und Aussagegrenze erfassen. Direkt beobachtete Ergebnisse, Nutzerberichte und bloße Rückrechnungen getrennt kennzeichnen. Historische Werte mit fehlerhaften Formeln nicht als Bestätigung späterer Korrekturen verwenden.

Laden, Rekuperation, Wiederverbindung und unterschiedliche Betriebszustände erhalten jeweils einen eigenen Prüfstand. Offene Punkte bleiben sichtbar, bis passende Belege vorliegen.

Veröffentliche ausschließlich bereinigte Auszüge: keine Zugangstoken, VIN, vollständigen App-Backups, Gerätekennungen oder Standort-/Fahrtspuren.
