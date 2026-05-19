# 5. Labor - Ktor kliensoldalon, Room és SQLDelight

## Bevezető


A labor során egy filmadatbázis alkalmazást fogunk elkészíteni Android, iOS és Desktop platformokra. Az alkalmazás egy külső API segítségével fogja lekérdezni a mostanság népszerű mozifilmek listáját, amelyeket megtekinthetünk és a címük alapján szűrhetünk rájuk. A filmeket figyelő listára is helyezhetjük, illetve levehetjük őket onnan. A figyelő listán lévő filmeket külön képernyőn is láthatjuk, ahol mindig csak a listán lévő filmek jelennek meg. A hálózati kommunikációhoz Ktor, a lokális adattárolás kialakításához Room és SQLDelight lesz felhasználva.

!!!Info "Room és SQLDelight egy projektben"
	Többféle lokális adatbázis technológiát használni ugyanazon projektben legtöbbször nem életszerű, nincs sok értelme. Esetünkben azért használjuk mégis mindkettőt, hogy lássunk példát mindkét technológiára és általánosabban megközelítésre (SQL first vs code first).
	
!!!Info "Clean Architecture és Dependency Injection"
	Mivel jelen labornak nem ezek a fő fókuszai, ezért az egyszerűség kedvéért az alkalmazásban csak egy egyszerű MVVM architektúrát, és kézi Dependency Injectiont fogunk használni. Ezek kifinomultabb verzióival egy korábbi laborban találkozhattunk már, és természetesen hasonlóan használhatóak lennének ezen a laboron is.
	
Az alkalmazás képernyője a következőképpen fog kinézni mobilon, illetve desktopon. Láthatjuk, hogy a filmek listázásához egy egyszerű adaptív megközelítést is használni fogunk: mobilon oszloposan egyszerre egy, desktopon táblázatszerűen egy sorban több filmet is megjelenítünk.

<p align="center">
<img src="./assets/mobile.png" width="240">
<img src="./assets/desktop.jpg" width="640">
</p>

A filmek adatforrása a [TheMovieDb](https://www.themoviedb.org/) lesz, mely biztosít egy REST-API-t filmek és sorozatok keresésére. Ehhez egy API kulcsot kell igényelni a [Developer](https://developer.themoviedb.org/docs/getting-started) weboldalon. Ehhez regisztrálni kell, majd egy key fog megjelenni a képernyő alján. Erre később szükségünk lesz. **Amennyiben nem szeretnénk a  regisztrációval és az API kulcs igényléssel foglalkozni, használhatjuk a következő API kulcsok valamelyikét:**

```kotlin
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI4N2IwMWI0Y2I1MTQwMjhkZDljMGVlMWE1NjE3Y2I1NCIsIm5iZiI6MTczOTM2MzAwMi40NzYsInN1YiI6IjY3YWM5MmJhMjFkMGE5MmQ0YjliYWFjMiIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.SNBeQWAYNi_QEl981CV6WwjCaFPZ6gGt3VE-V7Eho84
```

```kotlin
eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI1NjQ2YzI3OTE3Yjg5NjYyZTVmOWM0MWNjN2YzNjc4MiIsIm5iZiI6MTc0NTYwMjcxMi40MDQsInN1YiI6IjY4MGJjODk4ZDE0OGE4MmIwZDlkMWY0NSIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.5rlt97hjf-ckbGiVg_-0-kFGxL8-D0HQX99Vr6kp7cc
```

## Előkészületek

A feladatok megoldása során ne felejtsük el követni a [feladat beadás folyamatát](../../tudnivalok/github/GitHub.md).

### Git repository létrehozása és letöltése

1. Moodle-ben keressük meg a laborhoz tartozó meghívó URL-jét és annak segítségével hozzuk létre a saját repositoryt.

2. Várjuk meg, míg elkészül a repository, majd checkout-oljuk ki.

3. Hozzunk létre egy új ágat `megoldas` néven, és ezen az ágon dolgozzunk.

4. A `neptun.txt` fájlba írjuk bele a Neptun kódunkat. A fájlban semmi más ne szerepeljen, csak egyetlen sorban a Neptun kód 6 karaktere.


## Projekt létrehozása

Hozzuk létre a projektet az alábbiaknak megfelelően:

1. Az alkalmazás neve legyen `MovieDB`
2. A package name `hu.bme.aut.moviedb`
3. Válasszuk ki a projekt lokációját a Git repositorynkban, majd > Next
4. A minimum SDK az Android platform SDK minimum verzióját jelenti, hagyhatjuk a defaulton (API 26 "Oreo")
5. A *Build configuration language* `Kotlin DSL` legyen.

Ezután kell kiválasztanunk, hogy milyen platformokat szeretnénk támogatni a projektünkben.

6. Pipáljuk be az Android, iOS és Desktop platformokat.
7. A felhasználói felület kódjait is osszuk meg.
8. Servert most sem fogunk használni, és ne is pipáljuk be. Nemsokára látni fogjuk, miért.
9. Tesztekre nem lesz szükségünk.
10. Ha minden rendben > Finish.

### Projekt áttekintése és függőségek felvétele

A JetBrains nemrég ismét némileg változtatott a KMP projektekhez ajánlott struktúrán, és a varázsló által generált projekt már ezt a megközelítést mutatja. A lényeg szinte ugyanaz, viszont kisebb változások vannak. A projekt most az alábbi modulokból épül fel:

*   `shared:` ahogy eddig, most is a platformokra közös kódot tartalmazza. A composeApp modul viszont megszűnt, helyette a közös felhasználói felület kódját szintén ide, a shared modulba helyezzük. 
*   `androidApp, desktopApp, iosApp:` A különböző platformok belépési pontjait, és esetleges egyéb specifikus erőforrásait tartalmazza. Ez alól kivétel az iOS belépési pontja, a MainViewController ugyanis a shared modulon belül van, az iOS-nek megfelelő modulban.
*   A shared modulon belül megtaláljuk a megszokott `commonMain-t`, ide megy a közös kód. Találunk viszont platform-specifikus kódnak megfelelő modulokat is, ide helyezhetjük a közös kódhoz kapcsolódó platform-specifikus megvalósításokat. Ilyen lehet például egy közös kódban expect kulcsszóval ellátott konstrukció actual párja az adott platformon.

Vegyük fel most a szükséges függősegeket a projektünkbe. A legfőbb függőségek, amikre szükségünk lesz:

*   Ktor releváns függőségei: kliensek, szerializáció, core, auth
*   Room függőségei: runtime és compiler, bundled sqlite driver
*   Sqldelight függőségei: coroutines, android, native és sqlite
*   Ksp plugin a room egyes részeinek fordításidejű generálására
*   coil függőségei képkezeléshez: compose és network-ktor3
*   SQLDelight plugin az SQL delight fordításidejű generálására
*   Navigáció a Navigation3 könyvtárral


Az egyszerűség kedvéért most a projektünk teljes függőség-készletét egyben megadjuk. A `libs.versions.toml` tartalma a következő lesz:

```kotlin
[versions]
agp = "9.0.1"
android-compileSdk = "36"
android-minSdk = "24"
android-targetSdk = "36"
androidx-activity = "1.13.0"
androidx-appcompat = "1.7.1"
androidx-core = "1.18.0"
androidx-lifecycle = "2.11.0-beta01"
composeMultiplatform = "1.11.0"
kotlin = "2.3.21"
kotlinx-coroutines = "1.11.0"
material3 = "1.11.0-alpha07"

ktor = "3.5.0"
sqldelight = "2.3.2"
navigation3 = "1.1.1"
adaptive = "1.3.0-alpha06"
lifecycle = "2.10.0"
coil = "3.4.0"
datetime = "0.8.0"

room = "2.8.4"
sqlite = "2.6.2"

ksp = "2.3.7"

[libraries]
androidx-core-ktx = { module = "androidx.core:core-ktx", version.ref = "androidx-core" }
androidx-appcompat = { module = "androidx.appcompat:appcompat", version.ref = "androidx-appcompat" }
androidx-activity-compose = { module = "androidx.activity:activity-compose", version.ref = "androidx-activity" }
compose-uiTooling = { module = "org.jetbrains.compose.ui:ui-tooling", version.ref = "composeMultiplatform" }
androidx-lifecycle-viewmodelCompose = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose", version.ref = "androidx-lifecycle" }
androidx-lifecycle-runtimeCompose = { module = "org.jetbrains.androidx.lifecycle:lifecycle-runtime-compose", version.ref = "androidx-lifecycle" }
compose-runtime = { module = "org.jetbrains.compose.runtime:runtime", version.ref = "composeMultiplatform" }
compose-foundation = { module = "org.jetbrains.compose.foundation:foundation", version.ref = "composeMultiplatform" }
compose-material3 = { module = "org.jetbrains.compose.material3:material3", version.ref = "material3" }
compose-ui = { module = "org.jetbrains.compose.ui:ui", version.ref = "composeMultiplatform" }
compose-components-resources = { module = "org.jetbrains.compose.components:components-resources", version.ref = "composeMultiplatform" }
compose-uiToolingPreview = { module = "org.jetbrains.compose.ui:ui-tooling-preview", version.ref = "composeMultiplatform" }
kotlinx-coroutinesSwing = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-swing", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }

navigation3-ui = { module = "org.jetbrains.androidx.navigation3:navigation3-ui", version.ref = "navigation3" }
navigation3-adaptive = { module = "org.jetbrains.compose.material3.adaptive:adaptive-navigation3", version.ref = "adaptive" }
navigation3-viewmodel = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-navigation3", version.ref = "lifecycle" }

ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
ktor-client-json = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
ktor-client-serialization = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }
ktor-client-android = { module = "io.ktor:ktor-client-android", version.ref = "ktor" }
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor" }
ktor-client-auth = { module = "io.ktor:ktor-client-auth", version.ref = "ktor" }

sqldelight-coroutines = { module = "app.cash.sqldelight:coroutines-extensions", version.ref = "sqldelight" }
sqldelight-android = { module = "app.cash.sqldelight:android-driver", version.ref = "sqldelight" }
sqldelight-native = { module = "app.cash.sqldelight:native-driver", version.ref = "sqldelight" }
sqldelight-sqlite = { module = "app.cash.sqldelight:sqlite-driver", version.ref = "sqldelight" }

androidx-sqlite-bundled = { module = "androidx.sqlite:sqlite-bundled", version.ref = "sqlite" }
androidx-room-runtime = { module = "androidx.room:room-runtime", version.ref = "room" }
androidx-room-compiler = { module = "androidx.room:room-compiler", version.ref = "room" }

datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version.ref = "datetime" }

coil-compose = { module = "io.coil-kt.coil3:coil-compose", version.ref = "coil" }
coil-network-ktor3 = { module = "io.coil-kt.coil3:coil-network-ktor3", version.ref = "coil" }

[plugins]
androidApplication = { id = "com.android.application", version.ref = "agp" }
androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
composeMultiplatform = { id = "org.jetbrains.compose", version.ref = "composeMultiplatform" }
composeCompiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
kotlinJvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
kotlinMultiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
sqldelight = { id = "app.cash.sqldelight", version.ref = "sqldelight" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

A `build.gradle.kts` (shared) szkriptünk pedig az alábbi:

```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

plugins {
    alias(libs.plugins.kotlinMultiplatform)
    alias(libs.plugins.androidMultiplatformLibrary)
    alias(libs.plugins.composeMultiplatform)
    alias(libs.plugins.composeCompiler)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.sqldelight)
    alias(libs.plugins.ksp)
}

