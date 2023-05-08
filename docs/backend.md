# Backend - REST-Server

Ehe wir uns der [IndexedDB-API](https://developer.mozilla.org/de/docs/Web/API/IndexedDB_API) zuwenden, erstellen wir zunächst eine "richtige" Datenbank für unsere Posts. Für diese Datenbank stellen wir die Implementierung einer Schnittstelle bereit, so dass wir die wesentlichen Datenbankanfragen darüber ausführen können. Diese wesentlichen Datenbankfragen werden mit [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) abgekürzt, für <strong>C</strong>reate, <strong>R</strong>ead, <strong>U</strong>pdate und <strong>D</strong>elete. Das bedeutet, wir implementieren Funktionalitäten, mit denen wir einen neuen `post` in die Datenbank einfügen (*create*), aus der Datenbank auslesen (*read*), in der Datenbank aktualisieren (*update*) und aus der Datenbank löschen (*delete*) können. 

Die Schnittstelle, die wir implementieren, ist eine sogenannte [REST-API](https://www.redhat.com/de/topics/api/what-is-a-rest-api). *REST* steht für [*Representational State Transfer*](https://de.wikipedia.org/wiki/Representational_State_Transfer) und basiert auf einigen wenigen Prinzipien:

1. Alles wird als eine *Ressource* betrachtet, z.B. `post`.
2. Jede Ressource ist durch *URIs* (*Uniform Resource Identifiers*) eindeutig identifizierbar, z.B. `http://localhost/posts`.
3. Es werden die [Standard-HTTP-Methoden](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol#HTTP-Anfragemethoden) verwendet, also `GET`, `POST`, `PUT`, `UPDATE`.  
4. Ressourcen können in verschiedenen Formaten vorliegen, z.B. in [HTML](https://html.spec.whatwg.org/multipage/), [XML](https://www.w3.org/TR/xml/), [JSON](https://jsonapi.org/format/), ...
5. Die Kommunikation ist *zustandslos*. Jede einzelne HTTP-Anfrage wird komplett isoliert bearbeitet. Es gibt keinerlei Anfragehistorie. 

Das bedeutet, wir erstellen ein Backend (einen REST-Server), an den HTTP-Anfragen mit der eindeutig identifizierbaren Ressource gestellt werden. Das Backend erstellt daraus die entsprechende SQL-Query. Das Resultat der Datenbankanfrage wird im `JSON`- oder `HTML`- oder `XML`- oder in einem anderen Format bereitsgestellt.

![restapi](./files/61_restapi.png)

Prinzipiell gibt es also ein *Mapping* von HTTP-Anfragen auf SQL-Anfragen:

|CRUD |SQL |MongoDB |HTTP |
|-----|----|--------|-----|
|create |INSERT |insertOne(), insertMany() |POST |
|read |SELECT |findOne(), find() |GET |
|update |UPDATE |updateOne(), updateMany() |PUT (oder PATCH)|
|delete |DELETE |deleteOne(), deleteMany() |DELETE |

Zur Unterscheidung zwischen `PUT` und `PATCH` siehe z.B. [hier](https://www.geeksforgeeks.org/difference-between-put-and-patch-request/) oder [hier](https://stackoverflow.com/questions/21660791/what-is-the-main-difference-between-patch-and-put-request).
Wir wollen uns ein Backend erstellen, über das wir unsere Daten verwalten. Dazu überlegen wir uns zunächst ein paar sogenannte *Endpunkte* (siehe Prinzipien von REST oben) und die Zugriffsmethoden, mit denen wir auf unsere Daten zugreifen wollen.


| Methode | URL | Bedeutung |
|---------|-----|-----------|
| GET     | /posts | hole alle Datensätze |
| GET     | /posts/11 | hole den Datensatz mit der id=11 |
| POST    | /posts | füge einen neuen Datensatz hinzu |
| PUT     | /posts/11 | ändere den Datensatz mit der id=11 |
| DELETE  | /posts/11 | lösche den Datensatz mit der id=11 |

Der Wert der `id` ist natürlich nur ein Beispiel. Es soll für alle `id`-Werte funktionieren, die in unserem Datensatz enthalten sind. Korrekterweise beschreiben wir die Endpunkte mit variabler `id` besser durch `/posts/:id` oder `/posts/{id}`.

## OpenAPI

Für eine Dokumentation der zu erstellenden REST-API ist [OpenAPI](https://www.openapis.org/) geeignet. Unter [https://app.swaggerhub.com/home](https://app.swaggerhub.com/home) steht ein Werkzeug zur Verfügung, um eine solche API-Dokumentation zu erstellen. Sie müssen sich dort registrieren und einloggen. Klicken Sie `Create New`, um eine neue Dokumentation zu beginnen:

<figure markdown>
  ![openapi](./files/301_openapi.png){ width="300" }
  <figcaption>Eingabemaske für neue REST-API</figcaption>
</figure>


Es wird automatisch erstellt:

=== "Simple Inventory API"

```yaml linenums="1"
openapi: 3.0.0
servers:
  # Added by API Auto Mocking Plugin
  - description: SwaggerHub API Auto Mocking
    url: https://virtserver.swaggerhub.com/jfreiheit/Posts-API/1.0.0
info:
  description: This is a simple API
  version: "1.0.0"
  title: Simple Inventory API
  contact:
    email: you@your-company.com
  license:
    name: Apache 2.0
    url: 'http://www.apache.org/licenses/LICENSE-2.0.html'
tags:
  - name: admins
    description: Secured Admin-only calls
  - name: developers
    description: Operations available to regular developers
paths:
  /inventory:
    get:
      tags:
        - developers
      summary: searches inventory
      operationId: searchInventory
      description: |
        By passing in the appropriate options, you can search for
        available inventory in the system
      parameters:
        - in: query
          name: searchString
          description: pass an optional search string for looking up inventory
          required: false
          schema:
            type: string
        - in: query
          name: skip
          description: number of records to skip for pagination
          schema:
            type: integer
            format: int32
            minimum: 0
        - in: query
          name: limit
          description: maximum number of records to return
          schema:
            type: integer
            format: int32
            minimum: 0
            maximum: 50
      responses:
        '200':
          description: search results matching criteria
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/InventoryItem'
        '400':
          description: bad input parameter
    post:
      tags:
        - admins
      summary: adds an inventory item
      operationId: addInventory
      description: Adds an item to the system
      responses:
        '201':
          description: item created
        '400':
          description: 'invalid input, object invalid'
        '409':
          description: an existing item already exists
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/InventoryItem'
        description: Inventory item to add
components:
  schemas:
    InventoryItem:
      type: object
      required:
        - id
        - name
        - manufacturer
        - releaseDate
      properties:
        id:
          type: string
          format: uuid
          example: d290f1ee-6c54-4b01-90e6-d701748f0851
        name:
          type: string
          example: Widget Adapter
        releaseDate:
          type: string
          format: date-time
          example: '2016-08-29T09:12:33.001Z'
        manufacturer:
          $ref: '#/components/schemas/Manufacturer'
    Manufacturer:
      required:
        - name
      properties:
        name:
          type: string
          example: ACME Corporation
        homePage:
          type: string
          format: url
          example: 'https://www.acme-corp.com'
        phone:
          type: string
          example: 408-867-5309
      type: object
```

Zur Erläuterung:

- Unter dem Schlüssel `paths` (Zeile `20`) ist ein Pfad (eine *Route*) definiert, nämlich `/inventory`. Bei diesem Pfad handelt es sich um ein sogenanntes [*Path Item Object*](https://swagger.io/specification/#path-item-object)
- Der Pfad `/inventory` enthält zwei sogenannte [*Operation Objects*](https://swagger.io/specification/#operation-object), nämlich `GET` (Zeile `22`) und `POST` (Zeile `63`). Ein *Operation Object* kann verschiedene Eigenschaften beinhalten:

	- `tags`: Schlüsselwörter, um die API-Dokumentation zu gruppieren (siehe unten im Bild `Dokumentation der REST-API` die Gruppen `admins` und `developers`)
	- `summary`: dient der Erläuterung eines Endpunktes (siehe unten im Bild `Dokumentation der REST-API` die Erläuterungen `searches inventory` und `adds an inventory item`)
	- `description`: beschreibt die Funktionalität des Endpunktes detaillierter. Erscheint in der Dokumentation bei den Details eines Endpunktes (siehe unten im Bild `Get /inventroy-Endpunkt im Detail`)
	- `responses`: beschreibt die Rückgabe des Endpunktes. Es handelt sich um ein [*Responses Object*](https://swagger.io/specification/#responses-object). Diese können nach HTTP-Statuscodes unterteilt werden. Neben der `description` für den Statuscode kann dabei insbesondere der Typ der `responses` definiert werden. In der `content`-Eigenschaft wird zunächst der Typ der akzeptierten Response definiert, z.B. `application/json` oder `image/png`. Dann wird spezifiziert, welcher Datentyp zurückgeben wird. Die Zeilen `57-60` beschrieben bspw., dass ein Array von `InventoryItems` zurückgegeben wird. Ein solches `InventoryItem` ist unter der Eigenschaft `schemas` definiert. Mithilfe von `$ref: '#/components/schemas/InventoryItem'` wird auf dieses Schema referenziert. 
- Unter dem Schlüssel `components` können `schemas`, `responses`, `parameters`, `examples`, `requestBodies`, `headers` usw. spezifiziert werden. Mithilfe von `$ref` kann dann auf jede dieser Komponenten referenziert werden. In obigem Beispiel wurde das Schema `InventoryItem` und das Schema `Manufacturer` definiert. Diese Schemen entsprechen den verwendeten Datenmodellen. 


<figure markdown>
  ![openapi](./files/303_openapi.png){ width="300" }
  <figcaption>Dokumentation der REST-API</figcaption>
</figure>


<figure markdown>
  ![openapi](./files/302_openapi.png){ width="300" }
  <figcaption>GET /inventory-Endpunkt im Detail</figcaption>
</figure>

### YAML

Die obige Beschreibung ist übrigens in [YAML](https://yaml.org/). Ursprünglich stand YAML für *Yet Anaother Markup Language*. Jetzt sagt die Spezifikation von YAML aber *YAML Ain't Markup Language*. Es hat Ähnlichkeiten zu JSON, kommt allerdings ohne Klammerung aus. Dafür spielt das Einrücken eine Rolle. OpenAPI unterstützt sowohl JSON als auch YAML. 

### Die '/posts'-Routen

Wir spezifizieren zunächst die `/posts`-Routen, also `GET /posts` und `POST /posts`.

=== "Posts-API"

```yaml linenums="1"
openapi: 3.0.0
servers:
  # Added by API Auto Mocking Plugin
  - description: SwaggerHub Server
    url: https://virtserver.swaggerhub.com/jfreiheit/Posts-API/1.0.0
info:
  description: REST-API für IKT-PWA HTWInsta
  version: "1.0.0"
  title: Posts-API
  contact:
    email: freiheit@htw-berlin.de
  license:
    name: Apache 2.0
    url: 'http://www.apache.org/licenses/LICENSE-2.0.html'
tags:
  - name: Posts
    description: CRUD für Posts
  - name: Images
    description: CRUD für Bilder
paths:
  /posts:
    get:
      tags:
        - Posts
      summary: lese alle Posts
      operationId: getAllPosts
      description: download Array aller verfügbaren Posts
      responses:
        '200':
          description: alle verfügbaren Posts geladen
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Post'
    post:
      tags:
        - Posts
      summary: füge einen neuen Post hinzu
      operationId: createNewPost
      description: neuen Post erzeugen und speichern
      responses:
        '201':
          description: Post created
        '409':
          description: Post existiert bereits
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Post'
        description: neuer Post
components:
  schemas:
    Post:
      type: object
      required:
        - title
        - location
        - image_id
      properties:
        title:
          type: string
          example: H-Gebäude
        location:
          type: string
          example: Campus Wilhelminenhof
        image_id:
          type: string
          example: Campus Wilhelminenhof
```


### Die '/posts/{id}'-Routen

Nun fügen wir noch die `/posts/{id}`-Routen hinzu, also `GET /posts/{id}`, `PUT /posts/{id}` und `DELETE /posts/{id}`.


=== "Posts-API"

```yaml linenums="54"
  /posts/{id}:
    get:
      tags:
        - Posts
      summary: lese einen Post mit der passenden id
      operationId: getOnePost
      description: download entsprechenden Post
      parameters:
        - name: id
          in: path
          required: true
          description: Post-ID
          schema:
            type : string
      responses:
        '200':
          description: Post mit entsprechender id geladen
          content:
            application/json:
              schema:
                type: object
                items:
                  $ref: '#/components/schemas/Post'
        '404':
          description: Post bzw. id nicht gefunden
    put:
      tags:
        - Posts
      summary: ändere einen Post mit der passenden id
      operationId: updateOnePost
      description: aktualisiere entsprechenden Post
      parameters:
        - name: id
          in: path
          required: true
          description: Post-ID
          schema:
            type : string
      responses:
        '200':
          description: Post mit entsprechender id aktualisiert
          content:
            application/json:
              schema:
                type: object
                items:
                  $ref: '#/components/schemas/Post'
        '404':
          description: Post bzw. id nicht gefunden
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Post'
        description: zu aktualisierender Post
    delete:
      tags:
        - Posts
      summary: lösche einen Post mit der passenden id
      operationId: deleteOnePost
      description: lösche entsprechenden Post
      parameters:
        - name: id
          in: path
          required: true
          description: Post-ID
          schema:
            type : string
      responses:
        '200':
          description: Post mit entsprechender id gelöscht
          content:
            application/json:
              schema:
                type: object
                items:
                  $ref: '#/components/schemas/Post'
        '404':
          description: Post bzw. id nicht gefunden
```

### Die vollständige YAML-Datei

Hier die vollständige [YAML-Datei](./files/openapi.yaml), die Sie auch in Postman importieren können.

??? "openapi.yaml"
	```yaml
		openapi: 3.0.0
		info:
		  title: Posts-API
		  description: REST-API für IKT-PWA HTWInsta
		  contact:
		    email: freiheit@htw-berlin.de
		  license:
		    name: Apache 2.0
		    url: http://www.apache.org/licenses/LICENSE-2.0.html
		  version: 1.0.0
		servers:
		- url: https://virtserver.swaggerhub.com/jfreiheit/Posts-API/1.0.0
		  description: SwaggerHub Server
		tags:
		- name: Posts
		  description: CRUD für Posts
		- name: Images
		  description: CRUD für Bilder
		paths:
		  /posts:
		    get:
		      tags:
		      - Posts
		      summary: lese alle Posts
		      description: download Array aller verfügbaren Posts
		      operationId: getAllPosts
		      responses:
		        "200":
		          description: alle verfügbaren Posts geladen
		          content:
		            application/json:
		              schema:
		                type: array
		                items:
		                  $ref: '#/components/schemas/Post'
		                x-content-type: application/json
		      x-swagger-router-controller: Posts
		    post:
		      tags:
		      - Posts
		      summary: füge einen neuen Post hinzu
		      description: neuen Post erzeugen und speichern
		      operationId: createNewPost
		      requestBody:
		        description: neuer Post
		        content:
		          application/json:
		            schema:
		              $ref: '#/components/schemas/Post'
		      responses:
		        "201":
		          description: Post created
		        "409":
		          description: Post existiert bereits
		      x-swagger-router-controller: Posts
		  /posts/{id}:
		    get:
		      tags:
		      - Posts
		      summary: lese einen Post mit der passenden id
		      description: download entsprechenden Post
		      operationId: getOnePost
		      parameters:
		      - name: id
		        in: path
		        description: Post-ID
		        required: true
		        style: simple
		        explode: false
		        schema:
		          type: string
		      responses:
		        "200":
		          description: Post mit entsprechender id geladen
		          content:
		            application/json:
		              schema:
		                type: array
		                items:
		                  $ref: '#/components/schemas/Post'
		                x-content-type: application/json
		        "404":
		          description: Post bzw. id nicht gefunden
		      x-swagger-router-controller: Posts
		    put:
		      tags:
		      - Posts
		      summary: ändere einen Post mit der passenden id
		      description: aktualisiere entsprechenden Post
		      operationId: updateOnePost
		      parameters:
		      - name: id
		        in: path
		        description: Post-ID
		        required: true
		        style: simple
		        explode: false
		        schema:
		          type: string
		      requestBody:
		        description: zu aktualisierender Post
		        content:
		          application/json:
		            schema:
		              $ref: '#/components/schemas/Post'
		      responses:
		        "200":
		          description: Post mit entsprechender id aktualisiert
		          content:
		            application/json:
		              schema:
		                type: array
		                items:
		                  $ref: '#/components/schemas/Post'
		                x-content-type: application/json
		        "404":
		          description: Post bzw. id nicht gefunden
		      x-swagger-router-controller: Posts
		    delete:
		      tags:
		      - Posts
		      summary: lösche einen Post mit der passenden id
		      description: lösche entsprechenden Post
		      operationId: deleteOnePost
		      parameters:
		      - name: id
		        in: path
		        description: Post-ID
		        required: true
		        style: simple
		        explode: false
		        schema:
		          type: string
		      responses:
		        "200":
		          description: Post mit entsprechender id gelöscht
		          content:
		            application/json:
		              schema:
		                type: array
		                items:
		                  $ref: '#/components/schemas/Post'
		                x-content-type: application/json
		        "404":
		          description: Post bzw. id nicht gefunden
		      x-swagger-router-controller: Posts
		components:
		  schemas:
		    Post:
		      required:
		      - image_id
		      - location
		      - title
		      type: object
		      properties:
		        title:
		          type: string
		          example: H-Gebäude
		        location:
		          type: string
		          example: Campus Wilhelminenhof
		        image_id:
		          type: string
		          example: Campus Wilhelminenhof
		      example:
		        location: Campus Wilhelminenhof
		        title: H-Gebäude
		        image_id: Campus Wilhelminenhof
	```

![openapi](./files/309_openapi.png)

![openapi](./files/310_openapi.png)

## Ein Node.js-Projekt mit Express

Wir starten damit, uns ein `node.js`-Projekt zu erstellen. Dazu erstellen wir uns zunächst einen Ordner `backend`, wechseln in diesen Ordner und führen dann `npm init` aus:

```bash
mkdir backend
cd backend
npm init
```

Sie werden ein paar Sachen gefragt. Im Prinzip können Sie immer `Enter` drücken, außer beim `entry point`. Dort können Sie gleich `server.js` eingeben. Sie können das aber auch noch später in der `package.json` ändern.

```bash
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (backend) 
version: (1.0.0) 
description: Backend REST-API
entry point: (index.js) server.js
test command: 
git repository: 
keywords: rest api backend mongodb
author: J. Freiheit
license: (ISC) 
About to write to /Users/jornfreiheit/Sites/IKT22/05_Backend/00_skript/backend/package.json:

{
  "name": "backend",
  "version": "1.0.0",
  "description": "Backend REST-API",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "rest",
    "api",
    "backend",
    "mongodb"
  ],
  "author": "J. Freiheit",
  "license": "ISC"
}


Is this OK? (yes) 
```


Die `package.json` wurde erstellt. Nun benötigen wir noch das Modul [Express](https://expressjs.com/de/). Express bietet uns eine unkomplizierte *Middleware* für die Weiterverwaltung von `http`-Anfragen an die Datenbank und zurück. 

```bash
npm install express --save
```

Die Option `--save` muss eigentlich nicht mehr angegeben werden, aber unter [Express](https://expressjs.com/de/) steht es noch so. Sie erhalten eine Meldung in der Form:

```bash
% npm install express --save

added 57 packages, and audited 58 packages in 887ms

7 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

In der `package.json` wurde die entsprechende Abhängigkeit eingetragen: 

=== "package.json"
	```json linenums="1" hl_lines="18"
	{
	  "name": "backend",
	  "version": "1.0.0",
	  "description": "Backend REST-API",
	  "main": "server.js",
	  "scripts": {
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "keywords": [
	    "rest",
	    "api",
	    "backend",
	    "mongodb"
	  ],
	  "author": "J. Freiheit",
	  "license": "ISC",
	  "dependencies": {
	    "express": "^4.18.0"
	  }
	}
	``` 


#### server.js erstellen und implementieren

Öffnen Sie nun das `backend`-Projekt in Ihrer IDE und erstellen Sie sich dort eine Datei `server.js` mit folgendem Inhalt:

=== "server.js"
	```javascript linenums="1"
	const express = require('express');
	const routes = require('./routes');

	const app = express();
	const PORT = 3000;

	app.use(express.json());
	app.use('/', routes);

	app.listen(PORT, (error) => {
	    if (error) {
	        console.log(error);
	    } else {
	        console.log(`server running on http://localhost:${PORT}`);
	    }
	});
	``` 

Das bedeutet, wir importieren `express` (Zeile `1`), erzeugen uns davon ein Objekt und speichern dieses in der Variablen `app` (Zeile `4`). Wir legen in einer Konstanten `PORT` die Portnummer `3000` fest (Zeile `5` - die Portnummer können Sie wählen). Das `backend` ist somit unter `http://localhost:3000` verfügbar. Das eigentliche Starten des Webservers erfolgt in den Zeilen `10-16` durch Aufruf der `listen()`-Funktion von `express`. Die Syntax der `listen()`-Funktion ist generell wie folgt:

```bash
app.listen([port[, host[, backlog]]][, callback])
```

Wir übergeben als ersten Parameter die `PORT`-Nummer (`3000`) und als zweiten Parameter eine (anonyme) Funktion als sogenannten *callback*. *Callbacks* sind [hier](../promises/#callbacks) näher erläutert. Die anonyme Funktion wird durch die `listen()`-Funktion aufgerufen. Sollte ein Fehler aufgetreten sein (z.B. wenn der Port bereits belegt ist), wird der anonymen Funktion ein `error`-Objekt übergeben. Ist das der Fall, wird der Fehler auf der Konsole ausgegeben. Wird der anonymen Funktion kein Objekt übergeben, wurde der Webserver korrekt gestartet und die entsprechende Meldung erscheint auf der Konsole. 

Beachten Sie auch die verwendete Syntax `${PORT}` im sogenannte [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals). Beachten Sie, dass *template literals* nicht in einfachen (`'`) oder doppelten (`"`) Anführungsstrichen stehen, sondern in <code>&#96;</code> (*backticks*). 

### Router

Noch lässt sich unser Programm aber nicht ausführen. Wir benötigen im Projektordner noch eine Datei `routes.js`. Diese wird nämlich in der `server.js` bereits in Zeile `2` eingebunden und in Zeile `8` verwendet. 

=== "routes.js"
	```javascript linenums="1"
	const express = require('express');
	const router = express.Router();

	// eine GET-Anfrage
	router.get('/', async(req, res) => {

	    res.send({ message: "Hello FIW!" });
	});

	module.exports = router;
	``` 


Beim `Router` handelt es sich um eine *Middleware* (siehe [hier](https://expressjs.com/de/guide/using-middleware.html)), die die Routen verwaltet und `request`-Objekte an die entsprechende Routen weiterleitet und `response`-Objekte empfängt. In unserer `routes.js` haben wir zunächst eine `GET`-Anfrage implementiert (Zeile `5`). Das `request`-Objekt heißt hier `req`. Das verwenden wir aber gar nicht. Das `respones`-Objekt heißt hier `res` und wird durch die Anfrage erzeugt. Wir senden in der `response` ein JavaScript-Objekt zurück, das einen Schlüssel `message` enthält. 

In der `server.js` haben wir mit `app.use(express.json())` (Zeile `7`) angegeben, dass alle JavaScript-Objekte in der `response` nach JSON umgewandelt werden sollen. Wenn nun die URL `localhost:3000` aufgerufen wird, dann wird ein `request` ausgelöst, den wir hier mit `Hello FIW!` als `response` beantworten (Zeilen `5-8`). 

Wichtig ist, dass wir `router` mit `module.exports` exportieren, damit es von anderen Modulen importiert und genutzt werden kann. Siehe dazu z.B. [hier](https://www.sitepoint.com/understanding-module-exports-exports-node-js/). Meine Empfehlung ist, **(noch) nicht** das [neue ESM6-Format](https://www.sitepoint.com/understanding-es6-modules/) zu nutzen! 

Noch "läuft" unser Backend aber noch nicht. Wir müssen es erst starten. 

### Starten des Projektes und Installation von nodemon

Das Projekt lässt sich nun starten. Wir geben dazu im Terminal im `backend`-Ordner

```bash
node server.js
```

ein. Im Terminal erscheint 

```bash
server running on http://localhost:3000 
```

und wenn Sie im Browser die URL `http://localhost:3000/` eingeben, wird dort

![backend](./files/91_backend.png)

angezeigt. Sie können auch Postman öffnen und `http://localhost:3000` eintragen (`GET`-Methode):

![backend](./files/92_postman.png)


Wann immer wir jetzt jedoch etwas an der Implementierung ändern, müssen wir im Terminal zunächst den Webserver mit 

```bash
Strg-C		// bzw. Control-C
```

stoppen, um ihn dann wieder mit `node server.js` zu starten. Um das zu umgehen, gibt es das Paket [nodemon](https://www.npmjs.com/package/nodemon). Da es nur sinnvoll während der Entwicklung eingesetzt werden kann (und sollte), installieren wir es als eine *development dependency*:

```bash
npm install --save-dev nodemon
```

Die `package.json` sieht daraufhin so aus:

=== "package.json"
	```json linenums="1" hl_lines="20-22"
	{
	  "name": "backend",
	  "version": "1.0.0",
	  "description": "Backend REST-API",
	  "main": "server.js",
	  "scripts": {
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "keywords": [
	    "rest",
	    "api",
	    "backend",
	    "mongodb"
	  ],
	  "author": "J. Freiheit",
	  "license": "ISC",
	  "dependencies": {
	    "express": "^4.18.0"
	  },
	  "devDependencies": {
	    "nodemon": "^2.0.16"
	  }
	}
	```

Zur Verwendung von `nodemon` fügen wir in die `package.json` unter `"scripts"` noch die Eigenschaft `watch` (frei gewählt) und den dazugehörigen Wert `nodemon server.js` ein:

=== "package.json"
	```json linenums="1" hl_lines="7"
	{
	  "name": "backend",
	  "version": "1.0.0",
	  "description": "Backend REST-API",
	  "main": "server.js",
	  "scripts": {
	    "watch": "nodemon ./server.js",
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "keywords": [
	    "rest",
	    "api",
	    "backend",
	    "mongodb"
	  ],
	  "author": "J. Freiheit",
	  "license": "ISC",
	  "dependencies": {
	    "express": "^4.18.0"
	  },
	  "devDependencies": {
	    "nodemon": "^2.0.16"
	  }
	}
	```

Nun lässt sich die Anwendung mithilfe von `nodemon` per 

```bash
npm run watch
```

starten und muss auch nicht mehr gestoppt und neu gestartet werden, wenn Änderungen an der Implementierungen durchgeführt wurden. Die Ausgabe im Terminal nach Eingabe von `npm run watch` ist ungefähr so:

```bash

> backend@1.0.0 watch
> nodemon ./server.js

[nodemon] 2.0.16
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node ./server.js`
server running on http://localhost:3000

```

Hier nur zum Verständnis. Angenommen, wir ändern bspw. in der `server.js` die Zeile `8` zu 

```js
app.use('/api', routes);
```

, dann würden alle Routen, die wir in `routes.js` definieren, unter `localhost:3000/api` verfügbar sein. Wenn wir dann also z.B. in der `routes.js` die Zeile `5` zu 

```js
router.get('/fiw', async(req, res) => {
```

ändern, dann ist der GET-Endpunkt `localhost:3000/api/fiw`. 


### Mongoose installieren

[MongoDB](https://www.mongodb.com/de-de) ist die am meisten verwendete *NoSQL (not only SQL)* Datenbank. Sie basiert nicht auf Relationen, Tabellen und ihren Beziehungen zueinander (ist also keine *relationale* Datenbank), sondern speichert Dokumente in JSON-ähnlichem Format. Die [Community Edition der MongoDB](https://github.com/mongodb/mongo) ist Open Source und kostenlos verfügbar. 
Sollten Sie mit *Visual Studio Code* arbeiten, sollten Sie sich am besten die [MongoDB for VS Code](https://code.visualstudio.com/docs/azure/mongodb)-Ereiterung installieren. 

Zur Verwendung von *MongoDB* im Backend verwenden wir das Modul [Mongoose](https://mongoosejs.com/). Wir installieren *Mongoose* mithilfe von

```bash
npm install mongoose --save
```

In die `package.json` wird das Paket und die entsprechende Abhängigkeit eingetragen:

=== "package.json"
	```json linenums="1" hl_lines="20"
	{
	  "name": "backend",
	  "version": "1.0.0",
	  "description": "Backend REST-API",
	  "main": "server.js",
	  "scripts": {
	    "watch": "nodemon ./server.js",
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "keywords": [
	    "rest",
	    "api",
	    "backend",
	    "mongodb"
	  ],
	  "author": "J. Freiheit",
	  "license": "ISC",
	  "dependencies": {
	    "express": "^4.18.0",
	    "mongoose": "^6.3.1"
	  },
	  "devDependencies": {
	    "nodemon": "^2.0.16"
	  }
	}
	```

*Mongoose* stellt eine einfach zu verwendende Schnittstelle zwischen Node.js und MongoDB bereit. Die
MongoDB benötigen wir aber trotzdem (wir könnten jedoch auch eine Cloud von MongoDB oder z.B. `mlab.com` verwenden). Bevor wir uns mit der MongoDB verbinden, erstellen wir zunächst noch eine Datenbank. 

### MongoDB Compass

Um sich Ihre MongoDB-datenbanken anzuschauen, empfehle ich Ihnen das Tool [MongoDB Compass](https://www.mongodb.com/de-de/products/compass). Download und Installation sind normalerweise einfach. 


### Dotenv für sichere Zugangsdaten

Für die "geheimen" Zugangsdaten (die jetzt noch gar nicht "geheim" sind) verwenden wir das [dotenv](https://www.npmjs.com/package/dotenv)-Paket:

```bash
npm install dotenv --save
```

Im Projektordner erstellen wir und eine Datei `.env` (mit vorangestelltem Punkt!) und schreiben darin:


=== ".env"
	```js linenums="1"
	DB_CONNECTION = mongodb://127.0.0.1:27017/posts
	```

Beachten Sie, dass der Wert nicht in Hochkomma steht und dass auch kein Semikolon folgt! 
Wir fügen `dotenv` in die `server.js` ein und greifen mithilfe von `process.env.DB_CONNECTION` auf den Wert von `DB_CONNECTION` zu:

=== "server.js"
	```js linenums="1" hl_lines="4 21"
	const express = require('express');
	const routes = require('./routes');
	const mongoose = require('mongoose');
	require('dotenv').config();

	const app = express();
	const PORT = 3000;

	app.use(express.json());
	app.use('/', routes);

	app.listen(PORT, (error) => {
	    if (error) {
	        console.log(error);
	    } else {
	        console.log(`Server started and listening on port ${PORT} ... `);
	    }
	});

	// connect to mongoDB
	mongoose.connect(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
	const db = mongoose.connection;
	db.on('error', console.error.bind(console, 'connection error:'));
	db.once('open', () => {
	    console.log('connected to DB');
	});
	```

In Zeile `4` wird das `dotenv`-Paket importiert. Mithilfe der `config()`-Funktion wird die `.env`-Datei eingelesen. Auf die in der `.env`-Datei hinterlegten Schlüssel-Werte-Paare (mit `=` dazwischen) kann dann mittels `process.env.<Schlüssel>` zugegriffen werden (siehe Zeile `21`). 

Beachten Sie, die `.env`-Datei in die `.gitignore` einzutragen. Die `.env`-Datei sollte **nicht** committed werden!


### Ein Model erstellen

Mongoose ist Schema-basiert. Ein Schema kann man sich wie ein Datenmodell vorstellen. Tatsächlich wird es verwendet, um ein entsprechendes Mongoose-Model zu erstellen. Ein Schema wird unter Aufruf des Konstruktors (`new Schema()`) in Mongoose erstellt. Unter Verwendung des Schemas wird dann mithilfe der `model()`-Funktion das Datenmodell erzeugt. 

Wir werden im Folgenden zeigen, wie ein Schema für `posts` erstellt wird. Das Datenmodell heißt dann `Post`. Um später auch weitere Schemata, z.B. für `user` o.ä. zu entwicklen und diese zu trennen, erstellen wir das Schema in einem eigenen Ordner `models`. Das bedeutet, wir erstellen im Projektordner 

- ein Ordner `models` und
- darin eine Datei `models/posts.js`

Die Datei `posts.js` bekommt folgenden Inhalt:

=== "models/posts.js"
	```javascript
	const mongoose = require('mongoose');

	const schema = new mongoose.Schema({
	    title: String,
	    location: String,
	    image_id: String
	});

	module.exports = mongoose.model('Post', schema);
	```

Weiterführende Informationen zu Mongoose-Models finden Sie z.B. [hier](https://mongoosejs.com/docs/models.html). Das Thema Schema wird z.B. [hier](https://mongoosejs.com/docs/guide.html) näher erläutert. 


### Zugriffe auf die Datenbank

Nun haben wir alles, was wir benötigen, um unsere Anfragen zu implementieren. Wir nutzen den `express.Router`, um die Routen zu definieren und können mithilfe des Mongoose-Models auf die MongoDB zugreifen. Wir werden nun sukzessive alle Anfragen in die `routes.js` einfügen. 

#### R - read all

Wir beginnen mit der Anfrage, alle Daten aus der Datenbank auszulesen. Für die MongoDB erfolgt dies mit der Funktion `find()`. In `routes.js` ändern wir unsere `GET`-Anfrage wie folgt: 

=== "routes.js"
	```javascript linenums="1" hl_lines="3 6-10"
	const express = require('express');
	const router = express.Router();
    const Post = require('./models/posts');

	// GET all posts
	router.get('/posts', async(req, res) => {
	    const allPosts = await Post.find();
	    console.log(allPosts);
	    res.send(allPosts);
	});

	module.exports = router;
	```

Beachten Sie, dass wir dazu nun das `Post`-Model in die `routes.js` einbinden (Zeile `3`). Die Route wird mit `localhost:3000/posts` definiert. Die anonyme Callback-Funktion enthält noch zwei Schlüsselwörter: `async` und `await`. Die Funktion `find()` ist ein *Promise* (siehe dazu [hier](../promises/#promises)). Die Funktion `find()` wird asynchron ausgeführt und "irgendwann" ist entweder das Ergebnis dieser Funktion verfügbar oder die Funktion gibt einen Fehler zurück. Auf eines der beiden wird gewartet (`await`). Nur eine als `async` deklarierte Funktion darf einen `await`-Aufruf enthalten (siehe dazu z.B. [hier](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Statements/async_function)).

Die Ausgabe der Werte auf die Konsole (Zeile `8`) ist natürlich nicht erforderlich und Sie können sie auch löschen, wenn Sie wollen. Wenn Sie nun in Postman `GET http://localhost:3000/posts` aufrufen, erscheinen alle Einträge aus der Datenbank. Allerdings haben wir dort noch keine Einträge. Wir bekommen deshalb ein leeres Array `[]` zurück.


#### C - create

Als nächstes implementieren wir einen Endpunkt, an dem wir einen neuen Datensatz in die Datenbank anlegen können. Dafür gibt es die http-Methode `POST`. Wir führen also nicht mehr eine `GET`-, sondern eine `POST`-Anfrage durch. Bei dieser `POST`-Anfrage wird der neue Datensatz an den Webserver mitgeschickt. Dies erfolgt im `body` des `request`-Objektes. Das Schreiben des Datensatzes in die Datenbank erfolgt mithilfe der `save()`-Funktion von MongoDB. 

=== "routes.js"
	```javascript linenums="12"
    // POST one post
	router.post('/posts', async(req, res) => {
	    const newPost = new Post({
	        title: req.body.title,
	        location: req.body.location,
	        image_id: req.body.image_id
	    })
	    await newPost.save();
	    res.send(newPost);
	});
	```

In den Zeilen `15-17` werden die Daten aus dem `body` des `request`-Objektes ausgelesen und mit diesen Daten ein neues `Post`-Objekt erzeugt. Dieses neue `Post`-Objekt (`newPost`) wird in Zeile `19` in die Datenbank gespeichert und in Zeile `20` als `response` zurückgeschickt.  

Nun geben wir in Postman `POST http://localhost:3000/posts` ein und befüllen den `Body` z.B. mit:

```json linenums="1"
{ 
    "title": "H-Gebäude",
    "location": "Campus Wilhelminenhof",
    "image_id": "test" 
}
``` 

Achten Sie darauf, dass in der zweiten Menüzeile rechts `JSON` ausgewählt ist (im Bild blau) - nicht `Text`. Wir klicken auf `Send` und es erscheint:

![postman](./files/95_postman.png)

Schauen Sie auch in MongoDB Compass nach, ob der Datensatz dort erscheint:

![phpmyadmin](./files/221_mongodb.png)


#### R - read one

Wir erweitern die `routes.js` um einen Endpunkt, der uns für eine gegebene `id` den entsprechenden Datensatz zurückliefert. Die `_id` werden von MongoDB automatisch vergeben und sind recht kryptisch, also z.B. `"626bdb36cd1af60df758d300"`. Wir können natürlich nach jedem beliebigen Wert für jeden Schlüssel in der Datenbank suchen. Wir nehmen hier beispielhaft die `_id`. 

Die `id` wird aus der URL des Endpunktes ausgelesen, d.h. wenn wir bspw. den Endpunkt `GET http://localhost:3000/posts/626bdb36cd1af60df758d300` eingeben, dann soll der Datensatz mit der `_id: 626bdb36cd1af60df758d300` im JSON-Format zurückgegeben werden. Wir nutzen dazu parametrisierte Routen und lesen die `id` aus der Parameterliste aus. Paremtrisierte Routen werden per `:` und dann den Namen des Parameters (hier `id`) erstellt. Um dann den Wert des Parametrs `id` aus der Parameterliste auszulesen, wird `params` verwendet. Im folgenden Code lassen wir `req.params` auf die Konsole ausgeben, um die Funktionsweise zu erläutern. Diese Ausgabe kann natürlich gelöscht werden (Zeile `27`). 

=== "routes.js"
	```javascript linenums="23"
    // GET one post via id
	router.get('/posts/:id', async(req, res) => {
	    try {
	        const post = await Post.findOne({ _id: req.params.id });
	        console.log(req.params);
	        res.send(post);
	    } catch {
	        res.status(404);
	        res.send({
	            error: "Post does not exist!"
	        });
	    }
	});
	``` 

Zum Finden eines einzelnen Datensatzes wird in MongoDB die Funktion `findOne()` verwendet (siehe [hier](https://docs.mongodb.com/manual/reference/method/db.collection.findOne)). Wird der Datensatz gefunden, d.h. existiert die entsprechende `_id`, dann wird dieser in der `response` zurückgesendet (Zeile `28`). Existiert er nicht, wird der HTTP-Statuscode `404` gesendet (Zeile `30`) und ein JSON mit der `error`-Nachricht `Post does not exist!` (Zeile `31`). 

Nach Neustart des Servers geben wir in Postman z.B. `GET http://localhost:3000/posts/626bdb36cd1af60df758d300` ein (bei Ihnen sind die `_id`-Werte andere!) und erhalten:

![postman](./files/97_postman.png)

Probieren Sie auch einmal `GET http://localhost:3000/posts/0` aus, um die Fehlermeldung als JSON zu sehen. 


#### U - update

Um einen bereits existierenden Datensatz zu ändern, kann entweder die HTTP-Anfrage `PUT` oder `PATCH` verwendet werden. Zur Unterscheidung zwischen `PUT` und `PATCH` siehe z.B. [hier](https://www.geeksforgeeks.org/difference-between-put-and-patch-request/) oder [hier](https://stackoverflow.com/questions/21660791/what-is-the-main-difference-between-patch-and-put-request). Um einen Datensatz in der MongoDB zu ändern, stehen prinzipiell mehrere Funktionen zur Verfüging:

- `updateOne()`: ändert einzelne (oder alle) Teile eines Datensatzes und sendet die `_id` zurück, falls ein neur Datensatz angelegt wurde,
- `findOneAndUpdate()`: ändert einzelne (oder alle) Teile eines Datensatzes und sendet den kompletten Datensatz zurück,
- `replaceOne()`: ändert den kompletten Datensatz. 

In der folgenden Implementierung haben wir uns für die HTTP-Anfragemethode `PATCH` und für die MongoDB-Funktion `updateOne()` entschieden. Diese Funktion erwartet als ersten Parameter einen `<filter>`, d.h. die Werte, nach denen nach einem Datensatz gesucht werden soll. Im folgenden Beispiel ist der Filter die `_id`. Dazu wird erneute ein Parameter `id` für die URL definiert. Der zweite Parameter der `updateOne()`-Funktion sind die zu ändernden Werte für diesen Datensatz. In der folgenden Implementierung werden diese zu ändernden Werte als ein JSON dem `body` des `request`-Objektes übergeben. Um zu ermöglichen, dass ein, zwei oder drei Schlüssel-Werte-Paare in diesem JSON enthalten sein können, prüfen wir die Einträge im `body` und setzen daraus ein neues `member`-Objekt zusammen, wenn es bereits in der Datenbank existiert (deshalb zunächst `findOne()`):

=== "router.js"
	```javascript linenums="37"
    // PATCH (update) one post
	router.patch('/posts/:id', async(req, res) => {
	    try {
	        const post = await Post.findOne({ _id: req.params.id })

	        if (req.body.title) {
	            post.title = req.body.title
	        }

	        if (req.body.location) {
	            post.location = req.body.location
	        }

	        if (req.body.image_id) {
	            post.image_id = req.body.image_id
	        }

	        await Post.updateOne({ _id: req.params.id }, post);
	        res.send(post)
	    } catch {
	        res.status(404)
	        res.send({ error: "Post does not exist!" })
	    }
	});
	```

Wir können diese Funktion in Postman ausprobieren, indem wir im `body` z.B. das JSON 

```json linenums="1"
{ 
    "image_id": "test_neu" 
}
```

mit unserem Request übergeben und `PATCH http://localhost:3000/posts/626bdb36cd1af60df758d300` wählen (bei Ihnen eine andere `id`!). Der Datensatz mit der `_id=626bdb36cd1af60df758d300` wird dann aktualisiert. 

![postman](./files/98_postman.png)

Schauen Sie auch in der Datenbank nach (z.B. in MongoDB Compass) und wählen auch ruhig nochmal `GET http://localhost:3000/posts` (z.B. in Postman).


#### D - delete one

Jetzt implementieren wir noch den Endpunkt, um einen Datensatz zu löschen. Dazu werden die HTTP-Anfragemethode `DELETE` und die MongoDB-Funktion `deleteOne()` verwendet. Im folgenden Beispiel wird der Datensatz erneut über die `_id` ermittelt und dafür erneut die parametrisierte URL ausgelesen:

=== "routes.js"
	```javascript linenums="87"
	// DELETE one post via id
	router.delete('/posts/:id', async(req, res) => {
	    try {
	        await Post.deleteOne({ _id: req.params.id })
	        res.status(204).send()
	    } catch {
	        res.status(404)
	        res.send({ error: "Post does not exist!" })
	    }
	});
	``` 

Wenn wir nun in Postman z.B. `DELETE http://localhost:3000/members/626bdb36cd1af60df758d300` wählen (bei Ihnen eine andere `id`!), wird der Datensatz mit der `_id=626bdb36cd1af60df758d300` aus der Datenbank gelöscht. 

Hier nochmal die vollständige `routes.js`:

??? "routes.js"
	```js linenums="1"
	const express = require('express');
	const router = express.Router();
    const Post = require('./models/posts');

	// GET all posts
	router.get('/posts', async(req, res) => {
	    const allPosts = await Post.find();
	    console.log(allPosts);
	    res.send(allPosts);
	});

    // POST one post
	router.post('/posts', async(req, res) => {
	    const newPost = new Post({
	        title: req.body.title,
	        location: req.body.location,
	        image_id: req.body.image_id
	    })
	    await newPost.save();
	    res.send(newPost);
	});

    // GET one post via id
	router.get('/posts/:id', async(req, res) => {
	    try {
	        const post = await Post.findOne({ _id: req.params.id });
	        console.log(req.params);
	        res.send(post);
	    } catch {
	        res.status(404);
	        res.send({
	            error: "Post does not exist!"
	        });
	    }
	});

    // PATCH (update) one post
	router.patch('/posts/:id', async(req, res) => {
	    try {
	        const post = await Post.findOne({ _id: req.params.id })

	        if (req.body.title) {
	            post.title = req.body.title
	        }

	        if (req.body.location) {
	            post.location = req.body.location
	        }

	        if (req.body.image_id) {
	            post.image_id = req.body.image_id
	        }

	        await Post.updateOne({ _id: req.params.id }, post);
	        res.send(post)
	    } catch {
	        res.status(404)
	        res.send({ error: "Post does not exist!" })
	    }
	});

	// DELETE one post via id
	router.delete('/posts/:id', async(req, res) => {
	    try {
	        await Post.deleteOne({ _id: req.params.id })
	        res.status(204).send()
	    } catch {
	        res.status(404)
	        res.send({ error: "Post does not exist!" })
	    }
	});

	module.exports = router;	
	```

### Cross-Origin Resource Sharing (CORS)

Die *Same Origin Policy (SOP)* ist ein Sicherheitskonzept, das clientseitig Skriptsprachen (also z.B. JavaScript oder CSS) untersagt, Ressourcen aus verschiedenen Herkunften zu verwenden, also von verschiedenen Servern. Dadurch soll verhindert werden, dass fremde Skripte in die bestehende Client-Server-Kommunikation eingeschleust werden. Gleiche *Herkunft (origin)* bedeutet, dass das gleiche Protokoll (z.B. `http` oder `https`), von der gleichen Domain (z.B. `localhost` oder `htw-berlin`) sowie dem gleichen Port (z.B. `80` oder `4200`) verwendet werden. Es müssen alle drei Eigenschaften übereinstimmen. 

Mit dem Aufkommen von Single Page Applications und dem darin benötigten AJAX kam jedoch der Bedarf auf, die SOP aufzuweichen. Es sollte möglich sein, dass z.B. JavaScript sowohl client-seitig das DOM ändert als auch einen Request an den Server (das Backend) sendet. Der Kompromiss, der dafür gefunden wurde, nennt sich *Cross-Origin Resource Sharing (CORS)*. Damit ist es möglich, für einige oder alle Anfragen zu definieren, dass sie im Sinne der SOP trotzdem erlaub sein sollen. 

Um CORS für Ihr Backend zu aktivieren, wechseln Sie im Terminal in Ihren `backend`-Ordner und geben dort

```bash
npm install cors
```

ein. Öffnen Sie dann die `server.js` und fügen Sie die hervorgehobenen Zeilen ein:

=== "server.js"
	```javascript linenums="1" hl_lines="2 11-12"
	const express = require('express');
    const cors = require('cors');
	const routes = require('./routes');
    const mongoose = require('mongoose');
	require('dotenv').config();

	const app = express();
	const PORT = 3000;

	app.use(express.json());
    // enable cors for all requests
	app.use(cors());
	app.use('/', routes);

	app.listen(PORT, (error) => {
	    if (error) {
	        console.log(error);
	    } else {
	        console.log(`server running on http://localhost:${PORT}`);
	    }
	});

    // connect to mongoDB
	mongoose.connect(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
	const db = mongoose.connection;
	db.on('error', console.error.bind(console, 'connection error:'));
	db.once('open', () => {
	    console.log('connected to DB');
	});
	```

Wenn Sie z.B. nur die `get`-Anfrage teilen wollen, dann wählen Sie nicht `app.use(cors());`, sondern 

```javascript
app.get("/", cors(), (req, res) => {
	    res.json({ message: "Hello FIW!" });
	});
```

Mehr zum CORS-Paket von node.js bzw. express finden Sie [hier](https://expressjs.com/en/resources/middleware/cors.html).

!!! success
		Das bis hier erstellte Backend ist unter [https://github.com/jfreiheit/IKT-PWA-Backend.git](https://github.com/jfreiheit/IKT-PWA-Backend.git) verfügbar. 


## Erweiterung um das Speichern von Bildern

Bis jetzt haben wir nur Daten im JSON-Format zwischen Frontend und Backend ausgetauscht und auch nur solche Daten in der MongoDB gespeichert. Bilder (und auch andere Dateien) sind [FormData-Objects](https://developer.mozilla.org/en-US/docs/Web/API/FormData/Using_FormData_Objects) im `multipart/form-data`-Format. Zur Behandlung solcher Daten verwenden wir ein *Middleware* für unser Backend, namens [Multer](https://www.npmjs.com/package/multer). 

!!! hint
		Wenn Sie nur am Code für unser Backend interessiert sind, dann können Sie auch direkt zu [Zusammenführen der Funktionalitäten](./#zusammenfuhren-der-funktionalitaten) springen. Im Folgenden werden die Entstehung aber näher erläutert und verschiedene Varianten diskutiert. 

MongoDB speichert Daten bis zu einer Größe von `16Mb` im Binärformat. Um auch größere Dateien (Bilder, Videos, pdf, ...) speichern zu können, werden die Dateien in *chunks* zerlegt und können dann aus diesen Stücken wieder zusammengesetzt werden. Dafür gibt es in der MongoDB eine [GridFS](https://docs.mongodb.com/manual/core/gridfs/)-Spezifikation (siehe auch [hier](https://medium.com/@kavitanambissan/uploading-and-retrieving-a-file-from-gridfs-using-multer-958dfc9255e8) oder [hier](https://www.topcoder.com/thrive/articles/storing-large-files-in-mongodb-using-gridfs)). Zur Verwendung von GridFS gibt es die beiden Pakte [multer-gridfs-storage](https://www.npmjs.com/package/multer-gridfs-storage) und [gridfs-stream](https://www.npmjs.com/package/gridfs-stream). 

Wir installieren im Backend-Projekt alle drei Pakete und zeigen im Folgenden deren Verwendung:

```bash
npm install multer multer-gridfs-storage gridfs-stream
```

Die `package.json` sollte nun ungefähr so aussehen:

=== "package.json"
	```js linenums="1"
	{
	  "name": "backend",
	  "version": "1.0.0",
	  "description": "Backend REST-API",
	  "main": "server.js",
	  "scripts": {
	    "watch": "nodemon ./server.js",
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "keywords": [
	    "rest",
	    "api",
	    "backend",
	    "mongodb"
	  ],
	  "repository": {
	    "type": "git",
	    "url": "https://github.com/jfreiheit/IKT-PWA-Backend.git"
	  },
	  "author": "J. Freiheit",
	  "license": "ISC",
	  "dependencies": {
	    "cors": "^2.8.5",
	    "dotenv": "^16.0.0",
	    "express": "^4.18.0",
	    "gridfs-stream": "^1.1.1",
	    "mongoose": "^6.3.1",
	    "multer": "^1.4.4",
	    "multer-gridfs-storage": "^5.0.2"
	  },
	  "devDependencies": {
	    "nodemon": "^2.0.16"
	  }
	}
	```

Wir kümmern uns nun zunächst darum, Bilder in die MongoDB *hochzuladen*.


### Upload von Bildern

Für den Upload der Bilder erstellen wir zunächst einen Ordner `middleware` und darin eine Datei `upload.js`. In dieser Datei wird unter Verwendung von `Multer` ein `GridFsStorage` eingerichtet. Die zu verwendende *Collection* benennen wir hier `fileupload` (siehe `bucketName`). Sie können diesen Namen frei wählen. Beachten Sie dann aber im Folgenden überall die Verwendung von `fileupload` (in der MongoDB entstehen die Collections `fileupload.files` und `fileupload.chunks` - siehe z.B. [hier](https://medium.com/@kavitanambissan/uploading-and-retrieving-a-file-from-gridfs-using-multer-958dfc9255e8) oder [hier](https://www.topcoder.com/thrive/articles/storing-large-files-in-mongodb-using-gridfs)).

=== "middleware/upload.js"
```js linenums="1"
const multer = require("multer");
const {
    GridFsStorage
} = require("multer-gridfs-storage");

const storage = new GridFsStorage({
    url: process.env.DB,
    options: { useNewUrlParser: true, useUnifiedTopology: true },
    file: (req, file) => {
        const match = ["image/png", "image/jpeg"];

        if (match.indexOf(file.mimetype) === -1) {
            console.log('file.mimetype === -1')
            return `${Date.now()}-jf-${file.originalname}`;
        }
        console.log('store');
        return {
            bucketName: 'posts',
            filename: `${Date.now()}-jf-${file.originalname}`,
        };
    },
});

module.exports = multer({ storage });
```

Beachten Sie, dass wir beim Upload der Bilder für diese Bilder Dateinamen mithilfe von `${Date.now()}-jf-${file.originalname}` erstellen bzw. festlegen. Damit diese Dateinamen eindeutig sind, wird mithilfe von `Date.now()` der aktuelle Zeitstempel verwendet. Der String `-jf-` in der Mitte kann natürlich auch durch Ihre Initialen ersetzt (oder weggelassen) werden. Außerdem wird auch noch der originale Dateiname verwendet. Insgesamt sollte sichergestellt werden, dass die Dateinamen eindeutig sind (deshalb auch `Date.now()`). 

In Zeile `10` werden die Dateitypen festgelegt, die akzeptiert werden, hier `png` und `jpeg`. Diese Liste kann erweitert oder eingegrenzt werden.

Diese *Middleware* nutzen wir nun für den `POST`-Request des Bildes und erstellen einen Ordner `routes` und darin eine Datei `upload.routes.js` mit folgendem Inhalt:

=== "routes/upload.route.js"
```js linenums="1"
const express = require('express');
const upload = require('../middleware/upload');
const router = express.Router();

router.post('/', upload.single('file'), (req, res) => {
    // req.file is the `file` file
    if (req.file === undefined) {

        return res.send({
            "message": "no file selected"
        });
    } else {
        console.log('req.file', req.file);
        const imgUrl = `http://localhost:4000/download/${req.file.filename}`;
        return res.status(201).send({
            url: imgUrl
        });
    }
})

module.exports = router;
```

In Zeile `5` wird die *multer-Middleware* mit `update.single('file')` aufgerufen. Neben der Funktion `.single(fieldname)` stehen auch die Funktionen `.array(fieldname[, maxCount])` und `.fields(field)` zur Verfügung, um gleichzeitig mehrere Dateien hochzuladen (siehe [multer](https://www.npmjs.com/package/multer)). 

Als *Response* wird die URL zurückgegeben, unter der das Bild heruntergeladen werden kann (`http://localhost:4000/download/${req.file.filename}`). 

### Routen ändern und einbinden


In der ursprünglichen Implementierung hatten wir die Datei `routes.js` noch im Projektordner gehabt. Wir benennen diese in `posts.routes.js` um und schieben sie ebenfalls in den `routes`-Ordner. Beachten Sie, dass Sie dadurch in der `server.js` auch den Import auf `const routes = require('./routes/posts.routes');` ändern müssen.

Da wir nun aber auch die `upload`-Route einbinden, ändern wir gleich mehrere Sachen. Zunächst wird der ursprüngliche `routes` zu `postRoutes` und die generelle Route dafür wird `/posts`:

=== "server.js"
```js linenums="1" hl_lines="3-4 14-15"
const express = require('express');
const cors = require('cors');
const postsRoutes = require('./routes/posts.routes');
const uploadRoutes = require('./routes/upload.routes');
const mongoose = require('mongoose');
require('dotenv').config();

const app = express();
const PORT = 3000;

app.use(express.json());
  // enable cors for all requests
app.use(cors());
app.use('/posts', postsRoutes);
app.use('/img', uploadRoutes);

app.listen(PORT, (error) => {
    if (error) {
        console.log(error);
    } else {
        console.log(`server running on http://localhost:${PORT}`);
    }
});

  // connect to mongoDB
mongoose.connect(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', () => {
    console.log('connected to DB');
});
```

In der `posts.routes.js` kann nun aus den URLs der Endpunkte jeweils das `/posts` entfernt werden:

=== "routes/posts.routes.js"
```js linenums="1" hl_lines="6 13 24 38 63"
const express = require('express');
const router = express.Router();
const Post = require('../models/posts');

// GET all posts
router.get('/', async(req, res) => {
    const allPosts = await Post.find();
    console.log(allPosts);
    res.send(allPosts);
});

// POST one post
router.post('/', async(req, res) => {
    const newPost = new Post({
        title: req.body.title,
        location: req.body.location,
        image_id: req.body.image_id
    })
    await newPost.save();
    res.send(newPost);
});

// POST one post via id
router.get('/:id', async(req, res) => {
    try {
        const post = await Post.findOne({ _id: req.params.id });
        console.log(req.params);
        res.send(post);
    } catch {
        res.status(404);
        res.send({
            error: "Post does not exist!"
        });
    }
});

// PATCH (update) one post
router.patch('/:id', async(req, res) => {
    try {
        const post = await Post.findOne({ _id: req.params.id })

        if (req.body.title) {
            post.title = req.body.title
        }

        if (req.body.location) {
            post.location = req.body.location
        }

        if (req.body.image_id) {
            post.image_id = req.body.image_id
        }

        await Post.updateOne({ _id: req.params.id }, post);
        res.send(post)
    } catch {
        res.status(404)
        res.send({ error: "Post does not exist!" })
    }
});

// DELETE one post via id
router.delete('/:id', async(req, res) => {
    try {
        await Post.deleteOne({ _id: req.params.id })
        res.status(204).send()
    } catch {
        res.status(404)
        res.send({ error: "Post does not exist!" })
    }
});

module.exports = router;
```

### Upload mithilfe von Postman

Das Hochladen der Bilder kann nun bereits getestet werden. Starten Sie das Backend. Öffnen Sie Postman und geben Sie als URL `http://localhost:3000/upload` ein und wählen als Anfragemethode `POST`. Klicken Sie auf `Body` und markieren dann `form-data`:

![upload](./files/244_file.png)

Geben Sie unter `KEY` den Schlüssel `file` ein (das wird manchmal vergessen und dann bekommen Sie einen `multer`-Fehler `unexpected field`!) und wählen Sie aus dem Dropdown-Menü `File`. Unter `VALUE` erscheint der Button `Select Files`. Klicken Sie darauf und wählen ein `png`- oder ein `jpeg`-Bild (kann auch `.jpg` sein) aus, das Sie hochladen wollen. Klicken Sie dann auf `Send`. Es erscheint:

![upload](./files/245_file.png)

Ich habe in diesem Beispiel die Datei `fiw.jpg` hochgeladen. 

Wenn Sie sich die MongoDB anschauen, dann finden Sie darin die beiden Collections `posts.files` und `posts.chunks`. In `posts.files` sind die Metadaten des hochgeladenen Bildes zu finden, z.B. 

```json
{
  "_id": {
    "$oid": "627399fc6820d87a03418810"
  },
  "length": 86584,
  "chunkSize": 261120,
  "uploadDate": {
    "$date": "2022-05-05T09:33:48.949Z"
  },
  "filename": "1651743228869-jf-fiw.jpg",
  "contentType": "image/jpeg"
}
```

Die dazugehörige `_id` finden Sie auch in `posts.chunks` (können Sie sich in der `mongosh` mit `db.posts.chunks.find({ _id: "627399fc6820d87a03418810" })` anschauen). Darin ist das Bild im Binary-Format gespeichert. 


### Download von Bildern

Für den Download der gespeicherten Bilder gehen wir ähnlich vor, wie beim Upload, benötigen dafür aber nicht mehr die *multer-Middleware*, dafür aber `gridfs-stream`. Wir erstellen im Ordner `routes` die Datei `download.routes.js` mit folgendem Inhalt:


=== "routes/download.routes.js"
```js linenums="1"
const express = require('express');
const mongoose = require('mongoose');
const Grid = require("gridfs-stream");
const router = express.Router();

const connect = mongoose.createConnection(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });

let gfs, gfsb;
connect.once('open', () => {
    // initialize stream
    gfsb = new mongoose.mongo.GridFSBucket(connect.db, {
        bucketName: "posts"
    });

    gfs = Grid(connect.db, mongoose.mongo);
});

router.get('/:filename', async(req, res) => {
    try {
        const cursor = await gfs.collection('posts').find({ filename: req.params.filename });
        cursor.forEach(doc => {
            console.log('doc', doc);
            gfsb.openDownloadStream(doc._id).pipe(res);
        })
    } catch (error) {
        console.log('error', error);
        res.send("not found");
    }
});

module.exports = router;
```


[GridFSBucket](https://mongodb.github.io/node-mongodb-native/3.2/api/GridFSBucket.html) ist eine Klasse aus der [Node.js-MongoDB-API](https://mongodb.github.io/node-mongodb-native/3.2/api/index.html). Diese hätten wir auch schon für das Upload verwenden können (siehe z.B. [hier](https://edupeeth.com/all-courses/nodejs/mongodb-gridfs-bucket)). 

Da wir über den Dateinamen auf die Datei zugreifen wollen, benötigen wir zunächst die entsprechende `_id` der Datei in der `posts.chunks`-Collection. Dazu greifen wir mithilfe von `find()` auf die `posts.files`-Collection zu und ermitteln die `_id`. Die `find()`-Funktion gibt einen sogenannten [Cursor](https://docs.mongodb.com/drivers/node/current/fundamentals/crud/read-operations/cursor/) auf das Array aller gefundenen Datensätze zurück. Mithilfe von `forEach()` durchlaufen wir dieses Array (enthält aber nur einen Datensatz) und ermitteln die `_id`. Mit der `openDownloadStream()`-Funktion der `GridFSBucket()`-Klasse öffnen wir den Download-Stream des Bildes und geben ihn als *response* `res` zurück. 

Wir binden die `download`-Route nun noch in unsere `server.js` ein:

=== "server.js"
```js linenums="1" hl_lines="5 16"
const express = require('express');
const cors = require('cors');
const postsRoutes = require('./routes/posts.routes');
const uploadRoutes = require('./routes/upload.routes');
const downloadRoute = require('./routes/download.routes');

const mongoose = require('mongoose');
require('dotenv').config();

const app = express();
app.use(express.json());
app.use(cors());

app.use('/posts', postsRoutes);
app.use('/image', uploadRoutes);
app.use('/download', downloadRoute);
	
app.listen(process.env.PORT, (error) => {
    if (error) {
        console.log(error);
    } else {
        console.log(`server running on http://localhost:${process.env.PORT}`);
    }
});

  // connect to mongoDB
mongoose.connect(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', () => {
    console.log('connected to DB');
});
```


### Download mithilfe von Postman

Der Test des `GET http://localhost:3000/download/:filename`-Endpunktes ist einfach. Dazu verwenden wir einfach die URL, die durch den Upload als *Response*  zurückgegeben wurde (im obigen Beispiel also `"http://localhost:3000/download/1651743228869-jf-fiw.jpg"`):

Geben Sie in Postman also Ihre URL ein, wählen `GET` und klicken `Send`. Es erscheint das Bild:

![upload](./files/246_file.png)

Da es sich um die GET-Methode handelt, können Sie die URL `http://localhost:3000/download/1651743228869-jf-fiw.jpg` natürlich auch in den Browser eingeben und das Bild erscheint. 

### Download als base64-String

Im oberen Beispiel wurde das Bild beim Download gestreamt. Wir betrachten nun die Möglichkeit, dass es als base64-String gesendet wird. Base64 ist ein Verfahren zur Kodierung von 8-Bit-Binärdaten (z.B. Bilder) in eine Zeichenfolge, die nur aus lesbaren, Codepage-unabhängigen ASCII-Zeichen besteht. Der String ist für ein Bild sehr lang. Er lässt sich aber leicht in eine Bildquelle umwandeln, damit das Bild dann im Frontend dargestellt werden kann. 

Wir werden in der `download.routes.js` beide Möglichkeiten lassen (also sowohl den Stream als auch den base64-String). Deshalb passen wir die Routen der Endpunkte an. Die `/show/:filename`-Route ist für den Stream und die `/send/:filename`-Route für den `base64`-String. Wir präsentieren zunächst den Code und erläutern ihn dann. Die `download.routes.js` sieht dann so aus:


=== "routes/download.routes.js"
```js linenums="1"
const express = require('express');
const mongoose = require('mongoose');
const Grid = require("gridfs-stream");
const multer = require('multer');
const router = express.Router();
const { GridFsStorage } = require('multer-gridfs-storage')
const { MongoClient } = require('mongodb');
require('dotenv').config();

const connect = mongoose.createConnection(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
const dbName = "posts";

let gfs, gfsb;
connect.once('open', () => {
    // initialize stream
    gfsb = new mongoose.mongo.GridFSBucket(connect.db, {
        bucketName: "posts"
    });

    gfs = Grid(connect.db, mongoose.mongo);
    // gfs.collection('file');
});

router.get('/show/:filename', async(req, res) => {
    try {
        const cursor = await gfs.collection('posts').find({ filename: req.params.filename });
        cursor.forEach(doc => {
            console.log('doc', doc);
              const id = doc._id.valueOf();
              console.log('doc._id', id);
            gfsb.openDownloadStream(doc._id).pipe(res);
        })
    } catch (error) {
        console.log('error', error);
        res.send("not found");
    }
});

router.get('/send/:filename', async(req, res) => {
      
      let fileName = req.params.filename;
      
      MongoClient.connect(process.env.DB_CONNECTION, (err, client) => {

          if(err){
              return res.send({title: 'Uploaded Error', message: 'MongoClient Connection error', error: err.errMsg});
          }
          
          const db = client.db(dbName);
          const collection = db.collection('posts.files');
          const collectionChunks = db.collection('posts.chunks');

          collection.find({filename: fileName}).toArray( (err, docs) => {
              if(err){
                  return res.send({title: 'File error', message: 'Error finding file', error: err.errMsg});
              }
      
              if(!docs || docs.length === 0){
                  return res.send({title: 'Download Error', message: 'No file found'});
              } else {
                  //Retrieving the chunks from the db
                  console.log('docs[0]._id', docs[0]._id)
                  collectionChunks.find({files_id : docs[0]._id}).sort({n: 1}).toArray( (err, chunks) => {
                      if(err){
                          return res.send({title: 'Download Error', message: 'Error retrieving chunks', error: err.errmsg});
                      }
                      if(!chunks || chunks.length === 0){
                          //No data found
                          return res.send({title: 'Download Error', message: 'No data found'});
                      }
                      //console.log('chunks', chunks)
                      //Append Chunks
                      let fileData = [];
                      for(let chunk of chunks){
                          console.log('chunk', chunk)
                          //This is in Binary JSON or BSON format, which is stored
                          //in fileData array in base64 endocoded string format
                          fileData.push(chunk.data.toString('base64'));
                      }

                      //Display the chunks using the data URI format
                      let finalFile = 'data:' + docs[0].contentType + ';base64,' + fileData.join('');
                      // console.log('finalFile', finalFile)
                      res.send({title: 'Image File', message: 'Image loaded from MongoDB GridFS', imgurl: finalFile});
                  }) // toArray
              } // else
          }) // toArray
      }) // connect
  }) // get

module.exports = router;
```

Die Funktion sieht auf den ersten Blick sehr umfangreich aus. Die meisten Zeilen sind jedoch für die Fehlerbehandlung. Wenn wir die Fehlerbahndlung (und die Kommentare und die Konsolenausgaben) entfernen, dann bleibt nur noch das übrig:

```js linenums="1"
router.get('/send/:filename', async(req, res) => {
        
        let fileName = req.params.filename;
        
        MongoClient.connect(process.env.DB_CONNECTION, (err, client) => {
            
            const db = client.db(dbName);
            const collection = db.collection('posts.files');
            const collectionChunks = db.collection('posts.chunks');

            collection.find({filename: fileName}).toArray( (err, docs) => {

                    collectionChunks.find({files_id : docs[0]._id}).sort({n: 1}).toArray( (err, chunks) => {

                        let fileData = [];
                        for(let chunk of chunks){
                            fileData.push(chunk.data.toString('base64'));
                        }

                        let finalFile = 'data:' + docs[0].contentType + ';base64,' + fileData.join('');
                        res.send({title: 'Image File', message: 'Image loaded from MongoDB GridFS', imgurl: finalFile});
                    }) // toArray
            }) // toArray
        }) // connect
    }) // get

```

Wir verwenden hier also direkt das [mongodb](https://www.npmjs.com/package/mongodb)-Paket, das ist der MongoDB-Treiber für Node.js. Wir stellen eine Verbindung mit der Datenbank `posts` her (Zeile `5`) und erstellen uns drei Variablen, eine für die Datenbank, eine für die `posts.files`-Collection und eine für die `posts.chunks`-Collection (siehe [GridFS](https://www.mongodb.com/docs/drivers/node/v3.6/fundamentals/gridfs/)). Dann suchen wir in der `posts.files`-Collection nach dem Dateinamen und ermitteln die zugehörige `_id` (speichern als `files_id`, erhalten wir aus `docs[0]._id` - siehe Zeile `13`). Nun suchen wir alle `chunks` mit dieser `files_id` und erhalten ein `chunks`-Array. Dieses Array enthält die `data` des jeweiligen `chunk`. Mithilfe der JavaScript-Funktion `toString('base64)` wandeln wir diese Daten in einen String um und speichern die String in das Array `fileData`. Dieses Array ist also ein Array aus Strings. Unter der Verwendung der `join('')`-Funktion von JavaScript werden alle diese String-Einträge zu einem String verbunden. Vor diesen String setzen wir noch `"data:image/jpeg;base64,"` (siehe z.B. [hier](https://elmah.io/tools/base64-image-encoder/)). 

Wenn wir nun ein Bild downloaden wollen, geben wir in Postman z.B. `http://localhost:3000/download/1652090780364-jf-htwbild4.jpg` ein und erhalten den sehr langen base64-String des Bildes. Die gleiche Funktionalität haben Sie auch, wenn Sie einen Online-Encoder verwenden, z.B. [base64-image.de](https://www.base64-image.de/).


### Delete von Bildern

Das Löschen der Bilder ist ganz ähnlich zum Download. Erstellen Sie die Datei `routes/delete.route.js`:

=== "routes/delete.routes.js"
```js linenums="1"
const express = require('express');
const mongoose = require('mongoose');
const Grid = require("gridfs-stream");
const router = express.Router();

const connect = mongoose.createConnection(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });

let gfs;
connect.once('open', () => {
    gfs = Grid(connect.db, mongoose.mongo);
    gfs.collection('posts');
});

router.delete('/:filename', async(req, res) => {
    try {
        await gfs.collection('posts').deleteOne({ filename: req.params.filename });
        res.send({
            "message": "deleted"
        });
    } catch (error) {
        console.log('error', error);
        res.send("An error occured.");
    }
});

module.exports = router;
```

und binden diese in die `server.js` ein:

=== "server.js"
```js linenums="1" hl_lines="6 17"
const express = require('express');
const cors = require('cors');
const postsRoutes = require('./routes/posts.routes');
const uploadRoutes = require('./routes/upload.routes');
const downloadRoute = require('./routes/download.routes');
const deleteRoute = require('./routes/delete.routes');
const mongoose = require('mongoose');
require('dotenv').config();

const app = express();
app.use(express.json());
app.use(cors());

app.use('/posts', postsRoutes);
app.use('/image', uploadRoutes);
app.use('/download', downloadRoute);
app.use('/delete', deleteRoute);

app.listen(process.env.PORT, (error) => {
	if (error) {
		console.log(error);
	} else {
		console.log(`server running on http://localhost:${process.env.PORT}`);
	}
});

// connect to mongoDB
mongoose.connect(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', () => {
	console.log('connected to DB');
});
```


### Delete mithilfe von Postman

Zum Testen verwenden Sie den gleichen Dateinamen wie beim Download und wählen als Anfragemethode `DELETE`. 


## Zusammenführen der Funktionalitäten

Wir haben nun recht viele Routen und Endpunkte in unserem Backend. Wir wollen aber gerne, dass es nur die fünf genannten Endpunkte gibt:

| Methode | URL | Bedeutung |
|---------|-----|-----------|
| GET     | /posts | hole alle Datensätze |
| GET     | /posts/11 | hole den Datensatz mit der id=11 |
| POST    | /posts | füge einen neuen Datensatz hinzu |
| PUT     | /posts/11 | ändere den Datensatz mit der id=11 |
| DELETE  | /posts/11 | lösche den Datensatz mit der id=11 |

Das bedeutet, wir binden den Upload und Download von Bildern nun in unsere `posts`-Routen ein. Die Funktionen werden dabei "etwas" umfangreicher. 

### Zum Verständnis

Wir verwenden [Multer](https://www.npmjs.com/package/multer) und [GridFs storage](https://www.npmjs.com/package/multer-gridfs-storage). Multer ist eine *Middleware* für Node.js, um Daten im `multipart/form-data`-Format zu verwalten. Die grundsätzliche Idee ist, dass im *Request* nicht nur ein `body`, sondern auch eine `file`-Eigenschaft enthalten ist (neben dem `header`). Multer verwendet einen `storage`, um Bilder (oder andere Dateien) zu speichern. Einen solchen `storage` bietet `GridFs storage`. Dieser kann sogar Dateien größer als 16 MB speichern und die Idee dabei ist, dass die Datei in zwei Collections gespeichert wird, in der `files`-Collection, welche die (Meta-)Informationen der Datei speichert und der `chunks`-Collection, die die eigentliche Datei (als Binärdaten) speichert. Eine Datei kann dabei in mehrere `chunks` unterteilt werden. Die folgende Abbildung zeigt das Prinzip von `GridFS`:


![upload](./files/251_file.png)

Für unser Datenmodell sieht die Auteilung der Daten somit wie folgt aus:

- in der `posts`-Collection speichern wir
	- die `_id` des Posts,
	- den `title` eines Posts,
	- die `location` und 
	- die `image_id`. Die `image_id` enthält den Dateinamen `filename` des Bildes.

- in der `posts.files`-Collection speichern wir (GridFs)
	- die `_id` der Datei,
	- die `length` der Datei,
	- die `chunkSize`, 
	- das `uploadDate`, 
	- den `filename` (siehe in `posts` die `image_id`) und 
	- den `contenType` (z.B. `image/jpeg`)

- in der `posts.chunks`-Collection speichern wir (GridFs)
	- die `_id` des Chunks,
	- die `files_id` (diese entspricht der `_id` der Datei in der `posts.files`-Collection),
	- ein `n` (fortlaufende Nummerierung der Chunks einer Datei beginnend mit `0`),
	- die `data` der Datei (in diesem Chunk)

Chunks kann es zu einer Datei mehrere geben. Alle `data` aller Chunks einer Datei bilden zusammen die Datei als Binär- (bzw. base64-) Daten. Die folgende Abbildung zeigt unser Datenmodell in der Datenbank `posts`:


![upload](./files/251_file.png)

Um z.B. einen Datensatz (einen Post) anzulegen, speichern wir also die zugehörigen Daten in der `posts`-Collection (inkl. dem `filename` der Datei), speichern die Meta-Informationen der Datei in der `posts.files`-Collection und die zugehörigen Binärdaten der Datei in `posts.chunks`. 

### POST - kompletter Datensatz

Die `POST`-Funktion für einen Datensatz ist nicht viel umfangreicher als zuvor:

=== "aus routes/posts.routes.js"
```js linenums="27"
  // POST one post
router.post('/', upload.single('file'), async(req, res) => {
    // req.file is the `file` file
    if (req.file === undefined) {
        return res.send({
            "message": "no file selected"
        });
    } else {
		console.log('req.body', req.body);
		const newPost = new Post({
			title: req.body.title,
			location: req.body.location,
			image_id: req.file.filename
		})
		await newPost.save();
		return res.send(newPost);
    }
})
``` 

Wichtig ist, dass `posts.routes.js` nun auch die `upload.js` einbindet:


=== "aus routes/posts.routes.js"
```js linenums="4"
const upload = require('../middleware/upload');
``` 

Beachten Sie, dass die Daten nun nicht mehr als JSON übergeben werden, sondern als `form-data`. Der Test mithilfe von Postman sieht deshalb nun so aus:

![posts](./files/247_file.png) 

Als Response bekommen Sie aber wieder ein JSON zurück, z.B.:

```json
{
    "title": "HTW Gebäude C",
    "location": "Campus Wilhelminenhof",
    "image_id": "1652166642127-jf-htwbild5.jpg",
    "_id": "627a0ff2305433d805b6b437",
    "__v": 0
}
```

Das Bild ist nun in der Collection `posts` einerseits in `posts.files` mit den Daten

![posts](./files/248_file.png) 

und in `posts.chunks` mit den Daten

![posts](./files/249_file.png) 

gespeichert. Beachten Sie, dass das Bild so klein ist, dass es in nur einem `chunk` gespeichert werden kann. Größere Dateien werden in mehrer `chunks` aufgeteilt. Alle diese `chunks`, die zu einem Bild gehören, haben dann dieselbe `files_id` (aber fortlaufende `_id`s).

In der `posts`-Collection sieht der Datensatz dann wie folgt aus:

![posts](./files/250_file.png) 

### GET - ein kompletter Datensatz

Jetzt den kompletten Datensatz mit einer bestimmten `_id` zu laden, ist etwas aufwendiger:

1. Wir laden zunächst aus der `posts`-Collection den Datensatz mit der `_id`. 
2. Aus diesem Datensatz lesen wir die `image_id` aus. Das ist der `filename` mit dem wir in der `posts.files`-Collection suchen. 
3. Aus der `posts.files`-Collection lesen wir den Datensatz mit dem `filename` aus und identifizieren dessen `_id`. 
4. Nach dieser `_id` suchen wir unter `files_id` in der `posts.chunks`-Collection und lesen alle zugehörigen `chunks` aus. 

Dazu schreiben wir uns zunächst eine Funktion `getOnePost(id)`, die ein `Promise` zurückgibt.

=== "aus routes/posts.routes.js"
```js linenums="23"
const connect = mongoose.createConnection(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
const collectionFiles = connect.collection('posts.files');
const collectionChunks = connect.collection('posts.chunks');

function getOnePost(id) {
	return new Promise( async(resolve, reject) => {
		try {
			const post = await Post.findOne({ _id: id });
			// console.log('post.image_id', post.image_id);
			let fileName = post.image_id;

			collectionFiles.find({filename: fileName}).toArray( async(err, docs) => {
				// console.log('docs', docs)

				collectionChunks.find({files_id : docs[0]._id}).sort({n: 1}).toArray( (err, chunks) => {
					

					const fileData = [];
					for(let chunk of chunks)
					{
						// console.log('chunk._id', chunk._id)
						fileData.push(chunk.data.toString('base64'));
					}

					let base64file = 'data:' + docs[0].contentType + ';base64,' + fileData.join('');
					let getPost = new Post({
						"title": post.title,
						"location": post.location, 
						"image_id": base64file
					});
					//console.log('getPost', getPost)
					resolve(getPost)
				})

			}) // toArray find filename

		} catch {
        	reject(new Error("Post does not exist!"));
        }
	})
}
``` 

Die Konsolenausgaben können natürlich auch alle entfernt werden, aber es lohnt sich vielleicht auch zu sehen, was an den einzelnen Stellen passiert. In Zeile `30` erfolgt der Zugriff auf die `posts`-Collection und es wird der Datensatz mit der `_id` ermittelt, welche als Parameter der URL übergeben wurde. Aus dem Ergebnis `post` wird dann der Dateiname des Bildes in Zeile `32` mithilfe von `post.image_id` ermittelt. 

In der `post.files`-Collection wird in Zeile `34` nach dem Datensatz mit dem entsprechenden `filename` gesucht. Die `_id` dieses Datensatzes ist der Wert von `files_id` in der Collection `posts.chunks`. Nach all diesen Einträgen wird in Zeile `37` gesucht. Aus allen `chunks` wird dann der `base64`-String erzeugt und dem `Post`-Objekt übergeben, welches als `resolve` der `Promise` zurückgeschickt wird. 

Diese Funktion können wir nun für unseren `get('/:id')`-Endpunkt verwenden. Die Funktion sieht dann wie folgt aus:

=== "aus routes/posts.routes.js"
```js linenums="102"
// GET one post via id
router.get('/:id', async(req, res) => {
	getOnePost(req.params.id)
	.then( (post) => {
		console.log('post', post);
		res.send(post);
	})
	.catch( () => {
		res.status(404);
        res.send({
            error: "Post does not exist!"
        });
	})
});
``` 

Wir übergeben der `getOnePost()`-Funktion die als Parameter übergebene `id` und senden den `resolve`-Wert der `Promise` als Response zurück. 


### GET - alle Datensätze

Der Ansatz, um alle Datensätze aus der MongoDB zu lesen, ist der gleiche, wie für einen Datensatz. Wir ermitteln sukzessive die `_id` alle Datensätze in der `posts`-Collection. Dazu schreiben wir uns eine Funktion `getAllPosts()`. In dieser laden wir zunächst alle `posts` und rufen dann für jeden einzelnen die `getOnePost(id`-Funktion auf:

=== "aus routes/posts.routes.js"
```js linenums="65"
function getAllPosts() {
	return new Promise( async(resolve, reject) => {
		const sendAllPosts = [];
		const allPosts = await Post.find();
		try {
			for(const post of allPosts) {
				console.log('post', post)
				const onePost = await getOnePost(post._id);
				sendAllPosts.push(onePost);
			}
			console.log('sendAllPosts', sendAllPosts)
			resolve(sendAllPosts)
		} catch {
				reject(new Error("Posts do not exist!"));
    }
	});
}
``` 

Die Verwendung dieser Funktion ist wie oben:


=== "aus routes/posts.routes.js"
```js linenums="12"
// GET all posts
router.get('/', async(req, res) => {
	
	getAllPosts()
	.then( (posts) => {
		res.send(posts);
	})
	.catch( () => {
		res.status(404);
        res.send({
            error: "Post do not exist!"
        });
	})
});
``` 

### DELETE - einen Datensatz

Wird ein Post gelöscht, müssen wir auch dafür sorgen, dass das zugehörige Bild aus der `posts.files` und der `posts.chunks` gelöscht wird. Das Löschen ist also dreistufig:

=== "aus routes/posts.routes.js"
```js linenums="12"
// DELETE one post via id
router.delete('/:id', async(req, res) => {
    try {
        const post = await Post.findOne({ _id: req.params.id })
        let fileName = post.image_id;
        await Post.deleteOne({ _id: req.params.id });
        await collectionFiles.find({filename: fileName}).toArray( async(err, docs) => {
            await collectionChunks.deleteMany({files_id : docs[0]._id});
        })
        await collectionFiles.deleteOne({filename: fileName});
        res.status(204).send()
    } catch {
        res.status(404)
        res.send({ error: "Post does not exist!" })
    }
});
```

### Zusammenfassung - Code des Backends

Hier nochmal alle wichtigen Dateien unseres Backends:


=== "server.js"
```js linenums="1"
const express = require('express');
const cors = require('cors');
const postsRoutes = require('./routes/posts.routes');
require('dotenv').config();
const mongoose = require('mongoose');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());
app.use(cors());
app.use('/posts', postsRoutes)

app.listen(PORT, (error) => {
    if(error) {
        console.log(error)
    } else {
        console.log(`server running on http://localhost:${PORT}`);
    }
})

/* die folgende Verbindung brauchen wir gar nicht, wird jeweils bei Bedarf erzeugt (mongoose) */
mongoose.connect(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true })
.then(
    () => console.log('connected to BD')
).catch(
    err => console.error(err, 'conncetion error')
)

const db = mongoose.connection;
```
=== "middleware/upload.js"
```js linenums="1"
const multer = require('multer');
const { GridFsStorage } = require('multer-gridfs-storage');
require('dotenv').config();

const storage = new GridFsStorage({
    url: process.env.DB_CONNECTION,
    options: { useNewUrlParser: true, useUnifiedTopology: true },
    file: (req, file) => {
        const match = ["image/png", "image/jpg", "image/jpeg"];

        if(match.indexOf(file.mimetype) === -1)
        {
            return `${Date.now()}-jf-${file.originalname}`;
        }

        console.log('store');
        return {
            bucketName: 'posts',
            filename: `${Date.now()}-jf-${file.originalname}`, 
            request: req
        }
    }
})

console.log('store', storage)

module.exports = multer({ storage });
```
=== "routes/posts.routes.js"
```js linenums="1"
const express = require('express');
const router = express.Router();
const Post = require('../models/posts')
const upload = require('../middleware/upload')
const mongoose = require('mongoose')
require('dotenv').config()

/* ----------------- POST ---------------------------- */

// POST one post
router.post('/', upload.single('file'), async(req, res) => {
    if(req.file === undefined)
    {
        return res.send({
            "message": "no file selected"
        })
    } else {
        const newPost = new Post({
            title: req.body.title,
            location: req.body.location,
            image_id: req.file.filename
        })
        console.log('newPost', newPost)
        await newPost.save();
        res.send(newPost)
    }
})

/* ----------------- GET ---------------------------- */

const connect = mongoose.createConnection(process.env.DB_CONNECTION, { useNewUrlParser: true, useUnifiedTopology: true });
const collectionFiles = connect.collection('posts.files');
const collectionChunks = connect.collection('posts.chunks');

function getOnePost(id) {
    return new Promise( async(resolve, reject) => {
        try {
            const post = await Post.findOne({ _id: id });
            let fileName = post.image_id;

            collectionFiles.find({filename: fileName}).toArray( async(err, docs) => {

                // sort({n: 1}) --> die chunks nach Eigenschaft n aufsteigend sortieren
                collectionChunks.find({files_id : docs[0]._id}).sort({n: 1}).toArray( (err, chunks) => {
                    
                    const fileData = [];
                    for(let chunk of chunks)
                    {
                        // console.log('chunk._id', chunk._id)
                        fileData.push(chunk.data.toString('base64'));
                    }

                    let base64file = 'data:' + docs[0].contentType + ';base64,' + fileData.join('');
                    let getPost = new Post({
                        "title": post.title,
                        "location": post.location, 
                        "image_id": base64file
                    });

                    resolve(getPost)
                })

            }) // toArray find filename

        } catch {
            reject(new Error("Post does not exist!"));
        }
    })
}

function getAllPosts() {
    return new Promise( async(resolve, reject) => {
        const sendAllPosts = [];
        const allPosts = await Post.find();
        try {
            for(const post of allPosts) {
                console.log('post', post)
                const onePost = await getOnePost(post._id);
                sendAllPosts.push(onePost);
            }
            console.log('sendAllPosts', sendAllPosts)
            resolve(sendAllPosts)
        } catch {
                reject(new Error("Posts do not exist!"));
        }
    });
}

// GET one post via id
router.get('/:id', async(req, res) => {
    getOnePost(req.params.id)
    .then( (post) => {
        console.log('post', post);
        res.send(post);
    })
    .catch( () => {
        res.status(404);
        res.send({
            error: "Post does not exist!"
        });
    })
});

// GET all posts
router.get('/', async(req, res) => {
    
    getAllPosts()
    .then( (posts) => {
        res.send(posts);
    })
    .catch( () => {
        res.status(404);
        res.send({
            error: "Post do not exist!"
        });
    })
});


/* ----------------- DELETE ---------------------------- */

// DELETE one post via id
router.delete('/:id', async(req, res) => {
    try {
        const post = await Post.findOne({ _id: req.params.id })
        let fileName = post.image_id;
        await Post.deleteOne({ _id: req.params.id });
        await collectionFiles.find({filename: fileName}).toArray( async(err, docs) => {
            await collectionChunks.deleteMany({files_id : docs[0]._id});
        })
        await collectionFiles.deleteOne({filename: fileName});
        res.status(204).send()
    } catch {
        res.status(404)
        res.send({ error: "Post does not exist!" })
    }
});

module.exports = router;
```
=== ".env"
```
DB_CONNECTION = mongodb://localhost:27017/posts 
PORT = 3000
```

### Zusammenfassung - die MongoDB `posts`

Hier einige Datensätze für die Datenbank `posts`:

=== "Collection posts"
```json linenums="1"
{"_id":{"$oid":"6278e79c6664ce70884dd0b0"},"title":"WH Eingang","location":"Campus Wilhelminenhof","image_id":"1652090780364-jf-htwbild4.jpg","__v":0}
{"_id":{"$oid":"627a0ff2305433d805b6b437"},"title":"HTW Gebäude C","location":"Campus Wilhelminenhof","image_id":"1652166642127-jf-htwbild5.jpg","__v":0}
{"_id":{"$oid":"627a7cad8ae16b1ba5f62f76"},"title":"Mensastrand","location":"Campus Wilhelminenhof Mensa","image_id":"1652194477890-jf-htwbild1.jpg","__v":0}
{"_id":{"$oid":"627a7cdf8ae16b1ba5f62f80"},"title":"Wiese Campus WH","location":"Campus Wilhelminenhof Gebäude C","image_id":"1652194527176-jf-htwbild2.jpg","__v":0}
{"_id":{"$oid":"627a7d0b8ae16b1ba5f62f84"},"title":"Berühmt wegen FIW","location":"Campus Wilhelminenhof Gebäude C","image_id":"1652194571822-jf-htwbild3.jpg","__v":0}
{"_id":{"$oid":"627a7d398ae16b1ba5f62f8b"},"title":"Haupttor HTW","location":"Wilhelminenhofstraße HTW","image_id":"1652194617191-jf-htwbild4.jpg","__v":0}
{"_id":{"$oid":"627a7d608ae16b1ba5f62f8f"},"title":"Gebäude C","location":"HTW Berlin","image_id":"1652194656102-jf-htwbild5.jpg","__v":0}
```

=== "Collection posts.files"
```json linenums="1"
{"_id":{"$oid":"6278d47e96a41858d66f1621"},"length":984341,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T08:44:46.239Z"},"filename":"1652085886102-jf-htwbild1.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"6278d5f902370f2c675993e9"},"length":1601800,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T08:51:05.478Z"},"filename":"1652086265414-jf-htwbild2.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"6278db738d2b5bc5968f453e"},"length":1601800,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T09:14:27.957Z"},"filename":"1652087667872-jf-htwbild2.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"6278db918d2b5bc5968f4546"},"length":1601800,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T09:14:57.113Z"},"filename":"1652087697071-jf-htwbild2.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"6278dd057c648b9e8e3bbb74"},"length":117492,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T09:21:09.873Z"},"filename":"1652088069823-jf-htwbild3.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"6278e74458477bf1223fa286"},"length":1038579,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T10:04:53.07Z"},"filename":"1652090692995-jf-htwbild4.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"6278e79c6664ce70884dd0ab"},"length":1038579,"chunkSize":261120,"uploadDate":{"$date":"2022-05-09T10:06:20.437Z"},"filename":"1652090780364-jf-htwbild4.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"627a0ff2305433d805b6b435"},"length":25449,"chunkSize":261120,"uploadDate":{"$date":"2022-05-10T07:10:42.191Z"},"filename":"1652166642127-jf-htwbild5.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"627a7cad8ae16b1ba5f62f71"},"length":984341,"chunkSize":261120,"uploadDate":{"$date":"2022-05-10T14:54:37.954Z"},"filename":"1652194477890-jf-htwbild1.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"627a7cdf8ae16b1ba5f62f78"},"length":1601800,"chunkSize":261120,"uploadDate":{"$date":"2022-05-10T14:55:27.23Z"},"filename":"1652194527176-jf-htwbild2.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"627a7d0b8ae16b1ba5f62f82"},"length":117492,"chunkSize":261120,"uploadDate":{"$date":"2022-05-10T14:56:11.84Z"},"filename":"1652194571822-jf-htwbild3.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"627a7d398ae16b1ba5f62f86"},"length":1038579,"chunkSize":261120,"uploadDate":{"$date":"2022-05-10T14:56:57.214Z"},"filename":"1652194617191-jf-htwbild4.jpg","contentType":"image/jpeg"}
{"_id":{"$oid":"627a7d608ae16b1ba5f62f8d"},"length":25449,"chunkSize":261120,"uploadDate":{"$date":"2022-05-10T14:57:36.11Z"},"filename":"1652194656102-jf-htwbild5.jpg","contentType":"image/jpeg"}
```

Die Collection `posts.chunks` ist sehr groß, deshalb [hier zum Download](./files/posts_chunks.json).