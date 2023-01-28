# Einstieg
Hier werden alle relevanten Informationen, Java Methoden und vieles mehr dokumentiert um einen möglichst unkomplizierten Eistieg in Neo4j zu erhalten.

## Installation 
Der größte Teil ist bereits installiert. Ich dokumentiere nur die für mich fehlenden Inhalte.

Die Anwendung/ Vorgehensweise

- Client Neoflix
- Framework-agnostic (Die Inhalte funktionieren unabhängig vom Framework)
- Zu Beginn wird Theorie beigebracht, um anhand von diesem Wissen anschließend verschiedene Funktionen in der API zu implementieren.
- Das Projekt läuft nur auf Java 17 und mit Maven. Der Webserver arbeitet auf Javalin. Die Authentifizierung läuft mit AUth0 und JWT Tokens. Passwörter werden mit bcrypt gemanaged. Getestet wird mit JUnit5.


## Neo4j-Projekt Orientierung

- Neo4j Sandbox ist ein kostenloses Service, welches die perfekte Entwicklungsumgebung ist, um mit Neo4j zu experimentieren.
-

## Java Funktionen

- Es werden die application.properties durch (1) abgerufen und alle Einstellungen sind dadurch mit (2) abrufbar.
```
(1) AppUtils.loadProperties()
(2) System.getProperty
```

# Der Neo4j Java Driver

Lernziele dieser Lektion

- Wissen wie der Java Driver installiert wird
- Den Aufbau des Connections-Strings verstehen
- Eine Instanz des Drivers soll erstellt werden können
- Die Verbindung überprüfen

## Der Driver

- Wird genutzt um eine Cypher-Abfrage in einer Graph-Datenbank auszuführen
- Es soll eine einzige Instanz des Drivers kreiert werden, welcher mit der ganzen Anwendung geteilt werden kann
- Maven Dependency wird vorausgesetzt
- So wird der Driver installiert, dabei benötigt er zwei Parameter, nämlich: den Connection String und den AuthToken.
```
// Import all relevant classes from neo4j-java-driver dependency
import org.neo4j.driver.*;
static String username = "neo4j";
static String password = "letmein!";
// Create a new Driver instance
   static Driver driver = GraphDatabase.driver("neo4j://localhost:7687",
            AuthTokens.basic(username, password));
// Check the Connectivity
driver.verifyConnectivity();
```
- So wird detaillierter eine Lokale Instanz des Drivers erstellt
```
var driver = GraphDatabase.driver(
  connectionString, // (1)
  authenticationToken, // (2)
  configuration // (3)
)
```

## Der Connection String

```
Scheme :// Initial Server Address : Port ? Additional Configuration
```

### Schema

- neo4j: unverschlüsselt
- neo4j+s: verschlüsselt + Neo4j Aura
- neo4j+ssc: verschlüsselt ohne automatische Verifikation
- 
- Bolt-Schema: Kann genutzt werden wenn man sich direkt mit einer Datenbank verbindet
- bolt: unverschlüsselt
- bolt+s: verschlüsselt
- bolt+ssc: verschlüsselt ohne automatische Verifikation
- 
- Die aktuell eingesetzte Verschlüsselung kann in neo4j.conf eingesehen werden
```
dbms.connector.bolt.enabled
```

### Additional COnfiguration hinter ?
- z.B. Latenzen reduzieren um die Performance zu verbessern

### Authentifizierungstoken

```
AuthToken authenticationToken = AuthTokens.basic(username, password);
```

### Zusätzliche Driver-Konfiguration

- Nutze dabei:
```
Config config = Config.builder(). ... .build();
```

# Interaktionen mit Neo4j

## Nach dem Kapitel kannst du:
- Eine Sitzung eröffnen und eine Arbeitseinheit innerhalb einer Transaktion eröffnen
- Lese und Schreib DB-Abfragen über den Driver ausführen
- Die Ergebnisse interpretieren
- Potentielle Fehler vom Driver beheben

## Sitzungen (Sessions) und Transaktionen (Transactions)

### Session

- Eine Session ist ein Kontainer von einer Sequenz aus Transaktionen
- Eine Sitzung eröffnen:
```
var session = driver.session();
```
- Eine Session soll mit der Try-With-Klausel genutzt werden, weil eine Session nach ihrer Nutzung geschlossen werden muss
```
try (var session = driver.session()) {
    // mache etwas mit der Session
}
```
- Eine Session kann optional konfiguriert werden
```
var sessionConfig = SessionConfig.builder()
        .withDefaultAccessMode(AccessMode.WRITE)
        .withDatabase("people")
        .build();

try (var session = driver.session(sessionConfig)) {
```
- Ohne eine genannte Datenbank wird eine default-Datenbank genutzt, es kann in neo4j.conf eingesehen werden

