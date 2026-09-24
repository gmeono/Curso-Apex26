# Guía de Ejercicios Prácticos

Esta guía contiene los pasos concretos para las sesiones prácticas del curso (1, 2, 4, 5, 7, 8, 9 y 10). Úsala junto con `03-temario-curso.md` (para el contexto de cada sesión) y `01-estrategia-git-sqlcl-projects.md` / `02-componentes-ambiente-local.md` (para el "por qué" detrás de cada paso).

Convenciones usadas en esta guía:

```text
<TU-NOMBRE>       → identificador corto de cada participante, ej. "gmeono"
<DEV_SCHEMA>      → esquema de desarrollo asignado
<APP_ID>          → ID de la aplicación de ejemplo en DEV
<REPO-CURSO>      → repositorio Git compartido del curso
```

---

## Sesión 1 — Git: conceptos fundamentales

**Preparación:** cada participante crea una carpeta local vacía, por ejemplo `C:\curso-git\practica-01`.

### Ejercicio 1.1 — Primer repositorio

1. Abre la carpeta en VS Code (`Archivo > Abrir carpeta`).
2. Crea dos archivos: `notas.txt` y `tareas.txt`, cada uno con una línea de texto cualquiera.
3. Abre el panel **Source Control** (ícono de rama en la barra lateral izquierda, o `Ctrl+Shift+G`).
4. Haz clic en **Initialize Repository** (esto ejecuta `git init` por ti).
5. Observa que ambos archivos aparecen bajo "Changes" — este es el *working directory*.

### Ejercicio 1.2 — Staging selectivo y commit

1. Pasa el mouse sobre `notas.txt` en el panel Source Control y haz clic en el `+` (esto lo mueve al *staging area*) — **deja `tareas.txt` sin stage** para ver que el staging es selectivo, no todo o nada.
2. Escribe un mensaje de commit, por ejemplo `feat: primer archivo de práctica`, y confirma — observa que `tareas.txt` sigue pendiente, sin confirmar.
3. Haz stage y commit de `tareas.txt` por separado.
4. Repite con ambos archivos: edita, guarda, stage, commit, con un mensaje distinto cada vez — hasta tener al menos 3 commits en total.

### Ejercicio 1.3 — Deshacer un cambio no confirmado

1. Edita `notas.txt` y guarda, sin hacer stage ni commit.
2. En el panel Source Control, pasa el mouse sobre el archivo y usa la opción **Discard Changes** (ícono de flecha circular).
3. Verifica que el archivo volvió exactamente a como estaba en el último commit — este es el "botón de deshacer" de Git antes de confirmar algo.

### Ejercicio 1.4 — Ver el historial y diferencias

1. Haz clic en un archivo desde el panel Source Control cuando tenga cambios sin confirmar: VS Code muestra el **diff** (línea agregada/eliminada) en un panel dividido.
2. Abre el historial de commits: Command Palette (`Ctrl+Shift+P`) → `Git: View History (Git Log)` (o revisa la extensión **Timeline** en el panel del explorador, sobre el archivo).
3. Selecciona dos commits que no sean consecutivos y compáralos — ¿qué cambió entre ellos en total?
4. Identifica: ¿cuántos commits llevas? ¿qué cambió en cada uno?

**Cierre de la sesión:** cada participante debe tener un repositorio local con al menos 3 commits sobre 2 archivos, debe haber descartado un cambio no confirmado, y debe poder explicar la diferencia entre "cambios sin guardar", "staged" y "commit". Se cierra con el quiz de repaso de la sesión (preguntas abajo; versión interactiva disponible para usar en vivo durante el curso).

### Quiz de repaso — Sesión 1

1. **¿Qué es el "working directory"?** → La carpeta de tu proyecto tal como la ves y editas normalmente.
2. **¿Qué hace el "staging area"?** → Te permite elegir exactamente qué cambios incluir en el próximo commit.
3. **¿Qué es un commit?** → Una fotografía guardada del proyecto en un momento específico.
4. **¿Qué pasa al usar "Discard Changes" sobre un cambio no confirmado?** → El archivo vuelve exactamente a como estaba en el último commit.
5. **¿Para qué sirve el historial (Git Log / Timeline)?** → Para ver todos los commits y comparar qué cambió entre ellos, incluso entre commits no consecutivos.

---

## Sesión 2 — Git: ramas y colaboración

**Requisito previo:** repositorio remoto compartido (`<REPO-CURSO>`) creado en GitHub, con cada participante ya invitado como colaborador.

### Ejercicio 2.1 — Conectar el repositorio local a GitHub

