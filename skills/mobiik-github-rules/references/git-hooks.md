# Git Hooks for Mobiik Rules

Generate these files when a developer asks to set up git hooks or configure a new repository.

## .githooks/commit-msg

```bash
#!/usr/bin/env bash
# Validates commit message follows Conventional Commits format (Mobiik rules)

COMMIT_MSG_FILE="$1"
COMMIT_MSG=$(head -n 1 "$COMMIT_MSG_FILE")

# Allow merge commits
if echo "$COMMIT_MSG" | grep -qE "^Merge "; then
  exit 0
fi

PATTERN="^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?!?: [a-z].+"

if ! echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
  echo ""
  echo "============================================"
  echo "  ERROR: Formato de commit inválido"
  echo "============================================"
  echo ""
  echo "  Tu mensaje: $COMMIT_MSG"
  echo ""
  echo "  Formato requerido:"
  echo "    <tipo>(<scope>)!?: <descripción>"
  echo ""
  echo "  Tipos permitidos:"
  echo "    feat, fix, docs, style, refactor,"
  echo "    perf, test, build, ci, chore, revert"
  echo ""
  echo "  Ejemplos válidos:"
  echo "    feat: add user authentication"
  echo "    fix(auth): resolve token expiration"
  echo "    feat(api)!: change response format"
  echo ""
  echo "  Reglas:"
  echo "    - La descripción debe iniciar con minúscula"
  echo "    - Dos puntos y espacio después del tipo"
  echo "    - El scope es opcional"
  echo ""
  echo "============================================"
  exit 1
fi
```

## .githooks/pre-commit

