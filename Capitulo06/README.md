# Práctica 6 — SensorMapApp: Geolocalización, Mapas y Sensores del Dispositivo

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 144 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Aplicar |
| **Proyecto** | `SensorMapApp` |
| **Paquete base** | `com.curso.android.avanzado.practica6` |
| **Directorio** | `~/AndroidStudioProjects/AvanzadoKotlin/Practica6_SensorMap` |

---

## 2. Descripción General

En esta práctica construirás **SensorMapApp**, una aplicación Android completa que integra tres capacidades fundamentales del dispositivo: permisos de ubicación en tiempo de ejecución, obtención de coordenadas GPS mediante `FusedLocationProviderClient`, visualización sobre Google Maps con Maps Compose, y lectura en tiempo real de sensores físicos (acelerómetro y sensor de luz). La aplicación utilizará una arquitectura basada en `ViewModel` + `StateFlow` + Jetpack Compose con navegación por `BottomNavigationBar` entre tres pantallas: **Permisos**, **Mapa** y **Sensores**.

---

## 3. Objetivos de Aprendizaje

Al completar esta práctica serás capaz de:

- [ ] Implementar el flujo completo de solicitud de permisos sensibles en Android 12+ usando `ActivityResultContracts.RequestMultiplePermissions`, manejando los estados concedido, denegado y denegado permanentemente con redirección a Settings.
- [ ] Obtener la ubicación actual del dispositivo mediante `FusedLocationProviderClient` usando `callbackFlow` y exponer actualizaciones continuas como `StateFlow` desde un `ViewModel`.
- [ ] Visualizar coordenadas GPS sobre un mapa interactivo con Maps Compose 6.4.1, incluyendo marcadores, ventanas de información y control programático de la cámara.
- [ ] Leer datos en tiempo real de los sensores `TYPE_ACCELEROMETER` y `TYPE_LIGHT` mediante `SensorManager`, procesando sus valores con `StateFlow` y `collectAsStateWithLifecycle()`.
- [ ] Gestionar correctamente el ciclo de vida de listeners de sensores y callbacks de ubicación usando `DisposableEffect` para evitar fugas de memoria.

---

## 4. Prerrequisitos

### Conocimientos requeridos

| Tema | Nivel |
|---|---|
| Jetpack Compose (Composables, State, `remember`, `LaunchedEffect`) | Intermedio |
| Corrutinas de Kotlin (`launch`, `collect`, `Flow`) | Intermedio |
| Patrón MVVM con `ViewModel` y `StateFlow` | Intermedio |
| Sistema de permisos de Android (Manifest + runtime) | Básico |
| Prácticas 1–5 del curso completadas | Obligatorio |

### Acceso y configuración previa

- Android Studio Quail 3 (2026.1.3 Patch 1) instalado y funcional.
- SDK Platforms API 30, 35, 36 y 37 descargados.
- AVDs creados para API 30 y 37 con imágenes **Google Play** (necesario para Play Services).
- **API Key de Google Maps** generada desde [Google Cloud Console](https://console.cloud.google.com/) con el servicio "Maps SDK for Android" habilitado.
- Conexión a Internet estable para descargar dependencias de Gradle.

---

## 5. Entorno del Laboratorio

### Hardware mínimo

| Componente | Requisito |
|---|---|
| Procesador | Intel Core i7 8ª gen+ / AMD Ryzen 7+ / Apple M1+ |
| RAM | 16 GB mínimo (32 GB recomendado) |
| Almacenamiento | 60 GB libres en SSD |
| Virtualización | VT-x / AMD-V / Apple Hypervisor habilitado en BIOS |

### Software

| Herramienta | Versión exacta |
|---|---|
| Android Studio | Quail 3 — 2026.1.3 Patch 1 |
| Kotlin | 2.2.10 |
| AGP | 9.3.2 |
| Gradle Wrapper | 8.14.1 |
| KSP | 2.2.10-1.0.31 |
| Compose BOM | 2026.02.01 |
| Google Play Services Location | 21.4.0 |
| Maps Compose | 6.4.1 |
| Core KTX | 1.19.0 |
| Activity Compose | 1.13.0 |
| Lifecycle Runtime KTX | 2.6.1 |

---

## 6. Instrucciones Paso a Paso

### Paso 1 — Crear el proyecto y configurar el catálogo de versiones

**Objetivo:** Crear el proyecto `SensorMapApp` desde la plantilla Empty Activity (Compose) y establecer todas las dependencias con versiones exactas en `libs.versions.toml`.

**Instrucciones:**

1. Abre Android Studio y selecciona **File → New → New Project**.
2. Elige la plantilla **Empty Activity** (Jetpack Compose).
3. Configura los siguientes parámetros:
   - **Name:** `SensorMapApp`
   - **Package name:** `com.curso.android.avanzado.practica6`
   - **Save location:** `~/AndroidStudioProjects/AvanzadoKotlin/Practica6_SensorMap`
   - **Minimum SDK:** API 30 (Android 11)
   - **Build configuration language:** Kotlin DSL
4. Haz clic en **Finish** y espera a que Gradle sincronice.

5. Abre el archivo `gradle/libs.versions.toml` y reemplaza su contenido completo con:

```toml
[versions]
agp = "9.3.2"
kotlin = "2.2.10"
ksp = "2.2.10-1.0.31"
composeBom = "2026.02.01"
activityCompose = "1.13.0"
coreKtx = "1.19.0"
lifecycleRuntimeKtx = "2.6.1"
playServicesLocation = "21.4.0"
mapsCompose = "6.4.1"
playServicesMaps = "19.1.0"
junit = "4.13.2"
androidxJunit = "1.3.0"
espressoCore = "3.7.0"
navigationCompose = "2.9.0"

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
play-services-location = { group = "com.google.android.gms", name = "play-services-location", version.ref = "playServicesLocation" }
play-services-maps = { group = "com.google.android.gms", name = "play-services-maps", version.ref = "playServicesMaps" }
maps-compose = { group = "com.google.maps.android", name = "maps-compose", version.ref = "mapsCompose" }
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidxJunit" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

6. Abre el archivo `build.gradle.kts` **del proyecto raíz** y verifica que contenga:

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
}
```

7. Abre el archivo `build.gradle.kts` **del módulo `app`** y configúralo así:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.curso.android.avanzado.practica6"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.curso.android.avanzado.practica6"
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
        buildConfig = true
    }
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.runtime.compose)
    implementation(libs.androidx.lifecycle.viewmodel.compose)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.material.icons.extended)
    implementation(libs.androidx.navigation.compose)

    // Google Play Services Location
    implementation(libs.play.services.location)

    // Google Maps
    implementation(libs.play.services.maps)
    implementation(libs.maps.compose)

    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    debugImplementation(libs.androidx.ui.tooling)
}
```

8. Verifica que `gradle/wrapper/gradle-wrapper.properties` contenga:

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.14.1-bin.zip
```

9. Sincroniza Gradle: **File → Sync Project with Gradle Files**.

