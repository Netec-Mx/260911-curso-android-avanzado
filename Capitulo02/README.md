---LAB_START---
LAB_ID: 02-00-01
---MARKDOWN---
# Práctica 2 — Interfaces Modernas con Jetpack Compose

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 288 minutos (~4 h 48 min) |
| **Complejidad** | Alta |
| **Nivel Bloom** | Crear |
| **Práctica** | Practica2_ComposeUI |
| **Paquete base** | `com.cursoadv.android.compose` |

---

## 2. Descripción General

En esta práctica construirás desde cero una aplicación Android multi-pantalla utilizando Jetpack Compose con Material 3. Partirás de un análisis comparativo entre el paradigma imperativo (XML/View) y el declarativo (Compose), progresarás por la construcción de composables, gestión de estado con `remember`/`mutableStateOf`, refactorización mediante State Hoisting, navegación tipada con Compose Navigation, creación de un tema personalizado con soporte claro/oscuro, y finalizarás con una lista dinámica alimentada por `StateFlow` que muestra tarjetas con imágenes cargadas por Coil. Al concluir, tendrás un shell visual funcional que servirá como base para las Prácticas 3, 4 y 5 del curso.

---

## 3. Objetivos de Aprendizaje

Al completar esta práctica serás capaz de:

- [ ] Construir composables básicos y compuestos utilizando `Text`, `Button`, `Image`, `Column`, `Row`, `Box` y `Modifier` para crear layouts complejos
- [ ] Gestionar el estado local de la UI con `remember` y `mutableStateOf`, e implementar State Hoisting para crear composables stateless reutilizables
- [ ] Configurar navegación multi-pantalla con `NavController`, `NavHost` y argumentos tipados entre destinos
- [ ] Aplicar un tema Material 3 personalizado con `ColorScheme`, `Typography` y soporte de tema claro/oscuro
- [ ] Crear listas dinámicas eficientes con `LazyColumn` utilizando tarjetas `Card`, imágenes asíncronas con Coil y datos provenientes de un `StateFlow`

---

## 4. Prerrequisitos

### Conocimientos requeridos

| Concepto | Nivel |
|---|---|
| Kotlin (corrutinas, Flows, sealed classes) | Avanzado — completado en Práctica 1 |
| StateFlow y su colección en ViewModels | Intermedio — Práctica 1 |
| Ciclo de vida de Activity en Android | Básico |
| Layouts Android (XML o Compose, al menos conceptual) | Básico |

### Acceso y herramientas

- Android Studio Quail 3 (2026.1.3 Patch 1) instalado y funcional
- SDK Platform API 36 y API 37 descargados
- AVD configurado con API 36 o dispositivo físico con Android 16+
- Conexión a Internet estable para descarga de dependencias Gradle y carga de imágenes remotas
- Directorio de trabajo: `~/AndroidStudioProjects/AvanzadoKotlin/`

---

## 5. Entorno del Laboratorio

### Hardware mínimo

| Componente | Requisito |
|---|---|
| Procesador | Intel Core i7 8.ª gen+ / AMD Ryzen 7+ / Apple M1+ |
| RAM | 16 GB mínimo (32 GB recomendado) |
| Disco | 60 GB libres en SSD |
| Virtualización | VT-x / AMD-V / Apple Hypervisor habilitado en BIOS |

### Software requerido

| Herramienta | Versión exacta |
|---|---|
| Android Studio | Quail 3 — 2026.1.3 Patch 1 |
| Kotlin | 2.2.10 |
| AGP | 9.3.2 |
| Gradle Wrapper | 8.14.1 |
| KSP | 2.2.10-1.0.31 |
| Compose BOM | 2026.02.01 |
| Activity Compose | 1.13.0 |
| Coil Compose | 3.1.0 |
| minSdk / compileSdk / targetSdk | 30 / 37 / 37 |

---

## 6. Instrucciones Paso a Paso

### Bloque 1 — Creación del Proyecto y Configuración de Dependencias (30 min)

**Objetivo:** Crear el proyecto `Practica2_ComposeUI` con la plantilla Empty Activity de Compose y configurar el catálogo de versiones completo.

#### Instrucciones

1. Abre Android Studio y selecciona **File → New → New Project**.
2. Elige la plantilla **Empty Activity** (la que genera Jetpack Compose, no Empty Views Activity).
3. Configura los campos:
   - **Name:** `Practica2_ComposeUI`
   - **Package name:** `com.cursoadv.android.compose`
   - **Save location:** `~/AndroidStudioProjects/AvanzadoKotlin/Practica2_ComposeUI`
   - **Minimum SDK:** API 30
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
navigationCompose = "2.9.0"
coilCompose = "3.1.0"
junit = "4.13.2"
androidxJunit = "1.3.0"
espressoCore = "3.7.0"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-ui = { group = "androidx.compose.ui", name = "ui" }
androidx-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
androidx-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
androidx-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
androidx-material3 = { group = "androidx.compose.material3", name = "material3" }
androidx-material-icons-extended = { group = "androidx.compose.material", name = "material-icons-extended" }
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigationCompose" }
coil-compose = { group = "io.coil-kt.coil3", name = "coil-compose", version.ref = "coilCompose" }
coil-network-okhttp = { group = "io.coil-kt.coil3", name = "coil-network-okhttp", version.ref = "coilCompose" }
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

7. Abre el archivo `app/build.gradle.kts` y configúralo así:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.cursoadv.android.compose"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.cursoadv.android.compose"
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
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.viewmodel.compose)
    implementation(libs.androidx.lifecycle.runtime.compose)
    implementation(libs.androidx.activity.compose)
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.material.icons.extended)
    implementation(libs.androidx.navigation.compose)
    implementation(libs.coil.compose)
    implementation(libs.coil.network.okhttp)

    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)

    debugImplementation(libs.androidx.ui.tooling)
    debugImplementation(libs.androidx.ui.test.manifest)
}
```

8. Haz clic en **Sync Now** en la barra de notificación de Gradle.

9. Verifica que el archivo `gradle/wrapper/gradle-wrapper.properties` contenga:

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.14.1-bin.zip
```

