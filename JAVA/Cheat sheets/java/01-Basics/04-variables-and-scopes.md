# Cheat Sheet · Variables and Scopes

> Repaso rápido de [`Pura/Java/01-Basics/04-variables-and-scopes.md`](../../../Pura/Java/01-Basics/04-variables-and-scopes.md) · Java 17+

## En 30 segundos

- **Scope** es *dónde se ve* una variable; **lifetime** es *cuánto vive*. No son lo mismo.
- El scope lo define el par de llaves donde se declara. El compilador lo resuelve; la JVM no tiene noción de scope.
- **Regla de oro (Effective Java, item 57): minimizá el scope de cada variable local.** Declarala donde se usa por primera vez e inicializala ahí mismo.
- Un **campo** hay que justificarlo. El sesgo por defecto es **local**.
- Los **campos no son polimórficos; los métodos sí**.

## Los ámbitos, uno por uno

| Ámbito | Qué abarca |
|---|---|
| **Class scope** | lo declarado entre las llaves de la clase, fuera de métodos: accesible desde cualquier método de esa clase |
| **Method scope** | lo declarado dentro de un método, incluidos sus parámetros |
| **Block scope** | cualquier par de llaves: `if`, `else`, `while`, `try` |
| **Loop scope** | la variable de la cabecera del `for` vive solo dentro del bucle |
| **`try-with-resources`** | el recurso vive dentro del bloque y se cierra al salir |
| **Flow scoping** | el de las variables de patrón, definido por análisis de flujo (Java 16+) |

Un método **no puede** ver las locales de otro, aunque uno llame al otro. Solo se comparte por campos, parámetros o valores de retorno.

```java
if (condicion) int x = 5;   // ✗ ERROR: un if sin llaves no crea bloque
```

No es un problema de scope sino de gramática: solo se permiten declaraciones dentro de un bloque. Otro argumento para poner siempre llaves.

```java
for (int i = 0; i < 3; i++) { }
for (int i = 0; i < 5; i++) { }   // ✔ legal: cada bucle tiene su ámbito

int i;                             // si necesitás el índice DESPUÉS del bucle
for (i = 0; i < 10; i++) { if (encontrado(i)) break; }
System.out.println("Se paró en " + i);
```

## El scope del `switch`: la trampa

En el `switch` clásico (`case ... :`) **todas las ramas comparten un único bloque y un único scope**.

```java
switch (dia) {
    case 1: int x = 10; break;
    case 2: int x = 20; break;      // ✗ "variable x is already defined"
}

switch (dia) {
    case 1: int x = 10; break;
    case 2: System.out.println(x);  // ✗ "might not have been initialized"
    // x ES visible aquí, pero el compilador no puede garantizar que se asignó
}
```

Es el ejemplo perfecto de que **scope y asignación son comprobaciones independientes**. Solución clásica: llaves por rama. Solución moderna (Java 14+): la **flecha**, que crea ámbito propio automáticamente.

```java
switch (dia) {
    case 1 -> { int x = 10; }
    case 2 -> { int x = 20; }   // ✔ sin conflicto
}
```

Esta es una de las razones —además de eliminar el fall-through accidental— por las que el `switch` con flecha es hoy la forma preferida.

## Flow scoping (Java 16+)

Con pattern matching, la variable del patrón está en ámbito **exactamente donde el compilador puede demostrar que el patrón coincidió**.

```java
if (obj instanceof String s && s.length() > 5) { }   // ✔ s en scope: el && solo evalúa si coincidió
if (obj instanceof String s || s.length() > 5) { }   // ✗ el || puede no haber coincidido

public String describir(Object obj) {
    if (!(obj instanceof String s)) {
        return "no es un String";      // aquí 's' NO está en scope
    }
    return "longitud " + s.length();   // aquí SÍ: si llegamos, coincidió
}
```

Ese último caso es el que hace que el pattern matching encaje tan bien con el estilo de *early return*.

## Shadowing vs hiding

**Shadowing**: entre ámbitos anidados del mismo contexto — un parámetro o local tapa a un campo. Es el único caso legítimo, y por eso existe `this`:

```java
public Empleado(String nombre) {
    this.nombre = nombre;    // ✔ this.nombre es el campo
}
public static void setEntorno(String entorno) {
    Config.entorno = entorno;   // 'this' no existe en contexto estático
}
```

**Hiding**: entre clases de una jerarquía — un campo de la subclase con el nombre de uno de la superclase. Y tiene una consecuencia que sorprende:

```java
class Padre { String nombre = "padre"; String getNombre() { return nombre; } }
class Hijo extends Padre { String nombre = "hijo"; String getNombre() { return nombre; } }

Padre p = new Hijo();
p.nombre;        // "padre"  (!)
p.getNombre();   // "hijo"
```

**Los campos se resuelven en compilación según el tipo declarado; los métodos en ejecución según el objeto real.** El objeto `Hijo` contiene **dos** campos `nombre` en memoria a la vez. **No ocultes campos nunca**: no hay caso de uso legítimo.

Java además **prohíbe** que una local sombree a otra local (a diferencia de C++, JavaScript o Rust): casi siempre es un error de tipeo.

```java
int x = 1;
{ int x = 2; }   // ✗ "variable x is already defined"
```

## Los 11 errores clásicos

