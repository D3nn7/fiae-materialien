```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 0 # Include headings up to the specified level
include: 
exclude: 
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```
## Beispielcode
```kotlin
data class Lkw(  
    val kennzeichen: String,  
    val nutzlastTonnen: Int,  
    val istElektro: Boolean,  
    val istVerfügbar: Boolean  
)  
  
fun filtereLkw(  
    lkwListe: List<Lkw>,  
    // Higher-Order Function: nimmt eine Funktion als Parameter  
    kriterium: (Lkw) -> Boolean  
): List<Lkw> {  

    println("filtereLkw - start")  

    val gefiltert = lkwListe.filter(kriterium)  

    println("filtereLkw - end")  
    return gefiltert // ruft die übergebene Funktion für jedes Lkw-Objekt auf  
}  
  
fun main() {  
    val flotte = listOf(  
        Lkw("AB-SP 100", 12, false, true),  
        Lkw("AB-SP 200", 8, true, false),  
        Lkw("AB-SP 300", 15, true, true),  
        Lkw("AB-SP 400", 9, false, true)  
    )  
  
    println("main - start")  
    val schwereElektroLkws = filtereLkw(  
        lkwListe = flotte,  
        kriterium = { lkw ->  
            println("higher-order - body")  
            istElektroUndGroesserAls10Tonnen(lkw = lkw)  
        }  
    )  
    println("main - end")  
    println("Schwere Elektro LKWs: $schwereElektroLkws")  
}  
  
private fun istElektroUndGroesserAls10Tonnen(lkw: Lkw): Boolean {  
    println("higher-order function called")  
    return lkw.istElektro && lkw.nutzlastTonnen > 10  
}
```
## Higher-Order Function

- `filtereLkw()` ist die **Higher-Order Function**, weil sie eine Funktion `(Lkw) -> Boolean` als Parameter entgegennimmt.
- Diese Funktion `kriterium` beschreibt die **Logik zum Filtern** und wird innerhalb von `filtereLkw()` mehrfach aufgerufen.
### Schritt für Schritt Ablauf
1. `main()` startet und ruft **einmal** `filtereLkw()` auf.
2. `filtereLkw()` läuft los, gibt „filtereLkw – start“ aus.
3. Danach ruft `filter()` intern für **jedes Lkw-Objekt** die übergebene Funktion `kriterium` auf.
4. `kriterium` ist in diesem Fall ein **Lambda**, das wiederum `istElektroUndGroesserAls10Tonnen()` aufruft.
5. Die Rückgabewerte (`true` oder `false`) bestimmen, ob das Lkw-Objekt in die Ergebnisliste aufgenommen wird.
6. Am Ende beendet sich `filtereLkw()` und gibt die Liste zurück.
### Vereinfachter Call-Stack
```
main()
 └── filtereLkw(flotte, kriterium)
       └── filter() über Liste
             ├── kriterium(Lkw1)
             │     └── istElektroUndGroesserAls10Tonnen(Lkw1)
             ├── kriterium(Lkw2)
             │     └── istElektroUndGroesserAls10Tonnen(Lkw2)
             └── ...
```
### Konsolen output
```txt
main - start
filtereLkw - start
higher-order - body
higher-order function called
higher-order - body
higher-order function called
higher-order - body
higher-order function called
higher-order - body
higher-order function called
filtereLkw - end
main - end
Schwere Elektro LKWs: [Lkw(kennzeichen=AB-SP 300, nutzlastTonnen=15, istElektro=true, istVerfügbar=true)]
```
### Wichtig zu verstehen
- `filtereLkw()` weiß **nicht selbst**, was "schwere Elektro-LKWs" sind.
- Sie bekommt nur gesagt: „Hier ist eine Funktion `(Lkw) -> Boolean`, benutze die zum Filtern.“
- Die konkrete Logik (`istElektroUndGroesserAls10Tonnen`) kommt von außen und ist austauschbar.
### Praxisbezug
- Einmal nach **Elektro-LKWs** filtern.
- Einmal nach **verfügbaren LKWs**.
- Einmal nach **allen über 10 Tonnen**.

> Mit Higher-Order Functions brauchst du **nur eine generische Funktion** und kannst unterschiedliche Logik flexibel von außen übergeben. 
## Lamdba
Ein **Lambda** ist eine **anonyme Funktion** – also eine Funktion ohne Namen, die man direkt dort hinschreibt, wo man sie braucht.
### Beispiel
Statt:
```kotlin
fun istElektroUndGroesserAls10Tonnen(lkw: Lkw): Boolean {
    return lkw.istElektro && lkw.nutzlastTonnen > 10
}
```
geht auch direkt:
```kotlin
val schwereElektroLkws = filtereLkw(flotte) { lkw ->
    lkw.istElektro && lkw.nutzlastTonnen > 10
}
```
### Aufbau eines Lambdas
Allgemein:
```kotlin
{ parameter1: Typ, parameter2: Typ -> Rückgabeausdruck }
```
Beispiel:
```kotlin
{ x: Int, y: Int -> x + y }
```
→ Funktion, die zwei **Int**s nimmt und ihre Summe zurückgibt.
### Kurzschreibweise mit `it`
Wenn ein Lambda **nur einen Parameter** hat:
```kotlin
// normal
{ lkw: Lkw -> lkw.istVerfügbar }

// Kurzschreibweise
{ it.istVerfügbar }
```
### Praxisbeispiele
```kotlin
// Mit eigener Funktion
val schwereElektroLkws1 = filtereLkw(flotte, ::istElektroUndGroesserAls10Tonnen)

// Mit Lambda (benanntes Argument)
val schwereElektroLkws2 = filtereLkw(flotte) { lkw ->
    lkw.istElektro && lkw.nutzlastTonnen > 10
}

// Mit Lambda (Kurzschreibweise mit it)
val schwereElektroLkws3 = filtereLkw(flotte) { 
    it.istElektro && it.nutzlastTonnen > 10
}
```
Alle drei Varianten machen **dasselbe**.
- **Variante 1** → klassische Funktion, gut bei komplexerer Logik.
- **Variante 2** → Lambda mit Parameternamen, sehr lesbar.
- **Variante 3** → super kompakt, ideal für kurze Bedingungen.
### Warum Lambdas?
- Weniger Boilerplate-Code.
- Logik steht direkt am Ort des Geschehens.
- Macht den Code flexibler und leichter lesbar.
- Besonders nützlich bei Listen-Operationen
- wie `map`, `filter`, `forEach`.

## Zusammengefasst
- **Higher-Order Functions** sind der **Rahmen**.
- **Lambdas** sind die **Bausteine**, die man flexibel hineinsetzen kann.