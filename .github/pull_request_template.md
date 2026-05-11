<!--
  Template de PR Mobiik (auto-aplicado org-wide desde Mobiik/.github).
  Llena cada sección. Si una no aplica, escribe "N/A" con justificación corta.
  Antes de submit, verifica que el título sigue Conventional Commits:
    feat(scope): descripción   |   fix(auth): ...   |   docs: ...
-->

## Objetivo

<!-- ¿Qué problema resuelve este PR? Una o dos frases. Sin "lo que hace" — eso va en Cambios. -->

## Cambios

<!-- Lista corta de los cambios principales (3-7 bullets). Lo importante, no exhaustivo. -->

-
-
-

## Tipo de cambio

<!-- Marca con `x` lo que aplica -->

- [ ] 🐛 Bug fix (no rompe nada)
- [ ] ✨ Nueva feature (no rompe nada)
- [ ] 💥 Breaking change (rompe API o comportamiento existente)
- [ ] 📝 Documentación
- [ ] 🔧 Refactor / chore (sin cambio funcional)
- [ ] ⚡ Performance
- [ ] 🧪 Tests
- [ ] 🔒 Seguridad

## Testing

<!-- ¿Cómo lo probaste? Comandos, escenarios, capturas si aplica. -->

-
-

## Checklist

- [ ] Mis commits siguen [Conventional Commits](https://www.conventionalcommits.org/) y mi email es `@mobiik.com`.
- [ ] Mi branch sigue el [naming convention](https://github.com/Mobiik/.github-private/blob/main/docs/standards/README.md#7-nombres-de-ramas-branch-naming).
- [ ] Agregué/actualicé tests para cubrir los cambios.
- [ ] Coverage local >= 80% (o el threshold del repo).
- [ ] Actualicé documentación relevante (README, comentarios, ADR si decisión estructural).
- [ ] No introduje nuevas vulnerabilidades (SAST + SCA verán esto en el pipeline).
- [ ] No hay secrets hardcodeados (pre-commit hook + Quality Gates lo verifican).
- [ ] Si es **breaking change**: agregué `!` al commit type y describo el impacto y plan de migración abajo.

## Breaking changes (si aplica)

<!-- Solo si marcaste 💥 arriba. Describe:
  - Qué deja de funcionar.
  - Qué necesitan hacer los consumidores para adaptarse.
  - Plan de comunicación / deprecación.
-->

## Plan de rollback

<!-- Si esto va a producción: cómo revertir si algo sale mal.
  Ver runbook: https://github.com/Mobiik/.github-private/blob/main/docs/runbooks/devops/deploy-rollback.md
-->

## Issues relacionados

<!-- Linkea con `Closes #123` o `Refs #456` -->

Closes #
Refs #

## Notas adicionales para reviewers

<!-- Áreas específicas donde quieres focus de review. Decisiones controvertidas. Trade-offs. -->
