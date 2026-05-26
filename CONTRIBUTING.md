# Contributing

> ⚠️ Esta guía de contribución es **org-wide** para Mobiik.
> La fuente canónica vive en [`Mobiik/.github/CONTRIBUTING.md`](https://github.com/Mobiik/.github/blob/main/CONTRIBUTING.md).
> Este archivo es auto-mantenido por el sistema **Quality Gates self-heal** (lunes 06:00 UTC).
> NO editar este archivo directamente — las modificaciones serán sobrescritas.

## Quick reference

- **Conventional Commits:** `feat:`, `fix:`, `docs:`, `chore:`, etc.
- **Branch naming:** `feature/...`, `bugfix/...`, `hotfix/...`, `release/vX.Y.Z`
- **Email:** `@mobiik.com` en author/committer (regla del ruleset)
- **PRs:** mínimo 1 aprobación + Quality Gate verde
- **Merge preferido:** Squash

## Antes de abrir tu primer PR

```bash
# Configurar email corporativo
git config --global user.email "tu.nombre@mobiik.com"

# Crear branch siguiendo naming convention
git checkout -b feature/<descripcion-corta>

# Conventional Commits format
git commit -m "feat(scope): descripción"
```

## Quality Gates pipeline

Tu PR pasa por **8 stages bloqueantes**:

1. pre-flight (Conventional Commits, file size, archivos prohibidos)
2. secrets-scan (Gitleaks + TruffleHog)
3. lint-format (por lenguaje detectado)
4. sast (Semgrep + CodeQL)
5. sca (Trivy + Dependabot alerts)
6. tests-coverage (≥80% por defecto)
7. sonarqube-gate (skip si no aplica)
8. build (compila + Docker + Trivy + SBOM)

Si algún stage te bloquea, lee [`docs/developer-guide.md`](https://github.com/Mobiik/.github/blob/main/docs/developer-guide.md).

## Política completa

Para flujo detallado, code review, troubleshooting de cada stage, y FAQs:

**📜 [Mobiik/.github/CONTRIBUTING.md](https://github.com/Mobiik/.github/blob/main/CONTRIBUTING.md)**

## Soporte

- 💬 Slack: `#platform-support`
- 📚 Docs internos: [`Mobiik/.github-private`](https://github.com/Mobiik/.github-private)
