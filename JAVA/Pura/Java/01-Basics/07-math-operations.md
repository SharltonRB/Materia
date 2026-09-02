# Math Operations

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** Los capítulos anteriores establecieron qué tipos numéricos existen y cuánto ocupan ([Data Types and Variables](03-data-types-and-variables.md)), dónde vive cada variable ([Variables and Scopes](04-variables-and-scopes.md)) y qué pasa cuando un valor cambia de tipo ([Type Casting](05-type-casting.md)). Este cubre lo que ocurre **cuando esos números se operan entre sí**: sumar, restar, multiplicar, dividir, redondear, comparar y formatear.

Es un tema que casi todo el mundo cree saber después de veinte minutos. `+`, `-`, `*`, `/`, y a otra cosa. Y sin embargo la aritmética es, medida en incidentes de producción, una de las áreas más traicioneras del lenguaje. No porque sea difícil, sino porque **falla en silencio**: no hay excepción, no hay warning del compilador, no hay línea roja en el IDE. El programa sigue corriendo y devuelve un número equivocado.

La lista de bugs concretos que salen de este capítulo es larga y todos son reales:

- Un contador de milisegundos que da `1471228928` en vez de `31536000000` porque alguien escribió `1000 * 60 * 60 * 24 * 365` sin una `L`.
- Un `binarySearch` que lanza `ArrayIndexOutOfBoundsException` sobre arrays grandes — bug que vivió **nueve años dentro del propio JDK**.
- Una factura de 434 centavos cuando debía ser 435, porque `4.35 * 100` vale `434.99999999999994`.
- Un `Math.abs()` que devuelve un número **negativo** y revienta el acceso a un array.
- Un reparto de turnos que asigna el índice `-1` porque el `hashCode` era negativo y `%` conserva el signo del dividendo.
- Un total de carrito que muestra `0.8999999999999999 €`.
- Un `String.format("%.2f", precio)` que escribe `1234,50` en el servidor de producción y `1234.50` en la máquina del desarrollador, rompiendo el parser del sistema que consume el archivo.
- Un sorteo "aleatorio" en el que un tercio de los participantes tiene un 40 % más de probabilidad de ganar que el resto.

Ninguno de esos programas lanzó una excepción. Todos devolvieron un número, con toda la confianza del mundo, y ese número estaba mal.

Vamos a cubrir el modelo completo: cómo decide Java el tipo de un resultado, en qué se diferencian los dos mundos aritméticos que conviven en el lenguaje (enteros y punto flotante), qué garantiza y qué no garantiza cada uno, qué ofrece la clase `Math`, cómo se redondea de verdad, cómo se representa dinero sin perder un céntimo y cómo se generan números aleatorios sin sesgarlos.

