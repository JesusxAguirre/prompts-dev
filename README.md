# Prompts Recomendados para Desarrollo

Estos prompts los podés usar en cualquier proyecto. Copialos y adaptalos a tu código.


---

## Documentación para No-Ingenieros (Pragmatic Domain)

```
Actúa como un Technical Writer Senior con alma de Mentor. Tu especialidad es el "Pragmatic Communication": transformar sistemas complejos en documentación que un Product Owner entienda a la primera.

TAREA: Analiza el siguiente [MÓDULO/CÓDIGO] y genera documentación en Markdown. Si es extenso, divídelo en varios archivos.

ESTRUCTURA:
1. El "Qué y Por qué": ¿Qué problema resuelve esto para el negocio?
2. El Proceso: Explica el flujo como una historia, sin jerga (usa analogías).
3. Conceptos Clave: Glosario de términos de dominio común.
4. Impacto: ¿Qué ganamos con esto y qué pasa si falla?

REGLA DE ORO: Prohibido usar palabras como "array", "endpoint", "database" o "middleware". Habla de procesos de negocio.
```


---

## Code Review

```
Revisá este PR buscando:
- Memory leaks y problemas de performance
- Casos borde sin manejar
- Violaciones de principios SOLID
- Inconsistencias con el estilo del repo (adjunto style guide)
```

---

## Migración de Código

```
Migrá este componente de React 18 a React 19:
- Convertí useEffect innecesarios a server components donde corresponda
- Usá las nuevas APIs de React 19 (use, useOptimistic)
- Mantené funcionalidad exacta
- Explicá cada cambio importante
```

---

## Documentación

```
Generá documentación profesional para esta función:
- JSDoc completo (@param, @returns, @throws, @example)
- README con ejemplos de uso
- Diagrama de flujo en Mermaid
- Lista de edge cases cubiertos
```

---

## Optimización de Performance

```
Optimizá esta función O(n²):
- Reducir complejidad temporal a O(n) o O(n log n)
- Usar estructuras de datos apropiadas (Set, Map, WeakMap)
- Explicar la ganancia de performance
- Incluir benchmark comparativo simple
```

---

## Testing

```
Escribí tests para esta función:
- Unit tests con Jest/Vitest
- Casos de éxito y error
- Edge cases (null, undefined, arrays vacíos, valores límite)
- Mocks para dependencias externas
- Coverage mínimo 80%
- funciones core 100% , 80% en resto, 0% en infra
```

---

## Debugging

```
Este código tiene un bug: [describir el comportamiento inesperado]

Analizá:
1. Qué está pasando exactamente
2. Por qué ocurre el bug
3. Cómo solucionarlo
4. Cómo prevenir bugs similares en el futuro
```

---

## Refactoring General

```
Refactorizá este código aplicando:
1. Principios SOLID
2. Early returns para reducir nesting
3. Extracción de funciones pequeñas y reutilizables
4. Nombres descriptivos para variables y funciones
5. Manejo de errores robusto
6. TypeScript con tipos estrictos
```

---

## Seguridad

```
Revisá este código buscando vulnerabilidades:
- Inyección SQL/NoSQL
- XSS (Cross-Site Scripting)
- CSRF
- Exposición de datos sensibles
- Validación de inputs insuficiente
- Dependencias desactualizadas con CVEs conocidos
```

---

## API Design

```
Diseñá un endpoint REST para [funcionalidad]:
- Método HTTP correcto
- Path siguiendo convenciones REST
- Request body con validación
- Response con códigos HTTP apropiados
- Manejo de errores consistente
- Documentación OpenAPI/Swagger
```

---

## SQL / Queries

```
Optimizá esta query SQL:
- Reducir tiempo de ejecución
- Usar índices apropiados
- Evitar N+1 queries
- Explicar el plan de ejecución
- Sugerir índices a crear si es necesario
```

---

## Git / Commits

```
Generá un mensaje de commit para estos cambios siguiendo Conventional Commits:
- Tipo: feat/fix/refactor/docs/test/chore
- Scope opcional
- Descripción concisa en imperativo
- Body explicando el "por qué" si es necesario
- Breaking changes si aplica
```

