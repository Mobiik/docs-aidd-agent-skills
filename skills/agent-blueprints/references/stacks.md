# Stack-Specific Adaptations

Reference file loaded by AgentBlueprints skill when needed.

## Node.js / TypeScript Fullstack

- Lint: `npx biome check .` or `npx eslint .`
- Test: `npx vitest run` or `npx jest`
- Type check: `npx tsc --noEmit`
- Verify: lint + type check + test
- Naming: camelCase vars, PascalCase components, SCREAMING_SNAKE constants
- Frameworks: Next.js, Remix, Astro, Express, Hono, Fastify
- ORMs: Prisma, Drizzle, TypeORM, Sequelize
- Detect via: package.json → dependencies/devDependencies

## Python Backend

- Lint: `ruff check .` or `flake8`
- Test: `pytest` or `python -m pytest`
- Type check: `mypy .` or `pyright`
- Naming: snake_case everything, PascalCase classes
- Frameworks: Django, FastAPI, Flask, Litestar
- ORMs: SQLAlchemy, Django ORM, Tortoise, Peewee
- Detect via: pyproject.toml, requirements.txt, Pipfile

## Go Backend

- Lint: `golangci-lint run`
- Test: `go test ./...`
- Naming: camelCase unexported, PascalCase exported
- Frameworks: Gin, Fiber, Echo, Chi, standard library
- ORMs: GORM, sqlx, ent, Bun
- Detect via: go.mod

## PHP Backend

- Lint: `./vendor/bin/phpstan analyse` or `./vendor/bin/phpcs`
- Test: `./vendor/bin/phpunit`
- Naming: camelCase methods, PascalCase classes, snake_case DB
- Frameworks: Laravel, Symfony
- ORMs: Eloquent, Doctrine
- Detect via: composer.json

## Ruby Backend

- Lint: `rubocop`
- Test: `rspec` or `rails test`
- Naming: snake_case everything, PascalCase classes
- Frameworks: Rails, Sinatra
- ORMs: ActiveRecord, Sequel
- Detect via: Gemfile

## Rust Backend

- Lint: `cargo clippy`
- Test: `cargo test`
- Naming: snake_case fns/vars, PascalCase types, SCREAMING_SNAKE constants
- Frameworks: Axum, Actix-web, Rocket
- ORMs: Diesel, SeaORM, sqlx
- Detect via: Cargo.toml

## Frontend-Specific

### React
- Components: functional only, hooks for state
- Patterns: custom hooks for shared logic, context for DI
- Files: component.tsx + component.test.tsx (co-located)

### Vue
- Components: Composition API (script setup)
- Patterns: composables for shared logic
- Files: Component.vue (SFC)

### Svelte
- Components: .svelte files with runes ($state, $derived)
- Patterns: stores for shared state

### CSS Approaches
- Tailwind: utility-first, @apply only in base components
- CSS Modules: .module.css co-located with component
- Styled Components: sc-* naming, theme for constants
