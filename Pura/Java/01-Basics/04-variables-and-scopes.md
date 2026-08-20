# Variables and Scopes

> **Bloque:** `01-Basics` · **Nivel de entrada:** Junior · **Versión base:** Java 17+

**Alcance de este documento.** El tema anterior ([Data Types and Variables](03-data-types-and-variables.md)) respondía *qué* puede contener una variable: los tipos, sus rangos, el casting, los wrappers. Este responde una pregunta distinta y, en la práctica diaria, más determinante: **dónde vive una variable, desde cuándo, hasta cuándo y quién puede verla**.

Es un tema que parece trivial durante una semana y luego explica la mitad de los bugs raros que vas a encontrar: el campo que quedó en `null` porque el constructor se asignó a sí mismo, el `static` que provocó un `OutOfMemoryError` en producción, el lambda que no compila porque la variable "no es effectively final", el campo de la clase padre que no se comporta de forma polimórfica.

Vamos a cubrir el modelo mental completo: las reglas del lenguaje, cómo las implementa el compilador, los errores clásicos uno por uno, y los criterios de diseño que separan a alguien que *conoce* la sintaxis de alguien que *decide* bien.

---

## Fundamentos

### Qué es el scope, y por qué un lenguaje lo necesita

El **scope** (ámbito) de una variable es la región del código fuente donde ese nombre es válido y se refiere a esa variable concreta.

La definición suena burocrática, así que vale la pena entender el problema que resuelve. Imaginá un lenguaje sin scopes, donde toda variable es visible desde cualquier punto del programa. Un programa de 200 líneas funcionaría. Uno de 200.000 líneas sería imposible de mantener por tres razones:

1. **Colisiones de nombres.** Si todo es global, dos partes del programa escritas por personas distintas no pueden usar `contador` sin pisarse.
2. **Imposibilidad de razonar localmente.** Para saber qué vale una variable tendrías que leer el programa entero, porque cualquier línea pudo modificarla.
3. **Desperdicio de memoria.** Toda variable tendría que existir durante toda la ejecución.

El scope resuelve las tres cosas a la vez. Java, como casi todos los lenguajes derivados de C, usa **scope léxico** (o estático): el ámbito de una variable se determina leyendo la estructura del texto del programa, no observando la ejecución. Esto es importante y no es universal — algunos lenguajes históricos usaban *scope dinámico*, donde la visibilidad dependía de quién llamaba a quién en tiempo de ejecución. Que Java sea léxico significa que **podés determinar el scope de cualquier variable mirando el código, sin ejecutarlo**. El compilador lo hace por vos, y por eso los errores de scope son errores de compilación, no fallos en producción.

### La regla base: las llaves delimitan el scope

En Java el mecanismo es casi todo el tiempo el mismo. Un par de llaves `{ }` abre un bloque, y **una variable declarada dentro de un bloque existe desde la línea de su declaración hasta la llave de cierre de ese bloque**.

```java
public class Ejemplo {

    void metodo() {
        int a = 1;          // 'a' nace aquí

        {                   // bloque anónimo: es Java válido
            int b = 2;      // 'b' nace aquí
            System.out.println(a);  // OK: 'a' es visible, estamos dentro de su bloque
            System.out.println(b);  // OK
        }                   // 'b' muere aquí

        System.out.println(a);  // OK
        // System.out.println(b);  // ERROR de compilación: cannot find symbol
    }
}
```

Prestá atención a dos detalles que se pasan por alto:

**Primero: el scope empieza en la declaración, no en la llave de apertura.** Esto no es obvio:

```java
void metodo() {
    System.out.println(x);  // ERROR: cannot find symbol
    int x = 5;
}
```

Aunque `x` está declarada "dentro del mismo bloque", el compilador lee de arriba abajo y en esa línea el nombre `x` todavía no significa nada. Esta regla se llama *declaración antes de uso* y aplica a variables locales. Como veremos, **los campos de una clase no siguen esta regla**, y esa asimetría es fuente de confusión.

**Segundo: se puede abrir un bloque sin motivo sintáctico.** El bloque anónimo del ejemplo no está atado a ningún `if` ni `for`. Es Java legal y sirve para acotar deliberadamente la vida de una variable auxiliar. Se usa poco, pero conviene saber que la llave es el mecanismo, y que `if`, `for`, `while`, `try` y los métodos simplemente *usan* ese mecanismo.

### Scope y lifetime no son lo mismo

Esta distinción es la que más rápido separa a un junior de alguien con criterio, y casi ninguna fuente introductoria la explica bien.

- **Scope** es una propiedad del *código fuente*: en qué región el nombre es válido. Lo decide el compilador.
- **Lifetime** (tiempo de vida) es una propiedad de la *ejecución*: cuánto tiempo existe realmente el dato en memoria. Lo decide la JVM.

Normalmente coinciden, y por eso se confunden. Pero se separan en el caso más importante de todos:

```java
void metodo() {
    List<String> lista;
    {
        lista = new ArrayList<>();
        lista.add("hola");
    }   // aquí NO se destruye el ArrayList
    System.out.println(lista.get(0));  // "hola"
}
```

Y en el caso inverso, que es el que de verdad importa:

```java
List<String> crearLista() {
    List<String> local = new ArrayList<>();
    local.add("dato");
    return local;
}   // 'local' sale de scope aquí...
```

Cuando `crearLista()` termina, la **variable** `local` desaparece: su nombre ya no significa nada y su espacio en el stack se libera. Pero el **objeto** `ArrayList` que ella apuntaba sigue perfectamente vivo en el heap, porque quien llamó al método se quedó con la referencia.

La formulación correcta, y la que conviene memorizar:

> En Java, el scope controla las **variables**, no los **objetos**. Una variable local es una etiqueta temporal; el objeto vive mientras alguien lo referencie, y de eso se ocupa el garbage collector, no el scope.

Esto explica por qué en Java no existe el equivalente al *dangling pointer* de C: no podés devolver la dirección de una variable local y que apunte a basura, porque nunca devolvés variables, devolvés referencias a objetos del heap que el GC mantiene vivos mientras hagan falta.

### Los cuatro tipos de variable

Java tiene exactamente cuatro clases de variable, y cada una tiene reglas propias de scope, de lifetime, de dónde se almacena y de si recibe valor por defecto. Esta tabla es el esqueleto de todo el documento:

| Tipo | Dónde se declara | Scope | Lifetime | Memoria | ¿Valor por defecto? |
|---|---|---|---|---|---|
| **Variable local** | Dentro de un método, constructor o bloque | Desde la declaración hasta el cierre del bloque | Mientras el bloque está en ejecución | Stack (el objeto apuntado, en heap) | **No** |
| **Parámetro** | En la firma de un método o constructor | Todo el cuerpo del método | Durante la llamada | Stack | No aplica (llega con valor) |
| **Instance field** (campo de instancia) | En la clase, fuera de métodos, sin `static` | Toda la clase (salvo métodos `static`) | Mientras el objeto esté vivo | Heap, dentro del objeto | **Sí** |
| **Static field** (campo de clase) | En la clase, fuera de métodos, con `static` | Toda la clase | Desde que la clase se carga hasta que se descarga | Metaspace / área de clase | **Sí** |

