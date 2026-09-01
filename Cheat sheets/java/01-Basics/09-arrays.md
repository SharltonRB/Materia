# Cheat Sheet · Arrays

> Repaso rápido de [`Pura/Java/01-Basics/09-arrays.md`](../../../Pura/Java/01-Basics/09-arrays.md) · Java 17+

## En 30 segundos

- Un array **es un objeto**: vive en el heap, hereda de `Object` y tiene un `Class` propio.
- **`length` es un campo, no un método** (a diferencia de `String.length()` y `List.size()`).
- **El tamaño es fijo desde la creación.** "Redimensionar" siempre significa crear otro y copiar.
- Asignar un array **no lo copia**: copia la referencia.
- Java **no tiene matrices**: tiene arrays de arrays, y eso explica los jagged arrays y la localidad de caché.

## Declarar y crear

```java
int[] datos;        // ✔ los corchetes van junto al TIPO
int datos[];        // legal, herencia de C, desaconsejado
int c[], d;         // ⚠️ solo c es un array; d es un int suelto

int[] a = new int[5];                  // tamaño, con valores por defecto
int[] b = {1, 2, 3};                   // literal — SOLO en la declaración
int[] c = new int[]{1, 2, 3};          // literal en cualquier sitio
```

### Valores por defecto

| Tipo del array | Cada posición vale |
|---|---|
| numérico entero | `0` |
| `float` / `double` | `0.0` |
| `char` | el carácter nulo |
| `boolean` | `false` |
| **referencia** | **`null`** ⚠️ |

Un `String[10]` recién creado son **10 `null`**, no 10 cadenas vacías.

## Acceder

```java
v[0]                  // primer elemento
v[v.length - 1]       // último
v[v.length]           // 💥 ArrayIndexOutOfBoundsException
v[-1]                 // 💥 también
```

**El error del `<=`:** `for (int i = 0; i <= v.length; i++)` lanza la excepción **garantizada**. Siempre `<`.

## La variable y el objeto: asignar no copia

```java
int[] a = {1, 2, 3};
int[] b = a;          // ← misma referencia, NO una copia
b[0] = 99;
a[0];                 // 99
```

### Las cuatro formas de copiar

| Forma | Cuándo |
|---|---|
| `a.clone()` | copia idéntica, la más directa |
| `Arrays.copyOf(a, n)` | copia con otro tamaño (recorta o rellena) |
| `Arrays.copyOfRange(a, i, j)` | un tramo |
| `System.arraycopy(...)` | copiar dentro de un array existente — es el motor de todas las anteriores |

**Todas son superficiales.** Con objetos, la copia comparte las mismas instancias; con una matriz, `clone()` copia el array de filas pero **las filas siguen compartidas**:

```java
int[][] copia = original.clone();                                   // ✗ superficial
int[][] copia = Arrays.stream(original).map(int[]::clone)
                      .toArray(int[][]::new);                       // ✔ profunda
```

## Comparar e imprimir: nada funciona como esperás

```java
System.out.println(datos);          // ✗ [I@1b6d3586
System.out.println(Arrays.toString(datos));      // ✔ [1, 2, 3]
System.out.println(Arrays.deepToString(matriz)); // ✔ para anidados

a == b;                  // ✗ identidad
a.equals(b);             // ✗ también identidad: no está sobrescrito
Arrays.equals(a, b);     // ✔ contenido
Arrays.deepEquals(m, n); // ✔ anidados
Arrays.mismatch(a, b);   // índice del primer elemento distinto (Java 9+)
```

Corolario: **un array nunca funciona como clave de `Map` ni dentro de un `Set`** — no detecta duplicados por contenido. Usá un `record` o una `List`.

## Recorrer

```java
for (int i = 0; i < v.length; i++) { }     // necesitás el índice, o escribir
for (int x : v) { }                        // solo leer
```

**El `for-each` no puede:** conocer el índice, modificar el array (asigna a una copia local), recorrer hacia atrás, saltar elementos, ni recorrer dos arrays en paralelo. Para cualquiera de esas cosas, `for` clásico.

```java
Arrays.stream(v).sum();
Arrays.stream(v).summaryStatistics();      // suma, media, máx, mín, count de una pasada
```

## Matrices: arrays de arrays

