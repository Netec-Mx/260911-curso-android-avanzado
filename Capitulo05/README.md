---LAB_START---
LAB_ID: 05-00-01
---MARKDOWN---
# Práctica 5 — Persistencia Avanzada con Room, DataStore y Estrategia Offline-First

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 216 minutos (≈ 3 h 36 min) |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |

---

## 2. Descripción General

En esta práctica culminante del curso, extenderás la aplicación construida en las Prácticas 3 y 4 para dotarla de persistencia local robusta y capacidad offline-first. Integrarás Room 2.8.4 como caché local con entidades relacionadas, DAOs reactivos y migraciones de esquema; configurarás DataStore Preferences para almacenar preferencias del usuario; implementarás paginación con Paging 3 conectado a Room; y construirás un repositorio híbrido que sirve datos locales inmediatamente mientras sincroniza con la API remota en segundo plano. El resultado será una aplicación Android production-ready con arquitectura MVVM + Clean Architecture + Hilt + Compose + Retrofit + Room + DataStore.

---

## 3. Objetivos de Aprendizaje

Al finalizar esta práctica, serás capaz de:

- [ ] Implementar una base de datos Room completa con entidades relacionadas (@ForeignKey, @Relation), DAOs con operaciones CRUD suspendidas y consultas reactivas mediante Flow, incluyendo una migración de esquema
- [ ] Configurar Paging 3 integrado con Room para paginar listas grandes de forma eficiente, recolectando `PagingData<T>` en Compose con `collectAsLazyPagingItems()`
- [ ] Almacenar y exponer preferencias del usuario mediante DataStore Preferences como `Flow<UserPreferences>`, comprendiendo las ventajas frente a Proto DataStore
- [ ] Diseñar un repositorio híbrido offline-first que emite datos de Room inmediatamente, lanza sincronización con la API en background y notifica a la UI automáticamente vía Flow reactivo
- [ ] Integrar Room, DataStore, Retrofit, Hilt y Compose en una arquitectura MVVM limpia con inyección de dependencias completa

---

## 4. Prerrequisitos

### Conocimientos Requeridos

| Requisito | Origen |
|---|---|
| Arquitectura MVVM con Hilt completamente configurada | Práctica 3 |
| Integración de Retrofit 3 con al menos un endpoint GET funcionando | Práctica 4 |
| Corrutinas y Flow de Kotlin para observar cambios reactivos | Capítulo 1 |
| Jetpack Compose: LazyColumn, StateFlow, collectAsState | Capítulo 2 |
| Navegación Compose y gestión de estado en ViewModels | Capítulo 3 |

### Acceso Requerido

- Proyecto de la Práctica 4 funcional en `~/AndroidStudioProjects/AvanzadoKotlin/Practica4_Red/`
- Conexión a Internet para descarga de dependencias Gradle y acceso a la API pública
- Android Studio Quail 3 (2026.1.3 Patch 1) con SDK API 36/37 instalados
- AVD configurado con API 35 o superior (o dispositivo físico con Android 11+)

---

## 5. Entorno de Laboratorio

### Hardware Mínimo

| Componente | Requisito |
|---|---|
| Procesador | Intel Core i7 8ª gen+ / AMD Ryzen 7+ / Apple M1+ |
| RAM | 16 GB mínimo (32 GB recomendado) |
| Disco | 60 GB libres en SSD |
| Virtualización | VT-x / AMD-V / Apple Hypervisor habilitado en BIOS |

### Software

| Herramienta | Versión |
|---|---|
| Android Studio | Quail 3 — 2026.1.3 Patch 1 |
| Kotlin | 2.2.10 |
| AGP | 9.3.2 |
| Gradle Wrapper | 8.14.1 |
| KSP | 2.2.10-1.0.31 |
| Compose BOM | 2026.02.01 |
| Room | 2.8.4 |
| Paging | 3.3.6 |
| DataStore Preferences | 1.1.7 |
| Retrofit | 3.0.0 |
| OkHttp BOM | 5.5.0 |
| Hilt | 2.56.2 |
| Play Services Location | 21.4.0 |

### Preparación Inicial

Copia el proyecto de la Práctica 4 como base para la Práctica 5:

```bash
cd ~/AndroidStudioProjects/AvanzadoKotlin/
cp -r Practica4_Red Practica5_Persistencia
cd Practica5_Persistencia
```

Abre el proyecto copiado en Android Studio y verifica que compila correctamente antes de continuar.

---

## 6. Instrucciones Paso a Paso

### Paso 1 — Configurar Dependencias de Room, Paging y DataStore

**Objetivo:** Añadir todas las dependencias necesarias al catálogo de versiones y a los scripts de build para habilitar Room 2.8.4, Paging 3, DataStore Preferences y sus integraciones con Hilt.

**Instrucciones:**

1. Abre el archivo `gradle/libs.versions.toml` y añade las versiones y bibliotecas necesarias. Conserva las versiones existentes de Retrofit, OkHttp, Hilt, Compose, etc., y agrega las nuevas:

```toml
[versions]
# === Versiones existentes (no modificar) ===
kotlin = "2.2.10"
agp = "9.3.2"
ksp = "2.2.10-1.0.31"
composeBom = "2026.02.01"
activityCompose = "1.13.0"
hilt = "2.56.2"
hiltNavigationCompose = "1.2.0"
retrofit = "3.0.0"
okhttpBom = "5.5.0"
coreKtx = "1.19.0"
lifecycleRuntimeKtx = "2.6.1"
playServicesLocation = "21.4.0"
junit = "4.13.2"
androidxJunit = "1.3.0"
espressoCore = "3.7.0"

# === Nuevas versiones para Práctica 5 ===
room = "2.8.4"
paging = "3.3.6"
pagingCompose = "3.3.6"
datastore = "1.1.7"

[libraries]
# === Existentes (mantener) ===
core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycleRuntimeKtx" }
activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hiltNavigationCompose" }
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-kotlinx = { group = "com.squareup.retrofit2", name = "converter-kotlinx-serialization", version.ref = "retrofit" }
okhttp-bom = { group = "com.squareup.okhttp3", name = "okhttp-bom", version.ref = "okhttpBom" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor" }
play-services-location = { group = "com.google.android.gms", name = "play-services-location", version.ref = "playServicesLocation" }
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidxJunit" }
espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }

# === Room ===
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-paging = { group = "androidx.room", name = "room-paging", version.ref = "room" }

# === Paging 3 ===
paging-runtime = { group = "androidx.paging", name = "paging-runtime-ktx", version.ref = "paging" }
paging-compose = { group = "androidx.paging", name = "paging-compose", version.ref = "pagingCompose" }

# === DataStore ===
datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

2. Abre `app/build.gradle.kts` y verifica que los plugins estén correctos. Luego añade las dependencias nuevas en el bloque `dependencies`:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)
    alias(libs.plugins.hilt)
}

android {
    namespace = "com.cursoadv.android.persistencia"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.cursoadv.android.persistencia"
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
    implementation(libs.core.ktx)
    implementation(libs.lifecycle.runtime.ktx)

    // Compose
    implementation(platform(libs.compose.bom))
    implementation(libs.activity.compose)
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.graphics)
    implementation(libs.compose.ui.tooling.preview)
    implementation(libs.compose.material3)
    debugImplementation(libs.compose.ui.tooling)

    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    // Retrofit + OkHttp
    implementation(libs.retrofit)
    implementation(libs.retrofit.converter.kotlinx)
    implementation(platform(libs.okhttp.bom))
    implementation(libs.okhttp)
    implementation(libs.okhttp.logging)

    // Room
    implementation(libs.room.runtime)
    implementation(libs.room.ktx)
    implementation(libs.room.paging)
    ksp(libs.room.compiler)

    // Paging 3
    implementation(libs.paging.runtime)
    implementation(libs.paging.compose)

    // DataStore
    implementation(libs.datastore.preferences)

    // Location
    implementation(libs.play.services.location)

    // Testing
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.espresso.core)
}
```

