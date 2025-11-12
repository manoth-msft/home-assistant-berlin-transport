# 🚉 Abfahrtszeiten im Berliner und Brandenburger ÖPNV (BVG/VBB) für Home Assistant

Diese Integration bringt **Live-Daten des öffentlichen Nahverkehrs** aus Berlin und Brandenburg direkt auf dein Home Assistant-Dashboard. Sie nutzt die offizielle VBB-API, um Echtzeit-Abfahrten von BVG- und VBB-Haltestellen abzurufen — einschließlich Liniennummern, Zielhaltestellen, Abfahrtszeiten und Verspätungen.

Egal ob du pendelst, die Kinder abholst oder dich einfach fragst, wann die nächste Ringbahn kommt:  
Diese Integration zeigt bevorstehende Abfahrten deiner ausgewählten Haltestellen in einem übersichtlichen, gut lesbaren Format.

> 🛠️ Dieses Projekt ist ein Fork der ursprünglichen Berlin-Transport-Integration von [vas3k](https://github.com/vas3k/home-assistant-berlin-transport) — erweitert um Filterfunktionen, Anpassungsmöglichkeiten und unabhängige Pflege.

![Beispielhafte Darstellung von Live-Abfahrten am Bahnhof S+U Gesundbrunnen in Berlin, ähnlich wie im Home Assistant-Dashboard.](./screenshots/timetable_card2.jpg)  
![Weiteres Beispiel](./screenshots/timetable_card3.jpg)  
![Weiteres Beispiel](./screenshots/timetable_card1.jpg)


## 💿 Installation

Diese Integration besteht aus zwei Komponenten:

1. **Sensor** – Ruft alle 90 Sekunden Echtzeit-Abfahrtsdaten von der [VBB Public API](https://v6.vbb.transport.rest/api.html#get-stopsiddepartures) ab. Dies ist das Repository, das du gerade betrachtest.
2. **Lovelace-Karte** – Zeigt bevorstehende Abfahrten in einem übersichtlichen, dashboardfreundlichen Format an. Sie befindet sich in einem [separaten Repository von vas3k](https://github.com/vas3k/lovelace-berlin-transport-card).

🔧 **Empfohlene Einrichtung**: Installiere beide Komponenten über [HACS](https://hacs.xyz/), um einfache Updates und eine nahtlose Integration in Home Assistant zu ermöglichen.

### Sensor-Komponente über HACS installieren

1. Füge dieses [Repository](https://github.com/vas3k/home-assistant-berlin-transport) als benutzerdefiniertes Repository in HACS hinzu (Kategorie: *Integration*).
2. Füge unter `Einstellungen` → `Geräte & Dienste` die Integration **Berlin (BVG) and Brandenburg (VBB) transport** hinzu.
3. Suche nach deiner Haltestelle. Teilweise Übereinstimmungen sind möglich – bis zu 15 relevante Haltestellen werden angezeigt.
4. Wähle die Haltestelle aus, die du überwachen möchtest.
5. *(Optional)* Konfiguriere zusätzliche Parameter:
   - **Richtung**: Verwende `stop_id`, um Abfahrten nach Richtung zu filtern. Gib die `stop_id` einer Zielhaltestelle oder eines Zwischenstopps entlang der gewünschten Linie an. Mehrere Werte sind als kommaseparierte Liste möglich. Siehe [unten](#how-do-i-find-my-stop_id), wie du die `stop_id` findest.
   - **Haltestellen ausschließen**: Liste von `stop_id`, die ausgeschlossen werden sollen. Nützlich, wenn BVG/VBB auch nahegelegene Haltestellen zurückliefert. Mehrere Werte sind als kommaseparierte Liste möglich.
   - **Zeitraum**: Gibt an, wie viele Minuten in die Zukunft Abfahrten abgerufen werden sollen. Standard: 10 Minuten.
   - **Gehzeit**: Zeit in Minuten, die du zur Haltestelle brauchst. Verhindert die Anzeige von Abfahrten, die du nicht rechtzeitig erreichst.
   - **Offizielle VBB-Linienfarben aktivieren**: Optional kannst du die offiziellen Linienfarben verwenden. Standardmäßig werden vordefinierte Farben genutzt.
   - **Ringbahn ⟳/⟲ ausblenden**: Optional kannst du Ringbahn-Fahrten ausblenden, die im Uhrzeigersinn oder gegen den Uhrzeigersinn verlaufen.  
     - Beispiel: Du möchtest Abfahrten von *Treptower Park* überwachen und setzt den Richtungsfilter auf *900077106 (S Sonnenallee)*, weil du nur Züge sehen willst, die im Uhrzeigersinn fahren. Die Ringbahn S42 ⟲ (gegen den Uhrzeigersinn) erreicht jedoch ebenfalls *S Sonnenallee*, sodass die API auch diese Abfahrten zurückliefert. Diese Option blendet solche Einträge aus.
   - **Suffix *(Berlin)* entfernen**: Die BVG hängt bei manchen Haltestellen den Zusatz „(Berlin)“ an. Diese Option entfernt den Zusatz automatisch.
   - **Transportarten**: Wähle aus, welche Verkehrsmittel (z. B. Bus, Fähre) angezeigt oder ausgeblendet werden sollen.
6. Fertig. Wenn du später Optionen ändern möchtest, führe die Schritte einfach erneut mit derselben Haltestelle aus. Die vorherige Entität wird automatisch überschrieben.

### Sensor-Komponente manuell installieren

Die Schritte zur manuellen Installation sind [hier beschrieben](./docs/manual_install.md).

### Lovelace-Karte hinzufügen

Wechsle zum Repository [lovelace-berlin-transport-card](https://github.com/vas3k/lovelace-berlin-transport-card) und folge dort den Installationsanweisungen.

## 👩‍💻 Technische Details

Diese Sensor-Komponente verwendet die VBB Public API, um alle Verkehrsdaten abzurufen.

- API-Dokumentation: [https://v6.vbb.transport.rest/api.html](https://v6.vbb.transport.rest/api.html)
- Rate-Limit: 100 Anfragen pro Minute
- Datenformat: [HAFAS](https://github.com/public-transport/hafas-client)

Die Komponente aktualisiert sich alle 60–90 Sekunden und sendet dabei für jede konfigurierte Haltestelle eine eigene Anfrage. Das reicht in der Regel aus, aber es wird nicht empfohlen, Dutzende Haltestellen gleichzeitig zu konfigurieren, da sonst das Rate-Limit überschritten werden kann.

Die VBB API ist gelegentlich instabil (wie du dir denken kannst) und liefert manchmal zufällige 503- oder Timeout-Fehler. Das ist normal. Ich habe bisher keinen Weg gefunden, das zu umgehen — es verursacht aber keine funktionalen Probleme, sondern nur Warnungen im Log.

Nach dem Abruf der API wird für jede Haltestelle eine eigene Entität erstellt. In `attributes.departures` werden bis zu 10 bevorstehende Abfahrten gespeichert. Der Entitätszustand selbst wird nicht aktiv verwendet — er zeigt lediglich die nächste Abfahrt in menschenlesbarer Form.  
Wenn du Ideen hast, wie man das besser nutzen könnte: gerne [ein Issue eröffnen](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/issues)!

> 🤔  
> Grundsätzlich ist das HAFAS-Format auch in vielen anderen Städten standardisiert, sodass du diese Komponente problemlos an andere Orte anpassen kannst. Inspiration findest du unter [transport.rest](https://transport.rest/).

## ❤️ Beiträge

Beiträge sind willkommen! Du kannst gerne einen [Pull Request](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/pulls) öffnen und zur Überprüfung einreichen.  
Falls du dir unsicher bist, eröffne ein [Issue](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/issues) und frage nach Rat.

## 🐛 Fehlerberichte und Feature-Wünsche

Da dies ein kleines Hobbyprojekt ist, kann ich leider keinen vollständigen Support oder Hilfe bei der Dashboard-Konfiguration garantieren. Ich hoffe auf dein Verständnis.

- **Wenn du einen Fehler findest** – eröffne ein [Issue](https://github.com/manoth-msft/home-assistant-bvg-vbb-departures/issues) und beschreibe die genauen Schritte zur Reproduktion. Screenshots, Logs und weitere Details helfen bei der Fehlersuche.
- **Wenn dir ein bestimmtes Feature fehlt**, beschreibe es in einem Issue oder versuche, es selbst zu implementieren.

## 👮‍♀️ Lizenz

- [MIT](./LICENSE.md)