10. Agrega el permiso de Internet en `app/src/main/AndroidManifest.xml` (necesario para Coil):

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
    <application
        ...
    </application>
</manifest>
```

#### Resultado esperado

Gradle sincroniza sin errores. El proyecto compila y muestra la pantalla predeterminada "Hello Android!" de Compose al ejecutar en el AVD.

#### Verificación

```bash
cd ~/AndroidStudioProjects/AvanzadoKotlin/Practica2_ComposeUI
./gradlew assembleDebug
```

El comando debe terminar con `BUILD SUCCESSFUL`.

---

### Bloque 2 — Análisis Comparativo XML vs Compose (25 min)

**Objetivo:** Comprender las diferencias fundamentales entre el paradigma imperativo (XML/View) y el declarativo (Compose) reconstruyendo una pantalla simple de saludo en ambos enfoques.

#### Instrucciones

1. Crea el directorio de referencia para el análisis. Este archivo **no se usará en la app final**, es solo para comparación conceptual. Crea el archivo `app/src/main/res/layout/activity_saludo_xml.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="24dp">

    <TextView
        android:id="@+id/tvSaludo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hola, Visitante"
        android:textSize="24sp"
        android:textStyle="bold" />

    <EditText
        android:id="@+id/etNombre"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="Escribe tu nombre" />

    <Button
        android:id="@+id/btnSaludar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Saludar" />
</LinearLayout>
```

2. Observa el código imperativo equivalente que se necesitaría en una Activity XML (solo referencia, **no lo implementes**):

```kotlin
// Enfoque IMPERATIVO — solo referencia conceptual
class SaludoXmlActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_saludo_xml)

        val tvSaludo = findViewById<TextView>(R.id.tvSaludo)
        val etNombre = findViewById<EditText>(R.id.etNombre)
        val btnSaludar = findViewById<Button>(R.id.btnSaludar)

        btnSaludar.setOnClickListener {
            val nombre = etNombre.text.toString()
            tvSaludo.text = if (nombre.isNotBlank()) "Hola, $nombre" else "Hola, Visitante"
        }
    }
}
```

3. Ahora crea la versión declarativa en Compose. Crea el paquete `ui.comparativa` dentro de `com.cursoadv.android.compose`. Crea el archivo `SaludoCompose.kt`:

```kotlin
package com.cursoadv.android.compose.ui.comparativa

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun PantallaSaludo() {
    var nombre by remember { mutableStateOf("") }
    var textoSaludo by remember { mutableStateOf("Hola, Visitante") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = textoSaludo,
            style = MaterialTheme.typography.headlineMedium
        )

        Spacer(modifier = Modifier.height(16.dp))

        OutlinedTextField(
            value = nombre,
            onValueChange = { nombre = it },
            label = { Text("Escribe tu nombre") },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(16.dp))

        Button(
            onClick = {
                textoSaludo = if (nombre.isNotBlank()) "Hola, $nombre" else "Hola, Visitante"
            }
        ) {
            Text("Saludar")
        }
    }
}
```

4. Modifica temporalmente `MainActivity.kt` para mostrar esta pantalla:

```kotlin
package com.cursoadv.android.compose

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.cursoadv.android.compose.ui.comparativa.PantallaSaludo

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MaterialTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    PantallaSaludo()
                }
            }
        }
    }
}
```

5. Ejecuta la app en el AVD. Escribe un nombre en el campo y presiona "Saludar".

6. **Reflexión documentada:** Crea un archivo `COMPARATIVA.md` en la raíz del proyecto con una tabla que resuma al menos 5 diferencias clave entre ambos enfoques (archivos necesarios, binding, reactividad, reutilización, testing).

#### Resultado esperado

La pantalla muestra un título "Hola, Visitante", un campo de texto y un botón. Al escribir un nombre y presionar "Saludar", el título cambia a "Hola, [nombre]".

#### Verificación

- La UI responde al clic del botón actualizando el texto
- No hay `findViewById` ni archivos XML de layout en uso activo
- El archivo `COMPARATIVA.md` existe con al menos 5 diferencias documentadas

---

### Bloque 3 — Composables Básicos y Personalizados con Modifier (35 min)

**Objetivo:** Construir composables reutilizables utilizando `Text`, `Button`, `Image`, `Column`, `Row`, `Box` y cadenas de `Modifier` incluyendo `padding`, `fillMaxSize`, `background`, `clip` y `aspectRatio`.

#### Instrucciones

1. Crea el paquete `ui.components` dentro de `com.cursoadv.android.compose`.

2. Crea el archivo `TarjetaPerfil.kt` con un composable que combine `Row`, `Box`, `Column` e `Image`:

```kotlin
package com.cursoadv.android.compose.ui.components

import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Person
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp

@Composable
fun TarjetaPerfil(
    nombre: String,
    descripcion: String,
    imagenUrl: String?,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 8.dp),
        shape = RoundedCornerShape(16.dp),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            // Avatar circular
            Box(
                modifier = Modifier
                    .size(64.dp)
                    .clip(CircleShape)
                    .background(MaterialTheme.colorScheme.primaryContainer),
                contentAlignment = Alignment.Center
            ) {
                Icon(
                    imageVector = Icons.Filled.Person,
                    contentDescription = "Avatar de $nombre",
                    modifier = Modifier.size(36.dp),
                    tint = MaterialTheme.colorScheme.onPrimaryContainer
                )
            }

            Spacer(modifier = Modifier.width(16.dp))

            Column(
                modifier = Modifier.weight(1f)
            ) {
                Text(
                    text = nombre,
                    style = MaterialTheme.typography.titleMedium,
                    fontWeight = FontWeight.Bold
                )
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = descripcion,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
fun TarjetaPerfilPreview() {
    MaterialTheme {
        TarjetaPerfil(
            nombre = "Ana García",
            descripcion = "Desarrolladora Android Senior",
            imagenUrl = null
        )
    }
}
```

3. Crea el archivo `BotonPrimario.kt` — un botón reutilizable con estilo consistente:

```kotlin
package com.cursoadv.android.compose.ui.components

import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun BotonPrimario(
    texto: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    habilitado: Boolean = true
) {
    Button(
        onClick = onClick,
        modifier = modifier
            .fillMaxWidth()
            .height(52.dp),
        enabled = habilitado,
        shape = RoundedCornerShape(12.dp),
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary
        )
    ) {
        Text(
            text = texto,
            style = MaterialTheme.typography.labelLarge
        )
    }
}
```

4. Crea el archivo `IndicadorEstado.kt` — un composable que usa `Box` con `aspectRatio`:

```kotlin
package com.cursoadv.android.compose.ui.components

