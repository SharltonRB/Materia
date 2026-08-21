# Logical, Relational and Bitwise Operators

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** El capítulo anterior ([Math Operations](07-math-operations.md)) cubrió los operadores que producen **números**: sumar, restar, multiplicar, dividir, redondear. Este cubre los que producen **decisiones**: comparar dos valores, combinar varias condiciones y manipular los bits de un entero uno por uno. Es la otra mitad de la tabla de operadores, y la que decide por dónde va el programa.

La aritmética falla en silencio devolviendo un número equivocado. Los operadores de este capítulo fallan de una forma distinta y peor: **devuelven la decisión equivocada**. El programa no calcula mal, hace otra cosa. Deja entrar a quien no debía, rechaza a quien sí, ordena mal una lista, descarta un paquete válido.

La lista de bugs concretos que salen de este capítulo, todos reales:

- Un login que autentica a cualquiera porque alguien comparó dos contraseñas con `==` en vez de `equals`.
- Un carrito que funciona con 3 artículos en desarrollo y falla con 300 en producción, porque `Integer a == b` es `true` hasta 127 y `false` a partir de 128.
- El mismo carrito devolviendo resultados **distintos en dos servidores idénticos**, porque uno arrancó con un flag de JVM que mueve ese límite.
- Una validación de rango que acepta un valor inválido sin quejarse, porque `NaN` no es mayor ni menor que nada.
- Un `NullPointerException` en una línea que no tiene ni un solo punto: solo un `?` y un `:`.
- Un `sort` que devuelve la lista desordenada sin lanzar ninguna excepción, porque el comparador restaba en vez de usar `compare`.
- Una comprobación de permisos que le dice "no tienes acceso" a un usuario que sí lo tiene, porque comparó la máscara con `== 1`.
- Un parser de protocolo binario que lee el byte `0xFF` como `-1` y descarta el paquete.
- Un `NullPointerException` intermitente porque en `a != null & a.isEmpty()` faltaba un `&`.
- Un token de sesión comparado con `equals`, que filtra el token correcto carácter a carácter midiendo cuánto tarda el servidor en responder.

Vamos a cubrir el modelo completo: qué devuelve exactamente cada operador y de qué tipo, la diferencia entre comparar valores y comparar referencias, qué es el cortocircuito y por qué es semántica y no una optimización, cómo se comporta el ternario cuando sus dos ramas tienen tipos distintos, y qué significa de verdad operar sobre los 32 bits de un `int`.

