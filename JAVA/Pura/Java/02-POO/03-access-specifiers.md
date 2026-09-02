# Access Specifiers

> **Bloque:** `02-POO` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25

**Alcance de este documento.** Los dos capítulos anteriores construyeron la clase ([Classes and Objects](01-classes-and-objects.md)) y sus piezas ([Attributes and Methods](02-attributes-and-methods.md)). Ambos usaron `private`, `public` y `protected` dando por sabido qué significan. Este capítulo cubre entero **el sistema de control de acceso de Java**: los cuatro niveles, dónde se puede aplicar cada uno, las reglas exactas que la documentación oficial simplifica, qué hace la JVM por debajo con los `private`, cómo la reflexión los atraviesa y cómo el sistema de módulos añadió desde Java 9 un nivel más que no es ninguno de los cuatro.

**Por qué este tema decide si tu código se puede mantener.** La visibilidad es la única herramienta que tiene Java para separar *lo que prometés* de *lo que podés cambiar mañana*. Todo lo que es `public` es un contrato que alguien va a usar y que ya no podés tocar. La mayoría de las bases de código que se vuelven imposibles de refactorizar no llegaron ahí por mala arquitectura, sino por haber escrito `public` sin pensarlo en sitios donde no hacía falta.

Y hay reglas concretas que casi nadie conoce y que hacen fallar la compilación: **`protected` no significa lo que dice la documentación de Oracle**, un constructor `protected` no se puede invocar con `new` desde una subclase de otro paquete, una clase anidada `protected` recibe un constructor implícito que también es `protected`, y desde Java 9 **sí se pueden escribir métodos `private` en una interfaz**, contra lo que afirma una de las fuentes de este capítulo. Todo está demostrado aquí con el mensaje literal de `javac` en JDK 25.

**Lo que NO entra aquí**, porque tiene documento propio: `static` y `final` (`04` y `05`), la encapsulación como principio de diseño (`08`), la herencia (`09`), las clases anidadas en profundidad (`15`) y los paquetes y el build (bloque `11`). Aquí aparecen solo en lo que afecta a la visibilidad.

---

## Índice

**Parte I — Qué es realmente el control de acceso**