3. Sincroniza Gradle haciendo clic en **Sync Now** o ejecutando:

```bash
./gradlew dependencies --configuration releaseRuntimeClasspath | head -80
```

**Resultado Esperado:**

La sincronización completa sin errores. En la salida de dependencias deberás ver `androidx.room:room-runtime:2.8.4`, `androidx.paging:paging-runtime-ktx:3.3.6` y `androidx.datastore:datastore-preferences:1.1.7` resueltos.

**Verificación:**

```bash
./gradlew assembleDebug 2>&1 | tail -5
```

Debe mostrar `BUILD SUCCESSFUL`.

---

### Paso 2 — Definir Entidades Room con Relaciones

**Objetivo:** Crear las entidades de la base de datos Room que modelan el dominio de la aplicación, incluyendo relaciones uno a muchos con @ForeignKey e índices para optimización de consultas.

**Instrucciones:**

1. Crea el paquete `data.local.entity` dentro de `com.cursoadv.android.persistencia`. Crea la entidad principal `PostEntity.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.local.entity

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.Index
import androidx.room.PrimaryKey

@Entity(
    tableName = "posts",
    indices = [Index(value = ["user_id"])]
)
data class PostEntity(
    @PrimaryKey
    val id: Int,

    @ColumnInfo(name = "user_id")
    val userId: Int,

    val title: String,

    val body: String,

    @ColumnInfo(name = "cached_at")
    val cachedAt: Long = System.currentTimeMillis()
)
```

2. Crea la entidad `CommentEntity.kt` con una relación de clave foránea hacia `PostEntity`:

```kotlin
package com.cursoadv.android.persistencia.data.local.entity

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.ForeignKey
import androidx.room.Index
import androidx.room.PrimaryKey

@Entity(
    tableName = "comments",
    foreignKeys = [
        ForeignKey(
            entity = PostEntity::class,
            parentColumns = ["id"],
            childColumns = ["post_id"],
            onDelete = ForeignKey.CASCADE
        )
    ],
    indices = [Index(value = ["post_id"])]
)
data class CommentEntity(
    @PrimaryKey
    val id: Int,

    @ColumnInfo(name = "post_id")
    val postId: Int,

    val name: String,

    val email: String,

    val body: String
)
```

3. Crea la clase de relación `PostWithComments.kt` usando `@Relation` y `@Embedded`:

```kotlin
package com.cursoadv.android.persistencia.data.local.entity

import androidx.room.Embedded
import androidx.room.Relation

data class PostWithComments(
    @Embedded
    val post: PostEntity,

    @Relation(
        parentColumn = "id",
        entityColumn = "post_id"
    )
    val comments: List<CommentEntity>
)
```

4. Crea la entidad `UserPreferenceEntity.kt` (opcional, para demostrar @Embedded en otro contexto):

```kotlin
package com.cursoadv.android.persistencia.data.local.entity

import androidx.room.Embedded

data class PostSummary(
    val id: Int,
    val title: String,
    @ColumnInfo(name = "user_id")
    val userId: Int
)
```

> **Nota:** `PostSummary` no es una @Entity sino una clase POJO que Room usará para consultas parciales con @Query.

**Resultado Esperado:**

Cuatro archivos Kotlin creados en `data.local.entity` sin errores de compilación. Las anotaciones de Room están correctamente aplicadas con claves foráneas, índices y relaciones.

**Verificación:**

Compila el proyecto: `./gradlew compileDebugKotlin`. No deben aparecer errores relacionados con las entidades.

---

### Paso 3 — Crear DAOs con Operaciones CRUD y Consultas Reactivas

**Objetivo:** Implementar los DAOs (Data Access Objects) de Room con operaciones suspend para escritura, Flow para lectura reactiva, consultas @Transaction para relaciones y PagingSource para paginación.

**Instrucciones:**

1. Crea el paquete `data.local.dao`. Crea `PostDao.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.local.dao

import androidx.paging.PagingSource
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Transaction
import androidx.room.Update
import com.cursoadv.android.persistencia.data.local.entity.PostEntity
import com.cursoadv.android.persistencia.data.local.entity.PostSummary
import com.cursoadv.android.persistencia.data.local.entity.PostWithComments
import kotlinx.coroutines.flow.Flow

@Dao
interface PostDao {

    // === Operaciones de escritura (suspend) ===

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(posts: List<PostEntity>)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(post: PostEntity)

    @Update
    suspend fun update(post: PostEntity)

    @Query("DELETE FROM posts WHERE id = :postId")
    suspend fun deleteById(postId: Int)

    @Query("DELETE FROM posts")
    suspend fun deleteAll()

    // === Consultas reactivas (Flow) ===

    @Query("SELECT * FROM posts ORDER BY id ASC")
    fun observeAll(): Flow<List<PostEntity>>

    @Query("SELECT * FROM posts WHERE id = :postId")
    fun observeById(postId: Int): Flow<PostEntity?>

    @Query("SELECT * FROM posts WHERE user_id = :userId ORDER BY id ASC")
    fun observeByUserId(userId: Int): Flow<List<PostEntity>>

    // === Consulta parcial ===

    @Query("SELECT id, title, user_id FROM posts ORDER BY id ASC")
    fun observeSummaries(): Flow<List<PostSummary>>

    // === Búsqueda de texto ===

    @Query("SELECT * FROM posts WHERE title LIKE '%' || :query || '%' OR body LIKE '%' || :query || '%' ORDER BY id ASC")
    fun searchPosts(query: String): Flow<List<PostEntity>>

    // === Relación con @Transaction ===

    @Transaction
    @Query("SELECT * FROM posts WHERE id = :postId")
    fun observePostWithComments(postId: Int): Flow<PostWithComments?>

    @Transaction
    @Query("SELECT * FROM posts ORDER BY id ASC")
    fun observeAllPostsWithComments(): Flow<List<PostWithComments>>

    // === Paginación con PagingSource ===

    @Query("SELECT * FROM posts ORDER BY id ASC")
    fun pagingSource(): PagingSource<Int, PostEntity>

    // === Consulta de staleness ===

    @Query("SELECT MIN(cached_at) FROM posts")
    suspend fun getOldestCacheTimestamp(): Long?

    @Query("SELECT COUNT(*) FROM posts")
    suspend fun getCount(): Int
}
```

