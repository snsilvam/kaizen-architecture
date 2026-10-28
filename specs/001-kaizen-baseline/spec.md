# Feature Specification: Línea base del producto Kaizen

**Feature Branch**: `001-kaizen-baseline`

**Created**: 2026-10-27

**Status**: Draft

**Input**: User description: "necesito crear el archivo spec.md del proyecto para que respete el SDD que estoy agregando en esta app"

> **Qué es este documento.** Kaizen ya existe y está en producción. Este spec registra **cómo
> se comporta hoy** el producto para que sea la fuente de la verdad del SDD (Principio IV de
> la constitución, "cero código sombra"). Las únicas reglas nuevas son las decididas en
> Clarifications sobre el Winter Arc (FR-017, FR-019a y FR-025a). Toda
> funcionalidad futura se especifica en su propio `specs/NNN-.../spec.md` y, si cambia algo de
> lo descrito aquí, debe decirlo explícitamente.

## Clarifications

### Session 2026-10-27

- Q: ¿Cualquier persona puede crear su cuenta en Kaizen, o solo deben entrar cuentas creadas a mano? → A: Registro abierto: cualquiera crea su cuenta con correo y contraseña o con Google, y puede recuperar su contraseña; cada uno ve solo sus datos.
- Q: ¿Qué pasa con el camino Winter Arc cuando termina la temporada, después del 2026-12-31? → A: Queda en solo lectura: sigue en el selector, se ven el reto y su analítica, pero no se pueden crear retos nuevos.
- Q: Si un reto está en curso cuando termina la temporada, ¿qué pasa con sus días después del 2026-12-31? → A: Todo reto se corta el 2026-12-31: el que no llegó al día 90 se cierra sin completar.
- Q: ¿La captura de facturas por foto queda en la línea base o pasa a un spec propio? → A: Sale de la línea base y la rama en desarrollo se descarta; si vuelve, se hará desde cero con su propio spec.
- Q: ¿Qué se hace con `/sebas` y `/habito-1`, que quedan por fuera de los tres caminos? → A: Se toleran como están, sin funcionalidades nuevas; retirarlos sería un spec aparte.
- Q: Si el reto ya no dura siempre 90 días, ¿cómo se define cuántos días dura cada reto? → A: La persona elige cuántos días dura al crearlo; lo completa si llega al último día sin perderlo. Lo importante es cumplir el compromiso consigo misma, no la duración.
- Q: ¿Cuántos días puede durar como mínimo y como máximo un reto, y se puede crear uno cuyo último día caiga después del 2026-12-31? → A: De 3 a 90 días, y el último día no puede caer después del 2026-12-31; lo demás se rechaza al crear el reto.
- Q: Si alguien completa un reto corto (por ejemplo de 3 días), ¿también llega a "Samurái", y cómo avanzan las etapas de forja? → A: Completar el reto siempre da "Samurái", dure lo que dure; las etapas intermedias siguen fijas por días forjados (1, 7, 21, 45, 66).
- Q: Con un reto ya en curso, ¿la persona puede cambiar cuántos días dura? → A: No; la duración se fija al crear el reto. Para ir más lejos, se completa y se crea un reto nuevo.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Entrar con sesión y elegir un camino (Priority: P1)

Cualquier persona puede crear su cuenta (correo y contraseña, o Google) e iniciar sesión; si
olvida la contraseña, puede pedir un correo para restablecerla. Después del login, y cada vez que quiera
cambiar, ve un selector con exactamente tres caminos: **Finanzas**, **Hábitos** y
**Winter Arc**. Al elegir uno entra a su herramienta, con su propia piel visual. Desde
cualquier camino puede volver al selector con un botón "Caminos".

**Why this priority**: sin sesión no hay acceso a ningún dato; el selector es la puerta de
los tres caminos.

**Independent Test**: iniciar sesión, ver los tres caminos, entrar a uno, volver al selector
y cerrar sesión.

**Acceptance Scenarios**:

1. **Given** una persona sin sesión, **When** abre cualquier dirección de Kaizen, **Then** ve
   la pantalla de login y ningún dato de negocio.
2. **Given** una persona que acaba de iniciar sesión, **When** termina el login, **Then** ve el
   selector con los tres caminos.
3. **Given** una persona con sesión que vuelve a abrir la herramienta, **When** la sesión sigue
   vigente, **Then** entra sin volver a escribir su contraseña.
