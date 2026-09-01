# Cheat Sheet · Loops

> Repaso rápido de [`Pura/Java/01-Basics/11-loops.md`](../../../Pura/Java/01-Basics/11-loops.md) · Java 17+

## En 30 segundos

- Cuatro formas: `while`, `do-while`, `for` clásico y `for-each`. Lo que las distingue es **cuándo se comprueba la condición** y **quién controla el avance**.
- El `for-each` se compila a un bucle con `Iterator`: por eso funciona con cualquier `Iterable` y por eso modificar la colección mientras se recorre lanza `ConcurrentModificationException`.
- **La variable del `for-each` es una copia**: asignarle algo no modifica la colección.
- Casi todos los bugs de bucles son de **frontera** (`<=` en vez de `<`), **avance** (un `continue` que salta el incremento) o **modificación durante el recorrido**.
- `forEach` **no es un bucle**: es una llamada a método, y por eso no admite `break`.

## Las cuatro formas

```java
while (cond) { }               // comprueba ANTES: puede no ejecutarse nunca
do { } while (cond);           // comprueba DESPUÉS: se ejecuta al menos una vez
for (init; cond; avance) { }   // las tres partes juntas y visibles
for (T x : iterable) { }       // recorrido completo, sin índice
```

**Las tres partes del `for` son opcionales**: `for (;;)` es un bucle infinito válido y es la forma idiomática de escribirlo (mejor que `while (true)`, aunque ambas valen).

**El scope de la variable de control:** declarada en la cabecera, muere con el bucle. Eso permite reutilizar `i` en otro bucle del mismo método; si necesitás el valor después, declarala fuera.

## Cuál elegir

| Si necesitás… | Usá |
|---|---|
| Recorrer todos los elementos | `for-each` |
| El índice, ir hacia atrás o saltando | `for` clásico |
| Recorrer dos colecciones en paralelo | `for` clásico con índice |
| Repetir un número conocido de veces | `for` clásico |
| Repetir hasta que ocurra algo | `while` |
| Ejecutar al menos una vez y luego comprobar | `do-while` |
| Un bucle de servicio que no debe terminar | `while (true)` con salida clara |

## `break`, `continue`, etiquetas y `return`

```java
break;          // sale del bucle (o del switch) más interno
continue;       // salta a la siguiente iteración
break etiqueta; // sale del bucle etiquetado
return;         // sale del método entero, esté donde esté
```

```java
externo:
for (...) {
    for (...) {
        if (encontrado) break externo;
    }
}
```

**Preferí extraer un método con `return`** antes que usar etiquetas: se lee mucho mejor. Las etiquetas son la única forma de salir de varios bucles a la vez sin extraer, y por eso siguen existiendo.

**Trampa clásica:** un `break` dentro de un `switch` que está dentro de un bucle **sale del `switch`, no del bucle**.

## Los bugs de frontera y avance

```java
for (int i = 0; i <= v.length; i++)      // ✗ ArrayIndexOutOfBoundsException garantizada
for (int i = 0; i < v.length; i++)       // ✔

for (int i = 0; i < 10; i++) { i++; }    // ✗ avanza de dos en dos
while (i < 10) { if (x) continue; i++; } // ✗ bucle infinito: el continue salta el avance

for (int i = 0; i != 10; i += 3)         // ✗ se salta el 10 y no para nunca
for (int i = 0; i < 10; i += 3)          // ✔ usá < y >, no !=

for (double x = 0.0; x != 1.0; x += 0.1) // ✗ nunca termina: 0.1 no es exacto en binario
for (int i = 0; i < 10; i++) { double x = i / 10.0; }   // ✔

for (int i = 0; i < 5; i++);             // ✗ el ; es el cuerpo del bucle
{ hacerAlgo(); }
```

### Desbordamiento del contador: el "bucle infinito" que sí termina

```java
for (int i = 0; i <= Integer.MAX_VALUE; i++)   // ✗ la condición NUNCA es falsa
for (byte i = 0; i < 128; i++)                 // ✗ 127 + 1 = -128
```

