# Coroutines, Flow und asynchrone Programmierung in Kotlin (JVM)

## Ziel

- Nebenläufigkeit und Asynchronität korrekt einordnen
- Coroutines strukturiert einsetzen
- Blocking vs. Non-Blocking verstehen
- Structured Concurrency anwenden
- `suspend`-Funktionen korrekt modellieren
- Mit `Flow` Datenströme verarbeiten
- Nebenläufigen Code testbar und wartbar gestalten

# 1. Warum asynchrone Programmierung?

Typische Szenarien:

- Netzwerkzugriffe
- Datenbankzugriffe
- Dateisystem
- Rechenintensive Operationen
- Parallelisierung

Ohne Nebenläufigkeit:

- Blockierende Threads
- Schlechte Performance
- Unresponsive Anwendungen

# 2. Thread vs. Coroutine

## 2.1 Thread

- Schwergewichtig
- Betriebssystem-verwaltet
- Hoher Overhead

## 2.2 Coroutine

- Leichtgewichtig
- Von Kotlin verwaltet
- Millionen möglich
- Kooperatives Scheduling

# 3. Gradle-Abhängigkeit

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")
}
```

# 4. Grundstruktur einer Coroutine

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        delay(1000)
        println("Hello from coroutine")
    }
    println("Start")
}
```

Wichtige Bausteine:

- `runBlocking`
- `launch`
- `delay`

# 5. suspend-Funktionen

`suspend` markiert Funktionen, die pausieren können.

```kotlin
suspend fun fetchData(): String {
    delay(1000)
    return "Daten"
}
```

Aufruf nur innerhalb einer Coroutine.

# 6. launch vs. async

## 6.1 launch

- Fire-and-forget
- Rückgabe: `Job`

```kotlin
val job = launch {
    delay(1000)
}
```

## 6.2 async

- Liefert Ergebnis
- Rückgabe: `Deferred<T>`

```kotlin
val deferred = async {
    delay(1000)
    42
}

val result = deferred.await()
```

# 7. Structured Concurrency

Prinzip:

> Coroutines sind hierarchisch organisiert.

Beispiel:

```kotlin
runBlocking {
    launch {
        launch {
            println("Child coroutine")
        }
    }
}
```

Wenn Parent beendet wird:

- Children werden automatisch beendet

# 8. Dispatcher

Bestimmen, auf welchem Thread ausgeführt wird.

```kotlin
withContext(Dispatchers.IO) {
    // IO-Operation
}
```

Wichtige Dispatcher:

- `Dispatchers.Default` (CPU)
- `Dispatchers.IO` (I/O)
- `Dispatchers.Unconfined` (nicht empfohlen)

Best Practice:

- Rechenintensiv → Default
- Blockierend → IO

# 9. Parallelisierung

```kotlin
runBlocking {
    val one = async { calculateOne() }
    val two = async { calculateTwo() }

    val result = one.await() + two.await()
}
```

Beide laufen parallel.

# 10. Exception Handling in Coroutines

Fehler in `launch`:

- Werden an Parent weitergegeben

```kotlin
runBlocking {
    launch {
        throw RuntimeException("Fehler")
    }
}
```

Mit `try-catch`:

```kotlin
runBlocking {
    try {
        launch {
            throw RuntimeException("Fehler")
        }.join()
    } catch (e: Exception) {
        println("Fehler abgefangen")
    }
}
```

# 11. SupervisorJob

Fehler isolieren:

```kotlin
val scope = CoroutineScope(SupervisorJob())

scope.launch {
    // Fehler hier beendet nicht andere Coroutines
}
```

# 12. Flow – Datenströme

Flow repräsentiert asynchrone Streams.

```kotlin
import kotlinx.coroutines.flow.*

fun numbers(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(100)
        emit(i)
    }
}
```

Sammeln:

```kotlin
runBlocking {
    numbers().collect {
        println(it)
    }
}
```

# 13. Flow-Operatoren

## map

```kotlin
numbers()
    .map { it * 2 }
    .collect { println(it) }
```

## filter

```kotlin
numbers()
    .filter { it % 2 == 0 }
```

## take

```kotlin
numbers()
    .take(3)
```

# 14. Cold vs. Hot Streams

Flow ist standardmäßig **cold**:

- Code wird erst bei `collect()` ausgeführt

# 15. SharedFlow / StateFlow

## StateFlow

- Hält aktuellen Wert
- Ideal für State-Management

```kotlin
val state = MutableStateFlow(0)
state.value = 1
```


# 16. Cancellation

Coroutines sind kooperativ abbrechbar.

```kotlin
val job = launch {
    repeat(1000) {
        delay(100)
    }
}

job.cancel()
```

# 17. Testen von Coroutines

```kotlin
import kotlinx.coroutines.test.runTest

@Test
fun `fetchData returns correct value`() = runTest {
    val result = fetchData()
    assertEquals("Daten", result)
}
```

# 18. Best Practices

- Kein `GlobalScope`
- Structured Concurrency verwenden
- Dispatcher bewusst wählen
- Blocking Code nicht im Default-Dispatcher
- Coroutines nicht als Ersatz für schlechte Architektur

# 19. Anti-Patterns

- `runBlocking` in Produktionscode
- Unkontrollierte CoroutineScopes
- Fehlende Fehlerbehandlung
- Blocking IO ohne Dispatcher.IO

# Definition of Done

- Unterschied Thread vs Coroutine verstanden
- `suspend` korrekt eingesetzt
- Structured Concurrency angewendet
- Flow korrekt genutzt
- Kein GlobalScope