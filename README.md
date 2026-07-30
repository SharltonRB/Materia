# Materia

Base de conocimiento personal de backend. No es un proyecto de software: es material de estudio y repaso, escrito con un objetivo concreto — **cubrir la distancia entre junior, mid y senior**.

Esa distancia no se cruza aprendiendo más sintaxis. Se cruza entendiendo *por qué* se elige una solución y *qué se sacrifica* al elegirla. Por eso cada tema termina en un archivo de trade-offs: es el contenido que separa a quien sabe usar una herramienta de quien sabe cuándo no usarla.

---

## Cómo está organizado

```
Materia/
├── Pura/
│   ├── Java/                       # Tecnología grande → se divide en bloques
│   │   ├── README.md               # Índice del roadmap de esa tecnología
│   │   └── 01-Basics/
│   │       ├── 01-basic-syntax.md  # Un archivo por tema puntual
│   │       └── 02-variables-y-tipos.md
│   └── Caching/                    # Tema acotado → archivos directos dentro
│       ├── 01-fundamentos.md
│       └── 02-practico.md
└── Cheat sheets/                   # Referencia rápida, destilada DESPUÉS de Pura/
```

Dos formas según el tamaño del tema:

- **Tecnología grande** (Java, PostgreSQL, Docker): `Pura/<Tech>/<bloque>/<tema>.md`. Los bloques van numerados para conservar el orden del roadmap, y nacen solo cuando se escribe su primer tema.
- **Tema acotado** (un patrón, un concepto suelto): `Pura/<Tema>/` con los archivos directamente dentro.

### Los cuatro niveles

La progresión junior→senior está siempre presente. Cambia solo su soporte, según la granularidad:

| Nivel | Qué contiene | En tema acotado | En tema puntual |
|---|---|---|---|
| Junior | Qué problema resuelve, modelo mental, vocabulario | `01-fundamentos.md` | `## Fundamentos` |
| Junior → Mid | Uso real, ejemplos, errores comunes | `02-practico.md` | `## En la práctica` |
| Mid → Senior | Internals, rendimiento, casos límite | `03-avanzado.md` | `## Avanzado` |
| Senior | Cuándo NO usarlo, alternativas, preguntas de entrevista | `04-tradeoffs-y-entrevista.md` | `## Trade-offs y entrevista` |

Hay una segunda progresión, transversal: el propio orden de los bloques. `01-Basics` es territorio junior; concurrencia, JVM e internals son territorio mid-senior.

En temas muy elementales las secciones `Avanzado` y `Trade-offs` serán cortas. No se rellenan con paja para cumplir la plantilla: si no hay sustancia real en ese nivel, se dice y se sigue.

### Convenciones

- **Idioma:** español, con los términos técnicos en inglés (*connection pooling*, *race condition*, *eventual consistency*). Así es como aparecen en la documentación oficial y en las entrevistas.
- **Cheat sheets:** se generan a partir de un tema ya escrito en `Pura/`, nunca antes. Evita que se desincronicen.
- **Carpetas:** se crean cuando se empieza a escribir el tema. Git no versiona carpetas vacías, así que crearlas todas de golpe no serviría de nada.

---

## Mapa de temas

Las etapas son un orden sugerido de estudio, no una jerarquía de carpetas: dentro de `Pura/` todas las carpetas están al mismo nivel.

### Etapa 0 — Fundamentos que no caducan

Lo que sigue siendo cierto cuando cambia el framework de moda. Se suele saltar, y es la razón más común por la que alguien se estanca en junior.

| Carpeta | Contenido |
|---|---|
| `Redes-y-HTTP` | TCP/IP, DNS, TLS, HTTP/1.1 vs 2 vs 3, métodos, status codes, headers, cookies |
| `Git` | Modelo de objetos, branching, rebase vs merge, resolución de conflictos, workflows |
| `Linux-y-Terminal` | Sistema de archivos, permisos, procesos, señales, pipes, systemd |
| `Algoritmos-y-Estructuras-de-Datos` | Complejidad Big-O, arrays, hash maps, árboles, grafos, sorting |
| `Concurrencia` | Procesos vs threads, race conditions, deadlocks, locks, async I/O, event loop |

### Etapa 1 — Lenguaje y calidad de código

| Carpeta | Contenido |
|---|---|
| [`Java`](Pura/Java/README.md) | Lenguaje principal, a fondo. Tipado, memoria, ecosistema · **en progreso** |
| `POO` | Encapsulación, herencia vs composición, polimorfismo, cuándo la POO estorba |
| `Programacion-Funcional` | Inmutabilidad, funciones puras, efectos secundarios, map/filter/reduce |
| `Clean-Code` | Naming, funciones pequeñas, comentarios que envejecen mal, code smells |
| `Manejo-de-Errores` | Excepciones vs valores de retorno, errores tipados, fallar rápido, no silenciar |
| `Testing` | Unitarios, integración, E2E, dobles de prueba, pirámide de tests, cobertura útil |
| `TDD` | Red-Green-Refactor, diseño guiado por tests, sus límites reales |

### Etapa 2 — Datos

