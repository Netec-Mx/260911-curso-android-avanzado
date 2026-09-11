---LAB_START---
LAB_ID: 03-00-01
---MARKDOWN---
# Práctica 3 — Arquitectura MVVM + Clean Architecture con Hilt

| Campo | Detalle |
|---|---|
| **Duración** | 216 minutos (3 h 36 min) |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |

## Descripción General

En esta práctica transformarás la aplicación Compose construida en la Práctica 2 en un proyecto con arquitectura de producción. Aplicarás Clean Architecture separando el código en tres capas (`data`, `domain`, `presentation`), implementarás ViewModels avanzados que exponen `StateFlow` y `SharedFlow`, crearás Use Cases invocables con `operator fun invoke()`, diseñarás el patrón Repository con interfaces en la capa de dominio e implementaciones en la capa de datos, y configurarás Hilt como framework de inyección de dependencias. Al finalizar, tendrás una arquitectura lista para conectarse a APIs reales (Práctica 4) y bases de datos locales (Práctica 5).

## Objetivos de Aprendizaje

- [ ] Reestructurar un proyecto Android en capas `data`, `domain` y `presentation` con dependencias unidireccionales siguiendo Clean Architecture.
- [ ] Implementar ViewModels avanzados que expongan `StateFlow<UiState<T>>` para estado de UI y `SharedFlow` para eventos únicos (snackbars, navegación).
- [ ] Crear Use Cases como clases individuales con `operator fun invoke()` que encapsulen lógica de negocio e invoquen repositorios.
- [ ] Configurar Hilt completo: `@HiltAndroidApp`, `@HiltViewModel`, módulos con `@Provides` y `@Binds`, e integración con Compose Navigation mediante `hiltViewModel()`.
- [ ] Modelar estados de UI y errores usando `sealed class UiState` con variantes `Loading`, `Success` y `Error`, incluyendo lógica de reintento.

## Prerrequisitos

### Conocimiento Previo

| Tema | Nivel Requerido |
|---|---|
| Kotlin: corrutinas, Flows, sealed classes | Práctica 1 completada |
| Jetpack Compose: composables, navegación | Práctica 2 completada |
| StateFlow y SharedFlow | Comprensión conceptual (Práctica 1) |
| Patrón MVC o MVP | Básico (punto de comparación) |

### Acceso y Recursos

- Proyecto de la Práctica 2 funcional (o disposición para crear el proyecto base desde cero siguiendo las instrucciones de esta guía).
- Conexión a Internet para descarga de dependencias Gradle.
- Cuenta de Google (para acceso a documentación de Android Developers si se requiere consulta).

## Entorno del Laboratorio

### Hardware Mínimo

| Componente | Requisito |
|---|---|
| Procesador | Intel Core i7 8ª gen+ / AMD Ryzen 7+ / Apple M1+ |
| RAM | 16 GB mínimo (32 GB recomendado) |
| Disco | 60 GB libres en SSD |
| Virtualización | VT-x / AMD-V habilitado en BIOS |

### Software Requerido

| Herramienta | Versión Exacta |
|---|---|
| Android Studio | Quail 3 (2026.1.3 Patch 1) |
| Kotlin | 2.2.10 |
| AGP | 9.3.2 |
| Gradle Wrapper | 8.14.1 |
| KSP | 2.2.10-1.0.31 |
| Compose BOM | 2026.02.01 |
| Hilt (Dagger Hilt) | 2.56.2 |
| Hilt Navigation Compose | 1.2.0 |
| Lifecycle ViewModel KTX | 2.9.1 |
| Activity Compose | 1.13.0 |
| compileSdk / targetSdk | 37 |
| minSdk | 30 |

### Configuración Inicial del Entorno

Abre una terminal y verifica que el directorio de trabajo existe:

```bash
mkdir -p ~/AndroidStudioProjects/AvanzadoKotlin/Practica3_Arquitectura
```

---

## Paso 1: Crear el Proyecto Base y Configurar el Catálogo de Versiones

### Objetivo

Crear un proyecto Android con Jetpack Compose desde Android Studio y configurar todas las dependencias exactas en `libs.versions.toml`, incluyendo Hilt, Lifecycle y Navigation.

### Instrucciones

1. Abre **Android Studio Quail 3**.

2. Selecciona **File → New → New Project → Empty Activity** (plantilla Compose).

3. Configura los siguientes valores:

   | Campo | Valor |
   |---|---|
   | Name | `Practica3_Arquitectura` |
   | Package name | `com.cursoadv.android.arquitectura` |
   | Save location | `~/AndroidStudioProjects/AvanzadoKotlin/Practica3_Arquitectura` |
   | Minimum SDK | API 30: Android 11 (R) |
   | Build configuration language | Kotlin DSL |

4. Haz clic en **Finish** y espera a que Gradle sincronice.

5. Abre el archivo `gradle/libs.versions.toml` y reemplaza **todo** su contenido con lo siguiente:

```toml
[versions]
agp = "9.3.2"
kotlin = "2.2.10"
ksp = "2.2.10-1.0.31"
compose-bom = "2026.02.01"
activity-compose = "1.13.0"
lifecycle = "2.9.1"
hilt = "2.56.2"
hilt-navigation-compose = "1.2.0"
core-ktx = "1.19.0"
junit = "4.13.2"
androidx-junit = "1.3.0"
espresso-core = "3.7.0"
kotlinx-coroutines = "1.10.2"

[libraries]
# Core
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "core-ktx" }

# Compose BOM
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
compose-navigation = { group = "androidx.navigation", name = "navigation-compose" }

# Activity
activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activity-compose" }

# Lifecycle
lifecycle-viewmodel-ktx = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version.ref = "lifecycle" }
lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle" }
lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycle" }
lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycle" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hilt-navigation-compose" }

# Coroutines
kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }

# Testing
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidx-junit" }
espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espresso-core" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinx-coroutines" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

6. Abre el archivo `build.gradle.kts` **del proyecto raíz** (`Practica3_Arquitectura/build.gradle.kts`) y asegúrate de que contiene:

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.hilt.android) apply false
}
```

7. Abre el archivo `app/build.gradle.kts` y reemplázalo completamente:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)
    alias(libs.plugins.hilt.android)
}