A los campos de instancia y de clase se les llama en conjunto **fields** o *member variables*. La documentación oficial de Oracle usa exactamente esta terminología: *fields* para los miembros de la clase, *local variables* para las de un método, y *parameters* para las de la firma.

Los cuatro tipos merecen tratamiento propio.

---

### 1. Variables locales

Se declaran dentro de un método, un constructor o un bloque.

```java
public class Calculadora {
    public int sumar(int a, int b) {
        int resultado = a + b;   // variable local
        return resultado;
    }
}
```

Sus cuatro propiedades definitorias:

**No tienen valor por defecto.** Esta es la diferencia más importante respecto a los campos, y la que produce el primer error de compilación de todo principiante:

```java
public void edadCachorro() {
    int edad;
    edad = edad + 7;   // ERROR: variable edad might not have been initialized
    System.out.println("Edad: " + edad);
}
```

El compilador rechaza el programa. No asume cero. Esto es una decisión de diseño deliberada: leer una variable local no inicializada casi siempre es un bug, así que Java prefiere fallar en compilación antes que darte un valor arbitrario. El mecanismo formal que hace esta comprobación se llama **definite assignment**, y lo vemos en la sección avanzada.

**No admiten modificadores de acceso.** Escribir `private int x;` dentro de un método es un error de sintaxis. Y tiene sentido: `private`, `public` y `protected` responden a "¿qué otras clases pueden ver esto?", y una variable local no es visible ni siquiera para otro método de su propia clase. El único modificador que acepta es `final`.

**Viven en el stack.** Cada llamada a un método crea un *stack frame* con espacio para sus variables locales. Cuando el método retorna, el frame se descarta entero. Por eso su lifetime es tan corto y su coste, prácticamente nulo.

**Son inherentemente thread-safe.** Consecuencia directa de lo anterior y muy poco apreciada por juniors: cada hilo tiene su propio stack, así que **dos hilos que ejecutan el mismo método tienen cada uno su propia copia de las variables locales**. Nunca hay que sincronizar el acceso a una variable local. Es la razón por la que "guardá el estado en variables locales, no en campos" es un consejo tan repetido en código concurrente.

### 2. Parámetros

Un parámetro es una variable declarada en la firma de un método, que recibe su valor en el momento de la llamada.

```java
public void escribirTexto(String texto1, String texto2) {
    System.out.print(texto1);
    texto1 = "nuevo valor";   // legal, pero mala práctica
}
```

A efectos de scope, **un parámetro se comporta exactamente como una variable local** declarada al principio del método: es visible en todo el cuerpo, vive en el stack y desaparece al retornar. La única diferencia es que llega ya inicializado, así que la regla de definite assignment no aplica.

Reasignar un parámetro es legal pero se considera mala práctica: hace que el nombre signifique dos cosas distintas en distintos puntos del método y complica la depuración. Muchos equipos lo prohíben marcando los parámetros como `final`:

```java
public void escribirTexto(final String texto1, final String texto2) {
    System.out.print(texto1);
    // texto1 = "otro";   // ahora no compila
}
```

**Java pasa siempre los argumentos por valor.** Merece una aclaración porque es la pregunta de entrevista más malinterpretada del lenguaje. Cuando pasás un objeto, lo que se copia es la *referencia*, no el objeto. Consecuencia práctica:

```java
void modificar(StringBuilder sb, String s) {
    sb.append(" modificado");   // SÍ afecta al objeto original
    s = "reasignado";           // NO afecta a nada fuera del método
}

public static void main(String[] args) {
    StringBuilder sb = new StringBuilder("original");
    String s = "original";
    new Ejemplo().modificar(sb, s);
    System.out.println(sb);  // "original modificado"
    System.out.println(s);   // "original"
}
```

El parámetro `sb` es una copia de la referencia: apunta al mismo objeto, así que mutarlo se ve desde fuera. El parámetro `s` también es una copia de una referencia, pero reasignarla solo cambia la copia local. Java **nunca** pasa por referencia; pasa referencias por valor, que no es lo mismo.

### 3. Instance fields (campos de instancia)

Se declaran dentro de la clase pero fuera de cualquier método, constructor o bloque, y sin `static`.

```java
public class Empleado {
    public String nombre;     // visible para cualquier clase
    private double salario;   // visible solo dentro de Empleado

    public Empleado(String nombre) {
        this.nombre = nombre;
    }

    public void setSalario(double salario) {
        this.salario = salario;
    }
}
```

Propiedades:

- **Pertenecen al objeto.** Cada instancia creada con `new` tiene su propio juego de campos. Dos empleados tienen dos `salario` independientes.
- **Se almacenan en el heap**, dentro del objeto.
- **Nacen con el objeto y mueren con él**, es decir, cuando el garbage collector lo recolecta.
- **Sí tienen valor por defecto**: `0` para tipos numéricos, `false` para `boolean`, y `null` para cualquier referencia. Esto es lo contrario de las variables locales, y la asimetría es deliberada: la JVM garantiza que la memoria de un objeto recién creado esté limpia, mientras que forzar la inicialización explícita de locales atrapa bugs.
- **Admiten modificadores de acceso**: `private`, `package-private` (el valor por defecto, sin escribir nada), `protected` y `public`.
- **Su scope es toda la clase**, incluidos los métodos declarados *antes* de la propia declaración del campo. Esto rompe la regla de "declaración antes de uso" que rige para las locales:

```java
public class Ejemplo {
    void imprimir() {
        System.out.println(mensaje);   // OK, aunque 'mensaje' se declara más abajo
    }

    private String mensaje = "hola";
}
```

Esto compila. El compilador procesa la clase en dos fases: primero registra todos los miembros, después analiza los cuerpos de los métodos. Por eso el orden de declaración de los campos no importa para la visibilidad — aunque sí importa, y mucho, para el **orden de inicialización**, como veremos.

Nota sobre encapsulación: aunque `public` es legal, la práctica establecida es declarar los campos `private` y exponerlos mediante métodos. El propio tutorial de Oracle lo plantea así: los campos privados solo se acceden directamente desde la clase, y el acceso externo se hace indirectamente con getters y setters. La razón no es dogmática: un campo público es parte de tu API pública para siempre, y no podés añadirle validación, logging ni cálculo derivado sin romper a todos tus clientes.

### 4. Static fields (campos de clase)

Se declaran con `static`, fuera de cualquier método.

```java
public class Empleado {
    private static double salarioMedio;
    public static final String DEPARTAMENTO = "Desarrollo";

    public static void main(String[] args) {
        salarioMedio = 1000;
        System.out.println(DEPARTAMENTO + " salario medio: " + salarioMedio);
    }
}
```

Propiedades:

- **Pertenecen a la clase, no a los objetos.** Existe **una sola copia**, sin importar cuántas instancias crees. Si un objeto la modifica, todos los demás ven el cambio.
- **Se acceden con el nombre de la clase**: `Empleado.DEPARTAMENTO`. Es legal accederlos con una referencia de instancia (`empleado.DEPARTAMENTO`), pero es una práctica desaconsejada porque sugiere falsamente que el valor pertenece al objeto; muchos linters lo marcan como warning.
- **Nacen cuando la clase se carga** y viven mientras la clase permanezca cargada, que en la práctica suele ser toda la ejecución del programa.
- **Tienen los mismos valores por defecto** que los campos de instancia.

El uso abrumadoramente mayoritario y recomendado es como **constante**, combinando `static final`:

