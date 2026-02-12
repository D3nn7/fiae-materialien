# Kotlin Basics

Diese Sammlung enthält strukturierte, fachlich fundierte Lernmaterialien für die Ausbildung zum Fachinformatiker für Anwendungsentwicklung mit Schwerpunkt Kotlin und saubere Softwareentwicklung.

Die Inhalte sind modular aufgebaut und folgen einem klaren didaktischen Fortschritt: von Sprachgrundlagen über objektorientierte Modellierung bis hin zu Architektur, Testing und professioneller Projektorganisation.

---

# Inhaltsübersicht

## 0 – Grundlagen: Setup und erste Schritte

- Entwicklungsumgebung (Kotlin/JVM, Gradle)
- Projektstruktur
- Erstes Programm
- Build-Konfiguration

## 1 – Datentypen, Operatoren und Null-Safety

- Primitive Datentypen
- Kontrollstrukturen
- Ausdrucksorientierung in Kotlin
- Nullable vs. Non-Nullable
- Elvis-Operator, Safe Calls

## 2 – Funktionen und funktionale Grundlagen

- Funktionsdefinitionen
- Default-Parameter
- Named Arguments
- Higher-Order Functions
- Lambdas
- Erweiterungsfunktionen

## 3 – Klassen, Konstruktoren und Modellierung

- Primär- und Sekundärkonstruktoren
- `init`-Blöcke
- Kapselung
- Data Classes
- Immutable Design
- Fachliche Objektmodellierung

## 4 – Vererbung, Interfaces und Polymorphie

- `open`, `override`
- Abstrakte Klassen
- Interfaces
- Liskov Substitution
- Sealed Classes
- Komposition statt Vererbung

## 5 – Collections und Generics

- List, Set, Map
- Mutable vs. Immutable Collections
- Funktionale Transformationen (`map`, `filter`, `fold`)
- Generics
- Sequences

## 6 – Fehlerbehandlung

- Exceptions
- `try-catch`
- Eigene Exceptions
- `require`, `check`
- Result-Typen
- Fehlerarchitektur

## 7 – Architekturprinzipien und SOLID

- Single Responsibility
- Open-Closed
- Liskov Substitution
- Interface Segregation
- Dependency Inversion
- Schichtenarchitektur
- Dependency Injection

## 8 – Testen und Qualitätssicherung

- Unit Tests mit JUnit
- AAA-Prinzip
- Exception-Tests
- Fake vs. Mock
- Testgetriebenes Denken

## 9 – Coroutines und asynchrone Programmierung

- `suspend`
- Structured Concurrency
- `launch`, `async`
- Dispatcher
- Flow
- Cancellation
- Testen von Coroutines

## 10 – Clean Code

- Naming-Konventionen
- Kleine Funktionen
- Guard Clauses
- DRY, KISS, YAGNI
- Refactoring-Techniken
- Code Smells

## 11 – Projektstruktur und Build-Organisation

- Gradle-Konfiguration
- Version Catalog
- Mehrmodul-Projekte
- Paketstruktur
- Repository-Standards
- Commit-Konventionen

# Qualitätsanspruch

Der Code in diesem Repository soll:

- Keine unnötige Mutabilität enthalten
- Keine magischen Zahlen enthalten
- Klar benannte Klassen und Funktionen verwenden
- SOLID-konform strukturiert sein
- Testbar sein
- Keine toten oder auskommentierten Codeblöcke enthalten

# Weiterführende Entwicklung

Nach Abschluss aller Kapitel:

- Eigenständiges Konsolenprojekt entwickeln
- Mehrmodul-Projekt umsetzen
- Eigene Architekturentscheidung dokumentieren
- Projekt mit Tests absichern
- Code-Review simulieren
