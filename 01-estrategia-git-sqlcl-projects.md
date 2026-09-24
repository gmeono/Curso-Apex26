# Estrategia de Git + SQLcl Projects

## Objetivo de este documento

Este documento define **cómo el equipo va a trabajar día a día** una vez terminado el curso: cómo se organizan las ramas de Git, cómo se promueve el código entre ambientes, y qué se revisa antes de aprobar un cambio. No es solo material didáctico — es el proceso de trabajo real que el equipo debe seguir en proyectos APEX con SQLcl Projects.

## Contexto y supuestos

- Existen **3 ambientes**: DESARROLLO (DEV), PRUEBAS (TEST) y PRODUCCIÓN (PROD).
- Una feature puede **permanecer un tiempo indefinido en DESARROLLO o en PRUEBAS** antes de ser promovida al siguiente ambiente — esto ya es parte de la forma de trabajo del equipo, y la estrategia de Git debe reflejarlo directamente, no forzar promociones rápidas.
- El equipo tanto **modifica aplicaciones APEX existentes** como **crea aplicaciones nuevas**.
- El repositorio Git es compartido por todo el equipo (GitHub).
- Cada desarrollador tiene su propio ambiente local configurado según `CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md`, apuntando a una conexión SQLcl llamada `apex-dev`.
- El código fuente de base de datos y APEX vive en el repositorio como **SQLcl Project** (`src/database/`, `.dbtools/`, etc.).

Si alguno de estos supuestos cambia en la práctica (por ejemplo, ambientes con más de un esquema, o varios equipos compartiendo el mismo repositorio), este documento debe ajustarse antes de adoptarlo.

---

## Principios rectores

### 1. Cada ambiente tiene su propia rama permanente

En vez de una sola rama con promociones rápidas, cada ambiente tiene un lugar estable donde vivir mientras una feature espera su turno:

```text
desarrollo  →  DESARROLLO (DEV)
pruebas     →  PRUEBAS (TEST)
main        →  PRODUCCIÓN (PROD)
```

Este mapeo 1:1 es intencional: **la rama en la que está el código ES el ambiente en el que está.** No hace falta razonar sobre tags ni artefactos sueltos para saber dónde vive algo — se ve directamente en Git.

### 2. Todo cambio nace en `desarrollo` — sin excepción

`pruebas` y `main` **nunca** reciben un commit directo. Solo reciben código que llegó primero a `desarrollo`. Esto aplica también a correcciones urgentes encontradas en pruebas o en producción (ver más abajo) — no hay una vía alterna "por urgencia" que rompa esta regla, solo una forma más rápida de mover un cambio que ya nació en `desarrollo`.

Esta es la regla más importante del documento: si se respeta siempre, `pruebas` y `main` nunca pueden divergir de `desarrollo` — siempre son un subconjunto de su historia, nunca una rama paralela con vida propia.

### 3. Promoción por lote es la norma; promoción parcial es la excepción documentada

Cuando el equipo decide que lo acumulado en `desarrollo` está listo, se promueve **todo junto** mediante un Pull Request de `desarrollo` hacia `pruebas` (y luego de `pruebas` hacia `main`). Esta es la forma normal de trabajar.

Cuando algo es urgente y no puede esperar al siguiente lote (un bug bloqueante en pruebas, un error crítico en producción), se usa **cherry-pick de un commit puntual** — nunca se edita directamente el ambiente afectado. Ver la sección de correcciones urgentes más abajo.

### 4. Se construye una vez, se promueve el mismo artefacto

Lo que se prueba en PRUEBAS debe ser exactamente lo que llega a PRODUCCIÓN. Como `pruebas` nunca recibe cambios que no hayan pasado antes por `desarrollo`, el artefacto que se genera al promover a `pruebas` es el mismo que luego se despliega a `main` — no se vuelve a exportar ni regenerar para producción.

### 5. Aprobación humana explícita antes de PRUEBAS y PRODUCCIÓN

Ningún despliegue a PRUEBAS o PRODUCCIÓN ocurre automáticamente. Siempre hay una persona que da luz verde, y queda registrado qué versión se aprobó y quién lo hizo.

---

## Modelo de ramas

```text
feature/A ──●──●──┐
bugfix/X  ────●────┤
                    ▼
desarrollo   ──●────●────●──────────────●────────▶   (única fuente de todo cambio, sin excepción)
                                \
                                 │ PR "promover a pruebas" (por lote)
                                 ▼
pruebas      ─────────────────●──────●───────●───▶   (solo recibe merges desde desarrollo — nunca se edita directo)
                                        \
                                         │ PR "promover a producción"
                                         ▼
main         ──────────────────────────●─────────▶   (= estado real de PRODUCCIÓN)
```