### Transaktionen

- Eine Transaktion besteht aus Arbeits-Einheiten mit der Datenbank
- Eine Transaktion muss ACID sein:
1. Atomic: Es wird nur die volle Transaktion durchgeführt, bei Fehlern erfolgt ein Abbruch
2. Consistent: Nur gültige Daten werden gespeichert
3. Isolated: Jede Read oder Write Transaktion steht für sich alleine und wird nicht von anderen Transaktionen beeinflusst
4. Durable: Gespeicherte Daten gehen nicht verloren, auch nicht bei externen Problemen (z.B. Stromausfall)
- Der Driver bietet 3 Typen von Transaktionen an:
1. Auto-commit Transactions: Einzelne Einheit die direkt ausgeführt wird, soll nicht mit Neo4j-Klustern genutzt werden
- Standard Zugriffsmodus ist WRITE, executeRead() bzw. executeWrite() überschreibt den Wert
```
var query = "MATCH () RETURN count(*) AS count";
var params = Values.parameters();
// Run a query in an auto-commit transaction
var res = session.run(query, params).single().get("count").asLong();
```
2. Read Transactions: Zum lesen
- executeRead()
```
// Run a query within a Read Transaction
var res = session.readTransaction(tx -> {
return tx.run("""
                MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
                WHERE m.title = $title // (1)
                RETURN p.name AS name
                LIMIT 10
                """,
        Values.parameters("title", "Arthur") // (2)
).list(r -> r.get("name").asString());
```
3. Write Transactions: Zum schreiben in die Datenbank
```
session.writeTransaction(tx -> {
  return tx.run(
          "CREATE (p:Person {name: $name})",
          Values.parameters("name", "Michael")).consume();
});
```
4. Zusätzlich!: Manuelles Erstellen von Transaktionen
```
// Open a new session
var session = driver.session(
        SessionConfig.builder()
                .withDefaultAccessMode(AccessMode.WRITE)
                .build());

// Manually create a transaction
var tx = session.beginTransaction();
```
- Operationen mit der Transaktion
```
try {
  // Perform an action
  tx.run(query, params);

  // Commit the transaction
  tx.commit();
} catch (Exception e) {
  // If something went wrong, rollback the transaction
  tx.rollback();
}
```
### Session schließen
```
session.close()
```

# Ergebnisse verarbeiten

## APIs

Der Java Driver stellt drei APIs zur Verfügen um Ergebnisse zu laden/ konsumieren

- Synchronous API: Most common/ Gebräuchlichste API
- Asynch API: You need different entry-points and helpers like a reactive framework/ Es werden unterschiedliche Einstiegspunkte und Helfer benötigt wie ein Reaktives Framework. Im Gegenzug erhält man effizientere und mehr zugeschnittene Ergebnisse
- Reactive API: You need different entry-points and helpers like a reactive framework/ Es werden unterschiedliche Einstiegspunkte und Helfer benötigt wie ein Reaktives Framework. Im Gegenzug erhält man effizientere und mehr zugeschnittene Ergebnisse

### Synchronous API

- wenn session.run() und/ oder tx.run() ausgeführt wird, gibt die Abfrage ein Ergebnis-Objekt zurück. Die Ergebnisse sind nur nach dem Erreichen des letzten Record ersichtlich.
```
try (var session = driver.session()) {

    var res = session.readTransaction(tx -> tx.run(
            "MATCH (p:Person) RETURN p.name AS name LIMIT 10").list());
    res.stream()
            .map(row -> row.get("name"))
            .forEach(System.out::println);
} catch (Exception e) {
    // There was a problem with the
    // database connection or the query
    e.printStackTrace();
}
```

### Asynch API

```
var session = driver.asyncSession();
session.readTransactionAsync(tx -> tx.runAsync(
                "MATCH (p:Person) RETURN p.name AS name LIMIT 10")

        .thenApplyAsync(res -> res.listAsync(row -> row.get("name")))
        .thenAcceptAsync(System.out::println)
        .exceptionallyAsync(e -> {
            e.printStackTrace();
            return null;
        })
);
```

### Reactive API
```
Flux.usingWhen(Mono.fromSupplier(driver::rxSession),
    session -> session.readTransaction(tx -> {
        var rxResult = tx.run(
                "MATCH (p:Person) RETURN p.name AS name LIMIT 10");
        return Flux
            .from(rxResult.records())
            .map(r -> r.get("name").asString())
            .doOnNext(System.out::println)
            .then(Mono.from(rxResult.consume()));
    }
    ), RxSession::close);
```

