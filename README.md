# Gestor de aplicaciones

Configuración de agentes para Claude Code que prepara el desarrollo de un gestor de aplicaciones, contratos, licencias y accesos para varias empresas.

El repositorio contiene los perfiles, sus instrucciones y las skills de referencia. La aplicación, sus paquetes y la infraestructura de Supabase todavía no están implementados.

## Agentes

| Perfil | Responsabilidad | Skill precargada |
| --- | --- | --- |
| [GestorAplication - Arquitect](.claude/agents/gestoraplication-arquitect.md) | Analiza requisitos, define la estructura global y los contratos, asigna tareas y revisa las entregas. | `gestor-arquitectura` |
| [GestorAplication - Backend](.claude/agents/gestoraplication-backend.md) | Implementa API, validaciones, reglas, permisos y persistencia dentro de las rutas asignadas. | Ninguna |
| [GestorAplication - Frontend](.claude/agents/gestoraplication-frontend.md) | Implementa pantallas, formularios, tablas e interacciones con diseño profesional y accesibilidad. | `emil-design-eng` |

Los tres perfiles seleccionan `model: opus`. Backend y frontend tienen deshabilitada la herramienta `Agent`; la coordinación corresponde al arquitecto. Los permisos restantes dependen de la sesión de Claude Code.

Solo el arquitecto mantiene la estructura global. Cada ejecutor recibe el resultado esperado, las rutas de edición y consulta, los contratos, las dependencias y las comprobaciones de su tarea.

## Tecnologías acordadas

- Frontend: React, TypeScript y Vite.
- Backend: Fastify sobre Node.js y TypeScript.
- Datos y autenticación: PostgreSQL y Auth de Supabase.

Las versiones y herramientas complementarias se concretarán durante la implementación. La estructura completa se conserva en la [guía del arquitecto](.claude/skills/gestor-arquitectura/references/estructura-del-gestor.md).

## Uso con Claude Code

Con Git y Claude Code instalados y una sesión autenticada con acceso a Opus:

```powershell
git clone https://github.com/andreecito/gestor-de-aplicaciones.git
cd gestor-de-aplicaciones
claude --model opus --agent gestoraplication-arquitect --name "GestorAplication - Arquitect"
```

Ejemplo de encargo inicial:

> Analiza el funcionamiento esperado del gestor. Define los módulos y contratos necesarios y prepara tareas acotadas para backend y frontend, indicando rutas de trabajo, dependencias y criterios de aceptación. Por ahora entrega el análisis y las tareas, sin implementar la aplicación.

Las reglas compartidas y el contexto del proyecto están en [CLAUDE.md](CLAUDE.md). El prototipo original y sus datos personales no se incluyen en este repositorio; si una tarea necesita inspeccionarlo, hay que disponer de su ubicación local.

## Skills y procedencia

`gestor-arquitectura` es una guía propia del proyecto. La skill de diseño `emil-design-eng`, de [Emil Kowalski](https://github.com/emilkowalski/skills), se incluye con su referencia `choreograph` y las licencias MIT originales. La [procedencia](.claude/skills/emil-design-eng/SOURCE.md) documenta la revisión instalada.

Las skills aportan instrucciones reutilizables; no entrenan permanentemente el modelo. La validación de los perfiles comprueba la configuración y no acredita por sí sola una aplicación o integración operativa.
