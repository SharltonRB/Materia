# Cheat Sheet · Conditionals

> Repaso rápido de [`Pura/Java/01-Basics/10-conditionals.md`](../../../Pura/Java/01-Basics/10-conditionals.md) · Java 17+ (con lo definitivo de Java 21)

## En 30 segundos

- La condición **tiene que ser un `boolean`**. No hay `if (1)` ni `if (lista)`.
- **`else if` no existe** como construcción: es un `else` cuyo cuerpo es otro `if`.
- **El orden de las ramas cambia el resultado**: la primera que se cumple gana.
- El `switch` moderno (flecha, expresión, patrones) resuelve de golpe el fall-through, el scope compartido y la exhaustividad.
- La mejor condicional es a menudo **la que no se escribe**: un `Map`, un `enum` o polimorfismo.

## `if`

```java
if (condicion) {
    // ...
} else if (otra) {     // ← en realidad: else { if (otra) { ... } }
} else {
}
```

**Dos bugs de sintaxis que compilan:**

```java
if (x > 100);          // ✗ el ; es el cuerpo del if; el bloque de abajo se ejecuta SIEMPRE
{ hacerAlgo(); }

if (x > 10)
    a();
    b();               // ✗ b() se ejecuta siempre: sin llaves, el if abarca UNA sentencia
```

El segundo es el bug **"goto fail"** de Apple (2014), que dejó abierta la verificación de certificados TLS. **Llaves siempre**, aunque el cuerpo sea una línea. Y activá `-Xlint:all`: el compilador detecta el punto y coma fantasma si se lo pedís.

**El dangling else** se resuelve siempre con **el `if` más cercano**, no con el que sugiere la indentación. Otra razón para las llaves.

## Condiciones bien escritas

```java
if (nombre == "admin")                  // ✗ identidad
if ("admin".equals(nombre))             // ✔ contenido y null-safe

if (integerA == integerB)               // ✗ falla a partir de 128
if (0.1 + 0.2 == 0.3)                   // ✗ nunca entra
if (Math.abs(a - b) < EPSILON)          // ✔

if (u != null && u.estaActivo())        // ✔ el orden de las guardas importa
if (u.estaActivo() && u != null)        // 💥 NPE

if (!estaLogueado && !tienePermiso)     // ✗ solo si fallan las DOS
if (!(estaLogueado && tienePermiso))    // ✔ De Morgan

if (validar() && registrarIntento())    // ✗ efectos colaterales: no registra los fallos

if (edad >= 18) return true; else return false;   // ✗
return edad >= 18;                                 // ✔
```

## El ternario

```java
condicion ? valorA : valorB
```

- **Es una expresión**: produce un valor. El `if` es una sentencia.
- **Tiene tipo**, y la promoción numérica actúa sobre ambas ramas: `cond ? 1 : 2.0` es **siempre `double`**.
- **Puede lanzar NPE** cuando mezcla un primitivo y un wrapper nullable:

```java
int n = flag ? mapa.get(k) : 0;                       // ✗ NPE latente
int n = Objects.requireNonNullElse(mapa.get(k), 0);   // ✔
```

**Usalo** para elegir entre dos valores del mismo tipo, en una línea. **No lo anides**.

## `switch`: las cuatro formas

|  | Etiqueta | Como sentencia | Como expresión |
|---|---|---|---|
| Clásico | `case X:` + `break` | sí | no |
| Flecha | `case X ->` | sí | **sí** |

```java
// Clásico: fall-through, scope compartido, break obligatorio
switch (nivel) {
    case PREMIUM: d = 0.10; break;
    case ORO:     d = 0.05; break;
    default:      d = 0;
}

// Flecha (Java 14+): sin fall-through, scope propio por rama
switch (nivel) {
    case PREMIUM -> d = 0.10;
    case ORO     -> d = 0.05;
}

// Expresión: devuelve un valor, y el compilador exige exhaustividad
double d = switch (nivel) {
    case PREMIUM -> 0.10;
    case ORO     -> 0.05;
    case BASICO  -> 0;
};

// yield: cuando la rama necesita un bloque
int dias = switch (mes) {
    case FEB -> 28;
    default  -> { int base = 30; yield base + ajuste(mes); }
};
```