4. **Given** una persona dentro de un camino, **When** pulsa "Caminos", **Then** vuelve al
   selector.
5. **Given** una dirección que no existe, **When** la abre, **Then** ve el selector de caminos.
6. **Given** una persona sin cuenta, **When** se registra con correo y contraseña o con Google,
   **Then** entra al selector de caminos sin datos de nadie más.
7. **Given** una persona que olvidó su contraseña, **When** pide restablecerla con su correo,
   **Then** recibe un correo para elegir una nueva.

---

### User Story 2 - Saber cuánto puedo gastar hoy (Finanzas) (Priority: P1)

La persona registra sus ingresos (nombre, monto y fecha) y sus gastos pendientes (nombre,
monto, categoría y fecha de vencimiento). El inicio de Finanzas responde las tres preguntas del
producto: cuánto dinero puede gastar hoy, cuántos días faltan para su próximo ingreso y si va
bien o mal respecto a su plan. Puede marcar un gasto como pagado o borrarlo.

**Why this priority**: es la razón original de Kaizen y responde las tres preguntas de la
constitución.

**Independent Test**: registrar un ingreso con fecha pasada, uno futuro y un gasto pendiente;
comprobar los números del inicio; marcar el gasto como pagado y ver cómo cambian.

**Acceptance Scenarios**:

1. **Given** ingresos recibidos por 1.000.000 y pagos realizados por 300.000, **When** abre el
   inicio de Finanzas, **Then** "disponible hoy" muestra 700.000.
2. **Given** además 200.000 en gastos pendientes, **When** abre el inicio, **Then** "disponible
   después de compromisos" muestra 500.000 y el plan aparece como "vas bien".
3. **Given** gastos pendientes que igualan o superan lo disponible hoy, **When** abre el
   inicio, **Then** el plan aparece "en riesgo".
4. **Given** un ingreso con fecha dentro de 5 días, **When** abre el inicio, **Then** ve ese
   ingreso como el próximo, con "faltan 5 días".
5. **Given** un gasto pendiente, **When** lo marca como pagado, **Then** "disponible hoy" baja
   en su monto y "disponible después de compromisos" no cambia.
6. **Given** un gasto que ya fue pagado, **When** intenta pagarlo otra vez, **Then** ve un
   mensaje amigable ("La finanza pendiente ya fue pagada antes.") y nada cambia.

---

### User Story 3 - Repetir mi identidad cada día (Hábitos) (Priority: P2)

La persona crea hábitos con un asistente por pasos inspirado en las cuatro leyes de *Hábitos
atómicos* (identidad, señal, atractivo, acción, versión mínima de 2 minutos, recompensa, hora
y lugar); solo el nombre es obligatorio y hay plantillas. La vista "hoy" muestra un anillo de
progreso del día y un check de un toque por hábito, más la semana. Cada hábito tiene un
detalle con calendario, estadísticas y metas de repeticiones.

**Why this priority**: es el segundo camino y el que más se usa a diario.

**Independent Test**: crear un hábito solo con nombre, marcarlo hecho hoy, ver el anillo
completo y el día marcado en su calendario; desmarcarlo y ver que se revierte.

**Acceptance Scenarios**:

1. **Given** un hábito activo sin repetición hoy, **When** pulsa su check, **Then** queda
   registrada la repetición de hoy y el anillo del día avanza.
2. **Given** un hábito ya marcado hoy, **When** intenta registrar otra repetición del mismo
   día, **Then** no se duplica (máximo una repetición por hábito y día).
3. **Given** una repetición, **When** la registra como "versión mínima", **Then** cuenta como
   repetición y queda señalada como mínima.
4. **Given** un hábito, **When** fija una meta de repeticiones, **Then** la meta debe ser de al
   menos 5 y el detalle muestra su avance hasta completarla.
5. **Given** un hábito, **When** lo pausa (inactivo), **Then** deja de aparecer en la vista
   "hoy" pero conserva su historial.

---

### User Story 4 - Cumplir el reto que me puse (Winter Arc) (Priority: P2)