2. Crea `CommentDao.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.local.dao

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import com.cursoadv.android.persistencia.data.local.entity.CommentEntity
import kotlinx.coroutines.flow.Flow

@Dao
interface CommentDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(comments: List<CommentEntity>)

    @Query("DELETE FROM comments WHERE post_id = :postId")
    suspend fun deleteByPostId(postId: Int)

    @Query("DELETE FROM comments")
    suspend fun deleteAll()

    @Query("SELECT * FROM comments WHERE post_id = :postId ORDER BY id ASC")
    fun observeByPostId(postId: Int): Flow<List<CommentEntity>>

    @Query("SELECT COUNT(*) FROM comments WHERE post_id = :postId")
    suspend fun getCountByPostId(postId: Int): Int
}
```

**Resultado Esperado:**

Dos interfaces DAO con anotaciones Room correctas. Las operaciones de escritura son `suspend`, las de lectura retornan `Flow<T>`, y la paginación retorna `PagingSource<Int, PostEntity>`.

**Verificación:**

El proyecto compila sin errores. Room generará las implementaciones de los DAOs durante la compilación con KSP.

---

### Paso 4 — Configurar la Base de Datos Room con Migración

**Objetivo:** Crear la clase @Database de Room con su Builder, implementar una primera migración de esquema y proveer la instancia mediante Hilt.

**Instrucciones:**

1. Crea el paquete `data.local`. Crea `AppDatabase.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.local

import androidx.room.Database
import androidx.room.RoomDatabase
import com.cursoadv.android.persistencia.data.local.dao.CommentDao
import com.cursoadv.android.persistencia.data.local.dao.PostDao
import com.cursoadv.android.persistencia.data.local.entity.CommentEntity
import com.cursoadv.android.persistencia.data.local.entity.PostEntity

@Database(
    entities = [
        PostEntity::class,
        CommentEntity::class
    ],
    version = 2,
    exportSchema = true
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun postDao(): PostDao
    abstract fun commentDao(): CommentDao
}
```

2. Crea `DatabaseMigrations.kt` en el mismo paquete para definir la migración de versión 1 a 2:

```kotlin
package com.cursoadv.android.persistencia.data.local

import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase

object DatabaseMigrations {

    val MIGRATION_1_2 = object : Migration(1, 2) {
        override fun migrate(db: SupportSQLiteDatabase) {
            // Migración: agregar columna cached_at a la tabla posts
            db.execSQL(
                "ALTER TABLE posts ADD COLUMN cached_at INTEGER NOT NULL DEFAULT 0"
            )
        }
    }

    val ALL_MIGRATIONS = arrayOf(MIGRATION_1_2)
}
```

> **Nota importante:** En esta práctica iniciaremos directamente con la versión 2 de la base de datos (que ya incluye `cached_at`). La migración está definida para el caso en que un usuario ya tuviera instalada la versión 1. Si estás ejecutando la app por primera vez, Room creará la base de datos directamente en versión 2.

3. Crea (o actualiza) el módulo Hilt para proveer la base de datos. Crea `di/DatabaseModule.kt`:

```kotlin
package com.cursoadv.android.persistencia.di

import android.content.Context
import androidx.room.Room
import com.cursoadv.android.persistencia.data.local.AppDatabase
import com.cursoadv.android.persistencia.data.local.DatabaseMigrations
import com.cursoadv.android.persistencia.data.local.dao.CommentDao
import com.cursoadv.android.persistencia.data.local.dao.PostDao
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    @Singleton
    fun provideDatabase(
        @ApplicationContext context: Context
    ): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database"
        )
            .addMigrations(*DatabaseMigrations.ALL_MIGRATIONS)
            .build()
    }

    @Provides
    fun providePostDao(database: AppDatabase): PostDao {
        return database.postDao()
    }

    @Provides
    fun provideCommentDao(database: AppDatabase): CommentDao {
        return database.commentDao()
    }
}
```

4. Habilita la exportación de esquemas Room añadiendo la configuración de KSP en `app/build.gradle.kts` dentro del bloque `android`:

```kotlin
ksp {
    arg("room.schemaLocation", "$projectDir/schemas")
    arg("room.incremental", "true")
    arg("room.generateKotlin", "true")
}
```

5. Añade el directorio `schemas` a `.gitignore` si lo deseas, o consérvalo en el repositorio para referencia de migraciones.

**Resultado Esperado:**

La base de datos Room está configurada con dos entidades, migración de esquema 1→2, y provisión completa mediante Hilt como Singleton. KSP generará las implementaciones de `AppDatabase_Impl`, `PostDao_Impl` y `CommentDao_Impl`.

**Verificación:**

```bash
./gradlew kspDebugKotlin 2>&1 | tail -10
```

Debe completar sin errores. Verifica que se genera el directorio `app/schemas/` con el archivo JSON del esquema de la base de datos.

---

### Paso 5 — Implementar DataStore Preferences

**Objetivo:** Configurar DataStore Preferences para almacenar preferencias del usuario (tema oscuro, último filtro aplicado) y exponerlas como Flow reactivo, proveyendo la instancia mediante Hilt.

**Instrucciones:**

1. Crea el paquete `data.datastore`. Crea `UserPreferencesKeys.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.datastore

import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey

object UserPreferencesKeys {
    val DARK_THEME = booleanPreferencesKey("dark_theme")
    val LANGUAGE = stringPreferencesKey("language")
    val LAST_FILTER_USER_ID = intPreferencesKey("last_filter_user_id")
    val CACHE_DURATION_MINUTES = intPreferencesKey("cache_duration_minutes")
}
```

