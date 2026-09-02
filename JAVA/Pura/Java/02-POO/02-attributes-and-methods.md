# Attributes and Methods

> **Bloque:** `02-POO` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25

**Alcance de este documento.** El capítulo anterior, [Classes and Objects](01-classes-and-objects.md), presentó la clase como la unidad que junta datos y comportamiento, y mostró de pasada que esos datos se llaman campos y ese comportamiento, métodos. Este capítulo se dedica entero a esas dos piezas: **cómo se declaran, qué modificadores admiten, cómo se accede a ellas, cómo se resuelven las llamadas y qué reglas del lenguaje producen los bugs más difíciles de ver**. Da por sabidos los tipos ([Data Types and Variables](../01-Basics/03-data-types-and-variables.md)), el ámbito de una variable ([Variables and Scopes](../01-Basics/04-variables-and-scopes.md)) y la mecánica básica de invocar un método ([Methods and Parameters](../01-Basics/12-methods-and-parameters.md)).

**Por qué este tema separa a un junior de un mid.** Casi todo el mundo sabe escribir `private String nombre;` y `public String getNombre()`. Lo que no sabe casi nadie con menos de dos años es que **los campos no son polimórficos y los métodos sí**, que **el tipo de retorno no forma parte de la firma**, que la sobrecarga se resuelve con el tipo *estático* de los argumentos, que `protected` no significa lo que dice la documentación oficial de Oracle, o que un getter que devuelve la lista interna deja tu objeto abierto a que cualquiera lo modifique desde fuera. Cada una de esas afirmaciones está demostrada aquí con código ejecutado en JDK 25 y con el mensaje literal del compilador.

**Lo que NO entra aquí**, porque tiene documento propio en este bloque: los modificadores de acceso a fondo (`03-Access Specifiers`), `static` y `final` como temas completos (`04` y `05`), los bloques de inicialización y el ciclo de vida (`06`), el paso por valor frente a referencia (`07`) y la encapsulación como principio de diseño (`08`). Aquí aparecen solo en la medida en que hacen falta para declarar y usar correctamente un atributo o un método.

---

## Índice

**Parte I — El lío terminológico: atributo, campo y property**

