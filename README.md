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


**Why**: Estos patterns evitan los errores más comunes de integridad y performance en producción.
**Where**: pidb-ddl/scripts/ — cualquier archivo .sql de migración o DDL
**Learned**:
--- TEMPORAL TABLE (Auditabilidad automática) ---
```sql
updated_at_sys DATETIME2(7) GENERATED ALWAYS AS ROW START NOT NULL,
valid_until    DATETIME2(7) GENERATED ALWAYS AS ROW END HIDDEN NOT NULL,
PERIOD FOR SYSTEM_TIME (updated_at_sys, valid_until)
-- Al final de CREATE TABLE:
WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.ht_tabla_nombre));
```
Query histórico: SELECT * FROM tabla FOR SYSTEM_TIME AS OF '2025-01-01'
--- XOR CONSTRAINT (exactamente 1 de N FKs debe ser NOT NULL) ---
```sql
campaign_id     INT NULL,
zone_id         INT NULL,
project_id      INT NULL,
client_id       INT NULL,
CONSTRAINT ck_table_xor_zone CHECK (
    (CASE WHEN campaign_id IS NOT NULL THEN 1 ELSE 0 END +
     CASE WHEN zone_id     IS NOT NULL THEN 1 ELSE 0 END +
     CASE WHEN project_id  IS NOT NULL THEN 1 ELSE 0 END +
     CASE WHEN client_id   IS NOT NULL THEN 1 ELSE 0 END) = 1
)
-- Filtered indexes para XOR (performance):
CREATE INDEX ix_table_campaign ON table(campaign_id) WHERE campaign_id IS NOT NULL;
CREATE INDEX ix_table_zone     ON table(zone_id)     WHERE zone_id IS NOT NULL;
```
--- ENUM como CHECK (tabla de 5 filas fijas, NO FK) ---
```sql
status NVARCHAR(20) NOT NULL DEFAULT 'Draft'
    CONSTRAINT ck_table_status CHECK (status IN ('Draft','Published','Archived','Deleted'))
```
--- SOFT DELETE con filtered index ---
```sql
deleted_at DATETIME2(7) NULL CONSTRAINT df_table_deleted_at DEFAULT NULL,
CREATE INDEX ix_table_deleted ON table(deleted_at) WHERE deleted_at IS NULL;
-- Query activos siempre: WHERE deleted_at IS NULL
```
--- EXTENDED PROPERTIES (documentación en DB) ---
```sql
EXEC sp_AddExtendedProperty
    @name = N'MS_Description',
    @value = N'Descripción clara del propósito de esta columna',
    @level0type = N'SCHEMA', @level0name = N'dbo',
    @level1type = N'TABLE',  @level1name = N'nombre_tabla',
    @level2type = N'COLUMN', @level2name = N'nombre_columna';
```
--- COMPOSITE INDEX para dashboard queries ---
```sql
CREATE INDEX ix_table_project_status_created 
    ON table(project_id, status, created_at)
    INCLUDE (column_a, column_b);  -- INCLUDE solo para queries críticas frecuentes
```
--- CHECKLIST PRE-DEPLOY ---
- FK Integrity: insertar FK inválido → debe FALLAR
- CHECK Constraints: valor fuera de ENUM → debe FALLAR
- XOR Constraints: 2 valores o 0 valores → debe FALLAR
- Temporal Tables: UPDATE → verificar historial en ht_*
- Soft Delete: queries con/sin WHERE deleted_at IS NULL, project=pidb-ddl, title=DB SQL Patterns — Temporal Tables, XOR, Soft Delete, Extended Properties, topic_key=schema/sql-patterns, type=pattern]

