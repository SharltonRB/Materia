# Backend

Roadmap de estudio de desarrollo backend. Sigue la estructura del [Backend Roadmap](https://roadmap.sh/backend), reordenada donde el orden pedagógico y el del roadmap no coinciden.

Es el roadmap más transversal del repositorio: aquí van los conceptos que valen en cualquier lenguaje. Lo específico de Java vive en `Pura/Java` y lo específico de Spring en `SpringBoot/pura`.

---

## Bloques

### `01-Fundamentos-de-Internet`
El nodo *Introduction* del roadmap. Son los temas que todo el mundo da por sabidos y casi nadie sabe con precisión.

| # | Tema | Estado |
|---|---|---|
| 01 | How does the internet work: de un clic al paquete | ⬜ |
| 02 | Direcciones IP, puertos, TCP y UDP | ⬜ |
| 03 | What is HTTP: métodos, status codes, headers | ⬜ |
| 04 | HTTP/1.1, HTTP/2 y HTTP/3 | ⬜ |
| 05 | What is a Domain Name | ⬜ |
| 06 | DNS and how it works | ⬜ |
| 07 | What is hosting: shared, VPS, dedicado, cloud, serverless | ⬜ |
| 08 | Browsers and how they work | ⬜ |
| 09 | Cookies, sesiones y estado en un protocolo sin estado | ⬜ |
| 10 | Diagnóstico de red: curl, dig, traceroute, DevTools | ⬜ |

### `02-Bases-Previas`
Los nodos *Frontend Basics* y *Pick a Backend Language*. Bloque corto: lo justo para entender al consumidor de tu API y para elegir con criterio.

| # | Tema | Estado |
|---|---|---|
| 01 | HTML: lo que un backend necesita saber | ⬜ |
| 02 | CSS: lo que un backend necesita saber | ⬜ |
| 03 | JavaScript y el modelo de ejecución del navegador | ⬜ |
| 04 | Cómo consume una API un frontend moderno | ⬜ |
| 05 | Elegir un lenguaje backend: criterios reales | ⬜ |
| 06 | Panorama: Java, Go, Python, C#, Node, Rust, PHP, Ruby | ⬜ |

### `03-Control-de-Versiones`
Los nodos *Version Control Systems* y *Repo Hosting Services*.

| # | Tema | Estado |
|---|---|---|
| 01 | Qué resuelve un sistema de control de versiones | ⬜ |
| 02 | Git: modelo de objetos y las tres zonas | ⬜ |
| 03 | Ramas, merge y rebase | ⬜ |
| 04 | Conflictos y cómo deshacer cualquier cosa | ⬜ |
| 05 | Remotos, fetch, pull y push | ⬜ |
| 06 | GitHub y GitLab: issues, PRs y protección de ramas | ⬜ |
| 07 | Workflows de equipo: trunk-based, GitHub Flow, Git Flow | ⬜ |
| 08 | Code review: dar y recibir feedback | ⬜ |

### `04-Bases-de-Datos-Relacionales`
El nodo *Relational Databases* con *Migrations* y *N+1 Problem*.

| # | Tema | Estado |
|---|---|---|
| 01 | Modelo relacional: tablas, claves y relaciones | ⬜ |
| 02 | SQL: SELECT, filtrado y ordenación | ⬜ |
| 03 | JOINs: los cinco tipos y cuándo usar cada uno | ⬜ |
| 04 | Agregaciones, GROUP BY y HAVING | ⬜ |
| 05 | Subconsultas, CTEs y funciones de ventana | ⬜ |
| 06 | INSERT, UPDATE, DELETE y UPSERT | ⬜ |
| 07 | DDL: crear esquemas, tipos y restricciones | ⬜ |
| 08 | Normalización y cuándo desnormalizar | ⬜ |
| 09 | El panorama: PostgreSQL, MySQL, MariaDB, SQLite, MS SQL, Oracle | ⬜ |
| 10 | Migrations: versionar el esquema | ⬜ |
| 11 | ORMs: qué resuelven y qué esconden | ⬜ |
| 12 | El problema N+1: detectarlo y resolverlo | ⬜ |

### `05-APIs`
El nodo *Learn about APIs* con *API Styles* y *Open API Specs*.

| # | Tema | Estado |
|---|---|---|
| 01 | Qué es una API y qué contrato establece | ⬜ |
| 02 | REST: los seis principios y el nivel de madurez real | ⬜ |
| 03 | Diseño de recursos, rutas y verbos | ⬜ |
| 04 | Códigos de estado y diseño de errores | ⬜ |
| 05 | Paginación, filtrado y ordenación | ⬜ |
| 06 | Versionado de APIs y cambios rompedores | ⬜ |
| 07 | Idempotencia y reintentos seguros | ⬜ |
| 08 | Rate limiting y cuotas | ⬜ |
| 09 | JSON APIs y JSON:API como especificación | ⬜ |
| 10 | GraphQL: schema, resolvers y sus costes | ⬜ |
| 11 | gRPC y Protocol Buffers | ⬜ |
| 12 | SOAP y por qué todavía te lo vas a encontrar | ⬜ |
| 13 | Elegir estilo de API: tabla de decisión | ⬜ |
| 14 | Open API Specs: documentar y generar | ⬜ |
| 15 | Validación de entrada en la frontera | ⬜ |

### `06-Autenticacion-y-Autorizacion`
El nodo *Authentication* con todos sus hijos.

| # | Tema | Estado |
|---|---|---|
| 01 | Autenticación frente a autorización | ⬜ |
| 02 | Basic Authentication | ⬜ |
| 03 | Cookie Based Auth y gestión de sesiones | ⬜ |
| 04 | Token Authentication | ⬜ |
| 05 | JWT: estructura, firma y errores frecuentes | ⬜ |
| 06 | Refresh tokens, revocación y expiración | ⬜ |
| 07 | OAuth 2.0: roles, flujos y cuál usar | ⬜ |
| 08 | OpenID Connect | ⬜ |
| 09 | SAML y SSO empresarial | ⬜ |
| 10 | Autorización: RBAC, ABAC y ACL | ⬜ |
| 11 | Multifactor y passkeys | ⬜ |
| 12 | Sesión frente a token: tabla de decisión | ⬜ |

### `07-Seguridad`
Los nodos *Web Security*, *Hashing Algorithms*, *OWASP Risks* y *API Security Best Practices*.

| # | Tema | Estado |
|---|---|---|
| 01 | Modelo de amenazas: pensar como atacante | ⬜ |
| 02 | Hashing: MD5, SHA y por qué no sirven para contraseñas | ⬜ |
| 03 | bcrypt, scrypt y Argon2: salt, coste y almacenamiento | ⬜ |
| 04 | Cifrado simétrico y asimétrico | ⬜ |
| 05 | SSL/TLS: handshake, certificados y cadenas de confianza | ⬜ |
| 06 | HTTPS, HSTS y terminación TLS | ⬜ |
| 07 | OWASP Top 10 explicado con ejemplos | ⬜ |
| 08 | Inyección SQL y consultas parametrizadas | ⬜ |
| 09 | XSS, CSRF y CSP | ⬜ |
| 10 | CORS: qué protege y qué no | ⬜ |
| 11 | Gestión de secretos y configuración sensible | ⬜ |
| 12 | Server Security: superficie de ataque y hardening | ⬜ |
| 13 | Dependencias vulnerables y cadena de suministro | ⬜ |
| 14 | API Security Best Practices: checklist | ⬜ |

### `08-Caching`
| # | Tema | Estado |
|---|---|---|
| 01 | Por qué cachear y qué se rompe al hacerlo | ⬜ |
| 02 | HTTP Caching: Cache-Control, ETag y revalidación | ⬜ |
| 03 | CDN y caché de borde | ⬜ |
| 04 | Caché en aplicación y caché distribuida | ⬜ |
| 05 | Redis: estructuras, persistencia y usos más allá de la caché | ⬜ |
| 06 | Memcached y cuándo sigue teniendo sentido | ⬜ |
| 07 | Patrones: cache-aside, read-through, write-through, write-behind | ⬜ |
| 08 | Invalidación, TTL y datos obsoletos | ⬜ |
| 09 | Stampede, avalancha y penetración de caché | ⬜ |

### `09-Servidores-Web`
| # | Tema | Estado |
|---|---|---|
| 01 | Qué hace un servidor web y qué hace tu aplicación | ⬜ |
| 02 | Reverse proxy, forward proxy y load balancer | ⬜ |
| 03 | Nginx: configuración, rutas y TLS | ⬜ |
| 04 | Apache, Caddy y MS IIS | ⬜ |
| 05 | Balanceo de carga y health checks | ⬜ |
| 06 | Compresión, keep-alive y timeouts | ⬜ |
| 07 | X-Forwarded-For y la IP real del cliente | ⬜ |
| 08 | Servir estáticos, uploads y streaming | ⬜ |

### `10-Testing`
| # | Tema | Estado |
|---|---|---|
| 01 | Qué garantiza un test y qué no | ⬜ |
| 02 | Unit Testing | ⬜ |
| 03 | Integration Testing | ⬜ |
| 04 | Functional y end-to-end testing | ⬜ |
| 05 | Dobles de prueba: mock, stub, fake, spy | ⬜ |
| 06 | Pirámide de tests y sus críticas | ⬜ |
| 07 | Tests de contrato entre servicios | ⬜ |
| 08 | Cobertura, tests inestables y salud de la suite | ⬜ |
| 09 | Tests de carga y de rendimiento | ⬜ |

### `11-CI-CD`
| # | Tema | Estado |
|---|---|---|
| 01 | Integración continua: el concepto y la disciplina | ⬜ |
| 02 | Anatomía de un pipeline | ⬜ |
| 03 | GitHub Actions, GitLab CI y alternativas | ⬜ |
| 04 | Build reproducible y gestión de artefactos | ⬜ |
| 05 | Entornos, promoción y gestión de configuración | ⬜ |
| 06 | Estrategias de despliegue: blue-green, canary, rolling | ⬜ |
| 07 | Feature flags y desacoplar despliegue de release | ⬜ |
| 08 | Rollback y migraciones de base de datos en el pipeline | ⬜ |
| 09 | Secretos en CI y seguridad del pipeline | ⬜ |

### `12-Contenedores-y-Orquestacion`
| # | Tema | Estado |
|---|---|---|
| 01 | Qué es un contenedor: namespaces y cgroups | ⬜ |
| 02 | Docker: imágenes, capas y contenedores | ⬜ |
| 03 | Escribir un Dockerfile decente | ⬜ |
| 04 | Multi-stage builds, tamaño y seguridad de imagen | ⬜ |
| 05 | Volúmenes, redes y variables de entorno | ⬜ |
| 06 | Docker Compose para desarrollo local | ⬜ |
| 07 | Registries y versionado de imágenes | ⬜ |
| 08 | LXC y otras tecnologías de contenedor | ⬜ |
| 09 | Kubernetes: pods, deployments, services | ⬜ |
| 10 | Config, secretos, ingress y almacenamiento en Kubernetes | ⬜ |
| 11 | Escalado, probes y límites de recursos | ⬜ |
| 12 | Cuándo Kubernetes es una mala idea | ⬜ |

### `13-Bases-de-Datos-Avanzado`
El nodo *More about Databases*.

| # | Tema | Estado |
|---|---|---|
| 01 | Transactions: qué garantiza una transacción | ⬜ |
| 02 | ACID en detalle | ⬜ |
| 03 | Niveles de aislamiento y anomalías | ⬜ |
| 04 | Bloqueo optimista y pesimista | ⬜ |
| 05 | Database Indexes: B-tree, hash y otros | ⬜ |
| 06 | Cuándo un índice ayuda y cuándo estorba | ⬜ |
| 07 | Planes de ejecución y EXPLAIN | ⬜ |
| 08 | Profiling Performance y consultas lentas | ⬜ |
| 09 | Connection pooling | ⬜ |
| 10 | Failure Modes: qué falla y cómo se manifiesta | ⬜ |
| 11 | Backups, restauración y recuperación ante desastres | ⬜ |

### `14-Escalado-de-Datos`
Los nodos *Scaling Databases* y *NoSQL Databases*.

| # | Tema | Estado |
|---|---|---|
| 01 | Escalado vertical y horizontal | ⬜ |
| 02 | Data Replication: primario-réplica y multi-primario | ⬜ |
| 03 | Consistencia eventual y lag de réplica | ⬜ |
| 04 | Sharding Strategies y claves de partición | ⬜ |
| 05 | CAP Theorem y su lectura correcta | ⬜ |
| 06 | Cuándo NoSQL y cuándo no | ⬜ |
| 07 | Document DBs: MongoDB, CouchDB | ⬜ |
| 08 | Key-Value: Redis, DynamoDB | ⬜ |
| 09 | Column DBs: Cassandra, ClickHouse, ScyllaDB | ⬜ |
| 10 | Graph DBs: Neo4j, Neptune, DGraph | ⬜ |
| 11 | Time Series: InfluxDB, TimescaleDB | ⬜ |
| 12 | Realtime DBs: Firebase, RethinkDB | ⬜ |
| 13 | Modelado de datos en NoSQL | ⬜ |
| 14 | Elegir base de datos: tabla de decisión | ⬜ |

### `15-Mensajeria-y-Busqueda`
Los nodos *Message Brokers* y *Search Engines*.

| # | Tema | Estado |
|---|---|---|
| 01 | Comunicación síncrona frente a asíncrona | ⬜ |
| 02 | Colas, tópicos y patrones de mensajería | ⬜ |
| 03 | RabbitMQ: exchanges, colas y routing | ⬜ |
| 04 | Kafka: log distribuido, particiones y consumer groups | ⬜ |
| 05 | Entrega: at-most-once, at-least-once, exactly-once | ⬜ |
| 06 | Idempotencia, orden y dead letter queues | ⬜ |
| 07 | Outbox pattern y consistencia con la base de datos | ⬜ |
| 08 | Search Engines: por qué no basta un LIKE | ⬜ |
| 09 | Índice invertido, análisis y relevancia | ⬜ |
| 10 | Elasticsearch y OpenSearch | ⬜ |
| 11 | Solr y alternativas | ⬜ |
| 12 | Sincronizar la base de datos con el índice | ⬜ |

### `16-Datos-en-Tiempo-Real`
| # | Tema | Estado |
|---|---|---|
| 01 | Short polling y long polling | ⬜ |
| 02 | Server Sent Events | ⬜ |
| 03 | WebSockets: handshake, frames y ciclo de vida | ⬜ |
| 04 | Elegir mecanismo: tabla de decisión | ⬜ |
| 05 | Escalar conexiones persistentes | ⬜ |
| 06 | Presencia, reconexión y entrega garantizada | ⬜ |

### `17-Arquitectura`
Los nodos *Architectural Patterns*, *System Design* y *Design & Architecture*.

| # | Tema | Estado |
|---|---|---|
| 01 | Monolito: el punto de partida correcto | ⬜ |
| 02 | Monolito modular | ⬜ |
| 03 | Microservicios: qué resuelven y qué cuestan | ⬜ |
| 04 | Definir límites de servicio | ⬜ |
| 05 | Datos en microservicios: sagas y consistencia | ⬜ |
| 06 | SOA y su relación con los microservicios | ⬜ |
| 07 | Serverless y funciones | ⬜ |
| 08 | Service Mesh | ⬜ |
| 09 | Arquitectura orientada a eventos | ⬜ |
| 10 | Twelve Factor Apps | ⬜ |
| 11 | Arquitectura en capas, hexagonal y clean | ⬜ |
| 12 | System Design: método para atacar un diseño | ⬜ |
| 13 | Estimación de capacidad y cuellos de botella | ⬜ |
| 14 | Documentar decisiones: ADRs | ⬜ |

### `18-Observabilidad-y-Resiliencia`
Los nodos *Observability*, *Mitigation Strategies* y *Building For Scale*.

| # | Tema | Estado |
|---|---|---|
| 01 | Monitoring frente a observabilidad | ⬜ |
| 02 | Los tres pilares: logs, métricas y trazas | ⬜ |
| 03 | Instrumentation y OpenTelemetry | ⬜ |
| 04 | Logging estructurado y correlación | ⬜ |
| 05 | Métricas: tipos, cardinalidad y Prometheus | ⬜ |
| 06 | Trazas distribuidas | ⬜ |
| 07 | Telemetry, dashboards y alertas que no queman | ⬜ |
| 08 | SLI, SLO y error budgets | ⬜ |
| 09 | Timeouts, reintentos y backoff | ⬜ |
| 10 | Circuit Breaker y bulkhead | ⬜ |
| 11 | Throttling, backpressure y loadshifting | ⬜ |
| 12 | Graceful Degradation | ⬜ |
| 13 | Incidentes, on-call y post-mortems sin culpables | ⬜ |

### `19-IA-Asistida-al-Desarrollo`
El nodo *AI in Development*.

| # | Tema | Estado |
|---|---|---|
| 01 | How LLMs work: lo necesario para usarlos bien | ⬜ |
| 02 | AI vs Traditional Coding: qué cambia y qué no | ⬜ |
| 03 | Embeddings y vectores | ⬜ |
| 04 | RAG: recuperación aumentada | ⬜ |
| 05 | Prompting Techniques | ⬜ |
| 06 | Herramientas: Claude Code, Cursor, Copilot y similares | ⬜ |
| 07 | Agentes, MCP y skills | ⬜ |
| 08 | Applications: code review, refactoring y documentación | ⬜ |
| 09 | Límites, alucinaciones y revisión del código generado | ⬜ |

### `20-Funcionalidades-con-IA`
El nodo *Building AI-powered features*.

| # | Tema | Estado |
|---|---|---|
| 01 | Integrar un proveedor: Anthropic, OpenAI, Gemini | ⬜ |
| 02 | Function Calling y uso de herramientas | ⬜ |
| 03 | Structured Outputs y validación de respuestas | ⬜ |
| 04 | Streaming de respuestas | ⬜ |
| 05 | Bases de datos vectoriales y búsqueda semántica | ⬜ |
| 06 | Coste, latencia, límites de tasa y caché | ⬜ |
| 07 | Evaluación y pruebas de features con LLM | ⬜ |
| 08 | Seguridad: inyección de prompt y datos sensibles | ⬜ |

---

## Orden sugerido

`01` → `03` → `02` → `04` → `05` → `06` → `07` → `10` → `08` → `09` → `11` → `12` → `13` → `18` → `15` → `16` → `14` → `17` → `19` → `20`

Cuatro desviaciones respecto al roadmap, a propósito:

- **Control de versiones (`03`) casi al principio.** Es la herramienta con la que vas a estudiar todo lo demás.
- **Testing (`10`) mucho antes de su posición en el roadmap.** No depende de contenedores, colas ni arquitectura, y cambia cómo estudiás todo lo posterior.
- **Observabilidad (`18`) justo después de contenedores.** Desplegar algo que no podés observar es la forma más cara de aprender.
- **Escalado de datos (`14`) y arquitectura (`17`) al final.** Son decisiones que solo tienen sentido cuando ya entendés el sistema que estás escalando.

---

## Solapamiento con otras carpetas del repositorio

El criterio es: **aquí el concepto general, en la carpeta de la tecnología el cómo concreto.**

| Tema | Aquí | En su carpeta |
|---|---|---|
| Testing | `10` — tipos de test, dobles, pirámide | `Pura/Java/13-Testing` — JUnit, Mockito · `SpringBoot/pura/08-Testing` — slices, MockMvc |
| Autenticación | `06` — OAuth2, JWT, sesiones | `SpringBoot/pura/07-Spring-Security` — cadena de filtros |
| Bases de datos | `04`, `13`, `14` — SQL, ACID, índices | `Pura/Java/15-Acceso-a-Datos` · `SpringBoot/pura/06-Acceso-a-Datos` |
| Caching | `08` — patrones e invalidación | `SpringBoot/pura/09` — `@Cacheable` |
| Microservicios | `17` — límites, datos, fallos | `SpringBoot/pura/10` — Eureka, Gateway, Feign |
| Mensajería | `15` — Kafka, RabbitMQ, entrega | `SpringBoot/pura/09` — Spring for Kafka |
| Observabilidad | `18` — los tres pilares, SLOs | `SpringBoot/pura/11` — Actuator, Micrometer |