Donde más se nota el salto a mid. La mayoría de los problemas de rendimiento en producción son problemas de base de datos.

| Carpeta | Contenido |
|---|---|
| `SQL` | Joins, agregaciones, subqueries, window functions, CTEs |
| `PostgreSQL` | MVCC, tipos, EXPLAIN ANALYZE, vacuum, extensiones |
| `Modelado-de-Datos` | Normalización y cuándo desnormalizar, claves, relaciones, migraciones |
| `Transacciones` | ACID, niveles de aislamiento, anomalías, locks, deadlocks |
| `Indices` | B-tree, hash, parciales, compuestos, por qué un índice puede empeorar todo |
| `ORMs` | Mapeo objeto-relacional, N+1, lazy vs eager loading, cuándo bajar a SQL |
| `Redis` | Estructuras de datos, TTL, persistencia, pub/sub, uso como caché vs como store |
| `MongoDB` | Modelo de documentos, agregaciones, cuándo NoSQL es la elección equivocada |

### Etapa 3 — APIs

| Carpeta | Contenido |
|---|---|
| `REST` | Recursos, verbos, status codes, HATEOAS, paginación, filtrado, versionado |
| `GraphQL` | Schema, resolvers, N+1 y DataLoader, cuándo compensa frente a REST |
| `gRPC` | Protobuf, streaming, contratos, uso en comunicación interna |
| `Autenticacion-y-Autorizacion` | Sesiones vs JWT, OAuth 2.0, OIDC, refresh tokens, RBAC vs ABAC |
| `Validacion-de-Entrada` | Validación en los límites del sistema, esquemas, sanitización |
| `Documentacion-de-APIs` | OpenAPI/Swagger, contract-first, versionado de contratos |

### Etapa 4 — Diseño y arquitectura

| Carpeta | Contenido |
|---|---|
| `SOLID` | Los cinco principios, con ejemplos de cuándo aplicarlos empeora el código |
| `Patrones-de-Diseño` | Creacionales, estructurales, de comportamiento; sobreingeniería |
| `Arquitectura-Hexagonal` | Puertos y adaptadores, inversión de dependencias, testabilidad |
| `Clean-Architecture` | Capas, regla de dependencia, entidades vs casos de uso |
| `DDD` | Lenguaje ubicuo, agregados, bounded contexts, cuándo NO hacer DDD |
| `Refactoring` | Catálogo de refactors, deuda técnica, refactor seguro con tests |

### Etapa 5 — Infraestructura y operación

Un mid escribe código que funciona. Un senior escribe código que se puede desplegar, observar y arreglar a las 3 de la mañana.

| Carpeta | Contenido |
|---|---|
| `Docker` | Imágenes, capas, multi-stage builds, redes, volúmenes, Compose |
| `CI-CD` | Pipelines, estrategias de despliegue (blue-green, canary), rollback |
| `Observabilidad` | Logs estructurados, métricas, tracing distribuido, alertas que no se ignoran |
| `Seguridad` | OWASP Top 10, inyección, XSS, CSRF, gestión de secretos, criptografía aplicada |
| `Servidores-Web` | Nginx, reverse proxy, load balancing, terminación TLS |
| `Cloud` | Modelo de responsabilidad compartida, cómputo, storage, redes, costes |

### Etapa 6 — Escala y sistemas distribuidos

| Carpeta | Contenido |
|---|---|
| `Caching` | Estrategias (cache-aside, write-through), invalidación, stampede, CDN |
| `Mensajeria` | Colas vs streams, Kafka, RabbitMQ, at-least-once, idempotencia |
| `Microservicios` | Descomposición, comunicación, fallos parciales, saga, cuándo el monolito gana |
| `Arquitectura-Event-Driven` | Eventos vs comandos, event sourcing, CQRS |
| `Sistemas-Distribuidos` | Teorema CAP, consistencia eventual, consenso, relojes, particiones |
| `Escalabilidad` | Vertical vs horizontal, sharding, réplicas, rate limiting, backpressure |
| `System-Design` | Método para diseñar sistemas en entrevistas y en la vida real |

### Etapa 7 — Lo que no es código

Nadie llega a senior solo escribiendo código. Esta etapa suele ser el techo real.

| Carpeta | Contenido |
|---|---|
| `Decisiones-Tecnicas` | ADRs, comparar alternativas, defender una decisión, reconocer cuándo fue mala |
| `Code-Review` | Qué buscar, cómo dar feedback, qué no bloquear |
| `Estimacion-y-Planificacion` | Descomponer trabajo, incertidumbre, negociar alcance |
| `Metodologias` | Agile, Scrum, Kanban, trunk-based development |

---

## Estado

| Etapa | Temas | Escritos |
|---|---|---|
| 0 — Fundamentos | 5 | 0 |
| 1 — Lenguaje y calidad | 7 | Java en progreso |
| 2 — Datos | 8 | 0 |
| 3 — APIs | 6 | 0 |
| 4 — Diseño y arquitectura | 6 | 0 |
| 5 — Infraestructura | 6 | 0 |
| 6 — Escala | 7 | 0 |
| 7 — No técnico | 4 | 0 |
| **Total** | **49** | **0** |