```java
public static final int MAX_REINTENTOS = 3;
```

La convención de nombres cambia en ese caso: las constantes `static final` se escriben en `UPPER_SNAKE_CASE`, mientras que el resto de variables usa `camelCase`.

Un `static` **mutable** es otra historia y merece sospecha. Es estado global compartido: rompe el aislamiento entre tests, no es thread-safe salvo que lo sincronices, y es una causa clásica de memory leaks. Lo tratamos en detalle en los errores.

### La relación entre `static` y el scope

Hay una regla de visibilidad asimétrica que confunde a todo el mundo al principio:

```java
public class Ejemplo {
    private int campoInstancia = 1;
    private static int campoEstatico = 2;

    public void metodoInstancia() {
        System.out.println(campoInstancia);  // OK
        System.out.println(campoEstatico);   // OK
    }

    public static void metodoEstatico() {
        // System.out.println(campoInstancia);  // ERROR
        System.out.println(campoEstatico);      // OK
    }
}
```

**Un método estático no puede acceder a miembros de instancia; un método de instancia sí puede acceder a miembros estáticos.**

La razón deja de ser arbitraria en cuanto se entiende: un método estático se invoca sobre la *clase* (`Ejemplo.metodoEstatico()`), y en ese momento puede que no exista **ningún** objeto. Preguntar por `campoInstancia` sería preguntar "¿el salario de qué empleado?" cuando no hay ningún empleado. Al revés no hay problema: si tenés un objeto, su clase está necesariamente cargada, así que los estáticos existen con seguridad.

Dicho en términos de scope: **los miembros de instancia están en el scope de un objeto concreto, identificado por la referencia implícita `this`**. Un método estático no tiene `this`, y por eso ese scope no está disponible. Es también la explicación del error más famoso del principiante: `non-static variable cannot be referenced from a static context`, que aparece al intentar usar un campo de instancia desde `main`, que es estático.

---

## Los ámbitos, uno por uno

Con los cuatro tipos claros, revisemos las regiones concretas donde el scope se manifiesta. La taxonomía sigue la que usa Baeldung, ampliada.

### Class scope

Todo lo declarado entre las llaves de la clase, fuera de métodos y bloques, tiene ámbito de clase: es accesible desde cualquier método de esa clase.

```java
public class Cachorro {
    private int edad;                          // instance field
    public static String RAZA = "Bulldog";     // static field

    public void setEdad(int edad) { this.edad = edad; }
    public int getEdad() { return edad; }
}
```

### Method scope

Todo lo declarado dentro de un método —incluidos sus parámetros— es visible solo dentro de ese método.

```java
public void procesar() {
    int contador = 0;
    contador++;
}
// 'contador' no existe fuera
```

Un método **no puede** ver las variables locales de otro método, aunque estén en la misma clase y aunque uno llame al otro. La única forma de compartir datos entre métodos es a través de campos, parámetros o valores de retorno. Esto no es una limitación: es lo que hace que un método sea una unidad razonable de código.

### Block scope

Cualquier par de llaves crea un ámbito. Los bloques de `if`, `else`, `while` y `try` funcionan así:

```java
public void ejemplo(boolean condicion) {
    if (condicion) {
        String mensaje = "dentro del if";
        System.out.println(mensaje);
    }
    // System.out.println(mensaje);  // ERROR: no existe aquí
}
```

Un caso que sorprende: un `if` **sin llaves** no crea bloque, y por tanto no podés declarar nada en él.

```java
if (condicion) int x = 5;   // ERROR de compilación
```

No es un problema de scope propiamente dicho, sino de gramática: la especificación solo permite declaraciones de variables locales dentro de un bloque, no como sentencia única de un `if`. Es otro argumento a favor de la regla de estilo de usar siempre llaves.

### Loop scope

Una variable declarada en la cabecera de un `for` vive solo dentro del bucle:

```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);   // OK
}
// System.out.println(i);   // ERROR: 'i' ya no existe
```

Esto tiene un beneficio práctico inmediato: podés reutilizar `i` en otro bucle del mismo método sin conflicto, porque cada bucle tiene su propio ámbito.

```java
for (int i = 0; i < 3; i++) { /* ... */ }
for (int i = 0; i < 5; i++) { /* ... */ }   // perfectamente legal
```

Y la contrapartida: si necesitás el valor del índice *después* del bucle, tenés que declararlo fuera.

```java
int i;
for (i = 0; i < 10; i++) {
    if (encontrado(i)) break;
}
System.out.println("Se paró en " + i);   // ahora sí es visible
```

El `for-each` sigue la misma regla, y con un matiz relevante que veremos al hablar de lambdas: **cada iteración crea conceptualmente una variable nueva**, no reutiliza la misma.

```java
for (String nombre : nombres) {
    System.out.println(nombre);
}
// 'nombre' no existe aquí
```

### Scope en `switch`: la trampa

El `switch` clásico (con `case ... :`) tiene una peculiaridad que rompe la intuición de todo el mundo: **todas las ramas comparten un único bloque, y por tanto un único scope**.

```java
switch (dia) {
    case 1:
        int x = 10;
        System.out.println(x);
        break;
    case 2:
        int x = 20;   // ERROR: variable x is already defined
        break;
}
```

Peor todavía, esta variante compila pero falla:

```java
switch (dia) {
    case 1:
        int x = 10;
        break;
    case 2:
        System.out.println(x);   // ERROR: variable x might not have been initialized
        break;
}
```

`x` **es visible** en `case 2` porque comparte el bloque del `switch`, pero el compilador no puede garantizar que se haya ejecutado la asignación, así que lo rechaza por definite assignment. Es un caso perfecto para ver que scope y asignación son comprobaciones independientes.

La solución clásica es abrir llaves explícitas por rama:

```java
switch (dia) {
    case 1: {
        int x = 10;
        System.out.println(x);
        break;
    }
    case 2: {
        int x = 20;   // ahora sí: son bloques distintos
        break;
    }
}
```

Y la solución moderna, desde Java 14, es usar la **flecha** (`->`), que crea un ámbito propio por rama automáticamente:

```java
switch (dia) {
    case 1 -> {
        int x = 10;
        System.out.println(x);
    }
    case 2 -> {
        int x = 20;   // sin conflicto
        System.out.println(x);
    }
}
```

Esta es una de las razones —además de eliminar el *fall-through* accidental— por las que el `switch` con flecha se considera hoy la forma preferida.

### Scope en `try-with-resources`

El recurso declarado en la cabecera del `try` está en scope solo dentro del bloque, y se cierra automáticamente al salir:

```java
try (BufferedReader br = new BufferedReader(new FileReader("datos.txt"))) {
    System.out.println(br.readLine());
}   // br.close() se llama aquí, automáticamente
// 'br' ya no existe
```

Es el ejemplo más limpio de scope y lifetime alineados a propósito: el lenguaje ata la liberación del recurso al final del ámbito. Desde Java 9 también podés usar una variable *effectively final* declarada antes, sin redeclararla dentro del paréntesis.

### Flow scoping: el scope que depende del flujo (Java 16+)

Este es el matiz más moderno del tema, y casi ningún tutorial lo cubre. Con el *pattern matching* de `instanceof` (JEP 394, definitivo en Java 16), la variable que se enlaza en el patrón tiene un ámbito que **no** se define por llaves, sino por análisis de flujo.