1. [Por qué este capítulo empieza por las palabras](#1-por-qué-este-capítulo-empieza-por-las-palabras)
2. [Lo que dice la especificación del lenguaje](#2-lo-que-dice-la-especificación-del-lenguaje)
3. [Campo, variable local y parámetro](#3-campo-variable-local-y-parámetro)
4. [Property es un par de métodos, no un dato](#4-property-es-un-par-de-métodos-no-un-dato)
5. [El falso amigo java.util.Properties](#5-el-falso-amigo-javautilproperties)

**Parte II — Declarar atributos**

6. [Anatomía de una declaración de campo](#6-anatomía-de-una-declaración-de-campo)
7. [Los valores por defecto y por qué las variables locales no los tienen](#7-los-valores-por-defecto-y-por-qué-las-variables-locales-no-los-tienen)
8. [Campos de instancia frente a campos estáticos](#8-campos-de-instancia-frente-a-campos-estáticos)
9. [final y los blank finals](#9-final-y-los-blank-finals)
10. [final no significa inmutable](#10-final-no-significa-inmutable)
11. [static final y las constantes de verdad](#11-static-final-y-las-constantes-de-verdad)
12. [transient y volatile](#12-transient-y-volatile)
13. [El orden de inicialización y la referencia adelantada ilegal](#13-el-orden-de-inicialización-y-la-referencia-adelantada-ilegal)
14. [Cuándo algo NO debe ser un campo](#14-cuándo-algo-no-debe-ser-un-campo)

**Parte III — Acceder a los atributos**

15. [Los cuatro niveles de acceso aplicados a campos](#15-los-cuatro-niveles-de-acceso-aplicados-a-campos)
16. [La letra pequeña de protected](#16-la-letra-pequeña-de-protected)
17. [Por qué un campo público es casi siempre un error](#17-por-qué-un-campo-público-es-casi-siempre-un-error)
18. [Shadowing y la palabra this](#18-shadowing-y-la-palabra-this)
19. [Field hiding: los campos no son polimórficos](#19-field-hiding-los-campos-no-son-polimórficos)

**Parte IV — Declarar métodos**

20. [Anatomía de una declaración de método](#20-anatomía-de-una-declaración-de-método)
21. [La firma: qué entra y qué queda fuera](#21-la-firma-qué-entra-y-qué-queda-fuera)
22. [Parámetros y argumentos](#22-parámetros-y-argumentos)
23. [Java pasa siempre por valor](#23-java-pasa-siempre-por-valor)
24. [Parámetros final y la reasignación de parámetros](#24-parámetros-final-y-la-reasignación-de-parámetros)
25. [Tipo de retorno, void y varios return](#25-tipo-de-retorno-void-y-varios-return)
26. [Varargs](#26-varargs)
27. [Métodos de instancia frente a métodos estáticos](#27-métodos-de-instancia-frente-a-métodos-estáticos)
28. [throws forma parte de la declaración](#28-throws-forma-parte-de-la-declaración)

**Parte V — Sobrecarga**

29. [Qué es sobrecargar y qué no lo es](#29-qué-es-sobrecargar-y-qué-no-lo-es)
30. [Las tres fases de la resolución de sobrecarga](#30-las-tres-fases-de-la-resolución-de-sobrecarga)
31. [Manda el tipo estático, no el objeto real](#31-manda-el-tipo-estático-no-el-objeto-real)
32. [null y la llamada ambigua](#32-null-y-la-llamada-ambigua)
33. [Cuándo la sobrecarga hace daño](#33-cuándo-la-sobrecarga-hace-daño)

**Parte VI — Atributos y métodos juntos: los accesores**

34. [El par getter y setter automático no es encapsulación](#34-el-par-getter-y-setter-automático-no-es-encapsulación)
35. [Copias defensivas](#35-copias-defensivas)
36. [La convención JavaBeans y sus reglas exactas](#36-la-convención-javabeans-y-sus-reglas-exactas)
37. [Records: accesores sin get](#37-records-accesores-sin-get)
38. [Method chaining y APIs fluidas](#38-method-chaining-y-apis-fluidas)

**Parte VII — Diseño**

39. [Tell, dont ask](#39-tell-dont-ask)
40. [Métodos cortos y una sola responsabilidad](#40-métodos-cortos-y-una-sola-responsabilidad)
41. [Cuántos campos son demasiados](#41-cuántos-campos-son-demasiados)
42. [Nombrar campos y métodos](#42-nombrar-campos-y-métodos)

**Parte VIII — Qué pasa por debajo**

43. [Dónde vive cada campo](#43-dónde-vive-cada-campo)
44. [El coste real de un getter](#44-el-coste-real-de-un-getter)
45. [static no significa más rápido](#45-static-no-significa-más-rápido)

**Parte IX — Cierre**

46. [Casos de uso reales](#46-casos-de-uso-reales)
47. [Anti-patrones](#47-anti-patrones)
48. [Checklist y tabla de decisión](#48-checklist-y-tabla-de-decisión)
49. [Autoevaluación](#49-autoevaluación)
50. [Fuentes](#50-fuentes)

---

# Parte I — El lío terminológico: atributo, campo y property

## 1. Por qué este capítulo empieza por las palabras

Este es el único tema de Java donde la confusión no viene de que el concepto sea difícil, sino de que **tres palabras distintas se usan para cosas parecidas pero no iguales**, y cada tutorial elige una sin avisar de que los demás usan otra.

W3Schools titula su página *Java Class Attributes* y abre diciendo: «In Java, variables declared inside a class are called "attributes"». Jenkov titula la suya *Java Fields* y dice: «A Java field is a variable inside a class». Spring, Hibernate y Jackson hablan todo el rato de *properties*. Y el enunciado de este capítulo se llama *Attributes and Methods*.

¿Son sinónimos? **No del todo, y la diferencia importa** en cuanto tocás un framework. Vamos a fijar el vocabulario con la única fuente que no admite discusión: la especificación del lenguaje.

## 2. Lo que dice la especificación del lenguaje

La *Java Language Specification* (JLS) es el documento normativo de Java. Su capítulo 8 se titula *Classes* y es donde se define qué puede contener una clase.

Descargando el capítulo 8 de la JLS de Java SE 25 y contando apariciones de cada término, el resultado es contundente:

| Término buscado en el capítulo 8 de la JLS | Apariciones |
|---|---|
| `attribute` | **0** |
| `property` | **0** |
| `field` | **224** |
| `instance variable` | 38 |
| `class variable` | 17 |

La palabra **atributo no existe en la especificación de Java**. Tampoco *property*. El lenguaje solo conoce **campos** (*fields*), que a su vez se dividen en *variables de instancia* y *variables de clase*. La sección se llama, literalmente, `8.3. Field Declarations`.

De aquí salen tres conclusiones prácticas:

1. **"Atributo" es una palabra del dominio de la orientación a objetos en general**, heredada de UML y del análisis, no de Java. Se entiende perfectamente y no es incorrecta al hablar, pero no es la palabra del lenguaje. Cuando leas un mensaje de error del compilador, dirá *field*, nunca *attribute*.
2. **"Property" tiene un significado técnico preciso en Java, y no es el de campo.** Lo vemos en la sección 4.
3. Si alguien discute contigo sobre si algo es un atributo o un campo, la discusión es sobre inglés, no sobre Java. Como resume una de las respuestas más votadas de Stack Overflow sobre el tema: *«the words attribute and property basically aren't in it \[la JLS\]. These are english terms»*.

**En este documento** usaré **campo** cuando hable del lenguaje (que es lo que verás en los errores del compilador) y **atributo** como sinónimo coloquial cuando hable de diseño. **Property** queda reservada para su significado técnico.

## 3. Campo, variable local y parámetro

W3Schools dice: «variables declared inside a class are called attributes». Esa frase es **imprecisa de una forma que confunde a todo principiante**, porque una variable declarada dentro de un método también está, literalmente, declarada dentro de una clase, y no es un campo.

La distinción real es **dónde** se declara:

```java
public class Cuenta {

    private double saldo;          // (1) CAMPO de instancia: en el cuerpo de la clase
    private static int totales;    // (2) CAMPO estático: en el cuerpo de la clase

    public void ingresar(double importe) {   // (3) importe es un PARÁMETRO
        double comision = importe * 0.01;    // (4) comision es una VARIABLE LOCAL
        this.saldo += importe - comision;
    }
}
```

| | Dónde se declara | Cuándo nace | Cuándo muere | ¿Valor por defecto? |
|---|---|---|---|---|
| Campo de instancia | cuerpo de la clase | al crear el objeto | cuando el objeto es basura | **Sí** |
| Campo estático | cuerpo de la clase, con `static` | al cargar la clase | al descargar la clase | **Sí** |
| Variable local | dentro de un método o bloque | al ejecutarse la línea | al salir del bloque | **No** |
| Parámetro | entre los paréntesis del método | al invocar el método | al terminar el método | recibe el argumento |

La diferencia de la última columna no es un detalle: es **una regla del compilador**, y la vemos con el error exacto en la sección 7.

Solo (1) y (2) son campos. Solo ellos forman el **estado** del objeto: lo que el objeto recuerda entre una llamada y la siguiente. `comision` desaparece al terminar `ingresar()`; `saldo` sigue ahí.

> **Regla práctica.** Si el dato tiene que sobrevivir a la llamada, es un campo. Si solo sirve para el cálculo de este método, es una variable local. Convertir en campo algo que debería ser local es uno de los errores de diseño más frecuentes, y lo tratamos en la sección 14.

## 4. Property es un par de métodos, no un dato

Aquí está la distinción que de verdad paga la pena aprender, porque es la que usan **Spring, Hibernate, Jackson, JSF, Lombok y prácticamente cualquier framework de Java**.

Una **property** (propiedad) en el sentido de la especificación JavaBeans **no es un campo: es un par de métodos con un nombre que sigue un patrón**. Puede haber un campo detrás, o puede no haberlo.

Verifiquémoslo. Esta clase tiene tres campos privados y cinco métodos:

```java
public static class Bean {
    private String nombre = "n";
    private boolean activo = true;
    private Boolean borrado = Boolean.FALSE;

    public String getNombre()      { return nombre; }
    public void   setNombre(String n) { this.nombre = n; }
    public boolean isActivo()      { return activo; }
    public Boolean getBorrado()    { return borrado; }

    // property CALCULADA: no hay ningún campo llamado longitudNombre
    public int getLongitudNombre() { return nombre.length(); }
}
```

Preguntándole a `java.beans.Introspector` qué properties ve, la salida real en JDK 25 es:

```
activo[r=isActivo,w=-]
borrado[r=getBorrado,w=-]
longitudNombre[r=getLongitudNombre,w=-]
nombre[r=getNombre,w=setNombre]
```

Leé eso con atención, porque contiene tres lecciones:

1. **`longitudNombre` es una property y no existe ningún campo con ese nombre.** La property la crea el método `getLongitudNombre()`, punto. Esto es lo que permite que Jackson serialice a JSON un campo calculado que no está almacenado en ninguna parte.
2. **`activo` se detecta por `isActivo()`**, no por el campo. El patrón `isX()` solo vale para `boolean` primitivo.
3. **`borrado` se detecta por `getBorrado()`** aunque el campo sea `Boolean`. Con el envoltorio `Boolean` la convención exige `get`, no `is`. Escribir `isBorrado()` devolviendo `Boolean` hace que la property **no se detecte**, y es una fuente clásica de "por qué este campo no sale en el JSON".

En resumen:

| Concepto | Qué es exactamente | Quién lo define |
|---|---|---|
| **Campo** (*field*) | una casilla de memoria dentro del objeto o de la clase | la JLS, §8.3 |
| **Atributo** | palabra genérica de OO para "un dato de un objeto" | nadie; es inglés corriente |
| **Property** | un par `getX()`/`setX()` (o solo el getter) | la especificación JavaBeans |

## 5. El falso amigo java.util.Properties

Buscando "Java properties" es muy fácil acabar en `java.util.Properties`, que **no tiene nada que ver con los atributos de una clase**. Es una estructura de datos para leer y escribir ficheros de configuración `clave=valor`:

```java
Properties props = new Properties();
props.setProperty("email", "john@doe.com");
String email = props.getProperty("email");
```

Merece un apartado porque es un ejemplo perfecto de **cómo un mal diseño de clase envenena una API durante treinta años**, y eso es exactamente el tema de este capítulo.

`Properties` **hereda de `Hashtable<Object,Object>`** en lugar de contener uno. Consecuencia: además de `setProperty(String,String)`, expone el `put(Object,Object)` heredado, que acepta cualquier cosa. El Javadoc oficial lo admite con estas palabras: *«Their use is strongly discouraged as they allow the caller to insert entries whose keys or values are not Strings»*.

Veamos qué pasa de verdad. Este código compila sin un solo aviso:

```java
Properties props = new Properties();
props.setProperty("ok", "valor");
props.put("numero", 42);          // legal: viene de Hashtable

System.out.println(props.getProperty("numero"));
System.out.println(props.get("numero"));
props.store(new StringWriter(), "prueba");
System.out.println(props.stringPropertyNames());
```

Salida real en JDK 25:

```
getProperty("numero") -> null
get("numero") -> 42
store() -> ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String
stringPropertyNames() -> [ok]
```

Tres comportamientos venenosos en cuatro líneas:

- **`getProperty("numero")` devuelve `null`** aunque la clave existe, porque `getProperty` comprueba que el valor sea `String` y si no lo es contesta `null`. Un dato guardado que se lee como ausente.
- **`stringPropertyNames()` lo omite en silencio.** La entrada existe pero es invisible.
- **`store()` explota con `ClassCastException`**, y lo hace en el momento de guardar, que puede ser horas después de meter el dato. El objeto queda, en palabras del propio Javadoc, *"compromised"*.

La lección de diseño, que aplicaremos durante todo el capítulo: **`Properties` debería haber contenido un `Map` en un campo privado en vez de heredar de `Hashtable`**. Al heredar, se vio obligada a exponer métodos que rompen su propia invariante ("todo es `String`"). Como resume la respuesta más votada de Stack Overflow sobre el tema: *«What's now appreciated by most educated observers is that Properties should never have inherited from Map at all. It should instead wrap around Map»*.

Y no se pudo arreglar nunca por compatibilidad hacia atrás. Cuando en Java 5 se añadieron los genéricos, no pudieron declararla `Hashtable<String,String>` porque habría roto todo el código anterior que hacía `put` con no-Strings.

---

# Parte II — Declarar atributos

## 6. Anatomía de una declaración de campo

Jenkov da esta sintaxis, y es correcta como aproximación:

```
[access_modifier] [static] [final] type name [= initial value] ;
```

La forma completa que admite la JLS incluye más piezas:

```
[anotaciones] [modificador de acceso] [static] [final] [transient] [volatile] tipo nombre [= valor] ;
```

Desmontada pieza a pieza sobre un ejemplo real:

```java
public class Factura {

    @Deprecated                                     // (1) anotación
    private static final BigDecimal IVA_GENERAL      // (2)(3)(4) acceso, static, final
            = new BigDecimal("0.21");                // (7) inicializador

    private final String numero;                     // (5) tipo, (6) nombre
    private BigDecimal base;
    private transient String cacheFormateado;        // no se serializa
    private volatile boolean pagada;                 // visible entre hilos

    private int a, b, c;                             // 3 campos en una declaración
}
```

| Pieza | Obligatoria | Qué hace |
|---|---|---|
| (1) anotaciones | no | metadatos (`@Deprecated`, `@Column`, `@JsonIgnore`) |
| (2) modificador de acceso | no | quién puede verlo; sin él, acceso de paquete |
| (3) `static` | no | pertenece a la clase, no al objeto |
| (4) `final` | no | solo se le puede asignar una vez |
| (5) tipo | **sí** | qué puede guardar |
| (6) nombre | **sí** | cómo se lo nombra |
| (7) inicializador | no | valor inicial explícito |

Sobre `private int a, b, c;`: es legal declarar varios campos en una línea, pero **no lo hagas**. Con tipos array se vuelve una trampa: en `int[] x, y;` ambos son arrays, pero en `int x[], y;` solo `x` lo es.

> **El orden de los modificadores.** El compilador acepta `final static private` igual que `private static final`, pero la convención universal (y la que usa el propio JDK) es: acceso, luego `static`, luego `final`. Salirse de ella no rompe nada pero delata a alguien que no ha leído código de verdad.

## 7. Los valores por defecto y por qué las variables locales no los tienen

**Todo campo se inicializa automáticamente** aunque no le des valor. Esta clase no asigna nada:

```java
static class Defaults {
    int i; long l; double d; float f; short sh; byte by; boolean b; char c; String s; int[] arr;
}
```

Salida real al imprimir sus campos en JDK 25:

```
int=0 long=0 double=0.0 float=0.0
short=0 byte=0 boolean=false
char como entero=0  String=null  array=null
```

La tabla completa:

| Tipo | Valor por defecto |
|---|---|
| `byte`, `short`, `int`, `long` | `0` |
| `float`, `double` | `0.0` |
| `char` | el carácter de código 0 |
| `boolean` | `false` |
| cualquier referencia (`String`, arrays, tus clases) | `null` |

Nótese que el valor por defecto de `char` se imprime como un carácter invisible; por eso arriba se muestra convertido a entero, donde se ve que vale `0`.

**Las variables locales no tienen valor por defecto, y usarlas sin inicializar no compila.** Este es el contraste exacto:

```java
public class LocalSinInit {
    int campo;                       // vale 0 automáticamente
    void m() {
        int local;                   // no vale nada
        System.out.println(campo + local);
    }
}
```

Error real de `javac` en JDK 25:

```
error: variable local might not have been initialized
    void m() { int local; System.out.println(campo + local); }
                                                     ^
```

**¿Por qué esta asimetría?** No es un capricho. Un campo puede escribirse desde cualquier método y en cualquier orden, así que el compilador no puede demostrar que fue inicializado antes de leerlo; la JVM lo pone a cero al reservar la memoria para que nunca contenga basura. Una variable local vive en un fragmento de código que el compilador sí puede analizar entero, así que puede exigirte que la inicialices y avisarte cuando no lo hiciste. La regla se llama *definite assignment* y es una de las cosas que hace a Java más seguro que C.

> **Consecuencia práctica que sorprende a los juniors.** `private String nombre;` no lanza error al leerlo: devuelve `null`, y el `NullPointerException` aparece más tarde, en otro sitio, cuando alguien hace `nombre.length()`. El valor por defecto no te protege, solo retrasa el problema. Por eso validar en el constructor (capítulo anterior, sección 21) es tan importante.

## 8. Campos de instancia frente a campos estáticos

Un **campo de instancia** vive dentro de cada objeto: cada objeto tiene el suyo. Un **campo estático** vive en la clase: hay uno solo, compartido por todos los objetos y accesible sin crear ninguno.

```java
static class Contador {
    static int creados;      // uno solo, para toda la clase
    final int id;            // uno por objeto

    Contador() { creados++; this.id = creados; }
}
```

Creando tres objetos, la salida real es:

```
ids=1,2,3  Contador.creados=3
```

Cada objeto guardó su `id`, pero `creados` es el mismo contador que los tres incrementaron.

**Cómo se accede a cada uno:**

```java
Contador c = new Contador();
System.out.println(c.id);              // instancia: hace falta un objeto
System.out.println(Contador.creados);  // estático: se accede por la CLASE
System.out.println(c.creados);         // legal, pero MAL ESTILO
```

La tercera línea compila y muchos IDEs la marcan en amarillo. Funciona, pero **miente**: hace parecer que `creados` pertenece al objeto `c` cuando pertenece a la clase. Accedé siempre a lo estático por el nombre de la clase.

**Cuándo un campo debe ser estático:**

| Situación | ¿`static`? | Por qué |
|---|---|---|
| Constante compartida (`MAX_REINTENTOS`) | Sí, con `final` | no depende de ningún objeto |
| Un `Logger` por clase | Sí, `static final` | crear uno por objeto es desperdiciar memoria |
| Contador de instancias creadas | Sí | es información de la clase |
| Caché compartida mutable | **Peligroso** | es estado global; ver anti-patrón 4 |
| El nombre de un usuario | No | varía por objeto; es su estado |

> **La trampa del estado estático mutable.** Un `static` no `final` es una variable global con otro nombre. Vive mientras viva la clase, no la ve el recolector de basura, la comparten todos los hilos sin sincronización y hace que los tests se contaminen entre sí (el test B falla porque el test A dejó basura en el `static`). Los `static` legítimos son casi siempre `final` y de un tipo inmutable.

## 9. final y los blank finals

Un campo `final` **solo admite una asignación**. Después, el compilador rechaza cualquier intento:

```java
public class FinalAssign {
    final int x = 10;
    void romper() { this.x = 25; }
}
```

Error real:

```
error: cannot assign a value to final variable x
    void romper() { this.x = 25; }
                        ^
```

Hasta aquí, todas las fuentes coinciden. Ahora la parte donde **Jenkov se equivoca**. Su tutorial de campos afirma:

> «A `final` field **must have an initial value assigned to it**, and once set, the value cannot be changed again.»

Leído literalmente, eso dice que un campo `final` necesita inicializador en la declaración. **Es falso.** Java permite el llamado **blank final**: declararlo sin valor y asignarlo en el constructor. Esta clase compila y funciona:

```java
static class BlankFinal {
    final int x;                  // sin inicializador
    final List<String> items;

    BlankFinal(int x) {
        this.x = x;               // se asigna aquí, una vez
        this.items = new ArrayList<>();
    }
}
```

Salida real: `blank final x=42`.

Y no es un truco marginal: **es el patrón fundamental de las clases inmutables**. Sin blank finals no podrías escribir esto, que es como se construye cualquier objeto de valor serio:

```java
public final class Dinero {
    private final BigDecimal cantidad;
    private final Currency moneda;

    public Dinero(BigDecimal cantidad, Currency moneda) {
        if (cantidad == null || moneda == null) throw new IllegalArgumentException("nulo");
        this.cantidad = cantidad;
        this.moneda = moneda;
    }
}
```

La regla verdadera es: **un campo `final` debe estar asignado exactamente una vez cuando termina cada constructor**. Puede ser en la declaración, en un bloque de inicialización o en el constructor, pero no en dos sitios a la vez ni en ninguno.

```java
final int a = 1;                    // OK: en la declaración
final int b;   { b = 2; }           // OK: en un bloque de inicialización
final int c;   Clase() { c = 3; }   // OK: en el constructor
final int d;                        // ERROR: variable d not initialized in the default constructor
final int e = 1;  Clase() { e = 2; }  // ERROR: variable e might already have been assigned
```

## 10. final no significa inmutable

Este es probablemente el malentendido número uno sobre `final`, y W3Schools lo alimenta al decir: «The `final` keyword is useful when you want a variable to **always store the same value**, like PI».

Con un `int` es cierto. Con un objeto **no**, y la diferencia es enorme.

`final` congela **la referencia**, no el objeto al que apunta. Continuando con la clase de la sección anterior:

```java
BlankFinal bf = new BlankFinal(42);
bf.items.add("mutado");        // la lista es final... y se modifica sin problema
```

Salida real: `final List mutada=[mutado]`.

No se puede hacer `bf.items = new ArrayList<>()`, pero sí `bf.items.add(...)`. La caja está clavada al suelo; su contenido se puede cambiar.

```java
final List<String> lista = new ArrayList<>();
lista.add("hola");        // OK: muta el objeto
lista = new ArrayList<>(); // ERROR: cannot assign a value to final variable lista
```

**Cómo conseguir inmutabilidad de verdad**, que es lo que la gente cree estar consiguiendo con `final`:

```java
public final class Pedido {                    // (1) clase final: nadie la extiende
    private final List<String> lineas;         // (2) campo final

    public Pedido(List<String> lineas) {
        this.lineas = List.copyOf(lineas);     // (3) copia inmutable al entrar
    }

    public List<String> getLineas() {
        return lineas;                         // (4) ya es inmutable: seguro devolverla
    }
}
```

Los cuatro puntos son necesarios. Quitá cualquiera y la inmutabilidad se rompe: sin (3), quien te pasó la lista puede seguir modificándola; sin (4) bien hecho, quien la recibe puede modificarla. Volveremos a esto en la sección 35.

> **Nota sobre `List.copyOf`.** Devuelve una lista inmutable de verdad: al intentar `add` lanza `UnsupportedOperationException`, comprobado en la sección 35. No confundir con `Collections.unmodifiableList`, que devuelve una *vista* no modificable: si alguien conserva la lista original y la modifica, la vista refleja el cambio.

## 11. static final y las constantes de verdad

La combinación `static final` es la forma canónica de declarar una constante:

```java
public class Configuracion {
    public static final int MAX_REINTENTOS = 3;
    public static final String CODIFICACION = "UTF-8";
    public static final Duration TIMEOUT = Duration.ofSeconds(30);
}
```

**Convención de nombres:** mayúsculas con guiones bajos. Es la única excepción a `camelCase` en Java y la respetan todas las bases de código.

**Por qué `static` además de `final`:** si solo fuera `final`, cada objeto llevaría su propia copia de un valor que nunca cambia. Con `static` existe una sola.

**El detalle que casi nadie conoce: las constantes de compilación se copian en el código que las usa.** Si una constante es `static final` de tipo primitivo o `String`, y su valor se conoce en compilación, el compilador **sustituye el valor literalmente** en cada punto de uso (*constant folding*, JLS §13.1).

Consecuencia práctica muy real: si publicás una librería con `public static final int VERSION = 1;` y luego la cambiás a `2`, **el código que ya estaba compilado contra la versión 1 sigue diciendo 1** aunque actualices el `.jar`. Hay que recompilar los clientes. Es una fuente clásica de "actualicé la dependencia y el valor viejo sigue apareciendo".

La forma de evitarlo cuando el valor puede cambiar es impedir que sea constante de compilación:

```java
public static final int VERSION = Integer.parseInt("2");  // ya no es constante de compilación
```

**Y la trampa más común de todas:**

```java
public static final List<String> PAISES = new ArrayList<>(List.of("ES", "FR"));
```

Eso **no es una constante**. Cualquiera puede hacer `Configuracion.PAISES.add("XX")` y afectar a todo el programa. La forma correcta:

```java
public static final List<String> PAISES = List.of("ES", "FR");   // inmutable de verdad
```

## 12. transient y volatile

Son los dos modificadores de campo que ningún tutorial de introducción explica, y ambos aparecen en cuanto tocás código real.

### 12.1 transient

Marca un campo para que **no se serialice**. Se usa en datos que no deben viajar (secretos), que no tienen sentido fuera de este proceso (una conexión abierta) o que son caché recalculable.

```java
static class ConToken implements Serializable {
    String usuario = "ana";
    transient String token = "SECRETO";
    static String global = "estatico";
}
```

Serializando el objeto, cambiando después el estático y deserializando, la salida real es:

```
usuario=ana  token=null  static global=cambiado-despues
```

Dos hechos verificados:

- **`token` volvió como `null`**, no como `"SECRETO"`. Al deserializar, los campos `transient` reciben su valor por defecto; el inicializador de la declaración **no se vuelve a ejecutar**, porque la deserialización no llama al constructor.
- **`global` muestra `cambiado-despues`**, no `estatico`. Los campos `static` tampoco se serializan (pertenecen a la clase, no al objeto), así que lo que se ve es el valor actual de la clase, no el que había al serializar.

> **La trampa de `transient`.** Como el campo vuelve a `null` sin pasar por el constructor, cualquier objeto deserializado puede tener campos nulos que tu código da por inicializados. Si usás serialización de Java, hacés bien en revisar `readObject` o, mejor, en no usar serialización nativa: es la fuente de la mitad de las vulnerabilidades críticas de la plataforma.

### 12.2 volatile

Garantiza la **visibilidad entre hilos**: cuando un hilo escribe un campo `volatile`, cualquier otro hilo que lo lea después ve el valor nuevo. Sin `volatile`, la JVM puede cachear el valor en un registro y un hilo puede quedarse mirando eternamente un valor viejo.

```java
public class Servicio {
    private volatile boolean parado = false;   // lo escribe un hilo, lo lee otro

    public void parar() { parado = true; }
    public void bucle() { while (!parado) { trabajar(); } }
}
```

Sin `volatile`, ese `while` puede no terminar nunca aunque otro hilo haya llamado a `parar()`.

**Lo que `volatile` NO hace:** no hace atómicas las operaciones compuestas. `contador++` sobre un campo `volatile` sigue siendo insegura, porque son tres operaciones (leer, sumar, escribir) y dos hilos pueden intercalarse. Para eso hace falta `AtomicInteger` o `synchronized`. El tema completo pertenece al bloque de concurrencia; aquí basta con saber que el modificador existe y qué problema resuelve.

## 13. El orden de inicialización y la referencia adelantada ilegal

Los inicializadores de campo y los bloques de inicialización **se ejecutan en el orden textual en que aparecen**, antes del cuerpo del constructor. Comprobémoslo:

```java
static class Orden {
    int campo  = registra("1-campo");
    { registra("2-bloque"); }
    int campo2 = registra("3-campo2");
    Orden() { registra("4-constructor"); }
}
```

Salida real: `1-campo 2-bloque 3-campo2 4-constructor`

Es decir: **el orden es el del código fuente, mezclando campos y bloques**, y el constructor va siempre el último. Si movés el bloque arriba del todo, se ejecuta el primero.

De aquí sale una regla del compilador que sorprende:

```java
public class ForwardRef {
    int a = b + 1;      // usa b antes de declararlo
    int b = 2;
}
```

Error real:

```
error: illegal forward reference
    int a = b + 1;
            ^
```

**Java prohíbe leer en un inicializador un campo declarado más abajo.** Tiene sentido: en ese momento `b` todavía vale 0, así que `a` valdría 1 y no 3, y el lenguaje prefiere el error a la sorpresa silenciosa.

Curiosidad que conviene conocer porque delata el mecanismo: la prohibición es **solo para el uso simple del nombre**. Esto compila y produce `a = 1`, porque calificar con `this` desactiva la comprobación:

```java
int a = this.b + 1;   // compila; b todavía vale 0, así que a vale 1
int b = 2;
```

Es legal y es una bomba. Si ves algo así en producción, es un bug esperando.

## 14. Cuándo algo NO debe ser un campo

Convertir en campo lo que debería ser una variable local es el error de diseño más común entre juniors, y produce bugs muy difíciles de rastrear.

**Anti-ejemplo:**

```java
public class Calculadora {
    private int resultado;      // ¿por qué es un campo?
    private int temporal;       // esto es basura de un cálculo

    public int sumar(int a, int b) {
        temporal = a + b;
        resultado = temporal;
        return resultado;
    }
}
```

Problemas concretos:

1. **No es reentrante ni seguro entre hilos.** Dos hilos llamando a `sumar` a la vez se pisan `temporal`.
2. **Filtra información entre llamadas.** Después de `sumar(2,3)`, el objeto "recuerda" 5 sin razón.
3. **Hace imposible razonar sobre el objeto.** ¿Cuál es un estado válido de `Calculadora`? Ninguno definible.

Correcto:

```java
public class Calculadora {
    public int sumar(int a, int b) {
        return a + b;           // sin estado: la clase podría ser sin campos
    }
}
```

**Criterio de decisión:**

| Preguntá | Si la respuesta es... | Entonces |
|---|---|---|
| ¿Tiene que sobrevivir entre llamadas a métodos? | No | variable local |
| ¿Forma parte de la identidad o el estado del objeto? | No | variable local |
| ¿Lo usan varios métodos como dato compartido real? | No | variable local |
| ¿Es solo el resultado intermedio de un cálculo? | Sí | variable local |
| ¿Lo puse ahí para no tener que pasarlo como parámetro? | Sí | **variable local o parámetro** |

La última fila es la que más se incumple. Un campo usado como parámetro encubierto convierte un método puro en uno dependiente del orden de las llamadas.

---

# Parte III — Acceder a los atributos

## 15. Los cuatro niveles de acceso aplicados a campos

Jenkov los enumera como "cuatro posibles modificadores de acceso: public, protected, package, private". La lista de niveles es correcta, pero la palabra `package` **no es un modificador que puedas escribir**: el nivel de paquete es lo que obtenés al no poner ninguno. El propio Jenkov lo aclara después ("You don't actually write the package modifier"), pero la tabla induce a error.

| Nivel | Palabra clave | Lo ve la misma clase | El mismo paquete | Una subclase de otro paquete | Cualquiera |
|---|---|---|---|---|---|
| privado | `private` | Sí | No | No | No |
| paquete | *(ninguna)* | Sí | Sí | No | No |
| protegido | `protected` | Sí | Sí | **Sí, con matices** | No |
| público | `public` | Sí | Sí | Sí | Sí |

```java
public class Cliente {
    private   String email;      // solo dentro de Cliente
              String posicion;   // dentro del paquete
    protected String nombre;     // paquete + subclases
    public    String ciudad;     // todo el mundo
}
```

Ese ejemplo es el de Jenkov y él mismo advierte que no usarías los cuatro en la misma clase. Tiene razón, y hay que ir más lejos: **para campos, la respuesta correcta es `private` en el 95% de los casos**. Las otras tres son excepciones que hay que justificar.

## 16. La letra pequeña de protected

La documentación oficial de Oracle (*The Java Tutorials*) dice:

> «The `protected` modifier specifies that the member can only be accessed within its own package (as with package-private) and, in addition, **by a subclass of its class in another package**.»

W3Schools dice lo mismo, más corto: «The code is accessible in the same package and subclasses».

**Ambas descripciones son incompletas de una forma que hace fallar la compilación.** La subclase puede acceder al miembro protegido, sí, **pero solo a través de una referencia de su propio tipo o de un subtipo**, nunca a través de una referencia del tipo del padre.

Demostración con dos paquetes:

```java
// p1/Base.java
package p1;
public class Base { protected int secreto = 1; }

// p2/Sub.java
package p2;
import p1.Base;
public class Sub extends Base {
    void desdeSubclase() {
        System.out.println(this.secreto);   // OK
        Base otro = new Base();
        System.out.println(otro.secreto);   // ERROR
    }
}
```

Error real de `javac`:

```
error: secreto has protected access in Base
        System.out.println(otro.secreto);
                               ^
```

`Sub` **es** subclase de `Base` y está accediendo a un miembro `protected`, exactamente lo que la documentación de Oracle describe como permitido. Y no compila.

La regla verdadera está en la JLS §6.6.2:

> «A protected member or constructor of an object may be accessed from outside the package in which it is declared only by **code that is responsible for the implementation of that object**. \[...\] If the access is by a qualified name Q.Id \[...\] then the access is permitted if and only if the type of the expression Q is S or a subclass of S.»

En castellano: dentro de la subclase `S`, podés tocar el miembro protegido de **objetos que sean `S` o subclase de `S`**, no de cualquier objeto del padre.

**Por qué existe esta restricción.** La propia JLS lo explica: sin ella, cualquiera podría leer miembros protegidos de cualquier objeto con este truco: declarar una subclase `S` de `C`, escribir en ella un método que reciba un `C` y devuelva su campo protegido, y llamarlo desde fuera. `protected` sería equivalente a `public`.

> **Consecuencia práctica.** `protected` es mucho más estrecho de lo que parece y por eso los IDEs sugieren "cambiá a public" cuando choca con este límite. La respuesta correcta casi nunca es subirlo a `public`: es preguntarse por qué una clase necesita hurgar en el campo de otro objeto en lugar de pedirle que haga algo (sección 39). **Para campos, `protected` es una mala idea casi siempre**: acopla las subclases a la representación interna del padre y te impide cambiarla. Si una subclase necesita el dato, dale un método protegido, no el campo.

## 17. Por qué un campo público es casi siempre un error

W3Schools presenta los campos sin modificador de acceso y sin advertencia alguna, y sus ejemplos los usan sistemáticamente:

```java
public class Main {
  int x = 5;               // acceso de paquete, mutable, sin control
}
```

Es didácticamente cómodo y profesionalmente inaceptable. Cuatro razones concretas:

**1. Pierdes la capacidad de validar.**

```java
public class Persona { public int edad; }

p.edad = -5;      // nadie lo impide
```

Con un campo privado y un mutador podés rechazarlo. Con el campo público, cualquiera puede dejar el objeto en un estado imposible, y el fallo aparecerá muy lejos del sitio donde se causó.

**2. Pierdes la capacidad de cambiar la representación interna.** Si `public double precio` es parte de tu API, el día que descubras que necesitás `BigDecimal` (ver [Type Casting](../01-Basics/05-type-casting.md) sobre por qué `double` no sirve para dinero) tenés que romper a todos tus clientes. Con `getPrecio()` cambiás el campo por dentro y nadie se entera.

**3. No podés observar los accesos.** Ni log, ni caché, ni carga perezosa, ni disparar un evento al cambiar. Todo eso vive en el método.

**4. No es polimórfico.** Es la sección siguiente, y es la razón menos conocida y la más grave.

**La excepción real**, para no caer en el dogma: un campo `public static final` de un tipo inmutable es correcto y habitual (`Integer.MAX_VALUE`, `Duration.ZERO`). Y en una clase local de ámbito muy reducido, usada como estructura de datos de tres líneas, un campo público puede ser razonable. Hoy, además, para eso existen los `record`, que resuelven el caso sin renunciar a la inmutabilidad.

## 18. Shadowing y la palabra this

Cuando un parámetro se llama igual que un campo, **el parámetro gana** dentro de ese método: se dice que lo *ensombrece* (*shadowing*).

```java
static class Punto {
    private final int x, y;

    Punto(int x, int y) {
        this.x = x;    // this.x es el CAMPO; x a secas es el PARÁMETRO
        this.y = y;
    }
}
```

Salida real de `new Punto(3, 4)`: `x=3 y=4`.

Sin `this`, el desastre es silencioso:

```java
Punto(int x, int y) {
    x = x;    // se asigna el parámetro a sí mismo; el campo queda en 0
    y = y;
}
```

Eso **compila sin error**. Muchos compiladores dan un aviso (`self-assignment`), pero el programa arranca y el objeto queda con los campos a cero. Es un bug clásico de madrugada.

**Cuándo usar `this`:**

| Situación | ¿`this`? |
|---|---|
| Hay shadowing (parámetro y campo con el mismo nombre) | **Obligatorio** |
| No hay ambigüedad | Opcional; muchos equipos lo omiten |
| Devolver el propio objeto para encadenar | `return this;` |
| Pasar el objeto actual a otro método | `otro.registrar(this)` |

> **Sobre el estilo.** Hay equipos que exigen `this.` en todos los accesos a campos para distinguirlos de las variables locales de un vistazo. Es una decisión de equipo defendible. Lo que no es defendible es omitirlo donde hay shadowing.

## 19. Field hiding: los campos no son polimórficos

Esta es **la diferencia más importante entre un atributo y un método en Java**, y la que decide muchas preguntas de entrevista de nivel mid.

Cuando una subclase declara un campo con el mismo nombre que el padre, **no lo sobrescribe: lo oculta** (*hiding*). Los dos campos existen a la vez en el mismo objeto, y cuál se lee **lo decide el tipo de la referencia en tiempo de compilación**, no el objeto real.

```java
static class Padre {
    String etiqueta = "Padre";
    String getEtiqueta() { return "Padre"; }
}
static class Hijo extends Padre {
    String etiqueta = "Hijo";
    @Override String getEtiqueta() { return "Hijo"; }
}
```

Ahora, con una referencia de tipo `Padre` apuntando a un objeto `Hijo`:

```java
Padre p = new Hijo();
System.out.println(p.etiqueta);        // ?
System.out.println(p.getEtiqueta());   // ?
System.out.println(((Hijo) p).etiqueta);
```

Salida real en JDK 25:

```
p.etiqueta=Padre   p.getEtiqueta()=Hijo
((Hijo)p).etiqueta=Hijo
```

Leelo dos veces. **El mismo objeto, en la misma línea, dice ser `Padre` por el campo y `Hijo` por el método.**

| | Se resuelve | Cuándo | Con qué |
|---|---|---|---|
| **Campos** | estáticamente | en compilación | el **tipo de la referencia** |
| **Métodos** | dinámicamente | en ejecución | el **tipo real del objeto** |

Los métodos usan *late binding* (la JVM busca la implementación real en la tabla de métodos del objeto). Los campos no: el compilador resuelve el acceso mirando el tipo declarado, y ahí se acabó.

**Por qué esto importa de verdad**, más allá de la curiosidad:

1. **Es la razón definitiva para no usar campos públicos ni protegidos.** Un campo forma parte del contrato de una forma que no admite ser refinada por las subclases. Un getter sí.
2. **Ocultar campos es siempre un error.** No hay ningún caso legítimo en el que quieras dos campos con el mismo nombre en la misma jerarquía. Si te pasa, es que copiaste una declaración sin querer. Ponés `private` en los campos del padre y el problema desaparece.
3. **Explica bugs reales en jerarquías de frameworks.** Si extendés una clase de librería y por casualidad usás el mismo nombre de campo, no sobrescribís nada: creás un campo nuevo, y los métodos heredados del padre siguen leyendo el suyo, que nunca se actualiza.

> **Regla que podés llevarte tal cual:** *los campos se heredan, pero no se sobrescriben.* Si querés que una subclase cambie un dato, no le des acceso al campo: dale un método que lo devuelva, y que lo sobrescriba.

---

# Parte IV — Declarar métodos

## 20. Anatomía de una declaración de método

Jenkov abre su tutorial de métodos con este ejemplo:

```java
public MyClass{

    public void writeText(String text) {
        System.out.print(text);
    }
}
```

**Ese código no compila.** Le falta la palabra `class`. Error real de `javac` en JDK 25:

```
error: class, interface, annotation type, enum, record, method or field expected
public MyClass{
       ^
```

Aparece dos veces en la misma página, lleva ahí desde 2015 y es un buen recordatorio de que las fuentes hay que ejecutarlas, no citarlas. Lo correcto es `public class MyClass {`.

Vista completa de lo que puede llevar un método:

```
[anotaciones] [acceso] [static] [final] [abstract] [synchronized] [native] [<T>] tipoRetorno nombre(parámetros) [throws ...] { cuerpo }
```

Sobre un ejemplo:

```java
public class Repositorio {

    @SafeVarargs                                           // (1) anotación
    public          // (2) acceso
    static          // (3) pertenece a la clase
    final           // (4) no se puede ocultar en subclases
    <T extends Comparable<T>>                              // (5) parámetro de tipo
    List<T>         // (6) tipo de retorno
    ordenar         // (7) nombre
    (boolean descendente, T... datos)                      // (8) parámetros
    throws IllegalArgumentException {                      // (9) excepciones declaradas
        return List.of(datos);                             // (10) cuerpo
    }
}
```

| Pieza | Obligatoria | Nota |
|---|---|---|
| (1) anotaciones | no | `@Override` es la más útil: el compilador verifica que de verdad sobrescribís algo |
| (2) acceso | no | sin él, acceso de paquete |
| (3) `static` | no | se invoca sobre la clase |
| (4) `final` | no | impide sobrescribirlo |
| (5) genéricos | no | tema del bloque `04-Genericos` |
| (6) tipo de retorno | **sí** | `void` si no devuelve nada |
| (7) nombre | **sí** | verbo, en `camelCase` |
| (8) parámetros | **sí** los paréntesis | pueden estar vacíos |
| (9) `throws` | no | obligatorio para excepciones comprobadas |
| (10) cuerpo | sí, salvo `abstract`/`native` | |

## 21. La firma: qué entra y qué queda fuera

**La firma de un método son su nombre y los tipos de sus parámetros. Nada más.** No entran ni el tipo de retorno, ni los modificadores, ni las excepciones, ni los nombres de los parámetros.

Esto tiene una consecuencia inmediata y muy preguntada: **no se puede sobrecargar por tipo de retorno**.

```java
public class Overload {
    int  valor(int x) { return x; }
    char valor(int x) { return 'a'; }
}
```

Error real:

```
error: method valor(int) is already defined in class Overload
    char valor(int x) { return 'a'; }
         ^
```

**Por qué el lenguaje lo prohíbe.** Porque en Java podés ignorar el valor devuelto. Si ambos métodos existieran, esta línea sería irresoluble:

```java
valor(5);   // ¿cuál de los dos? No hay ninguna información para decidir
```

La respuesta más votada de Stack Overflow sobre el tema lo resume así: *«Because you are not required to capture the return value of a method in Java, in which case the compiler can not decide which overload to use»*.

**El matiz que separa a un senior:** esa restricción es **del lenguaje Java, no de la JVM**. En el bytecode, el descriptor de un método sí incluye el tipo de retorno, y la JVM admite perfectamente dos métodos que solo difieran en él. El propio Java lo aprovecha para implementar los *retornos covariantes* mediante métodos puente (*bridge methods*):

```java
public class Test1 {
    @Override public Test1 clone() throws CloneNotSupportedException {
        return (Test1) super.clone();
    }
}
```

En el bytecode de esa clase hay **dos** métodos `clone()`: uno que devuelve `Test1` y otro que devuelve `Object`, generado por el compilador. Difieren solo en el tipo de retorno. Por eso Scala y Kotlin pueden hacer cosas que Java no: la limitación está en la gramática, no en la máquina.

## 22. Parámetros y argumentos

W3Schools da la definición correcta y conviene fijarla porque se usan mal todo el tiempo:

> «When a parameter is passed to the method, it is called an argument.»

- **Parámetro**: la variable declarada en la firma. Es una plantilla.
- **Argumento**: el valor concreto que pasás al llamar.

```java
static void saludar(String nombre) {    // nombre es un PARÁMETRO
    System.out.println("Hola " + nombre);
}

saludar("Liam");                        // "Liam" es un ARGUMENTO
```

Los parámetros son variables locales del método: nacen al invocarlo y mueren al terminar. Tienen que coincidir en **número, orden y tipo compatible** con los argumentos.

**Cuántos parámetros son demasiados.** No hay regla del lenguaje (el límite técnico son 255), pero sí una regla de oficio: **a partir de cuatro, algo huele mal**. Dos motivos concretos:

```java
crearUsuario("Ana", "Pérez", 30, true, false, true, "ES");
```

1. **Nadie puede leer eso.** ¿Qué es el segundo `true`?
2. **Dos parámetros del mismo tipo se pueden intercambiar sin que el compilador se entere.** Si `crearUsuario(String nombre, String apellido)` recibe los argumentos al revés, compila y falla en producción.

Soluciones, en orden de preferencia:

```java
// 1. Agrupar en un objeto que tenga sentido de dominio
crearUsuario(new NombreCompleto("Ana", "Pérez"), new Preferencias(...));

// 2. Tipos propios en vez de primitivos, para que el compilador te proteja
crearUsuario(new Nombre("Ana"), new Apellido("Pérez"));

// 3. Builder, cuando hay muchos opcionales
Usuario.builder().nombre("Ana").apellido("Pérez").pais("ES").build();
```

## 23. Java pasa siempre por valor

Este tema tiene capítulo propio en el bloque (`07-Pass by Value vs Pass by Reference`), pero hace falta el resumen aquí porque afecta a cómo escribís los métodos.

**Java pasa siempre por valor. Siempre. Sin excepciones.** Lo que confunde es que, cuando el parámetro es de tipo referencia, **el valor que se copia es la referencia**, no el objeto.

```java
static void reasignar(List<String> lista) {
    lista = new ArrayList<>();      // solo cambia la copia local de la referencia
    lista.add("nuevo");
}

static void mutar(List<String> lista) {
    lista.add("nuevo");             // modifica el objeto que ambos comparten
}
```

```java
List<String> original = new ArrayList<>(List.of("a"));
reasignar(original);   // original sigue siendo [a]
mutar(original);       // original pasa a ser [a, nuevo]
```

Jenkov toca esto de pasada al hablar de parámetros `final`, y su explicación es correcta: *«if the parameter is a reference to an object, the reference cannot be changed, but values inside the object can still be changed»*.

La regla que hay que interiorizar: **un método nunca puede cambiar a qué apunta la variable de quien lo llamó, pero sí puede cambiar el contenido del objeto apuntado.** De ahí que las copias defensivas (sección 35) sean necesarias.

## 24. Parámetros final y la reasignación de parámetros

Un parámetro puede declararse `final`, lo que impide reasignarlo dentro del método:

```java
public void escribir(final String texto1, final String texto2) {
    texto1 = "otro";   // ERROR: final parameter texto1 may not be assigned
}
```

Jenkov advierte sobre reasignar parámetros, y su consejo es bueno pero tibio: *«you should be careful doing that, as it may lead to confusing code. If you think you can handle it, go ahead»*.

**El consejo profesional es más rotundo: no reasignes parámetros.** No es cuestión de si "podés manejarlo", sino de tres problemas objetivos:

1. **Pierdes el valor original.** Si a mitad del método necesitás saber qué te pasaron, ya no podés.
2. **Rompe el rastro al depurar.** El parámetro dice una cosa en la pila de llamadas y otra en el cuerpo.
3. **Confunde al lector**, que asume razonablemente que un parámetro es lo que le pasaron.

La alternativa cuesta una línea:

```java
public String normalizar(String entrada) {
    String resultado = entrada;        // copia local con nombre propio
    if (resultado == null) resultado = "";
    return resultado.trim().toLowerCase();
}
```

**¿Poner `final` en todos los parámetros?** Es una decisión de equipo. A favor: documenta la intención y el compilador la impone. En contra: añade ruido a cada firma. Muchos equipos modernos prefieren no escribirlo y confiar en el análisis del IDE. Lo importante es la disciplina, no la palabra.

## 25. Tipo de retorno, void y varios return

Un método devuelve **como máximo un valor**, de un solo tipo. Si no devuelve nada, el tipo es `void`.

```java
public int sumar(int a, int b) { return a + b; }
public void registrar(String s) { System.out.println(s); }   // sin return
```

**Varios `return` en un método son legales y a menudo mejores.** Jenkov lo explica bien: solo se ejecuta uno, y en cuanto se ejecuta el método termina.

```java
public String concatenar(String a, String b, boolean invertido) {
    if (invertido) return b + a;
    return a + b;
}
```

Existe una vieja escuela que exige un único punto de salida. En Java moderno, **las salidas tempranas producen código más plano y legible** que anidar `if`:

```java
// Con salidas tempranas: 1 nivel de anidamiento
public BigDecimal calcular(Pedido p) {
    if (p == null) throw new IllegalArgumentException("pedido nulo");
    if (p.estaVacio()) return BigDecimal.ZERO;
    if (p.esExento()) return p.base();
    return p.base().multiply(IVA);
}
```

**Devolver `null` es casi siempre una mala decisión.** Obliga a cada llamante a comprobarlo y el que se olvide recibe un `NullPointerException` lejos del origen. Alternativas por orden de preferencia:

| En lugar de devolver `null` | Devolvé |
|---|---|
| un objeto que puede no existir | `Optional<T>` |
| una colección vacía | `List.of()`, nunca `null` |
| un texto ausente | `""` si tiene sentido, o `Optional<String>` |
| un fallo | lanzá una excepción |

> **Sobre `Optional`:** es para tipos de **retorno**, no para campos ni parámetros. Usar `Optional` como campo lo hace serializable-hostil y añade una indirección sin ganancia. La regla la fijaron los propios diseñadores de la API en Java 8.

## 26. Varargs

Un método puede recibir un número variable de argumentos con `...`. Ni Jenkov ni W3Schools lo mencionan en sus páginas de métodos, y es de uso diario:

```java
public static int sumar(int... numeros) {
    int total = 0;
    for (int n : numeros) total += n;
    return total;
}

sumar();            // 0
sumar(1, 2, 3);     // 6
sumar(new int[]{1, 2, 3});   // también vale: por dentro es un array
```

**Reglas de obligado cumplimiento:**

1. Solo puede haber **un** parámetro varargs y tiene que ser **el último**.
2. Dentro del método, `numeros` es un array normal; puede tener longitud 0, pero nunca es `null` si lo llamás normalmente.
3. Varargs y array son la misma firma: no podés sobrecargar `f(int...)` y `f(int[])`.

**El detalle de rendimiento:** cada llamada a un método varargs **crea un array**. En un bucle caliente de millones de iteraciones eso es basura que el recolector tiene que limpiar. Por eso el JDK sobrecarga sus propios métodos: `List.of()` tiene versiones para 0, 1, 2... hasta 10 elementos, y solo la undécima usa varargs.

**Y la trampa de `null`:**

```java
static void f(String... args) { System.out.println(args.length); }

f(null);   // NullPointerException: args es null, no un array de un elemento
```

Al pasar `null` a secas, Java lo interpreta como el array entero, no como un elemento. Hay que escribir `f((String) null)`.

## 27. Métodos de instancia frente a métodos estáticos

Un **método de instancia** necesita un objeto y puede usar `this`. Un **método estático** pertenece a la clase, se invoca sin objeto y **no puede usar `this` ni acceder a campos de instancia**.

```java
public class Temperatura {
    private final double celsius;

    public Temperatura(double celsius) { this.celsius = celsius; }

    public double aFahrenheit() {                  // instancia: usa el estado
        return celsius * 9 / 5 + 32;
    }

    public static double convertir(double celsius) {   // estático: solo su entrada
        return celsius * 9 / 5 + 32;
    }
}
```

W3Schools explica `static` así: «`static` means that the method belongs to the Main class and not an object of the Main class». Es correcto, pero en sus ejemplos usa `static` en todo simplemente para poder llamarlo desde `main` sin crear objetos, sin explicar que **eso no es una decisión de diseño sino una comodidad didáctica**. Un principiante que copia ese estilo acaba escribiendo clases enteras de métodos estáticos, que es programación procedural con sintaxis de objetos.

**Cuándo un método debe ser estático:**

| Situación | ¿`static`? |
|---|---|
| No usa ningún campo de instancia | Sí, y el IDE te lo sugerirá |
| Es una función pura de sus argumentos (`Math.max`) | Sí |
| Es una factoría (`List.of`, `Optional.of`) | Sí |
| Lee o modifica el estado del objeto | **No** |
| Lo hiciste estático para "no tener que instanciar" | **No**: revisá el diseño |

> **El coste oculto de un método estático:** no se puede sobrescribir ni sustituir por un doble en los tests. Si `ServicioPago.cobrar(...)` es estático, no hay forma limpia de simularlo en un test unitario sin herramientas de reescritura de bytecode. Un método de instancia sobre una interfaz se sustituye en una línea. Por eso el exceso de `static` es una de las causas más comunes de código imposible de testear.

**Ocultamiento de métodos estáticos.** Igual que con los campos, un método estático redefinido en una subclase **no se sobrescribe: se oculta**, y se resuelve por el tipo estático. Es el mismo fenómeno de la sección 19 y la misma conclusión: no lo hagas.

## 28. throws forma parte de la declaración

Si un método puede lanzar una excepción comprobada, tiene que declararlo:

```java
public String concatenar(String a, String b) throws MiExcepcion {
    if (a == null) throw new MiExcepcion("a era null");
    return a + b;
}
```

Dos precisiones que Jenkov no hace:

1. **`throws` no forma parte de la firma.** No podés sobrecargar cambiando solo las excepciones, igual que con el tipo de retorno.
2. **Solo es obligatorio para excepciones comprobadas** (las que heredan de `Exception` sin pasar por `RuntimeException`). Para `IllegalArgumentException`, `NullPointerException` y demás no comprobadas, `throws` es opcional y sirve solo como documentación.

El diseño de excepciones tiene bloque propio (`03-Excepciones`). Aquí basta con saber que la cláusula existe y que forma parte del contrato que un método publica.

---

# Parte V — Sobrecarga

## 29. Qué es sobrecargar y qué no lo es

**Sobrecargar** (*overloading*) es declarar varios métodos con **el mismo nombre y distinta lista de parámetros** en la misma clase.

```java
static int    sumar(int a, int b)       { return a + b; }
static double sumar(double a, double b) { return a + b; }
static int    sumar(int a, int b, int c){ return a + b + c; }
```

W3Schools lo explica correctamente en lo esencial, pero su página tiene dos defectos que conviene señalar. El primero es cosmético y revelador: **sus tres bloques de código de ejemplo están etiquetados como `csharp`** en el HTML, en una página del tutorial de Java. El segundo es de fondo y sí importa: **no menciona en ningún momento que no se puede sobrecargar por tipo de retorno**, que es justo el error que va a cometer quien lea solo esa página. Lo demostramos en la sección 21.

**Sobrecargar no es sobrescribir.** Es la confusión de vocabulario más frecuente:

| | Sobrecarga (*overloading*) | Sobrescritura (*overriding*) |
|---|---|---|
| Dónde | misma clase (o heredado) | subclase |
| Qué cambia | la lista de parámetros | nada: misma firma |
| Cuándo se decide | **compilación** | **ejecución** |
| Se llama | polimorfismo estático | polimorfismo dinámico |
| Anotación | ninguna | `@Override` |

## 30. Las tres fases de la resolución de sobrecarga

Cuando hay varias sobrecargas candidatas, el compilador aplica **tres fases en orden**, y **se queda con la primera que encuentre un candidato aplicable**. Esto está en la JLS §15.12.2 y explica resultados que parecen mágicos.

| Fase | Qué se permite |
|---|---|
| 1 | conversiones de **ensanchado** de primitivos (`int` a `long`), sin boxing ni varargs |
| 2 | además, **autoboxing/unboxing** |
| 3 | además, **varargs** |

Demostración con tres sobrecargas que compiten por la misma llamada:

```java
static String h(long x)     { return "h(long)"; }
static String h(Integer x)  { return "h(Integer)"; }
static String h(int... x)   { return "h(int...)"; }

h(5);   // ¿cuál?
```

Salida real en JDK 25:

```
h(5) -> h(long)
```

**Gana `h(long)`**, aunque `h(Integer)` parezca "más cercano" a un `int`. Razón: en la fase 1, ensanchar `int` a `long` ya da un candidato válido, así que el compilador ni siquiera considera el boxing ni el varargs.

La misma regla explica la prioridad del varargs:

```java
static String g(int a, int b) { return "g(int,int)"; }
static String g(int... a)     { return "g(int...)"; }
```

Salida real:

```
g(1,2)   -> g(int,int)
g(1,2,3) -> g(int...)
g()      -> g(int...)
```

Con dos argumentos gana la versión fija; el varargs solo entra cuando no queda otra. **El varargs es siempre el último recurso.**

> **La consecuencia que muerde.** Añadir una sobrecarga a una clase existente puede **cambiar en silencio a qué método van llamadas que ya estaban escritas**, sin ningún error ni aviso. Si añadís `procesar(long)` a una clase que ya tenía `procesar(Integer)`, todas las llamadas con un `int` cambian de destino al recompilar. Por eso añadir sobrecargas a una API pública es un cambio más delicado de lo que parece.

## 31. Manda el tipo estático, no el objeto real

Corolario directo de lo anterior, y una de las preguntas de entrevista más discriminantes:

```java
static String f(Object o) { return "f(Object)"; }
static String f(String s) { return "f(String)"; }

Object o = "soy un String";
System.out.println(f(o));
System.out.println(f("literal"));
```

Salida real:

```
f(o) con Object o = "..." -> f(Object)
f("literal") -> f(String)
```

**El objeto es un `String` en ambos casos.** Pero en el primero la variable está declarada como `Object`, y la sobrecarga se decide **en compilación con el tipo declarado**, no en ejecución con el tipo real.

Contrastalo con la sobrescritura de la sección 19: allí `p.getEtiqueta()` devolvía `"Hijo"` porque los métodos sobrescritos sí miran el objeto real. **Sobrecarga: tipo estático. Sobrescritura: tipo dinámico.** Las dos palabras se parecen y hacen lo contrario.

**Dónde muerde esto en la vida real:** el caso más famoso es `List.remove`:

```java
List<Integer> lista = new ArrayList<>(List.of(10, 20, 30));
lista.remove(1);                      // remove(int índice): borra el de la POSICIÓN 1 -> quita el 20
lista.remove(Integer.valueOf(1));     // remove(Object): borra el VALOR 1 -> no hay, no borra nada
```

Dos métodos sobrecargados, `remove(int)` y `remove(Object)`, y el resultado depende de si escribís un literal o un objeto. Es un bug clásico en listas de enteros.

## 32. null y la llamada ambigua

`null` encaja en cualquier tipo de referencia, así que la sobrecarga con `null` tiene reglas propias.

**Caso 1: hay una sobrecarga más específica que las demás.** Gana la más específica.

```java
static String f(Object o) { return "f(Object)"; }
static String f(String s) { return "f(String)"; }

f(null);
```

Salida real: `f(null) -> f(String)`

`String` es subtipo de `Object`, o sea más específico, así que gana.

**Caso 2: las alternativas no tienen relación de herencia.** No compila.

```java
public class AmbiguousNull {
    static String f(String s)  { return "String"; }
    static String f(Integer i) { return "Integer"; }
    public static void main(String[] a) { System.out.println(f(null)); }
}
```

Error real:

```
error: reference to f is ambiguous
        System.out.println(f(null));
                           ^
  both method f(String) in AmbiguousNull and method f(Integer) in AmbiguousNull match
```

`String` e `Integer` son hermanos: ninguno es más específico que el otro. La solución es decirle al compilador cuál querés con un casting:

```java
f((String) null);    // ahora sí
```

## 33. Cuándo la sobrecarga hace daño

La sobrecarga es útil cuando **varias formas de expresar lo mismo** deben comportarse igual. Es dañina cuando se usa para meter comportamientos distintos bajo un nombre común.

**Buen uso** (el mismo concepto, distintas comodidades de entrada):

```java
public void log(String mensaje) { ... }
public void log(String mensaje, Throwable causa) { ... }
public void log(String plantilla, Object... args) { ... }
```

**Mal uso** (cosas distintas con el mismo nombre):

```java
public void procesar(Pedido p)   { /* lo guarda en base de datos */ }
public void procesar(Usuario u)  { /* le manda un email */ }
public void procesar(String s)   { /* lo parsea */ }
```

Tres operaciones sin nada en común compartiendo nombre. Quien lea `procesar(x)` no sabe qué pasa sin ir a mirar el tipo de `x`. **Nombres distintos habrían dicho la verdad:** `guardarPedido`, `notificarUsuario`, `parsear`.

**Reglas de higiene con sobrecargas:**

1. Las sobrecargas deben hacer **lo mismo** con distintas entradas.
2. Si podés, hacé que unas deleguen en otras, para que solo haya un sitio con lógica:
   ```java
   public void log(String m)               { log(m, null); }
   public void log(String m, Throwable c)  { /* única implementación real */ }
   ```
3. Evitá sobrecargar con el **mismo número de parámetros y tipos convertibles entre sí** (`int`/`long`/`Integer`/`Object`). Es la receta exacta para los problemas de las secciones 30 a 32.
4. Si la ambigüedad es inevitable, usá **nombres distintos**. El JDK lo hace: `Integer.valueOf(int)` y `Integer.parseInt(String)` podrían haber sido sobrecargas y son métodos con nombres distintos, a propósito.

---

# Parte VI — Atributos y métodos juntos: los accesores

## 34. El par getter y setter automático no es encapsulación

Esta es la crítica más importante del capítulo y la que más cuesta aceptar, porque contradice lo que enseña casi todo tutorial.

El patrón que aprende todo el mundo:

```java
public class Persona {
    private String nombre;
    private int edad;

    public String getNombre()          { return nombre; }
    public void setNombre(String n)    { this.nombre = n; }
    public int getEdad()               { return edad; }
    public void setEdad(int e)         { this.edad = e; }
}
```

Se enseña como "encapsulación". **No lo es.** Comparalo con la versión de campos públicos:

```java
public class Persona {
    public String nombre;
    public int edad;
}
```

¿Qué puede hacer un cliente en cada caso? **Exactamente lo mismo**: leer y escribir cualquier campo a voluntad. La primera versión tiene doce líneas más y la misma superficie de ataque. `p.setEdad(-5)` funciona igual de bien que `p.edad = -5`.

**Qué gana realmente el getter/setter trivial:**

| Ventaja | ¿Real? |
|---|---|
| Podés cambiar la representación interna sin romper clientes | **Sí**, y es la razón de peso |
| Los frameworks (Jackson, Hibernate, JSF) los necesitan | **Sí**, es una razón práctica |
| Podés añadir validación **después** | Sí, aunque para entonces habrá cien llamantes |
| "Es encapsulación" | **No** |

**Qué sería encapsular de verdad**: no exponer el dato, sino la operación.

```java
public class CuentaBancaria {
    private BigDecimal saldo;
    private final List<Movimiento> movimientos = new ArrayList<>();

    // NO hay setSaldo(): el saldo no se "pone", se modifica con operaciones válidas

    public void ingresar(BigDecimal importe) {
        if (importe.signum() <= 0) throw new IllegalArgumentException("importe no positivo");
        saldo = saldo.add(importe);
        movimientos.add(Movimiento.ingreso(importe));
    }

    public void retirar(BigDecimal importe) {
        if (importe.signum() <= 0) throw new IllegalArgumentException("importe no positivo");
        if (saldo.compareTo(importe) < 0) throw new SaldoInsuficienteException(saldo, importe);
        saldo = saldo.subtract(importe);
        movimientos.add(Movimiento.retirada(importe));
    }

    public BigDecimal getSaldo() { return saldo; }   // lectura sí; escritura no
}
```

Diferencias medibles con la versión de setters:

- **El saldo no puede quedar negativo**, nunca, desde ninguna parte del programa.
- **No puede cambiar sin que se registre un movimiento.** La invariante "saldo = suma de movimientos" se cumple por construcción.
- **La lógica está en un solo sitio.** Con `setSaldo`, la validación estaría repetida en cada llamante, o en ninguno.

> **Regla práctica.** Un getter suele estar justificado. **Un setter casi nunca lo está**: si un campo puede cambiar, preguntate qué operación del dominio lo cambia y expon esa operación. `pedido.setEstado(ENVIADO)` es peor que `pedido.enviar()`, porque el segundo puede comprobar que el pedido estaba pagado.

## 35. Copias defensivas

Un getter que devuelve una colección o un objeto mutable **entrega el control del estado interno**, aunque el campo sea `private final`.

```java
static class Pedido {
    private final List<String> lineas = new ArrayList<>();

    Pedido add(String s) { lineas.add(s); return this; }
    List<String> getLineasFiltrando() { return lineas; }               // FUGA
    List<String> getLineasCopia()     { return List.copyOf(lineas); }  // seguro
}
```

Y ahora el ataque, que no requiere ningún truco:

```java
Pedido ped = new Pedido().add("libro");
ped.getLineasFiltrando().add("INYECTADO DESDE FUERA");
```

Salida real:

```
tras mutar el getter: [libro, INYECTADO DESDE FUERA]
getLineasCopia() -> UnsupportedOperationException (copia inmutable)
```

**El campo era `private final` y aun así un extraño modificó el estado del pedido.** `private` protege la referencia, no el objeto (sección 10).

**Las tres fugas y sus arreglos:**

| Fuga | Arreglo |
|---|---|
| Getter que devuelve la colección interna | devolver `List.copyOf(...)` o `Collections.unmodifiableList(...)` |
| Constructor que guarda la colección que le pasan | copiar al entrar: `this.x = List.copyOf(x)` |
| Getter que devuelve un objeto mutable (`Date`, array) | devolver una copia; o mejor, usar tipos inmutables (`LocalDate`) |

Ejemplo completo con las dos direcciones cubiertas:

```java
public final class Reserva {
    private final List<String> huespedes;
    private final LocalDate entrada;          // inmutable por diseño: no hace falta copiar

    public Reserva(List<String> huespedes, LocalDate entrada) {
        this.huespedes = List.copyOf(huespedes);   // copia AL ENTRAR
        this.entrada = Objects.requireNonNull(entrada);
    }

    public List<String> getHuespedes() { return huespedes; }   // ya inmutable
    public LocalDate getEntrada()      { return entrada; }
}
```

> **Cuándo NO hace falta copiar.** Si el tipo es inmutable (`String`, `Integer`, `LocalDate`, `BigDecimal`, un `record` de campos inmutables), devolverlo directamente es correcto y no cuesta nada. La copia defensiva solo es necesaria con tipos mutables. Esa es una razón de peso para preferir tipos inmutables en los campos.

## 36. La convención JavaBeans y sus reglas exactas

Las reglas de nombres no son estética: **los frameworks las leen literalmente**. Equivocarse en una letra hace que un campo desaparezca de un JSON o de una tabla.

| Tipo del dato | Lectura | Escritura |
|---|---|---|
| cualquiera | `getNombre()` | `setNombre(String)` |
| `boolean` primitivo | `isActivo()` **o** `getActivo()` | `setActivo(boolean)` |
| `Boolean` (envoltorio) | `getBorrado()` — **`is` no vale** | `setBorrado(Boolean)` |

Verificado en la sección 4 con `java.beans.Introspector`: `isActivo()` sobre `boolean` produce la property `activo`, y `getBorrado()` sobre `Boolean` produce `borrado`. Si se hubiera escrito `isBorrado()` devolviendo `Boolean`, la property **no aparecería**.

**Reglas adicionales que causan bugs reales:**

1. **El nombre de la property es el del método sin el prefijo, con la primera letra en minúscula.** `getNombre()` da `nombre`.
2. **Excepción de las dos mayúsculas seguidas:** si tras el prefijo hay dos mayúsculas, el nombre se deja tal cual. `getURL()` da la property `URL`, no `uRL`. Por eso `getURL()` y `getUrl()` producen nombres distintos (`URL` y `url`) y un JSON distinto.
3. **Una property puede ser de solo lectura** (solo getter). Es lo normal en objetos inmutables.
4. **Puede no haber campo detrás**, como `getLongitudNombre()` en la sección 4.

> **Consejo de campo.** Cuando un campo "no aparece en el JSON" o "Hibernate no lo persiste", el 90% de las veces la causa es el nombre del accesor: `isX` sobre un `Boolean`, una mayúscula de más, o directamente falta el getter. Antes de tocar anotaciones, comprobá el nombre.

## 37. Records: accesores sin get

Desde Java 16, un `record` genera automáticamente los campos, el constructor, `equals`, `hashCode`, `toString` y los accesores.

```java
record PuntoR(int x, int y) {}
```

Comprobemos qué genera realmente. Salida real en JDK 25:

```
pr.x()=1  toString=PuntoR[x=1, y=2]
métodos públicos declarados: [equals, hashCode, toString, x, y]
```

**El accesor se llama `x()`, no `getX()`.** Es la decisión de diseño más comentada de los records: rompen la convención JavaBeans a propósito, porque un record no es un bean sino un *transporte transparente de datos*.

Consecuencia práctica: **una librería que busque `getX()` no encontrará nada en un record.** Jackson lo soporta desde la 2.12 con soporte específico; frameworks antiguos pueden no hacerlo. Es lo primero que hay que comprobar antes de convertir DTOs en records.

Comparación de lo que hay que escribir a mano:

```java
// Clase tradicional inmutable: ~40 líneas con equals, hashCode y toString
public final class Punto {
    private final int x, y;
    public Punto(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { ... }
    @Override public String toString() { ... }
}

// Record: 1 línea, con lo mismo
public record Punto(int x, int y) {}
```

Los records tienen su propio capítulo en este bloque (`16-Records`). Aquí importa solo el efecto sobre atributos y métodos: **los campos de un record son `private final` siempre y sus accesores no llevan `get`**.

## 38. Method chaining y APIs fluidas

Jenkov dedica una página al *method chaining*: hacer que un método devuelva `this` para poder encadenar llamadas.

```java
static class Nodo {
    final List<Nodo> hijos = new ArrayList<>();
    Nodo addChild(Nodo n) { hijos.add(n); return this; }
}

Nodo raiz = new Nodo().addChild(new Nodo()).addChild(new Nodo()).addChild(new Nodo());
```

Salida real: `hijos encadenados=3`.

Conviene señalar que **el ejemplo de Jenkov, tal como está publicado, no compila**: escribe `public List<Node< children = new ArrayList<Node>();` con un `<` donde va un `>`. Error real:

```
error: > or ',' expected
    public List<Node< children = new ArrayList<Node>();
                               ^
```

Es una errata, pero está en las dos versiones del ejemplo de esa página. La versión correcta es `List<Node> children`.

**Las limitaciones que Jenkov sí identifica bien** son reales y merecen recordarse: encadenando no podés "subir" al padre ni referirte al objeto sobre el que estás llamando en medio de la cadena.

**Cuándo el encadenamiento es correcto:**

| Uso | ¿Bien? |
|---|---|
| Builder (`Usuario.builder().nombre("Ana").build()`) | Sí, es su caso canónico |
| API de configuración (`HttpClient.newBuilder().connectTimeout(...)`) | Sí |
| Streams (`lista.stream().filter(...).map(...)`) | Sí, aunque ahí cada paso devuelve un objeto nuevo |
| Setters de una entidad mutable que devuelven `this` | **Cuidado** |

La última fila necesita explicación. Convertir los setters en encadenables:

```java
public Persona setNombre(String n) { this.nombre = n; return this; }
```

...rompe la convención JavaBeans, que exige que el setter devuelva `void`. Muchos frameworks dejan de reconocerlo como property. Si querés fluidez en un objeto mutable, usá nombres sin `set` (`nombre(...)`, `conNombre(...)`) o, mejor, un builder aparte.

> **La diferencia entre fluido e inmutable.** `addChild` devuelve `this` y muta el objeto. Los métodos de `String` (`trim().toLowerCase()`) devuelven **objetos nuevos** y no mutan nada. Los dos se encadenan igual y son cosas muy distintas: en el segundo caso podés compartir el original entre hilos sin miedo. Si diseñás una API fluida, decidí conscientemente cuál de las dos es y documentalo.

---

# Parte VII — Diseño

## 39. Tell, dont ask

El principio que resume todo este capítulo: **decile al objeto qué hacer, no le preguntes sus datos para decidir vos**.

Anti-ejemplo (*ask*):

```java
if (cuenta.getSaldo().compareTo(importe) >= 0) {
    cuenta.setSaldo(cuenta.getSaldo().subtract(importe));
}
```

El llamante extrae el dato, razona sobre él y devuelve el resultado. Problemas: la regla de negocio vive fuera del objeto, se repite en cada sitio que retire dinero, y nada impide que alguien se salte el `if`.

Correcto (*tell*):

```java
cuenta.retirar(importe);   // la cuenta sabe si puede y qué hacer si no
```

**Cómo detectar violaciones de este principio en tu código:** buscá secuencias de `getX()` seguidas de un cálculo y un `setX()`. Casi siempre son un método que le falta a la clase.

```java
// huele mal
BigDecimal total = BigDecimal.ZERO;
for (Linea l : pedido.getLineas()) total = total.add(l.getPrecio());

// mejor: el pedido sabe cuánto vale
BigDecimal total = pedido.total();
```

Esto conecta con el **modelo anémico** que ya trató el capítulo anterior (sección 45): una clase con solo campos, getters y setters, y toda la lógica en un "servicio" externo. Es programación procedural disfrazada.

> **Matiz honesto.** Los DTOs, los objetos de configuración y los mapeos de base de datos **sí** son legítimamente anémicos: su trabajo es transportar datos, no comportarse. El error es aplicar esa forma a las clases que modelan el dominio.

## 40. Métodos cortos y una sola responsabilidad

Un método debería hacer **una cosa** y su nombre debería decirla entera.

**Señales de que un método hace demasiado:**

| Señal | Qué indica |
|---|---|
| Su nombre lleva "y" (`validarYGuardar`) | son dos métodos |
| Necesita comentarios que separan bloques | esos bloques son métodos |
| Pasa de ~30 líneas | probablemente son varios |
| Tiene más de 3 niveles de anidamiento | falta extraer o usar salidas tempranas |
| No podés describirlo en una frase | no tiene una responsabilidad clara |

```java
// Antes: hace cuatro cosas
public void procesarPedido(Pedido p) {
    if (p == null || p.getLineas().isEmpty()) throw new IllegalArgumentException();
    BigDecimal total = BigDecimal.ZERO;
    for (Linea l : p.getLineas()) total = total.add(l.getPrecio());
    if (total.compareTo(LIMITE) > 0) total = total.multiply(DESCUENTO);
    repositorio.guardar(p);
    emailService.enviar(p.getCliente().getEmail(), "Pedido confirmado");
}

// Después: cada paso con nombre
public void procesarPedido(Pedido p) {
    validar(p);
    BigDecimal total = calcularTotalConDescuento(p);
    p.confirmar(total);
    repositorio.guardar(p);
    notificar(p);
}
```

El segundo se lee como una frase. Y cada parte es testeable por separado.

## 41. Cuántos campos son demasiados

No hay un número mágico, pero sí una señal: **si los campos se agrupan naturalmente, esos grupos son clases que faltan**.

```java
// 9 campos
public class Cliente {
    private String nombre, apellido;
    private String calle, ciudad, codigoPostal, pais;
    private String email, telefono;
    private LocalDate alta;
}
```

Los cuatro del medio son una dirección; los dos siguientes, un contacto:

```java
public class Cliente {
    private final NombreCompleto nombre;
    private final Direccion direccion;
    private final Contacto contacto;
    private final LocalDate alta;
}
```

Ganancias concretas: podés validar el código postal en un solo sitio, reutilizar `Direccion` en `Proveedor`, y `enviarA(Direccion)` deja de aceptar cuatro strings intercambiables por error.

**Otra señal, más fina:** si distintos métodos usan **subconjuntos disjuntos** de los campos, la clase son en realidad dos clases pegadas. Si `metodoA` toca los campos 1-3 y `metodoB` toca los 4-6 y nunca se cruzan, no hay cohesión.

## 42. Nombrar campos y métodos

Las convenciones de Java son férreas y no negociables en un equipo:

| Elemento | Convención | Ejemplo |
|---|---|---|
| Campo de instancia | `camelCase`, sustantivo | `fechaCreacion` |
| Campo `static final` | `MAYUSCULAS_CON_GUION_BAJO` | `MAX_REINTENTOS` |
| Método | `camelCase`, **verbo** | `calcularTotal()` |
| Getter | `getX()` / `isX()` | `getNombre()`, `isActivo()` |
| Método que devuelve `boolean` | `is`, `has`, `can`, `should` | `hasPermiso()` |

**Errores de nombrado que se ven todos los días:**

| Mal | Bien | Por qué |
|---|---|---|
| `private String str;` | `private String nombreCliente;` | el tipo ya dice que es un String |
| `private List<Pedido> list;` | `private List<Pedido> pedidos;` | plural, sin el tipo |
| `public void data()` | `public Datos obtenerDatos()` | un método es una acción |
| `private boolean flag;` | `private boolean estaPagado;` | ¿qué significa `true`? |
| `private int n;` | `private int numeroIntentos;` | ilegible en tres meses |
| `private boolean noActivo;` | `private boolean activo;` | los booleanos negados producen dobles negaciones |

La última merece énfasis. Con `noActivo`, comprobar que algo está activo se escribe `if (!noActivo)`, y a partir de ahí todo el código se vuelve un acertijo. **Nombrá siempre el booleano en positivo.**

---

# Parte VIII — Qué pasa por debajo

## 43. Dónde vive cada campo

Saber esto explica el rendimiento y el consumo de memoria:

| Elemento | Dónde vive | Cuántas copias |
|---|---|---|
| Campo de instancia | en el objeto, en el heap | una por objeto |
| Campo estático | en los metadatos de la clase | una por clase cargada |
| Variable local | en la pila del hilo | una por invocación |
| Parámetro | en la pila del hilo | una por invocación |
| El objeto apuntado por un campo | en el heap | compartido si varios apuntan |

Consecuencias prácticas:

1. **Cada campo de instancia cuesta memoria por objeto.** Un `long` de más en una clase de la que hay diez millones de instancias son ~80 MB.
2. **Las variables locales son gratis en comparación**: viven en la pila, se liberan solas al volver del método y nunca las visita el recolector.
3. **Un campo estático no lo recoge nunca el recolector** mientras la clase esté cargada. Un `static Map` que crece sin control es una fuga de memoria perfecta.
4. **Un campo `final` de tipo inmutable se puede compartir entre hilos** sin ningún tipo de sincronización. Es el argumento de rendimiento a favor de la inmutabilidad.

## 44. El coste real de un getter

Duda razonable de todo el que aprende esto: si un getter es una llamada a un método, ¿no es más lento que leer el campo directamente?

**En la práctica, no.** El compilador JIT de HotSpot hace *inlining* de métodos pequeños y muy llamados: sustituye la llamada por el acceso directo al campo. Un getter trivial, tras unos miles de invocaciones, acaba compilado exactamente al mismo código máquina que la lectura directa.

Por eso el argumento "uso campos públicos por rendimiento" **no se sostiene** en código de aplicación normal. La diferencia es cero después de que el JIT caliente el código.

> **Advertencia metodológica, aprendida a base de errores.** Medir esto con un bucle casero da números sin sentido: el JIT detecta que el resultado no se usa y elimina el trabajo entero, produciendo tiempos absurdos como 0,1 nanosegundos por llamada. Cualquier medición de este tipo necesita JMH, que se encarga de impedir esas optimizaciones. Sin JMH, lo que estás midiendo no es lo que creés. El tema tiene su sitio en el bloque de JVM.

## 45. static no significa más rápido

Otro mito. Un método estático evita pasar la referencia `this` y se resuelve con `invokestatic` en lugar de `invokevirtual`, lo que en teoría es marginalmente más barato. En la práctica:

- La diferencia es **irrelevante** frente a cualquier trabajo real del método.
- El JIT desvirtualiza las llamadas de instancia cuando solo hay una implementación cargada, con lo que la diferencia se evapora.
- **El coste de diseño es alto**: un método estático no se puede sustituir en un test ni sobrescribir.

**Elegí `static` por semántica** ("esto no depende de ningún objeto"), nunca por rendimiento.

---

# Parte IX — Cierre

## 46. Casos de uso reales

### 46.1 Entidad de dominio con invariantes

El caso más común en backend: una clase que representa algo del negocio y que **no puede existir en estado inválido**.

```java
public class Suscripcion {
    private final UUID id;
    private final UUID clienteId;
    private EstadoSuscripcion estado;
    private LocalDate finVigencia;

    public Suscripcion(UUID clienteId, LocalDate finVigencia) {
        this.id = UUID.randomUUID();
        this.clienteId = Objects.requireNonNull(clienteId, "clienteId");
        this.finVigencia = Objects.requireNonNull(finVigencia, "finVigencia");
        if (finVigencia.isBefore(LocalDate.now()))
            throw new IllegalArgumentException("vigencia en el pasado");
        this.estado = EstadoSuscripcion.ACTIVA;
    }

    public void cancelar(LocalDate cuando) {
        if (estado != EstadoSuscripcion.ACTIVA)
            throw new IllegalStateException("solo se cancela una suscripción activa: " + estado);
        this.estado = EstadoSuscripcion.CANCELADA;
        this.finVigencia = cuando;
    }

    public boolean estaVigente(LocalDate fecha) {
        return estado == EstadoSuscripcion.ACTIVA && !fecha.isAfter(finVigencia);
    }

    public EstadoSuscripcion getEstado()   { return estado; }
    public LocalDate getFinVigencia()      { return finVigencia; }
}
```

Decisiones y su porqué: `id` y `clienteId` son `final` porque nunca cambian; **no hay `setEstado`**, solo `cancelar()`, que comprueba la transición; `estaVigente` es un método de consulta que evita que el llamante replique la regla.

### 46.2 Objeto de valor inmutable

```java
public record Dinero(BigDecimal cantidad, Currency moneda) {
    public Dinero {                                    // constructor compacto: valida
        Objects.requireNonNull(cantidad, "cantidad");
        Objects.requireNonNull(moneda, "moneda");
        if (cantidad.scale() > moneda.getDefaultFractionDigits())
            throw new IllegalArgumentException("demasiados decimales para " + moneda);
    }

    public Dinero mas(Dinero otro) {
        if (!moneda.equals(otro.moneda))
            throw new IllegalArgumentException("monedas distintas");
        return new Dinero(cantidad.add(otro.cantidad), moneda);   // devuelve uno NUEVO
    }
}
```

`mas` no muta: devuelve un objeto nuevo. Esa es la diferencia entre una API fluida mutable y una inmutable (sección 38).

### 46.3 Clase de utilidad sin estado

```java
public final class Validadores {                 // final: nadie la extiende
    private Validadores() {                      // constructor privado: nadie la instancia
        throw new AssertionError("clase de utilidad");
    }

    public static boolean esEmailValido(String s) {
        return s != null && s.matches("^[^@\\s]+@[^@\\s]+\\.[^@\\s]+$");
    }
}
```

Aquí `static` **sí** es la decisión correcta: no hay estado que guardar y la clase es solo un espacio de nombres.

### 46.4 Constante compartida bien hecha

```java
public final class Reintentos {
    public static final int MAXIMO = 3;
    public static final Duration ESPERA_INICIAL = Duration.ofMillis(200);
    public static final List<Integer> CODIGOS_REINTENTABLES = List.of(429, 502, 503, 504);
    private Reintentos() {}
}
```

`List.of` y no `new ArrayList<>`: sin eso, cualquiera podría añadir un código y afectar a todo el proceso (sección 11).

## 47. Anti-patrones

**1. Setters para todo**

```java
public class Pedido {
    public void setEstado(EstadoPedido e) { this.estado = e; }
}
pedido.setEstado(EstadoPedido.ENVIADO);   // ¿estaba pagado? nadie lo comprobó
```
*Arreglo:* exponé la operación de dominio, `pedido.enviar()`, que valida la transición.

**2. Getter que devuelve la colección interna**

```java
public List<Linea> getLineas() { return lineas; }
```
*Arreglo:* `return List.copyOf(lineas);`. Demostrado en la sección 35: sin eso, cualquiera muta tu objeto.

**3. Campo público mutable**

```java
public String[] NOMBRES = {"a", "b"};
```
*Arreglo:* `private static final List<String> NOMBRES = List.of("a","b");`. Un array `public static final` es mutable en sus elementos: la constante es la referencia, no el contenido.

**4. Estado estático mutable**

```java
public class Sesion { public static Usuario actual; }
```
*Arreglo:* pasá el usuario como parámetro o inyectalo. Un `static` mutable es una variable global: rompe los tests, no es seguro entre hilos y no se recolecta nunca.

**5. Ocultar un campo del padre**

```java
class Padre { protected String nombre = "P"; }
class Hijo extends Padre { protected String nombre = "H"; }   // NO sobrescribe
```
*Arreglo:* poné `private` en el padre y un getter. Demostrado en la sección 19: el mismo objeto responde `"Padre"` por el campo y `"Hijo"` por el método.

**6. Sobrecargar con tipos convertibles entre sí**

```java
void procesar(int id) { }
void procesar(Integer id) { }
void procesar(long id) { }
```
*Arreglo:* dejá una sola, o poné nombres distintos. Con las tres, `procesar(5)` va a `procesar(int)` y `procesar(Integer.valueOf(5))` a otra, sin ningún aviso.

**7. Campo que debería ser variable local**

```java
private int temporal;   // usado solo dentro de un método
```
*Arreglo:* declaralo dentro del método. Como campo, no es seguro entre hilos y filtra datos entre llamadas (sección 14).

**8. Reasignar parámetros**

```java
public void f(String s) { s = s.trim(); }
```
*Arreglo:* `String limpio = s.trim();`. Perdés el valor original y confundís al que depura.

**9. Devolver `null` en vez de una colección vacía**

```java
public List<Pedido> buscar(String q) { if (nada) return null; }
```
*Arreglo:* `return List.of();`. Así el llamante puede iterar sin comprobar nada.

**10. Setters encadenables en un bean**

```java
public Persona setNombre(String n) { this.nombre = n; return this; }
```
*Arreglo:* o devolvé `void` y respetá JavaBeans, o hacé un builder aparte. A medias, los frameworks dejan de ver la property.

## 48. Checklist y tabla de decisión

**Antes de dar por bueno un campo:**

- [ ] ¿Es `private`? Si no, ¿por qué?
- [ ] ¿Podría ser `final`? (respuesta por defecto: sí)
- [ ] Si es `final` y de tipo mutable, ¿hay copia defensiva al entrar y al salir?
- [ ] Si es `static`, ¿es también `final` y de tipo inmutable?
- [ ] ¿Su nombre dice qué contiene, sin repetir el tipo?
- [ ] Si es `boolean`, ¿está en positivo?
- [ ] ¿Tiene que sobrevivir entre llamadas, o debería ser una variable local?
- [ ] ¿No oculta ningún campo de una superclase?

**Antes de dar por bueno un método:**

- [ ] ¿Su nombre es un verbo y describe todo lo que hace?
- [ ] ¿Hace una sola cosa?
- [ ] ¿Cuatro parámetros o menos?
- [ ] ¿Devuelve algo mutable que debería copiar?
- [ ] ¿Devuelve `null` donde podría devolver `Optional` o una colección vacía?
- [ ] Si es `static`, ¿es porque no depende del estado, o solo por comodidad?
- [ ] Si sobrescribe, ¿lleva `@Override`?
- [ ] Si lo estoy sobrecargando, ¿hacen las variantes realmente lo mismo?

**Tabla de decisión rápida:**

| Quiero... | Usá |
|---|---|
| un dato que no cambia nunca en el objeto | `private final` asignado en el constructor |
| un dato compartido por todas las instancias | `private static final` de tipo inmutable |
| exponer un dato para lectura | getter que devuelva un tipo inmutable o una copia |
| permitir que cambie el estado | un método de dominio (`enviar()`), no un setter |
| un dato temporal de un cálculo | variable local |
| varias formas de invocar lo mismo | sobrecargas que deleguen en una implementación única |
| un número variable de argumentos | varargs, sabiendo que crea un array por llamada |
| una clase que solo transporta datos | `record` |
| una función que no depende de estado | método `static` en una clase de utilidad |

## 49. Autoevaluación

**1. ¿Qué imprime este código y por qué?**

```java
class Padre { String x = "P"; String getX() { return "P"; } }
class Hijo extends Padre { String x = "H"; String getX() { return "H"; } }

Padre p = new Hijo();
System.out.println(p.x + " " + p.getX());
```

<details><summary>Respuesta</summary>

Imprime **`P H`**.

El campo `x` se resuelve **en compilación** usando el tipo de la referencia, que es `Padre`, así que devuelve `"P"`. El método `getX()` se resuelve **en ejecución** usando el tipo real del objeto, que es `Hijo`, así que devuelve `"H"`.

Los campos se **ocultan** (*hiding*), no se sobrescriben. Es la razón definitiva para declarar los campos `private`. Verificado en JDK 25 en la sección 19.
</details>

**2. ¿Compila esta clase? Si no, ¿cuál es el error exacto?**

```java
public class C {
    int  valor(int x) { return x; }
    char valor(int x) { return 'a'; }
}
```

<details><summary>Respuesta</summary>

**No compila.** Error: `method valor(int) is already defined in class C`.

La firma de un método son su **nombre y los tipos de sus parámetros**; el tipo de retorno no entra. Ambos métodos tienen la firma `valor(int)`, así que son el mismo.

El motivo de fondo: en Java se puede ignorar el valor devuelto, así que `valor(5);` sería irresoluble. Curiosidad de nivel senior: **la JVM sí admite** dos métodos que difieran solo en el retorno, y Java lo aprovecha para los métodos puente de los retornos covariantes.
</details>

**3. ¿Qué imprime?**

```java
static String h(long x)    { return "long"; }
static String h(Integer x) { return "Integer"; }
static String h(int... x)  { return "varargs"; }

System.out.println(h(5));
```

<details><summary>Respuesta</summary>

Imprime **`long`**.

La resolución de sobrecarga tiene tres fases y para en la primera que encuentra candidato: (1) ensanchado de primitivos, (2) autoboxing, (3) varargs. Ensanchar `int` a `long` ya funciona en la fase 1, así que ni se consideran `Integer` ni el varargs.

Regla: **ensanchado gana a boxing, y boxing gana a varargs.** Verificado en JDK 25.
</details>

**4. ¿Es cierto que un campo `final` debe llevar inicializador en su declaración?**

<details><summary>Respuesta</summary>

**No.** Es lo que afirma el tutorial de Jenkov y es falso.

Java permite los **blank finals**: declararlo sin valor y asignarlo en el constructor. La regla real es que debe estar asignado **exactamente una vez** al terminar cada constructor: ni cero veces ni dos.

Es el patrón base de toda clase inmutable:
```java
private final BigDecimal cantidad;
public Dinero(BigDecimal c) { this.cantidad = c; }
```
</details>

**5. El campo es `private final`. ¿Puede alguien de fuera modificar la lista?**

```java
private final List<String> lineas = new ArrayList<>();
public List<String> getLineas() { return lineas; }
```

<details><summary>Respuesta</summary>

**Sí, perfectamente:** `objeto.getLineas().add("lo que sea")`.

`final` congela la **referencia**, no el objeto apuntado, y `private` protege el campo, no el contenido. El getter entrega la lista real, así que quien la recibe tiene control total del estado interno.

Arreglo: `return List.copyOf(lineas);`, que lanza `UnsupportedOperationException` ante cualquier modificación. Y copiar también **al entrar**, en el constructor. Verificado en la sección 35.
</details>

**6. Esto compila dentro de una subclase en otro paquete. ¿Verdadero o falso?**

```java
package p2;
public class Sub extends p1.Base {
    void m() { p1.Base otro = new p1.Base(); System.out.println(otro.campoProtegido); }
}
```

<details><summary>Respuesta</summary>

**Falso.** Error: `campoProtegido has protected access in Base`.

La documentación oficial de Oracle dice que un miembro `protected` es accesible «by a subclass of its class in another package», y esa descripción está incompleta. La JLS §6.6.2 añade que el acceso por una expresión calificada `Q.Id` solo se permite **si el tipo de `Q` es la propia subclase o un subtipo suyo**.

`this.campoProtegido` sí compilaría. `otro.campoProtegido`, siendo `otro` de tipo `Base`, no.

Existe para que `protected` no acabe siendo equivalente a `public`: sin la restricción, cualquiera podría leer miembros protegidos de cualquier objeto declarando una subclase intermedia.
</details>

**7. ¿Qué diferencia hay entre un campo, un atributo y una property?**

<details><summary>Respuesta</summary>

- **Campo** (*field*): el único término del lenguaje. La JLS §8.3 los define; en el capítulo 8 aparece 224 veces, mientras que *attribute* y *property* aparecen **0 veces**.
- **Atributo**: palabra genérica de orientación a objetos, heredada de UML. No es incorrecta al hablar, pero no es terminología de Java.
- **Property**: término de la especificación **JavaBeans**. Es un **par de métodos** `getX()`/`setX()`, no un dato. Puede existir sin campo detrás: `getLongitudNombre()` crea la property `longitudNombre` aunque no haya ningún campo con ese nombre.

Esto último es lo que hace funcionar a Jackson, Hibernate y Spring.
</details>

**8. ¿Qué imprime y por qué?**

```java
List<Integer> lista = new ArrayList<>(List.of(10, 20, 30));
lista.remove(1);
System.out.println(lista);
```

<details><summary>Respuesta</summary>

Imprime **`[10, 30]`**.

`List` tiene dos métodos sobrecargados: `remove(int índice)` y `remove(Object o)`. El literal `1` es un `int`, así que se elige `remove(int)` y se borra el elemento de la **posición** 1, o sea el `20`.

Para borrar el **valor** 1 habría que escribir `lista.remove(Integer.valueOf(1))`. Es el ejemplo canónico de por qué sobrecargar con tipos convertibles entre sí es peligroso.
</details>

**9. Tras serializar y deserializar, ¿qué vale `token`?**

```java
class C implements Serializable {
    String usuario = "ana";
    transient String token = "SECRETO";
}
```

<details><summary>Respuesta</summary>

Vale **`null`**, no `"SECRETO"`.

`transient` excluye el campo de la serialización. Al deserializar, recibe el valor por defecto de su tipo, y el **inicializador de la declaración no se vuelve a ejecutar**, porque la deserialización no pasa por el constructor.

Es una fuente real de `NullPointerException`: código que asume que el campo está inicializado porque lo ve en la declaración. Los campos `static` tampoco se serializan, por la misma razón: pertenecen a la clase, no al objeto.
</details>

**10. ¿Por qué `props.getProperty("numero")` devuelve `null` si la clave existe?**

```java
Properties props = new Properties();
props.put("numero", 42);
System.out.println(props.getProperty("numero"));
```

<details><summary>Respuesta</summary>

Devuelve **`null`** porque `getProperty` comprueba que el valor sea `String` y, si no lo es, contesta `null` en lugar de fallar.

La causa raíz es un error de diseño: `Properties` **hereda** de `Hashtable<Object,Object>` en vez de contener un `Map`, así que expone `put(Object,Object)`, que acepta cualquier cosa y rompe su invariante "todo es String".

Efectos verificados: `get("numero")` sí devuelve `42`; `stringPropertyNames()` omite la entrada en silencio; y `store()` lanza `ClassCastException` — posiblemente horas más tarde. El Javadoc llama al objeto *"compromised"*.

La lección de diseño: **preferí composición a herencia** cuando heredar te obliga a exponer métodos que rompen tus reglas.
</details>

## 50. Fuentes

Las fuentes se listan con lo que aportan y **con sus errores señalados**, porque varios de sus ejemplos no compilan y algunas de sus afirmaciones son falsas. Todo lo marcado como error se comprobó en JDK 25 (Temurin 25.0.3+9).

### Fuentes primarias

- **[Java Language Specification, Java SE 25 — capítulo 8, *Classes*](https://docs.oracle.com/javase/specs/jls/se25/html/jls-8.html)**. La referencia normativa. §8.3 define las declaraciones de campo; §8.4 las de método; §8.4.2 define la firma. Es la fuente de la tabla terminológica de la sección 2: *attribute* 0 apariciones, *property* 0, *field* 224.
- **JLS §6.6.2, *Details on protected Access*.** La regla completa de `protected`, que ni Oracle Tutorials ni W3Schools reproducen correctamente.
- **JLS §15.12.2, *Compile-Time Step 2: Determine Method Signature*.** Las tres fases de la resolución de sobrecarga.
- **[Javadoc de `java.util.Properties`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Properties.html)**. Documenta explícitamente que `put` y `putAll` pueden dejar el objeto *"compromised"*.

### Fuentes que se pidieron para este capítulo

- **[Jenkov — Java Classes](https://jenkov.com/tutorials/java/classes.html)**. Buena visión general de los bloques que componen una clase. Sirvió de punto de entrada a `fields.html` y `methods.html`.
- **[Jenkov — Java Fields](https://jenkov.com/tutorials/java/fields.html)** (no estaba en la lista original; se llegó por la navegación lateral). La mejor de las tres para la sintaxis de declaración. **Dos problemas:** afirma que «A `final` field **must have an initial value assigned to it**», lo cual es falso — los *blank finals* existen y son la base de las clases inmutables (sección 9); y presenta `package` como si fuera uno de los cuatro modificadores de acceso, cuando el nivel de paquete se obtiene **no escribiendo** ninguno. Última actualización: 2015.
- **[Jenkov — Java Methods](https://jenkov.com/tutorials/java/methods.html)**. Explicación clara de parámetros, retorno y `throws`. **Error de compilación:** su primer ejemplo, repetido dos veces en la página, escribe `public MyClass{` sin la palabra `class`; `javac` responde `class, interface, annotation type, enum, record, method or field expected`. **Omisiones importantes:** no menciona varargs, ni la distinción instancia/estático, ni que el tipo de retorno no forma parte de la firma. Última actualización: 2015.
- **[Jenkov — Java Method Chaining](https://jenkov.com/tutorials/java/method-chaining.html)** (llegado por la navegación lateral). Buena explicación del patrón y honesta sobre sus límites. **Error de compilación:** escribe `public List<Node< children` con `<` en lugar de `>`, en los dos ejemplos; `javac` responde `> or ',' expected`.
- **[Jenkov — Java Properties](https://jenkov.com/tutorials/java-collections/properties.html)**. Documentación correcta y completa de `java.util.Properties`. **Aviso importante:** esta página **no trata de los atributos de una clase**, pese a llamarse "Properties". Es una clase de configuración y pertenece al bloque de colecciones. Se usó aquí como caso de estudio de mal diseño (sección 5).
- **[W3Schools — Java Class Attributes](https://www.w3schools.com/java/java_class_attributes.asp)**. Los ejemplos más simples de todos, útiles para empezar. **Imprecisión:** «variables declared inside a class are called attributes» — una variable declarada dentro de un método también está dentro de una clase y no es un campo (sección 3). **Terminología:** "attribute" no es un término del lenguaje. **Simplificación engañosa:** dice que `final` sirve «when you want a variable to always store the same value», lo que es falso para objetos mutables (sección 10).
- **[W3Schools — Java Class Methods](https://www.w3schools.com/java/java_class_methods.asp)**. Correcta en lo que cubre. **Problema pedagógico:** usa `static` en casi todos los ejemplos por comodidad, sin explicar que no es una decisión de diseño; quien copia ese estilo acaba escribiendo código procedural.
- **[W3Schools — Java Method Overloading](https://www.w3schools.com/java/java_methods_overloading.asp)**. **Omisión grave:** no menciona en ningún momento que no se puede sobrecargar por tipo de retorno, que es justo el error que va a cometer el lector. **Detalle revelador:** los tres bloques de código de esa página están etiquetados como `csharp` en el HTML, en el tutorial de Java.
- **[W3Schools — Java Modifiers](https://www.w3schools.com/java/java_modifiers.asp)**. Tabla útil de niveles de acceso. **Incompleta en `protected`:** «accessible in the same package and subclasses» omite la restricción de la JLS §6.6.2 que hace fallar la compilación del ejemplo de la sección 16.

### Discusiones de comunidad consultadas

- **[What is the difference between field, variable, attribute, and property in Java?](https://stackoverflow.com/questions/10115588/what-is-the-difference-between-field-variable-attribute-and-property-in-java)**. El hilo de referencia sobre terminología. Aporta la definición operativa que se usa aquí: *«Property is the getter and setter combination»* y *«Attribute is a vague term \[...\] Try to avoid using that term»*.
- **[the difference between field, variable, attribute, and property in Java](https://stackoverflow.com/questions/65483826/the-difference-between-field-variable-attribute-and-property-in-java)**. La respuesta que motivó comprobar la JLS: *«You'll find that the words attribute and property basically aren't in it»*. Confirmado midiendo el documento.
- **[OOP Terminology: class, attribute, property, field, data member](https://stackoverflow.com/questions/16751269/oop-terminology-class-attribute-property-field-data-member)**. Buen ejemplo de property calculada sin campo detrás (`getCircumference()`), que es el que inspiró la verificación con `Introspector`.
- **[Java: protected access across packages](https://stackoverflow.com/questions/3540640/java-protected-access-across-packages)**. La mejor explicación del *porqué* de la restricción de `protected`: sin ella, `protected` sería equivalente a `public` mediante una subclase intermedia.
- **[Sub-class not able to access the protected method of super-class when the reference is of type super-class](https://stackoverflow.com/questions/48624229/sub-class-not-able-to-access-the-protected-method-of-super-class-when-the-refere)**. Cita el pasaje exacto de la JLS y su motivación oficial.
- **[Java - why no return type based method overloading?](https://stackoverflow.com/questions/2744511/java-why-no-return-type-based-method-overloading)**. Además del motivo habitual, aporta el matiz de nivel senior: la JVM **sí** permite métodos que difieran solo en el retorno, y Java lo usa para los métodos puente de los retornos covariantes.
- **[Does a method's signature in Java include its return type?](https://stackoverflow.com/questions/16149285/does-a-methods-signature-in-java-include-its-return-type)**. Muestra los dos `clone()` en el bytecode, uno devolviendo `Test1` y otro `Object`.
- **[Why does java.util.Properties implement Map<Object,Object> and not Map<String,String>?](https://stackoverflow.com/questions/873510/why-does-java-util-properties-implement-mapobject-object-and-not-mapstring-st)**. El diagnóstico de diseño citado en la sección 5: *«Properties should never have inherited from Map at all. It should instead wrap around Map»*, y la explicación de por qué la compatibilidad hacia atrás impidió arreglarlo.

### Verificación

Todos los ejemplos, salidas y mensajes de error de este documento se ejecutaron en:

```
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9 (build 25.0.3+9-LTS)
```

Los mensajes de `javac` se reproducen literalmente. Los ocho casos que no compilan (`public MyClass{`, `List<Node<`, sobrecarga por retorno, asignación a `final`, referencia adelantada, `null` ambiguo, local sin inicializar y `protected` entre paquetes) se compilaron uno a uno para capturar el texto exacto del error.
