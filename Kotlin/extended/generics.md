# Generics

**Typsicherheit, Wiederverwendbarkeit & saubere API-Designs**

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:
- verstehen, was Generics sind und warum sie wichtig sind
- typsichere, wiederverwendbare APIs schreiben
- `in` und `out` korrekt einsetzen (Varianz)
- typische Fehler mit Generics vermeiden
- Generics sinnvoll in Android-Architekturen einsetzen

## 1. Warum Generics?

Ohne Generics würdest du:

- mit `Any` arbeiten
- ständig casten müssen
- Laufzeitfehler riskieren

Mit Generics bekommst du:

- Compile-Time-Typsicherheit
- saubere APIs
- weniger Casting
- weniger Bugs

## 2. Einfaches Beispiel ohne Generics (problematisch)

```kotlin
class Box {
    var value: Any? = null
}
```

Problem:

- Typ ist nicht bekannt
- Cast notwendig
- ClassCastException möglich

## 3. Dasselbe mit Generics

```kotlin
class Box<T> {
    var value: T? = null
}
```

Verwendung:

```kotlin
val stringBox = Box<String>()
stringBox.value = "Hallo"
```

Vorteile:

- Typ ist klar definiert
- kein Casting
- keine Laufzeitüberraschungen

## 4. Generics in Funktionen

```kotlin
fun <T> printItem(item: T) {
    println(item)
}
```

Wichtig:

- `<T>` steht vor dem Funktionsnamen
- T ist ein Platzhalter für einen Typ

## 5. Generics mit Constraints

Manchmal darf ein Typ nur bestimmte Eigenschaften haben.

```kotlin
fun <T : Number> sum(a: T, b: T): Double {
    return a.toDouble() + b.toDouble()
}
```

Hier gilt:

- T muss von `Number` erben
- Compiler verhindert falsche Nutzung

## 6. Varianz: `in` und `out` verstehen (sehr wichtig)

### Problemstellung

Stell dir vor:
```kotlin
List<String>
```

Ist das eine:

```kotlin
List<Any>
```

Antwort: **Nein.**

Warum?

- Kotlin Generics sind standardmäßig invariant.

## 7. `out` – Producer

```kotlin
interface Producer<out T> {
    fun produce(): T
}
```

Bedeutung:

- T wird nur ausgegeben
- keine Eingabe von T erlaubt

> `out` = der Typ kommt **heraus**

Beispiel:
```kotlin
List<out T>
```

Deshalb ist `List<String>` als `List<Any>` lesbar.

## 8. `in` – Consumer

```kotlin
interface Consumer<in T> {
    fun consume(item: T)
}
```

Bedeutung:

- T wird nur konsumiert    
- nichts wird zurückgegeben

> `in` = der Typ geht **hinein**

## 9. Wann brauchst du Varianz?

Typische Android-Beispiele:
- Adapter-Interfaces
- Repository-Abstraktionen
- Event-Systeme

Wenn du:
- nur Daten zurückgibst → `out`
- nur Daten entgegennimmst → `in`


## 10. Reified Generics (Inline-Funktionen)

Normalerweise sind Generics zur Laufzeit nicht vorhanden (Type Erasure).

Mit `reified` kannst du den Typ zur Laufzeit nutzen.

```kotlin
inline fun <reified T> isType(value: Any): Boolean {
    return value is T
}
```

Wichtig:

- nur in `inline` Funktionen möglich
- häufig genutzt bei JSON oder Reflection

## 11. Typische Fehler (bitte vermeiden)

- Unnötige Generics  
- `Any` statt generischer Typen  
- Falsche Varianz  
- Unklare Typbezeichnungen

## 12. Generics im Android-Kontext

### Beispiel: Result Wrapper

```kotlin
sealed class Result<T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error<T>(val message: String) : Result<T>()
}
```

Verwendung:

```kotlin
Result<String>
Result<List<User>>
```

Vorteile:

- flexibel
- wiederverwendbar
- typsicher

## 13. Thread-Safety & Generics

Generics machen Code nicht automatisch thread-sicher.

Wenn du:
```kotlin
class Repository<T>
```

verwendest, musst du trotzdem:
- Nebenläufigkeit beachten
- Mutable State vermeiden
- Synchronisierung korrekt einsetzen

## 14. Selbst-Check

-  Ich verstehe Type Erasure
-  Ich kann Generics in Klassen & Funktionen nutzen
-  Ich weiß, wann ich `in` oder `out` verwende
-  Ich vermeide unnötige Typunsicherheit

## 15. Offizielle Referenzen

- [Kotlin Generics](https://kotlinlang.org/docs/generics.html)
- [Variance](https://kotlinlang.org/docs/generics.html#variance)
- [Reified Type Parameters](https://kotlinlang.org/docs/inline-functions.html#reified-type-parameters)