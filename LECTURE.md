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
1. Atomic
2. Consistent
3. Isolated
4. Durable
- Der Driver bietet 3 Typen von Transaktionen an:
1. Auto-commit Transactions: Einzelne Einheit die direkt ausgeführt wird, soll nicht mit Neo4j-Klustern genutzt werden
```
var query = "MATCH () RETURN count(*) AS count";
var params = Values.parameters();
// Run a query in an auto-commit transaction
var res = session.run(query, params).single().get("count").asLong();
```
2. Read Transactions: Zum lesen
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