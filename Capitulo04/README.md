---LAB_START---
LAB_ID: 04-00-01
---MARKDOWN---
# Práctica 4 — Integración de APIs REST con Retrofit 3, OkHttp 5 y Arquitectura MVVM

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 216 minutos (3 horas 36 minutos) |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar |

## 2. Descripción General

En esta práctica construirás una capa de red completa para una aplicación Android que consume la API pública JSONPlaceholder. Partiendo de la arquitectura MVVM con Hilt establecida en la Práctica 3, integrarás Retrofit 3.0.0 con OkHttp 5.5.0, implementarás serialización JSON con Kotlinx Serialization 1.8.1, configurarás interceptores de autenticación y logging, y propagarás estados de red (`Loading`, `Success`, `Error`) desde el repositorio hasta la UI de Compose mediante `StateFlow`. Al finalizar, la aplicación mostrará posts reales obtenidos de internet y permitirá crear nuevos posts mediante operaciones POST.

## 3. Objetivos de Aprendizaje

Al completar esta práctica, serás capaz de:

- [ ] Configurar Retrofit 3 con OkHttp 5 como cliente HTTP, integrándolos mediante módulos Hilt con `@Provides` y `@Singleton`
- [ ] Implementar operaciones GET y POST con autenticación Bearer Token y API Key usando interceptores personalizados de OkHttp
- [ ] Aplicar serialización y deserialización JSON con Kotlinx Serialization, mapeando DTOs de red a modelos de dominio
- [ ] Manejar errores de red de forma robusta usando `sealed class NetworkResult<T>` con `Flow`, operadores `catch`, `retry` y timeouts
- [ ] Propagar estados de red (`Loading`/`Success`/`Error`) desde la capa de datos hasta la UI de Compose mediante `StateFlow`

## 4. Prerrequisitos

### Conocimientos Requeridos

| Conocimiento | Nivel | Referencia |
|---|---|---|
| Arquitectura MVVM con Hilt (Práctica 3) | Completada | Capítulo 3 del curso |
| Corrutinas y Flow de Kotlin | Intermedio | Práctica 1 del curso |
| Jetpack Compose básico | Intermedio | Práctica 2 del curso |
| Protocolo HTTP (GET, POST, headers, códigos de respuesta) | Básico | Lección 4.1 |

### Accesos Requeridos

- Conexión a Internet estable (mínimo 20 Mbps) para acceder a `jsonplaceholder.typicode.com`
- Android Studio Quail 3 (2026.1.3 Patch 1) instalado y funcional
- AVD configurado con API 36 o dispositivo físico con Android 12+ (API 30+)

## 5. Entorno de Laboratorio

### Hardware Mínimo

| Componente | Requisito |
|---|---|
| Procesador | Intel Core i7 8ª gen+ / AMD Ryzen 7+ / Apple M1+ |
| RAM | 16 GB mínimo (32 GB recomendado) |
| Almacenamiento | 60 GB libres en SSD |
| Virtualización | VT-x / AMD-V / Hypervisor habilitado en BIOS |

### Software Requerido

| Herramienta | Versión Exacta |
|---|---|
| Android Studio | Quail 3 (2026.1.3 Patch 1) |
| Kotlin | 2.2.10 |
| AGP | 9.3.2 |
| Gradle Wrapper | 8.14.1 |
| KSP | 2.2.10-1.0.31 |
| Compose BOM | 2026.02.01 |
| Hilt | 2.56.2 |
| Retrofit | 3.0.0 |
| OkHttp BOM | 5.5.0 |
| Kotlinx Serialization | 1.8.1 |
| compileSdk / targetSdk | 37 |
| minSdk | 30 |

### Preparación Inicial del Directorio

```bash
# Verificar que existe el directorio de trabajo del curso
mkdir -p ~/AndroidStudioProjects/AvanzadoKotlin/Practica4_Red
```

## 6. Instrucciones Paso a Paso

---

### Paso 1: Crear el Proyecto Android en Android Studio

**Objetivo:** Crear un proyecto nuevo con la plantilla Empty Activity (Compose) y configurar la estructura base del proyecto.

**Instrucciones:**

1. Abre Android Studio Quail 3.

2. Selecciona **File → New → New Project**.

3. Elige la plantilla **Empty Activity** (la plantilla con Jetpack Compose).

4. Configura los campos del proyecto:

   | Campo | Valor |
   |---|---|
   | Name | `Practica4_Red` |
   | Package name | `com.cursoadv.android.red` |
   | Save location | `~/AndroidStudioProjects/AvanzadoKotlin/Practica4_Red` |
   | Minimum SDK | API 30: Android 11 (R) |
   | Build configuration language | Kotlin DSL |

5. Haz clic en **Finish** y espera a que Gradle sincronice completamente.

6. Verifica que el proyecto compila ejecutando **Build → Make Project** (Ctrl+F9 / Cmd+F9).

**Resultado esperado:** El proyecto se crea sin errores y la sincronización de Gradle finaliza correctamente.

**Verificación:**

```
BUILD SUCCESSFUL
```

---

### Paso 2: Configurar el Catálogo de Versiones (`libs.versions.toml`)

**Objetivo:** Definir todas las dependencias con versiones exactas bloqueadas en el catálogo centralizado de versiones.

**Instrucciones:**

1. Abre el archivo `gradle/libs.versions.toml` en la raíz del proyecto.

2. Reemplaza **todo** su contenido con lo siguiente:

```toml
[versions]
agp = "9.3.2"
kotlin = "2.2.10"
ksp = "2.2.10-1.0.31"
compose-bom = "2026.02.01"
activity-compose = "1.13.0"
core-ktx = "1.19.0"
lifecycle-runtime-ktx = "2.9.1"
lifecycle-viewmodel-compose = "2.9.1"
hilt = "2.56.2"
hilt-navigation-compose = "1.2.0"
retrofit = "3.0.0"
okhttp-bom = "5.5.0"
kotlinx-serialization = "1.8.1"
kotlinx-serialization-retrofit = "1.0.0"
junit = "4.13.2"
androidx-junit = "1.3.0"
espresso-core = "3.7.0"

[libraries]
# AndroidX Core
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "core-ktx" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activity-compose" }

# Lifecycle
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycle-runtime-ktx" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle-viewmodel-compose" }

# Compose BOM
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hilt-navigation-compose" }

# Retrofit
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-kotlinx-serialization = { group = "com.squareup.retrofit2", name = "converter-kotlinx-serialization", version.ref = "retrofit" }

# OkHttp BOM
okhttp-bom = { group = "com.squareup.okhttp3", name = "okhttp-bom", version.ref = "okhttp-bom" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor" }

# Kotlinx Serialization
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinx-serialization" }

# Testing
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidx-junit" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espresso-core" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

3. Guarda el archivo.

**Resultado esperado:** El catálogo define todas las versiones de forma centralizada sin usar `latest`, `+` ni rangos.

**Verificación:** Revisa visualmente que cada `version.ref` apunta a una versión numérica exacta en la sección `[versions]`.

---

### Paso 3: Configurar `build.gradle.kts` del Proyecto (Nivel Raíz)

**Objetivo:** Registrar todos los plugins necesarios en el archivo de build del proyecto raíz.

**Instrucciones:**

1. Abre `build.gradle.kts` en la raíz del proyecto (el que NO está dentro de `app/`).

2. Asegúrate de que su contenido sea:

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.hilt.android) apply false
}
```

3. Guarda el archivo.

**Resultado esperado:** Todos los plugins se declaran sin aplicarse, listos para ser activados en el módulo `app`.

---

### Paso 4: Configurar `build.gradle.kts` del Módulo `app`

**Objetivo:** Aplicar los plugins, configurar el SDK, las opciones de compilación y todas las dependencias del proyecto.

**Instrucciones:**

1. Abre `app/build.gradle.kts`.

2. Reemplaza **todo** su contenido con:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.ksp)
    alias(libs.plugins.hilt.android)
}

android {
    namespace = "com.cursoadv.android.red"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.cursoadv.android.red"
        minSdk = 30
        targetSdk = 37
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        debug {
            buildConfigField("String", "BASE_URL", "\"https://jsonplaceholder.typicode.com/\"")
            buildConfigField("String", "API_KEY", "\"practica4-demo-api-key-2025\"")
        }
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            buildConfigField("String", "BASE_URL", "\"https://jsonplaceholder.typicode.com/\"")
            buildConfigField("String", "API_KEY", "\"practica4-demo-api-key-2025\"")
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
        buildConfig = true
    }
}

dependencies {
    // AndroidX Core
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.activity.compose)

    // Lifecycle
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.viewmodel.compose)

    // Compose BOM
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.graphics)
    implementation(libs.compose.ui.tooling.preview)
    implementation(libs.compose.material3)
    debugImplementation(libs.compose.ui.tooling)

    // Hilt
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)

    // Retrofit
    implementation(libs.retrofit)
    implementation(libs.retrofit.kotlinx.serialization)

    // OkHttp BOM
    implementation(platform(libs.okhttp.bom))
    implementation(libs.okhttp)
    implementation(libs.okhttp.logging)

    // Kotlinx Serialization
    implementation(libs.kotlinx.serialization.json)

    // Testing
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
}
```

3. Guarda el archivo y ejecuta **Sync Now** en la barra amarilla que aparece.

4. Espera a que la sincronización de Gradle finalice sin errores.

**Resultado esperado:**

```
BUILD SUCCESSFUL
```

**Verificación:** En la ventana **Build**, confirma que no hay errores de resolución de dependencias. Todas las librerías deben descargarse correctamente.

---

### Paso 5: Configurar el Permiso de Internet y la Clase Application con Hilt

**Objetivo:** Habilitar el acceso a internet y configurar Hilt como framework de inyección de dependencias.

**Instrucciones:**

1. Abre `app/src/main/AndroidManifest.xml` y añade el permiso de Internet **antes** de la etiqueta `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".RedApp"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Practica4Red"
        tools:targetApi="37">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.Practica4Red">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

> **Nota:** Ajusta `android:theme` al nombre de tema que Android Studio generó para tu proyecto. Verifica el nombre exacto en `res/values/themes.xml`.

2. Crea la clase `RedApp.kt` en el paquete `com.cursoadv.android.red`:

```kotlin
package com.cursoadv.android.red

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class RedApp : Application()
```

3. Abre `MainActivity.kt` y asegúrate de que tiene la anotación `@AndroidEntryPoint`:

```kotlin
package com.cursoadv.android.red

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.cursoadv.android.red.ui.theme.Practica4RedTheme
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Practica4RedTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    // Se completará en pasos posteriores
                }
            }
        }
    }
}
```

> **Nota:** Ajusta `Practica4RedTheme` al nombre del tema Compose que Android Studio generó automáticamente. Revisa el archivo en `ui/theme/Theme.kt`.

4. Compila el proyecto con **Build → Make Project**.

**Resultado esperado:** El proyecto compila exitosamente con Hilt configurado y el permiso de Internet declarado.

**Verificación:**

```
BUILD SUCCESSFUL
```

---

### Paso 6: Crear los Modelos de Datos — DTOs y Entidades de Dominio

**Objetivo:** Definir las clases de datos serializables para la comunicación con la API (DTOs) y las entidades de dominio separadas, aplicando el principio de separación de capas.

**Instrucciones:**

1. Crea la siguiente estructura de paquetes dentro de `com.cursoadv.android.red`:

```
com.cursoadv.android.red
├── data
│   ├── model
│   ├── network
│   └── repository
├── di
├── domain
│   ├── model
│   └── usecase
├── ui
│   ├── screen
│   └── theme
└── util
```

2. Crea el DTO de red `PostDto.kt` en `data/model/`:

```kotlin
package com.cursoadv.android.red.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class PostDto(
    @SerialName("id")
    val id: Int,

    @SerialName("userId")
    val userId: Int,

    @SerialName("title")
    val title: String,

    @SerialName("body")
    val body: String
)
```

3. Crea el DTO para crear un post `CreatePostDto.kt` en `data/model/`:

```kotlin
package com.cursoadv.android.red.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class CreatePostDto(
    @SerialName("userId")
    val userId: Int,

    @SerialName("title")
    val title: String,

    @SerialName("body")
    val body: String
)
```

4. Crea el DTO de comentarios `CommentDto.kt` en `data/model/`:

