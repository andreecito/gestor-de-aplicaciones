---
name: gestor-arquitectura
description: "Guía exclusiva de GestorAplication - Arquitect para definir y revisar la estructura global del gestor con React/TypeScript/Vite, Fastify/Node.js/TypeScript y Supabase, y preparar asignaciones acotadas. Backend y frontend reciben el contexto y las rutas de su tarea; no cargan esta guía global."
---

# Arquitectura del gestor de aplicaciones

Guía local del proyecto para convertir requisitos en una estructura consistente. Complementa el flujo de órdenes del arquitecto definido en `.claude/agents/gestoraplication-arquitect.md`; las instrucciones del usuario y la configuración vigente del proyecto prevalecen sobre los ejemplos de esta guía.

## Responsable y distribución del contexto

Esta guía y el árbol global son herramientas del agente arquitecto. Solo ese rol los carga para definir y mantener la organización del proyecto. Backend y frontend ejecutan tareas con las rutas, responsabilidades, contratos y reglas que el arquitecto les entrega; no usan esta skill para elegir carpetas o reconstruir toda la aplicación. Si se invoca desde un ejecutor, remite la decisión de estructura al arquitecto y trabaja con el contexto de la asignación disponible.

## Punto de partida

**Decisión del usuario: usar Fastify como framework de backend sobre Node.js con TypeScript.** La estructura, los contratos y las asignaciones deben respetar esa elección; un cambio de framework se propone al usuario antes de aplicarse.

Lee [estructura-del-gestor.md](references/estructura-del-gestor.md) antes de proponer carpetas, límites de módulos o contratos nuevos. Después inspecciona el proyecto real: no supongas que su árbol coincide con el ejemplo.

El alcance conocido comprende empresas, aplicaciones, contratos, personas, cuentas y asignaciones de licencias o accesos. Distingue los operadores que inician sesión en el gestor de las personas y cuentas inventariadas. Un nombre ambiguo del proyecto no incorpora almacenamiento de contraseñas ni otro proceso de negocio sin un requerimiento concreto.

El stack acordado es `frontend/` con React + TypeScript + Vite y `backend/` con Fastify sobre Node.js + TypeScript, con Supabase PostgreSQL y Auth. Cada parte tiene su configuración, dependencias, arranque, compilación y pruebas. Las migraciones se organizan en `backend/supabase/`. Las versiones y herramientas complementarias se concretan al implementar; no vuelvas a tratar Vite o Fastify como decisiones pendientes ni añadas servicios para satisfacer una plantilla.

## Cómo definir la estructura

1. Relaciona cada resultado solicitado con las pantallas, operaciones, entidades y permisos que necesita. Explicita las dudas que cambien el diseño y separa lo confirmado de lo propuesto.
2. Identifica los módulos de negocio afectados y sus responsables. Un módulo agrupa una capacidad del gestor; una carpeta compartida contiene solo piezas que realmente tienen varios consumidores.
3. Propón un árbol con `frontend/` y `backend/` separados y una explicación breve de cada responsabilidad. El detalle interno se adapta al pedido. Si en el futuro existe código con otra organización, planifica los cambios necesarios con el arquitecto antes de mover archivos o romper importaciones.
4. Define las dependencias permitidas y los contratos entre frontend, servidor y datos. Incluye validación de entradas, salidas, errores y estados. Compartir tipos TypeScript no sustituye validar datos recibidos en ejecución.
5. Asigna una única ubicación y un responsable para cada regla, contrato y migración. La interfaz puede anticipar una validación para ayudar al usuario; las reglas de integridad se garantizan donde se ejecuta la operación.
6. Como arquitecto, incorpora la estructura a tu definición vigente y deriva tareas autosuficientes con rutas concretas de edición y consulta, responsabilidades, contratos, dependencias y aceptación. Transmite a cada ejecutor solo el fragmento y conocimiento necesarios para su trabajo; no le encargues aprender la guía global. No generes código cuando solo se haya pedido propuesta o planificación.

## Límites que deben conservar los ejecutores

- Frontend y backend se comunican mediante la API acordada. No se importan sus carpetas `src/` entre sí ni se comparten clientes privados de persistencia o credenciales.
- Un módulo del servidor expone sus operaciones necesarias; otros módulos no acceden a su implementación interna para evitar el contrato.
- Los esquemas de las rutas Fastify son la fuente ejecutable del contrato de API. `contracts/openapi.json` será su exportación pública; los tipos o clientes de frontend se generan desde ella cuando se implemente esa herramienta. No se copian modelos internos de la base de datos al navegador. El arquitecto asigna un responsable para mantener sincronizados esquemas, exportación y consumidores.
- Las migraciones en `backend/supabase/migrations/` son la fuente versionada del esquema y de sus políticas. Un cambio de tabla se coordina con los contratos y consumidores afectados. El frontend usa Supabase Auth con configuración pública; Fastify verifica identidad y autorización para las operaciones de negocio.
- Las referencias técnicas no reemplazan las decisiones del usuario. Las modificaciones de estructura, contratos o límites compartidos vuelven al arquitecto antes de que los ejecutores las apliquen.
- Al delegar, transmite el fragmento pertinente de la definición vigente y las referencias concretas de la tarea. No envíes la ruta de esta skill como una lectura obligatoria del ejecutor ni presupongas que conoce tu contexto.

## Comprobar que la propuesta y el cambio encajan

Revisa las rutas modificadas, las importaciones y el contrato que conecta cada parte. Detecta duplicación de módulos, reglas o clientes, archivos fuera de la superficie asignada y dependencias circulares introducidas por el cambio.

Cuando exista implementación, verifica el comportamiento pertinente: tipos y compilación con los comandos reales del proyecto; validación de entradas en la API; permisos por empresa; consistencia de operaciones concurrentes; y el recorrido de interfaz a persistencia para un flujo que cruce capas. Una estructura dibujada o una prueba aislada no acredita esa integración.

Comprueba además la compatibilidad de plugins Fastify, la correspondencia entre esquemas y OpenAPI, y que la compilación Vite no incorpore secretos ni código privado del servidor. Los esquemas de respuesta deben seleccionar los campos públicos de cada operación.

En cambios de licencias o importación, comprueba integridad en base de datos, repetición de operaciones y fallos parciales cuando apliquen. No aceptes una retirada local como confirmación de baja en un proveedor externo.

Entrega el funcionamiento esperado, el árbol propuesto o cambio de árbol, los contratos, los responsables y tareas, y la evidencia disponible. Ajusta la extensión al pedido. Mantén la propuesta en la conversación salvo que el usuario pida guardarla; esta skill no exige carpetas de planificación.
