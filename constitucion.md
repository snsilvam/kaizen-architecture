## Features principales
 
### 1. Autenticación de Usuarios
- Los usuarios deben poder registrarse e iniciar sesión usando un correo electrónico y contraseña.
- Solo los usuarios con una sesión activa pueden visualizar la disponibilidad completa y realizar finanzas.
- Los usuarios solo pueden gestionar sus propias finanzas, no las de terceros.
 
### 2. Exploración y Selección de Finanzas
- El sistema debe listar estáticamente las finanzas disponibles: ingresos, gastos, mercado.
- ingresos: pasivos financieros del usuario.
- gastos: activos financieros del usuario.
- mercado: modulo para ayudar al usuario a manejar sus finanzas con el mercado.
- El sistema debe indicar claramente qué es una deuda.
- El sistema debe indicar claramente cuanto tiene el usuario disponible y cuanto tiene comprometido en pago de duda o pendientes.
 
### 3. Exploración y Selección Habits
- El usuario puede ver un bloque horario disponible con la informacion de sus habitos.
- Es un CRUD simple para administrar habitos de un usuario
- No es solo crear habitos es mantenerlos.
 
### 4. Exploración y Selección Winter-arc
- **Regla Crítica (Prevención de colisión):** Antes de confirmar, el sistema debe re-validar que el bloque siga disponible. Solo esta habilitado los ultimos 3 meses de cada año.
- El usuario debe tener un panel donde pueda ver la lista de sus retos pasados y su historial de winter arc por año.
- El modulo debe mostrar: Un contador en segundo que muetres un retroceso de 90 dias del 90 al 1, tomando como referencia la fecha actual del año, el winter arc solo sucede en los 90 dias finalez del año.
- El usuario puede repetir sus habitos tipo racha duolingo, con el winter-arc.
 
## Lo que NO se va a construir en esta iteración (Non-Goals)
 
- NO habrá pasarela de pagos integrados (el pago se manejará presencialmente en el club).
- NO habrá notificaciones externas (ni correos transaccionales de confirmación, ni mensajes SMS/WhatsApp).
- NO habrá sistema de "matchmaking" para buscar compañeros de juego o torneos.
