# Static Keyword

> **Bloque:** `02-POO` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25

**Alcance de este documento.** Los tres capítulos anteriores construyeron la clase ([Classes and Objects](01-classes-and-objects.md)), sus piezas ([Attributes and Methods](02-attributes-and-methods.md)) y quién puede verlas ([Access Specifiers](03-access-specifiers.md)). Los tres asumieron algo sin decirlo: que cada objeto tiene su propia copia de cada campo. Este capítulo cubre entero **el mecanismo que rompe esa regla**: `static`, la palabra que mueve un miembro desde los objetos hacia la clase.

Es un capítulo mucho más grande de lo que su nombre sugiere. `static` no es un modificador más: es la puerta de entrada al **modelo de inicialización de clases de la JVM**, que es donde viven algunos de los bugs más difíciles de diagnosticar de Java. Un bloque `static` que falla deja la clase inservible para el resto de la vida del proceso. Dos clases que se referencian entre sí en sus inicializadores pueden bloquear dos hilos en un deadlock que **el detector de deadlocks de la JVM no ve**. Y una constante `static final` puede hacer que un cambio de valor no llegue nunca a producción, porque el compilador la copió dentro de otro `.class` hace tres meses.

**Por qué este tema separa a un junior de un senior.** `static` es fácil de escribir y difícil de deshacer. Un campo `static` mutable es estado global: sobrevive a los tests, se comparte entre hilos sin sincronización, no se puede sustituir por un doble de prueba y mantiene vivos objetos que el recolector no puede liberar. La mayoría de los desarrolladores aprenden `static` como «lo que hay que poner delante de `main`» y lo siguen usando así durante años. Este documento explica qué significa de verdad, qué hace la JVM por debajo, y —más importante— **cuándo no usarlo**.

Todo lo que se afirma aquí está comprobado ejecutándolo en JDK 25. Cuando una de las fuentes del capítulo dice algo que no se sostiene al compilarlo, se señala con el mensaje literal de `javac` o la salida real del programa.

**Lo que NO entra aquí**, porque tiene documento propio: `final` en profundidad (`05`), los bloques de inicialización de instancia y el ciclo de vida del objeto (`06`), la encapsulación como principio (`08`), la herencia (`09`), el polimorfismo y el binding (`11`), las clases anidadas en detalle (`15`), los records (`16`) y los enums (`17`). Aquí aparecen solo en lo que toca a `static`.

---

## Índice

**Parte I — Qué significa static**

