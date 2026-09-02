# Arrays

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** Hasta aquí los capítulos trataron **valores sueltos**: qué tipos existen ([Data Types and Variables](03-data-types-and-variables.md)), dónde vive cada variable ([Variables and Scopes](04-variables-and-scopes.md)), cómo se operan ([Math Operations](07-math-operations.md)) y cómo se comparan ([Logical, Relational and Bitwise Operators](08-logical-relational-bitwise-operators.md)). Este es el primero que trata **conjuntos de valores**: cómo guardar muchos elementos del mismo tipo en una sola variable y trabajar con ellos.

El array es la estructura de datos más antigua y más simple de Java, y también la que más suposiciones falsas arrastra. Casi nadie lo usa directamente en código de aplicación moderno —para eso está `List`—, pero está **debajo de todo lo demás**: `ArrayList` es un array, `StringBuilder` es un array, `HashMap` es un array, `String` es un array. Entender cómo funciona un array es entender por qué las colecciones se comportan como se comportan.

Y es un tema con trampas propias, casi todas derivadas de un hecho que se aprende tarde: **un array es un objeto, pero un objeto que no implementa nada de lo que uno espera de un objeto**. No sabe imprimirse, no sabe compararse, no sabe copiarse en profundidad, y encima el sistema de tipos le hace una excepción que abre un agujero que ningún otro tipo de Java tiene.

La lista de bugs concretos que salen de este capítulo, todos reales:

- Un log que en vez de los datos imprime `[I@1b6d3586` y no hay forma de saber qué había dentro.
- Un `if (a == b)` sobre dos arrays de contenido idéntico que devuelve `false` siempre.
- Un `HashSet<int[]>` que acepta el mismo array mil veces porque nunca detecta duplicados.
- Una copia "de seguridad" con `clone()` que no protege nada, porque el array era de dos dimensiones.
- Un `Arrays.fill(matriz, fila)` que hace que escribir en una celda modifique las tres filas a la vez.
- Un `Arrays.asList(numeros)` que devuelve una lista de **un** elemento en vez de la lista de números, y compila sin un solo aviso.
- Un `list.add(...)` sobre esa lista que lanza `UnsupportedOperationException` en producción.
- Un `binarySearch` que dice "no encontrado" sobre un elemento que sí está, porque nadie ordenó el array primero.
- Un `ArrayStoreException` en tiempo de ejecución sobre una línea que el compilador aprobó sin quejarse.
- Un `ClassCastException` al castear el resultado de `toArray()` a `String[]`.
- Un recorrido de matriz que tarda casi un 30 % más por leer las columnas en vez de las filas.

Vamos a cubrir el modelo completo: qué es un array en memoria, por qué asignar uno no lo copia, cómo se copia de verdad, cómo se recorre, qué hay en `java.util.Arrays`, por qué los arrays y los genéricos se llevan mal, y —lo más importante para el día a día— **cuándo no usar un array**.