kotlin {
    listOf(
        iosArm64(),
        iosSimulatorArm64()
    ).forEach { iosTarget ->
        iosTarget.binaries.framework {
            baseName = "Shared"
            isStatic = true
        }
    }

    jvm()

    androidLibrary {
        namespace = "hu.bme.aut.moviedb.shared"
        compileSdk = libs.versions.android.compileSdk.get().toInt()
        minSdk = libs.versions.android.minSdk.get().toInt()

        compilerOptions {
            jvmTarget = JvmTarget.JVM_11
        }
        androidResources {
            enable = true
        }
        withHostTest {
            isIncludeAndroidResources = true
        }
    }

    sourceSets {
        val commonMain by getting {
            dependencies {
                // Compose UI
                implementation(libs.compose.runtime)
                implementation(libs.compose.foundation)
                implementation(libs.compose.material3)
                implementation(libs.compose.ui)
                implementation(libs.compose.components.resources)
                implementation(libs.compose.uiToolingPreview)

                // Lifecycle & ViewModel
                implementation(libs.androidx.lifecycle.viewmodelCompose)
                implementation(libs.androidx.lifecycle.runtimeCompose)

                // Navigation 3
                implementation(libs.navigation3.ui)
                implementation(libs.navigation3.adaptive)
                implementation(libs.navigation3.viewmodel)

                // Ktor Networking
                implementation(libs.ktor.client.core)
                implementation(libs.ktor.client.json)
                implementation(libs.ktor.client.serialization)
                implementation(libs.ktor.client.auth)

                // Coroutines
                implementation(libs.kotlinx.coroutines.core)
                // SQLDelight
                implementation(libs.sqldelight.coroutines)

                // Room
                api(libs.androidx.room.runtime)
                implementation(libs.androidx.sqlite.bundled)

                // DateTime
                implementation(libs.datetime)

                // Coil
                implementation(libs.coil.compose)
                implementation(libs.coil.network.ktor3)
            }
        }

        val androidMain by getting {
            dependencies {
                implementation(libs.ktor.client.android)
                implementation(libs.sqldelight.android)
                implementation(libs.compose.uiTooling)
                implementation(libs.androidx.appcompat)
                implementation(libs.androidx.core.ktx)
            }
        }

        val jvmMain by getting {
            dependencies {
                implementation(libs.ktor.client.okhttp)
                implementation(libs.sqldelight.sqlite)
                implementation(libs.kotlinx.coroutinesSwing)
            }
        }

        val iosMain by creating {
            dependsOn(commonMain)
            dependencies {
                implementation(libs.ktor.client.darwin)
                implementation(libs.sqldelight.native)
            }
        }

        val iosArm64Main by getting {
            dependsOn(iosMain)
        }

        val iosSimulatorArm64Main by getting {
            dependsOn(iosMain)
        }
    }
}

dependencies {
    add("kspAndroid", libs.androidx.room.compiler)
    add("kspIosSimulatorArm64", libs.androidx.room.compiler)
    add("kspIosArm64", libs.androidx.room.compiler)
    add("kspJvm", libs.androidx.room.compiler)
}

