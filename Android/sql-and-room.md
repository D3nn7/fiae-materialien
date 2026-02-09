# SQLite & Room in Android

**Lokale Daten persistent, thread-sicher und wartbar speichern**

## Ziel dieses Dokuments
Nach der Durcharbeitung dieses Dokuments kannst du:
- den Unterschied zwischen SQLite und Room erklären
- Room korrekt einrichten und nutzen
- Daten **thread-sicher** speichern und laden
- typische Fehler (Main-Thread-Zugriffe, Leaks, Inkonsistenzen) vermeiden
- Room sauber mit ViewModel & State verwenden
- Migrationen verstehen und korrekt vorbereiten

## 1. Warum lokale Datenhaltung?
Nicht alle Daten gehören:
- ins Netzwerk
- in den Speicher (RAM)

Typische Anwendungsfälle:
- Caching
- Offline-Funktionalität
- Einstellungen
- Verlauf / Historie

> Dafür ist eine lokale Datenbank ideal.

## 2. SQLite vs. Room

### SQLite (direkt)
- sehr low-level
- viel Boilerplate
- fehleranfällig
- keine Compile-Time-Prüfung

### Room (empfohlen)
- Abstraktion über SQLite
- Compile-Time-Checks für SQL
- saubere API
- Lifecycle- & Coroutine-freundlich

> **Room ist der empfohlene Standard für Android.**

Offizielle Referenz:  
[https://developer.android.com/training/data-storage/room](https://developer.android.com/training/data-storage/room?utm_source=chatgpt.com)

## 3. Abhängigkeiten einbinden

```gradle
implementation("androidx.room:room-runtime:2.6.1")
kapt("androidx.room:room-compiler:2.6.1")
implementation("androidx.room:room-ktx:2.6.1")
```

Hinweis:
- Version zentral verwalten
- `room-ktx` für Coroutines-Unterstützung

## 4. Zentrale Bausteine von Room

Room besteht aus drei Kernkomponenten:
1. **Entity** → Tabellenstruktur
2. **DAO** → Datenzugriff
3. **Database** → Datenbank-Instanz

> Alle drei sind zwingend notwendig.

## 5. Entity definieren

### Beispiel: einfache Tabelle
```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "notes")
data class NoteEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val title: String,
    val content: String
)
```

### Wichtige Regeln
- Entities sind **reine Datenklassen**
- keine Logik
- keine Referenzen auf UI oder Context

## 6. DAO – Datenzugriff definieren
```kotlin
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query

@Dao
interface NoteDao {

    @Query("SELECT * FROM notes")
    suspend fun getAllNotes(): List<NoteEntity>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertNote(note: NoteEntity)

    @Query("DELETE FROM notes WHERE id = :id")
    suspend fun deleteNote(id: Long)
}
```

### Warum `suspend`?
- Room erzwingt **keine DB-Zugriffe im Main Thread**
- Coroutine-Support ist Best Practice

## 7. RoomDatabase definieren
```kotlin
import androidx.room.Database
import androidx.room.RoomDatabase

@Database(
    entities = [NoteEntity::class],
    version = 1
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun noteDao(): NoteDao
}
```

### Wichtige Punkte
- `version` ist Pflicht
- jede Schema-Änderung → neue Version

## 8. Datenbank-Instanz erstellen

### Singleton-Pattern (empfohlen)
```kotlin
import android.content.Context
import androidx.room.Room

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

### Warum so?
- thread-sicher
- nur eine Instanz
- kein Memory Leak (Application Context)

## 9. Repository – saubere Trennung
```kotlin
class NoteRepository(
    private val noteDao: NoteDao
) {

    suspend fun loadNotes(): List<NoteEntity> {
        return noteDao.getAllNotes()
    }

    suspend fun addNote(title: String, content: String) {
        noteDao.insertNote(
            NoteEntity(
                title = title,
                content = content
            )
        )
    }
}
```

> ViewModel spricht **nur** mit dem Repository, nie direkt mit dem DAO.

## 10. Room & ViewModel

### Beispiel ViewModel
```kotlin
class NotesViewModel(
    private val repository: NoteRepository
) : ViewModel() {

    fun loadNotes() = liveData {
        emit(repository.loadNotes())
    }
}
```

Oder mit StateFlow:
- Repository → suspend
- ViewModel → State
- UI → Anzeige

## 11. Thread-Safety & Performance

### Do
- DB-Zugriffe nur in Coroutines
- Application Context verwenden
- kleine, gezielte Queries

### Don’t
- Datenbank im Fragment erstellen
- Main-Thread-Zugriffe
- große, unlimitierte Queries

## 12. Migrationen

### Problem
- Schema ändert sich
- App-Update
- alte Daten bleiben bestehen

### Lösung: Migration
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL(
            "ALTER TABLE notes ADD COLUMN timestamp INTEGER NOT NULL DEFAULT 0"
        )
    }
}

```

Einbinden:
```kotlin
Room.databaseBuilder(...)
    .addMigrations(MIGRATION_1_2)
    .build()

```

> Migrationen **niemals ignorieren**.

## 13. Typische Fehler
-  Datenbank im Fragment  
- Zugriff im Main Thread  
- Keine Versionierung  
- `fallbackToDestructiveMigration()` ohne Wissen  
- Logik in Entities

## 14. Debugging von Room

### Hilfreich
- Logcat zeigt SQL-Fehler
- Emulator → Device File Explorer
- DB mit Android Studio Database Inspector ansehen

Offizielle Referenz:  
[https://developer.android.com/studio/inspect/database](https://developer.android.com/studio/inspect/database)

## 15. Selbst-Check
-  Ich nutze Room, nicht SQLite direkt
-  Kein DB-Zugriff im Main Thread
-  Repository kapselt Datenzugriff
-  Migrationen sind vorbereitet
-  Daten bleiben nach Update erhalten

## 16. Offizielle Quellen

- [Room Overview](https://developer.android.com/training/data-storage/room)
- [Defining Entities](https://developer.android.com/training/data-storage/room/defining-data)
- [DAOs](https://developer.android.com/training/data-storage/room/accessing-data)
- [Database Inspector](https://developer.android.com/studio/inspect/database)