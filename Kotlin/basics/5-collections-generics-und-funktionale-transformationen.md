# Collections, Generics und funktionale Transformationen

## Ziel

- Unterschied zwischen `List`, `Set` und `Map` sicher verstehen
- Mutable vs. Immutable Collections korrekt einsetzen
- Generics fachlich korrekt modellieren
- Collections idiomatisch mit funktionalen Operationen verarbeiten
- Seiteneffekte vermeiden und saubere Transformationen schreiben
- Typische Performance- und Designfehler vermeiden

---

# 1. Grundlagen: Collections in Kotlin

Kotlin unterscheidet zwischen:

- **Read-only Collections**
- **Mutable Collections**

Wichtig:  
Read-only bedeutet **nicht veränderbar über dieses Interface**, aber nicht zwingend wirklich immutable.

---

## 1.1 List

Geordnete Sammlung von Elementen.

```kotlin
val numbers: List<Int> = listOf(1, 2, 3)
```

Mutable Variante:

```kotlin
val mutableNumbers: MutableList<Int> = mutableListOf(1, 2, 3)
mutableNumbers.add(4)
```

Best Practice:

- Standardmäßig `List`
- `MutableList` nur wenn Mutation fachlich notwendig

## 1.2 Set

Keine doppelten Elemente.

```kotlin
val uniqueValues = setOf(1, 2, 2, 3)
// Ergebnis: [1, 2, 3]
```

Mutable:

```kotlin
val mutableSet = mutableSetOf(1, 2)
mutableSet.add(3)
```

Einsatz:

- Wenn Eindeutigkeit wichtig ist
- Membership-Checks (`contains`) performanter als List

## 1.3 Map

Schlüssel-Wert-Paare.

```kotlin
val ages = mapOf(
    "Max" to 21,
    "Anna" to 25
)
```

Zugriff:

```kotlin
val age = ages["Max"]
```

Mutable:

```kotlin
val mutableAges = mutableMapOf<String, Int>()
mutableAges["Max"] = 21
```

Best Practice:

- Nullable Zugriff berücksichtigen (`Map[key]` kann null liefern    
- `getValue()` nur wenn Schlüssel garantiert existiert

# 2. Iteration

## 2.1 for-Schleife

```kotlin
for (number in numbers) {
    println(number)
}
```

Mit Index:

```kotlin
for ((index, value) in numbers.withIndex()) {
    println("$index -> $value")
}
```

# 3. Funktionale Transformationen

Kotlin Collections bieten deklarative Operationen.

## 3.1 map

Transformation jedes Elements.

```kotlin
val doubled = numbers.map { it * 2 }
```

## 3.2 filter

Elemente nach Bedingung auswählen.

```kotlin
val even = numbers.filter { it % 2 == 0 }
```

## 3.3 filterNot

```kotlin
val odd = numbers.filterNot { it % 2 == 0 }
```

---

## 3.4 first / firstOrNull

```kotlin
val firstEven = numbers.first { it % 2 == 0 }
val safeFirstEven = numbers.firstOrNull { it % 2 == 0 }
```

Best Practice:

- `firstOrNull` verwenden, wenn Nicht-Vorhandensein möglich ist.

## 3.5 any / all / none

```kotlin
val hasEven = numbers.any { it % 2 == 0 }
val allPositive = numbers.all { it > 0 }
val noneNegative = numbers.none { it < 0 }
```

## 3.6 reduce

Reduziert Collection auf einen Wert.

```kotlin
val sum = numbers.reduce { acc, value -> acc + value }
```

Problem:

- Wirft Exception bei leerer Liste

## 3.7 fold

Sicherer als reduce.

```kotlin
val sum = numbers.fold(0) { acc, value -> acc + value }
```

Best Practice:

- `fold` bevorzugen, wenn Startwert bekannt ist.

## 3.8 sumOf

```kotlin
val total = numbers.sumOf { it }
```

Mit Objekten:

```kotlin
data class Product(val price: Double)

val products = listOf(Product(10.0), Product(20.0))
val totalPrice = products.sumOf { it.price }
```

# 4. Ketten von Transformationen

```kotlin
val result = numbers
    .filter { it > 2 }
    .map { it * 2 }
    .sortedDescending()
```

Prinzip:

- Transformationen bleiben rein
- Keine Mutation
- Klar lesbare Pipeline

# 5. Mutable vs. Immutable – Designentscheidungen

Schlecht:

```kotlin
val list = mutableListOf<Int>()
for (i in 1..10) {
    list.add(i * 2)
}
```

Besser:

```kotlin
val list = (1..10).map { it * 2 }
```

Vorteile:

- Keine Seiteneffekte
- Klarere Intention
- Weniger Fehlerquellen

# 6. Generics

Generics erlauben typsichere, wiederverwendbare Strukturen.

## 6.1 Einfache Generic-Klasse

```kotlin
class Box<T>(val value: T)
```

Verwendung:

```kotlin
val intBox = Box(10)
val stringBox = Box("Hallo")
```

## 6.2 Generic-Funktion

```kotlin
fun <T> printItem(item: T) {
    println(item)
}
```

---

## 6.3 Type Constraints

```kotlin
fun <T : Number> doubleValue(value: T): Double {
    return value.toDouble() * 2
}
```

# 7. Variance (Einführung)

Kotlin nutzt:

- `out` (Covariance)
- `in` (Contravariance)

Beispiel:

```kotlin
class Producer<out T>(private val value: T) {
    fun produce(): T = value
}
```

Grundidee:

- `out` → Typ wird nur produziert
- `in` → Typ wird nur konsumiert

Für den Einstieg:

- Generics ohne Variance korrekt einsetzen
- Variance erst bei API-Design bewusst verwenden

# 8. Performance-Aspekte

- `map`/`filter` erzeugen neue Collections
- Lange Ketten können Speicher erzeugen

Alternative:

- `asSequence()` bei großen Datenmengen
```kotlin
val result = numbers
    .asSequence()
    .filter { it > 2 }
    .map { it * 2 }
    .toList()
```
Sequences sind lazy.

---

# 9. Typische Fehler

- Unnötige `mutableListOf`
- `reduce` auf leeren Listen
- Seiteneffekte in `map`
- Zu komplexe Lambda-Ausdrücke
- Null-Safety ignorieren bei `Map`-Zugriffen

## Definition of Done

- Unterschied zwischen `List`, `Set`, `Map` klar
- Mutable nur bewusst eingesetzt
- Funktionale Transformationen idiomatisch
- Generics korrekt verstanden
- Keine unnötigen Seiteneffekte