2. Crea el modelo de datos `UserPreferences.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.datastore

data class UserPreferences(
    val darkTheme: Boolean = false,
    val language: String = "es",
    val lastFilterUserId: Int = -1,
    val cacheDurationMinutes: Int = 30
)
```

3. Crea `UserPreferencesRepository.kt` que encapsula la lectura y escritura:

```kotlin
package com.cursoadv.android.persistencia.data.datastore

import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.edit
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import java.io.IOException
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class UserPreferencesRepository @Inject constructor(
    private val dataStore: DataStore<Preferences>
) {

    val userPreferencesFlow: Flow<UserPreferences> = dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(androidx.datastore.preferences.core.emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            UserPreferences(
                darkTheme = preferences[UserPreferencesKeys.DARK_THEME] ?: false,
                language = preferences[UserPreferencesKeys.LANGUAGE] ?: "es",
                lastFilterUserId = preferences[UserPreferencesKeys.LAST_FILTER_USER_ID] ?: -1,
                cacheDurationMinutes = preferences[UserPreferencesKeys.CACHE_DURATION_MINUTES] ?: 30
            )
        }

    suspend fun updateDarkTheme(enabled: Boolean) {
        dataStore.edit { preferences ->
            preferences[UserPreferencesKeys.DARK_THEME] = enabled
        }
    }

    suspend fun updateLanguage(language: String) {
        dataStore.edit { preferences ->
            preferences[UserPreferencesKeys.LANGUAGE] = language
        }
    }

    suspend fun updateLastFilterUserId(userId: Int) {
        dataStore.edit { preferences ->
            preferences[UserPreferencesKeys.LAST_FILTER_USER_ID] = userId
        }
    }

    suspend fun updateCacheDuration(minutes: Int) {
        dataStore.edit { preferences ->
            preferences[UserPreferencesKeys.CACHE_DURATION_MINUTES] = minutes
        }
    }
}
```

4. Crea el módulo Hilt para proveer DataStore. Crea `di/DataStoreModule.kt`:

```kotlin
package com.cursoadv.android.persistencia.di

import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.preferencesDataStore
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

// Extensión de nivel superior: crea una única instancia de DataStore
private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(
    name = "user_preferences"
)

@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {

    @Provides
    @Singleton
    fun provideDataStore(
        @ApplicationContext context: Context
    ): DataStore<Preferences> {
        return context.dataStore
    }
}
```

> **Comparación con Proto DataStore:** Preferences DataStore usa claves tipadas (`booleanPreferencesKey`, `stringPreferencesKey`) pero no valida la estructura completa de los datos. Proto DataStore usa Protocol Buffers para definir un esquema fuertemente tipado, lo que garantiza consistencia de tipos en compilación. Para preferencias simples, Preferences DataStore es suficiente y más sencillo de configurar. Para datos estructurados complejos, Proto DataStore es preferible.

**Resultado Esperado:**

DataStore Preferences configurado con cuatro claves de preferencia, un repositorio que expone `Flow<UserPreferences>` y métodos suspend para escritura, todo provisto mediante Hilt.

**Verificación:**

Compila el proyecto completo: `./gradlew assembleDebug`. No deben aparecer errores.

---

### Paso 6 — Construir el Repositorio Híbrido Offline-First

**Objetivo:** Implementar el patrón offline-first que emite datos de Room inmediatamente, lanza sincronización con la API en background, actualiza Room con datos nuevos y deja que Room notifique automáticamente a la UI vía Flow reactivo.

**Instrucciones:**

1. Primero, asegúrate de tener la interfaz de API de Retrofit de la Práctica 4. Si no existe, créala en `data.remote/ApiService.kt`. Usaremos JSONPlaceholder como API de ejemplo:

```kotlin
package com.cursoadv.android.persistencia.data.remote

import com.cursoadv.android.persistencia.data.remote.dto.CommentDto
import com.cursoadv.android.persistencia.data.remote.dto.PostDto
import retrofit2.http.GET
import retrofit2.http.Path
import retrofit2.http.Query

interface ApiService {

    @GET("posts")
    suspend fun getPosts(): List<PostDto>

    @GET("posts/{id}")
    suspend fun getPostById(@Path("id") id: Int): PostDto

    @GET("posts")
    suspend fun getPostsByUserId(@Query("userId") userId: Int): List<PostDto>

    @GET("posts/{postId}/comments")
    suspend fun getCommentsByPostId(@Path("postId") postId: Int): List<CommentDto>
}
```

2. Crea los DTOs en `data.remote.dto/`:

```kotlin
// PostDto.kt
package com.cursoadv.android.persistencia.data.remote.dto

import kotlinx.serialization.Serializable

@Serializable
data class PostDto(
    val userId: Int,
    val id: Int,
    val title: String,
    val body: String
)
```

```kotlin
// CommentDto.kt
package com.cursoadv.android.persistencia.data.remote.dto

import kotlinx.serialization.Serializable

@Serializable
data class CommentDto(
    val postId: Int,
    val id: Int,
    val name: String,
    val email: String,
    val body: String
)
```

3. Crea los mappers en `data.mapper/PostMapper.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.mapper

import com.cursoadv.android.persistencia.data.local.entity.CommentEntity
import com.cursoadv.android.persistencia.data.local.entity.PostEntity
import com.cursoadv.android.persistencia.data.remote.dto.CommentDto
import com.cursoadv.android.persistencia.data.remote.dto.PostDto
import com.cursoadv.android.persistencia.domain.model.Comment
import com.cursoadv.android.persistencia.domain.model.Post

// DTO -> Entity
fun PostDto.toEntity(): PostEntity = PostEntity(
    id = id,
    userId = userId,
    title = title,
    body = body,
    cachedAt = System.currentTimeMillis()
)

fun CommentDto.toEntity(): CommentEntity = CommentEntity(
    id = id,
    postId = postId,
    name = name,
    email = email,
    body = body
)

// Entity -> Domain
fun PostEntity.toDomain(): Post = Post(
    id = id,
    userId = userId,
    title = title,
    body = body
)

fun CommentEntity.toDomain(): Comment = Comment(
    id = id,
    postId = postId,
    name = name,
    email = email,
    body = body
)

// Extensiones para listas
fun List<PostDto>.toEntities(): List<PostEntity> = map { it.toEntity() }
fun List<CommentDto>.toEntities(): List<CommentEntity> = map { it.toEntity() }
fun List<PostEntity>.toDomain(): List<Post> = map { it.toDomain() }
fun List<CommentEntity>.toDomain(): List<Comment> = map { it.toDomain() }
```

4. Crea los modelos de dominio en `domain.model/`:

```kotlin
// Post.kt
package com.cursoadv.android.persistencia.domain.model

data class Post(
    val id: Int,
    val userId: Int,
    val title: String,
    val body: String
)
```

