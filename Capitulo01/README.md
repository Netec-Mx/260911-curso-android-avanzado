# Kotlin Avanzado — Sealed Classes, Corrutinas, Flows y Programación Funcional

## 1. Metadatos

| Campo | Valor |
|---|---|
| **Duración** | 216 minutos (≈ 3 h 36 min) |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar |

---

## 2. Descripción General

En esta práctica crearás desde cero un proyecto Android con Jetpack Compose que sirve como laboratorio integral de Kotlin avanzado. A lo largo de seis bloques progresivos construirás una jerarquía `Result<T>` con sealed classes, aplicarás scope functions y funciones de orden superior, implementarás corrutinas con distintos Dispatchers, gestionarás estado reactivo con `StateFlow` y `SharedFlow`, manejarás cancelación y excepciones, y finalmente construirás un pipeline de datos reactivo con `Flow`. El proyecto resultante será la referencia técnica para las prácticas 2 a 5 del curso.

---

## 3. Objetivos de Aprendizaje

Al finalizar esta práctica serás capaz de:

- [ ] Modelar estados y resultados de operaciones usando `sealed class`, `enum class` y genéricos en Kotlin, creando una jerarquía `Result<T>` propia.
- [ ] Escribir código idiomático con lambdas, funciones de orden superior y las cinco scope functions (`let`, `run`, `with`, `apply`, `also`).
- [ ] Crear y gestionar corrutinas con `launch`, `async`/`await` y distintos `Dispatchers`, aplicando cancelación estructurada y `CoroutineExceptionHandler`.
- [ ] Implementar `StateFlow` para estado de UI y `SharedFlow` para eventos únicos, integrándolos con un `ViewModel` de Compose.
- [ ] Construir un pipeline reactivo con `Flow` usando operadores `map`, `filter`, `catch` y `flowOn`, visualizando los resultados en una interfaz Compose mínima.

---

## 4. Prerrequisitos

### Conocimientos previos

| Requisito | Nivel esperado |
|---|---|
| Kotlin básico (clases, interfaces, herencia, null safety) | Sólido |
| Android Studio y creación de proyectos Android | Intermedio |
| Programación asíncrona y problema del hilo principal | Conceptual |
| Jetpack Compose básico (composables, `@Composable`) | Básico |

### Acceso y cuentas

- Android Studio **Quail 3 (2026.1.3 Patch 1)** instalado y funcional.
- SDK Platform **API 36** y **API 37** descargados desde SDK Manager.
- Conexión a Internet estable para descarga de dependencias Gradle.
- Al menos un AVD configurado (API 30, 35, 36 o 37) o un dispositivo físico con Android 11+.

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
| kotlinx-coroutines-android | 1.10.2 |
| Lifecycle Runtime KTX | 2.6.1 |
| Lifecycle ViewModel Compose | (gestionada por BOM) |

### Preparación del directorio de trabajo

Abre una terminal y ejecuta:

```bash
mkdir -p ~/AndroidStudioProjects/AvanzadoKotlin/Practica1_KotlinAvanzado
```

> **Nota:** Todo el proyecto se creará dentro de `~/AndroidStudioProjects/AvanzadoKotlin/Practica1_KotlinAvanzado`.

---

## 6. Instrucciones Paso a Paso

### Paso 1 — Crear el proyecto en Android Studio

**Objetivo:** Generar el proyecto base con la plantilla Empty Activity (Compose) y configurar el baseline tecnológico completo.

**Instrucciones:**

1. Abre **Android Studio Quail 3**.
2. Selecciona **File → New → New Project**.
3. En la lista de plantillas, elige **Empty Activity** (la plantilla de Jetpack Compose, no Empty Views Activity).
4. Configura los campos:
   - **Name:** `Practica1_KotlinAvanzado`
   - **Package name:** `com.cursoadv.android.kotlin`
   - **Save location:** `~/AndroidStudioProjects/AvanzadoKotlin/Practica1_KotlinAvanzado`
   - **Minimum SDK:** API 30 (Android 11)
   - **Build configuration language:** Kotlin DSL (build.gradle.kts)
5. Haz clic en **Finish** y espera a que Gradle sincronice.

6. Abre el archivo `gradle/libs.versions.toml` y reemplaza **todo** su contenido con lo siguiente:

```toml
[versions]
agp = "9.3.2"
kotlin = "2.2.10"
ksp = "2.2.10-1.0.31"
composeBom = "2026.02.01"
activityCompose = "1.13.0"
coreKtx = "1.19.0"
lifecycleRuntimeKtx = "2.6.1"
coroutines = "1.10.2"
junit = "4.13.2"
androidxJunit = "1.3.0"
espressoCore = "3.7.0"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-compose-ui = { group = "androidx.compose.ui", name = "ui" }
androidx-compose-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
androidx-compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3" }
androidx-compose-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
androidx-compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose" }
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "coroutines" }
kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "coroutines" }
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidxJunit" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

7. Abre el archivo **`build.gradle.kts`** de nivel **proyecto** (raíz) y verifica que contiene:

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.ksp) apply false
}
```

