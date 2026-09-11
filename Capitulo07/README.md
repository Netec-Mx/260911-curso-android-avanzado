---LAB_START---
LAB_ID: 07-00-01
---MARKDOWN---
# WeatherGeoApp — Proyecto Final Integrador

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 144 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |

## 2. Descripción General

En esta práctica culminante del curso avanzado, construirás **WeatherGeoApp**: una aplicación Android completa que consume la API pública de Open-Meteo para mostrar datos meteorológicos basados en la ubicación GPS del usuario o en ciudades buscadas manualmente. La aplicación implementa arquitectura MVVM con separación por capas, persistencia offline-first con Room, preferencias reactivas con DataStore, consumo de API REST con Retrofit 3 + OkHttp 5, y una interfaz completa en Jetpack Compose con Material 3 y navegación multi-pantalla. Este proyecto integra todas las competencias técnicas desarrolladas a lo largo del curso en un producto funcional de calidad profesional.

## 3. Objetivos de Aprendizaje

Al completar esta práctica, serás capaz de:

- [ ] Diseñar e implementar una arquitectura MVVM completa con capas `data`, `domain` y `presentation` para una aplicación Android de nivel producción
- [ ] Consumir la API REST de Open-Meteo usando Retrofit 3.0.0 con OkHttp BOM 5.5.0, implementando manejo de errores con `sealed class`, interceptores de logging y serialización JSON con Gson
- [ ] Persistir datos localmente con Room 2.8.4 y sincronizar con datos remotos mediante el patrón offline-first (cache-then-network)
- [ ] Gestionar preferencias de usuario con DataStore Preferences 1.1.1 de forma reactiva usando Flows y exponer estado de UI con `StateFlow` + `collectAsStateWithLifecycle`
- [ ] Construir una UI multi-pantalla con Jetpack Compose, Navigation Compose, Material 3 y componentes avanzados como `SwipeToDismissBox`

## 4. Prerrequisitos

### Conocimientos Requeridos

| Competencia | Nivel |
|---|---|
| Patrón MVVM y separación de responsabilidades | Intermedio-Alto |
| Corrutinas Kotlin: `viewModelScope`, `Flow`, `StateFlow` | Intermedio-Alto |
| Jetpack Compose: composables, estado, `LazyColumn` | Intermedio |
| APIs REST: verbos HTTP, códigos de respuesta, JSON | Intermedio |
| SQL básico y bases de datos relacionales | Básico-Intermedio |
| Práctica 6 completada (permisos, ubicación GPS) | Obligatorio |

### Acceso Requerido

- Android Studio Quail 3 (2026.1.3 Patch 1) instalado con SDKs API 36 y 37
- Conexión a Internet estable para consumir la API de Open-Meteo
- AVD configurado con API 35 o superior (recomendado API 36)
- Proyecto SensorMapApp de Práctica 6 accesible para reutilizar `LocationRepository`

## 5. Entorno del Laboratorio

### Hardware Mínimo

| Componente | Requisito |
|---|---|
| Procesador | Intel Core i7 8ª gen+ / AMD Ryzen 7+ / Apple M1+ |
| RAM | 16 GB mínimo (32 GB recomendado) |
| Disco | 60 GB libres en SSD |
| Virtualización | VT-x / AMD-V habilitado en BIOS |

### Software y Versiones

| Herramienta | Versión Exacta |
|---|---|
| Android Studio | Quail 3 — 2026.1.3 Patch 1 |
| Kotlin | 2.2.10 |
| AGP | 9.3.2 |
| Gradle Wrapper | 8.14.1 |
| KSP | 2.2.10-1.0.31 |
| Compose BOM | 2026.02.01 |
| Retrofit | 3.0.0 |
| OkHttp BOM | 5.5.0 |
| Gson | 2.11.0 |
| Room | 2.8.4 |
| DataStore Preferences | 1.1.1 |
| Play Services Location | 21.4.0 |
| compileSdk / targetSdk | 37 |
| minSdk | 30 |

### Preparación del AVD

Antes de comenzar, verifica que tienes un AVD funcional con ubicación configurable:

1. Abre **Device Manager** en Android Studio
2. Verifica que existe un AVD con API 35 o 36
3. Inicia el AVD → abre **Extended Controls** (⋮) → **Location**
4. Configura las coordenadas: **Latitude: 19.4326**, **Longitude: -99.1332** (Ciudad de México)
5. Haz clic en **Set Location**

## 6. Instrucciones Paso a Paso

---

### Paso 1: Crear el Proyecto y Configurar la Estructura Base

**Objetivo:** Crear el proyecto WeatherGeoApp con la plantilla Empty Activity y configurar el catálogo de versiones, dependencias y estructura de paquetes MVVM.

**Instrucciones:**

1. En Android Studio, selecciona **File → New → New Project → Empty Activity** (plantilla Compose).

2. Configura los parámetros del proyecto:
   - **Name:** `WeatherGeoApp`
   - **Package name:** `com.curso.android.avanzado.practica7`
   - **Save location:** `~/AndroidStudioProjects/AvanzadoKotlin/Practica7_WeatherGeoApp`
   - **Minimum SDK:** API 30 (Android 11)
   - **Build configuration language:** Kotlin DSL

3. Una vez creado el proyecto, abre `gradle/libs.versions.toml` y reemplaza su contenido completo con el siguiente catálogo de versiones:

```toml
[versions]
agp = "9.3.2"
kotlin = "2.2.10"
ksp = "2.2.10-1.0.31"
composeBom = "2026.02.01"
activityCompose = "1.13.0"
coreKtx = "1.19.0"
lifecycleRuntimeKtx = "2.6.1"
navigationCompose = "2.9.0"
room = "2.8.4"
retrofit = "3.0.0"
okhttpBom = "5.5.0"
gson = "2.11.0"
datastore = "1.1.1"
playServicesLocation = "21.4.0"
junit = "4.13.2"
androidxJunit = "1.3.0"
espressoCore = "3.7.0"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-ui = { group = "androidx.compose.ui", name = "ui" }
androidx-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
androidx-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
androidx-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-material3 = { group = "androidx.compose.material3", name = "material3" }
androidx-material-icons-extended = { group = "androidx.compose.material", name = "material-icons-extended" }
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigationCompose" }
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
okhttp-bom = { group = "com.squareup.okhttp3", name = "okhttp-bom", version.ref = "okhttpBom" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor" }
gson = { group = "com.google.code.gson", name = "gson", version.ref = "gson" }
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }
play-services-location = { group = "com.google.android.gms", name = "play-services-location", version.ref = "playServicesLocation" }
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidxJunit" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

4. Abre el archivo `build.gradle.kts` **del proyecto raíz** y verifica que contiene:

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.ksp) apply false
}
```

