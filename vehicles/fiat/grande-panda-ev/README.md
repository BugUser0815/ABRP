# Fiat Grande Panda EV – native ABRP-Anbindung

## Ergebnis und Prüfstand

ABRP liest SoC, Batteriespannung, Batteriestrom, Batterietemperatur, SoH und verbleibende Energie direkt vom OBDLink CX. Die Leistung wird aus Strom und Spannung berechnet. Car Scanner wird für den laufenden Betrieb dieser nativen Verbindung nicht benötigt.

Die Stromformel wurde von `/32` auf **`/10`** korrigiert. Der Nutzer bestätigte am **07.11.2026**, dass die Leistung damit im Fahrbetrieb passt. Konkrete neue Messwerte oder ein Fahrprotokoll wurden dazu nicht übermittelt. Rekuperation und Laden sind weiterhin nicht geprüft.

| Angabe | Dokumentierter Stand |
| --- | --- |
| Fahrzeug | Fiat Grande Panda EV; genaue Modelljahr-/Steuergerätevariante nicht erfasst |
| Adapter | OBDLink CX, Bluetooth LE |
| ABRP | iOS-App 7.1.13 (6060) |
| Testumgebung | iPhone 13 Pro Max, iOS 27.0 |
| Fahrzeugzustand bei direkter Beobachtung | Stehend, READY; zeitweise Innenraumheizung auf maximaler Wärme |
| Protokoll | 6, ISO 15765-4 CAN mit 11-Bit-Adressen und 500 kbit/s |
| Keep-alive | Leer |
| Konfiguration zuletzt geändert | 06.11.2026, 15:28 Uhr, Europe/Berlin |

Zum Vergleich wurde Car Scanner 2.2.42 mit dem Profil `Citroen e-C3 CC21 (2024-) / Fiat Grande Panda EV (2025-) / Opel Frontera EV (2024-)` verwendet. Nur am genannten Grande Panda geprüft; keine Freigabe für die anderen Fahrzeuge dieses Profils.

## PID-Zuordnung

`A` und `B` bezeichnen die Nutzdatenbytes nach der positiven UDS-Antwortkennung und der DID. Herleitung und Beispiele stehen in [calculations.md](calculations.md).

| Steuergerät / Anfrage → Antwort | Anfrage | ABRP-Schlüssel | Formel im nativen Editor | Einheit | Prüfstand |
| --- | --- | --- | --- | --- | --- |
| VCU, `6A2` → `682` | `22D523` | `voltage` | `(A*256+B)/32` | V | Native Werte beobachtet |
| VCU, `6A2` → `682` | `22D552` | `current` | `(Signed(A)*256+B)/10` | A | Divisor 10 im Editor erhalten; Leistung später vom Nutzer beim Fahren bestätigt |
| VCU, `6A2` → `682` | `22D44C` | `soc` | `(A*256+B)*100/8192` | % | Native Übernahme beobachtet; keine unabhängige Kalibrierung |
| BMS, `6B4` → `694` | `22D4ED` | `batt_temp` | `A-40` | °C | Native Werte und Vergleich mit Car Scanner beobachtet |
| BMS, `6B4` → `694` | `22D409` | `soh` | `A` | % | 99 % SoH; ABRP zeigt 1 % Degradation |
| BMS, `6B4` → `694` | `22D511` | `soe` | `(A*256+B)*32` | **Wh** | Rohdaten/Car-Scanner-Aufzeichnung abgeglichen; native Anzeige bestätigt |

ABRP berechnet `power` aus `current × voltage / 1000` in kW. Dafür wurde kein eigener Leistungs-PID hinzugefügt. Beobachtete Editorgrenzen: Spannung **200…450 V**, Strom **−300…300 A**. Grenzen für die übrigen Einträge sind nicht vollständig dokumentiert; die JSON-Referenz ergänzt dort keine geratenen Werte. Die Eignung der Stromgrenzen bei Laden und Rekuperation ist noch zu prüfen.

## Initialisierung und Reihenfolge

Die vollständige Reihenfolge steht in [pid-config.json](pid-config.json). Die Abfolge und der Wechsel der Antwortfilter sind Teil der Konfiguration.

Initialisierung, 13 Befehle:

```text
ATZ
ATD
ATE0
ATH1
ATAT1
ATL0
ATS0
ATM0
ATSP6
ATSH6B4
ATFCSH6B4
ATFCSD300000
ATFCSM1
```

Zyklischer Abruf, 13 Befehle:

```text
ATSH6A2
ATFCSH6A2
ATCRA682
22D523
22D552
22D44C
ATAR
ATSH6B4
ATFCSH6B4
ATCRA694
22D4ED
22D409
22D511
```

`ATSH` setzt die Anfrageadresse, `ATFCSH` die Adresse für Flow-Control-Frames und `ATCRA` den Antwortfilter. `ATAR` hebt den bisherigen Empfangsfilter vor dem Wechsel zum BMS auf. `ATFCSD300000` und `ATFCSM1` konfigurieren die Flow-Control-Behandlung des Adapters. Die verwendeten UDS-Anfragen mit Service `22` lesen Daten; es wurden keine Steuergeräteparameter geschrieben.

## Im ABRP-Editor einrichten

1. Car Scanner vom Adapter trennen. Gleichzeitige Abfragen beider Apps führten im Versuch zu vermischten Antworten und unplausiblen Werten.
2. In ABRP das richtige Fahrzeug wählen und **Live-Daten → Bearbeiten → OBD-PID-Editor** öffnen. Die Bezeichnungen können sich mit der App-Version ändern.
3. Protokoll 6, leeren Keep-alive, Initialisierung und geordnete Datenbefehle aus `pid-config.json` übernehmen. Jedem Messbefehl den Schlüssel und die Formel aus der Tabelle zuweisen; Datentyp jeweils **Number**.
4. In Fahrzeugnähe die PIDs testen, die verfügbaren Ergebnisse prüfen und die Konfiguration über die angebotene Speicherfunktion übernehmen. Falls erforderlich, die OBD-Verbindung anschließend neu aufbauen.
5. In **Live-Daten** Quelle **OBD** und einen frischen Zeitstempel kontrollieren. Ein älterer Wert von Car Scanner oder einer anderen Quelle bestätigt die neue Konfiguration nicht.

Bei der letzten Stromänderung war der Adapter nicht erreichbar: Der Test scheiterte am Verbindungsaufbau und die Schaltfläche **Speichern** blieb deaktiviert. Dass `/10` nach Schließen des gesamten Editors und erneutem Öffnen erhalten blieb, wurde geprüft. Die anschließende Bestätigung der Leistung beim Fahren stammt vom Nutzer. Dieser Unterschied ist im [Prüfprotokoll](validation.md) festgehalten.

## Quellen

- [ABRP: OBD PID Editor](https://abrp.featurebase.app/help/articles/2478825-obd-pid-editor) – Einrichtung und Testablauf.
- [Iternio: ev-obd-pids](https://github.com/Iternio-Planning-AB/ev-obd-pids) – Befehlsreihenfolge und unterstützte Formelausdrücke, insbesondere `Signed()`.
- [Car Scanner: Profiländerungen](https://www.carscanner.info/profile-database-changelog/) – Einordnung des gemeinsamen Fahrzeugprofils.
- Eigene direkte Beobachtungen im ABRP-Editor und in den Live-Daten, bereinigte Antworten aus einem Car-Scanner-Diagnoselog sowie dessen gespeicherte Energieaufzeichnung vom 06.11.2026. Rohdateien und Screenshots sind nicht veröffentlicht.
