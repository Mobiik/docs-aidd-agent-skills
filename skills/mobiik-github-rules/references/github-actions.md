# GitHub Actions Workflow for Mobiik Rules

Generate this file as `.github/workflows/mobiik-rules.yml` when asked.

```yaml
name: Mobiik Rules Validation

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  pull-requests: write
  contents: read

jobs:
  validate:
    name: Validar reglas de Mobiik
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Validar mensajes de commit
        id: commits
        run: |
          PATTERN="^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?!?: [a-z].+"
          MERGE_PATTERN="^Merge "
          ERRORS=""
          PASS=true

          COMMITS=$(git log --format="%H %s" origin/${{ github.base_ref }}..HEAD)

          while IFS= read -r line; do
            [ -z "$line" ] && continue
            SHA="${line%% *}"
            MSG="${line#* }"

            if echo "$MSG" | grep -qE "$MERGE_PATTERN"; then
              continue
            fi

            if ! echo "$MSG" | grep -qE "$PATTERN"; then
              ERRORS="${ERRORS}\n- \`${SHA:0:7}\`: \`${MSG}\`"
              PASS=false
            fi
          done <<< "$COMMITS"

          if [ "$PASS" = false ]; then
            echo "status=fail" >> $GITHUB_OUTPUT
            # Use a temp file to preserve newlines
            printf '%b\n' "$ERRORS" > /tmp/commit_errors.txt
          else
            echo "status=pass" >> $GITHUB_OUTPUT
          fi

      - name: Validar emails de autor
        id: emails
        run: |
          ERRORS=""
          PASS=true

          COMMITS=$(git log --format="%H %ae" origin/${{ github.base_ref }}..HEAD)

          while IFS= read -r line; do
            [ -z "$line" ] && continue
            SHA="${line%% *}"
            EMAIL="${line#* }"

            if ! echo "$EMAIL" | grep -qE "@mobiik\.com$"; then
              ERRORS="${ERRORS}\n- \`${SHA:0:7}\`: \`${EMAIL}\`"
              PASS=false
            fi
          done <<< "$COMMITS"

          if [ "$PASS" = false ]; then
            echo "status=fail" >> $GITHUB_OUTPUT
            printf '%b\n' "$ERRORS" > /tmp/email_errors.txt
          else
            echo "status=pass" >> $GITHUB_OUTPUT
          fi

      - name: Validar nombre de rama
        id: branch
        run: |
          BRANCH="${{ github.head_ref }}"
          PATTERN="^(main|develop|qa|feature\/.+|bugfix\/.+|hotfix\/.+|release\/v[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9.]+)?|support\/.+|dependabot\/.+|renovate\/.+)$"

          if echo "$BRANCH" | grep -qE "$PATTERN"; then
            echo "status=pass" >> $GITHUB_OUTPUT
          else
            echo "status=fail" >> $GITHUB_OUTPUT
            echo "branch=$BRANCH" >> $GITHUB_OUTPUT
          fi

      - name: Escanear archivos con secretos
        id: secrets
        run: |
          CHANGED_FILES=$(git diff --name-only --diff-filter=ACM origin/${{ github.base_ref }}..HEAD)
          BLOCKED=""
          LEAKS=""
          PASS=true

          while IFS= read -r file; do
            [ -z "$file" ] && continue
            [ ! -f "$file" ] && continue
            basename=$(basename "$file")
            lowercase=$(echo "$basename" | tr '[:upper:]' '[:lower:]')

            # Whitelist: .example, .sample, .template, .dist
            if echo "$lowercase" | grep -qE '\.(example|sample|template|dist)(\.|$)'; then
              continue
            fi

            # Blocked file patterns
            is_blocked=false
            if echo "$lowercase" | grep -qE '^\.env($|\..+)'; then is_blocked=true; fi
            if echo "$lowercase" | grep -qE '^(credentials\.json|serviceaccountkey\.json|gcloud-service-key\.json|token\.json|\.npmrc|\.pypirc|id_rsa|id_ed25519|id_ecdsa)$'; then is_blocked=true; fi
            if echo "$lowercase" | grep -qE '^firebase-adminsdk.*\.json$'; then is_blocked=true; fi
            if echo "$lowercase" | grep -qE '\.(pem|key|p12|pfx|jks|secret)$'; then is_blocked=true; fi

            if [ "$is_blocked" = true ]; then
              BLOCKED="${BLOCKED}\n- \`${file}\`"
              PASS=false
              continue
            fi

            # Content scanning (skip files > 1MB and lock files)
            if echo "$lowercase" | grep -qE '\.(lock|lockb|min\.js|min\.css)$'; then
              continue
            fi
            file_size=$(wc -c < "$file" 2>/dev/null || echo "0")
            if [ "$file_size" -gt 1048576 ]; then
              continue
            fi

            # Scan for secret patterns
            leak_type=""
            if grep -qE -- "-----BEGIN (RSA|EC|DSA|OPENSSH|PGP) PRIVATE KEY-----" "$file" 2>/dev/null; then leak_type="clave privada"; fi
            if [ -z "$leak_type" ] && grep -qE "AKIA[0-9A-Z]{16}" "$file" 2>/dev/null; then leak_type="AWS Access Key"; fi
            if [ -z "$leak_type" ] && grep -qE "gh[pos]_[A-Za-z0-9]{36,}" "$file" 2>/dev/null; then leak_type="GitHub token"; fi
            if [ -z "$leak_type" ] && grep -qE "glpat-[A-Za-z0-9\-]{20,}" "$file" 2>/dev/null; then leak_type="GitLab token"; fi
            if [ -z "$leak_type" ] && grep -qE "sk-[A-Za-z0-9]{20,}" "$file" 2>/dev/null; then leak_type="API secret key (sk-*)"; fi
            if [ -z "$leak_type" ] && grep -qE "xox[bpors]-[A-Za-z0-9\-]{10,}" "$file" 2>/dev/null; then leak_type="Slack token"; fi
            if [ -z "$leak_type" ] && grep -qE "SG\.[A-Za-z0-9\-]{22}\.[A-Za-z0-9\-]{43}" "$file" 2>/dev/null; then leak_type="SendGrid API key"; fi
            if [ -z "$leak_type" ] && grep -qE "AIza[A-Za-z0-9\-_]{35}" "$file" 2>/dev/null; then leak_type="Google API key"; fi

            if [ -n "$leak_type" ]; then
              LEAKS="${LEAKS}\n- \`${file}\` → ${leak_type}"
              PASS=false
            fi
          done <<< "$CHANGED_FILES"

          if [ "$PASS" = false ]; then
            echo "status=fail" >> $GITHUB_OUTPUT
            [ -n "$BLOCKED" ] && printf '%b\n' "$BLOCKED" > /tmp/blocked_files.txt
            [ -n "$LEAKS" ] && printf '%b\n' "$LEAKS" > /tmp/secret_leaks.txt
          else
            echo "status=pass" >> $GITHUB_OUTPUT
          fi

      - name: Publicar resumen
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');

            const commitStatus = '${{ steps.commits.outputs.status }}';
            const emailStatus = '${{ steps.emails.outputs.status }}';
            const branchStatus = '${{ steps.branch.outputs.status }}';
            const secretsStatus = '${{ steps.secrets.outputs.status }}';
            const branchName = '${{ steps.branch.outputs.branch }}';

            const icon = (s) => s === 'pass' ? ':white_check_mark:' : ':x:';

            let body = `## Validación de Reglas Mobiik\n\n`;
            body += `| Regla | Estado |\n|-------|--------|\n`;
            body += `| Mensajes de commit (Conventional Commits) | ${icon(commitStatus)} |\n`;
            body += `| Email de autor (@mobiik.com) | ${icon(emailStatus)} |\n`;
            body += `| Nombre de rama | ${icon(branchStatus)} |\n`;
            body += `| Protección de secretos | ${icon(secretsStatus)} |\n\n`;

            if (commitStatus === 'fail') {
              let errors = '';
              try { errors = fs.readFileSync('/tmp/commit_errors.txt', 'utf8'); } catch(e) {}
              body += `### :x: Commits con formato inválido\n`;
              body += `Los siguientes commits no cumplen con el formato Conventional Commits:\n${errors}\n\n`;
              body += `**Formato requerido:** \`<tipo>(<scope>)!?: <descripción>\`\n`;
              body += `**Tipos permitidos:** feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert\n\n`;
            }

            if (emailStatus === 'fail') {
              let errors = '';
              try { errors = fs.readFileSync('/tmp/email_errors.txt', 'utf8'); } catch(e) {}
              body += `### :x: Emails no corporativos\n`;
              body += `Los siguientes commits no usan email @mobiik.com:\n${errors}\n\n`;
              body += `**Solución:** \`git config user.email "nombre@mobiik.com"\` y luego \`git commit --amend --reset-author\`\n\n`;
            }

            if (branchStatus === 'fail') {
              body += `### :x: Nombre de rama inválido\n`;
              body += `La rama \`${branchName}\` no cumple con la convención de nombres.\n\n`;
              body += `**Patrones permitidos:** main, develop, qa, feature/*, bugfix/*, hotfix/*, release/vX.Y.Z, support/*\n\n`;
            }

            if (secretsStatus === 'fail') {
              let blocked = '';
              let leaks = '';
              try { blocked = fs.readFileSync('/tmp/blocked_files.txt', 'utf8'); } catch(e) {}
              try { leaks = fs.readFileSync('/tmp/secret_leaks.txt', 'utf8'); } catch(e) {}

              body += `### :x: Secretos detectados\n`;
              if (blocked) {
                body += `**Archivos bloqueados (variables de entorno / credenciales):**\n${blocked}\n\n`;
              }
              if (leaks) {
                body += `**Patrones de secretos encontrados en contenido:**\n${leaks}\n\n`;
              }
              body += `**Solución:** Elimina los archivos/secretos del PR. Usa \`.env.example\` para templates y variables de entorno reales en tu plataforma de deployment.\n\n`;
            }

            if (commitStatus === 'pass' && emailStatus === 'pass' && branchStatus === 'pass' && secretsStatus === 'pass') {
              body += `:tada: **Todas las validaciones pasaron correctamente.**\n`;
            }

            // Find existing comment to update
            const { data: comments } = await github.rest.issues.listComments({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
            });

            const botComment = comments.find(c =>
              c.user.type === 'Bot' && c.body.includes('Validación de Reglas Mobiik')
            );

            if (botComment) {
              await github.rest.issues.updateComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                comment_id: botComment.id,
                body,
              });
            } else {
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                body,
              });
            }

      - name: Fallar si hay errores
        if: always()
        run: |
          if [ "${{ steps.commits.outputs.status }}" = "fail" ] || \
             [ "${{ steps.emails.outputs.status }}" = "fail" ] || \
             [ "${{ steps.branch.outputs.status }}" = "fail" ] || \
             [ "${{ steps.secrets.outputs.status }}" = "fail" ]; then
            echo "::error::Una o más validaciones de Mobiik fallaron. Revisa el comentario del PR para más detalles."
            exit 1
          fi
```
