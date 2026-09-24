# Componentes del Ambiente de Desarrollo Local

## Objetivo de este documento

Explicar, en lenguaje simple, **qué es y para qué sirve** cada componente del ambiente de desarrollo que se instala en `CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md`. La idea es que, antes de instalar nada, cada participante entienda qué papel juega cada pieza y por qué se necesita.

## Vista general

```text
VS Code
│
├── Extensión Claude Code          ← el "asistente" que ejecuta tareas por ti
│   ├── CLAUDE.md                  ← las reglas que ese asistente debe seguir en este proyecto
│   ├── .mcp.json                  ← cómo el asistente se conecta a la base de datos
│   └── .claude/skills/            ← conocimiento especializado de Oracle APEX/DB que el asistente puede usar
│
├── Oracle SQL Developer for VS Code   ← tu "ventana" humana a la base de datos
│
├── Git (+ extensión GitHub Pull Requests)  ← control de versiones del código
│
└── SQLcl 26.1+
    ├── SQLcl Projects              ← organiza el código de base de datos/APEX como archivos de proyecto
    ├── APEXlang                    ← representa una app APEX como archivos de texto editables
    └── SQLcl MCP                   ← el "puente" que permite al asistente usar SQLcl de forma controlada
           │
           ▼
      Base de datos Oracle (DEV)
          │
          └── Aplicación APEX 26.1
```

Ninguna de estas piezas reemplaza a las demás — cada una resuelve un problema distinto, y juntas forman el flujo completo: **editar código → validarlo/compilarlo → verlo funcionar en la base de datos → guardar el progreso en Git**.

---

## VS Code

**Qué es:** el editor de código donde vivirá casi todo el trabajo diario: editar archivos, ver diferencias de código, manejar Git, y conversar con la extensión de Claude Code.

**Para qué sirve aquí:** es el "hub" central. Todas las demás piezas (SQL Developer, Git, Claude Code) aparecen como paneles o extensiones dentro de VS Code, en vez de ser programas separados que hay que ir cambiando.

**Analogía:** si el proyecto fuera una obra de construcción, VS Code es la oficina donde están todos los planos, herramientas y comunicación — no construye por sí solo, pero es donde ocurre todo el trabajo.

---

## Extensión Claude Code

**Qué es:** un asistente de inteligencia artificial integrado en VS Code, capaz de leer el código del proyecto, editar archivos, ejecutar comandos (con tu aprobación) y usar herramientas como SQLcl.

**Para qué sirve aquí:** en vez de escribir manualmente cada `apex validate`, cada script de compilación o cada consulta a `USER_ERRORS`, se le puede pedir en lenguaje natural ("valida esta app y corrige los errores") y el asistente ejecuta los pasos, mostrando qué hace en cada momento y pidiendo confirmación antes de acciones sensibles (borrar datos, desplegar a producción, etc.).

**Importante:** no reemplaza el criterio del desarrollador — el asistente sigue las reglas definidas en `CLAUDE.md` y pide aprobación humana para cualquier cosa riesgosa.

---

## `CLAUDE.md`

**Qué es:** un archivo de texto en la raíz del repositorio con instrucciones específicas del proyecto para la extensión Claude Code (ej. "usa siempre la conexión `apex-dev`", "nunca uses producción sin permiso explícito").

**Para qué sirve aquí:** asegura que el asistente se comporte de forma consistente y segura en este proyecto en particular, sin que cada desarrollador tenga que repetir las mismas instrucciones cada vez que abre una conversación.

**Analogía:** es como el manual de inducción que le entregarías a cualquier persona nueva del equipo — excepto que aquí lo "lee" el asistente en cada sesión.

---

## `.claude/skills/` (Skills de Oracle APEX y Base de Datos)

**Qué es:** paquetes de conocimiento especializado (publicados por Oracle) que le enseñan a la extensión Claude Code las reglas específicas de APEXlang y de desarrollo Oracle — sintaxis correcta, buenas prácticas, comandos válidos de SQLcl — en vez de depender solo de lo que el modelo ya sabía de forma general.

**Para qué sirve aquí:** reduce el riesgo de que el asistente "invente" sintaxis de APEXlang que no existe, o sugiera comandos de SQLcl incorrectos.

---

## Oracle SQL Developer for VS Code

**Qué es:** la extensión oficial de Oracle para explorar bases de datos directamente desde VS Code: ver tablas, vistas, paquetes, ejecutar consultas SQL manualmente, gestionar conexiones.

**Para qué sirve aquí:** es la herramienta que un desarrollador humano usa para **mirar e investigar** la base de datos — revisar una tabla, probar una consulta suelta, diagnosticar un problema — sin necesidad de pasar por el asistente ni por la línea de comandos.

**Diferencia clave con SQLcl MCP:** esta extensión es para uso manual e interactivo. SQLcl MCP (más abajo) es el canal que usa el asistente para ejecutar acciones de forma automatizada. Ambas pueden apuntar a la misma base de datos DEV sin problema.

---

## Git

**Qué es:** el sistema de control de versiones que registra el historial de cambios del código: quién cambió qué, cuándo, y permite volver atrás o trabajar en paralelo sin pisarse el trabajo entre compañeros.