8. Abre el archivo **`build.gradle.kts`** de nivel **módulo** (`app/build.gradle.kts`) y reemplázalo con:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.cursoadv.android.kotlin"
    compileSdk = 37

    defaultConfig {
        applicationId = "com.cursoadv.android.kotlin"
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
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.ui.graphics)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.compose.material3)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.viewmodel.compose)
    implementation(libs.kotlinx.coroutines.android)
    implementation(libs.kotlinx.coroutines.core)

    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)

    debugImplementation(libs.androidx.compose.ui.tooling)
    debugImplementation(libs.androidx.compose.ui.test.manifest)
}
```

9. Haz clic en **Sync Now** en la barra amarilla superior. Espera a que la sincronización termine sin errores.

**Resultado esperado:**

```
BUILD SUCCESSFUL in Xs
```

La ventana **Build** de Android Studio muestra sincronización exitosa sin errores ni warnings de versiones.

**Verificación:**

- En la pestaña **Build** no aparecen errores.
- En **File → Project Structure → Modules**, `compileSdk` muestra `37`, `minSdk` muestra `30`.
- No existe ninguna referencia a `kotlinCompilerExtensionVersion` (el plugin `kotlin.compose` lo gestiona automáticamente con Kotlin 2.x).

---

### Paso 2 — Sealed Classes y Enum Classes para modelar estados

**Objetivo:** Crear una jerarquía `Result<T>` propia con sealed classes y un enum class para categorías de operación, estableciendo el patrón de modelado de estados que se usará en todo el curso.

**Instrucciones:**

1. En el panel de proyecto, haz clic derecho sobre el paquete `com.cursoadv.android.kotlin` → **New → Package**. Nombra el paquete `model`.

2. Dentro de `com.cursoadv.android.kotlin.model`, crea un nuevo archivo Kotlin llamado **`OperationResult.kt`** con el siguiente contenido:

```kotlin
package com.cursoadv.android.kotlin.model

/**
 * Sealed class genérica que modela el resultado de cualquier operación.
 * Demuestra: sealed class, genéricos, data class, object.
 */
sealed class OperationResult<out T> {

    /** Estado de carga — no contiene datos */
    data object Loading : OperationResult<Nothing>()

    /** Operación exitosa — contiene el dato de tipo T */
    data class Success<T>(val data: T) : OperationResult<T>()

    /** Operación fallida — contiene el mensaje y opcionalmente la excepción */
    data class Error(
        val message: String,
        val exception: Throwable? = null
    ) : OperationResult<Nothing>()
}
```

3. En el mismo paquete `model`, crea **`TaskCategory.kt`**:

```kotlin
package com.cursoadv.android.kotlin.model

/**
 * Enum class que categoriza las tareas simuladas.
 * Demuestra: enum class con propiedades y funciones miembro.
 */
enum class TaskCategory(val displayName: String, val priority: Int) {
    NETWORK("Red", 1),
    DATABASE("Base de datos", 2),
    COMPUTATION("Cómputo", 3),
    UI("Interfaz", 4);

    fun isHighPriority(): Boolean = priority <= 2
}
```

4. En el mismo paquete `model`, crea **`Task.kt`**:

```kotlin
package com.cursoadv.android.kotlin.model

/**
 * Data class que representa una tarea simulada.
 */
data class Task(
    val id: Int,
    val name: String,
    val category: TaskCategory,
    val durationMs: Long,
    val completed: Boolean = false
)
```

5. Crea un nuevo paquete `util` dentro de `com.cursoadv.android.kotlin`. Dentro de él, crea **`ResultExtensions.kt`**:

```kotlin
package com.cursoadv.android.kotlin.util

import com.cursoadv.android.kotlin.model.OperationResult

/**
 * Funciones de extensión sobre OperationResult.
 * Demuestra: extension functions, when exhaustivo, genéricos.
 */
fun <T> OperationResult<T>.toDisplayString(): String = when (this) {
    is OperationResult.Loading -> "⏳ Cargando..."
    is OperationResult.Success -> "✅ Éxito: $data"
    is OperationResult.Error -> "❌ Error: $message"
}

/**
 * Transforma el dato contenido en Success sin alterar Loading ni Error.
 * Demuestra: función de orden superior con genéricos.
 */
inline fun <T, R> OperationResult<T>.mapResult(
    transform: (T) -> R
): OperationResult<R> = when (this) {
    is OperationResult.Loading -> OperationResult.Loading
    is OperationResult.Success -> OperationResult.Success(transform(data))
    is OperationResult.Error -> OperationResult.Error(message, exception)
}
```

**Resultado esperado:**

El proyecto compila sin errores. La estructura de paquetes se ve así:

```
com.cursoadv.android.kotlin
├── model
│   ├── OperationResult.kt
│   ├── Task.kt
│   └── TaskCategory.kt
├── util
│   └── ResultExtensions.kt
└── MainActivity.kt
```

**Verificación:**

- Haz **Build → Rebuild Project**. La compilación debe completarse con `BUILD SUCCESSFUL`.
- Verifica que `OperationResult` tiene exactamente tres subtipos y que el `when` en `toDisplayString` es exhaustivo (el IDE no muestra advertencia de ramas faltantes).

---

### Paso 3 — Lambdas, funciones de orden superior y scope functions

**Objetivo:** Implementar un repositorio simulado que demuestre el uso idiomático de lambdas, funciones de orden superior y las cinco scope functions de Kotlin.

**Instrucciones:**

1. Crea el paquete `data` dentro de `com.cursoadv.android.kotlin`.

2. Dentro de `data`, crea **`TaskRepository.kt`**:

```kotlin
package com.cursoadv.android.kotlin.data

import com.cursoadv.android.kotlin.model.Task
import com.cursoadv.android.kotlin.model.TaskCategory

/**
 * Repositorio simulado que demuestra lambdas, HOF y scope functions.
 */
class TaskRepository {

