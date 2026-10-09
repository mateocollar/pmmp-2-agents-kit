# pmmp-2-agents-kit

![Logo de pmmp-2-agents-kit](https://github.com/mateocollar/pmmp-2-agents-kit/blob/main/assets/logo.png)

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![markdownlint](https://github.com/mateocollar/pmmp-2-agents-kit/actions/workflows/lint-markdown.yml/badge.svg)](https://github.com/mateocollar/pmmp-2-agents-kit/actions/workflows/lint-markdown.yml)

Conjunto de reglas `AGENTS.md` ("constitution files") para que agentes de IA escriban plugins de PocketMine-MP 2.0.0 (MCPE 0.15.10, protocolo 84, PHP 7.0.14) sin romper compatibilidad.

## ¿Por qué?

Los modelos de lenguaje están entrenados sobre código moderno: PocketMine-MP 3/4/5 y PHP 7.1-8.x. Cuando vibecodeás un plugin para este stack de 2016, el agente tiende a generar código que **ni siquiera compila** en el servidor real:

- `void`, tipos anulables `?int`, `match`, arrow functions `fn()`, `??=` (PHP 7.1+ / 8.x).
- Namespaces modernos como `pocketmine\player\Player` o `pocketmine\world\World` (PM 3-5).
- IDs de items en string (`"minecraft:stone"`), que en MCPE 0.15.10 todavía no existen.

Estas reglas se inyectan en tu agente y **prevalecen sobre su conocimiento entrenado**: definen el stack exacto (PM 2.0.0 / PHP 7.0.14), tablas PROHIBIDO/PERMITIDO con sus equivalencias, mapeo PM5 → PM2 y un bloque SELF-CHECK que el agente debe auto-verificar antes de entregar código.

## Contenido

| Ruta | Idioma | Descripción |
| --- | --- | --- |
| [es/AGENTS.md](es/AGENTS.md) | Español | Constitución maestra. |
| [en/AGENTS.md](en/AGENTS.md) | Inglés | Traducción espejo de la versión maestra. |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Español | Cómo contribuir al proyecto. |
| [LICENSE](LICENSE) | — | Licencia Apache 2.0. |

## Cómo usarlo

Elegí la versión según tu idioma (`es/AGENTS.md` o `en/AGENTS.md`) y apuntala a tu agente. Tres formas habituales:

### Herramientas que leen `AGENTS.md` en la raíz

Si tu herramienta busca `AGENTS.md` automáticamente en la raíz del repositorio, copiá la versión que uses como `AGENTS.md` dentro de tu proyecto de plugins.

### Claude Code (`CLAUDE.md`)

Creá un `CLAUDE.md` en la raíz de tu plugin:

```markdown
# CLAUDE.md

Aplicá las reglas de @es/AGENTS.md antes de escribir cualquier código PHP.
```

### Cursor (`.cursor/rules/`)

Creá `.cursor/rules/pmmp-2-legacy.mdc` con este frontmatter y pegá debajo el contenido completo de `es/AGENTS.md`:

```markdown
---
description: Reglas obligatorias para PocketMine-MP 2.0.0 / PHP 7.0
alwaysApply: true
---
```

### GitHub Copilot

Creá `.github/copilot-instructions.md` con el contenido de `es/AGENTS.md`: Copilot lo tiene en cuenta en los chats del repositorio.

### Otras herramientas

Muchos CLIs aceptan lectura explícita de archivos, por ejemplo: `aider --read es/AGENTS.md`.

## Roadmap

- **v2**: presets de reglas (por ejemplo, estricto / relajado) y una referencia de la API de PocketMine-MP 2.0.0 en un repositorio aparte.

## Contribuir

Las contribuciones de la comunidad son bienvenidas: leé [CONTRIBUTING.md](CONTRIBUTING.md) antes de abrir un PR.

## Licencia

Distribuido bajo [Apache 2.0](LICENSE).
