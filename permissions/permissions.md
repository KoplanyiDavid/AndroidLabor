# Futásidejű engedélyek

## Bevezető

Android 6.0 (API level 23, Marshmallow) verziótól kezdve a felhasználó futásidőben adhatja meg, vagy utasíthatja el az alkalmazás által kért engedélyeket, és nem az alkalmazás telepítésekor vagy frissítésekor. Dönthet úgy, hogy bizonyos engedélyeket nem ad meg egy alkalmazásnak, így nagyobb fokú irányítás kerül a kezébe. Az alkalmazásengedélyeket később bármikor módosíthatja a rendszerszintű alkalmazás beállításoknál.

Az engedélyek két kategóriába vannak sorolva: *normal* és *dangerous*
A *normal* kategóriába tartozó engedélyek nem jelentenek közvetlen kockázatot a felhasználó személyes adataira, ezeket az engedélyeket a rendszer automatikusan megadja az alkalmazásnak, ha szüksége van rá.

A *dangerous* kategóriába tartozó engedélyek lehetőséget adhatnak az alkalmazásnak a felhasználó személyes adataihoz való hozzáféréshez. Ebben az esetben a felhasználónak kell megadni az engedélyt az alkalmazás számára. Ennek a közvetlen következménye az, hogy az alkalmazásokat fel kell készíteni arra az esetre, ha nincs megadva egy adott funkció működéséhez elengedhetetlen engedély.

