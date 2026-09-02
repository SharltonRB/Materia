# Basic Syntax

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+

**Alcance de este documento:** las reglas del lenguaje que rigen *cómo se escribe* Java — estructura del archivo, identificadores, palabras reservadas, literales, sentencias, bloques, salida por consola, comentarios, paquetes y modificadores.

**Lo que NO entra aquí**, porque tiene documento propio: variables y tipos de dato (`02`), operadores (`03`), control de flujo (`04`), arrays (`05`), métodos (`06`) y Strings (`07`).

Las citas textuales marcadas provienen de la documentación oficial de Oracle en [dev.java](https://dev.java/learn/language-basics/) y del tutorial de [W3Schools](https://www.w3schools.com/java/). Ver *Fuentes* al final.

---

## Fundamentos

### Por qué la sintaxis de Java es como es

Java nació con una restricción de diseño que lo explica casi todo: *el mismo programa debe correr en cualquier máquina*. Eso obligó a meter un paso intermedio entre tu código y el procesador.

```
Hola.java  ──javac──►  Hola.class  ──JVM──►  ejecución
 (fuente)              (bytecode)           (máquina real)
```

El bytecode no es código máquina: es un formato intermedio que la **JVM** (Java Virtual Machine) traduce a instrucciones nativas al vuelo. De ahí el eslogan *write once, run anywhere*.

La consecuencia práctica para la sintaxis es que Java es de **tipado estático**: el compilador quiere saberlo todo antes de ejecutar nada. Por eso declaras tipos, por eso hay tanta palabra clave, y por eso muchos errores te saltan al compilar en vez de a las 3 de la mañana en producción. La verbosidad no es un descuido — es el precio de esa verificación previa.

### El programa mínimo

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

La regla de partida, tal como la enuncia W3Schools:

> *"Every line of code that runs in Java must be inside a `class`."*

De ahí salen tres consecuencias inmediatas:

1. **Todo vive dentro de una clase.** No existen funciones sueltas a nivel de archivo (con la excepción de Java 25, más abajo).
2. **El nombre de la clase empieza por mayúscula.** Es convención, no obligación del compilador — pero universal.
3. **El archivo se llama igual que la clase.** Si la clase es `Main`, el archivo es `Main.java`. Si no coinciden, no compila.

### Anatomía de `main`

Esa firma no es arbitraria: es un **contrato** con la JVM, que busca exactamente ese método para arrancar. W3Schools lo resume así: *"el método `main()` es obligatorio en todo programa Java. Es donde el programa empieza a ejecutarse."*

Desglose palabra por palabra:

| Palabra | Qué significa | Por qué está ahí |
|---|---|---|
| `public` | accesible desde cualquier parte | la JVM lo invoca desde fuera de tu código; si fuera `private` no podría verlo |
| `static` | pertenece a la clase, no a un objeto | al arrancar todavía no existe ninguna instancia — no hay objeto sobre el que llamarlo |
| `void` | no devuelve nada | el código de salida del proceso se comunica con `System.exit(int)`, no con un `return` |
| `main` | nombre exacto | es el nombre que la JVM tiene codificado como punto de entrada |
| `String[] args` | argumentos de línea de comandos | lo que escribes tras el nombre del programa al ejecutarlo |

`String[] args` admite variantes válidas: `String args[]` (sintaxis heredada de C, desaconsejada) y `String... args` (varargs). El nombre `args` es convención: `main(String[] parametros)` funciona igual.

### Imprimir por consola

```java
System.out.println("Hello World");
```

W3Schools desglosa la expresión pieza a pieza, y merece la pena porque es la primera línea que escribe todo el mundo sin entenderla:

| Pieza | Qué es |
|---|---|
| `System` | una clase integrada de Java (vive en `java.lang`) |
| `out` | un miembro de `System` que representa la **salida** estándar |
| `println` | el método: *print line*, imprimir línea |

Existen dos métodos, y la diferencia es exactamente una:

```java
System.out.println("Hola");   // imprime y salta de línea
System.out.print("Hola");     // imprime y NO salta de línea
```

```java
System.out.print("Hello World! ");
System.out.print("I will print on the same line.");
// → Hello World! I will print on the same line.
```

Como `print()` no separa nada, si lo usas tienes que añadir los espacios a mano. Por eso casi todo el material didáctico usa `println()`: la salida se lee mejor.

**El texto va siempre entre comillas dobles.** Sin ellas es un error de compilación, porque Java intenta interpretarlo como código:

```java
System.out.println("This sentence will work!");
System.out.println(This sentence will produce an error);   // ✗ no compila
```

### Compilar y ejecutar

```bash
javac Main.java      # produce Main.class
java Main            # ojo: sin la extensión .class
```

Desde **Java 11** puedes saltarte la compilación explícita para un archivo suelto ([JEP 330](https://openjdk.org/jeps/330)):

```bash
java Main.java       # compila en memoria y ejecuta
```

Y desde **Java 9** existe `jshell`, un REPL para probar sintaxis sin escribir una clase entera:

```
jshell> 2 + 2
$1 ==> 4
jshell> "hola".toUpperCase()
$2 ==> "HOLA"
```

Para aprender sintaxis, `jshell` ahorra muchísimo tiempo: pruebas una expresión y ves el resultado sin ceremonia.

### La estructura de un archivo `.java`

El orden **no es libre**. Un archivo tiene como mucho tres partes, en esta secuencia:

```java
package com.empresa.facturacion;   // 1. Paquete (opcional, máximo uno, primera línea)

import java.util.List;             // 2. Imports (los que hagan falta)
import java.util.Map;

public class Factura {             // 3. Declaraciones de tipo
    // ...
}
```

Reglas asociadas:

- El nombre del archivo **debe** coincidir con la clase `public` que contiene.
- Un archivo puede declarar varias clases, pero **como mucho una `public`**.
- El paquete debe coincidir con la ruta de carpetas: `com/empresa/facturacion/Factura.java`.
- Si omites `package`, la clase cae en el *default package* — aceptable para un ejercicio, inaceptable en un proyecto real (no se puede importar desde otros paquetes).

### Identificadores: lo legal frente a lo convencional

Un identificador es cualquier nombre que tú pones: clases, métodos, variables, paquetes. Oracle define la regla así:

> *"Variable names are case-sensitive. A variable's name can be any legal identifier — an unlimited-length sequence of Unicode letters and digits, beginning with a letter, the dollar sign `$`, or the underscore character `_`."*

En claro, **lo que el compilador acepta**:

- Empieza por letra, `$` o `_`. **Nunca** por dígito.
- Después: letras, dígitos, `$` y `_`.
- Longitud ilimitada, Unicode admitido (`año`, `precio_€` son legales).
- Sensible a mayúsculas: `total` y `Total` son variables distintas.
- No puede ser una palabra reservada.
- Sin espacios ni guiones (`mi-variable` es una resta, no un nombre).
- Desde Java 9, `_` a secas es palabra reservada y ya no vale como nombre.

```java
int $precio;        // legal (pero $ se reserva por convención para código generado)
int _contador;      // legal, desaconsejado
int año;            // legal
int 2ndIntento;     // ✗ no compila: empieza por dígito
int class;          // ✗ no compila: palabra reservada
```

Y **lo que la convención dicta**, que es distinto. Oracle es explícito en tres puntos:

- *"Empieza siempre por letra, nunca por `$` o `_`"* — aunque sean legales.
- *"Use full words instead of cryptic abbreviations"* — `gearRatio`, no `gr`.
- Una sola palabra, todo en minúscula: `speed`. Varias palabras, camelCase: `currentGear`, `gearRatio`. Constantes, mayúsculas con guion bajo: `NUM_GEARS`.

Resumen para el resto del lenguaje:

| Elemento | Convención | Ejemplo |
|---|---|---|
| Clase, interfaz, record, enum | `PascalCase` | `CuentaBancaria` |
| Método, variable | `camelCase` | `calcularSaldo`, `saldoActual` |
| Constante (`static final`) | `UPPER_SNAKE_CASE` | `TASA_INTERES_MAXIMA` |
| Paquete | todo minúscula, sin guiones | `com.empresa.facturacion` |
| Parámetro de tipo genérico | una letra mayúscula | `T`, `E`, `K`, `V` |

Los nombres se escriben en inglés en la inmensa mayoría de proyectos, incluso en equipos hispanohablantes. Aquí uso español solo por claridad didáctica.

### Palabras reservadas

50 palabras que el lenguaje se queda para sí. Las que más vas a ver, con su función:

| Palabra | Para qué sirve |
|---|---|
| `class`, `interface`, `enum` | declarar tipos |
| `extends`, `implements` | herencia e implementación de interfaces |
| `public`, `protected`, `private` | modificadores de acceso |
| `static` | miembro de la clase, accesible sin instanciar |
| `final` | no reasignable / no heredable / no sobrescribible |
| `abstract` | sin implementación; la clase no se puede instanciar |
| `new` | crear objetos |
| `this`, `super` | el objeto actual / la clase padre |
| `void` | el método no devuelve nada |
| `return` | termina el método y devuelve un valor |
| `if`, `else`, `switch`, `case`, `default` | condicionales |
| `for`, `while`, `do`, `break`, `continue` | bucles y control |
| `try`, `catch`, `finally`, `throw`, `throws` | excepciones |
| `boolean`, `byte`, `char`, `short`, `int`, `long`, `float`, `double` | tipos primitivos |
| `package`, `import` | organización del código |
| `instanceof` | comprobar el tipo de un objeto |
| `synchronized`, `volatile`, `transient`, `native`, `strictfp` | modificadores especializados |
| `assert` | comprobaciones de depuración |
| `const`, `goto` | **reservadas pero sin uso** |

`const` y `goto` existen solo para que el compilador dé un error claro a quien venga de C++, en vez de un críptico "identificador inesperado".

Aparte hay tres **literales reservados** — `true`, `false` y `null` — que técnicamente no son keywords pero tampoco puedes usarlos como nombres.

> **Nota de lectura crítica.** La tabla de keywords de W3Schools lista `var`, `module`, `exports` y `requires` como palabras clave, y omite `const`, `goto` y `strictfp`. Es impreciso. Esas cuatro son **palabras clave contextuales**: reservadas solo en ciertas posiciones. Por eso `int var = 5;` compila sin problema — fuera de una declaración de tipo, `var` es un identificador normal. Se diseñó así para poder añadir palabras al lenguaje sin romper código antiguo. Otras contextuales: `record`, `sealed`, `permits`, `non-sealed`, `yield`.
>
> Que una fuente muy usada tenga este error es en sí una lección: contrasta siempre con la [especificación oficial](https://docs.oracle.com/javase/specs/) cuando algo huele raro.

### Literales

Un literal es un valor escrito directamente en el código.

**Enteros** — cuatro bases:

```java
int decimal = 42;
int hexadecimal = 0x2A;      // prefijo 0x
int octal = 052;             // prefijo 0 ← fuente clásica de bugs
int binario = 0b101010;      // prefijo 0b (Java 7+)

long grande = 9000000000L;   // sufijo L obligatorio si excede int
```

El literal octal es una trampa heredada de C: `int codigo = 0755;` no vale 755. Si escribes un cero delante para alinear columnas, cambias el valor.

**Separador de dígitos** (Java 7+) — puramente visual, el compilador lo ignora:

```java
int millon = 1_000_000;
long tarjeta = 1234_5678_9012_3456L;
int mascara = 0b1010_0101;
```

**Punto flotante:**

```java
double d = 3.14;             // double por defecto
float f = 3.14f;             // sufijo f obligatorio
double cientifica = 1.5e3;   // 1500.0
```

**Caracteres, texto y el resto:**

```java
char letra = 'A';
char unicode = '\u0041';   // también 'A'
String texto = "Hola";
boolean activo = true;
String vacio = null;
```

`'A'` con comillas simples es un `char`; `"A"` con dobles es un `String`. No son intercambiables.

### Secuencias de escape

Dentro de un `char` o un `String`, la barra invertida cambia el significado del siguiente carácter:

| Escape | Significado |
|---|---|
| `\n` | salto de línea |
| `\t` | tabulación |
| `\r` | retorno de carro |
| `\"` | comilla doble |
| `\'` | comilla simple |
| `\\` | barra invertida |
| `\b` | retroceso |
| `\f` | salto de página |
| `\s` | espacio (Java 15+, útil en text blocks) |
| `\uXXXX` | carácter Unicode por código hexadecimal |

```java
System.out.println("Dijo: \"hola\"\ny se fue");
String ruta = "C:\\Users\\ana";   // en Windows, cada \ se duplica
```

### Expresiones, sentencias y bloques

Tres niveles que el lenguaje trata de forma distinta. Esta es la clasificación formal de Oracle, y conviene aprenderla bien porque casi ningún tutorial la explica.

**Expresión** — combina variables, operadores e invocaciones para producir **un único valor**:

```java
1 + 2 + 3
saldo > 1000
lista.size()
cadence = 0          // sí: una asignación es una expresión, devuelve el valor asignado
```

Oracle avisa de dos cosas aquí. La primera es que **el orden de evaluación importa** salvo en operaciones conmutativas, y recomienda paréntesis explícitos aunque no hagan falta, por legibilidad:

```java
int r = 1 + 2 * 3;       // 7 — hay que saberse la precedencia
int r = 1 + (2 * 3);     // 7 — se lee solo
```

La segunda es la **aritmética de punto flotante**, que produce resultados que parecen imposibles:

```java
double d = 1.0 - 0.9;
System.out.println(d);          // 0.09999999999999998
System.out.println(d == 0.1);   // false
```

No es un bug de Java: `double` usa base 2 y no puede representar exactamente ciertos decimales. Regla práctica: **nunca compares `double` con `==`**, y para dinero usa `BigDecimal`.

**Sentencia** — una unidad completa de ejecución. Oracle distingue **tres tipos**:

| Tipo | Qué es | Ejemplo |
|---|---|---|
| De expresión | una expresión terminada en `;` | `saldo = 100;` `contador++;` `imprimir();` `new Factura();` |
| De declaración | declara una variable | `int total;` |
| De control de flujo | regula el orden de ejecución | `if`, `for`, `while`, `return` |

Solo cuatro clases de expresión valen como sentencia: **asignación, incremento/decremento, invocación de método y creación de objeto**. Por eso esto no compila:

```java
1 + 2;    // ✗ calcula un valor y lo tira — no es una sentencia válida
```

**Bloque** — en palabras de Oracle, *"a group of zero or more statements between balanced braces"*. Cuenta como **una sola sentencia**, y por eso puede ir donde se espera una:

```java
if (saldo > 0) {
    int comision = calcular();
    aplicar(comision);
}
```

Las variables declaradas dentro de un bloque solo existen dentro de él (*scope*).

### Espacios en blanco y formato

El compilador ignora por completo espacios, tabulaciones y saltos de línea. Esto es legal:

```java
public class Hola{public static void main(String[] a){System.out.println("x");}}
```

Compila perfecto y es indefendible. El formato existe para humanos. El estilo dominante:

- 4 espacios de indentación (no tabuladores).
- Llave de apertura al final de la línea, no en línea propia (estilo K&R).
- Un espacio alrededor de los operadores binarios: `a + b`, no `a+b`.
- Línea de unos 100-120 caracteres máximo.

No lo memorices: configura el formateador del IDE una vez y olvídate. Las referencias habituales son [Google Java Style](https://google.github.io/styleguide/javaguide.html) y las convenciones originales de Oracle.

### Comentarios

Sirven para dos cosas distintas: explicar el código, y **desactivar código temporalmente** mientras pruebas alternativas.

```java
// De una línea: todo lo que sigue hasta el fin de línea se ignora
System.out.println("Hello World");

System.out.println("Hello World");   // también al final de una línea

/* De varias líneas.
   Todo lo que hay entre las marcas se ignora. */

/**
 * Javadoc: documenta la API pública.
 * De este se genera documentación HTML automáticamente.
 *
 * @param cantidad importe a ingresar, debe ser positivo
 * @return el nuevo saldo tras el ingreso
 * @throws IllegalArgumentException si cantidad es negativa
 */
```

La convención habitual, tal como la formula W3Schools: **`//` para comentarios cortos, `/* */` para los largos.**

Solo el tercero es más que texto ignorado: `javadoc` lo procesa y tu IDE lo muestra al pasar el ratón sobre el método. Etiquetas más usadas: `@param`, `@return`, `@throws`, `@deprecated`, `@see`, `{@link}`.

Cuidado: los comentarios `/* */` **no anidan**. Un `*/` interior cierra el bloque y el resto se convierte en código roto.

### Paquetes e imports

```java
import java.util.List;              // clase concreta — lo habitual
import java.util.*;                 // todo el paquete — desaconsejado
import static java.lang.Math.PI;    // miembro estático: usas PI, no Math.PI
import static java.lang.Math.*;     // todos los estáticos de Math
```

Todo `java.lang` (`String`, `System`, `Integer`, `Math`, `Object`…) se importa solo. Por eso nunca escribes `import java.lang.String`, ni `import java.lang.System` para poder usar `System.out.println`.

El import con `*` no cuesta rendimiento (es una directiva de compilación, no carga nada extra), pero se evita porque oculta de dónde viene cada clase y provoca colisiones: si importas `java.util.*` y `java.awt.*`, `List` es ambiguo y no compila.

`import static` úsalo con moderación. En tests (`assertEquals`, `assertThat`) mejora mucho la lectura; en producción, abusar deja llamadas sin contexto de dónde salen.

### Modificadores: vista general

Se estudian a fondo en el bloque de POO, pero aparecen desde el primer día.

**De acceso:**

| Modificador | Visible desde |
|---|---|
| `public` | cualquier parte |
| `protected` | mismo paquete + subclases |
| *(ninguno)* | solo el mismo paquete (*package-private*) |
| `private` | solo la propia clase |

**De no acceso** (los más frecuentes): `static` (pertenece a la clase), `final` (no reasignable / heredable / sobrescribible), `abstract` (sin implementación), `synchronized` (control de concurrencia).

### Java 25: la versión compacta

Desde **Java 25** el programa mínimo se reduce a esto, sin clase envolvente y sin `static` ([JEP 512](https://openjdk.org/jeps/512)):

```java
void main() {
    IO.println("Hola, mundo");
}
```

El compilador envuelve el contenido en una clase implícita que nunca ves, e importa automáticamente las librerías de uso común. `IO` es una clase nueva (`java.lang.IO`) que evita tener que explicar `System.out` el primer día.

La JVM busca el punto de entrada en este orden: `public static void main(String[])` → `public static void main()` → `void main(String[])` → `void main()`.

Existe para que un principiante escriba su primer programa sin entender `public`, `static`, `void` y arrays de golpe. **Pero prácticamente todo el código real que vas a leer y escribir usa la forma clásica**, así que domina esa primero. Llegó en 2025 tras cuatro rondas de preview (JEPs 445, 463, 477 y 495).

---

## En la práctica

### Los errores que vas a cometer

**1. Comparar Strings con `==`**

El error número uno, y lo peor es que a veces *parece* funcionar:

```java
String a = "hola";
String b = "hola";
System.out.println(a == b);        // true  ← y esto te engaña

String c = new String("hola");
System.out.println(a == c);        // false ← el mismo texto, resultado distinto
System.out.println(a.equals(c));   // true  ← lo correcto
```

`==` compara **referencias** (¿son el mismo objeto en memoria?). `.equals()` compara **contenido**. Con literales de texto Java reutiliza el mismo objeto (ver *string pool*), y por eso el primer caso da `true` — lo cual es peor que si fallara siempre, porque te deja creer que entendiste algo que no entendiste.

**Regla:** con objetos, siempre `.equals()`. `==` solo para primitivos y para comprobar `null`.

**2. `Main method not found`**

Casi siempre es la firma mal escrita:

```java
public static void main(String args)      // ✗ falta [] — es un array
public void main(String[] args)           // ✗ falta static
public static void Main(String[] args)    // ✗ M mayúscula
public static int main(String[] args)     // ✗ debe ser void
```

**3. Punto y coma de más en un `if`**

```java
if (saldo > 1000);              // ← ese ; cierra el if aquí mismo
{
    System.out.println("Rico"); // se ejecuta SIEMPRE
}
```

Compila sin errores. El `;` es un bloque vacío y las llaves quedan sueltas.

**4. Confiar en la indentación**

```java
if (x > 0)
    System.out.println("positivo");
    System.out.println("siempre se imprime");   // ← fuera del if
```

Sin llaves, el `if` solo abarca la sentencia siguiente. **Pon siempre las llaves**, aunque el cuerpo sea una línea.

**5. `NullPointerException`**

Llamar a algo sobre una referencia que vale `null`. Desde **Java 14** ([JEP 358](https://openjdk.org/jeps/358), activado por defecto en 15) el mensaje dice exactamente qué era null:

```
Cannot invoke "String.length()" because "<local1>" is null
```

**6. El cero delante en un número**

```java
int codigoPostal = 08001;   // ✗ no compila: 8 y 9 no existen en octal
int hora = 010;             // compila, pero vale 8, no 10
```

**7. Nombre de archivo y clase distintos**

`public class Factura` dentro de `factura.java` no compila. La `F` importa.

**8. Comparar `double` con `==`**

```java
if (total == 0.1) { ... }   // puede fallar aunque "debería" ser 0.1
```

Usa un margen de tolerancia, o `BigDecimal` si hay dinero de por medio.

### Argumentos de línea de comandos

```java
public class Saludo {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Uso: java Saludo <nombre>");
            System.exit(1);              // código distinto de 0 = error
        }
        System.out.println("Hola, " + args[0]);
    }
}
```

```bash
java Saludo Ana        # → Hola, Ana
```

A diferencia de C, `args[0]` es el **primer argumento**, no el nombre del programa. Y `args` nunca es `null`: si no pasas nada, es un array de longitud 0.

### `var`: inferencia de tipos (Java 10+)

```java
var nombres = new ArrayList<String>();   // ✔ el tipo se ve en la misma línea
var total = 0;                           // ✔ obviamente int

var resultado = servicio.procesar();     // ✗ ¿qué devuelve esto?
```

`var` **no** convierte a Java en un lenguaje de tipado dinámico: el tipo sigue siendo fijo y se decide al compilar, simplemente no lo escribes. Solo vale para variables locales — no para campos, parámetros ni tipos de retorno. Y no puede inicializarse a `null`, porque entonces no hay tipo que inferir.

**Regla práctica:** úsalo cuando el tipo sea evidente leyendo esa misma línea.

### Text blocks (Java 15+)

```java
String consulta = """
        SELECT id, nombre
        FROM usuarios
        WHERE activo = true
        """;
```

La indentación común se elimina automáticamente, tomando como referencia la línea menos indentada (incluida la de las comillas de cierre). Dentro no hace falta escapar las comillas dobles.

### `System.out.println` fuera de un ejercicio

Para aprender está perfecto. En una aplicación real es un problema: no tiene niveles de severidad, no se puede filtrar ni desactivar por configuración, no lleva timestamp ni contexto, y escribe de forma síncrona y bloqueante. En producción se usa una librería de logging (SLF4J con Logback, típicamente).

---

## Avanzado

### Lo que el compilador escribe por ti

Buena parte de la sintaxis es **azúcar sintáctico**: formas cómodas que `javac` traduce a otra cosa.

**Concatenación de Strings**

```java
String saludo = "Hola, " + nombre + ". Tienes " + edad + " años.";
```

Los `String` son inmutables, así que esto no puede modificar nada en el sitio. Hasta Java 8, el compilador generaba un `StringBuilder` encadenando `append()`. Desde **Java 9** ([JEP 280](https://openjdk.org/jeps/280)) genera `invokedynamic` delegando en `StringConcatFactory`: la estrategia se decide en tiempo de ejecución, lo que permite a la JVM optimizarla mejor.

La consecuencia práctica no cambió: concatenar con `+` **dentro de un bucle** sigue siendo mala idea, porque se crea un objeto nuevo en cada vuelta. Ahí usas `StringBuilder`.

**El bucle for-each**

```java
for (String nombre : nombres) { ... }
```

se convierte en un bucle con `Iterator` — por eso funciona con cualquier `Iterable` y por eso modificar la colección mientras la recorres lanza `ConcurrentModificationException`.

**Autoboxing y la caché de enteros**

La conversión automática entre `int` e `Integer` se traduce a `Integer.valueOf()`, que mantiene una **caché de instancias entre -128 y 127**:

```java
Integer a = 127, b = 127;
System.out.println(a == b);   // true   ← misma instancia cacheada

Integer c = 128, d = 128;
System.out.println(c == d);   // false  ← instancias distintas
```

No es un bug: es la caché, y refuerza la regla de usar siempre `.equals()` con objetos.

### Los escapes Unicode se procesan *antes* de leer el código

El detalle de sintaxis más contraintuitivo de Java. Las secuencias `\uXXXX` no se resuelven al interpretar un `String`: se resuelven en una fase previa, **sobre el texto fuente entero**, antes incluso de separarlo en tokens.

Consecuencia: esto se ejecuta, a pesar de estar en un comentario.

```java
// \u000A System.out.println("esto sí se imprime");
```

`\u000A` es un salto de línea, así que el compilador termina el comentario ahí y lo que sigue pasa a ser código real. Por el mismo motivo esto **no** compila:

```java
String ruta = "C:\Users";     // ✗ \U... intenta leerse como escape Unicode
```

Rareza histórica: Java se diseñó para que el fuente fuera representable en ASCII puro. Aparece en entrevistas de nivel alto como pregunta trampa. En código real, nunca lo aproveches.

### Ver el bytecode

Nada de esto hay que creérselo. Se puede mirar:

```bash
javac Hola.java
javap -c Hola.class     # desensambla el bytecode
```

Es el mejor antídoto contra aprender Java de memoria: ves exactamente en qué se convirtió tu código.

### El string pool

Los literales de texto se guardan en una zona compartida (el *string pool*). Cuando el compilador ve `"hola"` dos veces, ambas referencias apuntan al mismo objeto. `new String("hola")` fuerza un objeto nuevo saltándose el pool — razón por la cual casi nunca deberías escribirlo. `.intern()` devuelve la versión del pool.

### Por qué `main` es `static`, de verdad

Sin `static`, la JVM tendría que crear una instancia de tu clase antes de llamar al método. ¿Con qué constructor? ¿Con qué argumentos? ¿Y si lanza una excepción? Es un problema del huevo y la gallina, y la solución histórica fue evitarlo: un método de clase no necesita instancia.

Java 25 lo resuelve por el otro lado — permite un `main` de instancia y hace que sea *la JVM* quien construya el objeto con el constructor sin argumentos.

---

## Trade-offs y entrevista

### Por qué Java es verboso, y qué compras con ello

| Ganas | Pagas |
|---|---|
| Errores detectados al compilar, no en producción | Más líneas para lo mismo |
| Refactor automático fiable (renombrar es seguro) | Ceremonia repetitiva |
| Código legible por alguien que no lo escribió | Curva inicial más lenta |
| Herramientas potentes (el IDE entiende todo) | Sensación de burocracia |

La verbosidad es una **apuesta deliberada**: optimizar la lectura y el mantenimiento a costa de la escritura. Se escribe una vez y se lee cientos. En un equipo grande, con código que vive años y rota de manos, esa apuesta suele salir a cuenta. En un script de 40 líneas, no — y por eso ahí se usa Python.

Java 8 en adelante (lambdas, `var`, records, text blocks, la forma compacta de Java 25) ha ido recortando ceremonia sin abandonar el tipado estático. La crítica de "Java es insoportablemente verboso" suele venir de alguien que lo dejó en la versión 7.

### `var`: ¿mejora la legibilidad o la esconde?

A favor: elimina la repetición absurda (`Map<String, List<Pedido>> m = new HashMap<>()`), y desplaza el foco al nombre de la variable, que suele informar más que el tipo.

En contra: al leer un *diff* en GitHub no tienes IDE, y `var x = servicio.buscar()` no te dice nada. El tipo explícito es documentación que no envejece.

Consenso práctico: `var` cuando el lado derecho hace obvio el tipo; tipo explícito cuando no.

### Preguntas de entrevista

**¿Java es compilado o interpretado?**
Los dos. `javac` compila a bytecode; la JVM lo interpreta al principio y, cuando detecta código caliente, lo compila a nativo con el JIT.

**¿Por qué `main` es `static`?**
Porque debe poder invocarse sin que exista ninguna instancia de la clase.

**Diferencia entre `==` y `.equals()`**
`==` compara referencias; `.equals()` compara contenido según lo que defina la clase.

**¿Por qué `Integer a = 127, b = 127; a == b` da `true` pero con `128` da `false`?**
Por la caché de `Integer.valueOf()` entre -128 y 127.

**¿Cuáles son los tres tipos de sentencia en Java?**
De expresión, de declaración y de control de flujo.

**¿Por qué `1 + 2;` no compila si `contador++;` sí?**
Solo cuatro clases de expresión valen como sentencia: asignación, incremento/decremento, invocación de método y creación de objeto.

**¿Cuántas clases `public` puede tener un archivo `.java`?**
Una como máximo, y debe llamarse igual que el archivo.

**¿`goto` existe en Java?**
Está reservada pero no implementada. Se reservó para dar un error claro a quien viniera de C++.

**¿Es `var` una palabra reservada?**
No: es un *nombre de tipo reservado*, contextual. `int var = 5;` compila.

**¿Diferencia entre `print` y `println`?**
`println` añade un salto de línea al final; `print` no.

**¿Por qué `0.1 + 0.2 != 0.3`?**
Porque `double` es binario en base 2 y no representa exactamente esos decimales. Para dinero, `BigDecimal`.

---

## Alcance y temas relacionados

| Lo que te habrás preguntado leyendo esto | Dónde se estudia |
|---|---|
| Qué le pasa a este código al compilarlo y ejecutarlo | [`02-lifecycle-of-a-program.md`](02-lifecycle-of-a-program.md) |
| Qué es `int`, `double`, `boolean`, y cuánto ocupan | [`03-data-types-and-variables.md`](03-data-types-and-variables.md) |
| Qué hace `+`, `%`, `&&`, `++`, y su precedencia | `04-operadores.md` |
| `if`, `switch`, `for`, `while`, `break` | `05-control-de-flujo.md` |
| `String[]` y cómo se manejan los arrays | `06-arrays.md` |
| Cómo se declaran y sobrecargan métodos | `07-metodos.md` |
| La clase `String` y sus operaciones | `08-strings.md` |

---

## Fuentes

Páginas efectivamente consultadas para este documento, no solo listadas:

**dev.java** (documentación oficial de Oracle)
- [Language Basics](https://dev.java/learn/language-basics/) — índice del recorrido
- [Creating Variables and Naming Them](https://dev.java/learn/language-basics/variables/) — reglas y convenciones de identificadores
- [Expressions, Statements and Blocks](https://dev.java/learn/language-basics/expressions-statements-blocks/) — los tres tipos de sentencia, definición de bloque, aviso sobre punto flotante

**W3Schools**
- [Java Syntax](https://www.w3schools.com/java/java_syntax.asp) — estructura mínima, relación archivo/clase
- [Java Output](https://www.w3schools.com/java/java_output.asp) — `println` vs `print`, desglose de `System.out`
- [Java Comments](https://www.w3schools.com/java/java_comments.asp) — tipos de comentario y convención de uso
- [Java Keywords](https://www.w3schools.com/java/java_ref_keywords.asp) — tabla de referencia (con las imprecisiones señaladas arriba)

**Referencia normativa**
- [The Java Language Specification](https://docs.oracle.com/javase/specs/) — capítulos 3 (estructura léxica) y 7 (paquetes)
- [JEP 512 — Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512)
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