```kotlin
package com.cursoadv.android.red.data.model

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class CommentDto(
    @SerialName("id")
    val id: Int,

    @SerialName("postId")
    val postId: Int,

    @SerialName("name")
    val name: String,

    @SerialName("email")
    val email: String,

    @SerialName("body")
    val body: String
)
```

5. Crea la entidad de dominio `Post.kt` en `domain/model/`:

```kotlin
package com.cursoadv.android.red.domain.model

data class Post(
    val id: Int,
    val userId: Int,
    val title: String,
    val body: String
)
```

6. Crea la entidad de dominio `Comment.kt` en `domain/model/`:

```kotlin
package com.cursoadv.android.red.domain.model

data class Comment(
    val id: Int,
    val postId: Int,
    val name: String,
    val email: String,
    val body: String
)
```

7. Crea el mapper `PostMapper.kt` en `data/model/`:

```kotlin
package com.cursoadv.android.red.data.model

import com.cursoadv.android.red.domain.model.Comment
import com.cursoadv.android.red.domain.model.Post

fun PostDto.toDomain(): Post = Post(
    id = id,
    userId = userId,
    title = title,
    body = body
)

fun CommentDto.toDomain(): Comment = Comment(
    id = id,
    postId = postId,
    name = name,
    email = email,
    body = body
)

fun Post.toCreateDto(): CreatePostDto = CreatePostDto(
    userId = userId,
    title = title,
    body = body
)
```

**Resultado esperado:** Seis archivos creados sin errores de compilación. Los DTOs están anotados con `@Serializable` y los mappers convierten entre capas.

**Verificación:** Ejecuta **Build → Make Project** y confirma `BUILD SUCCESSFUL`.

---

### Paso 7: Crear la Clase Sellada `NetworkResult<T>`

**Objetivo:** Implementar el patrón de estados de red tipado que se usará en toda la capa de datos para representar Loading, Success y Error.

**Instrucciones:**

1. Crea `NetworkResult.kt` en el paquete `util/`:

```kotlin
package com.cursoadv.android.red.util

sealed class NetworkResult<out T> {
    data object Loading : NetworkResult<Nothing>()
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(
        val code: Int = -1,
        val message: String = "Error desconocido"
    ) : NetworkResult<Nothing>()
}
```

2. Crea la función auxiliar `safeApiCall` en `util/SafeApiCall.kt`:

```kotlin
package com.cursoadv.android.red.util

import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.onStart
import retrofit2.HttpException
import java.io.IOException
import java.net.SocketTimeoutException

fun <T> safeApiCall(
    apiCall: suspend () -> T
): Flow<NetworkResult<T>> = flow {
    val response = apiCall()
    emit(NetworkResult.Success(response))
}.onStart {
    emit(NetworkResult.Loading)
}.catch { throwable ->
    when (throwable) {
        is SocketTimeoutException -> emit(
            NetworkResult.Error(
                code = 408,
                message = "Tiempo de espera agotado. Verifica tu conexión."
            )
        )
        is IOException -> emit(
            NetworkResult.Error(
                code = -1,
                message = "Error de conexión. Verifica tu acceso a Internet."
            )
        )
        is HttpException -> emit(
            NetworkResult.Error(
                code = throwable.code(),
                message = throwable.message() ?: "Error HTTP ${throwable.code()}"
            )
        )
        else -> emit(
            NetworkResult.Error(
                code = -1,
                message = throwable.localizedMessage ?: "Error desconocido"
            )
        )
    }
}
```

**Resultado esperado:** Dos archivos que encapsulan todo el manejo de errores de red de forma reutilizable.

**Verificación:** Compila el proyecto. No deben existir errores.

---

### Paso 8: Definir la Interfaz del Servicio API con Retrofit

**Objetivo:** Crear la interfaz declarativa de Retrofit con métodos suspend para operaciones GET y POST contra JSONPlaceholder.

**Instrucciones:**

1. Crea `JsonPlaceholderApi.kt` en `data/network/`:

```kotlin
package com.cursoadv.android.red.data.network

import com.cursoadv.android.red.data.model.CommentDto
import com.cursoadv.android.red.data.model.CreatePostDto
import com.cursoadv.android.red.data.model.PostDto
import retrofit2.http.Body
import retrofit2.http.GET
import retrofit2.http.Header
import retrofit2.http.POST
import retrofit2.http.Path
import retrofit2.http.Query

interface JsonPlaceholderApi {

    @GET("posts")
    suspend fun getPosts(): List<PostDto>

    @GET("posts/{id}")
    suspend fun getPostById(
        @Path("id") postId: Int
    ): PostDto

    @GET("posts")
    suspend fun getPostsByUser(
        @Query("userId") userId: Int
    ): List<PostDto>

    @GET("posts/{postId}/comments")
    suspend fun getCommentsByPost(
        @Path("postId") postId: Int
    ): List<CommentDto>

    @POST("posts")
    suspend fun createPost(
        @Body post: CreatePostDto,
        @Header("X-Custom-Api-Key") apiKey: String
    ): PostDto
}
```

> **Nota técnica:** JSONPlaceholder no requiere autenticación real, pero incluimos el header `@Header("X-Custom-Api-Key")` con fines didácticos para practicar la inyección de API Keys por petición. Además, configuraremos un interceptor Bearer Token en el siguiente paso.

**Resultado esperado:** Una interfaz con cinco endpoints que cubren GET con `@Path`, `@Query`, y POST con `@Body` y `@Header`.

**Verificación:** Compila el proyecto. No deben existir errores de importación.

---

### Paso 9: Crear los Interceptores de OkHttp

**Objetivo:** Implementar un interceptor de autenticación Bearer Token y configurar el interceptor de logging para depuración.

**Instrucciones:**

1. Crea `AuthInterceptor.kt` en `data/network/`:

```kotlin
package com.cursoadv.android.red.data.network

import okhttp3.Interceptor
import okhttp3.Response
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class AuthInterceptor @Inject constructor() : Interceptor {

    // En una aplicación real, este token se obtendría de un
    // TokenManager que lo recupera de DataStore o SharedPreferences
    private var bearerToken: String = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.demo-token"

    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()

        val authenticatedRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer $bearerToken")
            .header("Content-Type", "application/json; charset=UTF-8")
            .header("Accept", "application/json")
            .build()

        return chain.proceed(authenticatedRequest)
    }

    /**
     * Permite actualizar el token dinámicamente.
     * En producción se usaría tras un refresh token exitoso.
     */
    fun updateToken(newToken: String) {
        bearerToken = newToken
    }
}
```

