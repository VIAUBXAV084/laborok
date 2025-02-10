# Labor 2 - Compose Multiplatform alapú UI készítése

A mai laboron egy egyszerűbb cross-platform alkalmazást fogunk elkészíteni Kotlin Multiplatform (KMP), illetve Compose Multiplatform használatával. A labor nagy része vezetett, azonban önálló feladataitok is lesznek. Az előző laboron már megismert Fleetet fogjuk használni most is.

## Előkészületek

A feladatok megoldása során ne felejtsd el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozd létre a saját repository-dat.

2. Várd meg, míg elkészül a repository, majd checkout-old ki.

    !!! tip ""
        Egyetemi laborokban, ha a checkout során nem kér a rendszer felhasználónevet és jelszót, és nem sikerül a checkout, akkor valószínűleg a gépen korábban megjegyzett felhasználónévvel próbálkozott a rendszer. Először töröld ki a mentett belépési adatokat (lásd [itt](../../tudnivalok/github/GitHub-credentials.md)), és próbáld újra.

3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.

4. A `neptun.txt` fájlba írd bele a Neptun kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

5. Indítsuk el az Android Studio-t, majd nyissuk meg a projektet.

6. Ellenőrízzük, hogy a létrejött projekt lefordul és helyesen működik!

## A projekt létrehozása

