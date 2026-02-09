# Fused Location Provider
**Standortdaten effizient, korrekt und lifecycle-sicher nutzen

## Ziel dieses Dokuments
Nach der Durcharbeitung dieses Dokuments kannst du:
- den Fused Location Provider korrekt einsetzen
- Standortdaten **ressourcenschonend** abrufen
- Updates **lifecycle-sicher** starten und stoppen
- typische Fehler (Battery Drain, Crashes, falsche Genauigkeit) vermeiden
- Location sauber mit ViewModel und State verarbeiten
- verstehen, **wann** und **wie oft** Standortupdates sinnvoll sind

## 1. Was ist der Fused Location Provider?
Der Fused Location Provider ist die **empfohlene Android-API** für Standortbestimmung.

Er kombiniert:
- GPS
- WLAN
- Mobilfunk
- Sensoren

> Android entscheidet selbst, **welche Quelle** aktuell am besten passt.

Offizielle Referenz:  
https://developer.android.com/training/location

## 2. Warum nicht direkt GPS?

Direkter GPS-Zugriff:
- verbraucht viel Akku
- funktioniert schlecht in Gebäuden
- erfordert permanente Sensor-Nutzung

Der Fused Location Provider:
- ist akkusparend
- passt Genauigkeit dynamisch an
- ist stabiler und robuster

## 3. Voraussetzungen

### 3.1 Abhängigkeit einbinden

`implementation("com.google.android.gms:play-services-location:21.0.1")`

> Version zentral verwalten (z. B. Version Catalog).

---

### 3.2 Permission im Manifest
```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

> ACHTUNG: Runtime Permission ist zusätzlich erforderlich (siehe Permissions-Dokument).

---

## 4. Grundprinzip: Standort nur bei Bedarf

### Wichtige Regel
> Standortupdates dürfen **nicht dauerhaft laufen**, wenn sie nicht gebraucht werden.

Typische Fehler:
- Updates in `onCreate()` starten
- nie wieder stoppen
- unnötig hohe Genauigkeit

## 5. LocationRequest korrekt konfigurieren
```kotlin
import com.google.android.gms.location.LocationRequest
import com.google.android.gms.location.Priority

val locationRequest = LocationRequest.Builder(
    Priority.PRIORITY_BALANCED_POWER_ACCURACY,
    10_000L
).setMinUpdateIntervalMillis(5_000L)
 .build()

```

### Erklärung
- `PRIORITY_BALANCED_POWER_ACCURACY` → guter Kompromiss
- 10 Sekunden Intervall → ausreichend für viele Use-Cases
- kein unnötiger GPS-Dauerbetrieb

Offizielle Referenz:  
https://developer.android.com/reference/com/google/android/gms/location/LocationRequest

---

## 6. FusedLocationProviderClient erstellen
```kotlin
import com.google.android.gms.location.LocationServices  

val fusedLocationClient =     LocationServices.getFusedLocationProviderClient(requireContext())
```

> ACHTUNG: Verwende **keinen Activity-Context**, wenn nicht nötig.

## 7. LocationCallback implementieren
```kotlin
import com.google.android.gms.location.LocationCallback
import com.google.android.gms.location.LocationResult

private val locationCallback = object : LocationCallback() {
    override fun onLocationResult(result: LocationResult) {
        val location = result.lastLocation ?: return
        // Standort weiterverarbeiten
    }
}
```
### Wichtig
- nur letzte Location verwenden
- keine UI-Referenzen hier halten

## 8. Standortupdates starten
```kotlin
@SuppressLint("MissingPermission")
private fun startLocationUpdates() {
    fusedLocationClient.requestLocationUpdates(
        locationRequest,
        locationCallback,
        Looper.getMainLooper()
    )
}
```

> ACHTUNG: Diese Methode darf **nur** aufgerufen werden, wenn die Permission gewährt ist.

## 9. Standortupdates stoppen (sehr wichtig!)
```kotlin
private fun stopLocationUpdates() {
    fusedLocationClient.removeLocationUpdates(locationCallback)
}
```

> Jede gestartete Location-Anfrage **muss** wieder gestoppt werden.


## 10. Lifecycle-sichere Einbindung im Fragment

### Empfohlenes Muster
```kotlin
override fun onStart() {
    super.onStart()
    startLocationUpdates()
}

override fun onStop() {
    super.onStop()
    stopLocationUpdates()
}
```

Warum?
- Updates nur, wenn Fragment sichtbar ist
- kein unnötiger Akkuverbrauch
- keine Leaks

## 11. Location & ViewModel

### Best Practice
- Fragment sammelt Location
- ViewModel verarbeitet State
- UI reagiert auf State

`viewModel.onNewLocation(location)`

> ViewModel kennt **keinen** LocationClient.

---

## 12. Häufige Fehler

-  LocationUpdates ohne Permission  
- Updates nie stoppen  
- Zu hohe Genauigkeit ohne Grund  
- LocationCallback hält View-Referenzen  
- Standortabfragen im ViewModel

## 13. Debugging von Location-Problemen

### Typische Checks
- ist Permission wirklich gewährt?
- läuft das Fragment noch?
- kommen Updates rein?
- ist Google Play Services verfügbar?

Beispiel-Log:
```kotlin
Log.d(TAG, "New location: ${location.latitude}, ${location.longitude}")
```

## 15. Selbst-Check
-  Permission korrekt geprüft
-  LocationRequest sinnvoll konfiguriert
-  Updates werden gestoppt
-  Kein Akku-Dauerverbrauch
-  Kein UI-Zugriff im Callback

## 16. Offizielle Quellen
-  [Location Overview](https://developer.android.com/training/location)
-  [Request Location Updates](https://developer.android.com/training/location/request-updates)
- [LocationRequest API](https://developer.android.com/reference/com/google/android/gms/location/LocationRequest)