**Resultado esperado:** El proyecto compila sin errores. La ventana Build muestra `BUILD SUCCESSFUL`.

**Verificación:**

```
Build > Rebuild Project → BUILD SUCCESSFUL en la consola de Build Output
```

---

### Paso 2 — Configurar la API Key de Google Maps de forma segura

**Objetivo:** Almacenar la API Key de Google Maps en `local.properties` y referenciarla desde `AndroidManifest.xml` sin exponerla en el repositorio.

**Instrucciones:**

1. Abre el archivo `local.properties` (ubicado en la raíz del proyecto, **ya incluido en `.gitignore`**) y agrega al final:

```properties
MAPS_API_KEY=TU_API_KEY_REAL_AQUI
```

> ⚠️ **Reemplaza `TU_API_KEY_REAL_AQUI`** con tu clave real obtenida de Google Cloud Console.

2. En el archivo `build.gradle.kts` del módulo `app`, dentro del bloque `defaultConfig`, agrega la lectura de la clave. Modifica el bloque `defaultConfig` para que quede así:

```kotlin
defaultConfig {
    applicationId = "com.curso.android.avanzado.practica6"
    minSdk = 30
    targetSdk = 37
    versionCode = 1
    versionName = "1.0"

    testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"

    // Leer API Key desde local.properties
    val localProperties = java.util.Properties()
    val localPropertiesFile = rootProject.file("local.properties")
    if (localPropertiesFile.exists()) {
        localProperties.load(localPropertiesFile.inputStream())
    }
    val mapsApiKey = localProperties.getProperty("MAPS_API_KEY") ?: ""
    manifestPlaceholders["MAPS_API_KEY"] = mapsApiKey
    buildConfigField("String", "MAPS_API_KEY", "\"$mapsApiKey\"")
}
```

3. Abre `app/src/main/AndroidManifest.xml` y configúralo con los permisos y la metadata de Maps:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permisos de ubicación -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <!-- Permiso opcional para sensores corporales -->
    <uses-permission android:name="android.permission.BODY_SENSORS" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.SensorMapApp">

        <!-- Google Maps API Key -->
        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="${MAPS_API_KEY}" />

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.SensorMapApp">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

4. Sincroniza Gradle nuevamente.

**Resultado esperado:** La compilación es exitosa. La API Key no aparece en ningún archivo versionado.

**Verificación:** Ejecuta `Build > Rebuild Project`. Verifica que `BuildConfig.MAPS_API_KEY` esté disponible en el código (se validará en pasos posteriores).

---

### Paso 3 — Crear la estructura de paquetes y modelos de datos

**Objetivo:** Establecer la organización del código fuente y definir las clases de estado (`sealed class`) que se usarán en toda la aplicación.

**Instrucciones:**

1. Dentro de `com.curso.android.avanzado.practica6`, crea los siguientes paquetes (clic derecho sobre el paquete → New → Package):

```
com.curso.android.avanzado.practica6
├── data
│   ├── location
│   └── sensor
├── ui
│   ├── navigation
│   ├── permissions
│   ├── map
│   └── sensors
└── viewmodel
```

2. Crea el archivo `data/location/LocationUiState.kt`:

```kotlin
package com.curso.android.avanzado.practica6.data.location

sealed class LocationUiState {
    data object Loading : LocationUiState()
    data class Success(
        val latitude: Double,
        val longitude: Double,
        val accuracy: Float
    ) : LocationUiState()
    data class Error(val message: String) : LocationUiState()
}
```

3. Crea el archivo `data/sensor/SensorData.kt`:

```kotlin
package com.curso.android.avanzado.practica6.data.sensor

data class AccelerometerData(
    val x: Float = 0f,
    val y: Float = 0f,
    val z: Float = 0f
) {
    val magnitude: Float
        get() = kotlin.math.sqrt(x * x + y * y + z * z)
}
```

4. Crea el archivo `data/location/PermissionState.kt`:

```kotlin
package com.curso.android.avanzado.practica6.data.location

data class PermissionState(
    val fineLocationGranted: Boolean = false,
    val coarseLocationGranted: Boolean = false,
    val permanentlyDenied: Boolean = false,
    val shouldShowRationale: Boolean = false
) {
    val anyLocationGranted: Boolean
        get() = fineLocationGranted || coarseLocationGranted
}
```

**Resultado esperado:** La estructura de paquetes está creada y los tres archivos de modelo compilan sin errores.

**Verificación:** `Build > Make Module 'app'` → BUILD SUCCESSFUL.

---

### Paso 4 — Implementar la pantalla de permisos

**Objetivo:** Crear el flujo completo de solicitud de permisos de ubicación con manejo de los tres estados: concedido, denegado con rationale y denegado permanentemente.

**Instrucciones:**

1. Crea el archivo `viewmodel/PermissionViewModel.kt`:

```kotlin
package com.curso.android.avanzado.practica6.viewmodel

import androidx.lifecycle.ViewModel
import com.curso.android.avanzado.practica6.data.location.PermissionState
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update

class PermissionViewModel : ViewModel() {

    private val _permissionState = MutableStateFlow(PermissionState())
    val permissionState: StateFlow<PermissionState> = _permissionState.asStateFlow()

    fun updatePermissionResult(permissions: Map<String, Boolean>) {
        _permissionState.update { current ->
            current.copy(
                fineLocationGranted = permissions[
                    android.Manifest.permission.ACCESS_FINE_LOCATION
                ] ?: current.fineLocationGranted,
                coarseLocationGranted = permissions[
                    android.Manifest.permission.ACCESS_COARSE_LOCATION
                ] ?: current.coarseLocationGranted
            )
        }
    }

    fun setShouldShowRationale(shouldShow: Boolean) {
        _permissionState.update { it.copy(shouldShowRationale = shouldShow) }
    }

    fun setPermanentlyDenied(denied: Boolean) {
        _permissionState.update { it.copy(permanentlyDenied = denied) }
    }
}
```

2. Crea el archivo `ui/permissions/PermissionScreen.kt`:

