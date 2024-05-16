# Übung 9 – Slim-Framework

HYP2UE_T1 Hypermedia 2 Serverseitige Programmierung | 27.05.2024 | Wolfgang Hochleitner | Code-along

Diese Übung dient als praktische Einführung in das Slim-Framework. Anhand des `slim/slim-skeleton`-Projekts wird der grundsätzliche Aufbau einer Slim-Applikation analysiert und um einen Datenbankzugriff und das Rendern von Twig-Templates erweitert.

## Repository clonen und Branches erstellen

Clonen Sie im ersten Schritt das Repository ihres GitHub-Classroom-Teams, damit sie lokal (im Docker-Container) damit arbeiten können. Verwenden Sie dazu PhpStorm ("Get from VCS") oder einen Git-Client Ihrer Wahl.

Clonen Sie das Repository in das `webapp` Verzeichnis des Docker Containers, sodass das Projekt im Verzeichnis `webapp/t1-ue09-YourTeamName` zu liegen kommt.

Für diese Code-Along-Übung empfiehlt es sich, dass beide Teammitglieder getrennt voneinander arbeiten, um selbständig die Inhalte umsetzen zu können. Es wird empfohlen, dass sich jede Person einen eigenen Branch anlegt und dort ihren Zwischenstand committed und pusht. Unter "Git → New Branch..." wird ein neuer Branch mit beliebigem Namen (z.B. Name des Teammitglieds) erstellt. Ist "Checkout branch" angehakt, wird gleich auf diesen Branch gewechselt. Beim ersten Push wird dieser Branch automatisch auf GitHub angelegt.

## slim/slim-skeleton installieren

