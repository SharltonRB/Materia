# Spring Boot

Roadmap de estudio de Spring y Spring Boot. Sigue la estructura del [Spring Boot Roadmap](https://roadmap.sh/spring-boot), reordenada para que el framework se entienda de dentro hacia fuera: primero el contenedor de beans, después la magia de la autoconfiguración.

Los ejemplos asumen **Spring Boot 3.x sobre Java 17+**. Cuando algo cambió respecto a Spring Boot 2.x o a Spring Framework 5 se indica de forma explícita — hay mucho material en internet escrito para versiones anteriores y varias de las anotaciones más comunes se renombraron o quedaron obsoletas.

---

## Bloques

### `01-Introduccion`
El nodo *Introduction* del roadmap. La pregunta que ordena el bloque no es qué es Spring, sino qué escribía la gente antes de que existiera.

| # | Tema | Estado |
|---|---|---|
| 01 | Terminology: el vocabulario mínimo de Spring | ⬜ |
| 02 | Why use Spring: qué problema resolvió y a qué coste | ⬜ |
| 03 | Architecture: Spring Framework, Boot, Data, Security y Cloud | ⬜ |
| 04 | Spring Boot vs Spring clásico y la configuración XML | ⬜ |
| 05 | Crear un proyecto: Initializr, estructura y dependencias | ⬜ |
| 06 | El ciclo de vida de una petición, de punta a punta | ⬜ |

### `02-Core-IoC-y-DI`
*Spring IOC*, *Dependency Injection*, *Spring Bean Scope*, *Annotations* y *Configuration*. Es el bloque más importante del roadmap entero: todo lo demás son abstracciones construidas encima del contenedor.

| # | Tema | Estado |
|---|---|---|
| 01 | Inversión de control: el concepto antes del framework | ⬜ |
| 02 | El contenedor: BeanFactory y ApplicationContext | ⬜ |
| 03 | Definir beans: @Component, @Bean y el escaneo de componentes | ⬜ |
| 04 | Inyección por constructor, setter y campo | ⬜ |
| 05 | Resolver ambigüedad: @Qualifier, @Primary, listas y mapas de beans | ⬜ |
| 06 | Bean scopes: singleton, prototype, request, session | ⬜ |
| 07 | Ciclo de vida de un bean y sus hooks | ⬜ |
| 08 | Dependencias circulares y por qué Boot 3 las prohíbe por defecto | ⬜ |
| 09 | Configuration: properties, YAML y orden de precedencia | ⬜ |
| 10 | @ConfigurationProperties y configuración tipada | ⬜ |
| 11 | Profiles y configuración por entorno | ⬜ |
| 12 | Beans condicionales: @Conditional y @Profile | ⬜ |
| 13 | Las anotaciones de Spring: mapa y meta-anotaciones | ⬜ |
| 14 | Eventos de aplicación y @EventListener | ⬜ |

### `03-Spring-AOP`
| # | Tema | Estado |
|---|---|---|
| 01 | Qué es una preocupación transversal | ⬜ |
| 02 | Vocabulario: aspecto, join point, pointcut, advice, weaving | ⬜ |
| 03 | Proxies dinámicos y CGLIB: cómo lo hace Spring | ⬜ |
| 04 | Escribir un aspecto y expresiones pointcut | ⬜ |
| 05 | La trampa de la autoinvocación: por qué tu @Transactional no se aplica | ⬜ |
| 06 | AOP en la práctica: logging, métricas, auditoría, reintentos | ⬜ |

### `04-Spring-Boot-Core`
Los nodos *Spring Boot Starters*, *Autoconfiguration*, *Embedded Server* y *Actuators*.

| # | Tema | Estado |
|---|---|---|
| 01 | Starters: qué son y qué arrastran | ⬜ |
| 02 | Autoconfiguración: el mecanismo por dentro | ⬜ |
| 03 | Depurar la autoconfiguración y el informe de condiciones | ⬜ |
| 04 | Sobrescribir y excluir autoconfiguración | ⬜ |
| 05 | Escribir tu propio starter | ⬜ |
| 06 | Embedded server: Tomcat, Jetty, Undertow y su thread pool | ⬜ |
| 07 | Actuator: endpoints, exposición y seguridad | ⬜ |
| 08 | Health checks, readiness y liveness | ⬜ |
| 09 | CommandLineRunner, ApplicationRunner y arranque | ⬜ |
| 10 | DevTools y ciclo de desarrollo | ⬜ |

### `05-Spring-MVC`
El nodo *Spring MVC* con sus hijos: *Servlet*, *Architecture*, *Components*, *JSP Files*. Se enfoca hacia APIs REST, que es lo que se escribe hoy.

| # | Tema | Estado |
|---|---|---|
| 01 | La API Servlet: el modelo que hay debajo de todo | ⬜ |
| 02 | Architecture: DispatcherServlet y el flujo de una petición | ⬜ |
| 03 | Components: handler mapping, adapters, view resolvers | ⬜ |
| 04 | @Controller y @RestController | ⬜ |
| 05 | Mapeo de rutas y variables de ruta | ⬜ |
| 06 | Binding de parámetros, cuerpo y cabeceras | ⬜ |
| 07 | Serialización JSON con Jackson | ⬜ |
| 08 | Validación con Bean Validation | ⬜ |
| 09 | Manejo de errores: @ExceptionHandler, @ControllerAdvice y Problem Details | ⬜ |
| 10 | ResponseEntity, códigos de estado y diseño de respuestas | ⬜ |
| 11 | Filtros e interceptores: cuál usar y cuándo | ⬜ |
| 12 | CORS en Spring | ⬜ |
| 13 | Subida de ficheros y streaming | ⬜ |
| 14 | Vistas del lado servidor: JSP, Thymeleaf y su lugar hoy | ⬜ |
| 15 | Documentar la API: springdoc y OpenAPI | ⬜ |

### `06-Acceso-a-Datos`
Los nodos *Hibernate* (con *Transactions*, *Relationships*, *Entity Lifecycle*) y *Spring Data* (*JPA*, *JDBC*, *MongoDB*).

| # | Tema | Estado |
|---|---|---|
| 01 | DataSource, pool de conexiones y HikariCP | ⬜ |
| 02 | JdbcTemplate y acceso sin ORM | ⬜ |
| 03 | JPA e Hibernate: estándar frente a implementación | ⬜ |
| 04 | Entidades, identificadores y estrategias de generación | ⬜ |
| 05 | Entity Lifecycle: transient, managed, detached, removed | ⬜ |
| 06 | El persistence context, el flush y la caché de primer nivel | ⬜ |
| 07 | Relationships: OneToMany, ManyToOne, ManyToMany y el lado propietario | ⬜ |
| 08 | Lazy vs eager y LazyInitializationException | ⬜ |
| 09 | El problema N+1: detectarlo y resolverlo | ⬜ |
| 10 | Transactions: @Transactional, propagación y aislamiento | ⬜ |
| 11 | Rollback, excepciones y los casos en que no funciona | ⬜ |
| 12 | Spring Data JPA: repositorios y derivación de queries | ⬜ |
| 13 | @Query, JPQL, Criteria y SQL nativo | ⬜ |
| 14 | Paginación, ordenación y proyecciones | ⬜ |
| 15 | Auditoría y bloqueo optimista | ⬜ |
| 16 | Spring Data JDBC: el modelo alternativo | ⬜ |
| 17 | Spring Data MongoDB | ⬜ |
| 18 | Migraciones con Flyway o Liquibase | ⬜ |
| 19 | Cuándo el ORM estorba | ⬜ |

### `07-Spring-Security`
| # | Tema | Estado |
|---|---|---|
| 01 | Arquitectura: la cadena de filtros de seguridad | ⬜ |
| 02 | SecurityFilterChain y la configuración moderna sin WebSecurityConfigurerAdapter | ⬜ |
| 03 | Authentication: AuthenticationManager, providers y UserDetailsService | ⬜ |
| 04 | Almacenamiento de contraseñas y PasswordEncoder | ⬜ |
| 05 | Authorization: roles, autoridades y reglas por ruta | ⬜ |
| 06 | Seguridad a nivel de método y @PreAuthorize | ⬜ |
| 07 | Sesiones frente a tokens | ⬜ |
| 08 | JWT Authentication: emisión, validación y errores frecuentes | ⬜ |
| 09 | OAuth2 y OpenID Connect: los flujos que importan | ⬜ |
| 10 | Resource server y client con Spring Security | ⬜ |
| 11 | CSRF, CORS y cabeceras de seguridad | ⬜ |
| 12 | Testing de código protegido | ⬜ |
| 13 | Anti-patrones y fallos clásicos de configuración | ⬜ |

### `08-Testing`
Los nodos *@SpringBootTest*, *@MockBean*, *Mock MVC* y *JPA Test*.

| # | Tema | Estado |
|---|---|---|
| 01 | Qué test necesita contexto de Spring y cuál no | ⬜ |
| 02 | @SpringBootTest: modos, coste y caché de contexto | ⬜ |
| 03 | Test slices: @WebMvcTest, @DataJpaTest, @JsonTest y compañía | ⬜ |
| 04 | MockMvc: probar controladores sin servidor | ⬜ |
| 05 | @MockBean, @MockitoBean y la sustitución de beans | ⬜ |
| 06 | JPA Test: base de datos embebida frente a Testcontainers | ⬜ |
| 07 | Datos de prueba, transacciones y limpieza entre tests | ⬜ |
| 08 | TestRestTemplate y WebTestClient | ⬜ |
| 09 | Configuración y perfiles de test | ⬜ |
| 10 | Tiempo de la suite: el problema que nadie ataca a tiempo | ⬜ |

### `09-Capacidades-Transversales`
No son nodos del roadmap pero aparecen en cualquier servicio real de Spring Boot.

| # | Tema | Estado |
|---|---|---|
| 01 | Caching: @Cacheable, proveedores e invalidación | ⬜ |
| 02 | Tareas programadas con @Scheduled | ⬜ |
| 03 | Ejecución asíncrona con @Async y su thread pool | ⬜ |
| 04 | Cliente HTTP: RestClient, WebClient y RestTemplate | ⬜ |
| 05 | Mensajería: Spring for Kafka y RabbitMQ | ⬜ |
| 06 | Reintentos y resiliencia con Spring Retry | ⬜ |
| 07 | Internacionalización y mensajes | ⬜ |
| 08 | Gestión de secretos y configuración sensible | ⬜ |

### `10-Microservicios-y-Spring-Cloud`
Los nodos *Microservices* y *Spring Cloud* con todos sus hijos.

| # | Tema | Estado |
|---|---|---|
| 01 | Qué aporta Spring Cloud y qué resuelve la plataforma | ⬜ |
| 02 | Service discovery con Eureka | ⬜ |
| 03 | Cloud Config: configuración centralizada y refresco | ⬜ |
| 04 | Spring Cloud Gateway: rutas, filtros y rate limiting | ⬜ |
| 05 | Spring Cloud OpenFeign: clientes declarativos | ⬜ |
| 06 | Circuit Breaker con Resilience4j: timeouts, bulkhead y fallback | ⬜ |
| 07 | Micrometer: métricas y export a Prometheus | ⬜ |
| 08 | Trazas distribuidas y correlación de peticiones | ⬜ |
| 09 | Comunicación entre servicios: síncrona frente a eventos | ⬜ |
| 10 | Cuándo Spring Cloud es innecesario | ⬜ |

### `11-Produccion`
| # | Tema | Estado |
|---|---|---|
| 01 | Empaquetado: fat JAR, capas y imagen Docker | ⬜ |
| 02 | Buildpacks y GraalVM native image | ⬜ |
| 03 | Arranque, apagado elegante y señales | ⬜ |
| 04 | Logging estructurado y correlación | ⬜ |
| 05 | Tuning del thread pool y del pool de conexiones | ⬜ |
| 06 | Memoria y JVM dentro de un contenedor | ⬜ |
| 07 | Diagnóstico de un servicio Spring Boot lento | ⬜ |
| 08 | Actualizar de versión mayor sin romper todo | ⬜ |

### `12-Alternativas-y-Contexto`
| # | Tema | Estado |
|---|---|---|
| 01 | WebFlux y el modelo reactivo: cuándo compensa | ⬜ |
| 02 | Virtual threads en Spring Boot 3.2+ frente a reactivo | ⬜ |
| 03 | Quarkus, Micronaut y Helidon | ⬜ |
| 04 | Críticas fundadas a Spring: magia, arranque, acoplamiento | ⬜ |

---

## Orden sugerido

`01` → `02` → `03` → `04` → `05` → `08` → `06` → `07` → `09` → `11` → `10` → `12`

Tres desviaciones respecto al roadmap, a propósito:

- **AOP (`03`) va temprano.** El roadmap lo pone como un nodo más de *Introduction*, pero sin entender los proxies no se explica por qué `@Transactional` falla al llamar a un método desde la misma clase — el bug más repetido de Spring.
- **Testing (`08`) antes que datos y seguridad.** Es lo que te permite verificar que entendiste los bloques siguientes en vez de suponerlo.
- **Spring Cloud (`10`) casi al final.** El roadmap lo coloca justo después de Spring Data, pero los microservicios son una decisión de arquitectura, no una etapa de aprendizaje de Spring.

---

## Solapamiento con otras carpetas del repositorio

Este bloque toca temas que ya tienen carpeta propia. El criterio es: **aquí va el cómo de Spring, allá el qué y el porqué.**

| Tema | Aquí | En su carpeta |
|---|---|---|
| Inyección de dependencias | `02` — el contenedor de Spring | `Inyeccion-de-Dependencias` — IoC como principio |
| Transacciones y JPA | `06` — `@Transactional`, Hibernate | `Transacciones`, `ORMs` — aislamiento, N+1 como concepto |
| Seguridad | `07` — la cadena de filtros | `Autenticacion-y-Autorizacion` — OAuth2, JWT, sesiones |
| Testing | `08` — slices, MockMvc | `Testing` — tipos de test, dobles, cobertura |
| Microservicios | `10` — Eureka, Gateway, Feign | `Microservicios` — límites, datos, fallos |
| Frameworks web | todo | `Pura/Java/16-Frameworks-Web` — redundante, conviene eliminarlo |
