# Claude Code Skills — Recursos y ruta de aprendizaje

## 1. Documentación oficial de Claude Code

**Claude Code — Skills: guía oficial**  
https://code.claude.com/docs/es/skills

Es el mejor punto de partida. Explica `SKILL.md`, dónde colocar los Skills, `description`, archivos auxiliares, invocación automática/manual, argumentos, subagents, troubleshooting, etc.

La idea básica es:

```text
.claude/
└── skills/
    └── mi-skill/
        ├── SKILL.md
        ├── examples/
        ├── scripts/
        └── templates/
```

Y el corazón es:

```markdown
---
description: Describe cuándo Claude debe usar este skill
---

# Mi Skill

Instrucciones detalladas para Claude...
```

---

## 2. Documentación oficial sobre cómo crear Skills

**Creating custom skills — Anthropic**  
https://claude.com/docs/skills/how-to

Esta es particularmente buena para entender **cómo diseñar un Skill bueno**, no solamente cómo crear el archivo.

Anthropic recomienda que un Skill:

- resuelva una tarea específica y repetible;
- tenga instrucciones claras;
- incluya ejemplos cuando sean útiles;
- defina cuándo debe utilizarse;
- se concentre en un workflow, en lugar de intentar hacer todo.

---

## 3. Agent Skills — el estándar detrás de Claude Code

**Agent Skills — Claude Platform Docs**  
https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview

Muy importante si querés aprender esto **en serio y de forma portable**.

Los Skills siguen el estándar **Agent Skills**, por lo que el concepto no está limitado exclusivamente a Claude Code.

La documentación explica la estructura y cómo funcionan en los distintos productos de Anthropic.

---

## 4. Plugins de Claude Code

**Create plugins — Claude Code Docs**  
https://code.claude.com/docs/en/plugins

Esto te conviene después de dominar Skills.

### La diferencia conceptual

#### Skill individual

```text
.claude/
└── skills/
    └── review-code/
        └── SKILL.md
```

→ Ideal para tus propios workflows o para un proyecto.

#### Plugin

```text
mi-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
├── agents/
├── hooks/
└── ...
```

→ Pensado para compartir, versionar y distribuir:

- Skills
- Agents
- Hooks
- MCP servers

---

## 5. Ejemplos reales en GitHub

### Claude Skill Template

**GitHub:**  
https://github.com/sidtheone/claude-skill-template

Este repo está bueno para estudiar Skills ya estructurados.

Tiene:

- templates;
- ejemplos;
- scripts;

y sigue el estándar **Agent Skills**.

### Claude Code Setup Guide — Skills

**GitHub:**  
https://github.com/Teng-AI/claude-code-setup-guide/blob/main/docs/03-skills.md

Tiene ejemplos de Skills orientados a workflows reales de desarrollo, como:

```text
/pre-implement
/test-gaps
/refactor
/debug
```

---

# 🚀 Mi recomendación para aprender

No me quedaría solamente leyendo documentación.

Haría este recorrido:

---

## Nivel 1 — Skill simple

Crear:

```text
~/.claude/skills/code-review/SKILL.md
```

Que haga algo como:

> Analizar el código modificado, encontrar bugs potenciales, problemas de seguridad y tests faltantes.

El objetivo en este nivel es entender:

- estructura de un Skill;
- `SKILL.md`;
- `description`;
- cuándo Claude decide utilizarlo;
- cómo invocarlo;
- cómo escribir instrucciones efectivas.

---

## Nivel 2 — Skill con contexto dinámico

Por ejemplo, que ejecute:

```bash
git diff
git status
```

Y utilice el resultado como contexto para realizar el análisis.

La idea es pasar de:

```text
"Instrucciones estáticas"
```

a:

```text
"Instrucciones + información actual del proyecto"
```

---

## Nivel 3 — Skill con archivos auxiliares

Por ejemplo:

```text
code-review/
├── SKILL.md
├── checklist.md
├── examples/
│   └── good-review.md
└── scripts/
    └── validate.sh
```

Esto permite separar:

- instrucciones principales;
- checklists;
- ejemplos;
- scripts;
- templates;
- documentación adicional.

El `SKILL.md` puede actuar como punto de entrada y cargar estos recursos cuando sean necesarios.

---

## Nivel 4 — Skills especializados

Una vez entendido el concepto, crear varios Skills especializados.

Por ejemplo:

```text
/feature
/bugfix
/code-review
/security-review
/write-tests
/refactor
/release
```

Cada Skill debería tener **un propósito claro**.

Por ejemplo:

```text
/code-review
```

se ocupa exclusivamente de revisar código.

Mientras que:

```text
/write-tests
```

se ocupa de analizar código existente y generar tests.

Esto evita crear un único Skill gigantesco que intente hacer absolutamente todo.

---

## Nivel 5 — Plugin

Cuando tengas varios Skills que quieras reutilizar entre proyectos o compartir con otras personas, pasar al concepto de **Plugin**.

Por ejemplo:

```text
my-development-plugin/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── code-review/
│   ├── security-review/
│   ├── write-tests/
│   └── refactor/
├── agents/
├── hooks/
└── ...
```

Ahí ya empezás a construir algo que puede convertirse en un conjunto de herramientas reutilizables para Claude Code.

---

# ⭐ Lo más importante que vas a aprender

Hay una distinción que vale mucho entender:

```text
CLAUDE.md
    ≠
Skill
    ≠
Agent
    ≠
Hook
    ≠
MCP
```

Claude Code tiene varias formas de extenderse y cada una tiene un propósito diferente.

### CLAUDE.md

Información e instrucciones generales que Claude debe conocer sobre un proyecto, repositorio o entorno.

### Skill

Agrega **conocimiento y workflows reutilizables** para realizar tareas específicas.

Ejemplos:

```text
/code-review
/write-tests
/refactor
/security-review
```

### Agent / Subagent

Permite delegar una tarea a un agente con contexto, instrucciones y herramientas propias.

Es útil para trabajos que conviene ejecutar de forma aislada o especializada.

### Hook

Permite reaccionar automáticamente ante determinados eventos de Claude Code.

Por ejemplo, ejecutar una acción cuando ocurre determinado evento dentro del workflow.

### MCP

Permite conectar Claude Code con **herramientas y servicios externos**.

Por ejemplo:

```text
Claude Code
    ↓
    MCP
    ↓
Base de datos / API / Servicio externo
```

### Plugin

Permite empaquetar y distribuir varias capacidades juntas:

```text
Plugin
├── Skills
├── Agents
├── Hooks
└── MCP
```

---

# 🧭 Ruta recomendada

En resumen, yo seguiría este orden:

```text
1. SKILL.md
      ↓
2. Crear un Skill simple
      ↓
3. Agregar contexto dinámico
      ↓
4. Agregar scripts / examples / templates
      ↓
5. Crear varios Skills especializados
      ↓
6. Aprender Agents / Subagents
      ↓
7. Aprender Hooks
      ↓
8. Aprender MCP
      ↓
9. Empaquetar todo como Plugin
```

El objetivo final sería poder pasar de:

```text
"Quiero que Claude haga X"
```

a:

```text
"Voy a construir un Skill que convierte X
en un workflow reproducible y reutilizable."
```