Caso instructivo: `for (int i = 1; i <= 10; --i)` se cita en muchos tutoriales como "bucle infinito". **No lo es**: el contador desborda a `Integer.MIN_VALUE` y el bucle termina tras unos 2.147 millones de vueltas, en menos de un segundo. La lección es doble: los tutoriales se equivocan, y comprobarlo cuesta segundos.

## Modificar mientras se recorre

```java
for (String s : lista) { if (cond(s)) lista.remove(s); }   // 💥 ConcurrentModificationException
```

**Las tres soluciones correctas:**

```java
lista.removeIf(s -> cond(s));                       // ✔ la más limpia
Iterator<String> it = lista.iterator();             // ✔ si la lógica es compleja
while (it.hasNext()) { if (cond(it.next())) it.remove(); }
for (String s : new ArrayList<>(lista)) { }         // ✔ copiar antes de recorrer
```

**`fail-fast` no es una garantía.** El `modCount` es una comprobación *best-effort*: hay casos que **no lanzan** la excepción y terminan el recorrido antes de tiempo en silencio — el más famoso, borrar el penúltimo elemento de un `ArrayList`, que hace que `hasNext()` devuelva `false` sin avisar. No confíes en la excepción como red de seguridad.

Para varios hilos: colecciones concurrentes (`CopyOnWriteArrayList`, `ConcurrentHashMap`), que iteran sobre una instantánea o son débilmente consistentes.

## Recorrer un `Map`

```java
for (K k : mapa.keySet()) { V v = mapa.get(k); }   // ✗ una búsqueda extra por vuelta
for (Map.Entry<K,V> e : mapa.entrySet()) { }       // ✔
mapa.forEach((k, v) -> { });                       // ✔ si no necesitás salir
```

## Rendimiento: lo que sí importa

| Problema | Por qué duele |
|---|---|
| **`get(i)` sobre una `LinkedList`** | O(n) por acceso → el bucle entero es **O(n²)**. Con `for-each` es O(n) |
| **Concatenar `String` con `+=`** | crea un objeto por vuelta → O(n²). Usá `StringBuilder` o `String.join` |
| **Trabajo invariante dentro** | `l.matches(REGEX)` **compila el regex en cada vuelta**: sacá el `Pattern` fuera |
| **Consultas dentro del bucle (N+1)** | una llamada a base de datos o red por elemento. Batching o `JOIN` |
| **Bucle anidado sobre la misma colección** | O(n²) donde un `HashMap` da O(n) |
| **Espera activa** (`while (cola.isEmpty()) {}`) | quema un núcleo entero. Usá una operación bloqueante |
| **Recorrer una matriz por columnas** | desperdicia la línea de caché sin ganar nada |

## Bucles y streams

```java
lista.forEach(x -> { });     // NO es un bucle: es una llamada a método
```

Por eso **`break` y `continue` no compilan** dentro de una lambda, y un `return` solo sale de la lambda, no del método. (Algunas fuentes dicen que `break` "lanza una excepción" ahí: es falso, es un **error de compilación**.)

**Iteración externa** (el bucle: vos controlás el avance) vs **interna** (el stream: la biblioteca controla el avance y vos decís qué hacer).

| Quiero | Stream |
|---|---|
| Transformar una colección en otra | `stream().map(...).toList()` |
| Buscar el primero que cumple | `filter(...).findFirst()` |
| ¿Alguno / todos / ninguno? | `anyMatch` / `allMatch` / `noneMatch` |
| Sumar, contar, máximo | `mapToInt(...).sum()`, `count()`, `max()` |
| Agrupar por clave | `Collectors.groupingBy(...)` |
| Unir textos | `String.join` / `Collectors.joining` |
| Borrar los que cumplen | `removeIf` |
| Transformar en sitio | `replaceAll` / `Arrays.setAll` |

