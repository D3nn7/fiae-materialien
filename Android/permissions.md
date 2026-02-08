# Permissions in Android

Runtime Permissions, Nutzerkommunikation & Sonderfälle

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:

- Runtime Permissions korrekt prüfen und anfordern
- dem Nutzer verständlich erklären, warum eine Permission benötigt wird
- alle typischen Zustände sauber behandeln (erlaubt, abgelehnt, dauerhaft abgelehnt)
- Permission-Ergebnisse lifecycle-sicher verarbeiten
- Location-Permissions korrekt unterscheiden und einsetzen
- häufige Crashes und UX-Fehler vermeiden

## 1. Grundprinzip: Permissions gehören zur Laufzeit

Seit Android 6.0 werden gefährliche Berechtigungen zur Laufzeit vergeben.

Das bedeutet:

- Ein Eintrag im AndroidManifest.xml reicht nicht aus
- Der Nutzer entscheidet aktiv
- Die Entscheidung kann jederzeit geändert werden

Offizielle Referenz:
https://developer.android.com/training/permissions/requesting

## 2. Arten von Permissions (Überblick)

### Normale Permissions

- automatisch gewährt
- keine Nutzerinteraktion
- z. B. Internet

### Gefährliche Permissions (Runtime)

- erfordern Nutzerzustimmung
- z. B. Location, Kamera, Kontakte

> Dieses Dokument fokussiert sich auf Runtime Permissions.

## 3. Der korrekte Ablauf

Die 4 Schritte

1. Prüfen, ob Permission bereits gewährt ist
2. Erklären, warum sie benötigt wird (falls nötig)
3. Anfordern
4. Ergebnis behandeln

> Jeder dieser Schritte ist wichtig. Keinen überspringen.

## 4. Permission im Manifest deklarieren

Beispiel: Location

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

Wichtig:

- Manifest = technische Voraussetzung
- Entscheidung fällt erst zur Laufzeit

## 5. Prüfen, ob Permission gewährt ist

```kotlin
val hasPermission =
    ContextCompat.checkSelfPermission(
        requireContext(),
        Manifest.permission.ACCESS_FINE_LOCATION
    ) == PackageManager.PERMISSION_GRANTED
```

> Nie einfach davon ausgehen, dass eine Permission vorhanden ist.

## 6. Dem Nutzer erklären, warum die Permission nötig ist

### Wann erklären?

Wenn:

- der Nutzer die Permission bereits einmal abgelehnt hat
- der Zweck nicht offensichtlich ist

```kotlin
val shouldExplain =
    shouldShowRequestPermissionRationale(
        Manifest.permission.ACCESS_FINE_LOCATION
    )
```

### UX-Regel

> Erklären vor der Anfrage – nicht danach.

Beispieltext:

> „Wir benötigen deinen Standort, um dir lokale Informationen anzeigen zu können.“

Offizielle Empfehlung:
https://developer.android.com/training/permissions/usage-notes

## 7. Permission anfordern

### Registrierung (einmal)

```kotlin
private val permissionLauncher =
    registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { granted ->
        if (granted) {
            onPermissionGranted()
        } else {
            onPermissionDenied()
        }
    }
```

### Anfrage starten

```kotlin
permissionLauncher.launch(
    Manifest.permission.ACCESS_FINE_LOCATION
)
```

### Warum dieses API?

- lifecycle-sicher
- kein veraltetes `onRequestPermissionsResult`
- empfohlener Standard

Offizielle Referenz:
https://developer.android.com/training/permissions/requesting#request-permission

## 8. Permission-Ergebnis korrekt behandeln

### Fall 1: Permission gewährt

```kotlin
private fun onPermissionGranted() {
    // Feature starten (z. B. Location Updates)
}
```

### Fall 2: Permission abgelehnt (temporär)

- Nutzer hat „Ablehnen“ gewählt
- Anfrage ist später erneut möglich

Reaktion:

- Feature deaktivieren
- klare Info anzeigen

### Fall 3: Permission dauerhaft abgelehnt

Der Nutzer hat:

- „Nicht mehr fragen“ gewählt
  ODER
- die Permission manuell in den Systemeinstellungen entzogen

Erkennung

```kotlin
val permanentlyDenied =
    !shouldShowRequestPermissionRationale(
        Manifest.permission.ACCESS_FINE_LOCATION
    )
```

### Richtige Reaktion

- keine weitere Anfrage starten
- Nutzer informieren
- optional: Weiterleitung zu App-Einstellungen

```kotlin
val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
    data = Uri.fromParts("package", requireContext().packageName, null)
}
startActivity(intent)
```

## 9. Location Permissions – wichtige Unterscheidung

### Fine vs. Coarse Location

| Permission             | Genauigkeit |
| ---------------------- | ----------- |
| ACCESS_COARSE_LOCATION | grob        |
| ACCESS_FINE_LOCATION   | genau       |

Empfehlung:

- zuerst coarse
- nur bei Bedarf fine

Offizielle Referenz:
https://developer.android.com/training/location/permissions

## 10. Typische Fehler

- Permission wird im Code nicht geprüft
- Feature startet trotz fehlender Permission
- Mehrfaches Anfordern ohne Erklärung
- Navigation oder Logik im Permission-Callback ohne Lifecycle-Check
- App ist ohne Permission nicht mehr bedienbar

## 11. Debugging von Permission-Problemen

Prüfen:

- Log-Ausgaben bei jedem Schritt
- wird Callback wirklich aufgerufen?
- ist das Fragment noch aktiv?

Beispiel:

```kotlin
Log.d(TAG, "Permission granted: $granted")
```

## 12. Selbst-Check

- Permission wird vor Nutzung geprüft
- Nutzer bekommt eine verständliche Erklärung
- Dauerhaft abgelehnte Permission wird erkannt
- App bleibt stabil ohne Permission
- Keine Endlosschleifen beim Anfordern

## 13. Offizielle Quellen

- [Request App Permissions](https://developer.android.com/training/permissions/requesting)
- [Permission Usage Notes](https://developer.android.com/training/permissions/usage-notes)
- [Location Permissions](https://developer.android.com/training/location/permissions)
- [Activity Result APIs](https://developer.android.com/training/basics/intents/result)