- **`desarrollo`** — recibe todo Pull Request de toda rama de trabajo. Puede acumular varias features terminadas esperando su turno de promoción; eso es esperado y normal.
- **`pruebas`** — recibe únicamente Pull Requests que vienen de `desarrollo`. Una vez ahí, puede quedarse el tiempo que el equipo necesite para probar.
- **`main`** — recibe únicamente Pull Requests que vienen de `pruebas`, y representa en todo momento el estado real de PRODUCCIÓN.
- **`feature/<ticket>-<descripción>`** y **`bugfix/<ticket>-<descripción>`** — ramas cortas de trabajo, siempre creadas desde `desarrollo`, nunca desde `pruebas` ni `main`.

No existen ramas `hotfix/*` que partan de un tag de producción. Cualquier corrección, urgente o no, sigue naciendo en `desarrollo` (ver más abajo).

---

## Convenciones de nombres

| Elemento | Convención | Ejemplo |
|---|---|---|
| Rama de feature | `feature/<ticket-o-tema>` | `feature/APEX-102-reporte-facturas` |
| Rama de corrección | `bugfix/<ticket-o-tema>` | `bugfix/APEX-108-total-incorrecto` |
| Tag al promover a pruebas | `v<major>.<minor>.<patch>-pruebas` | `v1.2.0-pruebas` |
| Tag al promover a producción | `v<major>.<minor>.<patch>` | `v1.2.0` |
| Mensaje de commit | `<tipo>: <descripción breve>` | `fix: corrige cálculo de impuesto en pkg_facturas` |

Tipos de commit sugeridos: `feat`, `fix`, `refactor`, `docs`, `chore`.

---

## Flujo de trabajo diario (desarrollo en DEV)

```text
1. Actualizar desarrollo localmente (Pull)
2. Crear rama feature/... desde desarrollo
3. Editar APEXlang y/o PL/SQL localmente
4. Compilar y validar contra DEV (apex validate / compilación PL/SQL)
5. Confirmar que no quedan objetos INVALID ni errores de validación
6. Hacer commit(s) descriptivos
7. Push de la rama
8. Abrir Pull Request hacia desarrollo
9. Revisión de otro desarrollador
10. Fusionar (squash o merge, según convenga) y eliminar la rama
```

Este ciclo es el mismo tanto para modificar una app APEX existente como para crear una nueva, y es exactamente el mismo tanto para una feature nueva como para la corrección de un bug — la única diferencia está en el nombre de la rama (`feature/` vs `bugfix/`).

Una feature fusionada a `desarrollo` puede quedarse ahí indefinidamente sin causar ningún problema — no bloquea a nadie más, y no necesita "esperar" a nada para que otros sigan trabajando.

---

## Promoción a PRUEBAS

Se ejecuta cuando el equipo decide que el contenido acumulado en `desarrollo` está listo para pasar a PRUEBAS. No ocurre en cada Pull Request individual, y no requiere que absolutamente todo lo que hay en `desarrollo` esté "terminado en el mismo momento" — solo que lo que se promueve sea seguro de probar en conjunto.

```text
1. Pull Request: desarrollo → pruebas
2. Revisión y aprobación del PR
3. Merge
4. connect -name apex-dev
5. project export
6. project stage
7. project verify verify-stage
8. project release -version <X.Y.Z>
9. project gen-artifact ...
10. git tag vX.Y.Z-pruebas
11. git push origin vX.Y.Z-pruebas
12. project deploy -file <artefacto>   (contra la conexión de PRUEBAS)
```

Antes del paso 12, quien ejecuta la promoción debe confirmar explícitamente que la conexión usada es **PRUEBAS**, no DEV ni PROD.

## Promoción a PRODUCCIÓN

Solo después de que PRUEBAS fue validado por QA o por el usuario funcional:

```text
1. Pull Request: pruebas → main
2. Revisión y aprobación explícita del responsable de PRODUCCIÓN
3. Merge
4. Usar el MISMO artefacto ya generado y probado en PRUEBAS (no generar uno nuevo)
5. project deploy -file <mismo-artefacto>   (contra la conexión de PRODUCCIÓN)
6. git tag vX.Y.Z
7. git push origin vX.Y.Z
8. Verificar en PROD (health-check.sql, apex_release, objetos INVALID)
```

La aprobación de producción debe quedar registrada (por ejemplo, como comentario en el Pull Request, en el ticket, o en un canal del equipo) — no es un paso puramente técnico.

---

## Correcciones urgentes en PRUEBAS