La persona crea un reto eligiendo cuántos días durará, al menos 3 de sus hábitos activos (su
"código") y escribiendo su compromiso con su yo de enero. Lo que importa es repetir y cumplir
lo que se comprometió consigo misma, no que el reto sea largo. Cada día se forja si cumple
**todas** las reglas del código. Ve una cuadrícula con los días de su reto, su horario del día,
las etapas de forja del samurái,
sus deudas y una cuenta atrás de la temporada que termina el 2026-12-31 a medianoche de
Bogotá.

**Why this priority**: es el tercer camino y tiene fecha de cierre; su valor se pierde si no
funciona durante la temporada 2026.

**Independent Test**: con 3 hábitos activos, crear un reto de 3 días, marcar hoy los 3
hábitos, sincronizar el día y verlo forjado en una cuadrícula de 3 días.

**Acceptance Scenarios**:

1. **Given** menos de 3 hábitos activos, **When** intenta crear el reto, **Then** se le impide y
   se le ofrecen 3 hábitos sugeridos (ejercicio, comida sana, meditar) que puede crear de un
   toque.
2. **Given** un reto vigente, **When** intenta crear otro, **Then** se rechaza: solo hay un reto
   vigente a la vez.
3. **Given** todas las reglas cumplidas hoy, **When** sincroniza el día, **Then** hoy queda como
   forjado.
4. **Given** un día pasado, **When** intenta modificarlo, **Then** se rechaza: solo se escribe
   hoy y los días cerrados son inmutables.
5. **Given** 3 días fallados seguidos (fuera de la tregua), **When** se evalúa el reto, **Then**
   el reto queda perdido.
6. **Given** un domingo, **When** se evalúa, **Then** es día libre: no suma ni rompe la racha y
   no deja deuda.
7. **Given** un día fallado, **When** en sus 3 días siguientes (sin contar domingo) registra con
   la fecha del fallo una repetición de cada regla que faltó, **Then** la deuda queda pagada y
   ese día deja de contar como fallo para la racha.
8. **Given** un día fallado antes de la fecha de tregua configurada, **When** se evalúa,
   **Then** no puede hacer perder el reto.
9. **Given** un reto activo, **When** edita su código (≥ 3 hábitos), **Then** el cambio vale
   desde hoy y los días pasados se juzgan con las reglas que tenían.
10. **Given** el último día del reto cumplido sin perderlo, **When** se evalúa, **Then** el reto
    queda completado y la persona es "Samurái" sin importar la duración (un reto de 3 días
    completado pasa directo a "Samurái"); puede ver su analítica (hábito más cumplido, éxito
    por día de la semana y hábito ancla).
11. **Given** la temporada terminada (después del 2026-12-31 en Bogotá), **When** entra a
    Winter Arc, **Then** ve su reto y su analítica en solo lectura y no puede crear un reto
    nuevo.
12. **Given** un reto activo que el 2026-12-31 no ha llegado a su último día, **When** termina
    ese día en Bogotá, **Then** el reto se cierra como "sin completar": no es "Samurái" ni
    "perdido", y conserva sus días forjados y su etapa de forja.
13. **Given** una persona que crea un reto, **When** elige su duración en días, **Then** el reto
    dura exactamente esos días y la cuadrícula muestra solo esos días.
14. **Given** una persona que crea un reto, **When** elige menos de 3 días, más de 90 o una
    duración cuyo último día cae después del 2026-12-31, **Then** se rechaza con un mensaje
    amigable que le dice cuántos días puede elegir.
15. **Given** un reto activo, **When** quiere alargarlo o acortarlo, **Then** no es posible: la
    duración quedó fija al crearlo; al completarlo puede crear un reto nuevo más largo.

---

### User Story 5 - Registrar mi sueño del día (Priority: P3)

Desde un botón 💤 en Hábitos y en Winter Arc, la persona abre un formulario con la hora a la que
despertó hoy, la hora a la que se acostará y cuántas horas durmió anoche. Ve su historial de
las últimas 5 noches con un semáforo de horas de sueño y si despertó a su hora.

**Why this priority**: apoya el hábito "Despertar 5AM" pero no es un camino propio.

**Independent Test**: abrir 💤, guardar el registro de hoy, cerrarlo y volver a abrirlo con los
datos guardados y el historial actualizado.

**Acceptance Scenarios**:

1. **Given** el formulario de hoy, **When** guarda despertar, acostarse y duración, **Then**
   queda un único registro para hoy (guardar otra vez lo reemplaza).
