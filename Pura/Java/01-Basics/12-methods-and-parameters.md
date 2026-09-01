# Methods and Parameters

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25 (Temurin 25.0.3)

**Alcance de este documento.** Los dos capítulos anteriores, [Conditionals](10-conditionals.md) y [Loops](11-loops.md), completaron el control de flujo dentro de un bloque de código. Este capítulo cierra el bloque `01-Basics` con la construcción que permite **salir de ese bloque y volver**: el método. Es la primera herramienta de abstracción del lenguaje, y la última pieza que falta antes de entrar en objetos.

Un **método** es un bloque de código con nombre, que recibe valores, hace algo y opcionalmente devuelve un resultado. Esa frase cabe en un tuit y se aprende en diez minutos. Lo que no cabe es todo lo que Java decide **por vos** cada vez que escribís un paréntesis: qué método concreto se llama cuando hay varios con el mismo nombre, qué se copia y qué se comparte al pasar un argumento, en qué momento se toma esa decisión, y qué queda grabado en el `.class` para siempre.

Los huecos aquí tienen un rasgo propio: casi ninguno se manifiesta como una excepción. Un bucle mal escrito se cuelga y se nota; un método mal diseñado **compila, ejecuta y devuelve un resultado incorrecto**, o funciona durante dos años hasta que alguien recompila una biblioteca.

La lista de bugs concretos que salen de este capítulo, todos reales y todos verificados aquí:

- Un método `intercambiar(a, b)` que parece intercambiar dos objetos y no intercambia nada.
- Un parámetro reasignado dentro del método, que el autor cree que se ve desde fuera.
- Un `getter` que devuelve la lista interna del objeto y permite que cualquiera la modifique desde fuera.
- Un `return` dentro de un `finally` que **se traga una excepción** en curso y devuelve un valor como si nada hubiera pasado.
- Dos métodos que solo se diferencian en el tipo de retorno, y un error de compilación que una fuente muy leída explica al revés.
- Un `lista.remove(1)` que borra la **posición** 1 en vez del **valor** 1.
- Un `Arrays.asList(arrayDeInt)` que devuelve una lista de **un** elemento en vez de tres.
- Un `f(1)` que llama a `f(long)` aunque exista `f(Integer)`, y otro que deja de compilar el día que alguien añade una sobrecarga.
- Un cliente compilado que sigue llamando al método viejo después de actualizar la biblioteca, **sin recompilar y sin avisar**.
- Un `factorial(13)` que devuelve 1.932.053.504 en un ejemplo de tutorial que nadie marca como roto.
- Un `StackOverflowError` cuya profundidad no es una constante del lenguaje, sino que cambia según el método, la opción `-Xss` y la ejecución.
- Un método con 256 parámetros que no compila, en contra de lo que afirma explícitamente una de las fuentes.

Vamos a cubrir el modelo completo: la anatomía de una declaración, qué hace realmente la JVM al invocar, por qué Java pasa siempre por valor y qué significa eso con objetos, las reglas del retorno, el algoritmo de tres fases con el que el compilador elige entre métodos sobrecargados, los varargs y sus trampas, la recursión y sus límites físicos, los dos techos duros de la plataforma, y —lo que separa un método que funciona de uno que se puede mantener— cómo se diseña una firma.