> **Nota importante:** JSONPlaceholder ignora los headers de autenticación, pero el interceptor funciona correctamente y envía los headers en cada petición. Puedes verificarlo en los logs de OkHttp.

**Resultado esperado:** Un interceptor inyectable que agrega automáticamente el header `Authorization: Bearer <token>` a todas las peticiones HTTP.

**Verificación:** Compila el proyecto exitosamente.

---

### Paso 10: Crear el Módulo Hilt de Red (`NetworkModule`)

**Objetivo:** Configurar Retrofit 3, OkHttp 5 con interceptores, logging y timeouts, y proveer todas las dependencias de red mediante Hilt.

**Instrucciones:**

1. Crea `NetworkModule.kt` en `di/`:

```kotlin
package com.cursoadv.android.red.di

import com.cursoadv.android.red.BuildConfig
import com.cursoadv.android.red.data.network.AuthInterceptor
import com.cursoadv.android.red.data.network.JsonPlaceholderApi
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
        prettyPrint = false
        isLenient = true
        encodeDefaults = true
    }

    @Provides
    @Singleton
    fun provideHttpLoggingInterceptor(): HttpLoggingInterceptor {
        return HttpLoggingInterceptor().apply {
            level = if (BuildConfig.DEBUG) {
                HttpLoggingInterceptor.Level.BODY
            } else {
                HttpLoggingInterceptor.Level.NONE
            }
        }
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor,
        loggingInterceptor: HttpLoggingInterceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(loggingInterceptor)
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            .retryOnConnectionFailure(true)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(
        okHttpClient: OkHttpClient,
        json: Json
    ): Retrofit {
        val contentType = "application/json".toMediaType()
        return Retrofit.Builder()
            .baseUrl(BuildConfig.BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(json.asConverterFactory(contentType))
            .build()
    }

    @Provides
    @Singleton
    fun provideJsonPlaceholderApi(
        retrofit: Retrofit
    ): JsonPlaceholderApi {
        return retrofit.create(JsonPlaceholderApi::class.java)
    }
}
```

**Resultado esperado:** Un módulo Hilt completo que provee `Json`, `OkHttpClient` (con auth interceptor, logging interceptor y timeouts de 30s), `Retrofit` y `JsonPlaceholderApi` como singletons.

**Verificación:** Compila el proyecto. Hilt debe procesar las anotaciones sin errores KSP.

---

### Paso 11: Implementar el Repositorio con Flow y NetworkResult

**Objetivo:** Crear el repositorio que consume la API a través de Retrofit y expone los resultados como `Flow<NetworkResult<T>>`.

**Instrucciones:**

1. Crea la interfaz `PostRepository.kt` en `domain/` (no dentro de `domain/model`, sino directamente en `domain/`):

```kotlin
package com.cursoadv.android.red.domain

import com.cursoadv.android.red.domain.model.Comment
import com.cursoadv.android.red.domain.model.Post
import com.cursoadv.android.red.util.NetworkResult
import kotlinx.coroutines.flow.Flow

interface PostRepository {
    fun getPosts(): Flow<NetworkResult<List<Post>>>
    fun getPostById(postId: Int): Flow<NetworkResult<Post>>
    fun getCommentsByPost(postId: Int): Flow<NetworkResult<List<Comment>>>
    fun createPost(post: Post): Flow<NetworkResult<Post>>
}
```

2. Crea la implementación `PostRepositoryImpl.kt` en `data/repository/`:

```kotlin
package com.cursoadv.android.red.data.repository

import com.cursoadv.android.red.BuildConfig
import com.cursoadv.android.red.data.model.toDomain
import com.cursoadv.android.red.data.model.toCreateDto
import com.cursoadv.android.red.data.network.JsonPlaceholderApi
import com.cursoadv.android.red.domain.PostRepository
import com.cursoadv.android.red.domain.model.Comment
import com.cursoadv.android.red.domain.model.Post
import com.cursoadv.android.red.util.NetworkResult
import com.cursoadv.android.red.util.safeApiCall
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class PostRepositoryImpl @Inject constructor(
    private val api: JsonPlaceholderApi
) : PostRepository {

    override fun getPosts(): Flow<NetworkResult<List<Post>>> =
        safeApiCall {
            api.getPosts().map { it.toDomain() }
        }

    override fun getPostById(postId: Int): Flow<NetworkResult<Post>> =
        safeApiCall {
            api.getPostById(postId).toDomain()
        }

    override fun getCommentsByPost(postId: Int): Flow<NetworkResult<List<Comment>>> =
        safeApiCall {
            api.getCommentsByPost(postId).map { it.toDomain() }
        }

    override fun createPost(post: Post): Flow<NetworkResult<Post>> =
        safeApiCall {
            api.createPost(
                post = post.toCreateDto(),
                apiKey = BuildConfig.API_KEY
            ).toDomain()
        }
}
```

3. Crea el módulo Hilt para vincular la interfaz con la implementación. Crea `RepositoryModule.kt` en `di/`:

```kotlin
package com.cursoadv.android.red.di

import com.cursoadv.android.red.data.repository.PostRepositoryImpl
import com.cursoadv.android.red.domain.PostRepository
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
    abstract fun bindPostRepository(
        impl: PostRepositoryImpl
    ): PostRepository
}
```

**Resultado esperado:** El repositorio encapsula todas las llamadas de red usando `safeApiCall`, emitiendo `Loading` → `Success`/`Error` automáticamente.

**Verificación:** Compila el proyecto exitosamente.

---

### Paso 12: Crear los Use Cases

**Objetivo:** Implementar los casos de uso que encapsulan la lógica de negocio entre el repositorio y el ViewModel.

**Instrucciones:**

1. Crea `GetPostsUseCase.kt` en `domain/usecase/`:

```kotlin
package com.cursoadv.android.red.domain.usecase

import com.cursoadv.android.red.domain.PostRepository
import com.cursoadv.android.red.domain.model.Post
import com.cursoadv.android.red.util.NetworkResult
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

class GetPostsUseCase @Inject constructor(
    private val repository: PostRepository
) {
    operator fun invoke(): Flow<NetworkResult<List<Post>>> =
        repository.getPosts()
}
```