```kotlin
package com.curso.android.avanzado.practica6.ui.permissions

import android.Manifest
import android.content.Intent
import android.net.Uri
import android.provider.Settings
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.CheckCircle
import androidx.compose.material.icons.filled.LocationOn
import androidx.compose.material.icons.filled.Warning
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedButton
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.core.app.ActivityCompat
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.curso.android.avanzado.practica6.data.location.PermissionState
import com.curso.android.avanzado.practica6.viewmodel.PermissionViewModel

@Composable
fun PermissionScreen(
    permissionViewModel: PermissionViewModel
) {
    val permissionState by permissionViewModel.permissionState
        .collectAsStateWithLifecycle()
    val context = LocalContext.current
    var showRationaleDialog by remember { mutableStateOf(false) }

    val locationPermissions = arrayOf(
        Manifest.permission.ACCESS_FINE_LOCATION,
        Manifest.permission.ACCESS_COARSE_LOCATION
    )

    val permissionLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        permissionViewModel.updatePermissionResult(permissions)

        val allDenied = permissions.values.none { it }
        if (allDenied) {
            // Verificar si el usuario denegó permanentemente
            val activity = context as? android.app.Activity
            if (activity != null) {
                val shouldShowFine = ActivityCompat
                    .shouldShowRequestPermissionRationale(
                        activity,
                        Manifest.permission.ACCESS_FINE_LOCATION
                    )
                val shouldShowCoarse = ActivityCompat
                    .shouldShowRequestPermissionRationale(
                        activity,
                        Manifest.permission.ACCESS_COARSE_LOCATION
                    )
                if (!shouldShowFine && !shouldShowCoarse) {
                    // Denegado permanentemente
                    permissionViewModel.setPermanentlyDenied(true)
                } else {
                    // Denegado pero se puede mostrar rationale
                    permissionViewModel.setShouldShowRationale(true)
                    showRationaleDialog = true
                }
            }
        }
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Icon(
            imageVector = Icons.Default.LocationOn,
            contentDescription = "Ubicación",
            modifier = Modifier.size(80.dp),
            tint = if (permissionState.anyLocationGranted)
                MaterialTheme.colorScheme.primary
            else
                MaterialTheme.colorScheme.outline
        )

        Spacer(modifier = Modifier.height(24.dp))

        Text(
            text = "Permisos de Ubicación",
            style = MaterialTheme.typography.headlineMedium,
            fontWeight = FontWeight.Bold
        )

        Spacer(modifier = Modifier.height(16.dp))

        // Tarjeta de estado de permisos
        PermissionStatusCard(permissionState)

        Spacer(modifier = Modifier.height(24.dp))

        when {
            permissionState.anyLocationGranted -> {
                Icon(
                    imageVector = Icons.Default.CheckCircle,
                    contentDescription = "Concedido",
                    tint = MaterialTheme.colorScheme.primary,
                    modifier = Modifier.size(48.dp)
                )
                Spacer(modifier = Modifier.height(8.dp))
                Text(
                    text = "¡Permisos concedidos! Puedes usar el Mapa y la Ubicación.",
                    textAlign = TextAlign.Center,
                    style = MaterialTheme.typography.bodyLarge
                )
            }

            permissionState.permanentlyDenied -> {
                Icon(
                    imageVector = Icons.Default.Warning,
                    contentDescription = "Denegado",
                    tint = MaterialTheme.colorScheme.error,
                    modifier = Modifier.size(48.dp)
                )
                Spacer(modifier = Modifier.height(8.dp))
                Text(
                    text = "Los permisos fueron denegados permanentemente. " +
                            "Debes habilitarlos desde la configuración de la app.",
                    textAlign = TextAlign.Center,
                    style = MaterialTheme.typography.bodyLarge,
                    color = MaterialTheme.colorScheme.error
                )
                Spacer(modifier = Modifier.height(16.dp))
                Button(
                    onClick = {
                        val intent = Intent(
                            Settings.ACTION_APPLICATION_DETAILS_SETTINGS,
                            Uri.fromParts("package", context.packageName, null)
                        )
                        context.startActivity(intent)
                    },
                    colors = ButtonDefaults.buttonColors(
                        containerColor = MaterialTheme.colorScheme.error
                    )
                ) {
                    Text("Abrir Configuración")
                }
            }

            else -> {
                Text(
                    text = "Esta aplicación necesita acceso a tu ubicación " +
                            "para mostrar tu posición en el mapa.",
                    textAlign = TextAlign.Center,
                    style = MaterialTheme.typography.bodyLarge
                )
                Spacer(modifier = Modifier.height(16.dp))
                Button(
                    onClick = { permissionLauncher.launch(locationPermissions) },
                    modifier = Modifier.fillMaxWidth(0.7f)
                ) {
                    Text("Solicitar Permisos")
                }
            }
        }
    }

    // Diálogo de Rationale
    if (showRationaleDialog) {
        AlertDialog(
            onDismissRequest = { showRationaleDialog = false },
            title = { Text("Permiso necesario") },
            text = {
                Text(
                    "La ubicación es necesaria para mostrar tu posición " +
                            "en el mapa y registrar coordenadas GPS. " +
                            "Sin este permiso, las funciones del mapa no estarán disponibles."
                )
            },
            confirmButton = {
                TextButton(onClick = {
                    showRationaleDialog = false
                    permissionLauncher.launch(locationPermissions)
                }) {
                    Text("Reintentar")
                }
            },
            dismissButton = {
                TextButton(onClick = { showRationaleDialog = false }) {
                    Text("Cancelar")
                }
            }
        )
    }
}

@Composable
private fun PermissionStatusCard(state: PermissionState) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surfaceVariant
        )
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = "Estado actual:",
                style = MaterialTheme.typography.titleSmall,
                fontWeight = FontWeight.Bold
            )
            Spacer(modifier = Modifier.height(8.dp))
            PermissionRow(
                label = "Ubicación precisa (GPS)",
                granted = state.fineLocationGranted
            )
            PermissionRow(
                label = "Ubicación aproximada (Red)",
                granted = state.coarseLocationGranted
            )
        }
    }
}

@Composable
private fun PermissionRow(label: String, granted: Boolean) {
    Text(
        text = if (granted) "✅ $label" else "❌ $label",
        style = MaterialTheme.typography.bodyMedium,
        modifier = Modifier.padding(vertical = 2.dp)
    )
}
```

**Resultado esperado:** La pantalla de permisos muestra el estado actual, permite solicitar permisos, muestra un diálogo de rationale y redirige a Settings cuando se deniega permanentemente.

**Verificación:** Compila el módulo con `Build > Make Module 'app'`. No debe haber errores.

---

### Paso 5 — Implementar el repositorio y ViewModel de ubicación

**Objetivo:** Crear `LocationRepository` con `FusedLocationProviderClient` y exponer las actualizaciones de ubicación como `Flow` consumido por un `ViewModel`.

**Instrucciones:**

1. Crea el archivo `data/location/LocationRepository.kt`:

```kotlin
package com.curso.android.avanzado.practica6.data.location

import android.annotation.SuppressLint
import android.content.Context
import android.location.Location
import android.os.Looper
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationCallback
import com.google.android.gms.location.LocationRequest
import com.google.android.gms.location.LocationResult
import com.google.android.gms.location.LocationServices
import com.google.android.gms.location.Priority
import kotlinx.coroutines.channels.awaitClose
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.callbackFlow

class LocationRepository(context: Context) {

    private val fusedLocationClient: FusedLocationProviderClient =
        LocationServices.getFusedLocationProviderClient(context)

    private val locationRequest: LocationRequest =
        LocationRequest.Builder(
            Priority.PRIORITY_HIGH_ACCURACY,
            5000L // intervalMillis = 5 segundos
        )
            .setMinUpdateIntervalMillis(2000L)
            .setWaitForAccurateLocation(false)
            .build()

    /**
     * Emite actualizaciones de ubicación continuas usando callbackFlow.
     * El Flow se cierra automáticamente cuando el colector se cancela.
     */
    @SuppressLint("MissingPermission")
    fun getLocationUpdates(): Flow<Location> = callbackFlow {
        val locationCallback = object : LocationCallback() {
            override fun onLocationResult(result: LocationResult) {
                result.lastLocation?.let { location ->
                    trySend(location)
                }
            }
        }

        fusedLocationClient.requestLocationUpdates(
            locationRequest,
            locationCallback,
            Looper.getMainLooper()
        )

        // Cuando el Flow se cancela, remover las actualizaciones
        awaitClose {
            fusedLocationClient.removeLocationUpdates(locationCallback)
        }
    }
}
```

