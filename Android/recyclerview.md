# RecyclerView & ViewHolder

**Performante Listen und sauberes Recycling**

---

## Ziel dieses Dokuments

Nach der Durcharbeitung dieses Dokuments kannst du:
- RecyclerView korrekt einsetzen
- verstehen, **warum ViewHolder existieren**
- Performante Listen ohne Ruckeln bauen
- typische Fehler (falsches Recycling, falsche Listener, Leaks) vermeiden
- Updates effizient durchführen
- RecyclerView sauber mit ViewModel & State verwenden

## 1. Warum RecyclerView?

RecyclerView ist der **Standard** für Listen in Android, weil er:
- Views **wiederverwendet** (Recycling)
- nur das rendert, was sichtbar ist
- große Datenmengen performant darstellen kann
- flexibel (Listen, Grids, horizontale Listen)

Offizielle Referenz:  
https://developer.android.com/guide/topics/ui/layout/recyclerview

## 2. Das Grundprinzip: Recycling verstehen

### Wichtiges Konzept

> **Eine View gehört nicht zu einem Datenobjekt.**  
> Sie wird immer wieder neu gebunden.

Das bedeutet:
- dieselbe View zeigt nacheinander verschiedene Daten
- **onBindViewHolder** wird sehr oft aufgerufen
- falscher Code hier führt sofort zu Bugs oder Ruckeln

## 3. Die wichtigsten Bausteine

### RecyclerView
- Container für die Liste
### Adapter
- verbindet Daten mit Views
### ViewHolder
- hält Referenzen auf Views
- vermeidet teure `findViewById`-Aufrufe

## 4. Einfaches Datenmodell

```kotlin
data class ListItem(
	val id: Long,
	val title: String,
	val subtitle: String
)
```

## 5. Layout für ein Listenelement

```xml
<!-- item_list.xml --> 
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android" 
	android:orientation="vertical"
	android:padding="16dp"
	android:layout_width="match_parent"
	android:layout_height="wrap_content">      
	<TextView
		android:id="@+id/titleText"
		android:textSize="16sp"
		android:layout_width="wrap_content"         
		android:layout_height="wrap_content"/>      
	<TextView
		android:id="@+id/subtitleText"
		android:textSize="14sp"
		android:layout_width="wrap_content"         
		android:layout_height="wrap_content"/>
</LinearLayout>
```

## 6. ViewHolder korrekt implementieren

```kotlin
import android.view.View
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView 

class ListItemViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) { 
     private val titleText: TextView = itemView.findViewById(R.id.titleText)     
     private val subtitleText: TextView = itemView.findViewById(R.id.subtitleText)      
     fun bind(item: ListItem) {
	     titleText.text = item.title
	     subtitleText.text = item.subtitle
     }
}
```
### Wichtig

- **keine Logik** im ViewHolder
- keine Listener mit außenliegenden Referenzen
- nur binden, was angezeigt wird

## 7. Adapter – sauber & performant

### Adapter mit `ListAdapter`
```kotlin
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter

class ListItemAdapter :
    ListAdapter<ListItem, ListItemViewHolder>(DiffCallback) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ListItemViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_list, parent, false)
        return ListItemViewHolder(view)
    }

    override fun onBindViewHolder(holder: ListItemViewHolder, position: Int) {
        holder.bind(getItem(position))
    }

    companion object {
        private val DiffCallback = object : DiffUtil.ItemCallback<ListItem>() {
            override fun areItemsTheSame(old: ListItem, new: ListItem): Boolean {
                return old.id == new.id
            }

            override fun areContentsTheSame(old: ListItem, new: ListItem): Boolean {
                return old == new
            }
        }
    }
}
```

---

## 8. Warum `ListAdapter` + `DiffUtil`?

### Vorteile
- nur **geänderte** Items werden neu gebunden
- Animationen automatisch
- kein manuelles `notifyDataSetChanged()`

> `notifyDataSetChanged()` ist fast immer falsch.

Offizielle Referenz:  
[https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil)

## 9. RecyclerView im Fragment
```kotlin
class ListFragment : Fragment(R.layout.fragment_list) {

    private val adapter = ListItemAdapter()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val recyclerView = view.findViewById<RecyclerView>(R.id.recyclerView)

        recyclerView.adapter = adapter
        recyclerView.setHasFixedSize(true)

        // Beispiel: Daten setzen
        adapter.submitList(
            listOf(
                ListItem(1, "Titel 1", "Untertitel"),
                ListItem(2, "Titel 2", "Untertitel")
            )
        )
    }
}
```
### Warum `setHasFixedSize(true)`?
- RecyclerView weiß, dass sich die Größe nicht ändert
- bessere Performance

## 10. Klicks korrekt behandeln

### Richtiger Ansatz
```kotlin
class ListItemAdapter(
    private val onItemClick: (ListItem) -> Unit
) : ListAdapter<ListItem, ListItemViewHolder>(DiffCallback) {

    override fun onBindViewHolder(holder: ListItemViewHolder, position: Int) {
        val item = getItem(position)
        holder.bind(item)
        holder.itemView.setOnClickListener {
            onItemClick(item)
        }
    }
}
```

### Warum so?
- keine ViewHolder-Referenzen nach außen
- kein Memory Leak
- Klick bezieht sich auf **aktuelles Item**

## 11. Typische Fehler (bitte vermeiden)
- Logik in `onBindViewHolder`  
- Netzwerk-/DB-Zugriffe im Adapter  
- `notifyDataSetChanged()`  
- Position speichern (`position` ist nicht stabil!)  
- Listener mit Fragment-/Activity-Referenzen

## 12. RecyclerView & State

### Best Practice
- ViewModel liefert **Liste als State**
- Fragment sammelt State
- Adapter bekommt **neue Liste**
`adapter.submitList(state.items)`

> RecyclerView bleibt dumm, ViewModel denkt.

## 13. Debugging & Performance-Tipps
- Logs **nicht** in `onBindViewHolder`
- `LayoutInspector` bei UI-Problemen nutzen
- Ruckeln = meist zu viel Arbeit im Bind
- Große Bilder → vorher skalieren

## 14. Selbst-Check

-  Ich verstehe Recycling
-  Ich nutze ListAdapter
-  Kein `notifyDataSetChanged()`
-  Keine Logik im Adapter
-  Performance ist stabil

## 15. Offizielle Quellen
- [RecyclerView Overview](https://developer.android.com/guide/topics/ui/layout/recyclerview)
- [ViewHolder Pattern](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.ViewHolder?utm_source=chatgpt.com)
- [DiffUtil](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil)