# Loops

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** El capítulo anterior, [Conditionals](10-conditionals.md), rompió la línea recta del programa hacia los lados: permitió elegir entre caminos. Este la rompe hacia atrás: permite **volver sobre el mismo código**. Con condicionales y bucles ya está completo el control de flujo estructurado; todo lo demás que hace un programa se construye encima de estas dos ideas.

Un **bucle** ejecuta un bloque de código repetidamente mientras se cumpla una condición. Suena tan simple como el `if`, y como el `if`, casi todo el mundo lo aprende en veinte minutos y arrastra los huecos durante años.

Los huecos importan por una razón distinta a la de las condicionales. Una condición mal escrita elige la rama equivocada **una vez**. Un bucle mal escrito repite el error **miles de veces**, y los tres modos de fallo que tiene son de los peores que existen: no termina nunca, procesa un elemento de más o de menos, o degrada de O(n) a O(n²) sin que nada en el código lo delate.

La lista de bugs concretos que salen de este capítulo, todos reales:

- Un `for` seguido de un punto y coma que ejecuta su cuerpo una sola vez, y que el compilador acepta sin avisar.
- Un `for (int i = 0; i <= v.length; i++)` que lanza `ArrayIndexOutOfBoundsException` en el último elemento.
- Un `for (int i = 0; i <= Integer.MAX_VALUE; i++)` que **nunca** termina, por mucho que la condición parezca finita.
- Un `for (byte i = 0; i < 128; i++)` que tampoco termina, por la misma razón y con un tipo distinto.
- Un bucle que un tutorial muy visitado llama "infinito" y que en realidad **termina tras 2.147.483.650 vueltas**, dato que este capítulo mide.
- Un bucle anidado cuyo contador interno no se reinicia, y cuyo bloque interior solo se ejecuta en la primera vuelta.
- Un `ConcurrentModificationException` al borrar de una lista mientras se recorre.
- El mismo borrado que **no** lanza ninguna excepción y deja la lista mal, porque el elemento era el penúltimo.
- Un `list.get(i)` dentro de un `for` que convierte un recorrido en O(n²) al cambiar `ArrayList` por `LinkedList`, sin tocar una línea del bucle.
- Un `resultado += texto` dentro de un bucle que crea un objeto nuevo en cada vuelta.
- Un `break` dentro de un `forEach` que no compila, y que una fuente muy leída describe como "lanza una excepción".

Vamos a cubrir el modelo completo: las cuatro formas de repetir que tiene Java, cómo se controla un bucle desde dentro, los modos de fallo del contador, qué genera realmente el compilador cuando escribís un `for-each`, qué pasa cuando modificás una colección mientras la recorrés, por qué un bucle puede volverse cuadrático sin cambiar de forma, en qué se diferencia un `stream` de un bucle, y —lo más importante para el día a día— **cuándo el bucle sobra**.

**Sobre la verificación.** Todos los outputs, mensajes de excepción, errores del compilador y volcados de bytecode de este documento se ejecutaron realmente en un JDK 25. Las cinco fuentes de referencia usadas para prepararlo contienen entre todas **cuatro afirmaciones falsas comprobables**, incluida una que confunde dos excepciones distintas y otra que llama "excepción" a lo que es un error de compilación. Está todo señalado en la [sección 52](#52-fuentes) con el resultado real al lado.

---

## Índice

**Parte I — Las cuatro formas de repetir**

1. [Qué es un bucle y qué problema resuelve](#1-qué-es-un-bucle-y-qué-problema-resuelve)
2. [Las cuatro formas y qué las distingue](#2-las-cuatro-formas-y-qué-las-distingue)
3. [El while](#3-el-while)
4. [El do-while](#4-el-do-while)
5. [El for clásico](#5-el-for-clásico)
6. [Las tres partes del for son opcionales](#6-las-tres-partes-del-for-son-opcionales)
7. [El scope de la variable de control](#7-el-scope-de-la-variable-de-control)
8. [El for-each](#8-el-for-each)
9. [Cuál elegir](#9-cuál-elegir)

**Parte II — Controlar el bucle desde dentro**

10. [break](#10-break)
11. [continue](#11-continue)
12. [Bucles anidados](#12-bucles-anidados)
13. [Etiquetas](#13-etiquetas)
14. [return dentro de un bucle](#14-return-dentro-de-un-bucle)
15. [El bucle infinito deliberado](#15-el-bucle-infinito-deliberado)

**Parte III — Los modos de fallo del contador**

16. [Off-by-one](#16-off-by-one)
17. [El punto y coma fantasma](#17-el-punto-y-coma-fantasma)
18. [Modificar la variable de control dentro del cuerpo](#18-modificar-la-variable-de-control-dentro-del-cuerpo)
19. [Desbordamiento del contador](#19-desbordamiento-del-contador)
20. [El bucle infinito que no lo es](#20-el-bucle-infinito-que-no-lo-es)
21. [Contadores que no se reinician](#21-contadores-que-no-se-reinician)
22. [Comparar con distinto en vez de con menor](#22-comparar-con-distinto-en-vez-de-con-menor)
23. [Contadores decimales](#23-contadores-decimales)

**Parte IV — El for-each por dentro**

24. [Qué genera el compilador](#24-qué-genera-el-compilador)
25. [Iterable e Iterator](#25-iterable-e-iterator)
26. [La variable del for-each es una copia](#26-la-variable-del-for-each-es-una-copia)
27. [Lo que el for-each no puede hacer](#27-lo-que-el-for-each-no-puede-hacer)
28. [Recorrer un Map](#28-recorrer-un-map)

**Parte V — Modificar mientras se itera**

29. [ConcurrentModificationException](#29-concurrentmodificationexception)
30. [Fail-fast no es una garantía](#30-fail-fast-no-es-una-garantía)
31. [El caso que no lanza la excepción](#31-el-caso-que-no-lanza-la-excepción)
32. [Iterator.remove](#32-iteratorremove)
33. [removeIf](#33-removeif)
34. [Recorrer hacia atrás](#34-recorrer-hacia-atrás)
35. [Copiar antes de recorrer](#35-copiar-antes-de-recorrer)
36. [Colecciones concurrentes](#36-colecciones-concurrentes)

**Parte VI — Rendimiento**

37. [El bucle cuadrático escondido](#37-el-bucle-cuadrático-escondido)
38. [Concatenar texto dentro de un bucle](#38-concatenar-texto-dentro-de-un-bucle)
39. [Trabajo repetido dentro del bucle](#39-trabajo-repetido-dentro-del-bucle)
40. [Localidad de caché](#40-localidad-de-caché)
41. [Por qué este capítulo no trae cifras](#41-por-qué-este-capítulo-no-trae-cifras)

**Parte VII — Streams frente a bucles**

42. [forEach no es un bucle](#42-foreach-no-es-un-bucle)
43. [Iteración interna frente a externa](#43-iteración-interna-frente-a-externa)
44. [Cómo se sale de un stream](#44-cómo-se-sale-de-un-stream)
45. [Qué hace mejor cada uno](#45-qué-hace-mejor-cada-uno)

**Parte VIII — Diseño**

46. [Cuándo el bucle sobra](#46-cuándo-el-bucle-sobra)
47. [Bucles legibles](#47-bucles-legibles)

**Parte IX — Cierre**

48. [Casos de uso reales](#48-casos-de-uso-reales)
49. [Anti-patrones](#49-anti-patrones)
50. [Checklist y tabla de decisión](#50-checklist-y-tabla-de-decisión)
51. [Autoevaluación](#51-autoevaluación)
52. [Fuentes](#52-fuentes)

---

# Parte I — Las cuatro formas de repetir

## 1. Qué es un bucle y qué problema resuelve

Supongamos que hay que imprimir un mensaje cinco veces. Sin bucles:

```java
System.out.println("Java es divertido");
System.out.println("Java es divertido");
System.out.println("Java es divertido");
System.out.println("Java es divertido");
System.out.println("Java es divertido");
```

Funciona, y ya tiene tres problemas. Cambiar el mensaje obliga a editar cinco líneas. Imprimirlo mil veces es inviable. Y lo decisivo: **el número de repeticiones tiene que conocerse mientras se escribe el programa**, no mientras se ejecuta. Si el número viene de un fichero o de una petición HTTP, esta técnica no sirve para nada.

Un bucle resuelve las tres cosas:

```java
int n = 5;
for (int i = 1; i <= n; ++i) {
    System.out.println("Java es divertido");
}
```

Salida:

```
Java es divertido
Java es divertido
Java es divertido
Java es divertido
Java es divertido
```

Cambiar `n` cambia el comportamiento sin tocar el bucle, y `n` puede venir de donde sea.

La definición formal, que conviene tener precisa: **un bucle ejecuta repetidamente un bloque de sentencias mientras una expresión booleana de control se evalúe a `true`**. Fijate en la palabra *booleana*: como en el `if`, y por las mismas razones que vimos en el [capítulo anterior](10-conditionals.md), la condición de un bucle en Java tiene que ser un `boolean`. No hay valores *truthy*.

Todo bucle tiene, explícita o implícitamente, tres piezas:

1. **Inicialización** — el estado con el que arranca.
2. **Condición de continuación** — se evalúa antes (o después) de cada vuelta.
3. **Avance** — lo que cambia el estado para que la condición acabe siendo falsa.

Si falta la tercera, o si no acerca el estado al final, el bucle no termina. La mayoría de los bugs de la Parte III son variantes de esa única frase.

## 2. Las cuatro formas y qué las distingue

Java tiene cuatro construcciones de repetición:

| Forma | Sintaxis | Cuándo comprueba | Se ejecuta al menos una vez |
|---|---|---|---|
| `while` | `while (cond) { }` | Antes | No |
| `do-while` | `do { } while (cond);` | Después | **Sí** |
| `for` clásico | `for (init; cond; avance) { }` | Antes | No |
| `for-each` | `for (T x : coleccion) { }` | Antes | No |

Las dos preguntas que las separan:

**¿Sabés de antemano cuántas vueltas hay que dar?** Si el número de repeticiones se conoce o se puede calcular antes de empezar, el `for` es la forma natural, porque reúne las tres piezas en una sola línea. Si la repetición depende de algo que solo se sabe durante el bucle —una respuesta del usuario, líneas de un fichero, un reintento hasta que funcione—, el `while` es la forma natural.

**¿El cuerpo tiene que ejecutarse al menos una vez?** Solo el `do-while` lo garantiza.

Hay una quinta forma de repetir que **no es un bucle** y que merece mención desde ya: `forEach` y el Stream API. Se parecen mucho a un bucle por fuera y son otra cosa por dentro; la Parte VII los trata en detalle.

## 3. El while

Repite mientras la condición sea verdadera. La comprueba **antes** de cada vuelta, incluida la primera.

```java
while (condicion) {
    // cuerpo
}
```

Ejemplo:

```java
int i = 0;
while (i < 5) {
    System.out.println(i);
    i++;
}
```

Salida:

```
0
1
2
3
4
```

Fijate en las tres piezas de la sección 1, ahora separadas: la inicialización (`int i = 0`) está **antes** del bucle, la condición está en el `while`, y el avance (`i++`) está **dentro** del cuerpo. Esa dispersión es la debilidad del `while` como contador, y la razón por la que existe el `for`.

**El aviso que toda fuente repite, y con razón:** si te olvidás del `i++`, la condición nunca cambia y el bucle no termina. Es el bucle infinito más frecuente del mundo.

```java
int i = 0;
while (i < 5) {
    System.out.println(i);   // falta i++ — no termina nunca
}
```

**Si la condición es falsa desde el principio, el cuerpo no se ejecuta ni una vez:**

```java
int i = 10;
while (i < 5) {
    System.out.println("esto no se imprime nunca");
    i++;
}
```

No imprime nada. Es el comportamiento correcto y es justo lo que distingue al `while` del `do-while`.

**Dónde brilla el `while`:** cuando el final no se puede contar de antemano.

```java
// Leer líneas hasta que se acabe el fichero
String linea;
while ((linea = lector.readLine()) != null) {
    procesar(linea);
}

// Reintentar hasta que funcione o se agoten los intentos
int intentos = 0;
while (!conectado && intentos < MAX_INTENTOS) {
    conectado = intentarConectar();
    intentos++;
}
```

El primer ejemplo usa un idioma que conviene reconocer: la asignación dentro de la condición. `(linea = lector.readLine()) != null` asigna **y** compara en una expresión. Es legal porque el resultado de la comparación es un `boolean`, y los paréntesis externos son obligatorios por precedencia. Es de los pocos sitios donde una condición con efecto colateral está aceptada por convención, aunque la alternativa sin efectos —leer antes del bucle y al final del cuerpo— sea más explícita.

**Sobre el nombre `i`.** Es un convenio, no una regla: viene de *index* o *iterator*, y arrastra desde la notación matemática de sumatorios y desde Fortran. Para bucles anidados se sigue con `j` y `k`. Está bien para índices sin significado propio; en cuanto la variable signifique algo (`fila`, `intento`, `posicionActual`), poné el nombre real.

## 4. El do-while

Idéntico al `while` salvo en una cosa: comprueba la condición **después** de ejecutar el cuerpo. Por lo tanto el cuerpo se ejecuta **siempre al menos una vez**.

```java
do {
    // cuerpo
} while (condicion);
```

El punto y coma final es obligatorio, y es la única de las cuatro formas que lo lleva. Olvidarlo es un error de sintaxis inmediato.

```java
int i = 0;
do {
    System.out.println(i);
    i++;
} while (i < 5);
```

Salida: `0 1 2 3 4`, igual que el `while` equivalente.

La diferencia se ve cuando la condición es falsa desde el principio. Verificado en JDK 25:

```java
int i = 10;
do {
    System.out.println("i es " + i);
    i++;
} while (i < 5);
```

Salida real:

```
I1 se ejecuta una vez aunque i=10 y la condicion i<5 es falsa
```

Un `while` no habría impreso nada. El `do-while` imprime una vez y luego comprueba.

**Dónde se usa de verdad.** El `do-while` es la forma menos frecuente de las cuatro, y tiene un nicho concreto: cuando la condición **depende de algo que solo existe después de ejecutar el cuerpo**.

```java
// Pedir datos hasta que sean válidos: hay que pedirlos al menos una vez
int opcion;
do {
    System.out.println("Elegí una opción (1-3):");
    opcion = scanner.nextInt();
} while (opcion < 1 || opcion > 3);

// Menú: se muestra siempre, y se repite hasta que el usuario salga
String eleccion;
do {
    mostrarMenu();
    eleccion = leerEleccion();
    procesar(eleccion);
} while (!eleccion.equals("salir"));
```

En ambos casos, con un `while` habría que duplicar el cuerpo antes del bucle para que la condición tuviera algo que evaluar. Esa duplicación es la señal de que lo que hacía falta era un `do-while`.

## 5. El for clásico

El `for` reúne las tres piezas —inicialización, condición y avance— en una sola línea:

```java
for (inicializacion; condicion; avance) {
    // cuerpo
}
```

Ejemplo:

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

Salida: `0 1 2 3 4`.

**El orden exacto de ejecución**, que conviene tener claro porque explica varios comportamientos:

1. Se ejecuta la **inicialización**, una única vez.
2. Se evalúa la **condición**. Si es falsa, el bucle termina.
3. Se ejecuta el **cuerpo**.
4. Se ejecuta el **avance**.
5. Se vuelve al paso 2.

Nótese que el avance ocurre **después** del cuerpo, no antes. Y que la condición se evalúa una vez más de las que se ejecuta el cuerpo: para cinco vueltas, la condición se evalúa seis veces.

La tabla de una ejecución completa de `for (int i = 1; i <= 3; ++i)`:

| Vuelta | `i` al evaluar | Condición `i <= 3` | Acción |
|---|---|---|---|
| 1 | 1 | `true` | ejecuta cuerpo, luego `i` pasa a 2 |
| 2 | 2 | `true` | ejecuta cuerpo, luego `i` pasa a 3 |
| 3 | 3 | `true` | ejecuta cuerpo, luego `i` pasa a 4 |
| 4 | 4 | `false` | termina |

Variantes habituales del avance:

```java
for (int i = 0; i <= 10; i = i + 2) { }   // de dos en dos
for (int i = 5; i > 0; i--) { }           // cuenta atrás
for (int i = 1; i <= n; i *= 2) { }       // potencias de dos
```

Y como en el `while`, si la condición es falsa desde el principio el cuerpo no se ejecuta:

```java
for (int i = 10; i < 5; i++) {
    System.out.println("nunca se imprime");
}
```

**`i++` frente a `++i`.** En la posición de avance de un `for` son **exactamente equivalentes**, porque el valor de la expresión se descarta. La diferencia entre pre y post incremento solo importa cuando el resultado se usa (`int b = a++;` frente a `int b = ++a;`), y eso está en [Math Operations](07-math-operations.md). Verás las dos formas en tutoriales distintos; no hay ninguna razón de rendimiento para preferir una.

## 6. Las tres partes del for son opcionales

Las tres secciones del `for` pueden omitirse. Los dos puntos y coma, no.

```java
for (;;) {
    // bucle infinito: sin condición, se asume true
}
```

Otras combinaciones legales:

```java
int i = 0;
for (; i < 5; i++) { }        // inicialización fuera

for (int i = 0; i < 5; ) {    // avance dentro del cuerpo
    System.out.println(i);
    i++;
}

for (int i = 0, j = 10; i < j; i++, j--) { }   // varias variables, con coma
```

Ese último merece un apunte. La coma dentro de un `for` **no es el operador coma de C**: es una sintaxis específica del `for` que permite listar varios inicializadores o varias expresiones de avance. Las variables declaradas en la inicialización tienen que ser **del mismo tipo**; declarar un `int` y un `String` en la misma cabecera no compila.

**`for (;;)` frente a `while (true)`.** Son equivalentes en comportamiento y en bytecode. `while (true)` se lee mejor y es lo que usa la mayoría del código moderno; `for (;;)` sobrevive por tradición de C. Hay una diferencia real y curiosa que conecta con el capítulo anterior:

```java
while (false) { System.out.println("hola"); }   // NO COMPILA
if (false)    { System.out.println("hola"); }   // compila sin problema
```

Verificado en JDK 25:

```
F4.java:3: error: unreachable statement
        while (false) { System.out.println("hola"); }
                      ^
```

Y el `if` equivalente compila sin un solo aviso. La razón es deliberada: el análisis de alcanzabilidad del JLS trata el `if` como caso especial **precisamente** para que funcione el idioma de la compilación condicional (`if (DEBUG) { ... }` con `DEBUG` constante), donde eliminar el bloque es lo deseado. En un `while`, código inalcanzable siempre es un error del programador.

## 7. El scope de la variable de control

Cuando la variable se declara dentro del `for`, **existe solo dentro del bucle**:

```java
for (int i = 0; i < 3; i++) { }
System.out.println(i);   // NO COMPILA
```

Verificado:

```
F3.java:4: error: cannot find symbol
        System.out.println(i);
                           ^
  symbol:   variable i
  location: class F3
```

Cuando se declara fuera, sobrevive al bucle y conserva el valor con el que terminó. Verificado en JDK 25:

```java
int j;
for (j = 0; j < 3; j++) { }
System.out.println(j);   // imprime 3
```

Salida real:

```
F1 declarada fuera, sobrevive: j=3
```

Fijate en que vale **3**, no 2: el bucle terminó porque el avance llevó `j` a 3 y ahí la condición falló. El último valor de la variable de control es siempre el primero que **no** cumple la condición.

**La regla de estilo:** declarala dentro siempre que puedas. Limitar el alcance evita que se reutilice por accidente y permite usar el mismo nombre en dos bucles seguidos. Declarala fuera solo cuando necesites su valor final, que es exactamente el caso de las búsquedas:

```java
int posicion;
for (posicion = 0; posicion < v.length; posicion++) {
    if (v[posicion] == buscado) break;
}
if (posicion == v.length) {
    System.out.println("no encontrado");
}
```

Aunque para eso, casi siempre es mejor extraer un método que devuelva el índice.

## 8. El for-each

El `for-each` —oficialmente *enhanced for statement*, y en la jerga *for mejorado*— recorre todos los elementos de un array o una colección sin contador:

```java
for (Tipo elemento : coleccion) {
    // cuerpo
}
```

Se lee "para cada elemento de la colección". Los dos puntos se leen como "en".

```java
String[] coches = {"Volvo", "BMW", "Ford", "Mazda"};

for (String coche : coches) {
    System.out.println(coche);
}
```

Salida:

```
Volvo
BMW
Ford
Mazda
```

Comparado con el `for` clásico equivalente:

```java
for (int i = 0; i < coches.length; i++) {
    System.out.println(coches[i]);
}
```

El `for-each` elimina tres cosas que pueden salir mal: el índice inicial, la condición de parada y el avance. Con ellas desaparecen de golpe el off-by-one, el `ArrayIndexOutOfBoundsException` y el bucle infinito por avance olvidado. **No es azúcar sintáctico: es una construcción con menos superficie de error.**

Funciona con arrays y con cualquier cosa que implemente `Iterable`:

```java
List<String> nombres = List.of("Ana", "Luis", "Marta");
for (String n : nombres) {
    System.out.println(n);
}

Set<Integer> ids = Set.of(1, 2, 3);
for (int id : ids) {
    System.out.println(id);
}
```

Una precisión sobre una definición que circula mucho. W3Schools lo describe como *"used exclusively to loop through elements in an array (or other data structures)"*. La palabra *exclusively* es engañosa: el `for-each` no es una construcción de arrays extendida a colecciones, sino al revés — está definido sobre `Iterable`, y los arrays son el **caso especial** que el compilador trata aparte porque un array no implementa `Iterable`. La [sección 24](#24-qué-genera-el-compilador) muestra que el compilador genera **dos códigos completamente distintos** según el caso.

Y lo que **no** acepta, verificado en JDK 25:

```java
for (String k : unMap) { }   // NO COMPILA
```

```
F7.java:4: error: for-each not applicable to expression type
        for (String k : m) { System.out.println(k); }
                        ^
  required: array or java.lang.Iterable
  found:    Map<String,Integer>
```

El mensaje es exacto y didáctico: hace falta un array o un `Iterable`, y `Map` no es ninguna de las dos cosas. Cómo se recorre un `Map` está en la [sección 28](#28-recorrer-un-map).

## 9. Cuál elegir

El árbol de decisión completo, que resume la Parte I:

| Situación | Forma |
|---|---|
| Recorrer todos los elementos de una colección o array | **`for-each`** |
| Recorrer todos los elementos y necesitás el índice | `for` clásico |
| Recorrer una parte (saltando, hacia atrás, de dos en dos) | `for` clásico |
| Recorrer dos colecciones en paralelo | `for` clásico con índice |
| Modificar la colección mientras la recorrés | `Iterator` explícito o `removeIf` |
| Repetir un número conocido de veces | `for` clásico |
| Repetir hasta que pase algo | `while` |
| Repetir al menos una vez y luego comprobar | `do-while` |
| Bucle deliberadamente infinito con salida interna | `while (true)` |

> **La regla por defecto:** si el `for-each` sirve, usá el `for-each`. Es el que menos puede fallar. Solo bajá al `for` clásico cuando necesites el índice de verdad, y cuando lo hagas, preguntate primero si el índice era realmente necesario o si estabas escribiendo C con sintaxis de Java.

---

# Parte II — Controlar el bucle desde dentro

## 10. break

`break` termina el bucle inmediatamente. La ejecución continúa en la primera sentencia después del bucle.

```java
for (int i = 0; i < 10; i++) {
    if (i == 4) {
        break;
    }
    System.out.println(i);
}
```

Salida: `0 1 2 3`. Al llegar a `i == 4` sale, y el `4` no se imprime.

Funciona igual en `while` y en `do-while`:

```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
    if (i == 4) {
        break;
    }
}
```

Salida: `0 1 2 3`.

**El uso real del `break`** es la búsqueda: recorrer hasta encontrar algo y parar, porque seguir sería trabajo desperdiciado.

```java
Usuario encontrado = null;
for (Usuario u : usuarios) {
    if (u.getId().equals(idBuscado)) {
        encontrado = u;
        break;   // ya está, no tiene sentido seguir
    }
}
```

Ese patrón, con una variable declarada fuera y un `break`, es tan común que conviene conocer su alternativa más limpia: extraerlo a un método y usar `return`, que veremos en la [sección 14](#14-return-dentro-de-un-bucle).

**Ojo con `break` dentro de un `switch` dentro de un bucle.** El `break` pertenece a la construcción más interna que lo admita, así que ahí sale del `switch`, **no** del bucle:

```java
for (String cmd : comandos) {
    switch (cmd) {
        case "salir":
            break;      // sale del switch, NO del for. El bucle sigue.
        default:
            procesar(cmd);
    }
}
```

Es un bug clásico y silencioso. Las soluciones son una etiqueta ([sección 13](#13-etiquetas)) o —mucho mejor— usar la sintaxis de flecha del `switch` moderno, que no necesita `break` en absoluto, como vimos en [Conditionals](10-conditionals.md).

## 11. continue

`continue` salta el resto del cuerpo y pasa a la siguiente iteración. No termina el bucle.

```java
for (int i = 0; i < 10; i++) {
    if (i == 4) {
        continue;
    }
    System.out.println(i);
}
```

Salida: `0 1 2 3 5 6 7 8 9`. El `4` se salta; el resto continúa.

La formulación que mejor funciona para recordarlos:

- **`break`** = parar el bucle del todo.
- **`continue`** = saltarse esta vuelta y seguir con la siguiente.

**El detalle que hace daño en un `while`.** En un `for`, el avance está en la cabecera y se ejecuta igualmente tras un `continue`. En un `while`, el avance está en el cuerpo, y `continue` **se lo salta**:

```java
int i = 0;
while (i < 10) {
    if (i == 4) {
        continue;      // BUCLE INFINITO: nunca llega al i++
    }
    System.out.println(i);
    i++;
}
```

Al llegar a `i == 4`, el `continue` vuelve a la condición sin incrementar `i`, así que `i` se queda en 4 para siempre. La versión correcta incrementa antes de saltar:

```java
int i = 0;
while (i < 10) {
    if (i == 4) {
        i++;
        continue;
    }
    System.out.println(i);
    i++;
}
```

Esa duplicación del `i++` es fea y es exactamente el síntoma de que ahí lo que correspondía era un `for`.

**Cuándo usar `continue`.** Es la cláusula de guarda del capítulo anterior, aplicada a un bucle: filtrar temprano lo que no interesa y dejar el cuerpo principal sin indentar.

```java
// Con continue: plano y legible
for (Pedido p : pedidos) {
    if (p.estaCancelado()) continue;
    if (p.getTotal().signum() <= 0) continue;
    if (!p.getCliente().estaActivo()) continue;

    procesar(p);
}

// Sin continue: tres niveles de anidamiento
for (Pedido p : pedidos) {
    if (!p.estaCancelado()) {
        if (p.getTotal().signum() > 0) {
            if (p.getCliente().estaActivo()) {
                procesar(p);
            }
        }
    }
}
```

## 12. Bucles anidados

Un bucle dentro de otro. El interno se ejecuta **por completo** en cada vuelta del externo.

```java
for (int i = 1; i <= 2; i++) {
    System.out.println("Externo: " + i);
    for (int j = 1; j <= 3; j++) {
        System.out.println("  Interno: " + j);
    }
}
```

Salida:

```
Externo: 1
  Interno: 1
  Interno: 2
  Interno: 3
Externo: 2
  Interno: 1
  Interno: 2
  Interno: 3
```

El total de ejecuciones del cuerpo interno es el **producto**: 2 × 3 = 6. Esa multiplicación es lo que hay que tener presente siempre, porque es donde nace la complejidad cuadrática. Dos bucles anidados sobre la misma colección de 10.000 elementos son 100.000.000 de iteraciones.

Los usos legítimos más comunes:

```java
// Recorrer una matriz
for (int fila = 0; fila < m.length; fila++) {
    for (int col = 0; col < m[fila].length; col++) {
        System.out.print(m[fila][col] + " ");
    }
    System.out.println();
}

// Tabla de multiplicar
for (int i = 1; i <= 10; i++) {
    for (int j = 1; j <= 10; j++) {
        System.out.printf("%4d", i * j);
    }
    System.out.println();
}
```

En el recorrido de matrices, fijate en `m[fila].length` y no `m[0].length`: en Java las filas pueden tener longitudes distintas, como vimos en [Arrays](09-arrays.md).

**La señal de alarma.** Si te encontrás con dos bucles anidados sobre **colecciones distintas** buscando coincidencias, casi siempre hay una versión con un `Map` que es lineal en vez de cuadrática:

```java
// O(n*m)
for (Pedido p : pedidos) {
    for (Cliente c : clientes) {
        if (p.getClienteId().equals(c.getId())) {
            p.setCliente(c);
        }
    }
}

// O(n+m)
Map<String, Cliente> indice = clientes.stream()
        .collect(Collectors.toMap(Cliente::getId, c -> c));
for (Pedido p : pedidos) {
    p.setCliente(indice.get(p.getClienteId()));
}
```

## 13. Etiquetas

`break` y `continue` afectan al bucle **más interno** que los contiene. Cuando hace falta salir de varios niveles a la vez, Java tiene una construcción que casi nadie enseña: las **etiquetas**.

Una etiqueta es un identificador seguido de dos puntos, colocado justo antes de un bucle:

```java
externo:
for (int i = 1; i < 5; i++) {
    for (int k = 1; k < 3; k++) {
        System.out.print(i + ":" + k + " ");
        if (i == 2) break externo;
    }
}
```

Verificado en JDK 25, salida real:

```
G1 break externo -> 1:1 1:2 2:1
```

Sin la etiqueta, el `break` habría salido solo del bucle interno y el externo habría seguido hasta `i = 4`. Con ella, sale de los dos de golpe.

`continue` etiquetado salta a la **siguiente iteración del bucle etiquetado**, abandonando lo que quede del interno:

```java
primero:
for (int i = 1; i < 6; i++) {
    for (int k = 1; k < 5; k++) {
        if (i == 3 || k == 2) continue primero;
        System.out.print("i=" + i + ",k=" + k + " ");
    }
}
```

Salida real:

```
G2 continue etiquetado -> i=1,k=1 i=2,k=1 i=4,k=1 i=5,k=1
```

Cada vuelta externa imprime solo `k=1`, porque al llegar a `k=2` salta a la siguiente `i`. Y la `i=3` no aparece en absoluto.

**La etiqueta tiene que envolver al bucle.** No es un `goto`: no se puede saltar a cualquier sitio. Verificado:

```java
outer:
for (int i = 0; i < 3; i++) { }
for (int j = 0; j < 3; j++) { break outer; }   // NO COMPILA
```

```
F8.java:5: error: undefined label: outer
        for (int j = 0; j < 3; j++) { break outer; }
                                      ^
```

**Cuándo usarlas, y cuándo no.** Las etiquetas tienen mala prensa por su parecido con `goto`, y la reputación es en parte injusta: un `break` etiquetado solo puede saltar **hacia adelante y hacia afuera**, que es una forma muy restringida y perfectamente estructurada de salir.

Dicho eso, en la práctica su presencia suele indicar que el bloque debería ser un método:

```java
// Con etiqueta
boolean encontrado = false;
externo:
for (int i = 0; i < m.length; i++) {
    for (int j = 0; j < m[i].length; j++) {
        if (m[i][j] == objetivo) {
            encontrado = true;
            break externo;
        }
    }
}

// Extraído a un método: sin etiqueta, sin variable de bandera
static boolean contiene(int[][] m, int objetivo) {
    for (int[] fila : m) {
        for (int x : fila) {
            if (x == objetivo) return true;
        }
    }
    return false;
}
```

La segunda versión es más corta, más fácil de testear y no necesita ninguna construcción exótica. **Regla práctica: antes de escribir una etiqueta, probá a extraer un método.** Si después de intentarlo la etiqueta sigue siendo lo más claro —pasa en parsers y en recorridos de matrices con estado—, usala sin complejos.

## 14. return dentro de un bucle

`return` sale del **método entero**, y por tanto también del bucle, sin importar cuántos niveles haya.

```java
static Usuario buscar(List<Usuario> usuarios, String id) {
    for (Usuario u : usuarios) {
        if (u.getId().equals(id)) {
            return u;      // sale del bucle Y del método
        }
    }
    return null;
}
```

Comparado con la versión con `break` y variable externa:

```java
static Usuario buscar(List<Usuario> usuarios, String id) {
    Usuario encontrado = null;
    for (Usuario u : usuarios) {
        if (u.getId().equals(id)) {
            encontrado = u;
            break;
        }
    }
    return encontrado;
}
```

La primera es mejor: una variable menos, un estado menos que seguir, y la intención se lee en la línea donde ocurre. Es el mismo argumento de las cláusulas de guarda del capítulo anterior — **salir temprano y a menudo**.

Devolver `null` no es ideal; `Optional<Usuario>` comunica mejor la ausencia:

```java
static Optional<Usuario> buscar(List<Usuario> usuarios, String id) {
    for (Usuario u : usuarios) {
        if (u.getId().equals(id)) {
            return Optional.of(u);
        }
    }
    return Optional.empty();
}
```

Y el `finally` sigue ejecutándose aunque salgas con `return` desde dentro del bucle, que es lo que hace seguros los `try-with-resources` alrededor de bucles.

## 15. El bucle infinito deliberado

No todos los bucles infinitos son bugs. Algunos son el diseño correcto:

```java
while (true) {
    Peticion p = cola.tomar();     // bloquea hasta que haya algo
    procesar(p);
}
```

Un servidor, un consumidor de una cola de mensajes, un bucle de eventos de una interfaz gráfica: todos son bucles que **no deben terminar** mientras el proceso viva. La salida ocurre por otra vía: un `break` condicionado, un `return`, una excepción, o la parada del proceso.

Las tres formas equivalentes:

```java
while (true) { }
for (;;) { }
do { } while (true);
```

Un detalle que conecta con el análisis del compilador: si un método termina en un bucle infinito sin salida, **no hace falta `return`**, porque el compilador sabe que el final del método es inalcanzable.

**La regla para un bucle infinito deliberado:** que se vea que es deliberado. `while (true)` con un `break` claro y comentado se entiende; un bucle que resulta infinito por accidente en el avance es un bug. Y en cualquier bucle que espere trabajo, hay que asegurarse de que la espera **bloquee** en vez de consumir CPU: `cola.take()` bloquea, `while (cola.isEmpty()) { }` es un *busy-wait* que quema un núcleo entero sin hacer nada.

---

# Parte III — Los modos de fallo del contador

Todos los bugs de esta parte tienen la misma raíz: **la relación entre el estado inicial, la condición y el avance está mal**. Se presentan por separado porque cada uno se detecta de una forma distinta.

## 16. Off-by-one

El error más frecuente de la programación, y tiene nombre propio: *off-by-one*, error por uno. Procesar un elemento de más o uno de menos.

```java
int[] v = {10, 20, 30};

for (int i = 0; i <= v.length; i++) {   // MAL: <= en vez de <
    System.out.println(v[i]);
}
```

Resultado: imprime `10 20 30` y luego lanza

```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3
```

La causa está en que **los índices van de `0` a `length - 1`**. Un array de 3 elementos tiene índices 0, 1 y 2; el índice 3 no existe. La condición correcta es `i < v.length`.

El error simétrico es más silencioso, porque no lanza nada:

```java
for (int i = 0; i < v.length - 1; i++) {   // MAL: se salta el último
    System.out.println(v[i]);
}
```

Imprime `10 20`. El `30` se pierde y nadie se entera.

Y el tercero, empezar en 1:

```java
for (int i = 1; i < v.length; i++) {   // MAL: se salta el primero
    System.out.println(v[i]);
}
```

**Las plantillas correctas**, que conviene tener memorizadas:

```java
for (int i = 0; i < n; i++)         // los n elementos, hacia delante
for (int i = n - 1; i >= 0; i--)    // los n elementos, hacia atrás
for (int i = 1; i <= n; i++)        // contar de 1 a n (no es un índice)
```

La primera y la segunda son índices; la tercera es un contador de vueltas y por eso usa `<=`. Confundir ambos usos es el origen de la mitad de los off-by-one.

**La defensa real no es tener cuidado: es no usar índices.**

```java
for (int x : v) {
    System.out.println(x);
}
```

El `for-each` **no puede** tener un off-by-one. No hay índice que equivocar. Es el argumento más fuerte a su favor y la razón de la regla por defecto de la [sección 9](#9-cuál-elegir).

Y cuando el índice sea inevitable, recordá que `length` es un campo en arrays (`v.length`), un método en `String` (`s.length()`) y otro método distinto en colecciones (`lista.size()`). Confundirlos no compila, pero cuesta un minuto cada vez.

## 17. El punto y coma fantasma

El mismo error que vimos con el `if` en el capítulo anterior, y aquí es peor porque el síntoma despista más:

```java
for (int i = 0; i < 5; i++);
{
    System.out.println("hola");
}
```

Imprime `hola` **una sola vez**. El `;` es una sentencia vacía y se convierte en el cuerpo del `for`, así que el bucle da sus cinco vueltas sin hacer nada. El bloque de llaves de abajo no pertenece al bucle: es un bloque suelto que se ejecuta una vez.

Con un `while` el resultado es mucho peor:

```java
int i = 0;
while (i < 5);      // BUCLE INFINITO
{
    System.out.println(i);
    i++;
}
```

Aquí el programa se cuelga: el cuerpo del `while` es la sentencia vacía, `i` nunca se incrementa, y la condición nunca deja de ser verdadera.

Como en el capítulo anterior, el compilador **solo avisa si se lo pedís**, con `javac -Xlint:all`:

```
warning: [empty] empty statement after if
```

Activá `-Xlint:all` y `-Werror` en el build. Es la defensa más barata que existe contra esta familia de errores.

## 18. Modificar la variable de control dentro del cuerpo

Legal, y casi siempre un bug:

```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
    i++;                      // avanza dos veces por vuelta
}
```

Imprime `0 2 4 6 8`. El bucle avanza dos posiciones por iteración porque hay dos incrementos: el del cuerpo y el de la cabecera.

El caso realmente peligroso es cuando la modificación es condicional:

```java
for (int i = 0; i < v.length; i++) {
    if (esEspecial(v[i])) {
        i++;                  // "saltarse el siguiente"
    }
    procesar(v[i]);
}
```

Ese código puede leer fuera del array: si el último elemento es especial, `i++` lo lleva a `v.length` y `v[i]` explota. Y aunque funcione, es imposible de seguir.

**La regla:** la variable de control se modifica **solo en la cabecera del `for`**. Si necesitás saltar elementos, lo correcto es `continue` con una condición explícita, o un `while` donde el avance sea parte visible de la lógica, o —mejor— reformular el problema para que no haga falta saltar.

El caso del `while` es distinto y ahí sí es normal modificar la variable en el cuerpo, porque es el único sitio donde puede estar. Precisamente por eso el `for` es preferible cuando hay un contador: **concentra el avance en un lugar único y visible**.

## 19. Desbordamiento del contador

Este produce bucles infinitos que parecen perfectamente acotados. Verificado en JDK 25:

```java
for (int i = Integer.MAX_VALUE - 3; i <= Integer.MAX_VALUE; i++) {
    // NUNCA TERMINA
}
```

Salida real:

```
B1 INFINITO: i <= MAX_VALUE nunca es falso
```

La razón: para que el bucle termine, `i` tendría que ser **mayor** que `Integer.MAX_VALUE`, y ningún `int` puede serlo. Cuando `i` vale `MAX_VALUE` y se incrementa, desborda a `Integer.MIN_VALUE`, que también cumple `i <= MAX_VALUE`. El bucle da la vuelta para siempre.

La misma trampa con un tipo más pequeño, y es más fácil de encontrar en código real:

```java
for (byte i = 120; i < 128; i++) {
    // NUNCA TERMINA
}
```

Salida real:

```
B2 INFINITO: un byte nunca llega a 128
```

Un `byte` va de -128 a 127. Nunca vale 128, así que `i < 128` es siempre verdadero. Al pasar de 127 desborda a -128 y vuelve a subir.

**Las defensas:**

1. **Usá `int` para contadores**, salvo que tengas una razón fuerte. Los tipos pequeños ahorran memoria en arrays grandes, no en variables locales.
2. **Si el límite es cercano al máximo del tipo, usá `<` en vez de `<=`**, o subí a `long`.
3. **Para aritmética que no debe desbordar en silencio**, la JDK trae `Math.addExact`, `Math.multiplyExact` y compañía, que lanzan `ArithmeticException` en vez de dar la vuelta. Está tratado en [Math Operations](07-math-operations.md).

## 20. El bucle infinito que no lo es

Este merece sección propia porque es un **error factual de una de las fuentes de este capítulo**, y comprobarlo enseña más que aceptarlo.

Programiz presenta este código bajo el epígrafe *"Infinite for Loop"*:

```java
for (int i = 1; i <= 10; --i) {
    System.out.println("Hello");
}
```

El razonamiento aparente es que `i` empieza en 1, decrece, y `i <= 10` sigue siendo verdadero para siempre. Parece impecable.

**Pero el bucle termina.** Verificado en JDK 25:

```
A1 TERMINO tras 2147483650 vueltas en 536 ms
A2 2^31 = 2147483648
```

Dos mil ciento cuarenta y siete millones de vueltas, en poco más de medio segundo, y termina.

Por qué: `i` baja de 1 hacia los negativos hasta llegar a `Integer.MIN_VALUE` (-2.147.483.648). El siguiente `--i` **desborda** y lo lleva a `Integer.MAX_VALUE` (2.147.483.647). Y ese valor **no** cumple `i <= 10`, así que el bucle sale.

El número exacto de vueltas confirma el mecanismo: desde 1 hasta `MIN_VALUE` hay 2.147.483.649 pasos, más la vuelta final, dan las 2.147.483.650 medidas.

Tres lecciones de aquí, y la tercera es la que importa:

1. **Un bucle "infinito" por decremento sobre un `int` no es infinito**, es finito y muy largo. Con un `long` sí sería infinito a efectos prácticos.
2. **En la práctica da igual**: un bucle que tarda medio segundo en no hacer nada sigue siendo un bug. El consejo de Programiz —revisá la condición y el avance— es correcto aunque la etiqueta sea imprecisa.
3. **Merece la pena ejecutar lo que se lee.** La afirmación era plausible, venía de una fuente respetable y era falsa. Medio segundo de cómputo la desmiente.

## 21. Contadores que no se reinician

En bucles anidados, si el contador del bucle interno se declara **fuera** de los dos, no vuelve a su valor inicial en cada vuelta externa. El resultado es que el bucle interno se ejecuta completo una sola vez.

Verificado en JDK 25, replicando la estructura de un ejemplo que Programiz presenta sin señalar el problema:

```java
int i = 1, j = 1;
while (i <= 3) {
    System.out.print("[ext " + i + "] ");
    while (j <= 3) {
        if (j == 2) { j++; continue; }
        System.out.print("int" + j + " ");
        j++;
    }
    i++;
}
```

Salida real:

```
H1 [ext 1] int1 int3 [ext 2] [ext 3]
H2 el bucle interno solo corre en la PRIMERA vuelta porque j no se reinicia
```

En la primera vuelta externa, `j` sube de 1 a 4. En la segunda y la tercera, `j` ya vale 4, la condición `j <= 3` es falsa de entrada, y el bucle interno no se ejecuta en absoluto.

A veces eso es exactamente lo que se busca —recorrer dos secuencias con un solo puntero compartido, como en un *merge*— pero casi nunca. Lo que suele quererse es reiniciar:

```java
int i = 1;
while (i <= 3) {
    int j = 1;              // se reinicia en cada vuelta externa
    while (j <= 3) {
        // ...
        j++;
    }
    i++;
}
```

**Con un `for` este bug no puede existir**, porque la inicialización está pegada a la condición:

```java
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {   // j nace y muere en cada vuelta externa
        // ...
    }
}
```

Es otro argumento concreto —no estético— para preferir el `for` cuando hay contadores.

## 22. Comparar con distinto en vez de con menor

```java
for (int i = 0; i != v.length; i++) { }
```

Funciona si el avance es exactamente de uno y nadie toca `i`. Deja de funcionar en cuanto una de esas dos cosas cambia:

```java
for (int i = 0; i != 10; i += 3) { }   // NUNCA TERMINA: 0,3,6,9,12,... salta el 10
```

`i` toma los valores 0, 3, 6, 9, 12… y nunca es exactamente 10. Con `<` el bucle habría terminado igual:

```java
for (int i = 0; i < 10; i += 3) { }    // 0,3,6,9 y para
```

**La regla:** usá siempre `<` o `>` en las condiciones de bucle, nunca `!=`. Un operador de orden es robusto ante saltos, avances variables y desbordamientos; la igualdad exige que el contador aterrice justo en el valor de parada.

La excepción legítima es recorrer una estructura enlazada, donde no hay orden que comparar:

```java
for (Nodo n = cabeza; n != null; n = n.siguiente) { }
```

Ahí `!=` es lo correcto, porque la condición de parada es "llegué al final", no "pasé de cierto número".

## 23. Contadores decimales

Un contador `double` combina los problemas de los bucles con los del punto flotante:

```java
for (double x = 0.0; x != 1.0; x += 0.1) {
    System.out.println(x);
}
```

Nunca termina. `0.1` no es representable exactamente en binario, así que la suma acumula error y `x` pasa de `0.9999999999999999` a `1.0999999999999999` sin valer nunca `1.0`.

Cambiar a `<` mejora la robustez pero no elimina la sorpresa:

```java
for (double x = 0.0; x < 1.0; x += 0.1) {
    System.out.println(x);
}
```

Este sí termina, pero da **once** vueltas en vez de diez en algunos rangos, y los valores impresos son `0.7000000000000001` y similares. El número de iteraciones depende de cómo se acumule el error, que es justo lo que no querés que decida la lógica de tu programa.

**La solución es contar con enteros y derivar el decimal:**

```java
for (int i = 0; i < 10; i++) {
    double x = i / 10.0;      // exacto en el número de vueltas
    System.out.println(x);
}
```

El contador es un `int`, así que el número de iteraciones es exactamente 10, y el valor decimal se calcula a partir de él. El error de representación sigue existiendo en `x`, pero ya no controla el bucle.

Para dinero, la respuesta es `BigDecimal` o trabajar en céntimos, como vimos en [Conditionals](10-conditionals.md).

---

# Parte IV — El for-each por dentro

## 24. Qué genera el compilador

El `for-each` es azúcar sintáctico: el compilador lo traduce a otra cosa antes de generar bytecode. Y traduce a **dos cosas distintas** según lo que recorras.

**Sobre un array**, el compilador genera un bucle indexado con un índice oculto y la longitud cacheada. Verificado con `javap -c` sobre este código:

```java
static int sumaArray(int[] v) {
    int s = 0;
    for (int x : v) { s += x; }
    return s;
}
```

Bytecode real (extracto):

```
 5: arraylength
19: iaload
30: goto          10
```

Ninguna llamada a método. `arraylength` lee la longitud, `iaload` lee un elemento del array, `goto` cierra el bucle. Es exactamente lo que produciría un `for` clásico escrito a mano. **Un `for-each` sobre un array no asigna nada y no tiene coste.**

El código equivalente que escribe el compilador:

```java
int[] copia = v;
int longitud = copia.length;
for (int idx = 0; idx < longitud; idx++) {
    int x = copia[idx];
    s += x;
}
```

**Sobre un `Iterable`**, en cambio, genera un bucle con iterador:

```java
static int sumaLista(List<Integer> l) {
    int s = 0;
    for (int x : l) { s += x; }
    return s;
}
```

Bytecode real:

```
 3: invokeinterface // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
10: invokeinterface // InterfaceMethod java/util/Iterator.hasNext:()Z
19: invokeinterface // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
27: invokevirtual   // Method java/lang/Integer.intValue:()I
```

El código equivalente:

```java
Iterator<Integer> it = l.iterator();
while (it.hasNext()) {
    int x = it.next();     // con unboxing
    s += x;
}
```

**Tres conclusiones prácticas de este contraste:**

1. **Sobre arrays, el `for-each` es gratis.** No hay objeto `Iterator`, no hay llamadas virtuales.
2. **Sobre colecciones, se crea un `Iterator` por bucle.** Un objeto, y una llamada a interfaz por elemento. En la práctica es irrelevante salvo en bucles calentísimos, donde el JIT además suele eliminar la asignación por *escape analysis*.
3. **Ese `Integer.intValue()` del final es la trampa real.** Recorrer una `List<Integer>` con `int x` desempaqueta en cada vuelta. Si además la lista se construyó con *autoboxing*, hay un objeto por elemento. Para colecciones numéricas grandes y cómputo intensivo, un `int[]` o un `IntStream` evitan por completo ese trabajo.

## 25. Iterable e Iterator

Para que una clase propia funcione con `for-each`, tiene que implementar `Iterable`:

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}

public interface Iterator<E> {
    boolean hasNext();
    E next();
    default void remove() { throw new UnsupportedOperationException(); }
}
```

Solo dos métodos obligatorios. Un ejemplo completo de un rango de enteros:

```java
public class Rango implements Iterable<Integer> {
    private final int desde;
    private final int hasta;

    public Rango(int desde, int hasta) {
        if (hasta < desde) {
            throw new IllegalArgumentException("hasta no puede ser menor que desde");
        }
        this.desde = desde;
        this.hasta = hasta;
    }

    @Override
    public Iterator<Integer> iterator() {
        return new Iterator<>() {
            private int actual = desde;

            @Override
            public boolean hasNext() {
                return actual < hasta;
            }

            @Override
            public Integer next() {
                if (!hasNext()) {
                    throw new NoSuchElementException();
                }
                return actual++;
            }
        };
    }
}
```

Con eso, esto ya funciona:

```java
for (int i : new Rango(0, 5)) {
    System.out.println(i);   // 0 1 2 3 4
}
```

**Dos detalles del contrato que se saltan constantemente:**

1. **`next()` debe lanzar `NoSuchElementException`** cuando no hay más elementos, no devolver `null` ni comportarse de forma indefinida.
2. **`hasNext()` no debe tener efectos colaterales.** Puede llamarse varias veces seguidas sin avanzar. Un iterador que consume al comprobar rompe todo el código que lo use.

Y una consecuencia útil: `Iterable` es la razón por la que el `for-each` funciona igual sobre `List`, `Set`, `Queue`, tus propias clases y cualquier cosa futura. **El bucle no sabe qué está recorriendo**, y por eso el mismo código sirve para todo.

## 26. La variable del for-each es una copia

La variable declarada en el `for-each` es una **variable local nueva** a la que se copia el valor de cada elemento. Asignarle algo cambia la copia, no la colección. Verificado en JDK 25:

```java
int[] v = {1, 2, 3};
for (int x : v) { x = x * 10; }
System.out.println(Arrays.toString(v));
```

Salida real:

```
E1 array de primitivos tras x=x*10: [1, 2, 3]
```

El array no cambia. Es el error de principiante clásico: parece que se está modificando y no se modifica nada.

Con objetos hay un matiz importante, y también verificado:

```java
StringBuilder[] sb = { new StringBuilder("a"), new StringBuilder("b") };

for (StringBuilder s : sb) { s.append("!"); }        // muta el objeto
System.out.println(Arrays.toString(sb));

for (StringBuilder s : sb) { s = new StringBuilder("z"); }   // reasigna la copia
System.out.println(Arrays.toString(sb));
```

Salida real:

```
E2 mutar el objeto SI afecta: [a!, b!]
E3 reasignar NO afecta: [a!, b!]
```

**La regla completa:** lo que se copia es la **referencia**, no el objeto. Por eso:

- **Mutar el objeto apuntado sí afecta** a lo que hay en la colección (`s.append("!")`).
- **Reasignar la variable no afecta** a nada (`s = new StringBuilder("z")`).

Para **reemplazar** elementos hace falta el índice, o `replaceAll`:

```java
for (int i = 0; i < v.length; i++) { v[i] = v[i] * 10; }   // arrays
lista.replaceAll(s -> s.toUpperCase());                     // List, Java 8+
Arrays.setAll(v, i -> v[i] * 10);                           // arrays, funcional
```

## 27. Lo que el for-each no puede hacer

El `for-each` compra seguridad a cambio de capacidad. Lo que pierde:

**1. No hay índice.** No podés saber en qué posición estás, ni comparar con el elemento anterior o siguiente, ni recorrer dos colecciones en paralelo.

```java
// Necesita índice: for clásico
for (int i = 1; i < v.length; i++) {
    if (v[i] < v[i - 1]) { System.out.println("desordenado en " + i); }
}
```

Si solo necesitás un número de orden para mostrar, un contador aparte sirve y sigue siendo legible:

```java
int n = 1;
for (String linea : lineas) {
    System.out.println(n++ + ": " + linea);
}
```

**2. Solo va hacia delante, de uno en uno y hasta el final.** No hay recorrido inverso, ni saltos, ni parciales, sin recurrir a otra construcción.

**3. No se puede modificar la colección.** Es el tema de toda la Parte V.

**4. No se puede eliminar el elemento actual de forma segura.** Para eso está `Iterator.remove` ([sección 32](#32-iteratorremove)).

**5. No sirve para `Map` directamente**, como vimos en la sección 8.

Lo que **sí** conserva, y conviene decirlo porque a veces se confunde con `forEach`: **`break`, `continue` y `return` funcionan con normalidad dentro de un `for-each`**. Es un bucle del lenguaje como cualquier otro.

## 28. Recorrer un Map

`Map` no implementa `Iterable`, así que hay que recorrer alguna de sus tres vistas.

```java
Map<Integer, String> nombres = new HashMap<>();
nombres.put(1, "Larry");
nombres.put(2, "Steve");
nombres.put(3, "James");
```

**Por entradas — la forma correcta por defecto:**

```java
for (Map.Entry<Integer, String> e : nombres.entrySet()) {
    System.out.println(e.getKey() + " " + e.getValue());
}
```

**Solo claves, o solo valores:**

```java
for (Integer clave : nombres.keySet()) { }
for (String valor : nombres.values()) { }
```

**El anti-patrón que hay que reconocer:**

```java
// MAL: una búsqueda por cada clave
for (Integer clave : nombres.keySet()) {
    String valor = nombres.get(clave);   // segundo acceso innecesario
    System.out.println(clave + " " + valor);
}
```

Recorrer `keySet()` y hacer `get()` dentro duplica el trabajo: la entrada ya se había localizado al iterar. Sobre un `HashMap` cada `get` es O(1) y el coste es un factor constante; sobre un `TreeMap` cada `get` es O(log n) y el recorrido pasa de O(n) a O(n log n). Usá `entrySet()`.

**Con `forEach` y un `BiConsumer`**, disponible desde Java 8:

```java
nombres.forEach((clave, valor) -> System.out.println(clave + " " + valor));
```

`Map.forEach` no viene de `Iterable` —`Map` no lo implementa— sino que es un método propio de `Map` que acepta un `BiConsumer` en lugar de un `Consumer`, precisamente para poder recibir clave y valor a la vez.

**Un aviso sobre el orden**, que ninguna de las fuentes de este capítulo menciona y que produce tests intermitentes: el orden de recorrido depende de la implementación. `HashMap` **no garantiza ningún orden** y puede cambiarlo entre versiones de Java o al redimensionar. Si necesitás orden, la estructura tiene que dártelo: `LinkedHashMap` conserva el de inserción y `TreeMap` ordena por clave. Escribir un test que dependa del orden de un `HashMap` es escribir un test que fallará algún día.

---

# Parte V — Modificar mientras se itera

Esta es la parte del capítulo que separa a quien sabe escribir un bucle de quien sabe mantener código en producción. También es donde **las cinco fuentes de referencia fallan a la vez**: ninguna de las cinco menciona `ConcurrentModificationException`, y la única que trata el tema se equivoca de excepción.

## 29. ConcurrentModificationException

Borrar de una colección mientras se recorre con `for-each` lanza una excepción. Verificado en JDK 25:

```java
List<String> lista = new ArrayList<>(List.of("uno", "dos", "tres", "cuatro"));

for (String s : lista) {
    if (s.equals("dos")) {
        lista.remove(s);
    }
}
```

Salida real:

```
C1 CME al borrar 'dos'
```

`java.util.ConcurrentModificationException`, sin mensaje.

**El nombre engaña.** No hace falta concurrencia ni hilos: un solo hilo produce la excepción. Lo que significa "concurrent" aquí es que hubo **dos vías de acceso simultáneas a la misma colección**: el iterador, y la llamada directa a `remove`.

**Cómo se detecta.** Las colecciones de `java.util` llevan un contador interno, `modCount`, que se incrementa en cada cambio estructural (añadir, borrar, limpiar). Cuando se crea un iterador, este copia el valor actual en `expectedModCount`. En cada `next()` compara ambos:

```java
// Esquema de lo que hace ArrayList.Itr
final void checkForComodification() {
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

Si no coinciden, alguien modificó la colección por detrás del iterador y este ya no puede garantizar un recorrido correcto. Aborta.

Un **cambio estructural** es el que altera el tamaño. Cambiar el valor de un elemento existente **no** lo es:

```java
for (String s : lista) {
    lista.set(0, "otro");     // NO lanza: set no es estructural
}
```

Lo mismo ocurre con `forEach`. Verificado:

```java
List<String> d = new ArrayList<>(List.of("Larry", "Steve", "James"));
d.forEach(n -> { if (n.equals("Larry")) d.remove(n); });
```

```
C4 ArrayList mutable + forEach -> ConcurrentModificationException
```

## 30. Fail-fast no es una garantía

Este comportamiento se llama **fail-fast**: fallar cuanto antes en vez de producir resultados incorrectos en silencio. Es una buena política, y tiene un límite que casi nadie conoce y que está escrito en el Javadoc de `ArrayList`:

> *"the fail-fast behavior of iterators should be used only to detect bugs"*

La documentación es explícita en que **no se puede confiar en que la excepción salte**:

> *"Note that the fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification."*

Es decir: `ConcurrentModificationException` es una **ayuda de depuración**, no un mecanismo de corrección. No escribas código que dependa de que se lance, ni la captures para "manejar" la situación. Si la ves, hay un bug de diseño que hay que arreglar en el sitio donde se modifica la colección.

La siguiente sección muestra exactamente por qué esa advertencia está ahí.

## 31. El caso que no lanza la excepción

Aquí está el bug de verdad, y es peor que la excepción. Verificado en JDK 25 con la **misma** estructura del ejemplo anterior, cambiando solo qué elemento se borra:

```java
List<String> lista = new ArrayList<>(List.of("uno", "dos", "tres", "cuatro"));

for (String s : lista) {
    if (s.equals("tres")) {     // el PENÚLTIMO
        lista.remove(s);
    }
}
System.out.println(lista);
```

Salida real:

```
C2 SIN excepcion al borrar el penultimo: [uno, dos, cuatro]  <-- el bug silencioso
```

**Ninguna excepción.** El programa continúa como si nada.

Por qué: el `hasNext()` de `ArrayList` es sencillamente

```java
public boolean hasNext() {
    return cursor != size;
}
```

Al borrar el penúltimo elemento, `size` baja de 4 a 3 justo cuando el cursor vale 3. `hasNext()` devuelve `false`, el bucle termina **sin llegar a llamar a `next()`**, y `checkForComodification()` nunca se ejecuta. La comprobación se salta porque el bucle acabó antes.

La consecuencia práctica: **el último elemento nunca se visita**. En este ejemplo `"cuatro"` no llega a procesarse. Si el cuerpo del bucle hiciera algo más que borrar —enviar un correo, sumar un importe, escribir en base de datos— ese último elemento se pierde en silencio.

Esto es lo que hace peligroso al patrón, mucho más que la excepción:

- Borrar cualquier elemento **excepto el penúltimo** → excepción ruidosa, se detecta en el acto.
- Borrar **el penúltimo** → sin excepción, resultado incorrecto, y el bug sobrevive a los tests.

Y explica por qué el Javadoc insiste en que el fail-fast solo sirve para detectar bugs: hay un caso concreto en el que no detecta nada.

> **La conclusión operativa:** nunca modifiques una colección mientras la recorrés con `for-each`. No porque salte una excepción, sino porque a veces **no** salta.

## 32. Iterator.remove

La forma correcta de eliminar durante un recorrido es usar el iterador explícitamente y borrar **a través de él**:

```java
List<String> lista = new ArrayList<>(List.of("uno", "dos", "tres", "cuatro"));

Iterator<String> it = lista.iterator();
while (it.hasNext()) {
    if (it.next().startsWith("t")) {
        it.remove();
    }
}
```

Verificado:

```
D1 Iterator.remove: [uno, dos, cuatro]
```

Funciona porque `Iterator.remove()` actualiza **las dos** cosas a la vez: la colección y el `expectedModCount` del propio iterador. No hay desincronización, así que no hay excepción.

Tres reglas del contrato:

1. **`remove()` borra el elemento devuelto por la última llamada a `next()`.** Hay que llamar a `next()` antes.
2. **No se puede llamar dos veces seguidas** sin un `next()` en medio: lanza `IllegalStateException`.
3. **No todos los iteradores lo soportan.** El de una colección inmutable lanza `UnsupportedOperationException`.

Esa última regla conecta con un error concreto de una de las fuentes. Baeldung ilustra la "modificación durante `forEach`" con este código:

```java
List<String> names = List.of("Larry", "Steve", "James", "Conan", "Ellen");
names.forEach(name -> {
    if (name.equals("Larry")) {
        names.remove(name);
    }
});
```

y afirma que *"the code above throws a ConcurrentModificationException"*. Verificado en JDK 25:

```
C3 List.of + remove -> UnsupportedOperationException, NO CME
```

Lanza `UnsupportedOperationException`, y no por iterar: **`List.of` devuelve una lista inmutable**, así que `remove` falla siempre, dentro o fuera de un bucle. El ejemplo no demuestra lo que dice demostrar. Para ver la `ConcurrentModificationException` hay que usar una lista mutable, como en el caso `C4` de la sección 29.

## 33. removeIf

Desde Java 8, para el caso habitual —borrar los elementos que cumplen una condición— hay un método directo:

```java
List<String> lista = new ArrayList<>(List.of("uno", "dos", "tres", "cuatro"));
lista.removeIf(s -> s.startsWith("t"));
```

Verificado:

```
D2 removeIf: [uno, dos, cuatro]
```

Es la forma preferida cuando encaja, por tres razones:

1. **Expresa la intención**: "quitá los que cumplan esto", en vez de describir la mecánica del recorrido.
2. **Es correcta por construcción**: no hay iterador que desincronizar.
3. **Puede ser más eficiente**: `ArrayList` lo sobrescribe con una implementación que marca los elementos a borrar en una pasada y luego compacta el array **una sola vez**. Borrar `k` elementos con `Iterator.remove` mueve memoria `k` veces; `removeIf` lo hace una.

Métodos hermanos que resuelven casos vecinos sin bucle:

```java
lista.replaceAll(s -> s.toUpperCase());   // transformar todos
lista.removeAll(otraLista);               // quitar los de otra colección
lista.retainAll(otraLista);               // quedarse solo con los de otra
mapa.replaceAll((k, v) -> v * 2);         // transformar valores de un Map
mapa.entrySet().removeIf(e -> e.getValue() == null);   // filtrar un Map
```

## 34. Recorrer hacia atrás

Cuando hace falta el índice y además borrar, recorrer **hacia atrás** evita el problema de raíz:

```java
List<String> lista = new ArrayList<>(List.of("uno", "dos", "tres", "cuatro"));
for (int i = lista.size() - 1; i >= 0; i--) {
    if (lista.get(i).startsWith("t")) {
        lista.remove(i);
    }
}
```

Verificado:

```
D3 for indexado hacia atras: [uno, dos, cuatro]
```

Funciona porque borrar en la posición `i` solo desplaza los elementos **posteriores**, que ya fueron visitados. Los que quedan por visitar están antes y conservan su índice.

El mismo bucle hacia delante **se salta elementos**:

```java
// MAL: al borrar, el siguiente elemento se desplaza a la posición actual
for (int i = 0; i < lista.size(); i++) {
    if (lista.get(i).startsWith("t")) {
        lista.remove(i);     // ahora lista.get(i) es OTRO elemento, y i++ lo salta
    }
}
```

Este es el caso que hace peligrosa la afirmación de Baeldung de que *"the for loop... permits us to modify the collection itself"*. Permitirlo, lo permite; hacerlo bien requiere recorrer hacia atrás o ajustar el índice a mano tras cada borrado, y eso el artículo no lo dice.

## 35. Copiar antes de recorrer

Cuando el cuerpo del bucle puede modificar la colección de formas difíciles de predecir —por ejemplo, porque llama a código que no controlás—, la salida más simple es recorrer una copia:

```java
for (String s : new ArrayList<>(lista)) {
    if (condicion(s)) {
        lista.remove(s);      // se modifica la original, se recorre la copia
    }
}
```

Es correcto y tiene un coste: una copia superficial de la lista. Para colecciones pequeñas es despreciable; para colecciones grandes en un bucle caliente, no.

El caso donde esto es la respuesta correcta y no un parche son los **listeners**:

```java
// Un listener puede desregistrarse a sí mismo durante la notificación
for (Listener l : new ArrayList<>(listeners)) {
    l.alOcurrir(evento);
}
```

Sin la copia, cualquier listener que llame a `removeListener(this)` rompería el recorrido. Es un patrón tan común que muchas bibliotecas usan directamente una `CopyOnWriteArrayList` para la lista de listeners, que es de lo que trata la sección siguiente.

## 36. Colecciones concurrentes

Cuando de verdad hay varios hilos, la respuesta no es capturar `ConcurrentModificationException` ni sincronizar a mano el bucle entero: es usar una colección diseñada para ello.

```java
// Lecturas muy frecuentes, escrituras raras (listeners, configuración)
List<Listener> listeners = new CopyOnWriteArrayList<>();

// Mapa de propósito general con acceso concurrente
Map<String, Sesion> sesiones = new ConcurrentHashMap<>();

// Cola productor-consumidor
BlockingQueue<Tarea> cola = new LinkedBlockingQueue<>();
```

Estas colecciones tienen iteradores **débilmente consistentes** (*weakly consistent*) en vez de fail-fast: nunca lanzan `ConcurrentModificationException`, y su recorrido refleja el estado de la colección en algún momento desde que se creó el iterador. Pueden o no ver las modificaciones posteriores, pero no fallan.

`CopyOnWriteArrayList` lleva la idea al extremo: cada escritura copia el array interno completo, así que los iteradores existentes siguen viendo la instantánea con la que empezaron. Eso hace las lecturas muy baratas y las escrituras caras, y por eso su nicho son las listas que se leen mil veces por cada vez que se escriben.

La comparación resumida:

| Tipo de iterador | Colecciones | Ante modificación externa |
|---|---|---|
| Fail-fast | `ArrayList`, `HashMap`, `HashSet`, `LinkedList` | lanza `ConcurrentModificationException` (casi siempre) |
| Débilmente consistente | `ConcurrentHashMap`, `CopyOnWriteArrayList`, `ConcurrentLinkedQueue` | no lanza; puede ver o no los cambios |
| Inmutable | `List.of`, `Map.of`, `Collections.unmodifiableList` | la modificación lanza `UnsupportedOperationException` |

Esa tercera fila es la que explica el error de Baeldung de la sección 32: confundió la primera con la tercera.

---

# Parte VI — Rendimiento

## 37. El bucle cuadrático escondido

Este es el problema de rendimiento más caro de esta parte del lenguaje, porque **no se ve en el bucle**.

```java
for (int i = 0; i < lista.size(); i++) {
    procesar(lista.get(i));
}
```

Sobre un `ArrayList`, `get(i)` es un acceso directo a un array: O(1). El bucle completo es O(n) y todo está bien.

Sobre una `LinkedList`, `get(i)` tiene que **recorrer la lista desde un extremo** hasta la posición `i`: O(n). El bucle completo pasa a ser **O(n²)**.

Y lo grave es que el cambio que provoca el desastre está en otro archivo:

```java
List<String> datos = new ArrayList<>();    // el bucle es O(n)
List<String> datos = new LinkedList<>();   // el mismo bucle es O(n²)
```

Una sola palabra, en la línea de creación, y el recorrido cambia de complejidad. El bucle no se toca, la revisión de código no ve nada raro, y el sistema funciona perfectamente en desarrollo con 50 elementos. Con 100.000 elementos, un recorrido que debería tardar milisegundos tarda minutos.

**La solución es el `for-each`**, que usa el iterador y por tanto avanza de nodo en nodo:

```java
for (String s : lista) {
    procesar(s);
}
```

Este bucle es O(n) sobre **cualquier** implementación de `List`, porque `Iterator.next()` es O(1) en todas. El `for-each` no solo es más seguro que el `for` indexado: es **más robusto ante cambios de estructura de datos**.

La razón por la que el `for-each` gana aquí está en la firma del contrato. Un `for` indexado asume acceso aleatorio barato, una suposición que `List` no garantiza y que solo cumplen las implementaciones que declaran `RandomAccess`. La JDK expone esa distinción como interfaz marcadora:

```java
if (lista instanceof RandomAccess) {
    for (int i = 0; i < lista.size(); i++) { procesar(lista.get(i)); }
} else {
    for (String s : lista) { procesar(s); }
}
```

Ese código existe de verdad dentro de `java.util.Collections`. En código de aplicación casi nunca hace falta: usá `for-each` y ya.

## 38. Concatenar texto dentro de un bucle

```java
String resultado = "";
for (String s : partes) {
    resultado += s;          // MAL
}
```

Las `String` son inmutables, así que `resultado += s` no modifica nada: **construye una cadena nueva** con el contenido de las dos. En un bucle de n vueltas eso son n objetos intermedios, y el trabajo total es cuadrático respecto al tamaño final, porque cada paso copia todo lo acumulado hasta entonces.

La forma correcta:

```java
StringBuilder sb = new StringBuilder();
for (String s : partes) {
    sb.append(s);
}
String resultado = sb.toString();
```

Un solo `StringBuilder` para todo el bucle, que crece amortizado. El trabajo total es lineal.

**El matiz que hay que conocer**, porque genera confusión: fuera de un bucle, `"a" + b + "c"` **ya se compila** a una concatenación eficiente —desde Java 9, mediante `invokedynamic` y `StringConcatFactory`—. No hace falta usar `StringBuilder` a mano para concatenaciones sueltas; el compilador lo hace mejor.

El problema es exclusivo del bucle: la optimización se aplica **por expresión**, y en un bucle la expresión se evalúa entera en cada vuelta. El compilador no puede sacar el acumulador fuera porque `resultado` es visible desde fuera del bucle.

Y para el caso más común de todos, ni siquiera hace falta el bucle:

```java
String resultado = String.join(", ", partes);
String resultado = partes.stream().collect(Collectors.joining(", "));
```

## 39. Trabajo repetido dentro del bucle

Todo lo que no depende de la iteración debería estar fuera:

```java
// MAL: compila el patrón en cada vuelta
for (String linea : lineas) {
    if (linea.matches("^\\d{4}-\\d{2}-\\d{2}$")) { procesar(linea); }
}

// BIEN: se compila una vez
private static final Pattern FECHA = Pattern.compile("^\\d{4}-\\d{2}-\\d{2}$");

for (String linea : lineas) {
    if (FECHA.matcher(linea).matches()) { procesar(linea); }
}
```

`String.matches` compila la expresión regular en cada llamada. En un bucle sobre un fichero de un millón de líneas, eso es un millón de compilaciones de regex.

El mismo principio se aplica a cualquier cosa cara e invariante:

```java
// MAL
for (Pedido p : pedidos) {
    Configuracion c = servicio.cargarConfiguracion();   // en cada vuelta
    aplicar(p, c);
}

// BIEN
Configuracion c = servicio.cargarConfiguracion();
for (Pedido p : pedidos) {
    aplicar(p, c);
}
```

El caso más caro de todos es la consulta a base de datos dentro del bucle, conocido como **problema N+1**: una consulta para traer la lista y una más por cada elemento. Es la causa número uno de lentitud en aplicaciones con ORM, y la solución es traer los datos relacionados en la consulta inicial (`JOIN FETCH`, `IN (...)`, carga por lotes) en vez de pedirlos de uno en uno.

```java
// N+1 consultas
for (Pedido p : repo.buscarTodos()) {
    Cliente c = repoClientes.buscarPorId(p.getClienteId());   // una consulta por pedido
}
```

**Lo que NO hay que hacer:** sacar cosas del bucle por superstición. `for (int i = 0; i < lista.size(); i++)` no necesita cachear `size()` en una variable: es una lectura de campo que el JIT inlinea sin problema, y cachearla introduce un bug si la lista cambia. Optimizá lo que es caro de verdad —E/S, regex, consultas, asignación de objetos—, no las lecturas de campos.

## 40. Localidad de caché

Cuando el bucle recorre estructuras grandes, el orden de acceso importa tanto como el número de operaciones.

```java
// Por filas: rápido
for (int i = 0; i < m.length; i++) {
    for (int j = 0; j < m[i].length; j++) {
        suma += m[i][j];
    }
}

// Por columnas: el mismo número de accesos, bastante más lento
for (int j = 0; j < m[0].length; j++) {
    for (int i = 0; i < m.length; i++) {
        suma += m[i][j];
    }
}
```

Los dos bucles hacen exactamente el mismo trabajo lógico. La diferencia es que cuando el procesador lee una posición de memoria trae una **línea de caché** entera —unos 64 bytes, es decir 16 `int`—. Recorriendo por filas, los siguientes quince accesos ya están en caché. Recorriendo por columnas, cada acceso salta a otra fila, que en Java es **otro objeto distinto en el heap**, y desperdicia casi toda la línea traída.

La regla general: **el índice que varía más rápido debe ser el último**. Está tratado con más detalle en [Arrays](09-arrays.md), incluida la medición.

## 41. Por qué este capítulo no trae cifras

Habrás notado que la Parte VI argumenta con complejidad algorítmica y con bytecode, pero no da tiempos.

Es deliberado, y por la misma razón que en [Conditionals](10-conditionals.md): **los bucles son la construcción que el JIT reescribe con más agresividad**. Un microbenchmark casero sobre un bucle mide, en el mejor de los casos, el banco de pruebas; en el peor, mide nada en absoluto.

Lo que hace el compilador JIT con un bucle incluye desenrollado, vectorización, eliminación de comprobaciones de límites, movimiento de código invariante fuera del bucle, y —lo más traicionero— **eliminación completa del bucle** si detecta que su resultado no se usa. Un bucle que suma un millón de enteros y no hace nada con el resultado puede compilarse a cero instrucciones.

Ese fallo no es teórico: aparece siempre que alguien mide un bucle sin JMH. Para medir de verdad hacen falta calentamiento, consumo del resultado con un `Blackhole`, varias forks y análisis estadístico, que es exactamente lo que da [JMH](https://openjdk.org/projects/code-tools/jmh/).

Lo que sí es sólido y reproducible:

- **La complejidad algorítmica.** Que `LinkedList.get(i)` sea O(n) es una propiedad de la estructura de datos, no una medición. El bucle de la sección 37 es cuadrático en cualquier máquina y con cualquier JIT.
- **El bytecode.** `javap -c` da el mismo resultado siempre y muestra que un `for-each` sobre array no crea `Iterator` y sobre `List` sí.
- **El número de objetos creados.** Que concatenar con `+=` en un bucle produzca un objeto por vuelta es un hecho del modelo de ejecución.

Las decisiones de diseño que importan se toman con esos tres, no con un número de milisegundos.

---

# Parte VII — Streams frente a bucles

## 42. forEach no es un bucle

Desde Java 8, `Iterable` tiene un método `forEach`:

```java
void forEach(Consumer<? super T> action)
```

Que se usa así:

```java
List<String> nombres = List.of("Larry", "Steve", "James");
nombres.forEach(nombre -> System.out.println(nombre));
nombres.forEach(System.out::println);          // con referencia a método
```

Se parece tanto a un `for-each` que es fácil creer que son la misma cosa con otra sintaxis. **No lo son.** `forEach` es una llamada a un método normal que recibe una lambda; el `for-each` es una construcción del lenguaje. De ahí salen diferencias que no son de estilo.

**1. `break` y `continue` no existen.** Verificado en JDK 25:

```java
nombres.forEach(n -> {
    if (n.equals("Steve")) break;
    System.out.println(n);
});
```

```
F1.java:5: error: break outside switch or loop
            if (n.isEmpty()) break;
                             ^
```

Es un **error de compilación**, no una excepción en ejecución. El cuerpo de la lambda es un método aparte, y `break` no tiene ningún bucle al que referirse. Baeldung describe este mismo caso diciendo que *"the code above throws an exception"*, lo cual es doblemente inexacto: no lanza nada porque **no llega a compilar**.

**2. No se puede mutar una variable local.** Verificado:

```java
int contador = 0;
nombres.forEach(n -> { contador++; });
```

```
F2.java:5: error: local variables referenced from a lambda expression must be final or effectively final
        l.forEach(n -> { contador++; });
                         ^
```

El mensaje real dice **"final or effectively final"**, un matiz que conviene precisar porque Baeldung lo describe como que *"lambda expression requires variables used inside them to be final"*: no hace falta escribir `final`, basta con no reasignar la variable. Esa es justamente la diferencia entre `final` y *effectively final*.

Para contar dentro de una lambda hay tres salidas, en orden de calidad:

```java
// 1. La correcta: no contar a mano
long n = nombres.stream().filter(s -> s.length() > 3).count();

// 2. Si de verdad hace falta un acumulador mutable
AtomicInteger contador = new AtomicInteger();
nombres.forEach(n -> contador.incrementAndGet());

// 3. La que no hay que usar: un array de un elemento como truco
int[] contador = {0};
nombres.forEach(n -> contador[0]++);
```

La tercera funciona porque lo *effectively final* es la referencia al array, no su contenido. Es un rodeo alrededor de una regla que existe por una razón, y en un stream paralelo produce resultados incorrectos sin avisar.

**3. `return` dentro de la lambda no sale del método**, solo termina esa invocación de la lambda. Es el equivalente a un `continue`, no a un `break` ni a un `return`.

**4. No se puede lanzar una excepción comprobada** desde la lambda, porque `Consumer.accept` no las declara. Un `for-each` normal sí puede.

## 43. Iteración interna frente a externa

La diferencia conceptual entre las dos formas tiene nombre.

**Iteración externa** — el `for-each` y el `Iterator`. El programador controla el recorrido: pide el siguiente elemento, decide cuándo parar, cuándo saltar.

```java
for (String nombre : nombres) {
    System.out.println(nombre);
}
```

**Iteración interna** — `forEach` y los streams. El programador dice **qué** hacer con cada elemento; la colección decide **cómo** recorrerse.

```java
nombres.forEach(nombre -> System.out.println(nombre));
```

La ventaja de ceder el control es que la biblioteca puede hacer cosas que el programador no podría: recorrer en paralelo, reordenar operaciones, fusionar pasadas, evaluar de forma perezosa. La desventaja es exactamente la misma: **si cediste el control, no lo tenés**, y de ahí salen las cuatro limitaciones de la sección anterior.

Con streams eso se hace explícito:

```java
List<String> resultado = pedidos.stream()
        .filter(p -> !p.estaCancelado())
        .map(Pedido::getReferencia)
        .sorted()
        .toList();
```

Ese pipeline no recorre la lista cuatro veces. Los streams son **perezosos**: `filter`, `map` y `sorted` solo describen el trabajo, y nada ocurre hasta la operación terminal (`toList`). Entonces se hace una sola pasada. El bucle equivalente escrito a mano requeriría cuidado para no crear listas intermedias.

Sobre `parallelStream()`, una advertencia que la fuente que lo menciona no da:

```java
List<String> nombres = List.of("Larry", "Steve", "James", "Conan", "Ellen");
nombres.parallelStream().forEach(LOG::info);
```

Baeldung presenta este código diciendo que *"for large collections, using forEach() with a parallel stream can improve performance"*. Dos problemas: el ejemplo tiene **cinco elementos**, donde el coste de repartir el trabajo entre hilos supera con creces al del trabajo mismo; y `forEach` sobre un stream paralelo **no garantiza el orden** de procesamiento, así que ese log saldrá desordenado. Si el orden importa, el método es `forEachOrdered`. El paralelismo solo compensa con muchos elementos, trabajo por elemento apreciable y operaciones sin estado compartido.

## 44. Cómo se sale de un stream

Como `break` no existe, la pregunta "¿cómo corto un stream?" es de las más frecuentes. La respuesta es que **hay operaciones de cortocircuito** que hacen ese trabajo, y son mejores que un `break` porque se leen como lo que significan:

| Lo que querés | En un bucle | En un stream |
|---|---|---|
| Parar al encontrar el primero | `break` tras encontrar | `findFirst()` / `findAny()` |
| Saber si existe alguno | `break` con bandera | `anyMatch(...)` |
| Saber si todos cumplen | `break` con bandera | `allMatch(...)` |
| Saber si ninguno cumple | `break` con bandera | `noneMatch(...)` |
| Quedarte con los n primeros | contador y `break` | `limit(n)` |
| Parar cuando deje de cumplirse | `break` con condición | `takeWhile(...)` (Java 9+) |
| Saltar los primeros | `continue` con contador | `skip(n)` / `dropWhile(...)` |

Estas operaciones son de **cortocircuito**: no recorren el resto de la colección.

```java
// Con bucle
Usuario encontrado = null;
for (Usuario u : usuarios) {
    if (u.getEdad() > 18) { encontrado = u; break; }
}

// Con stream
Optional<Usuario> encontrado = usuarios.stream()
        .filter(u -> u.getEdad() > 18)
        .findFirst();
```

```java
// takeWhile: para en cuanto la condición falla
List<Integer> hastaElPrimeroGrande = numeros.stream()
        .takeWhile(n -> n < 100)
        .toList();
```

Ojo con la diferencia entre `takeWhile` y `filter`: `filter` recorre **todo** y se queda con los que cumplen; `takeWhile` **para** en el primero que no cumple. Sobre `[1, 2, 300, 4]`, `filter(n -> n < 100)` da `[1, 2, 4]` y `takeWhile(n -> n < 100)` da `[1, 2]`.

Y si de verdad necesitás salir de un `forEach` por una condición arbitraria, la respuesta correcta es **no usar `forEach`**: usá un bucle. Lanzar una excepción para simular un `break` es un anti-patrón que rompe el flujo, penaliza el rendimiento y confunde a quien lea el código.

## 45. Qué hace mejor cada uno

| | Bucle (`for-each`, `for`) | Stream |
|---|---|---|
| Salir antes de tiempo | `break`, natural | operaciones de cortocircuito |
| Modificar variables locales | sí | no (effectively final) |
| Excepciones comprobadas | sí | incómodo |
| Efectos colaterales (E/S, logs) | natural | desaconsejado |
| Transformar y filtrar en cadena | verboso | natural |
| Paralelismo | manual | `parallelStream()` |
| Depurar paso a paso | fácil | más difícil |
| Índice del elemento | `for` clásico | no directamente |
| Rendimiento en bucles calientes | predecible | normalmente igual, a veces peor |

**La guía práctica:**

- **Usá un bucle** cuando el cuerpo tenga efectos colaterales (escribir, loguear, enviar), cuando necesites salir con `break`/`return` por lógica compleja, cuando manejes excepciones comprobadas o cuando necesites el índice.
- **Usá un stream** cuando estés **transformando datos**: filtrar, mapear, agrupar, reducir. Ahí el pipeline se lee como la descripción del resultado y el bucle equivalente se lee como una receta.
- **No conviertas un bucle en stream solo por modernizarlo.** Un `forEach` con una lambda de diez líneas y efectos colaterales es un bucle peor escrito, no código funcional.

---

# Parte VIII — Diseño

## 46. Cuándo el bucle sobra

Igual que la mitad de las condicionales del capítulo anterior no deberían existir, buena parte de los bucles que se escriben a mano ya están escritos en la biblioteca estándar.

**Buscar un elemento:**

```java
// A mano
Usuario encontrado = null;
for (Usuario u : usuarios) {
    if (u.getId().equals(id)) { encontrado = u; break; }
}

// Con la biblioteca
Optional<Usuario> encontrado = usuarios.stream()
        .filter(u -> u.getId().equals(id))
        .findFirst();
```

**Comprobar si existe alguno:**

```java
// A mano
boolean hayAdmin = false;
for (Usuario u : usuarios) {
    if (u.esAdmin()) { hayAdmin = true; break; }
}

// Con la biblioteca
boolean hayAdmin = usuarios.stream().anyMatch(Usuario::esAdmin);
```

**Sumar o contar:**

```java
// A mano
int total = 0;
for (Pedido p : pedidos) { total += p.getUnidades(); }

// Con la biblioteca
int total = pedidos.stream().mapToInt(Pedido::getUnidades).sum();
```

**Transformar una colección en otra:**

```java
// A mano
List<String> refs = new ArrayList<>();
for (Pedido p : pedidos) { refs.add(p.getReferencia()); }

// Con la biblioteca
List<String> refs = pedidos.stream().map(Pedido::getReferencia).toList();
```

**Agrupar:**

```java
// A mano: ocho líneas y un mapa que hay que inicializar con cuidado
Map<Estado, List<Pedido>> porEstado = new HashMap<>();
for (Pedido p : pedidos) {
    porEstado.computeIfAbsent(p.getEstado(), k -> new ArrayList<>()).add(p);
}

// Con la biblioteca
Map<Estado, List<Pedido>> porEstado = pedidos.stream()
        .collect(Collectors.groupingBy(Pedido::getEstado));
```

**Rellenar o transformar un array:**

```java
// A mano
for (int i = 0; i < v.length; i++) { v[i] = i * i; }

// Con la biblioteca
Arrays.setAll(v, i -> i * i);
```

La señal para reconocerlo: **si el bucle tiene una variable acumuladora declarada justo antes y devuelta justo después, casi siempre existe una operación de biblioteca que hace lo mismo con un nombre**. Y ese nombre —`anyMatch`, `groupingBy`, `sum`— comunica la intención mucho mejor que la mecánica del recorrido.

Lo que **no** hay que hacer es forzarlo. Un bucle con efectos colaterales, o con lógica que no encaja en el vocabulario de streams, se queda como bucle.

## 47. Bucles legibles

Cuatro reglas que se aplican a cualquier bucle, sea de la forma que sea.

**1. El cuerpo debería caber en la pantalla.** Un bucle cuyo cuerpo tiene sesenta líneas es un método esperando a nacer:

```java
// Antes
for (Pedido p : pedidos) {
    // 60 líneas de validación, cálculo, persistencia y notificación
}

// Después
for (Pedido p : pedidos) {
    procesarPedido(p);
}
```

Además de legible, la segunda versión es testeable: se puede probar `procesarPedido` con un solo pedido.

**2. Un bucle, una responsabilidad.** Un bucle que valida, calcula el total, escribe en base de datos y manda correos hace cuatro cosas. Separarlo en cuatro pasadas es a veces más lento y casi siempre más claro; y si el rendimiento importa, se puede volver a fusionar **después** de medirlo.

**3. Nombres que digan qué contienen.** `for (Pedido p : pedidos)` está bien. `for (Pedido x : lista)` no dice nada. El plural en la colección y el singular en el elemento es una convención que se lee sola.

**4. Máximo dos niveles de anidamiento.** Tres niveles ya obligan a mantener demasiado contexto en la cabeza; a partir de ahí, extraé el bucle interno a un método. Es el mismo argumento de la complejidad ciclomática del capítulo anterior, y la solución es la misma.

---

# Parte IX — Cierre

## 48. Casos de uso reales

### 48.1 Procesar un fichero línea a línea

```java
try (BufferedReader lector = Files.newBufferedReader(ruta)) {
    String linea;
    int numeroLinea = 0;
    while ((linea = lector.readLine()) != null) {
        numeroLinea++;
        if (linea.isBlank() || linea.startsWith("#")) continue;
        procesar(linea, numeroLinea);
    }
}
```

Por qué así: `while` porque el número de líneas no se sabe de antemano; `continue` para descartar vacías y comentarios sin anidar; el contador de línea aparte porque los mensajes de error tienen que decir dónde falló; y `try-with-resources` para cerrar el fichero aunque se lance una excepción dentro del bucle.

### 48.2 Reintentar una operación con espera creciente

```java
public Respuesta llamarConReintentos(Peticion peticion) throws InterruptedException {
    int intento = 0;
    long esperaMs = 100;

    while (true) {
        try {
            return cliente.enviar(peticion);
        } catch (ErrorTransitorio e) {
            intento++;
            if (intento >= MAX_INTENTOS) {
                throw new ServicioNoDisponibleException(
                        "Fallaron " + MAX_INTENTOS + " intentos", e);
            }
            Thread.sleep(esperaMs);
            esperaMs *= 2;      // backoff exponencial
        }
    }
}
```

Por qué así: `while (true)` con salida por `return` en el éxito y por `throw` al agotar intentos; la espera se duplica para no castigar a un servicio que ya está sufriendo; y `MAX_INTENTOS` es una constante con nombre, no un número suelto.

### 48.3 Procesar en lotes

```java
List<Registro> lote = new ArrayList<>(TAMANO_LOTE);
for (Registro r : registros) {
    lote.add(r);
    if (lote.size() == TAMANO_LOTE) {
        repositorio.guardarTodos(lote);
        lote.clear();
    }
}
if (!lote.isEmpty()) {
    repositorio.guardarTodos(lote);   // el último lote incompleto
}
```

Por qué así: insertar de uno en uno son n viajes a la base de datos. Y fijate en el `if` final: **el resto que no llena un lote completo es el olvido más frecuente de este patrón**, y produce pérdida silenciosa de datos.

### 48.4 Filtrar una lista en sitio

```java
pedidos.removeIf(p -> p.getEstado() == Estado.CANCELADO);
```

Por qué así: es la única forma correcta, breve y sin iterador explícito de eliminar durante un recorrido. Cualquier variante con `for-each` y `remove` es el bug de la [sección 31](#31-el-caso-que-no-lanza-la-excepción).

### 48.5 Construir un índice para evitar el bucle anidado

```java
Map<String, Cliente> porId = clientes.stream()
        .collect(Collectors.toMap(Cliente::getId, Function.identity()));

for (Pedido p : pedidos) {
    Cliente c = porId.get(p.getClienteId());
    if (c == null) {
        log.warn("Pedido {} referencia un cliente inexistente: {}", p.getId(), p.getClienteId());
        continue;
    }
    p.setCliente(c);
}
```

Por qué así: convierte un cruce O(n·m) en O(n+m). Y el `continue` con log maneja explícitamente el caso de datos inconsistentes, en vez de dejar un `null` que reventará más adelante lejos de su causa.

## 49. Anti-patrones

**1. Punto y coma después de la cabecera.**

```java
for (int i = 0; i < 5; i++);   // MAL: cuerpo vacío
{ hacerAlgo(); }
```

**2. `<=` con `length` o `size`.**

```java
for (int i = 0; i <= v.length; i++)   // MAL: ArrayIndexOutOfBoundsException
for (int i = 0; i < v.length; i++)    // bien
```

**3. Modificar la variable de control dentro del cuerpo.**

```java
for (int i = 0; i < 10; i++) { i++; }   // MAL: avanza de dos en dos
```

**4. `continue` en un `while` antes del avance.**

```java
while (i < 10) { if (x) continue; i++; }   // MAL: bucle infinito
```

**5. Contador que puede desbordar.**

```java
for (int i = 0; i <= Integer.MAX_VALUE; i++)   // MAL: nunca termina
for (byte i = 0; i < 128; i++)                 // MAL: nunca termina
```

**6. `!=` como condición de parada.**

```java
for (int i = 0; i != 10; i += 3)   // MAL: se salta el 10
for (int i = 0; i < 10; i += 3)    // bien
```

**7. Contador decimal.**

```java
for (double x = 0.0; x != 1.0; x += 0.1)   // MAL: nunca termina
for (int i = 0; i < 10; i++) { double x = i / 10.0; }   // bien
```

**8. Modificar la colección dentro de un `for-each`.**

```java
for (String s : lista) { if (cond(s)) lista.remove(s); }   // MAL
lista.removeIf(s -> cond(s));                              // bien
```

**9. `get(i)` sobre una lista que podría ser `LinkedList`.**

```java
for (int i = 0; i < lista.size(); i++) { procesar(lista.get(i)); }   // MAL: O(n²)
for (String s : lista) { procesar(s); }                              // bien: O(n)
```

**10. Concatenar `String` con `+=` dentro del bucle.**

```java
for (String s : partes) { r += s; }              // MAL
for (String s : partes) { sb.append(s); }        // bien
String r = String.join("", partes);              // mejor
```

**11. Trabajo invariante dentro del bucle.**

```java
for (String l : lineas) { if (l.matches(REGEX)) { } }   // MAL: compila el regex n veces
```

**12. Consultas dentro del bucle (N+1).**

```java
for (Pedido p : pedidos) { repo.buscarCliente(p.getClienteId()); }   // MAL
```

**13. Recorrer `keySet()` y hacer `get()` dentro.**

```java
for (K k : mapa.keySet()) { V v = mapa.get(k); }        // MAL
for (Map.Entry<K,V> e : mapa.entrySet()) { }            // bien
```

**14. `break` dentro de un `switch` creyendo que sale del bucle.**

```java
for (...) { switch (x) { case A: break; } }   // MAL: sale del switch
```

**15. Lanzar una excepción para simular `break` en un `forEach`.** Si necesitás salir, usá un bucle.

**16. Bucle de espera activa.**

```java
while (cola.isEmpty()) { }        // MAL: quema un núcleo
Tarea t = cola.take();            // bien: bloquea
```

## 50. Checklist y tabla de decisión

**Antes de dar por bueno un bucle:**

- [ ] ¿Hay algún `;` justo después de la cabecera?
- [ ] ¿La condición usa `<` o `>` en vez de `!=`?
- [ ] Si hay índices, ¿usa `< length` y no `<= length`?
- [ ] ¿El contador puede desbordar su tipo?
- [ ] ¿El contador es un entero y no un `double`?
- [ ] ¿La variable de control se modifica solo en la cabecera?
- [ ] En bucles anidados, ¿el contador interno se reinicia?
- [ ] ¿El bucle termina siempre, para cualquier entrada posible?
- [ ] ¿Se modifica la colección mientras se recorre?
- [ ] Si hay que borrar, ¿usa `removeIf` o `Iterator.remove`?
- [ ] ¿Hay algún `get(i)` que podría ser O(n)?
- [ ] ¿Hay concatenación de `String` con `+=` dentro?
- [ ] ¿Hay trabajo invariante que debería estar fuera?
- [ ] ¿Hay consultas o llamadas remotas dentro del bucle?
- [ ] ¿El anidamiento pasa de dos niveles?
- [ ] ¿El cuerpo cabe en la pantalla?
- [ ] ¿Existe un método de biblioteca que haga esto por vos?

**Qué construcción usar:**

| Si necesitás… | Usá |
|---|---|
| Recorrer todos los elementos | `for-each` |
| Recorrer y necesitás el índice | `for` clásico |
| Recorrer hacia atrás o saltando | `for` clásico |
| Recorrer dos colecciones en paralelo | `for` clásico con índice |
| Repetir un número conocido de veces | `for` clásico |
| Repetir hasta que ocurra algo | `while` |
| Ejecutar al menos una vez y luego comprobar | `do-while` |
| Bucle de servicio que no debe terminar | `while (true)` con salida clara |
| Borrar elementos que cumplen algo | `removeIf` |
| Borrar con lógica compleja durante el recorrido | `Iterator` explícito con `remove()` |
| Transformar todos los elementos en sitio | `replaceAll` / `Arrays.setAll` |
| Transformar una colección en otra | `stream().map(...).toList()` |
| Buscar el primero que cumple | `stream().filter(...).findFirst()` |
| Saber si existe alguno / todos / ninguno | `anyMatch` / `allMatch` / `noneMatch` |
| Sumar, contar, máximo, mínimo | `mapToInt(...).sum()`, `count()`, `max()` |
| Agrupar por una clave | `Collectors.groupingBy(...)` |
| Unir textos con separador | `String.join` / `Collectors.joining` |
| Salir de varios bucles a la vez | extraer un método con `return`; si no, etiqueta |
| Recorrer un `Map` | `entrySet()` o `Map.forEach` |
| Recorrer con varios hilos | colección concurrente, o `parallelStream()` |

## 51. Autoevaluación

**1. ¿Cuál es la única diferencia entre `while` y `do-while`, y cuándo importa?**

<details><summary>Respuesta</summary>

El `while` comprueba la condición **antes** de cada vuelta y el `do-while` **después**, así que el `do-while` ejecuta el cuerpo siempre al menos una vez. Importa cuando la condición depende de algo que solo existe tras ejecutar el cuerpo: pedir datos al usuario y validarlos, mostrar un menú, leer una respuesta. Verificado en JDK 25: con `int i = 10;` y condición `i < 5`, el `while` no imprime nada y el `do-while` imprime una vez. Si con un `while` tenés que duplicar el cuerpo antes del bucle, lo que hacía falta era un `do-while`.
</details>

**2. ¿En qué orden se ejecutan las partes de un `for` y cuántas veces se evalúa la condición?**

<details><summary>Respuesta</summary>

Inicialización una sola vez; luego se repite el ciclo condición → cuerpo → avance. El avance ocurre **después** del cuerpo, no antes. La condición se evalúa **una vez más** que el número de vueltas: para cinco iteraciones se evalúa seis veces, porque hace falta una evaluación falsa para salir. Por eso, si la variable de control se declara fuera, su valor al terminar es el primero que **no** cumple la condición: tras `for (j = 0; j < 3; j++)`, `j` vale 3, no 2.
</details>

**3. ¿Por qué `while (false) {}` no compila pero `if (false) {}` sí?**

<details><summary>Respuesta</summary>

Por el análisis de alcanzabilidad del JLS. En un `while` con condición constante falsa, el cuerpo es inalcanzable y eso es un error: `error: unreachable statement`. El `if` está exceptuado **a propósito** para que funcione el idioma de la compilación condicional —`if (DEBUG) { ... }` con `DEBUG` constante—, donde eliminar el bloque es justo lo que se busca. Verificado en JDK 25: el `while` da error y el `if` compila sin un solo aviso.
</details>

**4. ¿Cuáles son las tres formas del off-by-one y cuál es la defensa real?**

<details><summary>Respuesta</summary>

Usar `<=` con `length` (lanza `ArrayIndexOutOfBoundsException` porque los índices van de 0 a length-1), usar `length - 1` (se salta el último en silencio) y empezar en 1 (se salta el primero). La defensa no es tener cuidado: es **no usar índices**. Un `for-each` no puede tener un off-by-one porque no hay índice que equivocar. Cuando el índice sea inevitable, las plantillas son `for (int i = 0; i < n; i++)` hacia delante y `for (int i = n - 1; i >= 0; i--)` hacia atrás.
</details>

**5. ¿Por qué `for (byte i = 0; i < 128; i++)` no termina nunca?**

<details><summary>Respuesta</summary>

Porque un `byte` va de -128 a 127 y **nunca puede valer 128**, así que la condición `i < 128` es siempre verdadera. Al pasar de 127, el incremento desborda a -128 y el ciclo vuelve a empezar. Lo mismo ocurre con `for (int i = 0; i <= Integer.MAX_VALUE; i++)`: para salir haría falta un `int` mayor que `MAX_VALUE`, que no existe. Ambos verificados en JDK 25. La defensa es usar `int` para contadores y `<` en vez de `<=` cuando el límite roza el máximo del tipo.
</details>

**6. Programiz llama "bucle infinito" a `for (int i = 1; i <= 10; --i)`. ¿Es cierto?**

<details><summary>Respuesta</summary>

No. **Termina tras 2.147.483.650 vueltas**, medido en JDK 25 en 536 ms. `i` baja hacia los negativos hasta `Integer.MIN_VALUE`; el siguiente decremento desborda a `Integer.MAX_VALUE` (2.147.483.647), que **no** cumple `i <= 10`, y el bucle sale. En la práctica el consejo de la fuente sigue siendo válido —un bucle que tarda medio segundo en no hacer nada es un bug— pero la etiqueta es técnicamente falsa. Con un contador `long` sí sería infinito a efectos prácticos.
</details>

**7. ¿Qué genera el compilador para un `for-each` sobre un array y para uno sobre una `List`?**

<details><summary>Respuesta</summary>

Cosas distintas. Sobre un **array** genera un bucle indexado con la longitud cacheada: el bytecode muestra `arraylength`, `iaload` y `goto`, **sin ninguna llamada a método**. Sobre un **`Iterable`** genera un bucle con iterador: `List.iterator()`, `Iterator.hasNext()` y `Iterator.next()`. Verificado con `javap -c` en JDK 25. Consecuencias: sobre arrays el `for-each` es gratis; sobre colecciones crea un objeto `Iterator` y hace una llamada a interfaz por elemento; y si la colección es de wrappers, además desempaqueta con `Integer.intValue()` en cada vuelta.
</details>

**8. ¿Por qué `for (int x : v) { x = x * 10; }` no modifica el array?**

<details><summary>Respuesta</summary>

Porque `x` es una variable local nueva a la que se **copia** el valor de cada elemento; asignarle algo cambia la copia, no la casilla. Verificado: el array queda `[1, 2, 3]`. Con objetos hay un matiz: lo que se copia es la **referencia**, así que **mutar** el objeto sí afecta (`s.append("!")` deja `[a!, b!]`) pero **reasignar** la variable no (`s = new StringBuilder("z")` no cambia nada). Para reemplazar elementos hace falta el índice, `replaceAll` o `Arrays.setAll`.
</details>

**9. ¿Qué es exactamente `ConcurrentModificationException` y por qué el nombre engaña?**

<details><summary>Respuesta</summary>

Es la excepción que lanzan los iteradores fail-fast de `java.util` cuando detectan que la colección cambió estructuralmente durante el recorrido. El nombre engaña porque **no hace falta concurrencia**: un solo hilo la produce. "Concurrent" se refiere a que hubo dos vías de acceso a la vez, el iterador y la llamada directa a `remove`. El mecanismo es un contador `modCount` en la colección que el iterador copia en `expectedModCount` y compara en cada `next()`. Solo los cambios **estructurales** (los que alteran el tamaño) lo incrementan; un `set` sobre un elemento existente no lo hace.
</details>

**10. ¿Por qué borrar el penúltimo elemento de una lista durante un `for-each` es peor que borrar cualquier otro?**

<details><summary>Respuesta</summary>

Porque **no lanza ninguna excepción** y deja un resultado incorrecto. El `hasNext()` de `ArrayList` es `cursor != size`; al borrar el penúltimo, `size` baja justo hasta igualar al cursor, el bucle termina sin llamar a `next()` y la comprobación de `modCount` nunca se ejecuta. Verificado en JDK 25: borrar `"tres"` de `[uno, dos, tres, cuatro]` deja `[uno, dos, cuatro]` **sin excepción**, y `"cuatro"` nunca llega a procesarse. Es exactamente por esto que el Javadoc advierte de que el fail-fast solo debe usarse para detectar bugs, nunca para garantizar corrección.
</details>

**11. ¿Cuáles son las formas correctas de eliminar elementos durante un recorrido?**

<details><summary>Respuesta</summary>

Cuatro. **`removeIf`** es la preferida cuando la condición cabe en un predicado: expresa la intención y `ArrayList` la implementa compactando el array una sola vez, en vez de una por elemento borrado. **`Iterator.remove()`** para lógica más compleja: funciona porque actualiza a la vez la colección y el `expectedModCount`. **Recorrer hacia atrás** con índice, porque borrar en `i` solo desplaza elementos ya visitados. Y **recorrer una copia** (`new ArrayList<>(lista)`) cuando el cuerpo puede modificar la colección de formas impredecibles, como al notificar listeners.
</details>

**12. Baeldung dice que este código lanza `ConcurrentModificationException`: `List.of("Larry",...).forEach(n -> { if (n.equals("Larry")) names.remove(n); })`. ¿Es cierto?**

<details><summary>Respuesta</summary>

No. Lanza **`UnsupportedOperationException`**, verificado en JDK 25. `List.of` devuelve una lista **inmutable**, así que `remove` falla siempre, dentro o fuera de un bucle, y nunca se llega a detectar ninguna modificación concurrente. El ejemplo no demuestra lo que dice demostrar. Para ver la `ConcurrentModificationException` de verdad hace falta una lista mutable: con `new ArrayList<>(List.of(...))` el mismo código sí la lanza.
</details>

**13. ¿Por qué un `for (int i = 0; i < lista.size(); i++)` puede ser O(n²)?**

<details><summary>Respuesta</summary>

Porque `List.get(i)` no garantiza tiempo constante. Sobre `ArrayList` es O(1) y el bucle es O(n); sobre `LinkedList` es O(n) porque hay que recorrer los nodos hasta la posición, y el bucle pasa a O(n²). Lo peligroso es que el cambio que lo provoca está en **otra línea y otro archivo**: cambiar `new ArrayList<>()` por `new LinkedList<>()`. La solución es el `for-each`, que usa el iterador y es O(n) sobre cualquier implementación. La JDK expone esta distinción con la interfaz marcadora `RandomAccess`.
</details>

**14. ¿Por qué `resultado += s` dentro de un bucle es un problema si fuera del bucle no lo es?**

<details><summary>Respuesta</summary>

Porque las `String` son inmutables: cada `+=` construye un objeto nuevo copiando todo lo acumulado, así que n vueltas producen n objetos y trabajo cuadrático respecto al tamaño final. Fuera de un bucle no es problema porque desde Java 9 el compilador traduce `"a" + b + "c"` a una concatenación eficiente con `invokedynamic` y `StringConcatFactory`. Esa optimización se aplica **por expresión**, y en un bucle la expresión se reevalúa entera cada vuelta; el compilador no puede sacar el acumulador fuera porque es visible desde fuera. La solución es un `StringBuilder`, o mejor `String.join` / `Collectors.joining`.
</details>

**15. ¿Por qué `break` dentro de un `forEach` no funciona, y qué error da exactamente?**

<details><summary>Respuesta</summary>

Porque el cuerpo de un `forEach` es una **lambda**, es decir un método aparte, y `break` no tiene ningún bucle al que referirse. Es un **error de compilación**, no una excepción en ejecución: `error: break outside switch or loop`, verificado en JDK 25. Por la misma razón tampoco se puede mutar una variable local (`error: local variables referenced from a lambda expression must be final or effectively final`) ni lanzar excepciones comprobadas. Para cortar un stream están las operaciones de cortocircuito: `findFirst`, `anyMatch`, `limit`, `takeWhile`.
</details>

**16. ¿Qué diferencia hay entre iteración interna y externa?**

<details><summary>Respuesta</summary>

En la **externa** —`for-each`, `Iterator`— el programador controla el recorrido: pide el siguiente elemento y decide cuándo parar o saltar. En la **interna** —`forEach`, streams— el programador dice qué hacer con cada elemento y la colección decide cómo recorrerse. Ceder el control permite a la biblioteca paralelizar, reordenar, fusionar pasadas y evaluar de forma perezosa; el precio es exactamente el mismo: sin control no hay `break`, ni mutación de locales, ni excepciones comprobadas. Un pipeline de stream con `filter`, `map` y `sorted` no recorre la colección tres veces: es perezoso y hace una sola pasada al llegar a la operación terminal.
</details>

**17. ¿Cuál es la diferencia entre `filter` y `takeWhile`?**

<details><summary>Respuesta</summary>

`filter` recorre **todos** los elementos y conserva los que cumplen el predicado. `takeWhile` (Java 9+) es de **cortocircuito**: para en el primero que no cumple y descarta el resto sin mirarlo. Sobre `[1, 2, 300, 4]` con el predicado `n < 100`, `filter` da `[1, 2, 4]` y `takeWhile` da `[1, 2]`. `takeWhile` es el equivalente de un `break` con condición; `filter` es el equivalente de un `continue`.
</details>

**18. ¿Por qué no hay que recorrer `keySet()` haciendo `get()` dentro?**

<details><summary>Respuesta</summary>

Porque duplica el trabajo: la entrada ya se había localizado al iterar, y el `get` vuelve a buscarla. Sobre `HashMap` cada `get` es O(1) y el coste es un factor constante; sobre `TreeMap` cada `get` es O(log n) y el recorrido pasa de O(n) a O(n log n). La forma correcta es `entrySet()`, que da clave y valor de una vez, o `Map.forEach`, que recibe un `BiConsumer` en vez de un `Consumer` precisamente para eso.
</details>

**19. ¿Por qué este capítulo no incluye mediciones de tiempo?**

<details><summary>Respuesta</summary>

Porque los bucles son la construcción que el JIT reescribe con más agresividad: desenrollado, vectorización, eliminación de comprobaciones de límites, movimiento de código invariante y —lo más traicionero— **eliminación completa del bucle** cuando su resultado no se usa. Un microbenchmark casero mide el banco de pruebas, no el lenguaje. Lo que sí es reproducible y sostiene las mismas conclusiones: la complejidad algorítmica (que `LinkedList.get` sea O(n) es una propiedad de la estructura, no una medida), el bytecode con `javap -c`, y el número de objetos creados. Para medir de verdad hace falta JMH.
</details>

**20. ¿Cuándo hay que borrar un bucle en vez de escribirlo mejor?**

<details><summary>Respuesta</summary>

La señal es que el bucle tenga una **variable acumuladora declarada justo antes y devuelta justo después**: casi siempre existe una operación de biblioteca con nombre que hace lo mismo. Buscar es `findFirst`, comprobar es `anyMatch`/`allMatch`/`noneMatch`, sumar es `mapToInt(...).sum()`, transformar es `map(...).toList()`, agrupar es `Collectors.groupingBy`, unir texto es `String.join`, rellenar un array es `Arrays.setAll`, filtrar en sitio es `removeIf`. El nombre comunica la intención mucho mejor que la mecánica del recorrido. Lo que no hay que hacer es forzarlo: un bucle con efectos colaterales o con lógica que no encaja en el vocabulario de streams se queda como bucle.
</details>

## 52. Fuentes

**Documentación oficial y especificación**

- [JLS §14.12 — The while Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.12), [§14.13 — The do Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.13) y [§14.14 — The for Statement](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.14) — la gramática y la semántica exactas, incluida la del `for-each` en §14.14.2 con la traducción literal que hace el compilador.
- [JLS §14.22 — Unreachable Statements](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.22) — donde está escrita la excepción del `if` que explica por qué `while (false)` no compila y `if (false)` sí.
- [JLS §4.2.2 — Integer Operations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-4.html#jls-4.2.2) — la aritmética que da la vuelta en silencio, base de los bucles infinitos por desbordamiento.
- [`java.lang.Iterable`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Iterable.html) y [`java.util.Iterator`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Iterator.html) — el contrato completo, incluido que `next()` debe lanzar `NoSuchElementException`.
- [`java.util.ArrayList`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ArrayList.html) — la advertencia literal de que *"the fail-fast behavior of an iterator cannot be guaranteed"* y de que solo debe usarse para detectar bugs.
- [`java.util.ConcurrentModificationException`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/ConcurrentModificationException.html) — donde se aclara que no implica concurrencia entre hilos.
- [`Collection.removeIf`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Collection.html#removeIf(java.util.function.Predicate)) y [`List.replaceAll`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/List.html#replaceAll(java.util.function.UnaryOperator)) — las alternativas correctas al bucle con borrado.
- [`java.util.RandomAccess`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/RandomAccess.html) — la interfaz marcadora que documenta explícitamente cuándo un bucle indexado es preferible a uno con iterador, y viceversa.
- [The for Statement — The Java Tutorials](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/for.html) — la introducción canónica, incluida la del `for-each`.
- [JMH — Java Microbenchmark Harness](https://openjdk.org/projects/code-tools/jmh/) — la única forma seria de medir un bucle, y la razón por la que este capítulo no da cifras.

**Las cinco fuentes de referencia de este capítulo, y dónde se equivocan**

- [Programiz — Java for Loop](https://www.programiz.com/java-programming/for-loop) y sus subpáginas enlazadas desde el sidebar y los botones de navegación: [for-each](https://www.programiz.com/java-programming/enhanced-for-loop), [while y do-while](https://www.programiz.com/java-programming/do-while-loop), [break](https://www.programiz.com/java-programming/break-statement), [continue](https://www.programiz.com/java-programming/continue-statement) y [bucles anidados](https://www.programiz.com/java-programming/nested-loop). Es la más didáctica de las cinco: sus tablas de iteración paso a paso, con el valor de la variable y el resultado de la condición en cada vuelta, son el mejor recurso que encontré para explicar el `for` a alguien que empieza. **Error 1:** presenta `for (int i = 1; i <= 10; --i)` bajo el epígrafe *"Infinite for Loop"*. **No es infinito:** verificado en JDK 25, termina tras 2.147.483.650 vueltas en 536 ms, porque al llegar a `Integer.MIN_VALUE` el decremento desborda a `Integer.MAX_VALUE` y la condición falla. El consejo práctico es correcto; la afirmación técnica no. **Problema 2:** su ejemplo de `continue` en bucles anidados declara el contador interno `j` **fuera** de los dos bucles, con lo que el bucle interno solo se ejecuta en la primera vuelta externa. Lo verifiqué y el comportamiento es el descrito en la [sección 21](#21-contadores-que-no-se-reinician). El artículo muestra la salida correcta pero **no señala que esa estructura es un bug clásico**, así que un lector que la copie arrastrará el patrón. **Hueco:** no menciona `ConcurrentModificationException` en ninguna de sus páginas de bucles.
- [W3Schools — Java For Loop](https://www.w3schools.com/java/java_for_loop.asp) y [Java While Loop](https://www.w3schools.com/java/java_while_loop.asp), con sus subpáginas de [do/while](https://www.w3schools.com/java/java_while_loop_do.asp), [bucles anidados](https://www.w3schools.com/java/java_nested_loops.asp), [for-each](https://www.w3schools.com/java/java_foreach_loop.asp), [break y continue](https://www.w3schools.com/java/java_break.asp) y los ejemplos de la vida real. Correcta en lo que cubre; no le encontré ninguna afirmación falsa, y el detalle de explicar de dónde viene el nombre `i` es un buen toque. **Imprecisión:** describe el `for-each` como *"used exclusively to loop through elements in an array (or other data structures)"*. La palabra *exclusively* invierte la relación real: la construcción está definida sobre `Iterable`, y los **arrays** son el caso especial que el compilador trata aparte, precisamente porque un array no implementa `Iterable`. El bytecode de la [sección 24](#24-qué-genera-el-compilador) lo demuestra: son dos traducciones completamente distintas. **Hueco 1:** su página de break y continue **no menciona las etiquetas**, así que un lector no sabrá que existe forma de salir de dos bucles a la vez. **Hueco 2:** no menciona `ConcurrentModificationException`, ni el desbordamiento del contador, ni el problema del `continue` antes del avance en un `while`, que son los tres errores que de verdad llegan a producción.
- [Baeldung — A Guide to Java Loops](https://www.baeldung.com/java-loops). Pese al título, **no es una guía**: son seis secciones de las cuales tres consisten en una frase y un enlace a otro artículo ("For a detailed example, have a look at the dedicated post"). Enumera los cuatro tipos de bucle y **no explica el for-each pese a listarlo**, no menciona `break`, `continue`, etiquetas, `ConcurrentModificationException`, ni ninguno de los errores de contador. Como índice de otros artículos cumple; como recurso de aprendizaje sobre bucles no aporta nada que no esté en las dos fuentes anteriores.
- [Baeldung — Guide to the Java forEach Loop](https://www.baeldung.com/foreach-java). Este sí es sustancial, y es la única de las cinco fuentes que trata la distinción entre iteración interna y externa, que es un concepto valioso y bien explicado. Tiene **tres errores comprobables**. **Error 1, el más grave:** en la sección "Cannot Modify the Collection Itself" ilustra el problema con una lista creada con `List.of(...)` y afirma que *"the code above throws a ConcurrentModificationException"*. Verificado en JDK 25: lanza **`UnsupportedOperationException`**, porque `List.of` es inmutable y `remove` falla siempre, con o sin bucle. El ejemplo no demuestra lo que dice demostrar; para provocar la `ConcurrentModificationException` hay que usar `new ArrayList<>(List.of(...))`, y entonces sí se reproduce. **Error 2:** sobre `break` dentro de un `forEach` dice *"the code above throws an exception"*. No lanza nada: **no compila**. El mensaje real es `error: break outside switch or loop`, porque el cuerpo de la lambda es un método aparte sin ningún bucle al que referirse. Confundir un error de compilación con una excepción de ejecución no es un matiz: cambia por completo cuándo y cómo se detecta el problema. **Error 3, menor pero conceptual:** dice que *"lambda expression requires variables used inside them to be final, meaning their value can't be modified after initialization"*. El requisito real es **final o *effectively final***, como dice el propio mensaje del compilador: no hace falta escribir `final`, basta con no reasignar. **Imprecisión 4:** su ejemplo de `parallelStream()` para "mejorar el rendimiento con colecciones grandes" usa una lista de **cinco elementos**, donde el reparto entre hilos cuesta más que el trabajo, y no advierte de que `forEach` sobre un stream paralelo **no conserva el orden** —para eso está `forEachOrdered`—. **Imprecisión 5:** afirma que el `for` clásico *"permits us to modify the collection itself"*, sin advertir de que un `for` indexado hacia delante que borra elementos **se salta el siguiente en cada borrado**. Permitirlo lo permite; hacerlo bien exige recorrer hacia atrás.

**Discusiones y artículos de la comunidad** (consultados para contrastar los casos límite)

- [Iterating through a Collection, avoiding ConcurrentModificationException when removing objects in a loop](https://stackoverflow.com/questions/223918/iterating-through-a-collection-avoiding-concurrentmodificationexception-when-re) — el hilo canónico sobre el tema, con las cuatro soluciones y sus compromisos.
- [Why is ConcurrentModificationException not thrown when removing the second-to-last element?](https://stackoverflow.com/questions/9843665/java-concurrentmodificationexception-when-removing-second-to-last-element) — la explicación del `cursor != size` que produce el bug silencioso de la [sección 31](#31-el-caso-que-no-lanza-la-excepción).
- [How to break from Java stream forEach](https://www.baeldung.com/java-break-stream-foreach) — el recorrido por las alternativas de cortocircuito, y por qué lanzar una excepción para simular `break` es mala idea.
- [The Difference Between Collection.stream().forEach() and Collection.forEach()](https://www.baeldung.com/java-collection-stream-foreach) — el matiz sobre el orden y la modificación concurrente entre ambas formas.
- [Performance of traditional for loop vs Iterator/foreach in Java](https://stackoverflow.com/questions/2113216/which-is-more-efficient-a-for-each-loop-or-an-iterator) — por qué la respuesta correcta depende de `RandomAccess` y no de la sintaxis.
- [Java Language Specification: definite assignment and loops](https://docs.oracle.com/javase/specs/jls/se25/html/jls-16.html) — el análisis que permite al compilador saber que un método terminado en `for(;;)` no necesita `return`.

**Nota sobre la verificación.** Todos los outputs, mensajes de excepción y errores de compilación de este documento se obtuvieron ejecutando el código en Temurin JDK 25.0.3: los programas con `java Archivo.java`, los volcados de bytecode con `javap -c`, y los errores compilando a propósito ficheros que fallan con `javac`. El recuento de 2.147.483.650 vueltas de la [sección 20](#20-el-bucle-infinito-que-no-lo-es) se obtuvo con un contador `long` dentro del propio bucle y `System.nanoTime()` alrededor; es un recuento exacto de iteraciones, no una medida de rendimiento. **En este capítulo no hay comparativas de tiempo entre construcciones a propósito**, por las razones de la [sección 41](#41-por-qué-este-capítulo-no-trae-cifras): los bucles son lo que el JIT más reescribe, y un microbenchmark sin JMH describe el banco de pruebas y no el lenguaje. Los argumentos de rendimiento del documento se apoyan en complejidad algorítmica, bytecode y número de objetos creados, que son deterministas y reproducibles.
