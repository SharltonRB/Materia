# Cheat Sheet · Type Casting

> Repaso rápido de [`Pura/Java/01-Basics/05-type-casting.md`](../../../Pura/Java/01-Basics/05-type-casting.md) · Java 17+

## En 30 segundos

- **Conversión y cast no son lo mismo.** La conversión es el cambio de tipo; el cast `(Tipo) x` es solo *una de las formas* de pedirla. Muchas conversiones ocurren sin escribir ningún cast.
- Entre **primitivos** el cast **transforma bits**; entre **referencias** no transforma nada: solo cambia lo que el compilador cree y añade una comprobación en runtime.
- **Widening** es automático (pero puede perder precisión). **Narrowing** es manual y **desborda en silencio**.
- Un cast escrito solo para que el compilador se calle es deuda técnica. Si no podés explicar por qué el valor siempre cabe, validá.

## Los dos mundos

| | Entre primitivos | Entre referencias |
|---|---|---|
| Qué hace | **transforma los bits** | no transforma nada |
| Cuándo falla | nunca lanza: desborda en silencio | `ClassCastException` en runtime |
| Bytecode | `i2b`, `d2i`, `i2l`… | `checkcast` (solo el downcast) |
| Riesgo | pérdida de datos silenciosa | excepción ruidosa |

## Widening (automático)

```
byte → short → int → long → float → double
```

**El mito de la cadena única:** muchos tutoriales enseñan `byte → short → char → int`. **El eslabón `short → char` es falso**: `char` es sin signo (0…65535) y `short` con signo (−32768…32767); ninguno contiene al otro, así que ambas direcciones son narrowing y requieren cast.

**Widening que sí pierde datos** — tres casos, porque la mantisa es finita:

```java
(int)(float) 16777217      // 16777216  ← int → float pierde
// también: long → float y long → double
```

## Narrowing (manual)

```java
(int) 9.99         // 9    ← trunca hacia cero, NO redondea
(int) -3.7         // -3   ← hacia cero
Math.round(-3.7)   // -4   ← al más cercano
Math.floor(-3.7)   // -4.0 ← hacia abajo, y devuelve double
(byte) 130         // -126  😱 sin error, sin aviso
(int)(0.0/0.0)     // 0    ← NaN a entero da 0 por especificación (JLS 5.1.3)
```

Para positivos, el cast y `floor` coinciden; para negativos, no.

**Narrowing seguro:**

```java
Math.toIntExact(valorLong);      // lanza ArithmeticException si no cabe
(int) Math.round(d);             // si querías redondear
```

## Promoción numérica: la trampa invisible

**Todo tipo menor que `int` se promociona a `int` antes de operar.**

```java
byte a = 10, b = 20;
byte c = a + b;             // ✗ "possible lossy conversion from int to byte"
byte c = (byte)(a + b);     // ✔
```

Y la asignación compuesta **lleva un cast implícito**:

```java
byte b = 10;
b += 300;        // ✔ compila: equivale a b = (byte)(b + 300) → 54
b = b + 300;     // ✗ no compila
```

**Castear el operando, no el resultado:**

```java
double promedio = (double) (suma / cantidad);   // ✗ ya se perdió todo
double promedio = (double) suma / cantidad;     // ✔
```

## Los tres casos donde el compilador afloja

Asignar un literal `int` constante a un tipo menor **sin cast** es legal si el valor cabe:

```java
byte b = 100;      // ✔ constante de compilación que cabe
byte b = 130;      // ✗ no cabe
int x = 100;
byte b = x;        // ✗ x no es constante
```

## Upcast y downcast

```java
Object o = "hola";                    // upcast: implícito, siempre seguro
String s = (String) o;                // downcast: explícito, puede fallar
```

**El upcast no existe en runtime**: no genera ninguna instrucción, el bytecode es un simple `aload`. El downcast sí genera **`checkcast`**.

**Tipo estático** (lo que el compilador sabe) vs **tipo dinámico** (lo que el objeto realmente es). El cast solo cambia el primero.

**Downcast seguro, con pattern matching (Java 16+):**

```java
if (obj instanceof String s) { s.length(); }      // ✔ comprueba, castea y enlaza
```

> Si escribís tres `instanceof` seguidos, el problema es el **diseño**, no el casting. Buscá polimorfismo, o un `switch` con patterns sobre una `sealed interface`.

## Wrappers: los puzzles del autoboxing

```java
Integer a = 100, b = 100;   a == b;   // true   ← caché −128…127
Integer c = 1000, d = 1000; c == d;   // false  ← objetos distintos

Long l = 1;    // ✗ no compila: haría falta widening (int→long) SEGUIDA de boxing,
Long l = 1L;   // ✔ y esa combinación no está permitida
```

**Ensanchamiento y boxing no se combinan.** Es la regla que explica media docena de errores de compilación desconcertantes.

**Y el clásico de las listas:**

```java
List<Integer> lista = ...;
lista.remove(1);                      // remove(int index): borra la POSICIÓN 1
lista.remove(Integer.valueOf(1));     // remove(Object): borra el VALOR 1
```

La sobrecarga sin boxing gana siempre.

