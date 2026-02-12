# SOLID

**Die fünf Grundprinzipien sauberer Objektorientierung –> wartbar, testbar, erweiterbar**

---

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:
- die fünf SOLID-Prinzipien erklären
- erkennen, wann Code gegen SOLID verstößt
- Klassen sauber strukturieren
- Kopplung reduzieren
- bessere Architekturen für Android und Kotlin entwickeln

# Überblick: Was bedeutet SOLID?

|Buchstabe|Prinzip|
|---|---|
|S|Single Responsibility Principle|
|O|Open/Closed Principle|
|L|Liskov Substitution Principle|
|I|Interface Segregation Principle|
|D|Dependency Inversion Principle|

Diese fünf Prinzipien helfen dir, **robuste, wartbare Systeme** zu bauen.

# S — Single Responsibility Principle (SRP)

> Eine Klasse sollte genau **einen Grund zur Änderung** haben.

## Falsch
```kotlin
class UserManager {

    fun saveUser(user: User) {
        // Speichern in DB
    }

    fun sendEmail(user: User) {
        // E-Mail versenden
    }

    fun logAction(message: String) {
        // Logging
    }
}
```

Problem:
- Datenbank
- E-Mail
- Logging

Drei Verantwortlichkeiten → drei Änderungsgründe.

## Richtig
```kotlin
class UserRepository {
    fun saveUser(user: User) {}
}

class EmailService {
    fun sendEmail(user: User) {}
}

class Logger {
    fun log(message: String) {}
}
```

Jetzt:
- jede Klasse hat eine Aufgabe
- Änderungen sind isoliert
- besser testbar
# O — Open/Closed Principle (OCP)

> Klassen sollten **offen für Erweiterung**, aber **geschlossen für Modifikation** sein.

## Falsch
```kotlin
class DiscountCalculator {

    fun calculate(type: String, price: Double): Double {
        return when (type) {
            "student" -> price * 0.8
            "vip" -> price * 0.7
            else -> price
        }
    }
}
```

Problem:
- Jede neue Rabattart → Klasse ändern

## Richtig
```kotlin
interface DiscountStrategy {
    fun calculate(price: Double): Double
}

class StudentDiscount : DiscountStrategy {
    override fun calculate(price: Double) = price * 0.8
}

class VipDiscount : DiscountStrategy {
    override fun calculate(price: Double) = price * 0.7
}
```

Jetzt:
- neue Strategie = neue Klasse
- bestehender Code bleibt unverändert

# L — Liskov Substitution Principle (LSP)

> Unterklassen müssen sich wie ihre Basisklasse verhalten können.

## Falsch
```kotlin
open class Bird {
    open fun fly() {}
}

class Penguin : Bird() {
    override fun fly() {
        throw UnsupportedOperationException()
    }
}
```

Problem:
- Penguin ist kein „fliegender Vogel“
- verletzt LSP

## Richtig
```kotlin
interface Bird

interface FlyingBird : Bird {
    fun fly()
}

class Eagle : FlyingBird {
    override fun fly() {}
}

class Penguin : Bird
```

Jetzt:
- keine falschen Erwartungen
- saubere Hierarchie

# I — Interface Segregation Principle (ISP)

> Keine Klasse sollte Methoden implementieren müssen, die sie nicht braucht.

## Falsch
```
interface Worker {
    fun work()
    fun eat()
}

class Robot : Worker {
    override fun work() {}
    override fun eat() {
        throw UnsupportedOperationException()
    }
}
```

## ✅ Richtig
```kotlin
interface Workable {
    fun work()
}

interface Eatable {
    fun eat()
}

class Human : Workable, Eatable {
    override fun work() {}
    override fun eat() {}
}

class Robot : Workable {
    override fun work() {}
}
```

# D — Dependency Inversion Principle (DIP)

> High-Level-Module sollten nicht von Low-Level-Modulen abhängen.  
> Beide sollten von Abstraktionen abhängen.

## Falsch
```kotlin
class MySqlDatabase {
    fun save(data: String) {}
}

class UserService {
    private val database = MySqlDatabase()
}
```

UserService ist fest an MySQL gebunden.

## Richtig
```kotlin
interface Database {
    fun save(data: String)
}

class MySqlDatabase : Database {
    override fun save(data: String) {}
}

class UserService(
    private val database: Database
)
```

Jetzt:
- austauschbar
- testbar
- flexibel

# SOLID im Android-Kontext

### SRP
- ViewModel hält State
- Repository macht Datenzugriff
- Fragment macht UI

### OCP
- neue Feature-Strategie → neue Klasse
- kein riesiger `when`-Block

### LSP
- keine falschen Vererbungen
- keine „UnsupportedOperationException“

### ISP
- kleine, klare Interfaces
- keine Monster-Interfaces

### DIP
- ViewModel kennt Repository-Interface
- nicht konkrete Implementierung

# Typische Anti-Patterns

- God-Class  
- Utility-Monster  
- riesige `when`-Blöcke  
- direkte Implementierungsabhängigkeit  
- Copy-Paste-Logik

# Warum SOLID wichtig ist

SOLID führt zu:
- geringerer Kopplung
- höherer Testbarkeit
- besserer Erweiterbarkeit
- weniger Seiteneffekten
- stabilerer Architektur

# Selbst-Check

-  Jede Klasse hat eine klare Verantwortung
-  Ich erweitere Code ohne ihn zu verändern
-  Unterklassen verhalten sich korrekt
-  Interfaces sind klein und spezifisch
-  High-Level-Code hängt von Abstraktionen ab

# Weiterführende Referenzen

- Clean Architecture – Robert C. Martin
- [Kotlin Design Guidelines](https://kotlinlang.org/docs/coding-conventions.html)
- [SOLID Prinzipien (Erklärung allgemein)](https://refactoring.guru/design-patterns)