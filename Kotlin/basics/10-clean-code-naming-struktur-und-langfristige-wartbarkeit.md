# 10 – Clean Code: Naming, Struktur und langfristige Wartbarkeit in Kotlin

## Ziel

- Lesbaren und wartbaren Code schreiben
- Saubere Benennung konsequent anwenden
- Methoden und Klassen klar strukturieren
- Komplexität reduzieren
- Technische Schulden vermeiden
- Code-Reviews fachlich fundiert durchführen

# 1. Warum Clean Code entscheidend ist

Schlechter Code führt zu:

- Erhöhter Fehleranfälligkeit
- Schwieriger Erweiterbarkeit
- Unklarer Fachlogik
- Hohem Onboarding-Aufwand
- Technischer Verschuldung

Guter Code ist:

- Lesbar
- Vorhersagbar
- Testbar
- Änderbar
- Fachlich klar

# 2. Naming – Der wichtigste Faktor

## 2.1 Variablennamen

Schlecht:

```kotlin
val x = 10
val d = 0.2
```

Besser:

```kotlin
val maximumRetryCount = 10
val discountRate = 0.2
```

Regeln:

- Aussagekräftig
- Keine Abkürzungen
- Kein technischer Jargon ohne Kontext
- Fachsprache bevorzugen

## 2.2 Funktionsnamen

Schlecht:

```kotlin
fun doIt() { }
```

Besser:

```kotlin
fun calculateTotalPrice() { }
```

Regeln:

- Verb + Objekt
- Eindeutig
- Keine Mehrdeutigkeit

## 2.3 Boolean-Namen

Schlecht:

```kotlin
val active: Boolean
```

Besser:

```kotlin
val isActive: Boolean
val hasPermission: Boolean
val canProceed: Boolean
```

# 3. Funktionen klein halten

Schlecht:

```kotlin
fun processOrder(order: Order) {
    validate(order)
    calculateDiscount(order)
    saveToDatabase(order)
    sendEmail(order)
    logOrder(order)
}
```

Besser:

- Logik in klar getrennte Komponenten auslagern
- Single Responsibility einhalten

## 3.1 Maximale Funktionslänge

Richtwert:

- 5–20 Zeilen
- Eine klare Aufgabe
- Keine verschachtelten Kontrollstrukturen

# 4. Parameter reduzieren

Schlecht:

```kotlin
fun createUser(name: String, age: Int, email: String, phone: String)
```

Besser:

```kotlin
data class UserRegistration(
    val name: String,
    val age: Int,
    val email: String,
    val phone: String
)

fun registerUser(data: UserRegistration)
```

# 5. Vermeidung von Tiefen Verschachtelungen

Schlecht:

```kotlin
if (user != null) {
    if (user.isActive) {
        if (user.hasPermission) {
            process(user)
        }
    }
}
```

Besser:

```kotlin
if (user == null) return
if (!user.isActive) return
if (!user.hasPermission) return

process(user)
```

# 6. Frühzeitiges Beenden (Guard Clauses)

Erhöht Lesbarkeit.

# 7. Keine magischen Zahlen

Schlecht:

```kotlin
if (retryCount > 3) { }
```

Besser:

```kotlin
private const val MAX_RETRY_COUNT = 3

if (retryCount > MAX_RETRY_COUNT) { }
```

# 8. DRY – Don’t Repeat Yourself

Schlecht:

```kotlin
val total1 = price * 0.19
val total2 = otherPrice * 0.19
```

Besser:

```kotlin
private const val TAX_RATE = 0.19

fun calculateTax(amount: Double) = amount * TAX_RATE
```

# 9. KISS – Keep It Simple

Komplexität vermeiden:

Schlecht:

```kotlin
val result = list.filter { it % 2 == 0 }.map { it * 2 }.sorted().reversed()
```

Falls unnötig komplex → vereinfachen.

# 10. YAGNI – You Aren’t Gonna Need It

Nicht implementieren:

- Features ohne aktuellen Bedarf
- Abstraktionen ohne Use Case
- Über-Engineering

# 11. Kommentare richtig einsetzen

Schlecht:

```kotlin
// Addiere zwei Zahlen
fun add(a: Int, b: Int) = a + b
```

Besser:

- Kommentare nur bei:
    - Nicht offensichtlicher Logik
    - Fachlicher Besonderheit
    - Technischer Einschränkung

# 12. Logging statt Kommentieren

Keine erklärenden Kommentare für Debugging-Zwecke.  
Logging gezielt einsetzen.

# 13. Strukturierung von Dateien

- Eine Hauptklasse pro Datei
- Keine 1000-Zeilen-Dateien
- Logisch gruppierte Funktionen

# 14. Kotlin-spezifische Clean-Code-Prinzipien

## 14.1 val bevorzugen

Immutable Design ist sicherer.

## 14.2 Expression-Style nutzen

```kotlin
fun calculate(a: Int, b: Int) = a + b
```

## 14.3 Data Classes für Value Objects

## 14.4 Extension-Funktionen sparsam einsetzen

---

# 15. Refactoring-Techniken

- Extract Function
- Extract Class
- Rename
- Replace Conditional with Polymorphism
- Introduce Parameter Object

# 16. Code Smells

- Lange Methoden
- Lange Parameterlisten
- Primitive Obsession
- Feature Envy
- Duplicate Code
- Shotgun Surgery

# 17. Code Review Checkliste

- Ist Naming klar?
- Ist die Funktion zu lang?
- Gibt es versteckte Seiteneffekte?
- Ist der Code testbar?
- Sind Abhängigkeiten sauber abstrahiert?
- Gibt es unnötige Mutabilität?

# 18. Beispiel – Vorher / Nachher

Vorher:

```kotlin
fun calculate(order: Order): Double {
    var total = 0.0
    for (item in order.items) {
        total += item.price * item.quantity
    }
    if (total > 100) {
        total -= total * 0.1
    }
    return total
}
```

Nachher:

```kotlin
private const val DISCOUNT_THRESHOLD = 100.0
private const val DISCOUNT_RATE = 0.1

fun calculateTotal(order: Order): Double {
    val subtotal = order.items.sumOf { it.price * it.quantity }
    return applyDiscountIfEligible(subtotal)
}

private fun applyDiscountIfEligible(amount: Double): Double =
    if (amount > DISCOUNT_THRESHOLD)
        amount * (1 - DISCOUNT_RATE)
    else
        amount
```

Lesbarer, wartbarer, testbarer.

# Definition of Done

- Klare Benennung konsequent umgesetzt
- Kleine, fokussierte Funktionen
- Keine magischen Zahlen
- Geringe Verschachtelung
- Keine unnötigen Kommentare
- Refactoring angewendet