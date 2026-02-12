# Vererbung, abstrakte Klassen, Interfaces und Polymorphie

## Ziel

- Vererbung korrekt einsetzen und begrenzen
- Unterschied zwischen Klassenvererbung und Interfaces verstehen
- Abstrakte Klassen sinnvoll verwenden
- Polymorphie fachlich korrekt modellieren
- Überschreiben (`override`) sauber anwenden
- Typische Designfehler in OOP vermeiden

---

## 1. Grundprinzip: Vererbung in Kotlin

In Kotlin sind Klassen standardmäßig **final**.  
Das bedeutet: Sie können nicht vererbt werden, außer sie werden explizit als `open` markiert.

```kotlin
open class Animal(val name: String)
```

Vererbung:

```kotlin
class Dog(name: String) : Animal(name)
```

Wichtig:

- `:` leitet Vererbung ein.
- Konstruktorparameter der Basisklasse müssen übergeben werden.

## 2. Methoden überschreiben (`override`)

Methoden müssen explizit als `open` markiert sein, um überschrieben zu werden.

```kotlin
open class Animal(val name: String) {

    open fun makeSound(): String {
        return "Unbekanntes Geräusch"
    }
}

class Dog(name: String) : Animal(name) {

    override fun makeSound(): String {
        return "Wuff"
    }
}
```

Best Practice:

- Überschreibbare Methoden bewusst kennzeichnen.
- Keine unkontrollierte Offenheit von Klassen.

## 3. `final` gezielt einsetzen

Auch in `open`-Klassen können Methoden wieder final gemacht werden:

```kotlin
open class Animal {

    open fun move() { }

    final fun breathe() { }
}
```

## 4. Abstrakte Klassen

Abstrakte Klassen können nicht instanziiert werden.

```kotlin
abstract class Shape {

    abstract fun area(): Double

    fun describe(): String {
        return "Dies ist eine geometrische Form."
    }
}
```

Implementierung:

```kotlin
class Rectangle(
    val width: Double,
    val height: Double
) : Shape() {

    override fun area(): Double {
        return width * height
    }
}
```

Eigenschaften:

- Können konkrete und abstrakte Methoden enthalten
- Dienen als gemeinsame Basis für Spezialisierungen

## 5. Interfaces

Interfaces definieren Verträge.

```kotlin
interface Drivable {
    fun drive()
}
```

Implementierung:

```kotlin
class Car : Drivable {

    override fun drive() {
        println("Auto fährt")
    }
}
```

Mehrfachimplementierung:

```kotlin
interface Electric {
    fun charge()
}

class ElectricCar : Drivable, Electric {

    override fun drive() { }

    override fun charge() { }
}
```

---

## 6. Interface mit Default-Implementierung

```kotlin
interface Logger {

    fun log(message: String) {
        println("LOG: $message")
    }
}
```

Implementierung optional überschreiben:

```kotlin
class FileLogger : Logger
```

---

## 7. Abstrakte Klasse vs. Interface

|Abstrakte Klasse|Interface|
|---|---|
|Kann Zustand besitzen|Kein Zustand (keine Konstruktor-Parameter)|
|Einzelvererbung|Mehrfachimplementierung|
|Gemeinsame Basisstruktur|Vertrag/Capability|

Best Practice:

- Interface für Verhalten
- Abstrakte Klasse für gemeinsame Implementierungsbasis
- Vererbung nur bei echter "ist-ein"-Beziehung
 
## 8. Polymorphie

Polymorphie bedeutet:  
Objekte unterschiedlicher Klassen können über einen gemeinsamen Typ behandelt werden.

Beispiel:

```kotlin
open class Animal {
    open fun makeSound(): String = "?"
}

class Dog : Animal() {
    override fun makeSound() = "Wuff"
}

class Cat : Animal() {
    override fun makeSound() = "Miau"
}
```

Polymorphe Nutzung:

```kotlin
val animals: List<Animal> = listOf(Dog(), Cat())

for (animal in animals) {
    println(animal.makeSound())
}
```

Ergebnis:

- Zur Laufzeit wird die korrekte Implementierung aufgerufen.

## 9. Liskov Substitution Principle (LSP)

Grundsatz:

> Eine Unterklasse muss sich überall dort korrekt verhalten, wo die Basisklasse erwartet wird.

Schlecht:

```kotlin
open class Bird {
    open fun fly() { }
}

class Penguin : Bird() {
    override fun fly() {
        throw UnsupportedOperationException()
    }
}
```

Problem:

- Penguin ist kein gültiger Ersatz für Bird.

Besser:

```kotlin
interface Flyable {
    fun fly()
}

abstract class Bird

class Eagle : Bird(), Flyable {
    override fun fly() { }
}

class Penguin : Bird()
```

---

## 10. `is` und Smart Cast

Typprüfung:

```kotlin
if (animal is Dog) {
    println(animal.makeSound())
}
```

Kotlin führt Smart Cast durch.

Vermeiden:

- Komplexe `when`-Typprüfungen bei schlechter Modellierung
- Besser: Polymorphie korrekt nutzen

## 11. Sealed Classes (Einführung)

Sealed Classes begrenzen erlaubte Unterklassen.

```kotlin
sealed class Result

class Success(val data: String) : Result()
class Error(val message: String) : Result()
```

`when` ist exhaustiv:

```kotlin
fun handle(result: Result) = when (result) {
    is Success -> println(result.data)
    is Error -> println(result.message)
}
```

Kein `else` nötig.

## 12. Typische Fehler

- Vererbung statt Komposition
- Basisklassen mit zu viel Verantwortung
- Verletzung von LSP
- Zu tiefe Vererbungshierarchien
- Interface-Missbrauch ohne klare Semantik

## 13. Komposition statt Vererbung

Statt:

```kotlin
class ElectricCar : Car()
```

Besser:

```kotlin
class ElectricEngine {
    fun start() { }
}

class Car(
    private val engine: ElectricEngine
)

```

Prinzip:

> Bevorzuge Komposition gegenüber Vererbung.

## Definition of Done

- Vererbung bewusst und begrenzt eingesetzt
- Interfaces korrekt genutzt
- LSP verstanden und berücksichtigt
- Polymorphie ohne `if`-Typprüfung umgesetzt
- Sealed Classes korrekt eingesetzt