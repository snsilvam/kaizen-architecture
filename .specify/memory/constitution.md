<!--
Sync Impact Report
- Version change: 1.0.0 → 1.1.0 (MINOR: guía ampliada; un reto de 90 días sigue siendo válido)
- Modified principles: I. Tres caminos fijos al servicio de tres preguntas — Winter Arc deja de
  ser "reto de 90 días" y pasa a durar lo que la persona elige (3 a 90 días) dentro de la temporada
- Added sections: ninguna
- Removed sections: ninguna
- Templates: no se tocan (leen la constitución al ejecutarse)
- Deuda técnica explícita: el backend fija la duración en `ChallengeDays = 90`
  (`kaizensnsilvam-backend/internal/winterarc/entity.go`) y las etapas de forja del front asumen
  90 días. Se resuelve con un spec nuevo que implemente FR-017, FR-019a y FR-025a de
  `specs/001-kaizen-baseline/spec.md`.
- Follow-up TODOs: ninguno
-->
# Kaizen Constitution

Kaizen es una herramienta de mejora personal continua: finanzas y hábitos personales para
usuarios autenticados, con la disciplina diaria del 1 % como idea central.

## Core Principles

### I. Tres caminos fijos al servicio de tres preguntas

- El sistema MUST manejar exactamente tres caminos, cada uno un módulo/herramienta distinta:
  **Finanzas**, **Hábitos** y **Winter Arc**. Agregar, quitar o fusionar caminos es una enmienda
  MAJOR de esta constitución.
- **Finanzas**: gestionar el dinero personal con disciplina; mejora continua del 1 % cada día.
  Toda funcionalidad de finanzas MUST responder al menos una de: ¿cuánto puedo gastar hoy?,
  ¿cuántos días faltan para mi próximo ingreso?, ¿voy bien o mal respecto a mi plan?
- **Hábitos**: gestionar hábitos personales; lo importante no es el hábito sino repetir la
  identidad cada día. Las funcionalidades MUST reforzar la repetición diaria.
- **Winter Arc**: reto de mejora personal; lo importante es repetir y cumplir el compromiso con
  uno mismo, no la duración. Cada reto dura los días que la persona elige al crearlo, de 3 a 90,
  y es vigente solo durante 2026: su último día MUST NOT pasar del 2026-12-31, cuando termina la
  temporada. Las funcionalidades MUST NOT extenderlo más allá de esa fecha sin enmienda.
- Montos en COP como enteros, sin decimales ni moneda en el backend.

Razón: un alcance fijo y pequeño evita que la herramienta se disperse.

### II. Herramienta, no web app

- Kaizen MUST sentirse como una herramienta diaria al estilo Duolingo: acciones de un toque,
  progreso visible (rachas, anillos, etapas), urgencia y recompensa por la constancia.
- Las pantallas MUST ser mobile-first y priorizar la acción de hoy sobre la consulta o la
  configuración.
- Una funcionalidad que solo agrega formularios o listados sin aportar al hábito diario MUST
  justificarse en su `spec.md`.

Razón: el valor está en volver cada día, no en administrar datos.

### III. Sesión activa e identidad desde el token (NON-NEGOTIABLE)

- Todo flujo MUST exigir un usuario con sesión activa (Firebase Auth); no hay pantallas ni
  endpoints de negocio anónimos (solo salud y documentación son públicos).
- La identidad MUST salir del ID token verificado en el backend; el `user_id` nunca viaja en
  body ni query, y toda consulta MUST filtrar por el UID autenticado.
- Antes de leer, editar o borrar un recurso por id, el service MUST verificar que pertenece al
  usuario.

Razón: son datos personales; un fallo aquí expone finanzas o hábitos de otra persona.

### IV. KISS, SOLID y cero código sombra

- La solución MUST ser la más simple que cumpla el `spec.md`: sin capas nuevas, reflexión,
  frameworks pesados, colas ni asincronía salvo que el spec lo pida.
- Se aplican SOLID con mesura: interfaces pequeñas del lado del consumidor, una responsabilidad
  por service/handler, dependencias inyectadas a mano en el composition root.
- **Cero código sombra**: se construye estrictamente lo documentado en `spec.md`. Nada "por si
  acaso" (pasarelas de pago, perfiles complejos, configuraciones no pedidas, refactors estéticos).
- Dependencias nuevas MUST justificarse en el plan.

