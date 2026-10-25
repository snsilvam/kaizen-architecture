# Kaizen — memoria del proyecto

Workspace con **cuatro repositorios Git independientes** (la carpeta raíz no es un repo).
Cada commit, branch y PR se hace dentro del repo que corresponda.

| Carpeta | Qué es | Stack |
|---|---|---|
| `kaizensnsilvam-backend/` | API REST | Go 1.25 · Gin · GORM · PostgreSQL · Firebase Admin |
| `front-kaizensnsilvam.com/` | SPA web | React 19 · TypeScript · Vite · Tailwind 4 · shadcn/base-ui · Firebase Auth |
| `kaizen-mobile/` | App Android (APK sin Play Store, compilada con EAS Build); v1: login con correo, caminos y dashboard de Finanzas en solo lectura. Ver su `README.md` | React Native 0.86 · Expo SDK 57 · Expo Router · Firebase JS SDK |
| `gcp-cloud-functions/` | Experimentos con Cloud Functions Gen 2 (Go); no forma parte del flujo principal | Go · Firestore · PostgreSQL |

Todo corre en el proyecto GCP `kaizensnsilvam`, región `us-central1`.

## Propósito de producto

Kaizen responde tres preguntas; toda funcionalidad nueva debe aportar a alguna:

1. ¿Cuánto dinero puedo gastar hoy?
2. ¿Cuántos días faltan para mi próximo ingreso?
3. ¿Voy bien o voy mal respecto a mi plan?

Tras el login el usuario elige entre dos "apps" que comparten backend y sesión:
**Kaizen** (finanzas) y **Kaizen Habits** (hábitos, `/habits`).
Montos en COP como enteros (`int64`), sin decimales ni moneda en el backend.

## Big picture

```
Firebase Auth ──ID token──▶ Frontend (Cloud Run, nginx :8080)
      ▲                          │ fetch + Authorization: Bearer <idToken>
      │ VerifyIDToken (ADC)      ▼
      └───────────────── Backend (Cloud Run :8080) ──VPC privada──▶ PostgreSQL (VM)
                               + Cloud Run Job `/app/migrate`
```

- El frontend consume **solo** la API REST. El backend nunca genera HTML (salvo Swagger UI embebido).
- La identidad sale **siempre del token**: el `user_id` nunca viaja en body ni query.
  Cada consulta filtra por el UID del usuario autenticado.

## Backend (`kaizensnsilvam-backend/`)

### Capas (Clean Architecture, KISS)

```
internal/http/router.go + middleware (CORS → Auth)
  → internal/http/handlers/<x>_handler.go   valida request, lee UID, llama al service. Sin lógica de negocio.
  → internal/<dominio>/service.go            reglas de negocio (use cases)
  → internal/<dominio>/repository.go         interfaz (Repository / Reader), definida en el dominio
  → internal/infrastructure/sql/<x>_repository.go   implementación GORM
  → PostgreSQL
```

Estructura de cada dominio: `entity.go` (struct con tags GORM/JSON + `Validate()`), `errors.go`
(errores centinela), `repository.go` (interfaz), `service.go`, `service_test.go`.

- **Composition root**: `cmd/api/main.go` cablea a mano repos → services → handlers → `router.New(...)`.
  Un handler nuevo implica tocar `main.go` y la firma de `router.New`.
- **Identidad**: `middleware.Auth` verifica el token y guarda `auth.Identity` en `context.Context`.
  El dominio lee `auth.UID(ctx)` / `auth.Of(ctx)` (paquete `internal/auth`), sin depender de Gin ni Firebase.
- **Interfaces pequeñas del lado del consumidor**: `dashboard` depende de `income.Reader` y
  `pending_payment.Reader`; `market` declara su propio `BudgetStore` (lo cumple el repo de pagos) y `CategoryReader`.
- `GetByID`/`Delete` de repos **no filtran por dueño**: el service debe verificar propiedad antes.
- Tiempo: los services que dependen de "hoy" reciben `now func() time.Time` para poder testear.
- Errores de API: siempre `{"error": "mensaje"}`.

### Dominios, tablas y endpoints

