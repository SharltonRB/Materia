# Java

Roadmap de estudio de Java. Cada bloque se crea cuando se escribe su primer tema.

Los ejemplos asumen **Java 17+** (LTS). Cuando algo depende de una versión concreta, se indica de forma explícita — Java cambió mucho entre 8 y 25, y mezclar versiones es una fuente clásica de confusión.

---

## Bloques

### `01-Basics`
| # | Tema | Estado |
|---|---|---|
| 01 | [Basic Syntax](01-Basics/01-basic-syntax.md) | ✅ |
| 02 | [Lifecycle of a Program](01-Basics/02-lifecycle-of-a-program.md) | ✅ |
| 03 | [Data Types and Variables](01-Basics/03-data-types-and-variables.md) | ✅ |
| 04 | [Variables and Scopes](01-Basics/04-variables-and-scopes.md) | ✅ |
| 05 | [Type Casting](01-Basics/05-type-casting.md) | ✅ |
| 06 | [Strings and Methods](01-Basics/06-strings-and-methods.md) | ✅ |
| 07 | Operadores | ⬜ |
| 08 | Control de flujo | ⬜ |
| 09 | Arrays | ⬜ |
| 10 | Métodos y parámetros | ⬜ |

### `02-POO`
Clases y objetos · Encapsulación · Herencia vs composición · Polimorfismo · Interfaces · Clases abstractas · Records · Enums · Sealed classes

### `03-Tipos-y-Genericos`
Genéricos · Wildcards · Type erasure · `var` · Autoboxing · Casting

### `04-Colecciones`
List, Set, Map · Implementaciones y su complejidad · `equals`/`hashCode` · Comparable y Comparator · Iteradores

### `05-Excepciones`
Checked vs unchecked · Jerarquía · try-with-resources · Buenas prácticas · Errores de diseño frecuentes

### `06-Streams-y-Funcional`
Lambdas · Method references · Interfaces funcionales · Stream API · Collectors · `Optional`

### `07-Concurrencia`
Threads · `synchronized` y locks · Modelo de memoria · `ExecutorService` · `CompletableFuture` · Virtual threads (Java 21+)

### `08-JVM-y-Memoria`
Class loading · Heap y stack · Garbage collection · JIT · Profiling y diagnóstico

### `09-IO`
Ficheros · NIO.2 · Streams de datos · Serialización

### `10-Modulos-y-Build`
Paquetes · JPMS · Maven · Gradle · Estructura de un proyecto

### `11-Testing`
JUnit 5 · Mockito · AssertJ · Testcontainers

---

## Orden sugerido

`01` → `02` → `05` → `04` → `03` → `06` → `11` → `07` → `08`

Excepciones (`05`) va pronto a propósito: se tropieza con ellas desde el primer día. Concurrencia (`07`) y JVM (`08`) van al final no por dificultad de sintaxis, sino porque sin entender objetos, memoria y colecciones no se comprende *por qué* fallan.