2. **Given** noches de 7 h o más, de 6 a 7 h y de menos de 6 h, **When** ve el historial,
   **Then** se marcan como bien, alerta y corta.
3. **Given** un día pasado, **When** intenta editarlo, **Then** no es posible: solo se escribe
   hoy.

---

### User Story 6 - Hacer mercado con presupuesto (Priority: P3)

La persona crea una lista de mercado con presupuesto y día de mercado; esa lista cuenta como
un gasto pendiente en Finanzas. Agrega productos con cantidad y precio estimado, y al
comprarlos registra el precio real. Una barra fija muestra lo gastado, lo proyectado y un
semáforo.

**Why this priority**: refina la pregunta "¿cuánto puedo gastar hoy?" para el gasto más
frecuente.

**Independent Test**: crear una lista de 200.000, agregar 3 productos, comprar uno y ver el
semáforo y el total proyectado.

**Acceptance Scenarios**:

1. **Given** una lista nueva, **When** se crea, **Then** aparece como gasto pendiente de
   categoría mercado en Finanzas, por el monto del presupuesto.
2. **Given** productos pendientes y comprados, **When** ve la lista, **Then** "gastado" suma
   cantidad × precio real de los comprados y "proyectado" le suma cantidad × estimado de los
   pendientes.
3. **Given** que lo restante es ≤ 20 % del presupuesto, **When** ve la barra, **Then** el
   semáforo está en alerta; con restante ≤ 0, en excedido (sin bloquear compras).
4. **Given** una lista ya pagada, **When** intenta modificarla, **Then** se rechaza: es de solo
   lectura.

---

### User Story 7 - Consultar Finanzas desde el celular (Priority: P3)

La persona instala la app Android (fuera de la tienda), inicia sesión con correo y ve el
selector de caminos; solo Finanzas abre, y muestra el inicio de Finanzas en solo lectura.

**Why this priority**: acceso rápido a la pregunta principal; no agrega capacidades nuevas.

**Independent Test**: instalar la app, iniciar sesión y comprobar que los números coinciden
con la web.

**Acceptance Scenarios**:

1. **Given** sesión iniciada en la app, **When** abre Finanzas, **Then** ve los mismos valores
   que en la web.
2. **Given** la app, **When** busca registrar o editar datos, **Then** no hay forma de hacerlo
   (v1 es solo lectura).

---

### Edge Cases

- Sesión expirada en medio de una acción: se vuelve al login sin mostrar errores técnicos.
- Una persona intenta ver, editar o borrar un dato de otra (por ejemplo, adivinando su
  identificador): se responde como si no existiera y nada cambia.
- Montos negativos, cero donde no aplica o con decimales: se rechazan con un mensaje claro.
- Sin ingresos registrados: "disponible hoy" es 0 y no hay próximo ingreso que mostrar.
- Cambio de día: "hoy" para hábitos, sueño y Winter Arc es el día de Bogotá; Finanzas usa hoy
  el día en UTC (deuda conocida, ver Assumptions).
- Winter Arc abierto después de varios días sin entrar: los días pendientes se cierran en
  orden al abrirlo, como si se hubieran juzgado a su hora.
- Borrar un hábito que es regla de un Winter Arc vigente: no se permite.
- Reto creado antes de la duración elegible (todos de 90 días) y empezado después del
  2026-10-02: sus días no caben en la temporada, así que como máximo termina "sin completar";
  con la tregua hasta el 2026-12-31 tampoco puede perderse. Un reto nuevo no puede quedar así,
  porque su último día debe caber en la temporada (FR-017).
- Quedan menos de 3 días de temporada: ya no cabe ningún reto nuevo y crearlo se rechaza.
- Fallo de red o del servidor: la persona ve un mensaje amigable y puede reintentar.

## Requirements *(mandatory)*

### Functional Requirements

**Acceso y caminos**

- **FR-001**: El sistema MUST exigir sesión iniciada (correo y contraseña, o Google) para
  cualquier dato o acción de negocio. El registro es abierto: cualquier persona MUST poder
  crear su cuenta en la web y restablecer su contraseña por correo.
- **FR-002**: El sistema MUST mostrar, tras el login y bajo demanda, un selector con
  exactamente tres caminos: Finanzas, Hábitos y Winter Arc.