android {
    namespace = "com.cursoadv.android.arquitectura"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.cursoadv.android.arquitectura"
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

    // Compose
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.graphics)
    implementation(libs.compose.ui.tooling.preview)
    implementation(libs.compose.material3)
    implementation(libs.compose.navigation)
    implementation(libs.activity.compose)

    // Lifecycle
    implementation(libs.lifecycle.viewmodel.ktx)
    implementation(libs.lifecycle.viewmodel.compose)
    implementation(libs.lifecycle.runtime.compose)
    implementation(libs.lifecycle.runtime.ktx)

    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    // Coroutines
    implementation(libs.kotlinx.coroutines.core)
    implementation(libs.kotlinx.coroutines.android)

    // Testing
    testImplementation(libs.junit)
    testImplementation(libs.kotlinx.coroutines.test)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.espresso.core)

    // Debug
    debugImplementation(libs.compose.ui.tooling)
    debugImplementation(libs.compose.ui.test.manifest)
}
```

8. Sincroniza Gradle: **File → Sync Project with Gradle Files** (o haz clic en el icono del elefante con la flecha de sincronización).

### Resultado Esperado

Gradle sincroniza sin errores. En la ventana **Build** debe aparecer:

```
BUILD SUCCESSFUL in Xs
```

### Verificación

En la pestaña **Project** de Android Studio, verifica que en **External Libraries** aparecen:

- `com.google.dagger:hilt-android:2.56.2`
- `androidx.hilt:hilt-navigation-compose:1.2.0`
- `androidx.lifecycle:lifecycle-viewmodel-ktx:2.9.1`
- `androidx.navigation:navigation-compose` (versión gestionada por BOM)

---

## Paso 2: Crear la Estructura de Paquetes por Capas

### Objetivo

Organizar el proyecto en la estructura de paquetes de Clean Architecture con tres capas principales: `data`, `domain` y `presentation`.

### Instrucciones

1. En el panel **Project** (vista **Android**), haz clic derecho sobre el paquete `com.cursoadv.android.arquitectura` dentro de `app/src/main/java/`.

2. Crea los siguientes paquetes usando **New → Package**. Cada paquete se crea relativo a `com.cursoadv.android.arquitectura`:

```
com.cursoadv.android.arquitectura/
├── data/
│   ├── model/
│   └── repository/
├── domain/
│   ├── model/
│   ├── repository/
│   └── usecase/
├── presentation/
│   ├── navigation/
│   ├── ui/
│   │   ├── components/
│   │   ├── detail/
│   │   └── list/
│   └── viewmodel/
└── di/
```

3. Para crear cada paquete, haz clic derecho en el paquete padre → **New → Package** e ingresa el nombre completo. Por ejemplo, para `data.model` ingresa `data.model` estando posicionado en `com.cursoadv.android.arquitectura`.

4. Crea también el paquete `di` (dependency injection) al mismo nivel que `data`, `domain` y `presentation`.

### Resultado Esperado

La estructura de paquetes en Android Studio muestra las cuatro carpetas principales (`data`, `di`, `domain`, `presentation`) con sus respectivos subpaquetes.

### Verificación

Navega por la estructura en el explorador de archivos del sistema operativo:

```bash
find ~/AndroidStudioProjects/AvanzadoKotlin/Practica3_Arquitectura/app/src/main/java/com/cursoadv/android/arquitectura -type d | sort
```

Deberías ver al menos 12 directorios correspondientes a la estructura definida.

---

## Paso 3: Definir la Capa de Dominio — Entidades, Interfaces de Repositorio y Sealed Class UiState

### Objetivo

Crear las entidades de dominio puras (sin dependencia de Android), las interfaces de repositorio y la sealed class `UiState` que modelará todos los estados posibles de la UI.

### Instrucciones

1. **Entidad de dominio:** Crea el archivo `Item.kt` en el paquete `domain.model`:

```kotlin
package com.cursoadv.android.arquitectura.domain.model

data class Item(
    val id: String,
    val title: String,
    val description: String,
    val category: String,
    val imageUrl: String,
    val isFavorite: Boolean = false
)
```

2. **Sealed class UiState:** Crea el archivo `UiState.kt` en el paquete `domain.model`:

```kotlin
package com.cursoadv.android.arquitectura.domain.model

sealed class UiState<out T> {
    data object Loading : UiState<Nothing>()

    data class Success<out T>(val data: T) : UiState<T>()

    data class Error(
        val message: String,
        val throwable: Throwable? = null
    ) : UiState<Nothing>()
}
```

> **Nota técnica:** Usamos `data object` (disponible desde Kotlin 1.9+) para `Loading` en lugar de `object`, lo que proporciona implementaciones automáticas de `toString()`, `equals()` y `hashCode()`.

3. **Interfaz del repositorio:** Crea el archivo `ItemRepository.kt` en el paquete `domain.repository`:

```kotlin
package com.cursoadv.android.arquitectura.domain.repository

import com.cursoadv.android.arquitectura.domain.model.Item
import kotlinx.coroutines.flow.Flow

interface ItemRepository {
    fun getItems(): Flow<List<Item>>
    suspend fun getItemById(id: String): Item?
    fun searchItems(query: String): Flow<List<Item>>
    suspend fun toggleFavorite(id: String)
}
```

> **Principio clave:** La interfaz vive en `domain` y no tiene ninguna dependencia de Android, Room, Retrofit ni ningún framework. Esto es el núcleo de Clean Architecture: la capa de dominio define contratos, y la capa de datos los implementa.

### Resultado Esperado

Tres archivos Kotlin creados sin errores de compilación. La capa `domain` no importa ninguna clase de Android.

### Verificación

Haz clic en **Build → Make Project** (Ctrl+F9 / Cmd+F9). El proyecto debe compilar sin errores. Inspecciona los imports de cada archivo y confirma que solo aparecen `kotlinx.coroutines.flow.Flow` y clases del propio paquete `domain`.

---

## Paso 4: Implementar los Use Cases (Interactors) con operator invoke()

### Objetivo

Crear tres Use Cases individuales que encapsulen cada operación de negocio, siguiendo el principio de responsabilidad única. Cada uno será invocable directamente como función gracias a `operator fun invoke()`.

### Instrucciones

1. **GetItemsUseCase:** Crea el archivo `GetItemsUseCase.kt` en `domain.usecase`:

```kotlin
package com.cursoadv.android.arquitectura.domain.usecase

import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

class GetItemsUseCase @Inject constructor(
    private val repository: ItemRepository
) {
    operator fun invoke(): Flow<List<Item>> {
        return repository.getItems()
    }
}
```

2. **GetItemDetailUseCase:** Crea `GetItemDetailUseCase.kt` en `domain.usecase`:

```kotlin
package com.cursoadv.android.arquitectura.domain.usecase