[Ezen az oldalon](https://developer.android.com/guide/topics/permissions/overview#normal-dangerous)  található az összes engedély kategóriánként.

Az `AndroidManifest.xml` fájlban kategóriától függetlenül meg kell adni az alkalmazás számára szükséges összes engedélyt, de ennek hatása eltér a futtató rendszer verziójától és a target SDK verziótól függően:

* Ha az eszköz Android 5.1 (API level 22) vagy alacsonyabb verziót futtat, **VAGY** az alkalmazás target SDK szintje 22 vagy kisebb, akkor a rendszer telepítéskor kéri el az összes szükséges engedélyt. Ha a felhasználó nem fogadja el egyben az összes kérést, akkor a telepítési folyamat leáll.

* Ha az eszköz Android 6.0 (API level 23) vagy nagyobb verziót futtat **ÉS** az alkalmazás target SDK szintje 23 vagy nagyobb, akkor az alkalmazás a futása során fogja elkérni a *dangerous* kategóriába tartozó engedélyeket, a *normal* engedélyeket pedig a rendszer automatikusan megadja. Ebben az esetben a  felhasználó bármikor bármelyik engedélyt megadhatja, vagy visszavonhatja. Megtagadott engedélyekkel az alkalmazás limitált funkcionalitással futhat tovább, erre a helyzetre is fel kell készülni.

Megjegyzés: a Google Play 2018 augusztusától megköveteli a legalább 26-os target SDK verziót új alkalmazásokra, 2018 novemberétől pedig már meglévő alkalmazások frissítéseire is. Ezzel a legalább 6.0-s Androidot futtató eszközökön elkerülhetetlenné vált a futásidejű engedélyek kezelése.

## Jogosultság ellenőrzése

Amennyiben az alkalmazás egy funkciójának *dangerous* kategóriába eső engedélyekre van szüksége, akkor minden esetben ellenőrízni kell még a funkció indítása előtt, hogy rendelkezik-e az engedélyekkel, hiszen az engedélyeket a felhasználó bármikor módosíthatja.

Az ellenőrzés a `ContextCompat.checkSelfPermission()` függvény meghívásával végezhető, ami a `PackageManager.PERMISSION_GRANTED` értékkel tér vissza, ha az alkalmazás rendelkezik a vizsgált engedéllyel, és `PackageManager.PERMISSION_DENIED` értékkel egyébként.

Az alábbi kódrészlet a felhasználó naptárához való hozzáférési engedélyt ellenőrzi egy `Activity`-ben:

```kotlin
val permissionResult = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_CALENDAR)
when (permissionResult) {
    PackageManager.PERMISSION_GRANTED -> writeToCalendar(event)
    else -> requestPermissions()
}
```

## Jogosultság elkérése

Engedélyek elkérésére az `ActivityCompat.requestPermissions()` függvény meghívásával van lehetőség, aminek eredményeképp egy nem testreszabható, beépített dialógust jelenít meg a rendszer.

```kotlin
ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_CONTACTS), 
    MY_PERMISSIONS_REQUEST_READ_CONTACTS)
```

A `MY_PERMISSIONS_REQUEST_READ_CONTACTS` ebben a kódrészletben egy általunk definiált konstans. Az engedélykérés végén a rendszer ezt az értéket adja vissza `requestCode`-ként az `onRequestPermissionsResult()` callbackben. 

## Jogosultság kérés magyarázata

Egyes esetekben szükség lehet arra, hogy tájékoztassuk a felhasználót arról, hogy miért kér az alkalmazás bizonyos *dangerous* engedélyeket. Ez növelheti a felhasználó bizalmát az alkalmazással szemben.
 
Ha egy jogosultságot a felhasználó egyszer elutasított, az `ActivityCompat.shouldShowRequestPermissionRationale()` függvény visszatérési értéke alapján eldönthető, hogy a kérdéses engedély újra elkérése a rendszer szerint szorul-e részletes magyarázatra:

```kotlin
// Permission is not granted
// Should we show an explanation?
if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_CONTACTS)) {
    // Show an explanation to the user *asynchronously* -- don't block
    // this thread waiting for the user's response! After the user
    // sees the explanation, try again to request the permission.
} else {
    // No explanation needed, we can request the permission.
}
```

Így a következő engedélykérés előtt a felhasználó megtudja, hogy miért van szükségünk az engedélyre.

## Kezdő lépések

A labor során egy egyszerű telefonkönyv alkalmazást fogunk elkészíteni. Az alkalmazás meg fogja jeleníteni a telefonon tárolt névjegyeket, illetve egy névjegyre kattintással hívást kezdeményez az ahhoz tartozó elsődleges telefonszámra.

Hozzunk létre egy új projektet Android Studio-ban! Válasszuk az  *Empty Activity*-t, az alkalmazás neve legyen `Contacts`, a package név `hu.bme.aut.android.contacts`. A minimum SDK szint legyen *API 19: Android 4.4 (KitKat)*, a többi beállítást hagyjuk változatlanul.

A létrejött Activity-t nevezzük át (<kbd>Shift+F6</kbd>) `ContactsActivity`-re, valamint a felület leíróját is `activity_contacts`-ra.

Miután létrejött a projekt, vegyük fel a `RecyclerView` könyvtárat függőségként a modul szintű `build.gradle` fájlban:

```groovy
dependencies {
    implementation 'androidx.recyclerview:recyclerview:1.0.0'
}
```

Ugyanebben a fájlban ellenőrizzük, hogy a `targetSdkVersion` értéke legalább 23.

Kattintsunk a *Sync Now* gombra.

## Felhasználói felület

Készítsük el az alkalmazás felhasználói felületét a `res/layout/activity_contacts.xml` fájlban. A felület egyetlen `RecyclerView`-ból fog állni, mely az eszközön tárolt névjegyeket fogja megjeleníteni.

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_contacts"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingBottom="@dimen/activity_vertical_margin"
    tools:context=".ContactsActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rvContacts"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</FrameLayout>
```

Hozzuk létre a hiányzó erőforrásokat `16dp` értékkel.

## Model

Hozzunk létre egy `model` package-et és benne a `Contact` osztályt, ami egy eszközön található névjegyet fog reprezentálni. Az egyszerűség kedvéért most csak a név és telefonszám adatokat tároljuk el benne.

```kotlin
class Contact(
    val name: String,
    val number: String
)
```

## Listaelem

Hozzuk létre az `item_contact.xml` layout erőforrást:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:background="#dddddd">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <ImageView
            android:id="@+id/ivContactImage"
            android:layout_width="55dp"
            android:layout_height="55dp"
            android:layout_marginStart="10dp"
            android:src="@drawable/ic_contact_phone_black_48dp"/>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            android:gravity="center_vertical">

            <TextView
                android:id="@+id/tvContactName"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginStart="10dp"
                android:textSize="16sp"
                android:textColor="@android:color/primary_text_light"
                android:text="@string/contact_name_placeholder"/>

            <TextView
                android:id="@+id/tvPhoneNumber"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginStart="10dp"
                android:textSize="14sp"
                android:textColor="@android:color/primary_text_light"
                android:text="@string/contact_number_placeholder"/>

        </LinearLayout>

    </LinearLayout>

</RelativeLayout>
```

Hozzuk létre a `res/values/strings.xml` fájlban a két hiányzó szöveges erőforrást:

```xml
<string name="contact_name_placeholder">Name is not set</string>
<string name="contact_number_placeholder">Phone number is not set</string>
```

Töltsük le az [ic_contact_phone_black_48dp.png](assets/ic_contact_phone_black_48dp.png) képet és másoljuk be a `res/drawable` mappába.

## Adapter

Hozzunk létre egy `adapter` nevű package-et és benne készítsük el a listát feltöltő adaptert `ContactsAdapter` néven.

```kotlin
class ContactsAdapter : RecyclerView.Adapter<ContactsAdapter.ContactViewHolder>() {

    private val contactList = mutableListOf<Contact>()

    var itemClickListener: ContactItemClickListener? = null

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_contact, null)
        return ContactViewHolder(view)
    }

    override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
        val contact = contactList[position]
        holder.tvContactName.text = contact.name
        holder.tvPhoneNumber.text = contact.number
        holder.contact = contact
    }

    override fun getItemCount(): Int {
        return contactList.size
    }

    fun setContacts(contacts: List<Contact>) {
        contactList.clear()
        contactList += contacts
        notifyDataSetChanged()
    }

    inner class ContactViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val ivContactImage: ImageView = itemView.ivContactImage
        val tvContactName: TextView = itemView.tvContactName
        val tvPhoneNumber: TextView = itemView.tvPhoneNumber

        var contact: Contact? = null

        init {
            itemView.setOnClickListener {
                contact?.let { itemClickListener?.onItemClick(it) }
            }
        }
    }

    interface ContactItemClickListener {
        fun onItemClick(contact: Contact)
    }
    
}
```

Hozzunk létre egy `CursorExtensions` Kotlin fájlt a `util` csomagban, és definiáljuk benne az alábbi `extensions functiont`:

```kotlin
fun Cursor.getStringByColumnName(colName: String) = this.getString(this.getColumnIndex(colName))
```

Ez pusztán olvashatóbbá teszi majd a következő kódunkat. A `Cursor` API-ja csak oszlopindex alapján tud visszaadni adatot,
de nekünk oszlopnév alapján kellene, és ehhez először az indexet kell lekérdezni az oszlopnév alapján. Ez a függvény
ezt a két lépést fogja össze.

Adjuk hozzá a `ContactsActivity`-hez az alábbi, névjegyek lekérdezését megvalósító függvényeket:

```kotlin
private fun ContentResolver.performQuery(
        @RequiresPermission.Read uri: Uri,
        projection: Array<String>? = null,
        selection: String? = null,
        selectionArgs: Array<String>? = null,
        sortOrder: String? = null
): Cursor? {
    return query(uri, projection, selection, selectionArgs, sortOrder)
}

private fun getAllContacts(): List<Contact> {
    contentResolver.performQuery(
            uri = ContactsContract.Contacts.CONTENT_URI,
            sortOrder = "${ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME} ASC"
    ).use { contactResultCursor ->
        return if (contactResultCursor == null) {
            emptyList()
        } else {
            getContacts(contactResultCursor)
        }
    }
}

private fun getContacts(contactCursor: Cursor): List<Contact> {
    val contactList = mutableListOf<Contact>()

    while (contactCursor.moveToNext()) {
        val hasPhoneNumber = contactCursor.getString(contactCursor.getColumnIndex(ContactsContract.Contacts.HAS_PHONE_NUMBER)).toInt()
        if (hasPhoneNumber != 0) {
            val id = contactCursor.getStringByColumnName(ContactsContract.Contacts._ID)
            val name = contactCursor.getStringByColumnName(ContactsContract.Contacts.DISPLAY_NAME)

            val contactPhoneNumber = getContactPhoneNumber(id)

            contactList += Contact(name, contactPhoneNumber)
        }
    }

    return contactList
}

private fun getContactPhoneNumber(id: String): String {
    contentResolver.performQuery(
            uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
            selection = "${ContactsContract.CommonDataKinds.Phone.CONTACT_ID} = ?",
            selectionArgs = arrayOf(id)
    ).use { phoneResultCursor ->
        return if (phoneResultCursor == null || !phoneResultCursor.moveToNext()) {
            ""
        } else {
            phoneResultCursor.getStringByColumnName(ContactsContract.CommonDataKinds.Phone.NUMBER)
        }
    }
}
```

Ebben az alábbi részleteket érdemes megfigyelni:
- Bevezettünk egy saját `performQuery` függvényt, ami csupán továbbhív a `ContentResolver` már meglévő `query` függvényébe a neki átadott paraméterekkel. Rendelkezik viszont default paraméter értékekkel, hogy ne kelljen sok, nehezen átlátható `null`-t átadnunk a meghívásakor, valamint mivel ez a függvény Kotlinban van írva, ezért meg tudjuk hívni elnevezett paraméterekkel, ami javítja a kód olvashatóságát.
- A `getContacts` függvényben a `while` első sorában láthatjuk azt a megoldást a `Cursor`-ból való adat kiolvasásra, amit a SQLite laboron már használtunk.

A `ContactsActivity` `onCreate` függvényében írjuk meg a `RecyclerView` inicializálását:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_contacts)

    val contactsAdapter = ContactsAdapter()
    rvContacts.layoutManager = LinearLayoutManager(this)
    rvContacts.adapter = contactsAdapter
    contactsAdapter.setContacts(getAllContacts())
}
```

Jelezzük a rendszer felé, hogy az alkalmazásnak szüksége van engedélyre a névjegyek olvasásához. Ehhez vegyük fel az `AndroidManifest.xml` fájlban a `manifest` tagen belül az alábbi sort:

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

## Teszt

Egyelőre nem valósítottunk meg futásidejű jogosulságkezelést a kódban, ezért az alkalmazás működésének kipróbálásához Android 6.0 előtti verziót futtató eszközre, vagy emulátorra van szükség. Újabb verzió esetén hibát kapunk az alkalmazás indulása során.

Próbáljuk ki az alkalmazást 6.0/API 23 előtti verzióval rendelkező eszközön  vagy emulátoron!
Amennyiben az eszközön nincsenek névjegyek, adjunk hozzá legalább egyet telefonszámmal ellátva a beépített névjegykezelő alkalmazásban.

<img src="./assets/contacts.png" width="400" align="middle">

Android 6.0 vagy magasabb verzión futtatva az alkalmazást hibát kapunk, hiszen a névjegyek beolvasásához szükséges engedély a *dangerous* kategóriába tartozik, ezt külön kell kezelni kód szinten (6.0 felett **ÉS** targetSdk 23+ esetén).

A kapott hiba az alábbihoz hasonló:

```text
java.lang.SecurityException: Permission Denial: opening provider
com.android.providers.contacts.ContactsProvider2 from ProcessRecord{...} (pid=13547, uid=10141)
requires android.permission.READ_CONTACTS or android.permission.WRITE_CONTACTS
```

## Futásidejű jogosultságkezelés megvalósítása

Módosítsuk az alkalmazást úgy, hogy futási időben kérje el a felhasználótól a manifestben deklarált *dangerous* engedélyt! Ebben a bevezetőben ismertetett függvények lesznek segítségünkre.

Emeljük ki a `ContactsActivity` `onCreate` metódusában található alábbi sorokat egy függvénybe, melynek a neve legyen `loadContacts`!

```kotlin
val contactsAdapter = ContactsAdapter()
rvContacts.layoutManager = LinearLayoutManager(this)
rvContacts.adapter = contactsAdapter
contactsAdapter.setContacts(getAllContacts())
```

Ezt Android Studio-ban legegyszerűbben a kiemelni kívánt kód kijelölésével, majd a <kbd>Ctrl+Alt+M</kbd> billentyűkombinációval tudjuk megtenni. Ezzel a következő függvény kapjuk:

```kotlin
private fun loadContacts() {
    val contactsAdapter = ContactsAdapter()
    rvContacts.layoutManager = LinearLayoutManager(this)
    rvContacts.adapter = contactsAdapter
    contactsAdapter.setContacts(getAllContacts())
}
```

A kiemelés után az `onCreate` függvény:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_contacts)

    loadContacts()
}
```

