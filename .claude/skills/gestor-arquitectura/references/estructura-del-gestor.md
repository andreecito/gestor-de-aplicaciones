# Organización de referencia para el gestor

Esta es una guía propia del proyecto, basada en las referencias técnicas al final. La separación en `frontend/` y `backend/` responde a la preferencia explícita del usuario. El detalle interno es una propuesta adaptable al alcance vigente; todavía no representa una aplicación implementada ni una plantilla oficial de los proveedores.

El agente arquitecto es el único responsable de consultar y mantener este esquema global. Backend y frontend reciben de él una asignación con el contexto funcional, fragmento de estructura, rutas de edición y consulta, contratos y comprobaciones de su tarea. No necesitan cargar esta guía ni decidir la organización de otros módulos para ejecutar esa asignación.

## Stack vigente y alcance funcional

| Parte | Elección acordada | Responsabilidad |
| --- | --- | --- |
| Interfaz | React + TypeScript | Pantallas, formularios, navegación y estados de interacción. |
| Desarrollo y compilación web | Vite | Servidor de desarrollo y generación del frontend distribuible. |
| Framework de backend, elegido por el usuario | Fastify sobre Node.js + TypeScript | API, operaciones del gestor, validación, permisos y coordinación. |
| Datos | Supabase PostgreSQL | Persistencia, integridad, transacciones y políticas de datos. |
| Identidad del operador | Supabase Auth | Inicio de sesión e identidad que verifica la API. |

Estas tecnologías ya están acordadas. Las versiones compatibles, el gestor de paquetes, el enrutador de React y las herramientas de pruebas se fijan al preparar la implementación. Storage e integraciones de proveedores se incorporan cuando una funcionalidad las necesite.

El producto sigue siendo un gestor de aplicaciones, contratos, licencias y accesos de varias empresas. El prototipo inicial ayuda a comprender los flujos y sus problemas; no acredita datos de producción ni cambia ese alcance por sí solo.

## Separación de frontend y backend

Mantén un repositorio con dos aplicaciones delimitadas. Cada una tiene su propio `package.json`, configuración TypeScript, comandos, dependencias y pruebas; pueden ejecutarse y compilarse por separado. El arquitecto define el gestor de paquetes y la estrategia de archivos de bloqueo cuando prepare la implementación.

```text
gestor-aplicacion/
├── frontend/
│   ├── src/
│   │   ├── app/             # Composición, rutas y proveedores de React
│   │   ├── features/        # Funcionalidades y sus pantallas
│   │   ├── components/      # Componentes compartidos
│   │   ├── lib/             # Cliente HTTP, autenticación y bibliotecas
│   │   └── main.tsx         # Entrada de la interfaz
│   ├── public/              # Archivos públicos necesarios
│   ├── tests/               # Pruebas de interfaz e integración visual
│   ├── index.html           # Entrada HTML de Vite
│   ├── vite.config.ts       # Configuración y desarrollo de frontend
│   ├── .env.example         # Nombres y valores de ejemplo sin secretos
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── src/
│   │   ├── app.ts           # Construye la aplicación y registra módulos
│   │   ├── server.ts        # Arranca y cierra el servidor
│   │   ├── config/          # Configuración validada del servidor
│   │   ├── plugins/         # Registro de capacidades transversales en Fastify
│   │   ├── modules/         # Capacidades y reglas del negocio
│   │   ├── integrations/    # Adaptación de proveedores externos
│   │   └── infrastructure/  # Clientes de datos y servicios transversales
│   ├── supabase/
│   │   ├── config.toml      # Configuración cuando se inicialice la CLI
│   │   ├── migrations/      # Esquema, funciones y políticas versionadas
│   │   └── seed.sql         # Datos sintéticos, cuando hagan falta
│   ├── tests/               # Pruebas del servidor y persistencia
│   ├── .env.example         # Configuración de ejemplo sin secretos
│   ├── package.json
│   └── tsconfig.json
├── contracts/
│   └── openapi.json         # Exportación de los esquemas de la API
├── .claude/
└── CLAUDE.md
```

Las dos carpetas principales se conservan en las propuestas y asignaciones de este proyecto. No añadas otra web o servidor paralelo para la misma responsabilidad. El árbol se materializa gradualmente con los archivos y pruebas necesarios para la entrega, sin carpetas vacías por anticipado. Los puntos de entrada y configuración mostrados corresponden a React/Vite y al servidor Fastify acordados.

