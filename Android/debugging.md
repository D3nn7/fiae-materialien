# Debugging in Android

Logcat, Debugger & sauberes Logging mit Log

## Ziel dieses Dokuments

Nach diesem Kapitel kannst du:

- Logcat gezielt nutzen, um Fehler schnell einzugrenzen
- Crash-Stacktraces lesen und verstehen
- Den Android Studio Debugger effektiv einsetzen
- Sinnvolles Logging schreiben
- Typische Android-Fehler systematisch analysieren
- Debugging betreiben, ohne neue Bugs oder Performance-Probleme zu erzeugen

## 1. Was bedeutet Debugging eigentlich?

Debugging heißt nicht:

- „Ich probiere mal irgendwas“
- „Ich starte die App 20-mal neu“

Debugging heißt:

- **Hypothese bilden → prüfen → Ergebnis bewerten**

Android gibt dir dafür sehr gute Werkzeuge. Du musst sie nur korrekt einsetzen.

## 2. Logcat – dein wichtigstes Diagnosewerkzeug

### Was ist Logcat?

Logcat ist der zentrale Log-Ausgabekanal von Android.
Dort landen:

- deine `Log.*()` Ausgaben
- Systemmeldungen
- Exceptions inkl. Stacktrace
- Lifecycle-Events
- Thread-Informationen

Offizielle Referenz:
https://developer.android.com/studio/debug/logcat

## 2.1 Log-Level

| Level   | Bedeutung            | Wann verwenden           |
| ------- | -------------------- | ------------------------ |
| VERBOSE | sehr detailliert     | fast nie                 |
| DEBUG   | Entwicklungsinfos    | Standard beim Entwickeln |
| INFO    | wichtige Statusinfos | selten                   |
| WARN    | unerwarteter Zustand | wenn etwas „komisch“ ist |
| ERROR   | Fehler               | bei echten Problemen     |

## 2.2 Sinnvolle Log-Tags

```kotlin
private const val TAG = "LocationViewModel"
```

**Regeln**:

- kurz
- eindeutig
- pro Klasse ein Tag

**Schlecht**:

```kotlin
Log.d("TEST", "hier")
```

**Gut**:

```kotlin
Log.d(TAG, "Location update started")
```

## 3. Logging mit Log

### Grundsyntax

```kotlin
Log.d(TAG, "Nachricht")
Log.w(TAG, "Unerwarteter Zustand")
Log.e(TAG, "Fehler aufgetreten", throwable)
```

### 3.1 Best Practices für Logging

**Do**

- Logs beschreiben Zustand oder Entscheidung
- Logs helfen dir, einen Ablauf nachzuvollziehen
- Logs sind kurz und eindeutig

**Don’t**

- keine personenbezogenen Daten loggen
- keine riesigen Objekte dumpen
- kein Logging in extrem häufig aufgerufenen Methoden (z. B. onBindViewHolder)

#### Beispiel: schlechtes vs. gutes Logging

**Schlecht**

```kotlin
Log.d(TAG, "hier 1")
Log.d(TAG, "hier 2")
Log.d(TAG, "hier 3")
```

**Gut**

```kotlin
Log.d(TAG, "Permission granted, starting location updates")
```

## 4. Stacktraces lesen (extrem wichtig)

Wenn deine App crasht, siehst du in Logcat einen Stacktrace.

### Beispiel (vereinfacht)

```kotlin
java.lang.NullPointerException
at com.example.MainViewModel.loadData(MainViewModel.kt:42)
at com.example.MainFragment.onViewCreated(MainFragment.kt:28)
```

Wie du das liest:

- erste Zeile → Art des Fehlers
- erste Zeile aus deinem Package → Ursache
- Dateiname + Zeilennummer → direkt dorthin springen

> Meist liegt der Fehler nicht in der letzten Zeile, sondern dort, wo dein Code erstmals auftaucht.

## 5. Android Studio Debugger

Offizielle Referenz:
https://developer.android.com/studio/debug

Der Debugger erlaubt dir:

- Code anzuhalten
- Variablen live zu inspizieren
- Schrittweise durch Code zu gehen

### 5.1 Breakpoints

Ein Breakpoint stoppt die App genau an einer Codezeile.

So nutzt du ihn sinnvoll:

- setze ihn dort, wo du eine Entscheidung erwartest
- nicht blind überall

### 5.2 Step-Optionen

| Action    | Bedeutung                  |
| --------- | -------------------------- |
| Step Over | aktuelle Zeile ausführen   |
| Step Into | in Methode hineinspringen  |
| Step Out  | aktuelle Methode verlassen |
| Resume    | App weiterlaufen lassen    |

### 5.3 Variablen & Watches

Während der Debugger pausiert ist:

- siehst du alle lokalen Variablen
- kannst du Expressions auswerten
- kannst du prüfen, ob dein Code das tut, was du denkst

## 6. Debugging mit Coroutines

Typische Frage

_„Warum kommt mein Ergebnis nicht zurück?“_

Vorgehen

1. Breakpoint vor withContext
2. Breakpoint nach withContext
3. Thread-Namen prüfen

```kotlin
Log.d(TAG, "Thread: ${Thread.currentThread().name}")
```

Erwartung:

- IO-Arbeit → DefaultDispatcher-worker
- UI → main

Offizielle Referenz:
https://developer.android.com/topic/libraries/architecture/coroutines

## 7. Häufige Debugging-Szenarien

### Szenario 1: Klick reagiert nicht

- Breakpoint im ClickListener
- prüfen: wird er erreicht?
- prüfen: richtige View-ID?

### Szenario 2: UI aktualisiert sich nicht

- kommt neuer State an?
- wird der Collector ausgeführt?
- ist die View noch vorhanden?

### Szenario 3: App crasht beim Zurück-Navigieren

- Stacktrace prüfen
- Zugriff auf View nach onDestroyView?
- falscher LifecycleOwner?

## 8. Debugging & Performance

Debug-Code kann selbst Probleme verursachen.

**Don’t**

- endlose Logs in Schleifen
- Debugger dauerhaft aktiv lassen
- große Objekte inspizieren

**Do**

- gezielt debuggen
- danach Logs reduzieren
- Code wieder sauber machen

## 9. Selbst-Check

- Ich kann Logcat filtern
- Ich verstehe Stacktraces
- Ich setze Breakpoints gezielt
- Ich weiß, welcher Thread läuft
- Mein Logging ist sinnvoll und sparsam

## 10. Offizielle Quellen

- [Logcat](https://developer.android.com/studio/debug/logcat)
- [Debug your app](https://developer.android.com/studio/debug)
- [Debugging Coroutines](https://developer.android.com/kotlin/coroutines/debugging)
