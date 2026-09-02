# Cheat Sheet · Lifecycle of a Program

> Repaso rápido de [`Pura/Java/01-Basics/02-lifecycle-of-a-program.md`](../../../Pura/Java/01-Basics/02-lifecycle-of-a-program.md) · Java 17+

## En 30 segundos

- Este es el tema que separa a quien "escribe Java" de quien "entiende Java".
- Las fases **carga → enlazado → inicialización no ocurren una vez al arrancar**, sino cada vez que se usa una clase nueva. Java carga clases perezosamente.
- `javac` **apenas optimiza**: traduce de forma literal. Toda la optimización pesada la hace el JIT en runtime, con datos reales de ejecución.
- La JVM **no muere cuando `main` termina**: muere cuando acaban todos los hilos **no-daemon**.
- El primer request es lento porque las clases no están cargadas y el código va interpretado. Eso es el *warm-up*.

## El mapa completo

| Fase | Cuándo | Quién | Qué produce |
|---|---|---|---|
| **Compilación** | build time | `javac` | bytecode (`.class`) |
| **1. Carga** (loading) | runtime, bajo demanda | ClassLoader | un objeto `java.lang.Class` |
| **2. Enlazado** (linking) | runtime, tras cargar | JVM | clase verificada e integrada |
| **3. Inicialización** | runtime, al primer uso real | JVM | `static` con sus valores reales |
| **4. Ejecución** | runtime | intérprete + JIT | el resultado del programa |
| **5. Terminación** | al final | JVM | código de salida del proceso |

## Compilación: qué es el bytecode

No es código máquina ni tu fuente: es un formato intermedio para una máquina que no existe físicamente. `javac` produce el mismo `.class` en cualquier sistema operativo.

```bash
javap -c Main.class     # desensambla
javap -v Main.class     # todo: constant pool, versión, flags, atributos
```

La JVM es una **máquina de pila**: no tiene registros, empuja valores y las instrucciones operan sobre ellos (`iadd` no lleva argumentos: coge los dos de arriba, suma y deja el resultado).

Todo `.class` empieza por el número mágico **`0xCAFEBABE`**. El campo de versión explica el `UnsupportedClassVersionError`: bytecode más nuevo que la JVM. Versión mayor **61 = Java 17**, **65 = Java 21**, **69 = Java 25**.

## Fase 1 — Carga

Encontrar los bytes del `.class`, leerlos y construir en memoria un objeto `java.lang.Class`. Es el que devuelve `miObjeto.getClass()` y la base de toda la reflexión.

| Cargador | Qué carga |
|---|---|
| **Bootstrap** | el núcleo (`java.lang`, `java.util`…). Escrito en nativo; `String.class.getClassLoader()` devuelve `null` |
| **Platform** | módulos de plataforma no del núcleo (`java.sql`, `java.xml`…). Desde Java 9 |
| **Application** (System) | **tu código** y las librerías del classpath |

> **Corrección frecuente en la documentación que circula:** los cargadores *Bootstrap / Extension / Application* y el `rt.jar` son **anteriores a Java 9**. En Java 9 desapareció `rt.jar` y el *Extension class loader* pasó a ser **Platform class loader**. Si una fuente te habla de `rt.jar`, describe Java 8.

**Delegación al padre:** un cargador no busca él primero, se lo pide al padre; solo si el padre no la tiene, la busca él. Existe **por seguridad**: nadie puede colar un `java.lang.String` malicioso, porque el bootstrap siempre gana la carrera.

## Fase 2 — Enlazado (tres pasos, se preguntan en entrevistas)

| Paso | Qué hace |
|---|---|
| **Verificación** | comprueba que el bytecode es legal y seguro **antes** de ejecutarlo. Existe porque el bytecode no tiene por qué venir de `javac` (Kotlin, Scala, un generador, un atacante). Si falla: `VerifyError` |
| **Preparación** | reserva memoria para los `static` y les da **el valor por defecto de su tipo**, no el que escribiste |
| **Resolución** | convierte referencias simbólicas (texto) en referencias directas. Es **perezosa** |

Si escribiste `static int contador = 42;`, al terminar la preparación vale **0**. El 42 llega en la fase siguiente.

## Fase 3 — Inicialización (`<clinit>`)

El compilador mete **todas** las asignaciones a campos estáticos y todos los bloques `static { }` — en orden textual — en un método `<clinit>` que solo llama la JVM.

**Cuatro reglas:**
1. **Orden textual**: de arriba abajo. Mover una línea cambia el resultado.
2. **La superclase primero**, y por completo.
3. **Una sola vez, y thread-safe**, garantizado por la JVM aunque compitan 20 hilos.
4. **Perezosa**: solo al primer uso real.

**Los cinco disparadores:** `new`; invocar un método `static`; leer/escribir un campo `static` **no constante**; reflexión (`Class.forName`); ser la clase del `main`.