**Sobre la verificación.** Todas las salidas, mensajes de excepción, errores del compilador y volcados de bytecode de este documento se ejecutaron realmente en un JDK 25. Las cinco fuentes de referencia usadas para prepararlo contienen entre todas **cuatro afirmaciones falsas comprobables**, incluida una que se contradice con el propio artículo en el que aparece y otra que niega la existencia de un límite que el compilador aplica. Está todo señalado en la [sección 61](#61-fuentes) con el resultado real al lado.

---

## Índice

**Parte I — Anatomía de un método**

1. [Qué es un método y qué problema resuelve](#1-qué-es-un-método-y-qué-problema-resuelve)
2. [Las partes de una declaración](#2-las-partes-de-una-declaración)
3. [Parámetros y argumentos](#3-parámetros-y-argumentos)
4. [El tipo de retorno y la sentencia return](#4-el-tipo-de-retorno-y-la-sentencia-return)
5. [void y los métodos que no devuelven nada](#5-void-y-los-métodos-que-no-devuelven-nada)
6. [Métodos estáticos y métodos de instancia](#6-métodos-estáticos-y-métodos-de-instancia)
7. [Qué hace la JVM cuando llamás a un método](#7-qué-hace-la-jvm-cuando-llamás-a-un-método)
8. [Lo que el método ve desde dentro](#8-lo-que-el-método-ve-desde-dentro)

**Parte II — Los parámetros son copias**

9. [Java pasa siempre por valor](#9-java-pasa-siempre-por-valor)
10. [Qué cambia cuando el argumento es un objeto](#10-qué-cambia-cuando-el-argumento-es-un-objeto)
11. [Por qué un método de intercambio no funciona](#11-por-qué-un-método-de-intercambio-no-funciona)
12. [Arrays, colecciones y StringBuilder](#12-arrays-colecciones-y-stringbuilder)
13. [El error que cometen casi todas las fuentes](#13-el-error-que-cometen-casi-todas-las-fuentes)
14. [Reasignar un parámetro, final y effectively final](#14-reasignar-un-parámetro-final-y-effectively-final)
15. [El orden de evaluación de los argumentos](#15-el-orden-de-evaluación-de-los-argumentos)
16. [Los nombres de los parámetros no viajan al class](#16-los-nombres-de-los-parámetros-no-viajan-al-class)

**Parte III — El retorno**

17. [missing return statement](#17-missing-return-statement)
18. [Código inalcanzable después de un return](#18-código-inalcanzable-después-de-un-return)
19. [return dentro de un finally](#19-return-dentro-de-un-finally)
20. [Devolver null](#20-devolver-null)
21. [Ignorar el valor devuelto](#21-ignorar-el-valor-devuelto)
22. [Devolver estructuras mutables](#22-devolver-estructuras-mutables)

**Parte IV — Sobrecarga**

23. [Qué es la sobrecarga y qué la define](#23-qué-es-la-sobrecarga-y-qué-la-define)
24. [La firma no incluye el tipo de retorno](#24-la-firma-no-incluye-el-tipo-de-retorno)
25. [El error real cuando dos métodos chocan](#25-el-error-real-cuando-dos-métodos-chocan)
26. [Sobrecargar cambiando solo el tipo](#26-sobrecargar-cambiando-solo-el-tipo)
27. [Las tres fases de la resolución](#27-las-tres-fases-de-la-resolución)
28. [Fase 1: ensanchamiento](#28-fase-1-ensanchamiento)
29. [Fase 2: boxing](#29-fase-2-boxing)
30. [Fase 3: varargs](#30-fase-3-varargs)
31. [El más específico gana, y el caso de null](#31-el-más-específico-gana-y-el-caso-de-null)
32. [Cuando ninguno es más específico](#32-cuando-ninguno-es-más-específico)
33. [Ensanchamiento y boxing no se combinan](#33-ensanchamiento-y-boxing-no-se-combinan)
34. [La resolución ocurre en compilación, y queda grabada](#34-la-resolución-ocurre-en-compilación-y-queda-grabada)
35. [Las trampas de la biblioteca estándar](#35-las-trampas-de-la-biblioteca-estándar)
36. [Cuándo no sobrecargar](#36-cuándo-no-sobrecargar)

**Parte V — Varargs**

37. [Qué es un varargs y en qué se traduce](#37-qué-es-un-varargs-y-en-qué-se-traduce)
38. [Las reglas de los varargs](#38-las-reglas-de-los-varargs)
39. [Nunca es null, salvo que insistas](#39-nunca-es-null-salvo-que-insistas)
40. [Varargs y arrays de primitivos](#40-varargs-y-arrays-de-primitivos)
41. [Varargs y sobrecarga](#41-varargs-y-sobrecarga)
42. [Contaminación del heap y SafeVarargs](#42-contaminación-del-heap-y-safevarargs)

**Parte VI — Recursión**

43. [Qué es la recursión y cuándo compensa](#43-qué-es-la-recursión-y-cuándo-compensa)
44. [La pila y el StackOverflowError](#44-la-pila-y-el-stackoverflowerror)
45. [Java no elimina las llamadas de cola](#45-java-no-elimina-las-llamadas-de-cola)
46. [El coste exponencial y la memoización](#46-el-coste-exponencial-y-la-memoización)
47. [El desbordamiento silencioso del factorial](#47-el-desbordamiento-silencioso-del-factorial)

**Parte VII — Los límites de la plataforma**

48. [Doscientos cincuenta y cinco parámetros](#48-doscientos-cincuenta-y-cinco-parámetros)
49. [Sesenta y cuatro kilobytes por método](#49-sesenta-y-cuatro-kilobytes-por-método)
50. [Métodos pequeños y el JIT](#50-métodos-pequeños-y-el-jit)

**Parte VIII — Diseño de la firma**

51. [Cuántos parámetros son demasiados](#51-cuántos-parámetros-son-demasiados)
52. [Parámetros booleanos y el orden](#52-parámetros-booleanos-y-el-orden)
53. [Nombres](#53-nombres)
54. [Validar en la frontera](#54-validar-en-la-frontera)
55. [Métodos puros y efectos](#55-métodos-puros-y-efectos)
56. [Un método, un nivel de abstracción](#56-un-método-un-nivel-de-abstracción)

**Parte IX — Cierre**

57. [Casos de uso reales](#57-casos-de-uso-reales)
58. [Anti-patrones](#58-anti-patrones)
59. [Checklist y tabla de decisión](#59-checklist-y-tabla-de-decisión)
60. [Autoevaluación](#60-autoevaluación)
61. [Fuentes](#61-fuentes)

---

# Parte I — Anatomía de un método

## 1. Qué es un método y qué problema resuelve

Supongamos que hay que calcular el precio con IVA de tres productos. Sin métodos:

```java
double precio1 = 100.0;
double total1 = precio1 + precio1 * 0.21;
System.out.println("Total: " + total1);

double precio2 = 250.0;
double total2 = precio2 + precio2 * 0.21;
System.out.println("Total: " + total2);

double precio3 = 39.90;
double total3 = precio3 + precio3 * 0.21;
System.out.println("Total: " + total3);
```

Funciona, y tiene los mismos tres problemas que tenía repetir un `println` cinco veces en el capítulo de bucles, más uno nuevo y peor. Los tres conocidos: cambiar el IVA obliga a editar tres líneas, hacerlo para mil productos es inviable, y el número de repeticiones tiene que conocerse al escribir el programa.

El nuevo es que **el `0.21` está escrito tres veces**. El día que el tipo pase al 22 %, cualquiera de las tres puede quedarse sin actualizar, y el programa seguirá compilando y ejecutándose con dos precios correctos y uno incorrecto. Un bucle no arregla eso: el bucle repite, pero no da un nombre a la operación.

Un método sí:

```java
static double conIva(double precio) {
    return precio + precio * 0.21;
}
```

Y ahora:

```java
System.out.println("Total: " + conIva(100.0));
System.out.println("Total: " + conIva(250.0));
System.out.println("Total: " + conIva(39.90));
```

Las tres ganancias son distintas entre sí, y conviene no confundirlas:

1. **Reutilización.** La fórmula existe una sola vez. Cambiarla es cambiar una línea.
2. **Nombre.** `conIva(100.0)` dice qué se está calculando. `100.0 + 100.0 * 0.21` dice cómo. El lector de un programa quiere el qué; el cómo solo le interesa si el qué falla.
3. **Aislamiento.** Lo que ocurre dentro del método no puede tocar las variables de fuera, salvo que se lo permitas explícitamente. Eso reduce la cantidad de cosas que hay que tener en la cabeza a la vez, que es el recurso escaso de verdad al programar.

La tercera es la que se subestima. Un programa de mil líneas escritas de corrido obliga a razonar sobre mil líneas simultáneamente. El mismo programa partido en cuarenta métodos de veinticinco líneas obliga a razonar sobre veinticinco cada vez, más los nombres de los otros treinta y nueve.

Java no tiene funciones sueltas: **todo método vive dentro de una clase o una interfaz**. Es una diferencia real con Python, JavaScript o C, y explica por qué hasta el programa más simple necesita una clase alrededor.

## 2. Las partes de una declaración

La declaración completa de un método tiene hasta seis partes, y solo tres son obligatorias:

```java
public static int contarPalabras(String texto) throws IOException {
    return texto.split("\\s+").length;
}
```

| Parte | En el ejemplo | ¿Obligatoria? |
|---|---|---|
| Modificadores de acceso y otros | `public static` | No |
| Tipo de retorno | `int` | **Sí** |
| Nombre | `contarPalabras` | **Sí** |
| Lista de parámetros entre paréntesis | `(String texto)` | **Sí**, aunque esté vacía |
| Cláusula de excepciones | `throws IOException` | No |
| Cuerpo entre llaves | `{ ... }` | Sí, salvo en métodos `abstract` o `native` |

El mínimo absoluto es entonces:

```java
void nada() { }
```

Eso es un método válido y completo: acceso por defecto (de paquete), no devuelve nada, no recibe nada, no lanza nada declarado y no hace nada.

Dos detalles que suelen sorprender:

**El orden de los modificadores es libre para el compilador, pero no para las convenciones.** `static public int f()` compila igual que `public static int f()`. La convención del JLS y de toda la biblioteca estándar es acceso primero, luego `static`, luego `final`. Escribirlo al revés es legal y nadie lo hace.

**Los paréntesis vacíos no son opcionales.** En Java, `void nada` sin paréntesis no declara un método: declara un campo mal escrito y da error. El paréntesis es lo que distingue un método de una variable, tanto al declarar como al llamar.

## 3. Parámetros y argumentos

Las dos palabras se usan como sinónimas en la conversación y no lo son:

- **Parámetro** es la variable declarada en la firma. Existe en el código fuente del método.
- **Argumento** es el valor concreto que se pasa al llamar. Existe en el sitio de la llamada.

```java
static void saludar(String nombre) {   // 'nombre' es un PARÁMETRO
    System.out.println("Hola, " + nombre);
}

saludar("Ana");                        // "Ana" es un ARGUMENTO
```

La distinción importa porque los mensajes del compilador la respetan al pie de la letra, y porque toda la Parte II de este capítulo trata de una sola idea: **el argumento se copia dentro del parámetro**.

Un método puede declarar cero, uno o muchos parámetros, separados por comas, cada uno con su tipo. Al llamar hay que respetar tres cosas: la cantidad, el orden y la compatibilidad de tipos.

```java
static void registrar(String usuario, int edad, boolean activo) { }

registrar("ana", 30, true);      // bien
registrar(30, "ana", true);      // error: los tipos no encajan en ese orden
registrar("ana", 30);            // error: falta un argumento
```

El orden es la parte peligrosa. Si dos parámetros consecutivos son del mismo tipo, invertirlos **compila sin un solo aviso**:

```java
static void transferir(long cuentaOrigen, long cuentaDestino, double importe) { }

transferir(destino, origen, importe);   // compila, y el dinero va al revés
```

Ningún compilador puede detectar eso. La defensa es de diseño, no de sintaxis, y se trata en la [sección 52](#52-parámetros-booleanos-y-el-orden).

**Los parámetros pueden tener cualquier tipo**: primitivos, clases, interfaces, arrays, enums, genéricos. Jenkov dice que se puede usar «cualquier tipo primitivo o cualquier clase de Java, y también tus propias clases»; es correcto pero se queda corto, porque omite los arrays, que son el caso que da lugar a la mitad de las confusiones de la Parte II.

## 4. El tipo de retorno y la sentencia return

El tipo de retorno se declara antes del nombre, y la sentencia `return` produce el valor:

```java
static int cuadrado(int n) {
    return n * n;
}
```

`return` hace dos cosas a la vez, y conviene verlas por separado:

1. **Determina el valor** que la llamada produce.
2. **Termina el método inmediatamente.** Nada de lo que venga después se ejecuta.

De la segunda se deriva que puede haber varios `return` en un método, pero solo uno se ejecuta por invocación:

```java
static String clasificar(int nota) {
    if (nota >= 90) return "sobresaliente";
    if (nota >= 70) return "notable";
    if (nota >= 50) return "aprobado";
    return "suspenso";
}
```

Este estilo —varios `return` tempranos— es el mismo *early return* que el capítulo de [Conditionals](10-conditionals.md) recomienda frente al anidamiento. Existe una escuela contraria, heredada de Pascal y del C estructurado, que exige **un único punto de salida**; en Java, con `try-with-resources` y `finally` para la liberación de recursos, esa regla ya no compra nada y sí cuesta un nivel de anidamiento por cada condición.

El valor devuelto debe ser **asignable** al tipo declarado. No basta con que "quepa":

```java
static int f() { return 3.0; }    // error: incompatible types: possible lossy conversion from double to int
static long g() { return 3; }     // bien: int se ensancha a long
static double h() { return 3; }   // bien: int se ensancha a double
```

Las reglas son exactamente las de la conversión de asignación del capítulo [Type Casting](05-type-casting.md): ensanchar es implícito, estrechar exige un cast explícito.

## 5. void y los métodos que no devuelven nada

`void` no es un tipo: es la marca de que **no hay valor de retorno**.

```java
static void registrar(String mensaje) {
    System.out.println("[LOG] " + mensaje);
}
```

En un método `void`, `return` sigue siendo legal, pero sin valor, y solo sirve para salir antes de tiempo:

```java
static void procesar(String texto) {
    if (texto == null || texto.isBlank()) {
        return;                       // salida temprana
    }
    System.out.println(texto.strip());
}
```

Tres cosas que no se pueden hacer con `void`:

```java
void f() { return 5; }              // error: incompatible types: unexpected return value
int x = metodoVoid();               // error: 'void' type not allowed here
System.out.println(metodoVoid());   // error: el mismo
```

Y una distinción de diseño que vale más que la sintaxis: **un método `void` solo puede ser útil por sus efectos**. Si no devuelve nada y no cambia nada observable, no hace nada. Eso significa que todo `void` es, por definición, un método con efectos colaterales, y que la pregunta "¿qué cambia este método?" siempre tiene respuesta para un `void` y a menudo no la tiene para uno que devuelve un valor. Se retoma en la [sección 55](#55-métodos-puros-y-efectos).

`Void` con mayúscula, que aparece en `Future<Void>` o `Callable<Void>`, es otra cosa: una clase que existe solo para rellenar un parámetro genérico cuando no hay resultado. Se ve en el bloque de concurrencia.

## 6. Métodos estáticos y métodos de instancia

```java
public class Calculadora {
    private int acumulado = 0;

    static int sumar(int a, int b) {       // estático: pertenece a la clase
        return a + b;
    }

    int acumular(int n) {                  // de instancia: pertenece al objeto
        acumulado += n;
        return acumulado;
    }
}
```

La diferencia operativa es cómo se llaman:

```java
int s = Calculadora.sumar(2, 3);          // por el nombre de la clase

Calculadora c = new Calculadora();
int a = c.acumular(5);                    // hace falta un objeto
```

La diferencia conceptual es qué datos ve cada uno. Un método de instancia recibe un parámetro invisible, `this`, que apunta al objeto sobre el que se llamó; por eso puede leer y escribir `acumulado`. Un método estático no tiene `this`, y de ahí sale el error más frecuente de los primeros días:

```java
public class Main {
    int contador = 0;

    public static void main(String[] args) {
        contador++;    // error: non-static variable contador cannot be referenced from a static context
    }
}
```

El mensaje es literal: no es que `contador` no exista, es que **no hay ningún objeto sobre el que leerlo**. `main` es estático porque la JVM tiene que poder llamarlo sin haber creado nada todavía.

Hay una consecuencia curiosa y verificada: como un método estático no usa la instancia, **llamarlo a través de una referencia nula funciona**.

```java
Frontera nula = null;
System.out.println(nula.metodoEstatico());   // imprime: un estatico no necesita instancia
```

Verificado en JDK 25: no lanza `NullPointerException`. El compilador solo usa el **tipo declarado** de la variable para localizar el método, y la referencia nunca se llega a leer. Es legal, es desconcertante y por eso todos los linters lo marcan: llamá siempre a los estáticos por el nombre de la clase.

**Cuándo usar cada uno.** Estático cuando el método no depende del estado de ningún objeto: `Math.max`, `Integer.parseInt`, `String.join`, un validador, una conversión. De instancia cuando la respuesta depende de los datos del objeto. La prueba rápida: si podés escribir el método sin tocar ningún campo, es estático, y hacerlo estático lo documenta.

## 7. Qué hace la JVM cuando llamás a un método

Al invocar un método pasan cuatro cosas, en orden:

1. Se evalúan los argumentos, de izquierda a derecha ([sección 15](#15-el-orden-de-evaluación-de-los-argumentos)).
2. Se crea un **marco de pila** (*stack frame*) nuevo: un bloque de memoria con espacio para los parámetros, las variables locales y la pila de operandos del método.
3. Se ejecuta el cuerpo. Al terminar, el marco se destruye y el valor de retorno pasa al marco de quien llamó.
4. La ejecución continúa justo después de la llamada.

El punto 2 es la clave física de la mitad de este capítulo: **los parámetros y las locales viven en el marco**, y el marco desaparece al volver. Por eso nada de lo que hagas con las variables locales de un método sobrevive a su retorno, y por eso una recursión demasiado profunda agota la pila ([sección 44](#44-la-pila-y-el-stackoverflowerror)).

En el bytecode, cada tipo de llamada tiene su propia instrucción. Volcando con `javap -c` una clase con métodos de varios tipos:

```
 2: invokestatic  #13   // Method suma:(II)I
 8: invokevirtual #19   // Method doble:(I)I
14: invokevirtual #23   // Method privado:(I)I
34: invokestatic  #32   // Method unir:(Ljava/lang/String;[Ljava/lang/String;)Ljava/lang/String;
39: invokeinterface #36,  1   // InterfaceMethod java/util/List.size:()I
47: invokevirtual #44   // Method java/lang/String.length:()I
```

Las cuatro instrucciones de invocación de la JVM son:

| Instrucción | Para qué |
|---|---|
| `invokestatic` | Métodos estáticos. No hay receptor. |
| `invokevirtual` | Métodos de instancia de una clase. El método concreto se decide en ejecución según el tipo real del objeto. |
| `invokeinterface` | Métodos llamados a través de una interfaz. |
| `invokespecial` | Constructores, `super.metodo()` y algunos casos de métodos privados. |

Hay un detalle verificado que contradice lo que dicen muchos textos: **el método privado del ejemplo se invoca con `invokevirtual`, no con `invokespecial`**. Desde Java 11, con el mecanismo de *nestmates* ([JEP 181](https://openjdk.org/jeps/181)), los miembros privados de un mismo *nest* son accesibles directamente y ya no requieren los métodos puente que el compilador generaba antes. Si leés un artículo anterior a 2018 que asegura que los privados siempre usan `invokespecial`, estaba en lo cierto entonces y no lo está ahora.

Lo que sí sobrevive intacto es lo importante: **en el `.class` queda escrito el descriptor completo** —`(II)I`, `(Ljava/lang/String;[Ljava/lang/String;)Ljava/lang/String;`— del método elegido. Eso será decisivo en la [sección 34](#34-la-resolución-ocurre-en-compilación-y-queda-grabada).

## 8. Lo que el método ve desde dentro

Un método ve tres grupos de cosas:

1. **Sus parámetros**, que son variables locales inicializadas con los argumentos.
2. **Sus propias variables locales**, declaradas en el cuerpo.
3. **Los campos accesibles**: los de la clase si es estático, más los del objeto si es de instancia.

Y no ve nada de quien lo llamó. Ni sus variables locales, ni el nombre con que le pasaron los valores, ni desde dónde le llamaron.

```java
static void f() {
    int secreto = 42;
    g();                 // g() no puede ver 'secreto' de ninguna manera
}

static void g() {
    // aquí 'secreto' no existe
}
```

Las reglas de visibilidad son las del capítulo [Variables and Scopes](04-variables-and-scopes.md), con dos añadidos propios de los métodos.

**Un parámetro es una variable local más.** Se puede leer, se puede escribir y se puede declarar `final`. Ocupa un hueco en el marco desde el primer momento, y por eso —a diferencia de una local declarada sin valor— **siempre está inicializado**.

**Un parámetro puede ocultar un campo** (*shadowing*):

```java
public class Usuario {
    private String nombre;

    public void setNombre(String nombre) {   // el parámetro oculta al campo
        this.nombre = nombre;                // 'this' desambigua
    }
}
```

Sin `this`, la línea `nombre = nombre;` se asignaría el parámetro a sí mismo y el campo quedaría intacto. Compila, no da ningún aviso y el objeto se queda vacío. Es un bug clásico de los primeros meses. El propio tutorial de Oracle avisa de que el *shadowing* de un campo por un parámetro se considera aceptable **solo** en constructores y en setters, precisamente porque ahí el `this.x = x` es un idioma reconocible; en cualquier otro método, poner al parámetro el mismo nombre que a un campo es pedir problemas.

---

# Parte II — Los parámetros son copias

## 9. Java pasa siempre por valor

Esta es la sección más importante del capítulo, y la que más literatura incorrecta tiene alrededor.

**Java pasa los argumentos siempre por valor. Sin excepciones, sin matices y sin casos especiales.** Lo que se copia es el contenido de la variable que pasás. Nada más.

Empecemos por el caso fácil, los primitivos:

```java
static void modificar(int x, int y) { x = 5; y = 10; }

int x = 1, y = 2;
modificar(x, y);
System.out.println("x=" + x + " y=" + y);
```

Salida real en JDK 25:

```
primitivos: x=1 y=2
```

Los parámetros `x` e `y` del método son variables **nuevas**, en el marco nuevo, inicializadas con una copia de 1 y 2. Asignarles otra cosa altera esas copias y nada más. Al volver, el marco desaparece con ellas dentro.

Hasta aquí no hay discusión: todas las fuentes lo cuentan bien. El desacuerdo empieza en la siguiente sección.

## 10. Qué cambia cuando el argumento es un objeto

Cuando el argumento es un objeto, lo que la variable **contiene** no es el objeto: es una referencia a él. Y eso es lo que se copia.

El resultado es que hay dos referencias distintas apuntando al mismo objeto. De ahí salen dos comportamientos que parecen contradictorios y no lo son:

```java
static void modificar(Foo a1, Foo b1) {
    a1.num++;          // muta el objeto compartido           -> SÍ se ve fuera
    b1 = new Foo(1);   // reasigna la copia de la referencia  -> NO se ve fuera
    b1.num++;
}

Foo a = new Foo(1), b = new Foo(1);
modificar(a, b);
```

Salida real en JDK 25:

```
objetos: a=Foo(2) b=Foo(1)
```

`a` cambió a 2 y `b` sigue en 1. Y la regla que explica ambos casos es una sola:

> Lo que se copia es la referencia. **Seguir la flecha y tocar el objeto** afecta a todo el mundo. **Cambiar hacia dónde apunta la flecha** afecta solo a la copia.

Vale la pena decirlo con las palabras que usa el tutorial de Oracle, que aquí es preciso: los tipos referencia también se pasan **por valor**; los valores de los campos del objeto *sí* pueden cambiarse dentro del método, pero reasignar el parámetro a un objeto nuevo **no tiene ningún efecto** sobre la referencia original.

Esta es la razón por la que la frase «Java pasa los objetos por referencia» es incorrecta pero se siente verdadera: los efectos de mutar coinciden con los de pasar por referencia. Solo se distinguen al reasignar, y la mayoría del código nunca reasigna un parámetro. La diferencia se hace visible en cuanto alguien intenta escribir un método de intercambio.

## 11. Por qué un método de intercambio no funciona

En un lenguaje con paso por referencia real —C++ con `&`, C# con `ref`— esto intercambia dos variables:

```java
static void intercambiar(Foo a, Foo b) {
    Foo t = a;
    a = b;
    b = t;
}

Foo p = new Foo(7), q = new Foo(8);
intercambiar(p, q);
```

Salida real en JDK 25:

```
swap: p=Foo(7) q=Foo(8)
```

**No pasó nada.** El método intercambió sus dos copias locales de las referencias, y al volver las tiró a la basura. `p` y `q` nunca se enteraron.

Este es el experimento decisivo, el que zanja la discusión: si Java pasara los objetos por referencia, este método funcionaría. No hay forma de escribir un `swap` genérico en Java, y esa imposibilidad **es** la demostración de que el paso es por valor.

Las alternativas cuando hace falta de verdad:

```java
// 1. Devolver el resultado (lo normal)
static <T> List<T> intercambiados(T a, T b) { return List.of(b, a); }

// 2. Intercambiar dentro de una estructura, que sí es un objeto compartido
Collections.swap(lista, 0, 1);

// 3. Un array o un contenedor de una casilla como caja mutable
static void intercambiar(Foo[] par) { Foo t = par[0]; par[0] = par[1]; par[1] = t; }
```

La tercera funciona porque el array **es** el objeto compartido y se está mutando su contenido, no reasignando el parámetro.

## 12. Arrays, colecciones y StringBuilder

Los tres casos que más aparecen en el día a día, verificados:

```java
static void tocarArray(int[] v) {
    v[0] = 99;                 // muta        -> se ve fuera
    v = new int[]{-1, -1};     // reasigna    -> no se ve fuera
}

static void tocarTexto(String s, StringBuilder sb) {
    s = s + " modificado";     // String es inmutable: crea otra, no se ve fuera
    sb.append(" modificado");  // muta el StringBuilder -> se ve fuera
    sb = new StringBuilder("otro");
    sb.append("!");            // sobre la copia: no se ve fuera
}

static void vaciar(List<String> lista) { lista.clear(); }
static void reemplazar(List<String> lista) { lista = new ArrayList<>(); lista.add("nuevo"); }
```

Salidas reales en JDK 25:

```
array: [99, 2, 3]
String: hola | StringBuilder: hola modificado
tras vaciar: []
tras reemplazar: [a]
```

Cuatro lecturas de esa salida:

- **El array cambió en la posición 0 y no en el resto.** Pasar un array a un método es entregar acceso de escritura a su contenido. No hay copia defensiva automática en ningún sitio de Java.
- **El `String` no cambió**, y no por el paso de parámetros, sino porque `String` es inmutable: `s + " modificado"` construye un objeto nuevo. Un parámetro `String` es, en la práctica, seguro.
- **El `StringBuilder` sí cambió**, porque es mutable y el método lo mutó. Pasar un `StringBuilder` es dar permiso para escribir.
- **`clear()` vació la lista de verdad; reasignarla no hizo nada.** Es el mismo par mutar/reasignar de siempre.

De aquí sale una regla de diseño que se retoma en la [sección 55](#55-métodos-puros-y-efectos): **el tipo de un parámetro comunica cuánto poder estás dando**. `String`, `int`, `LocalDate` o un `record` inmutable son seguros por construcción. `int[]`, `List`, `Map` o `StringBuilder` son permisos de escritura, aunque la firma no lo diga.

## 13. El error que cometen casi todas las fuentes

El artículo de Baeldung sobre paso de parámetros empieza perfecto — *«As far as Java is concerned, everything is strictly Pass-by-Value»* — desarrolla bien los dos casos, y luego **se contradice a sí mismo en la conclusión**, literalmente:

> *1. For Primitive types, parameters are pass-by-value*
> *2. For Object types, the object reference is pass-by-reference*

El punto 2 es falso, y el propio artículo lo demuestra dos párrafos más arriba: su ejemplo asigna `b1 = new Foo(1)` dentro del método y comprueba con un `assertEquals` que `b.num` sigue valiendo 1. **Si la referencia se pasara por referencia, esa aserción fallaría.** Lo reproduje en JDK 25 y el resultado es el que ya vimos: `a=Foo(2) b=Foo(1)`.

Lo que la conclusión quiere decir, mal dicho, es que *el efecto de mutar el objeto se ve desde fuera*. Lo que dice, tal como está escrito, es que Java tiene dos mecanismos de paso de parámetros. No los tiene.

El motivo de dedicarle una sección entera a un error de redacción ajeno es que **esta confusión concreta produce bugs**. Quien cree que los objetos se pasan por referencia escribe métodos de intercambio que no intercambian, espera que reasignar un parámetro se vea desde fuera, y no entiende por qué su `List` se vació cuando solo quería reemplazarla. La formulación correcta cabe en una línea:

> Java copia siempre el contenido de la variable. Si la variable contiene una referencia, se copia la referencia; el objeto no se copia nunca.

Y hay una prueba mecánica para decidir cualquier caso dudoso, sin teoría: **si el método usa `=` sobre el parámetro, no se ve fuera; si usa `.` para llegar al objeto, sí.**

## 14. Reasignar un parámetro, final y effectively final

Reasignar un parámetro es legal:

```java
static String normalizar(String texto) {
    texto = texto.strip();        // legal
    texto = texto.toLowerCase();  // legal
    return texto;
}
```

Jenkov lo describe bien y añade un aviso que conviene repetir: se puede, pero produce código confuso; si dudás, usá una variable local y dejá el parámetro intacto.

La razón de fondo es que el parámetro **documenta lo que se recibió**. Cuando lo reasignás, ese dato se pierde para el resto del método y para el depurador. Si en la línea 40 de un método largo `texto` ya no es lo que entró, cualquier lectura posterior parte de una premisa falsa. La alternativa cuesta una línea:

```java
static String normalizar(String textoOriginal) {
    String texto = textoOriginal.strip().toLowerCase();
    return texto;
}
```

Para impedirlo por completo está `final`:

```java
static String normalizar(final String texto) {
    texto = texto.strip();   // error: final parameter texto may not be assigned
    return texto;
}
```

Y aquí una precisión que Jenkov formula con exactitud y muchos otros no: `final` en un parámetro **solo impide reasignar la variable**. Si el parámetro es una referencia, la referencia no se puede cambiar, pero el objeto apuntado **sí se puede modificar**:

```java
static void f(final List<String> lista) {
    lista.add("esto sí se puede");     // legal: se muta el objeto
    lista = new ArrayList<>();          // error: no se puede reasignar
}
```

`final` no es inmutabilidad. Es solo una promesa de que el nombre seguirá apuntando al mismo sitio.

**Effectively final.** Desde Java 8 no hace falta escribir `final` para obtener su principal beneficio práctico: una variable o parámetro que **de hecho** nunca se reasigna es *effectively final*, y solo esas pueden capturarse dentro de una lambda o una clase anónima.

```java
static Runnable crear(String mensaje) {
    // mensaje = mensaje + "!";                 // si se descomenta, la lambda deja de compilar
    return () -> System.out.println(mensaje);
}
```

Con la línea comentada, compila. Al descomentarla, el error es `local variables referenced from a lambda expression must be final or effectively final`. Es decir: **reasignar un parámetro puede romper código que está tres líneas más abajo y que no lo menciona**. Otro motivo más para no hacerlo.

En cuanto a escribir `final` en todos los parámetros por sistema: es una discusión de estilo con años encima. Aporta poco a cambio de mucho ruido visual, y la propia biblioteca estándar no lo hace. Lo razonable es no ponerlo por defecto, y reservarlo para cuando el método es largo y querés garantizar que nadie reasigne.

## 15. El orden de evaluación de los argumentos

Java garantiza que los argumentos se evalúan **de izquierda a derecha**, y por completo, antes de entrar en el método. Es una garantía del lenguaje ([JLS §15.7](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.7)), no un detalle de implementación. Verificado:

```java
recibe(siguiente("a"), siguiente("b"), siguiente("c"));
```

```
  se evalua a -> 1
  se evalua b -> 2
  se evalua c -> 3
  recibe(1, 2, 3)
```

Que esté garantizado es una diferencia real con C y C++, donde el orden queda a criterio del compilador y el mismo código puede dar resultados distintos según la máquina. En Java el resultado es determinista.

Que sea determinista no significa que convenga depender de ello:

```java
procesar(cola.poll(), cola.poll(), cola.poll());   // legal, determinista y horrible
```

Funciona, siempre en el mismo orden, y aun así hay que reescribirlo: nadie que lea la llamada puede saber qué recibe cada parámetro sin conocer esta regla del JLS. Lo mismo vale para `f(i++, i)` y familia. El lenguaje define la respuesta; la persona que mantenga el código dentro de un año, no.

## 16. Los nombres de los parámetros no viajan al class

Un detalle que sorprende y tiene consecuencias prácticas: **los nombres de los parámetros no se guardan en el `.class` por defecto**. Verificado con reflection sobre una clase compilada de forma normal:

```
Nombres de los parametros vistos por reflection:
  int arg0 (nombre real presente? false)
  String arg1 (nombre real presente? false)
```

El compilador descarta `cantidad` y `descripcion` y deja `arg0` y `arg1`. `javap -v` confirma que no hay atributo `MethodParameters` en el fichero. Compilando con la opción `-parameters`, el atributo aparece y los nombres reales sobreviven.

Consecuencias reales:

- **Frameworks.** Spring, Jackson o JAX-RS pueden vincular parámetros por nombre solo si la clase se compiló con `-parameters`. Sin él hay que anotar todo a mano (`@RequestParam("cantidad")`, `@JsonProperty("cantidad")`). Spring Boot activa la opción en sus plantillas de Maven y Gradle precisamente por esto.
- **Mensajes de error.** Los mensajes útiles de `NullPointerException` ([JEP 358](https://openjdk.org/jeps/358), Java 14+) usan los nombres cuando están disponibles. Verificado en un fichero compilado sin información de depuración:

```
mensaje util de la JVM: Cannot invoke "String.length()" because "<local2>" is null
```

Con esa información presente, `<local2>` habría sido el nombre real de la variable. El `<local2>` que aparece ahí es exactamente lo que verás en un stacktrace de producción si el build no conserva esos datos.

- **Compatibilidad.** Cambiar el **nombre** de un parámetro no rompe nada binariamente, porque no forma parte de la firma ([sección 24](#24-la-firma-no-incluye-el-tipo-de-retorno)). Cambiar su **tipo** o su **orden**, sí.

---

# Parte III — El retorno

## 17. missing return statement

El compilador exige que **todos los caminos posibles** de un método no `void` terminen en un `return` o en un `throw`. No le basta con que exista un `return`: tiene que ser inevitable.

```java
static int sinRetorno(int n) {
    if (n > 0) { return 1; }
}
```

```
E1.java:4: error: missing return statement
  }
  ^
1 error
```

El error apunta a la llave de cierre, y eso es exactamente lo que significa: *«llegué hasta aquí y no había ningún valor que devolver»*. El compilador no razona sobre los valores posibles de `n`; razona sobre la estructura del código. Por eso esto también falla, aunque para un humano sea evidente que siempre devuelve algo:

```java
static int siempreDevuelve(boolean b) {
    if (b) { return 1; }
    if (!b) { return 2; }
}    // error: missing return statement
```

Y en cambio esto compila, porque `else` cierra la estructura:

```java
static int conElse(boolean b) {
    if (b) { return 1; }
    else   { return 2; }
}
```

Es el mismo análisis de alcanzabilidad del JLS que en el capítulo de bucles hacía que `while (false) {}` no compilase. Hay una excepción reconocible: un método cuyo cuerpo es `while (true)` sin `break` no necesita `return`, porque el compilador sabe que nunca llega al final.

La forma correcta de arreglar un `missing return statement` casi nunca es añadir un `return 0` al final. Ese cero es un valor inventado que el método devolverá en un caso que el autor no previó. Lo correcto suele ser:

```java
static int buscar(int[] v, int objetivo) {
    for (int i = 0; i < v.length; i++) {
        if (v[i] == objetivo) return i;
    }
    return -1;    // valor centinela DOCUMENTADO, no un relleno
}
```

O directamente lanzar, si no encontrar nada es un error de verdad. Un `return 0` puesto para callar al compilador es un bug esperando su turno.

## 18. Código inalcanzable después de un return

```java
static int tras(int n) {
    return n;
    System.out.println("nunca");
}
```

```
E2.java:4: error: unreachable statement
    System.out.println("nunca");
    ^
1 error
```

A diferencia de otros lenguajes, en Java el código inalcanzable es **un error, no un aviso**. La razón es que casi siempre indica un malentendido: alguien creía que `return` no cortaba el flujo, o hay un `return` de más pegado durante un refactor.

Cuidado con un caso vecino que **sí** compila y que no es lo que parece:

```java
static int f(int n) {
    if (true) return 1;
    return 2;              // compila: el JLS exceptúa el 'if' del análisis
}
```

Es la misma excepción de alcanzabilidad que ya apareció en el capítulo de [Loops](11-loops.md), la que existe para permitir la compilación condicional con `if (DEBUG)`. `while (true) return 1;` seguido de otra sentencia, en cambio, no compila.

## 19. return dentro de un finally

Este es el bug más silencioso de toda la Parte III.

```java
static int seTragaLaExcepcion() {
    try {
        throw new IllegalStateException("algo falló de verdad");
    } finally {
        return 0;
    }
}
```

Salida real en JDK 25:

```
seTragaLaExcepcion() = 0
```

**La excepción desapareció.** No se propaga, no se registra, no queda rastro de ella. El método devuelve 0 como si todo hubiera ido bien. Un `return` dentro de un `finally` descarta cualquier excepción en curso y también cualquier `return` del `try`.

El segundo caso es más sutil y también está verificado:

```java
static int pisaElValor() {
    int x = 1;
    try {
        return x;
    } finally {
        x = 99;
    }
}
```

```
pisaElValor() = 1
```

Devuelve **1**, no 99. El valor de retorno ya se copió al evaluar `return x`; que el `finally` cambie `x` después no altera esa copia. Es coherente con toda la Parte II —lo que se copia, se copió— pero sorprende a casi todo el mundo la primera vez.

Ambas cosas son legales y ambas están señaladas por las herramientas: `javac -Xlint:finally` avisa con `finally clause cannot complete normally`, y todos los analizadores estáticos lo marcan como error grave. La regla es simple y no admite excepciones:

> **Nunca pongas `return`, `break`, `continue` ni `throw` dentro de un `finally`.** El `finally` es para liberar recursos, y desde Java 7 ni siquiera para eso: para eso está `try-with-resources`.

## 20. Devolver null

Devolver `null` es legal y a veces inevitable, pero es una decisión de diseño con consecuencias: **traslada al que llama la obligación de acordarse**, y el compilador no le va a ayudar a acordarse.

```java
static Usuario buscar(String id) {
    // ...
    return null;   // no encontrado
}

buscar("x").getNombre();   // NullPointerException en cuanto alguien lo olvide
```

Las alternativas, en orden de preferencia:

```java
// 1. Optional, cuando "no hay resultado" es un caso normal
static Optional<Usuario> buscar(String id) { ... }
buscar("x").map(Usuario::getNombre).orElse("desconocido");

// 2. Lanzar, cuando no encontrarlo es un error del programa
static Usuario obtener(String id) {
    Usuario u = repo.buscar(id);
    if (u == null) throw new NoSuchElementException("no existe el usuario " + id);
    return u;
}

// 3. Un objeto neutro, cuando existe uno con sentido
static List<Pedido> pedidosDe(String id) { return List.of(); }   // nunca null
```

La tercera merece una regla propia, que vale la pena grabar: **un método que devuelve una colección o un array no debe devolver `null` nunca**. Debe devolver la colección vacía. `null` obliga a quien llama a escribir un `if` antes de cada bucle, y ese `if` es exactamente el que se olvida. La biblioteca estándar lo respeta: `List.of()`, `Collections.emptyList()` y `new int[0]` existen para eso.

Sobre `Optional`: es un tipo de **retorno**, no de parámetro ni de campo. Un `Optional` como parámetro obliga a quien llama a envolver el valor, y admite tres estados (`null`, vacío, presente) donde solo hacían falta dos. Esa recomendación viene de los propios diseñadores de la API en Java 8.

## 21. Ignorar el valor devuelto

Java permite descartar el valor devuelto por un método sin decir nada:

```java
lista.remove("x");        // devuelve boolean; casi siempre se ignora, y está bien
texto.trim();             // devuelve String; ignorarlo es SIEMPRE un bug
```

La segunda línea es un clásico absoluto. `String` es inmutable: `trim()` no modifica nada, devuelve otra cadena. Ignorar el resultado equivale a no haber hecho nada. Lo mismo con `toUpperCase`, `replace`, `substring`, `concat`, `strip` y toda la familia; y también con `BigDecimal.add`, `LocalDate.plusDays` o `Stream.filter`.

La forma correcta:

```java
texto = texto.trim();
```

Hay señales que ayudan a distinguir cuándo ignorar es un error: si el método devuelve **el mismo tipo del objeto sobre el que se llama** y ese tipo es inmutable, ignorarlo es siempre un bug. Muchas APIs modernas lo marcan con `@CheckReturnValue` (Error Prone, JSR-305) para que la herramienta avise; el JDK no tiene todavía una anotación estándar para esto.

## 22. Devolver estructuras mutables

Un método que devuelve una referencia a una estructura interna está entregando permiso de escritura sobre las tripas del objeto, exactamente igual que un parámetro mutable pero en el otro sentido. Verificado:

```java
static class Pedido {
    private final List<String> lineas = new ArrayList<>(List.of("camiseta"));
    private final int[] cantidades = {1};

    List<String> getLineasMal()  { return lineas; }               // expone el interior
    int[] getCantidadesMal()     { return cantidades; }           // expone el array
}

Pedido p = new Pedido();
p.getLineasMal().add("intruso");
p.getCantidadesMal()[0] = 9999;
```

```
tras tocar los getters mal: [camiseta, intruso] [9999]
```

El objeto quedó modificado desde fuera **sin usar ningún setter**, y el `private final` no impidió nada: `final` protege la referencia, no el contenido. Es el mismo malentendido de la [sección 14](#14-reasignar-un-parámetro-final-y-effectively-final), visto desde el retorno.

Las defensas, de más barata a más segura:

```java
List<String> getLineasBien()      { return List.copyOf(lineas); }              // copia inmutable
List<String> getLineasSoloLect()  { return Collections.unmodifiableList(lineas); } // vista de solo lectura
int[] getCantidadesBien()         { return cantidades.clone(); }               // copia del array
```

```
la copia inmutable rechaza el add: java.lang.UnsupportedOperationException
```

Las dos primeras no son equivalentes: `List.copyOf` hace una copia, así que los cambios posteriores en el original **no** se ven; `unmodifiableList` es una vista, así que sí se ven, y solo impide escribir a través de ella. Para un getter de un objeto que puede cambiar, la vista suele ser lo que se quiere; para devolver algo que se va a guardar por ahí, la copia.

Y la regla simétrica, igual de importante: **también hay que copiar al entrar**. Si un constructor guarda directamente la `List` que le pasan, quien se la pasó conserva una referencia y puede modificar el objeto después de construido.

```java
Pedido(List<String> lineas) {
    this.lineas = new ArrayList<>(lineas);   // copia defensiva de entrada
}
```

---

# Parte IV — Sobrecarga

## 23. Qué es la sobrecarga y qué la define

**Sobrecargar** (*overloading*) es declarar varios métodos con el mismo nombre y distinta lista de parámetros en el mismo ámbito.

```java
static void display(int a)    { System.out.println("int: " + a); }
static void display(String a) { System.out.println("String: " + a); }
static void display(double a) { System.out.println("double: " + a); }
```

Sirve para dar **un solo nombre a una sola idea** que se puede expresar con datos de distinta forma. La biblioteca estándar la usa a fondo: `System.out.println` tiene diez sobrecargas, `Math.max` cuatro, `String.valueOf` nueve. El caso contrario —`printlnInt`, `printlnDouble`, `printlnBoolean`— sería un desastre de API, y es el argumento que usan tanto Baeldung como Programiz para introducir el tema. Es correcto.

Lo que distingue una sobrecarga de otra es **el número, el tipo y el orden de los parámetros**. No cuenta nada más. En particular no cuentan:

- el tipo de retorno,
- los nombres de los parámetros,
- los modificadores (`public`, `static`, `final`…),
- la cláusula `throws`.

Baeldung lo verifica una por una en su artículo sobre firmas, y el resultado es el mismo en JDK 25: cambiar cualquiera de esas cuatro cosas da `method is already defined in class`.

## 24. La firma no incluye el tipo de retorno

La **firma** de un método es, según el [JLS §8.4.2](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.2), su nombre, sus parámetros de tipo (si es genérico) y los tipos de sus parámetros formales. Literalmente:

> *«Two methods or constructors, M and N, have the same signature if they have the same name, the same type parameters (if any), and, after adapting the formal parameter types of N to the type parameters of M, the same formal parameter types.»*

Y la consecuencia, también literal:

> *«It is a compile-time error to declare two methods with override-equivalent signatures in a class.»*

El tipo de retorno no aparece por ningún lado. Ojo con un matiz que casi nadie menciona: **el `.class` sí guarda el tipo de retorno**, dentro de lo que la JVM llama el *descriptor* (`(II)I` = dos `int`, devuelve `int`). El JLS lo dice explícitamente en §15.12.2: *«The descriptor (signature plus return type) of the most specific method is the one used at run time to perform the method dispatch.»* Es decir: la **firma** no incluye el retorno, el **descriptor** sí, y la resolución de sobrecarga trabaja con la firma.

De ahí la regla práctica: dos métodos que solo se diferencian en el tipo de retorno no pueden coexistir.

## 25. El error real cuando dos métodos chocan

Aquí hay un error de una fuente muy leída que merece la pena desmontar porque enseña algo. Baeldung, en su artículo sobre sobrecarga y sobrescritura, presenta este par:

```java
public int multiply(int a, int b)    { return a * b; }
public double multiply(int a, int b) { return a * b; }
```

y explica:

> *«In this case, the code simply wouldn't compile because of the method call ambiguity – the compiler wouldn't know which implementation of multiply() to call.»*

**La conclusión es correcta y la razón es falsa.** Verificado en JDK 25 con un caso equivalente:

```
FirmaRetorno.java:3: error: method print() is already defined in class FirmaRetorno
    public int print() { return 0; }
               ^
1 error
```

El error es `already defined in class`, no una ambigüedad. La diferencia no es cosmética, son tres cosas distintas:

1. Es un error **de declaración**, no de invocación. Se produce al declarar el segundo método, aunque **nadie lo llame nunca**. Basta con escribir la clase y compilar; no hace falta ninguna llamada.
2. La causa es que ambos tienen la **misma firma** —`print()`—, así que para el compilador uno es un duplicado del otro. No hay dos métodos entre los que elegir: hay un método declarado dos veces.
3. La ambigüedad real es otra cosa y tiene otro mensaje. Así se ve, verificada:

```
Ambiguo.java:9: error: reference to f is ambiguous
        f(null);
        ^
  both method f(String) in Ambiguo and method f(StringBuilder) in Ambiguo match
```

Ahí sí hay dos métodos distintos y perfectamente declarados, y el problema aparece **en la llamada**, no en la declaración.

Lo curioso es que el propio Baeldung, en su otro artículo —el de las firmas— reporta el mensaje correcto (*«we get a "method is already defined in class" error»*). Dos artículos del mismo sitio, sobre el mismo hecho, con explicaciones incompatibles. Es un buen recordatorio de que en Java **el mensaje del compilador es la fuente de verdad**, y comprobarlo cuesta veinte segundos.

## 26. Sobrecargar cambiando solo el tipo

Programiz cierra su página de sobrecarga con una lista de «puntos importantes» que contiene esta frase:

> *«It is not method overloading if we only change the return type of methods. There must be differences in the number of parameters.»*

La primera mitad es correcta. **La segunda es falsa**, y la contradice el propio artículo dos pantallas más arriba, donde dice que los métodos pueden diferir en «different number of parameters, different types of parameters, or both», y donde su ejemplo nº 2 sobrecarga `display(int)` con `display(String)` — mismo número de parámetros, distinto tipo.

Verificado en JDK 25 con tres sobrecargas de un solo parámetro cada una:

```
int: 1
String: Hola
double: 1.5
int: 97
double: 1.0
```

Compila y funciona. Las dos últimas líneas de esa salida son las interesantes:

- `display('a')` imprime **`int: 97`**. No hay `display(char)`, así que el `char` se ensancha a `int` y llega su valor Unicode.
- `display(1L)` imprime **`double: 1.0`**. No hay `display(long)`, así que el `long` se ensancha a `double` — no a `int`, que sería estrechar y no está permitido implícitamente.

Ambas son la mecánica del capítulo de [Type Casting](05-type-casting.md) aplicada a la selección de métodos, y ambas son el aperitivo de la sección siguiente.

## 27. Las tres fases de la resolución

Cuando hay varias sobrecargas candidatas, el compilador no elige "la que mejor pinta tiene": ejecuta un algoritmo de **tres fases** definido en el [JLS §15.12.2](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.12.2). Con las palabras de la especificación:

> *«Then, to ensure compatibility with the Java programming language prior to Java SE 5.0, the process continues in three phases.»*

| Fase | Qué permite | Qué **no** permite |
|---|---|---|
| **1 — strict invocation** | Conversiones de ensanchamiento (`int` → `long` → `double`) y subtipos | boxing, unboxing, varargs |
| **2 — loose invocation** | Todo lo anterior **más** boxing y unboxing | varargs |
| **3 — variable arity invocation** | Todo lo anterior **más** varargs | — |

El algoritmo prueba la fase 1 con todos los candidatos. Si encuentra al menos uno aplicable, **para ahí** y elige entre esos; solo si no encuentra ninguno pasa a la fase 2, y solo si tampoco, a la 3.

El porqué de este diseño está escrito en la propia especificación, y es historia del lenguaje:

> *«This guarantees that any calls that were valid in the Java programming language before Java SE 5.0 are not considered ambiguous as the result of the introduction of variable arity methods, implicit boxing and/or unboxing.»*

Java 5 introdujo a la vez el autoboxing y los varargs. Sin las fases, cualquier programa de Java 1.4 que llamara a un método sobrecargado podría haber empezado a compilar de otra manera —o a no compilar— al recompilarlo con el compilador nuevo. Las fases son una promesa de compatibilidad hacia atrás, no una elegancia de diseño.

La especificación añade un aviso que conviene leer despacio:

> *«However, the declaration of a variable arity method can change the method chosen for a given method invocation expression, because a variable arity method is treated as a fixed arity method in the first phase. For example, declaring `m(Object...)` in a class which already declares `m(Object)` causes `m(Object)` to no longer be chosen for some invocation expressions (such as `m(null)`), as `m(Object[])` is more specific.»*

Es decir: **añadir una sobrecarga puede cambiar a qué método va una llamada existente, sin tocar esa llamada**. Es el riesgo central de la sobrecarga y el argumento de la [sección 36](#36-cuándo-no-sobrecargar).

## 28. Fase 1: ensanchamiento

Con estas tres sobrecargas y la llamada `f(1)`:

```java
static void f(long x)     { System.out.println("f(long) — fase 1: widening"); }
static void f(Integer x)  { System.out.println("f(Integer) — fase 2: boxing"); }
static void f(int... x)   { System.out.println("f(int...) — fase 3: varargs"); }

f(1);
```

Salida real en JDK 25:

```
f(long) — fase 1: widening
```

Gana `f(long)`. El literal `1` es un `int`, no hay `f(int)`, y el ensanchamiento `int` → `long` está permitido en la fase 1. Como la fase 1 ya encontró un candidato, las otras dos ni se miran, aunque `f(Integer)` parezca "más natural" para quien lea el código.

Esta es la parte que Baeldung llama *type promotion* y describe correctamente. Su ejemplo: con `multiply(int, long)` y `multiply(int, int, int)` declarados, llamar con dos `int` promociona el segundo a `long`.

## 29. Fase 2: boxing

Si se elimina `f(long)` del ejemplo anterior, la fase 1 se queda sin candidatos —no hay ninguna conversión de ensanchamiento de `int` a `Integer`, porque el boxing **no** es un ensanchamiento— y entra la fase 2, que sí lo permite. Entonces gana `f(Integer)`.

El orden entre las dos primeras fases tiene una consecuencia práctica que salta en las entrevistas:

```java
static void g(long x)    { }
static void g(Integer x) { }

g(5);    // llama a g(long), no a g(Integer)
```

Ensanchar es **más barato** que empaquetar —no crea ningún objeto— y el lenguaje lo prefiere siempre. La regla corta: **primitivo antes que wrapper, siempre**.

Y otra que sale de la misma tabla: `f(int)` y `f(Integer)` pueden coexistir, porque `int` e `Integer` son tipos distintos. Verificado:

```
f(int)
f(Integer)
f(Object)
```

`f(1)` va a `f(int)` (fase 1, coincidencia exacta), `f(Integer.valueOf(1))` va a `f(Integer)` y `f("texto")` va a `f(Object)`. Tener las tres declaradas a la vez es legal y es una mala idea, porque el lector tiene que ejecutar el algoritmo en la cabeza para saber qué pasa.

## 30. Fase 3: varargs

La fase 3 es la última y solo se alcanza si las otras dos fallaron. De ahí sale la regla que la especificación enuncia sin rodeos:

> *«This ensures that a method is never chosen through variable arity method invocation if it is applicable through fixed arity method invocation.»*

En cristiano: **si existe una sobrecarga de aridad fija que sirve, el varargs nunca se usa**. Verificado por Baeldung en su artículo de firmas y reproducible:

```java
Number sum(Object term1, Object term2)     { ... }
Number sum(Object term1, Object... term2)  { ... }

obj.sum(new Object(), new Object());                       // -> sum(Object, Object)
obj.sum(new Object(), new Object(), new Object());         // -> sum(Object, Object...)
obj.sum(new Object(), new Object[]{new Object()});         // -> sum(Object, Object...)
```

Con dos argumentos gana siempre la versión fija. Para forzar la de varargs con dos argumentos hay que envolver el segundo en un array explícito.

## 31. El más específico gana, y el caso de null

Cuando varios candidatos son aplicables **en la misma fase**, el compilador elige el **más específico** ([JLS §15.12.2.5](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.12.2.5)). Informalmente: gana el que podría pasarle sus argumentos a los demás sin error.

De ahí sale el comportamiento de `null`, que sorprende a casi todo el mundo:

```java
static void g(Object o) { System.out.println("g(Object)"); }
static void g(String s) { System.out.println("g(String)"); }

g(null);
```

Salida real en JDK 25:

```
g(String)
```

`null` es compatible con cualquier tipo referencia, así que ambos son aplicables; entre `String` y `Object` gana `String` porque toda `String` es un `Object` y no al revés. Nada de esto depende del valor: es puro análisis de tipos en tiempo de compilación.

La consecuencia práctica es incómoda: **el día que alguien añada `g(Integer)`, la llamada `g(null)` dejará de compilar** por ambigüedad, aunque nadie haya tocado esa línea. Es exactamente el escenario del que avisa el JLS.

## 32. Cuando ninguno es más específico

Si dos candidatos son aplicables y ninguno es más específico, el compilador se planta:

```java
static void f(String s)        { }
static void f(StringBuilder b) { }

static void g(int a, long b) { }
static void g(long a, int b) { }

f(null);
g(1, 1);
```

```
Ambiguo.java:9: error: reference to f is ambiguous
  both method f(String) in Ambiguo and method f(StringBuilder) in Ambiguo match
Ambiguo.java:10: error: reference to g is ambiguous
  both method g(int,long) in Ambiguo and method g(long,int) in Ambiguo match
```

Los dos casos son distintos y los dos son clásicos. En `f(null)`, `String` y `StringBuilder` no tienen relación de herencia entre sí, así que ninguno gana. En `g(1, 1)`, cada candidato es mejor en un parámetro y peor en el otro; el JLS incluye este mismo patrón como ejemplo de ambigüedad, con `test(ColoredPoint, Point)` y `test(Point, ColoredPoint)`.

Las salidas cuando aparece esto:

```java
f((String) null);          // cast explícito para elegir
g(1, 1L);                  // fijar el tipo del literal
static void gConLong(long a, int b) { }   // renombrar y acabar con el problema
```

La tercera suele ser la correcta. Si dos sobrecargas son tan parecidas que el compilador no puede distinguirlas, el lector tampoco va a poder.

## 33. Ensanchamiento y boxing no se combinan

Una regla que parece arbitraria hasta que se conocen las fases:

```java
static void f(Long x) { }

int i = 1;
f(i);
```

```
NoCombina.java:6: error: incompatible types: int cannot be converted to Long
```

El razonamiento intuitivo —«`int` se ensancha a `long`, y `long` se empaqueta en `Long`»— es exactamente lo que **no** se permite. En la fase 1 solo hay ensanchamiento, y `Long` no es un ensanchamiento de `int`; en la fase 2 hay boxing, pero el boxing de un `int` produce `Integer`, y `Integer` no es subtipo de `Long`. Ninguna fase encadena las dos conversiones.

Las soluciones son explicitar el paso que falta:

```java
f((long) i);            // primero ensanchar, luego boxing implícito
f(Long.valueOf(i));     // convertir a mano
```

Es la misma familia de sorpresas que `List<Long> l; l.remove(1)` o que `Map<Long, X>.get(1)`, que compila y nunca encuentra nada porque busca un `Integer` en un mapa de claves `Long`.

## 34. La resolución ocurre en compilación, y queda grabada

Este es el experimento más importante del capítulo, porque desmonta la idea de que la sobrecarga es solo azúcar sintáctico.

Se compila un cliente contra una biblioteca con un solo método:

```java
public class Biblioteca {
    public static String saludar(Object o) { return "version 1: saludar(Object)"; }
}

public class Cliente {
    public static void main(String[] args) {
        System.out.println(Biblioteca.saludar("hola"));
    }
}
```

```
-- cliente compilado contra la version 1:
version 1: saludar(Object)
```

Ahora se **sustituye solo el `.class` de la biblioteca** por una versión que añade una sobrecarga más específica, sin recompilar el cliente:

```java
public class Biblioteca {
    public static String saludar(Object o) { return "version 2: saludar(Object)"; }
    public static String saludar(String s) { return "version 2: saludar(String)"; }
}
```

```
-- se sustituye SOLO el .class de la biblioteca (sin recompilar el cliente):
version 2: saludar(Object)
-- ahora se recompila tambien el cliente:
version 2: saludar(String)
```

Verificado en JDK 25. El mismo binario del cliente, con la misma biblioteca nueva, llama a `saludar(Object)`; recompilarlo —sin cambiar una sola letra de su código fuente— hace que llame a `saludar(String)`.

La explicación está en la [sección 7](#7-qué-hace-la-jvm-cuando-llamás-a-un-método): el compilador escribió en el `.class` del cliente el descriptor exacto del método elegido, `(Ljava/lang/Object;)Ljava/lang/String;`. La JVM en ejecución no reevalúa la sobrecarga; busca ese descriptor concreto. El JLS documenta este comportamiento con un ejemplo propio (15.12.2-3) y lo resume así:

> *«The most specific method is chosen at compile time; its descriptor determines what method is actually executed at run time. If a new method is added to a class, then source code that was compiled with the old definition of the class might not use the new method, even if a recompilation would cause this method to be chosen.»*

Tres consecuencias que hay que llevarse:

1. **La sobrecarga se resuelve en compilación** (*static binding*); la **sobrescritura** de un método heredado se resuelve en ejecución (*dynamic binding*). Son mecanismos distintos que comparten el aire de familia y poco más. La sobrescritura se ve en el bloque de POO.
2. **Añadir una sobrecarga a una biblioteca no es un cambio inocuo.** Es compatible a nivel binario —los clientes viejos siguen funcionando— pero puede cambiar el comportamiento de los clientes que se recompilen. Ese "puede" es exactamente lo que convierte un despliegue rutinario en una tarde larga.
3. **Si observás un comportamiento que no cuadra con el código fuente, sospechá de un `.class` antiguo.** Es la causa real de una parte de los "esto es imposible" de los proyectos grandes.

## 35. Las trampas de la biblioteca estándar

La propia JDK tiene sobrecargas que muerden. Estas cuatro, verificadas:

**`List.remove`.** `List<E>` declara `remove(int index)` y `remove(Object o)`. Con una lista de enteros, el desastre es inevitable:

```java
List<Integer> numeros = new ArrayList<>(List.of(10, 20, 30));
numeros.remove(1);                        // ¿borra el 1, o la posición 1?
numeros.remove(Integer.valueOf(10));
```

```
lista inicial: [10, 20, 30]
tras remove(1): [10, 30]
tras remove(Integer.valueOf(10)): [30]
```

`remove(1)` borró **la posición 1**, es decir el 20. La fase 1 encontró `remove(int)` con coincidencia exacta y ni miró `remove(Object)`. Para borrar por valor hay que empaquetar a mano.

**`println(char[])`.** `System.out.println` tiene una sobrecarga para `char[]` que imprime el contenido, mientras que la concatenación con `+` usa `toString()` y produce la representación del array:

```java
char[] letras = {'h','o','l','a'};
System.out.println(letras);          // imprime: hola
System.out.println("" + letras);     // imprime algo como [C@2f92e0f4
```

```
println(char[]) imprime: hola
concatenado con texto: true
```

(El `true` de la salida confirma que la cadena concatenada empieza por `[C@`.) Es la única sobrecarga de `println` que trata un array como contenido y no como objeto, y por eso `println("Valor: " + letras)` no imprime lo mismo que `println(letras)`.

**`printf` con un array.** Al ser `printf(String, Object...)`, pasarle un `Object[]` no lo mete como un solo argumento: lo **expande**.

```java
Object[] datos = {"Ana", 30};
System.out.printf("  %s tiene %d%n", datos);   // Ana tiene 30
```

**`Arrays.asList`** con un array de primitivos. Se trata en la [sección 40](#40-varargs-y-arrays-de-primitivos), y es la peor de todas.

## 36. Cuándo no sobrecargar

Con todo lo anterior sobre la mesa, la recomendación práctica es más restrictiva de lo que sugieren los tutoriales:

**Sobrecargá cuando:**

- Los métodos hacen **conceptualmente lo mismo** con datos equivalentes: `Math.abs(int)`, `Math.abs(double)`.
- Una versión es la otra con un parámetro por defecto: `crear(nombre)` que llama a `crear(nombre, LocalDate.now())`.
- Las listas de parámetros tienen **distinto número** de elementos: es el caso sin ambigüedad posible.

**No sobrecargues cuando:**

- Los métodos hacen **cosas distintas**. `procesar(Pedido)` y `procesar(Devolucion)` deberían llamarse `procesarPedido` y `procesarDevolucion`. Un nombre distinto no cuesta nada y elimina toda esta sección del problema.
- Los tipos de los parámetros son **convertibles entre sí** (`int`/`long`/`Integer`/`Object`). Ahí es donde las tres fases muerden.
- El mismo número de parámetros con tipos **relacionados por herencia**, si alguien puede pasar `null`.
- Estás escribiendo una **API pública o una biblioteca**: añadir sobrecargas después es un cambio con efectos a distancia, como demostró la sección anterior.

La alternativa casi siempre disponible son los **métodos factoría con nombre**, el patrón que la propia JDK adoptó al modernizarse:

```java
// en vez de sobrecargar el constructor tres veces
LocalDate.of(2026, 9, 1);
LocalDate.parse("2026-09-01");
LocalDate.ofEpochDay(20700);
```

Tres nombres distintos, cero ambigüedad, y la firma explica de dónde viene el dato.

---

# Parte V — Varargs

## 37. Qué es un varargs y en qué se traduce

Un parámetro **varargs** (*variable arity*, tres puntos suspensivos tras el tipo) permite llamar al método con cualquier número de argumentos de ese tipo, incluido ninguno.

```java
static String unir(String separador, String... partes) {
    return String.join(separador, partes);
}

unir("-");
unir("-", "a");
unir("-", "a", "b", "c");
```

Existe desde Java 5, y como bien resume Baeldung, antes había que elegir entre pasar un array explícito o declarar N sobrecargas.

Lo importante es que **un varargs es un array**. No es una construcción mágica: es azúcar sintáctico que el compilador convierte en un array en el **sitio de la llamada**. El bytecode de `unir("-", "a", "b")` lo enseña sin ambigüedad:

```
18: ldc           #26   // String -
20: iconst_2
21: anewarray     #8    // class java/lang/String
24: dup
25: iconst_0
26: ldc           #28   // String a
28: aastore
29: dup
30: iconst_1
31: ldc           #30   // String b
33: aastore
34: invokestatic  #32   // Method unir:(Ljava/lang/String;[Ljava/lang/String;)Ljava/lang/String;
```

`anewarray` crea un `String[2]`, dos `aastore` lo rellenan, y el método recibe **un array**. El descriptor lo confirma: `(Ljava/lang/String;[Ljava/lang/String;)`. En la firma que ve la JVM, `String...` es exactamente `String[]`.

La única diferencia que queda en el `.class` es un flag:

```
static java.lang.String unir(java.lang.String, java.lang.String...);
  descriptor: (Ljava/lang/String;[Ljava/lang/String;)Ljava/lang/String;
  flags: (0x0088) ACC_STATIC, ACC_VARARGS
```

`ACC_VARARGS` es lo único que le dice al compilador que puede construir el array por su cuenta cuando alguien llama a este método. En ejecución no significa nada.

Dos consecuencias de que el array se cree en el sitio de la llamada:

- **Cada llamada asigna un array nuevo.** Es barato pero no gratis, y por eso APIs muy calientes como los `Logger` declaran sobrecargas de 0, 1 y 2 argumentos además de la de varargs.
- **Dentro del método, el parámetro se usa como cualquier array**: `partes.length`, `partes[0]`, `for (String p : partes)`.

## 38. Las reglas de los varargs

Son solo dos, y el compilador las hace cumplir. Verificadas:

**1. Debe ser el último parámetro.**

```
Errores.java:10: error: varargs parameter must be the last parameter
    static void varargsNoUltimo(int... n, String s) { }
```

**2. Solo puede haber uno.** Es la misma regla vista desde otro ángulo: si hubiera dos, el primero no sería el último.

```
Errores.java:11: error: varargs parameter must be the last parameter
    static void dosVarargs(int... a, String... b) { }
```

Y una tercera que no es una regla sobre varargs sino sobre firmas, y que atrapa a todo el mundo alguna vez:

```java
static void f(String... a) { }
static void f(String[] a)  { }
```

```
E3.java:3: error: cannot declare both f(String[]) and f(String...) in E3
```

Como el varargs **es** un array en la firma, ambos métodos tienen la misma firma y chocan. Baeldung lo menciona correctamente: declarar la versión con `Object[]` colisiona con la de varargs.

Un apunte que sorprende: `main` puede declararse como varargs, y la JVM lo acepta como punto de entrada.

```java
public static void main(String... args) { }
```

```
main con varargs funciona, args.length=2
```

Es legal porque el descriptor resultante, `([Ljava/lang/String;)V`, es idéntico al de la forma habitual. Funciona, pero la convención es tan fuerte que escribirlo así solo consigue que el lector se detenga.

## 39. Nunca es null, salvo que insistas

Dentro del método, un parámetro varargs **nunca es `null` cuando no se pasan argumentos**: es un array vacío. Verificado:

```
--- varargs sin argumentos:
  n == null? false | length = 0
  n == null? false | length = 3
```

Esto es una garantía muy útil: no hace falta comprobar `null` antes de recorrerlo. `for (int x : n)` sobre cero elementos simplemente no itera.

Pero hay una forma de romperlo, y es pasar explícitamente un array nulo:

```java
recibir((String) null);      // un solo argumento, que vale null
recibir((String[]) null);    // el array ENTERO es null
```

```
recibir((String) null):
  partes == null? false
  length = 1
recibir((String[]) null):
  partes == null? true
  length = -1
```

En el primer caso el compilador construye un `String[1]` cuyo único elemento es `null`. En el segundo entiende que ya le estás dando el array, y le das `null`; el método recibe `null` y el primer `partes.length` lanza `NullPointerException`.

El caso peligroso de verdad es sin cast, cuando el tipo lo permite:

```java
recibir(null);   // aviso del compilador y, en ejecución, partes == null
```

`javac` avisa con `non-varargs call of varargs method with inexact argument type for last parameter`, y sugiere los dos casts. Es uno de los pocos avisos del compilador que conviene tratar siempre como error.

## 40. Varargs y arrays de primitivos

Esta es la trampa que más veces se ha visto en producción, y es consecuencia directa de que los primitivos no son objetos. Verificada:

```java
int[] primitivos = {1, 2, 3};
List<int[]> l1 = Arrays.asList(primitivos);

Integer[] wrappers = {1, 2, 3};
List<Integer> l2 = Arrays.asList(wrappers);
```

```
Arrays.asList sobre int[]:
  tamanio = 1 -> int[]
  con Integer[] tamanio = 3
```

`Arrays.asList` está declarado como `asList(T... a)`, y `T` solo puede ser un tipo referencia. Con un `int[]`, el único `T` posible es **`int[]` mismo**, así que el compilador construye un `int[][]` de un elemento y devuelve una lista con **un** elemento, que es el array entero. Con `Integer[]`, `T` es `Integer` y la lista tiene tres.

Lo grave es que **compila sin avisos** si el destino es `List<int[]>` o si se usa `var`. El síntoma aparece más tarde, como un `size()` que vale 1.

El mismo efecto con cualquier varargs genérico, verificado:

```
contar(int[3])     = 1
contar(Integer[3]) = 3
```

Las soluciones:

```java
List<Integer> lista = Arrays.stream(primitivos).boxed().toList();   // Java 16+
List<Integer> lista2 = IntStream.of(primitivos).boxed().collect(Collectors.toList());
```

Y la regla mnemotécnica: **un `int[]` es un objeto; tres `int` no lo son**. Cada vez que un array de primitivos se acerque a un varargs genérico, hay que parar y mirar.

## 41. Varargs y sobrecarga

Ya lo dice la fase 3: si hay una alternativa de aridad fija aplicable, gana ella. Pero hay un caso que sorprende incluso sabiéndolo, y es el que la especificación pone como ejemplo:

```java
static void m(Object o)    { }
static void m(Object... o) { }

m(null);   // llama a m(Object...), no a m(Object)
```

Ambos son aplicables en la fase 1 —`m(Object...)` se trata como `m(Object[])`—, y entre `Object` y `Object[]` el más específico es `Object[]`, porque todo `Object[]` es un `Object`. Añadir el varargs cambió el destino de una llamada que ya existía.

Dos consejos que salen de esto:

- **No sobrecargues un método con su propia versión varargs.** Si necesitás las dos, dales nombres distintos.
- **Si una API pública tiene un varargs, añadir sobrecargas después es un cambio de comportamiento**, no una mejora aditiva.

## 42. Contaminación del heap y SafeVarargs

Los genéricos y los arrays no se llevan bien: los genéricos se borran en compilación (*type erasure*) y los arrays conservan su tipo en ejecución. Un varargs genérico está justo en la intersección, y por eso el compilador avisa. Verificado:

```
HeapPollution.java:5: warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
    static String primeroDelPrimero(List<String>... listas) {
HeapPollution.java:14: warning: [unchecked] unchecked generic array creation for varargs parameter of type List<String>[]
```

Y esto es lo que puede pasar, con el ejemplo de Baeldung reproducido:

```java
static String primeroDelPrimero(List<String>... listas) {
    List<Integer> enteros = Collections.singletonList(42);
    Object[] objetos = listas;
    objetos[0] = enteros;          // contaminación del heap
    return listas[0].get(0);       // ClassCastException
}
```

```
ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String
```

Un `ClassCastException` sin un solo cast a la vista. El truco es que `List<String>[]` se puede asignar a `Object[]`, y a través de esa referencia se puede meter cualquier cosa; el array no puede comprobarlo porque su tipo en ejecución es `List[]`, sin parámetro.

La anotación **`@SafeVarargs`** silencia el aviso, y es una **promesa del autor**, no una comprobación. Solo se puede poner en métodos que no puedan ser sobrescritos (`static`, `final`, `private` o constructores), y solo es honesta si se cumplen dos condiciones:

1. El método **no guarda nada** en el array de varargs.
2. El método **no deja escapar** una referencia a ese array fuera de sí mismo.

Baeldung documenta las dos, y el segundo caso —el que se escapa— es el menos obvio:

```java
static <T> T[] toArray(T... arguments) { return arguments; }   // inseguro
```

Parece inofensivo y no lo es: devuelve el array que el compilador creó, cuyo tipo en ejecución puede no ser el que el llamante espera, y el `ClassCastException` sale en un sitio completamente distinto.

El caso legítimo es el de transportar argumentos y nada más:

```java
@SafeVarargs
static <T> List<T> lista(T... elementos) { return List.of(elementos); }
```

```
lista con @SafeVarargs: [a, b]
```

Es la misma condición que cumple `List.of` en la JDK. El resumen de Baeldung es exacto: el uso de varargs es seguro si sirve solo para transferir un número variable de argumentos del llamante al método, y nada más.

Un detalle de su artículo que conviene corregir al pasar: dice que los varargs *«thanks to their implicit autoboxing to and from Array»* ayudan a preparar el código para el futuro. **Autoboxing** es el nombre de la conversión entre primitivos y sus wrappers (`int` ↔ `Integer`); lo que hace el compilador con un varargs es **empaquetar los argumentos en un array**, que es otra transformación distinta y no tiene ese nombre. Es un desliz terminológico, no un error de comportamiento, pero mezclar los dos conceptos es justo lo que produce la confusión de la [sección 40](#40-varargs-y-arrays-de-primitivos).

---

# Parte VI — Recursión

## 43. Qué es la recursión y cuándo compensa

Un método **recursivo** es el que se llama a sí mismo. Toda recursión bien formada tiene dos partes:

1. Un **caso base**, que devuelve sin volver a llamarse.
2. Un **paso recursivo**, que se llama a sí mismo con un problema estrictamente más pequeño.

```java
static int sum(int k) {
    if (k > 0) {
        return k + sum(k - 1);   // paso recursivo
    } else {
        return 0;                // caso base
    }
}
```

Es el ejemplo de W3Schools, y su explicación —que llama *halting condition* al caso base y advierte de que sin ella la recursión es infinita, igual que un bucle— es correcta y está bien contada.

Lo que ninguna de las fuentes dice, y es lo que decide si usarla o no, es **cuándo compensa**. La regla no es de gusto:

**Usá recursión cuando la estructura de datos es recursiva.** Un árbol, un sistema de ficheros, un JSON anidado, una expresión con paréntesis. Ahí el código recursivo es más corto *y* más claro que el iterativo, porque la forma del código copia la forma del dato:

```java
static long tamanoTotal(File carpeta) {
    if (carpeta.isFile()) return carpeta.length();
    long total = 0;
    File[] hijos = carpeta.listFiles();
    if (hijos != null) {
        for (File hijo : hijos) total += tamanoTotal(hijo);   // recursión natural
    }
    return total;
}
```

**No uses recursión cuando la estructura es lineal.** `sum(k)`, `factorial(n)` o recorrer una lista son bucles disfrazados: la versión iterativa es igual de legible, no consume pila y no tiene un límite de profundidad.

```java
static int sumaIterativa(int k) {
    int total = 0;
    for (int i = 1; i <= k; i++) total += i;
    return total;
}
```

Los ejemplos de los tutoriales —factorial, cuenta atrás, sumar un rango— son didácticos precisamente porque son casos donde **no** se usaría recursión en código real. Conviene saberlo mientras se aprenden.

## 44. La pila y el StackOverflowError

Cada llamada crea un marco de pila ([sección 7](#7-qué-hace-la-jvm-cuando-llamás-a-un-método)). La pila de cada hilo tiene un tamaño fijo. Cuando la recursión es demasiado profunda, se agota:

```java
static void hundir() { profundidad++; hundir(); }
```

```
StackOverflowError a los 64459 marcos
```

Y aquí está el dato que casi nadie mide: **ese número no es una constante del lenguaje**. Ejecutando dos métodos distintos con tres tamaños de pila:

| Configuración | Método ligero (sin parámetros) | Método "pesado" (5 parámetros `long` + 5 locales) |
|---|---|---|
| `-Xss512k` | 8.497 marcos | 1.573 marcos |
| Por defecto | 47.224 marcos | 14.850 marcos |
| `-Xss8m` | 204.646 marcos | 71.531 marcos |

Todo medido en JDK 25, en la misma máquina, en la misma sesión. Tres conclusiones:

1. **La profundidad depende del tamaño del marco.** Más parámetros y más locales, menos llamadas caben. El método con cinco `long` y cinco locales aguanta menos de un tercio.
2. **Depende de `-Xss`.** Es una opción de la JVM, no del lenguaje. El valor por defecto cambia entre plataformas y versiones.
3. **No es reproducible entre ejecuciones.** La primera medición de este capítulo, con un método casi idéntico, dio 64.459 marcos, y la de la tabla dio 47.224. Cualquier código que dependa de una profundidad concreta está roto por definición.

Además, `StackOverflowError` es un **`Error`**, no una `Exception`. Se puede capturar —arriba se captura para poder contar—, pero no se debe: cuando ocurre, el estado del programa es dudoso y lo correcto es dejar morir el hilo.

Un `StackOverflowError` en producción significa casi siempre una de estas tres cosas, y ninguna se arregla subiendo `-Xss`:

- **Un caso base que no se alcanza**: la condición está mal o el paso recursivo no reduce el problema.
- **Una recursión mutua accidental**: `equals` que llama a `hashCode` que llama a `equals`, o un `toString` que imprime un objeto que lo contiene.
- **Una recursión legítima sobre datos más profundos de lo previsto**: ahí sí toca reescribirla de forma iterativa, con una pila explícita.

## 45. Java no elimina las llamadas de cola

Una **llamada de cola** (*tail call*) es aquella en la que la llamada recursiva es lo último que ocurre en el método: no queda nada pendiente al volver.

```java
static long sumaCola(long n, long acumulado) {
    if (n == 0) return acumulado;
    return sumaCola(n - 1, acumulado + n);   // llamada de cola
}
```

En Scheme, Scala (con `@tailrec`) o Kotlin (con `tailrec`), el compilador convierte esto en un bucle y la pila no crece. **En Java, no.** Verificado con tres tamaños de pila distintos:

```
sumaCola(1_000_000) -> StackOverflowError (Java no elimina la llamada de cola)
```

Falla con `-Xss512k`, con el valor por defecto y con `-Xss8m`. Java nunca ha implementado la eliminación de llamadas de cola; las razones históricas tienen que ver con el modelo de seguridad basado en inspección de la pila y con la calidad de los stacktraces, y sigue apareciendo periódicamente en la lista de deseos del proyecto Loom.

La consecuencia práctica: **reescribir una recursión "en forma de cola" no arregla nada en Java**. Si el problema es la profundidad, la solución es un bucle o una pila explícita:

```java
static long sumaIterativa(long n) {
    long total = 0;
    for (long i = n; i > 0; i--) total += i;
    return total;
}
```

## 46. El coste exponencial y la memoización

El otro modo de fallo de la recursión no es la pila, es el tiempo. El caso canónico:

```java
static long fibIngenuo(int n) {
    return n < 2 ? n : fibIngenuo(n - 1) + fibIngenuo(n - 2);
}
```

```
fibIngenuo(40) = 102334155 en 238 ms
```

Los 238 ms no son la medida interesante —eso depende de la máquina y del JIT—; lo interesante es **por qué**: el árbol de llamadas recalcula los mismos valores una y otra vez. `fib(38)` se calcula dos veces, `fib(37)` tres, `fib(36)` cinco… El número de llamadas crece como la propia sucesión, es decir de forma exponencial. Con `n = 50` este método no termina en un tiempo razonable; el problema no es Java.

La **memoización** guarda lo ya calculado:

```java
static long fibMemo(int n, long[] cache) {
    if (n < 2) return n;
    if (cache[n] != 0) return cache[n];
    return cache[n] = fibMemo(n - 1, cache) + fibMemo(n - 2, cache);
}
```

Ahora cada `n` se calcula una sola vez: de exponencial a lineal, con un array de `n` posiciones. Y la versión iterativa no necesita ni el array ni la pila:

```java
static long fibIterativo(int n) {
    long a = 0, b = 1;
    for (int i = 0; i < n; i++) { long t = a + b; a = b; b = t; }
    return a;
}
```

La lección general: **la recursión no es lenta; recalcular sí**. Cuando un método recursivo se llama con los mismos argumentos por distintas ramas, hay que cachear o cambiar de enfoque.

## 47. El desbordamiento silencioso del factorial

El ejemplo de factorial de W3Schools, tal cual está publicado:

```java
static int factorial(int n) {
    if (n > 1) {
        return n * factorial(n - 1);
    } else {
        return 1;
    }
}
```

Ejecutado en JDK 25:

```
factorial(12) = 479001600
factorial(13) = 1932053504
factorial(17) = -288522240
factorial(20) = -2102132736
13! real = 6227020800
```

**A partir de 13 el resultado es incorrecto, y a partir de 17 es negativo.** El método no falla, no lanza nada, no avisa: devuelve un `int` que ha dado la vuelta, exactamente como los contadores del capítulo de [Loops](11-loops.md). `13!` vale 6.227.020.800, que no cabe en un `int` (máximo 2.147.483.647).

La fuente no menciona el límite. Su aviso —«be careful with recursion: it's easy to accidentally write a method that never stops or uses too much memory»— habla de recursión infinita y de memoria, que son los dos riesgos que sí menciona, pero el bug que este ejemplo concreto tiene es aritmético y aparece en el decimotercer valor.

Las tres defensas, por orden de coste:

```java
static long factorialLong(int n) { ... }              // llega hasta 20!; 21! ya desborda
static BigInteger factorialGrande(int n) { ... }      // sin límite práctico
static int factorialSeguro(int n) {
    return n > 1 ? Math.multiplyExact(n, factorialSeguro(n - 1)) : 1;   // lanza ArithmeticException
}
```

`Math.multiplyExact` es la que conviene conocer: hace la misma multiplicación pero lanza `ArithmeticException: integer overflow` en vez de dar la vuelta en silencio. Para un ejemplo de tutorial, convertir un resultado incorrecto en una excepción sería una mejora enorme por dos palabras de código.

---

# Parte VII — Los límites de la plataforma

## 48. Doscientos cincuenta y cinco parámetros

Programiz afirma, en su página de métodos:

> *«parameter1/parameter2 - These are values passed to a method. We can pass any number of arguments to a method.»*

**No se puede.** El límite existe, está en la especificación de la JVM y el compilador lo aplica. Verificado generando dos clases:

```
== 255 ==
compila con 255 parametros
== 256 ==
Params256.java:2: error: too many parameters
1 error
```

255 compilan; 256 no. El motivo está en el formato del `.class`, y la [JVMS §4.11](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-4.html#jvms-4.11) lo dice sin rodeos:

> *«The number of method parameters is limited to 255 by the definition of a method descriptor (§4.3.3), where the limit includes one unit for this in the case of instance or interface method invocations.»*

Hay un matiz que se mide fácil y casi nunca se cuenta: **en un método de instancia el límite efectivo es 254**, porque `this` ocupa una de las 255 posiciones. Verificado con la misma clase de 255 parámetros, cambiando solo `static`:

```
== Instancia255 (this cuenta) ==
error: too many parameters
```

El mismo número de parámetros compila como estático y no compila como método de instancia. Y una precisión más: la cuenta es de **palabras**, no de parámetros; un `long` o un `double` ocupan **dos**. Un método estático con 128 parámetros `double` tampoco compila.

Nada de esto importa en el día a día por una razón evidente —un método con 255 parámetros es inmantenible mucho antes de ser ilegal—, pero sí importa en **código generado**: parsers, mapeadores, DTOs enormes y constructores de entidades con decenas de columnas han topado con este límite en proyectos reales. El límite razonable de diseño está en la [sección 51](#51-cuántos-parámetros-son-demasiados), y es unas cincuenta veces más bajo.

## 49. Sesenta y cuatro kilobytes por método

El segundo techo duro: **el cuerpo de un método no puede pasar de 65.535 bytes de bytecode**. Verificado generando un método con 20.000 sentencias:

```
== Enorme (64 KB de bytecode) ==
error: code too large
```

El límite no está en líneas de fuente sino en bytecode, así que no hay una equivalencia exacta; como referencia, un método con veinte mil sumas simples ya lo supera.

¿Quién se topa con esto en la práctica?

- **Código generado**: parsers de ANTLR, clases de constantes generadas por herramientas, mapeos gigantes.
- **Inicializadores estáticos enormes**: un `static { }` que rellena un array de decenas de miles de elementos entra en el mismo límite, porque el compilador lo mete en el método `<clinit>`.
- **Métodos escritos a mano de miles de líneas**: existen, y este error es la forma que tiene la plataforma de decir basta.

## 50. Métodos pequeños y el JIT

Existe la creencia de que partir un método en varios "cuesta llamadas" y hace el programa más lento. En Java es casi siempre al revés, y por un motivo concreto: **el JIT hace *inlining***, es decir, copia el cuerpo de los métodos pequeños en el sitio de la llamada, y a partir de ahí puede optimizar el conjunto.

El detalle importante es que el JIT decide por **tamaño del bytecode**, no por lo bonito que sea el código. Los umbrales por defecto de esta JVM, leídos directamente:

```
intx MaxInlineSize   = 35     {C2 product}
intx FreqInlineSize  = 325    {C2 pd product}
intx MaxInlineLevel  = 15     {C2 product}
intx InlineSmallCode = 2500   {C2 pd product}
```

Traducido: un método de hasta **35 bytes** de bytecode es candidato a inlining siempre; uno de hasta **325 bytes** lo es si se llama con frecuencia; y el compilador no anida más de **15 niveles** de inlining.

De ahí sale una conclusión que va en contra de la intuición: **un método grande es más caro que varios pequeños**, porque el grande no entra en el presupuesto de inlining y bloquea las optimizaciones aguas arriba. Partir en métodos pequeños y bien nombrados no es solo estilo; también ayuda al compilador.

Dos advertencias, en la misma línea que el capítulo de [Loops](11-loops.md):

- **Estos números son de esta JVM y de esta versión.** Son argumentos, no promesas: cambian entre versiones y entre C1 y C2.
- **En este capítulo no hay comparativas de tiempo entre construcciones, a propósito.** Medir el coste de una llamada con un microbenchmark casero es aún menos fiable que medir un bucle, porque lo primero que hace el JIT es eliminar la llamada. Para medir de verdad hace falta [JMH](https://openjdk.org/projects/code-tools/jmh/).

---

# Parte VIII — Diseño de la firma

Hasta aquí, cómo funcionan los métodos. A partir de aquí, cómo se escriben para que sigan siendo legibles dentro de dos años. Esta parte no tiene errores de compilación que enseñar: tiene decisiones.

## 51. Cuántos parámetros son demasiados

La plataforma permite 255 ([sección 48](#48-doscientos-cincuenta-y-cinco-parámetros)). La práctica tiene otros números:

| Parámetros | Veredicto |
|---|---|
| 0–2 | Cómodo. La mayoría de los métodos deberían estar aquí. |
| 3 | Aceptable si los tipos son distintos entre sí. |
| 4–5 | Señal de alarma. Suele haber un objeto escondido. |
| 6+ | Rehacer la firma. |

El problema de una lista larga no es escribirla: es **leer la llamada**.

```java
crearUsuario("ana", "López", 30, true, false, true, "ES");
```

Nadie puede saber qué significa el tercer `true` sin abrir la declaración. Y como los tipos se repiten, invertir dos argumentos compila sin un aviso ([sección 3](#3-parámetros-y-argumentos)).

Las tres salidas, en orden de preferencia:

**1. Agrupar en un objeto (`record`, desde Java 16).** Cuando varios parámetros viajan siempre juntos —un *data clump*—, ese grupo es un concepto que pide nombre:

```java
record Direccion(String calle, String ciudad, String pais, String codigoPostal) { }

void enviar(Pedido pedido, Direccion destino) { }    // 2 parámetros en vez de 5
```

**2. Un builder**, cuando hay muchos campos opcionales:

```java
Usuario u = Usuario.builder()
        .nombre("ana")
        .apellido("López")
        .edad(30)
        .activo(true)
        .build();
```

Cada valor va acompañado de su nombre en el sitio de la llamada, que es justo lo que Java no ofrece de serie.

**3. Sobrecargas con valores por defecto**, para el caso simple:

```java
void enviar(Pedido p, Direccion d)                { enviar(p, d, Prioridad.NORMAL); }
void enviar(Pedido p, Direccion d, Prioridad pr)  { ... }
```

Java no tiene parámetros con nombre ni valores por defecto —a diferencia de Kotlin, Python o C#—, y esta es la forma canónica de emularlos. Con la advertencia de toda la Parte IV: cada sobrecarga añadida es una decisión que el compilador tomará por vos.

## 52. Parámetros booleanos y el orden

Un `boolean` en una llamada no dice nada:

```java
generarInforme(datos, true, false);
```

Es el llamado *flag argument*, y suele indicar además que el método hace **dos cosas distintas** según la bandera. Tres alternativas, de mejor a peor:

```java
// 1. Dos métodos con nombres que explican la diferencia
generarInformeDetallado(datos);
generarInformeResumido(datos);

// 2. Un enum, que se lee en el sitio de la llamada
generarInforme(datos, Formato.DETALLADO, Destino.PANTALLA);

// 3. Si el boolean se queda, que el nombre lo salve
generarInforme(datos, /* incluirDetalle */ true, /* imprimir */ false);
```

La segunda tiene una ventaja añadida: un enum admite un tercer valor mañana; un `boolean`, no.

**El orden importa igual que los tipos.** Dos criterios que funcionan:

- **De más importante a más accesorio**: primero el objeto sobre el que se actúa, después los modificadores. Es lo que hace la JDK: `String.join(separador, partes)`, `Collections.sort(lista, comparador)`.
- **Coherencia dentro de la misma API**: si `copiar(origen, destino)` es el orden en un sitio, no puede ser `copiar(destino, origen)` en otro. Este error concreto es el que hace que `System.arraycopy(src, srcPos, dest, destPos, length)` haya que mirarla siempre.

Y cuando dos parámetros del mismo tipo son intercambiables por accidente, el tipo puede impedirlo:

```java
void transferir(long origen, long destino, double importe);            // fácil de invertir
void transferir(Cuenta origen, Cuenta destino, Importe importe);       // sigue siendo posible
void transferir(Transferencia t);                                       // imposible de invertir
```

## 53. Nombres

El nombre de un método es su documentación principal. Las convenciones de Java, que la biblioteca estándar respeta con notable disciplina:

| Tipo de método | Convención | Ejemplos de la JDK |
|---|---|---|
| Acción | verbo en imperativo | `add`, `remove`, `sort`, `close` |
| Consulta que devuelve un valor | sustantivo o `getX` | `size`, `length`, `getName` |
| Consulta que devuelve `boolean` | `is`, `has`, `can`, `contains` | `isEmpty`, `hasNext`, `containsKey` |
| Conversión | `toX` / `asX` / `valueOf` | `toString`, `asList`, `valueOf` |
| Factoría estática | `of`, `from`, `parse`, `create` | `List.of`, `LocalDate.parse` |

Hay un matiz entre `toX` y `asX` que la JDK respeta y casi nadie conoce: **`toX` crea algo nuevo, `asX` devuelve una vista** sobre lo mismo. `Arrays.asList` devuelve una vista respaldada por el array —modificarla modifica el array, y por eso no admite `add`—, mientras que `stream().toList()` construye una lista nueva. El nombre lo estaba diciendo.

Tres reglas más, todas con consecuencias prácticas:

- **El nombre debe decir el qué, no el cómo.** `ordenarConQuickSort` obliga a renombrar el día que cambies de algoritmo; `ordenar`, no.
- **Si el nombre necesita un "y", hay dos métodos.** `validarYGuardar` está confesando que hace dos cosas.
- **El nombre y el tipo de retorno tienen que estar de acuerdo.** Un método llamado `getX` que devuelve `void` o que lanza excepciones a la primera de cambio miente. Un `isValid` que además modifica el objeto, también.

## 54. Validar en la frontera

Un método público es una **frontera**: recibe datos que no controla. Un método privado está detrás de esa frontera y puede confiar en lo que ya se validó. La distinción evita el patrón de validar diez veces lo mismo.

```java
public void registrar(String email, int edad) {
    Objects.requireNonNull(email, "email no puede ser null");
    if (edad < 0 || edad > 150) {
        throw new IllegalArgumentException("edad fuera de rango: " + edad);
    }
    guardar(email.strip().toLowerCase(), edad);   // a partir de aquí, ya está validado
}

private void guardar(String email, int edad) { ... }   // no revalida
```

`Objects.requireNonNull` es la forma estándar y merece un apunte: **no es solo un `if` más corto**. Lanza en el momento y el sitio correctos, con el nombre del parámetro en el mensaje, en vez de dejar que el `null` viaje tres capas hacia dentro y explote donde nadie entienda por qué. Verificado:

```
requireNonNull: nombre no puede ser null
```

Las excepciones estándar para cada caso, que conviene usar en vez de inventar:

| Situación | Excepción |
|---|---|
| Argumento `null` no permitido | `NullPointerException` (vía `requireNonNull`) |
| Valor fuera de rango o con formato inválido | `IllegalArgumentException` |
| Índice fuera de límites | `IndexOutOfBoundsException` |
| El objeto no está en un estado válido para esa llamada | `IllegalStateException` |

Y el contrato hay que **escribirlo**, no solo comprobarlo:

```java
/**
 * Registra un usuario nuevo.
 *
 * @param email  dirección válida, no nula; se normaliza a minúsculas
 * @param edad   entre 0 y 150 inclusive
 * @throws IllegalArgumentException si la edad está fuera de rango
 */
```

El Javadoc de los parámetros es el único sitio donde caben las precondiciones que la firma no puede expresar. La firma dice `int edad`; solo el Javadoc puede decir que 200 no vale.

## 55. Métodos puros y efectos

Un método **puro** cumple dos cosas: devuelve siempre lo mismo para los mismos argumentos, y no cambia nada fuera de sí mismo.

```java
static double conIva(double precio) { return precio * 1.21; }        // puro
static void guardar(Pedido p)       { repositorio.insertar(p); }     // efecto
static long ahora()                 { return System.nanoTime(); }    // ni puro ni con efecto: impredecible
```

Los métodos puros son más fáciles de probar (no hacen falta mocks), de razonar (no hay que saber en qué orden se llaman), de cachear y de paralelizar. No todo puede serlo —un programa que no cambia nada no sirve— pero **la mezcla sí se puede evitar**:

```java
// mezcla cálculo y efecto: para probar el cálculo hay que simular el correo
void procesarPedido(Pedido p) {
    double total = p.lineas().stream().mapToDouble(Linea::importe).sum() * 1.21;
    repositorio.guardar(p, total);
    correo.enviarConfirmacion(p.cliente(), total);
}

// separados: el cálculo se prueba solo, con una línea
double calcularTotal(Pedido p) { ... }          // puro

void procesarPedido(Pedido p) {                 // solo orquesta
    double total = calcularTotal(p);
    repositorio.guardar(p, total);
    correo.enviarConfirmacion(p.cliente(), total);
}
```

Dos señales de que un método está mezclando:

- **Devuelve un valor y además cambia algo.** Con excepciones justificadas: `List.remove` devuelve `boolean` y muta, y está bien porque la mutación *es* el propósito.
- **Muta uno de sus parámetros.** Ya vimos que se puede ([sección 12](#12-arrays-colecciones-y-stringbuilder)); el problema es que la firma no lo dice. Entre `void ordenar(List<T> lista)` y `List<T> ordenada(List<T> lista)`, la segunda no engaña a nadie. Si mutás un parámetro, que el nombre lo grite y que el Javadoc lo repita.

## 56. Un método, un nivel de abstracción

El criterio más útil para decidir si un método es demasiado largo no es contar líneas: es mirar si **mezcla niveles**.

```java
void procesarFichero(Path ruta) {
    List<String> lineas = Files.readAllLines(ruta);       // alto nivel
    for (String linea : lineas) {
        int i = 0;
        while (i < linea.length() && linea.charAt(i) == ' ') i++;   // bajo nivel
        String limpia = linea.substring(i);
        if (!limpia.isEmpty() && limpia.charAt(0) != '#') {
            String[] partes = limpia.split("=", 2);
            // ...
        }
    }
}
```

El método salta de «leer un fichero» a «saltar espacios carácter a carácter» en cuatro líneas. La reescritura no cambia el número total de líneas, cambia dónde están:

```java
void procesarFichero(Path ruta) throws IOException {
    for (String linea : Files.readAllLines(ruta)) {
        if (esComentario(linea)) continue;
        registrar(parsearEntrada(linea));
    }
}
```

Ahora se lee como una frase, y cada detalle está en un método con nombre, un nivel más abajo. Es el mismo principio que en el capítulo de [Loops](11-loops.md) llevaba a extraer el cuerpo del bucle.

Las guías numéricas que se usan en la práctica, con la advertencia de que son señales y no leyes:

- **Menos de 50 líneas** por método; la mayoría deberían estar bastante por debajo.
- **Que quepa en una pantalla** sin desplazarse.
- **Un solo nivel de anidamiento** dentro del cuerpo, dos como máximo.
- **Un solo motivo para cambiarlo.**

Y el contrapunto honesto: **extraer métodos también tiene coste**. Veinte métodos privados de dos líneas cada uno, con nombres que no aportan nada, obligan a saltar veinte veces para entender un flujo. Si la única forma de nombrar el método extraído es repetir lo que hace su única línea, no había nada que extraer.

---

# Parte IX — Cierre

## 57. Casos de uso reales

### 57.1 Un validador puro y sus llamadas

```java
public final class Validaciones {

    private Validaciones() { }   // clase de utilidades: nadie debería instanciarla

    public static boolean esEmailValido(String email) {
        return email != null && email.matches("^[^@\\s]+@[^@\\s]+\\.[^@\\s]{2,}$");
    }

    public static String requerirNoVacio(String valor, String nombreCampo) {
        if (valor == null || valor.isBlank()) {
            throw new IllegalArgumentException(nombreCampo + " es obligatorio");
        }
        return valor.strip();
    }
}
```

Dos detalles de diseño: el constructor privado impide instanciar una clase que solo tiene estáticos, y `requerirNoVacio` **devuelve el valor normalizado** en vez de ser `void`, lo que permite encadenar `this.nombre = requerirNoVacio(nombre, "nombre")`. Es el patrón de `Objects.requireNonNull`.

### 57.2 Reintentos con un método que recibe comportamiento

```java
static <T> T conReintentos(int intentos, Supplier<T> operacion) {
    RuntimeException ultima = null;
    for (int i = 1; i <= intentos; i++) {
        try {
            return operacion.get();
        } catch (RuntimeException e) {
            ultima = e;
            dormir(100L * i);
        }
    }
    throw new IllegalStateException("falló tras " + intentos + " intentos", ultima);
}

// uso
Respuesta r = conReintentos(3, () -> cliente.consultar(id));
```

Un parámetro puede ser **código**, no solo datos. Es la puerta de entrada al bloque de programación funcional, y aquí ya se ve la ganancia: la política de reintentos se escribe una vez para cualquier operación.

### 57.3 Sobrecargas como valores por defecto

```java
public String formatear(BigDecimal importe) {
    return formatear(importe, Locale.getDefault(), 2);
}

public String formatear(BigDecimal importe, Locale locale) {
    return formatear(importe, locale, 2);
}

public String formatear(BigDecimal importe, Locale locale, int decimales) {
    // única implementación real
}
```

El patrón correcto: **una sola implementación** y las demás delegando. Si cada sobrecarga tuviera su propio cuerpo, cambiar la lógica exigiría acordarse de las tres.

### 57.4 Varargs para una API cómoda

```java
public static Filtro todos(Filtro... filtros) {
    List<Filtro> copia = List.of(filtros);      // copia inmutable: el array no escapa
    return dato -> copia.stream().allMatch(f -> f.acepta(dato));
}

Filtro f = todos(porFecha(hoy), porEstado(ACTIVO), porImporte(100));
```

La primera línea del cuerpo es la disciplina que enseña la [sección 42](#42-contaminación-del-heap-y-safevarargs): copiar el array de varargs y no dejar que la referencia se escape.

### 57.5 Recorrer un árbol de directorios

```java
static long tamanoTotal(Path ruta) throws IOException {
    if (Files.isRegularFile(ruta)) return Files.size(ruta);
    try (Stream<Path> hijos = Files.list(ruta)) {
        long total = 0;
        for (Path hijo : (Iterable<Path>) hijos::iterator) {
            total += tamanoTotal(hijo);
        }
        return total;
    }
}
```

El caso donde la recursión es la herramienta correcta: la estructura es recursiva y la profundidad está acotada por la del sistema de ficheros.

## 58. Anti-patrones

**1. Creer que los objetos se pasan por referencia.**

```java
static void intercambiar(Foo a, Foo b) { Foo t = a; a = b; b = t; }   // MAL: no hace nada
```

**2. Reasignar un parámetro y esperar que se vea fuera.**

```java
static void f(List<String> l) { l = new ArrayList<>(); }   // MAL: no cambia nada fuera
static void f(List<String> l) { l.clear(); }               // esto sí muta
```

**3. `return`, `break` o `continue` dentro de un `finally`.**

```java
try { ... } finally { return 0; }   // MAL: descarta la excepción en curso
```

**4. Añadir un `return` de relleno para callar al compilador.**

```java
return 0;   // MAL si 0 no significa nada; documentar el centinela o lanzar
```

**5. Devolver `null` en vez de una colección vacía.**

```java
return null;        // MAL: obliga a un if antes de cada bucle
return List.of();   // bien
```

**6. Ignorar el valor devuelto por un método de un tipo inmutable.**

```java
texto.trim();          // MAL: no hace nada
texto = texto.trim();  // bien
```

**7. Devolver la colección o el array interno sin copiar.**

```java
List<String> getLineas() { return lineas; }              // MAL: escritura desde fuera
List<String> getLineas() { return List.copyOf(lineas); } // bien
```

**8. Sobrecargar métodos que hacen cosas distintas.**

```java
void procesar(Pedido p);       // MAL si no tienen nada que ver
void procesar(Devolucion d);
```

**9. Sobrecargar con tipos convertibles entre sí.**

```java
void f(int x); void f(long x); void f(Integer x); void f(Object x);   // MAL: nadie sabe cuál sale
```

**10. Sobrecargar un método con su propia versión varargs.**

```java
void m(Object o); void m(Object... o);   // MAL: cambia el destino de m(null)
```

**11. Pasar un array de primitivos a un varargs genérico.**

```java
Arrays.asList(arrayDeInt);   // MAL: lista de 1 elemento
```

**12. Pasar `null` sin cast a un método varargs.**

```java
recibir(null);              // MAL: aviso del compilador y NPE en ejecución
recibir((String) null);     // explícito
```

**13. `@SafeVarargs` en un método que guarda en el array o lo deja escapar.**

```java
@SafeVarargs static <T> T[] toArray(T... a) { return a; }   // MAL: la promesa es falsa
```

**14. Recursión sobre una estructura lineal.**

```java
static int sum(int k) { return k > 0 ? k + sum(k - 1) : 0; }   // MAL: es un bucle con pila
```

**15. Recursión aritmética con `int` sin controlar el desbordamiento.**

```java
static int factorial(int n) { ... }   // MAL a partir de 13: resultado incorrecto en silencio
```

**16. Reescribir en forma de llamada de cola creyendo que Java la optimiza.** No lo hace.

**17. Un método con seis o más parámetros.**

```java
crearUsuario(a, b, c, d, e, f, g);   // MAL: agrupá en un record o usá un builder
```

**18. Parámetros booleanos en la llamada.**

```java
generarInforme(datos, true, false);   // MAL: usá enums o métodos con nombre
```

**19. Un parámetro con el mismo nombre que un campo, fuera de un constructor o un setter.**

```java
nombre = nombre;   // MAL: se asigna a sí mismo, el campo no cambia
```

**20. Mezclar cálculo y efecto en el mismo método.** Separar el cálculo puro permite probarlo sin simular nada.

**21. Llamar a un método estático a través de una instancia.**

```java
objeto.metodoEstatico();    // MAL: legal, confuso, y funciona incluso si objeto es null
Clase.metodoEstatico();     // bien
```

**22. Depender del orden de evaluación de los argumentos.**

```java
procesar(cola.poll(), cola.poll());   // MAL: determinista, pero ilegible
```

## 59. Checklist y tabla de decisión

**Antes de dar por bueno un método:**

- [ ] ¿El nombre dice qué hace, con un verbo, sin un "y" dentro?
- [ ] ¿Tiene tres parámetros o menos?
- [ ] ¿Hay algún `boolean` en la lista que debería ser un enum?
- [ ] ¿Dos parámetros consecutivos del mismo tipo que se puedan invertir por error?
- [ ] ¿Se valida lo que entra, si el método es público?
- [ ] ¿Se muta algún parámetro? Si sí, ¿lo dicen el nombre y el Javadoc?
- [ ] ¿Se reasigna algún parámetro sin motivo?
- [ ] ¿Todos los caminos devuelven algo con sentido, sin `return` de relleno?
- [ ] Si devuelve una colección, ¿devuelve vacía en vez de `null`?
- [ ] Si devuelve una estructura interna, ¿está copiada o es de solo lectura?
- [ ] ¿Hay algún `return` dentro de un `finally`?
- [ ] Si está sobrecargado, ¿las sobrecargas hacen lo mismo con datos equivalentes?
- [ ] Si está sobrecargado, ¿alguna llamada con `null` sería ambigua?
- [ ] Si tiene varargs, ¿el array se copia y no se deja escapar?
- [ ] Si es recursivo, ¿el caso base es alcanzable para toda entrada posible?
- [ ] Si es recursivo, ¿la profundidad máxima está acotada por los datos?
- [ ] ¿Cabe en una pantalla y se mantiene en un solo nivel de abstracción?
- [ ] ¿Se puede separar el cálculo del efecto?

**Qué construcción usar:**

| Si necesitás… | Usá |
|---|---|
| Una operación que no depende de ningún objeto | método `static` |
| Una operación que depende del estado del objeto | método de instancia |
| Un número variable de argumentos del mismo tipo | varargs, y copiá el array dentro |
| Un número variable ya empaquetado | un parámetro `List` o array |
| Muchos parámetros que viajan juntos | un `record` |
| Muchos parámetros opcionales | un builder |
| Un valor por defecto | una sobrecarga que delegue en la implementación única |
| El mismo nombre para operaciones distintas | **no**: nombres distintos |
| Varias formas de construir algo | factorías estáticas con nombre (`of`, `from`, `parse`) |
| Indicar "no hay resultado" | `Optional` como retorno, nunca como parámetro |
| Devolver una colección sin resultados | `List.of()`, nunca `null` |
| Devolver una estructura interna | `List.copyOf` (copia) o `unmodifiableList` (vista) |
| Rechazar un argumento nulo | `Objects.requireNonNull(x, "x")` |
| Rechazar un valor fuera de rango | `IllegalArgumentException` |
| Recorrer una estructura recursiva | recursión |
| Repetir sobre una estructura lineal | un bucle |
| Recursión con recálculo de subproblemas | memoización o versión iterativa |
| Aritmética que puede desbordar | `Math.multiplyExact`, `addExact`, o `BigInteger` |
| Pasar comportamiento, no datos | un parámetro funcional (`Supplier`, `Function`, `Predicate`) |
| Que los nombres de los parámetros sobrevivan | compilar con `-parameters` |

## 60. Autoevaluación

**1. ¿Java pasa los argumentos por valor o por referencia? ¿Cuál es la prueba?**

<details><summary>Respuesta</summary>

Siempre **por valor**, sin excepciones. Lo que se copia es el contenido de la variable; si la variable contiene una referencia, se copia la referencia, nunca el objeto. La prueba decisiva es que **no se puede escribir un método de intercambio**: verificado en JDK 25, un `intercambiar(Foo a, Foo b)` que hace `a = b; b = t;` deja `p=Foo(7) q=Foo(8)` intactos. Si el paso fuera por referencia, funcionaría. La regla mecánica para cualquier caso dudoso: si el método usa `=` sobre el parámetro, no se ve fuera; si usa `.` para llegar al objeto, sí.
</details>

**2. Baeldung concluye su artículo diciendo que «for Object types, the object reference is pass-by-reference». ¿Es cierto?**

<details><summary>Respuesta</summary>

No, y el propio artículo lo desmiente. Su ejemplo hace `b1 = new Foo(1)` dentro del método y comprueba que `b.num` sigue valiendo 1 al volver; si la referencia se pasara por referencia, esa aserción fallaría. Reproducido en JDK 25: `a=Foo(2) b=Foo(1)`. Lo que la frase quiere decir es que **el efecto de mutar el objeto se ve desde fuera**, que es verdad y es otra cosa. La contradicción está dentro del mismo artículo, que empieza afirmando —correctamente— que en Java *«everything is strictly Pass-by-Value»*.
</details>

**3. ¿Qué diferencia hay entre `final` en un parámetro y que el objeto sea inmutable?**

<details><summary>Respuesta</summary>

`final` solo impide **reasignar la variable**. Con `final List<String> lista`, `lista.add("x")` es legal y `lista = new ArrayList<>()` no lo es. La inmutabilidad es una propiedad del objeto, no de la variable que lo señala. Jenkov lo formula bien: si el parámetro es una referencia, la referencia no puede cambiar pero los valores dentro del objeto sí. Desde Java 8, además, no hace falta escribir `final` para el beneficio principal: una variable que de hecho no se reasigna es *effectively final* y puede capturarse en una lambda. Reasignar un parámetro puede romper una lambda escrita tres líneas más abajo.
</details>

**4. ¿Qué forma parte de la firma de un método y qué no?**

<details><summary>Respuesta</summary>

Según el [JLS §8.4.2](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.2), la firma es el **nombre**, los **parámetros de tipo** si es genérico y los **tipos de los parámetros formales**. No forman parte: el tipo de retorno, los nombres de los parámetros, los modificadores ni la cláusula `throws`. Matiz que casi nadie menciona: el `.class` **sí** guarda el tipo de retorno, dentro del *descriptor*; el propio JLS lo define como «signature plus return type» y aclara que es el descriptor —no la firma— lo que la JVM usa para despachar en ejecución.
</details>

**5. Dos métodos que solo difieren en el tipo de retorno no compilan. ¿Cuál es el error exacto, y por qué no es «ambigüedad»?**

<details><summary>Respuesta</summary>

El error es `method print() is already defined in class`, verificado en JDK 25. No es ambigüedad por tres razones: es un error **de declaración**, no de invocación; se produce aunque **nadie llame** al método; y la causa es que ambos tienen la misma firma, así que para el compilador uno es un duplicado del otro y no hay dos candidatos entre los que elegir. La ambigüedad real tiene otro mensaje: `reference to f is ambiguous ... both method f(String) and method f(StringBuilder) match`, y aparece en la línea de la llamada. Baeldung explica este caso al revés en su artículo de sobrecarga, aunque en otro artículo suyo lo reporta bien.
</details>

**6. Programiz afirma que para sobrecargar «there must be differences in the number of parameters». ¿Es cierto?**

<details><summary>Respuesta</summary>

No. Basta con que difieran el **número, el tipo o el orden**. Verificado con `display(int)`, `display(String)` y `display(double)`: tres sobrecargas de un solo parámetro cada una, que compilan y funcionan. El propio artículo se contradice: dos pantallas antes dice «different number of parameters, different types of parameters, or both», y su segundo ejemplo sobrecarga por tipo. Lo que sí es cierto es la primera mitad de la frase: cambiar solo el tipo de retorno no sirve.
</details>

**7. ¿Cuáles son las tres fases de la resolución de sobrecarga y por qué existen?**

<details><summary>Respuesta</summary>

**Fase 1**, ensanchamiento y subtipos, sin boxing ni varargs. **Fase 2**, además boxing y unboxing, sin varargs. **Fase 3**, además varargs. El compilador para en la primera fase que encuentre algún candidato. Existen por **compatibilidad hacia atrás**: el JLS dice literalmente que es *«to ensure compatibility with the Java programming language prior to Java SE 5.0»*, la versión que introdujo a la vez el autoboxing y los varargs. Sin las fases, un programa de Java 1.4 podría haber cambiado de comportamiento al recompilarse. De ahí también la regla derivada: un varargs **nunca** se elige si hay una alternativa de aridad fija aplicable.
</details>

**8. Con `f(long)`, `f(Integer)` y `f(int...)` declarados, ¿a cuál va `f(1)`?**

<details><summary>Respuesta</summary>

A **`f(long)`**, verificado en JDK 25. El literal `1` es un `int`; no existe `f(int)`, pero el ensanchamiento `int` → `long` está permitido en la fase 1, así que ahí termina la búsqueda y las otras dos ni se consideran. La regla corta que sale de esto: **primitivo antes que wrapper, y wrapper antes que varargs**. Ensanchar es más barato que empaquetar porque no crea ningún objeto.
</details>

**9. Con `g(Object)` y `g(String)` declarados, ¿a cuál va `g(null)`? ¿Y si alguien añade `g(Integer)`?**

<details><summary>Respuesta</summary>

Va a **`g(String)`**: `null` es compatible con cualquier tipo referencia, ambos son aplicables, y entre los dos gana el **más específico**, que es `String` porque toda `String` es un `Object` y no al revés. Si alguien añade `g(Integer)`, `String` e `Integer` no tienen relación entre sí, ninguno es más específico, y la llamada **deja de compilar** por ambigüedad sin que nadie haya tocado esa línea. Es el motivo principal para no sobrecargar con tipos relacionados en una API pública.
</details>

**10. ¿Por qué `f(i)` no compila si `f` recibe un `Long` e `i` es un `int`?**

<details><summary>Respuesta</summary>

Porque **ensanchamiento y boxing no se combinan**. El error real es `incompatible types: int cannot be converted to Long`. En la fase 1 solo hay ensanchamiento, y `Long` no es un ensanchamiento de `int`; en la fase 2 hay boxing, pero empaquetar un `int` produce `Integer`, que no es subtipo de `Long`. Ninguna fase encadena las dos conversiones. Hay que escribir el paso que falta: `f((long) i)` o `f(Long.valueOf(i))`. Es la misma familia de sorpresas que `Map<Long, X>.get(1)`, que compila y nunca encuentra nada.
</details>

**11. Un cliente compilado sigue llamando al método viejo tras actualizar la biblioteca. ¿Por qué?**

<details><summary>Respuesta</summary>

Porque **la sobrecarga se resuelve en tiempo de compilación** y el descriptor del método elegido queda escrito en el `.class` del cliente. Verificado: un cliente compilado contra una biblioteca que solo tenía `saludar(Object)` sigue imprimiendo `saludar(Object)` después de sustituir el `.class` de la biblioteca por otro que añade `saludar(String)`; recompilarlo —sin cambiar una letra de su fuente— hace que pase a llamar a `saludar(String)`. El JLS lo documenta: *«The most specific method is chosen at compile time; its descriptor determines what method is actually executed at run time»*. Corolario: añadir una sobrecarga a una biblioteca es compatible binariamente pero **puede cambiar el comportamiento de quien recompile**.
</details>

**12. ¿Qué hace `lista.remove(1)` sobre un `List<Integer>`?**

<details><summary>Respuesta</summary>

Borra **la posición 1**, no el valor 1. Verificado: sobre `[10, 20, 30]` deja `[10, 30]`. `List` declara `remove(int index)` y `remove(Object o)`; el literal `1` es un `int` y encaja exactamente con la primera en la fase 1, así que la de `Object` ni se mira. Para borrar por valor hay que empaquetar a mano: `remove(Integer.valueOf(10))`. Es la trampa de sobrecarga más famosa de la biblioteca estándar, y existe porque `List` es anterior al autoboxing.
</details>

**13. ¿En qué se traduce exactamente un parámetro varargs?**

<details><summary>Respuesta</summary>

En un **array**, creado en el **sitio de la llamada**. El bytecode de `unir("-", "a", "b")` muestra `anewarray String`, dos `aastore` y luego el `invokestatic`; el descriptor del método es `(Ljava/lang/String;[Ljava/lang/String;)`. Lo único que distingue un varargs de un array en el `.class` es el flag `ACC_VARARGS`, que solo le sirve al compilador para construir el array por su cuenta. De ahí dos consecuencias: cada llamada asigna un array nuevo, y no se pueden declarar `f(String...)` y `f(String[])` a la vez — el error es `cannot declare both f(String[]) and f(String...)`.
</details>

**14. ¿Cuándo un parámetro varargs vale `null`?**

<details><summary>Respuesta</summary>

Solo si se lo pasás explícitamente. Sin argumentos, el método recibe un **array vacío**, nunca `null`: verificado, `length = 0`. Pero `recibir((String[]) null)` sí entrega `null`, y el primer `partes.length` lanza `NullPointerException`. El caso intermedio, `recibir((String) null)`, construye un array de un elemento que vale `null`. Y `recibir(null)` sin cast produce el aviso `non-varargs call of varargs method with inexact argument type for last parameter`, que conviene tratar siempre como error.
</details>

**15. ¿Qué devuelve `Arrays.asList(new int[]{1,2,3})` y por qué?**

<details><summary>Respuesta</summary>

Una lista de **un** elemento, cuyo único elemento es el `int[]` entero. Verificado: `tamanio = 1 -> int[]`. `asList` está declarado como `asList(T... a)` y `T` solo puede ser un tipo referencia; con un `int[]` el único `T` posible es `int[]` mismo, así que el compilador crea un `int[][]` de un elemento. Con `Integer[]` el resultado es el esperado, tamaño 3. Lo grave es que compila sin avisos. La solución es `Arrays.stream(v).boxed().toList()`. Regla mnemotécnica: **un `int[]` es un objeto; tres `int` no lo son**.
</details>

**16. ¿Qué es la contaminación del heap y qué promete exactamente `@SafeVarargs`?**

<details><summary>Respuesta</summary>

Es lo que ocurre cuando una variable de tipo genérico acaba señalando a un objeto de otro tipo, y salta como `ClassCastException` en un sitio donde no hay ningún cast escrito. Un varargs genérico lo permite porque `List<String>[]` se puede asignar a `Object[]`, y el array no puede comprobar el parámetro de tipo porque se borró en compilación. Verificado con el ejemplo de Baeldung: `class java.lang.Integer cannot be cast to class java.lang.String`. `@SafeVarargs` **no comprueba nada**: es una promesa del autor de que el método (a) no guarda nada en el array y (b) no deja escapar la referencia a ese array. Solo se admite en métodos que no puedan sobrescribirse: `static`, `final`, `private` o constructores.
</details>

**17. ¿Cuánto vale `factorial(13)` con el ejemplo de W3Schools, y qué debería hacer el método?**

<details><summary>Respuesta</summary>

Devuelve **1.932.053.504**, cuando `13!` vale 6.227.020.800. Verificado en JDK 25; a partir de 17 los resultados son negativos. El método usa `int` y el resultado desborda en silencio, igual que los contadores del capítulo de bucles. La fuente no menciona el límite: sus advertencias hablan de recursión infinita y de memoria, no de aritmética. Las defensas son `long` (llega hasta `20!`), `BigInteger` (sin límite práctico) o `Math.multiplyExact`, que hace la misma multiplicación pero lanza `ArithmeticException: integer overflow` en vez de dar la vuelta.
</details>

**18. ¿A qué profundidad se produce un `StackOverflowError`?**

<details><summary>Respuesta</summary>

**No hay un número.** Verificado en la misma máquina y la misma sesión: un método sin parámetros aguanta 8.497 marcos con `-Xss512k`, 47.224 por defecto y 204.646 con `-Xss8m`; un método con cinco parámetros `long` y cinco locales aguanta 1.573, 14.850 y 71.531 respectivamente. Depende del tamaño del marco, de la opción `-Xss` de la JVM y de la ejecución concreta —otra medición de este mismo capítulo, con un método casi idéntico, dio 64.459—. Cualquier código que dependa de una profundidad concreta está roto. Además `StackOverflowError` es un `Error`, no una `Exception`: se puede capturar, pero no se debe.
</details>

**19. ¿Sirve reescribir una recursión en forma de llamada de cola?**

<details><summary>Respuesta</summary>

En Java, **no**. Verificado con `sumaCola(1_000_000)`: lanza `StackOverflowError` con `-Xss512k`, con el valor por defecto y con `-Xss8m`. Java nunca ha implementado la eliminación de llamadas de cola, a diferencia de Scala con `@tailrec` o Kotlin con `tailrec`. Si el problema es la profundidad, la solución es un bucle o una pila explícita. Y si el problema es el tiempo y no la pila, la solución suele ser memoización: `fibIngenuo(40)` tarda 238 ms porque recalcula los mismos valores un número exponencial de veces, no porque la recursión sea lenta.
</details>

**20. ¿Cuántos parámetros admite un método, y cuántos debería tener?**

<details><summary>Respuesta</summary>

La plataforma admite **255**, y 256 dan `error: too many parameters`, verificado. Programiz afirma lo contrario («we can pass any number of arguments to a method»). Dos matices medidos: en un método **de instancia** el límite efectivo es 254 porque `this` ocupa una posición —la misma clase compila como estática y no como de instancia—, y la cuenta es de palabras, así que un `long` o un `double` ocupan dos. En cuanto a cuántos *debería* tener: tres o menos; cuatro o cinco son señal de que hay un objeto escondido, y seis o más piden un `record`, un builder o repensar la firma.
</details>

## 61. Fuentes

**Documentación oficial y especificación**

- [JLS §8.4 — Method Declarations](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4) y [§8.4.1 — Formal Parameters](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.1) — la gramática completa de una declaración, incluidos los parámetros de aridad variable.
- [JLS §8.4.2 — Method Signature](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html#jls-8.4.2) — la definición exacta de firma, de subfirma y de *override-equivalent*, y la frase que produce el `already defined in class` de la [sección 25](#25-el-error-real-cuando-dos-métodos-chocan).
- [JLS §15.12.2 — Compile-Time Step 2: Determine Method Signature](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.12.2) — las tres fases, su justificación por compatibilidad con Java anterior a SE 5.0, el aviso de que declarar un varargs puede cambiar el método elegido para llamadas existentes, y el ejemplo 15.12.2-3, que es el original del experimento de la [sección 34](#34-la-resolución-ocurre-en-compilación-y-queda-grabada).
- [JLS §15.7 — Evaluation Order](https://docs.oracle.com/javase/specs/jls/se25/html/jls-15.html#jls-15.7) — la garantía de evaluación de izquierda a derecha de los argumentos.
- [JLS §14.22 — Unreachable Statements](https://docs.oracle.com/javase/specs/jls/se25/html/jls-14.html#jls-14.22) — el análisis que produce `unreachable statement` y `missing return statement`.
- [JVMS §4.3.3 — Method Descriptors](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-4.html#jvms-4.3.3) y [§4.11](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-4.html#jvms-4.11) — de donde sale el límite de 255 unidades de argumento, y el de 65.535 bytes por método.
- [`java.util.Objects`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Objects.html) — `requireNonNull` y familia, la forma estándar de validar en la frontera.
- [`java.lang.reflect.Parameter`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/reflect/Parameter.html) — `isNamePresent()` y la explicación de por qué los nombres son `arg0` sin `-parameters`.
- [`@SafeVarargs`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/SafeVarargs.html) — las dos condiciones que el autor promete al ponerla, y dónde se puede poner.
- [Defining Methods](https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html) y [Passing Information to a Method or a Constructor](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html) — The Java Tutorials. La segunda es la mejor exposición breve del paso por valor que existe, y la única de las fuentes generales que trata el *shadowing* de un campo por un parámetro y cuándo es aceptable. Su único defecto es la antigüedad: no menciona `record`, ni `Optional`, ni ninguna evolución posterior a Java 8.
- [JEP 181 — Nest-Based Access Control](https://openjdk.org/jeps/181) y [JEP 358 — Helpful NullPointerExceptions](https://openjdk.org/jeps/358) — el primero explica por qué un método privado se invoca hoy con `invokevirtual`; el segundo, de dónde salen los mensajes con `<local2>` de la [sección 16](#16-los-nombres-de-los-parámetros-no-viajan-al-class).
- [JMH — Java Microbenchmark Harness](https://openjdk.org/projects/code-tools/jmh/) — la razón por la que este capítulo, igual que el de bucles, no da cifras de rendimiento.

**Las cinco fuentes de referencia de este capítulo, y dónde se equivocan**

- [Jenkov — Java Methods](https://jenkov.com/tutorials/java/methods.html). La mejor de las cinco en un aspecto muy concreto: es la única que trata **los parámetros `final`** y lo hace con exactitud —*«if the parameter is a reference to an object, the reference cannot be changed, but values inside the object can still be changed»*—, que es justo la distinción que la mitad de los tutoriales se salta. También es la única que avisa de que reasignar un parámetro produce código confuso. No le encontré ninguna afirmación falsa. **El problema es la antigüedad**: la página está fechada el **3 de marzo de 2015**, y se nota en lo que falta. No menciona **varargs**, ni la **sobrecarga**, ni que Java pasa **por valor**, ni nada posterior a Java 8. **Hueco grave:** su sección «Parameters vs. Variables» enseña a modificar el valor de un parámetro sin decir en ningún momento que ese cambio **no se ve desde fuera**; un lector que llegue ahí sin más contexto puede deducir exactamente lo contrario de lo que ocurre.
- [W3Schools — Java Methods](https://www.w3schools.com/java/java_methods.asp), con sus subpáginas de [Method Parameters](https://www.w3schools.com/java/java_methods_param.asp), [Return Values](https://www.w3schools.com/java/java_methods_return.asp), [Method Overloading](https://www.w3schools.com/java/java_methods_overloading.asp), [Scope](https://www.w3schools.com/java/java_scope.asp) y [Recursion](https://www.w3schools.com/java/java_recursion.asp). Correcta y bien escalonada en lo que cubre; la distinción entre parámetro y argumento («`fname` is a parameter, while `Liam`, `Jenny` and `Anja` are arguments») es de las explicaciones más claras que hay. **Problema 1:** su ejemplo de `factorial(int n)` **está roto a partir de 13** y no lo advierte en ningún sitio. Verificado en JDK 25: `factorial(13)` devuelve 1.932.053.504 en vez de 6.227.020.800, y `factorial(17)` ya es negativo. Su aviso —*«it's easy to accidentally write a method that never stops or uses too much memory»*— cubre la recursión infinita y la memoria, pero no el desbordamiento aritmético, que es el bug que ese ejemplo concreto tiene. **Problema 2:** su ejemplo `sum(k)` lanza `StackOverflowError` con entradas grandes —verificado con `sum(100000)`— y la palabra `StackOverflowError` no aparece en la página. **Huecos:** no menciona **varargs** en ninguna parte del capítulo de métodos, ni el **paso por valor**, ni la **ambigüedad** de sobrecarga, ni que la resolución ocurre en compilación. Su página de sobrecarga se queda en «pueden tener el mismo nombre si difieren los parámetros», que es cierto y es la punta del iceberg.
- [Programiz — Java Methods](https://www.programiz.com/java-programming/methods) y [Java Method Overloading](https://www.programiz.com/java-programming/method-overloading). Buena estructura y buenos diagramas, con la distinción entre argumento actual y parámetro formal bien planteada. Tiene **dos afirmaciones falsas comprobables**. **Error 1:** *«We can pass any number of arguments to a method.»* **Falso**: verificado en JDK 25, un método estático con 255 parámetros compila y con 256 da `error: too many parameters`. El límite está en el formato del `.class` ([JVMS §4.11](https://docs.oracle.com/javase/specs/jvms/se25/html/jvms-4.html#jvms-4.11)), y además en un método **de instancia** baja a 254 porque `this` ocupa una posición — la misma clase compila como estática y no como de instancia. **Error 2:** en sus «Important Points», *«It is not method overloading if we only change the return type of methods. There must be differences in the number of parameters.»* La segunda frase es **falsa** y contradice al propio artículo, que dos pantallas antes dice «different number of parameters, different types of parameters, or both» y cuyo segundo ejemplo sobrecarga `display(int)` con `display(String)`, mismo número de parámetros. Verificado: tres sobrecargas de un parámetro cada una compilan y funcionan. **Hueco:** no menciona en ningún momento las fases de resolución ni la ambigüedad, así que un lector se queda creyendo que la sobrecarga siempre elige «lo obvio».
- [Baeldung — Pass-By-Value as a Parameter Passing Mechanism in Java](https://www.baeldung.com/java-pass-by-value-or-pass-by-reference). El desarrollo es correcto y los diagramas de pila y heap son útiles. **Error, y de los graves porque está en la conclusión**: el artículo termina con *«1. For Primitive types, parameters are pass-by-value / 2. For Object types, the object reference is pass-by-reference»*. El punto 2 es **falso** y **lo desmiente su propio ejemplo**, que reasigna `b1 = new Foo(1)` dentro del método y comprueba con un `assertEquals` que `b.num` sigue valiendo 1 al volver. Reproducido en JDK 25: `a=Foo(2) b=Foo(1)`. Un artículo que empieza diciendo *«everything is strictly Pass-by-Value»* no puede terminar diciendo que los objetos se pasan por referencia. **Imprecisión menor:** afirma que *«Both values and references are stored in the stack memory»*, lo cual vale para las variables locales pero no para las referencias que viven dentro de otros objetos, que están en el heap.
- [Baeldung — Method Overloading and Overriding in Java](https://www.baeldung.com/java-method-overload-override). La motivación de la sobrecarga (`multiply2()`, `multiply3()` como ejemplo de mala API) es la mejor de las cinco fuentes, y la explicación de *type promotion* y de *static binding* frente a *dynamic binding* es correcta. **Error:** sobre dos métodos que solo difieren en el tipo de retorno dice *«the code simply wouldn't compile because of the method call ambiguity – the compiler wouldn't know which implementation of multiply() to call»*. La conclusión es correcta y **la razón es falsa**: verificado en JDK 25, el error es `method print() is already defined in class`, es un error **de declaración** que se produce aunque nadie llame nunca al método, y no hay dos candidatos entre los que elegir sino un método declarado dos veces. La ambigüedad de verdad tiene otro mensaje (`reference to f is ambiguous`) y ocurre en la llamada. Lo llamativo es que el propio Baeldung, en [Does a Method's Signature Include the Return Type in Java?](https://www.baeldung.com/java-method-signature-return-type), reporta el mensaje correcto: dos artículos del mismo sitio, sobre el mismo hecho, con explicaciones incompatibles. Ese segundo artículo, por cierto, es el mejor de los cuatro de Baeldung usados aquí: verifica una por una las cuatro cosas que **no** forman parte de la firma y trata bien la firma efectiva de los varargs.
- [Baeldung — Varargs in Java](https://www.baeldung.com/java-varargs). Es la única de las cinco fuentes que trata la **contaminación del heap**, con dos ejemplos —el `ClassCastException` sin casts y el método que deja escapar el array— que son exactamente los dos casos que `@SafeVarargs` debe descartar, y reproduje ambos sin cambios. **Desliz terminológico:** dice que los varargs funcionan *«thanks to their implicit autoboxing to and from Array»*. **Autoboxing** es el nombre de la conversión entre primitivos y wrappers (`int` ↔ `Integer`); lo que hace el compilador con un varargs es empaquetar los argumentos en un array, que es otra transformación y no se llama así. No cambia ningún comportamiento, pero mezclar los dos conceptos es justo el origen de la confusión de `Arrays.asList(int[])` de la [sección 40](#40-varargs-y-arrays-de-primitivos), que este artículo no menciona. **Diferencia de versión, no error:** cita el aviso del compilador como `warning: [varargs] Possible heap pollution...`; en JDK 25, con el ejemplo de este capítulo, la categoría que aparece es `[unchecked]`.

**Discusiones y artículos de la comunidad** (consultados para contrastar los casos límite)

- [Is Java "pass-by-reference" or "pass-by-value"?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value) — el hilo canónico, con más de dos mil votos y con la explicación del `swap` como argumento decisivo.
- [Why widening beats both Boxing and var-args in Overloading of a method?](https://stackoverflow.com/questions/2128034/why-widening-beats-both-boxing-and-var-args-in-overloading-of-a-method) — el orden entre las tres fases con el mismo trío de sobrecargas que usa la [sección 28](#28-fase-1-ensanchamiento), y las dos razones que lo explican: compatibilidad con Java anterior a la 5 y coste de ejecución (ensanchar es una instrucción; empaquetar es una asignación en el heap).
- [How is an overloaded method chosen when a parameter is the literal null value?](https://stackoverflow.com/questions/13033037/how-is-an-overloaded-method-chosen-when-a-parameter-is-the-literal-null-value) — por qué `g(null)` elige `String` frente a `Object`, y por qué `String` frente a `StringBuffer` es ambiguo. Es el mismo par de casos de las secciones [31](#31-el-más-específico-gana-y-el-caso-de-null) y [32](#32-cuando-ninguno-es-más-específico).
- [Maximum number of parameters in Java method declaration](https://stackoverflow.com/questions/30581531/maximum-number-of-parameters-in-java-method-declaration) — el hilo que cita el pasaje exacto de la JVMS y explica los dos matices que se miden en la [sección 48](#48-doscientos-cincuenta-y-cinco-parámetros): que `this` gasta una unidad y que `long` y `double` gastan dos.
- [Why does the JVM still not support tail-call optimization?](https://stackoverflow.com/questions/3616483/why-does-the-jvm-still-not-support-tail-call-optimization) — las razones históricas: el modelo de seguridad basado en inspección de la pila y la necesidad de conservar el stacktrace.
- [Why is it considered good practice to return an empty collection?](https://stackoverflow.com/questions/36779498/why-is-it-considered-good-practice-to-return-an-empty-collection) — el argumento completo, incluido el matiz de que `Collections.emptyList()` cambia un `NullPointerException` por un `UnsupportedOperationException` si alguien intenta escribir en ella.

**Nota sobre la verificación.** Todas las salidas, mensajes de excepción, errores del compilador y volcados de bytecode de este documento se obtuvieron ejecutando el código en Temurin JDK 25.0.3: los programas con `java Archivo.java`, los errores compilando a propósito ficheros que fallan con `javac`, los volcados con `javap -c` y `javap -v`, y los umbrales del JIT con `java -XX:+PrintFlagsFinal -version`. Las clases de 255 y 256 parámetros y la de 20.000 sentencias se generaron con un script, porque escribirlas a mano no era razonable. El experimento de compatibilidad binaria de la [sección 34](#34-la-resolución-ocurre-en-compilación-y-queda-grabada) se hizo compilando el cliente contra la primera versión de la biblioteca, sustituyendo **solo** el `.class` de la biblioteca y volviendo a ejecutar sin recompilar el cliente. **En este capítulo no hay comparativas de tiempo entre construcciones a propósito**, por la misma razón que en el de [Loops](11-loops.md): lo primero que hace el JIT con un método pequeño es eliminarlo por *inlining*, de modo que un microbenchmark casero mide el banco de pruebas y no el lenguaje. Los únicos números de tiempo que aparecen —los 238 ms de `fibIngenuo(40)`— ilustran un coste **algorítmico** exponencial, no una comparación entre formas de escribir lo mismo. Las cifras de profundidad de pila son recuentos exactos de marcos, y se presentan justamente para demostrar que **no** son constantes reproducibles.