`contracts/` contiene la especificación pública de comunicación. La fuente ejecutable son los esquemas de ruta del backend; `openapi.json` se exporta desde la aplicación Fastify al implementar el soporte OpenAPI, con un plugin compatible como `@fastify/swagger`. Frontend y backend conservan paquetes independientes; el cliente web se deriva de esa especificación y no importa el código fuente del servidor. Antes de implementar la exportación, el arquitecto acuerda el contrato de la tarea en la conversación; no se inventan archivos generados.

## Frontend React, TypeScript y Vite

Organiza en `frontend/src/features/` el código específico de una capacidad, por ejemplo `applications/`, `companies/` o `licenses/`. Una funcionalidad puede incluir sus componentes, peticiones, tipos o pruebas cuando sean necesarios. Las pantallas se componen desde `frontend/src/app/`; las piezas reutilizadas por varias funcionalidades van a las ubicaciones compartidas.

El cliente de la API y el de autenticación se configuran en un lugar claro. Los componentes no repiten la construcción de clientes ni incorporan credenciales privilegiadas. Separa datos remotos, estado del formulario y estado visual. Define estados de carga, vacío, error y éxito acordes con el contrato.

Las peticiones de negocio van a la API del backend. El inicio de sesión puede usar Supabase Auth desde el navegador con la configuración pública correspondiente; el backend verifica la identidad y autorización al recibir las peticiones. No introduzcas un segundo camino de escritura directa a tablas desde un componente sin una decisión del arquitecto sobre permisos y reglas.

Evita dependencias circulares entre funcionalidades. Para una pantalla que combina varias, coordina la composición en la capa de aplicación en lugar de importar internals de una funcionalidad desde otra. No añadas cachés, stores o abstracciones hasta conocer la necesidad.

`index.html` y `src/main.tsx` son las entradas del frontend; `app/` conecta la navegación y los proveedores que se elijan. `vite.config.ts` mantiene la configuración web. El servidor de desarrollo Vite y el servidor Fastify son procesos distintos.

Las variables con prefijo `VITE_` se incorporan al código del navegador. Úsalas para configuración pública como la URL de API y la configuración pública de Supabase Auth; nunca para una contraseña de base de datos ni una clave privilegiada. Las variables reales se mantienen fuera del repositorio; `.env.example` describe la configuración necesaria con ejemplos sin secretos.

## Backend Fastify, Node.js y TypeScript

Cada módulo reúne sus operaciones y reglas. Una organización posible dentro de `backend/src/modules/licenses/` es:

```text
licenses/
├── index.ts        # Plugin que registra las rutas públicas del módulo
├── routes.ts       # Adaptación HTTP y registro de operaciones
├── schemas.ts      # Validación y contratos de la operación
├── service.ts      # Casos de uso y coordinación
└── repository.ts   # Acceso a persistencia cuando se necesita ese límite
```

Cada módulo se registra como un plugin Fastify con el prefijo y las dependencias que defina el arquitecto. `schemas.ts` declara las entradas y respuestas de sus rutas; `routes.ts` conecta HTTP con los casos de uso. No dupliques capas con el mismo cometido. Las pruebas siguen la convención real del proyecto. La ruta no debe concentrar validación, SQL, reglas y llamadas de proveedores en un único manejador.

Separa la construcción de la aplicación de la apertura del puerto: `app.ts` permite preparar el servidor para pruebas y `server.ts` controla su ejecución. Los clientes de persistencia, autenticación y observabilidad necesarios se conectan desde infraestructura, no desde componentes de React.

`plugins/` adapta esas capacidades al ciclo de vida de Fastify y `infrastructure/` mantiene los clientes y adaptadores necesarios. Evita dos instancias equivalentes por registrar un plugin. Define el orden de registro: configuración y recursos, plugin OpenAPI antes de las rutas que documenta, autenticación y módulos según sus dependencias. `server.ts` se encarga también del cierre ordenado.

Declara esquemas de entrada y salida en las rutas y usa validación en ejecución, por defecto con JSON Schema según las capacidades de Fastify. Elige una estrategia de tipado compatible y mantén alineados tipos y esquemas. Las consultas de autorización o comprobación de disponibilidad van a hooks o casos de uso adecuados, no a validadores de formato que ejecuten acceso a datos. Los esquemas de respuesta exponen solo los campos públicos de cada operación.