Antes:

```java
if (obj instanceof String) {
    String s = (String) obj;     // cast redundante
    System.out.println(s.length());
}
```

Desde Java 16:

```java
if (obj instanceof String s) {
    System.out.println(s.length());   // 's' ya está enlazada y tipada
}
```

Lo interesante es dónde está `s` en scope. El JEP lo llama **flow scoping**: *la variable del patrón está en ámbito exactamente allí donde el compilador puede demostrar que el patrón coincidió y la variable tiene valor asignado*. Eso produce comportamientos que parecen mágicos pero son perfectamente lógicos:

```java
// 's' está en scope en el lado derecho del && porque solo se evalúa si el match tuvo éxito
if (obj instanceof String s && s.length() > 5) { /* ... */ }

// 's' NO está en scope en el lado derecho del || : puede no haber coincidido
if (obj instanceof String s || s.length() > 5) { }   // ERROR de compilación
```

Y el caso más elegante, con negación y retorno temprano:

```java
public String describir(Object obj) {
    if (!(obj instanceof String s)) {
        return "no es un String";     // aquí 's' NO está en scope
    }
    // ...pero aquí SÍ, porque si llegamos a esta línea el patrón coincidió
    return "String de longitud " + s.length();
}
```

Este último ejemplo es el que hace que el pattern matching encaje tan bien con el estilo de *early return*. El scope se propaga siguiendo el razonamiento del compilador sobre el flujo de control, igual que ya hacía el análisis de definite assignment.

---

## En la práctica

### `this` y el shadowing intencionado

**Shadowing** (sombreado) ocurre cuando una variable de un ámbito interno tiene el mismo nombre que otra de un ámbito externo. La interna "tapa" a la externa: dentro de ese ámbito, el nombre se refiere a la interna.

El caso más habitual —y el único donde el shadowing es deliberado y correcto— es el constructor:

```java
public class Empleado {
    private String nombre;

    public Empleado(String nombre) {
        this.nombre = nombre;
    }
}
```

Aquí el parámetro `nombre` sombrea al campo `nombre`. Dentro del constructor, escribir `nombre` a secas significa *el parámetro*. Para llegar al campo hace falta la referencia explícita `this`.

`this` es una referencia implícita al objeto sobre el que se está ejecutando el método. `this.nombre` significa literalmente "el campo `nombre` de este objeto", y desambigua sin ninguna duda.

El equivalente para campos estáticos es cualificar con el nombre de la clase:

```java
public class Config {
    private static String entorno;

    public static void setEntorno(String entorno) {
        Config.entorno = entorno;   // 'this' no existe en contexto estático
    }
}
```

Por qué se acepta esta práctica en constructores y setters, si el shadowing en general es indeseable: porque el nombre del parámetro *debe* describir el dato, y el mejor nombre para el dato ya lo tiene el campo. Alternativas como `nombreParam` o `pNombre` son ruido. El consenso del ecosistema Java es usar el mismo nombre y `this`.

### Shadowing accidental vs. variable hiding

Conviene distinguir dos fenómenos que suenan parecido y se confunden constantemente, incluso en artículos técnicos.

**Shadowing** es entre ámbitos anidados dentro del mismo contexto: un parámetro o una local tapa a un campo.

**Hiding** (ocultación) es entre clases en una jerarquía de herencia: un campo de la subclase con el mismo nombre que uno de la superclase. Y tiene una consecuencia que sorprende a mucha gente:

```java
class Padre {
    String nombre = "padre";
    String getNombre() { return nombre; }
}

class Hijo extends Padre {
    String nombre = "hijo";     // oculta (no sobrescribe) el campo del padre
    String getNombre() { return nombre; }
}

public class Test {
    public static void main(String[] args) {
        Padre p = new Hijo();
        System.out.println(p.nombre);       // "padre"  (!)
        System.out.println(p.getNombre());  // "hijo"
    }
}
```

La lección: **los campos no son polimórficos, los métodos sí**. El acceso a un campo se resuelve en **tiempo de compilación**, según el *tipo declarado* de la referencia (`Padre p`), no según el objeto real. La llamada a un método se resuelve en **tiempo de ejecución**, según el objeto real (`Hijo`).

Además, el objeto `Hijo` contiene **dos** campos `nombre` distintos en memoria, ambos existentes simultáneamente. Desde `Hijo` se puede acceder al del padre con `super.nombre`.

Conclusión práctica: **no ocultes campos nunca**. No hay ningún caso de uso legítimo, produce código que se comporta de forma distinta según por qué referencia lo mires, y es una fuente de bugs muy difíciles de detectar en revisión. Si necesitás que una subclase aporte un valor distinto, usá un método sobrescrito o pasá el valor por constructor.

### Bloques de inicialización y el orden real de arranque

Los campos pueden inicializarse en tres sitios, y el orden importa. Java ofrece, además de la asignación en la declaración, dos tipos de bloque:

```java
public class Ejemplo {

    private static final Map<String, String> CACHE;
    private int contador;

    // 1. Bloque de inicialización ESTÁTICO: se ejecuta una vez, al cargar la clase
    static {
        CACHE = new HashMap<>();
        CACHE.put("clave", "valor");
        System.out.println("bloque estático");
    }

    // 2. Bloque de inicialización de INSTANCIA: se ejecuta antes de cada constructor
    {
        contador = 10;
        System.out.println("bloque de instancia");
    }

    public Ejemplo() {
        System.out.println("constructor");
    }
}
```

El orden garantizado al crear el primer objeto es:

1. **Al cargar la clase** (una sola vez): inicializadores de campos `static` y bloques `static`, en el orden textual en que aparecen.
2. **En cada `new`**: se reserva memoria y todos los campos de instancia reciben su valor por defecto (`0`, `false`, `null`).
3. Se ejecuta el constructor de la superclase (`super()`, implícito o explícito).
4. Inicializadores de campos de instancia y bloques de instancia, en orden textual.
5. El cuerpo del constructor.

Ese orden explica un error sutil que aparece cuando el orden textual de los campos importa:

```java
public class Roto {
    private int a = b + 1;   // ERROR: illegal forward reference
    private int b = 5;
}
```

Aunque el *scope* de `b` es toda la clase, su *inicialización* ocurre después, así que el compilador prohíbe explícitamente la referencia hacia adelante en los inicializadores. De nuevo: scope y lifetime son cosas distintas.

Los bloques de instancia se usan poco (normalmente basta el constructor), pero son útiles cuando varios constructores comparten lógica de inicialización. Los bloques estáticos sí son habituales para inicializar estructuras constantes complejas.

### `final` y `effectively final`

`final` aplicado a una variable significa **"no se puede reasignar"**. No significa que el objeto sea inmutable:

```java
final List<String> lista = new ArrayList<>();
lista.add("hola");            // PERFECTAMENTE LEGAL: mutamos el objeto
// lista = new ArrayList<>(); // ERROR: reasignamos la variable
```

Este malentendido es extremadamente común. `final` protege la *variable* (la flecha), no el *objeto* (la caja).

Un caso interesante es el **blank final**: un campo o local `final` sin valor inicial, que debe asignarse exactamente una vez antes de usarse:

```java
final int x;
if (condicion) {
    x = 1;
} else {
    x = 2;
}
System.out.println(x);   // OK: el compilador verifica que hay exactamente una asignación por camino
```

