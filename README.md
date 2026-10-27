# Kaizen — specs y reglas

Repo de arquitectura de Kaizen: aquí viven las reglas del proyecto y las especificaciones
(SDD, *Spec-Driven Development* con [Spec Kit](https://github.com/github/spec-kit)).
**No hay código de la aplicación**: el código vive en cuatro repos independientes que este repo
ignora (`.gitignore`).

## Qué hay aquí

| Ruta | Qué es |
|---|---|
| `.specify/memory/constitution.md` | La constitución: principios que prevalecen sobre todo lo demás |
| `specs/NNN-nombre/` | Un spec por funcionalidad (`spec.md`, luego `plan.md` y `tasks.md`) |
| `specs/001-kaizen-baseline/` | Línea base: cómo se comporta Kaizen hoy |
| `CLAUDE.md` | Guía operativa del workspace (arquitectura, dominios, comandos) |
| `.specify/` | Plantillas y scripts de Spec Kit |
| `.claude/skills/` | Los comandos `/speckit-*` para Claude Code |
| `constitucion.md` | Borrador original de la constitución (histórico) |

## Montar el workspace

Clona este repo y, dentro, los cuatro repos de código:

```bash
git clone https://github.com/snsilvam/kaizen-architecture.git kaizen-app
cd kaizen-app
git clone https://github.com/snsilvam/kaizensnsilvam-backend.git
git clone https://github.com/snsilvam/front-kaizensnsilvam.com.git
git clone git@github.com:snsilvam/kaizen-mobile.git
git clone https://github.com/snsilvam/kaizen-cloud-functions.git gcp-cloud-functions
```

| Carpeta | Qué es |
|---|---|
| `kaizensnsilvam-backend/` | API REST en Go |
| `front-kaizensnsilvam.com/` | SPA web en React + Vite |
| `kaizen-mobile/` | App Android (React Native + Expo) |
| `gcp-cloud-functions/` | Experimentos con Cloud Functions (fuera del flujo principal) |

No son submódulos a propósito: cada repo se versiona y despliega por su cuenta.

## Flujo SDD

Abre Claude Code en la raíz del workspace (no dentro de un repo) y sigue el orden:

1. `/speckit-specify <funcionalidad>` — crea `specs/NNN-nombre/spec.md`: qué y por qué, sin
   tecnología.
2. `/speckit-clarify` — opcional, resuelve ambigüedades del spec.
3. `/speckit-plan` — `plan.md`: cómo, qué repos toca y el contrato REST. Pasa el Constitution
   Check.
4. `/speckit-tasks` — `tasks.md`: tareas ordenadas, cada una con la ruta en su repo.
5. `/speckit-analyze` — opcional, revisa que spec, plan y tareas sean coherentes.
6. `/speckit-implement` — ejecuta las tareas en los repos de código.

Reglas:

- **Un solo Spec Kit, en la raíz.** Un spec por funcionalidad aunque toque varias capas; las
  capas aparecen en `plan.md` y `tasks.md`. No se hace `specify init` por repo.
- **Sin spec no hay código** (constitución, principio IV). Si un cambio contradice la
  constitución o un spec, primero se actualiza el spec.
- **Enlace spec ↔ código:** en cada repo tocado, la rama se llama como el spec
  (`NNN-nombre`) y el commit o PR menciona `specs/NNN-nombre`.
- **Cambiar las reglas:** `/speckit-constitution`, con versión SemVer en la constitución.

## Commits en este repo

Aquí solo se commitean specs, la constitución, `CLAUDE.md` y la configuración de Spec Kit.
Los commits de código van en su repo.
