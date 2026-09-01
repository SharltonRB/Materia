# Cheat Sheet · Logical, Relational and Bitwise Operators

> Repaso rápido de [`Pura/Java/01-Basics/08-logical-relational-bitwise-operators.md`](../../../Pura/Java/01-Basics/08-logical-relational-bitwise-operators.md) · Java 17+

## En 30 segundos

- En Java **`boolean` no es un número**: no hay `if (1)`, y por eso el clásico `if (x = 0)` de C no compila… salvo con booleanos, donde sí compila y es un bug.
- **El cortocircuito de `&&` y `||` es semántica, no una optimización**: es lo que hace que la guarda de `null` funcione.
- `==` sobre referencias compara **identidad**, no contenido. Con wrappers falla a partir de 128.
- Los operadores bitwise **promueven a `int`**: de ahí que la máscara `& 0xFF` sea obligatoria con `byte`.
- Un comparador escrito como resta (`a - b`) desborda y desordena en silencio.

## Relacionales

| Operador | Sobre qué |
|---|---|
| `<` `>` `<=` `>=` | solo tipos numéricos (y `char`) |
| `==` `!=` | numéricos, `boolean` y **referencias** |

**El compilador te salva** cuando los tipos son incomparables: `if ("a" == 1)` no compila. Pero no te salva con `Object`.

### Decimales

```java
Double.isNaN(x)              // ✔ NaN no es igual ni a sí mismo
0.0 == -0.0                  // true
Double.compare(0.0, -0.0)    // 1  ← compare y equals SÍ los distinguen
```

**Toda comparación de orden con `NaN` da `false`.** Por eso una validación en negativo acepta `NaN`: hay que escribirlas en positivo.

### Referencias: identidad vs igualdad

```java
if (nombre == "admin")                 // ✗ identidad
if ("admin".equals(nombre))            // ✔ contenido, y null-safe
if (Objects.equals(a, b))              // ✔ null-safe en ambos lados
if (Arrays.equals(v1, v2))             // ✔ arrays: equals NO compara contenido
```

**La caché de wrappers:** `Integer.valueOf` reutiliza instancias de −128 a 127 (`Character` de 0 a 127; `Boolean` siempre). Fuera de ese rango, `==` da `false`. Y **el límite superior se puede mover con `-XX:AutoBoxCacheMax`**: el mismo código puede dar respuestas distintas en dos máquinas.

**Cuándo se desempaqueta un wrapper:** con `<`, `>`, `<=`, `>=` **siempre** (y por eso un `null` ahí es NPE); con `==` y `!=` **solo si el otro operando es primitivo**.

### Ordenar: el comparador roto por resta

```java
lista.sort((a, b) -> a.getEdad() - b.getEdad());        // ✗ desborda con valores extremos
lista.sort(Comparator.comparingInt(Persona::getEdad));  // ✔
```

## Lógicos

| Operador | Cortocircuita | Cuándo usarlo |
|---|---|---|
| `&&` | **sí** | por defecto, siempre |
| `\|\|` | **sí** | por defecto, siempre |
| `&` | no | cuando querés **ambos** efectos secundarios (y lo comentás) |
| `\|` | no | ídem |
| `^` (XOR) | no | "exactamente uno de los dos" |
| `!` | — | negación |

```java
if (u != null && u.estaActivo())    // ✔ el && es lo que evita el NPE
if (u != null &  u.estaActivo())    // 💥 evalúa las dos: NPE
```

**El cortocircuito no es una optimización, es semántica.** Cambiar `&&` por `&` "porque da igual" cambia el significado del programa.

**Cuándo sí querés el que no cortocircuita:** validar un formulario entero para mostrar *todos* los errores, no solo el primero.

### De Morgan

```
!(a && b)  ≡  !a || !b
!(a || b)  ≡  !a && !b
```

El error clásico es negar cambiando solo los operandos:

```java
if (!estaLogueado && !tienePermiso) denegar();    // ✗ solo si fallan las DOS
if (!(estaLogueado && tienePermiso)) denegar();   // ✔ si falla cualquiera
```

### Precedencia (de más a menos)

```
!  →  aritméticos  →  <  >  <=  >=  →  ==  !=  →  &  →  ^  →  |  →  &&  →  ||  →  ?:  →  =
```

Lo que hay que recordar: **`&` `^` `|` tienen MENOS precedencia que `==`**, herencia de C que sorprende siempre. `a ^ b | c` y `a ^ (b | c)` dan resultados opuestos: **poné paréntesis**.