Desde Java 8 existe además el concepto de **effectively final** (efectivamente final): una variable que *no está* marcada `final` pero a la que **nunca se reasigna** después de su inicialización. El compilador la trata, a ciertos efectos, como si lo fuera.

```java
int a = 5;          // effectively final: nunca se reasigna
int b = 5;
b = 10;             // NO es effectively final
```

Esto importa por una razón muy concreta: las lambdas.

### Captura de variables en lambdas y clases anónimas

Una lambda puede leer variables del ámbito donde se define. A eso se le llama **captura**, y es lo que convierte a una lambda en un *closure*:

```java
public Runnable crearTarea() {
    String mensaje = "hola";               // effectively final
    return () -> System.out.println(mensaje);   // captura 'mensaje'
}
```

Pero hay una restricción estricta:

```java
public void ejemplo() {
    int contador = 0;
    Runnable r = () -> System.out.println(contador);
    contador++;   // ERROR: local variables referenced from a lambda expression
                  // must be final or effectively final
}
```

**Por qué existe esta restricción.** La explicación de fondo es la distinción stack/heap que ya vimos:

- Las variables locales viven en el **stack** del hilo que ejecuta el método.
- La lambda es un objeto que vive en el **heap** y puede sobrevivir al método que la creó (la devolvés, la guardás, la ejecutás en otro hilo).

Si la lambda pudiera referenciar directamente la variable del stack, tendríamos una referencia a memoria que ya no existe cuando el método retorna. La solución de Java es **capturar por valor**: el compilador copia el valor de la variable dentro del objeto lambda en el momento de crearla.

Y si es una copia, permitir reasignaciones sería engañoso: modificar la original después no afectaría a la copia, y modificar la copia no afectaría a la original. En vez de tolerar esa ambigüedad, Java exige que la variable no cambie nunca, con lo cual copia y original son indistinguibles por definición. La restricción también evita condiciones de carrera si la lambda se ejecuta en otro hilo.

Nótese que la restricción **solo aplica a variables locales**. Los campos de instancia y estáticos se pueden leer y modificar libremente desde una lambda, porque viven en el heap y se acceden a través de una referencia (`this` o la clase), no por copia:

```java
public class Contador {
    private int cuenta = 0;              // campo, no local

    public Runnable crearIncrementador() {
        return () -> cuenta++;           // LEGAL: es un campo
    }
}
```

Esto no es una laguna de la regla, es coherente: la lambda captura `this`, y a través de `this` accede al campo real. (Que sea *legal* no significa que sea *seguro* en concurrencia: ahí sí necesitás sincronización.)

**El truco clásico para "modificar" desde una lambda** —y por qué conviene desconfiar de él— es envolver el valor en un objeto mutable:

```java
int[] contador = {0};                       // la variable 'contador' nunca se reasigna
lista.forEach(x -> contador[0]++);          // mutamos el contenido del array
System.out.println(contador[0]);
```

Compila, porque la *variable* es effectively final aunque el *objeto* cambie. Pero suele ser una señal de que el problema está mal planteado: en la mayoría de casos la solución correcta es un `Stream` con un reductor o un collector, o un `AtomicInteger` si de verdad hay concurrencia.

### El caso del índice del bucle

Una consecuencia directa que confunde a quien viene de JavaScript:

```java
List<Runnable> tareas = new ArrayList<>();
for (int i = 0; i < 3; i++) {
    tareas.add(() -> System.out.println(i));   // ERROR de compilación
}
```

`i` se reasigna en cada iteración (`i++`), así que no es effectively final. Java **rechaza esto en compilación**, lo cual es una ventaja: en JavaScript con `var`, el equivalente compila y todos los closures acaban imprimiendo el mismo valor final, un bug clásico y silencioso.

El `for-each` sí funciona, porque cada iteración declara una variable nueva:

```java
for (String nombre : nombres) {
    tareas.add(() -> System.out.println(nombre));   // OK
}
```

Y si necesitás el índice, la solución idiomática es copiarlo a una local dentro del bucle:

```java
for (int i = 0; i < 3; i++) {
    final int indice = i;                          // nueva variable por iteración
    tareas.add(() -> System.out.println(indice));  // OK
}
```

### Scope en clases anidadas

Java permite declarar clases dentro de clases, y cada variante tiene reglas de acceso propias:

```java
public class Externa {
    private int campoExterno = 1;
    private static int campoEstatico = 2;

    // 1. Clase interna (inner): ligada a una instancia de Externa
    class Interna {
        void metodo() {
            System.out.println(campoExterno);    // OK: tiene acceso a 'this' de Externa
            System.out.println(campoEstatico);   // OK
        }
    }

    // 2. Clase anidada estática: NO ligada a ninguna instancia
    static class AnidadaEstatica {
        void metodo() {
            // System.out.println(campoExterno);  // ERROR: no hay instancia externa
            System.out.println(campoEstatico);    // OK
        }
    }

    void metodoConLocal() {
        int localExterna = 3;

        // 3. Clase local: declarada dentro de un método
        class Local {
            void metodo() {
                System.out.println(localExterna);   // OK si es effectively final
                System.out.println(campoExterno);   // OK
            }
        }
        new Local().metodo();
    }
}
```

La clase interna no estática mantiene una referencia implícita a la instancia externa, lo que le da acceso a sus campos pero también tiene un coste: **impide que el objeto externo sea recolectado mientras la interna viva**. Es una causa real de memory leaks, y la razón por la que la recomendación general es declarar las clases anidadas `static` salvo que necesites de verdad el acceso a la instancia externa.

Dentro de una clase interna, si necesitás desambiguar `this`, se usa la sintaxis cualificada:

```java
class Interna {
    private int valor = 10;
    void metodo() {
        System.out.println(this.valor);          // el de Interna
        System.out.println(Externa.this.valor);  // el de Externa
    }
}
```

---

## Los errores clásicos, uno por uno

### 1. Olvidar `this` en el constructor

El error número uno del principiante:

```java
public class Empleado {
    private String nombre;

    public Empleado(String nombre) {
        nombre = nombre;   // BUG: se asigna el parámetro a sí mismo
    }
}
```

Compila sin errores. El campo `nombre` queda en `null` y el bug aparece mucho después, como un `NullPointerException` en otro sitio.

Por qué pasa: dentro del constructor, ambos `nombre` se refieren al **parámetro**, porque sombrea al campo. La asignación es una operación inútil.

**Cómo protegerse:** los IDE y linters modernos lo detectan como *self-assignment*. Es un warning que conviene elevar a error en la configuración del proyecto. La corrección es siempre `this.nombre = nombre;`.

### 2. Ocultar campos en la herencia

```java
class Padre { protected String tipo = "padre"; }
class Hijo extends Padre { protected String tipo = "hijo"; }
```

Ya lo vimos: produce dos campos coexistiendo y acceso no polimórfico. **Nunca lo hagas.** Si la subclase necesita otro valor, sobrescribí un método o pasá el valor por el constructor del padre.

### 3. Declarar todas las variables al principio del método

Una costumbre heredada de C89, donde era obligatorio:

```java
// ANTI-PATRÓN
public void procesar(List<Pedido> pedidos) {
    int i;
    double total;
    Pedido pedido;
    String descripcion;
    // ...50 líneas después...
}
```