> **Nota importante:** La anotación `@SuppressLint("MissingPermission")` es segura aquí porque la verificación de permisos se realiza en la capa de UI antes de invocar este repositorio.

2. Crea el archivo `viewmodel/LocationViewModel.kt`:

```kotlin
package com.curso.android.avanzado.practica6.viewmodel

import android.content.Context
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import com.curso.android.avanzado.practica6.data.location.LocationRepository
import com.curso.android.avanzado.practica6.data.location.LocationUiState
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.launch

class LocationViewModel(
    private val locationRepository: LocationRepository
) : ViewModel() {

    private val _locationState =
        MutableStateFlow<LocationUiState>(LocationUiState.Loading)
    val locationState: StateFlow<LocationUiState> = _locationState.asStateFlow()

    private var isTracking = false

    fun startLocationUpdates() {
        if (isTracking) return
        isTracking = true

        viewModelScope.launch {
            locationRepository.getLocationUpdates()
                .catch { exception ->
                    _locationState.value = LocationUiState.Error(
                        exception.message ?: "Error al obtener la ubicación"
                    )
                    isTracking = false
                }
                .collect { location ->
                    _locationState.value = LocationUiState.Success(
                        latitude = location.latitude,
                        longitude = location.longitude,
                        accuracy = location.accuracy
                    )
                }
        }
    }

    /**
     * Factory para crear el ViewModel con dependencias.
     */
    class Factory(private val context: Context) : ViewModelProvider.Factory {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            val repository = LocationRepository(context.applicationContext)
            return LocationViewModel(repository) as T
        }
    }
}
```

**Resultado esperado:** `LocationRepository` envuelve `FusedLocationProviderClient` en un `callbackFlow` y `LocationViewModel` expone el estado como `StateFlow<LocationUiState>`.

**Verificación:** `Build > Make Module 'app'` → BUILD SUCCESSFUL.

---

### Paso 6 — Implementar la pantalla del mapa con Maps Compose

**Objetivo:** Crear un composable que muestre Google Maps con un marcador en la ubicación actual y un botón para centrar la cámara.

**Instrucciones:**

1. Crea el archivo `ui/map/MapScreen.kt`:

```kotlin
package com.curso.android.avanzado.practica6.ui.map

import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.MyLocation
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.FloatingActionButton
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.curso.android.avanzado.practica6.data.location.LocationUiState
import com.curso.android.avanzado.practica6.data.location.PermissionState
import com.curso.android.avanzado.practica6.viewmodel.LocationViewModel
import com.curso.android.avanzado.practica6.viewmodel.PermissionViewModel
import com.google.android.gms.maps.CameraUpdateFactory
import com.google.android.gms.maps.model.CameraPosition
import com.google.android.gms.maps.model.LatLng
import com.google.maps.android.compose.GoogleMap
import com.google.maps.android.compose.MapProperties
import com.google.maps.android.compose.MapUiSettings
import com.google.maps.android.compose.Marker
import com.google.maps.android.compose.MarkerState
import com.google.maps.android.compose.rememberCameraPositionState
import kotlinx.coroutines.launch

@Composable
fun MapScreen(
    permissionViewModel: PermissionViewModel,
    locationViewModel: LocationViewModel
) {
    val permissionState by permissionViewModel.permissionState
        .collectAsStateWithLifecycle()
    val locationState by locationViewModel.locationState
        .collectAsStateWithLifecycle()

    if (!permissionState.anyLocationGranted) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Text(
                text = "Se requieren permisos de ubicación.\n" +
                        "Ve a la pestaña 'Permisos' para concederlos.",
                style = MaterialTheme.typography.bodyLarge,
                modifier = Modifier.padding(24.dp)
            )
        }
        return
    }

    // Iniciar actualizaciones de ubicación cuando los permisos están concedidos
    LaunchedEffect(permissionState.anyLocationGranted) {
        if (permissionState.anyLocationGranted) {
            locationViewModel.startLocationUpdates()
        }
    }

    when (val state = locationState) {
        is LocationUiState.Loading -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Column(horizontalAlignment = Alignment.CenterHorizontally) {
                    CircularProgressIndicator()
                    Text(
                        text = "Obteniendo ubicación...",
                        modifier = Modifier.padding(top = 16.dp),
                        style = MaterialTheme.typography.bodyMedium
                    )
                }
            }
        }

        is LocationUiState.Error -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    text = "Error: ${state.message}",
                    color = MaterialTheme.colorScheme.error,
                    modifier = Modifier.padding(24.dp)
                )
            }
        }

        is LocationUiState.Success -> {
            MapContent(
                latitude = state.latitude,
                longitude = state.longitude,
                accuracy = state.accuracy
            )
        }
    }
}

@Composable
private fun MapContent(
    latitude: Double,
    longitude: Double,
    accuracy: Float
) {
    val currentLocation = LatLng(latitude, longitude)
    val coroutineScope = rememberCoroutineScope()

    val cameraPositionState = rememberCameraPositionState {
        position = CameraPosition.fromLatLngZoom(currentLocation, 15f)
    }

    // Animar la cámara cuando cambia la ubicación
    LaunchedEffect(latitude, longitude) {
        cameraPositionState.animate(
            CameraUpdateFactory.newLatLngZoom(currentLocation, 15f),
            durationMs = 1000
        )
    }

    Box(modifier = Modifier.fillMaxSize()) {
        GoogleMap(
            modifier = Modifier.fillMaxSize(),
            cameraPositionState = cameraPositionState,
            properties = MapProperties(
                isMyLocationEnabled = false // Lo manejamos con nuestro marcador
            ),
            uiSettings = MapUiSettings(
                zoomControlsEnabled = true,
                compassEnabled = true
            )
        ) {
            Marker(
                state = MarkerState(position = currentLocation),
                title = "Mi ubicación",
                snippet = String.format(
                    "Lat: %.6f | Lng: %.6f | Precisión: %.1f m",
                    latitude,
                    longitude,
                    accuracy
                )
            )
        }

        // Botón flotante para centrar la cámara
        FloatingActionButton(
            onClick = {
                coroutineScope.launch {
                    cameraPositionState.animate(
                        CameraUpdateFactory.newLatLngZoom(currentLocation, 15f),
                        durationMs = 600
                    )
                }
            },
            modifier = Modifier
                .align(Alignment.BottomEnd)
                .padding(16.dp),
            containerColor = MaterialTheme.colorScheme.primaryContainer
        ) {
            Icon(
                imageVector = Icons.Default.MyLocation,
                contentDescription = "Centrar en mi ubicación"
            )
        }

        // Información de coordenadas superpuesta
        Text(
            text = String.format(
                "%.6f, %.6f (±%.0f m)",
                latitude,
                longitude,
                accuracy
            ),
            modifier = Modifier
                .align(Alignment.TopCenter)
                .padding(top = 8.dp),
            style = MaterialTheme.typography.labelMedium,
            color = MaterialTheme.colorScheme.onSurface
        )
    }
}
```

