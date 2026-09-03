# Final Keyword

> **Bloque:** `02-POO` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25

**Alcance de este documento.** El capítulo anterior ([Static Keyword](04-static-keyword.md)) movió miembros de los objetos hacia la clase. Este cubre la otra palabra que aparece constantemente a su lado y que casi nadie entiende del todo: **`final`**, la que impide que algo cambie. Se aplica a variables, parámetros, campos, métodos y clases, y en cada sitio significa algo distinto.

El tema tiene una trampa central, y es tan famosa que la pregunta que la plantea en Stack Overflow lleva **más de 600.000 visitas y veinte respuestas**: *«¿cómo funciona `final` en Java? Yo todavía puedo modificar el objeto»*. La respuesta corta es que `final` **congela la variable, no el objeto**. La respuesta larga —qué es entonces la inmutabilidad, cómo se consigue de verdad, y qué garantiza `final` que ninguna otra construcción del lenguaje garantiza— es este documento.

**Por qué este tema separa a un junior de un senior.** `final` parece cosmético: una palabra que se pone «por buenas prácticas» y que muchos equipos ni usan. No lo es. Un campo `final` activa una regla del **modelo de memoria de Java** que hace que un objeto sea visible correctamente desde otros hilos **sin sincronización de ningún tipo** — la única forma que tiene el lenguaje de dar esa garantía gratis. Sin `final`, ese mismo objeto puede verse a medio construir desde otro hilo, con campos a cero, y el bug aparece una vez cada diez mil ejecuciones en producción. Es la diferencia entre saber escribir `final` y saber por qué existe.

**Lo que este capítulo desmiente.** Las dos fuentes del encargo tienen errores comprobables. Baeldung publica como «the compiler error» **mensajes que no son de `javac`** sino de Eclipse, los cuatro que cita. Y en el hilo de Stack Overflow, una respuesta muy votada afirma que la reflexión puede cambiar cualquier campo `final` («hold my beer»): en JDK 25 eso **solo sigue siendo cierto para campos de instancia**; con `static final` y con los componentes de un `record` lanza `IllegalAccessException`. Todo comprobado y con la salida real más abajo.

**Lo que NO entra aquí**, porque tiene documento propio: `static` (`04`, aunque `static final` se cruza constantemente y se enlaza), los bloques de inicialización y el ciclo de vida del objeto (`06`), la encapsulación (`08`), la herencia (`09`), el polimorfismo y el binding (`11`), los records (`16`), los enums (`17`) y las clases selladas (`18`). Aquí aparecen solo en lo que toca a `final`.

---

## Índice

**Parte I — Qué significa final**

