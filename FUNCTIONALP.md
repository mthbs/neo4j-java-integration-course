# Lambda Ausdrücke und Funktionale Programmierung

- Funkt. Programmieren erlaubt eine Parametrisierung des Verhaltens. Eine Sortiermethode arbeitet zwar immer gleich, aber ihr Verhalten anzupassen bietet nützliche Anwendungsfälle.
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


# 12.6 (Seite 817) Funktionale Programmierung mit Java 