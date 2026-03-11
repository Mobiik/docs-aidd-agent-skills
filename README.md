# Mobiik Agent Skills

Skills compartidos para asistentes de IA, que encapsulan los flujos de trabajo,
estándares y buenas prácticas de Mobiik.

## Skills disponibles

| Skill | Descripción |
|---|---|
| **agent-blueprints** | Especificación de referencia con plantillas para el framework de 5 agentes + 12 skills + 3 reglas |

## Instalación

### Opción 1: Marketplace en VS Code (recomendado para equipos)

Agrega la fuente del marketplace en tu `settings.json` de VS Code:

```json
{
  "chat.plugins.enabled": true,
  "chat.plugins.marketplaces": ["git@github.com:Mobiik/docs-aidd-agent-skills.git"]
}
```

Luego abre la vista de Extensiones (`Ctrl+Shift+X`), busca `@agentPlugins` e
instala **mobiik-agent-skills**.

### Opción 2: GitHub Copilot CLI

```bash
copilot plugin marketplace add git@github.com:Mobiik/docs-aidd-agent-skills.git
copilot plugin install mobiik-agent-skills@mobiik-copilot-marketplace
```

O instala directamente desde el repositorio:

```bash
copilot plugin install git@github.com:Mobiik/docs-aidd-agent-skills.git
```

### Opción 3: Ruta local

Clona el repo y registra el plugin localmente:

```json
{
  "chat.plugins.paths": {
    "/ruta/a/docs-aidd-agent-skills": true
  }
}
```

## Uso

Los skills se cargan automáticamente según el contexto. Cada skill se activa
cuando el agente detecta palabras clave o tareas descritas en sus condiciones
de activación.

## Estructura del plugin

```
docs-aidd-agent-skills/
├── plugin.json                        # Manifiesto del plugin
├── .github/
│   └── plugin/
│       └── marketplace.json           # Definición del marketplace
├── skills/
│   └── agent-blueprints/
│       ├── SKILL.md                   # Skill: especificación del framework
│       └── references/
│           └── stacks.md              # Adaptaciones por stack tecnológico
├── CHANGELOG.md
└── LICENSE
```

## Compatibilidad

| Plataforma | Soporte |
|---|---|
| VS Code + GitHub Copilot (v1.110+) | Nativo vía sistema de Agent Plugins |
| GitHub Copilot CLI | `copilot plugin install` |
| Cursor | Lee la salida `.claude/` a nivel de proyecto |
| Claude Code | Formato `.claude/` nativo |
| Codex, Amp, OpenCode | Compatible con `.claude/` |

## Licencia

[MIT](LICENSE)
