# Coroutines in Android

**Nebenläufigkeit, Lifecycle & strukturierte Parallelität**

## Ziel dieses Dokuments
Nach der Durcharbeitung dieses Dokuments kannst du:
- erklären, **warum Coroutines der empfohlene Standard** in Android sind
- Coroutines **thread-sicher und lifecycle-bewusst** einsetzen
- typische Fehler (Leaks, blockierter Main Thread, verlorene Jobs) vermeiden
- Dispatcher korrekt auswählen
- Coroutines sauber mit ViewModel und UI kombinieren
- performanten, wartbaren und modernen Android-Code schreiben

## 1. Warum Coroutines?
Moderne Android-Apps müssen:
- Netzwerkzugriffe ausführen
- Datenbanken abfragen
- auf Sensoren reagieren
- UI flüssig halten

> All das darf **nicht** im Main Thread passieren.
### Probleme klassischer Ansätze
- Callbacks → schwer lesbar
- Threads → fehleranfällig
- manuelles Abbrechen → komplex
### Lösung: Coroutines
- sequentieller Code-Stil
- einfache Nebenläufigkeit
- automatische Abbruchlogik

Offizielle Referenz:  
[https://developer.android.com/topic/libraries/architecture/coroutines](https://developer.android.com/topic/libraries/architecture/coroutines?utm_source=chatgpt.com)

## 2. Grundkonzept von Coroutines

> Eine Coroutine ist **kein Thread**, sondern eine leichtgewichtige Aufgabe.

Coroutines:
- laufen auf Dispatchern
- können pausieren (`suspend`)
- blockieren keinen Thread
- sind günstig in der Erstellung

## 3. Wichtige Begriffe
### `suspend`
- markiert eine Funktion, die pausieren darf
- blockiert keinen Thread
```kotlin
suspend fun loadData(): String
```

### CoroutineScope
- definiert **Lebensdauer** einer Coroutine
- wenn der Scope endet → Coroutine wird abgebrochen

### Dispatcher
- entscheidet, **wo** Code ausgeführt wird

## 4. Dispatcher richtig einsetzen

### Die wichtigsten Dispatcher

|Dispatcher|Zweck|
|---|---|
|`Dispatchers.Main`|UI|
|`Dispatchers.IO`|Netzwerk, DB|
|`Dispatchers.Default`|CPU-lastige Arbeit|

### Typischer Fehler
- alles auf `Main`  
- alles auf `IO`
### Richtige Denkweise
> **Der Code bestimmt den Dispatcher, nicht die Funktion.**

## 5. Strukturierte Nebenläufigkeit (sehr wichtig)

### Grundidee
> Jede Coroutine gehört zu einem Scope.  
> Kein Scope → keine Kontrolle.

Das bedeutet:
- keine „verwaisten“ Jobs
- automatisches Aufräumen
- weniger Leaks

Offizielle Erklärung:  
[https://developer.android.com/kotlin/coroutines/coroutines-best-practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)

## 6. Coroutines im ViewModel

### Warum ViewModel?
- lebt länger als UI
- ideal für Business-Logik
- bietet `viewModelScope`
### Beispiel
```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

class ExampleViewModel(
    private val repository: ExampleRepository
) : ViewModel() {

    fun loadData() {
        viewModelScope.launch {
            val result = withContext(Dispatchers.IO) {
                repository.loadData()
            }
            // State aktualisieren
        }
    }
}
```

### Warum ist das korrekt?
- Coroutine endet mit ViewModel
- IO läuft nicht im Main Thread
- kein manuelles Cancel nötig

## 7. Coroutines im Fragment

### Wichtigste Regel
> **Keine eigenen GlobalScopes verwenden.**

**Falsch**
```kotlin
GlobalScope.launch { ... }
```

### Richtig: Lifecycle-aware Scopes
```kotlin
viewLifecycleOwner.lifecycleScope.launch {
	// UI-bezogene Coroutine 
}
```

Oder:
```kotlin
viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
     // State sammeln 
 }
```

## 8. Parallelität mit `async`

### Wann sinnvoll?
- mehrere unabhängige Aufgaben
- Ergebnis wird später benötigt
```kotlin
viewModelScope.launch {
    val a = async(Dispatchers.IO) { loadA() }
    val b = async(Dispatchers.IO) { loadB() }

    val result = a.await() + b.await()
}
```
### Regel
> Nutze `async` **nur**, wenn du `await()` brauchst.

## 9. Fehlerbehandlung in Coroutines

### try/catch funktioniert!
```kotlin
viewModelScope.launch {
    try {
        val data = repository.loadData()
    } catch (e: Exception) {
        // Fehler behandeln
    }
}
```
### Wichtig
- Fehler beenden standardmäßig die Coroutine
- Eltern-Coroutine wird informiert

## 10. Cancellation verstehen

### Coroutines sind kooperativ abbrechbar
- Abbruch passiert an `suspend`-Punkten
- lange Schleifen müssen abbrechbar sein
```kotlin
while (isActive) {
    doWork()
}
```

> `isActive` prüfen, wenn nötig.

## 11. Thread-Safety & Shared State

### Grundregel
> Mutable Daten nur an **einem Ort** verändern.
### Empfehlungen
- State über `StateFlow`
- keine geteilten `var`s
- kein gleichzeitiger Zugriff ohne Schutz

## 12. Typische Fehler (bitte vermeiden)
- `GlobalScope`  
- Blockierender Code im Main Thread  
- Vergessene Dispatcher  
- Nebenläufige State-Manipulation  
- Coroutine ohne Lifecycle-Bezug

## 13. Debugging von Coroutines

### Hilfreich
```kotlin
Log.d(TAG, "Thread: ${Thread.currentThread().name}")
```
### Android Studio
- Coroutine Debugger aktivieren
- zeigt laufende Coroutines & Scopes

Offizielle Referenz:  
https://developer.android.com/kotlin/coroutines/debugging

## 14. Selbst-Check

-  Kein `GlobalScope`
-  Richtiger Dispatcher
-  Coroutine endet mit Lifecycle
-  Fehler werden behandelt
-  State wird sicher aktualisiert

## 15. Offizielle Quellen

- [Coroutines Overview](https://developer.android.com/topic/libraries/architecture/coroutines)
- [Best Practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
- [Debugging Coroutines](https://developer.android.com/kotlin/coroutines/debugging)