```bash
#!/usr/bin/env bash
# Validates author email, blocked secret files, and secret content patterns

HAS_ERRORS=false

# ── 1. Validate author email ──

AUTHOR_EMAIL=$(git config user.email)

if ! echo "$AUTHOR_EMAIL" | grep -qE "@mobiik\.com$"; then
  echo ""
  echo "============================================"
  echo "  ERROR: Email de autor no válido"
  echo "============================================"
  echo ""
  echo "  Email actual: $AUTHOR_EMAIL"
  echo "  Email requerido: *@mobiik.com"
  echo ""
  echo "  Para corregirlo ejecuta:"
  echo "    git config user.email \"tu.nombre@mobiik.com\""
  echo ""
  echo "============================================"
  HAS_ERRORS=true
fi

# ── 2. Secrets scanning (bypass with SKIP_SECRETS_SCAN=1) ──

if [ "${SKIP_SECRETS_SCAN:-0}" = "1" ]; then
  if [ "$HAS_ERRORS" = true ]; then
    exit 1
  fi
  exit 0
fi

STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)

if [ -z "$STAGED_FILES" ]; then
  if [ "$HAS_ERRORS" = true ]; then
    exit 1
  fi
  exit 0
fi

# ── 2a. Blocked file patterns ──

BLOCKED_FILES=""

while IFS= read -r file; do
  [ -z "$file" ] && continue
  basename=$(basename "$file")
  lowercase=$(echo "$basename" | tr '[:upper:]' '[:lower:]')

  # Whitelist: .example, .sample, .template, .dist suffixes
  if echo "$lowercase" | grep -qE '\.(example|sample|template|dist)(\.|$)'; then
    continue
  fi

  # Block: environment variable files
  if echo "$lowercase" | grep -qE '^\.env($|\..+)'; then
    BLOCKED_FILES="${BLOCKED_FILES}\n    - $file"
    continue
  fi

  # Block: credential and key files
  if echo "$lowercase" | grep -qE '^(credentials\.json|serviceaccountkey\.json|gcloud-service-key\.json|token\.json|\.npmrc|\.pypirc|id_rsa|id_ed25519|id_ecdsa)$'; then
    BLOCKED_FILES="${BLOCKED_FILES}\n    - $file"
    continue
  fi

  # Block: firebase admin SDK
  if echo "$lowercase" | grep -qE '^firebase-adminsdk.*\.json$'; then
    BLOCKED_FILES="${BLOCKED_FILES}\n    - $file"
    continue
  fi

  # Block: certificate and key extensions
  if echo "$lowercase" | grep -qE '\.(pem|key|p12|pfx|jks|secret)$'; then
    BLOCKED_FILES="${BLOCKED_FILES}\n    - $file"
    continue
  fi
done <<< "$STAGED_FILES"

if [ -n "$BLOCKED_FILES" ]; then
  echo ""
  echo "============================================"
  echo "  ERROR: Archivos con secretos detectados"
  echo "============================================"
  echo ""
  echo "  Los siguientes archivos están bloqueados:"
  printf '%b\n' "$BLOCKED_FILES"
  echo ""
  echo "  Solución:"
  echo "    git reset HEAD <archivo>"
  echo "    echo '<archivo>' >> .gitignore"
  echo ""
  echo "  Si es un archivo legítimo (mock/test):"
  echo "    SKIP_SECRETS_SCAN=1 git commit -m \"reason\""
  echo ""
  echo "  Archivos permitidos:"
  echo "    .env.example, .env.sample, .env.template"
  echo ""
  echo "============================================"
  HAS_ERRORS=true
fi

# ── 2b. Content scanning for secret patterns ──

SECRET_LEAKS=""

while IFS= read -r file; do
  [ -z "$file" ] && continue

  # Skip binary files and files > 1MB
  if git diff --cached --numstat "$file" | grep -q "^-"; then
    continue
  fi
  file_size=$(git cat-file -s ":$file" 2>/dev/null || echo "0")
  if [ "$file_size" -gt 1048576 ]; then
    continue
  fi

  # Skip lock files and common non-code files
  basename=$(basename "$file")
  if echo "$basename" | grep -qE '\.(lock|lockb|min\.js|min\.css)$'; then
    continue
  fi

  content=$(git show ":$file" 2>/dev/null || true)
  [ -z "$content" ] && continue

  # Private keys
  if echo "$content" | grep -qE -- "-----BEGIN (RSA|EC|DSA|OPENSSH|PGP) PRIVATE KEY-----"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → clave privada detectada"
    continue
  fi

  # AWS Access Key ID
  if echo "$content" | grep -qE "AKIA[0-9A-Z]{16}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → AWS Access Key detectada"
    continue
  fi

  # GitHub tokens
  if echo "$content" | grep -qE "gh[pos]_[A-Za-z0-9]{36,}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → GitHub token detectado"
    continue
  fi

  # GitLab tokens
  if echo "$content" | grep -qE "glpat-[A-Za-z0-9\-]{20,}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → GitLab token detectado"
    continue
  fi

  # OpenAI / Stripe secret keys
  if echo "$content" | grep -qE "sk-[A-Za-z0-9]{20,}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → API secret key detectada (sk-*)"
    continue
  fi

  # Slack tokens
  if echo "$content" | grep -qE "xox[bpors]-[A-Za-z0-9\-]{10,}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → Slack token detectado"
    continue
  fi

  # SendGrid API keys
  if echo "$content" | grep -qE "SG\.[A-Za-z0-9\-]{22}\.[A-Za-z0-9\-]{43}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → SendGrid API key detectada"
    continue
  fi

  # Google API keys
  if echo "$content" | grep -qE "AIza[A-Za-z0-9\-_]{35}"; then
    SECRET_LEAKS="${SECRET_LEAKS}\n    - $file → Google API key detectada"
    continue
  fi
done <<< "$STAGED_FILES"

if [ -n "$SECRET_LEAKS" ]; then
  echo ""
  echo "============================================"
  echo "  ERROR: Secretos detectados en contenido"
  echo "============================================"
  echo ""
  echo "  Se encontraron patrones de secretos en:"
  printf '%b\n' "$SECRET_LEAKS"
  echo ""
  echo "  Solución:"
  echo "    - Elimina el secreto del archivo"
  echo "    - Usa variables de entorno o un vault"
  echo "    - Usa placeholders: <TOKEN>, changeme"
  echo ""
  echo "  Si es un falso positivo (mock/test):"
  echo "    SKIP_SECRETS_SCAN=1 git commit -m \"reason\""
  echo ""
  echo "============================================"
  HAS_ERRORS=true
fi

if [ "$HAS_ERRORS" = true ]; then
  exit 1
fi
```