import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.unit.dp

@Composable
fun IndicadorEstado(
    etiqueta: String,
    valor: String,
    modifier: Modifier = Modifier
) {
    Box(
        modifier = modifier
            .aspectRatio(1.5f)
            .clip(RoundedCornerShape(12.dp))
            .background(MaterialTheme.colorScheme.secondaryContainer)
            .padding(12.dp),
        contentAlignment = Alignment.Center
    ) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Text(
                text = valor,
                style = MaterialTheme.typography.headlineSmall,
                color = MaterialTheme.colorScheme.onSecondaryContainer
            )
            Text(
                text = etiqueta,
                style = MaterialTheme.typography.bodySmall,
                color = MaterialTheme.colorScheme.onSecondaryContainer
            )
        }
    }
}
```

5. Verifica los composables usando la vista **Preview** de Android Studio (panel lateral derecho o `@Preview`).

#### Resultado esperado

Los tres composables se renderizan correctamente en la vista Preview: `TarjetaPerfil` muestra un avatar circular con nombre y descripción, `BotonPrimario` ocupa el ancho completo con esquinas redondeadas, e `IndicadorEstado` muestra un cuadro con proporción 1.5:1.

#### Verificación

- Cada archivo compila sin errores
- Las `@Preview` se renderizan en el panel de diseño de Android Studio
- Los `Modifier` encadenados producen el layout visual esperado

---

### Bloque 4 — Estado Local y Formulario Interactivo (40 min)

**Objetivo:** Gestionar el estado local de la UI con `remember` y `mutableStateOf`, implementando un formulario con validación en tiempo real y comprendiendo el ciclo de recomposición.

#### Instrucciones

1. Crea el paquete `ui.formulario` dentro de `com.cursoadv.android.compose`.

2. Crea el archivo `FormularioRegistro.kt` con un formulario completo con validación:

```kotlin
package com.cursoadv.android.compose.ui.formulario

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Email
import androidx.compose.material.icons.filled.Lock
import androidx.compose.material.icons.filled.Person
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.input.PasswordVisualTransformation
import androidx.compose.ui.unit.dp