import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import javax.inject.Inject

class GetItemDetailUseCase @Inject constructor(
    private val repository: ItemRepository
) {
    suspend operator fun invoke(id: String): Item? {
        return repository.getItemById(id)
    }
}
```

3. **SearchItemsUseCase:** Crea `SearchItemsUseCase.kt` en `domain.usecase`:

```kotlin
package com.cursoadv.android.arquitectura.domain.usecase

import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject

class SearchItemsUseCase @Inject constructor(
    private val repository: ItemRepository
) {
    operator fun invoke(query: String): Flow<List<Item>> {
        val trimmedQuery = query.trim().lowercase()
        return if (trimmedQuery.isBlank()) {
            repository.getItems()
        } else {
            repository.searchItems(trimmedQuery)
        }
    }
}
```

4. **ToggleFavoriteUseCase:** Crea `ToggleFavoriteUseCase.kt` en `domain.usecase`:

```kotlin
package com.cursoadv.android.arquitectura.domain.usecase

import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import javax.inject.Inject

class ToggleFavoriteUseCase @Inject constructor(
    private val repository: ItemRepository
) {
    suspend operator fun invoke(id: String) {
        repository.toggleFavorite(id)
    }
}
```

### Resultado Esperado

Cuatro Use Cases creados. Cada uno tiene una sola responsabilidad, recibe el repositorio por constructor con `@Inject`, y es invocable como función.

### Verificación

Compila el proyecto (**Build → Make Project**). No debe haber errores. Observa que `@Inject` proviene de `javax.inject.Inject` (incluido transitivamente por Hilt). Si aparece un error de import, verifica que `hilt-android` está en las dependencias.

---

## Paso 5: Implementar la Capa de Datos — Modelo de Datos y Repositorio con Datos Simulados

### Objetivo

Crear el modelo de datos de la capa `data`, implementar la interfaz `ItemRepository` con datos simulados (fake data) y un mecanismo de delay para simular latencia de red.

### Instrucciones

1. **Data Transfer Object (DTO):** Crea `ItemDto.kt` en `data.model`:

```kotlin
package com.cursoadv.android.arquitectura.data.model

import com.cursoadv.android.arquitectura.domain.model.Item

data class ItemDto(
    val id: String,
    val title: String,
    val description: String,
    val category: String,
    val imageUrl: String
) {
    fun toDomain(isFavorite: Boolean = false): Item {
        return Item(
            id = id,
            title = title,
            description = description,
            category = category,
            imageUrl = imageUrl,
            isFavorite = isFavorite
        )
    }
}
```

> **Nota:** El DTO tiene una función `toDomain()` que convierte el objeto de datos al modelo de dominio. Esta separación permite que los cambios en la API o la base de datos no afecten la capa de dominio.

2. **Fuente de datos falsa:** Crea `FakeDataSource.kt` en `data.repository`:

```kotlin
package com.cursoadv.android.arquitectura.data.repository

import com.cursoadv.android.arquitectura.data.model.ItemDto

object FakeDataSource {

    val items: List<ItemDto> = listOf(
        ItemDto(
            id = "1",
            title = "Kotlin Coroutines",
            description = "Programación asíncrona estructurada con corrutinas de Kotlin. " +
                "Aprende suspend functions, Dispatchers, Job y manejo de errores.",
            category = "Kotlin",
            imageUrl = "https://picsum.photos/seed/kotlin1/400/300"
        ),
        ItemDto(
            id = "2",
            title = "Jetpack Compose",
            description = "UI declarativa moderna para Android. " +
                "Composables, estado, recomposición y Material 3.",
            category = "UI",
            imageUrl = "https://picsum.photos/seed/compose2/400/300"
        ),
        ItemDto(
            id = "3",
            title = "Clean Architecture",
            description = "Separación de responsabilidades en capas: data, domain y presentation. " +
                "Dependencias unidireccionales y principios SOLID.",
            category = "Arquitectura",
            imageUrl = "https://picsum.photos/seed/clean3/400/300"
        ),
        ItemDto(
            id = "4",
            title = "Hilt - Inyección de Dependencias",
            description = "Framework de inyección de dependencias basado en Dagger. " +
                "Configuración con @HiltAndroidApp, @HiltViewModel y módulos.",
            category = "Arquitectura",
            imageUrl = "https://picsum.photos/seed/hilt4/400/300"
        ),
        ItemDto(
            id = "5",
            title = "Room Database",
            description = "Persistencia local con Room: entidades, DAOs, migraciones " +
                "y consultas reactivas con Flow.",
            category = "Datos",
            imageUrl = "https://picsum.photos/seed/room5/400/300"
        ),
        ItemDto(
            id = "6",
            title = "Retrofit y OkHttp",
            description = "Consumo de APIs REST con Retrofit 3 y OkHttp 5. " +
                "Interceptores, serialización JSON y manejo de errores.",
            category = "Red",
            imageUrl = "https://picsum.photos/seed/retrofit6/400/300"
        ),
        ItemDto(
            id = "7",
            title = "StateFlow y SharedFlow",
            description = "Flujos reactivos calientes para gestión de estado en ViewModels. " +
                "Diferencias con LiveData y patrones de uso.",
            category = "Kotlin",
            imageUrl = "https://picsum.photos/seed/flow7/400/300"
        ),
        ItemDto(
            id = "8",
            title = "Google Maps SDK",
            description = "Integración de mapas interactivos en Android con Google Maps SDK " +
                "y Maps Compose para visualización de ubicaciones.",
            category = "Ubicación",
            imageUrl = "https://picsum.photos/seed/maps8/400/300"
        )
    )
}
```

3. **Implementación del repositorio:** Crea `ItemRepositoryImpl.kt` en `data.repository`:

```kotlin
package com.cursoadv.android.arquitectura.data.repository