Ahelyett, hogy az `onCreate` függvényben azonnal meghívnánk ezt a `loadContacts` függvényt, kérjünk engedélyt a felhasználótól a névjegyek olvasására!

Adjuk hozzá az alábbi függvényeket a `ContactsActivity`-hez!

```kotlin
private fun showRationaleDialog(
    @StringRes title: Int = R.string.rationale_dialog_title,
    @StringRes explanation: Int,
    onPositiveButton: () -> Unit,
    onNegativeButton: () -> Unit = this::finish
) {
    val alertDialog = AlertDialog.Builder(this)
        .setTitle(title)
        .setMessage(explanation)
        .setCancelable(false)
        .setPositiveButton(R.string.proceed) { dialog, id ->
            dialog.cancel()
            onPositiveButton()
        }
        .setNegativeButton(R.string.exit) { dialog, id -> onNegativeButton() }
        .create()
    alertDialog.show()
}

private fun handleReadContactsPermission() {
    if (ContextCompat.checkSelfPermission(this,
            Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
        // Should we show an explanation?
        if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_CONTACTS)) {

            // Show an explanation to the user *asynchronously* -- don't block
            // this thread waiting for the user's response! After the user
            // sees the explanation, try again to request the permission.

            showRationaleDialog(
                explanation = R.string.contacts_permission_explanation,
                onPositiveButton = this::requestContactsPermission
            )

        } else {
            // No explanation needed, we can request the permission.
            requestContactsPermission()
        }
    } else {
        loadContacts()
    }
}

private fun requestContactsPermission() {
    ActivityCompat.requestPermissions(
        this,
        arrayOf(READ_CONTACTS),
        PERMISSIONS_REQUEST_READ_CONTACTS
    )
}
```

