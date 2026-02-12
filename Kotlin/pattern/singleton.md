# Singleton Pattern

**Globale Instanzen korrekt, thread-sicher und kontrolliert einsetzen**

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:
- erklären, was ein Singleton ist
- Singletons in Kotlin korrekt implementieren
- thread-sichere Initialisierung verstehen
- typische Fehler (Global State, versteckte Abhängigkeiten) vermeiden
- erkennen, wann ein Singleton sinnvoll – und wann problematisch – ist

## 1. Was ist ein Singleton?

Ein Singleton ist ein Objekt, von dem es:
- genau **eine Instanz** im gesamten Programm gibt
- einen **globalen Zugriffspunkt** gibt

Beispiel:

```kotlin
object Logger {
    fun log(message: String) {
        println(message)
    }
}

```

Hier garantiert Kotlin:
- nur eine Instanz
- thread-sichere Initialisierung

## 2. Warum Singletons existieren

Typische Anwendungsfälle:
- Logging
- Konfigurationsverwaltung
- Datenbank-Provider
- Cache-Manager
- zentrale Services

### Ziel
> Eine Ressource soll nur einmal existieren.

## 3. Kotlin `object` – der empfohlene Weg

Kotlin bietet eine native Singleton-Implementierung:

```kotlin
object AppConfig {
    val baseUrl = "https://api.example.com"
}
```

Vorteile:
- thread-sicher
- lazy initialisiert
- kein Boilerplate

Offizielle Referenz:  
[https://kotlinlang.org/docs/object-declarations.html](https://kotlinlang.org/docs/object-declarations.html)

---

## 4. Lazy Initialisierung verstehen

Kotlin `object` wird:
- erst beim ersten Zugriff initialisiert
- automatisch thread-sicher erzeugt

Du musst:
- keine Synchronisierung schreiben
- keine volatile Variablen nutzen

## 5. Singleton mit Parametern (Problemfall)

Ein `object` kann keinen Konstruktor mit Parametern haben.

Wenn du Parameter brauchst:
```kotlin
class Database private constructor() {

    companion object {

        @Volatile
        private var INSTANCE: Database? = null

        fun getInstance(): Database {
            return INSTANCE ?: synchronized(this) {
                INSTANCE ?: Database().also { INSTANCE = it }
            }
        }
    }
}
```

### Warum so kompliziert?

- thread-sicher
- nur eine Instanz
- doppelte Initialisierung verhindern

## 6. `@Volatile` und `synchronized` verstehen

### `@Volatile`
- stellt sicher, dass alle Threads denselben Wert sehen

### `synchronized`
- verhindert parallele Initialisierung

Dieses Muster nennt man:
> Double-Checked Locking

## 7. Android-relevantes Beispiel: Room Database

```kotlin
object DatabaseProvider {

    @Volatile
    private var INSTANCE: AppDatabase? = null

    fun getDatabase(context: Context): AppDatabase {
        return INSTANCE ?: synchronized(this) {
            INSTANCE ?: Room.databaseBuilder(
                context.applicationContext,
                AppDatabase::class.java,
                "app_database"
            ).build().also { INSTANCE = it }
        }
    }
}
```

Warum korrekt?
- Application Context → kein Leak
- nur eine Instanz
- thread-sicher

## 8. Wann ist ein Singleton sinnvoll?

### Sinnvoll bei:
- globaler Konfiguration
- Ressourcen-Managern
- Cache
- Logging

### Nicht sinnvoll bei:
- Business-Logik
- ViewModels
- Klassen mit wechselndem Zustand
- Test-sensitiven Komponenten

## 9. Typische Fehler (bitte vermeiden)

- Globaler mutable State  
- Singleton kennt UI  
- Singleton speichert Context (Activity!)  
- Singleton als Ersatz für saubere Architektur

> Ein Singleton ist kein Ersatz für Dependency Management.

## 10. Testbarkeit beachten

Problem:
- Singletons sind global
- schwer mockbar

Lösung:
- Interface definieren
- Singleton nur als Implementierung nutzen

Beispiel:
```kotlin
interface Logger {
    fun log(message: String)
}

object DefaultLogger : Logger {
    override fun log(message: String) {
        println(message)
    }
}
```

In Tests:
- FakeLogger verwenden

## 11. Thread-Safety & Mutable State

Ein Singleton ist nur als Instanz thread-sicher –  
nicht automatisch sein interner Zustand.

Beispiel problematisch:
```kotlin
object Counter {
    var value = 0
}
```

Mehrere Threads → Race Conditions.

Lösung:
- immutable State
- Synchronisierung
- oder atomare Typen

## 12. Singleton vs. Object vs. Companion

### `object`
- echtes Singleton

### `companion object`
- statische Methoden/Member
- kein vollständiger Singleton-Ersatz

Beispiel:
```kotlin
class Utils {
    companion object {
        fun helper() {}
    }
}
```

## 13. Selbst-Check

-  Ich verstehe Kotlin `object`
-  Ich weiß, wann ein Singleton sinnvoll ist
-  Ich kenne Double-Checked Locking
-  Ich speichere keinen Activity-Context
-  Ich verstehe die Testproblematik

## 14. Offizielle Referenzen

- [Object Declarations](https://kotlinlang.org/docs/object-declarations.html)
- [Kotlin Memory Model](https://kotlinlang.org/docs/memory-management.html)