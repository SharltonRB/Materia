# Classes and Objects

> **Bloque:** `02-POO` · **Nivel de entrada:** Junior · **Versión base:** Java 17+ · **Ejemplos verificados en:** JDK 25

**Alcance de este documento.** El bloque `01-Basics` terminó con métodos y parámetros: ya sabés declarar variables ([Data Types and Variables](../01-Basics/03-data-types-and-variables.md)), dónde vive cada una ([Variables and Scopes](../01-Basics/04-variables-and-scopes.md)), cómo se convierten unos tipos en otros ([Type Casting](../01-Basics/05-type-casting.md)) y cómo se agrupa el comportamiento en métodos ([Methods and Parameters](../01-Basics/12-methods-and-parameters.md)). Todo eso describe **procedimientos sueltos**. Este capítulo cubre la unidad con la que Java organiza de verdad un programa: la **clase**, que junta datos y comportamiento, y el **objeto**, que es cada ejemplar vivo de esa clase.

**Por qué este tema decide la calidad de todo lo que escribas después.** De aquí salen los bugs que no se ven en un `if` mal escrito sino en un diseño mal planteado: objetos a medio construir que otro hilo observa, campos que se creían inmutables y no lo eran, colecciones internas que alguien modifica desde fuera, un `equals` sin `hashCode` que hace desaparecer elementos de un `HashSet`, clases de mil líneas que nadie se atreve a tocar. Todos esos casos están aquí, con el código que los reproduce.

**Lo que NO entra aquí**, porque tiene documento propio dentro de este mismo bloque: herencia y composición, polimorfismo, interfaces, clases abstractas, records en profundidad, enums y clases selladas. Este capítulo los menciona cuando hace falta para entender qué es una clase, y se detiene ahí.

---

## Índice

