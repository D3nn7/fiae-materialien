# Factory Pattern

**Saubere Objekterzeugung, Entkopplung & testbare Architektur**

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:
- erklären, was das Factory Pattern ist
- erkennen, wann eine Factory sinnvoll ist
- saubere Factory-Implementierungen in Kotlin schreiben
- Kopplung reduzieren und Testbarkeit erhöhen
- Factory Pattern korrekt in Android-Architekturen einsetzen

## 1. Warum ein Factory Pattern?

### Problem

Direkte Objekterzeugung erzeugt Kopplung:

```kotlin
val repository = UserRepository()
```

Wenn sich:
- Konstruktor-Parameter ändern
- Implementierungen wechseln
- Tests andere Implementierungen brauchen

→ musst du überall Code anpassen.

### Ziel des Factory Patterns

> Die Erzeugung eines Objekts wird gekapselt.

Vorteile:
- bessere Wartbarkeit
- geringere Kopplung
- bessere Testbarkeit
- zentrale Kontrolle der Erstellung

---

## 2. Grundidee des Factory Patterns

Statt:
```kotlin
val user = User()
```

Verwendest du:
```kotlin
val user = UserFactory.create()
```

Die Factory entscheidet:

- welche Implementierung
- welche Parameter
- welche Konfiguration

## 3. Einfaches Beispiel

### Ohne Factory
```kotlin
class Logger

class UserService {
    private val logger = Logger()
}
```

Problem:
- `UserService` ist fest an `Logger` gebunden

### Mit Factory
```kotlin
class Logger

object LoggerFactory {
    fun create(): Logger {
        return Logger()
    }
}

class UserService(
    private val logger: Logger
)
```

Verwendung:
```kotlin
val service = UserService(LoggerFactory.create())
```

Jetzt:
- Logger kann ausgetauscht werden
- Tests können Fake-Logger nutzen

## 4. Factory mit Interface

### Interface definieren
```kotlin
interface PaymentProcessor {
    fun process(amount: Double)
}
```

### Implementierungen
```kotlin
class PaypalProcessor : PaymentProcessor {
    override fun process(amount: Double) {
        println("Processing via PayPal")
    }
}

class CreditCardProcessor : PaymentProcessor {
    override fun process(amount: Double) {
        println("Processing via Credit Card")
    }
}
```

### Factory implementieren
```kotlin
object PaymentProcessorFactory {

    fun create(type: String): PaymentProcessor {
        return when (type) {
            "paypal" -> PaypalProcessor()
            "creditcard" -> CreditCardProcessor()
            else -> throw IllegalArgumentException("Unknown type")
        }
    }
}
```

Verwendung:
```kotlin
val processor = PaymentProcessorFactory.create("paypal")
processor.process(100.0)
```

## 5. Factory Method Pattern

Beim Factory Method Pattern entscheidet eine Basisklasse, welche konkrete Implementierung erzeugt wird.

```kotlin
abstract class Creator {
    abstract fun createProduct(): Product
}

class ConcreteCreator : Creator() {
    override fun createProduct(): Product {
        return ConcreteProduct()
    }
}
```

Dieses Pattern ist nützlich bei:
- Framework-Architekturen
- erweiterbaren Systemen

## 6. Factory in Android (praktisch relevant)

Typische Android-Anwendungsfälle:
- ViewModel Factory
- Repository-Erzeugung
- Service-Erzeugung
- Parser-Strategien

Beispiel: ViewModel Factory
```kotlin
class UserViewModelFactory(
    private val repository: UserRepository
) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(UserViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return UserViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel")
    }
}
```

## 7. Vorteile des Factory Patterns

- Single Responsibility (Erzeugung getrennt von Nutzung)
- Open/Closed Principle
- bessere Testbarkeit
- geringere Abhängigkeiten

## 8. Wann KEINE Factory?

- Wenn das Objekt trivial ist  
- Wenn keine Konfigurationslogik existiert  
- Wenn keine Austauschbarkeit nötig ist

> Nicht jedes `new` braucht eine Factory.

## 9. Typische Fehler

- Factory mit zu viel Logik  
- Factory kennt UI  
- Globale Factory mit versteckten Abhängigkeiten  
- `when`-Blöcke, die ständig erweitert werden müssen

## 10. Thread-Safety beachten

Wenn eine Factory:
- Caches nutzt
- Singleton-Instanzen zurückgibt
→ Synchronisation beachten.

Beispiel:
```kotlin
@Volatile
private var instance: Repository? = null
```

## 11. Testbarkeit verbessern

In Tests:
```kotlin
val fakeProcessor = object : PaymentProcessor {
    override fun process(amount: Double) {
        // Testverhalten
    }
}
```

Factory wird im Test ersetzt oder umgangen.

## 12. Selbst-Check

-  Ich verstehe den Zweck des Factory Patterns
-  Ich weiß, wann eine Factory sinnvoll ist
-  Ich kann Factory & Interface kombinieren
-  Ich vermeide unnötige Komplexität

## 13. Offizielle Referenzen

- [Kotlin Design Patterns (Allgemeine Konzepte)](https://kotlinlang.org/docs/object-declarations.html)
- [Factory Pattern (Erklärung allgemein)](https://refactoring.guru/design-patterns/factory-method)