@Composable
fun FormularioRegistro() {
    // Estado local — cada variable dispara recomposición al cambiar
    var nombre by remember { mutableStateOf("") }
    var email by remember { mutableStateOf("") }
    var contrasena by remember { mutableStateOf("") }
    var confirmarContrasena by remember { mutableStateOf("") }
    var formularioEnviado by remember { mutableStateOf(false) }

    // Validaciones derivadas — se recalculan en cada recomposición
    val nombreValido = nombre.length >= 3
    val emailValido = android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
    val contrasenaValida = contrasena.length >= 8
    val contrasenasCoinciden = contrasena == confirmarContrasena && confirmarContrasena.isNotEmpty()
    val formularioValido = nombreValido && emailValido && contrasenaValida && contrasenasCoinciden

    Column(
        modifier = Modifier
            .fillMaxSize()
            .verticalScroll(rememberScrollState())
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "Registro de Usuario",
            style = MaterialTheme.typography.headlineMedium
        )

        Spacer(modifier = Modifier.height(24.dp))

        // Campo: Nombre
        OutlinedTextField(
            value = nombre,
            onValueChange = { nombre = it },
            label = { Text("Nombre completo") },
            leadingIcon = { Icon(Icons.Filled.Person, contentDescription = null) },
            isError = nombre.isNotEmpty() && !nombreValido,
            supportingText = {
                if (nombre.isNotEmpty() && !nombreValido) {
                    Text("Mínimo 3 caracteres")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(12.dp))

        // Campo: Email
        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Correo electrónico") },
            leadingIcon = { Icon(Icons.Filled.Email, contentDescription = null) },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
            isError = email.isNotEmpty() && !emailValido,
            supportingText = {
                if (email.isNotEmpty() && !emailValido) {
                    Text("Correo electrónico inválido")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(12.dp))

        // Campo: Contraseña
        OutlinedTextField(
            value = contrasena,
            onValueChange = { contrasena = it },
            label = { Text("Contraseña") },
            leadingIcon = { Icon(Icons.Filled.Lock, contentDescription = null) },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
            isError = contrasena.isNotEmpty() && !contrasenaValida,
            supportingText = {
                if (contrasena.isNotEmpty() && !contrasenaValida) {
                    Text("Mínimo 8 caracteres")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(12.dp))

        // Campo: Confirmar contraseña
        OutlinedTextField(
            value = confirmarContrasena,
            onValueChange = { confirmarContrasena = it },
            label = { Text("Confirmar contraseña") },
            leadingIcon = { Icon(Icons.Filled.Lock, contentDescription = null) },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
            isError = confirmarContrasena.isNotEmpty() && !contrasenasCoinciden,
            supportingText = {
                if (confirmarContrasena.isNotEmpty() && !contrasenasCoinciden) {
                    Text("Las contraseñas no coinciden")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(8.dp))

        // Indicador visual de progreso del formulario
        LinearProgressIndicator(
            progress = {
                listOf(nombreValido, emailValido, contrasenaValida, contrasenasCoinciden)
                    .count { it } / 4f
            },
            modifier = Modifier.fillMaxWidth(),
            color = if (formularioValido) MaterialTheme.colorScheme.primary
                    else MaterialTheme.colorScheme.outline
        )

        Spacer(modifier = Modifier.height(24.dp))

        Button(
            onClick = { formularioEnviado = true },
            enabled = formularioValido,
            modifier = Modifier
                .fillMaxWidth()
                .height(52.dp)
        ) {
            Text("Registrarse")
        }

        if (formularioEnviado) {
            Spacer(modifier = Modifier.height(16.dp))
            Card(
                colors = CardDefaults.cardColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                ),
                modifier = Modifier.fillMaxWidth()
            ) {
                Text(
                    text = "¡Bienvenido, $nombre! Registro exitoso.",
                    modifier = Modifier.padding(16.dp),
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
    }
}
```

3. Actualiza `MainActivity.kt` temporalmente para probar el formulario:

```kotlin
setContent {
    MaterialTheme {
        Surface(
            modifier = Modifier.fillMaxSize(),
            color = MaterialTheme.colorScheme.background
        ) {
            FormularioRegistro()
        }
    }
}
```

No olvides importar `com.cursoadv.android.compose.ui.formulario.FormularioRegistro`.

4. Ejecuta la app y prueba lo siguiente:
   - Escribe menos de 3 caracteres en el nombre → debe aparecer el mensaje de error
   - Escribe un email inválido → debe marcarse en rojo
   - Escribe contraseñas que no coincidan → debe indicar el error
   - Completa todo correctamente → la barra de progreso llega al 100% y el botón se habilita
   - Presiona "Registrarse" → aparece la tarjeta de confirmación

#### Resultado esperado

El formulario valida cada campo en tiempo real. La barra de progreso refleja cuántos campos son válidos. El botón solo se habilita cuando los 4 criterios se cumplen. Al enviar, aparece un mensaje de bienvenida.

#### Verificación

- Los mensajes de error aparecen y desaparecen reactivamente al modificar los campos
- La `LinearProgressIndicator` avanza proporcionalmente
- El botón cambia de deshabilitado a habilitado al completar la validación
- La tarjeta de confirmación aparece solo tras presionar el botón

---

### Bloque 5 — State Hoisting y Composables Stateless (35 min)

**Objetivo:** Refactorizar el formulario aplicando el patrón State Hoisting para separar el estado de la representación visual, creando composables stateless reutilizables y testables.

#### Instrucciones

1. Crea el archivo `FormularioRegistroState.kt` en el paquete `ui.formulario` para definir el estado como una data class:

```kotlin
package com.cursoadv.android.compose.ui.formulario

data class FormularioRegistroState(
    val nombre: String = "",
    val email: String = "",
    val contrasena: String = "",
    val confirmarContrasena: String = "",
    val formularioEnviado: Boolean = false
) {
    val nombreValido: Boolean get() = nombre.length >= 3
    val emailValido: Boolean
        get() = android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
    val contrasenaValida: Boolean get() = contrasena.length >= 8
    val contrasenasCoinciden: Boolean
        get() = contrasena == confirmarContrasena && confirmarContrasena.isNotEmpty()
    val formularioValido: Boolean
        get() = nombreValido && emailValido && contrasenaValida && contrasenasCoinciden
    val progreso: Float
        get() = listOf(nombreValido, emailValido, contrasenaValida, contrasenasCoinciden)
            .count { it } / 4f
}
```

2. Crea el archivo `FormularioRegistroEvents.kt` con una sealed interface para los eventos:

```kotlin
package com.cursoadv.android.compose.ui.formulario

sealed interface FormularioEvent {
    data class NombreCambiado(val valor: String) : FormularioEvent
    data class EmailCambiado(val valor: String) : FormularioEvent
    data class ContrasenaCambiada(val valor: String) : FormularioEvent
    data class ConfirmarContrasenaCambiada(val valor: String) : FormularioEvent
    data object Enviar : FormularioEvent
}
```

3. Crea el archivo `FormularioRegistroStateless.kt` — el composable **stateless** que recibe estado y emite eventos:

```kotlin
package com.cursoadv.android.compose.ui.formulario

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Email
import androidx.compose.material.icons.filled.Lock
import androidx.compose.material.icons.filled.Person
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.input.PasswordVisualTransformation
import androidx.compose.ui.unit.dp

@Composable
fun FormularioRegistroStateless(
    state: FormularioRegistroState,
    onEvent: (FormularioEvent) -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .verticalScroll(rememberScrollState())
            .padding(24.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "Registro de Usuario",
            style = MaterialTheme.typography.headlineMedium
        )

        Spacer(modifier = Modifier.height(24.dp))

        OutlinedTextField(
            value = state.nombre,
            onValueChange = { onEvent(FormularioEvent.NombreCambiado(it)) },
            label = { Text("Nombre completo") },
            leadingIcon = { Icon(Icons.Filled.Person, contentDescription = null) },
            isError = state.nombre.isNotEmpty() && !state.nombreValido,
            supportingText = {
                if (state.nombre.isNotEmpty() && !state.nombreValido) {
                    Text("Mínimo 3 caracteres")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(12.dp))

        OutlinedTextField(
            value = state.email,
            onValueChange = { onEvent(FormularioEvent.EmailCambiado(it)) },
            label = { Text("Correo electrónico") },
            leadingIcon = { Icon(Icons.Filled.Email, contentDescription = null) },
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
            isError = state.email.isNotEmpty() && !state.emailValido,
            supportingText = {
                if (state.email.isNotEmpty() && !state.emailValido) {
                    Text("Correo electrónico inválido")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(12.dp))

        OutlinedTextField(
            value = state.contrasena,
            onValueChange = { onEvent(FormularioEvent.ContrasenaCambiada(it)) },
            label = { Text("Contraseña") },
            leadingIcon = { Icon(Icons.Filled.Lock, contentDescription = null) },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
            isError = state.contrasena.isNotEmpty() && !state.contrasenaValida,
            supportingText = {
                if (state.contrasena.isNotEmpty() && !state.contrasenaValida) {
                    Text("Mínimo 8 caracteres")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(12.dp))

        OutlinedTextField(
            value = state.confirmarContrasena,
            onValueChange = { onEvent(FormularioEvent.ConfirmarContrasenaCambiada(it)) },
            label = { Text("Confirmar contraseña") },
            leadingIcon = { Icon(Icons.Filled.Lock, contentDescription = null) },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Password),
            isError = state.confirmarContrasena.isNotEmpty() && !state.contrasenasCoinciden,
            supportingText = {
                if (state.confirmarContrasena.isNotEmpty() && !state.contrasenasCoinciden) {
                    Text("Las contraseñas no coinciden")
                }
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(8.dp))

        LinearProgressIndicator(
            progress = { state.progreso },
            modifier = Modifier.fillMaxWidth(),
            color = if (state.formularioValido) MaterialTheme.colorScheme.primary
                    else MaterialTheme.colorScheme.outline
        )

        Spacer(modifier = Modifier.height(24.dp))

        Button(
            onClick = { onEvent(FormularioEvent.Enviar) },
            enabled = state.formularioValido,
            modifier = Modifier
                .fillMaxWidth()
                .height(52.dp)
        ) {
            Text("Registrarse")
        }

        if (state.formularioEnviado) {
            Spacer(modifier = Modifier.height(16.dp))
            Card(
                colors = CardDefaults.cardColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer
                ),
                modifier = Modifier.fillMaxWidth()
            ) {
                Text(
                    text = "¡Bienvenido, ${state.nombre}! Registro exitoso.",
                    modifier = Modifier.padding(16.dp),
                    style = MaterialTheme.typography.bodyLarge
                )
            }
        }
    }
}
```

4. Crea el archivo `FormularioRegistroContainer.kt` — el composable **stateful** que contiene el estado y lo pasa al stateless:

```kotlin
package com.cursoadv.android.compose.ui.formulario

import androidx.compose.runtime.*

@Composable
fun FormularioRegistroContainer() {
    var state by remember { mutableStateOf(FormularioRegistroState()) }

    FormularioRegistroStateless(
        state = state,
        onEvent = { event ->
            state = when (event) {
                is FormularioEvent.NombreCambiado ->
                    state.copy(nombre = event.valor)
                is FormularioEvent.EmailCambiado ->
                    state.copy(email = event.valor)
                is FormularioEvent.ContrasenaCambiada ->
                    state.copy(contrasena = event.valor)
                is FormularioEvent.ConfirmarContrasenaCambiada ->
                    state.copy(confirmarContrasena = event.valor)
                FormularioEvent.Enviar ->
                    state.copy(formularioEnviado = true)
            }
        }
    )
}
```

5. Actualiza `MainActivity.kt` para usar el contenedor:

```kotlin
import com.cursoadv.android.compose.ui.formulario.FormularioRegistroContainer

// Dentro de setContent:
FormularioRegistroContainer()
```

6. Ejecuta y verifica que el comportamiento es idéntico al del Bloque 4.

#### Resultado esperado

El formulario funciona exactamente igual que antes, pero ahora `FormularioRegistroStateless` es un composable puro que no gestiona estado propio — todo el estado se eleva al contenedor. Esto hace que el composable stateless sea testable con cualquier estado inyectado.

#### Verificación

- `FormularioRegistroStateless` no contiene ningún `remember` ni `mutableStateOf`
- `FormularioRegistroContainer` es el único punto de gestión de estado
- La funcionalidad es idéntica a la versión anterior
- La data class `FormularioRegistroState` encapsula toda la lógica de validación

---

### Bloque 6 — Tema Material 3 Personalizado con Soporte Claro/Oscuro (35 min)

**Objetivo:** Crear un sistema de diseño Material 3 personalizado con paleta de colores, tipografía y soporte de tema claro/oscuro usando `isSystemInDarkTheme()`.

#### Instrucciones

1. Crea el paquete `ui.theme` dentro de `com.cursoadv.android.compose`.

2. Crea el archivo `Color.kt`:

```kotlin
package com.cursoadv.android.compose.ui.theme

import androidx.compose.ui.graphics.Color

// Paleta clara
val PrimaryLight = Color(0xFF1A6B52)
val OnPrimaryLight = Color(0xFFFFFFFF)
val PrimaryContainerLight = Color(0xFFA4F2D3)
val OnPrimaryContainerLight = Color(0xFF002117)

val SecondaryLight = Color(0xFF4C6359)
val OnSecondaryLight = Color(0xFFFFFFFF)
val SecondaryContainerLight = Color(0xFFCEE9DB)
val OnSecondaryContainerLight = Color(0xFF082018)

val TertiaryLight = Color(0xFF3F6374)
val OnTertiaryLight = Color(0xFFFFFFFF)
val TertiaryContainerLight = Color(0xFFC2E8FC)
val OnTertiaryContainerLight = Color(0xFF001F2A)

val BackgroundLight = Color(0xFFFBFDF9)
val OnBackgroundLight = Color(0xFF191C1A)
val SurfaceLight = Color(0xFFFBFDF9)
val OnSurfaceLight = Color(0xFF191C1A)
val SurfaceVariantLight = Color(0xFFDBE5DE)
val OnSurfaceVariantLight = Color(0xFF404944)

val ErrorLight = Color(0xFFBA1A1A)
val OnErrorLight = Color(0xFFFFFFFF)

// Paleta oscura
val PrimaryDark = Color(0xFF88D6B8)
val OnPrimaryDark = Color(0xFF00382A)
val PrimaryContainerDark = Color(0xFF00513D)
val OnPrimaryContainerDark = Color(0xFFA4F2D3)

val SecondaryDark = Color(0xFFB3CCC0)
val OnSecondaryDark = Color(0xFF1E352C)
val SecondaryContainerDark = Color(0xFF354C42)
val OnSecondaryContainerDark = Color(0xFFCEE9DB)

val TertiaryDark = Color(0xFFA7CCE0)
val OnTertiaryDark = Color(0xFF0B3444)
val TertiaryContainerDark = Color(0xFF274B5C)
val OnTertiaryContainerDark = Color(0xFFC2E8FC)

val BackgroundDark = Color(0xFF191C1A)
val OnBackgroundDark = Color(0xFFE1E3DF)
val SurfaceDark = Color(0xFF191C1A)
val OnSurfaceDark = Color(0xFFE1E3DF)
val SurfaceVariantDark = Color(0xFF404944)
val OnSurfaceVariantDark = Color(0xFFBFC9C2)

val ErrorDark = Color(0xFFFFB4AB)
val OnErrorDark = Color(0xFF690005)
```

3. Crea el archivo `Type.kt`:

```kotlin
package com.cursoadv.android.compose.ui.theme

import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp

val AppTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp
    ),
    headlineMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.SemiBold,
        fontSize = 28.sp,
        lineHeight = 36.sp
    ),
    titleMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    ),
    bodyMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp
    ),
    labelLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp
    )
)
```

4. Crea el archivo `Theme.kt`:

```kotlin
package com.cursoadv.android.compose.ui.theme

import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.runtime.Composable

private val LightColorScheme = lightColorScheme(
    primary = PrimaryLight,
    onPrimary = OnPrimaryLight,
    primaryContainer = PrimaryContainerLight,
    onPrimaryContainer = OnPrimaryContainerLight,
    secondary = SecondaryLight,
    onSecondary = OnSecondaryLight,
    secondaryContainer = SecondaryContainerLight,
    onSecondaryContainer = OnSecondaryContainerLight,
    tertiary = TertiaryLight,
    onTertiary = OnTertiaryLight,
    tertiaryContainer = TertiaryContainerLight,
    onTertiaryContainer = OnTertiaryContainerLight,
    background = BackgroundLight,
    onBackground = OnBackgroundLight,
    surface = SurfaceLight,
    onSurface = OnSurfaceLight,
    surfaceVariant = SurfaceVariantLight,
    onSurfaceVariant = OnSurfaceVariantLight,
    error = ErrorLight,
    onError = OnErrorLight
)

private val DarkColorScheme = darkColorScheme(
    primary = PrimaryDark,
    onPrimary = OnPrimaryDark,
    primaryContainer = PrimaryContainerDark,
    onPrimaryContainer = OnPrimaryContainerDark,
    secondary = SecondaryDark,
    onSecondary = OnSecondaryDark,
    secondaryContainer = SecondaryContainerDark,
    onSecondaryContainer = OnSecondaryContainerDark,
    tertiary = TertiaryDark,
    onTertiary = OnTertiaryDark,
    tertiaryContainer = TertiaryContainerDark,
    onTertiaryContainer = OnTertiaryContainerDark,
    background = BackgroundDark,
    onBackground = OnBackgroundDark,
    surface = SurfaceDark,
    onSurface = OnSurfaceDark,
    surfaceVariant = SurfaceVariantDark,
    onSurfaceVariant = OnSurfaceVariantDark,
    error = ErrorDark,
    onError = OnErrorDark
)

@Composable
fun ComposeUITheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        content = content
    )
}
```

5. Actualiza `MainActivity.kt` para usar el tema personalizado:

```kotlin
package com.cursoadv.android.compose

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.Surface
import androidx.compose.material3.MaterialTheme
import androidx.compose.ui.Modifier
import com.cursoadv.android.compose.ui.formulario.FormularioRegistroContainer
import com.cursoadv.android.compose.ui.theme.ComposeUITheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ComposeUITheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    FormularioRegistroContainer()
                }
            }
        }
    }
}
```

6. Ejecuta la app. Cambia el tema del dispositivo/AVD a modo oscuro (**Settings → Display → Dark theme**) y verifica que los colores cambian automáticamente.

#### Resultado esperado

La app muestra una paleta verde personalizada en modo claro. Al activar el modo oscuro del sistema, los colores cambian automáticamente a la paleta oscura definida. La tipografía personalizada se aplica en todos los composables que usan `MaterialTheme.typography`.

#### Verificación

- Los colores primarios son tonos verdes (no el púrpura predeterminado de Material 3)
- El cambio entre modo claro y oscuro es automático e inmediato
- Los textos usan los estilos tipográficos definidos en `AppTypography`

---

### Bloque 7 — Navegación Multi-Pantalla con Compose Navigation (45 min)

**Objetivo:** Implementar navegación entre tres pantallas (Lista, Detalle, Perfil) usando `NavController`, `NavHost` y argumentos tipados con gestión del back stack.

#### Instrucciones

1. Crea el paquete `ui.navigation` dentro de `com.cursoadv.android.compose`.

2. Crea el archivo `Destinos.kt` con las rutas de navegación como constantes tipadas:

```kotlin
package com.cursoadv.android.compose.ui.navigation