### Redundancias que delatan

```java
if (flag == true)       // ✗ redundante, y abre la puerta a if (flag = true), que compila
if (flag)               // ✔
if (edad >= 18) return true; else return false;   // ✗
return edad >= 18;                                 // ✔
```

## El ternario

```java
condicion ? valorSiTrue : valorSiFalse
```

**Tiene tipo, y ese tipo puede lanzar una excepción:**

```java
condicion ? 1 : 2.0                  // siempre double, aunque tome la rama del 1
int n = flag ? mapa.get(k) : 0;      // 💥 NPE si get devuelve null y flag es true
int n = Objects.requireNonNullElse(mapa.get(k), 0);   // ✔
```

Ese NPE es especialmente traicionero: la promoción numérica obliga a desempaquetar el wrapper, y **solo falla cuando la condición cae de ese lado**.

**Usalo** para elegir entre **dos valores del mismo tipo**, en una línea. **No lo uses** anidado: `a ? (b ? x : y) : (c ? z : w)` es un árbol, no una tabla — eso es un `switch` o un `Map`.

## Bitwise

Modelo mental: 32 bits (o 64) en **complemento a dos**. El bit más alto es el signo.

| Operador | Qué hace |
|---|---|
| `&` | AND bit a bit — **consultar** y limpiar |
| `\|` | OR — **poner** flags |
| `^` | XOR — **alternar** |
| `~` | complemento (invierte todos los bits). **No es `!`** |
| `<<` | desplazar a la izquierda |
| `>>` | desplazar a la derecha **con signo** (rellena con el bit de signo) |
| `>>>` | desplazar a la derecha **sin signo** (rellena con ceros) |

```java
~5                     // -6, porque ~x == -x - 1
-8 >> 1                // -4   ✔ conserva el signo
-8 >>> 1               // 2147483644  ← rellena con ceros
1 << 33                // 2, NO 0: la distancia se toma módulo 32
1L << 33               // ✔ 8589934592 — si el destino es long, escribí 1L
```

### Las dos trampas más caras

```java
byte b = -1;
if (b == 0xFF) { }                     // ✗ SIEMPRE false: -1 no es 255
if ((b & 0xFF) == 0xFF) { }            // ✔ la máscara es obligatoria
if (Byte.toUnsignedInt(b) == 0xFF) { } // ✔ más legible
```

La causa: los operandos se **promueven a `int`** antes de operar, y `-1` se extiende con signo.

```java
if ((flags & LECTURA) == 1)     // ✗ solo funciona para el primer flag
if ((flags & LECTURA) != 0)     // ✔
```

### Flags: poner, quitar, alternar, consultar

```java
flags |=  MASCARA;    // poner
flags &= ~MASCARA;    // quitar
flags ^=  MASCARA;    // alternar
boolean tiene = (flags & MASCARA) != 0;   // consultar
```

**Pero casi siempre gana `EnumSet`:** se pierde medio nanosegundo y se gana tipado, logs legibles y la imposibilidad de equivocarse con la máscara. Internamente `EnumSet` ya es un bitfield. Para millones de banderas, `BitSet`.

### Utilidades que ya existen

`Integer.bitCount`, `highestOneBit`, `numberOfTrailingZeros`, `reverse`, `rotateLeft`, `toBinaryString`, `compareUnsigned`, `divideUnsigned`, `toUnsignedLong`. **Son intrínsecos del JIT**: escribir el bucle a mano es más lento y menos claro.

### Mitos

- **`>>` no divide por 2ⁿ.** Falso con negativos: `-7 >> 1` es `-4`, no `-3`. Lo que equivale es `Math.floorDiv`.
- **`>>>` no siempre devuelve un positivo.** Con distancia 0 (o múltiplo de 32) devuelve el valor intacto, signo incluido.
- **Reescribir aritmética como desplazamientos "para optimizar"** es inútil: el JIT ya lo hace, y mejor.
- **Intercambiar con XOR** falla cuando son la misma variable, y no es más rápido que una temporal.
- **"Cifrar" con XOR** contra una clave fija no es cifrado.

## Anti-patrones (24, condensados)