1. [El problema que resuelve la visibilidad](#1-el-problema-que-resuelve-la-visibilidad)
2. [Especificador o modificador: cómo se llama de verdad](#2-especificador-o-modificador-cómo-se-llama-de-verdad)
3. [Los cuatro niveles de un vistazo](#3-los-cuatro-niveles-de-un-vistazo)
4. [Dónde se puede aplicar cada modificador](#4-dónde-se-puede-aplicar-cada-modificador)
5. [El control de acceso no es seguridad](#5-el-control-de-acceso-no-es-seguridad)

**Parte II — Los cuatro niveles, uno a uno**

6. [private](#6-private)
7. [Acceso de paquete, el nivel sin palabra clave](#7-acceso-de-paquete-el-nivel-sin-palabra-clave)
8. [protected](#8-protected)
9. [public](#9-public)
10. [La tabla completa y cómo leerla sin equivocarse](#10-la-tabla-completa-y-cómo-leerla-sin-equivocarse)

**Parte III — La letra pequeña de protected**

11. [Lo que dicen las fuentes y por qué está incompleto](#11-lo-que-dicen-las-fuentes-y-por-qué-está-incompleto)
12. [La regla real de la JLS](#12-la-regla-real-de-la-jls)
13. [Constructores protected: new frente a super](#13-constructores-protected-new-frente-a-super)
14. [Clases anidadas protected y su constructor implícito](#14-clases-anidadas-protected-y-su-constructor-implícito)
15. [Por qué existe esta restricción](#15-por-qué-existe-esta-restricción)
16. [Cuándo usar protected](#16-cuándo-usar-protected)

**Parte IV — No solo miembros: clases, interfaces, enums**

17. [Clases de nivel superior: solo dos niveles](#17-clases-de-nivel-superior-solo-dos-niveles)
18. [La clase manda sobre sus miembros](#18-la-clase-manda-sobre-sus-miembros)
19. [Clases anidadas: los cuatro niveles](#19-clases-anidadas-los-cuatro-niveles)
20. [Varias clases en un mismo fichero](#20-varias-clases-en-un-mismo-fichero)
21. [Interfaces: público por defecto](#21-interfaces-público-por-defecto)
22. [Métodos privados en interfaces desde Java 9](#22-métodos-privados-en-interfaces-desde-java-9)
23. [Enums y sus constructores](#23-enums-y-sus-constructores)
24. [Records y su visibilidad](#24-records-y-su-visibilidad)
25. [Constructores privados y clases de utilidad](#25-constructores-privados-y-clases-de-utilidad)

**Parte V — Visibilidad y herencia**

26. [No se puede reducir la visibilidad al sobrescribir](#26-no-se-puede-reducir-la-visibilidad-al-sobrescribir)
27. [Sí se puede ampliarla](#27-sí-se-puede-ampliarla)
28. [Los métodos privados no se sobrescriben](#28-los-métodos-privados-no-se-sobrescriben)
29. [El caso del acceso de paquete al heredar](#29-el-caso-del-acceso-de-paquete-al-heredar)

**Parte VI — Qué pasa por debajo**

30. [Los modificadores en el fichero class](#30-los-modificadores-en-el-fichero-class)
31. [Nests: cómo accede una clase interna a lo privado](#31-nests-cómo-accede-una-clase-interna-a-lo-privado)
32. [La reflexión atraviesa private](#32-la-reflexión-atraviesa-private)
33. [El sistema de módulos: el nivel que no es ninguno de los cuatro](#33-el-sistema-de-módulos-el-nivel-que-no-es-ninguno-de-los-cuatro)
34. [Encapsulación fuerte desde Java 17](#34-encapsulación-fuerte-desde-java-17)
35. [add-opens y add-exports](#35-add-opens-y-add-exports)

**Parte VII — El paquete como unidad de diseño**

36. [El paquete no es una frontera infranqueable](#36-el-paquete-no-es-una-frontera-infranqueable)
37. [Package-private, el nivel más infrautilizado](#37-package-private-el-nivel-más-infrautilizado)
38. [Package by feature frente a package by layer](#38-package-by-feature-frente-a-package-by-layer)
39. [Split packages](#39-split-packages)
40. [Visibilidad y tests](#40-visibilidad-y-tests)

**Parte VIII — Diseño**

41. [Minimizar la accesibilidad](#41-minimizar-la-accesibilidad)
42. [Qué significa publicar algo](#42-qué-significa-publicar-algo)
43. [El orden canónico de los modificadores](#43-el-orden-canónico-de-los-modificadores)
44. [Árbol de decisión](#44-árbol-de-decisión)

**Parte IX — Cierre**

45. [Casos de uso reales](#45-casos-de-uso-reales)
46. [Anti-patrones](#46-anti-patrones)
47. [Checklist y tabla de decisión](#47-checklist-y-tabla-de-decisión)
48. [Autoevaluación](#48-autoevaluación)
49. [Fuentes](#49-fuentes)

---

# Parte I — Qué es realmente el control de acceso

## 1. El problema que resuelve la visibilidad

Imaginá que escribís esta clase y la publicás para que la use el resto del equipo:

```java
public class Cache {
    public Map<String, Object> datos = new HashMap<>();
    public int aciertos = 0;
    public int fallos = 0;
}
```

Funciona. Al día siguiente, quince clases distintas hacen `cache.datos.put(...)`, tres leen `cache.aciertos`, y una hace `cache.datos = null` por error.

Ahora descubrís que `HashMap` no sirve porque necesitás expiración por tiempo, y que `int` se desborda tras dos mil millones de aciertos. Querés cambiar la implementación.

**No podés.** Esas quince clases dependen de que `datos` sea exactamente un `Map`, y de que exista un campo llamado así. Cualquier cambio las rompe. Tu decisión de tres segundos de escribir `public` se convirtió en un contrato permanente.

La versión con visibilidad pensada:

```java
public class Cache {
    private final Map<String, Entrada> datos = new ConcurrentHashMap<>();
    private final LongAdder aciertos = new LongAdder();

    public Optional<Object> obtener(String clave) { ... }
    public void guardar(String clave, Object valor, Duration ttl) { ... }
    public Estadisticas estadisticas() { ... }
}
```

Ahora podés cambiar `ConcurrentHashMap` por Caffeine, `LongAdder` por lo que sea, o reescribir la clase entera. Mientras los tres métodos públicos sigan comportándose igual, **nadie se entera**.

Esa es toda la idea: **el control de acceso separa lo que prometés de lo que te reservás el derecho a cambiar**. No es burocracia ni ceremonia; es lo que te permite seguir trabajando dentro de seis meses.

## 2. Especificador o modificador: cómo se llama de verdad

El título de este capítulo dice *Access Specifiers* y conviene aclarar el vocabulario, porque es otro caso donde la palabra popular no es la del lenguaje.

Jenkov lo aclara con precisión en su tutorial:

> «Access modifiers are also sometimes referred to in daily speech as *Java access specifiers*, but the correct name is Java access modifiers.»

Tiene razón. La *Java Language Specification* usa **access modifier**; en la gramática, las producciones se llaman `ClassModifier`, `FieldModifier`, `MethodModifier`. La palabra *specifier* viene de C++, donde el término oficial sí es *access specifier* (`public:`, `private:`, `protected:`).

En este documento uso **modificador de acceso**, con "especificador" como sinónimo coloquial. No es un error decirlo, pero si escribís documentación técnica o respondés una entrevista, la palabra correcta en Java es *modifier*.

**Un detalle más de vocabulario**, importante porque los libros lo distinguen:

- **Modificadores de acceso**: `private`, `protected`, `public`. Son tres palabras, no cuatro.
- **Modificadores de no acceso**: `static`, `final`, `abstract`, `synchronized`, `native`, `transient`, `volatile`, `strictfp`, `default`, `sealed`.

El cuarto nivel de acceso, el de paquete, **no tiene palabra clave**: se obtiene no escribiendo ninguna de las tres.

## 3. Los cuatro niveles de un vistazo

Java tiene **cuatro niveles de acceso**, ordenados de más restrictivo a más abierto:

| Nivel | Se escribe | Se conoce también como |
|---|---|---|
| 1 | `private` | privado |
| 2 | *(nada)* | acceso de paquete, *package-private*, *default* |
| 3 | `protected` | protegido |
| 4 | `public` | público |

La progresión tiene un detalle que descoloca a casi todo el mundo:

```
private  <  acceso de paquete  <  protected  <  public
```

`protected` es **más abierto** que el acceso de paquete, no menos. Un miembro `protected` es visible para todo el paquete **más** las subclases externas. Mucha gente asume lo contrario porque "protegido" suena más restrictivo que "sin nada escrito".

Baeldung lo formula bien:

> «Between public and private access levels, there's the protected access modifier.»

aunque su frase se salta que el acceso de paquete también está en medio.

## 4. Dónde se puede aplicar cada modificador

Aquí hay una tabla que casi todas las fuentes simplifican. Jenkov publica esta:

| | private | default | protected | public |
|---|---|---|---|---|
| Class | No | Yes | No | Yes |
| Nested Class | Yes | Yes | Yes | Yes |
| Constructor | Yes | Yes | Yes | Yes |
| Method | Yes | Yes | Yes | Yes |
| Field | Yes | Yes | Yes | Yes |

Es **correcta**, y merece un reconocimiento por distinguir "Class" de "Nested Class", que es justo donde se equivocan otras fuentes. Baeldung también lo advierte: «a top-level class can only use public or default access modifiers».

Verifiquémoslo. Este fichero intenta lo prohibido:

```java
private class A {}
protected class B {}
```

Errores reales de `javac` en JDK 25:

```
TopLevel.java:1: error: modifier private not allowed here
private class A {}
        ^
TopLevel.java:2: error: modifier protected not allowed here
protected class B {}
```

**Por qué está prohibido**, y la explicación de Jenkov es la correcta: una clase de nivel superior `private` no podría usarla nadie, ni siquiera vos, porque no hay ningún "dentro" desde donde acceder a ella. Y `protected` no tiene sentido a ese nivel porque su mecanismo se apoya en la relación de herencia entre un miembro y la clase que lo declara.

La tabla completa, ampliada con lo que ninguna de las dos fuentes cubre:

| Se puede aplicar a | `private` | *(paquete)* | `protected` | `public` |
|---|---|---|---|---|
| Clase de nivel superior | No | **Sí** | No | **Sí** |
| Interfaz de nivel superior | No | **Sí** | No | **Sí** |
| Clase o interfaz anidada | **Sí** | **Sí** | **Sí** | **Sí** |
| Campo de clase | **Sí** | **Sí** | **Sí** | **Sí** |
| Método de clase | **Sí** | **Sí** | **Sí** | **Sí** |
| Constructor de clase | **Sí** | **Sí** | **Sí** | **Sí** |
| Campo de interfaz | No | No | No | implícito |
| Método abstracto de interfaz | No | No | No | implícito |
| Método `default` o `static` de interfaz | **Sí** (desde Java 9) | No | No | **Sí** |
| Constructor de `enum` | **Sí** | **Sí** | No | No |

Las cuatro últimas filas son las que producen errores de compilación inesperados, y las cubrimos en la Parte IV.

## 5. El control de acceso no es seguridad

Antes de entrar en detalle, hay que fijar una idea que evita malentendidos peligrosos: **los modificadores de acceso son una herramienta de diseño, no una medida de seguridad**.

Tres hechos que lo demuestran, todos verificados más adelante en este documento:

1. **La reflexión los atraviesa.** `field.setAccessible(true)` lee y escribe un campo `private` de tu propio código sin ningún problema (sección 32).
2. **El acceso de paquete no está sellado por defecto.** Cualquiera puede escribir una clase declarando `package com.tuempresa.interno;` y acceder a todo lo que sea de paquete (sección 36).
3. **Quien decide es el compilador.** Podés compilar contra una versión de una clase y ejecutar contra otra donde el miembro cambió de nivel; lo que salta entonces es un `IllegalAccessError` en ejecución, no un error de compilación.

Lo que sí protegen los modificadores es **contra el error honesto y contra el acoplamiento accidental**. Nadie va a usar tu campo interno sin querer si es `private`. Alguien decidido siempre podrá; la diferencia es que tendrá que hacerlo a propósito, escribiendo código que grita "estoy haciendo algo raro".

> **Consecuencia práctica.** Nunca pongas un secreto en un campo `private` creyendo que eso lo oculta. Un `private String claveApi = "..."` queda en texto claro dentro del `.class`, lo lee cualquiera con `javap -c` en dos segundos, y lo alcanza cualquier código del mismo proceso con reflexión.

---

# Parte II — Los cuatro niveles, uno a uno

## 6. private

**Lo ve solo la clase donde está declarado.** Ni las subclases, ni las clases del mismo paquete, ni nadie más.

```java
public class Reloj {
    private long tiempo = 0;

    public long getTiempo() { return this.tiempo; }
    public void setTiempo(long t) { this.tiempo = t; }
}
```

Jenkov describe bien el alcance: «only code inside the same class can access the variable, or call the method. Code inside subclasses cannot access the variable or method».

**El matiz que Jenkov no menciona y que sí importa:** "la misma clase" no significa "el mismo fichero `.class`". Significa **la misma clase de nivel superior, incluidas todas sus clases anidadas**. Por eso esto compila:

```java
public class Nido {
    private String secreto = "valor privado";
    private void metodoPrivado() { System.out.println("privado del externo"); }

    class Interna {
        void tocar() {
            System.out.println("la interna lee: " + secreto);   // accede a lo privado
            metodoPrivado();                                    // y lo invoca
        }
    }
}
```

Salida real en JDK 25:

```
la interna lee el campo privado del externo: valor privado
el externo ejecuta su metodo privado
```

Una clase anidada y su clase contenedora **se ven mutuamente todo lo privado**, en ambas direcciones. Cómo consigue la JVM permitirlo, siendo que para ella son dos ficheros `.class` distintos, es el tema de la sección 31, y la respuesta cambió en Java 11.

**Cuándo usar `private`:** por defecto, siempre. Es la respuesta correcta para prácticamente todos los campos y para todo método que sea un detalle de implementación.

## 7. Acceso de paquete, el nivel sin palabra clave

**Lo ve la propia clase y cualquier clase del mismo paquete.** Se obtiene **no escribiendo nada**:

```java
public class Reloj {
    long tiempo = 0;              // acceso de paquete
    void ajustar() { }            // acceso de paquete
}
```

Es el nivel con más nombres y peor prensa: *default*, *package-private*, *package access*, *friendly*. Jenkov lo llama "default (package)"; Baeldung dice «The default access modifier is also called package-private».

**Cuidado con la palabra `package`.** Jenkov enumera los niveles como «private, default (package), protected, public» y en su tutorial de campos llega a escribir «The `package` access modifier». No existe tal modificador: **no hay ninguna palabra clave que escribir**. El propio Jenkov lo aclara después («You don't actually write the package modifier»), pero la enumeración induce a error. Si escribís `package int x;` no compila.

**El dato histórico que explica por qué es el nivel por defecto** viene de James Gosling, el creador de Java, en una entrevista de 2000:

> «So public would have been a really bad thing to make the default. Private would probably have been a bad thing to make a default, if only because people actually don't write private methods that often. \[...\] And in looking at a bunch of code that I had, I decided that the most common thing that was reasonably safe was in the package.»

Y añade un arrepentimiento explícito sobre los campos:

> «it would've made a lot of sense for the default protection for an instance variable to be private.»

Es decir: **el propio diseñador de Java considera hoy que el nivel por defecto para los campos debería haber sido `private`**. Que sea de paquete es un accidente histórico, no una recomendación.

Gosling también explica por qué eligió el paquete en lugar del `friend` de C++:

> «I liked it rather than the friends notion, because with friends you kind of have to enumerate who all of your friends are, and so if you add a new class to a package, then you generally end up having to go to all of the classes in that package and update their friends.»

Este nivel está profundamente infrautilizado y le dedicamos la sección 37 entera.

## 8. protected

**Lo ve la propia clase, todo su paquete y las subclases, aunque estén en otro paquete.**

```java
public class Reloj {
    protected long tiempo = 0;
}

public class RelojInteligente extends Reloj {   // aunque esté en otro paquete
    public long getSegundos() { return this.tiempo / 1000; }
}
```

Ese es el ejemplo de Jenkov, y la idea funciona. Dos apuntes antes de seguir.

**Primero, el ejemplo tal como está publicado no compila.** Jenkov escribe:

```java
public class SmartClock() extends Clock{
```

con paréntesis después del nombre de la clase. Error real:

```
error: '{' expected
class SmartClock() extends Clock2{
                ^
```

Es una errata, pero conviene señalarla porque en la misma página hay otra: en el ejemplo del acceso de paquete escribe `public long readClock{` **sin los paréntesis del método**, y `javac` responde `';' expected`.

**Segundo, y mucho más importante:** la descripción de `protected` que dan Jenkov, Baeldung y la propia documentación de Oracle **está incompleta de una forma que hace fallar la compilación**. Es el tema de toda la Parte III.

## 9. public

**Lo ve todo el mundo.** Jenkov: «all code can access the class, field, constructor or method, regardless of where the accessing code is located».

```java
public class Reloj {
    public long tiempo = 0;
}
```

Con una salvedad grande desde Java 9: **`public` ya no significa "accesible desde cualquier parte"** si tu código vive en un módulo. Una clase `public` en un paquete que el módulo no exporta es inaccesible desde fuera. Lo vemos en la sección 33.

## 10. La tabla completa y cómo leerla sin equivocarse

Esta es la tabla de Baeldung, la versión canónica que circula por todas partes:

| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `public` | Y | Y | Y | Y |
| `protected` | Y | Y | Y | N |
| *(default)* | Y | Y | N | N |
| `private` | Y | N | N | N |

Es correcta **si sabés leer la columna "Subclass"**, y ahí está la trampa. Esa `Y` en la fila de `protected` significa "el código de una subclase puede acceder", pero **no** significa "de cualquier manera". Como veremos, hay formas de acceder desde una subclase que no compilan.

Además, la tabla tiene dos huecos que ninguna de las dos fuentes menciona:

1. **Falta la columna del módulo.** Desde Java 9 hay una quinta pregunta: ¿está exportado el paquete? Si no lo está, la columna "World" es `N` aunque sea `public`.
2. **La fila `private` dice `N` en "Subclass", y es cierto, pero es incompleto**: un miembro `private` sí es accesible desde una clase anidada, que puede estar en el mismo fichero pero es otra clase.

Versión corregida, que es la que conviene memorizar:

| Nivel | Misma clase | Clases anidadas | Mismo paquete | Subclase en otro paquete | Otro paquete | Otro módulo |
|---|---|---|---|---|---|---|
| `private` | Sí | **Sí** | No | No | No | No |
| *(paquete)* | Sí | Sí | Sí | No | No | No |
| `protected` | Sí | Sí | Sí | **Sí, con restricciones** | No | No |
| `public` | Sí | Sí | Sí | Sí | Sí | **Solo si se exporta** |

---

# Parte III — La letra pequeña de protected

## 11. Lo que dicen las fuentes y por qué está incompleto

Las tres descripciones más leídas del mundo dicen lo mismo:

**Oracle, The Java Tutorials:**
> «The `protected` modifier specifies that the member can only be accessed within its own package (as with package-private) and, in addition, by a subclass of its class in another package.»

**Jenkov:**
> «subclasses can access `protected` methods and member variables (fields) of the superclass. This is true even if the subclass is not located in the same package as the superclass.»

**Baeldung:**
> «we can access the member from the same package \[...\] as well as from all subclasses of its class, even if they lie in other packages»

Y Baeldung incluso muestra un caso que **contradice su propia descripción** sin darse cuenta. En su artículo dedicado a `protected`, presenta una clase anidada `protected`, la instancia desde una subclase en otro paquete, y reporta:

```
The constructor FirstClass.InnerClass() is not visible
```

Su comentario es: «We were expecting to instantiate our InnerClass with ease. However, we are getting a compilation error here too». Es decir: **la propia fuente documenta que su regla no se cumple, y no explica por qué**.

La explicación está en la especificación, y es lo que vemos ahora.

## 12. La regla real de la JLS

La *Java Language Specification*, §6.6.2, dice:

> «A protected member or constructor of an object may be accessed from outside the package in which it is declared **only by code that is responsible for the implementation of that object**.
>
> Let C be the class in which a protected member is declared. Access is permitted only within the body of a subclass S of C. In addition, if Id denotes an instance field or instance method, then: if the access is by a qualified name Q.Id, where Q is an ExpressionName, then the access is permitted **if and only if the type of the expression Q is S or a subclass of S**.»

En castellano llano: **dentro de tu subclase podés tocar el miembro protegido de objetos que sean de tu propio tipo o de un subtipo tuyo, no de cualquier objeto del padre.**

Demostración con dos paquetes:

```java
// p1/Base.java
package p1;
public class Base {
    protected int campo = 1;
    protected Base() {}
    protected void metodo() {}
    protected static class Anidada {}
}
```

```java
// p2/SubErr.java
package p2;
import p1.Base;
public class SubErr extends Base {
    void pruebas() {
        Base otro = new Base();
        System.out.println(otro.campo);
        Base.Anidada a = new Base.Anidada();
    }
}
```

Errores reales de `javac`, los tres a la vez:

```
p2/SubErr.java:5: error: Base() has protected access in Base
        Base otro = new Base();
                    ^
p2/SubErr.java:6: error: campo has protected access in Base
        System.out.println(otro.campo);
                               ^
p2/SubErr.java:7: error: Anidada() has protected access in Anidada
        Base.Anidada a = new Base.Anidada();
                         ^
```

`SubErr` **es** una subclase de `Base` y está en su cuerpo, exactamente el caso que las tres fuentes describen como permitido. Y las tres líneas fallan.

Ahora la versión que sí compila:

```java
// p2/SubOk.java
package p2;
import p1.Base;
public class SubOk extends Base {
    public SubOk() { super(); }          // (1) super() a un constructor protected: PERMITIDO
    void ok() {
        System.out.println(this.campo);  // (2) el campo, a través de this: PERMITIDO
        this.metodo();                   // (3) el método, a través de this: PERMITIDO
        Base anonima = new Base() {};    // (4) clase anónima: PERMITIDO
    }
}
```

Compila sin errores en JDK 25.

La diferencia entre las dos versiones es toda la regla: **`this.campo` sí, `otroObjeto.campo` no.**

## 13. Constructores protected: new frente a super

El caso de los constructores tiene su propia sección en la especificación, la §6.6.2.2, y es todavía más restrictiva:

> «If the access is by a superclass constructor invocation `super(...)` \[...\] then the access is permitted.
> If the access is by an anonymous class instance creation expression `new C(...){...}` \[...\] then the access is permitted.
> If the access is by a simple class instance creation expression `new C(...)` \[...\] **then the access is not permitted**. A protected constructor can be accessed by a class instance creation expression \[...\] only from within the package in which it is defined.»

Resumido en una tabla, verificada compilando cada caso:

| Desde una subclase en otro paquete | ¿Compila? |
|---|---|
| `super(...)` en su constructor | **Sí** |
| `new Base() {}` (clase anónima) | **Sí** |
| `new Base()` (creación normal) | **No** |
| `Base::new` (referencia a constructor) | **No** |

**Por qué la distinción tiene sentido.** `super(...)` y la clase anónima construyen un objeto que *es* de tu tipo (o de un subtipo). `new Base()` construye un `Base` a secas, un objeto del que tu subclase no es responsable. La regla es siempre la misma: podés tocar lo protegido de los objetos que vos implementás.

**Consecuencia de diseño muy útil:** un constructor `protected` es la forma canónica de decir *"esta clase solo se instancia heredando de ella o desde su propio paquete"*. Es lo que hacen las clases abstractas de las librerías serias.

## 14. Clases anidadas protected y su constructor implícito

Este es el caso que Baeldung documenta sin explicar, y tiene una causa concreta y sorprendente.

Cuando declarás una clase sin ningún constructor, el compilador le genera uno por defecto. La pregunta es: **¿con qué visibilidad?** La JLS §8.8.9 responde:

> «if the class is declared `protected`, then the default constructor is implicitly given the access modifier `protected`»

O sea: el constructor por defecto **hereda la visibilidad de la clase**. Una clase anidada `protected` recibe un constructor `protected`.

```java
public class Base {
    protected static class Anidada { }   // constructor implícito: protected Anidada() {}
}
```

Y ahí se dispara la regla de la sección anterior: **`new Base.Anidada()` es una creación normal, no un `super(...)` ni una clase anónima**, así que desde otro paquete no compila, aunque estés dentro de una subclase de `Base`:

```
error: Anidada() has protected access in Anidada
```

Fijate en el mensaje: dice *"has protected access **in Anidada**"*, no *"in Base"*. El compilador te está diciendo que el problema no es la clase anidada, **es su constructor**, y que para acceder a un miembro protegido de `Anidada` tendrías que ser subclase de `Anidada`, cosa que no sos.

**Los tres arreglos posibles**, en orden de preferencia:

```java
// 1. Constructor público explícito (si de verdad querés que se instancie fuera)
protected static class Anidada {
    public Anidada() {}
}

// 2. Una factoría en la clase contenedora, que sí ve lo suyo
public class Base {
    protected static class Anidada {}
    protected Anidada crearAnidada() { return new Anidada(); }
}

// 3. Heredar también de la anidada
public class Sub extends Base {
    protected static class MiAnidada extends Base.Anidada {}   // ahora sí sos subclase
}
```

> **La lección general.** Cada vez que veas «X has protected access in Y», leé con atención **cuál es la Y**. Si Y no es la clase que creías, es que el miembro inaccesible es un constructor implícito o un miembro de una clase anidada, no lo que estabas mirando.

## 15. Por qué existe esta restricción

No es un capricho ni un accidente. La propia JLS lo justifica:

> «The motivation behind the restriction on protected access is to prevent almost arbitrary access to protected members of objects. Suppose that m is a protected, nonstatic field declared in C. Without the restriction, any class X could read the content of the field m of any object of class C, using the following trick: define a subclass S of C \[...\]; declare a method in S that takes an object of class C as argument and returns the content of its m field; and have X call this method.»

El ataque, escrito en código:

```java
// Sin la restricción, esto convertiría cualquier protected en public
package cualquiera;
import libreria.ClaseConProtegido;

public class Puerta extends ClaseConProtegido {
    public static int abrir(ClaseConProtegido victima) {
        return victima.campoProtegido;   // ¡sin la regla, esto compilaría!
    }
}
```

Con esas cinco líneas, cualquiera podría leer los campos protegidos de **cualquier objeto** de la librería, con solo declarar una subclase intermedia. `protected` sería exactamente igual de abierto que `public`.

La restricción de la §6.6.2 cierra esa puerta: `victima` es de tipo `ClaseConProtegido`, no de tipo `Puerta`, así que el acceso no compila.

## 16. Cuándo usar protected

Ahora que se ve lo particular que es, la recomendación práctica:

| Situación | ¿`protected`? |
|---|---|
| Un **campo** que las subclases necesitan | **Casi nunca.** Dales un método |
| Un **método gancho** que las subclases deben poder sobrescribir | **Sí**, es su caso canónico |
| Un **constructor** de clase abstracta o clase base | **Sí**, muy idiomático |
| Un método de utilidad para el paquete | No: usá acceso de paquete |
| Algo que "por si acaso" alguien querrá extender | **No** |

Joshua Bloch, en *Effective Java*, es tajante:

> «For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. A protected member is part of the class's exported API and must be supported forever. \[...\] The need for protected members should be relatively rare.»

Esa frase — «a protected member is part of the class's exported API» — es la clave. Un miembro `protected` es tan público como uno `public`, solo que su público son las subclases. Y las subclases las escribe gente que no controlás.

**Para campos concretamente, `protected` es casi siempre un error:** acopla las subclases a tu representación interna y te impide cambiarla. Además arrastra el problema visto en el capítulo anterior: los campos se ocultan, no se sobrescriben.

---

# Parte IV — No solo miembros: clases, interfaces, enums

## 17. Clases de nivel superior: solo dos niveles

Ya lo comprobamos en la sección 4: una clase o interfaz de nivel superior solo puede ser `public` o de paquete. Bloch da la regla de uso:

> «If a top-level class or interface can be made package-private, it should be. By making it package-private, you make it part of the implementation rather than the exported API, and you can modify it, replace it, or eliminate it in a subsequent release without fear of harming existing clients. If you make it public, you are obligated to support it forever.»

En la práctica esto significa que en un paquete bien diseñado hay **una o dos clases públicas** (la fachada, la interfaz) y **el resto de paquete**.

## 18. La clase manda sobre sus miembros

Jenkov señala un punto que muchas fuentes olvidan y que es muy práctico:

> «the Java access modifier assigned to a Java class takes precedence over any access modifiers assigned to fields, constructors and methods of that class. If the class is marked with the `default` access modifier, then no other class outside the same Java package can access that class, including its constructors, fields and methods. It doesn't help that you declare these fields `public`, or even `public static`.»

Verificado:

```java
// p1/SoloPaquete.java
package p1;
class SoloPaquete { public static void visible() {} }   // clase de paquete, método público
```

```java
// p2/UsaPaquete.java
package p2;
public class UsaPaquete { void x() { p1.SoloPaquete.visible(); } }
```

Error real:

```
error: SoloPaquete is not public in p1; cannot be accessed from outside package
```

El método era `public` y da igual: **no se puede llegar a él porque no se puede nombrar la clase**. La visibilidad efectiva de un miembro es siempre **el mínimo entre la suya y la de su clase**.

Esto es una herramienta de diseño, no un estorbo: te permite escribir clases internas con métodos `public` cómodos entre ellas, sabiendo que el paquete entero es una caja cerrada.

## 19. Clases anidadas: los cuatro niveles

Una clase anidada es un **miembro** de la clase que la contiene, así que admite los cuatro niveles, igual que un campo:

```java
public class Externa {
    private   static class SoloAqui {}      // ni el paquete la ve
              static class DelPaquete {}    // la ve el paquete
    protected static class ParaHijas {}     // paquete + subclases (con la letra pequeña de la sección 14)
    public    static class ParaTodos {}     // todo el mundo
}
```

`private` en una clase anidada es enormemente útil y está infrautilizado. Bloch:

> «If a package-private top-level class (or interface) is used by only one class, consider making the top-level class a private nested class of the sole class that uses it. This reduces its accessibility from all the classes in its package to the one class that uses it.»

Es el nivel más restrictivo que existe en Java: visible desde una sola clase.

## 20. Varias clases en un mismo fichero

Un fichero `.java` puede contener varias clases de nivel superior, pero **solo una puede ser `public`, y debe llamarse igual que el fichero**:

```java
// Pedido.java
public class Pedido { ... }      // pública, coincide con el nombre del fichero
class LineaPedido { ... }        // de paquete, obligatoriamente
class CalculadoraIva { ... }     // de paquete
```

Es un patrón válido y muy útil para clases auxiliares pequeñas que solo usa la clase principal. Muchos equipos lo evitan por costumbre, pero el propio JDK lo usa.

## 21. Interfaces: público por defecto

Los miembros de una interfaz son implícitamente públicos. Jenkov lo explica así:

> «Java interfaces are meant to specify fields and methods that are publicly available in classes that implement the interfaces. Therefore you cannot use the `private` and `protected` access modifiers in interfaces. Fields and methods in interfaces are implicitly declared `public` if you leave out an access modifier, so you cannot use the default access modifier either.»

**La parte de `protected` y del acceso de paquete es correcta; la de `private` está desactualizada.** Vamos por partes.

Lo que sí es cierto:

```java
public interface Repositorio {
    int MAXIMO = 100;                  // implícitamente public static final
    void guardar(Object o);            // implícitamente public abstract
}
```

Escribir los modificadores implícitos compila, pero es ruido que los linters marcan:

```java
public static final int MAXIMO = 100;   // redundante
public abstract void guardar(Object o); // redundante
```

Y `protected` está efectivamente prohibido. Verificado:

```java
public interface IfaceErr {
    protected void malo();
}
```

```
error: modifier protected not allowed here
    protected void malo();
                   ^
```

Ahora la parte donde Jenkov se equivoca.

## 22. Métodos privados en interfaces desde Java 9

La afirmación «you cannot use the `private` \[...\] access modifier in interfaces» **es falsa desde Java 9**. El tutorial de Jenkov está fechado en 2018 y aun así no lo recoge.

Java 8 introdujo los métodos `default`, y enseguida apareció el problema: si dos métodos `default` comparten lógica, esa lógica tenía que vivir en un método público, contaminando la API. Java 9 lo resolvió permitiendo **métodos `private` en interfaces**.

Verifiquemos que compila:

```java
public interface IfaceOk {
    int CONSTANTE = 1;
    void publico();

    default void conDefault() { comun(); }

    private void comun() { System.out.println("privado de interfaz"); }
    private static void util() { System.out.println("privado estatico"); }

    static void estatico() { util(); }
}
```

Resultado real en JDK 25: **compila sin errores ni avisos**.

Las reglas de los métodos privados de interfaz:

| | Permitido |
|---|---|
| `private void m() { ... }` | Sí, con cuerpo obligatorio |
| `private static void m() { ... }` | Sí |
| `private abstract void m();` | **No**: privado y abstracto se contradicen |
| Lo llama otro método de la interfaz | Sí |
| Lo llama una clase que implementa la interfaz | **No**: es privado de la interfaz |

Caso de uso real, que es exactamente para lo que se añadieron:

```java
public interface Validador {
    List<String> errores(Solicitud s);

    default boolean esValida(Solicitud s)   { return errores(s).isEmpty(); }
    default String  resumen(Solicitud s)    { return formatear(errores(s)); }

    // detalle de implementación compartido: NO forma parte del contrato
    private String formatear(List<String> errs) {
        return errs.isEmpty() ? "OK" : String.join("; ", errs);
    }
}
```

Sin métodos privados de interfaz, `formatear` tendría que ser `default` y por tanto pública, y aparecería en todas las implementaciones y en el autocompletado de todos los usuarios.

**La tabla actualizada de visibilidad en interfaces:**

| Miembro de interfaz | Niveles posibles | Desde |
|---|---|---|
| Campo | `public static final` implícito | siempre |
| Método abstracto | `public` implícito | siempre |
| Método `default` | `public` implícito o `private` | `private` desde Java 9 |
| Método `static` | `public` implícito o `private` | `private` desde Java 9 |
| Tipo anidado | `public` implícito | siempre |

## 23. Enums y sus constructores

Un `enum` tiene una regla de visibilidad propia que sorprende: **su constructor no puede ser `public` ni `protected`**.

```java
enum E { A; public E() {} }
```

Error real:

```
error: modifier public not allowed here
enum E { A; public E() {} }
                   ^
```

**Por qué:** los valores de un `enum` son fijos y se crean solo dentro de la propia declaración. Permitir un constructor accesible desde fuera contradiría toda la idea. El constructor de un `enum` es implícitamente `private`, y solo podés escribir `private` explícito o nada.

```java
public enum Estado {
    ACTIVO("A"), CANCELADO("C");

    private final String codigo;

    Estado(String codigo) { this.codigo = codigo; }   // implícitamente private

    public String getCodigo() { return codigo; }
}
```

## 24. Records y su visibilidad

Un `record` genera automáticamente miembros con visibilidad fija:

```java
public record Punto(int x, int y) {}
```

Genera:

| Miembro generado | Visibilidad | ¿Se puede cambiar? |
|---|---|---|
| Campos `x`, `y` | `private final` | **No** |
| Accesores `x()`, `y()` | `public` | No se pueden reducir |
| Constructor canónico | la del propio record | Se puede ampliar, no reducir |

El constructor canónico debe tener **al menos** la visibilidad del record, así que esto no compila:

```java
public record Rango(int desde, int hasta) {
    private Rango { }          // ERROR: no se puede reducir la visibilidad del canónico
}
```

El patrón correcto para validar sin exponer es un constructor compacto más una factoría:

```java
public record Rango(int desde, int hasta) {
    public Rango {                                     // constructor compacto: valida
        if (desde > hasta) throw new IllegalArgumentException("rango invertido");
    }
    public static Rango de(int a, int b) { return new Rango(a, b); }
}
```

## 25. Constructores privados y clases de utilidad

Un constructor `private` impide que nadie instancie la clase desde fuera. Jenkov lo muestra y añade, con acierto: «Do not perceive the above example as an example of clever design in any way». Los usos legítimos son tres:

**1. Clase de utilidad, que no se debe instanciar nunca:**

```java
public final class Validadores {
    private Validadores() {
        throw new AssertionError("clase de utilidad, no instanciable");
    }
    public static boolean esEmail(String s) { ... }
}
```

El `throw` no es paranoia: sin él, la propia clase o la reflexión podrían crear una instancia inútil.

**2. Factorías estáticas, para controlar la creación:**

```java
public final class Conexion {
    private Conexion(String url) { ... }

    public static Conexion aBaseDeDatos(String url) { ... }
    public static Conexion enMemoria()              { ... }
}
```

Ventajas sobre un constructor público: los métodos tienen nombre, pueden devolver instancias cacheadas y pueden devolver un subtipo.

**3. Singleton**, aunque hoy la forma recomendada es un `enum`:

```java
public enum Registro {
    INSTANCIA;
    public void registrar(String s) { ... }
}
```

---

# Parte V — Visibilidad y herencia

## 26. No se puede reducir la visibilidad al sobrescribir

Jenkov lo explica correctamente:

> «the methods in the subclass cannot have less accessible access modifiers assigned to them than they had in the superclass»

Verificado:

```java
class P { public void m() {} }
class H extends P { @Override protected void m() {} }
```

Error real, con un mensaje muy claro:

```
error: m() in H cannot override m() in P
class H extends P { @Override protected void m() {} }
                                             ^
  attempting to assign weaker access privileges; was public
```

**Por qué el lenguaje lo prohíbe.** Es consecuencia directa del principio de sustitución: si `H` es un `P`, cualquier código que trate un `H` como un `P` tiene que poder llamar a todo lo que `P` promete. Si se pudiera reducir la visibilidad, esto rompería:

```java
P p = new H();
p.m();      // el compilador ve P.m(), que es pública: permite la llamada
            // en ejecución se invoca H.m(), que sería protegida: contradicción
```

## 27. Sí se puede ampliarla

La dirección contraria está permitida:

```java
class Padre { void m() {} }                        // acceso de paquete
class Hijo extends Padre { public void m() {} }    // ampliado a public: OK
```

Compila sin problemas. La escalera legal al sobrescribir:

| En la superclase | En la subclase puede ser |
|---|---|
| `private` | *(no se sobrescribe; ver sección 28)* |
| *(paquete)* | paquete, `protected`, `public` |
| `protected` | `protected`, `public` |
| `public` | solo `public` |

**Dónde se usa esto en la práctica:** `Object.clone()` es `protected`; una clase que quiera ofrecer clonación pública lo sobrescribe como `public`.

## 28. Los métodos privados no se sobrescriben

Un método `private` **no es visible desde la subclase**, así que declarar uno con la misma firma **no lo sobrescribe: crea uno nuevo, independiente**.

```java
class Padre {
    private String quien() { return "Padre"; }
    public String llamar() { return quien(); }   // se enlaza en compilación a Padre.quien()
}

class Hijo extends Padre {
    private String quien() { return "Hijo"; }    // método NUEVO, no sobrescribe nada
}
```

`new Hijo().llamar()` devuelve **`"Padre"`**, no `"Hijo"`. Es el mismo fenómeno que el ocultamiento de campos del capítulo anterior, y la señal de alarma es que **`@Override` sobre ese método no compila**: el compilador te avisa de que no estás sobrescribiendo nada.

> **Regla práctica.** Poné siempre `@Override` cuando creas estar sobrescribiendo. Es la única forma de que el compilador te diga que te equivocaste de firma, o que el método del padre era `private` y nunca fue sobrescribible.

## 29. El caso del acceso de paquete al heredar

Un caso poco conocido: un método de **acceso de paquete** sí se puede sobrescribir, pero **solo desde el mismo paquete**. Si la subclase está en otro paquete, no lo ve, así que no lo sobrescribe: declara uno nuevo, igual que con `private`.

```java
// p1
public class Padre {
    String quien() { return "Padre"; }          // acceso de paquete
    public String llamar() { return quien(); }
}

// p2
public class Hijo extends p1.Padre {
    public String quien() { return "Hijo"; }    // NO sobrescribe: Padre.quien() no era visible
}
```

`new Hijo().llamar()` devuelve `"Padre"`. Y `@Override` sobre `quien()` no compilaría.

Este comportamiento es una fuente real de bugs al extender clases de librerías: creés que estás cambiando el comportamiento y no estás cambiando nada.

---

# Parte VI — Qué pasa por debajo

## 30. Los modificadores en el fichero class

Los modificadores de acceso **no se borran al compilar**: quedan grabados en el `.class` como banderas (*flags*), y la JVM los comprueba al enlazar. De la especificación de la JVM:

| Bandera | Valor | Significado |
|---|---|---|
| `ACC_PUBLIC` | `0x0001` | accesible desde fuera del paquete |
| `ACC_PRIVATE` | `0x0002` | accesible solo dentro de la clase **y de su nest** |
| `ACC_PROTECTED` | `0x0004` | accesible desde las subclases |

Que ningún valor represente el acceso de paquete no es un descuido: **es la ausencia de las tres banderas**, igual que en el código fuente es la ausencia de palabra clave.

Consecuencia importante: **el control de acceso se comprueba dos veces**, en compilación y al cargar la clase. Si compilás `A` contra una versión de `B` donde `metodo()` era `public`, y luego ejecutás contra una versión donde pasó a ser `private`, no hay error de compilación: hay un **`IllegalAccessError` en ejecución**. Es el error clásico al mezclar versiones de un `.jar`.

## 31. Nests: cómo accede una clase interna a lo privado

Aquí está el detalle más interesante del capítulo, y no aparece en ninguna de las dos fuentes.

Vimos en la sección 6 que una clase anidada accede a los miembros `private` de la que la contiene. **Para la JVM son dos clases distintas**, en dos ficheros `.class` distintos (`Nido.class` y `Nido$Interna.class`). ¿Cómo puede una leer un campo privado de la otra?

**Hasta Java 10 la respuesta era: haciendo trampa.** El compilador generaba en secreto un método puente de acceso de paquete, llamado `access$000`, que servía de intermediario. La JEP 181 lo describe sin rodeos:

> «compilers frequently have to broaden the access of `private` members to `package`, through the addition of access bridges \[...\] **These bridges subvert encapsulation**, slightly increase the size of a deployed application, and can confuse users and tools.»

Es decir: durante veinte años, tus miembros `private` usados desde una clase anidada **en realidad eran de paquete en el bytecode**.

**Java 11 lo arregló con la JEP 181, *Nest-Based Access Control*.** Un *nest* es el conjunto formado por una clase de nivel superior y todos sus tipos anidados. Los miembros de un mismo nest pueden acceder a lo privado de los demás **directamente**, sin puentes.

Comprobémoslo en JDK 25. Con esta clase:

```java
public class Nido {
    private String secreto = "valor privado";
    private void metodoPrivado() { ... }

    class Interna {
        void tocar() { System.out.println(secreto); metodoPrivado(); }
    }
}
```

Preguntando en ejecución:

```
nest host de Interna : Nido
nest members de Nido : Nido  Nido$Interna
```

Mirando el bytecode de la clase interna con `javap -c`:

```
void tocar();
    0: getstatic     #19    // Field java/lang/System.out:Ljava/io/PrintStream;
    3: aload_0
    4: getfield      #7     // Field this$0:LNido;
    7: getfield      #25    // Field Nido.secreto:Ljava/lang/String;
```

**Accede al campo directamente con `getfield`.** No hay ninguna llamada a un método puente.

Y contando los puentes en la clase externa:

```
metodos puente access$000 en el externo (esperado: ninguno)
0
```

Los atributos nuevos del `.class`, visibles con `javap -v`:

```
NestHost: class Nido          <- en Nido$Interna.class
NestMembers:                  <- en Nido.class
  Nido$Interna
```

**Por qué esto le importa a un mid o senior**, más allá de la curiosidad:

1. **La encapsulación es real desde Java 11.** Antes, un `private` usado desde una clase anidada era alcanzable desde todo el paquete a través del puente.
2. **Cambió el comportamiento de la reflexión.** Antes, acceder por reflexión a un privado entre clases anidadas fallaba de formas sorprendentes; ahora la reflexión respeta el nest.
3. **Hay un efecto secundario documentado**: como la pertenencia al nest se guarda en el `.class` de la clase host, **ese fichero tiene que estar presente en ejecución**. Si una herramienta de empaquetado elimina una clase contenedora "vacía" que solo servía de envoltorio, obtenés un `NoClassDefFoundError` que antes no ocurría.
4. **Los nombres `access$000` que aparecían en los stack traces** desaparecieron. Si mantenés código antiguo y los ves, estás en Java 10 o anterior.

## 32. La reflexión atraviesa private

La demostración más directa de que el control de acceso no es seguridad:

```java
Nido n = new Nido();
Field f = Nido.class.getDeclaredField("secreto");
System.out.println(Modifier.toString(f.getModifiers()));
f.setAccessible(true);
System.out.println(f.get(n));
f.set(n, "MUTADO DESDE FUERA");
```

Salida real en JDK 25:

```
modificador declarado: private
leido por reflexion  : valor privado
tras escribirlo      : MUTADO DESDE FUERA
```

Un campo declarado `private` leído y **escrito** desde fuera, sin ningún truco y sin ningún aviso.

Esto no es un agujero: es una funcionalidad de la que dependen Spring, Hibernate, Jackson, JUnit y Mockito. Inyectar dependencias en campos privados o mapear un JSON a un objeto sin setters es exactamente esto.

**Lo que sí cambió** es que la reflexión ya no es ilimitada. Intentando lo mismo contra una clase del JDK:

```java
Field ff = String.class.getDeclaredField("value");
ff.setAccessible(true);
```

Resultado real en JDK 25:

```
java.lang.reflect.InaccessibleObjectException
   Unable to make field private final byte[] java.lang.String.value accessible:
   module java.base does not "opens java.lang" to unnamed module @7e9e5f8a
```

Tu propio código sí; el JDK no. La diferencia la explican los módulos.

## 33. El sistema de módulos: el nivel que no es ninguno de los cuatro

Ni Jenkov ni Baeldung mencionan los módulos en sus artículos de modificadores de acceso, y es una omisión importante: **desde Java 9, `public` dejó de significar "accesible desde cualquier parte"**.

El Java Platform Module System (JPMS) añade una pregunta previa a las cuatro de siempre: **¿el módulo exporta el paquete donde vive esta clase?**

```java
// module-info.java
module com.miempresa.pedidos {
    requires java.sql;                          // qué necesito

    exports com.miempresa.pedidos.api;          // qué ofrezco
    // com.miempresa.pedidos.interno NO se exporta: es invisible desde fuera
}
```

Una clase `public` en `com.miempresa.pedidos.interno` es **inaccesible** desde otro módulo, por muy pública que sea. Jenkov lo explica bien en su tutorial de módulos:

> «A Java module must explicitly tell which Java packages inside the module are to be exported (visible) to other Java modules \[...\] Packages that are not exported are also referred to as hidden packages, or encapsulated packages.»

**La tabla de visibilidad completa, ya con los módulos:**

| Quiero que lo vea... | Necesito |
|---|---|
| solo esta clase | `private` |
| este paquete | *(sin modificador)* |
| las subclases | `protected` |
| todo mi módulo | `public` |
| **otros módulos** | `public` **y** `exports` del paquete |
| la reflexión de otros módulos | `opens` del paquete |

La distinción entre `exports` y `opens` es la que confunde:

| Directiva | Permite | No permite |
|---|---|---|
| `exports p` | acceso normal en compilación y ejecución a los tipos públicos de `p` | reflexión profunda sobre miembros no públicos |
| `opens p` | reflexión profunda (`setAccessible`) sobre todo lo de `p` | *(no da acceso en compilación)* |
| `opens p to m` | lo anterior, solo al módulo `m` | |

Por eso los frameworks que usan reflexión piden `opens` y no `exports`: Hibernate necesita escribir campos privados de tus entidades, y `exports` no se lo permite.

## 34. Encapsulación fuerte desde Java 17

El caso de `String.value` de la sección 32 tiene una historia. La documentación de migración de Oracle la resume:

> «Some tools and libraries use reflection to access parts of the JDK that are meant for internal use only. \[...\] To aid migration, JDK 9 through JDK 16 allowed this reflection to continue, but emitted warnings about illegal reflective access. **However, JDK 17 is strongly encapsulated, so this reflection is no longer permitted by default.** Code that accesses non-public fields and methods of `java.*` APIs will throw an `InaccessibleObjectException`.»

La línea temporal, que conviene tener clara porque explica muchos errores al migrar:

| Versión | Reflexión sobre internos del JDK |
|---|---|
| 8 y anteriores | permitida sin aviso |
| 9 a 16 | permitida con **aviso** |
| **17 en adelante** | **prohibida**: `InaccessibleObjectException` |

La opción `--illegal-access`, que servía de escape, **quedó obsoleta en Java 17**: ponerla no hace nada salvo emitir un aviso.

**Por qué esto rompe proyectos al migrar.** El caso típico es una suite de tests que pasaba en Java 11 y en Java 17 falla con:

```
java.lang.reflect.InaccessibleObjectException: Unable to make field
private static final long java.util.ArrayList.serialVersionUID accessible:
module java.base does not "opens java.util" to unnamed module
```

Y el culpable no suele ser tu código: es una librería (Mockito, JMockit, un serializador antiguo) que hurgaba en internos del JDK.

## 35. add-opens y add-exports

Cuando de verdad hace falta, hay dos válvulas de escape, ambas en la línea de comandos:

```bash
# permitir reflexión profunda sobre un paquete del JDK
java --add-opens java.base/java.lang=ALL-UNNAMED -jar app.jar

# permitir acceso normal a un paquete interno no exportado
java --add-exports jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED -jar app.jar
```

`ALL-UNNAMED` significa "al módulo sin nombre", que es donde vive todo lo que cargás por classpath en lugar de por module path.

En Gradle:

```groovy
test {
    jvmArgs("--add-opens", "java.base/java.lang=ALL-UNNAMED")
    jvmArgs("--add-opens", "java.base/java.util=ALL-UNNAMED")
}
```

> **Tratalas como deuda técnica, no como solución.** Cada `--add-opens` es una promesa de que dependés de un detalle interno del JDK que puede desaparecer en la siguiente versión. Como resume una respuesta de Stack Overflow sobre el tema: *«you are playing with fire by relying on a private field of a Java SE class, because later versions of Java may change or remove it without any warning»*. La solución correcta es actualizar la librería que lo necesita.

---

# Parte VII — El paquete como unidad de diseño

## 36. El paquete no es una frontera infranqueable

Un detalle que sorprende: **cualquiera puede meter una clase en tu paquete**. No hace falta ningún permiso; basta con escribir la declaración correcta.

```java
// Un fichero cualquiera, en cualquier proyecto, en cualquier .jar
package com.tuempresa.interno;

public class Intruso {
    public static void espiar(ClaseDePaquete c) {
        System.out.println(c.campoDePaquete);   // compila: estamos "en el mismo paquete"
    }
}
```

Por eso el acceso de paquete **no es una medida de seguridad**. Es una convención de diseño que el compilador ayuda a mantener, no una barrera.

Hay dos formas de cerrar de verdad un paquete:

1. **Paquetes sellados** (*sealed packages*), declarando `Sealed: true` en el `MANIFEST.MF` del JAR. Impide que se carguen clases del mismo paquete desde otro JAR.
2. **Módulos**, que es la forma moderna: un paquete no exportado es inaccesible desde otro módulo, y **dos módulos no pueden contener el mismo paquete** (sección 39).

## 37. Package-private, el nivel más infrautilizado

Este nivel existe desde el día uno de Java, es el que sale por defecto, y casi nadie lo usa a propósito. Un artículo de ingeniería de Glovo explica por qué:

> «It was meant to be default access (Java design), but then IDEs and alternative JVM languages made `public` a default. \[...\] Because `public` is effectively a default, it's in many articles, code examples, open source projects, etc. So fewer people think package-private access is useful, etc. And it drives itself.»

Es un círculo vicioso: los IDE generan `public` al crear una clase, los tutoriales usan `public`, y la gente aprende que `public` es lo normal.

**Para qué sirve realmente.** Te permite tener un paquete con una API pública mínima y un montón de implementación oculta:

```
com.miempresa.facturacion/
├── Facturador.java          public interface   <- lo único que ve el resto
├── FacturadorImpl.java      class              <- de paquete
├── CalculadoraIva.java      class              <- de paquete
├── GeneradorNumero.java     class              <- de paquete
└── ValidadorNif.java        class              <- de paquete
```

Desde fuera del paquete, `Facturador` es todo lo que existe. Podés reescribir las otras cuatro clases enteras, fusionarlas o borrarlas, sin romper a nadie.

**Ventajas concretas**, más allá de la teoría:

1. **No aparece en el Javadoc.** Por defecto, Javadoc omite las clases de paquete: tu documentación pública muestra solo lo que de verdad es API.
2. **Reduce el autocompletado.** Quien use tu paquete ve una clase en lugar de cinco.
3. **Los frameworks modernos lo soportan.** Spring y JUnit 5 funcionan perfectamente con clases y métodos de paquete.
4. **El propio JDK lo usa intensamente.** `String` y `AbstractStringBuilder` comparten el array interno con acceso de paquete para poder compararse eficientemente sin exponerlo al mundo.

## 38. Package by feature frente a package by layer

Hay una relación directa entre cómo organizás los paquetes y cuánta visibilidad podés esconder.

**Package by layer** (por capa técnica):

```
com.miempresa/
├── controller/   PedidoController, ClienteController
├── service/      PedidoService, ClienteService
└── repository/   PedidoRepository, ClienteRepository
```

Con esta estructura, **todo tiene que ser `public`**: `PedidoController` está en otro paquete que `PedidoService`, así que el servicio no puede ser de paquete. La visibilidad no te sirve de nada.

**Package by feature** (por funcionalidad):

```
com.miempresa/
├── pedidos/     PedidoController(public), PedidoService, PedidoRepository
└── clientes/    ClienteController(public), ClienteService, ClienteRepository
```

Ahora `PedidoService` y `PedidoRepository` pueden ser de paquete: solo los usa `PedidoController`, que está al lado. **La única clase pública del paquete es la puerta de entrada.**

El mismo artículo de Glovo lo resume:

> «Contrary, package-by-layer forces you to use `public` access to make classes from different packages visible to each other. Using package-private access makes you think more in package-by-feature manner.»

Es una relación en los dos sentidos: organizar por funcionalidad te permite ocultar, y querer ocultar te empuja a organizar por funcionalidad.

## 39. Split packages

Con módulos aparece una restricción nueva que rompe proyectos al migrar: **dos módulos no pueden contener el mismo paquete**. Jenkov lo lista entre los fundamentos de los módulos ("Split Packages Not Allowed").

```
modulo-a  contiene  com.miempresa.util.Cadenas
modulo-b  contiene  com.miempresa.util.Fechas     <- MISMO paquete: prohibido
```

Antes de Java 9 esto era legal y bastante común: se repartía un paquete entre varios JAR. Con módulos, la JVM lo rechaza al arrancar.

**Por qué la restricción es correcta:** si dos módulos aportaran el mismo paquete, el acceso de paquete se convertiría en un agujero — una clase de `modulo-b` vería los miembros de paquete de `modulo-a` sin que este lo autorizara. Prohibir los paquetes partidos mantiene coherente el nivel de acceso de paquete.

## 40. Visibilidad y tests

La tensión clásica: querés testear un método que debería ser `private`.

Bloch marca el límite con precisión:

> «To facilitate testing, you may be tempted to make a class, interface, or member more accessible. This is fine up to a point. **It is acceptable to make a private member of a public class package-private in order to test it, but it is not acceptable to raise the accessibility any higher than that.**»

Es decir: `private` → paquete, sí; paquete → `public`, no. Y funciona porque **las herramientas de test ponen el código de test en el mismo paquete que el de producción**, aunque estén en directorios distintos:

```
src/main/java/com/miempresa/pedidos/Calculadora.java
src/test/java/com/miempresa/pedidos/CalculadoraTest.java   <- mismo paquete
```

Maven y Gradle juntan ambos árboles en el classpath de test, así que el test ve todo lo de paquete.

**El orden de preferencia**, de mejor a peor:

| Enfoque | Valoración |
|---|---|
| Testear el método privado **a través de la API pública** | Lo correcto casi siempre |
| Si el privado es complejo, **extraerlo a otra clase** con API propia | Suele ser la señal que faltaba |
| Subir de `private` a **paquete** y testearlo directo | Aceptable, con moderación |
| Subirlo a `public` para testearlo | **No** |
| Usar reflexión para invocar el privado | **No**: el test se rompe al renombrar |

Un estudio académico sobre 4.801 proyectos Java de Maven Central encontró que **los métodos testeados directamente son desproporcionadamente de acceso de paquete**: la gente hace exactamente lo que dice Bloch, sube la visibilidad justo lo mínimo para que el test alcance. Que sea lo más común no lo convierte en ideal, pero confirma que es la práctica aceptada.

> **La señal de alarma.** Si sentís la necesidad de testear muchos métodos privados, el problema no es la visibilidad: es que la clase hace demasiado. Cada privado complejo que querés testear por separado es una clase que está pidiendo salir.

---

# Parte VIII — Diseño

## 41. Minimizar la accesibilidad

La regla que resume todo el capítulo es el ítem de *Effective Java* dedicado a la accesibilidad:

> «**make each class or member as inaccessible as possible.** In other words, use the lowest possible access level consistent with the proper functioning of the software.»

Y el procedimiento concreto:

> «After carefully designing your class's public API, your reflex should be to make all other members private. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private. **If you find yourself doing this often, you should reexamine the design of your system.**»

La última frase es la más útil: subir la visibilidad no es solo un ajuste, es **información** de que tu descomposición en clases no es la correcta.

Baeldung llega a la misma conclusión:

> «It's good practice to use the most restrictive access level possible for any given member to prevent misuse. We should always use the private access modifier unless there's a good reason not to. Public access level should only be used if a member is part of an API.»

## 42. Qué significa publicar algo

Cuando escribís `public`, estás firmando un contrato con tres cláusulas:

1. **Alguien lo va a usar.** Si es visible, se usa. No importa si documentás que es interno.
2. **Ya no lo podés cambiar.** Cambiar la firma, el tipo de retorno o el comportamiento rompe a los clientes.
3. **No lo podés borrar.** Como mucho, marcarlo `@Deprecated` y esperar años.

Bloch lo dice de un tirón: «If you make it public, you are obligated to support it forever to maintain compatibility.»

Este es el motivo real de que las librerías serias tengan una API pública sorprendentemente pequeña. Y el motivo de que las que no lo hicieron —`java.util.Properties` heredando de `Hashtable`, visto en el capítulo anterior— arrastren su error durante treinta años.

## 43. El orden canónico de los modificadores

Baeldung dedica una sección a esto y aporta un dato que suele desconocerse:

> «The order of modifiers isn't strictly enforced in Java. However, the Java Language Specification (JLS) recommends a standard canonical order.»

El compilador acepta `final static private int X = 1;` igual que `private static final int X = 1;`. Pero la JLS recomienda un orden, y todo el ecosistema lo sigue:

**Para campos:**
```
Annotation  public/protected/private  static  final  transient  volatile
```

**Para métodos:**
```
Annotation  public/protected/private  abstract  static  final  synchronized  native  strictfp
```

**Para clases:**
```
Annotation  public/protected/private  abstract  static  final  sealed/non-sealed  strictfp
```

Ejemplo canónico:

```java
@Id
private static final long ID = 1;
```

Los IDE lo reordenan solo al formatear. Salirse del orden no rompe nada, pero es una de esas cosas que delatan código escrito sin haber leído código ajeno.

**Y una obviedad que conviene decir:** `public`, `protected` y `private` son **mutuamente excluyentes**. `public private int x;` no compila.

## 44. Árbol de decisión

Para cada cosa que declares, en este orden:

```
¿Forma parte de lo que esta clase promete al exterior?
├── NO ──> ¿Lo usa solo esta clase o sus anidadas?
│          ├── SÍ ──> private
│          └── NO ──> ¿Lo usa otra clase del MISMO paquete?
│                     ├── SÍ ──> sin modificador (acceso de paquete)
│                     └── NO ──> private, y revisá por qué existe
│
└── SÍ ──> ¿Su público son las subclases, o todo el mundo?
           ├── Subclases ──> ¿Es un método gancho o un constructor?
           │                 ├── SÍ ──> protected
           │                 └── NO (es un campo) ──> private + método protegido
           └── Todo el mundo ──> public
                                 └── ¿Estoy en un módulo?
                                     └──> además, exports del paquete
```

---

# Parte IX — Cierre

## 45. Casos de uso reales

### 45.1 Un paquete con una sola puerta de entrada

El patrón más rentable de todo el capítulo:

```java
// com/miempresa/facturacion/Facturador.java
package com.miempresa.facturacion;

public interface Facturador {                       // ÚNICO tipo público
    Factura emitir(Pedido pedido);

    static Facturador crear(Configuracion cfg) {    // factoría: oculta la implementación
        return new FacturadorImpl(new CalculadoraIva(cfg), new GeneradorNumero(cfg));
    }
}
```

```java
// com/miempresa/facturacion/FacturadorImpl.java
package com.miempresa.facturacion;

class FacturadorImpl implements Facturador {        // de paquete: invisible fuera
    private final CalculadoraIva iva;
    private final GeneradorNumero numeros;

    FacturadorImpl(CalculadoraIva iva, GeneradorNumero numeros) {   // constructor de paquete
        this.iva = iva;
        this.numeros = numeros;
    }

    @Override public Factura emitir(Pedido p) { ... }
}
```

```java
// com/miempresa/facturacion/CalculadoraIva.java
package com.miempresa.facturacion;

class CalculadoraIva {                              // de paquete
    BigDecimal calcular(Pedido p) { ... }           // método de paquete
}
```

Qué se gana: desde fuera **solo existen `Facturador` y `Factura`**. Podés partir `FacturadorImpl` en cinco clases, cambiar la firma de `CalculadoraIva.calcular` o borrarla, sin tocar a ningún cliente ni publicar una versión mayor.

### 45.2 Clase base con gancho protegido

El uso canónico y correcto de `protected`:

```java
public abstract class ProcesadorLotes {

    protected ProcesadorLotes() {}              // solo se instancia heredando

    public final Resultado ejecutar(List<Registro> registros) {   // final: el flujo no se toca
        Resultado r = new Resultado();
        for (Registro reg : registros) {
            try {
                procesar(reg);                  // el gancho
                r.exito();
            } catch (Exception e) {
                alFallar(reg, e);               // otro gancho, con comportamiento por defecto
                r.fallo();
            }
        }
        return r;
    }

    protected abstract void procesar(Registro r) throws Exception;   // obligatorio implementarlo

    protected void alFallar(Registro r, Exception e) {              // opcional sobrescribirlo
        System.getLogger("lotes").log(System.Logger.Level.WARNING, "fallo en " + r.id(), e);
    }
}
```

Decisiones y su porqué: `ejecutar` es `public final` porque es el contrato y su orden no debe cambiarse; `procesar` y `alFallar` son `protected` porque su público son las subclases; el constructor es `protected` porque la clase no tiene sentido suelta. **No hay ni un campo `protected`.**

### 45.3 Clase de utilidad cerrada

```java
public final class Nifs {                    // final: no se extiende
    private static final String LETRAS = "TRWAGMYFPDXBNJZSQVHLCKE";

    private Nifs() {                          // private: no se instancia
        throw new AssertionError("clase de utilidad");
    }

    public static boolean esValido(String nif) {
        if (nif == null || nif.length() != 9) return false;
        return letraDe(nif.substring(0, 8)) == nif.charAt(8);
    }

    private static char letraDe(String numero) {          // detalle interno
        return LETRAS.charAt(Integer.parseInt(numero) % 23);
    }
}
```

### 45.4 Método visible para test, y solo para eso

```java
public class Reconciliador {

    public Informe reconciliar(List<Movimiento> movs) {
        return new Informe(agrupar(movs), detectarDuplicados(movs));
    }

    // Acceso de paquete a propósito: la lógica es compleja y merece test directo.
    // NO es API pública. Si el test desaparece, esto vuelve a private.
    List<Grupo> agrupar(List<Movimiento> movs) { ... }

    private Set<String> detectarDuplicados(List<Movimiento> movs) { ... }
}
```

El comentario no es decorativo: documenta **por qué** la visibilidad es la que es, que es justo lo que un revisor necesita saber.

## 46. Anti-patrones

**1. `public` por defecto en todo**

```java
public class PedidoService {
    public PedidoRepository repo;
    public void validar(Pedido p) { }
    public BigDecimal calcularSubtotal(Pedido p) { }
}
```
*Arreglo:* el campo `private final`, `validar` y `calcularSubtotal` privados. Solo es público lo que otro paquete necesita llamar.

**2. Campos `protected` para las subclases**

```java
public abstract class Base { protected List<String> items = new ArrayList<>(); }
```
*Arreglo:* `private` más un método `protected List<String> getItems()`. El campo protegido acopla a las subclases a tu representación y, además, se oculta en vez de sobrescribirse.

**3. Subir a `public` para poder testear**

```java
public String formatearInterno(Pedido p) { ... }   // solo lo llama el test
```
*Arreglo:* dejarlo de paquete. El test vive en el mismo paquete y lo alcanza. Nunca subir a `public` por un test.

**4. Reflexión para saltarse `private` en código propio**

```java
Field f = Servicio.class.getDeclaredField("cache");
f.setAccessible(true);
```
*Arreglo:* si necesitás ese acceso, es que falta un método. La reflexión se rompe en silencio al renombrar el campo, porque el nombre es una cadena que el compilador no comprueba.

**5. `--add-opens` como solución permanente**

```
--add-opens java.base/java.lang=ALL-UNNAMED
```
*Arreglo:* tratarlo como deuda. Averiguá qué librería lo necesita y actualizala. Cada `--add-opens` es una apuesta a que un interno del JDK no cambie.

**6. Constructor público en una clase de utilidad**

```java
public class Utilidades { public static String limpiar(String s) { ... } }
```
*Arreglo:* añadir `private Utilidades() { throw new AssertionError(); }` y marcar la clase `final`. Sin eso, alguien la instanciará o la extenderá.

**7. Una interfaz pública por costumbre para cada clase**

```java
public interface PedidoService { ... }
public class PedidoServiceImpl implements PedidoService { ... }
```
Si hay una sola implementación y nadie de fuera necesita el tipo, la interfaz es ceremonia. *Arreglo:* una clase de paquete, y extraer la interfaz el día que haya una segunda implementación de verdad.

**8. `package by layer`, que obliga a que todo sea público**

```
controller/  service/  repository/
```
*Arreglo:* `package by feature`. Es la decisión estructural que hace posible usar el acceso de paquete.

**9. Confiar en `private` para guardar secretos**

```java
private static final String API_KEY = "sk-abc123";
```
*Arreglo:* variable de entorno o gestor de secretos. La cadena está en texto claro en el `.class`; `javap -c` la muestra en dos segundos.

**10. Escribir los modificadores implícitos de una interfaz**

```java
public interface Repo { public abstract void guardar(Object o); }
```
*Arreglo:* `void guardar(Object o);`. Los modificadores son redundantes y todos los linters los marcan.

## 47. Checklist y tabla de decisión

**Antes de dar por buena una declaración:**

- [ ] ¿Empecé por `private` y solo subí cuando el compilador me obligó?
- [ ] Si es `public`, ¿estoy dispuesto a mantenerlo para siempre?
- [ ] Si es `protected`, ¿es un método gancho o un constructor? (si es un campo, revisalo)
- [ ] Si subí la visibilidad para un test, ¿me quedé en acceso de paquete?
- [ ] ¿La clase de nivel superior podría ser de paquete en lugar de `public`?
- [ ] ¿Los modificadores están en el orden canónico?
- [ ] Si estoy en un módulo, ¿está exportado el paquete que hace falta, y solo ese?
- [ ] ¿Hay algún `--add-opens` en el build que ya nadie sabe por qué está?

**Tabla de decisión rápida:**

| Quiero que esto lo vea... | Uso |
|---|---|
| solo esta clase y sus anidadas | `private` |
| el resto de clases de este paquete | *(sin modificador)* |
| las subclases, aunque estén fuera | `protected` |
| cualquiera dentro de mi módulo | `public` |
| otros módulos | `public` + `exports` |
| un framework que usa reflexión | `opens` (no `exports`) |
| una clase auxiliar usada por una sola clase | clase anidada `private` |
| una clase auxiliar usada por el paquete | clase de nivel superior de paquete |
| un test, y nada más | subir de `private` a paquete |

## 48. Autoevaluación

**1. ¿Compila esto? `Sub` está en `p2` y `Base` en `p1`.**

```java
package p2;
public class Sub extends p1.Base {
    void m() {
        p1.Base otro = new p1.Base();     // Base tiene constructor protected
        System.out.println(otro.campo);   // campo es protected
    }
}
```

<details><summary>Respuesta</summary>

**No compila**, y falla en las dos líneas:

```
error: Base() has protected access in Base
error: campo has protected access in Base
```

La documentación de Oracle dice que un miembro `protected` es accesible «by a subclass of its class in another package», y `Sub` **es** una subclase. Pero la JLS §6.6.2 añade que el acceso por una expresión calificada `Q.Id` solo se permite si **el tipo de `Q` es la propia subclase o un subtipo suyo**. Aquí `otro` es de tipo `Base`, no de tipo `Sub`.

`this.campo` sí compilaría, y `super(...)` en el constructor también.
</details>

**2. ¿Es cierto que no se pueden usar métodos `private` en una interfaz?**

<details><summary>Respuesta</summary>

**No, es falso desde Java 9.** Lo afirma el tutorial de Jenkov, fechado en 2018, y ya estaba desactualizado al publicarse.

Esto compila sin avisos en JDK 25:

```java
public interface I {
    default void a() { comun(); }
    private void comun() { }
    private static void util() { }
}
```

Se añadieron porque los métodos `default` de Java 8 no tenían forma de compartir lógica sin exponerla: cualquier método auxiliar tenía que ser `default` y por tanto público.

Lo que sigue prohibido en interfaces es `protected` (`error: modifier protected not allowed here`) y el acceso de paquete.
</details>

**3. Este método `public` no se puede llamar desde otro paquete. ¿Por qué?**

```java
package p1;
class Ayudante { public static void util() {} }
```

<details><summary>Respuesta</summary>

Porque **la clase es de acceso de paquete**, y la visibilidad efectiva de un miembro es el mínimo entre la suya y la de su clase.

```
error: Ayudante is not public in p1; cannot be accessed from outside package
```

Da igual que el método sea `public`: desde fuera del paquete **no se puede ni nombrar la clase**, así que no hay forma de llegar al método.

Es una herramienta útil: permite escribir clases internas con métodos públicos cómodos entre ellas, sabiendo que el paquete entero es una caja cerrada.
</details>

**4. ¿Por qué falla esto, si `Sub` es subclase de `Base`?**

```java
// p1: public class Base { protected static class Anidada {} }
// p2:
public class Sub extends p1.Base {
    void m() { Base.Anidada a = new Base.Anidada(); }
}
```

<details><summary>Respuesta</summary>

```
error: Anidada() has protected access in Anidada
```

Fijate en el mensaje: el problema **no es la clase `Anidada`, es su constructor**. `Anidada` no declara ninguno, así que el compilador le genera uno por defecto, y la JLS §8.8.9 dice que ese constructor **hereda la visibilidad de la clase**: es `protected`.

A partir de ahí se aplica la JLS §6.6.2.2, que prohíbe invocar un constructor `protected` con `new` desde fuera de su paquete. Solo lo permite mediante `super(...)` o una clase anónima.

Arreglos: darle a `Anidada` un constructor `public`, ofrecer una factoría en `Base`, o hacer que `Sub` declare una anidada que extienda `Base.Anidada`.
</details>

**5. En Java 11 y posteriores, ¿cómo accede una clase interna a un campo `private` de la clase que la contiene?**

<details><summary>Respuesta</summary>

**Directamente, gracias a los *nests*** (JEP 181, Java 11).

Antes, el compilador generaba en secreto un método puente de acceso de paquete llamado `access$000` que hacía de intermediario. La propia JEP admite que «these bridges subvert encapsulation»: tu miembro `private` era, en el bytecode, de paquete.

Desde Java 11, el `.class` lleva dos atributos nuevos, `NestHost` y `NestMembers`, y la JVM permite el acceso directo. Verificado en JDK 25: el bytecode de la clase interna usa `getfield Nido.secreto` sin ningún intermediario, y la clase externa tiene **0 métodos puente**.

Efecto secundario a conocer: como la pertenencia al nest se guarda en el `.class` de la clase host, ese fichero **debe estar presente en ejecución**, aunque parezca no usarse.
</details>

**6. ¿Qué devuelve `new Hijo().llamar()`?**

```java
class Padre {
    private String quien() { return "Padre"; }
    public String llamar() { return quien(); }
}
class Hijo extends Padre {
    private String quien() { return "Hijo"; }
}
```

<details><summary>Respuesta</summary>

Devuelve **`"Padre"`**.

Un método `private` **no es visible desde la subclase**, así que `Hijo.quien()` no sobrescribe nada: es un método nuevo e independiente que casualmente se llama igual. La llamada dentro de `llamar()` se enlaza en compilación a `Padre.quien()`.

La señal que lo delata: poner `@Override` sobre `Hijo.quien()` **no compila**. Por eso conviene ponerlo siempre.

Lo mismo pasa con los métodos de acceso de paquete cuando la subclase está en otro paquete.
</details>

**7. ¿Por qué `setAccessible(true)` funciona sobre tu campo privado y falla sobre uno de `String`?**

<details><summary>Respuesta</summary>

Porque los módulos añadieron una barrera que los cuatro niveles clásicos no tienen.

Sobre tu propio código (que vive en el módulo sin nombre) funciona: verificado en JDK 25, se lee y se **escribe** un campo `private` sin ningún aviso. De eso dependen Spring, Hibernate, Jackson y JUnit.

Sobre `String.value` falla:

```
java.lang.reflect.InaccessibleObjectException: Unable to make field
private final byte[] java.lang.String.value accessible:
module java.base does not "opens java.lang" to unnamed module
```

`java.base` **no abre** (`opens`) el paquete `java.lang`. Desde JDK 9 a 16 esto emitía solo un aviso; **desde JDK 17 está prohibido** por la encapsulación fuerte. La válvula de escape es `--add-opens java.base/java.lang=ALL-UNNAMED`, que conviene tratar como deuda técnica.
</details>

**8. ¿Cuál es más restrictivo, `protected` o el acceso de paquete?**

<details><summary>Respuesta</summary>

**El acceso de paquete es más restrictivo.** El orden real es:

```
private  <  acceso de paquete  <  protected  <  public
```

`protected` incluye todo lo que ve el acceso de paquete **más** las subclases de otros paquetes. Mucha gente asume lo contrario porque "protegido" suena más cerrado que "sin nada escrito".

Consecuencia práctica: si dudás entre los dos, el acceso de paquete es la opción conservadora. `protected` publica el miembro a un conjunto de clases que ni conocés, porque cualquiera puede heredar.
</details>

**9. ¿Por qué el nivel por defecto de Java es el de paquete y no `private`?**

<details><summary>Respuesta</summary>

Por una decisión de James Gosling que él mismo lamenta. En una entrevista de 2000:

> «public would have been a really bad thing to make the default. Private would probably have been a bad thing to make a default, if only because people actually don't write private methods that often. \[...\] I decided that the most common thing that was reasonably safe was in the package.»

Y sobre los campos, explícitamente:

> «it would've made a lot of sense for the default protection for an instance variable to be private.»

También explica por qué eligió el paquete en vez del `friend` de C++: con `friend` hay que enumerar los amigos, y añadir una clase obliga a actualizar todas las demás.

Conclusión práctica: **que sea el nivel por defecto no lo convierte en el recomendado**. Para campos, escribí `private`.
</details>

**10. Tenés una clase `public` en un paquete que tu módulo no exporta. ¿Quién puede usarla?**

<details><summary>Respuesta</summary>

**Solo el código de tu propio módulo.** Desde Java 9, `public` dejó de significar "accesible desde cualquier parte".

El JPMS añade una pregunta previa a las cuatro de siempre: ¿el módulo exporta el paquete? Si `module-info.java` no dice `exports com.miempresa.interno;`, esa clase es invisible desde fuera, por muy `public` que sea.

Y hay dos directivas distintas que confundir cuesta caro:

- `exports p` — acceso normal en compilación y ejecución a los tipos públicos de `p`.
- `opens p` — reflexión profunda (`setAccessible`) sobre `p`, pero **no** da acceso en compilación.

Por eso Hibernate pide `opens` y no `exports`: necesita escribir campos privados de tus entidades por reflexión.
</details>

## 49. Fuentes

Las fuentes se listan con lo que aportan y **con sus errores señalados**. Todo lo marcado como error se comprobó en JDK 25 (Temurin 25.0.3+9).

### Fuentes primarias

- **[Java Language Specification, Java SE 25 — §6.6, *Access Control*](https://docs.oracle.com/javase/specs/jls/se25/html/jls-6.html)**. La referencia normativa. §6.6.1 define los niveles; **§6.6.2 es la sección clave de este capítulo**, la que ninguna de las dos fuentes recoge; §6.6.2.2 cubre el caso concreto de los constructores; §6.6.7 trae el ejemplo canónico con `Point` y `Point3d`.
- **JLS §8.8.9, *Default Constructor*.** Establece que el constructor implícito hereda la visibilidad de la clase, causa del error de la clase anidada `protected` (sección 14).
- **[JEP 181: Nest-Based Access Control](https://openjdk.org/jeps/181)**. Explica los métodos puente que existían antes de Java 11 y admite que «these bridges subvert encapsulation». Fuente de la sección 31.
- **[JEP 403: Strongly Encapsulate JDK Internals by Default](https://openjdk.org/jeps/403)** y la **[guía de migración de JDK 8 a versiones posteriores](https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html)**, de donde sale la línea temporal de la reflexión (permitida, con aviso, prohibida).

### Fuentes que se pidieron para este capítulo

- **[Jenkov — Java Access Modifiers](https://jenkov.com/tutorials/java/access-modifiers.html)**. La mejor de las dos en estructura: su tabla distingue correctamente clase de clase anidada, algo que casi nadie hace; acierta al aclarar que el nombre correcto es *access modifier* y no *access specifier*; y explica muy bien que la visibilidad de la clase manda sobre la de sus miembros. **Cuatro problemas:**
  - **Afirmación falsa:** «you cannot use the `private` and `protected` access modifiers in interfaces». Los métodos `private` de interfaz existen **desde Java 9** y compilan sin avisos en JDK 25 (sección 22). El tutorial está fechado en 2018.
  - **Error de compilación:** en el ejemplo del acceso de paquete escribe `public long readClock{` sin los paréntesis del método; `javac` responde `';' expected`.
  - **Error de compilación:** en el ejemplo de `protected` escribe `public class SmartClock() extends Clock{` con paréntesis tras el nombre de la clase; `javac` responde `'{' expected`.
  - **Descripción incompleta de `protected`**, la misma que la documentación de Oracle: omite la restricción de la JLS §6.6.2 (Parte III). Y llama `package` a un modificador que no existe, aunque después aclara que no se escribe.
- **[Baeldung — Access Modifiers in Java](https://www.baeldung.com/java-access-modifiers)**. Correcta y concisa. Aporta dos cosas que Jenkov no tiene: la advertencia explícita de que una clase de nivel superior solo admite `public` o *default*, y **la sección sobre el orden canónico de los modificadores** según la JLS, un detalle poco conocido y útil (sección 43). **Omisiones importantes:** no menciona la restricción de la JLS §6.6.2 sobre `protected`, ni el sistema de módulos, que desde Java 9 es una condición previa a los cuatro niveles.
- **[Baeldung — Java 'private' Access Modifier](https://www.baeldung.com/java-private-keyword)** (enlazado desde el artículo anterior). Cubre `private` en campos, constructores y métodos. **Inconsistencia:** el texto dice «we used the public constructor and the public method `changeId(customId)`» mientras el código de ese mismo ejemplo llama a `setPrivateId(...)`; los nombres y las salidas del ejemplo no cuadran entre sí.
- **[Baeldung — Java 'protected' Access Modifier](https://www.baeldung.com/java-protected-access-modifier)** (enlazado desde el artículo principal). El más útil de los tres, porque **documenta el caso de la clase anidada `protected` que no se puede instanciar desde una subclase de otro paquete**, con el mensaje `The constructor FirstClass.InnerClass() is not visible`. **El problema es que no lo explica:** dice «We were expecting to instantiate our InnerClass with ease. However, we are getting a compilation error here too» y pasa de largo. La causa —constructor implícito `protected` más JLS §6.6.2.2— es la sección 14 de este documento.
- **[Jenkov — Java Modules](https://jenkov.com/tutorials/java/modules.html)** (llegado por la navegación lateral, no estaba en el encargo). Imprescindible aquí: explica `exports`, la encapsulación de paquetes internos y la prohibición de los *split packages*.
- **[Jenkov — Java Packages](https://jenkov.com/tutorials/java/packages.html)** (llegado por la navegación lateral). Contexto sobre qué es un paquete y sus convenciones de nombres.

### Discusiones de comunidad consultadas

- **[Protected inner-class is NOT accessible from subclass of another package](https://stackoverflow.com/questions/17610498/protected-inner-class-is-not-accessible-from-subclass-of-another-package)**. El hilo que explica lo que Baeldung documenta sin explicar: cita la JLS §8.8.9 y muestra que el constructor implícito de una clase `protected` es `protected`.
- **[Protected constructor and accessibility](https://stackoverflow.com/questions/5150748/protected-constructor-and-accessibility)**. Cita literalmente la JLS §6.6.2.2 con los tres casos (`super(...)` sí, clase anónima sí, `new` no) y el ejemplo `Point`/`Point3d` de la especificación.
- **[Java: protected access across packages](https://stackoverflow.com/questions/3540640/java-protected-access-across-packages)**. La mejor explicación del *porqué* de la restricción: sin ella, `protected` sería equivalente a `public` mediante una subclase intermedia.
- **[Why did Java make package access default?](https://softwareengineering.stackexchange.com/questions/220053/why-did-java-make-package-access-default)**. Contiene la cita de **James Gosling** de 2000 usada en la sección 7, incluido su arrepentimiento explícito: «it would've made a lot of sense for the default protection for an instance variable to be private».
- **[Pros and cons of package private classes in Java?](https://stackoverflow.com/questions/6470556/pros-and-cons-of-package-private-classes-in-java)**. Aporta el dato de que las clases de paquete **no aparecen en el Javadoc**, y el ejemplo real de `String`/`AbstractStringBuilder` compartiendo su array interno con acceso de paquete.
- **[Java's default (package-private) access](https://medium.com/glovo-engineering/javas-default-access-7153b476ff5d)** (Glovo Engineering). La fuente de la sección 38: explica por qué `package by layer` obliga a que todo sea público y cómo `package by feature` hace posible ocultar.
- **[How to resolve InaccessibleObjectException for Field.setAccessible in JDK 17?](https://stackoverflow.com/questions/72982904/how-to-resolve-inaccessibleobjectexception-for-field-setaccessible-in-jdk-17)** y **[InaccessibleObjectException java 17 upgrade](https://stackoverflow.com/questions/77798030/inaccessibleobjectexception-java-17-upgrade)**. Casos reales de migración a Java 17 y uso de `--add-opens`, con el matiz de que el problema suele venir de una librería, no del código propio.
- **[Java 11 – Nest-Based Access Control](https://mkyong.com/java/java-11-nest-based-access-control/)** y **[Java Nest Based Access Control](https://www.baeldung.com/java-nest-based-access-control)**. Muestran el `javap` del método puente `access$000` antes de Java 11 y su desaparición después. Sirvieron para saber qué buscar en el bytecode; la comprobación de este documento se hizo sobre JDK 25.
- **Joshua Bloch, *Effective Java*, ítem «Minimize the accessibility of classes and members»**. La referencia de diseño de la Parte VIII: bajar todo a `private`, el límite de subir a paquete para testear y la advertencia de que un miembro `protected` es API pública para siempre.
- **[Private—Keep Out? Understanding How Developers Account for Visibility](https://philmcminn.com/publications/roslan2024.pdf)**. Estudio sobre 4.801 proyectos de Maven Central citado en la sección 40: los métodos testeados directamente son desproporcionadamente de acceso de paquete.

### Verificación

Todos los ejemplos, salidas y mensajes de error de este documento se ejecutaron en:

```
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9 (build 25.0.3+9-LTS)
```

Los mensajes de `javac` se reproducen literalmente. Se compilaron doce ficheros: dos para comprobar lo que **sí** compila (métodos `private` en interfaz; acceso a `protected` mediante `this`, `super()` y clase anónima) y el resto para capturar el texto exacto de los errores (clase de nivel superior `private`/`protected`, `protected` en interfaz, reducción de visibilidad al sobrescribir, constructor de `enum` público, los dos ejemplos de Jenkov, constructor y campo `protected` entre paquetes, clase anidada `protected` y método `public` en clase de paquete). El bytecode se inspeccionó con `javap -c -p` y `javap -v`.
