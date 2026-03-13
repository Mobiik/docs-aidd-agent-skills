---
name: mobiik-github-rules
description: Enforces Mobiik organization GitHub branch protection, contribution rules, and secrets leak prevention across all repositories. Compatible with any AI-powered editor or assistant (Cursor, VS Code, Windsurf, Claude Code, GitHub Copilot, etc.). Use this skill proactively whenever the AI agent detects git operations (commits, branch creation, push, PR preparation), repository setup, file creation involving environment variables or credentials, or when a developer asks about commit format, branch naming, email configuration, PR requirements, git hooks, or secrets protection. Also triggers when setting up a new repo, generating git hooks, or creating GitHub Actions workflows for rule enforcement. Applies to ALL Mobiik repositories and ALL branches.
---

# Mobiik GitHub Rules

Organization-level ruleset `branch-protection` (Active). Applies to **all repos** and **all branches**.

## Proactive Behavior

Do NOT wait to be asked. Automatically:
- Check `git config user.email` on first commit operation or new repo setup — block if not `@mobiik.com`
- Validate branch name on creation or switch — warn immediately if invalid
- Validate commit messages before/during commit
- Block force push attempts
- Remind PR requirements when developer mentions pushing or creating a PR
- Offer to install git hooks when setting up a new repo
- **Block commits containing secret files** — detect `.env`, credentials, private keys, tokens in staged files
- **Scan file contents for leaked secrets** — detect API keys, private keys, tokens, passwords in code
- **Generate or complement `.gitignore`** on every new repo setup — always include secrets patterns from [references/gitignore-template.md](references/gitignore-template.md)
- **Never write real secrets** in any file — always use placeholders like `your-api-key-here`, `changeme`, `<TOKEN>`

## Commit Message Validation (Conventional Commits)

**Pattern:** `<type>(<scope>)!?: <description>`

**Regex:** `^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?!?: [a-z].+`

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

Rules:
- Type is mandatory, must be from the allowed list
- Scope is optional, in parentheses
- `!` optional (breaking change)
- Colon + space after type/scope mandatory
- Description must start with **lowercase**

Valid: `feat: add user authentication` | `fix(auth): resolve token expiration` | `feat(api)!: change response format`

Invalid and why:
- `updated the login page` — missing type
- `update: changed button color` — "update" not a valid type
- `feat add new feature` — missing colon
- `fix: Added validation` — uppercase after colon

When detecting an invalid message, explain the error and provide corrected version.

## Author Email Validation

- Author email MUST match `@mobiik.com`
- Committer email: `@mobiik.com` OR `noreply@github.com` (GitHub merges)

Before any commit, check: `git config user.email`

Fix if wrong:
```bash
git config user.email "nombre@mobiik.com"
# If commit already made:
git commit --amend --reset-author
```

## Branch Naming Convention

**Full regex:** `^(main|develop|qa|feature\/.+|bugfix\/.+|hotfix\/.+|release\/v[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9.]+)?|support\/.+|dependabot\/.+|renovate\/.+)$`

| Pattern | Example |
|---------|---------|
| Main branches | `main`, `develop`, `qa` |
| Feature | `feature/user-auth` |
| Bugfix | `bugfix/issue-123` |
| Hotfix | `hotfix/security-patch` |
| Release | `release/v1.2.3`, `release/v2.0.0-beta.1` |
| Support | `support/client-xyz` |
| Dependabot | `dependabot/*` |
| Renovate | `renovate/*` |

Invalid: `user-authentication` (no prefix), `my-feature/something` (invalid prefix), `feature/my feature` (spaces), `release/new-version` (no semver)

Fix: `git branch -m old-name feature/correct-name && git push origin -u feature/correct-name`

## Branch Protection Rules

- **No deletion**: Branches cannot be deleted
- **No force push**: `git push --force` and `--force-with-lease` are **prohibited**
- Exception: `--force-with-lease` only when fixing author email on an unshared branch — confirm context before allowing

If developer attempts force push, block and explain why.

## Pull Request Requirements

- Minimum **1 approval** required
- All review conversation threads must be **resolved**
- Approvals **dismissed** on new push (re-review needed)
- Merge methods: merge, squash, rebase
- Code scanning must pass: **CodeQL** (blocks on HIGH+), **ESLint** (blocks on errors), **PMD** (blocks on errors)
- **Copilot Code Review** runs automatically on push (not on drafts)

Remind developers of these before PR creation.

## Secrets & Environment Variables Protection

### Philosophy

Defense in depth — 3 layers, each covers failures of the previous:
1. `.gitignore` (passive prevention) — files never tracked
2. `pre-commit` hook (active local block) — catches `git add -f` and content leaks
3. GitHub Actions workflow (server-side block) — impossible to bypass

### Blocked File Patterns

Files matching these patterns are **always blocked** from commits:

| Pattern | Description |
|---------|-------------|
| `.env`, `.env.local`, `.env.development`, `.env.production`, `.env.staging` | Environment variable files |
| `.env.*.local` | Local env overrides |
| `credentials.json`, `serviceAccountKey.json` | Cloud credentials |
| `gcloud-service-key.json`, `firebase-adminsdk*.json` | GCP/Firebase keys |
| `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.jks` | Certificates and private keys |
| `*.secret`, `token.json` | Generic secrets/tokens |
| `.npmrc`, `.pypirc` | Package manager auth tokens |
| `id_rsa`, `id_ed25519`, `id_ecdsa` | SSH private keys |