sqldelight {
    databases {
        create("WatchlistDatabase") {
            packageName.set("hu.bme.aut.moviedb.database.watchlist")
            srcDirs("src/commonMain/sqldelight")
            migrationOutputDirectory.set(file("src/commonMain/sqldelight/migrations"))
        }
    }
}
```

Tekintsük át röviden ennek a tartalmát is! Figyeljük meg az alábbiakat:

*   A `dependencies` blokkon belül állítjuk be, hogy mely projektekre szeretnénk bekapcsolni a Ksp-t. Esetünkben minden platformra szeretnénk. Ennek segítségével fogja tudni a Room generálni az adatbázisunk Constructor függvényeinek actual részeit.
*   Az `sqldelight` blokkban tudjuk konfigurálni az SQLDelight-ot. Megadjuk az adatbázis nevét, a generált fájlok package nevét, illetve az sql definíciós, és migrációs fájlok elérését. Ezeken az eléréseken fogja keresni az SQLDelight pluginje őket, ami itt van, azokból fog generálni.


Szinkronizáljuk a projektünket, aminek sikeresen le kell futnia.


## Modellek és lokális adattárolás alapjai (1 pont)

Az alkalmazásunk megvalósítását az adatréteg felől kezdjük el megvalósítani. Alakítsuk ki a szükséges adatosztályokat és DTO (Data Transfer Object)-kat! 

Hozzunk létre egy `data` (`hu.bme.aut.moviedb.data`) packaget a `commonMain`-en belül, majd vegyük fel a modell osztályainkat:

```kotlin
package hu.bme.aut.moviedb.data

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class Movie(
    val id: Int,
    val title: String,
    val posterPath: String? = null,
    val onWatchlist: Boolean = false,
)

@Serializable
data class MovieResponse(
    val id: Int,
    val title: String,
    @kotlinx.serialization.SerialName("poster_path") val posterPath: String? = null,
)

@Serializable
data class TMDbResponse(
    val page: Int,
    val results: List<MovieResponse>,
    @SerialName("total_pages") val totalPages: Int,
    @SerialName("total_results") val totalResults: Int,
)
```

A `Movie` az alkalmazásunk adatosztálya, a másik 2 osztály DTO szerepet tölt be. A `TMDbResponse` az API teljes válaszát fedi le, ami több film adatait tartalmazza. A `MovieResponse` egyetlen film objektum adatait reprezentálja.

Alakítsuk most ki a Room adatbázisunkhoz szükséges infrastruktúrát. Vegyünk fel egy `database` packaget, majd azon belül a következőkre lesz szükségünk.

Az adatbázisban tárolt entitásainkat a  `MovieEntity` osztály reprezentálja.

```kotlin
package hu.bme.aut.moviedb.database

import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlin.time.Clock

@Entity(tableName = "cached_movies")
data class MovieEntity(
    @PrimaryKey val id: Int,
    val title: String,
    val posterPath: String? = null,
    val timestamp: Long = Clock.System.now().toEpochMilliseconds(),
)
```

Szükségünk lesz a `MovieDao` interfészre is, amely az adatbázison végrehajtható műveleteinket fogja tartalmazni. Esetünkben ez az adatbázis lokális cacheként szolgál, ezért az adatbázisunk elnevezése ezt helyenként tükrözni fogja. A támogatott műveleteink az összes film lekérdezése, film beszúrása, az adatbázis teljes törlése, illetve a szűrés funkcióhoz filmek cím alapján való keresése.

```kotlin
package hu.bme.aut.moviedb.database

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query

@Dao
interface MovieDao {
    @Query("SELECT * FROM cached_movies ORDER BY timestamp DESC")
    suspend fun getAllCachedMovies(): List<MovieEntity>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertMovies(movies: List<MovieEntity>)

    @Query("DELETE FROM cached_movies")
    suspend fun clearCache()

    @Query("SELECT * FROM cached_movies WHERE title LIKE '%' || :query || '%'")
    suspend fun searchCached(query: String): List<MovieEntity>
}
```

Ezután hozzuk létre magát az adatbázist is. Ehhez deklarálnunk kell egy adatbázis absztrakt osztályt, majd annak felépítéséről is gondoskodnunk kell. A `MovieDatabase` osztályt a megfelelő annotációkkal látjuk el, majd deklaráljuk az `expect` kulcsszóval annak `MovieDatabaseConstructor` objektumát. Végül definiálunk egy közös függvényt, amely egy `builder` birtokában konfigurálja és létrehozza az adatbázis konkrét példányát.

```kotlin
package hu.bme.aut.moviedb.database
import androidx.room.ConstructedBy
import androidx.room.Database
import androidx.room.RoomDatabase
import androidx.room.RoomDatabaseConstructor
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.IO

@Database(
    entities = [MovieEntity::class],
    version = 1,
    exportSchema = false
)
@ConstructedBy(MovieDatabaseConstructor::class)
abstract class MovieDatabase : RoomDatabase() {
    abstract fun movieDao(): MovieDao
}

expect object MovieDatabaseConstructor : RoomDatabaseConstructor<MovieDatabase> {
    override fun initialize(): MovieDatabase
}

fun getRoomDatabase(
    builder: RoomDatabase.Builder<MovieDatabase>
): MovieDatabase {
    return builder
        .fallbackToDestructiveMigrationOnDowngrade(true)
        .setDriver(BundledSQLiteDriver())
        .setQueryCoroutineContext(Dispatchers.IO)
        .build()
}
```

Fontos kiemelni, hogy a `MovieDatabaseConstructor` `actual` megvalósításait nem nekünk kell kézzel létrehoznunk! Azokat a Ksp segítségével a Room automatikusan generálja nekünk fordítás közben. Fordítsuk a projektünket, ekkor a `shared` modul `build` mappájában a `generated.ksp` mappán belül találjuk a különböző platformokra a generált megvalósításokat. Ezekhez kézzel nem kell hozzányúlnunk, csupán a háttérben a Room megfelelő működéséhez kellenek.

Az adatbázisunk felépítéséhez viszont még hiányoznak a platform-specifikus megvalósítások, amelyek a `RoomDatabase.Builder<MovieDatabase>` megfelelő megvalósítását adják az adott platformon. Ezt felhasználva fogjuk tudni az előbb megírt függvényünk segítségével felépíteni az adatbázisunkat a közös kódban. Valósítsuk most meg ezeket is!

A `shared` `androidMain`moduljában hozzuk létre ugyanabba a packagebe (`hu.bme.aut.moviedb.database`) a `DatabaseBuilder.android.kt` fájlunkat, ahol az android kontextust és az adatbázis fájl elérését felhasználva gyártjuk le a megfelelő buildert:

```kotlin
package hu.bme.aut.moviedb.database

import android.content.Context
import androidx.room.Room
import androidx.room.RoomDatabase

fun getDatabaseBuilder(context: Context): RoomDatabase.Builder<MovieDatabase> {
    val appContext = context.applicationContext
    val dbFile = appContext.getDatabasePath("my_room.db")
    return Room.databaseBuilder<MovieDatabase>(
        context = appContext,
        name = dbFile.absolutePath
    )
}
```

A másik két platformon hasonlóan járunk el.

`DatabaseBuilder.ios.kt:`

```kotlin
package hu.bme.aut.moviedb.database

import androidx.room.Room
import androidx.room.RoomDatabase
import kotlinx.cinterop.ExperimentalForeignApi
import platform.Foundation.NSDocumentDirectory
import platform.Foundation.NSFileManager
import platform.Foundation.NSUserDomainMask

fun getDatabaseBuilder(): RoomDatabase.Builder<MovieDatabase> {
    val dbFilePath = documentDirectory() + "/movie_cache.db"
    return Room.databaseBuilder<MovieDatabase>(
        name = dbFilePath,
    )
}

@OptIn(ExperimentalForeignApi::class)
private fun documentDirectory(): String {
    val documentDirectory =
        NSFileManager.defaultManager.URLForDirectory(
            directory = NSDocumentDirectory,
            inDomain = NSUserDomainMask,
            appropriateForURL = null,
            create = false,
            error = null,
        )
    return requireNotNull(documentDirectory?.path)
}
```

`DatabaseBuilder.desktop.kt:`

```kotlin
package hu.bme.aut.moviedb.database

import androidx.room.Room
import androidx.room.RoomDatabase
import java.io.File