En Java es innecesario y perjudicial. Amplía el scope de cada variable a todo el método, lo que obliga al lector a rastrear dónde se usa realmente cada una y aumenta el riesgo de reutilizar una variable por accidente con un valor viejo.

**La regla, formulada por Joshua Bloch en *Effective Java* (Item 57): minimizá el scope de cada variable local.** En la práctica: declarala en el punto donde se usa por primera vez, y siempre que sea posible, inicializala en la misma declaración.

```java
// CORRECTO
public void procesar(List<Pedido> pedidos) {
    double total = 0;
    for (Pedido pedido : pedidos) {
        double subtotal = pedido.calcularSubtotal();
        total += subtotal;
    }
}
```

### 4. Usar un campo donde correspondía una variable local

Este es el error de diseño más caro de los que aparecen aquí, porque no lo detecta el compilador.

```java
// ANTI-PATRÓN
public class ProcesadorPedidos {
    private double total;          // ¿por qué es un campo?

    public double procesar(List<Pedido> pedidos) {
        total = 0;
        for (Pedido p : pedidos) {
            total += p.getSubtotal();
        }
        return total;
    }
}
```

Aquí `total` no es *estado del objeto*: es un resultado intermedio del cálculo. Convertirlo en campo produce tres problemas:

1. **No es thread-safe.** Dos hilos llamando a `procesar()` sobre la misma instancia se pisan el `total` mutuamente y ambos devuelven basura. Como variable local, cada hilo tendría la suya y el problema desaparecería.
2. **Alarga el lifetime innecesariamente.** El valor sobrevive a la llamada sin motivo.
3. **Miente sobre el modelo.** Cualquiera que lea la clase asumirá que `total` significa algo entre llamadas.

**El criterio de decisión, que conviene interiorizar:**

> Un campo representa **estado que el objeto necesita recordar entre llamadas**. Si el dato solo tiene sentido durante la ejecución de un método, es una variable local. Sin excepciones.

### 5. Estado mutable en `static`: memory leaks y tests frágiles

```java
// ANTI-PATRÓN
public class Cache {
    private static final Map<String, Usuario> USUARIOS = new HashMap<>();

    public static void registrar(Usuario u) {
        USUARIOS.put(u.getId(), u);   // nunca se elimina nada
    }
}
```

El problema es de lifetime. Un campo estático vive mientras la clase esté cargada, o sea, prácticamente toda la ejecución. En términos de garbage collection, ese `Map` actúa como **GC root**: todo lo que contenga —y todo lo que esos objetos referencien a su vez— es inalcanzable para el recolector. Si añadís sin eliminar nunca, el consumo de memoria crece de forma monótona hasta el `OutOfMemoryError`.

Es una de las causas más citadas de memory leaks en Java, junto con las variantes que retienen el `ClassLoader` completo (habitual en servidores de aplicaciones que redespliegan una app sin reiniciar la JVM: si un campo estático o un `ThreadLocal` retiene objetos de la app vieja, su ClassLoader nunca puede recolectarse, y con él se quedan todas sus clases).

Los efectos secundarios menos dramáticos pero igual de molestos:

- **Tests contaminados entre sí.** El estado estático persiste entre tests, así que el resultado depende del orden de ejecución.
- **Concurrencia rota.** Un `HashMap` estático accedido por varios hilos sin sincronizar puede corromperse.

**Alternativas:** una caché con expiración y tamaño máximo (`Caffeine`, `Guava Cache`), o mejor, un objeto de instancia cuyo ciclo de vida gestione un framework de inyección de dependencias. `static` mutable debería estar reservado a casos muy justificados y bien documentados.

### 6. Confundir `final` con inmutable

```java
private final List<String> items = new ArrayList<>();

public List<String> getItems() {
    return items;   // el llamante puede hacer items.clear()
}
```

`final` garantiza que el campo siempre apunte a esa lista; no garantiza que la lista no cambie. Si querés inmutabilidad real, devolvé una copia defensiva o una vista no modificable:

```java
public List<String> getItems() {
    return List.copyOf(items);   // Java 10+, copia inmutable
}
```

### 7. Fuga de `this` en el constructor

Un error avanzado pero de consecuencias graves:

```java
// ANTI-PATRÓN
public class Servicio {
    private final Config config;

    public Servicio(Registro registro) {
        registro.registrar(this);   // 'this' escapa antes de terminar la construcción
        this.config = new Config();
    }
}
```

Publicar `this` antes de que el constructor termine significa que otro código —u otro hilo— puede observar el objeto **a medio construir**, con campos `final` todavía sin asignar. Rompe incluso las garantías de visibilidad que el modelo de memoria de Java da a los campos `final`. La regla: no dejes que `this` escape del constructor, ni directamente, ni registrando listeners, ni arrancando hilos.

### 8. Asumir que el objeto se destruye al salir del scope

```java
void metodo() {
    Conexion c = abrirConexion();
}   // 'c' sale de scope, pero la conexión NO se cierra
```

Salir de scope no ejecuta nada. Java no tiene destructores. El objeto quedará *elegible* para el garbage collector, pero eso no libera recursos del sistema operativo (sockets, ficheros, conexiones), y ocurrirá en un momento indeterminado o nunca.

Para recursos siempre hay que cerrar explícitamente, y la forma correcta es `try-with-resources`:

```java
try (Conexion c = abrirConexion()) {
    // ...
}   // c.close() garantizado
```

### 9. Reutilizar una variable para dos propósitos

```java
// ANTI-PATRÓN
String temp = usuario.getNombre();
enviarSaludo(temp);
temp = usuario.getEmail();     // ahora significa otra cosa
enviarNotificacion(temp);
```

Legal, pero el nombre `temp` deja de significar algo estable, y en depuración tenés que reconstruir mentalmente en qué punto vale qué. Dos variables con nombres precisos son gratis: el compilador las optimiza al mismo código máquina.

### 10. Declarar variables dentro de un bucle "por eficiencia"

Un mito persistente dice que declarar dentro del bucle es más lento:

```java
for (int i = 0; i < 1000; i++) {
    String mensaje = "iteración " + i;   // ¿se "crea" 1000 veces?
}
```

No hay penalización por el hecho de declarar dentro. La variable local es una posición en el stack frame, que se reserva una sola vez al entrar al método. Declarar dentro del bucle **es lo correcto**, porque minimiza el scope. Lo que sí tiene coste es *crear objetos nuevos* en cada iteración, y eso es independiente de dónde declares la variable.

### 11. Sombrear una variable en un bloque anidado

A diferencia de otros lenguajes, Java **prohíbe** que una variable local sombree a otra variable local:

```java
void metodo() {
    int x = 1;
    {
        int x = 2;   // ERROR: variable x is already defined in method metodo()
    }
}
```

Esto es una decisión deliberada de diseño: el shadowing entre locales casi siempre es un error de tipeo, así que el lenguaje lo prohíbe. Solo se permite sombrear **campos**, que es el caso legítimo del constructor. Si venís de C++, JavaScript o Rust, donde esto sí se permite, es un cambio de expectativa.

---

## Avanzado

### Cómo implementa el compilador el scope

El scope es un fenómeno de **tiempo de compilación**. La JVM no tiene ninguna noción de "scope": trabaja con *slots* numerados en el stack frame.

