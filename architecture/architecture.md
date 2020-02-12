# UX alapok


## Bevezetés

Ma már igen sok alkalmazás található a Play áruházban, köszönhető ez többek között a terjesztés egyszerűségének és az alacsony belépési korlátnak. Azonban ezeknek az alkalmazásoknak igen magas hányada végzi alacsony értékeléssel és magas elégedetlenséggel. A legtöbb projekt fejlesztője késznek érzi az alkalmazást, azonban csak magára gondol és nem a felhasználóra. Ezen a laboron bemutatunk néhány egyszerű fogalmat és technikát, ami segít abban, hogy minőségibb alkalmazások szülessenek. Ehhez egy prototípus alkalmazást fogunk Material szemlélettel javítani.

Tartsuk szem előtt, hogy ez itt csak néhány egyszerű ötlet, és ahogy egy-két Material megjelenésű elemtől még nem lesz Material Design szerinti egy alkalmazás megjelenése, úgy mi sem leszünk mindenható designerek a labor után. Ez a témakör koránt sem olyan egyértelmű, mint a mérnöki tanulmányok jó része, az itt alkalmazott lépések nem feltétlenül univerzálisak.


## Material és UX alapok

A Material Design teljes útmutatója a [material.io/design](https://material.io/design/) címen érhető el. Ezt érdemes egyszer jól átnézni, és később egy-egy adott design kérdés felmerülésekor újra felkeresni a megfelelő részeit, hogy informált döntést tudjunk hozni, akár úgy is, hogy nincs külön designerünk az alkalmazásunkhoz.

* A Material Designban lényeges minden elem megjelenése. Például a *Floating Action Button* vetett árnyéka nem véletlen. Funkciója kiemelkedik, ezért megjelenítéskor is azt próbáljuk hangsúlyozni, hogy feljebb van. Az elemeknek van Z tengely szerinti [**magasságuk (elevation)**](https://material.io/design/environment/elevation.html), a rendszer ebből számolja az általuk vetett árnyékot.

* Az [**animációknak jelentéssel kell bírniuk**](https://material.io/design/motion/understanding-motion.html). Csak úgy nem animálunk dolgokat, mert az jól néz ki. Ha egy listaelemre vagy kártyára tapint a felhasználó, akkor azt az elemet átanimálhatjuk az új képernyővé úgy, hogy a kártya (vagy listaelem) olyan elemei, amik a következő képernyőn is szerepelnek elfoglalják új helyüket a második képernyőn. A platformon már régóta elérhetőek egyszerű áttűnések, lapozások a képernyőátmenetek közt. Erre azért van szükség, mert a felhasználót megzavarhatja a hirtelen váltás. A való életben sem villanással kerül az égre a madár, hanem lezajlik egy átmenet (a felszállás), ami átvezet a két állapot között. Emlékezzünk vissza: találkoztunk már olyan alkalmazással, ahol a képernyőváltás során oldalirányban úsztak ki-be a képernyők? Előre és visszafelé navigálásnál melyik irányba haladtak a képernyők?

* Bár laboron a beépített komponensek során nem futunk ebbe a problémába, de jegyezzük meg: [**minden felhasználói interakcióra legyen visszajelzés!**](https://material.io/design/interaction/gestures.html#properties) Gondoljunk bele mit teszünk, ha egy gombot megnyomva nem történik semmi (fel se villan a gomb nyomott állapota), mit feltételezünk? Jusson ez eszünkbe egyéni nézetek készítésekor is! A Material szemlélet ezt bővíti még azzal, hogy a kiváltott változás egy egyre nagyobb sugarú kör íve mentén történik, aminek középpontja az a hely, ahol a felhasználó kiváltotta a változást.
[Egy példa ilyen animációra.](http://material-design.storage.googleapis.com/publish/material_v_3/material_ext_publish/0B3T7oTWa3HiFdWhpd296VUhLTFk/animation_responsiveinteraction_radialreaction.webm?_=1)

* **Minimális gépelés.** Nehéz nagyobb fájdalmat okozni a felhsználónak annál, hogy mobilon kelljen gépelnie, nem véletlenül létezik annyi különböző gesztúra. Ha egyszerű bevitelről van szó (számok, intervallumok, dátumok) egyszerűsítsük csúszkával, vizuális naptári választóval.

* A Materialos színekhez jó segítség például [ez a (nem hivatalos) oldal](http://www.materialpalette.com/). Innen le is tudjuk tölteni xml formában a kapott színeket. Annyit érdemes tudni, hogy az "accent" színt csak a leghangsúlyosabb elemek kaphatják meg. Tipikusan ilyen a *Floating Action Button*, ami a képernyő legfontosabb funkcióját kell jelentse, ha van ilyen. Ugyanúgy másodlagos színt kapnak például a *Switch* és a *Seekbar* nézetek is. A hivatalos leírás is [részletesen taglalja a színek használatát](https://material.io/design/color/the-color-system.html).

* Ikonokat tervezni külön szakma, de mivel a legtöbb fejlesztő nem foglalkozik ilyesmivel másodállásban, ezért a Google nem csak néhány ikont készített el előre, hanem rengeteget. [Innen](https://material.io/tools/icons/) letölthetők a hivatalos ikonok. 

* A fejlesztői közösség az előbbit továbbgondolta, és létrehoztak egy oldalt ([materialdesignicons.com](https://materialdesignicons.com/)) ezen ikonok egyszerű kezelésére, illetve számos továbbival ki is egészítették a Google által nyújtott ikonkészletet. Alapszabály, hogy ne használjunk olyan ikont (bármennyire is jó lenne), ami már másik ismert funkciót jelöl. Hasonlóan kinéző komponenstől a felhasználó azt várja, hogy hasonlóan fog működni. A *Floating Action Button* funkciójának alapvetően egy pozitív cselekedetnek kell lennie (jó példa az új elem létrehozása, rossz példa a szín módosítása vagy a kuka törlése), így válasszunk ennek megfelelő ikont. A laborhoz mellékeljük a szükséges ikonokat, de az otthoni használathoz a javasolt mód a linkelt oldal használatával az alábbi:
  * Az Android 5.0 változatot töltsük le.
  * Válasszuk ki a nekünk kellő variánst. Szerepel fehérben, feketében, szürkében és mindhárom színhez 4 féle méretben.
  * Másoljuk az összes minősített mappát a célhelyre.

* Tartsunk megfelelő távolságokat az elemek között, különösen ügyelve az interaktív elemekre. Lesz olyan, akinek nálunk nagyobb az ujjbegye, gondoljunk rá is! A tartalom ne kezdődjön a képernyő 0. pixelénél! Az [layout ajánlásokban](https://material.io/design/layout/spacing-methods.html) elég részletesen taglalják a számokat: az új guideline szerint minden elem egy 8dp-s rácsban helyezkedik el. Ez alól kivételek a szövegek (amiknek alapvonala igazodik 4dp-hez) és a toolbar ikonjai (szintén 4dp). Tehát alapvetően mindennek a mérete vagy a távolsága n x 8dp. A kijelző szélétől tartandó margó például 16dp, az érinthető területek minimum mérete 48 x 48dp, a köztük tartandó távolság pedig minimum 8dp, de inkább több.

* A képi elemek legyenek inkább személyesek. Ne használjunk pár élettelen mosolyú modell arcát mutató stock fotókat, a képnek legyen köze a tartalomhoz. A személyes (felhasználó készítette) képek még jobbak. A képek töltsék ki a teret, amennyire csak lehet! Ez azt jelenti, hogy szélességben a teljes kijelzőt fedje, magasságban pedig lehetőleg valamilyen jellegzetes arány vonalát kövesse. Van [néhány ajánlás](https://material.io/design/communication/imagery.html) ezekre az arányokra – mármint arra hogy bizonyos képarányú elemek magassága hol helyezkedik el.

## Hasznos fejlesztői eszközök

Amikor a felhasználói felületet igazítjuk, nem mindig egyértelmű, hogy miért azt látjuk renderelve, amit. A *Settings -> Developer options* menüpontban találjuk az alábbiakat:

* **Show layout bounds**: Megmutatja mettől-meddig tartanak a nézetek, kiderülhet melyik eltartás (margin, padding) melyik nézethez tartozik.
* **Windows animation scale**, **Transition animation scale** és **Animator duration scale**: Segítségükkel lelassíthatóak, részletesebben megtekinthetőek az egyébként gyors animációk.
* **Debug GPU overdraw**: Megmutatja melyek azok a területek, amelyeknek tartalma többször is meg lett adva. Minél többször van szín rendelve egy pixelhez, annál sötétebb szín jelzi.
* **Profile GPU rendering**: Megmutatja mennyi ideig tartott lerenderelni az adott képkockákat. A képernyőn megjelenő vízszintes vonal jelzi a 16ms határát.  Ha egy oszlop e fölé ér, azt jelenti, hogy az a képkocka nem készült el időben a 60 fps-hez, ezért megakadt a felület.

Próbálják ki ezeket a funkciókat!


## Javítandó alkalmazás

Most, hogy néhány hasznos dolgot megismertünk, ideje letöltenünk a prototípust:

[PlacesToVisit.zip](./assets/PlacesToVisit.zip)

Tömörítsük ki a mappát, indítsuk el az Android Studio-t, majd nyissuk meg az alkalmazást.

<img src="./images/screen1_framed.png" width="200" align="middle">

Ennek az alkalmazásnak az a feladata, hogy meglátogatandó helyeket gyűjtsünk benne. A prototípus arra koncentrál, hogy minimális funkcionalitást valósítson meg gyorsan. Az adatok perzisztens tárolásához a Google által készített [*Room*](https://developer.android.com/topic/libraries/architecture/room.html) könyvtárat használja. Laborvezetővel tekintsék át a kódot és a működést! Főbb elemei:

* A Room számára annotációkkal adjuk meg, hogy pontosan hogyan történjen az adott modell objektum mentése.

    ```kotlin
    @Entity
    data class Place(
            @PrimaryKey(autoGenerate = true)
            var id: Int = 0,
            @ColumnInfo(name = "place_type")
            var placeType: PlaceType = PlaceType.fromInt(0),
            @ColumnInfo(name = "place_name")
            var placeName: String = "",
            @ColumnInfo(name = "description")
            var description: String = "",
            @ColumnInfo(name = "creation_date")
            var creationDate: Date = Date(0)
    )
    ```

* Az adatok manipulálását az úgynevezett DAO (Data Access Object) osztályokon keresztül végezetjük. Mi csak egy megfelelő annotációkkal ellátott interface-t definiálunk, az implementációt pedig (a korábban már látott Retrofithez hasonlóan) a könyvtár generálja majd nekünk. Az interface függvényein lévő annotációk magukért beszélnek - az `@Insert` beilleszt elemeket, a `@Delete` töröl, a `@Query`-be pedig tetszőleges SQL kódot írhatunk, kódkiegészítésel!

    ```kotlin
    @Dao
    interface PlaceDao {
    
        @Insert
        fun insertAll(vararg places: Place)
    
        @Query("SELECT * FROM Place")
        fun getAll(): List<Place>
    
        @Update
        fun updatePlace(place: Place): Int
    
        @Delete
        fun delete(place: Place)
    
    }
    ``` 

* A generált Dao implementáció egy `RoomDatabase`-ből származó osztály segítségével érhető el. Ez is egy `abstract` osztály, melyhez az implementációt a Room generálja majd. Az adatbázis általános beállításai itt adhatóak meg, szintén annotációkkal.

    ```kotlin
    @Database(entities = [Place::class], version = 1, exportSchema = false)
    @TypeConverters(DateTypeConverter::class, PlaceTypeConverter::class)
    abstract class PlaceDatabase : RoomDatabase() {
    
        abstract fun placeDao(): PlaceDao
    
    }
    ```

* Az adatbázisból példányt a `Room` osztály factory metódusainak segítségével kaphatunk.

    ```kotlin
    db = Room.databaseBuilder(applicationContext, 
            PlaceDatabase::class.java, "place-db").build()
    ```


### Menü

Új elemet az options menü megnyomásával lehet létrehozni, amely menüben más elem nincs is. Ez a menü tipikusan nem ilyen feladatokra szolgál, ezt inkább *Floating Action Button*-nel szokás megoldani. Ezért először is töröljük a menüt a nyitóképernyőről: távolítsuk el a `PlacesListActivity` meglévő `onCreateOptionsMenu` és `onOptionsItemSelected` metódusait, illetve a használt `menu_places_list.xml` erőforrást. 

Ezután hozzunk létre egy *Floating Action Button*-t, az `activity_places_list.xml`-ben az `include` után:

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:id="@+id/addButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_alignParentEnd="true"
    android:layout_alignParentBottom="true"
    android:layout_gravity="bottom|end"
    android:layout_margin="@dimen/fab_margin" />
```

Állítsuk be a megfelelő listenert is a `PlaceListActivity` `onCreate` függvényében:

```kotlin
addButton.setOnClickListener {
    showNewPlaceDialog()
}
```

### FAB ikon

A *Floating Action Button* ikonja fontos szerepet játszik. A felhasználónak első ránézésre tudnia kell belőle, hogy mire szolgál a gomb. Így tehát olyan ikont kell választanunk, amiből rögtön látszik, hogy a gomb elem hozzáadására szolgál. Töltsük le az ikont tartalmazó *zip* fájlt: [drawable.zip](./assets/drawable.zip)

Tömörítsük ki és tegyük a projektünkbe, majd állítsuk be a FAB ikonját az `activity_places_list.xml`-ben:

```xml
app:srcCompat="@drawable/ic_add_white_24dp"
```

### A lista fejléce

Jelenleg a `Toolbar`-on megjelenik az `Activity` neve, ami *PlacesToVisit*, alatta pedig egy `TextView`-ban a *Places to visit* felirat. Ezek közül az egyik felesleges, és szebb, ha a jobban olvasható *Places to visit*-et hagyjuk meg. Azonban egy üres `Toolbar`-nak nincs sok értelme, ezért inkább rakjuk fel ezt a szöveget oda, és távolítsuk el a `TextView`-t.

Ehhez először is az `activity_places_list.xml`-ben a `Toolbar` tagen belül vegyük fel az `app:title` attribútumot:

```xml
app:title="@string/places_to_visit"
```

A `content_places_list.xml`-ből pedig töröljük az ott lévő `TextView`-t, illetve a rá való hivatkozást a `RecyclerView`-ból.

Próbáljuk ki az alkalmazást!

<img src="./images/screen2_framed.png" width="200" align="middle">

### Üres lista

Kevesen készülnek arra a lehetőségre, hogy mi fogadja a felhasználót akkor, ha üres a listanézet. Célszerű ilyenkor az üres lista helyett valamilyen szöveget (esetleg illusztrációt) megjeleníteni. Írjuk ehhez át a `content_places_list.xml`-t. Adjunk hozzá egy `TextView`-t, a `RelativeLayout` közepére igazítva:

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
          android:id="@+id/emptyTV"
          android:layout_centerHorizontal="true"
          android:layout_centerVertical="true"
          style="@style/TextViewTitleStyle"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@string/add_places_to_visit"/>
```

Az üres nézet szöveges erőforrása legyen például:

```xml
<string name="add_places_to_visit">Add places to visit</string>
```

A `RecyclerView` és a `TextView` most egymáson helyezkedik el a nézetben. Ez nem lesz probléma, mivel mindig csak az egyik lesz látható. Ehhez meg kell oldanunk, hogy üres `RecyclerView` esetén csak a `TextView` jelenjen meg (és fordítva):

Először is hozzunk létre egy új custom `View`-t, ami képes ezt kezelni. Kerüljön a `view` package-be, a neve legyen `EmptyRecyclerView`, és származzon a `RecyclerView`-ból. Először implementáljuk a három kötelező konstruktorát:

```kotlin
class EmptyRecyclerView : RecyclerView {

    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context, attrs: AttributeSet?, defStyle: Int) : super(context, attrs, defStyle)
    
}
```

Vegyünk fel bele egy `emptyView` nevű property-t, amiben azt a `View`-t fogjuk tárolni, amit üres lista esetén meg szeretnénk jeleníteni:

```kotlin
var emptyView: View? = null
```

Ha bármilyen változás történik az adathalmazban, le kell ellenőriznünk, hogy a melyik felületet kell megjelenítenünk attól függően, hogy üres-e a lista. Erre szolgál a `checkIfEmpty` függvény. Implementáljuk ezt:

```kotlin
fun checkIfEmpty() {
    val emptyView = emptyView
    val adapter = adapter

    if (emptyView != null && adapter != null) {
        val emptyViewVisible = adapter.itemCount == 0

        emptyView.visibility = if (emptyViewVisible) View.VISIBLE else View.GONE
        this.visibility = if (emptyViewVisible) View.GONE else View.VISIBLE
    }
}
```

Látható, hogy a `checkIfEmpty` függvény az adapterben található elemek számának függvényében állítja az `EmptyRecyclerView`, és az `emptyView` láthatóságát.

Vegyünk fel egy `RecyclerView.AdapterDataObserver` példányt, aminek a feladata, hogy a listában történt változásokat lekezelje. Láthatjuk, hogy az `override`-olt függvényei megfelelnek a `RecyclerView.Adapter`-nél már ismert `notify...` hívásoknak. Bármilyen változás történik, a `checkIfEmpty` segítségével beállítjuk a megfelelő láthatóságokat. 

```kotlin
private val observer = object : RecyclerView.AdapterDataObserver() {
    override fun onChanged() {
        checkIfEmpty()
    }

    override fun onItemRangeInserted(positionStart: Int, itemCount: Int) {
        checkIfEmpty()
    }

    override fun onItemRangeRemoved(positionStart: Int, itemCount: Int) {
        checkIfEmpty()
    }
}
```

Ezt az ellenőrzést akkor is meg kéne tennünk, amikor beállítjuk az `emptyView` értékét. Adjunk hozzá a property-hez egy *custom setter*-t, amiben beállítjuk a *backing field* értékét, utána pedig meghívjuk a `checkIfEmpty` függvényt:

```kotlin
var emptyView: View? = null
    set(value) {
        field = value
        checkIfEmpty()
    }
```

>Egy Kotlin property igazából egy [backing field](https://kotlinlang.org/docs/reference/properties.html#backing-fields), egy getter függvény, és egy setter (amennyiben `var`-ként lett deklarálva) összessége. A getter alapértelmezett működése az, hogy visszaadja a backing field értékét, a setter pedig beállítja azt.

Az `EmptyRecyclerView` megfelelő működéséhez felül kell még írnunk a `setAdapter` függvényt. Ebben tudjuk beregisztrálni az imént létrehozott observerünket, ami kezeli az adathalmazban történt változásokat. Az adapter cseréje esetén a régi adapter változásairól is itt iratkozhatunk le:

```kotlin
override fun setAdapter(adapter: RecyclerView.Adapter<*>?) {
    val oldAdapter = this.adapter
    oldAdapter?.unregisterAdapterDataObserver(observer)

    super.setAdapter(adapter)

    adapter?.registerAdapterDataObserver(observer)

    checkIfEmpty()
}
```

Ezzel el is készült az `EmptyRecyclerView` osztályunk. Cseréljük le erre az eddig használt `RecyclerView`-t a `content_places_list.xml` fájlban:

```xml
<hu.bme.aut.android.placestovisit.view.EmptyRecyclerView
    android:id="@+id/placesListRV"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

A `PlacesListActivity` `onCreate` függvényében a `RecyclerView` inicializálása után állítsuk be az `emptyView`-jának az erre a célra létrehozott `TextView`-t:

```kotlin
placesListRV.emptyView = emptyTV
```

Próbáljuk ki az alkalmazást! Láthatjuk, hogy üres lista helyett valóban az *Add places to visit* felirat jelenik meg.

<img src="./images/screen3_framed.png" width="200" align="middle">
 
Segíthetünk a felhasználónak még annyiban, hogy megengedjük, hogy erre a feliratra rákattintva is vehessen fel új helyet. Ehhez vegyünk fel egy listenert a `PlacesListActivity` `onCreate` függvényébe:

```kotlin
emptyTV.setOnClickListener {
    showNewPlaceDialog()
}
```


### Dialógus és animációja

Az Android Lollipop verziójától lehetőségünk van képernyőátmenetek során elemeket megosztva animálni. Ebben az esetben azt fogjuk elérni, hogy amikor a felhasználó megérinti a FAB-ot, akkor az új helyszínt létrehozó képernyő abból animálódjon ki. A visszafelé portolás ebben az esetben nem tökéletes, 21-es API szint alatt ezt nem fogjuk látni. A megosztott animációhoz át kell írjuk a stílusainkat, azonban az ezekben használt új xml elemek csak API 21-től működnek.

Hozzunk létre egy új erőforrás mappát, a neve legyen `values-v21`! Az ebbe elhelyezett erőforrások fognak betöltésre kerülni, ha az eszköz 21-es, vagy újabb API szintű Androidot futtat. Ebben hozzunk létre egy `styles.xml` állományt, és az alkalmazás alaptémáját módosítsuk az alábbiaknak megfelelően:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">

        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>

        <!-- Customize your theme here. -->
        <item name="android:buttonStyle">@style/ButtonStyleRounded</item>

        <!-- enable window content transitions -->
        <item name="android:windowContentTransitions">true</item>

        <!-- enable overlapping of exiting and entering activities-->
        <item name="android:windowAllowEnterTransitionOverlap">true</item>
        <item name="android:windowAllowReturnTransitionOverlap">true</item>

    </style>

</resources>
```

Most meg kell adnunk az xml erőforrásokban, hogy miből mit szeretnénk animálni. Úgy párosítjuk össze a `View`-kat, hogy új attribútumot veszünk fel mind a *Floating Action Button*-höz, mind az `activity_create_place_to_visit.xml` gyökér eleméhez (ez egy `LinearLayout`). 

```xml
android:transitionName="create"
```

Ezután a `PlaceListActivity` `showNewPlaceDialog` metódusát egészítsük ki az alábbiak szerint:

```kotlin
private fun showNewPlaceDialog() {
    val options = ActivityOptionsCompat.makeSceneTransitionAnimation(
            this,
            addButton,
            "create")
    val intent = Intent(this, CreatePlaceToVisitActivity::class.java)
    startActivityForResult(intent, REQUEST_NEW_PLACE_CODE, options.toBundle())
}
```

Itt megadjuk, hogy melyik `View`-ból indul az animáció (`addButton`), és azt is, hogy milyen `transitionName`-mel kell dolgoznia a rendszernek (`"create"`). Az animációról szóló információt egy `Bundle`-lé alakítva tudjuk az `Intent`-be helyezni.

Próbáljuk ki az alkalmazást!

<img src="./images/screen4_framed.png" width="200" align="middle">


### Új hely felvétele

Nem a legjobb megoldás, hogy az alkalmazás második képernyője eredetileg dialógus stílusú. Töröljük a *Manifest*-ből és a stílusfájlokból a kapcsolódó stílusbejegyzést!

Próbáljuk ki az alkalmazást!

Az `Activity` nem tökéletes, hiszen ha nagyon hosszú leírást adunk neki, akkor a `Save` gomb kicsúszik a képernyőről és használhatatlan lesz. Rögzítsük tehát a gombot a képernyő aljára, a fölötte lévő tartalmat pedig tegyük görgethetővé!

Az `activity_create_place_to_visit.xml` gyökérelemét változtassuk `RelativeLayout`-ra, a benne lévő gombot pedig kössük az aljához:

```xml
android:layout_alignParentBottom="true"
```

A többi elemet pedig ágyazzuk be egy függőleges `LinearLayout`-ba, majd egy `ScrollView`-ba, amit kössünk felülre, és helyezzünk a gomb fölé:

```xml
<ScrollView
        android:layout_width="match_parent"
        android:layout_alignParentTop="true"
        android:layout_height="match_parent"
        android:layout_above="@+id/btnSave">
        
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
            
        <!-- A képernyő elemi a Save gombon kívül -->
        
        </LinearLayout>
        
</ScrollView>
```

### Snackbar

A `Toast` üzeneteknél már van egy sokkal szebb megoldás, ami a Material Designt követi: a [`Snackbar`](https://material.io/design/components/snackbars.html). Cseréljük le az alkalmazásban lévő `Toast` figyelmeztetéseket `Snackbar`-ra!

Ehhez írjunk egy külön `showText()` függvényt a `PlacesListActivity`-ben, ami a paraméterül kapott szöveges erőforrást jeleníti meg, majd használjuk ezt: 

```kotlin
private fun showText(@StringRes textRes: Int) {
    Snackbar.make(main_coordinator_layout, textRes, Snackbar.LENGTH_LONG).show()
}
```

A `Snackbar.make` függvény első paramétere egy `View`. Ide az `Activity`-nkben lévő `CoordinatorLayout`-ot adtuk meg. A függvény így hívható meg:

```kotlin
showText(R.string.cancelled)
```

Figyeljük meg, hogy a [`@StringRes`](https://developer.android.com/reference/android/support/annotation/StringRes) annotációnak hála az Android Studio ellenőrzi, hogy a paraméterként adott `Int` tényleg egy string erőforrás azonosítója, és például a `showText(42)` hívásra hibát jelez.

Próbáljuk ki a `Snackbar`-t!

<img src="./images/screen5_framed.png" width="200" align="middle">


## Önálló feladat

A fenti alapok segítségével alakítsa tovább az alkalmazást!

### Feladat 1 - Részletes nézet

Készítsen új képernyőt, ahol részletesen jeleníti meg az adott helyet!

### Feladat 2 - Lista javítása

Módosítsa a lista elemek leíróját (`row_place.xml`): `padding`, `margin`, `textStyle` és egyéb attribútumok segítségével tegye átláthatóbbá az elemek megjelenítését. 

### Feladat 3 - Swipe to delete

Valósítsa meg a swipe gesztussal való törlést (és esetleg módosítást). 

Példa megvalósítás: [https://medium.com/@ipaulpro/drag-and-swipe-with-recyclerview-b9456d2b1aaf](https://medium.com/@ipaulpro/drag-and-swipe-with-recyclerview-b9456d2b1aaf)

Egy másik példa megvalósítás: [https://medium.com/@kitek/recyclerview-swipe-to-delete-easier-than-you-thought-cff67ff5e5f6](https://medium.com/@kitek/recyclerview-swipe-to-delete-easier-than-you-thought-cff67ff5e5f6)