A `PERMISSIONS_REQUEST_READ_CONTACTS` egy általunk definiálandó `requestCode`. Amikor engedélyt kérünk, meg kell adni egy `requestCode`-ot is. Amikor az operációs rendszer visszatér a kérés eredményével, a kérést indító `Activity` `onRequestPermissionsResult()` függvényében visszakapjuk ezt az értéket. Ez alapján tudjuk kezelni, hogy éppen melyik engedélykérésre érkezett válasz.

Bármilyen érték adható neki, csak ezen az aktuális alkalmazáson belül számít. Jelen esetben legyen 100.

A `ContactsActivity`-ben hozzuk létre a `PERMISSIONS_REQUEST_READ_CONTACTS` konstanst a `ContactsActivity` `companion object`-jében: 

```kotlin
companion object {
    private const val PERMISSIONS_REQUEST_READ_CONTACTS = 100
}
```

A `strings.xml`-be vegyük fel a hiányzó szöveges erőforrásokat:

```xml
<string name="rationale_dialog_title">Attention!</string>
<string name="contacts_permission_explanation">The application needs to access your contacts to display them.</string>
<string name="exit">Exit</string>
<string name="proceed">Proceed</string>
```

A `handleReadContactsPermission` függvényben a `checkSelfPermission` függvény segítségével megvizsgáljuk, hogy az alkalmazás rendelkezik-e a `READ_CONTACTS` engedéllyel. Ha igen, akkor meghívjuk a `loadContacts` függvényt, ami ezt az engedélyt használva betölti a névjegyeket. 