object Destinos {
    const val LISTA = "lista"
    const val DETALLE = "detalle/{itemId}"
    const val PERFIL = "perfil"

    fun detalleConId(itemId: Int): String = "detalle/$itemId"
}
```

3. Crea el paquete `ui.screens`. Crea el archivo `PantallaLista.kt`:

```kotlin
package com.cursoadv.android.compose.ui.screens

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Person
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.nestedscroll.nestedScroll
import androidx.compose.ui.unit.dp

data class ElementoLista(
    val id: Int,
    val titulo: String,
    val subtitulo: String,
    val imagenUrl: String
)

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaLista(
    elementos: List<ElementoLista>,
    onElementoClick: (Int) -> Unit,
    onPerfilClick: () -> Unit
) {
    val scrollBehavior = TopAppBarDefaults.pinnedScrollBehavior()

    Scaffold(
        modifier = Modifier.nestedScroll(scrollBehavior.nestedScrollConnection),
        topBar = {
            TopAppBar(
                title = { Text("Explorar") },
                actions = {
                    IconButton(onClick = onPerfilClick) {
                        Icon(Icons.Filled.Person, contentDescription = "Perfil")
                    }
                },
                scrollBehavior = scrollBehavior
            )
        }
    ) { paddingValues ->
        if (elementos.isEmpty()) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(paddingValues),
                contentAlignment = androidx.compose.ui.Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        } else {
            LazyColumn(
                modifier = Modifier.padding(paddingValues),
                contentPadding = PaddingValues(vertical = 8.dp)
            ) {
                items(
                    items = elementos,
                    key = { it.id }
                ) { elemento ->
                    TarjetaElemento(
                        elemento = elemento,
                        onClick = { onElementoClick(elemento.id) }
                    )
                }
            }
        }
    }
}