## .githooks/pre-push

```bash
#!/usr/bin/env bash
# Validates branch name follows Mobiik naming convention

BRANCH=$(git symbolic-ref --short HEAD 2>/dev/null)

if [ -z "$BRANCH" ]; then
  # Detached HEAD, skip validation
  exit 0
fi

PATTERN="^(main|develop|qa|feature\/.+|bugfix\/.+|hotfix\/.+|release\/v[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9.]+)?|support\/.+|dependabot\/.+|renovate\/.+)$"

if ! echo "$BRANCH" | grep -qE "$PATTERN"; then
  echo ""
  echo "============================================"
  echo "  ERROR: Nombre de rama no válido"
  echo "============================================"
  echo ""
  echo "  Rama actual: $BRANCH"
  echo ""
  echo "  Patrones permitidos:"
  echo "    main, develop, qa"
  echo "    feature/<nombre>"
  echo "    bugfix/<nombre>"
  echo "    hotfix/<nombre>"
  echo "    release/v<X.Y.Z>"
  echo "    support/<nombre>"
  echo ""
  echo "  Ejemplos válidos:"
  echo "    feature/user-authentication"
  echo "    bugfix/issue-123"
  echo "    hotfix/security-patch"
  echo "    release/v1.2.3"
  echo ""
  echo "  Para renombrar tu rama:"
  echo "    git branch -m \"$BRANCH\" feature/nombre-correcto"
  echo ""
  echo "============================================"
  exit 1
fi
```

## setup-hooks.sh

```bash
#!/usr/bin/env bash
# Setup script for Mobiik git hooks
# Configures hooksPath, makes hooks executable, validates email

set -e

HOOKS_DIR=".githooks"

echo ""
echo "============================================"
echo "  Configurando hooks de Mobiik"
echo "============================================"
echo ""

# Check we are in a git repo
if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
  echo "  ERROR: No estás dentro de un repositorio git."
  exit 1
fi

# Check hooks directory exists
if [ ! -d "$HOOKS_DIR" ]; then
  echo "  ERROR: No se encontró el directorio $HOOKS_DIR"
  echo "  Asegúrate de que los hooks estén en el repositorio."
  exit 1
fi

# Make hooks executable
chmod +x "$HOOKS_DIR"/*
echo "  [OK] Hooks marcados como ejecutables"

# Configure git to use custom hooks directory
git config core.hooksPath "$HOOKS_DIR"
echo "  [OK] hooksPath configurado a $HOOKS_DIR"

# Validate email
CURRENT_EMAIL=$(git config user.email 2>/dev/null || echo "")

if echo "$CURRENT_EMAIL" | grep -qE "@mobiik\.com$"; then
  echo "  [OK] Email configurado: $CURRENT_EMAIL"
else
  echo ""
  echo "  ADVERTENCIA: Tu email no es corporativo."
  echo "  Email actual: ${CURRENT_EMAIL:-'(no configurado)'}"
  echo ""
  read -p "  Ingresa tu email @mobiik.com: " NEW_EMAIL
  if echo "$NEW_EMAIL" | grep -qE "@mobiik\.com$"; then
    git config user.email "$NEW_EMAIL"
    echo "  [OK] Email configurado: $NEW_EMAIL"
  else
    echo "  ERROR: El email debe terminar en @mobiik.com"
    exit 1
  fi
fi

echo ""
echo "  Configuración completada exitosamente."
echo "============================================"
echo ""
```
