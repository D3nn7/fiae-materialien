# Datentypen, Operatoren, Kontrollstrukturen und Null-Safety

## Ziel

- Grundlegende Datentypen sicher verwenden
- Unterschied zwischen `val` und `var` verstehen und korrekt einsetzen
- Kontrollstrukturen idiomatisch in Kotlin anwenden
- Null-Safety-Konzept von Kotlin verstehen und praktisch nutzen
- Typische Fehlerquellen vermeiden

---

## 1. Variablen und Konstanten

### `val` vs. `var`

- `val`: unveränderliche Referenz (bevorzugt)
- `var`: veränderliche Referenz (nur wenn notwendig)

```kotlin
val name = "Max"
var counter = 0

counter += 1
```

Best Practice:

- Standardmäßig `val` verwenden.
- `var` nur einsetzen, wenn eine tatsächliche Zustandsänderung fachlich erforderlich ist.

## 2. Grundlegende Datentypen

Kotlin ist statisch typisiert. Der Typ ist zur Compile-Zeit bekannt.

### Ganzzahlen

```kotlin
val a: Int = 10
val b: Long = 100L
val c: Short = 5
val d: Byte = 1
```

Standardtyp für ganze Zahlen ist `Int`.

### Gleitkommazahlen

```kotlin
val pi: Double = 3.14159
val f: Float = 2.5f
```

Standardtyp ist `Double`.

### Boolean

```kotlin
val isActive: Boolean = true
```

### Char

```kotlin
val letter: Char = 'A'
```

### String

```kotlin
val text: String = "Hallo Welt"
```

String-Interpolation:

```kotlin
val age = 21
println("Ich bin $age Jahre alt.")
println("Nächstes Jahr: ${age + 1}")
```

## 3. Typinferenz

Kotlin kann Typen oft automatisch ableiten:

```kotlin
val number = 42        // Int
val price = 19.99      // Double
val username = "alex"  // String
```

Best Practice:

- Typinferenz nutzen, wenn der Typ eindeutig ist.
- Explizite Typangabe bei:
    - öffentlichen APIs
    - komplexeren Rückgabetypen
    - didaktischen Beispielen

## 4. Operatoren

### Arithmetische Operatoren

```kotlin
val sum = 5 + 3
val diff = 10 - 4
val product = 2 * 6
val quotient = 8 / 2
val remainder = 7 % 3
```

### Vergleichsoperatoren

```kotlin
val result = 5 > 3
val equal = 10 == 10
val notEqual = 10 != 5
```

Wichtig:

- `==` prüft strukturelle Gleichheit (Inhalt).
- `===` prüft Referenzgleichheit.

Beispiel:

```kotlin
val a = "Test"
val b = "Test"

println(a == b)   // true
println(a === b)  // kann true oder false sein (Implementierungsdetail)
```

Im Normalfall wird `==` verwendet.

### Logische Operatoren

```kotlin
val isValid = true && false
val isEither = true || false
val isNot = !true
```

## 5. Kontrollstrukturen

### 5.1 if-Ausdruck

In Kotlin ist `if` ein Ausdruck und liefert einen Wert.

```kotlin
val number = 10

val result = if (number > 0) {
    "positiv"
} else {
    "nicht positiv"
}
```

Kurzform:

```kotlin
val result = if (number > 0) "positiv" else "nicht positiv"
```

Best Practice:

- `if` als Ausdruck nutzen, wenn ein Wert bestimmt wird.
- Keine unnötigen temporären Variablen erzeugen.

### 5.2 when-Ausdruck

`when` ersetzt komplexe `switch`-Strukturen.

```kotlin
val grade = 2

val text = when (grade) {
    1 -> "sehr gut"
    2 -> "gut"
    3 -> "befriedigend"
    else -> "andere Note"
}
```

Mehrere Werte:

```kotlin
when (grade) {
    1, 2 -> println("gut oder besser")
    else -> println("nicht gut genug")
}
```

Mit Bedingungen:

```kotlin
val number = 15

when {
    number < 0 -> println("negativ")
    number == 0 -> println("null")
    else -> println("positiv")
}
```

Best Practice:

- `when` bevorzugen bei mehreren Verzweigungen.
- Exhaustive `when` bei Enums und sealed classes später konsequent einsetzen.

### 5.3 Schleifen

#### for-Schleife

```kotlin
for (i in 1..5) {
    println(i)
}
```

Mit Schrittweite:

```kotlin
for (i in 1..10 step 2) {
    println(i)
}
```

Abwärts:

```kotlin
for (i in 10 downTo 1) {
    println(i)
}
```

#### while-Schleife

```kotlin
var counter = 0

while (counter < 5) {
    println(counter)
    counter++
}
```

Best Practice:

- `for` mit Ranges oder Collections bevorzugen.
- `while` nur verwenden, wenn Abbruchbedingung dynamisch ist.

## 6. Null-Safety – zentrales Kotlin-Konzept

Kotlin unterscheidet strikt zwischen:

- Nicht-nullbaren Typen: `String`
- Nullable Typen: `String?`

### Nicht-nullbar (Standard)

```kotlin
val name: String = "Max"
```

Zuweisung von `null` ist nicht erlaubt:

```kotlin
val name: String = null // Compiler-Fehler
```

### Nullable Typ

```kotlin
val name: String? = null
```

### 6.1 Sicherer Zugriff mit `?.`

```kotlin
val name: String? = "Max"

println(name?.length)
```

Wenn `name == null`, wird das Ergebnis `null`.

### 6.2 Elvis-Operator `?:`

Fallback-Wert definieren:

```kotlin
val name: String? = null

val length = name?.length ?: 0
```

Bedeutung:

- Wenn links `null`, dann rechts verwenden.

### 6.3 Not-Null-Assertion `!!`

```kotlin
val name: String? = "Max"
val length = name!!.length
```

Risiko:

- Wenn `name == null`, wird eine `NullPointerException` geworfen.

Best Practice:

- `!!` vermeiden.
- Nullable sauber modellieren und mit `?.` oder `?:` arbeiten.

### 6.4 Smart Casts

```kotlin
val input: String? = "Hallo"

if (input != null) {
    println(input.length) // automatisch als String behandelt
}
```

Der Compiler erkennt, dass `input` innerhalb des Blocks nicht null sein kann.

## 7. Saubere Modellierung mit Null-Safety

Schlecht:

```kotlin
fun getUserName(): String? {
    return null
}
```

Besser:

- Klare fachliche Entscheidung:
    - Kann kein Name existieren? Dann `String?`
    - Muss immer ein Name existieren? Dann `String`

Nullable nur dann verwenden, wenn `null` fachlich legitim ist.

## 8. Typische Fehlerquellen

- Unkontrollierte Nutzung von `!!`
- Übermäßiger Einsatz von `var`
- `if` als Statement statt als Ausdruck
- Fehlende Default-Zweige in `when`

## Definition of Done

- Datentypen können korrekt eingesetzt werden
- `if` und `when` werden als Ausdrücke verstanden
- Null-Safety wird ohne `!!` gelöst
- Übungen sind vollständig implementiert und getestet
- Kein unnötiger Einsatz von `var`