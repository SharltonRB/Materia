# Java

Roadmap de estudio de Java. Sigue la estructura del [Java Developer Roadmap](https://roadmap.sh/java), reordenada donde el orden pedagógico y el del roadmap no coinciden. Cada bloque se crea cuando se escribe su primer tema.

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
| 07 | [Math Operations](01-Basics/07-math-operations.md) | ✅ |
| 08 | [Logical, Relational and Bitwise Operators](01-Basics/08-logical-relational-bitwise-operators.md) | ✅ |
| 09 | [Arrays](01-Basics/09-arrays.md) | ✅ |
| 10 | [Conditionals](01-Basics/10-conditionals.md) | ✅ |
| 11 | [Loops](01-Basics/11-loops.md) | ✅ |
| 12 | [Methods and Parameters](01-Basics/12-methods-and-parameters.md) | ✅ |

### `02-POO`
Cubre las dos cajas del roadmap — *Basics of OOP* y *More about OOP* — en un orden que va de la estructura de una clase a los mecanismos que la hacen extensible.

| # | Tema | Estado |
|---|---|---|
| 01 | [Classes and Objects](02-POO/01-classes-and-objects.md) | ✅ |
| 02 | [Attributes and Methods](02-POO/02-attributes-and-methods.md) | ✅ |
| 03 | [Access Specifiers](02-POO/03-access-specifiers.md) | ✅ |
| 04 | [Static Keyword](02-POO/04-static-keyword.md) | ✅ |
| 05 | Final Keyword | ⬜ |
| 06 | Initializer Blocks and Object Lifecycle | ⬜ |
| 07 | Pass by Value vs Pass by Reference | ⬜ |
| 08 | Encapsulation | ⬜ |
| 09 | Inheritance | ⬜ |
| 10 | Method Overloading and Overriding | ⬜ |
| 11 | Polymorphism and Static vs Dynamic Binding | ⬜ |
| 12 | Abstraction and Abstract Classes | ⬜ |
| 13 | Interfaces | ⬜ |
| 14 | Inheritance vs Composition | ⬜ |
| 15 | Nested and Inner Classes | ⬜ |
| 16 | Records | ⬜ |
| 17 | Enums | ⬜ |
| 18 | Sealed Classes | ⬜ |
| 19 | Method Chaining and Fluent APIs | ⬜ |
| 20 | Packages and Project Structure | ⬜ |

### `03-Excepciones`
El nodo *Exception Handling* del roadmap, desglosado. Va inmediatamente después de POO porque la jerarquía de excepciones es el primer uso real de la herencia que vas a escribir.

| # | Tema | Estado |
|---|---|---|
| 01 | Qué es una excepción y qué ocurre al lanzarla | ⬜ |
| 02 | La jerarquía: Throwable, Error, Exception, RuntimeException | ⬜ |
| 03 | Checked vs unchecked | ⬜ |
| 04 | try, catch, multi-catch y finally | ⬜ |
| 05 | try-with-resources y AutoCloseable | ⬜ |
| 06 | throw, throws y la propagación | ⬜ |
| 07 | Excepciones propias y diseño de la jerarquía | ⬜ |
| 08 | Stack traces, causas encadenadas y suppressed exceptions | ⬜ |
| 09 | NullPointerException y los helpful NPE (Java 14+) | ⬜ |
| 10 | Anti-patrones y buenas prácticas | ⬜ |

### `04-Genericos`
No aparece como nodo propio en el roadmap, pero *Generic Collections* lo presupone entero. Sin este bloque, colecciones y Streams se usan a ciegas.

| # | Tema | Estado |
|---|---|---|
| 01 | Qué problema resuelven los genéricos | ⬜ |
| 02 | Clases, interfaces y métodos genéricos | ⬜ |
| 03 | Type erasure y sus consecuencias | ⬜ |
| 04 | Wildcards y el principio PECS | ⬜ |
| 05 | Límites: extends, super y tipos recursivos | ⬜ |
| 06 | Genéricos y arrays: por qué no se llevan | ⬜ |
| 07 | Heap pollution, unchecked warnings y @SafeVarargs | ⬜ |
| 08 | var, inferencia de tipos y diamante | ⬜ |
| 09 | Autoboxing y sus trampas | ⬜ |

### `05-Colecciones`
| # | Tema | Estado |
|---|---|---|
| 01 | El Java Collections Framework: mapa general | ⬜ |
| 02 | Array vs ArrayList | ⬜ |
| 03 | List: ArrayList, LinkedList y sus costes reales | ⬜ |
| 04 | Set: HashSet, LinkedHashSet, TreeSet | ⬜ |
| 05 | Map: HashMap por dentro, LinkedHashMap, TreeMap | ⬜ |
| 06 | El contrato equals/hashCode | ⬜ |
| 07 | Queue y Deque: ArrayDeque, PriorityQueue | ⬜ |
| 08 | Stack y por qué no se usa | ⬜ |
| 09 | Iterator, Iterable y ConcurrentModificationException | ⬜ |
| 10 | Comparable y Comparator | ⬜ |
| 11 | Colecciones inmutables y vistas no modificables | ⬜ |
| 12 | Colecciones concurrentes | ⬜ |
| 13 | Utilidades: Collections, Arrays, List.of y factorías | ⬜ |
| 14 | Elegir la colección correcta: tabla de decisión | ⬜ |

### `06-Funcional`
Las cajas *Functional Programming*, *Lambda Expressions* y *Optionals* del roadmap.

| # | Tema | Estado |
|---|---|---|
| 01 | Lambdas: sintaxis, captura y effectively final | ⬜ |
| 02 | Interfaces funcionales y @FunctionalInterface | ⬜ |
| 03 | El paquete java.util.function | ⬜ |
| 04 | Method references: las cuatro formas | ⬜ |
| 05 | Funciones de orden superior y composición | ⬜ |
| 06 | Stream API: creación, operaciones intermedias y terminales | ⬜ |
| 07 | Evaluación perezosa y el pipeline por dentro | ⬜ |
| 08 | Collectors y agrupaciones | ⬜ |
| 09 | Streams de primitivos y de ficheros | ⬜ |
| 10 | Streams paralelos: cuándo ayudan y cuándo empeoran | ⬜ |
| 11 | Optional: uso correcto y anti-patrones | ⬜ |
| 12 | Cuándo un bucle sigue siendo mejor | ⬜ |

### `07-Concurrencia`
| # | Tema | Estado |
|---|---|---|
| 01 | Procesos, hilos y el ciclo de vida de un Thread | ⬜ |
| 02 | Race conditions y secciones críticas | ⬜ |
| 03 | El Java Memory Model: visibilidad, reordenamiento y happens-before | ⬜ |
| 04 | volatile: qué garantiza y qué no | ⬜ |
| 05 | synchronized, monitores y locks intrínsecos | ⬜ |
| 06 | java.util.concurrent.locks y Condition | ⬜ |
| 07 | Atómicos y CAS | ⬜ |
| 08 | Deadlock, livelock e inanición | ⬜ |
| 09 | ExecutorService y thread pools | ⬜ |
| 10 | Future, CompletableFuture y composición asíncrona | ⬜ |
| 11 | ForkJoinPool y work stealing | ⬜ |
| 12 | Virtual threads (Java 21+) | ⬜ |
| 13 | Structured concurrency y scoped values | ⬜ |
| 14 | Diagnóstico: thread dumps, contención y tests de concurrencia | ⬜ |

### `08-JVM-y-Memoria`
No es un nodo del roadmap, pero es el bloque que explica *por qué* fallan la mayoría de los temas anteriores en producción.

| # | Tema | Estado |
|---|---|---|
| 01 | Arquitectura de la JVM | ⬜ |
| 02 | Class loading, linking e inicialización | ⬜ |
| 03 | Heap, stack, metaspace y áreas de memoria | ⬜ |
| 04 | Garbage collection: algoritmos y generaciones | ⬜ |
| 05 | Los recolectores actuales: G1, ZGC, Shenandoah, Serial | ⬜ |
| 06 | Referencias débiles, blandas y fantasma | ⬜ |
| 07 | Fugas de memoria en Java | ⬜ |
| 08 | El compilador JIT y las optimizaciones que aplica | ⬜ |
| 09 | Flags de la JVM que importan | ⬜ |
| 10 | Profiling y diagnóstico: JFR, jcmd, heap dumps | ⬜ |
| 11 | Benchmarking honesto con JMH | ⬜ |

### `09-APIs-de-la-Plataforma`
Los nodos sueltos del roadmap: *I/O Operations*, *File Operations*, *Date and Time*, *Regular Expressions*, *Networking* y *Cryptography*.

| # | Tema | Estado |
|---|---|---|
| 01 | I/O: streams de bytes y de caracteres | ⬜ |
| 02 | Encodings, Charset y el bug del UTF-8 | ⬜ |
| 03 | File Operations: NIO.2, Path y Files | ⬜ |
| 04 | NIO: buffers, channels y I/O no bloqueante | ⬜ |
| 05 | Serialización: Java nativa, JSON y por qué evitar la primera | ⬜ |
| 06 | Date and Time: java.time y el desastre que reemplazó | ⬜ |
| 07 | Zonas horarias, offsets y aritmética de fechas | ⬜ |
| 08 | Formateo, parseo y locales | ⬜ |
| 09 | Regular Expressions: sintaxis y la API Pattern/Matcher | ⬜ |
| 10 | Regex: rendimiento y backtracking catastrófico | ⬜ |
| 11 | Networking: sockets, URL y HttpClient (Java 11+) | ⬜ |
| 12 | Cryptography: hashing, HMAC y almacenamiento de contraseñas | ⬜ |
| 13 | Cryptography: cifrado simétrico y asimétrico con JCA | ⬜ |
| 14 | Keystores, certificados y TLS desde Java | ⬜ |

### `10-Anotaciones-y-Reflexion`
| # | Tema | Estado |
|---|---|---|
| 01 | Qué es una anotación y para qué sirve | ⬜ |
| 02 | Anotaciones integradas y meta-anotaciones | ⬜ |
| 03 | Retención: source, class, runtime | ⬜ |
| 04 | Anotaciones propias | ⬜ |
| 05 | Reflection: Class, Method, Field | ⬜ |
| 06 | Procesamiento de anotaciones en compilación | ⬜ |
| 07 | Cómo lo usan Spring, JPA y JUnit por debajo | ⬜ |
| 08 | Coste, riesgos y alternativas a la reflexión | ⬜ |

### `11-Modulos-y-Build`
| # | Tema | Estado |
|---|---|---|
| 01 | Paquetes, classpath y el JAR | ⬜ |
| 02 | JPMS: módulos, module-info y encapsulación fuerte | ⬜ |
| 03 | Migrar a módulos y por qué casi nadie lo hizo | ⬜ |
| 04 | Maven: ciclo de vida, POM y dependencias | ⬜ |
| 05 | Maven: resolución de versiones, scopes y multi-módulo | ⬜ |
| 06 | Gradle: modelo de tareas y build script | ⬜ |
| 07 | Gradle vs Maven: comparación honesta | ⬜ |
| 08 | Bazel y builds a gran escala | ⬜ |
| 09 | Empaquetado: fat JAR, jlink, jpackage y GraalVM native image | ⬜ |
| 10 | Gestión de dependencias: conflictos, vulnerabilidades y reproducibilidad | ⬜ |

### `12-Logging-y-Documentacion`
| # | Tema | Estado |
|---|---|---|
| 01 | Por qué existe una fachada: SLF4J | ⬜ |
| 02 | Logback: configuración y appenders | ⬜ |
| 03 | Log4j2, TinyLog y el resto del ecosistema | ⬜ |
| 04 | Niveles de log y qué registrar en cada uno | ⬜ |
| 05 | Logging estructurado y contexto (MDC) | ⬜ |
| 06 | Rendimiento, parámetros diferidos y datos sensibles | ⬜ |
| 07 | Javadoc: escribirlo, generarlo y publicarlo | ⬜ |
| 08 | Documentación que no envejece mal | ⬜ |

### `13-Testing`
| # | Tema | Estado |
|---|---|---|
| 01 | JUnit 5: arquitectura, ciclo de vida y anotaciones | ⬜ |
| 02 | Aserciones y AssertJ | ⬜ |
| 03 | Tests parametrizados y dinámicos | ⬜ |
| 04 | TestNG y en qué se diferencia | ⬜ |
| 05 | Mockito: stubs, mocks, spies y verificación | ⬜ |
| 06 | Sobre-mockeo y diseño testeable | ⬜ |
| 07 | Tests de integración y Testcontainers | ⬜ |
| 08 | REST Assured: probar APIs HTTP | ⬜ |
| 09 | Cucumber-JVM y behavior testing | ⬜ |
| 10 | JMeter y pruebas de carga | ⬜ |
| 11 | Cobertura con JaCoCo y mutation testing | ⬜ |

### `14-Inyeccion-de-Dependencias`
| # | Tema | Estado |
|---|---|---|
| 01 | El problema: acoplamiento y construcción de objetos | ⬜ |
| 02 | Inversión de control e inversión de dependencias | ⬜ |
| 03 | DI a mano, sin framework | ⬜ |
| 04 | Constructor, setter y field injection | ⬜ |
| 05 | Contenedores IoC: Spring, CDI, Guice, Dagger | ⬜ |
| 06 | Ciclo de vida y ámbitos de los beans | ⬜ |
| 07 | Dependencias circulares y otros olores | ⬜ |
| 08 | Cuándo un framework de DI es innecesario | ⬜ |

### `15-Acceso-a-Datos`
| # | Tema | Estado |
|---|---|---|
| 01 | JDBC: driver, Connection, Statement, ResultSet | ⬜ |
| 02 | PreparedStatement e inyección SQL | ⬜ |
| 03 | Transacciones desde JDBC | ⬜ |
| 04 | Connection pooling: HikariCP y dimensionado | ⬜ |
| 05 | JPA: el estándar, entidades y el EntityManager | ⬜ |
| 06 | Hibernate: contexto de persistencia, estados y flush | ⬜ |
| 07 | Mapeo de relaciones y lazy vs eager | ⬜ |
| 08 | El problema N+1 y cómo detectarlo | ⬜ |
| 09 | JPQL, Criteria y SQL nativo | ⬜ |
| 10 | Spring Data JPA: repositorios y derivación de queries | ⬜ |
| 11 | Migraciones con Flyway o Liquibase | ⬜ |
| 12 | Alternativas: EBean, jOOQ, MyBatis, JdbcTemplate | ⬜ |
| 13 | Cuándo bajar a SQL y abandonar el ORM | ⬜ |

### `16-Frameworks-Web`
| # | Tema | Estado |
|---|---|---|
| 01 | Qué hace un framework web y qué no | ⬜ |
| 02 | Servlets y el modelo de ejecución que hay debajo | ⬜ |
| 03 | Spring Core: contexto, beans y configuración | ⬜ |
| 04 | Spring Boot: autoconfiguración y starters | ⬜ |
| 05 | Spring MVC: controladores, binding y validación | ⬜ |
| 06 | Manejo de errores y respuestas de API | ⬜ |
| 07 | Configuración, perfiles y secretos | ⬜ |
| 08 | Spring Security: autenticación y autorización | ⬜ |
| 09 | Actuator, métricas y health checks | ⬜ |
| 10 | Testing en Spring Boot | ⬜ |
| 11 | WebFlux y el modelo reactivo | ⬜ |
| 12 | Quarkus, Micronaut y el arranque nativo | ⬜ |
| 13 | Javalin, Play y los frameworks ligeros | ⬜ |
| 14 | Cómo elegir: tabla comparativa | ⬜ |

---

## Orden sugerido

`01` → `02` → `03` → `04` → `05` → `06` → `13` → `07` → `08` → `09` → `10` → `11` → `12` → `14` → `15` → `16`

El orden del roadmap se altera en tres puntos, a propósito:

- **Genéricos (`04`) antes que colecciones.** El roadmap habla de *Generic Collections* como si fueran un tipo de colección; en realidad los genéricos son el mecanismo que sostiene toda la API. Verlos después obliga a usar `List<String>` sin entender qué significa el paréntesis angular.
- **Testing (`13`) mucho antes de su posición en el roadmap.** JUnit y Mockito no requieren nada de concurrencia, JVM ni frameworks, y todo lo que viene después se estudia mejor escribiendo tests que lo comprueben.
- **Concurrencia (`07`) y JVM (`08`) juntos y antes de los frameworks.** No por dificultad de sintaxis, sino porque un pool de conexiones, un `@Transactional` o un thread pool de Tomcat son incomprensibles sin ellos.

Los bloques `14`, `15` y `16` son los que más se solapan con las etapas de `Datos` y `APIs` del repositorio raíz: aquí se cubre la parte específica de Java, y el concepto general vive en su carpeta propia.
