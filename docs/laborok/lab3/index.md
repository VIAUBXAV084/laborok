# Labor 3 - Ktor Room Koin - MyMovieWatchList

## Bevezető

A labor során egy film figyelő lista alkalmazást fogunk elkészíteni. Androidra, IOS-re és Desktopra. Az alkalmazásban lehet majd filmekre keresni, melyeket egy figyelő listára lehet helyezni.

Az filmek adatforrása az [TheMovieDb](https://www.themoviedb.org/) lesz, mely bizotosít egy REST-API-t filmek és sorozatok keresésére. Ehhez egy API kulcsot kell igényelni a [Developer](https://developer.themoviedb.org/docs/getting-started) weboldalon. Ehhez regisztrálni kell majd egy key fog megjelenni a képernyő alján. Erre később szükésgünk lesz.

3 fő technológia amely használva lesz a labor során:
 - [Ktor](https://ktor.io/) mely segítésgével fog történni a halózati kommunikáció
 - [Room](https://developer.android.com/kotlin/multiplatform/room) mely az lokális adatbázist fogja nyújtani
 - [Koin](https://insert-koin.io/) depency keretrendszer


## Laborfeladatok TODO

A labor során az alábbi feladatokat kell megvalósítani:

1. Pozíciómeghatározás megvalósítása: 1 pont
1. Térképes nézet megvalósítása: 2 pont
1. Wear alkalmazás megvalósítása: 2 pont
1. Extra: Google maps funkciók megvalósítása


## Előkészületek

A feladatok megoldása során ne felejtsd el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keresd meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozd létre a saját repository-dat.

2. Várd meg, míg elkészül a repository, majd checkout-old ki.

    !!! tip ""
        Egyetemi laborokban, ha a checkout során nem kér a rendszer felhasználónevet és jelszót, és nem sikerül a checkout, akkor valószínűleg a gépen korábban megjegyzett felhasználónévvel próbálkozott a rendszer. Először töröld ki a mentett belépési adatokat (lásd [itt](../../tudnivalok/github/GitHub-credentials.md)), és próbáld újra.

3. Hozz létre egy új ágat `megoldas` néven, és ezen az ágon dolgozz.

4. A `neptun.txt` fájlba írd bele a Neptun kódodat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

5. Indítsuk el az Android Studio-t vagy az IntellIj-t, majd nyissuk meg a projektet.

6. Ellenőrízzük, hogy a létrejött projekt lefordul és látható.

![Starter stage](assets/hello_world.png)

### A projekt áttekintése

A kezdő project már tartalmazza az alkalamzáshoz szükséges összes függőséget. Ezt a `composeApp`-ban található `build.gradle.kts` fájlban lehet megtekinteni.

3+1 fő source set van definiálva:
- CommonMain : Itt található az összes nem platform speicifikus függőség
- AndroidMain : Android-hoz tartozó függéségek
- IosMain : Iosh-ez tartozó függéségek
- DesktopMain : A Desktop alkalmazáshoz tartozó függőségek

## Feladatok

### 1. Képernyők létrehozása, navigáció

Hozzunk létre egy **navigation** packaget belül(kotlin/hu/bme/aut/navigation) azon belől egy **Route enum class**-t az alábbi tartalommal:

```kotlin
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Check
import androidx.compose.material.icons.filled.Search
import androidx.compose.ui.graphics.vector.ImageVector

enum class Route(val title: String, val icon: ImageVector) {
    Search(
        title = "Search",
        icon = Icons.Default.Search
    ),
    Watchlist(
        title = "Watchlist",
        icon = Icons.AutoMirrored.Filled.List
    )
}
```

Ez az osztály fogja definiálni az egyes az egyes képernyőket a navigáció számára. Ahogy látható 2 fő képernyőből fog állni az alkalmazás egy Search képernyő, ahol lehet majd keresni az filmekre és egy WatchList képernyő.

Cél szerű lenne, hogy az alkalamzás a 2 mobilplatformon hasonlítson, ellenben desktopon eltérő layout-ja legyen. Ezt jelenleg csak kód duplikációval tudnánk elérni, vagyis az iosMain és androidMain-ben duplikálva az ő megjelenésükért szükséges kódok duplán lennének.

Ha jobban megnézzük, definiálva van a kezdő projektben már egy mobileMain source set a `build.gradle.kts`-ben, melyre dependálnak az ios és az android platformok.

```kotlin
val mobileMain by creating {
        dependsOn(commonMain)
    }

val androidMain by getting {
    dependsOn(mobileMain) ---> így az android eléri a mobileMain kódját
}

val iosX64Main by getting
val iosArm64Main by getting
val iosSimulatorArm64Main by getting
val desktopMain by getting

val iosMain by creating {
    dependsOn(mobileMain) ---> így az ios eléri a mobileMain kódját
    iosX64Main.dependsOn(this)
    iosArm64Main.dependsOn(this)
    iosSimulatorArm64Main.dependsOn(this)
}
```

Hozzunk létre egy mobileMain directory-t a többi sourceset mellett.

![Mobile Sourceset](assets/mobile_sourceset.png)

Ezután szükségünk lesz egy platformok megjelenésért felelős kódra.
A ui packegen belül hozzunk létre egy PlatformLayout.kt-t az alábbi tartalommal.

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import hu.bme.aut.navigation.Route

@Composable
expect fun PlatformLayout(
    modifier: Modifier,
    navigationBarItems: List<Route>,
    selectedItem: Route,
    onNavigationItemClick: (Route) -> Unit,
    content: @Composable (modifier: Modifier) -> Unit
)
```
Itt kihasználjuk az **expect-actual** mechanizmusát a KMP-nek és a platfromok mondják megmondani, hogyan jelennek meg az egyes képernyők.

Alt+Enter segítségével hozzuk létre a desktop és a mobile sourceseteken az actuális implementációkat, ekkor létrejön
`PlatformLayout.desktop.kt` és a `PlatformLayout.mobile.kt` fájlok.

`PlatformLayout.desktop.kt` :
```kotlin
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.material.NavigationRail
import androidx.compose.material.NavigationRailItem
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import hu.bme.aut.navigation.Route

@Composable
actual fun PlatformLayout(
    modifier: Modifier,
    navigationBarItems: List<Route>,
    selectedItem: Route,
    onNavigationItemClick: (Route) -> Unit,
    content: @Composable (modifier: Modifier) -> Unit
) {
    Row(modifier) {
        NavigationRail(
            modifier = Modifier.fillMaxWidth(0.2f)
        ) {
            navigationBarItems.forEach {
                navigationItem ->
                NavigationRailItem(
                    selected = selectedItem == navigationItem,
                    onClick = { onNavigationItemClick(navigationItem) },
                    icon = {},
                    label = {
                        Text(text = navigationItem.title)
                    }
                )

            }

        }
        content(Modifier)
    }
}
```

`PlatformLayout.mobile.kt`:
```kotlin
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import hu.bme.aut.navigation.Route

@Composable
actual fun PlatformLayout(
    modifier: Modifier,
    navigationBarItems: List<Route>,
    selectedItem: Route,
    onNavigationItemClick: (Route) -> Unit,
    content: @Composable (modifier: Modifier) -> Unit
) {
    Scaffold(
        bottomBar = {
            NavigationBar {
                navigationBarItems.forEach {
                        navigationItem ->
                    NavigationBarItem(
                        label = {
                            Text(text = navigationItem.title)
                        },
                        onClick = {
                            onNavigationItemClick(navigationItem)
                        },
                        selected = navigationItem == selectedItem,
                        icon = {
                            Icon(navigationItem.icon, contentDescription = null)
                        }
                    )
                }
            }
        }
       { paddingValues ->
        content(Modifier.padding(paddingValues).fillMaxSize())
    }
}
```

Következő lépés a navigáció léptrehozása a két képernyő között.

Ehhez az alábbi kód részre lesz szükség az `App.kt` fájlba.
```kotlin
AppTheme {
        val navController = rememberNavController()
        val backStackEntry by navController.currentBackStackEntryAsState()

        val currentScreen = Route.valueOf(
            backStackEntry?.destination?.route ?: Route.Search.title
        )

        PlatformLayout(
            modifier = Modifier.fillMaxSize(),
            navigationBarItems = Route.entries,
            onNavigationItemClick = {
                navController.navigate(it.title)
            },
            selectedItem = currentScreen,
            content = { contentModifier ->
                NavHost(
                    modifier = contentModifier,
                    navController = navController,
                    startDestination = Route.Search.title,
                ) {
                    composable(route = Route.Search.title) {
                        Text(text = Route.Search.title)
                    }
                    composable(route = Route.Watchlist.title) {
                        Text(text = Route.Watchlist.title)
                    }
                }
            }
        )
    }
```
Létrehozunk egy navControllert, mely feladat a navigálás a képernyők között, továbbá egy navHost-ot megy navController által közölt változások hatására cseréli a képernyőket, melyek jelenleg a 2 képernyő neve.

![Navigation](assets/navigation.png)

## 2. Hálózati kommunkáció: Ktor , Dependecy injection : Koin
Következő lépés az Api Kliens kialakítása Ktor segítségével.

Hozzunk létre egy data packaget és azon belül egy network packet, melyben a hálózati kommukációhoz tartozó kódok lesznek.

Egy `TMDbResponse.kt` fájlba az alábbi Api response reprezentáló osztályokat.
Itt látható egy filmről elfogjuk tárolni az azonsítóját, címét és posterjéhez tartozó path-t, mely nem minden film esetén bizotsított, ezért nullable.

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class MovieResponse(
    val id: Int,
    val title: String,
    @SerialName("poster_path") val posterPath: String? = null,
)

@Serializable
data class TMDbResponse(
    val results: List<MovieResponse>
)
````

Továbbá hozzunk létre egy `TMDBApiClient.kt` fájlt mely segítésgével fognak az Api hívások történni.
```kotlin
import io.ktor.client.*
import io.ktor.client.call.*
import io.ktor.client.request.*
import io.ktor.client.statement.*

class TMDBApiClient(private val client: HttpClient) {
    private val baseUrl = "https://api.themoviedb.org/3"

    suspend fun getPopularMovies(): List<MovieResponse> {
        val response: HttpResponse = client.get("$baseUrl/movie/popular") {
            parameter("language", "en-US")
            parameter("page", 1)
        }

        return response.body<TMDbResponse>().results
    }

    suspend fun searchMovie(title : String): List<MovieResponse> {
        val response: HttpResponse = client.get("$baseUrl/search/movie?query=$title") {
            parameter("language", "en-US")
            parameter("page", 1)
        }

        return response.body<TMDbResponse>().results
    }
}
```

Az egyszerűség kedvéért az alkalmazásban csak két api hívás lesz, a **getPopularMovies()** az TheMovieDB adatbázisban eltárolt aktuálisan legnépszerübb filmeket fogja visszaadni. A **searchMovie(title : String)** segítésével lehet majd szövegesen keresni.

Következő lépés az adott platformoknak megfelelő HttpClient létrehozása.
A Network packageben hozzunk létre egy `ProvideApiBaseClient.kt` fájlt az alábbi tartalommal.

```kotlin
import io.ktor.client.*
import io.ktor.client.plugins.auth.Auth
import io.ktor.client.plugins.auth.providers.BearerTokens
import io.ktor.client.plugins.auth.providers.bearer
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.plugins.logging.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

expect fun platformClient(): HttpClient

fun provideHttpBaseClient() = platformClient().config {
    install(ContentNegotiation) {
        json(
            Json {
                ignoreUnknownKeys = true
                encodeDefaults = false
            }
        )
    }
    install(Auth) {
        bearer {
            loadTokens {
                BearerTokens(
                    "IDE_AZ_IGÉNYELT_API_KULCS",
                    null
                )
            }
        }
    }

    install(Logging) {
        logger = object : Logger {
            override fun log(message: String) {
                println("[HttpClient] $message")
            }
        }
        level = LogLevel.ALL
    }
}
```
**platformClient()** feladata elkérni az adott platformoktól a megfelelő **HttpClienteket**, majd ezen a config blockon belül pluginok hozzáadása. Jelen esetben, a hálózati kommunkáció során a Content JSON-ok segítségével fog történni. Az Api hívások során az igényelt API token az Authorization header-ben kell megadni, ehhez a Auth plugin van használva és Bearer típusú lesz a kommunikácó, továbbá pedig szeretnénk debuggolás érdekében a hálózati kommunikáció megjelenjen a console-on.

Alt+Enter segítésével hozzuk létre a platformoknak megfelelő **HttpClienteket** tartalmazó fájlokat. Jelen esetben androidMain, iosMain és desktopMain platformokra lesz szükgések.

`Client.ios.kt`:
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.darwin.*

actual fun platformClient(): HttpClient {
    return HttpClient(Darwin)
}
```

`Client.android.kt`:
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.okhttp.*

actual fun platformClient(): HttpClient {
    return HttpClient(OkHttp)
}
```

`Client.desktop.kt`:
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.okhttp.*

actual fun platformClient(): HttpClient {
    return HttpClient(OkHttp)
}
```

## Repository

Data packagen belül hozzunk létre egy **Repository** packaget, azon belül egy **MovieRepository** interfacet.

`MovieRepository.kt` :
```kotlin
import hu.bme.aut.domain.model.Movie
import kotlinx.coroutines.flow.Flow

interface MovieRepository {
    fun storedMovies() : Flow<List<Movie>>

    fun getMovies() : Flow<List<Movie>>

    suspend fun searchMoveByTitle(title : String)

    suspend fun getPopularMovies()

    suspend fun addMovieToWatchList(movie: Movie)

    suspend fun removeMovieFromWatchList(movie: Movie)
}
```

Továbbá hozzuk létre a Movie modelt. Ehhez a data package-el egyszinten készítsünk egy domain packaget.
A domain-en belül pedig model packaget benne egy Movie data class-al.

Movie.kt:
```kotlin
data class Movie(
    val id: Int,
    val title: String,
    val posterPath: String? = null,
    val onWatchlist: Boolean = false,
)
```

Ezután implementáljuk a **MovieRepository-t** hozzunk létre egy impl packaget a repository-n belül benne az alábbi osztállyal.
```kotlin
import hu.bme.aut.data.mapper.toMovieDomain
import hu.bme.aut.data.mapper.toMovieEntity
import hu.bme.aut.data.network.TMDBApiClient
import hu.bme.aut.data.repository.MovieRepository
import hu.bme.aut.domain.model.Movie
import kotlinx.coroutines.flow.*

class MovieRepositoryImpl(
    private val tMDBApiClient: TMDBApiClient,
) : MovieRepository {
    private val movieResultFlow = MutableStateFlow<List<Movie>>(emptyList())

    override fun getMovies(): Flow<List<Movie>> = movieResultFlow

    override suspend fun searchMoveByTitle(title: String) {
        val movies = tMDBApiClient.searchMovie(title).map { it.toMovieDomain() }
        movieResultFlow.update {
            movies
        }
    }

    override suspend fun getPopularMovies() {
        val movies = tMDBApiClient.getPopularMovies().map {
            it.toMovieDomain()
        }
        movieResultFlow.update {
            movies
        }
    }

     override fun storedMovies(): Flow<List<Movie>> {
        TODO("Not yet implemented")
    }

    override suspend fun addMovieToWatchList(movie: Movie) {
        TODO("Not yet implemented")
    }

    override suspend fun removeMovieFromWatchList(movie: Movie) {
        TODO("Not yet implemented")
    }
}
```

Beillesztés utána látható, hogy szükséges lesz még egy Mapper készítése a Api model -> Domain model között.
Data packen belül hozzunk létre egy mapper packaget azon belül az **MovieMapper** alábbi tartalommal.

```kotlin
import hu.bme.aut.data.network.MovieResponse
import hu.bme.aut.domain.model.Movie

fun MovieResponse.toMovieDomain(onWatchList: Boolean = false) = Movie(
    this.id,
    this.title,
    "https://image.tmdb.org/t/p/w500/${this.posterPath}",
    onWatchList,
)
```
Egy **MovieResponse** posterPath-je nem tartalmazza a teljes útvonalat a képekez, ez azért van, mert TheMovieDB más subdomaint biztosít az api a kép elérése, továbbá lehet felbontást is változtatni (w500) a kisebb hálózati forgalom érdekében.

## Koin
A Koin egy Kotlin Multiplaformban használható dependecy injection framework. Segítségével az alkalmazásban lévő függőségek közti kapcsolat lehet deklarálni. Ehhez Koin moduleokban kell a függőség kapcsolatokat deklarálni és az adott függőségek életciklusát pl. singleton-e az adott függőség.

Először hozzunk létre egy **di** packaget data/domain-el egyszinten. Ebben a packgeben fogjuk definiálni az összes függőség közötti kapcsolatot.

Hozzunk létre egy DataModule.kt az alábbi tartalommal:

```kotlin
import hu.bme.aut.data.network.TMDBApiClient
import hu.bme.aut.data.network.provideHttpBaseClient
import hu.bme.aut.data.repository.MovieRepository
import hu.bme.aut.data.repository.impl.MovieRepositoryImpl
import org.koin.core.module.dsl.singleOf
import org.koin.dsl.bind
import org.koin.dsl.module

fun dataModule() = module {
    single {
        provideHttpBaseClient()
    }
    single {
        TMDBApiClient(get())
    }
    singleOf(::MovieRepositoryImpl) bind MovieRepository::class
}
```

//TODO mit csinál ez

## UI folytatása
A hozzunk létre egy feature packaget benne hozzunk létre egy search packaget. Ide helyezzük majd a különböző képerőnyet és a hozzájuk tartozó viewmodelleket.

Most elkészítjük a keresés képernyőt és a hozzátartozó ViewModelt.

Hozzunk létre egy **SearchScreenViewModel-t**.

`SearchScreenViewModel.kt`:
```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import hu.bme.aut.data.repository.MovieRepository
import kotlinx.coroutines.FlowPreview
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.collectLatest
import kotlinx.coroutines.flow.debounce
import kotlinx.coroutines.launch

@OptIn(FlowPreview::class)
class SearchScreenViewModel(
    private val movieRepository: MovieRepository
) : ViewModel() {
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery

    val movies = movieRepository.getMovies()

    init {
        getPopularMovies()
        viewModelScope.launch {
            _searchQuery.debounce(500).collectLatest {
                if (it.isNotEmpty()) {
                    movieRepository.searchMoveByTitle(it)
                } else {
                    getPopularMovies()
                }
            }
        }
    }

    private fun getPopularMovies() {
        viewModelScope.launch {
            movieRepository.getPopularMovies()
        }
    }

    fun searchQuery(title: String) {
        _searchQuery.value = title
    }
}
```
Ahogy a kódból is látszódik a viewmodel a konstruktorában a repository-t. Mikor létrejön a viewmodel init{} blokk meghívja a repository getPopularMovies() függvényét, hogy megtörténjen a hálózati hívás, továbbá felirakozik egy belső flow-ra, mely a keresési szabadszöveges keresés Stringjét tárolja. Debounce 500ms pedig késlelteti a keresést.

Hozzuk létre a képernyőt `SearchScreen.kt` a viewmodellel egyszinten.

`SearchScreen.kt`
```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.material.MaterialTheme
import androidx.compose.material.OutlinedTextField
import androidx.compose.material.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import hu.bme.aut.ui.component.PlatformMovieList
import hu.bme.aut.ui.component.card.MovieCard
import hu.bme.aut.ui.theme.AppTypography
import org.koin.compose.viewmodel.koinViewModel

@Composable
fun SearchScreen(
    viewModel: SearchScreenViewModel = koinViewModel()
) {
    val movies by viewModel.movies.collectAsStateWithLifecycle(emptyList())
    val searchQuery by viewModel.searchQuery.collectAsStateWithLifecycle("")
    Column(
        modifier = Modifier.background(MaterialTheme.colors.background)
    ) {
        OutlinedTextField(
            textStyle = AppTypography.body2.copy(color = MaterialTheme.colors.primary) ,
            value = searchQuery,
            onValueChange = {
                viewModel.searchQuery(it)
            },
            label = { Text("Search Movies") },
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp),
            singleLine = true
        )
        PlatformMovieList(movies){
            MovieCard(it)
        }
    }
}
```

Ahogy a kódból is látszódik a Screen konstukrában a viewModel-t a **koin** fogja bizotsítani a koinViewModel() hívás által, ehhez még készítenünk kell egy koin Module-t, ahol viewModelek függőségeit fogjuk definiálni.

Szükséges lenne továbbá, hogy az alkalmazásunk listanézete eltérjen a két platformon. Mobile képernyőkön egymás alatt jelenjenek (LazyList) meg az egyes film kártyák, de desktopon gridesítve(listVerticalGrid).

Hozzunk létre egy `PlatformMovieList.kt` fájlt, ui/component packageben, feladata paraméterként kapott movie-k layoutjának megszabása, de az egyes Movie-k megjelenítése a lamdbaként kapott composable függvény feladata.

`PlatformMovieList.kt`
```kotlin
import androidx.compose.runtime.Composable
import hu.bme.aut.domain.model.Movie

@Composable
expect fun PlatformMovieList(
    movieList: List<Movie>,
    drawMovie : @Composable (Movie) -> Unit
)
```

Mivel egységes megjelenítés szeretnénk android-on és ios-en, ezért mobileMain-ben hozzuk létre az implementációt.

`PlatformMovieList.mobile.kt`
```kotlin
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.runtime.Composable
import hu.bme.aut.domain.model.Movie

@Composable
actual fun PlatformMovieList(
    movieList: List<Movie>,
    drawMovie: @Composable (Movie) -> Unit
) {
    LazyColumn {
        items(movieList){ movie ->
            drawMovie(movie)
        }
    }
}
```


`PlatformMovieList.desktop`

```kotlin
import androidx.compose.foundation.lazy.grid.GridCells
import androidx.compose.foundation.lazy.grid.LazyVerticalGrid
import androidx.compose.foundation.lazy.grid.items
import androidx.compose.runtime.Composable
import androidx.compose.ui.unit.dp
import hu.bme.aut.domain.model.Movie

@Composable
actual fun PlatformMovieList(
    movieList: List<Movie>,
    drawMovie: @Composable (Movie) -> Unit
) {
    LazyVerticalGrid(
        columns = GridCells.Adaptive(minSize = 120.dp)
    ) {
        items(movieList) { movie ->
            drawMovie(movie)
        }
    }
}
```

Továbbra szükséges létrehoznunk egy `MovieCard.kt` fájlt, ezt létrehozhatjuk. feature/movie package-ben, azért kerül ide mert később további funkcionalitást fog kapni a kártya.

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import coil3.compose.AsyncImage
import hu.bme.aut.domain.model.Movie

@Composable
fun MovieCard(
    movie: Movie
) {
    Box {
        Card(
            modifier = Modifier
                .padding(8.dp),
            elevation = CardDefaults.cardElevation(),
        ) {
            if(movie.posterPath != null) {
                AsyncImage(
                    model = movie.posterPath,
                    contentDescription = "Poster for ${movie.title}",
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(200.dp),
                )
            } else{
                Text(movie.title)
            }
        }
    }
}
```

Ezután cseréljük ki a Placeholder Text(text = Route.Search.title) composable-t a SearchScreen()-re

`App.kt`:
```kotlin
composable(route = Route.Search.title) {
    SearchScreen()
}
```

Jelen állapotban minden kész, ahhoz hogy lehessen használni az alkalamazást, ellenben mégsem fog működni.
A **DI** függőségeket kell még definiálnunk. Ehhez a **DI** package-en belül hozzunk létre egy **ViewModelModule-t** az alábbi tartalommal.

`ViewModelModule.kt`:
```kotlin
import hu.bme.aut.feature.search.SearchScreenViewModel
import org.koin.core.module.dsl.viewModel
import org.koin.dsl.module

fun viewModelModule() = module {
    viewModel {
        SearchScreenViewModel(get())
    }
}
```

Továbbá egy AppModule-t mely az dataModule és a viewModelModule-t tartalmazza. Továbbá egy initializeKoin függvény, mely átadja a koin számára a definiált függőségeket.

`AppModule.kt`:
```kotlin
import org.koin.core.context.startKoin
import org.koin.dsl.module

fun appModule() = module {
    includes(
        dataModule(),
        viewModelModule()
    )
}

fun initializeKoin() = startKoin {
    modules(appModule())
}
```

Ezután szükséges minden platformon meghívni az initializeKoin függvényt, mellyel "bekapcsoljuk" a koint.

`MainActivity.kt`:
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        initializeKoin()
        setContent {
            App()
        }
    }
}
```

`main.kt`:
```kotlin
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application
import hu.bme.aut.di.initializeKoin

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        title = "MyMovieWatchList",
    ) {
        initializeKoin()
        App()
    }
}
```

`MainViewController.kt`:
```kotlin
import androidx.compose.ui.window.ComposeUIViewController
import hu.bme.aut.di.initializeKoin

fun MainViewController() = ComposeUIViewController {
    initializeKoin()
    App()
}
```

Továbbá AndroidManifest fájlban adjuk hozzá a következő sort az engedélyekhez, hogy az alkalmazás hozzáférjen az internethez.

`AndroidManifest.xml`:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />

```

Jelen állapotban indítható az alkalmazás, és kereshetőek filmek.

![Api result](assets/api_result.png)

## 3. Perzisztens adattárolás: Room

Következő lépés az alkalmazás fejlesztésében a perzisztens adattárolás létrehozása. Ehhez a Room könyvtár lesz használva.
A Room ismerős lehet az Android fejlesztésből, a könyvtár a 2.7.0-alpha01 már Multiplatform könyvtárrá vált így használható Desktop és IOS platformokon is. A Room az SQLlite relációs adatbázis felett egy absztrakciós réteg(ORM), mellyel könnyen lehet relációs adatszerkezeteket felépíteni.
[Részletes leírása](https://developer.android.com/kotlin/multiplatform/room)

4 fő elemből áll a Room bekötése az alkalmazásba
- Movie Entitás definiálás, mely a Movie táblát fogja reprezentálni az adatbázisban, melyből 1-1 sor az adatbázisban fog 1-1 objektum példányt jelenteni.
- **Dao** definiálása, melyben definiáljuk, hogy 1-1 entitáson milyen műveleteink vannak.
- **RoomDatabase** osztályból leszármazás, melyen definiáljuk az entitiásainkat és azokhoz tartozó Dao osztályainkat
- Platform specifikus kódrészek implementálása

Először hozzunk létre az MovieEntitás, a **data/database/entity** packagen belül az alábbi tartalommal:

`MovieEntity.kt`:
```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity
data class MovieEntity(
    @PrimaryKey(autoGenerate = false) val id: Int,
    val title: String,
    val posterPath: String? = null,
    val onWatchlist: Boolean,
)
```
Ahogy látható **Room-ból** származó Annotációkkal van kiegészítve egy kotlin data class, mely által a **Room Compiler** tudni fogja, hogy az adott entitást kell használnia és ahogy látszódik nem a **Room** fogja generálni az adott filmek ID-t, hanem a külső adatforrás által kapott id-val fogjuk a lokálisan elmenteni az egyes filmeket.

A database packen belül hozzuk létre egy `MovieDatabase.kt` fájlt az alábbi tartalommal.

`MovieDatabase.kt`:
```kotlin
import androidx.room.*
import hu.bme.aut.data.database.entity.MovieEntity
import kotlinx.coroutines.flow.Flow

@Database(entities = [MovieEntity::class], version = 1)
@ConstructedBy(AppDatabaseConstructor::class)
abstract class MovieDatabase : RoomDatabase() {
    abstract fun getDao(): MyMoviesDao
}

@Suppress("NO_ACTUAL_FOR_EXPECT")
expect object AppDatabaseConstructor : RoomDatabaseConstructor<MovieDatabase> {
    override fun initialize(): MovieDatabase
}

@Dao
interface MyMoviesDao {
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(item: MovieEntity)

    @Query("SELECT * FROM $MOVIE_ENTITY")
    fun getAllAsFlow(): Flow<List<MovieEntity>>

    @Query("SELECT * FROM $MOVIE_ENTITY WHERE id = :id")
    suspend fun getMovieById(id: Int): MovieEntity?

    @Delete
    suspend fun delete(toMovieEntity: MovieEntity)

    companion object{
        const val MOVIE_ENTITY = "MovieEntity"
    }
}
```

Ahogy látszódik itt is **Room** könyvtárból érkező annotációk által van definiálva az MovieDatabase osztály ellenben abstract osztály ként. Itt a Compiler az implementációt létrehozza a háttérben fordítási időben. Továbbá AppDataBaseConstuctor esetén pedig az adott platformhoz tartozó initilize methodot.

**MyMoviesDao** interface-ből látható, hogy az adatbázisba hozzáadhatunk majd, törölhetünk, lekérdezhetünk és lesz egy feliratkoziásunk az adatbázis tartalmazára.

Itt felmerül a kérdés, hogy oké jelenleg csak egy abstract osztályunk van, amelyen keresztül nem tudjuk elérni a dao-t és műveleteket végezni, hogyan lesz ebből valós példány.

Hozzunk létre egy `DataBaseModule.kt` fájlt a di package-en belül, az alábbi tartalamonnal.

`DataBaseModule.kt`:
```kotlin
import androidx.room.RoomDatabase
import hu.bme.aut.data.database.MovieDatabase
import hu.bme.aut.data.database.MyMoviesDao
import hu.bme.aut.di.PlatformParameters
import org.koin.core.module.Module
import org.koin.dsl.module

expect fun databaseBuilder(platformParameters: PlatformParameters) : RoomDatabase.Builder<MovieDatabase>

fun dataBaseModule(platformParameters: PlatformParameters): Module = module {
    includes(
        module {
            single {
                databaseBuilder(platformParameters).build()
            }
        },
        module {
            single<MyMoviesDao> {
                get<MovieDatabase>().getDao()
            }
        }
    )
}

const val DB_FILE_NAME = "mymoviews.db"
```

Room esetén picit bonyolultab a megolást kell alkalmazni az adatbázis létrehozásakor, ez azért van mert egyes platformok esetén Platform specifikus adatokra is szükség van az adatbázis létrehozásakor(Android esetén az android context). Ez jelen esetben egy **PlatformParameters** osztályban lesz elrejtve, ezáltal a lehető legtöbb kódot lehet a közös kódban implementálni.

Hozzunk létre egy **PlatformParameters** osztályt a di package-en belül az alábbi tartalommal.

`PlatformParameters.kt`:
```kotlin
import org.koin.core.module.Module

expect class PlatformParameters {
    fun getModule(): Module
}
```

Jelen esetben Android-on kívül üres modulet fog visszaadni a Desktop és IOS implementáció.

`PlatformParameters.desktop.kt` és `PlatformParameters.ios.kt`:
```kotlin
import org.koin.core.module.Module
import org.koin.dsl.module

actual class PlatformParameters {
    actual fun getModule(): Module {
        return module {}
    }
}
```

`PlatformParameters.android.kt`:
```kotlin
import android.content.Context
import org.koin.core.module.Module
import org.koin.dsl.module

actual class PlatformParameters(
    val context : Context
) {
    actual fun getModule(): Module {
        return module {
            single {
                context
            }
        }
    }
}
```

Ha ezek megvannak akkor létrehozthatjuk a platform specifikus implementációit a databaseBuilder függvénynek.

`DatabaseBuilde.android.kt`:
```kotlin
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import hu.bme.aut.data.database.MovieDatabase

actual fun databaseBuilder(platformParameters: PlatformParameters): RoomDatabase.Builder<MovieDatabase> {
    val dbFile = platformParameters.context.getDatabasePath(DB_FILE_NAME)
    return Room.databaseBuilder<MovieDatabase>(
        context = platformParameters.context.applicationContext,
        name = dbFile.absolutePath
    )
        .setDriver(BundledSQLiteDriver())
}
```

Ahogy látható az adatbázis létrehozásához szükséges az android application context.

`DatabaseBuilde.desktop.kt`:
```kotlin
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import hu.bme.aut.data.database.MovieDatabase
import hu.bme.aut.di.PlatformParameters
import org.koin.core.module.Module
import org.koin.dsl.module
import java.io.File

actual fun databaseBuilder(platformParameters: PlatformParameters): RoomDatabase.Builder<MovieDatabase> {
    val dbFile = File(System.getProperty("java.io.tmpdir"), DB_FILE_NAME)
    return Room.databaseBuilder<MovieDatabase>(
        name = dbFile.absolutePath,
    )
        .setDriver(BundledSQLiteDriver())
}
```

`DatabaseBuilde.ios.kt`:
```kotlin
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import hu.bme.aut.data.database.MovieDatabase
import hu.bme.aut.di.PlatformParameters
import org.koin.core.module.Module
import org.koin.dsl.module
import java.io.File

actual fun databaseBuilder(platformParameters: PlatformParameters): RoomDatabase.Builder<MovieDatabase> {
    val dbFile = File(System.getProperty("java.io.tmpdir"), DB_FILE_NAME)
    return Room.databaseBuilder<MovieDatabase>(
        name = dbFile.absolutePath,
    )
        .setDriver(BundledSQLiteDriver())
}
```

Most már csak DI app moduleba kell felvenni a database modulet és kiegészíteni a hívás láncot, hogy mindegyik platform, mikor inicializálja a koin-t átadja a PlatformParatmeters példányból egyet.

`AppModule.kt`:
```kotlin
fun appModule(platformParameters: PlatformParameters) = module {
    includes(
        dataBaseModule(platformParameters),
        dataModule(),
        viewModelModule()
    )
}

fun initializeKoin(platformParameters: PlatformParameters) = startKoin {
    modules(appModule(platformParameters))
}
```

Desktop és IOS platformon elég csak egy létrehozni egy PlatformParameters példányt.

Android esetén

`MainActivity.kt`:
```kotlin
initializeKoin(PlatformParameters(this)) ->elég a this mert Activity contextban vagyunk.
```


Következő lépés az adatbázis bekötése a **MovieRepository-be**.

Először szükséges lesz a MovieEntity-ből domain Movie modellre és vissza mappelés megírni.
Egészítsük ki a `MovieMapper.kt` fájlt az alábbi sorokkal.

`MovieMapper.kt`:
```kotlin
fun MovieEntity.toMovieDomain() = Movie(
    id, title, posterPath, onWatchlist
)

fun Movie.toMovieEntity() = MovieEntity(
    id, title,posterPath, onWatchlist
)
```

Következő lépés pedig a **MovieRepositoryImpl** bővítése, vagyis az adatbázisban tárol filmekre való feliratkozás hozzáadása. Továbbá a hozzáadás és törlés az adatbázisból.

`MovieRepositoryImpl.kt`:
```kotlin
override fun storedMovies() =
        moviesDao.getAllAsFlow().map {
            movieEntities -> movieEntities.map {
                it.toMovieDomain()
            }
        }

override suspend fun addMovieToWatchList(movie: Movie) {
    moviesDao.insert(movie.copy(onWatchlist = true).toMovieEntity())
}

override suspend fun removeMovieFromWatchList(movie: Movie) {
    moviesDao.delete(movie.toMovieEntity())
}
```

Majdnem kész vagyunk már, arra van még szükség, hogy ui-on megjelenjen a hozzáadó és a törlő gomb.
Ennek implementációjára 2 lehetőségünk van.

1. Minden  képernyő viewmodelje propagálja ezt a repository függvény majd egy lambdaként leadjuk az egyes film kártyáknak.

Rövid egyszerűsített kód példának:
 ```kotlin
class ScreenAViewModel(private val repository: Repository) {
    fun addMovieToWatchList(movie: Movie) {
        repository.addMovieToWatchList(movie)
    }
}

class ScreenBViewModel(private val repository: Repository) {
    fun addMovieToWatchList(movie: Movie) {
        repository.addMovieToWatchList(movie)
    }
}

@Composable
fun ScreenA(viewModel: ScreenAViewModel) {
    val movies by viewModel.movies.collectAsStateWithLifecycle()

    LazyColumn {
        items(movies) { movie ->
            MovieCard(
                movie = movie,
                addToWatchList = { viewModel.addMovieToWatchList(movie) }
            )
        }
    }
}

@Composable
fun ScreenB(viewModel: ScreenBViewModel) {
    val movies by viewModel.movies.collectAsStateWithLifecycle()

    LazyColumn {
        items(movies) { movie ->
            MovieCard(
                movie = movie,
                addToWatchList = { viewModel.addMovieToWatchList(movie) }
            )
        }
    }
}
 ```
 Ezzel a megoldással az a probléma, hogy kódduplikáció jelenik meg erre megoldás lehetne ha egy ős ViewModelbe rakni a közös függvényeket, de akkor minden képernyőn, ahol a hozzáadás törlés tulajdonsága kell a kártyának a MovieCard lambdainak be kellene külön leadni a viewmodel függvényeket.

 2. A képernyők kizárólag a MovieCard-ok megjelenítéséért felelnek, és a filmekhez kapcsolódó műveletek logikáját egy külön MovieCardViewModel-be helyezzük. Ezáltal csökkentjük a képernyők kódbeli összetettségét, miközben jobban elkülönítjük a függőségeket. Ez a megközelítés elősegíti a komponensek újrafelhasználhatóságát és egyszerűsíti a karbantarhatóságot. Hasonló elvet követ a [Decompose](https://github.com/arkivanov/Decompose) könyvtár is, amely lehetővé teszi az UI-elemek életciklusának és adatkezelésének függetlenítését a környező komponensektől.

Hozzuk létre a `MovieCardViewModel.kt`-t a MovieCard mellett.
```kotlin
class MovieCardViewModel(
    private val movieRepository: MovieRepository
) : ViewModel() {

    fun addToWatchlist(movie: Movie) {
        viewModelScope.launch {
            movieRepository.addMovieToWatchList(movie)
        }
    }

    fun removeFromWatchlist(movie: Movie) {
        viewModelScope.launch {
            movieRepository.removeMovieFromWatchList(movie)
        }
    }
}
```
Szükséges kiegészíteni a MovieCard composable-t a hozzátartozó viewmodel-el továbbá hozzáadás/törlésért felelős ui-al.
```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.unit.dp
import coil3.compose.AsyncImage
import hu.bme.aut.domain.model.Movie
import org.koin.compose.viewmodel.koinViewModel

@Composable
fun MovieCard(
    movie: Movie, viewModel: MovieCardViewModel = koinViewModel(),
) {
    Box {
        Card(
            modifier = Modifier
                .padding(8.dp),
            elevation = CardDefaults.cardElevation(),
        ) {
            if(movie.posterPath != null) {
                AsyncImage(
                    model = movie.posterPath,
                    contentDescription = "Poster for ${movie.title}",
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(200.dp),
                )
            } else{
                Text(movie.title)
            }
        }
        IconButton(
            modifier = Modifier.clip(CircleShape).background(MaterialTheme.colorScheme.primary),
            onClick = {
                if (movie.onWatchlist) {
                    viewModel.removeFromWatchlist(movie)
                } else {
                    viewModel.addToWatchlist(movie)
                }
            }
        ) {
            Icon(
                imageVector = (if (movie.onWatchlist) Icons.Default.Delete else Icons.Default.Add),
                contentDescription = if (movie.onWatchlist) "Remove from Watchlist" else "Add to Watchlist",
                tint = MaterialTheme.colorScheme.onPrimary
            )
        }
    }
}
```

Továbbá a ViewModel module-t szükséges kiegészíteni a MovieCardViewModel-el.
```koltin
fun viewModelModule() = module {
    viewModel {
        SearchScreenViewModel(get())
    }
    viewModel {
        MovieCardViewModel(get())
    }
}
```

Jelen állapotban indítható az alkalamzás és sikeresen mentődnek az filmek az adatbázisban, elleben a UI-on nem látjuk, hogy az elemek frissülnének, vagyis a hozzáadás gomb után törlés gombnak kéne megjelennie.
Ez azért van mert a UI-on lévő elemek az api hívás közvetlen eredményét mutatják és nincsenek vizsgálva, melyek vannak eltárolva az adatbázisban. Ez könnyen orvosolható már a repository rétegben.

Először szükséges lesz egy segédfüggvény, mely megkapja a api hívás eredményét továbbá az adatbázisban tárolt filmek eredményét, ha id alapján megtaláljuk az adott filmet akkor az onWatchList Boolean true értéket fog kapni.
Egészítsük ki a MovieRepositoryImpl-t az alábbi kódrésszel.

`MovieRepositoryImpl.kt`:
```kotlin
 private fun updateMovieResult(searchResult: List<Movie>, storedMovies: List<Movie>): List<Movie> {
    val storedMovieIds = storedMovies.map { it.id }.toSet()
    return searchResult.map { result ->
        result.copy(
            onWatchlist = result.id in storedMovieIds
        )
    }
}
```
A SearchMoviesViewModel jelen pillanatban movieResultFlow-ra van feliratkozva, vagyis annak flow a változására történik recomposition a UI-on, de szükséges lenne az adatábázis flow-ra történő feliratkozás esetén is változás, erre nyújt megoldást a **combine()** nyelvi elem, mely 2 flow-t alakít át egy flow-vá.

Változtassuk meg a **getMovies()** függvényt, mely már combine-t és a segédfüggvényt is alkalmazza.
```kotlin
override fun getMovies(): Flow<List<Movie>> =
        combine(movieResultFlow, storedMovies()) { searchResult, storedMovies ->
            updateMovieResult(searchResult, storedMovies)
        }
```

Újra forditás után már működni is fog hozzáadás és törlés gomb az egyes kártyákon.

![Add delete on cards](assets/add_delete_on_cards.png)

## Watchlist képernyő implementálása - önállófeladat.
Egészítsd ki a kódbázis, hogy a Watchlist képernyőn csak az adatbázisban tárolt filmek jelenjenek meg.

<details>
  <summary>📱 Kiegészítés - MacOs eszköz szükséges</summary>

  Próbáljuk ki az alkalmazást iOS platformon! Miután sikeresen lefordult az alkalmazás Android és Desktop platformon, nézzük meg, hogyan jelenik meg iOS-en.

  Ehhez Xcode-ban szükséges megnyitni az `iosApp` könyvtárat, majd a **Build** gombra kattintani egy kiválasztott iPhone szimulátoron.
  Ha mindent megfelelően csináltunk, egy funkcionalitásban megegyező alkalmazásunk lesz a többi platformmal.

  ![iOS](assets/ios.png)

</details>