| Dominio (`internal/`) | Tablas | Endpoints |
|---|---|---|
| `user` | `users` (`ia_feature`: acceso al módulo de facturas, manual) | `POST /users/sync` (idempotente, sin body), `GET /users/me` (incluye `iaFeature`) |
| `income` | `incomes` | `GET/POST /incomes`, `DELETE /incomes/:id` |
| `pending_payment` | `pending_payments`, `pending_payment_categories` | `GET/POST /pending-payments`, `PATCH /:id/mark-as-paid`, `PATCH /:id/mark-as-market-budget`, `DELETE /:id`; `GET /pending-payment-categories` |
| `market` (lista de mercado) | `market_items` (FK → `pending_payments`, CASCADE; `quantity`, `estimated_unit_price`, `price` = precio real por unidad, `purchased`) | `GET/POST /market`, `GET/PUT/DELETE /market/:id`, `POST /market/:id/items`, `PUT/DELETE /market/:id/items/:itemId`, `PATCH /:itemId/mark-as-purchased` (`{price, quantity?}`), `PATCH /:itemId/mark-as-pending` |
| `dashboard` | — (orquesta income + pending_payment) | `GET /dashboard` |
| `habit1` | `habit1_records` | `GET/POST /habit-1` (sueño; horas en `"HH:MM"`) |
| `kaizen-habits` (pkg `kaizen_habits`) | `kaizen_habits`, `kaizen_habit_repetitions`, `kaizen_habit_goals` | `/kaizen-habits` CRUD + `/:habitId/repetitions`, `/calendar`, `/stats`, `/goals` |
| `winterarc` | `winter_arc_logs`, `winter_arcs`, `winter_arc_rules` (FK → `kaizen_habits`, RESTRICT), `winter_arc_days` (solo días sincronizados; el grid de 90 se arma en Go) (SQL crudo con `database/sql`, no GORM) | `POST /api/winter-arc/setup` (≥ 3 hábitos activos propios, un reto vigente a la vez; `commitment` = respuesta a "¿Cuál es tu compromiso con tu yo de enero?", columna `winter_arcs.commitment`, ≤ 500, opcional en el backend por compatibilidad y obligatorio en el modal del front; sale también en el grid), `GET /api/winter-arc/current` (grid del reto más reciente en cualquier estado; 404 si nunca hubo; el grid de `current` y `:id/grid` trae `schedule`: las reglas ordenadas por la hora `"HH:MM"` del hábito, sin hora al final, con `done_today` sólo si el reto está activo — el front lo pinta en `WinterArcSchedule` con el "ahora" de Bogotá), `GET /api/winter-arc/:id/grid`, `POST /api/winter-arc/:id/sync-today` (éxito = todas las reglas con repetición kaizen hoy; sync y cierre guardan el detalle por regla en `winter_arc_days.habit_execution_log` JSONB), `PUT /api/winter-arc/:id/rules` (`{habit_ids}` = lista completa, ≥ 3; editar qué hábitos entran y salen del reto activo, si no 409; los nuevos deben ser hábitos activos, una regla pausada puede quedarse; **vale desde hoy**: `UpdateRules` hace `settle` antes, así los días pasados se cierran con las reglas viejas, y re-sincroniza hoy si ya tenía fila; responde el grid; en el front, botón "Editar código" de `WinterArcSchedule` → `WinterArcRulesDialog`. El `habit_execution_log` de cada día es la foto de sus reglas: la deuda se paga con las que faltaron ese día (`missingHabits`) y la analítica suma las reglas que salieron (`evaluatedHabitIDs`); una regla quitada deja de estar protegida por la FK RESTRICT), `GET /api/winter-arc/:id/analytics` (solo `completed`/`failed`, si no 409; hábito más cumplido, éxito por día de la semana y Hábito Ancla del último tercio de los días cerrados), `POST /api/winter-arc/log`, `GET /api/winter-arc/today` — inmutable, solo escribe hoy; reto de 90 días; sin proceso nocturno: al leer el reto (grid, current, sync-today, setup) `Service.settle` cierra en orden los días pasados abiertos (`winter_arc_days.closed_at`; el sync lo rechaza con 409), contando solo repeticiones con `registered_at` < medianoche Bogotá del día, actualiza `winter_arcs.consecutive_failed_days` y con 3 → `status=failed`, día 90 vivo → `completed`. **Tregua** (`WINTER_ARC_GRACE_UNTIL`, `YYYY-MM-DD`; en el deploy, la variable de GitHub o `2026-12-31` por defecto; vacía en local = sin tregua): los días hasta esa fecha topan la racha en 2 y no pierden el reto; uno perdido dentro de la tregua se reabre al leerlo (`Repository.Reopen`, solo si es el más reciente del usuario). **Domingo libre** (`RestWeekday`): se cierra sin resultado, no suma fallo ni reinicia la racha, no deja deuda, no cuenta en analytics ni como forjado, y el plazo de la deuda no lo cuenta (`dueDate`: la del sábado vence el miércoles); sync-today en domingo no escribe. Sale de la fecha (`Day.RestDay`, `is_rest_day` en el grid), así que vale también para los domingos cerrados antes: la racha que juzga cada día se rehace con `failureStreak` sobre los días guardados, no con la columna. Los tests del handler fijan el reloj con `Service.UseClock`. **Deuda**: un día fallado se paga en sus 3 días siguientes (`DebtPaymentDays`) registrando, con la fecha del fallo, una repetición de cada regla que faltó (de cualquier origen, también el calendario de `/habits`); `settle` lo detecta (corte = medianoche del día en que se juzga, o ahora), marca `winter_arc_days.debt_paid` y recalcula la racha (`failureStreak`): cuenta como cumplido para los 3 fallos pero `is_successful`/log no cambian (no suma en analytics, pero el front lo cuenta como día forjado en `countForgedDays`); con el reto perdido o completado ya no se paga. El grid trae `debts` (`WinterArcDebts` en el front) |
| `invoice` (rama `feature/invoices`, sin desplegar a 2026-10-09) | `invoices` (ítems en JSONB; índice único parcial `(user_id, cufe) WHERE cufe <> ''`) | `GET/POST /invoices` (POST multipart, campo `image`), `GET /invoices/:id`, `DELETE /invoices/:id` |
| `sleep` | `sleep_records` (FK → `kaizen_habits`, CASCADE; único `(user_id, record_date)`; `wake_time`/`bed_time` `"HH:MM"`, `sleep_minutes` 1–1440) | `GET /sleep/today` (asegura el hábito `Habito 01: Despertar 5AM` y devuelve `{date, habit, record \| null}`), `PUT /sleep/today` (upsert `{wakeTime, bedTime, sleepMinutes}`) — solo se escribe hoy; los dos responden además `history` (`internal/sleep/history.go`, higiene del sueño): los últimos 5 registros hasta hoy (`ListRecent`; hoy solo si ya se registró, los días sin registro se saltan), cada uno con `sleepStatus` (`ok` ≥ 7 h, `warning` 6–7 h, `short` < 6 h) y `wakeOnSchedule` (± 30 min de `typicalWakeTime`, la mediana de los despertares mostrados; con días pares, la menor central), más `averageSleepMinutes` y `onTrack` (todos a su hora y ninguna noche < 6 h). El front lo pinta en `SleepHistory`, arriba del formulario del modal. El registro del día D = despertar de D, acostarse la noche de D y duración de la noche D−1→D, todo declarado, nada calculado; "hoy" en la zona del hábito (Bogotá). El hábito lo busca o crea `KaizenHabitRepository.EnsureByName` (nombre exacto, el más antiguo si hay varios, `pg_advisory_xact_lock` por usuario: sin índice único en `kaizen_habits`). Es independiente de `habit1` |
| `health` | — | Públicos: `/health`, `/health/sql`, `/ready`, `/swagger`, `/docs/swagger.yaml` |