**Sobre la verificación.** Todos los outputs, mensajes de excepción, volcados de bytecode y tiempos de este documento se ejecutaron realmente en un JDK 25; ninguno está copiado de un tutorial. Esto importa especialmente aquí, porque las tres fuentes de referencia que se usaron para prepararlo **contienen errores factuales en este tema concreto**: tipos de retorno equivocados, resultados con decimales donde el método devuelve un entero, y una API descrita como si estuviéramos en Java 8 cuando la clase `Math` ha ganado más de una docena de métodos desde entonces. Los errores están señalados uno por uno en la [sección 55](#55-fuentes), con el output real al lado.

---

## Índice

**Parte I — El modelo numérico de Java**

1. [Qué es una operación aritmética para el compilador](#1-qué-es-una-operación-aritmética-para-el-compilador)
2. [Los dos mundos: aritmética entera y aritmética de punto flotante](#2-los-dos-mundos-aritmética-entera-y-aritmética-de-punto-flotante)
3. [Promoción numérica: el tipo del resultado no es el que crees](#3-promoción-numérica-el-tipo-del-resultado-no-es-el-que-crees)
4. [Precedencia, asociatividad y orden de evaluación](#4-precedencia-asociatividad-y-orden-de-evaluación)

**Parte II — Los operadores**

5. [Suma, resta y multiplicación](#5-suma-resta-y-multiplicación)
6. [División: un operador con dos comportamientos](#6-división-un-operador-con-dos-comportamientos)
7. [El operador de resto y por qué no es el módulo matemático](#7-el-operador-de-resto-y-por-qué-no-es-el-módulo-matemático)
8. [Asignación compuesta y su cast oculto](#8-asignación-compuesta-y-su-cast-oculto)
9. [Incremento y decremento: el clásico i igual a i más más](#9-incremento-y-decremento-el-clásico-i-igual-a-i-más-más)
10. [Operadores unarios y el signo](#10-operadores-unarios-y-el-signo)
11. [Desplazamientos de bits como aritmética](#11-desplazamientos-de-bits-como-aritmética)

**Parte III — Aritmética entera: el mundo exacto pero finito**

12. [Cómo se representan los enteros: complemento a dos](#12-cómo-se-representan-los-enteros-complemento-a-dos)
13. [Overflow silencioso: el bug que no lanza excepción](#13-overflow-silencioso-el-bug-que-no-lanza-excepción)
14. [El caso Math.abs de Integer.MIN_VALUE](#14-el-caso-mathabs-de-integermin_value)
15. [La familia Exact: aritmética que sí avisa](#15-la-familia-exact-aritmética-que-sí-avisa)
16. [División con negativos: barra, floorDiv y ceilDiv](#16-división-con-negativos-barra-floordiv-y-ceildiv)
17. [floorMod: el módulo que siempre da positivo](#17-floormod-el-módulo-que-siempre-da-positivo)
18. [División y resto por cero](#18-división-y-resto-por-cero)

**Parte IV — Punto flotante: el mundo aproximado**

19. [IEEE 754 explicado desde cero](#19-ieee-754-explicado-desde-cero)
20. [Por qué 0.1 más 0.2 no es 0.3](#20-por-qué-01-más-02-no-es-03)
21. [float contra double, y por qué float casi nunca](#21-float-contra-double-y-por-qué-float-casi-nunca)
22. [Los valores especiales: infinitos, NaN y el cero negativo](#22-los-valores-especiales-infinitos-nan-y-el-cero-negativo)
23. [Comparar decimales sin equivocarse](#23-comparar-decimales-sin-equivocarse)
24. [Cancelación catastrófica y pérdida por magnitud](#24-cancelación-catastrófica-y-pérdida-por-magnitud)
25. [Sumar muchos decimales: el error acumulado](#25-sumar-muchos-decimales-el-error-acumulado)
26. [strictfp y JEP 306: una reliquia que ya no hace falta](#26-strictfp-y-jep-306-una-reliquia-que-ya-no-hace-falta)

**Parte V — La clase Math**

27. [Qué es Math y cómo está construida](#27-qué-es-math-y-cómo-está-construida)
28. [Funciones básicas: abs, max, min, signum, copySign, clamp](#28-funciones-básicas-abs-max-min-signum-copysign-clamp)
29. [Redondeo: round, floor, ceil y rint](#29-redondeo-round-floor-ceil-y-rint)
30. [Potencias, raíces y logaritmos](#30-potencias-raíces-y-logaritmos)
31. [Trigonometría](#31-trigonometría)
32. [Herramientas de precisión: fma, ulp, nextUp y compañía](#32-herramientas-de-precisión-fma-ulp-nextup-y-compañía)
33. [Math contra StrictMath](#33-math-contra-strictmath)
34. [Referencia completa de java.lang.Math en JDK 25](#34-referencia-completa-de-javalangmath-en-jdk-25)

**Parte VI — Redondear y formatear de verdad**

35. [El redondeo casero que todo el mundo escribe y por qué falla](#35-el-redondeo-casero-que-todo-el-mundo-escribe-y-por-qué-falla)
36. [RoundingMode: las ocho estrategias](#36-roundingmode-las-ocho-estrategias)
37. [Formatear: String.format, DecimalFormat y el locale](#37-formatear-stringformat-decimalformat-y-el-locale)
38. [Dos APIs de la JDK que redondean el mismo número de forma distinta](#38-dos-apis-de-la-jdk-que-redondean-el-mismo-número-de-forma-distinta)

**Parte VII — Dinero y precisión decimal**

39. [Por qué double no sirve para dinero](#39-por-qué-double-no-sirve-para-dinero)
40. [BigDecimal: modelo mental](#40-bigdecimal-modelo-mental)
41. [Construir un BigDecimal sin arruinarlo](#41-construir-un-bigdecimal-sin-arruinarlo)
42. [equals contra compareTo](#42-equals-contra-compareto)
43. [Dividir con BigDecimal](#43-dividir-con-bigdecimal)
44. [La alternativa: enteros de centavos](#44-la-alternativa-enteros-de-centavos)
45. [BigInteger](#45-biginteger)
46. [Cuánto cuesta cada opción](#46-cuánto-cuesta-cada-opción)

**Parte VIII — Aleatoriedad**

47. [Math.random y qué hay debajo](#47-mathrandom-y-qué-hay-debajo)
48. [Random, semillas y reproducibilidad](#48-random-semillas-y-reproducibilidad)
49. [El anti-patrón del módulo en aleatorios](#49-el-anti-patrón-del-módulo-en-aleatorios)
50. [ThreadLocalRandom y concurrencia](#50-threadlocalrandom-y-concurrencia)
51. [RandomGenerator y SecureRandom](#51-randomgenerator-y-securerandom)

**Parte IX — Cierre**

52. [Casos de uso reales](#52-casos-de-uso-reales)
53. [Anti-patrones](#53-anti-patrones)
54. [Checklist y tabla de decisión](#54-checklist-y-tabla-de-decisión)
55. [Fuentes](#55-fuentes)
56. [Autoevaluación](#56-autoevaluación)

---

# Parte I — El modelo numérico de Java

## 1. Qué es una operación aritmética para el compilador

Cuando escribís esto:

```java
int a = 7;
int b = 2;
int c = a + b;
```

parece que simplemente "sumaste dos números". Pero el compilador tuvo que tomar tres decisiones antes de generar una sola instrucción:

1. **Qué tipo tiene cada operando.** Aquí los dos son `int`.
2. **A qué tipo hay que convertir los operandos para poder operarlos.** Si fueran de tipos distintos, uno de los dos tendría que subir de categoría.
3. **Qué tipo tiene el resultado.** No es una pregunta trivial: `byte + byte` no da `byte`.

Ese conjunto de reglas se llama **promoción numérica binaria** y está definido en la Java Language Specification. Es la fuente número uno de sorpresas en este tema, porque el tipo del resultado se decide **antes** de mirar la variable a la que vas a asignarlo.

La segunda idea clave es que la JVM no tiene un "sumador universal". Tiene instrucciones separadas por tipo: `iadd` suma enteros de 32 bits, `ladd` suma enteros de 64, `dadd` suma doubles. Podemos verlo directamente. Compilemos este método:

```java
static int timesTwo(int x) {
    return x * 2;
}
```

y miremos su bytecode con `javap -c`:

```
static int timesTwo(int);
  Code:
       0: iload_0      // carga el parámetro x en la pila
       1: iconst_2     // carga la constante 2
       2: imul         // multiplica dos ints
       3: ireturn      // devuelve un int
```

Todas las instrucciones empiezan por `i` de `int`. Si el método trabajara con `double`, veríamos `dload`, `dmul`, `dreturn`. **El tipo no es una anotación decorativa: determina literalmente qué instrucción de máquina se ejecuta.** Y como cada instrucción tiene sus propias reglas sobre overflow y redondeo, el tipo determina también qué errores son posibles.

Una tercera cosa que hace el compilador y conviene conocer desde el principio: **plegado de constantes** (*constant folding*). Si todos los operandos son constantes conocidas en compilación, la operación se resuelve ahí mismo y no llega a ejecutarse nunca:

```java
static int constantFolding() {
    return 60 * 60 * 24;
}
```

```
static int constantFolding();
  Code:
       0: ldc           #7    // int 86400
       2: ireturn
```

No hay multiplicaciones en el bytecode: hay un `86400` ya calculado. Esto es importante porque significa que escribir `60 * 60 * 24` en lugar de `86400` **no cuesta nada en ejecución** y se lee muchísimo mejor. Es una de las poquísimas veces en programación en las que la opción legible y la rápida coinciden sin esfuerzo.

## 2. Los dos mundos: aritmética entera y aritmética de punto flotante

Java tiene dos sistemas aritméticos que funcionan con reglas distintas y que hay que aprender por separado. Casi todos los bugs de este capítulo nacen de aplicar la intuición de uno de los mundos al otro.

**Mundo 1: aritmética entera** (`byte`, `short`, `int`, `long`, y `char`, que aritméticamente se comporta como un entero sin signo de 16 bits).

- Es **exacta**: si el resultado cabe en el tipo, es matemáticamente correcto. No hay redondeo, no hay aproximación.
- Es **finita**: si el resultado no cabe, no se produce ningún error. Los bits sobrantes se descartan y obtenés otro número, normalmente de signo contrario. Esto se llama **overflow** y es silencioso.
- La **división descarta la parte fraccionaria**: `7 / 2` es `3`, no `3.5`.
- Dividir por cero **lanza `ArithmeticException`**.

**Mundo 2: aritmética de punto flotante** (`float`, `double`).

- Es **aproximada**: la mayoría de los decimales que escribís no se pueden representar exactamente en binario, así que se guarda el valor representable más cercano.
- Tiene **rango enorme** (hasta ~1.8 × 10^308 en `double`) pero **precisión limitada** (unos 15-17 dígitos significativos en `double`).
- La división **conserva la parte fraccionaria**: `7.0 / 2` es `3.5`.
- Dividir por cero **no lanza nada**: produce `Infinity`, `-Infinity` o `NaN`.

Compilado y ejecutado:

```java
System.out.println(100 / 8);      // 12       <- entero, descarta el resto
System.out.println(100 % 8);      // 4        <- el resto que descartó
System.out.println(100.0 / 8);    // 12.5     <- punto flotante
```

La tabla mental que hay que tener grabada:

| | Aritmética entera | Punto flotante |
|---|---|---|
| Exactitud | Exacta dentro del rango | Aproximada casi siempre |
| Desbordamiento | Silencioso, da la vuelta | Da `Infinity` |
| División por cero | `ArithmeticException` | `Infinity` / `NaN` |
| División de `7` entre `2` | `3` | `3.5` |
| Apto para dinero | Sí, contando centavos | **No** |
| Apto para medidas físicas, gráficos, estadística | Limitado | Sí |

Regla práctica que resume el capítulo entero: **si el valor cuenta cosas discretas (unidades, centavos, índices, IDs), usá enteros. Si mide magnitudes continuas (distancia, temperatura, probabilidad, porcentajes de progreso), usá `double`. Si es dinero y tiene que cuadrar al céntimo, usá `BigDecimal` o enteros de centavos, nunca `double`.**

## 3. Promoción numérica: el tipo del resultado no es el que crees

Esta es la regla que más veces sorprende. Java **nunca opera con tipos más pequeños que `int`**. Antes de sumar dos `byte`, los convierte a `int`, hace la suma en 32 bits y devuelve un `int`.

```java
byte b1 = 10, b2 = 20;
int sum = b1 + b2;      // compila: el resultado ya es int
// byte suma = b1 + b2; // NO compila: incompatible types: possible lossy conversion from int to byte
```

Las reglas completas de la **promoción numérica binaria**, en orden:

1. Si alguno de los operandos es `double`, el otro se convierte a `double`.
2. Si no, si alguno es `float`, el otro se convierte a `float`.
3. Si no, si alguno es `long`, el otro se convierte a `long`.
4. Si no, **ambos se convierten a `int`**, incluso si los dos eran `byte`, `short` o `char`.

La regla 4 explica el comportamiento de `char`:

```java
System.out.println('a' + 1);           // 98     <- int, no 'b'
System.out.println((char) ('a' + 1));  // b      <- hay que volver a bajar a char
System.out.println('b' - 'a');         // 1      <- restar caracteres da un int
```

`'a' + 1` no produce un carácter: produce el `int` 98, porque `char` se promocionó a `int`. Para volver a ver una letra hay que castear de vuelta. Este es el mecanismo detrás de trucos como convertir un dígito a su valor numérico con `c - '0'`.

**El caso que rompe producción.** La promoción se aplica operando a operando, de izquierda a derecha, sin mirar el destino:

```java
long millisEnUnAnio = 1000 * 60 * 60 * 24 * 365;
System.out.println(millisEnUnAnio);   // 1471228928
```

El resultado correcto es `31 536 000 000`, que no cabe en un `int` (máximo `2 147 483 647`). Como los cinco operandos son literales `int`, **toda la multiplicación ocurre en aritmética de 32 bits**, desborda, y solo *después* el resultado ya corrupto se convierte a `long`. La `L` de la asignación llegó tarde.

La solución es forzar la promoción desde el primer operando:

```java
long millisEnUnAnio = 1000L * 60 * 60 * 24 * 365;
System.out.println(millisEnUnAnio);   // 31536000000
```

Con `1000L`, el primer producto ya es `long`, y por la regla 3 todo lo demás se promociona a `long`. Basta con marcar **el primero**; el resto se contagia.

El mismo mecanismo aplica a la división:

```java
System.out.println(1 / 2 * 2.0);   // 0.0   <- 1/2 se resolvió como int = 0
System.out.println(2.0 * 1 / 2);   // 1.0   <- 2.0*1 = 2.0 (double), luego /2 = 1.0
```

Las dos expresiones tienen los mismos números y dan resultados distintos. No es un capricho del lenguaje: en la primera, `1 / 2` se evalúa entre dos `int` **antes** de que el `2.0` entre en escena.

> **Regla mnemotécnica:** en una expresión mixta, el `double` "contagia" a todo lo que toca, pero solo a partir del momento en que participa. Todo lo que se calculó antes ya se calculó con las reglas de los enteros, y ese daño no se deshace.

## 4. Precedencia, asociatividad y orden de evaluación

**Precedencia** es qué operador se aplica primero cuando hay varios. De mayor a menor, para lo que nos ocupa:

| Nivel | Operadores | Asociatividad |
|---|---|---|
| 1 | `++` `--` (postfijos) | — |
| 2 | `++` `--` (prefijos), `+` `-` unarios, `~`, `!`, casts | derecha a izquierda |
| 3 | `*` `/` `%` | izquierda a derecha |
| 4 | `+` `-` (binarios) | izquierda a derecha |
| 5 | `<<` `>>` `>>>` | izquierda a derecha |
| 6 | `<` `>` `<=` `>=` `instanceof` | izquierda a derecha |
| 7 | `==` `!=` | izquierda a derecha |
| 8 | `&`, luego `^`, luego `\|` | izquierda a derecha |
| 9 | `&&`, luego `\|\|` | izquierda a derecha |
| 10 | `? :` (ternario) | derecha a izquierda |
| 11 | `=` `+=` `-=` `*=` `/=` `%=` … | derecha a izquierda |

```java
int r = 2 + 3 * 4;      // 14, no 20: * antes que +
int s = (2 + 3) * 4;    // 20
```

**Asociatividad** es qué pasa entre operadores del mismo nivel. `*`, `/` y `%` son asociativos por la izquierda, así que `100 / 5 * 2` es `(100 / 5) * 2 = 40`, no `100 / (5 * 2) = 10`. Este detalle importa mucho más de lo que parece cuando hay divisiones enteras de por medio, porque cambia el punto donde se pierden los decimales.

**Orden de evaluación** es otra cosa distinta y conviene no confundirla. Java garantiza que los operandos se evalúan **de izquierda a derecha**, siempre, incluso cuando la precedencia dice que la operación se aplica en otro orden. Esto es lo que hace predecibles expresiones con efectos secundarios:

```java
int j = 5;
int k = j++ + ++j;
System.out.println("k=" + k + " j=" + j);   // k=12 j=7
```

Paso a paso: `j++` se evalúa primero y aporta `5` (dejando `j` en 6); después `++j` incrementa a 7 y aporta `7`; la suma es `12`. Es determinista, pero **si necesitás razonar así para entender una línea, esa línea está mal escrita**. La recomendación profesional es simple: no combines más de un incremento sobre la misma variable en una expresión.

Sobre los paréntesis: son gratis en ejecución (el compilador solo los usa para construir el árbol) y valen oro en legibilidad. `(a + b) / 2` y `a + b / 2` se leen casi igual y significan cosas muy distintas. En código profesional, poner paréntesis explícitos en cualquier expresión con más de dos operadores no es señal de desconfianza en el lenguaje, es cortesía con quien lo lea después.

---

# Parte II — Los operadores

## 5. Suma, resta y multiplicación

Los tres operadores más simples, con el matiz de que en Java `+` está sobrecargado: si **alguno** de los operandos es `String`, deja de ser suma y pasa a ser concatenación.

```java
int a = 10, b = 20;
System.out.println(a + b);          // 30
System.out.println("total: " + a + b);   // total: 1020   <- concatena, no suma
System.out.println("total: " + (a + b)); // total: 30
```

Esto ocurre por la asociatividad izquierda: `"total: " + a` ya produjo un `String`, así que el siguiente `+` concatena. Es un clásico en logs y mensajes de error. Los detalles de cómo se compila la concatenación están en [Strings and Methods](06-strings-and-methods.md).

Todo lo demás sobre estos tres operadores está en las reglas de promoción de la sección anterior y en el overflow de la [sección 13](#13-overflow-silencioso-el-bug-que-no-lanza-excepción). Merece la pena recordar dos propiedades que **no** se cumplen en Java aunque se cumplan en matemáticas:

```java
// La suma de punto flotante NO es asociativa
System.out.println((0.1 + 0.2) + 0.3);   // 0.6000000000000001
System.out.println(0.1 + (0.2 + 0.3));   // 0.6
```

Agrupar de otra forma da otro resultado. Esto tiene consecuencias prácticas: significa que **sumar una lista de doubles en paralelo puede dar un total distinto que sumarla en serie**, porque el orden de agrupación cambia. Si alguna vez ves que `list.stream().sum()` y `list.parallelStream().sum()` difieren en el último decimal, no es un bug: es IEEE 754.

## 6. División: un operador con dos comportamientos

El mismo símbolo `/` hace dos cosas completamente distintas según los tipos.

**Si los dos operandos son enteros**, la división es entera: se calcula el cociente y **se descarta la parte fraccionaria**, truncando siempre **hacia cero**.

```java
System.out.println(100 / 8);    // 12   (12.5 truncado)
System.out.println(-7 / 2);     // -3   (-3.5 truncado hacia cero, no hacia -4)
```

Ese "hacia cero" es importante y volveremos sobre él en la [sección 16](#16-división-con-negativos-barra-floordiv-y-ceildiv): con números negativos, truncar hacia cero **no** es lo mismo que redondear hacia abajo.

**Si alguno es de punto flotante**, la división conserva decimales:

```java
System.out.println(100.0 / 8);   // 12.5
```

**El error clásico** es hacer la división entera sin darse cuenta y convertir después:

```java
int aciertos = 3, total = 7;

double malA = aciertos / total;                  // 0.0   <- ya se perdió todo
double malB = (double) (aciertos / total);       // 0.0   <- castear al final no arregla nada
double bien = (double) aciertos / total;         // 0.42857142857142855
double tambienBien = 100.0 * aciertos / total;   // 42.857142857142854
```

La forma correcta es **promover antes de dividir**, no después. `(double) aciertos / total` funciona porque el cast tiene mayor precedencia que la división, así que el primer operando ya es `double` cuando se divide.

Y el caso de los porcentajes con enteros, que aparece en todos los dashboards mal hechos:

```java
System.out.println(aciertos / total * 100);   // 0    <- 3/7 = 0, y 0*100 = 0
System.out.println(aciertos * 100 / total);   // 42   <- multiplicar primero salva el cálculo
```

Con enteros, **multiplicar antes de dividir** conserva precisión. El riesgo del truco es que `aciertos * 100` puede desbordar si `aciertos` es grande; con `long` o con `double` deja de ser un problema.

## 7. El operador de resto y por qué no es el módulo matemático

`%` devuelve **lo que sobra** de una división entera. La identidad que lo define es:

```
(a / b) * b + (a % b) == a
```

Como `/` trunca hacia cero, `%` tiene que compensar, y eso hace que **el signo del resultado sea el del dividendo** (el operando izquierdo), no el del divisor:

```java
System.out.println(-7 % 2);    // -1
System.out.println(7 % -2);    //  1
System.out.println(-10 % 3);   // -1
```

En matemáticas, el módulo de `-10` en base 3 es `2`, porque el resultado de una operación módulo *n* siempre vive en el rango `[0, n)`. En Java, `-10 % 3` es `-1`. **Java implementa el resto (`remainder`), no el módulo (`modulo`).** Son operaciones distintas que coinciden solo cuando ambos operandos son positivos, que es justo el caso en el que todo el mundo lo prueba.

De ahí sale un bug muy frecuente: usar `%` para calcular un índice.

```java
String[] turnos = {"mañana", "tarde", "noche"};
int hash = -12346;
System.out.println(hash % turnos.length);               // -1  -> ArrayIndexOutOfBoundsException
System.out.println(Math.floorMod(hash, turnos.length)); //  2  -> índice válido
```

Cualquier vez que uses `%` para repartir en *buckets*, rotar un array, calcular una posición circular o distribuir carga, y el operando izquierdo pueda ser negativo (un `hashCode()` lo es la mitad de las veces), necesitás `Math.floorMod`. Está en la [sección 17](#17-floormod-el-módulo-que-siempre-da-positivo).

**`%` también funciona con decimales**, algo que muchos tutoriales omiten:

```java
System.out.println(5.5 % 2);     // 1.5
System.out.println(-5.5 % 2);    // -1.5
```

Y ojo con el uso típico para detectar pares e impares:

```java
// MAL: falla con negativos
boolean esImpar = n % 2 == 1;    // para n = -3 da false, porque -3 % 2 es -1

// BIEN
boolean esImparOk = n % 2 != 0;
```

## 8. Asignación compuesta y su cast oculto

`x += 3` parece azúcar sintáctico puro para `x = x + 3`. **No lo es**, y la diferencia es una pregunta de entrevista recurrente.

La especificación (JLS §15.26.2) define `E1 op= E2` como equivalente a:

```
E1 = (T) ((E1) op (E2))
```

donde `T` es el tipo de `E1`. Ese `(T)` es **un cast explícito que el compilador inserta por vos**. Y un cast estrechante puede perder información sin avisar:

```java
byte b = 10;
// b = b + 300;   // NO compila: possible lossy conversion from int to byte
b += 300;         // compila perfectamente
System.out.println(b);   // 54
```

`10 + 300` es `310`; al truncarlo a 8 bits queda `54`. El compilador no dice nada porque el cast está en la definición del operador. En bytecode se ve literalmente:

```
static byte compoundOnByte(byte);
  Code:
       0: iload_0
       1: sipush        300
       4: iadd          // suma en int: 310
       5: i2b           // <-- cast int-a-byte insertado por el compilador
       6: istore_0
```

Ese `i2b` es el cast fantasma. Lo mismo pasa con decimales:

```java
int n = 5;
n += 1.9;                // compila
System.out.println(n);   // 6   (5 + 1.9 = 6.9, truncado a int = 6)

// int m = 5; m = m + 1.9;   // NO compila
```

La lista de operadores afectados es `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`.

**Cuándo importa esto en la práctica:** casi siempre en acumuladores. Si tenés un contador `short` o `byte` (típico en código que mapea estructuras binarias o registros de protocolo) y le sumás con `+=`, el desbordamiento pasa desapercibido. La recomendación es contar en `int` o `long` salvo que haya una razón de memoria muy concreta, y nunca acumular en tipos menores que `int`.

## 9. Incremento y decremento: el clásico i igual a i más más

`++` y `--` suman o restan uno. La única sutileza es dónde se colocan:

- **Postfijo** (`i++`): el valor de la expresión es el valor **anterior** al incremento.
- **Prefijo** (`++i`): el valor de la expresión es el valor **posterior** al incremento.

```java
int i = 5;
System.out.println(i++);   // 5   (imprime, luego incrementa)
System.out.println(i);     // 6

int j = 5;
System.out.println(++j);   // 6   (incrementa, luego imprime)
```

Como sentencia aislada (`i++;` en una línea propia, o en el paso de un `for`) **son idénticos** y no hay ninguna diferencia de rendimiento: el compilador genera la misma instrucción `iinc` en ambos casos. El mito de que `++i` es más rápido viene de C++ con iteradores de objetos, no de Java con primitivos.

Ahora, el caso que aparece en todos los exámenes de certificación y que confunde a todo el mundo:

```java
int i = 5;
i = i++;
System.out.println(i);   // 5   <- ¡no 6!
```

La variable no cambia. La explicación con el bytecode real es incontestable:

```
static int postIncrementSelfAssign(int);
  Code:
       0: iload_0      // apila el valor actual de i (5)
       1: iinc   0, 1  // incrementa la variable i a 6
       4: istore_0     // guarda en i lo que había en la pila: 5
       5: iload_0
       6: ireturn
```

El valor `5` se copió a la pila **antes** del incremento, y la asignación final lo escribió encima del `6`. El incremento ocurrió de verdad y luego fue pisado. Con la versión prefija, en cambio, sí funciona:

```
static int preIncrementSelfAssign(int);
  Code:
       0: iinc   0, 1  // incrementa primero
       3: iload_0      // apila el 6
       4: istore_0     // guarda 6
```

**Conclusión práctica:** `i = i++` no es un puzzle que haya que dominar, es código que nunca hay que escribir. Los linters serios (SpotBugs, Error Prone) lo marcan como error. Lo mismo aplica a `a[i++] = i` y familia: si el orden de evaluación importa para entender la línea, dividí la línea.

Un detalle útil: `++` funciona sobre `char` y sobre tipos menores que `int` **sin cast**, porque, igual que la asignación compuesta, lleva la conversión incorporada:

```java
char c = 'a';
c++;
System.out.println(c);   // b
```

## 10. Operadores unarios y el signo

`+x` no hace nada útil (salvo forzar la promoción a `int`, que casi nunca se usa a propósito). `-x` cambia el signo, y tiene una trampa que se explica en la [sección 14](#14-el-caso-mathabs-de-integermin_value):

```java
System.out.println(-Integer.MIN_VALUE);   // -2147483648   <- sigue siendo negativo
```

No es un error de impresión: el rango de `int` es asimétrico (`-2 147 483 648` a `+2 147 483 647`), así que el opuesto del mínimo **no existe** como `int` y la negación desborda hasta volver a él mismo.

## 11. Desplazamientos de bits como aritmética

Los desplazamientos no son "operadores matemáticos" en el sentido clásico, pero se usan como tales y merecen un lugar aquí porque su comportamiento con negativos sorprende.

- `<<` desplaza a la izquierda: equivale a multiplicar por 2^n.
- `>>` desplaza a la derecha **conservando el signo** (rellena con el bit de signo): equivale a dividir por 2^n **redondeando hacia abajo**.
- `>>>` desplaza a la derecha rellenando con ceros: trata el número como si no tuviera signo.

```java
System.out.println(7 >> 1);     //  3     igual que 7 / 2
System.out.println(-7 >> 1);    // -4     ¡distinto de -7 / 2, que da -3!
System.out.println(-7 >>> 1);   //  2147483644
System.out.println(1 << 31);    // -2147483648   <- desbordó al bit de signo
```

`-7 >> 1` da `-4` porque el desplazamiento redondea **hacia menos infinito**, mientras que `/` trunca **hacia cero**. Es exactamente la misma diferencia que hay entre `/` y `Math.floorDiv` ([sección 16](#16-división-con-negativos-barra-floordiv-y-ceildiv)). De hecho, `x >> 1` y `Math.floorDiv(x, 2)` coinciden siempre.

Otra trampa: **la cantidad de desplazamiento se toma módulo 32** (o 64 para `long`):

```java
System.out.println(1 << 32);    // 1    <- 32 % 32 = 0, no desplaza nada
System.out.println(1L << 32);   // 4294967296
```

**¿Conviene usar shifts en lugar de multiplicar y dividir?** Casi nunca. El JIT ya convierte `x * 2` en un desplazamiento cuando le conviene, y `x / 2` en la secuencia óptima. Escribir `x >> 1` para "optimizar" una división sacrifica legibilidad, introduce el cambio de comportamiento con negativos y no gana nada medible. Los shifts se justifican cuando estás manipulando bits de verdad: máscaras, flags, protocolos binarios, hashing.

La excepción legítima y famosa es el punto medio de una búsqueda binaria, que veremos en la [sección 13](#13-overflow-silencioso-el-bug-que-no-lanza-excepción).

---

# Parte III — Aritmética entera: el mundo exacto pero finito

## 12. Cómo se representan los enteros: complemento a dos

Para entender el overflow hay que ver primero cómo se guardan los negativos. Java usa **complemento a dos**: el bit más significativo indica el signo, y los negativos se representan como "el complemento más uno". En 8 bits (un `byte`):

| Valor | Bits |
|---|---|
| 0 | `00000000` |
| 1 | `00000001` |
| 127 | `01111111` |
| -128 | `10000000` |
| -1 | `11111111` |

Dos consecuencias que explican todo lo demás:

1. **El rango es asimétrico.** Hay un valor negativo más que positivo, porque el cero ocupa un lugar del lado positivo. Por eso `Integer.MIN_VALUE` es `-2147483648` pero `Integer.MAX_VALUE` es `2147483647`.
2. **Los valores forman un círculo.** Sumarle 1 a `01111111` (127) da `10000000`, que se lee como -128. La aritmética "da la vuelta" como un cuentakilómetros.

```java
System.out.println(Integer.MAX_VALUE + 1);   // -2147483648
System.out.println(Integer.MIN_VALUE - 1);   //  2147483647
```

Los rangos completos, que conviene tener a mano:

| Tipo | Bits | Mínimo | Máximo |
|---|---|---|---|
| `byte` | 8 | -128 | 127 |
| `short` | 16 | -32 768 | 32 767 |
| `char` | 16 | 0 | 65 535 (sin signo) |
| `int` | 32 | -2 147 483 648 | 2 147 483 647 |
| `long` | 64 | -9 223 372 036 854 775 808 | 9 223 372 036 854 775 807 |

El número que hay que memorizar es el de `int`: **algo más de dos mil cien millones**. Cualquier cantidad que pueda acercarse a ese orden de magnitud (milisegundos acumulados, bytes de un archivo grande, contadores de eventos de un sistema con tráfico, sumas de dinero en centavos de una empresa mediana) **no cabe en un `int`** y hay que pensarla en `long` desde el principio.

## 13. Overflow silencioso: el bug que no lanza excepción

Java **no detecta el desbordamiento de enteros**. No hay excepción, no hay flag, no hay warning. La operación devuelve el resultado truncado a los bits disponibles y el programa continúa como si nada.

```java
System.out.println(Integer.MAX_VALUE + 1);      // -2147483648
System.out.println(100000 * 100000);            // 1410065408
System.out.println(1000 * 60 * 60 * 24 * 365);  // 1471228928
```

Esa es la diferencia esencial con la aritmética de punto flotante, que al menos te da un `Infinity` reconocible. Aquí obtenés un número perfectamente plausible.

**El caso real más famoso: la búsqueda binaria de la JDK.** Durante nueve años, `java.util.Arrays.binarySearch` contenía este cálculo del punto medio:

```java
int mid = (low + high) / 2;
```

Cuando `low` y `high` son grandes, la suma desborda antes de dividir:

```java
int low = 1_500_000_000, high = 2_000_000_000;
System.out.println((low + high) / 2);        // -397483648   <- índice negativo
System.out.println(low + (high - low) / 2);  // 1750000000   <- correcto
System.out.println((low + high) >>> 1);      // 1750000000   <- correcto
```

El bug se corrigió en 2006 y la solución que se adoptó en la JDK es `(low + high) >>> 1`: el desplazamiento sin signo reinterpreta los 32 bits desbordados como un número sin signo y recupera el valor correcto. La alternativa `low + (high - low) / 2` es más legible y funciona igual. Joshua Bloch, que documentó el incidente, señaló que el mismo error estaba en el libro *Programming Pearls* de Jon Bentley desde 1986 y sobrevivió veinte años sin que nadie lo notara.

**Otro caso cotidiano: la media de dos enteros.**

```java
int a = 2_000_000_000, b = 2_000_000_000;
System.out.println((a + b) / 2);          // -147483648
System.out.println(((long) a + b) / 2);   // 2000000000
```

**Cómo se protege uno en la práctica**, por orden de preferencia:

1. **Elegir bien el tipo desde el diseño.** Si la magnitud puede superar los dos mil millones, es `long`. Cuesta 4 bytes más y ahorra un incidente.
2. **Marcar los literales.** `1000L * 60 * 60` en vez de `1000 * 60 * 60`.
3. **Usar los métodos `*Exact`** cuando el desbordamiento sería un error de negocio ([sección 15](#15-la-familia-exact-aritmética-que-sí-avisa)).
4. **Validar los rangos de entrada** antes de operar, si los datos vienen de fuera.

## 14. El caso Math.abs de Integer.MIN_VALUE

Esta es la joya de la corona de los bugs aritméticos, porque combina dos cosas que nadie sospecha: que `Math.abs` puede devolver un negativo y que hacerlo no lanza nada.

```java
System.out.println(Math.abs(Integer.MIN_VALUE));   // -2147483648
```

La razón es la asimetría del rango: `|-2147483648|` es `2147483648`, que **no existe** en `int`. La implementación es literalmente `(a < 0) ? -a : a`, y ya vimos que `-Integer.MIN_VALUE` vuelve a ser `Integer.MIN_VALUE`.

El javadoc lo dice con todas las letras, y casi nadie lo lee:

> "Note that if the argument is equal to the value of `Integer.MIN_VALUE`, the most negative representable `int` value, the result is that same value, which is negative."

**Por qué esto llega a producción.** El patrón afectado es tan común que aparece en proyectos grandes:

```java
// El patrón peligroso: repartir por hash
int bucket = Math.abs(clave.hashCode()) % numeroDeBuckets;
```

Si `clave.hashCode()` devuelve exactamente `Integer.MIN_VALUE` —cosa perfectamente posible, es un valor de 32 bits como cualquier otro—, `Math.abs` devuelve un negativo, el `%` conserva el signo, y `bucket` es negativo:

```java
System.out.println(Math.abs(Integer.MIN_VALUE) % 3);   // -2
```

Resultado: `ArrayIndexOutOfBoundsException` en un caso entre cuatro mil millones. Es decir: nunca en tests, alguna vez en producción, y de forma irreproducible. Proyectos como Akka han tenido issues abiertos exactamente por esto.

**Las tres soluciones correctas**, de mejor a peor:

```java
// 1. floorMod: resuelve el signo sin pasar por abs. La mejor.
int bucket = Math.floorMod(clave.hashCode(), numeroDeBuckets);

// 2. Enmascarar el bit de signo: idiomático en tablas hash de la propia JDK
int bucket2 = (clave.hashCode() & 0x7fffffff) % numeroDeBuckets;

// 3. absExact: no lo arregla, pero convierte el error silencioso en excepción
int bucket3 = Math.absExact(clave.hashCode()) % numeroDeBuckets;
```

`Math.absExact` existe desde Java 15 precisamente para esto:

```java
Math.absExact(Integer.MIN_VALUE);
// java.lang.ArithmeticException: Overflow to represent absolute value of Integer.MIN_VALUE
```

Falla ruidosamente en vez de devolver basura. Si no podés cambiar la lógica, al menos cambiá el modo de fallo.

## 15. La familia Exact: aritmética que sí avisa

Java 8 añadió un conjunto de métodos que hacen la misma operación que los operadores, pero **lanzan `ArithmeticException` en vez de desbordar en silencio**. JDK 18 y versiones posteriores completaron la familia.

```java
Math.addExact(Integer.MAX_VALUE, 1);
// java.lang.ArithmeticException: integer overflow

Math.multiplyExact(100000, 100000);
// java.lang.ArithmeticException: integer overflow

Math.toIntExact(3_000_000_000L);
// java.lang.ArithmeticException: integer overflow

Math.divideExact(Integer.MIN_VALUE, -1);
// java.lang.ArithmeticException: integer overflow
```

Ese último caso es interesante: `Integer.MIN_VALUE / -1` es la única división de enteros que desborda, por la misma asimetría de siempre. Con el operador `/` obtenés `-2147483648` sin ninguna queja.

La familia completa en JDK 25, con sobrecargas para `int` y `long`:

| Método | Qué hace | Desde |
|---|---|---|
| `addExact` | suma | 8 |
| `subtractExact` | resta | 8 |
| `multiplyExact` | multiplicación | 8 |
| `incrementExact` / `decrementExact` | `+1` / `-1` | 8 |
| `negateExact` | cambio de signo | 8 |
| `toIntExact` | `long` a `int` | 8 |
| `absExact` | valor absoluto | 15 |
| `divideExact` | división | 18 |
| `floorDivExact` / `ceilDivExact` | división con redondeo dirigido | 18 |
| `multiplyFull` | `int * int` devolviendo `long` (nunca desborda) | 9 |
| `multiplyHigh` | los 64 bits altos de un producto de `long` | 9 |

**Cuándo usarlos.** No en todas partes: dentro de un bucle caliente añaden una comprobación por operación. La regla razonable es usarlos **en las fronteras del dominio**, donde un desbordamiento significaría corromper datos de negocio:

```java
// Cálculo de un total de pedido a partir de entrada externa
public long calcularTotal(long precioUnitario, int cantidad) {
    if (cantidad < 0) throw new IllegalArgumentException("cantidad negativa: " + cantidad);
    return Math.multiplyExact(precioUnitario, cantidad);   // falla ruidosamente si desborda
}
```

Aquí es exactamente lo que querés: si alguien manda una cantidad absurda, preferís un error 500 con stack trace claro antes que un total negativo guardado en la base de datos.

`multiplyFull` merece una mención aparte porque resuelve el problema sin excepciones:

```java
System.out.println(100000 * 100000);                   // 1410065408   (desbordado)
System.out.println(Math.multiplyFull(100000, 100000)); // 10000000000  (correcto, es long)
```

Cuando multiplicás dos `int` y sabés que el resultado puede ser grande, `multiplyFull` es más limpio que castear a mano.

## 16. División con negativos: barra, floorDiv y ceilDiv

Ya vimos que `/` trunca hacia cero. `Math.floorDiv` redondea hacia menos infinito y `Math.ceilDiv` (Java 18) hacia más infinito:

```java
System.out.println(-100 / 9);               // -11   trunca hacia cero
System.out.println(Math.floorDiv(-100, 9)); // -12   redondea hacia abajo
System.out.println(Math.ceilDiv(-100, 9));  // -11   redondea hacia arriba
System.out.println(Math.ceilDiv(100, 9));   //  12
```

Con operandos positivos, `/` y `floorDiv` coinciden. Con negativos, difieren en uno. Cuál querés depende del dominio:

- **Coordenadas de una cuadrícula, índices, tiempos relativos a un origen:** casi siempre `floorDiv`, porque querés que el comportamiento sea uniforme a ambos lados del cero. Si dividís el eje en celdas de 10 y truncás hacia cero, la celda que contiene el 0 tiene el doble de ancho que las demás (va de -9 a 9).
- **Cálculo de páginas, lotes o contenedores necesarios:** `ceilDiv`, que además evita un truco viejo y frágil.

El truco viejo para redondear hacia arriba una división entera es este:

```java
int elementos = 95, porPagina = 10;
System.out.println((elementos + porPagina - 1) / porPagina);        // 10
System.out.println(Math.ceilDiv(elementos, porPagina));             // 10
System.out.println((int) Math.ceil((double) elementos / porPagina)); // 10
```

Las tres dan lo mismo aquí, pero no son equivalentes:

- `(n + size - 1) / size` **puede desbordar** si `n` está cerca de `Integer.MAX_VALUE`.
- La versión con `Math.ceil` pasa por `double` y **pierde exactitud** con valores por encima de 2^53.
- `Math.ceilDiv` no tiene ninguno de los dos problemas y dice en su nombre lo que hace.

Desde Java 18, `Math.ceilDiv` es la respuesta correcta a "¿cuántas páginas/lotes/bloques necesito?".

## 17. floorMod: el módulo que siempre da positivo

`Math.floorMod` es a `%` lo que `floorDiv` es a `/`: la versión que redondea hacia abajo. La consecuencia práctica es que **el signo del resultado es el del divisor**, no el del dividendo:

```java
System.out.println(-10 % 3);                //  -1
System.out.println(Math.floorMod(-10, 3));  //   2   <- el módulo matemático
System.out.println(Math.floorMod(10, -3));  //  -2   <- signo del divisor
System.out.println(Math.floorMod(-12345, 8)); // 7   <- frente a -12345 % 8, que da -1
```

Como el divisor suele ser un tamaño (positivo por definición), `floorMod(x, n)` devuelve siempre algo en `[0, n)`, que es exactamente lo que necesitás para un índice.

Los tres usos canónicos:

```java
// 1. Bucket por hash, a prueba de hashCode negativos
int indice = Math.floorMod(clave.hashCode(), tabla.length);

// 2. Rotación circular de un array (incluso con desplazamiento negativo)
int siguiente = Math.floorMod(actual + paso, elementos.length);

// 3. Aritmética de relojes y calendarios
int horaFinal = Math.floorMod(horaActual - 5, 24);   // cinco horas antes, sin negativos
```

Ojo con una cosa: `floorMod` **no** protege de la división por cero.

```java
Math.floorMod(10, 0);   // java.lang.ArithmeticException: / by zero
```

## 18. División y resto por cero

En aritmética entera, dividir o sacar resto por cero lanza excepción:

```java
1 / 0;   // java.lang.ArithmeticException: / by zero
1 % 0;   // java.lang.ArithmeticException: / by zero
```

En punto flotante, **no lanza nada**:

```java
System.out.println(1.0 / 0);     //  Infinity
System.out.println(-1.0 / 0);    // -Infinity
System.out.println(0.0 / 0.0);   //  NaN
System.out.println(0.0 % 0.0);   //  NaN
```

Esta asimetría es una fuente clásica de confusión, y tiene una consecuencia de diseño importante: **en un cálculo con `double`, un divisor cero no detiene el programa, contamina el resultado**. El `Infinity` o el `NaN` se propaga por todas las operaciones siguientes y aparece mucho más tarde, normalmente al formatear o al guardar en base de datos, cuando ya perdiste el contexto de dónde se originó.

```java
double media = suma / cantidad;    // si cantidad es 0, media es NaN o Infinity
double porcentaje = media * 100;   // sigue siendo NaN
System.out.println("Media: " + porcentaje);   // "Media: NaN" en la UI del cliente
```

La defensa es validar antes de dividir, como en cualquier lenguaje, pero con la conciencia extra de que aquí **el lenguaje no te va a avisar**:

```java
double media = cantidad == 0 ? 0.0 : suma / cantidad;
```

---

# Parte IV — Punto flotante: el mundo aproximado

## 19. IEEE 754 explicado desde cero

`float` y `double` implementan el estándar **IEEE 754**, que guarda los números en tres piezas, igual que la notación científica pero en binario:

```
valor = signo × mantisa × 2^exponente
```

Para un `double` (64 bits): 1 bit de signo, 11 de exponente, 52 de mantisa. Para un `float` (32 bits): 1, 8 y 23.

La idea central es que **el número se guarda como una fracción binaria**. Y ahí está el problema: igual que en decimal no podés escribir 1/3 con un número finito de dígitos (`0.3333...`), en binario **no podés escribir 1/10 con un número finito de bits**. `0.1` en binario es periódico: `0.0001100110011...`

Cuando escribís `0.1` en Java, no guardás un décimo. Guardás el `double` más cercano a un décimo. Podemos ver su valor exacto pidiéndole a `BigDecimal` que lo expanda:

```java
System.out.println(new BigDecimal(0.1));
// 0.1000000000000000055511151231257827021181583404541015625
```

Ese es el número que realmente tenés en memoria cuando escribís `0.1`. Es un valor perfectamente definido y determinista —no es "ruido aleatorio"—, simplemente no es exactamente un décimo.

Lo mismo con `float`, donde el desajuste es mayor porque hay menos bits:

```java
System.out.println(new BigDecimal(0.1f));
// 0.100000001490116119384765625
```

**La consecuencia estructural** es que la precisión de un `double` no es absoluta sino **relativa**: hay unos 15-17 dígitos significativos, se coloquen donde se coloquen. Cerca del 1 podés distinguir cambios de 2 × 10^-16; cerca de 10^16 el número representable más próximo ya está a 2 unidades de distancia. Esa distancia entre dos representables consecutivos tiene nombre: **ULP** (*unit in the last place*), y Java te la da:

```java
System.out.println(Math.ulp(1.0));    // 2.220446049250313E-16
System.out.println(Math.ulp(1.0f));   // 1.1920929E-7
```

De ahí sale este resultado, que parece imposible:

```java
System.out.println(9007199254740992.0 + 1);   // 9.007199254740992E15   <- no cambió
```

2^53 = 9 007 199 254 740 992 es el último entero que un `double` puede representar exactamente. Pasado ese punto, sumar 1 no cambia nada, porque el siguiente `double` representable está a 2 de distancia. Es la razón por la que **no se deben guardar identificadores grandes en `double`**: un ID de Twitter o un `long` de base de datos con más de 16 dígitos pierde precisión al pasar por un `double` (o por el `number` de JavaScript, que es exactamente lo mismo). Este es el motivo real de que muchas APIs serialicen los IDs como `String`.

## 20. Por qué 0.1 más 0.2 no es 0.3

Ahora que sabemos qué hay dentro, el resultado más famoso de la informática deja de ser magia:

```java
System.out.println(0.1 + 0.2);          // 0.30000000000000004
System.out.println(0.1 + 0.2 == 0.3);   // false
```

`0.1` es un poquito más que un décimo, `0.2` es un poquito más que dos décimos, la suma de los dos errores se acumula y el resultado cae en un `double` distinto del más cercano a `0.3`.

Y no es un caso aislado ni un ejemplo de laboratorio. Estos son valores cotidianos de cualquier carrito de compra:

```java
System.out.println(2.00 - 1.10);   // 0.8999999999999999
System.out.println(1.10 + 2.20);   // 3.3000000000000003
System.out.println(0.1 * 3);       // 0.30000000000000004
System.out.println(4.35 * 100);    // 434.99999999999994
```

Ese último es el que arruina facturas. El patrón "convertir euros a centavos multiplicando por 100 y truncando" produce un céntimo menos:

```java
System.out.println((int) (4.35 * 100));      // 434   <- faltó un céntimo
System.out.println(Math.round(4.35 * 100));  // 435   <- redondear en vez de truncar lo salva
```

Un dato interesante: en `float` la suma de arriba *parece* funcionar:

```java
System.out.println(0.1f + 0.2f);   // 0.3
```

No es que `float` sea más preciso —es mucho menos—, sino que al tener menos dígitos, `Float.toString` imprime menos y el error queda oculto bajo el redondeo de la impresión. **La imprecisión sigue ahí; simplemente no se ve.** Esto es peor, no mejor: un error invisible es un error que descubrís en producción.

## 21. float contra double, y por qué float casi nunca

| | `float` | `double` |
|---|---|---|
| Tamaño | 32 bits | 64 bits |
| Dígitos significativos | ~6-7 | ~15-17 |
| Último entero exacto | 2^24 = 16 777 216 | 2^53 ≈ 9 × 10^15 |
| Literal | `3.14f` | `3.14` (por defecto) |

```java
System.out.println(16777216f + 1);   // 1.6777216E7   <- no cambió
```

Un `float` deja de poder contar de uno en uno a partir de los **16 millones**. Si tenés un contador de visitas, un acumulador de bytes o un identificador en `float`, tiene fecha de caducidad cercana.

**Cuándo usar `float`:** cuando tenés millones de valores y la memoria o el ancho de banda importan de verdad (buffers de audio, geometría 3D, tensores de machine learning, series temporales masivas), y sabés que 7 dígitos alcanzan. En cualquier otro caso, `double` es la opción por defecto —de hecho, es literalmente el tipo por defecto de los literales decimales en Java— y no hay razón para desviarse.

**Un detalle práctico:** mezclar `float` y `double` en una expresión promociona a `double`, así que un `float` en medio de un cálculo con doubles no ahorra nada y sí introduce una pérdida de precisión en el punto donde el valor se guardó.

## 22. Los valores especiales: infinitos, NaN y el cero negativo

IEEE 754 reserva tres familias de valores que no son números normales. Conocerlas es la diferencia entre depurar un `NaN` en diez minutos o en un día.

**Infinitos.** Aparecen al dividir por cero o al desbordar el rango:

```java
System.out.println(1.0 / 0);                        // Infinity
System.out.println(Double.MAX_VALUE * 2);           // Infinity
System.out.println(Math.log(0));                    // -Infinity
```

A diferencia del overflow entero, el de punto flotante **es visible**: `Infinity` es inconfundible en un log.

**NaN** (*Not a Number*) aparece en operaciones sin resultado definido:

```java
System.out.println(0.0 / 0.0);                              // NaN
System.out.println(Math.sqrt(-1));                          // NaN
System.out.println(Math.log(-1));                           // NaN
System.out.println(Double.POSITIVE_INFINITY - Double.POSITIVE_INFINITY);  // NaN
```

`NaN` tiene una propiedad que rompe la intuición y causa bugs sutiles: **no es igual a nada, ni siquiera a sí mismo**.

```java
double nan = 0.0 / 0.0;
System.out.println(nan == nan);     // false
System.out.println(nan != nan);     // true
System.out.println(nan > 1);        // false
System.out.println(nan < 1);        // false
System.out.println(nan == 1);       // false
```

Las tres comparaciones dan `false` a la vez, cosa que ninguna otra pareja de números hace. Eso significa que un `if (a >= b) {...} else {...}` con `NaN` **siempre entra por el `else`**, aunque el `else` esté escrito pensando en "a es menor". Ordenar una lista con un comparador basado en `<` y `>` sobre valores que puedan ser `NaN` produce resultados incoherentes y puede lanzar `IllegalArgumentException: Comparison method violates its general contract`.

La forma correcta de detectarlo es `Double.isNaN(x)`. Y aquí viene la incoherencia que hay que conocer: **los métodos de la clase `Double` tratan `NaN` de forma distinta al operador `==`**:

```java
System.out.println(Double.compare(nan, nan));                        // 0     <- "iguales"
System.out.println(Double.valueOf(nan).equals(Double.valueOf(nan))); // true  <- "iguales"
System.out.println(nan == nan);                                      // false <- "distintos"
```

No es un bug: `equals` y `compareTo` necesitan definir un **orden total** para que las colecciones y los ordenamientos funcionen, así que definen `NaN` como igual a sí mismo y mayor que todo lo demás. El operador `==` sigue la semántica IEEE. La consecuencia práctica: `List.contains(Double.NaN)` devuelve `true` y `set.add(Double.NaN)` funciona como esperás, pero un `==` en tu código no.

**El cero negativo.** IEEE 754 tiene dos ceros, y la misma incoherencia aparece aquí:

```java
System.out.println(0.0 == -0.0);                                      // true
System.out.println(Double.compare(0.0, -0.0));                        // 1     <- distintos
System.out.println(Double.valueOf(0.0).equals(Double.valueOf(-0.0))); // false <- distintos
System.out.println(Math.min(0.0, -0.0));                              // -0.0
System.out.println(1.0 / 0.0 == 1.0 / -0.0);                          // false (Infinity vs -Infinity)
System.out.println(Math.ceil(-0.4));                                  // -0.0
```

`-0.0` sale de operaciones perfectamente normales, como `Math.ceil` de un negativo pequeño, o de multiplicar un positivo por cero negativo. Si ese valor termina en un `HashMap` como clave, o en un `equals` de un objeto de dominio, tendrás dos entradas que "valen cero" y no se encuentran entre sí. La defensa habitual es normalizar: `if (x == 0.0) x = 0.0;` convierte `-0.0` en `0.0` (porque la comparación con `==` los considera iguales y la asignación pone el positivo).

## 23. Comparar decimales sin equivocarse

De todo lo anterior sale la regla más repetida y peor aplicada del tema: **no compares `double` con `==`**.

```java
double a = 0.1 + 0.2, b = 0.3;
System.out.println(a == b);   // false
```

La solución habitual es comparar con una **tolerancia** (epsilon):

```java
System.out.println(Math.abs(a - b) < 1e-9);   // true
```

Pero un epsilon absoluto solo funciona en un rango de magnitudes. Con números grandes, la distancia mínima entre representables ya es mayor que el epsilon:

```java
// 1e16 y 1e16+2 son valores distintos, pero el epsilon absoluto no los considera "casi iguales"
System.out.println(Math.abs(1e16 - (1e16 + 2)) < 1e-9);   // false
```

La comparación robusta es **relativa a la magnitud**:

```java
static boolean casiIguales(double x, double y, double epsilonRelativo) {
    if (x == y) return true;                       // cubre infinitos idénticos y ceros
    double diferencia = Math.abs(x - y);
    return diferencia <= epsilonRelativo * Math.max(Math.abs(x), Math.abs(y));
}

System.out.println(casiIguales(1e16, 1e16 + 2, 1e-9));   // true
System.out.println(casiIguales(0.1 + 0.2, 0.3, 1e-9));   // true
```

Tres apuntes para no equivocarse eligiendo:

1. **Si el dominio tiene una tolerancia natural, usala.** Si medís euros, la tolerancia es medio céntimo; si medís milímetros, es una décima. Un epsilon derivado del dominio se justifica solo y se documenta solo. Un `1e-9` copiado de Stack Overflow no.
2. **Para ordenar, usá `Double.compare`**, que define un orden total y gestiona `NaN` y `-0.0` sin sorpresas.
3. **Si necesitás igualdad exacta, no uses `double`.** Es la señal de que el dominio es decimal y querés `BigDecimal` o enteros.

Y el anti-patrón que hay que reconocer en revisiones de código:

```java
// MAL: bucle que puede no terminar nunca
for (double d = 0.0; d != 1.0; d += 0.1) { ... }
```

Como `0.1` sumado diez veces da `0.9999999999999999`, la condición `d != 1.0` nunca se cumple en el punto esperado. Los bucles se cuentan **con enteros**:

```java
for (int i = 0; i < 10; i++) {
    double d = i / 10.0;
    ...
}
```

## 24. Cancelación catastrófica y pérdida por magnitud

Hay dos formas de perder precisión que van más allá del error de representación y que conviene reconocer, porque explican resultados absurdos en cálculos que parecían inocentes.

**Absorción.** Al sumar dos números de magnitudes muy distintas, el pequeño puede desaparecer entero:

```java
System.out.println(1e16 + 1);   // 1.0E16   <- el 1 se perdió
```

**Cancelación catastrófica.** Al restar dos números casi iguales, los dígitos significativos se cancelan y lo que queda son los dígitos de error, amplificados:

```java
double x = 1.0000001e8;
System.out.println(x * x - 1.00000020000001e16);   // 0.0
```

El resultado matemático correcto no es cero, pero los dígitos que lo expresaban se perdieron en el producto. Este fenómeno es la razón de que exista `Math.hypot`:

```java
System.out.println(Math.sqrt(1e200 * 1e200 + 1e200 * 1e200));  // Infinity   <- desbordó en el paso intermedio
System.out.println(Math.hypot(1e200, 1e200));                  // 1.414213562373095E200
```

La fórmula ingenua eleva al cuadrado y desborda antes de poder sacar la raíz. `Math.hypot` está implementado para reescalar internamente y evitarlo. La lección general: **las funciones de `Math` que parecen atajos de conveniencia suelen ser reimplementaciones numéricamente estables de fórmulas que fallan escritas a mano.** Lo mismo aplica a `Math.expm1(x)` (más preciso que `Math.exp(x) - 1` para x pequeños) y `Math.log1p(x)` (más preciso que `Math.log(1 + x)`).

Consecuencia de orden práctico: **el orden de las operaciones importa numéricamente**. Sumar una lista de menor a mayor pierde menos precisión que de mayor a menor; multiplicar antes de dividir conserva más dígitos que al revés. En cálculos financieros o científicos, ese detalle deja de ser cosmético.

## 25. Sumar muchos decimales: el error acumulado

Cada operación introduce un error de redondeo minúsculo. En un bucle largo, esos errores se suman:

```java
double acc = 0;
for (int i = 0; i < 100; i++) acc += 0.01;
System.out.println(acc);   // 1.0000000000000007

double acc2 = 0;
for (int i = 0; i < 10_000; i++) acc2 += 0.1;
System.out.println(acc2);  // 1000.0000000001588
```

Después de diez mil sumas, el error ya está en el décimo decimal. Con millones de operaciones (una simulación, un agregado sobre una tabla grande), el error crece hasta hacerse visible en la respuesta.

Existe una técnica clásica para mitigarlo, la **suma compensada de Kahan**, que va arrastrando el error perdido en una variable auxiliar y lo reinyecta:

```java
static double sumaKahan(double[] valores) {
    double suma = 0.0;
    double compensacion = 0.0;              // el error acumulado que aún no se ha aplicado
    for (double v : valores) {
        double y = v - compensacion;
        double t = suma + y;
        compensacion = (t - suma) - y;      // recupera lo que se perdió al redondear
        suma = t;
    }
    return suma;
}
```

Lo interesante es que **no hace falta escribirla**: la JDK ya la usa internamente en los streams de doubles.

```java
double naive = 0;
for (int i = 0; i < 10_000; i++) naive += 0.1;
System.out.println(naive);   // 1000.0000000001588

double conStream = DoubleStream.generate(() -> 0.1).limit(10_000).sum();
System.out.println(conStream);   // 1000.0
```

Mismo cálculo, mismo tipo, resultado exacto. `DoubleStream.sum()` y `average()` aplican compensación de Kahan. Es un argumento poco conocido a favor de usar streams para agregados numéricos: además de leerse mejor, **suman mejor**.

(Detalle honesto: la especificación de `DoubleStream.sum` no *garantiza* un resultado más preciso que la suma secuencial, solo dice que puede serlo; pero la implementación de referencia compensa, y en la práctica el resultado es notablemente mejor.)

## 26. strictfp y JEP 306: una reliquia que ya no hace falta

Si abrís código Java antiguo, veréis a veces la palabra clave `strictfp` en clases o métodos que hacen cálculos numéricos:

```java
public strictfp class Calculo { ... }
```

Historia corta: entre Java 1.2 y Java 16, la JVM tenía permitido usar la precisión extendida de 80 bits de los procesadores x87 para los cálculos intermedios, lo que hacía que **el mismo programa pudiera dar resultados distintos en máquinas distintas**. `strictfp` forzaba la semántica estricta de IEEE 754 a costa de rendimiento.

**Desde Java 17 (JEP 306, "Restore Always-Strict Floating-Point Semantics") toda la aritmética de punto flotante es estricta siempre.** La justificación técnica es que los procesadores modernos llevan SSE2 desde 2001, que hace la aritmética estricta sin penalización.

Por lo tanto:

- `strictfp` sigue compilando por compatibilidad, pero **no hace nada**.
- `javac` emite un lint warning si lo usás.
- En código nuevo, no lo escribas. En código viejo, se puede borrar sin cambiar el comportamiento en Java 17+.

Lo que **sí** sigue siendo cierto es que dos JVM distintas pueden dar resultados distintos en funciones trascendentes (`sin`, `cos`, `pow`, `exp`), porque `Math` permite hasta 1 o 2 ulps de error y cada implementación puede elegir su algoritmo. Para reproducibilidad bit a bit entre plataformas, la herramienta es `StrictMath`, no `strictfp` ([sección 33](#33-math-contra-strictmath)).

---

# Parte V — La clase Math

## 27. Qué es Math y cómo está construida

`java.lang.Math` es una clase de utilidad: `final`, con constructor privado, **todos sus miembros son `static`**. No se instancia nunca:

```java
// Math m = new Math();   // no compila
double r = Math.sqrt(16); // se usa así, siempre
```

Al estar en `java.lang`, no hace falta importarla.

Tiene tres constantes públicas:

| Constante | Valor | Desde |
|---|---|---|
| `Math.PI` | 3.141592653589793 | 1.0 |
| `Math.E` | 2.718281828459045 | 1.0 |
| `Math.TAU` | 6.283185307179586 (2π) | 19 |

Dos cosas estructurales que conviene entender antes de usar sus métodos:

**1. Casi todo trabaja en `double`.** Las funciones matemáticas (`sqrt`, `pow`, `log`, `sin`…) reciben y devuelven `double`. Solo un subconjunto (`abs`, `max`, `min`, `round`, `floorDiv`, la familia `Exact`, `clamp`) tiene sobrecargas enteras. Esto tiene una consecuencia que muerde:

```java
System.out.println(Math.pow(2, 10));   // 1024.0   <- double, aunque los argumentos sean enteros
int potencia = (int) Math.pow(2, 10);  // hay que castear
```

Y otra más sutil: **cuando pasás un `int` a un método que tiene sobrecargas `float` y `double`, Java elige `float`**, porque `int` → `float` es una conversión más específica. Por eso ocurre esto, que despista a mucha gente:

```java
System.out.println(Math.ulp(1.0));   // 2.220446049250313E-16   <- ulp(double)
System.out.println(Math.ulp(1));     // 1.1920929E-7            <- ¡ulp(float)!
```

El mismo "1" da dos resultados distintos según se escriba `1` o `1.0`. Cuando el resultado dependa de la precisión, escribí siempre el literal como `double`.

**2. Muchos de sus métodos son *intrinsics*.** El JIT los reconoce por nombre y los sustituye por una instrucción de máquina (`sqrtsd` para `Math.sqrt`, por ejemplo) en vez de llamar al método. Por eso `Math.sqrt` no es más lento que escribir la fórmula a mano, y por eso los benchmarks ingenuos de `Math` dan resultados engañosos.

Una medición propia sobre 20 millones de llamadas en JDK 25 (sin JMH, cifras indicativas de una sola máquina):

```
Math.sqrt(i)    20000000 veces:   14,1 ms
Math.pow(i,0.5) 20000000 veces:   12,3 ms
```

Es decir: en este caso `Math.pow(x, 0.5)` no fue más lento que `Math.sqrt(x)`, contra lo que dice el consejo popular, porque el JIT reconoce el patrón. El consejo sigue siendo válido por **legibilidad** (`sqrt` dice lo que hace), pero no lo defiendas como optimización sin medirlo.

## 28. Funciones básicas: abs, max, min, signum, copySign, clamp

**`abs`** — valor absoluto. Sobrecargas para `int`, `long`, `float`, `double`. Recordá el caso `Integer.MIN_VALUE` de la [sección 14](#14-el-caso-mathabs-de-integermin_value) y su hermano seguro `absExact`.

```java
System.out.println(Math.abs(-20));    // 20
System.out.println(Math.abs(-0.0));   // 0.0
```

**`max` / `min`** — el mayor y el menor de dos valores.

```java
System.out.println(Math.max(10, 20));   // 20
System.out.println(Math.min(10, 20));   // 10
```

Con decimales tienen un comportamiento definido para los casos raros: si algún argumento es `NaN`, el resultado es `NaN`; y distinguen `0.0` de `-0.0`:

```java
System.out.println(Math.min(0.0, -0.0));   // -0.0
System.out.println(Math.max(-0.0, 0.0));   //  0.0
```

Un uso frecuente es acotar por un extremo: `Math.max(0, valor)` para no bajar de cero. Para acotar por los dos, desde Java 21 hay algo mejor.

**`clamp`** (Java 21) — acota un valor entre un mínimo y un máximo:

```java
System.out.println(Math.clamp(15, 0, 10));   // 10
System.out.println(Math.clamp(-5, 0, 10));   // 0
System.out.println(Math.clamp(7, 0, 10));    // 7
```

Reemplaza el clásico `Math.max(min, Math.min(max, valor))`, que es fácil de escribir al revés por accidente. Además valida que `min <= max` y lanza `IllegalArgumentException` si no. Es ideal para barras de progreso, porcentajes, volúmenes, coordenadas dentro de un canvas y cualquier valor que venga de fuera y deba quedar en rango.

**`signum`** — devuelve `-1.0`, `0.0` o `1.0` según el signo:

```java
System.out.println(Math.signum(-5.0));   // -1.0
System.out.println(Math.signum(0.0));    //  0.0
System.out.println(Math.signum(-0.0));   // -0.0   <- conserva el cero negativo
```

Cuidado: solo tiene sobrecargas `float` y `double`. Para enteros se usa `Integer.signum(int)`, que sí devuelve un `int`.

**`copySign`** — toma la magnitud del primer argumento y el signo del segundo:

```java
System.out.println(Math.copySign(5, -1));   // -5.0
```

Útil para "aplicarle a este resultado el signo de aquel dato", típico en física y en geometría.

## 29. Redondeo: round, floor, ceil y rint

Cuatro métodos que redondean de cuatro maneras distintas. Elegir mal es una de las causas más frecuentes de descuadres de un céntimo.

| Método | Qué hace | Tipo de retorno |
|---|---|---|
| `Math.floor(double)` | hacia menos infinito | `double` |
| `Math.ceil(double)` | hacia más infinito | `double` |
| `Math.round(double)` | al entero más cercano, empates **hacia arriba** | **`long`** |
| `Math.round(float)` | al entero más cercano, empates hacia arriba | **`int`** |
| `Math.rint(double)` | al entero más cercano, empates **al par** | `double` |

```java
System.out.println(Math.floor(7.343));   // 7.0
System.out.println(Math.ceil(7.343));    // 8.0
System.out.println(Math.round(7.343));   // 7
```

**Trampa 1: `round` devuelve un entero, `floor` y `ceil` devuelven `double`.** Es incómodo pero coherente: `floor` y `ceil` conservan el tipo porque su resultado puede no caber en un `long`. Ojo con esto al leer tutoriales: es habitual ver escrito que `Math.round(23.445)` devuelve `23.0`. **No**: devuelve `23`, un `long`. Y `Math.round(2.5f)` devuelve un `int`, no un `long`.

**Trampa 2: `round` no redondea "medio arriba" como te enseñaron en la escuela, sino "medio hacia más infinito".** La diferencia solo se nota con negativos:

```java
System.out.println(Math.round(2.5));    //  3
System.out.println(Math.round(-2.5));   // -2   <- ¡no -3!
System.out.println(Math.round(0.5));    //  1
System.out.println(Math.round(-0.5));   //  0   <- ni -1
System.out.println(Math.round(-1.5));   // -1
```

La definición es `floor(x + 0.5)`. Para `-2.5`, eso es `floor(-2.0) = -2`. Si tu dominio espera HALF_UP de verdad (que -2.5 vaya a -3, como hacen las normas contables), `Math.round` **no** es lo que querés: necesitás `BigDecimal` con `RoundingMode.HALF_UP`.

**Trampa 3: `rint` redondea al par en los empates** (*banker's rounding*), lo que sorprende la primera vez:

```java
System.out.println(Math.rint(2.5));    // 2.0   <- al par
System.out.println(Math.rint(3.5));    // 4.0   <- al par
System.out.println(Math.rint(-2.5));   // -2.0
```

No es un capricho: redondear siempre hacia arriba en los empates introduce un sesgo estadístico al alza cuando se acumulan muchos valores. Redondear al par lo compensa. Es el modo por defecto de IEEE 754 y el que exige la normativa contable en varios países.

**Trampa 4: `Math.ceil` puede devolver cero negativo.**

```java
System.out.println(Math.ceil(-0.4));   // -0.0
```

Si eso se concatena en un mensaje al usuario, verá `-0.0`.

**Trampa 5, la más sutil: `Math.round` tuvo un bug de verdad.** Hasta Java 7, la implementación literal de `floor(x + 0.5)` fallaba en el valor `0.49999999999999994` (el `double` inmediatamente anterior a 0.5), porque sumarle 0.5 daba exactamente 1.0 por redondeo:

```java
System.out.println(Math.round(0.49999999999999994));   // 0   (correcto desde Java 8)
```

En Java 7 y anteriores devolvía `1`. Fue el bug JDK-8010430. Es una anécdota, pero ilustra algo importante: **incluso "sumar 0.5 y truncar" es más difícil de lo que parece en punto flotante.** Que la propia JDK se equivocara durante años es la mejor razón para no escribir tu propia rutina de redondeo.

## 30. Potencias, raíces y logaritmos

```java
System.out.println(Math.pow(2, 8));    // 256.0
System.out.println(Math.sqrt(25));     // 5.0
System.out.println(Math.cbrt(125));    // 5.0
System.out.println(Math.exp(1));       // 2.7182818284590455
System.out.println(Math.log(Math.E));  // 1.0
System.out.println(Math.log10(1000));  // 3.0
System.out.println(Math.hypot(3, 4));  // 5.0
```

Puntos de atención:

**`pow` siempre devuelve `double`, y eso lo hace inadecuado para potencias enteras grandes.**

```java
System.out.println((long) Math.pow(10, 23));   // 9223372036854775807   <- Long.MAX_VALUE, basura
System.out.println(new BigDecimal(1e23).toPlainString());  // 99999999999999991611392
```

El `double` más cercano a 10^23 **no es** 10^23; es un número que termina en `...991611392`. Para potencias enteras exactas hay que usar `BigInteger.pow` o multiplicar en un bucle con `Math.multiplyExact`. (JDK 25 añadió `Math.powExact(int, int)` y `Math.powExact(long, int)`, que calculan potencias enteras y lanzan `ArithmeticException` si desbordan, resolviendo el problema de raíz.)

**`sqrt` de un negativo da `NaN`, no lanza excepción:**

```java
System.out.println(Math.sqrt(-1));   // NaN
```

**Las raíces no son exactas:**

```java
System.out.println(Math.sqrt(2) * Math.sqrt(2));   // 2.0000000000000004
```

**`log` es logaritmo natural, no en base 10.** `Math.log(100)` es 4.6, no 2. Para base 10 está `log10`, y para cualquier otra base se usa el cambio de base:

```java
static double logEnBase(double valor, double base) {
    return Math.log(valor) / Math.log(base);
}
System.out.println(logEnBase(8, 2));   // 3.0
```

**`log(0)` da `-Infinity` y `log` de un negativo da `NaN`.** Ambos se propagan silenciosamente. En cálculos con logaritmos (entropía, log-likelihood, escalas logarítmicas de gráficos), validar que el argumento sea estrictamente positivo es obligatorio.

**Las funciones "raras" existen por precisión, no por conveniencia:** `expm1(x)` calcula `e^x - 1` con mucha más precisión que hacerlo en dos pasos cuando `x` es pequeño, y `log1p(x)` hace lo propio con `log(1 + x)`. Si trabajás con tasas de interés pequeñas o probabilidades cercanas a cero, usarlas cambia el resultado.

## 31. Trigonometría

Todas las funciones trigonométricas trabajan en **radianes**, no en grados. Es el error número uno del tema:

```java
System.out.println(Math.sin(90));                    // 0.8939966636005579   <- 90 radianes
System.out.println(Math.sin(Math.toRadians(90)));    // 1.0                  <- 90 grados
```

Conversión en ambos sentidos:

```java
System.out.println(Math.toRadians(180));     // 3.141592653589793
System.out.println(Math.toDegrees(Math.PI)); // 180.0
```

Catálogo:

| Función | Qué calcula |
|---|---|
| `sin`, `cos`, `tan` | seno, coseno, tangente (argumento en radianes) |
| `asin`, `acos`, `atan` | funciones inversas (resultado en radianes) |
| `atan2(y, x)` | ángulo del punto (x, y) respecto al eje X, en `[-π, π]` |
| `sinh`, `cosh`, `tanh` | hiperbólicas |
| `hypot(x, y)` | `sqrt(x² + y²)` sin desbordar |

**`atan2` merece atención**: es la que se usa de verdad para calcular el ángulo de un vector, porque `atan(y/x)` pierde el cuadrante (y explota si `x` es 0). Fíjate en el orden de los argumentos: **primero la `y`**.

```java
double angulo = Math.atan2(puntoY - centroY, puntoX - centroX);
double grados = Math.toDegrees(angulo);
```

**Los resultados no son exactos** y hay que contar con ello:

```java
System.out.println(Math.cos(Math.PI));       // -1.0
System.out.println(Math.sin(Math.PI));       // 1.2246467991473532E-16   <- no es 0
System.out.println(Math.tan(Math.PI / 4));   // 0.9999999999999999       <- no es 1
```

`Math.sin(Math.PI)` no da cero porque `Math.PI` **no es π**: es el `double` más cercano a π. El seno de ese número, correctamente calculado, es ese valor minúsculo. Comparar resultados trigonométricos con `== 0` está condenado al fracaso; se comparan con tolerancia.

## 32. Herramientas de precisión: fma, ulp, nextUp y compañía

Este grupo de métodos rara vez se usa en código de negocio, pero conocerlos separa a quien "usa Math" de quien entiende el punto flotante.

**`fma(a, b, c)`** (*fused multiply-add*, Java 9) calcula `a * b + c` con **un solo redondeo** al final, en vez de redondear el producto y luego la suma:

```java
System.out.println(0.1 * 0.1 - 0.01);           // 1.734723475976807E-18
System.out.println(Math.fma(0.1, 0.1, -0.01));  // 9.020562075079397E-19
```

Los dos son "casi cero", pero el segundo es el resultado correctamente redondeado de la operación exacta. En productos escalares, evaluación de polinomios y álgebra lineal, usar `fma` reduce el error acumulado a la mitad de las operaciones.

**`ulp(x)`** da la distancia al siguiente representable, es decir, **la resolución local del tipo** en ese punto. Es la forma honesta de elegir un epsilon:

```java
System.out.println(Math.ulp(1.0));       // 2.220446049250313E-16
System.out.println(Math.ulp(1000000.0)); // 1.1641532182693481E-10
```

**`nextUp` / `nextDown` / `nextAfter`** dan el vecino inmediato:

```java
System.out.println(Math.nextUp(1.0));   // 1.0000000000000002
```

Sirven para construir rangos semiabiertos exactos y para tests de límites.

**`multiplyHigh` / `unsignedMultiplyHigh`** dan los 64 bits altos del producto de dos `long`, que es lo que necesitás para implementar aritmética de 128 bits o generadores de números aleatorios.

**`getExponent`, `scalb`, `rint`, `IEEEremainder`, `signum`, `copySign`** completan el conjunto de operaciones de IEEE 754. De ellas, la que más despista es `IEEEremainder`, que **no** es lo mismo que `%`:

```java
System.out.println(3.0 % 2.0);                // 1.0
System.out.println(Math.IEEEremainder(3, 2)); // -1.0
```

`%` usa división truncada; `IEEEremainder` usa la división **redondeada al más cercano**, así que el resultado queda en `[-|d|/2, |d|/2]` y puede ser negativo con operandos positivos. Se usa en cálculos angulares, no en aritmética general.

## 33. Math contra StrictMath

`java.lang.StrictMath` tiene la misma API que `Math`, con una diferencia de contrato:

- **`StrictMath` garantiza resultados idénticos bit a bit en todas las plataformas y versiones.** Sus algoritmos están fijados (derivan de la biblioteca `fdlibm`).
- **`Math` permite implementaciones más rápidas** con hasta 1 o 2 ulps de error en las funciones trascendentes, y anima explícitamente a los generadores de código a usar instrucciones del procesador.

El javadoc de `Math` lo dice así:

> "Unlike some of the numeric methods of class `StrictMath`, all implementations of the equivalent functions of class `Math` are not defined to return the bit-for-bit same results. This relaxation permits better-performing implementations where strict reproducibility is not required."

En la práctica, en una máquina moderna, muchos métodos de `Math` delegan en `StrictMath` y dan lo mismo:

```java
System.out.println(StrictMath.sin(1) == Math.sin(1));   // true (en este JDK y esta CPU)
```

Que coincidan hoy no significa que coincidan en otra CPU, otro sistema operativo u otra versión del JDK. **Esa es toda la cuestión**: `Math` no promete nada, `StrictMath` sí.

**Cuándo usar `StrictMath`:** cuando el resultado tiene que ser idéntico entre ejecuciones en máquinas distintas. Los casos reales son: simulaciones distribuidas que deben converger al mismo estado, motores de juego con lógica determinista replicada entre clientes, sistemas de auditoría que recalculan y comparan, y tests que comparan resultados numéricos exactos entre entornos. Para todo lo demás, `Math`.

Las operaciones básicas (`+`, `-`, `*`, `/`, `sqrt`) **son estrictamente reproducibles siempre**, porque IEEE 754 las define como correctamente redondeadas. La divergencia solo puede aparecer en trascendentes: `sin`, `cos`, `tan`, `exp`, `log`, `pow` y sus derivadas.

## 34. Referencia completa de java.lang.Math en JDK 25

Esta tabla se generó con `javap java.lang.Math` sobre el JDK instalado, no copiada de una lista publicada. Incluye métodos que **no aparecen en ninguna de las tres fuentes de referencia** de este capítulo, porque son posteriores a ellas.

**Constantes:** `E`, `PI`, `TAU` (19).

**Básicas y comparación**

| Método | Notas |
|---|---|
| `abs(int/long/float/double)` | ver caso `MIN_VALUE` |
| `absExact(int/long)` | 15 — lanza si desborda |
| `max`, `min` (4 sobrecargas cada uno) | `NaN` gana; distingue `±0.0` |
| `clamp(...)` | **21** — acota entre mínimo y máximo |
| `signum(float/double)` | para enteros: `Integer.signum` |
| `copySign(float/double)` | magnitud de uno, signo del otro |

**Redondeo**

| Método | Devuelve |
|---|---|
| `ceil(double)`, `floor(double)`, `rint(double)` | `double` |
| `round(float)` | `int` |
| `round(double)` | `long` |

**Aritmética entera segura y dirigida**

| Método | Desde |
|---|---|
| `addExact`, `subtractExact`, `multiplyExact`, `incrementExact`, `decrementExact`, `negateExact`, `toIntExact` | 8 |
| `floorDiv`, `floorMod` | 8 |
| `multiplyFull`, `multiplyHigh` | 9 |
| `divideExact`, `floorDivExact`, `ceilDivExact`, `ceilDiv`, `ceilMod`, `unsignedMultiplyHigh` | 18 |
| `unsignedMultiplyExact`, `powExact`, `unsignedPowExact` | 25 |

**Exponenciales y logarítmicas:** `exp`, `expm1`, `log`, `log10`, `log1p`, `pow`, `sqrt`, `cbrt`, `hypot`.

**Trigonométricas:** `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `sinh`, `cosh`, `tanh`, `toDegrees`, `toRadians`.

**IEEE 754 de bajo nivel:** `fma` (9), `ulp`, `nextUp`, `nextDown`, `nextAfter`, `scalb`, `getExponent`, `IEEEremainder`.

**Aleatorio:** `random()`.

---

# Parte VI — Redondear y formatear de verdad

## 35. El redondeo casero que todo el mundo escribe y por qué falla

Tarde o temprano necesitás redondear a N decimales, mirás la API de `Math` y no encontrás nada. La solución "obvia" que aparece en todas partes es multiplicar, redondear y dividir:

```java
static double redondearMal(double valor, int decimales) {
    double factor = Math.pow(10, decimales);
    return Math.round(valor * factor) / factor;
}
```

Funciona en las pruebas rápidas y falla en producción:

```java
System.out.println(redondearMal(260.775, 2));    // 260.77   <- se esperaba 260.78
System.out.println(redondearMal(1000.0, 17));    // 92.23372036854776   <- ???
```

El primer fallo es de representación: `260.775` no existe como `double`; lo que hay guardado es un poquito **menos** de 260.775, así que redondear "hacia arriba en el empate" no aplica, porque no hay empate. El segundo es de desbordamiento: `Math.pow(10, 17)` multiplicado por 1000 supera el rango de `long`, `Math.round` lo satura en `Long.MAX_VALUE` y la división produce un número absurdo.

Conclusión: **no se puede redondear a N decimales de forma fiable operando sobre `double`.** Hay que salir del tipo. Las dos salidas correctas:

```java
// 1. BigDecimal, cuando necesitás el valor redondeado para seguir calculando
static double redondear(double valor, int decimales) {
    return new BigDecimal(Double.toString(valor))
            .setScale(decimales, RoundingMode.HALF_UP)
            .doubleValue();
}
System.out.println(redondear(260.775, 2));   // 260.78

// 2. Formateo, cuando solo necesitás mostrarlo
System.out.println(String.format(Locale.US, "%.2f", 260.775));   // 260.78
```

Fijate en el detalle de `Double.toString(valor)` dentro del constructor: es lo que convierte el `double` a su representación decimal **corta** ("260.775") antes de que `BigDecimal` lo interprete. Pasar el `double` directamente al constructor haría lo contrario: capturar el valor binario exacto con todos sus decimales de ruido ([sección 41](#41-construir-un-bigdecimal-sin-arruinarlo)).

## 36. RoundingMode: las ocho estrategias

`java.math.RoundingMode` define las políticas disponibles para `BigDecimal` y `DecimalFormat`:

| Modo | Qué hace | 2.5 → | -2.5 → |
|---|---|---|---|
| `UP` | se aleja del cero | 3 | -3 |
| `DOWN` | se acerca al cero (trunca) | 2 | -2 |
| `CEILING` | hacia más infinito | 3 | -2 |
| `FLOOR` | hacia menos infinito | 2 | -3 |
| `HALF_UP` | empates se alejan del cero | 3 | -3 |
| `HALF_DOWN` | empates se acercan al cero | 2 | -2 |
| `HALF_EVEN` | empates al par (*banker's rounding*) | 2 | -2 |
| `UNNECESSARY` | asegura que no hace falta redondear; si hace falta, lanza `ArithmeticException` | — | — |

Cuáles usar de verdad:

- **`HALF_UP`** es lo que la gente entiende por "redondear normal" y lo que suelen exigir las normas de facturación y los cálculos de impuestos en Latinoamérica y España.
- **`HALF_EVEN`** es el modo por defecto de IEEE 754 y de `DecimalFormat`, y el que exige la normativa contable de varios países (y el estándar para tipos de cambio en banca) porque no sesga los totales al alza. Es también el modo por defecto de `MathContext.DECIMAL128`.
- **`UNNECESSARY`** es una herramienta de diseño infravalorada: la usás cuando *sabés* que la operación debe ser exacta y querés que el programa reviente si algún día deja de serlo. Es el equivalente aritmético de una aserción.
- **`FLOOR` / `CEILING`** aparecen en dominios concretos: intereses a favor del cliente, cálculo de comisiones, unidades de embalaje.

**La regla de oro para dinero:** el modo de redondeo no es una decisión técnica, es una **decisión de negocio**. Debe estar escrito en algún sitio del dominio, no elegido al azar en cada clase de servicio. Un sistema donde unas partes usan `HALF_UP` y otras `HALF_EVEN` produce descuadres de céntimos entre informes que nadie sabe explicar.

## 37. Formatear: String.format, DecimalFormat y el locale

Redondear y formatear son cosas distintas. **Formatear es convertir a texto para mostrar**, y ahí entra un factor que rompe sistemas: el **locale**.

```java
System.out.println(Locale.getDefault());                              // es_CR
System.out.println(String.format("%.2f", 1234.5));                    // 1234,50
System.out.println(String.format(Locale.US, "%.2f", 1234.5));         // 1234.50
System.out.println(String.format(Locale.GERMANY, "%,.2f", 1234.5));   // 1.234,50
```

El mismo código produce `1234,50` o `1234.50` **según la configuración regional de la máquina donde corre**. Este es un bug real y frecuentísimo:

- Un CSV generado en un servidor con locale español usa coma decimal, y el sistema que lo consume interpreta el separador de campos mal.
- Un JSON construido con `String.format` produce `{"precio": 1234,50}`, que no es JSON válido.
- Un test pasa en la máquina del desarrollador (locale inglés) y falla en el CI (locale del contenedor).

**La regla es explícita:** siempre que el texto vaya a ser **leído por una máquina** (JSON, CSV, XML, logs estructurados, claves de caché, firmas), pasá `Locale.ROOT` o `Locale.US`. Cuando el texto vaya a ser **leído por una persona**, usá el locale del usuario, que no tiene por qué ser el del servidor.

```java
// Para máquinas
String json = String.format(Locale.ROOT, "{\"precio\": %.2f}", precio);

// Para personas
NumberFormat formato = NumberFormat.getCurrencyInstance(localeDelUsuario);
String visible = formato.format(precio);
```

`NumberFormat.getCurrencyInstance` es mejor que construir el símbolo a mano: coloca el símbolo de moneda donde corresponda en cada idioma, aplica los decimales correctos para esa moneda y usa los separadores locales.

## 38. Dos APIs de la JDK que redondean el mismo número de forma distinta

Este es el matiz que casi ninguna documentación menciona y que produce discusiones larguísimas en los equipos. Mismo número, dos APIs de la biblioteca estándar, dos resultados:

```java
System.out.println(String.format(Locale.US, "%.2f", 2.675));   // 2.68
System.out.println(new DecimalFormat("#.##").format(2.675));   // 2,67
```

La causa está en qué considera cada uno "el número":

```java
System.out.println(new BigDecimal(2.675));
// 2.67499999999999982236431605997495353221893310546875
```

El `double` que llamamos `2.675` es en realidad un poco **menor** que 2.675.

- **`DecimalFormat`** usa `HALF_EVEN` por defecto y opera sobre ese valor binario real. Como es menor que el punto medio, redondea hacia abajo: **2.67**.
- **`String.format`/`Formatter`** trabaja sobre la representación decimal **corta** del `double` (la que produce `Double.toString`, es decir `"2.675"`) y le aplica HALF_UP: **2.68**.

Ninguno está roto; tienen contratos distintos. Pero significa que **cambiar de una API de formateo a otra puede alterar los importes de tus facturas en un céntimo**, y que un informe generado con `DecimalFormat` puede no cuadrar con otro generado con `String.format` sobre los mismos datos.

Lo mismo pasa al construir el `BigDecimal` de una forma u otra:

```java
System.out.println(new BigDecimal("2.675").setScale(2, RoundingMode.HALF_UP));   // 2.68
System.out.println(new BigDecimal(2.675).setScale(2, RoundingMode.HALF_UP));     // 2.67
```

**La única defensa real es no llegar a esta situación:** si el valor es dinero, no debe haber pasado nunca por un `double`. Si nace como `BigDecimal` desde un `String` o desde un entero de centavos, `2.675` es exactamente 2.675 y todas las APIs coinciden.

---

# Parte VII — Dinero y precisión decimal

## 39. Por qué double no sirve para dinero

Resumiendo lo demostrado hasta aquí, con ejemplos que son literalmente operaciones de un carrito de compra:

```java
System.out.println(2.00 - 1.10);   // 0.8999999999999999
System.out.println(1.10 + 2.20);   // 3.3000000000000003
System.out.println(4.35 * 100);    // 434.99999999999994

double saldo = 0;
for (int i = 0; i < 10; i++) saldo += 0.1;
System.out.println(saldo);         // 0.9999999999999999
System.out.println(saldo == 1.0);  // false
```

Los tres problemas concretos que esto genera en un sistema real:

1. **Totales que no cuadran.** La suma de las líneas no coincide con el total calculado aparte, y el descuadre aparece en el céntimo.
2. **Comparaciones que fallan.** `if (saldo == 0)` no detecta un saldo que debería ser cero exacto y quedó en `1e-17`. La cuenta "no se cierra" nunca.
3. **Redondeos inconsistentes.** El mismo importe se muestra `19.99` en una pantalla y `19.98` en otra, según por qué API de formateo haya pasado.

La regla profesional, sin matices: **el dinero no se guarda en `double` ni en `float`, jamás.** Ni en la base de datos (nada de `FLOAT` o `DOUBLE`: `DECIMAL(19,4)` o entero), ni en el DTO, ni en el JSON de la API, ni en la variable intermedia. Basta con que el valor pase una sola vez por un `double` para que el daño esté hecho.

Hay dos representaciones correctas: `BigDecimal` y enteros de la unidad mínima. Vamos con las dos.

## 40. BigDecimal: modelo mental

Un `BigDecimal` es **un entero arbitrariamente grande más una escala**:

```
valor = unscaledValue × 10^(-scale)
```

`3.14` se guarda como `unscaledValue = 314`, `scale = 2`. No hay fracciones binarias en ninguna parte, así que **cualquier decimal que puedas escribir se representa exactamente**.

```java
BigDecimal bd = new BigDecimal("-12345.6789");
System.out.println(bd.precision());     // 9    dígitos significativos
System.out.println(bd.scale());         // 4    decimales
System.out.println(bd.signum());        // -1
System.out.println(bd.unscaledValue()); // -123456789
```

Tres propiedades estructurales que hay que interiorizar:

**1. Es inmutable.** Todos los métodos devuelven un objeto nuevo. El error de principiante es ignorar el retorno:

```java
BigDecimal total = new BigDecimal("100.00");
total.add(new BigDecimal("50.00"));            // no hace nada visible
System.out.println(total);                     // 100.00

total = total.add(new BigDecimal("50.00"));    // así sí
System.out.println(total);                     // 150.00
```

**2. No hay operadores.** `+`, `-`, `*`, `/`, `<` y `>` no funcionan; hay que usar `add`, `subtract`, `multiply`, `divide` y `compareTo`. Esto hace el código más verboso, y es el precio de la exactitud.

**3. La escala se propaga por reglas propias.** Multiplicar suma escalas:

```java
BigDecimal iva = new BigDecimal("100.00").multiply(new BigDecimal("0.21"));
System.out.println(iva);           // 21.0000
System.out.println(iva.scale());   // 4        <- 2 + 2
```

El resultado es correcto pero tiene cuatro decimales. Si eso se guarda o se muestra tal cual, verás `21.0000 €`. Por eso, en cálculos de dinero, **cada operación que produce un importe final debe terminar con un `setScale` explícito**:

```java
System.out.println(iva.setScale(2, RoundingMode.HALF_UP));   // 21.00
```

## 41. Construir un BigDecimal sin arruinarlo

Aquí está el error más común con esta clase, y es fatal porque **destruye la exactitud justo en el momento en que decidiste usar `BigDecimal` para tenerla**:

```java
System.out.println(new BigDecimal(0.1));
// 0.1000000000000000055511151231257827021181583404541015625

System.out.println(new BigDecimal("0.1"));   // 0.1
System.out.println(BigDecimal.valueOf(0.1)); // 0.1
```

`new BigDecimal(double)` captura el valor binario **exacto** del `double`, con toda su basura. No está mal implementado: hace exactamente lo que dice, y el problema es que el `double` que le pasaste ya era impreciso.

**Las tres formas correctas de construirlo:**

```java
new BigDecimal("19.99");                 // desde String: la mejor, exacta y explícita
BigDecimal.valueOf(19.99);               // pasa por Double.toString: da 19.99
BigDecimal.valueOf(1999, 2);             // desde unscaled + escala: 19.99, sin pasar por double
```

`BigDecimal.valueOf(double)` funciona porque internamente hace `new BigDecimal(Double.toString(val))`, es decir, usa la representación decimal corta. Es aceptable, pero si el valor ya venía de un cálculo con `double`, el daño puede ser anterior.

Constantes útiles: `BigDecimal.ZERO`, `ONE`, `TEN`. Usalas en vez de `new BigDecimal("0")`.

**En la práctica**, el valor debería entrar al sistema como texto o como entero y no tocar `double` nunca:

```java
// De JSON/formulario a dominio, sin double intermedio
BigDecimal precio = new BigDecimal(request.getPrecioComoString());

// De base de datos: JDBC devuelve BigDecimal para columnas DECIMAL/NUMERIC
BigDecimal precioDb = resultSet.getBigDecimal("precio");
```

Un aviso sobre Jackson: por defecto deserializa números decimales de JSON a `double`. Si el DTO declara `BigDecimal`, Jackson lo respeta, pero si en algún punto hay un `Object` o un `Map<String, Object>`, el valor pasa por `double` y se corrompe. Se corrige activando `DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS`.

## 42. equals contra compareTo

`BigDecimal` tiene dos nociones de igualdad y usar la equivocada es un bug clásico:

```java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("1.00");

System.out.println(a.equals(b));       // false   <- compara valor Y escala
System.out.println(a.compareTo(b));    // 0       <- compara solo el valor
System.out.println(a.scale() + " vs " + b.scale());   // 1 vs 2
```

`equals` considera que `1.0` y `1.00` son objetos distintos porque tienen escalas distintas. Matemáticamente son el mismo número; para `BigDecimal` no son el mismo objeto.

**Consecuencias prácticas:**

- **Para comparar importes, usá siempre `compareTo`:** `if (total.compareTo(BigDecimal.ZERO) == 0)`, nunca `total.equals(BigDecimal.ZERO)`, porque un total que resultó `0.00` no es `equals` a `ZERO` (que tiene escala 0).
- **Cuidado con `BigDecimal` como clave de `HashMap` o dentro de un `Set`:** `hashCode` es coherente con `equals`, así que `1.0` y `1.00` van a cubetas distintas y `contains` falla.
- **Si necesitás igualdad por valor en colecciones**, normalizá antes con `stripTrailingZeros()` o fijando una escala común con `setScale`.

```java
System.out.println(new BigDecimal("1.00").stripTrailingZeros());   // 1
System.out.println(new BigDecimal("100.0").stripTrailingZeros());  // 1E+2   <- notación científica
System.out.println(new BigDecimal("100.0").stripTrailingZeros().toPlainString()); // 100
```

Ese `1E+2` sorprende: `stripTrailingZeros` puede producir escalas negativas, y `toString()` entonces usa notación científica. Para mostrar al usuario, **usá siempre `toPlainString()`**.

## 43. Dividir con BigDecimal

La división es la operación que puede no terminar, y `BigDecimal` no adivina qué querés:

```java
new BigDecimal("1").divide(new BigDecimal("3"));
// java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
```

Como 1/3 tiene infinitos decimales y `BigDecimal` es exacto por diseño, no puede devolver un resultado. La solución es **decir siempre cuántos decimales querés y con qué política**:

```java
System.out.println(new BigDecimal("1").divide(new BigDecimal("3"), 4, RoundingMode.HALF_UP));   // 0.3333
```

O con un `MathContext`, que fija dígitos significativos en vez de decimales:

```java
System.out.println(new BigDecimal("1").divide(new BigDecimal("3"), MathContext.DECIMAL64));
// 0.3333333333333333
```

**Regla:** en código de dinero, `divide` sin `RoundingMode` es un bug esperando a que aparezca el divisor equivocado. Que los tests pasen solo significa que ningún caso probado dio decimal periódico.

**El problema del reparto.** Dividir dinero tiene una dificultad de negocio que ninguna API resuelve sola:

```java
BigDecimal parte = new BigDecimal("100.00").divide(new BigDecimal("3"), 2, RoundingMode.HALF_UP);
System.out.println(parte);                                  // 33.33
System.out.println(parte.multiply(new BigDecimal("3")));    // 99.99   <- falta un céntimo
```

Repartir 100 € entre tres personas produce un céntimo huérfano. Ninguna configuración de redondeo lo evita: es aritmética. Hay que decidir **explícitamente** qué se hace con él (se lo lleva el primero, el último, el de mayor importe, o se acumula en una cuenta de ajuste). El patrón robusto es calcular n-1 partes y obtener la última por diferencia:

```java
BigDecimal total = new BigDecimal("100.00");
int n = 3;
BigDecimal base = total.divide(BigDecimal.valueOf(n), 2, RoundingMode.DOWN);
BigDecimal ultima = total.subtract(base.multiply(BigDecimal.valueOf(n - 1)));
// 33.33, 33.33, 33.34  -> suma exactamente 100.00
```

## 44. La alternativa: enteros de centavos

La otra representación correcta es guardar la **unidad mínima indivisible** en un `long`: centavos, milésimas de moneda, satoshis.

```java
long precioCentavos = 1999;          // 19.99
long total = precioCentavos * 3;     // 5997
System.out.printf("%d.%02d%n", total / 100, total % 100);   // 59.97
```

**Ventajas:** exactitud total, aritmética rápida (un `long` es un registro de CPU), sin objetos ni GC, comparaciones con `==` y ordenación trivial. Es lo que usan por dentro los sistemas de pagos de alto volumen y las APIs de Stripe o PayPal, donde los importes viajan como enteros.

**Desventajas:** hay que llevar la escala en la cabeza (¿este `long` son centavos o milésimas?), los porcentajes y divisiones vuelven a necesitar cuidado, y el rango de `long` (unos 9.2 × 10^18) limita a ~92 billones de euros expresados en centavos —suficiente salvo para hiperinflación o criptomonedas con 18 decimales.

**La forma profesional de aplicarlo** es no dejar el `long` suelto, sino encapsularlo en un tipo de dominio:

```java
public record Dinero(long centavos, Currency moneda) {
    public Dinero {
        Objects.requireNonNull(moneda);
    }

    public Dinero mas(Dinero otro) {
        if (!moneda.equals(otro.moneda)) {
            throw new IllegalArgumentException("No se pueden sumar " + moneda + " y " + otro.moneda);
        }
        return new Dinero(Math.addExact(centavos, otro.centavos), moneda);
    }

    public Dinero por(int cantidad) {
        return new Dinero(Math.multiplyExact(centavos, cantidad), moneda);
    }

    @Override
    public String toString() {
        return String.format(Locale.ROOT, "%d.%02d %s", centavos / 100, Math.abs(centavos % 100), moneda);
    }
}
```

Fijate en las tres decisiones de diseño: es inmutable, valida que no se sumen monedas distintas (un error real y caro), y usa `Math.addExact`/`multiplyExact` para que un desbordamiento sea una excepción y no un saldo negativo.

**Cuál elegir:** `BigDecimal` si hay porcentajes, tipos de cambio, intereses o escalas variables, o si el equipo ya lo usa. Enteros si el dominio es de pagos simples y de alto volumen. Lo que no es una opción es `double`.

## 45. BigInteger

`BigInteger` es a los enteros lo que `BigDecimal` a los decimales: precisión arbitraria, sin límite de rango, inmutable, sin operadores.

```java
BigInteger factorial = BigInteger.ONE;
for (int i = 1; i <= 50; i++) {
    factorial = factorial.multiply(BigInteger.valueOf(i));
}
System.out.println(factorial);
// 30414093201713378043612608166064768844377641568960512000000000000
```

Usos reales: criptografía (RSA trabaja con enteros de miles de bits), combinatoria, identificadores de precisión arbitraria, y cualquier cálculo donde `long` se quede corto. Tiene utilidades específicas muy útiles: `modPow`, `gcd`, `isProbablePrime`, `sqrt`.

Para exponenciación entera exacta es la respuesta correcta al problema de `Math.pow` que vimos:

```java
System.out.println(BigInteger.TEN.pow(23));   // 100000000000000000000000   exacto
System.out.println((long) Math.pow(10, 23));  // 9223372036854775807        basura
```

## 46. Cuánto cuesta cada opción

Medición propia en JDK 25, 20 millones de operaciones, con calentamiento previo (sin JMH, así que tomalo como orden de magnitud, no como número absoluto):

```
long       20000000 sumas:    5,4 ms
double     20000000 sumas:   14,1 ms
BigDecimal 20000000 sumas:   57,0 ms   (~11x más lento que long)
```

Lo que hay que sacar de estos números:

1. **`BigDecimal` es más lento, pero no catastróficamente.** El factor de ~10x asusta escrito así; en términos absolutos son **57 milisegundos para veinte millones de sumas**. Una aplicación de negocio real hace, quizá, unos cientos de operaciones aritméticas por petición. El coste es literalmente invisible frente a una consulta a base de datos o una llamada HTTP.
2. **Optimizar dinero a `double` para ganar velocidad es una mala decisión casi siempre.** Estás cambiando exactitud —que es el requisito— por microsegundos que nadie va a notar.
3. **Donde `BigDecimal` sí duele es en bucles masivos**: agregaciones sobre millones de filas, cálculos de riesgo, simulaciones. Ahí la alternativa correcta no es `double`, son los enteros de centavos, que dan exactitud a velocidad de primitivo.
4. **`double` fue más lento que `long`** en esta prueba, lo que ilustra que "punto flotante es rápido" tampoco es automático.

---

# Parte VIII — Aleatoriedad

## 47. Math.random y qué hay debajo

`Math.random()` devuelve un `double` en el rango **[0.0, 1.0)**: cero incluido, uno excluido.

```java
double d = Math.random();
```

El patrón habitual para obtener un entero en un rango:

```java
int dado = (int) (Math.random() * 6) + 1;        // 1..6
int porcentaje = (int) (Math.random() * 101);    // 0..100
```

La lógica: multiplicar por N da `[0, N)`, castear a `int` trunca hacia abajo y da los enteros `0..N-1`, y sumar el mínimo desplaza el rango.

Lo que casi ningún tutorial explica es **qué hay debajo**: `Math.random()` crea, en la primera llamada, una única instancia compartida de `java.util.Random` y la reutiliza para siempre. Eso implica dos cosas:

1. Es **thread-safe**, pero mediante sincronización sobre una semilla compartida, lo que causa contención cuando muchos hilos la usan a la vez ([sección 50](#50-threadlocalrandom-y-concurrencia)).
2. **No se puede sembrar.** Si necesitás reproducibilidad, `Math.random()` no sirve.

Para un script, un ejemplo o un juego de consola está perfecto. En código de servidor concurrente, hay opciones mejores.

## 48. Random, semillas y reproducibilidad

`java.util.Random` da control y una API más rica:

```java
Random random = new Random();
int i = random.nextInt(100);        // 0..99
int j = random.nextInt(1, 7);       // 1..6      (Java 17+)
double d = random.nextDouble();     // [0.0, 1.0)
boolean b = random.nextBoolean();
long l = random.nextLong();
double g = random.nextGaussian();   // distribución normal, media 0, desviación 1
```

**`nextInt(bound)` es el método correcto** para obtener un entero en un rango: internamente descarta los valores que producirían sesgo, cosa que multiplicar o usar `%` no hace ([sección 49](#49-el-anti-patrón-del-módulo-en-aleatorios)).

**Con semilla, la secuencia es totalmente reproducible:**

```java
Random a = new Random(42);
Random b = new Random(42);
// a.nextInt(100) x5 -> 30 63 48 84 70
// b.nextInt(100) x5 -> 30 63 48 84 70   idéntica
```

Esto es una **funcionalidad**, no un defecto: es lo que hace posible escribir tests deterministas sobre código que usa aleatoriedad, reproducir una simulación exacta o depurar un caso que falló. El patrón profesional es inyectar el generador:

```java
public class Sorteo {
    private final RandomGenerator random;

    public Sorteo(RandomGenerator random) {   // en producción, el por defecto; en tests, con semilla
        this.random = random;
    }
    ...
}
```

Y el reverso de la moneda, que es un problema de seguridad real: **`Random` es predecible**. Su algoritmo es una congruencia lineal de 48 bits; observando unos pocos valores se puede reconstruir el estado interno y predecir todos los siguientes. Por eso **nunca se usa `Random` ni `Math.random()` para tokens de sesión, contraseñas temporales, códigos de recuperación, identificadores de pago ni nada con implicación de seguridad**. Para eso está `SecureRandom` ([sección 51](#51-randomgenerator-y-securerandom)).

## 49. El anti-patrón del módulo en aleatorios

El patrón que hay que aprender a reconocer en revisiones de código:

```java
int valor = Math.abs(random.nextInt()) % n;   // MAL, por dos motivos distintos
```

**Motivo 1: puede dar negativo.** Ya lo vimos: si `nextInt()` devuelve `Integer.MIN_VALUE`, `Math.abs` lo devuelve tal cual y el `%` conserva el signo.

```java
System.out.println(Math.abs(Integer.MIN_VALUE) % 3);   // -2
```

**Motivo 2: sesga la distribución.** El operador `%` reparte 2^32 valores posibles entre `n` cubetas. Si `n` no divide exactamente ese total, las primeras cubetas reciben un valor más que las demás. Con `n` pequeño el sesgo es despreciable, pero crece con `n`. Medido sobre 5 millones de muestras con `n = 1 500 000 000`:

```
zona sesgada [0, 647483647)
proporción observada con %       = 60,3 %
proporción esperada uniforme     = 43,2 %
proporción con nextInt(bound)    = 43,1 %
```

Los valores de la zona baja aparecen un **40 % más a menudo** de lo que deberían. Si eso es un sorteo, un reparto de recursos o una selección de servidor en un balanceador, la distribución está rota.

**La solución es usar el método que ya resuelve ambos problemas:**

```java
int valor = random.nextInt(n);                      // 0..n-1, uniforme
int enRango = random.nextInt(min, max);             // Java 17+, [min, max)
int conThreadLocal = ThreadLocalRandom.current().nextInt(1, 7);
```

`nextInt(bound)` implementa **rechazo**: descarta internamente los valores de la zona sesgada y vuelve a tirar. Cuesta un poco más en el caso raro y garantiza uniformidad.

## 50. ThreadLocalRandom y concurrencia

`Math.random()` y una instancia compartida de `Random` son thread-safe, pero lo consiguen con una actualización atómica (CAS) sobre una semilla común. Cuando muchos hilos piden números a la vez, todos compiten por la misma variable: los que pierden el CAS reintentan, y el rendimiento se degrada.

`java.util.concurrent.ThreadLocalRandom` resuelve el problema dándole a cada hilo su propio generador y su propia semilla, eliminando la contención por completo:

```java
int valor = ThreadLocalRandom.current().nextInt(1, 7);
double d = ThreadLocalRandom.current().nextDouble();
```

Dos reglas de uso:

1. **Nunca guardes la instancia en un campo compartido.** `ThreadLocalRandom.current()` debe llamarse **en el hilo que va a usarlo**, cada vez. Guardarla en un campo estático y usarla desde varios hilos anula la garantía.
2. **No se puede sembrar de forma útil.** `setSeed` lanza `UnsupportedOperationException`. Si necesitás reproducibilidad, usá `Random` o `SplittableRandom` con semilla.

**Cuándo usar cada uno:**

| Situación | Herramienta |
|---|---|
| Script, ejemplo, código de un solo hilo | `Math.random()` o `Random` |
| Necesitás reproducibilidad (tests, simulaciones) | `Random` con semilla |
| Código de servidor concurrente | `ThreadLocalRandom` |
| Streams paralelos, fork/join | `SplittableRandom` o `ThreadLocalRandom` |
| Seguridad (tokens, claves, códigos) | `SecureRandom` |

## 51. RandomGenerator y SecureRandom

Java 17 (JEP 356) reorganizó toda esta familia bajo la interfaz **`java.util.random.RandomGenerator`**, que ahora implementan `Random`, `ThreadLocalRandom`, `SplittableRandom` y una colección de algoritmos modernos.

```java
RandomGenerator generador = RandomGenerator.getDefault();
System.out.println(generador.getClass().getName());
// jdk.internal.random.L32X64MixRandom
```

Los algoritmos disponibles en JDK 25 se pueden listar en tiempo de ejecución:

```java
RandomGeneratorFactory.all().map(RandomGeneratorFactory::name).sorted().toList();
// [L128X1024MixRandom, L128X128MixRandom, L128X256MixRandom, L32X64MixRandom,
//  L64X1024MixRandom, L64X128MixRandom, L64X128StarStarRandom, L64X256MixRandom,
//  Random, SecureRandom, SplittableRandom, Xoroshiro128PlusPlus, Xoshiro256PlusPlus]
```

Se elige uno por nombre:

```java
RandomGenerator rng = RandomGenerator.of("Xoshiro256PlusPlus");
```

Estos generadores modernos tienen mejor calidad estadística y mejores propiedades para paralelismo que el `Random` original de 1995, que se mantiene solo por compatibilidad. **Para código nuevo, declarar las dependencias como `RandomGenerator`** (la interfaz) en vez de `Random` (la clase concreta) permite cambiar el algoritmo sin tocar el código y facilita los tests.

**`SecureRandom`** es un caso aparte: genera números criptográficamente seguros, alimentándose de la entropía del sistema operativo.

```java
SecureRandom seguro = new SecureRandom();
byte[] token = new byte[32];
seguro.nextBytes(token);
String tokenBase64 = Base64.getUrlEncoder().withoutPadding().encodeToString(token);
```

Es más lento que los demás y no debe usarse para simulaciones masivas, pero es **obligatorio** para todo lo que tenga implicaciones de seguridad. Regla simple: si un atacante ganaría algo prediciendo el número, es `SecureRandom`.

---

# Parte IX — Cierre

## 52. Casos de uso reales

**Caso 1: calcular el total de un pedido con IVA.**

```java
public record LineaPedido(String sku, BigDecimal precioUnitario, int cantidad) {
    public BigDecimal subtotal() {
        return precioUnitario.multiply(BigDecimal.valueOf(cantidad));
    }
}

public BigDecimal calcularTotal(List<LineaPedido> lineas, BigDecimal tasaIva) {
    BigDecimal base = lineas.stream()
            .map(LineaPedido::subtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);

    BigDecimal iva = base.multiply(tasaIva).setScale(2, RoundingMode.HALF_UP);
    return base.setScale(2, RoundingMode.HALF_UP).add(iva);
}
```

Puntos clave: `BigDecimal` de principio a fin, `reduce` con `BigDecimal.ZERO` como identidad, y `setScale` explícito **solo al final de cada importe presentable**, no en cada paso intermedio (redondear en cada línea introduce error acumulado).

**Caso 2: paginación.**

```java
public record Pagina<T>(List<T> contenido, int numeroPagina, int totalPaginas) {}

public <T> Pagina<T> paginar(List<T> todos, int numeroPagina, int tamano) {
    if (tamano <= 0) throw new IllegalArgumentException("tamaño debe ser positivo: " + tamano);

    int totalPaginas = Math.ceilDiv(todos.size(), tamano);       // Java 18+
    int paginaSegura = Math.clamp(numeroPagina, 0, Math.max(0, totalPaginas - 1));  // Java 21+

    int desde = paginaSegura * tamano;
    int hasta = Math.min(desde + tamano, todos.size());

    return new Pagina<>(todos.subList(desde, hasta), paginaSegura, totalPaginas);
}
```

`ceilDiv` en vez del truco `(n + size - 1) / size`, y `clamp` para que un número de página fuera de rango no reviente.

**Caso 3: repartir carga entre servidores por hash.**

```java
public Servidor elegirServidor(String claveSesion, List<Servidor> servidores) {
    int indice = Math.floorMod(claveSesion.hashCode(), servidores.size());
    return servidores.get(indice);
}
```

Una línea, y evita el bug de `Math.abs` con `Integer.MIN_VALUE` y el de `%` con hashes negativos.

**Caso 4: barra de progreso.**

```java
public int porcentajeCompletado(long procesados, long total) {
    if (total <= 0) return 0;                                  // evita la división por cero
    long porcentaje = Math.round(100.0 * procesados / total);  // multiplica antes de dividir
    return (int) Math.clamp(porcentaje, 0, 100);               // nunca fuera de [0, 100]
}
```

**Caso 5: cálculo de distancia geográfica (fórmula del haversine).**

```java
private static final double RADIO_TIERRA_KM = 6371.0;

public static double distanciaKm(double lat1, double lon1, double lat2, double lon2) {
    double dLat = Math.toRadians(lat2 - lat1);
    double dLon = Math.toRadians(lon2 - lon1);

    double a = Math.pow(Math.sin(dLat / 2), 2)
             + Math.cos(Math.toRadians(lat1)) * Math.cos(Math.toRadians(lat2))
             * Math.pow(Math.sin(dLon / 2), 2);

    return RADIO_TIERRA_KM * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}
```

Ejemplo típico de por qué existe la trigonometría en `Math`: conversión explícita a radianes, `atan2` en vez de `atan`, y una constante nombrada en vez de un 6371 suelto.

**Caso 6: reintentos con backoff exponencial.**

```java
public long calcularEsperaMs(int intento, long baseMs, long topeMs) {
    long espera = baseMs * (1L << Math.min(intento, 20));       // duplicar por intento, con tope al shift
    long conJitter = espera / 2 + ThreadLocalRandom.current().nextLong(espera / 2 + 1);
    return Math.min(conJitter, topeMs);
}
```

Tres decisiones aritméticas: `1L <<` (no `1 <<`, que desbordaría a los 31 intentos), un tope al exponente antes de desplazar, y *jitter* aleatorio para que mil clientes que fallaron a la vez no reintenten sincronizados.

## 53. Anti-patrones

**1. Escribir el literal sin `L` en cálculos de tiempo o tamaño.**

```java
// MAL
long ms = 1000 * 60 * 60 * 24 * 365;      // 1471228928
// BIEN
long ms2 = 1000L * 60 * 60 * 24 * 365;    // 31536000000
// MEJOR
Duration año = Duration.ofDays(365);
```

**2. Dividir enteros esperando decimales.**

```java
// MAL
double ratio = aciertos / total;
// BIEN
double ratio2 = (double) aciertos / total;
```

**3. Usar `double` para dinero.** Cualquier variable, campo, columna o DTO monetario en `double` o `float` es un bug latente.

**4. `Math.abs(x) % n` para índices.**

```java
// MAL
int i = Math.abs(clave.hashCode()) % n;
// BIEN
int i2 = Math.floorMod(clave.hashCode(), n);
```

**5. Comparar decimales con `==`.**

```java
// MAL
if (total == 0.0) { ... }
// BIEN (double)
if (Math.abs(total) < TOLERANCIA) { ... }
// MEJOR (dinero)
if (total.compareTo(BigDecimal.ZERO) == 0) { ... }
```

**6. `BigDecimal.equals` para comparar importes.** Devuelve `false` para `1.0` y `1.00`. Usá `compareTo`.

**7. `new BigDecimal(double)`.** Captura la basura binaria. Usá el constructor de `String` o `valueOf`.

**8. `divide` sin `RoundingMode`.** Funciona hasta que aparece un divisor que produce decimal periódico, y entonces lanza `ArithmeticException` en producción.

**9. Ignorar el retorno de un método de `BigDecimal`.** Es inmutable: `total.add(x);` no hace nada.

**10. Bucles controlados por un contador `double`.**

```java
// MAL: puede no terminar
for (double d = 0; d != 1.0; d += 0.1) { ... }
// BIEN
for (int i = 0; i < 10; i++) { double d = i / 10.0; ... }
```

**11. `Math.round(x * 100) / 100.0` para redondear a dos decimales.** Falla con valores como 260.775 y desborda con exponentes grandes.

**12. `String.format("%.2f", x)` sin locale para salida que consume una máquina.** Produce coma decimal en medio mundo.

**13. `Math.abs(random.nextInt()) % n` para sortear.** Sesga la distribución y puede dar negativo. Usá `nextInt(n)`.

**14. `Random` o `Math.random()` para tokens de seguridad.** Es predecible por diseño. Usá `SecureRandom`.

**15. `i = i++`.** No hace nada. Y `a[i++] = i++` hace algo, pero nadie sabe qué sin leer la JLS.

**16. Usar `float` "para ahorrar memoria" en una variable suelta.** El ahorro es de 4 bytes y el coste es perder la mitad de los dígitos significativos.

**17. `strictfp` en código nuevo.** No hace nada desde Java 17.

**18. Recalcular `Math.pow(10, n)` dentro de un bucle** para escalar decimales, en vez de usar `BigDecimal.setScale` o `movePointRight`.

## 54. Checklist y tabla de decisión

**Antes de dar por terminado un cálculo, revisá:**

- [ ] ¿Algún literal `int` participa en una multiplicación cuyo resultado puede pasar de dos mil millones? → añadir `L`.
- [ ] ¿Hay una división entre enteros donde esperás decimales? → castear **antes** de dividir.
- [ ] ¿El valor es dinero? → `BigDecimal` o `long` de centavos, nunca `double`.
- [ ] ¿Hay un `%` cuyo operando izquierdo puede ser negativo? → `Math.floorMod`.
- [ ] ¿Hay un `Math.abs` sobre un valor que podría ser `MIN_VALUE`? → `floorMod`, máscara o `absExact`.
- [ ] ¿Se compara algún `double` con `==` o `!=`? → tolerancia, `Double.compare` o cambiar de tipo.
- [ ] ¿El divisor puede ser cero? → validar (con enteros lanza, con decimales contamina).
- [ ] ¿Hay un `BigDecimal.divide` sin `RoundingMode`? → añadirlo.
- [ ] ¿Se compara `BigDecimal` con `equals`? → `compareTo`.
- [ ] ¿Algún `String.format` numérico va a un fichero, JSON o log estructurado? → `Locale.ROOT`.
- [ ] ¿El redondeo elegido (`HALF_UP` / `HALF_EVEN`) es el que exige el negocio? → escribirlo en el código y documentarlo.
- [ ] ¿El aleatorio tiene implicaciones de seguridad? → `SecureRandom`.
- [ ] ¿El aleatorio se usa desde varios hilos? → `ThreadLocalRandom`.

**Tabla de decisión: qué tipo uso**

| Necesito representar | Tipo | Por qué |
|---|---|---|
| Contadores, índices, IDs pequeños | `int` | rápido y suficiente hasta 2 × 10^9 |
| Tiempos en ms, bytes, contadores grandes | `long` | `int` desborda antes de lo que crees |
| Dinero con porcentajes o tipos de cambio | `BigDecimal` | exacto en decimal, escala controlada |
| Dinero en sistemas de alto volumen | `long` de centavos | exacto y a velocidad de primitivo |
| Medidas físicas, estadística, gráficos, ML | `double` | rango y velocidad, precisión suficiente |
| Millones de valores donde la memoria manda | `float` | la mitad de espacio, la mitad de dígitos |
| Enteros sin límite (cripto, combinatoria) | `BigInteger` | precisión arbitraria |

**Tabla de decisión: qué operación uso**

| Quiero | Uso | No uso |
|---|---|---|
| Dividir redondeando hacia abajo | `Math.floorDiv` | `/` con negativos |
| Dividir redondeando hacia arriba | `Math.ceilDiv` (18+) | `(n+d-1)/d` |
| Índice a partir de un hash | `Math.floorMod` | `Math.abs(h) % n` |
| Detectar desbordamiento | `Math.addExact` y familia | comprobaciones manuales |
| Acotar un valor a un rango | `Math.clamp` (21+) | `max(min, min(max, v))` |
| Redondear a N decimales | `BigDecimal.setScale` | `Math.round(x*100)/100.0` |
| Redondear al par (contabilidad) | `RoundingMode.HALF_EVEN` | `Math.round` |
| Entero aleatorio en un rango | `nextInt(min, max)` | `Math.abs(nextInt()) % n` |
| Potencia entera exacta | `BigInteger.pow` / `Math.powExact` (25+) | `(long) Math.pow(...)` |
| Hipotenusa sin desbordar | `Math.hypot` | `sqrt(x*x + y*y)` |
| Resultados idénticos entre plataformas | `StrictMath` | `strictfp` |

## 55. Fuentes

**Documentación oficial**

- [`java.lang.Math` — JDK 25 API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Math.html) — la referencia definitiva. Los "special cases" de cada método (qué pasa con `NaN`, infinitos y ceros con signo) están documentados uno por uno y casi nadie los lee.
- [`java.math.BigDecimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html) — incluye la explicación de la excepción `Non-terminating decimal expansion`.
- [JEP 306: Restore Always-Strict Floating-Point Semantics](https://openjdk.org/jeps/306) — por qué `strictfp` dejó de tener sentido en Java 17.
- [JDK-8010430: `Math.round` has surprising behavior for odd values of ulp 1](https://bugs.openjdk.org/browse/JDK-8010430) — el bug real de `Math.round` corregido en Java 8.
- [NUM00-J. Detect or prevent integer overflow — CERT Secure Coding](https://wiki.sei.cmu.edu/confluence/display/java/NUM00-J.+Detect+or+prevent+integer+overflow) — el overflow entero tratado como problema de seguridad, no de estilo.
- [Concurrent Random Numbers — The Java Tutorials](https://docs.oracle.com/javase/tutorial/essential/concurrency/threadlocalrandom.html) — por qué existe `ThreadLocalRandom`.

**Las tres fuentes de referencia de este capítulo, y dónde se equivocan**

- [Jenkov — Java Math Operators and Math Class](https://jenkov.com/tutorials/java/math-operators-and-math-class.html). Buena introducción a los operadores y a la aritmética entera frente a la de punto flotante. **Errores detectados:** presenta `Math.floorDiv(-100, 9)` como `-12.0` y `-100 / 9` como `-11.0`, con decimales, cuando ambas devuelven `int` (`-12` y `-11`); y presenta `Math.round(23.445)` como `23.0`, cuando `Math.round(double)` devuelve un `long` (`23`). Verificado en JDK 25. Además, describe la clase `Math` como si terminara en Java 8: no menciona `floorMod`, la familia `*Exact`, `clamp`, `ceilDiv` ni `fma`.
- [W3Schools — Java Math](https://www.w3schools.com/java/java_math.asp) y su [referencia de métodos](https://www.w3schools.com/java/java_ref_math.asp). La tabla de referencia es cómoda, pero está **incompleta y desactualizada**: omite `clamp` (21), `ceilDiv`, `ceilMod`, `divideExact`, `floorDivExact`, `ceilDivExact` (18), `absExact` (15), `fma`, `multiplyFull`, `multiplyHigh` (9), `powExact` (25) y la constante `TAU` (19). Describe `Math.random()` como "un número aleatorio entre 0 y 1" sin aclarar que el 1 está excluido, y `round(x)` sin indicar que el tipo de retorno cambia (`int` para `float`, `long` para `double`), que es justo lo que rompe la compilación cuando lo usás.
- [Baeldung — A Guide to the Java Math Class](https://www.baeldung.com/java-lang-math), más los artículos de su serie: [`Math.pow`](https://www.baeldung.com/java-math-pow), [redondear a N decimales](https://www.baeldung.com/java-round-decimal-number), [`BigDecimal` y `BigInteger`](https://www.baeldung.com/java-bigdecimal-biginteger), [overflow y underflow](https://www.baeldung.com/java-overflow-underflow). El recorrido por método es el más completo de los tres y el artículo de redondeo es honesto: muestra el método casero y explica por qué falla. **Matiz importante:** en la sección de `ulp` presenta `Math.ulp(1)` como `1.1920929E-7` sin explicar por qué, lo que induce a pensar que esa es la resolución de un `double`. No lo es: `Math.ulp(1.0)` vale `2.220446049250313E-16`. La diferencia es que el literal `1` es un `int` y Java resuelve la sobrecarga hacia `ulp(float)`. El dato publicado es correcto pero la conclusión que se saca de él, sin la explicación, es falsa. Igual que las otras dos fuentes, la guía está congelada en Java 8.

**Discusiones y artículos de la comunidad** (consultados para contrastar los casos límite)

- [TIL: `Math.abs` in Java is not always non-negative](https://rusche.me/blog/java-math-abs-negative/) y el [issue de Akka #25034](https://github.com/akka/akka/issues/25034) — el bug de `Math.abs(Integer.MIN_VALUE)` en código de producción real.
- [Java binary search overflow: the bug that hid for 9 years](https://onebug.dev/bugs/java-binary-search-integer-overflow) — la historia del `(low + high) / 2` dentro de la propia JDK.
- [Comparing Doubles in Java — Baeldung](https://www.baeldung.com/java-comparing-doubles) y [Why You Should Never Use `==` to Compare Floats and Doubles](https://ankurm.com/why-you-should-never-use-to-compare-floats-and-doubles-in-java/) — el problema del epsilon absoluto frente al relativo.
- [Modulo of Negative Numbers](https://torstencurdt.com/tech/modulo-of-negative-numbers/) — la distinción entre *remainder* y *modulo* explicada en varios lenguajes.
- [`ThreadLocalRandom` vs `Random`](https://java-performance.info/java-util-random-java-util-concurrent-threadlocalrandom-multithreaded-environments/) — la contención por CAS sobre la semilla compartida, medida.
- [Java Compound Assignment Operators: What Could Go Wrong?](https://amydegregorio.com/2020/02/24/java-compound-assignment-operators-what-could-go-wrong/) — el cast implícito de `+=` con ejemplos.
- [BigDecimal and "Non-terminating decimal expansion"](https://jaydeepm.wordpress.com/2009/06/04/bigdecimal-and-non-terminating-decimal-expansion-error/) — el error más frecuente al dividir importes.

**Nota sobre la verificación.** Todos los outputs de este documento se obtuvieron ejecutando el código en Temurin JDK 25.0.3 (`java Archivo.java`), y los volcados de bytecode con `javap -c`. La lista de métodos de la [sección 34](#34-referencia-completa-de-javalangmath-en-jdk-25) se generó con `javap java.lang.Math` sobre ese mismo JDK. Las cifras de rendimiento de las secciones 27 y 46 son mediciones propias con calentamiento previo, pero **sin JMH**: sirven como orden de magnitud, no como benchmark riguroso.

## 56. Autoevaluación

**1. ¿Qué imprime `System.out.println(1000 * 60 * 60 * 24 * 365);` y por qué?**

<details><summary>Respuesta</summary>

Imprime `1471228928`. Los cinco operandos son literales `int`, así que toda la multiplicación se hace en aritmética de 32 bits y desborda silenciosamente. El resultado correcto (`31536000000`) supera `Integer.MAX_VALUE`. Se arregla marcando el primer literal como `long`: `1000L * 60 * 60 * 24 * 365`. Asignar el resultado a un `long` no ayuda, porque la conversión ocurre después del desbordamiento.
</details>

**2. ¿Por qué `Math.abs(Integer.MIN_VALUE)` devuelve un número negativo?**

<details><summary>Respuesta</summary>

Porque el rango de `int` es asimétrico: va de `-2147483648` a `2147483647`, así que el valor absoluto de `Integer.MIN_VALUE` no existe como `int`. La implementación es `(a < 0) ? -a : a`, y la negación desborda hasta volver al mismo valor. Está documentado en el javadoc. Desde Java 15 existe `Math.absExact`, que lanza `ArithmeticException` en ese caso en vez de devolver un negativo.
</details>

**3. ¿Qué diferencia hay entre `-10 % 3` y `Math.floorMod(-10, 3)`?**

<details><summary>Respuesta</summary>

`-10 % 3` da `-1`: el operador `%` calcula el **resto** de una división truncada hacia cero, y el signo del resultado es el del dividendo. `Math.floorMod(-10, 3)` da `2`: calcula el **módulo** matemático, basado en división redondeada hacia abajo, y el signo del resultado es el del divisor. Para calcular índices a partir de valores que pueden ser negativos (como un `hashCode`), hay que usar `floorMod`.
</details>

**4. ¿Por qué `0.1 + 0.2 != 0.3`?**

<details><summary>Respuesta</summary>

Porque `double` guarda fracciones **binarias**, y ni 0.1 ni 0.2 ni 0.3 tienen representación binaria finita. Lo que se guarda es el `double` más cercano a cada uno; los errores se suman y el resultado (`0.30000000000000004`) cae en un `double` distinto del más cercano a 0.3. El valor exacto que guarda Java al escribir `0.1` es `0.1000000000000000055511151231257827021181583404541015625`.
</details>

**5. ¿Cuánto vale `i` después de `int i = 5; i = i++;`?**

<details><summary>Respuesta</summary>

Vale `5`. El bytecode lo explica: `iload_0` copia el valor actual (5) a la pila, `iinc 0, 1` incrementa la variable a 6, y `istore_0` guarda en la variable lo que había en la pila, que era 5, pisando el 6. El incremento ocurrió y fue sobrescrito. Con `i = ++i` sí quedaría en 6, pero ninguna de las dos formas debe escribirse.
</details>

**6. ¿Por qué `byte b = 10; b += 300;` compila, pero `b = b + 300;` no?**

<details><summary>Respuesta</summary>

Porque la JLS define `E1 op= E2` como `E1 = (T)((E1) op (E2))`, es decir, la asignación compuesta **inserta un cast implícito** al tipo de la izquierda. `b + 300` produce un `int`, que no se puede asignar a un `byte` sin cast explícito; con `+=` el compilador pone el cast por vos. El resultado es `54`, porque 310 truncado a 8 bits da 54. En el bytecode se ve la instrucción `i2b`.
</details>

**7. ¿Cuál es la diferencia entre `Math.round`, `Math.rint`, `Math.floor` y `Math.ceil`?**

<details><summary>Respuesta</summary>

`floor` redondea hacia menos infinito y `ceil` hacia más infinito; ambos devuelven `double`. `round` redondea al entero más cercano resolviendo los empates hacia más infinito (`floor(x + 0.5)`), y devuelve `long` para un `double` o `int` para un `float`. `rint` redondea al más cercano resolviendo los empates **al par** (banker's rounding) y devuelve `double`. La diferencia se ve con negativos y con empates: `Math.round(-2.5)` da `-2`, `Math.rint(2.5)` da `2.0` y `Math.rint(3.5)` da `4.0`.
</details>

**8. ¿Por qué `new BigDecimal(0.1)` no vale `0.1`?**

<details><summary>Respuesta</summary>

Porque ese constructor captura el valor binario **exacto** del `double` que se le pasa, y ese `double` no es un décimo exacto: es `0.1000000000000000055511151231257827021181583404541015625`. El daño ya estaba hecho antes de llamar al constructor. Hay que construirlo desde un `String` (`new BigDecimal("0.1")`) o con `BigDecimal.valueOf(0.1)`, que internamente usa `Double.toString` y produce `0.1`.
</details>

**9. ¿Por qué `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` es `false`?**

<details><summary>Respuesta</summary>

Porque `equals` en `BigDecimal` compara **valor y escala**: `1.0` tiene escala 1 y `1.00` escala 2, así que son objetos distintos aunque representen el mismo número. `compareTo` compara solo el valor numérico y devuelve `0`. Para comparar importes hay que usar `compareTo`; y hay que tener cuidado al usar `BigDecimal` como clave de `HashMap`, porque `hashCode` es coherente con `equals`.
</details>

**10. ¿Qué está mal en `int bucket = Math.abs(clave.hashCode()) % n;`?**

<details><summary>Respuesta</summary>

Dos cosas. Primero, si `hashCode()` devuelve `Integer.MIN_VALUE`, `Math.abs` devuelve ese mismo valor negativo y el `%` conserva el signo, produciendo un índice negativo y una `ArrayIndexOutOfBoundsException` intermitente e irreproducible. Segundo, en el caso general de aleatorios, el `%` sesga la distribución cuando el rango no divide exactamente al total. La forma correcta es `Math.floorMod(clave.hashCode(), n)`.
</details>

**11. ¿Por qué `1 / 0` lanza excepción pero `1.0 / 0` no?**

<details><summary>Respuesta</summary>

Porque son dos aritméticas distintas. La entera no tiene forma de representar "infinito", así que la JVM lanza `ArithmeticException: / by zero`. La de punto flotante sigue IEEE 754, que **define** los resultados: `Infinity`, `-Infinity` y `NaN` para `0.0/0.0`. El peligro es que ese valor no detiene el programa: se propaga por los cálculos siguientes y aparece mucho después, ya sin contexto.
</details>

**12. Un compañero propone usar `float` en vez de `double` para los importes "porque ocupa la mitad". ¿Qué le respondés?**

<details><summary>Respuesta</summary>

Que el problema no es el tamaño sino el tipo: **ninguno de los dos sirve para dinero**, porque ambos son binarios y no pueden representar exactamente valores como 0.10. Además `float` es peor: tiene 7 dígitos significativos y deja de poder contar de uno en uno a partir de 16 777 216, así que un importe grande en centavos ya pierde precisión. La respuesta correcta es `BigDecimal` o un `long` de centavos encapsulado en un tipo de dominio.
</details>

**13. ¿Qué imprime este bucle y por qué es peligroso?**

```java
for (double d = 0.0; d != 1.0; d += 0.1) { System.out.println(d); }
```

<details><summary>Respuesta</summary>

Es un bucle infinito. Sumar `0.1` diez veces da `0.9999999999999999`, no `1.0`, así que la condición `d != 1.0` nunca se vuelve falsa en el punto esperado y el bucle sigue indefinidamente. Los bucles se controlan con enteros y el valor decimal se deriva dentro: `for (int i = 0; i < 10; i++) { double d = i / 10.0; ... }`.
</details>

**14. ¿Cuándo usarías `StrictMath` en vez de `Math`?**

<details><summary>Respuesta</summary>

Cuando necesitás que el resultado sea **idéntico bit a bit** en cualquier plataforma, JVM o versión: simulaciones distribuidas que deben converger, lógica de juego replicada entre clientes, sistemas de auditoría que recalculan y comparan, tests numéricos exactos entre entornos. `Math` permite hasta 1-2 ulps de error en las funciones trascendentes a cambio de usar instrucciones del procesador. Las operaciones básicas (`+`, `-`, `*`, `/`, `sqrt`) son reproducibles siempre. Y `strictfp` no es la respuesta: desde Java 17 no hace nada.
</details>

**15. ¿Por qué `String.format("%.2f", 2.675)` y `new DecimalFormat("#.##").format(2.675)` dan resultados distintos?**

<details><summary>Respuesta</summary>

Porque operan sobre "el número" de forma distinta. El `double` llamado `2.675` es en realidad `2.674999999...`. `DecimalFormat` usa `HALF_EVEN` sobre ese valor binario real y redondea a `2.67`. `String.format` trabaja sobre la representación decimal corta (`"2.675"`, la de `Double.toString`) y le aplica HALF_UP, dando `2.68`. Ninguno está roto: tienen contratos distintos. La lección es que un importe que pasó por `double` ya perdió la información necesaria para redondearlo de forma inequívoca, y por eso el dinero no debe pasar nunca por `double`. (Además, `String.format` sin `Locale` cambia el separador decimal según la máquina.)
</details>

**16. ¿Qué ventaja tiene `Math.ceilDiv(n, size)` sobre `(n + size - 1) / size` para calcular páginas?**

<details><summary>Respuesta</summary>

Que no puede desbordar. `n + size - 1` desborda si `n` está cerca de `Integer.MAX_VALUE`, y el resultado sale negativo sin ningún aviso. La variante `(int) Math.ceil((double) n / size)` tiene otro problema: pasa por `double` y pierde exactitud por encima de 2^53. `Math.ceilDiv` (Java 18+) no tiene ninguno de los dos defectos y además expresa la intención en el nombre.
</details>

**17. ¿Por qué `Math.sin(Math.PI)` no devuelve exactamente 0?**

<details><summary>Respuesta</summary>

Porque `Math.PI` no es π: es el `double` más cercano a π, que difiere en aproximadamente 1.2 × 10^-16. El seno de ese número, calculado correctamente, es `1.2246467991473532E-16`, que es justo esa diferencia. El resultado no está mal: es el seno correcto del número que realmente le pasaste. Por eso los resultados trigonométricos nunca se comparan con `== 0`, sino con una tolerancia.
</details>

**18. Tenés que repartir 100,00 € entre tres socios. ¿Cómo lo hacés para que la suma cuadre?**

<details><summary>Respuesta</summary>

`100.00 / 3` con dos decimales da `33.33`, y `33.33 × 3` es `99.99`: sobra un céntimo que ninguna configuración de redondeo hace desaparecer. La solución es de negocio, no técnica: calcular n-1 partes redondeadas y obtener la última **por diferencia**, de modo que absorba el resto (`33.33`, `33.33`, `33.34`). Y hay que decidir y documentar explícitamente quién se lleva el céntimo sobrante.
</details>

**19. ¿Por qué `DoubleStream.sum()` da un resultado más preciso que sumar en un bucle?**

<details><summary>Respuesta</summary>

Porque la implementación de la JDK aplica **suma compensada de Kahan**: arrastra en una variable auxiliar el error de redondeo de cada paso y lo reinyecta en el siguiente. Sumar `0.1` diez mil veces en un bucle da `1000.0000000001588`; con `DoubleStream.sum()` da `1000.0`. La especificación no *garantiza* mayor precisión, solo permite que la haya, pero la implementación de referencia compensa.
</details>

**20. ¿Cuándo usarías `ThreadLocalRandom` en vez de `Math.random()`?**

<details><summary>Respuesta</summary>

En código concurrente. `Math.random()` usa una única instancia compartida de `Random` cuya semilla se actualiza con CAS: cuando muchos hilos compiten, los que pierden reintentan y el rendimiento cae. `ThreadLocalRandom` da a cada hilo su propio generador y elimina la contención. Hay que llamar a `ThreadLocalRandom.current()` dentro del hilo que lo usa, nunca guardar la instancia en un campo compartido. Y si hace falta reproducibilidad, ninguno de los dos sirve: se usa `Random` con semilla; si hace falta seguridad, `SecureRandom`.
</details>
