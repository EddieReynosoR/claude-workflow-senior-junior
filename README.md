# claude-workflow

Tu marketplace personal de Claude Code. Contiene un plugin, `senior-junior`, con tu flujo de trabajo:

1. Llegas con un plan, o pides discovery.
2. Claude revisa el plan como senior.
3. El plan aprobado se guarda en un archivo.
4. Claude lo ejecuta solo cuando tú lo pides.
5. Te entrega un reporte para que tú evalúes lo que hizo.

## Skills

| Skill | Cuándo se activa | Qué hace |
|---|---|---|
| `/discovery <feature>` | Solo cuando tú la escribes | Lee el código y te hace hasta 5 preguntas de alcance. Luego propone un plan y lo ajusta contigo. Cuando lo apruebas, lo guarda. |
| `/review-plan [plan]` | Cuando la escribes, o cuando compartes un plan con tu plantilla | Hace un review senior con IDs (G1, E1, R1, U1, Q1) y tú respondes por ID. Cuando apruebas, guarda tu plan con solo los cambios que aceptaste. |
| `/execute [ruta] [pasos]` | Solo cuando tú la escribes | Implementa los pasos del plan y verifica cada uno. Se detiene ante cualquier decisión que el plan no cubra y al final te deja un reporte para tu review. |

El nombre completo de cada skill es `/senior-junior:<skill>` (por ejemplo `/senior-junior:discovery`). El nombre corto también funciona mientras ningún otro comando lo use.

Ojo: la skill de review se llama `review-plan` porque `/review` ya es el alias del `/code-review` integrado de Claude Code.

En el turno en que corren `/discovery` y `/review-plan`, las herramientas de edición están bloqueadas, así que no pueden escribir código.

## Instalación

### Una sola vez

1. Guarda esta carpeta fuera de tus proyectos, por ejemplo en `~/dev/claude-workflow`.
2. Registra el marketplace:

```bash
claude plugin marketplace add ~/dev/claude-workflow
```

### En cada proyecto donde lo quieras

```bash
cd ~/ruta/a/tu-proyecto
claude plugin install senior-junior@eduardo-workflow --scope local
```

`--scope local` lo activa solo para ti y solo en ese proyecto. La configuración queda en `.claude/settings.local.json`, y Claude Code agrega ese archivo a tu ignore global de git, así que tu repo no cambia. Si ya tenías una sesión abierta, corre `/reload-plugins`.

Si lo quieres en todos tus proyectos, usa `--scope user` en lugar de `--scope local`.

Para apagarlo en un proyecto:

```bash
claude plugin disable senior-junior@eduardo-workflow --scope local
```

## Flujo

1. **Con plan:** compártelo con tu plantilla (Problem / Done when / Edge cases / My approach / What I don't know), o usa `/review-plan`. Responde por ID y di "aprobado".
   **Sin plan:** corre `/discovery <feature>`, responde las preguntas, ajusta la propuesta y di "aprobado".
2. Claude guarda el plan en `docs/plans/AAAA-MM-DD-<feature>.md` con `Status: approved`. Si al plan le faltan pasos, primero te propone el desglose y espera tu OK.
3. Corre `/execute docs/plans/<archivo>.md`. Si solo hay un plan aprobado, basta con `/execute`. Para correr solo algunos pasos: `/execute <ruta> 2 3`.
4. Revisas el reporte. En ese momento el plan está en `Status: in-review`. Puedes pedir correcciones, o aprobar y el plan pasa a `Status: done`.

`/execute` se detiene y te pregunta cuando encuentra:
- una decisión que el plan no cubre;
- código que no coincide con el plan;
- una verificación que sigue fallando;
- una acción destructiva (borrar datos, migraciones, deploy a una org compartida, force push).

No hace commit ni push a menos que tú se lo pidas.

## Dónde quedan los planes

Por defecto quedan en `docs/plans/`, dentro del proyecto. Para cambiar la carpeta, agrega esta línea al `CLAUDE.md` del proyecto:

```
Plans directory: .plans/
```

Si no quieres versionar los planes, agrega esa carpeta a tu `.gitignore` o a tu ignore global.

## Editar las skills

Cada skill vive en `plugins/senior-junior/skills/<skill>/SKILL.md`. Como el marketplace es una carpeta local, Claude Code lee esos archivos directamente: tus cambios se aplican en la siguiente sesión o al correr `/reload-plugins`.

La plantilla del plan está repetida en `discovery` y en `review-plan`. Se repite a propósito: si viviera en un archivo aparte fuera del proyecto, Claude Code pediría permiso para leerlo cada vez. Si la cambias, cámbiala en las dos.

Para revisar que todo siga bien después de editar:

```bash
claude plugin validate ~/dev/claude-workflow
```

## Usarlo en varias máquinas (opcional)

1. Sube la carpeta a un repo privado de GitHub.
2. Registra el marketplace desde ese repo:

```bash
claude plugin marketplace add <tu-usuario>/claude-workflow
```

En este modo Claude Code instala una copia del plugin. Cada vez que cambies algo:

1. Sube `version` en `plugins/senior-junior/.claude-plugin/plugin.json`.
2. Actualiza:

```bash
claude plugin marketplace update eduardo-workflow
claude plugin update senior-junior@eduardo-workflow
```

## Límites

- Las sesiones en la nube (claude.ai/code) no cargan los plugins instalados en tu máquina.
- En el chat de claude.ai el plugin no aplica; ahí el flujo vive en las instrucciones de tu proyecto.