Reglas de negocio clave:

- **Dashboard** (`internal/dashboard/service.go`):
  `availableToday = ingresos recibidos (fecha ≤ hoy) − pagos realizados`;
  `availableAfterCommitments = availableToday − pagos pendientes`;
  `planStatus = on_track` si `availableAfterCommitments > 0`, si no `at_risk`. Fechas normalizadas a UTC.
- **Mercado (lista de mercado)**: una lista **es** un pago pendiente con categoría de código `mercado` (monto =
  presupuesto, vencimiento = día de mercado); `POST/PUT/DELETE /market` crean, editan y borran ese pago, así el
  dashboard la cuenta como compromiso. Un producto nace pendiente (cantidad 1–999, estimado por unidad ≥ 0, 0 = "no sé")
  y pasa a comprado con su precio real por unidad (> 0, ≤ 100.000.000). `POST items` con `price` entra ya comprado:
  es el body de antes de la lista (`{name, price}`) y sigue funcionando. Al volver a pendiente se conserva `price`.
  `spent` = Σ qty × real de comprados; `estimatedTotal` = spent + Σ qty × estimado de pendientes. Semáforo calculado
  en backend, dos veces: `status` sobre `remaining` y `projectedStatus` sobre `projectedRemaining`; `ok` / `warning`
  (restante ≤ 20 %) / `exceeded` (restante ≤ 0; no se bloquea). Toda escritura responde el `Summary` recalculado;
  lista pagada (`closed`) = solo lectura (409). Las filas anteriores a la lista se migraron como compradas
  (`MarketRepository.addPurchasedColumn`).
