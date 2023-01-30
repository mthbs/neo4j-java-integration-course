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

# Maps

- Die wichtigsten Funktionen:

|                                                                 Map <K,V> |
|--------------------------------------------------------------------------:|
|                                                               size(): int |
|                                                        isEmpty(): boolean |
|                                              containsKey(Object): boolean |
|                                            containsValue(Object): boolean |
|                                                            get(Object): V |
|                                                              put(K, V): V |
|                                                         remove(Object): V |
|                               putAll(Map<? extends K, ? extends V>): void |
|                                                             clear(): void |
|                                                          keySet(): Set<K> |
|                                                   values(): Collection<V> |
|                                              entrySet(): Set<Entry<K, V>> |
|                                                   equals(Object): boolean |
|                                                           hashCode(): int |
|                                               getOrDefault (Object, V): V |
|                            forEach(BiConsumer<? super K, ? super V>: void |
|           replaceAll(BiFunction<? super K, ß super V, ß extends V>): void |
|                                                      putIfAbsent(K, V): V |
|                                           remove(Object, Object): boolean |
|                                                 replace(K, V, V): boolean |
|                                                          replace(K, V): V |
|                   computeIfAbsent(K, Function<? super K, ? extends V>): V |
|     computeIfPresent(K, BiFunction<? super K, ? super V, ? extends V>): v |
|               compute(K,BiFunction<? super K, ? super V, ? extends V>): V |
|               merge(K,V,BiFunction<? super K, ? super V, ? extends V>): V |
|                                                           of(): Map<K, V> |
|                                                       of(K, V): Map<K, V> |
|                                                 of(K, V, K, V): Map<K, V> |
|                                                                       ... |
| of(K, V, K, V, K, V, K, V, K, V, K, V, K, V, K, V, K, V, K, V): Map<K, V> |
|                    ofEntries(Entry[]<? extends K, ? extends V): Map<K, V> |
|                                                  entry(K, V): Entry<K, V> |

| Entry                                                           |
|-----------------------------------------------------------------|
| getKey(): K                                                     |
| getValue(): V                                                   |
| setValue(V): V                                                  |
| equals(Object): boolean                                         |
| hashCode(): int                                                 |
| comparingByKey(): Comparator<Entry<K, V>>                       |
| comparingByValue(): Comparator<Entry<K, V>>                     |
| comparingByKey(Comparator<? super K>): Comparator<Entry<K, V>>  |
| comparingByValue(Comparator<? super V>): Comparator<Entry<K,V>> |




- gehören zur Gruppe der Assoziativen Speichers (Key-Value-Paare)
- put(key,value)
- get(key)

## HashMap

- EIne schnelle Implementierung der Hash-Tabelle
  - Dabei müssen Schlüsselobjekte "hashbar" sein und damit equals() und hashCode() implementieren.
- Ideal um Werte schnell nach dem Schlüssel zu suchen, die Schlüssel werden nicht sortiert
- In allen HashMaps wird davor gewarnt Werte nachträglich zu verändern.

## TreeMap

- ETwas langsamer im Zugriff als HashMap
- Hält immer alle Schlüsselobjekte sortiert
  - Die Elemente werden in einen internen Binärbaum sortiert, dabei müssen sich die Schlüssel in eine Ordnung bringen lassen
- Eine Ordnung zwischen den Elementen muss her, entweder mit Comparable oder Comparator

## LinkedHashMap

- HashMap mit gleichzeitiger Speicherung der Reihenfolge

## Ansichten der Maps

- Eine Map kann auf drei Arten Sammlungen zurückgeben, über die sich iterieren lässt:

|                                                                                                             Map-Ansichten | 
|--------------------------------------------------------------------------------------------------------------------------:|
|                                                                                 keySet() liefert eine Menge der Schlüssel |
|                                                                                values() liefert eine Collection der Werte |
| entrySet() liefert eine Menge mit Map.Entry-Objekten. Die Map.Entry-Objekte speichern gleichzeitig den Key und den Value. |

## Map.Entry-Objekte

- Sinn: Momentaufnahme
- ENtry ist eine innere Schnittstelle von Map
  - deklariert eine API für den Zugriff auf Key-Value-Paare
  - Unbedingte Methoden: getKey(), getValue()
  - Optionale Methode: setValue()
- bieten Zugriff auf die Schlüssel und Werte

# Die Arbeitsweise einer Hash-Tabelle

- Nach dem Schlüssel wird nach der mathematischen Hash-Funktion ein Hashwert (auch Hashcode genannt) berechnet, dieser dient als Index für ein internes Array.
- Dieses Array hat am Anfang eine feste Größe
- Wenn eine Anfrage gestellt wird, muss einfach diese Berechnung erfolgen und wir können an dieser Stelle nachsehen.













