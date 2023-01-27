# Einstieg
Hier werden alle relevanten Informationen, Java Methoden und vieles mehr dokumentiert um einen möglichst unkomplizierten Eistieg in Neo4j zu erhalten.

# Installation 
Der größte Teil ist bereits installiert. Ich dokumentiere nur die für mich fehlenden Inhalte.

Die Anwendung/ Vorgehensweise

- Client Neoflix
- Framework-agnostic (Die Inhalte funktionieren unabhängig vom Framework)
- Zu Beginn wird Theorie beigebracht, um anhand von diesem Wissen anschließend verschiedene Funktionen in der API zu implementieren.
- Das Projekt läuft nur auf Java 17 und mit Maven. Der Webserver arbeitet auf Javalin. Die Authentifizierung läuft mit AUth0 und JWT Tokens. Passwörter werden mit bcrypt gemanaged. Getestet wird mit JUnit5.

# Der Neo4j Java Driver

Lernziele dieser Lektion

- Wissen wie der Java Driver installiert wird
- Der Aufbau des Connections-Strings soll verstanden werden
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
- 

# Neo4j-Projekt Orientierung

- Neo4j Sandbox ist ein kostenloses Service, welches die perfekte Entwicklungsumgebung ist, um mit Neo4j zu experimentieren.
- 

# Java Funktionen

- Es werden die application.properties durch (1) abgerufen und alle Einstellungen sind dadurch mit (2) abrufbar.
```
(1) AppUtils.loadProperties()
(2) System.getProperty
```
- sdsdsdsd