- **Categorías**: se comparan por `code` (`mercado`, `otros`), nunca por id o nombre.
- **Facturas**: se capturan por foto, nunca por formulario. **v1 no guarda la foto** (decisión KISS del 2026-10-09:
  solo se usa para que Claude la lea; se persisten los datos extraídos. Si se quiere después, lo simple es bytea en
  Postgres en tabla aparte, no Cloud Storage). `invoice.Service` depende de tres puertos: `Repository`, `FeatureGate`
  (lo implementa `UserRepository.HasIAFeature`) y `Extractor` (Claude vía API directa, `internal/infrastructure/llm`,
  structured outputs con JSON schema, `effort: low`, modelo en `INVOICE_MODEL`, default `claude-haiku-5-5`).
  En local, `cmd/fakeclaude` + `ANTHROPIC_BASE_URL=http://localhost:9090` prueba todo sin gastar tokens.
  **Acceso**: toda operación de `/invoices` exige `users.ia_feature = true` (boolean, default false, se activa a mano
  en PostgreSQL; `Upsert` del login no lo toca); si no, 403 y el front muestra `FeatureUnavailableDialog`. Es una
  validación mínima a propósito: la capa de autorización vendrá después.
  Orden de la captura: acceso → validar imagen (≤ 10 MB, JPEG/PNG/WebP detectado por bytes)
  → límite de 30 capturas/día Bogotá → modelo → CUFE duplicado (409) → fila. Solo el total es obligatorio; lo
  ilegible queda vacío/0 y `issued_on` en null; una lectura mala se corrige borrando y recapturando. Sin key la API
  arranca y la captura responde 503. Es un proyecto personal para aprender: Haiku por costo, el modelo se compara con datos.

### Migraciones

`cmd/migrate` (binario aparte, misma imagen). `internal/infrastructure/sql/migrate.go`:
`AutoMigrate` para users/habit1/incomes y `Migrate()` propio por repo cuando hay FK, índices
parciales o checks. **El orden importa**: categorías → pending_payments → kaizen_habits → sleep → market → invoices (sin FK) → winter_arc.
Una tabla nueva debe registrarse aquí.

### Comandos

```bash
cd kaizensnsilvam-backend
gcloud auth application-default login   # para verificar tokens en local
cp .env.example .env                    # DATABASE_URL obligatoria; GOOGLE_CLOUD_PROJECT=kaizensnsilvam
go run ./cmd/migrate
go run ./cmd/api                        # http://localhost:8080 → /swagger
go test ./...
```

Env: `DATABASE_URL`, `GOOGLE_CLOUD_PROJECT`, `PORT` (8080), `ENV`, `CORS_ALLOWED_ORIGINS`
(default `http://localhost:5173,http://localhost:3000`), `DB_MAX_OPEN_CONNS`, `DB_MAX_IDLE_CONNS`,
`DB_CONN_MAX_LIFETIME_MINUTES`, `DB_CONN_MAX_IDLE_MINUTES`, `ANTHROPIC_API_KEY` (en Cloud Run desde Secret
Manager, secreto `anthropic-api-key`), `INVOICE_MODEL`, `ANTHROPIC_BASE_URL` (solo local, apunta a `cmd/fakeclaude`), `WINTER_ARC_GRACE_UNTIL` (tregua del Winter Arc; fecha inválida = la API no arranca). Nunca loguear `DATABASE_URL` ni la API key.

## Frontend (`front-kaizensnsilvam.com/`)

### Estructura