**Resultado esperado:** La pantalla del mapa muestra un indicador de carga mientras espera la primera ubicación, luego renderiza Google Maps con un marcador y un FAB para centrar.

**Verificación:** `Build > Make Module 'app'` → BUILD SUCCESSFUL.

---

### Paso 7 — Implementar el repositorio y ViewModel de sensores

**Objetivo:** Crear `SensorRepository` para leer datos del acelerómetro y sensor de luz, exponerlos como `StateFlow`.

**Instrucciones:**

1. Crea el archivo `data/sensor/SensorRepository.kt`:

```kotlin
package com.curso.android.avanzado.practica6.data.sensor

import android.content.Context
import android.hardware.Sensor
import android.hardware.SensorEvent
import android.hardware.SensorEventListener
import android.hardware.SensorManager
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class SensorRepository(context: Context) {

    private val sensorManager: SensorManager =
        context.getSystemService(Context.SENSOR_SERVICE) as SensorManager

    private val accelerometer: Sensor? =
        sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)

    private val lightSensor: Sensor? =
        sensorManager.getDefaultSensor(Sensor.TYPE_LIGHT)

    // StateFlows para exponer datos de sensores
    private val _accelerometerData = MutableStateFlow(AccelerometerData())
    val accelerometerFlow: StateFlow<AccelerometerData> =
        _accelerometerData.asStateFlow()

    private val _lightData = MutableStateFlow(0f)
    val lightFlow: StateFlow<Float> = _lightData.asStateFlow()

    // Disponibilidad de sensores
    val isAccelerometerAvailable: Boolean = accelerometer != null
    val isLightSensorAvailable: Boolean = lightSensor != null

    // Listeners
    private val accelerometerListener = object : SensorEventListener {
        override fun onSensorChanged(event: SensorEvent) {
            if (event.sensor.type == Sensor.TYPE_ACCELEROMETER) {
                _accelerometerData.value = AccelerometerData(
                    x = event.values[0],
                    y = event.values[1],
                    z = event.values[2]
                )
            }
        }

        override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {
            // No se requiere acción
        }
    }

    private val lightListener = object : SensorEventListener {
        override fun onSensorChanged(event: SensorEvent) {
            if (event.sensor.type == Sensor.TYPE_LIGHT) {
                _lightData.value = event.values[0]
            }
        }

        override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {
            // No se requiere acción
        }
    }

    /**
     * Registra los listeners de sensores.
     * Debe llamarse en onResume / DisposableEffect.
     */
    fun startListening() {
        accelerometer?.let {
            sensorManager.registerListener(
                accelerometerListener,
                it,
                SensorManager.SENSOR_DELAY_UI
            )
        }
        lightSensor?.let {
            sensorManager.registerListener(
                lightListener,
                it,
                SensorManager.SENSOR_DELAY_UI
            )
        }
    }

    /**
     * Desregistra los listeners de sensores.
     * Debe llamarse en onPause / DisposableEffect onDispose.
     */
    fun stopListening() {
        sensorManager.unregisterListener(accelerometerListener)
        sensorManager.unregisterListener(lightListener)
    }
}
```

2. Crea el archivo `viewmodel/SensorViewModel.kt`:

```kotlin
package com.curso.android.avanzado.practica6.viewmodel

import android.content.Context
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.curso.android.avanzado.practica6.data.sensor.AccelerometerData
import com.curso.android.avanzado.practica6.data.sensor.SensorRepository
import kotlinx.coroutines.flow.StateFlow

class SensorViewModel(
    private val sensorRepository: SensorRepository
) : ViewModel() {

    val accelerometerData: StateFlow<AccelerometerData> =
        sensorRepository.accelerometerFlow

    val lightData: StateFlow<Float> =
        sensorRepository.lightFlow

    val isAccelerometerAvailable: Boolean =
        sensorRepository.isAccelerometerAvailable

    val isLightSensorAvailable: Boolean =
        sensorRepository.isLightSensorAvailable

    fun startListening() {
        sensorRepository.startListening()
    }

    fun stopListening() {
        sensorRepository.stopListening()
    }

    override fun onCleared() {
        super.onCleared()
        sensorRepository.stopListening()
    }

    class Factory(private val context: Context) : ViewModelProvider.Factory {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            val repository = SensorRepository(context.applicationContext)
            return SensorViewModel(repository) as T
        }
    }
}
```

**Resultado esperado:** Los sensores se registran/desregistran correctamente y sus datos fluyen a través de `StateFlow`.

**Verificación:** `Build > Make Module 'app'` → BUILD SUCCESSFUL.

---

### Paso 8 — Implementar la pantalla de sensores

**Objetivo:** Crear una UI en Compose que muestre los datos del acelerómetro y sensor de luz en tiempo real, con gestión correcta del ciclo de vida.

**Instrucciones:**

1. Crea el archivo `ui/sensors/SensorScreen.kt`:

```kotlin
package com.curso.android.avanzado.practica6.ui.sensors

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.DirectionsRun
import androidx.compose.material.icons.filled.LightMode
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.Icon
import androidx.compose.material3.LinearProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.DisposableEffect
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.curso.android.avanzado.practica6.data.sensor.AccelerometerData
import com.curso.android.avanzado.practica6.viewmodel.SensorViewModel

@Composable
fun SensorScreen(sensorViewModel: SensorViewModel) {
    val accelerometerData by sensorViewModel.accelerometerData
        .collectAsStateWithLifecycle()
    val lightValue by sensorViewModel.lightData
        .collectAsStateWithLifecycle()

    // Gestión del ciclo de vida: registrar al entrar, desregistrar al salir
    DisposableEffect(Unit) {
        sensorViewModel.startListening()
        onDispose {
            sensorViewModel.stopListening()
        }
    }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "Sensores del Dispositivo",
            style = MaterialTheme.typography.headlineMedium,
            fontWeight = FontWeight.Bold
        )

        Spacer(modifier = Modifier.height(8.dp))

        // Tarjeta del Acelerómetro
        AccelerometerCard(
            data = accelerometerData,
            isAvailable = sensorViewModel.isAccelerometerAvailable
        )

        // Tarjeta del Sensor de Luz
        LightSensorCard(
            luxValue = lightValue,
            isAvailable = sensorViewModel.isLightSensorAvailable
        )
    }
}

@Composable
private fun AccelerometerCard(
    data: AccelerometerData,
    isAvailable: Boolean
) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer
        )
    ) {
        Column(modifier = Modifier.padding(20.dp)) {
            Row(verticalAlignment = Alignment.CenterVertically) {
                Icon(
                    imageVector = Icons.Default.DirectionsRun,
                    contentDescription = "Acelerómetro",
                    modifier = Modifier.size(32.dp),
                    tint = MaterialTheme.colorScheme.onPrimaryContainer
                )
                Spacer(modifier = Modifier.width(12.dp))
                Text(
                    text = "Acelerómetro",
                    style = MaterialTheme.typography.titleLarge,
                    fontWeight = FontWeight.Bold,
                    color = MaterialTheme.colorScheme.onPrimaryContainer
                )
            }

            Spacer(modifier = Modifier.height(16.dp))

            if (!isAvailable) {
                Text(
                    text = "⚠️ Sensor no disponible en este dispositivo",
                    color = MaterialTheme.colorScheme.error
                )
            } else {
                // Valores de ejes
                AxisRow(label = "Eje X", value = data.x, unit = "m/s²")
                AxisRow(label = "Eje Y", value = data.y, unit = "m/s²")
                AxisRow(label = "Eje Z", value = data.z, unit = "m/s²")

                Spacer(modifier = Modifier.height(12.dp))

                // Magnitud con barra de progreso
                Text(
                    text = String.format(
                        "Magnitud: %.2f m/s²",
                        data.magnitude
                    ),
                    style = MaterialTheme.typography.bodyLarge,
                    fontWeight = FontWeight.SemiBold,
                    color = MaterialTheme.colorScheme.onPrimaryContainer
                )

                Spacer(modifier = Modifier.height(8.dp))

                // Barra visual: gravedad terrestre ≈ 9.81 m/s²
                // Normalizamos a un rango de 0-20 m/s²
                val progress = (data.magnitude / 20f).coerceIn(0f, 1f)
                LinearProgressIndicator(
                    progress = { progress },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(12.dp),
                    color = when {
                        data.magnitude > 15f -> MaterialTheme.colorScheme.error
                        data.magnitude > 12f ->
                            MaterialTheme.colorScheme.tertiary
                        else -> MaterialTheme.colorScheme.primary
                    }
                )

                Spacer(modifier = Modifier.height(4.dp))

                Text(
                    text = when {
                        data.magnitude > 15f -> "⚠️ Movimiento intenso"
                        data.magnitude > 12f -> "📳 Movimiento moderado"
                        else -> "📱 Estable"
                    },
                    style = MaterialTheme.typography.labelMedium,
                    color = MaterialTheme.colorScheme.onPrimaryContainer
                )
            }
        }
    }
}

@Composable
private fun AxisRow(label: String, value: Float, unit: String) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 2.dp),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        Text(
            text = label,
            style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onPrimaryContainer
        )
        Text(
            text = String.format("%+8.3f %s", value, unit),
            style = MaterialTheme.typography.bodyMedium,
            fontFamily = FontFamily.Monospace,
            color = MaterialTheme.colorScheme.onPrimaryContainer
        )
    }
}

@Composable
private fun LightSensorCard(
    luxValue: Float,
    isAvailable: Boolean
) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.secondaryContainer
        )
    ) {
        Column(modifier = Modifier.padding(20.dp)) {
            Row(verticalAlignment = Alignment.CenterVertically) {
                Icon(
                    imageVector = Icons.Default.LightMode,
                    contentDescription = "Sensor de Luz",
                    modifier = Modifier.size(32.dp),
                    tint = MaterialTheme.colorScheme.onSecondaryContainer
                )
                Spacer(modifier = Modifier.width(12.dp))
                Text(
                    text = "Sensor de Luz",
                    style = MaterialTheme.typography.titleLarge,
                    fontWeight = FontWeight.Bold,
                    color = MaterialTheme.colorScheme.onSecondaryContainer
                )
            }

            Spacer(modifier = Modifier.height(16.dp))

            if (!isAvailable) {
                Text(
                    text = "⚠️ Sensor no disponible en este dispositivo",
                    color = MaterialTheme.colorScheme.error
                )
            } else {
                Text(
                    text = String.format("%.1f lux", luxValue),
                    style = MaterialTheme.typography.displaySmall,
                    fontWeight = FontWeight.Bold,
                    fontFamily = FontFamily.Monospace,
                    color = MaterialTheme.colorScheme.onSecondaryContainer
                )

                Spacer(modifier = Modifier.height(8.dp))

                // Barra visual normalizada (máx ~40000 lux luz solar directa)
                val progress = (luxValue / 40000f).coerceIn(0f, 1f)
                LinearProgressIndicator(
                    progress = { progress },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(12.dp),
                    color = MaterialTheme.colorScheme.secondary
                )

                Spacer(modifier = Modifier.height(4.dp))

                Text(
                    text = when {
                        luxValue < 10f -> "🌙 Oscuridad"
                        luxValue < 200f -> "🏠 Interior"
                        luxValue < 1000f -> "☁️ Nublado"
                        luxValue < 10000f -> "🌤️ Sombra exterior"
                        else -> "☀️ Luz solar directa"
                    },
                    style = MaterialTheme.typography.labelMedium,
                    color = MaterialTheme.colorScheme.onSecondaryContainer
                )
            }
        }
    }
}
```

**Resultado esperado:** La pantalla muestra los valores del acelerómetro (X, Y, Z, magnitud) y del sensor de luz (lux) con indicadores visuales. `DisposableEffect` gestiona el registro y desregistro de listeners.

**Verificación:** `Build > Make Module 'app'` → BUILD SUCCESSFUL.

---

### Paso 9 — Implementar la navegación con BottomNavigationBar

**Objetivo:** Crear la estructura de navegación entre las tres pantallas usando Navigation Compose y una barra de navegación inferior.

**Instrucciones:**

1. Crea el archivo `ui/navigation/AppNavigation.kt`:

```kotlin
package com.curso.android.avanzado.practica6.ui.navigation

import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Map
import androidx.compose.material.icons.filled.Security
import androidx.compose.material.icons.filled.Sensors
import androidx.compose.ui.graphics.vector.ImageVector

sealed class Screen(
    val route: String,
    val title: String,
    val icon: ImageVector
) {
    data object Permissions : Screen(
        route = "permissions",
        title = "Permisos",
        icon = Icons.Default.Security
    )
    data object Map : Screen(
        route = "map",
        title = "Mapa",
        icon = Icons.Default.Map
    )
    data object Sensors : Screen(
        route = "sensors",
        title = "Sensores",
        icon = Icons.Default.Sensors
    )

    companion object {
        val items = listOf(Permissions, Map, Sensors)
    }
}
```

2. Abre `MainActivity.kt` y reemplaza su contenido completo con:

```kotlin
package com.curso.android.avanzado.practica6

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Icon
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.NavigationBar
import androidx.compose.material3.NavigationBarItem
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController
import com.curso.android.avanzado.practica6.ui.map.MapScreen
import com.curso.android.avanzado.practica6.ui.navigation.Screen
import com.curso.android.avanzado.practica6.ui.permissions.PermissionScreen
import com.curso.android.avanzado.practica6.ui.sensors.SensorScreen
import com.curso.android.avanzado.practica6.ui.theme.SensorMapAppTheme
import com.curso.android.avanzado.practica6.viewmodel.LocationViewModel
import com.curso.android.avanzado.practica6.viewmodel.PermissionViewModel
import com.curso.android.avanzado.practica6.viewmodel.SensorViewModel

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            SensorMapAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    SensorMapApp()
                }
            }
        }
    }
}

@Composable
fun SensorMapApp() {
    val navController = rememberNavController()

    // ViewModels compartidos a nivel de la Activity
    val permissionViewModel: PermissionViewModel = viewModel()
    val locationViewModel: LocationViewModel = viewModel(
        factory = LocationViewModel.Factory(
            androidx.compose.ui.platform.LocalContext.current
        )
    )
    val sensorViewModel: SensorViewModel = viewModel(
        factory = SensorViewModel.Factory(
            androidx.compose.ui.platform.LocalContext.current
        )
    )

    Scaffold(
        bottomBar = {
            BottomNavigationBar(navController = navController)
        }
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = Screen.Permissions.route,
            modifier = Modifier.padding(innerPadding)
        ) {
            composable(Screen.Permissions.route) {
                PermissionScreen(permissionViewModel = permissionViewModel)
            }
            composable(Screen.Map.route) {
                MapScreen(
                    permissionViewModel = permissionViewModel,
                    locationViewModel = locationViewModel
                )
            }
            composable(Screen.Sensors.route) {
                SensorScreen(sensorViewModel = sensorViewModel)
            }
        }
    }
}

@Composable
private fun BottomNavigationBar(navController: NavHostController) {
    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentRoute = navBackStackEntry?.destination?.route

    NavigationBar {
        Screen.items.forEach { screen ->
            NavigationBarItem(
                icon = {
                    Icon(
                        imageVector = screen.icon,
                        contentDescription = screen.title
                    )
                },
                label = { Text(screen.title) },
                selected = currentRoute == screen.route,
                onClick = {
                    navController.navigate(screen.route) {
                        // Evitar múltiples copias en el back stack
                        popUpTo(navController.graph.startDestinationId) {
                            saveState = true
                        }
                        launchSingleTop = true
                        restoreState = true
                    }
                }
            )
        }
    }
}
```

**Resultado esperado:** La aplicación muestra una barra de navegación inferior con tres ítems (Permisos, Mapa, Sensores). Los ViewModels se comparten entre pantallas, por lo que el estado de los permisos persiste al navegar.

**Verificación:** `Build > Make Module 'app'` → BUILD SUCCESSFUL.

---

### Paso 10 — Ejecutar y probar la aplicación en el emulador

**Objetivo:** Validar el funcionamiento completo de la aplicación en un AVD con Google Play Services, probando permisos, ubicación simulada y sensores.

**Instrucciones:**

1. Selecciona un AVD con imagen **Google Play** (API 37 recomendado) y ejecútalo.

2. Ejecuta la aplicación: **Run > Run 'app'** (o `Shift + F10`).

3. **Prueba de Permisos:**
   - La aplicación inicia en la pestaña "Permisos".
   - Verifica que se muestran los estados ❌ para ambos permisos.
   - Pulsa "Solicitar Permisos".
   - En el diálogo del sistema, selecciona **"While using the app"** (Mientras se usa la app).
   - Verifica que los estados cambian a ✅ y aparece el mensaje de confirmación.

4. **Prueba de denegación con rationale:**
   - Desinstala la app del emulador y vuelve a ejecutar.
   - Pulsa "Solicitar Permisos" y selecciona **"Don't allow"**.
   - Verifica que aparece el diálogo de rationale.
   - Pulsa "Reintentar" en el diálogo.

5. **Prueba de denegación permanente:**
   - Si el sistema ya no muestra el diálogo (tras dos denegaciones), verifica que aparece el botón "Abrir Configuración".
   - Pulsa el botón y verifica que se abre la pantalla de configuración de la app.

6. **Prueba de Ubicación en el Mapa:**
   - Concede los permisos y navega a la pestaña "Mapa".
   - Abre **Extended Controls** del emulador (botón `...` en la barra lateral del AVD).
   - Ve a **Location** y establece:
     - **Latitude:** `19.4326`
     - **Longitude:** `-99.1332`
   - Pulsa **Send**.
   - Verifica que el mapa muestra un marcador en Ciudad de México.
   - Toca el marcador para ver la ventana de información con coordenadas y precisión.
   - Pulsa el FAB (botón de centrar) y verifica que la cámara se anima hacia la posición.

7. **Prueba de Sensores:**
   - Navega a la pestaña "Sensores".
   - Verifica que los datos del acelerómetro se muestran (en el emulador, los valores pueden ser estáticos o simulados).
   - Verifica que el sensor de luz muestra un valor en lux.
   - Si usas un dispositivo físico, mueve el dispositivo y observa cómo cambian los valores del acelerómetro y la barra de progreso.

> **Nota sobre emuladores:** Los AVDs simulan el acelerómetro con valores estáticos (aproximadamente 0, 0, 9.81 m/s²). Para probar cambios dinámicos, usa **Extended Controls > Virtual Sensors > Accelerometer** y mueve los controles deslizantes, o prueba en un dispositivo físico.

**Resultado esperado:**

| Pantalla | Comportamiento esperado |
|---|---|
| Permisos | Solicitud funcional, rationale dialog, redirección a Settings |
| Mapa | Google Maps visible, marcador en coordenadas enviadas, FAB funcional |
| Sensores | Valores del acelerómetro y luz actualizándose en tiempo real |

**Verificación visual:**

- La barra de navegación inferior muestra tres ítems con íconos.
- Al navegar entre pestañas, el estado de los permisos se mantiene (no se resetea).
- El marcador del mapa muestra un `InfoWindow` al tocarlo.
- La barra de progreso del acelerómetro refleja la magnitud del vector de aceleración.

---

## 7. Validación y Pruebas

### Prueba 1 — Verificar ciclo de vida de sensores

1. Navega a la pestaña "Sensores" (los listeners se registran).
2. Navega a la pestaña "Mapa" (los listeners deben desregistrarse via `DisposableEffect`).
3. Abre **Logcat** y filtra por `SensorManager`.
4. Verifica que no hay mensajes de error sobre listeners no desregistrados.

### Prueba 2 — Verificar que la ubicación se detiene al destruir el ViewModel

1. Navega a la pestaña "Mapa" con permisos concedidos.
2. Cierra la aplicación completamente (swipe desde recientes).
3. En Logcat, verifica que no hay callbacks de ubicación ejecutándose después del cierre (el `callbackFlow` con `awaitClose` debe haber limpiado el callback).

