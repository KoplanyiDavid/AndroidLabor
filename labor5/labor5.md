# Labor 5 - Perzisztencia

## Felkészülés a laborra

A labor célja a relációs adatbáziskezelés bemutatása Android platformon. A feladatok során a korábban elkészített Todo lista alkalmazást fejlesztjük tovább. 

Az alkalmazás SQLite adatbázisban fogja tárolni a Todo elemeket, így azok nem fognak elveszni amikor a felhasználó elnavigál, vagy elforgatja a készüléket.

## Kiinduló projekt

Töltsük le a kiinduló projektet:

[Kiinduló project](./assets/Todo_kiindulo.zip)

Ez a Todo alkalmazás még csak memóriában tárolja a futása során létrehozott Todo elemeket.

Tömörítsük ki a projektet tartalmazó mappát, indítsuk el az Android Studio-t, majd nyissuk meg a projektet.

Nézzük meg a `model` package-ben lévő `Todo` osztályt, amit a korábbi laboron már létrehoztunk. Ehhez hozzáadtunk egy `id` nevű property-t, ami az adatbázisban fogja egyedien azonosítani a példányokat. Ennek a property-nek adtunk egy [default értéket](https://kotlinlang.org/docs/reference/functions.html#default-arguments), hogy az explicit megadása nélkül is tudjunk `Todo` példányokat létrehozni.

## Adattárolás SQLite adatbázisban

Célunk, hogy a a Todo objektumok memóriában tárolása helyett az alkalmazás egy SQLite adatbázisban perzisztensen mentse őket. Tehát az, hogy ne veszítsük el a felvett Todo-kat az alkalmazás bezárásakor. 

A `SimpleItemRecyclerViewAdapter` a módosítások után nem a memóriában tárolt Todo listát fogja megjeleníteni, hanem majd az adatbázisból olvassa ki és köti össze azt a `RecyclerView`-val.

A feladat megvalósítása három részből áll:

1. Todo objektumok perzisztens tárolásának megvalósítása adatbázissal
2. A memóriában tárolt listával dolgozó adapter átalakítása adatbázissal (`Cursor`-ral) működővé
3. Workflow átírása

### Todo-k tárolása adatbázisban

Az SQLite adatbáziskezelő használatához segítséget nyújt a platform, mégpedig az `SQLiteOpenHelper` osztállyal. Ebből származtatva olyan osztályt hozhatunk létre, ami referenciát szolgáltat az általunk használt teljes adatbázisra. Így több entitás osztály és adatbázis tábla esetén is elég egyetlen ilyen segédosztályt implementálni.

Hozzunk létre a projektben a `hu.bme.aut.android.todo` package-en belül egy új package-et `database` néven. Ezen belül hozzunk létre egy új osztályt `DatabaseHelper` néven, ami az `SQLiteOpenHelper` osztályból származik. 

A `DatabaseHelper` osztály:

```kotlin
class DatabaseHelper(
        context: Context,
        name: String,
        factory: SQLiteDatabase.CursorFactory,
        version: Int
) : SQLiteOpenHelper(context, name, factory, version) {

    override fun onCreate(db: SQLiteDatabase) {
        // TODO
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        // TODO
    }

}
```

Az `SQLiteOpenHelper` osztályból történő származtatás az ősosztály konstruktorának meghívásán kívül két metódus kötelező felüldefiniálását írja elő. Az `onCreate()`-ben kell futtatni a séma létrehozó SQL szkriptet, míg az `onUpgrade()`-ben kezelhetők le az adatbázis verzióváltásával kapcsolatos feladatok. Legegyszerűbb esetben itt törölhető, majd újra létrehozható az egész séma. Ekkor viszont a felhasználó elveszíti a tárolt adatait, ami általában nem elfogadható felhasználói élmény.

Az adatbáziskezelés során sok konstans jellegű változóval kell dolgoznunk, mint például a táblákban lévő oszlopok nevei, táblák neve, adatbázis fájl neve, séma létrehozó és törlő szkiptek, stb. Ezeket érdemes egy közös helyen tárolni, így szerkesztéskor vagy új entitás bevezetésekor nem kell a forrásfájlok között ugrálni, valamint egyszerűbb a teljes adatbázist létrehozó és törlő szkripteket generálni. Hozzunk létre egy új osztályt a `database` csomagban `DbConstants` néven, és konstans tagváltozóként vegyünk fel minden szükséges értéket:

```kotlin
object DbConstants {

    const val DATABASE_NAME = "data.db"
    const val DATABASE_VERSION = 1
    const val DATABASE_CREATE_ALL = Todo.DATABASE_CREATE
    const val DATABASE_DROP_ALL = Todo.DATABASE_DROP

    object Todo {
        const val DATABASE_TABLE = "todo"
        const val KEY_ROWID = "_id"
        const val KEY_TITLE = "title"
        const val KEY_PRIORITY = "priority"
        const val KEY_DUEDATE = "dueDate"
        const val KEY_DESCRIPTION = "description"

        const val DATABASE_CREATE =
                """create table if not exists $DATABASE_TABLE ( 
			$KEY_ROWID integer primary key autoincrement, 
			$KEY_TITLE text not null, 
			$KEY_PRIORITY integer, 
			$KEY_DUEDATE text, 
			$KEY_DESCRIPTION text
			); """

        const val DATABASE_DROP = "drop table if exists $DATABASE_TABLE;"
    }
	
}
```

Figyeljük meg, hogy a `DbConstants` osztályon belül létrehoztunk egy belső `Todo` nevű osztályt, amiben a Todo entitásokat tároló táblához tartozó konstans értékeket tároljuk. Amennyiben az alkalmazásunk több entitást is adatbázisban tárol (gyakori eset!), akkor érdemes az egyes osztályokhoz tartozó konstansokat külön-külön belső osztályokban tárolni. Így sokkal átláthatóbb és karbantarthatóbb lesz a kód, mint ha ömlesztve felvennénk a `DbConstants`-ba az összes tábla összes konstansát. Ezek a belső osztályok praktikusan ugyanolyan névvel léteznek, mint az entitás osztályok (jelen esetben mindkettő neve Todo).

Érdemes megfigyelni továbbá azt, hogy az osztályokat nem a `class` kulcsszóval deklaráltuk. Helyette az`object`-et használjuk, amivel a Kotlin nyelv azt biztosítja számunkra, hogy a `DbConstants` és a benne lévő `Todo` osztály is singletonként viselkednek, azaz az alkalmazás futtatásakor létrejön belőlük egy példány, további példányokat pedig nem lehet létrehozni belőlük.

Itt arra, hogy ezek az osztály példányok léteznek igazából nincs szükségünk, csak azért hozunk létre osztályokat, hogy például a `KEY_ROWID` közvetlen leírása helyett (ha csak egy fájl szintű konstans lenne) a `DbConstants.Todo.KEY_ROWID` szintaxist használjuk.

> Két fontos nyelvi elemet is használunk a konstansok létrehozásához: a string-ekbe paraméterek belefűzésére szolgáló [string template](https://kotlinlang.org/docs/reference/basic-types.html#string-templates)-eket a `$` segítségével, valamint a `"""` által határolt [raw string](https://kotlinlang.org/docs/reference/basic-types.html#string-literals)-eket, amelyek speciális tulajdonsága, hogy tartalmazhatnak új sorokat, illetve nem értelmezik a bennük lévő ` \ ` jeleket escape karakterként.

Következő lépésként implementáljuk a `DatabaseHelper` osztály metódusait. A konstruktor paraméterei közül törölhetjük a `CursorFactory`-t és a verziószámot. Az ősosztály konstruktorának hívásakor a megfelelő helyen adjunk át `null`-t, így az osztály az alapértelmezett `CursorFactory`-t fogja használni. Verzióként adjuk át a `DbConstants.DATABASE_VERSION` konstans értékét. Az `onCreate` és `onUpgrade` metódusokban használjuk a `DbConstants`-ban ebből a célból létrehozott konstansokat.

A módosított `DatabaseHelper` osztály:

```kotlin
class DatabaseHelper(
        context: Context,
        name: String
) : SQLiteOpenHelper(context, name, null, DbConstants.DATABASE_VERSION) {

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(DbConstants.DATABASE_CREATE_ALL)
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        db.execSQL(DbConstants.DATABASE_DROP_ALL)
        db.execSQL(DbConstants.DATABASE_CREATE_ALL)
    }

}
```

Az adatbázis séma létrehozásához és megnyitásához szükséges osztályok így már rendelkezésre állnak. A következő feladatunk az entitás osztályok és az adatbázis közötti kapcsolat implementálása. Ehhez meg kell írnunk azokat a kódrészleteket, melyek egy memóriában lévő Todo objektumot képesek adatbázisba írni, onnan visszaolvasni, módosítani, valamint törölni. (Természetesen más, akár összetettebb funkciók is implementálhatók itt.) Ezt a kódot az entitás osztály helyett a karbantarthatóságot szem előtt tartva érdemes egy külön osztályban megvalósítani a `database` csomagon belül. 

Legyen ez a `TodoDbLoader` osztály: 

```kotlin
class TodoDbLoader(private val context: Context) {

    private lateinit var dbHelper: DatabaseHelper
    private lateinit var db: SQLiteDatabase

    @Throws(SQLException::class)
    fun open() {
        dbHelper = DatabaseHelper(context, DbConstants.DATABASE_NAME)
        db = dbHelper.writableDatabase

        dbHelper.onCreate(db)
    }

    fun close() {
        dbHelper.close()
    }

    // CRUD és egyéb metódusok
}
```

Az elérhető `SQLException`-ök közül használjuk az `android.database` package-ben lévőt.

> A [`@Throws`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-throws/index.html) annotáció csak a [Java kóddal való interoperációt](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#checked-exceptions) segíti, ha tisztán Kotlin alkalmazásunk van, igazából nem kell használnunk. Java-val ellentétben Kotlinban ugyanis nincsenek kötelezően lekezelendő, ún. [checked exception](https://kotlinlang.org/docs/reference/exceptions.html#checked-exceptions)-ök.

A létrehozó, módosító és törlő metódusok a `TodoDbLoader`-en belül, a *CRUD és egyéb metódusok* komment után:

```kotlin
// INSERT
fun createTodo(todo: Todo): Long {
    val values = ContentValues()
    values.put(DbConstants.Todo.KEY_TITLE, todo.title)
    values.put(DbConstants.Todo.KEY_DUEDATE, todo.dueDate)
    values.put(DbConstants.Todo.KEY_DESCRIPTION, todo.description)
    values.put(DbConstants.Todo.KEY_PRIORITY, todo.priority.ordinal)

    return db.insert(DbConstants.Todo.DATABASE_TABLE, null, values)
}

// DELETE
fun deleteTodo(rowId: Long): Boolean {
    val deletedTodos = db.delete(
            DbConstants.Todo.DATABASE_TABLE,
            "${DbConstants.Todo.KEY_ROWID} = $rowId",
            null
    )
    return deletedTodos > 0
}

// UPDATE
fun updateTodo(newTodo: Todo): Boolean {
    val values = ContentValues()
    values.put(DbConstants.Todo.KEY_TITLE, newTodo.title)
    values.put(DbConstants.Todo.KEY_DUEDATE, newTodo.dueDate)
    values.put(DbConstants.Todo.KEY_DESCRIPTION, newTodo.description)
    values.put(DbConstants.Todo.KEY_PRIORITY, newTodo.priority.ordinal)

    val todosUpdated = db.update(
            DbConstants.Todo.DATABASE_TABLE,
            values,
            "${DbConstants.Todo.KEY_ROWID} = ${newTodo.id}",
            null
    )

    return todosUpdated > 0
}
```

Fussuk át és értelmezzük a bemásolt kódot!

A kód bemásolása után importáljuk be a megfelelő osztályokat. `Todo` osztály az alkalmazásunkban két helyen is szerepel, egyrészt entitás osztályként a `model` csomagban, másrészt az adatbázis konstansokat tároló belső osztályként a `database` csomagban. A bemásolt kód célja, hogy a Todo entitásokat létrehozza, módosítsa vagy törölje az adatbázisban, így az entitás osztályt importáljuk be (`hu.bme.aut.android.todo.model.Todo`).

Általában szükséges az egyetlen rekordot és a teljes rekord halmazra mutató `Cursor`-t visszaadó függvények implementálása is.

Ezek implementációja a `TodoDbLoader` osztályban, a *CRUD műveletek* után:

```kotlin
// Get all Todos
fun fetchAll(): Cursor {
    // Cursor pointing to the result set of all Todos with all fields (where = null)
    return db.query(
            DbConstants.Todo.DATABASE_TABLE,
            arrayOf(
                    DbConstants.Todo.KEY_ROWID,
                    DbConstants.Todo.KEY_TITLE,
                    DbConstants.Todo.KEY_DESCRIPTION,
                    DbConstants.Todo.KEY_DUEDATE,
                    DbConstants.Todo.KEY_PRIORITY
            ),
            null,
            null,
            null,
            null,
            DbConstants.Todo.KEY_TITLE
    )
}

// Querying for one of the Todos with the given id
fun fetchTodo(id: Long): Todo? {
    // Cursor pointing to a result set with 0 or 1 Todo
    val cursor = db.query(
            DbConstants.Todo.DATABASE_TABLE,
            arrayOf(
                    DbConstants.Todo.KEY_ROWID,
                    DbConstants.Todo.KEY_TITLE,
                    DbConstants.Todo.KEY_DESCRIPTION,
                    DbConstants.Todo.KEY_DUEDATE,
                    DbConstants.Todo.KEY_PRIORITY
            ),
            "${DbConstants.Todo.KEY_ROWID} = $id",
            null,
            null,
            null,
            DbConstants.Todo.KEY_TITLE
    )

    // Return with the found entry or null if there wasn't any with the given id
    return if (cursor.moveToFirst()) {
        getTodoByCursor(cursor)
    } else {
        null
    }
}

companion object {
    fun getTodoByCursor(c: Cursor): Todo {

        val priority = when (c.getInt(c.getColumnIndex(DbConstants.Todo.KEY_PRIORITY))) {
            0 -> Todo.Priority.LOW
            1 -> Todo.Priority.MEDIUM
            2 -> Todo.Priority.HIGH
            else -> Todo.Priority.LOW
        }

        return Todo(
                id = c.getLong(c.getColumnIndex(DbConstants.Todo.KEY_ROWID)),
                title = c.getString(c.getColumnIndex(DbConstants.Todo.KEY_TITLE)),
                priority = priority,
                dueDate = c.getString(c.getColumnIndex(DbConstants.Todo.KEY_DUEDATE)),
                description = c.getString(c.getColumnIndex(DbConstants.Todo.KEY_DESCRIPTION))
        )
    }
}
```

## Lista feltöltése adatbázisból

Ha a Todo-kat megjelenítő lista adapterét adatbázisból szeretnénk feltölteni, akkor a `RecyclerView.Adapter` helyett használhatunk egy külön erre a célra készült `CursorRecyclerViewAdapter` osztályt ősosztálynak, így leegyszerűsítve a feladatot. Ez az osztály a következő linken érhető el:

[https://gist.github.com/skyfishjy/443b7448f59be978bc59#file-cursorrecyclerviewadapter-java](https://gist.github.com/skyfishjy/443b7448f59be978bc59#file-cursorrecyclerviewadapter-java)

Hozzuk létre a `CursorRecyclerViewAdapter` nevű osztályt a `feature.list` package-ben és másoljuk be a kódját a fenti linkről.

Figyeljük meg, hogy a másolt kód *Java* nyelven íródott. Ez természetesen nem jelent problémát, a Kotlin nyelv interoperabilitási támogatásával gond nélkül tudjuk majd használni ezt az osztályt a Kotlinban írt kódunkból.

A megoldás hasonló az előző adapter kódjához, de itt az `onBindViewHolder` metódusban egy `Cursor`-t kapunk pozíció helyett, és ennek segítségével szerezhetjük meg a megjelenítendő adatokat. Ezen felül az osztály nem egy listában tárolja a `Todo` elemek referenciáit, hanem egy `Cursor`-t tart az elemek kiolvasásához.

Hozzuk létre a `TodoAdapter` osztályt a `feature.list` csomagban:

```kotlin
class TodoAdapter(context: Context, cursor: Cursor) : CursorRecyclerViewAdapter<TodoAdapter.ViewHolder>(context, cursor) {

    var itemClickListener: TodoItemClickListener? = null

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.row_todo, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, cursor: Cursor) {
        val todo = TodoDbLoader.getTodoByCursor(cursor)

        holder.todo = todo

        holder.tvTitle.text = todo.title
        holder.tvDueDate.text = todo.dueDate

        val resource = when (todo.priority) {
            Todo.Priority.LOW -> R.drawable.ic_low
            Todo.Priority.MEDIUM -> R.drawable.ic_medium
            Todo.Priority.HIGH -> R.drawable.ic_high
        }
        holder.ivPriority.setImageResource(resource)
    }

    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val tvDueDate: TextView = view.tvDueDate
        val tvTitle: TextView = view.tvTitle
        val ivPriority: ImageView = view.ivPriority

        var todo: Todo? = null

        init {
            itemView.setOnClickListener {
                todo?.let { itemClickListener?.onItemClick(it) }
            }

            itemView.setOnLongClickListener { clickedView ->
                todo?.let { itemClickListener?.onItemLongClick(clickedView, it) }
                true
            }
        }
    }

    interface TodoItemClickListener {
        fun onItemClick(todo: Todo)
        fun onItemLongClick(view: View, todo: Todo): Boolean
    }

}
```

## Egyedi Application osztály készítése

Készítsünk egy egyedi Application osztályt `TodoApplication` néven. Az `Application` az alkalmazás futása során folyamatosan jelen lévő objektum, melyet a futtatókörnyezet hoz létre automatikusan az alkalmazás indulásakor. Élettartama az alkalmazáshoz tartozó process élettartamához kötődik. A `TodoApplication` fogja fenntartani az adatbáziskapcsolatot reprezentáló TodoDbLoader objektumot, így azt nem kell minden `Activity`-ből külön-külön, erőforrás pazarló módon példányosítani.

A `hu.bme.aut.android.todo` package-en belül hozzuk létre a `TodoApplication` osztályt:

```kotlin
class TodoApplication : Application() {

	companion object {
		lateinit var todoDbLoader: TodoDbLoader
	  		private set
	}
	
	override fun onCreate() {
		super.onCreate()
	
		todoDbLoader = TodoDbLoader(this)
		todoDbLoader.open()
	}
	
	override fun onTerminate() {
		todoDbLoader.close()
		super.onTerminate()
	}
	
}
```

Figyeljük meg, hogy a fejlesztőkörnyezet figyelmeztet egy lehetséges hibára!

Ebben a példa alkalmazásban nem fog gondot okozni egy `Context` referencia "statikus" tárolása (mivel az `Application` által nyújtott `Context`-et tároljuk, aminek az élettartama megegyezik a statikus változó élettartamával), viszont érdemes tudni, hogy ez sokszor memóriaszivárgáshoz vezethet (például ha egy `Activity` `Context`-jét tárolnánk el így).

> A `todoDbLoader` property-nél leírt [`private set`](https://kotlinlang.org/docs/reference/properties.html#getters-and-setters) segítségével azt érjük el, hogy a property settere privát lesz, de a gettere publikus marad (ha magára a property-re helyeznénk el a `private` kulcsszót, mindkettő priváttá válna). Ezzel elérjük, hogy az aktuális osztályon kívülről ne lehessen átállítani az értékét.

Az `AndroidManifest.xml`-ben állítsuk be, hogy a rendszer a `TodoApplication` osztályt példányosítsa az alkalmazás induláskor. Ehhez a manifestben lévő `<application>` tagben fel kell vennünk egy attribútumot `android:name` néven, aminek az értéke legyen a `TodoApplication` minősített (fully-qualified) osztályneve:

```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:name=".TodoApplication">
```

Bár az `android:name` attribútum értékének egy fully-qualified osztálynévnek kell lennie, a rendszer elfogadja a manifestben beállított `package` névhez relatív értéket is. Figyeljük meg, hogy az `android:name` beállításakor a fejlesztőkörnyezet *Ctrl+Space* lenyomására segíti az osztálynév megadását.

Ezt követően a `TodoApplication` osztály companion object-jének `todoDbLoader` propertyjével bármikor elérhetjük az SQLite adatbázis kapcsolatunkat, a `TodoApplication.todoDbLoader` szintaxissal.

## TodoListActivity módosítása

Az adatbázis alapú perzisztencia használatához minden rendelkezésre áll, már csak a `TodoListActivity`-ben kell áttérnünk az adatok adatbázisban tárolásra.

Adjuk hozzá a `TodoListActivity`-hez az alábbi propertyket:

```kotlin
private lateinit var adapter: TodoAdapter

private lateinit var dbLoader: TodoDbLoader

private var loadTodosTask: LoadTodosTask? = null
```

*Töröljük ki* a `TodoListActivity`-ből a régi adaptert, azaz az alábbi propertyt:

```kotlin
private lateinit var simpleItemRecyclerViewAdapter: SimpleItemRecyclerViewAdapter
```

Az adapter váltás miatt szükséges az, hogy a `TodoListActivity` ne a `SimpleItemRecyclerViewAdapter`-ben definiált `TodoItemClickListener`-t implementálja, hanem a `TodoAdapter`-ben definiáltat.

A `TodoListActivity` új definíciója:

```kotlin
class TodoListActivity : AppCompatActivity(), TodoCreateFragment.TodoCreatedListener, TodoAdapter.TodoItemClickListener {
```

A `TodoListActivity`ben írjuk át az `onItemLongClick` függvényt is a `TodoAdapter.TodoItemClickListener` interfésznek megfelelően:

```kotlin
override fun onItemLongClick(view: View, todo: Todo): Boolean {
	val popup = PopupMenu(this, view)
	popup.inflate(R.menu.menu_todo)
	popup.setOnMenuItemClickListener { item ->
		when (item.itemId) {
			R.id.delete -> {
			  // TODO
			}
		}
		false
	}
	popup.show()
	return false
}
```

Módosítsuk a `setupRecyclerView` metódust úgy, hogy az új adapterünket használja, és csak ennek a `RecyclerView`-val és a `TodoListActivity`-vel való összekötését végezze el:

```kotlin
private fun setupRecyclerView() {
    adapter.itemClickListener = this
    rvTodoList.adapter = adapter
}
```

Ezek után már nincs szükség a korábban használt `SimpleItemRecyclerViewAdapter`-re, törölhetjük az osztályt tartalmazó fájlt a `feature.list` package-ből.

Törölni kell még az `onTodoCreated` függvény törzséből az alábbi sort, mert a `TodoAdapter` interfésze különbözik az eddig használt adapterétől.

```kotlin
simpleItemRecyclerViewAdapter.addItem(todo)
```

Az diszk I/O műveleteket javasolt a háttérszálon, aszinkron módon futtatni. ellenkező esetben könnyen megakaszthatják a UI szálat esetlegesen lassú lefutásukkal.

Készítsük el a `LoadTodosTask` osztályt a `database` csomagban, ami származzon az `AsyncTask` osztályból. Ez aszinkron módon kérdezi majd le az adatbázisunkból az összes elmentett `Todo` rekordot - az `AsyncTask` `doInBackground` függvénye háttérszálon fut, az ott előállított eredményt pedig az `onPostExecute` függvényben a fő szálon használhatjuk fel.

A `LoadTodosTask` osztály kódja az alábbi lesz:

```kotlin
class LoadTodosTask(
        private val listActivity: TodoListActivity,
        private val dbLoader: TodoDbLoader)
    : AsyncTask<Unit, Unit, Cursor>() {

    companion object {
        private const val TAG = "LoadTodosTask"
    }

    override fun doInBackground(vararg params: Unit): Cursor? {
        return try {

            val result = dbLoader.fetchAll()

            if (!isCancelled) {
                result
            } else {
                Log.d(TAG, "Cancelled, closing cursor")
                result.close()
                null
            }
        } catch (e: Exception) {
            Log.d(TAG, "An error occurred while fetching Todos")
            null
        }
    }

    override fun onPostExecute(result: Cursor?) {
        Log.d(TAG, "Fetch completed, displaying cursor results")
        listActivity.showTodos(result)
    }

}
```

> Egy újabb vezérlési struktúra expression-ként való viselkedését látjuk itt: a [`try-catch` blokk](https://kotlinlang.org/docs/reference/exceptions.html#try-is-an-expression) is rendelkezik visszatérési értékkel, amely sikeres futás esetén a `try`, exception esetén pedig a `catch` ág utolsó kifejezése.

A `TodoListActivity`-ben hozzuk létre a `showTodos` függvényt:

```kotlin
fun showTodos(todos: Cursor?) {
	if (todos != null) {
		adapter = TodoAdapter(applicationContext, todos)
		setupRecyclerView()
	} else {
		// Handle the case when there was an error while fetching todos
		// Not part of this exercise
	}
	loadTodosTask = null
}
```

Ezt követően a `TodoListActivity` életciklus-függvényeit fogjuk módosítani.

Az `onCreate()` függvényben kérjünk referenciát a `TodoApplication`-től a nyitott adatbázis kapcsolatot tartalmazó `TodoDbLoader`-re, illetve töröljük ki a függvényből a `setupRecyclerView()` hívást.

Az `onCreate()` függvény a módosítások után:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_todo_list)

	setSupportActionBar(toolbar)
	toolbar.title = title

	fab.setOnClickListener { view ->
		Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
				.setAction("Action", null).show()
	}

	if (todo_detail_container != null) {
		// The detail container view will be present only in the
		// large-screen layouts (res/values-w900dp).
		// If this view is present, then the
		// activity should be in two-pane mode.
		twoPane = true
	}

	dbLoader = TodoApplication.todoDbLoader
}
```

Írjuk meg az `onResume()` függvényt és indítsuk el benne a `Todo` lista tartalmának lekérdezését:

```kotlin
override fun onResume() {
	super.onResume()

	refreshList()
}
```

Készítsük is el a hiányzó `refreshList()` metódust, amely elindítja az aszinkron adatbázis lekérdező folyamatot:

```kotlin
private fun refreshList() {
	loadTodosTask?.cancel(false)

	val loadTodosTask = LoadTodosTask(this, dbLoader)
	loadTodosTask.execute()

	this.loadTodosTask = loadTodosTask
}
```

Írjuk meg az `onPause()` függvényt. Ha az alkalmazás háttérbe kerülésekor van folyamatban adatbázis lekérdezés, akkor megszakítjuk:

```kotlin
override fun onPause() {
	super.onPause()

	loadTodosTask?.cancel(false)
}
```

Írjuk meg az `onDestroy()` függvényt, amiben bezárjuk az adapterhez csatolt, Todo elemeket tartalmazó Cursor objektumot, ha létezik ilyen:

```kotlin
override fun onDestroy() {
	super.onDestroy()

	adapter.cursor?.close()
}
```

Ha minden kódrészlet a megfelelő helyen van, akkor az alkalmazás indulásakor ugyanaz a felhasználói felület jelenik meg, mint a kiinduló projekt esetén. Azonban most még üres listát látunk, hiszen üres az adatbázis. Ahhoz, hogy újra működjön a Todo létrehozás funkció, módosítanunk kell az `onTodoCreated` függvényt:

```kotlin
override fun onTodoCreated(todo: Todo) {
	dbLoader.createTodo(todo)
	refreshList()
}
```

A `dbLoader.createTodo(todo)` hívást természetesen az adatbázisból betöltés mintájára javasolt háttérszálon futtatni, ezt most a példa alkalmazásban az egyszerűsítés kedvéért nem tesszük meg.

Indítsuk el az alkalmazást! Próbáljuk ki, hogy forgatás, illetve az alkalmazás háttérbe küldése és visszahozása után is megmaradnak a létrehozott Todo-k.

## Önálló feladat

Az eredeti funkcionalitás további részeinek megvalósítása önálló feladatként oldandó meg:

-  Az új Todo beszúrását is végezzük egy `AsyncTask`-ban, háttérszálon!
-  Hosszan nyomásra működjön a kiválasztott Todo elem törlése
-  Az összes Todo elem törlésére is legyen lehetőség (pl. Options menüből elérve)