Ellenkező esetben megkérdezzük a rendszert, hogy a felhasználót kell-e tájékoztatni az engedélykérés létjogosultságáról (`shouldShowRequestPermissionRationale`). Ez a függvény akkor tér vissza `true` értékkel, ha a felhasználó korábban megtagadta az engedélyt az alkalmazástól. (Például azért, mert nem gondolta, hogy az adott funkcióhoz feltétlenül szükséges az engedély.) Ilyenkor érdemes egy magyarázatot adni, melyben leírjuk, hogy miért van feltétlenül szükség az engedélyre. Legyünk tömörek, a hosszú magyarázatokat a felhasználó nem fogja elolvasni, inkább letörli az alkalmazást. A magyarázat jelen esetben egy dialógus, mely rövid tájékoztatást ad az engedély szükségességéről.
Amennyiben nincs szükség magyarázatra, vagy a magyarázat dialógusablakában a *Proceed* gombra nyomott a felhasználó, akkor elkérjük az engedélyt (`requestPermissions`).

Cseréljük le a `ContactsActivity` `onCreate` függvényében a `loadContacts` hívást az előzőekben létrehozott `handleReadContactsPermission` hívásra:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_contacts)

    handleReadContactsPermission()
}
```

Kezeljük le az engedélykérés válaszát is úgy, hogy felülírjuk a `ContactsActivity` `onRequestPermissionResult` függvényét:

```kotlin
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
    when (requestCode) {
        PERMISSIONS_REQUEST_READ_CONTACTS -> {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // permission was granted, yay! Do the
                // contacts-related task you need to do.
                loadContacts()
            } else {
                // permission denied! Disable the
                // functionality that depends on this permission.
            }
            return
        }
    }
}
```

Amennyiben a felhasználó megadta az engedélyt, az alkalmazás betölti a névjegyeket.

Próbáljuk ki az alkalmazást 6.0+/API level 23+ eszközön! Figyeljük meg a magyarázó dialógust miután megtagadjuk az engedélyt, majd újraindítjuk az alkalmazást!

## Telefonszám hívása

Bővítsük a funkcionalitást úgy, hogy az alkalmazás egy adott névjegy elemre kattintás hatására megnyisson egy új Activity-t, ahonnan hívást tudunk indítani a névjegyben szereplő telefonszámra!

Először is a `loadContacts` függvény végén adjuk meg a `ContactsActivity`-t listenerként, hogy értesülni tudjunk az elem kiválasztásáról:

```kotlin
contactsAdapter.itemClickListener = this
```

Hozzunk létre egy új *Empty Activityt* `SingleContactActivity` néven! Ide majd át kell juttatnunk a névjegyben szereplő nevet és telefonszámot. Erre számos lehetőségünk nyílik, mi  most egyszerűen extraként az intent mellé fogjuk becsomagolni ezeket.

Módosítsuk úgy a `ContactsActivity`-t, hogy implementálja a `ContactsAdapter.ContactItemClickListener` interface-t, ezen tud szólni az adapter, hogy a lista valamelyik elemét kiválasztották:

```kotlin
class ContactsActivity : AppCompatActivity(), ContactsAdapter.ContactItemClickListener {...}
```

Ez után valósítsuk meg a `ContactsActivity`-ben az `OnItemClick` függvényt.


```kotlin
override fun onItemClick(contact: Contact) {
    val intent = Intent(this, SingleContactActivity::class.java)
    intent.putExtra(Contact.KEY_NAME, contact.name)
    intent.putExtra(Contact.KEY_NUMBER, contact.number)
    startActivity(intent)
}
```

Vegyük fel a két használt kulcsot a Contacts osztályba!

```kotlin
class Contact(
    val name: String,
    val number: String) {

    companion object{
        const val KEY_NAME = "KEY_NAME"
        const val KEY_NUMBER = "KEY_NUMBER"
    }
}
```

Készítsük el a `SingleContactActivity` felületét! Ez tartalmazni fog egy képet, a kontakt nevét valamint a kontakt telefonszámát. Alul található két gomb a híváshoz és az SMS küldéshez. Az `activity_single_contact.xml` tartalma:


```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_marginHorizontal="@dimen/activity_horizontal_margin"
    android:layout_marginVertical="@dimen/activity_vertical_margin"
    tools:context=".SingleContactActivity">

    <ImageView
        android:id="@+id/ivSingleContactImage"
        android:layout_width="wrap_content"
        android:layout_marginVertical="40dp"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true"
        android:src="@drawable/ic_contact_phone_black_48dp" />

    <TextView
        android:id="@+id/tvContactName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/ivSingleContactImage"
        android:layout_centerHorizontal="true"
        android:layout_marginVertical="@dimen/activity_vertical_margin"
        android:text="@string/contact_name_placeholder"
        android:textSize="30sp" />

    <TextView
        android:id="@+id/tvContactNumber"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/tvContactName"
        android:layout_centerHorizontal="true"
        android:layout_marginVertical="@dimen/activity_vertical_margin"
        android:text="@string/contact_number_placeholder"
        android:textSize="24sp" />

    <LinearLayout
        android:id="@+id/buttons"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:orientation="horizontal"
        android:weightSum="2">

        <Button
            android:id="@+id/buttonCall"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginHorizontal="@dimen/activity_horizontal_margin"
            android:layout_weight="1"
            android:text="@string/buttonCall" />

        <Button
            android:id="@+id/buttonSendSMS"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginHorizontal="@dimen/activity_horizontal_margin"
            android:layout_weight="1"
            android:text="@string/buttonSendSMS" />
    </LinearLayout>