**Sobre qué tipos se puede hacer `switch`:** `byte`, `short`, `char`, `int` y sus wrappers, `String` (Java 7+), `enum` (Java 5+) y — con patrones — cualquier referencia (Java 21).

**El scope compartido del `switch` clásico:** todas las ramas comparten un bloque, así que declarar `int x` en dos `case` no compila. La flecha lo resuelve.

**El fall-through** es un bug la mayoría de las veces… pero es **la respuesta correcta** cuando varios casos comparten cuerpo acumulando. Si lo usás a propósito, escribí `// fall through`.

### `null` y exhaustividad

```java
switch (texto) {          // ✗ NPE si texto es null: el default NO lo captura
    case "a" -> ...;
    default  -> ...;
}
switch (texto) {          // ✔ Java 21
    case null -> ...;
    case "a"  -> ...;
    default   -> ...;
}
```

**No pongas `default` en un `switch` sobre un `enum` o un tipo `sealed`:**

```java
return switch (estado) {
    case NUEVO  -> 48;
    case PAGADO -> 24;
    default     -> 0;      // ✗ se traga los estados futuros en silencio
};
```

Sin `default`, añadir un estado nuevo es **un error de compilación**, que es exactamente lo que querés. (Si el enum cambia tras compilar, el `switch` exhaustivo lanza `MatchException` en vez de devolver basura.)

## Pattern matching (Java 16 → 21)

```java
// instanceof con patrón + flow scoping
if (obj instanceof String s && s.length() > 5) { }

// patrones en las etiquetas case
String descripcion = switch (obj) {
    case Integer i when i > 100 -> "entero grande";   // guarda con 'when'
    case Integer i              -> "entero " + i;
    case String s               -> "texto de " + s.length();
    case null                   -> "nada";
    default                     -> "otra cosa";
};

// record patterns: descomponer
double area = switch (figura) {
    case Circulo(double r)        -> Math.PI * r * r;
    case Rect(double a, double b) -> a * b;
};
```

**Dominancia:** una etiqueta más general antes que una más específica **no compila** — el compilador detecta que la segunda sería inalcanzable. Ordená de específico a general.

**Tipos sellados** = uniones cerradas:

```java
sealed interface Pago permits Tarjeta, Efectivo, Transferencia { }
```

Con `sealed` + `record` + `switch` exhaustivo, el compilador garantiza que cubriste todos los casos, hoy y cuando alguien añada uno nuevo.

## Cuándo la condicional sobra

| En vez de | Usá |
|---|---|
| Cadena de `if` que traduce claves a valores | un `Map` |
| Comportamiento distinto por subtipo | **polimorfismo** |
| `if` sobre un `enum` | `switch` exhaustivo, o un campo del propio enum |
| `if (x == null) x = defecto;` | `Objects.requireNonNullElse` |
| Cuatro niveles de anidamiento | **cláusulas de guarda** con `return` temprano |

Las cláusulas de guarda son la herramienta más rentable del tema: rechazá lo inválido al principio y dejá el camino feliz sin indentar.

## Anti-patrones

| ✗ MAL | ✔ BIEN |
|---|---|
| `if (x > 100);` | quitar el `;` (y activar `-Xlint:all`) |
| `if` sin llaves con dos líneas | llaves siempre |
| `nombre == "admin"` | `"admin".equals(nombre)` |
| `!a && !b` para negar `a && b` | `!(a && b)` |
| Ternario con wrapper + primitivo | `requireNonNullElse` |
| `switch` sin `break` sin querer | sintaxis de flecha |
| `default` en `switch` sobre `enum`/`sealed` | quitarlo: que el compilador avise |
| Confiar en que `default` captura `null` | `case null` |
| Condiciones con efectos colaterales | sacarlos fuera |
| Cuatro niveles de anidamiento | guardas |
| `if (c) return true; else return false;` | `return c;` |
| `double` con `==` | tolerancia |
| Cadena de `if` que es una tabla | `Map` |
| Mezclar `->` y `:` en el mismo `switch` | migrar el `switch` entero |

