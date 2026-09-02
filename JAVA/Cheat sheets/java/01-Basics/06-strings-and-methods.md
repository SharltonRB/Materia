# Cheat Sheet · Strings and Methods

> Repaso rápido de [`Pura/Java/01-Basics/06-strings-and-methods.md`](../../../Pura/Java/01-Basics/06-strings-and-methods.md) · Java 17+

## En 30 segundos

- `String` **no es un primitivo** ni un array de `char`: es una clase `final` que envuelve un `byte[]`.
- **La inmutabilidad es la decisión que explica todo lo demás**: el pool, la seguridad, el thread-safety y por qué toda transformación devuelve un objeto nuevo.
- Comparar con `==` es **el error número uno**, y lo peor es que a veces *parece* funcionar.
- `length()` **no cuenta caracteres**: cuenta unidades de 16 bits. Con emojis, miente.
- Concatenar dentro de un bucle es O(n²).

## Representación interna

Desde Java 9 (**Compact Strings**, JEP 254) `String` guarda un `byte[]` más un indicador de codificación: si todo cabe en Latin-1, usa **1 byte por carácter** en vez de 2. Como las cadenas suelen ser el mayor consumidor de heap, el ahorro es enorme — y es invisible desde tu código.

## Inmutabilidad, literales y el pool

```java
String a = "hola";
String b = "hola";
String c = new String("hola");

a == b;          // true   ← ambos apuntan al mismo objeto del POOL
a == c;          // false  ← new fuerza un objeto nuevo, saltándose el pool
a.equals(c);     // true   ← lo correcto
```

- Los literales viven en el **string pool**, una zona compartida: el compilador reutiliza el mismo objeto.
- `new String("literal")` es un objeto extra sin ningún beneficio. **No lo escribas nunca.**
- `intern()` devuelve la versión del pool. Úsalo solo con motivo medido.
- **Constantes de compilación**: `"a" + "b"` se resuelve al compilar y va al pool; `"a" + variable` no.

**Por qué se eligió inmutable:** permite el pool (ahorro de memoria), hace `String` thread-safe sin sincronizar, permite cachear el `hashCode` (clave de `HashMap` barata) y evita que un valor validado cambie después de validarse.

## Comparar: el error número uno

```java
if (rol == "admin")            // ✗ MAL: compara referencias
if ("admin".equals(rol))       // ✔ BIEN: y además null-safe (la constante va primero)
if (Objects.equals(a, b))      // ✔ null-safe en ambos lados
```

| Necesito | Herramienta |
|---|---|
| ¿Mismo contenido? | `equals()` |
| ¿Mismo contenido, ignorando mayúsculas? | `equalsIgnoreCase()` |
| ¿Cuál va primero? (orden interno) | `compareTo()` — lexicográfico por code unit |
| ¿Cuál va primero? (para el usuario) | `Collator` — orden humano según idioma |
| Ignorar acentos y mayúsculas | `Collator` con fuerza `PRIMARY` |
| ¿Contiene este texto? | `contains()` |
| ¿Cumple este patrón **completo**? | `matches()` |
| ¿Contiene este patrón? | `Pattern` + `find()` |
| Comparar un tramo sin copiar | `regionMatches()` |
| **Comparar secretos** | `MessageDigest.isEqual` (evita el ataque por tiempo) |

`compareTo` ordena por valor de code unit: todas las mayúsculas van antes que las minúsculas, y los acentos quedan detrás de la `z`. Para listas que ve un humano, `Collator`.

## Longitud y contenido

```java
nombre.length() == 0     // funciona, pero menos expresivo
nombre.isEmpty()         // mejor
nombre.isBlank()         // ✔ correcto para validar entrada de usuario (Java 11+)
```

`isBlank()` es `true` para la cadena vacía, para un espacio y para tabuladores o saltos de línea. Es lo que querés casi siempre al validar formularios.

## Transformar: toda operación devuelve un objeto nuevo

```java
texto.trim();          // ✗ MAL: no hace nada, el resultado se tira
texto = texto.trim();  // ✔ BIEN
```

