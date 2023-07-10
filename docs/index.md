# Aktuelle Trends der IKT

Herzlich willkommen zur Veranstaltung **Aktuelle Trends der IKT**! 

---

***[Zoom-Link](https://htw-berlin.zoom.us/j/95908963080?pwd=UElCL3JYZ1k0MmdURm13cDYxaitYQT09) für die Übungszeiten für Ihre Fragen und Probleme bei Ihrer App-Entwicklung.***

---

### Grober Inhalt

Wir beschäftigen uns dieses Semester mit *Progressive Web Apps (PWA)*. Dieser Begriff ist 2015 bei Google entstanden. Progressive Web Apps bieten installierbare nativen Apps ähnliche Nutzererfahrungen sowohl auf dem Desktop als auch auf dem Smartphone, sind aber Webanwendungen, die im Browser laufen, also zum World Wide Web gehören. Typische Eigenschaften von Progressive Web Apps sind die Einbindung von Kamera und Mikrofon, dem eigenen Standort sowie die Fähigkeit, (zumindest teilweise) offline ausführbar zu sein. 

Nachfolgend der vorläufige Wochenplan (wird eventuell angepasst). 

| | Woche | Themen (Vorlesung) |  Aufgabe (Stand) | Abgabe Übung bis | 
|-|-------|--------------------|-----------------|------------------|
| 1. | 10.-14.04.2023 | Einführung und Organisatorisches |  - | - | 
| 2. | 17.-21.04.2023 | Grundgerüst und Application Manifest |  - | 30.04.2023 | 
| 3. | 24.-28.04.2023 | Service workers |  - | 07.05.2023 | 
| 4. | 01.-05.05.2023 | Promises und Fetch API |  - | 14.05.2023 | 
| 5. | 08.-12.05.2023 | Service workers und Caching |  - | 21.05.2023 | 
| 6. | 15.-19.05.2023 | MongoDB und Backend |  - | 28.05.2023 | 
| 7. | 22.-26.05.2023 | Bilder-Up- und Download (Backend) |  - | 04.06.2023 | 
| 8. | 29.-02.06.2023 | IndexedDB |  - | 11.06.2023 | 
| 9. | 05.-09.06.2023 | Kamera |  - | 18.06.2023 | 
| 10. | 12.-16.06.2023 | Geolocation |  Datenbank | - | 
| 11. | 19.-23.06.2023 | Hintergrundsynchronisation |  Backend | - | 
| 12. | 26.-30.06.2023 | Push-Notifikationen |  Backend | - |
| 13. | 03.-06.07.2023 | Wiederholung |  Frontend | - |
| 14. | 10.-14.07.2023 | Wiederholung |  Frontend | - |
|  |  |  | Gespräche 1.PZ 26.07.2023 (Abgabe 25.07.2023) | - |
|  |  |  | Gespräche 2.PZ 27.09.2023 (Abgabe 26.09.2023) | - |

### Organisatorisches 

Zur erfolgreichen Durchführung der Veranstaltung müssen Sie am Ende des Semesters die Lösung Ihrer Semesteraufgabe abgeben. Diese Aufgabe zusammen mit einem Gespräch, das wir über Ihre Lösung führen, wird bewertet. Die Bewertung entspricht dann der Modulnote. 

Die Übungen sind dafür vorgesehen, dass Sie im Semester sukzessive Ihre Lösung erstellen können. Wir beantworten in den Übungen Ihre Fragen und lösen gemeinsam Probleme. Jede Woche gibt es ein Thema, das Sie selbständig durcharbeiten und dann angepasst in Ihre Lösung integrieren können.  

Für die Kommunikation untereinander verwenden wir [**Slack**](https://slack.com/intl/de-de/). Dort können Sie alle inhaltlichen und organisatorischen Fragen stellen. Ich fände es gut, wenn ich dort möglichst wenig Fragen - zumindest die inhaltlichen - beantworten müsste, sondern eine Art internes **Diskussionsforum** entsteht. Es ist sehr gewünscht, dort Fragen zu stellen und noch mehr gewünscht, diese **von Ihnen dort beantwortet** zu sehen. Damit wäre allen geholfen und ich kann besser erkennen, wo noch Nachhol- bzw. Erläuterungsbedarf bei den meisten besteht. Außerdem lernen Sie beim Beantworten der Fragen nochmals deutlich mehr. Das wäre super, wenn das klappt!


### Semesteraufgabe

Die als Semesteraufgabe zu entwickelnde Webanwendung sollte

1. ein Frontend besitzen (muss nicht mit einem JavaScript-Framework erstellt werden),
2. das Frontend soll responsive sein (*mobile first*!),
3. ein Backend (damit Daten auf dem Server verwaltet werden können), 
4. eine Datenbank zur persistenten Speicherung von Daten (wir verwenden MongoDB, kann aber auch MariaDB, MySQL, PostgresQL oder auch SQLite oder ähnlich In-Apps-Datenbanken sein - aber nicht Firebase!),
5. installierbar sein,
6. offline nutzbar sein,
7. die IndexedDB verwenden,
8. Hintergrundsynchronisation verwenden,
9. Push-Nachrichten verwenden,
10. die Gelocation API verwenden,
11. die Kamera oder eine andere technische Schnittstelle (z.B. Sensoren, Mikrofon) verwenden,
12. eine aussagekräftige `README.md`-Datei enthalten, die sowohl die Anwendung (Screenshots) gut präsentiert, als auch alle Anweisungen zur Installation.

Von den Punkten 5.-11. sollten 

- 5 für eine 2,0 implementiert sein, 
- 6 für eine 1,7 und 
- 7 für eine 1,3. 
- Ist die Anwendung besonders toll und deployed, kann es auch eine 1,0 werden. 

Die Anwendung muss in einem Git-Dienst (GitHub, GitLab, ...) verfügbar sein. Die erstellte Anwendung soll präsentiert werden und in einem kurzen Gespräch (15-20min) wird die Implementierung besprochen. 

Hier eine Idee einer Anwendung, eine *Ausgabenverwaltung*:

- installierbare Webanwendung, 
- Formular für die Buchung einer Ausgabe 
	- Datum, 
	- Titel für die Ausgabe, 
	- Betrag, 
	- Foto des Kassenzettels, 
	- evtl. Geolocation des Ausgabeortes
- Übersicht über Ausgaben,
- offline verwendbar, d.h. Ausgabe wird in der IndexedDB gespeichert und erst, wenn wieder online, dann in der Datenbank,
- Push-Benachrichtigung, wenn Ausgabe in der Datenbank gespeichert (Hintergrundsynchronisation), 
- Backend ist zwingend erforderlich (für Speichern und Abrufen der Daten in die und aus der Datenbank),
- MongoDB zur persitenten Datenspeicherung,
- evtl. Nutzerverwaltung zur Verwaltung der eigenen Ausgaben.

Sie können natürlich auch eine eigene Anwendungsidee umsetzen! Viel Spaß und Erfolg!   


### Alte Videos

Hier sind die Videos aus **2021** verlinkt!!! (auf Wunsch) Der aktuelle Stoff ist aber teilweise abgeändert und angepasst! Insbesondere wird nun MongoDB als Datenbank verwendet und nicht PostgresQL. Es werden nun auch Bilder hochgeladen. Außerdem haben sich wieder einige APIs seit 2021 geändert. 

##### Grundgerüst

??? question "Grundgerüst"

	<iframe src="https://mediathek.htw-berlin.de/media/embed?key=9acae2bf623912843d1298721c67867a&width=720&height=405&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="405" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no"></iframe>


##### Manifest

??? question "Manifest"

	- Diese Woche wird unsere App mithilfe des [Application Manifest](./manifest/#web-app-manifest) **installierbar**. Dazu gibt es folgendes Video: 
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=247bb388ae29a4d849a3fcff5fae95db&width=720&height=450&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="450" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no"></iframe>



##### Promises und Fetch API

??? question "Promises und Fetch API"

	- Diese Woche betrachten wir den Lebenszyklus eines Service Workers, schauen uns Promises und die Fetch API an. Dazu dieses Video:
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=71d02c5d395a874c9b2b4821a140dc7f&width=720&height=389&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="389" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no"></iframe>
	- Sourcecode zur Vorlesung am 05.05.2021 (also alt!!!) [hier zum Herunterladen](./files/03_promise.zip)



##### Caching

??? question "Caching"

	- Diese Woche wird die App mithilfe von [Caching](./caching/#caching-mit-service-workern) auch offline nutzbar.
	- Video zur Vorlesung am 12.05.2021
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=bf498f35fe78e01a22d5fa909d4c4be8&width=720&height=389&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="389" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no"></iframe>
	- Sourcecode zur Vorlesung am 12.05.2021 (also alt!!!) [hier zum Herunterladen](./files/04_caching.zip)


##### MongoDB und Backend

??? question "MongoDB und Backend"

	- Diese Woche erstellen wir mithilfe von Node.js ein [Backend](./backend/#backend-rest-server). Dieses realisiert eine REST-API, die alle CRUD-Funktionalitäten für unsere Posts auf einer MongoDB zur Verfügung stellt.
	- Video zur Vorlesung am 26.05.2021
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=a6e060a195129540a18a7be89a3738e7&width=720&height=450&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="450" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no"></iframe>

##### Bilder-Up- und Download im Backend

??? question "Bilder-Up- und Download im Backend"

	- Diese Woche erweitern wir mithilfe von Multer und GridFS unser [Backend - Erweiterung um das Speichern von Bildern](./backend/#erweiterung-um-das-speichern-von-bildern), um den Up- und Download von Bildern zu ermöglichen. 


##### IndexedDB

??? question "IndexedDB"

	- Diese Woche arbeiten wir am Frontend weiter. Wir [binden das Backend an](./indexeddb/#das-backend-nutzen) und speichern die Posts-Datensätze in der Browser-Built-In-Datenbank [IndexedDB](./indexeddb/#erstellen-und-offnen-einer-indexeddb). 
	- Video zur Vorlesung am 09.06.2021
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=531c0ff5c9765951731a6e27fe41a11e&width=720&height=450&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="450" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no" aria-label="media embed code" style=""></iframe>


##### Kamera

??? question "Kamera"

	- Diese Woche nutzen wir die MediaDevices-API, um mithilfe der Kamera Fotos aufzunehmen, die Teil eines jeden Posts sind. Diese Fotos werden an das Backend übermittelt. 


##### Geolocation-API

??? question "Geolocation-API"

	- Diese Woche nutzen wir die Geolocation-API, um den eigenen Standort zu ermitteln. Dieser Standort wird verwendet, um die entsprechenden Adressinformationen als Lokation in unsere Posts einzutragen. Außerdem zeigen wir auf einer Landkarte diesen Standort an.


##### Hintergrundsynchronisation

??? question "Hintergrundsynchronisation"

	- Diese Woche wird der Post, den wir in das Formular eingeben, nicht direkt an das Backend gesendet, sondern zunächst in der IndexedDB abgelegt. Dort bleibt er solange, bis das Backend erreichbar ist. Dazu wird die SyncManager-API verwendet. Beim Service Worker wird eine Sync Task registriert. Diese wird ausgelöst, sobald die Anwendung online ist.
	- Video aus der Vorlesung:
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=41dea6287b458d7cb58818247711f4b0&width=720&height=420&autoplay=false&controls=true&autolightsoff=false&loop=false&chapters=false&playlist=false&related=false&responsive=false&t=0&loadonclick=true&thumb=true" data-src="https://mediathek.htw-berlin.de/media/embed?key=41dea6287b458d7cb58818247711f4b0&width=720&height=420&autoplay=false&controls=true&autolightsoff=false&loop=false&chapters=false&playlist=false&related=false&responsive=false&t=0&loadonclick=true" class="" width="720" height="420" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no" aria-label="media embed code" style=""></iframe>


##### Push-Benachrichtigungen

??? question "Push-Benachrichtigungen"

	- Diese Woche machen wir uns mit der [Push-benachrichtigung](./pushnotes/#push-notifications) vertraut. Diese verwenden wir, um die Nutzerinnen zu informieren, dass es Änderungen (Updates/Aktuelles) in der Webanwendung gibt. Die Nutzerinnen werden so an die Anwendung gebunden, denn Push-Notifications erhöhen das Interesse daran, die Anwendung erneut zu öffnen.     
	- Video zur Vorlesung am 23.06.2021
		<iframe src="https://mediathek.htw-berlin.de/media/embed?key=bd60899d214c44a0a15987921ddcf908&width=720&height=450&autoplay=false&autolightsoff=false&loop=false&chapters=false&related=false&responsive=false&t=0" data-src="" class="iframeLoaded" width="720" height="450" frameborder="0" allowfullscreen="allowfullscreen" allowtransparency="true" scrolling="no" aria-label="media embed code" style=""></iframe>
		