| ✗ MAL | ✔ BIEN |
|---|---|
| `nombreUsuario == "admin"` | `"admin".equals(nombreUsuario)` |
| `==` entre dos wrappers | `equals`, o pasar a primitivos |
| `equals` / `Objects.equals` sobre arrays | `Arrays.equals` / `deepEquals` |
| `(a, b) -> a.getEdad() - b.getEdad()` | `Comparator.comparingInt(...)` |
| Validar rango en negativo sobre `double` | escribirlo en positivo (si no, acepta `NaN`) |
| Cambiar `&&` por `&` | el cortocircuito es lo que evita el NPE |
| Efectos secundarios en el lado derecho de `&&` | sacarlos fuera |
| `a ^ b \| c` sin paréntesis | ponerlos |
| `if (flag == true)` | `if (flag)` |
| `condicion ? 1 : 2.0` | mismo tipo en ambas ramas |
| Ternario con primitivo + wrapper nullable | `requireNonNullElse` |
| Ternarios anidados | `switch` o `Map` |
| `datos[0] == 0xFF` | `Byte.toUnsignedInt(datos[0]) == 0xFF` |
| `(flags & M) == 1` | `!= 0` o `== M` |
| Acumular bits en `byte`/`short` | `int` o `long` (el cast oculto trunca) |
| `1 << n` con destino `long` | `1L << n` |
| Comparar secretos con `equals` | `MessageDigest.isEqual` |
| Flags con `int` donde cabía un `EnumSet` | `EnumSet` |

## Tablas de decisión

| Quiero comparar | Uso | No uso |
|---|---|---|
| Dos primitivos numéricos | `==`, `<`, `>` | — |
| Dos `double` | `Double.compare` o tolerancia | `==` |
| Dos objetos por contenido | `Objects.equals` | `==` |
| Dos wrappers | `equals`, o primitivos | `==` |
| Dos enums | `==` | `equals` (funciona, pero sobra) |
| Dos arrays | `Arrays.equals` | `equals` |
| Dos `BigDecimal` como importes | `compareTo(x) == 0` | `equals` |
| Con `null` | `== null` | `equals(null)` |
| Un secreto | `MessageDigest.isEqual` | `equals` |
| Para ordenar | `Comparator.comparingInt` | `(a, b) -> a - b` |
| El tipo de un objeto | `instanceof Tipo t` | `getClass() == ...` |

| Quiero | Uso | No uso |
|---|---|---|
| Combinar condiciones con guarda | `&&`, `\|\|` | `&`, `\|` |
| Evaluar ambos lados a propósito | `&`, `\|` **con comentario** | `&&` "arreglado" |
| Exactamente uno de dos | `^` o `!=` | dos `if` anidados |
| Elegir entre dos valores | ternario | `if` con variable mutable |
| Elegir entre muchos casos de un enum | `switch` como expresión | cadena de ternarios |
| Guardar muchos booleanos | `EnumSet` | flags con `int` |
| Guardar millones de booleanos | `BitSet` | `boolean[]` |
| Leer un byte sin signo | `Byte.toUnsignedInt` | `b & 0xFF` sin explicación |
| Dividir por una potencia de dos | `/` o `Math.floorDiv` | `>>` |
| Contar bits | `Integer.bitCount` | bucle a mano |
| Intercambiar dos variables | variable temporal | XOR |

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **`&&` vs `&`.** El primero cortocircuita: si el izquierdo decide el resultado, el derecho **no se evalúa**. Es semántica, no optimización.
- **¿Por qué `if (x = 0)` no compila en Java pero sí en C?** Porque `boolean` no es un número. Con booleanos **sí** compila: `if (flag = true)` siempre entra.
- **¿Por qué `Integer 128 == 128` es `false`?** Fuera de la caché de `valueOf` (−128…127) son objetos distintos.
- **¿Cuándo se desempaqueta un wrapper?** Siempre con `<`, `>`, `<=`, `>=`; con `==` solo si el otro operando es primitivo.
- **¿Qué hace `~5`?** −6: `~x == -x - 1`.
- **`>>` vs `>>>`.** Con signo vs con ceros. `>>` **no** equivale a dividir por 2ⁿ con negativos.
- **¿Por qué `1 << 33` da 2?** La distancia se toma módulo 32 en `int` (módulo 64 en `long`).
- **¿Por qué `datos[0] == 0xFF` es siempre `false`?** `byte` es con signo y se promueve a `int` extendiendo el signo: hace falta `& 0xFF`.
- **¿Por qué un comparador por resta está mal?** Desborda con valores extremos y produce un orden incorrecto en silencio.
- **¿Qué tipo tiene `cond ? 1 : 2.0`?** `double`, siempre — la promoción numérica actúa sobre ambas ramas.

</details>
