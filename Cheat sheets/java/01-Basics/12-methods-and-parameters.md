# Cheat Sheet · Methods and Parameters

> Repaso rápido de [`Pura/Java/01-Basics/12-methods-and-parameters.md`](../../../Pura/Java/01-Basics/12-methods-and-parameters.md) · Java 17+

## En 30 segundos

- **Java pasa siempre por valor. Siempre.** Con objetos se copia **la referencia**, no el objeto: por eso podés mutar lo que apunta, pero no reasignar la variable de quien llama.
- **La firma no incluye el tipo de retorno**: dos métodos que solo difieren en él no compilan.
- La sobrecarga se resuelve **en compilación**, en tres fases, y el resultado **queda grabado en el bytecode**.
- Un varargs **es un array** con azúcar sintáctico.
- Java **no elimina las llamadas de cola**. La recursión sobre estructuras lineales es un bucle con pila.

## Anatomía

```java
public static int sumar(int a, int b) throws MiExcepcion { return a + b; }
//  ^modif  ^ret  ^nombre ^parámetros    ^cláusula throws
```

**Parámetro** es lo que declara el método; **argumento** es lo que se pasa al llamarlo.

**Estático vs de instancia:** el estático pertenece a la clase, no tiene `this` y no puede tocar campos de instancia directamente. Llamarlo a través de una instancia (`objeto.metodoEstatico()`) es legal, confuso y **funciona incluso si `objeto` es `null`**.

## Paso por valor: el punto que casi todas las fuentes explican mal

```java
static void intercambiar(Foo a, Foo b) { Foo t = a; a = b; b = t; }   // ✗ no hace NADA fuera

static void f(List<String> l) { l = new ArrayList<>(); }   // ✗ no cambia nada fuera
static void f(List<String> l) { l.clear(); }               // ✔ esto SÍ muta el objeto
```

**La regla completa:** se copia el valor de la variable. Si es un primitivo, se copia el número; si es una referencia, se copia la dirección. Ambas variables apuntan al mismo objeto, así que **mutarlo se ve fuera**, pero **reasignar la variable local no**.

Decir "los objetos se pasan por referencia" es incorrecto y produce exactamente el bug del método `intercambiar`.

## Retorno

```java
if (cond) return 1;          // ✗ "missing return statement": falta el camino else
                             //   el compilador exige que TODO camino devuelva
```

Código después de un `return` es un `unreachable statement`: error de compilación.

```java
try { return 1; } finally { return 0; }   // ✗ el return del finally DESCARTA
                                          //   el valor y hasta la excepción en curso
```

**Nunca pongas `return`, `break` o `continue` en un `finally`.**

**Devolver `null` en vez de una colección vacía** obliga a un `if` en cada llamador. `return List.of();`.

**Ignorar el valor devuelto de un tipo inmutable** es un no-op silencioso: `texto.trim();` no hace nada.

**Devolver la estructura interna** regala el control del estado:

```java
List<String> getLineas() { return lineas; }               // ✗
List<String> getLineas() { return List.copyOf(lineas); }  // ✔ copia
List<String> getLineas() { return Collections.unmodifiableList(lineas); }  // ✔ vista
```

## Sobrecarga

**La firma = nombre + tipos de los parámetros (en orden).** No incluye el tipo de retorno, ni los nombres de los parámetros, ni `throws`.

### Las tres fases de la resolución

El compilador prueba **en orden**, y se queda en la primera fase donde encuentra un candidato:

| Fase | Qué permite |
|---|---|
| **1** | solo **ensanchamiento** (widening) — sin boxing, sin varargs |
| **2** | ensanchamiento **+ boxing/unboxing** |
| **3** | todo lo anterior **+ varargs** |

Consecuencias directas:

```java
void m(long x); void m(Integer x);
m(5);                    // llama a m(long): la fase 1 gana antes de considerar boxing

void m(Object o); void m(Object... o);
m(null);                 // llama a m(Object): varargs es la ÚLTIMA fase

f(null);                 // si hay f(String) y f(Integer): ambiguo, no compila
                         // el MÁS ESPECÍFICO gana; si ninguno lo es, error
```

**Ensanchamiento y boxing no se combinan:** `Long l = 1;` no compila (haría falta `int → long → Long`).

**La resolución ocurre en compilación y queda grabada.** Si cambiás las sobrecargas de una librería y no recompilás a quien la usa, seguirá llamando a la que se eligió entonces.

### La trampa canónica de la biblioteca estándar

```java
List<Integer> l = ...;
l.remove(1);                    // remove(int index) → borra la POSICIÓN 1
l.remove(Integer.valueOf(1));   // remove(Object)    → borra el VALOR 1
```

### Cuándo no sobrecargar