1. [El problema que resuelve final](#1-el-problema-que-resuelve-final)
2. [Los cinco sitios donde se escribe](#2-los-cinco-sitios-donde-se-escribe)
3. [Una advertencia sobre los mensajes de error](#3-una-advertencia-sobre-los-mensajes-de-error)

**Parte II — Variables final**

4. [Variables locales](#4-variables-locales)
5. [Parámetros final](#5-parámetros-final)
6. [Campos final de instancia](#6-campos-final-de-instancia)
7. [Blank finals: la asignación diferida](#7-blank-finals-la-asignación-diferida)
8. [static final y por qué el constructor no vale](#8-static-final-y-por-qué-el-constructor-no-vale)
9. [Asignación definida: la regla real](#9-asignación-definida-la-regla-real)
10. [La tabla de dónde se asigna cada uno](#10-la-tabla-de-dónde-se-asigna-cada-uno)

**Parte III — final no es inmutabilidad**

11. [La pregunta de las 600.000 visitas](#11-la-pregunta-de-las-600000-visitas)
12. [Qué congela final exactamente](#12-qué-congela-final-exactamente)
13. [El caso de la colección](#13-el-caso-de-la-colección)
14. [final frente a inmutable: dos ejes distintos](#14-final-frente-a-inmutable-dos-ejes-distintos)
15. [Cómo se construye una clase realmente inmutable](#15-cómo-se-construye-una-clase-realmente-inmutable)
16. [Inmutabilidad superficial y profunda](#16-inmutabilidad-superficial-y-profunda)
17. [Copias defensivas](#17-copias-defensivas)
18. [Las colecciones inmutables de la biblioteca](#18-las-colecciones-inmutables-de-la-biblioteca)

**Parte IV — effectively final**

19. [Qué es effectively final](#19-qué-es-effectively-final)
20. [Por qué lo exigen las lambdas](#20-por-qué-lo-exigen-las-lambdas)
21. [La regla exacta y sus sorpresas](#21-la-regla-exacta-y-sus-sorpresas)
22. [Los rodeos que no deberías dar](#22-los-rodeos-que-no-deberías-dar)

**Parte V — Métodos final**

23. [Qué impide un método final](#23-qué-impide-un-método-final)
24. [Por qué existen: el contrato que se rompe](#24-por-qué-existen-el-contrato-que-se-rompe)
25. [Llamar a métodos desde el constructor](#25-llamar-a-métodos-desde-el-constructor)
26. [private y static ya son finales de hecho](#26-private-y-static-ya-son-finales-de-hecho)
27. [Cuándo marcar un método final](#27-cuándo-marcar-un-método-final)

**Parte VI — Clases final**

28. [Qué impide una clase final](#28-qué-impide-una-clase-final)
29. [Por qué String es final](#29-por-qué-string-es-final)
30. [El coste de cerrar una clase](#30-el-coste-de-cerrar-una-clase)
31. [Quién es final sin escribirlo](#31-quién-es-final-sin-escribirlo)
32. [sealed: la tercera opción desde Java 17](#32-sealed-la-tercera-opción-desde-java-17)

**Parte VII — final y la JVM**

33. [Constant folding de variables locales](#33-constant-folding-de-variables-locales)
34. [La semántica de campos final en el modelo de memoria](#34-la-semántica-de-campos-final-en-el-modelo-de-memoria)
35. [Publicación segura sin sincronización](#35-publicación-segura-sin-sincronización)
36. [Lo que rompe la garantía: this escapando](#36-lo-que-rompe-la-garantía-this-escapando)
37. [final y reflexión](#37-final-y-reflexión)
38. [Lo que ya no se puede modificar](#38-lo-que-ya-no-se-puede-modificar)
39. [Rendimiento: el mito y lo real](#39-rendimiento-el-mito-y-lo-real)

**Parte VIII — Diseño**

40. [final en parámetros: el debate](#40-final-en-parámetros-el-debate)
41. [Campos final por defecto](#41-campos-final-por-defecto)
42. [Records: final sin escribirlo](#42-records-final-sin-escribirlo)
43. [final y los frameworks](#43-final-y-los-frameworks)
44. [final y serialización](#44-final-y-serialización)

**Parte IX — Cierre**

45. [Casos de uso reales](#45-casos-de-uso-reales)
46. [Anti-patrones](#46-anti-patrones)
47. [Checklist y tabla de decisión](#47-checklist-y-tabla-de-decisión)
48. [Autoevaluación](#48-autoevaluación)
49. [Fuentes](#49-fuentes)

---

# Parte I — Qué significa final

## 1. El problema que resuelve final

Java tiene un problema de fondo: **por defecto, todo puede cambiar**. Una variable se puede reasignar, un método se puede sobrescribir, una clase se puede extender. Eso da flexibilidad, y a cambio quita garantías.

Pensá en esto:

```java
class CuentaBancaria {
    private String iban;

    public CuentaBancaria(String iban) { this.iban = iban; }

    public String getIban() { return iban; }
}
```

Nada impide que dentro de seis meses alguien añada un `setIban(...)`. Nada impide que otro programador extienda `CuentaBancaria` y sobrescriba `getIban()` para devolver otra cosa. Y nada impide que, dentro de un método, una variable que apuntaba a la cuenta del cliente A termine apuntando a la del cliente B por un error de tres líneas más abajo.

`final` es la herramienta que tiene el lenguaje para decir **«esto ya está decidido»**. Y esa afirmación tiene tres beneficiarios distintos:

1. **El compilador**, que rechaza el código que la incumpla. Es la única de las tres que actúa antes de ejecutar nada.
2. **El lector**, que sabe que no hace falta buscar en 400 líneas si esa variable cambia. No cambia.
3. **La JVM**, que puede optimizar sabiendo que el valor no se mueve, y —esto es lo que casi nadie sabe— que aplica a los campos `final` una **regla especial del modelo de memoria** que hace seguro compartir el objeto entre hilos. Es la [Parte VII](#34-la-semántica-de-campos-final-en-el-modelo-de-memoria).

Baeldung, en el artículo que este capítulo usa como fuente, enfoca `final` casi por completo desde el primer ángulo —limitar la extensibilidad— y cierra diciendo: «Although we may not use the final keyword often in our internal code, it may be a good design solution». Es una conclusión demasiado tímida. La recomendación de la industria desde *Effective Java* va en la dirección contraria: **`final` debería ser el estado por defecto de los campos**, y quitarlo debería ser la decisión que se justifica.

## 2. Los cinco sitios donde se escribe

`final` aparece en cinco lugares, y significa algo distinto en cada uno. Esta tabla es el mapa del capítulo:

| Dónde | Qué impide | Sección |
|---|---|---|
| **Variable local** | Reasignarla después de darle valor | [4](#4-variables-locales) |
| **Parámetro** | Reasignarlo dentro del método | [5](#5-parámetros-final) |
| **Campo** | Reasignarlo tras la construcción | [6](#6-campos-final-de-instancia) |
| **Método** | Que una subclase lo sobrescriba | [23](#23-qué-impide-un-método-final) |
| **Clase** | Que alguien la extienda | [28](#28-qué-impide-una-clase-final) |

Los tres primeros son variantes del mismo concepto —una variable que se asigna una sola vez— y los dos últimos son sobre herencia. Baeldung agrupa igual: clases, métodos y variables.

Hay un sexto uso que **no existe**, y conviene descartarlo: no hay `final` para constructores. Un constructor no se hereda, así que no hay nada que impedir. Escribirlo es un error de compilación.

## 3. Una advertencia sobre los mensajes de error

Antes de empezar, un aviso que afecta a toda la lectura de la fuente principal.

Baeldung ilustra cada regla con lo que llama «the compiler error». Por ejemplo, para una variable local reasignada publica:

```
The final local variable i may already have been assigned
```

Y para una referencia:

```
The final local variable cat cannot be assigned. It must be blank and not using a compound assignment
```

**Ninguno de los dos es el mensaje de `javac`.** Son mensajes de **Eclipse JDT**, el compilador integrado del IDE Eclipse, que tiene sus propios textos. El compilador de referencia de Java dice otra cosa. Comprobado en JDK 25:

```java
final Cat cat = new Cat();
final int i = 1;
i = 2;
cat = new Cat();
```

```
F1.java:8: error: cannot assign a value to final variable i
        i = 2;
        ^
F1.java:9: error: cannot assign a value to final variable cat
        cat = new Cat();
        ^
2 errors
```

Lo mismo pasa con los otros dos que publica: el de la clase final (`The type BlackCat cannot subclass the final class Cat`) y el del parámetro. Los reales aparecen en sus secciones correspondientes.

**Por qué importa.** Si buscás en Google el mensaje que publica Baeldung porque te lo encontraste en tu terminal, no lo vas a encontrar, porque tu terminal dice otra cosa. Y al revés: al leer el artículo podés pensar que tu compilador se comporta distinto cuando solo cambia el texto. Todos los mensajes de este documento son de `javac` 25 y están copiados literalmente de la salida.

---

# Parte II — Variables final

## 4. Variables locales

El caso más simple: una variable dentro de un método que solo se puede asignar una vez.

```java
public void procesar() {
    final int maximo = 100;
    final String nombre = "informe";

    maximo = 200;      // error
}
```

```
error: cannot assign a value to final variable maximo
```

No hace falta asignarla en la misma línea de la declaración. Esto también es válido:

```java
final int resultado;

if (condicion) {
    resultado = 1;
} else {
    resultado = 2;
}

System.out.println(resultado);
```

El compilador comprueba que **por cualquier camino de ejecución** la variable se asigne exactamente una vez. Si añadís un tercer camino que no la asigna, o si asignás dos veces en la misma rama, falla. Es la regla de la sección [9](#9-asignación-definida-la-regla-real).

Este patrón tiene una ventaja real sobre la alternativa:

```java
int resultado = 0;          // valor basura que no significa nada
if (condicion) resultado = 1;
else           resultado = 2;
```

Aquí el `0` inicial es ruido: nunca se usa, pero un lector tiene que comprobar que en efecto se sobrescribe siempre. Con `final` no hay valor basura y el compilador te obliga a cubrir todos los caminos.

## 5. Parámetros final

`final` es legal delante de un parámetro, y significa que no se puede reasignar dentro del método:

```java
public void metodo(final int x) {
    x = 1;
}
```

Baeldung publica como error `The final local variable x cannot be assigned...`, que otra vez es de Eclipse. El de `javac` 25 es distinto y más claro, porque dice explícitamente que es un parámetro:

```
F3.java:1: error: final parameter x may not be assigned
public class F3 { public void m(final int x) { x = 1; } }
                                               ^
1 error
```

Insisto en lo mismo que en el resto de la parte: **impide reasignar el parámetro, no modificar el objeto que apunta**. Esto compila sin problema:

```java
public void metodo(final List<String> lista) {
    lista.add("hola");         // perfectamente legal
    lista = new ArrayList<>(); // esto no
}
```

Si el debate de estilo sobre poner `final` en todos los parámetros te interesa, está en la sección [40](#40-final-en-parámetros-el-debate). Adelanto la conclusión: aporta poco y hay razones para las dos posturas.

## 6. Campos final de instancia

Un campo `final` de instancia no se puede reasignar una vez construido el objeto:

```java
public class Persona {
    private final String nombre;
    private final LocalDate nacimiento;

    public Persona(String nombre, LocalDate nacimiento) {
        this.nombre = nombre;
        this.nacimiento = nacimiento;
    }
}
```

Cada objeto `Persona` tiene su propio `nombre`, se fija en el constructor y ya no cambia. Si alguien intenta añadir un setter:

```java
public void setNombre(String n) { this.nombre = n; }
```

```
error: cannot assign a value to final variable nombre
```

Esta es, con diferencia, **la forma más útil de `final`** y la que más deberías usar. Un campo `final` de instancia:

- Documenta que el valor forma parte de la identidad del objeto y no de su estado cambiante.
- Elimina de golpe una clase entera de bugs: nadie puede dejar el objeto en un estado intermedio.
- Es el requisito de la inmutabilidad (sección [15](#15-cómo-se-construye-una-clase-realmente-inmutable)).
- Activa la garantía del modelo de memoria (sección [34](#34-la-semántica-de-campos-final-en-el-modelo-de-memoria)).

Baeldung introduce aquí una distinción útil que conviene recoger. Dice que los campos `final` son «either constants or write-once fields», y propone una pregunta para separarlos: *«¿incluiríamos este campo si fuéramos a serializar el objeto? Si no, entonces no es parte del objeto, sino una constante»*. Es un buen criterio:

```java
public class Pedido {
    public static final int MAX_LINEAS = 50;    // constante: no es del objeto
    private final String referencia;            // write-once: sí es del objeto
}
```

## 7. Blank finals: la asignación diferida

Un campo `final` **no tiene por qué inicializarse en su declaración**. Puede declararse vacío y asignarse después, en el constructor. Se llama *blank final*:

```java
public class Configuracion {
    private final int puerto;          // blank final: sin valor aquí

    public Configuracion(String entorno) {
        if ("produccion".equals(entorno)) {
            puerto = 443;
        } else {
            puerto = 8080;
        }
    }
}
```

Esto es lo que hace que `final` sea práctico y no una jaula. Sin blank finals, un campo `final` solo podría tener un valor literal, y no se podría escribir prácticamente ninguna clase inmutable con constructor.

Las reglas del blank final de instancia:

1. Se debe asignar **en todos los constructores**, o en un bloque de inicialización de instancia.
2. Se debe asignar **exactamente una vez** por cada camino de ejecución.
3. No se puede leer antes de asignarlo.

Si una clase tiene varios constructores, cada uno tiene que asignarlo —o delegar en otro que lo haga con `this(...)`:

```java
public class Punto {
    private final int x;
    private final int y;

    public Punto(int x, int y) { this.x = x; this.y = y; }

    public Punto() { this(0, 0); }        // delega: no asigna directamente
}
```

Este mecanismo ya apareció en dos capítulos anteriores, porque **es el error más repetido de las fuentes de este repositorio**: Jenkov afirma en su página de campos que «a final field must have an initial value assigned to it», y eso es falso desde siempre. Está documentado en [Attributes and Methods](02-attributes-and-methods.md) y en [Static Keyword](04-static-keyword.md).

## 8. static final y por qué el constructor no vale

Aquí está el punto exacto que originó la pregunta del hilo de Stack Overflow. El autor tenía esto, y funcionaba:

```java
class Test {
    private final List foo;

    public Test() {
        foo = new ArrayList();
        foo.add("foo");
    }
}
```

Y al añadir `static`, dejó de compilar:

```java
private static final List foo;
```

Comprobado en JDK 25, el error es:

```
F2.java:3: error: cannot assign a value to static final variable foo
    public Test() { foo = new java.util.ArrayList<>(); }
                    ^
1 error
```

**La razón, explicada bien.** Un campo `static` pertenece a la clase, no a los objetos (capítulo [Static Keyword](04-static-keyword.md)). Existe **una sola copia**, creada al inicializar la clase, antes de que exista ningún objeto. Un constructor, en cambio, se ejecuta **una vez por objeto**: dos `new Test()` lo ejecutarían dos veces. Si el constructor pudiera asignar el campo `static final`, el segundo `new` lo estaría reasignando, que es justo lo que `final` prohíbe.

La respuesta más votada del hilo lo formula con precisión, y merece citarse porque es la clave del asunto entero:

> «Un constructor puede invocarse solo una vez por cada objeto creado […] Un método puede invocarse tantas veces como quieras (incluso ninguna) y el compilador lo sabe. […] Si asignamos el `final foo` dentro del constructor, el compilador sabe que el constructor se invocará una sola vez, así que no hay problema.»

Y de ahí sale, sin necesidad de memorizarla, la regla de dónde se asigna un `static final`: en la declaración, o en un **bloque `static`**, que es lo que hace de «constructor de la clase»:

```java
// opción 1: en la declaración
private static final List<String> FOO = new ArrayList<>();

// opción 2: en un bloque static
private static final List<String> FOO;
static {
    FOO = new ArrayList<>();
    FOO.add("inicial");
}
```

**Una respuesta del hilo se equivoca aquí, y conviene señalarlo** porque tiene votos positivos. Sostiene que, en el caso `static final`, el campo se quedaría a `null` y «I assume your code throws a NullPointerException when you attempt to add an item to the list». No es así: **el programa no llega a ejecutarse**, porque `javac` lo rechaza antes, como muestra la salida de arriba. El propio autor de la pregunta ya lo decía: «Now it is a compilation error».

## 9. Asignación definida: la regla real

La regla que aplica el compilador no es «hay que inicializar en la declaración» ni «hay que inicializar en el constructor». Es la de **asignación definida** (*definite assignment*), y dice dos cosas a la vez:

- **Definitivamente asignada** antes de leerse.
- **Definitivamente NO asignada** antes de cada asignación.

La segunda es la que produce los errores raros. Si el compilador no puede demostrar que la variable estaba sin asignar, falla:

```java
class Doble {
    static final int X;
    static { X = 1; X = 2; }
}
```

```
error: variable X might already have been assigned
```

Fijate en el `might`: el compilador no dice «ya estaba asignada», dice «podría estarlo». Es un análisis conservador, y por eso rechaza cosas que un humano ve correctas:

```java
final int x;
if (cond)  x = 1;
if (!cond) x = 2;        // el compilador NO deduce que son excluyentes
System.out.println(x);   // error: variable x might not have been initialized
```

Escrito como `if/else`, compila sin problema. El compilador razona sobre la estructura del código, no sobre el significado de las condiciones.

## 10. La tabla de dónde se asigna cada uno

Resumen de toda la Parte II en una tabla:

| Tipo de `final` | Dónde se puede asignar | Cuántas veces |
|---|---|---|
| Variable local | En la declaración o después, antes de usarla | Una por camino |
| Parámetro | Nunca: llega ya asignado | Cero |
| Campo de instancia | Declaración, bloque de instancia `{ }`, o **todos** los constructores | Una por camino |
| Campo `static` | Declaración o bloque `static { }` | Una por camino |

Y el error que da cada mal uso, con el texto de `javac` 25:

| Qué hiciste | Mensaje |
|---|---|
| Reasignar una local o campo | `cannot assign a value to final variable X` |
| Reasignar un parámetro | `final parameter x may not be assigned` |
| Asignar un `static final` desde el constructor | `cannot assign a value to static final variable foo` |
| Asignar dos veces | `variable X might already have been assigned` |
| No asignar en algún camino | `variable X might not have been initialized` |

---

# Parte III — final no es inmutabilidad

Esta parte es el corazón del capítulo y la razón de que la pregunta que lo origina tenga 600.000 visitas.

## 11. La pregunta de las 600.000 visitas

El código exacto del hilo:

```java
class Test {
    private final List foo;

    public Test() {
        foo = new ArrayList();
        foo.add("foo");        // Modification-1
    }

    public static void main(String[] args) {
        Test t = new Test();
        t.foo.add("bar");      // Modification-2
        System.out.println("print - " + t.foo);
    }
}
```

Y la pregunta: si `foo` es `final`, ¿por qué las dos modificaciones funcionan?

Baeldung plantea lo mismo con un gato:

```java
final Cat cat = new Cat();
cat.setWeight(5);
assertEquals(5, cat.getWeight());
```

Comprobado en JDK 25:

```
peso tras mutar objeto final = 5
```

Funciona. Y no hay ninguna contradicción, porque **`final` nunca prometió que el objeto no cambiara**.

## 12. Qué congela final exactamente

La respuesta aceptada del hilo lo dice en una frase que vale más que cualquier explicación larga:

> «`final` es solo sobre la referencia en sí, y no sobre el contenido del objeto referenciado.»

Y añade algo igual de importante:

> «Java no tiene el concepto de inmutabilidad de objetos; eso se consigue diseñando el objeto con cuidado, y es un esfuerzo lejos de trivial.»

El modelo mental que lo resuelve para siempre: **una variable de referencia no contiene el objeto, contiene una dirección**. `final` congela lo que hay dentro de la variable, es decir la dirección. No dice nada sobre lo que hay en esa dirección.

```
    final List<String> lista
    ┌───────────────────┐
    │  dirección 0xA7D2 │  ← esto es lo que final congela
    └─────────┬─────────┘
              │  (no se puede cambiar a otra dirección)
              ▼
    ┌─────────────────────────────┐
    │  ArrayList en 0xA7D2        │
    │  ["foo", "bar", ...]        │  ← esto puede cambiar todo lo que quiera
    └─────────────────────────────┘
```

Para los tipos primitivos no hay tal distinción, porque la variable **sí** contiene el valor. Por eso `final int i = 1` sí congela el uno de verdad. La respuesta de BambooleanLogic en el hilo separa los dos casos exactamente así:

> - **Tipos valor:** para `int`, `double`, etc., asegura que el valor no puede cambiar.
> - **Tipos referencia:** asegura que la referencia nunca cambiará, o sea que siempre apuntará al mismo objeto. No hace ninguna garantía sobre que los valores dentro del objeto referenciado sigan iguales.

Y una consecuencia que sorprende: **`String` es la excepción que confunde a todo el mundo**. `final String s = "hola"` parece congelar el texto, y lo hace, pero no por `final`: es porque `String` es inmutable por diseño. Si `String` tuviera un método `append`, `final` no lo impediría.

## 13. El caso de la colección

El caso concreto que hay que reconocer, porque aparece constantemente en código real:

```java
public class Configuracion {
    public static final List<String> ROLES = new ArrayList<>();
}
```

Parece una constante. No lo es en absoluto:

```java
Configuracion.ROLES.add("ADMIN");        // legal desde cualquier parte
Configuracion.ROLES.clear();             // también
Configuracion.ROLES = new ArrayList<>(); // esto sí falla
```

Es una **variable global mutable disfrazada de constante**, y encima compartida entre todos los hilos. La `ArrayList` no es segura entre hilos, así que dos peticiones simultáneas pueden corromperla.

Las tres soluciones, de mejor a peor:

```java
// 1. inmutable de verdad (Java 9+)
public static final List<String> ROLES = List.of("ADMIN", "USER");

// 2. vista no modificable sobre una lista privada
private static final List<String> INTERNA = new ArrayList<>();
public  static final List<String> ROLES = Collections.unmodifiableList(INTERNA);

// 3. si tiene que ser mutable, al menos que sea concurrente y no pública
private static final List<String> ROLES_MUTABLES = new CopyOnWriteArrayList<>();
```

La opción 2 tiene una trampa que conviene conocer: `unmodifiableList` devuelve una **vista**, no una copia. Si alguien conserva una referencia a `INTERNA` y la modifica, el cambio se ve a través de la vista. Solo es segura si `INTERNA` es realmente privada y nadie la deja escapar.

## 14. final frente a inmutable: dos ejes distintos

Una de las respuestas del hilo hace la distinción que ordena todo el asunto:

> 1. Cuando alguien menciona un **objeto final**, significa que la referencia no puede cambiarse, pero su estado sí.
> 2. Un **objeto inmutable** es aquel cuyo estado no puede cambiarse, pero su referencia sí.

Son dos ejes independientes, y por tanto hay cuatro combinaciones:

| | Variable normal | Variable `final` |
|---|---|---|
| **Objeto mutable** | `List<String> l = new ArrayList<>()`<br>Cambia todo | `final List<String> l = new ArrayList<>()`<br>Cambia el contenido, no la variable |
| **Objeto inmutable** | `String s = "a"`<br>Cambia la variable, no el texto | `final String s = "a"`<br>No cambia nada |

Solo la esquina inferior derecha es una constante de verdad. Las otras tres tienen alguna forma de cambio.

El ejemplo canónico del eje vertical es `String`:

```java
String x = new String("abc");
x = "BCG";
```

La variable `x` pasa a apuntar a otro texto, pero el texto `"abc"` **no cambió**: sigue existiendo intacto en memoria hasta que el recolector se lo lleve. Eso es inmutabilidad.

## 15. Cómo se construye una clase realmente inmutable

Si `final` no basta, ¿qué hace falta? La receta completa tiene cinco pasos, y los cinco son necesarios:

```java
public final class Dinero {                       // 1. clase final
    private final long centimos;                  // 2. campos private final
    private final String divisa;

    public Dinero(long centimos, String divisa) {
        this.centimos = centimos;
        this.divisa = Objects.requireNonNull(divisa);
    }

    public long getCentimos() { return centimos; }  // 3. sin setters
    public String getDivisa()  { return divisa; }

    public Dinero mas(Dinero otro) {                // 4. las operaciones devuelven objetos nuevos
        if (!divisa.equals(otro.divisa)) throw new IllegalArgumentException("divisas distintas");
        return new Dinero(centimos + otro.centimos, divisa);
    }
}
```

1. **La clase es `final`.** Sin esto, alguien puede extenderla y añadir estado mutable, o sobrescribir un getter para que devuelva valores distintos cada vez. La inmutabilidad se rompe desde fuera.
2. **Todos los campos son `private final`.** `private` para que nadie los toque desde fuera; `final` para que ni la propia clase pueda reasignarlos, y —muy importante— para activar la garantía de publicación segura de la sección [35](#35-publicación-segura-sin-sincronización).
3. **No hay setters** ni ningún método que modifique el estado.
4. **Las operaciones devuelven instancias nuevas** en lugar de modificar la actual. Es lo que hacen `String.toUpperCase()`, `LocalDate.plusDays()` o `BigDecimal.add()`.
5. **Si algún campo es un objeto mutable, hay que copiarlo** al entrar y al salir. Es la sección [17](#17-copias-defensivas).

El quinto paso es el que más se olvida, y es el tema de las dos secciones siguientes.

## 16. Inmutabilidad superficial y profunda

Una clase puede cumplir los cuatro primeros pasos y seguir siendo mutable por dentro:

```java
public final class Equipo {
    private final String nombre;
    private final List<String> jugadores;

    public Equipo(String nombre, List<String> jugadores) {
        this.nombre = nombre;
        this.jugadores = jugadores;
    }

    public List<String> getJugadores() { return jugadores; }
}
```

Todo es `final`. Y aun así:

```java
List<String> lista = new ArrayList<>(List.of("Ana"));
Equipo e = new Equipo("Rojo", lista);

lista.add("Luis");                 // fuga 1: por la referencia original
e.getJugadores().add("Marta");     // fuga 2: por el getter
```

El equipo «inmutable» tiene tres jugadores y nadie llamó a ningún setter. Hay **dos agujeros**:

- **Al entrar:** el constructor guardó la misma lista que le pasaron. Quien la creó conserva una referencia y puede seguir modificándola.
- **Al salir:** el getter devuelve la lista interna. Cualquiera que la reciba puede modificarla.

A esto se le llama **inmutabilidad superficial** (*shallow*): la clase es inmutable, pero lo que contiene no. La **profunda** (*deep*) exige que todo lo alcanzable desde el objeto sea también inmutable.

## 17. Copias defensivas

La solución a los dos agujeros es copiar en ambas direcciones:

```java
public final class Equipo {
    private final String nombre;
    private final List<String> jugadores;

    public Equipo(String nombre, List<String> jugadores) {
        this.nombre = nombre;
        this.jugadores = List.copyOf(jugadores);      // copia al entrar
    }

    public List<String> getJugadores() {
        return jugadores;                              // ya es inmutable: no hace falta copiar al salir
    }
}
```

`List.copyOf` (Java 10+) hace las dos cosas de golpe: copia el contenido y devuelve una lista inmutable. Con eso, el getter puede devolver el campo directamente y sigue siendo seguro.

Antes de Java 10 había que hacerlo en dos pasos:

```java
this.jugadores = Collections.unmodifiableList(new ArrayList<>(jugadores));
```

El `new ArrayList<>(...)` corta el vínculo con la lista de fuera; el `unmodifiableList` impide modificar la copia.

**Cuidado con el orden.** Esto está mal y es un error frecuente:

```java
this.jugadores = new ArrayList<>(Collections.unmodifiableList(jugadores));  // inútil
```

Copia una vista no modificable a una `ArrayList` normal, así que el resultado es perfectamente modificable. La envoltura tiene que ir **por fuera**.

**Y el caso más traicionero: los arrays.** No existen arrays inmutables en Java. `Arrays.asList` no ayuda, porque devuelve una vista sobre el mismo array:

```java
private final int[] datos;

public int[] getDatos() { return datos; }              // MAL: el llamante puede escribir en él
public int[] getDatosSeguro() { return datos.clone(); } // bien: copia
```

Esto vale también para `Date`, `Calendar` y cualquier objeto mutable de la biblioteca. Con `Date` la solución moderna es no usarlo: `LocalDate` y compañía ya son inmutables.

## 18. Las colecciones inmutables de la biblioteca

Java tiene tres familias, y confundirlas es habitual:

| Forma | Qué devuelve | Al modificar |
|---|---|---|
| `List.of(...)` (Java 9+) | Colección **inmutable** independiente | `UnsupportedOperationException` |
| `List.copyOf(col)` (Java 10+) | Copia inmutable de otra colección | `UnsupportedOperationException` |
| `Collections.unmodifiableList(l)` | **Vista** sobre la lista original | `UnsupportedOperationException`, pero cambia si cambia el original |
| `Arrays.asList(array)` | **Vista** de tamaño fijo sobre el array | `add`/`remove` fallan, `set` **funciona** |

Los dos últimos son los que dan sorpresas. `Arrays.asList` en particular es un híbrido raro: no se puede añadir ni quitar, pero **sí se puede sustituir un elemento**, y el cambio se refleja en el array original.

Dos detalles más sobre `List.of` que conviene saber antes de usarlo:

- **No admite `null`.** `List.of("a", null)` lanza `NullPointerException` en la creación.
- **La inmutabilidad es superficial.** `List.of(unObjetoMutable)` impide cambiar la lista, no el objeto de dentro.

---

# Parte IV — effectively final

## 19. Qué es effectively final

Java 8 introdujo un concepto que no tiene palabra clave: **effectively final** (*efectivamente final*). Una variable es effectively final si **podría llevar `final` sin que el programa dejara de compilar**. Es decir: se asigna una vez y nunca se reasigna, aunque no lleve la palabra.

```java
int a = 5;          // effectively final: nunca se reasigna
int b = 5;
b = 6;              // NO es effectively final
```

Una de las respuestas del hilo lo señala como pregunta de entrevista frecuente desde Java 8, y con razón: es un concepto que el lenguaje usa mucho y que no se ve escrito en ninguna parte.

## 20. Por qué lo exigen las lambdas

El sitio donde te lo vas a encontrar es este:

```java
public static void main(String[] a) {
    int contador = 0;
    Supplier<Integer> s = () -> contador;
    contador++;
    System.out.println(s.get());
}
```

```
F9.java:5: error: local variables referenced from a lambda expression must be final or effectively final
        Supplier<Integer> s = () -> contador;
                                    ^
1 error
```

El mensaje es literal de `javac` 25 y es de los más claros que produce el compilador.

**Por qué existe esta regla.** Las variables locales viven en la pila del hilo, y desaparecen cuando el método termina. Una lambda puede sobrevivir al método que la creó —podés guardarla, pasarla a otro hilo, ejecutarla más tarde—. Para que eso funcione, Java **copia** el valor de la variable dentro de la lambda en el momento de crearla.

Y ahí está el problema: si la variable original pudiera cambiar después, habría dos valores distintos con el mismo nombre, y ninguna respuesta buena sobre cuál debería ver la lambda. En vez de elegir una respuesta confusa, Java prohíbe la situación.

Otros lenguajes eligieron distinto: en C# o JavaScript las lambdas capturan la variable **por referencia**, y ven los cambios. Eso trae sus propios problemas —el clásico bucle que crea diez lambdas y todas devuelven el mismo número— que Java se ahorra.

La misma regla aplicaba antes de Java 8 a las **clases anónimas**, pero exigiendo la palabra `final` escrita. Java 8 relajó la exigencia: basta con que **de hecho** no cambie.

## 21. La regla exacta y sus sorpresas

Casos que suelen sorprender:

**Un parámetro sí puede ser effectively final:**

```java
void metodo(String nombre) {
    Runnable r = () -> System.out.println(nombre);   // OK si no reasignás nombre
}
```

**Los campos NO tienen esta restricción:**

```java
public class Contador {
    private int valor = 0;

    public Runnable crear() {
        return () -> valor++;      // legal: valor es un campo, no una local
    }
}
```

La lambda captura `this`, y a través de él llega al campo. Los campos viven en el heap y no desaparecen al terminar el método, así que no hay nada que copiar. **Ojo con la consecuencia**: esa lambda mantiene vivo el objeto `Contador` entero mientras exista, que es la misma familia de retención de la sección 31 del capítulo [Static Keyword](04-static-keyword.md).

**La variable del `for` clásico no es effectively final:**

```java
for (int i = 0; i < 3; i++) {
    tareas.add(() -> System.out.println(i));   // error: i cambia en cada vuelta
}
```

**Pero la del `for-each` sí lo es:**

```java
for (String s : lista) {
    tareas.add(() -> System.out.println(s));   // OK
}
```

La diferencia es que el `for-each` **declara una variable nueva en cada iteración**, mientras que el `for` clásico reutiliza la misma. Es una asimetría poco conocida y muy útil.

## 22. Los rodeos que no deberías dar

Cuando alguien se topa con la restricción, suele intentar esto:

```java
final int[] contador = {0};                     // truco del array de un elemento
Runnable r = () -> contador[0]++;
```

Funciona: el array es effectively final, y su contenido no. También circula la versión con `AtomicInteger`:

```java
AtomicInteger contador = new AtomicInteger();
Runnable r = () -> contador.incrementAndGet();
```

**Cuándo cada uno es aceptable:**

- El **array de un elemento** es un truco puro, ilegible y sin ninguna garantía entre hilos. Evitalo.
- El **`AtomicInteger`** es legítimo **si de verdad necesitás un contador compartido entre hilos**, porque es lo que hace. Usarlo solo para saltarse al compilador en código de un solo hilo es abusar de una herramienta de concurrencia.

En la mayoría de los casos, toparse con esta restricción es señal de que el código está pidiendo otra forma. Si estás sumando dentro de un `forEach`, lo que querés es un `Stream`:

```java
// en vez de esto
int[] suma = {0};
lista.forEach(x -> suma[0] += x);

// esto
int suma = lista.stream().mapToInt(Integer::intValue).sum();
```

---

# Parte V — Métodos final

## 23. Qué impide un método final

Un método `final` no puede ser sobrescrito por una subclase:

```java
class Perro {
    public final void sonido() { }
}

class PerroNegro extends Perro {
    public void sonido() { }
}
```

Baeldung publica como error un texto de Eclipse. El de `javac` 25 es:

```
F5.java:2: error: sonido() in PerroNegro cannot override sonido() in Perro
class PerroNegro extends Perro { public void sonido() {} }
                                             ^
  overridden method is final
1 error
```

Fijate en la segunda línea del mensaje: `overridden method is final`. `javac` da la causa aparte, que es más útil.

Lo que **sí** se puede hacer: la subclase puede seguir llamando al método, y puede añadir métodos nuevos. Solo no puede cambiar el comportamiento de ese.

## 24. Por qué existen: el contrato que se rompe

Baeldung da el ejemplo perfecto y merece desarrollarlo. La clase `Thread` es extensible —crear un hilo propio extendiendo `Thread` es normal— pero su método `isAlive()` es `final`:

> «Es imposible sobrescribir el método `isAlive()` correctamente por muchas razones. Una de ellas es que este método es nativo.»

Un método `native` está implementado fuera de Java, en código de la máquina virtual. Si una subclase pudiera sobrescribirlo y devolver `true` cuando el hilo está muerto, **la propia JVM haría cosas incorrectas**, porque usa ese método internamente.

Ese es el patrón general: un método debe ser `final` cuando **otro código depende de que haga exactamente lo que dice**. Baeldung lo formula así:

> «Si algunos métodos de nuestra clase son llamados por otros métodos, deberíamos considerar hacer `final` los métodos llamados. Si no, sobrescribirlos puede afectar al trabajo de quien los llama y causar resultados sorprendentes.»

Ejemplo concreto:

```java
public class ProcesadorPagos {
    public void procesar(Pago p) {
        validar(p);                 // si una subclase sobrescribe validar()...
        cobrar(p);                  // ...este cobro se ejecuta sin validación
    }

    protected final void validar(Pago p) {
        if (p.getImporte() <= 0) throw new IllegalArgumentException();
    }

    protected void cobrar(Pago p) { }
}
```

`validar` es `final` porque el resto del algoritmo asume que se ejecutó. `cobrar` no lo es, porque es precisamente el punto donde queremos que las subclases decidan. Esa combinación —esqueleto fijo con huecos concretos— es el patrón *Template Method*, y `final` es lo que lo hace fiable.

## 25. Llamar a métodos desde el constructor

Baeldung menciona el caso más peligroso de todos:

> «Si nuestro constructor llama a otros métodos, deberíamos en general declarar esos métodos `final` por la razón anterior.»

La razón es más grave de lo que el artículo sugiere. Un método sobrescribible llamado desde el constructor de la superclase **se ejecuta antes de que la subclase esté construida**:

```java
class Base {
    Base() {
        inicializar();               // llama al método sobrescrito
    }
    protected void inicializar() { }
}

class Derivada extends Base {
    private final String valor = "listo";

    @Override
    protected void inicializar() {
        System.out.println("valor = " + valor);
    }
}
```

```java
new Derivada();
```

Imprime **`valor = null`**. Y `valor` es un campo `final` con inicializador, que a simple vista nunca puede ser nulo.

La secuencia lo explica: el constructor de `Derivada` llama primero al de `Base`; `Base` llama a `inicializar()`, que está sobrescrito y por tanto ejecuta el de `Derivada`; y en ese momento **los inicializadores de campo de `Derivada` todavía no han corrido**. El campo tiene su valor por defecto.

Es el mismo mecanismo de valores por defecto de la sección 11 del capítulo [Static Keyword](04-static-keyword.md), aplicado a la instancia. La regla que evita el problema es tajante: **desde un constructor, llamá solo a métodos `private`, `static` o `final`**. Cualquiera de los tres es imposible de sobrescribir.

## 26. private y static ya son finales de hecho

Una precisión terminológica que aparece en entrevistas: **los métodos `private` y `static` no se pueden sobrescribir**, aunque no lleven `final`.

- Un método **`private`** no es visible desde la subclase, así que esta no puede sobrescribirlo. Si escribe un método con la misma firma, es un método nuevo y sin relación.
- Un método **`static`** no participa del despacho dinámico: si la subclase declara uno igual, lo **oculta** (*hiding*), no lo sobrescribe. Está desarrollado en la sección 37 del capítulo [Static Keyword](04-static-keyword.md).

Por eso escribir `private final void metodo()` es redundante. No es un error, pero no añade nada.

## 27. Cuándo marcar un método final

La recomendación práctica, ordenada:

**Marcalo `final` cuando:**

- Otros métodos de la clase dependen de su comportamiento exacto (Template Method).
- Se llama desde el constructor.
- Implementa una regla de negocio o de seguridad que no debe poder desactivarse.
- Forma parte del contrato de `equals`/`hashCode` en una clase pensada para heredarse.

**No lo marques cuando:**

- Estás escribiendo una clase interna de tu aplicación que nadie va a extender. El `final` no aporta y añade ruido.
- El método es un punto de extensión deliberado.
- El método ya es `private` o `static`.

Y una advertencia práctica que Baeldung no menciona y que te va a morder antes que ninguna otra: **Mockito no puede simular métodos `final` sin configuración adicional**. Si marcás `final` un método de un servicio, los tests que lo mockeen dejarán de funcionar. Está en la sección [43](#43-final-y-los-frameworks).

---

# Parte VI — Clases final

## 28. Qué impide una clase final

Una clase `final` no puede extenderse:

```java
final class Gato { }
class GatoNegro extends Gato { }
```

`javac` 25 dice:

```
F4.java:2: error: cannot inherit from final Gato
class GatoNegro extends Gato { }
                        ^
1 error
```

(Baeldung publica `The type BlackCat cannot subclass the final class Cat`, que es de Eclipse.)

Baeldung hace aquí una aclaración necesaria, y es la misma idea de la Parte III aplicada a las clases:

> «Nótese que la palabra `final` en la declaración de una clase no significa que los objetos de esta clase sean inmutables. Podemos cambiar los campos del objeto `Cat` libremente. […] Simplemente no podemos extenderla.»

Y la pregunta que cierra su sección tiene respuesta interesante: ¿qué diferencia hay entre marcar `final` todos los métodos de una clase y marcar `final` la clase? Que en el primer caso **todavía se puede extender y añadir métodos nuevos**; en el segundo, no. La clase `final` es estrictamente más restrictiva.

## 29. Por qué String es final

El razonamiento de Baeldung es correcto y vale la pena desarrollarlo, porque explica una decisión de diseño real del lenguaje:

> «Considerá la situación de poder extender la clase `String`, sobrescribir cualquiera de sus métodos y sustituir todas las instancias de `String` por instancias de nuestra subclase específica. El resultado de las operaciones sobre objetos `String` se volvería impredecible. Y dado que `String` se usa en todas partes, es inaceptable.»

Hay tres razones concretas, y las tres son buenas:

**1. Seguridad.** Muchas comprobaciones de seguridad reciben una `String` y la validan:

```java
void abrirFichero(String ruta) {
    if (!esRutaPermitida(ruta)) throw new SecurityException();
    // abrir ruta
}
```

Si `String` fuera extensible, alguien podría pasar una subclase cuyo método devolviera `"/tmp/seguro"` en la validación y `"/etc/passwd"` al abrir. Se llama ataque **TOCTOU** (*time of check to time of use*), y `final` lo hace imposible de raíz.

**2. El pool de cadenas.** Java reutiliza los literales de cadena: dos `"hola"` en distintos sitios del programa son el mismo objeto. Eso solo es seguro si nadie puede modificar una cadena ni sustituirla por algo que se comporte distinto.

**3. El hash cacheado.** `String` calcula su `hashCode` una vez y lo guarda. Si el contenido pudiera cambiar, el hash quedaría obsoleto y una `String` usada como clave de un `HashMap` se volvería irrecuperable.

Las otras clases `final` de la biblioteca lo son por razones parecidas: `Integer`, `Long`, `Double` y todos los envoltorios; `LocalDate` y la familia de `java.time`; `Optional`.

## 30. El coste de cerrar una clase

Baeldung es honesta y advierte del otro lado, cosa que muchos textos no hacen:

> «Nótese que hacer una clase `final` significa que ningún otro programador puede mejorarla. Imaginá que estamos usando una clase de la que no tenemos el código fuente y hay un problema con uno de sus métodos. Si la clase es `final`, no podemos extenderla para sobrescribir el método y arreglar el problema. En otras palabras, perdemos la extensibilidad, uno de los beneficios de la programación orientada a objetos.»

Es un argumento real, y tiene un contraargumento igual de real, que es el ítem correspondiente de *Effective Java*: **«diseñá y documentá para la herencia, o si no prohibila»**. Una clase que se puede extender pero no fue pensada para ello es una trampa: cualquier cambio interno futuro puede romper subclases que no controlás, porque dependen de detalles que nunca prometiste mantener.

La postura equilibrada:

| Contexto | Recomendación |
|---|---|
| Clase de una **librería pública** | `final` por defecto, salvo que documentes la herencia explícitamente |
| Clase **interna** de tu aplicación | No hace falta `final`; el equipo controla todos los usos |
| Clase que representa un **valor** (dinero, coordenada, identificador) | `final` siempre |
| Clase pensada como **punto de extensión** | Nunca `final`; documentá qué se puede sobrescribir |

Y desde Java 17 hay una opción intermedia que no existía cuando se escribió el artículo de Baeldung: `sealed`, sección [32](#32-sealed-la-tercera-opción-desde-java-17).

## 31. Quién es final sin escribirlo

Varias construcciones del lenguaje son `final` implícitamente. Comprobado por reflexión en JDK 25:

```java
System.out.println("record Punto es final? " + Modifier.isFinal(Punto.class.getModifiers()));
System.out.println("campo x del record es final? " + Modifier.isFinal(Punto.class.getDeclaredField("x").getModifiers()));
System.out.println("enum es final? " + Modifier.isFinal(DayOfWeek.class.getModifiers()));
System.out.println("String es final? " + Modifier.isFinal(String.class.getModifiers()));
System.out.println("Circulo (implementa sealed) es final? " + Modifier.isFinal(Circulo.class.getModifiers()));
```

```
record Punto es final? true
campo x del record es final? true
enum es final? true
String es final? true
Circulo (implementa sealed) es final? true
```

Es decir:

- **Los records son `final`.** No se pueden extender, y **sus componentes son campos `final`** aunque nunca escribas la palabra.
- **Los enums son `final`**… con una excepción: si alguna constante tiene cuerpo propio (`ACTIVO { ... }`), el enum se compila como clase abstracta con subclases anónimas, y entonces no lo es.
- Las clases que implementan una interfaz `sealed` deben ser `final`, `sealed` o `non-sealed`; en el ejemplo, `Circulo` es un record, y por tanto `final`.

## 32. sealed: la tercera opción desde Java 17

Hasta Java 17 solo había dos posturas: clase abierta a todo el mundo, o clase `final` cerrada a todos. Las **clases selladas** añaden la intermedia: abierta a una lista concreta.

```java
public sealed interface Forma permits Circulo, Cuadrado, Triangulo { }

public record Circulo(double radio) implements Forma { }
public record Cuadrado(double lado) implements Forma { }
public final class Triangulo implements Forma { }
```

`Forma` se puede implementar, pero **solo por esas tres**. Cualquier otra clase que lo intente no compila.

Cada subtipo permitido tiene que declarar su propia postura:

| Modificador del subtipo | Significa |
|---|---|
| `final` | Se cierra aquí |
| `sealed` | Continúa la jerarquía con otra lista permitida |
| `non-sealed` | Vuelve a abrirse a cualquiera |

El beneficio que justifica la funcionalidad es que **el compilador sabe que la lista está completa**, y eso hace exhaustivo un `switch`:

```java
double area(Forma f) {
    return switch (f) {
        case Circulo c   -> Math.PI * c.radio() * c.radio();
        case Cuadrado q  -> q.lado() * q.lado();
        case Triangulo t -> t.area();
        // sin default: el compilador comprueba que están todos
    };
}
```

Si mañana alguien añade un cuarto tipo a `permits` y olvida actualizar el `switch`, **el código no compila**. Con una jerarquía abierta eso es imposible de garantizar y hay que poner un `default` que lanza una excepción en tiempo de ejecución.

Cuándo usar cada una:

| Situación | Elección |
|---|---|
| Un tipo de valor concreto (dinero, ID) | `final` |
| Un conjunto cerrado y conocido de alternativas | `sealed` |
| Un punto de extensión para terceros | Ni una ni otra |

---

# Parte VII — final y la JVM

## 33. Constant folding de variables locales

El capítulo anterior demostró que un `static final` con valor literal se copia en el bytecode de quien lo usa. Lo mismo ocurre con una **variable local `final`**, y se ve igual de bien:

```java
public class F8 {
    public static void main(String[] a) {
        final int CONSTANTE = 100;
        int normal = 100;
        System.out.println("con final:  " + CONSTANTE);
        System.out.println("sin final:  " + normal);
    }
}
```

Bytecode real, en JDK 25:

```
 2: istore_1
 6: ldc           #13    // String con final:  100
 8: invokevirtual #15    // Method java/io/PrintStream.println:(Ljava/lang/String;)V
14: iload_1
15: invokedynamic #21,  0  // InvokeDynamic #0:makeConcatWithConstants:(I)Ljava/lang/String;
20: invokevirtual #15    // Method java/io/PrintStream.println:(Ljava/lang/String;)V
```

Los dos casos son visiblemente distintos:

- **Con `final`**: `ldc "con final:  100"`. El compilador resolvió la concatenación entera en tiempo de compilación y dejó un solo literal. La variable ni se lee.
- **Sin `final`**: `iload_1` para cargar la variable y `invokedynamic makeConcatWithConstants` para concatenar en ejecución.

La condición es la misma que en el capítulo anterior: `final` **y** valor que sea una expresión constante en tiempo de compilación. Un `final int x = calcular()` no se pliega.

Ojo con no sacar la conclusión equivocada: esto **no significa que poner `final` en las locales acelere el programa**. Solo ocurre con constantes literales, y el JIT hace optimizaciones equivalentes de todas formas. Está en la sección [39](#39-rendimiento-el-mito-y-lo-real).

## 34. La semántica de campos final en el modelo de memoria

Esta sección es la razón principal por la que `final` importa de verdad, y no aparece ni en Baeldung ni en ninguna de las veinte respuestas del hilo.

La especificación del lenguaje dedica una sección entera al asunto, **§17.5, *final Field Semantics***, y empieza así:

> «Los campos declarados `final` se inicializan una vez, pero nunca cambian en circunstancias normales. La semántica detallada de los campos `final` es algo distinta de la de los campos normales. En particular, los compiladores tienen mucha libertad para mover lecturas de campos `final` a través de barreras de sincronización y llamadas a métodos arbitrarios o desconocidos. Del mismo modo, se permite a los compiladores mantener el valor de un campo `final` **cacheado en un registro** y no recargarlo de memoria en situaciones donde un campo no `final` tendría que recargarse.»

Ese último detalle merece una corrección a una respuesta del hilo. Una de ellas afirma que los valores `final` «se cargan en la caché L1» mientras que los no finales van a L2 y se recargan de memoria principal. **Eso no es lo que dice la especificación ni lo que hace la JVM.** El compilador puede mantener el valor en un **registro del procesador**, que no es lo mismo que la caché L1 y no es una decisión que dependa de la palabra `final` a nivel de hardware. La afirmación de esa respuesta mezcla conceptos de arquitectura con una garantía del lenguaje que va por otro lado.

La garantía real es la que viene ahora, y es mucho más valiosa.

## 35. Publicación segura sin sincronización

La JLS continúa:

> «Los campos `final` también permiten a los programadores implementar objetos inmutables *thread-safe* sin sincronización. Un objeto inmutable *thread-safe* es visto como inmutable por todos los hilos, incluso si se usa una carrera de datos para pasar referencias al objeto entre hilos.»

Y da la regla operativa:

> «Un objeto se considera completamente inicializado cuando su constructor termina. Un hilo que solo puede ver una referencia a un objeto después de que ese objeto se haya inicializado completamente tiene garantizado ver los valores correctamente inicializados de los campos `final` de ese objeto.»

Traducido a lo que significa en la práctica. Considerá esta clase **sin** `final`:

```java
class Configuracion {
    int puerto;                    // NO final
    String host;

    Configuracion() {
        puerto = 8080;
        host = "localhost";
    }
}
```

Y este código en dos hilos:

```java
// hilo 1
config = new Configuracion();

// hilo 2
if (config != null) {
    System.out.println(config.puerto);    // puede imprimir 0
}
```

El hilo 2 **puede ver `config` no nulo y a la vez ver `puerto` valiendo 0**. Parece imposible, pero es legal: sin sincronización, el procesador y el compilador pueden reordenar la escritura de la referencia y la de los campos. El objeto se ve «a medio construir».

Con los campos `final`, esto **no puede pasar**:

```java
class Configuracion {
    final int puerto;
    final String host;

    Configuracion() {
        puerto = 8080;
        host = "localhost";
    }
}
```

La JLS define el mecanismo como una acción de **congelación** (*freeze*):

> «Sea `o` un objeto, y `c` un constructor de `o` en el que se escribe un campo `final` `f`. Una acción de congelación sobre el campo `final` `f` de `o` tiene lugar cuando `c` termina, normal o abruptamente.»

Cualquier hilo que vea la referencia después de esa congelación ve los campos `final` correctos. Y la garantía es **transitiva**: *Java Concurrency in Practice* lo formula así:

> «Cualquier variable que se pueda *alcanzar* a través de un campo `final` de un objeto correctamente construido (como los elementos de un array `final` o el contenido de un `HashMap` referenciado por un campo `final`) también está garantizado que sea visible para otros hilos.»

Es decir: si tenés un `final Map<String,String> config` que llenaste en el constructor y no volviste a tocar, **cualquier hilo que reciba el objeto ve el mapa completo**, sin `synchronized`, sin `volatile` y sin coste. Aleksey Shipilev, ingeniero de la JVM, lista esta como una de las cuatro formas de publicación segura en Java, junto a `synchronized`, `volatile` y las clases atómicas.

**Esto es lo que convierte `final` de detalle de estilo en herramienta de concurrencia.** Es la razón técnica de por qué los objetos inmutables son seguros entre hilos «gratis».

## 36. Lo que rompe la garantía: this escapando

La garantía tiene una condición, y la JLS la enuncia con claridad:

> «El modelo de uso de los campos `final` es simple: fijá los campos `final` de un objeto en el constructor de ese objeto; y **no escribas una referencia al objeto en construcción en un lugar donde otro hilo pueda verla antes de que el constructor termine**.»

Lo segundo se llama que **`this` se escape**, y ocurre más de lo que parece:

```java
public class Servicio {
    private final Config config;

    public Servicio(Config config) {
        this.config = config;
        Registro.registrar(this);        // MAL: this escapa antes de terminar
    }
}
```

```java
public class Listener {
    private final int umbral;

    public Listener(EventBus bus, int umbral) {
        bus.suscribir(this);             // MAL: el bus puede usarlo ya
        this.umbral = umbral;
    }
}
```

En los dos casos, otro hilo puede obtener la referencia **antes de la congelación**, y entonces todas las garantías desaparecen: puede ver `umbral` a cero. Shipilev lo resume sin rodeos: *«si alguien vio la instancia antes, todas las apuestas están canceladas: hay una cadena de acceso que evita la acción de congelación»*.

Las dos formas de arreglarlo:

```java
// 1. registrar fuera del constructor
Servicio s = new Servicio(config);
Registro.registrar(s);

// 2. factoría static que construye y luego publica
public static Servicio crear(Config c) {
    Servicio s = new Servicio(c);
    Registro.registrar(s);
    return s;
}
```

Hay una forma más sutil de que `this` escape y que cuesta ver: **llamar a un método sobrescribible desde el constructor**, que es exactamente el problema de la sección [25](#25-llamar-a-métodos-desde-el-constructor). El método de la subclase recibe un `this` a medio construir.

## 37. final y reflexión

Una de las respuestas del hilo, titulada «*A final variable can only be assigned once — Reflection: wowo wait, hold my beer*», demuestra que la reflexión puede reasignar un campo `final` tantas veces como quiera, y publica una salida donde el mismo campo `final` toma cinco valores.

**Comprobé si eso sigue siendo cierto en JDK 25, y la respuesta es «solo a medias».** El programa de prueba intenta modificar cuatro tipos distintos de campo `final`:

```java
static void intenta(String etiqueta, Object obj, Class<?> clase, String campo, Object nuevo) {
    try {
        Field f = clase.getDeclaredField(campo);
        f.setAccessible(true);
        f.set(obj, nuevo);
        System.out.println(etiqueta + " -> MODIFICADO, ahora vale " + f.get(obj));
    } catch (Throwable t) {
        System.out.println(etiqueta + " -> " + t.getClass().getSimpleName() + ": " + t.getMessage());
    }
}
```

Salida real en JDK 25:

```
final de instancia       -> MODIFICADO, ahora vale 99
   leido por codigo normal: i.valor = 99
static final (constante) -> IllegalAccessException: Can not set static final int field F6$Estatica.VALOR to java.lang.Integer
static final (calculado) -> IllegalAccessException: Can not set static final int field F6$EstaticaCalculada.VALOR to java.lang.Integer
final de un record       -> IllegalAccessException: Can not set final int field F6$Punto.x to java.lang.Integer
```

Es decir: la respuesta del hilo **sigue siendo válida solo para campos `final` de instancia en clases normales**. Para `static final` y para los componentes de un `record`, JDK 25 lo prohíbe.

Vale la pena saber además que la JLS **contempla** esta modificación y le da semántica, precisamente porque la deserialización la necesita:

> «En algunos casos, como la deserialización, el sistema necesitará cambiar los campos `final` de un objeto tras la construcción. Los campos `final` se pueden cambiar mediante reflexión y otros medios dependientes de la implementación. El único patrón en el que esto tiene semántica razonable es aquel en que un objeto se construye y después se actualizan sus campos `final`. **El objeto no debería hacerse visible a otros hilos, ni deberían leerse sus campos `final`, hasta que todas las actualizaciones estén completas.**»

Traducido: si modificás un campo `final` por reflexión sobre un objeto que ya circula por el programa, **el resultado es impredecible**, porque otras partes pueden haber cacheado el valor viejo. No es que «funcione»: es que funciona en el caso concreto que probaste.

## 38. Lo que ya no se puede modificar

El endurecimiento no fue de golpe, y conviene tener la línea temporal porque explica por qué tanto código y tantas respuestas de Stack Overflow ya no funcionan.

El truco clásico que circula desde 2010 es este:

```java
Field modifiersField = Field.class.getDeclaredField("modifiers");
modifiersField.setAccessible(true);
modifiersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);
field.set(null, nuevoValor);
```

Quita el bit `final` del propio objeto `Field` y luego asigna. **Desde Java 12 ya no funciona**: el campo `modifiers` de `Field` está en la lista de filtrado de la reflexión, y el intento produce `NoSuchFieldException: modifiers`.

La documentación de `Field.set` en Java 17 y posteriores lista los campos **no modificables**:

> - campos `static final` declarados en cualquier clase o interfaz
> - campos `final` declarados en una *hidden class*
> - campos `final` declarados en un `record`

Y `setAccessible` aclara su alcance:

> «Este método no se puede usar para habilitar acceso de **escritura** a un campo `final` no modificable. […] El flag `accessible` en `true` suprime las comprobaciones de control de acceso del lenguaje solo para habilitar acceso de **lectura** a estos campos.»

**Por qué se hizo.** El motivo aparece en el bug de OpenJDK que introdujo la restricción para records:

> «Los campos `final` no son de confianza porque no son verdaderamente finales y pueden modificarse mediante reflexión. Para las funcionalidades nuevas, es deseable hacer los campos `final` verdaderamente finales donde sea posible. […] Un compilador JIT puede confiar en que estos campos `final` son verdaderamente finales.»

Es un intercambio explícito: se pierde la capacidad de hackear el campo, y a cambio **el JIT puede optimizar de verdad**, tratando el valor como constante. En los records y en los `static final` de cualquier clase, ese trato ya se aplica.

La consecuencia práctica: si mantenés código que hace este tipo de reflexión —tests que fuerzan un `BuildConfig.DEBUG`, librerías de mocking antiguas, utilidades que sobrescriben constantes— **va a fallar al migrar a Java 17 o superior**, y la solución no es buscar un rodeo nuevo sino cambiar el diseño para no necesitarlo.

## 39. Rendimiento: el mito y lo real

Circula la idea de que `final` acelera el código. Conviene separar lo cierto de lo falso.

**Lo que es cierto:**

- Las **expresiones constantes** `final` se pliegan en tiempo de compilación, como demostró la sección [33](#33-constant-folding-de-variables-locales). Es un efecto real pero minúsculo, y solo aplica a literales.
- Los campos `static final` y los de `record` **son de confianza para el JIT** desde que dejaron de ser modificables por reflexión, y eso sí permite optimizaciones agresivas.
- La JLS permite mantener un campo `final` en un registro sin recargarlo (sección [34](#34-la-semántica-de-campos-final-en-el-modelo-de-memoria)).

**Lo que es falso o irrelevante:**

- Que un **método `final` sea más rápido** por evitar la tabla virtual. HotSpot usa *Class Hierarchy Analysis*: si en tiempo de ejecución solo hay una implementación cargada de un método, lo devirtualiza e inlinea **aunque no sea `final`**. Y si más adelante se carga una segunda implementación, deshace la optimización. El resultado suele ser idéntico.
- Que una **clase `final` sea más rápida**. Mismo argumento.
- Que `final` en variables locales mejore el rendimiento. El JIT ya sabe si una local cambia; no necesita que se lo digas.
- Que `final` haga que el valor «se cargue en caché L1». Como se explicó en la sección [34](#34-la-semántica-de-campos-final-en-el-modelo-de-memoria), esa afirmación de una respuesta del hilo mezcla la garantía del lenguaje con detalles de arquitectura que no funcionan así.

**Conclusión:** usá `final` por corrección, por claridad y por la garantía de publicación segura. No por velocidad. El único caso donde el rendimiento entra de verdad en la decisión es el de los campos de confianza para el JIT, y ahí la forma de conseguirlo no es escribir `final` sino usar un `record`.

---

# Parte VIII — Diseño

## 40. final en parámetros: el debate

Poner `final` en todos los parámetros y variables locales es una práctica que divide a los equipos. Los dos argumentos, honestamente:

**A favor:**

- Impide reasignar el parámetro, que es una práctica confusa: el nombre deja de significar «lo que me pasaron».
- Documenta que dentro del método nada cambia.
- Algunos linters y estilos corporativos lo exigen.

**En contra:**

- Es **muchísimo ruido**. Un método con cuatro parámetros gana cuatro palabras que no cambian el comportamiento.
- La protección es mínima: reasignar un parámetro es un error que se ve leyendo el método, y los IDEs ya lo avisan.
- **No protege de lo que la gente cree que protege.** `final List<X> lista` no impide `lista.add(...)`, que es el cambio que de verdad importa.

**Mi recomendación, y lo que hace la mayoría del código moderno:** no lo pongas por defecto en parámetros ni en locales, salvo que el estilo del equipo lo exija. Reservá tu presupuesto de `final` para donde sí cambia las cosas: **los campos**.

La excepción sensata es la variable local que se asigna en varias ramas, donde `final` sí aporta porque hace que el compilador verifique que se cubrieron todos los caminos:

```java
final int codigo;
switch (estado) {
    case ACTIVO   -> codigo = 1;
    case INACTIVO -> codigo = 2;
    default       -> throw new IllegalStateException();
}
```

## 41. Campos final por defecto

Aquí sí hay una recomendación fuerte, y es la contraria a la timidez de Baeldung.

**Empezá haciendo `final` todos los campos, y quitá el `final` solo cuando tengas una razón concreta.**

```java
public class ServicioPedidos {
    private final RepositorioPedidos repositorio;      // final
    private final PasarelaPago pasarela;               // final
    private int pedidosProcesados;                     // mutable: es un contador
}
```

Las razones:

1. La mayoría de los campos son **dependencias o datos de identidad** que se fijan al construir y no cambian nunca.
2. Un campo `final` hace imposible el estado a medias.
3. Activa la publicación segura (sección [35](#35-publicación-segura-sin-sincronización)).
4. Al leer la clase, los pocos campos sin `final` señalan **exactamente dónde está el estado mutable**, que es donde hay que mirar cuando algo falla.

Este último punto es el más valioso en clases grandes: si de doce campos solo uno es mutable, la superficie de cambio queda documentada en una línea.

## 42. Records: final sin escribirlo

Un `record` aplica de golpe casi toda la receta de la sección [15](#15-cómo-se-construye-una-clase-realmente-inmutable):

```java
public record Dinero(long centimos, String divisa) { }
```

Eso genera una clase `final`, con campos `private final`, constructor, getters, `equals`, `hashCode` y `toString`. Comprobado por reflexión en la sección [31](#31-quién-es-final-sin-escribirlo): tanto la clase como sus campos son `final`.

Lo que un record **no** hace por vos es la inmutabilidad profunda:

```java
public record Equipo(String nombre, List<String> jugadores) { }
```

```java
List<String> lista = new ArrayList<>(List.of("Ana"));
Equipo e = new Equipo("Rojo", lista);
lista.add("Luis");
System.out.println(e.jugadores());     // [Ana, Luis]
```

El record guardó la referencia que le pasaron. La solución es un **constructor compacto** que copie:

```java
public record Equipo(String nombre, List<String> jugadores) {
    public Equipo {
        jugadores = List.copyOf(jugadores);      // sin this., sin return
    }
}
```

Esa sintaxis sin paréntesis de parámetros es el constructor compacto: se ejecuta antes de asignar los campos, y lo que dejes en las variables es lo que se asigna. Es el sitio canónico para validar y para copiar.

## 43. final y los frameworks

Un aspecto práctico que ninguna de las dos fuentes menciona y que te va a afectar en cuanto uses un framework.

**Mockito.** Hasta Mockito 5 no podía simular clases ni métodos `final` sin activar el *mock maker* en línea. Desde Mockito 5 está activo por defecto, pero si estás en un proyecto con Mockito 3 o 4 y marcás una clase `final`, los tests dejan de compilar con un error del estilo `Cannot mock/spy final class`. La solución es crear el fichero `src/test/resources/mockito-extensions/org.mockito.plugins.MockMaker` con la línea `mock-maker-inline`.

**Spring.** Los proxies de Spring —los que implementan `@Transactional`, `@Cacheable`, `@Async` y la seguridad de método— se crean por dos vías: interfaces (JDK dynamic proxy) o subclases (CGLIB). Si Spring usa CGLIB y tu clase o tu método son `final`, **no puede crear el proxy**:

- Clase `final` → `Cannot subclass final class`.
- Método `final` → no da error, pero **la anotación se ignora silenciosamente**, que es mucho peor. Tu `@Transactional` no abre ninguna transacción.

Es el mismo tipo de fallo que la autoinvocación de `@Transactional`: la anotación está escrita y no hace nada.

**JPA e Hibernate.** Requieren que las entidades no sean `final` y que sus getters tampoco lo sean, porque usan proxies para el *lazy loading*. Una entidad `final` fuerza la carga inmediata de todo, o directamente falla.

**Conclusión práctica:** en clases de dominio puro, `final` a discreción. En clases gestionadas por un framework —servicios de Spring, entidades JPA— comprobá primero qué necesita el framework. Y si un `@Transactional` no está funcionando, mirá si el método es `final`.

## 44. final y serialización

La serialización de Java construye objetos **sin llamar al constructor**: reserva el objeto y escribe los campos directamente, por un mecanismo interno equivalente a la reflexión. Eso significa que un campo `final` puede acabar con un valor que ningún constructor produjo.

Dos consecuencias que conviene tener presentes:

1. **Las invariantes que valida el constructor no se aplican al deserializar.** Si tu constructor comprueba que el importe no sea negativo, un flujo serializado manipulado puede crear un objeto con importe negativo. Por eso existe `readObject` con validación, y por eso *Effective Java* dedica un capítulo entero a la serialización.
2. **Es el caso que la JLS contempla explícitamente** al permitir modificar campos `final` (sección [37](#37-final-y-reflexión)), con la condición de que el objeto no se publique hasta que todas las actualizaciones terminen.

Los **records** mejoran esto de forma notable: su deserialización **sí pasa por el constructor canónico**, así que las validaciones del constructor compacto se aplican también al deserializar. Es una razón más para preferirlos como tipos de datos.

---

# Parte IX — Cierre

## 45. Casos de uso reales

**1. Campos de dependencia inyectados.**

```java
public class ServicioPedidos {
    private final RepositorioPedidos repositorio;

    public ServicioPedidos(RepositorioPedidos repositorio) {
        this.repositorio = repositorio;
    }
}
```

La inyección por constructor con campo `final` es el patrón estándar en Spring, y no por casualidad: garantiza que el objeto nunca existe sin sus dependencias.

**2. Clases de valor.**

```java
public record Coordenada(double latitud, double longitud) { }
```

**3. Constantes.**

```java
public static final Duration TIMEOUT = Duration.ofSeconds(30);
```

**4. Objetos compartidos entre hilos sin sincronización.**

```java
public final class TablaDeTarifas {
    private final Map<String, BigDecimal> tarifas;

    public TablaDeTarifas(Map<String, BigDecimal> origen) {
        this.tarifas = Map.copyOf(origen);
    }

    public BigDecimal tarifa(String codigo) { return tarifas.get(codigo); }
}
```

Se construye una vez al arrancar y se comparte con todos los hilos sin `synchronized`. Correcto **gracias a** la garantía de la sección [35](#35-publicación-segura-sin-sincronización).

**5. Métodos que sostienen un algoritmo (Template Method).** Sección [24](#24-por-qué-existen-el-contrato-que-se-rompe).

**6. Variables locales asignadas en varias ramas.** Sección [40](#40-final-en-parámetros-el-debate).

## 46. Anti-patrones

**Anti-patrón 1: la falsa constante.**

```java
public static final List<String> ROLES = new ArrayList<>();
```

Parece constante; es una variable global mutable compartida entre hilos. **Solución:** `List.of(...)`. Sección [13](#13-el-caso-de-la-colección).

**Anti-patrón 2: creer que `final` hace inmutable.**

```java
public final class Usuario {
    private String nombre;
    public void setNombre(String n) { this.nombre = n; }
}
```

La clase es `final` y perfectamente mutable. **Solución:** la receta completa de la sección [15](#15-cómo-se-construye-una-clase-realmente-inmutable).

**Anti-patrón 3: exponer la colección interna.**

```java
private final List<String> items = new ArrayList<>();
public List<String> getItems() { return items; }
```

El `final` no protege nada: cualquiera puede modificar la lista. **Solución:** copia defensiva o `List.copyOf`. Sección [17](#17-copias-defensivas).

**Anti-patrón 4: `this` escapando del constructor.**

```java
public Servicio(EventBus bus) {
    bus.suscribir(this);        // otro hilo puede verlo a medio construir
    this.config = cargarConfig();
}
```

Anula la garantía del modelo de memoria. **Solución:** publicar después de construir. Sección [36](#36-lo-que-rompe-la-garantía-this-escapando).

**Anti-patrón 5: llamar a un método sobrescribible desde el constructor.**

Produce campos `final` valiendo `null`. **Solución:** desde el constructor, solo métodos `private`, `static` o `final`. Sección [25](#25-llamar-a-métodos-desde-el-constructor).

**Anti-patrón 6: forzar campos `final` por reflexión.**

```java
Field f = Config.class.getDeclaredField("TIMEOUT");
f.setAccessible(true);
f.set(null, 5000);
```

Ya no funciona con `static final` desde Java 12, y el JIT puede haber cacheado el valor viejo. **Solución:** hacer el valor configurable en tiempo de ejecución en vez de hackearlo. Sección [38](#38-lo-que-ya-no-se-puede-modificar).

**Anti-patrón 7: `final` en clases gestionadas por un framework.**

Un método `@Transactional` marcado `final` **no abre ninguna transacción**, sin dar ningún error. **Solución:** conocer las restricciones del framework. Sección [43](#43-final-y-los-frameworks).

## 47. Checklist y tabla de decisión

**Checklist de revisión:**

- [ ] Los campos son `final` salvo los que necesitan cambiar, y esos están identificados
- [ ] Ninguna «constante» `static final` apunta a una colección mutable
- [ ] Las clases de valor son `final` (o `record`) con todos los campos `private final`
- [ ] Los constructores hacen copia defensiva de los parámetros mutables
- [ ] Los getters no devuelven colecciones ni arrays internos sin copiar
- [ ] Ningún constructor publica `this` antes de terminar
- [ ] Ningún constructor llama a métodos sobrescribibles
- [ ] Los métodos llamados desde constructores son `private`, `static` o `final`
- [ ] No hay código que modifique campos `final` por reflexión
- [ ] Las clases gestionadas por Spring o JPA no son `final` donde el framework necesita proxies
- [ ] Las variables locales asignadas en varias ramas son `final`

**Tabla de decisión:**

| Quiero… | Uso |
|---|---|
| Un campo que no cambia tras construir | `private final` |
| Una constante de verdad | `static final` con tipo inmutable |
| Un tipo de datos inmutable | `record`, o clase `final` con la receta completa |
| Una lista constante | `List.of(...)` |
| Compartir un objeto entre hilos sin sincronizar | Todos los campos `final` y sin fuga de `this` |
| Impedir que se altere un algoritmo | Método `final` |
| Cerrar una clase de valor | Clase `final` |
| Un conjunto cerrado de subtipos | `sealed` con `permits` |
| Que una lambda capture algo | Variable `final` o effectively final |
| Que una entidad JPA funcione | **No `final`** |
| Que `@Transactional` funcione | **No `final`** en la clase ni en el método |

## 48. Autoevaluación

Doce preguntas.

**1. ¿Por qué compila `foo.add("bar")` si `foo` es `final`?**

<details><summary>Respuesta</summary>

Porque `final` congela la **variable**, no el objeto. `foo` guarda una dirección de memoria y esa dirección no puede cambiar; el `ArrayList` que hay en esa dirección puede cambiar todo lo que quiera. Lo que fallaría es `foo = new ArrayList<>()`.

Como resume la respuesta aceptada del hilo: «`final` es solo sobre la referencia en sí, y no sobre el contenido del objeto referenciado». Sección [12](#12-qué-congela-final-exactamente).
</details>

**2. ¿Por qué `private static final List foo;` asignado en el constructor no compila?**

<details><summary>Respuesta</summary>

Porque un campo `static` es único para toda la clase y el constructor se ejecuta una vez **por objeto**. Dos `new` lo asignarían dos veces, que es lo que `final` prohíbe. El compilador lo rechaza:

```
error: cannot assign a value to static final variable foo
```

Se asigna en la declaración o en un bloque `static`. Y ojo: es un **error de compilación**, no un `NullPointerException` en ejecución como afirma una de las respuestas del hilo. Sección [8](#8-static-final-y-por-qué-el-constructor-no-vale).
</details>

**3. ¿Es inmutable esta clase?**

```java
public final class Equipo {
    private final List<String> jugadores;
    public Equipo(List<String> j) { this.jugadores = j; }
    public List<String> getJugadores() { return jugadores; }
}
```

<details><summary>Respuesta</summary>

No. Tiene dos fugas: quien construyó la lista conserva una referencia y puede modificarla, y el getter entrega la lista interna a cualquiera.

Es **inmutabilidad superficial**. La solución es `List.copyOf(j)` en el constructor. Secciones [16](#16-inmutabilidad-superficial-y-profunda) y [17](#17-copias-defensivas).
</details>

**4. ¿Qué imprime?**

```java
class Base { Base() { inicializar(); } protected void inicializar() {} }
class Derivada extends Base {
    private final String valor = "listo";
    @Override protected void inicializar() { System.out.println(valor); }
}
new Derivada();
```

<details><summary>Respuesta</summary>

Imprime **`null`**, aunque `valor` sea un campo `final` con inicializador.

El constructor de `Base` llama a `inicializar()`, que está sobrescrito, y en ese momento los inicializadores de campo de `Derivada` todavía no han corrido. El campo tiene su valor por defecto.

Por eso, desde un constructor, solo se debe llamar a métodos `private`, `static` o `final`. Sección [25](#25-llamar-a-métodos-desde-el-constructor).
</details>

**5. ¿Qué es effectively final y dónde aparece?**

<details><summary>Respuesta</summary>

Una variable que **podría llevar `final` sin romper la compilación**: se asigna una vez y nunca se reasigna, aunque no lleve la palabra.

Aparece al capturar variables locales desde lambdas y clases anónimas: `local variables referenced from a lambda expression must be final or effectively final`. La razón es que la lambda **copia** el valor, y si el original pudiera cambiar habría dos valores con el mismo nombre. Secciones [19](#19-qué-es-effectively-final) y [20](#20-por-qué-lo-exigen-las-lambdas).
</details>

**6. ¿Por qué esto compila y lo de al lado no?**

```java
for (String s : lista)      { tareas.add(() -> print(s)); }   // OK
for (int i = 0; i < 3; i++) { tareas.add(() -> print(i)); }   // error
```

<details><summary>Respuesta</summary>

Porque el `for-each` **declara una variable nueva en cada iteración**, así que cada `s` es effectively final. El `for` clásico reutiliza la misma `i` y la incrementa, así que no lo es. Sección [21](#21-la-regla-exacta-y-sus-sorpresas).
</details>

**7. ¿Qué garantía da `final` que no da ninguna otra palabra del lenguaje?**

<details><summary>Respuesta</summary>

La **publicación segura sin sincronización**. Si todos los campos de un objeto son `final` y se asignan en el constructor sin que `this` escape, cualquier hilo que vea la referencia tras terminar el constructor ve los valores correctos —y todo lo alcanzable desde ellos— sin `synchronized`, sin `volatile` y sin coste.

Es la JLS §17.5 y la acción de *freeze* al final del constructor. Sin `final`, otro hilo puede ver la referencia no nula y los campos a cero. Sección [35](#35-publicación-segura-sin-sincronización).
</details>

**8. ¿Sigue funcionando la reflexión para cambiar un campo `final`?**

<details><summary>Respuesta</summary>

Solo para campos **de instancia en clases normales**. Comprobado en JDK 25:

- `final` de instancia → se modifica.
- `static final` → `IllegalAccessException`.
- Componente de un `record` → `IllegalAccessException`.

El truco clásico de quitar el bit `final` vía el campo `modifiers` de `Field` no funciona desde Java 12. Y aunque el caso de instancia funcione, la JLS advierte de que el resultado es impredecible si el objeto ya circula. Secciones [37](#37-final-y-reflexión) y [38](#38-lo-que-ya-no-se-puede-modificar).
</details>

**9. ¿Un método `final` es más rápido?**

<details><summary>Respuesta</summary>

En la práctica, no. HotSpot usa *Class Hierarchy Analysis*: si solo hay una implementación cargada, devirtualiza e inlinea aunque el método no sea `final`, y deshace la optimización si aparece otra.

Lo que sí es real: las expresiones constantes `final` se pliegan en compilación, y los campos `static final` y los de `record` son de confianza para el JIT desde que dejaron de ser modificables por reflexión. Sección [39](#39-rendimiento-el-mito-y-lo-real).
</details>

**10. ¿Por qué `String` es `final`?**

<details><summary>Respuesta</summary>

Por tres razones: **seguridad** (una subclase podría devolver una ruta en la validación y otra al usarla, un ataque TOCTOU), **el pool de cadenas** (los literales se comparten y solo es seguro si nadie los altera) y **el hash cacheado** (`String` guarda su `hashCode`, que quedaría obsoleto si el contenido cambiara). Sección [29](#29-por-qué-string-es-final).
</details>

**11. ¿Qué aporta `sealed` frente a `final`?**

<details><summary>Respuesta</summary>

Una opción intermedia: la jerarquía se abre **solo a una lista concreta** de subtipos declarada con `permits`. Como el compilador sabe que la lista está completa, un `switch` sobre esos tipos es exhaustivo y **no compila** si alguien añade un subtipo y olvida cubrirlo. Con `final` no hay jerarquía; con una clase abierta no hay exhaustividad. Sección [32](#32-sealed-la-tercera-opción-desde-java-17).
</details>

**12. Tu `@Transactional` no abre ninguna transacción y no hay ningún error. ¿Qué mirarías?**

<details><summary>Respuesta</summary>

Si el método es `final`. Spring implementa `@Transactional` con un proxy CGLIB que subclasea tu clase y sobrescribe el método; si el método es `final` no puede sobrescribirlo y **la anotación se ignora en silencio**. Si es la clase la que es `final`, al menos falla con `Cannot subclass final class`.

Lo mismo vale para `@Cacheable`, `@Async` y la seguridad de método. Sección [43](#43-final-y-los-frameworks).
</details>

## 49. Fuentes

Las fuentes se listan con lo que aportan y **con sus errores señalados**. Todo lo marcado como error se comprobó en JDK 25 (Temurin 25.0.3+9).

### Fuentes primarias

- **[Java Language Specification — §17.5, *final Field Semantics*](https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html)**. La sección que sostiene toda la Parte VII y que ninguna de las dos fuentes del encargo menciona. Define la acción de **freeze** al final del constructor, la garantía de visibilidad entre hilos, su transitividad, la condición de que `this` no escape, y la semántica de modificar un campo `final` por reflexión.
- **JLS §4.12.4, *final Variables*** y **§16, *Definite Assignment*.** La regla de asignación definida de la sección 9, que es la verdadera norma detrás de los blank finals.
- **[Field (Java SE 17 API)](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/reflect/Field.html)**. La lista literal de campos no modificables por reflexión: `static final` de cualquier clase, `final` de *hidden class* y `final` de `record`. Fuente de la sección 38.
- **[JDK-8247517: Final fields in records are not reflectively modifiable](https://bugs.openjdk.org/browse/JDK-8247517)**. Explica el intercambio: se prohíbe modificarlos para que **el JIT pueda confiar en ellos**. La frase «Final fields are not trusted because they are not truly final» resume el motivo del endurecimiento.
- **[JEP 395: Records](https://openjdk.org/jeps/395)** y **[JEP 409: Sealed Classes](https://openjdk.org/jeps/409)**. Los dos tipos que aplican `final` implícitamente y la alternativa intermedia de la sección 32.

### Fuentes que se pidieron para este capítulo

- **[Baeldung — The "final" Keyword in Java](https://www.baeldung.com/java-final)**. Buena estructura y buen ritmo: cubre clases, métodos y variables en ese orden, y acierta en dos cosas que muchos textos se saltan — la advertencia de que una clase `final` **no** es inmutable, y la crítica honesta al coste de cerrar una clase («no other programmer can improve it»). El ejemplo de `Thread.isAlive()` como método `final` nativo es el mejor de su categoría. Aporta también un criterio útil para distinguir constante de campo *write-once*: preguntarse si el campo se incluiría al serializar. **Cuatro problemas:**
  - **Los mensajes de error no son de `javac`.** Los cuatro que publica —`The final local variable i may already have been assigned`, `The final local variable cat cannot be assigned. It must be blank and not using a compound assignment`, `The type BlackCat cannot subclass the final class Cat` y el del parámetro— son de **Eclipse JDT**. `javac` 25 dice `cannot assign a value to final variable i`, `cannot inherit from final Gato` y `final parameter x may not be assigned`. Comprobado y contrastado en las secciones 3, 5, 23 y 28.
  - **Imprecisión sobre los campos `final`:** «any final field must be initialized before the constructor completes». Vale para los de instancia; para los `static final` es engañoso, porque se inicializan en el `<clinit>` de la clase, **antes** de que exista ningún objeto y sin relación con ningún constructor. De hecho asignarlos desde el constructor es un error de compilación (sección 8).
  - **Sección truncada:** en 4.3 anuncia «For static final fields, this means that we can initialize them:» y «For instance final fields, this means that we can initialize them:» **sin poner ningún ejemplo detrás** en ninguno de los dos casos.
  - **No menciona en absoluto** la semántica de campos `final` del modelo de memoria, que es la garantía más valiosa de la palabra clave; ni `effectively final`; ni las restricciones de reflexión; ni `sealed`; ni la interacción con frameworks. Su conclusión —«may not use the final keyword often in our internal code»— desaconseja de hecho lo que la industria recomienda desde *Effective Java*.
- **[Stack Overflow — How does the "final" keyword in Java work? (I can still modify an object.)](https://stackoverflow.com/questions/15655012/how-does-the-final-keyword-in-java-work-i-can-still-modify-an-object)**. 584 votos, 610.000 visitas y 20 respuestas. Es la mejor fuente que existe sobre la confusión central del tema, y varias respuestas son excelentes:
  - **La aceptada (Marko Topolnik)** da la formulación más limpia: «`final` is only about the reference itself, and not about the contents of the referenced object», y añade el punto que casi nadie dice: «Java has no concept of object immutability; this is achieved by carefully designing the object, and is a far-from-trivial endeavor».
  - **La más votada (AmitG, 668 votos)** explica por qué el constructor sirve para un `final` de instancia y no para un `static final`, con el argumento de «cuántas veces se puede invocar cada cosa». Es el razonamiento de la sección 8. Añade además una introducción a `effectively final`.
  - **BambooleanLogic** separa con precisión el comportamiento en tipos valor y tipos referencia.
  - **La respuesta «hold my beer»** demuestra la modificación de un campo `final` por reflexión, con bytecode incluido. **Parcialmente obsoleta:** en JDK 25 eso solo funciona con campos de instancia; `static final` y componentes de `record` lanzan `IllegalAccessException` (sección 37).
  - **Dos respuestas contienen errores** que este documento señala: una afirma que el caso `static final` produciría un `NullPointerException` en ejecución cuando es **error de compilación** (sección 8); otra sostiene que los valores `final` «se cargan en la caché L1» mientras los no finales van a L2, lo que mezcla una garantía del lenguaje con detalles de arquitectura y no es lo que dice la JLS, que habla de mantener el valor **en un registro** (sección 34).

### Discusiones y referencias de comunidad consultadas

- **[Safe Publication and Safe Initialization in Java](https://shipilev.net/blog/2014/safe-public-construction/)** (Aleksey Shipilev, ingeniero de la JVM). La referencia práctica sobre publicación segura: enumera las cuatro formas de conseguirla —bloqueo, `volatile`, atómicas y **campos `final`**— y explica el detalle de implementación de que HotSpot emite la barrera si se escribió **al menos un** campo `final`, extendiendo de hecho la garantía a los demás campos del constructor. Suya es también la advertencia de que «si alguien vio la instancia antes, todas las apuestas están canceladas».
- **[JMM and final field freeze](https://puredanger.github.io/tech.puredanger.com/2008/11/26/jmm-and-final-field-freeze/)** (Alex Miller). Reúne la formulación del JSR-133 FAQ y la cita de *Java Concurrency in Practice* §16.3 sobre la transitividad de la garantía, que es la que se usa en la sección 35.
- **[The Java Memory Model — final field semantics](https://www.cs.umd.edu/~pugh/java/memoryModel/newFinal.pdf)** (Jeremy Manson, William Pugh). El artículo académico que define la semántica. De aquí sale la formulación informal de la congelación y de las *dereference chains*.
- **[IllegalAccessException when trying to modify private static field](https://stackoverflow.com/questions/68404913/illegalaccessexception-when-trying-to-modify-private-static-field)** y **[java 17 reflection issue](https://stackoverflow.com/questions/74723932/java-17-reflection-issue)**. Los dos hilos que documentan que el truco del campo `modifiers` dejó de funcionar, con la referencia al cambio de Java 12 y los rodeos con `--add-opens` y `VarHandle` que tampoco sobreviven a Java 18.
- **[java 17, ReflectionHelpers.setStaticField does not work on a final field](https://stackoverflow.com/questions/76789468/java-17-the-reflectionhelpers-setstaticfield-does-not-work-on-a-final-field)**. Caso real de migración: un test que forzaba `BuildConfig.DEBUG` y dejó de funcionar. Ilustra la consecuencia práctica de la sección 38.
- **Joshua Bloch, *Effective Java*.** Ítem «Minimize mutability» para la receta de la sección 15, ítem «Make defensive copies when needed» para la 17, e ítem «Design and document for inheritance or else prohibit it» para la 30.

### Verificación

Todos los ejemplos, salidas y mensajes de error de este documento se ejecutaron en:

```
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9 (build 25.0.3+9-LTS)
```

Los mensajes de `javac` se reproducen literalmente. Se compilaron nueve ficheros: para capturar el texto exacto de los errores (reasignar una local y una referencia `final`, asignar un `static final` desde el constructor, reasignar un parámetro `final`, heredar de una clase `final`, sobrescribir un método `final`, y capturar una variable no *effectively final* desde una lambda) y para comprobar comportamiento en ejecución (mutación de un objeto tras una referencia `final`, modificación por reflexión de cuatro tipos de campo `final`, y qué tipos son `final` implícitamente según `Modifier.isFinal`).

El bytecode se inspeccionó con `javap -c` para demostrar el plegado de constantes de una variable local `final`: con `final` el compilador emite un único `ldc` con la cadena ya concatenada; sin `final`, un `iload` seguido de `invokedynamic makeConcatWithConstants`.