## Ergebnisse

- Das Result-Objekt enthält die gefundenen records/ Ergebnisse und zusätzlich ein Set mit Meta-Daten über die Abfrage und Ergebnisse
- Als record/ Ergebnis wird die individuelle Zeile von Ergebnissen bezeichnet. Auf diese kann auf verschiedene Weise zugegriffen werden, z.B. mit Iterator<Record>, stream(), list() oder single().
- Ein Record bezieht sich auf das RETURN-Statement in der Abfrage
- Die Metadaten sind durch die Methode "Result too(see below)" erreichbar.

### Einige ZUgriffsmöglichkeiten
- Einfacher Zugriff:
```
// Get single row, error when more/less than 1
Record row = res.single();
// Materialize list
List<Record> rows = res.list();
// Stream results
Stream<Record> rowStream = res.stream();

// iterate (sorry no foreach)
while (res.hasNext()) {
    var next = res.next();
}
```
- Detaillierterer Zugriff z.B. durch Filtrierung nach Spaltenname
```
// column names
row.keys();
// check for existence
row.containsKey("movie");
// number of columns
row.size();
// get a numeric value (int, long, float, double also possible)
Number count = row.get("movieCount").asInt(0);
// get a boolean value
boolean isDirector = row.get("isDirector").asBoolean();
// get node
row.get("movie").asNode();
```

## Meta-Daten: ResultSummary

```
ResultSummary summary = Result.consume();
```
ResultSummary beinhaltet folgende Komponenten:

- Statistiken: Wie viele Knoten/ Beziehungen erstellt, updated/ aktualisiert oder entfernt wurden.
- Abfrage-Typ
- Datenbank und Server Infos
- query plan/ Abfrageplan mit und ohne profile
- Benachrichtigung
- Mehr Informationen in der API docs für ResultSummary

### Beispiel: Abfrage, wie lange die Abfrage gebraucht hat
```
ResultSummary summary = res.consume();
// Time in milliseconds before receiving the first result
summary.resultAvailableAfter(TimeUnit.MILLISECONDS); // 10
// Time in milliseconds once the final result was consumed
summary.resultConsumedAfter(TimeUnit.MILLISECONDS); // 30
```

### SUmmaryCounters

- SummaryCounters ist über counters() erreichbar und funktioniert nur mit Write-Statements, es zeigt Update counts an, dieses kann mit counters.containsUpdates() überprüft werden.
```
SummaryCounters counters = summary.counters();
// some example counters
// nodes and relationships
counters.containsUpdates();
counters.nodesCreated();
counters.labelsAdded();
counters.relationshipsDeleted();
counters.propertiesSet();

// indexes and constraints
counters.indexesAdded();
counters.constraintsRemoved();
// updates to system db
counters.containsSystemUpdates();
counters.systemUpdates();
```

# Das Neo4j Typsystem

- Neo4j wurde in Java geschrieben, das j im Namen steht für java.
- Viele Datentypen wie LocalTime oder Node sind gleich zu den Java-Datentypen. Es gibt in mancherlei Hinsicht auch Unstimmigkeiten (in der Verarbeitung)
- Es gibt (u.a.) folgende Datentypen, verglichen zwischen Java und Cypher:

| Java-Datentyp                                                                                                            | Cypher-Datentyp | Bemerkungen |
|--------------------------------------------------------------------------------------------------------------------------|-----------------|------------:|
| List                                                                                                                     | List, Array     | Neo4j kann nur flache Arrays mit Strings, booleans oder zahlen speichern | 
| Long                                                                                                                     | Integer         |
| Double                                                                                                                   | Float           |
| IsoDuration                                                                                                              | Duration        |
| null, Map, Boolean, String, byte[], LocalDate, Time, LocalTime, DateTime, LocalDateTime, Point, Node, Relationship, Path | dasselbe        |

- Casting: Im Beispiel unten wird Year als Number gecastet
```
as{Type}()
```
```
Value nodeValue = row.get("movie");

// key names
Iterable<String> keys = nodeValue.keys();
// number of values / length of list
nodeValue.size();
// get all contained values
Iterable<Value> values = nodeValue.values();

// treat value as type, e.g. Node, Relationship, Path or primitive
Node node = nodeValue.asNode();

// string-key accessors
Value titleValue = node.get("title");
String title = titleValue.asString();
Number year = node.get("year").asNumber();
Integer releaseYear = node.get("year").asInt(0);

// index based accessors for lists and records
nodeValue.get(0);
```