fun getDatabaseBuilder(): RoomDatabase.Builder<MovieDatabase> {
    val dbFile = File(System.getProperty("java.io.tmpdir"), "my_room.db")
    return Room.databaseBuilder<MovieDatabase>(
        name = dbFile.absolutePath,
    )
}
```

Ezzel készen vagyunk a Room alapjaival, a hátralévő részeket később fogjuk befejezni. Valósítsuk meg előbb az SQLDelight alapjait is! Esetünkben ezt arra fogjuk használni, hogy a figyelő listán lévő elemeket külön, SQLDelight segítségével is fogjuk tárolni, egy másik lokális adatbázisban. A `commonMain` modulon belül, a `kotlin` folderrel egyszinten hozzunk létre egy `sqldelight` foldert is, azon belül pedig a `hu.bme.aut.moviedb.database.watchlist` packaget! Ide helyezzük el a migrációs szkriptünket, és az adatdefiníciós sq fájlunkat!

Hozzuk létre az `1.sqm` migrációs szkriptet:
```kotlin
CREATE TABLE IF NOT EXISTS watchlist_movie (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    poster_path TEXT,
    added_at INTEGER DEFAULT (strftime('%s', 'now'))
);
```

Ezt azért tesszük meg, hogy ha valamiért módosulna az adatbázis sémánk, akkor a `watchlist_movie` táblát ne próbálja meg az SQLDelight újra és újra létrehozni, mert az hibához vezetne.

!!!Info "Table `watchlist_movie` already exists hibaüzenet"
	Amennyiben mégis ilyen, vagy ehhez hasonló hibaüzenettel találkozunk, akkor a legegyszerűbben úgy tudjuk azt orvosolni, hogy teljesen reseteljük az adatbázisunkat az adott platformon. Mobilon ezt elsősorban az adatok törlésével (Wipe Data), desktopon a fájlrendszeren az adatbázisfájl kézi törlésével érhetjük el. Utóbbit a `desktopApp` desktop platform-specifikus belépési pontot tartalmazó projekt gyökerében találjuk.
	
A `WatchlistMovie.sq` definíciós fájlunk pedig:

Hozzuk létre az `1.sqm` migrációs szkriptet:
```kotlin
CREATE TABLE watchlist_movie (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    poster_path TEXT,
    added_at INTEGER DEFAULT (strftime('%s', 'now'))
);

selectAll:
SELECT * FROM watchlist_movie ORDER BY added_at DESC;

insert:
INSERT OR REPLACE INTO watchlist_movie(id, title, poster_path)
VALUES(?, ?, ?);

delete:
DELETE FROM watchlist_movie WHERE id = ?;

isInWatchlist:
SELECT EXISTS(SELECT 1 FROM watchlist_movie WHERE id = ?);

count:
SELECT COUNT(*) FROM watchlist_movie;
```

A Room generálásához hasonlóan itt is fordítás közben fognak létrejönni a generált fájljaink. Tegyük is ezt meg, az eredményt a `build.generated.sqldelight` folder alatt találjuk. Nézzük is meg röviden ezeket is!

Végezetül szükségünk lesz az SQLDelight adatbázis létrehozásához a megfelelő, platform-specifikus `SqlDriver` implementációkra is, melyre szüksége lesz az adatbázisnak. Vegyünk fel egy `DatabaseDriverFactory` `expect` kulcsszóval ellátott osztályt a `database` packagen belül:

```kotlin
package hu.bme.aut.moviedb.database

import app.cash.sqldelight.db.SqlDriver

@Suppress("EXPECT_ACTUAL_CLASSIFIERS_ARE_IN_BETA_WARNING")
expect class DatabaseDriverFactory {
    fun createDriver(): SqlDriver
}
```

Majd adjuk meg ennek a megfelelő megvalósításait az egyes platformokon! Továbbra is a `shared` modul megfelelő platform-specifikus projektjeinek a `database` packagejében dolgozzunk!

`DatabaseDriverFactory.android.kt`:

```kotlin
package hu.bme.aut.moviedb.database

import android.content.Context
import app.cash.sqldelight.db.SqlDriver
import app.cash.sqldelight.driver.android.AndroidSqliteDriver
import hu.bme.aut.moviedb.database.watchlist.WatchlistDatabase

@Suppress("EXPECT_ACTUAL_CLASSIFIERS_ARE_IN_BETA_WARNING")
actual class DatabaseDriverFactory(private val context: Context) {
    actual fun createDriver(): SqlDriver {
        return AndroidSqliteDriver(
            schema = WatchlistDatabase.Schema,
            context = context,
            name = "watchlist.db"
        )
    }
}
```

`DatabaseDriverFactory.ios.kt`:

```kotlin
package hu.bme.aut.moviedb.database

import app.cash.sqldelight.db.SqlDriver
import app.cash.sqldelight.driver.native.NativeSqliteDriver
import hu.bme.aut.moviedb.database.watchlist.WatchlistDatabase

@Suppress("EXPECT_ACTUAL_CLASSIFIERS_ARE_IN_BETA_WARNING")
actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver {
        return NativeSqliteDriver(
            schema = WatchlistDatabase.Schema,
            name = "watchlist.db"
        )
    }
}
```

`DatabaseDriverFactory.jvm.kt`:

```kotlin
package hu.bme.aut.moviedb.database

import app.cash.sqldelight.db.SqlDriver
import app.cash.sqldelight.driver.jdbc.sqlite.JdbcSqliteDriver
import hu.bme.aut.moviedb.database.watchlist.WatchlistDatabase

@Suppress("EXPECT_ACTUAL_CLASSIFIERS_ARE_IN_BETA_WARNING")
actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver {
        val driver = JdbcSqliteDriver("jdbc:sqlite:watchlist.db")
        if (!driver.fileExists()) {
            WatchlistDatabase.Schema.create(driver)

        }
        return driver
    }
}

private fun JdbcSqliteDriver.fileExists(): Boolean {
    return java.io.File("watchlist.db").exists()
}
```

Desktop esetén körültekintően kell eljárnunk, ugyanis a JDBC driver meglehetősen érzékeny a sémára. Gyakorlatilag minden futtatáson újra létre szeretné hozni az adatbázist, ami könnyen hibához vezethet. Ennek kivédésére ellenőrizzük, hogy létezik-e már az adatbázisfájl és csak akkor hozzuk létre, amennyiben még nem létezett.


!!!example "BEADANDÓ (1 pont)" 
	Ehhez a feladathoz nem szükséges képernyőképet készítenünk. Amennyiben elkészültünk, folytassuk tovább a következő feladattal.
	
	
## UI és navigáció (1 pont)

Térjünk most át a felhasználói felület megvalósítására! A  `commonMain`-en belül hozzuk létre a `ui` packaget, azon belül a `components`, `screens`, és `viewmodels` packageket!

Kezdjük a komponensekkel. Esetünkben csak egyre lesz szükség, a filmek megjelenítéséért felelős `MovieCard` composablere:

```kotlin
package hu.bme.aut.moviedb.ui.components

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.unit.dp
import coil3.compose.AsyncImage
import hu.bme.aut.moviedb.data.Movie

