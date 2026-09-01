# Cheat Sheet · Basic Syntax

> Repaso rápido de [`Pura/Java/01-Basics/01-basic-syntax.md`](../../../Pura/Java/01-Basics/01-basic-syntax.md) · Java 17+

## En 30 segundos

- Java es de **tipado estático**: el compilador quiere saberlo todo antes de ejecutar nada. La verbosidad es el precio de esa verificación previa.
- Todo el código vive dentro de una clase; el archivo se llama igual que la clase `public` que contiene.
- `javac` produce **bytecode** (`.class`), la JVM lo ejecuta. De ahí el *write once, run anywhere*.
- El compilador ignora espacios y saltos de línea. El formato existe para humanos.
- Con objetos siempre `.equals()`; `==` solo para primitivos y para comprobar `null`.

## El programa mínimo

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

```bash
javac Main.java      # produce Main.class
java Main            # sin la extensión .class
java Main.java       # Java 11+: compila en memoria y ejecuta (JEP 330)
jshell               # Java 9+: REPL para probar sin escribir una clase
```

### Por qué la firma de `main` es exactamente esa

| Palabra | Significado | Por qué está ahí |
|---|---|---|
| `public` | accesible desde cualquier parte | la JVM lo invoca desde fuera de tu código |
| `static` | pertenece a la clase, no a un objeto | al arrancar todavía no existe ninguna instancia |
| `void` | no devuelve nada | el código de salida se da con `System.exit(int)` |
| `main` | nombre exacto | es lo que la JVM tiene codificado como punto de entrada |
| `String[] args` | argumentos de línea de comandos | `args[0]` es el **primer argumento**, no el nombre del programa |

`args` nunca es `null`: si no pasas nada, es un array de longitud 0. Variantes válidas: `String args[]` (heredada de C, desaconsejada) y `String... args`.

## Estructura de un archivo `.java`

El orden **no es libre**. Como mucho tres partes, en esta secuencia:

```java
package com.empresa.facturacion;   // 1. opcional, máximo uno, primera línea
import java.util.List;             // 2. los imports que hagan falta
public class Factura { }           // 3. declaraciones de tipo
```

- Como mucho **una clase `public`** por archivo, y debe llamarse igual que el archivo.
- El paquete debe coincidir con la ruta de carpetas.
- Todo `java.lang` (`String`, `System`, `Integer`, `Math`, `Object`…) se importa solo.
- `import java.util.*` no cuesta rendimiento, pero oculta de dónde viene cada clase y provoca colisiones (`List` es ambiguo si importás `java.util.*` y `java.awt.*`).

## Identificadores

**Lo que el compilador acepta:** empieza por letra, `$` o `_` (nunca por dígito); después letras, dígitos, `$` y `_`; Unicode admitido (`año` es legal); sensible a mayúsculas; no puede ser palabra reservada. Desde Java 9, `_` a secas ya no vale como nombre.

**Lo que la convención dicta**, que es distinto:

| Elemento | Convención | Ejemplo |
|---|---|---|
| Clase, interfaz, record, enum | `PascalCase` | `CuentaBancaria` |
| Método, variable | `camelCase` | `calcularSaldo` |
| Constante (`static final`) | `UPPER_SNAKE_CASE` | `TASA_MAXIMA` |
| Paquete | todo minúscula, sin guiones | `com.empresa.facturacion` |
| Parámetro de tipo genérico | una letra mayúscula | `T`, `E`, `K`, `V` |

## Palabras reservadas

50 palabras que el lenguaje se queda para sí, más tres literales reservados (`true`, `false`, `null`).

- `const` y `goto` están **reservadas pero sin uso**: existen solo para dar un error claro a quien venga de C++.
- **`var`, `record`, `sealed`, `permits`, `yield` NO son keywords**: son *palabras clave contextuales*, reservadas solo en ciertas posiciones. Por eso `int var = 5;` compila. Se diseñó así para añadir palabras al lenguaje sin romper código antiguo.

> W3Schools lista `var` como keyword y omite `const`, `goto` y `strictfp`. Es impreciso — contrastá con la especificación cuando algo huela raro.

## Literales

```java
int decimal = 42;
int hexadecimal = 0x2A;      // prefijo 0x
int octal = 052;             // prefijo 0 ← fuente clásica de bugs
int binario = 0b101010;      // prefijo 0b (Java 7+)
long grande = 9000000000L;   // L obligatorio si excede int
double d = 3.14;             // double por defecto
float f = 3.14f;             // f obligatorio
double cientifica = 1.5e3;   // 1500.0
char letra = 'A';            // comillas SIMPLES
String texto = "Hola";       // comillas DOBLES
int millon = 1_000_000;      // separador visual (Java 7+), el compilador lo ignora
```

`'A'` es un `char`; `"A"` es un `String`. No son intercambiables.

### Escapes

| Escape | Significado | | Escape | Significado |
|---|---|---|---|---|
| `\n` | salto de línea | | `\"` | comilla doble |
| `\t` | tabulación | | `\\` | barra invertida |
| `\r` | retorno de carro | | `\s` | espacio (Java 15+) |
| `\b` | retroceso | | `\uXXXX` | carácter Unicode |

## Expresiones, sentencias y bloques

La clasificación formal de Oracle, que casi ningún tutorial explica:

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Expresión** | produce **un único valor** | `1 + 2`, `saldo > 1000`, `lista.size()` |
| **Sentencia** | unidad completa de ejecución, termina en `;` | `saldo = 100;`, `if (...)`, `int total;` |
| **Bloque** | cero o más sentencias entre llaves balanceadas | `{ ... }` — cuenta como **una sola** sentencia |

