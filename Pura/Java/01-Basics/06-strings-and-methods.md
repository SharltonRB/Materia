# Strings and Methods

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** Los temas anteriores establecieron qué tipos existen ([Data Types and Variables](03-data-types-and-variables.md)), dónde vive cada variable ([Variables and Scopes](04-variables-and-scopes.md)) y cómo se pasa un valor de un tipo a otro ([Type Casting](05-type-casting.md)). Este cubre **el tipo con el que más vas a trabajar en tu vida profesional y el que peor se enseña**: `String`.

Casi todo programa recibe texto, lo valida, lo transforma y lo devuelve. Un endpoint HTTP recibe strings. Un CSV es strings. Un JSON es strings. Un log es strings. Un nombre de usuario, un email, un identificador, una consulta SQL, una ruta de archivo: strings. Y sin embargo el tema se despacha habitualmente con una tabla de métodos y un `System.out.println`.

El problema es que `String` en Java tiene un modelo mental propio que no se parece al de ningún otro tipo que hayas visto hasta ahora: es un objeto que se comporta como si fuera un valor, es inmutable en un lenguaje donde casi nada lo es, tiene una zona de memoria dedicada solo para él, el compilador le aplica optimizaciones que no aplica a nada más, y su unidad interna (`char`) dejó de corresponderse con lo que un humano llama "carácter" hace más de veinte años.

De ese desajuste salen bugs concretos y caros: comparaciones que dan `true` en desarrollo y `false` en producción, formularios que aceptan un nombre y luego no lo encuentran en la base de datos, un `split` que descarta silenciosamente las últimas columnas de un CSV, un `toUpperCase()` que rompe la aplicación entera solo para los usuarios de Turquía, un contador de caracteres que dice 5 cuando ves 4, y un bucle de concatenación que funciona perfecto con 100 elementos y tumba el servidor con 100.000.

Vamos a cubrir el modelo completo: qué es un `String` por dentro, cómo se crea, cómo se compara, cómo se transforma, cómo se construye eficientemente, y qué pasa cuando el texto deja de ser ASCII.