import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.flow.update
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class ItemRepositoryImpl @Inject constructor() : ItemRepository {

    private val favoriteIds = MutableStateFlow<Set<String>>(emptySet())

    private val allItems = FakeDataSource.items

    override fun getItems(): Flow<List<Item>> {
        return favoriteIds.map { favorites ->
            allItems.map { dto ->
                dto.toDomain(isFavorite = dto.id in favorites)
            }
        }
    }

    override suspend fun getItemById(id: String): Item? {
        // Simula latencia de red
        delay(500L)
        val dto = allItems.find { it.id == id }
        val favorites = favoriteIds.value
        return dto?.toDomain(isFavorite = dto.id in favorites)
    }

    override fun searchItems(query: String): Flow<List<Item>> {
        return favoriteIds.map { favorites ->
            allItems
                .filter { dto ->
                    dto.title.lowercase().contains(query) ||
                        dto.description.lowercase().contains(query) ||
                        dto.category.lowercase().contains(query)
                }
                .map { dto ->
                    dto.toDomain(isFavorite = dto.id in favorites)
                }
        }
    }

    override suspend fun toggleFavorite(id: String) {
        delay(200L) // Simula operación de escritura
        favoriteIds.update { current ->
            if (id in current) current - id else current + id
        }
    }
}
```

### Resultado Esperado

La capa `data` está completa con un DTO, una fuente de datos falsa y una implementación concreta del repositorio que usa `MutableStateFlow` para gestionar favoritos reactivamente.

### Verificación

Compila el proyecto. Verifica que `ItemRepositoryImpl` implementa todos los métodos de `ItemRepository` sin errores. Confirma que la dependencia fluye en la dirección correcta: `data` → `domain` (la implementación conoce la interfaz, no al revés).

---

## Paso 6: Configurar Hilt — Application Class y Módulo de Inyección

### Objetivo

Configurar Hilt como framework de inyección de dependencias: crear la clase Application anotada, definir el módulo que vincula la interfaz del repositorio con su implementación, y registrar la Application en el Manifest.

### Instrucciones

1. **Application class:** Crea `ArquitecturaApp.kt` en el paquete raíz `com.cursoadv.android.arquitectura`:

```kotlin
package com.cursoadv.android.arquitectura

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class ArquitecturaApp : Application()
```

2. **Módulo de repositorio:** Crea `RepositoryModule.kt` en el paquete `di`:

```kotlin
package com.cursoadv.android.arquitectura.di

import com.cursoadv.android.arquitectura.data.repository.ItemRepositoryImpl
import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import dagger.Binds
import dagger.Module
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindItemRepository(
        impl: ItemRepositoryImpl
    ): ItemRepository
}
```

> **¿Por qué `@Binds` en lugar de `@Provides`?** Usamos `@Binds` cuando simplemente vinculamos una interfaz con su implementación concreta. Es más eficiente que `@Provides` porque no genera un método factory adicional. `@Provides` se usa cuando necesitas lógica de construcción personalizada (por ejemplo, configurar un cliente OkHttp).

3. **Registrar en AndroidManifest.xml:** Abre `app/src/main/AndroidManifest.xml` y agrega el atributo `android:name` a la etiqueta `<application>`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:name=".ArquitecturaApp"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Practica3Arquitectura"
        tools:targetApi="37">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.Practica3Arquitectura">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

> **Importante:** Si tu `AndroidManifest.xml` tiene contenido ligeramente diferente (por ejemplo, nombres de tema distintos), solo asegúrate de agregar `android:name=".ArquitecturaApp"` a la etiqueta `<application>`. No modifiques el resto innecesariamente.

### Resultado Esperado

Hilt está configurado a nivel de aplicación. El módulo vincula `ItemRepository` (interfaz) con `ItemRepositoryImpl` (implementación) como Singleton.

### Verificación

Compila el proyecto. Si Hilt está correctamente configurado con KSP, verás en la carpeta `app/build/generated/ksp/` archivos generados por Hilt (como `ArquitecturaApp_GeneratedInjector`). Si hay errores de compilación relacionados con Hilt, verifica que estás usando `ksp` y no `kapt` en `build.gradle.kts`.

---

## Paso 7: Implementar los ViewModels con StateFlow y SharedFlow

### Objetivo

Crear dos ViewModels anotados con `@HiltViewModel`: uno para la lista de ítems (con búsqueda y favoritos) y otro para el detalle, ambos exponiendo `StateFlow<UiState<T>>` para estado y `SharedFlow` para eventos únicos.

### Instrucciones

1. **Evento UI sealed class:** Crea `UiEvent.kt` en `presentation.viewmodel`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.viewmodel

sealed class UiEvent {
    data class ShowSnackbar(val message: String) : UiEvent()
    data class NavigateToDetail(val itemId: String) : UiEvent()
    data object NavigateBack : UiEvent()
}
```

2. **ItemListViewModel:** Crea `ItemListViewModel.kt` en `presentation.viewmodel`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.model.UiState
import com.cursoadv.android.arquitectura.domain.usecase.GetItemsUseCase
import com.cursoadv.android.arquitectura.domain.usecase.SearchItemsUseCase
import com.cursoadv.android.arquitectura.domain.usecase.ToggleFavoriteUseCase
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharedFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.flatMapLatest
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.flow.onStart
import kotlinx.coroutines.launch
import javax.inject.Inject