## Zahlen

- Alle Ganzzahlen: Integer (äquivalent zu long)
- Alle Gleitkommazahlen: Float (äquivalent zu double)
- BigDecimal oder BigInteger wird nicht unterstützt.

## Zeit-Datentypen

- Es wird java.time.* genutzt. Eine einzige AUsnahme ist IsoDuration, wofür ein eigener Datentyp genutzt wird.
- Date: 2022-01-02
- DateTime: 2022-01-02T01:02:03+04:00
- LocalDateTime: 2022-01-02T01:02:03+04:00
- LocalTime: 12:34:56
- OffsetTime: 12:34:56+04:00
- IsoDuration: P1M12D
```
// Driver consumes regular java.time.* datatypes
// Temporal Types from Value
var released = nodeValue.get("released");
released.asLocalDate();
released.asLocalDateTime();
released.asLocalTime();
released.asOffsetDateTime();

// custom duration type
IsoDuration duration = released.asIsoDuration();
```

## Knoten, Beziehungen und Pfade

- Superklasse Entity für Knoten und Beziehungen
### Knoten

```
// Execute a query within a read transaction
Result res = session.readTransaction(tx -> tx.run("""
                MATCH path = (person:Person)-[actedIn:ACTED_IN]->(movie:Movie)
                RETURN path, person, actedIn, movie,
                       size ( (person)-[:ACTED]->() ) as movieCount,
                       exists { (person)-[:DIRECTED]->() } as isDirector
                LIMIT 1
        """));
// Get a node
Node person = row.get("person").asNode();
```
Eine Knoten-Instanz hat drei Bestandteile
```
var nodeId = person.id(); // (1)
var labels = person.labels(); // (2) e.g. ['Person','Actor']
var properties = person.asMap(); // (3) e.g. {"name": "Tom Hanks", "tmdbId": "31" }
// Alternativer ZUgriff auf die Properties:
Value properties = Entity.get(name);
```
- id: interne Knoten-Id. Eine interne Id (!) kann mehrfach genutzt werden. Nutze stattdessen die Business-Keys anstatt den internen Ids.
- labels: ein Iterable<String>
- properties: eine Java Map mit allen EIgenschaften des Knotens

### Beziehungen

```
var actedIn = row.get("actedIn").asRelationship();

var relId = actedIn.id(); // (1)
String type = actedIn.type(); // (2)
var relProperties = actedIn.asMap(); // (3)
var startId = actedIn.startNodeId(); // (4)
var endId = actedIn.endNodeId(); // (5)
```
- identity: interne Id, nicht nutzen siehe Infos bei Knoten
- type: z.B. ACTED_IN
- properties: Java-Map: z.B. {"role" : "Woody"}
- startNodeId: interne Id (!) wird genutzt
- endNOdeId: interne Id (!) wird genutzt

### Pfade

- Es handelt sich hierbei um ein Pfad von Knoten und BEziehungen, als Instanz von Path
```
Path path = row.get("path").asPath();

Node start = path.start(); // (1)
Node end = path.end(); // (2)
var length = path.length(); // (3)
Iterable<Path.Segment> segments = path; // (4)
Iterable<Node> nodes = path.nodes(); // (5)
Iterable<Relationship> rels = path.relationships(); // (6)
```
- start: Knoten, wo der Pfad startet
- end: Knoten, wo der Pfad endet
- length: Alle Segmenten werden gezählt
- segments: Ein Pfad ist ein Iterable von Path.Segment
- nodes: Ein Iterable von den Knoten des Pfades
- relationships: Ein Iterable von den Beziehungen des Pfades

### Pfad-Segmente: PathSegment-objects

- Das Beispiel kann in zwei Segmente unterteilt werden, nämlich ...
```
p:Person)-[:ACTED_IN]->(m:Movie)-[:IN_GENRE]->(g:Genre)
```
- ... in:
```
1. (p:Person)-[:ACTED_IN]->(m:Movie)
2. (m:Movie)-[:IN_GENRE]->(g:genre)
```
```
path.forEach(segment -> {
    System.out.println(segment.start());
    System.out.println(segment.end());
    System.out.println(segment.relationship());
});
```

PathSegment-Objekte haben drei Eigenschaften
1. relationsship
2. start
3. end

## Die Werte in Java-Typen konvertieren

- Value und Record-Typen haben drei Funktionen
1. asObject()
2. asMap()
3. list()
- Zusätzliche Helfer-Funktionen
1. is{Type}: Fürs Casten, siehe oben
2. isTrue und isFalse 
3. isNull
4. isEmpty