Durante la compilación, `javac` mantiene una **tabla de símbolos** por ámbito, encadenada al ámbito padre. Al resolver un identificador busca en el ámbito actual, luego en el padre, y así hasta el ámbito de clase. El primer hallazgo gana — y eso, exactamente, *es* el shadowing: no una regla especial, sino la consecuencia natural del orden de búsqueda.

Al generar el bytecode, cada variable local se convierte en un índice numérico dentro del array de locales del frame. Y aquí aparece algo revelador: **el compilador reutiliza los slots de variables cuyos ámbitos no se solapan**.

```java
void metodo() {
    {
        int a = 1;
        System.out.println(a);
    }
    {
        int b = 2;      // puede ocupar el MISMO slot que 'a'
        System.out.println(b);
    }
}
```

Podés verificarlo compilando con información de depuración y desensamblando:

```bash
javac -g Ejemplo.java
javap -l -c Ejemplo.class
```

En la salida verás la `LocalVariableTable`, que asocia cada nombre a un slot y a un rango de bytecode. Ese rango **es** el scope, materializado. Sin `-g`, los nombres desaparecen del `.class` y solo quedan los números: prueba de que los nombres de las variables locales son puramente una comodidad para el programador.

### Definite assignment: el análisis detrás de "might not have been initialized"

La regla de que las locales deben inicializarse antes de usarse no es una heurística: está especificada formalmente en el capítulo 16 de la *Java Language Specification*, bajo el nombre de **definite assignment**.

El compilador realiza un análisis de flujo de datos: para cada uso de una variable, comprueba que **en todos los caminos de ejecución posibles** que llegan a ese punto, la variable haya recibido un valor. Si existe aunque sea un camino sin asignación, rechaza el programa.

```java
int x;
if (condicion) {
    x = 1;
}
System.out.println(x);   // ERROR: existe el camino en que 'condicion' es false
```

```java
int x;
if (condicion) {
    x = 1;
} else {
    x = 2;
}
System.out.println(x);   // OK: todos los caminos asignan
```

El análisis es conservador y a veces "no ve" cosas obvias para un humano:

```java
int x;
if (true) {
    x = 1;
}
System.out.println(x);   // OK: 'true' es constante, el compilador sí lo evalúa

final boolean SIEMPRE = true;
int y;
if (SIEMPRE) { y = 1; }
System.out.println(y);   // OK: constante final en tiempo de compilación

boolean variable = true;
int z;
if (variable) { z = 1; }
System.out.println(z);   // ERROR: aunque sepamos que siempre entra
```

La diferencia es que en los dos primeros casos la condición es una *constante en tiempo de compilación*, y en el tercero no. Entender esto evita horas de frustración con errores aparentemente absurdos.

El mismo mecanismo respalda el **flow scoping** del pattern matching y la comprobación de asignación única de los blank finals.

### Cómo se compila la captura de una lambda

Cuando escribís esto:

```java
public Runnable crear() {
    String mensaje = "hola";
    return () -> System.out.println(mensaje);
}
```

El compilador genera un método sintético que **recibe el valor capturado como parámetro**, más o menos equivalente a:

```java
private static void lambda$crear$0(String mensajeCapturado) {
    System.out.println(mensajeCapturado);
}
```

En tiempo de ejecución, `invokedynamic` construye una instancia que guarda ese valor copiado. Ver el mecanismo hace que la restricción de effectively final deje de parecer arbitraria: **el valor se copió en el momento de crear la lambda**, así que permitir reasignar el original solo generaría confusión sobre cuál de los dos valores es el bueno.

Un detalle de rendimiento que se aprecia en código muy caliente: una lambda que **no captura nada** puede reutilizar una única instancia (es *stateless*), mientras que una lambda que captura crea un objeto nuevo en cada evaluación. En bucles muy calientes, evitar capturas innecesarias tiene efecto medible.

### Scope y garbage collection

Que una variable salga de scope no dispara nada, pero sí afecta a la elegibilidad para recolección. Un caso interesante:

```java
void procesar() {
    byte[] enorme = new byte[100_000_000];
    usar(enorme);
    // 'enorme' sigue en scope hasta el final del método
    operacionMuyLarga();   // los 100 MB podrían seguir retenidos
}
```

Acotar el ámbito con un bloque ayuda al recolector a saber que el array ya no se necesita:

```java
void procesar() {
    {
        byte[] enorme = new byte[100_000_000];
        usar(enorme);
    }   // fin del scope: el slot puede reutilizarse y el array queda elegible
    operacionMuyLarga();
}
```

Matiz importante: en la práctica la JIT hace *liveness analysis* y suele considerar el objeto muerto tras su último uso real, incluso sin el bloque. Pero el comportamiento no está garantizado por la especificación, y en modo interpretado o con depuración activada puede diferir. Acotar el scope es la forma portable de expresar la intención.

### `ThreadLocal`: un scope por hilo

Java ofrece un mecanismo para variables cuyo ámbito no es léxico sino **por hilo**:

```java
private static final ThreadLocal<SimpleDateFormat> FORMATO =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

public String formatear(Date fecha) {
    return FORMATO.get().format(fecha);   // cada hilo tiene su propia instancia
}
```

Es útil para objetos costosos y no thread-safe (como `SimpleDateFormat`), o para propagar contexto —un ID de petición, el usuario autenticado— sin pasarlo como parámetro por toda la pila de llamadas.

Su gran riesgo es el lifetime: en un servidor con pool de hilos, los hilos se reutilizan indefinidamente, así que un valor guardado en un `ThreadLocal` y no eliminado sobrevive a la petición que lo creó. Es una causa clásica de memory leaks y de fugas de datos entre peticiones. **Siempre hay que llamar a `remove()` en un `finally`.**

### Scoped Values: el sustituto moderno

Java 25 finalizó (JEP 506) los **Scoped Values**, tras varias rondas de preview desde Java 20. Resuelven el mismo problema que `ThreadLocal` con un modelo mucho más seguro: el valor es **inmutable** y su ámbito está **delimitado sintácticamente** por la ejecución de un bloque.

```java
private static final ScopedValue<Usuario> USUARIO = ScopedValue.newInstance();

void manejarPeticion(Usuario u) {
    ScopedValue.where(USUARIO, u).run(() -> {
        procesar();          // dentro de este bloque, USUARIO.get() devuelve 'u'
    });
    // fuera del bloque, el binding ya no existe: no hace falta remove()
}
```

La diferencia conceptual con `ThreadLocal` es exactamente la que este documento viene distinguiendo: `ThreadLocal` tiene un lifetime abierto que hay que limpiar a mano; un Scoped Value tiene un **scope acotado** que se cierra solo. Además encaja de forma natural con los virtual threads de Java 21, donde crear millones de hilos hace inviable el modelo de `ThreadLocal`.

---

## Guía de decisión: dónde declaro esta variable

Un árbol de decisión práctico para el día a día:

**¿El dato solo se usa dentro de este método?**
→ Variable local, declarada lo más tarde posible, inicializada en la declaración.

**¿El dato lo aporta quien llama al método?**
→ Parámetro. Si el método tiene más de tres o cuatro, considerá agrupar en un objeto.

**¿El objeto necesita recordar el dato entre llamadas a métodos?**
→ Campo de instancia, `private`, y `final` siempre que sea posible.

