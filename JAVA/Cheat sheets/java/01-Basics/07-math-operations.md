# Cheat Sheet · Math Operations

> Repaso rápido de [`Pura/Java/01-Basics/07-math-operations.md`](../../../Pura/Java/01-Basics/07-math-operations.md) · Java 17+

## En 30 segundos

- Hay **dos aritméticas distintas** en Java: la entera (exacta, pero desborda en silencio) y la de punto flotante (aproximada, pero nunca lanza).
- **El tipo del resultado no es el que creés**: la promoción numérica decide antes de operar.
- El desbordamiento entero **no lanza excepción**. La familia `Math.*Exact` sí.
- **`double` no sirve para dinero.** Nunca. `BigDecimal` o `long` de centavos.
- `%` **no es el módulo matemático**: con negativos da negativo. El módulo es `Math.floorMod`.

## Promoción numérica

**Regla:** si algún operando es `double` → todo a `double`; si no, `float`; si no, `long`; **si no, todo a `int`** (incluidos `byte`, `short` y `char`).

```java
byte a = 10, b = 20;
byte c = a + b;             // ✗ el resultado de a+b es int
int  x = 1000 * 60 * 60 * 24 * 365;   // 1471228928 ← desborda ANTES de asignar
long y = 1000L * 60 * 60 * 24 * 365;  // 31536000000 ✔ la L al principio basta
```

Y la asignación compuesta **lleva un cast implícito**: `b += 300` compila (y trunca), `b = b + 300` no.

## División y resto

```java
7 / 2         // 3     ← división ENTERA: se trunca hacia cero
7 / 2.0       // 3.5   ← basta que un operando sea decimal
(double) aciertos / total     // ✔ castear ANTES de dividir
(double) (aciertos / total)   // ✗ ya se perdió todo

-7 / 2        // -3    ← trunca hacia cero, no hacia abajo
-7 % 2        // -1    ← el signo lo pone el DIVIDENDO
Math.floorDiv(-7, 2)   // -4  ← redondea hacia abajo
Math.floorMod(-7, 2)   //  1  ← el módulo matemático, siempre positivo
```

| Con enteros | Con decimales |
|---|---|
| `1 / 0` → **`ArithmeticException`** | `1.0 / 0.0` → **`Infinity`**, sin excepción |
| `0 / 0` → `ArithmeticException` | `0.0 / 0.0` → **`NaN`**, sin excepción |

Con decimales el fallo es **silencioso y se propaga** por todo el cálculo.

## Desbordamiento silencioso

```java
Integer.MAX_VALUE + 1        // -2147483648  😱 sin aviso
Math.abs(Integer.MIN_VALUE)  // -2147483648  😱 ¡un valor absoluto negativo!
```

`Math.abs(MIN_VALUE)` es negativo porque el rango es asimétrico: hay un valor negativo más que positivos. De ahí el anti-patrón `Math.abs(hash) % n`, que puede dar índice negativo.

**La familia `Exact`, desde Java 8** — lanza `ArithmeticException` en vez de dar la vuelta:

```java
Math.addExact, subtractExact, multiplyExact, incrementExact,
negateExact, absExact, toIntExact
```

## IEEE 754 y por qué `0.1 + 0.2 != 0.3`

`double` = 1 bit de signo + 11 de exponente + 52 de mantisa. Solo se representan exactamente las fracciones cuyo denominador es potencia de 2. El 0.1 no lo es, así que se guarda redondeado y el error se acumula.

```java
0.1 + 0.2                    // 0.30000000000000004
1.0 - 0.9 == 0.1             // false
double t = 0; for (int i=0;i<100;i++) t += 0.01;   // 1.0000000000000007
```

### Los valores especiales

```java
Double.isNaN(x)              // ✔ la ÚNICA forma correcta: NaN != NaN, ni consigo mismo
0.0 == -0.0                  // true
Double.compare(0.0, -0.0)    // 1  ← compare y equals SÍ los distinguen
```

`NaN` envenena todo: cualquier operación con `NaN` da `NaN`, y **toda comparación con `<`, `>`, `<=`, `>=` da `false`**. Por eso una validación escrita en negativo, `!(v < min || v > max)`, **acepta `NaN`**.

### Comparar decimales

```java
if (a == b)                              // ✗
if (Math.abs(a - b) < TOLERANCIA)        // ✔ double
if (bd.compareTo(BigDecimal.ZERO) == 0)  // ✔ BigDecimal
```

**`float` casi nunca:** 6-7 dígitos significativos frente a 15-16. El ahorro son 4 bytes; solo compensa en millones de valores.

## `Math`: lo que hay que conocer

