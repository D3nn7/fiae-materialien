# Annotations

**`@Suppress`, eigene Annotations, Retention & Target

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:
- verstehen, was Annotations in Kotlin sind
- `@Suppress` korrekt und verantwortungsvoll einsetzen
- eigene Annotations definieren
- `@Target` und `@Retention` erklären
- Annotations im Android-Alltag sinnvoll verwenden
- typische Missverständnisse vermeiden

## 1. Was sind Annotations?

Annotations sind **Metadaten für Code**.

Sie:
- verändern nicht direkt das Verhalten
- liefern zusätzliche Informationen
- werden vom Compiler oder Frameworks ausgewertet

Beispiele:
- `@Override`
- `@Deprecated`
- `@Suppress`
- `@JvmStatic`

Offizielle Referenz:  
[https://kotlinlang.org/docs/annotations.html](https://kotlinlang.org/docs/annotations.html)

## 2. `@Suppress` – Warnungen gezielt unterdrücken

### Beispiel

```kotlin
@Suppress("UNCHECKED_CAST")
fun castExample(value: Any): String {
    return value as String
}
```

### Bedeutung

Der Compiler würde normalerweise warnen:
- unsicherer Cast

Mit `@Suppress` sagst du:

> „Ich weiß, was ich hier tue.“


## 3. Wann darf man `@Suppress` verwenden?

### Erlaubt
- bei bewusstem, sicherem Cast
- bei API-Interop
- bei bekannten, dokumentierten Sonderfällen

### Nicht erlaubt
- um Fehler zu „verstecken“
- um Warnungen dauerhaft zu ignorieren
- aus Bequemlichkeit

## 4. Häufige Suppress-Typen

|Name|Bedeutung|
|---|---|
|`UNCHECKED_CAST`|unsicherer Cast|
|`DEPRECATION`|Nutzung veralteter API|
|`UNUSED_PARAMETER`|Parameter wird nicht genutzt|
|`MemberVisibilityCanBePrivate`|Sichtbarkeit könnte reduziert werden|

Beispiel:
```kotlin
@Suppress("UNUSED_PARAMETER")
fun example(unused: String) {
}
```

## 5. Scope von `@Suppress`

Du kannst Annotations setzen auf:

- Klasse
- Funktion
- Property
- Parameter
- Datei

### Beispiel auf Datei-Ebene

```kotlin
@file:Suppress("DEPRECATION")
```

Das wirkt auf die komplette Datei.

## 6. Eigene Annotation erstellen

### Grundsyntax

```kotlin
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
annotation class MyAnnotation
```

Das ist eine eigene Annotation.

## 7. `@Target` – Wo darf die Annotation verwendet werden?

Beispiele:
```kotlin
@Target(
    AnnotationTarget.CLASS,
    AnnotationTarget.FUNCTION,
    AnnotationTarget.FIELD
)
```

Typische Targets:

- `CLASS`
- `FUNCTION`
- `PROPERTY`
- `FIELD`
- `VALUE_PARAMETER`
- `FILE`

Ohne `@Target` kann die Annotation überall genutzt werden.

## 8. `@Retention` – Wie lange bleibt die Annotation erhalten?

### Drei Varianten

|Retention|Bedeutung|
|---|---|
|SOURCE|nur im Quellcode|
|BINARY|im Bytecode|
|RUNTIME|zur Laufzeit verfügbar|

Beispiel:
```kotlin
@Retention(AnnotationRetention.RUNTIME)
```

Wenn du Reflection brauchst → `RUNTIME`.

## 9. Annotation mit Parameter

```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class RequiresPermission(
    val permission: String
)
```

Verwendung:
```kotlin
@RequiresPermission("android.permission.ACCESS_FINE_LOCATION")
fun loadLocation() {
}
```

## 10. Android-relevante Annotations

Typische Android-Anwendungsfälle:
- `@RequiresPermission`
- `@WorkerThread`
- `@MainThread`
- `@CallSuper`
- `@Deprecated`

Beispiel:
```kotlin
@MainThread
fun updateUi() {
}
```

Diese Annotations helfen:
- Code-Verträge klar zu machen
- falsche Nutzung früh zu erkennen

## 11. `@Deprecated` korrekt einsetzen

```kotlin
@Deprecated(
    message = "Use loadNewData() instead",
    replaceWith = ReplaceWith("loadNewData()")
)
fun loadOldData() {
}
```

Vorteile:

- IDE zeigt Warnung
- automatische Quick-Fix-Vorschläge
- saubere API-Weiterentwicklung

## 12. Reflection & Runtime-Annahmen

Wenn eine Annotation `RUNTIME` Retention hat, kannst du sie zur Laufzeit lesen:
```kotlin
val annotations = MyClass::class.annotations
```

Wichtig:
- Reflection kostet Performance
- sparsam einsetzen

## 13. Typische Fehler (bitte vermeiden)

- `@Suppress` dauerhaft auf Datei-Ebene  
- Wichtige Warnungen ignorieren  
- RUNTIME-Retention ohne Bedarf  
- Unklare Annotation-Namen

## 14. Best Practices

- `@Suppress` nur minimal einsetzen
- Eigene Annotations klar benennen
- `Retention` bewusst wählen
- Annotations nicht als Ersatz für Logik missbrauchen
- Dokumentation ergänzen

## 15. Selbst-Check

-  Ich verstehe Target und Retention
-  Ich nutze `@Suppress` bewusst
-  Ich kann eigene Annotations definieren
-  Ich kenne den Unterschied zwischen SOURCE und RUNTIME
-  Ich missbrauche Annotations nicht

## 16. Offizielle Referenzen

- [Kotlin Annotations](https://kotlinlang.org/docs/annotations.html)
- [Annotation Targets](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-annotation-target/)
- [Retention](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-annotation-retention/)