@Composable
fun MovieCard(
    movie: Movie,
    onWatchlistToggle: () -> Unit,
    modifier: Modifier = Modifier,
    onClick: (() -> Unit)? = null,
) {
    val posterUrl = movie.posterPath?.let {
        "https://image.tmdb.org/t/p/w200$it"
    }

    Card(
        modifier = modifier
            .fillMaxWidth()
            .then(if (onClick != null) Modifier.clickable { onClick() } else Modifier),
        shape = RoundedCornerShape(12.dp),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(12.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {

            AsyncImage(
                model = posterUrl,
                contentDescription = movie.title,
                modifier = Modifier
                    .size(80.dp, 120.dp)
                    .clip(RoundedCornerShape(8.dp)),
                contentScale = ContentScale.Crop
            )

            Spacer(modifier = Modifier.width(16.dp))

            Column(
                modifier = Modifier.weight(1f)
            ) {
                Text(
                    text = movie.title,
                    style = MaterialTheme.typography.titleMedium,
                    maxLines = 2
                )

                Spacer(modifier = Modifier.height(8.dp))

                Button(
                    onClick = onWatchlistToggle,
                    modifier = Modifier.wrapContentSize(),
                    colors = ButtonDefaults.buttonColors(
                        containerColor = if (movie.onWatchlist)
                            MaterialTheme.colorScheme.secondary
                        else
                            MaterialTheme.colorScheme.primary
                    )
                ) {
                    Text(if (movie.onWatchlist) "Remove from Watchlist" else "Add to Watchlist")
                }
            }
        }
    }
}
```

Többnyire a megszokott elemeket láthatjuk itt. A poszterek betöltését a `Coil` segítségével valósítjuk meg. A kártyákra elhelyezünk egy szöveges gombot is, amelyet megnyomva tudjuk majd állítani, hogy az adott film figyelő listán van-e, vagy nem. A gomb felirata ennek megfelelően fog változni.


Jöjjenek most a Screenek, a `MovieListScreen` és a `WatchlistScreen`.

```kotlin
package hu.bme.aut.moviedb.ui.screens

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.lazy.staggeredgrid.LazyVerticalStaggeredGrid
import androidx.compose.foundation.lazy.staggeredgrid.StaggeredGridCells
import androidx.compose.foundation.lazy.staggeredgrid.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import hu.bme.aut.moviedb.ui.components.MovieCard
import hu.bme.aut.moviedb.ui.viewmodels.MovieListViewModel
import hu.bme.aut.moviedb.ui.viewmodels.SharedViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MovieListScreen(
    viewModel: MovieListViewModel,
    sharedViewModel: SharedViewModel,
    onNavigateToWatchlist: () -> Unit,
    isDesktop: Boolean = false
) {
    val uiState by viewModel.uiState.collectAsState()
    val searchQuery by viewModel.searchQuery.collectAsState()
    val watchlistIds by sharedViewModel.watchlistIds.collectAsState()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Movie Watcher") },
                actions = {
                    TextButton(
                        onClick = onNavigateToWatchlist
                    ) {
                        Text("Watchlist (${watchlistIds.size})")
                    }
                }
            )
        }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            OutlinedTextField(
                value = searchQuery,
                onValueChange = { viewModel.updateSearchQuery(it) },
                label = { Text("Search movies") },
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp),
                singleLine = true
            )

            when {
                uiState.isLoading -> {
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        CircularProgressIndicator()
                    }
                }
                uiState.error != null -> {
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        Column(horizontalAlignment = Alignment.CenterHorizontally) {
                            Text("Error: ${uiState.error}")
                            Spacer(modifier = Modifier.height(8.dp))
                            Button(onClick = { viewModel.updateSearchQuery(searchQuery) }) {
                                Text("Retry")
                            }
                        }
                    }
                }
                else -> {
                    if (isDesktop) {
                        LazyVerticalStaggeredGrid(
                            columns = StaggeredGridCells.Adaptive(250.dp),
                            modifier = Modifier.fillMaxSize(),
                            contentPadding = PaddingValues(16.dp),
                            horizontalArrangement = Arrangement.spacedBy(16.dp),
                            verticalItemSpacing = 16.dp
                        ) {
                            items(uiState.movies) { movie ->
                                MovieCard(
                                    movie = movie,
                                    onWatchlistToggle = {
                                        if (viewModel.isOnWatchlist(movie.id)) {
                                            viewModel.removeFromWatchlist(movie.id)
                                        } else {
                                            viewModel.addToWatchlist(movie)
                                        }
                                    }
                                )
                            }
                        }
                    } else {
                        LazyColumn(
                            modifier = Modifier.fillMaxSize(),
                            contentPadding = PaddingValues(16.dp),
                            verticalArrangement = Arrangement.spacedBy(12.dp)
                        ) {
                            items(uiState.movies) { movie ->
                                MovieCard(
                                    movie = movie,
                                    onWatchlistToggle = {
                                        if (viewModel.isOnWatchlist(movie.id)) {
                                            viewModel.removeFromWatchlist(movie.id)
                                        } else {
                                            viewModel.addToWatchlist(movie)
                                        }
                                    }
                                )
                            }
                        }
                    }
                }
            }
        }
    }
}
```

```kotlin
package hu.bme.aut.moviedb.ui.screens

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import hu.bme.aut.moviedb.ui.components.MovieCard
import hu.bme.aut.moviedb.ui.viewmodels.WatchlistViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun WatchlistScreen(
    viewModel: WatchlistViewModel,
    onNavigateBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("My Watchlist") },
                navigationIcon = {
                    TextButton(onClick = onNavigateBack) {
                        Text("Back")
                    }
                },
                actions = {
                    if (!uiState.isEmpty) {
                        TextButton(onClick = { viewModel.clearAllWatchlist() }) {
                            Text("Clear All")
                        }
                    }
                }
            )
        }
    ) { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            when {
                uiState.isLoading -> {
                    CircularProgressIndicator(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                uiState.isEmpty -> {
                    Column(
                        modifier = Modifier.align(Alignment.Center),
                        horizontalAlignment = Alignment.CenterHorizontally
                    ) {
                        Text(
                            text = "Your watchlist is empty",
                            style = MaterialTheme.typography.titleLarge,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                        Spacer(modifier = Modifier.height(8.dp))
                        Text(
                            text = "Add movies from the movie list screen",
                            style = MaterialTheme.typography.bodyMedium,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                    }
                }
                else -> {
                    LazyColumn(
                        modifier = Modifier.fillMaxSize(),
                        contentPadding = PaddingValues(16.dp),
                        verticalArrangement = Arrangement.spacedBy(12.dp)
                    ) {
                        items(uiState.movies) { movie ->
                            MovieCard(
                                movie = movie,
                                onWatchlistToggle = {
                                    viewModel.removeFromWatchlist(movie.id)
                                }
                            )
                        }
                    }
                }
            }
        }
    }
}
```

Itt is hasonlóan járunk el, mint az előző laborokon. Az adaptív listázást egyszerűen úgy oldjuk meg, hogy paraméterként átvesszük egy `boolean`-ban, mobilos, vagy desktopos felületről van-e éppen szó. Amíg az adatok betöltése zajlik, addig egy LoadingIndicator-t jelenítünk meg. Az esetleges hibákat vagy szélsőséges eseteket (pl. üres Watchlist) pedig a `uiState` megfelelő állapotára való reagálással kezeljük (isLoading, isEmpty, error).

A képernyőink persze még nem fordulnak, hiszen a ViewModel-ek még nincsenek készen. Valósítsuk most meg ezeket is!

Háromféle ViewModelt fogunk használni: mindkét képernyőnek a magáét, illetve egy `SharedViewModelt` is, amely a két képernyőre közös állapotot és annak műveleteit kezeli. Ez alatt a figyelő listához való tartozást, ahhoz való hozzáadást, levételt és lekérdezést értjük. Ennek segítségével érjük el, hogy ha az egyik képernyőn pl. leveszünk egy filmet a figyelő listáról, akkor a másikon is egyszrűen le fog kerülni: a közös ViewModel állapota frissül, amely alapján értesítést kap mindkét Screen és újrarajzolják a szükséges részeket.

Ismétlésképp tekintsük át a ViewModelek kódját röviden, értelmezzük azokat!

`MovieListViewModel:`

```kotlin
package hu.bme.aut.moviedb.ui.viewmodels


import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import hu.bme.aut.moviedb.data.Movie
import hu.bme.aut.moviedb.repository.MovieRepository
import hu.bme.aut.moviedb.repository.WatchlistRepository
import kotlinx.coroutines.Job
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch

data class MovieListUiState(
    val movies: List<Movie> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

class MovieListViewModel(
    private val movieRepository: MovieRepository,
    private val watchlistRepository: WatchlistRepository,
    private val sharedViewModel: SharedViewModel
) : ViewModel() {

    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()

    private val _uiState = MutableStateFlow(MovieListUiState())
    val uiState: StateFlow<MovieListUiState> = _uiState.asStateFlow()

    private var searchJob: Job? = null

    init {
        loadMovies()
        syncWatchlistStatus()
    }

    fun updateSearchQuery(query: String) {
        _searchQuery.value = query
        searchJob?.cancel()
        searchJob = viewModelScope.launch {
            delay(500)
            loadMovies()
        }
    }

    private fun loadMovies() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            try {
                val movies = movieRepository.getMovies(_searchQuery.value)
                val updatedMovies = movies.map { movie ->
                    movie.copy(onWatchlist = sharedViewModel.isOnWatchlist(movie.id))
                }
                _uiState.update { it.copy(movies = updatedMovies, isLoading = false) }
            } catch (e: Exception) {
                e.printStackTrace()  
                _uiState.update {
                    it.copy(isLoading = false, error = e.message ?: "Unknown error")
                }
            }
        }
    }

    private fun syncWatchlistStatus() {
        viewModelScope.launch {
            watchlistRepository.getAllWatchlistMovies()
                .collect { watchlistMovies ->
                    sharedViewModel.updateWatchlist(watchlistMovies)
                    val currentMovies = _uiState.value.movies
                    val updatedMovies = currentMovies.map { movie ->
                        movie.copy(onWatchlist = sharedViewModel.isOnWatchlist(movie.id))
                    }
                    _uiState.update { it.copy(movies = updatedMovies) }
                }
        }
    }

    fun addToWatchlist(movie: Movie) {
        viewModelScope.launch {
            watchlistRepository.addToWatchlist(movie)
            sharedViewModel.addToWatchlist(movie)
            _uiState.update { state ->
                val updatedMovies = state.movies.map {
                    if (it.id == movie.id) it.copy(onWatchlist = true) else it
                }
                state.copy(movies = updatedMovies)
            }
        }
    }

    fun removeFromWatchlist(movieId: Int) {
        viewModelScope.launch {
            watchlistRepository.removeFromWatchlist(movieId)
            sharedViewModel.removeFromWatchlist(movieId)
            _uiState.update { state ->
                val updatedMovies = state.movies.map {
                    if (it.id == movieId) it.copy(onWatchlist = false) else it
                }
                state.copy(movies = updatedMovies)
            }
        }
    }

    fun isOnWatchlist(movieId: Int): Boolean = sharedViewModel.isOnWatchlist(movieId)
}
```

`WatchlistViewModel:`

```kotlin
package hu.bme.aut.moviedb.ui.viewmodels


import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import hu.bme.aut.moviedb.data.Movie
import hu.bme.aut.moviedb.repository.WatchlistRepository
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch

data class WatchlistUiState(
    val movies: List<Movie> = emptyList(),
    val isLoading: Boolean = false,
    val isEmpty: Boolean = true
)

class WatchlistViewModel(
    private val watchlistRepository: WatchlistRepository,
    private val sharedViewModel: SharedViewModel
) : ViewModel() {

    private val _uiState = MutableStateFlow(WatchlistUiState())
    val uiState: StateFlow<WatchlistUiState> = _uiState.asStateFlow()

    init {
        loadWatchlist()
    }

    private fun loadWatchlist() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            watchlistRepository.getAllWatchlistMovies()
                .collect { movies ->
                    _uiState.update {
                        it.copy(
                            movies = movies,
                            isLoading = false,
                            isEmpty = movies.isEmpty()
                        )
                    }
                    sharedViewModel.updateWatchlist(movies)
                }
        }
    }

    fun removeFromWatchlist(movieId: Int) {
        viewModelScope.launch {
            watchlistRepository.removeFromWatchlist(movieId)
            sharedViewModel.removeFromWatchlist(movieId)
        }
    }

    fun clearAllWatchlist() {
        viewModelScope.launch {
            _uiState.value.movies.forEach { movie ->
                watchlistRepository.removeFromWatchlist(movie.id)
            }
            sharedViewModel.updateWatchlist(emptyList())
        }
    }
}
```

`SharedViewModel:`

```kotlin
package hu.bme.aut.moviedb.ui.viewmodels