@Composable
private fun TarjetaElemento(
    elemento: ElementoLista,
    onClick: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 4.dp)
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        ) {
            // Placeholder para imagen — se reemplazará con Coil en Bloque 8
            Surface(
                modifier = Modifier.size(72.dp),
                shape = MaterialTheme.shapes.medium,
                color = MaterialTheme.colorScheme.secondaryContainer
            ) {
                Box(contentAlignment = androidx.compose.ui.Alignment.Center) {
                    Text(
                        text = "${elemento.id}",
                        style = MaterialTheme.typography.headlineSmall,
                        color = MaterialTheme.colorScheme.onSecondaryContainer
                    )
                }
            }

            Spacer(modifier = Modifier.width(16.dp))

            Column(modifier = Modifier.weight(1f)) {
                Text(
                    text = elemento.titulo,
                    style = MaterialTheme.typography.titleMedium
                )
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = elemento.subtitulo,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }
}
```

4. Crea el archivo `PantallaDetalle.kt`:

```kotlin
package com.cursoadv.android.compose.ui.screens

import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaDetalle(
    itemId: Int,
    elemento: ElementoLista?,
    onVolverClick: () -> Unit
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Detalle") },
                navigationIcon = {
                    IconButton(onClick = onVolverClick) {
                        Icon(
                            Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
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
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            if (elemento != null) {
                // Imagen placeholder
                Surface(
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(200.dp),
                    shape = MaterialTheme.shapes.large,
                    color = MaterialTheme.colorScheme.primaryContainer
                ) {
                    Box(contentAlignment = Alignment.Center) {
                        Text(
                            text = "Imagen #${elemento.id}",
                            style = MaterialTheme.typography.headlineMedium,
                            color = MaterialTheme.colorScheme.onPrimaryContainer
                        )
                    }
                }

                Spacer(modifier = Modifier.height(24.dp))

                Text(
                    text = elemento.titulo,
                    style = MaterialTheme.typography.headlineMedium,
                    fontWeight = FontWeight.Bold
                )

                Spacer(modifier = Modifier.height(8.dp))

                Text(
                    text = elemento.subtitulo,
                    style = MaterialTheme.typography.bodyLarge,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )

                Spacer(modifier = Modifier.height(16.dp))

                Text(
                    text = "Este es el contenido detallado del elemento con ID $itemId. " +
                           "Aquí se mostraría información completa proveniente de una API " +
                           "o base de datos local en las prácticas posteriores.",
                    style = MaterialTheme.typography.bodyMedium
                )
            } else {
                Text(
                    text = "Elemento no encontrado",
                    style = MaterialTheme.typography.headlineSmall
                )
            }
        }
    }
}
```

5. Crea el archivo `PantallaPerfil.kt`:

```kotlin
package com.cursoadv.android.compose.ui.screens

import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.cursoadv.android.compose.ui.components.IndicadorEstado
import com.cursoadv.android.compose.ui.components.TarjetaPerfil

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaPerfil(
    onVolverClick: () -> Unit
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Mi Perfil") },
                navigationIcon = {
                    IconButton(onClick = onVolverClick) {
                        Icon(
                            Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
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
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            TarjetaPerfil(
                nombre = "Estudiante Android",
                descripcion = "Curso Avanzado de Kotlin y Compose",
                imagenUrl = null
            )

            Spacer(modifier = Modifier.height(24.dp))

            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.spacedBy(12.dp)
            ) {
                IndicadorEstado(
                    etiqueta = "Prácticas",
                    valor = "2/5",
                    modifier = Modifier.weight(1f)
                )
                IndicadorEstado(
                    etiqueta = "Nivel",
                    valor = "Avanzado",
                    modifier = Modifier.weight(1f)
                )
            }

            Spacer(modifier = Modifier.height(24.dp))

            // Aquí se reutiliza el formulario con State Hoisting
            Text(
                text = "Editar Perfil",
                style = MaterialTheme.typography.titleMedium,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(bottom = 8.dp)
            )
        }
    }
}
```

6. Crea el archivo `AppNavigation.kt` en el paquete `ui.navigation`:

```kotlin
package com.cursoadv.android.compose.ui.navigation

import androidx.compose.runtime.*
import androidx.navigation.NavHostController
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument
import com.cursoadv.android.compose.ui.screens.ElementoLista
import com.cursoadv.android.compose.ui.screens.PantallaDetalle
import com.cursoadv.android.compose.ui.screens.PantallaLista
import com.cursoadv.android.compose.ui.screens.PantallaPerfil

@Composable
fun AppNavigation(
    navController: NavHostController = rememberNavController()
) {
    // Datos simulados — en prácticas posteriores vendrán de un ViewModel/API
    val elementosSimulados = remember {
        (1..20).map { i ->
            ElementoLista(
                id = i,
                titulo = "Elemento #$i",
                subtitulo = "Descripción breve del elemento número $i",
                imagenUrl = "https://picsum.photos/seed/$i/200/200"
            )
        }
    }

    NavHost(
        navController = navController,
        startDestination = Destinos.LISTA
    ) {
        composable(Destinos.LISTA) {
            PantallaLista(
                elementos = elementosSimulados,
                onElementoClick = { itemId ->
                    navController.navigate(Destinos.detalleConId(itemId))
                },
                onPerfilClick = {
                    navController.navigate(Destinos.PERFIL)
                }
            )
        }

        composable(
            route = Destinos.DETALLE,
            arguments = listOf(
                navArgument("itemId") { type = NavType.IntType }
            )
        ) { backStackEntry ->
            val itemId = backStackEntry.arguments?.getInt("itemId") ?: 0
            val elemento = elementosSimulados.find { it.id == itemId }
            PantallaDetalle(
                itemId = itemId,
                elemento = elemento,
                onVolverClick = { navController.popBackStack() }
            )
        }

        composable(Destinos.PERFIL) {
            PantallaPerfil(
                onVolverClick = { navController.popBackStack() }
            )
        }
    }
}
```

7. Actualiza `MainActivity.kt` para usar la navegación:

```kotlin
package com.cursoadv.android.compose

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import com.cursoadv.android.compose.ui.navigation.AppNavigation
import com.cursoadv.android.compose.ui.theme.ComposeUITheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ComposeUITheme {
                AppNavigation()
            }
        }
    }
}
```

8. Ejecuta la app y verifica:
   - La pantalla de lista aparece como destino inicial
   - Al tocar un elemento, navega a la pantalla de detalle con el ID correcto
   - El botón de retroceso (flecha) en detalle regresa a la lista
   - El ícono de perfil en la barra superior navega a la pantalla de perfil
   - El botón de retroceso del sistema (hardware/gesto) funciona correctamente

#### Resultado esperado

La app navega fluidamente entre tres pantallas. Los argumentos se pasan correctamente (el ID del elemento aparece en la pantalla de detalle). El back stack se gestiona sin duplicados.

#### Verificación

- Navegar a Detalle #5 muestra "Elemento #5" con su descripción
- Presionar back desde Detalle regresa a Lista
- Navegar a Perfil y presionar back regresa a Lista
- No se acumulan pantallas duplicadas en el back stack

---

### Bloque 8 — Lista Dinámica con LazyColumn, Coil y StateFlow (48 min)

**Objetivo:** Construir una pantalla de lista con `LazyColumn`, tarjetas `Card` con imágenes cargadas por Coil (`AsyncImage`), datos dinámicos provenientes de un `StateFlow` simulado, y reemplazar los datos estáticos del bloque anterior.

#### Instrucciones

1. Crea el paquete `data` dentro de `com.cursoadv.android.compose`.

2. Crea el archivo `ElementoRepository.kt` con un repositorio simulado que emite datos mediante Flow:

```kotlin
package com.cursoadv.android.compose.data