| # | Error | Por qué duele |
|---|---|---|
| 1 | `nombre = nombre;` en el constructor | compila y no hace nada; el campo queda `null` y el NPE aparece en otro sitio |
| 2 | Ocultar campos en la herencia | dos campos coexistiendo y acceso no polimórfico |
| 3 | Declarar todo al principio del método | costumbre de C89; amplía el scope y obliga a rastrear cada variable |
| 4 | **Un campo donde iba una local** | no thread-safe, alarga el lifetime y miente sobre el modelo |
| 5 | Estado mutable en `static` | actúa como **GC root**: memory leak, tests contaminados, concurrencia rota |
| 6 | Confundir `final` con inmutable | `final List` sigue admitiendo `clear()` desde fuera |
| 7 | Fuga de `this` en el constructor | otro hilo puede ver el objeto a medio construir, con `final` sin asignar |
| 8 | Creer que salir de scope destruye el objeto | Java no tiene destructores: sockets y ficheros siguen abiertos |
| 9 | Reutilizar una variable para dos propósitos | el nombre deja de significar algo estable |
| 10 | No declarar dentro del bucle "por eficiencia" | mito: la local es un slot reservado una vez al entrar al método |
| 11 | Sombrear una local con otra local | no compila |

### El error 4, en detalle (el más caro)

```java
public class ProcesadorPedidos {
    private double total;                 // ✗ ¿por qué es un campo?
    public double procesar(List<Pedido> pedidos) {
        total = 0;
        for (Pedido p : pedidos) total += p.getSubtotal();
        return total;
    }
}
```

Dos hilos sobre la misma instancia se pisan el `total` y ambos devuelven basura. Como local, cada hilo tendría la suya.

> **El criterio:** un campo representa **estado que el objeto necesita recordar entre llamadas**. Si el dato solo tiene sentido durante la ejecución de un método, es una variable local. Sin excepciones.

### El error 5, en detalle

```java
private static final Map<String, Usuario> USUARIOS = new HashMap<>();   // nunca se elimina nada
```

Un campo estático vive mientras la clase esté cargada. Es un **GC root**: todo lo que contenga es inalcanzable para el recolector, y el consumo crece hasta el `OutOfMemoryError`. La variante que retiene el `ClassLoader` completo es la causa canónica de fugas en servidores que redespliegan sin reiniciar.

**Alternativas:** caché con expiración y tamaño máximo (Caffeine, Guava), o un objeto de instancia cuyo ciclo de vida gestione la inyección de dependencias.

### Errores 6 y 8, correcciones

```java
public List<String> getItems() { return List.copyOf(items); }   // copia inmutable (Java 10+)

try (Conexion c = abrirConexion()) { }   // c.close() garantizado
```

## Guía de decisión: dónde declaro esta variable

**¿El dato solo se usa dentro de este método?**
→ Local, declarada lo más tarde posible, inicializada en la declaración.

**¿Lo aporta quien llama?**
→ Parámetro. Con más de tres o cuatro, agrupá en un objeto.

**¿El objeto necesita recordarlo entre llamadas?**
→ Campo de instancia, `private`, y `final` siempre que se pueda.

**¿Es el mismo para todas las instancias y nunca cambia?**
→ `private static final` en `UPPER_SNAKE_CASE`.

**¿Es el mismo para todas las instancias pero cambia?**
→ **Frená y revisá el diseño.** Es estado global mutable. Casi siempre hay algo mejor: inyección de dependencias, un parámetro explícito, o una caché con expiración.

## Trade-offs

| | Campo | Local |
|---|---|---|
| Thread-safety | requiere sincronización | **segura por construcción** |
| Lifetime | vive con el objeto | vive con la llamada |
| Testabilidad | estado que preparar y limpiar | sin estado que arrastrar |
| Legibilidad | obliga a rastrear toda la clase | se entiende leyendo el método |

`static` gana en: constantes, funciones puras de utilidad, factorías. `static` pierde en cualquier cosa mutable — y el coste real no es de rendimiento, es de **testabilidad y concurrencia**: no se puede mockear, ni aislar entre tests, ni paralelizar sin sincronizar.

## Bajo el capó

El scope es un fenómeno de **compilación**. `javac` mantiene una tabla de símbolos por ámbito encadenada al padre; al resolver un identificador busca del ámbito actual hacia afuera y **el primer hallazgo gana** — eso *es* el shadowing, no una regla especial sino la consecuencia del orden de búsqueda.

En el bytecode cada local es un índice numérico en el array de locales del frame, y **el compilador reutiliza los slots de variables cuyos ámbitos no se solapan**.

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Local vs de instancia vs estática?** Dónde se declaran, dónde viven (stack / heap / área de clase), cuánto viven y si tienen valor por defecto. La respuesta que distingue a un buen candidato añade: las locales son thread-safe por construcción, las de instancia requieren sincronización si el objeto se comparte, y las estáticas son estado global.
- **¿Por qué una local no tiene valor por defecto y un campo sí?** Porque leer una local sin inicializar casi siempre es un bug y el compilador puede demostrarlo con *definite assignment*. Para un campo no puede: no sabe en qué orden se llamarán los métodos.
- **¿Por qué las lambdas exigen effectively final?** Porque las locales viven en el stack y la lambda puede sobrevivir al método: se capturan **por valor**, y permitir reasignar la original haría ambiguo qué valor observa.
- **`class A { String s = "A"; } class B extends A { String s = "B"; } A obj = new B(); obj.s`** → `"A"`. Los campos se resuelven en compilación según el tipo declarado. Es *hiding*, no *overriding*.
- **¿Java pasa por valor o por referencia?** Siempre por valor. Con objetos se copia la referencia: podés mutar el objeto, no reasignar la variable del llamante.
- **¿Se puede acceder a un campo de instancia desde un método estático?** No directamente: no tiene `this` y puede ejecutarse sin instancias. Sí a través de una referencia explícita.
- **¿Qué es flow scoping?** El ámbito de las variables de patrón, determinado por análisis de flujo: en scope solo donde el compilador puede demostrar que el patrón coincidió.

</details>
