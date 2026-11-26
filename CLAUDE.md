# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

Freshly generated Symfony 7.4 "webapp" skeleton (PHP >= 8.2). `src/Controller`, `src/Entity`, `src/Repository`, `migrations/` and `tests/` are still empty — there is no application code yet. Code lives under the `App\` namespace (`src/`), tests under `App\Tests\` (`tests/`).

The repo sits inside WSL (`/home/formaweb/SymClaude`, accessed from Windows via `\\wsl.localhost\Ubuntu\...`). Run `php`, `composer` and `bin/console` from the Linux side (e.g. `wsl -d Ubuntu --cd /home/formaweb/SymClaude <cmd>`), not with a Windows PHP.

## Commands

```bash
composer install                      # also runs cache:clear, assets:install, importmap:install
docker compose up -d                  # Postgres 16 (+ Mailpit via compose.override.yaml)
symfony serve -d                      # dev server, if the Symfony CLI is installed

php bin/console make:entity | make:controller | make:migration   # MakerBundle generators
php bin/console doctrine:migrations:migrate

php bin/phpunit                                   # all tests
php bin/phpunit tests/Path/To/SomeTest.php        # one file
php bin/phpunit --filter testMethodName           # one test
```

PHPUnit (v13) is configured with `failOnDeprecation`, `failOnNotice` and `failOnWarning` — any deprecation triggered from `src/` fails the suite. Tests run with `APP_ENV=test` (`.env.test`). No linter / static analyser is installed.

## Architecture notes

- **Config**: standard Symfony layout; services are autowired/autoconfigured from `src/` (`config/services.yaml`), routes come from controller attributes (`config/routes.yaml`).
- **Database**: Doctrine ORM 3 + Migrations, PostgreSQL 16 (`DATABASE_URL` in `.env`, container defined in `compose.yaml`). Override secrets in `.env.local`, not `.env`.
- **Messenger**: emails (`SendEmailMessage`) and notifier Chat/SMS messages are routed to the `async` transport (Doctrine, `MESSENGER_TRANSPORT_DSN`), so they are only delivered when a worker runs: `php bin/console messenger:consume async`. Failures go to the `failed` transport. `MAILER_DSN` defaults to `null://null`; point it at Mailpit to see mail locally.
- **Security**: only an in-memory user provider placeholder and an empty `main` firewall — no real authentication or access control yet.
- **Frontend**: no Node/Webpack. AssetMapper + importmap (`importmap.php`, add packages with `php bin/console importmap:require <pkg>`), Stimulus controllers in `assets/controllers/` (auto-registered via `assets/controllers.json`), and Turbo (Hotwire) enabled — forms/links are Turbo-driven by default. Entry point is `assets/app.js`; templates extend `templates/base.html.twig`.