Cuando se encuentra un bug en PRUEBAS que no puede esperar al siguiente lote de promoción:

```text
1. Crear bugfix/... desde desarrollo (NUNCA desde pruebas)
2. Corregir, compilar y validar contra DEV
3. Pull Request hacia desarrollo — se fusiona igual que cualquier otro cambio
4. Cherry-pick de ese commit específico hacia pruebas
   (una excepción documentada a la promoción por lote, no una vía paralela)
5. Desplegar solo ese cambio puntual a PRUEBAS
```

Como el fix ya existe en `desarrollo`, la siguiente promoción normal por lote lo incluirá de forma natural — no hay que recordar "reaplicarlo" en ningún lado, ni riesgo de perderlo.

## Correcciones urgentes en PRODUCCIÓN

Se sigue exactamente la misma regla, extendida un paso más para no saltarse la validación en pruebas incluso en una emergencia:

```text
1. Crear bugfix/... desde desarrollo (NUNCA desde main ni desde pruebas)
2. Corregir, compilar y validar contra DEV
3. Pull Request hacia desarrollo — se fusiona igual que cualquier otro cambio
4. Cherry-pick de ese commit hacia pruebas
5. Validación rápida en PRUEBAS (abreviada si la urgencia lo exige, pero no se omite)
6. Cherry-pick / Pull Request de ese mismo cambio hacia main
7. Desplegar a PRODUCCIÓN
```

Igual que con pruebas, como el fix nace en `desarrollo`, la siguiente promoción normal por lote lo arrastra de forma natural — no requiere ningún paso adicional de sincronización después.

---

## Reglas de Pull Request

Ningún Pull Request se aprueba si no cumple lo siguiente:

```text
[ ] La rama está actualizada respecto a su rama destino (sin conflictos pendientes)
[ ] El PL/SQL modificado compila sin errores en DEV
[ ] No quedan objetos INVALID nuevos por este cambio
[ ] El APEXlang modificado pasa `apex validate` sin errores (si aplica)
[ ] Se importó y probó en apex-dev (si el cambio incluye APEX)
[ ] No hay contraseñas, wallets ni credenciales en el diff
[ ] El mensaje de commit/PR describe qué cambia y por qué
[ ] Al menos otra persona del equipo revisó el diff
```

Para Pull Requests de promoción (`desarrollo → pruebas` y `pruebas → main`), además:

```text
[ ] Se confirmó explícitamente la conexión de destino antes de desplegar
[ ] Se generó el artefacto mediante project release / gen-artifact
[ ] Para pruebas → main: se está usando el mismo artefacto ya probado, no uno nuevo
[ ] Se etiquetó (git tag) el commit promovido
```

---

## Tabla resumen: rama ↔ comando SQLcl ↔ ambiente

| Ambiente | Rama permanente | Origen de los cambios | Mecanismo de promoción | Conexión SQLcl |
|---|---|---|---|---|
| Desarrollo | `desarrollo` | `feature/*`, `bugfix/*` vía PR | `project export` | `apex-dev` |
| Pruebas | `pruebas` | Solo merges desde `desarrollo` (lote o cherry-pick) | `project stage/verify/release/gen-artifact/deploy` | conexión de PRUEBAS |
| Producción | `main` | Solo merges desde `pruebas` | `project deploy` del mismo artefacto | conexión de PRODUCCIÓN |

---

## Qué NO hacer (anti-patrones)

- **No** hacer commits directos en `pruebas` o en `main`, bajo ninguna circunstancia — ni siquiera por urgencia. Toda corrección nace en `desarrollo`, incluso las de producción.
- **No** crear ramas `hotfix/*` desde un tag de producción — se crean desde `desarrollo`, como cualquier otra rama de trabajo.
- **No** volver a exportar/generar un artefacto distinto para producción — se promueve el mismo que se probó en pruebas.
- **No** desplegar a PRUEBAS o PRODUCCIÓN sin confirmar antes qué conexión está activa.
- **No** guardar contraseñas, wallets o `TNS_ADMIN` con credenciales dentro del repositorio.
- **No** editar manualmente `.apex/apexlang.json` — es metadata gestionada por el compilador.
- **No** fusionar un Pull Request con objetos `INVALID` o validación de APEXlang fallando.
- **No** usar cherry-pick como vía habitual de promoción — es la excepción para urgencias, no un atajo para evitar el proceso normal por lote.

---

## Referencias

- `CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md` — Fase 19 a 27 (ciclos de desarrollo, dependencias, workflow de release de SQLcl Project).
- Documentación oficial de SQLcl Projects: `help project`, `help project release`, `help project gen-artifact`.