| Operación | Detalle |
|---|---|
| `trim()` vs `strip()` | `trim` corta por debajo de `U+0020`; **`strip` (Java 11+) entiende Unicode** y quita espacios de ancho cero, no-break, etc. Usá `strip` |
| `replace` vs `replaceAll` | **`replace` es literal; `replaceAll` es regex.** `ip.replaceAll(".", "-")` destruye el string entero, porque `.` significa "cualquier carácter" |
| `split()` | descarta los campos vacíos **finales** por defecto. Para datos posicionales (CSV): `split(",", -1)` |
| `toUpperCase()` / `toLowerCase()` | **sin `Locale` se rompen en Turquía**: la `i` mayúscula turca no es `I`. Para comparar, `equalsIgnoreCase`; si hay que transformar, pasá `Locale.ROOT` |
| `substring()` | lanza `StringIndexOutOfBoundsException`; el índice final es **exclusivo** |
| API moderna | `repeat(n)`, `lines()`, `indent(n)`, `strip()`, `isBlank()`, `formatted()`, `chars()`, `codePoints()` |

## Construir texto: el coste real

```java
for (String s : lista) { r = r + s; }        // ✗ O(n²) — medido 91 veces más lento
StringBuilder sb = new StringBuilder();      // ✔ O(n)
for (String s : lista) { sb.append(s); }
String r = String.join("", lista);           // ✔ mejor aún si solo unís
```

`+` fuera de un bucle está perfecto: desde Java 9 (JEP 280) el compilador genera `invokedynamic` → `StringConcatFactory`, que la JVM optimiza mejor que un `StringBuilder` escrito a mano.

`StringBuilder` (rápido, no sincronizado) vs `StringBuffer` (sincronizado, prácticamente obsoleto: casi nunca se comparte un buffer entre hilos).

| Necesito | Herramienta |
|---|---|
| Una expresión simple | `+` |
| Concatenar en bucle | `StringBuilder` |
| Unir una colección | `String.join` / `Collectors.joining` |
| Unir con prefijo y sufijo | `StringJoiner` |
| Plantilla con formato | `formatted()` / `String.format` |
| Texto multilínea | Text block (`"""`) |
| Mensaje de log | placeholders `{}` |
| **SQL** | `PreparedStatement`, **nunca** concatenación |

### Logging: el caso que sí importa

```java
log.debug("Pedido " + id + " procesado");   // ✗ el String se construye SIEMPRE,
log.debug("Pedido {} procesado", id);       // ✔ aunque el nivel DEBUG esté apagado
```

### SQL: la vulnerabilidad más explotada de la historia

```java
String sql = "SELECT * FROM usuarios WHERE nombre = '" + nombre + "'";   // ✗ inyección SQL
```

Si `nombre` vale `' OR '1'='1`, devuelve la tabla entera. **Ningún text block ni `formatted()` arregla esto**: el problema no es la legibilidad, es la falta de separación entre instrucción y dato.

## Unicode: `char` no es un carácter

```java
"😊".length()                          // 2  ← surrogate pair, no 1
"😊".charAt(0)                         // basura: media letra
"😊".codePointCount(0, 2)              // 1  ✔
```

Tres niveles, y hay que saber cuál usar:

| Nivel | Qué es | Herramienta |
|---|---|---|
| **Code unit** | 16 bits — lo que cuenta `length()` | `length()`, para buffers e índices internos |
| **Code point** | un carácter Unicode real | `codePointCount()`, `codePoints()` |
| **Grafema** | lo que el usuario llama "un carácter" (`é` compuesta, emoji con modificador) | `BreakIterator` |

**Normalización:** `"café"` escrito con `é` precompuesta y con `e` + acento combinante son **dos Strings distintos** que se ven igual. `Normalizer.normalize(s, Form.NFC)` antes de comparar o guardar.

**Bytes:** especificá siempre el charset. `s.getBytes()` usa el del sistema y produce resultados distintos según la máquina; usá `s.getBytes(StandardCharsets.UTF_8)`.

## Conversiones y `null`

