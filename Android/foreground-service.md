# Foreground Service in Android

**Langlaufende Aufgaben & Standort im Hintergrund korrekt umsetzen

## Ziel dieses Dokuments
Nach der Durcharbeitung dieses Dokuments kannst du:
- erklären, **wann** ein Foreground Service notwendig ist
- einen Foreground Service korrekt starten und stoppen
- Standortupdates im Hintergrund **regelkonform** umsetzen
- Notifications korrekt und nutzerfreundlich anzeigen
- typische Fehler (Crashes, App-Abstürze, System-Kills) vermeiden
- verstehen, wie Foreground Services mit Lifecycle & Permissions zusammenspielen

## 1. Was ist ein Foreground Service?
Ein Foreground Service ist ein Service, der:
- **für den Nutzer sichtbar** ist
- eine **dauerhafte Notification** anzeigt
- vom System **nicht einfach beendet** wird
- für **laufende, wichtige Aufgaben** gedacht ist

Typische Anwendungsfälle:
- Standortverfolgung
- Navigation
- Fitness-Tracking
- aktive Medienwiedergabe

Offizielle Referenz:  
https://developer.android.com/guide/components/foreground-services

## 2. Wann brauchst du einen Foreground Service?

### Wichtige Regel
> **Sobald eine Aufgabe im Hintergrund läuft und für den Nutzer relevant ist, brauchst du einen Foreground Service.**

### Beispiel: Location
- Standort im Vordergrund → kein Service nötig
- Standort im Hintergrund → **Foreground Service Pflicht**

Ohne Foreground Service
- System stoppt Updates
- App wird beendet
- Verhalten ist unzuverlässig

## 3. Voraussetzungen

### 3.1 Permission im Manifest
```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

Zusätzlich:
- Runtime Permission für Location
- ggf. `FOREGROUND_SERVICE_LOCATION` (ab neueren Android-Versionen)

Offizielle Referenz:  
https://developer.android.com/guide/components/foreground-services#permissions

### 3.2 Service im Manifest deklarieren
```xml
<service
	android:name=".location.LocationForegroundService"    
	android:exported="false"
	android:foregroundServiceType="location"
/>
```

### Wichtig
- `foregroundServiceType` korrekt setzen
- falscher Typ → System-Fehler

## 4. Grundprinzip eines Foreground Service

### Ablauf
1. Service starten
2. **Innerhalb von 5 Sekunden** `startForeground()` aufrufen
3. Notification anzeigen
4. Aufgabe ausführen
5. Service sauber stoppen

> Schritt 2 ist **zwingend**.

---

## 5. Notification erstellen

### Notification Channel (einmalig)
```kotlin
private const val CHANNEL_ID = "location_service"

private fun createNotificationChannel(context: Context) {
    val channel = NotificationChannel(
        CHANNEL_ID,
        "Location Tracking",
        NotificationManager.IMPORTANCE_LOW
    )

    val manager =
        context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

    manager.createNotificationChannel(channel)
}
```

### Notification bauen
```kotlin
private fun buildNotification(context: Context): Notification {
    return NotificationCompat.Builder(context, CHANNEL_ID)
        .setContentTitle("Standort aktiv")
        .setContentText("Dein Standort wird im Hintergrund verwendet")
        .setSmallIcon(R.drawable.ic_location)
        .setOngoing(true)
        .build()
}
```

### UX-Regel
- klar
- ehrlich
- nicht irreführend

## 6. Foreground Service implementieren
```kotlin
class LocationForegroundService : Service() {

    override fun onCreate() {
        super.onCreate()
        createNotificationChannel(this)

        val notification = buildNotification(this)
        startForeground(1, notification)
    }

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {
        startLocationUpdates()
        return START_STICKY
    }

    override fun onDestroy() {
        stopLocationUpdates()
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```
### Erklärung
- `startForeground()` → Service bleibt aktiv
- `START_STICKY` → Service wird ggf. neu gestartet
- `onDestroy()` → Aufräumen nicht vergessen

## 7. Location im Foreground Service

### Grundregel
- Location-Code **lebt im Service**
- UI kennt den Service **nicht direkt**
- Service kennt keine Views
### Start der Updates
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

### Stoppen der Updates
```kotlin
private fun stopLocationUpdates() {
    fusedLocationClient.removeLocationUpdates(locationCallback)
}
```

## 8. Service starten und stoppen

### Service starten (z. B. aus Fragment)
```kotlin
val intent = Intent(requireContext(), LocationForegroundService::class.java)
ContextCompat.startForegroundService(requireContext(), intent)
```

### Service stoppen
```kotlin
requireContext().stopService(
    Intent(requireContext(), LocationForegroundService::class.java)
)
```

## 9. Typische Fehler (bitte vermeiden)
-  `startForeground()` nicht aufgerufen  
- Keine Notification angezeigt  
- Falscher `foregroundServiceType`  
- Location ohne Foreground Service im Hintergrund  
- Service läuft endlos ohne Nutzerwissen

## 10. Debugging von Foreground Services

### Typische Probleme
- Service startet, wird sofort beendet
- App crasht nach wenigen Sekunden
- Keine Location-Updates

### Checks
- Log in `onCreate`, `onStartCommand`, `onDestroy`
- Wird `startForeground()` rechtzeitig aufgerufen?
- Ist die Permission wirklich vorhanden?

Beispiel:
```kotlin
`Log.d(TAG, "Foreground service started")
```

## 11. Akku & Verantwortung
> Ein Foreground Service ist ein **Versprechen an den Nutzer**.

Deshalb:
- nur starten, wenn nötig
- sofort stoppen, wenn Aufgabe erledigt ist
- Notification nicht verstecken

## 12. Selbst-Check
-  Foreground Service nur bei Bedarf
-  Notification sichtbar & korrekt
-  Location nur mit Service im Hintergrund
-  Service wird sauber gestoppt
-  Kein UI-Code im Service

## 13. Offizielle Quellen
- [Foreground Services Overview](https://developer.android.com/guide/components/foreground-services)
- [Foreground Service Permissions](https://developer.android.com/guide/components/foreground-services#permissions)
- [Location in Foreground Services](https://developer.android.com/training/location/background)