import hu.bme.aut.moviedb.data.Movie
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class SharedViewModel {
    private val _watchlistMovies = MutableStateFlow<List<Movie>>(emptyList())
    val watchlistMovies: StateFlow<List<Movie>> = _watchlistMovies.asStateFlow()

    private val _watchlistIds = MutableStateFlow<List<Int>>(emptyList())
    val watchlistIds: StateFlow<List<Int>> = _watchlistIds.asStateFlow()

    fun updateWatchlist(movies: List<Movie>) {
        _watchlistMovies.value = movies
        _watchlistIds.value = movies.map { it.id }
    }

    fun isOnWatchlist(movieId: Int): Boolean {
        return _watchlistIds.value.contains(movieId)
    }

    fun addToWatchlist(movie: Movie) {
        if (!_watchlistIds.value.contains(movie.id)) {
            _watchlistIds.value += movie.id
            _watchlistMovies.value += movie
        }
    }

    fun removeFromWatchlist(movieId: Int) {
        _watchlistIds.value = _watchlistIds.value.filter { it != movieId }
        _watchlistMovies.value = _watchlistMovies.value.filter { it.id != movieId }
    }
}
```

Térjünk most át a navigációra, és az alkalmazásunk belépési pontjaira! Vegyük fel a `navigation` packaget a `commonMain`-en belül, azon belül definiáljuk a képernyőink konfigurációit!

`Screen.kt`:

```kotlin
package hu.bme.aut.moviedb.navigation

import androidx.navigation3.runtime.NavKey
import androidx.savedstate.serialization.SavedStateConfiguration
import kotlinx.serialization.Serializable
import kotlinx.serialization.modules.SerializersModule
import kotlinx.serialization.modules.polymorphic

sealed interface Screen: NavKey {

    @Serializable
    data object MovieListScreenDestination: Screen

    @Serializable
    data object WatchlistScreenDestination: Screen

    companion object {
        val config = SavedStateConfiguration {
            serializersModule = SerializersModule {
                polymorphic(NavKey::class) {
                    subclass(
                        MovieListScreenDestination::class,
                        MovieListScreenDestination.serializer()
                    )
                    subclass(
                        WatchlistScreenDestination::class,
                        WatchlistScreenDestination.serializer()
                    )
                }
            }
        }
    }
}
```

Majd írjuk meg a belépési pontunkat!

`App.kt`:

``` kotlin
package hu.bme.aut.moviedb

import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.adaptive.ExperimentalMaterial3AdaptiveApi
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation3.runtime.entryProvider
import androidx.navigation3.runtime.rememberNavBackStack
import androidx.navigation3.ui.NavDisplay
import androidx.room.RoomDatabase
import hu.bme.aut.moviedb.ui.screens.MovieListScreen
import hu.bme.aut.moviedb.ui.screens.WatchlistScreen
import hu.bme.aut.moviedb.ui.viewmodels.MovieListViewModel
import hu.bme.aut.moviedb.ui.viewmodels.SharedViewModel
import hu.bme.aut.moviedb.ui.viewmodels.WatchlistViewModel
import hu.bme.aut.moviedb.database.DatabaseDriverFactory
import hu.bme.aut.moviedb.database.MovieDatabase
import hu.bme.aut.moviedb.navigation.Screen
import hu.bme.aut.moviedb.network.ApiService
import hu.bme.aut.moviedb.repository.MovieRepository
import hu.bme.aut.moviedb.repository.WatchlistRepository