    // ── Datos de ejemplo ──────────────────────────────────────────────
    private val tasks: List<Task> = buildList {
        // 'apply' — configura el builder implícitamente
        add(Task(1, "Llamada API usuarios", TaskCategory.NETWORK, 1500L))
        add(Task(2, "Consulta Room local", TaskCategory.DATABASE, 800L))
        add(Task(3, "Cálculo de estadísticas", TaskCategory.COMPUTATION, 2000L))
        add(Task(4, "Renderizar gráfico", TaskCategory.UI, 300L))
        add(Task(5, "Sincronizar inventario", TaskCategory.NETWORK, 2500L))
        add(Task(6, "Migración de esquema", TaskCategory.DATABASE, 1200L))
        add(Task(7, "Compresión de imágenes", TaskCategory.COMPUTATION, 3000L))
        add(Task(8, "Animación de transición", TaskCategory.UI, 450L))
    }

    // ── Función de orden superior: filtra tareas por predicado ────────
    fun filterTasks(predicate: (Task) -> Boolean): List<Task> {
        return tasks.filter(predicate)
    }

    // ── Función de orden superior: transforma cada tarea ──────────────
    fun <R> transformTasks(transform: (Task) -> R): List<R> {
        return tasks.map(transform)
    }

    // ── Demostración de las cinco scope functions ─────────────────────

    /**
     * 'let' — ejecuta bloque solo si el valor no es null.
     * Retorna el resultado del bloque.
     */
    fun findTaskById(id: Int): String {
        return tasks.firstOrNull { it.id == id }
            ?.let { task ->
                "Encontrada: ${task.name} [${task.category.displayName}]"
            } ?: "Tarea con id=$id no encontrada"
    }

    /**
     * 'run' — ejecuta bloque en el contexto del objeto.
     * Retorna el resultado del bloque.
     */
    fun getTaskSummary(): String = tasks.run {
        "Total: $size tareas | " +
            "Completadas: ${count { it.completed }} | " +
            "Pendientes: ${count { !it.completed }}"
    }

    /**
     * 'with' — similar a run pero se pasa el objeto como argumento.
     * Retorna el resultado del bloque.
     */
    fun describeCategory(category: TaskCategory): String = with(category) {
        "Categoría: $displayName | Prioridad: $priority | Alta prioridad: ${isHighPriority()}"
    }

    /**
     * 'apply' — configura un objeto y retorna el mismo objeto.
     */
    fun createCompletedTask(original: Task): Task = original.copy().apply {
        // 'apply' retorna el mismo objeto; en data class usamos copy
        // Aquí demostramos el patrón: en la práctica, copy() ya genera el nuevo objeto
    }.let { it.copy(completed = true) }

    /**
     * 'also' — ejecuta una acción lateral y retorna el mismo objeto.
     * Ideal para logging / debugging.
     */
    fun getHighPriorityTasks(): List<Task> {
        return tasks
            .filter { it.category.isHighPriority() }
            .also { filtered ->
                println("LOG → Tareas de alta prioridad encontradas: ${filtered.size}")
            }
    }

    // ── Composición funcional con colecciones ─────────────────────────
    fun getPipelineDemo(): String {
        return tasks
            .filter { it.durationMs > 1000L }          // solo tareas "lentas"
            .sortedByDescending { it.durationMs }       // más lentas primero
            .groupBy { it.category }                    // agrupar por categoría
            .map { (category, groupTasks) ->            // transformar cada grupo
                "${category.displayName}: ${groupTasks.joinToString { it.name }}"
            }
            .joinToString(separator = "\n")
    }
}
```

**Resultado esperado:**

El archivo compila sin errores. La clase `TaskRepository` contiene métodos que demuestran cada scope function y funciones de orden superior.

**Verificación:**

- **Build → Rebuild Project** finaliza con éxito.
- Posiciona el cursor sobre cada scope function (`let`, `run`, `with`, `apply`, `also`) y verifica con `Ctrl+Q` (Quick Documentation) que el IDE reconoce el tipo de retorno correcto de cada una.

---

### Paso 4 — Corrutinas: launch, async/await y Dispatchers

**Objetivo:** Crear un ViewModel que ejecute operaciones concurrentes con corrutinas, demostrando `launch`, `async`/`await`, distintos `Dispatchers` y `withContext`.

**Instrucciones:**

1. Crea el paquete `viewmodel` dentro de `com.cursoadv.android.kotlin`.

2. Dentro de `viewmodel`, crea **`KotlinLabViewModel.kt`**:

```kotlin
package com.cursoadv.android.kotlin.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.cursoadv.android.kotlin.data.TaskRepository
import com.cursoadv.android.kotlin.model.OperationResult
import com.cursoadv.android.kotlin.model.Task
import com.cursoadv.android.kotlin.model.TaskCategory
import kotlinx.coroutines.CoroutineExceptionHandler
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.Job
import kotlinx.coroutines.async
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharedFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.filter
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.flowOn
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

class KotlinLabViewModel : ViewModel() {

    private val repository = TaskRepository()

    // ── Bloque 3: Estado principal con StateFlow ──────────────────────
    private val _uiState = MutableStateFlow<OperationResult<String>>(
        OperationResult.Loading
    )
    val uiState: StateFlow<OperationResult<String>> = _uiState.asStateFlow()

    // ── Bloque 4: Eventos únicos con SharedFlow ──────────────────────
    private val _events = MutableSharedFlow<UiEvent>(
        extraBufferCapacity = 5
    )
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()

    // ── Bloque 5: Job para cancelación ───────────────────────────────
    private var longRunningJob: Job? = null