La autenticación identifica al operador y el caso de uso comprueba sus permisos y alcance empresarial, aplica reglas y coordina persistencia e integraciones. No confíes en un identificador de empresa recibido del navegador como autorización. Las operaciones que requieren atomicidad deben completarse dentro de una transacción o una operación atómica apropiada en base de datos; varios llamados HTTP consecutivos no son una transacción.

Las integraciones encapsulan los detalles de los proveedores y sus estados de confirmación. Una operación con resultados parciales conserva información suficiente para consultar su estado o reintentar con control de duplicados. No añadas una cola o un proceso independiente sin una necesidad concreta.

## Contrato y comunicación entre las partes

Recorrido previsto: acción del usuario en frontend → petición HTTP autenticada → validación y caso de uso en backend → operación persistente → respuesta del contrato → actualización de la interfaz.

- Define por operación el método, ruta, entrada, salida, errores, permisos y estados. Los esquemas de la ruta Fastify son la fuente ejecutable; la conversación recoge las decisiones funcionales que deben reflejar. Un único responsable mantiene ambas cosas coherentes.
- Exporta OpenAPI desde esos esquemas y genera los tipos o cliente de frontend desde la exportación cuando se configure ese soporte. Comprueba la compatibilidad de plugins y generadores. Hasta entonces usa contratos explícitos; no declares una generación que aún no existe ni mantengas un OpenAPI manual contradictorio con el servidor.
- Mantén los tipos generados a partir de PostgreSQL dentro del acceso a datos del backend. Las respuestas públicas seleccionan únicamente los campos que necesita la interfaz; no devuelven automáticamente una fila completa de la base de datos.
- La validación se ejecuta al recibir datos en backend. El tipado TypeScript o la validación del formulario no sustituyen ese control.
- Una migración o un cambio de API actualiza los consumidores afectados y sus pruebas. No edites a mano archivos generados ni mantengas varias fuentes contradictorias para el mismo contrato.
- En desarrollo, configura las URLs y, si corresponde, el proxy o CORS para los orígenes necesarios. Separa configuración pública de la interfaz y configuración privada del servidor.

## Propiedad de archivos y ejecución local

| Superficie | Responsable previsto | Límite |
| --- | --- | --- |
| `frontend/` | Agente frontend | Interfaz y consumo del contrato. |
| `backend/` | Agente backend | API, reglas, persistencia e integraciones. |
| `backend/supabase/` | Backend o responsable de datos asignado | Una sola persona o agente mantiene cada migración. |
| `contracts/` | Responsable que designe el arquitecto | Los cambios se coordinan con ambas partes. |
| Configuración raíz | Arquitecto o ejecutor asignado | No cambia por iniciativas paralelas de ambos agentes. |

Al preparar la implementación, define por separado instalación, desarrollo, comprobación de tipos, pruebas y compilación para frontend y backend. Identifica versiones y variables necesarias sin copiar secretos. Si se usa Supabase local, la CLI se ejecuta desde `backend/` y el archivo de configuración correspondiente se crea al inicializarlo allí. Si se usa Supabase alojado, configura la conexión de desarrollo sin asumir que la base corre en localhost.

La aceptación integrada verifica el recorrido entre ambas aplicaciones y los permisos de los datos. Poder compilar cada parte por separado no acredita el funcionamiento conjunto.

## Entregas y comprobaciones que debe preparar el arquitecto

Para un pedido de proyecto completo, adapta este orden a sus dependencias. Para un cambio pequeño, define solo las tareas afectadas:

1. Acordar el flujo funcional de la primera entrega y sus permisos; concretar versiones y ejecución local.
2. Preparar configuración de frontend y backend, acceso del operador y esquema mínimo con migraciones. Definir los contratos de las primeras operaciones.
3. Entregar una funcionalidad de principio a fin, como registrar y consultar una aplicación, verificando los permisos y el resultado persistido.
4. Añadir contratos, personas, cuentas y asignaciones según el alcance; después importaciones, reportes e integraciones cuando sus datos y reglas estén definidos.

Frontend verifica tipos, compilación Vite y comportamiento de formularios y pantallas. Backend verifica tipos y operaciones Fastify, incluyendo respuestas y autorización; puede usar `fastify.inject()` para comprobar HTTP sin abrir un puerto. Los cambios de base de datos tienen comprobaciones de integridad y permisos con datos aislados. La generación de OpenAPI y cliente debe poder repetirse de forma reproducible cuando esté implementada.