5. Abre el archivo `app/build.gradle.kts` y reemplaza su contenido con:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)
}

android {
    namespace = "com.curso.android.avanzado.practica7"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.curso.android.avanzado.practica7"
        minSdk = 30
        targetSdk = 37
        versionCode = 1
        versionName = "1.0"
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }

    buildFeatures {
        compose = true
    }
}

dependencies {
    // Core
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.runtime.compose)
    implementation(libs.androidx.lifecycle.viewmodel.compose)
    implementation(libs.androidx.activity.compose)

    // Compose
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.material.icons.extended)
    implementation(libs.androidx.navigation.compose)
    debugImplementation(libs.androidx.ui.tooling)

    // Room
    implementation(libs.room.runtime)
    implementation(libs.room.ktx)
    ksp(libs.room.compiler)

    // Retrofit + OkHttp
    implementation(platform(libs.okhttp.bom))
    implementation(libs.retrofit)
    implementation(libs.retrofit.converter.gson)
    implementation(libs.okhttp)
    implementation(libs.okhttp.logging)
    implementation(libs.gson)

    // DataStore
    implementation(libs.datastore.preferences)

    // Location
    implementation(libs.play.services.location)

    // Testing
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
}
```

6. Abre `app/src/main/AndroidManifest.xml` y agrega los permisos necesarios antes de la etiqueta `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="WeatherGeoApp"
        android:supportsRtl="true"
        android:theme="@style/Theme.WeatherGeoApp">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.WeatherGeoApp">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

7. Crea la estructura de paquetes dentro de `com.curso.android.avanzado.practica7`. Haz clic derecho en el paquete raíz → **New → Package** y crea los siguientes paquetes uno por uno:

```
com.curso.android.avanzado.practica7/
├── data/
│   ├── api/
│   ├── db/
│   ├── datastore/
│   └── repository/
├── domain/
│   └── model/
├── presentation/
│   ├── navigation/
│   ├── screens/
│   └── viewmodel/
└── MainActivity.kt
```

8. Sincroniza el proyecto: **File → Sync Project with Gradle Files**. Espera a que la compilación finalice sin errores.

**Resultado Esperado:** El proyecto compila sin errores. La estructura de paquetes refleja la arquitectura MVVM por capas. Todas las dependencias se resuelven correctamente desde `libs.versions.toml`.

**Verificación:**
- En la ventana **Build**, confirma el mensaje `BUILD SUCCESSFUL`
- En el panel **Project**, verifica que los 8 subpaquetes existen bajo el paquete raíz
- Abre **File → Project Structure → Dependencies** y confirma que Retrofit 3.0.0, Room 2.8.4 y OkHttp BOM 5.5.0 aparecen listados

---

### Paso 2: Implementar la Capa de Red — Retrofit + OkHttp

**Objetivo:** Crear el servicio de API, los modelos de respuesta JSON y la configuración de red para consumir la API de Open-Meteo.

**Instrucciones:**

1. Crea el archivo `data/api/WeatherResponse.kt` con los data classes que mapean la respuesta JSON de Open-Meteo:

```kotlin
package com.curso.android.avanzado.practica7.data.api

import com.google.gson.annotations.SerializedName

data class WeatherResponse(
    @SerializedName("latitude") val latitude: Double,
    @SerializedName("longitude") val longitude: Double,
    @SerializedName("timezone") val timezone: String,
    @SerializedName("hourly") val hourly: HourlyData
)

data class HourlyData(
    @SerializedName("time") val time: List<String>,
    @SerializedName("temperature_2m") val temperature2m: List<Double>,
    @SerializedName("weather_code") val weatherCode: List<Int>
)
```

2. Crea el archivo `data/api/WeatherApiService.kt` con la interfaz de Retrofit:

```kotlin
package com.curso.android.avanzado.practica7.data.api

import retrofit2.http.GET
import retrofit2.http.Query

interface WeatherApiService {

    @GET("v1/forecast")
    suspend fun getWeatherForecast(
        @Query("latitude") latitude: Double,
        @Query("longitude") longitude: Double,
        @Query("hourly") hourly: String = "temperature_2m,weather_code",
        @Query("timezone") timezone: String = "auto",
        @Query("forecast_days") forecastDays: Int = 1
    ): WeatherResponse
}
```

3. Crea el archivo `data/api/RetrofitInstance.kt` con la configuración del cliente HTTP:

```kotlin
package com.curso.android.avanzado.practica7.data.api

import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

object RetrofitInstance {

    private const val BASE_URL = "https://api.open-meteo.com/"

    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    }

    private val okHttpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .build()

    private val retrofit: Retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val weatherApiService: WeatherApiService =
        retrofit.create(WeatherApiService::class.java)
}
```

4. Crea el archivo `data/api/WeatherRemoteDataSource.kt` que envuelve las llamadas en `Result<T>`:

```kotlin
package com.curso.android.avanzado.practica7.data.api

class WeatherRemoteDataSource(
    private val apiService: WeatherApiService = RetrofitInstance.weatherApiService
) {

    suspend fun fetchWeather(
        latitude: Double,
        longitude: Double
    ): Result<WeatherResponse> {
        return runCatching {
            apiService.getWeatherForecast(
                latitude = latitude,
                longitude = longitude
            )
        }
    }
}
```

5. Para verificar que la capa de red funciona, abre `MainActivity.kt` temporalmente y agrega una prueba rápida dentro de `onCreate` (la eliminaremos después):

```kotlin
package com.curso.android.avanzado.practica7

import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material3.Text
import androidx.lifecycle.lifecycleScope
import com.curso.android.avanzado.practica7.data.api.WeatherRemoteDataSource
import kotlinx.coroutines.launch

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Prueba temporal de la capa de red
        lifecycleScope.launch {
            val dataSource = WeatherRemoteDataSource()
            val result = dataSource.fetchWeather(19.4326, -99.1332)
            result.onSuccess { response ->
                Log.d("WeatherTest", "Timezone: ${response.timezone}")
                Log.d("WeatherTest", "Temps: ${response.hourly.temperature2m.take(5)}")
            }.onFailure { error ->
                Log.e("WeatherTest", "Error: ${error.message}")
            }
        }

        setContent {
            Text("WeatherGeoApp - Verificando API...")
        }
    }
}
```

6. Ejecuta la aplicación en el AVD. Abre **Logcat** y filtra por `WeatherTest`.

**Resultado Esperado:** En Logcat aparecen mensajes como:

```
D/WeatherTest: Timezone: America/Mexico_City
D/WeatherTest: Temps: [14.2, 13.8, 13.5, 13.1, 12.9]
```