```
src/main.tsx      <AuthProvider><ProtectedRoute><App/></ProtectedRoute></AuthProvider>
src/App.tsx       "router" casero por window.location.pathname (no hay react-router)
src/auth/         firebase.ts (ÚNICO archivo que importa el SDK), AuthContext, AuthProvider, useAuth, ProtectedRoute
src/pages/        Home, Login, SelectApp, RegisterIncome, RegisterPendingPayment, Market, Invoices, RegisterHabit1, Habits, WinterArc, WinterArcSetup, Sebas
src/hooks/        use<Recurso>.ts: estado {data, loading, error, reload} con useState/useEffect
src/services/     api.ts (request) + un archivo por recurso; mapean contrato backend → modelo UI
src/types/        contratos TS por recurso
src/components/   componentes de negocio; ui/ = primitivas shadcn
src/lib/          utilidades (habitDates, bushido, samuraiForge, utils/cn)
```

Rutas: `/` no tiene página: es el índice y `main.tsx` cambia la URL a `/caminos` antes de montar (con `replaceState`,
conserva query y hash); una ruta desconocida también muestra el selector. `/finanzas` Home (dashboard, `PATH_HOME.finanzas`;
los enlaces al inicio de finanzas usan esa constante, nunca `/`), `/ingresos`, `/gastos`, `/mercado` (lista de
mercado, mobile-first: `/mercado` elige o crea la lista y `?id=` es la lista con `MarketBudgetBar` fija bajo el header;
las hojas inferiores comparten `MarketSheet`), `/facturas` (captura por cámara; la foto se comprime
en el navegador en `src/lib/imageCompression.ts` antes de subirla; no se guarda, el detalle muestra solo los datos;
`request()` no fuerza `Content-Type` JSON cuando el body es `FormData`), `/habito-1` (tema samurái),
`/habits` (Kaizen Habits, sin `AppLayout`, piel `.habits-theme`: la vista "hoy" (`HabitsToday`: anillo del día y
check de un toque por hábito, con la semana de `useHabitsToday` = calendario de 7 días + stats por hábito) y
`?habito=<id>` el detalle (`HabitDetail`, con `pushState` para que "atrás" vuelva a hoy); crear/editar es el asistente
por pasos `HabitWizard` (las 4 leyes de Hábitos atómicos, plantillas en `src/lib/habitTemplates.ts`, solo el nombre es
obligatorio)), `/winter-arc`, `/winter-arc/setup` y `/winter-arc/nuevo-habito` (Winter Arc,
con su propio `WinterArcLayout`; `nuevo-habito` crea en `kaizen_habits` con un formulario corto, siempre activo, sin salir del camino;
con < 3 hábitos activos el setup ofrece "Sugerir hábitos": 3 sugerencias (ejercicio, comida sana, meditar) quemadas en
`src/lib/winterArcSuggestions.ts` (v1; luego las dará la IA con la misma forma), se crean con `POST /kaizen-habits` y llegan marcadas al código; `/winter-arc` abre el reto de `GET /api/winter-arc/current`, `?id=` sigue funcionando; cerrado con
cuenta atrás hasta `WINTER_ARC_OPENS_ON` en `src/lib/winterArcOpening.ts`), `/sebas`, `/caminos` (selector de caminos).
Identidad del Winter Arc: "la forja del samurái" — se apaga el cuerpo como máquina de placer y se enciende como
samurái enfocado en cumplir los hábitos (`WinterArcManifesto`). Sello `侍`, piel propia vía `.winter-arc-theme`
(redefine `--primary` & co. en `WinterArcLayout`), días cumplidos = "días forjados" que suben etapas de forja
(`src/lib/samuraiForge.ts`, `WinterArcForge`; el reto completado es siempre "Samurái").
`SelectApp` ofrece tres caminos: Finanzas, Hábitos y Winter Arc
(`src/lib/paths.ts`: `pathOf`, `PATH_HOME`; un camino nuevo obliga a completar los `Record<PathId, …>` de `PathSwitcher`,
`Loading` y `SelectApp`). Se muestra tras
cada login y en `/caminos`. Para cambiar de camino, todos los headers usan el mismo `PathSwitcher` (botón "Caminos");
el splash de carga (`AppSplash`/`PathSplash`) toma la piel del camino de la URL.
**Sueño no es un camino** (desde 2026-10-14; `/sleep` ya no existe y cae en el selector): es el botón 💤 (`SleepButton`)
de los headers de `/habits` (`HabitsHeader`) y Winter Arc (`WinterArcLayout`), que abre el formulario del día en un
modal (`SleepDialog`, hoja inferior en móvil; piel índigo `.sleep-theme`). Pide `GET /sleep/today` solo al abrirse,
porque ese GET crea el hábito `Habito 01: Despertar 5AM`.

