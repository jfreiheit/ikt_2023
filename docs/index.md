# Aktuelle Trends der IKT

Herzlich willkommen zur Veranstaltung **Aktuelle Trends der IKT**! 

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