**¿El valor es el mismo para todas las instancias y nunca cambia?**
→ `private static final` (o `public static final` si es parte de tu API), en `UPPER_SNAKE_CASE`.

**¿El valor es el mismo para todas las instancias pero cambia?**
→ Frená y revisá el diseño. Es estado global mutable. Casi siempre hay una alternativa mejor: inyección de dependencias, un parámetro explícito, o una caché con política de expiración.

---

## Trade-offs y entrevista

### Campo vs. variable local

| | Campo | Local |
|---|---|---|
| Thread-safety | Requiere sincronización | Segura por construcción |
| Lifetime | Vive con el objeto | Vive con la llamada |
| Testabilidad | Estado que hay que preparar y limpiar | Sin estado que arrastrar |
| Legibilidad | Obliga a rastrear toda la clase | Se entiende leyendo el método |

El sesgo por defecto debe ser **local**. Un campo hay que justificarlo.

### `static` vs. instancia

`static` gana en: constantes, funciones puras sin estado (métodos de utilidad), factorías.

`static` pierde en: cualquier cosa mutable. El coste real no es de rendimiento, es de **testabilidad y concurrencia**. Un `static` mutable no se puede sustituir por un mock, no se puede aislar entre tests y no se puede paralelizar sin sincronizar.

### Scope amplio vs. scope estrecho

Ampliar el scope nunca compra nada salvo, ocasionalmente, evitar recalcular un valor. Estrecharlo compra legibilidad, menor riesgo de reutilización accidental, mejores oportunidades para el optimizador y menor superficie de bug. La asimetría es tan clara que *Effective Java* lo formula como regla sin excepciones prácticas.

### Preguntas de entrevista

**¿Diferencia entre variable local, de instancia y estática?**
Dónde se declaran, dónde viven (stack / heap / área de clase), cuánto viven, y si tienen valor por defecto. La respuesta que distingue a un buen candidato añade: las locales son thread-safe por construcción, las de instancia requieren sincronización si el objeto se comparte, y las estáticas son estado global.

**¿Por qué una variable local no tiene valor por defecto y un campo sí?**
Porque leer una local sin inicializar casi siempre es un bug, y el compilador puede demostrarlo con definite assignment. Para un campo no puede: no sabe en qué orden se llamarán los métodos, así que la JVM garantiza memoria limpia.

**¿Por qué las lambdas exigen effectively final?**
Porque las locales viven en el stack y la lambda puede sobrevivir al método. Se capturan **por valor**, y permitir reasignar la original haría ambiguo qué valor observa la lambda. Los campos no tienen la restricción porque se acceden a través de una referencia al heap.

**¿Qué imprime este código?**
```java
class A { String s = "A"; }
class B extends A { String s = "B"; }
A obj = new B();
System.out.println(obj.s);
```
`"A"`. Los campos se resuelven en compilación según el tipo declarado; no son polimórficos. Es *hiding*, no *overriding*.

**¿Java pasa por valor o por referencia?**
Siempre por valor. Para objetos, se copia la referencia, así que podés mutar el objeto pero no reasignar la variable del llamante.

**¿Se puede acceder a un campo de instancia desde un método estático?**
No directamente: un método estático no tiene `this` y puede ejecutarse sin que exista ninguna instancia. Sí a través de una referencia explícita a un objeto.

**¿Qué es flow scoping?**
El ámbito de las variables de patrón (`instanceof String s`), determinado por análisis de flujo: la variable está en scope solo donde el compilador puede demostrar que el patrón coincidió.

---

## Alcance y temas relacionados

Lo que este documento **no** cubre y dónde continúa:

- **Tipos, rangos, wrappers y casting** → [03 - Data Types and Variables](03-data-types-and-variables.md)
- **Cómo se carga una clase y cuándo se inicializan sus estáticos** → [02 - Lifecycle of a Program](02-lifecycle-of-a-program.md)
- **Modificadores de acceso en profundidad, encapsulación** → bloque `02-POO`
- **Herencia y polimorfismo** (por qué los métodos sí son polimórficos) → bloque `02-POO`
- **Lambdas, closures y Stream API** → bloque `06-Streams-y-Funcional`
- **Modelo de memoria, visibilidad entre hilos, `volatile`** → bloque `07-Concurrencia`
- **Heap, stack, garbage collection y diagnóstico de leaks** → bloque `08-JVM-y-Memoria`

---

## Fuentes

**Recursos base:**
- [Variable Scope in Java - Baeldung](https://www.baeldung.com/java-variable-scope) — taxonomía de class / method / loop / bracket / nested scope. *Nota: el sitio devolvió HTTP 403 a los intentos de extracción automatizada; su contenido se recuperó de forma indirecta y se contrastó con las demás fuentes.*
- [Java Variables - Jenkov](https://jenkov.com/tutorials/java/variables.html) — los cuatro tipos de variable, declaración, asignación, convenciones de nombres, inferencia con `var`.
- [Java Fields - Jenkov](https://jenkov.com/tutorials/java/fields.html) — modificadores de acceso, static/no static, campos final, inicialización.
- [Java Methods - Jenkov](https://jenkov.com/tutorials/java/methods.html) — parámetros, parámetros final, variables locales.
- [Java Variables - W3Schools](https://www.w3schools.com/java/java_variables.asp) y [Java Scope - W3Schools](https://www.w3schools.com/java/java_scope.asp) — method scope, block scope, loop scope.
- [Java Variable Types - TutorialsPoint](https://www.tutorialspoint.com/java/java_variable_types.htm) y [Java Variable Scope - TutorialsPoint](https://www.tutorialspoint.com/java/java_variable_scope.htm) — locales, de instancia y de clase, con ejemplos y salidas.

**Documentación oficial:**
- [Declaring Member Variables - The Java Tutorials, Oracle](https://docs.oracle.com/javase/tutorial/java/javaOO/variables.html) — terminología oficial: fields, local variables, parameters.
- [JEP 394: Pattern Matching for instanceof](https://openjdk.org/jeps/394) — definición formal de flow scoping.

**Ampliación y discusiones técnicas:**
- [Why Do We Need Effectively Final? - Baeldung](https://www.baeldung.com/java-lambda-effectively-final-local-variables) — captura por valor y motivo de la restricción.
- [Variable and Method Hiding in Java - Baeldung](https://www.baeldung.com/java-variable-method-hiding) — diferencia entre hiding y overriding.
- [Static Fields and Garbage Collection - Baeldung](https://www.baeldung.com/java-static-fields-gc) y [Understanding Memory Leaks in Java - Baeldung](https://www.baeldung.com/java-memory-leaks) — estáticos como GC roots y fugas por ClassLoader.
- [Scoped Values in Java - Baeldung](https://www.baeldung.com/java-20-scoped-values) — alternativa moderna a `ThreadLocal`.
- [Variable Hiding and Variable Shadowing in Java - Scaler](https://www.scaler.com/topics/java/variable-hiding-and-variable-shadowing-in-java/) — casuística de sombreado.
- [Pattern matching and flow scoping in Java - Objectos](https://www.objectos.com.br/blog/pattern-matching-and-flow-scoping-in-java.html) — ejemplos de flow scoping con `&&`, `||` y negación.
- *Effective Java*, Joshua Bloch, 3.ª ed. — Item 57: "Minimize the scope of local variables".