Además, en Logcat con filtro `OkHttp`, verás el log completo de la petición HTTP con headers, URL y cuerpo de respuesta JSON.

**Verificación:**
- Confirma que la respuesta contiene `latitude`, `longitude`, `timezone` y datos en `hourly`
- Verifica que el `HttpLoggingInterceptor` muestra la URL completa: `https://api.open-meteo.com/v1/forecast?latitude=19.4326&longitude=-99.1332&hourly=temperature_2m%2Cweather_code&timezone=auto&forecast_days=1`
- Si ves `Error: Unable to resolve host`, verifica la conexión a Internet del AVD

---

### Paso 3: Implementar Room — Persistencia Local y Patrón Offline-First

**Objetivo:** Definir entidades, DAOs y la base de datos Room para almacenar datos meteorológicos y ciudades favoritas, habilitando el funcionamiento offline.

**Instrucciones:**

1. Crea el archivo `data/db/WeatherEntity.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.db

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "weather_cache")
data class WeatherEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val cityName: String,
    val latitude: Double,
    val longitude: Double,
    val temperatures: String,   // JSON serializado: "[14.2, 13.8, ...]"
    val weatherCodes: String,   // JSON serializado: "[0, 1, 2, ...]"
    val times: String,          // JSON serializado: "[\"2025-...\", ...]"
    val timestamp: Long = System.currentTimeMillis()
)
```

2. Crea el archivo `data/db/FavoriteCityEntity.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.db

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "favorite_cities")
data class FavoriteCityEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val name: String,
    val latitude: Double,
    val longitude: Double
)
```

3. Crea el archivo `data/db/WeatherDao.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.db

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import kotlinx.coroutines.flow.Flow

@Dao
interface WeatherDao {

    @Query("SELECT * FROM weather_cache ORDER BY timestamp DESC LIMIT 1")
    fun getLatestWeather(): Flow<WeatherEntity?>

    @Query("SELECT * FROM weather_cache WHERE cityName = :cityName ORDER BY timestamp DESC LIMIT 1")
    fun getWeatherByCity(cityName: String): Flow<WeatherEntity?>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertWeather(weather: WeatherEntity)

    @Query("DELETE FROM weather_cache WHERE timestamp < :threshold")
    suspend fun deleteOldCache(threshold: Long)
}
```

4. Crea el archivo `data/db/FavoriteCityDao.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.db

import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import kotlinx.coroutines.flow.Flow

@Dao
interface FavoriteCityDao {

    @Query("SELECT * FROM favorite_cities ORDER BY name ASC")
    fun getAllFavorites(): Flow<List<FavoriteCityEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertFavorite(city: FavoriteCityEntity)

    @Delete
    suspend fun deleteFavorite(city: FavoriteCityEntity)

    @Query("SELECT EXISTS(SELECT 1 FROM favorite_cities WHERE name = :name)")
    suspend fun isFavorite(name: String): Boolean
}
```

5. Crea el archivo `data/db/WeatherDatabase.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.db

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(
    entities = [WeatherEntity::class, FavoriteCityEntity::class],
    version = 1,
    exportSchema = false
)
abstract class WeatherDatabase : RoomDatabase() {

    abstract fun weatherDao(): WeatherDao
    abstract fun favoriteCityDao(): FavoriteCityDao

    companion object {
        @Volatile
        private var INSTANCE: WeatherDatabase? = null

        fun getInstance(context: Context): WeatherDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    WeatherDatabase::class.java,
                    "weather_geo_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

6. Crea el archivo `data/db/WeatherLocalDataSource.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.db

import kotlinx.coroutines.flow.Flow

class WeatherLocalDataSource(
    private val weatherDao: WeatherDao,
    private val favoriteCityDao: FavoriteCityDao
) {

    fun getLatestWeather(): Flow<WeatherEntity?> =
        weatherDao.getLatestWeather()

    fun getWeatherByCity(cityName: String): Flow<WeatherEntity?> =
        weatherDao.getWeatherByCity(cityName)

    suspend fun saveWeather(entity: WeatherEntity) {
        weatherDao.insertWeather(entity)
    }

    suspend fun clearOldCache(maxAgeMillis: Long = 3_600_000) { // 1 hora
        val threshold = System.currentTimeMillis() - maxAgeMillis
        weatherDao.deleteOldCache(threshold)
    }

    fun getAllFavorites(): Flow<List<FavoriteCityEntity>> =
        favoriteCityDao.getAllFavorites()

    suspend fun addFavorite(city: FavoriteCityEntity) {
        favoriteCityDao.insertFavorite(city)
    }

    suspend fun removeFavorite(city: FavoriteCityEntity) {
        favoriteCityDao.deleteFavorite(city)
    }

    suspend fun isFavorite(name: String): Boolean =
        favoriteCityDao.isFavorite(name)
}
```

7. Compila el proyecto: **Build → Make Project** (Ctrl+F9 / Cmd+F9).

**Resultado Esperado:** La compilación es exitosa. KSP genera las implementaciones de los DAOs y la base de datos en el directorio `build/generated/ksp/`.

**Verificación:**
- `BUILD SUCCESSFUL` sin errores ni warnings de Room
- En **Project** (vista Files), navega a `app/build/generated/ksp/debug/` y confirma que existen archivos como `WeatherDao_Impl.java` y `WeatherDatabase_Impl.java`

---

### Paso 4: Implementar el Modelo de Dominio y el Repositorio Offline-First

**Objetivo:** Crear los modelos de dominio, las sealed classes de estado de UI y el repositorio que coordina datos remotos y locales con estrategia cache-then-network.

**Instrucciones:**

1. Crea el archivo `domain/model/WeatherInfo.kt` con los modelos de dominio y estados de UI:

```kotlin
package com.curso.android.avanzado.practica7.domain.model

data class WeatherInfo(
    val cityName: String,
    val latitude: Double,
    val longitude: Double,
    val hourlyForecasts: List<HourlyForecast>,
    val isFromCache: Boolean = false
)

data class HourlyForecast(
    val time: String,
    val temperatureCelsius: Double,
    val weatherCode: Int
) {
    val temperatureFahrenheit: Double
        get() = temperatureCelsius * 9.0 / 5.0 + 32.0

    val weatherDescription: String
        get() = when (weatherCode) {
            0 -> "Despejado"
            1, 2, 3 -> "Parcialmente nublado"
            45, 48 -> "Niebla"
            51, 53, 55 -> "Llovizna"
            61, 63, 65 -> "Lluvia"
            71, 73, 75 -> "Nevada"
            80, 81, 82 -> "Chubascos"
            95, 96, 99 -> "Tormenta"
            else -> "Desconocido ($weatherCode)"
        }

    val weatherEmoji: String
        get() = when (weatherCode) {
            0 -> "☀️"
            1, 2, 3 -> "⛅"
            45, 48 -> "🌫️"
            51, 53, 55 -> "🌦️"
            61, 63, 65 -> "🌧️"
            71, 73, 75 -> "🌨️"
            80, 81, 82 -> "🌧️"
            95, 96, 99 -> "⛈️"
            else -> "❓"
        }
}