@OptIn(ExperimentalCoroutinesApi::class)
@HiltViewModel
class ItemListViewModel @Inject constructor(
    private val getItemsUseCase: GetItemsUseCase,
    private val searchItemsUseCase: SearchItemsUseCase,
    private val toggleFavoriteUseCase: ToggleFavoriteUseCase
) : ViewModel() {

    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()

    val uiState: StateFlow<UiState<List<Item>>> =
        _searchQuery
            .flatMapLatest { query ->
                if (query.isBlank()) {
                    getItemsUseCase()
                } else {
                    searchItemsUseCase(query)
                }
                    .map<List<Item>, UiState<List<Item>>> { items ->
                        UiState.Success(items)
                    }
                    .onStart { emit(UiState.Loading) }
                    .catch { e ->
                        emit(UiState.Error(
                            message = e.message ?: "Error desconocido al cargar ítems",
                            throwable = e
                        ))
                    }
            }
            .toStateFlow(UiState.Loading)

    private val _events = MutableSharedFlow<UiEvent>()
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()

    fun onSearchQueryChanged(query: String) {
        _searchQuery.value = query
    }

    fun onItemClicked(itemId: String) {
        viewModelScope.launch {
            _events.emit(UiEvent.NavigateToDetail(itemId))
        }
    }

    fun onToggleFavorite(itemId: String) {
        viewModelScope.launch {
            try {
                toggleFavoriteUseCase(itemId)
            } catch (e: Exception) {
                _events.emit(
                    UiEvent.ShowSnackbar("Error al actualizar favorito: ${e.message}")
                )
            }
        }
    }

    fun onRetry() {
        // Forzar re-emisión reseteando la query
        val currentQuery = _searchQuery.value
        _searchQuery.value = ""
        _searchQuery.value = currentQuery
    }

    private fun <T> kotlinx.coroutines.flow.Flow<T>.toStateFlow(
        initialValue: T
    ): StateFlow<T> {
        val stateFlow = MutableStateFlow(initialValue)
        viewModelScope.launch {
            this@toStateFlow.collect { value ->
                stateFlow.value = value
            }
        }
        return stateFlow.asStateFlow()
    }
}
```

3. **ItemDetailViewModel:** Crea `ItemDetailViewModel.kt` en `presentation.viewmodel`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.viewmodel

import androidx.lifecycle.SavedStateHandle
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.model.UiState
import com.cursoadv.android.arquitectura.domain.usecase.GetItemDetailUseCase
import com.cursoadv.android.arquitectura.domain.usecase.ToggleFavoriteUseCase
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharedFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class ItemDetailViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle,
    private val getItemDetailUseCase: GetItemDetailUseCase,
    private val toggleFavoriteUseCase: ToggleFavoriteUseCase
) : ViewModel() {

    private val itemId: String = checkNotNull(savedStateHandle["itemId"])

    private val _uiState = MutableStateFlow<UiState<Item>>(UiState.Loading)
    val uiState: StateFlow<UiState<Item>> = _uiState.asStateFlow()

    private val _events = MutableSharedFlow<UiEvent>()
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()

    init {
        loadItem()
    }

    private fun loadItem() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val item = getItemDetailUseCase(itemId)
                if (item != null) {
                    _uiState.value = UiState.Success(item)
                } else {
                    _uiState.value = UiState.Error(
                        message = "Ítem con ID '$itemId' no encontrado"
                    )
                }
            } catch (e: Exception) {
                _uiState.value = UiState.Error(
                    message = e.message ?: "Error desconocido al cargar el detalle",
                    throwable = e
                )
            }
        }
    }

    fun onToggleFavorite() {
        viewModelScope.launch {
            try {
                toggleFavoriteUseCase(itemId)
                // Recargar para reflejar el cambio
                loadItem()
                _events.emit(UiEvent.ShowSnackbar("Favorito actualizado"))
            } catch (e: Exception) {
                _events.emit(
                    UiEvent.ShowSnackbar("Error: ${e.message}")
                )
            }
        }
    }

    fun onRetry() {
        loadItem()
    }

    fun onNavigateBack() {
        viewModelScope.launch {
            _events.emit(UiEvent.NavigateBack)
        }
    }
}
```

### Resultado Esperado

Dos ViewModels creados con `@HiltViewModel`, cada uno exponiendo `StateFlow<UiState<T>>` para estado reactivo y `SharedFlow<UiEvent>` para eventos únicos. El `ItemListViewModel` usa `flatMapLatest` para búsqueda reactiva.

### Verificación

Compila el proyecto. Ambos ViewModels deben compilar sin errores. Verifica que:
- `@HiltViewModel` está presente en ambos.
- Todos los Use Cases se inyectan por constructor con `@Inject`.
- `SavedStateHandle` se inyecta en `ItemDetailViewModel` (Hilt lo proporciona automáticamente).

---

## Paso 8: Construir la Capa de Presentación — Composables de UI

### Objetivo

Implementar las pantallas de Compose que consumen el estado de los ViewModels, manejan los tres estados de `UiState` (Loading, Success, Error) y reaccionan a eventos únicos.

### Instrucciones

1. **Componentes reutilizables:** Crea `CommonComponents.kt` en `presentation.ui.components`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.ui.components

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.material3.Button
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp

@Composable
fun LoadingContent(
    modifier: Modifier = Modifier
) {
    Box(
        modifier = modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        CircularProgressIndicator()
    }
}

@Composable
fun ErrorContent(
    message: String,
    onRetry: () -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "⚠️",
            style = MaterialTheme.typography.displayLarge
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = "Ha ocurrido un error",
            style = MaterialTheme.typography.headlineSmall,
            color = MaterialTheme.colorScheme.error
        )
        Spacer(modifier = Modifier.height(8.dp))
        Text(
            text = message,
            style = MaterialTheme.typography.bodyMedium,
            textAlign = TextAlign.Center,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
        Spacer(modifier = Modifier.height(24.dp))
        Button(onClick = onRetry) {
            Text("Reintentar")
        }
    }
}
```

2. **Tarjeta de ítem:** Crea `ItemCard.kt` en `presentation.ui.components`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.ui.components

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.FavoriteBorder
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.maxLines
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import com.cursoadv.android.arquitectura.domain.model.Item

@Composable
fun ItemCard(
    item: Item,
    onClick: () -> Unit,
    onFavoriteClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Column(
                modifier = Modifier.weight(1f)
            ) {
                Text(
                    text = item.title,
                    style = MaterialTheme.typography.titleMedium,
                    maxLines = 1,
                    overflow = TextOverflow.Ellipsis
                )
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = item.category,
                    style = MaterialTheme.typography.labelMedium,
                    color = MaterialTheme.colorScheme.primary
                )
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = item.description,
                    style = MaterialTheme.typography.bodySmall,
                    maxLines = 2,
                    overflow = TextOverflow.Ellipsis,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
            Spacer(modifier = Modifier.width(8.dp))
            IconButton(onClick = onFavoriteClick) {
                Icon(
                    imageVector = if (item.isFavorite) {
                        Icons.Filled.Favorite
                    } else {
                        Icons.Filled.FavoriteBorder
                    },
                    contentDescription = if (item.isFavorite) {
                        "Quitar de favoritos"
                    } else {
                        "Agregar a favoritos"
                    },
                    tint = if (item.isFavorite) {
                        MaterialTheme.colorScheme.error
                    } else {
                        MaterialTheme.colorScheme.onSurfaceVariant
                    }
                )
            }
        }
    }
}
```

