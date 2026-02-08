# Fragments in Android

Fragment, FragmentManager & Navigation –> Lifecycle-sicher, state-stabil

## Ziel dieses Dokuments

Nach diesem Kapitel kannst du:

- Fragments korrekt und sicher einsetzen
- den Unterschied zwischen Fragment- und View-Lifecycle erklären
- typische Fragment-Bugs (Crashes, doppelte Aufrufe, Leaks) vermeiden
- den FragmentManager gezielt nutzen
- Navigation zwischen Screens kontrolliert und nachvollziehbar umsetzen

## 1. Was ist ein Fragment?

Ein Fragment ist ein wiederverwendbarer UI-Baustein, der:

- immer in einer Activity lebt
- einen eigenen Lifecycle hat
- dynamisch hinzugefügt, ersetzt oder entfernt werden kann
- Fragments sind kein Ersatz für Activities, sondern Screens oder Teil-Screens innerhalb einer Activity.

Offizielle Referenz:
https://developer.android.com/guide/fragments

## 2. Der wichtigste Punkt: Lifecycle verstehen

Fragment hat **ZWEI** Lifecycles

- Fragment-Lifecycle
- View-Lifecycle

```
Fragment existiert
 ├── View wird erstellt
 ├── View wird zerstört
 └── Fragment existiert evtl. weiter
```

> Die View kann weg sein, das Fragment aber noch leben.

Das ist die Ursache für viele Crashes.

## 3. Konsequenzen für deinen Code

**Niemals tun**

- Zugriff auf Views nach onDestroyView()
- ViewBinding als lateinit var im Fragment halten
- Coroutines an den Fragment-Lifecycle binden, die Views nutzen

**Immer tun**

- UI-Code nur zwischen onViewCreated() und onDestroyView()
- `viewLifecycleOwner` verwenden
- State aus dem ViewModel beobachten

Offizielle Referenz:
https://developer.android.com/guide/fragments/lifecycle

## 4. Minimal korrektes Fragment

```kotlin
class ExampleFragment : Fragment(R.layout.fragment_example) {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        // Hier: Views initialisieren, Listener setzen, State sammeln
    }
}
```

Warum kein Code in `onCreateView()`?

- unnötig kompliziert
- Fehleranfällig
- `onViewCreated()` ist der empfohlene Einstiegspunkt

## 5. FragmentManager

Der FragmentManager:

- verwaltet Fragments
- führt Fragment-Transaktionen aus
- verwaltet den Back Stack
- stellt Lifecycle-Callbacks bereit

Du hast zwei relevante Varianten:

- supportFragmentManager (Activity)
- childFragmentManager (Fragment in Fragment)

## 6. Fragment-Transaktionen verstehen

### Beispiel: Fragment ersetzen

```kotlin
parentFragmentManager.beginTransaction()
    .replace(R.id.container, DetailFragment())
    .addToBackStack(null)
    .commit()
```

Bedeutung

- replace → altes Fragment wird entfernt
- addToBackStack → Zurück-Navigation möglich
- commit → Transaktion ausführen

## 7. Häufige Fehler mit FragmentManager

### Fehler 1: Transaktion nach onSaveInstanceState

Symptom:

- Crash oder Warnung
- Fragment erscheint nicht

Ursache:

- State wurde bereits gespeichert

Lösung:

- Navigation nur aus UI-Events
- nicht aus asynchronen Callbacks ohne Lifecycle-Check

### Fehler 2: Mehrfaches Hinzufügen desselben Fragments

Symptom:

- doppelter UI-Aufbau
- mehrere Listener
- unerwartetes Verhalten

Ursache:

- Fragment wird bei Rotation erneut hinzugefügt

Lösung:

- Fragment nur einmal initialisieren
- ViewModel für State verwenden

## 8. Navigation zwischen Fragments

Grundidee

- Fragment A löst Navigation aus
- Activity oder Nav-Host verwaltet die Anzeige
- Daten werden explizit übergeben

### Beispiel: einfache Navigation

```kotlin
parentFragmentManager.beginTransaction()
    .replace(R.id.container, SecondFragment())
    .addToBackStack("Second")
    .commit()
```

Zurück-Navigation

```kotlin
parentFragmentManager.popBackStack()
```

## 9. Navigation-Requests aus Fragmenten

Wichtige Regel

> Ein Fragment **entscheidet**, dass navigiert wird –
> aber nicht **wie lange** es lebt.

Praktisch:

- Navigation wird durch Benutzeraktion ausgelöst
- kein automatisches Navigieren beim State-Update
- kein Navigieren aus Hintergrundthreads

## 10. Kommunikation zwischen Fragment und ViewModel

Richtiger Weg

- Fragment reagiert auf State
- ViewModel kennt keine Navigation
- Navigation passiert im Fragment

```kotlin
if (state.navigateToDetail) {
    parentFragmentManager.beginTransaction()
        .replace(R.id.container, DetailFragment())
        .addToBackStack(null)
        .commit()
}
```

## 11. Memory-Leak-Fallen bei Fragments

Typische Ursachen

- ViewBinding nicht freigegeben
- Listener nicht entfernt
- Coroutine läuft weiter und greift auf View zu

Richtige Denkweise

- Fragment lebt länger als seine View
- View darf jederzeit verschwinden
- ViewModel hält State, nicht die UI

## 12. Debugging von Fragment-Problemen

- Log Lifecycle-Methoden:
  ```kotlin
  Log.d(TAG, "onViewCreated")
  Log.d(TAG, "onDestroyView")
  ```

FragmentManager-Debugging:
https://developer.android.com/guide/fragments/debugging

13. Selbst-Check

- Ich kenne den Unterschied zwischen Fragment- und View-Lifecycle
- Ich nutze onViewCreated() korrekt
- Ich greife nie auf Views nach onDestroyView() zu
- Navigation ist nachvollziehbar
- Keine UI-Referenzen außerhalb der UI-Schicht

14. Offizielle Quellen

- [Fragments Overview](https://developer.android.com/guide/fragments)
- [Fragment Lifecycle](https://developer.android.com/guide/fragments/lifecycle)
- [Fragment Transactions](https://developer.android.com/guide/fragments/transactions)
- [Debugging Fragments](https://developer.android.com/guide/fragments/debugging)