## Checklist

- [ ] ¿Tiene llaves, aunque el cuerpo sea de una línea?
- [ ] ¿Hay algún `;` justo después de un `if`?
- [ ] ¿Compara objetos con `equals` y primitivos con `==`?
- [ ] ¿Hay algún `Integer`, `Long` o `Character` comparado con `==`?
- [ ] ¿Algún `double` comparado con `==`?
- [ ] ¿Las guardas de `null` están **antes** de desreferenciar?
- [ ] ¿Usa `&&` y `||` en lugar de `&` y `|`?
- [ ] ¿Alguna condición llama a métodos con efectos colaterales?
- [ ] Si es un ternario, ¿ambas ramas tienen el mismo tipo?
- [ ] Si es un `switch`, ¿usa la sintaxis de flecha? ¿Está escrito como expresión si produce un valor?
- [ ] Si es sobre `enum` o `sealed`, ¿evita el `default`?
- [ ] Si la referencia puede ser nula, ¿tiene `case null`?
- [ ] ¿El anidamiento pasa de tres niveles?
- [ ] ¿Existe una tabla, un polimorfismo o un tipo que haría innecesaria esta condicional?

## Qué construcción usar

| Si necesitás… | Usá |
|---|---|
| Decidir entre dos caminos con acciones distintas | `if / else` |
| Rechazar entradas inválidas al principio | cláusulas de guarda con `return` o `throw` |
| Elegir entre **dos valores** del mismo tipo | ternario |
| Un valor por defecto ante un `null` | `Objects.requireNonNullElse` |
| Elegir entre **varios valores** según un `enum` | `switch` expresión, con flecha, sin `default` |
| Ejecutar **acciones** distintas según un `enum` | `switch` sentencia, con flecha |
| Comparar contra muchas constantes | `switch` con flecha |
| Distinguir según el **tipo** en ejecución | `switch` con patrones, o polimorfismo |
| Descomponer un `record` | `switch` con record patterns |
| Modelar "una de N alternativas" | `sealed` + `record` + `switch` exhaustivo |
| Traducir claves a valores | un `Map` |
| Caer de un caso al siguiente acumulando | `switch` clásico con `:` y `// fall through` |

## Versión mínima por función

| Función | Desde |
|---|---|
| `switch` sobre `enum` | Java 5 |
| `switch` sobre `String` | Java 7 |
| Flecha `->`, `switch` expresión, `yield` | Java 14 |
| `instanceof` con patrón | Java 16 |
| `sealed` | Java 17 |
| Patrones en `case`, `case null`, guardas `when`, record patterns | **Java 21** |
| Patrones sobre primitivos | preview (Java 23-25) |

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Existe `else if` en Java?** No como construcción: es un `else` cuyo cuerpo es otro `if`.
- **¿Qué hace `if (x > 100);`?** El `;` es el cuerpo; lo que sigue se ejecuta siempre.
- **¿Qué es el dangling else?** Un `else` ambiguo se asocia siempre al `if` más cercano.
- **`switch` con `:` vs con `->`.** La flecha no cae al siguiente caso, crea scope por rama y puede usarse como expresión.
- **¿Por qué no poner `default` en un `switch` sobre `enum`?** Porque oculta los valores nuevos: sin él, añadir uno es error de compilación.
- **¿Qué pasa si el valor del `switch` es `null`?** Lanza NPE, salvo que haya `case null` (Java 21+).
- **¿Qué es el flow scoping?** El ámbito de la variable de patrón, determinado por análisis de flujo.
- **¿Qué es la dominancia entre etiquetas?** Poner una etiqueta general antes que una específica hace la segunda inalcanzable: no compila.
- **¿Qué genera un `switch` sobre `String`?** Un doble switch: primero sobre `hashCode`, luego `equals` para descartar colisiones.
- **`tableswitch` vs `lookupswitch`.** El primero cuando los casos son densos y contiguos (salto por índice, O(1)); el segundo cuando están dispersos (búsqueda binaria).

</details>