```java
class Pesada {
    static { System.out.println("¡inicializada!"); }
    static final int CONSTANTE = 42;
    static int variable = 42;
}
System.out.println(Pesada.CONSTANTE);   // NO inicializa: el compilador incrusta el 42
System.out.println(Pesada.variable);    // AQUÍ sí se inicializa
```

Consecuencia desagradable: si cambiás el valor de una constante en una librería y no recompilás a quien la usa, este sigue con el valor viejo.

### El idioma del *holder*: pereza y thread-safety gratis

```java
public class Servicio {
    private Servicio() { }
    private static class Holder {
        static final Servicio INSTANCIA = new Servicio();   // se crea al primer acceso
    }
    public static Servicio getInstancia() { return Holder.INSTANCIA; }
}
```

Cero `synchronized` escrito por vos, cero coste. Es el mejor ejemplo de por qué entender el ciclo de vida da código mejor.

## Fase 4 — Ejecución

### Orden de construcción de un objeto

1. Memoria reservada; todos los campos a su valor por defecto.
2. Constructor de la **superclase** (`super()`, explícito o implícito).
3. Inicializadores de instancia y asignaciones de campos, en orden textual.
4. Cuerpo del constructor.
5. Se devuelve la referencia.

### Áreas de memoria

| Área | Qué guarda | ¿Compartida? | Si se llena |
|---|---|---|---|
| **Heap** | todos los objetos y arrays | Sí | `OutOfMemoryError: Java heap space` |
| **Stack** | marcos de método: locales, parámetros, retorno | **No**, una por hilo | `StackOverflowError` |
| **Metaspace** | metadatos de clases, código de métodos | Sí | `OutOfMemoryError: Metaspace` |
| **PC Register** | instrucción actual | No, uno por hilo | — |
| **Native Method Stack** | llamadas JNI | No, una por hilo | — |

La recursión infinita da **`StackOverflowError`, no `OutOfMemoryError`**: son zonas distintas. El Metaspace sustituyó al PermGen en Java 8, vive en memoria nativa y es donde aparecen las fugas por recarga de clases.

### Interpretación → perfilado → JIT

La JVM **no elige** entre interpretar o compilar: hace las dos cosas, en este orden.

```
intérprete (nivel 0) → C1 (niveles 1-3) → C2 (nivel 4)
   arranque ya,          compila rápido,     compila lento,
   ejecución lenta       optimización        optimización agresiva
                         moderada            con el perfilado
```

| Nivel | Quién | Perfilado |
|---|---|---|
| 0 | intérprete | contadores básicos |
| 1 | C1 | ninguno (métodos triviales: un getter) |
| 2 | C1 | limitado |
| 3 | C1 | completo — el camino habitual antes de C2 |
| 4 | C2 | ninguno (ya usa el recogido) |

Recorrido típico de un método caliente: `0 → 3 → 4`.

**Lo que el JIT puede hacer y `javac` no**, porque observa la ejecución real: *inlining*, devirtualización, eliminación de código muerto, *escape analysis*, desenrollado de bucles, vectorización.

**Deoptimización:** C2 *especula*; si se equivoca, descarta el código compilado y vuelve al intérprete para recompilar con mejor información. Un programa puede empeorar y luego volver a mejorar.

**OSR (On-Stack Replacement):** si un método se llama **una sola vez** pero tiene un bucle de mil millones de vueltas, la JVM cuenta también los saltos hacia atrás y **sustituye el marco de pila en caliente**, saltando de interpretado a compilado a mitad de ejecución.

## Fase 5 — Terminación

```java
Thread t = new Thread(() -> { /* duerme 5 s */ });
t.start();
System.out.println("main terminó");   // el proceso sigue vivo 5 segundos más
```

| Forma | Qué ocurre |
|---|---|
| **Normal** | acaban todos los hilos no-daemon. Se ejecutan los shutdown hooks. Salida 0 |
| **`System.exit(n)`** | apagado inmediato. Se ejecutan los hooks. Salida `n` |

Un hilo **daemon** (`t.setDaemon(true)`) no cuenta: la JVM lo mata al salir. Un pool de hilos no cerrado es la causa clásica de procesos "colgados" en producción.

**Shutdown hooks** se ejecutan en terminación normal, con `System.exit()` y con `SIGTERM` (`docker stop`). **No** con `SIGKILL` (`kill -9`) ni si la JVM se cae. Nunca dependas de ellos para datos críticos.

**`finalize()`: no lo uses.** Deprecado para eliminación desde Java 18 (JEP 421). Su ejecución **nunca estuvo garantizada**.

| En vez de | Usá |
|---|---|
| `finalize()` para cerrar recursos | `AutoCloseable` + `try-with-resources` |
| `finalize()` como red de seguridad | `java.lang.ref.Cleaner` |
| control fino sobre inalcanzabilidad | `PhantomReference` |