2. Crea `GetPostDetailUseCase.kt` en `domain/usecase/`:

```kotlin
package com.cursoadv.android.red.domain.usecase

import com.cursoadv.android.red.domain.PostRepository
import com.cursoadv.android.red.domain.model.Comment
import com.cursoadv.android.red.domain.model.Post
import com.cursoadv.android.red.util.NetworkResult
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.combine
import javax.inject.Inject

data class PostDetail(
    val post: Post,
    val comments: List<Comment>
)

class GetPostDetailUseCase @Inject constructor(
    private val repository: PostRepository
) {
    operator fun invoke(postId: Int): Flow<NetworkResult<PostDetail>> {
        return combine(
            repository.getPostById(postId),
            repository.getCommentsByPost(postId)
        ) { postResult, commentsResult ->
            when {
                postResult is NetworkResult.Loading ||
                commentsResult is NetworkResult.Loading ->
                    NetworkResult.Loading

                postResult is NetworkResult.Error ->
                    NetworkResult.Error(postResult.code, postResult.message)

                commentsResult is NetworkResult.Error ->
                    NetworkResult.Error(commentsResult.code, commentsResult.message)

                postResult is NetworkResult.Success &&
                commentsResult is NetworkResult.Success ->
                    NetworkResult.Success(
                        PostDetail(
                            post = postResult.data,
                            comments = commentsResult.data
                        )
                    )

                else -> NetworkResult.Error(message = "Estado inesperado")
            }
        }
    }
}
```

3. Crea `CreatePostUseCase.kt` en `domain/usecase/`:

```kotlin
package com.cursoadv.android.red.domain.usecase

import com.cursoadv.android.red.domain.PostRepository
import com.cursoadv.android.red.domain.model.Post
import com.cursoadv.android.red.util.NetworkResult
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

class CreatePostUseCase @Inject constructor(
    private val repository: PostRepository
) {
    operator fun invoke(title: String, body: String, userId: Int = 1): Flow<NetworkResult<Post>> {
        val post = Post(id = 0, userId = userId, title = title, body = body)
        return repository.createPost(post)
    }
}
```

**Resultado esperado:** Tres casos de uso inyectables que encapsulan las operaciones de obtener posts, obtener detalle con comentarios y crear un post.

**Verificación:** Compila el proyecto exitosamente.

---

### Paso 13: Implementar el ViewModel con StateFlow

**Objetivo:** Crear el ViewModel que gestiona el estado de la UI reactivamente, consumiendo los Use Cases y exponiendo `StateFlow` a la capa de presentación.

**Instrucciones:**

1. Crea `PostsUiState.kt` en `ui/screen/`:

```kotlin
package com.cursoadv.android.red.ui.screen

import com.cursoadv.android.red.domain.model.Comment
import com.cursoadv.android.red.domain.model.Post

data class PostsUiState(
    val isLoading: Boolean = false,
    val posts: List<Post> = emptyList(),
    val errorMessage: String? = null,
    val selectedPost: Post? = null,
    val comments: List<Comment> = emptyList(),
    val isCreatingPost: Boolean = false,
    val createPostSuccess: Boolean = false,
    val createPostError: String? = null
)
```

2. Crea `PostsViewModel.kt` en `ui/screen/`:

```kotlin
package com.cursoadv.android.red.ui.screen

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.cursoadv.android.red.domain.usecase.CreatePostUseCase
import com.cursoadv.android.red.domain.usecase.GetPostDetailUseCase
import com.cursoadv.android.red.domain.usecase.GetPostsUseCase
import com.cursoadv.android.red.util.NetworkResult
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class PostsViewModel @Inject constructor(
    private val getPostsUseCase: GetPostsUseCase,
    private val getPostDetailUseCase: GetPostDetailUseCase,
    private val createPostUseCase: CreatePostUseCase
) : ViewModel() {

    private val _uiState = MutableStateFlow(PostsUiState())
    val uiState: StateFlow<PostsUiState> = _uiState.asStateFlow()

    init {
        loadPosts()
    }

    fun loadPosts() {
        viewModelScope.launch {
            getPostsUseCase().collect { result ->
                _uiState.update { currentState ->
                    when (result) {
                        is NetworkResult.Loading -> currentState.copy(
                            isLoading = true,
                            errorMessage = null
                        )
                        is NetworkResult.Success -> currentState.copy(
                            isLoading = false,
                            posts = result.data,
                            errorMessage = null
                        )
                        is NetworkResult.Error -> currentState.copy(
                            isLoading = false,
                            errorMessage = result.message
                        )
                    }
                }
            }
        }
    }

    fun loadPostDetail(postId: Int) {
        viewModelScope.launch {
            getPostDetailUseCase(postId).collect { result ->
                _uiState.update { currentState ->
                    when (result) {
                        is NetworkResult.Loading -> currentState.copy(
                            isLoading = true,
                            errorMessage = null
                        )
                        is NetworkResult.Success -> currentState.copy(
                            isLoading = false,
                            selectedPost = result.data.post,
                            comments = result.data.comments,
                            errorMessage = null
                        )
                        is NetworkResult.Error -> currentState.copy(
                            isLoading = false,
                            errorMessage = result.message
                        )
                    }
                }
            }
        }
    }

    fun createPost(title: String, body: String) {
        viewModelScope.launch {
            createPostUseCase(title = title, body = body).collect { result ->
                _uiState.update { currentState ->
                    when (result) {
                        is NetworkResult.Loading -> currentState.copy(
                            isCreatingPost = true,
                            createPostSuccess = false,
                            createPostError = null
                        )
                        is NetworkResult.Success -> currentState.copy(
                            isCreatingPost = false,
                            createPostSuccess = true,
                            createPostError = null
                        )
                        is NetworkResult.Error -> currentState.copy(
                            isCreatingPost = false,
                            createPostSuccess = false,
                            createPostError = result.message
                        )
                    }
                }
            }
        }
    }

    fun clearSelectedPost() {
        _uiState.update {
            it.copy(selectedPost = null, comments = emptyList())
        }
    }

    fun clearCreatePostState() {
        _uiState.update {
            it.copy(
                createPostSuccess = false,
                createPostError = null
            )
        }
    }

    fun dismissError() {
        _uiState.update {
            it.copy(errorMessage = null)
        }
    }
}
```