    // ── Bloque 3: CoroutineExceptionHandler ──────────────────────────
    private val exceptionHandler = CoroutineExceptionHandler { _, throwable ->
        viewModelScope.launch {
            _uiState.value = OperationResult.Error(
                message = throwable.message ?: "Error desconocido",
                exception = throwable
            )
            _events.emit(UiEvent.ShowSnackbar("Excepción capturada: ${throwable.message}"))
        }
    }

    // ══════════════════════════════════════════════════════════════════
    // BLOQUE 3 — Corrutinas con launch y async/await
    // ══════════════════════════════════════════════════════════════════

    /**
     * Demuestra 'launch' para fire-and-forget.
     * Simula una operación de red en Dispatchers.IO.
     */
    fun ejecutarOperacionSimple() {
        viewModelScope.launch(exceptionHandler) {
            _uiState.value = OperationResult.Loading

            // Cambiar a hilo IO para simular trabajo pesado
            val resultado = withContext(Dispatchers.IO) {
                delay(1500L) // Simula latencia de red
                repository.getTaskSummary()
            }

            // De vuelta en Main automáticamente (viewModelScope usa Main)
            _uiState.value = OperationResult.Success(resultado)
            _events.emit(UiEvent.ShowSnackbar("Operación simple completada"))
        }
    }

    /**
     * Demuestra 'async/await' para operaciones concurrentes.
     * Ejecuta dos tareas en paralelo y combina resultados.
     */
    fun ejecutarOperacionesConcurrentes() {
        viewModelScope.launch(exceptionHandler) {
            _uiState.value = OperationResult.Loading

            // Dos operaciones en paralelo con async
            val deferredNetwork = async(Dispatchers.IO) {
                delay(2000L) // Simula llamada de red
                repository.filterTasks { it.category == TaskCategory.NETWORK }
            }

            val deferredCompute = async(Dispatchers.Default) {
                delay(1500L) // Simula cómputo pesado
                repository.filterTasks { it.category == TaskCategory.COMPUTATION }
            }

            // await() suspende hasta que ambas terminen
            val networkTasks = deferredNetwork.await()
            val computeTasks = deferredCompute.await()

            val resumen = buildString {
                appendLine("═══ Resultados Concurrentes ═══")
                appendLine("Red (${networkTasks.size} tareas):")
                networkTasks.forEach { appendLine("  • ${it.name} — ${it.durationMs}ms") }
                appendLine("Cómputo (${computeTasks.size} tareas):")
                computeTasks.forEach { appendLine("  • ${it.name} — ${it.durationMs}ms") }
            }

            _uiState.value = OperationResult.Success(resumen)
            _events.emit(UiEvent.ShowSnackbar("Operaciones concurrentes finalizadas"))
        }
    }

    // ══════════════════════════════════════════════════════════════════
    // BLOQUE 4 — Scope functions demo (ejecuta desde UI)
    // ══════════════════════════════════════════════════════════════════

    fun ejecutarScopeFunctionsDemo() {
        viewModelScope.launch {
            _uiState.value = OperationResult.Loading
            delay(500L)

            val resultado = buildString {
                appendLine("═══ Scope Functions Demo ═══")
                appendLine()
                appendLine("▸ let (findTaskById):")
                appendLine("  ${repository.findTaskById(3)}")
                appendLine("  ${repository.findTaskById(99)}")
                appendLine()
                appendLine("▸ run (getTaskSummary):")
                appendLine("  ${repository.getTaskSummary()}")
                appendLine()
                appendLine("▸ with (describeCategory):")
                appendLine("  ${repository.describeCategory(TaskCategory.NETWORK)}")
                appendLine()
                appendLine("▸ also (getHighPriorityTasks):")
                val highPriority = repository.getHighPriorityTasks()
                highPriority.forEach { appendLine("  • ${it.name}") }
                appendLine()
                appendLine("▸ Pipeline funcional:")
                appendLine(repository.getPipelineDemo())
            }

            _uiState.value = OperationResult.Success(resultado)
        }
    }

    // ══════════════════════════════════════════════════════════════════
    // BLOQUE 5 — Cancelación estructurada
    // ══════════════════════════════════════════════════════════════════

    fun iniciarTareaLarga() {
        longRunningJob?.cancel() // Cancela la anterior si existe

        longRunningJob = viewModelScope.launch(exceptionHandler) {
            _uiState.value = OperationResult.Loading

            val pasos = 10
            val resultado = buildString {
                appendLine("═══ Tarea Larga (cancelable) ═══")
                for (i in 1..pasos) {
                    // delay() es un punto de cancelación cooperativa
                    delay(800L)
                    appendLine("Paso $i/$pasos completado")
                    _uiState.value = OperationResult.Success(toString())
                }
                appendLine("✅ Tarea completada sin cancelación")
            }
            _uiState.value = OperationResult.Success(resultado)
            _events.emit(UiEvent.ShowSnackbar("Tarea larga finalizada"))
        }

        longRunningJob?.invokeOnCompletion { throwable ->
            if (throwable is kotlinx.coroutines.CancellationException) {
                viewModelScope.launch {
                    _uiState.value = OperationResult.Error("Tarea cancelada por el usuario")
                    _events.emit(UiEvent.ShowSnackbar("Tarea cancelada"))
                }
            }
        }
    }

    fun cancelarTareaLarga() {
        longRunningJob?.cancel()
        longRunningJob = null
    }

    // ══════════════════════════════════════════════════════════════════
    // BLOQUE 5 — Manejo de excepciones con try/catch
    // ══════════════════════════════════════════════════════════════════

