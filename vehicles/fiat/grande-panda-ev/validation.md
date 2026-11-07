# Prüfprotokoll und offene Punkte

Dokumentiert am **07.11.2026**. Messungen und Änderungen fanden am **06.11.2026** statt; Uhrzeiten unten gelten für Europe/Berlin.

## Status je Messgröße

| Messgröße | Befund | Aussagegrenze |
| --- | --- | --- |
| SoC | Native OBD-Anzeige beobachtet, z. B. 74 % | Kein unabhängiger Lade-/Entladezyklus |
| Spannung | Native OBD-Werte beobachtet, z. B. 325 V | Keine parallele externe Spannungsmessung |
| Strom / Leistung mit `/10` | Editorwert nach Schließen/Öffnen erhalten; Nutzer bestätigt am 07.11.2026 die Leistung beim Fahren | Keine neuen Zahlen oder Fahr-Rohdaten übermittelt; Laden/Rekuperation offen |
| Temperatur | 6…8 °C während der Einrichtung, Quelle OBD; Vergleich mit Car Scanner | Kein unabhängiger Temperaturfühler; genaue Sensorposition nicht belegt |
| SoH / Degradation | 99 % / 1 %, Quelle OBD, <1 min | BMS-Angabe; keine unabhängige Kapazitätsmessung |
| Restenergie | Rohwertabgleich mit aufgezeichneter Car-Scanner-Energie; 27.552 Wh im nativen Test, 28 kWh in Live-Daten | Kein Vollzyklus zur Bestimmung der absolut nutzbaren Energie |

## Ablauf

### Verbindung und Trennung der Datenquellen

Car Scanner erreichte anfangs den OBDLink CX, aber nicht zuverlässig das Steuergerät. Nach erneutem Einstecken des Adapters und Umstellung der ECU-Initialisierung von **Custom** auf **Standard** kam die Verbindung zustande. Zusätzliche manuelle Test-/Keep-alive-Einträge wurden anschließend entfernt.

Bei gleichzeitigem Adapterzugriff durch ABRP und Car Scanner traten vermischte Antworten und unplausible Werte auf. Für Vergleiche wurde deshalb jeweils nur eine App mit dem Adapter verbunden. Zunächst wurden Werte über die Car-Scanner-Integration an ABRP übertragen; anschließend wurde die direkte native Anbindung eingerichtet und die Car-Scanner-Quelle aus ABRP entfernt.

Die funktionierende Standardinitialisierung in Car Scanner und die hier dokumentierte explizite ABRP-Initialisierung sind unterschiedliche Konfigurationen. Die frühere fehlerhafte Car-Scanner-Custom-Initialisierung ist keine Vorlage für die native ABRP-Anbindung.

### Erste native Anbindung

Am 06.11.2026 gegen 13:07 Uhr zeigte ABRP über die direkte OBD-Verbindung unter anderem **79 % SoC**, **325 V** und **7 °C**. Der Strom-/Leistungswert reagierte auf die Innenraumheizung: zuvor 0 A / 0 kW, danach ungefähr 2,2 kW mit der damaligen Stromformel `/32`.

Dieser Test belegt die Verbindung und eine Lastabhängigkeit. Er bestätigte **nicht** die absolute Strom-/Leistungsskalierung. Die Heizleistung war nur geschätzt. Die nachfolgende Fahrbeobachtung widerlegte die frühere Bewertung als korrekte Leistungsberechnung.

### SoH / Degradation

`22D409`, Formel `A`, ergab 99 % SoH. Die Konfiguration wurde im erfolgreichen PID-Test gespeichert. Gegen 13:40 Uhr zeigte ABRP **1 % Degradation**, Quelle **OBD**, Alter **<1 min**.

### Restenergie

Die Rohwertvarianten von `22D511` wurden mit einer Car-Scanner-Aufzeichnung verglichen. Die Anzeige folgte genau der Skalierung **32 Wh je Zählschritt**. Nach Korrektur der nativen Eingabeeinheit von kWh auf Wh und erfolgreichem Test/Speichern zeigte ABRP gegen 13:54 Uhr:

| Feld | Anzeige |
| --- | --- |
| SoC | 74 % |
| Spannung | 325 V |
| Batterietemperatur | 8 °C |
| Degradation | 1 % |
| Restenergie | 28 kWh, gerundet; PID-Test zuvor 27.552 Wh |
| Verbindung | OBD verbunden; relevante Felder <1 min |

Die damalige Leistungsanzeige von ungefähr 2 kW gehört noch zur alten Stromformel und ist kein Referenzwert für `/10`.

### Strom-/Leistungskorrektur und Fahrbestätigung

1. Der Nutzer meldete am 06.11.2026 bei voller Beschleunigung lediglich **25…28 kW** in ABRP.
2. Als Kandidat wurde die Korrektur des Stromdivisors von **32 auf 10** identifiziert: Faktor **3,2** auf Strom und daraus abgeleitete Leistung.
3. Der Nutzer beauftragte die Änderung, während er nicht am Fahrzeug war. Ein kurzzeitig eingetragener Faktor 3 wurde noch während der Bearbeitung durch den Divisor 10 ersetzt.
4. Um 15:28 Uhr wurde nach Schließen des gesamten PID-Editors bis zur Karte und erneutem Öffnen die Formel **`(Signed(A)*256+B)/10`** wieder gelesen. Grenzen blieben **−300…300 A**.
5. Zu diesem Zeitpunkt war kein Live-Test möglich: Der Adapter war nicht erreichbar; die Verbindung wurde abgelehnt und die Test-Schaltfläche **Speichern** blieb deaktiviert. Es gab dabei keine frischen Fahrzeugmesswerte. Das Wiederlesen belegt die erhaltene Editorkonfiguration, keinen damaligen erfolgreichen Fahrzeugtest oder App-Neustarttest.
6. Am **07.11.2026** bestätigte der Nutzer, die Leistung inzwischen geprüft zu haben: Sie passt. Damit gilt die **Leistungsgröße beim Fahren als vom Nutzer bestätigt**. Ein detaillierter Messverlauf und neue numerische Spitzenwerte liegen nicht vor.

## Verworfene Zuordnungen

| Frühere Zuordnung | Warum sie nicht weiterverwendet wird |
| --- | --- |
| BMS `22D511` als Strom, `(A*256+B)/10-62` | Das Signal wurde anhand der Energieaufzeichnung als Restenergie mit 32 Wh je Zählschritt zugeordnet |
| BMS `22D4DF` als Temperatur, Division durch 100 | Liefert ungefähr 33 statt einer plausiblen Batterietemperatur; ersetzt durch `22D4ED` |
| VCU `22D552` als Strom mit `/32` | Zu geringe Leistung im Fahrbetrieb; ersetzt durch `/10` |
| SoE im nativen Editor als kWh | Führte zu 0 kWh in Live-Daten; Wh-Eingabe bestätigte die richtige Größenordnung |

Kilometerzähler und Ladestatus wurden nicht als neue Messgrößen in diese Konfiguration aufgenommen. Frühere zwischengespeicherte Anzeigen sind keine neu bestätigten PID-Zuordnungen.

## Noch zu prüfen

- Vorzeichen und Größenordnung von Strom/Leistung bei Rekuperation sowie bei AC- und DC-Laden.
- Verhalten der Grenze ±300 A bei hohen Lade-/Entladeströmen; bei verworfenen Werten oder scheinbar feststehender Anzeige die Grenzen prüfen.
- Abgleich mit unabhängigem Leistungs-/Energiebezug, beispielsweise einem geeigneten Ladevorgang und dessen dokumentiertem Verlauf.
- App-Neustart, automatische Wiederverbindung nach Fahrzeugruhe und längerer Betrieb.
- Verhalten bei niedrigerem SoC, anderen Batterietemperaturen und anderen Steuergeräte-/Softwareständen.

Für weitere Prüfungen genügen normale Fahrten bzw. geeignete Aufzeichnungen. Der Fahrer bedient dabei das iPhone nicht; ein Beschleunigungsversuch ist nicht Teil des Prüfplans.
