# Maps und Iterators

## Lernziele

- Iterator und Iterable verstehen
- Die Methoden und Funktionsweise von Maps verstehen und anwenden können

# Iterator und Iterable

- Iterator und Iterable sind Schnittstellen, zum behandeln von Datensammlungen, wie zB Arrays, Listen, Sets, ...
- Iterator: Erlaubt Schritt für Schritt eine Sammlung zu durchlaufen
- Iterable: Liefert einen Iterator. Ist sein Produzent

## Schnittstelle Iterator
|                                     Iterator |
|---------------------------------------------:|
|                          hasNext() : boolean |
|                                   next() : E |
|                              remove() : void |
| forEachRemaining(Consumer<? super E>) : void |



- Benötigt Methoden, die:
  - die das nächste Element liefert
  - die Auskunft darüber gibt, ob der Datengeber noch weitere Elemente zur Verfügung stellt
  - 
  - hasNext(): Hast du mehr?
  - next(): Gib mir das Nächste!
  - 
- Iterieren sieht immer gleich aus:
```
while ( iterator.hasNext() ) {
    process ( iterator.next() );
}
```
- Es gibt kein zurück, iterieren geht immer nur vorwärts.
- die Iterator-Methode forEachRemaining()Consumer<? super E> action) führt ein belibiges Codefragment auf jedem Element aus
  - Somit kann die Iteration in ein Lambda-Ausdruck umgewandelt werden.
```
new Scanner ( System.in ).forEachRemaining( System.out::println ); 
```
- die Iterator-Methode remove löscht das zuletzt mit next() gelieferte Objekt.
```
Iterator<Path> iterator = Paths.get( "/chris/brain/java/9" ).iterator();
Iterator<Integer> iterator2 = new ArrayList<>( Arrays.asList( 4, 5, 8 ,2 )).iterator();
Iterator<String> iterator3 = new Scnanner("Hund Katze Maus);
while ( iterator.hasNext() )
  System.out.println ( iterator.next() );
```

## Schnittstelle Iterable

|                            Iterable |
|------------------------------------:|
|            iterator() : Iterator<T> |
| forEach(Consumer<? super T>) : void |
|      spliterator() : Spliterator<T> |

- Beispielsweise implementiert TreeSet Iterable immer mit

### Abrufen von Iterables: for-each

- forEach läuft alles ab, was vom Typ Iterable ist (Vorgang mir schon bekannt zB Liste-> forEach)
- forEach macht das Automatisch, was wir vorhin mit Iterator, while, hasNext() und next() gemach haben.
- Mann kann eigene Klassen von Iterable implementieren
  - rechts hinter Doppelpunkt in forEach
  - ein passender Iterator muss erzeugt werden und dessen Methoden hasNext() und next() implementiert werden
    - Es kann auch der Iterator implementiert werden: 
```
class WordIterable implements Iterable<String<, Iterator<String> {
  
  private StringTokenizer st;
  
  public WordIterable( String s ) {
    st = new StringTokenizer( s );
  }
  
  @Override
  public Iterator<String> iterator(){
    return this;
  }
  @Override
  public boolean hasNext(){
    ....
  }
  @Override
  public String next(){
    ...
  }
} 
```

# Weitermachen Buch Orange Seite 1009 HashMap und TreeMap