```java
int[][] m = new int[3][4];      // 3 filas de 4
m.length;                       // 3   ← número de FILAS
m[0].length;                    // 4   ← longitud de la PRIMERA fila

int[][] irregular = new int[3][];       // jagged: filas de distinta longitud
irregular[0] = new int[2];
irregular[1] = new int[5];
```

**Usá `m[i].length`, nunca `m[0].length`** en el bucle interno: con arrays irregulares falla o se salta datos.

**Localidad de caché:** recorrer **por filas** es notablemente más rápido que por columnas, porque cada fila es un bloque contiguo en memoria. Si podés elegir el orden de los bucles, filas primero.

## `java.util.Arrays`: el kit completo

| Quiero | Método |
|---|---|
| Imprimir | `toString` / `deepToString` |
| Comparar contenido | `equals` / `deepEquals` |
| Saber dónde difieren | `mismatch` |
| Copiar | `copyOf` / `copyOfRange` |
| Rellenar con un valor | `fill` |
| Rellenar con valores **distintos** | `setAll` |
| Ordenar | `sort` |
| Buscar (**ordenado**) | `binarySearch` |
| Convertir a stream | `stream` |
| Sumas de prefijos | `parallelPrefix` |

**`sort` usa dos algoritmos distintos:** *dual-pivot quicksort* para primitivos (rápido, **no estable** — da igual, los primitivos no tienen identidad) y **TimSort** para objetos (**estable**, que es lo que permite ordenar por varios criterios encadenados).

```java
Arrays.fill(v, new Punto());         // ✗ TODAS las posiciones comparten la MISMA instancia
Arrays.setAll(v, i -> new Punto());  // ✔ una instancia por posición
```

**`binarySearch` sin ordenar antes** devuelve resultados equivocados **sin lanzar nada**, y acierta a veces. Es de los bugs más difíciles de detectar.

## `Arrays.asList` y sus tres trampas

```java
List<String> l = Arrays.asList("a", "b");
l.add("c");                    // 💥 UnsupportedOperationException: tamaño fijo
l.set(0, "z");                 // ⚠️ escribe en el ARRAY ORIGINAL
Arrays.asList(arrayDeInt);     // ⚠️ devuelve una lista de UN elemento (el array entero)
```

Ese último caso compila sin aviso: los genéricos no aceptan primitivos, así que `int[]` se toma como un único objeto. La conversión correcta:

```java
Arrays.stream(arrayDeInt).boxed().toList();     // ✔
new ArrayList<>(Arrays.asList(...));            // ✔ si necesitás una lista mutable
```

## Covarianza y genéricos

```java
Object[] objetos = new String[3];
objetos[0] = 42;                 // 💥 ArrayStoreException en runtime
```

Los arrays son **covariantes** (un `String[]` vale como `Object[]`); los genéricos no. Ese es el agujero que Java dejó abierto en 1.0, y la razón de que la comprobación tenga que hacerse en ejecución.

Y por el mismo motivo (**erasure**) **no se puede crear un array genérico**: `new T[10]` no compila.

```java
(String[]) lista.toArray()        // ✗ ClassCastException: el array real es Object[]
lista.toArray(new String[0])      // ✔
lista.toArray(String[]::new)      // ✔ Java 11+
lista.toArray(new String[lista.size()])   // ⚠️ no es más rápido y tiene una carrera
```

## Varargs: un array disfrazado

`void f(String... args)` **es** `void f(String[] args)` con azúcar en la llamada. Dentro del método, `args` es un array normal.

## Array o `ArrayList`

| | Array | `ArrayList` |
|---|---|---|
| Tamaño | **fijo** | crece |
| Primitivos | sí, sin boxing | no: `List<Integer>` |
| API | mínima | completa |
| Covarianza | sí (peligrosa) | no (correcta) |
| Uso típico | rendimiento, datos binarios, tamaño conocido | **todo lo demás** |

**El array sigue siendo la respuesta correcta** en: `byte[]` para datos binarios, `char[]` para contraseñas, buffers de tamaño fijo, código muy caliente con primitivos, y matrices numéricas.

## Devolver arrays desde una API

```java
public String[] getNombres() { return nombres; }         // ✗ regala el estado interno
public String[] getNombres() { return nombres.clone(); } // ✔ copia defensiva
public List<String> getNombres() { return List.copyOf(lista); }  // ✔ mejor aún

public static final String[] CONSTANTE = {"a"};             // ✗ NO es constante: se puede escribir
public static final List<String> CONSTANTE = List.of("a");  // ✔

return null;           // ✗ obliga a un if en cada llamador
return new String[0];  // ✔ devolvé vacío
```