    fun ejecutarConErrorSimulado() {
        viewModelScope.launch {
            _uiState.value = OperationResult.Loading
            delay(1000L)

            try {
                withContext(Dispatchers.IO) {
                    // Simula un error de red
                    throw java.io.IOException("Timeout al conectar con el servidor")
                }
            } catch (e: java.io.IOException) {
                _uiState.value = OperationResult.Error(
                    message = "Error de red: ${e.message}",
                    exception = e
                )
                _events.emit(UiEvent.ShowSnackbar("Error capturado con try/catch"))
            }
        }
    }

    // ══════════════════════════════════════════════════════════════════
    // BLOQUE 6 — Flow pipeline reactivo
    // ══════════════════════════════════════════════════════════════════

    fun ejecutarFlowPipeline() {
        viewModelScope.launch {
            _uiState.value = OperationResult.Loading

            // Construir un cold Flow que emite tareas una a una
            taskFlow()
                .filter { it.durationMs > 500L }
                .map { task ->
                    "${task.category.displayName}: ${task.name} (${task.durationMs}ms)"
                }
                .catch { e ->
                    _uiState.value = OperationResult.Error("Error en Flow: ${e.message}")
                }
                .flowOn(Dispatchers.Default) // operadores anteriores en Default
                .collect { line ->
                    // collect se ejecuta en Main (contexto de viewModelScope)
                    val current = when (val state = _uiState.value) {
                        is OperationResult.Success -> state.data
                        else -> "═══ Flow Pipeline ═══\n"
                    }
                    _uiState.value = OperationResult.Success("$current\n$line")
                }

            _events.emit(UiEvent.ShowSnackbar("Flow pipeline completado"))
        }
    }

    /**
     * Cold Flow que emite tareas con un retardo simulado.
     */
    private fun taskFlow() = flow {
        val tasks = repository.filterTasks { true } // todas
        for (task in tasks) {
            delay(400L) // Simula procesamiento
            emit(task)
        }
    }

    // ── Eventos de UI ────────────────────────────────────────────────
    sealed class UiEvent {
        data class ShowSnackbar(val message: String) : UiEvent()
    }
}
```

**Resultado esperado:**

El ViewModel compila sin errores y contiene métodos para los seis bloques de la práctica: sealed classes (ya integradas en el estado), scope functions, launch, async/await, cancelación, excepciones y Flow.

**Verificación:**

- **Build → Rebuild Project** finaliza con `BUILD SUCCESSFUL`.
- El IDE no muestra warnings sobre APIs deprecadas.
- Verifica que `viewModelScope` se resuelve correctamente (proviene de `lifecycle-viewmodel-compose`).

---

### Paso 5 — Construir la interfaz de usuario con Compose

**Objetivo:** Crear una pantalla con botones para ejecutar cada bloque de la práctica, visualizando resultados en tiempo real y demostrando la integración de `StateFlow` y `SharedFlow` con Compose.

**Instrucciones:**

1. Crea el paquete `ui` dentro de `com.cursoadv.android.kotlin`.

2. Dentro de `ui`, crea **`KotlinLabScreen.kt`**:

```kotlin
package com.cursoadv.android.kotlin.ui

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.ExperimentalLayoutApi
import androidx.compose.foundation.layout.FlowRow
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedButton
import androidx.compose.material3.Scaffold
import androidx.compose.material3.SnackbarHost
import androidx.compose.material3.SnackbarHostState
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import androidx.lifecycle.viewmodel.compose.viewModel
import com.cursoadv.android.kotlin.model.OperationResult
import com.cursoadv.android.kotlin.util.toDisplayString
import com.cursoadv.android.kotlin.viewmodel.KotlinLabViewModel

@OptIn(ExperimentalLayoutApi::class)
@Composable
fun KotlinLabScreen(
    viewModel: KotlinLabViewModel = viewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }

    // Observar eventos únicos (SharedFlow)
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is KotlinLabViewModel.UiEvent.ShowSnackbar -> {
                    snackbarHostState.showSnackbar(event.message)
                }
            }
        }
    }

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
                .padding(16.dp)
                .verticalScroll(rememberScrollState()),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            // ── Título ───────────────────────────────────────────────
            Text(
                text = "Práctica 1 — Kotlin Avanzado",
                style = MaterialTheme.typography.headlineMedium
            )

            // ── Botones de acción ────────────────────────────────────
            FlowRow(
                horizontalArrangement = Arrangement.spacedBy(8.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                ActionButton("Scope Functions") {
                    viewModel.ejecutarScopeFunctionsDemo()
                }
                ActionButton("Launch Simple") {
                    viewModel.ejecutarOperacionSimple()
                }
                ActionButton("Async/Await") {
                    viewModel.ejecutarOperacionesConcurrentes()
                }
                ActionButton("Iniciar Tarea") {
                    viewModel.iniciarTareaLarga()
                }
                OutlinedButton(onClick = { viewModel.cancelarTareaLarga() }) {
                    Text("Cancelar Tarea")
                }
                ActionButton("Error Simulado") {
                    viewModel.ejecutarConErrorSimulado()
                }
                ActionButton("Flow Pipeline") {
                    viewModel.ejecutarFlowPipeline()
                }
            }

            // ── Panel de resultados ──────────────────────────────────
            ResultPanel(uiState)
        }
    }
}

@Composable
private fun ActionButton(
    label: String,
    onClick: () -> Unit
) {
    Button(
        onClick = onClick,
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary
        )
    ) {
        Text(text = label, fontSize = 13.sp)
    }
}