Tres tipos de sentencia: **de expresión**, **de declaración** y **de control de flujo**.

Y solo cuatro clases de expresión valen como sentencia: **asignación, incremento/decremento, invocación de método y creación de objeto**. Por eso:

```java
contador++;   // ✔ compila
1 + 2;        // ✗ no compila: calcula un valor y lo tira
```

## Comentarios

```java
// De una línea
/* De varias líneas. NO anidan: el primer cierre termina el bloque */
/**
 * Javadoc: documenta la API pública, genera HTML, lo muestra el IDE.
 * @param cantidad importe a ingresar, debe ser positivo
 * @return el nuevo saldo
 * @throws IllegalArgumentException si cantidad es negativa
 */
```

## Modificadores

| De acceso | Visible desde |
|---|---|
| `public` | cualquier parte |
| `protected` | mismo paquete + subclases |
| *(ninguno)* | solo el mismo paquete (*package-private*) |
| `private` | solo la propia clase |

De no acceso más frecuentes: `static` (pertenece a la clase), `final` (no reasignable / heredable / sobrescribible), `abstract` (sin implementación), `synchronized` (concurrencia).

## Añadidos modernos que ya vas a ver

```java
// var — Java 10+: inferencia SOLO en variables locales
var nombres = new ArrayList<String>();   // ✔ el tipo se ve en la misma línea
var resultado = servicio.procesar();     // ✗ ¿qué devuelve esto?
var sinValor;                            // ✗ no hay de dónde inferir
var nulo = null;                         // ✗ null no tiene tipo

// Text blocks — Java 15+: la indentación común se elimina sola
String consulta = """
        SELECT id, nombre
        FROM usuarios
        WHERE activo = true
        """;

// Java 25 (JEP 512): forma compacta, sin clase envolvente ni static
void main() {
    IO.println("Hola, mundo");
}
```

`var` **no** hace a Java dinámicamente tipado: el tipo se decide al compilar y es igual de fijo. La forma compacta de Java 25 existe para el primer día de un principiante; **el código real usa la forma clásica**.

## Errores frecuentes → corrección

| ✗ MAL | Qué pasa | ✔ BIEN |
|---|---|---|
| `if (a == "hola")` | compara referencias; a veces "parece" funcionar por el string pool | `"hola".equals(a)` |
| `public static void main(String args)` | `Main method not found`: falta `[]` | `String[] args` |
| `if (saldo > 1000);` | el `;` cierra el `if`; el bloque se ejecuta **siempre** | quitar el `;` |
| `if (x > 0)` sin llaves y dos líneas debajo | solo la primera línea entra en el `if` | llaves siempre |
| `int cp = 08001;` | no compila: 8 y 9 no existen en octal | `int cp = 8001;` |
| `class Factura` en `factura.java` | no compila; la mayúscula importa | renombrar el archivo |
| `if (total == 0.1)` | `double` es binario, puede fallar | tolerancia, o `BigDecimal` |
| `System.out.println` en producción | sin niveles, sin filtro, síncrono y bloqueante | SLF4J + Logback |

## Lo que el compilador escribe por vos

| Escribís | Genera realmente |
|---|---|
| `"a" + b + "c"` | hasta Java 8, `StringBuilder`; desde Java 9 (JEP 280), `invokedynamic` → `StringConcatFactory` |
| `for (String s : lista)` | un bucle con `Iterator` — por eso modificar la colección lanza `ConcurrentModificationException` |
| `Integer i = 42;` | `Integer.valueOf(42)`, que **cachea de −128 a 127** |

```java
Integer a = 127, b = 127;   System.out.println(a == b);   // true  ← caché
Integer c = 128, d = 128;   System.out.println(c == d);   // false ← objetos distintos
```

**Rareza de nivel alto:** los escapes `\uXXXX` se resuelven **antes** de separar el fuente en tokens, sobre el texto entero. Por eso un `\u000A` dentro de un comentario de línea lo termina y convierte el resto en código real, y `String r = "C:\Users";` **no compila**.

Nada de esto hay que creérselo: `javac Hola.java && javap -c Hola.class` muestra en qué se convirtió tu código.

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Java es compilado o interpretado?** Los dos. `javac` compila a bytecode; la JVM lo interpreta y compila a nativo lo caliente con el JIT.
- **¿Por qué `main` es `static`?** Porque debe poder invocarse sin que exista ninguna instancia de la clase.
- **`==` vs `.equals()`.** `==` compara referencias; `.equals()` compara contenido según lo defina la clase.
- **¿Por qué `Integer 127 == 127` es `true` y con `128` es `false`?** Por la caché de `Integer.valueOf()` entre −128 y 127.
- **¿Los tres tipos de sentencia?** De expresión, de declaración y de control de flujo.
- **¿Por qué `1 + 2;` no compila si `contador++;` sí?** Solo cuatro clases de expresión valen como sentencia: asignación, incremento/decremento, invocación y creación de objeto.
- **¿Cuántas clases `public` por archivo?** Una como máximo, y con el nombre del archivo.
- **¿Existe `goto`?** Reservada pero no implementada, para dar un error claro a quien venga de C++.
- **¿Es `var` una palabra reservada?** No: es un nombre de tipo reservado, contextual. `int var = 5;` compila.
- **¿`print` o `println`?** `println` añade el salto de línea; `print` no.
- **¿Por qué `0.1 + 0.2 != 0.3`?** `double` es base 2 y no representa esos decimales exactamente. Para dinero, `BigDecimal`.

</details>