---

## Arquitectura

```
Proponé una arquitectura para [sistema/feature]:
- Diagrama de componentes
- Flujo de datos
- Patrones de diseño a usar
- Consideraciones de escalabilidad
- Trade-offs de cada decisión
- Flujos y diagramas con MERMAID
```


## Database Design & Architecture
Diseñá un schema de base de datos production-ready para [SISTEMA/FEATURE].
### Contexto a proveer
- Motor de base de datos: [SQL Server / PostgreSQL / MySQL / otro]
- Tipo de carga: [OLTP / OLAP / mixto]
- Tablas involucradas: [listar]
- Relaciones conocidas: [describir]
- Volumen estimado: [filas/día, tamaño esperado]
---
### Filosofía aplicada
**Schema is Contract** — la DB dura 10+ años, el código cambia cada 6 meses.
Priorizar data integrity sobre app-layer validation. Los constraints son la última línea de defensa.
---
### Output esperado
1. **DDL Scripts** — separados por tabla, con constraints nombrados
2. **Diagrama ER** — en Mermaid o DBML
3. **Migration Plan** — orden de ejecución + rollback script
4. **Test Cases** — queries que validan cada constraint
5. **Decisions Log** — tabla con decisión, razón y trade-offs
---
### Reglas no negociables
**Naming:**
- PK siempre `id` (no `table_name_id`)
- FK con nombre real: `campaign_id`, no `entity_id`
- Constraints nombrados: `pk_`, `fk_`, `ck_`, `df_`, `uq_`, `ix_`
**Integridad:**
- Si algo puede ser FK, debe ser FK
- Sin polymorphic associations (`type` + `entity_id`) — usar FKs explícitas + XOR constraint
- ENUMs de catálogo fijo → CHECK constraint, no FK a tabla de 5 filas
**Auditabilidad (tablas transaccionales):**
- `created_at` — timestamp UTC, no editable
- `updated_at` — app-managed
- Soft delete: `deleted_at DATETIME NULL` (nunca `is_deleted BIT`, nunca DELETE físico)
- Temporal versioning si el motor lo soporta (SYSTEM_VERSIONING en SQL Server, histórico manual en Postgres)
**Performance:**
- Filtered index en `deleted_at` (WHERE deleted_at IS NULL)
- Filtered index en cada FK nullable de un XOR constraint
- Composite index para queries de dashboard: (project_id, status, created_at)
- INCLUDE solo en queries críticas frecuentes
**Documentación inline:**
- Comentario o extended property en cada columna
- Propósito de negocio en descripción de tabla
---
### Anti-patterns a rechazar
| ❌ Anti-pattern | ✅ Solución |
|---|---|
| Polymorphic: `type` + `entity_id` | FKs explícitas + XOR CHECK constraint |
| `is_deleted BIT` | `deleted_at DATETIME NULL` |
| DELETE físico en transaccionales | Soft delete siempre |
| FK a tabla de 5 filas fijas | CHECK constraint con valores permitidos |
| Columnas duplicadas de tabla relacionada | JOIN o computed column |
| Constraints sin nombre | Nombrado explícito siempre |
---
### Checklist pre-deploy
- [ ] FK Integrity: insertar FK inválido → debe FALLAR
- [ ] CHECK Constraints: valor fuera de rango → debe FALLAR  
- [ ] XOR Constraints: 0 valores o 2+ valores → debe FALLAR
- [ ] Soft Delete: query sin `WHERE deleted_at IS NULL` devuelve borrados
- [ ] Audit trail: UPDATE registra versión anterior
- [ ] Indexes: plan de ejecución usa Index Seek (no Scan) en queries principales
---
- CHECK Constraints: valor fuera de ENUM → debe FALLAR
- XOR Constraints: 2 valores o 0 valores → debe FALLAR
- Temporal Tables: UPDATE → verificar historial en ht_*
- Soft Delete: queries con/sin WHERE deleted_at IS NULL, project=pidb-ddl, title=DB SQL Patterns — Temporal Tables, XOR, Soft Delete, Extended Properties, topic_key=schema/sql-patterns, type=pattern]