- **FR-003**: Cada persona MUST ver y modificar solo sus propios datos.
- **FR-004**: Los errores MUST mostrarse como mensajes amigables en español, nunca como texto
  técnico.

**Finanzas**

- **FR-005**: Las personas MUST poder registrar, listar y borrar ingresos (nombre, monto
  entero en pesos colombianos, fecha).
- **FR-006**: Las personas MUST poder registrar, listar, marcar como pagados y borrar gastos
  pendientes (nombre, monto, categoría, vencimiento).
- **FR-007**: El sistema MUST calcular "disponible hoy" = ingresos con fecha ≤ hoy − pagos
  realizados.
- **FR-008**: El sistema MUST calcular "disponible después de compromisos" = disponible hoy −
  gastos pendientes, y mostrar el plan "vas bien" si es mayor que 0, si no "en riesgo".
- **FR-009**: El sistema MUST mostrar el próximo ingreso futuro y los días que faltan.
- **FR-010**: El sistema MUST rechazar pagar dos veces el mismo gasto.
- **FR-011**: Una lista de mercado MUST ser un gasto pendiente de categoría mercado con su
  presupuesto como monto, y MUST calcular gastado, proyectado y dos semáforos (actual y
  proyectado) con los umbrales de la Historia 6.
- **FR-012**: Una lista de mercado pagada MUST ser de solo lectura.

**Hábitos**

- **FR-013**: Las personas MUST poder crear, editar, pausar y borrar hábitos; solo el nombre es
  obligatorio.
- **FR-014**: El sistema MUST permitir una sola repetición por hábito y día, opcionalmente
  marcada como versión mínima y con una nota breve.
- **FR-015**: El sistema MUST mostrar la vista "hoy" con el progreso del día y la semana, y un
  detalle por hábito con calendario, estadísticas y metas (meta ≥ 5 repeticiones).
- **FR-016**: Las personas MUST poder registrar el sueño de hoy (despertar, acostarse,
  duración entre 1 minuto y 24 horas) y ver las últimas 5 noches con su semáforo.

**Winter Arc**

- **FR-017**: El sistema MUST permitir un único reto vigente por persona, con la duración en
  días que la persona elige al crearlo, al menos 3 hábitos activos propios como reglas y un
  compromiso de hasta 500 caracteres. La duración MUST estar entre 3 y 90 días y su último
  día MUST NOT caer después del 2026-12-31 (Bogotá); si no, crear el reto MUST rechazarse.
  Una vez creado, la duración MUST NOT cambiar; para ir más lejos se crea un reto nuevo
  cuando el actual termina.
- **FR-018**: Un día MUST quedar forjado solo si todas sus reglas tienen repetición registrada
  antes de la medianoche de Bogotá de ese día.
- **FR-019**: El sistema MUST cerrar los días pasados en orden al consultar el reto, marcar el
  reto como perdido con 3 fallos consecutivos y como completado al cumplir su último día.
- **FR-019a**: Al terminar el 2026-12-31 (Bogotá), todo reto activo MUST cerrarse: si llegó
  a su último día queda completado; si no, queda "sin completar". Ningún día posterior se juzga.
- **FR-019b**: Las etapas de forja MUST ganarse por días forjados (1, 7, 21, 45 y 66), sin
  importar la duración del reto, y un reto completado MUST mostrar siempre la etapa "Samurái".
- **FR-020**: Los domingos MUST ser días libres que no cuentan para la racha, la deuda ni la
  analítica.
- **FR-021**: Un día fallado MUST poder pagarse en sus 3 días siguientes (sin contar domingo)
  registrando, con la fecha del fallo, cada regla que faltó; pagado, deja de contar como fallo
  para la racha.
- **FR-022**: Antes de la fecha de tregua configurada, un fallo MUST NOT hacer perder el reto.
- **FR-023**: Las personas MUST poder cambiar las reglas del reto activo (≥ 3); el cambio vale
  desde hoy. Editar el reto no incluye su duración (FR-017).
- **FR-024**: El sistema MUST mostrar la analítica solo de retos cerrados (completados,
  perdidos o sin completar).
- **FR-025**: El camino Winter Arc MUST mostrar una cuenta atrás de temporada hasta el
  2026-12-31 a medianoche de Bogotá.
- **FR-025a**: Terminada la temporada, el camino Winter Arc MUST seguir en el selector en
  solo lectura: se ven el reto y su analítica, y crear un reto nuevo MUST rechazarse.