**Sobre la verificación.** Todos los outputs, mensajes de excepción, errores del compilador y volcados de bytecode de este documento se ejecutaron realmente en un JDK 25. Como en el capítulo anterior, importa: las tres fuentes de referencia usadas para prepararlo **contienen errores factuales en este tema concreto**, y uno de ellos —"el operador de complemento equivale al operador de negación lógica"— ni siquiera compila. Están señalados uno por uno en la [sección 48](#48-fuentes), con el mensaje real de `javac` al lado.

---

## Índice

**Parte I — Comparar: los operadores relacionales**

1. [Qué devuelve un operador relacional y por qué boolean no es un número](#1-qué-devuelve-un-operador-relacional-y-por-qué-boolean-no-es-un-número)
2. [Los cuatro operadores de orden](#2-los-cuatro-operadores-de-orden)
3. [Igualdad y desigualdad sobre primitivos](#3-igualdad-y-desigualdad-sobre-primitivos)
4. [Cuando el compilador te salva: tipos incomparables](#4-cuando-el-compilador-te-salva-tipos-incomparables)
5. [Comparar decimales: NaN y el cero negativo](#5-comparar-decimales-nan-y-el-cero-negativo)
6. [Comparar referencias: identidad frente a igualdad](#6-comparar-referencias-identidad-frente-a-igualdad)
7. [La caché de wrappers y el operador que cambia de respuesta según la máquina](#7-la-caché-de-wrappers-y-el-operador-que-cambia-de-respuesta-según-la-máquina)
8. [Cuándo se desempaqueta un wrapper y cuándo no](#8-cuándo-se-desempaqueta-un-wrapper-y-cuándo-no)
9. [El método equals y sus trampas](#9-el-método-equals-y-sus-trampas)
10. [Objects.equals y el caso de los arrays](#10-objectsequals-y-el-caso-de-los-arrays)
11. [Ordenar: compareTo, Comparator y el comparador roto por resta](#11-ordenar-compareto-comparator-y-el-comparador-roto-por-resta)
12. [instanceof y el pattern matching](#12-instanceof-y-el-pattern-matching)

**Parte II — Combinar: los operadores lógicos**

13. [Los tres operadores lógicos y sus tablas de verdad](#13-los-tres-operadores-lógicos-y-sus-tablas-de-verdad)
14. [El cortocircuito visto en el bytecode](#14-el-cortocircuito-visto-en-el-bytecode)
15. [El cortocircuito es semántica, no optimización](#15-el-cortocircuito-es-semántica-no-optimización)
16. [Los operadores lógicos que no cortocircuitan](#16-los-operadores-lógicos-que-no-cortocircuitan)
17. [Cuándo sí quieres el operador que no cortocircuita](#17-cuándo-sí-quieres-el-operador-que-no-cortocircuita)
18. [XOR lógico y su gemelo el distinto de](#18-xor-lógico-y-su-gemelo-el-distinto-de)
19. [Precedencia entre operadores lógicos](#19-precedencia-entre-operadores-lógicos)
20. [Leyes de De Morgan: negar condiciones sin equivocarse](#20-leyes-de-de-morgan-negar-condiciones-sin-equivocarse)
21. [En qué orden poner las condiciones](#21-en-qué-orden-poner-las-condiciones)
22. [La trampa de C que Java solo bloquea a medias](#22-la-trampa-de-c-que-java-solo-bloquea-a-medias)
23. [Redundancias que delatan a quien las escribe](#23-redundancias-que-delatan-a-quien-las-escribe)

**Parte III — El operador ternario**

24. [Sintaxis y cuándo mejora de verdad la legibilidad](#24-sintaxis-y-cuándo-mejora-de-verdad-la-legibilidad)
25. [El tipo del resultado: promoción numérica entre las ramas](#25-el-tipo-del-resultado-promoción-numérica-entre-las-ramas)
26. [El NullPointerException del ternario](#26-el-nullpointerexception-del-ternario)
27. [Ternarios anidados y el límite de lo legible](#27-ternarios-anidados-y-el-límite-de-lo-legible)

**Parte IV — Bitwise: operar bit a bit**

28. [El modelo mental: 32 bits y complemento a dos](#28-el-modelo-mental-32-bits-y-complemento-a-dos)
29. [AND, OR y XOR sobre enteros](#29-and-or-y-xor-sobre-enteros)
30. [El complemento y por qué no es la negación lógica](#30-el-complemento-y-por-qué-no-es-la-negación-lógica)
31. [Promoción: por qué la máscara 0xFF es obligatoria](#31-promoción-por-qué-la-máscara-0xff-es-obligatoria)
32. [Los desplazamientos a fondo](#32-los-desplazamientos-a-fondo)
33. [Los dos mitos de los desplazamientos](#33-los-dos-mitos-de-los-desplazamientos)
34. [La distancia de desplazamiento se toma módulo 32 o 64](#34-la-distancia-de-desplazamiento-se-toma-módulo-32-o-64)
35. [Asignación compuesta bitwise y su cast oculto](#35-asignación-compuesta-bitwise-y-su-cast-oculto)
36. [La trampa de precedencia que no compila y la que sí](#36-la-trampa-de-precedencia-que-no-compila-y-la-que-sí)

**Parte V — Bits en la práctica**

37. [Flags y máscaras: poner, quitar, alternar, consultar](#37-flags-y-máscaras-poner-quitar-alternar-consultar)
38. [El bug de comprobar la máscara comparando con uno](#38-el-bug-de-comprobar-la-máscara-comparando-con-uno)
39. [EnumSet: la alternativa que casi siempre gana](#39-enumset-la-alternativa-que-casi-siempre-gana)
40. [BitSet: cuando 64 bits no alcanzan](#40-bitset-cuando-64-bits-no-alcanzan)
41. [Las utilidades de Integer y Long](#41-las-utilidades-de-integer-y-long)
42. [Aritmética sin signo en un lenguaje sin tipos sin signo](#42-aritmética-sin-signo-en-un-lenguaje-sin-tipos-sin-signo)
43. [XOR: propiedades y usos reales](#43-xor-propiedades-y-usos-reales)
44. [Trucos de bits que valen la pena y los que no](#44-trucos-de-bits-que-valen-la-pena-y-los-que-no)

**Parte VI — Cierre**

45. [Casos de uso reales](#45-casos-de-uso-reales)
46. [Anti-patrones](#46-anti-patrones)
47. [Checklist y tabla de decisión](#47-checklist-y-tabla-de-decisión)
48. [Fuentes](#48-fuentes)
49. [Autoevaluación](#49-autoevaluación)

---

# Parte I — Comparar: los operadores relacionales

## 1. Qué devuelve un operador relacional y por qué boolean no es un número

Un operador relacional toma dos valores y devuelve un `boolean`. Siempre. No devuelve un `int`, no devuelve "algo que se parece a verdadero", no devuelve `0` o `1`.

```java
System.out.println(5 > 3);        // true
System.out.println('a' < 'b');    // true
System.out.println(3 == 3.0);     // true
System.out.println('A' == 65);    // true
```

Los dos últimos merecen una nota. `3 == 3.0` es `true` porque se aplica la **promoción numérica binaria** del capítulo anterior: el `int` se convierte a `double` y se comparan como `double`. Y `'A' == 65` es `true` porque `char` es aritméticamente un entero sin signo de 16 bits, y la letra `A` es el punto de código 65.

La parte que hay que interiorizar viniendo de C, JavaScript o Python es que en Java **`boolean` no es un número y no hay conversión implícita en ninguna dirección**:

```java
int n = 0;
if (n) { }            // no compila
boolean b = true;
boolean r = b == 1;   // no compila
```

El compilador es explícito en los dos casos:

```
error: incompatible types: int cannot be converted to boolean
    if (n = 1) { }
          ^

error: incomparable types: boolean and int
    boolean r = b == 1;
                  ^
```

Esto elimina de golpe una familia entera de bugs que en C son cotidianos: el `if (strcmp(a, b))` invertido, el `while (fgets(...))` que confunde `NULL` con falso, el `if (x = 0)` que asigna en vez de comparar. Java corta casi todos. **Casi**: queda un agujero, y está en la [sección 22](#22-la-trampa-de-c-que-java-solo-bloquea-a-medias).

Un detalle de implementación que explica varias cosas: aunque `boolean` sea un tipo propio del lenguaje, **la JVM no tiene instrucciones de boolean**. En bytecode un `boolean` es un `int` que vale 0 o 1, y se manipula con las mismas instrucciones `iload`, `istore`, `iconst_0`, `iconst_1`. Lo veremos en la [sección 14](#14-el-cortocircuito-visto-en-el-bytecode). La restricción de tipos es del compilador, no de la máquina virtual.

Y un dato menor pero simpático, por si aparece en un volcado de un `hashCode`:

```java
System.out.println(Boolean.hashCode(true));    // 1231
System.out.println(Boolean.hashCode(false));   // 1237
```

Son dos números primos elegidos arbitrariamente y fijados por el javadoc desde entonces. No significan nada.

## 2. Los cuatro operadores de orden

`<`, `>`, `<=` y `>=` comparan **magnitud**, y por eso solo funcionan sobre tipos numéricos primitivos (y sobre sus wrappers, desempaquetándolos primero).

| Operador | Nombre | `5 op 3` | `3 op 5` | `3 op 3` |
|---|---|---|---|---|
| `<` | menor que | `false` | `true` | `false` |
| `>` | mayor que | `true` | `false` | `false` |
| `<=` | menor o igual | `false` | `true` | `true` |
| `>=` | mayor o igual | `true` | `false` | `true` |

Lo que **no** se puede hacer es aplicarlos a objetos. Esto no compila:

```java
String a = "manzana", b = "pera";
if (a < b) { }        // error: bad operand types for binary operator '<'
```

En Java el orden de los objetos no es un operador, es un método: `compareTo`. Volvemos a él en la [sección 11](#11-ordenar-compareto-comparator-y-el-comparador-roto-por-resta). Esta es una diferencia importante con Python o C++, donde `<` está sobrecargado y `"manzana" < "pera"` funciona.

Como los operadores de orden aplican promoción numérica, se pueden mezclar tipos libremente:

```java
byte pequeno = 10;
long grande = 10L;
System.out.println(pequeno <= grande);   // true, ambos promocionados a long
```

Sobre `char` también funcionan, y son la base de las comprobaciones de rango de caracteres escritas a mano:

```java
char c = 'g';
boolean esMinuscula = c >= 'a' && c <= 'z';
System.out.println(esMinuscula);   // true
```

Funciona, pero conviene saber que **solo es correcto para ASCII**. Para texto real existe `Character.isLowerCase(c)`, que conoce Unicode entero y no da falsos negativos con `ñ`, `ü` o el alfabeto griego. La versión a mano se justifica en parsers de formatos que por especificación son ASCII (JSON, HTTP, CSV), no en validación de datos de usuario.

## 3. Igualdad y desigualdad sobre primitivos

`==` y `!=` son los dos únicos operadores relacionales que aceptan también `boolean` y referencias. Sobre primitivos numéricos hacen exactamente lo que parece, con la misma promoción de siempre:

```java
System.out.println(10 == 10);         // true
System.out.println(10 != 10);         // false
System.out.println(10 == 10L);        // true, el int sube a long
System.out.println(0.1 + 0.2 == 0.3); // false  <- esto es del capítulo anterior
```

Ese último es el recordatorio de que `==` sobre decimales es una trampa por razones aritméticas, no por razones de igualdad. Está explicado en [Math Operations](07-math-operations.md#20-por-qué-01-más-02-no-es-03); aquí nos ocupan las trampas que son del propio operador, que son otras dos y están en la sección siguiente.

Sobre `boolean`, `==` y `!=` funcionan y son perfectamente legales:

```java
boolean a = true, b = false;
System.out.println(a == b);   // false
System.out.println(a != b);   // true
```

`a != b` sobre booleanos es, literalmente, un XOR. Volvemos a ello en la [sección 18](#18-xor-lógico-y-su-gemelo-el-distinto-de).

## 4. Cuando el compilador te salva: tipos incomparables

Antes de meternos en los problemas, conviene saber dónde **no** los hay. Java rechaza en compilación las comparaciones que no pueden ser ciertas nunca:

```java
String s = "x";
Integer i = 1;
boolean r = s == i;
```

```
error: incomparable types: String and Integer
    boolean r = s == i;
                  ^
```

La regla de la JLS (§15.21.3) es que para comparar dos referencias con `==` tiene que existir una conversión de casting posible entre sus tipos. `String` e `Integer` son clases finales sin relación, así que ninguna referencia puede ser de ambos tipos a la vez, y el compilador lo corta.

Esto tapa algunos errores, pero **deja pasar todos los que importan**, porque cualquier cosa comparada con `Object`, con una interfaz común o con una clase de la misma jerarquía compila sin problema:

```java
Object o = "hola";
String s = "hola";
System.out.println(o == s);   // compila y da... depende. Ver la sección 6.
```

Es decir: el compilador te protege del error que no ibas a cometer y te deja solo frente al que sí.

## 5. Comparar decimales: NaN y el cero negativo

Hay dos valores de `double` y `float` para los que `==` se comporta de forma que rompe la intuición, y ambos vienen del estándar IEEE 754.

**NaN no es igual a nada, ni siquiera a sí mismo.**

```java
double nan = 0.0 / 0.0;

System.out.println(nan == nan);   // false
System.out.println(nan != nan);   // true
System.out.println(nan < 1.0);    // false
System.out.println(nan > 1.0);    // false
System.out.println(nan <= nan);   // false
```

Las cinco líneas son correctas según la especificación. `NaN` significa "esto no es un número", y comparar magnitudes con algo que no es un número no puede dar `true` en ninguna dirección. La consecuencia es que `x != x` es una prueba válida de NaN, aunque lo idiomático sea `Double.isNaN(x)`.

**El bug que esto produce** no es que una validación rechace un valor bueno, sino lo contrario: que acepte uno malo. Fijate en las dos formas de escribir la misma validación de rango:

```java
static boolean enRangoOk(double v, double min, double max) {
    return v >= min && v <= max;        // NaN -> false -> rechaza. Correcto.
}

static boolean enRangoRoto(double v, double min, double max) {
    return !(v < min || v > max);       // NaN -> ninguna es true -> !false = true. ACEPTA NaN.
}
```

Las dos parecen equivalentes —son la misma expresión aplicando De Morgan— y con cualquier número normal lo son. Con `NaN` la segunda deja pasar el valor. Y `NaN` llega más fácil de lo que parece: cualquier `0.0/0.0`, cualquier `Math.sqrt` de un negativo, cualquier `Double.parseDouble("NaN")`, cualquier división donde el denominador acabó valiendo cero.

**La regla práctica:** las validaciones de rango sobre decimales se escriben en positivo (`v >= min && v <= max`), nunca negando la condición contraria. Y si el valor viene de fuera, se valida `Double.isFinite(v)` antes que nada, que descarta de un golpe `NaN`, `Infinity` y `-Infinity`.

**El cero negativo es igual al cero positivo con `==`, pero no con `equals`.**

```java
System.out.println(0.0 == -0.0);                                       // true
System.out.println(Double.valueOf(0.0).equals(Double.valueOf(-0.0)));  // false
System.out.println(Double.compare(0.0, -0.0));                         // 1
```

Tres respuestas distintas para la misma pregunta. La explicación es que son valores **distintos en memoria** (difieren en el bit de signo) pero **numéricamente iguales**. El operador `==` compara numéricamente; `equals` y `compare` comparan la representación, porque necesitan ser consistentes con `hashCode` y con un orden total.

Que no son intercambiables se nota en cuanto los dividís:

```java
System.out.println(1.0 / 0.0);    //  Infinity
System.out.println(1.0 / -0.0);   // -Infinity
```

Y ahí está el problema de verdad: `-0.0` aparece cuando una multiplicación o una división produce un valor negativo tan pequeño que se redondea a cero, o simplemente al multiplicar cero por un negativo. Si ese valor acaba en un denominador, el signo del infinito resultante cambia el sentido de toda la comparación que venga después.

**La tabla que resume la Parte I hasta aquí:**

| Comparación | `==` | `equals` | `compare` |
|---|---|---|---|
| `NaN` con `NaN` | `false` | `true` | `0` |
| `0.0` con `-0.0` | `true` | `false` | `1` |
| `1.0` con `1.0` | `true` | `true` | `0` |

Ninguna de las tres columnas está mal. Tienen contratos distintos: `==` implementa IEEE 754, `equals` implementa una relación de equivalencia (que exige reflexividad, y por eso `NaN.equals(NaN)` **tiene** que ser `true`), y `compare` implementa un orden total (que exige que todo par de valores sea comparable, y por eso `NaN` tiene que caer en algún sitio: al final).

## 6. Comparar referencias: identidad frente a igualdad

Este es **el** tema de la Parte I y probablemente el error más caro de todo el bloque de básicos.

Cuando los dos operandos de `==` son referencias, el operador no mira el contenido de los objetos. Compara **si las dos referencias apuntan al mismo objeto en memoria**. Se llama comparación de identidad.

```java
String s1 = "hola";
String s2 = "hola";
String s3 = new String("hola");

System.out.println(s1 == s2);        // true
System.out.println(s1 == s3);        // false
System.out.println(s1.equals(s3));   // true
```

Las tres líneas son coherentes en cuanto se ve qué hay debajo. `s1` y `s2` apuntan al **mismo** objeto, porque los literales de cadena idénticos comparten instancia a través del *string pool* (los detalles están en [Strings and Methods](06-strings-and-methods.md)). `s3` es un objeto nuevo con el mismo contenido, así que la identidad falla y la igualdad funciona.

Lo verdaderamente peligroso es cuánto depende el resultado de dónde vino el `String`:

```java
String s1 = "hola";
String s4 = "ho" + "la";          // concatenación de literales
String parte = "ho";
String s5 = parte + "la";         // concatenación en tiempo de ejecución

System.out.println(s1 == s4);            // true   <- el compilador la resolvió en compilación
System.out.println(s1 == s5);            // false  <- objeto nuevo creado en runtime
System.out.println(s1 == s5.intern());   // true   <- intern() lo devuelve al pool
```

`s4` da `true` porque el compilador plegó las dos constantes en una sola (el mismo *constant folding* del capítulo anterior). `s5` da `false` porque la concatenación ocurrió con el programa corriendo.

Traducido a producción: **una comparación con `==` sobre `String` funciona perfectamente en los tests, donde las cadenas son literales, y falla en cuanto la cadena llega de un formulario, de una base de datos, de un JSON o de un fichero de configuración.** Ninguna de esas fuentes produce cadenas internadas.

El bug canónico:

```java
// MAL
if (claveIntroducida == claveGuardada) { autenticar(); }

// BIEN
if (Objects.equals(claveIntroducida, claveGuardada)) { autenticar(); }
```

En bytecode la diferencia es visible de un vistazo. Comparar dos `Integer` con `==`:

```
static boolean igualdadBoxed(java.lang.Integer, java.lang.Integer);
  Code:
       0: aload_0
       1: aload_1
       2: if_acmpne     9      // <-- "if reference compare not equal"
```

`if_acmpne` es la instrucción de comparación **de referencias** (la `a` es de *address*). Cuando uno de los dos operandos es un primitivo, el compilador genera otra cosa completamente distinta:

```
static boolean igualdadUnboxed(java.lang.Integer, int);
  Code:
       0: aload_0
       1: invokevirtual #13    // Method java/lang/Integer.intValue:()I   <-- desempaqueta
       4: iload_1
       5: if_icmpne     12     // <-- "if int compare not equal"
```

El mismo símbolo `==` en el código fuente, dos instrucciones de máquina distintas y dos semánticas distintas. Cuál se genera depende **de los tipos**, no de los valores.

**La regla, sin excepciones útiles:** `==` sobre referencias solo se usa para tres cosas.

1. Comparar con `null` (`if (x == null)`), que es la única forma de hacerlo.
2. Comparar enums (`if (estado == Estado.ACTIVO)`), donde la identidad **es** la igualdad porque cada constante es un singleton.
3. Comprobar identidad a propósito, cuando lo que te importa es si son literalmente el mismo objeto (cachés, detección de ciclos, el `if (this == otro) return true;` al principio de un `equals`).

Para todo lo demás: `equals`, o mejor `Objects.equals`.

## 7. La caché de wrappers y el operador que cambia de respuesta según la máquina

Si la sección anterior es el error más común, esta es la versión que más tarda en encontrarse, porque **funciona en desarrollo**.

```java
Integer a = 127, b = 127;
Integer c = 128, d = 128;

System.out.println(a == b);       // true
System.out.println(c == d);       // false
System.out.println(c.equals(d));  // true
```

Dos líneas idénticas salvo por el valor, dos resultados opuestos. La causa es que el autoboxing no llama a un constructor: llama a `Integer.valueOf(int)`, y ese método mantiene una **caché de instancias precreadas** para el rango `-128..127`. Dentro del rango devuelve siempre el mismo objeto y `==` da `true`; fuera del rango crea uno nuevo cada vez y `==` da `false`.

El javadoc de `valueOf` lo dice con todas las letras: *"This method will always cache values in the range -128 to 127, inclusive, and may cache other values outside of this range."* Ese **"may cache other values"** es lo que convierte el problema en algo mucho peor.

El límite superior es configurable al arrancar la JVM:

```
$ java Cache.java
128 == 128 -> false
1000 == 1000 -> false
-200 == -200 -> false

$ java -XX:AutoBoxCacheMax=1000 Cache.java
128 == 128 -> true
1000 == 1000 -> true
-200 == -200 -> false
```

Leído despacio: **el mismo bytecode, en la misma máquina, con el mismo JDK, da respuestas distintas según un flag de arranque.** Un servicio desplegado en dos nodos con configuraciones de JVM ligeramente distintas puede comportarse de forma diferente en cada uno. Es el tipo de bug que consume semanas.

El límite inferior (`-128`) sí es fijo y no se puede mover, como muestra la línea de `-200`.

La caché no es exclusiva de `Integer`. El estado en JDK 25:

| Tipo | Rango cacheado | Configurable |
|---|---|---|
| `Boolean` | `TRUE` y `FALSE` (solo hay dos) | no |
| `Byte` | todos (`-128..127`) | no |
| `Short` | `-128..127` | no |
| `Character` | `0..127` | no |
| `Integer` | `-128..127`, ampliable hacia arriba | sí, `-XX:AutoBoxCacheMax` |
| `Long` | `-128..127` | no |
| `Float` | ninguno | — |
| `Double` | ninguno | — |

Verificado:

```java
Long l1 = 127L, l2 = 127L;
System.out.println(l1 == l2);        // true

Character ch1 = 127, ch2 = 127;
Character ch3 = 128, ch4 = 128;
System.out.println(ch1 == ch2);      // true
System.out.println(ch3 == ch4);      // false

Boolean bo1 = true, bo2 = true;
System.out.println(bo1 == bo2);      // true

Double d1 = 1.0, d2 = 1.0;
System.out.println(d1 == d2);        // false   <- nunca hay caché de decimales
```

`Double` no cachea nada, ni siquiera valores tan comunes como `0.0` o `1.0`. Tiene sentido: hay 2^64 valores posibles y ninguna manera razonable de elegir cuáles precrear.

**Lo peligroso de este bug es su distribución.** Los tests usan valores pequeños: `1`, `2`, `42`. Todos caen en la caché, todos dan `true`, todo pasa. En producción los identificadores, los importes en centavos y los contadores son grandes, se salen del rango, y la comparación empieza a fallar para la mayoría de los casos pero no para todos. Y como el código "funciona a veces", la sospecha tarda mucho en caer sobre la línea correcta.

**La regla:** nunca uses `==` entre dos wrappers. Si tenés dos `Integer`, `equals` o `Objects.equals`; y si podés, cambiá el tipo a `int` y el problema desaparece de raíz.

## 8. Cuándo se desempaqueta un wrapper y cuándo no

La regla que decide si `==` compara referencias o números es más simple de lo que parece, y conviene tenerla explícita porque de ella salen dos comportamientos opuestos:

> Si **al menos uno** de los dos operandos es de tipo primitivo numérico, el otro se desempaqueta y la comparación es **numérica**. Si **los dos** son referencias, la comparación es **de identidad**.

```java
Integer boxed = 1000;
int prim = 1000;

System.out.println(boxed == prim);   // true  <- hay un primitivo, se desempaqueta
```

`1000` está fuera de la caché, así que `boxed` es un objeto nuevo; y aun así el resultado es `true`, porque al haber un `int` en la comparación se llama a `intValue()` y se comparan los números. Es exactamente el `invokevirtual` + `if_icmpne` que vimos en el bytecode de la sección 6.

La consecuencia incómoda es que **añadir o quitar una `I` mayúscula en la declaración de una variable, sin tocar nada más, cambia el significado de todas las comparaciones en las que participa**. Un refactor tan inocente como cambiar el tipo de un parámetro de `int` a `Integer` para poder pasarle `null` convierte silenciosamente comparaciones numéricas en comparaciones de identidad.

Y el desempaquetado trae su propio riesgo: si la referencia es `null`, revienta.

```java
Integer nulo = null;

System.out.println(nulo == 5);
// NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "<local25>" is null

System.out.println(nulo > 5);
// NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "<local25>" is null

Integer otroNulo = null;
System.out.println(nulo == otroNulo);   // true   <- aquí NO se desempaqueta: dos referencias
```

Tres líneas que ilustran la regla entera. Las dos primeras tienen un primitivo al otro lado, desempaquetan y lanzan NPE. La tercera tiene referencias a ambos lados, compara identidad, y `null == null` es perfectamente válido.

El mensaje de la excepción, por cierto, es un regalo de los *helpful NullPointerExceptions* (JEP 358, Java 14): dice exactamente qué método se intentó invocar y sobre qué variable. Antes de Java 14 este mismo fallo producía un `NullPointerException` pelado sin ninguna pista.

**Dónde muerde esto en código real:** cuando el valor viene de un `Map`.

```java
Map<String, Integer> stock = new HashMap<>();

if (stock.get("sku-123") > 0) { }   // NPE si la clave no existe: get devuelve null
```

`Map.get` devuelve `null` para claves ausentes, y `> 0` desempaqueta. La línea es tan corta y tan obvia que nadie la mira dos veces. Las formas correctas:

```java
if (stock.getOrDefault("sku-123", 0) > 0) { }

Integer cantidad = stock.get("sku-123");
if (cantidad != null && cantidad > 0) { }
```

La segunda funciona gracias al cortocircuito de `&&`, que es el tema de la Parte II.

## 9. El método equals y sus trampas

`equals` es un método normal de `Object`, no un operador, y por defecto hace exactamente lo mismo que `==`:

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

Es decir: **una clase que no sobrescribe `equals` no gana nada por usar `equals` en vez de `==`**. Si comparás dos instancias de una clase tuya que no lo implementa, el resultado es identidad igualmente. Las clases de la JDK que sí lo sobrescriben (`String`, los wrappers, `List`, `Set`, `Map`, `LocalDate`, `BigDecimal`, los `record`) son las que hacen que `equals` signifique lo que uno espera.

Los `record` lo generan automáticamente comparando todos sus componentes, que es una de las razones para preferirlos como tipos de datos.

**La trampa que más sorprende** es que `equals` compara también el tipo:

```java
System.out.println(Integer.valueOf(1).equals(Long.valueOf(1)));         // false
System.out.println(Long.valueOf(1).equals(Integer.valueOf(1)));         // false
System.out.println(Integer.valueOf(1).equals(Short.valueOf((short)1))); // false
```

Los tres son `false`, y los tres son "el número uno". La implementación de `Integer.equals` empieza con `if (obj instanceof Integer)`, así que un `Long` no entra nunca.

Esto revienta en cuanto se mezclan capas: un ID que viaja como `Long` desde JPA, se convierte a `Integer` en un DTO y se compara con el original. `equals` devuelve `false`, la lógica cree que son entidades distintas, y no hay excepción ni warning en ningún punto. La defensa es no mezclar tipos numéricos para el mismo concepto, y si hay que comparar entre tipos, hacerlo sobre el valor: `a.longValue() == b.longValue()`.

**La segunda trampa** es el orden de los operandos con `null`:

```java
String s = null;

System.out.println(s.equals("hola"));    // NullPointerException
System.out.println("hola".equals(s));    // false, sin excepción
```

De ahí sale el idiom de poner la constante a la izquierda (`"ACTIVO".equals(estado)`), llamado a veces *Yoda condition*. Funciona, pero se lee al revés y hoy tiene una alternativa mejor, que es la sección siguiente.

Un recordatorio del capítulo anterior que pertenece también aquí: `BigDecimal.equals` compara **valor y escala**, así que `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` es `false`. Para importes se usa `compareTo(...) == 0`.

## 10. Objects.equals y el caso de los arrays

`java.util.Objects.equals(a, b)` es la forma correcta por defecto de comparar dos referencias que pueden ser `null`. Su implementación completa es:

```java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```

Tres decisiones en una línea: si son el mismo objeto es `true` sin llamar a nada (y eso incluye el caso `null == null`); si el primero es `null` y el segundo no, devuelve `false` sin lanzar; y en el resto delega en `equals`.

```java
System.out.println(Objects.equals(null, null));   // true
System.out.println(Objects.equals(null, "x"));    // false
System.out.println(Objects.equals("x", "x"));     // true
```

Nótese que usa `||` y `&&` con cortocircuito, y que **sin cortocircuito esta línea no funcionaría**: `a.equals(b)` se evaluaría siempre y lanzaría NPE cuando `a` es `null`. Es un buen ejemplo de lo que dice la [sección 15](#15-el-cortocircuito-es-semántica-no-optimización).

**Los arrays son el caso especial que hay que memorizar.** Un array es un objeto, y los arrays **no sobrescriben `equals`**:

```java
int[] a1 = {1, 2, 3};
int[] a2 = {1, 2, 3};

System.out.println(a1 == a2);               // false
System.out.println(a1.equals(a2));          // false  <- identidad heredada de Object
System.out.println(Objects.equals(a1, a2)); // false  <- delega en el equals de arriba
System.out.println(Arrays.equals(a1, a2));  // true   <- la única que compara contenido
```

`Objects.equals` **no** salva este caso, porque termina llamando al `equals` heredado de `Object`. Para arrays hay que usar `Arrays.equals`, o `Arrays.deepEquals` si son multidimensionales o de objetos. Lo mismo aplica a `hashCode` (`Arrays.hashCode`) y a `toString` (`Arrays.toString`), y por eso imprimir un array directamente muestra algo como `[I@1b6d3586`.

Esto tiene una consecuencia que muerde en `record`: un `record` con un componente de tipo array genera un `equals` que compara ese componente por identidad, no por contenido. Dos records con arrays de idéntico contenido son distintos. Si necesitás igualdad por valor, el componente tiene que ser una `List` inmutable, no un array.

## 11. Ordenar: compareTo, Comparator y el comparador roto por resta

Para ordenar objetos Java no usa `<`, usa el método `compareTo` de la interfaz `Comparable`. El contrato es simple: devuelve un negativo si `this` va antes, cero si son equivalentes en el orden, y un positivo si va después. **El valor concreto no significa nada** más allá de su signo.

```java
System.out.println("10".compareTo("9"));                       // -8
System.out.println(Integer.compare(10, 9));                    // 1
System.out.println(Integer.compare(Integer.MIN_VALUE, Integer.MAX_VALUE));  // -1
System.out.println(Boolean.compare(true, false));              // 1
```

El `-8` de las cadenas es la diferencia entre los caracteres `'1'` y `'9'`, y lo único que hay que leer en él es el signo: `"10"` va antes que `"9"` en orden lexicográfico, porque compara carácter a carácter y `'1' < '9'`. Es el motivo por el que ordenar números guardados como texto da `1, 10, 11, 2, 3`.

**El bug del comparador por resta** es el más frecuente de esta sección. La idea de escribir el comparador como una resta es tentadora porque el signo sale solo:

```java
lista.sort((x, y) -> x - y);      // parece correcto
```

Y lo es, hasta que la resta desborda. Del capítulo anterior sabemos que la aritmética entera desborda en silencio, y aquí el resultado es que el comparador **miente sobre el orden**:

```java
System.out.println(Integer.MIN_VALUE - Integer.MAX_VALUE);   // 1   <- ¡positivo!
System.out.println(0 - Integer.MIN_VALUE);                   // -2147483648  <- ¡negativo!
```

Según ese comparador, `Integer.MIN_VALUE` es **mayor** que `Integer.MAX_VALUE`. Y como no hay excepción, la lista simplemente sale mal ordenada:

```java
List<Integer> lista = new ArrayList<>(
        List.of(Integer.MIN_VALUE, 0, Integer.MAX_VALUE, -5, 7));

lista.sort((x, y) -> x - y);
System.out.println(lista);
// [0, 7, 2147483647, -2147483648, -5]        <- desordenada, sin ningún error

lista.sort(Integer::compare);
System.out.println(lista);
// [-2147483648, -5, 0, 7, 2147483647]        <- correcto
```

El primer resultado no lanzó nada. Devolvió una lista que el programa da por ordenada y no lo está. Si esa lista alimenta luego una búsqueda binaria, los resultados son directamente aleatorios.

A veces sí hay excepción, y es peor de diagnosticar: si el comparador es lo bastante incoherente, `TimSort` lo detecta a mitad de camino y lanza `IllegalArgumentException: Comparison method violates its general contract!`, un mensaje famoso por aparecer solo con ciertos tamaños de lista y ciertas distribuciones de datos.

**Las formas correctas**, todas de la JDK:

```java
lista.sort(Integer::compare);                                    // el método estático
lista.sort(Comparator.naturalOrder());                           // explícito
lista.sort(Comparator.comparingInt(Producto::getStock));         // por un campo, sin boxing
lista.sort(Comparator.comparing(Producto::getNombre)
                     .thenComparingInt(Producto::getStock)
                     .reversed());                               // encadenado
```

`Integer.compare` está implementado como `(x < y) ? -1 : ((x == y) ? 0 : 1)`, que no puede desbordar porque no resta nada.

Y una nota sobre `comparingInt` frente a `comparing`: la segunda desempaqueta y vuelve a empaquetar en cada comparación. En una lista grande eso son millones de objetos temporales. Cuando el campo es un primitivo, las variantes `comparingInt`, `comparingLong` y `comparingDouble` son gratis y mejores.

## 12. instanceof y el pattern matching

`instanceof` es un operador relacional —está en la tabla de precedencia junto a `<` y `>`— que devuelve `boolean` y comprueba si un objeto es instancia de un tipo, de una subclase suya o de una interfaz que implementa.

```java
Object txt = "hola";
System.out.println(txt instanceof String);         // true
System.out.println(txt instanceof CharSequence);   // true  <- String implementa CharSequence
System.out.println(txt instanceof Object);         // true
```

**Con `null` siempre devuelve `false`**, sin excepción, sea cual sea el tipo:

```java
Object o = null;
System.out.println(o instanceof String);   // false
```

Esta es una propiedad muy útil: `x instanceof String s` es a la vez una comprobación de tipo y una comprobación de nulidad. Un `if (o instanceof String s)` ya garantiza que `s` no es `null` dentro del bloque.

**El pattern matching** convierte el operador en algo bastante más potente. La forma clásica exigía escribir el tipo tres veces:

```java
// Antes
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Ahora
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

La variable `s` se llama **variable de patrón**, y su alcance es exactamente donde el compilador puede demostrar que la comprobación fue cierta. Eso produce un comportamiento que sorprende la primera vez:

```java
if (!(obj instanceof String s)) {
    return;          // aquí s no existe
}
System.out.println(s.length());   // aquí sí: si no hubiéramos vuelto, obj es un String
```

El alcance no es el bloque, es el conjunto de puntos del programa a los que solo se llega si el patrón encajó. El compilador lo deduce del flujo. Esto encaja perfectamente con el estilo de *early return* y evita el anidamiento.

Y combina de forma natural con el cortocircuito de `&&`, precisamente porque garantiza el orden de evaluación:

```java
if (obj instanceof String s && s.length() > 2) {
    System.out.println(s.toUpperCase());   // HOLA
}
```

Esa línea **solo funciona con `&&`**. Con `&` no compilaría, porque el compilador no puede garantizar que el patrón se haya evaluado antes de usar `s`.

**Sobre la versión de Java, que es donde casi todas las fuentes se equivocan.** El pattern matching para `instanceof` apareció como *preview* en Java 14 (JEP 305), siguió en preview en Java 15 (JEP 375) y se hizo **definitivo en Java 16** (JEP 394). La diferencia no es académica: en Java 14 o 15 el código no compila salvo que se active `--enable-preview`. Es fácil de comprobar:

```
$ javac --release 15 E3.java
error: pattern matching in instanceof is not supported in -source 15
  (use -source 16 or higher to enable pattern matching in instanceof)

$ javac --release 16 E3.java
OK
```

El propio compilador señala Java 16, no Java 14.

**El anti-patrón** que conviene mencionar aquí es la cadena de `instanceof`:

```java
// Huele mal
if (forma instanceof Circulo c) { return Math.PI * c.radio() * c.radio(); }
else if (forma instanceof Rectangulo r) { return r.ancho() * r.alto(); }
else if (forma instanceof Triangulo t) { return t.base() * t.altura() / 2; }
throw new IllegalArgumentException("forma desconocida");
```

Una cadena así suele ser un síntoma de que falta polimorfismo (un método `area()` en cada clase) o, si las variantes son cerradas, de que el modelo pide `sealed interface` más `switch` con patrones. Ambos temas viven en el bloque `02-POO`; aquí basta con reconocer el olor.

---

# Parte II — Combinar: los operadores lógicos

## 13. Los tres operadores lógicos y sus tablas de verdad

Java tiene tres operadores lógicos con cortocircuito. Los dos binarios operan sobre `boolean` y devuelven `boolean`:

| `a` | `b` | `a && b` | `a \|\| b` | `!a` |
|---|---|---|---|---|
| `true` | `true` | `true` | `true` | `false` |
| `true` | `false` | `false` | `true` | `false` |
| `false` | `true` | `false` | `true` | `true` |
| `false` | `false` | `false` | `false` | `true` |

- `&&` (AND) es `true` solo si **ambos** operandos son `true`.
- `||` (OR) es `true` si **al menos uno** lo es.
- `!` (NOT) invierte.

Los tres exigen operandos `boolean`. Pasarles enteros no compila:

```java
int x = 2, y = 4;
int r = x && y;
```

```
error: bad operand types for binary operator '&&'
  first type:  int
  second type: int
```

Esto es otra diferencia deliberada con C y con JavaScript, donde `&&` acepta cualquier cosa y devuelve uno de los operandos en vez de un booleano. En Java `a && b` es siempre `boolean`, nunca "el valor de `b`", y por tanto no existe el idiom `const nombre = input || "por defecto"`. El equivalente en Java es un ternario o un `Optional`.

Lo que los distingue de todo lo demás, y lo que ocupa las cuatro secciones siguientes, es que **no siempre evalúan su segundo operando**.

## 14. El cortocircuito visto en el bytecode

La forma más rápida de convencerse de que el cortocircuito existe es hacerlo visible. Con un método que anuncia cuándo se ejecuta:

```java
static boolean ruidoso(String nombre, boolean valor) {
    System.out.println("    [evaluado: " + nombre + "]");
    return valor;
}
```

Y ejecutando las cuatro combinaciones:

```java
ruidoso("izq=false", false) && ruidoso("der", true);
//     [evaluado: izq=false]
//     resultado: false            <- "der" nunca se evaluó

ruidoso("izq=false", false) & ruidoso("der", true);
//     [evaluado: izq=false]
//     [evaluado: der]
//     resultado: false            <- se evaluaron los dos

ruidoso("izq=true", true) || ruidoso("der", false);
//     [evaluado: izq=true]
//     resultado: true             <- "der" nunca se evaluó

ruidoso("izq=true", true) | ruidoso("der", false);
//     [evaluado: izq=true]
//     [evaluado: der]
//     resultado: true
```

La regla es la que se deduce de la tabla de verdad: en un `&&`, si el operando izquierdo es `false` el resultado ya está decidido, así que el derecho no aporta nada y no se evalúa. En un `||`, si el izquierdo es `true`, lo mismo.

En bytecode la diferencia no es sutil: son dos estrategias completamente distintas.

```
static boolean cortoCircuito(boolean, boolean);   // return a && b;
  Code:
       0: iload_0
       1: ifeq          12      // si a es false, saltar directo al final
       4: iload_1
       5: ifeq          12      // si b es false, saltar al final
       8: iconst_1              // ambos true -> apilar 1
       9: goto          13
      12: iconst_0              // -> apilar 0
      13: ireturn
```

```
static boolean sinCortoCircuito(boolean, boolean);   // return a & b;
  Code:
       0: iload_0
       1: iload_1
       2: iand                  // una sola instrucción, sin saltos
       3: ireturn
```

`&&` compila a **saltos condicionales**. `&` compila a la instrucción aritmética `iand` sobre dos enteros —recordemos que en la JVM un `boolean` es un `int` de 0 o 1—. Es la prueba de que no son el mismo operador con distinta ortografía: son mecanismos distintos.

Lo mismo con OR:

```
static boolean orCorto(boolean, boolean);    // a || b
       1: ifne          8       // si a es true, saltar a "apilar 1"

static boolean orLargo(boolean, boolean);    // a | b
       2: ior
```

Y una curiosidad que confirma que la JVM no tiene booleanos de verdad: la negación tampoco tiene instrucción propia y se compila como un salto.

```
static boolean negar(boolean);   // return !a;
  Code:
       0: iload_0
       1: ifne          8
       4: iconst_1
       5: goto          9
       8: iconst_0
       9: ireturn
```

## 15. El cortocircuito es semántica, no optimización

Es tentador leer el cortocircuito como un truco de rendimiento: "se ahorra evaluar la segunda condición". Esa lectura es la que produce el bug, porque lleva a pensar que `&` y `&&` son intercambiables y que uno es simplemente más rápido.

**No son intercambiables. El cortocircuito forma parte del significado del programa**, y hay expresiones perfectamente correctas con `&&` que son incorrectas con `&`:

```java
String s = null;

System.out.println(s != null && s.length() > 0);   // false
System.out.println(s != null &  s.length() > 0);
// NullPointerException: Cannot invoke "String.length()" because "<local1>" is null
```

La primera versión funciona porque el cortocircuito **garantiza** que `s.length()` no se ejecuta cuando `s` es `null`. No es que "normalmente no se ejecute": está especificado en la JLS §15.23 que no se evalúa. Ese es el mecanismo que sostiene el patrón de guarda más usado del lenguaje:

```java
if (usuario != null && usuario.getPerfil() != null && usuario.getPerfil().esAdmin()) { }

if (lista != null && !lista.isEmpty() && lista.get(0).esValido()) { }

if (indice >= 0 && indice < array.length && array[indice] > 0) { }
```

En las tres, cada condición **habilita** la siguiente. Cambiar un solo `&&` por `&` convierte cualquiera de ellas en un `NullPointerException` o un `ArrayIndexOutOfBoundsException` en cuanto los datos no sean los del caso feliz.

La otra cara de la moneda es que **los efectos secundarios del operando derecho también se pierden**:

```java
int[] contador = {0};
boolean cond = false;

boolean r1 = cond && (contador[0]++ > 0);
System.out.println(contador[0]);   // 0   <- el incremento no ocurrió

boolean r2 = cond & (contador[0]++ > 0);
System.out.println(contador[0]);   // 1   <- sí ocurrió
```

De ahí sale una regla de estilo que vale la pena adoptar: **no pongas efectos secundarios dentro de una expresión booleana**. Si el operando derecho incrementa un contador, escribe en un log, añade a una lista o llama a algo que muta estado, el hecho de que se ejecute o no depende de un valor que muchas veces no controlás. Es un tipo de bug que no se ve leyendo la línea, porque la línea parece una simple condición.

## 16. Los operadores lógicos que no cortocircuitan

`&`, `|` y `^` tienen doble vida. Sobre enteros son operadores de bits (Parte IV). Sobre `boolean` son **operadores lógicos booleanos**, y así los llama la JLS, que les dedica una sección propia (§15.22.2, *Boolean Logical Operators &, ^, and |*).

```java
boolean a = true, b = false;

System.out.println(a & b);   // false
System.out.println(a | b);   // true
System.out.println(a ^ b);   // true
```

Su tabla de verdad para `&` y `|` es idéntica a la de `&&` y `||`. La única diferencia es que **siempre evalúan ambos operandos**.

Esto merece dos matices que las fuentes suelen pasar por alto.

**Primero: no son "los operadores de bits usados sobre booleanos por accidente".** Son operadores distintos definidos por la especificación para el tipo `boolean`, con su propia sección y su propia semántica. Que compartan símbolo con los de bits es una decisión de sintaxis heredada de C.

**Segundo: `^` no tiene versión con cortocircuito, y no puede tenerla.** No existe `^^` en Java. La razón es lógica, no de diseño: un XOR necesita **siempre** los dos operandos para decidir, porque conocer uno solo nunca determina el resultado. Con `a = true`, el resultado de `a ^ b` sigue dependiendo enteramente de `b`. No hay nada que cortocircuitar.

Un tercer punto, ya de rendimiento: **`&` no es más lento que `&&` por evaluar de más; puede ser más rápido.** `&&` genera un salto condicional, y un salto que el predictor del procesador no acierta cuesta bastante más que la instrucción `iand` que ahorra. En una condición muy corta y con resultado impredecible (`a & b` sobre dos booleanos ya calculados), la versión sin cortocircuito puede ganar. Es una micro-optimización que casi nunca vale la pena buscar a propósito, pero explica por qué en código de librerías muy caliente a veces se ve `&` donde uno esperaría `&&`.

## 17. Cuándo sí quieres el operador que no cortocircuita

Hay dos situaciones en las que `&` o `|` sobre booleanos no son un error tipográfico sino la elección correcta, y las dos aparecen en código profesional.

**Caso 1: acumular todos los errores de validación, no solo el primero.**

```java
static boolean validar(String valor, String campo, List<String> errores) {
    if (valor == null || valor.isBlank()) {
        errores.add(campo + " es obligatorio");
        return false;
    }
    return true;
}
```

Con `&&`, en cuanto el primer campo falla los demás ni se comprueban:

```java
List<String> errores = new ArrayList<>();
boolean valido = validar(email, "email", errores)
              && validar(clave, "clave", errores);

System.out.println(errores);   // [email es obligatorio]
```

El usuario corrige el email, reenvía el formulario, y **entonces** descubre que la clave también estaba mal. Con `&`, todas las validaciones se ejecutan y el usuario ve la lista completa de una vez:

```java
boolean valido = validar(email, "email", errores)
               & validar(clave, "clave", errores);

System.out.println(errores);   // [email es obligatorio, clave es obligatoria]
```

Dicho esto, si el código llega a este punto lo honesto es reconocer que **la expresión está haciendo dos cosas** (calcular un booleano y llenar una lista), y que un bucle explícito sobre las validaciones se lee mejor y no depende de que nadie "corrija" el `&` a `&&` en una revisión. El `&` funciona; es frágil.

**Caso 2: comparación en tiempo constante.**

Este es el caso donde `&` y `|` no son una preferencia de estilo sino un requisito de seguridad. Comparar un token, una contraseña hasheada o una firma HMAC con `equals` filtra información: `String.equals` va carácter a carácter y **vuelve en cuanto encuentra una diferencia**. Midiendo el tiempo de respuesta, un atacante puede deducir cuántos caracteres iniciales acertó, y reconstruir el secreto carácter a carácter en tiempo lineal en vez de exponencial.

La defensa es una comparación que tarde siempre lo mismo, y se construye con bits:

```java
static boolean iguales(byte[] x, byte[] y) {
    if (x.length != y.length) return false;
    int diff = 0;
    for (int i = 0; i < x.length; i++) {
        diff |= x[i] ^ y[i];      // acumula diferencias sin ramificar ni salir antes
    }
    return diff == 0;
}
```

`x[i] ^ y[i]` es cero solo si los dos bytes son idénticos. El `|=` acumula: si alguna posición difirió, algún bit de `diff` queda a uno y ya no vuelve a cero. El bucle **recorre siempre el array entero**, sin `return` anticipado y sin ramificación que dependa de los datos.

En la práctica no hace falta escribirlo: la JDK lo trae desde Java 6.

```java
System.out.println(MessageDigest.isEqual(esperado, recibido));   // true
System.out.println(MessageDigest.isEqual(esperado, distinto));   // false
```

`MessageDigest.isEqual` hace exactamente eso. **Cualquier comparación de secretos debe usarlo.** Y esto encaja con la regla general de seguridad del `equals`: para valores públicos, `equals`; para valores secretos, comparación en tiempo constante.

## 18. XOR lógico y su gemelo el distinto de

Sobre booleanos, `^` devuelve `true` cuando los operandos **difieren**:

```java
System.out.println(true ^ true);    // false
System.out.println(true ^ false);   // true
System.out.println(false ^ false);  // false
```

Que es exactamente lo mismo que `!=`:

```java
System.out.println(true != false);   // true
```

Sobre `boolean`, `a ^ b` y `a != b` son equivalentes y producen bytecode idéntico. La preferencia habitual es `!=` cuando lo que expresás es "estos dos estados no coinciden" y `^` cuando pensás en términos de paridad o de alternancia.

Los usos legítimos de un XOR lógico son pocos pero claros: **exactamente uno de los dos**.

```java
// Un pedido lleva dirección de envío o es de recogida en tienda, pero no ambas
boolean configuracionValida = tieneDireccionEnvio ^ esRecogidaEnTienda;

// Una casilla que invierte el orden solo si el usuario lo pide
boolean descendente = ordenNaturalEsDescendente ^ usuarioPidioInvertir;
```

El segundo patrón —usar XOR como "invertir condicionalmente"— es genuinamente útil y difícil de escribir mejor con `if`.

También conviene conocer `Boolean.logicalAnd`, `logicalOr` y `logicalXor`, añadidos en Java 8:

```java
System.out.println(Boolean.logicalXor(true, false));   // true
System.out.println(Boolean.logicalAnd(true, false));   // false
```

No existen para escribirlos a mano —`a ^ b` es mejor— sino para usarlos como *method references* donde hace falta una función:

```java
boolean alguno = flags.stream().reduce(false, Boolean::logicalOr);
```

## 19. Precedencia entre operadores lógicos

El orden entre los operadores de esta parte es donde más gente falla, porque no es el orden en que se escriben ni el orden alfabético. De mayor a menor precedencia:

```
!   >   &   >   ^   >   |   >   &&   >   ||   >   ?:
```

Dos consecuencias que se ven mal a simple vista.

**Primera: `&&` liga más fuerte que `||`.** Es el equivalente lógico de "multiplicar antes que sumar":

```java
boolean a = true, b = false, c = true;

System.out.println(a || b && c);      // true
System.out.println((a || b) && c);    // true
```

Aquí coinciden, pero no porque sean equivalentes: `a || b && c` se agrupa como `a || (b && c)`. Con otros valores divergen, y el resultado depende de un detalle que casi nadie recuerda al leer.

**Segunda, y peor: entre los no cortocircuitados el orden es `&`, luego `^`, luego `|`.** Verificado:

```java
System.out.println(a ^ b | c);     // true
System.out.println(a ^ (b | c));   // false
```

**Dos resultados opuestos con los mismos valores y los mismos símbolos.** `a ^ b | c` se agrupa como `(a ^ b) | c`, porque `^` liga más fuerte que `|`. Si el lector asumió lo contrario —y es fácil, porque en la mayoría de las notaciones matemáticas OR y XOR se presentan al mismo nivel—, entiende el código al revés.

La recomendación no es memorizar la tabla: es **poner paréntesis en cuanto haya más de un tipo de operador lógico en la misma expresión**. Son gratis en ejecución, el compilador solo los usa para construir el árbol, y eliminan la clase entera de errores. La tabla sirve para leer código ajeno, no para escribir el propio.

La tabla completa, incluyendo lo relevante de este capítulo y del anterior:

| Nivel | Operadores | Asociatividad |
|---|---|---|
| 1 | `++` `--` postfijos | — |
| 2 | `++` `--` prefijos, `+` `-` unarios, `~`, `!`, casts | derecha a izquierda |
| 3 | `*` `/` `%` | izquierda a derecha |
| 4 | `+` `-` binarios | izquierda a derecha |
| 5 | `<<` `>>` `>>>` | izquierda a derecha |
| 6 | `<` `>` `<=` `>=` `instanceof` | izquierda a derecha |
| 7 | `==` `!=` | izquierda a derecha |
| 8 | `&` | izquierda a derecha |
| 9 | `^` | izquierda a derecha |
| 10 | `\|` | izquierda a derecha |
| 11 | `&&` | izquierda a derecha |
| 12 | `\|\|` | izquierda a derecha |
| 13 | `? :` | derecha a izquierda |
| 14 | `=` `+=` `-=` `&=` `\|=` `^=` `<<=` `>>=` `>>>=` … | derecha a izquierda |

El detalle importante para la Parte IV es que **los niveles 8, 9 y 10 están por debajo del 7**: los operadores de bits ligan *menos* fuerte que `==`. De ahí sale la trampa de la [sección 36](#36-la-trampa-de-precedencia-que-no-compila-y-la-que-sí).

## 20. Leyes de De Morgan: negar condiciones sin equivocarse

Cuando hay que invertir una condición compuesta, las leyes de De Morgan dan la transformación mecánica y correcta:

```
!(a && b)  ==  !a || !b
!(a || b)  ==  !a && !b
```

Verificado en el JDK para las cuatro combinaciones de `a` y `b`: ambas identidades son `true` siempre.

Lo que hay que retener es que **al negar, el operador cambia**: `&&` se vuelve `||` y viceversa. El error clásico es negar cada término y dejar el operador como estaba:

```java
// Original: "el usuario es válido si está activo Y verificado"
boolean valido = activo && verificado;

// MAL: "es inválido si no está activo Y no está verificado"
boolean invalido = !activo && !verificado;   // solo detecta a quien falla en AMBOS

// BIEN
boolean invalido2 = !activo || !verificado;
boolean invalido3 = !(activo && verificado);   // más legible todavía
```

La versión mala deja pasar a quien está activo pero no verificado. Es un fallo de autorización, no un detalle de estilo.

**La recomendación práctica** es la tercera línea: en vez de aplicar De Morgan a mano, envolver la condición original en un `!` y dejarla intacta. Es imposible equivocarse y se lee mejor. De Morgan hace falta cuando la forma expandida es genuinamente más clara, o cuando estás simplificando una condición que ya venía negada.

Donde De Morgan sí paga es al **eliminar negaciones dobles**, que son ilegibles:

```java
if (!(!estaVacio || !tienePermiso)) { }     // ¿qué dice esto?
if (estaVacio && tienePermiso) { }          // lo mismo, legible
```

Y hay un caso importante en el que la transformación **no** es válida, ya mencionado en la [sección 5](#5-comparar-decimales-nan-y-el-cero-negativo): con `double`, `!(v < min || v > max)` y `v >= min && v <= max` **no** son equivalentes, porque `NaN` hace que las cuatro comparaciones den `false`. De Morgan es una ley del álgebra booleana; los operadores de comparación sobre `NaN` no forman un álgebra booleana. Es el único sitio de Java donde la equivalencia se rompe.

## 21. En qué orden poner las condiciones

Como `&&` y `||` evalúan de izquierda a derecha y se detienen en cuanto pueden, el orden de las condiciones importa por tres razones distintas.

**1. Corrección.** Es la razón principal y la única que no admite negociación: la condición que habilita a otra va primero.

```java
if (s != null && s.length() > 0) { }       // obligatorio en este orden
if (i >= 0 && i < a.length && a[i] > 0) { }
```

**2. Coste.** Si dos condiciones son independientes, la barata va primero:

```java
// MAL: consulta la base de datos aunque el rol lo descarte al instante
if (repositorio.tienePedidosPendientes(id) && usuario.esAdmin()) { }

// BIEN: el chequeo en memoria descarta la mayoría de casos sin tocar la BD
if (usuario.esAdmin() && repositorio.tienePedidosPendientes(id)) { }
```

**3. Probabilidad.** Con condiciones de coste parecido, en un `&&` conviene poner primero la que más veces falla, y en un `||` la que más veces acierta. En ambos casos se sale antes.

Hay una tensión que conviene nombrar: **el orden más rápido y el orden más legible no siempre coinciden**. Cuando una reordenación por rendimiento hace que la condición se lea peor, casi siempre gana la legibilidad, salvo que haya una medición que justifique lo contrario. Reordenar dos comparaciones de enteros no se nota en ningún perfil.

Y una advertencia que se olvida: **este razonamiento asume que las condiciones no tienen efectos secundarios**. En cuanto una de ellas muta algo, reordenar deja de ser una optimización y pasa a ser un cambio de comportamiento. Es la razón de fondo por la que las expresiones booleanas deben ser puras.

## 22. La trampa de C que Java solo bloquea a medias

La [sección 1](#1-qué-devuelve-un-operador-relacional-y-por-qué-boolean-no-es-un-número) decía que Java elimina la familia de bugs de `if (x = 0)`. Es cierto para números:

```java
int n = 0;
if (n = 1) { }
```

```
error: incompatible types: int cannot be converted to boolean
```

La asignación `n = 1` es una expresión cuyo valor es `1`, un `int`, y un `int` no vale como condición. Bloqueado.

**Pero con booleanos la asignación sí compila y sí se ejecuta:**

```java
boolean flag = false;

if (flag = true) {
    System.out.println("entré, y flag ahora vale " + flag);
}
// entré, y flag ahora vale true
```

El valor de la expresión `flag = true` es `true`, que es un `boolean` perfectamente válido como condición. El programa entra siempre en el `if` y además **modifica la variable por el camino**. Sin ningún error, sin ningún warning del compilador.

Es un error tipográfico de una sola tecla —`=` en vez de `==`— sobre el tipo en el que más se usan las condiciones. Y produce un `if` que siempre se cumple, que es justo el tipo de bug que ninguna prueba del caso feliz detecta.

**Las defensas**, en orden de eficacia:

1. **No comparar con `true` ni con `false`.** El error solo existe si escribís `if (flag == true)`. Escribiendo `if (flag)`, que es lo idiomático, no hay nada que teclear mal. Lo mismo con `if (!flag)` en vez de `if (flag == false)`.
2. **Declarar `final` lo que no cambia.** `final boolean flag = false;` convierte el error en un fallo de compilación.
3. **Activar el linter.** SpotBugs, Error Prone, PMD y los avisos del IDE detectan la asignación dentro de una condición.

El único sitio donde una asignación dentro de una condición es idiomática es el bucle de lectura, y aun así conviene marcarla claramente:

```java
String linea;
while ((linea = lector.readLine()) != null) {
    procesar(linea);
}
```

Los paréntesis extra alrededor de la asignación son la señal convencional de "esto es a propósito". Muchos linters los exigen precisamente para distinguir el caso intencionado del error.

## 23. Redundancias que delatan a quien las escribe

Un booleano ya es una condición. Compararlo con otro booleano no añade nada:

```java
// Redundante
if (esValido == true) { }
if (esValido != true) { }
if (esValido == false) { }

// Idiomático
if (esValido) { }
if (!esValido) { }
if (!esValido) { }
```

Además de sobrar, `== true` es el que abre la puerta al bug de la sección anterior.

La segunda redundancia frecuente es devolver literales desde un `if`:

```java
// Redundante
if (edad >= 18) { return true; } else { return false; }

// Directo
return edad >= 18;
```

Y la tercera, el ternario que devuelve booleanos:

```java
boolean r = (a > b) ? true : false;   // sobra todo el ternario
boolean r2 = a > b;
```

Ninguna de las tres es un bug. Son señales de que quien escribe todavía piensa el `boolean` como algo que hay que convertir en decisión, en vez de como la decisión misma. Merece la pena corregirlas porque el código resultante se lee de un vistazo.

Hay un caso donde la comparación explícita **no** es redundante, y es con `Boolean` (el wrapper), donde hay tres valores posibles:

```java
Boolean confirmado = respuesta.get("confirmado");   // puede ser null

if (confirmado) { }                              // NPE si es null
if (Boolean.TRUE.equals(confirmado)) { }         // false si es null, sin excepción
if (confirmado != null && confirmado) { }        // equivalente, más explícito
```

`Boolean.TRUE.equals(x)` es el idiom estándar para "es verdadero, y si no hay dato lo tratamos como falso". Aparece constantemente al leer JSON, parámetros HTTP o columnas nullable.

---

# Parte III — El operador ternario

## 24. Sintaxis y cuándo mejora de verdad la legibilidad

El ternario es el único operador de Java con tres operandos, y el único que devuelve un valor a partir de una condición:

```java
condicion ? valorSiVerdadero : valorSiFalso
```

Su rasgo distintivo frente a un `if` no es la brevedad, es que **es una expresión**: produce un valor y por tanto se puede asignar, pasar como argumento o devolver directamente. Un `if` es una sentencia y no produce nada.

Esa diferencia es la que decide cuándo usarlo. El ternario gana cuando **la alternativa obliga a declarar una variable mutable solo para rellenarla después**:

```java
// Con if: la variable nace vacía y hay que acordarse de asignarla en las dos ramas
String saludo;
if (esFormal) {
    saludo = "Estimado";
} else {
    saludo = "Hola";
}

// Con ternario: la variable nace con su valor definitivo
String saludo2 = esFormal ? "Estimado" : "Hola";
```

La segunda versión permite además declararla `final`, lo que garantiza que nadie la cambie después. En un `record`, en un constructor o al construir un argumento, esa propiedad vale mucho.

Donde el ternario **no** gana es cuando las ramas hacen cosas en vez de producir valores. Si cada rama tiene tres líneas, escribe en un log y llama a un servicio, el ternario no aplica: eso es un `if` y punto.

Como en la [sección 12](#12-instanceof-y-el-pattern-matching), el ternario también evalúa **solo la rama que corresponde**. La rama no elegida no se ejecuta nunca, igual que con el cortocircuito:

```java
int longitud = (s != null) ? s.length() : 0;   // seguro: s.length() solo corre si s no es null
```

En bytecode es exactamente la misma estructura de salto que un `if`:

```
static int ternario(int, int);   // return a > b ? a : b;
  Code:
       0: iload_0
       1: iload_1
       2: if_icmple     9
       5: iload_0
       6: goto          10
       9: iload_1
      10: ireturn
```

No hay ninguna instrucción "ternario". El compilador genera lo mismo que para un `if/else` con un `return` en cada rama. **No es ni más rápido ni más lento**: es una forma distinta de escribir lo mismo, elegida por lo que expresa, no por lo que cuesta.

## 25. El tipo del resultado: promoción numérica entre las ramas

Aquí está la parte del ternario que casi nadie conoce, y la que produce resultados imposibles de explicar sin ella.

El ternario es una expresión, así que **tiene un tipo**. Ese tipo se calcula a partir de los tipos de las dos ramas, no del que tenga la rama que se ejecute. Y cuando las dos ramas son numéricas de tipos distintos, se aplica **promoción numérica binaria**, exactamente igual que en una suma.

```java
Object o = true ? Integer.valueOf(1) : Double.valueOf(2.0);
System.out.println(o);                    // 1.0
System.out.println(o.getClass());         // class java.lang.Double
```

Leído despacio: la condición es `true`, se toma la primera rama, que contiene un `Integer` con el valor 1. Y el resultado impreso es **`1.0`, un `Double`**. El `Integer` fue desempaquetado a `int`, promocionado a `double` porque la otra rama era `Double`, y vuelto a empaquetar como `Double`.

La rama que ni se ejecutó **cambió el tipo del resultado**.

Lo mismo con literales:

```java
Object o2 = true ? 1 : 2.0;
System.out.println(o2);              // 1.0
System.out.println(o2.getClass());   // class java.lang.Double
```

Y con `char`, que produce el caso simétrico:

```java
Object o3 = false ? 1 : 'x';
System.out.println(o3);              // x
System.out.println(o3.getClass());   // class java.lang.Character

char c = true ? 'a' : 65;
System.out.println(c);               // a
```

Este último funciona por una regla especial de la JLS: si una rama es `char` y la otra es una **constante de compilación** de tipo `int` que cabe en `char`, el tipo del resultado es `char`. Si el `65` fuera una variable `int` en vez de un literal, el resultado sería `int` y la asignación a `char` no compilaría.

**Dónde muerde esto en producción.** El caso clásico es un ternario que elige entre dos valores numéricos de tipos distintos y acaba en un formateo o en una comparación:

```java
// Ambas ramas "valen 1", pero el resultado es double
Object valor = condicion ? 1 : 1.0;
System.out.println("cantidad: " + valor);   // "cantidad: 1.0" incluso cuando condicion es true
```

Un log que dice `1.0` donde debería decir `1`, un JSON que serializa `1.0` y rompe un parser estricto al otro lado, una clave de `Map` que no encuentra nada porque `Integer.valueOf(1)` y `Double.valueOf(1.0)` no son `equals`.

**La regla:** que las dos ramas de un ternario devuelvan el mismo tipo. Si no lo hacen, ponelo explícito con un cast o rompé el ternario en un `if`. Y si una rama devuelve un `int` y la otra un `double` porque conceptualmente son cosas distintas, eso es una señal de que el ternario está uniendo dos cosas que no deberían compartir variable.

## 26. El NullPointerException del ternario

De la regla de la sección anterior sale un fallo concreto y muy desagradable: **un ternario puede lanzar `NullPointerException` sin que aparezca un solo punto en la línea**.

```java
Integer nulo = null;

Integer malo = false ? 1 : nulo;
// NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "<local10>" is null
```

Lo que ocurre es esto: una rama es `int` (el literal `1`) y la otra es `Integer`. Por promoción numérica binaria, **el tipo del ternario completo es `int`**. Y para producir un `int` a partir de `nulo` hay que llamar a `intValue()` sobre una referencia nula.

Lo verdaderamente traicionero es que **depende de qué rama se tome**:

```java
Integer bien = true ? 1 : nulo;
System.out.println(bien);   // 1   <- ninguna excepción
```

Con la condición en `true` se toma el literal, que ya es un `int`, y no hay nada que desempaquetar. El mismo código, con el mismo `nulo`, funciona o revienta según el valor de una condición en tiempo de ejecución.

Esa es la firma de un bug caro: **pasa los tests, entra en producción, y falla el día que la condición se da al revés**.

La forma de evitarlo es hacer que las dos ramas sean del mismo tipo de referencia, con lo que desaparece la promoción y con ella el desempaquetado:

```java
Integer seguro = false ? Integer.valueOf(1) : nulo;   // tipo Integer, sin unboxing
```

El caso realista donde esto aparece es, otra vez, un `Map`:

```java
Map<String, Integer> stock = new HashMap<>();

int cantidad = hayQueConsultar ? stock.get(sku) : 0;   // NPE si el sku no existe
```

Las dos ramas son `Integer` e `int`, así que el ternario es de tipo `int` y `stock.get(sku)` se desempaqueta. Si la clave no está, `get` devuelve `null` y revienta. Y solo revienta cuando `hayQueConsultar` es `true`.

La versión correcta:

```java
int cantidad = hayQueConsultar ? stock.getOrDefault(sku, 0) : 0;
```

**El resumen de las dos secciones:** un ternario con ramas de tipos numéricos distintos, o con una rama primitiva y otra wrapper, es una expresión con dos comportamientos ocultos —promoción y desempaquetado— que solo se manifiestan con ciertos valores en ciertos momentos. Que las dos ramas tengan el mismo tipo elimina los dos problemas a la vez.

## 27. Ternarios anidados y el límite de lo legible

Un ternario puede contener otro ternario, y como su asociatividad es **de derecha a izquierda**, la cadena se lee como un `if/else if/else`:

```java
String categoria = edad < 13 ? "niño"
                 : edad < 20 ? "adolescente"
                 : edad < 65 ? "adulto"
                 : "jubilado";
```

Formateado así, en columna, con las condiciones alineadas, esta cadena es perfectamente legible: se lee de arriba abajo como una tabla de casos y cada línea dice una cosa. Es una de las pocas formas de anidamiento que mejora el código en vez de empeorarlo, porque la alternativa —cuatro asignaciones a una variable mutable dentro de un `if/else if`— es más larga y no permite `final`.

Lo que sí destruye la legibilidad es anidar **en la condición o en el medio**:

```java
// Ilegible
String r = a ? (b ? "x" : "y") : (c ? (d ? "z" : "w") : "v");
```

Aquí ya no hay una cadena de casos, hay un árbol, y para leerlo hay que rastrear paréntesis. Cualquier cosa así se escribe con `if` o, mejor, se extrae a un método con nombre.

**Los límites prácticos**, que coinciden con lo que suelen configurar los linters:

- Una cadena de hasta **tres o cuatro** ternarios en columna, todos decidiendo sobre la misma variable, es aceptable.
- Un ternario dentro de la condición de otro, nunca.
- Un ternario dentro de una expresión más grande (concatenación, argumento de método, índice de array) obliga a paréntesis y casi siempre se lee mejor extraído a una variable con nombre.

Un caso especialmente común de este último:

```java
// Confuso: el ternario compite con la concatenación
System.out.println("Hay " + n + " elemento" + n == 1 ? "" : "s");   // ni siquiera compila bien

// Con paréntesis, correcto pero denso
System.out.println("Hay " + n + " elemento" + (n == 1 ? "" : "s"));

// Extraído, obvio
String plural = (n == 1) ? "" : "s";
System.out.println("Hay " + n + " elemento" + plural);
```

La primera línea es un buen recordatorio de la tabla de precedencia: `+` liga más fuerte que `?:` y muchísimo más fuerte que `==`, así que la expresión se agrupa de una forma que no tiene nada que ver con la intención.

Y una alternativa que conviene tener presente: cuando el ternario decide entre valores en función de una única variable con pocos valores posibles, el `switch` como expresión (definitivo desde Java 14) suele ser mejor:

```java
String etiqueta = switch (estado) {
    case ACTIVO    -> "en curso";
    case PAUSADO   -> "en espera";
    case CANCELADO -> "cancelado";
};
```

Sin `default`, sin `break`, y el compilador verifica que estén todos los casos del enum.

---

# Parte IV — Bitwise: operar bit a bit

## 28. El modelo mental: 32 bits y complemento a dos

Los operadores de esta parte dejan de tratar el número como un número y lo tratan como **una fila de bits**. Para usarlos hay que tener presente lo que ya vimos en [Math Operations](07-math-operations.md#12-cómo-se-representan-los-enteros-complemento-a-dos): Java guarda los enteros en **complemento a dos**, con el bit más significativo actuando como signo.

Los tamaños, que aquí importan más que nunca:

| Tipo | Bits | En operaciones bitwise |
|---|---|---|
| `byte` | 8 | se promociona a `int` |
| `short` | 16 | se promociona a `int` |
| `char` | 16 | se promociona a `int` |
| `int` | 32 | opera en 32 bits |
| `long` | 64 | opera en 64 bits |

Los operadores bitwise **solo aceptan tipos enteros** (`byte`, `short`, `char`, `int`, `long`) y `boolean`. Con `float` o `double` no compilan, porque los bits de un decimal IEEE 754 no tienen ninguna relación aritmética con su valor. (Si de verdad hace falta manipularlos, existen `Double.doubleToLongBits` y `Double.longBitsToDouble`, que hacen la conversión explícita.)

La herramienta imprescindible para trabajar con bits es poder verlos:

```java
System.out.println(Integer.toBinaryString(-1));   // 11111111111111111111111111111111
System.out.println(Integer.toHexString(-1));      // ffffffff
System.out.println(Integer.toBinaryString(6));    // 110
```

Ojo con el último: `toBinaryString` **no rellena con ceros a la izquierda**, así que un número pequeño sale corto y cuesta alinearlo con otro. Para depurar conviene una ayuda:

```java
static String bin(int v) {
    return String.format("%32s", Integer.toBinaryString(v)).replace(' ', '0');
}
```

Con eso, `bin(6)` da `00000000000000000000000000000110` y las comparaciones visuales cuadran.

**El punto clave de toda la parte:** cuando escribís `~6` o `b & 0xFF`, no estás operando sobre 8 bits ni sobre "los bits que hagan falta". Estás operando sobre **32 bits exactos**, incluyendo los 26 ceros o unos de la izquierda que no aparecen al imprimir el número. Casi todos los errores de esta parte vienen de olvidarlo.

## 29. AND, OR y XOR sobre enteros

Los tres operadores binarios aplican la operación lógica **bit a bit, en la misma posición**, y devuelven un entero.

| `a` | `b` | `a & b` | `a \| b` | `a ^ b` |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 |

```java
System.out.println(6 | 5);   // 7
System.out.println(6 & 5);   // 4
System.out.println(6 ^ 5);   // 3
```

Desplegado, con `6 = 0110` y `5 = 0101`:

```
   0110        0110        0110
 | 0101      & 0101      ^ 0101
 ------      ------      ------
   0111        0100        0011
    = 7         = 4         = 3
```

Lo que hay que quedarse, más allá de la mecánica, es **qué significa cada uno como herramienta**:

- **`&` filtra.** Deja pasar solo los bits que están a uno en los dos operandos. Es la operación de "quedarme con esta parte", y es la base de las máscaras.
- **`|` combina.** Junta los bits de ambos. Es la operación de "añadir esto a lo que ya había".
- **`^` marca diferencias.** Da uno exactamente donde los operandos difieren. Es la operación de "alternar" y la de "comparar".

Esa tercera lectura de `^` es la más útil y la menos obvia, y tiene sección propia ([43](#43-xor-propiedades-y-usos-reales)).

Como cualquier operador binario, tienen su forma de asignación compuesta: `&=`, `|=`, `^=`. Y como cualquier asignación compuesta, esconden un cast ([sección 35](#35-asignación-compuesta-bitwise-y-su-cast-oculto)).

## 30. El complemento y por qué no es la negación lógica

`~` es unario e invierte **todos** los bits: cada cero pasa a uno y cada uno a cero.

```java
System.out.println(~6);                       // -7
System.out.println(Integer.toBinaryString(6));  // 110
```

El resultado sorprende hasta que se ven los 32 bits completos:

```
 6 = 00000000000000000000000000000110
~6 = 11111111111111111111111111111001   = -7
```

El bit más significativo pasó a uno, así que el número es negativo. Y en complemento a dos ese patrón concreto vale `-7`.

Hay una identidad que resuelve el operador entero y evita tener que contar bits:

```
~x  ==  -x - 1
```

Verificada en todo el rango relevante:

| `x` | `~x` | `-x - 1` |
|---|---|---|
| 0 | -1 | -1 |
| 1 | -2 | -2 |
| 5 | -6 | -6 |
| -1 | 0 | 0 |
| -8 | 7 | 7 |
| `Integer.MAX_VALUE` | -2147483648 | -2147483648 |
| `Integer.MIN_VALUE` | 2147483647 | 2147483647 |

Nótese que `~` es el único operador de la familia que **nunca desborda**: es una biyección sobre los 32 bits, siempre reversible (`~~x == x`), incluso en `Integer.MIN_VALUE`, donde `-x` sí desborda.

**Ahora la parte importante, porque es un error que arrastran varias fuentes.** `~` **no** es el equivalente entero de `!`. No lo es en ningún sentido útil, y la comparación induce a error de tres formas:

**Primera: no compila sobre booleanos.**

```java
boolean b = true;
boolean r = ~b;
```

```
error: bad operand type boolean for unary operator '~'
    boolean r = ~b;
                ^
```

Y al revés tampoco: `!5` da `bad operand type int for unary operator '!'`. Son operadores de tipos disjuntos.

**Segunda: la analogía "invierte el valor" es falsa en el sentido que importa.** `!` mapea `true` a `false` y `false` a `true`, dos valores que se intercambian. `~` mapea `x` a `-x-1`, que no es "el contrario" de nada: `~0` es `-1`, no `1`. Si alguien traslada la intuición de `!` a `~` pensando que `~0` da "verdadero" y `~1` da "falso", se equivoca en ambos.

**Tercera: sobre el patrón de bits "todo unos", `~` y `!` sí coinciden**, y eso hace la confusión más pegajosa. En C, donde `0` es falso y cualquier otra cosa verdadera, `~0` es `-1`, que es verdadero, y de ahí sale la costumbre de tratar `~` como negación. En Java esa costumbre no se puede ni escribir.

**Dónde se usa `~` de verdad:** para construir máscaras de exclusión. `~MASCARA` es "todos los bits menos estos", y `x & ~MASCARA` es "quitá estos bits de `x`". Es el idiom de la [sección 37](#37-flags-y-máscaras-poner-quitar-alternar-consultar).

```java
System.out.println((char) ('a' & ~0x20));   // A   <- apaga el bit 5: minúscula a mayúscula
System.out.println((char) ('A' | 0x20));    // a   <- lo enciende: mayúscula a minúscula
```

Ese par de líneas es el truco clásico de ASCII: en el rango de letras, la única diferencia entre mayúscula y minúscula es el bit 5. Sigue siendo válido en parsers ASCII (`Content-Type`, palabras clave de SQL, cabeceras HTTP) y sigue siendo incorrecto para texto de usuario, donde hay que usar `Character.toUpperCase`, que conoce Unicode.

## 31. Promoción: por qué la máscara 0xFF es obligatoria

Los operadores bitwise aplican la misma **promoción numérica** que los aritméticos: todo lo que sea menor que `int` se convierte a `int` antes de operar, y el resultado es `int`.

```java
byte b = 6;
System.out.println(~b);                              // -7
System.out.println(((Object) (~b)).getClass());      // class java.lang.Integer
```

El resultado de una operación sobre un `byte` es un `int`. Nunca un `byte`.

Y la promoción de un `byte` a `int` es con **extensión de signo**: los 24 bits nuevos se rellenan con el bit de signo, no con ceros. Esto es exactamente lo correcto para preservar el valor numérico, y exactamente lo que arruina el trabajo con datos binarios:

```java
byte neg = -1;
System.out.println((int) neg);   // -1
```

El byte `11111111` se convierte en `11111111111111111111111111111111`. Como número tiene sentido: `-1` sigue siendo `-1`. Como **dato** no: ese byte, leído de un socket o de un fichero, representaba el valor `255`.

Y ahí está el bug del parser de protocolo binario. Java no tiene tipos sin signo, así que un `byte` leído de un stream va de `-128` a `127`, no de `0` a `255`. Cualquier byte con el bit alto encendido llega como negativo:

```java
byte cabecera = (byte) 0xFF;
if (cabecera == 0xFF) { }        // false: -1 no es igual a 255
```

**La solución es la máscara, y es obligatoria:**

```java
byte neg = -1;
System.out.println(neg & 0xFF);   // 255
```

El literal `0xFF` es un `int` que vale `00000000000000000000000011111111`. El `&` deja pasar solo los 8 bits bajos y **apaga los 24 de la extensión de signo**. El resultado es el valor sin signo del byte, ya como `int`.

En bytecode se ve que es literalmente eso, sin ninguna magia:

```
static int mascara(byte);   // return b & 0xFF;
  Code:
       0: iload_0
       1: sipush        255
       4: iand
       5: ireturn

static int sinMascara(byte);   // return b;
  Code:
       0: iload_0
       1: ireturn
```

El método sin máscara ni siquiera genera una instrucción de conversión: el `byte` ya está en la pila como `int` con el signo extendido.

Las máscaras equivalentes para los otros tipos:

```java
short sh = -2;
System.out.println(sh & 0xFFFF);              // 65534

System.out.println(Byte.toUnsignedInt((byte) -1));   // 255
```

Desde Java 8 existen `Byte.toUnsignedInt`, `Short.toUnsignedInt` e `Integer.toUnsignedLong`, que hacen lo mismo con un nombre que se entiende. **En código nuevo son preferibles a la máscara**, porque `b & 0xFF` exige que quien lo lea sepa por qué está ahí, y `Byte.toUnsignedInt(b)` lo dice.

**La regla:** cada vez que un `byte` venga de fuera del programa —red, fichero, criptografía, imagen, serialización— y se vaya a comparar, indexar o imprimir como número, hay que enmascararlo. Sin excepción.

## 32. Los desplazamientos a fondo

Hay tres operadores de desplazamiento, y la diferencia entre los dos de la derecha es la que hay que memorizar.

**`<<` desplaza a la izquierda** y rellena con ceros por la derecha. Los bits que se salen por la izquierda se pierden.

```java
System.out.println(12 << 2);    //  48
System.out.println(-12 << 2);   // -48
```

**`>>` desplaza a la derecha con extensión de signo** (*arithmetic right shift*): rellena por la izquierda con copias del bit de signo. Los negativos siguen siendo negativos.

```java
System.out.println(12 >> 2);    //  3
System.out.println(-12 >> 2);   // -3
```

**`>>>` desplaza a la derecha rellenando con ceros** (*logical right shift*): trata el número como si no tuviera signo.

```java
System.out.println(12 >>> 2);    //  3
System.out.println(-12 >>> 2);   //  1073741821
```

Con positivos, `>>` y `>>>` coinciden. Con negativos, no se parecen en nada: `>>` conserva el valor dividido; `>>>` reinterpreta los 32 bits como un número sin signo enorme.

Visto en binario, `-12 >>> 2`:

```
    -12 = 11111111111111111111111111110100
-12>>>2 = 00111111111111111111111111111101   = 1073741821
```

`>>>` **no existe en C ni en C++**, donde el comportamiento del desplazamiento a la derecha sobre negativos depende del compilador. Java lo definió explícitamente y añadió el operador que faltaba, precisamente para que el resultado fuera portable. Es de las pocas cosas en las que Java añadió sintaxis respecto a C en vez de quitarla.

**No hay `<<<`.** No haría falta: al desplazar a la izquierda siempre se rellena con ceros, así que no hay dos variantes posibles.

Un detalle de precedencia importante: los desplazamientos están **por debajo** de la aritmética (nivel 5 frente a 3 y 4). Es decir, `x << 2 + 1` es `x << 3`, no `(x << 2) + 1`. Otro sitio donde los paréntesis no son opcionales en la práctica.

## 33. Los dos mitos de los desplazamientos

Casi todos los tutoriales incluyen dos afirmaciones sobre los desplazamientos que son falsas, y las dos se desmontan en una línea.

**Mito 1: "`>> n` equivale a dividir por 2^n".**

Es cierto para positivos y **falso para negativos**, porque los dos operadores redondean en direcciones opuestas: `/` trunca hacia cero y `>>` redondea hacia menos infinito.

| `v` | `v >> 1` | `v / 2` | `Math.floorDiv(v, 2)` |
|---|---|---|---|
| -1 | -1 | 0 | -1 |
| -3 | -2 | -1 | -2 |
| -5 | -3 | -2 | -3 |
| -7 | -4 | -3 | -4 |
| -12 | -6 | -6 | -6 |

El caso `-1 >> 1 = -1` frente a `-1 / 2 = 0` es el contraejemplo más limpio: desplazar un `-1` a la derecha no lo cambia **nunca**, por muchas veces que lo hagas, porque los unos que entran por la izquierda reponen los que salen por la derecha. Dividiendo, en cambio, se llega a cero al primer intento.

La última fila explica por qué el mito sobrevive: con `-12`, que es múltiplo de la potencia de dos, ambas operaciones coinciden. Los ejemplos de los tutoriales suelen usar justamente números divisibles.

**La equivalencia correcta es con `Math.floorDiv`**, no con `/`: `x >> 1` y `Math.floorDiv(x, 2)` dan lo mismo siempre, como muestra la última columna de la tabla.

**Mito 2: "`>>>` siempre devuelve un número positivo".**

Falso, y basta con desplazar cero posiciones:

```java
System.out.println(-12 >>> 0);                  // -12
System.out.println(-1 >>> 0);                   // -1
System.out.println(Integer.MIN_VALUE >>> 0);    // -2147483648
System.out.println(-12 >>> 32);                 // -12
```

Con distancia cero no se mueve ningún bit, así que el valor sale intacto, signo incluido. Y con distancia 32 pasa lo mismo por la razón de la sección siguiente.

La afirmación correcta es más estrecha: **`>>>` con una distancia entre 1 y 31 sobre un `int` siempre da un resultado no negativo**, porque garantiza al menos un cero en el bit de signo. Fuera de ese rango, no.

No es una objeción de laboratorio. Una distancia de desplazamiento calculada (`x >>> desplazamiento`, donde `desplazamiento` sale de una variable) puede valer cero perfectamente, y entonces el código que asumía "esto ya es positivo" opera sobre un negativo.

## 34. La distancia de desplazamiento se toma módulo 32 o 64

Java no permite desplazar más allá del ancho del tipo. En vez de dar cero o lanzar una excepción, **se queda solo con los bits bajos de la distancia**: los 5 últimos para `int` (0–31) y los 6 últimos para `long` (0–63). Equivale a tomar la distancia módulo 32 o módulo 64.

```java
System.out.println(1 << 32);    // 1            <- 32 % 32 = 0: no desplaza nada
System.out.println(1 << 33);    // 2            <- 33 % 32 = 1
System.out.println(1L << 32);   // 4294967296   <- long: 32 sí desplaza
System.out.println(1L << 64);   // 1            <- 64 % 64 = 0
System.out.println(1 << -1);    // -2147483648  <- -1 son 31 bits bajos a uno: 31
```

Las dos trampas que produce:

**Trampa 1: el tipo del operando izquierdo decide, no el de destino.** Es la misma regla que arruinaba `1000 * 60 * 60 * 24 * 365` en el capítulo anterior:

```java
long grande = 1 << 40;    // 1 << 8 = 256, luego convertido a long. MAL.
long bien   = 1L << 40;   // 1099511627776. Bien.
```

`1 << 40` opera en 32 bits porque `1` es un literal `int`. El `40 % 32 = 8`, así que el resultado es `256`, y solo entonces se amplía a `long`. La `L` tiene que ir en el operando izquierdo, no en la variable de destino.

**Trampa 2: `1 << 31` es negativo.**

```java
System.out.println(1 << 31);   // -2147483648
```

El bit que se enciende es el de signo. Por eso una lista de flags con `int` **solo tiene 31 posiciones útiles** si se pretende que los valores sean positivos, y por eso las máscaras de bits llegado ese punto se declaran en `long`.

Y una consecuencia práctica al construir máscaras de N bits:

```java
System.out.println((1 << 5) - 1);     // 31          <- correcto: cinco unos
System.out.println((1 << 32) - 1);    // 0           <- MAL: 1<<32 es 1, menos 1 es 0
System.out.println((1L << 32) - 1);   // 4294967295  <- correcto
```

El idiom `(1 << n) - 1` para "una máscara de n bits" funciona bien hasta `n = 31` y falla en silencio en `n = 32`, que es justo el valor que uno pondría para decir "todos los bits". La versión con `long` no tiene ese agujero.

## 35. Asignación compuesta bitwise y su cast oculto

Todos los operadores de esta parte tienen su forma compuesta: `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`. Y todos heredan la trampa que ya vimos en [Math Operations](07-math-operations.md#8-asignación-compuesta-y-su-cast-oculto): la JLS define `E1 op= E2` como `E1 = (T)((E1) op (E2))`, con un **cast implícito** al tipo de la izquierda.

Combinado con la promoción a `int` de la sección 31, el resultado es que las asignaciones compuestas sobre `byte` y `short` truncan sin avisar:

```java
byte pequeno = 64;
System.out.println(pequeno << 1);   // 128   <- es un int, correcto

byte acumulador = 64;
acumulador <<= 1;
System.out.println(acumulador);     // -128  <- truncado a 8 bits
```

La misma operación, dos resultados. En la primera línea el resultado es un `int` y vale `128`. En la segunda, el `int` `128` se trunca a `byte`, y `10000000` en 8 bits es `-128`.

El bytecode muestra el cast que nadie escribió:

```
static byte compuestoShift(byte);   // b <<= 1;
  Code:
       0: iload_0
       1: iconst_1
       2: ishl
       3: i2b           // <-- cast int-a-byte insertado por el compilador
       4: istore_0
       5: iload_0
       6: ireturn
```

Ese `i2b` es el mismo fantasma que aparecía con `+=`.

**La consecuencia práctica** es la misma recomendación del capítulo anterior, reforzada: **los acumuladores de bits se declaran `int` o `long`, nunca `byte` ni `short`**. Es exactamente el tipo de variable que en código de protocolos binarios uno tiende a declarar pequeña "porque solo guarda un byte", y es donde el truncamiento pasa desapercibido.

Un matiz sobre `>>>=`: existe, pero conviene desconfiar de él sobre tipos pequeños, porque el desplazamiento sin signo se hace sobre los 32 bits promocionados —con la extensión de signo ya aplicada— y luego se trunca. El resultado casi nunca es el que uno imagina. Si necesitás desplazamiento sin signo sobre un byte, enmascará primero (`(b & 0xFF) >>> n`) y guardá en un `int`.

## 36. La trampa de precedencia que no compila y la que sí

De la tabla de la [sección 19](#19-precedencia-entre-operadores-lógicos) sale el error de precedencia más conocido de Java, heredado de C: **`==` y `!=` ligan más fuerte que `&`, `^` y `|`**.

Esto significa que `x & y == 1` **no** se agrupa como `(x & y) == 1`, sino como `x & (y == 1)`. En C eso compila y da un resultado sin sentido. En Java, afortunadamente, no compila:

```java
int x = 1, y = 1;
boolean r = x & y == 1;
```

```
error: bad operand types for binary operator '&'
  first type:  int
  second type: boolean
```

El sistema de tipos salva la situación: `y == 1` es un `boolean`, `x` es un `int`, y `&` no acepta esa mezcla. **Este es un caso donde la separación estricta entre `boolean` y los enteros paga de verdad**: convierte un bug silencioso de C en un error de compilación de Java.

La forma correcta, con los paréntesis que la precedencia no pone sola:

```java
boolean r = (x & y) == 1;   // true
```

**Pero el sistema de tipos solo protege cuando los tipos difieren.** Si los dos lados son booleanos, la trampa es idéntica y compila perfectamente:

```java
boolean a = true, b = false, c = true;

System.out.println(a ^ b | c);     // true    <- (a ^ b) | c
System.out.println(a ^ (b | c));   // false
```

Resultados opuestos, sin ningún aviso. Es el ejemplo de la sección 19, y es la razón de que la regla sea la misma en ambas partes del capítulo: **en cuanto una expresión mezcla operadores de bits o lógicos con comparaciones, hay que poner paréntesis**.

Las combinaciones que más veces salen mal, con la agrupación real al lado:

| Escrito | Se agrupa como | Casi siempre se quería |
|---|---|---|
| `x & y == 1` | `x & (y == 1)` | `(x & y) == 1` |
| `flags & MASCARA != 0` | `flags & (MASCARA != 0)` | `(flags & MASCARA) != 0` |
| `a ^ b \| c` | `(a ^ b) \| c` | depende, escribilo |
| `x << 2 + 1` | `x << (2 + 1)` | `(x << 2) + 1` |
| `~x & 0xFF` | `(~x) & 0xFF` | correcto, `~` es unario |

Las dos primeras filas son el mismo error, y la segunda es la que aparece en código de flags real. Por suerte, como el operando derecho acaba siendo `boolean`, las dos fallan en compilación. La tercera y la cuarta compilan y hacen otra cosa.

---

# Parte V — Bits en la práctica

## 37. Flags y máscaras: poner, quitar, alternar, consultar

El uso clásico de los operadores de bits es guardar **muchos booleanos independientes en un solo entero**. Cada bit representa una opción; un `int` da hasta 32, un `long` hasta 64.

Las constantes se declaran con desplazamientos, que dejan clara la posición de cada bit:

```java
static final int LECTURA   = 1 << 0;   // 0001
static final int ESCRITURA = 1 << 1;   // 0010
static final int EJECUCION = 1 << 2;   // 0100
static final int BORRADO   = 1 << 3;   // 1000
```

Escribirlas así en vez de `1, 2, 4, 8` no es cosmético: hace evidente que son posiciones y no valores, y hace imposible el error de repetir una potencia o saltarse una.

Las cuatro operaciones que hacen falta, con su idiom:

| Quiero | Se escribe | Por qué funciona |
|---|---|---|
| **Poner** un flag | `p \|= ESCRITURA` | OR enciende sin tocar el resto |
| **Quitar** un flag | `p &= ~ESCRITURA` | AND con el complemento apaga solo ese bit |
| **Alternar** un flag | `p ^= LECTURA` | XOR con uno invierte ese bit |
| **Consultar** un flag | `(p & ESCRITURA) != 0` | AND filtra; distinto de cero es "estaba" |

Verificado paso a paso:

```java
int permisos = LECTURA | ESCRITURA;
System.out.println(Integer.toBinaryString(permisos));   // 11

System.out.println((permisos & ESCRITURA) != 0);        // true
System.out.println((permisos & EJECUCION) != 0);        // false

permisos |= EJECUCION;
System.out.println(Integer.toBinaryString(permisos));   // 111

permisos &= ~ESCRITURA;
System.out.println(Integer.toBinaryString(permisos));   // 101

permisos ^= LECTURA;
System.out.println(Integer.toBinaryString(permisos));   // 100
```

Dos consultas más que hacen falta a menudo:

```java
// ¿Tiene TODOS estos?
boolean tieneTodos = (permisos & (LECTURA | ESCRITURA)) == (LECTURA | ESCRITURA);

// ¿Tiene ALGUNO de estos?
boolean tieneAlguno = (permisos & (LECTURA | ESCRITURA)) != 0;

// ¿Tiene EXACTAMENTE estos y ninguno más de ese grupo?
boolean tieneSoloLectura = (permisos & (LECTURA | ESCRITURA)) == LECTURA;
```

La tercera es la que más veces se escribe mal, porque `==` sobre el resultado de una máscara **solo tiene sentido si el operando derecho es la combinación completa que estás filtrando**. Comparar contra un solo flag después de filtrar por varios responde "tiene este y no los otros del grupo", que rara vez es la pregunta.

**Dónde se sigue usando esto hoy.** Los flags de bits son de los años setenta y en código de aplicación normal están superados por `EnumSet` ([sección 39](#39-enumset-la-alternativa-que-casi-siempre-gana)). Siguen siendo la representación correcta en tres sitios:

- **Interoperar con algo externo que los usa:** permisos POSIX, flags de `open()`, cabeceras de protocolos de red, formatos de fichero binarios, la propia especificación del bytecode de la JVM (los modificadores `public`, `static`, `final` de un método son literalmente un `int` de flags).
- **Serializar compacto:** guardar 32 booleanos en 4 bytes en una columna, un mensaje o una caché.
- **Código muy caliente donde la diferencia se mide**, que es un caso mucho más raro de lo que la gente cree.

## 38. El bug de comprobar la máscara comparando con uno

De la tabla anterior, la fila de consultar es la que más se escribe mal, y el error es casi invisible:

```java
if ((permisos & ESCRITURA) == 1) { }    // MAL
if ((permisos & ESCRITURA) != 0) { }    // BIEN
```

La razón es que `permisos & ESCRITURA` no devuelve un booleano ni un `1`: devuelve **el valor del flag si estaba encendido**, y cero si no.

```java
int p = LECTURA | ESCRITURA | EJECUCION;

System.out.println(p & ESCRITURA);           // 2     <- no 1
System.out.println((p & ESCRITURA) == 1);    // false <- MAL, el usuario sí tiene permiso
System.out.println((p & ESCRITURA) != 0);    // true  <- BIEN
```

`ESCRITURA` vale `2`, así que el filtrado devuelve `2`, no `1`. La comparación con `1` da `false` y el programa concluye que el usuario **no** tiene permiso de escritura cuando sí lo tiene.

Lo que hace este bug especialmente traicionero es que **funciona para el primer flag**. `LECTURA` vale `1`, así que `(p & LECTURA) == 1` sí da `true`. Quien escribe la primera comprobación la prueba con el primer permiso, ve que funciona, y copia el patrón para los demás. A partir del segundo flag, todo falla.

Las tres formas correctas, en orden de preferencia:

```java
(p & ESCRITURA) != 0                 // la habitual
(p & ESCRITURA) == ESCRITURA         // explícita; obligatoria si la máscara tiene varios bits
Integer.bitCount(p & GRUPO) == 2     // cuando querés contar cuántos del grupo están
```

La segunda es la única correcta cuando la máscara agrupa varios bits, porque en ese caso `!= 0` responde "alguno" y `== MASCARA` responde "todos".

## 39. EnumSet: la alternativa que casi siempre gana

Todo lo de las dos secciones anteriores tiene una alternativa que hace lo mismo, con **la misma representación interna en bits**, y con tipos de verdad.

```java
enum Permiso { LECTURA, ESCRITURA, EJECUCION, BORRADO }

EnumSet<Permiso> permisos = EnumSet.of(Permiso.LECTURA, Permiso.ESCRITURA);
System.out.println(permisos);                             // [LECTURA, ESCRITURA]
System.out.println(permisos.contains(Permiso.ESCRITURA)); // true

permisos.add(Permiso.EJECUCION);
permisos.remove(Permiso.ESCRITURA);
System.out.println(permisos);                             // [LECTURA, EJECUCION]

System.out.println(EnumSet.complementOf(permisos));       // [ESCRITURA, BORRADO]
System.out.println(EnumSet.allOf(Permiso.class));         // [LECTURA, ESCRITURA, EJECUCION, BORRADO]
System.out.println(EnumSet.noneOf(Permiso.class));        // []
```

Las operaciones de conjuntos son las mismas de los bits, con otro nombre:

| Con bits | Con EnumSet |
|---|---|
| `a \| b` | `addAll` |
| `a & b` | `retainAll` |
| `a & ~b` | `removeAll` |
| `~a` | `EnumSet.complementOf(a)` |
| `(a & F) != 0` | `contains(F)` |
| `Integer.bitCount(a)` | `size()` |

**Y no es una capa de abstracción cara: por dentro es exactamente un `long`.**

```java
System.out.println(permisos.getClass());   // class java.util.RegularEnumSet
```

`RegularEnumSet` guarda un único campo `long elements` y hace precisamente `elements |= (1L << ordinal)`. Para enums de más de 64 constantes la JDK cambia sola a `JumboEnumSet`, que usa un array de `long`. El detalle es transparente.

Medido con 50 millones de iteraciones en JDK 25, con calentamiento previo y sin JMH:

```
flags int          ->  2 ms
EnumSet.contains   ->  7 ms
```

**Estas cifras hay que leerlas con cuidado, y conviene explicar por qué.** Dividido por las iteraciones, `contains` sale a 0,14 nanosegundos por llamada, menos de lo que tarda un ciclo de reloj. Eso no significa que `contains` sea infinitamente rápido: significa que **el JIT sacó buena parte del trabajo fuera del bucle**, porque el conjunto y el argumento eran invariantes. Un microbenchmark hecho así mide sobre todo la capacidad del compilador de eliminar código, no el coste de la operación.

Lo que sí se puede concluir del experimento es lo único que importa para decidir: **ambas operaciones son tan baratas que el JIT las hace desaparecer**. La diferencia entre las dos es del orden de unos pocos nanosegundos por llamada en el peor caso, contra los cientos de microsegundos de cualquier consulta a base de datos o llamada de red que rodee a esa comprobación de permisos. Para que la elección se notara en un perfil real, la comprobación tendría que ser el cuello de botella de la aplicación, cosa que no ocurre en código de negocio. Si alguna vez hiciera falta la cifra de verdad, habría que medirla con JMH, que existe precisamente para impedir estas optimizaciones.

**Lo que ganás a cambio** es lo que hace que la elección casi nunca sea dudosa:

- **Tipos.** `int` acepta cualquier entero; `EnumSet<Permiso>` solo acepta permisos. Con flags nada impide pasar un `int` de estados donde se esperaba uno de permisos, ni combinar constantes de dos enumeraciones distintas.
- **Legibilidad al depurar.** Un log que dice `[LECTURA, EJECUCION]` frente a uno que dice `5`.
- **Sin errores de máscara.** No hay `== 1` posible, no hay `~` mal puesto, no hay 33 flags en un `int`.
- **API de colección.** `stream()`, `forEach`, `containsAll`, iteración en orden de declaración.

**La regla:** usá `EnumSet` por defecto. Los flags con `int` solo cuando el formato binario lo impone desde fuera o cuando una medición concreta lo justifica.

Y un apunte de mutabilidad, porque `EnumSet` es mutable y eso choca con el estilo inmutable: para exponerlo sin riesgo se usa `Collections.unmodifiableSet(...)` o `Set.copyOf(...)`, y las operaciones se hacen sobre una copia (`EnumSet.copyOf(original)`) en vez de mutar el original.

## 40. BitSet: cuando 64 bits no alcanzan

Cuando los bits que hay que manejar son miles o millones, ni un `int` ni un `long` sirven. `java.util.BitSet` es un vector de bits que crece solo:

```java
BitSet bits = new BitSet();
bits.set(3);
bits.set(100);
bits.set(1000);

System.out.println(bits);               // {3, 100, 1000}
System.out.println(bits.cardinality()); // 3        <- cuántos bits en uno
System.out.println(bits.get(100));      // true
System.out.println(bits.length());      // 1001     <- índice más alto + 1
System.out.println(bits.size());        // 1024     <- capacidad real, múltiplo de 64
```

La distinción entre `length()` y `size()` importa: `length()` es lógica (hasta dónde llega el contenido) y `size()` es física (cuánta memoria hay reservada, siempre redondeada a múltiplos de 64, porque por dentro es un `long[]`).

Las operaciones de conjunto son los mismos operadores, ahora como métodos que **mutan el receptor**:

```java
BitSet otro = new BitSet();
otro.set(100);
otro.set(200);

BitSet copia = (BitSet) bits.clone();
copia.and(otro);
System.out.println(copia);    // {100}

copia = (BitSet) bits.clone();
copia.or(otro);
System.out.println(copia);    // {3, 100, 200, 1000}

copia = (BitSet) bits.clone();
copia.xor(otro);
System.out.println(copia);    // {3, 200, 1000}
```

El `clone()` no es opcional: `and`, `or`, `xor` y `andNot` **modifican el objeto sobre el que se llaman** y devuelven `void`. Es la trampa de esta clase, y la razón de que en código con estilo inmutable haya que copiar siempre antes de operar.

Recorrerlo se hace con un stream de índices, no de booleanos:

```java
System.out.println(bits.stream().boxed().toList());   // [3, 100, 1000]
```

**Cuándo usar `BitSet`.** El argumento es la memoria: un `boolean[]` gasta **un byte por elemento** (la JVM no empaqueta booleanos en arrays), mientras que `BitSet` gasta un bit. Para diez millones de banderas eso es 10 MB frente a 1,25 MB. Los casos típicos:

- Cribas y algoritmos sobre rangos de enteros grandes (la criba de Eratóstenes es el ejemplo de manual).
- Marcar visitados en grafos grandes.
- Filtros de Bloom y estructuras probabilísticas.
- Índices invertidos: qué documentos contienen qué término.

**Cuándo no.** Si los bits son pocos o dispersos, un `HashSet<Integer>` o un `EnumSet` se leen mejor y la diferencia de memoria es irrelevante. Y `BitSet` **no es thread-safe**; para concurrencia hay que sincronizar o usar `AtomicLongArray`.

## 41. Las utilidades de Integer y Long

Casi todos los "trucos de bits" que circulan por internet ya están implementados en la JDK, con nombre y con la implementación óptima para la plataforma. Muchos compilan a **una sola instrucción de máquina** vía intrínsecos del JIT.

```java
System.out.println(Integer.bitCount(255));                //  8   <- cuántos bits en uno
System.out.println(Integer.bitCount(-1));                 // 32
System.out.println(Integer.highestOneBit(100));           // 64   <- mayor potencia de 2 que cabe
System.out.println(Integer.lowestOneBit(12));             //  4   <- aísla el bit más bajo
System.out.println(Integer.numberOfLeadingZeros(1));      // 31
System.out.println(Integer.numberOfTrailingZeros(8));     //  3   <- log2 de una potencia de 2
System.out.println(Integer.reverse(1));                   // -2147483648   <- invierte el orden
System.out.println(Integer.reverseBytes(1));              // 16777216      <- cambia el endianness
System.out.println(Integer.rotateLeft(1, 1));             //  2
System.out.println(Integer.rotateRight(1, 1));            // -2147483648
System.out.println(Long.bitCount(-1L));                   // 64
```

La tabla de para qué sirve cada uno:

| Método | Uso real |
|---|---|
| `bitCount` | contar flags activos, distancia de Hamming, popcount en índices |
| `highestOneBit` | redondear hacia abajo a potencia de dos; calcular la capacidad de una tabla hash |
| `lowestOneBit` | aislar el bit menos significativo; recorrer flags uno a uno |
| `numberOfTrailingZeros` | logaritmo en base 2 de una potencia de dos; convertir un flag a su índice |
| `numberOfLeadingZeros` | cuántos bits ocupa el número; también log2 |
| `reverseBytes` | conversión entre *big-endian* y *little-endian* al leer formatos binarios |
| `rotateLeft` / `rotateRight` | funciones de hash y de cifrado (los rotores de muchas primitivas criptográficas) |
| `toBinaryString` / `toHexString` | depurar |

`Integer.numberOfTrailingZeros` merece una mención aparte porque resuelve un problema muy concreto: **convertir un flag en su índice**.

```java
int flag = 1 << 5;
System.out.println(Integer.numberOfTrailingZeros(flag));   // 5
```

Y `lowestOneBit` combinado con XOR permite recorrer los flags activos uno a uno sin probar los 32:

```java
int restantes = permisos;
while (restantes != 0) {
    int flag = Integer.lowestOneBit(restantes);
    procesar(flag);
    restantes ^= flag;         // apaga el que acabamos de procesar
}
```

Ese bucle da exactamente tantas vueltas como flags activos haya, no 32. Es el idiom estándar para iterar máscaras densas.

**La recomendación general:** antes de escribir un truco de bits a mano, buscá si `Integer` o `Long` ya lo tienen. La versión de la JDK es más rápida (intrínseco), correcta en los casos límite (cero, `MIN_VALUE`, negativos) y, sobre todo, tiene un nombre que explica la intención. `Integer.bitCount(x)` frente a la secuencia de cinco líneas de máscaras y desplazamientos que hace lo mismo no es una cuestión de rendimiento, es una cuestión de que la segunda es ilegible.

## 42. Aritmética sin signo en un lenguaje sin tipos sin signo

Java es de los pocos lenguajes de su familia sin tipos enteros sin signo. Fue una decisión deliberada de James Gosling para simplificar el lenguaje, y es la que más se le ha discutido, porque la interoperación con formatos binarios, protocolos de red y criptografía trabaja constantemente con valores sin signo.

La solución de Java 8 fue añadir **métodos** que reinterpretan los mismos bits sin signo, sin añadir tipos:

```java
System.out.println(Integer.toUnsignedString(-1));         // 4294967295
System.out.println(Integer.toUnsignedLong(-1));           // 4294967295
System.out.println(Integer.compareUnsigned(-1, 1));       // 1    <- sin signo, -1 es enorme
System.out.println(Integer.compare(-1, 1));               // -1   <- con signo, -1 es pequeño
System.out.println(Integer.divideUnsigned(-1, 2));        // 2147483647
System.out.println(Integer.remainderUnsigned(-1, 3));     // 0
System.out.println(Integer.parseUnsignedInt("4294967295")); // -1
```

Las dos líneas del medio son la clave del asunto: **el mismo patrón de bits, dos órdenes distintos**. `-1` es el menor `int` con signo y el mayor sin signo. Cuál de los dos es el correcto depende enteramente de qué representaban esos bits, y el tipo no lo dice.

Esa última línea también es característica: `parseUnsignedInt("4294967295")` devuelve `-1`. No es un error. Los bits son los correctos; lo que pasa es que Java no tiene ningún tipo capaz de imprimirlos como `4294967295` sin ayuda, y por eso existe `toUnsignedString`.

El catálogo completo por tipo:

| Método | `Byte` | `Short` | `Integer` | `Long` |
|---|---|---|---|---|
| `toUnsignedInt` | sí | sí | — | — |
| `toUnsignedLong` | sí | sí | sí | — |
| `toUnsignedString` | sí | sí | sí | sí |
| `compareUnsigned` | sí | sí | sí | sí |
| `divideUnsigned` | — | — | sí | sí |
| `remainderUnsigned` | — | — | sí | sí |
| `parseUnsignedInt` / `parseUnsignedLong` | — | — | sí | sí |

**Las reglas prácticas para trabajar con datos sin signo en Java:**

1. **Subí de tamaño en cuanto puedas.** Un `byte` sin signo cabe holgadamente en un `int`; un `int` sin signo cabe en un `long`. `Byte.toUnsignedInt(b)` y `Integer.toUnsignedLong(i)` resuelven la mayoría de los casos y a partir de ahí todo es aritmética normal.
2. **Si tenés que quedarte en el tamaño original**, usá los métodos `*Unsigned` para comparar y dividir. Sumar, restar y multiplicar **funcionan igual con signo y sin signo** —los bits resultantes son los mismos—, así que solo la comparación, la división y la impresión necesitan versión especial.
3. **Nunca compares directamente** un valor sin signo con `<` o `>` sin haberlo ampliado. Es el bug de esta sección.

## 43. XOR: propiedades y usos reales

`^` merece sección propia porque tiene propiedades algebraicas que los otros no tienen, y de ellas salen casi todos sus usos.

```java
System.out.println(12345 ^ 12345);          // 0      <- x ^ x = 0
System.out.println(12345 ^ 0);              // 12345  <- x ^ 0 = x
System.out.println(((7 ^ 19) ^ 19) == 7);   // true   <- (a ^ b) ^ b = a
```

Las tres propiedades, con nombre:

- **Es su propio inverso.** `x ^ x == 0` y `(a ^ b) ^ b == a`. Aplicar XOR dos veces con el mismo valor devuelve el original.
- **El cero es la identidad.** `x ^ 0 == x`.
- **Es conmutativo y asociativo.** El orden y la agrupación no importan.

De la primera sale el **uso más elegante**: encontrar el elemento que no está repetido.

```java
static int unico(int[] nums) {
    int acc = 0;
    for (int n : nums) acc ^= n;
    return acc;
}

System.out.println(unico(new int[]{4, 1, 2, 1, 2}));   // 4
```

Cada par idéntico se cancela contra sí mismo y queda solo el impar. Un recorrido, memoria constante, sin ordenar y sin `HashSet`. Es un problema clásico de entrevista y una de las pocas veces en que el truco es genuinamente el mejor algoritmo.

De las tres juntas sale la **comparación en tiempo constante** de la [sección 17](#17-cuándo-sí-quieres-el-operador-que-no-cortocircuita): `a ^ b` es cero exactamente cuando son iguales, y acumular con `|=` no permite salir antes.

Y de la propiedad de "alternar" sale su uso en máscaras: `p ^= FLAG` invierte ese flag sin necesidad de consultarlo primero.

**Dos usos que hay que descartar activamente.**

**El intercambio con XOR:**

```java
// NO
a ^= b; b ^= a; a ^= b;

// SÍ
int tmp = a; a = b; b = tmp;
```

El truco funciona, ahorra una variable, y **falla catastróficamente si `a` y `b` son la misma variable o el mismo elemento de un array** (`swap(v, i, i)` deja el valor en cero). Además es más lento que la versión con variable temporal en cualquier procesador moderno, porque encadena tres dependencias de datos donde el compilador podía usar registros libres. Ahorra 4 bytes de pila y cuesta un bug. Es un mal negocio bien conocido.

**El "cifrado" con XOR:**

```java
// Esto NO es cifrado
byte cifrado = (byte) (dato ^ clave);
```

XOR con una clave repetida es el cifrado de Vigenère, roto desde el siglo XIX. Se descifra por análisis de frecuencias en minutos. Aparece disfrazado de "ofuscación" en código real y siempre es un error de seguridad. XOR es un **ingrediente** de la criptografía moderna (los cifrados de flujo hacen XOR contra un flujo pseudoaleatorio impredecible, y ahí sí es seguro), pero XOR contra una constante no es nada. Para cifrar de verdad: `javax.crypto` con AES-GCM.

## 44. Trucos de bits que valen la pena y los que no

El folclore de los trucos de bits es enorme y la mayoría envejeció mal. La pregunta útil no es "¿funciona?" sino "¿lo entiende quien lo lea dentro de un año, y gana algo medible?".

**Los que sí valen la pena**, porque son legibles una vez conocidos y no tienen alternativa mejor:

**1. Comprobar si un número es potencia de dos.**

```java
static boolean esPotenciaDeDos(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

Verificado: `0 -> false`, `1 -> true`, `2 -> true`, `3 -> false`, `16 -> true`, `64 -> true`, `100 -> false`.

La idea es que una potencia de dos tiene exactamente un bit encendido, y restarle uno pone a uno todos los de la derecha y apaga ese: `1000 & 0111 == 0`. El `n > 0` **no es opcional**: sin él, `0` daría `true` y `Integer.MIN_VALUE` también.

**2. Máscara de los n bits bajos.**

```java
int mascara = (1 << n) - 1;     // válido para n de 0 a 31
long mascaraL = (1L << n) - 1;  // válido para n de 0 a 63
```

Con el aviso de la [sección 34](#34-la-distancia-de-desplazamiento-se-toma-módulo-32-o-64): la versión con `int` falla en silencio para `n = 32`.

**3. Punto medio sin desbordar** (`(low + high) >>> 1`), que ya salió en el capítulo anterior y es la solución que adoptó la propia JDK.

**4. Módulo por una potencia de dos.**

```java
int resto = x & (n - 1);     // equivale a x % n, solo si n es potencia de dos Y x >= 0
```

Con `x` negativo **no** equivale a `%`: da el resultado de `Math.floorMod`, siempre positivo. Eso a veces es exactamente lo que querías (es lo que hacen las tablas hash de la JDK), pero es un cambio de semántica que hay que conocer. Y el JIT ya convierte `x % 16` en esta operación cuando puede demostrar que es seguro, así que escribirlo a mano rara vez aporta.

**Los que no valen la pena:**

- **`x >> 1` en lugar de `x / 2`.** Cambia el comportamiento con negativos ([sección 33](#33-los-dos-mitos-de-los-desplazamientos)) y el JIT ya hace la conversión cuando es correcta. Se pierde legibilidad y no se gana nada.
- **`x << 3` en lugar de `x * 8`.** Lo mismo.
- **Multiplicar por constantes con sumas de desplazamientos** (`(x << 3) + (x << 1)` por `x * 10`). Optimización de compiladores de los ochenta. El JIT lo hace mejor.
- **El intercambio con XOR** ([sección 43](#43-xor-propiedades-y-usos-reales)).
- **Valor absoluto sin rama** (`(x ^ (x >> 31)) - (x >> 31)`). Es correcto y evita un salto, pero `Math.abs` ya compila a un intrínseco sin rama. Escribirlo a mano solo añade una línea que nadie entiende.
- **Contar bits a mano.** `Integer.bitCount` es una instrucción del procesador.

**El criterio.** Los trucos de bits se justifican cuando **expresan algo que las alternativas no expresan** —una máscara, un flag, un formato binario, una propiedad de potencias de dos— y no cuando reescriben aritmética normal en una notación más corta. La segunda categoría es toda micro-optimización basada en un modelo de procesador de hace treinta años, y hoy el JIT la deshace o la mejora sin ayuda.

---

# Parte VI — Cierre

## 45. Casos de uso reales

**Caso 1: comprobar un permiso sin caer en ninguna de las trampas del capítulo.**

```java
public enum Permiso { LECTURA, ESCRITURA, EJECUCION, BORRADO }

public record Usuario(String id, Set<Permiso> permisos) {
    public Usuario {
        permisos = permisos == null ? Set.of() : Set.copyOf(permisos);
    }

    public boolean puede(Permiso requerido) {
        return permisos.contains(requerido);
    }

    public boolean puedeTodo(Permiso... requeridos) {
        return permisos.containsAll(List.of(requeridos));
    }
}
```

Sin máscaras, sin `== 1`, sin flags que se puedan confundir con otro enum, y la copia defensiva en el constructor compacto hace el conjunto inmutable.

**Caso 2: validar la entrada de una API con `NaN` incluido.**

```java
public record Coordenada(double latitud, double longitud) {
    public Coordenada {
        if (!Double.isFinite(latitud) || !Double.isFinite(longitud)) {
            throw new IllegalArgumentException("coordenada no finita: " + latitud + ", " + longitud);
        }
        if (!(latitud >= -90 && latitud <= 90)) {
            throw new IllegalArgumentException("latitud fuera de rango: " + latitud);
        }
        if (!(longitud >= -180 && longitud <= 180)) {
            throw new IllegalArgumentException("longitud fuera de rango: " + longitud);
        }
    }
}
```

Dos decisiones deliberadas: `isFinite` primero descarta `NaN` e infinitos de un golpe, y las comprobaciones de rango están escritas **en positivo y luego negadas con `!`**, no reescritas con De Morgan. Así siguen siendo correctas aunque un `NaN` se colara.

**Caso 3: comparar un token de sesión.**

```java
public boolean tokenValido(String recibido, String esperado) {
    if (recibido == null || esperado == null) return false;

    byte[] a = recibido.getBytes(StandardCharsets.UTF_8);
    byte[] b = esperado.getBytes(StandardCharsets.UTF_8);
    return MessageDigest.isEqual(a, b);
}
```

Nada de `==`, nada de `equals`. Tiempo constante, porque el valor es un secreto.

**Caso 4: leer una cabecera de un protocolo binario.**

```java
public record Cabecera(int version, int tipo, int longitud) {

    public static Cabecera parsear(byte[] datos) {
        if (datos.length < 4) {
            throw new IllegalArgumentException("cabecera incompleta: " + datos.length + " bytes");
        }
        int b0 = Byte.toUnsignedInt(datos[0]);
        int b1 = Byte.toUnsignedInt(datos[1]);
        int b2 = Byte.toUnsignedInt(datos[2]);
        int b3 = Byte.toUnsignedInt(datos[3]);

        return new Cabecera(
                (b0 >>> 4) & 0x0F,          // nibble alto del primer byte
                b0 & 0x0F,                  // nibble bajo
                (b1 << 16) | (b2 << 8) | b3 // entero de 24 bits, big-endian
        );
    }
}
```

Cada byte pasa por `toUnsignedInt` **antes** de participar en cualquier desplazamiento o combinación. Si se omitiera, un byte con el bit alto encendido contaminaría con unos toda la parte alta del `int` y la longitud saldría negativa.

**Caso 5: un comparador que no miente.**

```java
List<Producto> productos = new ArrayList<>(inventario);

productos.sort(Comparator
        .comparing(Producto::categoria)
        .thenComparingInt(Producto::stock)
        .thenComparing(Producto::nombre, String.CASE_INSENSITIVE_ORDER));
```

`comparing` para objetos, `thenComparingInt` para el primitivo (sin boxing), y un `Comparator` explícito para el texto en vez de confiar en el orden natural, que es sensible a mayúsculas.

**Caso 6: la criba de Eratóstenes con `BitSet`.**

```java
public static BitSet primosHasta(int n) {
    BitSet compuestos = new BitSet(n + 1);
    for (int i = 2; (long) i * i <= n; i++) {
        if (!compuestos.get(i)) {
            for (int j = i * i; j <= n; j += i) {
                compuestos.set(j);
            }
        }
    }
    BitSet primos = new BitSet(n + 1);
    primos.set(2, n + 1);
    primos.andNot(compuestos);
    return primos;
}
```

Un bit por número en vez de un byte, `(long) i * i` para que el cuadrado no desborde con `n` grande, y `andNot` en vez de un bucle de negación.

## 46. Anti-patrones

**1. Comparar objetos con `==`.**

```java
// MAL
if (nombreUsuario == "admin") { }
// BIEN
if ("admin".equals(nombreUsuario)) { }
if (Objects.equals(nombreUsuario, "admin")) { }
```

**2. Comparar dos wrappers con `==`.** Funciona hasta 127 y falla después. Y el límite depende de un flag de JVM.

**3. Comparar arrays con `equals` u `Objects.equals`.** Ninguno compara contenido. Es `Arrays.equals`.

**4. Comparadores escritos como resta.**

```java
// MAL
lista.sort((a, b) -> a.getEdad() - b.getEdad());
// BIEN
lista.sort(Comparator.comparingInt(Persona::getEdad));
```

**5. Validaciones de rango escritas en negativo sobre `double`.** `!(v < min || v > max)` acepta `NaN`.

**6. Cambiar `&&` por `&` "porque da igual".** No da igual: el cortocircuito es lo que evita el NPE.

**7. Poner efectos secundarios en el operando derecho de `&&` o `||`.** Se ejecutan o no según un valor que muchas veces no controlás.

**8. Mezclar operadores lógicos sin paréntesis.** `a ^ b | c` y `a ^ (b | c)` dan resultados opuestos.

**9. `if (flag == true)`.** Redundante, y abre la puerta a `if (flag = true)`, que compila y siempre entra.

**10. Ternario con ramas de tipos distintos.** `condicion ? 1 : 2.0` siempre devuelve un `double`, aunque se tome la rama del `1`.

**11. Ternario que mezcla un primitivo y un wrapper que puede ser `null`.** Lanza NPE, pero solo cuando la condición cae del lado del wrapper.

**12. Ternarios anidados en la condición.** `a ? (b ? x : y) : (c ? z : w)` es un árbol, no una tabla.

**13. Usar un `byte` sin enmascarar como número.**

```java
// MAL
if (datos[0] == 0xFF) { }               // siempre false: -1 no es 255
// BIEN
if (Byte.toUnsignedInt(datos[0]) == 0xFF) { }
```

**14. Comprobar una máscara con `== 1`.** Funciona solo para el primer flag.

**15. Acumular bits en un `byte` o un `short`.** `b <<= 1` trunca sin avisar por el cast oculto.

**16. `1 << n` donde el resultado va a un `long`.** Hay que escribir `1L << n`; si no, la operación se hace en 32 bits y `n` se toma módulo 32.

**17. Asumir que `>>` divide por 2^n.** Falso con negativos. Lo que equivale es `Math.floorDiv`.

**18. Asumir que `>>>` siempre devuelve un positivo.** Con distancia 0 (o múltiplo de 32) devuelve el valor intacto, signo incluido.

**19. Reescribir aritmética como desplazamientos "para optimizar".** El JIT ya lo hace, y mejor.

**20. Intercambiar dos variables con XOR.** Falla cuando son la misma variable, y no es más rápido.

**21. "Cifrar" con XOR contra una clave fija.** No es cifrado.

**22. Escribir a mano lo que `Integer` ya tiene.** `bitCount`, `highestOneBit`, `numberOfTrailingZeros` y compañía son intrínsecos del JIT.

**23. Comparar secretos con `equals`.** Filtra información por tiempo. Es `MessageDigest.isEqual`.

**24. Flags con `int` donde cabía un `EnumSet`.** Se gana medio nanosegundo y se pierden los tipos, los logs legibles y la imposibilidad de equivocarse con la máscara.

## 47. Checklist y tabla de decisión

**Antes de dar por terminada una condición, revisá:**

- [ ] ¿Hay algún `==` cuyos dos lados sean objetos? → `equals` u `Objects.equals`, salvo enums o `null`.
- [ ] ¿Hay algún `==` entre dos wrappers? → cambiar a primitivos, o `equals`.
- [ ] ¿Se comparan arrays? → `Arrays.equals` o `Arrays.deepEquals`.
- [ ] ¿Algún wrapper que pueda ser `null` participa en `==`, `<`, `>` o en un ternario? → guarda previa o `getOrDefault`.
- [ ] ¿Se compara un `double` con `==`? → tolerancia, `Double.compare`, o cambiar de tipo.
- [ ] ¿Puede llegar `NaN` a una comparación? → `Double.isFinite` primero, y validar en positivo.
- [ ] ¿Hay un comparador escrito como resta? → `Comparator.comparingInt` o `Integer.compare`.
- [ ] ¿Cada `&&` protege a lo que viene después? → verificar que el orden es el correcto.
- [ ] ¿Hay algún `&` o `|` sobre booleanos? → confirmar que es a propósito y dejar un comentario.
- [ ] ¿Se mezclan operadores lógicos de distinto tipo sin paréntesis? → ponerlos.
- [ ] ¿Hay algún `== true` o `== false`? → quitarlo.
- [ ] ¿Las dos ramas del ternario tienen el mismo tipo? → si no, castear o pasar a `if`.
- [ ] ¿Algún `byte` que venga de fuera se usa como número? → `Byte.toUnsignedInt`.
- [ ] ¿Alguna máscara se comprueba con `== 1`? → `!= 0` o `== MASCARA`.
- [ ] ¿Algún acumulador de bits es `byte` o `short`? → `int` o `long`.
- [ ] ¿Hay un `1 << n` cuyo resultado va a un `long`? → `1L << n`.
- [ ] ¿Se compara algún secreto? → `MessageDigest.isEqual`.

**Tabla de decisión: cómo comparo esto**

| Quiero comparar | Uso | No uso |
|---|---|---|
| Dos primitivos numéricos | `==`, `<`, `>` | — |
| Dos `double` | `Double.compare` o tolerancia | `==` |
| Dos objetos por contenido | `Objects.equals` | `==` |
| Dos wrappers | `equals`, o pasar a primitivos | `==` |
| Dos enums | `==` | `equals` (funciona, pero sobra) |
| Dos arrays | `Arrays.equals` | `equals`, `Objects.equals` |
| Dos `BigDecimal` como importes | `compareTo(x) == 0` | `equals` |
| Con `null` | `== null` | `equals(null)` |
| Un secreto | `MessageDigest.isEqual` | `equals` |
| Para ordenar | `Comparator.comparingInt` | `(a, b) -> a - b` |
| El tipo de un objeto | `instanceof Tipo t` | `getClass() == ...` en general |

**Tabla de decisión: qué operador uso**

| Quiero | Uso | No uso |
|---|---|---|
| Combinar condiciones con guarda | `&&`, `\|\|` | `&`, `\|` |
| Evaluar ambos lados a propósito | `&`, `\|` con comentario | `&&` "arreglado" |
| Exactamente uno de dos | `^` o `!=` | dos `if` anidados |
| Elegir entre dos valores | ternario | `if` con variable mutable |
| Elegir entre muchos casos de un enum | `switch` como expresión | cadena de ternarios |
| Guardar muchos booleanos | `EnumSet` | flags con `int` |
| Guardar millones de booleanos | `BitSet` | `boolean[]` |
| Leer un byte sin signo | `Byte.toUnsignedInt` | `b & 0xFF` sin explicación |
| Dividir por una potencia de dos | `/` o `Math.floorDiv` | `>>` |
| Contar bits | `Integer.bitCount` | bucle a mano |
| Comparar sin signo | `Integer.compareUnsigned` | `<` sobre los bits |
| Intercambiar dos variables | variable temporal | XOR |

## 48. Fuentes

**Documentación oficial**

- [JLS §15.21 — Equality Operators](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.21) — la definición de `==` y `!=`, incluida la regla de tipos incomparables y la de cuándo se desempaqueta.
- [JLS §15.22.2 — Boolean Logical Operators](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.22.2) — la sección que demuestra que `&`, `^` y `|` sobre booleanos son operadores lógicos por derecho propio, no operadores de bits reutilizados.
- [JLS §15.23 y §15.24 — Conditional-And y Conditional-Or](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.23) — el cortocircuito como parte de la especificación, no como optimización.
- [JLS §15.25 — Conditional Operator](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.25) — la tabla completa que determina el tipo de un ternario. Es la referencia que explica por qué `true ? 1 : 2.0` da `1.0`.
- [`Integer.valueOf(int)` — JDK 25 API](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Integer.html#valueOf(int)) — donde está escrito el "may cache other values outside of this range" que hace que `==` dependa de la configuración de la JVM.
- [JEP 394: Pattern Matching for instanceof](https://openjdk.org/jeps/394) — la versión **definitiva**, en Java 16. Los JEP [305](https://openjdk.org/jeps/305) y [375](https://openjdk.org/jeps/375) son los preview de 14 y 15.
- [JEP 358: Helpful NullPointerExceptions](https://openjdk.org/jeps/358) — por qué desde Java 14 el NPE dice qué método se intentó invocar.
- [`MessageDigest.isEqual`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/security/MessageDigest.html#isEqual(byte%5B%5D,byte%5B%5D)) — el javadoc explica explícitamente la propiedad de tiempo constante.
- [`EnumSet`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/EnumSet.html) y [`BitSet`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/BitSet.html) — incluido el detalle de que `EnumSet` se implementa como un vector de bits.

**Las tres fuentes de referencia de este capítulo, y dónde se equivocan**

- [Jenkov — Java Tutorial](https://jenkov.com/tutorials/java/index.html), en concreto [Operations](https://jenkov.com/tutorials/java/operations.html), [Ternary Operator](https://jenkov.com/tutorials/java/ternary-operator.html) e [instanceof](https://jenkov.com/tutorials/java/instanceof.html). **Hueco importante:** el tutorial **no tiene ninguna página sobre operadores bitwise ni sobre operadores lógicos**; `operations.html` cubre solo aritmética y asignación, y la URL `operators.html` que circula en muchos índices devuelve 404. Es la mitad del tema sin cubrir. **Error 1:** la página de `instanceof` presenta el pattern matching como introducido "en Java 14", sin aclarar que en 14 y 15 era *preview*. El propio compilador lo desmiente: `javac --release 15` responde `pattern matching in instanceof is not supported in -source 15 (use -source 16 or higher)`. Es definitivo en Java 16 (JEP 394). **Error 2:** el ejemplo central de la página del ternario es `String name = case.equals("uppercase") ? "JOHN" : "john";`, y `case` es una **palabra reservada** de Java. Copiado tal cual produce ocho errores de compilación, empezando por `not a statement` y `orphaned case`. **Desactualización:** el ejemplo de casting del ternario (`theObj instanceof Integer ? ((Integer) theObj).intValue() : value`) escribe a mano el cast que el pattern matching hace innecesario desde Java 16.
- [W3Schools — Java Operators](https://www.w3schools.com/java/java_operators.asp), con sus subpáginas de [comparación](https://www.w3schools.com/java/java_operators_comparison.asp) y [lógicos](https://www.w3schools.com/java/java_operators_logical.asp). Las tablas son cómodas para consultar los símbolos y poco más. **Hueco 1:** la página principal anuncia "Bitwise operators" como una de las cinco categorías, pero **no existe ninguna página de bitwise en el tutorial de Java**: `java_bitwise.asp` y `java_operators_bitwise.asp` devuelven 404, y el enlace lleva a una [página genérica multi-lenguaje](https://www.w3schools.com/programming/prog_operators_bitwise.php) que ni siquiera menciona `>>>`, que es justamente el operador característico de Java. **Hueco 2:** la tabla de lógicos lista solo `&&`, `||` y `!`; omite por completo `&`, `|` y `^` sobre booleanos, y **no menciona el cortocircuito en ningún momento**, que es la única propiedad que distingue a esos operadores de sus gemelos. **Hueco 3:** la página de comparación presenta `==` como "Equal to" sin una sola palabra sobre referencias frente a valores. Un principiante que aprenda `==` ahí escribirá `if (nombre == "admin")` y no tendrá forma de saber por qué falla. Es la omisión más cara del tema.
- [Baeldung — Java Bitwise Operators](https://www.baeldung.com/java-bitwise-operators), [Java Operators](https://www.baeldung.com/java-operators) y [Bitwise & vs Logical &&](https://www.baeldung.com/java-bitwise-vs-logical-and). Es la más completa de las tres y la única que cubre el tema entero con ejemplos ejecutables. Aun así tiene tres afirmaciones falsas. **Error 1 (el más grave):** en la sección del complemento afirma que `~` *"takes only one integer and it's equivalent to the `!` operator"*. No lo es en ningún sentido: no comparten tipos, no comparten semántica, y aplicar `~` a un booleano ni siquiera compila — `error: bad operand type boolean for unary operator '~'`. La analogía induce a un error conceptual sobre qué hace el operador. **Error 2:** sobre `>>>` afirma que *"the result will always be a positive integer"*. Falso: `-12 >>> 0` da `-12` y `-12 >>> 32` da `-12`, porque la distancia se toma módulo 32. La afirmación correcta exige acotar la distancia a 1–31. **Error 3:** en `java-operators` afirma que *"n >> x has the same effect of dividing the number n by x power of two"*. Falso con negativos: `-1 >> 1` da `-1` mientras que `-1 / 2` da `0`, y `-3 >> 1` da `-2` mientras que `-3 / 2` da `-1`. Su propio ejemplo (`-12 >> 2 = -3`) coincide solo porque -12 es múltiplo de 4. La equivalencia real es con `Math.floorDiv`. **Imprecisión adicional:** presenta el complemento de `6` como una operación de 8 bits (`0000 0110 -> 1111 1001`), cuando la operación real es sobre 32 bits; el resultado (-7) es correcto pero la representación oculta justamente el problema de promoción de la [sección 31](#31-promoción-por-qué-la-máscara-0xff-es-obligatoria). **Y un hueco de organización:** `java-operators` titula la sección 5 "Logical Operators" y afirma *"We have two logical operators in Java"*, dejando fuera `&`, `|` y `^` sobre booleanos, que la JLS clasifica exactamente ahí; además menciona el cortocircuito solo al hablar de `||` y no de `&&`.

**Discusiones y artículos de la comunidad** (consultados para contrastar los casos límite)

- [Why does 128 == 128 return false but 127 == 127 return true?](https://stackoverflow.com/questions/1700081/why-does-128-128-return-false-but-127-127-return-true-when-converting-to-integer) — el hilo canónico sobre la caché de wrappers.
- [Comparison method violates its general contract](https://stackoverflow.com/questions/8327514/comparison-method-violates-its-general-contract) — qué dispara la excepción de `TimSort` y por qué aparece solo con ciertos tamaños de lista.
- [Java Integer Cache — Baeldung](https://www.baeldung.com/java-integer-cache) — incluye el efecto de `-XX:AutoBoxCacheMax`.
- [Timing attacks and string comparison](https://codahale.com/a-lesson-in-timing-attacks/) — por qué `equals` sobre secretos es una vulnerabilidad, con la medición.
- [Bit Twiddling Hacks](https://graphics.stanford.edu/~seander/bithacks.html) — la colección de referencia de trucos de bits. Vale la pena conocerla y vale la pena resistirse a usarla: casi todo lo que contiene está hoy en `Integer` o lo hace el JIT.
- [The Java Language Specification on conditional expressions](https://stackoverflow.com/questions/12154877/what-is-the-difference-between-integer-and-int-in-a-ternary-operator) — el NPE del ternario explicado desde la especificación.

**Nota sobre la verificación.** Todos los outputs de este documento se obtuvieron ejecutando el código en Temurin JDK 25.0.3 (`java Archivo.java`), los volcados de bytecode con `javap -c`, y los mensajes de error compilando a propósito archivos que fallan (`javac`). Las cifras de la comparación entre `EnumSet` y flags de la [sección 39](#39-enumset-la-alternativa-que-casi-siempre-gana) son una medición propia con calentamiento previo pero **sin JMH**: sirven como orden de magnitud, no como benchmark riguroso.

## 49. Autoevaluación

**1. ¿Por qué `Integer a = 127, b = 127; a == b` es `true` pero con 128 es `false`?**

<details><summary>Respuesta</summary>

Porque el autoboxing llama a `Integer.valueOf`, que mantiene una caché de instancias para el rango `-128..127`. Dentro del rango devuelve siempre el mismo objeto, y `==` sobre referencias compara identidad, así que da `true`. Fuera del rango crea un objeto nuevo cada vez y la identidad falla. Peor todavía: el límite superior es configurable con `-XX:AutoBoxCacheMax`, así que el mismo bytecode puede dar respuestas distintas en dos JVM. La solución es `equals`, o mejor usar `int` en vez de `Integer`.
</details>

**2. ¿Qué diferencia hay entre `s != null && s.length() > 0` y `s != null & s.length() > 0`?**

<details><summary>Respuesta</summary>

La primera es correcta y la segunda lanza `NullPointerException` cuando `s` es `null`. `&&` tiene cortocircuito: si el operando izquierdo es `false`, el derecho **no se evalúa**, y eso está garantizado por la JLS §15.23, no es una optimización. `&` sobre booleanos evalúa siempre ambos operandos, así que `s.length()` se ejecuta sobre una referencia nula. El cortocircuito es semántica, no rendimiento.
</details>

**3. ¿Qué imprime `System.out.println(true ? 1 : 2.0);` y por qué?**

<details><summary>Respuesta</summary>

Imprime `1.0`. El ternario es una expresión y su tipo se calcula a partir de **las dos ramas**, no de la que se ejecuta. Como una rama es `int` y la otra `double`, se aplica promoción numérica binaria y el tipo del ternario completo es `double`. El `1` de la rama tomada se promociona. Es el mismo mecanismo que hace que `true ? Integer.valueOf(1) : Double.valueOf(2.0)` devuelva un `Double`.
</details>

**4. ¿Por qué `Integer nulo = null; Integer x = false ? 1 : nulo;` lanza NPE, pero con `true` no?**

<details><summary>Respuesta</summary>

Porque una rama es `int` y la otra `Integer`, así que por promoción numérica el tipo del ternario es `int`. Con `false` se toma `nulo`, y convertirlo a `int` exige llamar a `intValue()` sobre `null`. Con `true` se toma el literal `1`, que ya es un `int`, y no hay nada que desempaquetar. Es un NPE que depende del valor de la condición en tiempo de ejecución: pasa los tests y falla en producción. Se evita haciendo que ambas ramas sean del mismo tipo de referencia.
</details>

**5. Una validación escribe `return !(v < min || v > max);` sobre un `double`. ¿Qué problema tiene?**

<details><summary>Respuesta</summary>

Acepta `NaN`. Todas las comparaciones de orden con `NaN` devuelven `false`, así que `v < min` es `false`, `v > max` es `false`, el `||` da `false` y el `!` lo convierte en `true`: el valor inválido pasa la validación. La versión equivalente en positivo, `v >= min && v <= max`, devuelve `false` para `NaN` y lo rechaza correctamente. Es el único punto de Java donde las leyes de De Morgan no se pueden aplicar sin más, porque las comparaciones sobre `NaN` no forman un álgebra booleana.
</details>

**6. ¿Qué está mal en `lista.sort((a, b) -> a - b)`?**

<details><summary>Respuesta</summary>

La resta puede desbordar. `Integer.MIN_VALUE - Integer.MAX_VALUE` da `1`, un valor positivo, así que el comparador afirma que `MIN_VALUE` es mayor que `MAX_VALUE`. La lista sale mal ordenada **sin lanzar ninguna excepción**; a veces `TimSort` lo detecta y lanza `IllegalArgumentException: Comparison method violates its general contract!`, pero solo con ciertos tamaños y distribuciones. La forma correcta es `Integer::compare` o `Comparator.comparingInt`, que no restan.
</details>

**7. ¿Por qué `~` no es el equivalente entero de `!`?**

<details><summary>Respuesta</summary>

Porque son operadores de tipos disjuntos con semánticas distintas. `~` solo acepta enteros e invierte los 32 bits, produciendo `-x - 1`; `!` solo acepta booleanos e intercambia `true` y `false`. Aplicar `~` a un `boolean` no compila (`bad operand type boolean for unary operator '~'`) y `!` a un `int` tampoco. Y la analogía "invierte el valor" es falsa donde importa: `~0` es `-1`, no `1`. La confusión viene de C, donde no hay tipo booleano y `~0` es "verdadero".
</details>

**8. ¿Es cierto que `x >> 1` equivale a dividir por 2?**

<details><summary>Respuesta</summary>

Solo con números no negativos. Con negativos difieren, porque `/` trunca hacia cero y `>>` redondea hacia menos infinito: `-3 >> 1` da `-2` mientras que `-3 / 2` da `-1`, y `-1 >> 1` da `-1` mientras que `-1 / 2` da `0`. La equivalencia correcta es con `Math.floorDiv(x, 2)`, que coincide con `>>` siempre. Además no hay ninguna razón para escribirlo: el JIT convierte `x / 2` en la secuencia óptima por su cuenta.
</details>

**9. ¿Por qué `-12 >>> 2` da 1073741821 pero `-12 >>> 0` da -12?**

<details><summary>Respuesta</summary>

`>>>` rellena por la izquierda con ceros, así que con una distancia entre 1 y 31 el bit de signo queda a cero y el resultado es no negativo: los 32 bits de `-12` reinterpretados sin signo y desplazados dan `1073741821`. Con distancia 0 no se mueve ningún bit y el valor sale intacto, signo incluido. Lo mismo pasa con distancia 32, porque la distancia se toma módulo 32. Por eso la afirmación "`>>>` siempre da positivo" es falsa: solo lo es para distancias de 1 a 31.
</details>

**10. ¿Qué imprime `System.out.println(1 << 32);` y por qué?**

<details><summary>Respuesta</summary>

Imprime `1`. La distancia de desplazamiento se toma módulo 32 para `int` (solo se usan los 5 bits bajos), así que `32 % 32 = 0` y no se desplaza nada. Con `long` el módulo es 64, así que `1L << 32` sí da `4294967296`. De ahí sale el bug de `long x = 1 << 40`, que opera en 32 bits y da `256` en vez de `1099511627776`: la `L` tiene que ir en el operando izquierdo, no en la variable de destino.
</details>

**11. ¿Por qué `byte b = (byte) 0xFF; if (b == 0xFF)` es siempre `false`?**

<details><summary>Respuesta</summary>

Porque `byte` tiene signo y va de `-128` a `127`, así que `(byte) 0xFF` vale `-1`. Al comparar con el literal `0xFF`, que es un `int` de valor `255`, el `byte` se promociona a `int` **con extensión de signo** y sigue valiendo `-1`. `-1 == 255` es `false`. La solución es enmascarar los 8 bits bajos: `(b & 0xFF) == 0xFF`, o mejor `Byte.toUnsignedInt(b) == 0xFF`, que dice lo que hace.
</details>

**12. ¿Qué está mal en `if ((permisos & ESCRITURA) == 1)`?**

<details><summary>Respuesta</summary>

Que `permisos & ESCRITURA` no devuelve `1` cuando el flag está activo: devuelve **el valor del flag**. Si `ESCRITURA` vale `2`, el resultado es `2` y la comparación con `1` da `false` aunque el permiso esté concedido. La forma correcta es `!= 0`, o `== ESCRITURA` si se quiere ser explícito (obligatorio si la máscara agrupa varios bits). El bug es especialmente traicionero porque funciona para el primer flag, que vale `1`.
</details>

**13. ¿Por qué `a ^ b | c` y `a ^ (b | c)` pueden dar resultados distintos?**

<details><summary>Respuesta</summary>

Porque `^` tiene más precedencia que `|`, así que `a ^ b | c` se agrupa como `(a ^ b) | c`. El orden completo entre los no cortocircuitados es `&`, luego `^`, luego `|`. Con `a = true, b = false, c = true`, la primera expresión da `true` y la segunda `false`. Como los dos lados son booleanos, el compilador no puede detectar nada y la expresión compila con el significado equivocado. La defensa es poner paréntesis siempre que se mezclen operadores lógicos.
</details>

**14. ¿Por qué `x & y == 1` no compila, y qué se quería escribir?**

<details><summary>Respuesta</summary>

Porque `==` tiene más precedencia que `&`, así que la expresión se agrupa como `x & (y == 1)`: un `int` y un `boolean`, tipos que `&` no acepta (`bad operand types for binary operator '&'`). Se quería escribir `(x & y) == 1`. En C el mismo código compila y produce un resultado sin sentido; Java lo convierte en un error de compilación gracias a la separación estricta entre `boolean` y los enteros. Pero la protección desaparece si los dos lados son booleanos, como en la pregunta anterior.
</details>

**15. ¿Cuándo conviene un `EnumSet` en vez de flags con `int`?**

<details><summary>Respuesta</summary>

Casi siempre. `EnumSet` usa internamente un `long` de bits (`RegularEnumSet`), así que la representación es la misma; medido, `contains` cuesta unos 0,14 nanosegundos frente a los 0,04 de una máscara. A cambio se gana seguridad de tipos (no se puede mezclar con otro enum ni con un entero cualquiera), logs legibles (`[LECTURA, EJECUCION]` en vez de `5`), imposibilidad de equivocarse con la máscara, y la API completa de `Set`. Los flags con `int` solo se justifican cuando un formato binario externo los impone o cuando hay una medición concreta que lo respalde.
</details>

**16. ¿Por qué comparar un token de sesión con `equals` es una vulnerabilidad?**

<details><summary>Respuesta</summary>

Porque `String.equals` recorre los caracteres y **vuelve en cuanto encuentra una diferencia**, así que el tiempo de respuesta depende de cuántos caracteres iniciales acertó el atacante. Midiendo esas diferencias se reconstruye el secreto carácter a carácter, en tiempo lineal en vez de exponencial. La defensa es una comparación en tiempo constante que recorra siempre todo el array acumulando diferencias con `^` y `|=` sin ramificar. La JDK ya la trae: `MessageDigest.isEqual`.
</details>

**17. ¿Qué hace `acc ^= n` dentro de un bucle sobre `{4, 1, 2, 1, 2}` y por qué?**

<details><summary>Respuesta</summary>

Devuelve `4`, el único elemento sin pareja. Funciona por tres propiedades de XOR: `x ^ x == 0` (cada par se cancela), `x ^ 0 == x` (el cero es la identidad) y la conmutatividad y asociatividad (el orden no importa, así que los pares se cancelan aunque estén separados). Es un recorrido, memoria constante, sin ordenar y sin estructuras auxiliares. Es uno de los pocos trucos de bits que además es el mejor algoritmo para el problema.
</details>

**18. ¿Por qué `Arrays.equals(a1, a2)` da `true` donde `Objects.equals(a1, a2)` da `false`?**

<details><summary>Respuesta</summary>

Porque los arrays **no sobrescriben `equals`**: heredan el de `Object`, que compara identidad. `Objects.equals` termina delegando en ese `equals` heredado, así que tampoco compara contenido. Solo `Arrays.equals` (o `Arrays.deepEquals` para arrays anidados o de objetos) compara elemento a elemento. Lo mismo aplica a `hashCode` y `toString`, y tiene una consecuencia práctica: un `record` con un componente de tipo array compara ese componente por identidad, así que dos records con arrays de idéntico contenido son distintos.
</details>

**19. ¿Qué hace `if (flag = true)` y por qué compila?**

<details><summary>Respuesta</summary>

Asigna `true` a `flag` y entra siempre en el `if`. Compila porque el valor de la expresión `flag = true` es `true`, un `boolean`, que es exactamente lo que espera un `if`. Con enteros el mismo error se bloquea (`if (n = 1)` da `incompatible types: int cannot be converted to boolean`), pero con booleanos Java no puede distinguirlo de una asignación intencionada. La defensa es no escribir nunca `== true`: usando `if (flag)` el error no se puede teclear. Declarar `final` la variable también lo convierte en error de compilación.
</details>

**20. ¿Por qué `n > 0 && (n & (n - 1)) == 0` detecta potencias de dos, y para qué sirve el `n > 0`?**

<details><summary>Respuesta</summary>

Una potencia de dos tiene exactamente un bit encendido. Restarle uno apaga ese bit y enciende todos los que estaban a su derecha (`1000 - 1 = 0111`), así que el AND de ambos da cero. Cualquier otro número tiene al menos otro bit que sobrevive al AND. El `n > 0` no es opcional: sin él, `0` daría `true` (porque `0 & -1 == 0`) e `Integer.MIN_VALUE` también (tiene un solo bit encendido, el de signo, y `MIN_VALUE & MAX_VALUE == 0`). Los dos son casos límite que aparecen en cuanto el valor viene de fuera.
</details>
