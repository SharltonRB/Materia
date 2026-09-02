# Conditionals

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** Los capítulos anteriores construyeron las piezas: qué tipos existen ([Data Types and Variables](03-data-types-and-variables.md)), dónde vive cada variable ([Variables and Scopes](04-variables-and-scopes.md)), cómo se convierten unos tipos en otros ([Type Casting](05-type-casting.md)), cómo se opera con números ([Math Operations](07-math-operations.md)), cómo se producen valores `boolean` ([Logical, Relational and Bitwise Operators](08-logical-relational-bitwise-operators.md)) y cómo se guardan conjuntos de valores ([Arrays](09-arrays.md)). Todo eso describe programas que se ejecutan **de arriba abajo, siempre igual**.

Este capítulo rompe esa línea recta. Una **condicional** es la construcción que permite que el programa tome caminos distintos según los datos. Es la frontera entre una calculadora y un programa: sin condicionales no hay validación, no hay manejo de errores, no hay reglas de negocio.

El tema parece trivial —`if`, `else`, `switch`, tres palabras— y por eso casi todo el mundo lo aprende en veinte minutos y arrastra los huecos durante años. Los huecos importan porque las condicionales son **donde viven los bugs**. Una expresión aritmética mal escrita da un número raro y alguien lo nota; una condición mal escrita ejecuta la rama equivocada en el 3 % de los casos y nadie lo nota hasta que hay dinero de por medio.

La lista de bugs concretos que salen de este capítulo, todos reales:

- Un `if` seguido de un punto y coma que ejecuta su bloque **siempre**, y que el compilador acepta.
- Un `else` que se ata al `if` equivocado y cambia el significado del programa sin cambiar una sola llave.
- Un `NullPointerException` lanzado por un operador ternario en una línea donde no se ve ninguna llamada a método.
- Un `map.getOrDefault(clave, false)` que lanza `NullPointerException` aunque el nombre del método prometa lo contrario.
- Un `true ? unInteger : unDouble` que devuelve `1.0` cuando el programador esperaba `1`.
- Una comparación `==` entre dos `Integer` que funciona en desarrollo con valores pequeños y falla en producción con valores grandes.
- Un `switch` sin `break` que aplica dos descuentos en vez de uno.
- Un `switch` sobre `String` que lanza `NullPointerException` aunque tenga `default`.
- Un `switch` exhaustivo sobre un `enum` que compila hoy y lanza `MatchException` mañana, cuando alguien añade una constante al `enum` y no recompila.
- Una cadena de `if / else if` que evalúa el mismo método tres veces porque nadie se fijó en que la condición tenía efectos colaterales.
- Un `if (usuario.getPerfil().esAdmin())` que revienta con `null` porque las guardas estaban en el orden equivocado.

Vamos a cubrir el modelo completo: qué es una condición y por qué en Java tiene que ser exactamente un `boolean`, las tres construcciones (`if`, ternario, `switch`), la revolución que Java 14 y Java 21 hicieron con el `switch`, el *pattern matching* que convierte al `switch` en algo que ya no se parece a lo que era, y —lo más importante para el día a día— **cuándo la condicional sobra y hay que borrarla**.