**Móvil**

- **FR-026**: La app Android MUST ofrecer login con correo, el selector de caminos y el inicio
  de Finanzas en solo lectura.

### Key Entities *(include if feature involves data)*

- **Persona usuaria**: quien inicia sesión; dueña de todos sus datos.
- **Ingreso**: nombre, monto entero en COP, fecha.
- **Gasto pendiente**: nombre, monto, categoría (identificada por código: mercado, otros…),
  vencimiento, estado pagado/pendiente.
- **Lista de mercado**: un gasto pendiente de categoría mercado más sus productos (nombre,
  cantidad 1–999, estimado por unidad, precio real por unidad, comprado o no).
- **Hábito**: nombre y campos de las cuatro leyes, hora, lugar, zona horaria, activo/pausado.
- **Repetición**: un hábito en un día; mínima o completa; nota breve.
- **Meta de hábito**: número de repeticiones objetivo (≥ 5), inicio y fecha de cumplimiento.
- **Registro de sueño**: uno por día; despertar, acostarse, minutos dormidos; ligado al hábito
  "Despertar 5AM".
- **Reto Winter Arc**: duración en días elegida al crearlo, estado (activo, completado,
  perdido, sin completar), compromiso,
  reglas (hábitos), días cerrados con su resultado, deuda pagada y detalle de qué reglas se cumplieron.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Una persona con sesión responde "¿cuánto puedo gastar hoy?" en menos de 5
  segundos desde que abre Finanzas.
- **SC-002**: Marcar un hábito como hecho hoy toma un solo toque desde la vista "hoy".
- **SC-003**: El 100 % de los intentos de acceder a datos ajenos o sin sesión son rechazados.
- **SC-004**: Ningún error mostrado a la persona contiene texto técnico (códigos internos,
  trazas, mensajes en inglés).
- **SC-005**: Los valores del inicio de Finanzas coinciden al peso entre la web y la app móvil.
- **SC-006**: Para cualquier reto, el resultado de cada día cerrado (forjado, fallado, libre,
  deuda pagada) es el mismo sin importar cuándo se abra el reto después.
- **SC-007**: Toda funcionalidad nueva posterior a esta línea base tiene su propio spec que
  referencia qué partes de este documento cambia.

## Assumptions

- Este spec describe el comportamiento vigente a 2026-10-27; solo introduce las reglas
  decididas en Clarifications (ver "Reglas decididas que el código aún no cumple"). Si el
  código y este documento difieren, se trata como defecto o como deuda a resolver con un spec
  nuevo, nunca en silencio.
- **Fuera de la línea base, tolerados sin nuevas funcionalidades**: la página personal oculta
  "/sebas" (no es un camino) y el hábito de sueño antiguo "/habito-1" (se solapa con el
  registro de sueño de la Historia 5; no se migran datos). Siguen funcionando como hoy; solo
  se corrigen si se rompen, y retirarlos requiere su propio spec.
- **Reglas decididas que el código aún no cumple** (necesitan su propio spec para
  implementarse): FR-025a, el bloqueo de retos nuevos tras la temporada; FR-019a, el corte
  de los retos activos el 2026-12-31 con el estado "sin completar"; y FR-017, la duración
  elegida por la persona. Hoy solo el frontend tiene la fecha de cierre: el servidor no
  rechaza crear un reto ni corta los activos, y todo reto dura 90 días. FR-017 contradice el
  Principio I de la constitución ("reto de 90 días"), que debe enmendarse antes de
  implementarlo.
- **Deudas conocidas que este spec no corrige**: Finanzas calcula "hoy" en UTC y no en hora de
  Colombia; la dirección de producción fija en el cliente de gastos.
- **Fuera de este spec**: la captura de facturas por foto. La versión en desarrollo se
  descartó (2026-10-27); si vuelve, se especifica desde cero con `/speckit-specify`.
- Solo existen montos en pesos colombianos, enteros y sin decimales.
- "Hoy" para Hábitos, sueño y Winter Arc es el día calendario de Bogotá.
- La tregua del Winter Arc es una fecha configurable por entorno; en producción, por defecto
  hasta 2026-12-31.
- Una sola persona por cuenta; no hay roles, perfiles compartidos ni pagos dentro de la
  herramienta.