**Salir antes de tiempo en un stream:** `findFirst`, `anyMatch`, `limit`, `takeWhile`. Nunca lanzar una excepción para simular un `break` — si necesitás salir de verdad, usá un bucle.

**El bucle sigue ganando** cuando hay índices, dos colecciones en paralelo, mucho estado mutable, o simplemente cuando se lee mejor.

## Anti-patrones (16, condensados)

| ✗ MAL | ✔ BIEN |
|---|---|
| `for (...);` con `;` | quitarlo |
| `i <= v.length` | `i < v.length` |
| Modificar `i` dentro del cuerpo | solo en la cabecera |
| `continue` antes del avance en un `while` | mover el avance, o usar `for` |
| Contador que puede desbordar | tipo adecuado y condición con `<` |
| `!=` como condición de parada | `<` o `>` |
| Contador `double` | contador `int` y derivar el decimal |
| Modificar la colección en un `for-each` | `removeIf` / `Iterator.remove` |
| `get(i)` sobre lista enlazada | `for-each` |
| `r += s` dentro del bucle | `StringBuilder` / `String.join` |
| Trabajo invariante dentro | sacarlo fuera |
| Consultas dentro del bucle | batching |
| `keySet()` + `get()` | `entrySet()` |
| `break` en un `switch` creyendo salir del bucle | etiqueta, o extraer método |
| Excepción para simular `break` en `forEach` | un bucle de verdad |
| `while (cola.isEmpty()) {}` | operación bloqueante (`take()`) |

## Checklist

- [ ] ¿Hay algún `;` justo después de la cabecera?
- [ ] ¿La condición usa `<` o `>` en vez de `!=`?
- [ ] Con índices, ¿usa `< length` y no `<= length`?
- [ ] ¿El contador puede desbordar su tipo?
- [ ] ¿El contador es entero y no `double`?
- [ ] ¿La variable de control se modifica solo en la cabecera?
- [ ] En bucles anidados, ¿el contador interno se reinicia?
- [ ] ¿El bucle termina siempre, para **cualquier** entrada posible?
- [ ] ¿Se modifica la colección mientras se recorre?
- [ ] Si hay que borrar, ¿usa `removeIf` o `Iterator.remove`?
- [ ] ¿Hay algún `get(i)` que podría ser O(n)?
- [ ] ¿Hay concatenación de `String` con `+=` dentro?
- [ ] ¿Hay trabajo invariante que debería estar fuera?
- [ ] ¿Hay consultas o llamadas remotas dentro?
- [ ] ¿El anidamiento pasa de dos niveles? ¿El cuerpo cabe en la pantalla?
- [ ] ¿Existe un método de biblioteca que haga esto por vos?

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **`while` vs `do-while`.** El segundo comprueba al final: se ejecuta **al menos una vez**.
- **¿En qué se compila el `for-each`?** En un bucle con `Iterator` (sobre arrays, en un `for` con índice).
- **¿Por qué no puedo modificar el elemento desde un `for-each`?** La variable es una **copia** de la referencia; asignarle algo no toca la colección.
- **¿Qué es `ConcurrentModificationException`?** La detección *fail-fast* por `modCount` al modificar una colección mientras se itera. **No está garantizada**: hay casos que no la lanzan.
- **¿Cómo borro elementos mientras recorro?** `removeIf`, `Iterator.remove`, o iterar sobre una copia.
- **¿Por qué `break` no funciona en un `forEach`?** Porque `forEach` no es un bucle: es una llamada a método con una lambda.
- **¿Cómo salgo antes de tiempo de un stream?** `findFirst`, `anyMatch`, `limit`, `takeWhile`.
- **¿Por qué recorrer una `LinkedList` con `get(i)` es O(n²)?** Cada `get` recorre desde el principio.
- **¿Cómo se recorre bien un `Map`?** `entrySet()`, no `keySet()` + `get()`.
- **¿`for (;;)` o `while (true)`?** Equivalentes; lo importante es que la salida esté clara.

</details>
