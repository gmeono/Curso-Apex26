# Temario del Curso — Desarrollo Oracle APEX 26 con Git y Claude Code

## Descripción general

Curso modular de **10 sesiones de hasta 1 hora cada una**, dirigido a desarrolladores Oracle con dominio de SQL/PL-SQL y conocimiento de principiante en APEX, sin experiencia previa en Git.

Al finalizar, el participante podrá:

1. Usar Git de forma autónoma para control de versiones, principalmente desde VS Code.
2. Entender qué hace cada componente del ambiente de desarrollo local (`02-componentes-ambiente-local.md`).
3. Instalar y configurar ese ambiente por sí mismo (`CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md`).
4. Modificar aplicaciones APEX existentes siguiendo el flujo de trabajo en equipo del proyecto (`01-estrategia-git-sqlcl-projects.md`).

## Prerrequisitos por participante

- Laptop con Windows, permisos para instalar software (o soporte de TI disponible durante las sesiones 4 y 5).
- Cuenta de GitHub y acceso al repositorio del curso.
- Acceso a una base de datos Oracle de desarrollo (compartida o individual) con APEX 26.1 instalado.
- Conocimientos previos: SQL, PL/SQL básico. No se requiere experiencia previa en Git ni en APEX.

## Aplicación de práctica

Se usa una **aplicación de ejemplo incluida en todo workspace de APEX** (por ejemplo, "Sample Database Application"), en vez de una aplicación propietaria del equipo. Esto permite que todos trabajen sobre lo mismo desde el día 1 sin depender de datos o aplicaciones reales del negocio. El facilitador debe confirmar, antes de la Sesión 7, que la app de ejemplo está disponible en el workspace de DEV que se usará durante el curso.

## Materiales usados en el curso

| Documento | Se usa en |
|---|---|
| `CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md` | Sesiones 4, 5 |
| `02-componentes-ambiente-local.md` | Sesión 3 |
| `01-estrategia-git-sqlcl-projects.md` | Sesiones 6, 9, 10 |
| `04-guia-ejercicios-practicos.md` | Sesiones 1, 2, 4, 5, 7, 8, 9, 10 |

---

## Sesión 1 — Git: conceptos fundamentales

**Tipo:** Conceptual + demo
**Objetivo:** Entender qué problema resuelve Git y los conceptos base: repositorio, commit, staging area, historial.
**Sin APEX todavía** — se usa una carpeta de práctica simple con archivos de texto.

**Contenido:**
- Apertura: por qué existe el control de versiones (escenario "antes/después de Git", ej. el clásico `proyecto_final_v2_ULTIMO.docx`)
- Repositorio, working directory, staging area, commit — explicado con diagrama y analogías (staging = carrito de compras, commit = fotografía del proyecto)
- El panel *Source Control* de VS Code
- Trabajar con varios archivos: el staging es selectivo, no todo o nada
- Deshacer un cambio no confirmado (Discard Changes) — la red de seguridad de Git
- Ver diferencias (diff) y comparar commits, incluso no consecutivos, en el historial
- Cierre: quiz de repaso de los conceptos de la sesión

**Resultado esperado:** cada participante crea su primer repositorio local, trabaja con varios archivos, hace varios commits, descarta un cambio no confirmado, y puede explicar la diferencia entre "cambios sin guardar", "staged" y "commit".

---

## Sesión 2 — Git: ramas y colaboración

**Tipo:** Práctica (repo simple, sin APEX)
**Objetivo:** Aprender ramas, fusión, conflictos y Pull Requests, todo desde VS Code.

**Contenido:**
- Crear y cambiar de rama desde VS Code
- Fusionar ramas (merge)
- Provocar y resolver un conflicto con el editor de fusión de VS Code
- Instalar y usar la extensión "GitHub Pull Requests and Issues": crear, revisar y fusionar un PR
- Referencia rápida: cómo se vería lo mismo en GitHub Desktop

**Resultado esperado:** cada participante crea una rama, la sube, abre un Pull Request, lo revisa con un compañero y lo fusiona.

---

## Sesión 3 — Componentes del ambiente de desarrollo local

**Tipo:** Conceptual (uso de `02-componentes-ambiente-local.md`)
**Objetivo:** Entender qué es y para qué sirve cada pieza antes de instalarlas.

**Contenido:**
- Recorrido guiado por el diagrama general del ambiente
- VS Code, extensión Claude Code, `CLAUDE.md`, skills
- Oracle SQL Developer for VS Code vs. SQLcl MCP
- SQLcl, SQLcl Projects, APEXlang
- Preguntas frecuentes de seguridad ("¿el asistente puede borrar algo sin que yo lo note?")

**Resultado esperado:** cada participante puede explicar, con sus propias palabras, para qué sirve cada componente antes de instalarlo.

---

## Sesión 4 — Instalación del ambiente (parte 1)

**Tipo:** Práctica (setup guide, Fases 0–4)
**Objetivo:** Dejar el checkout de Git y las herramientas base de Windows listas.

**Contenido:**
- Verificación del repositorio y la extensión Claude Code (ya instalada)
- Preflight de Windows: versiones de Java, SQLcl, Git, VS Code
- Actualización de Java/SQLcl si es necesario
- Configuración del repositorio Git (`.gitignore`, branch inicial)