</RelativeLayout>
```

Vegyük fel a szükséges erőforrásokat is:

```xml
    <string name="buttonCall">Call</string>
    <string name="buttonSendSMS">SMS</string>
```

A `SingleContactActivity` `OnCreate` függvényében be kell állítanunk a nevet és a telefonszámot. Ezt az intentből tudjuk kinyerni a *getStringExtra* függvénnyel:

```kotlin
class SingleContactActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_single_contact)

        tvContactName.text = intent.getStringExtra(Contact.KEY_NAME)
            ?: resources.getString(R.string.contact_name_placeholder)
        tvContactNumber.text = intent.getStringExtra(Contact.KEY_NUMBER)
            ?: resources.getString(R.string.contact_name_placeholder)
    }
}
```

### Futásidejű jogosultságkezelés

Ahhoz, hogy az alkalmazásunk hívásokat indíthasson, fel kell venni a következő engedélyt az `AndroidManifest.xml` fájlba a `READ_CONTACTS`-hez hasonlóan:

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

Ez az engedély is a *dangerous* kategóriába tartozik, ezért a telefonhívás indítását is futásidőben kell kezelnünk. Az imént áttekintettük a futásidejű engedélykérés és kezelés alapjait, azt hogy hogyan lehet ezt "kézzel" megoldani. A következőkben viszont bemutatjuk a [PermissionsDispatcher](https://github.com/permissions-dispatcher/PermissionsDispatcher) könyvtárat, ami annotációkkal könnyíti meg az engedélyek kezelését.

Ebben a következő annotációk érhetőek el (a vastaggal szedettek kötelezők):
* Először is el kell látnunk a **@RuntimePermissions** annotációval az Activityt vagy Fragmentet, hogy regisztrálni tudjuk az engedély kezeléséhez.
* Azt a függvényt, aminek a működéséhez szükség van az engedélyre, el kell látnunk a **@NeedsPermission** annotációval.
* Azt a függvényt, ami a szükséges magyarázatot jeleníti meg, el kell látnunk az *@OnShowRationale* annotációval
* Azt a függvényt, ami az engedély megtagadását kezeli le, el kell látnunk az *@OnPermissionDenied* annotációval
* Azt a függvényt, ami az engedély megadását, és a választás megjegyzését kezeli le, el kell látnunk az *@OnNeverAskAgain* annotációval

Kezdés képpen, vegyük fel a szükséges függőségeket a  modul szintű `build.gradle` fájlban. Először is szükségünk lesz a `PermissionsDispatcher` könyvtára, illetve az annotációk feldolgozásához a `PermissionsDispatcher processorra`. Ezt a kapt kulcsszóval tudjuk felvenni a függőségeink közé: 

```groovy
dependencies {
    implementation "org.permissionsdispatcher:permissionsdispatcher:4.6.0"
    kapt "org.permissionsdispatcher:permissionsdispatcher-processor:4.6.0"
}
```

Ahhoz, hogy a kapt kulcsszó is működjön és a gradle felismerje, engedélyeznünk kell a Kotlin Annotation Processing Toolt. Ezt szintén a  modul szintű `build.gradle` fájlban tudjuk bekapcsolni. Illesszük be a tetejére a következőt:

```groovy
apply plugin: 'kotlin-kapt'
```

Ezek után már használatra kész a `PermissionsDispatcher`. A futásidejű engedélyeket a `SingleContactActivity`-ben szeretnénk használni, ezért ezt először is lássuk el az osztályt a *@RuntimePermissions* annotációval.

A telefonhívást a *callPhone* függvényünk fogja végezni. Ebben összerakjuk az intentet a híváshoz, majd el is indítjuk:

```kotlin
@NeedsPermission(Manifest.permission.CALL_PHONE)
fun callPhoneNumber(phoneNumber: String) {
    val callIntent = Intent(Intent.ACTION_CALL)
    callIntent.data = Uri.parse("tel:$phoneNumber")
    startActivity(callIntent)
}
```

Mivel a hívásindítás *dangerous* engedély, a fordító jelzi is nekünk, lehetséges hogy a felhasználó nem adja meg az engedélyt, ekkor hibát kaphatunk. Ezért használjuk a *@NeedsPermission* annotációt. Ezzel a `PermissionsDispatcher` fordítás közben legenerálja nekünk a szükséges ellenőrzéseket. Már csak a lehetséges válaszokra kell felkészülnünk. Lehetséges, hogy a felhasználó megtagadja az engedélyt. Ekkor az *@OnPermissionDenied* annotációval ellátott függvény hívódik meg. Mi jelen esetben csak egy Toast-ot dobunk:

```kotlin
@OnPermissionDenied(Manifest.permission.CALL_PHONE)
fun onCallDenied() {
    Toast.makeText(this, getString(R.string.permission_denied_call), Toast.LENGTH_SHORT).show()
}
```

A string erőforrás értékének adjuk meg a "Call permission denied" szöveget.

Egy másik lehetőség, hogy a felhasználó egyszer már megtagadta az engedélyt, ekkor mutatnunk kell egy magyarázatot. Ekkor az *@OnShowRationale* annotációval ellátott függvény hívódik meg. Az előzekhez hasonlóan itt is egy *AlertDialog*-ot fogunk feldobni.

```kotlin
@OnShowRationale(Manifest.permission.CALL_PHONE)
fun showRationaleForCall(request: PermissionRequest) {
    val alertDialog = AlertDialog.Builder(this)
        .setTitle(title)
        .setMessage(R.string.call_permission_explanation)
        .setCancelable(false)
        .setPositiveButton(R.string.proceed) { dialog, id -> request.proceed() }
        .setNegativeButton(R.string.exit) { dialog, id -> request.cancel() }
        .create()
    alertDialog.show()
    }