import com.cursoadv.android.compose.ui.screens.ElementoLista
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow

class ElementoRepository {

    fun obtenerElementos(): Flow<List<ElementoLista>> = flow {
        // Simula latencia de red
        delay(1500)

        val elementos = (1..25).map { i ->
            ElementoLista(
                id = i,
                titulo = "Destino #$i",
                subtitulo = generarDescripcion(i),
                imagenUrl = "https://picsum.photos/seed/destino$i/400/300"
            )
        }
        emit(elementos)
    }

    private fun generarDescripcion(id: Int): String {
        val descripciones = listOf(
            "Un lugar fascinante para explorar",
            "Paisajes naturales impresionantes",
            "Cultura y gastronomía únicas",
            "Aventura y deportes extremos",
            "Historia y arquitectura milenaria"
        )
        return descripciones[(id - 1) % descripciones.size]
    }
}
```

3. Crea el paquete `viewmodel`. Crea el archivo `ListaViewModel.kt`:

```kotlin
package com.cursoadv.android.compose.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.cursoadv.android.compose.data.ElementoRepository
import com.cursoadv.android.compose.ui.screens.ElementoLista
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.launch

data class ListaUiState(
    val elementos: List<ElementoLista> = emptyList(),
    val cargando: Boolean = true,
    val error: String? = null
)