**Sobre la verificación.** Todos los outputs, mensajes de excepción, errores del compilador y volcados de bytecode de este documento se ejecutaron realmente en un JDK 25. Como en los capítulos anteriores, las tres fuentes de referencia usadas para prepararlo contienen errores y huecos en este tema; hay incluso **dos artículos de Baeldung que se contradicen entre sí** sobre qué algoritmo usa `Arrays.sort`. Está todo señalado en la [sección 50](#50-fuentes) con el resultado real al lado.

---

## Índice

**Parte I — Qué es un array**

1. [Qué es un array y qué problema resuelve](#1-qué-es-un-array-y-qué-problema-resuelve)
2. [Declarar: las dos sintaxis y cuál usar](#2-declarar-las-dos-sintaxis-y-cuál-usar)
3. [Crear: las tres formas y cuándo vale cada una](#3-crear-las-tres-formas-y-cuándo-vale-cada-una)
4. [Los valores por defecto](#4-los-valores-por-defecto)
5. [Acceder por índice y la excepción que más se ve](#5-acceder-por-índice-y-la-excepción-que-más-se-ve)
6. [length no es un método](#6-length-no-es-un-método)
7. [Los arrays son objetos y se puede comprobar](#7-los-arrays-son-objetos-y-se-puede-comprobar)

**Parte II — Memoria, referencias y copias**

8. [La variable y el objeto: dos cosas distintas](#8-la-variable-y-el-objeto-dos-cosas-distintas)
9. [Asignar un array no lo copia](#9-asignar-un-array-no-lo-copia)
10. [Las cuatro formas de copiar un array](#10-las-cuatro-formas-de-copiar-un-array)
11. [Copia superficial frente a copia profunda](#11-copia-superficial-frente-a-copia-profunda)
12. [La trampa de rellenar con objetos](#12-la-trampa-de-rellenar-con-objetos)
13. [Comparar arrays: por qué el operador y el método fallan](#13-comparar-arrays-por-qué-el-operador-y-el-método-fallan)
14. [El tamaño es fijo: qué significa exactamente](#14-el-tamaño-es-fijo-qué-significa-exactamente)

**Parte III — Recorrer**

15. [El for clásico](#15-el-for-clásico)
16. [El for-each y lo que revela su bytecode](#16-el-for-each-y-lo-que-revela-su-bytecode)
17. [Lo que el for-each no puede hacer](#17-lo-que-el-for-each-no-puede-hacer)
18. [Patrones de recorrido que conviene conocer](#18-patrones-de-recorrido-que-conviene-conocer)
19. [Streams sobre arrays](#19-streams-sobre-arrays)

**Parte IV — Arrays multidimensionales**

20. [Java no tiene matrices: tiene arrays de arrays](#20-java-no-tiene-matrices-tiene-arrays-de-arrays)
21. [Crear una matriz y lo que hace el bytecode](#21-crear-una-matriz-y-lo-que-hace-el-bytecode)
22. [Jagged arrays: filas de distinta longitud](#22-jagged-arrays-filas-de-distinta-longitud)
23. [Recorrer en dos dimensiones](#23-recorrer-en-dos-dimensiones)
24. [Localidad de caché: por filas frente a por columnas](#24-localidad-de-caché-por-filas-frente-a-por-columnas)

**Parte V — La clase java.util.Arrays**

25. [Qué es Arrays y cómo está organizada](#25-qué-es-arrays-y-cómo-está-organizada)
26. [Imprimir: toString y deepToString](#26-imprimir-tostring-y-deeptostring)
27. [Comparar: equals, deepEquals, compare y mismatch](#27-comparar-equals-deepequals-compare-y-mismatch)
28. [Ordenar: sort y sus dos algoritmos](#28-ordenar-sort-y-sus-dos-algoritmos)
29. [Ordenar con Comparator y el error que ya conocemos](#29-ordenar-con-comparator-y-el-error-que-ya-conocemos)
30. [Estabilidad del ordenamiento](#30-estabilidad-del-ordenamiento)
31. [Buscar: binarySearch y su contrato](#31-buscar-binarysearch-y-su-contrato)
32. [binarySearch sin ordenar: el fallo silencioso](#32-binarysearch-sin-ordenar-el-fallo-silencioso)
33. [Rellenar: fill, setAll y parallelPrefix](#33-rellenar-fill-setall-y-parallelprefix)
34. [System.arraycopy: el motor que hay debajo](#34-systemarraycopy-el-motor-que-hay-debajo)
35. [Arrays.asList y sus tres trampas](#35-arraysaslist-y-sus-tres-trampas)
36. [Referencia completa de java.util.Arrays](#36-referencia-completa-de-javautilarrays)

**Parte VI — Arrays, genéricos y varargs**

37. [Covarianza: el agujero que Java dejó abierto](#37-covarianza-el-agujero-que-java-dejó-abierto)
38. [ArrayStoreException](#38-arraystoreexception)
39. [Por qué no se puede crear un array genérico](#39-por-qué-no-se-puede-crear-un-array-genérico)
40. [toArray y sus tres formas](#40-toarray-y-sus-tres-formas)
41. [Varargs: un array disfrazado](#41-varargs-un-array-disfrazado)

**Parte VII — Cuándo usar un array**

42. [Array frente a ArrayList](#42-array-frente-a-arraylist)
43. [Cuándo un array sigue siendo la respuesta correcta](#43-cuándo-un-array-sigue-siendo-la-respuesta-correcta)
44. [Devolver arrays desde una API](#44-devolver-arrays-desde-una-api)
45. [Límites de tamaño y memoria](#45-límites-de-tamaño-y-memoria)

**Parte VIII — Cierre**

46. [Casos de uso reales](#46-casos-de-uso-reales)
47. [Anti-patrones](#47-anti-patrones)
48. [Checklist y tabla de decisión](#48-checklist-y-tabla-de-decisión)
49. [Autoevaluación](#49-autoevaluación)
50. [Fuentes](#50-fuentes)

---

# Parte I — Qué es un array

## 1. Qué es un array y qué problema resuelve

Imaginá que necesitás guardar las temperaturas de los siete días de la semana. Con lo visto hasta ahora, la única herramienta disponible son variables sueltas:

```java
double lunes = 21.5;
double martes = 23.0;
double miercoles = 19.8;
// ... y así hasta el domingo
```

Esto funciona, y es horrible por tres razones que conviene nombrar, porque son exactamente las que resuelve un array:

1. **No se puede recorrer.** No hay forma de escribir "para cada día, hacé algo". Hay que repetir el código siete veces.
2. **No escala.** Si mañana son 365 días en vez de 7, hay que escribir 365 variables.
3. **El número de elementos no es un dato del programa.** No se puede preguntar "¿cuántos días tengo?" ni pasar el conjunto entero a un método.

Un **array** es un objeto que contiene un número fijo de valores del mismo tipo, guardados en posiciones consecutivas y numeradas. La definición oficial de la documentación de Java es exactamente esa: *"an object containing a fixed number of values of the same type"*.

```java
double[] temperaturas = new double[7];
temperaturas[0] = 21.5;
temperaturas[1] = 23.0;
```

Las tres propiedades de la definición son las que hay que memorizar, porque cada una tiene consecuencias en todo el resto del capítulo:

- **Número fijo.** El tamaño se decide al crearlo y **no puede cambiar nunca**. No hay "añadir un elemento" a un array.
- **Del mismo tipo.** Un `int[]` guarda `int` y nada más. Un `String[]` guarda `String` (o subtipos, y ahí está el problema de la [sección 37](#37-covarianza-el-agujero-que-java-dejó-abierto)).
- **Indexado.** Se accede por un número entero, empezando en cero.

La imagen mental correcta es la de una fila de casilleros numerados, todos del mismo tamaño, pegados uno al lado del otro:

```
índice:     0      1      2      3      4      5      6
         ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┐
         │ 21.5 │ 23.0 │ 19.8 │  0.0 │  0.0 │  0.0 │  0.0 │
         └──────┴──────┴──────┴──────┴──────┴──────┴──────┘

                     length = 7 (siempre 7)
```

Ese detalle de que están **pegados en memoria** no es decorativo: es la razón de que un array sea la estructura más rápida de recorrer que existe, y la razón del efecto de caché de la [sección 24](#24-localidad-de-caché-por-filas-frente-a-por-columnas).

Y una aclaración que ahorra confusión más adelante: **en Java el array es una construcción del lenguaje, no una clase de biblioteca**. No hay ningún archivo `Array.java` en la JDK que puedas abrir. El compilador y la JVM los conocen directamente, con instrucciones de bytecode propias. Lo que sí existe es `java.util.Arrays`, que es una clase de **utilidades** para trabajar con arrays, y que veremos en la Parte V. Son cosas distintas y el parecido de nombre confunde a todo el mundo la primera vez.

## 2. Declarar: las dos sintaxis y cuál usar

Declarar un array es decir "esta variable va a apuntar a un array de este tipo". Hay dos sintaxis y **ambas compilan y significan exactamente lo mismo**:

```java
int[] numeros;    // corchetes junto al tipo
int numeros2[];   // corchetes junto al nombre
```

La segunda existe por compatibilidad con C, de donde Java heredó la sintaxis. **La primera es la que se usa en Java y la que hay que escribir**, por una razón concreta: los corchetes son parte del **tipo**, no del nombre. `int[]` se lee "array de int", que es exactamente lo que la variable contiene.

Que la segunda forma es una herencia incómoda se ve en cuanto se declaran varias variables en una línea:

```java
int[] a, b;     // a es int[], b es int[]     <- lo esperable
int c[], d;     // c es int[], pero d es int  <- ¡sorpresa!
```

En la segunda línea los corchetes solo aplican a `c`. Es una fuente de errores gratuita y otra razón para no usar esa forma nunca.

Con arrays de varias dimensiones la diferencia se vuelve directamente absurda:

```java
int[][] matriz;      // claro: array de arrays de int
int matriz2[][];     // lo mismo, peor escrito
int[] matriz3[];     // también lo mismo. Legal. Ilegible.
```

Las tres declaran el mismo tipo. La tercera es legal Java y no debería escribirse jamás.

**Una declaración no crea nada.** Esto es importante y se pasa por alto:

```java
int[] numeros;
System.out.println(numeros.length);   // no compila: variable numeros might not have been initialized
```

La declaración solo reserva una variable capaz de apuntar a un array. El array todavía no existe. Si la variable es un campo de clase en vez de una variable local, sí compila, pero vale `null` y revienta al usarla:

```java
int[] nulo = null;
System.out.println(nulo.length);
// NullPointerException: Cannot read the array length because "<local4>" is null
```

Ese mensaje tan específico es cortesía de los *helpful NullPointerExceptions* (JEP 358, Java 14), que ya vimos en el capítulo anterior.

## 3. Crear: las tres formas y cuándo vale cada una

Para que exista el array hay que crearlo. Hay tres sintaxis, y cada una tiene su momento.

**Forma 1: `new` con tamaño.** Crea el array vacío, relleno con los valores por defecto de la sección siguiente.

```java
int[] numeros = new int[5];
String[] nombres = new String[3];
```

Se usa cuando **conocés el tamaño pero todavía no los datos**: vas a rellenarlo en un bucle, leyendo de un fichero, o calculándolo.

**Forma 2: literal de array.** Crea y rellena en la misma expresión.

```java
int[] numeros = {1, 2, 3, 4, 5};
String[] dias = {"lunes", "martes", "miércoles"};
```

Es la más legible cuando **conocés los datos en el momento de escribir el código**. El tamaño se deduce: `numeros.length` vale 5.

Esta forma tiene una restricción que sorprende: **solo funciona en la declaración**. No se puede usar en una asignación posterior.

```java
int[] v;
v = {1, 2, 3};
```

```
error: illegal start of expression
    int[] v; v = {1,2,3};
                  ^
```

**Forma 3: `new` con literal.** Es la versión completa de la forma 2, y sirve donde la forma 2 no.

```java
int[] v;
v = new int[]{1, 2, 3};                   // sí compila
metodoQueRecibeArray(new int[]{1, 2, 3}); // sí compila
```

Se usa para asignar después de declarar, y para pasar un array literal directamente como argumento.

**Lo que no se puede hacer** es combinar tamaño y literal:

```java
int[] v = new int[3]{1, 2, 3};
```

```
error: array creation with both dimension expression and initialization is illegal
```

Tiene sentido: el tamaño ya está determinado por el número de elementos, así que declararlo otra vez solo permite contradecirse.

**Un detalle sobre el tamaño.** No tiene que ser una constante; puede ser cualquier expresión `int` calculada en tiempo de ejecución:

```java
int n = leerDelUsuario();
int[] datos = new int[n];
```

Pero si esa expresión es negativa, revienta:

```java
int[] mal = new int[-1];
// NegativeArraySizeException: -1
```

Es una excepción propia, distinta de `ArrayIndexOutOfBoundsException`, y es de las pocas veces que Java falla ruidosamente sobre un tamaño. Si `n` viene de fuera del programa, **hay que validarlo antes**, porque es un vector de denegación de servicio clásico: un `n` enorme provoca un `OutOfMemoryError` que tumba el proceso entero.

## 4. Los valores por defecto

Cuando creás un array con `new`, Java **garantiza** que todas sus posiciones quedan inicializadas. Nunca hay basura de memoria, a diferencia de C. Verificado uno por uno en JDK 25:

```java
System.out.println(Arrays.toString(new int[3]));      // [0, 0, 0]
System.out.println(Arrays.toString(new long[3]));     // [0, 0, 0]
System.out.println(Arrays.toString(new double[3]));   // [0.0, 0.0, 0.0]
System.out.println(Arrays.toString(new float[3]));    // [0.0, 0.0, 0.0]
System.out.println(Arrays.toString(new boolean[3]));  // [false, false, false]
System.out.println(Arrays.toString(new byte[3]));     // [0, 0, 0]
System.out.println(Arrays.toString(new short[3]));    // [0, 0, 0]
System.out.println(Arrays.toString(new String[3]));   // [null, null, null]
System.out.println(Arrays.toString(new Integer[3]));  // [null, null, null]
```

La tabla completa:

| Tipo | Valor por defecto |
|---|---|
| `byte`, `short`, `int`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | el carácter nulo `\u0000`, invisible al imprimir |
| `boolean` | `false` |
| Cualquier referencia | `null` |

El de `char` es el único que no se ve al imprimirlo, porque `\u0000` no dibuja nada. Se comprueba convirtiéndolo a número:

```java
System.out.println((int) (new char[3])[0]);   // 0
```

Y es un valor **distinto del espacio**: `\u0000` no es `' '`. Un `char[]` recién creado que se convierta a `String` produce una cadena llena de caracteres nulos, que en un log o en una base de datos causa problemas raros.

**La consecuencia práctica más importante** es la diferencia entre las dos últimas filas de la tabla:

```java
int[] contadores = new int[3];
System.out.println(contadores[0] + 1);   // 1, funciona

Integer[] contadoresBoxed = new Integer[3];
System.out.println(contadoresBoxed[0] + 1);
// NullPointerException: el elemento es null y se intenta desempaquetar
```

Un array de primitivos está **listo para usar** en cuanto se crea. Un array de wrappers, no: está lleno de `null` y cualquier operación aritmética sobre él revienta, por el mecanismo de desempaquetado del [capítulo anterior](08-logical-relational-bitwise-operators.md#8-cuándo-se-desempaqueta-un-wrapper-y-cuándo-no). Es una razón fuerte para preferir `int[]` sobre `Integer[]` siempre que se pueda.

## 5. Acceder por índice y la excepción que más se ve

Se accede con corchetes y un índice entero. **Los índices empiezan en 0** y el último es `length - 1`.

```java
int[] v = {10, 20, 30};

System.out.println(v[0]);   // 10   <- el primero
System.out.println(v[2]);   // 30   <- el último
v[1] = 99;
System.out.println(Arrays.toString(v));   // [10, 99, 30]
```

Que empiece en 0 no es un capricho: el índice es en realidad un **desplazamiento** desde el inicio del array. El elemento 0 está a cero posiciones del principio. Java lo heredó de C, donde `v[i]` significaba literalmente "la dirección de `v` más `i` veces el tamaño del elemento".

**La excepción.** Si el índice se sale del rango válido, Java lanza `ArrayIndexOutOfBoundsException`:

```java
int[] v = {1, 2, 3};

int x = v[3];
// ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3

int y = v[-1];
// ArrayIndexOutOfBoundsException: Index -1 out of bounds for length 3
```

Los mensajes de JDK 25 son excelentes: dicen el índice que se pidió y la longitud real. Con eso casi siempre se diagnostica sin abrir el depurador.

Su lugar en la jerarquía importa para saber qué capturar:

```
RuntimeException
  └─ IndexOutOfBoundsException
       └─ ArrayIndexOutOfBoundsException
```

Es una **unchecked exception**, así que el compilador no obliga a capturarla. Y eso es correcto: un índice fuera de rango casi siempre es un **bug de programación**, no una condición esperada. La respuesta correcta es arreglar la lógica, no envolverla en un `try/catch`.

**El error de contorno más común**, el que produce esta excepción más veces que ningún otro:

```java
// MAL: <= llega hasta length, que ya está fuera
for (int i = 0; i <= v.length; i++) { System.out.println(v[i]); }

// BIEN
for (int i = 0; i < v.length; i++) { System.out.println(v[i]); }
```

Un solo carácter de diferencia. La regla que evita pensarlo cada vez: **el bucle sobre un array se escribe siempre con `<`, nunca con `<=`**. Si alguna vez necesitás `<=`, es señal de que estás calculando el límite de otra forma y conviene mirarlo dos veces.

Y una nota importante: Java **comprueba los límites en cada acceso**, en tiempo de ejecución. Esa comprobación es lo que convierte un desbordamiento de buffer —la vulnerabilidad más explotada de la historia de C— en una excepción limpia. Tiene un coste, pero el JIT la elimina en la mayoría de los bucles cuando puede demostrar que el índice nunca se sale.

## 6. length no es un método

Para saber cuántos elementos tiene un array se usa `length`, **sin paréntesis**:

```java
int[] v = new int[5];
System.out.println(v.length);   // 5
```

Escribirlo con paréntesis no compila:

```java
System.out.println(v.length());
```

```
error: cannot find symbol
  symbol:   method length()
  location: variable v of type int[]
```

Esto confunde a todo el mundo al principio, porque Java usa tres formas distintas para la misma idea según el tipo:

| Tipo | Cómo se pide el tamaño |
|---|---|
| Array | `v.length` — campo, sin paréntesis |
| `String` | `s.length()` — método |
| `List`, `Set`, `Map` | `c.size()` — método |

No hay ninguna lógica profunda detrás: es una inconsistencia histórica del diseño de Java que ya no se puede arreglar sin romper todo el código existente. Hay que memorizarla y ya.

El motivo técnico de que `length` sea un campo y no un método es que **la JVM tiene una instrucción dedicada** para ello:

```
static int longitud(int[]);
  Code:
       0: aload_0
       1: arraylength      // <-- instrucción propia, una sola operación
       2: ireturn
```

`arraylength` lee la longitud directamente de la cabecera del objeto array. No hay llamada a método que optimizar. Es tan barato como leer un campo porque literalmente lo es.

`length` es además **final de hecho**: se puede leer pero no asignar. `v.length = 10;` no compila. Es coherente con que el tamaño sea inmutable.

## 7. Los arrays son objetos y se puede comprobar

Esta es la idea que reorganiza todo el resto del capítulo: **un array es un objeto**. No es un tipo primitivo, no es una construcción mágica del compilador que desaparece. Es una instancia que vive en el heap, tiene una cabecera y se recolecta con el recolector de basura.

Se puede comprobar directamente:

```java
int[] a = new int[3];

System.out.println(a.getClass().getName());        // [I
System.out.println(a.getClass().getSimpleName());  // int[]
System.out.println(a.getClass().getSuperclass());  // class java.lang.Object
System.out.println(a.getClass().isArray());        // true
System.out.println(Arrays.toString(a.getClass().getInterfaces()));
// [interface java.lang.Cloneable, interface java.io.Serializable]
```

Cuatro cosas que sacar de ahí:

**Primera: tienen una clase, y su nombre es raro.** El formato viene de la especificación de la JVM: un `[` por cada dimensión, seguido de un código de tipo.

| Array | `getName()` |
|---|---|
| `int[]` | `[I` |
| `double[]` | `[D` |
| `long[]` | `[J` |
| `boolean[]` | `[Z` |
| `char[]` | `[C` |
| `byte[]` | `[B` |
| `int[][]` | `[[I` |
| `String[]` | `[Ljava.lang.String;` |

Los objetos llevan `L` delante y `;` detrás. `long` es `J` y `boolean` es `Z` porque `L` y `B` ya estaban ocupados. **Esto vale la pena reconocerlo**, porque aparece constantemente en stack traces y en mensajes de `ClassCastException`, y saber leerlo ahorra tiempo.

**Segunda: heredan de `Object`.** Por eso tienen `toString()`, `equals()`, `hashCode()`, `getClass()`. Y por eso un `Object[]` puede contener arrays, y un método que recibe `Object` acepta un array.

**Tercera: implementan `Cloneable` y `Serializable`.** De ahí sale el método `clone()` de la [sección 10](#10-las-cuatro-formas-de-copiar-un-array).

**Cuarta, y la que causa todos los problemas: heredan las implementaciones de `Object` sin sobrescribir ninguna.** Un array no sabe imprimirse, ni compararse por contenido, ni calcular un hash de su contenido:

```java
int[] v = {1, 2, 3};
System.out.println(v);            // [I@1b6d3586   <- clase + hash de identidad
System.out.println(v.toString()); // lo mismo
```

Ese `[I@1b6d3586` es `getClass().getName()` + `@` + `Integer.toHexString(hashCode())`, que es literalmente lo que hace `Object.toString()`. **No dice absolutamente nada sobre el contenido.**

Esta única carencia es el origen de la mitad de las trampas del capítulo, y la razón entera de que exista `java.util.Arrays`: es la clase que aporta desde fuera todo lo que el array no trae de serie.

---

# Parte II — Memoria, referencias y copias

## 8. La variable y el objeto: dos cosas distintas

Para entender las copias hay que separar dos cosas que la sintaxis mezcla:

```java
int[] v = new int[3];
```

Aquí hay **dos entidades**:

1. **La variable `v`**, que guarda una *referencia* (una dirección). Vive donde vivan las variables de su ámbito: en el stack si es local, dentro del objeto si es un campo.
2. **El objeto array**, con sus tres casillas y su cabecera. Vive **siempre en el heap**, sin excepción.

```
   stack                      heap
 ┌────────┐            ┌──────────────────────┐
 │   v    │ ─────────▶ │ cabecera │ 0 │ 0 │ 0 │
 └────────┘            └──────────────────────┘
 la referencia            el objeto array
```

Esto vale también para los arrays de primitivos. `int[]` es un array **de** primitivos, pero el array en sí es un objeto: los `int` están en el heap, dentro de él. Un `int` suelto puede vivir en el stack; un `int[]` no.

La cabecera guarda, entre otras cosas, la longitud —que es lo que lee `arraylength`— y el tipo real del array, que es lo que hace posible detectar el `ArrayStoreException` de la [sección 38](#38-arraystoreexception).

De esta separación salen todas las consecuencias de las secciones siguientes. La regla en una frase: **cuando pasás un array a un método o lo asignás a otra variable, lo que viaja es la flecha, no la caja.**

## 9. Asignar un array no lo copia

Esta es la consecuencia inmediata, y la fuente del bug más frecuente de la Parte II:

```java
int[] orig = {1, 2, 3};
int[] alias = orig;      // NO copia: ahora las dos variables apuntan al mismo objeto

alias[0] = 99;
System.out.println(orig[0]);   // 99   <- el "original" cambió
```

```
 orig  ────┐
           ├──▶ [ 99 │ 2 │ 3 ]
 alias ────┘
```

No hay dos arrays. Hay uno, con dos nombres. Escribir por cualquiera de los dos afecta a lo que se lee por el otro.

Lo mismo pasa al pasar un array a un método. **Java pasa siempre por valor, pero el valor de una variable de array es la referencia**, así que el método recibe una copia de la flecha, que sigue apuntando al mismo objeto:

```java
static void ensuciar(int[] datos) {
    datos[0] = 999;           // modifica el array del llamador
}

static void noHaceNada(int[] datos) {
    datos = new int[]{7, 7};  // reasigna la variable local; el llamador no se entera
}

int[] v = {1, 2, 3};
ensuciar(v);
System.out.println(v[0]);   // 999

noHaceNada(v);
System.out.println(Arrays.toString(v));   // [999, 2, 3]  <- sin cambios
```

La distinción es clave y se pregunta en entrevistas constantemente: **el método puede modificar el contenido del array, pero no puede hacer que la variable del llamador apunte a otro array.**

**Dónde muerde en producción.** Un getter que devuelve un array interno regala el control del estado del objeto:

```java
public class Configuracion {
    private final int[] puertos = {8080, 8443};

    public int[] getPuertos() { return puertos; }   // MAL
}

Configuracion c = new Configuracion();
c.getPuertos()[0] = 1;    // acabamos de modificar el interior del objeto
```

El `final` no protege nada: impide reasignar la variable `puertos`, no modificar el array al que apunta. La defensa es devolver una **copia defensiva**:

```java
public int[] getPuertos() { return puertos.clone(); }
```

Y lo simétrico en la entrada: un constructor que guarda el array que le pasan queda atado a un objeto que el llamador todavía controla.

```java
public Configuracion(int[] puertos) {
    this.puertos = puertos.clone();   // copia al entrar
}
```

Esta pareja —copiar al entrar, copiar al salir— es el patrón estándar para exponer arrays sin perder el control, y la razón principal por la que hoy se prefiere devolver `List.copyOf(...)`, que no necesita copias porque no se puede modificar.

## 10. Las cuatro formas de copiar un array

Copiar de verdad significa crear un objeto nuevo con el mismo contenido. Hay cuatro herramientas y cada una tiene su hueco.

**1. `clone()`** — la más corta cuando querés una copia idéntica.

```java
int[] orig = {1, 2, 3};
int[] copia = orig.clone();

copia[0] = 99;
System.out.println(orig[0]);   // 1   <- intacto
```

Devuelve un array del mismo tipo y la misma longitud. No hace falta castear con arrays de primitivos.

**2. `Arrays.copyOf(original, nuevaLongitud)`** — cuando querés cambiar el tamaño al copiar.

```java
int[] orig = {1, 2, 3};

System.out.println(Arrays.toString(Arrays.copyOf(orig, 5)));   // [1, 2, 3, 0, 0]
System.out.println(Arrays.toString(Arrays.copyOf(orig, 2)));   // [1, 2]
```

Si pedís más, **rellena con el valor por defecto**; si pedís menos, trunca. Con objetos rellena con `null`:

```java
System.out.println(Arrays.toString(Arrays.copyOf(new String[]{"a"}, 3)));
// [a, null, null]
```

Este método es el que usa `ArrayList` internamente para crecer.

**3. `Arrays.copyOfRange(original, desde, hasta)`** — para copiar un trozo. `desde` es inclusivo y `hasta` exclusivo, como en todo Java.

```java
int[] orig = {1, 2, 3};
System.out.println(Arrays.toString(Arrays.copyOfRange(orig, 1, 3)));   // [2, 3]
```

Con una particularidad útil: **`hasta` puede pasarse del final**, y rellena con los valores por defecto en vez de lanzar.

```java
System.out.println(Arrays.toString(Arrays.copyOfRange(orig, 1, 6)));   // [2, 3, 0, 0, 0]
```

**4. `System.arraycopy(src, posSrc, dst, posDst, cantidad)`** — la de más bajo nivel, y la única que copia **dentro de un array existente**.

```java
int[] src = {1, 2, 3, 4, 5};
int[] dst = new int[5];
System.arraycopy(src, 1, dst, 0, 3);
System.out.println(Arrays.toString(dst));   // [2, 3, 4, 0, 0]
```

Es la que hay que usar cuando el destino ya existe y solo querés sobrescribir una parte. Los otros tres métodos siempre crean un objeto nuevo. De hecho, `clone`, `copyOf` y `copyOfRange` **llaman a `System.arraycopy` por dentro**: es el motor de todas las copias de Java, implementado como método nativo que el JIT traduce a la instrucción de copia de bloque del procesador.

Un detalle que casi nadie conoce: **maneja correctamente el solapamiento**, como el `memmove` de C.

```java
int[] v = {1, 2, 3, 4, 5};
System.arraycopy(v, 0, v, 1, 4);
System.out.println(Arrays.toString(v));   // [1, 1, 2, 3, 4]
```

Origen y destino son el mismo array y las regiones se pisan, y aun así el resultado es el correcto (desplazar todo un lugar a la derecha). Un bucle `for` escrito a la ligera produciría `[1,1,1,1,1]`. Es la forma correcta de insertar o eliminar elementos desplazando.

Sus errores son ruidosos, que es lo que se quiere:

```java
System.arraycopy(src, 0, dst, 0, 10);
// ArrayIndexOutOfBoundsException: arraycopy: last source index 10 out of bounds for int[5]

Object[] destino = new Integer[3];
System.arraycopy(new String[]{"a","b","c"}, 0, destino, 0, 3);
// ArrayStoreException: arraycopy: type mismatch: can not copy java.lang.String[] into java.lang.Integer[]
```

**La tabla de decisión:**

| Necesito | Uso |
|---|---|
| Copia idéntica | `original.clone()` |
| Copia con otro tamaño | `Arrays.copyOf` |
| Copiar un trozo | `Arrays.copyOfRange` |
| Copiar dentro de un array que ya existe | `System.arraycopy` |
| Desplazar elementos dentro del mismo array | `System.arraycopy` |

## 11. Copia superficial frente a copia profunda

Las cuatro formas de la sección anterior tienen algo en común que hay que entender bien: **copian los elementos tal cual**. Si los elementos son primitivos, eso es una copia completa. Si son referencias, se copian **las referencias**, no los objetos apuntados.

Eso se llama **copia superficial** (*shallow copy*), y con arrays de una dimensión de primitivos no supone ningún problema. Con arrays de objetos, y sobre todo con arrays de arrays, sí:

```java
int[][] anidado = {{1, 2}, {3, 4}};
int[][] clon = anidado.clone();

clon[0][0] = 99;
System.out.println(anidado[0][0]);         // 99    <- ¡el "original" cambió!
System.out.println(anidado[0] == clon[0]); // true  <- comparten la fila
```

`clone()` sobre un `int[][]` copia el array **externo**, que contiene dos referencias a las filas. Las filas siguen siendo las mismas:

```
anidado ──▶ [ ref0 │ ref1 ]
                │      │
                ▼      ▼
             [99│2]  [3│4]     <- compartidas
                ▲      ▲
                │      │
clon    ──▶ [ ref0 │ ref1 ]
```

**Esto convierte la copia defensiva de la sección 9 en una falsa sensación de seguridad.** Un `getMatriz() { return matriz.clone(); }` no protege nada: el llamador puede escribir en cualquier celda.

La copia profunda hay que hacerla a mano, fila por fila:

```java
static int[][] copiaProfunda(int[][] original) {
    int[][] copia = new int[original.length][];
    for (int i = 0; i < original.length; i++) {
        copia[i] = original[i].clone();
    }
    return copia;
}
```

O con streams, que funciona igual para matrices rectangulares e irregulares:

```java
int[][] copia = Arrays.stream(original)
                      .map(int[]::clone)
                      .toArray(int[][]::new);
```

Y para arrays de objetos mutables, ni siquiera eso basta: clonar el array copia las referencias a los objetos, que siguen siendo compartidos. Una copia realmente profunda exige clonar también cada elemento, y eso depende de que cada clase sepa clonarse. **Es una de las razones fuertes para trabajar con objetos inmutables**: si los elementos no se pueden modificar, la copia superficial es suficiente y el problema desaparece.

## 12. La trampa de rellenar con objetos

`Arrays.fill(array, valor)` pone el mismo valor en todas las posiciones. Con primitivos es exactamente lo que parece. Con objetos hay una sutileza que muerde:

```java
StringBuilder[] sbs = new StringBuilder[3];
Arrays.fill(sbs, new StringBuilder("x"));

sbs[0].append("!");
System.out.println(Arrays.toString(sbs));   // [x!, x!, x!]
System.out.println(sbs[0] == sbs[1]);       // true
```

**Las tres posiciones apuntan al mismo objeto.** `new StringBuilder("x")` se evaluó **una vez**, antes de llamar a `fill`, y lo que `fill` repite es esa única referencia. Modificar "uno" modifica los tres, porque no hay tres.

La versión peligrosa de verdad es con matrices, porque parece que funciona:

```java
int[][] mal = new int[3][];
int[] fila = new int[2];
Arrays.fill(mal, fila);

mal[0][0] = 7;
System.out.println(Arrays.deepToString(mal));   // [[7, 0], [7, 0], [7, 0]]
```

Escribir en una celda modificó las tres filas. Comparado con la forma correcta:

```java
int[][] bien = new int[3][2];
bien[0][0] = 7;
System.out.println(Arrays.deepToString(bien));  // [[7, 0], [0, 0], [0, 0]]
```

`new int[3][2]` crea **tres filas independientes**; `fill` con una fila crea tres referencias a la misma.

**La regla general:** `Arrays.fill` solo es seguro con primitivos, con `null`, o con objetos **inmutables** (`String`, `Integer`, `LocalDate`, un `record` de campos inmutables). Con cualquier objeto mutable hay que usar `setAll`, que llama al generador una vez por posición:

```java
StringBuilder[] correcto = new StringBuilder[3];
Arrays.setAll(correcto, i -> new StringBuilder("x"));   // tres objetos distintos
```

Esta es exactamente la misma trampa que `Collections.nCopies`, y aparece siempre que una API "repite" un valor.

## 13. Comparar arrays: por qué el operador y el método fallan

De la [sección 7](#7-los-arrays-son-objetos-y-se-puede-comprobar) sabemos que los arrays no sobrescriben `equals` ni `hashCode`. Esta es la consecuencia:

```java
int[] a1 = {1, 2, 3};
int[] a2 = {1, 2, 3};

System.out.println(a1 == a2);                 // false
System.out.println(a1.equals(a2));            // false
System.out.println(Objects.equals(a1, a2));   // false
System.out.println(Arrays.equals(a1, a2));    // true   <- la única correcta
```

Las tres primeras comparan **identidad**, porque `equals` sin sobrescribir es `this == obj`. Y `Objects.equals` tampoco salva la situación, porque delega en ese mismo `equals` heredado. Solo `Arrays.equals` compara el contenido.

Lo mismo con el hash:

```java
System.out.println(a1.hashCode() == a2.hashCode());             // false
System.out.println(Arrays.hashCode(a1) == Arrays.hashCode(a2)); // true
```

**Y de ahí sale un bug muy caro:** un array nunca funciona como clave de un `HashMap` ni como elemento de un `HashSet` si lo que querés es igualdad por contenido.

```java
Set<int[]> conjunto = new HashSet<>();
conjunto.add(new int[]{1, 2, 3});
conjunto.add(new int[]{1, 2, 3});
System.out.println(conjunto.size());   // 2   <- no detectó el duplicado
```

Cada `new` produce un objeto con identidad y hash propios. El `Set` los ve como distintos y crece indefinidamente. Es el tipo de fuga que se descubre en producción con un mapa de millones de entradas que debería tener cientos.

Las salidas son tres: usar una `List<Integer>` (que sí implementa `equals` por contenido), usar un `record` que envuelva los datos, o —si el array es de bytes y hace falta rendimiento— envolverlo en una clase propia que implemente `equals` y `hashCode` con `Arrays.equals` y `Arrays.hashCode`.

**Con arrays anidados hay un nivel más de problema.** `Arrays.equals` compara los elementos con `equals`, y si los elementos son arrays, vuelve a comparar identidad:

```java
int[][] m1 = {{1, 2}, {3, 4}};
int[][] m2 = {{1, 2}, {3, 4}};

System.out.println(Arrays.equals(m1, m2));      // false
System.out.println(Arrays.deepEquals(m1, m2));  // true
```

`deepEquals` se llama a sí misma recursivamente cada vez que encuentra un array. Lo mismo con `Arrays.hashCode` frente a `Arrays.deepHashCode`. **Para cualquier cosa con más de una dimensión, la versión `deep` es la única correcta.**

Un aviso del javadoc que conviene conocer: `deepEquals` y `deepToString` sobre un array que se contiene a sí mismo entran en recursión infinita. Es una situación rara pero posible con `Object[]`.

## 14. El tamaño es fijo: qué significa exactamente

*"Una vez creado un array, su tamaño no puede cambiar"* es la frase que aparece en todos los tutoriales, y conviene entender qué implica de verdad, porque no significa que el array sea inmutable.

**Lo que no se puede cambiar es la longitud.** El contenido sí:

```java
int[] v = new int[3];
v[0] = 1;          // permitido: modificar contenido
v.length = 10;     // no compila: length no se puede asignar
```

Un array es **mutable en contenido e inmutable en tamaño**. Es una combinación poco habitual y explica por qué no hay "añadir" ni "eliminar".

**Lo que la gente escribe cuando necesita eso** es un desplazamiento manual. Insertar en una posición:

```java
static void insertar(int[] array, int posicion, int valor) {
    System.arraycopy(array, posicion, array, posicion + 1, array.length - posicion - 1);
    array[posicion] = valor;
}
```

Y eliminar:

```java
static void eliminar(int[] array, int posicion) {
    System.arraycopy(array, posicion + 1, array, posicion, array.length - posicion - 1);
    array[array.length - 1] = 0;   // la última queda "vacía"
}
```

**Ojo con lo que estas operaciones realmente hacen**, porque los tutoriales las presentan sin la advertencia: el array sigue teniendo la misma longitud. Insertar **descarta el último elemento** para hacer sitio; eliminar **duplica** el último salvo que lo limpies a mano, y por eso la última línea de `eliminar` no es opcional. No hay ninguna forma de que el array pase de 3 a 4 elementos.

Si querés que crezca de verdad, hay que crear uno nuevo:

```java
int[] mayor = Arrays.copyOf(v, v.length + 1);
mayor[v.length] = nuevoValor;
```

Y eso es exactamente lo que hace `ArrayList` por dentro, con una diferencia crucial: **no crece de uno en uno, crece un 50 %**. Crecer de uno en uno hace que añadir *n* elementos cueste O(n²) copias, mientras que crecer proporcionalmente lo deja en O(n) amortizado.

**La conclusión práctica**, y es probablemente la más importante del capítulo: si el número de elementos **no se conoce de antemano o va a cambiar**, un array es la estructura equivocada. Usá `ArrayList`. El array es la respuesta cuando el tamaño está fijado por el problema.

---

# Parte III — Recorrer

## 15. El for clásico

La forma completa de recorrer un array es el `for` de tres partes, controlando el índice a mano:

```java
String[] dias = {"lunes", "martes", "miércoles"};

for (int i = 0; i < dias.length; i++) {
    System.out.println(i + ": " + dias[i]);
}
// 0: lunes
// 1: martes
// 2: miércoles
```

Las tres partes, para no darlas por sabidas:

- **Inicialización** (`int i = 0`): se ejecuta una vez, antes de todo. El índice empieza en 0 porque el primer elemento está en 0.
- **Condición** (`i < dias.length`): se comprueba **antes** de cada vuelta. Con `<`, nunca con `<=`.
- **Actualización** (`i++`): se ejecuta al final de cada vuelta.

La variable `i` **solo existe dentro del bucle**, que es lo correcto y encaja con lo visto en [Variables and Scopes](04-variables-and-scopes.md).

Su bytecode confirma algo que conviene saber:

```
static int sumaFor(int[]);    // for (int i = 0; i < a.length; i++) s += a[i];
  Code:
       0: iconst_0
       1: istore_1          // s = 0
       2: iconst_0
       3: istore_2          // i = 0
       4: iload_2
       5: aload_0
       6: arraylength       // <-- vuelve a leer la longitud EN CADA VUELTA
       7: if_icmpge     22
      10: iload_1
      11: aload_0
      12: iload_2
      13: iaload            // leer a[i]
      14: iadd
      15: istore_1
      16: iinc          2, 1
      19: goto          4
      22: iload_1
      23: ireturn
```

`arraylength` está **dentro** del bucle, en el desplazamiento 6. En cada iteración se vuelve a consultar la longitud. Esto no es un problema en la práctica —`arraylength` es una lectura de cabecera y el JIT la saca del bucle cuando puede demostrar que el array no cambia—, pero explica un idiom que se ve en código antiguo:

```java
for (int i = 0, n = dias.length; i < n; i++) { }
```

Cachear la longitud a mano fue una optimización real en las JVM de los noventa. **Hoy no aporta nada** y empeora la legibilidad. No lo escribas; solo hay que saber reconocerlo cuando aparece.

**Cuándo el `for` clásico es la elección correcta:** siempre que necesites el índice, recorrer al revés, saltar elementos, avanzar de dos en dos, recorrer solo una parte, o **escribir** en el array.

## 16. El for-each y lo que revela su bytecode

El `for` mejorado (o *for-each*, o *enhanced for*, introducido en Java 5) elimina el índice cuando no hace falta:

```java
String[] dias = {"lunes", "martes", "miércoles"};

for (String dia : dias) {
    System.out.println(dia);
}
```

Se lee "para cada `dia` en `dias`". La variable `dia` toma el valor de cada elemento, uno por uno, en orden.

Comparado con la versión clásica, desaparecen las tres fuentes de error del bucle manual: no hay índice que se pueda inicializar mal, no hay condición que se pueda escribir con `<=`, y no hay actualización que se pueda olvidar. **Para el caso de "recorrer todo de principio a fin leyendo", es estrictamente mejor.**

Lo interesante es que **no existe en la JVM**. Es azúcar sintáctico puro, y el bytecode lo demuestra:

```
static int sumaForEach(int[]);    // for (int x : a) s += x;
  Code:
       0: iconst_0
       1: istore_1          // s = 0
       2: aload_0
       3: astore_2          // copia la referencia del array a un local oculto
       4: aload_2
       5: arraylength
       6: istore_3          // <-- la longitud se guarda UNA VEZ, fuera del bucle
       7: iconst_0
       8: istore        4   // índice oculto = 0
      10: iload         4
      12: iload_3
      13: if_icmpge     33
      16: aload_2
      17: iload         4
      19: iaload            // leer a[i]
      20: istore        5   // <-- aquí se asigna a la variable del for-each
      22: iload_1
      23: iload         5
      25: iadd
      26: istore_1
      27: iinc          4, 1
      30: goto          10
      33: iload_1
      34: ireturn
```

Tres conclusiones que valen la pena:

**Primera: el compilador genera un `for` clásico con un índice oculto.** No hay ninguna instrucción nueva. `for-each` sobre un array es exactamente el bucle indexado, escrito por el compilador en vez de por vos.

**Segunda: cachea la longitud.** `arraylength` está en el desplazamiento 5, **antes** del bucle, y se guarda en un local. El `for` clásico la releía cada vuelta. Es una diferencia real, aunque en la práctica el JIT iguale las dos versiones.

**Tercera, y la más importante: sobre arrays no se crea ningún `Iterator`.** Esto sorprende porque sobre colecciones sí: un `for (String s : lista)` compila a `lista.iterator()`, `hasNext()`, `next()`. Sobre arrays no hay objeto intermedio, no hay llamadas a métodos y no hay basura que recolectar. **`for-each` sobre un array es gratis**, y quien evita usarlo "por rendimiento" está optimizando algo que no existe.

**En la línea 20 está la clave del apartado siguiente:** `istore 5` copia el valor a la variable del bucle. La variable es una **copia**, no un alias de la casilla del array.

## 17. Lo que el for-each no puede hacer

Cuatro limitaciones, y la primera es la que más confusión causa.

**1. No se puede modificar el array.**

```java
int[] v = {1, 2, 3};

for (int x : v) {
    x = x * 2;             // modifica la copia local, no el array
}
System.out.println(Arrays.toString(v));   // [1, 2, 3]   <- sin cambios

for (int i = 0; i < v.length; i++) {
    v[i] = v[i] * 2;       // esto sí
}
System.out.println(Arrays.toString(v));   // [2, 4, 6]
```

Es exactamente el `istore` del bytecode: `x` es una variable nueva a la que se copió el valor. Asignarle algo no toca el array.

**El matiz importante con objetos**, que casi ningún tutorial explica: no podés **reemplazar** el elemento, pero sí podés **modificar el objeto** al que apunta:

```java
StringBuilder[] sbs = {new StringBuilder("a"), new StringBuilder("b")};

for (StringBuilder sb : sbs) {
    sb.append("!");           // SÍ afecta al array: muta el objeto compartido
}
System.out.println(Arrays.toString(sbs));   // [a!, b!]

for (StringBuilder sb : sbs) {
    sb = new StringBuilder("z");  // NO afecta: reasigna la variable local
}
System.out.println(Arrays.toString(sbs));   // [a!, b!]
```

La distinción es la misma de la [sección 9](#9-asignar-un-array-no-lo-copia): se puede modificar lo apuntado, no se puede cambiar a dónde apunta la casilla.

**2. No da acceso al índice.** Si necesitás saber en qué posición estás —para imprimir un número de línea, para comparar con el elemento anterior, para escribir en otro array en la misma posición—, el `for-each` no sirve. La solución de llevar un contador aparte funciona pero anula la ventaja:

```java
int i = 0;
for (String dia : dias) {
    System.out.println(i + ": " + dia);
    i++;                     // si necesitás esto, usá el for clásico
}
```

**3. No se pueden recorrer dos arrays a la vez.** El `for-each` avanza sobre uno solo. Para recorrer dos en paralelo hace falta el índice:

```java
for (int i = 0; i < nombres.length; i++) {
    System.out.println(nombres[i] + " tiene " + edades[i] + " años");
}
```

**4. No se puede controlar el avance.** No hay forma de ir al revés, saltar de dos en dos, empezar por el segundo o parar antes del último. (`break` y `continue` sí funcionan, eso no cambia.)

**La regla de decisión**, que se puede aplicar sin pensar:

| Situación | Bucle |
|---|---|
| Leer todos los elementos, de principio a fin | `for-each` |
| Necesito el índice | `for` clásico |
| Voy a escribir en el array | `for` clásico |
| Recorro dos arrays en paralelo | `for` clásico |
| Recorro al revés o salteado | `for` clásico |
| Solo transformo o filtro para producir otra cosa | stream ([sección 19](#19-streams-sobre-arrays)) |

## 18. Patrones de recorrido que conviene conocer

**Al revés.** Empezando por el último índice y bajando:

```java
for (int i = v.length - 1; i >= 0; i--) {
    System.out.println(v[i]);
}
```

Ojo con las dos diferencias respecto al bucle normal: empieza en `length - 1` (no en `length`, que ya está fuera) y la condición es `>= 0` (no `> 0`, que se saltaría el primer elemento). Los dos son errores de contorno habituales.

**Saltando elementos.**

```java
for (int i = 0; i < v.length; i += 2) { }   // uno de cada dos
```

**Solo un rango.**

```java
for (int i = 1; i < v.length - 1; i++) { }  // todos menos el primero y el último
```

**Con el elemento anterior.** Un patrón muy común para detectar cambios o calcular diferencias:

```java
for (int i = 1; i < v.length; i++) {
    int diferencia = v[i] - v[i - 1];
}
```

Empezar en 1 no es un descuido: es lo que evita el `v[-1]` de la primera vuelta.

**Buscar y salir en cuanto se encuentra.**

```java
static int indiceDe(int[] v, int buscado) {
    for (int i = 0; i < v.length; i++) {
        if (v[i] == buscado) return i;
    }
    return -1;    // convención: -1 significa "no está"
}
```

Devolver `-1` es la convención de toda la JDK (`String.indexOf`, `List.indexOf`) y conviene respetarla. La alternativa moderna es `OptionalInt`, que obliga al llamador a tratar el caso de ausencia.

**Máximo y mínimo.** El patrón de inicializar con el valor extremo opuesto:

```java
int maximo = Integer.MIN_VALUE;
for (int x : v) {
    if (x > maximo) maximo = x;
}
```

Con una advertencia que los tutoriales omiten: **si el array está vacío, esto devuelve `Integer.MIN_VALUE`**, que es un valor plausible y silenciosamente equivocado. La versión robusta valida primero:

```java
static int maximo(int[] v) {
    if (v.length == 0) throw new IllegalArgumentException("array vacío");
    int maximo = v[0];                       // arrancar con un elemento real
    for (int i = 1; i < v.length; i++) {
        if (v[i] > maximo) maximo = v[i];
    }
    return maximo;
}
```

Arrancar con `v[0]` en vez de con `MIN_VALUE` es mejor por dos razones: no depende de conocer el valor extremo del tipo, y funciona igual si mañana el array es de `double`, donde el mínimo no es `Double.MIN_VALUE` (que es positivo) sino `-Double.MAX_VALUE`. Ese es, de hecho, un bug clásico: `Double.MIN_VALUE` es el **positivo más pequeño**, no el más negativo.

Y la versión que no hay que escribir a mano:

```java
int maximo = Arrays.stream(v).max().orElseThrow();
```

## 19. Streams sobre arrays

Desde Java 8, un array se puede convertir en un stream y aprovechar toda la API funcional. Es la forma más expresiva de transformar, filtrar y agregar.

```java
int[] ns = {5, 3, 8, 1};

System.out.println(Arrays.stream(ns).sum());                 // 17
System.out.println(Arrays.stream(ns).max().getAsInt());      // 8
System.out.println(Arrays.stream(ns).average().getAsDouble()); // 4.25
System.out.println(Arrays.toString(Arrays.stream(ns).sorted().toArray()));  // [1, 3, 5, 8]
System.out.println(Arrays.stream(ns).summaryStatistics());
// IntSummaryStatistics{count=4, sum=17, min=1, average=4,250000, max=8}
```

`summaryStatistics()` calcula las cinco métricas en **una sola pasada**, lo que la hace mejor que llamar a `sum()`, `max()`, `min()` y `average()` por separado.

**El detalle de tipos que hay que entender.** `Arrays.stream` está sobrecargado y devuelve cosas distintas:

| Argumento | Devuelve |
|---|---|
| `int[]` | `IntStream` |
| `long[]` | `LongStream` |
| `double[]` | `DoubleStream` |
| `T[]` (objetos) | `Stream<T>` |

Los streams primitivos (`IntStream` y compañía) tienen métodos que `Stream<T>` no tiene (`sum`, `average`, `summaryStatistics`) y **no hacen boxing**, así que son más rápidos. Para pasar de uno a otro:

```java
List<Integer> lista = Arrays.stream(ns).boxed().toList();       // IntStream -> Stream<Integer>
int[] devuelta = lista.stream().mapToInt(Integer::intValue).toArray();
```

**La trampa de `Stream.of`.** Existe una segunda forma de crear un stream desde un array, y con primitivos hace algo completamente distinto:

```java
String[] palabras = {"uno", "dos", "tres"};
System.out.println(Stream.of(palabras).count());   // 3   <- correcto

int[] ns = {5, 3, 8, 1};
System.out.println(Stream.of(ns).count());         // 1   <- ¡uno!
System.out.println(Stream.of(ns).findFirst().get().getClass().getName());   // [I
```

`Stream.of` recibe varargs de tipo `T`. Con un `String[]` funciona: `T` es `String` y el array se despliega en tres elementos. Con un `int[]`, como los primitivos no pueden ser un `T`, Java infiere `T = int[]` y crea un stream de **un solo elemento**, que es el array entero.

Compila sin ningún aviso y el error aparece mucho después, cuando un `count()` devuelve 1 o un `forEach` imprime `[I@...`. **Para arrays de primitivos hay que usar `Arrays.stream` o `IntStream.of`, nunca `Stream.of`.** Es la misma trampa que `Arrays.asList` de la [sección 35](#35-arraysaslist-y-sus-tres-trampas), y por la misma razón.

**Streams sobre un rango.** Se puede limitar el stream a una parte del array:

```java
Stream<String> parcial = Arrays.stream(palabras, 1, 3);   // solo "dos" y "tres"
```

El primer índice es inclusivo y el segundo exclusivo. Si están al revés, falla:

```java
Arrays.stream(palabras, 2, 1);
// ArrayIndexOutOfBoundsException: origin(2) > fence(1)
```

**Cuándo usar streams y cuándo no.** Para transformar, filtrar y agregar son claramente mejores: expresan la intención y evitan bucles con acumuladores. Para recorridos simples con efectos secundarios, un `for-each` se lee mejor que un `.forEach(...)`. Y en código muy caliente, un bucle sobre un array de primitivos sigue siendo más rápido que un stream, porque el stream monta una tubería de objetos. La regla razonable es **legibilidad por defecto, bucle cuando la medición lo justifique**.

---

# Parte IV — Arrays multidimensionales

## 20. Java no tiene matrices: tiene arrays de arrays

Esta distinción parece pedante y no lo es: explica todo lo demás de esta parte.

En lenguajes como C o Fortran, una matriz de 3×4 es **un solo bloque de memoria** de 12 celdas contiguas. En Java no existe eso. Un `int[][]` es **un array cuyos elementos son referencias a otros arrays**:

```java
int[][] matriz = new int[3][4];
```

```
matriz ──▶ [ ref0 │ ref1 │ ref2 ]        <- un int[][] de longitud 3
              │      │      │
              ▼      ▼      ▼
         [0│0│0│0] [0│0│0│0] [0│0│0│0]   <- tres int[] de longitud 4
```

Hay **cuatro objetos**, no uno: el array externo y las tres filas. Están en el heap, y nada garantiza que estén cerca unos de otros.

Se comprueba directamente:

```java
int[][] m = new int[3][4];

System.out.println(m.getClass().getName());     // [[I    <- array de array de int
System.out.println(m[0].getClass().getName());  // [I     <- una fila es un int[] normal
System.out.println(m.length);                   // 3      <- número de filas
System.out.println(m[0].length);                // 4      <- longitud de la primera fila
```

`m.length` es el número de **filas**, y `m[0].length` es la longitud de **esa fila concreta**. No hay ninguna propiedad que dé "el número de columnas", porque en general no existe tal cosa: cada fila puede tener la suya.

De esta estructura salen tres consecuencias que se desarrollan en el resto de la parte:

1. Las filas pueden tener **longitudes distintas** ([sección 22](#22-jagged-arrays-filas-de-distinta-longitud)).
2. Las filas son objetos independientes que se pueden **reasignar, compartir o dejar en `null`**.
3. Recorrer una matriz **no** es recorrer memoria contigua, y eso tiene un coste medible ([sección 24](#24-localidad-de-caché-por-filas-frente-a-por-columnas)).

## 21. Crear una matriz y lo que hace el bytecode

Hay tres formas, y la diferencia entre ellas es visible en el bytecode.

**Forma 1: todas las dimensiones.**

```java
int[][] m = new int[3][4];
```

Crea el array externo **y** las tres filas, todas de longitud 4, todas inicializadas a cero. El bytecode usa una instrucción dedicada:

```
static int[][] crearMatriz();     // new int[3][4]
  Code:
       0: iconst_3
       1: iconst_4
       2: multianewarray #9,  2    // <-- crea las dos dimensiones de golpe
       6: areturn
```

`multianewarray` con el argumento `2` significa "dos dimensiones". La JVM crea los cuatro objetos en una sola operación.

**Forma 2: solo la primera dimensión.**

```java
int[][] m = new int[3][];
```

Crea **solo el array externo**, con las tres filas en `null`:

```java
System.out.println(Arrays.toString(new int[3][]));   // [null, null, null]
```

Y el bytecode delata que aquí no hay nada especial:

```
static int[][] crearJagged();     // new int[3][]
  Code:
       0: iconst_3
       1: anewarray     #11       // class "[I"   <-- un array de referencias a int[]
       4: areturn
```

Es un `anewarray` corriente, el mismo que para `new String[3]`. **La prueba definitiva de que un `int[][]` no es más que un array de objetos cuyo tipo resulta ser `int[]`.**

Usar esta forma sin rellenar las filas produce un NPE muy característico:

```java
int[][] m = new int[2][];
int x = m[0][0];
// NullPointerException: Cannot load from int array because "<local5>[0]" is null
```

**Forma 3: literal.**

```java
int[][] m = {{1, 4, 2}, {3, 6, 8}};
System.out.println(Arrays.deepToString(m));   // [[1, 4, 2], [3, 6, 8]]
```

La más legible cuando los datos se conocen. Y la que hace evidente que cada `{...}` interno es un array propio.

**Más de dos dimensiones** funciona igual, aunque rara vez tiene sentido pasar de tres:

```java
int[][][] cubo = new int[2][3][4];   // 2 planos, 3 filas, 4 columnas
System.out.println(cubo.length);        // 2
System.out.println(cubo[0].length);     // 3
System.out.println(cubo[0][0].length);  // 4
```

Si te encontrás declarando un `int[][][][]`, casi siempre lo que falta es una clase que dé nombre a esas dimensiones.

## 22. Jagged arrays: filas de distinta longitud

Como cada fila es un array independiente, **no tienen por qué medir lo mismo**. Un array así se llama *jagged* (dentado) o irregular:

```java
int[][] triangulo = new int[4][];
for (int i = 0; i < 4; i++) {
    triangulo[i] = new int[i + 1];
}
System.out.println(Arrays.deepToString(triangulo));
// [[0], [0, 0], [0, 0, 0], [0, 0, 0, 0]]
```

O directamente con un literal:

```java
int[][] irregular = {
    {1},
    {2, 3},
    {4, 5, 6}
};
```

Esto **no se puede hacer en C**, y es una consecuencia directa del modelo de arrays de arrays.

**Dónde se usa de verdad:**

- Matrices triangulares (distancias entre pares, matrices de adyacencia simétricas): ahorra la mitad de la memoria.
- Listas de adyacencia en grafos: cada nodo tiene su número de vecinos.
- Agrupar datos por categoría cuando cada grupo tiene un tamaño distinto.
- El resultado de partir un texto en líneas y cada línea en campos.

**Lo que hay que tener presente al recorrerlas** es que `m[0].length` **no** vale para todas las filas. Este bucle revienta:

```java
// MAL: asume que todas las filas miden lo mismo que la primera
for (int i = 0; i < m.length; i++) {
    for (int j = 0; j < m[0].length; j++) {   // <-- m[0], fijo
        System.out.println(m[i][j]);
    }
}
```

Con un `triangulo` como el de arriba, la primera fila mide 1 y las demás más, así que se saltaría datos; y si la primera fuera la más larga, daría `ArrayIndexOutOfBoundsException`. La forma correcta consulta la longitud **de cada fila**:

```java
// BIEN
for (int i = 0; i < m.length; i++) {
    for (int j = 0; j < m[i].length; j++) {   // <-- m[i]
        System.out.println(m[i][j]);
    }
}
```

Escribir `m[i].length` en vez de `m[0].length` es correcto siempre, también para matrices rectangulares. **No hay ninguna razón para escribir la versión frágil.**

Y como las filas son referencias, pueden compartirse o quedar en `null`:

```java
int[][] m = new int[3][];
m[0] = new int[]{1, 2};
m[1] = m[0];        // dos filas, un solo array: modificar una modifica las dos
// m[2] sigue en null
```

Es la misma trampa de la [sección 12](#12-la-trampa-de-rellenar-con-objetos), y por eso un recorrido robusto de una matriz que no construiste vos debería comprobar `if (m[i] == null) continue;`.

## 23. Recorrer en dos dimensiones

El patrón estándar son dos bucles anidados: el externo sobre las filas, el interno sobre las columnas de esa fila.

```java
int[][] m = {{1, 4, 2}, {3, 6, 8}};

for (int i = 0; i < m.length; i++) {
    for (int j = 0; j < m[i].length; j++) {
        System.out.print(m[i][j] + " ");
    }
    System.out.println();
}
// 1 4 2
// 3 6 8
```

La convención de nombres `i` para filas y `j` para columnas viene de la notación matemática y conviene respetarla: cualquiera que lea el código espera ese orden.

Con `for-each` también funciona, y se lee bastante mejor cuando no hacen falta los índices:

```java
for (int[] fila : m) {
    for (int celda : fila) {
        System.out.print(celda + " ");
    }
    System.out.println();
}
```

Fijate en el tipo de la variable del bucle externo: **`int[]`, no `int`**. Cada elemento de un `int[][]` es una fila entera. Es un error de principiante muy común escribir `for (int fila : m)`, que no compila.

Y con streams, para agregar sobre toda la matriz:

```java
int suma = Arrays.stream(m).flatMapToInt(Arrays::stream).sum();
System.out.println(suma);   // 24
```

`flatMapToInt` aplana las filas en un único `IntStream`. Es el idiom estándar para "hacer algo con todas las celdas".

**Imprimir una matriz** tiene su propia trampa, que es la de la [sección 26](#26-imprimir-tostring-y-deeptostring):

```java
System.out.println(Arrays.toString(m));       // [[I@1b6d3586, [I@4554617c]
System.out.println(Arrays.deepToString(m));   // [[1, 4, 2], [3, 6, 8]]
```

`Arrays.toString` sobre una matriz imprime las **referencias** de las filas, porque cada fila es un objeto y `toString` no baja de nivel. Para cualquier cosa anidada, `deepToString`.

## 24. Localidad de caché: por filas frente a por columnas

Este apartado es lo que separa a un perfil mid de uno junior en una entrevista, y tiene una explicación física concreta.

Los dos bucles siguientes recorren exactamente las mismas celdas y calculan exactamente lo mismo. Solo cambia el orden:

```java
// Por filas: j varía en el bucle interno
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        suma += m[i][j];

// Por columnas: i varía en el bucle interno
for (int j = 0; j < n; j++)
    for (int i = 0; i < n; i++)
        suma += m[i][j];
```

Medido en JDK 25 sobre un `int[4000][4000]` (64 MB de datos):

```
por filas     -> 38 ms
por columnas  -> 49 ms
```

Casi un 30 % más lento, con el mismo número de operaciones. **La causa es la caché del procesador.**

Cuando la CPU lee una posición de memoria no trae ese dato solo: trae una **línea de caché** entera, típicamente 64 bytes, que son 16 `int`. Recorriendo **por filas**, los elementos consecutivos están pegados dentro de la misma fila, así que una sola lectura de memoria sirve para las 16 iteraciones siguientes. Recorriendo **por columnas**, cada acceso salta a otra fila —otro objeto, en otra zona del heap—, así que casi cada lectura desperdicia 60 de los 64 bytes traídos.

**La regla, que vale para cualquier lenguaje con memoria por filas:** el índice que varía más rápido debe ser el **último**. En Java, eso significa que `j` (la columna) va en el bucle interno.

Dos matices honestos sobre esta medición, porque el número que se suele citar es mucho mayor:

- **En Java la diferencia es menor que en C.** En C una matriz es un bloque contiguo y el salto de columna es perfectamente predecible; en Java cada fila es un objeto separado, así que ni siquiera el recorrido por filas es del todo contiguo entre filas. El modelo de arrays de arrays cuesta algo incluso en el caso bueno.
- **La medición es propia, con calentamiento pero sin JMH**, así que sirve como orden de magnitud, no como benchmark riguroso. Lo sólido es la dirección del efecto, no el 29 % exacto.

**Cuándo importa esto de verdad:** procesamiento de imágenes, multiplicación de matrices, simulaciones numéricas, cualquier bucle sobre millones de celdas. En una matriz de 10×10 no se nota nada y no vale la pena pensarlo.

Y si el rendimiento importa mucho, la solución que usan las bibliotecas numéricas serias es **aplanar la matriz en un solo array**:

```java
int[] plano = new int[filas * columnas];
// acceso: plano[i * columnas + j]
```

Eso recupera la contigüidad real de C: un solo objeto, sin indirecciones, sin cabeceras de fila. Es más incómodo de escribir y por eso solo se hace donde se ha medido que hace falta.

---

# Parte V — La clase java.util.Arrays

## 25. Qué es Arrays y cómo está organizada

`java.util.Arrays` es una clase de utilidades que existe desde Java 1.2. No se instancia (su constructor es privado) y todos sus métodos son `static`. Su razón de ser es la de la [sección 7](#7-los-arrays-son-objetos-y-se-puede-comprobar): **aportar desde fuera todo lo que un array no trae de serie.**

```java
import java.util.Arrays;
```

Casi todos sus métodos están **sobrecargados una vez por cada tipo primitivo más una versión para objetos**. Es decir, `Arrays.sort` no es un método: son nueve. Esto es una consecuencia directa de que los genéricos no funcionan con primitivos, y explica por qué la clase tiene más de 160 métodos para hacer una docena de cosas distintas.

Las familias son seis:

| Familia | Métodos |
|---|---|
| **Imprimir** | `toString`, `deepToString` |
| **Comparar** | `equals`, `deepEquals`, `hashCode`, `deepHashCode`, `compare`, `compareUnsigned`, `mismatch` |
| **Ordenar y buscar** | `sort`, `parallelSort`, `binarySearch` |
| **Crear y copiar** | `copyOf`, `copyOfRange`, `fill`, `setAll`, `setAll` paralelo |
| **Transformar** | `asList`, `stream`, `parallelPrefix` |
| **Otros** | `spliterator` |

Y hay un método que **no** está en esta clase pero que pertenece al mismo grupo: `System.arraycopy`, en `java.lang.System`. Está ahí por razones históricas, y es el que hace el trabajo real de casi todos los demás.

## 26. Imprimir: toString y deepToString

El problema y su solución:

```java
int[] v = {1, 2, 3};

System.out.println(v);                    // [I@1b6d3586
System.out.println(Arrays.toString(v));    // [1, 2, 3]
```

`Arrays.toString` produce los elementos entre corchetes separados por coma y espacio. Funciona con cualquier tipo de array de una dimensión.

Para arrays anidados hace falta la versión profunda:

```java
int[][] m = {{1, 2}, {3, 4}};

System.out.println(Arrays.toString(m));       // [[I@4554617c, [I@74a14482]
System.out.println(Arrays.deepToString(m));   // [[1, 2], [3, 4]]
```

`toString` llama al `toString()` de cada elemento, y el de un array es el heredado de `Object`. `deepToString` detecta que el elemento es un array y se llama a sí misma.

**Esto es lo primero que hay que aprender del capítulo en términos prácticos**, porque el `[I@...` en un log es una pérdida de tiempo garantizada. La regla:

- Array de una dimensión → `Arrays.toString`
- Cualquier cosa anidada → `Arrays.deepToString`
- Si no sabés cuál → `deepToString`, que también funciona con una dimensión de objetos

Un detalle sobre logging: pasar un array directamente a un logger tiene el mismo problema.

```java
log.info("datos: {}", datos);                       // datos: [I@1b6d3586
log.info("datos: {}", Arrays.toString(datos));      // datos: [1, 2, 3]
```

Con la salvedad de que SLF4J y Log4j **sí** despliegan un `Object[]` cuando lo reciben como argumentos variables, lo que produce comportamientos inconsistentes según el tipo del array. Envolver siempre en `Arrays.toString` evita la sorpresa.

## 27. Comparar: equals, deepEquals, compare y mismatch

Ya vimos `equals` y `deepEquals` en la [sección 13](#13-comparar-arrays-por-qué-el-operador-y-el-método-fallan). Java 9 añadió dos métodos más que resuelven preguntas que antes había que responder a mano.

**`compare` ordena dos arrays lexicográficamente**, igual que se ordenan palabras:

```java
int[] m1 = {1, 2, 3, 4};
int[] m2 = {1, 2, 9, 4};
int[] m3 = {1, 2, 3, 4};
int[] m4 = {1, 2, 3};

System.out.println(Arrays.compare(m1, m2));   // -1   m1 < m2 (3 < 9 en la posición 2)
System.out.println(Arrays.compare(m1, m3));   //  0   iguales
System.out.println(Arrays.compare(m1, m4));   //  1   m1 > m4 (m4 es un prefijo más corto)
```

El contrato es el de `Comparable`: negativo, cero o positivo. Compara elemento a elemento hasta encontrar una diferencia; si uno es prefijo del otro, el más corto va primero. Esto convierte a los arrays en algo ordenable sin escribir un comparador.

**`mismatch` devuelve el índice de la primera diferencia**, o `-1` si son iguales:

```java
System.out.println(Arrays.mismatch(m1, m2));   //  2   difieren en el índice 2
System.out.println(Arrays.mismatch(m1, m3));   // -1   no difieren
System.out.println(Arrays.mismatch(m1, m4));   //  3   m4 se acabó en el índice 3
```

Es enormemente útil para diagnosticar. Un test que falla comparando dos arrays grandes con `assertArrayEquals` dice "no son iguales"; `Arrays.mismatch` dice **dónde**:

```java
int diferencia = Arrays.mismatch(esperado, obtenido);
if (diferencia >= 0) {
    throw new AssertionError("difieren en el índice " + diferencia
        + ": esperado " + esperado[diferencia] + ", obtenido " + obtenido[diferencia]);
}
```

Ese mensaje ahorra mucho más tiempo que un "arrays first differed" pelado.

Ambos métodos tienen versiones que aceptan rangos y versiones `deep` no existen: para arrays anidados hay que recorrerlos a mano.

## 28. Ordenar: sort y sus dos algoritmos

```java
int[] ns = {5, 2, 1, 4, 8};
Arrays.sort(ns);
System.out.println(Arrays.toString(ns));   // [1, 2, 4, 5, 8]
```

Tres cosas que hay que saber antes de usarlo.

**1. Ordena en el sitio (*in place*).** No devuelve nada: **modifica el array que le pasás**. Si necesitás conservar el original, copiá primero:

```java
int[] ordenado = original.clone();
Arrays.sort(ordenado);
```

Es un error frecuente escribir `int[] x = Arrays.sort(v);`, que no compila porque `sort` devuelve `void`, y otro más sutil ordenar un array que otra parte del programa está usando.

**2. Se puede ordenar solo un rango.**

```java
int[] parcial = {9, 8, 7, 6, 5};
Arrays.sort(parcial, 1, 4);
System.out.println(Arrays.toString(parcial));   // [9, 6, 7, 8, 5]
```

Los índices 1 a 3 quedaron ordenados; el 0 y el 4 no se tocaron.

**3. Usa dos algoritmos distintos según el tipo, y esto sí importa.**

| Tipo de array | Algoritmo | Estable | Complejidad media |
|---|---|---|---|
| Primitivos (`int[]`, `double[]`…) | **Dual-Pivot Quicksort** | no | O(n log n) |
| Objetos (`String[]`, `Integer[]`…) | **TimSort** | **sí** | O(n log n) |

La elección no es arbitraria y tiene una lógica clara:

- Con **primitivos**, dos elementos iguales son **indistinguibles**: da igual cuál quede primero. Por eso se puede usar quicksort, que es más rápido y no necesita memoria extra, aunque no sea estable.
- Con **objetos**, dos elementos "iguales" según el comparador pueden ser objetos distintos, y el orden entre ellos sí puede importar. Por eso se usa TimSort, que es estable, a cambio de necesitar memoria auxiliar.

TimSort además está optimizado para datos **parcialmente ordenados**, que es el caso más común en la práctica: detecta tramos ya ordenados y los fusiona en vez de reordenarlos. Sobre datos casi ordenados baja a O(n).

**Sobre esto hay una contradicción entre dos artículos de Baeldung** que conviene señalar, porque es la clase de imprecisión que se copia: uno dice que los arrays de objetos usan *merge sort* y el otro dice *TimSort*. TimSort **es** un merge sort adaptativo, así que la primera afirmación no es exactamente falsa, pero pierde lo único que importa (la adaptatividad y la estabilidad) y contradice al otro artículo del mismo sitio. La respuesta correcta en una entrevista es TimSort.

**`parallelSort`** (Java 8) hace lo mismo repartiendo el trabajo en el *common ForkJoinPool*:

```java
Arrays.parallelSort(arrayGrande);
```

Solo compensa con arrays grandes —el umbral interno de la JDK está en 8192 elementos, por debajo del cual delega en `sort`— y en una máquina con varios núcleos libres. En un servidor cargado o dentro de una petición web puede ser **más lento**, porque compite por el mismo pool que usan los streams paralelos.

## 29. Ordenar con Comparator y el error que ya conocemos

Para objetos hay dos formas de decidir el orden:

**Orden natural**, si la clase implementa `Comparable`:

```java
String[] ss = {"pera", "Manzana", "uva"};
Arrays.sort(ss);
System.out.println(Arrays.toString(ss));   // [Manzana, pera, uva]
```

Fijate en que `Manzana` va primero: el orden natural de `String` es por punto de código, y las mayúsculas ASCII van antes que las minúsculas. Es un resultado que sorprende a los usuarios y por eso rara vez es lo que se quiere para texto visible.

**Orden a medida**, pasando un `Comparator`:

```java
Arrays.sort(ss, String.CASE_INSENSITIVE_ORDER);
Arrays.sort(personas, Comparator.comparing(Persona::apellido));
Arrays.sort(personas, Comparator.comparing(Persona::apellido)
                                .thenComparingInt(Persona::edad)
                                .reversed());
```

**Y aquí reaparece el error del capítulo anterior**, porque las fuentes de este tema lo repiten en sus ejemplos de ordenación: escribir el comparador como una resta.

```java
Arrays.sort(empleados, (e1, e2) -> e1.id() - e2.id());   // MAL
```

Verificado con tres empleados cuyos ids son `Integer.MAX_VALUE`, `Integer.MIN_VALUE` y `0`:

```
con resta        -> [caro, ana, beto]     <- desordenado
con comparingInt -> [beto, caro, ana]     <- correcto
```

La resta desborda y el comparador miente sobre el orden, sin lanzar ninguna excepción. Está explicado a fondo en [Logical, Relational and Bitwise Operators](08-logical-relational-bitwise-operators.md#11-ordenar-compareto-comparator-y-el-comparador-roto-por-resta). La forma correcta es siempre `Comparator.comparingInt`, que además evita el boxing.

**Un detalle sobre `null`.** El orden natural no los admite:

```java
String[] conNull = {"b", null, "a"};
Arrays.sort(conNull);
// NullPointerException
```

Y la solución es un comparador que los coloque explícitamente:

```java
Arrays.sort(conNull, Comparator.nullsFirst(Comparator.naturalOrder()));
System.out.println(Arrays.toString(conNull));   // [null, a, b]
```

`nullsFirst` y `nullsLast` existen precisamente para esto. Obligan a **decidir** dónde van los nulos en vez de descubrirlo con una excepción en producción.

## 30. Estabilidad del ordenamiento

Un algoritmo de ordenación es **estable** si los elementos que el comparador considera iguales conservan su orden relativo original.

```java
record Empleado(String nombre, int id) {}

Empleado[] est = {
    new Empleado("primero", 1),
    new Empleado("segundo", 1),
    new Empleado("tercero", 0)
};

Arrays.sort(est, Comparator.comparingInt(Empleado::id));
// [tercero, primero, segundo]
```

`primero` y `segundo` tienen el mismo `id`, y después de ordenar **siguen en ese orden**. Eso es la estabilidad, y es lo que garantiza TimSort para arrays de objetos.

**Por qué importa**, con el caso de uso que lo hace evidente: la estabilidad permite **ordenar por varios criterios encadenando ordenaciones**.

```java
Arrays.sort(empleados, Comparator.comparing(Empleado::nombre));   // primero por nombre
Arrays.sort(empleados, Comparator.comparingInt(Empleado::id));    // luego por id
// resultado: ordenado por id, y dentro de cada id, por nombre
```

Esto funciona **solo** porque la segunda ordenación no deshace la primera entre elementos de igual `id`. Con un algoritmo inestable el resultado sería arbitrario.

Dicho esto, encadenar `sort` es menos claro y más lento que construir un comparador compuesto, que es lo que se debe escribir:

```java
Arrays.sort(empleados, Comparator.comparingInt(Empleado::id)
                                 .thenComparing(Empleado::nombre));
```

**Y la advertencia:** los arrays de primitivos **no** están ordenados de forma estable, porque usan quicksort. Eso no es un problema —dos `int` iguales son intercambiables— pero conviene saberlo para no construir razonamientos sobre una garantía que no existe. La estabilidad es una propiedad de `sort` sobre `Object[]`, no de `sort` en general.

## 31. Buscar: binarySearch y su contrato

Buscar recorriendo es O(n). Sobre un array **ordenado** se puede hacer en O(log n) con búsqueda binaria, y la JDK la trae hecha:

```java
int[] ord = {0, 2, 4, 6, 8, 10};

System.out.println(Arrays.binarySearch(ord, 6));   // 3   <- el índice donde está
```

Lo interesante es qué devuelve cuando **no** encuentra el elemento:

```java
System.out.println(Arrays.binarySearch(ord, 7));    // -5
System.out.println(Arrays.binarySearch(ord, 12));   // -7
System.out.println(Arrays.binarySearch(ord, -1));   // -1
```

No devuelve simplemente `-1`. Devuelve un valor negativo que **codifica dónde habría que insertar el elemento** para mantener el orden. La fórmula exacta del javadoc es:

```
(-(punto de inserción) - 1)
```

Con `7`: el punto de inserción es 4 (entre el `6` y el `8`), así que devuelve `-(4) - 1 = -5`. Se recupera despejando:

```java
int r = Arrays.binarySearch(ord, 7);
if (r < 0) {
    int puntoDeInsercion = -(r) - 1;    // 4
}
```

**Por qué la fórmula lleva ese `-1`** es la parte que casi nadie explica y que hace que valga la pena entenderla: sin él, un punto de inserción 0 daría `-0`, que en aritmética entera es `0`, indistinguible de "encontrado en el índice 0". El `-1` desplaza todo el rango para que cualquier resultado negativo signifique inequívocamente "no encontrado".

Esa es exactamente la operación que necesita una estructura ordenada para insertar, y es la razón de que el método esté diseñado así en vez de devolver un simple booleano.

También acepta un rango y un comparador:

```java
Arrays.binarySearch(ord, 0, 4, 2);                        // buscar solo entre los índices 0 y 3
Arrays.binarySearch(nombres, "ana", String.CASE_INSENSITIVE_ORDER);
```

Con la regla de oro: **el comparador debe ser el mismo con el que se ordenó el array**. Si ordenaste con un criterio y buscás con otro, el resultado es basura sin ningún aviso.

## 32. binarySearch sin ordenar: el fallo silencioso

El javadoc lo dice con todas las letras: *"the array must be sorted... if it is not sorted, the results are undefined"*. Y "undefined" aquí significa exactamente lo peor: **no lanza excepción, devuelve un número equivocado**.

Verificado sobre un array desordenado:

```java
int[] desord = {5, 2, 8, 1, 9};

System.out.println(Arrays.binarySearch(desord, 8));   //  2   <- ¡correcto, por casualidad!
System.out.println(Arrays.binarySearch(desord, 2));   // -1   <- incorrecto: el 2 está en el índice 1
```

Este par de resultados es la mejor ilustración posible del problema. La primera búsqueda **acierta**, porque el algoritmo empezó por el elemento central y resultó ser el buscado. La segunda **falla** y afirma que el `2` no está, cuando está en el índice 1.

Es decir: el método funciona a veces. Y "funciona a veces" es la propiedad que hace que un bug sobreviva a los tests y llegue a producción, porque el caso de prueba que alguien escribió cae en la mitad afortunada.

**Las defensas:**

```java
// 1. Ordenar explícitamente justo antes, si el coste lo permite
Arrays.sort(datos);
int i = Arrays.binarySearch(datos, buscado);

// 2. Si el array ya debería estar ordenado, verificarlo en desarrollo
assert estaOrdenado(datos) : "el array debe estar ordenado";

// 3. Si no podés garantizar el orden, no uses búsqueda binaria
int i = indiceDe(datos, buscado);   // búsqueda lineal, siempre correcta
```

Y la reflexión de diseño: `binarySearch` sobre un array suelto es frágil porque **el orden es una precondición que el tipo no expresa**. Un `TreeSet` o un `TreeMap` garantizan el orden por construcción, y por eso son mejores cuando hay muchas búsquedas. Un array ordenado solo compensa cuando se ordena una vez y se busca muchísimas veces, y aun así conviene encapsularlo en una clase que impida modificarlo por fuera.

## 33. Rellenar: fill, setAll y parallelPrefix

**`fill`** pone el mismo valor en todas las posiciones, o en un rango:

```java
int[] v = new int[10];
Arrays.fill(v, 123);
System.out.println(Arrays.toString(v));   // [123, 123, ... , 123]

int[] v2 = new int[10];
Arrays.fill(v2, 3, 5, 123);
System.out.println(Arrays.toString(v2));  // [0, 0, 0, 123, 123, 0, 0, 0, 0, 0]
```

Con el aviso grande de la [sección 12](#12-la-trampa-de-rellenar-con-objetos): con objetos mutables, todas las posiciones comparten la misma instancia.

**`setAll`** (Java 8) rellena llamando a una función **una vez por índice**, lo que resuelve ese problema y además permite valores distintos:

```java
int[] cuadrados = new int[6];
Arrays.setAll(cuadrados, i -> i * i);
System.out.println(Arrays.toString(cuadrados));   // [0, 1, 4, 9, 16, 25]

StringBuilder[] sbs = new StringBuilder[3];
Arrays.setAll(sbs, i -> new StringBuilder("x"));  // tres objetos distintos
```

El parámetro de la lambda es **el índice**, no el valor actual. Es la forma idiomática de generar un array a partir de una fórmula.

Un detalle del javadoc que conviene conocer: si la lambda lanza una excepción a mitad, **el estado final del array queda sin definir**. Algunas posiciones estarán rellenas y otras no. Si eso importa, hay que construir en un array temporal y asignarlo al final.

**`parallelPrefix`** (Java 8) aplica una operación acumulativa:

```java
int[] acum = {1, 2, 3, 4};
Arrays.parallelPrefix(acum, Integer::sum);
System.out.println(Arrays.toString(acum));   // [1, 3, 6, 10]
```

Cada posición pasa a ser la acumulación de todas las anteriores: `1`, `1+2`, `1+2+3`, `1+2+3+4`. Es la operación de "suma prefija", útil para consultas de rango, histogramas acumulados y algoritmos de programación dinámica.

**La restricción es importante:** como se ejecuta en paralelo, la operación debe ser **asociativa** y sin efectos secundarios. Con una función no asociativa el resultado es inconsistente entre ejecuciones. Suma, producto, máximo y mínimo son asociativos; `(a, b) -> a + b * a` no lo es.

## 34. System.arraycopy: el motor que hay debajo

Ya lo vimos en la [sección 10](#10-las-cuatro-formas-de-copiar-un-array), pero merece un lugar propio porque es la operación sobre la que se construye todo lo demás.

```java
System.arraycopy(origen, posOrigen, destino, posDestino, cantidad);
```

Los cinco parámetros en orden: de dónde, desde qué índice, a dónde, a qué índice, cuántos elementos.

Es un **método nativo** con tratamiento especial del JIT: se compila a la instrucción de copia de bloque del procesador, que mueve muchos bytes por ciclo. Copiar un millón de `int` con `arraycopy` es órdenes de magnitud más rápido que con un bucle escrito a mano, porque el bucle procesa 4 bytes por iteración y la instrucción de bloque procesa 32 o 64.

Por eso lo usan por dentro `Arrays.copyOf`, `Arrays.copyOfRange`, `clone()`, `ArrayList.add` cuando crece, `StringBuilder.append`, `String.concat` y prácticamente cualquier operación de la JDK que mueva datos.

**Cuándo escribirlo directamente**, en vez de usar `copyOf`:

- Cuando el destino ya existe y solo querés sobrescribir una parte.
- Cuando estás desplazando dentro del mismo array (inserción, eliminación, buffers circulares).
- Cuando estás implementando una estructura de datos y controlás la memoria a mano.

En código de aplicación normal, `Arrays.copyOf` y `copyOfRange` son más legibles y hacen lo mismo. `arraycopy` es una herramienta de biblioteca.

## 35. Arrays.asList y sus tres trampas

`Arrays.asList` convierte un array en `List`, y es probablemente el método de esta clase que más bugs ha producido.

```java
Integer[] boxed = {1, 2, 3};
List<Integer> lista = Arrays.asList(boxed);
System.out.println(lista);   // [1, 2, 3]
```

Parece inofensivo. Tiene tres trampas.

**Trampa 1: la lista tiene tamaño fijo.**

```java
lista.add(4);      // UnsupportedOperationException
lista.remove(0);   // UnsupportedOperationException
```

La razón es que **no copia nada**: devuelve una vista sobre el array original. Y como el array no puede crecer, la lista tampoco. La clase que devuelve ni siquiera es `java.util.ArrayList`:

```java
System.out.println(lista.getClass().getName());   // java.util.Arrays$ArrayList
```

Es una clase **interna y privada de `Arrays`**, distinta de la que todo el mundo conoce, con el mismo nombre simple. Depurar esto sin saberlo es desesperante: el IDE muestra `ArrayList` y el `add` falla igualmente.

**Trampa 2: no es inmutable, es de tamaño fijo. Y escribe en el array.**

Esta es la que casi nadie conoce:

```java
Integer[] boxed = {1, 2, 3};
List<Integer> lista = Arrays.asList(boxed);

lista.set(0, 99);                          // sí funciona
System.out.println(lista);                 // [99, 2, 3]
System.out.println(Arrays.toString(boxed)); // [99, 2, 3]   <- ¡el array cambió!
```

`set` está permitido y **modifica el array subyacente**. Y al revés también: modificar el array cambia lo que ve la lista. Quien pase esa lista creyendo que es inmutable está regalando acceso de escritura a su array.

**Trampa 3: con primitivos hace algo completamente distinto, y compila.**

```java
int[] primitivos = {1, 2, 3};
List<int[]> raro = Arrays.asList(primitivos);

System.out.println(raro.size());          // 1     <- ¡uno!
System.out.println(raro.get(0).length);   // 3
```

`asList` recibe `T...`, y `T` no puede ser `int`. Java infiere `T = int[]` y devuelve una lista de **un solo elemento** que es el array entero. Es la misma trampa que `Stream.of` de la [sección 19](#19-streams-sobre-arrays).

Lo peor es que **compila sin ningún aviso**, y solo falla más tarde, cuando alguien recorre la lista esperando tres números. Nótese que si escribís `List<Integer> l = Arrays.asList(primitivos);` sí da error de compilación, así que el bug solo aparece con `var` o cuando el resultado se pasa directamente a otro método.

**Las alternativas correctas:**

| Quiero | Uso |
|---|---|
| Lista de solo lectura desde un array de objetos | `List.of(array)` |
| Copia inmutable | `List.copyOf(Arrays.asList(array))` |
| Lista mutable de verdad | `new ArrayList<>(Arrays.asList(array))` |
| Lista desde un array de primitivos | `Arrays.stream(array).boxed().toList()` |
| Vista de tamaño fijo, sabiendo lo que hacés | `Arrays.asList(array)` |

Y un apunte sobre `List.of`: es inmutable **de verdad** (`set` también lanza) y además **rechaza `null`**, mientras que `Arrays.asList` los acepta. Para código nuevo, `List.of` es casi siempre la respuesta.

## 36. Referencia completa de java.util.Arrays

Los métodos que hay que conocer, con la versión en que aparecieron. Todos existen sobrecargados para `byte`, `short`, `int`, `long`, `char`, `float`, `double`, `boolean` y `Object` salvo donde se indique.

| Método | Qué hace | Desde |
|---|---|---|
| `toString` | representación legible de una dimensión | 1.5 |
| `deepToString` | representación legible recursiva (solo `Object[]`) | 1.5 |
| `equals` | compara contenido de una dimensión | 1.2 |
| `deepEquals` | compara contenido recursivamente (solo `Object[]`) | 1.5 |
| `hashCode` | hash del contenido de una dimensión | 1.5 |
| `deepHashCode` | hash del contenido recursivo (solo `Object[]`) | 1.5 |
| `compare` | orden lexicográfico entre dos arrays | 9 |
| `compareUnsigned` | ídem tratando los valores como sin signo | 9 |
| `mismatch` | índice de la primera diferencia, o -1 | 9 |
| `sort` | ordena en el sitio | 1.2 |
| `parallelSort` | ordena en paralelo | 8 |
| `binarySearch` | busca en un array **ordenado** | 1.2 |
| `fill` | rellena con un valor | 1.2 |
| `setAll` | rellena con una función del índice | 8 |
| `parallelSetAll` | ídem en paralelo | 8 |
| `parallelPrefix` | acumulación en paralelo | 8 |
| `copyOf` | copia con nueva longitud | 6 |
| `copyOfRange` | copia de un rango | 6 |
| `asList` | vista como `List` (solo objetos) | 1.2 |
| `stream` | convierte en stream | 8 |
| `spliterator` | divisor para procesamiento paralelo | 8 |

Y fuera de la clase, pero del mismo grupo:

| Método | Dónde | Qué hace |
|---|---|---|
| `System.arraycopy` | `java.lang.System` | copia entre arrays existentes |
| `Array.getLength` | `java.lang.reflect.Array` | longitud de un array recibido como `Object` |
| `Array.newInstance` | `java.lang.reflect.Array` | crea un array de un tipo conocido en ejecución |

Las dos últimas son de reflexión y solo hacen falta cuando el tipo no se conoce en compilación —típicamente en frameworks—. `Array.newInstance` es, además, la única forma de sortear la limitación de la [sección 39](#39-por-qué-no-se-puede-crear-un-array-genérico).

---

# Parte VI — Arrays, genéricos y varargs

## 37. Covarianza: el agujero que Java dejó abierto

Esta sección explica una decisión de diseño de 1995 cuyas consecuencias siguen presentes en cada `String[]` que escribís.

**Los arrays de Java son covariantes.** Eso significa que si `Perro` es subtipo de `Animal`, entonces `Perro[]` es subtipo de `Animal[]`. Se puede asignar directamente:

```java
Object[] objetos = new String[3];   // compila sin ningún aviso
System.out.println(objetos.getClass().getName());   // [Ljava.lang.String;
```

La variable es de tipo `Object[]` pero el objeto sigue siendo un `String[]`, y lo sabe.

**Por qué Java hizo esto.** En 1995 no había genéricos. Sin covarianza, un método como este habría necesitado una sobrecarga por cada tipo de array del universo:

```java
static void imprimirTodos(Object[] elementos) {
    for (Object e : elementos) System.out.println(e);
}

imprimirTodos(new String[]{"a", "b"});     // funciona gracias a la covarianza
imprimirTodos(new Integer[]{1, 2});        // también
```

Era la única forma de escribir código genérico. La covarianza fue el precio que se pagó por poder escribir `Arrays.sort(Object[])` una sola vez.

**El problema** es que la asignación de tipos deja de ser segura. Si `objetos` es un `Object[]`, el compilador cree que puede guardar cualquier `Object` dentro. Pero el array real solo acepta `String`:

```java
Object[] objetos = new String[3];
objetos[0] = Integer.valueOf(42);
// ArrayStoreException: java.lang.Integer
```

Compila perfectamente y revienta en ejecución. **Es el único lugar de Java donde el compilador aprueba una asignación de tipos que la JVM luego rechaza.**

**Los genéricos, diseñados diez años después, tomaron la decisión contraria.** `List<String>` **no** es subtipo de `List<Object>`:

```java
List<Object> lista = new ArrayList<String>();   // no compila
```

Los genéricos son **invariantes**, precisamente para que el error se detecte en compilación. Cuando hace falta flexibilidad se usan wildcards (`List<? extends Object>`), que la dan de forma segura porque prohíben escribir.

La comparación resume el asunto:

| | Arrays | Genéricos |
|---|---|---|
| Relación de subtipos | covariante | invariante |
| Cuándo se detecta el error | ejecución (`ArrayStoreException`) | compilación |
| Información de tipo en ejecución | **sí**, el array la lleva | **no**, se borra (*type erasure*) |

Esa última fila es la que provoca el choque de la [sección 39](#39-por-qué-no-se-puede-crear-un-array-genérico): los arrays saben su tipo en ejecución y los genéricos no, así que no se pueden combinar.

**Un límite útil:** la covarianza **solo aplica a tipos de referencia**. Con primitivos no funciona:

```java
Object[] o = new int[3];
```

```
error: incompatible types: int[] cannot be converted to Object[]
```

Un `int[]` es un `Object` (se puede asignar a una variable `Object`), pero **no** es un `Object[]`, porque `int` no es subtipo de `Object`. Es una distinción que confunde y que conviene tener clara: `int[]` hereda de `Object`, no de `Object[]`.

## 38. ArrayStoreException

Es la excepción que materializa el problema de la sección anterior:

```java
Object[] objetos = new String[3];
objetos[0] = Integer.valueOf(42);
// ArrayStoreException: java.lang.Integer

Number[] nums = new Integer[2];
nums[0] = Double.valueOf(1.5);
// ArrayStoreException: java.lang.Double
```

El mensaje es escueto: solo el nombre de la clase que se intentó guardar. No dice cuál era el tipo esperado, lo que la hace más incómoda de diagnosticar de lo necesario.

**Cómo funciona por dentro.** Cada array guarda en su cabecera el tipo real de sus elementos. En cada escritura sobre un array de referencias, la JVM comprueba que el valor sea compatible con ese tipo. Es una comprobación en **cada asignación**, no solo al crearlo.

Eso tiene una consecuencia de rendimiento poco conocida: escribir en un `Object[]` es intrínsecamente más caro que escribir en un `int[]`, porque además del *bounds check* hay un *store check*. En bucles muy calientes sobre arrays de objetos, esa comprobación es medible. Es una de las razones por las que los arrays de primitivos son más rápidos, además del boxing.

**Dónde aparece en la práctica**, más allá del ejemplo de manual:

```java
// 1. Al pasar un array a un método que lo declara más general
static void rellenar(Object[] destino) {
    destino[0] = "texto";     // explota si le pasaron un Integer[]
}
rellenar(new Integer[3]);     // compila, revienta en ejecución

// 2. Con System.arraycopy entre tipos incompatibles
Object[] destino = new Integer[3];
System.arraycopy(new String[]{"a","b","c"}, 0, destino, 0, 3);
// ArrayStoreException: arraycopy: type mismatch: can not copy java.lang.String[] into java.lang.Integer[]

// 3. Con el toArray de una colección mal tipado
```

**La defensa** es de diseño, no de código: **no declares parámetros ni variables como `Object[]` o como un supertipo del array real** salvo que solo vayas a leer. Si un método necesita escribir en un array, debe declararlo con el tipo exacto. Y si necesita ser genérico y escribir, la respuesta correcta es una `List<T>`, no un array.

## 39. Por qué no se puede crear un array genérico

Esto no compila, y es una de las primeras cosas con las que uno choca al escribir código genérico:

```java
public class Caja<T> {
    T[] crear() { return new T[10]; }
}
```

```
error: generic array creation
    T[] crear() { return new T[10]; }
                         ^
```

**La razón es el choque de la tabla de la sección 37**, y merece la pena entenderla en vez de memorizar la restricción.

Un array **necesita saber su tipo en ejecución**: lo guarda en la cabecera y lo usa para las comprobaciones de `ArrayStoreException`. Un genérico **no tiene tipo en ejecución**: `T` se borra durante la compilación (*type erasure*) y se convierte en `Object`. Así que `new T[10]` le pide a la JVM que cree un array de un tipo que, en el momento de crearlo, ya no existe. No hay nada que poner en la cabecera.

Si el compilador lo permitiera creando un `Object[]`, la garantía se rompería:

```java
// Si esto compilara...
T[] array = (T[]) new Object[10];     // compila, con warning
String[] strings = (String[]) array;  // ClassCastException en ejecución
```

**Las tres soluciones**, en orden de preferencia:

**1. Usar una `List<T>`.** Es la respuesta correcta el 95 % de las veces.

```java
public class Caja<T> {
    private final List<T> elementos = new ArrayList<>();
}
```

**2. Guardar un `Object[]` y castear al leer.** Es lo que hace `ArrayList` internamente:

```java
public class MiLista<T> {
    private Object[] datos = new Object[10];

    @SuppressWarnings("unchecked")
    public T get(int i) {
        return (T) datos[i];    // cast sin comprobación, pero seguro por construcción
    }
}
```

El `@SuppressWarnings("unchecked")` está justificado aquí porque la clase controla qué entra en `datos`. **Solo es legítimo cuando podés demostrar la invariante**; si el array es accesible desde fuera, no.

**3. Pasar el tipo en ejecución y usar reflexión.** Es lo que hace `Arrays.copyOf`:

```java
@SuppressWarnings("unchecked")
static <T> T[] crear(Class<T> tipo, int tamano) {
    return (T[]) java.lang.reflect.Array.newInstance(tipo, tamano);
}

String[] ss = crear(String.class, 5);
System.out.println(ss.getClass().getName());   // [Ljava.lang.String;
```

Aquí sí hay un tipo real en ejecución, porque se lo pasaste explícitamente en el `Class<T>`. Es la técnica de las bibliotecas que tienen que devolver arrays tipados.

**Un caso relacionado que sí compila, con aviso:**

```java
List<String>[] ls = new List[3];
```

```
warning: [unchecked] unchecked conversion
  required: List<String>[]
  found:    List[]
```

Crear un array de un tipo *raw* y asignarlo a uno genérico compila con warning. Funciona en la práctica pero pierde la comprobación de tipos, y por eso *Effective Java* recomienda directamente **preferir listas a arrays** en cualquier código genérico. Es la conclusión práctica de toda esta sección.

## 40. toArray y sus tres formas

Convertir una colección en array es una operación cotidiana, y hay tres formas con comportamientos distintos.

**Forma 1: `toArray()` sin argumentos.** Devuelve siempre `Object[]`, aunque la lista sea de `String`:

```java
List<String> ls = new ArrayList<>(List.of("a", "b", "c"));

Object[] sinTipo = ls.toArray();
System.out.println(sinTipo.getClass().getName());   // [Ljava.lang.Object;
```

Y castearlo a `String[]` **no funciona**, porque el objeto realmente es un `Object[]`:

```java
String[] cast = (String[]) ls.toArray();
// ClassCastException: class [Ljava.lang.Object; cannot be cast to class [Ljava.lang.String;
```

Este es el bug clásico de la sección. Compila (el compilador acepta el cast entre tipos de array relacionados) y revienta en ejecución. Es, otra vez, la covarianza cobrándose su precio.

**Forma 2: `toArray(T[])` con un array de destino.** Es la que devuelve el tipo correcto:

```java
String[] conTipo = ls.toArray(new String[0]);
System.out.println(conTipo.getClass().getName());   // [Ljava.lang.String;
System.out.println(Arrays.toString(conTipo));       // [a, b, c]
```

El array que le pasás sirve de dos cosas: **indica el tipo** y, si es lo bastante grande, **se reutiliza como destino**.

Y ahí está la trampa de esta forma:

```java
String[] grande = ls.toArray(new String[10]);
System.out.println(grande.length);              // 10
System.out.println(Arrays.toString(grande));    // [a, b, c, null, null, null, null, null, null, null]
```

Si el array sobra, **no se recorta**: se rellenan las posiciones sobrantes con `null`. El resultado tiene longitud 10 para una lista de 3 elementos, y cualquier recorrido posterior se encuentra nulos.

Por eso el idiom correcto es **pasar un array de longitud cero**:

```java
ls.toArray(new String[0]);     // BIEN: la colección crea uno del tamaño exacto
ls.toArray(new String[ls.size()]);  // funciona, pero no es mejor
```

Durante años se recomendó la segunda forma "para evitar crear un array de más". Las mediciones modernas dieron la vuelta a ese consejo: **`new String[0]` es igual o más rápida** en las JVM actuales, porque el array vacío se optimiza y la colección puede usar `Arrays.copyOf` con el tamaño exacto en una sola operación. Además, `new String[size()]` tiene una condición de carrera si la colección cambia entre la llamada a `size()` y la copia.

**Forma 3: `toArray(IntFunction)` con un generador** (Java 11). La más clara:

```java
String[] conGenerador = ls.toArray(String[]::new);
System.out.println(Arrays.toString(conGenerador));   // [a, b, c]
```

Expresa la intención sin el array vacío ceremonial. **Para código nuevo es la mejor de las tres.**

Y desde un stream, la operación equivalente:

```java
String[] desdeStream = ls.stream().toArray(String[]::new);
int[] primitivos = lista.stream().mapToInt(Integer::intValue).toArray();
```

## 41. Varargs: un array disfrazado

Los parámetros variables (*varargs*, Java 5) permiten que un método reciba un número arbitrario de argumentos:

```java
static int sumar(int... ns) {
    int s = 0;
    for (int n : ns) s += n;
    return s;
}

System.out.println(sumar());          // 0
System.out.println(sumar(1, 2, 3));   // 6
```

**Lo que hay que entender es que `ns` es un array normal y corriente.** Los tres puntos son azúcar sintáctico: el compilador crea un array con los argumentos y se lo pasa al método. Dentro del cuerpo, `ns` es un `int[]` con todo lo que eso implica: tiene `length`, se recorre con `for-each`, se puede pasar a `Arrays.toString`.

Por eso también se puede pasar un array directamente:

```java
System.out.println(sumar(new int[]{4, 5}));   // 9
```

Las dos formas son intercambiables porque llegan a lo mismo.

**Las reglas de la declaración**, que el compilador impone:

- Solo puede haber **un** parámetro varargs por método.
- Tiene que ser el **último**: `metodo(String prefijo, int... ns)` es válido; `metodo(int... ns, String sufijo)` no.

**Las tres trampas de varargs**, verificadas:

**1. Pasar `null` da resultados distintos según el cast.**

```java
static void varargsMetodo(String... vs) {
    System.out.println(vs == null ? "null" : vs.length + " " + Arrays.toString(vs));
}

varargsMetodo("a", "b");            // 2 [a, b]
varargsMetodo();                    // 0 []
varargsMetodo(new String[]{"x"});   // 1 [x]
varargsMetodo((String) null);       // 1 [null]     <- array de un elemento nulo
varargsMetodo((String[]) null);     // null         <- el array entero es null
```

Las dos últimas líneas son la misma llamada con distinto cast y producen cosas opuestas. Por eso **un método varargs debe comprobar `if (vs == null)`** si es público y puede recibir cualquier cosa; el `for-each` sobre un array nulo lanza NPE.

**2. Sin argumentos se crea un array vacío, no `null`.** Eso es bueno y significa que el caso `sumar()` funciona sin tratamiento especial. Pero implica que **cada llamada asigna un array**, aunque esté vacío. En un método muy caliente llamado millones de veces, eso es basura innecesaria; por eso la JDK a veces ofrece sobrecargas fijas para 0, 1 y 2 argumentos junto a la versión varargs (`List.of` lo hace exactamente así).

**3. Varargs de genéricos genera un warning legítimo.**

```java
@SafeVarargs
static <T> List<T> deLista(T... elementos) {
    return List.of(elementos);
}
```

Sin `@SafeVarargs`, el compilador avisa de *possible heap pollution*: como no se puede crear un array de `T` ([sección 39](#39-por-qué-no-se-puede-crear-un-array-genérico)), crea un `Object[]` y lo castea. La anotación es la promesa de que el método **solo lee** del array y no lo expone. Ponerla en un método que escriba en el array o lo devuelva es mentir, y el bug que resulta es un `ClassCastException` en un sitio lejano.

**Dónde se usa varargs bien:** `String.format`, `List.of`, `Arrays.asList`, los métodos de logging, los constructores de tests. Es una herramienta para APIs cómodas de llamar, no para lógica interna.

---

# Parte VII — Cuándo usar un array

## 42. Array frente a ArrayList

Esta es la decisión práctica que más veces vas a tomar, y la respuesta corta es: **en código de aplicación, casi siempre `ArrayList`**. Pero conviene saber por qué, y cuándo la respuesta cambia.

`ArrayList` **es** un array por dentro. Guarda un `Object[] elementData` y lo reemplaza por uno mayor cuando se llena. Todo lo que aporta está construido encima de lo que vimos en este capítulo.

La comparación completa:

| | Array | `ArrayList` |
|---|---|---|
| Tamaño | fijo al crearlo | crece y encoge |
| Añadir / quitar | no existe | `add`, `remove` |
| Tamaño | `length` (campo) | `size()` (método) |
| Primitivos | `int[]` guarda `int` | guarda `Integer` (boxing) |
| Tipo en ejecución | lo conserva | se borra |
| Seguridad de tipos | covariante: falla en ejecución | invariante: falla en compilación |
| `toString` útil | no | sí |
| `equals` por contenido | no | sí |
| Sirve como clave de `Map` | no | sí |
| API | ninguna, todo con `Arrays` | `contains`, `indexOf`, `sort`, `stream`… |
| Memoria por elemento (`int`) | 4 bytes | ~20 bytes |

**La última fila merece explicación**, porque es el argumento real a favor de los arrays. Un `int` en un `int[]` ocupa exactamente 4 bytes. Un `Integer` en un `ArrayList<Integer>` ocupa la referencia (4 bytes con *compressed oops*) más el objeto `Integer` (16 bytes de cabecera y campo). Son aproximadamente **cinco veces más memoria**, más presión sobre el recolector, más saltos de puntero y peor localidad de caché.

Con mil elementos eso es irrelevante. Con cien millones —procesamiento de señales, imágenes, series temporales, motores de cálculo— es la diferencia entre caber en memoria y no caber.

La caché de `Integer` del capítulo anterior mitiga algo el problema para valores entre -128 y 127, pero cualquier dato real se sale de ese rango enseguida.

**La regla de decisión:**

- **Por defecto, `List`.** Es más segura, más expresiva y el coste es irrelevante en la inmensa mayoría del código.
- **Array cuando** el tamaño es fijo por naturaleza, cuando son primitivos y hay muchísimos, cuando una API te obliga, o cuando has **medido** que importa.

Y una recomendación de estilo que viene de *Effective Java*: aunque uses un array internamente, **no lo expongas en la firma de tus métodos públicos**. Devolvé `List`; el array es un detalle de implementación.

## 43. Cuándo un array sigue siendo la respuesta correcta

Hay casos donde el array no es una elección de rendimiento sino la opción natural, y conviene reconocerlos para no caer en el extremo de "nunca usar arrays".

**1. Datos binarios.** `byte[]` es el tipo universal de la JDK para bytes: entrada/salida, criptografía, red, imágenes, serialización.

```java
byte[] contenido = Files.readAllBytes(ruta);
byte[] hash = MessageDigest.getInstance("SHA-256").digest(contenido);
```

Ninguna API de este mundo devuelve `List<Byte>`, y con razón: sería veinte veces más memoria para el mismo dato.

**2. Cálculo numérico intensivo.** Matrices, vectores, series temporales, señales, píxeles, tablas de búsqueda. Aquí la memoria contigua y la ausencia de boxing son el motivo de que el programa termine.

**3. Tamaño fijo por definición del problema.** Los doce meses del año, los siete días de la semana, un tablero de ajedrez, los 256 valores posibles de un byte, una tabla de búsqueda precalculada.

```java
private static final String[] MESES = {
    "enero", "febrero", "marzo", "abril", "mayo", "junio",
    "julio", "agosto", "septiembre", "octubre", "noviembre", "diciembre"
};
```

Con un aviso: **un array `static final` no es inmutable**. Cualquiera puede escribir `MESES[0] = "hackeado"`. Si es público, hay que exponer una `List` inmutable:

```java
private static final String[] MESES_INTERNO = { ... };
public static final List<String> MESES = List.of(MESES_INTERNO);
```

**4. Implementar estructuras de datos.** Si estás escribiendo una tabla hash, un buffer circular, un montículo o una pila, el array es la primitiva sobre la que se construye.

**5. Interoperar con APIs que lo exigen.** `main(String[] args)`, `toArray`, `String.split`, `Class.getMethods`, JNI, y buena parte de las APIs anteriores a Java 5.

**6. Devolver varios valores primitivos sin crear un objeto.** Es discutible y hoy suele preferirse un `record`, pero sigue apareciendo en código de bajo nivel:

```java
static int[] divisionYResto(int a, int b) { return new int[]{a / b, a % b}; }
```

La versión moderna y más clara:

```java
record Division(int cociente, int resto) {}
```

## 44. Devolver arrays desde una API

Tres reglas que evitan los problemas más caros de este capítulo.

**Regla 1: nunca devuelvas el array interno directamente.** Ya lo vimos en la [sección 9](#9-asignar-un-array-no-lo-copia): regala el control del estado.

```java
public int[] getDatos() { return datos; }            // MAL
public int[] getDatos() { return datos.clone(); }    // aceptable
public List<Integer> getDatos() { return List.copyOf(lista); }   // mejor
```

La tercera opción es superior porque **no necesita copiar en cada llamada**: la lista inmutable se puede compartir sin riesgo. `clone()` en un getter llamado en un bucle es una fuente de basura silenciosa.

**Regla 2: nunca devuelvas `null` en lugar de un array vacío.**

```java
public String[] buscar(String criterio) {
    if (noHayResultados) return null;          // MAL
    if (noHayResultados) return new String[0]; // BIEN
}
```

Devolver `null` obliga a cada llamador a comprobarlo, y el que se olvide obtiene un NPE. Un array vacío se recorre sin problema y el bucle simplemente no da vueltas. Es la misma regla que para colecciones (`List.of()`), y una de las recomendaciones más antiguas de *Effective Java*.

Si el array vacío se devuelve mucho, conviene una constante para no crear objetos:

```java
private static final String[] VACIO = new String[0];
```

Es seguro compartir esa constante precisamente porque un array de longitud cero **no se puede modificar**: no tiene ninguna posición donde escribir. Es el único array inmutable que existe.

**Regla 3: en las entradas, copiá también.**

```java
public Configuracion(int[] puertos) {
    this.puertos = puertos.clone();   // el llamador conserva su array
}
```

Sin esa copia, el objeto queda atado a un array que otro controla, y una modificación posterior cambia su estado por la espalda.

## 45. Límites de tamaño y memoria

**El índice de un array es un `int`.** Eso pone un techo duro en `Integer.MAX_VALUE`, unos 2147 millones de elementos. No existen arrays con índices `long`.

En la práctica el límite real es algo menor, porque la JVM reserva unas posiciones para la cabecera:

```java
int[] enorme = new int[Integer.MAX_VALUE];
// OutOfMemoryError: Requested array size exceeds VM limit
```

Ese mensaje concreto —*"Requested array size exceeds VM limit"*— significa que pediste más de lo que la JVM puede direccionar, **no** que falte memoria. Es distinto del otro:

```
OutOfMemoryError: Java heap space
```

que sí significa que no hay heap suficiente. Distinguirlos ahorra tiempo: el primero se arregla cambiando el diseño, el segundo con `-Xmx`.

Por eso muchas clases de la JDK definen su tope unos bytes por debajo:

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

**El cálculo de memoria** conviene saber hacerlo:

| Array | Memoria aproximada |
|---|---|
| `boolean[n]` | n bytes (**no** n bits) |
| `byte[n]` | n bytes |
| `char[n]`, `short[n]` | 2n bytes |
| `int[n]`, `float[n]` | 4n bytes |
| `long[n]`, `double[n]` | 8n bytes |
| `Object[n]` | 4n u 8n bytes **solo de referencias**, más los objetos |

Más unos 16 bytes de cabecera en todos los casos.

La fila de `boolean[]` sorprende y es importante: **la JVM no empaqueta booleanos en bits**. Cada uno ocupa un byte entero. Para diez millones de banderas eso son 10 MB frente a los 1,25 MB de un `BitSet`, que es exactamente el caso de uso del [capítulo anterior](08-logical-relational-bitwise-operators.md#40-bitset-cuando-64-bits-no-alcanzan).

Y la última fila explica de dónde sale la diferencia de memoria de la [sección 42](#42-array-frente-a-arraylist): un `Object[]` guarda **punteros**, y los objetos apuntados están dispersos por el heap, cada uno con su propia cabecera.

**Una advertencia de seguridad** que cierra la Parte VII: si el tamaño de un array viene de una entrada externa —una cabecera de protocolo, un campo de un JSON, un parámetro HTTP—, **hay que validarlo antes de crear el array**.

```java
int tamano = leerDelCliente();
if (tamano < 0 || tamano > MAXIMO_RAZONABLE) {
    throw new IllegalArgumentException("tamaño inválido: " + tamano);
}
byte[] buffer = new byte[tamano];
```

Sin esa validación, un cliente malicioso pide un array de dos mil millones de elementos y tumba el proceso con un `OutOfMemoryError`. Es una vulnerabilidad de denegación de servicio clásica y aparece constantemente en parsers escritos a mano.

---

# Parte VIII — Cierre

## 46. Casos de uso reales

**Caso 1: contar frecuencias con un array como tabla de búsqueda.**

```java
public static int[] contarLetras(String texto) {
    int[] frecuencias = new int[26];
    for (char c : texto.toLowerCase().toCharArray()) {
        if (c >= 'a' && c <= 'z') {
            frecuencias[c - 'a']++;
        }
    }
    return frecuencias;
}
```

El truco es `c - 'a'`, que convierte una letra en un índice de 0 a 25 usando la aritmética de `char` del [capítulo 07](07-math-operations.md#3-promoción-numérica-el-tipo-del-resultado-no-es-el-que-crees). Un array de 26 posiciones es mucho más rápido y más pequeño que un `HashMap<Character, Integer>`, y aquí el tamaño está fijado por el problema. Es el uso canónico de un array como tabla de búsqueda.

**Caso 2: leer un fichero binario con validación de tamaño.**

```java
public static byte[] leerCabecera(InputStream in, int longitudDeclarada) throws IOException {
    if (longitudDeclarada < 0 || longitudDeclarada > MAX_CABECERA) {
        throw new IOException("longitud de cabecera inválida: " + longitudDeclarada);
    }
    byte[] buffer = new byte[longitudDeclarada];
    int leidos = in.readNBytes(buffer, 0, longitudDeclarada);
    if (leidos < longitudDeclarada) {
        throw new EOFException("cabecera incompleta: " + leidos + " de " + longitudDeclarada);
    }
    return buffer;
}
```

Validar el tamaño **antes** de reservar, y comprobar que se leyó todo lo pedido. Las dos comprobaciones son las que separan un parser robusto de uno que se puede tumbar con un paquete malformado.

**Caso 3: buffer circular con `System.arraycopy`.**

```java
public class BufferCircular {
    private final int[] datos;
    private int inicio = 0;
    private int tamano = 0;

    public BufferCircular(int capacidad) {
        if (capacidad <= 0) throw new IllegalArgumentException("capacidad: " + capacidad);
        this.datos = new int[capacidad];
    }

    public void añadir(int valor) {
        int posicion = (inicio + tamano) % datos.length;
        datos[posicion] = valor;
        if (tamano < datos.length) {
            tamano++;
        } else {
            inicio = (inicio + 1) % datos.length;   // sobrescribe el más antiguo
        }
    }

    public int[] contenido() {
        int[] salida = new int[tamano];
        for (int i = 0; i < tamano; i++) {
            salida[i] = datos[(inicio + i) % datos.length];
        }
        return salida;
    }
}
```

Un tamaño fijo, aritmética modular para dar la vuelta, y una copia al salir para no exponer el buffer interno. Es la estructura que hay debajo de cualquier "últimas N métricas" o "últimos N eventos".

**Caso 4: matriz de distancias triangular.**

```java
public static double[][] matrizDistancias(Punto[] puntos) {
    int n = puntos.length;
    double[][] d = new double[n][];
    for (int i = 0; i < n; i++) {
        d[i] = new double[i + 1];              // solo la mitad inferior
        for (int j = 0; j <= i; j++) {
            d[i][j] = puntos[i].distanciaA(puntos[j]);
        }
    }
    return d;
}
```

Como la distancia es simétrica, guardar la matriz completa desperdicia la mitad de la memoria. Un array irregular expresa exactamente esa estructura, y es el caso de uso que justifica los *jagged arrays*.

**Caso 5: copia defensiva en un objeto de dominio.**

```java
public final class Politica {
    private final String[] roles;

    public Politica(String[] roles) {
        this.roles = Objects.requireNonNull(roles, "roles").clone();
    }

    public List<String> roles() {
        return List.of(roles);       // vista inmutable, sin copiar en cada llamada
    }
}
```

Copia al entrar, vista inmutable al salir. `List.of` sobre el array crea una lista inmutable que puede compartirse sin riesgo, lo que evita el `clone()` en cada `getter`.

**Caso 6: convertir entre array y colección sin caer en las trampas.**

```java
// Array de primitivos -> List
List<Integer> lista = Arrays.stream(primitivos).boxed().toList();

// List -> array de primitivos
int[] devuelta = lista.stream().mapToInt(Integer::intValue).toArray();

// Array de objetos -> List mutable
List<String> mutable = new ArrayList<>(Arrays.asList(objetos));

// Array de objetos -> List inmutable
List<String> inmutable = List.of(objetos);

// List -> array de objetos
String[] array = lista.toArray(String[]::new);
```

Estas cinco líneas cubren prácticamente todas las conversiones que hacen falta, y ninguna cae en `Arrays.asList` sobre primitivos ni en `toArray()` sin tipo.

## 47. Anti-patrones

**1. Imprimir un array directamente.**

```java
System.out.println(datos);                    // [I@1b6d3586
System.out.println(Arrays.toString(datos));   // [1, 2, 3]
```

**2. Usar `Arrays.toString` con arrays anidados.** Imprime las referencias de las filas. Es `deepToString`.

**3. Comparar arrays con `==` o `equals`.** Comparan identidad. Es `Arrays.equals`, o `Arrays.deepEquals` si hay anidamiento.

**4. Usar un array como clave de `Map` o elemento de `Set`.** Nunca detecta duplicados por contenido.

**5. Confiar en `clone()` para copiar una matriz.** Es superficial: las filas siguen compartidas.

**6. `Arrays.fill` con un objeto mutable.** Las posiciones comparten la misma instancia. Es `setAll`.

**7. `Arrays.asList` sobre un array de primitivos.** Devuelve una lista de un elemento. Compila sin aviso.

**8. `Stream.of` sobre un array de primitivos.** Mismo problema: un stream de un elemento.

**9. Tratar el resultado de `Arrays.asList` como una lista normal.** No admite `add` ni `remove`, y su `set` escribe en el array original.

**10. `binarySearch` sin ordenar.** Devuelve resultados equivocados **sin lanzar nada**, y acierta a veces.

**11. Bucles con `<=`.**

```java
for (int i = 0; i <= v.length; i++) { }   // ArrayIndexOutOfBoundsException garantizada
```

**12. Usar `m[0].length` en el bucle interno de una matriz.** Falla o se salta datos con arrays irregulares. Es `m[i].length`.

**13. Recorrer una matriz por columnas cuando se puede por filas.** Desperdicia la caché sin ganar nada.

**14. Intentar modificar el array desde un `for-each`.** Asigna a la copia local. Hace falta el índice.

**15. Devolver el array interno desde un getter.** Regala el control del estado del objeto.

**16. Devolver `null` en vez de un array vacío.** Obliga a comprobar en cada llamador.

**17. `public static final Object[] CONSTANTE`.** No es constante: cualquiera puede escribir en ella.

**18. Castear el resultado de `toArray()` a `String[]`.** `ClassCastException`. Hay que usar `toArray(String[]::new)`.

**19. `toArray(new String[lista.size()])`.** No es más rápido que `new String[0]` y tiene una condición de carrera.

**20. Declarar un parámetro como `Object[]` para escribir en él.** Invita al `ArrayStoreException`.

**21. Crear un array cuyo tamaño viene de una entrada externa sin validarlo.** Denegación de servicio.

**22. `int[][][][]`.** Si necesitás cuatro dimensiones, lo que falta es una clase.

**23. Usar `int c[], d;`.** Solo `c` es un array. Los corchetes van junto al tipo.

**24. Acumular resultados en un array cuando el tamaño no se conoce.** Es un `ArrayList`.

**25. Comparadores escritos como resta al ordenar.** Desbordan y desordenan en silencio.

## 48. Checklist y tabla de decisión

**Antes de dar por terminado un código que use arrays, revisá:**

- [ ] ¿Se imprime algún array en un log o mensaje? → `Arrays.toString` o `deepToString`.
- [ ] ¿Se compara algún array? → `Arrays.equals` o `Arrays.deepEquals`, nunca `==` ni `equals`.
- [ ] ¿Algún array se usa como clave de `Map` o entra en un `Set`? → cambiar a `List` o a un `record`.
- [ ] ¿Hay algún `clone()` sobre una matriz? → es superficial, hacer copia profunda.
- [ ] ¿Hay un `Arrays.fill` con un objeto? → si es mutable, usar `setAll`.
- [ ] ¿Hay algún `Arrays.asList` o `Stream.of` sobre primitivos? → `Arrays.stream(...).boxed()`.
- [ ] ¿Se llama a `add` o `remove` sobre el resultado de `asList`? → envolver en `new ArrayList<>(...)`.
- [ ] ¿Hay un `binarySearch`? → confirmar que el array está ordenado con el mismo criterio.
- [ ] ¿Algún bucle usa `<=` sobre `length`? → cambiar a `<`.
- [ ] ¿El bucle interno de una matriz usa `m[0].length`? → cambiar a `m[i].length`.
- [ ] ¿Se recorre una matriz grande por columnas? → invertir los bucles si se puede.
- [ ] ¿Un `for-each` intenta modificar el array? → usar índice.
- [ ] ¿Algún getter devuelve un array interno? → copia defensiva o `List` inmutable.
- [ ] ¿Algún constructor guarda un array recibido? → `clone()` al entrar.
- [ ] ¿Algún método devuelve `null` en vez de array vacío? → devolver vacío.
- [ ] ¿Hay un `public static final` de tipo array? → exponer `List.of(...)`.
- [ ] ¿Se castea el resultado de `toArray()`? → usar `toArray(T[]::new)`.
- [ ] ¿El tamaño de algún array viene de fuera? → validar rango antes de crearlo.
- [ ] ¿El tamaño puede cambiar durante la vida del dato? → usar `List`.

**Tabla de decisión: qué estructura uso**

| Necesito | Uso | No uso |
|---|---|---|
| Colección que crece | `ArrayList` | array + `copyOf` |
| Tamaño fijo por el problema | array | `ArrayList` |
| Muchos primitivos | `int[]`, `double[]` | `List<Integer>` |
| Datos binarios | `byte[]` | `List<Byte>` |
| Millones de banderas | `BitSet` | `boolean[]` |
| Conjunto de constantes de un enum | `EnumSet` | array de flags |
| Clave compuesta de un `Map` | `record` o `List` | array |
| Exponer datos en una API pública | `List` | array |
| Constante pública | `List.of(...)` | `static final T[]` |
| Devolver "nada" | array vacío o `List.of()` | `null` |

**Tabla de decisión: qué operación uso**

| Quiero | Uso | No uso |
|---|---|---|
| Imprimir | `Arrays.toString` / `deepToString` | `println(array)` |
| Comparar contenido | `Arrays.equals` / `deepEquals` | `==`, `equals`, `Objects.equals` |
| Saber dónde difieren | `Arrays.mismatch` | bucle a mano |
| Copia idéntica | `clone()` | bucle a mano |
| Copia con otro tamaño | `Arrays.copyOf` | `new` + bucle |
| Copiar un trozo | `Arrays.copyOfRange` | bucle a mano |
| Copiar en un array existente | `System.arraycopy` | bucle a mano |
| Copia profunda de matriz | `stream().map(int[]::clone)` | `clone()` |
| Rellenar con valores distintos | `Arrays.setAll` | `Arrays.fill` |
| Ordenar objetos | `Arrays.sort` + `Comparator.comparing` | comparador por resta |
| Buscar en array ordenado | `Arrays.binarySearch` | búsqueda lineal |
| Buscar en array sin ordenar | bucle o stream | `binarySearch` |
| Array de primitivos → `List` | `Arrays.stream(v).boxed().toList()` | `Arrays.asList(v)` |
| `List` → array | `toArray(T[]::new)` | `(T[]) toArray()` |
| Agregar (suma, máximo, media) | `Arrays.stream(v).summaryStatistics()` | cuatro bucles |

## 49. Autoevaluación

**1. ¿Qué imprime `System.out.println(new int[]{1,2,3});` y por qué?**

<details><summary>Respuesta</summary>

Imprime algo como `[I@1b6d3586`. Los arrays son objetos que heredan de `Object` pero **no sobrescriben `toString()`**, así que se usa el de `Object`, que produce `getClass().getName() + "@" + Integer.toHexString(hashCode())`. El `[I` es el nombre interno de `int[]`: un `[` por dimensión y `I` por `int`. Para ver el contenido hay que usar `Arrays.toString(v)`, o `Arrays.deepToString(v)` si hay anidamiento.
</details>

**2. ¿Por qué `a1.equals(a2)` devuelve `false` para dos arrays de contenido idéntico?**

<details><summary>Respuesta</summary>

Porque los arrays tampoco sobrescriben `equals`, así que heredan el de `Object`, que es `this == obj`: comparación de **identidad**. Dos arrays creados por separado son objetos distintos. `Objects.equals` no ayuda porque delega en ese mismo `equals`. La única forma correcta es `Arrays.equals` para una dimensión y `Arrays.deepEquals` para arrays anidados. Lo mismo aplica a `hashCode`, y por eso un array nunca funciona como clave de un `HashMap`.
</details>

**3. ¿Qué diferencia hay entre `int[] b = a;` y `int[] b = a.clone();`?**

<details><summary>Respuesta</summary>

La primera **no copia nada**: `b` y `a` apuntan al mismo objeto, así que escribir por una variable se ve por la otra. La segunda crea un objeto nuevo con el mismo contenido, independiente del original. La confusión viene de que la sintaxis de asignación es la misma para primitivos y para referencias, pero en el segundo caso lo que se copia es la referencia, no el objeto.
</details>

**4. ¿Por qué `clone()` no sirve para copiar una matriz?**

<details><summary>Respuesta</summary>

Porque es una copia **superficial**: copia el array externo, cuyos elementos son referencias a las filas, y esas referencias siguen apuntando a las mismas filas. `clon[0][0] = 99` modifica también `original[0][0]`, y `original[0] == clon[0]` es `true`. Para una copia profunda hay que clonar fila por fila, con un bucle o con `Arrays.stream(m).map(int[]::clone).toArray(int[][]::new)`.
</details>

**5. ¿Qué hace `Arrays.fill(matriz, new int[2])` y por qué es peligroso?**

<details><summary>Respuesta</summary>

Pone **la misma referencia** en todas las filas de la matriz, porque `new int[2]` se evalúa una sola vez antes de la llamada. El resultado es que `matriz[0][0] = 7` deja la matriz en `[[7,0],[7,0],[7,0]]`: las tres filas son el mismo array. `fill` solo es seguro con primitivos, con `null` o con objetos inmutables. Para crear objetos distintos por posición hay que usar `Arrays.setAll`, que llama al generador una vez por índice.
</details>

**6. ¿Qué devuelve `Arrays.asList(new int[]{1,2,3}).size()` y por qué?**

<details><summary>Respuesta</summary>

Devuelve `1`. `asList` recibe `T...`, y como `int` no puede ser un tipo genérico, Java infiere `T = int[]` y crea una lista de **un solo elemento** que es el array entero. Compila sin ningún aviso. Con `Integer[]` funcionaría como se espera. La forma correcta para primitivos es `Arrays.stream(v).boxed().toList()`. `Stream.of` tiene exactamente la misma trampa por la misma razón.
</details>

**7. Además de no poder crecer, ¿qué otras dos sorpresas tiene la lista que devuelve `Arrays.asList`?**

<details><summary>Respuesta</summary>

Primera: **`set` sí funciona y escribe en el array original**, porque la lista es una vista, no una copia. Quien la reciba creyéndola inmutable puede modificar el array de quien se la pasó. Segunda: su clase no es `java.util.ArrayList` sino `java.util.Arrays$ArrayList`, una clase interna privada con el mismo nombre simple, lo que confunde al depurar. Para una lista mutable de verdad: `new ArrayList<>(Arrays.asList(...))`; para una inmutable de verdad: `List.of(...)`.
</details>

**8. ¿Qué devuelve `Arrays.binarySearch` cuando no encuentra el elemento?**

<details><summary>Respuesta</summary>

Un número negativo que codifica el punto de inserción, con la fórmula `-(punto de inserción) - 1`. Sobre `{0,2,4,6,8,10}`, buscar `7` devuelve `-5`, porque habría que insertarlo en el índice 4. Se recupera con `-(resultado) - 1`. El `-1` de la fórmula existe para que un punto de inserción 0 no dé `-0`, que es `0` y sería indistinguible de "encontrado en el índice 0".
</details>

**9. ¿Qué pasa si llamás a `binarySearch` sobre un array sin ordenar?**

<details><summary>Respuesta</summary>

Devuelve un resultado indefinido **sin lanzar ninguna excepción**. Y lo peor es que a veces acierta: sobre `{5,2,8,1,9}`, buscar `8` devuelve `2`, que es correcto por casualidad, mientras que buscar `2` devuelve `-1` cuando el `2` está en el índice 1. Esa mezcla de aciertos y fallos es lo que permite que el bug pase los tests y llegue a producción. Si no podés garantizar el orden, usá búsqueda lineal.
</details>

**10. ¿Por qué `for (int x : v) x = x * 2;` no modifica el array?**

<details><summary>Respuesta</summary>

Porque `x` es una variable local a la que se **copia** el valor de cada elemento; el bytecode lo muestra como un `istore` a un local nuevo. Asignarle algo cambia la copia, no la casilla del array. Con objetos hay un matiz: no podés reemplazar el elemento, pero sí **mutar el objeto** al que apunta (`sb.append("!")` sí afecta al array). Para escribir en el array hace falta el índice y un `for` clásico.
</details>

**11. ¿Qué diferencia hay en el bytecode entre un `for` clásico y un `for-each` sobre un array?**

<details><summary>Respuesta</summary>

El `for-each` compila a un bucle indexado con un índice oculto, y **cachea `arraylength` en un local antes del bucle**; el `for` clásico vuelve a leer la longitud en cada iteración. La diferencia práctica es nula porque el JIT las iguala. Lo importante es la otra conclusión: sobre arrays **no se crea ningún `Iterator`**, a diferencia de lo que ocurre con colecciones. `for-each` sobre un array no asigna nada y no tiene coste.
</details>

**12. ¿Cuántos objetos crea `new int[3][4]`?**

<details><summary>Respuesta</summary>

Cuatro: el array externo de longitud 3, cuyos elementos son referencias, y las tres filas de longitud 4. Java no tiene matrices de verdad, tiene arrays de arrays. Se comprueba con `m.getClass().getName()`, que da `[[I`, y `m[0].getClass().getName()`, que da `[I`. En bytecode se usa `multianewarray` con argumento 2; `new int[3][]` en cambio usa un simple `anewarray`, que es el mismo que para `new String[3]`.
</details>

**13. ¿Por qué recorrer una matriz por columnas es más lento que por filas?**

<details><summary>Respuesta</summary>

Por la caché del procesador. Cuando la CPU lee una posición trae una línea de caché entera (unos 64 bytes, 16 `int`). Recorriendo por filas, los siguientes 15 accesos ya están en caché; recorriendo por columnas, cada acceso salta a otra fila —otro objeto en el heap— y desperdicia casi toda la línea. Medido sobre `int[4000][4000]` en JDK 25: 38 ms por filas frente a 49 ms por columnas. La regla es que el índice que varía más rápido debe ser el último.
</details>

**14. ¿Qué es la covarianza de arrays y qué problema causa?**

<details><summary>Respuesta</summary>

Que si `String` es subtipo de `Object`, entonces `String[]` es subtipo de `Object[]`, así que `Object[] o = new String[3];` compila. El problema es que el compilador cree que se puede guardar cualquier `Object` ahí, pero el array real solo acepta `String`, y `o[0] = Integer.valueOf(42)` lanza `ArrayStoreException` en ejecución. Es el único sitio de Java donde el compilador aprueba una asignación que la JVM rechaza. Los genéricos, diseñados después, son invariantes precisamente para evitarlo.
</details>

**15. ¿Por qué `new T[10]` no compila dentro de una clase genérica?**

<details><summary>Respuesta</summary>

Porque un array necesita conocer su tipo **en ejecución** —lo guarda en la cabecera y lo usa para comprobar `ArrayStoreException`—, mientras que un parámetro genérico `T` se **borra** en compilación (*type erasure*) y no existe en ejecución. Son requisitos incompatibles. Las salidas son usar una `List<T>` (lo recomendable), guardar un `Object[]` y castear al leer con `@SuppressWarnings("unchecked")`, o pasar un `Class<T>` y usar `Array.newInstance`.
</details>

**16. ¿Cuál es la forma correcta de convertir una `List<String>` en `String[]`?**

<details><summary>Respuesta</summary>

`lista.toArray(String[]::new)` (Java 11+), o `lista.toArray(new String[0])`. Lo que **no** funciona es `(String[]) lista.toArray()`, porque el `toArray()` sin argumentos devuelve un `Object[]` real y el cast lanza `ClassCastException`. Y `toArray(new String[10])` sobre una lista de 3 elementos devuelve un array de longitud 10 con siete `null` al final: si el array sobra, no se recorta.
</details>

**17. ¿Qué algoritmo usa `Arrays.sort` y por qué depende del tipo?**

<details><summary>Respuesta</summary>

Dual-Pivot Quicksort para arrays de primitivos y TimSort para arrays de objetos. La razón es la estabilidad: dos primitivos iguales son indistinguibles, así que da igual el orden entre ellos y se puede usar quicksort, que es más rápido y no necesita memoria extra. Dos objetos "iguales" según el comparador pueden ser distintos, y conservar su orden relativo permite ordenar por varios criterios encadenando ordenaciones. TimSort además aprovecha los tramos ya ordenados y baja a O(n) sobre datos casi ordenados.
</details>

**18. ¿Qué significa que un ordenamiento sea estable y por qué importa?**

<details><summary>Respuesta</summary>

Que los elementos que el comparador considera iguales conservan su orden relativo original. Importa porque permite ordenar por varios criterios aplicando ordenaciones sucesivas: ordenar primero por nombre y luego por id deja el resultado ordenado por id y, dentro de cada id, por nombre. Con un algoritmo inestable ese resultado sería arbitrario. `Arrays.sort` es estable para `Object[]` (TimSort) pero **no** para primitivos (quicksort), aunque ahí no importe.
</details>

**19. ¿Por qué un getter no debe devolver el array interno de un objeto?**

<details><summary>Respuesta</summary>

Porque devuelve la referencia, así que el llamador puede escribir en el estado interno: `objeto.getDatos()[0] = 1` modifica el objeto por la espalda. El `final` del campo no protege: impide reasignar la variable, no modificar el array apuntado. Las soluciones son devolver `datos.clone()` (con el coste de copiar en cada llamada) o, mejor, devolver `List.of(datos)`, una vista inmutable que se puede compartir sin copiar.
</details>

**20. ¿Qué significa exactamente `OutOfMemoryError: Requested array size exceeds VM limit`?**

<details><summary>Respuesta</summary>

Que se pidió un array mayor de lo que la JVM puede direccionar, **no** que falte memoria. El índice de un array es un `int`, así que el techo es `Integer.MAX_VALUE`, y el límite real es unos bytes menor por la cabecera del objeto. Es distinto de `OutOfMemoryError: Java heap space`, que sí significa falta de heap y se arregla con `-Xmx`. El primero se arregla cambiando el diseño. Por eso varias clases de la JDK definen `MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8`.
</details>

## 50. Fuentes

**Documentación oficial**

- [Arrays — The Java Tutorials](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html) — la definición canónica de array como *"an object containing a fixed number of values of the same type"*.
- [JLS §10 — Arrays](https://docs.oracle.com/javase/specs/jls/se25/html/jls-10.html) — la especificación completa: creación, valores por defecto, covarianza y las reglas de `ArrayStoreException`.
- [JLS §4.10.3 — Subtyping among Array Types](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.10.3) — la regla de covarianza escrita formalmente.
- [`java.util.Arrays` — JDK 25 API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Arrays.html) — la referencia completa, incluida la fórmula `-(insertion point) - 1` de `binarySearch` y la advertencia de que el array debe estar ordenado.
- [`System.arraycopy`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/System.html#arraycopy(java.lang.Object,int,java.lang.Object,int,int)) — donde está documentado el manejo correcto del solapamiento.
- [JEP 358: Helpful NullPointerExceptions](https://openjdk.org/jeps/358) — por qué el NPE de un array nulo dice exactamente qué falló.
- [Java Object Layout (JOL)](https://openjdk.org/projects/code-tools/jol/) — la herramienta oficial para medir de verdad cuánto ocupa un array o un objeto en memoria.

**Las tres fuentes de referencia de este capítulo, y dónde se equivocan**

- [Jenkov — Java Arrays](https://jenkov.com/tutorials/java/arrays.html). Es la más completa de las tres en cuanto a operaciones prácticas: cubre inserción, eliminación, búsqueda de mínimo y máximo, y buena parte de `java.util.Arrays`. **Problema 1:** los métodos `insertIntoArray` y `removeFromArray` que propone se presentan sin la advertencia esencial de que **el array no cambia de tamaño**: insertar descarta silenciosamente el último elemento y eliminar deja el último duplicado. Un lector que los copie tal cual pierde datos sin enterarse. **Problema 2:** su ejemplo de ordenación con `Comparator` usa `return e1.employeeId - e2.employeeId;`, que es el comparador por resta que desborda con ids extremos; verificado en JDK 25, con `Integer.MAX_VALUE` y `Integer.MIN_VALUE` ese comparador devuelve la lista **desordenada** sin lanzar nada. **Hueco:** no menciona la covarianza, `ArrayStoreException`, `deepToString`, `deepEquals`, `setAll`, `mismatch`, `compare` ni `stream`, ni la diferencia entre copia superficial y profunda.
- [W3Schools — Java Arrays](https://www.w3schools.com/java/java_arrays.asp) y sus subpáginas de [recorrido](https://www.w3schools.com/java/java_arrays_loop.asp), [multidimensionales](https://www.w3schools.com/java/java_arrays_multi.asp) y [referencia de métodos](https://www.w3schools.com/java/java_ref_arrays.asp). Sirve como primer contacto de sintaxis y poco más. **Hueco 1:** la página de referencia de `java.util.Arrays` lista **siete métodos** (`compare`, `copyOf`, `deepEquals`, `equals`, `fill`, `mismatch`, `sort`). Faltan `toString` y `deepToString` —que son los que un principiante necesita antes que ninguno—, además de `asList`, `binarySearch`, `copyOfRange`, `hashCode`, `deepHashCode`, `setAll`, `stream`, `parallelSort` y `parallelPrefix`. Llamar a eso una referencia es generoso. **Hueco 2:** no menciona en ningún momento los **valores por defecto**, ni que un array de objetos nace lleno de `null`, ni `ArrayIndexOutOfBoundsException`, ni que los arrays son objetos, ni que asignar un array no lo copia. Son exactamente los cuatro conceptos que producen los bugs de este capítulo. **Acierto que conviene reconocer:** su página de multidimensionales **sí** señala explícitamente que las filas pueden tener longitudes distintas, cosa que muchos tutoriales omiten.
- [Baeldung — Arrays in Java: A Reference Guide](https://www.baeldung.com/java-arrays-guide) y [Guide to the java.util.Arrays Class](https://www.baeldung.com/java-util-arrays), de la [serie de arrays](https://www.baeldung.com/java-arrays-series). Es con diferencia la más completa y la única que cubre varargs, streams, `parallelPrefix` y la distinción entre `hashCode` y `deepHashCode`. Aun así hay tres cosas que corregir. **Contradicción interna:** el primer artículo afirma que *"the algorithms behind the sort method are quick sort and merge sort for primitive and other arrays, respectively"*, mientras que el segundo dice —correctamente— que *"Primitive types use a dual-pivot quicksort and Object types use Timsort"*. TimSort es un merge sort adaptativo, así que la primera formulación no es exactamente falsa, pero pierde las dos propiedades que importan (adaptatividad y **estabilidad**) y contradice al otro artículo del mismo sitio. **Error 2:** el primer artículo escribe el nombre de la excepción como `ArrayIndexOutOfBoundException`, sin la `s` de `Bounds`. Copiado a un bloque `catch` no compila. **Error 3:** afirma sobre `Arrays.asList` que *"it's not possible to use an array of primitive types"*. Es más grave que eso: **sí compila**, y silenciosamente devuelve una lista de un solo elemento (`Arrays.asList(new int[]{1,2,3}).size()` da `1`). Presentarlo como imposible sugiere que el compilador te protege, cuando el problema es justamente que no lo hace. **Imprecisión menor:** la tabla de benchmark de `parallelPrefix` etiqueta las filas de modo `avgt` con unidades `ops/s`, cuando ese modo reporta tiempo por operación; los números están invertidos respecto a su etiqueta.

**Discusiones y artículos de la comunidad** (consultados para contrastar los casos límite)

- [Arrays.asList() vs new ArrayList(Arrays.asList())](https://stackoverflow.com/questions/2607289/converting-array-to-list-in-java) — el hilo canónico sobre las tres trampas de `asList`.
- [Arrays of Wisdom of the Ancients](https://shipilev.net/blog/2016/arrays-wisdom-ancients/) — el análisis con JMH de Aleksey Shipilëv que desmontó el consejo de `toArray(new T[size])` y estableció que `new T[0]` es igual o mejor. Es la referencia definitiva sobre el tema.
- [Why are arrays covariant but generics are invariant?](https://stackoverflow.com/questions/18666710/why-are-arrays-covariant-but-generics-are-invariant) — la razón histórica explicada con la cita de los diseñadores.
- [How do I determine whether an array contains a particular value?](https://stackoverflow.com/questions/1128723/how-can-i-test-if-an-array-contains-a-certain-value) — el recordatorio de que `Arrays.asList(...).contains(...)` no funciona con primitivos.
- [Java Arrays.sort: dual-pivot quicksort](https://algs4.cs.princeton.edu/23quicksort/) y el [artículo original de Vladimir Yaroslavskiy](https://web.archive.org/web/20150511210832/http://iaroslavski.narod.ru/quicksort/DualPivotQuicksort.pdf) — el algoritmo que la JDK adoptó en Java 7.
- [TimSort: description of the algorithm](https://github.com/python/cpython/blob/main/Objects/listsort.txt) — la nota original de Tim Peters, escrita para Python y adoptada por Java.
- [What is heap pollution?](https://stackoverflow.com/questions/12462079/potential-heap-pollution-via-varargs-parameter) — por qué existe `@SafeVarargs`.

**Nota sobre la verificación.** Todos los outputs de este documento se obtuvieron ejecutando el código en Temurin JDK 25.0.3 (`java Archivo.java`), los volcados de bytecode con `javap -c`, y los mensajes de error compilando a propósito archivos que fallan (`javac`). La medición de recorrido por filas frente a columnas de la [sección 24](#24-localidad-de-caché-por-filas-frente-a-por-columnas) es propia, con calentamiento previo pero **sin JMH**: sirve como orden de magnitud y como confirmación de la dirección del efecto, no como benchmark riguroso. Las cifras de memoria de la [sección 45](#45-límites-de-tamaño-y-memoria) son estimaciones basadas en el layout de objetos de HotSpot con *compressed oops*; para medirlas de verdad hay que usar JOL.
