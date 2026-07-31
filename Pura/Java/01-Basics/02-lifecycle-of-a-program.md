# Lifecycle of a Program

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ (se señalan las novedades hasta Java 26)

**Alcance de este documento:** todo lo que le ocurre a tu código desde que lo escribes hasta que el proceso muere. Compilación, arranque de la JVM, carga de clases, enlazado, inicialización, ejecución, compilación JIT, y terminación.

**Por qué importa más de lo que parece.** Este es el tema que separa a quien "escribe Java" de quien "entiende Java". Cuando en producción el primer request tarda 2 segundos y el siguiente 8 milisegundos, cuando salta un `NoClassDefFoundError` que no tiene sentido, cuando un `static` se inicializa en un orden que no esperabas, o cuando hay que decidir entre arranque rápido y rendimiento máximo — todo eso se explica aquí.

---

## Fundamentos

### El mapa completo

Antes de entrar en detalle, este es el viaje entero. Guárdate este esquema: el resto del documento lo recorre paso a paso.

```
   TÚ                    BUILD TIME                    RUNTIME
   │                          │                            │
┌──▼───────┐         ┌────────▼────────┐       ┌───────────▼────────────┐
│ Main.java│         │     javac       │       │   java Main            │
│ (fuente) ├────────►│  compilación    ├──────►│   arranca la JVM       │
└──────────┘         └─────────────────┘       └───────────┬────────────┘
                       produce Main.class                  │
                          (bytecode)                        │
                                                            ▼
                                              ┌─────────────────────────┐
                                              │  1. LOADING             │  ClassLoader lee el
                                              │     (carga)             │  .class → objeto Class
                                              ├─────────────────────────┤
                                              │  2. LINKING             │
                                              │     • Verification      │  ¿el bytecode es legal?
                                              │     • Preparation       │  static = valores por defecto
                                              │     • Resolution        │  referencias simbólicas → reales
                                              ├─────────────────────────┤
                                              │  3. INITIALIZATION      │  static = valores reales
                                              │     (<clinit>)          │  bloques static se ejecutan
                                              ├─────────────────────────┤
                                              │  4. EJECUCIÓN           │  interpreta → perfila → JIT
                                              │     new, métodos, GC    │  compila lo "caliente"
                                              ├─────────────────────────┤
                                              │  5. TERMINACIÓN         │  hilos no-daemon acaban
                                              │     shutdown hooks      │  o System.exit()
                                              └─────────────────────────┘
```

Resumido en una tabla:

| Fase | Cuándo | Quién la hace | Qué produce |
|---|---|---|---|
| Compilación | build time | `javac` | bytecode (`.class`) |
| Carga | runtime, bajo demanda | ClassLoader | un objeto `java.lang.Class` |
| Enlazado | runtime, tras cargar | JVM | clase verificada e integrada |
| Inicialización | runtime, al primer uso real | JVM | `static` con sus valores |
| Ejecución | runtime | intérprete + JIT | resultado del programa |
| Terminación | al final | JVM | código de salida del proceso |

Un detalle que sorprende a todo el mundo: **las fases 1 a 3 no ocurren una vez al principio, sino cada vez que se usa una clase nueva**, a lo largo de toda la vida del programa. Java carga clases perezosamente.

---

### Fase 1 — Escritura y compilación

Escribes un archivo de texto plano con extensión `.java`. El nombre del archivo debe coincidir con la clase pública que contiene.

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        int a = 2;
        int b = 3;
        System.out.println(a + b);
    }
}
```

Lo compilas:

```bash
javac Main.java     # produce Main.class
```

**Qué es exactamente el bytecode.** No es código máquina y no es tu código fuente. Es un formato intermedio: instrucciones para una máquina que no existe físicamente, la **JVM**. Es lo que hace posible el *write once, run anywhere*: `javac` produce el mismo `.class` en Windows, macOS o Linux, y cada plataforma tiene su propia JVM que sabe traducirlo a su procesador.

Puedes verlo con `javap`:

```bash
javap -c Main.class
```

```
public static void main(java.lang.String[]);
  Code:
     0: iconst_2          // apila la constante 2
     1: istore_1          // la guarda en la variable local 1 (a)
     2: iconst_3          // apila la constante 3
     3: istore_2          // la guarda en la variable local 2 (b)
     4: getstatic     #7  // obtiene System.out
     7: iload_1           // apila a
     8: iload_2           // apila b
     9: iadd              // suma los dos valores de la pila
    10: invokevirtual #13 // llama a println
    13: return
