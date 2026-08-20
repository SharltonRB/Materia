# Type Casting

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25

**Alcance de este documento.** Los temas anteriores establecieron *qué* tipos existen ([Data Types and Variables](03-data-types-and-variables.md)) y *dónde vive* cada variable ([Variables and Scopes](04-variables-and-scopes.md)). Este responde la pregunta que aparece en cuanto empezás a combinar tipos: **cómo se pasa un valor de un tipo a otro, cuándo Java lo hace solo, cuándo te obliga a pedirlo, y qué se rompe cuando lo pedís mal**.

Es un tema traicionero. La sintaxis se aprende en cinco minutos — `(int) miDouble` y listo — y por eso casi todos los tutoriales lo despachan en una página. Pero detrás de esos paréntesis hay un conjunto de reglas del lenguaje que produce, en producción y en silencio: importes de dinero mal calculados, IDs de base de datos corruptos, `NullPointerException` en líneas donde no hay ningún `null` visible, `ClassCastException` a doscientas líneas del código que causó el problema, y comparaciones de enteros que dan `true` con 127 y `false` con 128.

Vamos a cubrir el modelo completo: las reglas exactas del lenguaje, qué hace el compilador y qué hace la JVM, los errores clásicos uno por uno con su causa real, y los criterios de diseño que distinguen a alguien que *escribe* un cast de alguien que *decide* si debería haber un cast.

**Sobre la verificación.** Todos los outputs, mensajes de error del compilador y volcados de bytecode de este documento fueron ejecutados realmente en un JDK 25, no copiados de tutoriales. Esto importa más de lo que parece: al preparar el documento encontré errores técnicos en tres de las cuatro fuentes de referencia más populares sobre el tema. Los señalo en su momento, porque saber *dónde* mienten los tutoriales es parte de dominar el tema.

---

## Índice

**Parte I — Fundamentos**
1. [Qué problema resuelve el casting](#1-qué-problema-resuelve-el-casting)
2. [Conversión y cast no son lo mismo](#2-conversión-y-cast-no-son-lo-mismo)
3. [El catálogo completo de conversiones](#3-el-catálogo-completo-de-conversiones)
4. [Los contextos: dónde ocurren las conversiones](#4-los-contextos-dónde-ocurren-las-conversiones)

**Parte II — Conversiones entre primitivos**

5. [Widening: la conversión automática](#5-widening-la-conversión-automática)
6. [El mito de la cadena única](#6-el-mito-de-la-cadena-única)
7. [Widening que sí pierde datos](#7-widening-que-sí-pierde-datos)
8. [Narrowing: la conversión manual](#8-narrowing-la-conversión-manual)
9. [Truncar no es redondear](#9-truncar-no-es-redondear)
10. [`char`, el tipo raro](#10-char-el-tipo-raro)
11. [Promoción numérica: la trampa invisible](#11-promoción-numérica-la-trampa-invisible)
12. [Los tres casos donde el compilador afloja](#12-los-tres-casos-donde-el-compilador-afloja)
13. [Cómo hacer un narrowing seguro](#13-cómo-hacer-un-narrowing-seguro)

**Parte III — Conversiones entre referencias**

14. [Tipo estático y tipo dinámico](#14-tipo-estático-y-tipo-dinámico)
15. [Upcasting](#15-upcasting)
16. [Downcasting](#16-downcasting)
17. [Qué pasa realmente en runtime: `checkcast`](#17-qué-pasa-realmente-en-runtime-checkcast)
18. [`instanceof` y pattern matching](#18-instanceof-y-pattern-matching)
19. [Arrays: covarianza y `ArrayStoreException`](#19-arrays-covarianza-y-arraystoreexception)
20. [Casting reflexivo: `Class.cast`](#20-casting-reflexivo-classcast)

**Parte IV — Boxing y unboxing**

21. [Qué son y dónde ocurren](#21-qué-son-y-dónde-ocurren)
22. [Los puzzles del autoboxing](#22-los-puzzles-del-autoboxing)

**Parte V — Genéricos y type erasure**

23. [Erasure: por qué el cast reaparece](#23-erasure-por-qué-el-cast-reaparece)
24. [Unchecked cast y heap pollution](#24-unchecked-cast-y-heap-pollution)

**Parte VI — Cierre**

25. [Lo que parece casting y no lo es](#25-lo-que-parece-casting-y-no-lo-es)
26. [Casos de uso reales](#26-casos-de-uso-reales)
27. [Anti-patrones](#27-anti-patrones)
28. [Checklist y tabla de decisión](#28-checklist-y-tabla-de-decisión)
29. [Autoevaluación](#29-autoevaluación)
30. [Fuentes](#30-fuentes)

---

# Parte I — Fundamentos

## 1. Qué problema resuelve el casting

Java es un lenguaje **estáticamente tipado**: cada variable tiene un tipo declarado, y el compilador verifica *antes de ejecutar* que solo le asignes valores compatibles. Es lo que hace que este código no llegue nunca a producción:

```java
int edad = "veinticinco";   // error de compilación, no de runtime
```

Esa rigidez es una virtud. Pero crea un problema práctico inmediato: **el mundo real mezcla tipos todo el tiempo**.

- Un usuario introduce `"42"` por teclado: eso es un `String`, y vos necesitás un `int`.
- Una división de dos `int` te da un `int`, pero querés el porcentaje con decimales.
- Una librería devuelve `Object` porque se escribió antes de que existieran los genéricos.
- Leés un archivo binario byte a byte, pero necesitás interpretarlo como enteros de 32 bits.
- Guardás números en un `ArrayList`, que solo acepta objetos, no primitivos.

El **type casting** (o más precisamente, el sistema de conversiones de tipo) es el mecanismo que Java ofrece para cruzar esas fronteras sin renunciar a la verificación estática. La idea central es esta:

> Java permite las conversiones **seguras** en silencio, exige que pidas explícitamente las **peligrosas**, y prohíbe en tiempo de compilación las **imposibles**.

Ese es todo el diseño. El resto del documento es aprender qué cae en cada categoría — y descubrir que la línea entre "segura" y "peligrosa" no está donde uno esperaría.

## 2. Conversión y cast no son lo mismo

Esta distinción es la primera que separa a alguien que copia código de alguien que lo entiende, y prácticamente ninguna fuente introductoria la hace.

- Una **conversión de tipo** (*type conversion*) es el acto de tomar un valor de tipo `S` y producir un valor de tipo `T`. Es un concepto del lenguaje.
- Un **cast** es un *operador* concreto: los paréntesis con un nombre de tipo dentro, `(T) expresión`. Es sintaxis.

La relación entre ambos no es uno a uno, y ahí está la sutileza.

**Hay conversiones sin cast.** Estas líneas contienen conversiones y no tienen paréntesis por ningún lado:

```java
int i = 9;
double d = i;                   // conversión int → double
Object o = "hola";              // conversión String → Object
Integer boxed = 5;              // conversión int → Integer
System.out.println("n=" + i);   // conversión int → String
```

**Y hay casts que no convierten nada.** Este cast no transforma ningún dato:

```java
Object o = obtenerAlgo();
String s = (String) o;     // no crea un String nuevo; verifica y reetiqueta
```

El objeto en memoria es exactamente el mismo antes y después. Lo único que cambió es qué tipo cree el compilador que tiene, más una verificación en runtime. Volveremos sobre esto en la [sección 15](#15-upcasting), donde el bytecode lo demuestra.

La formulación que conviene fijar:

> El cast es **una forma de pedir** una conversión, no la conversión en sí. Java hace muchas conversiones sin que las pidas, y algunos casts no producen ninguna conversión de datos.

Guardá esta distinción, porque explica por qué `(int) "42"` no compila (no existe ninguna conversión de `String` a `int`, así que el operador cast no tiene nada que invocar) mientras que `Integer.parseInt("42")` sí funciona (es una llamada a un método que *parsea*, que es otra cosa completamente distinta).

## 3. El catálogo completo de conversiones

El [capítulo 5 del Java Language Specification](https://docs.oracle.com/javase/specs/jls/se21/html/jls-5.html) — el documento normativo del lenguaje — define **trece tipos de conversión**. No hace falta memorizarlos, pero sí conviene ver el mapa entero una vez, porque casi todos los tutoriales cubren solo dos de los trece y dejan la impresión de que el tema es más chico de lo que es.

| # | Conversión | Qué hace | ¿Necesita cast? | ¿Puede fallar en runtime? |
|---|---|---|---|---|
| 1 | **Identity** | `T` → `T`, no hace nada | No | No |
| 2 | **Widening primitive** | `int` → `long`, `int` → `double`… | No | No (pero puede perder precisión) |
| 3 | **Narrowing primitive** | `double` → `int`, `int` → `byte`… | **Sí** | No lanza, pero corrompe el valor |
| 4 | **Widening + narrowing** | solo `byte` → `char` | **Sí** | No |
| 5 | **Widening reference** | `Perro` → `Animal` (upcast) | No | No |
| 6 | **Narrowing reference** | `Animal` → `Perro` (downcast) | **Sí** | **Sí: `ClassCastException`** |
| 7 | **Boxing** | `int` → `Integer` | No | No (salvo `OutOfMemoryError`) |
| 8 | **Unboxing** | `Integer` → `int` | No | **Sí: `NullPointerException`** |
| 9 | **Unchecked** | `List` → `List<String>` | No (avisa) | **Sí, más tarde y lejos** |
| 10 | **String** | cualquier cosa → `String` en `+` | No | No |
| 11 | **Value set** | ajustes internos de punto flotante | No | No |
| 12 | **Capture** | wildcards de genéricos | No | No |
| 13 | **Numeric promotion** | `byte` → `int` antes de operar | No | No |

Las tres columnas de la derecha son el mapa mental real de este documento. Fijate en el patrón:

- Las conversiones que **no necesitan cast** son las que el lenguaje considera seguras.
- El requisito de cast es Java diciéndote: *"esto puede salir mal; si lo escribís, asumo que sabés lo que hacés"*.
- Solo **tres** conversiones pueden explotar en runtime: narrowing reference (`ClassCastException`), unboxing (`NullPointerException`) y unchecked (`ClassCastException` diferida).

Y notá la asimetría más importante de todas, la que explica la mitad de los bugs de este tema:

> El narrowing **primitivo** nunca lanza una excepción. Simplemente te da un número equivocado y sigue adelante. El narrowing **de referencias** sí lanza. Es decir: el caso de los objetos falla ruidosamente, y el de los números falla en silencio.

Un `ClassCastException` es un mal día. Un `(byte) 300` que devuelve `44` sin decir nada puede ser un mes de auditoría contable.

## 4. Los contextos: dónde ocurren las conversiones

Java no aplica las mismas reglas en todas partes. El JLS define **cinco contextos** de conversión, y cada uno permite un subconjunto distinto. Esto explica varios "¿por qué esto compila acá y no allá?" que confunden mucho.

| Contexto | Dónde aparece | Qué permite |
|---|---|---|
| **Assignment** | `T x = expr;`, `return expr;`, argumento de método | Todas las widening, boxing, unboxing, y narrowing **de constantes** |
| **Invocation** | resolución de sobrecarga de métodos | Como assignment, **pero sin** narrowing de constantes |
| **Casting** | `(T) expr` | Todo lo anterior **más** las narrowing explícitas |
| **String** | operando de `+` cuando el otro es `String` | Cualquier tipo → `String` |
| **Numeric** | operandos de `+`, `-`, `*`, `/`, `<`… | Promoción numérica |

El caso práctico donde esto muerde:

```java
byte b = 42;                 // ✔ compila: contexto de asignación, 42 es constante y cabe

void recibir(byte b) { }
recibir(42);                 // ✘ NO compila: contexto de invocación, sin narrowing de constantes
recibir((byte) 42);          // ✔ con cast explícito sí
```

La misma constante `42`, aceptada en un sitio y rechazada en el otro. No es un capricho: en la resolución de sobrecarga, permitir narrowing implícito haría ambigua la elección entre `recibir(byte)`, `recibir(short)` y `recibir(int)`.

---

# Parte II — Conversiones entre primitivos

Esta es la parte que todos los tutoriales cubren. La vamos a cubrir bien, que es distinto.

## 5. Widening: la conversión automática

Una **widening primitive conversion** (conversión de ensanchamiento) lleva un valor a un tipo con **más capacidad**. Java la hace sola, sin pedirte nada, porque en el caso general no hay riesgo de que el valor no quepa.

```java
int numeroEntero = 9;
double numeroDecimal = numeroEntero;   // widening automática

System.out.println(numeroEntero);      // 9
System.out.println(numeroDecimal);     // 9.0
```

Fijate en el `9.0`: el valor numérico es el mismo, pero la representación interna cambió por completo. Un `int` guarda `9` como el patrón de bits `00000000 00000000 00000000 00001001`. Un `double` lo guarda en formato IEEE 754, con signo, exponente y mantisa. La conversión no es "añadir un `.0`": es reconstruir el número en otro formato.

El JLS define **exactamente 19** widening primitive conversions. Vale la pena verlas todas, porque la lista real no coincide con la que enseñan los tutoriales:

| Desde | Hacia |
|---|---|
| `byte` | `short`, `int`, `long`, `float`, `double` |
| `short` | `int`, `long`, `float`, `double` |
| `char` | `int`, `long`, `float`, `double` |
| `int` | `long`, `float`, `double` |
| `long` | `float`, `double` |
| `float` | `double` |

5 + 4 + 4 + 3 + 2 + 1 = 19. **`boolean` no aparece en ninguna fila ni en ninguna columna**: en Java, `boolean` no se convierte a nada ni nada se convierte a `boolean`. No existe el "`0` es falso" de C o JavaScript.

```java
boolean activo = 1;    // ✘ error: int cannot be converted to boolean
int n = true;          // ✘ error: boolean cannot be converted to int
```

## 6. El mito de la cadena única

Abrí las cuatro fuentes de referencia sobre este tema y las cuatro te van a dar alguna versión de esta cadena:

```
byte → short → char → int → long → float → double
```

W3Schools la presenta así. TutorialsPoint la escribe como `byte > short > char > int > long > float > double`. Simplilearn repite la misma secuencia.

**El eslabón `short → char` es falso.** No existe. Y `char → short` tampoco. Verificado contra el compilador:

```java
char ch = 'a';
short s = ch;      // ✘ error: possible lossy conversion from char to short

short s2 = 1;
char ch2 = s2;     // ✘ error: possible lossy conversion from short to char
```

Ambas direcciones son **narrowing**, no widening, y el compilador las rechaza sin cast explícito. La razón es que `char` y `short` ocupan los mismos 16 bits pero cubren rangos incompatibles:

| Tipo | Bits | Rango |
|---|---|---|
| `short` | 16 | −32.768 … 32.767 (con signo) |
| `char` | 16 | 0 … 65.535 (**sin signo**) |

`char` es el único tipo sin signo de Java. Un `short` de `-1` no cabe en un `char`, y un `char` de `40000` no cabe en un `short`. Ninguno contiene al otro, así que ninguna dirección puede ser una ampliación.

La estructura verdadera no es una cadena, es un **grafo con dos entradas** que confluyen en `int`:

```
byte ──→ short ──┐
                 ├──→ int ──→ long ──→ float ──→ double
char ────────────┘
```

`byte` y `char` son hermanos que nunca se hablan directamente. De hecho `byte → char` es la única conversión que el JLS clasifica aparte, como **"widening and narrowing"** (sección 5.1.4): primero se ensancha `byte` a `int`, después se estrecha `int` a `char`. Y requiere cast.

**Por qué importa esto más allá del dato curioso.** No es pedantería de especificación. Es un caso concreto de algo que vas a encontrar constantemente: los tutoriales optimizan para que memorices una regla simple, y la regla simple falla justo en el caso raro que te va a costar dos horas de debugging. La cadena única es una simplificación pedagógica razonable hasta el día que escribís `short s = miChar;` y no entendés por qué no compila.

> **Dos errores más en las mismas fuentes.** Simplilearn llama a widening *"Casting Down"* y a narrowing *"Casting Up"* — exactamente al revés de la convención universal, donde *up* significa ir hacia el supertipo. Además da como sintaxis del narrowing `larger_data_type variable_name = smaller_data_type_variable;`, que describe justamente lo contrario de un narrowing. Y las cuatro fuentes presentan `Integer.parseInt()` y `String.valueOf()` como si fueran type casting, cuando son llamadas a métodos de librería sin relación con el sistema de conversiones del lenguaje (ver [sección 25](#25-lo-que-parece-casting-y-no-lo-es)).

## 7. Widening que sí pierde datos

Este es probablemente el contenido más valioso de toda la Parte II, y no aparece en ninguna de las cuatro fuentes.

La narrativa estándar dice: *"widening es seguro y automático, narrowing es peligroso y manual"*. La primera mitad es falsa. El JLS lo dice de forma explícita en la sección 5.1.2: **tres widening conversions pueden perder información**.

| Conversión | Por qué pierde |
|---|---|
| `int` → `float` | `int` tiene 32 bits de precisión entera; `float` solo 24 bits de mantisa |
| `long` → `float` | `long` tiene 64 bits; `float` solo 24 |
| `long` → `double` | `long` tiene 64 bits; `double` solo 53 de mantisa |

La causa es que los tipos de punto flotante ganan **rango** a costa de **precisión**. Un `float` llega hasta 3,4 × 10³⁸, muchísimo más lejos que un `int` — pero no puede representar todos los enteros por el camino. Solo puede representar exactamente los enteros hasta 2²⁴ = 16.777.216.

Verificado en JDK 25:

```java
int original = 16777217;        // 2^24 + 1, el primer int que un float no puede representar
float comoFloat = original;     // widening: automática, sin aviso, sin cast
int devuelta = (int) comoFloat;

System.out.println(devuelta);                              // 16777216   ← se perdió 1
System.out.println(original == (int) (float) original);    // false
```

Ese `false` es demoledor. Java aceptó la conversión sin una sola advertencia, y el número cambió.

Con `long` → `double` el umbral es 2⁵³:

```java
long original = 9007199254740993L;   // 2^53 + 1
double comoDouble = original;        // widening automática
System.out.println((long) comoDouble);   // 9007199254740992   ← otra vez, se perdió 1
```

Y con `long` → `float` el desastre es mucho antes:

```java
System.out.println((float) 123456789123L);   // 1.2345679E11
```

De doce dígitos significativos quedan ocho.

**Consecuencia práctica.** Si tu sistema maneja identificadores numéricos grandes — IDs de base de datos, timestamps en nanosegundos, números de cuenta, importes en centavos — **nunca los pases por `float` ni por `double`**, ni siquiera "temporalmente" para un cálculo intermedio. El error no se manifiesta como excepción sino como dos registros distintos que de pronto tienen el mismo identificador.

> **La regla revisada, que sí es correcta:** widening nunca **desborda** (el valor nunca da la vuelta ni cambia de signo) y nunca lanza excepción. Pero puede **redondear**. "No hace falta cast" significa "el compilador no te va a molestar", no "no vas a perder nada".

## 8. Narrowing: la conversión manual

Una **narrowing primitive conversion** lleva un valor a un tipo de **menor capacidad**. Java exige que la pidas explícitamente con el operador cast:

```java
double decimal = 9.78;
int entero = (int) decimal;   // cast obligatorio

System.out.println(decimal);  // 9.78
System.out.println(entero);   // 9
```

Sin el cast, el compilador rechaza el código con un mensaje muy característico:

```
error: incompatible types: possible lossy conversion from double to int
```

Ese `possible lossy conversion` es el error de tipos más frecuente de Java. Cuando lo veas, el compilador te está diciendo: *"esto puede perder datos; si estás de acuerdo, escribí el cast"*.

El JLS define **22** narrowing primitive conversions:

| Desde | Hacia |
|---|---|
| `short` | `byte`, `char` |
| `char` | `byte`, `short` |
| `int` | `byte`, `short`, `char` |
| `long` | `byte`, `short`, `char`, `int` |
| `float` | `byte`, `short`, `char`, `int`, `long` |
| `double` | `byte`, `short`, `char`, `int`, `long`, `float` |

### 8.1. Entre enteros: se cortan los bits

Cuando estrechás entre tipos enteros, Java **no valida nada, no redondea y no lanza**. Simplemente **descarta los bits que sobran** y reinterpreta lo que queda.

```java
System.out.println((byte) 128);            // -128
System.out.println((byte) 300);            // 44
System.out.println((byte) 1000);           // -24
System.out.println((short) 70000);         // 4464
System.out.println((int) 4294967301L);     // 5
System.out.println((int) Long.MAX_VALUE);  // -1
```

Ninguno de esos resultados es un error del lenguaje: son exactamente lo que la especificación manda. Veamos el primero en binario, porque entender el mecanismo evita tener que memorizar los casos.

`128` como `int` ocupa 32 bits:

```
00000000 00000000 00000000 10000000
```

Un `byte` tiene 8 bits. La conversión conserva los **8 bits menos significativos** y tira el resto:

```
                            10000000
```

Ahora hay que leer esos 8 bits *como un `byte`*, y `byte` tiene signo en **complemento a dos**: el bit más alto es el bit de signo. Ese `1` inicial significa negativo, y el patrón `10000000` en complemento a dos vale exactamente **−128**.

El caso de `(int) Long.MAX_VALUE` es el más elegante: `Long.MAX_VALUE` es `0x7FFFFFFFFFFFFFFF`, sus 32 bits bajos son `0xFFFFFFFF`, y eso leído como `int` con signo es `-1`. El número más grande que Java puede representar se convierte en `-1`.

> **Truco mental:** el narrowing entre enteros equivale a `valor mod 2ⁿ`, reinterpretado con signo. Para `byte`, `n = 8`, así que es `valor mod 256` ajustado al rango −128…127. La regla [NUM12-J del SEI CERT](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/numeric-types-and-operations-num/num12-j) sugiere escribir `(byte)(i % 0x100)` cuando el truncamiento es intencional, precisamente para dejar por escrito que sabés lo que estás haciendo.

### 8.2. De punto flotante a entero: las reglas exactas

Acá el JLS es sorprendentemente específico, y las reglas no son las que uno adivinaría. Convertir un `float` o `double` a un tipo entero es un proceso de **dos pasos**.

**Paso 1** — se convierte a `long` (si el destino es `long`) o a `int` (en cualquier otro caso), con estas reglas:

| Valor de origen | Resultado |
|---|---|
| `NaN` | **`0`** |
| Se redondea hacia cero (trunca) y cabe | ese valor |
| Muy grande / `+Infinity` | `Integer.MAX_VALUE` o `Long.MAX_VALUE` |
| Muy pequeño / `-Infinity` | `Integer.MIN_VALUE` o `Long.MIN_VALUE` |

**Paso 2** — si el destino es `byte`, `short` o `char`, se aplica el truncamiento de bits de la sección anterior sobre el resultado del paso 1.

Verificado:

```java
System.out.println((int) 9.99);          // 9
System.out.println((int) -9.99);         // -9    ← hacia cero, no hacia abajo
System.out.println((int) (0.0 / 0.0));   // 0            NaN
System.out.println((int) (1.0 / 0.0));   // 2147483647   +Infinity → MAX_VALUE
System.out.println((int) (-1.0 / 0.0));  // -2147483648  -Infinity → MIN_VALUE
System.out.println((int) 1e20);          // 2147483647   satura, no da la vuelta
System.out.println((int) -1e20);         // -2147483648
```

Dos cosas merecen atención especial.

**El paso 1 satura, pero el paso 2 no.** Esto produce resultados que parecen aleatorios:

```java
System.out.println((short) Float.MAX_VALUE);  // -1
System.out.println((short) Float.MIN_VALUE);  // 0
```

`Float.MAX_VALUE` (≈3,4 × 10³⁸) satura en el paso 1 a `Integer.MAX_VALUE` = `0x7FFFFFFF`. El paso 2 se queda con los 16 bits bajos, `0xFFFF`, que como `short` vale **−1**. El número positivo más grande de un `float` se convierte en `-1`.

`Float.MIN_VALUE` es una trampa de nombre: **no es el más negativo**, es el positivo más pequeño (≈1,4 × 10⁻⁴⁵). Truncado hacia cero da `0`. Si querés el más negativo de un `float`, es `-Float.MAX_VALUE`. Esta confusión de nomenclatura afecta a `Float.MIN_VALUE` y `Double.MIN_VALUE`, pero no a `Integer.MIN_VALUE`, que sí es el más negativo.

**`NaN` → `0` es peligrosísimo en código de negocio.** `NaN` significa "el cálculo no tiene resultado válido" (típicamente una división `0.0/0.0`). Convertirlo a `int` produce un `0` perfectamente inocente que se propaga por el sistema como si fuera un dato legítimo:

```java
double promedio = sumaTotal / cantidadElementos;  // si cantidad es 0.0 → NaN
int mostrar = (int) promedio;                     // 0
// El informe dice "promedio: 0" en lugar de fallar. Nadie se entera.
```

### 8.3. `double` → `float`

```java
System.out.println((float) Double.MAX_VALUE);  // Infinity
System.out.println((float) Double.MIN_VALUE);  // 0.0
```

Aquí no hay truncamiento de bits: se aplica redondeo IEEE 754 al valor más cercano representable. Si el valor excede el rango de `float`, el resultado es `Infinity`; si es demasiado pequeño, `0.0`. **Nunca lanza excepción**, así que un `double` perfectamente válido puede convertirse en infinito sin ningún aviso.

## 9. Truncar no es redondear

Esta es, medida por volumen de preguntas en foros, **la confusión número uno del tema**. El cast a `int` no redondea: trunca hacia cero.

```java
System.out.println((int) 3.9);        // 3     no 4
System.out.println((int) -3.9);       // -3    no -4
System.out.println(Math.round(3.9));  // 4
System.out.println(Math.round(-3.9)); // -4
System.out.println(Math.floor(-3.9)); // -4.0
```

Prestá atención al caso negativo, que es donde la gente se equivoca incluso sabiendo que "trunca". **Truncar hacia cero no es lo mismo que redondear hacia abajo.** Para `-3.9`:

- `(int)` trunca **hacia cero** → `-3` (el resultado es *mayor* que el original)
- `Math.floor()` redondea **hacia abajo** → `-4.0`

Para números positivos ambos coinciden; para negativos difieren siempre. Si tu código calcula edades, cantidades o índices y alguna vez recibe un negativo, esta diferencia produce un off-by-one que solo aparece con datos negativos.

Y una tercera trampa, dentro de `Math.round`:

```java
System.out.println(Math.round(2.5));   // 3
System.out.println(Math.round(-2.5));  // -2    ← no es -3
```

`Math.round(x)` está definido como `floor(x + 0.5)`, lo que en los empates redondea **hacia +infinito**, no "hacia el más lejano de cero". Por eso `2.5 → 3` pero `-2.5 → -2`.

**Tabla de decisión:**

| Quiero… | Uso | `3.7` | `-3.7` |
|---|---|---|---|
| Descartar decimales | `(int) x` | `3` | `-3` |
| Al entero más cercano | `Math.round(x)` | `4` | `-4` |
| Siempre hacia abajo | `(int) Math.floor(x)` | `3` | `-4` |
| Siempre hacia arriba | `(int) Math.ceil(x)` | `4` | `-3` |
| Control fino (dinero) | `BigDecimal.setScale(n, RoundingMode…)` | — | — |

`Math.round(double)` devuelve `long`, y `Math.round(float)` devuelve `int`. Si necesitás un `int` a partir de un `double` vas a tener que castear el resultado: `int r = (int) Math.round(x);`.

### 9.1. El problema de fondo: los flotantes no son exactos

Antes de convertir, conviene recordar que el valor de partida ya puede estar mal:

```java
System.out.println(0.1 + 0.2);          // 0.30000000000000004
System.out.println(0.29 * 100);         // 28.999999999999996
System.out.println((int) (0.29 * 100)); // 28    ← esperabas 29
System.out.println(1.1 * 3);            // 3.3000000000000003
```

`double` y `float` son binarios: no pueden representar exactamente `0.1`, `0.29` ni ningún decimal cuya expansión binaria sea infinita, igual que en decimal no podés escribir 1/3 exacto. Cuando después truncás con `(int)`, ese error microscópico se convierte en un error de una unidad entera.

Es exactamente el bug de "el precio 0,29 € se convirtió en 28 céntimos". Y notá que no es predecible a ojo: `(int) (0.7 * 10)` sí da `7`, porque el redondeo IEEE 754 en ese caso particular cae del lado bueno. **La regla no es "a veces falla": es "no podés saberlo sin ejecutarlo".**

> **Regla dura para producción: no representes dinero con `double` ni con `float`.** Usá `BigDecimal` (construido desde `String`, nunca desde un literal `double`) o guardá enteros de centavos en un `long`. Este consejo aparece en cada discusión de foros sobre dinero en Java desde hace veinte años, y sigue siendo el error más caro que comete un junior.

## 10. `char`, el tipo raro

`char` merece su propia sección porque es simultáneamente un carácter y un número, y esa doble naturaleza produce comportamientos que sorprenden.

En Java, `char` es un entero **sin signo de 16 bits** que guarda una unidad de código UTF-16. Participa plenamente en la aritmética:

```java
char letra = 'A';
System.out.println((int) letra);        // 65
System.out.println((char) (letra + 1)); // B
System.out.println('a' + 1);            // 98    ← imprime número, no letra
System.out.println('a' + 'b');          // 195   ← suma de códigos
```

Fijate en las dos últimas líneas: **el resultado de sumar `char`s es un `int`**, no un `char` (es la promoción numérica de la [sección 11](#11-promoción-numérica-la-trampa-invisible)). Si querés volver a tener un carácter, necesitás el cast explícito de vuelta.

### 10.1. El truco de `- '0'` y cuándo falla

Convertir el carácter `'5'` al número `5` es una operación cotidiana, y hay un idiom clásico:

```java
System.out.println((int) '5');        // 53   ← el código Unicode, no el dígito
System.out.println('5' - '0');        // 5    ← el dígito
```

Funciona porque los dígitos ASCII son consecutivos: `'0'` es 48, `'1'` es 49, …, `'9'` es 57. Restar `'0'` cancela el desplazamiento.

Pero falla silenciosamente cuando el carácter no es un dígito:

```java
System.out.println('A' - '0');                        // 17   ← basura, no error
System.out.println(Character.getNumericValue('A'));   // 10   ← 'A' como dígito hexadecimal
System.out.println(Character.getNumericValue('5'));   // 5
```

Ninguno de los dos lanza excepción. `- '0'` da `17`, que no significa nada; `Character.getNumericValue` da `10`, porque interpreta `'A'` como dígito hexadecimal. Y si el carácter viene de una entrada internacional, la aritmética ASCII se rompe del todo: los dígitos árabes-índicos ocupan los code points 1632–1641, así que restarles `'0'` (48) da números de cuatro cifras.

**Criterio:** usá `- '0'` solo cuando ya validaste con `Character.isDigit()` que el carácter es un dígito ASCII. Si el origen es texto de usuario sin validar, `Character.getNumericValue()` es más legible y menos frágil, y para parsear una cadena entera lo correcto es `Integer.parseInt`.

### 10.2. `byte` → `char`: el caso que corrompe datos binarios

Recordá que `byte` tiene signo (−128…127) y `char` no (0…65535). Cuando convertís un `byte` negativo a `char`, primero se ensancha a `int` **propagando el signo**, y el resultado es un número enorme:

```java
byte b = -1;
System.out.println((int) (char) b);         // 65535
System.out.println((int) (byte) (char) -1); // -1   ← el viaje de vuelta sí funciona
```

Este es el origen de una familia entera de bugs al procesar streams binarios. Si leés bytes de un archivo o un socket y los tratás como caracteres, todo byte con el bit alto activo (valor ≥ 128, es decir, negativo como `byte`) se convierte en un carácter de 5 dígitos. El idiom correcto para leer un `byte` como valor sin signo 0…255 es:

```java
int sinSigno = b & 0xFF;   // enmascara los bits de signo propagados
```

Ese `& 0xFF` que aparece en todo el código de manipulación binaria de Java está ahí exactamente por esto.

## 11. Promoción numérica: la trampa invisible

Antes de ejecutar cualquier operación aritmética, Java **promociona** sus operandos. Es una conversión que ocurre sin que la escribas y sin que la veas.

**Promoción unaria** (para `-x`, `~x`, índices de array): si el operando es `byte`, `short` o `char`, se convierte a `int`.

**Promoción binaria** (para `+`, `-`, `*`, `/`, `%`, comparaciones), en este orden:

1. Si algún operando es `double` → los dos a `double`
2. Si no, si alguno es `float` → los dos a `float`
3. Si no, si alguno es `long` → los dos a `long`
4. Si no → **los dos a `int`**

El paso 4 es el que sorprende a todo el mundo: **no existe la aritmética de `byte`, `short` ni `char`**. La JVM no tiene instrucciones para sumar bytes. Todo se calcula en `int` como mínimo.

### 11.1. `byte + byte` no cabe en un `byte`

```java
byte a = 10, b = 20;
byte suma = a + b;          // ✘ error: possible lossy conversion from int to byte
byte suma = (byte)(a + b);  // ✔ 30
```

El mensaje de error desconcierta —"¿de dónde salió un `int`?"— hasta que sabés que `a + b` se promocionó a `int` antes de sumar. Por eso el resultado es `int` y asignarlo a `byte` es un narrowing.

Y con el cast, cuidado: el cast no arregla el desbordamiento, solo silencia al compilador.

```java
byte c = 100, d = 100;
System.out.println((byte)(c + d));  // -56    (200 no cabe en byte)
```

### 11.2. La división entera: el bug clásico

Este es el error de casting más frecuente del mundo, y aparece en la propia página de W3Schools como "ejemplo de la vida real" — bien resuelto allí, pero sin explicar por qué el orden importa.

```java
int aciertos = 423, total = 500;

System.out.println((double) (aciertos / total) * 100);  // 0.0    ✘
System.out.println((double) aciertos / total * 100);    // 84.6   ✔
```

En la primera línea, `aciertos / total` se evalúa **primero**, con dos `int`: la división entera de 423 entre 500 da `0`, y castear `0` a `double` da `0.0`. El cast llegó tarde.

En la segunda, el cast se aplica solo a `aciertos`, así que la división ya es `double / int` → promoción binaria → `double / double` = `0.846`.

> **La regla:** el cast tiene **mayor precedencia** que los operadores aritméticos, así que `(double) a / b` castea solo `a`. Para forzar aritmética decimal hay que convertir **un operando antes de operar**, no el resultado después.

Variantes equivalentes, por si preferís evitar el cast:

```java
double pct = aciertos * 100.0 / total;      // el literal 100.0 fuerza la promoción
double pct = (double) aciertos / total * 100;
```

### 11.3. Overflow antes de asignar

La misma lógica produce un bug más grave, porque no da `0` sino un número que parece plausible:

```java
int dias = 100_000;
long ms = dias * 24 * 60 * 60 * 1000;          // -1474199552   ✘
long ok = (long) dias * 24 * 60 * 60 * 1000;   // 8640000000000 ✔
```

La primera multiplicación se hace **entera en `int`**, desborda en silencio, y recién entonces el resultado (ya corrupto y negativo) se ensancha a `long`. Declarar la variable como `long` no ayuda: **el tipo del destino no influye en cómo se evalúa la expresión**.

En la segunda, castear el primer factor a `long` hace que toda la cadena de multiplicaciones se evalúe en `long`.

Este patrón aparece constantemente en cálculos de tiempo, tamaños de archivo y cantidades de bytes. Regla: **si el resultado va a un `long`, empezá la expresión en `long`.**

## 12. Los tres casos donde el compilador afloja

Java exige cast para todo narrowing… salvo en tres situaciones que conviene conocer porque rompen la regla que acabás de aprender.

### 12.1. Constantes en contexto de asignación

```java
byte b = 42;        // ✔ compila sin cast
byte c = 200;       // ✘ error: possible lossy conversion from int to byte
```

El literal `42` es un `int`. Debería requerir cast… pero el JLS hace una excepción: si el valor es una **expresión constante** conocida en compilación y **cabe** en el tipo destino, la conversión se permite implícitamente. Por eso `42` pasa y `200` no.

Lo mismo con variables `final` inicializadas con constantes:

```java
final int x = 42;
byte b = x;         // ✔ compila: x es constante en tiempo de compilación

final int y = calcular();
byte c = y;         // ✘ NO compila: final, pero no constante

int z = 42;
byte d = z;         // ✘ NO compila: cabe, pero no es final
```

La diferencia entre `y` y `x` es sutil e importante: `final` no significa "constante en compilación". Solo lo es si el compilador puede calcular su valor sin ejecutar nada.

### 12.2. Compound assignment: el cast escondido

Este es un rincón oscuro del lenguaje que produce bugs muy difíciles de ver.

```java
byte b = 10;
b = b + 300;    // ✘ NO compila
b += 300;       // ✔ compila... y da 54
```

Las dos líneas parecen equivalentes. No lo son. El JLS (sección 15.26.2) define `E1 op= E2` como equivalente a:

```java
E1 = (T)(E1 op E2)     // donde T es el tipo de E1
```

Es decir, **todo operador de asignación compuesta lleva un cast implícito incorporado**. Nunca vas a ver un error de compilación en un `+=`, `-=`, `*=` o `/=`, porque el cast siempre está ahí, invisible.

Verificado:

```java
byte b = 10;  b += 300;   // 54     (310 truncado a byte)
int i = 5;    i += 1.9;   // 6      (¡un double sumado a un int sin error!)
char c = 'a'; c += 1;     // 'b'
```

La línea `i += 1.9` es el caso más inquietante: `i = i + 1.9` no compilaría nunca (`double` a `int`), pero `i += 1.9` compila, ejecuta, y trunca el `6.9` a `6`. La herramienta [CodeQL tiene una consulta dedicada](https://codeql.github.com/codeql-query-help/java/java-implicit-cast-in-compound-assignment/) a detectar exactamente este patrón, lo que da una idea de cuántos bugs reales ha causado.

### 12.3. Los operadores `++` y `--`

Por la misma razón, el incremento sobre tipos pequeños funciona sin cast:

```java
byte b = 127;
b++;                 // ✔ compila, y ahora b vale -128
```

`b++` es `b = (byte)(b + 1)`. El desbordamiento es silencioso.

## 13. Cómo hacer un narrowing seguro

Si el narrowing entre primitivos nunca lanza excepción, la seguridad tiene que ponerla tu código. Hay tres estrategias, en orden de preferencia.

### 13.1. Usar los métodos `Exact` de `Math` (recomendado)

Desde Java 8, `Math.toIntExact(long)` hace la conversión **verificada**:

```java
long id = 3_000_000_000L;

int malo  = (int) id;              // -1294967296   silencioso
int bueno = Math.toIntExact(id);   // ArithmeticException: integer overflow
```

La familia completa incluye `Math.addExact`, `subtractExact`, `multiplyExact`, `incrementExact`, `negateExact` y `toIntExact`. Todos lanzan `ArithmeticException` en lugar de dar la vuelta en silencio.

> **Cuándo usarlo:** siempre que el valor venga de fuera de tu control — base de datos, API, entrada de usuario, archivo. Un fallo ruidoso en el borde del sistema es infinitamente más barato que un dato corrupto viajando hacia adentro.

### 13.2. Validar el rango explícitamente

Es la solución que recomienda la regla NUM12-J del SEI CERT, útil cuando querés un mensaje de error propio o un comportamiento distinto de la excepción:

```java
if (valor < Byte.MIN_VALUE || valor > Byte.MAX_VALUE) {
    throw new IllegalArgumentException("Valor fuera del rango de byte: " + valor);
}
byte b = (byte) valor;
```

Cada tipo tiene sus constantes `MIN_VALUE` y `MAX_VALUE`. Recordá la trampa: en `Float` y `Double`, `MIN_VALUE` es el **positivo más pequeño**, no el más negativo. Para flotantes, la comparación correcta es contra `-Float.MAX_VALUE`.

### 13.3. Documentar el truncamiento intencional

A veces el truncamiento *es* lo que querés (checksums, hashing, empaquetado de bits, protocolos binarios). En ese caso, dejalo explícito para el que lea el código dentro de un año:

```java
// Truncamiento intencional: el protocolo define el campo como 8 bits sin signo
byte campo = (byte) (valor & 0xFF);
```

---

# Parte III — Conversiones entre referencias

Cambiamos de mundo. Todo lo anterior manipulaba **valores**: se reescribían bits, se perdían decimales. Lo que viene manipula **referencias**, y el principio rector es opuesto:

> El casting de referencias **nunca modifica el objeto**. Solo cambia el tipo con el que el compilador lo mira, y a veces añade una verificación en runtime.

## 14. Tipo estático y tipo dinámico

Toda expresión en Java tiene dos tipos, y confundirlos es la raíz de casi todos los malentendidos de esta parte.

- **Tipo estático** (o declarado): el que escribiste en el código. Lo conoce el compilador. Determina **qué métodos podés llamar**.
- **Tipo dinámico** (o de runtime): la clase real del objeto que hay en el heap. Lo conoce la JVM. Determina **qué implementación se ejecuta**.

```java
Animal a = new Perro();
```

El tipo estático de `a` es `Animal`. El tipo dinámico del objeto es `Perro`. Y cada uno gobierna una cosa distinta:

```java
class Animal { String hablar() { return "..."; } }
class Perro extends Animal {
    String hablar() { return "guau"; }
    String buscarPelota() { return "corriendo"; }
}

Animal a = new Perro();
System.out.println(a.hablar());        // "guau"  ← tipo DINÁMICO decide la implementación
System.out.println(a.buscarPelota());  // ✘ NO COMPILA ← tipo ESTÁTICO decide qué es visible
System.out.println(a.getClass());      // class Perro
```

Este par de líneas es el tema entero en miniatura. El objeto **es** un `Perro` y se comporta como tal (`hablar()` devuelve "guau" por despacho dinámico). Pero el compilador solo ve un `Animal`, y `Animal` no tiene `buscarPelota()`, así que rechaza la llamada.

El casting de referencias es el mecanismo para **ajustar el tipo estático** y así cambiar qué es visible. Nunca toca el tipo dinámico.

## 15. Upcasting

**Upcasting** es convertir una referencia hacia un supertipo: de subclase a superclase, o de clase a interfaz. En el JLS se llama *widening reference conversion*.

Es **implícito, siempre seguro y no requiere cast**, porque todo `Perro` es por definición un `Animal`:

```java
Perro p = new Perro();

Animal a = p;              // upcast implícito
Object o = p;              // hasta Object, también implícito
Animal b = (Animal) p;     // el cast explícito es legal, pero redundante
```

Que sea seguro tiene una consecuencia que se puede *demostrar*: el upcast **no genera ninguna instrucción de bytecode**. Compilando este método:

```java
A upcast(B b) { return b; }   // B extends A
```

`javap -c` produce:

```
A upcast(B);
  Code:
     0: aload_1     // carga la referencia
     1: areturn     // la devuelve
```

Cero instrucciones de conversión. El upcast **no existe en tiempo de ejecución**: es puramente una anotación para el compilador. La referencia que entra es bit por bit la misma que sale.

```java
Perro p = new Perro();
Animal a = p;
System.out.println(a == p);       // true   ← es el mismo objeto
System.out.println(a.getClass()); // class Perro
```

### 15.1. Para qué sirve perder capacidades

A primera vista el upcasting parece contraproducente: renunciás a métodos a cambio de nada. Pero es la base del polimorfismo y de todo el diseño orientado a interfaces:

```java
// Sin upcasting: un método por cada tipo
void alimentarPerro(Perro p) { ... }
void alimentarGato(Gato g) { ... }

// Con upcasting: uno solo sirve para toda la jerarquía, presente y futura
void alimentar(Animal a) { a.comer(); }
```

Y es lo que permite las colecciones heterogéneas:

```java
List<Animal> refugio = List.of(new Perro(), new Gato(), new Loro());
for (Animal a : refugio) {
    System.out.println(a.hablar());   // cada uno responde según su tipo dinámico
}
```

El upcasting es **generalizar**: tratar cosas distintas de manera uniforme por lo que tienen en común.

## 16. Downcasting

**Downcasting** es lo contrario: convertir una referencia hacia un subtipo. En el JLS, *narrowing reference conversion*.

Requiere **cast explícito** y **puede fallar en runtime**, porque no todo `Animal` es un `Perro`:

```java
Animal a = new Perro();
Perro p = (Perro) a;                    // ✔ funciona: el objeto realmente es un Perro
System.out.println(p.buscarPelota());   // "corriendo" — ya es visible

Animal gato = new Gato();
Perro malo = (Perro) gato;              // ✘ ClassCastException en runtime
```

El mensaje exacto que produce la JVM:

```
java.lang.ClassCastException: class Gato cannot be cast to class Perro
  (Gato and Perro are in unnamed module of loader 'app')
```

> **La regla de oro:** solo podés hacer downcast a un tipo que el objeto **realmente tiene**. El cast no transforma un `Gato` en un `Perro`, igual que ponerle una etiqueta de "perro" a un gato no lo hace ladrar. El cast solo **afirma** algo sobre el objeto, y la JVM verifica si esa afirmación es cierta.

### 16.1. Downcast sobre `null`

Un caso que confunde en entrevistas: castear `null` nunca falla.

```java
Object nulo = null;
String s = (String) nulo;    // ✔ no lanza nada; s queda en null
```

`null` es asignable a cualquier tipo de referencia y el `checkcast` de la JVM lo deja pasar por definición. Consecuencia práctica: un downcast que "funcionó" no garantiza que tengas un objeto — puede que tengas `null` y el `NullPointerException` te llegue tres líneas después.

## 17. Qué pasa realmente en runtime: `checkcast`

Ya vimos que el upcast no genera bytecode. El downcast sí:

```java
B downcast(A a) { return (B) a; }
```

```
B downcast(A);
  Code:
     0: aload_1
     1: checkcast     #7    // class B
     4: areturn
```

Esa instrucción `checkcast` es todo el mecanismo. Su comportamiento es simple:

1. Si la referencia es `null` → pasa.
2. Si el tipo dinámico del objeto es compatible con el tipo indicado → pasa, sin modificar nada.
3. Si no → lanza `ClassCastException`.

Entender esto aclara tres cosas que suelen confundirse:

- **El cast tiene coste en runtime**, aunque es pequeño y la JIT lo optimiza bien cuando el tipo es predecible. Un upcast, en cambio, cuesta exactamente cero.
- **La verificación usa el tipo dinámico**, no el estático. Por eso el compilador puede aceptar un cast que la JVM rechaza.
- **El objeto no se toca.** No hay copia, ni conversión, ni asignación de memoria.

### 17.1. Qué rechaza el compilador

No todos los downcasts llegan a runtime. Si el compilador puede **demostrar** que el cast nunca puede tener éxito, es un error de compilación:

```java
String s = "hola";
Integer i = (Integer) s;   // ✘ error: String cannot be converted to Integer
```

`String` e `Integer` son clases *disjuntas*: ninguna hereda de la otra y ambas son `final`, así que no puede existir un objeto que sea las dos cosas. El compilador lo sabe y no deja pasar el código.

Pero basta que exista **una posibilidad teórica** para que compile y falle en runtime:

```java
Object o = "hola";
Integer i = (Integer) o;   // ✔ compila (Object podría contener un Integer)
                           // ✘ ClassCastException en runtime
```

La regla del JLS es matizada, y hay un caso que sorprende:

```java
void f(Object o)  { Runnable r = (Runnable) o; }  // ✔ compila
void g(String s)  { Runnable r = (Runnable) s; }  // ✘ NO compila
```

Castear a una **interfaz** desde una clase no `final` siempre compila: aunque esa clase no implemente la interfaz, una subclase suya podría hacerlo. Pero `String` es `final` — no puede tener subclases — y no implementa `Runnable`, así que el compilador puede descartarlo con certeza.

> **El principio general:** el compilador prohíbe lo **imposible**; la JVM verifica lo **improbable**.

## 18. `instanceof` y pattern matching

Si el downcast puede fallar, la solución obvia es preguntar antes.

### 18.1. La forma clásica

```java
if (animal instanceof Perro) {
    Perro p = (Perro) animal;
    p.buscarPelota();
}
```

`instanceof` devuelve `true` si el objeto es del tipo indicado o de un subtipo. Y **devuelve `false` para `null`**, siempre, sin lanzar nada:

```java
Object nulo = null;
System.out.println(nulo instanceof String);   // false
```

Ese detalle es útil: `if (x instanceof Foo f)` cubre a la vez la comprobación de tipo y la de nulidad.

El problema de esta forma clásica es que el tipo aparece dos veces, y el propio Oracle señala el riesgo: *"probar el tipo de un objeto con instanceof y luego asignarlo con un cast puede introducir errores, porque podrías cambiar el tipo de uno y olvidarte del otro"*.

### 18.2. Pattern matching (Java 16+)

Desde Java 16, `instanceof` puede declarar la variable directamente:

```java
if (animal instanceof Perro p) {
    p.buscarPelota();     // 'p' ya está disponible y tipada
}
```

Un solo tipo, sin cast, sin variable intermedia. Esta es la forma que deberías escribir hoy en cualquier proyecto Java 17+.

Las reglas de scope de la *pattern variable* son inteligentes: existe exactamente donde el compilador puede garantizar que el test fue verdadero.

```java
// Con && funciona: el corto-circuito garantiza que 'p' existe
if (animal instanceof Perro p && p.getEdad() > 5) { ... }

// Con || NO compila: se podría llegar al segundo operando con el test en false
if (animal instanceof Perro p || p.getEdad() > 5) { ... }   // ✘

// Con negación y early return, funciona al revés (muy útil)
if (!(animal instanceof Perro p)) {
    return "no es un perro";
}
return p.buscarPelota();    // ✔ 'p' está en scope: solo se llega aquí si el test fue true
```

Ese último patrón —negar, salir temprano, y usar la variable después— es idiomático en Java moderno y elimina un nivel de anidamiento.

### 18.3. Switch patterns (Java 21+)

Cuando hay varios tipos posibles, la cadena de `if/else if` se sustituye por un `switch`:

```java
sealed interface Figura permits Circulo, Rect, Tri {}
record Circulo(double r) implements Figura {}
record Rect(double w, double h) implements Figura {}
record Tri(double b, double h) implements Figura {}

static double area(Figura f) {
    return switch (f) {
        case Circulo c -> Math.PI * c.r() * c.r();
        case Rect r    -> r.w() * r.h();
        case Tri t     -> t.b() * t.h() / 2;
    };
}
```

Aquí no hay ni un cast, ni un `instanceof`, ni un `default`. Y hay algo mejor: como `Figura` es `sealed` (la lista de implementaciones es cerrada y conocida), **el compilador verifica que cubriste todos los casos**. Si mañana alguien añade `record Hexagono(...) implements Figura`, este `switch` deja de compilar hasta que lo manejes.

Eso convierte un posible error de runtime en un error de compilación, que es la mejor transformación disponible en ingeniería de software. Si venís de leer sobre el patrón Visitor, esto ofrece la misma garantía de exhaustividad con una fracción del código.

### 18.4. Cuándo el downcast indica un problema de diseño

Hay consenso amplio en la comunidad —desde el artículo clásico [*Prefer polymorphism over instanceof and downcasting*](https://www.artima.com/interfacedesign/PreferPoly.html) hasta las discusiones actuales— en que una cadena de `instanceof` suele ser un **code smell**.

```java
// ✘ Anti-patrón: el llamador decide el comportamiento según el tipo
double area(Object figura) {
    if (figura instanceof Circulo)   return 3.14 * ((Circulo) figura).getR() * ...;
    else if (figura instanceof Rect) return ((Rect) figura).getW() * ...;
    else if (figura instanceof Tri)  return ...;
    throw new IllegalArgumentException("desconocida");
}
```

El problema no es estético: cada tipo nuevo obliga a encontrar y modificar **todas** las cadenas de `instanceof` del proyecto, y si te olvidás de una, el fallo es en runtime.

```java
// ✔ Polimorfismo: cada tipo sabe calcular su área
interface Figura { double area(); }
record Circulo(double r) implements Figura {
    public double area() { return Math.PI * r * r; }
}
// Añadir un tipo nuevo no toca ninguna línea existente.
```

**Cuándo el downcast sí es legítimo:**

- Implementar `equals(Object)` — la firma la impone `Object`, no tenés alternativa.
- Interoperar con APIs anteriores a los genéricos que devuelven `Object`.
- Deserializar datos externos (JSON, `Map<String, Object>`, `ResultSet`).
- Recorrer estructuras cerradas y conocidas — y ahí, en Java 21+, usá `sealed` + `switch`, no `instanceof` suelto.

## 19. Arrays: covarianza y `ArrayStoreException`

Los arrays de Java tienen una propiedad que hoy se considera un error de diseño histórico: son **covariantes**. Si `Perro` es subtipo de `Animal`, entonces `Perro[]` es subtipo de `Animal[]`.

Suena razonable y abre un agujero en el sistema de tipos:

```java
Object[] arr = new String[3];   // ✔ compila: covarianza
arr[0] = 42;                    // ✘ ArrayStoreException: java.lang.Integer
```

El compilador acepta el código porque el tipo estático de `arr` es `Object[]` y un `Integer` es un `Object`. Pero en runtime el array **es** un `String[]`, y la JVM verifica el tipo en **cada escritura**. De ahí `ArrayStoreException`.

Los genéricos aprendieron la lección y son **invariantes**: `List<String>` **no** es subtipo de `List<Object>`.

```java
List<String> ls = new ArrayList<>();
List<Object> lo = ls;   // ✘ error: List<String> cannot be converted to List<Object>
```

Es más restrictivo, pero el error llega en compilación en vez de en runtime.

### 19.1. El clásico de `toArray()`

```java
List<String> lista = new ArrayList<>(List.of("a", "b"));

String[] bien = lista.toArray(new String[0]);     // ✔ ["a", "b"]

Object[] generico = lista.toArray();              // devuelve Object[], no String[]
String[] mal = (String[]) generico;               // ✘ ClassCastException
```

El mensaje real es característico y vale la pena reconocerlo:

```
class [Ljava.lang.Object; cannot be cast to class [Ljava.lang.String;
```

Esa notación `[L...;` es la forma interna de la JVM de nombrar arrays (`[` = array, `L...;` = tipo referencia). Cuando la veas en un stack trace, estás ante un problema de tipos de array.

La causa: `toArray()` sin argumentos crea internamente un `new Object[n]`. Aunque todos sus elementos sean `String`, el array **es** de tipo `Object[]`, y castearlo falla. La solución es `toArray(new String[0])`, que usa el array pasado como plantilla de tipo. Desde Java 11 también existe `toArray(String[]::new)`.

## 20. Casting reflexivo: `Class.cast`

Para casos genéricos o dinámicos, `java.lang.Class` ofrece equivalentes en forma de método:

```java
Object o = "texto";

System.out.println(String.class.isInstance(o));  // true      ≡ o instanceof String
System.out.println(String.class.cast(o));        // "texto"   ≡ (String) o
Integer.class.cast(o);                           // ClassCastException
```

El mensaje que produce `Class.cast` es distinto del cast normal, más escueto: `Cannot cast java.lang.String to java.lang.Integer`.

Estos métodos existen porque el cast sintáctico `(T) x` no se puede escribir cuando `T` es un parámetro de tipo genérico — el erasure lo borra. El idiom del **type token** resuelve ese problema:

```java
public <T> T leerConfig(String clave, Class<T> tipo) {
    Object valor = configuracion.get(clave);
    return tipo.cast(valor);    // cast verificado de verdad, no unchecked
}

Integer puerto = leerConfig("puerto", Integer.class);
```

La ventaja sobre `(T) valor` es real: aquí la verificación **sí ocurre**, en el momento correcto, con un `ClassCastException` que apunta al lugar del problema. Con `(T) valor` el compilador emite un *unchecked warning* y el fallo se difiere hasta vaya a saber dónde ([sección 24](#24-unchecked-cast-y-heap-pollution)).

También existe `asSubclass(Class)`, para estrechar un `Class<?>` a un `Class<? extends T>` de forma verificada.

---

# Parte IV — Boxing y unboxing

## 21. Qué son y dónde ocurren

Java tiene ocho primitivos (`int`, `double`, `boolean`…) y, para cada uno, una **clase wrapper** que lo envuelve en un objeto:

| Primitivo | Wrapper | Primitivo | Wrapper |
|---|---|---|---|
| `byte` | `Byte` | `int` | `Integer` |
| `short` | `Short` | `long` | `Long` |
| `char` | `Character` | `float` | `Float` |
| `boolean` | `Boolean` | `double` | `Double` |

Los wrappers existen porque **los genéricos y las colecciones no aceptan primitivos**:

```java
List<int> numeros = new ArrayList<>();       // ✘ no compila
List<Integer> numeros = new ArrayList<>();   // ✔
```

Desde Java 5, el compilador inserta la conversión por vos:

- **Autoboxing**: primitivo → wrapper, automático.
- **Unboxing**: wrapper → primitivo, automático.

```java
Integer objeto = 5;          // autoboxing:  Integer.valueOf(5)
int primitivo = objeto;      // unboxing:    objeto.intValue()

List<Integer> lista = new ArrayList<>();
lista.add(5);                // autoboxing
int primero = lista.get(0);  // unboxing
```

El bytecode lo confirma exactamente:

```
Integer box(int):     invokestatic  Integer.valueOf:(I)Ljava/lang/Integer;
int unbox(Integer):   invokevirtual Integer.intValue:()I
```

Es decir: **el autoboxing es azúcar sintáctico sobre `Integer.valueOf()`**, y ese detalle explica la mitad de los puzzles que vienen ahora.

## 22. Los puzzles del autoboxing

El autoboxing simplificó la sintaxis y, a cambio, introdujo una familia de bugs que siguen apareciendo en producción veinte años después. Estos cubren prácticamente todos los casos.

### 22.1. `NullPointerException` sin ningún `null` visible

Un wrapper puede ser `null`; un primitivo no. Cuando el unboxing recibe `null`, el `intValue()` implícito explota:

```java
Map<String, Integer> stock = new HashMap<>();
stock.put("teclado", 5);

int cantidad = stock.get("mouse");   // ✘ NullPointerException
```

No hay ningún `null` escrito en esa línea. `get("mouse")` devuelve `null`, y asignarlo a un `int` fuerza el unboxing. Desde Java 14, los *helpful NullPointerExceptions* explican qué pasó:

```
Cannot invoke "java.lang.Integer.intValue()" because the return value of
"java.util.Map.get(Object)" is null
```

**Prevención:**

```java
int cantidad = stock.getOrDefault("mouse", 0);   // ✔ explícito

Integer valor = stock.get("mouse");              // ✔ mantener el wrapper
if (valor != null) { ... }
```

**Regla de diseño:** usá primitivos siempre que puedas. Reservá los wrappers para cuando el `null` signifique algo real ("no hay dato", distinto de "el dato es 0"). Un campo `Integer` en una entidad JPA suele ser correcto, porque la columna puede ser `NULL`; una variable local `Integer contador` casi nunca lo es.

### 22.2. `==` con wrappers: el Integer cache

El más famoso de todos, y una pregunta de entrevista clásica:

```java
Integer a = 127, b = 127;
Integer c = 128, d = 128;

System.out.println(a == b);       // true
System.out.println(c == d);       // false   ← mismo código, distinto resultado
System.out.println(c.equals(d));  // true
```

La explicación está en el JLS 5.1.7: el autoboxing llama a `Integer.valueOf()`, y ese método mantiene una **caché de objetos** que la especificación obliga a tener para el rango **−128 … 127**. Dentro de ese rango se devuelve siempre la misma instancia; fuera, se crea una nueva.

Y `==` sobre referencias compara **identidad**, no valor. Con objetos cacheados coinciden por accidente; fuera de la caché, no.

El rango cacheado por especificación:

| Tipo | Valores garantizados en caché |
|---|---|
| `Boolean` | `true`, `false` |
| `Byte` | todos |
| `Character` | `\u0000` … `\u007F` (los 128 primeros) |
| `Short`, `Integer`, `Long` | −128 … 127 |

> El límite superior de la caché de `Integer` es configurable con `-XX:AutoBoxCacheMax=<n>`. Es decir: **el mismo código puede dar `true` o `false` según los flags de la JVM**. Otra razón para no depender jamás de este comportamiento.

**Regla absoluta:** comparar wrappers con `==` es un bug, salvo que estés comprobando identidad a propósito. Usá `equals()`, o mejor, hacé unboxing explícito:

```java
if (a.equals(b))                    { }   // ✔
if (a.intValue() == b.intValue())   { }   // ✔
if (a == b)                         { }   // ✘ funciona por casualidad hasta 127
```

**El matiz que salva:** si **uno** de los operandos es primitivo, `==` sí compara valores, porque el otro sufre unboxing automático:

```java
Integer boxed = 1000;
int primitivo = 1000;
System.out.println(boxed == primitivo);   // true   ← acá sí funciona
```

Esta asimetría es justamente lo que hace tan difícil detectar el bug leyendo código.

### 22.3. El ternario que lanza NPE

```java
Integer valor = null;
Integer resultado = true ? valor : 0;    // ✘ NullPointerException
```

Parece imposible: la rama que se ejecuta es `valor`, que es `null`, y el destino es un `Integer` que acepta `null`. ¿Por qué falla?

Porque el tipo del operador ternario se determina **antes** de elegir la rama. Al mezclar `Integer` con `int`, la promoción numérica binaria fija el tipo de toda la expresión en `int`, lo que fuerza el unboxing de la rama elegida. Y `null.intValue()` es un NPE.

La solución es no mezclar tipos en las ramas:

```java
Integer resultado = true ? valor : Integer.valueOf(0);   // ✔ ambas ramas Integer
```

Este comportamiento generó [múltiples bug reports en el JDK](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6977221), todos cerrados como "not an issue": es lo que la especificación manda.

### 22.4. `list.remove(1)` no hace lo que parece

`List<E>` tiene dos métodos `remove` sobrecargados: `remove(int index)` y `remove(Object o)`. Con una `List<Integer>` son indistinguibles a simple vista:

```java
List<Integer> nums = new ArrayList<>(List.of(10, 20, 30));

nums.remove(1);                       // → [10, 30]   eliminó el ÍNDICE 1
nums.remove(Integer.valueOf(10));     // → [20, 30]   eliminó el VALOR 10
```

La resolución de sobrecarga prefiere siempre la coincidencia exacta sin boxing. `1` es un `int`, así que gana `remove(int index)`.

Para eliminar por valor hay que forzar el tipo: `remove(Integer.valueOf(10))` o `remove((Integer) 10)`.

### 22.5. `Long l = 1;` no compila

```java
Long numero = 1;      // ✘ error: incompatible types: int cannot be converted to Long
Long numero = 1L;     // ✔
```

Para llegar de `int` a `Long` harían falta dos pasos: widening (`int` → `long`) y luego boxing (`long` → `Long`). El JLS permite **boxing seguido de widening reference**, pero **no widening primitive seguido de boxing**. Esa combinación simplemente no está en la lista de conversiones permitidas.

La solución es el sufijo `L`, que hace que el literal ya sea `long` y solo haga falta el boxing.

### 22.6. El coste oculto en rendimiento

El autoboxing crea objetos. En un bucle caliente, eso es asignación de memoria y presión sobre el garbage collector:

```java
// ✘ ~10 millones de objetos Long creados y descartados
Long suma = 0L;
for (long i = 0; i < 10_000_000; i++) {
    suma += i;          // unboxing + suma + boxing, en cada iteración
}

// ✔ cero objetos
long suma = 0L;
for (long i = 0; i < 10_000_000; i++) {
    suma += i;
}
```

La diferencia entre esas dos versiones —una sola letra mayúscula— suele ser de un orden de magnitud. Es el ejemplo canónico del ítem de *Effective Java* "prefer primitives to boxed primitives".

Para colecciones de números en código sensible al rendimiento, considerá arrays primitivos (`int[]`), `IntStream`/`LongStream`, o librerías especializadas como Eclipse Collections o fastutil.

---

# Parte V — Genéricos y type erasure

## 23. Erasure: por qué el cast reaparece

Los genéricos (Java 5) se implementaron mediante **type erasure**: el compilador verifica los tipos y después los **borra**. En el bytecode, `List<String>` y `List<Integer>` son el mismo tipo, `List`.

Se hizo así por compatibilidad con el código anterior a 2004. El precio se paga en runtime, donde la información de tipo genérico ya no existe.

Esto tiene una consecuencia directa sobre este tema: **el compilador reinserta los casts que vos no escribiste**. Compilá esto:

```java
String leer(List<String> lista) { return lista.get(0); }
```

Y el bytecode contiene un `checkcast` que no está en tu código fuente:

```
0: aload_1
1: iconst_0
2: invokeinterface List.get:(I)Ljava/lang/Object;   ← get devuelve Object
7: checkcast     #15   // class java/lang/String    ← cast sintético
10: areturn
```

`List.get()` devuelve `Object` a nivel de bytecode, porque el `<String>` fue borrado. El compilador añade el cast automáticamente.

> **Los genéricos no eliminan los casts: los escriben por vos y garantizan que sean correctos.** Es el mismo código que se escribía a mano en Java 1.4, con la diferencia de que ahora el compilador verifica la seguridad de antemano.

## 24. Unchecked cast y heap pollution

Si el tipo genérico no existe en runtime, el `checkcast` no puede verificarlo. De ahí el aviso más incomprendido del compilador:

```java
List crudo = obtenerListaLegacy();
List<String> tipada = (List<String>) crudo;
```

```
warning: [unchecked] unchecked cast
  required: List<String>
  found:    List
```

El compilador dice literalmente: *"emito este cast pero no puedo verificarlo; en runtime solo comprobaré que es una `List`, no que contenga `String`s"*. Si en realidad contiene `Integer`s, nada falla **en esa línea**.

### 24.1. Heap pollution: el fallo que viaja en el tiempo

```java
List<String> strings = new ArrayList<>();
List crudo = strings;        // warning: raw type
crudo.add(42);               // warning: unchecked call — pero se ejecuta sin problema

System.out.println(strings.size());   // 1  ← una List<String> con un Integer adentro

String s = strings.get(0);   // ✘ ClassCastException: Integer cannot be cast to String
```

Esto se llama **heap pollution**: una colección genérica contiene elementos del tipo equivocado. El daño se hace en la línea del `add`, pero la excepción aparece en el `get`, que puede estar en otra clase, en otro módulo, ejecutándose horas después.

Por eso los *unchecked warnings* merecen atención real: no son ruido del compilador, son la única señal que vas a recibir antes de que el problema se vuelva invisible.

### 24.2. Cuándo `@SuppressWarnings("unchecked")` es aceptable

A veces no hay alternativa —interoperar con APIs viejas, deserialización, reflexión—. El consenso de la comunidad y de *Effective Java* es claro:

> Usá `@SuppressWarnings("unchecked")` **solo** cuando puedas demostrar que el cast es seguro, en el **ámbito más pequeño posible**, y con un comentario que explique **por qué** es seguro.

```java
// ✘ Mal: anotación en toda la clase, oculta problemas futuros
@SuppressWarnings("unchecked")
public class Servicio { ... }

// ✔ Bien: ámbito mínimo y justificación
public <T> List<T> copiarDefensiva(List<T> original) {
    // Seguro: el array solo se usa internamente y nunca se expone,
    // así que ningún cliente puede insertar elementos de otro tipo.
    @SuppressWarnings("unchecked")
    T[] copia = (T[]) original.toArray();
    return Arrays.asList(copia);
}
```

Como la anotación no se puede poner sobre una expresión suelta, el idiom es declarar una variable local solo para acotar el ámbito.

**Alternativas antes de suprimir:**

1. Reconstruir la colección con el tipo correcto: `new ArrayList<String>(crudo)` — copia, pero es seguro y verificable.
2. Usar un type token: `Class<T>` + `tipo.cast(x)`, que sí verifica de verdad ([sección 20](#20-casting-reflexivo-classcast)).
3. Cambiar la firma para que el genérico fluya sin necesidad de cast.

---

# Parte VI — Cierre

## 25. Lo que parece casting y no lo es

Las cuatro fuentes de referencia de este documento presentan estas conversiones dentro del capítulo de type casting. **Técnicamente no lo son**, y confundirlas lleva a intentar cosas que no compilan.

### 25.1. `String` ↔ número

No existe ninguna conversión de tipo entre `String` e `int`. Ninguna. Son tipos completamente ajenos.

```java
String texto = "42";
int n = (int) texto;   // ✘ error: String cannot be converted to int
```

Lo que hay son **métodos de librería que parsean o formatean**:

```java
// String → número
int n = Integer.parseInt("42");             // → int primitivo
Integer obj = Integer.valueOf("42");        // → Integer (usa la caché)
double d = Double.parseDouble("42.5");

// número → String
String s1 = String.valueOf(42);
String s2 = Integer.toString(42);
String s3 = "" + 42;                        // funciona, pero es el menos claro
```

Y son operaciones que **fallan con datos inválidos**, a diferencia de los casts entre primitivos:

```java
Integer.parseInt("50.0");   // ✘ NumberFormatException: For input string: "50.0"
Integer.parseInt(" 50");    // ✘ NumberFormatException: For input string: " 50"
```

Esos dos casos son la causa de la mayoría de los `NumberFormatException` en producción: `parseInt` **no tolera decimales ni espacios**. El idiom para entrada de usuario:

```java
String entrada = leerDelUsuario().trim();       // los espacios hay que quitarlos a mano
try {
    int valor = Integer.parseInt(entrada);
} catch (NumberFormatException e) {
    // mensaje de error para el usuario, no un stack trace
}
```

Y para pasar de un texto decimal a entero hacen falta **dos pasos**, un parseo y un cast:

```java
int n = (int) Double.parseDouble("50.9");   // 50
```

### 25.2. `toString()` no es un cast

```java
Object o = obtenerAlgo();
String s1 = (String) o;         // cast: falla con CCE si no es un String
String s2 = o.toString();       // llamada a método: funciona con cualquier objeto
String s3 = String.valueOf(o);  // igual, pero devuelve "null" si o es null
```

Tres operaciones distintas con resultados distintos. `(String) o` exige que el objeto **sea** un `String`; `toString()` produce una *representación textual* de cualquier cosa; `String.valueOf(o)` hace lo mismo pero sin lanzar NPE cuando `o` es `null`.

### 25.3. Conversiones de dominio

Convertir un `PedidoEntity` en un `PedidoDTO` no es casting: es **mapeo**, y requiere código que construya un objeto nuevo. Si alguna vez sentís la tentación de escribir `(PedidoDTO) entidad`, la respuesta es que no existe herencia entre ellos y no debería existir.

## 26. Casos de uso reales

Dónde aparece esto de verdad, más allá de los ejercicios.

### 26.1. Porcentajes y promedios

El caso más frecuente, y el que W3Schools usa como ejemplo de la vida real:

```java
int puntajeMaximo = 500;
int puntajeUsuario = 423;

// Castear un operando ANTES de dividir
double porcentaje = (double) puntajeUsuario / puntajeMaximo * 100.0;
System.out.println(porcentaje);   // 84.6
```

### 26.2. Lectura de datos binarios

Protocolos de red y formatos de archivo trabajan en bytes; la lógica trabaja en `int`:

```java
byte[] cabecera = leerBytes(4);

// ✘ Mal: los bytes ≥ 128 son negativos y corrompen el resultado
int longitud = (cabecera[0] << 24) | (cabecera[1] << 16) | ...;

// ✔ Bien: enmascarar el signo en cada byte
int longitud = ((cabecera[0] & 0xFF) << 24)
             | ((cabecera[1] & 0xFF) << 16)
             | ((cabecera[2] & 0xFF) << 8)
             |  (cabecera[3] & 0xFF);
```

Sin el `& 0xFF`, cualquier byte con el bit alto activo se ensancha a `int` propagando el signo y arruina la operación de bits. Es la aplicación práctica de la [sección 10.2](#102-byte--char-el-caso-que-corrompe-datos-binarios).

### 26.3. IDs de base de datos

```java
long idBaseDatos = resultSet.getLong("id");

int idLegacy = (int) idBaseDatos;               // ✘ bomba de tiempo
int idLegacy = Math.toIntExact(idBaseDatos);    // ✔ falla ruidosamente si no cabe
```

El primero funciona perfectamente durante años, hasta que la tabla supera los 2.147.483.647 registros o el autoincremental salta. Entonces empieza a devolver IDs negativos que colisionan con registros existentes. El segundo falla el día que el problema aparece, con un stack trace que apunta al lugar exacto.

### 26.4. `equals()`

Uno de los pocos sitios donde el downcast es obligatorio y correcto:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Usuario otro)) return false;   // pattern matching: test + cast + null
    return this.id == otro.id;
}
```

El `instanceof` con pattern matching cubre de una vez el `null`, la comprobación de tipo y el cast. En Java 16+ no hay razón para escribirlo de otra forma.

### 26.5. JSON y `Map<String, Object>`

Trabajar con datos sin esquema es el reino del downcast:

```java
Map<String, Object> json = parsearJson(cuerpo);

// ✘ Frágil: dos formas de fallar en runtime
String nombre = (String) json.get("nombre");
int edad = (int) json.get("edad");     // CCE si el parser devolvió Long o Double

// ✔ Defensivo
Object valorEdad = json.get("edad");
int edad = switch (valorEdad) {
    case Integer i -> i;
    case Long l    -> Math.toIntExact(l);
    case Number n  -> n.intValue();
    case null      -> throw new IllegalArgumentException("falta 'edad'");
    default        -> throw new IllegalArgumentException("'edad' inválida: " + valorEdad);
};
```

Este ejemplo es especialmente traicionero porque los parsers de JSON eligen el tipo numérico según el valor: `{"edad": 30}` puede llegar como `Integer`, `Long` o `Double` según la librería y el tamaño del número. Un `(int)` directo funciona en desarrollo y falla con datos reales.

**La mejor solución, cuando podés:** no manipular `Map<String, Object>` a mano. Deserializá a un `record` tipado con Jackson o similar y dejá que la librería haga las conversiones y las valide.

## 27. Anti-patrones

### 27.1. Castear para silenciar al compilador

```java
// ✘ El cast tapa el problema en vez de resolverlo
long total = obtenerTotal();
int t = (int) total;   // "no compilaba, le puse el cast y ya está"
```

Un cast escrito para que el error desaparezca, sin razonar sobre el rango de valores, es deuda técnica que cobra intereses. Si no podés explicar por qué el valor siempre cabe, no lo castees: validá.

### 27.2. Cadenas de `instanceof`

Ya cubierto en la [sección 18.4](#184-cuándo-el-downcast-indica-un-problema-de-diseño). Si escribís tres `instanceof` seguidos, el problema es el diseño, no el casting.

### 27.3. Comparar wrappers con `==`

```java
if (usuario.getEdad() == otroUsuario.getEdad())   // ✘ si getEdad() devuelve Integer
```

Funciona en los tests (edades < 128) y falla con datos reales. Ver [sección 22.2](#222--con-wrappers-el-integer-cache).

### 27.4. `double` para dinero

```java
double precio = 19.99;
double total = precio * 3;            // 59.96999999999999
int centavos = (int) (total * 100);   // 5996, no 5997
```

Usá `BigDecimal` o enteros de centavos. Sin excepciones.

### 27.5. Castear el resultado en vez de un operando

```java
double promedio = (double) (suma / cantidad);   // ✘ ya se perdió todo
double promedio = (double) suma / cantidad;     // ✔
```

### 27.6. `@SuppressWarnings` en clases enteras

Suprime el aviso de hoy y todos los futuros, incluidos los que sí importan.

## 28. Checklist y tabla de decisión

### Antes de escribir un cast, preguntate:

- [ ] ¿Es entre primitivos o entre referencias? Son dos mundos con reglas opuestas.
- [ ] Si es narrowing primitivo: **¿el valor cabe siempre?** Si no podés demostrarlo, validá o usá `Math.toIntExact`.
- [ ] Si es de flotante a entero: **¿quiero truncar o redondear?** Son cosas distintas.
- [ ] Si es un downcast: **¿hay un `instanceof` antes?** Preferí pattern matching.
- [ ] Si es un downcast: **¿por qué no es polimorfismo?** El cast puede estar tapando un problema de diseño.
- [ ] Si es un unchecked cast: **¿puedo demostrar que es seguro?** Si no, no lo suprimas.
- [ ] Si hay wrappers: **¿puede ser `null`?** El unboxing de `null` es NPE.
- [ ] En una expresión aritmética: **¿el cast está antes de operar o después?** Antes es casi siempre lo correcto.

### Tabla de decisión rápida

| Situación | Solución |
|---|---|
| `int` → `double` para dividir | `(double) a / b` (antes de dividir) |
| `double` → `int` truncando | `(int) d` |
| `double` → `int` redondeando | `(int) Math.round(d)` |
| `long` → `int` con dato externo | `Math.toIntExact(l)` |
| `String` → `int` | `Integer.parseInt(s.trim())` en `try/catch` |
| `int` → `String` | `String.valueOf(n)` |
| `char` dígito → `int` | `Character.getNumericValue(c)` o `c - '0'` si ya validaste |
| `byte` → valor 0…255 | `b & 0xFF` |
| Supertipo → subtipo | `if (x instanceof Sub s)` |
| Varios subtipos posibles | `switch` con patterns sobre una `sealed interface` |
| `Object` a genérico | `Class<T>` + `tipo.cast(x)` |
| Dinero | `BigDecimal` o `long` de centavos; nunca `double` |
| Multiplicación que puede desbordar | castear el primer factor a `long`, o `Math.multiplyExact` |

## 29. Autoevaluación

Respondé sin ejecutar; las respuestas están abajo.

1. ¿Qué imprime `System.out.println((byte) 130);`?
2. ¿Por qué `byte c = a + b;` no compila si `a` y `b` son `byte`?
3. ¿Cuál es la diferencia entre `(int) -3.7`, `Math.round(-3.7)` y `Math.floor(-3.7)`?
4. ¿Es `short s = miChar;` una conversión válida sin cast? ¿Por qué?
5. ¿Puede una widening conversion perder información? Dá un ejemplo.
6. `Integer a = 100, b = 100;` → ¿`a == b`? ¿Y si valen 1000?
7. ¿Por qué `byte b = 10; b += 300;` compila pero `b = b + 300;` no?
8. ¿Qué instrucción de bytecode genera un upcast?
9. ¿Qué imprime `(int) (0.0 / 0.0)`?
10. ¿Por qué `Long l = 1;` no compila?
11. ¿Cuál es la diferencia entre `lista.remove(1)` y `lista.remove(Integer.valueOf(1))` en una `List<Integer>`?
12. ¿Por qué `(String[]) lista.toArray()` lanza `ClassCastException`?

<details>
<summary><strong>Respuestas</strong></summary>

1. **`-126`.** 130 en binario termina en `10000010`; esos 8 bits leídos en complemento a dos dan −126.
2. Por la **promoción numérica**: `a + b` se evalúa como `int`, y asignar un `int` a un `byte` es narrowing. Hace falta `(byte)(a + b)`.
3. `(int) -3.7` → **−3** (trunca hacia cero); `Math.round(-3.7)` → **−4** (al más cercano); `Math.floor(-3.7)` → **−4.0** (hacia abajo, y devuelve `double`). Para positivos, el cast y `floor` coinciden; para negativos, no.
4. **No es válida.** `char` es sin signo (0…65535) y `short` con signo (−32768…32767): ninguno contiene al otro, así que ambas direcciones son narrowing y requieren cast. La cadena `byte → short → char → int` que enseñan muchos tutoriales es incorrecta en ese eslabón.
5. **Sí**, en tres casos: `int`→`float`, `long`→`float` y `long`→`double`. Ejemplo: `(int)(float) 16777217` devuelve `16777216`.
6. Con 100: **`true`** (dentro de la caché −128…127). Con 1000: **`false`**. Siempre usá `equals()`.
7. Porque `b += 300` está definido como `b = (byte)(b + 300)`: el operador de asignación compuesta **incluye un cast implícito**. El resultado es `54`.
8. **Ninguna.** El upcast no existe en runtime; el bytecode es simplemente `aload`/`areturn`. El downcast sí genera `checkcast`.
9. **`0`.** Por especificación (JLS 5.1.3), `NaN` convertido a entero da `0` — sin excepción ni aviso.
10. Porque haría falta widening (`int`→`long`) **seguida de** boxing (`long`→`Long`), y esa combinación no está permitida. `Long l = 1L;` sí compila.
11. `remove(1)` llama a `remove(int index)` y elimina el elemento en la **posición** 1. `remove(Integer.valueOf(1))` llama a `remove(Object)` y elimina el **valor** 1. La sobrecarga sin boxing gana siempre.
12. Porque `toArray()` sin argumentos crea internamente un `Object[]`, y el tipo real del array es `Object[]` aunque sus elementos sean `String`. Se soluciona con `toArray(new String[0])`.

</details>

---

## 30. Fuentes

**Documentación normativa**

- [Java Language Specification, Capítulo 5: Conversions and Contexts](https://docs.oracle.com/javase/specs/jls/se21/html/jls-5.html) — la fuente de verdad: las 19 widening y 22 narrowing conversions, las reglas de `NaN`/infinito, el rango cacheado del boxing y los cinco contextos de conversión.
- [Oracle · Pattern Matching for instanceof (Java 21)](https://docs.oracle.com/en/java/javase/21/language/pattern-matching-instanceof.html) — sintaxis y reglas de scope de las pattern variables.
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) — switch con patterns y exhaustividad sobre tipos `sealed`.
- [Oracle · ArrayStoreException](https://docs.oracle.com/javase/8/docs/api/java/lang/ArrayStoreException.html) — la covarianza de arrays y su verificación en runtime.

**Recursos de referencia del tema**

- [W3Schools · Java Type Casting](https://www.w3schools.com/java/java_type_casting.asp) — widening y narrowing, el ejemplo del porcentaje. *Su cadena de conversión incluye el eslabón erróneo `short → char`.*
- [W3Schools · Wrapper Classes](https://www.w3schools.com/java/java_wrapper_classes.asp) — tabla primitivo↔wrapper y métodos `intValue()`/`toString()`.
- [TutorialsPoint · Java Type Casting](https://www.tutorialspoint.com/java/java_type_casting.htm) — implícito vs explícito. *Misma cadena incorrecta.*
- [TutorialsPoint · Autoboxing and Unboxing](https://www.tutorialspoint.com/java/java-autoboxing-unboxing.htm) — cuándo el compilador inserta las conversiones.
- [TutorialsPoint · Unicode System](https://www.tutorialspoint.com/java/java_unicode_system.htm) — `char` como valor numérico de 16 bits.
- [Programiz · Java Type Conversion](https://www.programiz.com/java-programming/typecasting) — la explicación más limpia de las dos conversiones básicas.
- [Programiz · Autoboxing and Unboxing](https://www.programiz.com/java-programming/autoboxing-unboxing) — ejemplos con colecciones.
- [Simplilearn · Type Casting in Java](https://www.simplilearn.com/tutorials/java-tutorial/type-casting-in-java) — motivación y ejemplos. *Invierte los términos "casting up/down" y presenta `parseInt`/`valueOf` como casting.*

**Análisis técnico y buenas prácticas**

- [SEI CERT NUM12-J · Ensure conversions to narrower types do not result in lost data](https://cmu-sei.github.io/secure-coding-standards/sei-cert-oracle-coding-standard-for-java/rules/numeric-types-and-operations-num/num12-j) — validación de rangos antes de un narrowing, con ejemplos conformes y no conformes.
- [CodeQL · Implicit narrowing conversion in compound assignment](https://codeql.github.com/codeql-query-help/java/java-implicit-cast-in-compound-assignment/) — el cast escondido de `+=` como defecto detectable automáticamente.
- [Artima · Prefer polymorphism over instanceof and downcasting](https://www.artima.com/interfacedesign/PreferPoly.html) — el argumento clásico contra las cadenas de `instanceof`.
- [DZone · Java's Ternary Is Tricky With Autoboxing/Unboxing](https://dzone.com/articles/javas-ternary-is-tricky-with-autoboxingunboxing) — análisis del NPE en el operador ternario.
- [DZone · Exact Conversion of Long to Int in Java](https://dzone.com/articles/exact-conversion-of-long-to-int-in-java) — `Math.toIntExact` frente al cast silencioso.
- [Baeldung · Java Warning "Unchecked Cast"](https://www.baeldung.com/java-warning-unchecked-cast) — cuándo y cómo acotar `@SuppressWarnings`.
- [Baeldung · Convert Between int and char](https://www.baeldung.com/java-convert-int-char) — `- '0'` frente a `Character.getNumericValue()` y sus límites.
- [InfoWorld · How to use Java generics to avoid ClassCastExceptions](https://www.infoworld.com/article/2257860/how-to-use-java-generics-to-avoid-classcastexceptions.html) — erasure y los casts sintéticos.

---

**Siguiente tema:** Operadores — precedencia, asociatividad, operadores de bits y cómo interactúan con la promoción numérica que vimos aquí.
