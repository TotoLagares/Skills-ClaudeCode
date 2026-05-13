---
name: sdd-reverse
description: Generate retroactive Spec-Driven Development documentation for an existing codebase. Reads a project's source code, infers architecture and behavior, and produces two complementary specs — a functional spec (what the system does, for whom, and why) and a technical spec (how it's built) — plus an interactive HTML viewer to browse them. Use this skill whenever the user asks for "specs from existing code", "reverse SDD", "document my project", "spec-driven docs for my repo", "retroactive specs", or wants to apply Spec-Driven Development to a project that already has code. Trigger even if the user only says "generate specs for this project" or "create SDD documentation" without mentioning reverse — if there's an existing codebase, this is the right skill. Outputs are markdown files plus a single self-contained HTML viewer.
---

# sdd-reverse

Genera documentación SDD (Spec-Driven Development) retroactiva para un proyecto que ya tiene código. Produce dos specs separadas (funcional y técnica) más un viewer HTML interactivo.

## Filosofía del skill

**Inferir, no preguntar.** El código es la fuente de verdad. El skill explora el repo, deduce todo lo que puede, y solo le pregunta al usuario cuando algo es genuinamente ambiguo y crítico (típicamente: dominio de negocio, audiencia, decisiones que el código no revela). Lo que se infiere se marca con `[INFERRED]`. Lo que es asunción riesgosa se marca con `[ASSUMPTION]` para que el usuario lo valide después.

**Dos specs, un corte limpio.** La regla de división es:
- **Funcional** = comportamiento observable. Si un cambio de código altera lo que un usuario o stakeholder percibe, va acá.
- **Técnica** = implementación. Si un cambio es invisible desde afuera (refactor, cambio de lib, optimización), va acá.

Si dudás dónde poner algo, preguntate: *¿le importa a un PM o solo a un dev?* PM → funcional. Dev → técnica.

## Workflow

Seguí estos pasos en orden. No saltees fases.

### Fase 1 — Reconocimiento del repo

Objetivo: entender qué tipo de proyecto es antes de leer nada en detalle.

1. Listar la raíz del repo y identificar:
   - Tipo de proyecto (manifests: `package.json`, `pyproject.toml`, `Cargo.toml`, `pom.xml`, `go.mod`, `Gemfile`, etc.)
   - Stack principal (lenguaje, framework si es evidente)
   - Presencia de README, docs/, ADRs, CHANGELOG
   - Estructura de carpetas top-level

2. Leer el README si existe — es la pista más rápida sobre el propósito del proyecto.

3. Identificar entrypoints:
   - Backend: `main.py`, `app.py`, `index.js`, `server.ts`, `cmd/*/main.go`, etc.
   - CLI: scripts en `bin/`, comandos en manifests
   - Frontend: `src/main.*`, `pages/`, `app/`

4. Construir un **inventario mental** de qué leer en profundidad. NO leas todo el repo — apuntá a:
   - Entrypoints (1-3 archivos)
   - Configuración (1-2 archivos)
   - Modelos/schemas de datos
   - Rutas/endpoints/handlers (si es un servicio)
   - Tests principales (revelan comportamiento esperado)

### Fase 2 — Lectura dirigida

Leé los archivos del inventario. Mientras leés, mantené en mente dos pilas de notas separadas:

**Pila funcional** — todo lo que sea comportamiento, reglas, casos de uso, validaciones de negocio, mensajes al usuario, estados visibles.

**Pila técnica** — todo lo que sea estructura, patrones, decisiones de stack, dependencias, contratos internos, deployment.

Ejemplo: un endpoint `POST /orders` con validación de stock.
- Funcional: "el sistema permite crear órdenes; rechaza la creación si no hay stock suficiente"
- Técnica: "endpoint REST POST /orders, valida con `OrderSchema`, llama a `InventoryService.check()`, persiste vía `OrderRepository`"

Si el repo es grande (>50 archivos relevantes), agrupá por módulo/feature antes de leer. No intentes leer todo.

### Fase 3 — Identificar gaps críticos