```

Fíjate en algo importante: **la JVM es una máquina de pila**. No tiene registros como un procesador real; empuja valores a una pila y las instrucciones operan sobre ella. `iadd` no lleva argumentos: coge los dos valores de arriba de la pila, los suma y deja el resultado.

**Lo que `javac` NO hace.** Aquí está el malentendido más común del tema: `javac` **apenas optimiza**. No desenrolla bucles, no hace inlining de métodos, no elimina código muerto de forma agresiva. Traduce tu código a bytecode de forma bastante literal. Toda la optimización pesada ocurre después, en tiempo de ejecución, a cargo del JIT. La razón es buena: en tiempo de ejecución hay información que en compilación no existe — qué ramas se toman de verdad, qué tipos aparecen realmente, qué métodos se llaman un millón de veces.

**Errores de compilación vs errores de ejecución.** `javac` detecta lo que puede comprobarse sin ejecutar: tipos incompatibles, variables no inicializadas, métodos que no existen, excepciones checked sin tratar. Todo lo demás (dividir por cero, `null`, índices fuera de rango) explota en runtime.

---

### Fase 2 — Arranque de la JVM

```bash
java Main
```

Ese comando **no ejecuta tu clase directamente**. Lo que hace es:

1. **Crear la JVM** — un proceso del sistema operativo. Reserva las áreas de memoria (heap, metaspace…), arranca sus hilos internos (GC, compiladores JIT).
2. **Arrancar el bootstrap class loader**, que carga las clases fundamentales del propio Java: `java.lang.Object`, `java.lang.String`, `java.lang.Class`, `java.lang.System`… Cientos de clases se cargan antes de que tu código exista.
3. **Cargar tu clase `Main`** y someterla a las fases de carga, enlazado e inicialización.
4. **Buscar e invocar `main`**.

Entre el paso 1 y el paso 4 pasan decenas de milisegundos en los que tu código todavía no ha hecho nada. Eso es el **coste de arranque de la JVM**, y es el problema que intentan resolver GraalVM Native Image, CRaC y Project Leyden, que veremos más adelante.

---

### Fase 3 — Carga (Loading)

Cargar una clase es encontrar su representación binaria (los bytes del `.class`), leerla y construir en memoria un objeto `java.lang.Class` que la representa.

```
archivo Main.class  ──ClassLoader──►  instancia de java.lang.Class
   (bytes en disco)                      (objeto en memoria)
```

Ese objeto `Class` es el que obtienes con `miObjeto.getClass()` o con `Main.class`, y es la base de toda la reflexión en Java.

#### Los tres class loaders

La JVM no usa un solo cargador, sino una **jerarquía de tres**:

| Cargador | Qué carga | Notas |
|---|---|---|
| **Bootstrap** | las clases del núcleo de Java (`java.lang`, `java.util`…) | escrito en código nativo, no en Java. `String.class.getClassLoader()` devuelve `null` |
| **Platform** | módulos de la plataforma que no son del núcleo (`java.sql`, `java.xml`…) | desde Java 9 |
| **Application** (o System) | **tu código** y las librerías del classpath | es el que carga `Main` |

> **Corrección respecto a mucha documentación que circula.** Vas a leer en muchos sitios —incluida una de las referencias de este documento— que los cargadores son *Bootstrap*, *Extension* y *Application*, y que el bootstrap lee de `rt.jar`. **Eso es anterior a Java 9.** En Java 9 desapareció `rt.jar` (sustituido por el sistema de módulos y el formato `jimage`) y el *Extension class loader* pasó a llamarse **Platform class loader**, con el mecanismo de extensiones (`java.ext.dirs`) eliminado por completo. Si una fuente te habla de `rt.jar`, está describiendo Java 8 o anterior.

#### El modelo de delegación al padre

Cuando se pide cargar una clase, el cargador **no la busca él primero**: se la pide a su padre, y solo si el padre no la encuentra, la busca él.

```
  Application  ──delega──►  Platform  ──delega──►  Bootstrap
       ▲                        ▲                      │
       └────── "no la tengo" ───┴──── "no la tengo" ───┘
              (entonces la busca cada uno)
```

**Por qué existe este mecanismo:** seguridad. Si alguien mete en tu classpath una clase llamada `java.lang.String` con código malicioso, nunca se cargará — porque el bootstrap loader siempre gana la carrera y devuelve la auténtica. Sin delegación, cualquier `.jar` podría suplantar el núcleo del lenguaje.

#### Carga perezosa

Las clases **no se cargan todas al arrancar**. Se cargan la primera vez que se necesitan de verdad. Compruébalo:

```bash
java -verbose:class Main
```

Verás cientos de líneas `[class,load] java.lang.Object source: jrt:/java.base` y, mucho más abajo, tu propia clase. Y si tu programa tiene una clase que nunca se usa, nunca aparecerá en esa lista.

---

### Fase 4 — Enlazado (Linking)

Integra la clase recién cargada en el estado de ejecución de la JVM. Son **tres pasos** y conviene distinguirlos, porque en entrevistas se preguntan.

#### 4.1 Verificación (Verification)

La JVM comprueba que el bytecode es legal y seguro **antes** de ejecutarlo: que la pila no se desborda ni se queda corta, que los tipos cuadran, que no se salta a posiciones arbitrarias, que las referencias a métodos y campos tienen sentido.

Esto existe porque el bytecode **no tiene por qué venir de `javac`**. Puede venir de otro lenguaje (Kotlin, Scala, Groovy), de un generador, o de un atacante. El verificador es la frontera de seguridad de la JVM. Si falla, obtienes un `VerifyError`.

#### 4.2 Preparación (Preparation)

Se reserva memoria para los campos `static` y **se les asignan sus valores por defecto** — no los que tú escribiste:

| Tipo | Valor por defecto |
|---|---|
| `int`, `short`, `byte`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | `'\u0000'` |
| `boolean` | `false` |
| cualquier referencia | `null` |

Es decir: si escribiste `static int contador = 42;`, al terminar la preparación `contador` vale **0**, no 42. El 42 llega en la fase siguiente. Esta distinción parece un tecnicismo, pero es exactamente lo que explica ciertos bugs de inicialización que veremos en la sección práctica.

#### 4.3 Resolución (Resolution)

El bytecode no contiene direcciones de memoria, sino **referencias simbólicas**: cadenas de texto tipo `"java/lang/System.out : Ljava/io/PrintStream;"`. La resolución convierte esos nombres en referencias directas a los datos reales ya cargados.

Es **perezosa**: la JVM puede posponer la resolución de cada referencia hasta el momento en que se usa por primera vez. Por eso una clase que no existe puede no dar error hasta que se ejecuta la línea que la necesita.

---

### Fase 5 — Inicialización

Ahora sí, los `static` reciben sus valores reales y se ejecutan los bloques `static`.

El compilador recoge **todas** las asignaciones a campos estáticos y todos los bloques `static { }`, en el orden en que aparecen en el código fuente, y los mete en un único método especial llamado `<clinit>` (*class initializer*). No puedes llamarlo tú; lo llama la JVM.

```java
public class Config {
    static int a = 1;

