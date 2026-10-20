# Contributing to watchpapa.tv

This guide shows you how to set up a local copy of watchpapa.tv, make a change and check it. It also covers configuration, deployment and common setup problems.

## Table of contents

- [Before you start](#before-you-start)
- [Set up a local copy](#set-up-a-local-copy)
- [Configuration](#configuration)
- [Project layout](#project-layout)
- [Code rules](#code-rules)
- [Make a change](#make-a-change)
- [Write commit messages](#write-commit-messages)
- [Run the tests](#run-the-tests)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)
- [Report a bug or ask a question](#report-a-bug-or-ask-a-question)

## Before you start

watchpapa.tv is a personal project under an all-rights-reserved licence. The project does not accept outside contributions without written permission. To offer help, contact the owner through the [contact page](https://watchpapa.tv/contact) first.

> [!NOTE]
> This file belongs to the technical documentation module. Links to files such as `CONTEXT.md`, `Documentation/` and the migrations point to the original watchpapa.tv repository. Some of these links may not work.

## Set up a local copy

### Prerequisites

* Node.js 22 and npm
  ```sh
  node --version   # should print v22.x
  ```
* A [TMDB API key](https://www.themoviedb.org/settings/api)
* A Supabase project that has the watchpapa schema. You need its URL, publishable key and session-pooler connection string. The numbered files in [Backend/src/db/migrations/](https://github.com/watchpapa/watchpapa/tree/production/Backend/src/db/migrations) create the schema.

### Installation

1. Clone the repository.
   ```sh
   git clone https://github.com/watchpapa/watchpapa.git
   cd watchpapa
   ```
2. Install the packages for the Worker and the frontend.
   ```sh
   (cd worker && npm install) && (cd Frontend && npm install)
   ```
   Both installs finish without errors. Only the root test suites need the root `package.json`.
3. Create the local configuration files from the examples. Then add your TMDB key and your Supabase URL and key.
   ```sh
   cp worker/.dev.vars.example worker/.dev.vars   # TMDB_API_KEY_SECRET
   cp Frontend/.env.example Frontend/.env         # VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY
   ```
4. Start the Worker API. Put the database connection string in a shell variable, not in `.dev.vars`.
   ```sh
   export CLOUDFLARE_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE="postgres://USER:PASS@HOST:5432/postgres?sslmode=require"
   cd worker && npm run dev
   ```
   Wrangler starts the API on `http://localhost:8787`.
5. Open a second terminal and check that the API runs.
   ```sh
   curl http://localhost:8787/health
   ```
   You should see this output.
   ```json
   {"status":"ok"}
   ```
6. Start the frontend.
   ```sh
   cd Frontend && npm run dev
   ```
7. Open `http://localhost:5173`. The home page shows live posters from TMDB. Vite forwards `/api` requests to the Worker, so you do not need to configure cross-origin access locally.

<p align="right">(<a href="#contributing-to-watchpapatv">back to top</a>)</p>

## Configuration

| What | Where | Notes |
|---|---|---|
| TMDB API key | `worker/.dev.vars` locally. `wrangler secret put TMDB_API_KEY_SECRET` in production. | A secret. Never commit it. |
| Database connection | Shell variable `CLOUDFLARE_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE` locally. `HYPERDRIVE` binding in production. | Use the Supabase session pooler URL. |
| Worker settings | `vars` in [worker/wrangler.jsonc](https://github.com/watchpapa/watchpapa/blob/production/worker/wrangler.jsonc) | `ALLOWED_ORIGINS`, `SUPABASE_URL`, `BATCH_MAX`, `BATCH_SUBREQ_BUDGET`, `IMPORT_CHUNK_MAX`, `RELEASES_MAX_SHOWS`, `SITEMAP_PAGES`, `RECOMMENDATIONS_MAX_SEEDS`, `RECOMMENDATIONS_PROVIDER_CHECK_MAX`, `SHOW_STATUS_BACKFILL_MAX` |
| Rate limits and cron | `worker/wrangler.jsonc` | The `RL_GLOBAL` and `RL_MUTATION` bindings, and a cron trigger that runs every minute |
| Frontend settings | `Frontend/.env` locally. Pages environment variables in production. | `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY`, `VITE_API_BASE_URL` |
| Test settings | Root `.env` | `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `TMDB_API_KEY_SECRET` |
| Backup and deploy secrets | GitHub Actions secrets | `SUPABASE_DB_POOLER_URL`, `AGE_PUBLIC_KEY`, `R2_*`, `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` |
| Database schema | `Backend/src/db/migrations/` | Numbered SQL migrations that you only append to |

> [!IMPORTANT]
> In production, set `VITE_API_BASE_URL` to `https://api.watchpapa.tv`. The frontend calls the Worker from another origin. Without this value, the app cannot reach the API.

<p align="right">(<a href="#contributing-to-watchpapatv">back to top</a>)</p>

## Project layout

```text
watchpapa/
├── Frontend/                 # React app (Cloudflare Pages)
│   ├── functions/            #   Pages Function: real 404s and link-preview tags
│   ├── src/pages/            #   one component per route: app/, auth/, admin/
│   ├── src/features/         #   data hooks grouped by domain (content, rating, follows)
│   ├── src/components/       #   shared interface components
│   └── src/lib/              #   API client, Supabase client, TMDB image URLs
├── worker/                   # Cloudflare Worker API (Hono)
│   ├── src/routes/           #   one file per route group, admin routes in routes/admin/
│   ├── src/tmdb/             #   TMDB client, normalisation, recommendations, adult-content filter
│   ├── src/import/           #   CSV import: resolve titles, commit rows
│   ├── test/                 #   Vitest unit tests
│   └── wrangler.jsonc        #   bindings, variables, cron trigger, routes
├── Backend/src/db/migrations/  # numbered SQL migrations
├── scripts/db-backup.sh      # nightly encrypted database backup
├── tests/                    # rls_tests, unit, e2e, backup, smoke, plus pytest checks
├── Documentation/            # release notes, deployment, database, security, compliance
├── .github/workflows/        # nightly database backup, manual Worker deploy
└── CONTEXT.md                # codebase reference: routes, schema, conventions
```

## Code rules

Follow these rules when you change code.

- Never store content such as movies, shows or people. Read it live through the Worker. Use the hooks in `Frontend/src/features/content/` to call those endpoints.
- Key user tables by `tmdb_id`. The frontend reads them with supabase-js under row-level security (RLS). Route anything that needs several tables at once through the Worker.
- Do not pass arrays as bind parameters in Worker queries. The database connection sets `fetch_types: false`, so `= ANY(${arr})` fails. Pass a comma-separated string with `string_to_array(...)` or use `IN ${sql(arr)}`.
- Add a new numbered migration file. Do not edit an existing one.
- Update [CONTEXT.md](https://github.com/watchpapa/watchpapa/blob/production/CONTEXT.md) after every meaningful change.

## Make a change

1. Agree the change with the owner. See [Before you start](#before-you-start).
2. Create a branch for your change. A push to the `production` branch deploys automatically, so do not push to it directly.
3. Make the change and follow the [code rules](#code-rules).
4. Run the tests that cover your change. See [Run the tests](#run-the-tests).
5. Update `CONTEXT.md` and any other documentation that the change affects.
6. Commit your work. Follow the [commit message rules](#write-commit-messages). If an AI tool helped, add a `Co-authored-by` line that names the tool.
7. Open a pull request and describe what you changed and why.

## Write commit messages

A clear commit message tells the next reader what changed and why. Follow these rules.

1. Use this structure.
   ```text
   <type>(<scope>): <summary>

   <body>

   <footer>
   ```
2. Choose one type.

   | Type | Use for |
   |---|---|
   | `feat` | A new feature |
   | `fix` | A bug fix |
   | `docs` | A documentation change |
   | `refactor` | A code change that does not alter behaviour |
   | `test` | A new or updated test |
   | `chore` | Build, tooling or dependency work |

3. Add a scope in brackets when it helps. Use the part of the project that changed, such as `worker`, `frontend` or `db`.
4. Write the summary in the imperative mood. Start with a verb, as in "Add follow limit check". Keep it under 72 characters. Do not end it with a period.
5. Leave one blank line after the summary. Use the body to explain why you made the change, not what the code does. Wrap lines at 72 characters.
6. Make one logical change per commit. Do not mix a fix with a refactor.
7. Put references in the footer, such as `Closes #12`. Add a `Co-authored-by` line if an AI tool helped.

This is a good example.

```text
fix(worker): stop passing arrays as bind parameters

The database connection sets fetch_types: false, so = ANY(${arr})
fails with a 500 error. Pass a comma-separated string instead.

Closes #12
```

## Run the tests

Run `npm install` once in the repository root before you run the root test suites.

| Command | Where | What it does |
|---|---|---|
| `npm run dev` | `Frontend/` | Starts the Vite development server |
| `npm run build` | `Frontend/` | Creates a production build in `dist/` |
| `npm run lint` | `Frontend/` | Runs ESLint |
| `npm run dev` | `worker/` | Starts the Worker locally with Wrangler |
| `npm test` | `worker/` | Runs the Worker unit tests |
| `npm run test:rls` | root | Tests the RLS policies and audit triggers against a live database inside rolled-back transactions |
| `npm run test:unit` | root | Tests the breached-password check and the password-write guard |
| `npm run test:e2e` | root | Runs Playwright against the real interface with all network calls mocked. Run `cd Frontend && npx playwright install chromium` first. |
| `npm run test:backup` | root | Tests the backup pipeline and runs a restore test. Needs Docker and [`age`](https://github.com/FiloSottile/age). |
| `npm run test:smoke` | root | Checks the live Supabase project and Worker. Creates and deletes one temporary user. |

<p align="right">(<a href="#contributing-to-watchpapatv">back to top</a>)</p>

## Deployment

A push to the `production` branch deploys both parts automatically.

| Part | How it deploys |
|---|---|
| Frontend | Cloudflare Pages git build. The project is `watchpapa` and the root directory is `Frontend/`. |
| Worker API | Cloudflare Workers Builds. The project is `watchpapa-api` and the root directory is `worker/`. The API runs at `api.watchpapa.tv`. |
| Database | You apply new files from `Backend/src/db/migrations/` to Supabase. |

If Workers Builds is unavailable, run the **Deploy Worker (manual backup)** workflow from the GitHub Actions tab.

A nightly GitHub Actions job backs up the database. It runs `pg_dump`, encrypts the dump with `age` and uploads it to a private Cloudflare R2 bucket. The bucket keeps backups for 30 days. The Supabase Free plan has no built-in backups. [Documentation/deployment.md](https://github.com/watchpapa/watchpapa/blob/production/Documentation/deployment.md) describes the secrets and the restore steps.

## Troubleshooting

| Problem | What to check |
|---|---|
| The site loads but every API call fails | Set `VITE_API_BASE_URL` to `https://api.watchpapa.tv` in the Pages environment. Add your origin to `ALLOWED_ORIGINS`. |
| Sign-in or database calls fail in the browser | Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` to the Pages production environment. Make sure no `Frontend/wrangler.toml` file exists. |
| The Worker returns 500 errors on user-data routes | Run `npx wrangler tail` and check the Hyperdrive binding ID in `wrangler.jsonc`. Also check that no query passes an array as a bind parameter. |
| A background import stops | Run `npx wrangler tail` and look for `[IMPORT]` lines every minute. The next run retries a step that fails. |
| Unknown URLs return 200 instead of 404 | Deploy `Frontend/functions/_middleware.js`. The Pages root directory must be `Frontend/`. |
| The Supabase project is paused | The Free plan pauses inactive projects. The Worker's cron query normally prevents this. Restore the project in the Supabase dashboard. |

## Report a bug or ask a question

To report a bug or ask a question, [open an issue](https://github.com/watchpapa/watchpapa/issues) or use the [contact page](https://watchpapa.tv/contact). Include the steps that cause the problem and what you expected to happen.

<p align="right">(<a href="#contributing-to-watchpapatv">back to top</a>)</p>