- Cuando los métodos **hacen cosas distintas**: ponéles nombres distintos.
- Cuando los tipos son **convertibles entre sí** (`f(int)`, `f(long)`, `f(Integer)`, `f(Object)`): nadie sabe cuál sale.
- Cuando sobrecargás un método **con su propia versión varargs**: cambia el destino de `m(null)`.

Alternativa idiomática: **factorías estáticas con nombre** (`of`, `from`, `parse`, `valueOf`).

## Varargs

```java
void f(String... args)     // ES void f(String[] args), con azúcar en la llamada
```

**Reglas:** como mucho uno, y **siempre el último** parámetro.

```java
f();                  // args es un array VACÍO, nunca null
f(null);              // ⚠️ aviso del compilador: args ES null → NPE al usarlo
f((String) null);     // ✔ explícito: array de un elemento que vale null

Arrays.asList(arrayDeInt);    // ⚠️ lista de UN elemento: int[] no es Integer[]
```

**Contaminación del heap:** un varargs genérico (`T...`) crea un array de un tipo que no existe en runtime. `@SafeVarargs` es una **promesa** de que solo leés del array. Es falsa si lo guardás o lo dejás escapar:

```java
@SafeVarargs static <T> T[] toArray(T... a) { return a; }   // ✗ la promesa es mentira
```

Y por seguridad, **copiá el array dentro del método** si lo vas a conservar: quien llama puede seguir teniendo la referencia.

## Recursión

**Cuándo compensa:** estructuras **recursivas** — árboles, grafos, JSON anidado, parsers. **Cuándo no:** estructuras lineales, donde es un bucle que además consume pila.

```java
static int sum(int k) { return k > 0 ? k + sum(k - 1) : 0; }   // ✗ es un bucle con pila
```

- **`StackOverflowError`** llega a las pocas miles de llamadas (la pila de un hilo son unos cientos de KB). El caso base tiene que ser alcanzable **para toda entrada posible**.
- **Java NO optimiza las llamadas de cola.** Reescribir en forma recursiva de cola no evita el desbordamiento.
- **Coste exponencial:** Fibonacci ingenuo recalcula los mismos subproblemas. Memoización o versión iterativa.
- **Desbordamiento silencioso:** `factorial(int)` da un resultado **incorrecto sin avisar a partir de 13**. Usá `long`, `BigInteger` o `Math.multiplyExact`.

## Límites que existen de verdad

| Límite | Valor |
|---|---|
| Parámetros por método | **255** (`long` y `double` cuentan por dos) |
| Tamaño del bytecode de un método | **64 KB** |
| Inlining del JIT | los métodos pequeños se incrustan; los grandes, no |

Ese último es el argumento de rendimiento a favor de los métodos pequeños: **el JIT los incrusta y desaparece la llamada**. "Extraer un método cuesta rendimiento" es un mito.

## Diseño de métodos

- **Tres parámetros o menos.** Con más, agrupá en un `record` o usá un builder.
- **Nada de parámetros booleanos en la llamada**: `generarInforme(datos, true, false)` no dice nada. Usá enums o métodos con nombre.
- **Cuidado con dos parámetros consecutivos del mismo tipo**: se invierten por error y compila igual.
- **Validá en la frontera**: `Objects.requireNonNull(x, "x")` y `IllegalArgumentException` en todo método público.
- **Separá cálculo y efecto**: un método puro se prueba sin simular nada.
- **Un método, un nivel de abstracción.** Y que quepa en una pantalla.
- **Los nombres de los parámetros no viajan al `.class`** salvo que compiles con `-parameters`. Frameworks como Spring lo necesitan.
- **El orden de evaluación de los argumentos es de izquierda a derecha**, garantizado — pero depender de ello (`procesar(cola.poll(), cola.poll())`) es ilegible.

## Anti-patrones (22, condensados)

| ✗ MAL | ✔ BIEN |
|---|---|
| Creer que los objetos se pasan por referencia | siempre por valor |
| Reasignar un parámetro esperando verlo fuera | mutar, o devolver el nuevo valor |
| `return` / `break` / `continue` en un `finally` | sacarlo |
| `return 0;` de relleno para callar al compilador | documentar el centinela o lanzar |
| `return null;` en vez de colección vacía | `List.of()` |
| `texto.trim();` sin asignar | `texto = texto.trim();` |
| Devolver la colección interna | `List.copyOf` / `unmodifiableList` |
| Sobrecargar métodos que hacen cosas distintas | nombres distintos |
| Sobrecargar con tipos convertibles entre sí | evitarlo |
| Sobrecargar un método con su versión varargs | evitarlo |
| `Arrays.asList(arrayDeInt)` | `Arrays.stream(v).boxed().toList()` |
| `recibir(null)` a un varargs | `recibir((String) null)` |
| `@SafeVarargs` en un método que deja escapar el array | copiar el array |
| Recursión sobre una estructura lineal | un bucle |
| Recursión aritmética con `int` sin control | `long`, `BigInteger` o `*Exact` |
| Confiar en la optimización de llamada de cola | Java no la hace |
| Seis o más parámetros | `record` o builder |
| `generarInforme(datos, true, false)` | enums o métodos con nombre |
| `nombre = nombre;` fuera de un setter | `this.nombre = nombre;` |
| Mezclar cálculo y efecto | separarlos |
| `objeto.metodoEstatico()` | `Clase.metodoEstatico()` |
| Depender del orden de evaluación de argumentos | extraer variables |