Az előző laboron is megismert [projektvarázsló](https://kmp.jetbrains.com/) segítségével hozzatok létre egy új projektet a **web** és **desktop** platformok targetálásával. Mobilos platformokra most nem lesz szükségünk. A projektet töltsétek le, csomagoljátok ki a zipet, és nyissátok meg a mappát Fleetben.

## Az alkalmazásunk nagy vonalakban

Egy todo list appot fogunk készíteni, ami három képernyőből fog állni: _feladatok_, _egy feladat részletei_, illetve _új feladat hozzáadása_. A desktopra fogunk fókuszálni mint platformra, de az érdekesség kedvéért megnézzük majd, hogyan is működik az appunk weben (ti. ez még csak alpha verzióban van).

Az alkalmazásunk architetkúrájának alapját az MVVM (Model-View-Viewmodel) minta lesz, ami az Androidos fejlesztők egyik kedvence.

!!!info "Mi is az az architekturális minta?"
       Az architekturális minták célja, hogy megkönnyítsék a fejlesztők dolgát, mert adnak egy keretet az alkalmazásnak, illetve egy közös mentális modellt az alkalmazás fejlesztőinek

<p align="center">
<img src="./assets/mvvm.png" width="813">
</p>

Az alábbi ábra jól összefoglalja az MVVM alkotóelemeinek kapcsolatát. Magas szinten a View "tud" a ViewModelről, a ViewModel pedig "tud" a Modelről, de a Model és a ViewModel nem ismerik egymást, és a ViewModel nem "tud" a View-ról.

Az MVVM minta használatának főbb előnyei:

- Elválasztja a logikát és a megjelenítést: A Model és a ViewModel különválasztása lehetővé teszi a tisztább, könnyebben karbantartható kódot.
- Tesztelhetőség: A ViewModel könnyen tesztelhető, mivel nem függ közvetlenül a felhasználói felülettől.
- Újrafelhasználhatóság: A ViewModelben lévő logika más nézetek között is újrahasználható.
- Jobb kódstrukturálás: A nagyobb alkalmazások esetén könnyen átláthatóbbá és skálázhatóbbá teszi a kódot.
- Könnyű adatáramlás: Az adatok az alkalmazás rétegein keresztül tisztán áramlanak, minimalizálva a hibák lehetőségét.

### Minden működik?

A projektvarázsló egy demóprojektet hozott létre nekünk. Ahhoz, hogy megbizonyosodjunk róla, minden helyesen működik-e, próbáljuk meg futtatni az alkalmazásunkat mind desktop, mind pedig webes platformon.

!!!tip ""
    Ehhez a felső sorban található Play-gombra kell nyomni, majd kiválasztani a megfelelő konfigurációt - ez desktopnál a Desktop névre hallgat, míg webnél a WasmJS-re.

!!!note Futtatási idők
    A desktopos alkalmazás indulása viszonylag gyors szokott lenni, míg a webesnél - első indításkor - akár perceket is kellhet várni, amíg letöltögeti a Fleet a szükséges dependenciákat.

Ha minden rendben működik, akkor ki is törölhetjük a `Greeting.kt` fájlunkat a `composeApp\src\commonMain\kotlin\org\example\srp_fe` mappából.

!!!note
    Ha _Safe delete_ dialógusablakkal talákozunk, az ne zavarjon minket, nyugodtan töröljük a fájlt

## Model

Először azt definiáljuk, hogyan is nézzen ki a modellosztálya egy todonak, azaz feladatnak. Ehhez hozzunk létre egy `model` mappát, majd abba egy `Todomodel.kt` nevű fájlt ott, ahonnan kitöröltük az előbb a két fájlt (`composeApp\src\commonMain\kotlin\org\example\srp_fe`).

!!! tip ""
    Ehhez jobb klikkeljünk az `srp_fe` mappára, majd válasszuk a **New folder...** opciót.

 A fájl tartalma legyen az alábbi:

```kotlin
package org.example.srp_fe.model

data class TodoModel(
    val title: String,
    val description: String?,
    val isCompleted: Boolean,
)
```

A fentiek jó kiindulási alapot fognak biztosítani egy általános jellegű todohoz.

## View

Most, hogy megvan, mi is képzi részét egy todo-nak, készítsük el a felhasználói felületet, azaz a három screenünket!

### Todok listája

#### A lista szerkezete

Először hozzunk létre egy `view` mappát a `model`-lel egy szintben, majd készítsünk benne egy `TodoListScreen.kt` fájlt. Ez a fájl fogja tartalmazni a todo-k listás megjelenítéséért felelős kódot. Hozzunk létre benne egy Composable függvényt, ami a todo-k listáját fogja megjeleníteni. Egyelőre egy előre megadott elemekből álló listát fogunk használni.

!!!info "Mi az a Composable függvény?"
       A Composable függvény (vagy simán csak Composable) egy speciális függvény, a Jetpack Compose-ban használt alapvető komponens, amely UI elemeket jelenít meg. Az ilyen függvények deklaratív módon írják le, hogyan épüljön fel a képernyő, és automatikusan frissülnek, ha az adatok változnak. Onnan lehet őket felismerni, hogy `@Composable` annotációval vannak ellátva.


```kotlin
package org.example.srp_fe.view

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material.icons.filled.Info
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import org.example.srp_fe.model.TodoModel

@OptIn(ExperimentalUuidApi::class)
@Composable
fun TodoListScreen(
    onAddButtonClicked: () -> Unit
) {
    val todos = listOf(
        TodoModel(
            id = Uuid.random().toString(), //random id-t generálunk neki
            title = "Bevásárlás",
            description = "Tej, tojás, kenyér",
            isCompleted = false,
        ),
        TodoModel(
            id = Uuid.random().toString(),
            title = "Befejezni a labort",
            description = "Ne felejtsd el beírni a Neptun-kódodat",
            isCompleted = false,
        ),
        TodoModel(
            id = Uuid.random().toString(),
            title = "Megetetni a macskát",
            description = "Ez fontos!",
            isCompleted = false,
        )
    )

    // ez az oszlop tartalmazza a screenünk elemeit
    Column(modifier = Modifier.padding(16.dp)) {
        Row(
            // az alábbi két sor segít a sor tartalmát az ablak két oldalára igazítani:
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text("To-Do List", style = MaterialTheme.typography.h4)

            Button(onClick = { onAddButtonClicked() }) {
            Text("Add New Task")
        }
        }

        Spacer(modifier = Modifier.height(16.dp))


        LazyColumn(modifier = Modifier.fillMaxSize()) {
            items(todos) { todo ->
                TodoItemView(todo)
            }
        }
    }
}
```

Így már fordulni fog a projektünk, megjelenítve a todo-s listánk tartalmát.

!!!info "Mi is az a különös annotáció a függvényünkön?"
           Mivel Kotlin Multiplatformon az Uuid library még nem stabil kiadás, csak experimental, ezt jeleznünk kell abban a függvényben, amelyikben használni akarjuk.

!!!example "BEADANDÓ (1 pont)"
	Készíts egy **képernyőképet**, melyen látszik az elindított desktop vagy webes app, rajta a beégetett todokkal. A képet a megoldásban a repository-ba f1.png néven töltsd föl!


#### A listaelemek

Ideje elkészíteni a listaelemeket megjelenítő függvényt. Ehhez ugyanezt a fájlt kell módosítanunk. Adjuk az alábbi függvényt a fájl aljához:

```kotlin
@Composable
fun TodoItemView(todo: TodoModel) {
    // A Card szép megjelenést ad a todoinknak
    Card(
        modifier = Modifier.fillMaxWidth().padding(vertical = 4.dp),
        elevation = 4.dp // ennek hatására "kiemelkedik a térből" minden kártya
    ) {
        Row(modifier = Modifier.padding(16.dp)) {

            Checkbox(
                checked = todo.isCompleted,
                onCheckedChange = {
                    // ezt majd később
                },
                modifier = Modifier.align(CenterVertically)
            )

            Column(modifier = Modifier.weight(1f)) {
                // A todo címe és leírása
                Text(todo.title, style = MaterialTheme.typography.body1)
                todo.description?.let { Text(it, style = MaterialTheme.typography.body2) }
            }

            Spacer(modifier = Modifier.height(8.dp))

            // Egy sor ikonokkal a szerkesztéshez és törléshez
            IconButton(onClick = { /* todo adatainak megtekintése */ }) {
                Icon(Icons.Default.Info, contentDescription = "Details")
            }
            IconButton(onClick = { /*todo törlése*/ }) {
                Icon(Icons.Default.Delete, contentDescription = "Delete")
            }
        }
    }
}
```

Ezek után módosítsuk a `TodoListScreen` függvényünket, hogy a todo-inkat az újonnan létrehozott `TodoItemView` Composable függvény segítségével jelenítsük meg:

```kotlin
LazyColumn(modifier = Modifier.fillMaxSize()) {
            items(todos) { todo ->
                TodoItemView(todo) // ez változott
            }
        }
```

Indsítsuk el az appunkat, és figyeljük meg a változásokat. Figyeld meg, hogy bár a gomboknak még nincs semmi hatása, az animáció működik így is.

### Navigáció

Ahhoz, hogy az appunkban navigálni tudjunk, a Compose Multiplatformos Navigation libraryt fogjuk használni.

!!! warning "Vigyázat, alpha-verzió!"
       Az itt használt könyvtár még alpha verzióban van, ami azt jelenti, hogy bár működik, production appokban nem ajánlott a használata, mivel merülhetnek fel nem várt hibák, illetve még nagyban változhat a működése is a könyvtárnak.

Ehhez fel kell vennünk ezt a libraryt függőségként, azaz módosítanunk kell a `composeApp/build.gradle` fájlunkat. A `commonMain.dependencies` blokkjába vegyük fel az alábbi sort:

```kotlin
implementation("org.jetbrains.androidx.navigation:navigation-compose:2.8.0-alpha10")
```

A navigáció implementálásához az alábbi lépésekre lesz szükségünk:
- Fel kell sorolnunk az útvonalakat, amik a navigációs gráf részét fogják képezni
- Készítenünk kell egy `NavHostController` példányt, ami felelős lesz a navigációért
- Hozzá kell adnunk egy `NavHost` Composable-t
    - Egyrészt ennek tudjuk megadni az alkalmazásunk kezdőképernyőjét
    - Másrészt pedig ez fogja a navigációs gráfunkat is létrehozni

Soroljuk fel az útvonalakat egy `enum class` tagjaiként az` App.kt` fájlunkban (az eddigi tartalmát kitörölhetjük):


```kotlin
package org.example.srp_fe

import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController
import org.example.srp_fe.view.TodoListScreen

enum class TodoScreen(val title: String) {
    List(title = "List To-dos"),
    Add(title = "Add To-do"),
    Details(title = "Details"),
}
```

!!!info ""
       Erre a rengeteg importra persze nincs mind szükség ebben a rövid kódrészleben, azonban a későbbiekben mindent fel fogunk használni belőle.

Megadtuk a három leendő képernyőnkhöz tartozó route-ot, valamint az adott képernyő címét (ezt majd minden képernyő meg fogja jeleníteni).

Hozzunk létre egy Composable-t (ugyanebbe a fájlba), ami az ablak tetején fog megjelenni, és az adott képernyő címének megjelenítéséért, illetve az adott oldalról történő visszanavigálás biztosításáért lesz felelős:

```kotlin
@Composable
fun TodoAppBar(
    currentScreen: TodoScreen,
    canNavigateBack: Boolean,
    navigateUp: () -> Unit,
) {
    TopAppBar(
        title = { Text(currentScreen.title) }, // az oldal címe
        navigationIcon = {
            if (canNavigateBack) { // paraméterként kapjuk, hogy mikor lehet visszafele navigálni
                IconButton(onClick = navigateUp) {
                    Icon(
                        imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                        contentDescription = "Back arrow"
                    )
                }
            }
        }
    )
}
```

Most adjuk hozzá újra (még mindig az `App.kt` fájlban) a korábban kitörölt `App` Composable függvényünket:

```kotlin
@Composable
fun App(
    navController: NavHostController = rememberNavController()
) {
    // jelenlegi backstack lekérése
    val backStackEntry by navController.currentBackStackEntryAsState()
    // jelenlegi képernyő lekérése
    val currentScreen = TodoScreen.valueOf(
        backStackEntry?.destination?.route?.substringBefore("/") ?: TodoScreen.List.name
    )

    Scaffold(
        topBar = {
            TodoAppBar(
                currentScreen = currentScreen,
                canNavigateBack = navController.previousBackStackEntry != null,
                navigateUp = {
                    navController.navigateUp()
                }
            )
        }
    ) { innerPadding ->

        // az említett NavHost létrehozása
        NavHost(
            navController = navController,
            startDestination = TodoScreen.List.name, // ez lesz a kezdőképernyőnk
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
        ) {
            // itt rendeljük össze a route-okat a képernyőkkel
            composable(route = TodoScreen.List.name) {
                TodoListScreen(
                    onAddButtonClicked = {
                        navController.navigate(TodoScreen.Add.name)
                    },
                )
            }
            //további képernyők..
        }
    }
}
```

### Todo hozzáadása

Most az új todo-k hozzáadásáért felelős képernyőt fogjuk elkészíteni. Ehhez hozzunk létre egy `AddTodoScreen.kt` nevű fájlt a `view` mappánkban az alábbi tartalommal:

```kotlin
package org.example.srp_fe.view

import androidx.compose.foundation.layout.*
import androidx.compose.material.MaterialTheme
import androidx.compose.material.OutlinedTextField
import androidx.compose.material.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import org.example.srp_fe.model.TodoModel
import androidx.compose.material.*
import androidx.compose.runtime.Composable

@OptIn(ExperimentalUuidApi::class)
@Composable
fun AddTodoScreen() {
    // a cím és a leírás mezők értékei
    var title by remember { mutableStateOf("") }
    var description by remember { mutableStateOf("") }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("Add New Task", style = MaterialTheme.typography.h4)

        Spacer(modifier = Modifier.height(16.dp))

        // szövegdoboz a címnek
        OutlinedTextField(
            value = title, // a szövegdoboz értéke a title változó
            onValueChange = { title = it }, // az érték változásakor állítsuk be a megfelelő változót az új értékre
            label = { Text("Title") },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(8.dp))

        // szövegdoboz a leírásnak
        // [...]

        Spacer(modifier = Modifier.height(16.dp))

        // "Mentés" gomb
        Button(
            onClick = {
                //todo hozzáadása, visszanavigálás az előző képernyőre
                }
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Save Task")
        }
    }
}
```

Nézzük végig a forráskódot, és értelmezzük azt a kommentek segítségével!

A `var title by remember { mutableStateOf("") }` és `var description by remember { mutableStateOf("") }` sorok arra szolgálnak, hogy tárolják a felhasználó által bevitt adatokat, miközben biztosítják, hogy ezek az értékek ne veszjenek el a komponens újraalkotásakor. A `remember` megőrzi az állapotot, míg a `mutableStateOf` lehetővé teszi a változók módosítását és az UI újrarenderelését, amikor az értékek változnak. A Composable függvények ugyanis érzékelik, ha egy általuk felhasznált, `mutableState` típusú objektum változott, és ilyenkor újrarenderelik magukat.

A cím módosítására szolgáló szövegdobozt megadtuk, ennek alapján készítsd el a leírás módosítására szolgáló szövegdobozt (alá)!

!!! tip ""
    Az elegáns megoldáshoz használd a `maxlines` paraméterét is az `OutlinedTextField`-nek.

Ezután menjünk vissza az `App.kt` fájlba, és adjuk hozzá ezt a képernyőt is a megfelelő route-tal a `NavHost`-hoz:

```kotlin
NavHost(
            navController = navController,
            startDestination = TodoScreen.List.name,
            modifier = Modifier
                .fillMaxSize()
                .padding(innerPadding)
        ) {
            composable(route = TodoScreen.List.name) {
                TodoListScreen(
                    onAddButtonClicked = {
                        navController.navigate(TodoScreen.Add.name)
                    },
                )
            }
            composable(route = TodoScreen.Add.name) {
                AddTodoScreen()
            }
            //további képernyők..
        }
```

Indítsuk el az alkalmazásunk. Az _Add New Task_ hatására egy új képernyőre kell jutnunk, ahonnan a bal felső sarokban található gombra kattintva vissza tudunk navigálni.

!!!example "BEADANDÓ (1 pont)"
	Készíts egy **képernyőképet**, melyen látszik az elindított desktop vagy webes app "Add todos" képernyője, rajta a két szövegdobozzal. A _title_ szövegdobozba írd be a Neptun-kódodat. A képet a megoldásban a repository-ba f2.png néven töltsd föl!


### Todo részletei

Már csak egy képernyőt kell létrehoznunk; azt, amelyik a todo részleteinek megtekintéséért felelős. Készítsünk ehhez egy `TodoDetails.kt` fájlt a `view` mappában. A tartalma legyen az alábbi:

```kotlin
package org.example.srp_fe.view

import androidx.compose.foundation.layout.*
import androidx.compose.material.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import org.example.srp_fe.model.TodoModel
import androidx.compose.material.*

@Composable
fun TodoDetailsScreen(todo: TodoModel) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = "Task Details", style = MaterialTheme.typography.h4)

        Spacer(modifier = Modifier.height(16.dp))

        Text(text = "Title: ${todo.title}", style = MaterialTheme.typography.body1)

        Spacer(modifier = Modifier.height(8.dp))

        Text(text = "Description: ${todo.description}", style = MaterialTheme.typography.body1)

        Spacer(modifier = Modifier.height(16.dp))

        Text(text = "Completed: ${todo.isCompleted}", style = MaterialTheme.typography.body1)
    }
}
```

Ne feledjük el, hogy az új screenünket "be kell kötni" az `App.kt`-ban is. Ez eggyel bonyolultabb lesz, mint idáig, ugyanis a `TodoDetailsScreen` vár paraméterként egy todo-t, ami viszont attól függ, mi lesz, hogy a user mire kattint a listát tartalmazó screenünkön. Adjunk hozzá először is egy új paramétert a már meglévő `TodoScreen`-es composable-ünkhöz (ne aggódjunk, ha az IDE aláhúzza, hiszen a `TodoDetailsScreen`-en is fel kell majd még ezt a paramétert vennünk):

```kotlin
    composable(route = TodoScreen.List.name) {
                TodoListScreen(
                    onAddButtonClicked = {
                        navController.navigate(TodoScreen.Add.name)
                    },
                    // vezessük be ezt az új paramétert a navigációhoz
                    onTodoClicked = { todo ->
                        navController.navigate("${TodoScreen.Details.name}/${todo.id}")
                    }
                )
            }
```

Ezen kívül pedig adjunk hozzá egy új composable-t a `TodoDetailsScreen`-nek:

```kotlin
    composable(
                route = "${TodoScreen.Details.name}/{todoId}",
                arguments = listOf(navArgument("todoId") { type = NavType.StringType })
            ) { backStackEntry ->
                val todoId = backStackEntry.arguments?.getString("todoId")
                val todo = todoList.firstOrNull { it.id == todoId }  // a todoListet még meg kell szereznünk a ViewModelünktől

                if (todo != null) {
                    TodoDetailsScreen(
                        todo = todo
                    )
                } else {
                    Text("Todo not found")
                }
            }
```

Ez a megoldás megpróbálja megszerezni a megfelelő Id-jű todo-t a ViewModelünktől, amit még csak most fogunk elkészíteni, szóval nem gond, ha hibát jelez az IDE.

Ezen kívül még a `TodoListScreen`-t is módosítani kell, hogy az újonnan létrehozott képernyőt meg is tudjuk nyitni. Az alábbi módosításokat végezzük el:

```kotlin
fun TodoListScreen(
    onAddButtonClicked: () -> Unit,
    onTodoClicked: (TodoModel) -> Unit // az új paraméter
)
```

```kotlin
@Composable
fun TodoItemView(todo: TodoModel, onTodoClicked: (TodoModel) -> Unit = {}) { //adjuk hozzá a callbacket paraméterként
```

```kotlin
IconButton(onClick = { onTodoClicked(todo) }) { // hívjuk meg itt a callbacket
                Icon(Icons.Default.Info, contentDescription = "Details")
            }
```

```kotlin
LazyColumn(modifier = Modifier.fillMaxSize()) {
            items(todos) { todo ->
                TodoItemView(todo, onTodoClicked) // adjuk át a callbacket a TodoItemView-nek
            }
        }
```

Ha ezekkel megvagyunk, a `TodoListScreen`-en nem kéne hibaüzenetet látnunk (az `App.kt` fájlban azonban még igen).

## ViewModel

Hozzunk létre egy `viewmodel` mappát a `model` és `view` mappák szintjén (tehát a `composeApp\src\commonMain\kotlin\org\example\srp_fe` mappába), majd hozzunk létre egy új fájlt benne `TodoViewModel.kt` néven:

```kotlin
package org.example.srp_fe.viewmodel

import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.mutableStateOf
import androidx.lifecycle.ViewModel
import org.example.srp_fe.model.TodoModel
import kotlin.uuid.ExperimentalUuidApi
import kotlin.uuid.Uuid

class TodoViewModel : ViewModel() {

    private val _todoList = mutableStateListOf<TodoModel>() // a lista elemei módosíthatóak
    val todoList: List<TodoModel> get() = _todoList // ...de csak ebből az osztályból

    private val titleState = mutableStateOf("")
    private val descriptionState = mutableStateOf("")

    @OptIn(ExperimentalUuidApi::class)
    fun addTodo() {
        if (titleState.value.isNotBlank()) {
            val newTodo = TodoModel(
                id = Uuid.random().toString(),
                title = titleState.value,
                description = descriptionState.value,
                isCompleted = false
            )
            _todoList.add(newTodo)

            titleState.value = ""
            descriptionState.value = ""
        }
    }

    fun updateTodo(todo: TodoModel){
        val index = _todoList.indexOf(todo)
        if (index != -1) {
            _todoList[index] = todo
        }
    }

    // így lehet a legegyszerűbben átállítani majd egy todo isCompleted állapotát
    fun toggleTodoCompletion(todo: TodoModel) {
        val index = _todoList.indexOfFirst { it.id == todo.id }
        if (index >= 0) {
            _todoList[index] = todo.copy(isCompleted = !todo.isCompleted)
        }
    }

    fun deleteTodo(todo: TodoModel) {
        _todoList.remove(todo)
    }
}
```

A ViewModelünk tartalmaz függvényeket a todo-k törléséhez, frissítéséhez és hozzáadásához is, valamint tárolja a todo-inkat egy listában. A lista hozzáférhető ugyan kívülről, de csak olvasásra, írásra nem.

Használjuk is fel ezeket a függvényeket az arra megfelelő helyeken!

Először az `App.kt`-t fogjuk módosítani. Adjuk hozzá az importokhoz az alábbi sort:
```kotlin
import org.example.srp_fe.viewmodel.TodoViewModel
```

Majd módosítsuk az `App` függvényünk fejlécét, hogy hozzunk létre egy viewModelt, amit aztán használni tudunk:
```kotlin

    @Composable
    fun App(
        navController: NavHostController = rememberNavController(),
        viewModel: TodoViewModel = viewModel { TodoViewModel() } // adjuk ezt hozzá
    )
```
!!!info "Miért ilyen furcsán példányosítjuk a viewModelt?"
       Azért, mert ez nem egy sima példányosítás. Ez egy speciális módja annak a Kotlinban, hogy létrehozzunk egy lifecycle-aware viewmodelt, vagy lekérjük azt, ha már létezik a scope-unkban. Ez a rekompozícióknál tud többek között nagyon hasznos lenni (nem hozunk létre új viewmodelt a UI minden egyes újrarajzolásakor).

Adjunk hozzá két új paramétert a `TodoListScreen` composable-jéhez az alábbiak szerint:

```kotlin
        composable(route = TodoScreen.List.name) {
                TodoListScreen(
                    todos = viewModel.todoList,
                    onAddButtonClicked = {
                        navController.navigate(TodoScreen.Add.name)
                    },
                    onDeleteClicked = { todo ->
                        viewModel.deleteTodo(todo)
                    },
                    onTodoClicked = { todo ->
                        navController.navigate("${TodoScreen.Details.name}/${todo.id}")
                    },
                )
            }
```
Majd adjunk hozzá egy új paramétert az `AddTodoScreen` copmosable-jéhez is:
```kotlin
 composable(route = TodoScreen.Add.name) {
                AddTodoScreen(
                    onAddTodo = { // új callback paraméter - hozzáadjuk a todo-t, majd visszanavigálunk
                        viewModel.addTodo(it)
                        navController.navigateUp()
                    }
                )
            }
```
Végül a `TodoDetailsScreen` composable-jében is adjunk hozzá egy új sort:

```kotlin
                val todoId = backStackEntry.arguments?.getString("todoId")
                val todoList = viewModel.todoList // hozzuk létre ezt a változót
                val todo = todoList.firstOrNull { it.id == todoId }
```

Ha ezzel megvagyunk, az újonnan hozzáadott paramétereket adjuk hozzá a `TodoListScreen.kt` fájlban, hogy így nézzen ki a függvény fejléce:

```kotlin
@OptIn(ExperimentalUuidApi::class)
@Composable
fun TodoListScreen(
    todos: List<TodoModel>,
    onDeleteClicked: (TodoModel) -> Unit,
    onAddButtonClicked: () -> Unit,
    onTodoClicked: (TodoModel) -> Unit
)
```

Ezen felül töröljük ki a hardcoded `val todos` változót is, hiszen már paraméterként kapjuk a todo-kat.

Próbáljuk ki az alkalmazásunkat! Hozzunk létre egy todo-t, majd győződjünk meg róla, hogy az valóban megjelent a listában.

Szükségünk lesz még arra, hogy a kuka gombra kattintva valóban törlődjön az adott todo. Ennek a megvalósítása innen már egyszerű, ez önálló feladat.

!!!example "BEADANDÓ (2 pont)"
	Készíts egy **képernyőképet**, melyen látszik az elindított desktop vagy webes app, mellette pedig a `TodoItemView` függvény a feladat megoldásával. A képet a megoldásban a repository-ba f3.png néven töltsd föl!

Egész jól áll már az appunk, csak épp egy dolgot nem tudunk benne csinálni - jelölni, hogy elkészültünk egy todo-val. Ehhez szerencsére a legtöbb dolog már rendelkezésünkre áll.
A `TodoListScreen` függvény, illetve a `TodoItemView` függvény fejléceibe (mindkettőbe!) vegyük fel az alábbi callbacket:
```kotlin
    onCheckedChanged: (TodoModel) -> Unit
```

Adjuk át ezt a callbacket a `LazyColumn`-ban:

```kotlin
TodoItemView(todo, onTodoClicked, onDeleteClicked, onCheckedChanged) // adjuk át a callbacket
```

Majd hívjuk meg, amikor a user a checkboxra nyom:

```kotlin
        Checkbox(
                checked = todo.isCompleted,
                onCheckedChange = {
                    onCheckedChanged(todo)
                },
                modifier = Modifier.align(CenterVertically)
            )
```

Készen is vagyunk. Indítsuk el az alkalmazást valamelyik platformon, vegyünk fel néhány todo-t, és állítsuk be valamelyiket készre. Az egyik todo neve legyen a Neptun-kódod.

!!!example "BEADANDÓ (2 pont)"
	Készíts egy **képernyőképet**, melyen látszik az elindított desktop vagy webes app, amiben fel van véve néhány todo, melyek közül be is van pipálva néhány. Az egyik todo neve legyen a Neptun.kódod. A képet a megoldásban a repository-ba f4.png néven töltsd föl!


utolsó feladat: "update" gomb megvalósítása önállóan - nem tudom, ez szorgalmi vagy kötelező legyen, vagy esetleg egyáltalán nem kell, szerinted? a viewmodelben megvan már hozzá a függvény, meg az "add" screenjét is lehet használni végül is, de egy totál kezdőnek nehéz lehet