1. [El problema que resuelve static](#1-el-problema-que-resuelve-static)
2. [Clase frente a instancia: el modelo mental](#2-clase-frente-a-instancia-el-modelo-mental)
3. [Dónde se puede escribir static](#3-dónde-se-puede-escribir-static)
4. [Por qué se llama static](#4-por-qué-se-llama-static)

**Parte II — Campos static**

5. [Campo de instancia frente a campo de clase](#5-campo-de-instancia-frente-a-campo-de-clase)
6. [El contador compartido y su trampa](#6-el-contador-compartido-y-su-trampa)
7. [Cómo acceder a un campo static](#7-cómo-acceder-a-un-campo-static)
8. [static final: la constante](#8-static-final-la-constante)
9. [La constante que se calcula: blank static final](#9-la-constante-que-se-calcula-blank-static-final)
10. [Dónde vive realmente un campo static](#10-dónde-vive-realmente-un-campo-static)
11. [Valores por defecto y el instante en que existen](#11-valores-por-defecto-y-el-instante-en-que-existen)

**Parte III — Métodos static**

12. [Qué es un método static](#12-qué-es-un-método-static)
13. [Las cuatro combinaciones de acceso](#13-las-cuatro-combinaciones-de-acceso)
14. [Por qué un método static no tiene this](#14-por-qué-un-método-static-no-tiene-this)
15. [El error non-static cannot be referenced from a static context](#15-el-error-non-static-cannot-be-referenced-from-a-static-context)
16. [Llamar a un método static sobre una referencia nula](#16-llamar-a-un-método-static-sobre-una-referencia-nula)
17. [Por qué main es static](#17-por-qué-main-es-static)

**Parte IV — Inicialización, el corazón del tema**

18. [Las tres fases: carga, enlace e inicialización](#18-las-tres-fases-carga-enlace-e-inicialización)
19. [clinit, el método que no escribiste](#19-clinit-el-método-que-no-escribiste)
20. [Bloques de inicialización static](#20-bloques-de-inicialización-static)
21. [El orden textual manda](#21-el-orden-textual-manda)
22. [Cuándo se inicializa una clase de verdad](#22-cuándo-se-inicializa-una-clase-de-verdad)
23. [Las constantes que no cargan la clase](#23-las-constantes-que-no-cargan-la-clase)
24. [La trampa del despliegue parcial](#24-la-trampa-del-despliegue-parcial)
25. [El orden en la jerarquía de herencia](#25-el-orden-en-la-jerarquía-de-herencia)
26. [Cuando la inicialización falla](#26-cuando-la-inicialización-falla)
27. [La clase en estado erróneo](#27-la-clase-en-estado-erróneo)
28. [Deadlock de inicialización](#28-deadlock-de-inicialización)
29. [Ciclos de inicialización sin deadlock](#29-ciclos-de-inicialización-sin-deadlock)

**Parte V — Clases anidadas static**

30. [Anidada static frente a interna](#30-anidada-static-frente-a-interna)
31. [La referencia oculta al objeto externo](#31-la-referencia-oculta-al-objeto-externo)
32. [Qué ve una clase anidada static](#32-qué-ve-una-clase-anidada-static)
33. [El idiom del holder: singleton perezoso](#33-el-idiom-del-holder-singleton-perezoso)
34. [Builder, el uso más frecuente](#34-builder-el-uso-más-frecuente)
35. [static dentro de una inner class desde Java 16](#35-static-dentro-de-una-inner-class-desde-java-16)

**Parte VI — static y herencia**

36. [Se heredan pero no se sobrescriben](#36-se-heredan-pero-no-se-sobrescriben)
37. [Hiding: qué versión se ejecuta](#37-hiding-qué-versión-se-ejecuta)
38. [Los campos también se ocultan](#38-los-campos-también-se-ocultan)
39. [Las reglas que el compilador sí aplica](#39-las-reglas-que-el-compilador-sí-aplica)
40. [Los métodos static de una interfaz no se heredan](#40-los-métodos-static-de-una-interfaz-no-se-heredan)
41. [static y genéricos](#41-static-y-genéricos)

**Parte VII — Otros usos de static**

42. [import static](#42-import-static)
43. [Clases de utilidad](#43-clases-de-utilidad)
44. [Factorías static](#44-factorías-static)
45. [static en enums y records](#45-static-en-enums-y-records)

**Parte VIII — Diseño, memoria y riesgos**

46. [static como raíz del recolector](#46-static-como-raíz-del-recolector)
47. [Fugas de memoria por campos static](#47-fugas-de-memoria-por-campos-static)
48. [static y concurrencia](#48-static-y-concurrencia)
49. [static y testabilidad](#49-static-y-testabilidad)
50. [static frente a inyección de dependencias](#50-static-frente-a-inyección-de-dependencias)
51. [Rendimiento: qué cambia de verdad](#51-rendimiento-qué-cambia-de-verdad)

**Parte IX — Cierre**

52. [Casos de uso reales](#52-casos-de-uso-reales)
53. [Anti-patrones](#53-anti-patrones)
54. [Checklist y tabla de decisión](#54-checklist-y-tabla-de-decisión)
55. [Autoevaluación](#55-autoevaluación)
56. [Fuentes](#56-fuentes)

---

# Parte I — Qué significa static

## 1. El problema que resuelve static

Antes de la palabra clave, el problema.

Hasta ahora, cada vez que escribías `new`, Java te daba un objeto nuevo con su propio juego de campos. Dos objetos, dos copias:

```java
class Empleado {
    String nombre;
    int salario;
}
```

```java
Empleado ana = new Empleado();
ana.nombre = "Ana";

Empleado luis = new Empleado();
luis.nombre = "Luis";
```

`ana.nombre` y `luis.nombre` son dos huecos de memoria distintos. Cambiar uno no toca el otro. Eso es exactamente lo que querés el 90 % del tiempo: cada empleado tiene su nombre.

Pero hay datos que **no pertenecen a ningún empleado en particular**:

- ¿Cuántos empleados se han creado en total?
- ¿Cuál es el salario mínimo legal, que es el mismo para todos?
- ¿Dónde está el logger que usan todos los empleados para escribir trazas?

Si guardás el salario mínimo como campo de instancia, cada empleado carga con una copia idéntica del mismo número. Con diez mil empleados, diez mil copias del mismo valor. Y peor: si mañana cambia el salario mínimo, hay que recorrer los diez mil objetos para actualizarlos.

Y hay un problema todavía más básico. ¿Dónde ponés el contador de empleados creados? No puede vivir dentro de un empleado, porque cada empleado tendría su propio contador y todos valdrían 1.

**`static` resuelve exactamente esto.** Marca un miembro como perteneciente a la clase y no a los objetos. Existe una sola copia, se crea antes de que exista el primer objeto, y sigue existiendo aunque no crees ninguno.

```java
class Empleado {
    static int totalCreados = 0;        // uno solo, de la clase
    static final int SALARIO_MINIMO = 1134;

    String nombre;                       // uno por objeto
    int salario;

    Empleado(String nombre) {
        this.nombre = nombre;
        totalCreados++;
    }
}
```

```java
new Empleado("Ana");
new Empleado("Luis");
System.out.println(Empleado.totalCreados);   // 2
```

Fijate en la última línea: se lee `Empleado.totalCreados`, con el **nombre de la clase**. No hace falta ningún objeto. Ese es el cambio conceptual entero.

## 2. Clase frente a instancia: el modelo mental

La imagen que hay que fijar es esta: **una clase es también una cosa que existe en tiempo de ejecución**, no solo un molde.

Cuando la JVM carga `Empleado`, crea en memoria un objeto que representa a la clase misma —el que devuelve `Empleado.class`— y **ahí es donde viven los campos `static`**. Los campos de instancia viven en cada objeto creado con `new`.

```
                    ┌──────────────────────────────┐
                    │  la clase Empleado           │
                    │  totalCreados = 2            │   ← campos static: una copia
                    │  SALARIO_MINIMO = 1134       │
                    └──────────────────────────────┘
                             ▲            ▲
                             │            │
              ┌──────────────┘            └──────────────┐
    ┌─────────────────────┐                    ┌─────────────────────┐
    │  objeto 1           │                    │  objeto 2           │
    │  nombre = "Ana"     │                    │  nombre = "Luis"    │
    │  salario = 2000     │                    │  salario = 2300     │
    └─────────────────────┘                    └─────────────────────┘
         campos de instancia: una copia por objeto
```

De esta imagen se deduce, sin memorizar nada, casi todo lo demás:

- Un método `static` **no puede** leer `nombre` directamente: está en la caja de abajo, y puede haber cero o mil cajas. ¿Cuál leería?
- Un método de instancia **sí puede** leer `totalCreados`: está en la caja de arriba, que es única y siempre está.
- Los campos `static` existen **antes** que cualquier objeto, porque la caja de arriba se crea al cargar la clase.
- Si el campo `static` guarda una referencia a un objeto grande, ese objeto **no se puede recolectar** mientras la clase esté cargada. Esto es la sección [46](#46-static-como-raíz-del-recolector) y es la causa de una familia entera de fugas de memoria.

Las tres fuentes de este capítulo coinciden en esta definición. freeCodeCamp: «When you declare a variable or a method as static, it belongs to the class, rather than a specific instance». Jenkov: «A static field belongs to the class. Thus, no matter how many objects you create of that class, there will only exist one field located in the class». Baeldung: «exactly a single copy of that field is created and shared among all instances of that class». Las tres dicen lo mismo y las tres tienen razón. A partir de aquí es donde empiezan a divergir de la realidad.

## 3. Dónde se puede escribir static

`static` se puede aplicar a cuatro cosas. Las cuatro tienen su parte en este documento:

| Se aplica a | Qué significa | Sección |
|---|---|---|
| **Campos** | Un solo valor compartido por la clase entera | [Parte II](#5-campo-de-instancia-frente-a-campo-de-clase) |
| **Métodos** | Se invoca sin objeto, no tiene `this` | [Parte III](#12-qué-es-un-método-static) |
| **Bloques de inicialización** | Código que corre una vez, al inicializar la clase | [20](#20-bloques-de-inicialización-static) |
| **Clases anidadas** | Clase interna sin vínculo con un objeto externo | [Parte V](#30-anidada-static-frente-a-interna) |

Y hay un quinto uso, que no es un modificador sino una forma de `import`: **`import static`**, que trae nombres static de otra clase al fichero y se cubre en la sección [42](#42-import-static).

Igual de importante es saber **dónde no se puede escribir**:

```java
public class NoSePuede {
    void metodo() {
        static int x = 5;          // error: no existe una variable local static
    }

    static class Anidada { }       // OK: clase anidada

    // class NivelSuperior { }     // una clase de nivel superior nunca es static
}
```

```
NoSePuede.java:3: error: illegal start of expression
        static int x = 5;
        ^
```

Las **variables locales nunca son `static`**. Java no tiene el equivalente de la variable local estática de C. Si necesitás que un método recuerde algo entre llamadas, ese algo es un campo, no una variable local.

Y una **clase de nivel superior tampoco puede ser `static`**, porque `static` significa «no depende de una instancia de la clase que la contiene» y una clase de nivel superior no está contenida en ninguna. El compilador lo rechaza con `modifier static not allowed here`.

## 4. Por qué se llama static

El nombre viene de C, donde `static` significaba «con duración de almacenamiento estática»: una variable que existe durante toda la ejecución del programa, no solo mientras dura una llamada. Java heredó la palabra y le dio un significado emparentado pero distinto.

Hay una lectura que ayuda mucho más a recordar el comportamiento, y que aparece bien formulada en una respuesta muy votada de Stack Overflow sobre por qué no se pueden sobrescribir los métodos `static`:

> «la palabra *static* es un antónimo de *dinámico*. Y la razón por la que no podés sobrescribir métodos static es que no hay despacho dinámico sobre miembros static: porque *static* significa literalmente *no dinámico*».

Esa lectura predice el comportamiento de la [Parte VI](#36-se-heredan-pero-no-se-sobrescriben) completa: si `static` quiere decir «se resuelve en tiempo de compilación, mirando el tipo escrito y no el objeto real», entonces no puede haber polimorfismo. Y en efecto no lo hay.

---

# Parte II — Campos static

## 5. Campo de instancia frente a campo de clase

La terminología formal, que conviene usar bien porque aparece en la especificación y en las entrevistas:

- Un campo `static` se llama **variable de clase** (*class variable*).
- Un campo no `static` se llama **variable de instancia** (*instance variable*).

Jenkov da la sintaxis completa de una declaración de campo, y es una buena referencia para memorizar el orden:

```
[modificador_de_acceso] [static] [final] tipo nombre [= valor inicial] ;
```

Solo el tipo y el nombre son obligatorios. En la práctica:

```java
public class Cliente {
    public    static final String PAIS_DEFECTO = "ES";   // constante de clase
    private   static int         totalClientes;          // variable de clase mutable
              static String      cachePlantilla;         // variable de clase, acceso de paquete

    private   String  email;                             // variable de instancia
    protected int     descuento;
}
```

La diferencia práctica se ve en dos líneas:

```java
Cliente.totalClientes = 10;              // campo static: se accede por la clase

Cliente c = new Cliente();               // campo de instancia: hace falta un objeto
c.email = "a@b.com";
```

## 6. El contador compartido y su trampa

El ejemplo canónico —lo usan las tres fuentes— es un contador de objetos creados. freeCodeCamp lo escribe así:

```java
public class Counter {
    public static int COUNT = 0;
    Counter() {
        COUNT++;
    }
}
```

```java
Counter c1 = new Counter();
Counter c2 = new Counter();
System.out.println(Counter.COUNT);       // 2
```

Baeldung usa la misma idea con un coche:

```java
public class Car {
    private String name;
    private String engine;

    public static int numberOfCars;

    public Car(String name, String engine) {
        this.name = name;
        this.engine = engine;
        numberOfCars++;
    }
}
```

El ejemplo funciona y es didáctico. Pero tal como está escrito **tiene tres problemas reales** que ninguna de las dos fuentes menciona, y que en código de producción son bugs:

**Problema 1: el campo es público y mutable.** `Counter.COUNT = 9999;` es código legal desde cualquier parte del programa. El contador no cuenta nada: es una variable global que cualquiera puede pisar. Lo correcto es `private` con un getter:

```java
public class Counter {
    private static int count = 0;

    Counter() { count++; }

    public static int getCount() { return count; }
}
```

**Problema 2: el nombre en mayúsculas engaña.** La convención `MAYUSCULAS_CON_GUION_BAJO` está reservada para constantes, es decir `static final`. Un campo `static` mutable llamado `COUNT` le dice al lector «esto no cambia», y cambia en cada constructor. Se nombra `count`, en `camelCase`.

**Problema 3, el grave: no es seguro entre hilos.** `count++` no es una operación atómica. Son tres pasos —leer, sumar uno, escribir— y dos hilos pueden intercalarse y perder incrementos. Con dos objetos en un solo hilo no se nota nunca; con dos hilos creando objetos en un servidor, sí. La versión correcta:

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Counter {
    private static final AtomicInteger COUNT = new AtomicInteger();

    Counter() { COUNT.incrementAndGet(); }

    public static int getCount() { return COUNT.get(); }
}
```

Fijate en el detalle: ahora el campo **sí** es `static final` y **sí** va en mayúsculas, porque la referencia no cambia; lo que cambia es el número de dentro del `AtomicInteger`. Esto está desarrollado en la sección [48](#48-static-y-concurrencia).

## 7. Cómo acceder a un campo static

Hay dos formas de leer un campo `static`, y solo una es correcta:

```java
Counter.getCount();          // por el nombre de la clase   -- correcto

Counter c = new Counter();
c.getCount();                // por una instancia           -- legal, pero confuso
```

Las dos compilan. Baeldung lo dice bien: «static fields can be accessed through an instance (e.g. `ford.numberOfCars++`) or directly from the class (e.g. `Car.numberOfCars++`). The latter is preferred, as it clearly indicates that it's a class variable rather than an instance variable». freeCodeCamp también lo menciona: «You can also access the static variable using any object of that class, such as `c1.COUNT`».

Lo que ninguna de las dos dice es **por qué el acceso por instancia es tan peligroso**. Este código lo explica solo:

```java
Contador a = new Contador();
Contador b = new Contador();

a.total = 5;
System.out.println(b.total);      // imprime 5
```

Un lector que no sepa que `total` es `static` leerá «pongo el total de `a` a 5» y verá salir un 5 al preguntar por `b`. La sintaxis miente sobre lo que pasa. Es tan reconocidamente malo que los IDEs y linters lo marcan por defecto: IntelliJ lo señala como *«Static member accessed via instance reference»* y Checkstyle tiene una regla dedicada.

Y llega al extremo de que **la referencia puede ser nula y funcionar igual**, que es la sección [16](#16-llamar-a-un-método-static-sobre-una-referencia-nula).

## 8. static final: la constante

La combinación `static final` es la más frecuente de todas y la única que casi nunca es discutible:

```java
public class Fisica {
    public static final double GRAVEDAD = 9.80665;
    public static final int    DIAS_SEMANA = 7;
    public static final String UNIDAD = "m/s2";
}
```

`static` porque el valor es el mismo para todo el programa; `final` porque no cambia nunca. Jenkov lo explica con precisión: «A field declared `static` and `final` is also called a *constant*», y añade la convención de nombres, en mayúsculas separadas por guion bajo.

Hay una trampa clásica con `final` que conviene desactivar ya: **`final` congela la referencia, no el objeto**.

```java
public class Config {
    public static final List<String> ROLES = new ArrayList<>();
}
```

```java
Config.ROLES.add("ADMIN");           // compila y funciona: la lista cambia
Config.ROLES = new ArrayList<>();    // error: cannot assign a value to final variable
```

```
Config.java:5: error: cannot assign a value to final variable ROLES
        Config.ROLES = new ArrayList<>();
              ^
```

La constante `ROLES` no se puede reapuntar, pero la lista a la que apunta es perfectamente mutable, y como es `public static` cualquier código del programa puede añadirle elementos. Es una variable global disfrazada de constante. La forma correcta:

```java
public static final List<String> ROLES = List.of("ADMIN", "USER");
```

`List.of` devuelve una lista inmutable: cualquier intento de `add` lanza `UnsupportedOperationException` en tiempo de ejecución. Solo con esa línea la constante es de verdad constante.

## 9. La constante que se calcula: blank static final

Jenkov afirma, en la página que este capítulo usa como fuente:

> «A `final` field cannot have its value changed. **A final field must have an initial value assigned to it**, and once set, the value cannot be changed again.»

**La segunda frase es falsa**, y es el mismo error que ya apareció en el capítulo [Attributes and Methods](02-attributes-and-methods.md). Un campo `final` puede declararse **sin valor** —se llama *blank final*— siempre que se le asigne exactamente una vez: en el constructor si es de instancia, en un bloque `static` si es de clase.

Comprobación en JDK 25:

```java
class ConBlank {
    static final int CALCULADO;          // blank static final: SIN inicializador
    final int porInstancia;              // blank final de instancia

    static { CALCULADO = 6 * 7; }

    ConBlank(int v) { this.porInstancia = v; }
}
```

```java
System.out.println(ConBlank.CALCULADO);            // 42
System.out.println(new ConBlank(9).porInstancia);  // 9
```

```
blank static final = 42
blank final instancia = 9
```

Compila y funciona. Esto no es una curiosidad académica: es **el mecanismo normal** para una constante cuyo valor no se puede escribir como literal. freeCodeCamp usa exactamente esta forma en su ejemplo de bloque `static`, aunque sin nombrarla:

```java
public class Saturn {
  public static final int MOON_COUNT;

  static {
    MOON_COUNT = 62;
  }
}
```

Ese `MOON_COUNT` sin inicializador es un blank static final, y compila precisamente porque la regla de Jenkov no es la real. La regla verdadera, en la JLS, es la de **asignación definida**: el compilador exige poder demostrar que la variable se asigna una vez y solo una antes de usarse. Si se asigna dos veces:

```java
class Doble {
    static final int X;
    static { X = 1; X = 2; }
}
```

```
Doble.java:3: error: variable X might already have been assigned
    static { X = 1; X = 2; }
                    ^
```

## 10. Dónde vive realmente un campo static

Baeldung afirma: «From the memory perspective, static variables are stored in the heap memory». La frase es correcta **hoy**, pero necesita dos matices que la fuente no da, y ambos importan si alguna vez vas a diagnosticar un problema de memoria.

**Matiz histórico.** Hasta Java 7 incluido, los campos `static` vivían en el **PermGen** (*Permanent Generation*), un área de memoria separada del heap con tamaño fijo. El error `java.lang.OutOfMemoryError: PermGen space` era pan de cada día en servidores de aplicaciones. Java 8 eliminó el PermGen (fue la [JEP 122](https://openjdk.org/jeps/122)): los metadatos de las clases pasaron al **Metaspace**, que crece dinámicamente fuera del heap, y las **variables static se movieron al objeto `java.lang.Class` de la clase**, que sí está en el heap. Por eso hoy la frase de Baeldung es cierta, y por eso en cualquier texto anterior a 2014 leerás lo contrario. Si mantenés una aplicación vieja, el flag `-XX:MaxPermSize` que veas en su arranque es de esa época y en Java 8+ se ignora.

**Matiz que importa más.** La distinción relevante no es *heap o no heap*, sino **qué es lo que se almacena**. Lo que vive con la clase es la **variable**, es decir el hueco de 4 u 8 bytes que guarda un `int` o una referencia. El **objeto apuntado** siempre estuvo en el heap normal, junto a todos los demás:

```java
class Cache {
    static Map<String, byte[]> datos = new HashMap<>();
}
```

Aquí `Cache.datos` es una referencia que vive con la clase, y ocupa 4 u 8 bytes. El `HashMap` y los cientos de megas de `byte[]` que pueda contener están en el heap corriente. Esto es exactamente lo que hace que una caché `static` sea peligrosa: no ocupa memoria por ser `static`, sino porque **impide que el recolector libere todo lo que cuelga de ella**, que es la sección [46](#46-static-como-raíz-del-recolector).

## 11. Valores por defecto y el instante en que existen

Igual que los campos de instancia, los campos `static` reciben un valor por defecto si no les asignás ninguno. La tabla es la misma del capítulo [Data Types and Variables](../01-Basics/03-data-types-and-variables.md):

| Tipo | Valor por defecto |
|---|---|
| `byte`, `short`, `int`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | el carácter de código cero |
| `boolean` | `false` |
| Cualquier referencia | `null` |

Lo interesante es **cuándo** reciben ese valor, porque no coincide con lo que uno supone. La JVM lo hace en la fase de **preparación**, que forma parte del enlace y ocurre **antes** de ejecutar una sola línea de tu código de inicialización. Es decir:

1. La JVM reserva el hueco de cada campo `static` y lo pone a su valor por defecto. Tu código todavía no ha corrido.
2. Solo después, en la fase de **inicialización**, se ejecutan tus asignaciones y tus bloques `static`.

Esta separación explica un comportamiento que desconcierta a mucha gente, y que se ve en el ejemplo de Baeldung de la clase `Pizza`. Baeldung lo publica con esta salida:

```
Static field of enclosing class is null
Non private static field of static class is null
Private static field of static class is null
Non-static field of enclosing class is false
```

Ejecutado literalmente en JDK 25, **la salida es exactamente esa**. Baeldung acierta. Lo que no hace es explicar el orden, que es lo único interesante del ejemplo. Reconstruido:

```java
public class Pizza {
    private static String cookedCount;
    private boolean isThinCrust;

    public static class PizzaSalesCounter {
        private static String orderedCount;
        public static String deliveredCount;

        PizzaSalesCounter() {
            System.out.println("Static field of enclosing class is " + Pizza.cookedCount);
            System.out.println("Non-static field of enclosing class is " + new Pizza().isThinCrust);
        }
    }

    Pizza() {
        System.out.println("Non private static field of static class is " + PizzaSalesCounter.deliveredCount);
        System.out.println("Private static field of static class is " + PizzaSalesCounter.orderedCount);
    }

    public static void main(String[] a) { new Pizza.PizzaSalesCounter(); }
}
```

El orden sale así porque **la segunda línea del constructor de `PizzaSalesCounter` construye un `Pizza` para poder leer `isThinCrust`**, y ese constructor imprime sus propias dos líneas antes de que la concatenación pueda terminar. La secuencia real es:

1. `PizzaSalesCounter()` imprime la línea 1.
2. Empieza a evaluar la línea 4, y para ello ejecuta `new Pizza()`.
3. El constructor de `Pizza` imprime las líneas 2 y 3.
4. Vuelve a `PizzaSalesCounter()`, que ya puede completar e imprimir la línea 4.

Y todos los valores son `null` y `false` porque **son los valores por defecto**: nadie asignó nada a esos campos. Ese es el punto que el ejemplo demuestra y que el artículo no llega a decir.

---

# Parte III — Métodos static

## 12. Qué es un método static

Un método `static` pertenece a la clase y se invoca sin objeto:

```java
public class Matematicas {
    public static int cuadrado(int n) {
        return n * n;
    }
}
```

```java
int r = Matematicas.cuadrado(5);     // 25 -- sin ningún new
```

Ya usaste decenas sin darte cuenta: `Math.max(...)`, `Integer.parseInt(...)`, `String.valueOf(...)`, `List.of(...)`, `Arrays.sort(...)`. Todos son métodos `static`. La biblioteca estándar los usa para lo mismo que vas a usarlos vos: **operaciones que no dependen del estado de ningún objeto**.

`Math.max(3, 7)` devuelve 7 sin necesitar saber nada más. No hay un «objeto Math» con estado del que dependa el resultado. Por eso es `static`, y por eso `Math` ni siquiera se puede instanciar: su constructor es privado.

freeCodeCamp lo resume bien: «A static method belongs to the class rather than instances. Thus, it can be called without creating instance of class», y lista dos restricciones:

> 1. Static method can not use non-static members (variables or functions) of the class.
> 2. Static method can not use `this` or `super` keywords.

La segunda es exacta. **La primera está mal formulada** y es una imprecisión que se repite en muchos tutoriales. Un método `static` sí puede usar miembros no static: lo que no puede es usarlos **directamente**, sin decir de qué objeto. Baeldung lo dice mejor: «static methods can't access instance variables and instance methods **directly**. They need some object reference to do so». La diferencia se ve en tres líneas:

```java
public class Servicio {
    private int contador;

    public static void malo() {
        contador++;                    // error: no se sabe de qué objeto
    }

    public static void bueno(Servicio s) {
        s.contador++;                  // perfecto: el objeto viene por parámetro
    }
}
```

Y de hecho **este patrón es la base de media biblioteca estándar**: `Objects.requireNonNull(obj)`, `Collections.sort(lista)` o `Arrays.asList(array)` son métodos `static` que trabajan sobre objetos que reciben como parámetro.

## 13. Las cuatro combinaciones de acceso

Baeldung publica la lista completa de qué puede acceder a qué, y es correcta. Vale la pena memorizarla porque responde por adelantado a la mitad de los errores de compilación de este tema:

| Desde | Puede acceder directamente a |
|---|---|
| Método **de instancia** | Campos de instancia, métodos de instancia, campos static, métodos static |
| Método **static** | Campos static, métodos static |
| Método **static** | Miembros de instancia **solo a través de una referencia** a un objeto |

La regla en una frase: **hacia arriba siempre se puede, hacia abajo hace falta un objeto**. Un método de instancia ya tiene su objeto (`this`) y además ve la clase; un método `static` solo ve la clase, y para bajar a un objeto necesita que alguien se lo dé.

## 14. Por qué un método static no tiene this

Esta es la explicación que convierte la regla en algo que no hay que memorizar.

Cuando llamás a un método de instancia, el compilador le pasa el objeto como un **parámetro invisible** llamado `this`. Estas dos cosas son casi lo mismo:

```java
// lo que escribís
persona.saludar();

// lo que ocurre, conceptualmente
saludar(persona);
```

Por eso dentro de un método de instancia podés escribir `nombre` a secas: es azúcar sintáctico para `this.nombre`.

Un método `static` **no recibe ese parámetro invisible**. No hay `this`. Y sin `this`, `nombre` no se refiere a nada: no hay ningún objeto del que sacarlo. De ahí salen las dos restricciones a la vez, sin que haya que aprenderlas por separado:

- No puede usar `this` ni `super`, porque no existen.
- No puede tocar campos ni métodos de instancia directamente, porque todos ellos se resuelven, por debajo, contra `this`.

Se puede comprobar en el bytecode. Un método de instancia sin parámetros tiene `args_size=1` —ese uno es `this`—; un método `static` sin parámetros tiene `args_size=0`.

## 15. El error non-static cannot be referenced from a static context

Es, con diferencia, el error de compilación más frecuente de todo el tema. Baeldung le dedica una sección entera, con este ejemplo:

```java
public class MyClass {
    int instanceVariable = 0;

    public static void staticMethod() {
        System.out.println(instanceVariable);
    }

    public static void main(String[] args) {
        MyClass.staticMethod();
    }
}
```

Baeldung dice que el error es «Non-static variable cannot be referenced from a static context». El mensaje literal de `javac` en JDK 25 es ligeramente distinto y más informativo, porque **nombra la variable**:

```
MyClass.java:5: error: non-static variable instanceVariable cannot be referenced from a static context
        System.out.println(instanceVariable);
                           ^
1 error
```

Dónde aparece de verdad este error en el día a día: **en `main`**. Es el primer método `static` que escribe todo el mundo, y desde ahí se intenta usar lo que hay en la clase:

```java
public class App {
    private String config = "produccion";

    private void arrancar() {
        System.out.println(config);
    }

    public static void main(String[] args) {
        arrancar();                     // error
        System.out.println(config);     // error
    }
}
```

Las dos soluciones, y cuál elegir:

**Solución 1, la correcta el 95 % de las veces: crear un objeto.**

```java
public static void main(String[] args) {
    new App().arrancar();
}
```

`main` es solo el punto de entrada. Su trabajo es construir el primer objeto y delegar; todo lo demás debería ser código de instancia normal.

**Solución 2, la que hay que evitar: marcar todo como `static`.**

```java
private static String config = "produccion";
private static void arrancar() { }
```

Compila y el error desaparece. Es lo que hace casi todo el mundo la primera vez, y es cómo nacen las clases donde absolutamente todo es `static`. El resultado es un programa que no es orientado a objetos, no se puede testear y no se puede reutilizar. Está desarrollado en el anti-patrón 2 de la sección [53](#53-anti-patrones).

## 16. Llamar a un método static sobre una referencia nula

Este es un caso que sorprende a casi todo el mundo, y que demuestra hasta qué punto el acceso por instancia es una ilusión sintáctica:

```java
public class E4 {
    static String saluda() { return "metodo static invocado"; }

    public static void main(String[] x) {
        E4 referenciaNula = null;
        System.out.println(referenciaNula.saluda());
    }
}
```

Cualquiera diría que eso lanza `NullPointerException`. En JDK 25:

```
llamada static sobre referencia null: metodo static invocado
```

**No lanza nada.** Funciona.

La razón es que el compilador, al ver que `saluda()` es `static`, **no usa la referencia para nada**. La descarta y emite una invocación contra la clase: en bytecode aparece `invokestatic E4.saluda()`, no `invokevirtual`. La variable `referenciaNula` solo sirvió para que el compilador dedujera de qué clase hablabas. Que valiera `null` es irrelevante, porque nunca se desreferencia.

Es una pregunta clásica de entrevista, y sirve para algo más que lucirse: es la prueba definitiva de que `objeto.metodoStatic()` **no es una llamada sobre el objeto**. Solo lo parece.

## 17. Por qué main es static

La firma que escribiste el primer día:

```java
public static void main(String[] args) { }
```

Ahora se puede explicar entera. La JVM necesita invocar ese método para arrancar el programa, y se encuentra con un problema de huevo y gallina: **para llamar a un método de instancia haría falta un objeto, y para construir un objeto haría falta que el programa ya estuviera corriendo**. ¿Con qué constructor lo crearía? ¿Con qué argumentos?

`static` rompe el ciclo: la JVM carga la clase, la inicializa e invoca `main` sin construir nada.

Las otras tres partes de la firma tienen la misma clase de justificación:

- **`public`**: la JVM está fuera de tu paquete y necesita verlo.
- **`void`**: el código de salida del proceso se fija con `System.exit`, no con un `return`.
- **`String[] args`**: los argumentos de la línea de comandos.

Vale la pena saber que **esto está cambiando**. Java 21 introdujo como preview los *programas simples*, y en Java 25 la funcionalidad se hizo definitiva con la [JEP 512](https://openjdk.org/jeps/512): ya se puede escribir un `main` de instancia, sin `static`, sin `public` y sin parámetros, en un fichero sin clase explícita:

```java
void main() {
    IO.println("Hola");
}
```

En ese caso la JVM sí construye una instancia de la clase implícita y llama al método sobre ella. Es decir: el `static` de `main` era una consecuencia de una decisión de arranque, no una necesidad del lenguaje, y treinta años después se ha podido quitar. Para el código normal, con clases explícitas, la firma clásica sigue siendo la que vas a escribir.

---

# Parte IV — Inicialización, el corazón del tema

Esta parte es la que convierte un capítulo sobre una palabra clave en un capítulo sobre la JVM. Todo lo que sigue es la causa de bugs reales, difíciles y caros.

## 18. Las tres fases: carga, enlace e inicialización

Jenkov escribe:

> «Static fields are created when the class is loaded. A class is loaded the first time it is referenced in your program.»

Las dos frases son **imprecisas**, y la segunda es directamente falsa, como se demuestra en la sección [23](#23-las-constantes-que-no-cargan-la-clase). El problema es que mezclan tres cosas que la JVM mantiene separadas. La especificación de la máquina virtual distingue:

**1. Carga (*loading*).** El class loader localiza el fichero `.class`, lee sus bytes y crea en memoria la representación de la clase, incluido el objeto `java.lang.Class`. Aquí todavía no se ha ejecutado nada tuyo.

**2. Enlace (*linking*).** Tiene tres subfases:

- **Verificación**: se comprueba que el bytecode sea correcto y seguro.
- **Preparación**: se reserva memoria para los campos `static` y **se ponen a su valor por defecto** (`0`, `false`, `null`). Sigue sin ejecutarse código tuyo.
- **Resolución**: se resuelven las referencias simbólicas a otras clases. Puede ser perezosa.

**3. Inicialización (*initialization*).** **Ahora sí** se ejecuta tu código: los inicializadores de los campos `static` y los bloques `static`, en orden textual. Esto ocurre en el método `<clinit>` de la sección siguiente.

La distinción no es burocracia. Explica por qué este código imprime `0` y no falla:

```java
class Ejemplo {
    static int x = calcular();
    static int y = 10;

    static int calcular() {
        return y;              // y todavía vale 0, su valor por defecto
    }
}
```

`y` ya **existe** cuando se llama a `calcular()` —se creó en la preparación— pero todavía no se le ha **asignado** el 10, porque los inicializadores corren en orden textual y `x` va antes. El resultado es `x == 0`. No hay error, ni excepción, ni aviso: solo un número mal. Es la sección [21](#21-el-orden-textual-manda) y la [29](#29-ciclos-de-inicialización-sin-deadlock).

## 19. clinit, el método que no escribiste

Cuando compilás una clase con campos `static` inicializados o bloques `static`, `javac` **fabrica un método** que no aparece en tu código fuente. Se llama `<clinit>` —de *class initializer*— y contiene, fusionados y en orden textual, todos los inicializadores de campos `static` y todos los bloques `static`.

Se puede ver. Esta clase:

```java
class Demo {
    static int a = 5;
    static { a = a * 2; }
    static final int CONSTANTE = 100;
}
```

produce este bytecode:

```
static {};
  Code:
     0: iconst_5
     1: putstatic     #7          // Field a:I
     4: getstatic     #7          // Field a:I
     7: iconst_2
     8: imul
     9: putstatic     #7          // Field a:I
    12: return
```

Tres detalles importantes se leen ahí:

- El bloque `static {}` del bytecode **es** `<clinit>`. Se ejecuta una sola vez, y la JVM garantiza que **exactamente un hilo** lo ejecuta: `<clinit>` está protegido por un bloqueo interno. Esa garantía es lo que hace seguro el idiom del holder de la sección [33](#33-el-idiom-del-holder-singleton-perezoso), y también lo que hace posible el deadlock de la [28](#28-deadlock-de-inicialización).
- Los inicializadores de campo y los bloques aparecen **entremezclados en el orden en que los escribiste**, no agrupados por tipo.
- `CONSTANTE` **no aparece por ninguna parte**. Su valor viaja en el *constant pool* del `.class`, no en código ejecutable. Ese es el mecanismo de la sección [23](#23-las-constantes-que-no-cargan-la-clase).

## 20. Bloques de inicialización static

Un bloque `static` es código suelto entre llaves, precedido de `static`, dentro de la clase:

```java
public class Registro {
    static Map<String, String> paises = new HashMap<>();

    static {
        paises.put("ES", "España");
        paises.put("AR", "Argentina");
        paises.put("MX", "México");
    }
}
```

Baeldung explica bien para qué sirve: «if the static variables require multi-statement logic during initialization we can use a static block instead», y lista los dos motivos reales:

> - to initialize static variables needs some additional logic apart from the assignment
> - to initialize static variables with a custom exception handling

El segundo motivo es el más práctico y el menos conocido. Un inicializador de campo no puede tener un `try`:

```java
static Properties config = cargar();     // ¿y si cargar() lanza IOException?
```

Con un bloque sí:

```java
public class Configuracion {
    static final Properties CONFIG;

    static {
        Properties p = new Properties();
        try (InputStream in = Configuracion.class.getResourceAsStream("/app.properties")) {
            p.load(in);
        } catch (IOException e) {
            throw new ExceptionInInitializerError(e);
        }
        CONFIG = p;
    }
}
```

Hay una restricción que conviene conocer: **un bloque `static` no puede lanzar excepciones comprobadas**. Si lo intentás:

```
error: initializer must be able to complete normally
```

Por eso hay que envolverlas en una no comprobada, como en el ejemplo. Y ahí aparece un dilema que la sección [26](#26-cuando-la-inicialización-falla) desarrolla: si ese `catch` se ejecuta, la clase queda **permanentemente inutilizable** en ese proceso.

**Una clase puede tener varios bloques `static`.** Baeldung lo muestra:

```java
public class StaticBlockDemo {
    public static List<String> ranks = new LinkedList<>();

    static {
        ranks.add("Lieutenant");
        ranks.add("Captain");
        ranks.add("Major");
    }

    static {
        ranks.add("Colonel");
        ranks.add("General");
    }
}
```

Se ejecutan en orden de aparición, como si fueran uno solo. La JLS lo dice literalmente: «execute either the class variable initializers and static initializers of the class […] **in textual order, as though they were a single block**».

## 21. El orden textual manda

freeCodeCamp afirma sobre los bloques `static`:

> «These blocks are executed immediately after declaration of static variables.»

**Es falso.** No se ejecutan «inmediatamente después de la declaración de las variables static»: se ejecutan intercalados con ellas, en el orden en que estén escritos. Un bloque que esté antes de un campo corre antes que ese campo.

Comprobación:

```java
public class E2 {
    static int a = traza("campo a");
    static { traza("bloque static 1"); }
    static int b = traza("campo b");
    static { traza("bloque static 2"); }

    static int traza(String s) { System.out.println("   ejecuta: " + s); return 0; }

    public static void main(String[] x) { System.out.println("main"); }
}
```

Salida real en JDK 25:

```
   ejecuta: campo a
   ejecuta: bloque static 1
   ejecuta: campo b
   ejecuta: bloque static 2
main
```

Intercalado, exactamente en el orden del fuente. Si la afirmación de freeCodeCamp fuera cierta, los dos campos correrían primero y luego los dos bloques.

**Por qué importa.** De aquí sale un bug clásico, que aparece en un hilo muy conocido de Stack Overflow sobre inicializadores static. Alguien escribe:

```java
public class Paises {
    static {
        todos = new HashMap<>();
        todos.put("ES", new Pais());
    }

    private static Map<String, Pais> todos = null;      // ¡el = null va después!
}
```

y se encuentra con que `todos` vale `null` en tiempo de ejecución. El bloque `static` construye el mapa, y **después** el inicializador del campo lo machaca con `null`, porque está escrito más abajo. La corrección es quitar el `= null` explícito, o mover la declaración por encima del bloque. Preferiblemente las dos cosas.

La regla práctica: **declarar siempre los campos `static` antes de los bloques `static` que los usan**, y no asignar `null` explícitamente, que es redundante con el valor por defecto.

## 22. Cuándo se inicializa una clase de verdad

La JLS enumera exactamente cinco situaciones que provocan la inicialización de una clase. Fuera de estas cinco, la clase **no se inicializa**:

> Una clase o interfaz `T` se inicializará inmediatamente antes de la primera ocurrencia de cualquiera de las siguientes:
>
> - `T` es una clase y se crea una instancia de `T`.
> - `T` es una clase y se invoca un método `static` declarado por `T`.
> - Se **asigna** un campo `static` declarado por `T`.
> - Se **usa** un campo `static` declarado por `T` **y el campo no es una variable constante**.
> - `T` es una clase de nivel superior y se ejecuta una sentencia `assert` léxicamente anidada en `T`.

Las cuatro primeras son las que vas a encontrarte. La última cláusula —«y el campo no es una variable constante»— es la que desmiente a Jenkov y merece sección propia.

Conviene fijar dos cosas que **no** disparan la inicialización, porque son fuente de confusión:

**Declarar una variable del tipo no inicializa la clase.**

```java
Servicio s;                            // NO inicializa Servicio
Servicio[] array = new Servicio[10];   // NO inicializa Servicio (solo el array)
```

Un array de `Servicio` no crea ningún `Servicio`, así que no hay motivo para inicializar la clase.

**Acceder a un campo static de la superclase a través de la subclase inicializa la superclase, no la subclase.**

```java
class Padre { static int x = 1; }
class Hijo extends Padre { static { System.out.println("Hijo init"); } }

System.out.println(Hijo.x);      // NO imprime "Hijo init"
```

`x` está declarado en `Padre`, no en `Hijo`. Escribir `Hijo.x` es solo otra manera de escribir `Padre.x`, y solo inicializa `Padre`.

## 23. Las constantes que no cargan la clase

Esta es la trampa más rentable de todo el capítulo, la que explica bugs de despliegue reales, y la que desmiente frontalmente la frase de Jenkov «a class is loaded the first time it is referenced in your program».

La JLS define **variable constante** (*constant variable*) así: una variable `final`, de tipo primitivo o `String`, inicializada con una **expresión constante en tiempo de compilación**. Para esas variables, y solo para esas, el compilador **copia el valor literalmente** en cada sitio donde se usan. La clase de origen desaparece del bytecode.

Prueba directa:

```java
class Config {
    static final int MAX = 100;                  // variable constante
    static final String NAME = "produccion";     // variable constante
    static final int CALCULADO = calcular();     // final, pero NO expresión constante
    static int mutable = 5;

    static { System.out.println("  >>> <clinit> de Config SE EJECUTO"); }

    static int calcular() { return 7; }
}
```

```java
public class E1 {
    public static void main(String[] a) {
        System.out.println("1) leo Config.MAX (final + literal):");
        System.out.println("   valor = " + Config.MAX);
        System.out.println("2) leo Config.NAME (final + literal String):");
        System.out.println("   valor = " + Config.NAME);
        System.out.println("3) leo Config.CALCULADO (final pero NO expresion constante):");
        System.out.println("   valor = " + Config.CALCULADO);
    }
}
```

Salida real en JDK 25:

```
1) leo Config.MAX (final + literal):
   valor = 100
2) leo Config.NAME (final + literal String):
   valor = produccion
3) leo Config.CALCULADO (final pero NO expresion constante):
  >>> <clinit> de Config SE EJECUTO
   valor = 7
```

Leer `Config.MAX` y `Config.NAME` **no ejecutó el bloque `static`**. La clase `Config` ni siquiera se inicializó. Solo el tercer acceso, a un campo `final` cuyo valor **no** es una expresión constante, la despertó.

El bytecode lo demuestra sin lugar a dudas. Así queda `main`:

```
 8: getstatic     #7    // Field java/lang/System.out:Ljava/io/PrintStream;
11: ldc           #23   // String    valor = 100
13: invokevirtual #15   // Method java/io/PrintStream.println:(Ljava/lang/String;)V
...
40: getstatic     #7    // Field java/lang/System.out:Ljava/io/PrintStream;
43: getstatic     #31   // Field Config.CALCULADO:I
46: invokedynamic #35,  0
```

En la línea 11 no hay ninguna referencia a `Config`: el compilador resolvió `"   valor = " + Config.MAX` **entero en tiempo de compilación** y dejó la cadena `"   valor = 100"` como literal. En cambio en la línea 43 sí hay un `getstatic Field Config.CALCULADO:I`, y **esa instrucción es la que dispara la inicialización**.

Las dos condiciones para que ocurra son estrictas: **`final`** y **expresión constante**. Basta con romper cualquiera de las dos para que el campo vuelva a comportarse normalmente:

```java
static final int A = 100;              // constante: se inlinea
static int B = 100;                    // NO final: no se inlinea
static final int C = calcular();       // no es expresión constante: no se inlinea
static final Integer D = 100;          // NO es primitivo ni String: no se inlinea
static final int[] E = {1, 2, 3};      // tampoco: es un array
```

El caso de `D` sorprende: `Integer` es un envoltorio, no un primitivo, así que **no** es una variable constante aunque parezca idéntica a `A`.

## 24. La trampa del despliegue parcial

Lo de la sección anterior no es una curiosidad: es un bug de producción con nombre propio, y una de las pocas formas que tiene Java de romper la compatibilidad binaria.

Escenario. Tenés una librería con una constante:

```java
// libreria-1.0.jar
public class Limites {
    public static final int MAX_REINTENTOS = 3;
}
```

Y una aplicación que la usa:

```java
// app.jar
if (intentos < Limites.MAX_REINTENTOS) { }
```

Compilás la aplicación contra la versión 1.0. En el `.class` de tu aplicación **no queda ninguna referencia a `Limites`**: queda un `3` literal.

Meses después cambiás la librería:

```java
// libreria-2.0.jar
public static final int MAX_REINTENTOS = 5;
```

Desplegás **solo el jar de la librería**, sin recompilar la aplicación. Y la aplicación **sigue usando 3**. No hay error, no hay aviso, no hay nada en los logs. El valor viejo está grabado dentro de tu bytecode.

Este comportamiento está reconocido en la JLS, en el capítulo de compatibilidad binaria, y hay una recomendación oficial: si una constante puede cambiar en el futuro, **no la declares como variable constante**. Convertila en algo que se resuelva en tiempo de ejecución:

```java
// en vez de esto
public static final int MAX_REINTENTOS = 5;

// esto, que ya no es una expresión constante
public static final int MAX_REINTENTOS = Integer.valueOf(5);

// o mejor, algo con sentido
public static final int MAX_REINTENTOS = leerDeConfiguracion("max.reintentos", 5);
```

La regla práctica: **`static final int` con literal está bien para constantes matemáticas o del dominio que no van a cambiar nunca** (`DIAS_SEMANA = 7`), y es peligroso para parámetros de configuración que sí podrían cambiar. Para esos, o se leen en tiempo de ejecución, o al menos hay que recordar recompilar todo lo que dependa de ellos.

## 25. El orden en la jerarquía de herencia

Cuando se inicializa una clase, la JVM inicializa primero su superclase. Siempre, y hasta arriba.

```java
class Padre { static { System.out.println("  [Padre] <clinit>"); } }
class Hijo extends Padre { static { System.out.println("  [Hijo] <clinit>"); } static void ping() {} }
```

```java
Hijo.ping();
```

Salida:

```
  [Padre] <clinit>
  [Hijo] <clinit>
```

Es de arriba hacia abajo, y ocurre antes de que se ejecute nada de la subclase. Hay dos detalles que conviene tener presentes:

- **Las interfaces implementadas no se inicializan** al inicializar la clase, salvo que declaren campos `static` no constantes. Una interfaz solo con métodos `default` y `static` no se inicializa por el hecho de implementarla.
- Este orden, combinado con un ciclo entre superclase y subclase, es precisamente el patrón que produce el deadlock más citado en la literatura, que es la sección [28](#28-deadlock-de-inicialización).

## 26. Cuando la inicialización falla

¿Qué pasa si el código de un bloque `static` lanza una excepción? La respuesta tiene consecuencias serias.

La JVM envuelve la excepción original en un **`ExceptionInInitializerError`** —un `Error`, no una `Exception`— y lo propaga. Pero lo importante viene después: **marca la clase como errónea**, permanentemente, para toda la vida del proceso.

```java
class Configuracion {
    static final int VALOR;
    static { VALOR = Integer.parseInt("no-es-un-numero"); }
}
```

```java
for (int intento = 1; intento <= 2; intento++) {
    try {
        System.out.println("intento " + intento + ": " + Configuracion.VALOR);
    } catch (Throwable t) {
        System.out.println("intento " + intento + " lanzo: " + t.getClass().getName());
        System.out.println("            causa: " + t.getCause());
    }
}
```

Salida real en JDK 25:

```
intento 1 lanzo: java.lang.ExceptionInInitializerError
            causa: java.lang.NumberFormatException: For input string: "no-es-un-numero"
intento 2 lanzo: java.lang.NoClassDefFoundError
            causa: java.lang.ExceptionInInitializerError: Exception java.lang.NumberFormatException: For input string: "no-es-un-numero" [in thread "main"]
```

Dos cosas distintas en dos intentos idénticos:

- **La primera vez**: `ExceptionInInitializerError`, con la excepción real como causa.
- **La segunda vez y todas las siguientes**: `NoClassDefFoundError`, porque la clase quedó marcada como errónea y la JVM ni intenta reinicializarla.

**No hay forma de recuperarse.** La clase no se puede reinicializar sin reiniciar la JVM (o sin descartar el class loader entero). Por eso hay una regla de oro: **nunca captures y descartes un `Error` de inicialización**. Si lo hacés, el `ExceptionInInitializerError` con la causa real desaparece y lo único que verás después son `NoClassDefFoundError` sin información útil.

## 27. La clase en estado erróneo

El `NoClassDefFoundError` de la sección anterior es probablemente el error peor bautizado de Java. Su nombre sugiere «no encuentro la clase», que es lo que hace `ClassNotFoundException`. Lo que significa en realidad es **«esta clase existe, se encontró y se cargó, pero su inicialización falló y por tanto no se puede usar»**.

El mensaje lo dice, aunque haya que fijarse: `Could not initialize class ...`, no `Could not find class ...`.

Aquí hay una **novedad importante que invalida los hilos antiguos de Stack Overflow** sobre este tema. Durante años, la queja recurrente fue que el `NoClassDefFoundError` no traía ninguna información sobre la causa original, y que si el primer `ExceptionInInitializerError` se había perdido en un log, no había forma humana de saber qué había fallado. Existe un bug abierto sobre esto desde 2014 ([JDK-8048190](https://bugs.openjdk.org/browse/JDK-8048190)), y las respuestas de la época proponían soluciones tan extremas como escribir un agente Java que instrumentara el constructor de `ExceptionInInitializerError`.

**Eso ya está resuelto.** Como se ve en la salida de arriba, en JDK 25 el `NoClassDefFoundError` del segundo intento **sí trae la causa encadenada**, con el mensaje original de la `NumberFormatException` y hasta el nombre del hilo donde ocurrió. Si estás diagnosticando esto en un JDK moderno, mirá la cadena de `Caused by`: la información que las respuestas viejas daban por perdida está ahí.

**Cómo evitar el problema de raíz.** El consejo que da la respuesta mejor valorada de aquel hilo sigue siendo el bueno:

> «Mi consejo sería evitar este problema evitando los inicializadores static tanto como puedas. Como se ejecutan durante el proceso de carga de clases, muchos frameworks no los manejan bien.»

Trasladado a reglas concretas:

- Un bloque `static` que hace **E/S** (leer un fichero, abrir una conexión, llamar a una API) es una bomba de relojería. Movelo a un método que se llame explícitamente al arrancar la aplicación, donde podés manejar el fallo, reintentar o fallar con un mensaje claro.
- Si el bloque `static` es inevitable, **no tragues la excepción**. Envolvela en `ExceptionInInitializerError` con la causa, como en el ejemplo de la sección [20](#20-bloques-de-inicialización-static).
- Al ver un `NoClassDefFoundError: Could not initialize class X`, **buscá hacia arriba en el log**. El error de verdad ocurrió antes.

## 28. Deadlock de inicialización

Este es el escenario que más gente descubre por las malas, porque no se parece a ningún deadlock que hayas visto.

La JVM garantiza que `<clinit>` se ejecuta una sola vez, y lo consigue con un **bloqueo por clase**. La JLS lo dice explícitamente: «For each class or interface C, there is a unique initialization lock LC», y el primer paso del procedimiento de inicialización es «Synchronize on the initialization lock, LC, for C. This involves waiting until the current thread can acquire LC».

Ese bloqueo es lo que hace seguros los singletons. Y es también lo que permite un deadlock si dos clases se inicializan mutuamente desde dos hilos:

```java
public class E9b {
    static class Foo {
        static {
            try { System.out.println("  inicializando Foo..."); Thread.sleep(1000); new Bar(); }
            catch (InterruptedException e) { throw new Error(e); }
            System.out.println("  Foo inicializada");
        }
    }
    static class Bar {
        static {
            try { System.out.println("  inicializando Bar..."); Thread.sleep(1000); new Foo(); }
            catch (InterruptedException e) { throw new Error(e); }
            System.out.println("  Bar inicializada");
        }
    }
    public static void main(String[] x) throws Exception {
        Thread t1 = new Thread(Foo::new, "hilo-Foo");
        Thread t2 = new Thread(Bar::new, "hilo-Bar");
        t1.start(); t2.start();
        t1.join(4000); t2.join(4000);
    }
}
```

Salida real en JDK 25:

```
  inicializando Foo...
  inicializando Bar...
RESULTADO: DEADLOCK -- ningun hilo termino
  hilo-Foo estado=RUNNABLE
     bloqueado en: E9b$Foo.<clinit>(E9b.java:4)
  hilo-Bar estado=RUNNABLE
     bloqueado en: E9b$Bar.<clinit>(E9b.java:11)
```

Ninguno de los dos hilos termina. `hilo-Foo` tiene el bloqueo de `Foo` y espera el de `Bar`; `hilo-Bar` tiene el de `Bar` y espera el de `Foo`.

**Y aquí está el detalle que hace que este deadlock sea tan difícil de diagnosticar.** Fijate en el estado de los hilos: **`RUNNABLE`**, no `BLOCKED`. Un deadlock normal de `synchronized` deja los hilos en `BLOCKED` y la JVM lo detecta sola. Este no. Comprobación directa con la API de gestión:

```java
ThreadMXBean bean = ManagementFactory.getThreadMXBean();
long[] enDeadlock = bean.findDeadlockedThreads();
long[] porMonitor = bean.findMonitorDeadlockedThreads();
```

```
findDeadlockedThreads() -> null (NO detecta nada)
findMonitorDeadlockedThreads() -> null (NO detecta nada)
```

**El detector de deadlocks de la JVM no ve este bloqueo.** Ni `findDeadlockedThreads()` ni `findMonitorDeadlockedThreads()` devuelven nada. Un thread dump mostrará dos hilos aparentemente activos, parados en `<clinit>`, sin ninguna línea de «waiting to lock» que te oriente. Si alguna vez ves un thread dump con varios hilos `RUNNABLE` cuyo tope de pila es `<clinit>` y la aplicación está colgada, ya sabés lo que es.

**El patrón realista.** El ejemplo con `Thread.sleep` es artificial para forzar el entrelazado, pero el patrón que lo produce en código real es mucho más inocente: **una clase cuyo inicializador `static` referencia a una subclase suya**.

```java
class Base {
    private static final Sub INSTANCIA = new Sub();
    static class Sub extends Base { }
}
```

Para inicializar `Base` hay que inicializar `Sub` (por el `new`), y para inicializar `Sub` hay que inicializar `Base` (porque es su superclase, sección [25](#25-el-orden-en-la-jerarquía-de-herencia)). En un solo hilo funciona, porque la JVM permite la reentrada del mismo hilo. Con dos hilos —uno entrando por `Base` y otro por `Sub`— se bloquea.

Esto no es teórico. Le pasó a **JavaPoet**, la librería de generación de código de Square: su clase `TypeName` tenía un campo `static` de tipo `ClassName`, y `ClassName` extiende `TypeName`. El equipo de Palantir escribió una regla de Error Prone, [`ClassInitializationDeadlock`](https://errorprone.info/bugpattern/ClassInitializationDeadlock), específicamente para prohibir este patrón, y el bug de OpenJDK [JDK-8037567](https://bugs.openjdk.org/browse/JDK-8037567) confirma que **es comportamiento esperado según la especificación** y que hay que evitarlo en el código de aplicación, no en la JVM.

**Cómo evitarlo:**

1. **No referencies subtipos de la clase actual desde su inicializador `static`.** Es la regla completa.
2. Si necesitás la constante, **sacala a otra clase** que no sea la superclase:

```java
class Constantes {
    static final Sub INSTANCIA = new Sub();
}
class Base { }
class Sub extends Base { }
```

3. Si la subclase debe seguir dentro, hacé que **solo se pueda construir a través del contenedor**: constructor privado y sin miembros `static` accesibles, de modo que ningún hilo pueda entrar por ella.

## 29. Ciclos de inicialización sin deadlock

Hay una variante del problema anterior que no bloquea nada y por eso es todavía más traicionera: **produce valores incorrectos silenciosamente**.

Cuando un hilo está inicializando una clase y, dentro de ese proceso, vuelve a necesitar la misma clase, la JVM **no se bloquea**: detecta que es el mismo hilo y deja pasar, asumiendo que la clase ya está lista. Pero no lo está: los campos que todavía no se han ejecutado siguen con su valor por defecto.

```java
class Primera {
    static int alfa = 5;
    static int beta = Segunda.gamma;
}

class Segunda {
    static int gamma = Primera.alfa;
}
```

Si el programa toca `Primera` primero:

1. Empieza a inicializar `Primera`.
2. Asigna `alfa = 5`.
3. Para `beta` necesita `Segunda.gamma`, así que inicializa `Segunda`.
4. `Segunda.gamma = Primera.alfa`. `Primera` **ya está en proceso de inicialización en este hilo**, así que se lee directamente: `alfa` vale 5. Correcto.
5. `beta` recibe 5.

Pero si se declaran en otro orden, o si el programa toca `Segunda` primero, el paso 4 lee `alfa` **antes** de que se le haya asignado el 5, y obtiene `0`. Jon Skeet documentó este comportamiento con detalle y lo describe como «pretty much your classic Heisenbug»: el resultado depende de qué clase se toque primero, así que **la suite de tests completa puede fallar y el test aislado pasar**.

Un hilo de Stack Overflow reporta el mismo síntoma con este código:

```java
private static final float SIDE = 100.0f * new Random().nextFloat();
private static final float[] POINTS = generatePoints();      // usa SIDE
```

`POINTS` se calcula antes que `SIDE`, así que `generatePoints()` lee `SIDE` valiendo `0.0`. Y lo desconcertante: si se cambia a `SIDE = 100.0f` a secas, **el problema desaparece**, porque entonces sí es una expresión constante y el compilador la inlinea (sección [23](#23-las-constantes-que-no-cargan-la-clase)). El mismo código, con y sin `new Random()`, se comporta de forma distinta.

**Reglas para no caer:**

- **Nunca hagas que el inicializador de una clase dependa de otra clase que dependa de la primera.** Si dos clases se necesitan mutuamente al arrancar, el diseño está mal.
- **Declará los campos `static` en orden de dependencia**: si `b` usa `a`, `a` va antes.
- IntelliJ avisa de esto con un mensaje bastante explícito: *«uses of non-final static variables during initialization of a class. Such uses may make the semantics of the code dependent on order of class creation, may cause variables to be used before initialized, and generally cause extremely difficult and confusing bugs»*. Hacele caso.

---

# Parte V — Clases anidadas static

## 30. Anidada static frente a interna

Java permite declarar una clase dentro de otra, y hay dos variantes que se parecen mucho en la sintaxis y se comportan de forma completamente distinta:

```java
public class Externa {

    static class Anidada { }      // clase anidada static (static nested class)

    class Interna { }             // clase interna (inner class)
}
```

Baeldung lo resume bien: «nested classes that we declare static are called static nested classes; nested classes that are non-static are called inner classes. The main difference between these two is that the inner classes have access to all members of the enclosing class (including private ones), whereas the static nested classes only have access to static members of the outer class».

La diferencia práctica se ve al instanciarlas:

```java
// anidada static: se construye sola
Externa.Anidada a = new Externa.Anidada();

// interna: necesita un objeto de la externa
Externa externa = new Externa();
Externa.Interna i = externa.new Interna();
```

Esa sintaxis `externa.new Interna()` es la pista visible de lo que ocurre por debajo: **una clase interna no puede existir sin un objeto de la clase que la contiene**. Está atada a él.

freeCodeCamp lo muestra correctamente con este ejemplo:

```java
public class Outer {
  public Outer() { }
  public static class Inner {
    public Inner() { }
  }
}
```

```java
Outer.Inner inner = new Outer.Inner();
```

## 31. La referencia oculta al objeto externo

Aquí hay un matiz que ninguna de las tres fuentes recoge, y que corrige una creencia muy extendida.

La versión popular dice: «una clase interna guarda una referencia oculta al objeto externo, y por eso consume más memoria; una anidada `static` no». Baeldung lo formula así al recomendar `static`: «This way, we won't couple it to the outer class, and **they won't require any heap or stack memory**».

La realidad, comprobada con `javap` en JDK 25, es más precisa. Estas tres clases:

```java
public class E12 {
    private int valorDelExterno = 42;

    class InternaQueUsaElExterno { int leer() { return valorDelExterno; } }
    class InternaQueNoLoUsa      { int leer() { return 7; } }
    static class Anidada         { int leer() { return 7; } }
}
```

producen esto:

```
===== E12$InternaQueUsaElExterno =====
class E12$InternaQueUsaElExterno {
  final E12 this$0;
  E12$InternaQueUsaElExterno(E12);
  int leer();
}
===== E12$InternaQueNoLoUsa =====
class E12$InternaQueNoLoUsa {
  E12$InternaQueNoLoUsa(E12);
  int leer();
}
===== E12$Anidada =====
class E12$Anidada {
  E12$Anidada();
  int leer();
}
```

Tres comportamientos distintos, no dos:

1. **La interna que usa el externo** tiene el campo sintético `final E12 this$0`. Es la referencia oculta de la que habla todo el mundo, y es real.
2. **La interna que NO usa el externo no tiene el campo.** `javac` lo ha eliminado porque nadie lo lee. El objeto ocupa lo mismo que la anidada `static`.
3. **La anidada static** no tiene campo ni parámetro.

Pero fijate en el punto 2: aunque no tenga el campo, **su constructor sigue exigiendo un `E12`**. El bytecode del constructor lo confirma: recibe el parámetro, lo pasa por `Objects.requireNonNull` y lo descarta con `pop`.

```
 0: aload_1
 1: dup
 2: invokestatic  #1     // Method java/util/Objects.requireNonNull:(Ljava/lang/Object;)Ljava/lang/Object;
 5: pop
 6: pop
 7: aload_0
 8: invokespecial #7     // Method java/lang/Object."<init>":()V
```

**Conclusión, que matiza a Baeldung:** el argumento de memoria es débil —el compilador ya optimiza ese caso— pero el argumento de fondo es correcto y más fuerte de lo que la fuente dice. Lo que gana `static` no son bytes: es que **la clase deja de necesitar un objeto externo para existir**. Y ese es el acoplamiento que importa, porque afecta a si podés construirla en un test, serializarla o moverla de sitio.

Donde la referencia sí cuesta memoria de verdad es en el caso 1, y ahí el problema no es el tamaño sino la **retención**: mientras alguien tenga una `InternaQueUsaElExterno` viva, el objeto `E12` entero no se puede recolectar, por grande que sea. Es la misma familia de problemas de la sección [47](#47-fugas-de-memoria-por-campos-static).

## 32. Qué ve una clase anidada static

Una clase anidada `static` **no** ve los campos de instancia del externo directamente, pero **sí** ve todos sus miembros `static`, incluidos los `private`:

```java
public class Externa {
    private static String secretoDeClase = "visible";
    private int campoDeInstancia = 7;

    static class Anidada {
        void probar() {
            System.out.println(secretoDeClase);        // OK: es static
            // System.out.println(campoDeInstancia);   // error: hace falta un objeto
            System.out.println(new Externa().campoDeInstancia);  // OK: con objeto
        }
    }
}
```

Ese acceso a un `private` desde otra clase es posible por el mecanismo de *nests* introducido en Java 11, que se explicó en el capítulo [Access Specifiers](03-access-specifiers.md): a efectos de control de acceso, una clase y sus anidadas son un mismo nido y pueden verse los miembros privados entre sí, sin métodos puente.

## 33. El idiom del holder: singleton perezoso

Este es el uso más elegante de una clase anidada `static`, y aprovecha directamente todo lo de la Parte IV.

El problema: querés una única instancia de un objeto caro, creada de forma segura entre hilos, y **solo si alguien la pide**. La solución ingenua es un campo `static final`:

```java
public class Servicio {
    private static final Servicio INSTANCIA = new Servicio();   // se crea siempre
    public static Servicio getInstancia() { return INSTANCIA; }
}
```

Es seguro entre hilos —`<clinit>` lo garantiza— pero **no es perezoso**: el objeto se crea en cuanto se inicializa `Servicio`, aunque nadie llame nunca a `getInstancia()`. Si la clase tiene además otros métodos `static` útiles, cualquiera de ellos dispara la creación.

El *initialization-on-demand holder idiom* resuelve las dos cosas. Baeldung lo menciona: «we can use a nested static class to implement the singleton pattern […] We use this method because it doesn't require any synchronization and is easy to learn and implement». Lo que no explica es **por qué funciona**, que es lo interesante:

```java
class Servicio {
    static { System.out.println("  [Servicio] <clinit>"); }
    static final String VERSION = "1.0";

    private Servicio() { System.out.println("  [Servicio] constructor: recurso caro creado"); }

    private static class Holder {
        static { System.out.println("  [Holder] <clinit>"); }
        static final Servicio INSTANCIA = new Servicio();
    }

    static Servicio getInstancia() { return Holder.INSTANCIA; }

    static void metodoSuelto() { System.out.println("  metodoSuelto() ejecutado"); }
}
```

Salida real en JDK 25:

```
1) llamo Servicio.metodoSuelto() -- inicializa Servicio pero NO Holder:
  [Servicio] <clinit>
  metodoSuelto() ejecutado
2) leo Servicio.VERSION (constante) -- no deberia hacer nada nuevo:
   1.0
3) ahora si llamo getInstancia():
  [Holder] <clinit>
  [Servicio] constructor: recurso caro creado
4) segunda llamada a getInstancia():
```

Léelo con atención, porque cada línea confirma una pieza distinta de este capítulo:

- **Paso 1**: llamar a un método `static` cualquiera inicializa `Servicio`, pero **no toca `Holder`**. `Holder` es una clase aparte y nadie la ha usado todavía.
- **Paso 2**: leer `VERSION` no hace nada, porque es una variable constante y se inlineó (sección [23](#23-las-constantes-que-no-cargan-la-clase)).
- **Paso 3**: la primera llamada a `getInstancia()` accede a `Holder.INSTANCIA`, y **ahí** se inicializa `Holder` y se crea el objeto caro.
- **Paso 4**: la segunda llamada **no imprime nada**. La clase ya está inicializada y `<clinit>` no se repite.

El idiom consigue las tres propiedades a la vez, y sin una sola línea de sincronización:

| Propiedad | De dónde sale |
|---|---|
| **Perezoso** | `Holder` solo se inicializa cuando se lee `Holder.INSTANCIA` |
| **Seguro entre hilos** | La JVM garantiza que `<clinit>` lo ejecuta un solo hilo |
| **Sin coste** | No hay `synchronized` ni `volatile` en el camino de lectura |

Es la razón por la que este patrón se prefiere al *double-checked locking*, que necesita `volatile`, es fácil de escribir mal y fue directamente incorrecto antes de Java 5.

**Advertencia honesta:** que el patrón sea elegante no significa que el singleton sea buena idea. Un singleton es estado global con otro nombre, y arrastra los problemas de las secciones [49](#49-static-y-testabilidad) y [50](#50-static-frente-a-inyección-de-dependencias). Si estás en una aplicación con inyección de dependencias, el contenedor ya te da instancias únicas sin ninguno de esos inconvenientes.

## 34. Builder, el uso más frecuente

freeCodeCamp cierra su sección de clases anidadas mencionando el patrón Builder, y tiene razón en que es el uso más común de una clase anidada `static` en código real:

```java
public class Pedido {
    private final String cliente;
    private final int cantidad;
    private final boolean urgente;

    private Pedido(Builder b) {
        this.cliente  = b.cliente;
        this.cantidad = b.cantidad;
        this.urgente  = b.urgente;
    }

    public static Builder builder() { return new Builder(); }

    public static class Builder {
        private String cliente;
        private int cantidad = 1;
        private boolean urgente = false;

        public Builder cliente(String c)   { this.cliente = c; return this; }
        public Builder cantidad(int n)     { this.cantidad = n; return this; }
        public Builder urgente(boolean u)  { this.urgente = u; return this; }

        public Pedido build() {
            if (cliente == null) throw new IllegalStateException("cliente es obligatorio");
            return new Pedido(this);
        }
    }
}
```

```java
Pedido p = Pedido.builder()
                 .cliente("Ana")
                 .cantidad(3)
                 .urgente(true)
                 .build();
```

**Por qué el `Builder` tiene que ser `static`.** Si no lo fuera, para construir el builder haría falta un `Pedido`… que es justo lo que estás intentando construir. La llamada sería `new Pedido().new Builder()`, lo cual no tiene ningún sentido. Es un caso donde `static` no es una optimización: es la única opción que funciona.

Y por qué está **anidada** en lugar de ser una clase suelta: porque `Builder` necesita acceso al constructor `private` de `Pedido`, y solo lo tiene estando dentro (por el mecanismo de nests). Eso permite que `Pedido` no tenga ningún constructor público: la única forma de crear uno es a través del builder, que valida.

## 35. static dentro de una inner class desde Java 16

Un cambio reciente que ninguna de las tres fuentes menciona, ni siquiera Baeldung, que actualizó su artículo en enero de 2024.

Históricamente, una clase **interna** (no `static`) tenía prohibido declarar cualquier miembro `static`, salvo constantes. La lógica era coherente: si la clase interna no puede existir sin un objeto externo, un miembro suyo que pertenece a la clase y no a los objetos resultaba conceptualmente confuso.

La [JEP 395](https://openjdk.org/jeps/395), la que hizo definitivos los records en Java 16, **relajó esa restricción**. El motivo fue práctico: los records anidados son implícitamente `static`, así que sin el cambio no se habría podido declarar un record dentro de una clase interna.

La JLS lo documenta: «Prior to Java SE 16, an inner class could not declare static initializers, and could only declare static members that were constant variables». Hoy:

```java
public class E6 {
    class Interna {                    // inner class, NO static
        static int contador = 0;       // prohibido antes de Java 16
        static void reset() { contador = 0; }
        record Punto(int x, int y) {}  // record anidado: implícitamente static
    }

    public static void main(String[] a) {
        E6.Interna.contador = 7;
        System.out.println("campo static dentro de inner class = " + E6.Interna.contador);
        System.out.println("record dentro de inner class = " + new E6.Interna.Punto(1, 2));
    }
}
```

Salida en JDK 25:

```
campo static dentro de inner class = 7
record dentro de inner class = Punto[x=1, y=2]
```

Compila y funciona sin avisos. En cualquier JDK anterior a 16, ese mismo fichero produce `error: Illegal static declaration in inner class`. Si mantenés código que todavía compila con Java 11 y ves ese error, ahora sabés que no es un fallo tuyo: es una restricción que ya no existe.

---

# Parte VI — static y herencia

## 36. Se heredan pero no se sobrescriben

Baeldung lo dice en una línea: «The same as for static fields, static methods can't be overridden. This is because static methods in Java are resolved at compile time, while method overriding is part of Runtime Polymorphism».

Es correcto, pero incompleto, y la parte que falta es la que confunde a todo el mundo. Hay que separar dos cosas que suenan igual:

- **¿Se heredan?** **Sí.** Una subclase puede llamar a los métodos `static` de su superclase sin cualificarlos. La JLS lo confirma: una clase hereda de su superclase todos los métodos concretos, «both static and instance», si son accesibles.
- **¿Se sobrescriben?** **No.** Si la subclase declara un método `static` con la misma firma, no lo sobrescribe: lo **oculta** (*hiding*).

La documentación oficial de Oracle lo formula en una tabla que vale la pena copiar entera, porque responde a todos los casos:

| Se define en la subclase un método con la misma firma que uno de la superclase | El de la superclase es de instancia | El de la superclase es static |
|---|---|---|
| **El de la subclase es de instancia** | Lo **sobrescribe** | **Error de compilación** |
| **El de la subclase es static** | **Error de compilación** | Lo **oculta** |

Las dos casillas de error son importantes: **no se puede convertir un método de instancia en static al heredar, ni al revés**. El compilador lo rechaza.

## 37. Hiding: qué versión se ejecuta

La diferencia entre ocultar y sobrescribir se ve en una sola ejecución:

```java
class Animal {
    static String metodoDeClase() { return "Animal.metodoDeClase"; }
    String metodoDeInstancia()    { return "Animal.metodoDeInstancia"; }
}

class Gato extends Animal {
    static String metodoDeClase() { return "Gato.metodoDeClase"; }
    @Override String metodoDeInstancia() { return "Gato.metodoDeInstancia"; }
}
```

```java
Gato gato = new Gato();
Animal comoAnimal = gato;

System.out.println(((Animal) gato).metodoDeClase());   // ¿cuál sale?
System.out.println(comoAnimal.metodoDeInstancia());    // ¿y cuál aquí?
```

Salida real en JDK 25:

```
referencia Animal -> metodo static   : Animal.metodoDeClase
referencia Gato   -> metodo static   : Gato.metodoDeClase
objeto Gato en variable Animal:
   metodo static   (hiding)   : Animal.metodoDeClase
   metodo instancia(override) : Gato.metodoDeInstancia
```

**El mismo objeto**, un `Gato`, produce respuestas distintas:

- El método **de instancia** ejecuta la versión de `Gato`. Manda **el objeto**, en tiempo de ejecución. Eso es polimorfismo.
- El método **static** ejecuta la versión de `Animal`. Manda **el tipo de la variable**, decidido en tiempo de compilación. Eso es hiding.

La regla en una frase, que es la que hay que llevarse a una entrevista: **en el override decide el objeto en tiempo de ejecución; en el hiding decide la referencia en tiempo de compilación**.

Y de ahí sale la recomendación práctica: **nunca declares en una subclase un método `static` con la misma firma que uno de la superclase**. No aporta nada y produce código cuyo comportamiento depende de cómo esté escrita la variable, no de qué objeto haya. Jon Skeet, en el hilo de Stack Overflow sobre este tema, lo dice sin rodeos: «I would recommend against declaring a static method in a subclass with the same name as a superclass static method anyway, to be honest. It's just going to lead to confusion».

**Un aviso sobre `@Override`.** La anotación **no compila** sobre un método `static` que oculta a otro:

```
error: method does not override or implement a method from a supertype
```

Es una red de seguridad útil: si creías estar sobrescribiendo y el compilador te rechaza el `@Override`, es que estabas ocultando.

## 38. Los campos también se ocultan

Lo mismo pasa con los campos, y aquí no hay ni siquiera la opción de sobrescribir: **los campos nunca se sobrescriben, solo se ocultan**, sean `static` o no.

```java
class Padre { static String etiqueta = "padre"; }
class Hijo extends Padre { static String etiqueta = "hijo"; }
```

```java
System.out.println(Padre.etiqueta);   // padre
System.out.println(Hijo.etiqueta);    // hijo
```

Cada clase tiene su propio campo, y el nombre coincide. Con referencias de instancia se vuelve directamente engañoso:

```java
Hijo h = new Hijo();
Padre p = h;

System.out.println(p.etiqueta);       // "padre"  -- manda el tipo de la variable
System.out.println(h.etiqueta);       // "hijo"
```

El mismo objeto, dos valores distintos, según cómo hayas declarado la variable. Es un anti-patrón sin excepciones: **no declares un campo con el mismo nombre que uno de la superclase**.

## 39. Las reglas que el compilador sí aplica

Aunque el hiding no sea override, el compilador aplica a los métodos ocultos **algunas** de las mismas reglas, lo cual genera mensajes de error desconcertantes.

**No se puede reducir la visibilidad:**

```java
class A { public static void foo() {} }
class B extends A { protected static void foo() {} }
```

```
error: foo() in B cannot override foo() in A
  attempting to assign weaker access privileges; was public
```

Fijate en el texto: dice **`cannot override`**, cuando técnicamente es hiding. Hay un hilo entero de Stack Overflow sobre esta confusión, y la explicación es la que dio Jon Skeet: la JLS redacta la regla como «If a method declaration d1 with return type R1 **overrides or hides** the declaration of another method d2…», y el compilador aplica una sola comprobación para los dos casos, con un solo mensaje. Es un defecto menor del mensaje de error, no un cambio de semántica.

**El tipo de retorno debe seguir siendo compatible**, con la misma regla de covarianza que en el override. También produce un mensaje que habla de `override`.

Saber esto ahorra tiempo: si ves `cannot override` sobre un método que sabés que es `static`, no estás loco. El mensaje está mal redactado; el comportamiento es hiding.

## 40. Los métodos static de una interfaz no se heredan

Desde Java 8, una interfaz puede tener métodos `static`. Y tienen una regla propia que sorprende: **no se heredan**. La documentación de Oracle lo dice en una línea perdida: «Static methods in interfaces are never inherited».

```java
interface Repo2 { static String crear() { return "x"; } }
class Impl implements Repo2 { }
```

```java
System.out.println(Impl.crear());
```

```
E7b.java:3: error: cannot find symbol
public class E7b { public static void main(String[] a) { System.out.println(Impl.crear()); } }
                                                                                ^
  symbol:   method crear()
  location: class Impl
1 error
```

`Impl.crear()` **no existe**. Hay que escribir `Repo2.crear()`, con el nombre de la interfaz.

Esto es distinto de las clases, donde `Subclase.metodoStaticDeLaSuperclase()` sí funciona. La asimetría es deliberada: una clase puede implementar varias interfaces, y si los métodos `static` se heredaran, dos interfaces con un método `static` de la misma firma producirían un conflicto irresoluble. Al no heredarse, el problema no existe.

La consecuencia práctica: los métodos `static` de interfaz son para **utilidades asociadas al concepto de la interfaz**, invocadas siempre por su nombre completo. `Comparator.comparing(...)`, `List.of(...)` y `Map.entry(...)` son exactamente eso.

## 41. static y genéricos

Un miembro `static` **no puede usar los parámetros de tipo de su clase**:

```java
class Caja<T> {
    T instancia;                 // OK
    static T compartido;         // ERROR
    static void guardar(T t) {}  // ERROR
}
```

```
E8.java:3: error: non-static type variable T cannot be referenced from a static context
    static T compartido;      // ERROR: T no existe sin instancia
           ^
E8.java:4: error: non-static type variable T cannot be referenced from a static context
    static void guardar(T t) {}
                        ^
2 errors
```

La razón encaja perfectamente con el modelo mental de la sección [2](#2-clase-frente-a-instancia-el-modelo-mental). `T` se concreta **por objeto**: en tiempo de ejecución existe una `Caja<String>` y una `Caja<Integer>`, pero **una sola clase `Caja`** con un solo campo `compartido`. ¿De qué tipo sería? No hay respuesta posible, así que el compilador lo prohíbe.

La solución cuando un método `static` necesita genéricos es **declarar su propio parámetro de tipo**, independiente del de la clase:

```java
class Caja<T> {
    T instancia;

    static <U> Caja<U> de(U valor) {         // U es del método, no de la clase
        Caja<U> c = new Caja<>();
        c.instancia = valor;
        return c;
    }
}
```

```java
Caja<String> c = Caja.de("hola");
```

El `<U>` va **antes del tipo de retorno**, y esa es la sintaxis que hay que memorizar. Es exactamente lo que hacen `List.of`, `Collections.emptyList` y `Optional.of` en la biblioteca estándar.

---

# Parte VII — Otros usos de static

## 42. import static

`import static` trae miembros `static` de otra clase para poder usarlos sin cualificar:

```java
import static java.lang.Math.PI;
import static java.lang.Math.max;
import static java.util.Arrays.asList;
```

```java
double area = PI * radio * radio;      // en vez de Math.PI
int m = max(a, b);                     // en vez de Math.max
```

También existe la forma con asterisco, `import static java.lang.Math.*`, que trae todo.

**Dónde brilla de verdad: los tests.** Sin `import static`, una aserción de JUnit es ruido:

```java
org.junit.jupiter.api.Assertions.assertEquals(esperado, real);
```

Con él:

```java
import static org.junit.jupiter.api.Assertions.*;

assertEquals(esperado, real);
```

Lo mismo con AssertJ (`assertThat`), Mockito (`mock`, `when`, `verify`) y Hamcrest. En un fichero de tests, `import static` es la norma, no la excepción.

**Dónde no usarlo.** El problema es que **destruye la pista de dónde viene cada nombre**. Al leer `max(a, b)` no sabés si es `Math.max`, un método de la clase actual o algo importado de otro sitio. Con un solo `import static java.lang.Math.*` en un fichero grande, el lector pierde el hilo.

Las reglas que funcionan:

1. Usalo cuando el nombre sea **inequívoco por sí mismo**: `assertEquals`, `PI`, `emptyList`.
2. Evitá el asterisco fuera de los tests. Importá miembro a miembro.
3. No lo uses para nombres genéricos y ambiguos: `of`, `get`, `create`, `valueOf`. `Duration.of(...)` se entiende; un `of(...)` suelto, no.

## 43. Clases de utilidad

Una clase de utilidad es una clase que solo contiene métodos `static` y no se instancia nunca. La biblioteca estándar está llena: `Math`, `Collections`, `Arrays`, `Objects`, `Files`. Baeldung las cita como el segundo uso principal de los métodos `static`: «we can use static methods to create utility or helper classes. Some popular examples are the JDK's `Collections` or `Math` utility classes, Apache's `StringUtils`, and Spring Framework's `CollectionUtils`».

La forma canónica de escribir una tiene tres piezas, y casi nadie pone las tres:

```java
public final class CadenaUtils {

    private CadenaUtils() {
        throw new AssertionError("Clase de utilidad, no se instancia");
    }

    public static boolean estaVacia(String s) {
        return s == null || s.isBlank();
    }

    public static String capitalizar(String s) {
        if (estaVacia(s)) return s;
        return Character.toUpperCase(s.charAt(0)) + s.substring(1);
    }
}
```

- **`final`** en la clase: nadie puede extenderla. Heredar de una clase de utilidad no tiene sentido, porque los métodos `static` no se sobrescriben.
- **Constructor `private`**: nadie puede instanciarla. Sin él, Java añade uno público implícito.
- **`throw` dentro del constructor**: cierra el último resquicio, la reflexión con `setAccessible(true)`.

**Cuándo una clase de utilidad es la respuesta correcta:** cuando las funciones son **puras** —mismo resultado para los mismos argumentos, sin estado, sin E/S, sin depender de nada externo—. `Math.max`, `Objects.requireNonNull` o `Collections.unmodifiableList` cumplen. Un formateador de fechas sin configuración, también.

**Cuándo no lo es:** en cuanto la clase necesita configuración, conexión, o algo que quieras sustituir en un test. `PagoUtils.cobrar(tarjeta, importe)` es una clase de utilidad solo de nombre: por dentro necesita una pasarela de pago, credenciales y una conexión de red. Eso es un servicio, y debe ser un objeto. Está desarrollado en el anti-patrón 3 de la sección [53](#53-anti-patrones).

## 44. Factorías static

Un método factoría `static` es un método que devuelve una instancia de la clase, como alternativa a llamar al constructor:

```java
public final class Temperatura {
    private final double celsius;

    private Temperatura(double celsius) { this.celsius = celsius; }

    public static Temperatura enCelsius(double c)     { return new Temperatura(c); }
    public static Temperatura enFahrenheit(double f)  { return new Temperatura((f - 32) / 1.8); }
    public static Temperatura enKelvin(double k)      { return new Temperatura(k - 273.15); }
}
```

```java
Temperatura t1 = Temperatura.enCelsius(25);
Temperatura t2 = Temperatura.enFahrenheit(77);
```

Las ventajas sobre un constructor, que es el ítem 1 de *Effective Java*:

1. **Tienen nombre.** `Temperatura.enFahrenheit(77)` se entiende; `new Temperatura(77)` no dice en qué escala está ese 77. Y como los constructores solo se distinguen por su firma, **no podrías tener los tres**: los tres reciben un `double`.
2. **No tienen que crear un objeto nuevo.** Pueden devolver uno cacheado. `Integer.valueOf(5)` devuelve siempre la misma instancia para valores pequeños; `Boolean.valueOf(true)` devuelve siempre `Boolean.TRUE`.
3. **Pueden devolver un subtipo.** `List.of(...)` devuelve implementaciones distintas según el número de elementos, sin que el que llama tenga que saberlo.

Ejemplos en la biblioteca estándar que ya usás: `List.of`, `Optional.of`, `Optional.empty`, `Integer.valueOf`, `LocalDate.of`, `Duration.ofSeconds`, `Path.of`, `Comparator.comparing`.

## 45. static en enums y records

Los dos tipos añadidos al lenguaje después de las clases tienen sus propias reglas con `static`, y ambas son consecuencia directa de lo visto.

**Enums.** Las constantes de un enum **son campos `static final` implícitos**. Esto:

```java
public enum Estado { ACTIVO, INACTIVO, SUSPENDIDO }
```

genera, por debajo, tres campos `public static final Estado`, inicializados en el `<clinit>` del enum. De ahí se deducen dos cosas:

- La creación de las constantes ocurre **antes** que cualquier bloque `static` que escribas tú en el enum, porque van primero en el orden textual.
- Un enum es, por construcción, el **singleton perezoso y seguro entre hilos** de la sección [33](#33-el-idiom-del-holder-singleton-perezoso), sin escribir nada. Por eso *Effective Java* recomienda el enum de un solo elemento como la mejor forma de implementar un singleton.

Hay una restricción que sorprende: **el constructor de un enum no puede acceder a sus propios campos `static` no constantes**, porque cuando corre el constructor esos campos todavía no se han inicializado. El compilador lo detecta:

```
error: illegal reference to static field from initializer
```

El patrón para sortearlo es una clase anidada que actúe de registro:

```java
public enum Moneda {
    EUR("€"), USD("$");

    private static class Registro {
        static final Map<String, Moneda> POR_SIMBOLO = new HashMap<>();
    }

    private final String simbolo;

    Moneda(String simbolo) {
        this.simbolo = simbolo;
        Registro.POR_SIMBOLO.put(simbolo, this);
    }

    public static Moneda porSimbolo(String s) { return Registro.POR_SIMBOLO.get(s); }
}
```

**Records.** Un record puede tener miembros `static` sin restricciones: campos, métodos y bloques. Lo que **no** puede tener son campos de instancia adicionales, porque su estado es exactamente el de sus componentes.

```java
public record Punto(int x, int y) {
    public static final Punto ORIGEN = new Punto(0, 0);      // OK

    public static Punto desdePolares(double r, double a) {   // OK: factoría static
        return new Punto((int) (r * Math.cos(a)), (int) (r * Math.sin(a)));
    }

    // private int cache;                                     // error: campo de instancia
}
```

Y un detalle relevante para la Parte V: **un record anidado es implícitamente `static`**. Escribir `static record Punto(...)` dentro de una clase es redundante pero legal. Esa implicitud es justo lo que motivó la relajación de la sección [35](#35-static-dentro-de-una-inner-class-desde-java-16).

---

# Parte VIII — Diseño, memoria y riesgos

## 46. static como raíz del recolector

Este es el hecho de rendimiento más importante de todo el capítulo, y ninguna de las tres fuentes lo menciona.

El recolector de basura trabaja partiendo de un conjunto de **raíces** (*GC roots*) y siguiendo referencias: todo lo alcanzable desde una raíz está vivo, todo lo demás se puede liberar. **Los campos `static` de las clases cargadas son raíces del recolector.**

La consecuencia es directa: **cualquier objeto alcanzable desde un campo `static` es inmortal** mientras la clase siga cargada. Y en una aplicación normal, las clases se cargan al arrancar y no se descargan nunca.

Comprobación:

```java
import java.lang.ref.WeakReference;

public class E10 {
    static byte[] enCampoStatic;
    byte[] enCampoDeInstancia;

    public static void main(String[] x) throws Exception {
        enCampoStatic = new byte[1024];
        WeakReference<byte[]> vigilaStatic = new WeakReference<>(enCampoStatic);

        E10 obj = new E10();
        obj.enCampoDeInstancia = new byte[1024];
        WeakReference<byte[]> vigilaInstancia = new WeakReference<>(obj.enCampoDeInstancia);

        obj = null;                       // la instancia queda inalcanzable
        System.gc(); Thread.sleep(200); System.gc(); Thread.sleep(200);

        System.out.println("array referenciado por campo static   -> " +
            (vigilaStatic.get() != null ? "SIGUE VIVO (no se recolecto)" : "recolectado"));
        System.out.println("array referenciado por campo instancia -> " +
            (vigilaInstancia.get() != null ? "sigue vivo" : "RECOLECTADO"));
    }
}
```

Salida real en JDK 25:

```
array referenciado por campo static   -> SIGUE VIVO (no se recolecto)
array referenciado por campo instancia -> RECOLECTADO
```

Dos arrays idénticos, creados en el mismo método, con destinos opuestos. El del campo de instancia murió con su objeto. El del campo `static` sigue ahí y seguirá hasta que termine el proceso.

Esto **no es un defecto**: es exactamente lo que querés para una constante o una caché deliberada. Se convierte en problema cuando el campo `static` acumula cosas sin límite, que es la sección siguiente.

## 47. Fugas de memoria por campos static

Con lo anterior, la fuga más común de Java se explica sola:

```java
public class Auditoria {
    private static final List<Evento> HISTORIAL = new ArrayList<>();

    public static void registrar(Evento e) {
        HISTORIAL.add(e);          // nunca se quita nada
    }
}
```

Cada evento registrado queda vivo para siempre. La aplicación funciona perfectamente en desarrollo, donde se registran veinte eventos, y se queda sin memoria en producción a los tres días. No hay excepción, no hay pista: solo un `OutOfMemoryError` cuando ya es tarde.

**Las tres formas de esta fuga que vas a encontrarte:**

1. **Colección `static` que solo crece.** El ejemplo de arriba. Caché sin política de expulsión, lista de listeners de la que nunca se da de baja nadie, mapa de sesiones que no se limpia.

2. **Fuga de class loader en servidores de aplicaciones.** Esta es más sutil y es la razón por la que existe el botón *«Find Leaks»* de Tomcat. La cadena de referencias, tal como la explicó Mark Thomas de Apache: un objeto retiene una referencia a su clase; **una clase retiene una referencia al class loader que la cargó**; y el class loader retiene una referencia a **todas** las clases que cargó. Por lo tanto, si al redesplegar una aplicación web queda una sola referencia viva a un objeto de esa aplicación desde fuera —un `ThreadLocal` no limpiado, un driver JDBC no desregistrado, un hilo de fondo que sigue corriendo—, **el class loader entero no se puede recolectar**, y con él ninguna de sus clases ni ninguno de sus campos `static`. Cada redespliegue añade una copia. Al cuarto o quinto, el servidor se cae.

3. **Objetos grandes cacheados sin querer.** Un `static Map<String, ResultadoCompleto>` que se llena con las respuestas de un endpoint, o un `static` que guarda el último objeto procesado y por tanto mantiene vivo todo su grafo de dependencias.

**Cómo evitarlas:**

| Situación | Solución |
|---|---|
| Caché | Librería con expulsión (Caffeine, Guava) o `LinkedHashMap` con `removeEldestEntry` |
| Listeners | Método de baja obligatorio, o `WeakReference` |
| `ThreadLocal` | `remove()` en un `finally`, siempre |
| Recursos en aplicación web | Cerrarlos en `contextDestroyed()` o en `destroy()` del servlet |
| Datos por petición | No guardarlos en `static`. Nunca |

Y una regla general que cubre casi todo: **un campo `static` que sea una colección mutable debería tener un tamaño máximo demostrable**. Si no podés decir cuál es su cota superior, es una fuga esperando a ocurrir.

## 48. static y concurrencia

Un campo `static` es, por definición, **compartido por todos los hilos** del proceso. Y esto tiene una consecuencia que mucha gente pasa por alto: convierte cualquier código que lo toque en código concurrente, aunque tu método parezca perfectamente secuencial.

En una aplicación web esto es la norma, no la excepción: el servidor atiende cada petición en un hilo distinto, y todos comparten los mismos campos `static`.

**El problema 1: las operaciones no atómicas.**

```java
static int contador = 0;

void incrementar() { contador++; }
```

`contador++` son tres operaciones: leer, sumar, escribir. Dos hilos pueden leer el mismo valor y escribir el mismo resultado, perdiendo un incremento. Con mil peticiones concurrentes, el contador dará menos de mil.

Soluciones, de menos a más:

```java
private static final AtomicInteger CONTADOR = new AtomicInteger();
CONTADOR.incrementAndGet();                     // atómico, sin bloqueo

private static final LongAdder CONTADOR2 = new LongAdder();
CONTADOR2.increment();                          // mejor bajo contención alta
```

**El problema 2: la visibilidad.** Aunque la operación sea atómica, un hilo puede no ver el valor escrito por otro, porque cada núcleo tiene sus cachés:

```java
static boolean parar = false;      // sin volatile

// hilo 1
while (!parar) { trabajar(); }     // puede no terminar nunca

// hilo 2
parar = true;
```

El hilo 1 puede leer indefinidamente una copia cacheada. La solución mínima es `volatile`, que garantiza que las escrituras se ven:

```java
static volatile boolean parar = false;
```

**El problema 3, el más caro: colecciones no sincronizadas.**

```java
static Map<String, Usuario> sesiones = new HashMap<>();
```

Un `HashMap` accedido por varios hilos sin sincronización puede corromperse. En versiones antiguas de Java podía incluso entrar en un bucle infinito durante el redimensionado, consumiendo una CPU al 100 % sin lanzar ninguna excepción. La solución:

```java
static final Map<String, Usuario> SESIONES = new ConcurrentHashMap<>();
```

**La regla que resume la sección:** si un campo `static` **no** es `final`, o siéndolo apunta a un objeto mutable, **tenés código concurrente y tenés que tratarlo como tal**. Las opciones seguras son tres: que sea `static final` apuntando a un objeto **inmutable**, que sea una estructura **concurrente** (`AtomicInteger`, `ConcurrentHashMap`), o que **no sea `static`**.

## 49. static y testabilidad

Este es el argumento que más peso tiene en el día a día, y el que hace que en muchos equipos `static` sea directamente sospechoso.

**Problema 1: el estado sobrevive entre tests.**

```java
public class Contador {
    private static int total = 0;
    public static void sumar() { total++; }
    public static int getTotal() { return total; }
}
```

```java
@Test void testA() { Contador.sumar(); assertEquals(1, Contador.getTotal()); }
@Test void testB() { Contador.sumar(); assertEquals(1, Contador.getTotal()); }
```

Uno de los dos falla, y **cuál falla depende del orden de ejecución**. El estado `static` no se reinicia entre tests porque la clase se carga una vez por JVM. Es la causa clásica de tests que pasan solos y fallan en la suite, o que fallan solo en el servidor de integración porque allí el orden es distinto.

**Problema 2: no se puede sustituir por un doble.**

```java
public class ServicioPedidos {
    public void procesar(Pedido p) {
        if (!PagoUtils.cobrar(p.getTarjeta(), p.getImporte())) {
            throw new PagoRechazadoException();
        }
    }
}
```

Para testear `procesar` sin cobrar de verdad, necesitás sustituir `PagoUtils.cobrar`. Y no se puede: la llamada a un método `static` se resuelve en tiempo de compilación contra la clase concreta. No hay interfaz que implementar, no hay objeto que inyectar, no hay nada que Mockito pueda interceptar por el camino normal.

Existen salidas —`Mockito.mockStatic()` desde Mockito 3.4, o PowerMock— pero son un parche. `mockStatic` requiere un `try` con recursos por cada test, tiene un coste de rendimiento notable y no funciona con hilos paralelos. Que exista la herramienta no significa que el diseño esté bien; significa que alguien tuvo que trabajar con código que no podía cambiar.

**La solución de diseño**, que es la de la sección siguiente:

```java
public class ServicioPedidos {
    private final PasarelaPago pasarela;          // interfaz, inyectada

    public ServicioPedidos(PasarelaPago pasarela) { this.pasarela = pasarela; }

    public void procesar(Pedido p) {
        if (!pasarela.cobrar(p.getTarjeta(), p.getImporte())) {
            throw new PagoRechazadoException();
        }
    }
}
```

```java
@Test
void rechazaSiLaPasarelaFalla() {
    PasarelaPago falsa = mock(PasarelaPago.class);
    when(falsa.cobrar(any(), anyDouble())).thenReturn(false);

    ServicioPedidos servicio = new ServicioPedidos(falsa);

    assertThrows(PagoRechazadoException.class, () -> servicio.procesar(unPedido()));
}
```

Sin herramientas especiales, sin estado compartido, y cada test independiente del resto.

## 50. static frente a inyección de dependencias

La pregunta de fondo: si un método `static` es más simple de escribir y de llamar, ¿por qué los frameworks modernos empujan en la dirección contraria?

Porque `static` **fija la decisión en tiempo de compilación**. Cuando escribís `PagoUtils.cobrar(...)`, estás diciendo «esta implementación exacta, para siempre, en todos los entornos». No hay forma de:

- usar una pasarela de prueba en el entorno de desarrollo y la real en producción,
- envolver la llamada para añadir reintentos o métricas,
- sustituirla en un test,
- tener dos configuraciones distintas conviviendo.

La inyección de dependencias hace lo contrario: el objeto declara **qué necesita** (una interfaz) y alguien de fuera decide **qué implementación** le da. Esa decisión se puede cambiar por entorno, por test o por configuración, sin tocar el código que la usa.

**La tabla de decisión honesta:**

| Usá `static` cuando… | Usá un objeto inyectado cuando… |
|---|---|
| La función es pura: mismos argumentos, mismo resultado | El comportamiento depende de configuración o entorno |
| No hay estado que recordar | Hay estado, conexión o recurso |
| No hay E/S, red ni base de datos | Hay E/S, red o base de datos |
| Nunca vas a querer otra implementación | Podrías querer otra implementación, aunque sea en un test |
| `Math.max`, `Objects.equals`, `String.valueOf` | `RepositorioClientes`, `PasarelaPago`, `ClienteHttp` |

**Un matiz importante para no pasarse.** Esto no significa que `static` esté mal. Significa que `static` es para **funciones**, no para **servicios**. `StringUtils.capitalizar(s)` como método `static` es correcto y siempre lo será: es una función pura sobre una cadena. `EmailUtils.enviar(destinatario, cuerpo)` como método `static` es un error de diseño desde el primer día, porque por dentro necesita un servidor SMTP, credenciales y una conexión de red.

La pregunta que resuelve casi todos los casos: **¿esto podría fallar por algo que está fuera del programa?** Si la respuesta es sí, no debería ser `static`.

## 51. Rendimiento: qué cambia de verdad

Circula la idea de que los métodos `static` son «más rápidos». Conviene ponerla en su sitio, porque es cierta en un sentido irrelevante y falsa en el que importa.

**Lo que es cierto a nivel de bytecode.** Una llamada `static` usa la instrucción `invokestatic`; una de instancia usa `invokevirtual` o `invokeinterface`. `invokestatic` no necesita resolver qué implementación llamar según el objeto, y no pasa el parámetro `this`. En teoría, menos trabajo.

**Por qué no importa en la práctica.** El compilador JIT de HotSpot **inlinea** las llamadas calientes, sean static o no. Para un método de instancia que en la práctica solo tiene una implementación cargada, el JIT aplica *monomorphic inline caching* y elimina el despacho por completo. El resultado, tras la compilación JIT, suele ser código idéntico.

Esta es exactamente la clase de medición donde el JIT engaña: al escribir el capítulo [Logical, Relational and Bitwise Operators](../01-Basics/08-logical-relational-bitwise-operators.md) apareció una medición de 0,14 ns por llamada, una cifra imposible que resultó ser el JIT sacando el trabajo entero fuera del bucle. Cualquier microbenchmark de `static` frente a instancia escrito con `System.nanoTime()` mide el optimizador, no el código. Si de verdad necesitás medirlo, se hace con JMH.

**Lo que sí tiene un coste real, y va en la dirección contraria:**

- Un campo `static` **mantiene vivo** todo lo que cuelga de él, para siempre (sección [46](#46-static-como-raíz-del-recolector)). Eso sí se nota en el perfil de memoria.
- Un bloque `static` pesado **retrasa el arranque** de la aplicación, y lo hace en el peor momento posible: durante la carga de clases, donde es difícil de perfilar. En aplicaciones serverless, donde el arranque en frío se factura, esto es dinero.
- La sincronización necesaria para usar estado `static` mutable de forma segura (sección [48](#48-static-y-concurrencia)) **sí** cuesta, y bastante más que un `invokevirtual`.

**Conclusión:** elegí `static` por diseño, nunca por rendimiento. La diferencia de velocidad de la llamada es indetectable; los efectos sobre memoria y arranque, no.

---

# Parte IX — Cierre

## 52. Casos de uso reales

Los siete lugares donde `static` es la respuesta correcta, con ejemplos de la biblioteca estándar o de código de producción normal.

**1. Constantes.**

```java
public final class Http {
    public static final int OK = 200;
    public static final int NOT_FOUND = 404;
    public static final String CABECERA_TIPO = "Content-Type";
}
```

Recordá el aviso de la sección [24](#24-la-trampa-del-despliegue-parcial): si el valor puede cambiar entre versiones de una librería publicada, no lo dejes como expresión constante.

**2. Funciones puras en clases de utilidad.**

```java
public final class Validador {
    private Validador() { }

    public static boolean esEmailValido(String s) {
        return s != null && s.matches("^[^@\\s]+@[^@\\s]+\\.[^@\\s]+$");
    }
}
```

**3. Factorías con nombre.**

```java
public static Temperatura enFahrenheit(double f) { return new Temperatura((f - 32) / 1.8); }
```

**4. El patrón Builder.** Sección [34](#34-builder-el-uso-más-frecuente). Es obligatorio que sea `static`.

**5. Loggers.**

```java
public class ServicioPedidos {
    private static final Logger log = LoggerFactory.getLogger(ServicioPedidos.class);
}
```

Es `static` porque el logger corresponde a la **clase**, no a cada objeto: crear uno por instancia sería un desperdicio y todos serían idénticos. Y es `final` porque nunca se reemplaza. Este es el idiom estándar de SLF4J y aparece en prácticamente cualquier proyecto Java.

**6. El holder para inicialización perezosa.** Sección [33](#33-el-idiom-del-holder-singleton-perezoso).

**7. Constantes precompiladas caras.**

```java
public class Parser {
    private static final Pattern FECHA = Pattern.compile("\\d{4}-\\d{2}-\\d{2}");

    public boolean esFecha(String s) { return FECHA.matcher(s).matches(); }
}
```

Compilar una expresión regular es caro; hacerlo una vez por clase en lugar de una vez por llamada es una de las optimizaciones más rentables que existen. Y `Pattern` es inmutable y seguro entre hilos, así que compartirlo no tiene riesgo. Ojo: `Matcher` **no** lo es, por eso se crea uno nuevo en cada llamada.

## 53. Anti-patrones

Los seis errores que hay que reconocer, en orden de frecuencia.

**Anti-patrón 1: el campo static mutable y público.**

```java
public class Sesion {
    public static Usuario usuarioActual;      // MAL
}
```

Es una variable global. Cualquier parte del programa puede cambiarla, no hay forma de saber quién lo hizo, y en una aplicación web con varios hilos es directamente incorrecta: dos peticiones simultáneas se pisan el usuario. **Solución:** pasá el usuario como parámetro, o usá el mecanismo de contexto de tu framework.

**Anti-patrón 2: hacer todo static para callar al compilador.**

```java
public class App {
    static String config;
    static Conexion conexion;
    static void arrancar() { }
    static void procesar() { }

    public static void main(String[] a) { arrancar(); procesar(); }
}
```

Nace de la sección [15](#15-el-error-non-static-cannot-be-referenced-from-a-static-context): el error molestaba y se resolvió poniendo `static` a todo. El resultado no es orientado a objetos, no se puede testear y no se puede tener dos configuraciones a la vez. **Solución:** `main` construye un objeto y delega; todo lo demás es código de instancia.

**Anti-patrón 3: servicios como clases de utilidad.**

```java
public class EmailUtils {
    public static void enviar(String a, String cuerpo) { /* SMTP, credenciales, red */ }
}
```

El sufijo `Utils` no convierte un servicio en una utilidad. Si por dentro hay red, disco o base de datos, es un servicio. **Solución:** interfaz más implementación inyectada (sección [50](#50-static-frente-a-inyección-de-dependencias)).

**Anti-patrón 4: la caché static sin límite.**

```java
private static final Map<String, Resultado> CACHE = new HashMap<>();
```

Sin expulsión, sin tamaño máximo y sin sincronización: tiene los tres problemas a la vez (secciones [47](#47-fugas-de-memoria-por-campos-static) y [48](#48-static-y-concurrencia)). **Solución:** Caffeine o Guava con política de expulsión, o al menos `ConcurrentHashMap` con un tamaño acotado de forma explícita.

**Anti-patrón 5: trabajo pesado en un bloque static.**

```java
static {
    conexion = DriverManager.getConnection(URL, USER, PASS);
}
```

Si falla, la clase queda inservible para siempre (sección [26](#26-cuando-la-inicialización-falla)), el error es difícil de diagnosticar y el arranque se ralentiza. **Solución:** un método de inicialización explícito, llamado cuando la aplicación arranca y puede manejar el fallo.

**Anti-patrón 6: ocultar un método static de la superclase.**

```java
class Base { static String nombre() { return "base"; } }
class Derivada extends Base { static String nombre() { return "derivada"; } }
```

Parece polimorfismo y no lo es (sección [37](#37-hiding-qué-versión-se-ejecuta)). El resultado depende de cómo esté declarada la variable. **Solución:** no lo hagas. Si necesitás comportamiento polimórfico, usá métodos de instancia.

## 54. Checklist y tabla de decisión

**Antes de escribir `static` delante de algo, tres preguntas:**

1. **¿Este dato o comportamiento pertenece a la clase o a cada objeto?** Si la respuesta contiene «depende de cuál», es de instancia.
2. **¿Podría necesitar otra implementación algún día, aunque sea solo en un test?** Si sí, no lo hagas `static`.
3. **Si es un campo: ¿es `final` y apunta a algo inmutable?** Si no, tenés estado global mutable y compartido entre hilos. Justificalo o cambialo.

**Checklist de revisión de código:**

- [ ] Todo campo `static` es `final`, salvo justificación explícita
- [ ] Los `static final` que apuntan a colecciones son inmutables (`List.of`, `Map.of`) o concurrentes
- [ ] Ningún campo `static` es `public` y mutable a la vez
- [ ] Las constantes que pueden cambiar entre versiones no son expresiones constantes
- [ ] Ninguna colección `static` crece sin cota
- [ ] Ningún bloque `static` hace E/S, red ni base de datos
- [ ] Ningún inicializador `static` referencia una subclase de su propia clase
- [ ] Los campos `static` están declarados antes de los bloques `static` que los usan
- [ ] Ninguna subclase declara un método `static` con la firma de uno de la superclase
- [ ] Las clases de utilidad son `final` y tienen constructor `private`
- [ ] Los `ThreadLocal` guardados en campos `static` se limpian con `remove()`
- [ ] Los métodos `static` que hay son funciones puras, no servicios

**Tabla de decisión rápida:**

| Quiero… | Uso |
|---|---|
| Un valor fijo compartido | `static final` con tipo inmutable |
| Un contador global | `static final AtomicInteger` (y preguntarme si lo necesito) |
| Una función pura | Método `static` en clase de utilidad `final` |
| Crear objetos con nombres claros | Factoría `static` |
| Construir un objeto complejo | Builder anidado `static` |
| Una única instancia perezosa | Holder anidado `static`, o enum |
| Un logger | `private static final Logger` |
| Una regex precompilada | `private static final Pattern` |
| Algo que hable con la red o la BD | **No `static`**: objeto inyectado |
| Algo que dependa del entorno | **No `static`**: objeto inyectado |
| Guardar datos de la petición actual | **No `static`**: parámetro o contexto del framework |

## 55. Autoevaluación

Doce preguntas. Si respondés bien diez, el tema está.

**1. ¿Qué imprime este programa y por qué?**

```java
class Config {
    static final int MAX = 100;
    static { System.out.println("init"); }
}
public class Main {
    public static void main(String[] a) { System.out.println(Config.MAX); }
}
```

<details><summary>Respuesta</summary>

Imprime solo `100`. **No imprime `init`.**

`MAX` es una *variable constante* —`final`, primitiva, inicializada con una expresión constante— así que el compilador copia el `100` directamente en `Main.class`. En el bytecode de `main` no queda ninguna referencia a `Config`, la clase nunca se inicializa y su bloque `static` no se ejecuta. Sección [23](#23-las-constantes-que-no-cargan-la-clase).

Si quitás el `final`, o cambiás el `100` por una llamada a un método, entonces sí se imprime `init`.
</details>

**2. ¿Compila esto? ¿Por qué?**

```java
class Caja<T> {
    static T contenido;
}
```

<details><summary>Respuesta</summary>

No compila: `non-static type variable T cannot be referenced from a static context`.

`T` se concreta por objeto, pero el campo `static` es uno solo para toda la clase. Habría una sola `Caja.contenido` compartida entre una `Caja<String>` y una `Caja<Integer>`, y no hay tipo posible para ella. Sección [41](#41-static-y-genéricos).
</details>

**3. ¿Qué imprime?**

```java
class Animal { static String nombre() { return "animal"; } }
class Gato extends Animal { static String nombre() { return "gato"; } }

Animal a = new Gato();
System.out.println(a.nombre());
```

<details><summary>Respuesta</summary>

Imprime `animal`.

Es *hiding*, no override. El método `static` se resuelve en tiempo de compilación según **el tipo de la variable** (`Animal`), no según el objeto (`Gato`). Con un método de instancia habría impreso `gato`. Sección [37](#37-hiding-qué-versión-se-ejecuta).
</details>

**4. ¿Lanza `NullPointerException`?**

```java
Servicio s = null;
s.metodoStatic();
```

<details><summary>Respuesta</summary>

No. Funciona con normalidad.

El compilador ve que el método es `static`, descarta la referencia y emite `invokestatic` contra la clase. La variable solo sirvió para deducir de qué clase hablabas; su valor nunca se usa. Sección [16](#16-llamar-a-un-método-static-sobre-una-referencia-nula).
</details>

**5. ¿En qué orden se ejecuta?**

```java
class X {
    static { System.out.println("bloque 1"); }
    static int a = imprimir("campo a");
    static { System.out.println("bloque 2"); }
}
```

<details><summary>Respuesta</summary>

`bloque 1`, `campo a`, `bloque 2`. **Orden textual estricto**, intercalando campos y bloques.

Esto desmiente la afirmación de freeCodeCamp de que los bloques «se ejecutan inmediatamente después de la declaración de las variables static». Sección [21](#21-el-orden-textual-manda).
</details>

**6. ¿Qué pasa la segunda vez que se accede a una clase cuyo bloque `static` lanzó una excepción?**

<details><summary>Respuesta</summary>

La primera vez se obtiene `ExceptionInInitializerError` con la excepción original como causa. **La segunda vez y todas las siguientes, `NoClassDefFoundError`**, porque la JVM marcó la clase como errónea y no vuelve a intentar inicializarla.

No hay forma de recuperarse sin reiniciar la JVM. Por eso nunca hay que capturar y descartar estos errores. En JDK 25, a diferencia de versiones antiguas, el `NoClassDefFoundError` sí trae la causa original encadenada. Secciones [26](#26-cuando-la-inicialización-falla) y [27](#27-la-clase-en-estado-erróneo).
</details>

**7. ¿Por qué el holder idiom es perezoso y seguro entre hilos sin usar `synchronized`?**

<details><summary>Respuesta</summary>

**Perezoso**: la clase `Holder` es una clase aparte, y solo se inicializa cuando alguien lee `Holder.INSTANCIA`. Inicializar la clase externa no la toca.

**Seguro**: la JVM garantiza que `<clinit>` lo ejecuta un único hilo, protegido por el bloqueo de inicialización de la clase. Esa garantía es de la especificación, no del programador.

**Sin coste**: una vez inicializada, leer el campo es una lectura normal, sin bloqueo. Sección [33](#33-el-idiom-del-holder-singleton-perezoso).
</details>

**8. ¿Por qué este código puede colgar la aplicación?**

```java
class Base {
    private static final Sub INSTANCIA = new Sub();
    static class Sub extends Base { }
}
```

<details><summary>Respuesta</summary>

Deadlock de inicialización. Para inicializar `Base` hay que inicializar `Sub` (por el `new`); para inicializar `Sub` hay que inicializar `Base` (es su superclase). Con un hilo funciona por reentrada; con dos hilos entrando por lados distintos, cada uno tiene un bloqueo y espera el otro.

Lo peor: los hilos quedan en estado `RUNNABLE`, y `ThreadMXBean.findDeadlockedThreads()` devuelve `null`. **El detector de deadlocks de la JVM no lo ve.** Le pasó a JavaPoet en producción. Sección [28](#28-deadlock-de-inicialización).
</details>

**9. ¿Cuál es la diferencia real de memoria entre una clase interna y una anidada `static`?**

<details><summary>Respuesta</summary>

Menos de la que se suele decir. `javac` **solo genera el campo `this$0` si la clase interna realmente usa algo del objeto externo**; si no lo usa, el objeto ocupa lo mismo que una anidada `static`.

La diferencia que sí existe siempre: el **constructor** de la interna exige una instancia del externo aunque no la guarde. Y cuando sí guarda `this$0`, mantiene vivo el objeto externo entero mientras la interna viva, que es un problema de retención, no de tamaño. Sección [31](#31-la-referencia-oculta-al-objeto-externo).
</details>

**10. ¿Por qué `Impl.crear()` no compila si `Impl` implementa una interfaz con un método `static crear()`?**

<details><summary>Respuesta</summary>

Porque **los métodos `static` de una interfaz no se heredan**. Hay que invocarlos por el nombre de la interfaz: `MiInterfaz.crear()`.

La asimetría con las clases es deliberada: una clase puede implementar varias interfaces, y si se heredaran, dos interfaces con la misma firma producirían un conflicto sin solución. Sección [40](#40-los-métodos-static-de-una-interfaz-no-se-heredan).
</details>

**11. Cambiás `public static final int TIMEOUT = 30` a `60` en una librería y desplegás solo el jar de la librería. ¿Qué ve la aplicación?**

<details><summary>Respuesta</summary>

Sigue viendo **30**. El valor se copió dentro del `.class` de la aplicación cuando se compiló contra la versión antigua, y allí sigue.

No hay error ni aviso: solo un valor obsoleto. Hay que recompilar la aplicación, o evitar que la constante sea una expresión constante. Sección [24](#24-la-trampa-del-despliegue-parcial).
</details>

**12. ¿Cuándo es correcto un método `static` y cuándo es un error de diseño?**

<details><summary>Respuesta</summary>

Es correcto cuando la función es **pura**: el resultado depende solo de los argumentos, no hay estado, no hay E/S y nunca vas a querer otra implementación. `Math.max`, `Objects.requireNonNull`, `String.valueOf`.

Es un error cuando esconde un **servicio**: algo que habla con la red, el disco o la base de datos, que depende de configuración, o que querrías sustituir en un test. `EmailUtils.enviar(...)` es el ejemplo canónico.

La pregunta que lo resuelve: **¿esto podría fallar por algo que está fuera del programa?** Si sí, no debería ser `static`. Sección [50](#50-static-frente-a-inyección-de-dependencias).
</details>

## 56. Fuentes

Las fuentes se listan con lo que aportan y **con sus errores señalados**. Todo lo marcado como error se comprobó en JDK 25 (Temurin 25.0.3+9).

### Fuentes primarias

- **[Java Language Specification, Java SE 25 — §12.4, *Initialization of Classes and Interfaces*](https://docs.oracle.com/javase/specs/jls/se25/html/jls-12.html)**. La referencia normativa de toda la Parte IV. **§12.4.1** enumera las cinco situaciones que disparan la inicialización, incluida la cláusula «and the field is not a constant variable» que desmiente a Jenkov; **§12.4.2** describe el procedimiento paso a paso, incluida la frase «For each class or interface C, there is a unique initialization lock LC» que explica el deadlock de la sección 28 y la garantía del holder idiom.
- **JLS §4.12.4, *final Variables*.** Define *variable constante*: `final`, de tipo primitivo o `String`, inicializada con una expresión constante. Es la definición exacta que determina qué se inlinea y qué no.
- **JLS §8.7, *Static Initializers*** y **§8.3.2, *Initialization of Fields*.** Establecen que los inicializadores de campo y los bloques `static` se ejecutan «in textual order, as though they were a single block», lo que desmiente a freeCodeCamp.
- **JLS §8.4.8, *Inheritance, Overriding, and Hiding*.** Define el hiding y la regla de §8.4.8.3 redactada como «overrides **or hides**», causa del mensaje de error engañoso de la sección 39.
- **JLS §8.1.3, *Inner Classes and Enclosing Instances*.** Contiene la frase «Prior to Java SE 16, an inner class could not declare static initializers, and could only declare static members that were constant variables», que fecha el cambio de la sección 35.
- **[JEP 395: Records](https://openjdk.org/jeps/395)** y **[Local and Nested Static Declarations](https://docs.oracle.com/en/java/javase/16/docs/specs/local-statics-jls.html)**. Documentan la relajación que permite miembros `static` dentro de clases internas desde Java 16.
- **[JEP 122: Remove the Permanent Generation](https://openjdk.org/jeps/122)**. La base del matiz histórico de la sección 10: por qué antes de Java 8 los campos `static` no estaban en el heap.
- **[JEP 512: Compact Source Files and Instance Main Methods](https://openjdk.org/jeps/512)**. El `main` de instancia, definitivo en Java 25, que quita a `main` la obligación de ser `static`.
- **[Overriding and Hiding Methods — The Java Tutorials](https://docs.oracle.com/javase/tutorial/java/IandI/override.html)**. Fuente de la tabla de la sección 36 y de la frase «Static methods in interfaces are never inherited».

### Fuentes que se pidieron para este capítulo

- **[freeCodeCamp — Java Static Keyword Explained With Examples](https://www.freecodecamp.org/news/java-static-keyword-explained/)**. La más breve de las tres y la mejor como primer contacto: cubre los cuatro usos de `static` con ejemplos mínimos que compilan, y su ejemplo del contador es el más claro de los tres. **Tres problemas:**
  - **Afirmación falsa:** «Static code blocks […] are executed immediately after declaration of static variables». No es así: se ejecutan **en orden textual, intercalados** con los inicializadores de campo. Demostrado en la sección 21 con salida real; si la afirmación fuera cierta, la salida sería otra.
  - **Imprecisión:** «Static method can not use non-static members (variables or functions) of the class». Sí puede, a través de una referencia a un objeto; lo que no puede es usarlos **directamente**. Baeldung lo formula correctamente. Sección 12.
  - **Erratas:** el método se llama `increment()` en el código y `increament()` en el texto que lo explica. Además, su ejemplo `Saturn.MOON_COUNT = 62` usa un blank static final sin nombrarlo ni explicar por qué compila sin inicializador, que es justo lo que la regla de Jenkov haría imposible.
- **[Jenkov — Java Fields](https://jenkov.com/tutorials/java/fields.html)**. Su sintaxis de declaración de campo (`[access_modifier] [static] [final] type name [= initial value]`) es la mejor referencia compacta que hay, y la separación entre campos static y no static con diagramas es didáctica. **Dos afirmaciones incorrectas**, ambas centrales para este capítulo:
  - **Falsa:** «A final field **must** have an initial value assigned to it». Los *blank finals* lo desmienten: un campo `final` puede declararse sin valor y asignarse una vez en un bloque `static` o en el constructor. Sección 9. Es el mismo error que ya se documentó en el capítulo [Attributes and Methods](02-attributes-and-methods.md).
  - **Imprecisa y falsa en su segunda mitad:** «Static fields are created when the class is loaded. **A class is loaded the first time it is referenced in your program**». Confunde carga con inicialización —los campos se crean en la *preparación*, con valores por defecto, y se inicializan después— y la segunda frase es directamente falsa: referenciar una variable constante **no carga la clase**, como demuestra el bytecode de la sección 23. La página está fechada en 2015.
  - Nota de navegación: el tutorial de Jenkov tiene páginas dedicadas a [Nested Classes](https://jenkov.com/tutorials/java/nested-classes.html) y [Access Modifiers](https://jenkov.com/tutorials/java/access-modifiers.html) enlazadas desde esta, que cubren la Parte V y el capítulo anterior respectivamente.
- **[Baeldung — A Guide to the Static Keyword in Java](https://www.baeldung.com/java-static)**. La más completa de las tres y la única que cubre los cuatro usos con profundidad. Aporta dos cosas que las otras no tienen: **la lista de las cuatro combinaciones de acceso** (sección 13), que es correcta y muy útil, y una sección dedicada al error «Non-static variable cannot be referenced from a static context». Su ejemplo `Pizza` **produce exactamente la salida que publica**, comprobado. **Tres carencias:**
  - **Explicación ausente:** publica la salida del ejemplo `Pizza` sin explicar por qué las líneas salen en ese orden. La causa —que la cuarta línea construye un `Pizza` cuyo constructor imprime las líneas dos y tres antes de que la concatenación termine— es lo único interesante del ejemplo y no aparece. Sección 11.
  - **Matiz faltante:** «static variables are stored in the heap memory» es cierto solo **desde Java 8**; antes vivían en el PermGen. Y lo que está con la clase es la *variable*, no el objeto apuntado. Sección 10.
  - **Argumento débil:** al recomendar clases anidadas `static` dice «they won't require any heap or stack memory». `javap` demuestra que `javac` ya elimina el campo `this$0` cuando la interna no usa el externo, así que el ahorro de memoria es nulo en ese caso; el argumento real y más fuerte es el acoplamiento del constructor. Sección 31.
  - El artículo **no menciona en ningún momento** la inicialización perezosa por constantes, el fallo de `<clinit>`, el deadlock de inicialización ni el cambio de Java 16, pese a estar actualizado en enero de 2024.

### Discusiones de comunidad consultadas

- **[Why reference to a static final field will not trigger class loading?](https://stackoverflow.com/questions/22650464/why-reference-to-a-static-final-field-will-not-trigger-class-loading)**. Cita la JLS §12.4 y §4.12.4 y resume el porqué: «The disadvantage is the confusion it might cause. The advantage is that referring to constants doesn't cause any unnecessary execution of code».
- **[Static block in Java not executed](https://stackoverflow.com/questions/16853747/static-block-in-java-not-executed)**. El caso práctico completo, con el `javap` que muestra el `sipush 9090` inlineado, y las dos formas de romper la constancia: quitar `final` o usar una expresión no constante.
- **[Why doesn't Java allow overriding of static methods?](https://stackoverflow.com/questions/2223386/why-doesnt-java-allow-overriding-of-static-methods)**. Origen de la lectura de la sección 4: «*static* es un antónimo de *dinámico*». También aporta el contexto de diseño: Java se dirigía a programadores de C++ y buscaba evitar las críticas de lentitud que había recibido Smalltalk.
- **[Are static methods inherited in Java?](https://stackoverflow.com/questions/10291949/are-static-methods-inherited-in-java)** y **[Overriding vs Hiding Java — Confused](https://stackoverflow.com/questions/10594052/overriding-vs-hiding-java-confused)**. Aclaran la distinción que casi todos los tutoriales se saltan: los métodos `static` **sí** se heredan, **no** se sobrescriben.
- **[Strange case of static method override in java](https://stackoverflow.com/questions/10297252/strange-case-of-static-method-override-in-java)**. Explica el mensaje de error engañoso de la sección 39 y recoge la recomendación de Jon Skeet de no declarar nunca un `static` con la firma de uno de la superclase.
- **[Class initialization deadlock mechanism explanation](https://stackoverflow.com/questions/53682182/class-initialization-deadlock-mechanism-explanation)**. Recorre paso a paso el procedimiento de la JVMS §5.5 aplicado al deadlock, indicando en qué paso concreto se bloquea cada hilo.
- **[Deadlock that you can't avoid](https://kohsuke.org/2010/09/01/deadlock-that-you-cant-avoid/)** (Kohsuke Kawaguchi, creador de Jenkins). El caso original, encontrado en Hudson. Su thread dump muestra los hilos en estado `RUNNABLE` dentro de `<clinit>`, que es la pista que se confirmó en la sección 28.
- **[Using static analysis to prevent Java class initialization deadlocks](https://blog.palantir.com/using-static-analysis-to-prevent-java-class-initialization-deadlocks-c2f31ca967d6)** (Palantir) y la regla **[ClassInitializationDeadlock](https://errorprone.info/bugpattern/ClassInitializationDeadlock)** de Error Prone. El caso real de JavaPoet y las tres formas de romper el ciclo. Referencian **[JDK-8037567](https://bugs.openjdk.org/browse/JDK-8037567)**, donde OpenJDK confirma que es comportamiento esperado y debe evitarse en el código de aplicación.
- **[Type initializer circular dependencies](https://codeblog.jonskeet.uk/2012/04/07/type-initializer-circular-dependencies/)** (Jon Skeet). Sobre C#, pero el mecanismo es idéntico al de Java y la descripción es insuperable: «Pretty much your classic Heisenbug». Fuente de la sección 29, incluida la observación de que la suite completa puede fallar y el test aislado pasar.
- **[When and why does Java read an uninitialised final static variable without compiler or runtime errors?](https://stackoverflow.com/questions/42505867/when-and-why-does-java-read-an-uninitialised-final-static-variable-without-compi)**. El caso de `SIDE` valiendo `0.0`, y el detalle de que quitar el `new Random()` hace desaparecer el síntoma porque convierte el campo en una expresión constante.
- **[How to analyse a NoClassDefFoundError caused by an ignored ExceptionInInitializerError?](https://stackoverflow.com/questions/2210720/how-to-analyse-a-noclassdeffounderror-caused-by-an-ignored-exceptionininitialize)**. El hilo clásico sobre el diagnóstico imposible, con la recomendación de evitar inicializadores `static` y el agente Java como último recurso. **Parcialmente obsoleto:** el JDK ya encadena la causa, como se comprobó en la sección 27.
- **[JDK-8048190: NoClassDefFoundError omits original ExceptionInInitializerError](https://bugs.openjdk.org/browse/JDK-8048190)**. El bug que documenta la mejora, con ejemplos del mensaje antiguo y del nuevo.
- **[Diagnosing and Fixing Memory Leaks in Web Applications](https://tomcat.apache.org/presentations/2010-08-05-javaone-Memory-Leaks-60mins.pdf)** (Mark Thomas, Apache Tomcat, JavaOne 2010). La cadena de referencias de la sección 47: objeto → clase → class loader → todas sus clases. Sigue siendo la mejor explicación de por qué una sola referencia perdida tumba un redespliegue.
- **[Static variables, Tomcat and memory leaks](https://stackoverflow.com/questions/19619953/static-variables-tomcat-and-memory-leaks)** y **[MemoryLeakProtection](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=103099543)** (wiki de Tomcat). Casos reales y las contramedidas que Tomcat aplica, incluida la de poner a `null` los campos `static` de las clases del webapp al parar la aplicación.
- **[ClassLoader Leaks in Hot-Reload Environments](https://blog.heaphero.io/java-classloader-leaks/)**. La checklist de limpieza de la sección 47: `ThreadLocal.remove()`, parar hilos, desregistrar drivers JDBC, cerrar executors.
- **Joshua Bloch, *Effective Java*.** Ítem 1 («Consider static factory methods instead of constructors») para la sección 44, e ítem 3 («Enforce the singleton property with a private constructor or an enum type») para las secciones 33 y 45.

### Verificación

Todos los ejemplos, salidas y mensajes de error de este documento se ejecutaron en:

```
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9 (build 25.0.3+9-LTS)
```

Los mensajes de `javac` se reproducen literalmente. Se compilaron catorce ficheros: para comprobar lo que **sí** compila (blank static final, llamada static sobre referencia nula, miembros static dentro de una inner class, record anidado en inner class, método static en interfaz) y para capturar el texto exacto de los errores (variable local `static`, `T` en contexto static, método static de interfaz invocado desde la implementación, asignación doble de un blank final). El ejemplo `Pizza` de Baeldung se copió y ejecutó literalmente para contrastar su salida publicada.

Las mediciones de comportamiento se hicieron con: `javap -c -p` para demostrar el inlining de constantes y la ausencia de `getstatic`; `javap -p` y `javap -v` para comparar los campos sintéticos de clases internas y anidadas; `java.lang.ref.WeakReference` con dos ciclos de `System.gc()` para comprobar la retención por campo `static`; y `java.lang.management.ThreadMXBean` (`findDeadlockedThreads` y `findMonitorDeadlockedThreads`) para verificar que el deadlock de inicialización no es detectable por la API estándar.
