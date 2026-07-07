# Architecture — `{{REPO_NAME}}`

> **Este es un esqueleto** generado automáticamente por `Mobiik/.github/scripts/new-repo-init.yml` cuando se inicializó este repo. El owner debe llenar cada sección con la realidad de su sistema.
>
> Para arquitectura del Quality Gates org-wide (NO de este repo), ver [`Mobiik/.github/docs/architecture.md`](https://github.com/Mobiik/.github/blob/main/docs/architecture.md).

---

## 1. Visión

**¿Qué problema resuelve este sistema?** (1-2 párrafos)

**¿Para qué usuario / equipo?**

**¿Qué lo diferencia de alternativas?**

---

## 2. Stack tecnológico

| Capa | Tecnología | Versión |
|---|---|---|
| Lenguaje principal | _(node / python / go / java / rust / ...)_ | _(20 / 3.12 / ...)_ |
| Framework | _(Express / FastAPI / Spring / Gin / ...)_ | |
| Base de datos | _(Postgres / Mongo / Redis / ...)_ | |
| Cache | | |
| Cola de mensajes | | |
| Container runtime | _(Docker / Podman / Containerd)_ | |
| Orquestador | _(K8s / ECS / Cloud Run / ...)_ | |
| Cloud provider | _(Azure / AWS / GCP / on-prem)_ | |

---

## 3. Diagrama de componentes

```
[Reemplazar con diagrama ASCII o link a Mermaid/draw.io]

    ┌──────────┐       ┌──────────┐       ┌──────────┐
    │ Cliente  │──────▶│ API/Edge │──────▶│ Servicio │
    └──────────┘       └──────────┘       └──────────┘
                                                │
                                                ▼
                                        ┌──────────┐
                                        │  DB / DS │
                                        └──────────┘
```

---

## 4. Componentes principales

| Componente | Responsabilidad | Tecnología | Owner |
|---|---|---|---|

---

## 5. Flujos de datos clave

### Flujo 1: _(nombre del caso de uso)_

1. Trigger → ...
2. Procesamiento → ...
3. Persistencia → ...
4. Salida → ...

### Flujo 2: ...

---

## 6. Integraciones externas

| Sistema | Propósito | Protocolo | Auth |
|---|---|---|---|

---

## 7. Almacenamiento y persistencia

- **¿Qué datos persiste el sistema?**
- **¿Dónde?** (DB, blob storage, cache, etc.)
- **Backup / retention policy:**
- **Encryption at rest / in transit:**

---

## 8. Decisiones de diseño (ADRs)

| ADR | Decisión | Status | Trade-off |
|---|---|---|---|

> Recomendado: documentar decisiones arquitectónicas en `docs/adr/0001-...` siguiendo [MADR](https://adr.github.io/madr/).

---

## 9. Quality Gates aplicado a este repo

Este repo está protegido por el sistema **Quality Gates org-wide** de Mobiik.

### Stages base (siempre activos)

1. ✅ **pre-flight** — Conventional Commits, file size <5MB, lang detect
2. ✅ **secrets-scan** — Gitleaks + TruffleHog (verified)
3. ✅ **lint-format** — Auto-detect (ESLint/Ruff/golangci/Checkstyle/Clippy)
4. ✅ **sast** — Semgrep + CodeQL (security-extended)
5. ✅ **sca** — Trivy fs + Dependabot HIGH+ alerts API
6. ✅ **tests-coverage** — ≥80% configurable
7. ⚠️ **sonarqube-gate** — corre si existe `sonar-project.properties`
8. ✅ **build** — Compila + Docker + Trivy image + SBOM
9. ✅ **quality-gate-result** — Aggregator final (if: always())

### Stages auto-activos según contenido del repo

| Stage | Se activa si | Cubre |
|---|---|---|
| 🔵 **Checkov** (4.5) | hay `*.tf`, `Dockerfile`, `Chart.yaml`, K8s manifests, `*.bicep` | IaC compliance (CIS, NIST) |
| 🔵 **Hadolint** (4.6) | hay `Dockerfile` | Dockerfile lint (best practices) |
| 🔵 **OSV-Scanner** (4.7) | siempre | Vulns complementarias a Trivy |
| 🔵 **kube-linter** (4.8) | hay K8s manifests / Helm | K8s reliability + security |

### Dependabot (18 ecosystems)

Cubre todo: npm, pip, nuget, maven, gradle, gomod, cargo, pub (Flutter), composer (PHP), bundler (Ruby), swift, mix (Elixir), docker, docker-compose, terraform, helm, devcontainers, github-actions.

### Renovate Bot (complementario)

Si tu repo necesita cobertura para Bash scripts, PowerShell, asdf `.tool-versions`, GitHub releases binarios → Renovate ya está configurado vía `renovate.json` (canonical, apunta al preset compartido `Mobiik/.github/configs/renovate.config.json`).

### Weekly Security Digest

Cada lunes 09:00 UTC: si tienes alertas Code Scanning / Secret Scanning / Dependabot HIGH+CRITICAL introducidas por tus commits (detectado vía `git blame`), recibirás un issue assigned a ti en este mismo repo con el resumen y SLA.

### Cómo deshabilitar un stage en tu repo

No es posible (eres del lado consumidor del Required Workflow). Para excepciones documentadas, abrir issue en `Mobiik/.github` solicitando review de `@Mobiik/platform-engineering`.

Referencia completa: [`Mobiik/.github/docs/architecture.md`](https://github.com/Mobiik/.github/blob/main/docs/architecture.md).

---

## 10. Cómo monitorear este sistema

- **Logs:** _(ubicación, formato, retention)_
- **Métricas:** _(Datadog / Grafana / Prometheus / CloudWatch / etc.)_
- **Alertas:** _(canal, escalación, on-call)_
- **Health checks:** _(endpoint, frecuencia)_

---

## 11. Cómo correr local

```bash
# Pre-requisitos
# ...

# Instalación
# ...

# Levantar
# ...

# Smoke test
# ...
```

Ver [`README.md`](README.md) para guía completa.