## `ClassNotFoundException` vs `NoClassDefFoundError`

| | `ClassNotFoundException` | `NoClassDefFoundError` |
|---|---|---|
| Tipo | `Exception` (checked) | `Error` |
| Cuándo | carga **explícita**: `Class.forName`, `loadClass` | estaba al compilar, no está en runtime |
| Causa típica | nombre mal escrito, driver JDBC ausente | falta en el classpath, **o la clase falló al inicializarse** |

El segundo caso es el traicionero: si `<clinit>` lanza, obtenés `ExceptionInInitializerError`; el **siguiente** uso da `NoClassDefFoundError`. **El error que ves en los logs no es el error real** — ocurrió antes, hay que buscar más arriba.

## JIT vs AOT

| | JIT (HotSpot) | AOT (Native Image) |
|---|---|---|
| Arranque | decenas/cientos de ms + warm-up | casi instantáneo (~0,1 s) |
| Rendimiento pico | muy alto, tras calentarse | inmediato, pero menor |
| Memoria en runtime | mayor | notablemente menor |
| Tiempo de build | segundos | minutos y gigabytes de RAM |
| **Reflexión, JNI, proxies** | sin problema | **requiere configuración explícita** |

Esa última fila es la que más duele: media plataforma Java (Spring, Hibernate, Jackson) se apoya en reflexión, que choca con la hipótesis de mundo cerrado.

| Prioridad | Elección |
|---|---|
| Throughput máximo, servicio de larga vida | **JIT clásico**: dejalo calentarse |
| Arranque en milisegundos, serverless, CLI | **Native Image** (AOT), si el stack lo soporta |
| Mejor arranque sin renunciar a reflexión ni tooling | **Project Leyden** (caché AOT, JDK 24+, hasta −40% de arranque) |
| Arranque instantáneo con estado ya caliente | **CRaC**, si podés coordinar los recursos |

> Durante años se vendió Native Image como el futuro inevitable. Leyden cambió el panorama: da buena parte del beneficio **sin** exigir mundo cerrado.

## Trampas y observación

```java
public class Trampa {
    static int a = b + 1;    // b todavía vale 0 (preparación), no 10
    static int b = 10;
}   // a=1, b=10
```

**Regla:** que un campo estático no dependa de otro declarado más abajo.

```bash
java -verbose:class Main            # qué clases se cargan y desde dónde
java -Xlog:class+load=info Main     # forma moderna (Java 9+)
java -Xlog:class+init=info Main     # la inicialización
java -XX:+PrintCompilation Main     # qué compila el JIT y a qué nivel
java -Xint Main                     # solo intérprete: la diferencia (10-50x) es el JIT
jps ; jcmd <pid> Thread.print       # procesos y volcado de hilos
```

**La excepción más desconcertante que existe:**

```
java.lang.ClassCastException: class Usuario cannot be cast to class Usuario
```

No es una broma: **la identidad de una clase es el par (nombre completo, class loader)**. La misma clase cargada por dos cargadores distintos son dos tipos incompatibles.

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Qué ocurre al ejecutar `java Main`?** Se crea la JVM, se reservan las áreas de memoria, el bootstrap carga el núcleo, se carga/enlaza/inicializa `Main`, y se invoca `main`.
- **¿Las tres fases del enlazado?** Verificación, preparación y resolución.
- **¿Preparación vs inicialización?** En preparación los `static` reciben el **valor por defecto**; en inicialización, el valor escrito, y se ejecutan los bloques `static`.
- **¿Cuándo se inicializa una clase?** `new`, método estático, campo estático no constante, reflexión, o contener el `main`.
- **¿Por qué la delegación al padre?** Seguridad: impide suplantar clases del núcleo desde el classpath.
- **¿`ClassNotFoundException` o `NoClassDefFoundError`?** La primera en carga explícita; la segunda cuando estaba al compilar pero no en runtime, o si su inicialización falló antes.
- **¿Qué son C1 y C2?** Los dos JIT de HotSpot: C1 rápido y moderado, C2 lento y agresivo con perfilado. La compilación por niveles usa ambos.
- **¿Qué es la deoptimización?** Descartar código compilado cuando una suposición especulativa resulta falsa, y volver al intérprete.
- **¿Termina la JVM cuando termina `main`?** No: cuando acaban todos los hilos no-daemon, o con `System.exit()`.
- **¿Se garantiza `finalize()`?** No, nunca. Y está deprecado para eliminación desde Java 18.
- **¿Por qué `0xCAFEBABE`?** Es el número mágico del `.class`; permite rechazar de inmediato lo que no es bytecode.
- **¿Por qué `Usuario` no se puede castear a `Usuario`?** Porque la identidad es (nombre + class loader).

</details>