Erzeugen Sie, analog zur Arbeit mit dem `fhooe/router-skeleton`, ein Basisprojekt mit [slim/slim-skeleton](https://packagist.org/packages/slim/slim-skeleton). Dieser Installiert das Slim-Framework (`slim/slim`), eine PSR-7-Implementierung (`slim/psr7`), einen Dependency-Injection Container (`php-di/php-di`) und einen Logger (`monolog/monolog`). Darüber hinaus sind einige Klassen enthalten, welche die Umsetzung der von Slim verwendeten [Action-Domain-Responder](https://github.com/pmjones/adr)-Architektur demonstrieren.

Starten Sie zunächst eine Bash-Shell im `webapp` Container:

```shell
docker exec -it webapp /bin/bash
```

Wechseln Sie in den Unterordner `t1-ue09-YourTeamName` und lassen Sie Composer den `slim/slim-skeleton` in ein Unterverzeichnis ihrer Wahl installieren, z.B. `slim`.

```bash
cd t1-ue09-YourTeamName/
composer create-project slim/slim-skeleton slim
```

### Basispfad anpassen

Öffnen Sie `public/index.php` und passen Sie in Zeile 39 den Basispfad an:

```php
$app->setBasePath("/t1-ue09-YourTeamName/slim/public");
```

Öffnen Sie die Hauptseite des Skeletons: 

http://localhost:8080/t1-ue09-YourTeamName/slim/public/

Der Text "Hello world!" wird angezeigt.

## Die Struktur des Slim-Skeletons erkunden

Wenn Sie die Struktur des Projekts bzw. die Datei `public/index.php` genauer betrachten, fällt auf, dass die Definitionen der Routen, Dependencies, Middleware und noch einiger Dinge nicht dort passieren, sondern in Dateien im Ordner `app` ausgelagert sind:

- `app/dependencies.php`: Definition der Dependencies im DI-Container.
- `app/middleware.php`: Einbinden der benötigten Middleware.
- `app/repositories.php`: Bekanntmachen der Repositories (Datenquellen) im DI-Container.
- `app/routes.php`: Definition der Routen und Aufrufen von Actions.
- `app/settings.php`: Definitionen der Einstellungen für die App und Dependencies im DI-Container.

### Routen testen

In `app/routes.php` sind drei Routen hinterlegt:

- `GET /`: Die Index-Route. Zeigt "Hello world!" an.
- `GET /users`: Zeigt eine Liste aller im System gespeicherten User\*innen an.
- `GET /users/{id}`: Zeigt den User mit der übergebenen `id` an.

Testen Sie die letzten beiden Routen und betrachten Sie die JSON-Ausgabe.

#### Anmerkungen zum Testen

Sollte ein Fehler angezeigt werden, dass Berechtigungen zum Schreiben fehlen, so darf der Logger nicht ins Verzeichnis `logs` schreiben. Führen Sie `chmod -R 777 .` im Verzeichnis `slim` aus, um überall Berechtigungen zu erstellen.

Die Routen `GET /users/1` bis `GET /users/5` zeigen eine Deprecation-Notice an:

> **Deprecated: Using ${var} in strings is deprecated, use {$var} instead in /var/www/html/t1-ue09-YourTeamName/slim/src/Application/Actions/User/ViewUserAction.php on line** *19*

Diese Warnung wird angezeigt, weil eine veraltete Syntax zur String-Interpolation in der Datei `ViewUserAction.php` zum Einsatz kommt. Diese wurde mit PHP 8.2 als deprecated markiert. Editieren Sie die Datei und verlagern die das $-Zeichen in die geschwungenen Klammern, um das Problem zu beheben.

### Die Klassen für die Applikationslogik

Im Ordner `src` sind die Klassen mit der Applikationslogik enthalten. Sie sind entsprechend der Action-Domain-Responder-Architektur angelegt. Es gibt also Action-Klassen, Domain-Klassen und eine Responder-Klasse. Die Struktur ist in drei Unterordner aufgeteilt:

- `Application`: Hier ist die Applikationslogik abgelegt. Diese ist durch weitere Unterordner gegliedert:
  - `Actions`: Die Action-Klassen sind für eine Aktion (ausgelöst durch eine Route) verantwortlich. Z.B. Alle User\*innen auflisten oder eine\*n bestimmten User\*in ausgeben.
  - `Handlers`: Diese Klassen werden im Fehlerfall aufgerufen und kümmern sich um entsprechende Ausgaben.
  - `Middleware`: Hier wird selbst geschriebene Middleware abgelegt.
  - `ResponseEmitter`: Diese Klasse kümmert sich um das Zurückgeben der Response.
  - `Settings`: Diese Klassen kapseln die Einstellungen für den DI-Container.
- `Domain`: Hier wird die Domainlogik untergebracht. Welche Klassen repräsentieren Daten und Datenspeicher? Hier finden sich eine Klasse für `User` und ein Interface für das `UserRepository`. Auch entsprechende Exceptions werden hier abgelegt.
- `Infrastructure`: In diesem Verzeichnis werden Klassen für die dahinterliegende Infrastruktur, d.h. die Datenspeicherung abgelegt. Hier liegt etwa eine Implementierung für ein `InMemoryUserRepository`, das User\*innen in PHP-Datenstrukturen speichert.

## Den Slim-Skeleton erweitern

Um mit dem Skeleton zu arbeiten, werden nun zwei Funktionalitäten hinzugefügt.

- Das Repository wird vom `InMemoryUserRepository` auf eine Variante umgestellt, welche die User\*innen mit PDO aus einer Datenbank ausliest (`DbUserRepository`).
- Twig-Templates werden als Dependency eingebunden, um damit die Index-Seite anzuzeigen.

### Ein Datenbank-Repository erstellen

Um das `InMemoryUserRepository` durch ein datenbankbasiertes abzulösen, sind die folgenden Schritte notwendig.

1. Zunächst wird in PhpMyAdmin die Datenbank für dieses Beispiel importiert: Unter http://localhost:8082/ wird PhpMyAdmin geöffnet. Über den Punkt "Importieren" wird die Datei `db/ue09_users.sql` importiert. Dies erzeugt die Datenbank "ue09_users" mit der Tabelle "user".
2. In `app/settings.php` werden die Einstellungen für PDO unter dem Schlüssel `db` hinzugefügt. Benötigt werden Host, Port, Datenbank, User, Passwort und PDO-Flags. Um nicht alles tippen zu müssen, kann der Eintrag hier kopiert werden:
   ```php
   'db' => [
       'host' => 'db',
       'port' => 3306,
       'database' => 'ue09_users',
       'username' => 'hypermedia',
       'password' => 'geheim',
       'flags' => [
           PDO::ATTR_PERSISTENT => true,
           PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
           PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
           PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8 COLLATE utf8_general_ci",
           PDO::MYSQL_ATTR_MULTI_STATEMENTS => false
       ]
   ],
   ```
3. In `app/dependencies.php` wird dem DI-Container mitgeteilt, wie ein PDO-Objekt erzeugt wird. Als Schlüssel sollte hier `PDO::class` verwendet werden, damit der Container korrektes Autowiring (Einfügen der Objekte) in den Konstruktoren der Action-Klassen durchführen kann.
4. Im Ordner `src/Infastructure/Persistence/User` wird die Klasse `DbUserRepository` angelegt. Sie implementiert das Interface `UserRepository`.
5. Der Konstruktor hat ein PDO-Objekt als Parameter. Der DI-Container kümmert sich um das Erzeugen und Einfügen des Objekts. Das Objekt wird einer Eigenschaft zugewiesen, damit es in allen Methoden verfügbar ist.
6. Die Methode `findAll()` wird implementiert: Hier werden alle Einträge aus der Tabelle `user` abgefragt und als Array (`PDO::FETCH_ASSOC`) zurückgegeben.
7. Die Methode `findUserOfId(int $id)` wird implementiert: Hier wird ein\*e User\*in mit einer gegebenen ID als `User`-Objekt zurückgegeben. Existiert diese\*r nicht, wird eine `UserNotFound`-Exception geworfen.
8. Um das Repository zu verwenden, wird in `app/repositories.php` die Zuordnung vom `InMemoryUserRepository` auf das `DbUserRepository` umgestellt.

### Twig-Templates zur Ausgabe von HTML einrichten

Um mit Twig HTML-Ausgabe erzeugen zu können, sind folgende Schritte notwendig:

1. Im Ordner `slim` wird die Komponente `slim/twig-view` mit composer eingebunden: `composer require slim/twig-view`.
2. In `app/settings.php` werden die Einstellungen unter dem Schlüssel `view` hinzugefügt. Benötigt werden der Template-Pfad, der Cache-Pfad und die Einstellung für Auto-Reload.
3. In `app/dependencies.php` wird dem DI-Container mitgeteilt, wie ein Twig-Objekt erzeugt wird. Als Schlüssel wird `Twig::class` verwendet.
4. In `app/middleware.php` wird die `TwigMiddleware` hinzugefügt.
5. Einen Ordner `templates` (im Verzeichnis `slim`) erstellen und dort ein Template `index.html.twig` anlegen. Es empfiehlt sich, von PhpStorm zunächst eine Datei `index.html` anzuglegen (um das HTML-Grundgerüst zu erhalten) und dieser dann die Erweiterung `.twig` zu geben. Das Template erhält belieben Inhalt (z.B. "Hello world!");
6. In `src/Application/Actions` eine Klasse `IndexAction` erstellen. Die Klasse erbt von `Action`.
7. Der Konstruktor erhält das Twig-Objekt und weist es einer Eigenschaft zu, sodass es in allen Methoden verfügbar ist.
8. In der Methode `action()` wird `render()` beim Twig-Objekt aufgerufen und das Ergebnis (die Response) zurückgegeben.
9. In `app/routes.php` wird bei der Index-Route statt der Callback-Funktion `IndexAction::class` angegeben, um die Aktion bei der Route auszulösen.

## Tipps und Richtlinien

- Bei Fragen oder Problemen zur Aufgabe eröffnen Sie ein Issue in ihrem Repository.