```kotlin
// Comment.kt
package com.cursoadv.android.persistencia.domain.model

data class Comment(
    val id: Int,
    val postId: Int,
    val name: String,
    val email: String,
    val body: String
)
```

5. Crea un sealed class para representar el resultado de operaciones en `domain.util/Resource.kt`:

```kotlin
package com.cursoadv.android.persistencia.domain.util

sealed class Resource<out T> {
    data class Success<T>(val data: T) : Resource<T>()
    data class Error(val message: String, val throwable: Throwable? = null) : Resource<Nothing>()
    data object Loading : Resource<Nothing>()
}
```

6. Ahora crea el repositorio híbrido offline-first en `data.repository/PostRepository.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.repository

import com.cursoadv.android.persistencia.data.datastore.UserPreferencesRepository
import com.cursoadv.android.persistencia.data.local.dao.CommentDao
import com.cursoadv.android.persistencia.data.local.dao.PostDao
import com.cursoadv.android.persistencia.data.mapper.toDomain
import com.cursoadv.android.persistencia.data.mapper.toEntities
import com.cursoadv.android.persistencia.data.mapper.toEntity
import com.cursoadv.android.persistencia.data.remote.ApiService
import com.cursoadv.android.persistencia.domain.model.Comment
import com.cursoadv.android.persistencia.domain.model.Post
import com.cursoadv.android.persistencia.domain.util.Resource
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class PostRepository @Inject constructor(
    private val apiService: ApiService,
    private val postDao: PostDao,
    private val commentDao: CommentDao,
    private val preferencesRepository: UserPreferencesRepository
) {

    /**
     * Patrón offline-first:
     * 1. Emite Loading
     * 2. Emite datos locales inmediatamente (si existen)
     * 3. Intenta sincronizar con la API en background
     * 4. Si la API responde, actualiza Room → Room notifica automáticamente vía Flow
     * 5. Si la API falla, los datos locales ya están disponibles
     */
    fun observePosts(): Flow<Resource<List<Post>>> = flow {
        emit(Resource.Loading)

        // Paso 1: Emitir datos locales inmediatamente
        val localPosts = postDao.observeAll().first()
        if (localPosts.isNotEmpty()) {
            emit(Resource.Success(localPosts.toDomain()))
        }

        // Paso 2: Verificar si los datos están obsoletos (staleness check)
        val shouldRefresh = shouldRefreshCache()

        if (shouldRefresh) {
            try {
                // Paso 3: Obtener datos frescos de la API
                val remotePosts = apiService.getPosts()

                // Paso 4: Actualizar Room (REPLACE resuelve conflictos)
                postDao.insertAll(remotePosts.toEntities())

                // Paso 5: Emitir datos actualizados
                val updatedPosts = postDao.observeAll().first()
                emit(Resource.Success(updatedPosts.toDomain()))
            } catch (e: Exception) {
                // Si la API falla pero tenemos datos locales, emitir error parcial
                val fallbackPosts = postDao.observeAll().first()
                if (fallbackPosts.isNotEmpty()) {
                    emit(Resource.Success(fallbackPosts.toDomain()))
                }
                emit(Resource.Error(
                    message = "Error al sincronizar: ${e.localizedMessage}",
                    throwable = e
                ))
            }
        }
    }

    /**
     * Observar posts reactivamente desde Room.
     * Room emite automáticamente cuando los datos cambian.
     */
    fun observePostsReactive(): Flow<List<Post>> {
        return postDao.observeAll().map { entities ->
            entities.toDomain()
        }
    }

    /**
     * Obtener un post con sus comentarios.
     * Patrón offline-first aplicado a detalle.
     */
    fun observePostWithComments(postId: Int): Flow<Resource<Pair<Post, List<Comment>>>> = flow {
        emit(Resource.Loading)

        // Emitir datos locales primero
        val localData = postDao.observePostWithComments(postId).first()
        if (localData != null) {
            emit(Resource.Success(
                Pair(
                    localData.post.toDomain(),
                    localData.comments.toDomain()
                )
            ))
        }

        // Sincronizar comentarios desde la API
        try {
            val remoteComments = apiService.getCommentsByPostId(postId)
            commentDao.deleteByPostId(postId)
            commentDao.insertAll(remoteComments.toEntities())

            val updatedData = postDao.observePostWithComments(postId).first()
            if (updatedData != null) {
                emit(Resource.Success(
                    Pair(
                        updatedData.post.toDomain(),
                        updatedData.comments.toDomain()
                    )
                ))
            }
        } catch (e: Exception) {
            emit(Resource.Error(
                message = "Error al cargar comentarios: ${e.localizedMessage}",
                throwable = e
            ))
        }
    }

    /**
     * Búsqueda de posts por texto.
     */
    fun searchPosts(query: String): Flow<List<Post>> {
        return postDao.searchPosts(query).map { it.toDomain() }
    }

    /**
     * Forzar sincronización completa.
     */
    suspend fun forceRefresh(): Resource<Unit> {
        return try {
            val remotePosts = apiService.getPosts()
            postDao.deleteAll()
            commentDao.deleteAll()
            postDao.insertAll(remotePosts.toEntities())
            Resource.Success(Unit)
        } catch (e: Exception) {
            Resource.Error(
                message = "Error al refrescar: ${e.localizedMessage}",
                throwable = e
            )
        }
    }

    /**
     * Lógica de staleness: determina si los datos locales necesitan actualización.
     * Usa la duración de caché configurada en DataStore.
     */
    private suspend fun shouldRefreshCache(): Boolean {
        val count = postDao.getCount()
        if (count == 0) return true

        val oldestTimestamp = postDao.getOldestCacheTimestamp() ?: return true
        val preferences = preferencesRepository.userPreferencesFlow.first()
        val cacheDurationMs = preferences.cacheDurationMinutes * 60 * 1000L
        val elapsed = System.currentTimeMillis() - oldestTimestamp

        return elapsed > cacheDurationMs
    }
}
```

7. Asegúrate de que el módulo Hilt de red (heredado de la Práctica 4) provee `ApiService`. Si no existe, crea `di/NetworkModule.kt`:

```kotlin
package com.cursoadv.android.persistencia.di

import com.cursoadv.android.persistencia.data.remote.ApiService
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import kotlinx.serialization.json.Json
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.kotlinx.serialization.asConverterFactory
import java.util.concurrent.TimeUnit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideJson(): Json = Json {
        ignoreUnknownKeys = true
        coerceInputValues = true
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        val logging = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
        return OkHttpClient.Builder()
            .addInterceptor(logging)
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(
        client: OkHttpClient,
        json: Json
    ): Retrofit {
        val contentType = "application/json".toMediaType()
        return Retrofit.Builder()
            .baseUrl("https://jsonplaceholder.typicode.com/")
            .client(client)
            .addConverterFactory(json.asConverterFactory(contentType))
            .build()
    }

    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

**Resultado Esperado:**

Un repositorio híbrido completo que implementa el patrón offline-first con lógica de staleness configurable vía DataStore. El flujo es: Loading → datos locales → sincronización API → actualización Room → datos frescos.

**Verificación:**

```bash
./gradlew assembleDebug 2>&1 | tail -5
```

`BUILD SUCCESSFUL`. No hay errores de inyección de dependencias ni de compilación.

---

### Paso 7 — Implementar Paginación con Paging 3 y Room

**Objetivo:** Configurar Paging 3 integrado con Room para paginar la lista de posts de forma eficiente, usando el PagingSource generado automáticamente por Room.

**Instrucciones:**

1. Crea `data.repository/PagingPostRepository.kt`:

```kotlin
package com.cursoadv.android.persistencia.data.repository

import androidx.paging.Pager
import androidx.paging.PagingConfig
import androidx.paging.PagingData
import androidx.paging.map
import com.cursoadv.android.persistencia.data.local.dao.PostDao
import com.cursoadv.android.persistencia.data.mapper.toDomain
import com.cursoadv.android.persistencia.domain.model.Post
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class PagingPostRepository @Inject constructor(
    private val postDao: PostDao
) {

    fun getPagedPosts(): Flow<PagingData<Post>> {
        return Pager(
            config = PagingConfig(
                pageSize = 20,
                prefetchDistance = 5,
                enablePlaceholders = false,
                initialLoadSize = 40
            ),
            pagingSourceFactory = { postDao.pagingSource() }
        ).flow.map { pagingData ->
            pagingData.map { entity -> entity.toDomain() }
        }
    }
}
```

> **Nota técnica:** Room genera automáticamente un `PagingSource<Int, PostEntity>` a partir de la consulta `@Query`. Pager gestiona la carga incremental. Cuando Room detecta cambios en la tabla `posts`, invalida el PagingSource y Pager recarga automáticamente.

**Resultado Esperado:**

Un repositorio de paginación que emite `Flow<PagingData<Post>>` con carga incremental de 20 elementos por página y prefetch de 5 elementos.

**Verificación:**

El proyecto compila sin errores.

---

### Paso 8 — Crear los ViewModels

**Objetivo:** Implementar los ViewModels que consumen los repositorios y exponen el estado a la UI mediante StateFlow y PagingData.

**Instrucciones:**

1. Crea `ui.viewmodel/PostListViewModel.kt`:

```kotlin
package com.cursoadv.android.persistencia.ui.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import androidx.paging.PagingData
import androidx.paging.cachedIn
import com.cursoadv.android.persistencia.data.datastore.UserPreferences
import com.cursoadv.android.persistencia.data.datastore.UserPreferencesRepository
import com.cursoadv.android.persistencia.data.repository.PagingPostRepository
import com.cursoadv.android.persistencia.data.repository.PostRepository
import com.cursoadv.android.persistencia.domain.model.Post
import com.cursoadv.android.persistencia.domain.util.Resource
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.flatMapLatest
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class PostListViewModel @Inject constructor(
    private val postRepository: PostRepository,
    private val pagingPostRepository: PagingPostRepository,
    private val preferencesRepository: UserPreferencesRepository
) : ViewModel() {

    // === Estado de la lista de posts (offline-first) ===
    private val _postsState = MutableStateFlow<Resource<List<Post>>>(Resource.Loading)
    val postsState: StateFlow<Resource<List<Post>>> = _postsState.asStateFlow()

    // === Paginación ===
    val pagedPosts = pagingPostRepository.getPagedPosts()
        .cachedIn(viewModelScope)

    // === Preferencias del usuario ===
    val userPreferences: StateFlow<UserPreferences> =
        preferencesRepository.userPreferencesFlow
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5_000),
                initialValue = UserPreferences()
            )

    // === Búsqueda ===
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()

    @OptIn(ExperimentalCoroutinesApi::class)
    val searchResults: StateFlow<List<Post>> = _searchQuery
        .flatMapLatest { query ->
            if (query.isBlank()) {
                postRepository.observePostsReactive()
            } else {
                postRepository.searchPosts(query)
            }
        }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = emptyList()
        )

    init {
        loadPosts()
    }

    fun loadPosts() {
        viewModelScope.launch {
            postRepository.observePosts().collect { resource ->
                _postsState.value = resource
            }
        }
    }

    fun onSearchQueryChanged(query: String) {
        _searchQuery.value = query
    }

    fun forceRefresh() {
        viewModelScope.launch {
            _postsState.value = Resource.Loading
            postRepository.forceRefresh()
            loadPosts()
        }
    }

    fun toggleDarkTheme() {
        viewModelScope.launch {
            val current = userPreferences.value.darkTheme
            preferencesRepository.updateDarkTheme(!current)
        }
    }

    fun updateCacheDuration(minutes: Int) {
        viewModelScope.launch {
            preferencesRepository.updateCacheDuration(minutes)
        }
    }
}
```

2. Crea `ui.viewmodel/PostDetailViewModel.kt`:

```kotlin
package com.cursoadv.android.persistencia.ui.viewmodel