data class FavoriteCity(
    val id: Long = 0,
    val name: String,
    val latitude: Double,
    val longitude: Double
)
```

2. Crea el archivo `domain/model/WeatherUiState.kt`:

```kotlin
package com.curso.android.avanzado.practica7.domain.model

sealed class WeatherUiState {
    data object Loading : WeatherUiState()
    data object Empty : WeatherUiState()
    data class Success(
        val weatherInfo: WeatherInfo,
        val isStale: Boolean = false  // true si los datos son del caché y no se pudo actualizar
    ) : WeatherUiState()
    data class Error(val message: String) : WeatherUiState()
}
```

3. Crea el archivo `data/repository/WeatherRepository.kt` con la lógica offline-first:

```kotlin
package com.curso.android.avanzado.practica7.data.repository

import com.curso.android.avanzado.practica7.data.api.WeatherRemoteDataSource
import com.curso.android.avanzado.practica7.data.api.WeatherResponse
import com.curso.android.avanzado.practica7.data.db.FavoriteCityEntity
import com.curso.android.avanzado.practica7.data.db.WeatherEntity
import com.curso.android.avanzado.practica7.data.db.WeatherLocalDataSource
import com.curso.android.avanzado.practica7.domain.model.FavoriteCity
import com.curso.android.avanzado.practica7.domain.model.HourlyForecast
import com.curso.android.avanzado.practica7.domain.model.WeatherInfo
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

class WeatherRepository(
    private val remoteDataSource: WeatherRemoteDataSource,
    private val localDataSource: WeatherLocalDataSource
) {

    private val gson = Gson()

    /**
     * Estrategia offline-first:
     * 1. Intenta obtener datos de la API
     * 2. Si tiene éxito, guarda en Room y retorna datos frescos
     * 3. Si falla, retorna datos cacheados marcados como "desactualizados"
     */
    suspend fun fetchWeather(
        latitude: Double,
        longitude: Double,
        cityName: String = "Ubicación actual"
    ): WeatherInfo {
        // Limpiar caché antiguo
        localDataSource.clearOldCache()

        val remoteResult = remoteDataSource.fetchWeather(latitude, longitude)

        return remoteResult.fold(
            onSuccess = { response ->
                val entity = mapResponseToEntity(response, cityName)
                localDataSource.saveWeather(entity)
                mapEntityToWeatherInfo(entity, isFromCache = false)
            },
            onFailure = {
                // Intentar obtener del caché
                throw CacheOrNetworkException(
                    networkError = it,
                    cityName = cityName
                )
            }
        )
    }

    fun observeLatestWeather(): Flow<WeatherInfo?> {
        return localDataSource.getLatestWeather().map { entity ->
            entity?.let { mapEntityToWeatherInfo(it, isFromCache = true) }
        }
    }

    // --- Favoritos ---

    fun observeFavorites(): Flow<List<FavoriteCity>> {
        return localDataSource.getAllFavorites().map { entities ->
            entities.map { entity ->
                FavoriteCity(
                    id = entity.id,
                    name = entity.name,
                    latitude = entity.latitude,
                    longitude = entity.longitude
                )
            }
        }
    }

    suspend fun addFavorite(city: FavoriteCity) {
        localDataSource.addFavorite(
            FavoriteCityEntity(
                name = city.name,
                latitude = city.latitude,
                longitude = city.longitude
            )
        )
    }

    suspend fun removeFavorite(city: FavoriteCity) {
        localDataSource.removeFavorite(
            FavoriteCityEntity(
                id = city.id,
                name = city.name,
                latitude = city.latitude,
                longitude = city.longitude
            )
        )
    }

    // --- Mappers ---

    private fun mapResponseToEntity(
        response: WeatherResponse,
        cityName: String
    ): WeatherEntity {
        return WeatherEntity(
            cityName = cityName,
            latitude = response.latitude,
            longitude = response.longitude,
            temperatures = gson.toJson(response.hourly.temperature2m),
            weatherCodes = gson.toJson(response.hourly.weatherCode),
            times = gson.toJson(response.hourly.time),
            timestamp = System.currentTimeMillis()
        )
    }

    private fun mapEntityToWeatherInfo(
        entity: WeatherEntity,
        isFromCache: Boolean
    ): WeatherInfo {
        val listDoubleType = object : TypeToken<List<Double>>() {}.type
        val listIntType = object : TypeToken<List<Int>>() {}.type
        val listStringType = object : TypeToken<List<String>>() {}.type

        val temps: List<Double> = gson.fromJson(entity.temperatures, listDoubleType)
        val codes: List<Int> = gson.fromJson(entity.weatherCodes, listIntType)
        val times: List<String> = gson.fromJson(entity.times, listStringType)

        val forecasts = times.indices.map { i ->
            HourlyForecast(
                time = times[i],
                temperatureCelsius = temps.getOrElse(i) { 0.0 },
                weatherCode = codes.getOrElse(i) { 0 }
            )
        }

        return WeatherInfo(
            cityName = entity.cityName,
            latitude = entity.latitude,
            longitude = entity.longitude,
            hourlyForecasts = forecasts,
            isFromCache = isFromCache
        )
    }
}

class CacheOrNetworkException(
    val networkError: Throwable,
    val cityName: String
) : Exception("Error de red para $cityName: ${networkError.message}")
```

4. Compila el proyecto para verificar que no hay errores.

**Resultado Esperado:** El proyecto compila exitosamente. Las clases de dominio y el repositorio están listos para ser consumidos por los ViewModels.

**Verificación:**
- `BUILD SUCCESSFUL`
- No hay warnings de tipos no resueltos en `WeatherRepository`
- Las importaciones de Gson, Room entities y modelos de dominio se resuelven correctamente

---

### Paso 5: Implementar DataStore Preferences

**Objetivo:** Crear un repositorio de preferencias de usuario con DataStore que almacene la unidad de temperatura, la última ciudad buscada y el modo oscuro, exponiendo cada preferencia como Flow reactivo.

**Instrucciones:**

1. Crea el archivo `data/datastore/UserPreferencesRepository.kt`:

```kotlin
package com.curso.android.avanzado.practica7.data.datastore

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

// Extensión para crear el DataStore como singleton
private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(
    name = "user_preferences"
)

class UserPreferencesRepository(private val context: Context) {