@Composable
private fun ResultPanel(state: OperationResult<String>) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = when (state) {
                is OperationResult.Loading -> MaterialTheme.colorScheme.surfaceVariant
                is OperationResult.Success -> MaterialTheme.colorScheme.secondaryContainer
                is OperationResult.Error -> MaterialTheme.colorScheme.errorContainer
            }
        )
    ) {
        Text(
            text = when (state) {
                is OperationResult.Loading -> "⏳ Cargando..."
                is OperationResult.Success -> state.data
                is OperationResult.Error -> "❌ ${state.message}"
            },
            modifier = Modifier.padding(16.dp),
            fontFamily = FontFamily.Monospace,
            fontSize = 13.sp,
            lineHeight = 18.sp
        )
    }
}
```

3. Abre **`MainActivity.kt`** y reemplaza su contenido con:

```kotlin
package com.cursoadv.android.kotlin

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.cursoadv.android.kotlin.ui.KotlinLabScreen
import com.cursoadv.android.kotlin.ui.theme.Practica1KotlinAvanzadoTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Practica1KotlinAvanzadoTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    KotlinLabScreen()
                }
            }
        }
    }
}
```

> **Nota sobre el tema:** Android Studio genera automáticamente el paquete `ui.theme` con `Theme.kt`, `Color.kt` y `Type.kt`. El nombre del tema composable generado suele ser `Practica1KotlinAvanzadoTheme` (basado en el nombre del proyecto). Si el nombre difiere, ajústalo para que coincida con el que generó tu plantilla. Puedes verificarlo abriendo `ui/theme/Theme.kt`.

**Resultado esperado:**

El proyecto compila y la aplicación muestra una pantalla con:
- Un título "Práctica 1 — Kotlin Avanzado".
- Siete botones de acción.
- Un panel de resultados que inicialmente muestra "⏳ Cargando...".

**Verificación:**

- **Build → Rebuild Project** → `BUILD SUCCESSFUL`.
- No hay errores de importación sin resolver (todas las dependencias están en `libs.versions.toml`).

---

### Paso 6 — Ejecutar y probar cada bloque funcional

**Objetivo:** Ejecutar la aplicación en un emulador o dispositivo y verificar el correcto funcionamiento de cada uno de los seis bloques de la práctica.

**Instrucciones:**

1. Selecciona un AVD (recomendado: API 35 o 36) o conecta un dispositivo físico.

2. Haz clic en **Run ▶** (o `Shift+F10`).

3. Una vez que la aplicación cargue, ejecuta cada prueba en orden:

#### Prueba 6.1 — Scope Functions

4. Pulsa el botón **"Scope Functions"**.
5. Observa que el panel cambia a fondo claro (Success) y muestra la salida de cada scope function (`let`, `run`, `with`, `also`) y el pipeline funcional.

**Resultado esperado para "Scope Functions":**

```
═══ Scope Functions Demo ═══

▸ let (findTaskById):
  Encontrada: Cálculo de estadísticas [Cómputo]
  Tarea con id=99 no encontrada

▸ run (getTaskSummary):
  Total: 8 tareas | Completadas: 0 | Pendientes: 8

▸ with (describeCategory):
  Categoría: Red | Prioridad: 1 | Alta prioridad: true

▸ also (getHighPriorityTasks):
  • Llamada API usuarios
  • Consulta Room local
  • Sincronizar inventario
  • Migración de esquema

▸ Pipeline funcional:
Cómputo: Compresión de imágenes, Cálculo de estadísticas
Red: Sincronizar inventario, Llamada API usuarios
Base de datos: Migración de esquema
```

#### Prueba 6.2 — Launch Simple

6. Pulsa **"Launch Simple"**.
7. Observa que el panel muestra "⏳ Cargando..." durante ~1.5 segundos y luego cambia al resumen de tareas.
8. Verifica que aparece un **Snackbar** con "Operación simple completada".

#### Prueba 6.3 — Async/Await

9. Pulsa **"Async/Await"**.
10. Observa que el panel muestra "⏳ Cargando..." durante ~2 segundos (las dos operaciones se ejecutan en paralelo; el tiempo total es el de la más lenta, no la suma).
11. El panel muestra las tareas de red y cómputo agrupadas.

**Resultado esperado para "Async/Await":**

```
═══ Resultados Concurrentes ═══
Red (2 tareas):
  • Llamada API usuarios — 1500ms
  • Sincronizar inventario — 2500ms
Cómputo (2 tareas):
  • Cálculo de estadísticas — 2000ms
  • Compresión de imágenes — 3000ms