La asignación concurrente del último cupo, las importaciones repetidas y una baja externa parcialmente fallida requieren casos específicos cuando se desarrollen esas funcionalidades. Define los comandos reales después de crear los paquetes; no presentes una lista de comandos previstos como pruebas ya ejecutadas.

## Datos y Supabase

- Separa la identidad del operador, sus pertenencias y permisos de las personas y cuentas registradas en el inventario.
- Distingue empresa pagadora, empresa de pertenencia y empresa beneficiaria. El dominio de correo es evidencia auxiliar, no una autorización.
- Modela producto, contrato, plan y asignación según el caso. No mezcles capacidad contratada, acceso habilitado y actividad observada en un mismo indicador.
- Define claves, restricciones e índices a partir de las operaciones. Versiona esquema y permisos en `backend/supabase/migrations/`, mediante cambios coherentes con los contratos.
- Cuando haya tablas expuestas mediante la Data API, define tanto privilegios como políticas RLS para las acciones y filas autorizadas. No supongas que una conexión de servidor privilegiada queda limitada por RLS: comprueba su rol y aplica los controles que correspondan.
- Mantén claves secretas fuera del navegador. Almacenamiento de archivos, tareas programadas y otros servicios se añaden solo si el requerimiento los necesita.
- Identifica la modalidad local: web/API locales conectadas a Supabase alojado, o servicios Supabase locales también. Las instrucciones de arranque deben corresponder a esa modalidad.

## Reglas del dominio que afectan el diseño

| Capacidad | Decisión que debe estar clara antes de repartir implementación |
| --- | --- |
| Empresas y operadores | Quién puede consultar o modificar datos de cada empresa. |
| Personas y cuentas | Qué identifica a cada una y cómo se revisa una coincidencia ambigua. |
| Aplicaciones y contratos | Qué se contrata, quién paga y qué unidades o planes existen. |
| Asignaciones | Qué consume capacidad, qué se reserva y cómo se evita asignar dos veces el último cupo. |
| Importaciones | Qué fuente prevalece, qué campos se actualizan y cómo se tratan errores o repeticiones. |
| Altas y bajas | Qué cambios son locales, cuáles dependen del proveedor y cómo se confirma cada resultado. |
| Reportes | Qué representa cada cálculo y de dónde provienen sus datos y fechas. |

La definición de estructura debe indicar dónde se implementa cada decisión. No crees todos estos módulos si el pedido solo afecta una parte del gestor.

## Referencias consultadas

Consultadas el 2026-11-25; comprobar documentación y versiones vigentes cuando se implemente.

- [React: crear una aplicación](https://react.dev/learn/creating-a-react-app) y [TypeScript](https://react.dev/learn/typescript): fundamentos de la aplicación y su tipado. No establecen un único árbol de carpetas.
- [Bulletproof React: organización por funcionalidades](https://github.com/alan2207/bulletproof-react/blob/master/docs/project-structure.md): referencia comunitaria para separar aplicación, funcionalidades y componentes compartidos.
- [Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs) y [Fastify: modularidad y plugins](https://fastify.dev/docs/latest/Guides/Plugins-Guide/): referencias del servidor acordado y su organización modular.
- [Vite](https://vite.dev/guide/) y [variables de entorno](https://vite.dev/guide/env-and-mode): desarrollo, compilación y configuración pública del frontend.
- [Fastify: validación y serialización](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/) y [exportación OpenAPI](https://github.com/fastify/fastify-swagger): esquemas ejecutables y especificación pública.
- [Node.js: paquetes](https://nodejs.org/api/packages.html) y [Fastify: separación de aplicación y servidor para pruebas](https://fastify.dev/docs/latest/Guides/Testing/): referencias para los límites de configuración y la preparación del servidor.
- [Supabase: desarrollo local](https://supabase.com/docs/guides/local-development), [seguridad de la Data API](https://supabase.com/docs/guides/api/securing-your-api) y [RLS](https://supabase.com/docs/guides/database/postgres/row-level-security): operación local y límites de acceso.
- [Supabase: migraciones](https://supabase.com/docs/guides/local-development/database-migrations) y [generación de tipos](https://supabase.com/docs/guides/api/rest/generating-types): esquema versionado y tipado derivado de la base de datos.

Las estructuras anteriores y la adaptación al dominio son decisiones propuestas para este proyecto. Ninguna de estas referencias acredita una implementación correcta sin revisión y comprobaciones del resultado.