**Resultado esperado:** Un ViewModel con Hilt que gestiona tres operaciones de red, exponiendo un único `StateFlow<PostsUiState>` inmutable a la UI.

**Verificación:** Compila el proyecto exitosamente.

---

### Paso 14: Construir la Interfaz de Usuario con Jetpack Compose

**Objetivo:** Crear las pantallas de Compose que consumen el `StateFlow` del ViewModel y muestran los estados Loading, Success y Error de forma reactiva.

**Instrucciones:**

1. Crea `PostListScreen.kt` en `ui/screen/`:

```kotlin
package com.cursoadv.android.red.ui.screen

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.PaddingValues
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.Refresh
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.CircularProgressIndicator
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
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import com.cursoadv.android.red.domain.model.Post

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PostListScreen(
    viewModel: PostsViewModel,
    onPostClick: (Int) -> Unit,
    onCreatePostClick: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()
    val snackbarHostState = remember { SnackbarHostState() }

    LaunchedEffect(uiState.errorMessage) {
        uiState.errorMessage?.let { message ->
            snackbarHostState.showSnackbar(message)
            viewModel.dismissError()
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Posts - JSONPlaceholder") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                ),
                actions = {
                    IconButton(onClick = { viewModel.loadPosts() }) {
                        Icon(
                            imageVector = Icons.Default.Refresh,
                            contentDescription = "Recargar"
                        )
                    }
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = onCreatePostClick) {
                Icon(
                    imageVector = Icons.Default.Add,
                    contentDescription = "Crear Post"
                )
            }
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            when {
                uiState.isLoading && uiState.posts.isEmpty() -> {
                    CircularProgressIndicator(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                uiState.posts.isNotEmpty() -> {
                    LazyColumn(
                        contentPadding = PaddingValues(16.dp),
                        verticalArrangement = Arrangement.spacedBy(12.dp)
                    ) {
                        items(
                            items = uiState.posts,
                            key = { it.id }
                        ) { post ->
                            PostCard(
                                post = post,
                                onClick = { onPostClick(post.id) }
                            )
                        }
                    }
                }
                !uiState.isLoading && uiState.posts.isEmpty() && uiState.errorMessage == null -> {
                    Text(
                        text = "No hay posts disponibles",
                        modifier = Modifier.align(Alignment.Center),
                        style = MaterialTheme.typography.bodyLarge
                    )
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
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface
        )
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text(
                    text = "#${post.id}",
                    style = MaterialTheme.typography.labelMedium,
                    color = MaterialTheme.colorScheme.primary
                )
                Text(
                    text = "Usuario ${post.userId}",
                    style = MaterialTheme.typography.labelSmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
            Spacer(modifier = Modifier.height(8.dp))
            Text(
                text = post.title.replaceFirstChar { it.uppercase() },
                style = MaterialTheme.typography.titleMedium,
                fontWeight = FontWeight.SemiBold,
                maxLines = 2,
                overflow = TextOverflow.Ellipsis
            )
            Spacer(modifier = Modifier.height(4.dp))
            Text(
                text = post.body,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                maxLines = 3,
                overflow = TextOverflow.Ellipsis
            )
        }
    }
}
```

2. Crea `PostDetailScreen.kt` en `ui/screen/`:

```kotlin
package com.cursoadv.android.red.ui.screen

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.PaddingValues
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
import androidx.compose.material3.HorizontalDivider
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.material3.TopAppBarDefaults
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import com.cursoadv.android.red.domain.model.Comment

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PostDetailScreen(
    viewModel: PostsViewModel,
    postId: Int,
    onBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()

    LaunchedEffect(postId) {
        viewModel.loadPostDetail(postId)
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Detalle del Post") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                ),
                navigationIcon = {
                    IconButton(onClick = {
                        viewModel.clearSelectedPost()
                        onBack()
                    }) {
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
            when {
                uiState.isLoading -> {
                    CircularProgressIndicator(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                uiState.errorMessage != null -> {
                    Text(
                        text = "Error: ${uiState.errorMessage}",
                        modifier = Modifier
                            .align(Alignment.Center)
                            .padding(16.dp),
                        color = MaterialTheme.colorScheme.error,
                        style = MaterialTheme.typography.bodyLarge
                    )
                }
                uiState.selectedPost != null -> {
                    LazyColumn(
                        contentPadding = PaddingValues(16.dp),
                        verticalArrangement = Arrangement.spacedBy(12.dp)
                    ) {
                        item {
                            Column {
                                Text(
                                    text = uiState.selectedPost!!.title
                                        .replaceFirstChar { it.uppercase() },
                                    style = MaterialTheme.typography.headlineSmall,
                                    fontWeight = FontWeight.Bold
                                )
                                Spacer(modifier = Modifier.height(4.dp))
                                Text(
                                    text = "Por Usuario ${uiState.selectedPost!!.userId}",
                                    style = MaterialTheme.typography.labelLarge,
                                    color = MaterialTheme.colorScheme.primary
                                )
                                Spacer(modifier = Modifier.height(12.dp))
                                Text(
                                    text = uiState.selectedPost!!.body,
                                    style = MaterialTheme.typography.bodyLarge
                                )
                            }
                        }

                        if (uiState.comments.isNotEmpty()) {
                            item {
                                HorizontalDivider(modifier = Modifier.padding(vertical = 8.dp))
                                Text(
                                    text = "Comentarios (${uiState.comments.size})",
                                    style = MaterialTheme.typography.titleMedium,
                                    fontWeight = FontWeight.SemiBold
                                )
                            }

                            items(
                                items = uiState.comments,
                                key = { it.id }
                            ) { comment ->
                                CommentCard(comment = comment)
                            }
                        }
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
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceVariant
        )
    ) {
        Column(modifier = Modifier.padding(12.dp)) {
            Text(
                text = comment.name.replaceFirstChar { it.uppercase() },
                style = MaterialTheme.typography.titleSmall,
                fontWeight = FontWeight.Medium
            )
            Text(
                text = comment.email,
                style = MaterialTheme.typography.labelSmall,
                color = MaterialTheme.colorScheme.primary
            )
            Spacer(modifier = Modifier.height(6.dp))
            Text(
                text = comment.body,
                style = MaterialTheme.typography.bodySmall
            )
        }
    }
}
```