```

#### Prueba 6.4 — Iniciar/Cancelar Tarea

12. Pulsa **"Iniciar Tarea"**.
13. Observa que el panel se actualiza progresivamente: "Paso 1/10", "Paso 2/10", etc.
14. **Antes de que llegue al paso 10**, pulsa **"Cancelar Tarea"**.
15. El panel debe mostrar "❌ Tarea cancelada por el usuario" con fondo rojo/error.
16. Aparece un Snackbar con "Tarea cancelada".

17. Pulsa **"Iniciar Tarea"** de nuevo y déjala completar los 10 pasos sin cancelar.
18. Al finalizar, el panel muestra "✅ Tarea completada sin cancelación".

#### Prueba 6.5 — Error Simulado

19. Pulsa **"Error Simulado"**.
20. Tras ~1 segundo, el panel muestra fondo de error con "❌ Error de red: Timeout al conectar con el servidor".
21. Aparece un Snackbar con "Error capturado con try/catch".

#### Prueba 6.6 — Flow Pipeline

22. Pulsa **"Flow Pipeline"**.
23. Observa que el panel se actualiza incrementalmente (cada ~400ms aparece una nueva línea) conforme el Flow emite cada tarea filtrada (solo las que tienen `durationMs > 500`).

**Resultado esperado para "Flow Pipeline" (al completar):**

```
═══ Flow Pipeline ═══
Red: Llamada API usuarios (1500ms)
Base de datos: Consulta Room local (800ms)
Cómputo: Cálculo de estadísticas (2000ms)
Red: Sincronizar inventario (2500ms)
Base de datos: Migración de esquema (1200ms)
Cómputo: Compresión de imágenes (3000ms)
```

> **Nota:** La tarea "Renderizar gráfico" (300ms) y "Animación de transición" (450ms) no aparecen porque el filtro excluye tareas con `durationMs <= 500`.

**Verificación general:**

- Todos los 6 bloques producen la salida esperada.
- Los Snackbars aparecen correctamente (esto verifica que `SharedFlow` funciona como canal de eventos).
- El color del panel cambia según el estado (`Loading` → gris, `Success` → verde claro, `Error` → rojo claro).
- La cancelación de la tarea larga funciona inmediatamente al pulsar "Cancelar Tarea".

---

### Paso 7 — Verificar la extensión mapResult y sealed class exhaustiva

**Objetivo:** Confirmar que la función de extensión `mapResult` y el patrón exhaustivo de `when` sobre sealed classes funcionan correctamente.

**Instrucciones:**

1. Dentro del paquete `viewmodel`, abre `KotlinLabViewModel.kt`.

2. Añade el siguiente método al final de la clase (antes de la llave de cierre):

```kotlin
    /**
     * Demuestra mapResult (extensión sobre OperationResult).
     * Transforma el contenido de Success sin alterar Loading/Error.
     */
    fun demostrarMapResult() {
        viewModelScope.launch {
            _uiState.value = OperationResult.Loading
            delay(800L)

            // Crear un Success con una lista de tareas
            val original: OperationResult<List<Task>> = OperationResult.Success(
                repository.filterTasks { it.category == TaskCategory.NETWORK }
            )

            // Usar mapResult para transformar List<Task> a String
            val transformed: OperationResult<String> = original.mapResult { tasks ->
                buildString {
                    appendLine("═══ mapResult Demo ═══")
                    appendLine("Original: List<Task> con ${tasks.size} elementos")
                    appendLine("Transformado a String:")
                    tasks.forEach { appendLine("  → ${it.name}") }
                }
            }

            _uiState.value = transformed
        }
    }
```

3. En `KotlinLabScreen.kt`, dentro del bloque `FlowRow`, añade un nuevo botón:

```kotlin
                ActionButton("mapResult") {
                    viewModel.demostrarMapResult()
                }
```

4. Ejecuta la aplicación de nuevo y pulsa **"mapResult"**.

**Resultado esperado:**

```
═══ mapResult Demo ═══
Original: List<Task> con 2 elementos
Transformado a String:
  → Llamada API usuarios
  → Sincronizar inventario
