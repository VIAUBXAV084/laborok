# Labor 3 - Hálózati kommunikáció és adatkezelés

## Bevezető

A labor során egy filmfigyelő lista alkalmazást fogunk elkészíteni Android, IOS és Desktop platformokra. Az alkalmazásban lehet majd filmekre keresni, melyeket egy figyelő listára lehet helyezni. Figyelmet fordítunk továbbá a projekt precíz strukturálására is (packagek szerkezetének kialakítása), hogy lássunk példát egy összetettebb, nagyobb projektekben átláthatóbban használható megközelítésre is. Bár a projektünkben önmagában ez nem lenne mindig indokolt, mert viszonylag kevés fájlt fognak tartalmazni a packagek. Ha az alkalmazásunk bonyolultabb lenne és jóval több funkcióból állna, akkor kifizetődőbb lenne, mint egy sokkal egyszerűbb megközelítés. Az egyszerűség kedvéért, hogy könnyebben lehessen követni a packagek hierarchiáját, új package létrehozásakor mindig megadjuk zárójelben a package teljes, fully qualified nevét.

A filmek adatforrása a [TheMovieDb](https://www.themoviedb.org/) lesz, mely biztosít egy REST-API-t filmek és sorozatok keresésére. Ehhez egy API kulcsot kell igényelni a [Developer](https://developer.themoviedb.org/docs/getting-started) weboldalon. Ehhez regisztrálni kell, majd egy key fog megjelenni a képernyő alján. Erre később szükségünk lesz. **Amennyiben nem szeretnénk a  regisztrációval és az API kulcs igényléssel foglalkozni, használhatjuk a következő API kulcsok valamelyikét:**

```kotlin
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI4N2IwMWI0Y2I1MTQwMjhkZDljMGVlMWE1NjE3Y2I1NCIsIm5iZiI6MTczOTM2MzAwMi40NzYsInN1YiI6IjY3YWM5MmJhMjFkMGE5MmQ0YjliYWFjMiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.SNBeQWAYNi_QEl981CV6WwjCaFPZ6gGt3VE-V7Eho84
```

```kotlin
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI1NjQ2YzI3OTE3Yjg5NjYyZTVmOWM0MWNjN2YzNjc4MiIsIm5iZiI6MTc0NTYwMjcxMi40MDQsInN1YiI6IjY4MGJjODk4ZDE0OGE4MmIwZDlkMWY0NSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.5rlt97hjf-ckbGiVg_-0-kFGxL8-D0HQX99Vr6kp7cc
```
 
3 fő technológia amely használva lesz a labor során:

 - [Ktor](https://ktor.io/), mely segítségével fog történni a halózati kommunikáció.
 - [Room](https://developer.android.com/kotlin/multiplatform/room), mely a lokális adatbázist fogja nyújtani.
 - [Koin](https://insert-koin.io/), dependency injection keretrendszer.

## Előkészületek

A feladatok megoldása során ne felejtsük el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keressük meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozzuk létre a saját repositoryt.

2. Várjuk meg, míg elkészül a repository, majd checkout-oljuk ki.

3. Hozzunk létre egy új ágat `megoldas` néven, és ezen az ágon dolgozzunk.

4. A `neptun.txt` fájlba írjuk bele a Neptun kódunkat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.

5. Indítsuk el az Android Studio-t vagy az Intellij-t, majd nyissuk meg a kapott kezdőprojektet.

6. Ellenőrizzük, hogy a létrejött projekt lefordul és futtatható mobil és desktop platformokon. A kiinduló projektünk tartalmazza az összes szükséges gradle függőséget, ezért ezeket nem kell már felvennünk a labor során.


![Starter stage](assets/hello_world.png)

## 1. Képernyők elrendezése és navigáció (1 pont)

A 2. laborhoz hasonlóan alakítsuk ki először az alkalmazás navigációjának alapját. Hozzunk létre a commonMain-ben egy **navigation** packaget, (kotlin/hu/bme/aut/navigation) azon belül egy **Route enum class**-t az alábbi tartalommal:

```kotlin
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.List
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

Ez az osztály fogja definiálni a képernyőket a navigáció számára. Ahogy látható, 2 fő képernyőből fog állni az alkalmazás: egy Search képernyő, ahol lehet majd keresni a filmekre és egy WatchList képernyő.

Célszerű lenne, hogy az alkalmazás a 2 mobilplatformon hasonlítson, ellenben desktopon eltérő layout-ja legyen. Ezt jelenleg csak kód duplikációval tudnánk elérni, vagyis az iosMain-ben és androidMain-ben duplikálva az ő megjelenésükért szükséges kódok duplán lennének. Ha jobban megnézzük, definiálva van a kezdőprojektben már egy mobileMain source set a `build.gradle.kts`-ben, melyre dependálnak az ios és az android platformok. Így fogjuk elérni azt, hogy mobil platformokon közös, de a többi (esetünkben desktop) platformtól különböző elrendezésben alakíthassuk ki a felhasználói felületünket.

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

Hozzunk létre egy _mobileMain_ directory-t a többi sourceset mellett (composeApp > src directoryn jobb klikk > New > Directory).

![Mobile Sourceset](assets/mobile_sourceset.png)

Ezután szükségünk lesz egy, a platformok navigációjának megjelenéséért felelős kódra. Itt fogjuk szabályozni, milyen elrendezésben jelenjen meg a navigáció felhasználói felülete mobil, illetve desktop platformokon. A közös kódban a ui packegen belül hozzunk létre egy PlatformLayout.kt-t az alábbi tartalommal.

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

Itt kihasználjuk az **expect-actual** mechanizmusát a KMP-nek és a platformok fogják megmondani, hogyan jelenjenek meg az egyes képernyők. A _PlatformLayout_ egy composable elem, amely a szokásos modifieren túl átveszi a navigation bar elemeit, az azokból éppen kijelölt elemet, a navigáció elemein való kattintáskor végzendő műveletet, illetve a layout tartalmát. 

!!!info "Automatikusan generált actual megvalósítások"
		A fejlesztőkörnyezetek alapvetően biztosítják ezt a funkciót, azonban ezek nem mindig működnek megfelelően. Létrehozhatjuk kézzel is ezeket az actual implementációkat, a szükséges fájlokat kézzel ugyanolyan packagebe, mint amilyenben azok expect párja található. Amennyiben automatikusan generáljuk, de azt tapasztaljuk, hogy nem generálta le a szükséges actual megvalósításokat, akkor ellenőrizzük le, hogy valóban hiányoznak-e. Desktop platformon például hajlamos a composeApp\build\generated\ksp\desktop\desktopMain\... (vagy ehhez hasonló) útvonalra generálni az actual megvalósítást tartalmazó fájlt. Amennyiben ezt tapasztaljuk, keressük meg itt, hozzuk létre kézzel a megfelelő package(ke)t, és helyezzük át ide a generált fájlt.Amennyiben valóban nem generálódott semmilyen fájl (ez viszonylag ritka, de nem lehetetlen), létrehozhatjuk kézzel, vagy megpróbálhatjuk újragenerálni.

Alt+Enter segítségével (vagy kézzel) hozzuk létre a desktop és a mobile sourceseteken az actual implementációkat, ekkor létrejönnek a `PlatformLayout.desktop.kt` és a `PlatformLayout.mobile.kt` fájlok. 

Ezek nézzenek ki az alábbi módon:

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
import androidx.compose.foundation.layout.padding
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
                navigationBarItems.forEach { navigationItem ->
                    NavigationBarItem(
                        label = { Text(text = navigationItem.title) },
                        onClick = { onNavigationItemClick(navigationItem) },
                        selected = navigationItem == selectedItem,
                        icon = { Icon(navigationItem.icon, contentDescription = null) }
                    )
                }
            }
        }
    ) { paddingValues ->
        content(
            modifier
                .padding(paddingValues)
                .fillMaxSize()
        )
    }
}
```

Desktop esetén egy oldalsó navigációs panelt (_NavigationRail_), mobil esetén pedig egy megszokott _NavigationBart_ használunk.

Használjuk fel ezután a _PlatformLayout-unkat_ az `App.kt` fájlunk fő composablejében, ez a _PlatformLayout_ fog renderelődni az alkalmazás indításakor. Alakítsuk ki a navigációt is az előző laboron látottakhoz hasonlóan, és adjuk meg a megfelelő elemeit a _PlatformLayout_ megfelelő paramétereinek:

```kotlin
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController
import hu.bme.aut.navigation.Route
import hu.bme.aut.ui.PlatformLayout
import hu.bme.aut.ui.theme.AppTheme
import org.jetbrains.compose.ui.tooling.preview.Preview

@Composable
@Preview
fun App() {
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
}
```

Futtassuk az alkalmazást, próbáljuk ki egy mobilos, és desktop platformon is. A következő(höz hasonló) elrendezést kell látnunk.

![Navigation](assets/navigation.png)

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk egy **képernyőképet**, amelyen az elindított alkalmazás nézete látszik desktopon és mobil platformon egymás mellett! A kép a megoldásban a repositoryban f1.png néven szerepeljen!

## 2. Hálózati kommunikáció és felhasználói felület kialakítása (2 pont)

### Ktor kliens felhasználása

Alakítsuk most ki a hálózati kommunikációnkat a Ktor kliens API segítségével.

Hozzunk létre a közös kódban egy _data_ (_hu.bme.aut.data_) packaget és azon belül egy _network_ (_hu.bme.aut.data.network_) packaget, melyben a hálózati kommukációhoz tartozó adatokhoz kapcsolódó részek lesznek. Hozzunk létre ide egy `TMDBResponse.kt` fájlt, ide fognak kerülni a kapcsolódó adatosztályok, melyek a használt API válaszaira adnak DTO-kat. Egy filmről el fogjuk tárolni az azonosítóját, címét és posterjéhez tartozó elérési útvonalat (path), mely nem minden film esetén biztosított, ezért nullablere állítjuk.

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
```

Ugyanebben a packageben készítsük el magát az API klienst (`TMDBApiClient.kt`), amely a Ktor kliens API segítségével fogja elvégezni a hálózati kommunikációt. Itt eltároljuk az API alap URL-jét, illetve definiáljuk a kétféle kérésünkhöz tartozó műveleteket. Az egyszerűség kedvéért az alkalmazásban csak két api hívás lesz, a **getPopularMovies()** a_ _TheMovieDB_ adatbázisban eltárolt aktuálisan legnépszerübb filmeket fogja visszaadni, a **searchMovie(title : String)** segítségével pedig lehet majd szövegesen keresni a filmek között. Az eredmény egy _HttpResponse_ típusú objektum lesz, amelynek body-jából tudjuk kinyerni a visszaadott adatokat. A kliens get kérését paraméterezzük is: beállítjuk a _language_ és _page_ paramétereket.


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

A következő lépés az adott platformoknak megfelelő HttpClient létrehozása lesz a közös kódban. Továbbra is a network packagen belül hozzunk létre egy `ProvideApiBaseClient.kt` fájlt az alábbi tartalommal. Az API kulcs helyére akár az igényelt, akár a megadott 2 api kulcs valamelyikét illesszük be!

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
                    "IDE_AZ_API_KULCS",
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

A **platformClient()** feladata elkérni az adott platformoktól a megfelelő **HttpClienteket**, majd ezen a config blockon belül pluginok hozzáadása. Jelen esetben, a hálózati kommunkáció során a tényleges tartalom JSON-ok formájában fog utazni, ezért ezt állítjuk be (_ContentNegotiation_ blokk _Json_ beállítása). Az Api hívások során az igényelt API tokent az _Authorization_ header-ben kell megadni, ehhez az _Auth_ plugint használjuk. A  kommunikáció _Bearer_ típusú lesz, itt adjuk meg az API kulcsot is. Továbbá  szeretnénk az egyszerűbb debuggolás érdekében, hogy a hálózati kommunikáció megjelenjen a console-on is.

Ezzel elkészült a közös kódban definiált _HttpClient_, ennek a konkrét megvalósításait a különböző platformokon használt megfelelő natív API segítségével tudjuk kialakítani. Alt+Enter segítségével vagy kézzel hozzuk létre a platformoknak megfelelő **HttpClienteket** tartalmazó fájlokat. Jelen esetben androidMain, iosMain és desktopMain platformokra lesz szükségünk. Ezekben platform-specifikus módon, az adott platformon használható _HttpClient-et_ (_OkHttp_ androidon és desktopon, _Darwin_ iOS-en) szolgáltatunk a Ktor számára.

`ProvideApiBaseClient.ios.kt`:
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.darwin.*

actual fun platformClient(): HttpClient {
    return HttpClient(Darwin)
}
```

`ProvideApiBaseClient.android.kt`:
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.okhttp.*

actual fun platformClient(): HttpClient {
    return HttpClient(OkHttp)
}
```

`ProvideApiBaseClient.desktop.kt`:
```kotlin
import io.ktor.client.*
import io.ktor.client.engine.okhttp.*

actual fun platformClient(): HttpClient {
    return HttpClient(OkHttp)
}
```

### Repository réteg kialakítása

A hálózati hívások (és egyéb, hozzájuk köthető származtatott műveletek) elérését tipikusan nem közvetlenül szoktuk végezni, hanem egy extra réteget, a Repository-t vezetjük be, amelyen keresztül fogjuk tudni elérni a megfelelő műveleteket. A Repository tehát absztrakcióként szolgál a HttpClient-ek fölött, amelyek a tényleges hálózati kommunikációt végzik. Így fogunk most mi is eljárni. A közös kódban a data packagen belül hozzunk létre egy _repository_ packaget (_hu.bme.aut.data.repository_), azon belül egy **MovieRepository** interfacet.

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

Itt deklaráljuk az alkalmazásunkban elérhető műveleteket. Láthatjuk, hogy suspend függvényekről van szó, amelyek Flow-n keresztül teszik elérhetővé az eredmények lekérdezését (amennyiben van ilyen).

Hozzuk létre a filmeket reprezentáló adatmodellt is. Ehhez a data package-el egyszinten készítsünk egy _domain_ packaget (_hu.bme.aut.domain_). A _domain-en_ belül pedig egy _model_ packaget vegyünk fel (_hu.bme.aut.domain.model_), benne egy _Movie_ data class-al. Ez az osztály lesz felelős a filmek alkalmazásunkon belüli reprezentálásáért.

Movie.kt:
```kotlin
data class Movie(
    val id: Int,
    val title: String,
    val posterPath: String? = null,
    val onWatchlist: Boolean = false,
)
```

Ezután implementáljuk a **MovieRepository-t**: hozzunk létre egy _impl_ packaget a repository-n belül (_hu.bme.aut.data.repository.impl_) benne az alábbi osztállyal (_MovieRepositoryImpl_).

```kotlin
import hu.bme.aut.data.mapper.toMovieDomain
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
Ez az osztály valósítja meg a Repository interface-ünket. Láthatjuk, hogy a korábbiakban elkészített _TMDBApiClient_ osztályunkat használja fel a hálózati kommunikáció elvégzéséhez. A _storedMovies_, _addMovieToWatchList_ és _removeMovieFromWatchList_ metódusok még nincsenek implementálva, azokkal később foglalkozunk majd.  A _searchMoveByTitle_ és _getPopularMovies_ metódusok viszont már meg vannak valósítva: az api kliens megfelelő metódusait meghívva megkapják az eredményt, majd a tagváltozóként felvett Flow-n keresztül (`movieResultFlow`) frissíti azt (`movieResultFlow.update`). Az eredményt a klienstől azonban a korábban elkészített _MovieResponse_ osztály (DTO) példányaként kapjuk meg, amely nem kompatibilis közvetlenül az alkalmazás modelljeként használt _Movie_ osztállyal.

Ezért szükségünk lesz még egy Mapper készítésére is az Api model -> Domain model között. Ezt már használjuk a Repositoryban (`it.toMovieDomain()`), azonban még nem valósítottuk meg, ezért jelenleg fordítási hibát kapunk rá. Ezért a data packagen belül hozzunk létre egy _mapper_ packaget (_hu.bme.aut.data.mapper_), azon belül egy **MovieMapper.kt** fájlt az alábbi tartalommal.

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

Itt egy kotlin nyelvi elemet, az _extension functiont_ használjuk: a _MovieResponse_ osztályt kiterjesztjük egy _toMovieDomain_ metódussal, amely paraméterként átveszi, hogy figyelő listán volt-e a film, majd elvégzi a konverziót a DTO és a model osztályok között. Ha mindent jól csináltunk, mostmár fordulnia kell a _MovieRepositoryImpl_ osztálynak is.

A **MovieResponse** posterPath-je nem tartalmazza a teljes útvonalat a képekhez. Ez azért van, mert a TheMovieDB más subdomaint biztosít a kép elérésére, továbbá lehet felbontást is változtatni (w500) a kisebb hálózati forgalom érdekében, amelyet szintén az elérés részeként (pl. /w500) adhatunk  meg.

### Dependency injection bevezetése

Alakítsuk most ki az alkalmazásunkban a dependency injection használatának alapjait is, a 2. laborhoz hasonló módon most is a _Koin_ segítségével. Hozzunk létre egy **di** (_hu.bme.aut.di_) packaget a data és domain packagekkel egyszinten. Ebben a packageben fogjuk definiálni az összes függőség közötti kapcsolatot. Hozzuk létre a Koin data moduleját, a _DataModule.kt_ fájlban az alábbi tartalommal:

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

Három típusú függőség injektálását definiáljuk. A HttpClient esetén a provideHttpBaseClient metódust hívjuk meg. a TMDPApiClient esetén egyszerűen példányosítjuk a TMDBApiClient osztályunkat, azonban figyelnünk kell arra is, hogy ez az osztály is vár egy függőséget konstruktorában (HttpClient). Ezért példányosításakor a Koin _get()_ metódusát hívva injektáljuk a függőségét. Az utolsó pedig a következőket éri el:
	
	1. A singleOf a single-hez hasonlóan egy singleton példányt hoz létre, csak ez az egy példány jön létre és minden, ezt a függőséget felhasználó objektum ugyanazt a példányt fogja használni.
	2. A ::::MovieRepositoryImpl Kotlinos konstruktor referencia. Lényegében azt jelenti, hogy "hozz létre a _MovieRepositoryImpl_ osztály egy példányát a konstruktorának felhasználásával".
	3. A `bind MovieRepository::class` a _MovieRepositoryImpl_ osztályt hozzáköti a _MovieRepository_ interfacehez, vagyis mindig, amikor egy _MovieRepository_ típusú függőséget várunk, egy _MovieRepositoryImpl_ példányt kell injektálni.


### Felhasználói felület folytatása
Folytassuk most a felhasználói felületünk kialakítását, valósítsuk meg a filmeket megjelenítő képernyőt és a hozzá tartozó infrastruktúrát is. Hozzunk létre a _ui_ packageben egy _feature_ (_hu.bme.aut.ui.feature_) packaget, benne hozzunk létre egy _search_ packaget (_hu.bme.aut.ui.feature.search_). Ide helyezzük majd a SearchScreen-hez köthető képernyőket és a hozzájuk tartozó ViewModeleket.

Készítsük most el a keresés képernyőt és a hozzátartozó ViewModelt.

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

Ahogy láthatjuk, a ViewModel a konstruktorában várja a repository-t, amelynek a műveleteit fogja hívni. Mikor létrejön a ViewModel, az init{} blokkjában meghívja a repository _getPopularMovies()_ függvényétegy korutinban (_viewModelScope.launch_), hogy megtörténjen a hálózati hívás, továbbá felirakozik egy belső flow-ra, mely a keresési szabadszöveges keresés Stringjét tárolja el. A Debounce 500ms szerepe pedig, hogy késlelteti a keresést fél másodpercig. A ViewModel létrejöttekor meghívja a a repository _getMovies_ metódusát is, amellyen feltölti a tárolt filmek listáját.

Hozzuk most létre a kereső képernyőt (`SearchScreen.kt`) a ViewModel-el egyszinten (_hu.bme.aut.ui.feature.search.SearchScreen_).

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
import hu.bme.aut.ui.feature.movie.MovieCard
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

A Screen konstruktorában a ViewModel-t a **koin** fogja biztosítani a koinViewModel() hívás által. Ehhez azonban még készítenünk kell egy koin Module-t, ahol ViewModelek függőségeit fogjuk definiálni.

A _PlatformMovieList_ és _MovieCard_ composablek importálása még egyelőre hibát okoz, hozzuk most létre őket. A korábbiaknak megfelelően ugyanis szükséges lenne, hogy az alkalmazásunk listanézete eltérjen a két platformon. Mobil képernyőkön egymás alatt jelenjenek (_LazyList_) meg az egyes film kártyák, de desktopon táblázatszerűen (_ListVerticalGrid_).

Hozzunk létre egy `PlatformMovieList.kt` fájlt a ui/component packageben (_hu.bme.aut.ui.component), ennek feladata lesz a paraméterként kapott filmek elrendezésének megszabása, de az egyes filmek megjelenítése a lamdbaként kapott composable függvény formájában lesz megadva.

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

Mivel egységes megjelenítést szeretnénk Androidon és iOS-en, ezért a mobileMain SourceSet-ben hozzuk létre az implementációt, ugyanolyan packageben, mint amit a közös kódban is létrehoztunk (_hu.bme.aut.ui.component_).

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

Itt definiáljuk, hogy mobilos platformok esetén egy egyszerű listaszerű megjelenítést szeretnénk a filmeknek, ahol a kapott lambda mondja meg, hogy egy konkrét filmet hogyan jelenítsünk meg.

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

Desktop esetén pedig egy gridesített megjelenítést alkalmazunk, egy film megjelenítése ugyanúgy a lambda segítségével történik.

A filmeket a megszokott Card-szerű formátumban fogjuk megjeleníteni, hozzuk most létre a közös kódban a `MovieCard.kt` fájlunkat, amely a szükséges Composable elemet fogja tartalmazni. Ezt a feature/movie packagebe helyezzük (_hu.bme.aut.feature.movie_), mert később további funkcionalitást is fog kapni a kártya, amely nem a kereséshez köthető.

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

Ezután az App.kt belépési pontján cseréljük ki az ideiglenes Text(text = Route.Search.title) composable-t a SearchScreen()-re:

`App.kt`:
```kotlin
composable(route = Route.Search.title) {
    SearchScreen()
}
```

Jelen állapotban minden kész ahhoz, hogy lehessen használni az alkalmazást, ellenben mégsem fog működni. A hiányzó, ViewModel-ekhez tartozó **DI** függőségeket kell még definiálnunk. Ehhez a közös kódbban a **DI** packagen belül (hu.bme.aut.di) hozzunk létre egy **ViewModelModule-t** az alábbi tartalommal.

`ViewModelModule.kt`:
```kotlin
import hu.bme.aut.ui.feature.search.SearchScreenViewModel
import org.koin.core.module.dsl.viewModel
import org.koin.dsl.module

fun viewModelModule() = module {
    viewModel {
        SearchScreenViewModel(get())
    }
}
```

Itt definiáljuk a ViewModel-ekkel kapcsolatos függőségeket, melyre speciális támogatást nyújt a Koin. Azt adjuk meg, hogy valahányszor a Koin _viewmodel()_ hívását használjuk (ahogy azt a SearchScreen-ben meg is tettük), a SearchScreenViewModel példányát injektáljuk. Annak konstruktor paraméterében a _get()_ hívás szerepe ugyanaz, mint amit korábban láttunk.

Vegyünk fel továbbá egy AppModule-t is, amely az alkalmazás moduljainak felsorolását tartalmazza. Vegyük észre, hogy több modult is használunk már: a korábban létrehozott DataModule-t, és az újonnan készített ViewModelModule-t. Ide helyezünk továbbá egy initializeKoin függvényt, mely átadja a koin számára a definiált függőségeket. Ezt azért tesszük meg, hogy a különböző platformokon ne kelljen azt mindenhol duplikálnunk.

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

Ezután szükséges minden platformon meghívni az initializeKoin függvényt, mellyel "bekapcsoljuk" a Koint az adott platformon.

Androidon:

`MainActivity.kt`:
```kotlin
import hu.bme.aut.di.initializeKoin

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

Desktopon:

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

iOS-en:

`MainViewController.kt`:
```kotlin
import androidx.compose.ui.window.ComposeUIViewController
import hu.bme.aut.di.initializeKoin

fun MainViewController() = ComposeUIViewController {
    initializeKoin()
    App()
}
```

Továbbá, az AndroidManifest fájlban adjuk hozzá a következő sort az engedélyekhez, hogy az alkalmazás hozzáférjen az internethez.

`AndroidManifest.xml`:
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />

```

Jelen állapotban indítható az alkalmazás, és kereshetőek a filmek. Próbáljuk ki, az alábbihoz hasonlót kell látnunk!

![Api result](assets/api_result.png)

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk egy **képernyőképet**, amelyen az elindított alkalmazás SearchScreen képernyője látszik desktopon és mobil platformon egymás mellett! A kép a megoldásban a repositoryban f2.png néven szerepeljen!

## 3. Perzisztens adattárolás: Room (1 pont)

Térjünk most rá a perzisztens adattárolás létrehozására. Ehhez a Room könyvtárat fogjuk használni, amely egy kliens oldali SQLite adatbázis felett biztosít ORM támogatást. A Room az android ökoszisztémában jól bevált és elterjedt megoldás, a 2.7.0-alpha01 verziótól kezdve pedig már Kotlin Multiplatform környezetben is használható.
[Részletes dokumentáció](https://developer.android.com/kotlin/multiplatform/room)

A Room bekötése az alkalmazásba 4 fő elemből áll, az alábbiakat kell elvégeznünk:
- **Movie Entitás** definiálás, mely a Movie táblát fogja reprezentálni az adatbázisban, melyből 1-1 sor az adatbázisban fog 1-1 objektum példányt jelenteni.
- **Dao** definiálása, melyben definiáljuk, hogy 1-1 entitáson milyen műveleteink vannak.
- **RoomDatabase** osztályból leszármazás, melyen definiáljuk az entitiásainkat és azokhoz tartozó Dao osztályainkat
- **Platform specifikus kódrészek** implementálása a Room inicializálásához.

Először hozzuk létre az entitásunkat a közös kódban a _data/database/entity_ (_hu.bme.aut.data.database.entity_) packagen belül az alábbi tartalommal:

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
Ahogy látható, a **Room-ból** származó Annotációkkal van kiegészítve egy kotlin data class, mely által a **Room Compiler** tudni fogja, hogy az adott entitást kell használnia. Továbbá nem szeretnénk, hogy a **Room** automatikusan generálja és kezelje az adott filmek elsődleges kulcsát (_PrimaryKey_ annotáció), hanem a külső API által kapott id-val fogjuk lokálisan elmenteni az egyes filmeket.

A database packagen belül hozzunk létre egy `MovieDatabase.kt` fájlt az alábbi tartalommal.

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

Itt is a _Room_ által biztosítótt annotációkkal látjuk el a **MovieDatabase** osztályunkat, ami esetünkben most egy absztrakt osztály lesz. Itt a Compiler az implementációt létrehozza a háttérben fordítási időben. Az AppDataBaseConstuctor esetén pedig az adott platformhoz tartozó initilize methodot is generálja nekünk.

A **MyMoviesDao** interfészben definiáljuk, hogy az adatbázisunkon milyen műveleteket tudunk végrehajtani.  Láthatjuk, hogy jelenleg beszúrni, minden adatot lekérdezni, egy adott film adatait olvasni, illetve törölni tudunk.

Bővítsük most ki a DI megoldásunkat a Room számára szükséges modullal is. Hozzunk létre egy `DataBaseModule.kt` fájlt a di package-en belül, az alábbi tartalommal.

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

const val DB_FILE_NAME = "mymovies.db"
```

Room esetén kissé bonyolultabb megoldást kell alkalmaznunk az adatbázis létrehozásakor. Ez azért van, mert egyes platformok esetén Platform specifikus adatokra is szükség lehet az adatbázis létrehozásakor (pl. Android esetén az android _context_). Ez jelen esetben egy **PlatformParameters** osztályban lesz elrejtve, ezáltal a lehető legtöbb kódot lehet a közös kódban implementálni. Ezen felül deklarálunk egy databaseBuilder _expect_ függvényt is, amely a Room adatbázis platform-specifikus inicializálásáért lesz felelős.

Hozzunk létre egy **PlatformParameters** osztályt a di package-en belül az alábbi tartalommal.

`PlatformParameters.kt`:
```kotlin
import org.koin.core.module.Module

expect class PlatformParameters {
    fun getModule(): Module
}
```

Egy expect classt használunk, vagyis minden platformnak szolgáltatnia kell egy megfelelő osztályt, amely képes visszaadni az adott platformon a megfelelő Koin modult.

Jelen esetben erre csak Androidon van szükségünk, ezért ezen kívül üres modult fog visszaadni a Desktop és IOS implementáció. Android esetén pedig a paraméterként átvett context-et szolgáltatjuk a modul paramétereként.

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

Ha ezzel megvagyunk, akkor létrehozthatjuk a platform specifikus implementációit a _databaseBuilder_ függvénynek.

`DatabaseBuilder.android.kt`:
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

Ahogy látható az adatbázis létrehozásához szükséges az android application context, az adatbázis driverjének pedig a _BundledSQLiteDriver_-t használjuk, a többi két platformon is.

`DatabaseBuilder.desktop.kt`:
```kotlin
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import hu.bme.aut.data.database.MovieDatabase
import java.io.File

actual fun databaseBuilder(platformParameters: PlatformParameters): RoomDatabase.Builder<MovieDatabase> {
    val dbFile = File(System.getProperty("java.io.tmpdir"), DB_FILE_NAME)
    return Room.databaseBuilder<MovieDatabase>(
        name = dbFile.absolutePath,
    )
        .setDriver(BundledSQLiteDriver())
}
```

`DatabaseBuilder.ios.kt`:
```kotlin
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import hu.bme.aut.data.database.MovieDatabase
import platform.Foundation.NSHomeDirectory

actual fun databaseBuilder(platformParameters: PlatformParameters): RoomDatabase.Builder<MovieDatabase> {
    val dbFile = NSHomeDirectory() + DB_FILE_NAME
    return Room.databaseBuilder<MovieDatabase>(
        name = dbFile,
    )
        .setDriver(BundledSQLiteDriver())
}
```

Most már csak a DI app moduljába kell felvennük a database modult és kiegészíteni a hívás láncot, hogy mindegyik platform, mikor inicializálja a koin-t, átadjon egy PlatformParatmeters példányt is paraméterként.

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

Desktop és IOS platformon elég csak létrehozni egy PlatformParameters példányt, és azt átadni paraméterként. Android esetén a platform paramétereként a _contextet_ is át kell adnunk. Mivel ezt a MainActivity-ben tesszük meg, az activity saját magát, vagyis _this_-t szolgáltatja paraméterként.

`MainActivity.kt`:
```kotlin
initializeKoin(PlatformParameters(this)) ->elég a this mert Activity contextban vagyunk.
```

Ezzel elkészültünk a DI kiegészítésével. A következő lépés az adatbázis bekötése lesz a **MovieRepositoryba**.

Ahogyan azt a DTO -> domain Movie mappingnél láttuk, hasonlóan szükségünk lesz a MovieEntity-ből domain Movie modellre és vissza irányú  mappelésre is.

Egészítsük ki a `MovieMapper.kt` fájlt az alábbi sorokkal.

`MovieMapper.kt`:
```kotlin
import hu.bme.aut.data.database.entity.MovieEntity

fun MovieEntity.toMovieDomain() = Movie(
    id, title, posterPath, onWatchlist
)

fun Movie.toMovieEntity() = MovieEntity(
    id, title,posterPath, onWatchlist
)
```

Bővítsük ezután a **MovieRepositoryImpl** osztályunkat az adatbázisban tárolt filmekre való feliratkozás, illetve a filmek adatbázisba való hozzáadás és abból való törlés műveleteinek megvalósításaival, melyhez a korábban elkészített MyMoviesDao-t használjuk fel.

`MovieRepositoryImpl.kt`:
```kotlin
import hu.bme.aut.data.database.MyMoviesDao
import hu.bme.aut.data.mapper.toMovieEntity

class MovieRepositoryImpl(
    private val tMDBApiClient: TMDBApiClient,
    private val moviesDao: MyMoviesDao,
) : MovieRepository {

...

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

Láthatjuk, hogy a WatchList-re való helyezés esetünkben nem jelent mást, mint a lokális adatbázisunkba való felvételt, amely filmeket egy külön képernyőn tekinthetjük majd meg.

Ezzel majdnem készen vagyunk már, arra van még szükség, hogy a felhasználói felületen jelenítsünk meg a filmeken egy hozzáadó / törlő gombot is, amelyekkel figyelőlistára helyezhetjük, illetve eltávolíthajuk onnan az adott filmet attól függően, hogy rajta van-e már.

Ennek implementációjára 2 alapvető lehetőségünk van.

Az első, hogy minden képernyő ViewModelje propagálja ezeket a műveleteket a repository függvényei felé, majd egy lambda paraméterként átadjuk azokat az egyes film kártyáknak.

Rövid egyszerűsített kód példánk:

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

 Ezzel a megoldással az a probléma, hogy láthatóan kódduplikáció jelenik meg. Erre megoldás lehetne, ha egy ős ViewModelbe raknánk a közös függvényeket, de akkor minden képernyőn, ahol a hozzáadás és törlés tulajdonsága kell a kártyának, a MovieCard lambdáinak  külön kellene átadni a ViewModel függvényeit.

 A másik megközelítésben a képernyők kizárólag a MovieCard-ok megjelenítéséért felelnek, és a filmekhez kapcsolódó műveletek logikáját egy külön MovieCardViewModel-be helyezzük. Ezáltal csökkentjük a képernyők kódbeli összetettségét, miközben jobban elkülönítjük a függőségeket. Ez a megközelítés elősegíti a komponensek újrafelhasználhatóságát és egyszerűsíti a kódunk karbantarthatóságát is. Hasonló elvet követ a [Decompose](https://github.com/arkivanov/Decompose) könyvtár is, amely lehetővé teszi a UI-elemek életciklusának és adatkezelésének függetlenítését a környező komponensektől. Célszerű tehát ezt a megoldást választanunk.

Hozzuk létre tehát a `MovieCardViewModel.kt`-t a MovieCard mellett (a _hu.bme.aut.ui.feature.movie_ packageben).
```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import hu.bme.aut.data.repository.MovieRepository
import hu.bme.aut.domain.model.Movie
import kotlinx.coroutines.launch

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


Szükséges még kiegészíteni a MovieCard composable-t a hozzátartozó ViewModel-el, továbbá a hozzáadás/törlésért felelős UI elemekkel.
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

Ne felejtsük el kiegészíteni a Dependency Injection megoldásunkat se: a ViewModel modulba vegyük fel a MovieCardViewModel-t is.
```kotlin
import hu.bme.aut.ui.feature.movie.MovieCardViewModel
import hu.bme.aut.ui.feature.search.SearchScreenViewModel
import org.koin.core.module.dsl.viewModel
import org.koin.dsl.module

fun viewModelModule() = module {
    viewModel {
        SearchScreenViewModel(get())
    }
    viewModel {
        MovieCardViewModel(get())
    }
}
```

Jelen állapotban indítható az alkalmazásunk és sikeresen mentődnek a filmek az adatbázisban, viszont a felhasználói felületen nem látjuk, hogy az elemek frissülnének, vagyis a hozzáadás gomb után törlés gombnak kéne megjelennie, majd törlés után újra a hozzáadásnak. Ez azért van, mert a UI-on lévő elemek az API hívás közvetlen eredményét mutatják és nincsenek vizsgálva, melyek vannak eltárolva az adatbázisban. Ez könnyen orvosolható már a repository rétegben.

Először vegyünk fel egy segédfüggvényt, mely megkapja az API hívás eredményét, továbbá az adatbázisban tárolt filmek listáját. Ha ID alapján megtaláljuk az adott filmet, akkor az onWatchList értékét beállítjuk logikai igazra.

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

A SearchMoviesViewModel jelen pillanatban a movieResultFlow-ra van feliratkozva, vagyis annak a változására történik recomposition a UI-on. Hasonlóan szükség lenne az adatábázis flow-ra történő feliratkozásra is, hogy az adatbázisban tárolt filmek változása esetén is frissüljön a SearchScreen. Erre nyújt megoldást a **combine()** nyelvi elem, mely 2 flowt alakít át egy flow-vá, és együttesen figyeli mindkettőt.

Változtassuk meg a **getMovies()** függvényt, mely már a _combinet_ és a segédfüggvényt is alkalmazza.
```kotlin
override fun getMovies(): Flow<List<Movie>> =
        combine(movieResultFlow, storedMovies()) { searchResult, storedMovies ->
            updateMovieResult(searchResult, storedMovies)
        }
```

Fordítsuk és indítsuk el az alkalmazásunkat desktop és mobilos platformon is! Mostmár működnie kell a hozzáadás és a törlés gomb változásainak is.

![Add delete on cards](assets/add_delete_on_cards.png)

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk egy **képernyőképet**, amelyen az elindított alkalmazás SearchScreen látszik desktopon és mobil platformon egymás mellett! Legyen olyan film is, amelyiken a hozzáadás gomb látszik, és olyan is, amelyiken a törlés gomb! A kép a megoldásban a repositoryban f3.png néven szerepeljen!

## Önálló feladat - Watchlist képernyő implementálása (1 pont)
Valósítsuk meg a WatchList képernyőt is! Egészítsük ki az alkalmazást úgy, hogy a Watchlist képernyőn csak az adatbázisban tárolt filmek jelenjenek meg!

??? tip "Segítség"
    A megoldás menete vázlatosan a SearchScreen-hez hasonló:
	
	1. Hozzuk létre és valósítsuk meg a WatchListScreen képernyő felületét! Használjuk fel a korábban létrehozott MovieCard composable függvényünket!
	2. Ha igényesek akarunk lenni, kövessük a SearchScreen-nél használt megközelítést, miszerint desktopon táblázatos, mobilon pedig listás megjelenítést használjunk!
	3. Hozzuk létre és valósítsuk meg a WatchListViewModel-t! Használjuk fel azt a WatchListScreen-ben!
	4. A ViewModel-ben használjuk fel a Repository szükséges szolgáltatásait!
	5. Egészítsük ki a DI ViewModel modulját az új ViewModel-el!
	6. Cseréljük ki a placeholder Text-et az App belépési pointján a WatchListScreenünkre!

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk egy **képernyőképet**, amelyen az elindított alkalmazás WatchListScreen képernyője látszik tetszőleges platformon! A képernyőn pontosan azok a filmek legyenek csak megjelenítve, amelyeket szerepelnek az adatbázisban. A kép a megoldásban a repositoryban f4.png néven szerepeljen!
	
<details>
  <summary>📱 Kiegészítés - MacOs eszköz szükséges</summary>

  Amennyiben van rá lehetőségünk, próbáljuk ki az alkalmazást iOS platformon! Miután sikeresen lefordult az alkalmazás Android és Desktop platformon, nézzük meg, hogyan jelenik meg iOS-en.

  Ehhez Xcode-ban szükséges megnyitni az `iosApp` könyvtárat, majd a **Build** gombra kattintani egy kiválasztott iPhone szimulátoron.   Ha mindent megfelelően csináltunk, egy funkcionalitásban megegyező alkalmazásunk lesz a többi platformmal.

  ![iOS](assets/ios.png)

</details>
