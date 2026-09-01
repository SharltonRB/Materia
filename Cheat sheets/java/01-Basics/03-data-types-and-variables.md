# Cheat Sheet · Data Types and Variables

> Repaso rápido de [`Pura/Java/01-Basics/03-data-types-and-variables.md`](../../../Pura/Java/01-Basics/03-data-types-and-variables.md) · Java 17+

## En 30 segundos

- Un tipo responde tres preguntas: **cuánta memoria ocupa**, **qué valores admite** y **qué operaciones acepta**.
- Dos familias: **primitivos** (8, guardan el valor) y **de referencia** (infinitos, guardan una dirección). Esta distinción explica el 80% de las confusiones del tema.
- Los **campos** reciben valor por defecto; las **variables locales, no**.
- De aquí salen los bugs más caros de la industria: dinero con decimales, comparaciones que fallan a partir de 127, contadores que se vuelven negativos y textos que se cortan con un emoji.

## Primitivos vs referencia

```java
int numero = 42;        // CONTIENE el 42
String texto = "hola";  // contiene una DIRECCIÓN a un objeto del heap
```

| Aspecto | Primitivos | De referencia |
|---|---|---|
| Cuántos hay | exactamente 8 | infinitos |
| Qué guardan | el valor directamente | una dirección al objeto |
| Dónde viven | stack (si son locales) o dentro del objeto | el objeto siempre en el heap |
| ¿Pueden ser `null`? | **No, nunca** | Sí |
| ¿Tienen métodos? | No (`5.length()` no existe) | Sí |
| Nombre | minúscula (`int`) | mayúscula (`String`) |
| `==` compara | **valores** | **referencias** ⚠️ |

## Los 8 tipos primitivos

| Tipo | Bits | Rango | Por defecto | Literal |
|---|---|---|---|---|
| `byte` | 8 | −128 a 127 | `0` | `byte b = 100;` |
| `short` | 16 | −32.768 a 32.767 | `0` | `short s = 10000;` |
| `int` | 32 | ±2.147.483.647 | `0` | `int i = 100000;` |
| `long` | 64 | ±9,22 × 10¹⁸ | `0L` | `long l = 15000000000L;` |
| `float` | 32 | IEEE 754, ~6-7 dígitos | `0.0f` | `float f = 5.75f;` |
| `double` | 64 | IEEE 754, ~15-16 dígitos | `0.0d` | `double d = 19.99;` |
| `char` | 16 | 0 a 65.535 | el carácter nulo | `char c = 'A';` |
| `boolean` | *no definido* | `true` / `false` | `false` | `boolean b = true;` |

- **Todos los enteros son con signo.** Java no tiene `unsigned`; fue un rechazo deliberado. Desde Java 8 hay métodos (`Integer.toUnsignedString`, `divideUnsigned`) para tratarlos como si no lo fueran.
- **El tamaño de `boolean` no está en la especificación.** En HotSpot: 1 byte en arrays, ranura de 4 bytes como campo suelto.
- `long correcto = 15000000000L;` — sin la `L` no compila (*integer number too large*).
- `float correcto = 5.75f;` — sin la `f` no compila: un literal decimal es `double`.

> Jenkov escribe `float myFloat = 199.99;` en su tutorial. **Ese código no compila.** Hasta los tutoriales veteranos tienen erratas: ejecutá lo que leas.

## Las cuatro categorías de variable

| | Local | De instancia | De clase (`static`) | Parámetro |
|---|---|---|---|---|
| Se declara | dentro de un método/bloque | en la clase, fuera de métodos | en la clase, con `static` | en la firma |
| Nace | al entrar en el bloque | con el `new` | al inicializar la clase | al invocar |
| Muere | al salir del bloque | cuando el GC recoge el objeto | al descargar la clase | al terminar |
| Copias | una por invocación | una por objeto | **una sola, compartida** | una por invocación |
| Valor por defecto | **ninguno** ⚠️ | sí | sí | el que le pases |
| Se almacena en | stack | dentro del objeto (heap) | metaspace | stack |

**Por qué la asimetría de los valores por defecto:** los campos se inicializan en la fase de *preparación* del enlazado. Las locales viven en la pila, que es memoria reutilizada llena de basura de llamadas anteriores; en vez de limpiarla, Java exige que las inicialices y el compilador lo verifica.

```java
int local;
System.out.println(local);   // ✗ "variable local might not have been initialized"

int resultado;
if (condicion) { resultado = 1; }
System.out.println(resultado);   // ✗ ¿y si condicion es false?
```

## `final` y `var`

```java
final int MAXIMO = 100;
MAXIMO = 200;                      // ✗ error de compilación

final List<String> lista = new ArrayList<>();
lista.add("hola");                 // ✔ ¡esto SÍ funciona!
lista = new ArrayList<>();         // ✗ esto no
```

**`final` sobre una referencia congela la referencia, no el objeto.** Para inmutabilidad real: `List.of(...)` o `List.copyOf(...)`.

Una local que nunca se reasigna es **effectively final** aunque no lleve la palabra, y solo esas pueden usarse dentro de una lambda.

```java
var mensaje = "Hola";              // ✔ String
var lista = new ArrayList<String>();
var sinValor;                      // ✗ no hay de dónde inferir
var nulo = null;                   // ✗ null no tiene tipo
private var campo = 1;             // ✗ no vale para campos
void m(var parametro) { }          // ✗ ni para parámetros
```

Solo en **locales**, en el `for` y en el `try-with-resources`. El tipo se decide al compilar y es igual de fijo.

## Wrappers, autoboxing y sus dos costes

| Primitivo | Wrapper | | Primitivo | Wrapper |
|---|---|---|---|---|
| `byte` | `Byte` | | `float` | `Float` |
| `short` | `Short` | | `double` | `Double` |
| `int` | **`Integer`** ⚠️ | | `char` | **`Character`** ⚠️ |
| `long` | `Long` | | `boolean` | `Boolean` |

**Existen por tres motivos:** las colecciones solo admiten objetos (`List<int>` no existe); pueden ser `null` (columna nullable, campo opcional de JSON); traen métodos y constantes (`Integer.parseInt`, `MAX_VALUE`, `Double.isNaN`).

```java
Integer objeto = 42;      // autoboxing:  Integer.valueOf(42)
int primitivo = objeto;   // unboxing:    objeto.intValue()
```

**Coste 1 — el NPE invisible.** No hay ninguna llamada visible, pero el compilador insertó `intValue()`:

```java
Map<String, Integer> stock = new HashMap<>();
int cantidad = stock.get("no-existe");                  // 💥 NPE: get() devolvió null
int cantidad = stock.getOrDefault("no-existe", 0);      // ✔
```

**Coste 2 — el rendimiento.** `Long suma = 0L;` dentro de un bucle de 10 millones crea 10 millones de objetos; con `long` son cero. Uno o dos órdenes de magnitud de diferencia. Por eso existen `IntStream`, `LongStream` y `DoubleStream` separados de `Stream<Integer>`.

> Jenkov usa `new Integer(45)`. **Deprecado desde Java 9 y marcado para eliminación.** La diferencia no es cosmética: `valueOf()` usa la caché, `new` siempre crea objeto.

## Casting

### Widening (automático)

```
byte → short → int → long → float → double
```

No hay pérdida posible… **salvo en tres casos**: `int`→`float`, `long`→`float` y `long`→`double` pierden precisión porque la mantisa es finita.

### Narrowing (manual y peligroso)

```java
double decimal = 9.78;
int entero = (int) decimal;   // 9 ← el .78 se PIERDE, no se redondea
```

**El desbordamiento es silencioso**, lo más peligroso del lenguaje:

```java
byte pequeño = (byte) 130;    // -126  😱  sin error, sin aviso, sin excepción
```

### Promoción numérica

**En una expresión aritmética, todo tipo menor que `int` se promociona a `int` antes de operar.**

```java
byte a = 10, b = 20;
byte suma = a + b;            // ✗ "possible lossy conversion from int to byte"
byte suma = (byte)(a + b);    // ✔

char c = 'A';
int codigo = c + 1;               // 66, es int
char siguiente = (char)(c + 1);   // 'B'
```

**Consecuencia:** usar `byte` o `short` "para ahorrar memoria" en variables sueltas es contraproducente — la JVM las alinea igual y encima obliga a castear. Solo compensan en **arrays grandes** (`byte[]` de un megabyte sí ocupa un megabyte) y en formatos binarios.

## Los 10 errores clásicos

| # | Error | Qué pasa |
|---|---|---|
| 1 | `Integer a = 128, b = 128; a == b` | `false`. La caché de `valueOf` cubre −128 a 127. Funciona en dev y falla en prod |
| 2 | `double` para dinero | `0.1 + 0.2 → 0.30000000000000004`. Un céntimo por transacción es un descuadre contable |
| 3 | Comparar `double` con `==` | `1.0 - 0.9 == 0.1` es `false` |
| 4 | Ignorar `Infinity` y `NaN` | `1.0/0.0` **no lanza**: da `Infinity`. Y `NaN == NaN` es `false` |
| 5 | Desbordamiento de `int` | `Integer.MAX_VALUE + 1` da `-2147483648`, sin aviso |
| 6 | Creer que `char` es un carácter | `"😊".length()` es **2**: surrogate pair |
| 7 | Wrappers donde bastan primitivos | objetos en el heap, unboxing y NPE latentes |
| 8 | `char` sumado | `'A' + 1` imprime `66`; y `"" + a + 1` da `"A1"` |
| 9 | Local sin inicializar | no compila (análisis de definite assignment) |
| 10 | `byte`/`short` "para ahorrar" | no ahorra nada y obliga a castear |

### Dinero, en detalle

```java
new BigDecimal(0.1)     // ✗ 0.1000000000000000055511... ¡ya venía roto!
new BigDecimal("0.1")   // ✔ exactamente 0.1
```

`BigDecimal` es **inmutable**: `total.add(x)` no modifica `total`, devuelve uno nuevo. Alternativa más simple: **la unidad mínima como entero** (`long centimos = 1999`), que es lo que hace Stripe.

**Cuándo sí usar `double`:** cálculo científico, gráficos, estadística, física — donde un error en el decimoquinto decimal es irrelevante.

### `char` y Unicode

```java
String emoji = "😊";
emoji.length();                                // 2  😱
emoji.codePointCount(0, emoji.length());       // 1  ✔
new StringBuilder("Hola 😊").reverse();        // ✗ invierte los surrogates y lo rompe
```

Aparece siempre en validaciones de longitud: un campo "máximo 100 caracteres" que en realidad cuenta unidades de 16 bits.

## Guía de decisión

| Necesito… | Usá | Por qué |
|---|---|---|
| Un entero normal | `int` | el tipo por defecto |
| Identificador, timestamp, contador sin techo | `long` | `int` desborda antes de lo que creés |
| **Dinero** | `BigDecimal` o `long` de céntimos | `double` pierde precisión |
| Cálculo científico o estadístico | `double` | rápido, el error es despreciable |
| Un decimal en general | `double`, no `float` | `float` da 6-7 dígitos; el ahorro rara vez compensa |
| Un sí/no | `boolean` | nunca un `int` 0/1 |
| Un valor que puede faltar | wrapper u `Optional` | los primitivos no admiten `null` |
| Meterlo en una colección | wrapper, obligatorio | los genéricos no aceptan primitivos |
| Un conjunto fijo de opciones | `enum` | ni `String` ni constantes `int` |
| Muchos bytes binarios | `byte[]` | aquí sí ahorra de verdad |

## Bajo el capó

- **Un `Integer` cuesta ~5 veces más que un `int`**: 12 bytes de cabecera + 4 del valor + 4-8 de la referencia. En un `List<Integer>` de un millón de elementos frente a `int[]`, son decenas de MB más el coste de GC.
- **IEEE 754:** `double` = 1 bit de signo + 11 de exponente + 52 de mantisa. Solo se representan exactamente las fracciones con denominador potencia de 2. El 0.1 no lo es.
- `0.0 == -0.0` es `true`, pero `Double.compare(0.0, -0.0)` devuelve `1`. `compare` y `equals` distinguen los ceros y tratan `NaN` como igual a sí mismo, al revés que `==`.
- **Compact Strings** (Java 9, JEP 254): `String` guarda un `byte[]`, no un `char[]`; si todo cabe en Latin-1 usa 1 byte por carácter.
- **Project Valhalla** trabaja en *value classes*: "se declara como una clase, funciona como un `int`". Aún en preview; seguilo, pero no construyas nada encima.

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Cuántos primitivos y cuáles?** Ocho: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`.
- **¿Primitivo vs referencia?** El primitivo guarda el valor; la referencia, una dirección al heap. Los primitivos no pueden ser `null` ni tienen métodos.
- **¿Por qué `Integer 127 == 127` es `true` y con `128` no?** Por la caché de `Integer.valueOf()` (−128 a 127).
- **¿Por qué `0.1 + 0.2 != 0.3`?** `double` es IEEE 754 en base 2 y 0.1 no tiene representación exacta.
- **¿`new BigDecimal(0.1)` o `("0.1")`?** Siempre desde `String`: desde `double` arrastrás el error que querías evitar.
- **¿Qué vale una local sin inicializar?** Nada: no compila. Solo los campos reciben valor por defecto.
- **¿Cuánto ocupa un `boolean`?** No está definido. En HotSpot, 1 byte en arrays y 4 como campo.
- **¿Por qué `byte c = a + b;` no compila?** Por la promoción numérica a `int`. Hace falta `(byte)(a + b)`.
- **¿`(byte) 130`?** −126. El desbordamiento en narrowing es silencioso.
- **¿`1/0` y `1.0/0.0` hacen lo mismo?** No: el primero lanza `ArithmeticException`, el segundo da `Infinity`.
- **¿Por qué `NaN == NaN` es `false`?** Por definición de IEEE 754. Se comprueba con `Double.isNaN()`.
- **¿`"😊".length()`?** 2 — surrogate pair. Para caracteres reales, `codePointCount()`.
- **¿`final List` impide añadir?** No: congela la referencia, no el objeto.
- **¿`var` hace a Java dinámico?** No: el tipo se infiere al compilar y es igual de fijo.

</details>