    companion object {
        private val TEMPERATURE_UNIT_KEY = stringPreferencesKey("temperature_unit")
        private val LAST_SEARCHED_CITY_KEY = stringPreferencesKey("last_searched_city")
        private val DARK_MODE_KEY = booleanPreferencesKey("dark_mode_enabled")

        const val UNIT_CELSIUS = "celsius"
        const val UNIT_FAHRENHEIT = "fahrenheit"
    }

    // --- Flows reactivos ---

    val temperatureUnit: Flow<String> = context.dataStore.data.map { prefs ->
        prefs[TEMPERATURE_UNIT_KEY] ?: UNIT_CELSIUS
    }

    val lastSearchedCity: Flow<String> = context.dataStore.data.map { prefs ->
        prefs[LAST_SEARCHED_CITY_KEY] ?: ""
    }

    val darkModeEnabled: Flow<Boolean> = context.dataStore.data.map { prefs ->
        prefs[DARK_MODE_KEY] ?: false
    }

    // --- Métodos de escritura ---

    suspend fun setTemperatureUnit(unit: String) {
        context.dataStore.edit { prefs ->
            prefs[TEMPERATURE_UNIT_KEY] = unit
        }
    }

    suspend fun setLastSearchedCity(city: String) {
        context.dataStore.edit { prefs ->
            prefs[LAST_SEARCHED_CITY_KEY] = city
        }
    }

    suspend fun setDarkModeEnabled(enabled: Boolean) {
        context.dataStore.edit { prefs ->
            prefs[DARK_MODE_KEY] = enabled
        }
    }
}
```

2. Crea el archivo `presentation/viewmodel/SettingsViewModel.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.curso.android.avanzado.practica7.data.datastore.UserPreferencesRepository
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.combine
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

data class SettingsUiState(
    val temperatureUnit: String = UserPreferencesRepository.UNIT_CELSIUS,
    val lastSearchedCity: String = "",
    val darkModeEnabled: Boolean = false
)

class SettingsViewModel(
    private val preferencesRepository: UserPreferencesRepository
) : ViewModel() {

    val uiState: StateFlow<SettingsUiState> = combine(
        preferencesRepository.temperatureUnit,
        preferencesRepository.lastSearchedCity,
        preferencesRepository.darkModeEnabled
    ) { unit, city, darkMode ->
        SettingsUiState(
            temperatureUnit = unit,
            lastSearchedCity = city,
            darkModeEnabled = darkMode
        )
    }.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5_000),
        initialValue = SettingsUiState()
    )

    fun setTemperatureUnit(unit: String) {
        viewModelScope.launch {
            preferencesRepository.setTemperatureUnit(unit)
        }
    }

    fun setDarkModeEnabled(enabled: Boolean) {
        viewModelScope.launch {
            preferencesRepository.setDarkModeEnabled(enabled)
        }
    }
}
```

3. Compila para verificar.

**Resultado Esperado:** Compilación exitosa. El `SettingsViewModel` expone un `StateFlow<SettingsUiState>` que combina tres preferencias reactivas de DataStore.

**Verificación:**
- `BUILD SUCCESSFUL`
- El `combine()` de tres Flows produce un único `StateFlow` con `stateIn`

---

### Paso 6: Implementar ViewModels — Weather, Favorites y Ubicación

**Objetivo:** Crear los ViewModels principales que gestionan el estado de la aplicación usando StateFlow, corrutinas y el patrón offline-first del repositorio.

**Instrucciones:**

1. Crea el archivo `data/repository/LocationRepository.kt` (adaptado de la Práctica 6):

```kotlin
package com.curso.android.avanzado.practica7.data.repository

import android.annotation.SuppressLint
import android.content.Context
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationServices
import com.google.android.gms.location.Priority
import com.google.android.gms.tasks.CancellationTokenSource
import kotlinx.coroutines.suspendCancellableCoroutine
import kotlin.coroutines.resume
import kotlin.coroutines.resumeWithException

data class LatLng(val latitude: Double, val longitude: Double)

class LocationRepository(context: Context) {

    private val fusedClient: FusedLocationProviderClient =
        LocationServices.getFusedLocationProviderClient(context)

    @SuppressLint("MissingPermission")
    suspend fun getCurrentLocation(): LatLng {
        return suspendCancellableCoroutine { continuation ->
            val cancellationToken = CancellationTokenSource()

            fusedClient.getCurrentLocation(
                Priority.PRIORITY_HIGH_ACCURACY,
                cancellationToken.token
            ).addOnSuccessListener { location ->
                if (location != null) {
                    continuation.resume(
                        LatLng(location.latitude, location.longitude)
                    )
                } else {
                    continuation.resumeWithException(
                        Exception("No se pudo obtener la ubicación actual")
                    )
                }
            }.addOnFailureListener { exception ->
                continuation.resumeWithException(exception)
            }

            continuation.invokeOnCancellation {
                cancellationToken.cancel()
            }
        }
    }
}
```

2. Crea el archivo `presentation/viewmodel/WeatherViewModel.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.curso.android.avanzado.practica7.data.datastore.UserPreferencesRepository
import com.curso.android.avanzado.practica7.data.repository.CacheOrNetworkException
import com.curso.android.avanzado.practica7.data.repository.LocationRepository
import com.curso.android.avanzado.practica7.data.repository.WeatherRepository
import com.curso.android.avanzado.practica7.domain.model.WeatherInfo
import com.curso.android.avanzado.practica7.domain.model.WeatherUiState
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.launch