**Parte I — Qué es una clase y qué es un objeto**
1. [Qué problema resuelve una clase](#1-qué-problema-resuelve-una-clase)
2. [Clase, objeto, instancia y referencia](#2-clase-objeto-instancia-y-referencia)
3. [La analogía del plano y dónde se rompe](#3-la-analogía-del-plano-y-dónde-se-rompe)
4. [Anatomía completa de una clase](#4-anatomía-completa-de-una-clase)
5. [El archivo java y sus reglas](#5-el-archivo-java-y-sus-reglas)

**Parte II — El estado: campos**

6. [Campos de instancia](#6-campos-de-instancia)
7. [Los valores por defecto y por qué existen](#7-los-valores-por-defecto-y-por-qué-existen)
8. [Campos estáticos](#8-campos-estáticos)
9. [final, constantes y la trampa de la inmutabilidad](#9-final-constantes-y-la-trampa-de-la-inmutabilidad)
10. [Dónde se declara cada cosa](#10-dónde-se-declara-cada-cosa)

**Parte III — El comportamiento: métodos**

11. [Métodos de instancia y this](#11-métodos-de-instancia-y-this)
12. [Métodos estáticos y cuándo son correctos](#12-métodos-estáticos-y-cuándo-son-correctos)
13. [Getters y setters: el malentendido más extendido](#13-getters-y-setters-el-malentendido-más-extendido)

**Parte IV — Crear objetos: constructores**

14. [Qué hace new realmente](#14-qué-hace-new-realmente)
15. [El constructor](#15-el-constructor)
16. [El constructor por defecto y el error que cometen casi todas las fuentes](#16-el-constructor-por-defecto-y-el-error-que-cometen-casi-todas-las-fuentes)
17. [Sobrecarga de constructores](#17-sobrecarga-de-constructores)
18. [this() para encadenar constructores](#18-this-para-encadenar-constructores)
19. [super() y la cadena hacia Object](#19-super-y-la-cadena-hacia-object)
20. [Constructores privados y factorías estáticas](#20-constructores-privados-y-factorías-estáticas)
21. [Validar en el constructor](#21-validar-en-el-constructor)
22. [Lanzar excepciones desde un constructor](#22-lanzar-excepciones-desde-un-constructor)

**Parte V — El orden de inicialización**

23. [El orden completo, paso a paso](#23-el-orden-completo-paso-a-paso)
24. [Bloques de inicialización de instancia](#24-bloques-de-inicialización-de-instancia)
25. [Bloques estáticos](#25-bloques-estáticos)
26. [El bug del método sobrescribible en el constructor](#26-el-bug-del-método-sobrescribible-en-el-constructor)
27. [La fuga de this](#27-la-fuga-de-this)

**Parte VI — La identidad del objeto**

28. [La variable y el objeto son dos cosas distintas](#28-la-variable-y-el-objeto-son-dos-cosas-distintas)
29. [Identidad frente a igualdad](#29-identidad-frente-a-igualdad)
30. [Lo que toda clase hereda de Object](#30-lo-que-toda-clase-hereda-de-object)
31. [toString: la primera impresión de tu clase](#31-tostring-la-primera-impresión-de-tu-clase)
32. [El contrato equals y hashCode](#32-el-contrato-equals-y-hashcode)

**Parte VII — Encapsulación**

33. [Los cuatro niveles de acceso](#33-los-cuatro-niveles-de-acceso)
34. [Por qué los campos van private](#34-por-qué-los-campos-van-private)
35. [Encapsulación real frente a getters y setters automáticos](#35-encapsulación-real-frente-a-getters-y-setters-automáticos)
36. [Copias defensivas](#36-copias-defensivas)
37. [Clases inmutables](#37-clases-inmutables)

**Parte VIII — Variedades de clase**

38. [Clase concreta, abstracta e interfaz](#38-clase-concreta-abstracta-e-interfaz)
39. [Clases anidadas: las cuatro formas](#39-clases-anidadas-las-cuatro-formas)
40. [Records: la clase de datos que escribe el compilador](#40-records-la-clase-de-datos-que-escribe-el-compilador)
41. [Clases de utilidad](#41-clases-de-utilidad)
42. [Qué ocupa un objeto en memoria](#42-qué-ocupa-un-objeto-en-memoria)

**Parte IX — Diseño de clases**

43. [Una clase, una responsabilidad](#43-una-clase-una-responsabilidad)
44. [Cohesión y acoplamiento](#44-cohesión-y-acoplamiento)
45. [El modelo anémico](#45-el-modelo-anémico)
46. [La god class](#46-la-god-class)
47. [Cuándo NO crear una clase](#47-cuándo-no-crear-una-clase)

**Parte X — Cierre**

48. [Casos de uso reales](#48-casos-de-uso-reales)
49. [Anti-patrones](#49-anti-patrones)
50. [Checklist y tabla de decisión](#50-checklist-y-tabla-de-decisión)
51. [Autoevaluación](#51-autoevaluación)
52. [Fuentes](#52-fuentes)

---

# Parte I — Qué es una clase y qué es un objeto

## 1. Qué problema resuelve una clase

Antes de definir nada, hay que ver el problema. Supongamos que hay que representar tres coches en un programa que todavía no usa clases. Con lo visto en `01-Basics`, solo tenemos variables sueltas y métodos estáticos:

```java
public class SinClases {
    public static void main(String[] args) {
        String marca1 = "Toyota";  String modelo1 = "Corolla";  int velocidad1 = 0;
        String marca2 = "Ford";    String modelo2 = "Focus";    int velocidad2 = 0;
        String marca3 = "Seat";    String modelo3 = "Ibiza";    int velocidad3 = 0;

        velocidad1 = acelerar(velocidad1, 30);
        velocidad2 = acelerar(velocidad2, 50);
    }

    static int acelerar(int velocidadActual, int incremento) {
        return velocidadActual + incremento;
    }
}
```

Funciona, y es exactamente lo que no hay que hacer. Los problemas son cuatro, y conviene nombrarlos porque **la clase existe para resolver cada uno**:

1. **Los datos que van juntos no están juntos.** `marca1`, `modelo1` y `velocidad1` describen *un mismo coche*, pero el lenguaje no lo sabe. Nada impide escribir `acelerar(velocidad1, 30)` y guardar el resultado en `velocidad2`. El compilador no puede protegerte porque no tiene forma de saber que esas variables están relacionadas.

2. **No escala.** Con tres coches ya es incómodo; con cien es imposible. Y un array por cada atributo (`String[] marcas, String[] modelos, int[] velocidades`) traslada el problema: ahora hay que mantener tres arrays sincronizados por índice, y un desajuste da datos corruptos en silencio.

3. **El comportamiento está separado del dato.** `acelerar` es una función que recibe un número y devuelve otro. No tiene ninguna relación declarada con "coche". Cualquiera puede llamarla con la temperatura del horno.

4. **No hay ninguna garantía.** Nada impide `velocidad1 = -400;`. No existe un sitio donde poner la regla "la velocidad nunca es negativa", porque no existe el concepto "coche" en el programa: solo hay enteros y cadenas sueltas.

La misma idea con una clase:

```java
public class Coche {
    private final String marca;
    private final String modelo;
    private int velocidad;

    public Coche(String marca, String modelo) {
        this.marca = marca;
        this.modelo = modelo;
        this.velocidad = 0;
    }

    public void acelerar(int incremento) {
        if (incremento < 0) {
            throw new IllegalArgumentException("El incremento no puede ser negativo: " + incremento);
        }
        this.velocidad += incremento;
    }

    public int getVelocidad() {
        return velocidad;
    }
}
```

```java
Coche corolla = new Coche("Toyota", "Corolla");
Coche focus   = new Coche("Ford", "Focus");

corolla.acelerar(30);
System.out.println(corolla.getVelocidad());   // 30
System.out.println(focus.getVelocidad());     // 0  ← independiente
```

Los cuatro problemas desaparecen a la vez: los tres datos viajan juntos en una unidad con nombre, crear el coche número cien cuesta una línea, `acelerar` **pertenece** al coche y no se puede llamar sobre otra cosa, y la regla del incremento negativo tiene un sitio natural donde vivir.

> **La definición que conviene retener.** Una clase es **un tipo nuevo que vos definís**, y que agrupa datos (*estado*) con las operaciones que tienen sentido sobre esos datos (*comportamiento*), controlando quién puede tocar qué. Los tipos primitivos vienen con el lenguaje; las clases son cómo se añaden tipos propios.

W3Schools lo formula desde el otro lado, y también es útil: *"la programación procedural consiste en escribir procedimientos o métodos que operan sobre los datos; la programación orientada a objetos consiste en crear objetos que contienen a la vez los datos y los métodos"*. Esa frase resume el cambio: el dato deja de ser algo que se pasa de función en función y pasa a ser algo que **tiene** funciones.

## 2. Clase, objeto, instancia y referencia

Cuatro palabras que se usan como sinónimas y no lo son. La confusión aquí es la raíz de la mitad de los malentendidos posteriores.

| Término | Qué es | Cuándo existe | Cuántos hay |
|---|---|---|---|
| **Clase** | la definición del tipo: qué campos y qué métodos | en tiempo de **compilación**; se carga una vez en runtime | una |
| **Objeto** | una región de memoria en el heap con valores concretos | en tiempo de **ejecución**, desde `new` hasta que el GC lo recoge | tantos como `new` ejecutes |
| **Instancia** | sinónimo exacto de objeto | igual | igual |
| **Referencia** | una variable que *apunta* a un objeto | mientras esté en scope | muchas pueden apuntar al mismo objeto |

Baeldung lo dice con una frase precisa que merece la pena citar: *"mientras las clases se traducen en tiempo de compilación, los objetos se crean a partir de las clases en tiempo de ejecución"*.

```java
Coche a = new Coche("Toyota", "Corolla");   // 1 clase, 1 objeto, 1 referencia
Coche b = a;                                 // 1 clase, 1 objeto, 2 referencias
Coche c = new Coche("Toyota", "Corolla");    // 1 clase, 2 objetos, 3 referencias
```

Tras esas tres líneas hay **una** clase `Coche`, **dos** objetos en el heap y **tres** variables. `a` y `b` apuntan al mismo objeto: si acelerás por `a`, `b` ve el cambio. `c` apunta a otro objeto distinto que casualmente tiene los mismos valores.

```
        STACK                          HEAP
   ┌───────────┐            ┌────────────────────────┐
   │ a  0x100  ├───────────►│ Coche: Toyota Corolla  │
   │ b  0x100  ├───────────►│ velocidad = 0          │
   ├───────────┤            └────────────────────────┘
   │ c  0x200  ├───────────►┌────────────────────────┐
   └───────────┘            │ Coche: Toyota Corolla  │
                            │ velocidad = 0          │
                            └────────────────────────┘
```

Este esquema es el que hay que tener en la cabeza durante todo el capítulo. La [sección 28](#28-la-variable-y-el-objeto-son-dos-cosas-distintas) vuelve sobre él con las consecuencias prácticas.

**Vocabulario adicional que vas a oír:**

- **Instanciar** = crear un objeto de una clase. "Instanciá un `Coche`" significa "ejecutá `new Coche(...)`".
- **Miembro** (*member*) = cualquier campo o método de la clase. Programiz lo señala explícitamente: *"los campos y métodos de una clase también se llaman miembros de la clase"*.
- **Atributo / campo / propiedad**. W3Schools llama *attributes* a las variables de una clase y aclara que *"otro nombre para los atributos es campos (fields)"*. En el mundo Java lo estándar es **campo** (*field*); "propiedad" se reserva normalmente para el par getter/setter según la convención JavaBeans. En este documento uso **campo**.

## 3. La analogía del plano y dónde se rompe

Todas las fuentes usan la misma analogía. Programiz: *"una clase es un plano (blueprint) del objeto […] podemos pensar en la clase como el boceto de una casa: contiene todos los detalles sobre suelos, puertas y ventanas. La casa es el objeto"*. W3Schools: *"una clase es como un constructor de objetos, o un plano para crear objetos"*.

La analogía es buena y conviene usarla al principio: **el plano no es la casa**, de un plano salen muchas casas, y cambiar una casa no cambia el plano ni las demás casas.

Pero tiene dos límites que casi nadie menciona, y que producen ideas equivocadas:

**Límite 1: el plano de una casa es pasivo; la clase no.** Una clase puede tener estado propio (campos `static`) y comportamiento propio (métodos `static`) que existen **sin que haya ningún objeto**. `Math.max(2, 3)` funciona sin crear nunca un `Math`. Si te quedás con "clase = plano inerte", los miembros estáticos no encajan en tu modelo mental.

**Límite 2: sugiere que los objetos de una clase son subtipos.** Aquí Programiz se equivoca de forma didácticamente peligrosa. Dice: *"un objeto se llama instancia de una clase. Por ejemplo, si `Bicycle` es una clase, entonces `MountainBicycle`, `SportsBicycle`, `TouringBicycle`, etc. pueden considerarse objetos de la clase"*.

**No.** `MountainBicycle` y `SportsBicycle` no son objetos: son **categorías** de bicicleta, y en Java se modelarían como subclases, como un `enum` de tipos o como un campo. Un objeto de `Bicycle` es *esta bicicleta concreta, la de serie 4471, que está en el garaje y tiene la cadena engrasada*. Confundir "subcategoría" con "instancia" lleva directamente a diseñar jerarquías de herencia donde correspondía un simple campo — que es uno de los errores de diseño más caros y frecuentes.

W3Schools usa el mismo esquema (clase `Fruit`, objetos `Apple`, `Banana`, `Mango`) y arrastra el mismo problema. Una manzana concreta que tenés en la mano es una instancia; "manzana" como categoría no lo es.

> **La analogía correcta, en una línea:** la clase es el **formulario en blanco**; el objeto es **un formulario rellenado**. Hay un solo diseño de formulario y miles de formularios rellenados, cada uno con sus datos.

Y hay un tercer error de vocabulario, también de W3Schools: *"cuando se crean los objetos individuales, heredan todas las variables y métodos de la clase"*. **Los objetos no heredan de su clase.** La herencia es una relación **entre clases** (`class Coche extends Vehiculo`). Un objeto no hereda: simplemente *es* una instancia de su clase, y por eso tiene los campos y métodos que la clase declara. Usar "heredar" aquí choca de frente con el significado que la palabra tendrá en el próximo capítulo.

## 4. Anatomía completa de una clase

Jenkov enumera los bloques de construcción de una clase: **campos, constructores, métodos y clases anidadas**. Es una buena lista de partida, aunque incompleta. Esta es la anatomía completa, con todo lo que puede aparecer dentro de unas llaves de clase:

```java
package com.empresa.flota;                          // 1. paquete

import java.util.List;                              // 2. imports

public class Coche extends Vehiculo implements Comparable<Coche> {   // 3. declaración

    public static final int VELOCIDAD_MAXIMA = 180;  // 4. constante de clase
    private static int unidadesCreadas = 0;          // 5. campo estático

    private final String matricula;                  // 6. campos de instancia
    private int velocidad;

    static {                                         // 7. bloque estático
        System.out.println("Clase Coche inicializada");
    }

    {                                                // 8. bloque de instancia
        velocidad = 0;
    }

    public Coche(String matricula) {                 // 9. constructor
        this.matricula = matricula;
        unidadesCreadas++;
    }

    public void acelerar(int incremento) {           // 10. método de instancia
        this.velocidad += incremento;
    }

    public static int getUnidadesCreadas() {         // 11. método estático
        return unidadesCreadas;
    }

    @Override
    public int compareTo(Coche otro) {               // 12. método heredado/implementado
        return Integer.compare(this.velocidad, otro.velocidad);
    }

    private static class Motor { }                   // 13. clase anidada
}
```

Trece elementos, y **ninguno es obligatorio salvo la declaración**. Jenkov lo señala bien: *"no todas las clases tienen campos, constructores y métodos. A veces tenés clases que solo contienen campos (datos), y a veces clases que solo contienen métodos (operaciones)"*.

La clase válida más pequeña que existe es esta:

```java
public class MyClass {
}
```

No tiene campos, ni métodos, ni constructor escrito — y aun así se puede instanciar con `new MyClass()`, por el motivo que veremos en la [sección 16](#16-el-constructor-por-defecto-y-el-error-que-cometen-casi-todas-las-fuentes).

**El orden convencional** de los miembros dentro de la clase (no es obligación del compilador, pero sí lo que espera cualquiera que lea tu código):

1. Campos estáticos, primero las constantes públicas.
2. Campos de instancia.
3. Bloques de inicialización.
4. Constructores.
5. Métodos, agrupados por afinidad, no por visibilidad.
6. Clases anidadas.

La única regla que **sí** impone el compilador sobre el orden aparece en la [sección 23](#23-el-orden-completo-paso-a-paso): un campo no puede usarse en un inicializador situado antes de su declaración.

## 5. El archivo java y sus reglas

Reglas que el compilador aplica sin excepción, y que producen errores desconcertantes cuando se ignoran:

**Regla 1 — Una clase `public` por archivo, con el mismo nombre.**

```java
// Coche.java
public class Coche { }      // ✔ el archivo DEBE llamarse Coche.java
```

Si el archivo se llama `coche.java` con minúscula, en macOS o Windows a veces compila (el sistema de ficheros no distingue mayúsculas) y en Linux falla. Es una fuente clásica de "en mi máquina funciona".

**Regla 2 — Un archivo puede tener varias clases, pero solo una `public`.**

```java
// Coche.java
public class Coche { }
class Motor { }             // ✔ legal: package-private, misma unidad de compilación
class Rueda { }             // ✔ legal
```

Esto es válido y a veces útil para clases auxiliares muy pequeñas. Pero en cuanto una clase auxiliar crece o la usa alguien más, se mueve a su propio archivo. **Cada clase genera su propio `.class`** aunque estén en el mismo `.java`: al compilar el ejemplo anterior aparecen `Coche.class`, `Motor.class` y `Rueda.class`.

**Regla 3 — El paquete debe coincidir con la ruta de carpetas.** `package com.empresa.flota;` obliga a que el archivo esté en `com/empresa/flota/`.

**Regla 4 — Sin `package`, la clase cae en el paquete por defecto.** Aceptable para un ejercicio de una sola clase; inaceptable en cualquier proyecto real, porque **una clase del paquete por defecto no puede importarse desde otro paquete**. No hay sintaxis para hacerlo.

W3Schools muestra el patrón de dos archivos que conviene entender pronto, porque es como se organiza el código de verdad: una clase con los datos y el comportamiento, y otra distinta con el `main` que la usa.

```java
// Coche.java — el modelo
public class Coche {
    int velocidad = 5;
}
```

```java
// Aplicacion.java — el punto de entrada
class Aplicacion {
    public static void main(String[] args) {
        Coche miCoche = new Coche();
        System.out.println(miCoche.velocidad);   // 5
    }
}
```

```bash
javac Coche.java Aplicacion.java
java Aplicacion
```

Fijate en el detalle: se ejecuta `java Aplicacion`, **la clase que contiene el `main`**, no la que contiene los datos. Y desde Java 11 podés saltarte la compilación explícita de un solo archivo (`java Aplicacion.java`), pero con dos archivos que dependen entre sí lo normal sigue siendo compilar primero.

---

# Parte II — El estado: campos

## 6. Campos de instancia

Un **campo** es una variable declarada directamente dentro de la clase, fuera de cualquier método. Jenkov da la sintaxis completa, y merece la pena memorizar el orden porque el compilador lo exige:

```
[modificador_de_acceso] [static] [final] tipo nombre [= valor_inicial] ;
```

Solo **tipo y nombre** son obligatorios. Todo lo demás es opcional:

```java
public class Empleado {
    private String nombre;                    // acceso + tipo + nombre
    int edad;                                 // solo tipo + nombre (package-private)
    protected double salario = 1000.0;        // con valor inicial
    public static final String EMPRESA = "ACME";   // todo junto
}
```

**Cada objeto tiene su propia copia de cada campo de instancia.** Esta es la propiedad que define el concepto y hay que verla funcionar:

```java
public class Lampara {
    boolean encendida;

    void encender() { encendida = true; }
    void apagar()   { encendida = false; }
}
```

```java
Lampara led     = new Lampara();
Lampara halogena = new Lampara();

led.encender();

System.out.println(led.encendida);       // true
System.out.println(halogena.encendida);  // false  ← no se ha visto afectada
```

Programiz explica exactamente este punto con su ejemplo de `Lamp`: *"la variable `isOn` definida dentro de la clase se llama variable de instancia, porque cuando creamos un objeto de la clase se le llama instancia, y cada instancia tendrá su propia copia de la variable"*.

**Acceder a un campo** se hace con el operador punto sobre una referencia:

```java
led.encendida = true;         // escribir
boolean b = led.encendida;    // leer
```

Ese acceso directo solo compila si el campo es visible desde donde estás (ver [sección 33](#33-los-cuatro-niveles-de-acceso)). En un diseño correcto los campos son `private` y esto **no** compila desde fuera — que es justamente el objetivo.

## 7. Los valores por defecto y por qué existen

Los campos, a diferencia de las variables locales, **reciben un valor por defecto automáticamente**. Verificado en JDK 25:

```java
class Defaults { int i; double d; boolean b; String s; char c; }

Defaults df = new Defaults();
// i=0  d=0.0  b=false  s=null  c=código 0 (el carácter nulo)
```

| Tipo del campo | Valor por defecto |
|---|---|
| `byte`, `short`, `int` | `0` |
| `long` | `0L` |
| `float`, `double` | `0.0` |
| `char` | el carácter nulo (código 0) |
| `boolean` | `false` |
| **cualquier referencia** | **`null`** |

**Por qué existe esta garantía y las locales no la tienen.** No es un capricho: es consecuencia directa del ciclo de vida que vimos en [Lifecycle of a Program](../01-Basics/02-lifecycle-of-a-program.md). Cuando se reserva memoria para un objeto en el heap, la JVM la deja **a cero** antes de ejecutar nada. El compilador no puede demostrar en qué orden se llamarán los métodos de un objeto, así que no puede exigir "inicializá este campo antes de leerlo"; en su lugar, la plataforma garantiza que nunca leerás basura. Con las variables locales sí puede demostrarlo (análisis de *definite assignment*), y por eso te obliga a inicializarlas.

**La consecuencia práctica que muerde:** el valor por defecto de una referencia es `null`, y `null` no es un objeto vacío ni una cadena vacía.

```java
public class Usuario {
    private String nombre;              // vale null, NO ""
    private List<String> roles;         // vale null, NO una lista vacía

    public int longitudDelNombre() {
        return nombre.length();         // 💥 NullPointerException
    }
}
```

Por eso una práctica muy extendida es **inicializar las colecciones en la declaración**, para que nunca sean `null`:

```java
private final List<String> roles = new ArrayList<>();   // ✔ nunca null
```

**Un matiz de estilo importante:** no escribas explícitamente el valor por defecto.

```java
private int contador = 0;        // ✗ redundante
private String nombre = null;    // ✗ redundante y engañoso
private int contador;            // ✔
private String nombre;           // ✔
```

Jenkov escribe `public String brand = null;` en su ejemplo de `Car`. Ese `= null` no hace nada: el campo ya valía `null`. Peor aún, sugiere al lector que la asignación es necesaria, y **en el caso de un campo `final` inicializado en el constructor produce un error de compilación** si se escribe dos veces. Escribí el valor inicial solo cuando sea distinto del valor por defecto.

## 8. Campos estáticos

Un campo `static` **pertenece a la clase, no a los objetos**. Existe una sola copia, compartida por todas las instancias, y existe aunque no hayas creado ninguna.

```java
public class Contador {
    private static int total = 0;    // UNA copia, compartida
    private final int id;            // una copia POR OBJETO

    public Contador() {
        total++;
        this.id = total;
    }

    public static int getTotal() { return total; }
    public int getId() { return id; }
}
```

```java
Contador a = new Contador();
Contador b = new Contador();
Contador c = new Contador();

System.out.println(a.getId());              // 1
System.out.println(c.getId());              // 3
System.out.println(Contador.getTotal());    // 3  ← se accede por la CLASE
```

Jenkov lo ilustra con precisión: *"un campo estático pertenece a la clase. Así, no importa cuántos objetos crees de esa clase, solo existirá un campo, ubicado en la clase, y su valor es el mismo se acceda desde el objeto que se acceda"*.

**Cómo se accede:** por el nombre de la clase, `Contador.getTotal()`. Java **permite** acceder a un miembro estático a través de una instancia (`a.getTotal()`), y esto es legal pero **profundamente engañoso**: sugiere que el valor depende del objeto cuando no es así. Funciona incluso si la referencia es `null`:

```java
Contador nulo = null;
System.out.println(nulo.getTotal());   // 3 — ¡no lanza NPE!
```

No hay NPE porque la llamada se resuelve en compilación contra la clase; la referencia ni se mira. Es un comportamiento que sorprende a todo el mundo y una razón más para **acceder siempre a los estáticos por el nombre de la clase**.

**Cuándo un campo estático es correcto:**

| Uso | ¿Correcto? |
|---|---|
| Constante (`static final`) | ✔ siempre |
| Contador global de instancias creadas | ⚠️ solo para métricas o depuración |
| Caché compartida mutable | ✗ casi nunca: ver abajo |
| Configuración global mutable | ✗ estado global disfrazado |

Un campo `static` **mutable** es estado global, con todo lo que eso implica: no es thread-safe sin sincronización, contamina los tests entre sí (el valor persiste de un test a otro y el resultado depende del orden de ejecución), y actúa como **GC root**, de modo que todo lo que referencie es inmortal mientras la clase esté cargada. Es una de las causas canónicas de fugas de memoria en Java. Está tratado en detalle en [Variables and Scopes](../01-Basics/04-variables-and-scopes.md).

## 9. final, constantes y la trampa de la inmutabilidad

Un campo `final` **no puede reasignarse una vez que tiene valor**. Debe recibirlo exactamente una vez: en la declaración, en un bloque de inicialización o en **todos** los constructores.

```java
public class Pedido {
    private final String id;            // se asigna en el constructor
    private final int maxLineas = 50;   // se asigna en la declaración

    public Pedido(String id) {
        this.id = id;                   // ✔ primera y única asignación
    }

    public void cambiar(String otro) {
        this.id = otro;                 // ✗ "cannot assign a value to final variable id"
    }
}
```

W3Schools lo presenta bien con su ejemplo: `final int x = 10;` y luego `myObj.x = 25;` produce *"cannot assign a value to a final variable"*.

**`static final` = constante.** Una sola copia, inmutable, en `UPPER_SNAKE_CASE`:

```java
public static final int VELOCIDAD_MAXIMA = 180;
public static final String EMPRESA = "ACME";
```

Un detalle con consecuencias que ya vimos en el ciclo de vida: las constantes `static final` de tipo primitivo o `String` inicializadas con una expresión constante **se incrustan literalmente en el bytecode de quien las usa**. Si cambiás el valor en una librería y no recompilás el código cliente, este sigue usando el valor viejo. No es un bug: es la especificación.

### La trampa: `final` congela la referencia, no el objeto

Este es **el malentendido número uno sobre `final`**, y produce clases que se creen inmutables y no lo son:

```java
public class Configuracion {
    private final List<String> servidores = new ArrayList<>();

    public List<String> getServidores() {
        return servidores;
    }
}
```

```java
Configuracion c = new Configuracion();
c.getServidores().add("hackeado");     // ✔ compila y funciona
c.getServidores().clear();             // ✔ también
```

El campo es `final`: **la referencia** no puede apuntar a otra lista. Pero **la lista sí puede modificarse**, y como el getter la devuelve tal cual, cualquiera puede vaciarla desde fuera. La clase no protege nada.

La solución está en la [sección 36](#36-copias-defensivas). Por ahora quedate con la regla:

> `final` es una promesa sobre **la variable**, no sobre **el objeto al que apunta**. Para inmutabilidad real hace falta que el objeto apuntado también sea inmutable (`List.of(...)`, `List.copyOf(...)`) o no exponerlo nunca.

**Un beneficio de `final` que casi nadie menciona:** un campo `final` tiene garantías especiales en el modelo de memoria de Java. Si un objeto se publica correctamente, cualquier hilo que lo vea **verá sus campos `final` completamente inicializados**, sin necesidad de sincronización. Esa garantía no existe para los campos no finales. Es un motivo técnico de peso —no solo estilístico— para declarar `final` todo campo que no necesite cambiar.

## 10. Dónde se declara cada cosa

Resumen operativo de las cuatro categorías de variable, ahora vistas desde la clase:

| | Campo de instancia | Campo estático | Variable local | Parámetro |
|---|---|---|---|---|
| Dónde | en la clase | en la clase, con `static` | dentro de un método | en la firma |
| Cuántas copias | una por objeto | **una en total** | una por llamada | una por llamada |
| Valor por defecto | sí | sí | **no** | el que reciba |
| Vive en | el objeto (heap) | metaspace | stack | stack |
| Thread-safe | no, si se comparte el objeto | no | **sí, por construcción** | sí |

**El árbol de decisión:**

- ¿El dato solo se usa dentro de este método? → **variable local**.
- ¿Lo aporta quien llama? → **parámetro**.
- ¿El objeto necesita recordarlo entre llamadas? → **campo de instancia**, `private` y `final` si se puede.
- ¿Es el mismo para todas las instancias y nunca cambia? → **`static final`**.
- ¿Es el mismo para todas y **cambia**? → parate y revisá el diseño. Es estado global mutable.

---

# Parte III — El comportamiento: métodos

## 11. Métodos de instancia y this

Un método de instancia opera sobre **un objeto concreto**, y recibe una referencia implícita a ese objeto llamada `this`.

```java
public class CuentaBancaria {
    private double saldo;

    public void ingresar(double cantidad) {
        this.saldo += cantidad;      // this.saldo = el saldo DE ESTE objeto
    }
}
```

```java
cuentaAna.ingresar(100);     // dentro del método, this == cuentaAna
cuentaLuis.ingresar(50);     // dentro del método, this == cuentaLuis
```

**`this` es la respuesta a la pregunta "¿el saldo de quién?"**. Sin él, un método de instancia no tendría forma de saber sobre qué objeto trabaja.

**Cuándo `this` es obligatorio y cuándo es opcional:**

```java
public class Punto {
    private int x;

    public void setX(int x) {
        x = x;              // ✗ OBLIGATORIO aquí: sin this se asigna el parámetro a sí mismo
        this.x = x;         // ✔ el parámetro TAPA (shadowing) al campo
    }

    public void reset() {
        x = 0;              // ✔ this es opcional: no hay ambigüedad
        this.x = 0;         // ✔ equivalente, más explícito
    }
}
```

Ese `x = x;` compila sin error y **no hace nada**. Es un bug clásico que los IDE marcan como *self-assignment*; conviene elevar ese aviso a error en la configuración del proyecto.

**Los otros dos usos de `this`:**

```java
public class Builder {
    private String nombre;

    public Builder conNombre(String nombre) {
        this.nombre = nombre;
        return this;                     // 1. devolver el propio objeto → method chaining
    }

    public Builder() {
        this("por defecto");             // 2. llamar a otro constructor (sección 18)
    }

    public Builder(String nombre) { this.nombre = nombre; }
}
```

`return this` permite encadenar: `new Builder().conNombre("a").conOtraCosa(...)`.

**Un método de instancia puede acceder a todo**: campos de instancia, campos estáticos, otros métodos de instancia y métodos estáticos. Un método estático, en cambio, no puede tocar nada de instancia — no tiene `this`.

## 12. Métodos estáticos y cuándo son correctos

Un método `static` pertenece a la clase. **No tiene `this`**, y por eso no puede acceder a campos ni métodos de instancia directamente.

```java
public class Calculadora {
    private int memoria;                     // campo de instancia

    public static int sumar(int a, int b) {
        return a + b;                        // ✔ solo usa sus parámetros
    }

    public static void malo() {
        memoria = 5;                         // ✗ "non-static variable memoria
                                             //    cannot be referenced from a static context"
    }
}
```

El mensaje de error es de los más frecuentes que verá un principiante, y ahora tiene explicación exacta: un método estático puede ejecutarse **sin que exista ningún objeto**, así que la pregunta "¿la memoria de qué objeto?" no tiene respuesta.

Es también la razón por la que `main` es `static`: la JVM tiene que poder invocarlo antes de que exista ninguna instancia de tu clase.

**Cuándo un método estático es la elección correcta:**

| Situación | Ejemplo |
|---|---|
| Función pura sobre sus parámetros | `Math.max`, `Integer.parseInt` |
| Factoría con nombre | `List.of`, `Optional.empty`, `LocalDate.parse` |
| Utilidad sin estado | `Collections.sort`, `Objects.requireNonNull` |
| El `main` | punto de entrada |

**Cuándo es un error:** cuando el método *debería* depender del estado de un objeto y lo estás forzando a recibirlo por parámetro. La señal de alarma es una clase llena de métodos estáticos que reciben siempre el mismo objeto como primer argumento:

```java
// ✗ señal de que esto debería ser un método de instancia
public static double calcularInteres(Cuenta cuenta, int meses) { ... }

// ✔
public double calcularInteres(int meses) { ... }   // dentro de Cuenta
```

**Coste oculto de los estáticos:** no se pueden sobrescribir (la resolución es en compilación, no polimórfica), no se pueden sustituir por un mock en un test sin herramientas especiales, y no se pueden inyectar. Un diseño con mucha lógica de negocio en métodos estáticos es un diseño difícil de probar.

## 13. Getters y setters: el malentendido más extendido

W3Schools presenta la receta canónica: *"declarar las variables de clase como `private`; proporcionar métodos públicos get y set para acceder y actualizar el valor"*.

```java
public class Persona {
    private String nombre;

    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
}
```

La receta es correcta **como mecánica**, y hay que conocerla porque es la convención JavaBeans que usan Spring, Jackson, JPA y medio ecosistema. Pero enseñada así, sin matices, produce el malentendido más extendido de la POO:

> **Poner un getter y un setter a cada campo no es encapsular. Es exponer los campos con tres líneas más de código.**

Comparalo:

```java
public class Persona {
    public String nombre;                                  // versión A
}

public class Persona {
    private String nombre;                                 // versión B
    public String getNombre() { return nombre; }
    public void setNombre(String n) { this.nombre = n; }
}
```

Desde fuera, **A y B permiten exactamente lo mismo**: leer y escribir el nombre sin ninguna restricción. B ocupa el triple y no aporta ninguna garantía nueva. Lo único que gana B es la posibilidad *futura* de añadir lógica sin romper a los clientes — que es una ventaja real, pero mucho menor de lo que se suele vender.

**Encapsular de verdad** significa que la clase **controla** su estado, no que lo exponga a través de métodos:

```java
public class CuentaBancaria {
    private double saldo;

    public double getSaldo() { return saldo; }      // ✔ leer está bien

    // ✗ NO existe setSaldo(double): nadie puede poner el saldo a lo que quiera

    public void ingresar(double cantidad) {
        if (cantidad <= 0) {
            throw new IllegalArgumentException("El ingreso debe ser positivo: " + cantidad);
        }
        saldo += cantidad;
    }

    public void retirar(double cantidad) {
        if (cantidad <= 0) {
            throw new IllegalArgumentException("La retirada debe ser positiva: " + cantidad);
        }
        if (cantidad > saldo) {
            throw new SaldoInsuficienteException(saldo, cantidad);
        }
        saldo -= cantidad;
    }
}
```

Esta clase **no tiene setters**. Tiene operaciones con nombres del dominio (`ingresar`, `retirar`) que garantizan un **invariante**: el saldo nunca queda negativo, y nunca cambia por una cantidad no positiva. Eso es encapsulación.

Baeldung llega a esta misma conclusión en su artículo, y conviene citarlo porque va más lejos que la receta habitual: *"los campos `type` y `model` no tienen getters ni setters, porque contienen datos internos de nuestros objetos. Podemos definirlos solo a través del constructor durante la inicialización. Además, el color puede leerse y cambiarse, mientras que la velocidad solo puede leerse, no cambiarse: forzamos los ajustes de velocidad a través de métodos públicos especializados `increaseSpeed()` y `decreaseSpeed()`"*.

**La regla práctica:**

| Necesito | Escribí |
|---|---|
| Que se pueda leer un dato | un getter |
| Que se pueda cambiar de forma arbitraria | un setter — **y justificá por qué** |
| Que se pueda cambiar respetando reglas | un método con nombre del dominio |
| Que no se pueda cambiar nunca | ni setter ni método: campo `final` |

Empezá por **no** escribir el setter. Añadilo solo cuando alguien necesite de verdad esa capacidad. Un objeto sin setters es más fácil de razonar, es thread-safe con mucho menos esfuerzo y no puede quedar en un estado inválido.

---

# Parte IV — Crear objetos: constructores

## 14. Qué hace new realmente

`new Coche("Toyota", "Corolla")` parece una operación, pero son **cinco pasos** bien diferenciados. Conocerlos explica media docena de comportamientos que si no parecen mágicos:

1. **Se reserva memoria en el heap** para todos los campos de instancia de la clase y de todas sus superclases.
2. **Todos los campos se ponen a su valor por defecto** (`0`, `false`, `null`). Todavía no se ha ejecutado ni una línea de tu código.
3. **Se ejecuta el constructor**, que a su vez:
   - primero llama al constructor de la superclase (`super()`, explícito o implícito),
   - luego ejecuta los inicializadores de campos y los bloques de instancia, en orden textual,
   - y por último el cuerpo del constructor.
4. **Se devuelve la referencia** al objeto recién creado.
5. Esa referencia se asigna a la variable.

El paso 2 es el que explica por qué los campos tienen valor por defecto, y el paso 3 es el que explica el bug de la [sección 26](#26-el-bug-del-método-sobrescribible-en-el-constructor).

**`new` no es la única forma de crear objetos.** Vale la pena saberlo porque cambia lo que podés asumir:

```java
String s = "hola";                        // literal: objeto del string pool, sin new
Integer i = 42;                           // autoboxing: Integer.valueOf(42)
int[] v = {1, 2, 3};                      // array: objeto, sin new visible
Runnable r = () -> System.out.println();  // lambda: clase generada en runtime
Coche copia = (Coche) original.clone();   // clone: sin llamar al constructor
Coche desde = mapper.readValue(json, Coche.class);  // deserialización: reflexión
```

Los dos últimos son importantes para el diseño: **`clone()` y la deserialización pueden crear objetos sin pasar por tu constructor**, saltándose todas las validaciones que hayas puesto. Si tu clase depende del constructor para mantener un invariante, y alguien la deserializa desde JSON, ese invariante puede romperse. Es una de las razones por las que los frameworks de persistencia exigen un constructor sin argumentos, y por las que conviene validar también en los métodos, no solo al construir.

## 15. El constructor

Un constructor es un bloque especial que se ejecuta al crear el objeto. Jenkov lo define bien: *"un constructor Java es un método especial que se llama cuando se instancia un objeto; su propósito es inicializar el objeto recién creado antes de que se use"*.

**Tres reglas sintácticas que lo distinguen de un método:**

1. **Se llama exactamente igual que la clase**, con las mismas mayúsculas.
2. **No declara tipo de retorno.** Ni siquiera `void`.
3. No se invoca por su nombre: lo invoca `new`.

```java
public class Coche {
    private final String marca;
    private final String modelo;

    public Coche(String marca, String modelo) {   // constructor
        this.marca = marca;
        this.modelo = modelo;
    }
}
```

**El error más sutil del tema** consiste en escribir un tipo de retorno por accidente:

```java
public class Coche {
    public void Coche(String marca) {    // ✗ ESTO NO ES UN CONSTRUCTOR
        this.marca = marca;
    }
}
```

Eso es un **método normal** que casualmente se llama `Coche`. Compila sin ningún aviso. Y como la clase ya no declara ningún constructor, el compilador le añade el constructor por defecto — así que `new Coche("Toyota")` **no compila** y `new Coche()` sí, dejando la marca a `null`. Si alguna vez ves "mi constructor no se ejecuta", esto es lo primero que hay que mirar.

## 16. El constructor por defecto y el error que cometen casi todas las fuentes

Aquí es donde las fuentes se contradicen entre sí, así que vamos al texto normativo. La **JLS §8.8.9** dice literalmente:

> *"If a class contains no constructor declarations, then a default constructor with no formal parameters and no throws clause is implicitly declared."*

Las tres consecuencias, en orden de importancia:

**1. El constructor por defecto solo aparece si NO declaraste ningún constructor.** No es "todas las clases tienen uno". Verificado con `javap` en JDK 25:

```java
class SinCtor { int x; }                                   // sin constructor declarado
class ConCtor { int x; ConCtor(int x) { this.x = x; } }    // con constructor de 1 arg
```

```
$ javap SinCtor
class SinCtor {
  int x;
  SinCtor();          ← generado por el compilador
}

$ javap ConCtor
class ConCtor {
  int x;
  ConCtor(int);       ← SOLO este. No hay versión sin argumentos
}
```

**2. En cuanto declarás un constructor, el de por defecto desaparece.** Y el código que hacía `new MiClase()` deja de compilar. Es la causa número uno de la pregunta "¿por qué se rompió mi código al añadir un constructor?" en Stack Overflow.

**3. El constructor por defecto no está vacío del todo:** invoca `super()` sin argumentos. La JLS lo dice: *"the default constructor takes no parameters and simply invokes the superclass constructor with no arguments"*. De ahí se sigue un error de compilación que desconcierta: si la superclase **no** tiene constructor sin argumentos accesible, la subclase no puede tener constructor por defecto, y falla.

Y un detalle que casi nadie sabe: **el constructor por defecto hereda el modificador de acceso de la clase**. Si la clase es `public`, el constructor generado es `public`; si es package-private, el constructor también.

### El error de Baeldung, verificado

Baeldung afirma, sobre su propia clase `Car` que ya declara un constructor de tres argumentos:

> *"Every Java class has an empty constructor by default. We use it if we don't provide a specific implementation as we did above. Here's how the default constructor would look for our Car class: `Car(){}`"*

**Es falso, y su propio ejemplo lo desmiente.** Compilado tal cual en JDK 25:

```java
class Car {
    String type, model, color;
    int speed;
    Car(String type, String model, String color) { ... }
}

Car c = new Car();     // ← lo que Baeldung dice que existe
```

```
CarBaeldung.java:7: error: constructor Car in class Car cannot be applied to given types;
    public static void main(String[] a) { Car c = new Car(); }
                                                  ^
  required: String,String,String
  found:    no arguments
  reason: actual and formal argument lists differ in length
```

La clase `Car` de ese artículo **no tiene** constructor vacío, precisamente porque declara uno con parámetros. La afirmación "toda clase Java tiene un constructor vacío por defecto" es incorrecta tal como está escrita.

Hay además una segunda imprecisión en la misma frase: *"este constructor simplemente inicializa todos los campos del objeto con sus valores por defecto"*. El constructor por defecto **no inicializa nada**: los campos ya están a cero por el paso 2 de `new` (lo hace la JVM al reservar la memoria). Lo único que hace el constructor por defecto es llamar a `super()`.

### Vocabulario preciso

Una distinción que se usa mal constantemente y que conviene tener clara, porque en una entrevista se nota:

| Término | Qué es |
|---|---|
| **Constructor por defecto** (*default constructor*) | el que **genera el compilador** cuando no declarás ninguno |
| **Constructor sin argumentos** (*no-arg constructor*) | uno que **escribís vos** y no recibe parámetros |

Un no-arg constructor escrito a mano **no es** un constructor por defecto, aunque el resultado se parezca. Muchos frameworks (JPA, Jackson) exigen "un constructor sin argumentos" — y aceptan tanto el generado como el escrito a mano, pero necesitan que exista alguno.

## 17. Sobrecarga de constructores

Una clase puede tener varios constructores, siempre que sus firmas difieran en número, tipo u orden de parámetros. Es la misma mecánica de sobrecarga que vimos en [Methods and Parameters](../01-Basics/12-methods-and-parameters.md), aplicada a la construcción.

```java
public class Coche {
    private final String marca;
    private final String modelo;
    private final String color;

    public Coche(String marca, String modelo) {
        this(marca, modelo, "blanco");            // delega
    }

    public Coche(String marca, String modelo, String color) {
        this.marca = marca;
        this.modelo = modelo;
        this.color = color;
    }
}
```

**El anti-patrón del constructor telescópico.** Cuando los parámetros opcionales crecen, la sobrecarga degenera:

```java
public Pizza(int tamano) { ... }
public Pizza(int tamano, boolean queso) { ... }
public Pizza(int tamano, boolean queso, boolean pepperoni) { ... }
public Pizza(int tamano, boolean queso, boolean pepperoni, boolean champinones) { ... }
```

```java
new Pizza(30, true, false, true);    // ¿qué significa cada booleano?
```

Ilegible en la llamada, y con dos parámetros del mismo tipo consecutivos es cuestión de tiempo que alguien los invierta sin que el compilador se entere. Las salidas son un **builder** o pasar un objeto de parámetros (un `record`). Se tratan en la [sección 48](#48-casos-de-uso-reales).

## 18. this() para encadenar constructores

`this(...)` llama a otro constructor de **la misma clase**. Reglas:

- Debe ser **la primera sentencia** del constructor.
- No puede combinarse con `super()` en el mismo constructor: o uno u otro.
- No puede haber ciclos (`A` llama a `B` y `B` a `A`): el compilador lo detecta como *recursive constructor invocation*.

```java
public class Rectangulo {
    private final int ancho;
    private final int alto;

    public Rectangulo() {
        this(1, 1);                    // ✔ delega en el constructor completo
    }

    public Rectangulo(int lado) {
        this(lado, lado);              // ✔ cuadrado
    }

    public Rectangulo(int ancho, int alto) {   // ← el ÚNICO que asigna
        if (ancho <= 0 || alto <= 0) {
            throw new IllegalArgumentException("Dimensiones positivas: " + ancho + "x" + alto);
        }
        this.ancho = ancho;
        this.alto = alto;
    }
}
```

> **El patrón que conviene interiorizar:** que haya **un solo constructor que haga el trabajo real** y que todos los demás deleguen en él con `this(...)`. Así la validación y la asignación viven en un único sitio. Si cada constructor asigna por su cuenta, tarde o temprano uno se olvida de validar.

## 19. super() y la cadena hacia Object

`super(...)` llama al constructor de la superclase. Igual que `this(...)`, debe ir en la primera línea.

**Si no escribís ni `this(...)` ni `super(...)`, el compilador inserta `super()` sin argumentos.** Es decir, estos dos constructores son idénticos:

```java
public Coche() { }
public Coche() { super(); }
```

La consecuencia es que **construir un objeto siempre recorre toda la jerarquía hasta `Object`**, de arriba abajo. Con `class Deportivo extends Coche extends Vehiculo`, el orden real es: constructor de `Object` → `Vehiculo` → `Coche` → `Deportivo`.

**El error de compilación que produce:** si la superclase solo tiene constructores con parámetros, la subclase **está obligada** a llamar a `super(...)` explícitamente.

```java
class Vehiculo {
    Vehiculo(int ruedas) { }          // no hay constructor sin argumentos
}

class Coche extends Vehiculo {
    Coche() { }                        // ✗ "constructor Vehiculo in class Vehiculo
                                       //    cannot be applied to given types"
    Coche() { super(4); }              // ✔
}
```

Este es exactamente el caso que en Stack Overflow aparece como "por qué mi subclase no compila si no he tocado nada": alguien añadió un constructor con parámetros a la clase padre y eliminó sin querer su constructor por defecto.

## 20. Constructores privados y factorías estáticas

Un constructor puede tener cualquier modificador de acceso, incluido `private`. Eso **impide instanciar la clase desde fuera**, y habilita tres patrones muy usados:

**1. Factoría estática con nombre.** El patrón dominante en la JDK moderna:

```java
public class Temperatura {
    private final double celsius;

    private Temperatura(double celsius) {       // constructor privado
        this.celsius = celsius;
    }

    public static Temperatura enCelsius(double c) {
        return new Temperatura(c);
    }

    public static Temperatura enFahrenheit(double f) {
        return new Temperatura((f - 32) * 5 / 9);
    }
}
```

```java
Temperatura t1 = Temperatura.enCelsius(25);
Temperatura t2 = Temperatura.enFahrenheit(77);
```

Con constructores sobrecargados esto **sería imposible**: `Temperatura(double)` dos veces es la misma firma. Las factorías resuelven eso y además:

- **Tienen nombre**, así que la llamada se lee sola.
- **No están obligadas a crear un objeto nuevo**: pueden devolver uno cacheado (`Integer.valueOf`, `Boolean.valueOf`).
- **Pueden devolver un subtipo**, lo que permite cambiar la implementación sin tocar a los clientes (`List.of` devuelve clases distintas según el número de elementos).

Ejemplos de la JDK: `List.of`, `Optional.of`, `Integer.valueOf`, `LocalDate.parse`, `Stream.empty`.

**2. Clase de utilidad no instanciable** (ver [sección 41](#41-clases-de-utilidad)).

**3. Singleton.** El idioma correcto en Java aprovecha las garantías de inicialización de clase que vimos en el ciclo de vida:

```java
public class Servicio {
    private Servicio() { }

    private static class Holder {
        static final Servicio INSTANCIA = new Servicio();
    }

    public static Servicio getInstancia() { return Holder.INSTANCIA; }
}
```

Perezoso y thread-safe sin escribir un solo `synchronized`, porque la JVM garantiza que `<clinit>` se ejecuta una sola vez. (Dicho esto: el singleton es un patrón del que se abusa muchísimo. En una aplicación moderna, lo que casi siempre querés es un objeto normal cuyo ciclo de vida gestione el framework de inyección de dependencias.)

## 21. Validar en el constructor

El constructor es **la frontera** de tu clase: el único punto por el que un objeto entra en existencia. Si validás ahí, el resto de la clase puede asumir que el estado es válido.

```java
public class Usuario {
    private final String email;
    private final int edad;

    public Usuario(String email, int edad) {
        this.email = Objects.requireNonNull(email, "email no puede ser null");
        if (email.isBlank()) {
            throw new IllegalArgumentException("email no puede estar en blanco");
        }
        if (edad < 0 || edad > 150) {
            throw new IllegalArgumentException("edad fuera de rango: " + edad);
        }
        this.edad = edad;
    }
}
```

**Qué excepción lanzar:**

| Problema | Excepción |
|---|---|
| Argumento `null` no permitido | `NullPointerException` (vía `Objects.requireNonNull`) |
| Valor fuera de rango o con formato inválido | `IllegalArgumentException` |
| Combinación de argumentos incoherente | `IllegalArgumentException` |

Que `requireNonNull` lance `NullPointerException` sorprende, pero es lo correcto y es lo que hace toda la JDK: falla **inmediatamente y con un mensaje que dice qué era null**, en vez de dejar que el `null` se propague y explote tres capas más abajo sin contexto.

> **El principio, que vale para todo el bloque de POO:** *fail fast*. Un objeto mal construido que no falla al construirse es una bomba de relojería: el error aparecerá lejos, en otro hilo o en otra petición, y el stack trace no dirá nada del sitio real donde se originó.

## 22. Lanzar excepciones desde un constructor

Un constructor puede lanzar excepciones, checked incluidas. Si lo hace, **el objeto nunca llega a existir para quien lo pidió**: `new` no devuelve nada y la referencia no se asigna.

```java
public class Conexion {
    public Conexion(String url) throws IOException {
        if (!esValida(url)) {
            throw new IOException("URL inaccesible: " + url);
        }
    }
}
```

**El matiz importante:** el objeto **sí se creó** en el heap antes de que el constructor fallara; simplemente queda inalcanzable y el GC lo recogerá. Eso importa cuando el constructor ya había adquirido un recurso:

```java
public class Recurso {
    private final FileInputStream fichero;

    public Recurso(String ruta, int limite) throws IOException {
        this.fichero = new FileInputStream(ruta);      // recurso adquirido
        if (limite < 0) {
            throw new IllegalArgumentException("limite negativo");   // ✗ ¡fichero queda abierto!
        }
    }
}
```

Si la validación falla después de abrir el fichero, **el descriptor queda abierto y nadie lo cerrará**, porque nadie tiene una referencia al objeto para llamar a `close()`. La corrección es sencilla y es una buena regla general: **validá antes de adquirir nada**.

```java
public Recurso(String ruta, int limite) throws IOException {
    if (limite < 0) {                                  // ✔ validar primero
        throw new IllegalArgumentException("limite negativo");
    }
    this.fichero = new FileInputStream(ruta);          // adquirir después
}
```

---

# Parte V — El orden de inicialización

## 23. El orden completo, paso a paso

Este es uno de los pocos temas donde conviene memorizar una secuencia, porque explica bugs reales y aparece en entrevistas. El orden lo fija la JLS (§12.4 y §12.5), y es este:

**La primera vez que se usa la clase (una sola vez en toda la vida del programa):**

1. Se inicializa la **superclase** por completo.
2. Se ejecutan los **campos estáticos** y los **bloques `static`**, mezclados, **en orden textual**.

**Cada vez que se crea un objeto:**

3. Se reserva memoria y **todos los campos de instancia van a su valor por defecto**.
4. Se ejecuta el constructor, que empieza por `this(...)` o `super(...)`.
5. Se ejecutan los **inicializadores de campos de instancia** y los **bloques de instancia**, mezclados, **en orden textual**.
6. Se ejecuta el **resto del cuerpo del constructor**.

Fijate en el punto crítico: **los pasos 5 y 6 ocurren DESPUÉS del constructor de la superclase**. Esa inversión es la causa del bug de la sección 26.

Verificado en JDK 25:

```java
class Super {
    Super() { System.out.println("2. ctor Super"); }
}
class Test extends Super {
    private int three = 3;
    { System.out.println("3. bloque de instancia de Test"); }
    Test() { super(); System.out.println("4. cuerpo del ctor de Test"); }
}
```

```
1. antes del new
2. ctor Super
3. bloque de instancia de Test
4. cuerpo del ctor de Test
```

**La restricción de orden textual** que sí impone el compilador:

```java
public class Trampa {
    int a = b + 1;     // ✗ "illegal forward reference"
    int b = 10;
}
```

Con campos de instancia el compilador lo rechaza. Con campos **estáticos** accedidos a través del nombre de la clase, en cambio, compila y da un resultado sorprendente (`a` vale 1, no 11), porque en ese momento `b` todavía tiene su valor por defecto. Es el mismo fenómeno de la fase de *preparación* que vimos en [Lifecycle of a Program](../01-Basics/02-lifecycle-of-a-program.md).

## 24. Bloques de inicialización de instancia

Un bloque de instancia son unas llaves sueltas dentro de la clase, sin `static`:

```java
public class Ejemplo {
    private List<String> items;

    {
        items = new ArrayList<>();
        items.add("por defecto");
    }
}
```

**Qué hace el compilador con ellos:** los **copia al principio de cada constructor**, justo después de la llamada a `super()`. La documentación oficial de Oracle lo dice explícitamente: *"el compilador de Java copia los bloques inicializadores dentro de cada constructor"*. Se puede comprobar con `javap`: el bytecode del constructor contiene las asignaciones del bloque antes que las suyas propias.

**Cuándo usarlos:** prácticamente nunca. Su único caso legítimo es compartir código de inicialización entre **varios** constructores que no pueden delegar entre sí — y casi siempre es mejor arreglar eso con `this(...)`. Su gran problema es que **separan el código de inicialización del constructor**, así que quien lee el constructor no ve todo lo que ocurre.

Donde sí se ven en la práctica es en el idioma de la "doble llave", que conviene **reconocer para no usarlo**:

```java
List<String> lista = new ArrayList<>() {{    // ✗ anti-patrón
    add("a");
    add("b");
}};
```

Eso crea una **subclase anónima** de `ArrayList` con un bloque de instancia. Parece elegante y tiene tres problemas: genera una clase extra por cada uso, mantiene una referencia implícita a la clase que la contiene (posible fuga de memoria), y rompe `equals` en algunos contextos. Desde Java 9 la alternativa es `List.of("a", "b")`.

## 25. Bloques estáticos

Se ejecutan **una sola vez**, cuando la clase se inicializa, y sirven para lógica que no cabe en una asignación simple:

```java
public class Configuracion {
    private static final Map<String, String> DEFECTOS;

    static {
        Map<String, String> m = new HashMap<>();
        m.put("host", "localhost");
        m.put("puerto", "8080");
        DEFECTOS = Collections.unmodifiableMap(m);
    }
}
```

Oracle lo formula así: *"una clase puede tener cualquier número de bloques de inicialización estática, y pueden aparecer en cualquier parte del cuerpo de la clase. El sistema en tiempo de ejecución garantiza que se llaman en el orden en que aparecen en el código fuente"*.

**Tres cosas que hay que saber:**

- Se ejecutan en el **primer uso real** de la clase, no al arrancar el programa. Java carga clases perezosamente.
- La JVM garantiza que ocurren **una sola vez y de forma thread-safe**, aunque compitan veinte hilos. Es la base del idioma del *holder* de la sección 20.
- **Si un bloque estático lanza una excepción**, obtenés un `ExceptionInInitializerError`, y todos los usos posteriores de esa clase fallan con `NoClassDefFoundError`. El segundo error no dice nada del problema real: hay que buscar el primero, más arriba en el log.

## 26. El bug del método sobrescribible en el constructor

Este es el bug clásico de la Parte V, aparece en entrevistas senior y en código de producción. Verificado en JDK 25:

```java
class Super {
    Super() {
        System.out.println("ctor Super -> llama a printThree()");
        printThree();                       // ✗ método sobrescribible
    }
    void printThree() { System.out.println("   (Super.printThree)"); }
}

class Test extends Super {
    private int three = 3;
    @Override void printThree() {
        System.out.println("   Test.printThree ve three = " + three);
    }
}
```

```java
Test t = new Test();
t.printThree();
```

**Salida real:**

```
ctor Super -> llama a printThree()
   Test.printThree ve three = 0        ← ¡0, no 3!
   Test.printThree ve three = 3        ← ahora sí
```

**Por qué.** El constructor de `Super` se ejecuta **antes** de los inicializadores de campos de `Test` (paso 4 antes que paso 5, sección 23). Pero la llamada a `printThree()` es **polimórfica**: se despacha a la versión de `Test`, que ya existe. Resultado: un método de `Test` se ejecuta sobre un objeto `Test` **que todavía no está inicializado**, y ve `three = 0`, el valor por defecto.

Con un campo `final` es aún más desconcertante: se puede observar un campo `final` con dos valores distintos a lo largo de la vida del objeto.

**La regla, sin excepciones:**

> Un constructor solo debe llamar a métodos **`private`, `static` o `final`** de su propia clase. Esos tres son los únicos que no se pueden sobrescribir. Si la clase entera es `final`, cualquier método suyo es seguro.

Es exactamente lo que recomienda la documentación de Oracle cuando propone usar *final methods* para inicializar campos: *"el método es `final` porque llamar a métodos no finales durante la inicialización de la instancia puede causar problemas"*.

## 27. La fuga de this

La variante avanzada del problema anterior, y de consecuencias más graves porque involucra concurrencia:

```java
public class Servicio {
    private final Config config;

    public Servicio(Registro registro) {
        registro.registrar(this);          // ✗ 'this' escapa antes de terminar
        this.config = new Config();
    }
}
```

Publicar `this` antes de que el constructor termine significa que otro código —o **otro hilo**— puede observar el objeto **a medio construir**, con campos `final` todavía sin asignar. Rompe incluso las garantías especiales que el modelo de memoria de Java da a los campos `final`, que mencionamos en la sección 9.

**Las tres formas en que `this` se fuga**, y no siempre son evidentes:

```java
public class Fuga {
    public Fuga(EventBus bus) {
        bus.registrar(this);                       // 1. pasarlo explícitamente

        bus.registrar(new Listener() {             // 2. una clase interna NO estática
            public void onEvent() { }              //    captura this implícitamente
        });

        new Thread(this::procesar).start();        // 3. arrancar un hilo con this
    }
}
```

La segunda es la más traicionera: una clase interna no estática (y una lambda que capture un miembro de instancia) **contiene una referencia implícita al objeto que la contiene**, aunque no escribas `this` en ninguna parte.

**La regla:** no dejes que `this` salga del constructor, ni directamente, ni registrando listeners, ni arrancando hilos. Si necesitás registrarte en algún sitio, hacelo en un método `iniciar()` separado que se llame **después** de construir, o usá una factoría estática que construya primero y registre después:

```java
public static Servicio crear(Registro registro) {
    Servicio s = new Servicio();       // construcción completa
    registro.registrar(s);             // publicación después
    return s;
}
```

---

# Parte VI — La identidad del objeto

## 28. La variable y el objeto son dos cosas distintas

Ya lo vimos en la sección 2, pero ahora con las consecuencias prácticas, que son las que producen bugs.

**Asignar una referencia no copia el objeto:**

```java
Coche a = new Coche("Toyota", "Corolla");
Coche b = a;              // NO es una copia: es otro nombre para el mismo objeto

b.acelerar(50);
System.out.println(a.getVelocidad());    // 50  ← a también cambió
```

**Pasar un objeto a un método tampoco lo copia.** Java **siempre pasa por valor**, pero lo que se copia es *la referencia*:

```java
static void pintar(Coche c) {
    c.setColor("rojo");        // ✔ muta el objeto: se ve fuera
    c = new Coche("Otro");     // ✗ reasigna la copia local: NO se ve fuera
}
```

Es exactamente el mecanismo de [Methods and Parameters](../01-Basics/12-methods-and-parameters.md), y explica por qué un método `intercambiar(a, b)` no puede funcionar en Java.

**Copiar de verdad un objeto** requiere trabajo explícito, y hay tres niveles:

```java
// 1. Copia superficial: campos copiados, objetos referenciados COMPARTIDOS
Coche copia = new Coche(original.getMarca(), original.getModelo());

// 2. Constructor de copia (el idioma preferido en Java)
public Coche(Coche otro) {
    this.marca = otro.marca;
    this.opciones = new ArrayList<>(otro.opciones);   // copia la lista
}

// 3. Copia profunda: también se copian los objetos referenciados, recursivamente
```

Sobre `clone()`: existe, pero está desaconsejado desde hace años (*Effective Java*, item 13). Es superficial por defecto, requiere implementar `Cloneable` —una interfaz vacía y engañosa—, no llama al constructor y su contrato es ambiguo. **Usá un constructor de copia o una factoría estática.**

## 29. Identidad frente a igualdad

Dos preguntas distintas que el principiante confunde y que en Java tienen operadores distintos:

| Pregunta | Operador | Qué compara |
|---|---|---|
| ¿Son **el mismo objeto**? | `==` | la referencia: la dirección |
| ¿Son **equivalentes**? | `.equals()` | lo que la clase decida |

```java
Coche a = new Coche("Toyota", "Corolla");
Coche b = new Coche("Toyota", "Corolla");
Coche c = a;

a == b;          // false ← objetos distintos, aunque los datos coincidan
a == c;          // true  ← misma referencia
a.equals(b);     // depende: sin equals propio, false (ver abajo)
```

Verificado en JDK 25 con una clase sin `equals` propio: `x == y` da `false` y `x.equals(y)` **también da `false`**, porque el `equals` heredado de `Object` compara identidad.

**La regla operativa:** `==` solo para primitivos, para enums y para comprobar `null`. Para todo lo demás, `equals` — y si la referencia puede ser nula, `Objects.equals(a, b)`.

## 30. Lo que toda clase hereda de Object

**Toda clase en Java hereda de `java.lang.Object`**, aunque no escribas `extends`. Estas dos declaraciones son idénticas:

```java
public class Coche { }
public class Coche extends Object { }
```

Eso significa que tu clase, desde el primer día y sin que escribas nada, ya tiene estos métodos:

| Método | Qué hace por defecto | ¿Conviene sobrescribirlo? |
|---|---|---|
| `toString()` | `NombreClase@hashHex` | **Casi siempre sí** |
| `equals(Object)` | compara identidad (`==`) | Sí, si el objeto representa un *valor* |
| `hashCode()` | número derivado de la identidad | **Siempre que sobrescribas `equals`** |
| `getClass()` | el objeto `Class` en runtime | No: es `final` |
| `clone()` | copia superficial, `protected` | Preferí un constructor de copia |
| `finalize()` | nada útil | **No: deprecado desde Java 18** |
| `wait`, `notify`, `notifyAll` | primitivas de concurrencia | No: son `final` |

Es la razón por la que `Object` es el tipo más general del lenguaje y por la que una `List<Object>` puede contener cualquier cosa.

## 31. toString: la primera impresión de tu clase

Por defecto obtenés algo así, verificado:

```java
System.out.println(new Defaults());     // Defaults@1460a8c0
```

Ese texto es el nombre de la clase y el `hashCode` en hexadecimal. **No sirve para nada** salvo confirmar que hay un objeto. Y aparece exactamente donde más molesta: en los logs de producción, en los mensajes de error y en el depurador.

```java
@Override
public String toString() {
    return "Coche[marca=" + marca + ", modelo=" + modelo + ", velocidad=" + velocidad + "]";
}
```

**Tres reglas para un buen `toString`:**

1. **Que incluya la información que identifica al objeto**, no todos sus campos.
2. **Que nunca lance una excepción.** Un `toString` que falla rompe el log que intentaba diagnosticar otro problema, y el stack trace resultante apunta al sitio equivocado.
3. **Que nunca incluya datos sensibles.** Contraseñas, tokens, números de tarjeta y datos personales acaban en los ficheros de log, que suelen tener menos protección que la base de datos.

```java
@Override
public String toString() {
    return "Usuario[email=" + email + ", password=" + password + "]";   // ✗ fuga de credenciales
}
```

Este es un fallo de seguridad real y frecuente: alguien añade un `log.debug("usuario: {}", usuario)` y las contraseñas terminan en texto plano en el disco.

## 32. El contrato equals y hashCode

**La regla que hay que grabar:** si sobrescribís `equals`, **tenés que** sobrescribir `hashCode`. No es una recomendación de estilo, es un contrato del que dependen todas las colecciones basadas en hash.

**Qué pasa si lo incumplís:**

```java
public class Punto {
    private final int x, y;
    public Punto(int x, int y) { this.x = x; this.y = y; }

    @Override
    public boolean equals(Object o) {           // ✗ equals SIN hashCode
        if (!(o instanceof Punto p)) return false;
        return x == p.x && y == p.y;
    }
}
```

```java
Set<Punto> set = new HashSet<>();
set.add(new Punto(1, 2));
set.contains(new Punto(1, 2));      // false  😱
```

El objeto está en el `Set`, `equals` dice que son iguales, y `contains` dice que no está. La razón: `HashSet` primero calcula el `hashCode` para saber **en qué cubeta buscar**; como los dos objetos tienen hashes distintos (heredados de la identidad), mira en una cubeta equivocada y ni siquiera llega a llamar a `equals`. El elemento se vuelve **inalcanzable**, y como clave de un `HashMap` produce fugas de memoria: entradas que nunca se pueden recuperar ni borrar.

**El contrato de `equals`** (JLS y Javadoc de `Object`), cinco propiedades:

| Propiedad | Significa |
|---|---|
| **Reflexiva** | `x.equals(x)` es `true` |
| **Simétrica** | si `x.equals(y)` entonces `y.equals(x)` |
| **Transitiva** | si `x.equals(y)` e `y.equals(z)`, entonces `x.equals(z)` |
| **Consistente** | invocaciones repetidas dan lo mismo si el objeto no cambia |
| **Frente a null** | `x.equals(null)` es siempre `false`, nunca lanza |

**El contrato de `hashCode`:**

- Si dos objetos son `equals`, **deben** tener el mismo `hashCode`.
- Lo contrario **no** es obligatorio: dos objetos distintos pueden compartir hash (colisión). Es normal y las colecciones lo gestionan.

**La implementación correcta hoy:**

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;                  // atajo de rendimiento
    if (!(o instanceof Punto p)) return false;   // cubre null y tipo incorrecto a la vez
    return x == p.x && y == p.y;
}

@Override
public int hashCode() {
    return Objects.hash(x, y);
}
```

`instanceof` con patrón (Java 16+) resuelve en una línea lo que antes eran tres: descarta `null`, comprueba el tipo y declara la variable ya casteada.

**Tres avisos prácticos:**

1. **No uses campos mutables en `equals`/`hashCode`** si el objeto va a vivir en un `Set` o como clave de un `Map`. Si el campo cambia, el hash cambia, y el objeto queda en la cubeta equivocada: desaparece de su propia colección.
2. **La firma es `equals(Object)`, no `equals(MiClase)`.** Escribir `public boolean equals(Punto p)` crea una **sobrecarga**, no una sobrescritura: compila, parece funcionar en tus pruebas y las colecciones siguen usando el `equals` de `Object`. Poné siempre `@Override`, que convierte ese error en un fallo de compilación.
3. **Si la clase solo transporta datos, usá un `record`** ([sección 40](#40-records-la-clase-de-datos-que-escribe-el-compilador)) y el compilador escribe los tres métodos por vos, correctamente.

---

# Parte VII — Encapsulación

## 33. Los cuatro niveles de acceso

| Modificador | La propia clase | Mismo paquete | Subclase (otro paquete) | Todos |
|---|---|---|---|---|
| `private` | ✔ | ✗ | ✗ | ✗ |
| *(ninguno)* — package-private | ✔ | ✔ | ✗ | ✗ |
| `protected` | ✔ | ✔ | ✔ | ✗ |
| `public` | ✔ | ✔ | ✔ | ✔ |

**Para clases de nivel superior** solo hay dos opciones: `public` o package-private. No existe una clase de nivel superior `private` o `protected`.

Dos matices que casi ninguna fuente menciona y que sí se preguntan:

- **`protected` incluye el paquete.** Un miembro `protected` es visible para *todo* el paquete, no solo para las subclases. Es más permisivo de lo que la mayoría cree.
- **`protected` en otro paquete solo funciona a través de la propia subclase.** Una subclase en otro paquete puede acceder a `this.campoProtegido`, pero **no** a `otraInstancia.campoProtegido` si esa instancia es del tipo padre.

**La regla de diseño:** empezá por `private` en todo y **subí el nivel solo cuando alguien lo necesite de verdad**. Es asimétrico y por eso importa: ampliar la visibilidad después es trivial; reducirla rompe a todos los que ya dependían de ella. Cada miembro `public` es una promesa que vas a tener que mantener.

## 34. Por qué los campos van private

Un campo público no es solo un riesgo de escritura: es una **decisión de diseño irreversible**.

```java
public class Circulo {
    public double radio;          // ✗
}
```

Los cuatro problemas concretos:

1. **No hay invariantes.** `c.radio = -5;` compila. La clase no puede impedirlo.
2. **No se puede cambiar la representación.** Si mañana querés guardar el diámetro, o cachear el área, o pasar a `BigDecimal`, no podés: todo el código que hace `c.radio` se rompe.
3. **No se puede observar el acceso.** No hay dónde poner un log, una métrica, una carga perezosa o una comprobación de permisos.
4. **No hay control de concurrencia.** Cualquiera escribe el campo desde cualquier hilo sin sincronización.

Con el campo privado, los cuatro problemas siguen abiertos como opciones futuras, y ninguno rompe a los clientes:

```java
public class Circulo {
    private double radio;

    public Circulo(double radio) {
        if (radio <= 0) throw new IllegalArgumentException("radio debe ser positivo: " + radio);
        this.radio = radio;
    }

    public double getRadio() { return radio; }
    public double getArea() { return Math.PI * radio * radio; }
}
```

**La excepción legítima:** las constantes `public static final` de tipos inmutables. `public static final int MAX = 100;` está bien. Un array o una colección mutable como constante pública **no** lo está, porque no es realmente constante (ver sección 36).

Jenkov comenta que *"para clases simples que transportan datos podés declarar todos los campos `public`"*. Es una postura defendible en código de usar y tirar, pero para eso Java tiene desde la versión 16 una herramienta específica y mucho mejor: los `record`, que dan campos públicos de solo lectura con `equals`, `hashCode` y `toString` incluidos.

## 35. Encapsulación real frente a getters y setters automáticos

Ampliando lo de la sección 13, porque es el punto donde se separa un diseño mid de uno junior.

**Encapsulación no es "poner private y añadir accesores". Es que la clase sea la única responsable de su estado, y que exponga *operaciones*, no *datos*.**

```java
// ✗ La clase es un saco de datos: la lógica vive fuera
public class Carrito {
    private List<Linea> lineas;
    public List<Linea> getLineas() { return lineas; }
    public void setLineas(List<Linea> l) { this.lineas = l; }
}

// en el servicio, a saber en cuántos sitios:
carrito.getLineas().add(nuevaLinea);
double total = carrito.getLineas().stream().mapToDouble(Linea::importe).sum();
```

```java
// ✔ La clase protege su estado y expone operaciones del dominio
public class Carrito {
    private final List<Linea> lineas = new ArrayList<>();

    public void anadir(Producto p, int cantidad) {
        if (cantidad <= 0) throw new IllegalArgumentException("cantidad debe ser positiva");
        if (lineas.size() >= 100) throw new CarritoLlenoException();
        lineas.add(new Linea(p, cantidad));
    }

    public void quitar(Producto p) { lineas.removeIf(l -> l.producto().equals(p)); }

    public BigDecimal total() {
        return lineas.stream()
                     .map(Linea::importe)
                     .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public List<Linea> lineas() { return List.copyOf(lineas); }   // solo lectura
}
```

En la primera versión, la regla "máximo 100 líneas" hay que repetirla en cada sitio que añada una línea, y basta con que alguien la olvide. En la segunda **es imposible saltársela**, porque no hay otra puerta.

> **La pregunta que conviene hacerse:** si borro esta clase y dejo solo sus campos en un `Map`, ¿pierdo algo? Si la respuesta es "no", la clase no está encapsulando nada.

## 36. Copias defensivas

El agujero más frecuente en clases que se creen encapsuladas: exponer una referencia a un objeto mutable interno.

```java
public class Reserva {
    private final Date fecha;               // Date es MUTABLE
    private final List<String> invitados;

    public Reserva(Date fecha, List<String> invitados) {
        this.fecha = fecha;                 // ✗ guardamos la referencia del llamante
        this.invitados = invitados;         // ✗ ídem
    }

    public Date getFecha() { return fecha; }              // ✗ la regalamos
    public List<String> getInvitados() { return invitados; }  // ✗ ídem
}
```

**Dos ataques, ambos triviales:**

```java
// 1. Por la entrada
Date d = new Date();
Reserva r = new Reserva(d, new ArrayList<>());
d.setTime(0);                        // ¡cambia la fecha DE LA RESERVA!

// 2. Por la salida
r.getInvitados().clear();            // vacía la lista interna
r.getFecha().setTime(0);             // cambia la fecha
```

La clase es `final` en sus campos y aun así su estado cambia desde fuera. **La corrección es copiar en ambas fronteras:**

```java
public class Reserva {
    private final LocalDate fecha;               // ✔ mejor: tipo inmutable
    private final List<String> invitados;

    public Reserva(LocalDate fecha, List<String> invitados) {
        this.fecha = Objects.requireNonNull(fecha);
        this.invitados = List.copyOf(invitados);       // ✔ copia al ENTRAR
    }

    public LocalDate getFecha() { return fecha; }      // ✔ inmutable: se puede devolver
    public List<String> getInvitados() { return invitados; }   // ✔ ya es inmutable
}
```

**Tres reglas:**

1. **Copiá al entrar** todo objeto mutable que recibas y vayas a guardar.
2. **Copiá al salir**, o devolvé una vista inmutable, todo objeto mutable interno.
3. **Preferí tipos inmutables** y el problema desaparece: `LocalDate` en vez de `Date`, `List.of` en vez de `ArrayList`, `String` en vez de `StringBuilder`.

`List.copyOf` (Java 10+) hace las dos cosas: copia y devuelve una lista inmutable. `Collections.unmodifiableList` **no copia**: envuelve, así que si alguien conserva la lista original puede seguir modificándola y la "vista inmutable" reflejará los cambios.

## 37. Clases inmutables

Un objeto inmutable es aquel cuyo estado **no puede cambiar tras la construcción**. Es, con diferencia, la decisión de diseño más rentable de todo este capítulo.

**Las cinco reglas para construir una:**

1. Todos los campos `private final`.
2. Ningún método que modifique el estado (ningún setter).
3. La clase `final`, o los constructores privados con factorías — para que nadie herede y añada mutabilidad.
4. **Copia defensiva al entrar** de cualquier campo mutable.
5. **Copia defensiva al salir**, o exponer solo tipos inmutables.

```java
public final class Dinero {
    private final BigDecimal cantidad;
    private final String moneda;

    public Dinero(BigDecimal cantidad, String moneda) {
        this.cantidad = Objects.requireNonNull(cantidad);
        this.moneda = Objects.requireNonNull(moneda);
    }

    // Las "modificaciones" devuelven un objeto NUEVO
    public Dinero mas(Dinero otro) {
        if (!moneda.equals(otro.moneda)) {
            throw new IllegalArgumentException("Monedas distintas: " + moneda + " y " + otro.moneda);
        }
        return new Dinero(cantidad.add(otro.cantidad), moneda);
    }
}
```

**Lo que ganás:**

| Ventaja | Por qué |
|---|---|
| **Thread-safe gratis** | si nada cambia, no hay condiciones de carrera |
| Se puede cachear y compartir sin miedo | nadie puede corromperlo |
| Sirve como clave de `Map` sin sorpresas | su `hashCode` nunca cambia |
| Más fácil de razonar y depurar | el valor que viste al construirlo es el que hay |
| No necesita copias defensivas | no hay nada que proteger |

**Lo que pagás:** un objeto nuevo por cada "modificación". En la mayoría de aplicaciones es irrelevante; en un bucle muy caliente puede importar, y ahí existe el patrón *builder* mutable que produce un objeto inmutable al final (es lo que hacen `String` y `StringBuilder`).

Ejemplos de la JDK: `String`, `Integer` y todos los wrappers, `BigDecimal`, `BigInteger`, `LocalDate` y toda la API `java.time`, `List.of(...)`.

> **El consejo por defecto:** hacé inmutable toda clase que puedas, y mutable solo la que lo necesite de verdad. Es la recomendación de *Effective Java* (item 17) y la que más problemas evita a largo plazo.

---

# Parte VIII — Variedades de clase

## 38. Clase concreta, abstracta e interfaz

Baeldung dedica un artículo entero a definir "clase concreta", y la definición es simple: **una clase concreta es aquella de la que se puede crear una instancia con `new`**. Dicho de otro modo: *"todas las clases que no son abstractas son clases concretas"*.

```java
public class Coche {                        // CONCRETA: todos sus métodos tienen cuerpo
    public String pitar() { return "beep"; }
    public String conducir() { return "vroom"; }
}
Coche c = new Coche();                      // ✔
```

```java
public abstract class Vehiculo {            // ABSTRACTA: tiene un método sin cuerpo
    public abstract String pitar();
    public String conducir() { return "zoom"; }   // puede tener métodos implementados
}
Vehiculo v = new Vehiculo();                // ✗ no compila
```

```java
public interface Conducible {               // INTERFAZ: contrato
    void pitar();
    void conducir();
}
```

| | Clase concreta | Clase abstracta | Interfaz |
|---|---|---|---|
| `new` directo | ✔ | ✗ | ✗ |
| Métodos sin cuerpo | ✗ | ✔ (puede) | ✔ (por defecto) |
| Métodos con cuerpo | ✔ | ✔ | ✔ (`default`, desde Java 8) |
| Campos de instancia | ✔ | ✔ | ✗ (solo `public static final`) |
| Constructor | ✔ | ✔ (lo llama la subclase) | ✗ |
| Cuántas se pueden heredar | 1 | 1 | **varias** |

Baeldung añade un matiz que conviene retener: **una clase es concreta si ninguno de sus métodos queda sin implementar, vengan las implementaciones de donde vengan**. Una clase que extiende una abstracta e implementa el método que faltaba es concreta, aunque herede el resto:

```java
public class CocheDeLujo extends Vehiculo implements Conducible {
    public String pitar() { return "beep"; }      // implementa lo que faltaba
    // conducir() se hereda de Vehiculo
}
CocheDeLujo c = new CocheDeLujo();                // ✔ es concreta
```

Ejemplos de la JDK: concretas `HashMap`, `ArrayList`, `LinkedList`; abstractas `AbstractMap`, `AbstractList`; interfaces `Map`, `List`, `Set`.

Todo esto se desarrolla en los capítulos de interfaces y clases abstractas de este mismo bloque. Aquí basta con la distinción.

## 39. Clases anidadas: las cuatro formas

Jenkov las menciona como uno de los bloques de construcción de una clase. Hay **cuatro** tipos y la diferencia entre los dos primeros es la que más problemas causa en producción:

```java
public class Externa {
    private int campo = 1;

    static class Estatica { }                  // 1. clase anidada estática
    class Interna { }                          // 2. clase interna (no estática)

    void metodo() {
        class Local { }                        // 3. clase local
        Runnable r = new Runnable() {          // 4. clase anónima
            public void run() { }
        };
    }
}
```

**1 vs 2 — la diferencia crítica.** Una clase **interna** (no estática) mantiene una **referencia implícita al objeto que la contiene**. Una **estática** no.

```java
Externa.Estatica e = new Externa.Estatica();          // no necesita una Externa

Externa ext = new Externa();
Externa.Interna i = ext.new Interna();                 // necesita una instancia de Externa
```

La consecuencia práctica es una fuga de memoria clásica: si guardás una clase interna en algún sitio de larga vida (una caché, un listener estático, una colección), **estás manteniendo vivo también el objeto externo entero**, aunque no lo uses. Con objetos grandes, el impacto es considerable.

> **La regla:** declará `static` toda clase anidada, salvo que necesites de verdad acceder al estado de la instancia externa. Es exactamente lo que dice *Effective Java* (item 24), y es el fallo que más veces marca un analizador estático en código real.

**3 y 4 — locales y anónimas.** Las anónimas eran omnipresentes antes de Java 8 para listeners y comparadores; hoy una lambda las sustituye en casi todos los casos donde la interfaz tiene un solo método. Siguen siendo útiles cuando hace falta implementar varios métodos o mantener estado.

**Cuándo usar una clase anidada:** cuando el tipo solo tiene sentido dentro de su contenedor y no aporta nada como clase de nivel superior — un `Node` dentro de una `LinkedList`, un `Entry` dentro de un `Map`, un `Builder` dentro de la clase que construye.

## 40. Records: la clase de datos que escribe el compilador

Desde **Java 16** existe una forma de declarar clases cuyo único propósito es transportar datos inmutables:

```java
public record Punto(int x, int y) { }
```

Esa línea genera automáticamente:

- dos campos `private final`,
- un constructor con los dos parámetros (el *constructor canónico*),
- dos métodos de acceso, `x()` e `y()` (sin el prefijo `get`),
- `equals` y `hashCode` correctos y coherentes entre sí,
- un `toString` legible: `Punto[x=1, y=2]`.

El equivalente escrito a mano son unas 40 líneas, y es donde se cuela el error de la sección 32 (`equals` sin `hashCode`).

**Se pueden validar y añadir métodos:**

```java
public record Punto(int x, int y) {
    public Punto {                                    // constructor compacto
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordenadas no negativas");
        }
    }

    public double distanciaAlOrigen() {
        return Math.hypot(x, y);
    }

    public static Punto origen() { return new Punto(0, 0); }
}
```

**Limitaciones que hay que conocer:** un record es implícitamente `final`, no puede extender otra clase (sí implementar interfaces), y todos sus campos son `final`. Además, **la inmutabilidad de un record es superficial**: si un componente es una `List` mutable, el record no la copia por vos.

```java
public record Equipo(String nombre, List<String> miembros) {
    public Equipo {
        miembros = List.copyOf(miembros);      // ✔ copia defensiva en el constructor compacto
    }
}
```

**Cuándo usar un record:** DTOs, respuestas de API, claves compuestas de un `Map`, valores de retorno múltiples, eventos. **Cuándo no:** cuando la clase tiene comportamiento real, identidad propia (una entidad JPA) o estado que debe cambiar.

## 41. Clases de utilidad

Una clase que solo agrupa métodos estáticos y **no está pensada para instanciarse**: `Math`, `Collections`, `Arrays`, `Objects`.

```java
public final class ValidadorEmail {          // 1. final: nadie la extiende

    private ValidadorEmail() {               // 2. constructor privado
        throw new AssertionError("Clase de utilidad, no instanciable");   // 3. red de seguridad
    }

    public static boolean esValido(String email) {
        return email != null && email.contains("@");
    }
}
```

Los tres detalles importan: sin el constructor privado, el compilador genera uno público por defecto y la clase se puede instanciar sin sentido; el `throw` protege incluso frente a la reflexión y frente a un `new` escrito dentro de la propia clase.

**Aviso de diseño:** una clase de utilidad es la forma correcta para funciones puras y sin estado. Pero si empezás a acumular ahí lógica de negocio, estás volviendo a la programación procedural que la sección 1 quería evitar. La señal de alarma es un `Utils` o `Helper` de 500 líneas que recibe siempre los mismos objetos: eso es comportamiento que pertenece a esos objetos.

## 42. Qué ocupa un objeto en memoria

Un objeto no ocupa solo la suma de sus campos. En HotSpot de 64 bits con *compressed oops* (lo habitual):

```
Cabecera del objeto:   12 bytes  (mark word 8 + class pointer 4)
Campos:                 lo que sumen
Padding:                hasta múltiplo de 8
```

```java
class Vacia { }                    // 16 bytes (12 de cabecera + 4 de padding)
class ConInt { int x; }            // 16 bytes (12 + 4, justo)
class ConLong { long x; }          // 24 bytes (12 + 8 + 4 de padding)
```

Un `Integer` ocupa **16 bytes** frente a los 4 de un `int`, más 4-8 de la referencia que lo apunta: unas cinco veces más. Es la razón técnica de la división primitivo/objeto que vimos en [Data Types and Variables](../01-Basics/03-data-types-and-variables.md), y de que existan `IntStream` y `int[]` separados de `Stream<Integer>` y `List<Integer>`.

**Consecuencia práctica:** crear millones de objetos pequeños tiene un coste real, no tanto por la memoria como por la presión sobre el recolector de basura. Pero **no es una excusa para diseñar mal**: el JIT hace *escape analysis* y puede evitar la creación de objetos que no salen del método. Diseñá con clases y medí antes de optimizar.

---

# Parte IX — Diseño de clases

## 43. Una clase, una responsabilidad

El criterio más útil para decidir si una clase está bien planteada:

> Una clase debe tener **una sola razón para cambiar**.

```java
// ✗ tres razones para cambiar: el cálculo, el formato del informe y el esquema de la BD
public class Empleado {
    public double calcularNomina() { }
    public String generarInformePDF() { }
    public void guardarEnBaseDeDatos() { }
}
```

Si cambia la fórmula de la nómina, tocás `Empleado`. Si cambia el diseño del PDF, tocás `Empleado`. Si migrás de MySQL a Postgres, tocás `Empleado`. Tres equipos distintos modificando la misma clase por motivos que no tienen nada que ver.

```java
// ✔ cada clase cambia por un solo motivo
public class Empleado { }                      // el dato y sus reglas
public class CalculadoraNomina { }             // el cálculo
public class GeneradorInformePDF { }           // la presentación
public class RepositorioEmpleados { }          // la persistencia
```

**El test práctico:** describí lo que hace la clase en una frase. Si necesitás un "y", probablemente sean dos clases. `"Calcula la nómina de un empleado"` está bien. `"Calcula la nómina y genera el PDF y lo guarda"` no.

## 44. Cohesión y acoplamiento

Dos métricas informales pero muy útiles para leer un diseño.

**Cohesión alta = bien.** Los miembros de la clase trabajan sobre lo mismo. Señal de baja cohesión: grupos de métodos que usan grupos disjuntos de campos. Si los métodos 1-5 solo tocan los campos A y B, y los métodos 6-10 solo tocan C y D, ahí hay dos clases esperando a que las separes.

**Acoplamiento bajo = bien.** La clase depende de pocas cosas, y de abstracciones antes que de implementaciones.

```java
// ✗ acoplada a una implementación concreta
public class ServicioPedidos {
    private final MySqlRepositorioPedidos repo = new MySqlRepositorioPedidos();
}

// ✔ depende de una abstracción, y la recibe desde fuera
public class ServicioPedidos {
    private final RepositorioPedidos repo;

    public ServicioPedidos(RepositorioPedidos repo) {
        this.repo = Objects.requireNonNull(repo);
    }
}
```

La segunda versión se puede probar con un doble de test, se puede cambiar de base de datos sin tocarla, y **declara sus dependencias en el constructor**: leyendo la firma sabés exactamente qué necesita para funcionar. Esa es la idea detrás de la inyección de dependencias, y no requiere ningún framework: es un constructor.

## 45. El modelo anémico

Un anti-patrón tan extendido que mucha gente lo considera la forma normal de programar en Java:

```java
// ✗ clase anémica: solo datos, cero comportamiento
public class Pedido {
    private List<Linea> lineas;
    private String estado;
    private BigDecimal total;
    // 20 getters y 20 setters
}

// y toda la lógica en un servicio
public class ServicioPedidos {
    public void confirmar(Pedido p) {
        if (p.getEstado().equals("BORRADOR") && !p.getLineas().isEmpty()) {
            p.setEstado("CONFIRMADO");
            p.setTotal(calcularTotal(p));
        }
    }
}
```

**Por qué es un problema:** la regla "solo se puede confirmar un borrador con líneas" vive **fuera** del objeto que debería garantizarla. Cualquiera puede hacer `pedido.setEstado("CONFIRMADO")` y saltarse la validación. Y cuando aparezca un segundo servicio que también confirme pedidos, la regla se duplicará o divergirá.

```java
// ✔ el objeto protege sus propias reglas
public class Pedido {
    private final List<Linea> lineas = new ArrayList<>();
    private Estado estado = Estado.BORRADOR;

    public void confirmar() {
        if (estado != Estado.BORRADOR) {
            throw new IllegalStateException("Solo se confirma un borrador, estado actual: " + estado);
        }
        if (lineas.isEmpty()) {
            throw new IllegalStateException("No se puede confirmar un pedido sin líneas");
        }
        estado = Estado.CONFIRMADO;
    }
}
```

Ahora **no existe forma** de confirmar un pedido inválido, porque no hay setter de estado. La regla vive en un solo sitio y es imposible saltársela.

**Matiz honesto:** el modelo anémico tiene usos legítimos. Un DTO que solo transporta datos entre capas, o una entidad JPA que un framework rellena por reflexión, son casos donde la ausencia de comportamiento es correcta — y para el primero, hoy lo idiomático es un `record`. El problema no es que existan clases sin comportamiento: es que **todas** las clases del dominio lo sean.

## 46. La god class

La clase que lo sabe todo y lo hace todo. Se reconoce sin leerla:

- Más de 500-800 líneas.
- Nombre vago: `Manager`, `Processor`, `Handler`, `Utils`, `Helper`, `Service` a secas.
- Decenas de campos, muchos usados por un solo método.
- Todo el equipo la modifica, y siempre hay conflictos de merge en ella.

**Cómo se llega:** nadie la diseña así. Empieza con 100 líneas razonables, y cada vez que hace falta algo nuevo "es más rápido" añadirlo ahí que crear una clase. Un año después son 3.000 líneas que nadie entiende entera.

**Cómo se sale:** identificá grupos de campos que se usan juntos y extraé cada grupo con sus métodos a una clase propia. Un IDE moderno hace ese refactor de forma segura. Lo importante es hacerlo **antes** de que la clase sea intocable.

## 47. Cuándo NO crear una clase

El error opuesto también existe, y en equipos que acaban de descubrir el diseño limpio es igual de frecuente.

**No crees una clase cuando:**

| Situación | Alternativa |
|---|---|
| Solo envuelve un valor sin añadir reglas | usá el tipo directamente |
| Es un DTO de tres campos sin comportamiento | un `record` |
| Solo agrupa funciones puras | métodos estáticos en una clase de utilidad |
| Es un conjunto fijo de opciones | un `enum` |
| "Por si acaso hace falta después" | YAGNI: creala cuando haga falta |
| Una interfaz con una sola implementación y sin planes de más | la clase directamente |

Ese último punto merece énfasis: crear `IServicio` + `ServicioImpl` por sistema, sin que exista ni vaya a existir una segunda implementación, no aporta nada. Duplica el número de archivos, obliga a saltar dos veces para leer el código, y la supuesta "flexibilidad futura" casi nunca se usa. Extraé la interfaz **cuando aparezca la segunda implementación** o cuando la necesites para un test: el IDE lo hace en dos clics.

**Y el caso contrario, donde una clase sí paga:** cuando un tipo primitivo esconde reglas. Un `String email` que se valida en catorce sitios distintos pide a gritos ser una clase `Email` que se valide una vez en su constructor y que, a partir de ahí, no pueda ser inválida. Es lo que se llama *tipos fuertes* o *value objects*, y elimina categorías enteras de bugs.

---

# Parte X — Cierre

## 48. Casos de uso reales

### 48.1. Value object: un tipo que no puede ser inválido

El patrón que más bugs elimina por línea escrita. En vez de pasear un `String` que *debería* ser un email:

```java
public final class Email {
    private static final Pattern PATRON = Pattern.compile("^[^@\\s]+@[^@\\s]+\\.[^@\\s]+$");

    private final String valor;

    private Email(String valor) { this.valor = valor; }

    public static Email de(String valor) {
        Objects.requireNonNull(valor, "email no puede ser null");
        String limpio = valor.strip().toLowerCase(Locale.ROOT);
        if (!PATRON.matcher(limpio).matches()) {
            throw new IllegalArgumentException("Email inválido: " + valor);
        }
        return new Email(limpio);
    }

    public String valor() { return valor; }

    @Override public boolean equals(Object o) {
        return o instanceof Email e && valor.equals(e.valor);
    }
    @Override public int hashCode() { return valor.hashCode(); }
    @Override public String toString() { return valor; }
}
```

A partir de aquí, un método que recibe un `Email` **no necesita validarlo**: si existe la instancia, es válido. La validación y la normalización (recortar espacios, pasar a minúsculas) ocurren en un solo sitio.

### 48.2. Builder: cuando hay muchos parámetros opcionales

La salida al constructor telescópico de la sección 17:

```java
public final class Pizza {
    private final int tamano;
    private final boolean queso;
    private final boolean pepperoni;

    private Pizza(Builder b) {
        this.tamano = b.tamano;
        this.queso = b.queso;
        this.pepperoni = b.pepperoni;
    }

    public static Builder builder(int tamano) { return new Builder(tamano); }

    public static final class Builder {
        private final int tamano;              // obligatorio
        private boolean queso;                 // opcionales, con su valor por defecto
        private boolean pepperoni;

        private Builder(int tamano) { this.tamano = tamano; }

        public Builder conQueso()     { this.queso = true; return this; }
        public Builder conPepperoni() { this.pepperoni = true; return this; }

        public Pizza build() {
            if (tamano < 20 || tamano > 50) {
                throw new IllegalArgumentException("Tamaño fuera de rango: " + tamano);
            }
            return new Pizza(this);
        }
    }
}
```

```java
Pizza p = Pizza.builder(30).conQueso().conPepperoni().build();
```

Compará con `new Pizza(30, true, true, false, false)`. El builder se lee solo, los parámetros opcionales no obligan a escribir `false` repetido, y el objeto resultante es inmutable. El coste es escribir el builder — que un IDE genera, y que en un `record` con pocos campos no compensa.

### 48.3. Entidad frente a value object

Una distinción de modelado que separa un diseño junior de uno senior:

| | Entidad | Value object |
|---|---|---|
| Se identifica por | un **id** propio | **sus valores** |
| Dos con los mismos datos | son objetos **distintos** | son **el mismo** valor |
| `equals` compara | solo el id | todos los campos |
| Suele ser | mutable | **inmutable** |
| Ejemplos | `Usuario`, `Pedido`, `Cuenta` | `Email`, `Dinero`, `LocalDate`, `Direccion` |

Dos usuarios llamados "Ana García" son personas distintas: la entidad `Usuario` compara por id. Dos importes de 10 EUR son el mismo valor: `Dinero` compara por cantidad y moneda.

Confundirlos produce bugs concretos: si `Usuario.equals` compara todos los campos, cambiar el teléfono de un usuario lo convierte en "otro usuario" para cualquier `Set` donde estuviera.

### 48.4. Estado con máquina de estados

Cuando un objeto tiene un ciclo de vida, las transiciones válidas son parte de su contrato:

```java
public class Pedido {
    public enum Estado { BORRADOR, CONFIRMADO, ENVIADO, ENTREGADO, CANCELADO }

    private Estado estado = Estado.BORRADOR;

    public void confirmar() { transicionar(Estado.BORRADOR, Estado.CONFIRMADO); }
    public void enviar()    { transicionar(Estado.CONFIRMADO, Estado.ENVIADO); }

    private void transicionar(Estado esperado, Estado nuevo) {
        if (estado != esperado) {
            throw new IllegalStateException(
                "No se puede pasar a " + nuevo + " desde " + estado + "; se esperaba " + esperado);
        }
        estado = nuevo;
    }
}
```

Sin setter público de estado, las transiciones imposibles simplemente **no se pueden escribir**. El mensaje de error, además, dice exactamente qué pasó.

## 49. Anti-patrones

**1. Campos públicos.**
```java
public String nombre;                  // MAL
private String nombre;                 // bien
```

**2. Getter y setter para cada campo, por sistema.** No es encapsular: es exponer los campos con más ceremonia. Escribí solo los accesores que alguien necesite.

**3. Devolver la colección interna sin copiar.**
```java
public List<Linea> getLineas() { return lineas; }              // MAL
public List<Linea> getLineas() { return List.copyOf(lineas); } // bien
```

**4. Guardar sin copiar un objeto mutable recibido.** El llamante conserva la referencia y puede cambiarlo después.

**5. Confundir `final` con inmutable.** `final List` sigue admitiendo `add` y `clear`.

**6. Llamar a un método sobrescribible desde el constructor.** Se ejecuta sobre un objeto a medio construir y ve los campos a cero. Solo `private`, `static` o `final`.

**7. Dejar escapar `this` en el constructor.** Directamente, con una clase interna, o arrancando un hilo.

**8. `equals` sin `hashCode`.** El objeto desaparece de cualquier `HashSet` o `HashMap`.

**9. Escribir `public boolean equals(MiClase c)`.** Es una sobrecarga, no una sobrescritura. Poné `@Override` siempre.

**10. Usar campos mutables en `equals`/`hashCode`** de objetos que viven en colecciones basadas en hash.

**11. Escribir un método con el nombre de la clase y tipo de retorno.** No es un constructor: es un método normal que nunca se ejecuta.

**12. Asignar el parámetro a sí mismo.** `nombre = nombre;` compila y no hace nada. Es `this.nombre = nombre;`.

**13. Estado mutable en `static`.** Estado global: rompe los tests, la concurrencia y actúa como GC root.

**14. Acceder a un miembro estático a través de una instancia.** `objeto.metodoEstatico()` funciona incluso con `objeto` a `null`, y engaña al lector.

**15. Clases anidadas no estáticas por defecto.** Mantienen viva la instancia externa. Declaralas `static` salvo que necesites lo contrario.

**16. El idioma de la doble llave** (`new ArrayList<>() {{ add("a"); }}`). Genera una subclase anónima y una referencia implícita al contenedor.

**17. Adquirir recursos antes de validar en el constructor.** Si la validación falla, el recurso queda abierto y nadie puede cerrarlo.

**18. Modelo anémico en todo el dominio.** Clases sin comportamiento y toda la lógica en servicios: las reglas se duplican y se pueden saltar.

**19. God class.** Nombre vago, cientos de líneas, todo el equipo tocándola.

**20. Interfaz con una sola implementación "por si acaso".** Extraela cuando aparezca la segunda.

**21. Constructor telescópico.** Cinco booleanos en la llamada y nadie sabe cuál es cuál.

**22. Clase de utilidad sin constructor privado.** Se puede instanciar sin sentido, porque el compilador genera el constructor público.

**23. `toString` que expone datos sensibles.** Contraseñas y tokens acaban en los logs.

**24. Usar `clone()`.** Superficial, sin constructor, con contrato ambiguo. Constructor de copia.

## 50. Checklist y tabla de decisión

**Antes de dar por buena una clase:**

- [ ] ¿El nombre es un sustantivo concreto, sin `Manager`, `Helper` ni `Utils` genéricos?
- [ ] ¿Se describe en una frase sin usar "y"?
- [ ] ¿Todos los campos son `private`?
- [ ] ¿Todos los campos que no cambian son `final`?
- [ ] ¿Hay algún setter que no necesite existir?
- [ ] ¿El constructor deja el objeto en un estado **siempre válido**?
- [ ] ¿Se valida todo lo que entra por el constructor, con la excepción adecuada?
- [ ] ¿Se valida **antes** de adquirir cualquier recurso?
- [ ] ¿Hay un único constructor que haga el trabajo real y los demás delegan con `this(...)`?
- [ ] ¿El constructor llama solo a métodos `private`, `static` o `final`?
- [ ] ¿`this` se queda dentro del constructor?
- [ ] ¿Se copian los objetos mutables al entrar y al salir?
- [ ] Si sobrescribe `equals`, ¿sobrescribe también `hashCode`? ¿Y lleva `@Override`?
- [ ] ¿`toString` es útil y no filtra datos sensibles?
- [ ] ¿Las clases anidadas son `static` salvo que necesiten la instancia externa?
- [ ] ¿Podría ser un `record`? ¿Podría ser inmutable?
- [ ] ¿Cabe en la pantalla, o al menos en unos cientos de líneas?

**Qué construcción usar:**

| Si necesitás… | Usá |
|---|---|
| Un tipo con datos **y** reglas | una clase normal, con campos `private final` |
| Transportar datos inmutables | un **`record`** |
| Un conjunto fijo de opciones | un **`enum`** |
| Agrupar funciones puras sin estado | clase `final` con constructor privado y métodos `static` |
| Varias formas de construir con la misma firma | **factorías estáticas** con nombre |
| Muchos parámetros opcionales | un **builder** |
| Muchos parámetros obligatorios que viajan juntos | un `record` de parámetros |
| Un tipo que no puede ser inválido | un **value object** con constructor privado y factoría validadora |
| Impedir que se instancie | constructor `private` |
| Impedir que se extienda | clase `final` |
| Un tipo que solo tiene sentido dentro de otro | clase anidada **`static`** |
| Estado compartido por todas las instancias | `static final` — y si es mutable, replanteá el diseño |
| Un objeto con ciclo de vida | un `enum` de estados y métodos de transición |

**Qué campo o miembro elegir:**

| Necesito | Elección |
|---|---|
| Dato propio de cada objeto | campo de instancia `private` |
| Dato compartido e inmutable | `private static final` |
| Operación que depende del objeto | método de instancia |
| Operación que solo depende de sus argumentos | método `static` |
| Exponer un dato para lectura | getter |
| Permitir un cambio con reglas | método con nombre del dominio, **no** un setter |
| Prohibir un cambio | campo `final`, sin setter |

## 51. Autoevaluación

Respondé sin ejecutar; las respuestas están abajo.

1. ¿Cuántos objetos y cuántas referencias hay tras `Coche a = new Coche(); Coche b = a; Coche c = new Coche();`?
2. ¿Toda clase Java tiene un constructor sin argumentos?
3. ¿Qué imprime este código y por qué?
   ```java
   class Super { Super() { imprimir(); } void imprimir() { } }
   class Sub extends Super { int x = 5; @Override void imprimir() { System.out.println(x); } }
   new Sub();
   ```
4. ¿Cuál es el orden exacto de inicialización al hacer `new` sobre una subclase?
5. ¿Qué diferencia hay entre un *default constructor* y un *no-arg constructor*?
6. `private final List<String> lista = new ArrayList<>();` — ¿puede alguien de fuera vaciar esa lista?
7. ¿Por qué `objetoNulo.metodoEstatico()` no lanza `NullPointerException`?
8. Si sobrescribo `equals` pero no `hashCode`, ¿qué pasa exactamente al meter el objeto en un `HashSet`?
9. ¿Por qué `public boolean equals(Punto p)` es un bug?
10. ¿Qué diferencia hay entre una clase anidada `static` y una clase interna?
11. ¿Qué genera exactamente `public record Punto(int x, int y) { }`?
12. ¿Por qué no se debe llamar a un método sobrescribible desde el constructor?
13. ¿Cuándo el compilador **no** añade el constructor por defecto?
14. ¿Qué imprime `System.out.println(new Object())` y por qué?

<details>
<summary><strong>Respuestas</strong></summary>

1. **Dos objetos y tres referencias.** `a` y `b` apuntan al mismo objeto; `c` a otro distinto.
2. **No.** Solo si la clase **no declara ningún constructor**. En cuanto declarás uno con parámetros, el sin argumentos desaparece y `new MiClase()` deja de compilar (JLS §8.8.9, verificado con `javap`).
3. **Imprime `0`.** El constructor de `Super` se ejecuta antes de los inicializadores de campos de `Sub`, pero la llamada es polimórfica y va a `Sub.imprimir()`, que ve `x` con su valor por defecto.
4. Superclase completa → memoria a valores por defecto → `super()` → inicializadores de campos y bloques de instancia en orden textual → cuerpo del constructor.
5. El **default constructor** lo genera el compilador cuando no declarás ninguno; un **no-arg constructor** lo escribís vos. El resultado se parece, pero solo el primero es "por defecto".
6. **Sí, si existe un getter que la devuelva tal cual.** `final` congela la referencia, no el contenido. Hace falta `List.copyOf` o una vista inmutable.
7. Porque la llamada se resuelve en compilación contra la **clase**, no contra la referencia: esta ni se consulta. Es legal, engañoso, y por eso los estáticos se llaman por el nombre de la clase.
8. `HashSet` calcula el `hashCode` para elegir la cubeta. Como los hashes difieren (heredados de la identidad), busca en la cubeta equivocada y **nunca llega a llamar a `equals`**: `contains` devuelve `false` aunque el objeto esté dentro.
9. Porque la firma de `Object` es `equals(Object)`. Con otro parámetro estás **sobrecargando**, no sobrescribiendo: las colecciones siguen usando el `equals` de `Object`. `@Override` lo convierte en error de compilación.
10. La **interna** (no estática) mantiene una referencia implícita al objeto externo, lo que la ata a él y puede provocar fugas de memoria. La **estática** no, y se instancia sin necesitar una instancia externa.
11. Dos campos `private final`, el constructor canónico, los accesores `x()` e `y()`, y `equals`, `hashCode` y `toString` coherentes.
12. Porque se ejecuta la versión de la subclase sobre un objeto cuyos campos todavía no se han inicializado: se leen valores por defecto. Solo `private`, `static` o `final` son seguros.
13. Cuando la clase **declara cualquier constructor**, sea con parámetros o sin ellos.
14. Algo como `java.lang.Object@1460a8c0`: el `toString` por defecto imprime el nombre de la clase y el `hashCode` en hexadecimal. Por eso conviene sobrescribirlo.

</details>

## 52. Fuentes

**Recursos aportados (páginas leídas íntegramente, siguiendo su navegación interna)**

- [Jenkov · Java Classes](https://jenkov.com/tutorials/java/classes.html) — bloques de construcción de una clase, definición, instanciación
- [Jenkov · Java Fields](https://jenkov.com/tutorials/java/fields.html) — sintaxis completa de declaración, estáticos frente a de instancia, `final`
- [Jenkov · Java Constructors](https://jenkov.com/tutorials/java/constructors.html) — declaración, sobrecarga, `this()`, `super()`, modificadores, excepciones
- [Programiz · Java Class and Objects](https://www.programiz.com/java-programming/class-objects) — analogía del plano, variables de instancia, miembros de la clase
- [W3Schools · Java OOP](https://www.w3schools.com/java/java_oop.asp) — procedural frente a orientado a objetos, DRY
- [W3Schools · Java Classes and Objects](https://www.w3schools.com/java/java_classes.asp) — crear clases y objetos, múltiples objetos, patrón de dos archivos
- [W3Schools · Java Class Attributes](https://www.w3schools.com/java/java_class_attributes.asp) — atributos, acceso, modificación, `final`
- [W3Schools · Java Class Methods](https://www.w3schools.com/java/java_class_methods.asp) — métodos estáticos y de instancia, acceso con el punto
- [W3Schools · Java Constructors](https://www.w3schools.com/java/java_constructors.asp) — constructor básico y con parámetros
- [W3Schools · Java Modifiers](https://www.w3schools.com/java/java_modifiers.asp) — tabla de modificadores de acceso y de no acceso
- [W3Schools · Java Encapsulation](https://www.w3schools.com/java/java_encapsulation.asp) — getters y setters, campos privados
- [Baeldung · Java Classes and Objects](https://www.baeldung.com/java-classes-objects) — clases en compilación frente a objetos en ejecución, control de acceso
- [Baeldung · Concrete Class in Java](https://www.baeldung.com/java-concrete-class) — definición de clase concreta frente a abstracta e interfaz

**Búsquedas propias: normativa, documentación oficial y foros**

- [JLS §8.8.9 — Default Constructor](https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html) — el texto normativo sobre cuándo se genera el constructor por defecto y qué modificador de acceso recibe
- JLS §12.4 y §12.5 — inicialización de clases y creación de instancias: la fuente definitiva del orden de la Parte V
- [Oracle · Initializing Fields](https://docs.oracle.com/javase/tutorial/java/javaOO/initial.html) — bloques de inicialización estáticos y de instancia, y la recomendación de usar métodos `final` para inicializar
- [Stack Overflow · Do I really need to define default constructor in Java?](https://stackoverflow.com/questions/3641114/do-i-really-need-to-define-default-constructor-in-java) — el malentendido más frecuente sobre el constructor por defecto, con la cita literal de la JLS
- [Stack Overflow · Why does the default parameterless constructor go away when you create one with parameters](https://stackoverflow.com/questions/11792207/why-does-the-default-parameterless-constructor-go-away-when-you-create-one-with) — el *porqué* del diseño, con su historia desde C++
- [Stack Overflow · Are fields initialized before constructor code is run in Java?](https://stackoverflow.com/questions/14805547/are-fields-initialized-before-constructor-code-is-run-in-java) — orden de inicialización con referencia a §12.4.2 y §12.5
- [Stack Overflow · Java: Calling superclass' constructor which calls overridden method](https://stackoverflow.com/questions/12100449/java-calling-superclass-constructor-which-calls-overridden-method-which-sets-a) — el bug de la sección 26, reproducido por varios usuarios
- [Stack Overflow · Why is it considered bad practice to call a method from within a constructor?](https://stackoverflow.com/questions/18348797/why-is-it-considered-bad-practice-in-java-to-call-a-method-from-within-a-const) — la explicación completa a partir de §12.5, y el problema de la fuga de `this`
- [Stack Overflow · Why are instance initialization blocks executed before constructors](https://stackoverflow.com/questions/31101110/why-are-instance-initialization-blocks-executed-before-constructors) — con el `javap` que muestra los bloques copiados dentro del constructor

> ### Dónde se equivocan las fuentes
>
> Señalarlo no es descortesía: es lo que separa leer un tutorial de entender el lenguaje. Todo lo siguiente se comprobó compilando y ejecutando en **JDK 25**.
>
> **1. Baeldung afirma que toda clase tiene un constructor vacío por defecto, y su propio ejemplo lo desmiente.** Escribe: *"Every Java class has an empty constructor by default […] Here's how the default constructor would look for our Car class: `Car(){}`"*. Pero esa clase `Car` **declara** un constructor de tres argumentos, así que el compilador **no** genera ninguno vacío. `new Car()` falla con `constructor Car in class Car cannot be applied to given types; required: String,String,String; found: no arguments`. La regla correcta es la de la JLS §8.8.9: el constructor por defecto aparece **solo si la clase no declara ningún constructor**.
>
> **2. Baeldung también dice que ese constructor "inicializa todos los campos con sus valores por defecto".** No hace tal cosa: los campos ya están a cero porque la JVM limpia la memoria al reservarla. Lo único que hace el constructor por defecto es invocar `super()`.
>
> **3. Programiz confunde instancia con subcategoría.** Escribe que si `Bicycle` es una clase, entonces *"`MountainBicycle`, `SportsBicycle`, `TouringBicycle` pueden considerarse objetos de la clase"*. No lo son: son **categorías**, que en Java serían subclases, un `enum` o un campo. Un objeto es *esta bicicleta concreta*. La confusión lleva a diseñar jerarquías de herencia donde bastaba un campo, que es un error de diseño caro. W3Schools arrastra el mismo problema con `Fruit` → `Apple`, `Banana`, `Mango`.
>
> **4. W3Schools dice que los objetos "heredan" las variables y métodos de su clase.** La herencia es una relación **entre clases**. Un objeto no hereda de su clase: *es* una instancia de ella. Usar ese verbo aquí choca con el significado que tendrá en el capítulo de herencia.
>
> **5. Jenkov escribe `public String brand = null;`.** Dos problemas en una línea: el campo es `public` (rompe la encapsulación que él mismo describe después) y el `= null` es redundante, porque ese ya es el valor por defecto de una referencia. Escribir el valor por defecto explícitamente añade ruido y, en un campo `final`, produce un error de compilación.
>
> **6. Ninguna de las seis fuentes aportadas menciona** el orden real de inicialización, el bug del método sobrescribible en el constructor, la fuga de `this`, el contrato `equals`/`hashCode`, las copias defensivas ni los `record`. Son exactamente los temas que separan a un junior de un mid, y proceden de la búsqueda propia y del conocimiento previo.