@OptIn(ExperimentalMaterial3AdaptiveApi::class)
@Composable
fun App(
    modifier: Modifier = Modifier,
    isDesktop: Boolean = false,
    databaseDriverFactory: DatabaseDriverFactory,
    databaseBuilder: RoomDatabase.Builder<MovieDatabase>
) {
    val apiService = remember { ApiService() }
    val movieRepository = remember { MovieRepository(apiService, databaseBuilder) }
    val watchlistRepository = remember { WatchlistRepository(databaseDriverFactory) }
    val sharedViewModel = remember { SharedViewModel() }

    val movieListViewModel: MovieListViewModel = viewModel {
        MovieListViewModel(movieRepository, watchlistRepository, sharedViewModel)
    }
    val watchlistViewModel: WatchlistViewModel = viewModel {
        WatchlistViewModel(watchlistRepository, sharedViewModel)
    }

    val backStack = rememberNavBackStack(
        configuration = Screen.config,
        Screen.MovieListScreenDestination
    )

    NavDisplay(
        modifier = modifier.fillMaxSize(),
        backStack = backStack,
        onBack = { backStack.removeLastOrNull() },
        entryProvider = entryProvider {

            entry<Screen.MovieListScreenDestination> { _ ->
                MovieListScreen(
                    viewModel = movieListViewModel,
                    sharedViewModel = sharedViewModel,
                    onNavigateToWatchlist = {
                        backStack.add(Screen.WatchlistScreenDestination)
                    },
                    isDesktop = isDesktop
                )
            }

            entry<Screen.WatchlistScreenDestination> { _ ->
                WatchlistScreen(
                    viewModel = watchlistViewModel,
                    onNavigateBack = { backStack.removeLast() }
                )
            }
        }
    )
}
```

Itt láthatjuk, hogy paraméterként átvesszük a korábban megírt `DatabaseDriverFactoryt` és a RoomDatabase.Buildert`. Itt vesszük át azt is, hogy desktopon vagyunk-e, vagy mobilos platformon. Ezen felül vegyük észre, hogy a kézzel való függőséginjektálás is itt történik: manuálisan hozzuk létre a megfelelő példányokat, és adjuk tovább őket a tőlük függő szereplőknek. Az alkalmazásunk még nem fordul, hiszen még nem vagyunk teljesen készen, valósítsuk meg a továbbiakban a hátralévő részeket is!

!!!example "BEADANDÓ (1 pont)" 
	Ehhez a feladathoz nem szükséges képernyőképet készítenünk. Amennyiben elkészültünk, folytassuk tovább a következő feladattal.
	
## Hálózatkezelés, repository és utolsó simítások (1 pont)

Kezdük a hálózatkezelés megvalósításával! Vegyünk fel a közös kódban egy `network` packaget, azon belül készítsük el az API szolgáltatásunkat!

`ApiService.kt`:

```kotlin
package hu.bme.aut.moviedb.network

import hu.bme.aut.moviedb.data.Movie
import hu.bme.aut.moviedb.data.MovieResponse
import hu.bme.aut.moviedb.data.TMDbResponse
import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.plugins.auth.Auth
import io.ktor.client.plugins.auth.providers.BearerTokens
import io.ktor.client.plugins.auth.providers.bearer
import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
import io.ktor.client.request.get
import io.ktor.client.request.parameter
import io.ktor.serialization.kotlinx.json.json
import kotlinx.serialization.json.Json

class ApiService {
    private val baseUrl = "https://api.themoviedb.org/3"

    private val client = HttpClient {
        install(ContentNegotiation) {
            json(Json {
                ignoreUnknownKeys = true
                encodeDefaults = false
            })
        }
        install(Auth) {
            bearer {
                loadTokens {
                    BearerTokens(
                        "API_KEY_HERE",
                        null
                    )
                }
            }
        }

    }

    suspend fun getPopularMovies(): List<MovieResponse> {
        val response = client.get("$baseUrl/movie/popular") {
            parameter("language", "en-US")
            parameter("page", 1)
        }
        val rawResponse = response.body<String>()

        val body = response.body<TMDbResponse>()

        return response.body<TMDbResponse>().results
    }

    suspend fun searchMovies(title: String): List<MovieResponse> {
        val response = client.get("$baseUrl/search/movie?query=$title") {
            parameter("language", "en-US")
            parameter("page", 1)
        }
        return response.body<TMDbResponse>().results
    }

    fun toMovie(response: MovieResponse, onWatchlist: Boolean = false): Movie {
        return Movie(
            id = response.id,
            title = response.title,
            posterPath = response.posterPath,
            onWatchlist = onWatchlist
        )
    }
}
```

Itt konfiguráljuk a Ktor kliensünket, aminek segítségével fogunk kommunikálni az API-val. A `ContentNegotiation` plugin az adatcsere formátumát állítja Json-re, az `Auth` plugin pedig megadja, hogy `BearerToken` alapú hitelesítést kérünk. Itt adjuk meg az API kulcsunkat, illesszük be a megfelelő kulcsot ide! Ezen felül itt adhatnánk meg refresh tokent is, amennyiben szükségünk lenne rá.

!!!Info "API kulcs tárolása"
	Valós, éles alkalmazásban az esetek többségében ne tároljunk API kulcsot kliensoldalon! Amennyiben fizetős kulcsról van szó, semmiképp se adjuk a kliens kezébe, mert annak kódja kellő szakértelem mellett visszafejthető, így az API kulcsunk a felhasználó kezébe kerülhet. Ezen felül a felhasználó nagyobb jogosultságokat is szerezhet így az alkalmazásunkhoz, mint amivel rendelkeznie kellene, hiszen közvetlenül hívhatja az API-t a kulcs segítségével. Helyette a kulcsot tároljuk szerver oldalon, és a kliensünk kommunikáljon azzal. Amennyiben ez nem okoz gondot, tárolhatjuk kliensoldalon a kulcsot, de ekkor is inkább egy környezeti változóból célszerű kiolvasni, ne közvetlenül a forráskódba illesszük be.
	
Megjegyzendő, hogy a megfelelő platform-specifikus `HttpClient`-et nem szükséges nekünk kézzel beállítanunk (pl. expect-actual segítségével), a Ktor automatikusan magától megtalálja az adott platformhoz megfelelő enginet (OkHttp, Darwin).

Valsítsuk most meg a reposytorykat is!