Razón: proyecto personal para aprender; cada pieza extra es costo y deuda.

### V. Errores amigables y HTTP semántico

- **UI**: nunca se muestran errores crudos, stack traces ni mensajes técnicos. Todo error MUST
  traducirse a un mensaje amigable en español (p. ej. "La finanza pendiente ya fue pagada antes.").
- **Backend**: MUST responder códigos HTTP semánticos — 400 petición inválida, 401 no
  autenticado, 403 sin acceso, 404 no encontrado, 409 conflicto de estado — con el cuerpo
  `{"error": "mensaje"}`.
- Los errores de dominio MUST ser centinelas del dominio, mapeados a HTTP en el handler.
- Nunca se loguean secretos (`DATABASE_URL`, API keys, tokens).

Razón: el usuario debe saber qué pasó y qué hacer; el cliente debe poder decidir por el código.

## Stack tecnológico y estructura

- **Frontend web** (`front-kaizensnsilvam.com/`): React + Vite, TypeScript, Tailwind CSS para
  los estilos; desplegado en Cloud Run (nginx). Migrar a otro framework (p. ej. Next.js) es un
  cambio de arquitectura que requiere su propio spec.
- **Backend** (`kaizensnsilvam-backend/`): Go, API REST en Cloud Run. Es la única puerta a los
  datos; el frontend y el móvil consumen solo la API.
- **Cloud backend** (`gcp-cloud-functions/`): Cloud Functions Gen 2 en Go; experimental, fuera
  del flujo principal.
- **Base de datos**: PostgreSQL en una VM de GCP (red privada) para el flujo principal; Firestore
  solo en los experimentos de Cloud Functions. PostgreSQL se accede solo desde repositorios.
- **Mobile** (`kaizen-mobile/`): app Android en React Native + Expo, APK fuera de Play Store
  compilada con EAS Build; v1 = login con correo, caminos y dashboard de Finanzas en solo lectura.
- **Lenguajes**: TypeScript en toda la capa de presentación (web y móvil); Go en todo el backend.
- **Estructura plana**: un repositorio Git por capa, en la raíz del workspace, sin anidar
  proyectos ni crear carpetas organizativas vacías. Todo corre en el proyecto GCP
  `kaizensnsilvam`, región `us-central1`.

## Estilo de código y flujo SDD

- **Estilo**: programación funcional y componentes funcionales con hooks en React; clases solo
  si son obligatorias. En Go, código idiomático: paquetes por dominio, errores como valores,
  interfaces pequeñas.
- **Nomenclatura**: `camelCase` para funciones y variables, `PascalCase` para tipos, interfaces y
  componentes (en Go, la exportación sigue las reglas del lenguaje). Nombres de negocio
  (`RegisterIncome`, `MarkAsPaid`), sin sufijos técnicos. Textos de UI en español.
- **Fuente de la verdad**: el `spec.md` de cada funcionalidad. Si una instrucción contradice esta
  constitución o el spec tiene una falla lógica, el agente MUST detenerse, advertir el problema y
  pedir que se actualice el `spec.md` antes de tocar código.
- **Contratos**: si cambia el contrato REST, MUST actualizarse `docs/swagger.yaml` del backend y
  los `types/` + `services/` de cada cliente que use el recurso; se mantiene la compatibilidad
  hacia atrás de la API.
- **Calidad**: tests del backend en el service con dobles simples; el frontend MUST pasar
  `typecheck` (lo exige el CI).

## Governance

- Esta constitución prevalece sobre cualquier otra práctica. `CLAUDE.md` es la guía operativa
  del día a día y no puede contradecirla; si lo hace, gana la constitución y se corrige la guía.
- **Enmiendas**: se proponen con `/speckit-constitution`, quedan registradas en este archivo y,
  si invalidan código existente, MUST incluir un plan de migración o la deuda técnica explícita.
- **Versionado** (SemVer): MAJOR = principio eliminado o redefinido de forma incompatible (p. ej.
  cambiar los caminos); MINOR = principio o sección nuevos o ampliados; PATCH = redacción.
- **Cumplimiento**: cada `plan.md` MUST pasar el Constitution Check antes de implementar; toda
  complejidad que viole un principio MUST justificarse por escrito en el plan o eliminarse.

**Version**: 1.1.0 | **Ratified**: 2026-10-27 | **Last Amended**: 2026-10-27