3. Crea `CreatePostScreen.kt` en `ui/screen/`:

```kotlin
package com.cursoadv.android.red.ui.screen

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.Button
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
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CreatePostScreen(
    viewModel: PostsViewModel,
    onBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()
    var title by remember { mutableStateOf("") }
    var body by remember { mutableStateOf("") }
    val snackbarHostState = remember { SnackbarHostState() }

    LaunchedEffect(uiState.createPostSuccess) {
        if (uiState.createPostSuccess) {
            snackbarHostState.showSnackbar("¡Post creado exitosamente!")
            viewModel.clearCreatePostState()
            title = ""
            body = ""
        }
    }

    LaunchedEffect(uiState.createPostError) {
        uiState.createPostError?.let { error ->
            snackbarHostState.showSnackbar("Error: $error")
            viewModel.clearCreatePostState()
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Crear Nuevo Post") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                ),
                navigationIcon = {
                    IconButton(onClick = onBack) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
                        )
                    }
                }
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            OutlinedTextField(
                value = title,
                onValueChange = { title = it },
                label = { Text("Título del post") },
                modifier = Modifier.fillMaxWidth(),
                singleLine = true,
                enabled = !uiState.isCreatingPost
            )

            Spacer(modifier = Modifier.height(16.dp))

            OutlinedTextField(
                value = body,
                onValueChange = { body = it },
                label = { Text("Contenido del post") },
                modifier = Modifier
                    .fillMaxWidth()
                    .height(200.dp),
                maxLines = 8,
                enabled = !uiState.isCreatingPost
            )

            Spacer(modifier = Modifier.height(24.dp))

            Button(
                onClick = {
                    if (title.isNotBlank() && body.isNotBlank()) {
                        viewModel.createPost(title = title, body = body)
                    }
                },
                modifier = Modifier.fillMaxWidth(),
                enabled = title.isNotBlank() && body.isNotBlank() && !uiState.isCreatingPost
            ) {
                if (uiState.isCreatingPost) {
                    CircularProgressIndicator(
                        color = MaterialTheme.colorScheme.onPrimary
                    )
                } else {
                    Text("Enviar Post (POST)")
                }
            }

            Spacer(modifier = Modifier.height(8.dp))

            Text(
                text = "Se enviará un POST a jsonplaceholder.typicode.com/posts",
                style = MaterialTheme.typography.labelSmall,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}
```

**Resultado esperado:** Tres pantallas Compose completas que reaccionan a los estados de red del ViewModel.

**Verificación:** Compila el proyecto exitosamente.

---

### Paso 15: Configurar la Navegación y Conectar la UI con MainActivity

**Objetivo:** Implementar la navegación entre pantallas y conectar todo el flujo en `MainActivity`.

**Instrucciones:**

1. Crea `AppNavigation.kt` en `ui/`:

```kotlin
package com.cursoadv.android.red.ui

import androidx.compose.runtime.Composable
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument
import com.cursoadv.android.red.ui.screen.CreatePostScreen
import com.cursoadv.android.red.ui.screen.PostDetailScreen
import com.cursoadv.android.red.ui.screen.PostListScreen
import com.cursoadv.android.red.ui.screen.PostsViewModel

sealed class Screen(val route: String) {
    data object PostList : Screen("post_list")
    data object PostDetail : Screen("post_detail/{postId}") {
        fun createRoute(postId: Int) = "post_detail/$postId"
    }
    data object CreatePost : Screen("create_post")
}

@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    // ViewModel compartido a nivel de navegación
    val viewModel: PostsViewModel = hiltViewModel()

    NavHost(
        navController = navController,
        startDestination = Screen.PostList.route
    ) {
        composable(Screen.PostList.route) {
            PostListScreen(
                viewModel = viewModel,
                onPostClick = { postId ->
                    navController.navigate(Screen.PostDetail.createRoute(postId))
                },
                onCreatePostClick = {
                    navController.navigate(Screen.CreatePost.route)
                }
            )
        }

        composable(
            route = Screen.PostDetail.route,
            arguments = listOf(
                navArgument("postId") { type = NavType.IntType }
            )
        ) { backStackEntry ->
            val postId = backStackEntry.arguments?.getInt("postId") ?: 1
            PostDetailScreen(
                viewModel = viewModel,
                postId = postId,
                onBack = { navController.popBackStack() }
            )
        }

        composable(Screen.CreatePost.route) {
            CreatePostScreen(
                viewModel = viewModel,
                onBack = { navController.popBackStack() }
            )
        }
    }
}
```

2. Actualiza `MainActivity.kt` para usar la navegación:

```kotlin
package com.cursoadv.android.red

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.cursoadv.android.red.ui.AppNavigation
import com.cursoadv.android.red.ui.theme.Practica4RedTheme
import dagger.hilt.android.AndroidEntryPoint

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Practica4RedTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    AppNavigation()
                }
            }
        }
    }
}
```

> **Nota:** Ajusta `Practica4RedTheme` al nombre exacto del tema generado por Android Studio en `ui/theme/Theme.kt`.

3. Compila y ejecuta la aplicación en el emulador o dispositivo físico.

**Resultado esperado:** La aplicación se lanza, muestra un indicador de carga circular y luego presenta la lista de 100 posts obtenidos de JSONPlaceholder.

**Verificación:**

- [ ] La pantalla principal muestra una lista scrollable de posts con título y cuerpo
- [ ] Al tocar un post, se navega a la pantalla de detalle con comentarios
- [ ] Al tocar el FAB (+), se navega a la pantalla de creación de posts
- [ ] Al enviar un post, aparece un Snackbar confirmando la creación exitosa

---

### Paso 16: Verificar los Interceptores y Logs de Red

**Objetivo:** Confirmar que los interceptores de autenticación y logging están funcionando correctamente revisando el Logcat.

**Instrucciones:**

1. Con la aplicación ejecutándose, abre la ventana **Logcat** en Android Studio.

2. En el filtro de Logcat, escribe `okhttp` para filtrar los logs del interceptor.

3. Observa que cada petición HTTP incluye:
   - El header `Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.demo-token`
   - El header `Content-Type: application/json; charset=UTF-8`
   - El cuerpo completo de la respuesta JSON (nivel BODY)