### Allowed Exceptions (Whitelist)

Files ending with these suffixes are **explicitly allowed**:
- `.example` — e.g. `.env.example`, `credentials.example.json`
- `.sample` — e.g. `.env.sample`
- `.template` — e.g. `.env.template`
- `.dist` — e.g. `.env.dist`

### Content Scanning Patterns (Conservative)

Scan **all staged file contents** for these high-confidence patterns:

| Pattern | Detects |
|---------|---------|
| `-----BEGIN (RSA\|EC\|DSA\|OPENSSH\|PGP) PRIVATE KEY-----` | Private keys |
| `AKIA[0-9A-Z]{16}` | AWS Access Key IDs |
| `ghp_[A-Za-z0-9]{36,}` | GitHub Personal Access Tokens |
| `gho_[A-Za-z0-9]{36,}` | GitHub OAuth Tokens |
| `ghs_[A-Za-z0-9]{36,}` | GitHub Server Tokens |
| `glpat-[A-Za-z0-9\-]{20,}` | GitLab Personal Access Tokens |
| `sk-[A-Za-z0-9]{20,}` | OpenAI / Stripe secret keys |
| `xox[bpors]-[A-Za-z0-9\-]{10,}` | Slack Tokens |
| `SG\.[A-Za-z0-9\-]{22}\.[A-Za-z0-9\-]{43}` | SendGrid API Keys |
| `AIza[A-Za-z0-9\-_]{35}` | Google API Keys |

These patterns have very low false-positive rates. Do NOT scan binary files, lock files, or files larger than 1MB.

### Bypass Flag

For legitimate cases (e.g., committing a test fixture with a fake key), developers can bypass secrets scanning:

```bash
SKIP_SECRETS_SCAN=1 git commit -m "test: add mock credentials fixture"
```

This bypass must be documented in the commit message explaining why. The GitHub Actions workflow **cannot** be bypassed — it always runs on PRs.

### .gitignore Generation

On every new repo setup, **always generate or complement** the `.gitignore` using the template in [references/gitignore-template.md](references/gitignore-template.md). If a `.gitignore` already exists, append only the missing secrets patterns without duplicating entries.

## Recommended Workflow

```bash
git checkout develop && git pull origin develop
git checkout -b feature/descriptive-name
git add . && git commit -m "feat(module): add new functionality"
git push -u origin feature/descriptive-name
# Create PR → wait for review → resolve comments → merge
```

## Troubleshooting Auto-Fixes

| Error | Fix |
|-------|-----|
| Email doesn't match | `git config user.email "nombre@mobiik.com"` then `git commit --amend --reset-author` |
| Invalid commit message | `git commit --amend -m "feat: corrected message"` |
| Invalid branch name | `git branch -m old-name feature/correct-name` then `git push origin -u feature/correct-name` |
| Blocked secret file detected | Remove from staging: `git reset HEAD <file>` then add to `.gitignore` |
| Secret pattern in file content | Remove the secret, use env variable or vault reference instead |
| Legitimate file blocked | Use bypass: `SKIP_SECRETS_SCAN=1 git commit -m "reason"` |
| Code scanning vulnerabilities | Guide to Security tab, fix code, push |
| PR needs approval | Minimum 1 reviewer, all threads resolved |

## Git Hooks Generation

When asked to set up hooks, generate the files described in [references/git-hooks.md](references/git-hooks.md). This includes:
1. `.githooks/commit-msg` — validates conventional commit format
2. `.githooks/pre-commit` — validates author email is `@mobiik.com` **and** scans for blocked secret files and secret content patterns
3. `.githooks/pre-push` — validates branch name pattern
4. `setup-hooks.sh` — configures hooksPath, makes hooks executable, validates email

All hook error messages must be in **Spanish**. Code comments in English.

### Cross-platform compatibility (macOS, Linux, Windows Git Bash)

Hooks run via `#!/usr/bin/env bash`. On Windows, Git for Windows executes hooks through its bundled MSYS2 bash regardless of the user's terminal (PowerShell, cmd, etc.). Follow these rules to guarantee portability:

| Rule | Wrong | Correct | Why |
|------|-------|---------|-----|
| End-of-options for grep | `grep -qE "-----BEGIN..."` | `grep -qE -- "-----BEGIN..."` | BSD grep (macOS) interprets leading dashes as options |
| Escape sequences output | `echo -e "$VAR"` | `printf '%b\n' "$VAR"` | `echo -e` behavior is POSIX-undefined; fails on some shells/systems |
| Head first line | `head -1` | `head -n 1` | `head -1` is deprecated POSIX syntax |

Always apply these rules when generating hooks or workflows.

## .gitignore Generation

When setting up a new repo, **always** generate or complement the `.gitignore` using the template in [references/gitignore-template.md](references/gitignore-template.md).

## GitHub Actions Workflow Generation

When asked, generate `.github/workflows/mobiik-rules.yml` as described in [references/github-actions.md](references/github-actions.md). The workflow:
- Triggers on `pull_request`
- Validates commit messages, author emails, branch names
- **Scans for blocked secret files and secret content patterns in changed files**
- Posts summary comment with pass/fail per check
- Error messages in **Spanish**

## Bypass (Emergencies Only)

Team `andreshma` has bypass permissions. Use **only** when absolutely necessary and document the reason.