**Sobre la verificación.** Todos los outputs, mensajes de excepción, tiempos y volcados de bytecode de este documento fueron ejecutados realmente en un JDK 25, no copiados de tutoriales. Como en el capítulo anterior, esto importa: al preparar el documento encontré que **una de las tres fuentes de referencia principales afirma algo que fue cierto hasta Java 8 y es falso desde Java 9** — y lo sigue presentando como el comportamiento actual. Lo señalo en la [sección 29](#29-concatenación-con--qué-genera-realmente-el-compilador), con el bytecode al lado, porque saber dónde envejecen los tutoriales es parte de dominar el tema.

---

## Índice

**Parte I — Qué es un `String`**
1. [El problema que resuelve `String`](#1-el-problema-que-resuelve-string)
2. [`String` no es un primitivo, ni un array de `char`](#2-string-no-es-un-primitivo-ni-un-array-de-char)
3. [Representación interna: de `char[]` a `byte[]`](#3-representación-interna-de-char-a-byte)
4. [Inmutabilidad: la decisión que explica todo lo demás](#4-inmutabilidad-la-decisión-que-explica-todo-lo-demás)

**Parte II — Crear strings**

5. [Literales y `new String()`](#5-literales-y-new-string)
6. [El String pool](#6-el-string-pool)
7. [Constantes de compilación](#7-constantes-de-compilación)
8. [`intern()`: qué hace y cuándo usarlo](#8-intern-qué-hace-y-cuándo-usarlo)
9. [Escapes y caracteres especiales](#9-escapes-y-caracteres-especiales)
10. [Text blocks](#10-text-blocks)

**Parte III — Comparar strings**

11. [`==` vs `equals()`: el error número uno](#11--vs-equals-el-error-número-uno)
12. [`equalsIgnoreCase()` y sus límites](#12-equalsignorecase-y-sus-límites)
13. [`compareTo()`: orden lexicográfico](#13-compareto-orden-lexicográfico)
14. [Orden humano: `Collator`](#14-orden-humano-collator)
15. [`contentEquals`, `regionMatches` y `matches`](#15-contentequals-regionmatches-y-matches)
16. [Comparar sin morir en el intento: null-safety](#16-comparar-sin-morir-en-el-intento-null-safety)

**Parte IV — Inspeccionar y extraer**

17. [`length()`, `isEmpty()`, `isBlank()`](#17-length-isempty-isblank)
18. [Buscar: `indexOf`, `contains` y familia](#18-buscar-indexof-contains-y-familia)
19. [`substring()` y sus excepciones](#19-substring-y-sus-excepciones)
20. [`trim()` vs `strip()`](#20-trim-vs-strip)

**Parte V — Transformar**

21. [Toda transformación devuelve un objeto nuevo](#21-toda-transformación-devuelve-un-objeto-nuevo)
22. [Mayúsculas, minúsculas y el bug del locale turco](#22-mayúsculas-minúsculas-y-el-bug-del-locale-turco)
23. [`replace` vs `replaceAll` vs `replaceFirst`](#23-replace-vs-replaceall-vs-replacefirst)
24. [`split()`: el método que más sorprende](#24-split-el-método-que-más-sorprende)
25. [Unir: `join` y `StringJoiner`](#25-unir-join-y-stringjoiner)
26. [La API moderna: `repeat`, `lines`, `indent` y compañía](#26-la-api-moderna-repeat-lines-indent-y-compañía)

**Parte VI — Construir strings**

27. [El coste real de construir texto](#27-el-coste-real-de-construir-texto)
28. [`StringBuilder` y `StringBuffer`](#28-stringbuilder-y-stringbuffer)
29. [Concatenación con `+`: qué genera realmente el compilador](#29-concatenación-con--qué-genera-realmente-el-compilador)
30. [`String.format()` y `formatted()`](#30-stringformat-y-formatted)
31. [Concatenación en logging: el caso que sí importa](#31-concatenación-en-logging-el-caso-que-sí-importa)

**Parte VII — Unicode: cuando el texto deja de ser ASCII**

32. [`char` no es un carácter](#32-char-no-es-un-carácter)
33. [Code points y surrogate pairs](#33-code-points-y-surrogate-pairs)
34. [Grafemas: lo que el usuario llama "un carácter"](#34-grafemas-lo-que-el-usuario-llama-un-carácter)
35. [Normalización: dos textos idénticos que no son iguales](#35-normalización-dos-textos-idénticos-que-no-son-iguales)
36. [Bytes y encodings](#36-bytes-y-encodings)

**Parte VIII — Conversiones**

37. [De `String` a número y de vuelta](#37-de-string-a-número-y-de-vuelta)
38. [`valueOf`, `toString` y el `null`](#38-valueof-tostring-y-el-null)

**Parte IX — Cierre**

39. [Strings y seguridad: por qué las contraseñas van en `char[]`](#39-strings-y-seguridad-por-qué-las-contraseñas-van-en-char)
40. [Casos de uso reales](#40-casos-de-uso-reales)
41. [Anti-patrones](#41-anti-patrones)
42. [Checklist y tabla de decisión](#42-checklist-y-tabla-de-decisión)
43. [Autoevaluación](#43-autoevaluación)
44. [Fuentes](#44-fuentes)

---

# Parte I — Qué es un `String`

## 1. El problema que resuelve `String`

Empecemos por lo básico: una computadora no almacena texto. Almacena números. La letra `A` no existe en la memoria; lo que existe es el número 65, más un acuerdo previo sobre que el 65 se dibuja como `A`. Ese acuerdo se llama **codificación de caracteres**.

Un texto, entonces, es una **secuencia ordenada de números** que representan caracteres. La palabra `"Hola"` es la secuencia `72, 111, 108, 97`.

Un lenguaje de programación tiene que ofrecerte alguna forma de trabajar con esas secuencias. Java podría haber elegido lo que eligió C: que el texto sea simplemente un array de caracteres, y que vos te encargues del resto.

```java
// Esto es válido en Java, y es lo más parecido al enfoque de C
char[] saludo = {'H', 'o', 'l', 'a'};
```

Ese enfoque funciona, pero deja todo el trabajo del lado del programador. ¿Cómo comparás dos de estos? Recorriendo posición por posición. ¿Cómo los concatenás? Reservando un array nuevo del tamaño de la suma y copiando ambos. ¿Cómo buscás una palabra dentro de otra? Escribiendo el algoritmo de búsqueda. Cada uno de esos pasos es una oportunidad de equivocarse, y en C históricamente lo fue: buena parte de las vulnerabilidades de seguridad de los últimos cuarenta años son errores de manejo de buffers de texto.

Java tomó otro camino: **encapsular la secuencia de caracteres dentro de una clase que ofrece todas las operaciones ya resueltas y que no permite modificar el contenido**. Esa clase es `java.lang.String`.

```java
String saludo = "Hola";
System.out.println(saludo.length());              // 4
System.out.println(saludo.toUpperCase());         // HOLA
System.out.println(saludo.contains("ol"));        // true
System.out.println(saludo + " mundo");            // Hola mundo
```

Lo que estás viendo en esas cuatro líneas es el valor real de `String`: la secuencia sigue estando ahí abajo, pero vos no la tocás nunca. Trabajás con operaciones de alto nivel que ya están escritas, probadas y optimizadas por gente que dedicó años a eso.

## 2. `String` no es un primitivo, ni un array de `char`

Esta sección existe porque la confusión que corrige es la fuente de la mitad de los errores del resto del documento.

En Java hay **ocho tipos primitivos**: `byte`, `short`, `int`, `long`, `float`, `double`, `char` y `boolean`. Los viste en [Data Types and Variables](03-data-types-and-variables.md). `String` **no está en esa lista**. `String` es una **clase**, y toda variable de tipo `String` es una **referencia** a un objeto que vive en el heap.

Esto tiene tres consecuencias inmediatas que conviene fijar antes de seguir.

**Consecuencia 1: una variable `String` puede ser `null`.**

Un `int` siempre tiene un valor numérico. Un `String` puede no apuntar a ningún objeto:

```java
int numero;          // si es campo de instancia, vale 0
String texto;        // si es campo de instancia, vale null
```

Y `null` no es lo mismo que `""`. Un string vacío es un objeto que existe y contiene cero caracteres. `null` es la ausencia de objeto. Llamar a cualquier método sobre `null` lanza `NullPointerException`. Esta distinción va a reaparecer en la [sección 16](#16-comparar-sin-morir-en-el-intento-null-safety) y en la [sección 17](#17-length-isempty-isblank).

**Consecuencia 2: `==` compara referencias, no contenido.**

Como `String` es una referencia, el operador `==` hace lo que hace con cualquier referencia: pregunta si las dos variables apuntan **al mismo objeto**, no si tienen el mismo texto. Esto es tan importante y tan contraintuitivo que tiene su propia sección ([sección 11](#11--vs-equals-el-error-número-uno)).

**Consecuencia 3: `String` no es un `char[]`, aunque se le parezca.**

Son tipos distintos y no intercambiables. Existen conversiones explícitas en ambas direcciones:

```java
String texto = "Hola";
char[] array = texto.toCharArray();       // String -> char[]
String vuelta = new String(array);        // char[] -> String
String vuelta2 = String.valueOf(array);   // equivalente
```

Pero cuidado con una trampa clásica: los arrays en Java no tienen un `toString()` útil.

```java
char[] array = {'H', 'o', 'l', 'a'};
System.out.println(array);                    // Hola   (println tiene sobrecarga para char[])
System.out.println("valor: " + array);        // valor: [C@1b6d3586   (concatenación usa toString())
System.out.println(Arrays.toString(array));   // [H, o, l, a]
```

Esas tres líneas dan tres resultados distintos con el mismo array. La primera funciona porque `System.out.println` tiene una sobrecarga específica que recibe `char[]`. La segunda usa concatenación, que llama a `toString()`, y el `toString()` heredado de `Object` imprime el tipo y el hash. Es un error que aparece cuando alguien intenta loguear un array de caracteres.

### El punto clave

`String` es una clase, pero el lenguaje la trata de forma especial en varios lugares:

- Tiene **sintaxis literal** propia (`"texto"`), algo que ninguna otra clase tiene.
- El operador `+` funciona con ella, y es el **único** operador sobrecargado de Java.
- El compilador le aplica optimizaciones específicas ([sección 7](#7-constantes-de-compilación)).
- Tiene una zona de memoria dedicada ([sección 6](#6-el-string-pool)).
- Se puede usar en `switch`.

Esa mezcla de "es una clase normal" y "el lenguaje la trata distinto" es exactamente lo que hace que el tema confunda. La regla para no perderse: **`String` es un objeto en todo lo que importa para la semántica (referencias, `null`, `equals`), y tiene atajos sintácticos para que escribirlo sea cómodo.**

## 3. Representación interna: de `char[]` a `byte[]`

Saber qué hay dentro de un `String` no es trivia: explica el consumo de memoria de tu aplicación y varios comportamientos de la Parte VII.

### Hasta Java 8

Internamente, un `String` guardaba un array de `char`:

```java
// java.lang.String, hasta Java 8 (simplificado)
private final char[] value;
```

Un `char` en Java ocupa **2 bytes** (16 bits), porque Java usa **UTF-16** internamente. Esa decisión se tomó en los años noventa, cuando se creía que 16 bits alcanzaban para todos los caracteres del mundo.

El problema práctico: la enorme mayoría del texto que procesan las aplicaciones reales es ASCII o Latin-1 — letras sin acento, dígitos, signos de puntuación. Todos esos caracteres entran en **1 byte**. Guardarlos en 2 bytes significa que **la mitad de la memoria dedicada a texto eran ceros**. Y como los strings suelen ocupar un porcentaje enorme del heap de una aplicación Java típica, eso era mucha memoria desperdiciada.

### Desde Java 9: Compact Strings

Java 9 introdujo los **compact strings** ([JEP 254](https://openjdk.org/jeps/254)). El campo interno cambió a:

```java
// java.lang.String, desde Java 9 (simplificado)
private final byte[] value;
private final byte coder;      // 0 = LATIN1, 1 = UTF16
```

La idea: al crear el string, la JVM examina su contenido. Si **todos** los caracteres entran en Latin-1, guarda 1 byte por carácter y marca `coder = LATIN1`. Si aunque sea uno necesita más, guarda 2 bytes por carácter para todos y marca `coder = UTF16`.

Cada método consulta el `coder` y delega en la implementación correspondiente:

```java
// Así se ve internamente un método de String desde Java 9
public int indexOf(int ch, int fromIndex) {
    return isLatin1()
      ? StringLatin1.indexOf(value, ch, fromIndex)
      : StringUTF16.indexOf(value, ch, fromIndex);
}
```

Esto es transparente: tu código no cambia. Pero explica cosas que verás en producción:

- Una aplicación que migra de Java 8 a Java 11+ suele bajar su consumo de heap de forma notable, sin tocar una línea de código.
- Un string con **un solo** emoji o carácter chino ocupa el doble de bytes por carácter que uno equivalente en ASCII, aunque el resto del contenido sea idéntico.

> **Nota histórica.** Java 6 tuvo un intento previo, la opción `-XX:+UseCompressedStrings`, que se eliminó en Java 7 por efectos secundarios sobre el rendimiento. Compact strings es el diseño que sí funcionó, y está activo por defecto.

### Por qué esto solo es posible gracias a la inmutabilidad

Fijate en el detalle: la decisión de usar 1 o 2 bytes se toma **una sola vez, al crear el string**. Si el contenido pudiera cambiar después, habría que reevaluar la codificación en cada modificación, y todo el esquema se caería. Lo que hace viable esta optimización es que el contenido nunca cambia. Que es justo el tema de la sección siguiente.

## 4. Inmutabilidad: la decisión que explica todo lo demás

**Un objeto `String`, una vez creado, no puede modificarse. Nunca. Bajo ninguna circunstancia.**

Esta es la propiedad central del tipo, y si la entendés de verdad, el 80% de los errores del resto del documento se vuelven obvios.

### Qué significa en la práctica

Ningún método de `String` modifica el string sobre el que lo llamás. Todos **devuelven un objeto nuevo**:

```java
String texto = "hola";
texto.toUpperCase();
System.out.println(texto);       // hola   <- NO cambió
```

Este es el error más común de las primeras semanas. `toUpperCase()` creó un `String` nuevo con el contenido `"HOLA"`, nadie lo guardó, y el recolector de basura lo eliminó. La variable `texto` sigue apuntando al objeto original, que nunca cambió porque no puede cambiar.

Lo correcto es asignar el resultado:

```java
String texto = "hola";
texto = texto.toUpperCase();
System.out.println(texto);       // HOLA
```

Ojo con lo que pasó acá: no se modificó ningún objeto. Se creó uno nuevo y se **reapuntó la variable**. El objeto `"hola"` original sigue existiendo intacto (y si es un literal, sigue en el pool).

> Esta distinción entre "cambiar el objeto" y "cambiar a dónde apunta la variable" es la misma que vimos con `final` en [Variables and Scopes](04-variables-and-scopes.md). Una variable `String` no `final` puede reapuntarse; el objeto al que apunta jamás cambia.

### Por qué los diseñadores del lenguaje eligieron esto

No fue una decisión estética. Hay cinco razones concretas, y conviene conocerlas porque son exactamente las que te van a preguntar en una entrevista.

**1. Permite el pool.** Si el contenido no cambia, la JVM puede almacenar una sola copia de cada literal y repartir la misma referencia a todo el que la pida ([sección 6](#6-el-string-pool)). Con strings mutables esto sería catastrófico: cambiar el texto en un lugar lo cambiaría en todos los demás.

**2. Seguridad.** Los strings son el vehículo de datos sensibles: rutas de archivo, URLs de conexión, nombres de clase, parámetros de red. Considerá este patrón, típico de código con control de acceso:

```java
void operacionCritica(String nombreUsuario) {
    if (!esAlfanumerico(nombreUsuario)) {     // validación
        throw new SecurityException();
    }
    inicializarBaseDeDatos();                 // trabajo intermedio
    conexion.executeUpdate("UPDATE Clientes SET Estado = 'Activo' "
                         + " WHERE Usuario = '" + nombreUsuario + "'");
}
```

Si `String` fuera mutable, otro hilo podría modificar el contenido de `nombreUsuario` **entre la validación y el uso**. Validarías un valor y ejecutarías otro. Es la vulnerabilidad conocida como TOCTOU (*time-of-check to time-of-use*). Al ser inmutable, lo que validaste es exactamente lo que se usa.

**3. Thread-safety gratis.** Un objeto que no puede cambiar puede compartirse entre cualquier cantidad de hilos sin sincronización, sin locks y sin riesgo. Es el único tipo de objeto del que se puede afirmar eso sin analizar nada más.

**4. Cacheo del hash.** `String` se usa masivamente como clave de `HashMap`. Como el contenido no cambia, el `hashCode()` tampoco, así que se calcula una vez y se guarda:

```java
// java.lang.String
private int hash;               // cache del hashCode
```

Si `String` fuera mutable, cambiar el texto de una clave ya insertada en un mapa la volvería inencontrable: el mapa la buscaría en el bucket del hash nuevo, pero está guardada en el del viejo.

**5. Permite optimizaciones del compilador y la JVM.** Constant folding ([sección 7](#7-constantes-de-compilación)) y compact strings ([sección 3](#3-representación-interna-de-char-a-byte)) dependen de la inmutabilidad.

### El precio

La inmutabilidad no es gratis. Cada transformación crea un objeto nuevo, y eso significa asignación de memoria y trabajo del recolector de basura. En operaciones aisladas es irrelevante. En bucles es un desastre — y esa es exactamente la razón de existir de `StringBuilder`, que veremos en la [Parte VI](#parte-vi--construir-strings).

### Un matiz para entrevistas: `final` no es inmutable

Son cosas distintas y se confunden todo el tiempo:

```java
final String a = "hola";
a = "chau";            // ERROR de compilación: no podés reapuntar la variable

String b = "hola";
b = b.toUpperCase();   // OK: la variable no es final, se reapunta
```

`final` es una propiedad de **la variable** (no se puede reasignar). La inmutabilidad es una propiedad **del objeto** (su estado no cambia). `String` es inmutable siempre, sea `final` la variable o no.

---

# Parte II — Crear strings

## 5. Literales y `new String()`

Hay dos formas de crear un string, y no son equivalentes.

### La forma literal

```java
String saludo = "Hola mundo";
```

Todo texto entre comillas dobles en el código fuente es un **literal de string**. El compilador lo registra en el pool de constantes de la clase, y en tiempo de ejecución la JVM lo coloca en el String pool.

### La forma con constructor

```java
String saludo = new String("Hola mundo");
```

Esto hace **dos** cosas: el literal `"Hola mundo"` va al pool (porque sigue siendo un literal en el código fuente), y además `new` crea un objeto **adicional** en el heap, fuera del pool, con una copia del contenido.

### Por qué esto importa

```java
String a = "Hello World";
String b = "Hello World";
String c = new String("Hello World");

System.out.println(a == b);        // true
System.out.println(a == c);        // false
System.out.println(a.equals(c));   // true
```

*(Salida verificada en JDK 25.)*

`a` y `b` son **el mismo objeto**: ambos son literales idénticos, así que apuntan a la única copia del pool. `c` es un objeto distinto con el mismo contenido.

### La regla práctica

> **Usá siempre la forma literal. `new String(...)` es un anti-patrón en el 99,9% de los casos.**

No aporta nada, consume memoria extra, y encima rompe la comparación con `==` de una forma que confunde a quien lea el código. Herramientas como SonarQube, SpotBugs o los propios avisos del IDE lo marcan como problema.

Los casos legítimos son tan raros que conviene enumerarlos para cerrar la duda:

```java
// 1. Copia defensiva de un char[] mutable (relevante en manejo de contraseñas)
char[] buffer = leerDeConsola();
String copia = new String(buffer);   // el String no cambia si después limpiás el buffer

// 2. Construir desde bytes con una codificación explícita
String texto = new String(bytes, StandardCharsets.UTF_8);

// 3. Forzar una identidad distinta (usos muy específicos: locks por instancia, tests del pool)
String candado = new String("recurso");
```

El segundo caso es el único que vas a usar de verdad, y no es realmente "crear un string duplicado": es decodificar bytes ([sección 36](#36-bytes-y-encodings)).

## 6. El String pool

El **String pool** (también llamado *string constant pool* o *intern pool*) es una zona de memoria donde la JVM guarda **una única copia de cada valor de string literal**.

### Cómo funciona

Cuando la JVM va a materializar un literal:

1. Busca en el pool si ya existe un string con ese contenido exacto.
2. Si existe, devuelve **la referencia al que ya está**. No asigna memoria nueva.
3. Si no existe, crea el objeto, lo agrega al pool y devuelve su referencia.

Ese proceso se llama **interning**.

```java
String s1 = "Baeldung";
String s2 = "Baeldung";
// s1 y s2 son literalmente la misma dirección de memoria
```

### Dónde vive el pool

Un detalle que aparece en entrevistas: **hasta Java 6 el pool estaba en PermGen**, un espacio de memoria de tamaño fijo y limitado. Llenarlo provocaba `OutOfMemoryError: PermGen space`. **Desde Java 7 el pool vive en el heap normal**, así que participa del garbage collection habitual y su tamaño lo limita el heap. PermGen desapareció por completo en Java 8, reemplazado por Metaspace.

### Por qué existe

Por memoria. En una aplicación real, el mismo literal aparece miles de veces: `"id"`, `"name"`, `"UTF-8"`, `"error"`, cada clave de configuración, cada nombre de columna. Sin el pool, cada aparición sería un objeto distinto. Con el pool, hay uno solo.

### El efecto secundario incómodo

El pool tiene una consecuencia que arruina la vida a los principiantes: **hace que `==` funcione, a veces**.

```java
String a = "hola";
String b = "hola";
if (a == b) { ... }        // true — parece que == compara texto
```

Funciona. Y entonces alguien concluye que `==` sirve para comparar strings, escribe todo el programa así, y meses después el mismo código falla porque el string ahora viene de un `Scanner`, de una consulta a base de datos o de un parseo de JSON — y esos no son literales, así que no están en el pool:

```java
Scanner sc = new Scanner(System.in);
String entrada = sc.nextLine();      // el usuario escribe: hola
System.out.println(entrada == "hola");        // false
System.out.println(entrada.equals("hola"));   // true
```

Este es, literalmente, el bug más frecuente de Java. Volvemos sobre él en la [sección 11](#11--vs-equals-el-error-número-uno).

## 7. Constantes de compilación

Acá hay una sutileza que casi ningún tutorial explica, y que produce resultados desconcertantes si no la conocés.

El compilador de Java resuelve en tiempo de compilación toda expresión de string cuyos operandos sean **constantes** — un mecanismo llamado *constant folding*. El resultado se convierte también en un literal, y por lo tanto va al pool.

Ejecutá esto mentalmente antes de leer la salida:

```java
String a = "Hello World";

String d = "Hello" + " World";          // (1)
String parte = "World";
String e = "Hello " + parte;            // (2)
final String parteFinal = "World";
String f = "Hello " + parteFinal;       // (3)

System.out.println(a == d);   // ?
System.out.println(a == e);   // ?
System.out.println(a == f);   // ?
```

Salida real en JDK 25:

```
true
false
true
```

Vamos caso por caso:

- **(1) `"Hello" + " World"`** — ambos operandos son literales. El compilador hace la cuenta y en el `.class` no queda ninguna concatenación: queda directamente el literal `"Hello World"`, que es el mismo objeto del pool que `a`. Por eso `true`.
- **(2) `"Hello " + parte`** — `parte` es una variable normal. El compilador no puede saber su valor en tiempo de compilación (podría reasignarse). La concatenación ocurre **en runtime** y produce un objeto nuevo fuera del pool. Por eso `false`.
- **(3) `"Hello " + parteFinal`** — `parteFinal` es `final` **y** está inicializada con un literal. Eso la convierte en una *constant variable* según la especificación del lenguaje (JLS §4.12.4). El compilador sí puede sustituir su valor, hace el folding, y otra vez queda un literal. Por eso `true`.

Lo mismo aplica a campos `static final`:

```java
static final String CONST = "Hello";
// ...
System.out.println(a == (CONST + " World"));   // true
```

### Para qué sirve saber esto

Para dos cosas.

Primero, para **no sacar conclusiones erróneas experimentando con `==`**. Mucha gente prueba `==` con literales, ve que funciona, y generaliza mal.

Segundo, porque explica por qué las anotaciones aceptan concatenaciones:

```java
@RequestMapping(BASE_PATH + "/usuarios")   // compila solo si BASE_PATH es constante
```

Los valores de anotación deben ser constantes de compilación. Si `BASE_PATH` es `static final String` inicializada con literal, funciona. Si no lo es, error de compilación. Es el mismo mecanismo.

## 8. `intern()`: qué hace y cuándo usarlo

`intern()` te permite meter manualmente un string en el pool:

```java
String dinamico = new StringBuilder("ho").append("la").toString();

System.out.println(dinamico == "hola");            // false
System.out.println(dinamico.intern() == "hola");   // true
System.out.println(dinamico.equals("hola"));       // true
```

*(Salida verificada en JDK 25.)*

`intern()` busca en el pool un string con el mismo contenido. Si lo encuentra, devuelve **esa** referencia; si no, agrega el string al pool y devuelve su propia referencia.

### La trampa clásica

`intern()` **no modifica** el string sobre el que lo llamás — no podría, porque los strings son inmutables. Devuelve una referencia. Si no la asignás, no pasó nada:

```java
String s = new String("hola");
s.intern();                        // el valor devuelto se descarta
System.out.println(s == "hola");   // false

String t = new String("hola");
t = t.intern();                    // ahora sí
System.out.println(t == "hola");   // true
```

Es exactamente el mismo error que llamar a `toUpperCase()` sin asignar.

### Cuándo usarlo

**Casi nunca.** El caso legítimo es muy concreto: estás cargando en memoria millones de objetos donde un campo de texto se repite muchísimo (por ejemplo, un campo `pais` o `moneda` en 10 millones de registros parseados). Sin `intern()` tendrías 10 millones de strings `"ARS"`; con `intern()`, uno solo y 10 millones de referencias.

Aun así, hoy hay alternativas mejores y más predecibles:

```java
// Alternativa: pool propio, con control total y sin tocar el pool de la JVM
Map<String, String> cache = new HashMap<>();
String canonico = cache.computeIfAbsent(valor, Function.identity());
```

Ventajas del pool propio: podés medir su tamaño, vaciarlo cuando querés, y no competís con la tabla interna de la JVM (que es de tamaño fijo por defecto y cuyo rendimiento se degrada al llenarse; se ajusta con `-XX:StringTableSize`).

> **Regla:** si no medíste un problema de memoria concreto causado por strings duplicados, no uses `intern()`.

## 9. Escapes y caracteres especiales

Dentro de un literal, algunos caracteres no se pueden escribir directamente: las comillas dobles cerrarían el literal, y el salto de línea no se puede teclear dentro de comillas. Para eso está la **barra invertida** (`\`) como carácter de escape.

| Escape | Resultado | Uso típico |
|---|---|---|
| `\"` | `"` | Comilla doble dentro del literal |
| `\'` | `'` | Comilla simple (necesaria en `char`, opcional en `String`) |
| `\\` | `\` | Barra invertida literal |
| `\n` | Salto de línea (LF) | Nueva línea |
| `\r` | Retorno de carro (CR) | Fin de línea en Windows, junto a `\n` |
| `\t` | Tabulador | Alineación |
| `\b` | Backspace | Raro |
| `\f` | Form feed | Muy raro |
| `\s` | Espacio | Java 15+, útil en text blocks |
| `\uXXXX` | Carácter Unicode | `\u00F1` es `ñ` |

```java
String conComillas = "Somos los llamados \"Vikingos\" del norte.";
String conBarra    = "El carácter \\ se llama barra invertida.";
String multilinea  = "Primera línea\nSegunda línea";
String tabulado    = "\tTexto indentado con un tab";
```

### Dos errores frecuentes

**Rutas de Windows.** Cada barra invertida se escribe doble:

```java
String malo  = "C:\Users\ana\docs";     // ERROR de compilación: \U y \a no son escapes válidos
String bueno = "C:\\Users\\ana\\docs";  // correcto
```

Mejor todavía: no hardcodees separadores. Usá `Path`:

```java
Path ruta = Path.of("C:", "Users", "ana", "docs");   // portable
```

**Regex dentro de literales.** Un patrón de regex ya usa `\` para sus propios escapes, y el literal Java consume una capa. Para que la regex reciba `\d`, tenés que escribir `\\d`:

```java
String soloDigitos = "\\d+";       // la regex recibe: \d+
```

Esta doble capa es una fuente inagotable de confusión; reaparece en las secciones [23](#23-replace-vs-replaceall-vs-replacefirst) y [24](#24-split-el-método-que-más-sorprende).

### `\uXXXX` se procesa antes que todo

Un detalle esotérico pero real: los escapes `\uXXXX` se procesan en la **fase léxica**, antes de analizar la sintaxis. Es decir, funcionan incluso fuera de literales, con resultados absurdos:

```java
// Esto compila y es un comentario que rompe el código:
// \u000A System.out.println("ejecutado");
```

`\u000A` es un salto de línea, así que el compilador termina el comentario ahí y lo que sigue se convierte en código real. Es una curiosidad conocida, y la razón por la que nunca conviene escribir `\u` en comentarios.

## 10. Text blocks

Antes de Java 15, escribir texto multilínea era doloroso:

```java
String json = "{\n"
            + "  \"id\": 1,\n"
            + "  \"nombre\": \"Ana\"\n"
            + "}";
```

Hay más ruido de escapes y concatenación que contenido. Los **text blocks** ([JEP 378](https://openjdk.org/jeps/378), estándar desde Java 15) resuelven esto:

```java
String json = """
        {
          "id": 1,
          "nombre": "Ana"
        }""";
```

Salida real:

```
{
  "id": 1,
  "nombre": "Ana"
}
```

Un text block empieza con `"""` seguido **obligatoriamente** de un salto de línea, y termina con `"""`. Dentro no hace falta escapar comillas dobles ni saltos de línea.

### La regla de indentación (la parte que confunde)

El compilador no incluye la indentación del código fuente. Calcula la **indentación mínima común** entre todas las líneas no vacías **y la línea de cierre**, y la elimina de todas.

Esto significa que **la posición del `"""` de cierre controla la indentación resultante**:

```java
String a = """
           texto
           """;      // cierre alineado con el texto -> sin indentación

String b = """
           texto
       """;          // cierre 4 espacios a la izquierda -> "    texto\n"
```

Es el mecanismo que permite indentar el bloque de acuerdo al código que lo rodea sin que esa indentación acabe en el string.

### El salto de línea final

Detalle que causa fallos de tests:

```java
String sinSalto = """
        Ana""";        // NO termina en \n  (cierre pegado al texto)

String conSalto = """
        Ana
        """;           // SÍ termina en \n  (cierre en su propia línea)
```

Verificado: `json.endsWith("\n")` devuelve `false` en el primer ejemplo. Si comparás contra un valor esperado y no te cuadra por un carácter, esto suele ser la causa.

### Escapes propios de text blocks

Java 15 agregó dos escapes pensados para estos bloques:

```java
// \ al final de línea: une líneas sin insertar salto
String unaLinea = """
        una sola \
        linea""";
// -> "una sola linea"

// \s: preserva espacios finales que el compilador borraría
String conEspacios = """
        fin con espacio\s\s
        """;
// -> "fin con espacio  \n"
```

*(Ambos outputs verificados en JDK 25.)*

El segundo importa porque el compilador **elimina los espacios al final de cada línea** de un text block. Si necesitás preservarlos —en un formato de ancho fijo, por ejemplo— `\s` es la forma.

### Casos de uso reales

Los text blocks brillan con contenido que ya tiene su propia sintaxis:

```java
// SQL
String query = """
        SELECT u.id, u.nombre, p.total
        FROM usuarios u
        JOIN pedidos p ON p.usuario_id = u.id
        WHERE u.activo = true
          AND p.fecha >= ?
        ORDER BY p.fecha DESC
        """;

// JSON de test, sin escapar una sola comilla
String payloadEsperado = """
        {"status":"OK","items":[1,2,3]}""";

// HTML
String plantilla = """
        <div class="card">
          <h2>%s</h2>
        </div>
        """.formatted(titulo);
```

Ese último combina text block con `formatted()` ([sección 30](#30-stringformat-y-formatted)), que es el patrón idiomático para plantillas hoy.

> **Sobre String Templates.** Java 21 y 22 incluyeron como *preview* una función llamada String Templates (`STR."Hola \{nombre}"`), pensada como interpolación segura. **Fue retirada en Java 23** y no está disponible en Java 25; el diseño se está rehaciendo. Si ves ejemplos con `STR.`, son de una versión preview que ya no existe. Para interpolar, usá `formatted()` o `String.format()`.

---

# Parte III — Comparar strings

## 11. `==` vs `equals()`: el error número uno

Esta es la sección más importante del documento.

### La regla

> **`==` compara si dos variables apuntan al mismo objeto. `equals()` compara el contenido. Para texto, siempre `equals()`.**

```java
String a = "hola";
String b = new String("hola");

System.out.println(a == b);        // false  -> objetos distintos
System.out.println(a.equals(b));   // true   -> mismo contenido
```

### Por qué el error sobrevive tanto tiempo

Porque `==` **funciona** en el escenario donde la gente aprende:

```java
String a = "hola";
String b = "hola";
System.out.println(a == b);   // true
```

Ambos son literales, el pool les da la misma referencia, y `==` da `true`. El código pasa los tests. Se despliega. Y falla el día que el string deja de ser un literal:

```java
// Todos estos producen strings FUERA del pool:
String deConsola   = scanner.nextLine();
String deArchivo   = Files.readString(path);
String deJson      = objectMapper.readValue(...);
String deBaseDatos = resultSet.getString("nombre");
String concatenado = "ho" + variable;
String recortado   = otro.substring(0, 4);
```

Ninguno está internado. `==` devuelve `false` aunque el texto sea idéntico. Y el bug es especialmente cruel porque **es intermitente en apariencia**: funciona en los tests unitarios (que usan literales) y falla con datos reales.

### Cómo se comporta `equals()`

```java
String s1 = "usando equals";
String s2 = "usando equals";
String s3 = "USANDO EQUALS";
String s4 = new String("usando equals");

s1.equals(s2);      // true
s1.equals(s4);      // true  -> el origen no importa, solo el contenido
s1.equals(s3);      // false -> distingue mayúsculas
s1.equals(null);    // false -> nunca lanza excepción
```

Tres propiedades a retener: compara carácter por carácter, **distingue mayúsculas**, y **devuelve `false` con `null`** en vez de explotar.

### El único uso legítimo de `==` con strings

Cuando querés saber deliberadamente si son el mismo objeto. En código de aplicación, prácticamente nunca. Aparece en implementaciones internas como optimización previa:

```java
// Patrón real dentro de java.lang.String.equals
public boolean equals(Object anObject) {
    if (this == anObject) return true;   // atajo: mismo objeto -> iguales seguro
    // ... comparación carácter por carácter
}
```

Ese `==` es una optimización, no la comparación.

### Cómo detectarlo antes de que llegue a producción

Casi todas las herramientas de análisis estático lo detectan: SonarQube (regla S1698), SpotBugs (`ES_COMPARING_STRINGS_WITH_EQ`), y los propios avisos de IntelliJ IDEA y Eclipse. Si estás en un equipo, esto debería estar en el pipeline de CI.

## 12. `equalsIgnoreCase()` y sus límites

Cuando la diferencia de mayúsculas no debe contar:

```java
String s1 = "usando equals ignore case";
String s2 = "USANDO EQUALS IGNORE CASE";
System.out.println(s1.equalsIgnoreCase(s2));   // true
```

Es lo correcto para comparar emails, nombres de usuario o cabeceras HTTP.

### El anti-patrón que reemplaza

Mucha gente escribe esto:

```java
if (a.toLowerCase().equals(b.toLowerCase())) { ... }   // MAL
```

Tiene tres problemas: crea dos objetos innecesarios, depende del locale por defecto (y por lo tanto puede romperse en Turquía — [sección 22](#22-mayúsculas-minúsculas-y-el-bug-del-locale-turco)), y es más largo. `equalsIgnoreCase()` no crea objetos intermedios y compara carácter a carácter.

### Sus límites reales

`equalsIgnoreCase` compara carácter a carácter aplicando reglas de mayúsculas/minúsculas **individuales**. Eso deja fuera casos donde el mapeo no es uno a uno:

```java
System.out.println("ß".equalsIgnoreCase("SS"));   // false
```

*(Verificado en JDK 25.)*

En alemán, la eszett `ß` en mayúsculas es `SS` — dos caracteres. Como el mapeo no es carácter a carácter, `equalsIgnoreCase` no lo resuelve. De hecho `"ß".toUpperCase()` sí devuelve `"SS"`, con longitud 2 partiendo de longitud 1.

Si necesitás comparación insensible a mayúsculas **correcta en cualquier idioma**, la herramienta es `Collator` ([sección 14](#14-orden-humano-collator)).

## 13. `compareTo()`: orden lexicográfico

`equals()` responde "¿son iguales?". `compareTo()` responde "¿cuál va primero?", que es lo que hace falta para ordenar.

Devuelve un `int`: negativo si el receptor va antes, cero si son iguales, positivo si va después.

```java
System.out.println("abc".compareTo("def"));    // -3
System.out.println("abc".compareTo("abd"));    // -1
System.out.println("abc".compareTo("ABC"));    // 32
System.out.println("abc".compareTo("abcd"));   // -1
```

*(Salida verificada en JDK 25.)*

### De dónde salen esos números

No son arbitrarios, y entenderlos evita malinterpretar el método:

- **`"abc"` vs `"def"` → -3**: compara posición por posición; en la primera difieren: `'a'` (97) − `'d'` (100) = −3.
- **`"abc"` vs `"abd"` → -1**: coinciden hasta la tercera; `'c'` (99) − `'d'` (100) = −1.
- **`"abc"` vs `"ABC"` → 32**: `'a'` (97) − `'A'` (65) = 32. **Las minúsculas van después de las mayúsculas.**
- **`"abc"` vs `"abcd"` → -1**: uno es prefijo del otro; devuelve la diferencia de longitudes: 3 − 4 = −1.

> **Error a evitar:** no interpretes el valor exacto. Lo único garantizado por el contrato es el **signo**. Escribí `if (a.compareTo(b) < 0)`, nunca `if (a.compareTo(b) == -1)`.

### La consecuencia práctica: el orden "natural" no es alfabético

```java
List<String> palabras = List.of("zorro", "ñandu", "avion", "Ávila", "banana", "Zeta");
Collections.sort(new ArrayList<>(palabras));
// [Zeta, avion, banana, zorro, Ávila, ñandu]
```

*(Salida verificada en JDK 25.)*

Ese orden no es el que espera ningún usuario. `Zeta` va primero porque las mayúsculas tienen código menor; `Ávila` y `ñandu` van al final porque sus caracteres acentuados están más arriba en Unicode. **`compareTo()` ordena por número de code point, no por alfabeto humano.**

Para orden insensible a mayúsculas existe un comparador listo:

```java
lista.sort(String.CASE_INSENSITIVE_ORDER);
// [avion, banana, Zeta, zorro, Ávila, ñandu]
```

Mejor, pero los acentos siguen al final. La solución completa es la sección siguiente.

## 14. Orden humano: `Collator`

Cuando el orden lo va a **ver un usuario**, `compareTo()` no sirve. La herramienta correcta es `java.text.Collator`, que aplica las reglas de ordenación del idioma:

```java
List<String> palabras = new ArrayList<>(
        List.of("zorro", "ñandu", "avion", "Ávila", "banana", "Zeta"));

palabras.sort(Collator.getInstance(Locale.forLanguageTag("es")));
// [Ávila, avion, banana, ñandu, Zeta, zorro]
```

*(Salida verificada en JDK 25.)*

Ahora sí: los acentos se ordenan junto a su letra base, la `ñ` va después de la `n`, y las mayúsculas no alteran la posición.

### Cuándo usar cada uno

| Situación | Herramienta |
|---|---|
| Ordenar para mostrar a un usuario | `Collator` con el locale correspondiente |
| Ordenar claves internas, IDs, tokens | `compareTo()` (rápido y determinista) |
| Índices, estructuras ordenadas, claves de mapa | `compareTo()` — **nunca** `Collator` |
| Comparar ignorando acentos y mayúsculas | `Collator` con `setStrength(Collator.PRIMARY)` |

```java
Collator c = Collator.getInstance(Locale.forLanguageTag("es"));
c.setStrength(Collator.PRIMARY);          // ignora acentos y mayúsculas
System.out.println(c.compare("Ávila", "avila") == 0);   // true
```

Esto último es lo que querés para un buscador donde escribir "avila" debe encontrar "Ávila".

> **Advertencia de rendimiento:** `Collator` es bastante más lento que `compareTo()`. Para ordenar una lista que se muestra en pantalla, irrelevante. Para ordenar un millón de claves internas, no.

## 15. `contentEquals`, `regionMatches` y `matches`

Tres métodos de comparación menos conocidos que resuelven problemas concretos.

### `contentEquals`: comparar con un `StringBuilder`

`equals()` devuelve `false` si el argumento no es un `String`. Un `StringBuilder` con el mismo texto no es igual:

```java
String s = "abc";
StringBuilder sb = new StringBuilder("abc");

System.out.println(s.equals(sb));           // false — tipos distintos
System.out.println(s.contentEquals(sb));    // true  — compara contenido
```

*(Verificado en JDK 25.)*

Esto sorprende a mucha gente. `contentEquals` acepta cualquier `CharSequence` (`String`, `StringBuilder`, `StringBuffer`, `CharBuffer`).

Relacionado: **`StringBuilder` no sobrescribe `equals()`**, así que compara por identidad:

```java
StringBuilder a = new StringBuilder("a");
StringBuilder b = new StringBuilder("a");
System.out.println(a.equals(b));       // false (!)
System.out.println(a.compareTo(b));    // 0 — compareTo sí compara contenido (Java 11+)
```

*(Verificado en JDK 25.)*

Es una fuente de bugs sutiles cuando alguien mete `StringBuilder` en un `Set` o lo usa como clave de mapa. **No uses `StringBuilder` como clave de nada.**

### `regionMatches`: comparar un tramo sin crear substrings

Para comparar una porción sin asignar objetos intermedios:

```java
String frase = "El gato negro";
System.out.println(frase.regionMatches(3, "gato", 0, 4));                 // true
System.out.println("El GATO negro".regionMatches(true, 3, "gato", 0, 4)); // true (ignoreCase)
```

*(Verificado en JDK 25.)*

La alternativa ingenua, `frase.substring(3, 7).equals("gato")`, crea un `String` nuevo solo para tirarlo. En un parser que procesa millones de líneas, la diferencia es real.

### `matches`: regex, con una trampa importante

`matches()` comprueba si el string **completo** cumple un patrón:

```java
System.out.println("one two three".matches("two"));        // false (!)
System.out.println("one two three".matches(".*two.*"));    // true
System.out.println("one two three".contains("two"));       // true
```

*(Salida verificada en JDK 25.)*

**`matches()` está anclado a los dos extremos.** No busca el patrón dentro del texto: exige que el patrón describa la cadena entera. Quien viene de otros lenguajes donde `match` busca coincidencias parciales se equivoca acá sistemáticamente.

Si querés buscar dentro, usá `contains()` (texto literal) o `Pattern`/`Matcher` con `find()` (regex).

Además, `matches()` **compila la regex en cada llamada**. Dentro de un bucle, precompilá:

```java
// MAL en un bucle
for (String linea : lineas) {
    if (linea.matches("\\d{4}-\\d{2}-\\d{2}")) { ... }
}

// BIEN
private static final Pattern FECHA = Pattern.compile("\\d{4}-\\d{2}-\\d{2}");
for (String linea : lineas) {
    if (FECHA.matcher(linea).matches()) { ... }
}
```

## 16. Comparar sin morir en el intento: null-safety

`equals()` tolera un `null` **como argumento**, pero no como receptor:

```java
String s = null;
System.out.println("abc".equals(s));   // false — seguro
System.out.println(s.equals("abc"));   // NullPointerException
```

De ahí sale una técnica defensiva conocida como **Yoda condition**: poner primero la constante.

```java
if ("admin".equals(rolUsuario)) { ... }    // seguro aunque rolUsuario sea null
if (rolUsuario.equals("admin")) { ... }    // NPE si rolUsuario es null
```

Se lee peor, pero elimina una clase entera de errores. En código donde el valor puede venir de fuera (request, base de datos, configuración), vale la pena.

### Alternativas más limpias

```java
// java.util.Objects: null-safe en ambos lados
Objects.equals(a, b);                 // true si ambos son null

// Si el null es un caso válido de negocio, hacelo explícito
Optional.ofNullable(rolUsuario)
        .filter("admin"::equals)
        .isPresent();
```

`Objects.equals(a, b)` es la opción por defecto cuando ninguno de los dos lados está garantizado.

### La comparación con `null` que nunca falla

```java
if (texto == null) { ... }        // correcto: == con null siempre es seguro
```

Este es otro uso legítimo de `==`: comparar contra `null`. No podés usar `equals` para eso.

---

# Parte IV — Inspeccionar y extraer

## 17. `length()`, `isEmpty()`, `isBlank()`

### `length()`

Devuelve la cantidad de **unidades de código UTF-16** (`char`) del string:

```java
String txt = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
System.out.println(txt.length());   // 26
```

Dos advertencias que conviene fijar desde ya:

1. **Es `length()` con paréntesis.** En arrays es `array.length` sin paréntesis (es un campo). Confundirlos es un error de compilación clásico de las primeras semanas.
2. **No cuenta "caracteres" en el sentido humano.** Cuenta unidades UTF-16. Para texto ASCII coinciden; para emojis y muchos alfabetos, no. Es el tema de la [Parte VII](#parte-vii--unicode-cuando-el-texto-deja-de-ser-ascii).

### `isEmpty()` vs `isBlank()`

```java
System.out.println("".isEmpty());      // true
System.out.println("   ".isEmpty());   // false   <- tiene 3 caracteres
System.out.println("   ".isBlank());   // true    <- son todos espacios
```

*(Verificado en JDK 25.)*

- `isEmpty()` (Java 6+): `length() == 0`. Nada más.
- `isBlank()` (Java 11+): vacío **o** compuesto solo por espacios en blanco.

**Para validar entrada de usuario, casi siempre querés `isBlank()`.** Un formulario donde alguien escribe tres espacios en el campo "nombre" no debería pasar la validación, y `isEmpty()` la deja pasar.

### El chequeo completo, con `null`

El error que sigue a esto es olvidar el `null`:

```java
// MAL: NPE si nombre es null
if (nombre.isBlank()) { ... }

// BIEN: orden correcto, aprovechando el cortocircuito de ||
if (nombre == null || nombre.isBlank()) { ... }
```

El orden importa: `||` evalúa de izquierda a derecha y corta en cuanto encuentra `true`, así que si `nombre` es `null` nunca se llama a `isBlank()`. Invertir los operandos rompe el código.

Si usás Spring o Apache Commons, ya hay utilidades para esto:

```java
StringUtils.hasText(nombre);      // Spring: no null, no vacío, no solo espacios
StringUtils.isNotBlank(nombre);   // Apache Commons Lang
```

## 18. Buscar: `indexOf`, `contains` y familia

### `indexOf` y `lastIndexOf`

`indexOf` devuelve la posición de la primera aparición, o **`-1` si no está**:

```java
String s = "Hello World";
System.out.println(s.indexOf("World"));   // 6
System.out.println(s.indexOf("z"));       // -1
```

*(Verificado en JDK 25.)*

Ese `-1` es la fuente de un error clásico:

```java
// MAL: si no está, substring(-1 + 1) da resultados absurdos en vez de fallar
String extension = archivo.substring(archivo.indexOf(".") + 1);

// BIEN: verificá antes
int punto = archivo.lastIndexOf(".");
String extension = (punto >= 0) ? archivo.substring(punto + 1) : "";
```

Notá que ahí usé `lastIndexOf`, no `indexOf`: para extraer la extensión de `"backup.tar.gz"` querés la **última** aparición.

**Buscar todas las apariciones** es un patrón que conviene tener memorizado:

```java
String texto = "is this good or is this bad?";
String buscado = "is";

int index = texto.indexOf(buscado);
while (index != -1) {
    System.out.println(index);
    index = texto.indexOf(buscado, index + 1);   // seguir desde la siguiente posición
}
// 0, 5, 16, 21
```

La sobrecarga `indexOf(String, int fromIndex)` es la que permite avanzar. Si olvidás el `+ 1`, tenés un bucle infinito.

### `contains`

Cuando solo te importa si está o no:

```java
if (texto.contains("error")) { ... }
```

Es más legible que `indexOf(...) != -1` y hace exactamente lo mismo por dentro. **Usá `contains` salvo que necesites la posición.**

Importante: `contains` busca **texto literal**, no regex. Esto no falla:

```java
"precio: 10.50".contains(".")    // true — el punto es un punto, no "cualquier carácter"
```

### `startsWith` y `endsWith`

```java
String s = "This is a good day to code";

System.out.println(s.startsWith("This"));      // true
System.out.println(s.startsWith("is", 5));     // true  — busca desde el índice 5
System.out.println(s.endsWith("code"));        // true
System.out.println("archivo.pdf".endsWith(".pdf"));   // true
```

*(Verificado en JDK 25.)*

La sobrecarga con offset de `startsWith` es poco conocida y muy útil en parsers: permite comprobar un prefijo en una posición arbitraria sin cortar el string.

Uso típico:

```java
if (url.startsWith("https://")) { ... }
if (archivo.endsWith(".pdf")) { ... }
```

> Para nombres de archivo, ojo con las mayúsculas: `"FOTO.PDF".endsWith(".pdf")` es `false`. Si el origen no es controlado, normalizá primero con `toLowerCase(Locale.ROOT)`.

## 19. `substring()` y sus excepciones

Extrae una porción. **El índice de inicio es inclusivo y el de fin es exclusivo:**

```java
String s = "Hello World";
System.out.println(s.substring(0, 5));   // Hello
System.out.println(s.substring(6));      // World
```

La forma de recordar el rango: `substring(a, b)` devuelve `b - a` caracteres. `substring(0, 5)` devuelve 5.

### Las excepciones

Los mensajes reales del JDK 25 son bastante claros:

```java
"abc".substring(2, 1);
// StringIndexOutOfBoundsException: Range [2, 1) out of bounds for length 3

"abc".charAt(5);
// StringIndexOutOfBoundsException: Index 5 out of bounds for length 3
```

*(Verificado en JDK 25.)*

Tres reglas para no toparte con ellas: el inicio no puede ser mayor que el fin, el fin no puede superar `length()`, y ninguno puede ser negativo. Un `substring(0, length())` es válido; un `substring(0, length() + 1)` no.

Nota: `substring(0)` y `substring(0, length())` devuelven **el mismo objeto**, no una copia:

```java
String base = "abcdef";
System.out.println(base.substring(0) == base);      // true
System.out.println(base.substring(0, 6) == base);   // true
```

*(Verificado en JDK 25.)* Es una optimización: si el rango cubre todo el string y este es inmutable, no hace falta copiar nada.

### El caso histórico: la fuga de memoria de `substring`

Este es un clásico de entrevistas senior y explica una diferencia de rendimiento real entre versiones.

**Hasta Java 6**, `substring()` no copiaba nada: el nuevo `String` compartía el `char[]` del original y guardaba un `offset` y un `count`. Era O(1), rapidísimo. Pero tenía una consecuencia grave:

```java
// En Java 6: 'inicial' mantiene vivo el array de 10 MB completo
String textoEnorme = leerArchivoDe10MB();
String inicial = textoEnorme.substring(0, 10);
// aunque textoEnorme salga de scope, su char[] no se puede liberar
```

Diez caracteres retenían diez megabytes. Era una fuga de memoria silenciosa y muy difícil de diagnosticar.

**Desde Java 7u6**, `substring()` copia el rango con `Arrays.copyOfRange`. Ahora es O(n) —más lento— pero cada `String` es completamente independiente y el original puede recolectarse.

La conclusión práctica para hoy: **`substring` tiene coste proporcional a la longitud del resultado.** Llamarlo dentro de un bucle sobre un texto grande no es gratis. Si solo necesitás comparar una porción, `regionMatches` ([sección 15](#15-contentequals-regionmatches-y-matches)) evita la copia.

## 20. `trim()` vs `strip()`

Los dos eliminan espacios al principio y al final. La diferencia parece cosmética y no lo es.

```java
String texto = "  Y corrió por el campo   ";
System.out.println(texto.trim());    // "Y corrió por el campo"
System.out.println(texto.strip());   // "Y corrió por el campo"
```

Los espacios internos **no** se tocan en ningún caso, y ambos devuelven un objeto nuevo.

### La diferencia real

- **`trim()`** (Java 1.0) elimina todo carácter con código **menor o igual a `U+0020`** (el espacio ASCII). Es una definición de los años noventa, anterior a que Unicode fuera una preocupación.
- **`strip()`** (Java 11) elimina todo lo que `Character.isWhitespace()` considere espacio, incluidos los espacios de otros sistemas de escritura.

Verificado en JDK 25 con un espacio ideográfico (`U+3000`, el espacio usado en japonés y chino):

```java
String ideo = "　hola　";          // U+3000 a ambos lados, longitud 6
System.out.println(ideo.trim().length());    // 6  <- trim NO lo eliminó
System.out.println(ideo.strip().length());   // 4  <- strip sí
```

Si procesás texto que puede venir en cualquier idioma, `trim()` te deja basura invisible.

### El caso que ninguno resuelve: el espacio duro

Acá está el hallazgo que más problemas causa en la práctica. El **espacio de no separación** (`U+00A0`, *non-breaking space*) es el que producen HTML (`&nbsp;`), Word y muchos sistemas de copiar-pegar. Verificado en JDK 25:

```java
String nbsp = " hola ";              // longitud 6
System.out.println(nbsp.trim().length());      // 6  <- no lo elimina
System.out.println(nbsp.strip().length());     // 6  <- tampoco
```

**Ni `trim()` ni `strip()` lo eliminan**, porque `Character.isWhitespace(' ')` es `false` — Unicode lo clasifica deliberadamente como "no separador", ya que su función es *impedir* el corte de línea.

Este es el motivo real detrás de un bug recurrente: un usuario pega un valor desde una página web o un Excel, tu código hace `.trim()`, la comparación falla, y el string "se ve idéntico" en el log. Si el dato viene de fuentes no controladas:

```java
// Normalizar cualquier espacio Unicode antes de validar
String limpio = valor.replaceAll("[\\p{Z}\\s]+", " ").strip();
```

`\p{Z}` es la categoría Unicode de separadores, que sí incluye `U+00A0`.

### Variantes direccionales

Java 11 agregó las versiones de un solo lado:

```java
"  x  ".stripLeading();    // "x  "
"  x  ".stripTrailing();   // "  x"
```

Útiles cuando la indentación importa pero los espacios finales no.

---

# Parte V — Transformar

## 21. Toda transformación devuelve un objeto nuevo

Antes de recorrer los métodos, conviene repetir la regla de la [sección 4](#4-inmutabilidad-la-decisión-que-explica-todo-lo-demás) porque aplica a **todos** los que siguen:

```java
String original = "  Hola Mundo  ";

original.trim();          // no hace nada útil: el resultado se descarta
original.toUpperCase();   // idem
original.replace("a", "e");

System.out.println(original);   // "  Hola Mundo  " — intacto
```

La forma correcta es asignar, y como cada método devuelve un `String`, se pueden encadenar:

```java
String limpio = original.strip()
                        .toUpperCase(Locale.ROOT)
                        .replace(" ", "_");
System.out.println(limpio);   // HOLA_MUNDO
```

Cada eslabón de esa cadena crea un objeto intermedio. Para tres operaciones sobre un string corto, irrelevante. Dentro de un bucle sobre un millón de líneas, es exactamente el tipo de cosa que se mide antes de optimizar.

## 22. Mayúsculas, minúsculas y el bug del locale turco

```java
String txt = "Hello World";
System.out.println(txt.toUpperCase());   // HELLO WORLD
System.out.println(txt.toLowerCase());   // hello world
```

Hasta acá, aburrido. Ahora la parte importante.

### El bug

**`toUpperCase()` y `toLowerCase()` sin argumentos usan el locale por defecto de la JVM**, que depende de la configuración del sistema operativo donde corre el programa. Y hay un idioma donde las reglas de mayúsculas son distintas: el turco.

El turco tiene cuatro letras "i": `i`/`İ` (con punto) y `ı`/`I` (sin punto). Convertir `i` a mayúscula da `İ`, no `I`.

Verificado en JDK 25:

```java
Locale tr = Locale.forLanguageTag("tr");

System.out.println("title".toUpperCase(tr));            // TİTLE   (!)
System.out.println("title".toUpperCase(Locale.ROOT));   // TITLE
System.out.println("TITLE".toLowerCase(tr));            // tıtle   (!)

System.out.println("title".toUpperCase(tr).equals("TITLE"));   // false
```

### Por qué esto tumba aplicaciones enteras

Porque el patrón afectado está en todas partes:

```java
// Normalización de una cabecera HTTP
if (header.toUpperCase().equals("CONTENT-TYPE")) { ... }

// Comparación de un comando
if (comando.toLowerCase().equals("list")) { ... }

// Construcción del nombre de un método por reflexión
String getter = "get" + campo.substring(0, 1).toUpperCase() + campo.substring(1);
```

En un servidor configurado con locale turco, **todos** fallan. Y el fallo no es un error de compilación ni una excepción: es una comparación que da `false` donde debería dar `true`. La aplicación funciona en todo el mundo y se rompe solo para un país, lo que hace el diagnóstico desesperante.

Es un bug lo bastante famoso como para tener nombre propio ("the Turkish locale bug") y para haber afectado a Gradle, Apache Commons, Appium y un largo etcétera. En OpenJDK llegó a proponerse **deprecar** `toLowerCase()` y `toUpperCase()` sin argumentos ([JDK-8305907](https://bugs.openjdk.org/browse/JDK-8305907)).

### La regla

> **Si la transformación es para lógica interna (comparar, construir claves, normalizar identificadores), usá siempre `Locale.ROOT`. Si es para mostrar al usuario, usá el locale del usuario.**

```java
// Lógica interna: determinista, igual en cualquier máquina
String clave = valor.toUpperCase(Locale.ROOT);

// Presentación: respeta el idioma del usuario
String titulo = nombre.toUpperCase(locale Usuario);
```

Y para comparar, mejor todavía: no conviertas nada, usá `equalsIgnoreCase()` ([sección 12](#12-equalsignorecase-y-sus-límites)).

### El otro detalle: la longitud puede cambiar

Convertir mayúsculas no es una operación uno a uno. Verificado en JDK 25:

```java
System.out.println("ß".toUpperCase());       // SS      longitud 1 -> 2
System.out.println("ﬃ".toUpperCase());       // FFI     longitud 1 -> 3
```

Si tu código asume que `s.toUpperCase().length() == s.length()`, está mal. Es raro que importe, pero cuando importa (buffers de tamaño fijo, columnas de base de datos con límite) importa mucho.

## 23. `replace` vs `replaceAll` vs `replaceFirst`

Los nombres de estos tres métodos son, sin exagerar, un error de diseño de la API. Lo que los diferencia **no** es cuántas ocurrencias reemplazan.

| Método | Qué recibe | Cuántas reemplaza |
|---|---|---|
| `replace(char, char)` | Caracteres literales | **Todas** |
| `replace(CharSequence, CharSequence)` | Texto literal | **Todas** |
| `replaceAll(String, String)` | **Regex** | Todas |
| `replaceFirst(String, String)` | **Regex** | Solo la primera |

**`replace` reemplaza todas las ocurrencias, igual que `replaceAll`.** La diferencia real es que `replace` trata el argumento como texto literal y `replaceAll` lo trata como expresión regular.

### La demostración

```java
System.out.println("1.2.3".replace(".", "-"));      // 1-2-3
System.out.println("1.2.3".replaceAll(".", "-"));   // -----
```

*(Salida verificada en JDK 25.)*

En el segundo caso, `.` es el metacarácter de regex que significa "cualquier carácter", así que reemplazó los cinco caracteres. Este es el bug: alguien quiere cambiar puntos por guiones, elige `replaceAll` porque "quiero todas", y destruye el string.

### La regla

> **Si no necesitás una expresión regular, usá `replace`.** Es más rápido (no compila una regex) y no tiene sorpresas con metacaracteres.

Los metacaracteres que hay que escapar en regex son: `. ^ $ * + ? ( ) [ ] { } | \`

Si el patrón viene de una variable, usá `Pattern.quote`:

```java
String buscado = obtenerDeUsuario();       // podría contener "."
texto.replaceAll(Pattern.quote(buscado), "X");   // trata todo como literal
```

### La segunda trampa: el reemplazo también es especial

En `replaceAll` y `replaceFirst`, **la cadena de reemplazo tampoco es literal**. `$` introduce referencias a grupos capturados y `\` escapa. Verificado en JDK 25:

```java
"x".replaceAll("x", "$1");
// IndexOutOfBoundsException: No group 1
```

Si querés un `$` literal en el reemplazo hay que escaparlo:

```java
System.out.println("precio".replaceAll("precio", "10\\$"));   // 10$
```

Este error aparece con formatos de moneda y con contraseñas generadas que contienen `$`. La solución general:

```java
texto.replaceAll(regex, Matcher.quoteReplacement(reemplazo));
```

### Los grupos, cuando sí los querés

El lado positivo de esa sintaxis es que permite reordenar contenido:

```java
String fecha = "2026-08-19";
System.out.println(fecha.replaceAll("(\\d{4})-(\\d{2})-(\\d{2})", "$3/$2/$1"));
// 19/08/2026
```

`$1`, `$2`, `$3` son los grupos capturados por los paréntesis. Es una forma compacta de reformatear sin parsear.

### `replaceFirst`

```java
String texto = "one two three two one";
System.out.println(texto.replaceFirst("two", "five"));   // one five three two one
System.out.println(texto.replaceAll("two", "five"));     // one five three five one
```

## 24. `split()`: el método que más sorprende

`split()` divide un string en un array usando un delimitador:

```java
String[] partes = "peter,james,thomas".split(",");
// [peter, james, thomas]
```

Parece trivial. Tiene al menos cuatro comportamientos que sorprenden, y todos producen bugs reales.

### Sorpresa 1: el argumento es una regex

Igual que `replaceAll`. Esto rompe con separadores comunes. Verificado en JDK 25:

```java
System.out.println(Arrays.toString("192.168.1.1".split(".")));    // []   longitud 0 (!)
System.out.println(Arrays.toString("192.168.1.1".split("\\.")));  // [192, 168, 1, 1]

System.out.println(Arrays.toString("a|b".split("|")));            // [a, |, b]  (!)
System.out.println(Arrays.toString("a|b".split("\\|")));          // [a, b]
```

Dividir por `.` devuelve un **array vacío**: como `.` coincide con todo, cada carácter es un separador, todos los fragmentos quedan vacíos, y (por la sorpresa 2) se eliminan todos. Dividir por `|` es peor: `|` significa "o" en regex, así que el patrón es "vacío o vacío", y el resultado incluye el propio separador.

Los separadores que necesitan escape son los mismos metacaracteres de la sección anterior: `. | ( ) [ ] { } ^ $ * + ? \`

### Sorpresa 2: los campos vacíos finales desaparecen

Esta es la que corrompe datos de forma silenciosa. Verificado en JDK 25:

```java
System.out.println(Arrays.toString("a,b,,,".split(",")));       // [a, b]
System.out.println(Arrays.toString("a,b,,,".split(",", -1)));   // [a, b, , , ]
System.out.println(Arrays.toString(",,a,b".split(",")));        // [, , a, b]
```

Por defecto, **`split` elimina los campos vacíos del final**, pero conserva los del principio. Es una asimetría difícil de justificar y muy fácil de sufrir:

```java
// CSV donde las últimas columnas son opcionales
String linea = "Ana,Pérez,,";              // apellido2 y teléfono vacíos
String[] campos = linea.split(",");
System.out.println(campos.length);         // 2, no 4
String telefono = campos[3];               // ArrayIndexOutOfBoundsException
```

**La solución es pasar un límite negativo**, que conserva todos los campos:

```java
String[] campos = linea.split(",", -1);    // longitud 4
```

> **Regla:** si parseás datos estructurados donde la posición de las columnas importa, usá **siempre** `split(regex, -1)`.

### Sorpresa 3: el string vacío devuelve un array de tamaño 1

```java
System.out.println("".split(",").length);   // 1, no 0
```

*(Verificado en JDK 25.)* El único elemento es el string vacío. Si tu código hace `if (partes.length == 0)` para detectar entrada vacía, nunca entra.

### El parámetro `limit`, completo

```java
String s = "A man drove with a car.";
System.out.println(Arrays.toString(s.split("a", 2)));
// [A m, n drove with a car.]
```

*(Verificado en JDK 25.)*

- **`limit > 0`**: aplica el patrón como mucho `limit - 1` veces; el resto queda entero en el último elemento. Útil para separar `clave=valor` cuando el valor puede contener `=`.
- **`limit == 0`** (por defecto): aplica el patrón todas las veces y **descarta los vacíos finales**.
- **`limit < 0`**: aplica el patrón todas las veces y **conserva todo**.

El caso de `limit = 2` tiene un uso cotidiano:

```java
String config = "url=https://ejemplo.com/a=b";
String[] par = config.split("=", 2);
// [url, https://ejemplo.com/a=b]   <- correcto
```

### Sobre precompilar el `Pattern`

Un consejo muy repetido es "no uses `split`, precompilá un `Pattern`". Lo medí en JDK 25 con 200.000 repeticiones sobre `"a,b,c,d,e,f,g,h"`:

```
String.split(",")   : 62,1 ms
Pattern.split(csv)  : 67,3 ms
```

**No hubo mejora.** La razón es que `String.split` tiene un *fast path*: cuando el patrón es un único carácter que no es metacarácter (o dos caracteres donde el primero es `\`), no compila ninguna regex y hace la división directamente.

La conclusión matizada: para delimitadores simples, `split` está bien y precompilar no aporta. Para patrones **reales** (`"\\s*,\\s*"`, alternativas, cuantificadores), precompilar sí evita recompilar la regex en cada llamada y la diferencia se nota.

### Alternativas para casos reales

```java
// Dividir y limpiar espacios en un paso
String[] partes = " car , jeep, scooter ".trim().split("\\s*,\\s*");

// Con streams, si además querés filtrar
List<String> limpias = Arrays.stream(entrada.split(","))
                             .map(String::trim)
                             .filter(s -> !s.isEmpty())
                             .toList();
```

Y una advertencia importante: **para CSV real, no uses `split`.** Un CSV admite comas dentro de comillas, comillas escapadas y saltos de línea dentro de un campo. `"Pérez, Ana",30` tiene dos campos, no tres. Para eso existen librerías (OpenCSV, Apache Commons CSV, Jackson CSV). `split` sirve para formatos simples que vos controlás.

## 25. Unir: `join` y `StringJoiner`

La operación inversa de `split`.

### `String.join`

```java
System.out.println(String.join(" - ", "a", "b", "c"));       // a - b - c
System.out.println(String.join(",", List.of("x", "y")));     // x,y
```

*(Verificado en JDK 25.)*

Acepta tanto varargs como cualquier `Iterable`. Es la forma más simple y la que deberías usar por defecto. Reemplaza al patrón manual con bucle y borrado del último separador:

```java
// El anti-patrón que String.join elimina
StringBuilder sb = new StringBuilder();
for (String s : lista) {
    sb.append(s).append(",");
}
sb.deleteCharAt(sb.length() - 1);   // borrar la coma sobrante... si la lista no está vacía
```

Ese `deleteCharAt` lanza excepción con lista vacía. `String.join` no tiene ese problema.

### `StringJoiner`

Cuando además necesitás prefijo y sufijo:

```java
StringJoiner sj = new StringJoiner(", ", "[", "]");
sj.add("uno").add("dos");
System.out.println(sj);      // [uno, dos]
```

Y tiene un detalle bien pensado para el caso vacío:

```java
StringJoiner vacio = new StringJoiner(", ", "[", "]");
vacio.setEmptyValue("VACIO");
System.out.println(vacio);   // VACIO   (en vez de "[]")
```

*(Verificado en JDK 25.)*

### Con streams

```java
String csv = usuarios.stream()
                     .map(Usuario::nombre)
                     .collect(Collectors.joining(", ", "[", "]"));
```

`Collectors.joining` usa `StringJoiner` por dentro. Es la forma idiomática cuando ya estás en un pipeline de streams.

## 26. La API moderna: `repeat`, `lines`, `indent` y compañía

Java 11 a 15 agregaron métodos que eliminan mucho código repetitivo. Todos verificados en JDK 25.

```java
System.out.println("ab".repeat(3));          // ababab      (Java 11)
System.out.println("a\nb\nc".lines().count());  // 3         (Java 11)
System.out.println("x".indent(4));           // "    x\n"    (Java 12)
System.out.println("Hola %s, %d".formatted("Ana", 3));   // Hola Ana, 3   (Java 15)
```

### `repeat(n)`

Reemplaza el bucle manual. Útil para separadores y padding:

```java
System.out.println("-".repeat(40));    // línea separadora
```

### `lines()`

Devuelve un `Stream<String>` con las líneas, y **maneja correctamente los tres finales de línea** (`\n`, `\r\n`, `\r`), a diferencia de `split("\n")` que deja un `\r` colgando en archivos de Windows:

```java
long noVacias = contenido.lines()
                         .filter(l -> !l.isBlank())
                         .count();
```

Ventaja adicional: es perezoso, así que procesar un archivo enorme no materializa un array con todas las líneas.

### `stripIndent()` y `translateEscapes()`

Dos métodos de Java 15 que existen sobre todo para dar soporte a los text blocks, pero se pueden usar sueltos:

```java
String ind = "   Hey\n   This\n   is\n   indented.";
System.out.println(ind.stripIndent());
// Hey
// This
// is
// indented.
```

`stripIndent()` calcula la indentación mínima común y la elimina de todas las líneas.

```java
String esc = "Hey,\\n no es salto real.";
System.out.println(esc);                      // Hey,\n no es salto real.
System.out.println(esc.translateEscapes());   // Hey,
                                              //  no es salto real.
```

*(Verificado en JDK 25.)*

`translateEscapes()` interpreta las secuencias de escape que están **como texto** dentro del string. Sirve cuando leés de un archivo de configuración un valor como `separador=\t` y necesitás el tabulador real, no las dos letras.

### `chars()` y `codePoints()`

Convierten el string en un stream de enteros:

```java
System.out.println("hola".chars()
                         .mapToObj(ch -> (char) ch)
                         .map(String::valueOf)
                         .collect(Collectors.joining("|")));   // h|o|l|a
```

*(Verificado en JDK 25.)*

Fijate en el detalle molesto: `chars()` devuelve un `IntStream`, no un `Stream<Character>`, así que hay que castear. La diferencia entre `chars()` y `codePoints()` es central para Unicode y la vemos en la [sección 33](#33-code-points-y-surrogate-pairs).

---

# Parte VI — Construir strings

## 27. El coste real de construir texto

Acá es donde la inmutabilidad pasa la factura.

Concatenar dos strings crea uno nuevo copiando ambos. Hacerlo una vez es barato. Hacerlo en un bucle es cuadrático: en la iteración *n* copiás *n* caracteres, así que el total es 1 + 2 + 3 + ... + n, es decir O(n²).

Lo medí en JDK 25 con 40.000 iteraciones:

```java
String acc = "";
for (int i = 0; i < 40_000; i++) acc = acc + "x";
```

```
concat  : 66,9 ms
builder :  0,73 ms
ratio   : 91x
```

**91 veces más lento**, y la diferencia crece con el tamaño: con 400.000 iteraciones no serían 10 veces más, serían cerca de 100 veces más. Es el patrón que funciona en desarrollo con datos de prueba y colapsa en producción.

La versión correcta:

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 40_000; i++) sb.append("x");
String resultado = sb.toString();
```

### Dónde aparece esto en código real

```java
// Construir una consulta
String sql = "SELECT * FROM t WHERE 1=1";
for (Filtro f : filtros) {
    sql = sql + " AND " + f.columna() + " = ?";     // MAL
}

// Serializar una colección
String csv = "";
for (Registro r : registros) {
    csv = csv + r.toCsvLine() + "\n";               // MAL
}

// Construir un mensaje en un bucle de procesamiento
for (Error e : errores) {
    mensaje += e.getDescripcion() + "; ";           // MAL (+= es lo mismo)
}
```

`+=` sobre un `String` dentro de un bucle es exactamente el mismo problema.

## 28. `StringBuilder` y `StringBuffer`

`StringBuilder` es un **buffer de caracteres mutable**. A diferencia de `String`, `append` modifica el objeto existente:

```java
String inmutable = "abc";
inmutable = inmutable + "def";        // crea un objeto nuevo

StringBuilder sb = new StringBuilder("abc");
sb.append("def");                     // modifica el mismo objeto
```

### La API esencial

```java
StringBuilder sb = new StringBuilder("abc");
sb.append(1).append(true)      // append acepta cualquier tipo
  .insert(0, ">>")             // insertar en una posición
  .reverse();                  // invertir
System.out.println(sb);        // eurt1cba>>
```

*(Verificado en JDK 25.)*

Todos los métodos devuelven `this`, así que se encadenan. Otros útiles: `delete(int, int)`, `deleteCharAt(int)`, `replace(int, int, String)`, `setLength(int)`, `setCharAt(int, char)`, `indexOf(String)`.

`setLength(0)` es la forma idiomática de reutilizar el mismo buffer en un bucle externo sin reasignar memoria.

### La capacidad

`StringBuilder` guarda un array interno que crece cuando se llena. Verificado en JDK 25:

```java
new StringBuilder().capacity();          // 16
new StringBuilder("abc").capacity();     // 19  (16 + longitud del contenido inicial)
```

Cada vez que se llena, se asigna un array mayor y se copia todo. Si sabés el tamaño aproximado, dárselo evita esas copias:

```java
StringBuilder sb = new StringBuilder(estimacion);
```

Es una micro-optimización real pero menor: solo vale la pena en bucles calientes con tamaños grandes y conocidos.

### `StringBuffer`: el hermano viejo

`StringBuffer` existe desde Java 1.0 y tiene **la misma API**, con una diferencia: todos sus métodos son `synchronized`, es decir, thread-safe. `StringBuilder` se agregó en Java 5 como versión sin sincronizar.

| | `StringBuilder` | `StringBuffer` |
|---|---|---|
| Desde | Java 5 | Java 1.0 |
| Thread-safe | No | Sí (`synchronized`) |
| Velocidad | Mayor | Menor |
| Cuándo usar | **Siempre, por defecto** | Prácticamente nunca |

> **Regla: usá `StringBuilder`.** La sincronización de `StringBuffer` cuesta y casi nunca sirve: un buffer que se construye dentro de un método es una variable local, y las variables locales no se comparten entre hilos ([Variables and Scopes](04-variables-and-scopes.md)). Si de verdad varios hilos escriben en el mismo buffer, la sincronización por método de `StringBuffer` casi seguro tampoco alcanza — necesitarías coordinar la secuencia completa de operaciones, no cada `append` por separado.

### El error del `equals`

Ya apareció en la [sección 15](#15-contentequals-regionmatches-y-matches) pero merece repetirse porque es sutil:

```java
new StringBuilder("a").equals(new StringBuilder("a"));   // false
```

`StringBuilder` **no** sobrescribe `equals()`. Compará con `.toString().equals(...)` o con `contentEquals`.

## 29. Concatenación con `+`: qué genera realmente el compilador

Esta sección corrige una afirmación que aparece en fuentes muy populares y que **fue cierta hasta Java 8**.

### Lo que dicen los tutoriales

Que el compilador traduce esto:

```java
String tres = uno + " " + dos;
```

en esto:

```java
String tres = new StringBuilder(uno).append(" ").append(dos).toString();
```

Eso era exacto **hasta Java 8**. Desde Java 9 ya no lo es.

### Lo que hace el compilador hoy

[JEP 280 "Indify String Concatenation"](https://openjdk.org/jeps/280), incorporado en Java 9, cambió la estrategia. Compilé este método con JDK 25:

```java
static String demoConcat(String nombre, int edad) {
    return "Hola " + nombre + ", tenes " + edad + " anios";
}
```

Y el bytecode real (`javap -c`) es:

```
static java.lang.String demoConcat(java.lang.String, int);
  Code:
     0: aload_0
     1: iload_1
     2: invokedynamic #7,  0    // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;I)Ljava/lang/String;
     7: areturn
```

**No hay ningún `StringBuilder`.** Hay una única instrucción `invokedynamic` que delega en el método `makeConcatWithConstants` de `java.lang.invoke.StringConcatFactory`.

### Por qué se hizo así

Porque la estrategia de concatenación queda **fuera del bytecode**. Antes, el `.class` contenía las llamadas a `StringBuilder` grabadas a fuego: mejorar la concatenación exigía recompilar. Ahora el `.class` solo dice "concatená estos argumentos con esta receta", y en tiempo de ejecución la JVM elige la implementación. Puede calcular el tamaño exacto del resultado y asignar el array una sola vez, sin los redimensionamientos de `StringBuilder`.

El resultado práctico: **la concatenación de una sola expresión es hoy más rápida que escribir el `StringBuilder` a mano**. Ese es el consejo actual y va en contra de lo que enseñan muchos tutoriales.

```java
// BIEN: legible y rápido
String mensaje = "Usuario " + id + " creado en " + fecha;

// PEOR: más largo y no más rápido
String mensaje = new StringBuilder().append("Usuario ").append(id)
                                    .append(" creado en ").append(fecha).toString();
```

### Entonces, ¿el consejo del bucle sigue vigente?

**Sí, y esto es lo importante.** Mirá el bytecode del bucle, también de JDK 25:

```
26: invokedynamic #13,  0   // InvokeDynamic #1:makeConcatWithConstants:(...)
31: astore_1
35: goto          11
```

La llamada `invokedynamic` está **dentro** del bucle: se ejecuta una vez por iteración, y cada una produce un `String` nuevo copiando todo el acumulado. La optimización de JEP 280 aplica a **una expresión de concatenación**, no puede fusionar iteraciones distintas.

La regla completa queda así:

| Situación | Qué usar |
|---|---|
| Una expresión, aunque tenga muchos `+` | **`+`** — legible y óptimo |
| Concatenar dentro de un bucle | **`StringBuilder`** |
| Unir una colección con separador | **`String.join` / `Collectors.joining`** |
| Plantilla con formato | **`formatted()` / `String.format`** |
| Logging | **placeholders `{}` del logger** ([sección 31](#31-concatenación-en-logging-el-caso-que-sí-importa)) |

> **Nota sobre la fuente.** El tutorial de Jenkov —una de las tres referencias de este documento— sigue afirmando que la concatenación se traduce a `StringBuilder`. Fue cierto durante años y hoy no lo es. Su página está fechada en 2020, once años después de Java 9. Es un buen recordatorio de que en Java conviene verificar contra el JDK que usás, no contra el tutorial mejor posicionado.

## 30. `String.format()` y `formatted()`

Cuando necesitás una plantilla con valores insertados y control sobre el formato:

```java
String saludo = String.format("Hola %s, bienvenido a %s!", nombre, sitio);
```

Desde Java 15 existe la versión de instancia, que se lee mejor con text blocks:

```java
String saludo = "Hola %s, bienvenido a %s!".formatted(nombre, sitio);
```

*(Verificado en JDK 25.)* Son equivalentes; `formatted()` evita el prefijo `String.format(`.

### La sintaxis del especificador

```
%[índice$][flags][ancho][.precisión]conversión
```

Las conversiones que vas a usar:

| Especificador | Para qué |
|---|---|
| `%s` | Cualquier objeto (llama a `toString()`) |
| `%d` | Enteros |
| `%f` | Decimales |
| `%.2f` | Decimales con 2 posiciones |
| `%,d` | Enteros con separador de miles |
| `%x` / `%o` | Hexadecimal / octal |
| `%b` | Booleano |
| `%n` | Salto de línea **de la plataforma** |
| `%%` | Un `%` literal |

Ejemplos verificados en JDK 25:

```java
String.format("%-10s|%10s", "izq", "der");    // "izq       |       der"
String.format(Locale.US, "%,.2f", 1234567.891);  // "1,234,567.89"
```

El `-` alinea a la izquierda; el número es el ancho mínimo. Es lo que se usa para tablas en consola.

**El índice de argumento** permite reutilizar un valor:

```java
String.format("Hola %2$s, bienvenido a %1$s!", "Baeldung", "Folks");
// Hola Folks, bienvenido a Baeldung!
```

### El locale, otra vez

`String.format` sin locale usa el locale por defecto de la JVM. Verificado en JDK 25:

```java
String.format(Locale.ROOT, "%.2f", 1234.5);                    // 1234.50
String.format(Locale.forLanguageTag("es-ES"), "%.2f", 1234.5); // 1234,50
```

**El separador decimal cambia.** Si ese string va a un archivo CSV, a un JSON o a una API, generar `1234,50` en un servidor con locale español rompe el consumidor. Regla idéntica a la de `toUpperCase`:

> Formato para máquinas → `Locale.ROOT`. Formato para humanos → locale del usuario.

Para dinero, ni una cosa ni la otra: usá `NumberFormat.getCurrencyInstance(locale)` o `BigDecimal` con `DecimalFormat`, que manejan redondeo y símbolo de moneda correctamente.

### Cuándo no usar `format`

`String.format` es notablemente más lento que la concatenación: parsea la plantilla en cada llamada. Para un mensaje ocasional da igual; en un bucle caliente o en logging, no ([sección 31](#31-concatenación-en-logging-el-caso-que-sí-importa)).

## 31. Concatenación en logging: el caso que sí importa

Este patrón está en todos los proyectos y casi siempre mal:

```java
log.debug("Procesando pedido " + pedido.getId() + " del cliente " + cliente.getNombre());
```

El problema: **la concatenación se ejecuta siempre**, incluso si el nivel `DEBUG` está desactivado. Java evalúa los argumentos antes de llamar al método. Construís el string, se lo pasás al logger, y el logger lo descarta.

En un endpoint con 10.000 peticiones por segundo y varios `log.debug` desactivados, eso es trabajo puro de CPU y presión sobre el recolector de basura, a cambio de nada.

### La solución: placeholders

SLF4J (y Log4j2) aceptan plantillas con `{}`:

```java
log.debug("Procesando pedido {} del cliente {}", pedido.getId(), cliente.getNombre());
```

Ahora el mensaje **solo se construye si el nivel está activo**. Si `DEBUG` está apagado, el logger comprueba el nivel, ve que no corresponde, y devuelve sin formatear nada.

### `String.format` en logging es peor todavía

```java
log.debug(String.format("Pedido %s", id));    // MAL
```

Combina lo peor: se ejecuta siempre **y** el parseo de la plantilla de `format` es más caro que la concatenación. SonarQube marca esto (regla S2629).

### El caso que los placeholders no cubren

Si calcular el argumento es caro, el placeholder no ayuda: el argumento se evalúa igual.

```java
log.debug("Estado: {}", calcularEstadoCompleto());   // se ejecuta siempre
```

Dos soluciones:

```java
// 1. Guarda explícita
if (log.isDebugEnabled()) {
    log.debug("Estado: {}", calcularEstadoCompleto());
}

// 2. Supplier (Log4j2, SLF4J 2.x): se evalúa solo si hace falta
log.debug("Estado: {}", () -> calcularEstadoCompleto());
```

---

# Parte VII — Unicode: cuando el texto deja de ser ASCII

Esta parte es la que separa a alguien que "sabe usar strings" de alguien que puede escribir software que funciona en todo el mundo. También es la que ninguna de las tres fuentes de referencia cubre con profundidad.

## 32. `char` no es un carácter

Empecemos por desarmar la suposición central.

`char` en Java es un entero sin signo de **16 bits**. Puede representar valores de `0` a `65535`. Cuando Java se diseñó, se creía que 16 bits alcanzaban para todos los caracteres de todos los idiomas — la promesa original de Unicode.

Esa promesa se rompió en 1996, cuando Unicode 2.0 amplió el rango a más de un millón de posiciones. Java ya estaba publicado, con `char` de 16 bits en su especificación y en la firma de cientos de métodos. Cambiarlo era imposible.

La solución fue **UTF-16 con pares suplentes** (*surrogate pairs*): los caracteres que no entran en 16 bits se representan con **dos** `char` consecutivos.

Consecuencia directa:

> **Un `char` no es un carácter. Es una unidad de código UTF-16. Para el texto ASCII coinciden; para el resto, no necesariamente.**

Esto no es teórico. Los emojis, buena parte de los ideogramas CJK poco frecuentes, los alfabetos históricos y los símbolos matemáticos viven fuera de los primeros 65.536 puntos.

## 33. Code points y surrogate pairs

Los términos que hay que separar:

- **Code point**: el número que Unicode asigna a un carácter. El emoji 😀 es `U+1F600`.
- **Code unit** (unidad de código): cada `char` de 16 bits en la representación interna.
- Un code point ≤ `U+FFFF` ocupa **1** code unit. Uno mayor ocupa **2** (un par suplente).

### La demostración

Verificado en JDK 25 con el string `"Hi 😀"` — que cualquier humano diría que tiene 4 caracteres:

```java
String emoji = "Hi 😀";

System.out.println(emoji.length());                                // 5   (!)
System.out.println(emoji.codePointCount(0, emoji.length()));       // 4
System.out.println(emoji.chars().count());                         // 5
System.out.println(emoji.codePoints().count());                    // 4
System.out.println(emoji.charAt(3));                               // ?   (medio emoji)
System.out.println(emoji.substring(0, 4));                         // Hi ? (roto)
```

`length()` devuelve **5** porque el emoji ocupa dos `char`. Y `charAt(3)` devuelve la **primera mitad** del par suplente, que por sí sola no representa nada — por eso se imprime como `?`.

### Los bugs que esto produce

**1. Límites de caracteres mal calculados.**

```java
// Un tweet de 280 caracteres con emojis se rechaza antes de tiempo
if (mensaje.length() > 280) { rechazar(); }        // MAL
if (mensaje.codePointCount(0, mensaje.length()) > 280) { rechazar(); }   // mejor
```

**2. Truncado que parte un carácter por la mitad.**

```java
// Si el corte cae en medio de un par suplente, generás texto inválido
String preview = texto.substring(0, 100);          // MAL
```

El resultado tiene un `char` suplente huérfano. Al serializarlo a JSON o guardarlo en base de datos, aparece el rombo con el signo de pregunta, o directamente un error de codificación.

La forma segura:

```java
// Ajustar el corte al límite de code point más cercano
int fin = texto.offsetByCodePoints(0, Math.min(100, texto.codePointCount(0, texto.length())));
String preview = texto.substring(0, fin);
```

**3. Iterar con `charAt`.**

```java
// MAL: procesa medio emoji en cada mitad
for (int i = 0; i < s.length(); i++) {
    procesar(s.charAt(i));
}

// BIEN: itera por code points
s.codePoints().forEach(cp -> procesar(cp));
```

### Lo que sí funciona

`StringBuilder.reverse()` respeta los pares suplentes. Verificado:

```java
new StringBuilder("Hi 😀").reverse();   // "😀 iH"  — el emoji queda intacto
```

Está explícitamente documentado que no invierte las mitades de un par. Es un detalle de calidad de la biblioteca estándar que conviene conocer, porque la implementación ingenua de "invertir un string" que mucha gente escribe a mano sí rompe el emoji.

## 34. Grafemas: lo que el usuario llama "un carácter"

Hay un tercer nivel, por encima del code point. Verificado en JDK 25 con la bandera argentina 🇦🇷:

```java
String bandera = "🇦🇷";

System.out.println(bandera.length());                                  // 4
System.out.println(bandera.codePointCount(0, bandera.length()));       // 2
// grafemas contados con BreakIterator                                 // 1
```

Una sola cosa visible. Dos code points (las letras regionales `A` y `R`, que el sistema dibuja como bandera). Cuatro `char`.

Ese "una sola cosa visible" se llama **grafema** (*grapheme cluster*), y es lo que un usuario llama "un carácter". Aparece también en emojis con modificador de tono de piel, familias, y letras con acentos combinantes.

Para contarlos hay que usar `BreakIterator`:

```java
BreakIterator bi = BreakIterator.getCharacterInstance();
bi.setText(texto);
int grafemas = 0;
while (bi.next() != BreakIterator.DONE) {
    grafemas++;
}
```

### Los tres niveles, resumidos

| Nivel | Cómo se cuenta | `"Hi 😀"` | `"🇦🇷"` |
|---|---|---|---|
| Code units (`char`) | `length()` | 5 | 4 |
| Code points | `codePointCount()` | 4 | 2 |
| Grafemas | `BreakIterator` | 4 | 1 |

**Cuál usar:** para un límite que ve el usuario ("máximo 20 caracteres"), grafemas. Para procesamiento de texto, code points. Para índices y buffers internos, code units. `length()` es lo correcto solo cuando sabés que el texto es ASCII.

## 35. Normalización: dos textos idénticos que no son iguales

Este es el bug más difícil de diagnosticar de todo el documento, porque **el texto se ve exactamente igual**.

Unicode permite escribir `é` de dos formas:

1. **NFC (precompuesta):** un solo code point, `U+00E9`.
2. **NFD (descompuesta):** dos code points, `e` (`U+0065`) + acento combinante (`U+0301`).

Verificado en JDK 25:

```java
String nfc = "é";     // precompuesta
String nfd = "é";     // e + acento combinante

System.out.println(nfc.length());          // 1
System.out.println(nfd.length());          // 2
System.out.println(nfc.equals(nfd));       // false   (!)
System.out.println(nfd.contains("e"));     // true    (!)
System.out.println(nfc.contains("e"));     // false
```

Los dos se imprimen como `é`. Son visualmente indistinguibles en cualquier editor, terminal o navegador. Y `equals` dice que son distintos, porque son secuencias de bytes distintas.

### Dónde te muerde

- **macOS usa NFD** en los nombres de archivo; Linux y Windows usan NFC. Un archivo llamado `José.pdf` creado en Mac no se encuentra buscando `"José.pdf"` escrito en Linux.
- Un usuario pega su nombre desde una fuente que usa NFD, se guarda así en la base de datos, y después el login por nombre falla.
- Dos registros que parecen duplicados exactos no se detectan como duplicados.
- Un buscador no encuentra un término que está visiblemente en el texto.

### La solución

**Normalizá en la frontera del sistema**: al recibir el dato, antes de guardarlo o compararlo.

```java
import java.text.Normalizer;

String normalizado = Normalizer.normalize(entrada, Normalizer.Form.NFC);
```

Verificado: tras normalizar ambos a NFC, `equals` devuelve `true`.

`NFC` es la forma recomendada para almacenamiento e intercambio (es lo que recomienda el W3C para la web). Las cuatro formas son `NFC`, `NFD`, `NFKC`, `NFKD`; las dos con `K` además unifican equivalencias de compatibilidad (convierten la ligadura `ﬃ` en `ffi`, por ejemplo), lo cual es útil para búsquedas pero destructivo para almacenar.

### Bonus: quitar acentos

La descomposición tiene un uso práctico muy común — generar slugs o búsquedas insensibles a acentos:

```java
String sinAcentos = Normalizer.normalize(texto, Normalizer.Form.NFD)
                              .replaceAll("\\p{M}", "");   // borra las marcas combinantes
// "Ávila" -> "Avila"
```

El truco: NFD separa la letra de su acento, y `\p{M}` (categoría Unicode *Mark*) elimina los acentos sueltos.

## 36. Bytes y encodings

Un `String` en memoria es una secuencia de caracteres. Un archivo, un socket o una respuesta HTTP son secuencias de **bytes**. Convertir entre ambos requiere una **codificación**.

```java
String texto = "Café";

byte[] b1 = texto.getBytes();                          // PELIGROSO
byte[] b2 = texto.getBytes(StandardCharsets.UTF_8);    // correcto

String vuelta = new String(b2, StandardCharsets.UTF_8);
```

### El problema de la versión sin argumentos

`getBytes()` y `new String(byte[])` usan el **charset por defecto de la plataforma**. Ese valor dependía históricamente del sistema operativo y de la configuración regional: UTF-8 en Linux, `windows-1252` en muchos Windows.

Resultado clásico: un programa escribe un archivo en una máquina y otro lo lee en otra, y todos los acentos aparecen como `Ã©` o `?`. El famoso *mojibake*.

> **Nota de versión importante.** [JEP 400](https://openjdk.org/jeps/400), incluido en **Java 18**, cambió el charset por defecto a **UTF-8 en todas las plataformas**. En Java 18+ el riesgo bajó muchísimo. Pero seguí siendo explícito: hace el código autoexplicativo, y protege si alguien arranca la JVM con `-Dfile.encoding` distinto o si el código corre en Java 17 o anterior.

### Caracteres no representables

Si el charset destino no puede representar un carácter, no hay excepción: hay pérdida silenciosa.

```java
"Café".getBytes(StandardCharsets.US_ASCII);
// la 'é' se convierte en '?' — y no hay forma de recuperarla
```

Para detectarlo hay que usar la API de `CharsetEncoder` con `onUnmappableCharacter(CodingErrorAction.REPORT)`.

### La regla

> Especificá siempre el charset. Usá `StandardCharsets.UTF_8` salvo que un formato externo exija otro. Nunca uses los strings mágicos `"UTF-8"` cuando exista la constante: `StandardCharsets` evita errores de tipeo y no lanza `UnsupportedEncodingException`.

---

# Parte VIII — Conversiones

## 37. De `String` a número y de vuelta

### De `String` a número

```java
int i     = Integer.parseInt("42");
long l    = Long.parseLong("42");
double d  = Double.parseDouble("3.14");
boolean b = Boolean.parseBoolean("true");
```

Los métodos `parseXxx` devuelven **primitivos**. Los `valueOf` devuelven los **wrappers** (`Integer`, `Long`...), lo que sirve si vas a meterlos en una colección:

```java
Integer boxed = Integer.valueOf("42");
```

### La excepción

Si el texto no es un número válido, `NumberFormatException`. Y es más estricto de lo que la gente espera. Verificado en JDK 25:

```java
Integer.parseInt("42 ");
// NumberFormatException: For input string: "42 "
```

**Un espacio al final alcanza para que falle.** No hace `trim` implícito. Un valor que viene de un formulario, un CSV o una variable de entorno casi siempre necesita limpieza previa:

```java
int valor = Integer.parseInt(entrada.strip());
```

Otros casos que fallan: `"42.0"` con `parseInt` (el punto no es válido para enteros), `"1e3"` con `parseInt`, `""`, `null` y `"0x1F"`.

`NumberFormatException` es **unchecked**, así que el compilador no te obliga a capturarla. Es una de las causas más frecuentes de errores 500 en endpoints que reciben parámetros de texto:

```java
// Patrón defensivo para entrada externa
public static Optional<Integer> parsearEntero(String valor) {
    if (valor == null || valor.isBlank()) {
        return Optional.empty();
    }
    try {
        return Optional.of(Integer.parseInt(valor.strip()));
    } catch (NumberFormatException ex) {
        return Optional.empty();
    }
}
```

### De número a `String`

Tres formas equivalentes en resultado:

```java
String a = String.valueOf(10);     // recomendada
String b = Integer.toString(10);
String c = "" + 10;                // funciona, pero es un truco
```

`String.valueOf` es la preferible: es explícita, tiene sobrecarga para todos los tipos y **maneja `null`** sin lanzar excepción (a diferencia de `Object::toString`).

### La trampa de la precedencia

Verificado en JDK 25:

```java
System.out.println("" + 0.1 + 0.2);    // 0.10.2
System.out.println(0.1 + 0.2 + "");    // 0.30000000000000004
System.out.println('a' + 1);           // 98
System.out.println("" + 'a' + 1);      // a1
```

El operador `+` se evalúa de izquierda a derecha. En la primera línea, `"" + 0.1` ya produce un `String`, así que el `+ 0.2` es concatenación. En la segunda, la suma numérica ocurre primero.

La tercera es la trampa clásica: `'a' + 1` promociona el `char` a `int` y suma, dando 98. Es el mismo mecanismo de promoción numérica de [Type Casting](05-type-casting.md).

La solución cuando importa: paréntesis explícitos.

```java
System.out.println("Total: " + (0.1 + 0.2));
```

> Nota al margen: el `0.30000000000000004` no es un bug de strings, es aritmética de punto flotante binario. Para dinero, `BigDecimal`.

## 38. `valueOf`, `toString` y el `null`

### La concatenación con `null` no falla

Verificado en JDK 25:

```java
String nulo = null;
System.out.println("valor=" + nulo);       // valor=null
```

La especificación del lenguaje dice que si un operando de concatenación es una referencia `null`, se convierte en la cadena `"null"`. **No lanza `NullPointerException`.**

Esto es cómodo y peligroso a la vez: `"Hola " + usuario.getNombre()` produce `"Hola null"` en vez de fallar, y ese texto termina en la interfaz del usuario o en la base de datos.

### `String.valueOf` y su ambigüedad

```java
System.out.println(String.valueOf((Object) null));   // "null"
System.out.println(String.valueOf((char[]) null));   // NullPointerException
```

*(Verificado en JDK 25.)*

Dos sobrecargas, dos comportamientos opuestos. Y hay algo peor: esto **no compila**:

```java
String.valueOf(null);   // error: reference to valueOf is ambiguous
```

El compilador no sabe si elegir `valueOf(Object)` o `valueOf(char[])`. Es un caso famoso de la API, y el motivo por el que conviene ser explícito con el cast cuando el valor puede ser `null`.

### `Objects.toString`

La forma limpia de tener un valor por defecto:

```java
Objects.toString(valor);            // "null" si es null
Objects.toString(valor, "N/D");     // "N/D" si es null
Objects.requireNonNullElse(valor, "N/D");
```

### `toString()` en tus propias clases

Todo objeto hereda `toString()` de `Object`, que devuelve algo como `com.ejemplo.Usuario@1b6d3586`: el nombre de la clase y el hash en hexadecimal. Inútil para depurar.

```java
public class Usuario {
    private final String nombre;
    private final String email;

    @Override
    public String toString() {
        return "Usuario{nombre='" + nombre + "', email='" + email + "'}";
    }
}
```

Con `record`, Java lo genera solo:

```java
public record Usuario(String nombre, String email) {}
// toString() -> Usuario[nombre=Ana, email=ana@ejemplo.com]
```

> **Advertencia de seguridad:** nunca incluyas contraseñas, tokens ni datos personales sensibles en `toString()`. Los frameworks de logging llaman a `toString()` constantemente, y así es como las credenciales terminan en texto plano en un archivo de log. En un `record` que contenga un secreto, sobrescribí `toString()` a mano.

---

# Parte IX — Cierre

## 39. Strings y seguridad: por qué las contraseñas van en `char[]`

Una pregunta habitual en entrevistas, con una respuesta que se entiende sola después de la [sección 4](#4-inmutabilidad-la-decisión-que-explica-todo-lo-demás).

Fijate en esta firma de la biblioteca estándar:

```java
// javax.swing.JPasswordField
public char[] getPassword()      // devuelve char[], no String

// java.io.Console
public char[] readPassword()     // idem
```

No es un capricho histórico: es deliberado.

### El razonamiento

**1. No podés borrar un `String`.** Es inmutable: no hay forma de sobrescribir su contenido. La contraseña queda en el heap hasta que el recolector de basura pase por ahí — y no controlás cuándo. Un volcado de memoria (*heap dump*) tomado en ese intervalo la contiene en texto plano. Con un `char[]` sí podés:

```java
char[] password = console.readPassword();
try {
    autenticar(password);
} finally {
    Arrays.fill(password, '\0');    // borrado explícito e inmediato
}
```

**2. El pool empeora las cosas.** Si la contraseña llega como literal o se interna, vive en el String pool, que está diseñado para retener valores.

**3. Logging accidental.** Un `String` se imprime tal cual en cualquier log, mensaje de excepción o `toString()`. Un `char[]` imprime `[C@1b6d3586`, que no revela nada.

Esto no es folclore: la [Java Cryptography Architecture](https://docs.oracle.com/en/java/javase/17/security/java-cryptography-architecture-jca-reference-guide.html) lo recomienda explícitamente.

### El matiz honesto

En la práctica, muchos frameworks web (incluido Spring Security) reciben la contraseña como `String` porque llega dentro de un cuerpo HTTP que ya fue parseado como texto. En ese contexto, insistir en `char[]` en tu capa no borra las copias que el framework ya creó.

La conclusión útil: **usá `char[]` donde controlés el ciclo de vida completo, y nunca guardes ni loguees contraseñas en claro**. El almacenamiento va con hash y salt (bcrypt, scrypt, Argon2), nunca en texto plano.

## 40. Casos de uso reales

Patrones que vas a escribir de verdad, con lo aprendido aplicado.

### Validar y normalizar entrada de usuario

```java
public static String normalizarEmail(String entrada) {
    if (entrada == null || entrada.isBlank()) {
        throw new IllegalArgumentException("El email es obligatorio");
    }
    return Normalizer.normalize(entrada, Normalizer.Form.NFC)
                     .strip()
                     .toLowerCase(Locale.ROOT);
}
```

Cuatro decisiones deliberadas: `isBlank` en vez de `isEmpty`, normalización Unicode antes de comparar, `strip` en vez de `trim`, y `Locale.ROOT` porque es lógica interna.

### Generar un slug para URLs

```java
private static final Pattern NO_ALFANUMERICO = Pattern.compile("[^a-z0-9]+");
private static final Pattern MARCAS = Pattern.compile("\\p{M}");

public static String slug(String titulo) {
    String sinAcentos = MARCAS.matcher(
            Normalizer.normalize(titulo, Normalizer.Form.NFD)).replaceAll("");
    return NO_ALFANUMERICO.matcher(sinAcentos.toLowerCase(Locale.ROOT))
                          .replaceAll("-")
                          .replaceAll("^-|-$", "");
}
// "Cómo programar en Java" -> "como-programar-en-java"
```

Los patrones están precompilados como constantes porque este método se llama a menudo.

### Parsear un archivo delimitado

```java
public static List<Registro> parsear(Path archivo) throws IOException {
    try (Stream<String> lineas = Files.lines(archivo, StandardCharsets.UTF_8)) {
        return lineas.skip(1)                        // cabecera
                     .filter(l -> !l.isBlank())
                     .map(l -> l.split(";", -1))     // -1: conservar campos vacíos
                     .filter(c -> c.length == 4)
                     .map(c -> new Registro(c[0].strip(), c[1].strip(), c[2], c[3]))
                     .toList();
    }
}
```

Las decisiones clave: charset explícito, `split` con `-1`, validación de longitud antes de indexar, y `try-with-resources` porque `Files.lines` mantiene el archivo abierto.

### Construir un informe

```java
public String informe(List<Pedido> pedidos) {
    StringBuilder sb = new StringBuilder(pedidos.size() * 64);
    sb.append("-".repeat(50)).append('\n');
    sb.append(String.format(Locale.ROOT, "%-20s %10s %15s%n", "CLIENTE", "ITEMS", "TOTAL"));

    for (Pedido p : pedidos) {
        sb.append(String.format(Locale.ROOT, "%-20s %10d %15.2f%n",
                                p.cliente(), p.items(), p.total()));
    }
    return sb.toString();
}
```

`StringBuilder` porque hay bucle, capacidad inicial estimada, y `Locale.ROOT` para que el separador decimal sea estable.

### Enmascarar datos sensibles para logs

```java
public static String enmascarar(String tarjeta) {
    if (tarjeta == null || tarjeta.length() < 4) {
        return "****";
    }
    String ultimos = tarjeta.substring(tarjeta.length() - 4);
    return "*".repeat(tarjeta.length() - 4) + ultimos;
}
// "4111111111111111" -> "************1111"
```

### Comparar de forma segura en autenticación

```java
// MAL: el tiempo de ejecución revela cuántos caracteres coinciden
if (tokenRecibido.equals(tokenEsperado)) { ... }

// BIEN: comparación en tiempo constante
if (MessageDigest.isEqual(tokenRecibido.getBytes(StandardCharsets.UTF_8),
                          tokenEsperado.getBytes(StandardCharsets.UTF_8))) { ... }
```

`equals` de `String` corta en cuanto encuentra una diferencia. Midiendo tiempos, un atacante puede deducir el prefijo correcto carácter a carácter (*timing attack*). Para comparar secretos, `MessageDigest.isEqual`.

### `switch` sobre strings

Válido desde Java 7, y con flechas desde Java 14:

```java
String resultado = switch (comando) {
    case "start" -> "arrancando";
    case "stop"  -> "parando";
    default      -> "comando desconocido";
};
```

*(Verificado en JDK 25.)*

Por dentro compila a un `switch` sobre `hashCode()` más una verificación con `equals` — porque, como vimos, los hashes pueden colisionar:

```java
System.out.println("Aa".hashCode());   // 2112
System.out.println("BB".hashCode());   // 2112   (!)
System.out.println("Aa".equals("BB")); // false
```

*(Verificado en JDK 25.)* Es el ejemplo canónico de colisión de hash en `String`, y la razón por la que `hashCode` nunca sustituye a `equals`.

**Cuidado:** `switch` sobre un `String` `null` lanza `NullPointerException`. Verificá antes.

## 41. Anti-patrones

Los errores que se ven en revisiones de código, ordenados por frecuencia.

### 1. Comparar con `==`

```java
if (rol == "admin") { ... }              // MAL
if ("admin".equals(rol)) { ... }         // BIEN
```

### 2. Concatenar dentro de un bucle

```java
for (String s : lista) { r = r + s; }              // MAL: O(n²), medido 91x más lento
StringBuilder sb = new StringBuilder();            // BIEN
for (String s : lista) { sb.append(s); }
```

### 3. `new String("literal")`

```java
String s = new String("hola");   // MAL: objeto extra sin ningún beneficio
String s = "hola";               // BIEN
```

### 4. Ignorar el valor devuelto

```java
texto.trim();                    // MAL: no hace nada
texto = texto.trim();            // BIEN
```

### 5. `toUpperCase()`/`toLowerCase()` sin `Locale`

```java
if (h.toUpperCase().equals("CONTENT-TYPE"))          // MAL: se rompe en Turquía
if (h.equalsIgnoreCase("Content-Type"))              // BIEN
```

### 6. `replaceAll` cuando querías texto literal

```java
ip.replaceAll(".", "-");         // MAL: destruye el string entero
ip.replace(".", "-");            // BIEN
```

### 7. `split` sin `-1` al parsear datos posicionales

```java
String[] c = linea.split(",");        // MAL: pierde columnas vacías finales
String[] c = linea.split(",", -1);    // BIEN
```

### 8. Concatenar en logging

```java
log.debug("Pedido " + id + " procesado");    // MAL: se construye siempre
log.debug("Pedido {} procesado", id);        // BIEN
```

### 9. Construir SQL concatenando

```java
// MAL: inyección SQL
String sql = "SELECT * FROM usuarios WHERE nombre = '" + nombre + "'";

// BIEN: consulta parametrizada
PreparedStatement ps = con.prepareStatement("SELECT * FROM usuarios WHERE nombre = ?");
ps.setString(1, nombre);
```

Si `nombre` vale `' OR '1'='1`, la primera versión devuelve la tabla entera. Es la vulnerabilidad más explotada de la historia del software, y nace de tratar código como texto. **Ningún text block ni `formatted()` arregla esto**: el problema no es la legibilidad, es la falta de separación entre instrucción y dato.

### 10. `length() == 0` en vez de `isEmpty()`, y `isEmpty()` donde va `isBlank()`

```java
if (nombre.length() == 0)   // funciona, pero menos expresivo
if (nombre.isEmpty())       // mejor
if (nombre.isBlank())       // correcto para validar entrada de usuario
```

### 11. Asumir que `length()` cuenta caracteres

```java
if (mensaje.length() > 280)                                  // MAL con emojis
if (mensaje.codePointCount(0, mensaje.length()) > 280)       // mejor
```

### 12. `StringBuilder` como clave de mapa o en un `Set`

```java
Set<StringBuilder> set = new HashSet<>();   // MAL: no sobrescribe equals/hashCode
```

## 42. Checklist y tabla de decisión

### Antes de dar por terminado código que manipula texto

- [ ] ¿Comparé con `equals()` y no con `==`?
- [ ] ¿Puede el valor ser `null`? ¿La constante va primero, o uso `Objects.equals`?
- [ ] ¿Valido con `isBlank()` si es entrada de usuario?
- [ ] ¿Hay concatenación dentro de un bucle?
- [ ] ¿`toUpperCase`/`toLowerCase`/`format` llevan `Locale` explícito?
- [ ] ¿Uso `replace` donde no necesito regex?
- [ ] ¿`split` lleva `-1` si las posiciones importan?
- [ ] ¿Especifiqué el charset en toda conversión a bytes?
- [ ] ¿El texto puede traer emojis o acentos combinantes? ¿Normalizo?
- [ ] ¿Hay datos sensibles que puedan acabar en un log o un `toString()`?
- [ ] ¿Construyo SQL/HTML/JSON concatenando? (parametrizá o usá una librería)
- [ ] ¿Asigné el resultado de cada transformación?

### Qué usar para construir texto

| Necesito | Herramienta |
|---|---|
| Una expresión simple | `+` |
| Concatenar en bucle | `StringBuilder` |
| Unir una colección | `String.join` / `Collectors.joining` |
| Unir con prefijo y sufijo | `StringJoiner` |
| Plantilla con formato | `formatted()` / `String.format` |
| Texto multilínea | Text block |
| Mensaje de log | Placeholders `{}` |
| SQL | `PreparedStatement`, nunca concatenación |

### Qué usar para comparar

| Necesito | Herramienta |
|---|---|
| ¿Mismo contenido? | `equals()` |
| ¿Mismo contenido, ignorando mayúsculas? | `equalsIgnoreCase()` |
| ¿Cuál va primero? (interno) | `compareTo()` |
| ¿Cuál va primero? (para el usuario) | `Collator` |
| Ignorar acentos y mayúsculas | `Collator` con `PRIMARY` |
| ¿Contiene este texto? | `contains()` |
| ¿Cumple este patrón completo? | `matches()` |
| ¿Contiene este patrón? | `Pattern` + `find()` |
| Comparar un tramo sin copiar | `regionMatches()` |
| Comparar secretos | `MessageDigest.isEqual` |

### Qué usar para contar

| Necesito | Herramienta |
|---|---|
| Tamaño de buffer, índices internos | `length()` |
| Caracteres reales (texto internacional) | `codePointCount()` |
| Lo que el usuario ve como un carácter | `BreakIterator` |

## 43. Autoevaluación

Respondé sin ejecutar el código. Las respuestas están al final de cada bloque.

**1.** ¿Qué imprime?

```java
String a = "java";
String b = "ja" + "va";
String parte = "ja";
String c = parte + "va";
System.out.println((a == b) + " " + (a == c) + " " + a.equals(c));
```

<details><summary>Respuesta</summary>

`true false true`. `b` se resuelve por constant folding y queda como literal en el pool. `c` se construye en runtime porque `parte` no es una constante de compilación, así que es otro objeto. El contenido siempre es el mismo.
</details>

**2.** ¿Cuánto vale `partes.length`?

```java
String[] partes = "a,b,,".split(",");
```

<details><summary>Respuesta</summary>

`2`. Los campos vacíos finales se descartan con el límite por defecto. Con `split(",", -1)` daría `4`.
</details>

**3.** ¿Qué imprime?

```java
System.out.println("1.2.3".replaceAll(".", "-"));
```

<details><summary>Respuesta</summary>

`-----`. `replaceAll` interpreta el argumento como regex, y `.` significa "cualquier carácter". Con `replace(".", "-")` daría `1-2-3`.
</details>

**4.** ¿Qué imprime `texto`?

```java
String texto = "hola";
texto.toUpperCase();
System.out.println(texto);
```

<details><summary>Respuesta</summary>

`hola`. Los strings son inmutables; `toUpperCase()` devuelve un objeto nuevo que se descarta.
</details>

**5.** ¿Cuánto vale `length()` y cuántos "caracteres" ve un usuario?

```java
String s = "Hi 😀";
System.out.println(s.length());
```

<details><summary>Respuesta</summary>

`length()` devuelve `5`; el usuario ve 4. El emoji está fuera del plano básico y ocupa dos `char` (par suplente). `codePointCount(0, s.length())` devuelve `4`.
</details>

**6.** ¿Por qué esto puede fallar en un servidor turco?

```java
if (header.toUpperCase().equals("CONTENT-TYPE")) { ... }
```

<details><summary>Respuesta</summary>

Sin `Locale`, usa el locale por defecto. En turco, `i` en mayúscula es `İ` (con punto), así que `"content-type".toUpperCase()` da `CONTENT-TYPE` con `İ` y la comparación falla. Solución: `equalsIgnoreCase("Content-Type")` o `toUpperCase(Locale.ROOT)`.
</details>

**7.** ¿Qué devuelve?

```java
System.out.println("one two three".matches("two"));
```

<details><summary>Respuesta</summary>

`false`. `matches()` exige que el patrón describa la cadena **completa**. Para buscar dentro: `contains("two")` o `Pattern.find()`.
</details>

**8.** ¿Cuál es el problema y cómo se arregla?

```java
String r = "";
for (String s : milElementos) { r += s; }
```

<details><summary>Respuesta</summary>

Cada iteración crea un `String` nuevo copiando todo lo acumulado: O(n²). Medido, 91 veces más lento que la alternativa con 40.000 elementos. Se arregla con `StringBuilder` o, si solo hay que unir, con `String.join`.
</details>

**9.** ¿Por qué `equals` devuelve `false` si ambos se imprimen como `José`?

<details><summary>Respuesta</summary>

Uno está en NFC (la `é` es un solo code point) y el otro en NFD (`e` + acento combinante). Son secuencias distintas. Se resuelve normalizando ambos con `Normalizer.normalize(s, Form.NFC)` antes de comparar.
</details>

**10.** ¿Qué tiene de malo?

```java
log.debug("Usuario " + usuario.getNombre() + " autenticado");
```

<details><summary>Respuesta</summary>

La concatenación se ejecuta aunque `DEBUG` esté desactivado, porque Java evalúa los argumentos antes de llamar al método. Se arregla con placeholders: `log.debug("Usuario {} autenticado", usuario.getNombre())`.
</details>

**11.** ¿Por qué `getPassword()` devuelve `char[]` y no `String`?

<details><summary>Respuesta</summary>

Porque un `String` es inmutable y no se puede borrar de memoria: queda en el heap hasta que pase el GC, visible en un volcado de memoria. Un `char[]` se puede sobrescribir con `Arrays.fill(pw, '\0')` inmediatamente después de usarlo. Además no va al pool y no se imprime en claro en los logs.
</details>

**12.** ¿En qué compila hoy `"a" + b + "c"` y por qué importa?

<details><summary>Respuesta</summary>

En una única instrucción `invokedynamic` a `makeConcatWithConstants` (JEP 280, Java 9+), **no** en `StringBuilder` como afirman muchos tutoriales. Importa porque significa que para una sola expresión el `+` es óptimo y escribir `StringBuilder` a mano no aporta. Dentro de un bucle sigue siendo un problema, porque la llamada se ejecuta una vez por iteración.
</details>

## 44. Fuentes

**Fuentes de referencia principales**

- [Jenkov — Java Strings](https://jenkov.com/tutorials/java/strings.html) — buena cobertura de la API. **Con reserva:** su sección de rendimiento de concatenación describe el comportamiento previo a Java 9 (`StringBuilder`) como si fuera el actual; ver [sección 29](#29-concatenación-con--qué-genera-realmente-el-compilador).
- [W3Schools — Java Strings](https://www.w3schools.com/java/java_strings.asp) y sus subpáginas de [concatenación](https://www.w3schools.com/java/java_strings_concat.asp), [números y strings](https://www.w3schools.com/java/java_strings_numbers.asp), [caracteres especiales](https://www.w3schools.com/java/java_strings_specchars.asp) y la [referencia completa de métodos](https://www.w3schools.com/java/java_ref_string.asp) — útil como índice de métodos; no cubre locale, Unicode ni rendimiento.
- [Baeldung — All About String in Java](https://www.baeldung.com/java-string) y los artículos de la serie: [String Pool](https://www.baeldung.com/java-string-pool), [Why String Is Immutable](https://www.baeldung.com/java-string-immutable), [Compact Strings](https://www.baeldung.com/java-9-compact-string), [Text Blocks](https://www.baeldung.com/java-text-blocks), [StringBuilder vs StringBuffer](https://www.baeldung.com/java-string-builder-string-buffer), [Comparing Strings](https://www.baeldung.com/java-compare-strings), [Split a String](https://www.baeldung.com/java-split-string), [Guide to java.util.Formatter](https://www.baeldung.com/java-string-formatter), [String Concatenation with Invoke Dynamic](https://www.baeldung.com/java-string-concatenation-invoke-dynamic), [Storing Passwords](https://www.baeldung.com/java-storing-passwords).

**Especificación y JEPs**

- [JLS §4.12.4 — Constantes de compilación](https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html#jls-4.12.4)
- [JEP 254 — Compact Strings](https://openjdk.org/jeps/254)
- [JEP 280 — Indify String Concatenation](https://openjdk.org/jeps/280)
- [JEP 378 — Text Blocks](https://openjdk.org/jeps/378)
- [JEP 400 — UTF-8 by Default](https://openjdk.org/jeps/400)
- [JEP 459 — String Templates (retirado en Java 23)](https://openjdk.org/jeps/459)
- [JDK-8305907 — Propuesta de deprecar `toLowerCase()`/`toUpperCase()` sin locale](https://bugs.openjdk.org/browse/JDK-8305907)
- [Java Cryptography Architecture — manejo de contraseñas](https://docs.oracle.com/en/java/javase/17/security/java-cryptography-architecture-jca-reference-guide.html)

**Artículos y discusiones consultados para contrastar**

- [Claes Redestad — String concatenation, redux](https://cl4es.github.io/2019/05/14/String-Concat-Redux.html) — análisis de rendimiento de JEP 280 por un ingeniero de OpenJDK.
- [Matt Ryall — The infamous Turkish locale bug](https://mattryall.net/blog/the-infamous-turkish-locale-bug) y [Gary Gregory — The Java Lowercase Conversion Surprise in Turkey](https://garygregory.wordpress.com/2015/11/03/java-lowercase-conversion-turkey/)
- [Chris Newland — String.split include empty trailing strings](https://www.chrisnewland.com/java-stringsplit-include-empty-trailing-strings--156)
- [javarevisited — How substring works in Java, memory leak fixed in JDK 1.7](https://javarevisited.blogspot.com/2011/10/how-substring-in-java-works.html)
- [Kaustubh Saha — A Java Developer's Guide to Surviving Unicode Strings](https://medium.com/@kaustubh.saha/a-java-developers-guide-to-surviving-unicode-strings-6a00cf94309c)
- [jSparrow — Avoid Concatenation in Logging Statements](https://jsparrow.github.io/rules/avoid-concatenation-in-logging-statements.html)

**Verificación**

Todos los ejemplos, salidas, mensajes de excepción, mediciones de tiempo y volcados de bytecode fueron ejecutados en **OpenJDK Temurin 25.0.3+9 (LTS)** sobre macOS, usando `java`, `javac` y `javap -c`. Las mediciones de tiempo son indicativas (ejecución única, sin JMH) y sirven para mostrar órdenes de magnitud, no cifras absolutas.

---

## Alcance y temas relacionados

Este documento cubre `String` y su ecosistema inmediato. Quedan fuera, y se tratan en otros bloques:

- **Expresiones regulares en profundidad** (`Pattern`, `Matcher`, grupos con nombre, lookahead) — merecen su propio tema.
- **`Optional`** para modelar la ausencia de texto — bloque `06-Streams-y-Funcional`.
- **`equals`/`hashCode`** como contrato general — bloque `04-Colecciones`.
- **`BigDecimal`** y formato de dinero — pendiente.
- **I/O y `Charset`** en detalle — bloque `09-IO`.
- **Internacionalización** completa (`ResourceBundle`, `MessageFormat`) — pendiente.


