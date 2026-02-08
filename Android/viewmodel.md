# ViewModel in Android (Views)

Sauberer UI-State und thread-sicher

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:

- UI-State korrekt außerhalb von Activity und Fragment halten
- Konfigurationsänderungen (z. B. Rotation) ohne erneutes Laden überstehen
- Nebenläufige Aufgaben thread-sicher umsetzen
- Memory Leaks zuverlässig vermeiden
- ViewModels ohne zusätzliche Frameworks sauber erstellen
- Code schreiben, der den offiziellen Android Best Practices entspricht

## 1. Warum ein ViewModel?

Ein ViewModel ist eine Klasse, die:

- an den Lifecycle eines Screens gebunden ist
- UI-Logik von UI-Code trennt
- Konfigurationsänderungen überlebt
- automatisch aufgeräumt wird, wenn der Screen endgültig verschwindet

### Typische Probleme ohne ViewModel

- Rotation → Fragment wird neu erstellt → Daten werden erneut geladen
- Asynchrone Tasks laufen weiter → Zugriff auf zerstörte Views
- Speicherlecks durch Referenzen auf Context, View oder Activity

Das ViewModel löst genau diese Probleme.

Offizielle Referenz:
https://developer.android.com/topic/libraries/architecture/viewmodel

## 2. Zentrale Grundregel

**Ein ViewModel kennt keine Views.**

Ein ViewModel darf niemals halten oder referenzieren:

- `Activity`
- `Fragment`
- `View`
- `Context`
- `ViewBinding`
- UI-spezifische Klassen

> Alles, was mit Darstellung oder Benutzerinteraktion zu tun hat, gehört ausschließlich in die UI-Schicht.

## 3. Architektur-Überblick

```graphql
Fragment / Activity
        ↓ beobachtet State
      ViewModel
        ↓ ruft Logik auf
     Repository
```

- UI: zeigt Daten an, reagiert auf State
- ViewModel: hält State, steuert Logik
- Repository: kapselt Datenquellen (z. B. Netzwerk, Datenbank, Location)

Diese Trennung macht den Code testbar, wartbar und robust.

### 4. UI-State korrekt modellieren

### Grundprinzip

- State ist immutable nach außen
- Änderungen erfolgen nur im ViewModel

### Beispiel für einen UI-State

```kotlin
data class ExampleUiState(
    val isLoading: Boolean = false,
    val content: String? = null,
    val errorMessage: String? = null
)
```

Vorteile:

- klar definierter Zustand
- keine halbfertigen UI-Zwischenstände
- leichter testbar

## 5. ViewModel mit StateFlow

### Warum StateFlow?

- thread-sicher
- lifecycle-freundlich
- klarer, beobachtbarer State
- empfohlener Standard laut
  Android-Dokumentation

Offizielle Referenz:
https://developer.android.com/topic/libraries/architecture/coroutines

### Beispiel: ViewModel-Implementierung

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

class ExampleViewModel(
    private val repository: ExampleRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(ExampleUiState())
    val uiState: StateFlow<ExampleUiState> = _uiState.asStateFlow()

    fun loadData() {
        viewModelScope.launch {
            _uiState.value = _uiState.value.copy(isLoading = true)

            val result = withContext(Dispatchers.IO) {
                repository.loadData()
            }

            _uiState.value = ExampleUiState(
                isLoading = false,
                content = result
            )
        }
    }
}
```

### Warum ist das korrekt?

- `viewModelScope` wird automatisch gecancelt
- IO-Arbeit läuft nicht auf dem Main Thread
- kein Zugriff auf UI-Objekte
- State wird kontrolliert und atomar aktualisiert

## 6. Fragment: State sicher sammeln

### Wichtig bei Fragments

Der Lifecycle der View ist **kürzer** als der Lifecycle des Fragments.

> Deshalb immer `viewLifecycleOwner` verwenden.

Offizielle Referenz:
https://developer.android.com/guide/fragments/lifecycle

### Beispiel: Fragment-Implementierung

```kotlin
import android.os.Bundle
import android.view.View
import androidx.fragment.app.Fragment
import androidx.fragment.app.viewModels
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import kotlinx.coroutines.launch

class ExampleFragment : Fragment(R.layout.fragment_example) {

    private val viewModel: ExampleViewModel by viewModels {
        ExampleViewModelFactory(ExampleRepository())
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    // progressBar.isVisible = state.isLoading
                    // textView.text = state.content ?: ""
                }
            }
        }

        // button.setOnClickListener { viewModel.loadData() }
    }
}
```

### Warum `repeatOnLifecycle(STARTED)`?

- Sammlung stoppt automatisch, wenn die View nicht sichtbar ist
- kein Zugriff auf zerstörte Views
- deutlich weniger Crashes und Leaks

## 7. ViewModel mit Abhängigkeiten erstellen

### Factory-Pattern (empfohlen)

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider

class ExampleViewModelFactory(
    private val repository: ExampleRepository
) : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(ExampleViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return ExampleViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

### Vorteile:

- explizite Abhängigkeiten
- keine versteckte Magie
- gut testbar

Offizielle Referenz:
https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-factories

## 8. Thread-Safety & Performance

### Do

- IO-Arbeit → Dispatchers.IO
- CPU-lastige Arbeit → Dispatchers.Default
- UI-Updates → automatisch über StateFlow

### Don’t

- Blockierende Aufrufe im Main Thread
- globale mutable Variablen
- parallele State-Manipulation ohne Kontrolle

### 9. Häufige Fehler

- ViewModel speichert Context
- Coroutine ohne klaren Scope
- UI-Zugriff aus Hintergrundthread
- State wird direkt aus dem Fragment verändert
- Daten werden bei jeder Rotation neu geladen

## 10. Selbst-Check (Review)

- Keine UI-Referenzen im ViewModel
- State nur über StateFlow
- Nebenläufigkeit korrekt verteilt
- `repeatOnLifecycle` im Fragment genutzt
- ViewModel über Factory erstellt

## 11. Offizielle Quellen

- [ViewModel Overview](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [ViewModel Factories](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-factories)
- [Coroutines & Lifecycle](https://developer.android.com/topic/libraries/architecture/coroutines)
- [Fragment Lifecycle](https://developer.android.com/guide/fragments/lifecycle)
