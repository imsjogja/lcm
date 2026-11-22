# Contributing to LCM

## Development standards
- Read `README.md` and all relevant `docs/` decisions before adding features.
- PHP: PSR-12/Pint, strict validation, typed return values, small domain services, policies; avoid controller business logic.
- TypeScript: strict types, components by use case, no `any` without justification, deterministic frontend state.
- SQL: schema migrations, indexes and FK constraints; avoid N+1 queries and implicit tenant joins.
- External integrations: HTTP timeouts, signed requests, idempotency, retry/backoff, correlation IDs and redacted logs.
- Reverb: always authorize private channels and keep event payloads minimal.
- Changes to KPI or fee calculations require unit tests and an explicit rule version.
- For any auth/tenant code, add cross-tenant denial tests.

## Suggested workflow
1. Write acceptance criteria and sample input/output.
2. Implement one vertical slice with feature/unit tests.
3. Add a synthetic dataset or redacted fixture.
4. Run formatting, type checks, unit tests and browser smoke tests.
5. Review security/tenant scoping, query plans and queue failure recovery before merge.

## Commit style
Use `docs:`, `feat:`, `fix:`, `test:`, `refactor:`, `chore:` with concise descriptions. No secrets, raw vendor tokens, customer records or production dumps in Git.

## Local baseline commands once the app is bootstrapped
```sh
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
npm ci
npm run dev
php artisan serve
php artisan queue:work
php artisan reverb:start --debug
```
The commands are **planned**, not currently runnable: the repository is initially documentation-only.