| Familia | Métodos |
|---|---|
| Básicos | `abs`, `max`, `min`, `signum`, `copySign`, **`clamp` (Java 21+)** |
| Redondeo | `round` (al más cercano, empate hacia arriba), `floor`, `ceil`, `rint` (al par más cercano) |
| Potencias | `pow`, `sqrt`, `cbrt`, `exp`, `log`, `log10`, **`hypot`** (sin desbordar) |
| División | `floorDiv`, `floorMod`, **`ceilDiv` (Java 18+)** |
| Seguros | `addExact`, `multiplyExact`, `toIntExact`, `absExact` |
| Precisión | `fma`, `ulp`, `nextUp`, `nextDown` |

`Math` vs `StrictMath`: `Math` puede usar intrínsecos del procesador (más rápido, resultados que pueden variar un ulp entre plataformas); `StrictMath` garantiza resultados **idénticos en todas las máquinas**. Si necesitás reproducibilidad exacta, `StrictMath`.

**`strictfp` no hace nada desde Java 17** (JEP 306): ahora todo es estricto por defecto. Verlo en código nuevo es una reliquia.

## Redondeo: el que todo el mundo escribe mal

```java
Math.round(x * 100) / 100.0     // ✗ falla con valores como 260.775 y desborda con exponentes grandes
new BigDecimal(String.valueOf(x)).setScale(2, RoundingMode.HALF_UP)   // ✔
```