class WeatherViewModel(
    private val weatherRepository: WeatherRepository,
    private val locationRepository: LocationRepository,
    private val preferencesRepository: UserPreferencesRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow<WeatherUiState>(WeatherUiState.Empty)
    val uiState: StateFlow<WeatherUiState> = _uiState.asStateFlow()

    private val _temperatureUnit = MutableStateFlow(UserPreferencesRepository.UNIT_CELSIUS)
    val temperatureUnit: StateFlow<String> = _temperatureUnit.asStateFlow()

    init {
        viewModelScope.launch {
            preferencesRepository.temperatureUnit.collect { unit ->
                _temperatureUnit.value = unit
            }
        }
        loadInitialData()
    }

    private fun loadInitialData() {
        viewModelScope.launch {
            val lastCity = preferencesRepository.lastSearchedCity.first()
            if (lastCity.isNotBlank()) {
                // Si hay una ciudad guardada, cargar datos del caché
                weatherRepository.observeLatestWeather().collect { cached ->
                    if (cached != null) {
                        _uiState.value = WeatherUiState.Success(
                            weatherInfo = cached,
                            isStale = true
                        )
                    }
                }
            }
        }
    }

    fun fetchWeather(latitude: Double, longitude: Double, cityName: String = "Ubicación actual") {
        viewModelScope.launch {
            _uiState.value = WeatherUiState.Loading
            try {
                val weatherInfo = weatherRepository.fetchWeather(latitude, longitude, cityName)
                _uiState.value = WeatherUiState.Success(weatherInfo = weatherInfo)
                preferencesRepository.setLastSearchedCity(cityName)
            } catch (e: CacheOrNetworkException) {
                // Intentar mostrar datos cacheados
                val cached = weatherRepository.observeLatestWeather().first()
                if (cached != null) {
                    _uiState.value = WeatherUiState.Success(
                        weatherInfo = cached,
                        isStale = true
                    )
                } else {
                    _uiState.value = WeatherUiState.Error(
                        "Sin conexión y sin datos en caché: ${e.networkError.message}"
                    )
                }
            } catch (e: Exception) {
                _uiState.value = WeatherUiState.Error(
                    e.message ?: "Error desconocido"
                )
            }
        }
    }

    fun fetchWeatherForCurrentLocation() {
        viewModelScope.launch {
            _uiState.value = WeatherUiState.Loading
            try {
                val location = locationRepository.getCurrentLocation()
                fetchWeather(location.latitude, location.longitude, "Mi ubicación")
            } catch (e: Exception) {
                _uiState.value = WeatherUiState.Error(
                    "Error al obtener ubicación: ${e.message}"
                )
            }
        }
    }
}
```

3. Crea el archivo `presentation/viewmodel/FavoritesViewModel.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.curso.android.avanzado.practica7.data.repository.WeatherRepository
import com.curso.android.avanzado.practica7.domain.model.FavoriteCity
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class FavoritesViewModel(
    private val weatherRepository: WeatherRepository
) : ViewModel() {

    val favorites: StateFlow<List<FavoriteCity>> =
        weatherRepository.observeFavorites()
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5_000),
                initialValue = emptyList()
            )

    fun addFavorite(city: FavoriteCity) {
        viewModelScope.launch {
            weatherRepository.addFavorite(city)
        }
    }

    fun removeFavorite(city: FavoriteCity) {
        viewModelScope.launch {
            weatherRepository.removeFavorite(city)
        }
    }
}
```

4. Crea el archivo `presentation/viewmodel/ViewModelFactory.kt` para instanciar los ViewModels con dependencias manuales (sin Hilt):

```kotlin
package com.curso.android.avanzado.practica7.presentation.viewmodel

import android.content.Context
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.curso.android.avanzado.practica7.data.api.WeatherRemoteDataSource
import com.curso.android.avanzado.practica7.data.datastore.UserPreferencesRepository
import com.curso.android.avanzado.practica7.data.db.WeatherDatabase
import com.curso.android.avanzado.practica7.data.db.WeatherLocalDataSource
import com.curso.android.avanzado.practica7.data.repository.LocationRepository
import com.curso.android.avanzado.practica7.data.repository.WeatherRepository

class ViewModelFactory(private val context: Context) : ViewModelProvider.Factory {

    private val database by lazy { WeatherDatabase.getInstance(context) }
    private val localDataSource by lazy {
        WeatherLocalDataSource(database.weatherDao(), database.favoriteCityDao())
    }
    private val remoteDataSource by lazy { WeatherRemoteDataSource() }
    private val weatherRepository by lazy {
        WeatherRepository(remoteDataSource, localDataSource)
    }
    private val locationRepository by lazy { LocationRepository(context) }
    private val preferencesRepository by lazy { UserPreferencesRepository(context) }

    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return when {
            modelClass.isAssignableFrom(WeatherViewModel::class.java) -> {
                WeatherViewModel(
                    weatherRepository, locationRepository, preferencesRepository
                ) as T
            }
            modelClass.isAssignableFrom(FavoritesViewModel::class.java) -> {
                FavoritesViewModel(weatherRepository) as T
            }
            modelClass.isAssignableFrom(SettingsViewModel::class.java) -> {
                SettingsViewModel(preferencesRepository) as T
            }
            else -> throw IllegalArgumentException("ViewModel desconocido: ${modelClass.name}")
        }
    }
}
```

5. Compila el proyecto.

**Resultado Esperado:** Compilación exitosa. Los tres ViewModels están listos con sus respectivos `StateFlow` para ser consumidos por la UI.

**Verificación:**
- `BUILD SUCCESSFUL`
- `WeatherViewModel` expone `uiState: StateFlow<WeatherUiState>` y `temperatureUnit: StateFlow<String>`
- `FavoritesViewModel` expone `favorites: StateFlow<List<FavoriteCity>>`
- `SettingsViewModel` expone `uiState: StateFlow<SettingsUiState>`

---

### Paso 7: Construir la Interfaz de Usuario con Jetpack Compose

**Objetivo:** Implementar las cuatro pantallas principales (Home, Search, Favorites, Settings), la navegación con Navigation Compose y la barra de navegación inferior con Material 3.

**Instrucciones:**

1. Crea el archivo `presentation/navigation/AppNavigation.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.navigation

import kotlinx.serialization.Serializable

// Rutas typesafe para Navigation Compose
@Serializable data object HomeRoute
@Serializable data object SearchRoute
@Serializable data object FavoritesRoute
@Serializable data object SettingsRoute
```

> **Nota:** Si la serialización de Kotlin no está disponible para las rutas typesafe con tu versión de Navigation Compose, usa rutas String como alternativa. En este caso usaremos String routes para máxima compatibilidad.

Reemplaza el contenido de `AppNavigation.kt` con la versión basada en String routes:

```kotlin
package com.curso.android.avanzado.practica7.presentation.navigation

import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.Home
import androidx.compose.material.icons.filled.Search
import androidx.compose.material.icons.filled.Settings
import androidx.compose.ui.graphics.vector.ImageVector

sealed class Screen(val route: String, val title: String, val icon: ImageVector) {
    data object Home : Screen("home", "Inicio", Icons.Default.Home)
    data object Search : Screen("search", "Buscar", Icons.Default.Search)
    data object Favorites : Screen("favorites", "Favoritos", Icons.Default.Favorite)
    data object Settings : Screen("settings", "Ajustes", Icons.Default.Settings)
}

val bottomNavItems = listOf(
    Screen.Home,
    Screen.Search,
    Screen.Favorites,
    Screen.Settings
)
```

2. Crea el archivo `presentation/screens/HomeScreen.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.screens

import android.Manifest
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.MyLocation
import androidx.compose.material3.Button
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.curso.android.avanzado.practica7.data.datastore.UserPreferencesRepository
import com.curso.android.avanzado.practica7.domain.model.HourlyForecast
import com.curso.android.avanzado.practica7.domain.model.WeatherUiState
import com.curso.android.avanzado.practica7.presentation.viewmodel.WeatherViewModel

