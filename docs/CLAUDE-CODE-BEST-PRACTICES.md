# Token Optimization — Claude Code Best Practices

Técnicas probadas para mejorar el proceso de desarrollo obteniendo mejor resultado con menos tokens.

---

## 1. CLAUDE.md / AGENTS.md — Mantener bajo 200 líneas

El archivo se carga en **cada mensaje**. Cada línea extra = tokens constantes.

- Eliminar secciones duplicadas (ej: auto-invoke + skills reference = misma info dos veces)
- Mover boilerplate (build commands, credenciales) a archivos separados
- Eliminar meta-instrucciones que Claude ya hace por defecto
- Regla: si Claude ya lo hace bien sin la instrucción, elimínala o conviértela en hook

---

## 2. Context Window — `/compact` y `/clear`

| Técnica | Cuándo | Ahorro |
|---------|--------|--------|
| `/cost` | Antes de decidir si compactar o limpiar | Visibilidad del gasto real |
| `/clear` | Entre tareas no relacionadas | Evita arrastrar contexto inútil |
| `/compact Focus on [tarea]` | Cada 30-50 intercambios | 60-80% reducción de contexto |
| `/rewind` | Cuando Claude tomó un camino equivocado | Evita gastar tokens corrigiendo una dirección errónea |
| Reescribir prompt desde cero | Después de 2 correcciones fallidas | Más barato que seguir iterando |
| `/btw` | Preguntas rápidas de sintaxis, lookups | No contamina historial principal |

---

## 3. Modelo correcto para cada tarea

| Tarea | Modelo | Costo relativo |
|-------|--------|----------------|
| Arquitectura, debugging complejo, diseño | Opus | 1x (máximo) |
| Implementación, edición, CRUD | Sonnet (`/model sonnet`) | 0.6x |
| Exploración, búsqueda, tareas batch | Haiku (subagentes) | 0.2x |

Usar `/model sonnet` para implementación rutinaria. Reservar Opus para decisiones de arquitectura y debugging. Usar `/fast` cuando la latencia importa más que el costo (activa modo rápido en Opus sin cambiar el modelo).

---

## 4. Subagentes — Explorar sin contaminar contexto

Cada archivo que Claude lee se queda en el contexto principal. Los subagentes exploran en contexto separado y devuelven solo el resumen.

| Usar subagente | Usar herramientas directas |
|---|---|
| Investigar impacto cross-módulo | Editar un archivo específico |
| Buscar patrones en el codebase | Correr un comando conocido |
| Code review post-implementación | Lookup rápido de un tipo/función |
| Tareas que leen muchos archivos | Tareas donde necesitas el contexto después |

Se pueden definir agentes custom en `.claude/agents/` con modelo específico (ej: explorador en Sonnet, reviewer en Opus).

---

## 5. Hooks — Automatizar lo repetitivo

Convertir reglas de AGENTS.md en hooks reduce el tamaño del archivo Y garantiza ejecución.

| Hook | Trigger | Reemplaza |
|------|---------|-----------|
| Auto-format TS | PostToolUse en `Write\|Edit` para `.ts/.tsx` | "Corre format antes de finalizar" |
| Type-check pre-commit | PreToolUse en `Bash(git commit:*)` | Regla manual en AGENTS.md |
| Lint post-edit | PostToolUse en `Write\|Edit` | "Lint antes de finalizar" |

Cada regla → hook = menos texto en AGENTS.md + ejecución garantizada.

---

## 6. Prompts específicos > abiertos

| Malo (explora 50+ archivos) | Bueno (lee 1-2 archivos) |
|---|---|
| "Investiga el sistema de auth" | "Lee `src/auth/tokenRefresh.ts` y explica el refresh flow" |
| "Revisa el módulo de reservas" | "Qué hace `ReservationService.cancel()`?" |
| "Optimiza el rendimiento" | "Mide queries en `GET /api/sessions` y reduce N+1" |

---

## 7. MEMORY.md — Mantener limpio

- Límite práctico: ~200 líneas antes de truncamiento
- Cada ~10 sesiones, podar entradas redundantes
- Separar notas detalladas en archivos temáticos referenciados desde MEMORY.md
- Solo guardar: gotchas, patrones que previenen errores, preferencias del usuario
- NO guardar: estructura de archivos, git history, soluciones de debugging (ya están en el código)

---

## 8. Worktrees — Trabajo paralelo

- `claude --worktree` crea checkout aislado para sesiones paralelas
- Útil para: backend endpoint en un worktree, frontend component en otro
- Límite práctico: 3-5 worktrees antes de que el context-switching domine
- Crear `.worktreeinclude` con archivos locales necesarios (`.env`, configs)

---

## Resumen priorizado

| # | Acción | Esfuerzo | Ahorro estimado |
|---|--------|----------|-----------------|
| 1 | AGENTS.md bajo 200 líneas | 30 min | ~52% menos tokens base/mensaje |
| 2 | Sonnet para implementación | 0 min | ~40% menos costo |
| 3 | `/clear` entre tareas, `/compact` en sesiones largas | Hábito | ~30-50% por sesión |
| 4 | Subagentes para exploración | Hábito | Contexto principal limpio |
| 5 | Hooks de format/typecheck | 15 min | Menos reglas en AGENTS.md |
| 6 | `/btw` para lookups rápidos | Hábito | No contamina historial |
| 7 | Prompts específicos con paths | Hábito | Menos archivos leídos |