### Prueba 3 — Verificar navegación y persistencia de estado

1. Concede permisos en la pestaña "Permisos".
2. Navega a "Mapa" → verifica que detecta los permisos concedidos sin solicitarlos de nuevo.
3. Navega a "Sensores" → navega de vuelta a "Permisos" → verifica que los estados siguen en ✅.

### Prueba 4 — Probar en AVD API 30 (opcional)

1. Ejecuta la aplicación en un AVD con API 30.
2. Observa que el diálogo de permisos del sistema tiene un formato diferente al de API 37 (sin la opción "Only this time").
3. Verifica que el flujo de permisos funciona correctamente en ambas versiones.

---

## 8. Solución de Problemas

### Problema 1: El mapa aparece en gris o no se renderiza

**Síntomas:** La pantalla del mapa muestra un área gris o un grid vacío en lugar del mapa de Google Maps. No hay errores de compilación.

**Causa:** La API Key de Google Maps no está configurada correctamente, no tiene el servicio "Maps SDK for Android" habilitado en Google Cloud Console, o la restricción de la clave no incluye el SHA-1 de la app de debug.

**Solución:**

1. Verifica que `local.properties` contiene la clave correcta:
   ```properties
   MAPS_API_KEY=AIzaSy...tu_clave_real
   ```

2. Verifica en Google Cloud Console que "Maps SDK for Android" está **habilitado**.

3. Obtén el SHA-1 de tu certificado de debug ejecutando en la terminal de Android Studio:
   ```bash
   ./gradlew signingReport
   ```
   Busca el SHA-1 bajo `Variant: debug` y agrégalo como restricción de la API Key en Cloud Console (o temporalmente elimina las restricciones para depurar).

4. Limpia y reconstruye:
   ```
   Build > Clean Project
   Build > Rebuild Project
   ```

5. Verifica en Logcat filtrando por `Google Maps Android API` que no aparezcan errores de autenticación.

---

### Problema 2: `SecurityException` al solicitar ubicación — "Missing location permission"

**Síntomas:** La aplicación se cierra con un crash `java.lang.SecurityException: Client must have ACCESS_FINE_LOCATION or ACCESS_COARSE_LOCATION permission` al navegar a la pestaña del mapa.

**Causa:** El `LocationViewModel.startLocationUpdates()` se invoca antes de que los permisos hayan sido concedidos. Esto puede ocurrir si el `LaunchedEffect` en `MapScreen` se ejecuta cuando `permissionState.anyLocationGranted` es `true` pero el sistema aún no ha propagado el permiso, o si se modificó el flujo eliminando la verificación de permisos.

**Solución:**

1. Verifica que `MapScreen` contiene la guarda temprana (early return) antes de cualquier operación de ubicación:
   ```kotlin
   if (!permissionState.anyLocationGranted) {
       // Mostrar mensaje y retornar
       return
   }
   ```

2. Verifica que el `LaunchedEffect` está condicionado correctamente:
   ```kotlin
   LaunchedEffect(permissionState.anyLocationGranted) {
       if (permissionState.anyLocationGranted) {
           locationViewModel.startLocationUpdates()
       }
   }
   ```

3. Si el problema persiste, agrega una verificación adicional en `LocationRepository` antes de solicitar actualizaciones:
   ```kotlin
   import android.content.pm.PackageManager
   import androidx.core.content.ContextCompat

   // Dentro de getLocationUpdates(), antes de requestLocationUpdates:
   if (ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION)
       != PackageManager.PERMISSION_GRANTED) {
       close(SecurityException("Permiso de ubicación no concedido"))
       return@callbackFlow
   }
   ```

4. Reconstruye y ejecuta nuevamente.

---

## 9. Limpieza

Al finalizar la práctica, si necesitas liberar espacio o preparar el entorno para la siguiente sesión:

1. **Detener el emulador:** Cierra el AVD desde la ventana del emulador o desde **Device Manager** en Android Studio.

2. **Revocar permisos de prueba** (opcional): En el AVD, ve a **Settings > Apps > SensorMapApp > Permissions** y revoca los permisos de ubicación para poder repetir las pruebas de permisos.

3. **No eliminar el proyecto:** Este proyecto `SensorMapApp` será reutilizado y extendido en la **Práctica 7** del curso.

4. **Proteger la API Key:** Verifica que `local.properties` está listado en `.gitignore` antes de subir el proyecto a cualquier repositorio:
   ```bash
   cat .gitignore | grep local.properties
   ```
   Debe aparecer `local.properties` en la salida.

---

## 10. Resumen

En esta práctica has construido **SensorMapApp**, una aplicación Android que integra tres pilares fundamentales del acceso a hardware y servicios del dispositivo:

| Sección | Concepto clave implementado |
|---|---|
| **Permisos** | `ActivityResultContracts.RequestMultiplePermissions`, rationale dialog, redirección a Settings para denegación permanente |
| **Ubicación** | `FusedLocationProviderClient` con `callbackFlow`, `LocationRequest.Builder` con `PRIORITY_HIGH_ACCURACY`, `StateFlow<LocationUiState>` |
| **Mapas** | Maps Compose 6.4.1 (`GoogleMap`, `Marker`, `CameraPositionState`), API Key segura via `local.properties` y `BuildConfig` |
| **Sensores** | `SensorManager` con `TYPE_ACCELEROMETER` y `TYPE_LIGHT`, `SensorEventListener`, `DisposableEffect` para ciclo de vida |
| **Arquitectura** | ViewModels compartidos, `sealed class` para estados, `collectAsStateWithLifecycle()`, Navigation Compose con `BottomNavigationBar` |

### Conceptos clave reforzados

- Los **permisos en tiempo de ejecución** requieren manejar tres estados distintos, no solo "concedido" o "denegado".
- **`callbackFlow`** es el patrón idiomático de Kotlin para convertir callbacks tradicionales (como `LocationCallback`) en `Flow` reactivos.
- **`DisposableEffect`** es esencial para gestionar recursos que requieren limpieza (sensores, callbacks) en Compose.
- La **API Key de Google Maps** nunca debe estar hardcodeada ni versionada; `local.properties` + `BuildConfig` es el patrón seguro.

### Recursos adicionales

- [Documentación oficial: Solicitar permisos en tiempo de ejecución](https://developer.android.com/training/permissions/requesting)
- [FusedLocationProviderClient — Referencia de API](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderClient)
- [Maps Compose Library — GitHub](https://github.com/googlemaps/android-maps-compose)
- [Guía de sensores de Android](https://developer.android.com/guide/topics/sensors/sensors_overview)
- [StateFlow y SharedFlow — Kotlin Docs](https://kotlinlang.org/docs/stateflow-and-sharedflow.html)
- [DisposableEffect en Compose — Android Developers](https://developer.android.com/develop/ui/compose/side-effects#disposableeffect)

---