@Composable
fun HomeScreen(
    weatherViewModel: WeatherViewModel,
    modifier: Modifier = Modifier
) {
    val uiState by weatherViewModel.uiState.collectAsStateWithLifecycle()
    val tempUnit by weatherViewModel.temperatureUnit.collectAsStateWithLifecycle()

    val locationPermissionLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        val fineGranted = permissions[Manifest.permission.ACCESS_FINE_LOCATION] ?: false
        val coarseGranted = permissions[Manifest.permission.ACCESS_COARSE_LOCATION] ?: false
        if (fineGranted || coarseGranted) {
            weatherViewModel.fetchWeatherForCurrentLocation()
        }
    }

    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        // Botón de ubicación
        Button(
            onClick = {
                locationPermissionLauncher.launch(
                    arrayOf(
                        Manifest.permission.ACCESS_FINE_LOCATION,
                        Manifest.permission.ACCESS_COARSE_LOCATION
                    )
                )
            },
            modifier = Modifier.fillMaxWidth()
        ) {
            Icon(
                imageVector = Icons.Default.MyLocation,
                contentDescription = null,
                modifier = Modifier.size(20.dp)
            )
            Spacer(modifier = Modifier.size(8.dp))
            Text("Usar mi ubicación")
        }

        Spacer(modifier = Modifier.height(16.dp))

        // Contenido según estado
        when (val state = uiState) {
            is WeatherUiState.Loading -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
            is WeatherUiState.Empty -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Text(
                        text = "Presiona el botón para obtener el clima\nde tu ubicación actual",
                        textAlign = TextAlign.Center,
                        style = MaterialTheme.typography.bodyLarge,
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
            is WeatherUiState.Success -> {
                val info = state.weatherInfo
                val isCelsius = tempUnit == UserPreferencesRepository.UNIT_CELSIUS

                if (state.isStale) {
                    Card(
                        colors = CardDefaults.cardColors(
                            containerColor = MaterialTheme.colorScheme.tertiaryContainer
                        ),
                        modifier = Modifier.fillMaxWidth()
                    ) {
                        Text(
                            text = "⚠️ Datos del caché (sin conexión)",
                            modifier = Modifier.padding(12.dp),
                            style = MaterialTheme.typography.bodySmall
                        )
                    }
                    Spacer(modifier = Modifier.height(8.dp))
                }

                Text(
                    text = info.cityName,
                    style = MaterialTheme.typography.headlineMedium,
                    fontWeight = FontWeight.Bold
                )
                Text(
                    text = "📍 ${String.format("%.4f", info.latitude)}, ${String.format("%.4f", info.longitude)}",
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )

                Spacer(modifier = Modifier.height(16.dp))

                // Temperatura actual (primera hora)
                val currentForecast = info.hourlyForecasts.firstOrNull()
                if (currentForecast != null) {
                    CurrentWeatherCard(currentForecast, isCelsius)
                }

                Spacer(modifier = Modifier.height(16.dp))

                Text(
                    text = "Pronóstico por hora",
                    style = MaterialTheme.typography.titleMedium,
                    fontWeight = FontWeight.SemiBold
                )

                Spacer(modifier = Modifier.height(8.dp))

                LazyColumn {
                    items(info.hourlyForecasts) { forecast ->
                        HourlyForecastItem(forecast, isCelsius)
                    }
                }
            }
            is WeatherUiState.Error -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Column(horizontalAlignment = Alignment.CenterHorizontally) {
                        Text(
                            text = "❌ Error",
                            style = MaterialTheme.typography.headlineSmall,
                            color = MaterialTheme.colorScheme.error
                        )
                        Spacer(modifier = Modifier.height(8.dp))
                        Text(
                            text = state.message,
                            textAlign = TextAlign.Center,
                            style = MaterialTheme.typography.bodyMedium
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                        Button(onClick = {
                            weatherViewModel.fetchWeatherForCurrentLocation()
                        }) {
                            Text("Reintentar")
                        }
                    }
                }
            }
        }
    }
}

@Composable
private fun CurrentWeatherCard(forecast: HourlyForecast, isCelsius: Boolean) {
    Card(
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer
        ),
        modifier = Modifier.fillMaxWidth()
    ) {
        Column(
            modifier = Modifier.padding(20.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(
                text = forecast.weatherEmoji,
                style = MaterialTheme.typography.displayLarge
            )
            Spacer(modifier = Modifier.height(8.dp))
            val temp = if (isCelsius) forecast.temperatureCelsius
                       else forecast.temperatureFahrenheit
            val unit = if (isCelsius) "°C" else "°F"
            Text(
                text = "${String.format("%.1f", temp)}$unit",
                style = MaterialTheme.typography.displaySmall,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = forecast.weatherDescription,
                style = MaterialTheme.typography.titleMedium
            )
        }
    }
}

@Composable
private fun HourlyForecastItem(forecast: HourlyForecast, isCelsius: Boolean) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 2.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp, vertical = 10.dp),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            // Hora (extraer solo HH:mm del formato ISO)
            val timeDisplay = forecast.time.substringAfter("T", forecast.time)
            Text(
                text = timeDisplay,
                style = MaterialTheme.typography.bodyMedium,
                modifier = Modifier.weight(1f)
            )
            Text(
                text = forecast.weatherEmoji,
                style = MaterialTheme.typography.titleMedium
            )
            val temp = if (isCelsius) forecast.temperatureCelsius
                       else forecast.temperatureFahrenheit
            val unit = if (isCelsius) "°C" else "°F"
            Text(
                text = "${String.format("%.1f", temp)}$unit",
                style = MaterialTheme.typography.bodyLarge,
                fontWeight = FontWeight.SemiBold,
                modifier = Modifier.weight(1f),
                textAlign = TextAlign.End
            )
        }
    }
}
```

3. Crea el archivo `presentation/screens/SearchScreen.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.screens

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Search
import androidx.compose.material3.Card
import androidx.compose.material3.Icon
import androidx.compose.material3.ListItem
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.curso.android.avanzado.practica7.presentation.viewmodel.WeatherViewModel

// Lista predefinida de ciudades para búsqueda
// (en una app real se usaría una API de geocoding)
data class CitySearchResult(
    val name: String,
    val latitude: Double,
    val longitude: Double,
    val country: String
)

