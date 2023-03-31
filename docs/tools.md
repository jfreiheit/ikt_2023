# Werkzeuge

## Chrome

Es wird empfohlen, Chrome als Browser zu verwenden, da dieser Browser die besten Entwicklertools für Progressive Web Apps zur Verfügung stellt. Insbesondere ist es empfehlenswert, das Plugin [Lighthouse](https://chrome.google.com/webstore/detail/lighthouse) für die Developertools von Chrome zu installieren. 

## Lighthouse 

[Lighthouse](https://chrome.google.com/webstore/detail/lighthouse) ist ein Plugin für die Chrome-Developertools, mit dessen Hilfe gemessen werden kann, wie *progressive* eine App ist. Installieren Sie sich dieses Plugin, um auch die Performance Ihrer PWA zu messen.

## Integrated Development Environment (IDE)

Für die Webentwicklung stehen Ihnen viele gute Entwicklungswerkzeuge zur Verfügung. Für welches Sie sich entscheiden, bleibt Ihnen überlassen. Hier eine Auswahl der aus meiner Sicht besten Entwicklungswerkzeuge:

- [IntelliJ IDEA](https://www.jetbrains.com/de-de/idea/)
- [PhpStorm](https://www.jetbrains.com/de-de/phpstorm/)
- [WebStorm](https://www.jetbrains.com/de-de/webstorm/)
- [Sublime Text](https://www.sublimetext.com/)
- [Atom](https://atom.io/)
- [Visual Studio Code](https://code.visualstudio.com/)

Für die Tools von Jetbrains benötigen Sie einen Account. Mit Ihrer HTW-E-Mail-Adresse bekommen Sie aber eine kostenlose Hochschullizenz und können so die Enterprise-Versionen kostenlos nutzen. Sublime Text ist Shareware und fragt regelmäßig, ob Sie spenden möchten.  


## Node.js

[Node.js](https://nodejs.org/en/) ist eine JavaScript-Laufzeitumgebung. Node.js reagiert auf Ereignisse und antwortet asynchron. Das bedeutet, dass die Ausführung einer Ereignisbearbeitung nicht zum Blockieren der Laufzeitumgebung führt, sondern nebenläufig weitere Ereignisse eintreffen können, die ebenfalls asynchron behandelt werden. Dies geschieht mithilfe des *Callback-Patterns*. Laden Sie sich [hier](https://nodejs.org/en/download/) die aktuellste Version von Node.js herunter und installieren Sie diese auf Ihrem Rechner.

## Android Studio

Um unsere PWAs als mobile Webanwendungen zu emulieren, benutzen wir [Android Studio](https://developer.android.com/studio). Es ist für Mac, Linux und Windows verfügbar. Laden Sie es sich [herunter](https://developer.android.com/studio#downloads) und installieren Sie es. Sie können es herunterladen und installieren, ohne einen Google-Account anzulegen (es gibt auch keinen Grund, das zu tun ;-)).

## https für localhost

- für Mac siehe [hier](https://medium.com/@jonsamp/how-to-set-up-https-on-localhost-for-macos-b597bcf935ee)

## https für Webserver

- siehe [hier](https://letsencrypt.org/getting-started/)
- siehe [hier](https://certbot.eff.org/)

## Ngrok

[Ngrok](https://ngrok.com/) stellt einen sicheren Tunnel zu einem Webserver her. Ngrok wirkt wie ein Proxy, der einer Anwendung suggeriert, mit einem Webserver über eine sichere Verbindung zu kommunizieren, d.h. die Verbindung wirkt wie eine `https`-Verbindung. Die Installation ist einfach, benötigt aber Registrierungsdaten (zur Erzeugung des Authentifizierungstokens). Nach dem [Download](https://ngrok.com/download) wird das Paket entpackt und mit dem Authentifizierungstoken aufgerufen. Nach dem Starten der Webanwendung stellt man mit `ngrok http <Port>` den sicheren Tunnel her, wobei `<Port>` für den Port steht, unter dem die Anwednung auf dem Webserver läuft.  

## Icons erzeugen und in die manifest.json eintragen

Es ist ziemlich mühsam, alle benötigten Icons für die unterschiedlichen Plattformen zu erzeugen und dann noch die entsprechenden Einträge in der `manifest.json` vorzunehmen. Zum Glück gibt es aber ein Werkzeug, das das für uns übernimmt: [pwa-asset-generator](https://www.npmjs.com/package/pwa-asset-generator). Sie benötigen nur das Ausgangsicon in Originalgröße und alles andere wird für Sie erledgt. Alles weitere dazu steht [hier](https://www.npmjs.com/package/pwa-asset-generator).

## WebApp-Manifest-Generator

Bei der Erstellung Ihrer `manifest.json` können Sie sich auch unterstützen lassen, nämlich [hier](https://app-manifest.firebaseapp.com/) oder [hier](https://www.dunplab.it/web-app-manifest-generator).

## Workbox

[Workbox](https://developers.google.com/web/tools/workbox) ist eine JavaScript-Bibliothek, die alle wesentlichen Funktionalitäten von *Service Workern* bereitstellt. 

## MongoDB

Es gibt zwei Möglichkeiten, [MongoDB](https://www.mongodb.com/) zu verwenden: entweder Sie nutzen das Cloud-Angebot, also eine Remote-MongoDB. Diese nennt sich [MongoDB Atlas](https://www.mongodb.com/atlas/database). Oder Sie installieren sich die MongoDB "on-premise", also lokal auf Ihrem Rechner. Dazu wählen Sie unter `mongodb.com` den Reiter `Products` on dort unter `Community Edition` den Link `Community Server`. Dann landen Sie auf [https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community). Dort können Sie sich die MongoDB herunterladen und installieren. Installationsanleitungen finden Sie unter [https://www.mongodb.com/docs/manual/installation/](https://www.mongodb.com/docs/manual/installation/). Wichtig ist, dass die MongoDB einmalig mit `mongod` starten.

## MongoDB Compass

Um sich Ihre MongoDB-Datenbanken anzuschauen (und auch, um Operationen darauf auszuführen), empfehle ich Ihnen das Tool [MongoDB Compass](https://www.mongodb.com/de-de/products/compass). Download und Installation sind normalerweise einfach. 

## Insomnia REST

Eine gute Alternative zu Postman ist [Insomnia](https://insomnia.rest/). Sehr empfehlenswert! Aber Postman auch.

## Interssante Links zu PWA

- [PWA Checklist](https://web.dev/pwa-checklist/)
- [PWA API](https://pwafire.org/)
- [11 Examples of Progressive Web Apps](https://medium.com/@the_manifest/11-examples-of-progressive-web-apps-944f6db25a5a)
- [How-to: Progressive Web Apps praktisch erklärt](https://entwickler.de/online/web/progressive-web-apps-tutorial-tipps-579830771.html)
- [Chrome Developer Summit 2020](https://youtube.com/playlist?list=PLNYkxOF6rcIDzLmWaDwfHVZJl1Q5RFgOR)
- [12 Best Examples of Progressive Web Apps (PWAs) in 2020](https://www.simicart.com/blog/progressive-web-apps-examples/)
- [Lighthouse Performance Scoring](https://web.dev/performance-scoring/)
- [Service worker Spezifikation](https://html.spec.whatwg.org/multipage/workers.html#workers)