1. En GitHub, crea (o usa) el repositorio `<REPO-CURSO>`.
2. En VS Code, Command Palette → `Git: Add Remote` → pega la URL del repositorio, nómbralo `origin`.
3. Sube tu historial: botón de sincronización en la esquina inferior izquierda, o Command Palette → `Git: Push`.

### Ejercicio 2.2 — Crear una rama y hacer un cambio

1. Haz clic en el nombre de la rama actual en la barra de estado (esquina inferior izquierda) → **Create new branch...**
2. Nómbrala `feature/<TU-NOMBRE>-practica-ramas`.
3. Edita `notas.txt`, guarda, haz stage y commit.
4. Sube la rama: botón de sincronización (VS Code te ofrecerá publicar la rama la primera vez).

### Ejercicio 2.3 — Crear, revisar y fusionar un Pull Request

1. Instala la extensión **GitHub Pull Requests and Issues** (Extensions view, si no está ya instalada para el curso).
2. Abre el panel de la extensión (ícono de GitHub en la barra lateral) → **Create Pull Request**, con destino `main`.
3. Pide a un compañero que revise tu PR desde el mismo panel (puede comentar directamente sobre líneas del diff).
4. Una vez aprobado, haz clic en **Merge Pull Request** desde VS Code.
5. Elimina la rama local y remota ya fusionada (VS Code lo ofrece automáticamente tras el merge).

### Ejercicio 2.4 — Provocar y resolver un conflicto

1. En pareja: ambos crean una rama distinta desde `main` y editan **la misma línea** de `notas.txt`.
2. El primero sube su rama y fusiona su PR sin problema.
3. El segundo, al intentar actualizar su rama (`Git: Pull` o `Git: Merge Branch... → main`), obtendrá un conflicto.
4. VS Code marca el archivo en conflicto y ofrece el editor de fusión de 3 vías: elige **Accept Current**, **Accept Incoming**, **Accept Both**, o edita manualmente.
5. Resuelve, haz stage del archivo resuelto, y completa el commit de merge.
6. Sube la rama y fusiona su propio Pull Request.

### Referencia — lo mismo en GitHub Desktop

Si en algún momento prefieres una vista alternativa: abre GitHub Desktop, selecciona el mismo repositorio, y verás las mismas ramas, cambios e historial. Crear una rama es el botón **New Branch**; un conflicto se resuelve con el botón **Open in [editor]**, que en este curso seguirá siendo VS Code.

**Cierre de la sesión:** cada participante debe haber fusionado al menos un Pull Request propio y resuelto al menos un conflicto.

---

## Sesión 4 — Instalación del ambiente (parte 1)

Sigue `CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md`, Fases 0 a 4, con estos puntos de control:

### Checkpoint 4.1 — Repositorio del proyecto

