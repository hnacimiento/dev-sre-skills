---
name: como-usar-el-plugin
description: Guía para instalar y usar el plugin dev-sre-skills, pensada para alguien sin experiencia previa con plugins de Claude Code.
sources: cowork
---

# Cómo usar el plugin `dev-sre-skills`

Esta guía asume que no usaste plugins de Claude Code antes. No hace falta que
entiendas los detalles técnicos del sistema de plugins — con seguir los pasos
de acá alcanza.

## ¿Qué es esto?

`dev-sre-skills` es un paquete instalable que contiene los 9 skills de este
repositorio (`sre-engineering-mindset`, `sre-bash`, `sre-security`,
`sre-observability`, `sre-release-deployment`, `sre-incident-review`,
`sre-documentation`, `sre-testing`, `sre-slo`). Antes, para usarlos en un
proyecto había que copiar las carpetas a mano. Ahora se instalan una vez con
dos comandos y quedan disponibles donde vos decidas — en todos tus proyectos,
o solo en uno.

**Importante:** instalar el plugin no cambia en nada lo que los skills hacen
o cómo razonan. Siguen siendo exactamente los mismos archivos `SKILL.md` y
`references/` de siempre. Lo único que cambia es *cómo llegan* a un proyecto.

## Paso 1: Instalar Claude Code (si todavía no lo tenés)

Si ya usás Claude Code en tu máquina, saltá a el Paso 2.

Si no, seguí las instrucciones oficiales de instalación en
https://code.claude.com — el instalador varía según tu sistema operativo
(en Windows, que es lo que usás vos, hay un instalador nativo o se puede
usar `npm`).

## Paso 2: Agregar este repositorio como "marketplace"

Abrí una terminal donde tengas Claude Code (`claude`) y, dentro de una
sesión de Claude Code, corré:

```
/plugin marketplace add hnacimiento/dev-sre-skills
```

Esto no instala nada todavía — solo le dice a Claude Code "este repositorio
de GitHub tiene plugins disponibles". Es un paso que hacés **una sola vez**
por máquina (no una vez por proyecto).

## Paso 3: Instalar el plugin

Con el marketplace ya agregado, corré:

```
/plugin install dev-sre-skills@dev-sre-skills
```

Claude Code te va a preguntar en qué **alcance (scope)** instalarlo. Acá es
donde decidís si querés los skills en todos tus proyectos o en uno solo —
ver la sección siguiente.

## ¿Todos los proyectos, o uno en particular?

Cuando el instalador te pregunte el scope, tenés tres opciones. Para la
gran mayoría de los casos (y para lo que buscabas vos originalmente:
"reusable en todos los proyectos futuros"), la respuesta es la primera:

- **User scope ("para vos, en todos los proyectos")** — elegí esta opción.
  El plugin queda instalado a nivel de tu usuario en esa máquina, y va a
  estar disponible automáticamente en cualquier proyecto que abras con
  Claude Code de ahora en más, sin que tengas que hacer nada más. Es la
  opción equivalente a lo que antes lograbas copiando las carpetas a
  `~/.claude/skills/`.

- **Project scope ("para todos los que trabajen en este repositorio")** —
  elegí esta opción solo si estás en un proyecto compartido con otras
  personas y querés que el plugin quede declarado en la configuración del
  repositorio (`.claude/settings.json`), de forma que cualquiera que clone
  ese proyecto y confíe en la carpeta lo tenga disponible también. No aplica
  si sos el único que usa estos skills.

- **Local scope ("solo para vos, solo en este proyecto")** — elegí esta
  opción si querés probar el plugin en un proyecto puntual antes de
  decidir si lo querés en todos lados, sin afectar nada más.

Si en algún momento instalaste con "User scope" (todos los proyectos) y
después querés que un proyecto puntual *no* lo use, no hace falta
desinstalarlo globalmente: podés desactivarlo solo para ese proyecto con
`/plugin disable dev-sre-skills@dev-sre-skills` estando parado en ese
proyecto.

## Paso 4: Confirmar que está activo

Después de instalar, Claude Code te va a decir si el plugin ya está activo
("Plugin is now active") o si necesitás correr:

```
/reload-plugins
```

para activarlo sin reiniciar la terminal.

Para chequear en cualquier momento qué tenés instalado:

```
/plugin list
```

## Uso del día a día

No hay nada más que aprender para el uso normal: los skills siguen
disparándose solos, igual que antes, cuando Claude Code detecta que la
tarea que le pediste encaja con lo que un skill describe (por ejemplo, si
le pedís que revise un script de bash para producción, `sre-bash` y
`sre-engineering-mindset` se activan solos). No hace falta invocarlos por
nombre.

Si alguna vez quisieras forzar el uso de un skill puntual en vez de dejar
que Claude decida solo, el nombre pasa a tener el prefijo del plugin, por
ejemplo `/dev-sre-skills:sre-security` en vez de simplemente mencionarlo —
pero esto es opcional y no hace falta para el uso normal.

## Cuando actualicemos el contenido de los skills

Cada vez que se corrija o mejore algo en este repositorio (como los 4
ajustes de la ronda de validación empírica) y se suba a GitHub, para traer
esa actualización a tu instalación corrés, desde cualquier sesión de Claude
Code:

```
/plugin marketplace update dev-sre-skills
```

y si hace falta, `/reload-plugins` después. Esto reemplaza tener que ir
proyecto por proyecto reescribiendo archivos a mano.

## Si algo no funciona

- Si `/plugin` te dice que el comando no existe: actualizá Claude Code a la
  última versión (`npm install -g @anthropic-ai/claude-code@latest` si lo
  instalaste por npm, o el instalador nativo correspondiente) y reiniciá la
  terminal.
- Si instalaste el plugin pero no ves los skills disparándose: corré
  `/plugin list` para confirmar que aparece como instalado y habilitado, y
  si hace falta `/reload-plugins`.
- Si el problema persiste, como último recurso: `rm -rf ~/.claude/plugins/cache`
  (esto borra el caché local del sistema de plugins, no tu repositorio) y
  reinstalá.
