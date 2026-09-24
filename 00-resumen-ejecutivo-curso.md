# Resumen del Curso — Git y Desarrollo Oracle APEX 26 para el Equipo

## 1. Datos generales

- **Curso:** Control de versiones con Git y desarrollo Oracle APEX 26 con SQLcl Projects
- **Público objetivo:** 2 desarrolladores Oracle del equipo, con dominio de SQL/PL-SQL y conocimiento de principiante en APEX. Sin experiencia previa en Git.
- **Modalidad:** sesiones modulares, trabajando directamente sobre VS Code con la extensión Claude Code
- **Duración estimada:** 10 sesiones de aproximadamente 1 hora cada una (tiempo estimado)

## 2. Contexto y justificación

Actualmente el equipo desarrolla en SQL/PL-SQL y Oracle APEX sin un sistema de control de versiones. Esto significa:

- No existe historial de qué cambió, cuándo y por qué en el código de base de datos ni en las aplicaciones APEX.
- No hay un paso de revisión antes de que un cambio se aplique.
- No existe un proceso definido para mover un cambio de forma segura entre los tres ambientes (desarrollo, pruebas, producción) — el criterio de cuándo y cómo promover algo depende de la memoria individual, no de un proceso documentado.

## 3. Objetivo general

Que el equipo adquiera la capacidad de desarrollar y mantener aplicaciones Oracle APEX de forma colaborativa, trazable y seguro entre los tres ambientes del proyecto, usando Git y SQLcl Projects como parte de su flujo de trabajo diario.

### Objetivos específicos

1. El equipo domina los conceptos básicos de Git y lo usa de forma autónoma desde VS Code.
2. El equipo entiende qué es y para qué sirve cada componente del ambiente de desarrollo local.
3. El equipo es capaz de instalar y configurar su propio ambiente de desarrollo APEX 26/SQLcl Projects sin depender de terceros.
4. El equipo puede modificar aplicaciones APEX existentes siguiendo un flujo de trabajo colaborativo real, incluyendo la promoción de cambios entre ambientes.

## 4. Alcance del curso — temas a tratar

| Bloque | Contenido | Sesiones |
|---|---|---|
| 1. Fundamentos de Git | Conceptos base, ramas, colaboración y Pull Requests, todo desde VS Code | 1–2 |
| 2. Ambiente de desarrollo local | Qué es y para qué sirve cada componente (VS Code, extensión Claude Code, SQLcl, SQLcl Projects, APEXlang) | 3 |
| 3. Instalación y configuración | Montaje del ambiente completo siguiendo la guía de instalación del proyecto | 4–5 |
| 4. Estrategia de equipo | Cómo el equipo trabajará junto: ramas por ambiente (desarrollo/pruebas/producción), Pull Requests, promoción de cambios | 6 |
| 5. Práctica de desarrollo APEX | Ciclo completo de desarrollo — APEXlang, PL/SQL, trabajo en equipo y promoción entre ambientes — sobre una aplicación de práctica | 7–10 |

## 5. Resultado esperado

Al finalizar, el equipo cuenta con:

- Capacidad de trabajar con control de versiones de forma autónoma, sin depender de apoyo externo para las tareas básicas de Git.
- Un proceso de promoción de cambios entre desarrollo, pruebas y producción estandarizado y trazable — cada cambio queda documentado, revisado, y es posible saber exactamente qué versión está en cada ambiente en cualquier momento.
- Un ambiente de desarrollo local reproducible, que cualquier miembro del equipo puede instalar siguiendo la misma guía.
- Documentación propia del equipo que queda como activo permanente — no solo como apuntes de un curso puntual, sino como el proceso de trabajo real que el equipo sigue después.

## 6. Tiempo y recursos necesarios

- **Tiempo:** 10 sesiones de aproximadamente 1 hora cada una (estimado), en formato modular — no requiere bloquear al equipo un día completo.
- **Herramientas:** sin costo adicional. Todo el curso corre sobre herramientas ya disponibles: VS Code, la extensión Claude Code (ya instalada), Oracle SQL Developer for VS Code, y extensiones gratuitas de VS Code (GitHub Pull Requests and Issues, Git Graph).
- **Logística necesaria:** acceso a una base de datos de DESARROLLO con Oracle APEX 26.1, y preferiblemente un ambiente de PRUEBAS separado (aunque no indispensable — el simulacro de promoción puede explicarse sin ejecutarse si no hay uno disponible).

## 7. Materiales que se entregan

Documentación que queda como activo del equipo después del curso, no solo como material de clase. Se entrega en formato Word (.docx):

| Documento | Contenido |
|---|---|
| Guía de instalación y configuración del ambiente | Guía técnica de instalación y configuración del ambiente de desarrollo |
| Estrategia de Git y SQLcl Projects | El proceso de trabajo en equipo: ramas por ambiente, Pull Requests, promoción entre desarrollo/pruebas/producción |
| Componentes del ambiente de desarrollo local | Qué es y para qué sirve cada componente del ambiente de desarrollo |
| Temario del curso | Guía detallada de las 10 sesiones del curso |
| Guía de ejercicios prácticos | Ejercicios paso a paso para cada sesión práctica |