## Checklist

- [ ] ¿El nombre dice qué hace, con un verbo, sin un "y" dentro?
- [ ] ¿Tiene tres parámetros o menos?
- [ ] ¿Hay algún `boolean` que debería ser un enum?
- [ ] ¿Dos parámetros consecutivos del mismo tipo que se puedan invertir?
- [ ] ¿Se valida lo que entra, si el método es público?
- [ ] ¿Se muta algún parámetro? Si sí, ¿lo dicen el nombre y el Javadoc?
- [ ] ¿Se reasigna algún parámetro sin motivo?
- [ ] ¿Todos los caminos devuelven algo con sentido?
- [ ] Si devuelve una colección, ¿devuelve vacía en vez de `null`?
- [ ] Si devuelve una estructura interna, ¿está copiada o es de solo lectura?
- [ ] ¿Hay algún `return` dentro de un `finally`?
- [ ] Si está sobrecargado, ¿las sobrecargas hacen lo mismo con datos equivalentes? ¿Alguna llamada con `null` sería ambigua?
- [ ] Si tiene varargs, ¿el array se copia y no se deja escapar?
- [ ] Si es recursivo, ¿el caso base es alcanzable para toda entrada? ¿La profundidad está acotada?
- [ ] ¿Cabe en una pantalla y se mantiene en un solo nivel de abstracción?
- [ ] ¿Se puede separar el cálculo del efecto?

## Qué construcción usar

| Si necesitás… | Usá |
|---|---|
| Una operación que no depende de ningún objeto | método `static` |
| Una operación que depende del estado del objeto | método de instancia |
| Un número variable de argumentos del mismo tipo | varargs, **copiando el array dentro** |
| Un número variable ya empaquetado | un parámetro `List` o array |
| Muchos parámetros que viajan juntos | un `record` |
| Muchos parámetros opcionales | un builder |
| Un valor por defecto | una sobrecarga que delegue en la implementación única |
| El mismo nombre para operaciones distintas | **no**: nombres distintos |
| Varias formas de construir algo | factorías estáticas con nombre (`of`, `from`, `parse`) |
| Indicar "no hay resultado" | `Optional` **como retorno**, nunca como parámetro |
| Devolver una colección sin resultados | `List.of()`, nunca `null` |
| Rechazar un argumento nulo | `Objects.requireNonNull(x, "x")` |
| Rechazar un valor fuera de rango | `IllegalArgumentException` |
| Recorrer una estructura recursiva | recursión |
| Repetir sobre una estructura lineal | un bucle |
| Recursión con recálculo de subproblemas | memoización o versión iterativa |
| Aritmética que puede desbordar | `Math.multiplyExact`, `addExact` o `BigInteger` |
| Pasar comportamiento, no datos | `Supplier`, `Function`, `Predicate` |
| Que los nombres de los parámetros sobrevivan | compilar con `-parameters` |

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Java pasa por valor o por referencia?** **Siempre por valor.** Con objetos se copia la referencia: podés mutar el objeto, no reasignar la variable del llamante.
- **¿Por qué un método `intercambiar(a, b)` no funciona?** Porque reasigna copias locales de las referencias.
- **¿Qué forma la firma de un método?** Nombre y tipos de los parámetros. **No** el tipo de retorno.
- **¿Se puede sobrecargar cambiando solo el retorno?** No: no compila.
- **¿Las tres fases de la resolución de sobrecarga?** Ensanchamiento; ensanchamiento + boxing; y luego varargs.
- **¿Por qué `Long l = 1;` no compila?** Widening y boxing no se combinan.
- **¿`list.remove(1)` sobre `List<Integer>`?** Llama a `remove(int index)`: borra la posición, no el valor.
- **¿Cuándo se resuelve la sobrecarga?** En compilación, y queda grabada en el bytecode.
- **¿Qué es la contaminación del heap y para qué es `@SafeVarargs`?** Un array genérico cuyo tipo no existe en runtime; la anotación promete que el método solo lee de él.
- **¿Optimiza Java las llamadas de cola?** No.
- **¿Cuántos parámetros admite un método?** 255 (`long` y `double` cuentan por dos).
- **¿Qué pasa con un `return` dentro de un `finally`?** Descarta el valor de retorno y hasta la excepción en curso.

</details>
