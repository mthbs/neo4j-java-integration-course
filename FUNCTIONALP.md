# Lambda Ausdrücke und Funktionale Programmierung

## Lernziele
- Besseres Verständnis von Lambda und seiner Syntax
- Das Einsatzgebiet von Lambdas verstehen
- Iterator und Iterable verstehen
- Die Methoden und Funktionsweise von Streams verstehen und anwenden können

# Lambda Audrücke

- Funkt. Programmieren erlaubt eine Parametrisierung des Verhaltens. Eine Sortiermethode arbeitet zwar immer gleich, aber ihr Verhalten anzupassen bietet nützliche Anwendungsfälle.
- Funkt. Programmierung arbeitet immer mit funkt. Interfaces
- Code = Daten (Erweiterung der Sichtweise für funktionales Programmieren)
## Lambda Ausdruck
- lässt das, was sich der Compiler aus dem Kontext herleiten kann weg.
- Lambdas können (u. a.)
1. innere Klassen ersetzen
```
Arrays.sort(words, (String s1, String s2) -> {return s1.trim().compareTo(s2.trim()); });
```
2. als lokale Variablen deklariert werden
```
Comparator<String> c = (String s1, String s2) -> {return s1.trim().compareTo(s2.trim()); }
Arrays.sort(words, c);
```
- Typ-Inferenz: Der Java-Compiler kann viele Typen aus dem Kontext ablesen.
- Beim Lambda zeigt this immer auf das Objekt, in dem der Lambda-Ausdruck eingebettet ist.
- Lambdas Abkürzen mit Hilfe der Methodenreferenz:
```
Arrays.sort( words, (String s1, String s2) -> StringUtils.compareTrimmed(s1, s2) );
Arrays.sort( words, StringUtils::compareTrimmed);

// Unterschiede bei statischen, Objekt-Typen
Klasse::statischeMethode
obj::objektMethode
TypVonObjekt::objektMethode
```
- Konstruktorreferenz:
```
interface DateFactory { Date create(); }

DateFactory factory = Date::new;
System.out.println( factory.create() );
// Die letzten beiden Zeilen zusammengefasst:
System.out.println( ((DateFactory) Date::new).create() );
```


# Funktionale Programmierung mit Java

- Es ist hilfreich funkt. Programmierung gedanklich von der Mathematik zu lösen, denn die meisten Programme haben nichts mit mathematischen Funktionen zu tun, aber mit formal beschriebenen Methoden.
- Der Gedanke bei funtkionalen Programmiersprachen ist der, ohne Zustände auszukommen.
- 
## Programmierparadigmen

- Programmierparadigma: Beschreibungsform, in der der Entwickler sein Problem beschreibt, damit der Computer es ausführen kann.
1. imperative Programmierung: Java-Most-Used: Anweisungen stehen im Mittelpunkt. Imperativ = Befehlsform = tue dies, tue das. Beschreibt das Wie zur Problemlösung
    - Beispiel: Berechnung der Fakultät (1)
2. deklarative Programmierung: Beschreibt das Was, also die Anforderungen ohne sich in genauen Abläufen zu verstricken. Zum Beispiel Suche nach *.html oder Datenbankabfragen. Sind kürzer, leichter Erweiterbar, verständlicher. Eine deklarative Beschreibung braucht eine Art Ablaufumgebung. Typische Algorithmen können auch deklarativ formuliert werden.
3. funktionale Programmierung: Funktionen stehen im Mittelpunkt. Im Idealfall besteht ein zustandsloses Verhalten, in dem viel mit Rekursion gearbeitet wird.
   - Beispiel: Berechnung der Fakultät (2)

### Beispiel: Berechnung der Fakultät:

```
// imperativer Weg:
public static int factorial( int n ){
    int result = 1;
    for( int i = 1; i <= n; i++ )
        result *= i;
    return result;
```
```
// funktionaler Weg:
public static int factorial( int n ){
    return n == 0 ? 1 : n * factorial( n-1 );
}
```

## Funkt. Programmierung mit Java

- Wenn Funktionen als Argumente übergeben werden
- Wenn Funktionen zurückgegeben werden
- Eine Kombination aus beiden Konzepten ist Zukunftsweisend

### Anwendungsgebiete

| Code                                     | Bedeutung                                               |
|------------------------------------------|---------------------------------------------------------|
| Comparator c = (c1,c2)-> ...             | Implementiert eine Funktion über einen Lambda-Ausdruck. |
| Arrays.sort(T[] a, COmparator c)         | Nimmt eine Funktion als Argument an.                    |
| Collections.reverseOrder(Comparator cmp) | Nimmt eine Funktion an und liefert auch eine zurück.    |

# Comparator

```
Comparator.naturalOrder();
```

# Stream-API

- Die Aufgabe ist es die Daten komfortabel zu erfragen
- Im Zentrum stehen Funktionen zum Filtern, Abbilden und Reduzieren von Daten.
- parallelStream() ist günstig für die Arbeit mit Threads
- collect(): Ergebnisse in einen Container schreiben
- map(): z.B.: map( Integer::toUnsignedString )
- Gruppieren
  - collect(Collectors.groupingBy( Thread::getState )); --> Map<Thread.State, List<Thread>>
- sorted()
- filter()
  - lässt alle Elemente da, die dem Kriterium genügen. If-Pardon.
- distinct

# Pause bei Seite 823, wegen Übergang zum Thema Iterator und Maps