**Sobre la verificación.** Todos los outputs, mensajes de excepción, errores del compilador y volcados de bytecode de este documento se ejecutaron realmente en un JDK 25. Como en los capítulos anteriores, las fuentes de referencia usadas para prepararlo contienen errores; una de ellas afirma literalmente que *"todas las sentencias condicionales devuelven bool"*, que es falso en Java de tres maneras distintas. Está todo señalado en la [sección 52](#52-fuentes) con el resultado real al lado.

---

## Índice

**Parte I — La decisión más simple**

1. [Qué es una condicional y qué problema resuelve](#1-qué-es-una-condicional-y-qué-problema-resuelve)
2. [La condición tiene que ser un boolean](#2-la-condición-tiene-que-ser-un-boolean)
3. [El if](#3-el-if)
4. [Las llaves que el compilador no exige y vos sí](#4-las-llaves-que-el-compilador-no-exige-y-vos-sí)
5. [El punto y coma fantasma](#5-el-punto-y-coma-fantasma)
6. [El else](#6-el-else)
7. [El else if no existe](#7-el-else-if-no-existe)
8. [El dangling else](#8-el-dangling-else)
9. [El orden de las ramas cambia el resultado](#9-el-orden-de-las-ramas-cambia-el-resultado)

**Parte II — Escribir condiciones que no mienten**

10. [Los operadores que producen la condición](#10-los-operadores-que-producen-la-condición)
11. [Cortocircuito y por qué no es solo una optimización](#11-cortocircuito-y-por-qué-no-es-solo-una-optimización)
12. [Negar bien con las leyes de De Morgan](#12-negar-bien-con-las-leyes-de-de-morgan)
13. [Comparar objetos con el operador de identidad](#13-comparar-objetos-con-el-operador-de-identidad)
14. [La caché de enteros y el bug que aparece en el 128](#14-la-caché-de-enteros-y-el-bug-que-aparece-en-el-128)
15. [Comparar cadenas de texto](#15-comparar-cadenas-de-texto)
16. [Comparar decimales](#16-comparar-decimales)
17. [Condiciones sobre null y el orden de las guardas](#17-condiciones-sobre-null-y-el-orden-de-las-guardas)
18. [Efectos colaterales dentro de una condición](#18-efectos-colaterales-dentro-de-una-condición)

**Parte III — El operador ternario**

19. [Qué es el ternario y en qué se diferencia del if](#19-qué-es-el-ternario-y-en-qué-se-diferencia-del-if)
20. [El ternario tiene tipo y ese tipo puede lanzar una excepción](#20-el-ternario-tiene-tipo-y-ese-tipo-puede-lanzar-una-excepción)
21. [La promoción numérica del ternario](#21-la-promoción-numérica-del-ternario)
22. [Ternarios anidados](#22-ternarios-anidados)
23. [Cuándo usar el ternario y cuándo no](#23-cuándo-usar-el-ternario-y-cuándo-no)

**Parte IV — El switch clásico**

24. [Qué es un switch y por qué no es una cadena de if](#24-qué-es-un-switch-y-por-qué-no-es-una-cadena-de-if)
25. [Sobre qué tipos se puede hacer switch](#25-sobre-qué-tipos-se-puede-hacer-switch)
26. [break y el fallthrough](#26-break-y-el-fallthrough)
27. [Cuándo el fallthrough es la respuesta correcta](#27-cuándo-el-fallthrough-es-la-respuesta-correcta)
28. [El default](#28-el-default)
29. [El scope compartido del switch clásico](#29-el-scope-compartido-del-switch-clásico)
30. [Qué hace por dentro un switch sobre String](#30-qué-hace-por-dentro-un-switch-sobre-string)
31. [tableswitch y lookupswitch](#31-tableswitch-y-lookupswitch)

**Parte V — El switch moderno**

32. [La flecha](#32-la-flecha)
33. [El switch como expresión](#33-el-switch-como-expresión)
34. [yield](#34-yield)
35. [Exhaustividad](#35-exhaustividad)
36. [El default implícito de los enums y MatchException](#36-el-default-implícito-de-los-enums-y-matchexception)
37. [Las cuatro formas del switch](#37-las-cuatro-formas-del-switch)
38. [El switch y null](#38-el-switch-y-null)

**Parte VI — Pattern matching**

39. [instanceof con patrón y flow scoping](#39-instanceof-con-patrón-y-flow-scoping)
40. [Patrones en las etiquetas case](#40-patrones-en-las-etiquetas-case)
41. [Guardas con when](#41-guardas-con-when)
42. [Dominancia entre etiquetas](#42-dominancia-entre-etiquetas)
43. [Record patterns](#43-record-patterns)
44. [Tipos sellados y uniones cerradas](#44-tipos-sellados-y-uniones-cerradas)
45. [Patrones sobre primitivos](#45-patrones-sobre-primitivos)

**Parte VII — Diseño**

46. [Cuándo la condicional sobra](#46-cuándo-la-condicional-sobra)
47. [Anidamiento, complejidad ciclomática y cláusulas de guarda](#47-anidamiento-complejidad-ciclomática-y-cláusulas-de-guarda)

**Parte VIII — Cierre**

48. [Casos de uso reales](#48-casos-de-uso-reales)
49. [Anti-patrones](#49-anti-patrones)
50. [Checklist y tabla de decisión](#50-checklist-y-tabla-de-decisión)
51. [Autoevaluación](#51-autoevaluación)
52. [Fuentes](#52-fuentes)

---

# Parte I — La decisión más simple

## 1. Qué es una condicional y qué problema resuelve

Todo lo escrito hasta este capítulo se ejecuta en línea recta. El programa arranca en la primera instrucción, sigue a la segunda, y así hasta el final. Da igual cuántas veces lo corras y con qué datos: hace exactamente lo mismo.

Eso sirve para calcular, no para decidir. Un programa útil necesita responder preguntas del tipo "¿este usuario tiene permiso?", "¿el carrito está vacío?", "¿la fecha ya pasó?", y **hacer cosas distintas según la respuesta**.

Una **condicional** es una construcción del lenguaje que evalúa una condición y, según el resultado, ejecuta un bloque de código u otro. Es la forma que tiene un lenguaje de programación de expresar una bifurcación.

La definición que da una de las fuentes de este capítulo es correcta en su idea general: *"conditional statements are features of programming languages that tell the computer to execute certain actions, provided certain conditions are met"*. Lo que le falta es la parte que importa en Java, y que vamos a construir a lo largo del documento: **en Java hay varias construcciones condicionales y no son intercambiables**, porque unas son *sentencias* y otras son *expresiones*, y esa diferencia tiene consecuencias que llegan hasta el sistema de tipos.

Java tiene exactamente estas herramientas para decidir:

| Construcción | Qué es | Produce un valor | Desde |
|---|---|---|---|
| `if` / `else` | Sentencia | No | Java 1.0 |
| Operador ternario `? :` | Expresión | Sí | Java 1.0 |
| `switch` (sentencia) | Sentencia | No | Java 1.0 |
| `switch` (expresión) | Expresión | Sí | Java 14 |

Hay una forma más de bifurcar que no es una condicional pero cumple la misma función: el **polimorfismo**. La veremos en la [sección 46](#46-cuándo-la-condicional-sobra), porque buena parte de las condicionales que se escriben en Java no deberían existir.

> **Vocabulario.** A lo largo del capítulo se distingue *sentencia* (statement) de *expresión* (expression). Una **expresión** produce un valor y por tanto tiene un tipo: `2 + 2`, `nombre.length()`, `edad > 18`. Una **sentencia** hace algo pero no vale nada: no se puede asignar a una variable. Un `if` es una sentencia; `int x = if (a) 1 else 2;` no compila en Java, aunque sí en Kotlin o Scala. Esta distinción, que ahora parece formal, es la que explica por qué existe el operador ternario y por qué Java 14 tuvo que inventar el `switch` como expresión.

## 2. La condición tiene que ser un boolean

En Java, la expresión que va dentro de los paréntesis de un `if` **tiene que ser de tipo `boolean` o `Boolean`**. No "algo que se parezca a verdadero". No un número. No una referencia.

Esto separa a Java de C, C++, JavaScript o Python, donde el `0`, la cadena vacía, `null` o una lista vacía se consideran *falsy* y cualquier otra cosa *truthy*. En Java ese concepto no existe:

```java
int x = 5;
if (x) { }        // NO COMPILA
String s = "hola";
if (s) { }        // NO COMPILA
```

El compilador dice exactamente:

```
error: incompatible types: int cannot be converted to boolean
```

Parece una restricción molesta y es una de las mejores decisiones de diseño del lenguaje, porque elimina de raíz el bug más clásico de C:

```java
int x = 0;
if (x = 5) { }    // NO COMPILA en Java
```

Verificado en JDK 25, el error es:

```
E9.java:3: error: incompatible types: int cannot be converted to boolean
        if (x = 5) { }
              ^
```

En C esa línea compila, asigna `5` a `x`, evalúa el resultado de la asignación (`5`, que es distinto de cero, o sea *truthy*) y entra en el `if`. Es una fuente inagotable de bugs que en Java simplemente no puede ocurrir.

**El matiz que sí muerde.** La restricción no protege cuando la variable ya es `boolean`:

```java
boolean activo = false;
if (activo = true) {        // COMPILA. Asigna, no compara.
    System.out.println("siempre entra, y además dejó activo en true");
}
```

Esto compila porque `activo = true` es una expresión de tipo `boolean`. Es el único caso en que el bug de C sobrevive en Java. La defensa es no escribir nunca `== true`: en lugar de `if (activo == true)` se escribe `if (activo)`, y en lugar de `if (activo == false)` se escribe `if (!activo)`. Si nunca escribís `= true` dentro de un `if`, el bug no aparece.

**El otro matiz: `Boolean` con mayúscula.** La condición también acepta un `Boolean` (el wrapper), porque Java le aplica *unboxing* automático. Y ahí sí hay un agujero:

```java
Boolean permitido = null;
if (permitido) {           // COMPILA, y lanza NullPointerException
    System.out.println("nunca llega aquí");
}
```

El *unboxing* de `null` llama a `permitido.booleanValue()` sobre una referencia nula. En JDK 25, gracias a los *helpful NullPointerExceptions*, el mensaje lo dice con nombre y apellido:

```
Cannot invoke "java.lang.Boolean.booleanValue()" because "permitido" is null
```

Este caso reaparece con más fuerza en la [sección 20](#20-el-ternario-tiene-tipo-y-ese-tipo-puede-lanzar-una-excepción), porque el operador ternario lo produce en situaciones donde nadie lo espera.

## 3. El if

La forma básica:

```java
if (condicion) {
    // se ejecuta solo si condicion es true
}
```

Un ejemplo mínimo, con la condición escrita directamente:

```java
if (20 > 18) {
    System.out.println("20 es mayor que 18");
}
```

Lo normal es que la condición use variables:

```java
int x = 20;
int y = 18;
if (x > y) {
    System.out.println("x es mayor que y");
}
```

Y lo más limpio es que la condición sea ya un `boolean` con nombre:

```java
boolean luzEncendida = true;
if (luzEncendida) {
    System.out.println("La luz está encendida");
}
```

Esa tercera forma merece una nota de estilo que no es cosmética. Comparar:

```java
if (u.getEdad() >= 18 && u.getPais().equals("AR") && !u.estaBloqueado()) { ... }
```

con:

```java
boolean esAdultoArgentinoActivo = u.getEdad() >= 18
        && u.getPais().equals("AR")
        && !u.estaBloqueado();

if (esAdultoArgentinoActivo) { ... }
```

La segunda versión no es más rápida ni más corta: es **nombrable**. El nombre de la variable documenta la intención, y cuando la condición falla en producción se puede loguear ese `boolean` sin recalcular nada. Extraer condiciones complejas a variables o a métodos con nombre es la técnica más barata que existe para hacer legible un fichero lleno de condicionales.

## 4. Las llaves que el compilador no exige y vos sí

Java permite omitir las llaves cuando el cuerpo del `if` es **una sola sentencia**:

```java
if (x > 10)
    System.out.println("mayor que 10");
```

Eso funciona. El problema aparece al añadir una segunda línea:

```java
if (x > 10)
    System.out.println("mayor que 10");
    System.out.println("esto se imprime SIEMPRE");
```

La indentación sugiere que las dos líneas están dentro del `if`. No es así: sin llaves, **solo la primera sentencia pertenece al `if`**. La segunda está fuera y se ejecuta pase lo que pase. Java ignora la indentación por completo; es un detalle visual sin significado para el compilador.

Este error tiene un nombre propio en la industria porque causó una vulnerabilidad grave real: el bug **"goto fail"** de Apple en 2014, en el código de verificación de certificados TLS de iOS y macOS. Un `goto fail;` duplicado y sin llaves hacía que la validación de la firma del servidor se saltara siempre, y el `if` que debía protegerla no protegía nada. Todas las conexiones TLS quedaban expuestas a un ataque de intermediario. Era C, no Java, pero la construcción del lenguaje es idéntica y el fallo se replica igual.

La regla práctica, que recomiendan tanto la fuente de este capítulo (*"always using braces makes your code clearer, easier to read, and prevents subtle bugs"*) como todas las guías de estilo serias:

> **Poné siempre las llaves.** Incluso para una sola línea. Incluso si la línea es corta. El coste son dos caracteres; el beneficio es que añadir una línea seis meses después no cambia el significado del programa.

La única excepción defendible, y hay desacuerdo genuino sobre ella entre desarrolladores con experiencia, es la cláusula de guarda de una sola línea al principio de un método:

```java
if (pedido == null) return;
```

Cabe en un renglón, se lee como una unidad y no invita a añadir nada debajo. La [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html#s4.1.1-braces-always-used) no la permite —exige llaves siempre—, mientras que otras guías sí. Elegí una postura y aplicala en todo el proyecto; lo que hace daño es mezclar.

## 5. El punto y coma fantasma

Este es distinto y es peor, porque el error es invisible:

```java
int x = 5;
if (x > 100);
{
    System.out.println("esto se ejecuta SIEMPRE, aunque x valga 5");
}
```

Salida real en JDK 25:

```
G1 este bloque se ejecuta SIEMPRE aunque x=5
```

Lo que pasa es que `;` es una **sentencia vacía**, y es una sentencia válida en Java. El `if` la toma como su cuerpo completo. Cuando `x > 100` es verdadero, ejecuta la nada. El bloque de llaves que viene después no es el cuerpo del `if`: es un bloque suelto, independiente, que se ejecuta siempre.

El compilador no da error porque el código es legal. Sí da un aviso, pero **solo si se lo pedís explícitamente**:

```
$ javac -Xlint:all Ejemplo.java
Ejemplo.java:124: warning: [empty] empty statement after if
        if (x > 100) ;
                     ^
```

Sin `-Xlint`, silencio absoluto. Conviene activar `-Xlint:all` en el build del proyecto y, mejor todavía, tratar los avisos como errores con `-Werror`. En Maven se configura en el `maven-compiler-plugin`; en Gradle, con `options.compilerArgs`.

## 6. El else

El `else` ejecuta un bloque cuando la condición del `if` es falsa. No lleva condición propia, precisamente porque su condición es "todo lo demás":

```java
if (condicion) {
    // condicion es true
} else {
    // condicion es false
}
```

Un ejemplo:

```java
boolean estaLloviendo = false;

if (estaLloviendo) {
    System.out.println("¡Llevate un paraguas!");
} else {
    System.out.println("No llueve, no hace falta paraguas");
}
```

Salida: `No llueve, no hace falta paraguas`.

Otro, con una comparación numérica:

```java
int hora = 20;

if (hora < 18) {
    System.out.println("Buen día.");
} else {
    System.out.println("Buenas tardes.");
}
```

Salida: `Buenas tardes.`

La garantía que da el par `if / else` es que **exactamente una de las dos ramas se ejecuta**: nunca las dos, nunca ninguna. Eso lo convierte en la herramienta correcta para expresar una alternativa real y binaria. Cuando lo que tenés no es una alternativa sino una serie de comprobaciones independientes, el `if / else` es la construcción equivocada y conviene revisar el diseño.

## 7. El else if no existe

Esto sorprende a mucha gente, y entenderlo aclara varias cosas de golpe: **`else if` no es una construcción de Java**. No hay ninguna palabra clave `elseif` como el `elif` de Python. Lo que existe es un `else` cuyo cuerpo es, a su vez, otro `if`.

Estas dos versiones son el mismo programa:

```java
// Como se escribe siempre
if (hora < 12) {
    System.out.println("Buenos días.");
} else if (hora < 18) {
    System.out.println("Buenas tardes.");
} else {
    System.out.println("Buenas noches.");
}
```

```java
// Lo que realmente entiende el compilador
if (hora < 12) {
    System.out.println("Buenos días.");
} else {
    if (hora < 18) {
        System.out.println("Buenas tardes.");
    } else {
        System.out.println("Buenas noches.");
    }
}
```

La convención de escribir `} else if (` en la misma línea existe solo para que una cadena de diez casos no se indente diez niveles a la derecha. Es azúcar visual, no sintáctico.

De aquí salen tres consecuencias prácticas:

1. **Las condiciones se evalúan de arriba abajo y la primera que sea verdadera gana.** Las demás ni siquiera se evalúan.
2. **Solo se ejecuta una rama.** Toda la cadena es un único `if` anidado, así que en cuanto una condición acierta, el resto del árbol se descarta.
3. **El `else` final es opcional.** Si no está y ninguna condición se cumple, no se ejecuta nada. Ese "no se ejecuta nada" silencioso es una fuente clásica de bugs; lo trata la [sección 28](#28-el-default) al hablar del `default` del `switch`, donde el problema es idéntico.

## 8. El dangling else

Como `else if` es en realidad un `if` anidado, aparece una ambigüedad conocida: cuando hay dos `if` y un solo `else`, ¿a cuál pertenece el `else`?

```java
int a = 1, b = 99;

if (a > 0)
    if (b > 100)
        System.out.println("b es mayor que 100");
    else
        System.out.println("¿a qué if pertenece este else?");
```

La indentación sugiere que el `else` es del `if` externo, y que como `a > 0` es verdadero, no debería imprimirse nada. Ejecutado en JDK 25, la salida real es:

```
H1 el else se ata al if INTERNO, no al externo
```

**La regla de Java es que el `else` se asocia siempre al `if` más cercano que aún no tenga `else`.** Está en el [JLS §14.5](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html), y es la misma regla de C, C++ y C#.

La solución es la de la sección 4: **llaves siempre**. Con llaves, la ambigüedad desaparece porque el bloque marca el límite:

```java
if (a > 0) {
    if (b > 100) {
        System.out.println("b es mayor que 100");
    }
} else {
    System.out.println("ahora sí: este else es del if externo");
}
```

## 9. El orden de las ramas cambia el resultado

En una cadena de `if / else if`, las condiciones se prueban en orden y gana la primera que acierta. Por lo tanto **el orden es parte de la lógica, no un detalle de presentación**.

El error clásico es poner una condición general antes de una específica:

```java
// MAL: la primera rama se traga todo
int nota = 95;

if (nota >= 60) {
    System.out.println("Aprobado");
} else if (nota >= 90) {
    System.out.println("Sobresaliente");   // INALCANZABLE
}
```

Con `nota = 95`, la primera condición ya es verdadera, así que imprime `Aprobado` y nunca llega a la segunda. La rama de sobresaliente es código muerto, y **el compilador no avisa**: a diferencia de lo que ocurre con el `switch` moderno —donde este mismo error es un error de compilación, como veremos en la [sección 42](#42-dominancia-entre-etiquetas)—, en una cadena de `if` es perfectamente legal.

La versión correcta ordena de más específico a más general:

```java
if (nota >= 90) {
    System.out.println("Sobresaliente");
} else if (nota >= 60) {
    System.out.println("Aprobado");
} else {
    System.out.println("Suspenso");
}
```

Fijate en algo que se aprende tarde: en la versión correcta, la segunda condición **no necesita** escribirse como `nota >= 60 && nota < 90`. El `else` ya garantiza que `nota < 90`, porque si no lo fuera habría entrado en la rama anterior. Escribir el rango completo en cada rama es redundante, alarga las condiciones y —esto es lo importante— **crea la posibilidad de que los rangos no encajen** y quede un hueco por el que no pasa nadie.

---

# Parte II — Escribir condiciones que no mienten

## 10. Los operadores que producen la condición

Esta sección es un recordatorio; el tratamiento completo está en [Logical, Relational and Bitwise Operators](08-logical-relational-bitwise-operators.md). Lo que hace falta aquí es la tabla de referencia.

**Operadores relacionales** — comparan y producen un `boolean`:

| Operador | Significado | Ejemplo verdadero |
|---|---|---|
| `<` | menor que | `3 < 5` |
| `<=` | menor o igual | `5 <= 5` |
| `>` | mayor que | `7 > 2` |
| `>=` | mayor o igual | `5 >= 5` |
| `==` | igual a | `4 == 4` |
| `!=` | distinto de | `4 != 5` |

Los cuatro de orden (`<`, `<=`, `>`, `>=`) solo funcionan con tipos numéricos primitivos (y sus wrappers, con *unboxing*). No funcionan con `String`, ni con objetos, ni con `boolean`. Para ordenar objetos está `compareTo`, que se ve en el bloque de colecciones.

**Operadores lógicos** — combinan condiciones:

| Operador | Nombre | Verdadero cuando |
|---|---|---|
| `&&` | AND condicional | ambos operandos son verdaderos |
| `\|\|` | OR condicional | al menos uno es verdadero |
| `!` | NOT | el operando es falso |
| `&` | AND lógico | ambos son verdaderos, sin cortocircuito |
| `\|` | OR lógico | al menos uno es verdadero, sin cortocircuito |
| `^` | XOR | exactamente uno es verdadero |

Un ejemplo de cada uno de los tres primeros, que son los que se usan a diario:

```java
int a = 200, b = 33, c = 500;

if (a > b && c > a) {
    System.out.println("ambas condiciones se cumplen");
}

if (a > b || a > c) {
    System.out.println("al menos una se cumple");
}

if (!(a > b)) {
    System.out.println("a NO es mayor que b");
}
```

Y un ejemplo combinado, del tipo que aparece en cualquier sistema con control de acceso:

```java
boolean estaLogueado = true;
boolean esAdmin = false;
int nivelSeguridad = 2;

if (estaLogueado && (esAdmin || (nivelSeguridad >= 1 && nivelSeguridad <= 2))) {
    System.out.println("Acceso concedido");
} else {
    System.out.println("Acceso denegado");
}
```

Los paréntesis de ese ejemplo no son opcionales por precedencia —`&&` liga más fuerte que `||`, así que sin ellos el significado cambiaría— pero aunque lo fueran, conviene ponerlos. Una condición con tres operadores lógicos y sin paréntesis es una invitación a que el siguiente lector la interprete mal.

## 11. Cortocircuito y por qué no es solo una optimización

`&&` y `||` **cortocircuitan**: si el resultado ya está decidido con el operando izquierdo, el derecho no se evalúa.

- `false && loQueSea` es `false` sin mirar la derecha.
- `true || loQueSea` es `true` sin mirar la derecha.

Sus primos `&` y `|` hacen la misma operación lógica pero **siempre evalúan los dos lados**. Verificado en JDK 25 con un método que imprime al ser llamado:

```java
static boolean efecto(String etiqueta) {
    System.out.println("   evaluado: " + etiqueta);
    return true;
}

if (false && efecto("derecha-&&")) { }   // no imprime nada
if (false &  efecto("derecha-&"))  { }   // imprime "evaluado: derecha-&"
```

Salida real:

```
F1 con &&:
F2 con & (no cortocircuita):
   evaluado: derecha-&
```

Mucha gente aprende esto como "`&&` es más rápido porque a veces se ahorra una evaluación". Esa lectura es cierta y es la menos importante. **El cortocircuito es un mecanismo de corrección, no de rendimiento**, y el idioma que lo demuestra es este:

```java
if (usuario != null && usuario.estaActivo()) { ... }
```

Si `usuario` es `null`, la parte izquierda es `false` y `usuario.estaActivo()` **nunca se llama**. Sin cortocircuito, esa línea lanzaría `NullPointerException`. Cambiala por `&` y el programa revienta:

```java
if (usuario != null & usuario.estaActivo()) { ... }   // NullPointerException si es null
```

El mismo patrón, con `||`, protege el caso contrario:

```java
if (texto == null || texto.isBlank()) {
    throw new IllegalArgumentException("el texto es obligatorio");
}
```

Si `texto` es `null`, la izquierda es `true`, se cortocircuita, y `isBlank()` no llega a llamarse.

> **Regla práctica.** En una condición, usá siempre `&&` y `||`. Los operadores `&` y `|` sobre `boolean` existen por completitud del lenguaje y porque son los mismos símbolos que las operaciones bit a bit sobre enteros, pero en una condición no aportan nada y sí quitan la protección del cortocircuito. Si ves un `&` entre dos booleanos en código real, en la enorme mayoría de los casos es una errata de `&&`.

## 12. Negar bien con las leyes de De Morgan

Negar una condición compuesta es donde más se equivoca la gente. La intuición dice que para negar `a && b` hay que escribir `!a && !b`, y es falso.

Las **leyes de De Morgan** dan la transformación correcta:

```
!(a && b)  ==  !a || !b
!(a || b)  ==  !a && !b
```

Lo importante es que **el operador cambia**: el AND se vuelve OR y el OR se vuelve AND. En código:

```java
// Original
if (estaLogueado && tienePermiso) { permitir(); }

// Negación CORRECTA
if (!estaLogueado || !tienePermiso) { denegar(); }

// Negación INCORRECTA — es una condición distinta
if (!estaLogueado && !tienePermiso) { denegar(); }
```

La tercera versión solo deniega cuando fallan **las dos** cosas. Un usuario logueado pero sin permiso pasa el filtro. Es exactamente el aspecto que tiene una vulnerabilidad de autorización en una revisión de código apresurada.

Una comprobación exhaustiva sobre las cuatro combinaciones posibles deja el punto cerrado:

| `estaLogueado` | `tienePermiso` | `!(a && b)` correcto | `!a && !b` incorrecto |
|---|---|---|---|
| `true` | `true` | `false` | `false` |
| `true` | `false` | **`true`** | **`false`** ← discrepan |
| `false` | `true` | **`true`** | **`false`** ← discrepan |
| `false` | `false` | `true` | `true` |

En la práctica, la mejor defensa contra este error es no negar condiciones compuestas: darles nombre y negar el nombre.

```java
boolean puedeOperar = estaLogueado && tienePermiso;
if (!puedeOperar) { denegar(); }
```

## 13. Comparar objetos con el operador de identidad

Con primitivos, `==` compara valores y hace lo que uno espera. Con **referencias**, `==` compara identidad: pregunta si las dos variables apuntan al **mismo objeto en memoria**, no si su contenido es equivalente.

```java
String a = new String("hola");
String b = new String("hola");

System.out.println(a == b);        // false — son dos objetos distintos
System.out.println(a.equals(b));   // true  — su contenido es igual
```

Esta es la causa número uno de bugs de comparación en Java. La regla es simple de enunciar:

> Para primitivos, `==`. Para objetos, `equals`.

Lo que la complica es que **`==` compila siempre** sobre referencias de tipos compatibles. El compilador no puede saber que querías comparar contenido, así que no avisa. El bug es silencioso y depende de dónde vinieron los objetos.

Hay dos matices que conviene conocer:

**Primero, `equals` sobre `null`.** Llamar a `a.equals(b)` con `a` nulo lanza `NullPointerException`. Cuando el que puede ser nulo es el de la izquierda, hay dos salidas idiomáticas:

```java
// Objects.equals maneja null en ambos lados
if (Objects.equals(a, b)) { ... }

// O poner la constante a la izquierda (idioma "Yoda")
if ("ADMIN".equals(rol)) { ... }   // seguro aunque rol sea null
```

El segundo idioma se llama *Yoda condition* y genera discusión: es seguro pero se lee al revés. Con `Objects.equals` disponible desde Java 7, hoy es preferible la primera forma salvo que compares contra un literal, donde el idioma Yoda sigue siendo compacto y claro.

**Segundo, `equals` tiene que estar implementado.** `equals` viene heredado de `Object`, y la implementación por defecto **compara identidad**, exactamente igual que `==`. Si tu clase no sobrescribe `equals`, llamarlo no aporta nada:

```java
class Punto {
    int x, y;
    Punto(int x, int y) { this.x = x; this.y = y; }
}

Punto p1 = new Punto(1, 2);
Punto p2 = new Punto(1, 2);
System.out.println(p1.equals(p2));   // false — no hay equals propio
```

Los `record` resuelven esto de oficio, porque el compilador les genera `equals`, `hashCode` y `toString` a partir de sus componentes:

```java
record Punto(int x, int y) {}

Punto p1 = new Punto(1, 2);
Punto p2 = new Punto(1, 2);
System.out.println(p1.equals(p2));   // true
```

## 14. La caché de enteros y el bug que aparece en el 128

Este merece sección propia porque es el bug de condicionales más traicionero de Java: **funciona con valores pequeños y falla con valores grandes**.

Verificado en JDK 25:

```java
Integer a = 127, b = 127;
Integer c = 128, d = 128;

System.out.println(a == b);        // true
System.out.println(c == d);        // false
System.out.println(c.equals(d));   // true
```

Salida real:

```
P1 127==127 -> true
P2 128==128 -> false
P3 equals   -> true
```

La razón está en el *autoboxing*. Cuando el compilador convierte un `int` en un `Integer`, no llama a `new Integer(...)`: llama a `Integer.valueOf(int)`. Y ese método mantiene una **caché de instancias** para el rango de `-128` a `127`, que el [JLS §5.1.7](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.7) exige que exista. Dentro de ese rango, dos *boxings* del mismo número devuelven **el mismo objeto**, así que `==` da `true` por casualidad. Fuera del rango, se crean objetos nuevos y `==` da `false`.

Por qué es tan peligroso:

- Los tests unitarios se escriben con valores pequeños (`1`, `2`, `42`), y pasan.
- Producción usa ids reales, precios, contadores. Ahí falla.
- El límite superior de la caché es configurable con `-XX:AutoBoxCacheMax`, así que el mismo código puede comportarse distinto en dos entornos con distintos flags de JVM.

La regla no cambia: **nunca compares wrappers con `==`**. Y si lo que tenés es un `Integer` y lo que necesitás es un `int`, desempaquetalo explícitamente y comparalo como primitivo:

```java
if (idA.intValue() == idB.intValue()) { ... }   // correcto y explícito
if (Objects.equals(idA, idB)) { ... }           // correcto y tolera null
```

El mismo comportamiento afecta a `Long`, `Short`, `Byte` y `Character`. `Boolean` cachea sus dos únicos valores, así que `==` siempre "funciona" ahí, lo cual solo sirve para reforzar la costumbre equivocada.

## 15. Comparar cadenas de texto

Las `String` combinan los dos problemas anteriores y añaden uno propio: el **pool de cadenas**. Verificado en JDK 25:

```java
String s1 = "hola";
String s2 = "hola";
String s3 = new String("hola");
String s4 = "ho" + "la";          // constante, resuelta en compilación
String parte = "ho";
String s5 = parte + "la";         // concatenación en ejecución
```

| Comparación | Resultado real | Por qué |
|---|---|---|
| `s1 == s2` | `true` | los literales idénticos comparten instancia en el pool |
| `s1 == s3` | `false` | `new String` fuerza un objeto nuevo |
| `s1 == s4` | `true` | `"ho" + "la"` es una constante en tiempo de compilación |
| `s1 == s5` | `false` | la concatenación en ejecución crea un objeto nuevo |
| `s1.equals(s5)` | `true` | el contenido es el mismo |

La fila que hace daño es la cuarta. `s4` y `s5` son visualmente la misma operación; la única diferencia es que en `s4` los dos trozos son literales que el compilador puede juntar, y en `s5` uno viene de una variable. Un cambio inocente —extraer `"ho"` a una variable para reutilizarla— convierte un `==` que funcionaba en un `==` que falla.

> **Regla sin excepciones:** comparar `String` con `==` está mal aunque funcione. Usá `equals`, o `equalsIgnoreCase` cuando las mayúsculas no importen.

Dato conectado con la Parte IV: el `switch` sobre `String` **no** tiene este problema, porque internamente usa `equals`, no `==`. Verificado:

```java
String s5 = parte + "la";                   // objeto nuevo, s5 != "hola"
String r = switch (s5) {
    case "hola" -> "el switch SÍ casa";
    default -> "no casa";
};
// imprime: el switch SÍ casa
```

Por qué funciona así está en la [sección 30](#30-qué-hace-por-dentro-un-switch-sobre-string).

## 16. Comparar decimales

`double` y `float` no representan exactamente la mayoría de los números decimales, así que compararlos con `==` es pedir problemas:

```java
if (0.1 + 0.2 == 0.3) {
    System.out.println("nunca entra");
}
System.out.println(0.1 + 0.2);   // 0.30000000000000004
```

Las tres salidas correctas, en orden de preferencia:

```java
// 1. Comparar con tolerancia (epsilon)
double epsilon = 1e-9;
if (Math.abs(a - b) < epsilon) { ... }

// 2. Para dinero y cualquier cosa con decimales exactos: BigDecimal
if (precio.compareTo(otroPrecio) == 0) { ... }

// 3. Trabajar en enteros (céntimos en vez de euros)
if (centimosA == centimosB) { ... }
```

Ojo con `BigDecimal`: su `equals` compara **también la escala**, así que `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` es `false`. Para comparar valor numérico hay que usar `compareTo(...) == 0`. Es un caso donde `equals` existe, está bien implementado, y aun así no es lo que querés.

El tratamiento completo del punto flotante está en [Math Operations](07-math-operations.md); aquí basta con la consecuencia para las condicionales.

## 17. Condiciones sobre null y el orden de las guardas

Cuando una condición navega por una cadena de objetos, el orden de las comprobaciones es lo único que separa el código correcto del `NullPointerException`:

```java
// MAL: si usuario es null, revienta antes de comprobarlo
if (usuario.getPerfil().esAdmin() && usuario != null) { ... }
```

Dos errores en una línea: se llama a `getPerfil()` antes de comprobar que `usuario` no es nulo, y no se comprueba el resultado de `getPerfil()`. La versión correcta aprovecha el cortocircuito para encadenar guardas de fuera hacia dentro:

```java
if (usuario != null && usuario.getPerfil() != null && usuario.getPerfil().esAdmin()) { ... }
```

Funciona, pero llama a `getPerfil()` dos veces y se lee mal. Tres alternativas mejores, según el contexto:

```java
// 1. Variable intermedia: una sola llamada, más legible
Perfil perfil = usuario == null ? null : usuario.getPerfil();
if (perfil != null && perfil.esAdmin()) { ... }

// 2. Optional, cuando la API ya lo devuelve
boolean esAdmin = Optional.ofNullable(usuario)
        .map(Usuario::getPerfil)
        .map(Perfil::esAdmin)
        .orElse(false);

// 3. La mejor: que el null no exista
//    Si Usuario garantiza que perfil nunca es null (validado en el constructor),
//    la condición vuelve a ser una línea:
if (usuario.getPerfil().esAdmin()) { ... }
```

La tercera opción es la que distingue a un perfil senior: **la mayoría de las comprobaciones de `null` son el síntoma de un diseño que permite estados imposibles**. Un objeto que no puede construirse en estado inválido no necesita que lo comprueben en cada uso.

**Una trampa concreta que vale la pena verificar.** El siguiente código parece a prueba de balas y no lo es:

```java
Map<String, Boolean> flags = new HashMap<>();
flags.put("feature", null);       // clave presente, valor null

boolean activo = flags.containsKey("feature") ? flags.get("feature") : false;
```

Verificado en JDK 25, lanza:

```
Cannot invoke "java.lang.Boolean.booleanValue()" because the return value of "java.util.Map.get(Object)" is null
```

`containsKey` devuelve `true` porque la clave existe, así que se evalúa `flags.get("feature")`, que devuelve `null`, y el *unboxing* a `boolean` explota. Y lo que casi nadie sabe: **`getOrDefault` tiene exactamente el mismo problema**:

```java
boolean activo = flags.getOrDefault("feature", false);   // también lanza NPE
```

Porque `getOrDefault` devuelve el valor por defecto solo cuando **la clave está ausente**, no cuando el valor almacenado es `null`. Lo dice su Javadoc, y casi nadie lo lee. Verificado en JDK 25:

```
Cannot invoke "java.lang.Boolean.booleanValue()" because the return value of "java.util.Map.getOrDefault(Object, Object)" is null
```

La forma segura:

```java
boolean activo = Boolean.TRUE.equals(flags.get("feature"));   // false, sin excepción
```

## 18. Efectos colaterales dentro de una condición

Una condición debería **preguntar**, no **hacer**. Cuando hace algo, el orden de evaluación deja de ser un detalle interno y pasa a ser lógica de negocio, con dos consecuencias medibles.

**Primera: el cortocircuito puede saltarse el efecto.**

```java
if (validar() && registrarIntento()) { ... }
```

Si `validar()` devuelve `false`, `registrarIntento()` no se ejecuta nunca. Es un bug de auditoría: se pierden justo los intentos fallidos, que son los que interesaba registrar.

**Segunda: una cadena de `if / else if` reevalúa.** Verificado en JDK 25 con un contador global:

```java
contador = 0;
if (inc() == 99) { }
else if (inc() == 99) { }
else if (inc() == 99) { }
System.out.println("inc() se llamó " + contador + " veces");
```

Salida:

```
R3 cadena if/else if evaluo inc() 3 veces
```

Tres llamadas. Si `inc()` fuera una consulta a base de datos, son tres viajes a la red por una sola decisión.

El `switch` no tiene ese problema, porque **evalúa su selector exactamente una vez**. Verificado:

```java
contador = 0;
switch (inc()) {
    case 1 -> System.out.println("caso 1, contador=" + contador);
    default -> System.out.println("default, contador=" + contador);
}
// imprime: caso 1, contador=1
```

Esa es una razón real —no estética— para preferir un `switch` sobre una cadena de `if` cuando todas las ramas miran el mismo valor: garantiza una única evaluación. La otra razón es de rendimiento y está en la [sección 31](#31-tableswitch-y-lookupswitch).

---

# Parte III — El operador ternario

## 19. Qué es el ternario y en qué se diferencia del if

El operador condicional `? :` es el único operador de Java que toma **tres** operandos, de ahí el nombre coloquial de *ternario*. Su forma es:

```java
variable = (condicion) ? valorSiVerdadero : valorSiFalso;
```

Se lee "si la condición es verdadera, tomá el primer valor; si no, el segundo". Comparado con el `if` equivalente:

```java
// Con if
int hora = 20;
String saludo;
if (hora < 18) {
    saludo = "Buen día.";
} else {
    saludo = "Buenas tardes.";
}

// Con ternario
int hora = 20;
String saludo = (hora < 18) ? "Buen día." : "Buenas tardes.";
```

Y como es una expresión, se puede usar directamente donde se espera un valor, sin variable intermedia:

```java
System.out.println((hora < 18) ? "Buen día." : "Buenas tardes.");
lista.add(n % 2 == 0 ? "par" : "impar");
enviarCorreo(destinatario, esUrgente ? Prioridad.ALTA : Prioridad.NORMAL);
```

**La diferencia con el `if` no es de longitud, es de categoría.** El `if` es una sentencia: ejecuta cosas. El ternario es una expresión: **produce un valor y tiene un tipo**. Esa es la única razón por la que el ternario existe. En un lenguaje donde el `if` fuera una expresión —Kotlin, Scala, Rust— el operador ternario sería redundante y de hecho no existe.

De esa diferencia salen tres consecuencias que se ven en la práctica:

1. **Un ternario permite declarar la variable como `final`.** Con `if`, hay que declarar primero y asignar después, así que la variable no puede ser `final` de forma directa. Con ternario, sí:

   ```java
   final String saludo = (hora < 18) ? "Buen día." : "Buenas tardes.";
   ```

2. **Las dos ramas de un ternario tienen que producir un valor.** No podés poner `System.out.println(...)` en una rama y un valor en la otra: no compila. El `if` no tiene esa restricción, y por eso permite escribir ramas asimétricas donde una hace una cosa y otra hace otra completamente distinta.

3. **El ternario obliga a que ambas ramas tengan un tipo común.** Y aquí es donde empieza el problema serio de las dos secciones siguientes.

## 20. El ternario tiene tipo y ese tipo puede lanzar una excepción

Esta es la sección más importante de la Parte III y una de las más importantes del capítulo, porque produce un `NullPointerException` en una línea donde no se ve ninguna llamada a método.

El caso mínimo, verificado en JDK 25:

```java
Integer i = null;
Integer resultado = true ? i : 0;    // NullPointerException
```

Mensaje real:

```
Cannot invoke "java.lang.Integer.intValue()" because "<local1>" is null
```

Lo desconcertante es que la variable de destino es un `Integer`, que admite `null`; el valor asignado es `i`, que vale `null`; y aun así explota. Cambiando **solo** el segundo operando, deja de fallar:

```java
Integer i = null;
Integer cero = 0;
Integer resultado = true ? i : cero;   // null, sin excepción
```

Verificado:

```
A2 NPE (unboxing de null): Cannot invoke "java.lang.Integer.intValue()" because "<local1>" is null
A3 sin NPE porque ambos son Integer: null
```

**Qué está pasando.** El ternario, como toda expresión, tiene que tener **un** tipo, decidido en compilación. El [JLS §15.25](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.25) especifica cómo se calcula, y la regla que muerde es esta: si uno de los operandos es de tipo primitivo `T` y el otro es el wrapper de `T`, **el tipo de la expresión es el primitivo `T`**.

En `true ? i : 0` los operandos son `Integer` e `int`. Aplicando la regla, el tipo de toda la expresión es `int`. Para producir un `int`, el compilador tiene que desempaquetar `i` llamando a `i.intValue()`. Como `i` es `null`, ahí muere. El resultado ni siquiera llega a reempaquetarse para asignarse al `Integer` de destino.

En `true ? i : cero` ambos operandos son `Integer`, no hay ninguna conversión que hacer, y `null` viaja intacto.

**El caso realista.** Nadie escribe `true ? i : 0`. Lo que sí se escribe todos los días es esto:

```java
// El bug que llega a producción
Integer cantidad = pedido.getCantidad();       // puede devolver null
int total = (cantidad != null) ? cantidad : 0; // esto está BIEN

// Y esta variante, que parece igual y está MAL
Map<String, Integer> stock = obtenerStock();
Integer unidades = stock.containsKey(sku) ? stock.get(sku) : 0;
```

La segunda falla si el mapa contiene la clave con valor `null`, exactamente como vimos en la [sección 17](#17-condiciones-sobre-null-y-el-orden-de-las-guardas). El `0` fuerza el tipo a `int`, y `stock.get(sku)` se desempaqueta.

Este es uno de los hilos más repetidos de Stack Overflow —hay al menos cinco preguntas distintas con miles de votos sobre el mismo asunto— y la conclusión de todos es la misma que da el JLS: **no es un bug, es el comportamiento especificado**. El grupo de expertos de JSR 201, que introdujo el *autoboxing* en Java 5, conocía el problema y decidió no cambiarlo.

**Las defensas, en orden de preferencia:**

```java
// 1. Que ambos operandos sean del mismo tipo de referencia
Integer unidades = stock.containsKey(sku) ? stock.get(sku) : Integer.valueOf(0);

// 2. Usar el método de la API pensado para esto
int unidades = stock.getOrDefault(sku, 0);   // ojo: solo si el mapa no guarda nulls

// 3. Optional, que hace explícita la ausencia
int unidades = Optional.ofNullable(stock.get(sku)).orElse(0);

// 4. Objects.requireNonNullElse (Java 9+), la más legible de todas
int unidades = Objects.requireNonNullElse(stock.get(sku), 0);
```

La cuarta es la que conviene aprender: `Objects.requireNonNullElse(valor, porDefecto)` dice exactamente lo que hace y no tiene ninguna sorpresa de tipos.

> **Regla operativa.** Cuando escribas un ternario cuyas ramas mezclen un wrapper con un primitivo, parate un segundo. Si el wrapper puede ser `null`, tenés un `NullPointerException` esperando. Los IDE modernos avisan de esto: IntelliJ lo marca como *"Unboxing of ... may produce NullPointerException"*. Ese aviso no es ruido, es el bug.

## 21. La promoción numérica del ternario

La misma regla del JLS §15.25 produce un segundo efecto, menos peligroso pero más desconcertante: **el ternario puede cambiar el tipo del valor que devuelve**.

Verificado en JDK 25:

```java
Object o = true ? Integer.valueOf(1) : Double.valueOf(2.0);
System.out.println(o + " / " + o.getClass().getName());
```

Salida real:

```
B1 valor=1.0 clase=java.lang.Double
```

La condición es `true`, así que uno esperaría el `Integer` con valor `1`. Lo que sale es un `Double` con valor `1.0`. La rama que se eligió fue la correcta, pero **el tipo de la expresión completa se decidió en compilación mirando las dos ramas**: `Integer` y `Double` se desempaquetan a `int` y `double`, se aplica promoción numérica binaria, el tipo común es `double`, y el `1` se convierte en `1.0` antes de reempaquetarse en un `Double`.

Si eso ya sorprende, el siguiente par es peor, porque **el mismo código imprime cosas distintas según si el otro operando es una constante o una variable**:

```java
char c = 'A';
int n = 66;

var x = true ? c : 66;   // constante literal
var y = true ? c : n;    // variable
```

Salida real:

```
B2 (char, constante 66) -> A clase=java.lang.Character
B3 (char, int variable) -> 65 clase=java.lang.Integer
```

En el primer caso el tipo es `char` y se imprime la letra `A`. En el segundo el tipo es `int` y se imprime `65`, el código numérico de `A`. La única diferencia entre las dos líneas es que `66` es una constante en tiempo de compilación cuyo valor **cabe** en un `char`, y `n` es una variable cuyo valor el compilador no conoce.

La regla del JLS que lo explica: *"if one of the operands is of type T where T is byte, short, or char, and the other operand is a constant expression of type int whose value is representable in type T, then the type of the conditional expression is T"*. Con una variable no se puede aplicar, así que cae en el caso general: promoción numérica binaria hasta `int`.

Nadie necesita memorizar la tabla completa. Lo que hay que llevarse es la conclusión operativa:

> **Cuando las dos ramas de un ternario tengan tipos distintos, hacé la conversión explícita.** Si querés un `double`, escribí `1.0` en la otra rama. Si querés un `Object`, casteá. Dejar que el compilador elija por vos es correcto según la especificación y sorprendente según cualquier lector.

## 22. Ternarios anidados

Un ternario puede contener otro, porque su resultado es una expresión:

```java
int hora = 22;
String mensaje = (hora < 12) ? "Buenos días."
               : (hora < 18) ? "Buenas tardes."
               : "Buenas noches.";
```

Formateado así —una rama por línea, alineadas— se lee razonablemente y equivale a una cadena de `if / else if`. La asociatividad del operador es **por la derecha**, así que `a ? b : c ? d : e` se agrupa como `a ? b : (c ? d : e)`, que es justo lo que uno quiere.

El problema aparece cuando el anidamiento se mete en la rama del medio:

```java
// Ilegible: el anidamiento está en la rama del "entonces"
String r = a ? (b ? "x" : "y") : "z";
```

o cuando hay más de dos niveles, o cuando las condiciones son largas. En ese punto la cadena de `if / else if` gana en claridad, y el `switch` moderno gana todavía más.

La recomendación de la fuente es sensata: *"use the ternary operator for short, simple choices. For longer or more complex logic, the regular if...else is easier to read."*

Mi versión operativa, algo más concreta:

- **Un nivel de ternario:** bien, siempre.
- **Dos niveles, formateados en columna, condiciones cortas:** aceptable, sobre todo para mapear rangos.
- **Tres niveles o más, o anidamiento en la rama verdadera:** refactorizá. Casi siempre es un `switch` con `when`, un mapa, o un método con cláusulas de guarda.

## 23. Cuándo usar el ternario y cuándo no

El ternario brilla en un caso concreto: **elegir entre dos valores del mismo tipo para asignarlos o pasarlos**. Fuera de ahí, es peor que un `if`.

**Buenos usos:**

```java
// Elegir un valor
int max = (a > b) ? a : b;

// Valor por defecto
String nombre = (entrada != null) ? entrada : "anónimo";

// Dentro de una llamada, evitando una variable temporal
log.info("Procesados {} elemento{}", n, n == 1 ? "" : "s");

// Dentro de un stream, donde una sentencia no cabe
lista.stream().map(x -> x > 0 ? "positivo" : "no positivo").toList();
```

Ese último caso merece énfasis: dentro de una lambda de una sola expresión, **el `if` no cabe** porque es una sentencia. Ahí el ternario no es una preferencia de estilo, es la única opción sin añadir llaves y un `return`.

**Malos usos:**

```java
// 1. Cuando las ramas hacen cosas en vez de producir valores
boolean ignorado = condicion ? hacerA() : hacerB();   // usar if

// 2. Cuando las ramas tienen tipos distintos (sección 21)
Object o = flag ? 1 : 2.0;                            // devuelve Double siempre

// 3. Cuando mezcla wrapper y primitivo y el wrapper puede ser null (sección 20)
int n = flag ? mapa.get(k) : 0;                       // NullPointerException latente

// 4. Cuando la condición o las ramas son largas
String s = usuario != null && usuario.getPerfil() != null && usuario.getPerfil().esAdmin()
        ? construirPanelAdministrador(usuario, configuracion, permisos)
        : construirPanelBasico(usuario);               // usar if, o extraer métodos
```

**Comparación resumida:**

| | `if / else` | Ternario |
|---|---|---|
| Categoría | Sentencia | Expresión |
| Produce valor | No | Sí |
| Permite `final` en la asignación | No directamente | Sí |
| Ramas pueden ser asimétricas | Sí | No |
| Sirve dentro de una lambda de una expresión | No | Sí |
| Riesgo de conversión de tipo inesperada | No | **Sí** |
| Legible con más de dos casos | Sí | Regular |

---

# Parte IV — El switch clásico

## 24. Qué es un switch y por qué no es una cadena de if

El `switch` compara un valor —el **selector**— contra una lista de casos, y ejecuta el código asociado al que coincida:

```java
switch (expresion) {
    case x:
        // código
        break;
    case y:
        // código
        break;
    default:
        // código
}
```

Un ejemplo:

```java
int dia = 4;
switch (dia) {
    case 1: System.out.println("Lunes"); break;
    case 2: System.out.println("Martes"); break;
    case 3: System.out.println("Miércoles"); break;
    case 4: System.out.println("Jueves"); break;
    case 5: System.out.println("Viernes"); break;
    case 6: System.out.println("Sábado"); break;
    case 7: System.out.println("Domingo"); break;
}
// Imprime: Jueves
```

Casi todos los tutoriales presentan el `switch` como "una alternativa más limpia a una cadena de `if / else if`". Esa descripción es cómoda y **oculta la diferencia que explica todo el comportamiento raro del `switch`**, incluido el `break`.

La formulación correcta la da un artículo de Java Magazine de Oracle: el `if / else` es una **bifurcación**, mientras que el `switch` clásico es un **salto**. El `switch` no elige entre bloques alternativos; calcula una dirección y **salta a una etiqueta dentro de un único bloque**, y desde ahí sigue ejecutando hacia abajo. Es, en sus palabras, *"a forward-only goto mechanism"*.

Todo lo demás se deduce de ahí:

- Por qué hace falta `break`: porque no hay nada que detenga la ejecución al llegar al siguiente `case`. Las etiquetas son destinos de salto, no fronteras.
- Por qué las variables declaradas en un `case` son visibles en los demás: porque todo es un solo bloque, con un solo *scope*.
- Por qué el selector se evalúa una sola vez: porque se usa para calcular a dónde saltar.
- Por qué es rápido: porque un salto calculado es O(1) frente a los O(n) de una cadena de comparaciones.

Las cuatro diferencias frente a una cadena de `if`:

| | Cadena de `if / else if` | `switch` |
|---|---|---|
| Qué compara | Cualquier condición booleana | Un valor contra constantes o patrones |
| Cuántas veces evalúa el sujeto | Una por rama | **Una sola** |
| Coste | O(n) comparaciones | O(1) con salto calculado |
| Puede comparar rangos | Sí (`x > 10 && x < 20`) | No con constantes; sí con guardas desde Java 21 |

## 25. Sobre qué tipos se puede hacer switch

Históricamente el `switch` fue muy restrictivo, y esas restricciones se han ido levantando versión a versión. El estado actual:

| Tipo del selector | Desde | Nota |
|---|---|---|
| `byte`, `short`, `char`, `int` | Java 1.0 | los enteros que caben en un `int` |
| `Byte`, `Short`, `Character`, `Integer` | Java 5 | con *unboxing*, y por tanto con riesgo de NPE |
| `enum` | Java 5 | el caso de uso ideal del `switch` |
| `String` | Java 7 | usa `equals` por dentro |
| Cualquier tipo de referencia | Java 21 | solo con etiquetas de patrón |
| `long`, `float`, `double`, `boolean` | Java 23+ (preview) | ver la [sección 45](#45-patrones-sobre-primitivos) |

Las dos ausencias que llaman la atención en la lista clásica son **`long` y `boolean`**, y las razones son distintas. `long` quedó fuera porque el bytecode del `switch` usa índices de 32 bits; `boolean` porque con dos valores un `if` ya basta. Ambas se levantan con la función preview de la sección 45.

**Las constantes de los `case` tienen sus propias reglas:**

```java
switch (x) {
    case 1:             // literal: OK
    case 2 + 3:         // expresión constante: OK, vale 5
    case MAX:           // static final int MAX = 10: OK
    case obtenerN():    // NO COMPILA: no es constante en compilación
}
```

Una etiqueta `case` tiene que ser una **expresión constante en tiempo de compilación**, porque el compilador necesita conocer todos los valores para construir la tabla de saltos. Y no puede haber dos iguales:

```java
switch (x) {
    case 1: break;
    case 1: break;   // error
}
```

```
E3.java:5: error: duplicate case label
            case 1: break;
            ^
```

Ese es un chequeo valioso que una cadena de `if / else if` no tiene: repetir una condición en una cadena de `if` compila sin quejarse y deja código muerto.

## 26. break y el fallthrough

Cuando la ejecución entra por un `case`, **sigue hacia abajo hasta encontrar un `break`, un `return`, un `throw` o el final del `switch`**, atravesando las etiquetas siguientes como si no existieran. Eso se llama *fallthrough*, y no es un bug del lenguaje: es la consecuencia directa de que el `switch` sea un salto.

Verificado en JDK 25:

```java
int dia = 2;
StringBuilder sb = new StringBuilder();
switch (dia) {
    case 1:
        sb.append("uno ");
    case 2:
        sb.append("dos ");
    case 3:
        sb.append("tres ");
        break;
    case 4:
        sb.append("cuatro ");
}
System.out.println(sb);
```

Salida real:

```
D1 day=2 produce: dos tres
```

Con `dia = 2` se ejecutan **dos** ramas: la del `2` y la del `3`, porque la primera no tiene `break`. La del `1` no se ejecuta porque el salto entró después.

El daño real de esto no es académico. El caso canónico es el descuento acumulado:

```java
// BUG: un cliente PREMIUM recibe los tres descuentos
switch (nivel) {
    case PREMIUM:
        descuento += 0.10;
    case ORO:
        descuento += 0.05;
    case PLATA:
        descuento += 0.02;
}
```

Un cliente `PREMIUM` sale con un 17 % de descuento en vez de un 10 %. El código compila, pasa una revisión rápida y solo se detecta cuando cuadra el departamento financiero.

**El compilador puede avisar, pero no lo hace por defecto.** Verificado:

```
$ javac -Xlint:all T1.java
T1.java:81: warning: [fallthrough] possible fall-through into case
            case 2:
            ^
T1.java:83: warning: [fallthrough] possible fall-through into case
            case 3:
            ^
2 warnings
```

Fijate que el aviso solo aparece para los casos con código encima; el `case 4:` que sigue a un `break` no genera aviso. Herramientas como [Error Prone](https://errorprone.info/bugpattern/FallThrough) de Google elevan esto a error de compilación, y la [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html#s4.8.4.2-switch-fall-through) exige que todo grupo de sentencias termine abruptamente o lleve un comentario explícito indicando que la caída es intencionada.

**La solución definitiva llegó en Java 14**: la sintaxis de flecha, que no cae nunca. Está en la [sección 32](#32-la-flecha), y es la razón por la que hoy el `fallthrough` debería ser una rareza en código nuevo.

## 27. Cuándo el fallthrough es la respuesta correcta

Hay un uso legítimo, y conviene conocerlo para reconocerlo en código ajeno: **agrupar varios casos que comparten el mismo tratamiento**, dejando etiquetas vacías consecutivas.

```java
switch (mes) {
    case 12:
    case 1:
    case 2:
        estacion = "Verano";
        break;
    case 3:
    case 4:
    case 5:
        estacion = "Otoño";
        break;
    default:
        estacion = "Otra";
}
```

Aquí la caída de `case 12:` a `case 1:` es deliberada y no tiene ningún riesgo, porque entre las etiquetas no hay código. El compilador no emite aviso de `fallthrough` en este caso precisamente porque distingue entre "caída sobre un grupo vacío" y "caída con código en medio".

Aun así, **desde Java 14 esto se escribe mejor con la lista de constantes separadas por comas**, que era una de las mejoras del JEP 361:

```java
switch (mes) {
    case 12, 1, 2 -> estacion = "Verano";
    case 3, 4, 5  -> estacion = "Otoño";
    default       -> estacion = "Otra";
}
```

Con eso, el único uso legítimo que le quedaba al `fallthrough` desaparece. Lo que queda es el *fallthrough con código*, que sigue teniendo un nicho muy estrecho —máquinas de estado, parsers de formatos binarios, código que procesa niveles acumulativos— y que conviene marcar siempre con un comentario:

```java
switch (token) {
    case ESCAPE:
        procesarEscape();
        // fall through — un escape también consume el carácter siguiente
    case NORMAL:
        consumirCaracter();
        break;
}
```

## 28. El default

`default` es la etiqueta que se elige cuando ninguna otra coincide. Es el equivalente al `else` final de una cadena de `if`:

```java
int dia = 4;
switch (dia) {
    case 6:
        System.out.println("Hoy es sábado");
        break;
    case 7:
        System.out.println("Hoy es domingo");
        break;
    default:
        System.out.println("Esperando el fin de semana");
}
// Imprime: Esperando el fin de semana
```

Cuatro cosas que conviene saber y que los tutoriales suelen omitir:

**1. No hace falta `break` si `default` va al final.** Porque no hay nada más abajo a lo que caer. Es una excepción de conveniencia, no una regla distinta.

**2. `default` no tiene que ir al final.** Es legal ponerlo en medio, y entonces sí necesita `break`. No lo hagas: se lee fatal y no aporta nada.

**3. Omitir `default` en una sentencia `switch` es legal y silencioso.** Si ningún caso coincide, no pasa nada en absoluto. Es el mismo problema del `else` ausente de la [sección 7](#7-el-else-if-no-existe): un valor inesperado se procesa como si fuera correcto, sin dejar rastro. Para código de producción, la costumbre sana es que el `default` **falle ruidosamente**:

```java
switch (estado) {
    case NUEVO -> procesarNuevo();
    case PAGADO -> procesarPagado();
    default -> throw new IllegalStateException("Estado no soportado: " + estado);
}
```

**4. `default` NO captura `null`.** Esta es la que sorprende. Un `switch` sobre una referencia nula lanza `NullPointerException` **antes** de mirar las etiquetas, así que tener un `default` no protege de nada. Lo trata en detalle la [sección 38](#38-el-switch-y-null).

Y hay un quinto punto, contraintuitivo, que solo aplica al `switch` moderno sobre `enum`: **a veces conviene NO poner `default`**. Está en la [sección 36](#36-el-default-implícito-de-los-enums-y-matchexception).

## 29. El scope compartido del switch clásico

Como el `switch` clásico es un único bloque con etiquetas dentro, **todas las variables declaradas en cualquier `case` comparten el mismo *scope***:

```java
switch (dia) {
    case LUNES:
    case MARTES:
        int temp = calcular();
        break;
    case MIERCOLES:
        int temp2 = otro();     // no puedo llamarla 'temp': ya existe
        break;
    default:
        int temp3 = tercero();  // tampoco
}
```

Esto obliga a inventar nombres (`temp`, `temp2`, `temp3`) que no significan nada, y crea una situación peor: una variable declarada en el `case LUNES` **existe** en el `case MIERCOLES`, pero **no está inicializada** si el salto entró por ahí. El compilador lo detecta con su análisis de asignación definitiva y da error si se usa, pero el diseño invita a la confusión.

Es una de las tres irregularidades que el JEP 361 citó explícitamente como motivación para rediseñar el `switch`: *"the default scoping in switch blocks (the whole block is treated as one scope)"*.

La sintaxis de flecha lo arregla de raíz, porque cada rama es un bloque propio:

```java
switch (dia) {
    case LUNES, MARTES -> {
        int temp = calcular();      // scope propio
        usar(temp);
    }
    case MIERCOLES -> {
        int temp = otro();          // mismo nombre, sin conflicto
        usar(temp);
    }
}
```

## 30. Qué hace por dentro un switch sobre String

El `switch` sobre `String` llegó en Java 7, y entender cómo funciona resuelve de golpe tres dudas frecuentes.

El compilador lo traduce a **dos** `switch` encadenados. Verificado con `javap -c` sobre este código:

```java
static int texto(String s) {
    switch (s) {
        case "alpha": return 1;
        case "beta": return 2;
        default: return 0;
    }
}
```

Bytecode real (extracto):

```
5: invokevirtual  #7   // Method java/lang/String.hashCode:()I
8: lookupswitch  { // 2
39: invokevirtual #15  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
53: invokevirtual #15  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
62: lookupswitch  { // 2
```

Lo que hace, paso a paso:

1. Llama a `s.hashCode()`.
2. Un `lookupswitch` sobre ese entero decide a qué grupo de candidatos ir.
3. Dentro del grupo, llama a `s.equals("alpha")` o `s.equals("beta")` para confirmar.
4. Un segundo `lookupswitch`, ahora sobre un índice interno, salta al código real del caso.

De ahí salen las tres respuestas:

**Por qué un `switch` sobre `String` funciona con cadenas creadas en ejecución.** Porque el paso 3 usa `equals`, no `==`. Es la razón por la que el ejemplo de la [sección 15](#15-comparar-cadenas-de-texto) casaba correctamente con un `String` construido concatenando.

**Por qué las colisiones de `hashCode` no rompen nada.** Dos cadenas distintas pueden tener el mismo `hashCode`; el par clásico es `"Aa"` y `"BB"`, ambos con hash `2112`. Si el `switch` se fiara solo del hash, elegiría mal. Verificado en JDK 25:

```
hashCode("Aa")=2112 hashCode("BB")=2112
E1 Aa -> rama Aa
E1 BB -> rama BB
```

Correcto en ambos casos, porque el `equals` del paso 3 desempata. El hash solo sirve para reducir candidatos rápido.

**Por qué lanza `NullPointerException` con `null`.** Porque el paso 1 llama a `s.hashCode()` sobre la referencia. Verificado, el mensaje lo dice literalmente:

```
Cannot invoke "String.hashCode()" because "<local1>" is null
```

Ese mensaje es una ventana directa a la implementación: revela que hay una llamada a `hashCode` que nadie escribió.

**Rendimiento.** Es rápido, y no por magia: `String` cachea su `hashCode` tras el primer cálculo, así que el paso 1 es una lectura de campo salvo la primera vez. Un `switch` de veinte cadenas hace una lectura de campo, un salto y un `equals`, frente a los veinte `equals` de una cadena de `if`.

## 31. tableswitch y lookupswitch

La JVM tiene **dos** instrucciones distintas para el `switch`, y cuál elige el compilador determina si tu `switch` es O(1) o O(log n).

- **`tableswitch`** — una tabla de saltos indexada. Guarda un array de direcciones y salta directamente a la posición `valor - minimo`. Es **O(1)**: una resta, una comprobación de rango y un salto. Requiere que los valores sean **densos y contiguos**, porque los huecos ocupan espacio en la tabla.
- **`lookupswitch`** — una tabla de pares `(clave, dirección)` ordenada por clave, recorrida con búsqueda binaria. Es **O(log n)**. Se usa cuando los valores están dispersos.

El compilador elige comparando el coste en espacio y tiempo de las dos opciones. Verificado en JDK 25 con `javap -c`:

```java
// Casos densos: 1, 2, 3, 4
static int denso(int x) { switch (x) { case 1: ... case 4: ... } }
```

```
1: tableswitch   { // 1 to 4
        default: 44
```

```java
// Casos dispersos: 1, 1000, 1000000
static int disperso(int x) { switch (x) { case 1: ... case 1000000: ... } }
```

```
1: lookupswitch  { // 3
        default: 45
```

Y la cadena de `if / else if` equivalente al caso denso compila a **cuatro instrucciones de salto condicional** (`if_icmp`), que se ejecutan una tras otra. Ahí está la diferencia de complejidad, en el bytecode, sin necesidad de medir nada.

**Consecuencias prácticas, y sus límites:**

1. Un `switch` sobre un `enum` es siempre `tableswitch`, porque el compilador usa el `ordinal()` de cada constante, que es denso por construcción. Los `switch` sobre `enum` son la forma más rápida de despachar que tiene Java.
2. Un `switch` con casos muy dispersos degrada a `lookupswitch`, pero O(log n) sigue siendo mucho mejor que O(n).
3. **Nada de esto justifica reescribir código.** Para menos de cinco ramas la diferencia es irrelevante y el JIT puede optimizar ambas formas. El argumento real para preferir `switch` sobre una cadena de `if` sigue siendo la legibilidad, la evaluación única del selector y —sobre todo— la comprobación de exhaustividad de la Parte V.

> **Sobre las cifras.** En este capítulo no hay medidas de tiempo. El análisis de bytecode es determinista y se puede reproducir con `javap -c`; un microbenchmark casero sobre construcciones que el JIT reescribe agresivamente daría números que dicen más del banco de pruebas que del lenguaje. Para medir de verdad hace falta JMH, y aun así la conclusión no cambiaría ninguna decisión de diseño.

---

# Parte V — El switch moderno

Entre Java 14 y Java 21, el `switch` dejó de ser la construcción heredada de C que acabamos de ver y se convirtió en la herramienta más potente del lenguaje para expresar decisiones. Los cambios llegaron en dos JEP: el [JEP 361](https://openjdk.org/jeps/361) (Java 14) trajo la flecha y el `switch` como expresión, y el [JEP 441](https://openjdk.org/jeps/441) (Java 21) trajo los patrones.

Casi ningún tutorial de nivel introductorio cubre esto. Las dos fuentes de referencia de este capítulo describen el `switch` con `case x: ... break;` como si fuera la única forma que existe, y una de ellas incluso llama "switch expression" a lo que es una sentencia. En 2026, escribir un `switch` con `break` en código nuevo es como escribir un bucle con `Iterator` explícito: funciona, pero delata.

## 32. La flecha

La primera mejora del JEP 361 es una forma nueva de etiqueta: `case L ->`. Si la etiqueta coincide, **se ejecuta solo lo que está a la derecha de la flecha, y el `switch` termina**. No hay caída.

Antes:

```java
switch (dia) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}
```

Después:

```java
switch (dia) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

Trece líneas contra cuatro, y de propina **desaparecen tres clases de bug**: el `break` olvidado, el `fallthrough` accidental y el scope compartido.

Tres detalles de sintaxis:

**A la derecha de la flecha solo puede ir una expresión, un bloque, o un `throw`.** Nada más. Si necesitás varias sentencias, van entre llaves, y esas llaves crean un scope propio:

```java
case MONDAY -> {
    int calculado = calcular();
    registrar(calculado);
    notificar(calculado);
}
```

**Se pueden listar varias constantes separadas por comas.** Eso elimina el único uso legítimo del `fallthrough` que quedaba, como vimos en la [sección 27](#27-cuándo-el-fallthrough-es-la-respuesta-correcta).

**No se pueden mezclar los dos estilos en el mismo `switch`.** Verificado en JDK 25:

```java
switch (x) {
    case 1 -> System.out.println("uno");
    case 2: System.out.println("dos"); break;   // error
}
```

```
E8.java:5: error: different case kinds used in the switch
            case 2: System.out.println("dos"); break;
            ^
```

Es una restricción deliberada y sensata: un `switch` mitad flecha y mitad dos puntos sería ilegible respecto de dónde cae y dónde no.

> **Nota sobre la flecha.** El `->` es visualmente el mismo token que el de las lambdas, pero **no es una lambda**. No crea un objeto, no captura variables, no difiere la ejecución. Es solo un separador de sintaxis. La confusión es común y merece decirse en voz alta.

## 33. El switch como expresión

La segunda mejora del JEP 361 es más profunda: el `switch` puede **producir un valor**, y por tanto usarse donde se espera una expresión.

El problema que resuelve. Este patrón aparecía en todas partes:

```java
int numLetras;
switch (dia) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetras = 6;
        break;
    case TUESDAY:
        numLetras = 7;
        break;
    // ...
    default:
        throw new IllegalStateException("Wat: " + dia);
}
```

Tiene tres defectos que el JEP nombra explícitamente: es repetitivo, hay que declarar la variable sin inicializar (así que no puede ser `final`), y **nada garantiza que todas las ramas le asignen algo**. Si una rama se olvida, la variable queda sin asignar y el error aparece lejos de su causa.

La versión como expresión:

```java
int numLetras = switch (dia) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```

Fijate en el **punto y coma final después de la llave de cierre**: es la marca de que esto es una expresión asignada a una variable, no un bloque. Olvidarlo es el error de sintaxis más común al empezar.

Lo que se gana:

1. La variable se declara y asigna en una sola sentencia, y puede ser `final`.
2. El compilador **exige que todas las ramas produzcan un valor** (sección 35).
3. Se puede usar directamente como argumento, como valor de retorno o dentro de otra expresión:

   ```java
   return switch (estado) {
       case NUEVO -> 48;
       case PAGADO -> 24;
       case ENVIADO -> 8;
   };
   ```

Un ejemplo completo, verificado en JDK 25:

```java
enum Status { NEW, PAID, SHIPPED }

static int slaHours(Status st) {
    return switch (st) {
        case NEW -> 48;
        case PAID -> 24;
        case SHIPPED -> 8;
    };
}
```

Salida:

```
M1 NEW -> 48h
M1 PAID -> 24h
M1 SHIPPED -> 8h
```

Sin `default`, sin `break`, sin variable temporal. Y compila **precisamente porque** cubre todas las constantes del `enum`; si faltara una, sería error de compilación.

**Una restricción que sorprende:** dentro de un `switch` como expresión **no se puede usar `return`, `break` ni `continue` para salir hacia fuera**. Una expresión tiene que producir un valor o lanzar una excepción; no puede abandonar el método a mitad. El JEP lo ilustra así:

```java
z: for (int i = 0; i < MAX; ++i) {
    int k = switch (e) {
        case 0: yield 1;
        default: continue z;   // ERROR: salto ilegal a través de una expresión switch
    };
}
```

## 34. yield

Cuando una rama de un `switch` como expresión necesita más de una sentencia, se usa un bloque. Pero un bloque no produce valor por sí solo, así que hace falta una palabra para decir "el valor de esta rama es este": **`yield`**.

```java
int j = switch (dia) {
    case MONDAY  -> 0;
    case TUESDAY -> 1;
    default      -> {
        int k = dia.toString().length();
        int resultado = f(k);
        yield resultado;
    }
};
```

Verificado en JDK 25:

```java
static String yieldDemo(int n) {
    return switch (n) {
        case 1 -> "uno";
        default -> {
            String base = "n=" + n;
            yield base.toUpperCase(Locale.ROOT);
        }
    };
}
```

Salida: `N1 uno / N=7`

Tres cosas sobre `yield`:

**Solo hace falta con bloques.** Si la rama es una única expresión, el valor es esa expresión y no se escribe `yield`.

**También funciona con la sintaxis de dos puntos.** Esto es lo que hace posible el "tipo 2" de la taxonomía de la sección 37:

```java
int resultado = switch (s) {
    case "Foo": yield 1;
    case "Bar": yield 2;
    default:
        System.out.println("Ni Foo ni Bar");
        yield 0;
};
```

**`yield` no es una palabra clave.** Es un *identificador restringido*, como `var`. Eso significa que un método llamado `yield` sigue siendo legal y el código antiguo no se rompe. La ambigüedad de `yield(x)` —¿llamada a método o sentencia `yield` con paréntesis?— se resuelve a favor de la sentencia; para llamar al método hay que cualificarlo con `this.yield(x)` o `Clase.yield(x)`.

**Por qué `yield` y no `break`.** Historia útil: en la primera preview (JEP 325, Java 12) se usaba `break valor;`. La respuesta de la comunidad fue que resultaba confuso, porque `break` ya significaba otra cosa. El JEP 354 lo cambió a `yield` en Java 13 y así quedó. La ventaja del par final es que desambigua: **un `switch` sentencia puede ser destino de un `break`, y un `switch` expresión puede ser destino de un `yield`**, nunca al revés.

## 35. Exhaustividad

Esta es la razón por la que el `switch` moderno importa, y no es una cuestión de sintaxis: es una **red de seguridad del compilador**.

Un `switch` como **expresión** tiene que ser exhaustivo: para todo valor posible del selector tiene que haber una etiqueta que lo cubra. Es una consecuencia lógica de ser una expresión: si el valor no casara con nada, la expresión no tendría valor, y eso es imposible.

Verificado en JDK 25:

```java
static int f(Object o) {
    return switch (o) {
        case String s -> s.length();
        case Integer i -> i;
    };
}
```

```
E1.java:3: error: the switch expression does not cover all possible input values
        return switch (o) {
               ^
```

Hay tres formas de conseguir exhaustividad:

**1. Un `default`.** Cubre todo lo demás, siempre funciona.

```java
return switch (o) {
    case String s -> s.length();
    case Integer i -> i;
    default -> 0;
};
```

**2. Cubrir todas las constantes de un `enum`.** Sin `default`:

```java
return switch (estado) {   // Status tiene exactamente NEW, PAID, SHIPPED
    case NEW -> 48;
    case PAID -> 24;
    case SHIPPED -> 8;
};
```

**3. Cubrir todos los subtipos permitidos de un tipo `sealed`.** Es el caso más potente y lo veremos en la [sección 44](#44-tipos-sellados-y-uniones-cerradas).

**La decisión de diseño que hay que entender.** Las opciones 2 y 3 no son solo "más cortas que poner `default`". Son **mejores**, y por una razón concreta: si mañana alguien añade una constante al `enum` o un subtipo al `sealed`, la versión sin `default` **deja de compilar** y te obliga a decidir qué hacer con el caso nuevo. La versión con `default` compila igual y se traga el caso nuevo en silencio.

```java
// Con default: añadir Status.CANCELLED compila y devuelve 0. Bug silencioso.
return switch (estado) {
    case NEW -> 48;
    case PAID -> 24;
    default -> 0;
};

// Sin default: añadir Status.CANCELLED es un error de compilación. El compilador te avisa.
return switch (estado) {
    case NEW -> 48;
    case PAID -> 24;
    case SHIPPED -> 8;
};
```

> **Regla de diseño.** Sobre `enum` y tipos `sealed`, **evitá el `default`**. Es contraintuitivo —parece menos defensivo— y es exactamente al revés: renunciar al `default` convierte al compilador en tu revisor de código para todos los cambios futuros del dominio. Es una de las prácticas que más separa a un perfil mid de uno senior, y la recomiendan tanto el JEP 361 como Nicolai Parlog en su guía sobre `switch`.

**¿Y las sentencias `switch`?** Una sentencia `switch` clásica **no** necesita ser exhaustiva; puede no hacer nada. Pero desde Java 21, una sentencia `switch` que use **patrones** sí está obligada a ser exhaustiva. El JEP 441 lo dice como uno de sus objetivos: *"increase the safety of switch statements by requiring that pattern switch statements cover all possible input values"*. Stephen Colebourne criticó públicamente que no se aplicara la misma regla a los `switch` clásicos, argumentando que *"keeping a bad design from 20 years ago is a worse sin"* que romper la ortogonalidad. Tiene razón, y es útil saber por qué el lenguaje quedó así: compatibilidad hacia atrás.

## 36. El default implícito de los enums y MatchException

Aquí hay un detalle que casi nadie conoce y que produce una excepción cuyo nombre la mayoría de los desarrolladores Java no ha visto nunca.

Cuando escribís un `switch` como expresión sobre un `enum` cubriendo todas sus constantes, el compilador acepta el código. Pero **inserta un `default` invisible**, porque sabe algo que vos no controlás: el `enum` y el código que hace `switch` sobre él se compilan por separado y pueden desincronizarse. Alguien puede recompilar el `enum` con una constante nueva sin recompilar tu clase.

El experimento, ejecutado de verdad en JDK 25:

```java
// Paso 1: el enum tiene DOS constantes
public enum Status { NEW, PAID }

// Paso 2: una clase hace switch exhaustivo sobre él, sin default
public class Client {
    static int sla(Status s) {
        return switch (s) {
            case NEW  -> 48;
            case PAID -> 24;
        };
    }
}
```

Se compilan ambos y funciona:

```
NEW -> 48
PAID -> 24
```

**Paso 3:** el `enum` gana una constante y se recompila **solo el `enum`**:

```java
public enum Status { NEW, PAID, SHIPPED }
```

**Paso 4:** se ejecuta el `Client` viejo contra el `enum` nuevo, sin recompilarlo. Salida real:

```
NEW -> 48
PAID -> 24
SHIPPED -> LANZA java.lang.MatchException: null
```

Eso es el `default` implícito haciendo su trabajo. `MatchException` es una excepción introducida en Java 21 precisamente para este caso; en Java 14 a 20 lo que se lanzaba era un `IncompatibleClassChangeError`.

**Qué hay que llevarse de aquí:**

1. **Un `switch` exhaustivo sin `default` no es "inseguro en ejecución".** El compilador te cubre; simplemente falla ruidosamente en vez de en silencio, que es lo correcto.
2. **`MatchException` en producción significa una cosa muy concreta:** hay una desincronización de compilación. Un JAR se actualizó y otro no. La solución no es añadir un `default`: es recompilar.
3. **Esto refuerza el consejo de la sección 35.** Sin `default`, el problema se detecta en compilación durante un build limpio. Con `default`, el caso nuevo se procesa mal en silencio y nunca se detecta.

## 37. Las cuatro formas del switch

Las dos decisiones —flecha o dos puntos, sentencia o expresión— son **ortogonales**, así que existen cuatro combinaciones. Esta taxonomía se la debemos a Stephen Colebourne, que la usó para argumentar que dos de las cuatro sobran.

| | Sintaxis de dos puntos | Sintaxis de flecha |
|---|---|---|
| **Sentencia** | **Tipo 1** — el clásico. Cae por defecto. No exhaustivo. Admite `return`, `break`, `continue`. Un solo scope. | **Tipo 3** — no cae. No exhaustivo (salvo con patrones). Admite `return`, `break`, `continue`. Scope por rama. |
| **Expresión** | **Tipo 2** — cae por defecto. Exhaustivo. Requiere `yield`. Un solo scope. No admite `return`. | **Tipo 4** — no cae. Exhaustivo. `yield` solo en bloques. Scope por rama. No admite `return`. |

Ejemplos de cada uno:

```java
// Tipo 1: el clásico
switch (x) {
    case 1: hacerAlgo(); break;
    default: otraCosa();
}

// Tipo 2: expresión con dos puntos (rara, evitala)
int r = switch (x) {
    case 1: yield 10;
    default: yield 0;
};

// Tipo 3: sentencia con flecha
switch (x) {
    case 1 -> hacerAlgo();
    default -> otraCosa();
}

// Tipo 4: expresión con flecha (la que querés casi siempre)
int r = switch (x) {
    case 1 -> 10;
    default -> 0;
};
```

**La guía práctica**, que coincide con la de Nicolai Parlog y con la de la propia respuesta más votada de Stack Overflow sobre el tema:

- **Usá el tipo 4 siempre que el `switch` produzca un valor.** Es estrictamente superior: exhaustivo, sin caídas, con scope por rama.
- **Usá el tipo 3 cuando el `switch` ejecute acciones** y no haya un valor natural que devolver.
- **Usá el tipo 1 solo si necesitás `fallthrough` real con código**, que es raro.
- **No uses el tipo 2 nunca.** Existe por ortogonalidad, no por utilidad.

El resumen de una línea: **flecha siempre; expresión cuando haya un valor**.

## 38. El switch y null

Históricamente, un `switch` sobre una referencia nula lanza `NullPointerException` **antes de evaluar ninguna etiqueta**. Tener `default` no ayuda, porque el fallo ocurre antes.

Verificado en JDK 25:

```java
String s = null;
switch (s) {
    case "a" -> System.out.println("a");
    default -> System.out.println("default");
}
```

```
C1 NPE: default NO captura null. msg=Cannot invoke "String.hashCode()" because "<local1>" is null
```

Por eso, antes de Java 21, el idioma obligado era sacar la comprobación fuera:

```java
// Prior to Java 21
static void testFooBarOld(String s) {
    if (s == null) {
        System.out.println("Oops!");
        return;
    }
    switch (s) {
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
```

El JEP 441 consideró que, si el `switch` iba a aceptar cualquier tipo de referencia, esa comprobación aparte era *"an arbitrary distinction which invites needless boilerplate and opportunities for error"*. Así que introdujo la etiqueta **`case null`**:

```java
// As of Java 21
static void testFooBarNew(String s) {
    switch (s) {
        case null         -> System.out.println("Oops");
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
```

Verificado en JDK 25:

```
C2 case null si lo captura
```

**La regla completa**, que conviene memorizar porque es sutil:

> El comportamiento del `switch` ante un selector `null` lo determinan **siempre** sus etiquetas. Si hay `case null`, se ejecuta esa rama. Si no lo hay, se lanza `NullPointerException`, exactamente como antes. **El `default` no captura `null`.**

Esa última frase es una decisión deliberada de compatibilidad: si `default` capturara `null`, todo el código escrito antes de Java 21 cambiaría de comportamiento al recompilarse.

También existe la forma combinada `case null, default`, para cuando querés tratar el nulo igual que todo lo demás:

```java
Object obj = null;
switch (obj) {
    case null, default -> System.out.println("nulo o cualquier otra cosa");
}
```

Verificado:

```
C3 'case null, default' unifica ambos
```

Y no, `case null` no vale sobre un selector primitivo. Verificado:

```
E4.java:4: error: incompatible types: <null> cannot be converted to int
            case null -> {}
                 ^
```

---

# Parte VI — Pattern matching

Hasta aquí, todas las condicionales de este capítulo preguntan por **valores**: ¿es igual a 3?, ¿es mayor que 18?, ¿es la cadena "PAID"? El *pattern matching* añade una pregunta distinta: **¿tiene esta forma?** Y si la tiene, extrae las partes en el mismo gesto.

Es la línea de trabajo del proyecto Amber de OpenJDK, y llegó en etapas: `instanceof` con patrón en Java 16 ([JEP 394](https://openjdk.org/jeps/394)), patrones en `switch` y *record patterns* en Java 21 (JEPs [441](https://openjdk.org/jeps/441) y [440](https://openjdk.org/jeps/440)).

## 39. instanceof con patrón y flow scoping

El idioma que Java arrastró durante veinte años:

```java
// Antes de Java 16
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}
```

Tres apariciones del tipo `String` para una sola idea, y un cast que el compilador ya sabe que es seguro pero no puede evitarte escribir. Desde Java 16:

```java
// Java 16+
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

`String s` es un **patrón de tipo**. Si `obj` es un `String` en ejecución, el `instanceof` da `true` **y** la variable `s` queda inicializada con `obj` casteado. El cast desaparece porque lo hace el lenguaje.

**Flow scoping.** Lo interesante es dónde está en scope la variable `s`, porque no sigue la regla habitual de las llaves: está en scope **allí donde el compilador puede demostrar que el patrón casó**. Eso permite escribir esto, verificado en JDK 25:

```java
static void flowScoping(Object obj) {
    if (!(obj instanceof String s)) {
        System.out.println("no es String; 's' no está en scope aquí");
        return;
    }
    System.out.println("'s' SÍ está en scope tras el return: " + s.length());
}
```

Salida con `"hola"`:

```
I2 's' SI esta en scope tras el return: 4
```

`s` se usa **fuera** del `if` y funciona. El razonamiento del compilador: si el flujo llega a esa línea, es porque el `if` no se cumplió; el `if` era la negación del patrón; luego el patrón casó. Es análisis de flujo, no de bloques.

La misma lógica hace que esto funcione con `&&` y falle con `||`:

```java
if (obj instanceof String s && s.length() > 3) { ... }   // OK: si se evalúa la derecha, casó
if (obj instanceof String s || s.length() > 3) { ... }   // ERROR: 's' puede no estar asignada
```

Y usar la variable fuera de su alcance da un error normal de símbolo no encontrado. Verificado:

```
E6.java:4: error: cannot find symbol
        System.out.println(s);
                           ^
  symbol:   variable s
```

**Un detalle importante:** un patrón de tipo **nunca casa con `null`**. `null instanceof String s` es `false`, sin excepción. Eso hace que `if (obj instanceof String s)` sea a la vez una comprobación de tipo y una comprobación de nulidad, lo cual elimina una guarda.

## 40. Patrones en las etiquetas case

Java 21 permitió que una etiqueta `case` contenga un patrón en vez de una constante. Eso cambia el `switch` de raíz: **ya no compara por igualdad, compara por coincidencia de patrón**.

El código que se escribía antes:

```java
// Antes de Java 21
static String formatter(Object obj) {
    String formatted = "unknown";
    if (obj instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (obj instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (obj instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (obj instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}
```

El JEP 441 enumera exactamente qué está mal aquí, y merece la pena leerlo despacio porque es un argumento de diseño, no de estilo: *"this approach allows coding errors to remain hidden because we have used an overly general control construct"*. Nada obliga a que cada rama asigne `formatted`. Si una rama se olvida, hay un bug y el compilador calla. Además, una cadena de `if` es O(n) por construcción.

Con patrones en `case`:

```java
// Java 21+
static String formatterPatternSwitch(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> obj.toString();
    };
}
```

Ahora el compilador exige exhaustividad, cada rama produce un valor por construcción, y el despacho puede ser O(1).

**El tipo del selector se relajó.** Antes tenía que ser un entero, un `enum` o un `String`. Ahora puede ser **cualquier tipo de referencia**, lo que abre el `switch` a casos que antes eran impensables:

```java
record Point(int i, int j) {}
enum Color { RED, GREEN, BLUE }

static void typeTester(Object obj) {
    switch (obj) {
        case null     -> System.out.println("null");
        case String s -> System.out.println("String");
        case Color c  -> System.out.println("Color: " + c);
        case Point p  -> System.out.println("Record: " + p);
        case int[] ia -> System.out.println("Array de int de longitud " + ia.length);
        default       -> System.out.println("Otra cosa");
    }
}
```

**Constantes de `enum` cualificadas.** Un cambio menor pero práctico: antes, en un `switch` sobre un `enum`, las etiquetas tenían que ser el nombre simple de la constante. Ahora se admite el nombre cualificado, lo que permite hacer `switch` sobre un supertipo:

```java
sealed interface Currency permits Coin {}
enum Coin implements Currency { HEADS, TAILS }

static void bien(Currency c) {
    switch (c) {
        case Coin.HEADS -> System.out.println("Cara");
        case Coin.TAILS -> System.out.println("Cruz");
    }
}
```

Cuando el selector no es del tipo del `enum`, la cualificación es **obligatoria**: escribir `case TAILS` ahí es error de compilación.

## 41. Guardas con when

Un patrón de tipo casa con **todos** los valores de ese tipo. A menudo hace falta afinar más. Antes de Java 21 eso obligaba a partir la lógica entre la etiqueta y un `if` dentro de la rama:

```java
// Torpe: la condición está partida en dos sitios
switch (obj) {
    case String s:
        if (s.length() == 1) { ... }
        else { ... }
        break;
}
```

La solución es la **guarda**: una expresión booleana tras la palabra `when`, que se evalúa solo si el patrón casó.

```java
switch (obj) {
    case String s when s.length() == 1 -> ...
    case String s                      -> ...
}
```

Un ejemplo completo del JEP, que muestra cómo las guardas convierten una cascada de condiciones en una lista plana y legible:

```java
static void testStringNew(String response) {
    switch (response) {
        case null -> { }
        case String s when s.equalsIgnoreCase("YES") -> System.out.println("You got it");
        case String s when s.equalsIgnoreCase("NO")  -> System.out.println("Shame");
        case String s -> System.out.println("Sorry?");
    }
}
```

Toda la complejidad del test está **a la izquierda** de la flecha, y la lógica de negocio a la derecha. El JEP lo describe como *"a more readable style of switch programming"* y tiene razón.

Verificado en JDK 25 con guardas sobre `record`:

```java
static String describe(Shape s) {
    return switch (s) {
        case Circle(double r) when r > 100 -> "círculo enorme r=" + r;
        case Circle(double r) -> "círculo r=" + r;
        case Square(double side) when side == 0 -> "cuadrado degenerado";
        case Square(double side) -> "cuadrado lado=" + side;
        case Rect(double w, double h) when w == h -> "rectángulo que es cuadrado " + w;
        case Rect(double w, double h) -> "rectángulo " + w + "x" + h;
    };
}
```

Salida real:

```
L1 circulo enorme r=200.0
L1 circulo r=1.0
L1 cuadrado degenerado
L1 rectangulo que es cuadrado 4.0
L1 rectangulo 2.0x5.0
```

Fijate en que esto hace algo que el `switch` clásico **no podía hacer en absoluto**: comparar rangos. `case Circle(double r) when r > 100` es una condición de orden, no de igualdad. El `switch` ha absorbido una capacidad que era exclusiva del `if`.

**Dos restricciones:**

**Solo los patrones admiten guarda.** Una constante no. Verificado:

```
E5.java:4: error: guards are only allowed for case with a pattern
            case "hola" when s.length() > 2 -> {}
                        ^
```

**La guarda no cuenta para la exhaustividad.** El compilador no analiza qué hace la expresión de la guarda —es indecidible en general— así que un `switch` cuyas ramas sean todas guardadas **no es exhaustivo**, aunque a ojo cubran todo. Siempre hace falta al menos una rama sin guarda, o un `default`.

## 42. Dominancia entre etiquetas

Con patrones, más de una etiqueta puede casar con el mismo valor. Si el selector es un `String`, casan tanto `case String s` como `case CharSequence cs`. ¿Cuál gana?

La regla es simple: **gana la primera que aparece en el bloque**. No hay búsqueda del mejor ajuste.

```java
static void first(Object obj) {
    switch (obj) {
        case String s       -> System.out.println("Una cadena: " + s);
        case CharSequence cs -> System.out.println("Una secuencia de longitud " + cs.length());
        default -> { }
    }
}
```

Correcto: un `String` entra por la primera rama; un `StringBuilder`, por la segunda.

Ahora al revés:

```java
static void error(Object obj) {
    switch (obj) {
        case CharSequence cs -> System.out.println("Secuencia");
        case String s        -> System.out.println("Cadena");   // inalcanzable
        default -> { }
    }
}
```

`String` es subtipo de `CharSequence`, así que la primera etiqueta se traga todos los `String` y la segunda **no puede casar con nada**. Java trata eso como código inalcanzable y lo convierte en **error de compilación**. Verificado en JDK 25:

```
E2.java:5: error: this case label is dominated by a preceding case label
            case String s -> System.out.println(s);
                 ^
```

Se dice que la primera etiqueta **domina** a la segunda. Las reglas concretas:

- Un patrón domina a otro si todo valor que casa con el segundo casa también con el primero. Es decir, si el tipo del segundo es subtipo del primero.
- Un patrón **sin** guarda domina al **mismo** patrón **con** guarda: `case String s` domina a `case String s when s.length() > 0`.
- Un patrón con guarda domina a otro solo si su guarda es la constante `true`. El compilador no analiza guardas más allá de eso.
- Un patrón puede dominar a una constante: `case Integer i` domina a `case 42`.

De ahí sale el **orden canónico** de las etiquetas, que el JEP recomienda explícitamente:

```java
Integer i = ...;
switch (i) {
    case -1, 1 -> ...                   // 1. constantes
    case Integer j when j > 0 -> ...    // 2. patrones con guarda
    case Integer j -> ...               // 3. patrones sin guarda
}
```

> **Esto es una mejora enorme sobre el `if`.** Recordá la [sección 9](#9-el-orden-de-las-ramas-cambia-el-resultado): en una cadena de `if / else if`, poner la condición general antes que la específica deja código muerto y **el compilador no dice nada**. Con patrones en `switch`, ese mismo error no compila. El compilador dejó de ser un traductor y se convirtió en un revisor.

También es error tener **dos etiquetas que lo capturan todo**. Verificado:

```
E2.java: error: this case label is dominated by a preceding case label
```

Un `case Object o` seguido de `default` no compila, porque ambos cubren todo.

## 43. Record patterns

Un patrón de tipo comprueba el tipo y liga una variable. Un **record pattern** va más allá: comprueba el tipo **y descompone el objeto en sus componentes**.

Con un patrón de tipo hay que sacar las partes a mano:

```java
record Point(int x, int y) {}

static void printSum(Object obj) {
    if (obj instanceof Point p) {
        int x = p.x();
        int y = p.y();
        System.out.println(x + y);
    }
}
```

Con un record pattern, la extracción está en el propio patrón:

```java
static void printSum(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x + y);
    }
}
```

`Point(int x, int y)` casa si el valor es un `Point`, y al casar **llama a los accesores por vos** e inicializa `x` e `y`. Los nombres de las variables no tienen que coincidir con los de los componentes.

**Anidan.** Ahí está la potencia real:

```java
record Point(int x, int y) {}
enum Color { RED, GREEN, BLUE }
record ColoredPoint(Point p, Color c) {}
record Rectangle(ColoredPoint upperLeft, ColoredPoint lowerRight) {}

static void printColorOfUpperLeftPoint(Rectangle r) {
    if (r instanceof Rectangle(ColoredPoint(Point p, Color c), ColoredPoint lr)) {
        System.out.println(c);
    }
}
```

Un solo patrón navega tres niveles de objetos y saca el dato del fondo. La comparación que hace el JEP es acertada: construir el objeto anidando constructores y destruirlo anidando patrones son operaciones simétricas.

```java
// Construir
Rectangle r = new Rectangle(new ColoredPoint(new Point(x1, y1), c1),
                            new ColoredPoint(new Point(x2, y2), c2));

// Destruir
if (r instanceof Rectangle(ColoredPoint(Point(var x, var y), var c), var lr)) { ... }
```

**Cuatro reglas prácticas:**

1. **`var` funciona dentro de un patrón** y el compilador infiere el tipo del componente. `Point(var a, var b)` es lo mismo que `Point(int a, int b)`.
2. **`null` no casa con ningún record pattern.** Si un componente anidado es `null`, el patrón anidado falla y el conjunto no casa. Eso centraliza el manejo de errores: o casa todo, o no casa nada.
3. **Los genéricos se infieren.** `Box(Box(var s))` sobre un `Box<Box<String>>` infiere que `s` es un `String`.
4. **Solo funcionan con `record`.** No con clases normales, porque un `record` garantiza la correspondencia uno a uno entre componentes y accesores. Extenderlo a clases arbitrarias está en el trabajo futuro del JEP.

## 44. Tipos sellados y uniones cerradas

Esta sección junta todo lo anterior y produce la construcción más valiosa de la Parte VI.

Una interfaz o clase **`sealed`** declara explícitamente quiénes pueden implementarla o extenderla:

```java
sealed interface Shape permits Circle, Square, Rect {}
record Circle(double r) implements Shape {}
record Square(double side) implements Shape {}
record Rect(double w, double h) implements Shape {}
```

El compilador ahora **conoce la lista completa** de subtipos posibles. Y por tanto puede verificar que un `switch` los cubre todos, sin `default`. Verificado en JDK 25:

```java
static double area(Shape s) {
    return switch (s) {
        case Circle c -> Math.PI * c.r() * c.r();
        case Square q -> q.side() * q.side();
        case Rect r   -> r.w() * r.h();
    };
}
```

Salida real:

```
K1 Circle[r=2.0] area=12,57
K1 Square[side=3.0] area=9,00
K1 Rect[w=2.0, h=5.0] area=10,00
```

**Por qué esto importa tanto.** `sealed` + `record` + `switch` con patrones es la forma que tiene Java de expresar lo que en otros lenguajes se llama **tipo suma** o **unión etiquetada**: un valor que es exactamente una de N alternativas conocidas. Y la propiedad clave es esta:

> Si mañana alguien añade `record Triangle(double b, double h) implements Shape {}`, **todos los `switch` del proyecto que traten `Shape` sin `default` dejan de compilar de golpe**, y el compilador te señala uno por uno los sitios que hay que actualizar.

Eso convierte una tarea de arqueología —"buscar todos los sitios donde se trata una forma"— en una lista que te da el build. Es la razón por la que este patrón se ha vuelto el idioma preferido para modelar estados, resultados de operaciones y árboles sintácticos en Java moderno.

Un ejemplo realista, del tipo que aparece en cualquier servicio:

```java
sealed interface ResultadoPago permits Aprobado, Rechazado, Pendiente {}
record Aprobado(String idTransaccion, BigDecimal importe) implements ResultadoPago {}
record Rechazado(String motivo, int codigo) implements ResultadoPago {}
record Pendiente(Instant reintentarA) implements ResultadoPago {}

String mensaje = switch (resultado) {
    case Aprobado(String id, var importe) -> "Cobrados " + importe + " (" + id + ")";
    case Rechazado(String motivo, int cod) when cod >= 500 -> "Error del banco: " + motivo;
    case Rechazado(String motivo, var cod) -> "Rechazado: " + motivo;
    case Pendiente(Instant cuando) -> "Reintentamos a las " + cuando;
};
```

Sin `default`, sin casts, sin comprobaciones de `null`, y con la garantía de que añadir un cuarto resultado rompe la compilación aquí.

**La exhaustividad con record patterns es más sutil.** El compilador analiza también los componentes. El JEP 440 da el ejemplo:

```java
sealed interface I permits C, D {}
final class C implements I {}
final class D implements I {}
record Pair<T>(T x, T y) {}

Pair<I> p;

// Exhaustivo: I es sealed, así que C y D cubren todo
switch (p) {
    case Pair<I>(I i, C c) -> ...
    case Pair<I>(I i, D d) -> ...
}

// NO exhaustivo: falta el par (D, D)
switch (p) {                          // error
    case Pair<I>(C fst, D snd) -> ...
    case Pair<I>(D fst, C snd) -> ...
    case Pair<I>(I fst, C snd) -> ...
}
```

## 45. Patrones sobre primitivos

Queda una asimetría: todo lo anterior funciona con tipos de referencia. Los primitivos siguen fuera, salvo como componentes de un record pattern.

El [JEP 455](https://openjdk.org/jeps/455) y sus sucesores están cerrando ese hueco. **En JDK 25 sigue siendo una función preview**, verificado ejecutando el compilador:

```
P1.java:10: error: primitive patterns are a preview feature and are disabled by default.
        if (i instanceof byte b) { System.out.println("cabe en byte: " + b); }
            ^
  (use --enable-preview to enable primitive patterns)
```

Compilando con `--enable-preview --release 25` funciona. Salida real:

```
grande 42
cabe en byte: 100
no cabe en byte: 300
uno / diez mil millones / otro 7
```

Lo que habilita, cuando deje de ser preview:

**1. `instanceof` sobre primitivos, que comprueba si la conversión es exacta:**

```java
if (i instanceof byte b) {   // ¿el valor de i cabe en un byte sin perder información?
    usar(b);
}
```

Eso sustituye al idioma manual de toda la vida:

```java
if (i >= -128 && i <= 127) {
    byte b = (byte) i;
    usar(b);
}
```

La idea central del JEP es elegante: `instanceof` siempre fue *"la precondición de un cast seguro"*, y con primitivos esa precondición depende del **valor**, no solo del tipo. Verificado arriba: `100` cabe en un `byte`, `300` no.

**2. `switch` sobre `long`, `float`, `double` y `boolean`**, que estaban prohibidos:

```java
long v = ...;
switch (v) {
    case 1L              -> ...;
    case 10_000_000_000L -> ...;
    case long x          -> ... x ...;
}
```

**3. Un `case` que captura el resto en vez de un `default` ciego:**

```java
switch (x.getStatus()) {
    case 0 -> "okay";
    case 1 -> "warning";
    case 2 -> "error";
    case int i -> "estado desconocido: " + i;   // el valor está disponible
}
```

> **Recomendación.** No uses esto en producción todavía. Las funciones preview requieren `--enable-preview` tanto al compilar como al ejecutar, la bandera obliga a que la versión de compilación y la de ejecución coincidan exactamente, y la sintaxis puede cambiar entre versiones. Conocerlo sí importa: indica hacia dónde va el lenguaje, y explica por qué la lista de tipos válidos para el selector de la [sección 25](#25-sobre-qué-tipos-se-puede-hacer-switch) está a punto de dejar de tener excepciones.

---

# Parte VII — Diseño

## 46. Cuándo la condicional sobra

Todo lo anterior trata de **cómo escribir** una condicional. Esta sección trata de la pregunta que separa un perfil mid de uno senior: **si tiene que existir**.

Una parte grande de las condicionales de un sistema real no expresan una decisión: expresan que a alguien le faltó una abstracción. Cuatro casos concretos, con su reemplazo.

### 46.1 El switch sobre un tipo, que quiere ser polimorfismo

```java
// Huele mal
double area(Figura f) {
    switch (f.getTipo()) {
        case CIRCULO: return Math.PI * f.getRadio() * f.getRadio();
        case CUADRADO: return f.getLado() * f.getLado();
        default: throw new IllegalArgumentException();
    }
}
```

El problema no es el `switch`: es que `Figura` tiene `getRadio()` y `getLado()`, y solo uno de los dos tiene sentido en cada instante. La clase modela estados imposibles.

La solución clásica es el polimorfismo: cada subtipo sabe calcular su área y no hay condicional en ninguna parte.

```java
interface Figura { double area(); }
record Circulo(double radio) implements Figura {
    public double area() { return Math.PI * radio * radio; }
}
record Cuadrado(double lado) implements Figura {
    public double area() { return lado * lado; }
}
```

**Pero cuidado con aplicarlo automáticamente.** El polimorfismo pone cada operación dentro de su tipo, lo cual es ideal si vas a **añadir tipos** a menudo y regular si vas a **añadir operaciones**: cada operación nueva (`perimetro`, `serializar`, `dibujar`) obliga a tocar todas las clases, y mete responsabilidades ajenas en el modelo de dominio.

Esa tensión tiene nombre —*el problema de la expresión*— y la respuesta moderna de Java es la de la [sección 44](#44-tipos-sellados-y-uniones-cerradas): con `sealed` + `switch` exhaustivo, la operación vive fuera de los tipos y **el compilador sigue garantizando que están todos cubiertos**. Es lo mejor de ambos mundos, y por eso un `switch` sobre un `sealed` ya no es un olor a código como lo era un `switch` sobre un campo `tipo`.

| Situación | Preferí |
|---|---|
| Añadís tipos a menudo, las operaciones son estables | Polimorfismo |
| Añadís operaciones a menudo, los tipos son estables | `sealed` + `switch` |
| La lógica no pertenece al dominio (serializar, renderizar) | `sealed` + `switch` |
| El "tipo" es un campo `String` o `int` | Ninguno de los dos: arreglá el modelo primero |

### 46.2 La cadena de if que quiere ser un mapa

```java
// Huele mal
String codigoPais(String pais) {
    if (pais.equals("Argentina")) return "AR";
    else if (pais.equals("Brasil")) return "BR";
    else if (pais.equals("Chile")) return "CL";
    // ... veinte más
    else return "??";
}
```

Esto no es lógica: son datos escritos como código. Cada país nuevo obliga a recompilar.

```java
private static final Map<String, String> CODIGOS = Map.of(
        "Argentina", "AR",
        "Brasil", "BR",
        "Chile", "CL");

String codigoPais(String pais) {
    return CODIGOS.getOrDefault(pais, "??");
}
```

La señal para reconocerlo: **todas las ramas tienen la misma forma y solo cambian los literales**. Si podés escribir las ramas como filas de una tabla, es una tabla.

### 46.3 El if sobre null que quiere ser un tipo

```java
// Huele mal
Usuario u = repo.buscar(id);
if (u != null) {
    if (u.getEmail() != null) {
        enviar(u.getEmail());
    }
}
```

Tres niveles para una acción. Las salidas, en orden de calidad:

```java
// Aceptable: cláusulas de guarda
Usuario u = repo.buscar(id);
if (u == null) return;
if (u.getEmail() == null) return;
enviar(u.getEmail());

// Mejor: la API expresa la ausencia en el tipo
repo.buscarOpcional(id)
    .map(Usuario::getEmail)
    .ifPresent(this::enviar);

// La mejor: el null no existe porque el constructor lo impide
//   Usuario valida que email no sea null, así que la condición desaparece
enviar(repo.buscar(id).getEmail());
```

### 46.4 El if que devuelve un booleano

```java
// Huele mal
if (edad >= 18) {
    return true;
} else {
    return false;
}
```

La condición **ya es** el valor:

```java
return edad >= 18;
```

Variante frecuente y también innecesaria:

```java
if (lista.isEmpty() == true) { ... }   // sobra el == true
if (lista.isEmpty() != false) { ... }  // peor todavía
if (lista.isEmpty()) { ... }           // así
```

## 47. Anidamiento, complejidad ciclomática y cláusulas de guarda

La **complejidad ciclomática** es una métrica que cuenta los caminos independientes de un método. La forma práctica de calcularla: uno, más uno por cada `if`, `case`, `&&`, `||`, `?:`, `catch` y bucle.

Se usa como umbral en herramientas de análisis estático (SonarQube, Checkstyle, PMD) porque correlaciona bien con dos cosas medibles: la cantidad de tests necesarios para cubrir el método —hacen falta al menos tantos casos como caminos— y la probabilidad de que tenga bugs. Los umbrales habituales son **10 para avisar y 15 para bloquear**.

El anidamiento es peor que el número total de ramas, porque obliga a mantener todas las condiciones activas en la cabeza a la vez:

```java
// Cuatro niveles: hay que recordar cuatro condiciones para leer la línea del fondo
public void procesar(Pedido pedido) {
    if (pedido != null) {
        if (pedido.getItems() != null && !pedido.getItems().isEmpty()) {
            if (pedido.getCliente().estaActivo()) {
                if (pedido.getTotal().compareTo(BigDecimal.ZERO) > 0) {
                    cobrar(pedido);
                } else {
                    log.warn("total no positivo");
                }
            } else {
                log.warn("cliente inactivo");
            }
        } else {
            log.warn("pedido sin items");
        }
    }
}
```

La técnica que lo arregla es la **cláusula de guarda**: sacar cada caso de rechazo al principio, con salida temprana, y dejar el camino feliz al final sin indentar.

```java
public void procesar(Pedido pedido) {
    if (pedido == null) {
        return;
    }
    if (pedido.getItems() == null || pedido.getItems().isEmpty()) {
        log.warn("pedido sin items");
        return;
    }
    if (!pedido.getCliente().estaActivo()) {
        log.warn("cliente inactivo");
        return;
    }
    if (pedido.getTotal().compareTo(BigDecimal.ZERO) <= 0) {
        log.warn("total no positivo");
        return;
    }

    cobrar(pedido);
}
```

La complejidad ciclomática es la misma —no se eliminó ninguna rama—, pero **la carga cognitiva no lo es**. En la segunda versión, al llegar a `cobrar(pedido)` no hay ninguna condición pendiente: todo lo que podía fallar ya salió. Y cada guarda se lee sola, sin contexto.

Tres notas sobre esto:

**El debate del "single return".** Existe una escuela que sostiene que un método debe tener un único punto de salida. Nació con lenguajes sin gestión automática de recursos, donde cada salida obligaba a liberar memoria a mano. En Java, con `try-with-resources` y recolector de basura, ese argumento no aplica, y forzar un único `return` produce exactamente el anidamiento que acabamos de eliminar. La práctica moderna es la contraria: **salir temprano y a menudo**.

**Extraer métodos también reduce el anidamiento.** Cuando las guardas son muchas o tienen lógica, un método `validar(pedido)` que lanza una excepción con el motivo suele ser mejor que cinco guardas seguidas.

**El indicador más honesto no es la métrica, es el ancho.** Si tenés que hacer scroll horizontal para leer una línea, o si la línea empieza pasada la mitad de la pantalla, hay demasiado anidamiento. No hace falta ninguna herramienta para detectarlo.

---

# Parte VIII — Cierre

## 48. Casos de uso reales

Cinco situaciones sacadas de código de producción, con la construcción que corresponde a cada una.

### 48.1 Validar la entrada de una petición

Cláusulas de guarda al principio del método, que fallan rápido y con un mensaje útil.

```java
public Reserva crear(SolicitudReserva solicitud) {
    if (solicitud == null) {
        throw new IllegalArgumentException("La solicitud es obligatoria");
    }
    if (solicitud.fechaEntrada() == null || solicitud.fechaSalida() == null) {
        throw new IllegalArgumentException("Las fechas son obligatorias");
    }
    if (!solicitud.fechaSalida().isAfter(solicitud.fechaEntrada())) {
        throw new IllegalArgumentException("La salida debe ser posterior a la entrada");
    }
    if (solicitud.huespedes() < 1 || solicitud.huespedes() > MAX_HUESPEDES) {
        throw new IllegalArgumentException(
                "Los huéspedes deben estar entre 1 y " + MAX_HUESPEDES);
    }

    return repositorio.guardar(construir(solicitud));
}
```

Por qué así: cada condición es independiente y tiene su mensaje; el orden va de lo más barato de comprobar a lo más caro; y el camino correcto queda en una sola línea sin indentar.

### 48.2 Mapear un estado a una acción

`switch` como expresión sobre un `enum`, sin `default`.

```java
public Duration tiempoDeEspera(EstadoPedido estado) {
    return switch (estado) {
        case RECIBIDO  -> Duration.ofHours(48);
        case PAGADO    -> Duration.ofHours(24);
        case PREPARADO -> Duration.ofHours(8);
        case ENVIADO   -> Duration.ofDays(5);
        case ENTREGADO -> Duration.ZERO;
    };
}
```

Por qué así: si mañana el negocio añade `EstadoPedido.DEVUELTO`, este método deja de compilar y alguien tiene que decidir su plazo. Con un `default -> Duration.ZERO` el estado nuevo tendría plazo cero en silencio.

### 48.3 Procesar el resultado de una operación que puede fallar de varias formas

`sealed` + record patterns + guardas.

```java
sealed interface RespuestaApi permits Ok, ErrorCliente, ErrorServidor, Timeout {}
record Ok(String cuerpo) implements RespuestaApi {}
record ErrorCliente(int codigo, String detalle) implements RespuestaApi {}
record ErrorServidor(int codigo) implements RespuestaApi {}
record Timeout(Duration transcurrido) implements RespuestaApi {}

public Accion decidir(RespuestaApi respuesta) {
    return switch (respuesta) {
        case Ok(String cuerpo) -> Accion.procesar(cuerpo);
        case ErrorCliente(int cod, var detalle) when cod == 429 -> Accion.reintentarConEspera();
        case ErrorCliente(int cod, var detalle) -> Accion.descartar("Error " + cod + ": " + detalle);
        case ErrorServidor(int cod) -> Accion.reintentar();
        case Timeout(Duration t) when t.toSeconds() > 30 -> Accion.alertar();
        case Timeout(Duration t) -> Accion.reintentar();
    };
}
```

Por qué así: el `429` (demasiadas peticiones) necesita un trato distinto del resto de errores de cliente, y la guarda lo expresa en la etiqueta en vez de en un `if` anidado. No hay casts, no hay `default`, y añadir una quinta respuesta rompe la compilación.

### 48.4 Elegir un valor por defecto

Ternario, o mejor, el método de la biblioteca.

```java
// Correcto
String nombreMostrado = Objects.requireNonNullElse(usuario.apodo(), usuario.nombre());

// También correcto, cuando el cálculo del respaldo es caro
String nombreMostrado = Objects.requireNonNullElseGet(usuario.apodo(), this::calcularNombre);
```

Por qué así: `requireNonNullElse` no tiene la trampa de conversión de tipos de la [sección 20](#20-el-ternario-tiene-tipo-y-ese-tipo-puede-lanzar-una-excepción), y `requireNonNullElseGet` evita calcular el respaldo cuando no hace falta.

### 48.5 Aplicar una regla de negocio con varios factores

Condiciones con nombre.

```java
public boolean puedeCancelar(Reserva reserva, Usuario usuario, Instant ahora) {
    boolean esElTitular = reserva.titularId().equals(usuario.id());
    boolean tienePermisoAdmin = usuario.roles().contains(Rol.ADMIN);
    boolean estaDentroDePlazo = ahora.isBefore(reserva.fechaEntrada().minus(PLAZO_CANCELACION));
    boolean estaEnEstadoCancelable = reserva.estado() == EstadoReserva.CONFIRMADA;

    return estaEnEstadoCancelable
            && (tienePermisoAdmin || (esElTitular && estaDentroDePlazo));
}
```

Por qué así: la última expresión se lee como la regla de negocio escrita en castellano, y cuando falla se pueden loguear los cuatro booleanos para saber exactamente cuál falló. La versión de una sola línea con todo metido dentro sería igual de correcta e imposible de depurar.

## 49. Anti-patrones

**1. El punto y coma después del `if`.**

```java
if (x > 100);           // MAL: el cuerpo del if es la sentencia vacía
{ hacerAlgo(); }
```
Activá `-Xlint:all`. El compilador lo detecta si se lo pedís.

**2. Omitir las llaves.**

```java
if (x > 10)
    a();
    b();                // MAL: b() se ejecuta siempre
```
Llaves siempre. Es el bug "goto fail".

**3. Comparar objetos con `==`.**

```java
if (nombre == "admin") { ... }        // MAL
if ("admin".equals(nombre)) { ... }   // bien
```
Y nunca comparar wrappers con `==`: falla a partir de 128.

**4. Negar mal una condición compuesta.**

```java
if (!estaLogueado && !tienePermiso) { denegar(); }   // MAL
if (!(estaLogueado && tienePermiso)) { denegar(); }  // bien
```

**5. Mezclar un wrapper y un primitivo en un ternario.**

```java
int n = flag ? mapa.get(k) : 0;                      // MAL: NPE latente
int n = Objects.requireNonNullElse(mapa.get(k), 0);  // bien
```

**6. `switch` sin `break` cuando no se quería caída.**

```java
switch (nivel) {
    case PREMIUM: d += 0.10;     // MAL: cae y acumula
    case ORO: d += 0.05;
}
switch (nivel) {                 // bien
    case PREMIUM -> d = 0.10;
    case ORO -> d = 0.05;
}
```

**7. Poner `default` en un `switch` sobre un `enum` o un `sealed`.**

```java
return switch (estado) {
    case NUEVO -> 48;
    case PAGADO -> 24;
    default -> 0;               // MAL: se traga los estados futuros en silencio
};
```
Sin `default`, añadir un estado es un error de compilación, que es lo que querés.

**8. Confiar en que `default` captura `null`.**

```java
switch (texto) {                // MAL: lanza NPE si texto es null
    case "a" -> ...;
    default -> ...;
}
switch (texto) {                // bien
    case null -> ...;
    case "a" -> ...;
    default -> ...;
}
```

**9. Condiciones con efectos colaterales.**

```java
if (validar() && registrarIntento()) { ... }   // MAL: no registra los fallos
```

**10. Anidar cuatro niveles en vez de usar guardas.** Ver la [sección 47](#47-anidamiento-complejidad-ciclomática-y-cláusulas-de-guarda).

**11. Devolver `true` o `false` desde un `if`.**

```java
if (edad >= 18) return true; else return false;   // MAL
return edad >= 18;                                 // bien
```

**12. Comparar `double` con `==`.**

```java
if (0.1 + 0.2 == 0.3) { ... }                     // MAL: nunca entra
if (Math.abs(a - b) < EPSILON) { ... }            // bien
```

**13. Una cadena de `if` que es una tabla.** Ver la [sección 46.2](#462-la-cadena-de-if-que-quiere-ser-un-mapa).

**14. Mezclar sintaxis de flecha y de dos puntos.** No compila, pero se intenta a menudo al migrar código viejo. Migrá el `switch` entero de una vez.

## 50. Checklist y tabla de decisión

**Antes de dar por buena una condicional:**

- [ ] ¿Tiene llaves, aunque el cuerpo sea de una línea?
- [ ] ¿Hay algún `;` inmediatamente después de un `if`?
- [ ] ¿Compara objetos con `equals` y primitivos con `==`?
- [ ] ¿Hay algún `Integer`, `Long` o `Character` comparado con `==`?
- [ ] ¿Algún `double` comparado con `==`?
- [ ] ¿Las guardas de `null` están antes de desreferenciar?
- [ ] ¿Usa `&&` y `||` en lugar de `&` y `|`?
- [ ] ¿Alguna condición llama a métodos con efectos colaterales?
- [ ] Si es un ternario, ¿ambas ramas tienen el mismo tipo de referencia?
- [ ] Si es un `switch`, ¿usa la sintaxis de flecha?
- [ ] Si es un `switch` que produce un valor, ¿está escrito como expresión?
- [ ] Si es un `switch` sobre un `enum` o un `sealed`, ¿evita el `default`?
- [ ] Si es un `switch` sobre una referencia que puede ser nula, ¿tiene `case null`?
- [ ] ¿El anidamiento pasa de tres niveles?
- [ ] ¿Existe una tabla, un polimorfismo o un tipo que haría innecesaria esta condicional?

**Qué construcción usar:**

| Si necesitás… | Usá |
|---|---|
| Decidir entre dos caminos con acciones distintas | `if / else` |
| Rechazar entradas inválidas al principio de un método | Cláusulas de guarda con `return` o `throw` |
| Elegir entre **dos valores** del mismo tipo | Ternario |
| Elegir un valor por defecto ante un `null` | `Objects.requireNonNullElse` |
| Elegir entre **varios valores** según un `enum` | `switch` expresión, con flecha, sin `default` |
| Ejecutar **acciones** distintas según un `enum` | `switch` sentencia, con flecha |
| Comparar contra muchas constantes de `String` o `int` | `switch` con flecha |
| Distinguir según el **tipo** en ejecución | `switch` con patrones, o polimorfismo |
| Descomponer un `record` y decidir según sus partes | `switch` con record patterns |
| Modelar "una de N alternativas" | `sealed` + `record` + `switch` exhaustivo |
| Comprobar tipo y castear | `instanceof` con patrón |
| Traducir claves a valores | Un `Map`, no una condicional |
| Comportamiento distinto por subtipo, con tipos que crecen | Polimorfismo |
| Caer de un caso al siguiente acumulando | `switch` clásico con `:` y comentario `// fall through` |

**Versión mínima de Java por función:**

| Función | Desde |
|---|---|
| `switch` sobre `enum` | Java 5 |
| `switch` sobre `String` | Java 7 |
| Flecha `->`, `switch` expresión, `yield` | Java 14 |
| `instanceof` con patrón | Java 16 |
| `sealed` | Java 17 |
| Patrones en `case`, `case null`, guardas `when`, record patterns | Java 21 |
| Patrones sobre primitivos | Preview en Java 23-25 |

## 51. Autoevaluación

**1. ¿Por qué `if (x = 5)` no compila en Java pero sí en C?**

<details><summary>Respuesta</summary>

Porque en Java la condición de un `if` tiene que ser de tipo `boolean` o `Boolean`, y el resultado de `x = 5` es un `int`. El error exacto es `incompatible types: int cannot be converted to boolean`. En C cualquier valor distinto de cero cuenta como verdadero, así que la asignación se acepta y el `if` se cumple. Java no tiene el concepto de *truthy*. El agujero que queda es cuando la variable ya es `boolean`: `if (activo = true)` sí compila, y por eso conviene no escribir nunca `== true`.
</details>

**2. ¿Qué imprime este código y por qué?**

```java
int x = 5;
if (x > 100);
{ System.out.println("hola"); }
```

<details><summary>Respuesta</summary>

Imprime `hola`. El `;` que sigue al `if` es una sentencia vacía y se convierte en el cuerpo completo del `if`; cuando la condición se cumple, ejecuta la nada. El bloque entre llaves no pertenece al `if`: es un bloque suelto que se ejecuta siempre. El compilador solo avisa si se le pasa `-Xlint:all`, y el aviso es `warning: [empty] empty statement after if`.
</details>

**3. En un `if` anidado con un solo `else`, ¿a qué `if` se asocia el `else`?**

<details><summary>Respuesta</summary>

Al `if` más cercano que todavía no tenga `else`, independientemente de la indentación. Es la regla del *dangling else*, especificada en el JLS §14.5 e idéntica a la de C, C++ y C#. La indentación no significa nada para el compilador, así que un `else` indentado para parecer del `if` externo será del interno igualmente. La defensa es poner siempre llaves.
</details>

**4. ¿Cuál es la diferencia real entre `&&` y `&` sobre dos booleanos?**

<details><summary>Respuesta</summary>

Ambos calculan el mismo AND lógico, pero `&&` **cortocircuita**: si el operando izquierdo es `false`, el derecho no se evalúa. `&` evalúa siempre los dos. La diferencia importa por corrección, no por rendimiento: el idioma `if (u != null && u.estaActivo())` protege de un `NullPointerException` precisamente porque la derecha no se evalúa cuando `u` es `null`. Con `&` esa misma línea lanza la excepción.
</details>

**5. ¿Por qué `Integer a = 128, b = 128; a == b` da `false` y con `127` da `true`?**

<details><summary>Respuesta</summary>

Por la caché de `Integer.valueOf`, que el JLS §5.1.7 obliga a mantener para el rango de -128 a 127. El *autoboxing* llama a `valueOf`, no a `new`, así que dentro de ese rango dos boxings del mismo número devuelven **el mismo objeto** y `==` da `true` por casualidad. Fuera del rango se crean objetos distintos. Es especialmente traicionero porque los tests usan valores pequeños y pasan, y producción usa ids reales y falla. La regla: nunca compares wrappers con `==`.
</details>

**6. ¿Por qué `Integer r = true ? unIntegerNulo : 0;` lanza `NullPointerException`?**

<details><summary>Respuesta</summary>

Porque el tipo de la expresión ternaria se decide en compilación mirando **las dos** ramas. La regla del JLS §15.25 dice que si un operando es de tipo primitivo `T` y el otro es su wrapper, el tipo de toda la expresión es `T`. Aquí los operandos son `Integer` e `int`, así que el tipo es `int`, y para producir un `int` hay que llamar a `intValue()` sobre el `Integer` nulo. El mensaje real lo confirma: `Cannot invoke "java.lang.Integer.intValue()" because "<local1>" is null`. Si el segundo operando fuera un `Integer` en vez de un `int`, no habría unboxing y el resultado sería `null` sin excepción.
</details>

**7. ¿Qué imprime `Object o = true ? Integer.valueOf(1) : Double.valueOf(2.0);`?**

<details><summary>Respuesta</summary>

`1.0`, y `o.getClass()` es `java.lang.Double`. La condición es `true`, así que se elige la rama del `Integer`, pero el tipo de la expresión completa se calculó en compilación: ambos wrappers se desempaquetan, se aplica promoción numérica binaria, el tipo común de `int` y `double` es `double`, y el `1` se convierte a `1.0` antes de reempaquetarse. La rama elegida fue la correcta; el tipo del resultado, no el esperado.
</details>

**8. ¿Por qué `switch` sobre un `String` nulo lanza `NullPointerException` aunque tenga `default`?**

<details><summary>Respuesta</summary>

Porque el `switch` sobre `String` se compila a una llamada a `hashCode()` sobre el selector antes de evaluar ninguna etiqueta. El mensaje lo delata: `Cannot invoke "String.hashCode()" because "<local1>" is null`. El `default` no se llega a considerar. Desde Java 21 existe `case null` para capturarlo, y la decisión de que `default` **no** capture `null` es deliberada, para no cambiar el comportamiento del código escrito antes.
</details>

**9. Si `"Aa"` y `"BB"` tienen el mismo `hashCode`, ¿cómo distingue un `switch` entre ellos?**

<details><summary>Respuesta</summary>

Con `equals`. El compilador genera dos `switch` encadenados: el primero es un `lookupswitch` sobre `hashCode()` que reduce los candidatos, y dentro de cada grupo se llama a `String.equals(...)` para desempatar; un segundo `lookupswitch` sobre un índice interno salta al código real. Verificado: ambos tienen hash `2112` y cada uno entra en su rama correcta. Como consecuencia, un `switch` sobre `String` funciona con cadenas construidas en ejecución, a diferencia de `==`.
</details>

**10. ¿Qué diferencia hay entre `tableswitch` y `lookupswitch`?**

<details><summary>Respuesta</summary>

`tableswitch` es una tabla de saltos indexada: salta directamente a la posición `valor - mínimo`, en O(1). Requiere valores densos y contiguos. `lookupswitch` es una tabla de pares clave-dirección ordenada, recorrida con búsqueda binaria, en O(log n); se usa con valores dispersos. Verificado con `javap -c`: casos `1,2,3,4` producen `tableswitch { // 1 to 4` y casos `1, 1000, 1000000` producen `lookupswitch { // 3`. Un `switch` sobre `enum` es siempre `tableswitch` porque usa el `ordinal()`.
</details>

**11. ¿Por qué un `switch` como expresión tiene que ser exhaustivo y uno como sentencia no?**

<details><summary>Respuesta</summary>

Porque una expresión tiene que producir un valor. Si el selector no casara con ninguna etiqueta, no habría valor que devolver, lo cual es imposible. Una sentencia, en cambio, puede simplemente no hacer nada. La excepción moderna: desde Java 21, una **sentencia** `switch` que use patrones sí debe ser exhaustiva, porque el JEP 441 quiso aumentar la seguridad de esa forma nueva sin romper la compatibilidad de la vieja.
</details>

**12. ¿Por qué conviene NO poner `default` en un `switch` sobre un `enum`?**

<details><summary>Respuesta</summary>

Porque sin `default` el compilador exige que estén todas las constantes, y si mañana alguien añade una al `enum`, el `switch` **deja de compilar** y obliga a decidir qué hacer con el caso nuevo. Con `default`, la constante nueva se procesa por la rama por defecto en silencio, que casi nunca es lo correcto. Es contraintuitivo —parece menos defensivo— y es lo contrario: convierte al compilador en revisor de todos los cambios futuros del dominio.
</details>

**13. ¿Qué es `MatchException` y cuándo aparece?**

<details><summary>Respuesta</summary>

Es una excepción introducida en Java 21 que lanza el `default` **implícito** que el compilador inserta en los `switch` exhaustivos. Aparece cuando el `enum` o la jerarquía `sealed` cambió después de compilar el `switch` y no se recompiló el cliente. Verificado: compilando un `enum` con `NEW, PAID`, un `switch` exhaustivo sobre él, y luego recompilando **solo** el `enum` con una tercera constante, ejecutar el cliente viejo da `SHIPPED -> LANZA java.lang.MatchException: null`. En Java 14-20 lo que se lanzaba era `IncompatibleClassChangeError`. Verla en producción significa desincronización de compilación: la solución es recompilar, no añadir un `default`.
</details>

**14. ¿Cuáles son las cuatro formas de `switch` y cuál hay que usar?**

<details><summary>Respuesta</summary>

Las dos decisiones —dos puntos o flecha, sentencia o expresión— son ortogonales, así que hay cuatro: tipo 1 (sentencia con dos puntos, el clásico, cae), tipo 2 (expresión con dos puntos, cae, requiere `yield`), tipo 3 (sentencia con flecha, no cae) y tipo 4 (expresión con flecha, no cae, exhaustiva). La guía: usá el tipo 4 siempre que haya un valor que producir, el tipo 3 para acciones, el tipo 1 solo si necesitás caída real con código, y el tipo 2 nunca. Resumido: **flecha siempre; expresión cuando haya un valor**.
</details>

**15. ¿Qué es el flow scoping de una variable de patrón?**

<details><summary>Respuesta</summary>

Es que la variable introducida por un patrón está en scope **allí donde el compilador puede demostrar que el patrón casó**, en vez de dentro de un bloque delimitado por llaves. Por eso funciona escribir `if (!(obj instanceof String s)) return;` y usar `s` después del `if`: si el flujo llega ahí, el patrón casó necesariamente. Por la misma lógica, `obj instanceof String s && s.length() > 3` compila y `obj instanceof String s || s.length() > 3` no.
</details>

**16. ¿Por qué `case String s` antes de `case CharSequence cs` compila y al revés no?**

<details><summary>Respuesta</summary>

Por la regla de **dominancia**. La primera etiqueta que casa es la que gana, así que si `case CharSequence cs` va primero se traga todos los `String` y `case String s` queda inalcanzable. Java lo trata como código muerto y da error de compilación: `this case label is dominated by a preceding case label`. Al revés no hay problema porque un `StringBuilder` sigue llegando a la segunda rama. El orden canónico es: constantes, luego patrones con guarda, luego patrones sin guarda. Es una mejora clara sobre la cadena de `if`, donde el mismo error compila en silencio.
</details>

**17. ¿Casa un record pattern con `null`?**

<details><summary>Respuesta</summary>

No. Ni los record patterns ni los patrones de tipo casan nunca con `null`; `null instanceof String s` es `false` sin lanzar nada. Con patrones anidados eso se propaga: si un componente interno es `null`, el patrón anidado falla y el patrón completo no casa. Es una propiedad útil, porque centraliza el manejo de errores: o casa la estructura entera, o no casa. Para tratar el nulo en un `switch` hace falta una etiqueta `case null` explícita.
</details>

**18. ¿Qué gana un `switch` sobre un tipo `sealed` frente a uno sobre `Object` con `default`?**

<details><summary>Respuesta</summary>

Que el compilador conoce la lista cerrada de subtipos y puede verificar exhaustividad sin `default`. La consecuencia práctica es la que importa: al añadir un subtipo nuevo a la jerarquía, **todos** los `switch` del proyecto que la traten sin `default` dejan de compilar, y el build enumera los sitios que hay que actualizar. Convierte una búsqueda manual propensa a olvidos en una lista generada. Es la forma que tiene Java de expresar un tipo suma.
</details>

**19. ¿Sigue siendo `switch` sobre `long` una función preview en Java 25?**

<details><summary>Respuesta</summary>

Sí. Verificado compilando en JDK 25: sin banderas da `error: primitive patterns are a preview feature and are disabled by default`, y con `--enable-preview --release 25` compila y ejecuta. La función viene del JEP 455 y sus sucesores, y además de `switch` sobre `long`, `float`, `double` y `boolean`, habilita `instanceof` sobre primitivos —que comprueba si la conversión es **exacta**, no solo si el tipo encaja: `100 instanceof byte` es verdadero y `300 instanceof byte` es falso—. No usarlo en producción todavía; la bandera obliga a que las versiones de compilación y ejecución coincidan exactamente.
</details>

**20. ¿Cuándo hay que borrar una condicional en vez de escribirla mejor?**

<details><summary>Respuesta</summary>

Cuatro señales. **Una** cuando todas las ramas tienen la misma forma y solo cambian los literales: eso es una tabla, va en un `Map`. **Dos** cuando el `switch` es sobre un campo `tipo` y la clase tiene getters que solo valen para algunos tipos: el modelo permite estados imposibles, hay que partirlo en subtipos. **Tres** cuando la condicional es una comprobación de `null` sobre algo que nunca debería ser nulo: el arreglo es que el constructor lo impida. **Cuatro** cuando el `if` devuelve `true` o `false`: la condición ya es el valor. La pregunta a hacerse no es "cómo escribo mejor esta condicional" sino "por qué existe".
</details>

## 52. Fuentes

**Documentación oficial y especificación**

- [JLS §14.9 — The if Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.9) — incluye la regla del *dangling else*.
- [JLS §14.11 — The switch Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.11) y [§14.11.1 — Switch Blocks](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.11.1) — la gramática de las etiquetas, incluidas `case null` y las guardas.
- [JLS §15.25 — Conditional Operator](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.25) — las tablas que determinan el tipo del ternario. Es la referencia que explica los dos comportamientos sorprendentes de las secciones 20 y 21.
- [JLS §15.28 — switch Expressions](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.28) — exhaustividad y `yield`.
- [JLS §5.1.7 — Boxing Conversion](https://docs.oracle.com/javase/specs/jls/se25/html/jls-5.html#jls-5.1.7) — donde se exige la caché de -128 a 127.
- [JEP 361: Switch Expressions](https://openjdk.org/jeps/361) — la flecha, la expresión y `yield`. Explica por qué se descartó `break valor` en favor de `yield`.
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) — patrones en `case`, `case null`, guardas, dominancia y exhaustividad.
- [JEP 440: Record Patterns](https://openjdk.org/jeps/440) — descomposición y anidamiento.
- [JEP 394: Pattern Matching for instanceof](https://openjdk.org/jeps/394) — el origen del *flow scoping*.
- [JEP 455: Primitive Types in Patterns, instanceof, and switch](https://openjdk.org/jeps/455) — la función preview de la sección 45, con la definición de conversión *exacta*.
- [JEP 358: Helpful NullPointerExceptions](https://openjdk.org/jeps/358) — por qué los mensajes de NPE de este capítulo nombran el método y la variable exactos.

**Las dos fuentes de referencia de este capítulo, y dónde se equivocan**

- [Educative — What are conditional statements in programming?](https://www.educative.io/answers/what-are-conditional-statements-in-programming). Sirve como definición conceptual del tema y para poco más; **no es un artículo sobre Java**, y esto no es un detalle menor: todos sus ejemplos están en C#, incluida la sintaxis de `switch`, que en Java difiere desde Java 14. **Error 1, el más grave:** afirma literalmente *"bear in mind that all conditional statements return bool, i.e., true or false"*. Es falso en tres sentidos distintos. Una sentencia `if` no devuelve nada, porque es una sentencia. Un `switch` como expresión devuelve el tipo que sea —`int`, `String`, `Duration`—, no un booleano. Y lo que sí es booleano es la **condición**, que es otra cosa. Confundir la condición con la construcción es exactamente el error conceptual que la [sección 1](#1-qué-es-una-condicional-y-qué-problema-resuelve) tiene que deshacer. **Error 2:** su ejemplo "completo" del `else` abre con `using namespace Conditional;`, que es sintaxis de C++, no de C#; en C# sería `namespace Conditional`. **Error 3:** ese mismo ejemplo declara `public void Weather(string myDay)` **dentro** de `static void Main`, lo cual no compila. **Error 4:** el identificador aparece escrito de tres formas distintas en el mismo bloque —`myDay`, `myday`, `MyDay`— y faltan puntos y coma en tres llamadas a `Console.WriteLine`. Ninguno de sus fragmentos ilustrativos compilaría. **Error 5:** describe el `switch` diciendo que *"each case is tested until a true value is returned"*, como si cada `case` evaluara una condición booleana; en realidad compara por igualdad —o por coincidencia de patrón desde Java 21— y salta. **Error 6:** llama *"switch expressions"* a lo que son sentencias `switch`, y afirma que *"each block is terminated by a break keyword"*, que es justamente lo que dejó de ser cierto en Java con la sintaxis de flecha, donde `break` no solo no hace falta sino que **no se permite**.
- [W3Schools — Java If...Else](https://www.w3schools.com/java/java_conditions.asp) y sus subpáginas: [else](https://www.w3schools.com/java/java_conditions_else.asp), [else if](https://www.w3schools.com/java/java_conditions_elseif.asp), [Short Hand If...Else](https://www.w3schools.com/java/java_conditions_shorthand.asp), [Nested If](https://www.w3schools.com/java/java_conditions_nested.asp), [Logical Operators](https://www.w3schools.com/java/java_conditions_logical.asp), [Real-Life Examples](https://www.w3schools.com/java/java_conditions_reallife.asp) y [Switch](https://www.w3schools.com/java/java_switch.asp). Es correcta en lo que cubre —no le encontré ninguna afirmación falsa— y sus recomendaciones de estilo son buenas. **Dos aciertos que conviene reconocer**, porque no todos los tutoriales de este nivel los tienen: insiste en poner llaves siempre (*"always using braces makes your code clearer, easier to read, and prevents subtle bugs"*), y avisa explícitamente de no poner un punto y coma después de `if (condition)`, que es el anti-patrón de la [sección 5](#5-el-punto-y-coma-fantasma). **El problema es lo que falta, y falta casi todo lo posterior a 2011.** Su página de `switch` describe solo la forma `case x: ... break;`, con dos entradas en el menú lateral (la lección y un ejercicio). No menciona la sintaxis de flecha, ni el `switch` como expresión, ni `yield`, ni la exhaustividad, ni `case null`, ni las guardas `when`, ni los patrones, ni los record patterns. Un lector que aprenda `switch` ahí escribirá Java de 2011 en 2026. **Hueco 2:** su página del ternario no menciona en ningún momento que la expresión tiene un tipo calculado a partir de ambas ramas, así que no puede advertir del `NullPointerException` de la [sección 20](#20-el-ternario-tiene-tipo-y-ese-tipo-puede-lanzar-una-excepción) ni de la promoción de la [sección 21](#21-la-promoción-numérica-del-ternario), que son los dos únicos peligros reales del operador. **Hueco 3:** la página de `switch` no advierte de que un selector `null` lanza `NullPointerException`. **Imprecisión:** al explicar el `break` dice que detener las comparaciones *"saves a lot of execution time"*. Es engañoso: el ahorro real del `switch` no viene del `break` sino de la instrucción `tableswitch`, que salta en O(1) sin comparar nada; el `break` solo evita ejecutar el código de los casos siguientes.

**Discusiones y artículos de la comunidad** (consultados para contrastar los casos límite)

- [Java NPE in ternary operator with autoboxing?](https://stackoverflow.com/questions/7811608/java-npe-in-ternary-operator-with-autoboxing) — el hilo canónico sobre el NPE del ternario, con la explicación de que el tipo de la expresión es `int` y no `Integer`.
- [Java autoboxing and ternary operator madness](https://stackoverflow.com/questions/25417438/java-autoboxing-and-ternary-operator-madness) — el mismo problema en su forma más habitual: un `Map.get` que devuelve `null` en una rama y un literal numérico en la otra.
- [Returning null as an int permitted with ternary operator but not if statement](https://stackoverflow.com/questions/8098953/returning-null-as-an-int-permitted-with-ternary-operator-but-not-if-statement) — por qué `return (true ? null : 0);` compila y `if` no, recorriendo las reglas del JLS §15.25 una por una.
- [Why does the ternary operator unexpectedly cast integers?](https://stackoverflow.com/questions/8002603/why-does-the-ternary-operator-unexpectedly-cast-integers) — el caso `Integer` con `Double`, y la regla de la constante que cabe en el tipo menor.
- [Bug ID JDK-6977221: Ternary operator bug related to autoboxing](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6977221) — reportado como bug y cerrado como comportamiento especificado.
- [How To Use switch In Modern Java](https://nipafx.dev/java-switch/), de Nicolai Parlog — la mejor guía de decisión sobre las tres dimensiones ortogonales del `switch`. De aquí sale la recomendación de evitar `default` para que los cambios del dominio produzcan errores de compilación.
- [Java switch: 4 wrongs don't make a right](https://blog.joda.org/2019/11/java-switch-4-wrongs-dont-make-right.html), de Stephen Colebourne — la crítica al diseño y la taxonomía de los cuatro tipos de `switch` de la [sección 37](#37-las-cuatro-formas-del-switch). Es una lectura valiosa precisamente por estar en desacuerdo con la decisión oficial.
- [Design implications of Java's switch statements and switch expressions](https://blogs.oracle.com/javamagazine/post/java-switch-statements-expressions), en Java Magazine — de donde viene la caracterización del `switch` clásico como *"forward-only goto mechanism"*, que es la que hace comprensible el `fallthrough`.
- [Switch expression vs switch statement: which one to use](https://stackoverflow.com/questions/78717399/switch-expression-vs-switch-statement-which-one-to-use) — el desglose de las tres dimensiones independientes y por qué la de flecha debería ser la opción por defecto.
- [Error Prone — FallThrough](https://errorprone.info/bugpattern/FallThrough) — la regla de análisis estático que convierte el `fallthrough` no comentado en error.
- [Google Java Style Guide §4.8.4.2](https://google.github.io/styleguide/javaguide.html#s4.8.4.2-switch-fall-through) — la convención de exigir terminación abrupta o comentario explícito en cada grupo de sentencias.
- [How is Java 7 switch-on-string implemented?](https://www.benf.org/other/cfr/java7switchonstring.html) — el desazucarado completo a `hashCode` más `equals`, confirmado en este capítulo con `javap`.
- [Apple's SSL/TLS bug (goto fail)](https://www.imperialviolet.org/2014/02/22/applebug.html), analizado por Adam Langley — la vulnerabilidad real causada por un `if` sin llaves.

**Nota sobre la verificación.** Todos los outputs, mensajes de excepción y errores de compilación de este documento se obtuvieron ejecutando el código en Temurin JDK 25.0.3: los programas con `java Archivo.java`, los volcados de bytecode con `javap -c`, los avisos del compilador con `javac -Xlint:all`, y los errores compilando a propósito archivos que fallan. El experimento de `MatchException` de la [sección 36](#36-el-default-implícito-de-los-enums-y-matchexception) se hizo compilando el `enum` y su cliente por separado, recompilando solo el `enum` con una constante adicional y reejecutando el cliente sin recompilar. Los ejemplos de la [sección 45](#45-patrones-sobre-primitivos) se compilaron con `--enable-preview --release 25`. **En este capítulo no hay medidas de tiempo a propósito:** las construcciones condicionales son justamente las que más agresivamente reescribe el JIT, y un microbenchmark casero sobre ellas produce números que describen el banco de pruebas y no el lenguaje. El análisis de bytecode con `javap -c`, en cambio, es determinista y reproducible, y sostiene las mismas conclusiones sin inventar cifras.
