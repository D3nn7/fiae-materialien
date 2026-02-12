# 6 – Fehlerbehandlung, Exceptions und sauberes Error-Handling

## Ziel

- Exceptions korrekt verstehen und einsetzen
- Unterschied zwischen erwartbaren Fehlern und Programmierfehlern erkennen
- `try-catch-finally` sauber verwenden
- Eigene Exceptions definieren
- `require`, `check` und `assert` korrekt einsetzen
- Alternativen zu Exceptions (z. B. Result-Typen) verstehen
- Robuste, testbare Fehlerbehandlung implementieren

---

# 1. Grundlagen: Was ist eine Exception?

Eine Exception signalisiert einen **unerwarteten oder fehlerhaften Zustand** zur Laufzeit.

Beispiel:

```kotlin
val number = "abc".toInt() // NumberFormatException
```

Wenn eine Exception nicht behandelt wird:

- Das Programm bricht ab
- Der Stacktrace wird ausgegeben

# 2. try-catch-finally

Grundstruktur:
```kotlin
try {
    // potenziell fehlerhafter Code
} catch (e: Exception) {
    // Fehlerbehandlung
} finally {
    // wird immer ausgeführt
}
```

Beispiel:

```kotlin
fun parseNumber(input: String): Int? {
    return try {
        input.toInt()
    } catch (e: NumberFormatException) {
        null
    }
}
```

Wichtig:

- `try` ist in Kotlin ein Ausdruck und kann einen Wert zurückgeben.

# 3. Mehrere Catch-Blöcke

```kotlin
try {
    val result = 10 / 0
} catch (e: ArithmeticException) {
    println("Division durch Null")
} catch (e: Exception) {
    println("Allgemeiner Fehler")
}
```

Best Practice:

- Spezifische Exceptions zuerst
- Keine pauschalen `catch (Exception)` ohne Begründung

# 4. Eigene Exceptions definieren

```kotlin
class InvalidEmailException(message: String) : Exception(message)
```

Verwendung:

```kotlin
fun validateEmail(email: String) {
    if (!email.contains("@")) {
        throw InvalidEmailException("Ungültige E-Mail-Adresse")
    }
}
```

Best Practice:

- Fachlich aussagekräftige Exception-Namen
- Keine generischen `Exception`-Würfe

# 5. require, check und assert

## 5.1 require

Für **Eingabevalidierung**:

```kotlin
fun withdraw(amount: Double) {
    require(amount > 0) { "Betrag muss positiv sein" }
}
```

Wirft:

- `IllegalArgumentException`

## 5.2 check

Für **internen Zustand**:

```kotlin
fun process(state: String?) {
    check(state != null) { "State darf nicht null sein" }
}
```

Wirft:

- `IllegalStateException`

## 5.3 assert

Für Debug-Zwecke:

```kotlin
assert(value > 0)
```

Nur aktiv, wenn Assertions eingeschaltet sind.

# 6. Wann Exceptions verwenden?

Exceptions sind sinnvoll bei:

- Unerwarteten Systemfehlern
- Infrastrukturproblemen
- Programmierfehlern
- Nicht kontrollierbaren Zuständen

Nicht sinnvoll bei:

- Erwartbaren fachlichen Ergebnissen
- Validierung, die Teil normaler Logik ist

# 7. Alternative: Nullable Rückgabe

Statt:

```kotlin
fun findUser(id: Int): User {
    throw UserNotFoundException()
}
```

Besser:

```kotlin
fun findUser(id: Int): User?
```

Und aufrufseitig:

```kotlin
val user = findUser(1)
if (user != null) {
    // ...
}
```

# 8. Result-Typ

Kotlin bietet `Result<T>`:

```kotlin
fun parse(input: String): Result<Int> {
    return runCatching { input.toInt() }
}
```

Verarbeitung:

```kotlin
val result = parse("123")

result
    .onSuccess { println("Erfolg: $it") }
    .onFailure { println("Fehler: ${it.message}") }
```

## Eigener Result-Typ (Domain-orientiert)

```kotlin
sealed class OperationResult<out T> {

    data class Success<T>(val value: T) : OperationResult<T>()

    data class Failure(val reason: String) : OperationResult<Nothing>()
}
```

Verwendung:

```kotlin
fun divide(a: Int, b: Int): OperationResult<Int> {
    return if (b == 0) {
        OperationResult.Failure("Division durch Null")
    } else {
        OperationResult.Success(a / b)
    }
}
```

Verarbeitung:

```kotlin
when (val result = divide(10, 2)) {
    is OperationResult.Success -> println(result.value)
    is OperationResult.Failure -> println(result.reason)
}
```

Vorteile:

- Explizite Fehlerbehandlung
- Keine versteckten Laufzeitabbrüche
- Bessere Testbarkeit

# 9. Ressourcen sicher schließen

Mit `use` (AutoCloseable):

```kotlin
File("test.txt").bufferedReader().use { reader ->
    println(reader.readText())
}
```

`use`:

- Schließt Ressource automatisch
- Entspricht try-with-resources in Java

# 10. Anti-Patterns

## 10.1 Exception als Kontrollfluss

Schlecht:

```kotlin
try {
    val user = findUser(id)
} catch (e: Exception) {
    // normaler Ablauf
}
```

## 10.2 Leerer Catch-Block

```kotlin
catch (e: Exception) {
}
```

Unterdrückt Fehler vollständig.

## 10.3 Zu breite Exception-Behandlung

```kotlin
catch (e: Exception)
```

Versteckt Programmierfehler.

# 11. Logging statt Verschlucken

Fehler mindestens protokollieren:

```kotlin
catch (e: IOException) {
    println("Fehler beim Lesen: ${e.message}")
}
```

In realen Projekten:

- Logging-Framework nutzen
- Keine `println` in Produktionscode

# 12. Designrichtlinien

- Exceptions für unerwartete Fehler
- Nullable oder Result für erwartbare Fälle
- Keine versteckten Seiteneffekte
- Keine Business-Logik in Catch-Blöcken
- Validierung früh (Fail Fast)

# Definition of Done

- Unterschied zwischen erwartbarem Fehler und Exception verstanden
- `try-catch` korrekt eingesetzt
- Eigene Exceptions sinnvoll modelliert
- Result-Typ angewendet
- Keine leeren Catch-Blöcke