import androidx.lifecycle.SavedStateHandle
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.cursoadv.android.persistencia.domain.model.Comment
import com.cursoadv.android.persistencia.domain.model.Post
import com.cursoadv.android.persistencia.domain.util.Resource
import com.cursoadv.android.persistencia.data.repository.PostRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class PostDetailViewModel @Inject constructor(
    private val postRepository: PostRepository,
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    private val postId: Int = savedStateHandle.get<Int>("postId") ?: -1

    private val _detailState = MutableStateFlow<Resource<Pair<Post, List<Comment>>>>(Resource.Loading)
    val detailState: StateFlow<Resource<Pair<Post, List<Comment>>>> = _detailState.asStateFlow()

    init {
        if (postId != -1) {
            loadPostDetail()
        }
    }

    private fun loadPostDetail() {
        viewModelScope.launch {
            postRepository.observePostWithComments(postId).collect { resource ->
                _detailState.value = resource
            }
        }
    }
}
```

**Resultado Esperado:**

Dos ViewModels con inyección Hilt que exponen estados reactivos: `PostListViewModel` con soporte para offline-first, paginación, búsqueda y preferencias; `PostDetailViewModel` con carga de post y comentarios.

**Verificación:**

```bash
./gradlew assembleDebug 2>&1 | tail -5
```

`BUILD SUCCESSFUL`.

---

### Paso 9 — Construir la Interfaz de Usuario con Compose

**Objetivo:** Crear las pantallas Compose que consumen los ViewModels, mostrando la lista paginada de posts, búsqueda, indicadores de estado offline-first y pantalla de detalle con comentarios.

**Instrucciones:**

1. Crea `ui.screen/PostListScreen.kt`:

```kotlin
package com.cursoadv.android.persistencia.ui.screen

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.DarkMode
import androidx.compose.material.icons.filled.LightMode
import androidx.compose.material.icons.filled.Refresh
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Scaffold
import androidx.compose.material3.SnackbarHost
import androidx.compose.material3.SnackbarHostState
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.material3.TopAppBarDefaults
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.paging.LoadState
import androidx.paging.compose.collectAsLazyPagingItems
import com.cursoadv.android.persistencia.domain.model.Post
import com.cursoadv.android.persistencia.domain.util.Resource
import com.cursoadv.android.persistencia.ui.viewmodel.PostListViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PostListScreen(
    onPostClick: (Int) -> Unit,
    viewModel: PostListViewModel = hiltViewModel()
) {
    val postsState by viewModel.postsState.collectAsState()
    val searchQuery by viewModel.searchQuery.collectAsState()
    val searchResults by viewModel.searchResults.collectAsState()
    val preferences by viewModel.userPreferences.collectAsState()
    val pagedPosts = viewModel.pagedPosts.collectAsLazyPagingItems()
    val snackbarHostState = remember { SnackbarHostState() }

    // Mostrar errores como Snackbar
    LaunchedEffect(postsState) {
        if (postsState is Resource.Error) {
            snackbarHostState.showSnackbar(
                message = (postsState as Resource.Error).message
            )
        }
    }

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) },
        topBar = {
            TopAppBar(
                title = { Text("Posts Offline-First") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                ),
                actions = {
                    IconButton(onClick = { viewModel.toggleDarkTheme() }) {
                        Icon(
                            imageVector = if (preferences.darkTheme)
                                Icons.Default.LightMode else Icons.Default.DarkMode,
                            contentDescription = "Cambiar tema"
                        )
                    }
                    IconButton(onClick = { viewModel.forceRefresh() }) {
                        Icon(
                            imageVector = Icons.Default.Refresh,
                            contentDescription = "Forzar sincronización"
                        )
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
            // Barra de búsqueda
            OutlinedTextField(
                value = searchQuery,
                onValueChange = { viewModel.onSearchQueryChanged(it) },
                label = { Text("Buscar posts...") },
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp, vertical = 8.dp),
                singleLine = true
            )

            // Indicador de estado
            when (postsState) {
                is Resource.Loading -> {
                    Box(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(8.dp),
                        contentAlignment = Alignment.Center
                    ) {
                        Row(
                            horizontalArrangement = Arrangement.spacedBy(8.dp),
                            verticalAlignment = Alignment.CenterVertically
                        ) {
                            CircularProgressIndicator(
                                modifier = Modifier.height(20.dp)
                            )
                            Text(
                                text = "Sincronizando...",
                                style = MaterialTheme.typography.bodySmall
                            )
                        }
                    }
                }
                else -> { /* No mostrar nada adicional */ }
            }

            // Lista paginada o resultados de búsqueda
            if (searchQuery.isNotBlank()) {
                // Modo búsqueda: lista simple
                LazyColumn(
                    modifier = Modifier.fillMaxSize(),
                    verticalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    items(searchResults.size) { index ->
                        PostCard(
                            post = searchResults[index],
                            onClick = { onPostClick(searchResults[index].id) }
                        )
                    }
                }
            } else {
                // Modo normal: lista paginada
                LazyColumn(
                    modifier = Modifier.fillMaxSize(),
                    verticalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    items(pagedPosts.itemCount) { index ->
                        val post = pagedPosts[index]
                        if (post != null) {
                            PostCard(
                                post = post,
                                onClick = { onPostClick(post.id) }
                            )
                        }
                    }

                    // Estado de carga de paginación
                    when (pagedPosts.loadState.append) {
                        is LoadState.Loading -> {
                            item {
                                Box(
                                    modifier = Modifier
                                        .fillMaxWidth()
                                        .padding(16.dp),
                                    contentAlignment = Alignment.Center
                                ) {
                                    CircularProgressIndicator()
                                }
                            }
                        }
                        is LoadState.Error -> {
                            item {
                                Text(
                                    text = "Error al cargar más posts",
                                    modifier = Modifier.padding(16.dp),
                                    color = MaterialTheme.colorScheme.error
                                )
                            }
                        }
                        is LoadState.NotLoading -> { /* Nada */ }
                    }
                }
            }
        }
    }
}

@Composable
fun PostCard(
    post: Post,
    onClick: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp)
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = post.title,
                style = MaterialTheme.typography.titleMedium,
                maxLines = 2,
                overflow = TextOverflow.Ellipsis
            )
            Spacer(modifier = Modifier.height(4.dp))
            Text(
                text = post.body,
                style = MaterialTheme.typography.bodyMedium,
                maxLines = 3,
                overflow = TextOverflow.Ellipsis,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
            Spacer(modifier = Modifier.height(4.dp))
            Text(
                text = "Usuario #${post.userId}",
                style = MaterialTheme.typography.labelSmall,
                color = MaterialTheme.colorScheme.primary
            )
        }
    }
}
```

2. Crea `ui.screen/PostDetailScreen.kt`:

```kotlin
package com.cursoadv.android.persistencia.ui.screen

import androidx.compose.foundation.layout.Arrangement
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
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import com.cursoadv.android.persistencia.domain.model.Comment
import com.cursoadv.android.persistencia.domain.util.Resource
import com.cursoadv.android.persistencia.ui.viewmodel.PostDetailViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PostDetailScreen(
    onNavigateBack: () -> Unit,
    viewModel: PostDetailViewModel = hiltViewModel()
) {
    val detailState by viewModel.detailState.collectAsState()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Detalle del Post") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
                        )
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
            when (val state = detailState) {
                is Resource.Loading -> {
                    CircularProgressIndicator(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }

                is Resource.Success -> {
                    val (post, comments) = state.data
                    LazyColumn(
                        modifier = Modifier
                            .fillMaxSize()
                            .padding(16.dp),
                        verticalArrangement = Arrangement.spacedBy(12.dp)
                    ) {
                        // Cabecera del post
                        item {
                            Text(
                                text = post.title,
                                style = MaterialTheme.typography.headlineSmall
                            )
                            Spacer(modifier = Modifier.height(8.dp))
                            Text(
                                text = post.body,
                                style = MaterialTheme.typography.bodyLarge
                            )
                            Spacer(modifier = Modifier.height(16.dp))
                            Text(
                                text = "Comentarios (${comments.size})",
                                style = MaterialTheme.typography.titleMedium,
                                color = MaterialTheme.colorScheme.primary
                            )
                        }

                        // Lista de comentarios
                        items(comments) { comment ->
                            CommentCard(comment = comment)
                        }
                    }
                }

                is Resource.Error -> {
                    Column(
                        modifier = Modifier.align(Alignment.Center),
                        horizontalAlignment = Alignment.CenterHorizontally
                    ) {
                        Text(
                            text = state.message,
                            color = MaterialTheme.colorScheme.error,
                            style = MaterialTheme.typography.bodyLarge
                        )
                    }
                }
            }
        }
    }
}