```

**Verificación:**

- La transformación `List<Task> → String` se realizó dentro de `mapResult` sin perder el tipo `OperationResult`.
- Si cambias `OperationResult.Success(...)` por `OperationResult.Loading` o `OperationResult.Error(...)`, `mapResult` los pasa sin modificar (puedes probarlo temporalmente).

---

## 7. Validación y Pruebas

### Prueba de compilación completa

```bash
cd ~/AndroidStudioProjects/AvanzadoKotlin/Practica1_KotlinAvanzado
./gradlew assembleDebug
```

**Resultado esperado:**

```
BUILD SUCCESSFUL in XXs
XX actionable tasks: XX executed
```

### Lista de verificación funcional

| # | Criterio | ✅ |
|---|---|---|
| 1 | `libs.versions.toml` contiene todas las versiones exactas del baseline | ☐ |
| 2 | `compileSdk = 37`, `minSdk = 30`, `targetSdk = 37` | ☐ |
| 3 | No existe referencia a `kotlinCompilerExtensionVersion` | ☐ |
| 4 | `OperationResult<T>` tiene exactamente 3 subtipos: `Loading`, `Success`, `Error` | ☐ |
| 5 | `when` sobre `OperationResult` es exhaustivo (sin rama `else`) | ☐ |
| 6 | Botón "Scope Functions" muestra salida de `let`, `run`, `with`, `also` y pipeline | ☐ |
| 7 | Botón "Launch Simple" muestra carga y luego resultado tras ~1.5s | ☐ |
| 8 | Botón "Async/Await" completa en ~2s (no 3.5s) demostrando paralelismo | ☐ |
| 9 | "Iniciar Tarea" + "Cancelar Tarea" produce mensaje de cancelación | ☐ |
| 10 | "Error Simulado" muestra panel de error y Snackbar | ☐ |
| 11 | "Flow Pipeline" actualiza el panel incrementalmente | ☐ |
| 12 | "mapResult" transforma `Success` sin alterar otros estados | ☐ |
| 13 | Snackbars aparecen correctamente (SharedFlow funciona) | ☐ |
| 14 | No hay warnings de APIs deprecadas en la pestaña Build | ☐ |

---

## 8. Solución de Problemas

### Problema 1: Error `Unresolved reference: collectAsStateWithLifecycle`

**Síntomas:**

El IDE marca en rojo la llamada `viewModel.uiState.collectAsStateWithLifecycle()` en `KotlinLabScreen.kt` con el mensaje:

```
Unresolved reference: collectAsStateWithLifecycle
```

**Causa:**

La función `collectAsStateWithLifecycle` pertenece al artefacto `lifecycle-runtime-compose`, que no está incluido explícitamente en las dependencias. En algunas versiones del Compose BOM este artefacto se incluye transitivamente, pero en otras no.

**Solución:**

1. Abre `gradle/libs.versions.toml` y añade en la sección `[libraries]`:

```toml
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose" }
```

> **Nota:** No se especifica versión porque el Compose BOM gestiona la versión. Si el BOM no la cubre, añade `version.ref = "lifecycleRuntimeKtx"`.

2. En `app/build.gradle.kts`, añade:

```kotlin
implementation(libs.androidx.lifecycle.runtime.compose)
```

3. Haz **Sync Now** y reconstruye el proyecto.

---

### Problema 2: La aplicación lanza `NetworkOnMainThreadException` o se congela al pulsar un botón

**Síntomas:**

Al pulsar cualquier botón que simula trabajo pesado, la aplicación se congela o en Logcat aparece:

```
android.os.NetworkOnMainThreadException
```

O bien la UI no responde durante varios segundos.

**Causa:**

Se eliminó accidentalmente la llamada a `withContext(Dispatchers.IO)` o `withContext(Dispatchers.Default)` dentro de alguna función del ViewModel, causando que `delay()` o el trabajo simulado se ejecute bloqueando el hilo principal. Aunque `delay()` es suspendida y no bloquea, si se reemplazó por `Thread.sleep()` durante una prueba, esto sí bloquea el hilo principal.

**Solución:**

1. Verifica que **todas** las operaciones simuladas usan `delay()` (función suspendida) y **no** `Thread.sleep()`.
2. Confirma que las operaciones de cómputo o I/O están envueltas en `withContext(Dispatchers.IO)` o `withContext(Dispatchers.Default)`.
3. Verifica que el `viewModelScope.launch` no tiene `Dispatchers.IO` como contexto principal (debe usar el predeterminado `Dispatchers.Main` para poder actualizar `_uiState`):

```kotlin
// ✅ Correcto
viewModelScope.launch {
    val result = withContext(Dispatchers.IO) { /* trabajo pesado */ }
    _uiState.value = OperationResult.Success(result) // Main
}

// ❌ Incorrecto — no puede actualizar UI desde IO
viewModelScope.launch(Dispatchers.IO) {
    _uiState.value = ... // Crash o comportamiento inesperado
}
```

---

## 9. Limpieza

Esta práctica no requiere limpieza de recursos externos (no se crearon bases de datos, servicios ni cuentas). El proyecto se conserva como referencia para las prácticas 2 a 5.

Si deseas liberar espacio del caché de Gradle:

```bash
cd ~/AndroidStudioProjects/AvanzadoKotlin/Practica1_KotlinAvanzado
./gradlew clean
```

Para detener daemons de Gradle activos:

```bash
./gradlew --stop
```

---

## 10. Resumen

### Conceptos aplicados en esta práctica

| Bloque | Concepto clave | Archivos principales |
|---|---|---|
| 1 | `sealed class`, `enum class`, genéricos, `data class`, `data object` | `OperationResult.kt`, `TaskCategory.kt`, `Task.kt` |
| 2 | Lambdas, HOF, `let`, `run`, `with`, `apply`, `also`, pipeline funcional | `TaskRepository.kt` |
| 3 | `launch`, `async`/`await`, `Dispatchers`, `withContext` | `KotlinLabViewModel.kt` |
| 4 | `StateFlow`, `SharedFlow`, `collectAsStateWithLifecycle` | `KotlinLabViewModel.kt`, `KotlinLabScreen.kt` |
| 5 | Cancelación cooperativa, `CoroutineExceptionHandler`, `try`/`catch` | `KotlinLabViewModel.kt` |
| 6 | `flow { }`, `filter`, `map`, `catch`, `flowOn`, `collect` | `KotlinLabViewModel.kt` |

### Puntos clave

- **`sealed class`** permite modelar estados finitos con exhaustividad verificada por el compilador — no se necesita rama `else` en `when`.
- **`StateFlow`** mantiene siempre un valor actual y es ideal para estado de UI; **`SharedFlow`** no retiene valores y es ideal para eventos únicos.
- **`async`/`await`** permite ejecutar operaciones en paralelo real, reduciendo el tiempo total al de la operación más lenta.
- **La cancelación cooperativa** requiere que el código suspendido use funciones como `delay()`, `yield()` o verifique `isActive` para responder a la cancelación.
- **`flowOn`** cambia el Dispatcher de los operadores *anteriores* en la cadena, no de los posteriores ni del `collect`.
- **`CoroutineExceptionHandler`** captura excepciones no manejadas en corrutinas lanzadas con `launch`; no funciona con `async` (que encapsula la excepción en el `Deferred`).

### Recursos adicionales

- [Guía de corrutinas de Kotlin — JetBrains](https://kotlinlang.org/docs/coroutines-guide.html)
- [StateFlow y SharedFlow — Android Developers](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [Scope functions — Documentación oficial de Kotlin](https://kotlinlang.org/docs/scope-functions.html)
- [Structured Concurrency — Roman Elizarov](https://elizarov.medium.com/structured-concurrency-722d765aa952)
- [Flow en Android — Android Developers](https://developer.android.com/kotlin/flow)

---