3. **Pantalla de lista:** Crea `ItemListScreen.kt` en `presentation.ui.list`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.ui.list

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Clear
import androidx.compose.material.icons.filled.Search
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.SnackbarHost
import androidx.compose.material3.SnackbarHostState
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.material3.TopAppBar
import androidx.compose.material3.TopAppBarDefaults
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.cursoadv.android.arquitectura.domain.model.UiState
import com.cursoadv.android.arquitectura.presentation.ui.components.ErrorContent
import com.cursoadv.android.arquitectura.presentation.ui.components.ItemCard
import com.cursoadv.android.arquitectura.presentation.ui.components.LoadingContent
import com.cursoadv.android.arquitectura.presentation.viewmodel.ItemListViewModel
import com.cursoadv.android.arquitectura.presentation.viewmodel.UiEvent

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ItemListScreen(
    viewModel: ItemListViewModel,
    onNavigateToDetail: (String) -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val searchQuery by viewModel.searchQuery.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    // Recoger eventos únicos
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.NavigateToDetail -> onNavigateToDetail(event.itemId)
                is UiEvent.ShowSnackbar -> snackbarHostState.showSnackbar(event.message)
                is UiEvent.NavigateBack -> { /* No aplica en lista */ }
            }
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Temas del Curso") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            // Barra de búsqueda
            TextField(
                value = searchQuery,
                onValueChange = viewModel::onSearchQueryChanged,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp, vertical = 8.dp),
                placeholder = { Text("Buscar temas...") },
                leadingIcon = {
                    Icon(Icons.Filled.Search, contentDescription = "Buscar")
                },
                trailingIcon = {
                    if (searchQuery.isNotEmpty()) {
                        IconButton(
                            onClick = { viewModel.onSearchQueryChanged("") }
                        ) {
                            Icon(Icons.Filled.Clear, contentDescription = "Limpiar")
                        }
                    }
                },
                singleLine = true
            )

            // Contenido según estado
            when (val state = uiState) {
                is UiState.Loading -> LoadingContent()
                is UiState.Error -> ErrorContent(
                    message = state.message,
                    onRetry = viewModel::onRetry
                )
                is UiState.Success -> {
                    if (state.data.isEmpty()) {
                        ErrorContent(
                            message = "No se encontraron resultados para \"$searchQuery\"",
                            onRetry = { viewModel.onSearchQueryChanged("") }
                        )
                    } else {
                        LazyColumn(
                            contentPadding = PaddingValues(16.dp),
                            verticalArrangement = Arrangement.spacedBy(12.dp)
                        ) {
                            items(
                                items = state.data,
                                key = { it.id }
                            ) { item ->
                                ItemCard(
                                    item = item,
                                    onClick = { viewModel.onItemClicked(item.id) },
                                    onFavoriteClick = {
                                        viewModel.onToggleFavorite(item.id)
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

4. **Pantalla de detalle:** Crea `ItemDetailScreen.kt` en `presentation.ui.detail`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.ui.detail

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.FavoriteBorder
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.FloatingActionButton
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.SnackbarHost
import androidx.compose.material3.SnackbarHostState
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.material3.TopAppBarDefaults
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.model.UiState
import com.cursoadv.android.arquitectura.presentation.ui.components.ErrorContent
import com.cursoadv.android.arquitectura.presentation.ui.components.LoadingContent
import com.cursoadv.android.arquitectura.presentation.viewmodel.ItemDetailViewModel
import com.cursoadv.android.arquitectura.presentation.viewmodel.UiEvent

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ItemDetailScreen(
    viewModel: ItemDetailViewModel,
    onNavigateBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowSnackbar -> snackbarHostState.showSnackbar(event.message)
                is UiEvent.NavigateBack -> onNavigateBack()
                is UiEvent.NavigateToDetail -> { /* No aplica en detalle */ }
            }
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(
                        text = when (val state = uiState) {
                            is UiState.Success -> state.data.title
                            else -> "Detalle"
                        }
                    )
                },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(
                            Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
                        )
                    }
                },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        },
        floatingActionButton = {
            val state = uiState
            if (state is UiState.Success) {
                FloatingActionButton(
                    onClick = viewModel::onToggleFavorite,
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                ) {
                    Icon(
                        imageVector = if (state.data.isFavorite) {
                            Icons.Filled.Favorite
                        } else {
                            Icons.Filled.FavoriteBorder
                        },
                        contentDescription = "Favorito",
                        tint = if (state.data.isFavorite) {
                            MaterialTheme.colorScheme.error
                        } else {
                            MaterialTheme.colorScheme.onPrimaryContainer
                        }
                    )
                }
            }
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { paddingValues ->
        when (val state = uiState) {
            is UiState.Loading -> LoadingContent(
                modifier = Modifier.padding(paddingValues)
            )
            is UiState.Error -> ErrorContent(
                message = state.message,
                onRetry = viewModel::onRetry,
                modifier = Modifier.padding(paddingValues)
            )
            is UiState.Success -> ItemDetailContent(
                item = state.data,
                modifier = Modifier.padding(paddingValues)
            )
        }
    }
}

@Composable
private fun ItemDetailContent(
    item: Item,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(24.dp)
    ) {
        // Categoría como chip visual
        Text(
            text = item.category.uppercase(),
            style = MaterialTheme.typography.labelLarge,
            color = MaterialTheme.colorScheme.primary
        )
        Spacer(modifier = Modifier.height(8.dp))

        // Título
        Text(
            text = item.title,
            style = MaterialTheme.typography.headlineMedium
        )
        Spacer(modifier = Modifier.height(16.dp))

        // Descripción completa
        Text(
            text = item.description,
            style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
        Spacer(modifier = Modifier.height(24.dp))

        // Información adicional
        Text(
            text = "ID: ${item.id}",
            style = MaterialTheme.typography.bodySmall,
            color = MaterialTheme.colorScheme.outline
        )
        Spacer(modifier = Modifier.height(4.dp))
        Text(
            text = if (item.isFavorite) "★ Marcado como favorito" else "☆ No es favorito",
            style = MaterialTheme.typography.bodyMedium,
            color = if (item.isFavorite) {
                MaterialTheme.colorScheme.error
            } else {
                MaterialTheme.colorScheme.outline
            }
        )
    }
}
```

### Resultado Esperado

Cuatro archivos de Compose creados: dos componentes reutilizables y dos pantallas completas. Cada pantalla consume `StateFlow` con `collectAsStateWithLifecycle()` y maneja los tres estados de `UiState`.

### Verificación

Compila el proyecto. Si aparece un error en el import `androidx.compose.material.icons.Icons`, asegúrate de que `compose-material3` está incluido en las dependencias (los íconos de Material están incluidos transitivamente). El proyecto debe compilar sin errores.

---

## Paso 9: Configurar la Navegación con Hilt y Conectar MainActivity

### Objetivo

Crear el grafo de navegación de Compose integrando `hiltViewModel()` para inyectar ViewModels automáticamente, y configurar `MainActivity` con `@AndroidEntryPoint`.

### Instrucciones

1. **Rutas de navegación:** Crea `NavRoutes.kt` en `presentation.navigation`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.navigation

object NavRoutes {
    const val ITEM_LIST = "item_list"
    const val ITEM_DETAIL = "item_detail/{itemId}"

    fun itemDetail(itemId: String): String = "item_detail/$itemId"
}
```

2. **Grafo de navegación:** Crea `AppNavGraph.kt` en `presentation.navigation`:

```kotlin
package com.cursoadv.android.arquitectura.presentation.navigation

import androidx.compose.runtime.Composable
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.navigation.NavHostController
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument
import com.cursoadv.android.arquitectura.presentation.ui.detail.ItemDetailScreen
import com.cursoadv.android.arquitectura.presentation.ui.list.ItemListScreen
import com.cursoadv.android.arquitectura.presentation.viewmodel.ItemDetailViewModel
import com.cursoadv.android.arquitectura.presentation.viewmodel.ItemListViewModel

@Composable
fun AppNavGraph(
    navController: NavHostController = rememberNavController()
) {
    NavHost(
        navController = navController,
        startDestination = NavRoutes.ITEM_LIST
    ) {
        composable(route = NavRoutes.ITEM_LIST) {
            val viewModel: ItemListViewModel = hiltViewModel()
            ItemListScreen(
                viewModel = viewModel,
                onNavigateToDetail = { itemId ->
                    navController.navigate(NavRoutes.itemDetail(itemId))
                }
            )
        }

        composable(
            route = NavRoutes.ITEM_DETAIL,
            arguments = listOf(
                navArgument("itemId") { type = NavType.StringType }
            )
        ) {
            val viewModel: ItemDetailViewModel = hiltViewModel()
            ItemDetailScreen(
                viewModel = viewModel,
                onNavigateBack = { navController.popBackStack() }
            )
        }
    }
}
```

> **Punto clave:** `hiltViewModel()` es la función de `hilt-navigation-compose` que crea el ViewModel con inyección de dependencias automática. El `SavedStateHandle` del `ItemDetailViewModel` recibe automáticamente el argumento `itemId` de la ruta de navegación.

3. **Actualizar MainActivity:** Abre `MainActivity.kt` y reemplaza su contenido:

```kotlin
package com.cursoadv.android.arquitectura

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.cursoadv.android.arquitectura.presentation.navigation.AppNavGraph
import com.cursoadv.android.arquitectura.ui.theme.Practica3ArquitecturaTheme
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Practica3ArquitecturaTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    AppNavGraph()
                }
            }
        }
    }
}
```

> **Nota:** El nombre del tema `Practica3ArquitecturaTheme` puede variar según lo que generó Android Studio. Verifica el nombre exacto en el archivo `ui/theme/Theme.kt` de tu proyecto y ajústalo si es necesario.

### Resultado Esperado

La navegación está completamente configurada. `MainActivity` tiene `@AndroidEntryPoint` (requerido por Hilt). Los ViewModels se inyectan automáticamente con `hiltViewModel()`.

### Verificación

Compila el proyecto con **Build → Make Project**. Si compila exitosamente, el grafo de inyección de Hilt está correctamente configurado. Si hay errores de Hilt, la sección de Troubleshooting al final de esta guía cubre los casos más comunes.

---

## Paso 10: Ejecutar, Probar la Aplicación y Verificar la Arquitectura

### Objetivo

Ejecutar la aplicación en un emulador o dispositivo físico, validar que todas las capas funcionan correctamente y verificar los flujos de datos entre capas.

### Instrucciones

1. **Configurar AVD:** Si no tienes un emulador configurado, ve a **Tools → Device Manager → Create Virtual Device**. Selecciona un Pixel 7 con API 36 o 37.

2. **Ejecutar la aplicación:** Haz clic en **Run** (▶️) o presiona Shift+F10. Espera a que la aplicación se instale e inicie.

3. **Pruebas funcionales manuales:** Ejecuta las siguientes verificaciones en orden:

   | # | Acción | Resultado Esperado |
   |---|---|---|
   | 1 | La app inicia | Se muestra brevemente un indicador de carga y luego la lista de 8 temas |
   | 2 | Desplázate por la lista | Las 8 tarjetas se muestran con título, categoría y descripción |
   | 3 | Toca el ícono de corazón en "Kotlin Coroutines" | El corazón cambia a rojo (lleno), indicando favorito |
   | 4 | Toca el corazón rojo nuevamente | El corazón vuelve a gris (vacío), se desmarca el favorito |
   | 5 | Escribe "arquitectura" en la barra de búsqueda | Se filtran y muestran solo 2 ítems: "Clean Architecture" y "Hilt - Inyección de Dependencias" |
   | 6 | Escribe "xyz123" en la barra de búsqueda | Se muestra el mensaje "No se encontraron resultados" con botón Reintentar |
   | 7 | Toca el botón "X" en la barra de búsqueda | Se limpia la búsqueda y se muestran todos los ítems |
   | 8 | Toca la tarjeta "Room Database" | Se navega a la pantalla de detalle con título, categoría, descripción completa e ID |
   | 9 | En el detalle, toca el FAB de corazón | El corazón cambia a rojo y aparece un Snackbar "Favorito actualizado" |
   | 10 | Toca la flecha de retroceso | Se regresa a la lista; el ítem "Room Database" muestra el corazón rojo |

4. **Verificar flujo de datos reactivo:** Marca un ítem como favorito en la lista, navega al detalle de ese ítem y confirma que aparece como favorito. Regresa a la lista y confirma que sigue marcado. Esto valida que el `MutableStateFlow<Set<String>>` del repositorio funciona como fuente única de verdad.

5. **Verificar rotación de pantalla:** Rota el emulador (Ctrl+Flecha izquierda/derecha). La lista debe mantener su estado (query de búsqueda, favoritos) gracias a que `StateFlow` en el `ViewModel` sobrevive a la recreación de la Activity.

### Resultado Esperado

La aplicación funciona completamente: lista con búsqueda, favoritos reactivos, navegación a detalle, eventos de Snackbar, y persistencia de estado ante rotación.

### Verificación

Todas las 10 pruebas funcionales de la tabla anterior pasan satisfactoriamente. La aplicación no presenta crashes ni ANRs. En **Logcat**, no deben aparecer excepciones no controladas.

---

## Validación y Pruebas

Para validar la arquitectura de forma programática, crea una prueba unitaria del `GetItemsUseCase`.

1. Crea el directorio de test si no existe: `app/src/test/java/com/cursoadv/android/arquitectura/domain/usecase/`

2. Crea `GetItemsUseCaseTest.kt`:

```kotlin
package com.cursoadv.android.arquitectura.domain.usecase

import com.cursoadv.android.arquitectura.domain.model.Item
import com.cursoadv.android.arquitectura.domain.repository.ItemRepository
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.test.runTest
import org.junit.Assert.assertEquals
import org.junit.Test

class GetItemsUseCaseTest {

    private val fakeItems = listOf(
        Item("1", "Test Item", "Desc", "Cat", "url", false),
        Item("2", "Another Item", "Desc2", "Cat2", "url2", true)
    )

    private val fakeRepository = object : ItemRepository {
        override fun getItems(): Flow<List<Item>> = flowOf(fakeItems)
        override suspend fun getItemById(id: String): Item? = fakeItems.find { it.id == id }
        override fun searchItems(query: String): Flow<List<Item>> = flowOf(
            fakeItems.filter { it.title.lowercase().contains(query) }
        )
        override suspend fun toggleFavorite(id: String) { /* no-op */ }
    }

    @Test
    fun `invoke returns all items from repository`() = runTest {
        val useCase = GetItemsUseCase(fakeRepository)

        val result = useCase().first()

        assertEquals(2, result.size)
        assertEquals("Test Item", result[0].title)
        assertEquals(true, result[1].isFavorite)
    }
}
```

3. Ejecuta la prueba: haz clic derecho sobre el archivo → **Run 'GetItemsUseCaseTest'**, o desde terminal:

```bash
cd ~/AndroidStudioProjects/AvanzadoKotlin/Practica3_Arquitectura
./gradlew test
```

### Resultado Esperado

```
> Task :app:testDebugUnitTest
GetItemsUseCaseTest > invoke returns all items from repository PASSED

BUILD SUCCESSFUL
```

La prueba demuestra que el Use Case es completamente testeable porque el repositorio está abstraído detrás de una interfaz, permitiendo sustituirlo por un fake sin ninguna dependencia de Android.

---

## Solución de Problemas

### Problema 1: Error de compilación de Hilt — "Expected @HiltAndroidApp to have a value"

**Síntomas:** Al compilar, aparece un error similar a:

```
[ksp] Expected @HiltAndroidApp to have a value. Did you forget to apply the Gradle Plugin?
```

O bien:

```
error: [Hilt] Hilt Activity must be attached to an @HiltAndroidApp Application.
```

**Causa:** El plugin de Hilt no está aplicado en el `build.gradle.kts` del proyecto raíz, o el plugin `hilt-android` no se declaró correctamente en `libs.versions.toml`.

**Solución:**

1. Verifica que `build.gradle.kts` **raíz** contiene:
   ```kotlin
   alias(libs.plugins.hilt.android) apply false
   ```

2. Verifica que `app/build.gradle.kts` contiene:
   ```kotlin
   alias(libs.plugins.hilt.android)
   ```

3. Verifica que en `libs.versions.toml` el plugin está declarado como:
   ```toml
   hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
   ```

4. Ejecuta **File → Invalidate Caches → Invalidate and Restart**, luego sincroniza Gradle y recompila.

---

### Problema 2: SavedStateHandle no recibe el argumento "itemId" — crash en ItemDetailViewModel

**Síntomas:** Al navegar a la pantalla de detalle, la aplicación crashea con:

```
java.lang.IllegalStateException: Required value was null.
```

En la línea:
```kotlin
private val itemId: String = checkNotNull(savedStateHandle["itemId"])
```

**Causa:** El nombre del argumento en la ruta de navegación no coincide con la clave usada en `SavedStateHandle`. Esto ocurre típicamente cuando la ruta dice `{itemId}` pero el `navArgument` usa un nombre diferente, o cuando se navega a una ruta sin incluir el valor del argumento.

**Solución:**

1. Verifica que la ruta en `NavRoutes` usa exactamente `{itemId}`:
   ```kotlin
   const val ITEM_DETAIL = "item_detail/{itemId}"
   ```

2. Verifica que `navArgument` usa el mismo nombre:
   ```kotlin
   navArgument("itemId") { type = NavType.StringType }
   ```

3. Verifica que la función de navegación construye la ruta correctamente:
   ```kotlin
   fun itemDetail(itemId: String): String = "item_detail/$itemId"
   ```

4. Verifica que al navegar se pasa un `itemId` no nulo:
   ```kotlin
   navController.navigate(NavRoutes.itemDetail(itemId))
   ```

Los tres nombres deben coincidir exactamente: `{itemId}` en la ruta, `"itemId"` en `navArgument`, y `savedStateHandle["itemId"]` en el ViewModel.

---

## Limpieza

Si deseas liberar espacio en disco o preparar el entorno para la siguiente práctica:

```bash
# Limpiar artefactos de compilación (libera ~500 MB)
cd ~/AndroidStudioProjects/AvanzadoKotlin/Practica3_Arquitectura
./gradlew clean

# (Opcional) Eliminar la caché de Gradle de este proyecto
rm -rf .gradle/
```

> **No elimines el proyecto completo.** Las Prácticas 4 y 5 construirán sobre esta base arquitectónica, conectando Retrofit para APIs REST y Room para persistencia local.

---

## Resumen

En esta práctica implementaste una arquitectura de producción completa para una aplicación Android con Jetpack Compose. Los logros clave son:

| Capa | Componentes Creados | Principio Aplicado |
|---|---|---|
| `domain` | `Item`, `UiState`, `ItemRepository` (interfaz), 4 Use Cases | Independencia de frameworks, SRP |
| `data` | `ItemDto`, `FakeDataSource`, `ItemRepositoryImpl` | Implementación concreta, fuente única de verdad |
| `presentation` | 2 ViewModels, 2 pantallas, 2 componentes reutilizables | MVVM, estado reactivo con StateFlow |
| `di` | `RepositoryModule`, `ArquitecturaApp` | Inversión de dependencias con Hilt |
| `navigation` | `NavRoutes`, `AppNavGraph` con `hiltViewModel()` | Integración Hilt + Compose Navigation |

**Conceptos arquitectónicos consolidados:**

- **Dependencias unidireccionales:** `presentation` → `domain` ← `data`. La capa de dominio no conoce ni a Android ni a ningún framework.
- **StateFlow para estado, SharedFlow para eventos:** El ViewModel expone