```

A *showRationaleForCall* függvényünk bemenő paraméterként megkapja az engedélykérést. Így mind a pozitív, mind a negatív gombokra be tudjuk állítani ennek az eseménykezelését. Ehhez itt is lambda kifejezéseket használunk.

A string erőforrás értékének adjuk meg a "The application needs permission to make phone calls. It will not initiate calls without explicit user intention." szöveget.

Már csak egy lehetsőség maradt: mi van akkor, ha a felhasználó megadta az engedélyt. Korábban ezt az *onRequestPermissionsResult* függvénnyel kezeltük le. Tegyük most is ezt. Azonban a helyett, hogy megírnánk kézzel a lehetséges visszatéréseket, bízzuk ezt is a `PermissionsDispatcher`re:

```kotlin
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    // NOTE: delegate the permission handling to generated method
    onRequestPermissionsResult(requestCode, grantResults)
    }
```

Itt a kétparaméteres *onRequestPermissionsResult* függvény egy generált függvény, ami továbbadja a kezelést.

Ahhoz, hogy működjön a hívásunk, már csak egy dolgunk van. El kell készítenünk a hívás gomb eseménykezelőjét. Itt is egy generált függvényt fogunk hívni a telefonhíváshoz. a *callPhoneNumber* függvényünkből a `PermissionsDispatcher` a *callPhoneNumberWithPermissionCheck* függvényt generálja, ami elvégzi nekünk az engedélykérést. Tehát az eseménykezelő az *onCreate* végére:

```kotlin
...
buttonCall.setOnClickListener {
    callPhoneNumberWithPermissionCheck(tvContactNumber.text.toString())
}
```

Teszteljük a hívás funkciót 6.0+/API level 23+ emulátoron!

<img src="./assets/contacts.png" width="400" align="middle">

## Önálló feladatok

Valósítsa meg az SMS küldés funkciót! 

Az ehhez szükséges engedély:

```xml
<uses-permission android:name="android.permission.SEND_SMS"/>
```