## Genéricos: erasure y heap pollution

Los genéricos se borran al compilar (**erasure**), así que **el cast reaparece** en el bytecode donde vos no lo escribiste. De ahí:

```java
(String[]) lista.toArray()     // ✗ ClassCastException: toArray() crea un Object[]
lista.toArray(new String[0])   // ✔
```

**Unchecked cast:** el compilador avisa de que no puede verificarlo. `@SuppressWarnings` en clases enteras suprime el aviso de hoy **y todos los futuros**, incluidos los que sí importan.

**Covarianza de arrays** — el agujero que Java dejó abierto:

```java
Object[] objetos = new String[3];
objetos[0] = 42;                 // 💥 ArrayStoreException en runtime
```

Los arrays son covariantes; los genéricos no. Por eso `List<Object> l = new ArrayList<String>()` no compila, y eso es lo correcto.

## Lo que parece casting y no lo es

| Escribís | Qué es realmente |
|---|---|
| `Integer.parseInt("42")` | **parsing**, no casting: interpreta texto |
| `String.valueOf(42)` | formateo a texto |
| `(String) objeto` | casting real (de referencia) |
| `Character.getNumericValue(c)` | conversión semántica |
| `c - '0'` | aritmética sobre el código del carácter |

## Anti-patrones

1. **Castear para silenciar al compilador** sin razonar sobre el rango.
2. **Cadenas de `instanceof`** — es un problema de diseño.
3. **Comparar wrappers con `==`** — funciona en los tests (edades < 128) y falla con datos reales.
4. **`double` para dinero** — `(int)(0.29 * 100)` da **28**, no 29: el céntimo desaparece con un literal que parece inocente. (`4.35` da 434; `2.00 - 1.10` da `0.8999999999999999`.)
5. **Castear el resultado en vez del operando.**
6. **`@SuppressWarnings` en clases enteras.**

## Checklist antes de escribir un cast

- [ ] ¿Es entre primitivos o entre referencias? Son dos mundos con reglas opuestas.
- [ ] Si es narrowing primitivo: **¿el valor cabe siempre?** Si no podés demostrarlo, validá o usá `Math.toIntExact`.
- [ ] Si es de flotante a entero: **¿quiero truncar o redondear?** Son cosas distintas.
- [ ] Si es un downcast: ¿hay un `instanceof` antes? ¿Y por qué no es polimorfismo?
- [ ] Si es unchecked: ¿puedo demostrar que es seguro? Si no, no lo suprimas.
- [ ] Si hay wrappers: ¿puede ser `null`? El unboxing de `null` es NPE.
- [ ] En una expresión aritmética: ¿el cast está **antes** de operar? Casi siempre es lo correcto.

## Tabla de decisión

| Situación | Solución |
|---|---|
| `int` → `double` para dividir | `(double) a / b` (antes de dividir) |
| `double` → `int` truncando | `(int) d` |
| `double` → `int` redondeando | `(int) Math.round(d)` |
| `long` → `int` con dato externo | `Math.toIntExact(l)` |
| `String` → `int` | `Integer.parseInt(s.trim())` en `try/catch` |
| `int` → `String` | `String.valueOf(n)` |
| `char` dígito → `int` | `Character.getNumericValue(c)`, o `c - '0'` si ya validaste |
| `byte` → valor 0…255 | `b & 0xFF` |
| Supertipo → subtipo | `if (x instanceof Sub s)` |
| Varios subtipos posibles | `switch` con patterns sobre una `sealed interface` |
| `Object` a genérico | `Class<T>` + `tipo.cast(x)` |
| **Dinero** | `BigDecimal` o `long` de centavos; nunca `double` |
| Multiplicación que puede desbordar | castear el primer factor a `long`, o `Math.multiplyExact` |

## Autoevaluación

<details><summary><strong>Respuestas</strong></summary>

1. **`(byte) 130`** → `-126`. Los 8 bits `10000010` leídos en complemento a dos.
2. **`byte c = a + b;`** no compila por la promoción numérica a `int`.
3. **`(int) -3.7` = −3** (trunca), **`Math.round(-3.7)` = −4**, **`Math.floor(-3.7)` = −4.0**.
4. **`short s = miChar;`** no es válida: `char` es sin signo y `short` con signo, ninguno contiene al otro.
5. **¿Puede el widening perder datos?** Sí: `int`→`float`, `long`→`float`, `long`→`double`.
6. **`Integer 100 == 100`** → `true` (caché); **con 1000** → `false`.
7. **`b += 300` compila y `b = b + 300` no** porque la asignación compuesta incluye un cast implícito.
8. **¿Qué bytecode genera un upcast?** Ninguno. El downcast genera `checkcast`.
9. **`(int)(0.0 / 0.0)`** → `0`, por especificación, sin excepción ni aviso.
10. **`Long l = 1;`** no compila: widening + boxing no se combinan.
11. **`remove(1)` vs `remove(Integer.valueOf(1))`**: posición vs valor.
12. **`(String[]) lista.toArray()`** lanza `ClassCastException`: el array real es `Object[]`. Usá `toArray(new String[0])`.

</details>