Antes de escribir, hacé una pasada mental: ¿qué NO te dice el código que sí necesita una spec?

Gaps típicos que requieren preguntar al usuario:
- **Audiencia / usuarios reales** — el código no te dice si es B2B, interno, consumer
- **Problema de negocio que resuelve** — el "para qué" del proyecto
- **Decisiones explícitas vs accidentes** — ¿por qué este stack? ¿fue elección o herencia?
- **Estado del proyecto** — ¿producción, MVP, legacy en mantenimiento?

**Regla de las preguntas críticas:** hacé MÁXIMO 3-5 preguntas, todas juntas en un solo turno. Solo preguntá lo que no podés inferir y que cambia sustancialmente el contenido de las specs. Si podés inferirlo razonablemente, NO preguntes — marcá `[INFERRED]` y seguí.

### Fase 4 — Generar las specs

Usá los templates en `templates/functional-spec.md` y `templates/technical-spec.md`. Leé ambos templates antes de escribir para entender la estructura completa.

**Reglas de redacción:**
- Prosa clara y directa, oraciones cortas. Evitá jerga innecesaria.
- En la spec funcional: hablá de comportamiento y usuarios, no de clases ni archivos.
- En la spec técnica: sé concreto sobre archivos, módulos, librerías y versiones cuando sea relevante.
- Marcá `[INFERRED]` lo deducido del código. Marcá `[ASSUMPTION]` lo que asumiste sin evidencia fuerte. Marcá `[CONFIRMED]` lo que el usuario te confirmó explícitamente.
- Si una sección no aplica al proyecto, escribí "N/A" con una línea explicando por qué — no la borres.

**Output paths:**
- `/mnt/user-data/outputs/functional-spec.md`
- `/mnt/user-data/outputs/technical-spec.md`

### Fase 5 — Viewer HTML

Generá un viewer HTML único e interactivo que renderice ambas specs lado a lado con navegación. Usá `assets/viewer-template.html` como base — leé ese archivo y reemplazá los placeholders con el contenido de las specs convertido a HTML.

El viewer debe:
- Tener dos tabs (Funcional / Técnica) o un layout split
- Tener una sidebar con tabla de contenidos clickeable
- Resaltar visualmente los marcadores `[INFERRED]`, `[ASSUMPTION]`, `[CONFIRMED]`
- Ser un archivo HTML único, sin dependencias externas (todo inline: CSS, JS, contenido)
- Funcionar abriendo el archivo directamente en el navegador (no requiere servidor)

Output path: `/mnt/user-data/outputs/sdd-viewer.html`

### Fase 6 — Entregar

Presentá los tres archivos al usuario con `present_files`, en este orden:
1. `sdd-viewer.html` (es lo más visual, va primero)
2. `functional-spec.md`
3. `technical-spec.md`

Acompañá la entrega con un resumen breve (máximo 4-5 líneas) que incluya:
- Qué tipo de proyecto detectaste
- Cuántas asunciones quedaron marcadas para revisar
- Qué archivos clave leíste
- Sugerencia de próximo paso (típicamente: revisar `[ASSUMPTION]` y confirmarlas)

NO escribas un resumen extenso — el usuario va a abrir el viewer.

## Edge cases

- **Repo vacío o casi vacío:** decile al usuario que no hay suficiente código para SDD reverso y sugerile el flujo forward (specs primero).
- **Monorepo:** preguntá cuál subproyecto documentar. No documentes todos a menos que el usuario lo pida explícitamente.
- **Código en múltiples lenguajes:** documentá el principal y mencioná los otros como "componentes auxiliares" en la spec técnica.
- **Sin README ni docs:** dependés 100% del código. Marcá agresivamente con `[INFERRED]` y `[ASSUMPTION]`.
- **Tests ausentes:** mencionalo en la spec técnica como gap conocido.

## Archivos del skill

- `templates/functional-spec.md` — plantilla de la spec funcional con todas las secciones requeridas
- `templates/technical-spec.md` — plantilla de la spec técnica con todas las secciones requeridas
- `assets/viewer-template.html` — template del viewer HTML interactivo