@Composable
fun CommentCard(comment: Comment) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 1.dp),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceVariant
        )
    ) {
        Column(modifier = Modifier.padding(12.dp)) {
            Text(
                text = comment.name,
                style = MaterialTheme.typography.titleSmall
            )
            Text(
                text = comment.email,
                style = MaterialTheme.typography.labelSmall,
                color = MaterialTheme.colorScheme.primary
            )
            Spacer(modifier = Modifier.height(4.dp))
            Text(
                text = comment.body,
                style = MaterialTheme.typography.bodySmall
            )
        }
    }
}
```

3. Crea la navegación en `ui.navigation/AppNavigation.kt`:

```kotlin
package com.cursoadv.android.persistencia.ui.navigation

import androidx.compose.runtime.Composable
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument
import com.cursoadv.android.persistencia.ui.screen.PostDetailScreen
import com.cursoadv.android.persistencia.ui.screen.PostListScreen

@Composable
fun AppNavigation() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = "posts"
    ) {
        composable("posts") {
            PostListScreen(
                onPostClick = { postId ->
                    navController.navigate("posts/$postId")
                }
            )
        }

        composable(
            route = "posts/{postId}",
            arguments = listOf(
                navArgument("postId") { type = NavType.IntType }
            )
        ) {
            PostDetailScreen(
                onNavigateBack = { navController.popBackStack() }
            )
        }
    }
}
```

4. Actualiza `MainActivity.kt`:

```kotlin
package com.cursoadv.android.persistencia

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.hilt.navigation.compose.hiltViewModel
import com.cursoadv.android.persistencia.ui.navigation.AppNavigation
import com.cursoadv.android.persistencia.ui.theme.Practica5Theme
import com.cursoadv.android.persistencia.ui.viewmodel.PostListViewModel
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Practica5Theme {
                AppNavigation()
            }
        }
    }
}
```

5. Asegúrate de que la clase `Application` tiene la anotación `@HiltAndroidApp`:

```kotlin
package com.cursoadv.android.persistencia

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class PersistenciaApp : Application()
```

6. Verifica que `AndroidManifest.xml` tiene el permiso de Internet y la referencia a la Application:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".PersistenciaApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Práctica 5"
        android:supportsRtl="true"
        android:theme="@style/Theme.Practica5">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

**Resultado Esperado:**

La aplicación muestra una lista paginada de posts obtenidos vía offline-first (primero de Room, luego sincronizados con la API). La barra de búsqueda filtra en Room localmente. Al tocar un post, se navega al detalle con comentarios. El botón de tema oscuro persiste la preferencia en DataStore.

**Verificación:**

Ejecuta la aplicación en el AVD o dispositivo:

```bash
./gradlew installDebug
adb shell am start -n com.cursoadv.android.persistencia/.MainActivity
```

- La pantalla principal debe mostrar posts cargados desde la API (primera ejecución) o desde Room (ejecuciones posteriores).
- El indicador "Sincronizando..." debe aparecer brevemente mientras se contacta la API.
- La búsqueda debe filtrar posts en tiempo real.
- Al navegar al detalle, se deben mostrar comentarios del post.

---

### Paso 10 — Probar el Comportamiento Offline

**Objetivo:** Verificar que la aplicación funciona correctamente sin conexión a Internet, sirviendo datos desde la caché de Room.

**Instrucciones:**

1. Ejecuta la aplicación con Internet habilitado y espera a que cargue los posts (primera sincronización).

2. Desactiva Internet en el AVD:

```bash
adb shell svc wifi disable
adb shell svc data disable
```

3. Cierra la aplicación completamente (desde recientes) y vuelve a abrirla.

4. Verifica que:
   - Los posts se muestran inmediatamente desde Room.
   - Aparece un mensaje de error tipo Snackbar indicando fallo de sincronización.
   - La búsqueda funciona correctamente sobre datos locales.
   - La navegación al detalle funciona (si los comentarios fueron cacheados previamente).

5. Reactiva Internet:

```bash
adb shell svc wifi enable
```

6. Pulsa el botón de refrescar (ícono de recarga) en la barra superior y verifica que los datos se resincronizan.

**Resultado Esperado:**

La aplicación funciona completamente offline con datos cacheados. Al restaurar la conexión, la sincronización actualiza los datos transparentemente.

**Verificación:**

Observa los logs de la aplicación:

```bash
adb logcat | grep -E "(OkHttp|Room|PostRepository)" --line-buffered
```

Deberás ver:
- Con Internet: peticiones HTTP exitosas y operaciones INSERT en Room.
- Sin Internet: error de conexión en OkHttp, pero la UI sigue mostrando datos de Room.

---

## 7. Validación y Pruebas

### Prueba Integral del Flujo Offline-First

| # | Acción | Resultado Esperado |
|---|---|---|
| 1 | Primera ejecución con Internet | Posts cargados de API, almacenados en Room, mostrados en UI |
| 2 | Segunda ejecución con Internet (< 30 min) | Posts servidos de Room inmediatamente, sin llamada a API |
| 3 | Ejecución sin Internet | Posts de Room mostrados, Snackbar con error de red |
| 4 | Búsqueda "qui" sin Internet | Posts filtrados localmente por título/body |
| 5 | Detalle de post con Internet | Post + comentarios cargados y cacheados |
| 6 | Cambio de tema oscuro → cerrar → abrir | Preferencia persistida en DataStore |
| 7 | Forzar sincronización (botón refresh) | Datos eliminados y recargados desde API |
| 8 | Scroll en lista paginada | Carga incremental sin bloqueo de UI |

### Prueba de Migración de Base de Datos

Para verificar que la migración funciona, puedes escribir un test instrumentado (opcional):

```kotlin
@RunWith(AndroidJUnit4::class)
class MigrationTest {
    @get:Rule
    val helper = MigrationTestHelper(
        InstrumentationRegistry.getInstrumentation(),
        AppDatabase::class.java
    )

    @Test