private val predefinedCities = listOf(
    CitySearchResult("Ciudad de México", 19.4326, -99.1332, "México"),
    CitySearchResult("Guadalajara", 20.6597, -103.3496, "México"),
    CitySearchResult("Monterrey", 25.6866, -100.3161, "México"),
    CitySearchResult("Buenos Aires", -34.6037, -58.3816, "Argentina"),
    CitySearchResult("Bogotá", 4.7110, -74.0721, "Colombia"),
    CitySearchResult("Lima", -12.0464, -77.0428, "Perú"),
    CitySearchResult("Santiago", -33.4489, -70.6693, "Chile"),
    CitySearchResult("Madrid", 40.4168, -3.7038, "España"),
    CitySearchResult("Nueva York", 40.7128, -74.0060, "EE.UU."),
    CitySearchResult("Tokio", 35.6762, 139.6503, "Japón"),
    CitySearchResult("Londres", 51.5074, -0.1278, "Reino Unido"),
    CitySearchResult("París", 48.8566, 2.3522, "Francia"),
    CitySearchResult("Cancún", 21.1619, -86.8515, "México"),
    CitySearchResult("Medellín", 6.2442, -75.5812, "Colombia"),
    CitySearchResult("Quito", -0.1807, -78.4678, "Ecuador")
)

@Composable
fun SearchScreen(
    weatherViewModel: WeatherViewModel,
    onCitySelected: () -> Unit,
    modifier: Modifier = Modifier
) {
    var searchQuery by remember { mutableStateOf("") }

    val filteredCities = remember(searchQuery) {
        if (searchQuery.isBlank()) predefinedCities
        else predefinedCities.filter {
            it.name.contains(searchQuery, ignoreCase = true) ||
            it.country.contains(searchQuery, ignoreCase = true)
        }
    }

    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        OutlinedTextField(
            value = searchQuery,
            onValueChange = { searchQuery = it },
            label = { Text("Buscar ciudad") },
            leadingIcon = {
                Icon(Icons.Default.Search, contentDescription = null)
            },
            modifier = Modifier.fillMaxWidth(),
            singleLine = true
        )

        Spacer(modifier = Modifier.height(12.dp))

        LazyColumn {
            items(filteredCities) { city ->
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(vertical = 2.dp)
                        .clickable {
                            weatherViewModel.fetchWeather(
                                latitude = city.latitude,
                                longitude = city.longitude,
                                cityName = city.name
                            )
                            onCitySelected()
                        }
                ) {
                    ListItem(
                        headlineContent = { Text(city.name) },
                        supportingContent = {
                            Text("${city.country} • ${String.format("%.2f", city.latitude)}, ${String.format("%.2f", city.longitude)}")
                        }
                    )
                }
            }
        }
    }
}
```

4. Crea el archivo `presentation/screens/FavoritesScreen.kt`:

```kotlin
package com.curso.android.avanzado.practica7.presentation.screens

import androidx.compose.animation.animateColorAsState
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Card
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.FloatingActionButton
import androidx.compose.material3.Icon
import androidx.compose.material3.ListItem
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Scaffold
import androidx.compose.material3.SwipeToDismissBox
import androidx.compose.material3.SwipeToDismissBoxValue
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.material3.rememberSwipeToDismissBoxState
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.curso.android.avanzado.practica7.domain.model.FavoriteCity
import com.curso.android.avanzado.practica7.presentation.viewmodel.FavoritesViewModel
import com.curso.android.avanzado.practica7.presentation.viewmodel.WeatherViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun FavoritesScreen(
    favoritesViewModel: FavoritesViewModel,
    weatherViewModel: WeatherViewModel,
    onCitySelected: () -> Unit,
    modifier: Modifier = Modifier
) {
    val favorites by favoritesViewModel.favorites.collectAsStateWithLifecycle()
    var showAddDialog by remember { mutableStateOf(false) }

    Scaffold(
        floatingActionButton = {
            FloatingActionButton(onClick = { showAddDialog = true }) {
                Icon(Icons.Default.Add, contentDescription = "Agregar favorito")
            }
        },
        modifier = modifier
    ) { innerPadding ->
        if (favorites.isEmpty()) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(innerPadding),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    text = "No tienes ciudades favoritas.\nPresiona + para agregar una.",
                    textAlign = TextAlign.Center,
                    style = MaterialTheme.typography.bodyLarge,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        } else {
            LazyColumn(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(innerPadding)
                    .padding(horizontal = 16.dp)
            ) {
                items(
                    items = favorites,
                    key = { it.id }
                ) { city ->
                    val dismissState = rememberSwipeToDismissBoxState(
                        confirmValueChange = { dismissValue ->
                            if (dismissValue == SwipeToDismissBoxValue.EndToStart) {
                                favoritesViewModel.removeFavorite(city)
                                true
                            } else false
                        }
                    )

                    SwipeToDismissBox(
                        state = dismissState,
                        backgroundContent = {
                            val color by animateColorAsState(
                                targetValue = MaterialTheme.colorScheme.errorContainer,
                                label = "dismiss_bg"
                            )
                            Box(
                                modifier = Modifier
                                    .fillMaxSize()
                                    .background(color)
                                    .padding(horizontal = 20.dp),
                                contentAlignment = Alignment.CenterEnd
                            ) {
                                Icon(
                                    Icons.Default.Delete,
                                    contentDescription = "Eliminar",
                                    tint = MaterialTheme.colorScheme.onErrorContainer
                                )
                            }
                        },
                        enableDismissFromStartToEnd = false
                    ) {
                        Card(
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(vertical = 2.dp)
                                .clickable {
                                    weatherViewModel.fetchWeather(
                                        city.latitude,
                                        city.longitude,
                                        city.name
                                    )
                                    onCitySelected()
                                }
                        ) {
                            ListItem(
                                headlineContent = { Text(city.name) },
                                supportingContent = {
                                    Text(
                                        "${String.format("%.4f", city.latitude)}, ${String.format("%.4f", city.longitude)}"
                                    )
                                }
                            )
                        }
                    }
                }
            }
        }
    }

    if (showAddDialog) {
        AddFavoriteDialog(
            onDismiss = { showAddDialog = false },
            onAdd = { name, lat, lng ->
                favoritesViewModel.addFavorite(
                    FavoriteCity(name = name, latitude = lat, longitude = lng)
                )
                showAddDialog = false
            }
        )
    }
}

@Composable
private fun AddFavoriteDialog(
    onDismiss: () -> Unit,
    onAdd: (String, Double, Double) -> Unit
) {
    var name by remember { mutableStateOf("") }
    var latitude by remember { mutableStateOf("") }
    var longitude by remember { mutableStateOf("") }

    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Agregar ciudad favorita") },
        text = {
            Column {
                OutlinedTextField(
                    value = name,
                    onValueChange = { name = it },
                    label = { Text("Nombre de la ciudad") },
                    singleLine = true,
                    modifier = Modifier.fillMaxWidth()
                )
                Spacer(modifier = Modifier.height(8.dp))
                OutlinedTextField(
                    value = latitude,
                    onValueChange = { latitude = it },
