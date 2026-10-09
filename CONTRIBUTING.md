# Contribuir a pmmp-2-agents-kit

¡Gracias por tu interés en mejorar las reglas! Este documento explica cómo participar.

## El proyecto es abierto

- Mantenedor oficial: [mateocollar](https://github.com/mateocollar).
- Aunque hay un mantenedor, el proyecto es **abierto**: issues, discusiones y pull requests de la comunidad son bienvenidos.
- Empezá mirando los issues abiertos; si encontrás una regla incorrecta o que falta, abrí uno con la plantilla correspondiente.

## Cómo reportar algo

- Bugs o comportamientos inesperados del repo → plantilla [Reporte de bug](.github/ISSUE_TEMPLATE/bug_report.md).
- Reglas nuevas o cambios de contenido → plantilla [Solicitud de función](.github/ISSUE_TEMPLATE/feature_request.md).
- Dudas o colaboración directa → los `contact_links` del repositorio apuntan al mantenedor.

## Flujo de trabajo con Git (obligatorio)

1. **Hacé un fork** del repositorio y clonalo localmente.
2. **Creá una rama descriptiva** desde `main`, usando uno de los prefijos de la tabla de abajo (por ejemplo, `docs/regla-php71` o `fix/typo-readme`).
3. **Commiteá con Conventional Commits**: `feat:`, `fix:`, `docs:`, `chore:`. En inglés o español, pero consistente en toda la rama, y con cuerpo explicando el **porqué** del cambio.
4. **Abrí un Pull Request a `main`** describiendo QUÉ cambia y POR QUÉ, completando el checklist de la plantilla. Nada de cambios fuera de scope.
5. **Mantené sincronizados `en/` y `es/`**: todo cambio de contenido en `es/AGENTS.md` aplica también a `en/AGENTS.md`, y viceversa.

### Prefijos de rama

| Prefijo | Cuándo usarlo | Ejemplo |
| --- | --- | --- |
| `docs/` | Documentación, AGENTS.md, README | `docs/regla-php71` |
| `fix/` | Corrección de erratas o reglas equivocadas | `fix/typo-readme` |
| `feat/` | Reglas o archivos nuevos | `feat/advertencia-nbt` |
| `chore/` | CI, plantillas de `.github/`, tooling | `chore/actualizar-linter` |

### Tipos de commit

| Tipo | Cuándo usarlo |
| --- | --- |
| `feat:` | Agrega una regla o un archivo nuevo. |
| `fix:` | Corrige una regla incorrecta o una errata. |
| `docs:` | Mejora la redacción sin cambiar el significado. |
| `chore:` | Cambios de infraestructura (CI, plantillas). |

## Regla de reescrituras

Se aceptan **ediciones de contenido** (corregir, ampliar, matizar una regla), pero **no reescrituras completas** de los `AGENTS.md` sin discutirlas antes en un issue.

## Calidad del contenido

- Markdown válido: headings jerárquicos, tablas bien formadas y sin placeholders. `[VERIFICAR]` es la única marca permitida.
- Todo snippet PHP debe ser compilable en PHP 7.0.14: revisá la sección de restricciones de [es/AGENTS.md](es/AGENTS.md) antes de commitear.
- El CI (`lint-markdown.yml`) corre markdownlint en cada push y PR a `main`; debe pasar antes del merge.

## Licencia

Al contribuir, aceptás que tu aporte se distribuya bajo [Apache 2.0](LICENSE).