**Para qué sirve aquí:** todo el código de base de datos (paquetes, vistas) y de APEX (`.apx`) vive en un repositorio Git. Es la base de la estrategia de trabajo en equipo descrita en `01-estrategia-git-sqlcl-projects.md`.

En este curso, Git se usa principalmente desde el panel **Source Control** de VS Code, no desde la línea de comandos ni desde una aplicación aparte — ver `03-temario-curso.md` y `04-guia-ejercicios-practicos.md`.

---

## Extensión "GitHub Pull Requests and Issues"

**Qué es:** una extensión de VS Code (mantenida por GitHub/Microsoft) que permite crear, revisar, comentar y fusionar Pull Requests sin salir del editor.

**Para qué sirve aquí:** el flujo de trabajo en equipo (ver Documento 1) depende de Pull Requests para que alguien más revise cada cambio antes de integrarlo a `main`. Esta extensión evita tener que alternar entre VS Code y el navegador para eso.

---

## Git Graph (opcional)

**Qué es:** una extensión de VS Code que dibuja visualmente el historial de ramas y commits del repositorio.

**Para qué sirve aquí:** ayuda a "ver" conceptos como ramas, merges y el modelo trunk-based del Documento 1, en vez de imaginarlos solo a partir de texto.

---

## SQLcl

**Qué es:** el cliente de línea de comandos de Oracle para ejecutar SQL y PL/SQL, y la herramienta base sobre la que corren tanto SQLcl Projects como APEXlang.

**Para qué sirve aquí:** es el motor detrás de casi todo lo que ocurre entre el repositorio y la base de datos: exportar objetos, compilar paquetes, exportar/importar APEXlang, y generar los artefactos de release.

---

## SQLcl Projects

**Qué es:** una funcionalidad de SQLcl que organiza el código de base de datos y APEX de un esquema como una estructura de archivos versionable (`src/`, `.dbtools/`, `dist/`, `artifact/`), en vez de vivir únicamente dentro de la base de datos.

**Para qué sirve aquí:** es lo que permite tratar la base de datos "como código" — exportarla a archivos de texto, editarla localmente, versionarla en Git, y generar artefactos de despliegue reproducibles con `project export / stage / verify / release / gen-artifact / deploy`.

---

## APEXlang

**Qué es:** un formato de Oracle APEX 26.1 que representa una aplicación APEX completa (páginas, componentes compartidos, procesos) como archivos de texto (`.apx`) en vez de solo como metadata dentro de la base de datos.

**Para qué sirve aquí:** permite editar una aplicación APEX con las mismas herramientas y el mismo flujo que el resto del código (editor de texto, diffs, revisión en Pull Request), y validar los cambios (`apex validate`) antes de importarlos de vuelta a la base de datos (`apex import`).

**Importante:** el archivo `.apex/apexlang.json` es metadata generada por el compilador — nunca se edita a mano.

---

## SQLcl MCP (Model Context Protocol)

**Qué es:** un modo especial en el que SQLcl se expone como una herramienta que la extensión Claude Code puede invocar directamente, en vez de requerir que un humano escriba cada comando.

**Para qué sirve aquí:** es el "puente" que permite pedirle al asistente cosas como "compila este paquete y dime si hay errores" y que realmente lo ejecute contra la base de datos DEV, dentro de límites de seguridad configurados (por ejemplo, restringiendo qué tipo de comandos puede ejecutar — ver el nivel de restricción `-R 1` en el setup guide).

**Analogía:** si SQL Developer es "el volante" que maneja un humano, SQLcl MCP es el "piloto automático" que solo puede operar dentro de los límites que se le configuraron.

---

## Cómo interactúan entre sí (flujo típico)

```text
1. Desarrollador pide un cambio en lenguaje natural
        │
        ▼
2. Extensión Claude Code lee CLAUDE.md y las skills relevantes
        │
        ▼
3. Inspecciona el código actual (Git) y la base de datos (vía SQLcl MCP)
        │
        ▼
4. Edita archivos: PL/SQL y/o APEXlang (.apx)
        │
        ▼
5. Compila / valida usando SQLcl (vía MCP)
        │
        ▼
6. Importa a la base de datos DEV y verifica el resultado
        │
        ▼
7. El desarrollador revisa el diff, confirma en Oracle SQL Developer si quiere,
   y hace commit + Pull Request usando Git desde VS Code
```

## Preguntas frecuentes

**¿Necesito saber programar en Python o Java para usar esto?**
No. El asistente y las herramientas manejan la parte técnica de ejecución; el conocimiento que sí necesitas es SQL/PL-SQL y los conceptos de Git y APEX que cubre este curso.

**¿El asistente puede borrar o dañar algo sin que yo me dé cuenta?**
No debería: `CLAUDE.md` y la configuración de SQLcl MCP están diseñados para pedir aprobación explícita antes de operaciones destructivas o de despliegues a PRUEBAS/PRODUCCIÓN. Aun así, la revisión humana del diff antes de aprobar un Pull Request es la última línea de defensa.

**¿Qué pasa si prefiero hacer algo manualmente en vez de pedírselo al asistente?**
Perfecto — Oracle SQL Developer for VS Code y SQLcl desde la terminal siguen disponibles en todo momento para trabajo manual.
