# Launcher++

## Bevezető

A labor célja az 4. laboron elkészített *Launcher* alkalmazás kibővítése.

A kiinduló projekt letölthető innen: [Kiinduló projekt](./assets/LauncherLabor_uptodate.zip) 

Az ötös eléréséhez a minimum követelményeken túl egy további, szabadon válaszott funkció beépítése szükséges. Az alkalmazások elkészítéséhez érdemes az óra anyagát és a példakódokat alapul venni az `android.telephony` csomag használata során.

[Telefónia Példa](./assets/TelefoniaPelda.zip) 

Két nézettel kell kibővíteni az alkalmazást (a `ViewPager`-t). A két új nézettel szemben támasztott követelményeket az alábbi fejezetek tartalmazzák.

## Hívásnapló

A feladat egy hívásnapló nézet megvalósítása.

A nézet legyen képes megjeleníteni a részletes hívásnaplót, külön jelezve az egyes hívás típusokat (bejövő, kimenő, nem fogadott). Extra feladatok lehetnek például:

* A hívásnapló ne csak a telefonszámokat mutassa, hanem nevet, képet is, amennyiben a névjegyzékben megtalálható
* Hívásnapló szűrése, rendezése

## Kedvencek nézet

Egy egyszerű lista nézet rajta egy hozzáadás gombbal. A hozzáadás gombra kattintva ki lehet választani egy contact-ot, majd sikeres kiválasztás után bekerül a listába a contact a telefonszámával együtt. Az adott elem jobb oldalán legyen egy hívás gomb, amire rákattintva fel lehet hívni az adott telefonszámot.

### Példa kód a contact kiválasztásához

Kontakt választó feldobása:

```kotlin
val intent = Intent(Intent.ACTION_PICK, ContactsContract.Contacts.CONTENT_URI)
startActivityForResult(intent, REQUEST_PICK_CONTACT)
```

Eredmény lekezelése:

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent) {
    super.onActivityResult(requestCode, resultCode, data)
    if (resultCode == Activity.RESULT_OK) {
        when (requestCode) {
            REQUEST_PICK_CONTACT -> {
                val contactUri = data.data ?: return
                val cursor = contentResolver.query(contactUri, null, null, null, null)
                cursor?.use { extractContact(it) }
            }
        }
    }
}

private fun extractContact(cursor: Cursor?) {
    if (cursor != null && cursor.moveToFirst()) {
        val id = cursor.getString(cursor.getColumnIndex(BaseColumns._ID))
        val name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME))
        Log.d("Launcher", "name: $name")
        val cursorPhone = queryPhoneData(id) ?: return
        cursorPhone.use { innerCursor ->
            if (innerCursor.moveToFirst()) {
                val contactNumber = innerCursor.getString(innerCursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER))
                Log.d("Launcher", "contactNumber: $contactNumber")
            }
        }
    }
}

private fun queryPhoneData(id: String): Cursor? {
    return contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
            arrayOf(ContactsContract.CommonDataKinds.Phone.NUMBER),
            ContactsContract.CommonDataKinds.Phone.CONTACT_ID + " = ? AND " +
                    ContactsContract.CommonDataKinds.Phone.TYPE + " = " +
                    ContactsContract.CommonDataKinds.Phone.TYPE_MOBILE,
            arrayOf(id),
            null)
}
```

A példa kód lekérdezi a kiválasztott contact-ot és megkeresi hozzá azt a telefonszámát, ami mobilként van megjelölve.

## Feltöltendő anyag

Az AUT portálra feltöltendő az alkalmazás projekt könyvtárán felül egy 2-3 oldalas felhasználói kézikönyv, ami leírja az elkészített szoftver funkcióit, és képeket is tartalmaz minden releváns képernyőről. A felhasználói kézikönyv is kötelező része a feladatnak, nélküle az anyag nem értékelhető!
