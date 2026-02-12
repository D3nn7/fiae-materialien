# Architekturprinzipien, SOLID und sauberes Softwaredesign

## Ziel

- Grundprinzipien sauberer Softwarearchitektur verstehen
- SOLID-Prinzipien korrekt anwenden
- Verantwortlichkeiten klar trennen
- Abhängigkeiten bewusst modellieren
- Testbare und wartbare Strukturen entwerfen
- Anti-Patterns erkennen und vermeiden

---

# 1. Warum Architektur wichtig ist

Ohne klare Architektur entstehen:

- Unstrukturierter Code
- Enge Kopplung
- Schwer testbare Komponenten
- Hoher Wartungsaufwand
- Fehleranfällige Erweiterungen

Gute Architektur bedeutet:

- Klare Verantwortlichkeiten
- Geringe Kopplung
- Hohe Kohäsion
- Erweiterbarkeit ohne Codebruch

---

# 2. Grundprinzipien sauberer Software

## 2.1 Single Responsibility Principle (SRP)

> Eine Klasse sollte genau eine fachliche Verantwortung haben.

Schlecht:

```kotlin
class UserService {

    fun saveUser(user: User) { }

    fun sendEmail(user: User) { }

    fun generateReport(user: User) { }
}
```

Problem:

- Speichern
- Kommunikation
- Reporting

Besser:

```kotlin
class UserRepository {
    fun save(user: User) { }
}

class EmailService {
    fun send(user: User) { }
}

class ReportService {
    fun generate(user: User) { }
}
```

## 2.2 Open-Closed Principle (OCP)

> Klassen sollten offen für Erweiterung, aber geschlossen für Modifikation sein.

Schlecht:

```kotlin
fun calculateDiscount(type: String): Double {
    return when (type) {
        "VIP" -> 0.2
        "REGULAR" -> 0.1
        else -> 0.0
    }
}
```

Bei neuem Typ muss Code geändert werden.

Besser:

```kotlin
interface DiscountStrategy {
    fun calculate(): Double
}

class VipDiscount : DiscountStrategy {
    override fun calculate() = 0.2
}

class RegularDiscount : DiscountStrategy {
    override fun calculate() = 0.1
}
```

Erweiterung durch neue Implementierungen, ohne bestehenden Code zu verändern.

## 2.3 Liskov Substitution Principle (LSP)

> Unterklassen müssen sich wie ihre Basisklasse verhalten.

Verstoß:

```kotlin
open class Payment {
    open fun refund() { }
}

class NonRefundablePayment : Payment() {
    override fun refund() {
        throw UnsupportedOperationException()
    }
}
```

Besser:

- Interface nur für refundfähige Zahlungen definieren

## 2.4 Interface Segregation Principle (ISP)

> Kleine, spezifische Interfaces statt großer, allgemeiner.

Schlecht:

```kotlin
interface Worker {
    fun work()
    fun eat()
}
```

Besser:

```kotlin
interface Workable {
    fun work()
}

interface Eatable {
    fun eat()
}
```

## 2.5 Dependency Inversion Principle (DIP)

> High-Level-Module sollen nicht von Low-Level-Modulen abhängen, sondern beide von Abstraktionen.

Schlecht:

```kotlin
class OrderService {

    private val repository = MySqlOrderRepository()

    fun save(order: Order) {
        repository.save(order)
    }
}
```

Starke Kopplung.

Besser:

```kotlin
interface OrderRepository {
    fun save(order: Order)
}

class OrderService(
    private val repository: OrderRepository
) {
    fun save(order: Order) {
        repository.save(order)
    }
}
```

Dependency Injection:

```kotlin
val service = OrderService(MySqlOrderRepository())
```

# 3. Schichtenarchitektur (Layered Architecture)

Typische Struktur:

- Presentation Layer
- Application/Use Case Layer
- Domain Layer
- Infrastructure Layer

## 3.1 Domain Layer

- Enthält Geschäftslogik
- Keine Framework-Abhängigkeiten

```kotlin
data class Order(val id: String, val amount: Double)

class OrderValidator {
    fun validate(order: Order) {
        require(order.amount > 0)
    }
}
```

## 3.2 Application Layer

- Orchestriert Use Cases
- Koordiniert Domain und Infrastruktur

```kotlin
class CreateOrderUseCase(
    private val repository: OrderRepository
) {
    fun execute(order: Order) {
        repository.save(order)
    }
}
```

## 3.3 Infrastructure Layer

- Datenbank
- Netzwerk
- File-System

```kotlin
class MySqlOrderRepository : OrderRepository {
    override fun save(order: Order) {
        // DB-Logik
    }
}
```

# 4. Abhängigkeiten korrekt steuern

Regel:

> Abhängigkeiten zeigen immer nach innen (zur Domain).

Domain kennt keine Datenbank.  
Domain kennt keine Frameworks.

# 5. Komposition statt Vererbung

Schlecht:

```kotlin
class PremiumUser : User()
```

Besser:

```kotlin
class User(
    val role: Role
)

enum class Role {
    STANDARD,
    PREMIUM
}
```

# 6. Testbarkeit als Qualitätsindikator

Schlecht testbar:

```kotlin
class PaymentService {

    fun process() {
        val connection = DatabaseConnection()
        connection.save()
    }
}
```

Besser:

```kotlin
class PaymentService(
    private val repository: PaymentRepository
)
```

Erlaubt:

- Mocking
- Unit Tests
- Austauschbarkeit

# 7. Anti-Patterns

## 7.1 God Class

- Zu viele Methoden
- Zu viele Verantwortlichkeiten

## 7.2 Feature Envy

- Klasse greift intensiv auf interne Details einer anderen Klasse zu

## 7.3 Anämisches Domainmodell

- Nur Daten
- Keine Logik
- Logik verteilt in Services

# 8. Saubere Paketstruktur (Beispiel)

```kotlin
de.example.app
 ├── domain
 │    ├── model
 │    ├── service
 │
 ├── application
 │    ├── usecase
 │
 ├── infrastructure
 │    ├── repository
 │
 └── presentation
      ├── controller
```

---

# 9. Refactoring-Beispiel

Vorher:

```kotlin
class OrderManager {

    fun create(order: Order) {
        if (order.amount <= 0) {
            throw IllegalArgumentException()
        }
        Database.save(order)
        Email.sendConfirmation(order)
    }
}
```

Nachher:

- OrderValidator
- OrderRepository
- NotificationService
- CreateOrderUseCase

Klare Verantwortlichkeiten.

# Definition of Done

- SOLID-Prinzipien verstanden
- Abhängigkeiten korrekt abstrahiert
- Keine direkte Infrastrukturabhängigkeit in Domain
- Dependency Injection angewendet
- Keine God Classes