```java
Integer.parseInt("42")     // devuelve int; lanza NumberFormatException
Integer.valueOf("42")      // devuelve Integer (usa la caché)
String.valueOf(objeto)     // ✔ null-safe: devuelve "null"
objeto.toString()          // ✗ NPE si objeto es null
```

## Seguridad: por qué las contraseñas van en `char[]`

Un `String` es inmutable: **no se puede borrar de memoria**, y puede quedar en el heap hasta que el GC decida, apareciendo en un volcado. Un `char[]` se sobrescribe con `Arrays.fill(pwd, ' ')` en cuanto se usa. Por eso `JPasswordField.getPassword()` devuelve `char[]`.

## Anti-patrones (por frecuencia en revisiones)

| # | ✗ MAL | ✔ BIEN |
|---|---|---|
| 1 | `rol == "admin"` | `"admin".equals(rol)` |
| 2 | `r = r + s` en bucle | `StringBuilder` / `String.join` |
| 3 | `new String("hola")` | `"hola"` |
| 4 | `texto.trim();` | `texto = texto.trim();` |
| 5 | `h.toUpperCase().equals("X")` | `h.equalsIgnoreCase("X")` |
| 6 | `ip.replaceAll(".", "-")` | `ip.replace(".", "-")` |
| 7 | `linea.split(",")` en CSV | `linea.split(",", -1)` |
| 8 | `log.debug("id " + id)` | `log.debug("id {}", id)` |
| 9 | SQL concatenado | `PreparedStatement` |
| 10 | `length() == 0` para validar | `isBlank()` |
| 11 | `mensaje.length() > 280` con emojis | `codePointCount(...) > 280` |
| 12 | `StringBuilder` como clave de `Map` | no sobrescribe `equals`/`hashCode` |

## Checklist antes de dar por terminado código con texto

- [ ] ¿Comparé con `equals()` y no con `==`?
- [ ] ¿Puede ser `null`? ¿La constante va primero, o uso `Objects.equals`?
- [ ] ¿Valido con `isBlank()` si es entrada de usuario?
- [ ] ¿Hay concatenación dentro de un bucle?
- [ ] ¿`toUpperCase`/`toLowerCase`/`format` llevan `Locale` explícito?
- [ ] ¿Uso `replace` donde no necesito regex?
- [ ] ¿`split` lleva `-1` si las posiciones importan?
- [ ] ¿Especifiqué el charset en toda conversión a bytes?
- [ ] ¿El texto puede traer emojis o acentos combinantes? ¿Normalizo?
- [ ] ¿Hay datos sensibles que puedan acabar en un log o un `toString()`?
- [ ] ¿Construyo SQL/HTML/JSON concatenando?
- [ ] ¿Asigné el resultado de cada transformación?

## Preguntas de entrevista

<details><summary><strong>Respuestas</strong></summary>

- **¿Por qué `String` es inmutable?** Permite el pool, la hace thread-safe, permite cachear el `hashCode` y evita que un valor validado cambie después.
- **`==` vs `equals` en Strings.** `==` compara referencias; con literales da `true` por el pool, lo que engaña. Siempre `equals`.
- **¿Qué es el string pool y qué hace `intern()`?** Zona compartida donde viven los literales; `intern()` devuelve la instancia del pool.
- **`StringBuilder` vs `StringBuffer`.** El segundo está sincronizado; casi nunca se necesita.
- **¿Qué genera `"a" + b`?** Desde Java 9, `invokedynamic` con `StringConcatFactory`.
- **`replace` vs `replaceAll`.** Literal vs regex.
- **¿Por qué `split(",")` pierde columnas?** Descarta los campos vacíos finales; se corrige con `split(",", -1)`.
- **¿Cuánto vale `"😊".length()`?** 2 — es un surrogate pair de UTF-16.
- **¿Por qué las contraseñas en `char[]`?** Un `String` inmutable no se puede borrar de memoria.
- **¿Qué es el bug del locale turco?** `"i".toUpperCase()` no da `"I"` en `tr-TR`, y rompe comparaciones de cabeceras y claves.

</details>