**`RoundingMode`** tiene ocho estrategias. Las dos que importan: **`HALF_UP`** (lo que enseñan en el colegio) y **`HALF_EVEN`** (*banker's rounding*, el que exige la contabilidad porque no sesga al alza en grandes volúmenes). Escribí cuál elegiste y por qué.

Y ojo: **dos APIs de la JDK pueden redondear el mismo número de forma distinta**, según el modo y el locale que apliquen por defecto. No des por hecho que `Math.round`, `String.format` y `BigDecimal` coinciden.

## Dinero

```java
new BigDecimal(0.1)          // ✗ 0.1000000000000000055511... ya venía roto
new BigDecimal("0.1")        // ✔ exacto
BigDecimal.valueOf(0.1)      // ✔ pasa por String internamente

bd1.equals(bd2)              // ✗ false para 1.0 vs 1.00 (compara también la escala)
bd1.compareTo(bd2) == 0      // ✔

total.add(x);                // ✗ es INMUTABLE: no hace nada
total = total.add(x);        // ✔

a.divide(b)                  // ✗ ArithmeticException si el decimal es periódico
a.divide(b, 2, RoundingMode.HALF_UP)   // ✔
```

**La alternativa más simple y rápida:** enteros de la unidad mínima (`long centimos = 1999`). Exacto y a velocidad de primitivo. Es lo que hace Stripe.

**`BigInteger`** para enteros sin límite: criptografía, combinatoria, factoriales grandes.

## Aleatoriedad

```java
Math.abs(random.nextInt()) % n   // ✗ sesga la distribución y puede dar negativo
random.nextInt(n)                // ✔
random.nextInt(min, max)         // ✔ Java 17+

new Random()                     // predecible POR DISEÑO: nunca para seguridad
SecureRandom                     // ✔ tokens, contraseñas, IDs de sesión
ThreadLocalRandom.current()      // ✔ si hay varios hilos (Random se contiende)
RandomGenerator                  // interfaz moderna (Java 17+), varios algoritmos
```

Una semilla fija hace la secuencia **reproducible**, lo que es excelente para tests y catastrófico para seguridad.

## Los 18 anti-patrones, condensados

| # | ✗ MAL | ✔ BIEN |
|---|---|---|
| 1 | `long ms = 1000 * 60 * 60 * 24 * 365;` | `1000L * ...`, o `Duration.ofDays(365)` |
| 2 | `double r = aciertos / total;` | `(double) aciertos / total` |
| 3 | `double` para dinero | `BigDecimal` o `long` de centavos |
| 4 | `Math.abs(clave.hashCode()) % n` | `Math.floorMod(clave.hashCode(), n)` |
| 5 | `if (total == 0.0)` | tolerancia, o `compareTo` |
| 6 | `BigDecimal.equals` para importes | `compareTo` |
| 7 | `new BigDecimal(double)` | constructor de `String` o `valueOf` |
| 8 | `divide` sin `RoundingMode` | añadirlo siempre |
| 9 | `total.add(x);` sin asignar | `total = total.add(x);` |
| 10 | `for (double d=0; d!=1.0; d+=0.1)` | contador `int`, y derivar el decimal |
| 11 | `Math.round(x*100)/100.0` | `BigDecimal.setScale` |
| 12 | `String.format("%.2f", x)` sin locale | `Locale.ROOT` si lo consume una máquina |
| 13 | `Math.abs(nextInt()) % n` | `nextInt(n)` |
| 14 | `Random` para tokens | `SecureRandom` |
| 15 | `i = i++` | no hace nada; y `a[i++] = i++` es ilegible |
| 16 | `float` "para ahorrar memoria" | 4 bytes a cambio de la mitad de dígitos |
| 17 | `strictfp` en código nuevo | no hace nada desde Java 17 |
| 18 | `Math.pow(10, n)` dentro del bucle | `movePointRight` / `setScale` |

## Checklist antes de dar por terminado un cálculo

- [ ] ¿Algún literal `int` en una multiplicación que puede pasar de 2.000 millones? → añadir `L`.
- [ ] ¿División entre enteros donde esperás decimales? → castear **antes**.
- [ ] ¿Es dinero? → `BigDecimal` o `long` de centavos.
- [ ] ¿Un `%` cuyo operando izquierdo puede ser negativo? → `Math.floorMod`.
- [ ] ¿Un `Math.abs` sobre un valor que podría ser `MIN_VALUE`? → `floorMod`, máscara o `absExact`.
- [ ] ¿Se compara algún `double` con `==`? → tolerancia o `Double.compare`.
- [ ] ¿Puede llegar `NaN`? → `Double.isFinite` primero, y validar en positivo.
- [ ] ¿El divisor puede ser cero? → validar (con enteros lanza, con decimales contamina).
- [ ] ¿`BigDecimal.divide` sin `RoundingMode`? → añadirlo.
- [ ] ¿`BigDecimal` comparado con `equals`? → `compareTo`.
- [ ] ¿Algún `String.format` numérico va a un fichero, JSON o log estructurado? → `Locale.ROOT`.
- [ ] ¿El redondeo (`HALF_UP` / `HALF_EVEN`) es el que exige el negocio? → documentarlo.
- [ ] ¿El aleatorio tiene implicaciones de seguridad? → `SecureRandom`. ¿Varios hilos? → `ThreadLocalRandom`.

## Tablas de decisión

| Necesito representar | Tipo | Por qué |
|---|---|---|
| Contadores, índices, IDs pequeños | `int` | rápido y suficiente hasta 2 × 10⁹ |
| Tiempos en ms, bytes, contadores grandes | `long` | `int` desborda antes de lo que creés |
| Dinero con porcentajes o tipos de cambio | `BigDecimal` | exacto en decimal, escala controlada |
| Dinero en alto volumen | `long` de centavos | exacto y a velocidad de primitivo |
| Medidas físicas, estadística, gráficos, ML | `double` | rango y velocidad |
| Millones de valores donde la memoria manda | `float` | la mitad de espacio, la mitad de dígitos |
| Enteros sin límite | `BigInteger` | precisión arbitraria |

| Quiero | Uso | No uso |
|---|---|---|
| Dividir redondeando hacia abajo | `Math.floorDiv` | `/` con negativos |
| Dividir redondeando hacia arriba | `Math.ceilDiv` (18+) | `(n+d-1)/d` |
| Índice a partir de un hash | `Math.floorMod` | `Math.abs(h) % n` |
| Detectar desbordamiento | `Math.addExact` y familia | comprobaciones manuales |
| Acotar a un rango | `Math.clamp` (21+) | `max(min, min(max, v))` |
| Redondear a N decimales | `BigDecimal.setScale` | `Math.round(x*100)/100.0` |
| Redondear al par (contabilidad) | `RoundingMode.HALF_EVEN` | `Math.round` |
| Entero aleatorio en un rango | `nextInt(min, max)` | `Math.abs(nextInt()) % n` |
| Hipotenusa sin desbordar | `Math.hypot` | `sqrt(x*x + y*y)` |
| Resultados idénticos entre plataformas | `StrictMath` | `strictfp` |

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Por qué `0.1 + 0.2 != 0.3`?** IEEE 754 es base 2 y 0.1 es periódico en binario.
- **¿Qué pasa con `Integer.MAX_VALUE + 1`?** Da la vuelta a `MIN_VALUE`, sin excepción. Con `Math.addExact` sí lanza.
- **¿Por qué `Math.abs(Integer.MIN_VALUE)` es negativo?** El rango es asimétrico: no existe el positivo correspondiente.
- **`%` vs `Math.floorMod`.** `%` toma el signo del dividendo; `floorMod` siempre da el signo del divisor.
- **`1/0` vs `1.0/0.0`.** Excepción vs `Infinity`.
- **¿Por qué `NaN != NaN`?** Por definición de IEEE 754. Se comprueba con `Double.isNaN`.
- **¿Por qué `double` no sirve para dinero?** Errores de representación que se acumulan.
- **¿`new BigDecimal(0.1)` o `("0.1")`?** Siempre `String`.
- **¿Por qué `BigDecimal.equals` da `false` entre `1.0` y `1.00`?** Compara también la escala. Usá `compareTo`.
- **¿`HALF_UP` o `HALF_EVEN`?** `HALF_EVEN` en contabilidad: no sesga al alza en grandes volúmenes.
- **¿`Random` o `SecureRandom`?** `Random` es predecible por diseño; para seguridad, `SecureRandom`.

</details>
