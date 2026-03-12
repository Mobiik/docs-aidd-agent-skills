# .gitignore Template for Mobiik Projects

Always generate or complement a `.gitignore` with these patterns when setting up a new repository. If a `.gitignore` already exists, append only the missing patterns from the secrets section.

```gitignore
# ══════════════════════════════════════════
# Mobiik - Secrets & Environment Variables
# ══════════════════════════════════════════

# Environment variable files
.env
.env.local
.env.development
.env.production
.env.staging
.env.*.local

# Cloud credentials
credentials.json
serviceAccountKey.json
gcloud-service-key.json
firebase-adminsdk*.json

# Certificates and private keys
*.pem
*.key
*.p12
*.pfx
*.jks

# SSH keys
id_rsa
id_ed25519
id_ecdsa

# Generic secrets and tokens
*.secret
token.json

# Package manager auth
.npmrc
.pypirc

# ══════════════════════════════════════════
# Common IDE and OS files
# ══════════════════════════════════════════

# macOS
.DS_Store
.AppleDouble
.LSOverride
._*
.Spotlight-V100
.Trashes

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini

# Linux
*~
.directory

# JetBrains IDEs
.idea/

# VSCode / Cursor
.vscode/
!.vscode/settings.json
!.vscode/extensions.json

# ══════════════════════════════════════════
# Dependencies and build artifacts
# ══════════════════════════════════════════

# Node.js
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*

# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
.venv/
venv/

# Java / Kotlin
*.class
*.jar
*.war
build/
.gradle/

# General build
out/
tmp/
temp/
coverage/
.nyc_output/

# Logs
*.log
logs/
```

## Usage Notes

- **Always include** the secrets section — it is mandatory for all Mobiik repos
- The IDE/OS and dependencies sections are recommended but can be adapted per project
- `.env.example`, `.env.sample`, `.env.template`, and `.env.dist` are NOT ignored — they are safe to commit
- When complementing an existing `.gitignore`, only add missing patterns from the secrets section to avoid breaking project-specific rules
