# Testen: Unit Tests, Mocking und testgetriebenes Denken

## Ziel

- Bedeutung von Tests für Qualität und Wartbarkeit verstehen
- Unit Tests mit Kotlin und JUnit strukturieren
- Testfälle fachlich korrekt formulieren
- Abhängigkeiten mit Test Doubles isolieren
- Mocking-Grundlagen verstehen
- Testgetriebenes Denken (TDD) anwenden

# 1. Warum testen?

Tests erfüllen mehrere Aufgaben:

- Verifikation fachlicher Korrektheit
- Absicherung gegen Regression
- Dokumentation des erwarteten Verhaltens
- Ermöglichung von Refactoring
- Qualitätsindikator für Architektur

Grundsatz:

> Nicht Code testen – Verhalten testen.

# 2. Arten von Tests

## 2.1 Unit Tests

- Testen eine einzelne Klasse oder Funktion
- Isolieren Abhängigkeiten
- Schnell ausführbar
- Kein Netzwerk, keine Datenbank

## 2.2 Integrationstests

- Testen Zusammenspiel mehrerer Komponenten
- Nutzen reale Infrastruktur

Im Fokus hier: **Unit Tests**

# 3. Test-Setup (Gradle)

Abhängigkeit:

```kotlin
dependencies {
    testImplementation(kotlin("test"))
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
}
```

Gradle-Konfiguration:

```gradle
tasks.test {
    useJUnitPlatform()
}
```

Ordnerstruktur:

```
src
 ├── main
 │    └── kotlin
 └── test
      └── kotlin

```

# 4. Grundstruktur eines Unit Tests

Beispielklasse:

```kotlin
class Calculator {

    fun add(a: Int, b: Int): Int = a + b
}
```

Test:

```kotlin
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Test

class CalculatorTest {

    @Test
    fun `add should return correct sum`() {
        // Arrange
        val calculator = Calculator()

        // Act
        val result = calculator.add(2, 3)

        // Assert
        assertEquals(5, result)
    }
}
```

# 5. AAA-Prinzip

- Arrange
- Act
- Assert

Struktur konsequent einhalten.

# 6. Testnamen

Testnamen sollen:

- Verhalten beschreiben
- Fachlich formuliert sein
- Klar und präzise sein

Beispiel:

```kotlin
fun `withdraw should reduce balance when amount is valid`() { }
```

Nicht:

```kotlin
fun test1() { }
```

# 7. Exception testen

Beispielklasse:

```kotlin
class Account(private var balance: Double) {

    fun withdraw(amount: Double) {
        require(amount > 0)
        require(amount <= balance)
        balance -= amount
    }
}
```

Test:

```kotlin
import org.junit.jupiter.api.assertThrows

@Test
fun `withdraw should throw when amount exceeds balance`() {
    val account = Account(100.0)

    assertThrows<IllegalArgumentException> {
        account.withdraw(200.0)
    }
}
```

# 8. Parametrisierte Tests

Für mehrere Eingabewerte:

```kotlin
import org.junit.jupiter.params.ParameterizedTest
import org.junit.jupiter.params.provider.CsvSource

class BmiTest {

    @ParameterizedTest
    @CsvSource(
        "50, 1.6, 19.53",
        "80, 1.8, 24.69"
    )
    fun `bmi calculation should be correct`(weight: Double, height: Double, expected: Double) {
        val bmi = weight / (height * height)
        assertEquals(expected, bmi, 0.01)
    }
}
```

# 9. Test Doubles

Arten:

- Dummy
- Stub
- Fake
- Spy
- Mock

# 10. Mocking mit Mockito (Beispiel)

Dependency:

```gradle
testImplementation("org.mockito.kotlin:mockito-kotlin:5.1.0")
```

Beispiel:

```kotlin
interface UserRepository {
    fun save(user: User)
}
```

Service:

```kotlin
class RegisterUserUseCase(
    private val repository: UserRepository
) {
    fun execute(user: User) {
        repository.save(user)
    }
}
```

Test:

```kotlin
import org.mockito.kotlin.*

class RegisterUserUseCaseTest {

    @Test
    fun `execute should call repository save`() {
        val repository = mock<UserRepository>()
        val useCase = RegisterUserUseCase(repository)
        val user = User("Max")

        useCase.execute(user)

        verify(repository).save(user)
    }
}
```

# 11. Fake statt Mock (bevorzugt)

In vielen Fällen besser:

```kotlin
class InMemoryUserRepository : UserRepository {

    val savedUsers = mutableListOf<User>()

    override fun save(user: User) {
        savedUsers.add(user)
    }
}
```

Test:

```kotlin
@Test
fun `execute should store user in repository`() {
    val repository = InMemoryUserRepository()
    val useCase = RegisterUserUseCase(repository)
    val user = User("Max")

    useCase.execute(user)

    assertEquals(1, repository.savedUsers.size)
}
```

Best Practice:

- Fake bevorzugen
- Mock nur bei komplexer Interaktion

# 12. Testgetriebenes Denken (TDD)

Zyklus:

1. Red – Test schreiben (fehlschlägt)
2. Green – Minimal implementieren
3. Refactor – Code verbessern

Beispiel:

Test zuerst:

```kotlin
@Test
fun `divide should return correct result`() {
    val calculator = Calculator()
    assertEquals(5, calculator.divide(10, 2))
}
```

Dann Implementierung.

# 13. Eigenschaften guter Tests

- Schnell
- Isoliert
- Deterministisch
- Lesbar
- Wartbar

# 14. Anti-Patterns

- Logik im Test
- Mehrere Verantwortlichkeiten pro Test
- Abhängigkeit von externer Infrastruktur
- Zeitabhängige Tests ohne Kontrolle
- Testen von Implementierungsdetails statt Verhalten

# 15. Coverage ist kein Qualitätsmerkmal

Hohe Coverage ≠ gute Tests

Wichtiger:

- Kritische Pfade abgesichert
- Edge Cases getestet
- Fehlerfälle getestet

# Definition of Done

- Unit Tests sind klar strukturiert  
- AAA-Prinzip wird eingehalten
- Exceptions werden getestet
- Fake statt unnötiger Mocks
- Keine Infrastrukturabhängigkeiten
- Tests laufen deterministisch