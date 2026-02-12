# Funktionen, Parameter, Rückgabewerte und funktionale Grundlagen

## Ziel

- Funktionen strukturiert und wiederverwendbar definieren
- Parameter, Rückgabewerte und Typen korrekt modellieren
- Default-Parameter und Named Arguments sinnvoll einsetzen
- Single-Expression-Funktionen verstehen
- Grundlagen funktionaler Konzepte in Kotlin anwenden
- Lesbare und wartbare Funktionssignaturen entwerfen

---

## 1. Grundaufbau von Funktionen

Allgemeine Struktur:

```kotlin
fun funktionsName(parameter: Typ): RückgabeTyp {
    // Implementierung
    return wert
}
```

Beispiel:

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

Wichtige Aspekte:
- `fun` leitet eine Funktion ein.
- Parameter werden mit Typ deklariert.
- Der Rückgabetyp steht nach `:`.
- `return` beendet die Funktion mit einem Wert.

## 2. Funktionen ohne Rückgabewert

Wenn keine Rückgabe erfolgt, wird `Unit` verwendet:

```kotlin
fun printMessage(message: String): Unit {
    println(message)
}
```

`Unit` kann weggelassen werden:

```kotlin
fun printMessage(message: String) {
    println(message)
}
```

Best Practice:

- `Unit` nicht explizit angeben, außer aus didaktischen Gründen.

## 3. Single-Expression-Funktionen

Wenn eine Funktion nur aus einem Ausdruck besteht:

```kotlin
fun multiply(a: Int, b: Int): Int = a * b
```

Typinferenz möglich:

```kotlin
fun multiply(a: Int, b: Int) = a * b
```

Best Practice:

- Kurzform verwenden, wenn sie die Lesbarkeit verbessert.
- Bei komplexer Logik Blockform verwenden.

## 4. Parameter

### 4.1 Mehrere Parameter

```kotlin
fun createUser(name: String, age: Int, isAdmin: Boolean): String {
    return "Name: $name, Alter: $age, Admin: $isAdmin"
}
```

### 4.2 Default-Parameter

```kotlin
fun createUser(name: String, age: Int = 18): String {
    return "Name: $name, Alter: $age"
}
```

Aufruf:

```kotlin
createUser("Max")
createUser("Anna", 25)
```

Best Practice:

- Default-Parameter statt überladener Methoden verwenden.
- Sinnvolle Standardwerte definieren.

## 5. Named Arguments

Parameter können beim Aufruf benannt werden:

```kotlin
createUser(name = "Max", age = 30)
```

Reihenfolge kann geändert werden:

```kotlin
createUser(age = 30, name = "Max")
```

Vorteile:

- Erhöhte Lesbarkeit
- Vermeidung von Parameterverwechslungen
- Klarheit bei vielen Parametern

Best Practice:

- Named Arguments bei 3+ Parametern oder bei gleichen Typen verwenden.

## 6. Rückgabetypen korrekt modellieren

Schlecht:

```kotlin
fun calculateDiscount(price: Double): Double?
```

Besser:

- Klare fachliche Entscheidung:
    - Gibt es immer einen Rabattwert? Dann `Double`
    - Kann es keinen geben? Dann `Double?` oder besser:
        - Alternativmodellierung (z. B. Result-Typ, später)

Grundsatz:

- Nullable nur verwenden, wenn `null` fachlich sinnvoll ist.

## 7. Funktionen als First-Class Citizens

In Kotlin sind Funktionen Werte.

Beispiel:

```kotlin
val square: (Int) -> Int = { x -> x * x }
```

Verwendung:

```
println(square(4))
```

## 8. Lambda-Ausdrücke

Allgemeine Form:

```kotlin
{ parameter -> ausdruck }
```

Beispiel:

```kotlin
val greet: (String) -> String = { name -> "Hallo $name" }
```

Typ kann oft weggelassen werden:

```kotlin
val greet = { name: String -> "Hallo $name" }
```

## 9. Higher-Order Functions

Funktionen, die andere Funktionen als Parameter erhalten oder zurückgeben.

Beispiel:

```kotlin
fun operate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}
```

Verwendung:

```kotlin
val result = operate(4, 5) { x, y -> x + y }
println(result)
```

---

## 10. Standardbibliothek nutzen

Beispiel mit `List`:

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)

val doubled = numbers.map { it * 2 }
val even = numbers.filter { it % 2 == 0 }
val sum = numbers.reduce { acc, value -> acc + value }
```

Wichtige Funktionen:

- `map`
- `filter`
- `forEach`
- `reduce`
- `fold`
- `any`
- `all`
- `none`

Best Practice:

- Iterative Logik mit `map`/`filter` etc. ausdrücken.
- Keine unnötigen `for`-Schleifen bei Collection-Transformationen.

## 11. Lokale Funktionen

Funktionen können innerhalb anderer Funktionen definiert werden:

```kotlin
fun process(value: Int): Int {
    fun validate(input: Int): Boolean = input >= 0
    
    if (!validate(value)) {
        throw IllegalArgumentException("Ungültiger Wert")
    }
    
    return value * 2
}
```

Einsatz:

- Logik kapseln
- Wiederverwendung innerhalb eines Kontexts

## 12. Erweiterungsfunktionen (Extension Functions)

Ermöglichen das Erweitern bestehender Typen.

```kotlin
fun String.isLong(): Boolean {
    return this.length > 10
}
```

Verwendung:

```kotlin
val text = "Hallo Welt"
println(text.isLong())
```

Best Practice:

- Erweiterungsfunktionen nur verwenden, wenn sie fachlich zum Typ passen.
- Keine versteckte Business-Logik in Extensions.

## 13. Saubere Funktionsgestaltung

### 13.1 Single Responsibility

Jede Funktion sollte:

- eine klar definierte Aufgabe haben
- leicht testbar sein
- keinen unnötigen Zustand verändern

### 13.2 Keine versteckten Seiteneffekte

Schlecht:

```kotlin
var counter = 0

fun increment() {
    counter++
}
```

Besser:

```kotlin
fun increment(counter: Int): Int = counter + 1
```

Bevorzugt:

- Reine Funktionen (gleicher Input → gleicher Output)
- Keine Abhängigkeit von globalem Zustand

## 14. Typische Fehler

- Zu viele Parameter
- Verwendung von `var` statt Rückgabewerten
- Nullable-Typen ohne fachliche Begründung
- Unklare Funktionsnamen

## Definition of Done

- Funktionen sind klar benannt und strukturiert
- Default-Parameter werden korrekt eingesetzt
- Higher-Order Functions werden verstanden
- Keine unnötigen globalen Zustände