4. Navega a la pantalla de crear post, ingresa un título y cuerpo, y envía. Verifica en Logcat que:
   - Se registra una petición `POST` a `https://jsonplaceholder.typicode.com/posts`
   - El cuerpo de la petición contiene el JSON serializado con `userId`, `title` y `body`
   - El header `X-Custom-Api-Key` aparece con el valor `practica4-demo-api-key-2025`
   - La respuesta devuelve un código `201 Created`

**Resultado esperado en Logcat (fragmento):**

```
--> POST https://jsonplaceholder.typicode.com/posts
Content-Type: application/json; charset=UTF-8
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.demo-token
X-Custom-Api-Key: practica4-demo-api-key-2025
Content-Length: 67

{"userId":1,"title":"Mi primer post","body":"Contenido de prueba"}
--> END POST (67-byte body)

<-- 201 Created https://jsonplaceholder.typicode.com/posts (245ms)
content-type: application/json; charset=utf-8

{"userId":1,"title":"Mi primer post","body":"Contenido de prueba","id":101}
<-- END HTTP (78-byte body)
```

**Verificación:** Los logs muestran claramente los headers inyectados por el `AuthInterceptor`, el API Key por petición y el cuerpo JSON serializado/deserializado correctamente.

---

### Paso 17: Probar el Manejo de Errores de Red

**Objetivo:** Verificar que la aplicación maneja correctamente los errores de red mostrando mensajes apropiados al usuario.

**Instrucciones:**

1. **Simular error de conexión:** Activa el **Modo Avión** en el emulador (desliza desde la barra de estado o usa Settings → Network).

2. Toca el botón de **Recargar** (ícono de refresh) en la barra superior de la lista de posts.

3. Observa que:
   - Se muestra brevemente el indicador de carga
   - Aparece un Snackbar con el mensaje: "Error de conexión. Verifica tu acceso a Internet."

4. **Simular error HTTP:** Desactiva el Modo Avión. Temporalmente, para probar un error 404, modifica la URL base en `build.gradle.kts` a una ruta inválida:

```kotlin
buildConfigField("String", "BASE_URL", "\"https://jsonplaceholder.typicode.com/invalid/\"")
```

5. Sincroniza Gradle, ejecuta la app y observa que se muestra un error HTTP en el Snackbar.

6. **Restaura** la URL correcta después de la prueba:

```kotlin
buildConfigField("String", "BASE_URL", "\"https://jsonplaceholder.typicode.com/\"")
```

7. Sincroniza y recompila para dejar la app funcional.

**Resultado esperado:** La aplicación nunca se cierra inesperadamente (no crash). Todos los errores se capturan y se muestran como mensajes amigables al usuario.

**Verificación:**

- [ ] Error de conexión muestra mensaje descriptivo en Snackbar
- [ ] Error HTTP muestra código y mensaje en Snackbar
- [ ] La aplicación permanece estable y permite reintentar

---

## 7. Validación y Pruebas

### Lista de Verificación Final

Ejecuta cada una de estas verificaciones para confirmar que la práctica está completa:

| # | Verificación | Estado |
|---|---|---|
| 1 | El proyecto compila sin errores ni warnings críticos | ☐ |
| 2 | La lista de posts se carga desde `jsonplaceholder.typicode.com` y muestra 100 posts | ☐ |
| 3 | Al tocar un post, se muestra su detalle con título, cuerpo y lista de comentarios | ☐ |
| 4 | La pantalla de crear post envía un POST y muestra confirmación exitosa | ☐ |
| 5 | El Logcat muestra los headers `Authorization` y `Content-Type` en cada petición | ☐ |
| 6 | El Logcat muestra el header `X-Custom-Api-Key` en la petición POST | ☐ |
| 7 | El logging muestra nivel BODY con cuerpo de request y response completos | ☐ |
| 8 | Con Modo Avión activado, se muestra un error descriptivo sin crash | ☐ |
| 9 | El indicador `CircularProgressIndicator` aparece durante la carga | ☐ |
| 10 | La navegación entre las tres pantallas funciona correctamente | ☐ |

### Estructura Final del Proyecto

Verifica que la estructura de archivos coincide con:

```
app/src/main/java/com/cursoadv/android/red/
├── RedApp.kt
├── MainActivity.kt
├── data/
│   ├── model/
│   │   ├── PostDto.kt
│   │   ├── CreatePostDto.kt
│   │   ├── CommentDto.kt
│   │   └── PostMapper.kt
│   ├── network/
│   │   ├── JsonPlaceholderApi.kt
│   │   └── AuthInterceptor.kt
│   └── repository/
│       └── PostRepositoryImpl.kt
├── di/
│   ├── NetworkModule.kt
│   └── RepositoryModule.kt
├── domain/
│   ├── PostRepository.kt
│   ├── model/
│   │   ├── Post.kt
│   │   └── Comment.kt
│   └── usecase/
│       ├── GetPostsUseCase.kt
│       ├── GetPostDetailUseCase.kt
│       └── CreatePostUseCase.kt
├── ui/
│   ├── AppNavigation.kt
│   ├── screen/
│   │   ├── PostsUiState.kt
│   │   ├── PostsViewModel.kt
│   │   ├── PostListScreen.kt
│   │   ├── PostDetailScreen.kt
│   │   └── CreatePostScreen.kt
│   └── theme/
│       └── (archivos generados por Android Studio)
└── util/
    ├── NetworkResult.kt
    └── SafeApiCall.kt
```

---

## 8. Solución de Problemas

### Problema 1: Error de compilación `Unresolved reference: BuildConfig`

**Síntomas:** Al compilar el proyecto, aparece el error:

```
Unresolved reference: BuildConfig
```

en `PostRepositoryImpl.kt` o `NetworkModule.kt`.

**Causa:** La característica `buildConfig` no está habilitada en el bloque `buildFeatures` del archivo `build.gradle.kts` del módulo `app`, o la clase `BuildConfig` aún no se ha generado tras agregar los campos `buildConfigField`.

**Solución:**

1. Verifica que `app/build.gradle.kts` contiene:

```kotlin
buildFeatures {
    compose = true
    buildConfig = true
}
```

2. Ejecuta **Build → Clean Project** seguido de **Build → Rebuild Project**.

3. Si persiste, ejecuta desde terminal:

```bash
cd ~/AndroidStudioProjects/Avanzado