    static {
        System.out.println("bloque estático, a vale " + a);
        a = 2;
    }

    static int b = a + 10;   // usa el valor actual de a → 12

    public static void main(String[] args) {
        System.out.println("a=" + a + ", b=" + b);
    }
}
```

```
bloque estático, a vale 1
a=2, b=12
```

**Reglas de la inicialización:**

1. **Orden textual.** Se ejecuta en el orden en que aparece en el archivo, de arriba abajo. Cambiar de sitio una línea cambia el resultado.
2. **La superclase primero.** Antes de inicializar una clase, la JVM inicializa su clase padre por completo.
3. **Ocurre una sola vez, y es thread-safe.** La JVM garantiza que `<clinit>` se ejecuta exactamente una vez aunque veinte hilos usen la clase a la vez. Esta garantía es gratuita y es la base de un patrón muy elegante que veremos en la práctica.
4. **Es perezosa.** Solo ocurre al primer uso real.

**Los cinco disparadores de la inicialización** — la clase se inicializa cuando:

- Se crea una instancia con `new`.
- Se invoca un método `static` suyo.
- Se lee o escribe un campo `static` suyo (salvo constantes `static final` de tipo primitivo o `String`, que el compilador incrusta directamente y no disparan nada).
- Se invoca mediante reflexión (`Class.forName`).
- Es la clase que contiene el `main` que arranca el programa.

Demostración de la pereza:

```java
class Pesada {
    static { System.out.println("¡Pesada se ha inicializado!"); }
    static final int CONSTANTE = 42;
    static int variable = 42;
}

public class Demo {
    public static void main(String[] args) {
        System.out.println("inicio");
        System.out.println(Pesada.CONSTANTE);   // NO inicializa: es constante de compilación
        System.out.println(Pesada.variable);    // AQUÍ sí se inicializa
    }
}
```

```
inicio
42
¡Pesada se ha inicializado!
42
```

Ese comportamiento con `static final` sorprende siempre. El compilador sustituye `Pesada.CONSTANTE` por el literal `42` en el bytecode, así que en tiempo de ejecución no hay ninguna referencia a `Pesada`. Tiene una consecuencia práctica desagradable: si cambias el valor de una constante en una librería y no recompilas el código que la usa, este seguirá con el valor viejo.

---

### Fase 6 — Instanciación y ejecución

#### Crear objetos

La forma explícita es `new`, pero Java crea objetos **implícitamente** en muchos más sitios de los que parece:

```java
String s = "hola";                        // literal → objeto String (del pool)
Integer i = 42;                           // autoboxing → Integer.valueOf(42)
String t = "a" + variable;                // concatenación → nuevo String
Runnable r = () -> System.out.println();  // lambda → objeto generado en runtime
List<String> l = new ArrayList<>();       // new explícito
```

El caso de la lambda es especialmente interesante: **se genera una clase nueva en tiempo de ejecución**. La instrucción `invokedynamic` invoca a `LambdaMetafactory`, que fabrica la clase la primera vez y la reutiliza después. O sea, tu programa carga clases que no existían en disco.

**El orden de construcción** de un objeto, que casi nadie conoce completo:

1. Se reserva memoria en el heap y todos los campos se ponen a su valor por defecto (`0`/`null`/`false`).
2. Se ejecuta el constructor de la superclase (`super()`, explícito o implícito).
3. Se ejecutan los inicializadores de instancia y las asignaciones de campos, en orden textual.
4. Se ejecuta el cuerpo del constructor.
5. Se devuelve la referencia.

```java
class Padre {
    Padre() { System.out.println("2. constructor Padre"); }
}

class Hijo extends Padre {
    int x = imprimir("3. campo de Hijo");

    { System.out.println("4. bloque de instancia"); }

    Hijo() {
        super();                                  // paso 2 (implícito si no lo escribes)
        System.out.println("5. constructor Hijo");
    }

    static int imprimir(String s) { System.out.println(s); return 0; }