**Resultado esperado:** cada participante tiene su checkout local listo y las versiones de herramientas verificadas.

---

## Sesión 5 — Instalación del ambiente (parte 2)

**Tipo:** Práctica (setup guide, Fases 5–11)
**Objetivo:** Dejar el ambiente completamente funcional: conexión a DEV, SQLcl Project, MCP y `CLAUDE.md`.

**Contenido:**
- Crear/verificar la conexión SQLcl `apex-dev`
- Inicializar el SQLcl Project en el repositorio
- Instalar los skills de Oracle APEX y Base de Datos
- Configurar `.mcp.json` (SQLcl como servidor MCP) y `.claude/settings.json`
- Crear `CLAUDE.md` del proyecto
- Verificación final: el asistente puede conectarse a DEV y ejecutar una consulta de prueba

**Resultado esperado:** ambiente 100% funcional, verificado con el checklist de la Fase 26 del setup guide.

---

## Sesión 6 — Estrategia de equipo (Git + SQLcl Projects)

**Tipo:** Conceptual (uso de `01-estrategia-git-sqlcl-projects.md`)
**Objetivo:** Entender cómo el equipo trabajará junto después del curso.

**Contenido:**
- El modelo de ramas por ambiente: `desarrollo` (DEV), `pruebas` (TEST), `main` (PROD)
- La regla central: todo cambio nace en `desarrollo`, sin excepción — ni siquiera los hotfixes de producción
- Promoción por lote (normal) vs. cherry-pick (excepción documentada para urgencias)
- Cómo se promueve el código: Pull Request `desarrollo → pruebas`, luego `pruebas → main`, usando SQLcl Project releases y tags de versión
- Reglas de Pull Request del equipo
- Anti-patrones a evitar

**Resultado esperado:** cada participante entiende el flujo completo antes de aplicarlo en las sesiones prácticas siguientes.

---

## Sesión 7 — Primer ciclo de desarrollo APEX

**Tipo:** Práctica
**Objetivo:** Completar el ciclo completo de un cambio APEX simple, de principio a fin.

**Contenido:**
- Detectar y exportar la aplicación de ejemplo como APEXlang
- Crear una rama `feature/...` desde `desarrollo`
- Modificar una página de la aplicación de ejemplo
- Validar (`apex validate`) y corregir errores
- Importar a `apex-dev` y verificar el resultado

**Resultado esperado:** cada participante completa un cambio real de APEX validado e importado a DEV.

---

## Sesión 8 — Ciclo de desarrollo con PL/SQL

**Tipo:** Práctica
**Objetivo:** Completar un ciclo de cambio que combina PL/SQL y APEX.

**Contenido:**
- Modificar un paquete PL/SQL de la aplicación de ejemplo
- Compilar contra DEV y revisar `USER_ERRORS`
- Actualizar la página APEX relacionada con el cambio
- Ejecutar los scripts de diagnóstico (`health-check.sql`, `invalid-objects.sql`)

**Resultado esperado:** cada participante practica el orden de dependencia correcto: base de datos primero, APEX después.

---

## Sesión 9 — Flujo de equipo en acción

**Tipo:** Práctica
**Objetivo:** Simular el trabajo real en equipo, incluyendo un conflicto.

**Contenido:**
- Cada participante trabaja en su propia rama (desde `desarrollo`) sobre la misma aplicación de ejemplo
- Se provoca intencionalmente un conflicto (dos personas modifican la misma página o el mismo paquete)
- Resolución del conflicto siguiendo las reglas de Pull Request del Documento 1
- Revisión cruzada de Pull Requests entre compañeros hacia `desarrollo`

**Resultado esperado:** cada participante vive, en un entorno controlado, la dinámica real de colaboración que usará en el trabajo diario.

---

## Sesión 10 — Simulacro de promoción y cierre del curso

**Tipo:** Práctica + cierre
**Objetivo:** Completar el ciclo con la promoción de una versión a través de los ambientes.

**Contenido:**
- Pull Request `desarrollo → pruebas` (promoción por lote)
- `project stage` → `project verify` → `project release -version X.Y.Z` → `project gen-artifact`
- Crear el tag `vX.Y.Z-pruebas` y simular el despliegue a PRUEBAS (`project deploy`)
- Pull Request `pruebas → main`, usando el mismo artefacto ya probado
- Crear el tag `vX.Y.Z` (producción) y simular el despliegue a PRODUCCIÓN
- Ejemplo guiado de una corrección urgente por cherry-pick (sin editar `pruebas` ni `main` directamente)
- Checklist final de cierre del curso
- Espacio de preguntas abiertas

**Resultado esperado:** cada participante entiende y puede ejecutar el ciclo completo, desde una rama de feature hasta un release promovido a través de los dos saltos de ambiente (`desarrollo → pruebas → main`), y sabe dónde encontrar cada documento de referencia después del curso.

**Nota para el facilitador:** si no se cuenta con un ambiente físico de PRUEBAS separado para el curso, esta sesión puede simularse usando una segunda conexión/esquema de práctica, dejando explícito a los participantes que en su proyecto real el destino sería el ambiente de PRUEBAS verdadero.