1. Clona (o abre) el repositorio del proyecto real del equipo — distinto del repositorio de práctica de las Sesiones 1–2.
2. Verifica en la terminal integrada (`` Ctrl+` ``):
   ```powershell
   git remote -v
   git branch --show-current
   ```
3. Confirma con el facilitador: ¿este es el repositorio correcto? ¿está vacío o ya tiene contenido?

### Checkpoint 4.2 — Herramientas de Windows

1. Ejecuta las verificaciones de la Fase 1 del setup guide (`code --version`, `java -version`, `sql -version`, extensión Claude Code activa).
2. Anota tus valores de `JAVA_PATH` y `SQLCL_PATH` — los necesitarás en la Sesión 5.
3. Si alguna versión no cumple el mínimo (Fase 2), pide ayuda al facilitador antes de continuar — no todos deben actualizar Java/SQLcl al mismo tiempo si ya cumplen la versión.

### Checkpoint 4.3 — `.gitignore` y ramas permanentes

1. Verifica o crea `.gitignore` según la Fase 4.3 del setup guide.
2. Confirma o crea las tres ramas permanentes del proyecto: `desarrollo`, `pruebas` y `main` (ver `01-estrategia-git-sqlcl-projects.md`). Si el repositorio recién se está creando, `desarrollo` es la rama de trabajo diario — la mayoría de los Pull Requests apuntarán ahí, no a `main`.

**Cierre de la sesión:** cada participante tiene el repositorio del proyecto real abierto en VS Code, con las versiones de herramientas verificadas y anotadas.

---

## Sesión 5 — Instalación del ambiente (parte 2)

Continúa con las Fases 5 a 11 del setup guide.

### Checkpoint 5.1 — Conexión a DEV

1. Sigue la Fase 5: crea o verifica tu conexión SQLcl `apex-dev`.
2. Verifica con:
   ```sql
   select user, sys_context('USERENV','DB_NAME') db_name from dual;
   select version_no from apex_release;
   ```

### Checkpoint 5.2 — SQLcl Project

1. Sigue la Fase 6: inicializa el SQLcl Project en la raíz del repositorio.
2. Sigue la Fase 7: exporta el baseline (`project export`) y revisa `git status` — no hagas commit todavía.

### Checkpoint 5.3 — Skills, MCP y `CLAUDE.md`

1. Sigue la Fase 8: instala los skills de Oracle APEX y Base de Datos.
2. Sigue la Fase 9: configura `.mcp.json` con tu `SQLCL_PATH` verificado en la Sesión 4.
3. Sigue la Fase 10: verifica que la extensión Claude Code puede conectarse vía MCP.
4. Sigue la Fase 11: crea/confirma `CLAUDE.md`.

### Checkpoint 5.4 — Verificación final

Ejecuta el checklist de la Fase 26 del setup guide. No continúes a la Sesión 6 si algún punto del checklist de "Local tools", "Repositorio" o "Base de datos" está pendiente.

**Cierre de la sesión:** ambiente 100% funcional y verificado.

---

## Sesión 7 — Primer ciclo de desarrollo APEX

### Ejercicio 7.1 — Identificar y exportar la aplicación de ejemplo

1. Conectado a `apex-dev`, identifica el `<APP_ID>` de la aplicación de ejemplo (Fase 12 del setup guide).
2. Exporta como APEXlang (Fase 13):
   ```text
   apex export -applicationid <APP_ID> -exptype apexlang
   ```

### Ejercicio 7.2 — Rama y cambio

1. Crea una rama `feature/<TU-NOMBRE>-practica-apex` desde `desarrollo`.
2. Elige una página sencilla de la aplicación de ejemplo (por ejemplo, un reporte) y modifícala: cambia un título, agrega una columna a un Interactive Report, o ajusta una condición de visualización.

### Ejercicio 7.3 — Validar e importar

1. Valida:
   ```text
   apex validate -input <PATH_TO_APEXLANG_APP>
   ```
2. Corrige cualquier error de validación.
3. Importa a `apex-dev`:
   ```text
   apex import -input <PATH_TO_APEXLANG_APP>
   ```
4. Verifica el cambio abriendo la aplicación en el navegador (Runtime de APEX).

### Ejercicio 7.4 — Commit

1. Revisa `git diff` y `git status`.
2. Haz stage y commit del cambio APEXlang desde el panel Source Control de VS Code.
3. No hagas Pull Request todavía — eso se practica en la Sesión 9.

**Cierre de la sesión:** cada participante tiene un cambio de página real, validado e importado a DEV, commiteado en su propia rama.

---

## Sesión 8 — Ciclo de desarrollo con PL/SQL

### Ejercicio 8.1 — Modificar un paquete

1. Ubica un paquete PL/SQL usado por la aplicación de ejemplo bajo `src/database/<DEV_SCHEMA>/package_body/`.
2. Realiza un cambio simple y no destructivo (por ejemplo, agregar un `NULL;` con un comentario, o ajustar un mensaje de log, evitando cambiar lógica de negocio real durante la práctica).

### Ejercicio 8.2 — Compilar y revisar errores

1. Compila:
   ```text
   @src/database/<DEV_SCHEMA>/package_spec/<paquete>.pks
   @src/database/<DEV_SCHEMA>/package_body/<paquete>.pkb
   ```
2. Ejecuta los scripts de diagnóstico:
   ```text
   @scripts/show-errors.sql
   @scripts/invalid-objects.sql
   ```
3. Si hay errores, corrígelos y vuelve a compilar hasta que el objeto quede `VALID`.

### Ejercicio 8.3 — Conectar el cambio con APEX

1. Si el paquete alimenta una página APEX de la aplicación de ejemplo, actualiza esa página (por ejemplo, el origen de un reporte) para reflejar el cambio.
2. Repite el ciclo de validación/importación de la Sesión 7 para esa página.

### Ejercicio 8.4 — Commit

1. Revisa `git diff` (debe incluir tanto el archivo `.pkb`/`.pks` como el `.apx` si aplica).
2. Haz commit del cambio completo.

**Cierre de la sesión:** cada participante practicó el orden correcto — base de datos primero, APEX después — y sabe leer `USER_ERRORS`.

---

## Sesión 9 — Flujo de equipo en acción

### Ejercicio 9.1 — Trabajo paralelo

1. En grupos de 2–3, cada participante crea su propia rama `feature/<TU-NOMBRE>-equipo` desde `desarrollo` (ya actualizado con los cambios de las Sesiones 7 y 8, fusionados previamente por el facilitador o por cada quien).
2. Cada quien realiza un cambio distinto en la misma aplicación de ejemplo.

### Ejercicio 9.2 — Conflicto intencional

1. El facilitador asigna a dos participantes del mismo grupo la misma página o el mismo paquete para modificar.
2. Ambos hacen commit y suben su rama.
3. El primero abre y fusiona su Pull Request sin problema.
4. El segundo actualiza su rama contra `desarrollo` y resuelve el conflicto resultante, siguiendo el mismo procedimiento de la Sesión 2 (Ejercicio 2.4), ahora sobre archivos APEXlang/PL-SQL reales.

### Ejercicio 9.3 — Revisión cruzada

1. Cada Pull Request debe ser revisado por un compañero antes de fusionarse, usando el checklist de "Reglas de Pull Request" del Documento 1.
2. El revisor debe confirmar explícitamente: ¿compiló sin errores? ¿validó APEXlang? ¿se probó en DEV?

**Cierre de la sesión:** todo el grupo tiene sus cambios fusionados a `desarrollo`, y `desarrollo` refleja el estado combinado de todos los cambios de la sesión — listo para promoverse en la Sesión 10.

---

## Sesión 10 — Simulacro de promoción y cierre

### Ejercicio 10.1 — Promover de `desarrollo` a `pruebas`

1. Con `desarrollo` actualizado tras la Sesión 9, abre un Pull Request `desarrollo → pruebas` con los cambios acumulados.
2. Revísalo en grupo usando la sección "Para Pull Requests de promoción" del checklist de Reglas de Pull Request (Documento 1).
3. Fusiona el PR.

### Ejercicio 10.2 — Generar el release y el artefacto

Conectado a `apex-dev`:

```text
project export
project stage
project verify verify-stage
```

Revisa la salida de `project verify verify-stage` en grupo — ¿hay advertencias? ¿qué significan?

```text
project release -version 1.0.0
project gen-artifact ...
```

Usa `help project gen-artifact` si la sintaxis exacta reportada por la versión instalada de SQLcl difiere.

### Ejercicio 10.3 — Tag y "despliegue" simulado a PRUEBAS

1. Desde VS Code: Command Palette → `Git: Create Tag`, nombra el tag `v1.0.0-pruebas` sobre el commit recién fusionado en `pruebas`, y publícalo (`Git: Push Tags`).
2. Si hay una conexión de PRUEBAS disponible para el curso, ejecuta:
   ```text
   project deploy -file <artefacto>
   ```
   contra esa conexión, y verifica con `scripts/health-check.sql`.
3. Si **no** hay un ambiente de PRUEBAS separado disponible, el facilitador explica en pantalla qué comando se ejecutaría y contra qué conexión, dejando claro que en el proyecto real este paso sí se ejecuta contra un ambiente físico distinto.

### Ejercicio 10.4 — Promover de `pruebas` a `main` (producción)

1. Abre un Pull Request `pruebas → main`, usando el **mismo artefacto** generado en el Ejercicio 10.2 — no se vuelve a exportar ni a generar uno nuevo.
2. Fusiona tras una aprobación explícita simulada (en el curso, el facilitador da esa aprobación en voz alta, para que quede claro que es un paso humano, no automático).
3. Crea el tag `v1.0.0` (sin sufijo, ya representa producción) sobre el commit fusionado en `main`, y publícalo.
4. Si hay una conexión de PRODUCCIÓN disponible para el curso (normalmente no la habrá), el facilitador explica en pantalla el `project deploy` correspondiente en vez de ejecutarlo.

### Ejercicio 10.5 — Corrección urgente por cherry-pick (demostración guiada)

Este ejercicio lo dirige el facilitador para todo el grupo, ya que solo hace falta un ejemplo, no que cada participante lo repita:

1. El facilitador simula un bug encontrado en `pruebas`.
2. Se corrige en una rama `bugfix/...` creada desde `desarrollo` — **nunca** directo en `pruebas`.
3. Se fusiona a `desarrollo` mediante Pull Request, igual que cualquier otro cambio.
4. Se hace cherry-pick de ese commit hacia `pruebas`: clic derecho sobre el commit en el panel de historial de Git (o Command Palette → `Git: Cherry Pick`) → seleccionar el commit del fix.
5. Discusión en grupo: ¿por qué no se edita `pruebas` directamente? ¿qué problema evita esta regla la próxima vez que se promueva `desarrollo → pruebas` por lote?

### Cierre del curso

1. Recorrido rápido del checklist final: ¿cada participante tiene su ambiente funcional? ¿entiende el modelo de ramas por ambiente (`desarrollo` → `pruebas` → `main`)? ¿sabe dónde están los 4 documentos de referencia?
2. Espacio abierto de preguntas.
3. Recordatorio de dónde queda todo para consulta futura: `CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md`, y los documentos 01 a 04 de este curso.