    public static void main(String[] args) {
        System.out.println("1. antes del new");
        new Hijo();
    }
}
```

#### Las áreas de memoria en ejecución

Mientras el programa corre, la JVM mantiene varias zonas bien diferenciadas:

| Área | Qué guarda | Compartida entre hilos | Qué pasa si se llena |
|---|---|---|---|
| **Heap** | todos los objetos y arrays | Sí | `OutOfMemoryError: Java heap space` |
| **Stack** | marcos de método: variables locales, parámetros, dirección de retorno | **No**, una por hilo | `StackOverflowError` |
| **Metaspace** | metadatos de clases, código de métodos | Sí | `OutOfMemoryError: Metaspace` |
| **PC Register** | dirección de la instrucción actual | No, uno por hilo | — |
| **Native Method Stack** | llamadas a código nativo (JNI) | No, una por hilo | — |

Dos consecuencias prácticas que conviene interiorizar:

- **La recursión infinita da `StackOverflowError`, no `OutOfMemoryError`.** Son zonas distintas: cada llamada anidada añade un marco a la pila del hilo, que es pequeña (unos cientos de KB).
- **El Metaspace** sustituyó al viejo *PermGen* en Java 8. Vive en memoria nativa, no en el heap, y crece dinámicamente. Es donde aparecen las fugas cuando una aplicación recarga clases sin parar (típico en servidores de aplicaciones con *hot redeploy*).

#### Interpretación, perfilado y compilación JIT

Aquí está el corazón del rendimiento de Java. La JVM **no elige entre interpretar o compilar: hace las dos cosas, en este orden**.

```
  Método nuevo
      │
      ▼
 ┌─────────────┐  se ejecuta pocas veces
 │ INTÉRPRETE  │  arranque inmediato, ejecución lenta
 │  (nivel 0)  │  va contando cuántas veces se llama
 └──────┬──────┘
        │ supera el umbral → "está caliente"
        ▼
 ┌─────────────┐
 │  C1  (JIT)  │  compila rápido, optimización moderada
 │ niveles 1-3 │  sigue recogiendo datos de perfilado
 └──────┬──────┘
        │ sigue calentándose (miles de invocaciones)
        ▼
 ┌─────────────┐
 │  C2  (JIT)  │  compila lento, optimización agresiva
 │  (nivel 4)  │  usa el perfilado para especular
 └─────────────┘
```

Los dos compiladores de HotSpot tienen filosofías opuestas:

- **C1** (*client compiler*): optimiza rápido y de forma conservadora. Prioriza que el código compilado esté disponible cuanto antes.
- **C2** (*server compiler*): recoge datos de perfilado antes de actuar, y luego optimiza de forma agresiva. Tarda más, pero el código resultante es mucho mejor.

Desde Java 8, la **compilación por niveles** (*tiered compilation*) está activada por defecto y combina ambos: C1 entra pronto para que no estés interpretando eternamente, mientras se acumula el perfilado que C2 necesitará.

**Lo que el JIT puede hacer y `javac` no.** Como observa la ejecución real, puede *especular*:

- **Inlining**: mete el cuerpo de un método pequeño dentro de quien lo llama, eliminando la llamada.
- **Devirtualización**: si en la práctica una llamada polimórfica siempre resuelve a la misma implementación, la convierte en llamada directa e incluso la incrusta.
- **Eliminación de código muerto**: si una rama nunca se toma, desaparece.
- **Escape analysis**: si un objeto nunca sale del método, puede no crearlo en el heap.
- **Desenrollado de bucles**, vectorización, y más.

**Deoptimización.** Como C2 *especula*, puede equivocarse. Si asumió que una llamada siempre iba a una implementación y de pronto aparece otra, el código compilado se descarta y la ejecución vuelve al intérprete para recompilarse con la nueva información. Que un programa pueda "desoptimizarse" y luego mejorar otra vez es una de las cosas más difíciles de creer para quien viene de C++.

**La consecuencia visible: el warm-up.** Un método recién ejecutado va lento. Tras miles de invocaciones, puede ser 10 o 100 veces más rápido. Por eso el primer request de un servicio Java tarda mucho más que el número mil, y por eso medir rendimiento en Java sin calentar antes produce números sin sentido (para eso existe JMH).

#### El recolector de basura, en paralelo

Durante toda la ejecución, uno o varios hilos de GC trabajan de fondo liberando objetos inalcanzables. No tienes que liberar memoria a mano — pero tampoco tienes control preciso sobre cuándo ocurre. Es un tema propio (`08-JVM-y-Memoria`); aquí basta con saber que ocurre concurrentemente con tu código y que forma parte del ciclo de vida.

---

### Fase 7 — Terminación

#### La JVM no muere cuando `main` termina

Este es probablemente el malentendido más extendido del tema. La JVM se cierra cuando **todos los hilos no-daemon** han terminado, no cuando `main` retorna.

```java
public class NoMuere {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            try { Thread.sleep(5000); } catch (InterruptedException e) { }
            System.out.println("el hilo terminó");
        });
        t.start();
        System.out.println("main terminó");
    }
}
```

```
main terminó
        ← aquí el proceso sigue vivo 5 segundos
el hilo terminó
```

Un hilo marcado como **daemon** (`t.setDaemon(true)`) no cuenta: la JVM lo mata sin contemplaciones al salir. Los hilos del GC son daemon, por ejemplo. Es una fuente clásica de procesos "colgados" en producción: un pool de hilos no cerrado impide que el proceso muera.

#### Las dos formas de terminar

| Forma | Qué ocurre |
|---|---|
| **Terminación normal** | todos los hilos no-daemon acaban. Se ejecutan los shutdown hooks. Código de salida 0 |
| **`System.exit(n)`** | inicia el apagado inmediatamente. Se ejecutan los shutdown hooks. Código de salida `n` |

`System.exit(n)` delega en `Runtime.getRuntime().exit(n)`. Por convención, `0` significa éxito y cualquier otro valor, error — lo que permite encadenar tu programa en scripts.

#### Shutdown hooks

Un *shutdown hook* es un hilo que la JVM ejecuta justo antes de morir. Sirve para cerrar recursos, vaciar buffers o avisar a otro sistema.

```java
public class ConHook {
    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("cerrando conexiones...");
        }));

        System.out.println("trabajando");
    }
}
```

```
trabajando
cerrando conexiones...
```

Se ejecutan tanto en terminación normal como con `System.exit()`, y también cuando el sistema envía `SIGTERM` (por ejemplo, `docker stop`). **No** se ejecutan si el proceso recibe `SIGKILL` (`kill -9`) ni si la JVM se cae por un error grave. Nunca dependas de ellos para datos críticos.

#### `finalize()`: no lo uses

Vas a encontrarlo en documentación antigua descrito como "el método que se llama antes de que el GC destruya el objeto". Estado real en 2026:

**`finalize()` está deprecado para su eliminación** desde Java 18 ([JEP 421](https://openjdk.org/jeps/421)). Sigue existiendo en Java 25, pero con avisos de deprecación y con la posibilidad de desactivarlo por línea de comandos. Los problemas que llevaron a esa decisión son de seguridad, rendimiento y fiabilidad: **su ejecución no está garantizada**, puede no ocurrir nunca, ocurre en un hilo impredecible y puede resucitar objetos.

Las alternativas correctas:

| En vez de | Usa |
|---|---|
| `finalize()` para cerrar recursos | `AutoCloseable` + `try-with-resources` |
| `finalize()` como red de seguridad | `java.lang.ref.Cleaner` |
| control fino sobre inalcanzabilidad | `PhantomReference` |

```java
try (var conexion = abrirConexion()) {
    conexion.consultar();
}   // se cierra aquí, garantizado, incluso si hay excepción
```

#### Descarga de clases (Class Unloading)

Una clase se descarga cuando **su class loader se vuelve inalcanzable** y el GC lo recolecta. Las cargadas por el bootstrap loader nunca se descargan. Esto importa en servidores de aplicaciones y en cualquier sistema que recargue código en caliente: si algo mantiene una referencia al class loader viejo, sus clases nunca se liberan y el Metaspace crece hasta reventar. Es la causa canónica de los `OutOfMemoryError: Metaspace` tras varios redespliegues.

---

## JIT vs AOT: las dos filosofías

Ya vimos que el JIT compila en tiempo de ejecución. La alternativa es compilar **antes** de ejecutar: *Ahead-Of-Time*.

### Compilación AOT con GraalVM Native Image

GraalVM analiza tu aplicación en tiempo de build bajo la **hipótesis de mundo cerrado** (*closed-world assumption*): asume que todas las clases que se van a usar están disponibles en ese momento. Con esa información produce un **ejecutable nativo** para un sistema operativo y arquitectura concretos, que no necesita JVM instalada.

| | JIT (HotSpot) | AOT (Native Image) |
|---|---|---|
| **Arranque** | decenas o cientos de ms, más warm-up | casi instantáneo (~0,1 s) |
| **Rendimiento pico** | muy alto, tras calentarse | inmediato, pero normalmente menor |
| **Optimización** | dinámica, con datos reales de ejecución | estática, sin información de ejecución |
| **Memoria en runtime** | mayor | notablemente menor |
| **Tamaño de imagen** | grande (incluye la JVM) | pequeño |
| **Tiempo de build** | segundos | minutos, y consume gigabytes de RAM |
| **Recolectores disponibles** | muchos (G1, ZGC, Shenandoah…) | pocos |
| **Depuración y tooling** | excelente, ecosistema maduro | limitada |
| **Reflexión, JNI, proxies dinámicos** | sin problema | requiere configuración explícita |

Esa última fila es la que más duele en la práctica. Media plataforma Java (Spring, Hibernate, Jackson) se apoya en reflexión y generación dinámica de clases, que chocan de frente con la hipótesis de mundo cerrado. Los frameworks lo resuelven generando metadatos de configuración en tiempo de build, pero es trabajo extra y una fuente constante de fricción.

**Cuándo compensa AOT:** funciones serverless (donde pagas por el arranque), CLIs, microservicios que escalan a cero, contenedores que arrancan y mueren constantemente.
**Cuándo no:** servicios de larga vida con alto throughput, donde el JIT tiene tiempo de sobra para calentarse y acaba ganando.

### El camino intermedio: Project Leyden

Aquí es donde las referencias de este documento se quedan cortas, porque el panorama cambió en 2025-2026. **Project Leyden** ataca el problema del arranque sin renunciar a la JVM ni a sus características dinámicas.

La idea: hacer una **ejecución de entrenamiento** de la aplicación, durante la cual la JVM registra qué clases se cargan y enlazan, y guardarlo en una **caché AOT**. En las ejecuciones siguientes, la JVM lee esa caché y se salta el trabajo de carga y enlazado.

| JEP | Versión | Qué aporta |
|---|---|---|
| [JEP 483](https://openjdk.org/jeps/483) | JDK 24 | carga y enlazado AOT. Hasta un **40% menos de tiempo de arranque**, sin tocar el código |
| JEP 514 | JDK 25 | simplifica el flujo de trabajo (menos pasos manuales) |
| JEP 515 | JDK 25 | perfilado AOT: el JIT calienta antes porque hereda perfiles de la ejecución de entrenamiento |
| JEP 516 | JDK 26 | caché independiente del GC (habilita ZGC) y caché base incluida en el propio JDK |

Lo interesante de este enfoque es que **no exige mundo cerrado**: la reflexión, los proxies dinámicos y la generación de clases en runtime siguen funcionando. No alcanza el arranque de Native Image, pero no obliga a renunciar a nada.

### CRaC: congelar y descongelar

Otra vía: **CRaC** (*Coordinated Restore at Checkpoint*) toma una instantánea del proceso Java ya arrancado y calentado, y la restaura después. El arranque pasa de minutos a milisegundos porque, literalmente, no hay arranque: se reanuda un proceso que ya estaba listo. A cambio, exige que la aplicación coopere cerrando y reabriendo sus recursos externos alrededor del checkpoint.

---

## En la práctica

### Observar cada fase con tus propias manos

Esta es la mejor forma de que el tema deje de ser abstracto. Todos estos comandos vienen con el JDK:

```bash
# FASE 1 — ver el bytecode generado
javap -c Main.class              # instrucciones
javap -v Main.class              # todo: constant pool, versión, flags, atributos

# FASE 3 — ver qué clases se cargan y desde dónde
java -verbose:class Main
java -Xlog:class+load=info Main            # forma moderna (Java 9+)

# FASE 4-5 — ver la inicialización
java -Xlog:class+init=info Main

# FASE 6 — ver qué compila el JIT y a qué nivel
java -XX:+PrintCompilation Main

# Forzar solo intérprete, para comparar rendimiento
java -Xint Main

# Ver el proceso vivo
jps                              # lista procesos Java
jcmd <pid> VM.class_hierarchy    # jerarquía de clases cargadas
jcmd <pid> Thread.print          # volcado de hilos
```

Un experimento que recomiendo hacer una vez: coge un bucle que llame a un método un millón de veces, mídelo con `-Xint` y sin él. La diferencia (fácilmente 10-50x) es el JIT haciendo su trabajo, y no se olvida.

### `ClassNotFoundException` vs `NoClassDefFoundError`

Pregunta clásica de entrevista, y confusión diaria real. Las dos dicen "falta una clase", pero por motivos distintos:

| | `ClassNotFoundException` | `NoClassDefFoundError` |
|---|---|---|
| **Tipo** | `Exception` (checked) | `Error` |
| **Cuándo** | carga **explícita**: `Class.forName("...")`, `loadClass()` | la clase **estaba** al compilar pero no está en runtime |
| **Causa típica** | nombre mal escrito, driver JDBC ausente | dependencia que falta en el classpath, o **la clase falló al inicializarse** |

El segundo caso de `NoClassDefFoundError` es el traicionero. Si el `<clinit>` de una clase lanza una excepción, obtienes un `ExceptionInInitializerError`. Pero el **siguiente** intento de usar esa clase da `NoClassDefFoundError`, porque la JVM la marcó como errónea. Resultado: el error que ves en los logs no es el error real — este ocurrió antes, y hay que buscar más arriba.

```java
class Config {
    static final String URL = System.getenv("DB_URL").trim();  // NPE si la variable no existe
}
// Primer uso        → ExceptionInInitializerError (causado por NullPointerException)
// Usos siguientes   → NoClassDefFoundError: Could not initialize class Config
```

### El idioma del *holder*: pereza y thread-safety gratis

Aprovechando que `<clinit>` está garantizado thread-safe y perezoso, existe una forma elegante de crear un singleton con inicialización diferida y sin `synchronized`:

```java
public class Servicio {
    private Servicio() { }

    private static class Holder {
        static final Servicio INSTANCIA = new Servicio();   // se crea al primer acceso
    }

    public static Servicio getInstancia() {
        return Holder.INSTANCIA;
    }
}
```

`Holder` no se inicializa hasta que alguien llama a `getInstancia()`. Y cuando ocurre, la JVM garantiza que solo pasa una vez aunque haya cien hilos compitiendo. Cero bloqueos escritos por ti, cero coste. Es el ejemplo más bonito de por qué entender el ciclo de vida da código mejor.

### Por qué el primer request es lento

Consecuencia directa de todo lo anterior, y algo que vas a explicar en tu equipo tarde o temprano:

1. Al recibir el primer request, muchas clases **todavía no están cargadas**. Se cargan, verifican, enlazan e inicializan en ese momento.
2. El código va **interpretado**: aún no hay nada compilado.
3. Las cachés están vacías, los pools de conexiones se están llenando.

Hacia el request número mil, las clases están cargadas y los métodos calientes compilados por C2. De ahí prácticas como el *warm-up* previo a poner una instancia en producción, o no enviarle tráfico real hasta que haya procesado tráfico sintético.

### Trampas de inicialización estática

```java
public class Trampa {
    static int a = b + 1;    // b todavía vale 0 (preparación), no 10
    static int b = 10;

    public static void main(String[] args) {
        System.out.println("a=" + a + ", b=" + b);   // a=1, b=10
    }
}
```

`a` vale 1, no 11. La razón es exactamente lo que vimos en la fase de preparación: cuando se ejecuta la primera línea, `b` ya tiene memoria reservada pero todavía su valor por defecto. **Regla práctica:** no hagas que un campo estático dependa de otro declarado más abajo.

---

## Avanzado

### Anatomía de un archivo `.class`

Con `javap -v` puedes ver la estructura completa. Todo `.class` empieza por el número mágico **`0xCAFEBABE`** — literalmente esos cuatro bytes, una broma de los diseñadores originales que lleva ahí desde 1995. Sirve para que la JVM rechace de inmediato cualquier archivo que no sea bytecode.

```
magic                0xCAFEBABE
minor/major version  la versión del formato → determina el JDK mínimo
constant pool        tabla de todas las constantes, nombres y referencias simbólicas
access flags         public, final, abstract...
this_class           esta clase
super_class          su padre
interfaces           las que implementa
fields               los campos
methods              los métodos, cada uno con su bytecode
attributes           metadatos: código fuente, anotaciones, tabla de líneas...
```

El **constant pool** es el corazón del archivo: todas las cadenas, nombres de método y referencias a otras clases viven ahí, y el bytecode las referencia por índice (los `#7`, `#13` que veías en la salida de `javap -c`).

El campo de versión explica un error muy frecuente: `UnsupportedClassVersionError` significa que intentas ejecutar bytecode compilado con un JDK más nuevo que tu JVM. La versión mayor 61 corresponde a Java 17, la 65 a Java 21, la 69 a Java 25.

### Los niveles de compilación por dentro

La compilación por niveles tiene cinco escalones, y `-XX:+PrintCompilation` los muestra:

| Nivel | Quién | Perfilado |
|---|---|---|
| 0 | intérprete | contadores básicos |
| 1 | C1 | sin perfilado (métodos triviales: se compilan y se olvidan) |
| 2 | C1 | perfilado limitado |
| 3 | C1 | perfilado completo — el camino habitual antes de C2 |
| 4 | C2 | ninguno (ya usa el perfilado recogido) |

El recorrido típico de un método caliente es `0 → 3 → 4`. Que un método salte a nivel 1 y se quede ahí significa que la JVM lo consideró trivial (un getter, por ejemplo): no merece la pena perfilar algo que se va a incrustar de todos modos.

### On-Stack Replacement (OSR)

¿Qué pasa si un método se llama **una sola vez** pero contiene un bucle de mil millones de iteraciones? Con el mecanismo normal nunca se compilaría, porque el contador de invocaciones vale 1.

Para eso existe el **OSR**: la JVM cuenta también los saltos hacia atrás (las iteraciones del bucle) y, si superan el umbral, compila el método **y sustituye el marco de pila en caliente**, saltando del código interpretado al compilado en mitad de la ejecución. Es una de las piezas de ingeniería más impresionantes de HotSpot.

### Class loaders personalizados: cómo funcionan los frameworks

Los class loaders no son solo teoría; son la base de casi toda la infraestructura Java:

- **Servidores de aplicaciones** (Tomcat, WildFly) dan a cada aplicación desplegada su propio class loader. Así dos aplicaciones pueden usar versiones distintas de la misma librería sin colisionar, y desplegar una no afecta a la otra.
- **Spring Boot fat jar** usa un class loader propio que sabe leer `.jar` anidados dentro del `.jar` principal — algo que el cargador estándar no soporta.
- **OSGi** construye todo su sistema de módulos sobre cargadores aislados.
- **Hot reload** (JRebel, DevTools) recarga clases descartando y recreando cargadores.

Y de ahí sale una de las excepciones más desconcertantes que existen:

```
java.lang.ClassCastException: class Usuario cannot be cast to class Usuario
```

No es un error de la JVM ni una broma. **La identidad de una clase es el par (nombre completamente cualificado, class loader).** La misma clase cargada por dos cargadores distintos produce dos tipos **incompatibles** que casualmente se llaman igual. Si alguna vez ves ese mensaje, ya sabes dónde mirar.

### Por qué el bytecode no es un detalle irrelevante

Todo el ecosistema JVM se apoya en que el bytecode es un formato público y estable:

- **Otros lenguajes** (Kotlin, Scala, Clojure, Groovy) compilan a bytecode y por eso interoperan con Java y con todas sus librerías.
- **Los agentes de instrumentación** (`-javaagent`) modifican el bytecode al cargarlo: así funcionan los profilers, las herramientas de APM (New Relic, Datadog) y las de cobertura (JaCoCo). Tu código se reescribe en el aire, entre la carga y el enlazado.
- **Librerías como Mockito o Hibernate** generan clases en tiempo de ejecución (proxies, subclases) fabricando bytecode al vuelo.

---

## Trade-offs y entrevista

### La decisión real: arranque contra rendimiento sostenido

Es el trade-off central del tema, y ya no es binario:

| Prioridad | Elección |
|---|---|
| Throughput máximo, servicio de larga vida | **JIT clásico** (HotSpot). Déjalo calentarse |
| Arranque en milisegundos, serverless, CLI | **Native Image** (AOT), si el stack lo soporta |
| Arranque mejor sin renunciar a reflexión ni tooling | **Project Leyden** (caché AOT, JDK 24+) |
| Arranque instantáneo con estado ya caliente | **CRaC**, si puedes coordinar los recursos |

Una nota de honestidad que conviene tener clara: durante años se vendió Native Image como el futuro inevitable de Java. La aparición de Leyden cambió el panorama, porque ofrece buena parte del beneficio de arranque **sin** exigir mundo cerrado ni renunciar a la dinámica del lenguaje. Para la mayoría de aplicaciones empresariales, hoy es la vía de menor fricción.

### La lentitud del arranque como precio del dinamismo

Todo lo que hace lenta la puesta en marcha de Java —carga perezosa, verificación, enlazado, interpretación previa— es exactamente lo que le da su flexibilidad: cargar código en caliente, instrumentar sin recompilar, generar clases en runtime, optimizar con datos reales. C++ arranca al instante pero no puede hacer nada de eso. No es que Java esté mal diseñado: eligió otro punto del espacio de compromisos.

### Preguntas de entrevista

**¿Qué ocurre exactamente al ejecutar `java Main`?**
Se crea la JVM, se reservan las áreas de memoria, el bootstrap loader carga las clases del núcleo, luego se carga, enlaza e inicializa `Main`, y por último se invoca `main`.

**¿Cuáles son las tres fases del enlazado?**
Verificación, preparación y resolución.

**Diferencia entre preparación e inicialización.**
En la preparación los campos `static` reciben sus **valores por defecto** (0, null, false). En la inicialización reciben los valores que escribiste y se ejecutan los bloques `static`.

**¿Cuándo se inicializa una clase?**
Al crear una instancia, invocar un método estático, acceder a un campo estático no constante, invocarla por reflexión, o si contiene el `main`.

**¿Por qué existe el modelo de delegación al padre?**
Por seguridad: impide que código del classpath suplante clases del núcleo como `java.lang.String`.

**¿`ClassNotFoundException` o `NoClassDefFoundError`?**
La primera, en carga explícita (`Class.forName`). La segunda, cuando la clase estaba al compilar pero no en runtime — o cuando su inicialización falló antes.

**¿Java es compilado o interpretado?**
Ambos. `javac` compila a bytecode; la JVM interpreta ese bytecode y compila a nativo lo que se ejecuta con frecuencia.

**¿Qué son C1 y C2?**
Los dos compiladores JIT de HotSpot. C1 compila rápido con optimización moderada; C2 compila lento con optimización agresiva basada en perfilado. La compilación por niveles usa ambos.

**¿Qué es la deoptimización?**
Descartar código compilado cuando una suposición especulativa resulta falsa, volviendo al intérprete para recompilar con mejor información.

**¿Termina la JVM cuando termina `main`?**
No. Termina cuando acaban todos los hilos **no-daemon**, o cuando se llama a `System.exit()`.

**¿Se garantiza que se ejecute `finalize()`?**
No, nunca estuvo garantizado. Además está deprecado para eliminación desde Java 18. Usa `try-with-resources` o `Cleaner`.

**¿Por qué `0xCAFEBABE`?**
Es el número mágico que identifica un archivo `.class`. Permite rechazar de inmediato archivos que no son bytecode.

**¿Por qué el mismo `Usuario` no se puede castear a `Usuario`?**
Porque la identidad de una clase es (nombre + class loader). Cargada por dos loaders distintos, son dos tipos diferentes.

---

## Alcance y temas relacionados

| Lo que te habrás preguntado leyendo esto | Dónde se estudia |
|---|---|
| Cómo se escribe la sintaxis que aquí se compila | `01-basic-syntax.md` |
| Heap, stack, generaciones y recolectores en detalle | bloque `08-JVM-y-Memoria` |
| Hilos daemon, pools y concurrencia | bloque `07-Concurrencia` |
| `try-with-resources` y `AutoCloseable` | bloque `05-Excepciones` |
| Paquetes, JPMS, Maven y estructura del build | bloque `10-Modulos-y-Build` |

---

## Fuentes

Páginas efectivamente leídas para este documento:

**Referencias aportadas**
- [Life cycle of a Java program — StarterTutorials](https://www.startertutorials.com/corejava/life-cycle-java-program.html) — las tres fases básicas: edición, compilación, ejecución
- [How the JVM executes Java code — César Soto Valero](https://www.cesarsotovalero.net/blog/how-the-jvm-executes-java-code.html) — la estructura loading / linking / initialization / instantiation / unloading, los class loaders y el ejemplo de cargador personalizado
- [Compilation in Java: JIT vs AOT — BellSoft](https://bell-sw.com/blog/compilation-in-java-jit-vs-aot/) — C1 y C2, tiered compilation, ventajas y desventajas de AOT, GraalVM Native Image, CRaC

**Consultas adicionales para actualizar el material**
- [JEP 483: Ahead-of-Time Class Loading & Linking](https://openjdk.org/jeps/483) y [Project Leyden](https://foojay.io/pedia/project-leyden/) — la caché AOT de JDK 24-26
- [Java Applications Can Start 40% Faster in Java 24 — InfoQ](https://www.infoq.com/news/2025/03/java-24-leyden-ships/)
- [Run Into the New Year with Java's Ahead-of-Time Cache Optimizations — Inside.java](https://inside.java/2026/01/09/run-aot-cache/)
- [JEP 421: Deprecate Finalization for Removal](https://openjdk.org/jeps/421) — estado real de `finalize()`

**Referencia normativa**
- [The Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se21/html/) — el capítulo 5 (*Loading, Linking, and Initializing*) es la fuente definitiva de las fases 3 a 5

> **Nota sobre las fuentes.** Dos de las referencias describen el estado de Java 8: mencionan `rt.jar` y el *Extension class loader*, ambos eliminados en Java 9. Y el artículo de JIT vs AOT es anterior a Project Leyden, que cambió el panorama de forma sustancial. En este documento se ha corregido y actualizado lo necesario, señalándolo donde corresponde.
