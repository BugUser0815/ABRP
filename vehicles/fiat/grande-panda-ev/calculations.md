# Berechnungsgrundlagen

## Bytes und Vorzeichen

Bei `22D523` ist eine positive Antwort beispielsweise `62 D5 23 28 9C`. `62` ist die positive Antwort auf Service `22`; `D5 23` ist die DID. Erst danach beginnen die Formelbytes: `A=0x28`, `B=0x9C`. CAN-Adresse, Längen-/Transportinformationen und Antwortkennung gehören nicht zu `A` und `B`.

Für zwei vorzeichenlose Bytes gilt:

```text
u16 = A × 256 + B
```

Für einen vorzeichenbehafteten 16-Bit-Wert:

```text
s16 = Signed(A) × 256 + B
```

ABRP behandelt `Signed(A)` laut Iternio-Dokumentation als vorzeichenbehaftetes 8-Bit-Byte. Bei `A ≥ 128` ist dessen Wert somit `A−256`. `Signed()` muss auf das obere Byte angewendet werden, bevor es mit 256 multipliziert wird. Ein negativer Rohwert ist rechnerisch darstellbar; die physische Vorzeichenrichtung beim Laden/Rekuperieren wurde noch nicht praktisch bestätigt.

## Batteriespannung – VCU `22D523`

```text
U[V] = u16 / 32
```

Beispiel `28 9C`: `10396 / 32 = 324,875 V`. Die Formel wurde bei der nativen Anbindung verwendet; die Anzeige lag im Stand je nach Last ungefähr zwischen 324 und 332 V.

## Batteriestrom und Leistung – VCU `22D552`

Aktueller Konfigurationsstand:

```text
I[A] = s16 / 10
P[kW] = I[A] × U[V] / 1000
```

**Der Stromdivisor ist 10; der Spannungsdivisor bleibt 32.** Die bisherige Stromformel mit `/32` darf nicht aus dem Spannungs-PID abgeleitet werden.

Mit `/32` reagierte die Anzeige zwar auf die Heizung, zeigte nach Nutzerbeobachtung bei voller Beschleunigung aber nur 25…28 kW. Der Standtest belegte nur eine Laständerung: Die tatsächliche Heizleistung war nicht unabhängig gemessen worden. Die damalige Bewertung als korrekte Leistungsberechnung war daher zu weitgehend.

Die Korrektur von `/32` auf `/10` erhöht Strom und daraus berechnete Leistung um:

```text
32 / 10 = 3,2
25 kW × 3,2 = 80 kW
28 kW × 3,2 = 89,6 kW
```

Ein zunächst erwogener Faktor 3 (`s16 × 3 / 32`) wäre gleichbedeutend mit einem Divisor von 10,666…; gespeichert wurde letztlich `/10`. Der Nutzer bestätigte am 07.11.2026, dass die Leistung damit beim Fahren passt. Die Zahlen 80…89,6 kW sind eine Rückrechnung der alten Anzeige, keine neu aufgezeichneten Messwerte.

Rechenbeispiel aus einem früheren stationären Test: `00 C9` entspricht `s16=201`. Mit `/10` ergibt das **20,1 A**. Zusammen mit `324,875 V` wären das **6,5299875 kW**. Die seinerzeit mit `/32` angezeigten `6,28125 A` und `2,04062109375 kW` sind historische Werte der alten Formel und kein weiterer Test der korrigierten Formel.

Die Leistungsanzeige beschreibt die aus der Batterie entnommene bzw. zugeführte elektrische Leistung, sofern das Stromsignal korrekt interpretiert ist. Ein Vergleich mit der mechanischen Motornennleistung allein ersetzt keine Messung; Nebenverbraucher und Umwandlungsverluste sind darin nicht identisch abgebildet.

## Ladezustand – VCU `22D44C`

```text
SoC[%] = u16 × 100 / 8192
```

Der VCU-Wert wurde für die ABRP-SoC-Anzeige gewählt. Bei der Einrichtung wurden unter anderem rund 79 % und später 74 % nativ angezeigt. Die Skalierung wurde anhand der Fahrzeug-/App-Anzeige verwendet, nicht über einen vollständigen Lade- und Entladezyklus kalibriert. Ein BMS-interner SoC ist nicht automatisch mit der Cockpitanzeige identisch.

## Batterietemperatur – BMS `22D4ED`

