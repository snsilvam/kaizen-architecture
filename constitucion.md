# Constitucion del Proyecto: Kaizen App - Herramienta para la mejora personal.

## 1. Naturaleza del Proyecto
Esta es una aplicacion para la gestion de finanzas de personales, tambien la gestion de habtios personales. Su proposito es permitir a los usuarios autenticarse y gestionar sus finanzas/habitos para la mejora personal continua. 

## 2. Stack Tecnologico (Reglas de Implementacion)
- **Frontend / UI:** Next.js, React, Usar Tailwind CSS para los estilos. Cloud Run.
- **Backend:** Go · Cloud Run.
- **Cloud Backend:** Cloud Functions GCP - Go.
- **Base de Datos:** Postgresql Virtual machine GCP - Firestore. 
- **Mobile:** App Android (APK sin Play Store, compilada con EAS Build); v1: login con correo, caminos y dashboard de Finanzas en solo lectura.
- **Lenguaje:** TypeScript en todo la capa de presentacion. Golang en todo el backend.

## 3. Reglas de Dominio y Logica de Negocio
Estas reglas son inmutables. El agente de IA debe respetarlas estrictamente:
- **Caminos:** El sistema SOLO maneja 3 caminos fijos: Finanzas, Habitos y winter-arc(solo disponible hasta diciembre del 2026). Cada camino es un modulode la app, o un tipo de herramienta diferente.
- **Finanzas:** Permite gestionar las finanzas personales de un usuario de manera mas eficiente y disciplinada. Kaizen mejora continua cada dia del 1%.
- **Habitos:** Permite gestionar los habitos personales de un usuario de manera mas eficiente y disciplinada. Kaizen lo importante no es el habito es repetir su identidad cada dia. 
- **Winter ARC:** Reto de 90 dias para la mejora personal, vigente solo por 2026.
- **Herramienta no web app!:** Kiazen es una herramienta no una simple app o web app, por eso su enfoque debe ser tipo app duolingo.
- **Autenticacion:** Todo flujo exige que haya un usuario con sesion activa. 

## 4. Estructura y Estilo de Codigo
- **Estructura Plana:** Evitar la sobreingenieri­a. Usar una estructura simple: `/frontend`, `/backend`, y `/mobile`.
- **Estilo:** Priorizar la programacion funcional y los componentes funcionales (Hooks en React). Evitar el uso de clases a menos que sea obligatorio. Respetar las buenas practicas de Go y el proposito del lenguaje.
- **Nomenclatura:** Usar `camelCase` para funciones/variables y `PascalCase` para Interfaces/Tipos. Clean Code.

## 5. Manejo de Errores y Validaciones
- **UI:** Nunca exponer errores crudos o *stack traces* al usuario final. Todo error tecnico debe traducirse a un mensaje amigable (ej: "La finanza pendiente ya fue pagada antes.").
- **Backend:** Retornar siempre codigos de estado HTTP semanticos (400 peticion invalida, 401 no autenticado, 409 conflicto de reserva).

## 6. Comportamiento del Agente de IA (Reglas SDD)
- **SOLID:** Respeta las buenas practicas de software y los principios SOLID.
- **KISS:** Respeta las buenas practicas de software y los principios KISS. Simple.
- **Cero Codigo Sombra (Shadow Code):** Construye estrictamente lo documentado en `spec.md`. No agregues caracteri­sticas "por si acaso" (no pasarelas de pago, no perfiles complejos, etc.).
- **Fuente de la Verdad:** Si una instruccion del usuario contradice esta constitucion o si detectas una falla logica, detente. Advierte del problema y solicita actualizar el `spec.md` antes de tocar el codigo fuente.
