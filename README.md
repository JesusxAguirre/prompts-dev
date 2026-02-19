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


## Database Design & Architecture - Production SQL Server

```
Diseñá schemas de base de datos production-ready siguiendo principios pragmáticos de integridad, performance y mantenibilidad. Prioriza claridad sobre "cleverness". El schema debe sobrevivir 10+ años.

### Filosofía Core

**Schema is Contract**
- DB dura 10+ años, código cambia cada 6 meses
- Prioriza data integrity sobre app-layer validation
- FK constraints son la última línea de defensa
- Si algo puede ser FK, DEBE ser FK

**Auditabilidad por Defecto**
- Temporal Tables (SYSTEM_VERSIONING = ON) para historial automático
- Soft deletes con deleted_at (nunca DELETE físico)
- created_by, created_at, updated_at obligatorios en transaccionales

**Pragmatismo sobre Pureza**
- Nombres que un PM entienda sin preguntar
- Respeta legacy naming (breaking changes cuestan más que nombres "feos")
- Junior DBA debe entender schema en 10 minutos sin docs

---

### Naming Conventions

**Prefijos de tabla:**
- ma_* → Master/catálogos (seed data, cambios lentos)
- operational_* → Transaccionales (eventos, alta frecuencia de cambios)

**Primary Keys:**
- SIEMPRE id (no table_name_id)
- Razón: Schema ya provee contexto (profiles.id es obvio)
- Excepción: Legacy tables con naming inconsistente (respetar)

**Foreign Keys:**
- Usa nombre real: campaign_id (no generic_id)
- FK autodocumentado apunta a qué tabla sin ver constraints

**Timestamps:**
- created_at → DATETIME2(7) UTC para API/logs
- updated_at → DATETIME2(7) app-managed (puede ser local time)
- updated_at_sys → GENERATED ALWAYS (system-managed, NO tocar)
- valid_until → GENERATED ALWAYS (temporal versioning)

**Soft Deletes:**
- deleted_at DATETIME2(7) NULL (NO is_deleted BIT)
- Razón: Auditable (cuándo se borró, no solo si/no)
- Index obligatorio: WHERE deleted_at IS NULL (filtered)

---

### Foreign Keys: Data Integrity NO NEGOCIABLE

**Anti-Pattern: Polymorphic Associations**
- ❌ zone_type + zone_id (imposible crear FK real)
- Problemas: orphan records, app validation bypasseable, queries complejas
- ✅ Usar: 4 FKs explícitos + XOR constraint
- Beneficio: DB garantiza integridad, queries simples, errores claros

**Anti-Pattern: ENUM como FK innecesario**
- ❌ FK a tabla de 5 filas fijas (JOIN extra en todas queries)
- ✅ Usar: NVARCHAR + CHECK constraint
- Excepción: Estados dinámicos o necesitas metadata (color, icon, sort_order)

**XOR Constraint Pattern:**
- Cuando exactamente UNO de N FKs debe ser NOT NULL
- CHECK con CASE sums = 1
- Combina con filtered indexes (WHERE column IS NOT NULL)

---

### Performance: Indexes Estratégicos

**Filtered Indexes (nullable FKs):**
- WHERE column IS NOT NULL
- Razón: 75% reducción de espacio cuando solo 25% de filas usan FK
- Obligatorio en XOR patterns

**Composite Indexes (bucket queries):**
- Cubre WHERE + GROUP BY + ORDER BY en una pasada
- Index Seek > Index Scan
- Ejemplo: (project_id, status, created_at) para dashboard queries

**Covering Indexes (INCLUDE):**
- Usar CON CUIDADO (infla tamaño)
- Solo para queries críticas frecuentes
- INCLUDE columnas SELECTeadas pero no filtradas

**Index Naming:**
- ix_table_columns (descriptivo)
- ix_table_deleted para soft delete filtered indexes

---

### Temporal Tables: Auditabilidad Gratis

**Setup obligatorio en transaccionales:**
- updated_at_sys GENERATED ALWAYS AS ROW START
- valid_until GENERATED ALWAYS AS ROW END (HIDDEN)
- PERIOD FOR SYSTEM_TIME
- WITH SYSTEM_VERSIONING = ON (HISTORY_TABLE = schema.ht_table)

**Qué hace:**
- Cada UPDATE mueve versión vieja a ht_* con valid_until = NOW()
- Nueva versión queda con valid_until = 9999-12-31 (= activa)
- Query histórico: FOR SYSTEM_TIME AS OF / ALL

---

### Documentation: Regla R015

**sp_AddExtendedProperty OBLIGATORIO:**
- Column descriptions en TODAS las columnas
- Table description con propósito de negocio
- Beneficio: SSMS muestra tooltips, DBA entiende sin docs externas
- Formato: nivel de detalle que explique qué, por qué, cuándo usar

---

### Constraint Naming

**Patterns consistentes:**
- pk_table → Primary Key
- fk_table_column → Foreign Key
- ck_table_description → Check Constraint
- df_table_column → Default Constraint
- uq_table_column → Unique Constraint
- ix_table_columns → Index

**Razón:**
- Error messages auto-explicativos
- Troubleshooting rápido sin buscar en scripts

---

### Normalización vs Denormalización

**Normalizar (default OLTP):**
- Transaccionales con FK constraints
- Relaciones 1:N reales
- Data integrity crítica

**Denormalizar (con cuidado):**
- Agregados calculados lentos (computed columns PERSISTED)
- Snapshots históricos (evitar JOINs a tablas cambiantes)
- Reports/analytics (performance > consistency)

**NUNCA duplicar:**
- Columnas que existen en tabla relacionada vía FK
- Usar JOINs o computed columns

---

### Testing Pre-Deploy

**Checklist:**
- FK Integrity: Insertar FK inválido (debe fallar)
- CHECK Constraints: Valor fuera de ENUM (debe fallar)
- XOR Constraints: 2 valores o 0 valores (debe fallar)
- Temporal Tables: UPDATE y verificar historial en ht_*
- Soft Delete: UPDATE deleted_at y queries con/sin filtro

---

### Naming: Inglés vs Español

**Híbrido pragmático:**
- Inglés para conceptos universales (PERMANENT, FIXED_TERM, Draft, Published)
- Español donde negocio lo exige (legacy tables, términos legales sin equivalente)
- Column descriptions en español si audiencia es 100% hispanohablante

---

### Golden Rules

1. Schema is Contract - DB dura 10+ años
2. Constraints > Validation - DB valida, app sugiere
3. Explicit > Implicit - FK explícitos, no polymorphic
4. Audit by Default - Temporal tables + soft delete
5. Index for Reads, Constrain for Writes
6. Document Everything - sp_AddExtendedProperty es tu amigo
7. Pragmatism > Purity - Respetar legacy naming

---

### Output Esperado

Generá:
1. DDL Scripts (separados por tabla)
2. Diagrama ER (Mermaid/DBML)
3. Migration Plan (orden ejecución + rollback)
4. Test Cases (queries validación constraints)
5. Performance Analysis (índices y queries esperadas)
6. Decisions Log (tabla con decisión, razón, trade-offs)

---

### Referencias

- Bill Karwin - "SQL Antipatterns" (Ch 6: Polymorphic Associations)
- C.J. Date - "Database Design and Relational Theory"
- Joe Celko - "SQL for Smarties" (Ch 3: Naming)
- Martin Fowler - "PEAA" (Identity Field, FK Mapping)
- Microsoft Docs - Temporal Tables, Filtered Indexes
```

