# Klassen, Objekte, Konstruktoren und objektorientierte Modellierung

## Ziel

- Klassen und Objekte korrekt definieren und verwenden
- Konstruktoren und Initialisierungsblöcke verstehen
- Eigenschaften (Properties) sauber modellieren
- Kapselung und Sichtbarkeiten korrekt einsetzen
- Fachliche Modelle in saubere Objektstrukturen überführen
- Erste Grundlage für SOLID-konforme Architektur legen

---

## 1. Grundkonzept: Klasse und Objekt

Eine **Klasse** ist ein Bauplan.  
Ein **Objekt** ist eine konkrete Instanz dieser Klasse.

Beispiel:

```kotlin
class User {
    val name: String = "Max"
}
```

Instanz erzeugen:

```kotlin
val user = User()
println(user.name)
```

## 2. Eigenschaften (Properties)

### 2.1 Konstruktor mit Properties

Typische Kotlin-Form:

```kotlin
class User(
    val name: String,
    var age: Int
)
```

Erzeugen:

```kotlin
val user = User("Max", 21)
```

Wichtig:

- `val` → unveränderlich
- `var` → veränderlich

Best Practice:

- Standardmäßig `val` verwenden.
- Mutabilität bewusst und minimal einsetzen.

## 3. Primärkonstruktor

Der Primärkonstruktor steht direkt hinter dem Klassennamen:

```kotlin
class Product(val name: String, val price: Double)
```

Keine explizite `constructor`-Schreibweise nötig.

## 4. Initialisierungsblock (`init`)

Wird bei Instanziierung ausgeführt:

```kotlin
class User(val name: String, val age: Int) {

    init {
        require(age >= 0) { "Alter darf nicht negativ sein" }
    }
}
```

`require`:

- Wirft `IllegalArgumentException`, wenn Bedingung falsch ist.

Best Practice:

- Validierung direkt im Konstruktor oder `init`.
- Objekte niemals in ungültigem Zustand erzeugen.

## 5. Sekundärkonstruktoren

Werden mit `constructor` definiert:

```kotlin
class User(val name: String, val age: Int) {

    constructor(name: String) : this(name, 18)
}
```

Best Practice:

- Möglichst Default-Parameter statt Sekundärkonstruktor.
- Sekundärkonstruktor nur bei echter Notwendigkeit.

## 6. Methoden in Klassen

```kotlin
class BankAccount(
    val owner: String,
    private var balance: Double
) {

    fun deposit(amount: Double) {
        require(amount > 0)
        balance += amount
    }

    fun withdraw(amount: Double) {
        require(amount > 0)
        require(amount <= balance)
        balance -= amount
    }

    fun getBalance(): Double {
        return balance
    }
}
```

Wichtige Aspekte:

- `balance` ist `private`
- Zugriff nur über Methoden

Prinzip: **Kapselung**

## 7. Sichtbarkeiten (Visibility Modifiers)

- `public` (Standard)
- `private`
- `protected`
- `internal`

Beispiel:

```kotlin
class Example {

    private val secret = "intern"

    internal fun internalLogic() { }

    public fun exposed() { }
}
```

Best Practice:

- Standardmäßig `private` oder `internal`
- Öffentlich nur, was Teil der API ist

## 8. Custom Getter und Setter

```kotlin
class Rectangle(val width: Double, val height: Double) {

    val area: Double
        get() = width * height
}
```

Mit Setter:

```kotlin
var temperature: Double = 0.0
    set(value) {
        field = value.coerceIn(-100.0, 100.0)
    }
```

`field`:

- Referenziert das tatsächliche Speicherfeld.

## 9. Data Classes

Für reine Datenmodelle:

```kotlin
data class User(
    val id: Int,
    val name: String
)
```

Automatisch generiert:

- `toString()`
- `equals()`
- `hashCode()`
- `copy()`
- `componentN()`

Beispiel:

```kotlin
val user1 = User(1, "Max")
val user2 = user1.copy(name = "Anna")
```

Best Practice:

- Data Classes für DTOs und Value Objects.
- Keine Business-Logik in Data Classes.

## 10. Objekt-Modellierung – Von Fachlichkeit zu Code

Beispiel: Bestellsystem

Fachlich:

- Kunde
- Produkt
- Bestellung
- Bestellposition

Saubere Modellierung:

```kotlin
data class Product(
    val id: String,
    val name: String,
    val price: Double
)

data class OrderItem(
    val product: Product,
    val quantity: Int
) {
    init {
        require(quantity > 0)
    }

    fun totalPrice(): Double = product.price * quantity
}

class Order(
    val id: String,
    val items: List<OrderItem>
) {
    fun totalAmount(): Double =
        items.sumOf { it.totalPrice() }
}
```

Wichtige Prinzipien:

- Jede Klasse hat klare Verantwortung
- Validierung im Konstruktor
- Berechnungslogik dort, wo sie fachlich hingehört

## 11. Immutable vs. Mutable Modelle

Immutable (bevorzugt):

```kotlin
data class User(val name: String)
```

Mutable:

```kotlin
class User(var name: String)
```

Best Practice:

- Immutable Standard
- Mutable nur bei echten Zustandsänderungen
- In Domain-Modellen möglichst unveränderlich denken

## 12. Typische Fehler

- Alles als `var`
- Keine Validierung im Konstruktor
- Business-Logik im `main`
- Verletzung der Kapselung
- God-Class (zu viele Verantwortlichkeiten)

## Definition of Done

- Klassen sind klar strukturiert
- Konstruktorvalidierung wird verwendet
- Kapselung korrekt umgesetzt
- Data Classes sinnvoll eingesetzt
- Keine unnötige Mutabilität