Flujo: `ProtectedRoute` → loading → `Login` → `SelectApp` → contenido.
`AuthProvider` llama `POST /users/sync` al hacer login **y** al restaurar sesión (no bloquea si falla).

Convenciones:

- Toda salida de red pasa por `services/api.ts` → `request<T>(path, init, includeApiKey)`.
  Añade `Authorization: Bearer`, lanza `ApiError(message, status)`, maneja 204.
  Las llamadas actuales pasan `includeApiKey = false` (ver deuda técnica).
- Los componentes nunca importan Firebase: usan `useAuth()`.
- Sin Redux ni librerías de estado; sin LocalStorage para la sesión.
- Alias `@` → `src/`. Textos de UI en español.

### Comandos

```bash
cd front-kaizensnsilvam.com
npm install
cp .env.example .env   # VITE_API_URL, VITE_FIREBASE_*, VITE_SEBAS_KEY, VITE_USE_MOCK
npm run dev            # http://localhost:5173
npm run typecheck      # tsc --noEmit (lo corre el CI)
npm run build
```

`VITE_*` se incrustan en el bundle en build time: nada de ahí es secreto.

## CI/CD

Ambos repos: **`main` = integración (no despliega); push a `production` = deploy.**
GitHub Actions → OIDC (Workload Identity, providers y service accounts separados por repo) →
Artifact Registry → Cloud Run.

- Backend (`.github/workflows/backend-deploy.yml`): build → push → **Cloud Run Job de migraciones (`--wait`)** → deploy del servicio.
- Frontend (`.github/workflows/front-deploy.yml`): `npm ci` → typecheck → build Docker con `VITE_*` como build args → push → deploy.

## Reglas de trabajo

- KISS. Sin capas nuevas, sin reflexión, sin frameworks pesados, sin Pub/Sub ni asincronía salvo pedido explícito.
- Un caso de uso por iteración; nombres de negocio (`RegisterIncome`, `MarkAsPaid`), sin sufijos tipo `Entity`.
- Postgres solo desde repositorios. Nunca desde handlers ni services.
- Si cambia el contrato REST: actualizar `kaizensnsilvam-backend/docs/swagger.yaml` **y** los `types/` + `services/` del frontend
  (y de `kaizen-mobile/` si el recurso ya está en la app).
- Tests del backend en el service (`service_test.go`) con dobles simples; evitar mocks innecesarios.
- Explicar el diseño y esperar confirmación si el cambio afecta la arquitectura.
- No refactors masivos ni estéticos no pedidos; no dependencias nuevas sin necesidad.
- Mantener la compatibilidad hacia atrás de la API.

## Deuda técnica conocida (a 2026-09-24)

- **URL de producción hardcodeada**: `front-kaizensnsilvam.com/src/services/pendingPayments.ts` usa la URL de Cloud Run
  en vez de `VITE_API_URL` → en local los gastos pegan contra producción.
- **`X-API-Key` no funcional**: el backend no lo valida y el CORS no lo permite
  (`internal/http/middleware/cors.go`, `Allow-Headers` sin `X-API-Key`). Enviarlo rompe el preflight.
- **Winter Arc** usa el prefijo `/api/...`, a diferencia del resto de rutas.
- **Tres modelos de hábitos**: `habit1` (`/habito-1`, sueño con rituales y repeticiones; crea solo el día siguiente),
  `kaizen-habits` (genérico) y `sleep` (modal 💤, registro diario ligado al hábito kaizen `Habito 01: Despertar 5AM`).
  `habit1` y `sleep` se solapan; no se migraron datos entre ellos.
- **Dashboard en UTC**: el usuario está en Colombia (UTC−5); el corte de "hoy" se desfasa 5 h.
- **`DATABASE_URL` como env var en texto plano** en el deploy (debería ser `--set-secrets` / Secret Manager).
- **Docs desactualizados**: el `AGENTS.md` del frontend habla de Firestore (hoy es PostgreSQL); el README del frontend
  menciona Firebase Hosting, deploy desde `main`, `/families` y el Home con mocks. Confiar en el código, no en esos docs.