`MovieRepository.kt:`
```kotlin
package hu.bme.aut.moviedb.repository

import androidx.room.RoomDatabase
import androidx.sqlite.driver.bundled.BundledSQLiteDriver
import hu.bme.aut.moviedb.data.Movie
import hu.bme.aut.moviedb.database.MovieDatabase
import hu.bme.aut.moviedb.database.MovieEntity
import hu.bme.aut.moviedb.network.ApiService
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.IO

class MovieRepository(
    private val apiService: ApiService,
    databaseBuilder: RoomDatabase.Builder<MovieDatabase>
) {
    private val database: MovieDatabase by lazy {
        getRoomDatabase(databaseBuilder)
    }

    private val movieDao by lazy { database.movieDao() }

    private fun getRoomDatabase(
        builder: RoomDatabase.Builder<MovieDatabase>
    ): MovieDatabase {
        return builder
            .fallbackToDestructiveMigrationOnDowngrade(true)
            .setDriver(BundledSQLiteDriver())
            .setQueryCoroutineContext(Dispatchers.IO)
            .build()
    }

    suspend fun getMovies(query: String): List<Movie> {
        val cached = if (query.isBlank()) {
            movieDao.getAllCachedMovies()
        } else {
            movieDao.searchCached(query)
        }

        return if (cached.isNotEmpty()) {
            cached.map { Movie(it.id, it.title, it.posterPath) }
        } else {
            val responses = if (query.isBlank()) {
                apiService.getPopularMovies()
            } else {
                apiService.searchMovies(query)
            }
            val movies = responses.map { apiService.toMovie(it) }

            val entities = movies.map { MovieEntity(it.id, it.title, it.posterPath) }
            movieDao.insertMovies(entities)

            movies
        }
    }

    suspend fun refreshCache() {
        val popular = apiService.getPopularMovies()
        val entities = popular.map { MovieEntity(it.id, it.title, it.posterPath) }
        movieDao.insertMovies(entities)
    }
}
```

`WatchlistRepository.kt:`

```kotlin
package hu.bme.aut.moviedb.repository


import app.cash.sqldelight.coroutines.asFlow
import app.cash.sqldelight.coroutines.mapToList
import hu.bme.aut.moviedb.data.Movie
import hu.bme.aut.moviedb.database.DatabaseDriverFactory
import hu.bme.aut.moviedb.database.watchlist.WatchlistDatabase
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.IO
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

@Suppress("EQUALITY_NOT_APPLICABLE_WARNING")
class WatchlistRepository(
    driverFactory: DatabaseDriverFactory
) {
    private val database = WatchlistDatabase(driverFactory.createDriver())
    private val queries = database.watchlistMovieQueries

    fun getAllWatchlistMovies(): Flow<List<Movie>> {
        return queries.selectAll()
            .asFlow()
            .mapToList(Dispatchers.IO)
            .map { list ->
                list.map { movie ->
                    Movie(
                        id = movie.id.toInt(),
                        title = movie.title,
                        posterPath = movie.poster_path,
                        onWatchlist = true
                    )
                }
            }
    }

    suspend fun addToWatchlist(movie: Movie) {
        queries.insert(
            id = movie.id.toLong(),
            title = movie.title,
            poster_path = movie.posterPath
        )
    }

    suspend fun removeFromWatchlist(movieId: Int) {
        queries.delete(movieId.toLong())
    }

    suspend fun isOnWatchlist(movieId: Int): Boolean {
        return queries.isInWatchlist(movieId.toLong()).executeAsOne()
    }

    suspend fun getWatchlistCount(): Long {
        return queries.count().executeAsOne()
    }
}
```

Mindkét repository adatforrásként szolgál a ViewModelek felé. Tekintsük át röviden őket, értelmezzük azokat!

 A `MovieRepository` átveszi a korábban, platform-specifikusan előállított `RoomDatabase.Builder` példányunkat, és annak segítségével felépíti a Room adatbázist. Ennek Dao objektumán keresztül tud adatot írni oda, illetve olvasni onnan. A lazy inicializáció segítségével csak akkor fogja inicializálni az adatbázist, amikor először ténylegesen szükség van rá. A `WatchlistRepository` hasonlóan működik, viszont itt az SQLDelight adatbázist hozzuk létre és ehhez a korábban létrehozott `DatabaseDriverFactory` példányunkat használjuk fel. Ezután az adatbázis `queries` tulajdonságán keresztül tudunk írni bele, illetve olvasni belőle.
 
 Végezetül valósítsuk meg az alkalmazásunk platform-specifikus belépési pontjait is! Ne felejtsük el, hogy ezeket nem a `shared` modulon belül, hanem a külső projektekben találjuk, kivéve az iOS-t!. Az adott platform-specifikus belépési pontból természetesen el tudjuk érni a `shared`-en belül lévő, adott platformra vonatkozó kódot is.
 
 `MainActivity.kt:`
 
```kotlin
 package hu.bme.aut.moviedb

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import hu.bme.aut.moviedb.database.DatabaseDriverFactory
import hu.bme.aut.moviedb.database.getDatabaseBuilder

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            App(
                isDesktop = false,
                databaseDriverFactory = DatabaseDriverFactory(applicationContext),
                databaseBuilder = getDatabaseBuilder(applicationContext)
            )
        }
    }
}
```
 
`main.kt:`
 
```kotlin
 package hu.bme.aut.moviedb

import hu.bme.aut.moviedb.database.DatabaseDriverFactory

import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application
import hu.bme.aut.moviedb.database.getDatabaseBuilder

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        title = "Movie DB"
    ) {
        App(
            isDesktop = true,
            databaseDriverFactory = DatabaseDriverFactory(),
            databaseBuilder = getDatabaseBuilder()
        )
    }
}
```
 
`MainViewController.kt:`

```kotlin
package hu.bme.aut.moviedb

import hu.bme.aut.moviedb.database.DatabaseDriverFactory
import androidx.compose.ui.window.ComposeUIViewController
import hu.bme.aut.moviedb.database.getDatabaseBuilder

fun mainViewController() = ComposeUIViewController  {
        App(
            isDesktop = false,
            databaseDriverFactory = DatabaseDriverFactory(),
            databaseBuilder = getDatabaseBuilder()
        )
}
```

Ezzel elkészültünk, fordítsuk és futtassuk az alkalmazást! Próbáljuk ki mobilos és desktop platformon is! A labor elején látottakhoz hasonlót kell látnunk.

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk **2 képernyőképet**, amelyen látszik az alkalmazás mobilos és desktopos platformon is! Mindkét képen legalább 1-2 filmet helyezzünk figyelő listára is! 

	A képeket a megoldásban a repositoryba f1a.png és f1b.png néven töltsük fel!


## Önálló feladat - MovieDetailsScreen (1 pont)

Valósítsuk meg, hogy az egyes filmek részletesebb adatait is meg lehessen tekinteni egy részletező képernyőn! Egy adott filmre kattintva jelenítsük meg annak részletesebb adatait! A szükséges adatokat a labor során használt API-tól tudjuk lekérdezni.


??? tip "Segítség"
	A megoldás menete például a következő lehet:
	
	*	A szükséges endpoint az API-n: https://api.themoviedb.org/3/movie/{movie_id}
	*   Vegyünk fel új API hívást a Service-ünkben!
	*   Vegyük fel a hozzátartozó új szükséges adatosztályokat!
	*   Valósítsuk meg a hozzátartozó új Screent!
	*   Bővítsük ki a navigációt, egy adott filmre kattintva az új képernyőre, onnan vissza kell tudnunk jutni!

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk **egy képernyőképet**, amelyen látszik egy film részletező képernyője!

	A képet a megoldásban a repositoryba f2.png néven töltsük fel!

## Önálló feladat - Részletes adatok tárolása Room adatbázisban (1 pont)

Valósítsuk meg, hogy a filmek részletes adatait is tároljuk el adatbázisban! A részletes adatok legyenek külön entitások, mutasson rájuk idegenkulcs a filmek entitásaiból!

??? tip "Segítség"
	A megoldás menete például a következő lehet:
	
	*   Vegyük fel az új entitást!
	*   Készítsük el a hozzátartozó Daot a szükséges metódusokkal!
	*   Egészítsük ki az adatbázist, hogy az új entitást is tartalmazza!
	*   Módosítsuk a szükséges részeit a meglévő alkalmazásnak!

!!!example "BEADANDÓ (1 pont)" 
	Készítsünk **egy képernyőképet**, amin látszik az új Dao és új Entitás kódja!

	A képet a megoldásban a repositoryba f3.png néven töltsük fel!
