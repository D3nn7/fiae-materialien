## Gradle in Android

Libraries anbinden, Abhängigkeiten verstehen, Versionskonflikte vermeiden

# Ziel dieses Dokuments

Nach diesem Kapitel kannst du:

- Libraries sauber und reproduzierbar einbinden
- verstehen, woher eine Abhängigkeit kommt und warum sie eingebunden ist
- Versionskonflikte erkennen und beheben
- Build-Probleme systematisch debuggen
- Abhängigkeiten so verwalten, dass dein Projekt wartbar und zukunftssicher bleibt

## 1. Grundverständnis: Was macht Gradle hier eigentlich?

Gradle ist das Build-System von Android.
Für Abhängigkeiten bedeutet das konkret:

- Gradle lädt Libraries aus definierten Repositories
- löst transitive Abhängigkeiten auf
- entscheidet, welche Version tatsächlich verwendet wird
- baut daraus dein APK / AAB

Wenn etwas „komisch“ läuft, liegt es **sehr oft** an Abhängigkeiten.

## 2. Repositories – woher kommen Libraries?

Typischerweise findest du Repositories in der `settings.gradle` oder `build.gradle` (Project):

```gradle
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

**Wichtige Regeln**

- `google()` → AndroidX, Google APIs
- `mavenCentral()` → die meisten Open-Source-Libs
- keine unbekannten Repositories „einfach so“ hinzufügen

> Je weniger Repositories, desto stabiler der Build.

Offizielle Referenz:
https://developer.android.com/build/dependencies

## 3. Abhängigkeiten einbinden

### Klassisches Beispiel (Modul: app)

```gradle
dependencies {
    implementation("androidx.recyclerview:recyclerview:1.3.2")
}
```

Bedeutung:

- `implementation` → intern genutzt, nicht nach außen sichtbar
- Gruppe: androidx.recyclerview
- Artifact: recyclerview
- Version: 1.3.2

## 4. Dependency-Konfigurationen verstehen

Die wichtigsten Konfigurationen
| Konfiguration | Bedeutung |
| ------------- | --------- |
| implementation | Standard, empfohlen |
| api | wird an abhängige Module weitergereicht |
| testImplementation | nur für Unit-Tests |
| androidTestImplementation | nur für Instrumentation-Tests |
| debugImplementation | nur für Debug-Build |

> Verwende immer implementation, außer du hast einen sehr guten Grund, etwas nach außen sichtbar zu machen.

## 5. Versionsverwaltung sauber lösen

### Warum zentrale Versionsverwaltung?

Probleme ohne zentrale Versionen:

- gleiche Library mit unterschiedlichen Versionen
- schwer wartbar
- Upgrades dauern unnötig lange

### Empfohlene Lösung: Version Catalog

Datei: `gradle/libs.versions.toml`

```gradle
[versions]
recyclerview = "1.3.2"
coroutines = "1.8.1"

[libraries]
recyclerview = { group = "androidx.recyclerview", name = "recyclerview", version.ref = "recyclerview" }
coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "coroutines" }
```

### Verwendung im Modul

```gradle
dependencies {
    implementation(libs.recyclerview)
    implementation(libs.coroutines.core)
}
```

Vorteile:

- alle Versionen an einem Ort
- einfache Updates
- weniger Fehlerquellen

Offizielle Referenz:
https://docs.gradle.org/current/userguide/platforms.html#sub:version-catalog

## 6. Transitive Abhängigkeiten verstehen

Wenn du eine Library einbindest, bringt sie oft weitere Libraries mit.

Beispiel:

```
Library A
 ├── Library B
 └── Library C
```

Gradle entscheidet dann:

- welche Version von B/C verwendet wird
- meist: höchste kompatible Version

> Das kann zu Überraschungen führen.

## 7. Versionskonflikte erkennen

Typische Symptome

- Build bricht ohne klaren Grund ab
- Runtime-Crashes (NoSuchMethodError)
- Warnungen wie:
  ```
  Duplicate class found
  ```

Dependency-Tree anzeigen

```sh
./gradlew app:dependencies
```

Oder gezielt:

```sh
./gradlew app:dependencyInsight --dependency recyclerview
```

Damit siehst du:

- wer die Library einbindet
- welche Version tatsächlich verwendet wird

## 8. Konflikte gezielt lösen

### Variante 1: Version erzwingen

```gradle
dependencies {
    implementation("androidx.recyclerview:recyclerview") {
        version {
            strictly("1.3.2")
        }
    }
}
```

### Variante 2: Transitive Abhängigkeit ausschließen

```gradle
implementation("example:some-lib:1.0.0") {
    exclude(group = "androidx.recyclerview", module = "recyclerview")
}
```

> Nur tun, wenn du genau weißt, was du tust.

## 9. Debugging von Gradle-Problemen

Typische Fragen, die du dir stellen solltest

- Welche Version wird wirklich verwendet?
- er zieht diese Dependency rein?
- Gibt es doppelte Klassen?
- Ist das Repository korrekt?

### Sehr hilfreich

```gradle
./gradlew build --info
./gradlew build --scan
```

## 10. Performance & Best Practices

**Do**

- Version Catalog nutzen
- implementation bevorzugen
- Abhängigkeiten regelmäßig aufräumen
- nur Libraries einbinden, die du wirklich brauchst

**Don’t**

- Versionsnummern überall im Projekt verstreuen
- alte, ungewartete Libraries einbinden
- Build-Warnungen ignorieren

## 12. Selbst-Check

- Ich weiß, wo Abhängigkeiten definiert sind
- Ich verstehe implementation vs. api
- Ich kann den Dependency-Tree lesen
- Ich kann Versionskonflikte lösen
- Mein Build ist reproduzierbar

## 13. Offizielle Quellen

- [Abhängigkeiten verwalten](https://developer.android.com/build/dependencies)
- [Gradle Version Catalogs](https://docs.gradle.org/current/userguide/platforms.html#sub:version-catalog)
- [Dependency Insight](https://docs.gradle.org/current/userguide/viewing_debugging_dependencies.html)
