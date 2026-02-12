# Grundlagen: Kotlin-Setup und erste Schritte

## Ziel

- Eine lauffähige Kotlin-Entwicklungsumgebung einrichten
- Den Unterschied zwischen Kotlin/JVM, Kotlin Script und Android-Kotlin einordnen
- Ein erstes Kotlin-Programm schreiben, ausführen und strukturiert ablegen
- Grundbegriffe (Compiler, Runtime, JDK, Build-Tool) sicher verwenden

## Voraussetzungen

- Zugriff auf einen Entwicklungsrechner mit Internet
- Grundverständnis von Dateien/Ordnern und Kommandozeile (Terminal/PowerShell)

## Begriffe und Einordnung

### Kotlin-Plattformen
Kotlin ist eine Sprache, die auf verschiedene Zielplattformen kompiliert werden kann:

- **Kotlin/JVM**: Kompiliert zu JVM-Bytecode und läuft auf der Java Virtual Machine.
  - Typisch für Backend, Desktop, Tools, erste Lernprojekte.
- **Kotlin/Android**: Ebenfalls JVM-basiert, aber mit Android-Tooling, Build-System und Framework.
- **Kotlin/Native**: Kompiliert zu nativen Binaries (z. B. iOS).
- **Kotlin/JS**: Kompiliert zu JavaScript.

Für den Einstieg ist **Kotlin/JVM** ideal: schnell aufgesetzt, minimaler Overhead, unmittelbares Feedback.

### JDK, Compiler, Runtime

- **JDK (Java Development Kit)**: Enthält den Java-Compiler, die JVM und Tools. Kotlin nutzt das JDK, um auf der JVM zu laufen.
- **Kotlin-Compiler**: Übersetzt Kotlin-Quelltext (`.kt`) in Bytecode (JVM) oder andere Zielartefakte.
- **Runtime**: Umgebung, in der das Programm ausgeführt wird (bei Kotlin/JVM: JVM).
- **Build-Tool**: Automatisiert Kompilieren, Testen, Abhängigkeiten, Ausführen. Im Kotlin-Umfeld meist **Gradle**.

## Tooling-Entscheidung für Lernprojekte

Empfohlen für reproduzierbares Arbeiten und spätere Android-Nähe:

- **IntelliJ IDEA Community** oder **Android Studio**
- **Gradle** als Build-Tool
- **JDK 17** (stabil, weit verbreitet, kompatibel mit aktuellen Toolchains)

> Hinweis: Auch wenn Android später ins Spiel kommt, ist JDK 17 heute der gängige Standard für moderne Kotlin/Gradle-Projekte.

## Setup Schritt für Schritt (Kotlin/JVM mit Gradle)

### 1) IDE installieren

Option A (empfohlen für Kotlin/JVM):
- IntelliJ IDEA Community

Option B (wenn ohnehin Android-Entwicklung geplant ist):
- Android Studio (enthält IntelliJ-Basis)

Wichtig:
- IDE nach Installation einmal starten, Standardpfade akzeptieren.
- Sicherstellen, dass die IDE ein JDK erkennt oder installieren kann.

### 2) JDK installieren/konfigurieren

Ziel:
- JDK 17 lokal verfügbar
- IDE nutzt JDK 17 für Projekte

Prüfen über Terminal:

```bash
java -version
javac -version
```

Erwartung:

Ausgabe enthält `17` (oder kompatibles LTS-JDK, falls im Team anders festgelegt)

### 3) Neues Projekt anlegen (Gradle + Kotlin)
In IntelliJ/Android Studio:
- New Project
- Gradle
- Kotlin/JVM
- Project SDK: JDK 17
- Projektname: z. B. kotlin-grundlagen
- Projekt erstellen

Typische Projektstruktur (vereinfacht):
```
kotlin-grundlagen/
  build.gradle.kts
  settings.gradle.kts
  src/
    main/
      kotlin/
    test/
      kotlin/
```
Diese Struktur ist wichtig, weil sie Konventionen folgt, die in vielen Projekten und Toolchains genutzt werden.

## Erstes Programm: Hello World

Lege eine Datei an:

`src/main/kotlin/Main.kt`

```kotlin
fun main() {
    println("Hello, Kotlin!")
}
```

Ausführen:
- In der IDE: Run-Konfiguration für `main` nutzen
- Oder über Gradle: `./gradlew run` (falls `application`-Plugin konfiguriert ist)

## Gradle-Grundkonfiguration (minimal)

Viele Projektgeneratoren richten Gradle bereits korrekt ein. Falls nicht, ist ein minimales Setup hilfreich.

### build.gradle.kts (Beispiel)
```gradle
plugins {
    kotlin("jvm") version "2.0.0"
    application
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation(kotlin("test"))
}

application {
    mainClass.set("MainKt")
}

kotlin {
    jvmToolchain(17)
}

```

Erläuterungen:

- `kotlin("jvm")`: Kotlin-Plugin für die JVM
- `application`: Ermöglicht `gradlew run` und definiert `mainClass`
- `repositories`: Woher Abhängigkeiten geladen werden (hier: Maven Central)
- `kotlin { jvmToolchain(17) }`: Erzwingt konsistente JDK-Version beim Build

> Wichtig: `MainKt` ist der von Kotlin generierte Klassenname für eine Datei `Main.kt` mit Top-Level `main`.

## Kotlin-Grundsyntax: Orientierung ohne Detailtiefe

### Funktionen

- Funktionen werden mit `fun` definiert.
- Rückgabetypen werden hinter dem Parameterblock angegeben.

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

Kurzform (Expression Body):

```kotlin
fun add(a: Int, b: Int): Int = a + b
```

### Variablen

- `val`: unveränderlich (bevorzugt)
- `var`: veränderlich (sparsam einsetzen)

```kotlin
val name: String = "Alex"
var counter: Int = 0
counter += 1
```

Typinferenz ist üblich:

```kotlin
val age = 21
```

### Strings

```kotlin
val n = "Alex"
println("Hallo, $n")
println("2 + 3 = ${2 + 3}")
```
## Best Practices für den Einstieg

- **Reproduzierbarkeit vor Komfort**: Projekte immer mit Gradle anlegen, nicht nur einzelne Skripte.
- **Kleine Schritte, schnelle Rückkopplung**: Nach jeder Änderung kompilieren/ausführen.
- **Konventionen nutzen**: Projektstruktur und Namensgebung wie von Gradle/IDE vorgeschlagen.
- **`val` bevorzugen**: Unveränderlichkeit reduziert Fehler und erleichtert Verständnis.
- **Saubere Datei- und Paketstruktur**: Früh damit anfangen, auch wenn das Projekt klein ist.

## Häufige Probleme und schnelle Checks

### IDE findet JDK nicht

- Project SDK prüfen (Project Structure)
- `JAVA_HOME` korrekt setzen (je nach Betriebssystem)
- Terminal-Check: `java -version`

### Gradle Build schlägt fehl

- Gradle Sync ausführen
- Internetzugang / Proxy prüfen (Maven Central erreichbar)
- JDK-Version prüfen: `kotlin { jvmToolchain(17) }` und IDE-JDK müssen zusammenpassen

### `MainKt` wird nicht gefunden

- Datei heißt `Main.kt` und enthält `fun main()`
- `application { mainClass.set("MainKt") }` ist korrekt
- Gradle neu synchronisieren
## Definition of Done

- Projekt lässt sich in der IDE öffnen und ohne Fehler bauen
- `Hello, Kotlin!` wird ausgegeben
- Gradle-Sync läuft sauber durch
- JDK 17 ist als Toolchain konfiguriert