```text
T[°C] = A − 40
```

Beispiel `2E`: `46−40 = 6 °C`. Diese Größenordnung wurde in Car Scanner und in ABRP beobachtet; die native Temperatur stieg im weiteren Verlauf auf 7…8 °C. Die genaue räumliche Zuordnung des BMS-Temperatursignals ist nicht unabhängig dokumentiert; daraus wird keine minimale oder maximale Zelltemperatur abgeleitet.

Der vorher verwendete Befehl `22D4DF` mit Division durch 100 lieferte ungefähr 33. Das war kein belastbarer Temperaturwert. Diese alte Zuordnung wurde entfernt.

## SoH und Degradation – BMS `22D409`

```text
SoH[%] = A
Degradation[%] = 100 − SoH[%]
```

Beispiel `63`: dezimal 99, also **99 % SoH**. ABRP zeigte daraus **1 % Degradation** mit Quelle OBD und frischem Zeitstempel. Das ist der vom BMS gemeldete Gesundheitswert, keine separat durchgeführte Kapazitätsmessung.

## Verbleibende Energie – BMS `22D511`

```text
SoE[Wh]  = u16 × 32
SoE[kWh] = u16 × 0,032
```

**Im getesteten nativen ABRP-PID-Editor ist `soe` in Wh einzutragen.** Die Oberfläche zeigt kWh an. Diese Beobachtung bezieht sich auf ABRP iOS 7.1.13 (6060); sie ist keine allgemeine Aussage zur Einheit aller Versionen oder externer Telemetrie-APIs.

Die Skalierung wurde mit der aufgezeichneten Car-Scanner-Größe `[BMS] Traction battery Actual Energy` in Wh abgeglichen:

| Nutzdaten hex | Dezimal | Ergebnis mit ×32 | Beobachtung in Car Scanner |
| --- | --- | --- | --- |
| `03 A8` | 936 | 29.952 Wh | 29.952 Wh |
| `03 A9` | 937 | 29.984 Wh | 29.984 Wh |
| `03 A7` | 935 | 29.920 Wh | 29.920 Wh |
| `03 A6` | 934 | 29.888 Wh | 29.888 Wh |
| `03 A5` | 933 | 29.856 Wh | 29.856 Wh |
| `03 A4` | 932 | 29.824 Wh | 29.824 Wh |
| `03 A3` | 931 | 29.792 Wh | 29.792 Wh |
| `03 A2` | 930 | 29.760 Wh | 29.760 Wh |

Dies ist ein Vergleich der Werte und ihrer 32-Wh-Schritte, keine zeitlich synchronisierte zweite Messung jedes einzelnen Telegramms. Ein späterer nativer PID-Test ergab **27.552 Wh = 27,552 kWh**. ABRP zeigte gerundet **28 kWh**, Quelle **OBD**, Alter **<1 min**.

Die zunächst eingetragene kWh-Formel `u16 × 0,032` ergab im PID-Test zwar 27,712, in den Live-Daten aber **0 kWh**. Nach Umstellung auf `u16 × 32` erschien die korrekte Größenordnung. Die Division durch 1000 erfolgt für die Anzeige; sie gehört hier nicht in die gespeicherte native `soe`-Formel.

SoE wird direkt aus dem BMS-Signal gelesen, nicht aus `SoC × Nennkapazität` berechnet. Der Abgleich bestätigt die Übereinstimmung mit Car Scanner und die Darstellung in ABRP; die absolut nutzbare Energie bis zur Fahrabschaltung wurde nicht durch einen Vollzyklus bestimmt.

## Belege und Grenzen

Die Formelsyntax und Befehlsreihenfolge folgen der [Iternio-Dokumentation](https://github.com/Iternio-Planning-AB/ev-obd-pids). Die konkrete Zuordnung dieses Fahrzeugs beruht auf den dokumentierten Beobachtungen.

Ein [Erfahrungsbericht zum ë-C3/Frontera](https://www.c3forum.de/thread/227-live-daten-ueber-obd2-dongle-und-carscanner/?pageNo=12) nennt ebenfalls einen Faktor 3,2 bei Car-Scanner-Verbrauchswerten. Das war ein Hinweis bei der Fehlersuche, keine Herstellerdokumentation für die DID `D552`. Entscheidend für den aktuellen Prüfstand ist die nachfolgende Fahrbestätigung des Nutzers.