## Anti-patrones (25, condensados)

| ✗ MAL | ✔ BIEN |
|---|---|
| `println(array)` | `Arrays.toString` / `deepToString` |
| `Arrays.toString` en anidados | `deepToString` |
| `==` o `equals` entre arrays | `Arrays.equals` |
| Array como clave de `Map` o en un `Set` | `record` o `List` |
| `clone()` sobre una matriz | copia profunda |
| `Arrays.fill` con objeto mutable | `setAll` |
| `Arrays.asList` / `Stream.of` sobre primitivos | `Arrays.stream(v).boxed()` |
| `add`/`remove` sobre `asList` | `new ArrayList<>(...)` |
| `binarySearch` sin ordenar | ordenar con el mismo criterio |
| `i <= v.length` | `i < v.length` |
| `m[0].length` en el bucle interno | `m[i].length` |
| Recorrer una matriz por columnas | por filas |
| Modificar el array desde un `for-each` | usar índice |
| Devolver el array interno | copia defensiva |
| Devolver `null` | array vacío |
| `public static final T[]` | `List.of(...)` |
| `(String[]) toArray()` | `toArray(String[]::new)` |
| Tamaño de array desde entrada externa sin validar | validar (denegación de servicio) |
| `int[][][][]` | lo que falta es una clase |
| Acumular en array de tamaño desconocido | `ArrayList` |
| Comparador por resta al ordenar | `Comparator.comparingInt` |

## Tablas de decisión

| Necesito | Uso | No uso |
|---|---|---|
| Colección que crece | `ArrayList` | array + `copyOf` |
| Tamaño fijo por el problema | array | `ArrayList` |
| Muchos primitivos | `int[]`, `double[]` | `List<Integer>` |
| Datos binarios | `byte[]` | `List<Byte>` |
| Millones de banderas | `BitSet` | `boolean[]` |
| Constantes de un enum | `EnumSet` | array de flags |
| Clave compuesta de un `Map` | `record` o `List` | array |
| Exponer datos en una API | `List` | array |
| Constante pública | `List.of(...)` | `static final T[]` |
| Devolver "nada" | array vacío / `List.of()` | `null` |

| Quiero | Uso |
|---|---|
| Copia idéntica | `clone()` |
| Copia con otro tamaño | `Arrays.copyOf` |
| Copiar un trozo | `Arrays.copyOfRange` |
| Copiar en un array existente | `System.arraycopy` |
| Copia profunda de matriz | `stream().map(int[]::clone)` |
| Rellenar con valores distintos | `Arrays.setAll` |
| Buscar en array ordenado | `Arrays.binarySearch` |
| Array de primitivos → `List` | `Arrays.stream(v).boxed().toList()` |
| `List` → array | `toArray(T[]::new)` |
| Suma, máximo, media | `Arrays.stream(v).summaryStatistics()` |

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿`length` es método o campo?** Campo. En `String` es método (`length()`) y en las colecciones es `size()`.
- **¿Un array es un objeto?** Sí: vive en el heap, hereda de `Object` y tiene su propio `Class`.
- **¿Por qué `Arrays.toString` y no `println`?** Los arrays no sobrescriben `toString`: imprimen tipo y hash de identidad.
- **¿Por qué un array no sirve como clave de `Map`?** No sobrescribe `equals`/`hashCode`: usa identidad.
- **¿`clone()` de una matriz es copia profunda?** No: copia el array de filas, pero las filas siguen compartidas.
- **¿Qué es `ArrayStoreException`?** La comprobación en runtime que compensa la covarianza de arrays.
- **¿Por qué no se puede hacer `new T[10]`?** Por erasure: en runtime no existe el tipo `T`.
- **¿Qué algoritmo usa `Arrays.sort`?** Dual-pivot quicksort para primitivos (no estable), TimSort para objetos (estable).
- **¿Qué pasa si hago `binarySearch` sin ordenar?** Devuelve resultados incorrectos sin lanzar nada.
- **¿Las tres trampas de `Arrays.asList`?** Tamaño fijo, `set` escribe en el array original, y con primitivos devuelve una lista de un elemento.

</details>