class ListaViewModel : ViewModel() {

    private val repository = ElementoRepository()

    private val _uiState = MutableStateFlow(ListaUiState())
    val uiState: StateFlow<ListaUiState> = _uiState.asStateFlow()

    init {
        cargarElementos()
    }

    private fun cargarElementos() {
        viewModelScope.launch {
            _uiState.value = ListaUiState(cargando = true)

            repository.obtenerElementos()
                .catch { e ->
                    _uiState.value = ListaUiState(
                        cargando = false,
                        error = e.message ?: "Error desconocido"
                    )
                }
                .collect { elementos ->
                    _uiState.value = ListaUiState(
                        elementos = elementos,
                        cargando = false
                    )
                }
        }
    }

    fun recargar() {
        cargarElementos()
    }
}
```

4. Actualiza `PantallaLista.kt` para usar `AsyncImage` de Coil y aceptar el estado de carga:

```kotlin
package com.cursoadv.android.compose.ui.screens

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Person
import androidx.compose.material.icons.filled.Refresh
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.input.nestedscroll.nestedScroll
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.unit.dp
import coil3.compose.AsyncImage
import com.cursoadv.android.compose.viewmodel.ListaUiState

data class ElementoLista(
    val id: Int,
    val titulo: String,
    val subtitulo: String,
    val imagenUrl: String
)

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PantallaLista(
    uiState: ListaUiState,
    onElementoClick: (Int) -> Unit,
    onPerfilClick: () -> Unit,
    onRecargar: () -> Unit
) {
    val scrollBehavior = TopAppBarDefaults.pinnedScrollBehavior()

    Scaffold(
        modifier = Modifier.nestedScroll(scrollBehavior.nestedScrollConnection),
        topBar = {
            TopAppBar(
                title = { Text("Explorar Destinos") },
                actions = {
                    if (!uiState.cargando) {
                        IconButton(onClick = onRecargar) {
                            Icon(Icons.Filled.Refresh, contentDescription = "Recargar")
                        }
                    }
                    IconButton(onClick = onPerfilClick) {
                        Icon(Icons.Filled.Person, contentDescription = "Perfil")
                    }
                },
                scrollBehavior = scrollBehavior
            )
        }
    ) { paddingValues ->
        when {
            uiState.cargando -> {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues),
                    contentAlignment = Alignment.Center
                ) {
                    Column(horizontalAlignment = Alignment.CenterHorizontally) {
                        CircularProgressIndicator()
                        Spacer(modifier = Modifier.height(16.dp))
                        Text("Cargando destinos...")
                    }
                }
            }

            uiState.error != null -> {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues),
                    contentAlignment = Alignment.Center
                ) {
                    Column(horizontalAlignment = Alignment.CenterHorizontally) {
                        Text(
                            text = "Error: ${uiState.error}",
                            color = MaterialTheme.colorScheme.error
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                        Button(onClick = onRecargar) {
                            Text("Reintentar")
                        }
                    }
                }
            }

            else -> {
                LazyColumn(
                    modifier = Modifier.padding(paddingValues),
                    contentPadding = PaddingValues(vertical = 8.dp)
                ) {
                    items(
                        items = uiState.elementos,
                        key = { it.id }
                    ) { elemento ->
                        TarjetaElementoConImagen(
                            elemento = elemento,
                            onClick = { onElementoClick(elemento.id) }
                        )
                    }
                }
            }
        }
    }
}

@Composable
private fun TarjetaElementoConImagen(
    elemento: ElementoLista,
    onClick: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 6.dp)
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(12.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
